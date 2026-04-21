---
name: fetch-trends-x
description: 用 Playwright MCP 爬 X 首頁右側「流行趨勢」sidebar，回傳台灣 trending topic 與推文量
---

## Output
```json
{
  "platform": "x",
  "fetched_at": "2026-04-20T14:00:00+08:00",
  "region": "TW",
  "trends": [
    { "topic": "Arsenal", "category": "台灣 的流行趨勢", "volume": null, "rank": 1 }
  ]
}
```

## 流程
1. `browser_navigate` → `https://x.com/home`（需先登入）
2. 右側 complementary「流行趨勢」region 抽前 10 個 trend
3. 每個 trend 抓 topic（`#xxx` 或純文字）、category（「台灣 的流行趨勢」之類）、volume
   - 若「N 萬則貼文」/「N 千則貼文」→ 轉成整數；找不到 → null
4. 回傳 JSON

## 不要做
- 不要呼叫 X API
- 不要過濾主題 — 相關性過濾由 caller `/analyze-hotspots` 負責
- 不要寫入 Drive
