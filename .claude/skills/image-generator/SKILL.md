---
name: image-generator
description: 為一篇貼文產生或從本地素材庫挑選一張配圖，回傳本地相對路徑
---

## Input
- `caption`：貼文文字（用來提示圖片風格）
- `product_id`：產品 id（會優先在 `media/assets/<product_id>/` 找現成圖）
- `platform`：決定比例（fb/ig=1:1, ig story/tiktok=9:16, yt thumbnail=16:9）

## Output
```json
{ "image_path": "media/assets/<slug>.jpg", "source": "library|generated" }
```

`image_path` 是**相對於專案根目錄**的路徑。

## 流程
1. 先 `ls media/assets/<product_id>/` 列現有圖
2. 若找到合適的 → 直接挑一張，回傳 `source=library`
3. 否則：
   - 用 LLM 把 caption 摘要成英文 prompt（強調產品技術感、白底、無浮誇）
   - 呼叫圖片產生 API（caller 提供，預設 OpenAI Images）
   - 把產生的圖存到 `media/assets/<product_id>/generated/<slug>.jpg`
   - 回傳相對路徑 `media/assets/<product_id>/generated/<slug>.jpg`

## 不要做
- 不要把圖上傳 Drive — **全部留在本地 `media/assets/`**
- 不要在貼文內嵌入競品標誌或人臉
- 不要重複使用過去 14 天用過的圖

## 檔案命名慣例
- 產品類圖：`md905.jpg`, `md905-epr.jpg`, `md903-fake-cable.jpg`
- 案例類圖：`md903-case.jpg`, `wireless-fod.jpg`
- 產生圖：`<product_id>/generated/<YYYYMMDD-HHMM>.jpg`
- 短影音：`<product_id>-short-<slug>.mp4`
