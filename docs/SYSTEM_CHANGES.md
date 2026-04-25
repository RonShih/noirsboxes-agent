# 系統層級改動紀錄（noirsboxes-agent 部署注意事項）

這份文件列出**不在專案 git repo 裡**、但對部署**必要**的系統層級設定。交付廠商或換電腦時照這份重建。

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

## 3. Telegram Channel（Anthropic 官方 plugin）

### 安裝位置
```
~/.claude/channels/telegram/.env       ← bot token（自動 gitignore）
~/.claude/channels/telegram/inbox/     ← 收到的圖 / 影片自動下載到這
```

### 啟動指令
```bash
claude --channels plugin:telegram@claude-plugins-official
```

詳細設定步驟見 `docs/HOW_TO_USE.md` 的「Telegram Channel 設定」。

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
│   └── stats-history.json    ← 每週 stats 歷史（/weekly-report 產出時建）
├── media/
│   └── assets/               ← 圖片、影片素材
└── reports/
    └── <YYYY-WWW>.md         ← 週報 markdown
```

TG 上傳的素材會自動下載到 `~/.claude/channels/telegram/inbox/`，Claude 直接拿來發文。

### 換機器時
- Git push `config/`、`.claude/`、`docs/` → 新機器 pull
- `media/assets/` **不進 git**（檔案大）→ rsync / zip 傳
- `browser_profiles/` 不進 git → 新機器要重跑 `/first-time-login`

### `config/secrets.yaml` 目前空的
目前無實際作用、保留為未來 OpenAI API key 等用途的 placeholder。

---

## 6. 常見坑（踩過的）

| 坑 | 現象 | 解法 |
|---|---|---|
| FB 靜默封鎖自動發文 | UI 流程 OK、dialog 關了、貼文不見 | 平台 anti-automation；間歇性、下次可能過 |
| 一直跳權限提示 | session 沒讀到 bypass 設定 | `~/.claude/settings.json` 加 `defaultMode: bypassPermissions`、Cmd+Q 重啟 |
| Channel session 看不到 Playwright | Playwright 只裝在專案 `.claude/settings.json` | `claude mcp add playwright -s user -- ...`（user-level）|
| 改完 settings 沒生效 | Claude Code 啟動時只讀一次 | Cmd+Q 重開 Claude Code |
| Playwright 開新 Chromium 卡住 | profile lock — 你開著手動登入的視窗 | 關掉手動 Chromium 視窗 |

---

## 7. 交付 / 換機器 checklist

```
[ ] clone repo
[ ] brew install oven-sh/bun/bun
[ ] claude mcp add playwright -s user -- npx -y @playwright/mcp@latest --user-data-dir <絕對路徑>/browser_profiles
[ ] 建 ~/.claude/settings.json（複製本文件 §1 內容）
[ ] 重啟 Claude Code
[ ] 把 media/assets/ 底下素材傳到新機器（rsync / zip）
[ ] /plugin install telegram@claude-plugins-official、配對 TG bot
[ ] /first-time-login 登入 5 平台
[ ] 關掉 Playwright Chromium 視窗
[ ] claude --channels plugin:telegram@claude-plugins-official 啟動 bot session
```

---

## 8. 專案內 commands（slash commands）

路徑 `.claude/commands/*.md` — 手動在 Claude Code 對話或 TG bot 對話打 `/xxx`。

| Command | 用途 |
|---|---|
| `/first-time-login` | Playwright 打開 5 平台等人類登入 → 存 cookies 到 `browser_profiles/` |
| `/publish-now` | 依當下 TG 對話意圖直接呼叫 publish-* skill 發文 |
| `/test-post` | 5 平台 smoke test |
| `/weekly-report` | 併發 collect-stats-* → 產 markdown 週報到 `reports/` |
| `/analyze-hotspots` | 併發 fetch-trends-* → 算熱點排序 |

---

## 10. 專案內 skills

路徑 `.claude/skills/<name>/SKILL.md` — command 呼叫或 Claude 自己 call。不會被人類直接 slash-invoke。

### 發文類（5 個，對應 5 平台）
| Skill | 輸入 | 輸出 |
|---|---|---|
| `publish-facebook` | caption / hashtags / local_image_path | `{post_url}` |
| `publish-instagram` | 同上 | 同上 |
| `publish-x` | caption / local_image_path（選填）| 同上 |
| `publish-youtube` | title / description / tags / local_video_path | `{video_url}` |
| `publish-tiktok` | caption / hashtags / local_video_path | `{video_url}` |

共通鐵則見 `CLAUDE.md` 的「共通行為規則」章節。

### 內容產製類
| Skill | 用途 |
|---|---|
| `content-writer` | 產 caption + hashtags（依平台風格）|
| `image-generator` | 產配圖 / 選素材 |
| `seo-metadata` | 產 YT title / description / tags |

### 資料存取類
| Skill | 用途 |
|---|---|
| `local-reader` | 列素材檔、檢查路徑 |
| `local-writer` | 寫週報 markdown、append stats |

### 熱點分析類（5 個，對應 5 平台）
| Skill | 用途 |
|---|---|
| `fetch-trends-{facebook,instagram,x,youtube,tiktok}` | 抓對應平台熱門主題 |

### 數據收集類（5 個）
| Skill | 用途 |
|---|---|
| `collect-stats-{facebook,instagram,x,youtube,tiktok}` | 收過去 7 天每篇 reach / engagement / best-time |

### 報告類
| Skill | 用途 |
|---|---|
| `report-writer` | 產週報 markdown 字串 |

---

## 9. 專案內 config 檔

| 檔案 | 用途 | Git |
|---|---|---|
| `config/brand.yaml` | 品牌資訊、5 平台 handle / URL、`storage` 區塊 | commit |
| `config/products.yaml` | 產品線、給 content-writer 抽主題 | commit |
| `config/secrets.yaml` | 預留 placeholder | **gitignore** |

---

## 10. Dataflow

```
你（TG）→ 訊息 / 上傳圖
   ↓
~/.claude/channels/telegram/inbox/ ← 圖自動下載
   ↓
Channel session 收到、Claude 解讀意圖
   ↓
直接呼叫 publish-<platform> skill（不走 calendar、不走 cron）
   ↓
Playwright 操作瀏覽器發文
   ↓
回報 post URL 給 TG
```

週末產報告的支線：

```
你（TG）→ /weekly-report
   ↓
併發呼叫 collect-stats-* × 5
   ↓
report-writer 產 markdown 字串
   ↓
local-writer 寫入 reports/<YYYY-WWW>.md
   ↓
append stats 到 data/stats-history.json
```

**完全本地 — 無任何外部 API 依賴（除了 5 個社群平台本身）。**
