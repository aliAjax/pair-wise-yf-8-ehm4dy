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

- `PATCH /sections/:id/check` 标记已核对（`{"checked": true}`）需同时满足：
  - 该区间没有未解决问题（`status !== "resolved"` 的问题数为 0）；
  - 同曲目内所有起始拍更早（`startBeat` 更小）的区间均已核对。
- 条件不满足时返回 `409 {"error","reasons":[...],"data":区间}`，区间状态保持不变，不写库。
- 取消核对（`{"checked": false}`）始终允许，无前置条件。
- 已核对区间通过 `POST /issues` 新增问题时，区间自动恢复为待核对（`checked=false`，写回数据文件），响应中 `revertedSection` 为 `true`。
- 通过 `PATCH /issues/:id/status` 解决问题（包括解决最后一个问题）不会自动把区间标记为已核对，需重新试奏后再次调用 check 接口。
- 进度（`GET /tunes/:id/progress`）始终按真实的已核对区间数统计，自动回退会即时反映到 `checkedSections`/`percent`。

## 闭环示例

```bash
curl http://127.0.0.1:3019/tunes/tune_demo/progress
curl -X POST http://127.0.0.1:3019/issues \
  -H 'Content-Type: application/json' \
  -d '{"tuneId":"tune_demo","sectionId":"section_demo_2","type":"错孔","beat":45,"lane":9,"description":"第45拍第9轨多打孔"}'
```
