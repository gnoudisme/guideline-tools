# Hướng Dẫn Toàn Tập Render — Deploy Web App & Database Dễ Dàng

> **Cấp độ:** Người mới bắt đầu → deploy được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10

---

## Mục Lục

1. [Render Là Gì?](#1-render-là-gì)
2. [Tạo Tài Khoản](#2-tạo-tài-khoản)
3. [Web Services — Deploy Backend](#3-web-services--deploy-backend)
4. [Static Sites — Deploy Frontend](#4-static-sites--deploy-frontend)
5. [PostgreSQL](#5-postgresql)
6. [Redis](#6-redis)
7. [Background Workers](#7-background-workers)
8. [Cron Jobs](#8-cron-jobs)
9. [Environment Variables](#9-environment-variables)
10. [Custom Domains & HTTPS](#10-custom-domains--https)
11. [Blueprints — render.yaml](#11-blueprints--renderyaml)
12. [Logs & Metrics](#12-logs--metrics)
13. [Dự Án Thực Tế: Deploy Full-Stack](#13-dự-án-thực-tế-deploy-full-stack)
14. [Pricing Chi Tiết & Free Tier](#14-pricing-chi-tiết--free-tier)
15. [So Sánh Với Đối Thủ](#15-so-sánh-với-đối-thủ)
16. [Troubleshooting](#16-troubleshooting)
17. [Best Practices](#17-best-practices)
18. [FAQ](#18-faq)

---

## 1. Render Là Gì?

### 1.1. Định Nghĩa

**Render** là nền tảng cloud (PaaS) để deploy web app, API, database, cron job... với vài click. Nổi tiếng với **free tier hào phóng** cho web service (750 giờ/tháng).

### 1.2. Các Loại Service Trên Render

| Loại | Dùng cho | Ví dụ |
|------|----------|-------|
| **Web Service** | Backend API, web app chạy server | Express, Django, Rails, Go API |
| **Static Site** | Frontend tĩnh | React/Vite build, HTML/CSS, docs |
| **PostgreSQL** | Database quan hệ | Mọi app cần SQL |
| **Redis** | Cache, session, queue | Cache API, BullMQ |
| **Background Worker** | Job chạy nền | Xử lý ảnh, gửi email |
| **Cron Job** | Chạy theo lịch | Backup DB, báo cáo hàng ngày |
| **Private Service** | Service nội bộ (không public) | Microservice gọi nội bộ |

### 1.3. So Sánh Nhanh

| Tiêu chí | Render | Railway | Vercel | Heroku |
|----------|--------|---------|--------|--------|
| Free web service | ✅ 750h/tháng (sleep khi idle) | $5 credit | ✅ Rộng rãi | ❌ Không còn |
| Free PostgreSQL | ✅ 90 ngày | $5 credit | ❌ | ❌ |
| Deploy từ GitHub | ✅ | ✅ | ✅ | ✅ |
| Docker support | ✅ | ✅ | ❌ | ✅ |
| Cron job free | ✅ | ⚠️ | ❌ | ❌ |
| Độ dễ dùng | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

---

## 2. Tạo Tài Khoản

1. Vào https://render.com → **Get Started**
2. Đăng ký bằng **GitHub** (khuyên dùng — để deploy từ repo), GitLab, Google, hoặc email
3. Authorize Render truy cập GitHub
4. Vào Dashboard → sẵn sàng tạo service!

> 💡 Đăng ký bằng GitHub giúp bước deploy sau nhanh hơn (không cần nhập lại credentials).

---

## 3. Web Services — Deploy Backend

### 3.1. Chuẩn Bị Code

Render tự phát hiện ngôn ngữ qua file đặc trưng (giống Railway). **Bắt buộc:** app listen ở `process.env.PORT` và host `0.0.0.0`.

**Node.js (Express):**

`package.json`:
```json
{
  "name": "my-api",
  "scripts": {
    "start": "node server.js",
    "build": "npm run build-client"
  },
  "engines": { "node": "20.x" },
  "dependencies": { "express": "^4.18.2" }
}
```

`server.js`:
```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;  // ← BẮT BUỘC dùng process.env.PORT

app.get('/', (req, res) => res.json({ message: 'Hello từ Render! 🎨' }));
app.get('/health', (req, res) => res.json({ status: 'ok' }));

app.listen(PORT, '0.0.0.0', () => {
  console.log(`Chạy ở port ${PORT}`);
});
```

**Python (Flask + Gunicorn):**

`requirements.txt`:
```
flask==3.0.0
gunicorn==21.2.0
psycopg2-binary==2.9.9
```

`app.py`:
```python
import os
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify(message='Hello từ Render! 🎨')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=int(os.environ.get('PORT', 5000)))
```

### 3.2. Các Bước Tạo Web Service

1. Dashboard → **New +** → **Web Service**
2. **Connect** repo GitHub của bạn
3. Điền thông tin:

| Trường | Điền gì | Ví dụ |
|--------|---------|-------|
| **Name** | Tên service | `my-api` |
| **Region** | Gần user nhất | `Singapore` (gần VN nhất!) |
| **Branch** | Nhánh deploy | `main` |
| **Runtime** | Ngôn ngữ | `Node` / `Python` / `Go` / `Docker` |
| **Build Command** | Lệnh build | `npm install` hoặc `pip install -r requirements.txt` |
| **Start Command** | Lệnh chạy | `npm start` hoặc `gunicorn app:app` |
| **Instance Type** | Gói máy | `Free` (lúc đầu) |

4. Bấm **Create Web Service** → Render bắt đầu build & deploy (2-5 phút)
5. Xong → bạn có URL: `https://my-api.onrender.com`

> 💡 **Chọn Region Singapore** cho app phục vụ user Việt Nam — latency thấp nhất (~30-50ms từ VN).

### 3.3. Auto-Deploy

Mặc định: mỗi lần push lên branch đã chọn → Render tự deploy lại.

**Tắt:** Service → **Settings** → **Auto-Deploy** → chọn `No`.

**Deploy tay:** Service → **Manual Deploy** → **Deploy latest commit**.

### 3.4. Deploy Bằng Docker

Nếu có `Dockerfile` trong repo, chọn **Runtime: Docker**:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
# Render tự inject PORT, không cần EXPOSE cũng được
CMD ["node", "server.js"]
```

> ⚠️ Render free tier **không** hỗ trợ Docker (cần gói trả phí). Dùng native runtime nếu muốn free.

### 3.5. Health Check

Render tự ping app để biết nó còn sống. Trong **Settings** → **Health Check Path**: điền `/health`.

Nếu app không trả lời 200 OK → Render đánh dấu **unhealthy** và có thể restart.

---

## 4. Static Sites — Deploy Frontend

### 4.1. Khi Nào Dùng Static Site?

- Frontend React/Vue/Vite build ra HTML/CSS/JS tĩnh
- Blog, docs, landing page
- **Miễn phí 100%**, không bị sleep, bandwidth 100GB/tháng

### 4.2. Các Bước

1. **New +** → **Static Site** → Connect repo
2. Điền:

| Trường | Ví dụ (Vite) | Ví dụ (Create React App) |
|--------|-------------|-------------------------|
| **Build Command** | `npm run build` | `npm run build` |
| **Publish Directory** | `dist` | `build` |

3. **Create** → có URL `https://my-app.onrender.com`

### 4.3. Publish Directory Theo Framework

| Framework | Build Command | Publish Directory |
|-----------|--------------|-------------------|
| Vite | `npm run build` | `dist` |
| Create React App | `npm run build` | `build` |
| Vue CLI | `npm run build` | `dist` |
| Next.js (static export) | `npm run build` | `out` |
| Hugo | `hugo` | `public` |
| Docusaurus | `npm run build` | `build` |

### 4.4. SPA Routing (React Router...)

Nếu dùng React Router (URL như `/users/123`), cần redirect mọi request về `index.html`:

Tạo file `public/_redirects`:
```
/*    /index.html   200
```

Hoặc trong **Settings** → **Redirects/Rewrites** → thêm rule.

### 4.5. Biến Môi Trường Cho Frontend

Với Vite, biến phải bắt đầu bằng `VITE_`:
```
VITE_API_URL=https://my-api.onrender.com
```

> ⚠️ Đổi biến môi trường → phải **rebuild** (Manual Deploy → Clear build cache & deploy).

---

## 5. PostgreSQL

### 5.1. Tạo Database

1. **New +** → **PostgreSQL**
2. Điền:

| Trường | Gợi ý |
|--------|-------|
| **Name** | `myapp-db` |
| **Region** | `Singapore` (cùng region với app để nhanh!) |
| **PostgreSQL Version** | Mới nhất (16) |
| **Instance Type** | `Free` (lúc đầu) |

3. **Create** → đợi 2-3 phút

### 5.2. Lấy Connection String

Service database → tab **Info** → **Internal Database URL** và **External Database URL**:

```
# Internal (app + DB cùng Render, cùng region) — NHANH, MIỄN PHÍ
postgresql://user:pass@dpg-xxx-a:5432/mydb

# External (kết nối từ máy local, DBeaver...)
postgresql://user:pass@dpg-xxx-a.singapore-postgres.render.com:5432/mydb
```

> 💡 App trên Render nối DB bằng **Internal URL** — nhanh hơn và không tốn bandwidth!

### 5.3. Kết Nối Từ App

**Node.js:**
```javascript
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false }
});
```

**Python:**
```python
import os, psycopg2
conn = psycopg2.connect(os.environ['DATABASE_URL'])
```

Thêm biến trong Web Service → **Environment** → `DATABASE_URL` = Internal URL.

### 5.4. Free Tier Giới Hạn

| Giới hạn | Chi tiết |
|----------|----------|
| Thời gian | **90 ngày**, sau đó tự xóa (có email nhắc trước) |
| Storage | 1GB |
| Kết nối | Giới hạn |

> ⚠️ **Free PostgreSQL chỉ tồn tại 90 ngày!** Đủ cho học tập/prototype. App thật nên nâng lên gói trả phí ($6/tháng) hoặc dùng Neon/Supabase free.

### 5.5. Backup

```bash
# Backup
pg_dump "postgresql://user:pass@host:5432/db" > backup-$(date +%Y%m%d).sql

# Restore
psql "postgresql://user:pass@host:5432/db" < backup-20261010.sql
```

Render gói trả phí có **automatic backups** hàng ngày + point-in-time recovery.

---

## 6. Redis

### 6.1. Tạo Redis

**New +** → **Redis** → đặt tên → **Create**.

Lấy **Internal Redis URL** trong tab Info:
```
redis://red-xxx-a:6379
```

### 6.2. Dùng Trong App

**Node.js (cache):**
```javascript
const redis = require('redis');
const client = redis.createClient({ url: process.env.REDIS_URL });
await client.connect();

// Cache kết quả query nặng 5 phút
const cacheKey = 'products:all';
let products = await client.get(cacheKey);
if (!products) {
  products = JSON.stringify(await db.query('SELECT * FROM products'));
  await client.setEx(cacheKey, 300, products);
}
res.json(JSON.parse(products));
```

**Rate limiting:**
```javascript
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const limiter = rateLimit({
  store: new RedisStore({ client }),
  windowMs: 60 * 1000,  // 1 phút
  max: 100,             // tối đa 100 requests
});
app.use('/api/', limiter);
```

---

## 7. Background Workers

### 7.1. Khi Nào Dùng?

- Xử lý ảnh/video sau khi user upload
- Gửi email hàng loạt
- Đồng bộ dữ liệu định kỳ
- Bất kỳ task nào **không cần trả lời ngay** cho user

### 7.2. Tạo Worker

1. **New +** → **Background Worker** → Connect repo
2. **Build Command:** `npm install`
3. **Start Command:** `npm run worker` (không phải `npm start`!)

`package.json`:
```json
{
  "scripts": {
    "start": "node server.js",
    "worker": "node worker.js"
  }
}
```

`worker.js`:
```javascript
const { Worker } = require('bullmq');
const IORedis = require('ioredis');

const connection = new IORedis(process.env.REDIS_URL);

const worker = new Worker('jobs', async (job) => {
  console.log(`Xử lý job ${job.id}:`, job.data);
  // ... xử lý ...
  return { done: true };
}, { connection });

worker.on('completed', (job) => console.log(`Job ${job.id} xong!`));
worker.on('failed', (job, err) => console.error(`Job ${job.id} lỗi:`, err));

console.log('Worker đang chạy...');
```

> ⚠️ Worker **không** có public URL — chỉ giao tiếp qua Redis/DB với web service.

### 7.3. Kiến Trúc Điển Hình

```
User upload ảnh
      ↓
Web Service → đẩy job vào Redis queue → trả lời ngay "Đang xử lý!"
      ↓
Background Worker ← lấy job từ queue
      ↓
  Resize ảnh, upload S3, cập nhật DB
```

---

## 8. Cron Jobs

### 8.1. Tạo Cron Job

1. **New +** → **Cron Job** → Connect repo
2. Điền:

| Trường | Ví dụ |
|--------|-------|
| **Command** | `node scripts/daily-report.js` |
| **Schedule** | `0 7 * * *` (7h sáng mỗi ngày, giờ UTC) |

### 8.2. Cú Pháp Cron

```
┌───────────── phút (0-59)
│ ┌─────────── giờ (0-23, UTC!)
│ │ ┌───────── ngày trong tháng (1-31)
│ │ │ ┌─────── tháng (1-12)
│ │ │ │ ┌───── thứ trong tuần (0-6, 0=Chủ nhật)
│ │ │ │ │
* * * * *
```

| Schedule | Ý nghĩa |
|----------|---------|
| `0 7 * * *` | 7h sáng mỗi ngày (UTC) = 14h VN |
| `*/15 * * * *` | Mỗi 15 phút |
| `0 0 * * 0` | Nửa đêm Chủ nhật hàng tuần |
| `0 0 1 * *` | Ngày 1 mỗi tháng |

> ⚠️ Render dùng **giờ UTC**! VN = UTC+7, nên 7h sáng VN = `0 0 * * *`.

### 8.3. Ví Dụ: Script Backup DB Hàng Ngày

`scripts/backup.js`:
```javascript
const { exec } = require('child_process');
const fs = require('fs');

const date = new Date().toISOString().split('T')[0];
const cmd = `pg_dump "${process.env.DATABASE_URL}" | gzip > /tmp/backup-${date}.sql.gz`;

exec(cmd, (err) => {
  if (err) {
    console.error('Backup lỗi:', err);
    process.exit(1);
  }
  console.log(`Backup xong: backup-${date}.sql.gz`);
  // TODO: upload lên S3/R2 để lưu lâu dài
});
```

`package.json`:
```json
{
  "scripts": {
    "backup": "node scripts/backup.js"
  },
  "dependencies": {
    "pg": "^8.11.0"
  }
}
```

---

## 9. Environment Variables

### 9.1. Thêm Biến

Service → **Environment** → **Add Environment Variable**:

```
DATABASE_URL=postgresql://...
JWT_SECRET=bi-mat-tuyet-doi
NODE_ENV=production
```

**Thêm hàng loạt:** Bấm **Add from .env** → paste nội dung file `.env`:
```
DATABASE_URL=postgresql://user:pass@host/db
REDIS_URL=redis://...
STRIPE_KEY=sk_test_...
```

### 9.2. Sync Giữa Các Service

Khi tạo PostgreSQL, Render cho nút **Connect** → chọn web service → tự động thêm `DATABASE_URL` vào service đó. Tiện hơn copy tay!

### 9.3. Secret Files (File Bí Mật)

Một số app cần file (không phải biến), ví dụ `service-account.json` của Firebase:

**Environment** → **Add Secret File**:
- **Filename:** `service-account.json`
- **Contents:** paste nội dung JSON

File sẽ nằm ở `/etc/secrets/service-account.json` trong container.

```javascript
const admin = require('firebase-admin');
admin.initializeApp({
  credential: admin.credential.cert('/etc/secrets/service-account.json')
});
```

---

## 10. Custom Domains & HTTPS

### 10.1. Thêm Domain

Service → **Settings** → **Custom Domains** → **Add Custom Domain** → nhập `api.congty.vn`.

Render cho bản ghi DNS cần thêm:

```
# Với root domain (congty.vn)
Type:  A
Name:  @
Value: 216.24.57.1

# Với subdomain (api.congty.vn)
Type:  CNAME
Name:  api
Value: my-api.onrender.com
```

### 10.2. HTTPS Tự Động

Sau khi DNS propagate (5 phút - 24h), Render tự cấp SSL miễn phí. Không cần làm gì!

Kiểm tra: `https://api.congty.vn` → ổ khóa xanh là xong.

### 10.3. Cloudflare + Render

Nếu dùng Cloudflare:
1. Thêm CNAME như trên
2. SSL mode trong Cloudflare để **Full** (không phải Flexible)
3. Tắt proxy (đám mây xám) lúc verify đầu tiên, bật lại sau

---

## 11. Blueprints — render.yaml

### 11.1. Blueprint Là Gì?

**Blueprint** là file `render.yaml` định nghĩa **toàn bộ hạ tầng** bằng code. Push lên GitHub → Render tự tạo hết services, databases, cron jobs theo file.

```
Code (render.yaml) → Render tự tạo:
  ✅ Web service
  ✅ PostgreSQL
  ✅ Redis
  ✅ Cron job
  ✅ Biến môi trường liên kết
```

### 11.2. Ví Dụ render.yaml Đầy Đủ

```yaml
# render.yaml
services:
  # 1. Backend API
  - type: web
    name: my-api
    runtime: node
    plan: starter
    buildCommand: npm install
    startCommand: npm start
    healthCheckPath: /health
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        fromDatabase:
          name: myapp-db
          property: connectionString
      - key: REDIS_URL
        fromService:
          type: redis
          name: myapp-redis
          property: connectionString
      - key: JWT_SECRET
        generateValue: true  # tự sinh secret ngẫu nhiên!

  # 2. Background worker
  - type: worker
    name: myapp-worker
    runtime: node
    plan: starter
    buildCommand: npm install
    startCommand: npm run worker
    envVars:
      - key: REDIS_URL
        fromService:
          type: redis
          name: myapp-redis
          property: connectionString

  # 3. Cron job backup
  - type: cron
    name: db-backup
    runtime: node
    plan: starter
    schedule: "0 0 * * *"
    buildCommand: npm install
    startCommand: npm run backup
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: myapp-db
          property: connectionString

  # 4. Frontend static
  - type: web
    name: myapp-frontend
    runtime: static
    buildCommand: npm run build
    staticPublishPath: ./dist
    envVars:
      - key: VITE_API_URL
        value: https://my-api.onrender.com

databases:
  - name: myapp-db
    plan: starter
    postgresMajorVersion: "16"

  - name: myapp-redis
    plan: starter
    # Redis trên Render dùng type riêng:
    # (trong bản mới: type: redis tách riêng)
```

### 11.3. Dùng Blueprint

1. Push `render.yaml` lên repo (thư mục gốc)
2. Dashboard → **New +** → **Blueprint** → chọn repo
3. Render đọc file → hiện preview tất cả resources sẽ tạo
4. **Apply** → xong!

> 💡 Blueprint là **Infrastructure as Code** — team review được hạ tầng qua pull request, dễ nhân bản môi trường staging!

---

## 12. Logs & Metrics

### 12.1. Xem Logs

Service → tab **Logs**:
- **Deploy logs:** quá trình build
- **Runtime logs:** app đang chạy (realtime, tự cuộn)

**Lọc log:**
```
# Chỉ hiện lỗi
type:error

# Tìm theo text
"database connection"
```

### 12.2. Log Retention

| Gói | Giữ log bao lâu |
|-----|----------------|
| Free | 7 ngày |
| Starter+ | 30 ngày |

> 💡 Cần log lâu dài → ship sang Papertrail, Datadog, hoặc tự lưu vào DB.

### 12.3. Metrics

Service → tab **Metrics**:
- CPU, Memory theo thời gian
- HTTP requests/giây, response time
- Bandwidth

**Cảnh báo:** Nếu memory tăng đều không giảm → memory leak. Restart service là fix tạm, fix code mới là fix thật.

---

## 13. Dự Án Thực Tế: Deploy Full-Stack

### 13.1. Kiến Trúc

```
                    ┌─────────────────┐
                    │  Static Site    │
                    │  (Vite + React) │
                    │  myapp.onrender │
                    └────────┬────────┘
                             │ API calls
                             ▼
┌──────────┐        ┌─────────────────┐        ┌──────────┐
│  Cron    │        │  Web Service    │───────▶│ Postgres │
│ (backup) │        │  (Express API)  │        └──────────┘
└──────────┘        └────────┬────────┘        ┌──────────┐
                             └────────────────▶│  Redis   │
                                               └──────────┘
```

### 13.2. Backend API

`server.js`:
```javascript
const express = require('express');
const cors = require('cors');
const { Pool } = require('pg');
const redis = require('redis');

const app = express();
app.use(cors());
app.use(express.json());

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false }
});

const redisClient = redis.createClient({ url: process.env.REDIS_URL });
redisClient.connect().catch(console.error();

// Health check
app.get('/health', (req, res) => res.json({ ok: true }));

// Lấy tasks (có cache Redis 60s)
app.get('/api/tasks', async (req, res) => {
  const cached = await redisClient.get('tasks:all');
  if (cached) return res.json(JSON.parse(cached));

  const { rows } = await pool.query(
    'SELECT * FROM tasks ORDER BY created_at DESC LIMIT 50'
  );
  await redisClient.setEx('tasks:all', 60, JSON.stringify(rows));
  res.json(rows);
});

// Tạo task (xóa cache)
app.post('/api/tasks', async (req, res) => {
  const { rows } = await pool.query(
    'INSERT INTO tasks (title) VALUES ($1) RETURNING *',
    [req.body.title]
  );
  await redisClient.del('tasks:all');
  res.status(201).json(rows[0]);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, '0.0.0.0', () => console.log(`API chạy port ${PORT}`));
```

### 13.3. render.yaml Hoàn Chỉnh

```yaml
services:
  - type: web
    name: taskapp-api
    runtime: node
    plan: starter
    buildCommand: npm install
    startCommand: npm start
    healthCheckPath: /health
    envVars:
      - key: DATABASE_URL
        fromDatabase: { name: taskapp-db, property: connectionString }
      - key: REDIS_URL
        fromService: { type: redis, name: taskapp-redis, property: connectionString }

  - type: web
    name: taskapp-web
    runtime: static
    buildCommand: npm run build
    staticPublishPath: ./dist
    envVars:
      - key: VITE_API_URL
        value: https://taskapp-api.onrender.com

databases:
  - name: taskapp-db
    plan: starter
```

### 13.4. Các Bước Deploy

1. Push code + `render.yaml` lên GitHub
2. Render → **New +** → **Blueprint** → chọn repo → **Apply**
3. Đợi 5-10 phút cho tất cả services chạy
4. Chạy migration: vào web service → **Shell** tab → `npx prisma migrate deploy` (hoặc SQL trực tiếp)
5. Mở frontend URL → test tạo task!

---

## 14. Pricing Chi Tiết & Free Tier

### 14.1. Free Tier

| Service | Free giới hạn |
|---------|--------------|
| **Web Service** | 750 giờ/tháng, **sleep sau 15 phút không có traffic**, 512MB RAM |
| **Static Site** | 100GB bandwidth/tháng, không sleep |
| **PostgreSQL** | 90 ngày, 1GB storage, sau đó tự xóa |
| **Redis** | 90 ngày |
| **Cron Job** | 750 giờ/tháng |

> ⚠️ **Web service free bị SLEEP!** Request đầu tiên sau khi sleep mất 30-60s để "dậy". Không phù hợp API cần phản hồi nhanh 24/7.

**Giữ không sleep (mẹo):** Dùng UptimeRobot ping `/health` mỗi 10 phút. (Hơi "lách", nhưng nhiều người làm vậy cho project cá nhân.)

### 14.2. Gói Trả Phí

| Gói | Giá | RAM | CPU |
|-----|-----|-----|-----|
| **Starter** | $7/tháng/service | 512MB | 0.5 CPU |
| **Standard** | $25/tháng | 2GB | 1 CPU |
| **Pro** | $85/tháng | 4GB | 2 CPU |
| **Pro Plus** | $175/tháng | 8GB | 4 CPU |

**Database:**
| Gói | Giá | Storage |
|-----|-----|---------|
| Basic | $6/tháng | 1GB |
| Standard | $20-50/tháng | 16-32GB |

### 14.3. Ước Tính Chi Phí Thực Tế

**Side project nhỏ:**
```
1 Web Service (Starter):     $7
1 PostgreSQL (Basic):        $6
1 Static Site:               $0 (free)
1 Redis (Basic):             $6 (nếu cần)
─────────────────────────
Tổng:                        ~$13-19/tháng
```

**So với VPS Hetzner** ($5-6/tháng cho 4GB RAM): Render đắt hơn nhưng **không cần quản trị server**.

---

## 15. So Sánh Với Đối Thủ

| Tiêu chí | Render | Railway | Vercel | Fly.io |
|----------|--------|---------|--------|--------|
| Free web service | ✅ 750h (sleep) | $5 credit | ✅ | ✅ 3 VMs nhỏ |
| Free PostgreSQL | ✅ 90 ngày | $5 credit | ❌ | ✅ 3GB |
| Region gần VN | ✅ Singapore | ✅ Singapore | ✅ Singapore | ❌ |
| Docker | ✅ (trả phí) | ✅ | ❌ | ✅ |
| Cron job | ✅ | ✅ | ❌ | ✅ |
| Blueprint/IaC | ✅ render.yaml | ⚠️ | ❌ | ✅ fly.toml |
| Độ dễ dùng | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

**Chọn Render khi:** Cần free tier ổn định, thích cấu hình rõ ràng, cần cron job free.
**Chọn Railway khi:** Muốn linh hoạt tối đa, nhiều loại service, UI đẹp.
**Chọn Vercel khi:** Frontend Next.js, cần edge network nhanh nhất.

---

## 16. Troubleshooting

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `Port scan timeout` | App không listen đúng PORT | Dùng `process.env.PORT`, host `0.0.0.0` |
| Build fail `npm ERR!` | Sai Node version | Thêm `engines` vào package.json |
| `Connection refused` DB | Sai connection string | Dùng Internal URL, kiểm tra DB đang chạy |
| Service cứ restart liên tục | Health check fail / crash khi start | Xem logs, test start command ở local |
| Free service chậm request đầu | Bị sleep | Nâng gói hoặc dùng UptimeRobot ping |
| `Disk quota exceeded` | Free chỉ có 512MB disk | Dọn node_modules, dùng .dockerignore |
| Static site 404 khi refresh | SPA routing | Thêm `_redirects` hoặc rewrite rule |
| Biến môi trường không có hiệu lực | Quên redeploy | Sau khi đổi biến → Manual Deploy |
| Cron không chạy đúng giờ | Nhầm múi giờ | Render dùng UTC, VN = UTC+7 |
| Custom domain không verify | DNS sai | Kiểm tra bản ghi A/CNAME, đợi propagate |

---

## 17. Best Practices

### 17.1. Code
- ✅ Luôn dùng `process.env.PORT`, listen `0.0.0.0`
- ✅ Có endpoint `/health`
- ✅ Graceful shutdown:
```javascript
process.on('SIGTERM', () => {
  console.log('SIGTERM received, shutting down...');
  server.close(() => process.exit(0));
});
```
- ❌ Không lưu file vào local disk (mất khi redeploy) → dùng S3/R2

### 17.2. Database
- ✅ App và DB **cùng region** (Singapore)
- ✅ Dùng Internal URL trong app
- ✅ Connection pooling (PgBouncer hoặc Prisma)
- ✅ Backup trước khi migrate lớn

### 17.3. Chi Phí
- ✅ Free tier cho dev/staging, trả phí cho production
- ✅ Tắt service không dùng
- ✅ Dùng Blueprint để dễ quản lý và nhân bản

### 17.4. Bảo Mật
- ✅ Secret trong Environment Variables, không commit
- ✅ `.env` trong `.gitignore`
- ✅ Rate limiting cho API public
- ✅ CORS chỉ cho domain frontend của bạn:
```javascript
app.use(cors({ origin: 'https://myapp.onrender.com' }));
```

---

## 18. FAQ

**Hỏi: Render free có thật sự free không?**
Đáp: Có. Web service 750h/tháng + static site + PostgreSQL 90 ngày đều free không cần thẻ.

**Hỏi: Tại sao API của tôi lúc nhanh lúc chậm?**
Đáp: Free tier bị sleep sau 15 phút idle. Request đầu tiên phải đợi "đánh thức" 30-60s.

**Hỏi: Render có region Việt Nam không?**
Đáp: Không, gần nhất là Singapore (~30-50ms từ VN, chấp nhận được).

**Hỏi: Có hỗ trợ WebSocket không?**
Đáp: Có, trên web service (không phải static site).

**Hỏi: Deploy monorepo thế nào?**
Đáp: Tạo nhiều services từ cùng repo, mỗi service đặt **Root Directory** khác nhau.

**Hỏi: Render vs Railway nên chọn cái nào?**
Đáp: Render đơn giản, free tier rõ ràng, cron free. Railway linh hoạt hơn, UI đẹp hơn. Thử cả 2 với free tier rồi quyết định!

**Hỏi: Có thể chạy Docker Compose không?**
Đáp: Không trực tiếp. Mỗi service là 1 container riêng, liên kết qua Blueprint (render.yaml).

---

## Phụ Lục: Cheat Sheet

```yaml
# render.yaml mẫu tối thiểu
services:
  - type: web
    name: my-api
    runtime: node
    buildCommand: npm install
    startCommand: npm start
    healthCheckPath: /health
    envVars:
      - key: DATABASE_URL
        fromDatabase:
          name: mydb
          property: connectionString

databases:
  - name: mydb
    plan: free
```

| Thao tác | Vị trí |
|----------|--------|
| Tạo web service | Dashboard → New + → Web Service |
| Tạo database | New + → PostgreSQL / Redis |
| Deploy bằng code | New + → Blueprint (render.yaml) |
| Xem logs | Service → Logs |
| Thêm biến môi trường | Service → Environment |
| Custom domain | Service → Settings → Custom Domains |
| Manual deploy | Service → Manual Deploy |
| Shell vào container | Service → Shell |

**Links:**
- Docs: https://render.com/docs
- Status: https://status.render.com
- Cộng đồng: https://community.render.com

---
