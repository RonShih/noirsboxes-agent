---
name: gdrive-reader
description: 讀本地內容日曆 (data/calendar.json) 與本地素材 (media/assets/)。名字沿用但已完全本地化，不再碰 Google Drive / Sheet。
---

## 用途

從本地讀取：
1. **內容日曆** — `data/calendar.json`（JSON 物件，含 `rows[]` 陣列）
2. **本地素材** — `media/assets/*`（圖片、影片）

（舊版透過 Apps Script + Drive MCP 存取，現已全部本地化。保留 skill 名字避免改動 call sites。）

## 三種操作模式

### 1. read_calendar

**用途**：讀整份 calendar，回傳 JSON

**實作**：
```bash
cat /Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/data/calendar.json
```

**回傳**：JSON object，包含 `_schema` + `rows[]`

### 2. filter_due_rows

**用途**：給 /publish-now 專用 — 讀 calendar、計算當下哪些列在時間窗內且 status=scheduled

**邏輯**：
```
讀 data/calendar.json
讀 config/schedule.yaml 的 cron 區塊（early_window_minutes、publish_window_minutes）
對 rows[] 的每列：
  if status != "scheduled": skip
  slot_datetime = parse(row.date + " " + row.time, tz="Asia/Taipei")
  diff_min = (now - slot_datetime) / 60
  if -early_window_minutes <= diff_min <= publish_window_minutes:
    → due（加入輸出）
  elif diff_min > publish_window_minutes:
    → stale（跳過，保留 status=scheduled 給人工看）
  else:
    → 太早（跳過）
```

**回傳**：due 列的陣列（含 `row_id`、`platform`、`caption`、`image_path` / `video_path` 等）

### 3. get_asset_path

**用途**：給一個素材相對路徑，回傳絕對路徑 & 確認存在

**輸入**：`image_path` 或 `video_path`（相對於專案根目錄，例如 `media/assets/md905.jpg`）

**實作**：
```bash
test -f /Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/<path> && \
  echo /Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/<path>
```

**回傳**：absolute path（若存在） / error（若不存在）

## 前置條件

- `data/calendar.json` 存在（否則 `/generate-calendar` 先跑）
- `media/assets/` 存在，且 calendar 裡 row.image_path / video_path 對應的實體檔要存在

## 失敗處理

- JSON 解析失敗 → 回 `{ "error": "calendar.json parse failed", "hint": "check syntax" }`
- asset 路徑指到不存在檔 → 回 `{ "error": "asset not found", "path": "..." }`
- 不要丟例外，一律結構化回傳

## 不要做

- 不要走 Drive MCP、不要走 Apps Script、不要走任何 Google 服務
- 不要嘗試從 URL 下載 — 素材必須已在 `media/assets/` 底下
- 不要在 skill 內快取或重排資料（交給 command 處理）
