---
name: collect-stats-tiktok
description: 用 Playwright MCP 進入 TikTok Studio Analytics，擷取近 7 天影片互動數據
---

## Input
- `days`：預設 7

## Output
```json
{
  "platform": "tiktok",
  "followers": 0,
  "follower_delta_7d": 0,
  "videos": [
    { "url": "...", "published_at": "2026-04-15", "views": 0, "likes": 0, "comments": 0, "shares": 0, "avg_watch_time_sec": 0 }
  ],
  "top_3": ["url1", "url2", "url3"],
  "best_times": [
    { "day": "wed", "hour": 20, "score": 0.88 }
  ]
}
```

`best_times` 欄位 schema 同 `collect-stats-facebook`。

## 流程
1. `browser_navigate` → `https://www.tiktok.com/tiktokstudio/analytics`
2. 在時間篩選切到「過去 7 天 / Last 7 days」
3. 從 Overview 抓：
   - 粉絲總數（Followers）
   - 7 天粉絲成長（Follower growth）
4. 切到「內容 / Content」或「影片 / Videos」分頁，逐列抽：
   - `url`：點開單則影片找到分享連結 `https://www.tiktok.com/@{handle}/video/{id}`，或直接用列中連結
   - `published_at`
   - `views` / `likes` / `comments` / `shares`
   - `avg_watch_time_sec`：若在清單看不到，開單則影片 detail 頁面抓
5. 按 `views` 排序取 top 3
6. 切到「追蹤者 → 追蹤者活躍時段」熱圖，抽前 3 高分時段寫入 `best_times`
   - 粉絲少於 ~100 時 TikTok 不顯示 → 回傳空 `[]`
7. 回傳 JSON

## 不要做
- 不要嘗試呼叫 TikTok API
- 不要把資料寫入 Drive — 由 caller 交給 report-writer
- TikTok 數字有「K/M/B」縮寫（例：`1.2K`），必須轉成純整數再輸出
- 隱私權為「僅自己 / Only Me」或狀態為「內容審查中」的影片要標記 `visibility: "private"`，並建議 caller 從 top_3 排除
