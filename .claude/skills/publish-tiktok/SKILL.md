---
name: publish-tiktok
description: 用 Playwright 在 TikTok Studio 上傳短影音。
---

> 共通規則見 `CLAUDE.md`「共通行為規則」章節，本檔只描述 TikTok 特有的步驟。

## Input
- `local_video_path`：本地影片絕對路徑
- `caption`：說明文字
- `hashtags`：list of string

## Output
```json
{ "video_url": "https://..." }
```

## 流程
1. `browser_navigate` → `https://www.tiktok.com/tiktokstudio/upload`
2. `browser_file_upload` 傳 `local_video_path`
3. 等預覽載入
4. caption 框輸入 `caption + " " + hashtags.join(" ")`
5. 點「Post」
6. 等跳到 `/tiktokstudio/content`、抓影片 URL

## TikTok 特有的坑
- 「開啟自動內容檢查？」dialog → **點「開啟」**（取消會被降級為「僅自己」）
- 「已新增編輯功能」alert → 點「知道了」
- 發佈前驗證可見度欄位是「所有人」（被降級就手動切回）
- 對自動化敏感，要求額外驗證時回 `{ "error": "TikTok blocked / verification required" }`
