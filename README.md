# Claude Dev Skills

四个开发流程编排 skill，用于 [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)。

它们是**元流程**（meta-workflow）——不做具体编码，而是编排其他 skill 的执行顺序，确保走完整流程、不跳步。

## 总览

| Skill | 用途 | 一句话 |
|-------|------|--------|
| [`/dev-clarify`](skills/dev-clarify.md) | **需求澄清** | 需求模糊时先跑，产出 dev-go 可直接执行的文档 |
| [`/dev-go`](skills/dev-go.md) | 新功能开发 | 从需求到交付的完整流程 |
| [`/dev-fix`](skills/dev-fix.md) | Bug 修复 | TDD 修复法：根因 → RED → GREEN → 验证 |
| [`/dev-refactor`](skills/dev-refactor.md) | 代码重构 | 有安全网的小步重构，行为不变 |

```
需求模糊？
    │
    ↓
/dev-clarify ─── 澄清需求，产出文档
    │
    ↓
/dev-go      ─── 开发新功能（全流程）
    │
    ├── 遇到 bug → /dev-fix（修完回来继续）
    │
    └── 代码臭了 → /dev-refactor（另开分支，改完合回来）
```

---

## `/dev-clarify` — 需求澄清流程

需求模糊时的入口。扮演产品经理角色，通过结构化对话把模糊想法转化为 dev-go 可直接执行的需求文档。

| Phase | 这一步做什么 |
|-------|-------------|
| **0. 层级诊断** | 判断需求在 L0（冲动）到 L4（可执行）的哪层，检测方案污染 |
| **1. 方案分离** | 检测到"用 X 做 Y"时，剥离方案，先搞清楚问题本身 |
| **2. 逐层下降** | 一次一问，引导用户下降到 L3-L4；答不出给选项 |
| **3. 六问验证** | 收集 WHO / AS-IS / PAIN / WHY / DONE / LIMIT |
| **4. 三重校验** | 可行性、矛盾检测、需求重构 |
| **5. 技术方案推介** | 推介 2-3 个方案，用户选定，写入文档 |
| **6. 文档产出 → dev-go** | 输出结构化需求文档，直接移交 dev-go 执行 |

---

## `/dev-go` — 新功能开发流程

从零实现一个功能，覆盖从理解需求到代码交付的全部环节。是四个 skill 中最重的，串联了 8 个下游 skill。

| Phase | 这一步做什么 | 调用的 Skill | Skill 做什么 |
|-------|-------------|-------------|-------------|
| **0. Context Sync** | `git status` + `git log -5`，重读 CLAUDE.md / MEMORY.md，确认工作区干净。发现异常 → 停下问用户 | — | — |
| **1. Research** | 读相关代码和文档，理解现有架构。需求模糊时引导用户澄清 | `superpowers:brainstorming` | 用结构化提问帮用户把模糊想法变成明确需求 |
| **2. Plan** | 产出实施计划，存到 `docs/plans/YYYY-MM-DD-<topic>.md` | `superpowers:writing-plans` | 生成带步骤、风险点、验收标准的结构化计划文档 |
| **2b. Worktree** | 改动涉及 3+ 文件或需要 subagent 时，创建隔离 worktree | `superpowers:using-git-worktrees` | 在 `.claude/worktrees/` 创建独立工作副本，失败可整个丢弃 |
| **3. Execute** | 按计划逐任务交给 subagent 执行 | `superpowers:subagent-driven-development` | 用 subagent（独立上下文）并行执行子任务，保护主窗口上下文 |
| **3b. Diff Review** | 主窗口审查 `git diff`，检查 5 类高风险点（常量映射/返回值/条件逻辑/import/回归） | — | — |
| **4. Verify** | 应用启动无崩溃 + 关键路径运行时验证 | `superpowers:verification-before-completion` | 强制运行时验证，不允许"编译过了就完事" |
| **4. Verify** *(遇到 bug 时)* | 验证阶段发现 bug → 切换到修复流程 | `superpowers:systematic-debugging` | 系统化收集证据、缩小范围、定位根因 |
| **4b. E2E** | 涉及用户交互时，手动或脚本模拟完整操作路径 | — | — |
| **5. Knowledge Sync** | 更新项目知识体系（L1 CLAUDE.md / L2 rules/ / L3 docs/INDEX.md） | — | — |
| **6. Handback** | 重要改动先过 code review，产出交接报告（改了什么/验证结果/需测试的路径） | `superpowers:requesting-code-review` | 派独立 code-reviewer agent 审查代码质量 |
| **6. Handback** *(分支收尾)* | 分支合并、清理临时文件 | `superpowers:finishing-a-development-branch` | 处理分支合并策略、清理 worktree、更新状态 |

---

## `/dev-fix` — Bug 修复流程

收到 bug 报告后的标准修复流程。核心是 TDD 修复法：先证明 bug 存在（RED），再写最小修复（GREEN）。串联 5 个下游 skill。

| Phase | 这一步做什么 | 调用的 Skill | Skill 做什么 |
|-------|-------------|-------------|-------------|
| **0. Context Sync** | `git status` + `git log -5`，确认当前分支状态 | — | — |
| **1. Root Cause** | 收集证据、复现问题、定位根因。**铁律：没找到根因不许进下一步** | `superpowers:systematic-debugging` | 引导系统化调试：日志分析 → 假设 → 最小复现 → 根因确认 |
| **2. Test First** | 针对根因写一个会失败的测试（RED），确认测试确实失败 | `superpowers:test-driven-development` | TDD 红绿循环：先写失败测试，再写让测试通过的代码 |
| **3. Minimal Fix** | 写最小修复代码让测试通过（GREEN），**不做额外重构** | `superpowers:test-driven-development` | 同上，确保只写刚好让测试绿的代码 |
| **3b. Diff Review** | 审查 `git diff`：新增常量是否与现有约定一致、是否引入回归 | — | — |
| **4. Verify** | 全量测试 + **运行时验证修复路径和相邻路径**（修了 A 模块 → 验证 B 模块没坏） | `superpowers:verification-before-completion` | 强制运行时验证，防止"修 A 坏 B" |
| **5. Knowledge Sync** | 修复揭示了新陷阱？→ 写入 rules/ 或 CLAUDE.md，防止再犯 | — | — |
| **6. Review** | 修复涉及 3+ 文件或核心逻辑时，派 code-reviewer 审查 | `superpowers:requesting-code-review` | 派独立 code-reviewer agent 审查修复质量 |
| **7. Commit** | 安全检查 + 提交 | `commit` | 检查敏感文件、生成语义化 commit message、提交 |

---

## `/dev-refactor` — 代码重构流程

改善代码结构，**不改变外部行为**。每一步必须可验证、可回退。串联 6 个下游 skill。

| Phase | 这一步做什么 | 调用的 Skill | Skill 做什么 |
|-------|-------------|-------------|-------------|
| **0. Context Sync** | `git status` + `git log -5`，确认无未提交变更。有脏状态 → 先 commit 或 stash | — | — |
| **1. 现状分析** | 识别代码坏味道：大文件（>300行）、重复代码、过度耦合、职责混乱。产出问题清单 | — | — |
| **2. 安全网** | 检查测试覆盖率。覆盖不足 → **先补测试再重构**。没有测试保护的代码不允许重构 | `superpowers:test-driven-development` | 补充缺失的测试用例，建立重构的安全网 |
| **3. Plan** | 拆成小步骤，每步只做一件事（提取函数/移动文件/重命名/拆分模块） | `superpowers:writing-plans` | 生成结构化重构计划，每步粒度要小到能单步 `git revert` |
| **3b. Worktree** | **必须隔离**，重构出问题时可以整个丢弃 | `superpowers:using-git-worktrees` | 创建独立工作副本，重构失败可零成本回退 |
| **4. Execute** | 逐步执行，**每步之后跑测试**。测试红 → 立即回退该步，不继续 | `superpowers:subagent-driven-development` | 用 subagent 执行每个重构步骤，每步后自动验证 |
| **4b. Diff Review** | 审查 `git diff`：确认只有结构变化、无行为变化、无遗漏的依赖关系 | — | — |
| **5. Verify** | 全量测试 + 应用启动 + 关键路径运行时验证 | `superpowers:verification-before-completion` | 强制运行时验证，确保重构没有偷偷改变行为 |
| **6. Knowledge Sync** | 架构变了 → 更新 CLAUDE.md；模块规则变了 → 更新 rules/；plan 完成 → 更新 INDEX.md | — | — |
| **7. Review** | 派 code-reviewer 审查重构质量 | `superpowers:requesting-code-review` | 审查重构是否保持了公共 API 契约、是否引入不必要的复杂度 |
| **8. Commit** | 提交 | `commit` | 安全检查 + 语义化 commit message |

---

## 下游 Skill 速查

所有被引用的下游 skill 汇总：

| Skill | 来源 | 一句话职责 | 被调用次数 |
|-------|------|-----------|:---------:|
| `superpowers:verification-before-completion` | [superpowers] | 运行时验证，防止"编译过了就完事" | 3 |
| `superpowers:requesting-code-review` | [superpowers] | 派 code-reviewer agent 审查变更 | 3 |
| `superpowers:test-driven-development` | [superpowers] | TDD 红绿循环 | 3 |
| `superpowers:subagent-driven-development` | [superpowers] | 用 subagent 并行执行子任务 | 2 |
| `superpowers:writing-plans` | [superpowers] | 生成结构化实施计划 | 2 |
| `superpowers:using-git-worktrees` | [superpowers] | 创建隔离 worktree | 2 |
| `superpowers:systematic-debugging` | [superpowers] | 系统化定位 bug 根因 | 2 |
| `superpowers:brainstorming` | [superpowers] | 引导需求探索和澄清 | 1 |
| `superpowers:finishing-a-development-branch` | [superpowers] | 分支收尾（合并/清理） | 1 |
| `commit` | 内置 / 自定义 | 安全检查 + git commit | 2 |

[superpowers]: https://github.com/anthropics/claude-code-superpowers

---

## 共享设计原则

1. **不自动 commit** — 等用户确认
2. **Phase 0 同步上下文** — git status + 项目记忆，在脏状态上不开工
3. **Knowledge Sync** — 每次完成都更新项目知识体系（L1/L2/L3 三层）
4. **Diff Review** — subagent 返回后必须人工审查 diff
5. **运行时验证 > 编译通过** — 必须实际跑起来，不只是语法正确

## 前置依赖

| 依赖 | 说明 |
|------|------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) | Anthropic 官方 CLI |
| [Claude Code Superpowers](https://github.com/anthropics/claude-code-superpowers) | 提供 9 个下游 skill |
| `commit` skill | Claude Code 内置或自定义的 `/commit` 命令 |

## 安装

```bash
# 全局安装（所有项目可用）
cp skills/*.md ~/.claude/commands/

# 项目级安装（仅当前项目）
cp skills/*.md .claude/commands/
```

安装后在 Claude Code 中用 `/dev-go`、`/dev-fix`、`/dev-refactor` 调用。

## 适用场景

这些 skill 设计为**语言和框架无关**，适用于任何使用 Claude Code 的项目。它们假设你的项目有：

- Git 版本控制
- `CLAUDE.md` 项目说明文件
- `docs/` 文档目录（可选）

## License

MIT
