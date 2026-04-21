---
description: 對應驗收 #7 — 從本地 media/assets/ 拉一支影片，自動上傳 YouTube 並填完整 metadata
argument-hint: "[本地影片檔名，例：md905-long-tutorial.mp4；省略時會列 media/assets/*.mp4 讓使用者挑]"
---

步驟：

1. **定位影片**：
   - 若 caller 給了檔名 → 用 `media/assets/<filename>`
   - 若沒給 → `ls media/assets/*.mp4` 列出所有影片讓使用者挑（給絕對路徑選單）
   - 找不到 → 回報「請把影片放到 `media/assets/` 再跑一次」並停止

2. 從檔名（或 caller 補充）抽出產品 / 主題關鍵字

3. 呼叫 skill `seo-metadata`：輸入主題 + 目標市場語言（預設英文 + 中文標題各一），產出
   - title（≤ 100 字元）
   - description（含產品連結 https://www.noirsboxes.com/、3 行品牌簡介、章節時間戳）
   - tags（10–15 個）
   - thumbnail_hint（要不要自動截一張影片中段當縮圖）

4. 呼叫 skill `publish-youtube`，傳入：
   - `local_video_path`：該影片的**絕對路徑**（`/Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/media/assets/xxx.mp4`）
   - `title` / `description` / `tags` / `visibility`

5. 上傳完成後：
   - 回報 YouTube URL 給使用者
   - （選）把這筆紀錄 append 到 `data/uploads.json` 或 `data/calendar.json`（依 caller 決定）

## 驗收檢查
- [ ] YouTube 後台出現該影片
- [ ] title / description / tags 都已填入
- [ ] 回報 URL 可點進去看

## 不要做
- **不要碰 Google Drive** — 影片必須已在 `media/assets/`
- 不要寫 `logs/*.png`（截圖）
- 不要自動重試上傳 — YT 上傳失敗通常是 channel verification 或檔案問題，回錯讓人處理
