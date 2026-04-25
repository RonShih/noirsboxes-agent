---
name: publish-youtube
description: 用 Playwright 在 YouTube Studio 上傳影片並填 metadata。
---

> 共通規則見 `CLAUDE.md`「共通行為規則」章節，本檔只描述 YT 特有的步驟。

## Input
- `local_video_path`：本地影片絕對路徑
- `title`：YT 標題
- `description`：YT 描述
- `tags`：list of string
- `visibility`：public | unlisted | private（預設 public）
- `playlist`：選填

## Output
```json
{ "video_url": "https://www.youtube.com/watch?v=..." }
```

## 流程
1. `browser_navigate` → `https://studio.youtube.com/`
2. 「Create」→「Upload videos」
3. `browser_file_upload` 傳 `local_video_path`
4. 等 Details 頁載入
5. 清空 title 欄 → 輸入 `title`
6. 清空 description 欄 → 貼 `description`
7. 「Show more」→ Tags 欄輸入 `tags.join(", ")`
8. 「Audience」→ 選「No, it's not made for kids」
9. 連點兩次「Next」跳過 elements / checks
10. Visibility 頁選對應 radio button
11. 等 Save 按鈕可點 → 點「Save / Publish」
12. 等「Video published」對話框 → 抓影片 URL

## YT 特有的坑
- 大檔上傳要 5-15 分 → `browser_wait_for` 等進度條消失 / 「Processing complete」
- 偶爾要求「Verify channel」→ 回 `{ "error": "need channel verification" }` 中止
