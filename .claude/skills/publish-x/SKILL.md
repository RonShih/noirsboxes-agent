---
name: publish-x
description: 用 Playwright 在 X (Twitter) 發一則推文，可選帶本地圖片。
---

> 共通規則見 `CLAUDE.md`「共通行為規則」章節，本檔只描述 X 特有的步驟。

## Input
- `caption`：≤ 280 字元（含 hashtags）
- `local_image_path`：選填，本地圖檔絕對路徑

## Output
```json
{ "post_url": "https://x.com/<handle>/status/..." }
```

## 流程
1. `browser_navigate` → `https://x.com/home`
2. 點「Post」按鈕（或用快捷鍵 `n`）
3. 輸入 caption
4. 若有 `local_image_path` → 點「新增圖片」、`browser_file_upload` 傳檔
5. 用 `Meta+Enter`（macOS）或點「Post」送出
6. 等頁面刷新、從 URL 或 toast 抓 post URL
