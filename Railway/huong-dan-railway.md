# Hướng Dẫn Toàn Tập Railway — Deploy App Không Cần Biết DevOps

> **Cấp độ:** Người mới bắt đầu → deploy được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10

---

## Mục Lục

1. [Railway Là Gì?](#1-railway-là-gì)
2. [Tạo Tài Khoản & Setup](#2-tạo-tài-khoản--setup)
3. [Deploy App Đầu Tiên Từ GitHub](#3-deploy-app-đầu-tiên-từ-github)
4. [Environment Variables](#4-environment-variables)
5. [Databases: PostgreSQL, MySQL, Redis, MongoDB](#5-databases-postgresql-mysql-redis-mongodb)
6. [Custom Domain](#6-custom-domain)
7. [Railway CLI](#7-railway-cli)
8. [Logs & Monitoring](#8-logs--monitoring)
9. [Scaling & Performance](#9-scaling--performance)
10. [Pricing Chi Tiết](#10-pricing-chi-tiết)
11. [Dự Án Thực Tế: Deploy Full-Stack App](#11-dự-án-thực-tế-deploy-full-stack-app)
12. [Troubleshooting](#12-troubleshooting)
13. [Best Practices](#13-best-practices)
14. [FAQ](#14-faq)

---

## 1. Railway Là Gì?

### 1.1. Định Nghĩa

**Railway** là nền tảng **PaaS (Platform as a Service)** — bạn push code lên GitHub, Railway tự động build và chạy app cho bạn. Không cần cấu hình server, không cần biết Docker (nhưng biết thì tốt hơn).

Slogan của Railway: *"Deploy code, not infrastructure"* — bạn lo code, Railway lo hạ tầng.

### 1.2. So Sánh Với Các Nền Tảng Khác

| Tiêu chí | Railway | Render | Vercel | Heroku | VPS tự quản |
|----------|---------|--------|--------|--------|-------------|
| Deploy từ GitHub | ✅ 1 click | ✅ | ✅ | ✅ | ❌ Tự làm |
| Database 1-click | ✅ Postgres/Redis/MySQL/Mongo | ✅ Postgres/Redis | ❌ (dùng Neon/Supabase) | ✅ | ❌ Tự cài |
| Hỗ trợ mọi ngôn ngữ | ✅ | ✅ | Chủ yếu JS | ✅ | ✅ |
| Free tier | $5 credit/tháng | 750h free | Rộng rãi | Không còn | Trả tiền VPS |
| Giá khi scale | Trung bình | Trung bình | Đắt khi nhiều bandwidth | Đắt | Rẻ nhất |
| Độ dễ dùng | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |

### 1.3. Khi Nào Nên Dùng Railway?

**Nên dùng:**
- ✅ Side project, MVP — cần deploy nhanh
- ✅ Backend API (Node.js, Python, Go, Rust...)
- ✅ Cần database mà không muốn tự quản
- ✅ Team nhỏ, không có DevOps

**Không nên:**
- ❌ App cần GPU (Railway chưa hỗ trợ)
- ❌ Traffic cực lớn (chi phí sẽ cao hơn VPS)
- ❌ Cần kiểm soát hạ tầng chi tiết

---

## 2. Tạo Tài Khoản & Setup

### 2.1. Đăng Ký

1. Vào https://railway.app
2. Bấm **Start a New Project** → **Login with GitHub**
3. Authorize Railway truy cập GitHub
4. Xong! Bạn được tặng **$5 credit miễn phí mỗi tháng**

### 2.2. Tổng Quan Dashboard

Sau khi đăng nhập, bạn thấy:

```
┌─────────────────────────────────────────┐
│  Railway Dashboard                       │
├─────────────────────────────────────────┤
│  [+ New Project]                         │
│                                          │
│  Projects của bạn:                       │
│  📦 my-api          ● Running            │
│  📦 blog-frontend   ● Running            │
│  📦 postgres-db     ● Running            │
└─────────────────────────────────────────┘
```

**Khái niệm cơ bản:**

| Khái niệm | Giải thích |
|-----------|-----------|
| **Project** | 1 dự án = 1 tập hợp các service liên quan |
| **Service** | 1 thành phần chạy: web app, database, worker... |
| **Environment** | Môi trường: production, staging, development |
| **Deployment** | 1 lần deploy (mỗi lần push code = 1 deployment mới) |

### 2.3. Tạo Project Mới

Bấm **+ New Project**, chọn 1 trong:

| Option | Khi nào dùng |
|--------|--------------|
| **Deploy from GitHub repo** | Có sẵn code trên GitHub (phổ biến nhất) |
| **Empty project** | Tạo project trống rồi thêm service sau |
| **Database** | Chỉ cần database (Postgres/Redis...) |
| **Template** | Deploy app mẫu có sẵn (1-click) |

---

## 3. Deploy App Đầu Tiên Từ GitHub

### 3.1. Chuẩn Bị Code

Railway tự phát hiện framework qua các file đặc trưng:

| File | Framework Railway đoán |
|------|------------------------|
| `package.json` | Node.js |
| `requirements.txt` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `Gemfile` | Ruby |
| `Dockerfile` | Docker (ưu tiên cao nhất) |

**Ví dụ app Node.js đơn giản:**

`package.json`:
```json
{
  "name": "my-api",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "engines": {
    "node": "20.x"
  }
}
```

`server.js`:
```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.json({ message: 'Hello từ Railway! 🚂' });
});

app.get('/health', (req, res) => {
  res.json({ status: 'ok', time: new Date().toISOString() });
});

app.listen(PORT, () => {
  console.log(`Server chạy ở port ${PORT}`);
});
```

> ⚠️ **QUAN TRỌNG:** App PHẢI listen ở `process.env.PORT` (Railway cấp port động), không hardcode port cố định!

**Ví dụ app Python (Flask):**

`requirements.txt`:
```
flask==3.0.0
gunicorn==21.2.0
```

`app.py`:
```python
import os
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/')
def home():
    return jsonify(message='Hello từ Railway! 🚂')

@app.route('/health')
def health():
    return jsonify(status='ok')

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 5000))
    app.run(host='0.0.0.0', port=port)
```

`Procfile` (cho Python, để Railway biết lệnh chạy):
```
web: gunicorn app:app
```

### 3.2. Các Bước Deploy

**Bước 1:** Push code lên GitHub repo.

**Bước 2:** Trên Railway → **New Project** → **Deploy from GitHub repo** → chọn repo.

**Bước 3:** Railway tự động:
1. 🔍 Phát hiện framework
2. 📦 Cài dependencies (`npm install`, `pip install`...)
3. 🔨 Build (nếu cần)
4. ▶️ Chạy app với lệnh start

**Bước 4:** Bấm vào service → tab **Settings** → **Networking** → **Generate Domain** để có URL public:
```
https://my-api-production.up.railway.app
```

**Bước 5:** Mở URL → thấy `{"message": "Hello từ Railway! 🚂"}` là thành công! 🎉

### 3.3. Auto-Deploy

Mặc định, mỗi lần bạn **push code lên nhánh chính** (main), Railway tự động deploy lại. Xem tiến trình ở tab **Deployments**.

**Tắt auto-deploy** (nếu muốn deploy tay): Service → Settings → **Source** → tắt **Auto Deploy**.

### 3.4. Cấu Hình Build & Start Tùy Chỉnh

Nếu Railway đoán sai, chỉnh tay trong **Settings**:

| Cài đặt | Ví dụ |
|---------|-------|
| **Build Command** | `npm run build` |
| **Start Command** | `npm start` |
| **Root Directory** | `frontend/` (nếu code trong subfolder) |
| **Watch Paths** | `src/**` (chỉ deploy khi file trong src đổi) |

**Dùng Dockerfile** (kiểm soát hoàn toàn):

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

Railway ưu tiên Dockerfile nếu thấy file này trong repo.

---

## 4. Environment Variables

### 4.1. Thêm Biến Môi Trường

Service → tab **Variables** → **+ New Variable**:

```
DATABASE_URL=postgresql://user:pass@host:5432/db
JWT_SECRET=siêu-bí-mật-không-cho-ai-biết
NODE_ENV=production
```

**Hoặc import hàng loạt** (dạng `.env`):
```
DATABASE_URL=postgresql://...
JWT_SECRET=abc123
API_KEY=xyz789
```
Paste vào → Railway tự parse từng dòng.

### 4.2. Biến Có Sẵn Của Railway

Railway tự inject một số biến:

| Biến | Ý nghĩa |
|------|---------|
| `PORT` | Port app phải listen (bắt buộc dùng!) |
| `RAILWAY_ENVIRONMENT` | Tên environment (production/staging) |
| `RAILWAY_PROJECT_ID` | ID của project |
| `RAILWAY_SERVICE_ID` | ID của service |

### 4.3. Tham Chiếu Biến Giữa Các Service

Khi bạn thêm PostgreSQL vào project, Railway tạo biến `DATABASE_URL` ở service database. Để app dùng được:

Trong service app → Variables → thêm:
```
DATABASE_URL=${{Postgres.DATABASE_URL}}
```

Cú pháp `${{TenService.TEN_BIEN}}` — Railway tự thay bằng giá trị thật khi deploy!

### 4.4. Biến Cho Từng Environment

- Tab **Variables** → chọn environment (production/staging)
- Mỗi environment có bộ biến riêng
- Ví dụ: `API_URL` ở staging trỏ test API, ở production trỏ API thật

---

## 5. Databases: PostgreSQL, MySQL, Redis, MongoDB

### 5.1. Thêm Database 1-Click

Trong project → **+ New** → **Database** → chọn:

| Database | Khi nào dùng |
|----------|--------------|
| **PostgreSQL** | Database chính cho hầu hết app (khuyên dùng) |
| **MySQL** | App cũ dùng MySQL, hoặc thích MySQL hơn |
| **Redis** | Cache, session, queue, rate limiting |
| **MongoDB** | Dữ liệu phi cấu trúc, prototype nhanh |

Railway tự tạo service database + sinh connection string.

### 5.2. Kết Nối Từ App

**PostgreSQL + Node.js:**

```javascript
// Dùng biến DATABASE_URL từ Railway
const { Pool } = require('pg');
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false } // Railway yêu cầu SSL
});

app.get('/users', async (req, res) => {
  const result = await pool.query('SELECT * FROM users LIMIT 10');
  res.json(result.rows);
});
```

**PostgreSQL + Python:**

```python
import os
import psycopg2

conn = psycopg2.connect(os.environ['DATABASE_URL'])
cur = conn.cursor()
cur.execute("SELECT version();")
print(cur.fetchone())
```

**Redis + Node.js:**

```javascript
const redis = require('redis');
const client = redis.createClient({ url: process.env.REDIS_URL });
await client.connect();

// Cache ví dụ
await client.set('user:1', JSON.stringify({ name: 'An' }), { EX: 3600 });
const cached = await client.get('user:1');
```

### 5.3. Truy Cập Database Từ Máy Local

Service database → tab **Connect** → Railway cho connection string public:

```bash
# PostgreSQL
psql "postgresql://postgres:password@containers-us-west-xxx.railway.app:7890/railway"

# Hoặc dùng GUI: DBeaver, TablePlus, pgAdmin
# Host, port, user, password lấy từ tab Connect
```

> ⚠️ Chỉ bật public access khi cần. Xong việc nên tắt để bảo mật.

### 5.4. Backup Database

```bash
# Backup PostgreSQL
pg_dump "postgresql://..." > backup-$(date +%Y%m%d).sql

# Restore
psql "postgresql://..." < backup-20261010.sql
```

**Tự động backup** bằng cron service (xem mục Cron ở phần Render, hoặc dùng GitHub Actions).

### 5.5. So Sánh Database Trên Railway

| Database | Giá (ước tính) | Phù hợp |
|----------|---------------|---------|
| PostgreSQL | ~$5-10/tháng (dùng ít) | App production nhỏ-vừa |
| Redis | ~$3-5/tháng | Cache/session |
| MySQL | ~$5-10/tháng | Tương tự Postgres |
| MongoDB | ~$5-10/tháng | Prototype, dữ liệu linh hoạt |

> 💡 **Mẹo tiết kiệm:** Database chạy 24/7 nên tốn tiền liên tục. App test/staging có thể xóa database khi không dùng.

---

## 6. Custom Domain

### 6.1. Thêm Domain Riêng

Service → **Settings** → **Networking** → **Custom Domain** → nhập `api.congty.vn`.

Railway sẽ cho bạn 1 bản ghi **CNAME**:

```
Type:  CNAME
Name:  api
Value: abc123.up.railway.app
```

### 6.2. Cấu Hình DNS

Vào nơi quản lý domain (Cloudflare, Namecheap, GoDaddy...) thêm bản ghi CNAME trên.

**Với Cloudflare:**
1. DNS → Add record → Type: CNAME, Name: `api`, Target: `abc123.up.railway.app`
2. Proxy status: để **DNS only** (tắt đám mây vàng) lần đầu để Railway verify nhanh
3. Sau khi verify xong có thể bật proxy lại

### 6.3. HTTPS Tự Động

Railway tự cấp chứng chỉ SSL miễn phí (Let's Encrypt) sau khi domain verify xong. Không cần làm gì thêm!

Kiểm tra: mở `https://api.congty.vn` → phải có ổ khóa xanh.

---

## 7. Railway CLI

### 7.1. Cài Đặt

```bash
# macOS / Linux
curl -fsSL https://railway.app/install.sh | sh

# Hoặc bằng npm
npm i -g @railway/cli

# Kiểm tra
railway --version
```

### 7.2. Đăng Nhập & Liên Kết Project

```bash
# Đăng nhập (mở trình duyệt)
railway login

# Liên kết thư mục hiện tại với project
railway link
# → Chọn project → environment → service
```

### 7.3. Các Lệnh Thường Dùng

```bash
# Xem trạng thái
railway status

# Xem biến môi trường
railway variables

# Thêm biến
railway variables set API_KEY=abc123

# Xem log realtime
railway logs

# Chạy lệnh trong môi trường Railway (dùng biến của service)
railway run python manage.py migrate
railway run npm run seed

# Mở shell vào service
railway shell

# Deploy từ local (không cần push GitHub)
railway up

# Mở dashboard trên trình duyệt
railway open
```

### 7.4. Workflow Với CLI

```bash
# 1. Link project
railway link

# 2. Chạy migration dùng DB của Railway
railway run npx prisma migrate deploy

# 3. Test biến môi trường
railway run node -e "console.log(process.env.DATABASE_URL)"

# 4. Deploy nhanh từ local
railway up
```

---

## 8. Logs & Monitoring

### 8.1. Xem Logs

**Trên dashboard:** Service → tab **Deployments** → chọn deployment → xem log build + runtime.

**Bằng CLI:**
```bash
# Log realtime
railway logs

# Log của deployment cụ thể
railway logs --deployment <id>
```

### 8.2. Các Loại Log

| Tab | Nội dung |
|-----|----------|
| **Build Logs** | Quá trình cài dependencies, build |
| **Deploy Logs** | Quá trình start app |
| **HTTP Logs** | Request đến app (nếu bật) |

### 8.3. Health Check

Railway tự kiểm tra app có "sống" không. Cấu hình trong **Settings**:

| Cài đặt | Gợi ý |
|---------|-------|
| **Healthcheck Path** | `/health` (endpoint trả về 200) |
| **Healthcheck Timeout** | 30 giây |

**Code endpoint health:**

```javascript
// Node.js
app.get('/health', (req, res) => res.status(200).json({ ok: true }));
```

```python
# Python Flask
@app.route('/health')
def health():
    return {'ok': True}, 200
```

Nếu health check fail liên tục → Railway restart service.

### 8.4. Metrics

Service → tab **Metrics** xem:
- CPU usage
- Memory usage
- Network in/out
- Disk usage

> 💡 Nếu memory tăng liên tục không giảm → có thể bị **memory leak**. Kiểm tra code!

---

## 9. Scaling & Performance

### 9.1. Vertical Scaling (Tăng Tài Nguyên)

Service → **Settings** → tăng **CPU / Memory** cho service.

| Gói | CPU | RAM | Giá ước tính |
|-----|-----|-----|-------------|
| Hobby | 0.5 vCPU | 512MB | Trong $5 credit |
| Pro | Tùy chỉnh | Tùy chỉnh | Tính theo usage |

### 9.2. Replicas (Chạy Nhiều Bản Sao)

**Settings** → **Replicas** → tăng số lượng (ví dụ 3).

```
Request → Railway Load Balancer → Replica 1
                                   → Replica 2
                                   → Replica 3
```

> ⚠️ App phải **stateless** (không lưu session/file local) thì mới scale replicas được!

### 9.3. Tối Ưu Chi Phí

- Service staging/dev: **tắt khi không dùng** (bấm ⏸️)
- Database test: xóa khi xong việc
- Dùng Redis cache để giảm query DB (giảm tiền DB)
- Đặt **usage limits** trong project settings để không bị "đốt" tiền

---

## 10. Pricing Chi Tiết

### 10.1. Các Gói

| Gói | Giá | Bao gồm |
|-----|-----|---------|
| **Trial** | $5 credit/tháng (miễn phí) | Dùng thử, không cần thẻ |
| **Hobby** | $5/tháng | $5 usage + $5 credit thêm = $10 tổng |
| **Pro** | $20/tháng | $20 usage included, sau đó tính thêm theo dùng |
| **Team/Enterprise** | Liên hệ | Nhiều member, SSO, support riêng |

### 10.2. Tính Tiền Theo Gì?

Railway tính theo **tài nguyên thực dùng**:

| Tài nguyên | Đơn giá (ước tính) |
|------------|-------------------|
| CPU | ~$0.000231/vCPU/phút |
| RAM | ~$0.000231/GB/phút |
| Network egress | Miễn phí (khác với nhiều PaaS!) |

**Ví dụ:** 1 app Node.js nhỏ (0.5 CPU, 512MB RAM) chạy 24/7 ≈ $5/tháng.

### 10.3. Mẹo Dùng Free Tier Hiệu Quả

1. **$5 credit** đủ cho: 1 API nhỏ + 1 Postgres nhỏ
2. Tắt service khi không dùng (đi ngủ tắt, sáng bật)
3. Dùng SQLite thay Postgres cho app siêu nhỏ (lưu ý: mất data khi redeploy!)
4. Theo dõi usage trong **Project → Usage** hàng ngày

---

## 11. Dự Án Thực Tế: Deploy Full-Stack App

### 11.1. Kiến Trúc

```
┌─────────────┐     ┌─────────────┐     ┌──────────────┐
│  Frontend   │────▶│  Backend    │────▶│  PostgreSQL  │
│  (Vite)     │     │  (Express)  │     │              │
└─────────────┘     └─────────────┘     └──────────────┘
       │                   │
       ▼                   ▼
  Static hosting      Web service
  (Railway/Vercel)    (Railway)
```

### 11.2. Backend (Express + Prisma)

`schema.prisma`:
```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Task {
  id        Int      @id @default(autoincrement())
  title     String
  done      Boolean  @default(false)
  createdAt DateTime @default(now())
}
```

`server.js`:
```javascript
const express = require('express');
const { PrismaClient } = require('@prisma/client');
const cors = require('cors');

const prisma = new PrismaClient();
const app = express();
app.use(cors());
app.use(express.json());

app.get('/api/tasks', async (req, res) => {
  const tasks = await prisma.task.findMany({ orderBy: { createdAt: 'desc' } });
  res.json(tasks);
});

app.post('/api/tasks', async (req, res) => {
  const task = await prisma.task.create({ data: { title: req.body.title } });
  res.status(201).json(task);
});

app.patch('/api/tasks/:id', async (req, res) => {
  const task = await prisma.task.update({
    where: { id: Number(req.params.id) },
    data: { done: req.body.done },
  });
  res.json(task);
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`API chạy ở port ${PORT}`));
```

`package.json`:
```json
{
  "scripts": {
    "start": "node server.js",
    "postinstall": "prisma generate",
    "migrate": "prisma migrate deploy"
  },
  "dependencies": {
    "@prisma/client": "^5.0.0",
    "cors": "^2.8.5",
    "express": "^4.18.2"
  },
  "devDependencies": {
    "prisma": "^5.0.0"
  }
}
```

### 11.3. Các Bước Deploy

**Bước 1 — Tạo project, thêm PostgreSQL:**
- New Project → Deploy from GitHub (repo backend)
- + New → Database → PostgreSQL

**Bước 2 — Link DATABASE_URL:**
- Service backend → Variables → thêm:
```
DATABASE_URL=${{Postgres.DATABASE_URL}}
```

**Bước 3 — Chạy migration:**
```bash
railway link  # chọn service backend
railway run npx prisma migrate deploy
```

**Bước 4 — Tạo domain:**
- Settings → Networking → Generate Domain
- Test: `https://xxx.up.railway.app/api/tasks`

**Bước 5 — Deploy frontend:**
- Frontend dùng Vite, build ra static → có thể deploy lên Railway (static service) hoặc Vercel
- Set `VITE_API_URL=https://xxx.up.railway.app` trong biến môi trường của frontend

---

## 12. Troubleshooting

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `Application failed to respond` | App không listen đúng PORT | Dùng `process.env.PORT`, listen `0.0.0.0` |
| Build fail: `npm ERR!` | Sai version Node, thiếu dependency | Thêm `engines` vào package.json, kiểm tra `npm install` local |
| `ECONNREFUSED` database | Sai DATABASE_URL | Kiểm tra biến `${{Postgres.DATABASE_URL}}`, đảm bảo DB đang chạy |
| Deploy treo ở "Building" | Build quá lâu, hết memory | Tăng memory, tối ưu Dockerfile (multi-stage) |
| 502 Bad Gateway | App crash khi start | Xem deploy logs, kiểm tra lệnh start |
| Hết $5 credit | Dùng quá nhiều | Vào Usage xem service nào tốn, tắt bớt |
| Domain custom không verify | DNS chưa đúng | Kiểm tra CNAME, đợi DNS propagate (vài phút - 24h) |
| `railway link` không thấy project | Chưa login hoặc sai account | `railway logout` rồi `railway login` lại |

### Xem Log Chi Tiết Khi Lỗi

```bash
# Log build của deployment mới nhất
railway logs --deployment $(railway deployments | head -1)
```

---

## 13. Best Practices

### 13.1. Code

- ✅ Luôn dùng `process.env.PORT`, không hardcode
- ✅ Có endpoint `/health` trả về 200
- ✅ Log ra stdout (Railway tự thu thập)
- ✅ Xử lý `SIGTERM` để shutdown graceful:
```javascript
process.on('SIGTERM', async () => {
  console.log('Nhận SIGTERM, đang tắt...');
  await prisma.$disconnect();
  server.close(() => process.exit(0));
});
```
- ❌ Không lưu file upload vào local disk (mất khi redeploy) → dùng S3/Cloudinary

### 13.2. Database

- ✅ Dùng connection pooling (PgBouncer hoặc Prisma pool)
- ✅ Chạy migration qua `railway run`, không chạy tay
- ✅ Backup định kỳ bằng `pg_dump`
- ❌ Không expose database public nếu không cần

### 13.3. Bảo Mật

- ✅ Mọi secret để trong Variables, không commit `.env`
- ✅ Thêm `.env` vào `.gitignore`
- ✅ Dùng `DATABASE_URL` với SSL
- ✅ Rate limiting cho API public

### 13.4. Chi Phí

- ✅ Đặt usage limit trong project settings
- ✅ Tắt service dev/staging khi không dùng
- ✅ Theo dõi Usage tab hàng tuần
- ✅ Dùng 1 database chung cho nhiều service dev nếu được

---

## 14. FAQ

**Hỏi: Railway có free thật không?**
Đáp: Có $5 credit/tháng, không cần thẻ. Đủ chạy 1 API nhỏ + 1 DB nhỏ.

**Hỏi: Railway khác Vercel thế nào?**
Đáp: Vercel mạnh về frontend/serverless. Railway mạnh về backend long-running + database 1-click. Nhiều người dùng cả 2: frontend trên Vercel, backend+DB trên Railway.

**Hỏi: Có hỗ trợ WebSocket không?**
Đáp: Có, vì Railway chạy container long-running, WebSocket hoạt động bình thường.

**Hỏi: Deploy có mất dữ liệu database không?**
Đáp: Không. Database là service riêng, redeploy app không ảnh hưởng.

**Hỏi: Có thể chạy cron job không?**
Đáp: Có — tạo service riêng chạy cron, hoặc dùng tính năng Cron trong project settings (tùy gói).

**Hỏi: Railway có ở Việt Nam không? Server đặt ở đâu?**
Đáp: Railway là công ty Mỹ, server ở US/EU. Từ VN ping khoảng 200-300ms. Với app realtime nhạy latency, cân nhắc thêm CDN hoặc cache.

---

## Phụ Lục: Cheat Sheet

```bash
# Cài CLI
curl -fsSL https://railway.app/install.sh | sh

# Đăng nhập & link
railway login
railway link

# Biến môi trường
railway variables
railway variables set KEY=value

# Logs
railway logs

# Chạy lệnh với env của service
railway run npx prisma migrate deploy

# Deploy từ local
railway up

# Mở dashboard
railway open
```

| Thao tác | Vị trí trên Dashboard |
|----------|----------------------|
| Tạo domain public | Service → Settings → Networking → Generate Domain |
| Thêm database | Project → + New → Database |
| Xem usage/tiền | Project → Usage |
| Tắt auto-deploy | Service → Settings → Source |
| Custom domain | Service → Settings → Networking → Custom Domain |
| Tăng CPU/RAM | Service → Settings → Resources |
| Xem metrics | Service → Metrics |

---

## 15. Monorepo & Multi-Service

### 15.1. Deploy Monorepo

Nếu repo có nhiều app (frontend + backend):

```
my-project/
├── frontend/          # Vite app
│   └── package.json
├── backend/           # Express API
│   └── package.json
└── package.json       # root (có thể không có)
```

**Cách 1 — 2 services riêng:**
1. Tạo service 1: Root Directory = `backend`
2. Tạo service 2: Root Directory = `frontend`
3. Mỗi service deploy độc lập

**Cách 2 — Dùng Dockerfile riêng:**
```
backend/Dockerfile
frontend/Dockerfile
```

### 15.2. Service Giao Tiếp Nội Bộ

Các service trong cùng project gọi nhau qua **private network** (nhanh, không tốn egress):

```
# Backend gọi database - dùng private URL
DATABASE_URL=postgresql://postgres:pass@postgres.railway.internal:5432/railway
```

Railway tự tạo DNS nội bộ: `<service-name>.railway.internal`.

> 💡 Dùng private network khi service gọi nhau — nhanh hơn và không tính tiền network!

---

## 16. Cron Jobs & Background Workers

### 16.1. Background Worker

Worker chạy liên tục, không cần public URL:

```javascript
// worker.js - xử lý queue
const { Worker } = require('bullmq');
const redis = require('ioredis');

const connection = new redis(process.env.REDIS_URL);
const worker = new Worker('emails', async job => {
  console.log('Gửi email:', job.data);
  // ... logic gửi email
}, { connection });
```

`package.json`:
```json
{
  "scripts": {
    "worker": "node worker.js"
  }
}
```

Deploy: tạo service mới từ cùng repo, **Start Command** = `npm run worker`, **không** tạo public domain.

### 16.2. Cron Job

**Cách 1 — Service cron riêng:**

```javascript
// cron.js
const cron = require('node-cron');
const { Pool } = require('pg');
const pool = new Pool({ connectionString: process.env.DATABASE_URL });

// Chạy lúc 2h sáng mỗi ngày: dọn dữ liệu cũ
cron.schedule('0 2 * * *', async () => {
  console.log('Bắt đầu dọn dữ liệu cũ...');
  await pool.query("DELETE FROM logs WHERE created_at < NOW() - INTERVAL '30 days'");
  console.log('Xong!');
});

console.log('Cron worker đang chạy...');
```

**Cách 2 — Railway Cron (nếu gói hỗ trợ):**
Project → **+ New** → **Cron** → đặt lịch → chọn service/command.

---

## 17. So Sánh Chi Tiết Railway vs Đối Thủ

### 17.1. Railway vs Render

| Tiêu chí | Railway | Render |
|----------|---------|--------|
| Free tier | $5 credit/tháng | 750 giờ free (sleep khi idle) |
| Database 1-click | ✅ PG/MySQL/Redis/Mongo | ✅ PG/Redis |
| Deploy từ GitHub | ✅ | ✅ |
| Private network | ✅ | ✅ |
| Giá | Tính theo usage thực | Gói cố định + usage |
| Phù hợp | Linh hoạt, nhiều loại service | Đơn giản, web service truyền thống |

### 17.2. Railway vs VPS (DigitalOcean/Hetzner)

| Tiêu chí | Railway | VPS |
|----------|---------|-----|
| Setup | 5 phút | 1-2 giờ (cài đặt, bảo mật...) |
| Giá (app nhỏ) | ~$5/tháng | ~$4-6/tháng (Hetzner) |
| Giá (app lớn) | Đắt hơn VPS | Rẻ hơn khi scale |
| Database | 1-click | Tự cài + backup |
| SSL | Tự động | Tự cấu hình |
| Scaling | 1 click | Tự làm |

**Quy tắc:** App nhỏ/vừa, cần nhanh → Railway. App lớn, traffic cao, team có DevOps → VPS rẻ hơn.

---

## 18. Thuật Ngữ (Glossary)

| Thuật ngữ | Giải thích |
|-----------|-----------|
| PaaS | Platform as a Service — nền tảng lo hạ tầng giúp bạn |
| Service | 1 thành phần chạy (app, database, worker...) |
| Deployment | 1 lần deploy code mới |
| Environment | Môi trường (production/staging) |
| Private network | Mạng nội bộ giữa các service, nhanh + miễn phí |
| Health check | Kiểm tra định kỳ xem app còn sống không |
| Replica | Bản sao của service để chịu tải |
| Egress | Dữ liệu đi ra internet (một số PaaS tính tiền) |

---
