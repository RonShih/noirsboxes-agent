---
name: local-writer
description: 寫本地週報 markdown、append stats history、儲存產生的素材。
---

## 三種模式

### 1. write_report_markdown

**用途**：給 `/weekly-report` 用，寫週報到 `reports/<filename>`。

**Input**：
- `filename`：例如 `2026-W17.md`
- `markdown`：完整字串

**實作**：直接 Write tool 寫 `reports/<filename>`（會自動 mkdir reports/）

**回傳**：`{ "path": "reports/2026-W17.md" }`

### 2. append_stats_history

**用途**：把每週 stats 累積到 `data/stats-history.json`（首次自動建檔）。

**Input**：`entries`，陣列，例如：
```json
[
  { "week": "2026-W17", "platform": "facebook", "posts": 3, "total_reach": 1234, "top_post": {...}, "best_times": [...] },
  ...
]
```

**實作**：
1. 若 `data/stats-history.json` 不存在 → 建空骨架 `{ "weeks": [] }`
2. append entries 到 `weeks` 陣列
3. 原子寫入：`.tmp` + `mv`

**回傳**：`{ "appended": N }`

### 3. save_asset

**用途**：把產生的圖 / 影片放進 `media/assets/`。

**Input**：
- `name`：檔名（例如 `md905-epr.jpg`）
- 來源：`base64` 或 `source_url`（任一）

**實作**：
- base64 → `echo <base64> | base64 -D > media/assets/<name>`
- URL → `curl -sL -o media/assets/<name> <url>`

**回傳**：`{ "path": "media/assets/<name>" }`

## 失敗處理

- 並發寫入用 `.tmp` + `mv` 原子寫入
- 磁碟不足 → 回 `{ "error": "disk full" }`
- 不丟例外、結構化回傳
