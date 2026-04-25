# NoirsBoxes Social Agent

NoirsBoxes 社群小編 agent。透過 Telegram 對 Claude 下指令，由 Claude 用 Playwright MCP 自動發文到 5 平台（FB / IG / X / YouTube / TikTok）。

## 設計

- 不接平台官方 API：所有發文 / 上片 / 抓數據 → Playwright MCP 操作真瀏覽器
- 儲存全部本地：素材 `media/assets/`、週報 `reports/`、stats `data/stats-history.json`
- 執行入口 = 5 個 slash command + 21 個 skill，全部用 markdown 編排

## 開始使用

完整設定步驟、日常 workflow、故障排除：[`docs/HOW_TO_USE.md`](./docs/HOW_TO_USE.md)

最簡形式：

```bash
cd noirsboxes-agent
claude --channels plugin:telegram@claude-plugins-official
```

首次必做：登入 5 平台、安裝 telegram channel plugin、用 user-level 加 Playwright MCP — 全部步驟在 HOW_TO_USE。

## 目錄結構

```
noirsboxes-agent/
├── CLAUDE.md                   # 身分、品牌語調、共通行為規則
├── README.md
├── .gitignore
│
├── .claude/
│   ├── settings.json           # MCP 設定 + permissions.allow
│   ├── commands/               # 5 個 slash command
│   └── skills/                 # 21 個 skill（SKILL.md）
│
├── config/
│   ├── brand.yaml              # 品牌、社群帳號、本地儲存路徑
│   ├── products.yaml           # 產品清單
│   └── secrets.yaml            # 🔒 gitignored；預留 placeholder
│
├── media/assets/               # 🟡 gitignored；圖 / 影素材
├── reports/                    # 週報 markdown
├── browser_profiles/           # 🔒 gitignored；Chromium 登入狀態
│
└── docs/
    ├── HOW_TO_USE.md           # 使用手冊
    ├── SYSTEM_CHANGES.md       # 系統層級設定 / 交付 checklist
    └── WARNINGS.md             # 已知問題 / 限制
```

## 換客戶 / 重用

- 換品牌 → 改 `config/brand.yaml` + `CLAUDE.md`
- 換產品 → 改 `config/products.yaml`
- 平台 UI 改版 → 改對應 `publish-*/SKILL.md`
