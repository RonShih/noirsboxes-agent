---
name: report-writer
description: 把多平台 stats + 本地 calendar 整合成 Markdown 週報，含 3 條優化建議
---

## Input
- `stats`：list of stats objects（FB / IG / X / YT / TikTok）
- `calendar`：`data/calendar.json` 的完整內容（dict with `rows[]`）
- `week_label`：例如 `"2026-W17"`

## Output
```json
{ "markdown": "# NoirsBoxes 社群週報 2026-W17\n...", "filename": "2026-W17.md" }
```

（`markdown` 是完整字串；caller 再交給 `gdrive-writer` 的 `write_report_markdown` 存 `reports/<filename>`）

## 報告結構（強制）
1. **本週概況**：各平台發文數（計畫 vs 實際 published / failed）、總互動、粉絲數變化
2. **互動 Top 5**（跨平台合併排序）
3. **與上週對比**：成長率表格
4. **內容類型分析**：哪類主題（產品介紹 / 技術知識 / ...）互動最高
5. **3 條優化建議**：每條格式
   - 觀察：基於數據的事實
   - 建議：具體可執行的下週動作
   - 預期效益
6. **「目前排程 vs best_times」對照表**（來自 stats.best_times）— **不要自動改 schedule.yaml，讓人決定**

## 規則
- 必讀 `config/brand.yaml` 確保語氣一致
- 數字用「,」千分位
- 不要編造沒有的數據 — 缺值寫「N/A」
- 結尾附「資料抓取時間」

## 不要做
- 不要呼叫其他 skill，純資料處理 + LLM 摘要
- **不要輸出到 Google Doc / Drive** — 純 markdown 字串回給 caller，caller 存本地
