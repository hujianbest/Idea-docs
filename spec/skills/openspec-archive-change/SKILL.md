---
name: openspec-archive-change
description: 在实验性工作流程中归档已完成的变更。当用户想要在实施完成后最终确定并归档变更时使用。
license: MIT
compatibility: 需要 openspec CLI。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.2.0"
---

在实验性工作流程中归档已完成的变更。

**输入**：可选指定变更名称。如果省略，检查是否可以从对话上下文中推断。如果模糊或不明确，必须提示可用的变更。

**步骤**

1. **如果未提供变更名称，提示选择**

   运行 `openspec list --json` 获取可用变更。使用 **AskUserQuestion 工具** 让用户选择。

   仅显示活动变更（尚未归档）。
   如果可用，包含每个变更使用的架构。

   **重要**：不要猜测或自动选择变更。始终让用户选择。

2. **检查产物完成状态**

   运行 `openspec status --change "<name>" --json` 检查产物完成情况。

   解析 JSON 以了解：
   - `schemaName`：正在使用的工作流程
   - `artifacts`：产物列表及其状态（`done` 或其他）

   **如果有任何产物不是 `done`：**
   - 显示警告列出未完成的产物
   - 使用 **AskUserQuestion 工具** 确认用户想要继续
   - 用户确认后继续

3. **检查任务完成状态**

   读取任务文件（通常是 `tasks.md`）以检查未完成的任务。

   计算标记为 `- [ ]`（未完成）与 `- [x]`（已完成）的任务。

   **如果发现未完成的任务：**
   - 显示警告显示未完成任务的数量
   - 使用 **AskUserQuestion 工具** 确认用户想要继续
   - 用户确认后继续

   **如果任务文件不存在**：继续，不显示任务相关警告。

4. **评估增量规范同步状态**

   在 `openspec/changes/<name>/specs/` 检查增量规范。如果不存在，继续不同步提示。

   **如果增量规范存在：**
   - 将每个增量规范与其对应的主规范在 `openspec/specs/<capability>/spec.md` 进行比较
   - 确定将应用什么更改（添加、修改、删除、重命名）
   - 在提示前显示组合摘要

   **提示选项：**
   - 如果需要更改："立即同步（推荐）"、"归档但不同步"
   - 如果已同步："立即归档"、"仍然同步"、"取消"

   如果用户选择同步，使用 Task 工具（subagent_type: "general-purpose", prompt: "使用 Skill 工具调用 openspec-sync-specs 进行变更 '<name>'。增量规范分析：<包含分析的增量规范摘要>"）。无论选择什么，继续归档。

5. **执行归档**

   如果归档目录不存在则创建：
   ```bash
   mkdir -p openspec/changes/archive
   ```

   使用当前日期生成目标名称：`YYYY-MM-DD-<change-name>`

   **检查目标是否已存在：**
   - 如果是：失败并显示错误，建议重命名现有归档或使用不同日期
   - 如果否：将变更目录移动到归档

   ```bash
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

6. **显示摘要**

   显示归档完成摘要，包括：
   - 变更名称
   - 使用的架构
   - 归档位置
   - 是否同步了规范（如果适用）
   - 关于任何警告的说明（未完成的产物/任务）

**成功时的输出**

```
## 归档完成

**变更：** <change-name>
**架构：** <schema-name>
**归档到：** openspec/changes/archive/YYYY-MM-DD-<name>/
**规范：** ✓ 已同步到主规范（或"无增量规范"或"跳过同步"）

所有产物完成。所有任务完成。
```

**防护措施**
- 如果未提供，始终提示变更选择
- 使用产物图（openspec status --json）进行完成检查
- 不要因警告而阻止归档 - 只需通知并确认
- 移动到归档时保留 .openspec.yaml（它随目录移动）
- 显示清晰的摘要
- 如果请求同步，使用 openspec-sync-specs 方法（代理驱动）
- 如果增量规范存在，始终运行同步评估并在提示前显示组合摘要
