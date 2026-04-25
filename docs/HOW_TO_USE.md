# 如何使用這個專案（noirsboxes-agent）

給接手的人 / 廠商看的實戰手冊。讀完可以馬上上手。

---

## 這個專案是什麼

社群小編 agent，用 Playwright 開真瀏覽器發文到 5 平台（FB / IG / X / YouTube / TikTok）。透過 Telegram 對 Claude 下指令、或 Claude Code 對話直接打 slash command。

## 兩種使用模式

| 模式 | 時機 | 誰觸發 |
|---|---|---|
| **Telegram 模式**（主要）| 日常使用、想對話式 debug | TG 群組打指令給 bot |
| **直接模式** | 在電腦前 / 第一次設定 | Claude Code 對話打 slash command |

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

主流程：**Telegram bot 觸發**。

### 啟動 bot session（每次開機後一次）

開終端機：

```bash
cd /Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent
claude --channels plugin:telegram@claude-plugins-official
```

讓這個終端機**一直開著**，bot 才會回應 TG 訊息。Cmd+Q / Ctrl+C 關掉就斷線。

### TG 上的常用指令

| 你在 TG 打 | Claude 會做的事 |
|---|---|
| **\[上傳圖\] +** `發 IG，caption: ...` | 圖自動下載、Claude 用 publish-instagram |
| **\[上傳影片\] +** `發 YT short，title: ...` | 影片下載、Claude 用 publish-youtube |
| `/publish-now <platform>，caption: ...` | 直接呼叫對應 publish-* skill |
| `/weekly-report` | 產週報 markdown 到 `reports/<YYYY-WWW>.md` |
| `/analyze-hotspots` | 5 平台熱點分析 |
| `幫我發到 FB+IG` | Claude 用同個 caption / image 多平台同發 |
| `FB 剛剛為什麼失敗？` | Claude 看上文記憶解釋 |
| `再試一次` | Claude 重新跑剛剛的發文 |

**對話會累積 context**：你問「FB 為什麼失敗」後接著說「再試一次」，Claude 知道指哪一篇。

### 想開新對話（清乾淨 context）

當對話太長想重置：

- **方法 A**：終端機 Ctrl+C → 重啟 `claude --channels ...`
- **方法 B**：在 Claude session 內打 `/clear`（保留 session、清歷史）

### 替代：直接在 Claude Code 對話下指令

不在 TG 旁邊時，電腦上 cd 進專案、`claude` 進去後一樣的 slash command 也會跑。

⚠️ 已知坑見 [`docs/WARNINGS.md`](./WARNINGS.md)。

---

## Telegram Channel 設定

### 前置條件

- Claude Code v2.1.80+（`claude --version` 確認）
- 用 claude.ai 帳號登入（不支援 console / API key）
- 安裝 [Bun](https://bun.sh)：`brew install oven-sh/bun/bun`
- Team / Enterprise 帳號要 admin 在 settings 開 `channelsEnabled: true`（個人帳號不用）

### 設定步驟

#### 1. 申請 Telegram bot token

Telegram 找 **@BotFather**、`/newbot`、取名（username 要以 `bot` 結尾）、拿 token（`7891234567:AAExxx...` 格式）。

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

token 存到 `~/.claude/channels/telegram/.env`。

#### 4. 退出後加 `--channels` 重啟

```bash
cd /Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent
claude --channels plugin:telegram@claude-plugins-official
```

#### 5. 配對 TG 帳號（首次）

- TG 找你 bot、傳任何訊息（例：`hi`）
- bot 回配對碼（`pairing code: ABCD1234`）
- 在 Claude Code 終端機打：
  ```
  /telegram:access pair ABCD1234
  /telegram:access policy allowlist
  ```

---

## 5 個 slash commands

| Command | 做什麼 | 什麼時候用 |
|---|---|---|
| `/first-time-login` | 人工登入 5 平台 | 首次 setup / cookie 過期 |
| `/publish-now` | 直接發指定貼文到指定平台 | 主要發文指令 |
| `/test-post` | 5 平台各發一則 `test` | 驗收、偵測平台風控 |
| `/weekly-report` | 產週報 markdown | 週末做總結時 |
| `/analyze-hotspots` | 5 平台熱點分析 | 想找題材靈感時 |

---

## 換客戶 / 重用

這個 repo 是配置驅動的，換品牌只改設定檔：

| 要改什麼 | 改哪 |
|---|---|
| 品牌名、Tagline、禁用詞 | `config/brand.yaml` + `CLAUDE.md` |
| 社群帳號（handle / url）| `config/brand.yaml` 的 `socials` |
| 產品線 | `config/products.yaml` |
| 平台 UI 改版（FB / IG 改版）| 對應 `.claude/skills/publish-*/SKILL.md` |

---

## 故障排除速查

| 症狀 | 解法 |
|---|---|
| 首次跑很久才開 Chromium | 正常，等 1-3 分鐘下載 |
| `command not found: npx` | `brew install node` |
| Playwright "browser not found" | `npx playwright install chromium` |
| `/first-time-login` 找不到 `mcp__playwright__*` | `claude mcp add playwright -s user -- npx -y @playwright/mcp@latest --user-data-dir <絕對路徑>/browser_profiles` |
| 一直跳權限提示 | `~/.claude/settings.json` 加 `bypassPermissions`，Cmd+Q 重啟 |
| 發文後貼文不見（FB / IG）| 平台靜默 block 自動化、間歇性，下次可能過 |
| Playwright 開新 Chromium 卡住 | profile lock — 關掉手動開的 Chromium 視窗 |

完整踩過的坑見 `docs/SYSTEM_CHANGES.md` §6。

---

## 還有問題？

1. **架構層問題** → `docs/SYSTEM_CHANGES.md`
2. **品牌語調問題** → `CLAUDE.md`
3. **某個 skill 的邏輯** → `.claude/skills/<name>/SKILL.md`
4. **某個 command 的步驟** → `.claude/commands/<name>.md`

這個專案刻意寫得「Markdown 即文件」，所有邏輯都在 Markdown 裡、可以邊改邊讀。
