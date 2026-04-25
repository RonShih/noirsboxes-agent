---
name: local-reader
description: 檢查本地素材檔存在 / 列素材目錄。
---

## 兩種模式

### 1. resolve_asset_path

**用途**：拿到素材的相對路徑（或檔名）→ 回傳絕對路徑 + 確認存在。publish-* skill 收到 `local_image_path` / `local_video_path` 之前先用這個驗。

**Input**：`path`（相對於專案根、或絕對路徑、或 `~/.claude/channels/telegram/inbox/...`）

**回傳**：
```json
{ "abs_path": "/Users/ron/Desktop/SocialMediaAgent/noirsboxes-agent/media/assets/xxx.jpg", "exists": true, "size_bytes": 123456 }
```

不存在 → `{ "error": "asset not found", "path": "..." }`

### 2. list_assets

**用途**：列出 `media/assets/` 或 `~/.claude/channels/telegram/inbox/` 底下的素材，方便 Claude 跟使用者確認用哪張圖。

**Input**：`dir`（選填，預設兩個都列）

**回傳**：陣列 `[{ name, abs_path, size_bytes, modified_at }]`

## 不要做

- 不要做下載、轉檔、resize
- 不要快取，每次都即時 stat
