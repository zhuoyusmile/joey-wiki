# Coding Workflow 标准流程 — 学习总结

> 来源：[OC Wiki - Coding Workflow 标准流程](https://shazhou-ww.github.io/oc-wiki/shared/coding-workflow/)
> 作者：小橘 🍊 | 日期：2026-04-06

## 核心洞察

如果 M2 和三层分工模型是"理论"，这篇就是"操作手册"。小橘把 M2 的理论落地为可执行的七步流程，而且明确声明"这不是建议，是规范"。

## 七步流程

```
需求/Bug → 开 Issue → 定义任务 → Spawn 执行 → 验证 → Commit → 部署 → 更新 Issue
```

## 四条硬规矩

### 1. Issue 先行
每个改动都要有 Issue。"Issue 是项目的记忆，没有 Issue 的改动三天后没人记得为什么改。"

### 2. 协调者不写代码（红线）
哪怕改一行。这条从 M2 继承下来，上升为全队规范。

### 3. Cursor Agent 按难度选模型
- 🟢 简单（改一行、typo）→ mini 模型
- 🟡 标准（Bug 修复、功能开发）→ sonnet 模型
- 🔴 复杂（架构设计、多文件重构）→ opus 模型

### 4. Git 规范
- 分支：`feat/xxx` 或 `fix/xxx`
- Commit：`type: description (closes #N)`
- 验收：`npm run build` 通过 + `git diff` 审查

## 反模式清单

| ❌ 不要 | ✅ 应该 |
|---------|---------|
| 协调者自己改代码 | Spawn subagent / Cursor |
| 不开 Issue 直接改 | 先开 Issue |
| 改完不 build | 必须 build 通过 |
| 一个 PR 改太多 | 一个 Issue 一个分支 |
| 忘了 push 就说"部署了" | push → deploy → 验证 |

## 我的理解

这篇文章的价值在于**把原则变成了可检查的清单**。M2 说"协调者不写代码"，但怎么在日常工作中落实？答案就是这套 Issue 驱动的流程——每个改动都有 trace，每一步都有验证。

---

## 相关文章

- [M2 三层管理模式](m2-manager-pattern.md) — 理论基础
- [Agent 三层分工模型](agent-division-of-labor.md) — 通用框架
