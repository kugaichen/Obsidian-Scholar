
```dataviewjs
// ================= 专业版配置区域 =================
const sourceFileName = "我的时间计划"; // 数据源文件名

// --- 专业配色方案 ---
const theme = {
    primaryBg: "#e3f2fd",      // 日期节点背景 (清新蓝)
    primaryStroke: "#64b5f6",  // 日期节点边框
    todoBg: "#ffffff",         // 待办任务背景
    todoStroke: "#e0e0e0",     // 待办任务边框 (浅灰)
    doneBg: "#e8f5e9",         // 已完成任务背景 (淡绿)
    doneStroke: "#66bb6a",     // 已完成任务边框
    lineColor: "#90a4ae",      // 默认连接线颜色 (蓝灰)
    warningColor: "#ff7043",   // 临近期限警告色 (橙红)
    todayMarkerBg: "#fff9c4",  // "今日"标记背景 (淡黄)
    edgeLabelBg: "#ffffff",    // 连接线标签背景
    overdueBg: "#f5f5f5",      // 已过时日期背景 (浅灰)
    overdueStroke: "#9e9e9e",  // 已过时日期边框 (中灰)
    overdueText: "#757575"     // 已过时文本颜色
};

// --- 时间管理配置 ---
const warningDays = 2; // 剩余天数少于多少天时显示警告色
// ===================================================

// --- 工具函数 ---
// 计算两个日期之间的天数差
function getDaysDiff(dateStr1, dateStr2) {
    const d1 = new Date(dateStr1); d1.setHours(0,0,0,0);
    const d2 = new Date(dateStr2); d2.setHours(0,0,0,0);
    if (isNaN(d1) || isNaN(d2)) return null;
    return Math.round((d2 - d1) / (1000 * 60 * 60 * 24));
}
// 获取今天的日期字符串 (YYYY-MM-DD)
function getTodayStr() {
    const d = new Date();
    return `${d.getFullYear()}-${String(d.getMonth() + 1).padStart(2, '0')}-${String(d.getDate()).padStart(2, '0')}`;
}

// --- 核心逻辑 ---
const page = dv.page(sourceFileName);
if (!page) {
    dv.paragraph(`⚠️ 错误：找不到笔记 "${sourceFileName}"。`);
} else {
    const tFile = app.vault.getAbstractFileByPath(page.file.path);
    if (!tFile) dv.paragraph(`⚠️ 错误：无法获取文件对象。`);
    else {
        const content = await app.vault.read(tFile);
        const todayStr = getTodayStr();

        // --- 1. 解析数据 ---
        function parseTimelineData(text) {
            const lines = text.split('\n'); const data = []; let current = null;
            const dateRgx = /^###\s+(\d{4}-\d{1,2}-\d{1,2})(.*)/;
            const taskRgx = /^\s*-\s*\[([ xX])\]\s*(.*)/;
            for (let line of lines) {
                line = line.trim(); if (!line) continue;
                const dMatch = line.match(dateRgx);
                if (dMatch) {
                    if (current) data.push(current);
                    current = { date: dMatch[1], subTitle: dMatch[2].trim().replace(/^\|/, '').trim(), tasks: [], allDone: true, hasTasks: false };
                } else if (current) {
                    const tMatch = line.match(taskRgx);
                    if (tMatch) {
                        current.hasTasks = true; const isDone = tMatch[1] !== ' ';
                        let txt = tMatch[2].trim().replace(/"/g, "'");
                        if (isDone) txt = `✅ ${txt}`; else { txt = `⬜ ${txt}`; current.allDone = false; }
                        current.tasks.push(txt);
                    }
                }
            }
            if (current) data.push(current); return data;
        }
        const data = parseTimelineData(content);

        if (data.length === 0) dv.paragraph("⚠️ 无有效数据。请检查源文件格式。");
        else {
            // --- 2. 生成 Mermaid ---
            let mermaid = `%%{init: {'theme': 'base', 'themeVariables': {'primaryColor': '${theme.primaryBg}', 'edgeLabelBackground':'${theme.edgeLabelBg}', 'lineColor': '${theme.lineColor}'}}}%%\n`;
            mermaid += `graph LR\n\n`;

            // 样式定义
            mermaid += `classDef dateStyle fill:${theme.primaryBg},stroke:${theme.primaryStroke},stroke-width:2px,color:#333,rx:8,ry:8\n`;
            mermaid += `classDef todoStyle fill:${theme.todoBg},stroke:${theme.todoStroke},stroke-width:1px,color:#555,rx:6,ry:6,text-align:left,stroke-dasharray:3,3\n`;
            mermaid += `classDef doneStyle fill:${theme.doneBg},stroke:${theme.doneStroke},stroke-width:2px,color:#2e7d32,rx:6,ry:6,text-align:left,font-weight:bold\n`;
            mermaid += `classDef todayStyle fill:${theme.todayMarkerBg},stroke:#fdd835,stroke-width:3px,color:#333,rx:15,ry:15,stroke-dasharray:5,5\n`;
            // 新增：已过时样式
            mermaid += `classDef overdueStyle fill:${theme.overdueBg},stroke:${theme.overdueStroke},stroke-width:2px,color:${theme.overdueText},rx:8,ry:8,stroke-dasharray:2,2\n`;
            // 新增：已过时连接线样式
            mermaid += `classDef overdueLineStyle stroke:${theme.overdueStroke},stroke-width:2px,stroke-dasharray:3,3\n`;
            // 连接线样式
            mermaid += `linkStyle default stroke:${theme.lineColor},stroke-width:3px\n\n`;

            let nodes = [], tasks = [], dateIds = [], todoIds = [], doneIds = [], linkStyles = [];
            let overdueDateIds = []; // 存储已过时日期节点ID
            let todayNodeId = null;

            // 查找今天在时间轴中的位置
            let todayIndex = -1;
            for(let i=0; i<data.length; i++) {
                if (data[i].date === todayStr) { todayIndex = i; break; }
                if (i < data.length - 1 && todayStr > data[i].date && todayStr < data[i+1].date) { todayIndex = i + 0.5; break; }
            }

            // 创建节点
            data.forEach((item, i) => {
                const did = `D${i}`, tid = `T${i}`;
                let label = `**${item.date}**`;
                if (item.subTitle) label += `<br>${item.subTitle}`;
                
                // 检查日期是否已过时（早于今天）
                const isOverdue = item.date < todayStr;
                
                // 如果今天是某个节点日期，在标签中加入标记
                if (i === todayIndex) {
                    label += `<br>📍今日`;
                } else if (isOverdue) {
                    // 已过时日期添加标记
                    label += `<br>⏰已过时`;
                }
                
                nodes.push(`${did}["${label}"]`);
                dateIds.push(did);
                
                // 如果是已过时日期，记录到overdueDateIds
                if (isOverdue) {
                    overdueDateIds.push(did);
                }
                
                if (item.hasTasks) {
                    tasks.push(`${did} --- ${tid}["${item.tasks.join('<br>')}"]`);
                    item.allDone ? doneIds.push(tid) : todoIds.push(tid);
                }
            });

            // 拼接主干与时间标签
            mermaid += `%% 主干时间线\n`;
            for (let i = 0; i < nodes.length; i++) {
                mermaid += nodes[i];
                if (i < nodes.length - 1) {
                    const nextDate = data[i+1].date;
                    const totalDays = getDaysDiff(data[i].date, nextDate);
                    const daysFromToday = getDaysDiff(todayStr, nextDate);
                    
                    let label = `**共${totalDays}天**`;
                    let isWarning = false;
                    let isOverdueLine = false; // 标记是否为已过时连接线

                    // 智能显示进度与警告
                    if (todayStr > data[i].date && todayStr < nextDate) {
                        const passed = getDaysDiff(data[i].date, todayStr);
                        label = `已过${passed}天 / **剩${daysFromToday}天**`;
                        if (daysFromToday <= warningDays && daysFromToday >= 0) isWarning = true;
                    } else if (todayStr <= data[i].date) {
                         if (totalDays <= warningDays) isWarning = true;
                    } else if (todayStr > nextDate) {
                        // 整个时间段都已过时
                        label = `⏰已过时 ${totalDays}天`;
                        isOverdueLine = true;
                    }

                    mermaid += ` -- |${label}| --> `;
                    // 设置警告色连接线
                    if (isWarning) {
                        linkStyles.push(`linkStyle ${i} stroke:${theme.warningColor},stroke-width:4px;`);
                    }
                    // 设置已过时连接线样式
                    if (isOverdueLine) {
                        linkStyles.push(`linkStyle ${i} stroke:${theme.overdueStroke},stroke-width:2px,stroke-dasharray:3,3;`);
                    }
                }
            }
            mermaid += `\n\n`;

            // 如果今天在两个日期之间，添加今日标记节点
            if (todayIndex !== -1 && todayIndex !== Math.floor(todayIndex)) {
                const prevNodeIdx = Math.floor(todayIndex);
                todayNodeId = "TM";
                mermaid += `${todayNodeId}(("📍<br>今日"))\n`;
                mermaid += `D${prevNodeIdx} -.-> ${todayNodeId}\n\n`;
            }
            
            // 添加任务节点
            if (tasks.length) {
                mermaid += `%% 任务节点\n`;
                mermaid += `${tasks.join('\n')}\n\n`;
            }
            
            // 应用样式
            if (dateIds.length) mermaid += `class ${dateIds.join(',')} dateStyle\n`;
            if (overdueDateIds.length) mermaid += `class ${overdueDateIds.join(',')} overdueStyle\n`;
            if (todoIds.length) mermaid += `class ${todoIds.join(',')} todoStyle\n`;
            if (doneIds.length) mermaid += `class ${doneIds.join(',')} doneStyle\n`;
            if (todayNodeId && todayIndex !== Math.floor(todayIndex)) mermaid += `class ${todayNodeId} todayStyle\n`;
            if (linkStyles.length) mermaid += `${linkStyles.join('\n')}\n`;

            // 输出 Mermaid 图表
            dv.paragraph("```mermaid\n" + mermaid + "\n```");
        }
    }
}
```


