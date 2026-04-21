---
description: 對應驗收 #6 — 從本地內容日曆讀取到期列，自動發布到 5 平台（FB / IG / X / YouTube / TikTok）
argument-hint: "[row_id 或 'due'（預設 due，發布當下時段所有 status=scheduled 的列）]"
---

步驟：

0. **短路檢查（省 token 用）** — 在做任何事之前先判斷有沒有 due 列：
   - 呼叫 `gdrive-reader`（本地讀 `data/calendar.json`，**不要碰 Playwright**）
   - 依下述規則算出 due 列
   - 若 due 列數 = 0 → **立刻結束、直接 return**，不要啟動 browser、不要做任何 Playwright 操作、不要「順便確認登入狀態」
   - 若 due 列數 ≥ 1 → 進入步驟 2 開始發文

1. 讀 calendar 的規則（步驟 0 用的）：
   - 資料源：`data/calendar.json`（本地 JSON，不是 Google Sheet）
   - 若參數是 row_id → 讀該 `row_id` 的列
   - 若是 `due`（預設）→ 讀所有符合下列條件的列（依 `config/schedule.yaml` 的 `timezone`）：
     - `status = scheduled`
     - `-cron.early_window_minutes ≤ (當下 - date+time) ≤ cron.publish_window_minutes`
     - 即：當下時間落在 `[date+time, date+time + 10 min]` 區間內（預設值）
   - **三種情境：**
     - 準時發：當下 ≥ slot 且 ≤ slot + 10 分 → 發
     - stale：當下比 slot 晚 > 10 分 → 跳過、notes 寫 `stale (>{window} min)`、保留 `scheduled` 給人工處理
     - 早於 slot（當下 < slot） → 不發，等下一輪 routine tick

2. 對每一列依 `platform` 欄分流呼叫對應 skill（單列單平台）：

   | platform 欄值 | 呼叫的 skill | 輸入 |
   |---|---|---|
   | `facebook` | `publish-facebook` | `caption` / `hashtags` / `local_image_path`（= row.image_path，絕對路徑） |
   | `instagram` | `publish-instagram` | 同上 |
   | `x` | `publish-x` | `caption`（含 hashtags 後 ≤ 280 字）、`local_image_path`（選填） |
   | `youtube` | `publish-youtube` | `title`(=caption 截 <100 字) / `description` / `tags`(=hashtags) / `local_video_path`（= row.video_path，絕對路徑） |
   | `tiktok` | `publish-tiktok` | `caption` / `hashtags` / `local_video_path` |

   未匹配任何 platform（空值或錯字） → status=`failed`，notes 寫 `unknown platform: <value>`

   **素材路徑**：素材**全部已在本地** `media/assets/` 底下。publish-* skill 收的路徑應該是 **absolute path**（`/Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/media/assets/xxx.jpg`）。**不要做任何下載動作**。

3. 每篇發完：
   - 呼叫 `gdrive-writer` 的 `update_calendar_row`，更新該 `row_id`：
     - `status = "published"`
     - `post_url = <平台回傳的永久連結>`
     - `published_at = <ISO8601 時間戳>`
     - `notes` 附加發布時間

4. 失敗的列 → `status = "failed"`，notes 寫錯誤訊息
   - 不要重試（skill 本身也不重試）
   - 不要因為一列失敗就停掉整批；繼續跑下一列

5. **所有列都處理完後 — 關瀏覽器**：
   - 呼叫 `mcp__playwright__browser_close` 把 browser 收掉
   - 目的：避免每次 routine tick 都留著一個 Chromium 視窗累積。policy 是「發完就關、沒發就不開」。
   - 步驟 0 直接 return 時（沒 due 列）不用 call browser_close（因為根本沒開）。

## 驗收檢查
- [ ] 5 平台有該貼文
- [ ] `data/calendar.json` 對應列 `status` → `published`、`post_url` 有值

## 不要做
- 不要把 X/YT/TikTok 當成「超出範圍」略過 — 都要發（驗收 #7 的長片另有 `/upload-youtube` 手動指令）
- 不要把多平台塞進同一列 — 一列一平台
- **不要碰 Google Drive / Sheet**。日曆是本地 `data/calendar.json`，素材是本地 `media/assets/`。
- **不要寫任何檔到 `logs/`** — 不寫 run-level md log、也**不要截圖存檔**（`browser_take_screenshot`）。`data/calendar.json` 的 `notes` + `status` 欄是唯一權威紀錄。發文成功失敗都只更新 calendar，不存 png / md。`browser_snapshot` 是讀頁面結構給 Claude 看的、不是截圖、**該用就用**。
- **不要基於歷史列的 notes 預測當下列會失敗、然後跳過或換順序**：
  - ❌ 看到過去某列「FB login gate」→ 推論「FB 封鎖自動化」→ 跳過今天的 FB 列
  - ❌ 「IG 可能類似 → 先試最安全的 X」→ 換順序或 skip
  - ✅ 每一列是獨立事件，照 `platform` 欄呼叫對應 skill、步驟跑完才能下判斷
  - ✅ 失敗了才 `status=failed`、notes 寫失敗原因，然後跑下一列；不要「預測」失敗
