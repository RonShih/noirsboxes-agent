---
name: collect-stats-youtube
description: 用 Playwright MCP 進入 YouTube Studio Analytics，擷取近 7 天互動數據
---

## Input
- `days`：預設 7

## Output
```json
{
  "platform": "youtube",
  "subscribers": 0,
  "subscriber_delta_7d": 0,
  "videos": [
    { "url": "...", "published_at": "...", "views": 0, "watch_time_min": 0, "likes": 0, "comments": 0 }
  ],
  "top_3": ["url1", "url2", "url3"],
  "best_times": [
    { "day": "tue", "hour": 20, "score": 0.85 }
  ]
}
```

`best_times` 欄位 schema 同 `collect-stats-facebook`。

## 流程
1. `browser_navigate` → `https://studio.youtube.com/`
2. 左側點「Analytics」
3. 切到「Last 7 days」
4. 在「Content」分頁逐列抽 video URL / views / watch time / likes / comments
5. 從 Overview 卡片抓訂閱數
6. Top 3 by views
7. 切到「觀眾 → 觀眾在 YouTube 上的時段」熱圖，抽前 3 高分時段寫入 `best_times`
   - 頻道訂閱數過低時 YT 不顯示熱圖 → 回傳空 `[]`
8. 回傳 JSON
