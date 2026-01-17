# Awesome Skills

这里汇集了为 AI Agent 精心设计的标准化能力（Skills）与工作流（Workflows）。每一个 Skill 都经过精心打磨，旨在通过标准化的过程控制（SOP），将复杂的开发任务转化为高质量、可复用的执行步骤。

## 🎯 愿景 (Vision)

通过定义明确的**过程资产**（Process Assets），让 AI 的产出具备：
*   **确定性 (Determinism)**：减少随机性，确保产出符合预期。
*   **高质量 (High Quality)**：内置最佳实践、代码规范与质量门禁。
*   **可解释性 (Explainability)**：每一步都有据可依，易于审查与回溯。

## 📦 Skills 列表 (Skill List)

### 1. Backend Workflow (后端标准化工作流)
*   **路径**: `backend-workflow/`
*   **简介**: 从详细设计文档到后端代码落地的全流程标准化方案。
*   **核心特性**:
    *   **⚖️ 任务分级 (Task Grading)**：根据任务规模（S/M/L）动态调整流程繁简度，拒绝杀鸡用牛刀。
    *   **🛡️ 质量门禁 (Quality Gates)**：内置 DDL 确认、API 契约确认、覆盖率审计等关键检查点。
    *   **🧪 测试驱动 (TDD)**：强制要求单元测试，追求 ≥85% 的核心业务覆盖率。
    *   **🔄 闭环反馈 (Feedback Loop)**：实现细节反哺设计文档，确保文档与代码的一致性。
*   **适用场景**: Java (Spring Boot) / Python 后端开发。

## 🚀 如何使用 (Usage)

这些 Skill 旨在被 AI Agent (如 Antigravity) 在执行特定任务时加载并遵循。

1.  **加载 Skill**: Agent 在识别到相关任务时，会主动读取对应目录下的 `SKILL.md`。
2.  **遵循指引**: Agent 将严格按照 Skill 中定义的步骤（Step-by-Step）执行，并在关键节点请求用户确认。
3.  **产出交付**: 最终生成符合 Skill 定义的标准交付物（代码、测试、文档）。

## 🤝 贡献 (Contribution)

欢迎沉淀更多的通用工作流至此仓库，扩充 AI 的技能树。
