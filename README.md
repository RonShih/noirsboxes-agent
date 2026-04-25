# NoirsBoxes Social Agent

對應 `NoirsBoxes_requirement.md` **T2 — 自主完成小編工作** 的最小可行交付。

驗收條目（出自 PRD §3.4）：
- **#5** 自動產生一週內容日曆（含文案 + 配圖 + Hashtag）
- **#6** 指定時間自動發布至 Facebook 與 Instagram
- **#7** T1 影片自動上傳 YouTube，含完整 Metadata
- **#8** 每週自動產生社群運營報告，含互動數據與優化建議

---

## 設計

- **不接平台官方 API**：所有發文 / 上片 / 抓數據 → Playwright MCP 操作真瀏覽器
- **儲存全部本地**（2026-04-21 起）：
  - 內容日曆 = `data/calendar.json`
  - 素材（圖 / 影）= `media/assets/`
  - 週報 = `reports/<YYYY-WWW>.md`
  - 每週 stats = `data/stats-history.json`
- **執行入口 = 7 個 slash command**（詳見 `docs/SYSTEM_CHANGES.md` §9）
- **Skill 是單一能力**（21 個），command 編排 skill。換客戶或換平台只改 markdown

### 為什麼完全本地（不接 Google Drive / Sheet）
- Apps Script 端點會壞、OAuth 過期要重接、Drive MCP 下載大檔會爆 context
- 本地 I/O 快、穩、廠商交付簡單（不用設 Google 帳號）
- 代價：素材要手動傳（rsync / zip），不適合多人雲端協作

---

## 前置需求

```bash
node --version      # >= 18
claude --version    # Claude Code 已安裝
```

必要 MCP：

- **Playwright MCP** — `npx @playwright/mcp@latest`，`.claude/settings.json` 已預設 `--user-data-dir`

Drive / Apps Script / Doc 的 MCP **不再需要**。

---

## 驗收流程

```bash
cd noirsboxes-agent
claude
```

進 Claude Code 後依序執行：

1. `/first-time-login` — 人工登入 FB / IG / X / YT / TikTok 各一次（cookie 存 `browser_profiles/`）
2. 編輯 `config/brand.yaml`：填品牌資訊與社群帳號
3. 放素材到 `media/assets/`（圖、影片）
4. `/generate-calendar` — 對應驗收 **#5**（產下週 13 列到 `data/calendar.json`）
5. `/publish-now` — 對應驗收 **#6**（讀 calendar、發到 5 平台）
6. `/upload-youtube` — 對應驗收 **#7**（單次手動上傳長片）
7. `/weekly-report` — 對應驗收 **#8**（產 `reports/<YYYY-WWW>.md`）

---

## 目錄結構

```
noirsboxes-agent/
├── CLAUDE.md                  # 品牌語調 / 工作原則
├── README.md
├── .gitignore
│
├── .claude/
│   ├── settings.json          # MCP 設定 + permissions.allow
│   ├── commands/              # 7 個 slash command
│   └── skills/                # 21 個 skill（SKILL.md）
│
├── config/
│   ├── brand.yaml             # 品牌資產 + 本地儲存路徑
│   ├── products.yaml          # 產品清單
│   ├── schedule.yaml          # 每週發文 slots + cron 窗口
│   └── secrets.yaml           # 🔒 gitignored；本地化後基本無用
│
├── data/                      # 🟢 本地資料源（取代 Google Sheet）
│   ├── calendar.json          # 內容日曆
│   └── stats-history.json     # 每週 stats（gitignored）
│
├── media/
│   └── assets/                # 🟡 gitignored；圖 / 影素材（另外用 rsync 傳）
│
├── reports/                   # 週報 markdown 檔
│
├── browser_profiles/          # 🔒 gitignored；Playwright Chromium 登入狀態
│
└── docs/
    └── SYSTEM_CHANGES.md      # 系統層級改動 + 交付 checklist
```

---

## 觸發方式

主要走 **Telegram + [Claude Channels](https://code.claude.com/docs/en/channels)**（Anthropic 官方）：

- 在 TG 群組對 Claude 對話下指令（`/publish-now`、`/weekly-report` 等）
- 同個 chat 累積 context，可以追問「FB 為什麼失敗」、「再試一次」
- TG 上傳的圖會自動下載到 `~/.claude/channels/telegram/inbox/`，Claude 直接拿來發文

設定流程：見 [`docs/HOW_TO_USE.md`](./docs/HOW_TO_USE.md) 的「Telegram Channel 設定」章節。

### 環境需求

- Claude Code v2.1.80+ + claude.ai 登入（非 API key）
- 必須一直開著 `claude --channels ...` 終端機
- Playwright MCP 必須裝在 user-level（`claude mcp add playwright -s user ...`）

---

## 換客戶 / 重用

- 換品牌 → 改 `config/brand.yaml` + `CLAUDE.md`
- 換產品 → 改 `config/products.yaml`
- 平台 UI 改版 → 改對應 `publish-*/SKILL.md` 的步驟描述
- 換儲存（Notion / Airtable）→ 只改 `gdrive-reader` / `gdrive-writer` 兩個 skill

---

## 故障排除

詳見 [`docs/SYSTEM_CHANGES.md`](./docs/SYSTEM_CHANGES.md) §6「常見坑」。
