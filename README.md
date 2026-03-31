# Claude Dev Skills

四个开发流程编排 skill，覆盖从模糊需求到代码交付的完整开发生命周期。用于 [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)。

## 为什么需要这些 skill？

Claude 擅长写代码，但容易跳步——直接下手写，跳过需求澄清、计划、验证。结果是：

- 需求没搞清就开始实现，白做了
- 没有测试保护就重构，悄悄改了行为
- 说完成但没运行验证，上线出 bug

这四个 skill 是**元流程**（meta-workflow）——它们不写代码，而是强制按正确顺序做事，确保 Claude 走完整流程、不跳步。

---

## 完整工作流

```
有一个想法或任务
        │
        ▼
需求清楚吗？
   │           │
  否           是
   │           │
   ▼           ▼
/dev-clarify  /dev-go ── 开发新功能（全流程）
   │                         │
   │                         ├── 遇到 bug ──→ /dev-fix
   └──────────→ /dev-go      │
                             └── 代码变臭 ─→ /dev-refactor
```

**四个入口，一条龙：**

1. **需求模糊** → `/dev-clarify` 先把需求问清楚，产出文档，再进 `/dev-go`
2. **需求清楚** → 直接 `/dev-go` 实现
3. **出了 bug** → `/dev-fix` 用 TDD 修复法定位根因、修复、验证
4. **代码变臭** → `/dev-refactor` 有安全网的小步重构，保证行为不变

---

## 四件套速查

| Skill | 触发时机 | 一句话 |
|-------|---------|--------|
| [`/dev-clarify`](skills/dev-clarify.md) | 需求模糊、想法不清晰 | PM 角色把模糊想法转化为可执行需求文档 |
| [`/dev-go`](skills/dev-go.md) | 需求清楚，要实现一个功能 | 从需求到交付的完整开发流程 |
| [`/dev-fix`](skills/dev-fix.md) | 发现 bug，需要修复 | TDD 修复法：先定根因，再 RED→GREEN→验证 |
| [`/dev-refactor`](skills/dev-refactor.md) | 代码结构变差，需要整理 | 有安全网的小步重构，外部行为不变 |

---

## `/dev-clarify` — 需求澄清

需求模糊时的入口。扮演产品经理角色，通过结构化对话把想法转化为 `/dev-go` 可直接执行的文档，不需要额外翻译。

**核心流程：**

| Phase | 做什么 |
|-------|--------|
| 层级诊断 | 判断需求在 L0（冲动）到 L4（可执行）的哪层，检测方案污染 |
| 方案分离 | 用户说"用 X 做 Y"时，先剥离方案，搞清楚要解决什么问题 |
| 六问收集 | WHO / AS-IS / PAIN / WHY / DONE / LIMIT，逐层引导 |
| 三重校验 | 可行性 + 矛盾检测 + 需求重构 |
| 技术方案推介 | 推介 2-3 个方案，用户选定，写入文档 |
| 文档产出 | 输出结构化需求文档 → 移交 `/dev-go` |

**输出文档包含：** 一句话需求 / 六维度需求画像 / 选定技术方案 / 预期产物

---

## `/dev-go` — 新功能开发

收到清晰需求后的完整实现流程。是四个 skill 中最重的，串联 8 个下游 skill。

| Phase | 做什么 |
|-------|--------|
| Context Sync | `git status` + 重读 CLAUDE.md，确认工作区干净 |
| Research | 读代码和文档，理解现有架构 |
| Plan | 产出实施计划，存 `docs/plans/` |
| Worktree | 3+ 文件或需要 subagent 时，创建隔离 worktree |
| Execute | subagent 逐任务并行执行 |
| Diff Review | 主窗口审查 `git diff`，检查高风险变更点 |
| Codex Review | 检测到 Codex 时必须执行，未安装则跳过（见下方说明） |
| Verify | 应用启动 + 关键路径运行时验证 |
| Knowledge Sync | 更新 CLAUDE.md / rules/ / docs/INDEX.md |
| Handback | code review + 交接报告 |

---

## `/dev-fix` — Bug 修复

TDD 修复法：先证明 bug 存在，再写最小修复，禁止没有根因分析就动手。

| Phase | 做什么 |
|-------|--------|
| Root Cause | 收集证据、复现、定位根因（**铁律：没根因不进下一步**） |
| Test First | 写一个会失败的测试证明 bug 存在（RED） |
| Minimal Fix | 最小修复让测试通过（GREEN），不做额外重构 |
| Diff Review | 审查修复的 `git diff`，检查是否引入回归 |
| Codex Review | 检测到 Codex 时必须执行，未安装则跳过（见下方说明） |
| Verify | 全量测试 + 修复路径 + 相邻路径（防止修 A 坏 B） |
| Knowledge Sync | 发现新陷阱 → 写入 rules/ 防止再犯 |
| Commit | 安全检查 + 提交 |

---

## `/dev-refactor` — 代码重构

**前提：必须有测试保护。** 每步只做一件事，每步之后跑测试，失败立即回退。

| Phase | 做什么 |
|-------|--------|
| 现状分析 | 识别坏味道：大文件、重复代码、过度耦合 |
| 安全网 | 检查测试覆盖率，不足先补测试 |
| Plan | 拆成小步骤，每步粒度要小到能单步 `git revert` |
| Worktree | **必须隔离**，重构失败可整个丢弃 |
| Execute | 逐步执行，每步后跑测试，红了立即回退 |
| Diff Review | 审查 `git diff`，确认只有结构变化、无行为变化 |
| Codex Review | 检测到 Codex 时必须执行，用对抗性审查质疑重构决策（见下方说明） |
| Verify | 全量测试 + 应用启动 + 关键路径验证 |
| Commit | 提交 |

---

## 共享设计原则

所有四个 skill 共同遵守：

1. **不自动 commit** — 等用户确认再提交
2. **Context 优先** — 每次开工前同步 `git status` + 项目记忆，不在脏状态上开工
3. **运行时验证 > 编译通过** — 必须实际跑起来，不只是语法正确
4. **Diff Review** — subagent 返回后必须审查 diff，不盲信
5. **Knowledge Sync** — 完成后更新项目知识体系，经验沉淀不丢失

---

## 前置依赖

| 依赖 | 说明 | 必需 |
|------|------|:----:|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) | Anthropic 官方 CLI | ✅ |
| [Claude Code Superpowers](https://github.com/anthropics/claude-code-superpowers) | 提供 9 个下游 skill（brainstorming、TDD、subagent 等） | ✅ |
| `commit` skill | 内置或自定义的 `/commit` 命令 | ✅ |
| [Codex Plugin](https://github.com/openai/codex-plugin-cc) | OpenAI Codex 独立代码审查（安装后 Diff Review 阶段自动启用） | 可选 |

---

## 安装

```bash
# 全局安装（所有项目可用）
cp skills/*.md ~/.claude/commands/

# 项目级安装（仅当前项目）
cp skills/*.md .claude/commands/
```

安装后在 Claude Code 中直接输入 `/dev-clarify`、`/dev-go`、`/dev-fix`、`/dev-refactor` 调用。

---

### Codex 审查（可选）

安装 [Codex 插件](https://github.com/openai/codex-plugin-cc)后，dev-go / dev-fix / dev-refactor 的 Diff Review 阶段会自动增加 Codex 独立审查步骤。

**自动检测逻辑**：流程启动时 `which codex` 检测本机环境——有 Codex CLI 且已安装插件则必须执行；有 CLI 没插件会提示安装；没有 CLI 则静默跳过。

需要：ChatGPT 订阅（含免费版）或 OpenAI API Key + Node.js 18.18+。

```bash
# 1. 安装 Codex CLI
npm install -g @openai/codex

# 2. 登录
codex login

# 3. 安装 Claude Code 插件
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/reload-plugins

# 4. 验证
/codex:setup
```

## 适用场景

语言和框架无关，适用于任何使用 Claude Code 的项目。假设项目有：

- Git 版本控制
- `CLAUDE.md` 项目说明文件（[三层知识架构](https://github.com/anthropics/claude-code-superpowers)）

---

## License

MIT
