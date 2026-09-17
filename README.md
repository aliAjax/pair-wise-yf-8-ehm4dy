# 手摇风琴纸带打孔API

纯后端零依赖Node服务，使用 `data/db.json` 持久化曲目、纸带区间和试奏问题。

## 启动

```bash
PORT=3019 node server.js
```

## 主要接口

- `GET /health`
- `GET /tunes`
- `POST /tunes`
- `GET /tunes/:id/progress`
- `GET /tunes/:id/sections`
- `POST /tunes/:id/sections`
- `GET /tunes/:id/unchecked-sections`
- `PATCH /sections/:id/check`
- `GET /issues?tuneId=&status=`
- `POST /issues`
- `PATCH /issues/:id/status`

## 试奏核对闭环

- `PATCH /sections/:id/check`（请求体 `{"checked":true}` 或省略）只有在以下条件都满足时才能把区间标记为已核对：
  1. 该区间下没有未解决问题（`status !== "resolved"`）；
  2. 同一曲目内所有起始拍（`startBeat`）更早的区间都已核对。
- 条件不满足时返回 `409`，响应体包含 `error` 和结构化 `reasons`（`open_issues` / `earlier_sections_unchecked`，附相关 id 列表），区间核对状态与备注保持不变。
- `{"checked":false}` 取消核对不受门槛限制。
- 已核对区间一旦出现未解决问题（`POST /issues` 新增，或 `PATCH /issues/:id/status` 把已解决问题重新打开），自动恢复为待核对（响应 `meta.sectionRevertedToUnchecked: true`）。
- 最后一个问题解决后区间**不会**自动通过，必须再次显式调用 check 接口重新核对。
- 新建区间一律以待核对开始（创建时传入的 `checked` 字段被忽略）。
- `GET /tunes/:id/progress` 始终按真实的已核对区间数统计，自动回退后进度立即下降。

## 闭环示例

```bash
curl http://127.0.0.1:3019/tunes/tune_demo/progress
curl -X POST http://127.0.0.1:3019/issues \
  -H 'Content-Type: application/json' \
  -d '{"tuneId":"tune_demo","sectionId":"section_demo_2","type":"错孔","beat":45,"lane":9,"description":"第45拍第9轨多打孔"}'
```
