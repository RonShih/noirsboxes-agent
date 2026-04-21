---
description: 對應驗收 #8 — 巡 5 平台收近 7 天互動數據，產週報本地 Markdown 檔；best_times 只做人類可讀對照表、不動 schedule.yaml
---

步驟：

1. **併發**呼叫 5 個 stats skill：
   - `collect-stats-facebook`
   - `collect-stats-instagram`
   - `collect-stats-x`
   - `collect-stats-youtube`
   - `collect-stats-tiktok`

   每個回傳：7 天 posts + 粉絲變化 + top 3 + `best_times`（各自 Analytics 熱圖前 3 時段）

2. 彙整 stats 成 dict

3. 呼叫 `gdrive-reader` 讀 `data/calendar.json`，用來跟實際發文量對帳（例：上週排 9 篇、實際 published 7 篇、失敗 2 篇）

4. 呼叫 `report-writer`（輸入：stats + calendar dict），產 Markdown 週報，含：
   - 本週發文總覽（5 平台篇數 vs 計畫數）
   - 互動 Top 5（跨平台）
   - 對比 W-1 成長率
   - **3 條優化建議**（題材、發布時段、Hashtag）— 驗收 #8 要求
   - **「本週最佳時段 vs 目前排程」對照表**：
     ```
     | 平台 | 目前 schedule.yaml slots | 本週 Analytics best_times | 差異 |
     | FB  | Tue 10:00, Wed 14:00    | Tue 20:00, Wed 18:00     | ⚠️ |
     ```
     讓使用者**自行決定**要不要手動調 `schedule.yaml`。**不要自動覆寫。**

5. 呼叫 `gdrive-writer` 的 `write_report_markdown`：
   - 檔名：`reports/<YYYY-WWW>.md`（例如 `reports/2026-W17.md`）
   - 內容：完整 markdown

6. 呼叫 `gdrive-writer` 的 `append_calendar_rows` 把 stats 寫到 `data/stats-history.json`（首次執行會建立）
   - 結構：`[{ week: "2026-W17", platform: "facebook", posts: 3, total_reach: ..., top_post: {...}, best_times: [...] }]`

7. 輸出：本週週報檔路徑（例如 `/Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/reports/2026-W17.md`）

## 驗收檢查

- [ ] `reports/<YYYY-WWW>.md` 存在
- [ ] 內容含 5 平台互動數字 + 至少 3 條優化建議
- [ ] 內有「目前 slots vs best_times」對照表
- [ ] `data/stats-history.json` 新增本週列

## 不要做

- **不要動 `config/schedule.yaml`**。Tue+Wed 這類商業規則由人類掌控，best_times 只是建議
- 不要把週報寫到 Google Doc / Drive — **統一本地 markdown**
- 不要把 stats 只存在記憶中；必須寫進 `data/stats-history.json`
- 不要在週報內宣稱「已自動更新排程」— 沒做就是沒做
