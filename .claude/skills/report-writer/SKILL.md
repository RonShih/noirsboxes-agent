---
name: report-writer
description: 把多平台 stats 整合成 Markdown 週報，含 3 條優化建議。
---

## Input
- `stats`：list of stats objects（FB / IG / X / YT / TikTok）
- `week_label`：例如 `"2026-W17"`

## Output
```json
{ "markdown": "# NoirsBoxes 社群週報 2026-W17\n...", "filename": "2026-W17.md" }
```

（`markdown` 是完整字串；caller 再交給 `local-writer.write_report_markdown` 存 `reports/<filename>`）

## 報告結構（強制）
1. **本週概況**：各平台發文數、總互動、粉絲數變化
2. **互動 Top 5**（跨平台合併排序）
3. **與上週對比**：成長率表格
4. **內容類型分析**：哪類主題互動最高
5. **3 條優化建議**：每條 = 觀察 / 建議 / 預期效益
6. **best_times 對照表**（純資訊，不調整任何排程）

## 規則
- 必讀 `config/brand.yaml` 確保語氣一致
- 數字用「,」千分位
- 缺值寫「N/A」、不編造
- 結尾附「資料抓取時間」

## 不要做
- 不要呼叫其他 skill — 純資料處理 + LLM 摘要
- 不要輸出到外部服務 — 純 markdown 字串回 caller
