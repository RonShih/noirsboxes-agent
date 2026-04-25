---
description: 併發爬 5 平台熱點，交叉過濾出跟 NoirsBoxes 產品（充電器/PD/快充）相關的主題，輸出建議清單
argument-hint: "[top_n 預設 10]"
---

## 流程

1. **併發**呼叫 5 個 skill：
   - `fetch-trends-x`
   - `fetch-trends-youtube`
   - `fetch-trends-tiktok`
   - `fetch-trends-instagram`
   - `fetch-trends-facebook`
2. 合併所有 trends，為每項算相關性分數：
   - **硬相關（+10）**：命中 `快充 | PD | USB-C | Type-C | 充電 | GaN | 無線充 | adapter | charger | power delivery`
   - **軟相關（+5）**：科技/3C/電子/手機/iPhone/Android/蘋果/Pixel/筆電/Switch/AirPods
   - **跨平台加分（+3）**：同一主題在 ≥ 2 平台都看到
3. 依分數降序取 top_n（預設 10），輸出 Markdown 表：

   ```
   | 分數 | 主題 | 來源平台 | 建議切入角度 |
   | 18 | iPhone 17 電池 | X + YT | 用 MD-905 實測 iPhone 17 快充速率 |
   ```

4. 不寫 Drive、不動 Sheet — 純終端輸出給使用者

## 不要做
- 不要把相關性過濾塞進 `fetch-trends-*` skill — 過濾邏輯集中在這裡
- 不要產 caption — caption 是 `content-writer` 的事
- 不要快取；每次跑都抓即時資料
