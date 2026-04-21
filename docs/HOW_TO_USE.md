# 如何使用這個專案（noirsboxes-agent）

給接手的人 / 廠商看的實戰手冊。讀完可以馬上上手。

---

## 這個專案是什麼

一套自動化社群小編，用 Playwright 開真瀏覽器替你發文到 5 平台（FB / IG / X / YouTube / TikTok），內容日曆和素材全部存本地、不碰 Google Drive。

## 兩種使用模式

| 模式 | 時機 | 誰觸發 |
|---|---|---|
| **手動模式** | 第一次設定、測試、突發狀況 | 你在 Claude Code 對話裡打 slash command |
| **自動模式** | 日常營運、無人值守 | Claude Code scheduled task 按 cron 自動跑 |

---

## 初次設定（10 分鐘）

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

### 3. Clone 專案 + 初始化
```bash
cd ~/Desktop
# 從 git 或 zip 拿到 noirsboxes-agent/
cd noirsboxes-agent
ls config/       # 應該看到 brand / products / schedule.yaml
```

### 4. 首次登入 5 平台
在 Claude Code 裡打：
```
/first-time-login
```

Playwright 會開 Chromium → 依序引導你登入 FB / IG / X / YT / TikTok。登完 cookies 寫進 `browser_profiles/shared/`，之後 routine 靠這個登入狀態。

**登完請關掉 Chromium 視窗**（避免 profile lock）。

### 5. 準備素材
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
| routine 一直跳權限提示 | `~/.claude/settings.json` 沒 reload | Cmd+Q 重啟 Claude Code |
| 發文後貼文不見（FB/IG） | 平台靜默 block 自動化 | 看 `feedback_fb_silent_drop` memory；間歇性、下次可能過 |
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
