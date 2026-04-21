---
name: content-writer
description: 給定產品 ID + 主題 + 平台 + 語言，產出符合品牌調性的貼文 caption 與 hashtags
---

## Input
- `product_id`：對應 `config/products.yaml` 的 id
- `theme`：選自 `content_themes`，例如「技術知識」
- `platform`：facebook / instagram / tiktok / youtube / x
- `language`：zh-TW / en / ja / de（預設 zh-TW + en 並排）

## Output（JSON）
```json
{
  "caption": "...",
  "hashtags": ["#...", "#..."]
}
```

## 規則
1. 必讀 `config/brand.yaml` 的 `voice` 段，**不可使用 forbidden_words**
2. 平台字數限制：
   - X：≤ 280 字元（含 hashtags）
   - Instagram：caption ≤ 2200，hashtag 5–10 個
   - Facebook：≤ 500 字
   - TikTok / YouTube Shorts：≤ 150 字 + 強行動呼籲
3. Hashtag 必含品牌標 `#NoirsBoxes`
4. 結尾若可加 CTA，導去 https://www.noirsboxes.com/

## 不要做
- 不要編造未驗證的測試數字或認證
- 不要承諾交期 / 折扣（除非 caller 明確傳入 promotion 欄位）
- 不要呼叫其他 skill — 純 LLM 任務
