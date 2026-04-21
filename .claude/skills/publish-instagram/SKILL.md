---
name: publish-instagram
description: 用 Playwright MCP 在 Instagram 發一篇含圖貼文（Feed post，非 Story）
---

## Input
- `caption`：貼文文字
- `hashtags`：list of string
- `local_image_path`：本地圖檔的**絕對路徑**（`/Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/media/assets/xxx.jpg`）

## Output
```json
{ "post_url": "https://..." }
```

## 流程
1. 確認 `local_image_path` 實體存在（`test -f`），不存在回 `{ "error": "image not found", "path": "..." }`
2. `browser_navigate` → `https://www.instagram.com/`
3. 點左側 sidebar 的「Create / 建立」
4. 點「Post / 貼文」
5. `browser_file_upload` 直接傳 `local_image_path`
6. 點「Next / 下一步」兩次（裁切 → 篩選器）
7. 在 caption 框輸入 `caption + "\n.\n.\n.\n" + hashtags.join(" ")`
   - IG 的慣例：tags 與正文用 3 行點隔開
8. 點「Share / 分享」
9. 等待回到首頁
10. 讀 `config/brand.yaml` 的 `socials.instagram.url`，`browser_navigate` 到該頁面抓最新貼文 URL

## 失敗處理
- 同 publish-facebook
- IG 對自動化敏感 → 失敗時不要重試，錯誤訊息回在 error 字串

## 不要做
- 不要傳影片（這個 skill 只處理 Feed 圖片）
- 不要超過 IG hashtag 上限 30 個
- **不要下載任何 Drive URL** — 輸入已是本地絕對路徑
- **不要刪除 `local_image_path`** — 素材是共享的，會被其他列重用
- **不要 `browser_take_screenshot` 存檔到 logs/** — 不產生任何 .png
- （`browser_snapshot` 是讀頁面結構、不是截圖，**該用就用**）
- **不要因為「過去某列失敗 / IG 曾經卡登入 / 可能對自動化敏感」就跳過不發**。你被 call 就是代表 /publish-now 已經判斷這列要發。**照步驟跑到底**，只有在實際遇到錯誤時才回 `error`。不要「預測」失敗。
