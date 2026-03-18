# Claude Dev Skills

三个开发流程编排 skill，用于 [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview)。

它们是**元流程**（meta-workflow）——不做具体编码，而是编排每次开发任务的执行顺序，确保走完整流程、不跳步。

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

## 共享设计原则

1. **不自动 commit**，等用户确认
2. **Phase 0 同步上下文**（git status + 项目记忆）
3. **Knowledge Sync**（每次完成都更新项目知识体系）
4. **Diff Review** 在 subagent 返回后必做
5. **运行时验证** > 编译通过

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
