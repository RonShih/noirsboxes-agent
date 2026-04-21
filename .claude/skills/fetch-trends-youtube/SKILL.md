---
name: fetch-trends-youtube
description: 用 Playwright MCP 爬 YouTube Taiwan 熱門頁，回傳熱門影片標題與頻道
---

## Output
```json
{
  "platform": "youtube",
  "fetched_at": "2026-04-20T14:00:00+08:00",
  "region": "TW",
  "trends": [
    { "topic": "影片標題...", "channel": "頻道名", "views": 123456, "url": "https://www.youtube.com/watch?v=...", "rank": 1 }
  ]
}
```

## 流程
1. `browser_navigate` → `https://www.youtube.com/feed/trending?gl=TW`
2. 前 20 個影片逐項抽：標題 / channel / views（`1.2M views` → 1200000）/ 完整 video URL
3. 回傳 JSON（相關性由 caller 處理）

## 不要做
- 不要點進影片看內容
- 不要嘗試用 YouTube Data API
- 數字縮寫（K/M/B、萬、億）必須轉成純整數再輸出
