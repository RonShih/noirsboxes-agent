---
name: publish-youtube
description: 用 Playwright MCP 在 YouTube Studio 上傳影片並填入 metadata
---

## Input
- `local_video_path`：本地影片絕對路徑
- `title`：YT 標題
- `description`：YT 描述
- `tags`：list of string
- `visibility`：public | unlisted | private（預設 public）
- `playlist`：選填，若有則加入該播放清單

## Output
```json
{ "video_url": "https://www.youtube.com/watch?v=..." }
```

## 流程
1. 確認 `local_video_path` 實體存在（`test -f`），不存在回 `{ "error": "video not found", "path": "..." }`
2. `browser_navigate` → `https://studio.youtube.com/`
3. 點右上「Create」→「Upload videos」
4. `browser_file_upload` 直接傳 `local_video_path`（本地檔，無需下載）
4. 等「Details」頁載入
5. 清空 title 欄 → 輸入 `title`
6. 清空 description 欄 → 貼入 `description`
7. 「Show more」→ 在 Tags 欄輸入 `tags.join(", ")`
8. 「Audience」→ 選「No, it's not made for kids」
9. 連點兩次「Next」跳過 elements / checks 頁
10. 在 Visibility 頁選對應 radio button
11. 等到 Save 按鈕可點 → 點「Save / Publish」
12. 等「Video published」對話框 → 抓影片 URL

## 不要做
- **不要下載任何 Drive URL** — 輸入已是本地絕對路徑
- **不要刪除 `local_video_path`** — 素材共享，其他列會重用
- **不要 `browser_take_screenshot` 存檔到 logs/** — 不產生任何 .png
- （`browser_snapshot` 是讀頁面結構、不是截圖，**該用就用**）
- **不要因為「過去某列失敗 / YT 曾經要求 channel verification」就跳過不發**。你被 call 就是代表 /publish-now 已經判斷這列要發。**照步驟跑到底**，只有在實際遇到錯誤時才回 `error`。不要「預測」失敗。

## 已知坑
- 上傳大檔可能要 5–15 分鐘 → 用 `browser_wait_for` 等到進度條消失或「Processing complete」
- YT Studio 偶爾要求「Verify channel」→ 錯誤訊息回「need channel verification」並中止
