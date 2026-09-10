# Car Rental

一個使用 **Laravel** 開發的租車網站專案，包含車輛搜尋與預約、Google 會員登入、會員訂單管理，以及租車後台管理功能。

> 本 README 的安裝流程只整理目前 Repository 中可以確認的部分。  
> 目前專案 **沒有包含完整的租車系統資料表 Migration，也依賴額外的管理員驗證服務**，因此 Clone 後不能保證只執行 `php artisan migrate` 就能完整啟動所有功能。

---

## 專案功能

### 前台租車

- 首頁與公告
- 依條件搜尋可租車輛
- 查詢車輛已被預約的日期
- 選擇取車 / 還車日期與時間
- 選擇取車地點
- 選擇保險方案
- 自動計算租車費用
- 建立租車預約
- 避免同一車輛預約時間衝突

### 會員功能

- Google OAuth 登入
- 新會員手機號碼驗證流程
- 會員中心
- 查看個人租車訂單
- 修改訂單保險方案
- 取消符合條件的訂單
- 更新會員手機號碼
- Session 登入狀態管理

### 管理後台

後台主要提供：

- 訂單管理
- 訂單狀態更新
- 車輛管理
- 車型管理
- 保險方案管理
- 取還車地點管理
- 會員資料查看
- 網站圖片管理

管理區主要路徑：

```text
/manager
```

登入頁路徑則由 `.env` 的 `ADMIN_URL` 控制，未設定時預設為：

```text
/admin/login
```

---

## 主要資料

從目前程式碼可以確認租車系統會使用下列主要資料：

```text
member
car
model
rental
insurance
location
```

其中：

- `member`：會員資料
- `car`：實際車輛與車牌
- `model`：車型、品牌、排氣量、圖片等
- `rental`：租車訂單
- `insurance`：保險方案與費用
- `location`：取車 / 還車地點

---

## 技術

目前 Repository 使用：

- PHP 8.2+
- Laravel 12
- Laravel Socialite
- Blade
- Vite
- Tailwind CSS 4
- JavaScript
- SQLite（`.env.example` 預設設定）

前端版型部分參考 ThemeWagon 的 **CarRentals** Template。

---

# 安裝

## 1. Clone Repository

```bash
git clone https://github.com/LiaoZike/Car_Rental.git
cd Car_Rental
```

---

## 2. 安裝 PHP 套件

```bash
composer install
```

---

## 3. 建立 `.env`

### Windows PowerShell

```powershell
Copy-Item .env.example .env
```

### Linux / macOS

```bash
cp .env.example .env
```

接著產生 Laravel Application Key：

```bash
php artisan key:generate
```

---

## 4. 安裝前端套件

```bash
npm install
```

開發模式：

```bash
npm run dev
```

或建立正式靜態資源：

```bash
npm run build
```

---

# Database 注意事項

目前 `.env.example` 預設：

```env
DB_CONNECTION=sqlite
```

Laravel 預設資料庫可以建立在：

```text
database/database.sqlite
```

如果檔案不存在，可以自行建立空檔案。

Windows PowerShell：

```powershell
New-Item database/database.sqlite -ItemType File
```

Linux / macOS：

```bash
touch database/database.sqlite
```

接著可以執行：

```bash
php artisan migrate
```

但是請注意：

> **目前 Repository 內的 Migration 只有 Laravel 預設的 `users`、`cache`、`jobs` 等資料表。**

租車系統實際需要的：

```text
member
car
model
rental
insurance
location
```

等資料表 **目前沒有對應的 Migration**。

因此：

```bash
php artisan migrate
```

只能建立 Laravel 本身需要的基本資料表，**無法建立完整租車系統資料庫**。

若要完整執行本專案，需要：

1. 匯入原本開發時使用的租車資料庫 Schema / SQL；或
2. 依目前 Model 與 SQL 查詢重新建立對應 Migration。

在資料表尚未補齊前，車輛搜尋、會員、預約、訂單與後台等依賴資料庫的功能可能會發生 SQL Error。

---

# Google 登入設定

本專案透過 Laravel Socialite 使用 Google OAuth。

`config/services.php` 會讀取：

```env
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT=
```

這些欄位目前沒有完整列在 `.env.example` 中，因此若需要使用 Google 登入，請自行加入 `.env`。

本機開發範例：

```env
GOOGLE_CLIENT_ID=YOUR_CLIENT_ID
GOOGLE_CLIENT_SECRET=YOUR_CLIENT_SECRET
GOOGLE_REDIRECT=http://127.0.0.1:8000/auth/google/callback
```

Google OAuth Console 中的 Authorized Redirect URI 也必須與 `GOOGLE_REDIRECT` 一致。

登入流程：

```text
/auth/google/redirect
        ↓
Google OAuth
        ↓
/auth/google/callback
        ↓
既有會員 → 登入
新會員   → 手機驗證流程
```

> Google Client ID / Client Secret 不應提交至 Git Repository。

---

# 管理員登入注意事項

這個專案的管理員驗證 **不是單純使用 Laravel 本身的帳號資料表**。

`AdminAuthController` 會呼叫額外的驗證服務：

```text
AUTH_URL/admin/login
AUTH_URL/admin/logout
```

並使用 JWT Token 保護 `/manager` 後台。

因此若需要管理後台功能，`.env` 還需要設定：

```env
AUTH_URL=http://127.0.0.1:5000
```

但目前程式中的管理員驗證流程仍有一部分直接連向：

```text
http://127.0.0.1:5000/admin/verify
```

所以原本開發環境應該還需要一個運行於 **Port 5000** 的外部 Auth Server。

> 該 Auth Server 不在目前這個 Repository 中，因此只 Clone `Car_Rental` 無法直接使用完整的管理員登入功能。

---

# 啟動 Laravel

完成基本設定後：

```bash
php artisan serve
```

預設網址：

```text
http://127.0.0.1:8000
```

如果前端使用 Vite 開發模式，請另外開一個 Terminal：

```bash
npm run dev
```

也可以使用 Laravel 專案目前提供的 Composer Script：

```bash
composer run dev
```

它會同時啟動 Laravel Server、Queue、Pail Log 與 Vite。

---

# 主要 Route

### 使用者端

```text
/                           首頁
/notice                     公告
/rental/search              租車搜尋
/rental/search/car/{id}     查詢車輛可租日期
/rental/reserve             建立租車預約
/member                     會員中心
```

### Google 登入

```text
/auth/google/redirect
/auth/google/callback
/auth/google/verify
/auth/google/logout
```

### 管理端

```text
/manager
/manager/rental/{status}
/manager/car-management
/manager/insurance-management
/manager/store-info
/manager/members
/manager/pictureManagement
```

---

# 專案結構

```text
Car_Rental/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   └── Middleware/
│   └── Models/
├── config/
├── database/
│   ├── migrations/
│   └── seeders/
├── public/
├── resources/
│   └── views/
├── routes/
│   └── web.php
├── storage/
├── .env.example
├── composer.json
└── package.json
```

---

## Reference

前端模板參考：

- ThemeWagon - CarRentals Website Template

部分程式碼與文字開發過程使用 ChatGPT 協助。

---

## 備註

這是一個租車網站實作專案，重點包含租車搜尋與預約流程、Google OAuth 會員系統、租車訂單處理，以及前後台管理功能。

目前 Repository 並未包含原始租車資料庫完整 Schema 與獨立的管理員 Auth Server，因此 README 僅提供可由目前專案檔案確認的安裝流程與環境需求。
