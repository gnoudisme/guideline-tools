# Hướng Dẫn Toàn Tập Turso — SQLite Ở Mọi Nơi (Edge Database)

> **Cấp độ:** Người mới bắt đầu → dùng được thật trong dự án
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10
>
> ⚠️ Turso = SQLite chạy ở edge (gần user nhất) + sync về cloud. Free tier hào phóng, không cần thẻ.

---

## Mục Lục

1. [Turso Là Gì?](#1-turso-là-gì)
2. [libSQL Là Gì? Quan Hệ Với SQLite](#2-libsql-là-gì-quan-hệ-với-sqlite)
3. [Turso vs Neon vs Supabase vs PlanetScale](#3-turso-vs-neon-vs-supabase-vs-planetscale)
4. [Đăng Ký Tài Khoản](#4-đăng-ký-tài-khoản)
5. [Cài Đặt Turso CLI](#5-cài-đặt-turso-cli)
6. [Tạo Database Đầu Tiên](#6-tạo-database-đầu-tiên)
7. [Turso CLI — Lệnh Đầy Đủ](#7-turso-cli--lệnh-đầy-đủ)
8. [Làm Việc Với SQL Trên Turso](#8-làm-việc-với-sql-trên-turso)
9. [Groups & Locations — Đưa Data Ra Edge](#9-groups--locations--đưa-data-ra-edge)
10. [Embedded Replicas — SQLite Local Sync Cloud](#10-embedded-replicas--sqlite-local-sync-cloud)
11. [Kết Nối Từ Node.js](#11-kết-nối-từ-nodejs)
12. [Kết Nối Từ Python](#12-kết-nối-từ-python)
13. [Dùng Với Drizzle ORM](#13-dùng-với-drizzle-orm)
14. [Dùng Với Prisma](#14-dùng-với-prisma)
15. [Auth Token — Xác Thực](#15-auth-token--xác-thực)
16. [Branching Trên Turso](#16-branching-trên-turso)
17. [Pricing & Free Tier](#17-pricing--free-tier)
18. [Troubleshooting](#18-troubleshooting)
19. [Best Practices & Cheat Sheet](#19-best-practices--cheat-sheet)

---

## 1. Turso Là Gì?

### 1.1. Định nghĩa

**Turso** là nền tảng **SQLite phân tán ở edge** — tức là database SQLite của bạn được replicate đến hàng chục datacenter trên thế giới, và app luôn đọc từ **vị trí gần user nhất** (vài ms thay vì vài trăm ms).

Ý tưởng cốt lõi:

```
Truyền thống:  User (VN) ──200ms──► DB (US) ──200ms──► User
Turso:         User (VN) ──5ms──► Replica (Singapore) ──► sync về Primary
```

- **Write:** đi về primary location (1 nơi bạn chọn)
- **Read:** từ replica gần nhất (hàng chục locations)
- **Kết quả:** đọc nhanh như SQLite local, nhưng data đồng bộ cloud

### 1.2. Ai đứng sau Turso?

- Team tạo ra **libSQL** (fork của SQLite, open source)
- Gọi vốn từ các quỹ lớn, được cộng đồng dev yêu thích vì DX (developer experience) tốt
- Triết lý: "SQLite là database bị đánh giá thấp nhất — nó đủ cho 99% app"

### 1.3. Khi nào dùng Turso?

**Nên dùng:**
- App global, user ở nhiều châu lục → cần read latency thấp
- App vừa/nhỏ, read nhiều write ít (blog, e-commerce catalog, docs)
- Edge functions (Cloudflare Workers, Vercel Edge) — Turso sinh ra cho môi trường này
- Muốn đơn giản: không muốn quản lý Postgres, thích SQL quen thuộc
- Offline-first / local-first apps (nhờ embedded replicas)

**Cân nhắc kỹ:**
- Write-heavy ở nhiều region (write vẫn phải về primary → latency cao nếu user xa primary)
- Cần tính năng Postgres đặc thù (JSONB operators phức tạp, PostGIS, full-text search nâng cao)
- Dữ liệu quan hệ cực phức tạp với transaction phân tán nặng

---

## 2. libSQL Là Gì? Quan Hệ Với SQLite

### 2.1. libSQL

**libSQL** là fork open-source của SQLite do team Turso phát triển, thêm:
- **Replication:** sync giữa primary và replicas
- **Remote access:** truy cập qua HTTP/WebSocket (SQLite gốc chỉ đọc file local)

```sql
-- 99% SQL của SQLite chạy nguyên si trên libSQL/Turso
CREATE TABLE users (id INTEGER PRIMARY KEY, ten TEXT, email TEXT UNIQUE);
INSERT INTO users (ten, email) VALUES ('An', 'an@example.com');
SELECT * FROM users WHERE email = 'an@example.com';
```

### 2.2. SQLite vs libSQL vs Turso

| | SQLite | libSQL | Turso |
|---|---|---|---|
| Bản chất | File DB local | Fork của SQLite + replication | Cloud platform chạy libSQL |
| Chạy ở đâu | Máy của bạn | Bất kỳ đâu | Edge locations toàn cầu |
| Replication | Không | Có (tự host được) | Có (managed) |
| Dùng khi nào | App local, test | Tự host distributed SQLite | Muốn managed, không lo ops |

> 💡 Bạn có thể tự host libSQL mà không cần Turso — nhưng Turso lo hết replication, backup, dashboard cho bạn.

---

## 3. Turso vs Neon vs Supabase vs PlanetScale

| Tiêu chí | Turso | Neon | Supabase | PlanetScale |
|---|---|---|---|---|
| Engine | SQLite (libSQL) | PostgreSQL | PostgreSQL | MySQL (Vitess) |
| Mô hình | Edge replicas | Serverless, scale-to-zero | BaaS all-in-one | Serverless MySQL |
| Read latency global | **Rất thấp** (edge) | Phụ thuộc region | Phụ thuộc region | Thấp (edge cache) |
| Branching | Có | **Mạnh nhất** | Không | Có |
| Free tier | 500 DBs, 9GB storage | 0.5GB, 100h compute | 500MB, pause 7 ngày | 5GB storage |
| Embedded/local sync | ✅ Độc quyền | ❌ | ❌ | ❌ |
| Phù hợp | App global, edge | App cần Postgres + branching | Cần auth/storage sẵn | Team quen MySQL |

**Chọn Turso khi:** muốn đơn giản nhất, app global, hoặc làm việc nhiều với edge functions.

---

## 4. Đăng Ký Tài Khoản

1. Vào https://turso.tech → **Sign Up** (hoặc https://app.turso.tech)
2. Đăng nhập bằng **GitHub** (khuyên dùng) / Google
3. Không cần thẻ tín dụng
4. Sau khi đăng nhập, bạn vào được **Turso Dashboard** — nhưng 90% thao tác làm qua **CLI** cho nhanh

---

## 5. Cài Đặt Turso CLI

CLI là cách chính để quản lý Turso.

### 5.1. Cài đặt

```bash
# macOS / Linux (script chính thức)
curl -sSfL https://get.tur.so/install.sh | bash

# macOS (Homebrew)
brew install tursodatabase/tap/turso

# Windows (PowerShell)
irm https://get.tur.so/install.ps1 | iex

# Kiểm tra
turso --version
```

> 💡 Sau khi cài bằng script, có thể cần thêm vào PATH:
> ```bash
> export PATH="$HOME/.turso/bin:$PATH"
> # Thêm dòng trên vào ~/.bashrc hoặc ~/.zshrc
> ```

### 5.2. Đăng nhập

```bash
turso auth login
# → Mở trình duyệt, bấm Authorize
# → CLI báo "Logged in successfully"

# Kiểm tra ai đang đăng nhập
turso auth whoami
```

### 5.3. Cập nhật CLI

```bash
turso update
```

---

## 6. Tạo Database Đầu Tiên

```bash
# Tạo database tên mydb (tự chọn location gần nhất)
turso db create mydb

# Chỉ định location (gần VN: Singapore = sin)
turso db create mydb --location sin

# Xem danh sách locations
turso db locations

# Xem database vừa tạo
turso db show mydb
```

Output mẫu:

```
Name:           mydb
ID:             12345678-90ab-cdef-1234-567890abcdef
Group:          default
Version:        sqld-0.24.9
Locations:      sin
Size:           24.6 kB
Archived:       false
```

### Lấy connection info

```bash
# URL để kết nối
turso db show mydb --url
# → libsql://mydb-tenban.turso.io

# Tạo auth token
turso db tokens create mydb
# → eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCJ9...

# Tạo token có thời hạn (an toàn hơn cho production)
turso db tokens create mydb --expiration 720h   # 30 ngày
```

> ⚠️ **Token như password** — không commit lên Git, lưu vào biến môi trường.

---

## 7. Turso CLI — Lệnh Đầy Đủ

### 7.1. Quản lý database

```bash
# Liệt kê
turso db list

# Tạo
turso db create <ten> [--location sin] [--group default]

# Xem chi tiết
turso db show <ten>

# Xóa (cẩn thận!)
turso db destroy <ten>
# → hỏi xác nhận, gõ tên database để confirm

# Đổi tên
turso db rename <ten-cu> <ten-moi>
```

### 7.2. Shell SQL trực tiếp

```bash
# Mở shell, gõ SQL trực tiếp
turso db shell mydb

# Trong shell:
sqlite> CREATE TABLE users (id INTEGER PRIMARY KEY, ten TEXT, email TEXT);
sqlite> INSERT INTO users (ten, email) VALUES ('An', 'an@example.com');
sqlite> SELECT * FROM users;
sqlite> .tables
sqlite> .schema users
sqlite> .quit

# Chạy 1 lệnh SQL duy nhất (không vào shell)
turso db shell mydb "SELECT COUNT(*) FROM users"

# Chạy từ file
turso db shell mydb < schema.sql
```

### 7.3. Tokens

```bash
# Tạo token
turso db tokens create mydb

# Tạo token hết hạn sau 24h
turso db tokens create mydb --expiration 24h

# Liệt kê token (chỉ hiện prefix, không hiện full)
turso db tokens list mydb   # (tùy version CLI)

# Thu hồi token
turso db tokens invalidate mydb
```

### 7.4. Inspect & info

```bash
# Dung lượng, version, locations
turso db inspect mydb

# Xem stats
turso db show mydb --url
```

### 7.5. Groups

```bash
# Liệt kê groups
turso group list

# Tạo group ở Singapore
turso group create mygroup --location sin

# Tạo DB trong group
turso db create mydb --group mygroup

# Thêm location vào group (mở rộng edge)
turso group locations add mygroup nrt   # thêm Tokyo
```

---

## 8. Làm Việc Với SQL Trên Turso

Turso dùng SQLite dialect — nếu bạn biết SQLite là biết Turso.

### 8.1. Tạo bảng

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  ten TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  tuoi INTEGER CHECK (tuoi >= 0),
  ngay_tao TEXT DEFAULT (datetime('now'))
);

CREATE TABLE products (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  ten TEXT NOT NULL,
  gia INTEGER NOT NULL CHECK (gia >= 0),
  hang TEXT,
  ton_kho INTEGER DEFAULT 0
);

CREATE TABLE orders (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL REFERENCES users(id),
  tong_tien INTEGER NOT NULL,
  trang_thai TEXT DEFAULT 'cho_xu_ly',
  ngay_dat TEXT DEFAULT (datetime('now'))
);
```

### 8.2. CRUD

```sql
-- CREATE
INSERT INTO users (ten, email, tuoi)
VALUES ('Nguyễn Văn An', 'an@example.com', 25);

INSERT INTO users (ten, email, tuoi) VALUES
  ('Trần Thị Bình', 'binh@example.com', 30),
  ('Lê Văn Chi', 'chi@example.com', 22);

-- Lấy id vừa insert
SELECT last_insert_rowid();

-- READ
SELECT * FROM users;
SELECT ten, email FROM users WHERE tuoi > 25 ORDER BY tuoi DESC LIMIT 5;

-- UPDATE
UPDATE products SET ton_kho = ton_kho - 1 WHERE id = 1;
UPDATE users SET tuoi = 26 WHERE email = 'an@example.com';

-- DELETE
DELETE FROM users WHERE id = 3;
```

### 8.3. Query nâng cao

```sql
-- JOIN
SELECT o.id, u.ten, o.tong_tien, o.trang_thai
FROM orders o
JOIN users u ON u.id = o.user_id
WHERE o.trang_thai = 'da_thanh_toan';

-- GROUP BY
SELECT hang, COUNT(*) AS so_sp, AVG(gia) AS gia_tb
FROM products
GROUP BY hang
ORDER BY so_sp DESC;

-- Subquery
SELECT ten FROM products
WHERE gia > (SELECT AVG(gia) FROM products);

-- LIKE (tìm kiếm đơn giản)
SELECT * FROM products WHERE ten LIKE '%iPhone%';

-- UPSERT (SQLite 3.24+)
INSERT INTO users (email, ten, tuoi)
VALUES ('an@example.com', 'An Mới', 26)
ON CONFLICT (email) DO UPDATE SET ten = excluded.ten, tuoi = excluded.tuoi;
```

### 8.4. Khác biệt SQLite vs Postgres cần nhớ

| | SQLite/Turso | Postgres |
|---|---|---|
| Auto increment | `INTEGER PRIMARY KEY AUTOINCREMENT` | `SERIAL` / `GENERATED` |
| Boolean | Không có — dùng `INTEGER` 0/1 | Có `BOOLEAN` |
| Timestamp | `TEXT` + `datetime('now')` | `TIMESTAMPTZ DEFAULT NOW()` |
| Upsert | `ON CONFLICT DO UPDATE` | `ON CONFLICT DO UPDATE` (giống!) |
| `RETURNING` | Có (3.35+) | Có |
| Kiểu dữ liệu | Linh hoạt (type affinity) | Chặt chẽ |

---

## 9. Groups & Locations — Đưa Data Ra Edge

### 9.1. Group là gì?

**Group** = 1 primary + nhiều replicas ở các locations bạn chọn. Database thuộc về 1 group.

```
Group "global":
  Primary: sin (Singapore)
  Replicas: nrt (Tokyo), fra (Frankfurt), iad (Virginia)
```

- Write → về `sin`
- Read từ Tokyo → replica `nrt` (vài ms)
- Read từ EU → replica `fra`

### 9.2. Tạo group multi-region

```bash
# Tạo group với primary ở Singapore
turso group create global --location sin

# Thêm replica Tokyo + Frankfurt
turso group locations add global nrt
turso group locations add global fra

# Tạo DB trong group
turso db create myapp --group global

# Kiểm tra
turso db show myapp
# Locations: sin, nrt, fra
```

### 9.3. Chọn location

```bash
turso db locations
```

Một số mã phổ biến:

| Mã | Thành phố | Gần VN? |
|---|---|---|
| `sin` | Singapore | ✅ Gần nhất |
| `hkg` | Hong Kong | ✅ Gần |
| `nrt` | Tokyo | ✅ Khá gần |
| `fra` | Frankfurt | Xa |
| `iad` | Virginia, US | Xa |

> 💡 **Luôn đặt primary ở `sin`** nếu user chủ yếu ở VN/SEA.

### 9.4. Xóa location khỏi group

```bash
turso group locations remove global fra
```

---

## 10. Embedded Replicas — SQLite Local Sync Cloud

Tính năng **độc quyền** của Turso: app của bạn chạy **SQLite file local** (cực nhanh, offline được), và Turso tự **sync 2 chiều** với cloud.

```
┌─────────────┐   sync    ┌──────────────┐
│ App + SQLite │ ◄──────► │ Turso Cloud  │
│ file local   │  2 chiều  │ (primary)    │
└─────────────┘           └──────────────┘
```

### 10.1. Khi nào dùng?

- **Mobile/desktop app** cần offline-first
- **Edge functions** muốn read latency = 0 (đọc file local)
- **Sync engine** cho app đa thiết bị

### 10.2. Ví dụ Node.js

```bash
npm install @libsql/client
```

```js
import { createClient } from "@libsql/client";

// Lần đầu: sync từ cloud về file local
// Lần sau: đọc local, sync ngầm
const db = createClient({
  url: "file:local.db",
  syncUrl: process.env.TURSO_URL,        // libsql://...
  syncToken: process.env.TURSO_TOKEN,
  syncInterval: 60,                       // sync mỗi 60 giây
});

// Dùng như bình thường — cực nhanh vì local
await db.execute("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, ten TEXT)");
await db.execute({ sql: "INSERT INTO users (ten) VALUES (?)", args: ["An"] });
const rs = await db.execute("SELECT * FROM users");
console.log(rs.rows);

// Sync thủ công khi cần
await db.sync();
```

### 10.3. Lưu ý

- Conflict: last-write-wins (đơn giản, phù hợp hầu hết app)
- Không thay thế được giải pháp sync phức tạp kiểu CRDT nếu cần merge logic tinh vi

---

## 11. Kết Nối Từ Node.js

### 11.1. Remote (qua HTTP)

```bash
npm install @libsql/client dotenv
```

```js
// db.js
import { createClient } from "@libsql/client";

export const db = createClient({
  url: process.env.TURSO_URL,      // libsql://mydb-xxx.turso.io
  authToken: process.env.TURSO_TOKEN,
});
```

```js
// app.js
import { db } from "./db.js";

// SELECT
const rs = await db.execute("SELECT id, ten, email FROM users WHERE tuoi > ?", [25]);
for (const row of rs.rows) {
  console.log(row.ten, row.email);
}
// row truy cập: row.ten hoặc row[1]

// INSERT (parameterized — chống SQL injection)
await db.execute({
  sql: "INSERT INTO users (ten, email, tuoi) VALUES (?, ?, ?)",
  args: ["Phạm Văn Đức", "duc@example.com", 28],
});

// UPDATE / DELETE
await db.execute({ sql: "UPDATE users SET tuoi = ? WHERE email = ?", args: [26, "an@example.com"] });
await db.execute({ sql: "DELETE FROM users WHERE id = ?", args: [3] });

// Batch (nhiều câu trong 1 round-trip — nhanh hơn)
await db.batch([
  { sql: "INSERT INTO users (ten, email) VALUES (?, ?)", args: ["A", "a@x.com"] },
  { sql: "INSERT INTO users (ten, email) VALUES (?, ?)", args: ["B", "b@x.com"] },
  "UPDATE products SET ton_kho = ton_kho - 1 WHERE id = 1",
]);
```

### 11.2. Transaction

```js
const tx = await db.transaction();
try {
  await tx.execute({ sql: "UPDATE accounts SET so_du = so_du - ? WHERE id = ?", args: [100000, "A"] });
  await tx.execute({ sql: "UPDATE accounts SET so_du = so_du + ? WHERE id = ?", args: [100000, "B"] });
  await tx.commit();
  console.log("✅ Thành công");
} catch (e) {
  await tx.rollback();
  console.error("❌ Rollback:", e.message);
}
```

### 11.3. Ví dụ Express API

```js
import express from "express";
import { db } from "./db.js";

const app = express();
app.use(express.json());

app.get("/users", async (req, res) => {
  const limit = Math.min(parseInt(req.query.limit) || 20, 100);
  const rs = await db.execute({ sql: "SELECT id, ten, email FROM users LIMIT ?", args: [limit] });
  res.json(rs.rows);
});

app.get("/users/:id", async (req, res) => {
  const rs = await db.execute({ sql: "SELECT * FROM users WHERE id = ?", args: [req.params.id] });
  if (rs.rows.length === 0) return res.status(404).json({ error: "Không tìm thấy" });
  res.json(rs.rows[0]);
});

app.post("/users", async (req, res) => {
  const { ten, email, tuoi } = req.body;
  if (!ten || !email) return res.status(400).json({ error: "Thiếu ten/email" });
  try {
    const rs = await db.execute({
      sql: "INSERT INTO users (ten, email, tuoi) VALUES (?, ?, ?) RETURNING id",
      args: [ten, email, tuoi ?? null],
    });
    res.status(201).json({ id: Number(rs.rows[0].id) });
  } catch (e) {
    if (e.message.includes("UNIQUE")) return res.status(409).json({ error: "Email đã tồn tại" });
    throw e;
  }
});

app.listen(3000, () => console.log("API chạy ở :3000"));
```

---

## 12. Kết Nối Từ Python

```bash
pip install libsql-client python-dotenv
```

```python
import os
import libsql_client
from dotenv import load_dotenv

load_dotenv()

client = libsql_client.create_client_sync(
    url=os.getenv("TURSO_URL"),
    auth_token=os.getenv("TURSO_TOKEN"),
)

# CREATE TABLE
client.execute("""
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        ten TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        tuoi INTEGER
    )
""")

# INSERT
client.execute(
    "INSERT INTO users (ten, email, tuoi) VALUES (?, ?, ?)",
    ["Nguyễn Văn An", "an@example.com", 25],
)

# SELECT
rs = client.execute("SELECT id, ten, email FROM users WHERE tuoi > ?", [20])
for row in rs.rows:
    print(row)  # tuple: (1, 'Nguyễn Văn An', 'an@example.com')

# Lấy theo tên cột (nếu driver hỗ trợ columns)
cols = rs.columns
for row in rs.rows:
    print(dict(zip(cols, row)))

# UPDATE / DELETE
client.execute("UPDATE users SET tuoi = ? WHERE email = ?", [26, "an@example.com"])

# Batch
client.batch([
    "INSERT INTO users (ten, email) VALUES ('A', 'a@x.com')",
    "INSERT INTO users (ten, email) VALUES ('B', 'b@x.com')",
])

client.close()
```

### Dùng với FastAPI

```python
from fastapi import FastAPI, HTTPException
import libsql_client, os
from dotenv import load_dotenv

load_dotenv()
app = FastAPI()
client = libsql_client.create_client_sync(
    url=os.getenv("TURSO_URL"), auth_token=os.getenv("TURSO_TOKEN")
)

@app.get("/users")
def list_users(limit: int = 20):
    rs = client.execute("SELECT id, ten, email FROM users LIMIT ?", [min(limit, 100)])
    cols = rs.columns
    return [dict(zip(cols, r)) for r in rs.rows]

@app.post("/users")
def create_user(ten: str, email: str, tuoi: int = None):
    try:
        rs = client.execute(
            "INSERT INTO users (ten, email, tuoi) VALUES (?, ?, ?) RETURNING id",
            [ten, email, tuoi],
        )
        return {"id": rs.rows[0][0]}
    except Exception as e:
        if "UNIQUE" in str(e):
            raise HTTPException(409, "Email đã tồn tại")
        raise
```

---

## 13. Dùng Với Drizzle ORM

```bash
npm install drizzle-orm @libsql/client
npm install -D drizzle-kit
```

```js
// db/schema.js
import { sqliteTable, integer, text } from "drizzle-orm/sqlite-core";

export const users = sqliteTable("users", {
  id: integer("id").primaryKey({ autoIncrement: true }),
  ten: text("ten").notNull(),
  email: text("email").notNull().unique(),
  tuoi: integer("tuoi"),
});
```

```js
// db/index.js
import { drizzle } from "drizzle-orm/libsql";
import { createClient } from "@libsql/client";
import * as schema from "./schema.js";

const client = createClient({
  url: process.env.TURSO_URL,
  authToken: process.env.TURSO_TOKEN,
});
export const db = drizzle(client, { schema });
```

```js
import { db } from "./db/index.js";
import { users } from "./db/schema.js";
import { eq, gt } from "drizzle-orm";

await db.insert(users).values({ ten: "An", email: "an@example.com", tuoi: 25 });
const list = await db.select().from(users).where(gt(users.tuoi, 20)).limit(10);
await db.update(users).set({ tuoi: 26 }).where(eq(users.email, "an@example.com"));
await db.delete(users).where(eq(users.id, 1));
```

```bash
# drizzle-kit với turso
# drizzle.config.js
export default {
  schema: "./db/schema.js",
  out: "./drizzle",
  dialect: "turso",
  dbCredentials: { url: process.env.TURSO_URL, authToken: process.env.TURSO_TOKEN },
};
npx drizzle-kit push
```

---

## 14. Dùng Với Prisma

Prisma hỗ trợ Turso qua driver `libsql`:

```bash
npm install prisma @prisma/client @prisma/adapter-libsql @libsql/client
```

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
  previewFeatures = ["driverAdapters"]
}

datasource db {
  provider = "sqlite"
  url      = env("TURSO_URL")
}

model User {
  id    Int     @id @default(autoincrement())
  ten   String
  email String  @unique
  tuoi  Int?
}
```

```js
// db.js
import { PrismaClient } from "@prisma/client";
import { PrismaLibSQL } from "@prisma/adapter-libsql";

const adapter = new PrismaLibSQL({
  url: process.env.TURSO_URL,
  authToken: process.env.TURSO_TOKEN,
});
export const prisma = new PrismaClient({ adapter });
```

> ⚠️ Prisma + Turso vẫn dùng SQLite dialect — một số tính năng Postgres-only của Prisma không áp dụng.

---

## 15. Auth Token — Xác Thực

### 15.1. Tạo và quản lý

```bash
# Token vĩnh viễn (cẩn thận khi dùng)
turso db tokens create mydb

# Token hết hạn sau 7 ngày (khuyên dùng cho dev)
turso db tokens create mydb --expiration 168h

# Token cho CI/CD: hết hạn sau 1h
turso db tokens create mydb --expiration 1h
```

### 15.2. Quy tắc an toàn

1. **Không commit token** — dùng `.env`
2. **Token ngắn hạn** cho CI/CD và môi trường tạm
3. **Thu hồi** khi nhân sự rời team: `turso db tokens invalidate mydb`
4. **Tách token** theo môi trường: dev / staging / production

```bash
# .env (KHÔNG commit!)
TURSO_URL=libsql://mydb-xxx.turso.io
TURSO_TOKEN=eyJhbGciOi...
```

```gitignore
.env
.env.local
*.db
*.sqlite
```

---

## 16. Branching Trên Turso

Turso cũng hỗ trợ database branching (tương tự Neon):

```bash
# Tạo branch từ database hiện tại
turso db create mydb-dev --from-db mydb

# Kiểm tra
turso db list
```

**Workflow:**

```
mydb (production)
  └── mydb-dev (test migration)
  └── mydb-pr-123 (test feature)
```

1. Branch để test migration nguy hiểm
2. OK → apply lên production
3. Lỗi → xóa branch, production nguyên vẹn

> 💡 Kết hợp với GitHub Actions: mỗi PR tự tạo branch DB riêng để test.

---

## 17. Pricing & Free Tier

### 17.1. Free tier (2026)

| Tài nguyên | Free |
|---|---|
| Databases | 500 |
| Storage | 9 GB total |
| Row reads | 500M / tháng |
| Row writes | 25M / tháng |
| Locations | Tất cả locations |
| Thẻ tín dụng | Không cần |

> 💡 Free tier của Turso thuộc loại hào phóng nhất trong các DB serverless.

### 17.2. Gói trả phí (Scaler)

| Gói | Giá | Thêm gì |
|---|---|---|
| Scaler | ~$29/tháng | Thêm storage, reads/writes, support ưu tiên |

### 17.3. Cách tính usage

- **Row read:** mỗi row đọc = 1 (query trả về 100 rows = 100 reads)
- **Row write:** mỗi row ghi = 1
- **Tip tiết kiệm:** dùng `LIMIT`, tránh `SELECT *` bừa bãi, dùng embedded replica cho read nặng

---

## 18. Troubleshooting

| Lỗi | Nguyên nhân | Cách fix |
|---|---|---|
| `401 Unauthorized` | Sai/hết hạn token | Tạo token mới: `turso db tokens create` |
| `404 Not Found` | Sai database URL | `turso db show <ten> --url` lấy lại URL |
| Kết nối chậm từ VN | Primary ở xa (US/EU) | Tạo lại DB với `--location sin` |
| `UNIQUE constraint failed` | Trùng unique field | Kiểm tra dữ liệu trước khi insert |
| `no such table` | Chưa migrate / sai DB | Kiểm tra `turso db shell` xem tables |
| CLI `not logged in` | Hết session | `turso auth login` lại |
| Write conflict trên replica | Ghi vào replica (chỉ primary được ghi) | Đảm bảo dùng primary URL cho write |
| `database is locked` (embedded) | Nhiều process ghi cùng file | Dùng WAL mode hoặc gom write vào 1 process |

### Debug nhanh

```bash
# Test kết nối
turso db shell mydb "SELECT 1"

# Xem info chi tiết
turso db inspect mydb

# Test từ app: in URL (che token!)
echo ${TURSO_URL%%:*}  # chỉ hiện phần trước token nếu có
```

---

## 19. Best Practices & Cheat Sheet

### 19.1. Best practices

1. **Primary ở `sin`** nếu user chính ở VN/SEA
2. **Read từ replica gần nhất**, write về primary
3. **Dùng `?` placeholders** — không bao giờ nối chuỗi vào SQL
4. **`LIMIT` mọi query list** — tránh đọc hàng triệu rows
5. **Index** cho cột hay WHERE/JOIN: `CREATE INDEX idx_users_email ON users(email)`
6. **Batch** nhiều câu lệnh 1 lần thay vì round-trip liên tục
7. **Token ngắn hạn** cho CI/CD, rotate định kỳ
8. **Branch trước migration** nguy hiểm

### 19.2. Cheat sheet CLI

```bash
turso auth login                    # đăng nhập
turso auth whoami                    # xem user
turso db list                        # liệt kê DB
turso db create mydb --location sin  # tạo DB ở Singapore
turso db show mydb                   # chi tiết
turso db show mydb --url             # lấy URL
turso db tokens create mydb          # tạo token
turso db shell mydb                  # mở SQL shell
turso db shell mydb "SELECT 1"       # chạy 1 lệnh
turso db destroy mydb                # xóa DB
turso db locations                   # xem locations
turso group list                     # liệt kê groups
turso group create g --location sin  # tạo group
```

### 19.3. Cheat sheet SQL (SQLite)

```sql
-- Tạo bảng
CREATE TABLE t (id INTEGER PRIMARY KEY AUTOINCREMENT, ten TEXT NOT NULL);

-- Index
CREATE INDEX idx_t_ten ON t(ten);
CREATE UNIQUE INDEX idx_t_email ON users(email);

-- Upsert
INSERT INTO t (email, ten) VALUES ('a@x.com', 'An')
ON CONFLICT(email) DO UPDATE SET ten=excluded.ten;

-- Phân trang
SELECT * FROM t ORDER BY id LIMIT 20 OFFSET 40;

-- Đếm
SELECT COUNT(*) FROM t WHERE tuoi > 25;

-- Ngày giờ
SELECT datetime('now');                    -- UTC hiện tại
SELECT datetime('now', '+7 hours');         -- giờ VN
```

### 19.4. Tài liệu tham khảo

- 📘 Docs: https://docs.turso.tech
- 📦 libSQL: https://github.com/tursodatabase/libsql
- 💬 Discord: https://discord.gg/turso
- 🗺️ Locations: `turso db locations`

---

*📅 Cập nhật: 2026-10-10 — Tổng 19 phần, từ setup đến edge multi-region.*

---

## 20. Sync Chi Tiết — Cách Embedded Replica Hoạt Động

### 20.1. Cơ chế sync

```
1. App ghi vào file SQLite local (WAL mode)
2. Background thread đẩy WAL frames lên Turso cloud
3. Cloud áp dụng vào primary
4. Pull: lấy frames mới từ cloud về local
```

- **Đọc:** luôn từ file local → latency ~0ms, offline vẫn đọc được
- **Ghi:** local trước (nhanh), sync sau (eventual consistency)
- **Conflict:** last-write-wins theo timestamp

### 20.2. Cấu hình sync nâng cao

```js
import { createClient } from "@libsql/client";

const db = createClient({
  url: "file:local.db",
  syncUrl: process.env.TURSO_URL,
  syncToken: process.env.TURSO_TOKEN,
  syncInterval: 30,          // sync mỗi 30s (mặc định 60)
});

// Sync thủ công sau thao tác quan trọng
await db.execute("INSERT INTO orders ...");
await db.sync();  // đẩy ngay, không đợi interval

// Kiểm tra trạng thái sync
const info = await db.sync();
console.log("Synced frames:", info);
```

### 20.3. Xử lý offline

```js
async function saveOrder(order) {
  // Luôn ghi local trước — offline vẫn OK
  await db.execute({
    sql: "INSERT INTO orders (data, synced) VALUES (?, 0)",
    args: [JSON.stringify(order)],
  });

  // Thử sync, thất bại thì để lần sau
  try {
    await db.sync();
    await db.execute("UPDATE orders SET synced = 1 WHERE synced = 0");
  } catch (e) {
    console.log("Offline — sẽ sync sau");
  }
}

// Khi online trở lại: sync hàng đợi
window.addEventListener("online", () => db.sync());
```

---

## 21. Migrate Lên Turso

### 21.1. Từ SQLite local

Dễ nhất — vì cùng engine:

```bash
# Cách 1: push file trực tiếp (DB nhỏ)
turso db create mydb --location sin
# Dùng tool sync: copy file lên qua libsql shell
sqlite3 local.db .dump | turso db shell mydb

# Cách 2: dùng script (DB lớn, kiểm soát tốt hơn)
```

```python
import sqlite3, libsql_client

local = sqlite3.connect("local.db")
remote = libsql_client.create_client_sync(url="libsql://...", auth_token="...")

# Copy từng bảng
for (table,) in local.execute(
    "SELECT name FROM sqlite_master WHERE type='table' AND name NOT LIKE 'sqlite_%'"
):
    cols = [r[1] for r in local.execute(f"PRAGMA table_info({table})")]
    remote.execute(f"DELETE FROM {table}")  # xóa cũ nếu cần
    for row in local.execute(f"SELECT * FROM {table}"):
        placeholders = ",".join("?" * len(row))
        remote.execute(f"INSERT INTO {table} VALUES ({placeholders})", list(row))
    print(f"Copied {table}")
```

### 21.2. Từ Postgres/MySQL

1. Export schema, convert kiểu dữ liệu thủ công (SERIAL → INTEGER PRIMARY KEY AUTOINCREMENT, TIMESTAMPTZ → TEXT, BOOLEAN → INTEGER)
2. Export data ra CSV: `COPY users TO 'users.csv' CSV HEADER` (Postgres)
3. Import vào Turso:

```bash
turso db shell mydb << 'EOF'
.mode csv
.import users.csv users
EOF
```

```python
# Hoặc bằng Python cho kiểm soát tốt hơn
import csv, libsql_client

remote = libsql_client.create_client_sync(url="...", auth_token="...")
with open("users.csv") as f:
    reader = csv.DictReader(f)
    batch = []
    for row in reader:
        batch.append((
            "INSERT INTO users (ten, email, tuoi) VALUES (?, ?, ?)",
            [row["ten"], row["email"], int(row["tuoi"] or 0)],
        ))
        if len(batch) >= 500:
            remote.batch(batch)
            batch = []
    if batch:
        remote.batch(batch)
```

---

## 22. Turso Với Các Framework

### 22.1. Next.js + Drizzle + Turso (stack khuyên dùng)

```bash
npx create-next-app@latest myapp
npm install drizzle-orm @libsql/client
npm install -D drizzle-kit
```

```js
// lib/db.js
import { drizzle } from "drizzle-orm/libsql";
import { createClient } from "@libsql/client";
import * as schema from "./schema";

const client = createClient({
  url: process.env.TURSO_URL,
  authToken: process.env.TURSO_TOKEN,
});
export const db = drizzle(client, { schema });
```

```js
// app/api/users/route.js
import { db } from "@/lib/db";
import { users } from "@/lib/schema";
import { NextResponse } from "next/server";

export async function GET() {
  const list = await db.select().from(users).limit(20);
  return NextResponse.json(list);
}
```

### 22.2. SvelteKit

```js
// src/lib/db.js
import { createClient } from "@libsql/client";
import { TURSO_URL, TURSO_TOKEN } from "$env/static/private";

export const db = createClient({ url: TURSO_URL, authToken: TURSO_TOKEN });
```

```js
// src/routes/api/users/+server.js
import { db } from "$lib/db";
import { json } from "@sveltejs/kit";

export async function GET() {
  const rs = await db.execute("SELECT * FROM users LIMIT 20");
  return json(rs.rows);
}
```

### 22.3. Nuxt

```js
// server/utils/db.js
import { createClient } from "@libsql/client";

export const db = createClient({
  url: process.env.TURSO_URL,
  authToken: process.env.TURSO_TOKEN,
});
```

```js
// server/api/users.get.js
import { db } from "../utils/db";

export default defineEventHandler(async () => {
  const rs = await db.execute("SELECT * FROM users LIMIT 20");
  return rs.rows;
});
```

---

## 23. Quan Sát & Giới Hạn Tài Nguyên

### 23.1. Xem usage

```bash
# Dashboard: https://app.turso.tech → chọn DB → Usage
# Xem qua CLI:
turso db inspect mydb
```

Theo dõi:
- **Row reads/writes:** so với quota free tier
- **Storage:** dung lượng DB
- **Locations:** số replica đang chạy

### 23.2. Giảm reads

```sql
-- Tồi: đọc toàn bộ bảng mỗi lần
SELECT * FROM products;

-- Tốt: chỉ cột cần + LIMIT
SELECT id, ten, gia FROM products LIMIT 20;

-- Tốt hơn: cache ở app cho data ít đổi
```

```js
// Cache đơn giản với TTL
const cache = new Map();
async function getProducts() {
  const hit = cache.get("products");
  if (hit && Date.now() - hit.ts < 60000) return hit.data;
  const rs = await db.execute("SELECT id, ten, gia FROM products LIMIT 50");
  cache.set("products", { data: rs.rows, ts: Date.now() });
  return rs.rows;
}
```

### 23.3. Khi nào nâng cấp?

- Vượt 500M reads/tháng liên tục
- Cần nhiều hơn 9GB storage
- Cần support ưu tiên / SLA

---

## 24. Câu Hỏi Thường Gặp (FAQ)

**Hỏi: Turso có mất data không khi free?**
Đáp: Không. Free tier không xóa data, chỉ giới hạn quota.

**Hỏi: Ghi vào replica được không?**
Đáp: Không — replica chỉ đọc. Write luôn về primary. Nếu cần ghi ở nhiều region, cân nhắc kiến trúc khác.

**Hỏi: Latency ghi từ VN về primary ở Singapore?**
Đáp: ~20-50ms — chấp nhận được cho hầu hết app. Chỉ thành vấn đề với game realtime/giao dịch tần suất cao.

**Hỏi: Có thể dùng Turso thay SQLite local trong dev không?**
Đáp: Có, nhưng dev local nên dùng file SQLite thuần cho nhanh, chỉ trỏ Turso ở staging/production.

**Hỏi: Backup thế nào?**
Đáp: Turso tự backup. Ngoài ra tự dump định kỳ: `turso db shell mydb ".dump" > backup.sql`.

**Hỏi: Khác gì D1 (Cloudflare)?**
Đáp: D1 cũng là SQLite edge nhưng khóa chặt vào Cloudflare Workers. Turso chạy mọi nơi (Vercel, Fly, VPS, local).

---

*Cập nhật: đã bổ sung sync chi tiết, migrate, framework integrations, FAQ — tổng 24 phần.*
