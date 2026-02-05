# Obsidian-Scholar

A Literature Reading, Knowledge Organization, and Time Management Workflow with Zotero and Obsidian

Obsidian-Scholar是一套面向研究型阅读的**Zotero–Obsidian联动工作流**，目标是在不增加阅读负担的前提下，实现：

- 在 **Zotero中根据颜色标记自动完成类别标注**
- 在**Obsidian中根据模板生成可检索的阅读笔记**
- 构建**基于文献笔记自动生成的可视化检索表格**

  <img src="assets\paper_map.png" al="image-20260127201813892"  />
  
- 根据计划**自动化生成时间线可视化表**

  <img src="assets\Timeline.png" alt="image-20260127211004304" />

---

### 1. 动机

在长期的论文阅读实践中，常见的痛点包括：

- Zotero中的批注被遗忘指数高且难以被系统性整理与回顾
- Obsidian中的笔记不适合阅读（个人）
- 文献整理需要进一步耗费精力

本项目的核心设计目标是：

> **让“阅读行为本身”自然地产生结构化研究数据，而不是事后整理。**

### 2. 工作流概览

整个系统由 Zotero 与 Obsidian 协同完成，其逻辑流程如下：

1. 在 Zotero中阅读PDF并进行颜色标注
2. 颜色→ 语义标签→元数据注释（自动完成）
3. Obsidian定期拉取文献与新注释
4. 基于模板生成标准化Paper Note
5. 通过Dataview构建跨文献的结构化视图

<img src="assets\workflow.png" alt="image-20260127152312407" style="zoom: 67%" />

---

### 3. Zotero侧配置

#### 3.1 核心机制

- 不同高亮颜色对应不同类型的阅读思考笔记（Background / Motivation / Method / Experiment / Contribution 等）
- Zotero在保存注释时，自动将颜色映射为标准化标签
- 标签直接写入注释文本，作为后续Obsidian导入的元数据来源

#### 3.2 相关配置

- 插件安装：

  - Actions and Tags for Zotero（用于基于颜色自动打标签）

    <img src="assets\action.png" alt="image-20260127153137617" width="700" />

  - Better BibTex for Zotero（用于规范引用格式）

    <img src="assets\bibtex.png" alt="image-20260127153028463" width="700" />

  - Ethereal Style（用于自定义标签样式）

    <img src="assets\style.png" alt="image-20260127153059400" width="700" />

#### 3.3 动作配置文件

```bash
zotero_config/actions-zotero.yml
```

---

### 4. Obsidian侧配置

#### 4.1 核心机制

- 从Zotero抓取文献与阅读注释，自动生成Paper Note
- Zotero注释以元数据形式导入，可进一步进行总结、优缺点分析与标签整理
- 基于统一的Paper Note结构，集中构建可视化研究视图

#### 4.2 插件安装

- Dataview（用于生成结构化表格）

- Zotero Integration（用于Obsidian与Zotero联动）

- Templater（用于标签与状态的自动更新）

#### 4.3 Zotero Integration配置

- 基础配置：

  <img src="assets\integration_1.png" alt="image-20260127155206020" width="700" />

- 文件与模板导入配置

  包含文件管理、图片管理、引用格式及模板配置。其中：

  - Output Path用于统一生成笔记的命名与路径结构
  - Template File用于定义导入后Paper Note的整体格式

    <img src="assets\integration.png" alt="image-20260127155247825" width="700" />

 - Paper Note模板文件：
        ```
        obsidian_config/Zotero_Import_template.md
        ```
  - 可视化效果：

    <img src="assets\note.png" alt="image-20260127201057387"  />

#### 4.4 Dateview配置

在 Paper Note 之上，本系统通过Dataview构建跨文献的统一视图，包括：

1. 按年份 / 会议排序的论文列表
2. 带 Pros / Cons 的横向比较表
3. 不同研究主题下的文献聚合

- Obsidian 外观样式（CSS）:

  <img src="assets\css.png" alt="image-20260127201303520" width="700" />

  ```
  obsidian_config/dataview-theme.css
  ```
- Dataview 查询笔记：

  ```
  source/Paper_Reading_Map.md
  ```

- 最终可视化效果：

  <img src="assets\paper_map.png" al="image-20260127201813892"  />

#### 4.5 笔记标签更新自动化脚本

- Templater配置：

  <img src="assets\Templater_1.png" alt="image-20260127204954959" width="700" />

  <img src="assets\Templater_2.png" alt="image-20260127205015945" width="700" />

  <img src="assets\Templater_3.png" alt="image-20260127205028210" width="700" />

- Templater模板文件：

  ```
  source/_templates/Tags_updating_auto_template.md
  ```
  
- 使用方式：

  - 利用Templater对模板的操作进行快捷键绑定：

    <img src="assets\tags_1.png" alt="image-20260127205756521" width="700" />

    <img src="assets\tags_2.png" alt="image-20260127205815310" width="700" />

  - 使用快捷键即可以在Paper Note中将topics和status中的元数据直接填入tags（标签中）
  
### 5. 时间线（Timeline）功能配置   

除文献阅读外，该工作流还提供一个**轻量级时间线视图**，依靠Obsidian的模块和dataviewjs功能定制，方便项目管理和计划调整

#### 5.1 核心机制

- 利用笔记对项目周期进行计划制定和关键节点提醒
- 通过dataviewjs自动对项目计划和关键节点进行抓取自动生成时间线可视化

#### 5.2 配置实现

- 时间计划笔记：

  ```
  source/我的时间计划.md
  ```
  
- Dataviewjs配置：

  ```
  source/TimeLine_for_everything.md
  ```

#### 5.3 可视化效果

<img src="assets\Timeline.png" alt="image-20260127211004304" />


### 6. 文件结构

  ```
  obsidian-scholar/
  ├── README.md
  ├── zotero_config/
  │  └── actions-zotero.yml                     //插件配置文件定义高亮颜色与语义标签之间的映射规则
  ├── obsidian_config/
  │  └── dataview-theme.css                     //表格与视图的自定义样式文件，改进显示效果
  ├── source/
  │   ├── _templates/
  │   │   └── Zotero_Import_template.md         //Note模板，定义笔记的结构与字段
  │   │   └── Tags_updating_auto_template.md    //动作模板，关联快捷键更新标签
  |	└── Paper_Reading_Map.md                  //基于Dataview的文献全局视图
  |	└── 我的时间计划.md                        //研究与项目时间规划笔记
  |	└── TimeLine_for_everything.md            //DataviewJS时间线视图
  └── LICENSE
  ```
