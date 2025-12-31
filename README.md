# ArvivaCRM 🚀

<p align="center">
	<b>Full‑stack CRM (Angular SSR + Express + MySQL/Sequelize + Socket.IO)</b><br/>
	<i>Kurumsal ölçekte satış/lead yönetimi, Kanban, iş emirleri, sosyal mesajlaşma ve Meta/WhatsApp/Instagram entegrasyonları</i>
</p>

<p align="center">
	<img alt="Angular" src="https://img.shields.io/badge/Angular-19-red?logo=angular" />
	<img alt="Node" src="https://img.shields.io/badge/Node.js-18+-3c873a?logo=node.js&logoColor=white" />
	<img alt="Express" src="https://img.shields.io/badge/Express-4-black?logo=express" />
	<img alt="MySQL" src="https://img.shields.io/badge/MySQL-8-blue?logo=mysql&logoColor=white" />
	<img alt="Sequelize" src="https://img.shields.io/badge/Sequelize-6-52B0E7?logo=sequelize" />
	<img alt="Socket.IO" src="https://img.shields.io/badge/Socket.IO-4-black?logo=socket.io" />
	<img alt="SSR" src="https://img.shields.io/badge/SSR-Angular%20SSR-7b1fa2" />
</p>

<details>
	<summary><b>✨ Hızlı İçindekiler</b></summary>

	- [Genel Bakış](#genel-bakış)
	- [Mimari](#mimari)
	- [Özellikler (Modül Bazlı)](#özellikler-modül-bazlı)
	- [Frontend Route Yapısı](#frontend-route-yapısı)
	- [Backend API Route Yapısı](#backend-api-route-yapısı)
	- [Meta / Instagram / WhatsApp (Webhook + Mesajlaşma)](#meta--instagram--whatsapp-webhook--mesajlaşma)
	- [Yetkilendirme Modeli](#yetkilendirme-modeli)
	- [System Config & Runtime Secret Yönetimi](#system-config--runtime-secret-yönetimi)
	- [Gerçek Zamanlı Event Sistemi (Socket.IO)](#gerçek-zamanlı-event-sistemi-socketio)
	- [Dosya & Medya Yönetimi](#dosya--medya-yönetimi)
	- [Yedekleme & Operasyon](#yedekleme--operasyon)
	- [Kurulum & Çalıştırma](#kurulum--çalıştırma)
	- [Build / SSR / Prerender](#build--ssr--prerender)
	- [Güvenlik Notları](#güvenlik-notları)
	- [Klasör Yapısı](#klasör-yapısı)

</details>

---

## Genel Bakış

ArvivaCRM; **lead ve müşteri yönetimi**, **Kanban süreçleri**, **iş emri (work order) operasyonları**, **notlar ve hatırlatıcılar**, **kampanya analitiği**, **Google Drive tabanlı dosyalama** ve **çok kanallı mesajlaşma** (WhatsApp/Instagram/Messenger) gibi modülleri tek bir platformda birleştirir.

Bu repo; hem **Angular (SSR) web uygulamasını** hem de aynı process içerisinde çalışan **Express API katmanını** barındırır.

---

## Mimari

```mermaid
flowchart TB
	U[👤 Kullanıcı / Tarayıcı] -->|HTTPS| A[Angular SSR + SPA]
	A -->|/api/*| API[Express API]
	API -->|Sequelize| DB[(MySQL)]
	A <--> |Socket.IO| WS[(Realtime Gateway)]
	EXT[Meta / Instagram / WhatsApp Webhooks] -->|/api/*/webhook| API
	API --> FS[(public/uploads)]
```

**Ana bileşenler**
- **Angular 19 SSR**: Server-side rendering + hydration
- **Express**: REST API + webhook endpoint’leri
- **Sequelize**: MySQL ORM
- **Socket.IO**: Kanban, müşteri güncellemeleri ve chat için realtime yayın
- **Runtime Config**: Prod secrets yönetimi (DB + şifreleme + env fallback)

---

## Özellikler (Modül Bazlı)

### 🧭 Dashboard
- KPI/istatistik endpoint’leri (ör: günlük randevular, hatırlatıcılar, aktiviteler, kullanıcı logları)
- Rol/permission bazlı görünürlük

### 👥 Customers & Lead Yönetimi
- Müşteri listesi ve detay sayfası
- Telefon bazlı hızlı arama, kanban kartı ile ilişkilendirme
- Sorumlu atama / sorumlu bazlı filtreleme
- Müşteri notları, yorumları, hatırlatıcıları, aktiviteleri

### 🧩 Kanban Süreç Yönetimi
- Board/column/card modeli
- Kart taşıma (move), görüntülendi işaretleme, export, bulk move
- Realtime yayın ile board güncellemeleri

### 🗓️ Opportunities / Appointments
- Randevu listesi, hatırlatıcılarla ilişki

### 🧾 Work Orders
- İş emri listesi
- Takvim görünümü ve kanban görünümü
- Workflow tanımları ve sorumluların tamamladığı adımlar

### 💬 Chat (Internal + Social Unified Inbox)
- Kullanıcılar arası internal chat (conversation + participants + message)
- Sosyal kanalların tek bir “inbox” altında ele alınması (WhatsApp/IG/Messenger)
- Okundu/typing/online gibi realtime event’ler
- Dosya yükleme/ek (chat upload) altyapısı

### 📊 Reports / Analysis
- Dashboard analitikleri
- Plot/satış analitiği uçları
- Kampanya analitiği (Meta insights, lead bağlama vb.)

### 🗂️ Inventory (Stok / Plot)
- Plot CRUD + bulk update
- Project bazlı plot listesi

### ☁️ Arviva Drive (Google Drive Entegrasyonu)
- Drive üzerinde listeleme/yükleme/taşıma/silme gibi endpoint’ler

---

## Frontend Route Yapısı

Route’lar Angular tarafında lazy-load (loadComponent) edilir ve **AuthGuard + Permission Guard** ile korunur.

Kaynak: `src/app/app.routes.ts`

| Route | Guard | Resolver | Amaç |
|---|---|---|---|
| `/dashboard` | `AuthGuard` + `dashboardGuard` | `dashboardResolver` | Ana panel |
| `/campaign-analytics` | `AuthGuard` + `campaignsGuard` | - | Kampanya analitiği |
| `/stok` | `AuthGuard` + `inventoryGuard` | `PlotResolverService` | Stok/plot |
| `/sales-analysis` | `AuthGuard` + `analysisGuard` | - | Satış analizi |
| `/kanban` | `AuthGuard` + `kanbanGuard` | `KanbanResolverService` | Kanban |
| `/reports` | `AuthGuard` + `reportsGuard` | - | Raporlar |
| `/customers` | `AuthGuard` + `customersGuard` | `CustomerResolverService` | Müşteriler |
| `/customers/:id` | `AuthGuard` + `customersGuard` | - | Müşteri detayı |
| `/opportunities` | `AuthGuard` + `appointmentsGuard` | `AppointmentResolverService` | Fırsatlar/Randevular |
| `/notes` | `AuthGuard` + `notesGuard` | - | Notlar |
| `/project-management` | `AuthGuard` + `projectsGuard` | - | Proje yönetimi |
| `/user-management` | `AuthGuard` + `userManagementGuard` | - | Kullanıcı/rol yönetimi |
| `/chat` | `AuthGuard` + `chatGuard` | - | Mesajlaşma |
| `/work-orders` | `AuthGuard` + `workOrdersGuard` | - | İş emirleri |
| `/system-config` | `AuthGuard` + `settingsGuard` | - | Sistem ayarları |
| `/arviva-drive` | `AuthGuard` + `filesGuard` | - | Drive |
| `/giris-yap` | `GuestGuard` | - | Login |
| `/profile` | `AuthGuard` | - | Profil |
| `/video-call/:roomId` | - | - | Video görüşme odası |
| `/auth/callback` | - | - | OAuth callback |
| `/error/404`, `/error/500` | - | - | Hata sayfaları |

---

## Backend API Route Yapısı

Sunucu entry: `src/server.ts`

Tüm API’ler `/api` altında mount edilir:

| Base Path | Router | Not |
|---|---|---|
| `/api` | `general.router.ts` | Website lead girişleri + temel listeler |
| `/api/user` | `user.router.ts` | Login/refresh/logout/profile/avatar |
| `/api/permissions` | `permissions.router.ts` | Permission okuma/yazma |
| `/api/customers` | `customer.router.ts` | Müşteri CRUD + ilişkiler |
| `/api/customerviewer` | `customerviewer.router.ts` | Detay ekranı yoğun uçlar |
| `/api/kanban` | `kanban.router.ts` | Board/column/card aksiyonları |
| `/api/appointments` | `appointments.router.ts` | Randevu işlemleri |
| `/api/activities` | `activities.router.ts` | Aktivite + event akışı |
| `/api/notes` | `notes.router.ts` | Notlar |
| `/api/inventory` | `inventory.router.ts` | Plot/stock |
| `/api/work-orders` | `work-orders.router.ts` | İş emirleri |
| `/api/work-order-workflows` | `work-order-workflow.router.ts` | Workflow tanımları |
| `/api/chat` | `chat.router.ts` | Chat + unified inbox |
| `/api/notifications` | `notification.router.ts` | Bildirim & hatırlatıcı uçları |
| `/api/analysis` | `analysis.router.ts` | Analiz rapor uçları |
| `/api/plot-sales-analysis` | `plot-sales-analysis.router.ts` | Plot/satış analizi |
| `/api/campaign-analytics` | `campaign-analytics.router.ts` | Meta insights vb. |
| `/api/meta` | `meta.router.ts` | Meta Leadgen + Messenger webhook |
| `/api/instagram` | `instagram.router.ts` | Instagram webhook + mesaj |
| `/api/whatsapp` | `whatsapp.router.ts` | WhatsApp webhook + mesaj |
| `/api/google-drive` | `google-drive.router.ts` | Drive işlemleri |
| `/api/system-config` | `system-config.router.ts` | Runtime config + backup + restart |

> Not: API’lerin çoğu `authenticateToken` ile korunur.

---

## Meta / Instagram / WhatsApp (Webhook + Mesajlaşma)

Webhook endpoint’leri server start sırasında loglanır:
- `.../api/meta/webhook`
- `.../api/instagram/webhook`
- `.../api/whatsapp/webhook`

Bu katman; gelen event’leri **müşteri oluşturma/güncelleme**, **Kanban kartı üretme**, **bildirim üretme**, **CAPI event gönderme** ve **realtime broadcast** akışlarına bağlar.

**Önemli tasarım kararı**
- Token/secret gibi değerler **hard-code edilmez** ✅
- Tercih edilen kaynak: **System Config (AppConfig tablosu)** → **ENV fallback**

---

## Yetkilendirme Modeli

### 🔐 Authentication (JWT)
- Token; header (`Authorization: Bearer ...`), cookie (`token`) veya query (`?token=...`) üzerinden okunabilir.
- Refresh token cookie üzerinden doğrulanır ve access token otomatik yenilenebilir.

### 🧾 Authorization (Permission Modules)
Frontend tarafında modül bazlı permission guard uygulanır:
- Kaynak: `src/app/guards/permission.guard.ts`
- Modüller: `dashboard`, `customers`, `appointments`, `campaigns`, `projects`, `work-orders`, `kanban`, `inventory`, `reports`, `analysis`, `settings`, `user-management`, `chat`, `notes`, `files`, ...

Backend tarafında permission yönetimi:
- `/api/permissions/me`
- `/api/permissions/user/:userId`
- `/api/permissions/user/:userId/bulk`

---

## System Config & Runtime Secret Yönetimi

Amaç: Prod ortamında **token/secret yönetimini** UI üzerinden güvenli şekilde yapmak ve kod deploy’u gerektirmeden güncellemek.

Kaynak: `src/utils/runtime-config.ts` + `src/routes/system-config.router.ts`

Özellikler:
- Tanımlı secret listesi (Meta/IG/WhatsApp) + maskeli görüntüleme
- DB içinde AES-256-GCM ile şifreleme (opsiyonel): `APP_CONFIG_ENCRYPTION_KEY`
- ENV fallback (bootstrapping ve acil durum)

API:
- `GET /api/system-config/definitions` (admin)
- `PUT /api/system-config/:key` (admin)
- `DELETE /api/system-config/:key` (admin)

---

## Gerçek Zamanlı Event Sistemi (Socket.IO)

Server Socket.IO üzerinde örnek event’ler:
- Room: `join-room`, `leave-room`
- Chat: `chat:join-conversation`, `chat:typing-start`, `chat:message-read` ...
- Broadcast: `customer-update`, `kanban-update`, `whatsapp:new-message` ...

Kaynaklar:
- Socket server: `src/server.ts`
- Broadcast orchestration: `src/middlewares/broadcast.ts`

---

## Dosya & Medya Yönetimi

- Statik servis: `/uploads` → `public/uploads`
- Avatar upload: `POST /api/user/profile/avatar` (multer)

Klasörler:
- `public/uploads/avatars`
- `public/uploads/chat`
- `public/uploads/workflow`

---

## Yedekleme & Operasyon

### 💾 Otomatik mysqldump
- Sunucu açılışında otomatik backup scheduler başlar (saatlik).
- Kaynak: `src/utils/mysqldump-backup.ts` + `src/server.ts`

### 🧰 Manuel Backup (Admin)
- `POST /api/system-config/backup`
- `GET /api/system-config/backup/download`

### ♻️ Restart / Rebuild (Admin)
- `POST /api/system-config/restart`
	- Lokal: `src/restart.trigger` yazıp nodemon ile rebuild tetikler
	- Prod (PM2): `pm2 stop 0 -> ng build -> pm2 start 0`

---

## Kurulum & Çalıştırma

### 1) Gereksinimler
- Node.js 18+
- MySQL 8+

### 2) ENV ayarları
`.env.example` dosyasını `.env` olarak kopyalayın ve doldurun.

### 3) Install
```bash
npm install
```

### 4) Dev (Angular dev-server)
```bash
npm run start
```

### 5) SSR Prod run
```bash
npm run build
npm run serve:ssr:arvivacrm
```

> Not: Cookie `Secure=true` kullandığı için iframe/SSO senaryolarında **HTTPS** gereklidir.

---

## Build / SSR / Prerender

- Angular build config: `angular.json`
- SSR entry: `src/server.ts`
- Server bootstrap: `src/main.server.ts`
- Prerender route listesi: `prerender-routes.txt`

---

## Güvenlik Notları

✅ Bu repoda secret değerler **kod içine gömülmemelidir**:
- DB erişimi: `DB_HOST/DB_USER/DB_PASS/DB_NAME/DB_PORT`
- JWT secret’lar: `JWT_SECRET`, `JWT_REFRESH_SECRET`
- Entegrasyon token’ları: System Config (AppConfig) veya ENV

Önerilen minimumlar:
- Reverse proxy arkasında `trust proxy` aktif (sunucuda mevcut)
- HTTPS (cookie `SameSite=None` + `Secure` gereksinimi)
- CORS politikasını prod’da daraltma (şu an geniş)

---

## Klasör Yapısı

```
src/
	app/                 # Angular UI
	routes/              # Express routers (/api/*)
	models/              # Sequelize modelleri
	middlewares/         # Auth + broadcast + vb.
	utils/               # Yedekleme, runtime config, job scheduler, helpers
public/
	uploads/             # Avatar/chat/workflow upload alanı
backups/               # mysqldump çıktıları (runtime)
```

---

## Referans / İnceleme Notu 🏢

Bu README; mimariyi ve sistem davranışını inceleme amaçlı detaylı anlatır. Üretim ortamına alınmadan önce güvenlik, logging ve CORS gibi başlıkların kurum standartlarına göre gözden geçirilmesi önerilir.
