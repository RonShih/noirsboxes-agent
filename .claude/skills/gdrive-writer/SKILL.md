---
name: gdrive-writer
description: 寫本地內容日曆 (data/calendar.json) 與本地週報 (reports/*.md)。名字沿用但已完全本地化，不再碰 Google Drive / Sheet / Doc。
---

## 用途

所有寫入都針對本地檔：

1. **日曆 append / update** → 直接改 `data/calendar.json`
2. **週報 markdown** → 建 `reports/<name>.md`
3. **其他資源** — 貼文截圖、素材等一律寫在 `media/assets/` 或 `media/_tmp/`

（舊版用 Apps Script + Drive MCP，現已全部本地化。保留 skill 名字避免改動 call sites。）

## 四種操作模式

### 1. append_calendar_rows

**用途**：新增一批貼文列到 `data/calendar.json`（主要給 /generate-calendar 用）

**Input**:
- `rows`: 陣列，每筆 object 要符合 schema
  ```json
  {
    "date": "2026-04-21",
    "time": "10:00",
    "platform": "facebook",
    "topic": "...",
    "caption": "...",
    "hashtags": ["#..."],
    "image_path": "media/assets/xxx.jpg",
    "video_path": null,
    "status": "scheduled",
    "notes": "..."
  }
  ```

**實作**：
1. `cat` 讀現有 `data/calendar.json`
2. 用 `jq` 或重寫 JSON：計算下個 row_id、append 新列到 `rows[]`
3. 原子寫入：先寫到 `data/calendar.json.tmp` → `mv` 覆蓋
  ```bash
  jq --argjson new "$NEW_ROWS_JSON" \
    '.rows += ($new | [.[] | . + {row_id: (input_line_number)}])' \
    data/calendar.json > data/calendar.json.tmp && \
    mv data/calendar.json.tmp data/calendar.json
  ```
  （實務上用 Python 或 node 小 script 比較好算 row_id；也可以直接讓 Claude 用 Edit tool 修）

**Output**: `{ "appended": N }`

### 2. update_calendar_row

**用途**：更新單列（最常用 — 發文完後把 status=scheduled 改成 published）

**Input**:
- `row_id`: 整數
- `updates`: object，例如 `{ "status": "published", "notes": "...", "post_url": "https://..." }`

**實作**：
```bash
# 用 jq 篩出 row_id 對應的列、merge updates
jq --arg rid "$ROW_ID" --argjson upd "$UPDATES_JSON" \
  '.rows |= map(if .row_id == ($rid | tonumber) then . + $upd else . end)' \
  data/calendar.json > data/calendar.json.tmp && \
  mv data/calendar.json.tmp data/calendar.json
```

**Output**: `{ "updated": 1 }` 或 `{ "error": "row_id not found" }`

### 3. write_report_markdown

**用途**：週報用 — 寫純 markdown 到 `reports/`

**Input**:
- `filename`: 例如 `2026-W17.md`
- `markdown`: markdown 字串

**實作**：直接 `Write tool` 寫 `reports/<filename>`

**Output**: `{ "path": "reports/2026-W17.md" }`

### 4. save_asset

**用途**：把產生的圖 / 影片放進 `media/assets/`

**Input**:
- `name`: 檔名（例如 `md905-epr.jpg`）
- `content`: base64 或 binary path
- 或 `source_url`: 若來源是網路圖（還要 `curl -o`）

**實作**：
- 若 base64 → `echo <base64> | base64 -D > media/assets/<name>`
- 若 URL → `curl -sL -o media/assets/<name> <url>`

**Output**: `{ "path": "media/assets/<name>" }`

## 前置條件

- `data/` 目錄存在
- `data/calendar.json` 存在（若不存在，`/generate-calendar` 首次執行會建）
- `reports/` 目錄存在
- `media/assets/` 目錄存在

## 失敗處理

- JSON 解析失敗 → 回 `{ "error": "calendar.json corrupted" }`
- 並發寫入衝突（同秒兩個 process 寫同檔）→ 用 `.tmp` + `mv` 原子寫入、`flock` 加鎖可選
- 磁碟不足 → 回 `{ "error": "disk full" }`

## 不要做

- 不要走 Apps Script、Drive MCP、Google Sheet API
- 不要把 `data/calendar.json` 備份到 Drive（使用者要備份自己做）
- 不要在 skill 內修改超出呼叫者授權的欄位（只改 `updates` 裡列出的 key）
