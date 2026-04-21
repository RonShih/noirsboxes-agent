---
description: 首次部署：開啟 Playwright 瀏覽器，引導使用者人工登入 5 個社群平台一次，登入態會持久化在 ./browser_profiles
---

依下列順序執行，每個平台都「打開頁面 → 暫停 → 等使用者按 Enter 確認登入完成 → 截圖存證」：

1. Facebook — `https://www.facebook.com/`
2. Instagram — `https://www.instagram.com/`
3. TikTok — `https://www.tiktok.com/login`
4. YouTube Studio — `https://studio.youtube.com/`
5. X — `https://x.com/login`

執行步驟：

1. 用 `mcp__playwright__browser_navigate` 開第一個平台
2. 顯示訊息：「請在瀏覽器手動登入 [平台名]，登入完成後在此回覆 'ok' 我會繼續下一個」
3. 等使用者回覆 `ok` → 用 `mcp__playwright__browser_take_screenshot` 截圖到 `logs/login-<platform>.png`
4. 重複到 5 個平台都完成
5. 最後告訴使用者：登入態已存到 `./browser_profiles`，之後 publish-* 會自動沿用
