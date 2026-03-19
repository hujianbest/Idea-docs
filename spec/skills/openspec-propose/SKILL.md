---
name: openspec-propose
description: 提出新变更，一步生成所有产物。当用户想要快速描述他们想要构建的内容并获得完整的提案、设计、规范和任务准备实施时使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.2.0"
---

提出新变更 - 一步创建变更并生成所有产物。

我将创建一个包含产物的变更：
- proposal.md（什么和为什么）
- design.md（如何）
- tasks.md（实施步骤）

准备好实施时，运行 /opsx-apply

---

**输入**：用户的请求应包含变更名称（kebab-case）或他们想要构建的内容的描述。

**步骤**

1. **如果未提供清晰的输入，询问他们想要构建什么**

   使用 **AskUserQuestion 工具**（开放式，无预设选项）询问：
   > "您想要处理什么变更？描述您想要构建或修复的内容。"

   从他们的描述中，派生一个 kebab-case 名称（例如，"添加用户认证" → `add-user-auth`）。

   **重要**：在了解用户想要构建什么之前，不要继续。

2. **创建变更目录**
   ```bash
   openspec new change "<name>"
   ```
   这会在 `openspec/changes/<name>/` 创建一个带有 `.openspec.yaml` 的脚手架变更。

3. **获取产物构建顺序**
   ```bash
   openspec status --change "<name>" --json
   ```
   解析 JSON 以获取：
   - `applyRequires`：实施前所需的产物 ID 数组（例如 `["tasks"]`）
   - `artifacts`：所有产物及其状态和依赖项的列表

4. **按顺序创建产物直到准备应用**

   使用 **TodoWrite 工具** 跟踪产物的进度。

   按依赖顺序循环遍历产物（首先是没有待处理依赖项的产物）：

   a. **对于每个 `ready`（依赖项已满足）的产物**：
      - 获取指令：
        ```bash
        openspec instructions <artifact-id> --change "<name>" --json
        ```
      - 指令 JSON 包括：
        - `context`：项目背景（对您的约束 - 不要包含在输出中）
        - `rules`：产物特定规则（对您的约束 - 不要包含在输出中）
        - `template`：输出文件使用的结构
        - `instruction`：此产物类型的架构特定指导
        - `outputPath`：写入产物的位置
        - `dependencies`：用于上下文的已完成产物
      - 读取任何已完成依赖项文件以获取上下文
      - 使用 `template` 作为结构创建产物文件
      - 应用 `context` 和 `rules` 作为约束 - 但不要将它们复制到文件中
      - 显示简短进度："已创建 <artifact-id>"

   b. **继续直到所有 `applyRequires` 产物完成**
      - 创建每个产物后，重新运行 `openspec status --change "<name>" --json`
      - 检查 `applyRequires` 中的每个产物 ID 在产物数组中是否具有 `status: "done"`
      - 当所有 `applyRequires` 产物完成时停止

   c. **如果产物需要用户输入**（上下文不明确）：
      - 使用 **AskUserQuestion 工具** 澄清
      - 然后继续创建

5. **显示最终状态**
   ```bash
   openspec status --change "<name>"
   ```

**输出**

完成所有产物后，总结：
- 变更名称和位置
- 创建的产物列表及简短描述
- 准备情况："所有产物已创建！准备好实施。"
- 提示："运行 `/opsx-apply` 或让我开始实施任务。"

**产物创建指南**

- 遵循 `openspec instructions` 中每个产物类型的 `instruction` 字段
- 架构定义每个产物应包含的内容 - 遵循它
- 在创建新产物之前读取依赖产物以获取上下文
- 使用 `template` 作为输出文件的结构 - 填充其部分
- **重要**：`context` 和 `rules` 是对您的约束，而不是文件的内容
  - 不要将 `<context>`、`<rules>`、`<project_context>` 块复制到产物中
  - 这些指导您编写的内容，但绝不应出现在输出中

**防护措施**
- 创建实施所需的所有产物（由架构的 `apply.requires` 定义）
- 在创建新产物之前始终读取依赖产物
- 如果上下文严重不明确，询问用户 - 但倾向于做出合理决策以保持势头
- 如果该名称的变更已存在，询问用户是否要继续它或创建新变更
- 在继续下一个产物之前验证每个产物文件是否在写入后存在
