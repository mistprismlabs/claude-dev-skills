# Claude Dev Skills

三个开发流程编排 skill，用于 [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)。

它们是**元流程**（meta-workflow）——不做具体编码，而是编排其他 skill 的执行顺序，确保走完整流程、不跳步。

## Skills

| Skill | 用途 | 核心理念 |
|-------|------|---------|
| [`/dev-go`](skills/dev-go.md) | 完整开发流程 | context sync → research → plan → execute → verify → knowledge sync → handback |
| [`/dev-fix`](skills/dev-fix.md) | Bug 修复 | 根因定位 → 写失败测试（RED）→ 最小修复（GREEN）→ 验证修复路径 + 相邻路径 |
| [`/dev-refactor`](skills/dev-refactor.md) | 代码重构 | 安全网（补测试）→ 小步重构 → 每步跑测试 → 行为不变 |

## 三者关系

```
/dev-go      ─── 建新功能（全流程）
    │
    ├── 遇到 bug → /dev-fix（修完回来继续）
    │
    └── 代码臭了 → /dev-refactor（另开分支，改完合回来）
```

## 依赖关系

这三个 skill 是**编排层**，通过串联下游 skill 来工作。下表列出所有依赖：

### 下游 Skill 清单

| 下游 Skill | 来源 | 职责 | 被谁调用 |
|------------|------|------|---------|
| `superpowers:brainstorming` | [superpowers](https://github.com/anthropics/claude-code-superpowers) | 需求模糊时引导探索和澄清 | `/dev-go` Phase 1 |
| `superpowers:writing-plans` | superpowers | 产出结构化实施计划 | `/dev-go` Phase 2, `/dev-refactor` Phase 3 |
| `superpowers:using-git-worktrees` | superpowers | 创建隔离 worktree 保护主分支 | `/dev-go` Phase 2b, `/dev-refactor` Phase 3b |
| `superpowers:subagent-driven-development` | superpowers | 用 subagent 并行执行子任务 | `/dev-go` Phase 3, `/dev-refactor` Phase 4 |
| `superpowers:systematic-debugging` | superpowers | 系统化定位 bug 根因 | `/dev-go` Phase 4, `/dev-fix` Phase 1 |
| `superpowers:test-driven-development` | superpowers | TDD 红绿循环 | `/dev-fix` Phase 2-3, `/dev-refactor` Phase 2 |
| `superpowers:verification-before-completion` | superpowers | 运行时验证，防止"编译过了就完事" | `/dev-go` Phase 4, `/dev-fix` Phase 4, `/dev-refactor` Phase 5 |
| `superpowers:requesting-code-review` | superpowers | 派 code-reviewer 审查变更 | `/dev-go` Phase 6, `/dev-fix` Phase 6, `/dev-refactor` Phase 7 |
| `superpowers:finishing-a-development-branch` | superpowers | 分支收尾（合并/清理） | `/dev-go` Phase 6 |
| `commit` | Claude Code 内置 / 自定义 | 安全检查 + git commit | `/dev-fix` Phase 7, `/dev-refactor` Phase 8 |

### 调用矩阵

哪个 skill 调用了哪些下游：

| 下游 Skill | `/dev-go` | `/dev-fix` | `/dev-refactor` |
|------------|:---------:|:----------:|:---------------:|
| brainstorming | Phase 1 | — | — |
| writing-plans | Phase 2 | — | Phase 3 |
| using-git-worktrees | Phase 2b | — | Phase 3b |
| subagent-driven-development | Phase 3 | — | Phase 4 |
| systematic-debugging | Phase 4 | Phase 1 | — |
| test-driven-development | — | Phase 2-3 | Phase 2 |
| verification-before-completion | Phase 4 | Phase 4 | Phase 5 |
| requesting-code-review | Phase 6 | Phase 6 | Phase 7 |
| finishing-a-development-branch | Phase 6 | — | — |
| commit | — | Phase 7 | Phase 8 |

### 互相调用

三个 dev skill 之间也存在调用关系：

- `/dev-go` Phase 4 遇到 bug → 调用 `/dev-fix`
- `/dev-refactor` 发现 bug → 记录，另开 `/dev-fix`（不在重构分支修）

## 共享设计原则

1. **不自动 commit**，等用户确认
2. **Phase 0 同步上下文**（git status + 项目记忆）
3. **Knowledge Sync**（每次完成都更新项目知识体系）
4. **Diff Review** 在 subagent 返回后必做
5. **运行时验证** > 编译通过

## 前置依赖

使用这些 skill 前需要安装：

1. **[Claude Code Superpowers](https://github.com/anthropics/claude-code-superpowers)** — 提供 9 个下游 skill
   ```bash
   # 按 superpowers 项目的安装说明操作
   ```

2. **commit skill** — Claude Code 内置或自定义的 `/commit` 命令

## 安装

复制 `skills/` 下的 `.md` 文件到你的 Claude Code 命令目录：

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
