# 旅遊語言自主學習 APP — 後端規格文件

## 1. 技術選型

| 項目 | 選擇 | 說明 |
|------|------|------|
| 語言 | Python 3.11+ | 主要開發語言 |
| Web 框架 | FastAPI | 自動產生 API 文件（Swagger UI），效能佳 |
| ORM | SQLAlchemy 2.x | 資料庫操作抽象層，支援多種資料庫 |
| 資料驗證 | Pydantic v2 | FastAPI 內建整合，型別安全 |
| 資料庫（開發） | SQLite | 零配置，方便本地開發 |
| 資料庫（生產） | PostgreSQL | 穩定的生產環境關聯式資料庫 |
| 資料庫遷移 | Alembic | 管理資料庫結構版本變更 |
| 跨域設定 | FastAPI CORSMiddleware | 允許前端跨域存取 |
| 部署 | Google Cloud Run 或 Render | 容器化部署，按需擴展 |
| 容器化 | Docker | 確保開發與生產環境一致 |

---

## 2. 資料模型

### 2.1 分類（Category）
代表旅遊短語的場景分類。

| 欄位 | 型別 | 限制 | 說明 |
|------|------|------|------|
| id | Integer | Primary Key, Auto Increment | 唯一識別碼 |
| slug | String(50) | Unique, Not Null | URL 友善識別名稱（如 `airport`） |
| name_zh | String(50) | Not Null | 中文名稱（如：機場） |
| name_en | String(50) | Not Null | 英文名稱（如：Airport） |
| name_ja | String(50) | Not Null | 日文名稱（如：空港） |
| description_zh | Text | Nullable | 分類的中文場景說明 |
| icon | String(10) | Nullable | 分類的 Emoji 圖示（如：✈️） |
| sort_order | Integer | Default 0 | 前端排列順序 |
| created_at | DateTime | Default now() | 建立時間 |

---

### 2.2 短語（Phrase）
旅遊學習的核心內容單元。

| 欄位 | 型別 | 限制 | 說明 |
|------|------|------|------|
| id | Integer | Primary Key, Auto Increment | 唯一識別碼 |
| category_id | Integer | Foreign Key → Category.id | 所屬分類 |
| phrase_zh | String(200) | Not Null | 中文說明（如：點餐） |
| phrase_en | String(300) | Not Null | 英文短語（如：I'd like to order, please.） |
| phrase_ja | String(300) | Not Null | 日文短語（如：注文をお願いします。） |
| pronunciation_en | String(300) | Nullable | 英文發音標示（IPA 或拼音） |
| pronunciation_ja | String(300) | Nullable | 日文平假名標注 |
| image_url | String(500) | Nullable | 情境圖片的 URL |
| sort_order | Integer | Default 0 | 同分類內的排列順序 |
| created_at | DateTime | Default now() | 建立時間 |
| updated_at | DateTime | Default now(), On Update | 最後更新時間 |

---

### 2.3 上下文例句（ContextExample）
每個短語可有多個上下文例句。

| 欄位 | 型別 | 限制 | 說明 |
|------|------|------|------|
| id | Integer | Primary Key, Auto Increment | 唯一識別碼 |
| phrase_id | Integer | Foreign Key → Phrase.id | 所屬短語 |
| sentence_en | Text | Not Null | 英文例句 |
| sentence_zh | Text | Not Null | 中文翻譯 |
| sentence_ja | Text | Nullable | 日文例句 |
| sort_order | Integer | Default 0 | 排列順序 |

---

## 3. API 端點規格

### 基礎設定
- **Base Path**：`/api`
- **Response 格式**：JSON
- **字元編碼**：UTF-8
- **API 文件**：自動產生於 `/docs`（Swagger UI）及 `/redoc`

---

### 3.1 取得所有分類

**`GET /api/categories`**

**描述：** 取得所有旅遊場景分類清單，依 `sort_order` 升冪排列。

**Query 參數：** 無

**成功回應 `200 OK`：**
```json
[
  {
    "id": 1,
    "slug": "airport",
    "name_zh": "機場",
    "name_en": "Airport",
    "name_ja": "空港",
    "description_zh": "機場出入境、登機相關用語",
    "icon": "✈️",
    "sort_order": 1
  }
]
```

---

### 3.2 取得短語列表

**`GET /api/phrases`**

**描述：** 取得短語列表，支援分類篩選與關鍵字搜尋，並支援分頁。

**Query 參數：**

| 參數 | 型別 | 必填 | 說明 |
|------|------|------|------|
| category_slug | string | 否 | 依分類 slug 篩選（如 `airport`） |
| search | string | 否 | 關鍵字搜尋（比對 phrase_zh / phrase_en / phrase_ja） |
| page | integer | 否 | 頁碼，預設 1（最小值 1） |
| page_size | integer | 否 | 每頁筆數，預設 20（最大值 100） |

**成功回應 `200 OK`：**
```json
{
  "total": 85,
  "page": 1,
  "page_size": 20,
  "items": [
    {
      "id": 1,
      "category": {
        "id": 1,
        "slug": "airport",
        "name_zh": "機場",
        "name_en": "Airport",
        "name_ja": "空港",
        "icon": "✈️"
      },
      "phrase_zh": "辦理登機手續",
      "phrase_en": "I'd like to check in, please.",
      "phrase_ja": "チェックインをお願いします。",
      "pronunciation_en": "/aɪd laɪk tə tʃɛk ɪn/",
      "pronunciation_ja": "チェックインをおねがいします",
      "image_url": "https://example.com/images/airport_checkin.jpg",
      "sort_order": 1
    }
  ]
}
```

**錯誤回應 `422 Unprocessable Entity`：**
```json
{
  "detail": [
    {
      "loc": ["query", "page"],
      "msg": "ensure this value is greater than 0",
      "type": "value_error.number.not_gt"
    }
  ]
}
```

---

### 3.3 取得單一短語詳情

**`GET /api/phrases/{phrase_id}`**

**描述：** 取得指定 ID 的短語完整資訊，包含所屬分類與所有上下文例句。

**路徑參數：**

| 參數 | 型別 | 說明 |
|------|------|------|
| phrase_id | integer | 短語的唯一識別碼 |

**成功回應 `200 OK`：**
```json
{
  "id": 1,
  "category": {
    "id": 1,
    "slug": "airport",
    "name_zh": "機場",
    "name_en": "Airport",
    "name_ja": "空港",
    "icon": "✈️"
  },
  "phrase_zh": "辦理登機手續",
  "phrase_en": "I'd like to check in, please.",
  "phrase_ja": "チェックインをお願いします。",
  "pronunciation_en": "/aɪd laɪk tə tʃɛk ɪn/",
  "pronunciation_ja": "チェックインをおねがいします",
  "image_url": "https://example.com/images/airport_checkin.jpg",
  "context_examples": [
    {
      "id": 1,
      "sentence_en": "Excuse me, I'd like to check in, please. Here is my passport.",
      "sentence_zh": "不好意思，我要辦理登機，這是我的護照。",
      "sentence_ja": "すみません、チェックインをお願いします。パスポートはこちらです。",
      "sort_order": 1
    }
  ],
  "related_phrases": [
    {
      "id": 2,
      "phrase_zh": "托運行李",
      "phrase_en": "I'd like to check my luggage.",
      "phrase_ja": "荷物を預けたいです。",
      "image_url": "https://example.com/images/airport_luggage.jpg"
    }
  ],
  "created_at": "2024-01-15T08:00:00Z",
  "updated_at": "2024-01-15T08:00:00Z"
}
```

**錯誤回應 `404 Not Found`：**
```json
{
  "detail": "Phrase not found"
}
```

---

### 3.4 健康檢查

**`GET /health`**

**描述：** 確認服務運行狀態，供部署平台（如 Cloud Run）探測使用。

**成功回應 `200 OK`：**
```json
{
  "status": "ok"
}
```

---

## 4. 錯誤處理規範

| HTTP 狀態碼 | 使用情境 |
|-------------|---------|
| 200 OK | 請求成功 |
| 422 Unprocessable Entity | 請求參數驗證失敗（FastAPI 預設行為） |
| 404 Not Found | 指定資源不存在 |
| 500 Internal Server Error | 伺服器內部錯誤（包含資料庫連線失敗） |

---

## 5. CORS 設定

允許前端應用跨域存取，設定如下：

| 項目 | 值 |
|------|---|
| 允許的來源（Origins） | 開發：`http://localhost:5173`；生產：前端部署的網域 |
| 允許的 HTTP 方法 | `GET`, `OPTIONS` |
| 允許的 Headers | `Content-Type` |
| 最大預檢快取時間 | 600 秒 |

---

## 6. 環境變數

| 變數名稱 | 說明 | 範例值 |
|---------|------|--------|
| `DATABASE_URL` | 資料庫連線字串 | `sqlite:///./dev.db` 或 `postgresql://user:pass@host/dbname` |
| `ALLOWED_ORIGINS` | CORS 允許的前端來源（逗號分隔） | `http://localhost:5173,https://myapp.web.app` |
| `APP_ENV` | 應用執行環境 | `development` 或 `production` |

---

## 7. 資料庫遷移流程（Alembic）

```bash
# 初始化遷移環境（僅首次）
alembic init alembic

# 根據 Model 變更自動產生遷移腳本
alembic revision --autogenerate -m "描述變更"

# 套用遷移至資料庫
alembic upgrade head

# 回滾上一版
alembic downgrade -1
```

---

## 8. 初始資料種子（Seed Data）

專案應提供 `seed.py` 或 SQL 腳本，初始化以下資料以供開發與測試：

**分類（最少 7 個）：**
機場、飯店、餐廳、交通、購物、觀光、緊急狀況

**每個分類至少 5 個短語範例，每個短語至少 1 個上下文例句。**

---

## 9. 專案目錄結構（建議）

```
backend/
├── app/
│   ├── main.py              # FastAPI 應用入口，設定 CORS、路由
│   ├── database.py          # 資料庫連線設定（SQLAlchemy engine, session）
│   ├── models/              # SQLAlchemy ORM 模型
│   │   ├── category.py
│   │   ├── phrase.py
│   │   └── context_example.py
│   ├── schemas/             # Pydantic 請求 / 回應 Schema
│   │   ├── category.py
│   │   └── phrase.py
│   ├── routers/             # API 路由
│   │   ├── categories.py
│   │   └── phrases.py
│   └── crud/                # 資料庫查詢邏輯
│       ├── category.py
│       └── phrase.py
├── alembic/                 # 資料庫遷移
├── seed.py                  # 種子資料腳本
├── requirements.txt         # Python 依賴套件
├── Dockerfile               # 容器化設定
└── .env.example             # 環境變數範本
```

---

## 10. 容器化（Docker）

`Dockerfile` 基本設定原則：
- 基礎映像：`python:3.11-slim`
- 應用監聽 Port：`8000`
- 啟動命令：`uvicorn app.main:app --host 0.0.0.0 --port 8000`
- 生產環境建議加入 `--workers 2` 以支援並發請求
