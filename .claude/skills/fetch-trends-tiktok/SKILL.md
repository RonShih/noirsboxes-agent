---
name: fetch-trends-tiktok
description: 用 Playwright MCP 爬 TikTok Creative Center 台灣熱門 hashtag，回傳近 7 天 trend
---

## Output
```json
{
  "platform": "tiktok",
  "fetched_at": "2026-04-20T14:00:00+08:00",
  "region": "TW",
  "trends": [
    { "topic": "#快充", "volume": 125000, "growth_pct": 23, "rank": 1 }
  ]
}
```

## 流程
1. `browser_navigate` → `https://ads.tiktok.com/business/creativecenter/inspiration/popular/hashtag/pc/zh?period=7&countryCode=TW`
2. 從 hashtag 表格抽前 20 項：hashtag 名稱 / 貼文量 / 7 日成長率
3. 回傳 JSON

## 不要做
- Creative Center 需要登入 TikTok for Business（不同於 TikTok Studio）。若被導到登入頁 → 截圖 `logs/trends-tiktok-blocked.png` 並回傳 `{"trends": [], "note": "creative center login required"}`
- 不要走付費 Marketing API
- 縮寫（K/M/B）轉成純整數再輸出
