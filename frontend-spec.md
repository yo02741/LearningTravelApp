# 旅遊語言自主學習 APP — 前端規格文件

## 1. 技術選型

| 項目 | 選擇 | 說明 |
|------|------|------|
| 核心框架 | React 18 | 組件化架構，生態豐富 |
| 路由管理 | React Router v6 | 單頁應用路由 |
| 狀態管理 | React Context + useReducer | 輕量管理全域狀態（收藏、學習進度、篩選條件） |
| 樣式方案 | CSS Modules 或 Tailwind CSS | 模組化避免樣式衝突，Tailwind 加快 UI 開發 |
| HTTP 客戶端 | Axios | 與後端 API 通訊 |
| 語音播放 | Web Speech API（SpeechSynthesis） | 瀏覽器原生發音，免費且免安裝 |
| 本地儲存 | localStorage | 儲存收藏清單與學習進度 |
| 建置工具 | Vite | 快速開發伺服器與建置 |
| 部署平台 | Firebase Hosting 或 Vercel | 靜態資源 CDN 托管 |

---

## 2. 頁面結構與路由

```
/                     首頁（短語列表）
/phrase/:id           短語詳情頁
/favorites            我的收藏頁
/category/:slug       分類瀏覽頁
```

---

## 3. 頁面規格

### 3.1 首頁（短語列表頁）`/`

**功能描述：**
應用的主入口，展示所有可學習的旅遊短語，並提供搜尋與分類篩選。

**頁面組成：**
- **頂部導覽列（Header）**
  - 左側：應用 Logo / 名稱
  - 右側：「我的收藏」入口圖示（顯示收藏數量 badge）
- **搜尋列（Search Bar）**
  - 輸入框：placeholder 為「搜尋短語（中文、英文、日文）...」
  - 輸入時即時過濾短語列表（debounce 300ms）
  - 有內容時顯示清除按鈕
- **分類篩選標籤列（Category Filter）**
  - 水平可捲動的標籤列
  - 選項：全部 | 機場 | 飯店 | 餐廳 | 交通 | 購物 | 觀光 | 緊急狀況
  - 選中的標籤以不同樣式凸顯
- **短語卡片列表（Phrase Card List）**
  - 以響應式格線排列（手機 1 欄，平板 2 欄，桌機 3 欄）
  - 無搜尋結果時顯示「找不到符合的短語」提示

**短語卡片（Phrase Card）元件內容：**
- 情境圖片（固定比例，圖片載入前顯示 placeholder）
- 分類標籤（如：餐廳）
- 中文說明（粗體，如：點餐）
- 英文短語（如：I'd like to order, please.）
- 日文短語（如：注文をお願いします。）
- 「已學習」標記按鈕（打勾圖示，已標記時顏色改變）
- 「收藏」按鈕（愛心圖示，已收藏時顏色改變）
- 「發音播放」按鈕（喇叭圖示，點擊以 Web Speech API 朗讀英文短語）

**互動行為：**
- 點擊卡片主區域 → 跳轉至短語詳情頁
- 點擊「已學習」→ 切換學習狀態（存入 localStorage）
- 點擊「收藏」→ 切換收藏狀態（存入 localStorage）
- 點擊「發音播放」→ 以瀏覽器語音朗讀英文短語，播放期間圖示動態變化

---

### 3.2 短語詳情頁 `/phrase/:id`

**功能描述：**
展示單一短語的完整學習內容，包括雙語翻譯、發音、上下文例句與圖片。

**頁面組成：**
- **返回按鈕** — 點擊返回上一頁
- **情境圖片** — 全寬顯示，代表該短語的使用場景
- **分類標籤與中文說明**
- **短語資訊區塊**
  - 英文短語（大字顯示）
  - 英文發音標示（如：/aɪd laɪk tə ˈɔːrdər/）
    - 右側提供「顯示 / 隱藏發音」切換按鈕
    - 隱藏時以「●●●」替代顯示
  - 播放英文發音按鈕
  - 日文短語（中字顯示）
  - 日文發音標示（平假名標注）
    - 同樣提供顯示 / 隱藏切換
  - 播放日文發音按鈕
- **上下文例句區塊**
  - 英文例句
  - 中文翻譯
  - 日文例句（可選）
- **操作按鈕列**
  - 「已學習」標記按鈕
  - 「收藏」按鈕
- **相關短語推薦**（同分類的其他短語，最多 4 張卡片）

---

### 3.3 我的收藏頁 `/favorites`

**功能描述：**
集中展示使用者已收藏的短語，方便快速複習。

**頁面組成：**
- **頂部導覽列** — 包含返回按鈕與頁面標題「我的收藏」
- **收藏計數** — 顯示「共 N 個收藏」
- **短語卡片列表** — 與首頁相同的卡片元件
- **空狀態提示** — 當無收藏時，顯示「還沒有收藏的短語，去首頁探索吧！」並提供返回首頁連結

---

### 3.4 分類瀏覽頁 `/category/:slug`

**功能描述：**
展示特定場景分類下的所有短語。

**頁面組成：**
- **頂部導覽列** — 包含返回按鈕與分類名稱
- **分類說明** — 簡短描述此分類的使用場景
- **短語卡片列表** — 同首頁列表格式
- **空狀態提示** — 若分類下無資料，顯示提示訊息

---

## 4. 全域元件

### 4.1 底部導覽列（Mobile Navigation Bar）
僅在手機螢幕寬度下顯示（< 768px）。

| 圖示 | 標籤 | 目標路由 |
|------|------|---------|
| 🏠 首頁 | 首頁 | `/` |
| 🔖 收藏 | 我的收藏 | `/favorites` |

### 4.2 載入中狀態（Loading Skeleton）
- 資料尚未載入時，以 Skeleton 佔位元件取代卡片，避免版面跳動。

### 4.3 錯誤提示（Error State）
- API 請求失敗時，顯示友善錯誤訊息與「重試」按鈕。

---

## 5. 響應式設計規格

| 斷點 | 寬度範圍 | 列表欄位數 | 備註 |
|------|---------|-----------|------|
| Mobile | < 768px | 1 欄 | 底部導覽列顯示 |
| Tablet | 768px – 1024px | 2 欄 | 頂部導覽列顯示 |
| Desktop | > 1024px | 3 欄 | 頂部導覽列顯示 |

---

## 6. 狀態管理

### 全域狀態（Context）

```
AppContext:
  - favorites: string[]          已收藏的短語 ID 清單
  - learned: string[]            已標記學習的短語 ID 清單
  - toggleFavorite(id): void     切換收藏狀態
  - toggleLearned(id): void      切換學習狀態
```

### 本地儲存鍵值

| 鍵名 | 資料格式 | 說明 |
|------|---------|------|
| `lt_favorites` | JSON 字串陣列 | 已收藏的短語 ID |
| `lt_learned` | JSON 字串陣列 | 已標記學習的短語 ID |

---

## 7. API 串接規格

### 基礎設定
- Base URL：從環境變數 `VITE_API_BASE_URL` 讀取
- 請求 Header：`Content-Type: application/json`
- 錯誤處理：統一捕捉 HTTP 4xx / 5xx 錯誤並顯示提示

### 使用的 API 端點

| 用途 | HTTP 方法 | 路徑 |
|------|---------|------|
| 取得所有分類 | GET | `/api/categories` |
| 取得短語列表（支援分類與關鍵字篩選） | GET | `/api/phrases` |
| 取得單一短語詳情 | GET | `/api/phrases/:id` |

---

## 8. 發音功能規格

### Web Speech API 使用方式
- 使用 `window.speechSynthesis.speak()` 進行文字轉語音
- 英文短語：設定語言為 `en-US`
- 日文短語：設定語言為 `ja-JP`
- 播放前先呼叫 `window.speechSynthesis.cancel()` 取消上一次播放
- 若瀏覽器不支援 SpeechSynthesis，隱藏播放按鈕並顯示「瀏覽器不支援語音功能」提示

---

## 9. 效能最佳化

- **圖片延遲載入（Lazy Loading）**：使用 `loading="lazy"` 屬性或 Intersection Observer，僅在圖片進入可視區域時才載入。
- **搜尋防抖（Debounce）**：搜尋輸入加入 300ms debounce，避免每次按鍵都觸發 API 請求。
- **API 結果快取**：使用 `useEffect` 搭配 `useMemo` 或第三方快取套件（如 SWR / React Query）減少重複請求。
- **Code Splitting**：以 `React.lazy` 和 `Suspense` 進行路由層級的程式碼分割，縮短首頁載入時間。

---

## 10. 專案目錄結構（建議）

```
src/
├── components/        # 共用 UI 元件
│   ├── PhraseCard/
│   ├── SearchBar/
│   ├── CategoryFilter/
│   ├── BottomNav/
│   └── LoadingSkeleton/
├── pages/             # 頁面元件
│   ├── Home/
│   ├── PhraseDetail/
│   ├── Favorites/
│   └── Category/
├── context/           # 全域狀態
│   └── AppContext.jsx
├── hooks/             # 自訂 Hook
│   ├── usePhrase.js
│   ├── useSpeech.js
│   └── useLocalStorage.js
├── services/          # API 呼叫邏輯
│   └── api.js
├── utils/             # 工具函式
└── App.jsx
```
