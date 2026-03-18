---
description: 完整开发流程编排：context check → research → plan → execute → verify → handback
---

# /dev-go — 开发全流程编排

收到任务后按以下顺序执行，不跳过阶段。

## 流程

| Phase | 做什么 | Skill |
|-------|--------|-------|
| 0. Context Sync | `git status` + `git log -5`，重读 CLAUDE.md / MEMORY.md。发现异常 → 告知用户，等确认 | — |
| 1. Research | 读相关代码和文档。需求模糊 → 提问澄清 | `superpowers:brainstorming` |
| 2. Plan | 产出实施计划，存 `docs/plans/YYYY-MM-DD-<topic>.md` | `superpowers:writing-plans` |
| 2b. Worktree | 3+ 文件或使用 subagent 时必须先建 | `superpowers:using-git-worktrees` |
| 3. Execute | subagent 逐任务执行（见下方 subagent 规则） | `superpowers:subagent-driven-development` |
| 3b. Diff Review | 主窗口审查 `git diff`，重点检查关键变更点（见下方规则） | — |
| 4. Verify | 应用无崩溃 + 关键路径运行时验证。有 bug → /dev-fix | `superpowers:verification-before-completion` + `superpowers:systematic-debugging` |
| 4b. E2E | 涉及用户交互流程时，手动或脚本模拟操作路径 | — |
| 5. Knowledge Sync | **更新知识体系**（见下方规则） | — |
| 6. Handback | 重要改动先过 code-reviewer，报告：改了什么 / 验证结果 / 需测试的路径 | `superpowers:requesting-code-review` + `superpowers:finishing-a-development-branch` |

## 规则

- **不自动 commit**，等用户确认
- 可自主使用 subagent / Agent Teams / 任意 skill
- 只在以下情况停下来问：阻断无法解决 / 重大方向分叉 / Phase 0 发现异常

## Subagent 指令规则（Phase 3）

给 subagent 的 prompt 必须包含：
1. **必读文件清单**（"先把这些文件全部读完再动手"）
2. **现有代码约定**（从 CLAUDE.md/rules 中提取关键约束告知 subagent）
3. **运行时验证要求**（不能只编译通过，必须包含实例化测试或 API 调用验证）
4. **禁止重复定义**（映射表、工具函数等只在一个文件定义，其他文件 import）

## Diff Review 规则（Phase 3b）

subagent 返回后，主窗口 `git diff` 检查以下高风险点：

- **新增常量/映射**：是否与现有约定一致
- **函数返回值**：新模块的 result 格式是否与现有格式兼容
- **组件显隐逻辑**：UI 组件的条件显示是否覆盖了新增场景
- **import 路径**：是否有重复定义（同一个函数/常量在两个文件各写一份）
- **现有代码影响**：改了共享函数后，原有模块的逻辑是否仍然正确

发现问题 → 当场修复后再进 Phase 4，不要带着问题往后走。

## 知识沉淀（Phase 5 必做）

任务完成后，对照三层架构逐层检查是否需要更新：

| 层 | 文件 | 什么时候需要更新 |
|----|------|----------------|
| L1 | `CLAUDE.md` | 架构变化、新模块、新约束、新命令 |
| L2 | `.claude/rules/*.md` | 模块级规则变化（新的设计决策、陷阱、约定） |
| L3 | `docs/INDEX.md` | 新增/完成 plan、TODO 状态变化 |

**判断标准**：如果下次新对话打开项目，CLAUDE.md 里的信息会导致误解或遗漏 → 必须更新。

## 文档维护

- 新建文档 → 放对应子目录（plans/design/reference/），同步在 `docs/INDEX.md` 加一行
- 临时调试记录、AI 提示词 → 不保存进 docs/
- plans/ 文件永久保留，不移动，靠日期前缀排序
