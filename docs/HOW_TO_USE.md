# 如何使用這個專案（noirsboxes-agent）

給接手的人 / 廠商看的實戰手冊。讀完可以馬上上手。

---

## 這個專案是什麼

一套自動化社群小編，用 Playwright 開真瀏覽器替你發文到 5 平台（FB / IG / X / YouTube / TikTok），內容日曆和素材全部存本地、不碰 Google Drive。

## 三種使用模式

| 模式 | 時機 | 誰觸發 |
|---|---|---|
| **手動模式** | 第一次設定、測試、突發狀況 | 你在 Claude Code 對話裡打 slash command |
| **Telegram 模式（Claude Channels）** | 出門用手機操作、想對話式 debug | 你在 TG 群組打指令給 bot，bot 把訊息推進你的 Claude session |
| **自動模式（routine）** | 日常營運、無人值守 | Claude Code scheduled task 按 cron 自動跑 |

---

## 初次設定（10 分鐘）

### 0. 先確認環境

```bash
node --version   # >= 18（npx 會用到）
npm --version    # 可以跑就好
claude --version # Claude Code 已安裝
```

沒裝 Node：`brew install node`（macOS）或去 https://nodejs.org。

### 1. 安裝 Claude Code
從 https://claude.com/claude-code 下載 Desktop 版、登入。

### 2. 設 `~/.claude/settings.json`（使用者層權限）

打開 `~/.claude/settings.json`，貼進：

```json
{
  "extraKnownMarketplaces": {
    "claude-plugins-official": {
      "source": { "source": "github", "repo": "anthropics/claude-plugins-official" }
    }
  },
  "skipAutoPermissionPrompt": true,
  "skipDangerousModePermissionPrompt": true,
  "permissions": {
    "defaultMode": "bypassPermissions",
    "allow": [
      "Bash(curl *)", "Bash(cd *)", "Bash(ls *)", "Bash(mkdir *)", "Bash(rm *)",
      "Read", "Write", "Edit", "Glob", "Grep",
      "mcp__playwright__*",
      "mcp__scheduled-tasks__*"
    ]
  }
}
```

**Cmd+Q 重啟 Claude Code**（settings 只在啟動時載入）。

### 3. Clone 專案
```bash
cd ~/Desktop
git clone https://github.com/RonShih/noirsboxes-agent
cd noirsboxes-agent
ls config/       # 應該看到 brand / products / schedule.yaml
```

**不用跑 `npm install`** — 專案沒有 `package.json`，所有 Playwright 相關套件靠 `npx` 按需下載。

### 4. 首次啟動：讓 Playwright 自己裝好

第一次在 Claude Code 裡呼叫任何 Playwright 指令（例如 `/first-time-login`）時，會發生：

```
Claude Code 讀 .claude/settings.json
  ↓
啟動 npx -y @playwright/mcp@latest
  ↓
npm 從 registry 下載 @playwright/mcp          ← 約 30 秒、存到 ~/.npm/_npx/
  ↓
Playwright 檢查要不要下載 Chromium 執行檔
  ↓
若沒有 → 自動下載 Chromium（~150 MB）         ← 1-3 分鐘、存到 ~/Library/Caches/ms-playwright/
  ↓
開瀏覽器
```

**全程自動**，你不用手動 npm install 或下載 Chromium。

若卡在下載或出錯，可以手動加速一次：
```bash
npx -y @playwright/mcp@latest --help    # 預先抓 mcp 套件
npx playwright install chromium          # 預先抓 Chromium
```

### 4b. 若 `.claude/settings.json` 的 MCP 設定不生效（備援）

有些環境 / Claude Code 版本**不會自動 spawn** 專案層 settings.json 裡的 MCP server（特別是用 CLI 而非 Desktop app 時）。若打開 Claude Code 之後 `/first-time-login` 找不到 `mcp__playwright__*` 工具，改用 CLI **手動註冊**：

```bash
cd /Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent

claude mcp add playwright \
  -s project \
  -- npx -y @playwright/mcp@latest \
  --user-data-dir /Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/browser_profiles
```

選項說明：
- `-s project` — 註冊在專案層（**別人 clone 也吃這份**，存在 `.mcp.json`）
- `-s user` — 只註冊在你這台電腦
- `-s local`（預設）— 只這個目錄、不同步

**絕對路徑不能省略**`--user-data-dir`，否則 scheduled task 的 Chromium profile 會錯亂（踩過的坑）。

註冊完確認：
```bash
claude mcp list
# 應看到 playwright: npx -y @playwright/mcp@latest ... - ✓ Connected
```

若要移除重裝：
```bash
claude mcp remove playwright -s project
```

### 5. 首次登入 5 平台
在 Claude Code 裡打：
```
/first-time-login
```

Playwright 會開 Chromium（首次可能等 1-3 分鐘下載）→ 依序引導你登入 FB / IG / X / YT / TikTok。登完 cookies 寫進 `browser_profiles/shared/`，之後 routine 靠這個登入狀態。

**登完請關掉 Chromium 視窗**（避免 profile lock）。

### 6. 準備素材
把圖片、影片放到 `media/assets/`：
```
media/assets/
├── md905.jpg          ← MD-905 產品照
├── md903-fake-cable.jpg
├── md905-short.mp4    ← 短影音
└── ...
```

檔名對應 `data/calendar.json` 裡 row 的 `image_path` / `video_path`。

---

## 日常 workflow

### 模式 A — 全手動（你每天上 Claude Code 操作）

#### 週日：產下週排程
```
/generate-calendar
```
這會讀 `config/schedule.yaml` 的 slots、呼叫 content-writer 產 caption、image-generator 選素材、寫進 `data/calendar.json`。

**建議跑完人工審一次**（打開 `data/calendar.json` 看看文案、改不滿意的）。

#### 每小時（或隨時）：發文
```
/publish-now
```
這會掃 `data/calendar.json`、找「現在到 10 分內」該發的 row、依 platform 發出去。沒要發的就一句「no due rows, exit」結束。

#### 週日晚：產週報
```
/weekly-report
```
產 `reports/2026-W17.md`，有 5 平台互動數據、Top 5 貼文、3 條優化建議、best_times vs schedule.yaml 對照表。

**想改排程時段**：你自己改 `config/schedule.yaml` 的 slots（週報不會自動改）。

### 模式 B — 自動模式（routine cron）

要讓它自動跑，建 scheduled tasks：

#### 建 `publish-due`
- Claude Code UI → Scheduled Tasks → Create
- **taskId**: `publish-due`
- **Cron**: `0 * * * *`（每小時）
- **Prompt**: 見下方模板
- 建好後按一次 **Run now** 預先批准所有工具權限

#### 建 `generate-weekly`
- **taskId**: `generate-weekly`
- **Cron**: `0 3 * * 0`（週日 03:00）
- **Prompt**: 類似 publish-due 的格式，但執行 `/generate-calendar`

#### 建 `weekly-review`（選配）
- **Cron**: `0 22 * * 0`（週日 22:00）
- **Prompt**: 執行 `/weekly-report`

### Scheduled task prompt 模板

```
你現在要在 noirsboxes-agent 專案中執行 routine 發文流程。

**工作目錄要求：**
必須在 /Users/<你的名字>/Desktop/SocialMediaAgent/noirsboxes-agent 直接跑，不要在 worktree 裡

**任務：**
執行 /publish-now

**行為鐵則：**
- 每一列（row）獨立、照 /publish-now 步驟跑、不跳 platform、不換順序
- 不要因為歷史 notes 說過「FB 會被 block」就跳過不發

**省 token：**
1. gdrive-reader 先讀 data/calendar.json 找 due 列
2. due = 0 → 一句話「no due rows, exit」收工
3. due ≥ 1 → 進 step 2 發文

**登入：**
- 不要執行 /first-time-login（scheduled task 不能處理互動登入）
- 直接試發、卡登入頁就讓 skill 回 failed + 原因

**瀏覽器：**
- 有 due 列才開、/publish-now 結尾會 browser_close

**嚴禁：**
- ❌ 不要寫 logs/*.md
- ❌ 不要 browser_take_screenshot 存檔
- ❌ 不要碰 Google Drive / Apps Script / Doc
- ✅ 權威紀錄只有 data/calendar.json

**回報：**
逐列回報 published / failed + 原因。
```

---

## Telegram Channel 設定（模式 B）

用 [Claude Channels](https://code.claude.com/docs/en/channels)（Anthropic 官方）把 Telegram 接進你**已開的 Claude Code session**，可以在 TG 群組對 Claude 對話下指令。

### 為什麼用 Channels 而不是自製 bot

| | Claude Channels（這份用法） | 自製 spawn bot |
|---|---|---|
| 開新 chat | 重啟 Claude session | 每次自動 |
| 對話式 debug（同 chat 追問）| ✅ Claude 記得前文 | ❌ 每次無記憶 |
| Claude Code 要不要先開 | **必須**（且 `--channels` 啟動）| 不用 |
| 官方支援 | ✅ Anthropic 維護 | ❌ 自己維護 |

我們選 **Channels**：對話式 debug 比批次發文更需要 context 累積。

### 前置條件

- Claude Code v2.1.80+（`claude --version` 確認）
- 用 **claude.ai 帳號**登入（不支援 console / API key）
- 安裝 [Bun](https://bun.sh)：`brew install oven-sh/bun/bun`（plugin 是 Bun 寫的）
- Team / Enterprise 帳號要 admin 在 settings 開 `channelsEnabled: true`（個人帳號不用）

### 設定步驟

#### 1. 申請 Telegram bot token

1. Telegram 找 **@BotFather**、`/newbot`
2. 取顯示名稱、英文 username（要以 `bot` 結尾）
3. 拿到 token（`7891234567:AAExxx...` 格式）

#### 2. 安裝官方 telegram plugin

進 Claude Code 對話打：

```
/plugin marketplace add anthropics/claude-plugins-official
/plugin install telegram@claude-plugins-official
/reload-plugins
```

> 若 marketplace 找不到 → `/plugin marketplace update claude-plugins-official` 再試。

#### 3. 配置 token

```
/telegram:configure 7891234567:AAExxx...
```

token 會存到 `~/.claude/channels/telegram/.env`（自動 gitignore，不在你專案 repo）。

#### 4. 退出後加 `--channels` 重啟

關掉現在的 Claude Code session，在 noirsboxes-agent 目錄重開：

```bash
cd /Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent
claude --channels plugin:telegram@claude-plugins-official
```

這個 session 會接住 bot 的訊息。

#### 5. 配對你的 TG 帳號（首次）

- 打開 TG，找你剛建的 bot，傳任何訊息（例：`hi`）
- bot 回一組配對碼（`pairing code: ABCD1234`）
- 切回 Claude Code 終端機打：

```
/telegram:access pair ABCD1234
```

- 鎖定只允許你發訊息：

```
/telegram:access policy allowlist
```

### 日常使用

Claude Code 開著（with `--channels`）的時候：

```
你（TG）→ /publish-now
Claude → [操作 Playwright 發 5 平台]
Claude → 發完了：FB ✅、IG ✅、X ✅、YT ❌（channel verification needed）
你（TG）→ YT 為什麼失敗？
Claude → 因為 Ron拾 channel 還沒驗證，需要去 youtube.com/account 完成驗證
你（TG）→ 那其他四個的 post URL?
Claude → FB: ... IG: ... X: ... TikTok: ...
你（TG）→ 把 YT 那行改成 status=cancelled
Claude → [編輯 data/calendar.json] 改完了
```

整段在同個 TG 群組、同個 Claude session、累積 context。

### 開新 chat（清乾淨 context）

當對話太長想重置：

- **方法 A**：終端機 Ctrl+C 結束 → `claude --channels plugin:telegram@claude-plugins-official` 重起
- **方法 B**：在 Claude session 內打 `/clear`（保留 session、清對話歷史）

### 注意

- **Claude Code 關了 bot 就停**（沒人接訊息）— 不適合無人值守、用 routine 補
- token 等於 bot 密碼，洩漏出去任何人都能控制
- 一個 bot 只能綁一個 Claude session（多個 session 會搶 polling）
- 想完全無人值守的場景 → 用模式 C（cron routine）

---

## 7 個 slash commands（快查表）

| Command | 做什麼 | 什麼時候用 |
|---|---|---|
| `/first-time-login` | 人工登入 5 平台 | 首次 setup / cookie 過期 |
| `/generate-calendar` | 產下週排程到 `data/calendar.json` | 週日手動 or routine |
| `/publish-now` | 掃 calendar 發到期的 row | 每小時 routine |
| `/test-post` | 5 平台各發一則 `test` | 驗收、偵測平台風控 |
| `/upload-youtube` | 手動上傳長片 | 單次大檔上傳 |
| `/weekly-report` | 產週報 markdown | 週日晚 |
| `/analyze-hotspots` | 熱點分析 | 想找題材靈感時 |

---

## 常見操作情境

### 想新增一則貼文（不走 content-writer）
直接編輯 `data/calendar.json`，append 到 `rows[]`：
```json
{
  "row_id": 20,
  "date": "2026-04-25",
  "time": "10:00",
  "platform": "facebook",
  "topic": "custom",
  "caption": "你想說的話",
  "hashtags": ["#xxx"],
  "image_path": "media/assets/your-image.jpg",
  "video_path": null,
  "status": "scheduled",
  "notes": "手動加"
}
```
記得把對應素材放進 `media/assets/`。

### 想取消一則已排的貼文
編輯 `data/calendar.json`、把該 row 的 `status` 從 `scheduled` 改成 `cancelled`。

### 想重發一則失敗的
編輯 `data/calendar.json`、把該 row 的 `status` 從 `failed` 改回 `scheduled`、調整 `time` 到未來。

### 想停掉 routine（出遠門時）
Claude Code UI → Scheduled Tasks → 找到 publish-due → disable。

### 想暫時讓 FB 不發（平台政策敏感）
編輯 `data/calendar.json`、把所有 `platform: "facebook"` 的 row `status` 改成 `cancelled`（或刪掉）。

---

## 換客戶 / 重用

這個 repo 是**配置驅動**的，換品牌只改設定檔、不改程式：

| 要改什麼 | 改哪 |
|---|---|
| 品牌名、Tagline、禁用詞 | `config/brand.yaml` + `CLAUDE.md` |
| 社群帳號 (handle / url) | `config/brand.yaml` 的 `socials` 區塊 |
| 產品線 | `config/products.yaml` |
| 發文時段 | `config/schedule.yaml` |
| 平台 UI 改版（FB/IG 改版） | 對應 `.claude/skills/publish-*/SKILL.md` |
| 資料源從本地改 Notion/Airtable | 只改 `gdrive-reader` + `gdrive-writer` 兩個 skill |

---

## 故障排除速查

| 症狀 | 可能原因 | 解法 |
|---|---|---|
| 首次跑很久才開 Chromium | 正在下載 Playwright + Chromium | 正常，等 1-3 分鐘 |
| `command not found: npx` | 沒裝 Node.js | `brew install node` 或下載 nodejs.org |
| Playwright 啟動失敗 "browser not found" | Chromium 沒下載完整 | 跑 `npx playwright install chromium` |
| `npx @playwright/mcp` 下載失敗 | 網路 / 防火牆 | 換網、或檢查 npm registry 設定 |
| `/first-time-login` 找不到 `mcp__playwright__*` 工具 | `.claude/settings.json` 的 MCP 區塊沒被 Claude Code 自動 spawn | 用 `claude mcp add` 手動註冊（見 Step 4b）|
| routine 一直跳權限提示 | `~/.claude/settings.json` 沒 reload | Cmd+Q 重啟 Claude Code |
| 發文後貼文不見（FB/IG） | 平台靜默 block 自動化 | 間歇性、下次可能過 |
| routine 跑完所有列 status 都 scheduled | 窗口條件對不上 | 看 `config/schedule.yaml` 的 `publish_window_minutes` 與 cron tick 時機 |
| 跑 `/publish-now` 說 `image not found` | 素材不在 `media/assets/` | 檢查 `data/calendar.json` 的 `image_path` 有無對應實體檔 |
| Playwright 開新 Chromium 卡住 | profile lock（你開著舊視窗）| 關掉手動開的 Chromium 視窗再跑 |
| Scheduled task 每次 run 都要重登 | 用了 `isolation: worktree` | task 設定不要用 worktree |

完整故障排除見 `docs/SYSTEM_CHANGES.md` §6。

---

## 還有問題？

1. **架構層問題** → `docs/SYSTEM_CHANGES.md`
2. **品牌語調問題** → `CLAUDE.md`
3. **某個 skill 的邏輯** → `.claude/skills/<name>/SKILL.md`
4. **某個 command 的步驟** → `.claude/commands/<name>.md`

這個專案刻意寫得「Markdown 即文件」，所有邏輯都在 Markdown 裡、可以邊改邊讀。
