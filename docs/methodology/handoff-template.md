# Handoff Template

> 用途：长任务 agent 在切 session、交接、进入 blocked / waiting_user / review 节点时使用。

```markdown
# Handoff Report

- Task ID:
- Agent:
- From State:
- To State:
- Recorded At:

## Completed
- 使用 `||` 或项目符号列出已完成内容（至少 1 条）

## Remaining
- 当前还没做完的内容（必填）

## Risks
- 已知风险 / 不确定性（必填）

## Next Steps
- 接手者第一步该做什么（必填）

## Artifact Ref
- 使用可解析格式之一：
  - `git:<7位以上commit>`
  - `plan:v<N>`
  - `file:<路径>`
  - `report:<标识符>`

## Optional
- Checkpoint Path:
- Notes:
```

## 质量标准

handoff 不是日志，而是接力棒。至少要让下一位 agent 只读本文件和最新 checkpoint，就知道：
1. 你做完了什么
2. 还剩什么
3. 风险在哪
4. 下一步从哪里开始
5. 该去看哪个文件，而不是重读全量原文
