---
name: publish-instagram
description: 用 Playwright 在 Instagram 發一篇 Feed 含圖貼文（非 Story）。
---

> 共通規則見 `CLAUDE.md`「共通行為規則」章節，本檔只描述 IG 特有的步驟。

## Input
- `caption`：貼文文字
- `hashtags`：list of string（IG 上限 30 個）
- `local_image_path`：本地圖檔絕對路徑

## Output
```json
{ "post_url": "https://..." }
```

## 流程
1. `browser_navigate` → `https://www.instagram.com/`
2. 點左側 sidebar「Create / 建立」→「Post / 貼文」
3. `browser_file_upload` 傳 `local_image_path`
4. 「Next / 下一步」兩次（跳過裁切 → 篩選器）
5. caption 框輸入 `caption + "\n.\n.\n.\n" + hashtags.join(" ")`
   - IG 慣例：tags 與正文用 3 行點隔開
6. 點「Share / 分享」
7. 等回到首頁
8. 讀 `config/brand.yaml` 的 `socials.instagram.url`、`browser_navigate` 過去抓最新貼文 URL

## IG 特有的坑
- IG 對自動化敏感、容易被風控 → 卡登入頁 / 卡 2FA 就回 `error`，不要重試
- 這個 skill 只處理 Feed 圖片；要傳影片用其他 skill
