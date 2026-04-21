---
name: publish-facebook
description: 用 Playwright MCP 在 Facebook 粉專發一篇含圖貼文（已假設使用者已透過 first-time-login 登入）
---

## Input
- `caption`：貼文文字
- `hashtags`：list of string，會接在 caption 結尾
- `local_image_path`：本地圖檔的**絕對路徑**（`/Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/media/assets/xxx.jpg`）

## Output
```json
{ "post_url": "https://..." }
```

## 流程（用 Playwright MCP tools）
1. 確認 `local_image_path` 實體存在（`test -f`），不存在回 `{ "error": "image not found", "path": "..." }`
2. 讀 `config/brand.yaml` 的 `socials.facebook.url`，`mcp__playwright__browser_navigate` 到該 URL（個人版直接發到 timeline；品牌粉專則發到該專頁）
3. 點「建立貼文 / Create post」按鈕（用 `browser_snapshot` 找元素 ref）
4. 在文字框輸入 `caption + "\n\n" + hashtags.join(" ")`
5. 點圖片上傳鈕 → `browser_file_upload` 直接傳 `local_image_path`（**不要經過 _tmp 拷貝**，本地檔直接用）
6. 等預覽載入（`browser_wait_for` text 含「準備就緒 / Ready」或圖片預覽出現）
7. 點「發佈 / Post」
8. 等待回到動態消息頁
9. 嘗試從新貼文擷取永久連結（若失敗則回傳粉專 URL）

## 不要做
- **不要 `browser_take_screenshot` 存檔到 logs/** — 不產生任何 .png。錯誤訊息直接回在 error 字串裡。
- （`browser_snapshot` 是讀頁面結構、不是截圖，**該用就用**，Claude 要靠它找按鈕位置）
- **不要下載任何 Drive URL** — 輸入已是本地絕對路徑
- **不要刪除 `local_image_path`** — 素材是共享的，會被其他列重用
- **不要因為「過去某列失敗 / FB 曾經卡登入 / 可能封鎖自動化」就跳過不發**。你被 call 就是代表 /publish-now 已經判斷這列要發。**照步驟跑到底**，只有在實際遇到錯誤時才回 `error`。不要「預測」失敗。

## 失敗處理
- 任何 step 失敗 → 回傳 `{ "error": "..." }`
- 不要重試超過 1 次

## 已知坑
- FB 偶爾跳「您的密碼是否已洩漏」對話框 → 找 `Not now` / `稍後再說` 並關閉後繼續
- FB 有時會靜默吞掉自動化貼文（UI 全過但貼文沒出現）→ 確認失敗就回 `error` 寫清楚「post dialog closed but not visible in timeline」
