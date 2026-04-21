# `data/` — 本地資料存放

這個目錄**刻意留空**。檔案在 `.gitignore` 裡，clone 下來是空的，各自產生。

## 會出現什麼檔

| 檔 | 產生方式 | 說明 |
|---|---|---|
| `calendar.json` | 跑 `/generate-calendar` 產生 | 內容日曆（每篇貼文一列） |
| `stats-history.json` | 跑 `/weekly-report` 累積 | 每週 stats 歷史 |

## `calendar.json` schema（參考）

執行 `/generate-calendar` 後會產類似這樣的 JSON：

```json
{
  "_schema": {
    "version": 1,
    "fields": [
      "row_id", "date", "time", "platform", "topic",
      "caption", "hashtags", "image_path", "video_path",
      "status", "notes", "post_url", "published_at"
    ],
    "status_enum": ["scheduled", "published", "failed", "cancelled"],
    "platform_enum": ["facebook", "instagram", "x", "youtube", "tiktok"]
  },
  "rows": [
    {
      "row_id": 1,
      "date": "2026-04-21",
      "time": "10:00",
      "platform": "facebook",
      "topic": "產品介紹",
      "caption": "...",
      "hashtags": ["#..."],
      "image_path": "media/assets/xxx.jpg",
      "video_path": null,
      "status": "scheduled",
      "notes": "..."
    }
  ]
}
```

發完文後 `/publish-now` 會把 `status` 從 `scheduled` 改成 `published`（或 `failed`），並補上 `post_url` + `published_at`。

## 為什麼不進 git

- 裡面會有**實測後的 post URL**（例：x.com/your-handle/status/...）
- 每個客戶的內容日曆都不一樣，不應共用
- 頻繁 append / update，commit 雜訊大

要備份就自己用 rsync / Google Drive / iCloud 處理。
