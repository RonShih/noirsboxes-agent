---
description: 直接發一則貼文到指定的一個或多個平台
argument-hint: "[platforms（逗號分隔，例 fb,ig,x）] [其他指示如 caption / image_path]"
---

依使用者意圖（含 args + TG 對話前後文 + 上傳的素材）發文到指定平台。

## 步驟

1. **解析意圖** — 從以下來源拼出要發什麼、發到哪：
   - 指令參數（若有 `fb,ig` 等 platform 字串）
   - TG 訊息文字（caption、hashtags、要發到哪幾個平台）
   - TG 上傳的圖 / 影片（路徑通常是 `~/.claude/channels/telegram/inbox/<id>.jpg`）
   - 之前對話的上下文（例：「再試一次」=> 用上次的內容）

2. **確認素材路徑為本地絕對路徑**。需要圖 / 影片但找不到時，回問使用者該用哪張，**不要自己亂猜或從 Drive 下載**。

3. **依平台分發到對應 skill**（單平台單呼叫；多平台時依序執行、各自獨立判斷成敗）：

   | platform | 呼叫的 skill | 必填 input |
   |---|---|---|
   | `facebook` | `publish-facebook` | caption + hashtags + local_image_path |
   | `instagram` | `publish-instagram` | caption + hashtags + local_image_path |
   | `x` | `publish-x` | caption（含 hashtags 後 ≤ 280 字）+ local_image_path（選填） |
   | `youtube` | `publish-youtube` | title + description + tags + local_video_path |
   | `tiktok` | `publish-tiktok` | caption + hashtags + local_video_path |

4. **回報成果** — 逐平台列：✅ + post_url，或 ❌ + 失敗原因。

5. **關瀏覽器** — 全部跑完呼叫 `mcp__playwright__browser_close`。

## 驗收

- [ ] 指定平台都有貼文
- [ ] 對話回了每個平台的 post_url 或失敗原因
