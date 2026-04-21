---
name: publish-x
description: 用 Playwright MCP 在 X (Twitter) 發一則推文，可選帶本地圖片
---

## Input
- `caption`：≤ 280 字元（含 hashtags）
- `local_image_path`：選填，本地圖檔**絕對路徑**

## Output
```json
{ "post_url": "https://x.com/<handle>/status/..." }
```

## 流程
1. `browser_navigate` → `https://x.com/home`
2. 點「Post」按鈕（或用快捷鍵 `n`）
3. 輸入 caption
4. 若有 `local_image_path`：
   - `test -f` 確認檔案存在
   - 點「新增圖片」按鈕 → `browser_file_upload` 直接傳 `local_image_path`
5. 用 `Meta+Enter`（macOS）或點「Post」送出
6. 等頁面刷新、從 URL 或 toast 抓到 post URL
7. 回傳 post URL

## 不要做
- **不要下載任何 Drive URL** — 圖片若存在就是本地絕對路徑
- **不要刪除 `local_image_path`** — 素材共享
- **不要 `browser_take_screenshot` 存檔到 logs/** — 不產生任何 .png
- （`browser_snapshot` 是讀頁面結構、不是截圖，**該用就用**）
- **不要因為「過去某列失敗」就跳過不發**。你被 call 就是代表 /publish-now 已經判斷這列要發。**照步驟跑到底**，只有在實際遇到錯誤時才回 `error`。不要「預測」失敗、不要換順序、不要自作主張「先試比較安全的」。

## 狀態
- 實測可自動發（2026-04-21 probe 確認，`ronshih_` 帳號成功發文）
