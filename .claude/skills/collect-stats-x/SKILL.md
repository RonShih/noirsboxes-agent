---
name: collect-stats-x
description: 用 Playwright MCP 進入 X（Twitter）個人資料頁，擷取近 7 天貼文互動數據
---

## Input
- `days`：預設 7

## Output
```json
{
  "platform": "x",
  "followers": 0,
  "follower_delta_7d": 0,
  "posts": [
    { "url": "...", "published_at": "2026-04-15", "views": 0, "likes": 0, "reposts": 0, "replies": 0, "bookmarks": 0 }
  ],
  "top_3": ["url1", "url2", "url3"],
  "best_times": [
    { "day": "wed", "hour": 15, "score": 0.74 }
  ]
}
```

`best_times` 欄位 schema 同 `collect-stats-facebook`。

## 流程
1. `browser_navigate` → `https://x.com/NoirsBoxes`（帳號見 `config/brand.yaml` socials.x.handle）
2. 從 profile header 抓粉絲數（「追隨者」數字）
3. 在貼文列表往下捲（`browser_press_key` PageDown 數次）直到看到 `days` 天前的貼文
4. 逐則貼文抽以下欄位（每則文章 `article` 底下的 group：「N 則回覆、M 次轉發、K 個喜歡、J 個書籤、V 次觀看」）：
   - `url`：`/url` 屬性 `/{handle}/status/{id}`
   - `published_at`：`time` 元素
   - `views` / `likes` / `reposts` / `replies` / `bookmarks`
5. 若需要更精細數據（例如 profile visit / impressions 分母），對每則點 `/status/{id}/analytics`（僅本人可看）
6. 過濾超過 `days` 天前的貼文
7. 按 `views` 排序取 top 3
8. **計算 `best_times`（X 沒 Premium 沒官方熱圖，走自建 fallback）：**
   - 若 Sheet 的 `stats_history` tab 有 ≥ 20 筆歷史貼文（跨 ≥ 4 週累積，由 `/weekly-report` 寫入）→ 本次也把當週 posts append 進去，然後對所有歷史貼文分組 `(day_of_week, hour)`、取平均 views，score 歸一化後取前 3
   - 若不足 20 筆 → 回傳 `[]`，不要用當週 7 則貼文硬算（樣本太小失真）
9. 回傳 JSON

## 不要做
- 不要嘗試呼叫 X API（沒 token 會失敗）
- 不要把資料寫入 Drive — 由 caller 交給 report-writer
- 數字有「千/萬/億」這類中文縮寫（例：「3.6萬」）時，必須先轉成純整數再輸出
