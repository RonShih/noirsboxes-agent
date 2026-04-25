# AGENT_RULES — 共通行為規則

**任何 command / skill 開始執行前必須先讀這份。** CLAUDE.md 已在「全域行為規則」章節指向這裡。

下列規則適用於**所有** skill / command，個別 SKILL.md 只描述該平台特有的步驟，不再重複這裡的條目。

---

## 1. 失敗處理

- 失敗結構化回 `{ "error": "<原因>" }`，不要丟 exception
- 不要自動重試超過 1 次
- 不要為了「成功」而偽造資料 — 失敗就老實回失敗
- 失敗時清楚告訴使用者哪一步、為什麼

## 2. 檔案 / 紀錄

- **不要** `browser_take_screenshot` 存檔到任何地方
- `browser_snapshot`（讀頁面結構，不存檔）**該用就用** — Claude 靠它找 element ref
- **不要**寫任何 run-level log 檔到 `logs/`
- 持久狀態只在合理位置：素材 `media/assets/` + TG inbox、報告 `reports/`、stats `data/stats-history.json`

## 3. 素材路徑

- publish-* skill 收到的 `local_image_path` / `local_video_path` 一律是**絕對路徑**（`/Users/.../media/assets/xxx.jpg` 或 `~/.claude/channels/telegram/inbox/xxx.jpg`）
- 開始之前先用 `local-reader.resolve_asset_path` 確認檔案存在；不存在就回 `{ "error": "asset not found" }`
- **不要刪除** `local_image_path` / `local_video_path` 指到的檔（素材會被多次重用）
- **不要從任何 URL 下載**素材；所有素材必須已在本地

## 4. 不要做平台預測

- 不要因為「過去某列失敗 / 平台 X 之前卡登入 / 平台 Y 對自動化敏感」就**主動跳過、換順序、改策略**
- 你被 call 就代表 caller 已經判斷這個任務該做。**照步驟跑到底**，只有實際遇到錯誤時才回 `error`
- 失敗了才寫 `{ error }`，不要「預測」失敗

## 5. 不要碰 Google 雲端

- 不要走 Google Drive、Google Sheet、Google Doc、Apps Script
- 不要嘗試 OAuth、不要要求 API key
- 所有資料都本地

## 6. 帳密 / 登入

- 不要在程式碼或 markdown 寫死任何帳密 / token
- 一律 reuse Playwright persistent profile（`browser_profiles/shared/`）的 cookie
- session 過期讓使用者重跑 `/first-time-login`，不要嘗試自動登入

## 7. Skill 之間零相依

- Command 編排 skill；skill 內部**不要呼叫另一個 skill**
- 例外：command 自己可以併發呼叫多個 skill（例：/weekly-report 併發 5 個 collect-stats-*）

## 8. 瀏覽器生命週期

- 開了瀏覽器、做完事 → 呼叫 `mcp__playwright__browser_close`
- 不要把使用者手動開的 Chromium 視窗強制關掉（profile lock 衝突時優先回報，不要硬搶）
