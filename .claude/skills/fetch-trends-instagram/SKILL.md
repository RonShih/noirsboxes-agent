---
name: fetch-trends-instagram
description: IG 沒有官方 trending，改爬品牌相關 hashtag 的「熱門」tab，回傳最近爆文
---

## Input
- `seed_hashtags`：預設讀 `config/products.yaml` 的核心關鍵字。若缺，fallback 成 `["快充", "PD", "USB-C", "充電器", "GaN"]`

## Output
```json
{
  "platform": "instagram",
  "fetched_at": "2026-04-20T14:00:00+08:00",
  "trends": [
    { "hashtag": "#快充", "topic": "貼文短摘要或 alt-text", "url": "https://www.instagram.com/p/...", "likes": 0, "rank": 1 }
  ]
}
```

## 流程
1. 對每個 seed hashtag：
   - `browser_navigate` → `https://www.instagram.com/explore/tags/{tag}/`
   - 切到「熱門」tab，取前 9 張貼文
   - 每張抽：貼文 URL / likes（若 IG 已隱藏則 null）/ 短摘要（alt-text 或 caption 前 50 字）
2. 合併所有 hashtag 結果、去重（by URL）
3. 按 likes 降序取前 20（likes 為 null 的排後面）

## 不要做
- 每個 hashtag 最多等 3 秒載入，不 scroll 超過 2 屏（避免 IG rate limit）
- 不要嘗試 IG Graph API
- 不要寫入 Drive
