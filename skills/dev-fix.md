---
description: 调试修复流程编排：定位根因 → 写测试 → 最小修复 → 验证 → 审查 → 提交
---

# /dev-fix — 调试修复流程编排

收到 bug 报告或发现异常后按以下顺序执行，不跳过阶段。

## 流程

| Phase | 做什么 | Skill |
|-------|--------|-------|
| 0. Context Sync | `git status` + `git log -5`，重读 CLAUDE.md / MEMORY.md。确认当前分支状态 | — |
| 1. Root Cause | 收集证据，复现问题，定位根因。**禁止未定位根因就动手修** | `superpowers:systematic-debugging` |
| 2. Test First | 针对根因写失败测试，确认测试确实失败（RED） | `superpowers:test-driven-development` |
| 3. Minimal Fix | 写最小修复代码让测试通过（GREEN），不做额外重构 | `superpowers:test-driven-development` |
| 3b. Diff Review | 审查修复的 `git diff`：新增常量是否与现有约定一致、是否引入回归 | — |
| 3c. Codex Review | Codex 可用时**必须执行**，不可用则跳过（见下方 Codex 检测规则）。`/codex:review` 独立审查修复代码，涉及核心逻辑时用 `/codex:adversarial-review` | `codex:review` |
| 4. Verify | 全量测试 + **运行时验证修复路径和相邻路径**（防止修 A 坏 B） | `superpowers:verification-before-completion` |
| 5. Knowledge Sync | 修复揭示了新约束/陷阱？→ 写入 rules/ 或 CLAUDE.md，防止再犯 | — |
| 6. Review | 修复涉及 3+ 文件或核心逻辑时，派 code-reviewer 审查 | `superpowers:requesting-code-review` |
| 7. Commit | 安全检查 + 提交 | `commit` |

## 规则

- **不自动 commit**，等用户确认
- Phase 1 是铁律：没有根因分析，不允许进入 Phase 2
- 简单 bug（单文件、明显原因）可以压缩 Phase 1-3 为一步，但必须过 Phase 4
- Phase 4 必须验证**修复路径 + 相邻路径**：修了一个模块 → 同时验证其他模块没坏
- 只在以下情况停下来问：根因不明需要更多信息 / 修复方案有多个方向 / 修复影响范围超预期

## Codex 检测规则（Phase 0 执行，Phase 3c 使用）

Phase 0 时运行 `which codex` 检测 Codex CLI：

1. **有 CLI + 有 `codex:review` skill** → Phase 3c 必须执行
2. **有 CLI + 没有插件** → 提示用户："检测到本机已安装 Codex CLI，是否安装 Claude Code 插件以启用独立代码审查？"并给出安装步骤。用户同意则安装后执行 Phase 3c，拒绝则跳过
3. **没有 CLI** → 跳过 Phase 3c
