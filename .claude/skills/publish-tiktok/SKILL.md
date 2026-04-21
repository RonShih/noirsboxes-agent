---
name: publish-tiktok
description: 用 Playwright MCP 在 TikTok Studio 上傳短影音（驗收 #6/#7 不要求，先預留骨架供未來擴充）
---

## Input
- `local_video_path`
- `caption`
- `hashtags`

## Output
```json
{ "video_url": "https://..." }
```

## 流程
1. 確認 `local_video_path` 實體存在（`test -f`），不存在回 `{ "error": "video not found", "path": "..." }`
2. `browser_navigate` → `https://www.tiktok.com/tiktokstudio/upload`
3. `browser_file_upload` 直接傳 `local_video_path`（本地檔，無需下載）
4. 等預覽
5. caption 框輸入 `caption + " " + hashtags.join(" ")`
6. 點「Post」

## 不要做
- **不要下載任何 Drive URL** — 輸入已是本地絕對路徑
- **不要刪除 `local_video_path`** — 素材共享
- **不要 `browser_take_screenshot` 存檔到 logs/** — 不產生任何 .png
- （`browser_snapshot` 是讀頁面結構、不是截圖，**該用就用**）
- **不要因為「過去某列失敗 / TikTok 對自動化敏感」就跳過不發**。你被 call 就是代表 /publish-now 已經判斷這列要發。**照步驟跑到底**，只有在實際遇到錯誤時才回 `error`。不要「預測」失敗。

## 狀態
- **未列入驗收**，僅保留入口供未來擴充
- 第一版若使用者沒下指令呼叫，不需執行
