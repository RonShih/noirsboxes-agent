---
name: collect-stats-facebook
description: 用 Playwright MCP 進入 Facebook 粉專洞察頁，擷取近 7 天互動數據
---

## Input
- `days`：回看天數（預設 7）

## Output
```json
{
  "platform": "facebook",
  "page_followers": 1234,
  "follower_delta_7d": 12,
  "posts": [
    { "url": "...", "published_at": "2026-04-15", "reach": 0, "likes": 0, "comments": 0, "shares": 0 }
  ],
  "top_3": ["url1", "url2", "url3"],
  "best_times": [
    { "day": "tue", "hour": 20, "score": 0.82 },
    { "day": "thu", "hour": 19, "score": 0.78 },
    { "day": "sat", "hour": 14, "score": 0.71 }
  ]
}
```

`best_times` 欄位 schema（所有 `collect-stats-*` skill 共用）：`day` 用小寫 `mon|tue|...|sun`、`hour` 用 0–23、`score` 歸一化到 0–1。

## 流程
1. `browser_navigate` → `https://www.facebook.com/BlackMagicNoirsBoxes/insights/`
   - 若找不到 insights 入口，改 navigate 到粉專首頁，從左側選「專業主控板」→「洞察報告」
2. 切換時間範圍到「過去 7 天」
3. 用 `browser_snapshot` 截 DOM，從互動表格逐列抽：post URL / 觸及 / 讚 / 留言 / 分享
4. 從「總覽」抓粉絲數與成長
5. 排序找出互動數 Top 3
6. 到「互動 → Best Times / 最佳時段」熱圖，抽前 3 高分時段寫入 `best_times`（score 以熱圖色階正規化）
   - 若 FB 該月沒累積足夠數據、熱圖不顯示 → 回傳空 `[]`
7. 整理成 JSON 回傳

## 不要做
- 不要嘗試呼叫 Graph API
- 不要嘗試把資料寫入 Drive — 由 caller 統一交給 report-writer
