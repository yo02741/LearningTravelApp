# 旅遊語言自主學習 APP - 實施計劃

## 項目概述
一款響應式網頁應用（可在手機上操作），專注於提供旅遊相關的英文與日文短語集。用戶可以無需登錄即可使用，學習和練習常用旅遊用語。

## 核心特性
- **短語集展示**：包含中英文翻譯、發音（支援隱藏模式）、圖片、上下文例句
- **響應式設計**：在桌機與手機上都能良好運作
- **無帳號登錄**：簡化初版，直接使用即可
- **易於分享/發布**：可透過 Firebase Hosting 自建公開發布

## 技術棧選擇
- **前端**：React（新技術學習機會）
- **後端**：Python + Flask/FastAPI（發揮既有優勢）
- **資料庫**：SQLite（本地開發）/ PostgreSQL（生產環境）
- **部署**：Firebase Hosting（前端）+ Cloud Run/自建伺服器（後端）或 Vercel + 簡易後端

## 開發階段

### 第一階段：核心架構與基礎功能
1. **項目初始化**
   - React 前端專案建置
   - Python 後端專案建置
   - 資料庫設計（短語集、分類、媒體關聯）

2. **後端 API 開發**
   - 設計資料模型（短語、分類、發音、圖片、例句）
   - 實作 REST API 端點（列表、搜尋、詳細內容）
   - 資料庫 CRUD 操作

3. **前端 UI 開發**
   - 主頁面 / 短語列表頁
   - 短語詳細卡片（含發音、發音隱藏開關）
   - 搜尋與篩選功能
   - 響應式設計（Mobile-first）

4. **整合與測試**
   - 前後端連接驗證
   - 基礎功能測試
   - 跨裝置響應式測試

### 第二階段：增強功能（未來迭代）
- 用戶進度追蹤（LocalStorage 或簡單 Firebase）
- 發音練習（語音辨識）
- 分享功能
- 離線模式

## 資料結構設計

### 短語表
```
id, phrase_en, phrase_ja, phrase_zh, category, pronunciation_en, pronunciation_ja, image_url, context_example, created_at
```

### 分類表
```
id, name_en, name_ja, name_zh, description
```

## 部署方案
- **前端**：React 構建 → Firebase Hosting 或 Vercel
- **後端**：Python Flask/FastAPI → Google Cloud Run 或自建伺服器
- **資料庫**：開發用 SQLite，生產用 PostgreSQL（雲端託管）

## 關鍵考量
- 發音支援：使用瀏覽器原生 Web Speech API 或整合第三方服務
- 圖片資源：考慮使用免費圖庫或自製圖片
- 分享機制：可先透過 URL 分享或簡單的資料匯出功能

---

## 後續步驟
1. 確認短語數據來源與初始數據量
2. 決定是否需要管理後台（用於新增/編輯短語）
3. 選定具體的後端部署平台
4. 開始第一階段開發
