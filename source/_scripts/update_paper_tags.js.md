async function update_paper_tags(tp) {
    // 获取当前活动文件
    const file = tp.file.find_tfile(tp.file.path(true));
    if (!file) {
        new Notice("未找到文件！");
        return;
    }

    // 使用 Obsidian 官方 API 安全修改 Frontmatter
    await app.fileManager.processFrontMatter(file, (frontmatter) => {
        // 1. 获取 topics 和 status，如果不存在则初始化为空数组
        let topics = frontmatter["topics"];
        let status = frontmatter["status"];

        // 2. 规范化数据：确保它们都是数组格式
        // 如果是单个字符串 (例如 "#reading")，转为 ["#reading"]
        // 如果是 null 或 undefined，转为 []
        const topicsList = Array.isArray(topics) ? topics : (topics ? [topics] : []);
        const statusList = Array.isArray(status) ? status : (status ? [status] : []);

        // 3. 合并数组
        let allTags = [...topicsList, ...statusList];

        // 4. 去重 (利用 Set 特性)
        let uniqueTags = [...new Set(allTags)];

        // 5. 过滤空值
        uniqueTags = uniqueTags.filter(t => t && t.trim() !== "");

        // 6. 写入 tags 字段
        frontmatter["tags"] = uniqueTags;
    });

    new Notice("Tags 已根据 Topics 和 Status 更新！");
}

module.exports = update_paper_tags;