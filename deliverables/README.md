# deliverables/ — 交付物目录

承接方把成品上传到这里：

```
deliverables/<order_no>/<文件名>
```

- `<order_no>` = UUMit 订单号
- `<文件名>` = 必须与 `tasks/<order_no>/spec.json` 的 `filename` 完全一致
- 单文件 ≤ **100 MB**（超限见 `docs/deliverable-spec.md` 第 4 节）

## 提交格式

```
git add deliverables/<order_no>/<文件名>
git commit -m "deliver: <order_no> <文件名>"
git push
```

然后改 `tasks/<order_no>/status.json`：`state` → `"submitted"`。

## 硬性约束

上传前请对照 `docs/deliverable-spec.md`：素材免版权、无水印、规格达标。

**提交前自检**：文件在不在、能不能打开、时长/分辨率对不对。
