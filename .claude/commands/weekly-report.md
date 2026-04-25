---
description: 巡 5 平台收近 7 天互動數據，產週報本地 Markdown 檔
---

## 步驟

1. **併發**呼叫 5 個 stats skill：
   - `collect-stats-facebook`
   - `collect-stats-instagram`
   - `collect-stats-x`
   - `collect-stats-youtube`
   - `collect-stats-tiktok`

   每個回傳：7 天 posts + 粉絲變化 + top 3 + `best_times`（各自 Analytics 熱圖前 3 時段）

2. 彙整 stats 成 dict

3. 呼叫 `report-writer`（輸入：stats），產 Markdown 週報，含：
   - 本週各平台發文數 + 總互動 + 粉絲數變化
   - 互動 Top 5（跨平台合併排序）
   - 對比 W-1 成長率
   - 3 條優化建議（題材、發布時段、Hashtag）
   - 各平台 Analytics best_times 對照表（純資訊，不自動調整任何排程）

4. 呼叫 `local-writer.write_report_markdown` 寫 `reports/<YYYY-WWW>.md`

5. 呼叫 `local-writer.append_stats_history` 把 stats append 到 `data/stats-history.json`（首次自動建檔）

6. 輸出：週報絕對路徑

## 驗收

- [ ] `reports/<YYYY-WWW>.md` 存在
- [ ] 內容含 5 平台互動數字 + 至少 3 條優化建議
- [ ] `data/stats-history.json` 新增本週列
