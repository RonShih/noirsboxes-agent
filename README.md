# NoirsBoxes Social Agent

NoirsBoxes 社群小編 agent。透過 Telegram 對 Claude 下指令，由 Claude 用 Playwright MCP 自動發文到 5 平台（FB / IG / X / YouTube / TikTok）。

---

## 設計

- 不接平台官方 API：所有發文 / 上片 / 抓數據 → Playwright MCP 操作真瀏覽器
- 儲存全部本地：素材 `media/assets/`、週報 `reports/`、stats `data/stats-history.json`
- 執行入口 = 5 個 slash command + 21 個 skill，全部用 markdown 編排

---

## 前置需求

```bash
node --version      # >= 18
claude --version    # >= v2.1.80
bun --version       # 任何版本（給 channel plugin 用）
```

必要 MCP：

- **Playwright MCP** — `claude mcp add playwright -s user -- npx -y @playwright/mcp@latest --user-data-dir <絕對路徑>/browser_profiles`

---

## 啟動流程

```bash
cd noirsboxes-agent
claude --channels plugin:telegram@claude-plugins-official
```

首次需要：

1. `/first-time-login` — 人工登入 FB / IG / X / YT / TikTok（cookie 存 `browser_profiles/`）
2. 編輯 `config/brand.yaml`：填品牌資訊與社群帳號
3. 放素材到 `media/assets/`，或在 TG 上傳由 bot 自動接收

之後在 TG 對 bot 下指令：
- 直接傳「發 FB，caption: ...」+ 圖
- `/publish-now` — 給定平台 + 內容、Claude 直接發
- `/weekly-report` — 產週報
- `/analyze-hotspots` — 5 平台熱點分析

---

## 目錄結構

```
noirsboxes-agent/
├── CLAUDE.md                   # 品牌語調 / 工作原則
├── README.md
├── .gitignore
│
├── .claude/
│   ├── settings.json           # MCP 設定 + permissions.allow
│   ├── AGENT_RULES.md          # 全域行為規則（任何任務先讀這個）
│   ├── commands/               # 5 個 slash command
│   └── skills/                  # 21 個 skill（SKILL.md）
│
├── config/
│   ├── brand.yaml              # 品牌資產 + 本地儲存路徑
│   ├── products.yaml           # 產品清單
│   └── secrets.yaml            # 🔒 gitignored；本地化後基本無用
│
├── media/
│   └── assets/                 # 🟡 gitignored；圖 / 影素材（另用 rsync 傳）
│
├── reports/                    # 週報 markdown 檔
│
├── browser_profiles/           # 🔒 gitignored；Playwright Chromium 登入狀態
│
└── docs/
    ├── HOW_TO_USE.md           # 使用手冊
    ├── SYSTEM_CHANGES.md       # 系統層級改動 + 交付 checklist
    └── WARNINGS.md             # 已知問題 / 限制
```

---

## 換客戶 / 重用

- 換品牌 → 改 `config/brand.yaml` + `CLAUDE.md`
- 換產品 → 改 `config/products.yaml`
- 平台 UI 改版 → 改對應 `publish-*/SKILL.md` 的步驟描述

---

## 故障排除

詳見 [`docs/SYSTEM_CHANGES.md`](./docs/SYSTEM_CHANGES.md) §6「常見坑」。
