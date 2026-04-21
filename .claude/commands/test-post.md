---
description: 一次性 demo — 在 FB / X / IG / YouTube / TikTok 個人帳號各發一則測試貼文，驗證 Playwright MCP 流程打通。不寫 calendar、純手動測試。
---

## 目標
證明 5 個平台的發文鏈路全通：
- **FB / X**：純文字貼文
- **IG**：從 `media/assets/` 讀圖片 → 上傳
- **YouTube / TikTok**：從 `media/assets/` 讀短影片 → 上傳

**注意**：發到使用者「個人」帳號（不是 NoirsBoxes 官方），`first-time-login` 時請登入個人帳號。

## 前置：確認素材存在

素材應該已經在 `media/assets/` 底下：
- 任一張 `.jpg` / `.png` → IG 用
- 任一支 `.mp4` → YT + TikTok 共用

檢查：
```bash
ls media/assets/*.jpg | head -1
ls media/assets/*.mp4 | head -1
```

找不到就回報使用者「請先把素材放到 `media/assets/`」並停止 IG / YT / TikTok 三個任務。

## 五個任務（順序執行，每個失敗後停下來問使用者要不要繼續）

### Task 1 — Facebook 個人 Timeline 發 `test`

1. `browser_navigate` → `https://www.facebook.com/`
2. `browser_snapshot` 找「在想些什麼？/ What's on your mind?」輸入框
3. 點開 → 輸入 `test` → 點「發佈 / Post」
4. 等回到動態消息頁
5. 回報 post URL（若能抓到）

### Task 2 — X（Twitter）個人帳號發 `test from noirsboxes-agent`

1. `browser_navigate` → `https://x.com/home`
2. `browser_snapshot` 找「What's happening? / 有什麼新鮮事？」輸入框
3. 輸入 `test from noirsboxes-agent` → 點「Post / 發文」（或 `Meta+Enter`）
4. 等 tweet 出現在 timeline
5. 回報 post URL

### Task 3 — Instagram 個人帳號發圖片貼文（caption: `test`）

素材：`media/assets/<第一張圖>.jpg`（絕對路徑）

1. `browser_navigate` → `https://www.instagram.com/`
2. 點左側「Create / 建立」→「Post / 貼文」
3. `browser_file_upload` 直接傳該圖的**絕對路徑**
4. 點「Next」兩次跳過濾鏡、編輯
5. caption 欄輸入 `test` → 點「Share / 分享」
6. 若 IG 要求 2FA 或被風控 → 回 `{ "error": "IG blocked" }`，**跳過此任務**
7. 成功則回報 post URL

### Task 4 — YouTube 個人頻道上傳短影片（title: `test`）

素材：`media/assets/<第一支 mp4>.mp4`（絕對路徑）

1. `browser_navigate` → `https://studio.youtube.com/`
2. 點「Create」→「Upload videos」
3. `browser_file_upload` 直接傳該影片的**絕對路徑**
4. Title 輸入 `test`
5. 其餘欄位接受預設（視情況選「Not made for kids」）
6. 點「Publish / 發布」為 Public
7. 回報 video URL

### Task 5 — TikTok 個人帳號上傳短影片（caption: `test`）

素材：`media/assets/<同一支 mp4>.mp4`

1. `browser_navigate` → `https://www.tiktok.com/tiktokstudio/upload`
2. `browser_file_upload` 直接傳該影片的**絕對路徑**
3. caption 輸入 `test`（清空預設填入的檔名再輸入）
4. **處理彈窗 — 非常重要：**
   - 「開啟自動內容檢查？」dialog → **點「開啟」**（不要點取消，取消會讓 TikTok 把可見度降級為「僅自己」）
   - 「已新增編輯功能」alert → 點「知道了」
5. **發佈前驗證可見度欄位仍是「所有人」**（若被 downgrade 到「僅自己」要手動切回）
6. 點「發佈」
7. 等 URL 跳到 `/tiktokstudio/content`
8. 驗收：可見度應是「所有人」而非「僅自己」
9. 若 TikTok 要求額外驗證 → 回 `{ "error": "TikTok blocked" }`，**跳過此任務**

## 結束時輸出

```
| Platform | Status    | URL / 原因 |
| FB       | ✅/❌     | ... |
| X        | ✅/❌     | ... |
| IG       | ✅/⚠️/❌  | ... |
| YouTube  | ✅/⚠️/❌  | ... |
| TikTok   | ✅/⚠️/❌  | ... |
```

## 不要做
- 不要重試超過 1 次
- 不要動到 NoirsBoxes 官方帳號 / 粉專
- **不要碰 Google Drive** — 素材必須已在 `media/assets/`
- **不要寫 `data/calendar.json`**（test-post 是純驗證，不影響正式 calendar）
- 不要 `browser_take_screenshot` 存檔到 logs/（`browser_snapshot` 讀頁面結構可用）
