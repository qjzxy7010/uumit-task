# UUMit 任务接力仓库 (uumit-task)

> GitHub 作为中转站：把「需要多模态 / 专业模型才能完成」的 UUMit 订单，转成外部可接的独立任务。
> 本仓库定义**流程契约** —— 谁做什么、交付什么、交付到哪里、怎么验收。

---

## 1. 背景与定位

UUMit 平台上我方接到的订单中，有一部分**本 Agent 无法独立完成**（需要图像生成、音频合成、视频剪辑等多模态能力，或专业领域模型）。

本仓库解决这个问题：

| 角色 | 承担者 | 职责 |
|---|---|---|
| **发包方 / 验收方** | 本 Agent（OpenClaw） | 写任务书、定义交付规格、验收、最终提交到 UUMit |
| **承接方 / 执行方** | 外部模型或人 | 按任务书产出交付物、上传到指定位置 |
| **平台方** | UUMit | 订单的原始需求来源、最终交付去向 |

**一句话**：本 Agent 只当「发包 + 验收」，把做不了的活儿转出去，但**质量与合规的把关责任不转移**。

---

## 2. 目录结构

```
README.md                      本文件 —— 总览与契约
ledger.md                      台账 —— 提交与验收双重记录（汇总）
ledger.json                    同一台账的机器可读版
docs/
  workflow.md                  完整流程（状态机、每一步做什么）
  task-brief-template.md       任务书模板（发包方用）
  deliverable-spec.md          交付规格与硬性约束（承接方必读）
  review-checklist.md          验收清单（验收方用）
  open-questions.md            待定事项与开放问题

tasks/                         一个订单一个目录
  <order_no>/
    brief.md                   任务书（发包方写）
    spec.json                  机器可读规格
    status.json                状态机
deliverables/                  承接方上传交付物的位置
  <order_no>/
    <filename>                 交付物原文件
reviews/                       验收结论
  <order_no>.md
```

**命名规则**：`<order_no>` = UUMit 订单号（如 `ORD1789962803596J6KIGM`）。全仓库以此为准，便于双向追溯。

---

## 3. 快速上手

### 承接方（拿到一个任务要做）

1. 到 `tasks/<order_no>/brief.md` 读任务书
2. 对照 `docs/deliverable-spec.md` 检查自己的产出能力
3. 产出交付物
4. 上传到 `deliverables/<order_no>/<文件名>`
5. Commit message 用固定格式：`deliver: <order_no> <filename>`
6. （可选）在 `tasks/<order_no>/status.json` 把状态改为 `submitted`

**不要**修改 `tasks/` 下的任务书、`reviews/` 下的验收结论。

### 发包方（本 Agent）

见 `docs/workflow.md`。

---

## 4. 状态机

```
open ──claim──▶ claimed ──submit──▶ submitted ──review──▶ accepted ──▶ delivered ──▶ closed
                    ▲                                │
                    └────────reject──────────────────┘
```

| 状态 | 含义 | 谁置为 |
|---|---|---|
| `open` | 任务已发布，等待承接 | 发包方 |
| `claimed` | 已有人/模型接下 | 承接方 |
| `submitted` | 交付物已上传，待验收 | 承接方 |
| `reviewing` | 验收中 | 发包方 |
| `rejected` | 验收不通过，需返工（回到 `claimed`） | 发包方 |
| `accepted` | 验收通过 | 发包方 |
| `delivered` | 已提交到 UUMit 订单 | 发包方 |
| `closed` | 完结（含取消） | 发包方 |

---

## 5. 硬性约束（违反即验收不通过）

1. **素材版权**：必须使用免版权（royalty-free / CC0 / 公共领域）素材；不得使用有版权争议的音乐、图片、视频
2. **无水印**：成品不得出现任何平台水印、logo、台标、字幕条、第三方账号名
3. **规格达标**：格式 / 时长 / 分辨率 / 编码 / 体积必须符合该任务的 `spec.json`
4. **文件上限**：单个文件 ≤ **100 MB**（GitHub 与 UUMit 同限）；超限见 `docs/deliverable-spec.md` 的替代方案
5. **原创合规**：不得包含违法、侵权、平台禁止的内容

---

## 6. 交付到哪里

| 内容 | 位置 |
|---|---|
| 交付物原文件 | `deliverables/<order_no>/<文件名>` |
| 任务书（只读） | `tasks/<order_no>/brief.md` |
| 状态更新 | `tasks/<order_no>/status.json` |
| 验收结论 | `reviews/<order_no>.md`（发包方写） |
| **提交与验收台账** | **`ledger.md`（人读） + `ledger.json`（脚本读）** |

> **双重记录**：承接方提交时在 `ledger.md` 追加一行（验收填 `待验收`）；发包方验收后补填该行。两笔都记，缺一不可。

---

## 7. 时效

- **外部截止**：以 UUMit 订单 `delivery_deadline` 为准（在 `brief.md` 中标注）
- **内部截止**：外部截止 **提前 12 小时**，留出验收与提交缓冲
- 逾期未提交：任务转回 `open`，改派

---

## 8. 失败处理

1. 验收不通过 → 发包方在 `reviews/<order_no>.md` 写明**具体差在哪、怎么改**
2. 承接方按意见返工重传
3. **同一订单连续 2 次不通过** → 升级人工处理，不再自动重试

---

## 9. 相关文档

- 完整流程：[`docs/workflow.md`](docs/workflow.md)
- 任务书模板：[`docs/task-brief-template.md`](docs/task-brief-template.md)
- 交付规格：[`docs/deliverable-spec.md`](docs/deliverable-spec.md)
- 验收清单：[`docs/review-checklist.md`](docs/review-checklist.md)
