# Awesome Skills

这里汇集了为 AI Agent 精心设计的标准化能力（Skills）与工作流（Workflows）。每一个 Skill 都经过精心打磨，旨在通过标准化的过程控制（SOP），将复杂的开发任务转化为高质量、可复用的执行步骤。

国内汇总链接：
https://thecolaris.feishu.cn/wiki/ZQ1uwtOaTiB7jqkA1UAcX4xpnZd?from=from_copylink

## 🎯 愿景 (Vision)

通过定义明确的**过程资产**（Process Assets），让 AI 的产出具备：
*   **确定性 (Determinism)**：减少随机性，确保产出符合预期。
*   **高质量 (High Quality)**：内置最佳实践、代码规范与质量门禁。
*   **可解释性 (Explainability)**：每一步都有据可依，易于审查与回溯。

## 🚀 如何使用 (Usage)

这些 Skill 旨在被 AI Agent (如 Antigravity) 在执行特定任务时加载并遵循。

1.  **加载 Skill**: Agent 在识别到相关任务时，会主动读取对应目录下的 `SKILL.md`。
2.  **遵循指引**: Agent 将严格按照 Skill 中定义的步骤（Step-by-Step）执行，并在关键节点请求用户确认。
3.  **产出交付**: 最终生成符合 Skill 定义的标准交付物（代码、测试、文档）。

## 配置地址
| 工具               | 项目路径            | 全局路径                        | 文档                                                                                    |
| ------------------ | ------------------- | ------------------------------- | ------------------------------------------------------------------------------------------- |
| **Antigravity**    | `.agent/skills/`    | `~/.gemini/antigravity/skills/` | [Antigravity Skills](https://antigravity.google/docs/skills)                                |
| **Claude Code**    | `.claude/skills/`   | `~/.claude/skills/`             | [Claude Code Skills](https://code.claude.com/docs/en/skills)                                |
| **Gemini CLI**     | `.gemini/skills/`   | `~/.gemini/skills/`             | [Gemini CLI Skills](https://geminicli.com/docs/cli/skills/)                                 |

## 常用 Skills 中文导航站
### https://skillstore.io/zh-hans/skills
一个集中展示和搜索社区 Skills 的中文导航站，帮助你快速发现和安装适合的 Skill。
### https://claude.com/download
claude 桌面版下载地址

## claude常用插件市场
### 使用方式：
从 GitHub 添加 
`/plugin marketplace add owner/repo`

例如：`/plugin marketplace add anthropics/claude-code`

从 Git URL 添加 
`/plugin marketplace add https://gitlab.com/company/plugins.git`

指定分支或标签：`/plugin marketplace add https://gitlab.com/company/plugins.git#v1.0.0`

从本地路径添加 `/plugin marketplace add ./my-marketplace`

### 地址信息：
| 名称 | 介绍 | 地址|
| -- | -- | -- |
| claude-code | claude code官方插件库 | anthropics/claude-code |



## 📦 Skills 列表

| Skill | 一句话介绍 | 项目地址 |
|-------|-----------|-----------|
| **backend-workflow** | 从详细设计文档到后端代码落地的标准化工作流，支持任务分级、质量门禁与 TDD。 ||
| **docx** | Word 文档创建、编辑与分析，支持修订追踪、批注与格式保留。 ||
| **pdf** | PDF 处理工具包，涵盖文本/表格提取、创建新 PDF、合并/拆分与表单填写。 ||
| **pptx** | PowerPoint 演示文稿创建、编辑与分析，支持 HTML 转 PPT 与模板复用。 ||
| **skill-creator** | Skill 创建指南，帮助设计和打包有效的 AI Agent 能力扩展包。 ||
| **skill-installer** | 从 GitHub 或精选列表一键安装 Skill 到本地环境。 ||
| **skill-writer** | Claude Code Agent Skill 编写指南，引导创建符合规范的 SKILL.md。 ||
| **ui-ux-pro-max** | UI/UX 设计智能库，内置 50 种风格、21 套配色、50 组字体搭配与多框架最佳实践。 ||
| **xlsx** | Excel 电子表格创建、编辑与分析，支持公式、格式与数据可视化。 ||
| **obsidian-skills** | 给 AI 注入专业的 Obsidian 知识 |https://github.com/kepano/obsidian-skills|
| **code-review-skill** |代码审查skills|https://github.com/awesome-skills/code-review-skill|







## 🤝 贡献

欢迎沉淀更多的通用工作流至此仓库，扩充 AI 的技能树。
