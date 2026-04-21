# 系統層級改動紀錄（noirsboxes-agent 部署注意事項）

這份文件列出**不在專案 git repo 裡**、但對 routine 自動化**必要**的系統層級設定。交付廠商或換電腦時要照這份重建。

---

## 🔄 2026-04-21 大改：完全本地化

**專案不再依賴 Google Drive / Sheet / Doc / Apps Script**。所有資料改本地儲存：

| 舊（雲端）| 新（本地）|
|---|---|
| Google Sheet `content-calendar-2026` | `data/calendar.json` |
| Google Drive `media_folder_id` 圖片 | `media/assets/` |
| Google Doc 週報 | `reports/<YYYY-WWW>.md` |
| Apps Script endpoint（寫 Sheet）| 直接 read/write `data/calendar.json` |
| Drive MCP 下載檔 | 素材必須已在 `media/assets/` |

**好處**：
- 不用維護 Apps Script 部署（之前常壞）
- 不用 OAuth 反覆授權 Drive
- 速度快（本地 I/O vs 網路 API）
- 交付給廠商簡單（不用設 Google 帳號 / Apps Script）

**壞處**：
- 素材要手動管（從 Drive 下載到 `media/assets/` 後 commit 或分發）
- 多人協作要自己同步 `data/calendar.json`（用 git 或共用儲存）
- 換機器要把 `data/` + `media/assets/` 手動搬

**若要切回雲端版**：以前的 gdrive 設定保留在 `config/brand.yaml` 底下註解區。

---

## 1. 使用者層級 Claude Code 設定

### 檔案位置
```
~/.claude/settings.json
```

### 最小必要內容
```json
{
  "skipAutoPermissionPrompt": true,
  "skipDangerousModePermissionPrompt": true,
  "permissions": {
    "defaultMode": "bypassPermissions",
    "allow": [
      "Bash(curl *)",
      "Bash(cd *)",
      "Bash(ls *)",
      "Bash(mkdir *)",
      "Bash(rm *)",
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "mcp__playwright__*",
      "mcp__scheduled-tasks__*"
    ]
  }
}
```

### 為什麼需要這個
- Scheduled task 啟動時的 Claude session **CWD 不保證是專案目錄**
- 所以專案層 `.claude/settings.json` 不一定被讀到
- 權限放使用者層 = 任何 session（含 routine）都吃得到，不會中途卡權限提示

### 副作用
**影響這台電腦上所有 Claude Code 專案**。清單盡量收窄（只列 noirsboxes 實際用到的 tool），其他專案碰到未列工具還是會問。

### 改完要做的事
**重啟 Claude Code**（Cmd+Q → 重開），settings 不會熱 reload。

---

## 2. 專案層級 Claude Code 設定（備援）

### 檔案位置
```
.claude/settings.json                (commit 進 repo)
```

### 內容
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "-y",
        "@playwright/mcp@latest",
        "--user-data-dir",
        "/Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/browser_profiles"
      ]
    }
  },
  "permissions": {
    "defaultMode": "bypassPermissions",
    "allow": [
      "Read", "Write", "Edit", "Glob", "Grep", "Bash",
      "WebFetch", "WebSearch",
      "mcp__playwright__*", "mcp__*"
    ]
  }
}
```

### 要改的地方（換機器時）
- `--user-data-dir` 裡的絕對路徑要改成新機器上 `noirsboxes-agent/browser_profiles` 的實際位置

### 為什麼留著
當你**手動**在 Claude Code 內做事（CWD 在專案）時，這份 settings 會覆蓋使用者層，讓專案內操作更寬鬆。

---

## 3. Scheduled Tasks（Routine）

### 存放位置
```
~/.claude/scheduled-tasks/<taskId>/SKILL.md       ← Prompt（人類可編輯）
~/Library/Application Support/Claude/...          ← Cron 表達式、狀態（Claude Desktop 管，別手動改）
```

**關鍵**：scheduled tasks **不在專案 repo 裡**，要手動重建。

### 目前有的 tasks

| taskId | Cron / 觸發 | 用途 |
|---|---|---|
| `test-5min` | `*/5 * * * *` | **測試用** — 每 5 分跑 /publish-now（production 時該刪）|
| `calendar-test` | manual only | 手動觸發 /generate-calendar（產下週內容到 Sheet）|

### Production 時該有什麼（目前還沒建）

| taskId | Cron | 用途 |
|---|---|---|
| `publish-due` | `0 * * * *`（每小時）| routine 自動發文 |
| `generate-weekly-calendar` | `0 3 * * 0`（每週日 03:00）| 週日凌晨產下週日曆 |

### 重建 task 的步驟

1. Claude Code UI 點「Scheduled Tasks」→「Create」
2. 填 taskId、cron
3. Prompt 從 `routines/<taskId>.md`（**建議之後把 prompt 模板存進 repo**）複製貼上
4. 按一次「Run now」預先批准所有 MCP 工具（會彈權限提示、全按允許、會存到 task）

### Task prompt 要遵守的鐵則（已寫在現有 task prompt 裡）

- 工作目錄必須是 `/Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent`
- 不要在 worktree isolation 模式跑（Playwright profile 會是空的）
- 不要執行 `/first-time-login`（要人類互動）
- 不要因為過去失敗列的 notes 預測失敗就跳過
- 不寫 `logs/*.md`、不截圖、不 snapshot
- 發完 `browser_close`
- 沒 due 列就立刻 return

---

## 4. 瀏覽器 profile（登入狀態）

### 位置
```
<專案根>/browser_profiles/shared/Default/Cookies    ← 重要，登入全靠這個
```

### 建立方式
1. 專案裝好後第一次跑 `/first-time-login`
2. 手動登入 FB / IG / X / YouTube / TikTok 各一次
3. Cookies 寫進上面路徑

### 換機器時
- `browser_profiles/` **不進 git repo**（會超大、含隱私）→ 新機器要重跑 `/first-time-login`
- **別把兩台機器的 profile 互抄**（可能被平台判定盜用 cookie）

### 執行 routine 時
- Playwright Chromium 視窗必須**關著**，否則 scheduled task 起新 Chromium 會撞 profile lock

---

## 5. 本地資料儲存（取代 Drive / Apps Script）

### 目錄結構
```
<專案根>/
├── data/
│   ├── calendar.json         ← 內容日曆（取代 content-calendar-2026 Sheet）
│   └── stats-history.json    ← 每週 stats 歷史（/weekly-report 產出）
├── media/
│   └── assets/               ← 圖片、影片素材
└── reports/
    └── <YYYY-WWW>.md         ← 週報 markdown
```

### `data/calendar.json` schema
```json
{
  "_schema": { "fields": [...], "status_enum": [...], "platform_enum": [...] },
  "rows": [
    {
      "row_id": 1,
      "date": "2026-04-21",
      "time": "10:00",
      "platform": "facebook",
      "topic": "產品介紹",
      "caption": "...",
      "hashtags": ["#..."],
      "image_path": "media/assets/md905.jpg",
      "video_path": null,
      "status": "scheduled",
      "notes": "...",
      "post_url": null,
      "published_at": null
    }
  ]
}
```

### 換機器時
- Git push `data/`、`config/`、`.claude/`、`docs/` → 新機器 pull
- `media/assets/` **不進 git**（檔案可能很大）→ 分開用 rsync / zip / 雲端硬碟傳
- `browser_profiles/` 不進 git → 新機器要重跑 `/first-time-login`
- **不需要建 Apps Script、不需要 OAuth Drive**

### `config/secrets.yaml` 目前空的
本地化後這個檔基本沒作用（保留為未來 OpenAI API key 等用途的 placeholder）。

---

## 6. 常見坑（踩過的）

| 坑 | 現象 | 解法 |
|---|---|---|
| Scheduled task 在 worktree 裡跑 | 每次都是新 Chromium profile、cookies 不持久 → 全部平台卡登入 | Task 設定不要用 worktree isolation |
| FB 靜默封鎖自動發文 | 整個 UI 流程 OK、dialog 關了、但貼文不見 | FB 自動化偵測封鎖，非 bug |
| Routine 每次都問權限 | Claude session CWD 非專案、讀不到專案 settings | 把 permissions 放**使用者層** settings |
| Run now 批准只存該 task | 新 task 又要重批准 | 用使用者層 bypass 規則避免 |
| 改完 settings 沒生效 | Claude Code 啟動時只讀一次 | **Cmd+Q 重開 Claude Code** |
| 本地 calendar.json 並發寫入 | 兩個 session 同秒改檔 → 覆蓋 | gdrive-writer 用 `.tmp` + `mv` 原子寫入 |
| 素材不在 `media/assets/` | publish-* 回 `{ error: image not found }` | 先放素材、再排 calendar |

---

## 7. 交付 / 換機器 checklist

```
[ ] clone repo
[ ] 建 ~/.claude/settings.json（複製本文件 §1 內容）
[ ] 重啟 Claude Code
[ ] 確認 data/calendar.json 存在（若無先跑 /generate-calendar）
[ ] 把 media/assets/ 底下素材傳到新機器（rsync / zip）
[ ] 執行 /first-time-login 登入 5 平台
[ ] 關掉 Playwright Chromium 視窗
[ ] 手動建 scheduled tasks（publish-due / generate-weekly-calendar）
[ ] 對每個 task 按一次 Run now 預先批准權限
[ ] 等下一個 cron tick 驗證自動發文
```

**不再需要**：npm install、Apps Script、Drive MCP OAuth

---

## 8. 非系統層但容易忘的設定

- `config/schedule.yaml` 的 `cron.publish_window_minutes` 和 `early_window_minutes`（目前 `[slot, slot+10 min]`）
- 跟 task cron 頻率要搭配 — 5 分 cron 配 10 分窗口、1 小時 cron 配 10 分窗口（只有整點 slot 才會被命中）

---

## 9. 專案內 commands（slash commands）

路徑 `.claude/commands/*.md` — 手動在對話打 `/xxx` 或 scheduled task 呼叫。

| Command | 用途 | 觸發方式 |
|---|---|---|
| `/publish-now` | 讀 Sheet 找 due 列 → 依 platform 發到對應平台 → 更新 Sheet | scheduled task（`test-5min` / 未來 `publish-due`）|
| `/generate-calendar` | 依 `schedule.yaml` 產下週排程 → 寫進 Sheet | scheduled task `calendar-test`（手動）|
| `/first-time-login` | Playwright 打開 5 平台等人類登入 → 存 cookies 到 `browser_profiles/` | **只能人類互動**、不能 schedule |
| `/test-post` | 5 平台測試發文（驗收用） | 手動 |
| `/upload-youtube` | 長片手動上傳 YouTube（驗收 #7） | 手動 |
| `/weekly-report` | 產週報 Google Doc（不寫回 schedule.yaml）| 手動 |
| `/analyze-hotspots` | 並行跑 5 個 fetch-trends-* 算熱點 → 排序 | 手動 |

---

## 10. 專案內 skills

路徑 `.claude/skills/<name>/SKILL.md` — command 呼叫或 Claude 自己 call。不會被人類直接 slash-invoke。

### 發文類（5 個，對應 5 平台）
| Skill | 輸入 | 輸出 | 被誰 call |
|---|---|---|---|
| `publish-facebook` | caption / hashtags / image_url | `{post_url}` | `/publish-now` |
| `publish-instagram` | 同上 | 同上 | 同上 |
| `publish-x` | caption / image_url（選填）| 同上 | 同上 |
| `publish-youtube` | title / description / tags / local_video_path | `{video_url}` | 同上 |
| `publish-tiktok` | caption / hashtags / local_video_path | `{video_url}` | 同上 |

**共通鐵則**（已寫在各 SKILL.md）：
- 不 screenshot / snapshot 存檔
- 不因「歷史失敗」就跳過、不換順序
- 失敗回 `{error}`，不重試

### 內容產製類
| Skill | 用途 | 被誰 call |
|---|---|---|
| `content-writer` | 產 caption + hashtags（依平台風格）| `/generate-calendar` |
| `image-generator` | 產配圖 / 選 Drive 素材 | `/generate-calendar` |
| `seo-metadata` | 產 YT title / description / tags | `/upload-youtube` |

### 資料存取類
| Skill | 用途 | 實作 |
|---|---|---|
| `gdrive-reader` | **名字沿用、已本地化** — 讀 `data/calendar.json` + `media/assets/` | 本地檔 I/O |
| `gdrive-writer` | **名字沿用、已本地化** — 寫 `data/calendar.json` + `reports/*.md` | 本地檔 I/O（jq 或 Edit tool）|

### 熱點分析類（5 個，對應 5 平台）
| Skill | 用途 | 被誰 call |
|---|---|---|
| `fetch-trends-facebook` | 抓 FB 熱門主題 | `/analyze-hotspots`（並行）|
| `fetch-trends-instagram` | IG 熱門 | 同上 |
| `fetch-trends-x` | X 熱門 | 同上 |
| `fetch-trends-youtube` | YT 熱門 | 同上 |
| `fetch-trends-tiktok` | TT 熱門 | 同上 |

### 數據收集類（5 個）
| Skill | 用途 | 被誰 call |
|---|---|---|
| `collect-stats-facebook` | 收 FB 過去 7 天每篇 reach / engagement / best-time | `/weekly-report` |
| `collect-stats-instagram` | IG 同上 | 同上 |
| `collect-stats-x` | X 同上 | 同上 |
| `collect-stats-youtube` | YT 同上 | 同上 |
| `collect-stats-tiktok` | TT 同上 | 同上 |

### 報告類
| Skill | 用途 | 被誰 call |
|---|---|---|
| `report-writer` | 產週報 markdown 字串（caller 存本地 `reports/`）| `/weekly-report` |

---

## 11. 專案內 config 檔

| 檔案 | 用途 | Git | 重要性 |
|---|---|---|---|
| `config/brand.yaml` | 品牌資訊、5 平台 handle / URL、`storage` 區塊定義本地路徑 | commit | ★★★ |
| `config/products.yaml` | 產品線（MD-905 等）、給 content-writer 抽主題 | commit | ★★ |
| `config/schedule.yaml` | 每週發文時段 slots + cron 窗口設定 | commit | ★★★ |
| `config/secrets.yaml` | **本地化後基本無用**（保留給未來 OpenAI key）| **gitignore** | ★ |

---

## 12. 整個 dataflow（本地版）

```
週日 03:00            /generate-calendar (手動或未來 cron)
                      ├─ 讀 config/schedule.yaml slots
                      ├─ 讀 config/brand.yaml + products.yaml
                      ├─ call content-writer 產 caption
                      ├─ call image-generator 選/產素材 → media/assets/
                      └─ call gdrive-writer.append_calendar_rows
                          → 寫進 data/calendar.json
                                  ↓
data/calendar.json（本地排程資料源）
  rows[] 每筆 = 一篇貼文：date / time / platform / caption / hashtags
                         / image_path / video_path / status
                                  ↓
每小時                   scheduled task (publish-due) → /publish-now
                         ├─ Step 0: gdrive-reader.filter_due_rows
                         │            讀 data/calendar.json、篩 due 列
                         │  → 0 列就 return
                         ├─ Step 2: 依 platform 分發
                         │  └─ publish-<platform> skill 發文
                         │     （直接傳 local_image_path/local_video_path）
                         ├─ Step 3: gdrive-writer.update_calendar_row
                         │            status=published、post_url=...
                         ├─ Step 4: 失敗 status=failed + notes
                         └─ Step 5: browser_close
                                  ↓
                         社群平台（FB / IG / X / YT / TT）出貼文
                                  ↓
週日夜                  /weekly-report (手動)
                         ├─ call collect-stats-* ×5（並行）
                         ├─ call gdrive-reader 讀 data/calendar.json 對帳
                         ├─ call report-writer 產 markdown 字串
                         └─ call gdrive-writer.write_report_markdown
                             → reports/<YYYY-WWW>.md
                         另外 append stats 到 data/stats-history.json
```

**完全本地 — 無任何外部 API 依賴（除了 5 個社群平台本身）。**
