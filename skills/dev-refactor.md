---
description: 重构流程编排：现状分析 → 安全网（补测试）→ 小步重构 → 验证行为不变 → 审查 → 提交
---

# /dev-refactor — 重构流程编排

改善代码结构，不改变外部行为。每一步都必须可验证、可回退。

## 流程

| Phase | 做什么 | Skill |
|-------|--------|-------|
| 0. Context Sync | `git status` + `git log -5`，确认无未提交变更。有脏状态 → 先 commit 或 stash | — |
| 1. 现状分析 | 识别代码坏味道：大文件（>300行）、重复代码、过度耦合、职责混乱。产出问题清单 | — |
| 2. 安全网 | 检查测试覆盖率。覆盖不足 → 先补测试再重构。**没有测试保护的代码不允许重构** | `superpowers:test-driven-development` |
| 3. Plan | 拆成小步骤，每步只做一件事（提取函数/移动文件/重命名/拆分模块）。存 `docs/plans/` | `superpowers:writing-plans` |
| 3b. Worktree | 必须隔离，重构出问题时可以整个丢弃 | `superpowers:using-git-worktrees` |
| 4. Execute | 逐步执行，**每步之后跑测试**。测试红 → 立即回退该步，不要继续 | `superpowers:subagent-driven-development` |
| 4b. Diff Review | 审查 `git diff`：只有结构变化、无行为变化、无遗漏的依赖关系 | — |
| 4c. Codex Review | Codex 可用时**必须执行**，不可用则跳过（见下方 Codex 检测规则）。`/codex:adversarial-review` 质疑重构决策和替代方案 | `codex:adversarial-review` |
| 5. Verify | 全量测试 + 应用启动 + 关键路径运行时验证 | `superpowers:verification-before-completion` |
| 6. Knowledge Sync | 架构变了 → 更新 CLAUDE.md；模块规则变了 → 更新 rules/；plan 完成 → 更新 INDEX.md | — |
| 7. Review | 派 code-reviewer 审查重构质量 | `superpowers:requesting-code-review` |
| 8. Commit | 提交 | `commit` |

## 规则

- **不自动 commit**，等用户确认
- **行为不变是铁律**：重构过程中如果发现 bug，记录下来，但不在重构分支里修——另开 /dev-fix 流程
- 每步重构粒度要小：能 `git revert` 单步回退
- 只在以下情况停下来问：测试覆盖不足需要决定补哪些 / 重构方向有多个选择 / 发现改动范围超预期

## Codex 检测规则（Phase 0 执行，Phase 4c 使用）

Phase 0 时运行 `which codex` 检测 Codex CLI：

1. **有 CLI + 有 `codex:adversarial-review` skill** → Phase 4c 必须执行
2. **有 CLI + 没有插件** → 提示用户："检测到本机已安装 Codex CLI，是否安装 Claude Code 插件以启用独立代码审查？"并给出安装步骤。用户同意则安装后执行 Phase 4c，拒绝则跳过
3. **没有 CLI** → 跳过 Phase 4c

## Subagent 指令规则

重构类 subagent 的 prompt 必须包含：
1. **原始代码的完整依赖关系清单**（拆分时最容易丢依赖）
2. **公共 API 契约**（哪些属性/方法被外部调用，重构后必须保持）
3. **运行时验证要求**（编译通过 + 测试通过 + 实例化测试，不能只编译通过就算完）

## 常见重构模式

| 坏味道 | 重构手法 |
|--------|---------|
| 大文件（>300行） | 按职责拆分模块 |
| 重复代码 | 提取共享函数/组件 |
| 深层嵌套 | 提前 return / 提取子函数 |
| 数据泥团 | 提取数据类/接口 |
| 过长参数列表 | 引入参数对象 |
| 散弹式修改 | 内聚到同一模块 |
