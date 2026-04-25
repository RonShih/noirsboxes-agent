---
name: publish-facebook
description: 用 Playwright 在 Facebook 個人 timeline / 粉專發一篇含圖貼文。
---

> 共通規則見 `CLAUDE.md`「共通行為規則」章節，本檔只描述 FB 特有的步驟。

## Input
- `caption`：貼文文字
- `hashtags`：list of string（接在 caption 結尾）
- `local_image_path`：本地圖檔絕對路徑

## Output
```json
{ "post_url": "https://..." }
```

## 流程
1. 讀 `config/brand.yaml` 的 `socials.facebook.url`，`browser_navigate` 過去
2. 點「建立貼文 / Create post」
3. 文字框輸入 `caption + "\n\n" + hashtags.join(" ")`
4. 點圖片上傳鈕 → `browser_file_upload` 傳 `local_image_path`
5. `browser_wait_for` 等預覽載入完
6. 點「發佈 / Post」
7. 等回到動態消息頁、抓新貼文永久連結（失敗則回傳粉專 URL）

## FB 特有的坑
- 偶爾跳「您的密碼是否已洩漏」對話框 → 點 `Not now` / `稍後再說` 關閉後繼續
- 有時 dialog 正常關閉但貼文不出現（FB 反自動化偵測） → 回 `error` 寫「post dialog closed but not visible in timeline」
