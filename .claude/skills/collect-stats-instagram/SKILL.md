---
name: collect-stats-instagram
description: 用 Playwright MCP 進入 Instagram Insights，擷取近 7 天互動數據
---

## Input
- `days`：預設 7

## Output
同 `collect-stats-facebook` 結構（含 `best_times`），`platform="instagram"`。

## 流程
1. `browser_navigate` → `https://www.instagram.com/black_magic_noirsboxes/`
2. 點「Professional dashboard」
3. 點「View insights」→ 切到「Last 7 days」
4. 從 Content 表格逐列抽 post URL / Reach / Likes / Comments / Shares / Saves
5. 從首頁 header 取 Followers 數
6. 排序 Top 3
7. 切到「觀眾 / Audience → Most Active Times」熱圖，抽前 3 高分時段寫入 `best_times`
   - 若是新帳號、IG 顯示「資料不足」→ 回傳空 `[]`
8. 回傳 JSON
