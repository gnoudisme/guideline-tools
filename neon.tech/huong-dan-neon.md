# Hướng Dẫn Toàn Tập Neon — Serverless Postgres

> **Cấp độ:** Người mới bắt đầu → dùng được thật trong dự án
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10
>
> ⚠️ Neon là Postgres "serverless": tách compute và storage, scale về 0 khi không dùng. Free tier hào phóng, không cần thẻ.

---

## Mục Lục

1. [Neon Là Gì?](#1-neon-là-gì)
2. [Neon vs Postgres Thường vs Supabase](#2-neon-vs-postgres-thường-vs-supabase)
3. [Đăng Ký & Tạo Project](#3-đăng-ký--tạo-project)
4. [Kết Nối Database](#4-kết-nối-database)
5. [SQL Editor Trên Dashboard](#5-sql-editor-trên-dashboard)
6. [Branching — Tính Năng "Git Cho Database"](#6-branching--tính-năng-git-cho-database)
7. [Kết Nối Từ Node.js](#7-kết-nối-từ-nodejs)
8. [Neon Serverless Driver (@neondatabase/serverless)](#8-neon-serverless-driver)
9. [Kết Nối Từ Python](#9-kết-nối-từ-python)
10. [Dùng Với Prisma ORM](#10-dùng-với-prisma-orm)
11. [Dùng Với Drizzle ORM](#11-dùng-với-drizzle-orm)
12. [Autoscaling & Scale to Zero](#12-autoscaling--scale-to-zero)
13. [Point-in-Time Restore](#13-point-in-time-restore)
14. [Quản Lý Với Neon CLI](#14-quản-lý-với-neon-cli)
15. [Bảo Mật — Roles & Connection String](#15-bảo-mật--roles--connection-string)
16. [Pricing & Free Tier](#16-pricing--free-tier)
17. [Troubleshooting](#17-troubleshooting)
18. [Best Practices & Cheat Sheet](#18-best-practices--cheat-sheet)

---

## 1. Neon Là Gì?

### 1.1. Định nghĩa

**Neon** (neon.tech) là nền tảng **serverless Postgres** — tức là PostgreSQL đầy đủ tính năng nhưng được "serverless hóa":

- **Tách compute và storage:** compute (CPU/RAM xử lý query) tách riêng khỏi storage (dữ liệu trên disk). Khi app không query, compute **scale về 0** → không tốn tiền, không tốn tài nguyên.
- **Postgres thật 100%:** Không phải "Postgres-like". Mọi extension, SQL syntax, tool (psql, Prisma, Drizzle) đều chạy bình thường.
- **Khởi động nhanh:** Cold start ~500ms (nhanh hơn nhiều so với provision RDS).

**Kiến trúc đơn giản:**

```
App của bạn
    │
    ▼ (Postgres wire protocol)
Neon Proxy ──► Compute (autoscale, scale-to-zero)
                    │
                    ▼
              Storage (tách riêng, luôn sẵn sàng)
```

### 1.2. Ai tạo ra Neon?

- Founded 2021 bởi team từng làm ở **SingleStore** và **Databricks**
- Gọi vốn lớn từ các quỹ đầu tư mạo hiểm
- Open source phần storage engine (pageserver) trên GitHub

### 1.3. Khi nào dùng Neon?

**Nên dùng:**
- Side project, MVP cần Postgres mà không muốn quản lý server
- App có traffic thất thường (lúc đông lúc vắng) → scale to zero tiết kiệm
- Cần **database branching** cho dev/staging/preview (tính năng độc quyền, mục 6)
- Deploy trên Vercel/Netlify/Cloudflare Workers (serverless-friendly)

**Cân nhắc kỹ:**
- App cần connection persistent lâu, traffic đều 24/7 → VPS Postgres có thể rẻ hơn
- Cần extension Postgres quá đặc thù mà Neon chưa hỗ trợ (kiểm tra danh sách extension được hỗ trợ)

---

## 2. Neon vs Postgres Thường vs Supabase

### 2.1. So sánh nhanh

| Tiêu chí | Neon | Postgres tự host (VPS) | Supabase |
|---|---|---|---|
| Bản chất | Serverless Postgres | Postgres gốc tự quản | Postgres + BaaS (auth, storage, realtime) |
| Quản lý server | Không cần | Tự lo toàn bộ | Không cần |
| Scale to zero | ✅ Có | ❌ Không | ✅ (gói mới có) |
| Database branching | ✅ Điểm mạnh nhất | ❌ Tự làm thủ công | ❌ Không có |
| Free tier | 0.5GB storage, hào phóng | Tốn tiền VPS | 500MB, pause sau 7 ngày idle |
| Cold start | ~500ms | Không có | Tương tự |
| Phù hợp | Dev cần branching, app serverless | Production lớn, cần kiểm soát | App cần auth/storage/realtime sẵn |

### 2.2. Chọn cái nào?

- **Chỉ cần database Postgres ngon, rẻ, dễ:** → **Neon**
- **Cần cả auth, file storage, realtime:** → **Supabase**
- **Team lớn, cần kiểm soát hạ tầng:** → **RDS / tự host**

> 💡 Neon và Supabase không loại trừ nhau — nhiều team dùng Neon làm database chính vì branching, Supabase cho auth.

---

## 3. Đăng Ký & Tạo Project

### 3.1. Đăng ký

1. Vào https://neon.tech → **Sign Up**
2. Đăng nhập bằng **GitHub / Google / Email**
3. Không cần thẻ tín dụng cho free tier

### 3.2. Tạo project

1. Dashboard → **New Project**
2. Điền:
   - **Name:** ví dụ `shopdb`
   - **Region:** chọn gần VN nhất — `Singapore (ap-southeast-1)` hoặc `Tokyo`
   - **Postgres version:** để mặc định (bản mới nhất, ví dụ 16/17)
3. Bấm **Create Project** → xong trong vài giây

### 3.3. Cấu trúc project

```
Project: shopdb
├── Branch: main (mặc định)
│   ├── Database: neondb (mặc định)
│   │   └── Tables: users, products, ...
│   └── Compute: endpoint chính
├── Branch: dev (bạn tự tạo)
└── Branch: feature-x (tự động cho preview deploy)
```

- **Project** ≈ 1 cụm Postgres
- **Branch** ≈ 1 bản copy độc lập của data (như git branch)
- **Database** ≈ database Postgres truyền thống trong branch
- **Compute endpoint** ≈ địa chỉ để app kết nối

### 3.4. Tạo thêm database

Trong 1 branch có thể có nhiều database:

1. Vào project → **Databases** → **New Database**
2. Đặt tên (ví dụ `analytics`), chọn branch chứa nó
3. Connection string sẽ khác nhau theo database

---

## 4. Kết Nối Database

### 4.1. Lấy connection string

1. Dashboard → chọn project → bấm **Connect** (hoặc **Dashboard** → **Connection Details**)
2. Chọn:
   - **Branch:** main
   - **Database:** neondb
   - **Role:** owner (mặc định)
3. Copy string dạng:

```
postgresql://neondb_owner:AbCdEfGh1234@ep-cool-darkness-123456.ap-southeast-1.aws.neon.tech/neondb?sslmode=require
```

Cấu trúc:

```
postgresql://[user]:[password]@[endpoint-host]/[dbname]?sslmode=require
```

> ⚠️ **Luôn giữ `?sslmode=require`.** Neon bắt buộc SSL.

### 4.2. Kết nối bằng psql

```bash
# Cài psql (nếu chưa có)
sudo apt install -y postgresql-client

# Kết nối (dán connection string)
psql "postgresql://neondb_owner:AbCdEfGh1234@ep-cool-darkness-123456.ap-southeast-1.aws.neon.tech/neondb?sslmode=require"

# Test
neondb=> SELECT version();
neondb=> \dt
neondb=> \q
```

### 4.3. Các tham số connection string hay dùng

```
# Thêm tên app (hiện trong monitoring)
?sslmode=require&application_name=myapp

# Pooling: dùng PgBouncer của Neon (khuyên dùng cho serverless)
# Thay port 5432 → 6543 và thêm -pooler vào host:
postgresql://user:pass@ep-xxx-pooler.ap-southeast-1.aws.neon.tech/neondb?sslmode=require
```

> 💡 **Pooler (PgBouncer):** Bắt buộc dùng khi deploy trên Vercel/Lambda/Cloudflare Workers — vì serverless tạo quá nhiều connection ngắn hạn, pooler gộp lại giúp không vượt giới hạn.

---

## 5. SQL Editor Trên Dashboard

Neon có SQL editor ngay trên web — không cần cài gì để vọc.

### 5.1. Mở editor

Dashboard → project → tab **SQL Editor** → chọn branch + database.

### 5.2. Tạo bảng và thêm dữ liệu

```sql
-- Tạo bảng users
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  ten VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  tuoi INT CHECK (tuoi >= 0),
  ngay_tao TIMESTAMPTZ DEFAULT NOW()
);

-- Tạo bảng products
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  ten VARCHAR(200) NOT NULL,
  gia BIGINT NOT NULL CHECK (gia >= 0),
  hang VARCHAR(100),
  ton_kho INT DEFAULT 0,
  ngay_tao TIMESTAMPTZ DEFAULT NOW()
);

-- Thêm dữ liệu
INSERT INTO users (ten, email, tuoi) VALUES
  ('Nguyễn Văn An', 'an@example.com', 25),
  ('Trần Thị Bình', 'binh@example.com', 30),
  ('Lê Văn Chi', 'chi@example.com', 22);

INSERT INTO products (ten, gia, hang, ton_kho) VALUES
  ('iPhone 15 Pro', 28990000, 'Apple', 50),
  ('Galaxy S24', 22990000, 'Samsung', 30),
  ('Xiaomi 14', 14990000, 'Xiaomi', 100);
```

### 5.3. Truy vấn cơ bản

```sql
-- Tất cả user
SELECT * FROM users;

-- Lọc + sắp xếp + giới hạn
SELECT ten, gia FROM products
WHERE gia >= 20000000
ORDER BY gia DESC
LIMIT 5;

-- JOIN
SELECT o.id, u.ten, o.tong_tien
FROM orders o
JOIN users u ON u.id = o.user_id;

-- Gom nhóm
SELECT hang, COUNT(*) AS so_sp, AVG(gia)::BIGINT AS gia_tb
FROM products
GROUP BY hang
ORDER BY so_sp DESC;
```

### 5.4. Xem schema

```sql
-- Liệt kê bảng
\dt

-- Cấu trúc bảng
\d users

-- Xem indexes
\di
```

---

## 6. Branching — Tính Năng "Git Cho Database"

Đây là **tính năng độc quyền** làm Neon nổi bật: tạo bản copy của database **ngay lập tức** (copy-on-write, không tốn thời gian copy dữ liệu).

### 6.1. Branch là gì?

- Mỗi branch là 1 **bản sao độc lập** của database tại thời điểm tạo
- Thay đổi trên branch con **không ảnh hưởng** branch cha
- Tạo branch mới chỉ mất **vài giây** dù data hàng GB (nhờ copy-on-write)

### 6.2. Tạo branch trên dashboard

1. Project → tab **Branches** → **New Branch**
2. Chọn **Parent branch:** main
3. Đặt tên: `dev`, `staging`, `feature-xyz`
4. Bấm **Create** → có ngay endpoint riêng

### 6.3. Workflow thực tế

```
main (production)
 ├── dev (team dev vọc thoải mái)
 ├── staging (test trước khi release)
 └── pr-123 (tự động tạo cho mỗi pull request)
```

**Ví dụ:** Muốn test migration nguy hiểm:

```bash
# 1. Tạo branch test
# (trên dashboard hoặc CLI)

# 2. Chạy migration trên branch test
psql "$TEST_BRANCH_URL" -f migration.sql

# 3. OK → chạy trên main
# Lỗi → xóa branch test, main vẫn nguyên vẹn
```

### 6.4. Branching + Vercel Preview

Khi dùng Neon + Vercel:
- Mỗi Pull Request → Vercel tạo preview deploy
- Có thể cấu hình để mỗi PR tự tạo 1 Neon branch riêng
- Test tính năng với data thật mà **không động vào production**

### 6.5. Reset branch

Branch dev bị "bẩn" sau nhiều lần test? **Reset** về trạng thái của parent:

1. Branches → chọn branch → **Reset from parent**
2. Chọn parent (main) và thời điểm
3. Xong — branch sạch như mới

### 6.6. So sánh với cách truyền thống

| Cách truyền thống | Neon branching |
|---|---|
| Dump production (chậm, GB data) | Tạo branch (vài giây) |
| Restore vào staging (chậm) | Copy-on-write tức thì |
| Data cũ ngay sau khi restore | Có thể branch từ **bất kỳ thời điểm nào** (PITR) |
| Tốn disk gấp đôi | Chỉ tốn phần data thay đổi |

---

## 7. Kết Nối Từ Node.js

### 7.1. Dùng `pg` (node-postgres)

```bash
npm install pg dotenv
```

```js
// db.js
const { Pool } = require('pg');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: { rejectUnauthorized: false },  // Neon yêu cầu SSL
  max: 10,                             // số connection tối đa trong pool
});

pool.on('error', (err) => console.error('Pool error:', err));

module.exports = pool;
```

```js
// app.js
const pool = require('./db');

async function main() {
  // Query đơn giản
  const { rows } = await pool.query('SELECT * FROM users LIMIT 5');
  console.log(rows);

  // Parameterized query (CHỐNG SQL INJECTION)
  const email = 'an@example.com';
  const res = await pool.query(
    'SELECT * FROM users WHERE email = $1',
    [email]
  );
  console.log(res.rows[0]);

  // Insert và lấy id trả về
  const insert = await pool.query(
    'INSERT INTO users (ten, email, tuoi) VALUES ($1, $2, $3) RETURNING id',
    ['Phạm Văn Đức', 'duc@example.com', 28]
  );
  console.log('New user id:', insert.rows[0].id);
}

main().finally(() => pool.end());
```

### 7.2. Transaction với pg

```js
const client = await pool.connect();
try {
  await client.query('BEGIN');
  await client.query(
    'UPDATE accounts SET so_du = so_du - $1 WHERE id = $2',
    [100000, 'accA']
  );
  await client.query(
    'UPDATE accounts SET so_du = so_du + $1 WHERE id = $2',
    [100000, 'accB']
  );
  await client.query('COMMIT');
  console.log('✅ Chuyển tiền thành công');
} catch (e) {
  await client.query('ROLLBACK');
  console.error('❌ Rollback:', e.message);
} finally {
  client.release();
}
```

---

## 8. Neon Serverless Driver

### 8.1. Tại sao cần driver riêng?

Môi trường **serverless/edge** (Vercel Edge, Cloudflare Workers) không hỗ trợ TCP socket truyền thống mà `pg` cần. Neon cung cấp driver dùng **HTTP/WebSocket** thay thế.

```bash
npm install @neondatabase/serverless
```

### 8.2. Cách dùng cơ bản (HTTP)

```js
import { neon } from '@neondatabase/serverless';

const sql = neon(process.env.DATABASE_URL);

// Query như tagged template — tự chống SQL injection
const users = await sql`SELECT * FROM users WHERE tuoi > ${25}`;
console.log(users);

// Insert
const [newUser] = await sql`
  INSERT INTO users (ten, email, tuoi)
  VALUES (${'An'}, ${'an2@example.com'}, ${26})
  RETURNING id, ten
`;
```

### 8.3. Transaction

```js
import { neon } from '@neondatabase/serverless';
const sql = neon(process.env.DATABASE_URL);

await sql.transaction([
  sql`UPDATE accounts SET so_du = so_du - 100000 WHERE id = 'accA'`,
  sql`UPDATE accounts SET so_du = so_du + 100000 WHERE id = 'accB'`,
]);
```

### 8.4. Khi nào dùng driver nào?

| Môi trường | Driver nên dùng |
|---|---|
| Node.js server (Express, Fastify) | `pg` + pooler |
| Vercel Serverless Functions | `@neondatabase/serverless` |
| Vercel Edge / Cloudflare Workers | `@neondatabase/serverless` (bắt buộc) |
| Prisma / Drizzle | Cấu hình theo docs của ORM (mục 10, 11) |

---

## 9. Kết Nối Từ Python

### 9.1. Dùng psycopg2

```bash
pip install psycopg2-binary python-dotenv
```

```python
import os
import psycopg2
from psycopg2.extras import RealDictCursor
from dotenv import load_dotenv

load_dotenv()

conn = psycopg2.connect(os.getenv("DATABASE_URL"))
# DATABASE_URL=postgresql://user:pass@ep-xxx.ap-southeast-1.aws.neon.tech/neondb?sslmode=require

# Đọc: trả về dict thay vì tuple
with conn.cursor(cursor_factory=RealDictCursor) as cur:
    cur.execute("SELECT id, ten, email FROM users WHERE tuoi > %s", (25,))
    for row in cur.fetchall():
        print(row["ten"], row["email"])

# Ghi: nhớ commit
with conn.cursor() as cur:
    cur.execute(
        "INSERT INTO users (ten, email, tuoi) VALUES (%s, %s, %s) RETURNING id",
        ("Phạm Văn Đức", "duc@example.com", 28),
    )
    new_id = cur.fetchone()[0]
    conn.commit()
    print("New id:", new_id)

conn.close()
```

### 9.2. Dùng SQLAlchemy

```bash
pip install sqlalchemy psycopg2-binary
```

```python
from sqlalchemy import create_engine, text
import os

engine = create_engine(os.getenv("DATABASE_URL"), pool_size=5)

with engine.connect() as conn:
    result = conn.execute(text("SELECT COUNT(*) FROM users"))
    print("Total users:", result.scalar())

    # Transaction tự động commit/rollback
    with engine.begin() as trans:
        trans.execute(text("UPDATE accounts SET so_du = so_du - 100000 WHERE id='accA'"))
        trans.execute(text("UPDATE accounts SET so_du = so_du + 100000 WHERE id='accB'"))
```

### 9.3. Dùng với FastAPI

```python
from fastapi import FastAPI, HTTPException
import psycopg2
from psycopg2.extras import RealDictCursor
import os

app = FastAPI()

def get_conn():
    return psycopg2.connect(
        os.getenv("DATABASE_URL"),
        cursor_factory=RealDictCursor,
    )

@app.get("/users")
def list_users(limit: int = 20):
    conn = get_conn()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT id, ten, email FROM users LIMIT %s", (limit,))
            return cur.fetchall()
    finally:
        conn.close()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    conn = get_conn()
    try:
        with conn.cursor() as cur:
            cur.execute("SELECT * FROM users WHERE id = %s", (user_id,))
            row = cur.fetchone()
            if not row:
                raise HTTPException(404, "Không tìm thấy user")
            return dict(row)
    finally:
        conn.close()
```

---

## 10. Dùng Với Prisma ORM

### 10.1. Setup

```bash
npm install prisma @prisma/client
npx prisma init
```

```env
# .env — dùng pooler cho production
DATABASE_URL="postgresql://user:pass@ep-xxx-pooler.ap-southeast-1.aws.neon.tech/neondb?sslmode=require"
# Direct connection cho migrate
DIRECT_URL="postgresql://user:pass@ep-xxx.ap-southeast-1.aws.neon.tech/neondb?sslmode=require"
```

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

model User {
  id      Int      @id @default(autoincrement())
  ten     String
  email   String   @unique
  tuoi    Int?
  ngayTao DateTime @default(now()) @map("ngay_tao")
  orders  Order[]

  @@map("users")
}

model Product {
  id     Int    @id @default(autoincrement())
  ten    String
  gia    BigInt
  hang   String?
  tonKho Int    @default(0) @map("ton_kho")

  @@map("products")
}

model Order {
  id       Int      @id @default(autoincrement())
  userId   Int      @map("user_id")
  user     User     @relation(fields: [userId], references: [id])
  tongTien BigInt   @map("tong_tien")
  ngayDat  DateTime @default(now()) @map("ngay_dat")

  @@map("orders")
}
```

### 10.2. Migrate

```bash
# Tạo migration từ schema
npx prisma migrate dev --name init

# Đẩy schema trực tiếp (cho prototype, không tạo migration file)
npx prisma db push

# Mở Prisma Studio (GUI)
npx prisma studio
```

> ⚠️ **Với Neon:** `migrate dev` cần `DIRECT_URL` (không qua pooler) vì pooler không hỗ trợ DDL transaction tốt.

### 10.3. Query với Prisma Client

```js
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

// Create
const user = await prisma.user.create({
  data: { ten: 'An', email: 'an@example.com', tuoi: 25 },
});

// Read
const users = await prisma.user.findMany({
  where: { tuoi: { gt: 20 } },
  orderBy: { ngayTao: 'desc' },
  take: 10,
  include: { orders: true },   // join orders
});

// Update
await prisma.user.update({
  where: { email: 'an@example.com' },
  data: { tuoi: 26 },
});

// Delete
await prisma.user.delete({ where: { id: 1 } });
```

---

## 11. Dùng Với Drizzle ORM

Drizzle nhẹ hơn Prisma, gần SQL hơn, rất hợp với Neon serverless.

```bash
npm install drizzle-orm @neondatabase/serverless
npm install -D drizzle-kit
```

```js
// db/schema.js
import { pgTable, serial, varchar, integer, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  ten: varchar('ten', { length: 100 }).notNull(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  tuoi: integer('tuoi'),
  ngayTao: timestamp('ngay_tao', { withTimezone: true }).defaultNow(),
});
```

```js
// db/index.js
import { drizzle } from 'drizzle-orm/neon-http';
import { neon } from '@neondatabase/serverless';
import * as schema from './schema.js';

const sql = neon(process.env.DATABASE_URL);
export const db = drizzle(sql, { schema });
```

```js
// usage
import { db } from './db/index.js';
import { users } from './db/schema.js';
import { eq, gt, desc } from 'drizzle-orm';

// Insert
await db.insert(users).values({ ten: 'An', email: 'an@example.com', tuoi: 25 });

// Select
const result = await db.select().from(users)
  .where(gt(users.tuoi, 20))
  .orderBy(desc(users.ngayTao))
  .limit(10);

// Update / Delete
await db.update(users).set({ tuoi: 26 }).where(eq(users.email, 'an@example.com'));
await db.delete(users).where(eq(users.id, 1));
```

```bash
# Migrate với drizzle-kit
npx drizzle-kit push
npx drizzle-kit studio   # GUI
```

---

## 12. Autoscaling & Scale to Zero

### 12.1. Cơ chế

- **Scale to zero:** Sau ~5 phút không có query, compute tắt → **không tốn compute hours**
- **Cold start:** Query đầu tiên sau khi tắt mất ~500ms để bật lại
- **Autoscale:** Khi tải tăng, Neon tự tăng CU (Compute Units)

### 12.2. Cấu hình

Dashboard → project → **Compute** → chỉnh:
- **Min CU:** 0 (cho phép scale to zero) hoặc 0.25 (luôn bật, tránh cold start)
- **Max CU:** giới hạn trần (tránh bill bất ngờ)

### 12.3. Khi nào TẮT scale-to-zero?

- App production cần latency thấp ổn định → set Min CU = 0.25+
- Cron job chạy mỗi phút (compute sẽ chẳng bao giờ tắt được)

### 12.4. Theo dõi usage

Dashboard → **Monitoring**: xem compute hours, storage, data transfer theo thời gian thực.

---

## 13. Point-in-Time Restore

Neon lưu lịch sử thay đổi → có thể khôi phục về **bất kỳ thời điểm nào** trong retention window.

### 13.1. Restore qua dashboard

1. Branches → **New Branch**
2. Chọn parent branch + **Point in time** (chọn ngày giờ)
3. Đặt tên branch mới → Create
4. Branch mới chứa data đúng thời điểm đã chọn

### 13.2. Trường hợp dùng

- Ai đó `DELETE` nhầm toàn bộ bảng lúc 14:30 → tạo branch từ 14:25
- Test migration trên data "của 1 tuần trước"

### 13.3. Retention

- Free tier: 7 ngày (kiểm tra docs mới nhất vì có thể thay đổi)
- Gói trả phí: retention dài hơn

---

## 14. Quản Lý Với Neon CLI

```bash
# Cài đặt
npm install -g neonctl

# Đăng nhập
neonctl auth

# Liệt kê projects
neonctl projects list

# Tạo project
neonctl projects create --name shopdb --region-id aws-ap-southeast-1

# Liệt kê branches
neonctl branches list --project-id <project-id>

# Tạo branch
neonctl branches create --project-id <id> --name dev

# Lấy connection string
neonctl connection-string --project-id <id> --branch main

# Chạy SQL trực tiếp
neonctl sql --project-id <id> --branch main --sql "SELECT COUNT(*) FROM users"
```

---

## 15. Bảo Mật — Roles & Connection String

### 15.1. Tạo role riêng cho app (không dùng owner)

```sql
-- Tạo role chỉ có quyền trên database shopdb
CREATE ROLE app_user WITH LOGIN PASSWORD 'mat-khau-manh';
GRANT CONNECT ON DATABASE neondb TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;
```

Sau đó tạo connection string với role này cho app production.

### 15.2. Quy tắc an toàn

1. **Không commit connection string** lên Git — dùng biến môi trường
2. **Dùng pooler** (`-pooler` host) cho app serverless
3. **Rotate password** định kỳ: Dashboard → Roles → Reset password
4. **Tách role:** `owner` cho migrate, `app_user` cho app chạy

```env
# .env (KHÔNG commit file này!)
DATABASE_URL=postgresql://app_user:xxx@ep-xxx-pooler.../neondb?sslmode=require
```

```gitignore
# .gitignore
.env
.env.local
```

---

## 16. Pricing & Free Tier

### 16.1. Free tier (2026)

| Tài nguyên | Free tier |
|---|---|
| Projects | 10 |
| Storage | 0.5 GB / project |
| Compute hours | 100 giờ / tháng |
| Branches | Không giới hạn (trong storage) |
| Thẻ tín dụng | Không cần |

> 💡 100 compute hours + scale-to-zero: với side project, gần như không bao giờ hết.

### 16.2. Gói trả phí (Launch/Scale)

| Gói | Giá từ | Điểm khác |
|---|---|---|
| Launch | ~$19/tháng | Thêm storage, compute, PITR dài hơn |
| Scale | Pay-as-you-go | Trả theo dùng thực tế |

### 16.3. Ước tính chi phí

- **Blog cá nhân** (vài trăm visit/ngày): free tier đủ
- **SaaS nhỏ** (10k user): khoảng $20-50/tháng
- **Tip tiết kiệm:** bật scale-to-zero, dùng pooler, tránh query N+1

---

## 17. Troubleshooting

| Lỗi | Nguyên nhân | Cách fix |
|---|---|---|
| `connection refused` / timeout | Sai host, quên `sslmode=require` | Kiểm tra connection string, thêm `?sslmode=require` |
| `password authentication failed` | Sai password, chưa encode ký tự đặc biệt | Encode `@`→`%40`, `/`→`%2F`; reset password trên dashboard |
| `too many clients` | Vượt max connections | Dùng pooler host (`-pooler`), giảm pool size |
| Cold start chậm (~500ms) | Compute đang scale từ 0 | Set Min CU > 0 cho production |
| Prisma migrate lỗi trên pooler | Pooler không hỗ trợ DDL | Dùng `DIRECT_URL` (non-pooler) cho migrate |
| `SSL SYSCALL error` | Network gián đoạn | Retry với backoff; kiểm tra firewall |
| Branch không thấy data mới | Đang nhìn nhầm branch | Kiểm tra branch trong connection string / dashboard |
| Hết free compute hours | Query quá nhiều | Bật scale-to-zero, kiểm tra Monitoring xem ai ngốn |

### Debug connection

```bash
# Test TCP + SSL
psql "postgresql://..." -c "SELECT 1"

# Xem connections hiện tại
SELECT count(*) FROM pg_stat_activity;

# Xem query đang chạy
SELECT pid, usename, query, state
FROM pg_stat_activity
WHERE state != 'idle';
```

---

## 18. Best Practices & Cheat Sheet

### 18.1. Best practices

1. **Dùng pooler** cho mọi app serverless/production
2. **Branch cho mọi thứ:** dev, staging, PR preview, test migration
3. **Migrate bằng DIRECT_URL**, app chạy bằng pooler URL
4. **Index** cho foreign key và field hay filter
5. **Không SELECT \*** trong production — liệt kê cột cần thiết
6. **Backup logic:** PITR + branch định kỳ trước migration lớn
7. **Monitoring:** để ý compute hours, set budget alert

### 18.2. Cheat sheet SQL thường dùng

```sql
-- Tạo bảng
CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  ten TEXT NOT NULL,
  gia BIGINT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Thêm cột
ALTER TABLE products ADD COLUMN hang TEXT;

-- Index
CREATE INDEX idx_products_hang ON products (hang);
CREATE UNIQUE INDEX idx_users_email ON users (email);

-- Upsert
INSERT INTO users (email, ten) VALUES ('a@x.com', 'An')
ON CONFLICT (email) DO UPDATE SET ten = EXCLUDED.ten;

-- Xóa trùng
DELETE FROM users a USING users b
WHERE a.id < b.id AND a.email = b.email;

-- Kích thước bảng
SELECT pg_size_pretty(pg_total_relation_size('products'));
```

### 18.3. Tài liệu tham khảo

- 📘 Docs: https://neon.tech/docs
- 🔌 Serverless driver: https://github.com/neondatabase/serverless
- 🖥️ CLI: https://neon.tech/docs/reference/cli-install
- 💬 Discord cộng đồng: https://discord.gg/neon

---

*📅 Cập nhật: 2026-10-10 — Tổng 18 phần, từ setup đến production với branching.*

---

## 19. Migrate Dữ Liệu Lên Neon

### 19.1. Từ Postgres local (pg_dump/pg_restore)

```bash
# Dump từ local
pg_dump "postgresql://postgres:matkhau@localhost:5432/shopdb" \
  --no-owner --no-acl -f backup.sql

# Restore lên Neon (dùng DIRECT_URL, không qua pooler)
psql "postgresql://user:pass@ep-xxx.ap-southeast-1.aws.neon.tech/neondb?sslmode=require" \
  -f backup.sql
```

### 19.2. Từ MySQL

Dùng `pgloader` để convert tự động:

```bash
# Cài pgloader
sudo apt install -y pgloader

# Tạo file migrate.load
cat > migrate.load << 'EOF'
LOAD DATABASE
  FROM mysql://root:matkhau@localhost/shopdb
  INTO postgresql://user:pass@ep-xxx.neon.tech/neondb?sslmode=require
  WITH include drop, create tables, create indexes, reset sequences
  SET maintenance_work_mem to '128MB';
EOF

pgloader migrate.load
```

### 19.3. Từ MongoDB / NoSQL

Không có tool tự động hoàn hảo — quy trình thủ công:

1. Export MongoDB ra JSON: `mongoexport --collection=users --out=users.json`
2. Viết script Python chuyển JSON → INSERT SQL
3. Chạy trên Neon branch test trước

```python
import json, psycopg2

conn = psycopg2.connect("postgresql://...")
cur = conn.cursor()

with open("users.json") as f:
    for line in f:
        doc = json.loads(line)
        cur.execute(
            "INSERT INTO users (ten, email) VALUES (%s, %s) ON CONFLICT DO NOTHING",
            (doc.get("ten"), doc.get("email")),
        )
conn.commit()
```

---

## 20. Monitoring & Observability

### 20.1. Dashboard Monitoring

Neon Dashboard → **Monitoring** cho thấy:
- **Compute usage:** CU-hours theo giờ/ngày
- **Storage:** dung lượng theo branch
- **Active connections:** số connection đang mở
- **Top queries:** query chậm nhất (gói trả phí)

### 20.2. Query chậm — tìm và fix

```sql
-- Bật pg_stat_statements (thường đã bật sẵn)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Top 10 query chậm nhất
SELECT query, calls, mean_exec_time::INT AS avg_ms, max_exec_time::INT AS max_ms
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Reset stats
SELECT pg_stat_statements_reset();
```

### 20.3. Kiểm tra index không dùng

```sql
-- Index chưa bao giờ được scan (có thể xóa)
SELECT schemaname, tablename, indexrelname
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND indexrelname NOT LIKE '%pkey%';
```

### 20.4. Kích thước database

```sql
-- Dung lượng từng bảng
SELECT tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Tổng database
SELECT pg_size_pretty(pg_database_size(current_database()));
```

---

## 21. Neon Với Các Framework Phổ Biến

### 21.1. Next.js (App Router)

```bash
npx create-next-app@latest myapp
npm install @neondatabase/serverless
```

```js
// lib/db.js
import { neon } from '@neondatabase/serverless';
export const sql = neon(process.env.DATABASE_URL);
```

```js
// app/api/users/route.js
import { sql } from '@/lib/db';
import { NextResponse } from 'next/server';

export async function GET() {
  const users = await sql`SELECT id, ten, email FROM users LIMIT 20`;
  return NextResponse.json(users);
}

export async function POST(req) {
  const { ten, email } = await req.json();
  const [user] = await sql`
    INSERT INTO users (ten, email) VALUES (${ten}, ${email})
    RETURNING id, ten, email
  `;
  return NextResponse.json(user, { status: 201 });
}
```

### 21.2. Django

```python
# settings.py
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "neondb",
        "USER": "neondb_owner",
        "PASSWORD": os.getenv("DB_PASSWORD"),
        "HOST": "ep-xxx.ap-southeast-1.aws.neon.tech",
        "PORT": "5432",
        "OPTIONS": {"sslmode": "require"},
        # Pool cho production
        "CONN_MAX_AGE": 60,
    }
}
```

```bash
python manage.py migrate
```

### 21.3. Laravel

```env
# .env
DB_CONNECTION=pgsql
DB_HOST=ep-xxx-pooler.ap-southeast-1.aws.neon.tech
DB_PORT=6543
DB_DATABASE=neondb
DB_USERNAME=neondb_owner
DB_PASSWORD=xxx
```

> 💡 Laravel + Neon: dùng pooler port **6543** để tránh hết connections với queue workers.

---

## 22. So Sánh Chi Tiết Neon vs Đối Thủ

### 22.1. Tính năng

| Tính năng | Neon | Supabase | PlanetScale | RDS Postgres |
|---|---|---|---|---|
| Branching | ✅ Instant | ❌ | ✅ | ❌ |
| Scale to zero | ✅ | ✅ (mới) | ✅ | ❌ |
| Free tier | 0.5GB | 500MB | 5GB | Không free |
| Extensions PG | Hầu hết | Hầu hết + custom | N/A (MySQL) | Tất cả |
| Pooler built-in | ✅ PgBouncer | ✅ Supavisor | ✅ | Tự setup |
| Dashboard SQL editor | ✅ | ✅ | ✅ Web console | ❌ |

### 22.2. Khi nào KHÔNG nên dùng Neon?

- Cần **PostGIS** phiên bản quá mới (kiểm tra supported extensions)
- Workload **write-heavy 24/7** với latency cực thấp → dedicated RDS có thể ổn định hơn
- Yêu cầu **data residency** đặc biệt (Neon regions có hạn)
- Team đã có DBA quản lý Postgres on-prem tốt

---

## 23. Câu Hỏi Thường Gặp (FAQ)

**Hỏi: Free tier có bị xóa data không?**
Đáp: Không. Khác với Supabase (pause sau 7 ngày idle), Neon free không xóa data. Compute scale về 0 nhưng data vẫn nguyên.

**Hỏi: Cold start 500ms có ảnh hưởng UX không?**
Đáp: Chỉ query đầu tiên sau idle. Với web app, có thể warm bằng cron ping mỗi 4 phút, hoặc set Min CU > 0.

**Hỏi: Branch có tốn tiền không?**
Đáp: Branch dùng copy-on-write — chỉ tốn storage cho phần data **thay đổi** so với parent. Tạo 10 branch từ main 100MB mà không sửa gì ≈ tốn thêm ~0MB.

**Hỏi: Có thể dùng Neon cho production thật không?**
Đáp: Có. Nhiều startup chạy production trên Neon. Nhớ: dùng pooler, set Min CU phù hợp, bật budget alert.

**Hỏi: Migrate khỏi Neon có khó không?**
Đáp: Không — vì là Postgres chuẩn, `pg_dump` ra và restore ở đâu cũng được. Không bị vendor lock-in.

---

*Cập nhật: đã bổ sung migrate, monitoring, framework integrations, FAQ — tổng 23 phần.*
