---
name: seo-metadata
description: 給定影片主題與目標市場，產生 YouTube SEO 最佳化的 title / description / tags
---

## Input
- `topic`：影片主題（例如 "PD 3.1 PPS 握手過程實測"）
- `product_id`：對應產品
- `language`：主要語言（預設雙語標題：中文標題 + 英文標題用「｜」分隔）
- `duration_seconds`：影片長度，用於決定要不要建立章節時間戳

## Output
```json
{
  "title": "PD 3.1 PPS 握手實測｜PD Tester Hands-on with PPS Negotiation",
  "description": "...",
  "tags": ["PD 3.1", "PPS", "USB-C", "..."],
  "thumbnail_hint": "auto" 
}
```

## 規則
- **title** ≤ 100 字元，前 60 字元是 SEO 黃金區
- **description** 結構：
  1. 第 1 行：一句話價值主張
  2. 接著 3 行品牌簡介 + 官網連結 https://www.noirsboxes.com/
  3. 若 duration ≥ 90 秒 → 自動建議章節時間戳
  4. 結尾：訂閱 CTA、銷售信箱 sales@mdv.com.tw
- **tags** 10–15 個，混合「廣詞 + 長尾詞」，含 `NoirsBoxes`、`充電器測試`、`PD tester` 等
- 必讀 `config/brand.yaml` 的 forbidden_words

## 不要做
- 不要塞無關熱門關鍵字（YT 演算法會懲罰）
- 不要在 title 用全大寫或 clickbait
