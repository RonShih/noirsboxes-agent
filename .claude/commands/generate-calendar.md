---
description: 對應驗收 #5 — 產生下週內容日曆（文案 + 配圖 + Hashtag），寫入本地 data/calendar.json
argument-hint: "[週次起始日期 YYYY-MM-DD，省略 = 下週一]"
---

目標：產出**下週 7 天 × 5 平台**的內容日曆到 `data/calendar.json`，每列含 date / time / platform / topic / caption / hashtags / image_path / video_path / status=`scheduled`。

每平台每週至少 **3 篇貼文 + 2 支短影音**（PRD §3.2 FR-2.2）。

步驟：

1. 讀 `config/brand.yaml`、`config/products.yaml`、`config/schedule.yaml`

2. 用 `schedule.yaml` 的 `slots` 計算下週每個平台 slot 的 `date + time`（依 `timezone`），每個 slot 從 `content_themes` 與 `products` 抽選主題

3. 對每一篇貼文，呼叫 skill `content-writer` 產生 caption + hashtags（依平台調整風格）

4. 對每一篇貼文，呼叫 skill `image-generator` 產生或選取配圖：
   - **輸出是本地絕對路徑**（例如 `/Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/media/assets/<slug>.jpg`）
   - 若是短影音 slot（YT / TikTok），image-generator 回傳 `video_path`；否則回傳 `image_path`
   - 路徑存相對於專案根目錄的格式（`media/assets/xxx.jpg`）進 calendar

5. 全部準備好後，呼叫 skill `gdrive-writer` 的 `append_calendar_rows`，把所有列加到 `data/calendar.json` 的 `rows[]`

6. 最後輸出：本週共產生幾列、`data/calendar.json` 路徑

## 驗收檢查
- [ ] `data/calendar.json` 中存在下週 7 天的列
- [ ] 每平台至少 3 篇貼文 + 短影音
- [ ] 每列 caption / hashtags / image_path (或 video_path) 都非空
- [ ] 每個 image_path / video_path 指到的本地檔**實際存在**（`test -f` 可驗）

## 不要做
- 不要寫到 Google Sheet / Drive — **所有日曆資料都存本地**
- 不要用 Drive URL 當 image_url — **必須是本地 `media/assets/` 路徑**
- 不要跳過任何平台 — 預設 5 平台全產
