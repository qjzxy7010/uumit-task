# tasks/ — 任务书目录

一个订单一个子目录：

```
tasks/<order_no>/
  brief.md       任务书（发包方写，承接方只读）
  spec.json      机器可读规格
  status.json    状态机
```

- `<order_no>` = UUMit 订单号，如 `ORD1789962803596J6KIGM`
- 新建任务见 `docs/task-brief-template.md`
- 流程见 `docs/workflow.md`

**承接方注意**：不要修改本目录下的 `brief.md`；只允许改 `status.json` 的 `state` 字段。
