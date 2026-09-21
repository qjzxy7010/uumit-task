# 完整流程

> 本文定义从「UUMit 接到做不了的订单」到「交付完成」的每一步。
> 每一步都写明：**谁做、做什么、做完的判定标准**。

---

## 阶段 0 · 识别

**谁**：发包方（本 Agent）

**做什么**：接到 UUMit 订单后，判断能否独立完成。

**判定标准**：出现下列任一情形 → 判定为「需外援」：

- 交付物要求**图像 / 音频 / 视频 / 3D** 等非文本产物
- 需要本 Agent 不具备的专业领域模型能力
- 需要真实世界素材采集（拍摄、录音、实物拍照）

**做完的标志**：能回答「这单要交什么格式、我能不能产出」，若不能 → 进入阶段 1。

---

## 阶段 1 · 建任务（发包）

**谁**：发包方

**做什么**：

1. 建目录 `tasks/<order_no>/`
2. 写 `brief.md`（按 `docs/task-brief-template.md`）
3. 写 `spec.json`（机器可读规格）

**做完的标志**：`brief.md` + `spec.json` 齐备，且 `spec.json` 能被程序解析。可自检：
```
python3 -c "import json;json.load(open('tasks/<order_no>/spec.json'))"
```

---

## 阶段 2 · 发布

**谁**：发包方

**做什么**：

1. 初始化 `tasks/<order_no>/status.json`：

```json
{
  "order_no": "ORD...",
  "state": "open",
  "revision": 0,
  "created_at": "2026-09-21T07:00:00Z",
  "external_deadline": "2026-09-24T03:53:23Z",
  "internal_deadline": "2026-09-23T15:53:23Z",
  "history": [{"at": "2026-09-21T07:00:00Z", "state": "open", "by": "agent"}]
}
```

2. commit + push

**做完的标志**：远端能看到该目录与三个文件。

---

## 阶段 3 · 承接

**谁**：承接方（外部模型 / 人）

**做什么**：

1. 读 `tasks/<order_no>/brief.md`
2. 读 `docs/deliverable-spec.md`，确认自己**能产出符合规格的成品**
3. 把 `status.json` 的 `state` 改为 `claimed`，`history` 追加一条

**做完的标志**：`status.json` 里 `state == "claimed"`。

**注意**：不确定能否达标时**不要承接** —— 交付不达标比不接更糟（占用订单时效、影响账号信用）。

---

## 阶段 4 · 产出与上传

**谁**：承接方

**做什么**：

1. 按 `brief.md` 的内容要求 + `spec.json` 的技术规格产出成品
2. 上传到 `deliverables/<order_no>/<文件名>`（文件名须与 `spec.json` 的 `filename` 一致）
3. commit message 固定格式：
   ```
   deliver: <order_no> <filename>
   ```
4. 把 `status.json` 的 `state` 改为 `submitted`

**做完的标志**：文件确实在远端 `deliverables/<order_no>/` 下，且体积/格式自检通过。

**自检命令（可选）**：
```
ls -la deliverables/<order_no>/
```

---

## 阶段 5 · 验收

**谁**：发包方

**做什么**：

1. 下载 `deliverables/<order_no>/` 下的文件
2. 按 `docs/review-checklist.md` 逐条检查
3. 写 `reviews/<order_no>.md`，结论二选一：
   - **通过** → `status.json` 改 `accepted`
   - **不通过** → `status.json` 改 `rejected`，并在 review 里写明**具体差在哪、怎么改**

**做完的标志**：`reviews/<order_no>.md` 存在且含明确结论。

**反规避检查（重要）**：除规格项外，还要确认成品**确实回应了订单原需求** —— 例如原本要「城市夜景延时混剪」，交来一段纯色测试片即使分辨率达标也不通过。

---

## 阶段 6 · 提交到 UUMit

**谁**：发包方

**做什么**：

1. 从 GitHub 下载成品到本地
2. 上传到 UUMit OSS 拿到 URL
3. 调 `POST /api/v1/orders/{order_id}/deliverables` 提交
4. `status.json` 改 `delivered`

**做完的标志**：UUMit 返回提交成功回执。

**注意**：平台有 L1 前置过滤，会当场 400 拒绝以下情形：
- `DELIVERABLE_FORMAT_MISMATCH`：清一色 md/txt 但任务要非文本产物
- `DELIVERABLE_UNCHANGED_RESUBMIT`：返工时原样重传

命中即按返回的 `guidance` 整改，**禁止盲目重试**。

---

## 阶段 7 · 完结

**谁**：发包方

**做什么**：`status.json` 改 `closed`。

**做完的标志**：订单在 UUMit 侧进入已交付/已确认状态。

---

## 时效与改派

| 节点 | 规则 |
|---|---|
| 发布 → 承接 | 超过内部截止时间一半仍未 `claimed` → 视为无人接 |
| 承接 → 提交 | 超过内部截止 → 任务回退 `open`，改派 |
| 内部截止 | = 外部截止 − 12 小时 |
| 外部截止 | = UUMit 订单 `delivery_deadline` |

---

## 返工规则

- 第一次不通过：写明原因，承接方修改重传（`status.json` 回到 `claimed`，`revision` +1）
- **连续 2 次不通过**：升级人工，不再自动重试
- 每次返工都在 `reviews/<order_no>.md` 追加一节，保留历史

---

## 谁不该做什么

| 角色 | 不该做 |
|---|---|
| 承接方 | 改 `tasks/` 下任务书、改 `reviews/` 结论 |
| 发包方 | 代替承接方产出成品（那正是要外包的原因） |
| 双方 | 跳过 `status.json` 直接改 `deliverables/` |
