# 已知問題（Warnings）

記錄目前專案的問題、限制、踩過的坑。新發現一條就 append 一條。

---

## 1. TG bot 會一直沿用對話 context

**症狀**：用 `claude --channels plugin:telegram@claude-plugins-official` 啟動後，所有 TG 訊息都進**同一個 Claude session**，對話 context 持續累積。

**影響**：

- 對話越長 → 每次回應 token 越貴
- 舊對話會影響新指令的判斷（例：上次 FB 失敗 → 這次可能被「預測」會失敗）
- Auto-compact 觸發後可能丟失你以為還在的 state

**解法**：

- `/clear` — 在 Claude session 內清對話歷史（保留 session）
- Ctrl+C → 重啟 `claude --channels ...` — 完全重置

**狀態**：by design（Channels 就是同 session 累積，要靠人手動清）。

---
