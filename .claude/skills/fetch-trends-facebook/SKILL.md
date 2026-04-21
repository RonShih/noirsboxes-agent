---
name: fetch-trends-facebook
description: FB 沒有公開 trending，改用 Google Trends Taiwan Daily 當代理訊號
---

## Output
```json
{
  "platform": "facebook-proxy(google-trends)",
  "fetched_at": "2026-04-20T14:00:00+08:00",
  "region": "TW",
  "trends": [
    { "topic": "關鍵字", "volume_est": "200K+", "rank": 1 }
  ]
}
```

## 流程
1. `browser_navigate` → `https://trends.google.com/trending?geo=TW`
2. 抽「每日熱門搜尋」列表前 20 個：關鍵字 + 搜尋量估計（保留原字串如「200+」「1,000+」即可）
3. 回傳 JSON；caller 把這個當 FB 的代理訊號

## 不要做
- 不要用 Google Trends 官方 API（需 OAuth）
- 若 UI 改版找不到元素 → 回傳 `{"trends": [], "note": "Google Trends UI changed"}`，不要 throw
- 不要寫入 Drive
