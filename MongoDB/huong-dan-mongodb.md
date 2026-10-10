# Hướng Dẫn Toàn Tập MongoDB — Từ Zero Đến Hero

> **Cấp độ:** Người mới bắt đầu → dùng được thật trong dự án
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10
>
> ⚠️ Bài này dùng MongoDB Atlas (free tier) và ví dụ local. Không cần thẻ tín dụng để bắt đầu.

---

## Mục Lục

1. [MongoDB Là Gì?](#1-mongodb-là-gì)
2. [SQL vs NoSQL — So Sánh Chi Tiết](#2-sql-vs-nosql--so-sánh-chi-tiết)
3. [Cài Đặt & Chuẩn Bị](#3-cài-đặt--chuẩn-bị)
4. [MongoDB Atlas — Cluster Miễn Phí Trên Cloud](#4-mongodb-atlas--cluster-miễn-phí-trên-cloud)
5. [MongoDB Compass — GUI Trực Quan](#5-mongodb-compass--gui-trực-quan)
6. [mongosh — Shell Dòng Lệnh](#6-mongosh--shell-dòng-lệnh)
7. [CRUD Cơ Bản](#7-crud-cơ-bản)
8. [Truy Vấn Nâng Cao — Filter, Sort, Projection](#8-truy-vấn-nâng-cao)
9. [Toán Tử Truy Vấn Đầy Đủ](#9-toán-tử-truy-vấn-đầy-đủ)
10. [Update Operators](#10-update-operators)
11. [Indexes — Tối Ưu Tốc Độ](#11-indexes--tối-ưu-tốc-độ)
12. [Aggregation Pipeline](#12-aggregation-pipeline)
13. [Data Modeling — Thiết Kế Schema](#13-data-modeling--thiết-kế-schema)
14. [Mongoose — MongoDB Với Node.js](#14-mongoose--mongodb-với-nodejs)
15. [PyMongo — MongoDB Với Python](#15-pymongo--mongodb-với-python)
16. [Transactions](#16-transactions)
17. [Atlas Search & Vector Search](#17-atlas-search--vector-search)
18. [Backup & Restore](#18-backup--restore)
19. [Pricing & Các Gói](#19-pricing--các-gói)
20. [Troubleshooting](#20-troubleshooting)
21. [Best Practices & Cheat Sheet](#21-best-practices--cheat-sheet)

---

## 1. MongoDB Là Gì?

### 1.1. Định nghĩa

**MongoDB** là hệ quản trị cơ sở dữ liệu **NoSQL** dạng **document-oriented** phổ biến nhất thế giới. Thay vì lưu dữ liệu trong bảng (table) với hàng và cột như MySQL/PostgreSQL, MongoDB lưu dữ liệu dưới dạng **document** theo định dạng **BSON** (Binary JSON).

**Ví dụ 1 document user:**

```json
{
  "_id": ObjectId("671a1b2c3d4e5f60718293a4"),
  "ten": "Nguyễn Văn An",
  "email": "an@example.com",
  "tuoi": 25,
  "dia_chi": {
    "thanh_pho": "Hà Nội",
    "quan": "Cầu Giấy",
    "duong": "Trần Duy Hưng 123"
  },
  "so_thich": ["đọc sách", "du lịch", "code"],
  "dang_hoat_dong": true,
  "ngay_tao": ISODate("2026-10-10T00:00:00Z")
}
```

Nhận thấy gì? Một document có thể chứa **object lồng nhau** (`dia_chi`) và **mảng** (`so_thich`) — điều mà SQL phải cần 3 bảng + JOIN mới làm được.

### 1.2. Thuật ngữ đối chiếu SQL ↔ MongoDB

| SQL (MySQL/Postgres) | MongoDB | Giải thích |
|---|---|---|
| Database | Database | Như nhau |
| Table | **Collection** | Tập hợp các document |
| Row | **Document** | 1 bản ghi dạng BSON/JSON |
| Column | **Field** | 1 trường trong document |
| Primary key | **_id** | Tự sinh ObjectId nếu không chỉ định |
| JOIN | **$lookup** / embedding | Nối dữ liệu |
| GROUP BY | **Aggregation pipeline** | Gom nhóm, tính toán |

### 1.3. Khi nào dùng MongoDB?

**Nên dùng khi:**
- Dữ liệu có cấu trúc linh hoạt, hay thay đổi (ví dụ: product với thuộc tính khác nhau mỗi loại)
- Cần scale ngang (horizontal scaling) dễ dàng
- Dữ liệu phân cấp, lồng nhau (comment trong post, order chứa items)
- Prototype nhanh, chưa chốt schema
- Real-time apps, IoT, content management, catalog

**Không nên dùng khi:**
- Cần JOIN phức tạp nhiều bảng (dữ liệu quan hệ chặt chẽ) → dùng PostgreSQL
- Cần transaction ACID mạnh trên nhiều document phức tạp (dù MongoDB đã hỗ trợ transaction)
- Dữ liệu dạng bảng chuẩn, ít thay đổi schema

### 1.4. Lịch sử ngắn

- **2009:** Ra mắt bởi 10gen (sau đổi tên thành MongoDB Inc.)
- **Tên gọi:** "Mongo" từ "humongous" (khổng lồ) — ám chỉ khả năng scale
- **Hiện tại:** Database NoSQL phổ biến nhất (DB-Engines ranking top 5 liên tục)

---

## 2. SQL vs NoSQL — So Sánh Chi Tiết

### 2.1. Mô hình dữ liệu

**SQL (quan hệ):**

```sql
-- 3 bảng: users, orders, order_items
SELECT u.ten, o.ngay_dat, i.ten_sp, i.so_luong
FROM users u
JOIN orders o ON o.user_id = u.id
JOIN order_items i ON i.order_id = o.id
WHERE u.id = 1;
```

**MongoDB (document):**

```js
// 1 collection orders, document đã chứa sẵn items
db.orders.findOne({ user_id: 1 })
// → trả về luôn cả mảng items, không cần JOIN
```

### 2.2. Bảng so sánh đầy đủ

| Tiêu chí | SQL (PostgreSQL) | MongoDB |
|---|---|---|
| Mô hình | Bảng, hàng, cột | Document BSON/JSON |
| Schema | Cố định, phải khai báo trước | Linh hoạt (schema-less) |
| JOIN | Mạnh, chuẩn | Có `$lookup` nhưng không phải thế mạnh |
| Transaction | ACID đầy đủ, lâu đời | Có multi-document ACID (từ 4.0) |
| Scale | Chủ yếu scale dọc (mạnh máy lên) | Scale ngang dễ (sharding) |
| Truy vấn | SQL chuẩn | MQL (MongoDB Query Language) / Aggregation |
| Phù hợp | Dữ liệu quan hệ chặt | Dữ liệu linh hoạt, phân cấp |

### 2.3. Schema linh hoạt nghĩa là gì?

Trong 1 collection, các document **không bắt buộc giống nhau**:

```js
// Document 1: user đầy đủ
{ ten: "An", email: "an@x.com", tuoi: 25 }

// Document 2: user thiếu tuổi, thêm field mới — vẫn OK!
{ ten: "Bình", email: "binh@x.com", sdt: "0901234567" }
```

SQL sẽ báo lỗi ngay nếu insert thiếu cột NOT NULL. MongoDB thì thoải mái — nhưng **đừng lạm dụng**: vẫn nên có schema validation (mục 13).

---

## 3. Cài Đặt & Chuẩn Bị

### 3.1. Cách 1: Dùng MongoDB Atlas (khuyên dùng)

Không cần cài gì cả — tạo cluster free trên cloud (mục 4). Chỉ cần:
- Tài khoản MongoDB Atlas (đăng ký email, không cần thẻ)
- Connection string để kết nối từ app

### 3.2. Cách 2: Cài MongoDB Community Server local

**Ubuntu/Debian:**

```bash
# Import GPG key
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Thêm repo
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] \
  https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Cài đặt
sudo apt update
sudo apt install -y mongodb-org

# Khởi động service
sudo systemctl start mongod
sudo systemctl enable mongod

# Kiểm tra
sudo systemctl status mongod
```

**macOS (Homebrew):**

```bash
brew tap mongodb/brew
brew install mongodb-community
brew services start mongodb-community
```

**Windows:**
- Tải installer `.msi` tại https://www.mongodb.com/try/download/community
- Chạy installer → chọn "Complete" → cài như service

**Docker (nhanh nhất để thử):**

```bash
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=matkhau123 \
  -v mongodb_data:/data/db \
  mongo:7.0

# Kiểm tra
docker ps
docker logs mongodb
```

### 3.3. Cài mongosh (shell mới)

```bash
# Ubuntu
sudo apt install -y mongodb-mongosh

# macOS
brew install mongosh

# Kiểm tra
mongosh --version
```

> 💡 **Lưu ý:** `mongo` (shell cũ) đã deprecated. Luôn dùng `mongosh`.

---

## 4. MongoDB Atlas — Cluster Miễn Phí Trên Cloud

### 4.1. Tạo tài khoản

1. Vào https://www.mongodb.com/cloud/atlas/register
2. Đăng ký bằng email / Google / GitHub
3. **Không cần thẻ tín dụng** cho gói M0 miễn phí

### 4.2. Tạo cluster miễn phí (M0)

1. Sau khi đăng nhập → **Create a deployment**
2. Chọn **M0 Free** (512MB storage — đủ để học và prototype)
3. Chọn **Cloud provider:** AWS / Google Cloud / Azure (chọn gần VN: AWS `ap-southeast-1` Singapore)
4. Đặt tên cluster (ví dụ: `Cluster0`)
5. Bấm **Create Deployment** → đợi 3-5 phút

### 4.3. Tạo database user

1. Vào **Database Access** (menu trái)
2. **Add New Database User**
3. Authentication: **Password** → đặt username/password mạnh
4. Database User Privileges: **Read and write to any database** (cho môi trường dev)
5. **Add User**

> ⚠️ **Bảo mật:** Không bao giờ dùng user `admin`/`password123`. Password nên dài, có ký tự đặc biệt.

### 4.4. Whitelist IP (Network Access)

1. Vào **Network Access** → **Add IP Address**
2. Tùy chọn:
   - **Allow Access from Anywhere** (`0.0.0.0/0`): tiện cho dev nhưng kém an toàn
   - **Add Current IP Address**: chỉ IP hiện tại của bạn (an toàn hơn)
3. **Confirm**

> 💡 Khi deploy app lên server (Vercel, Render...), IP sẽ đổi → lúc đó mới cần `0.0.0.0/0` hoặc whitelist IP của server.

### 4.5. Lấy connection string

1. Vào **Database** → bấm **Connect** trên cluster
2. Chọn **Drivers** → **Node.js** (hoặc Python)
3. Copy connection string:

```
mongodb+srv://<username>:<password>@cluster0.abc123.mongodb.net/?retryWrites=true&w=majority
```

4. Thay `<username>` và `<password>` bằng thông tin đã tạo. **Encode ký tự đặc biệt** trong password (ví dụ `@` → `%40`).

**Thêm tên database vào string:**

```
mongodb+srv://an:matkhau@cluster0.abc123.mongodb.net/shopdb?retryWrites=true&w=majority
```

### 4.6. Test kết nối bằng mongosh

```bash
mongosh "mongodb+srv://an:matkhau@cluster0.abc123.mongodb.net/shopdb"

# Nếu thành công:
# shopdb>
```

---

## 5. MongoDB Compass — GUI Trực Quan

### 5.1. Cài đặt

Tải tại https://www.mongodb.com/products/tools/compass — có bản cho Windows/macOS/Ubuntu.

### 5.2. Kết nối

1. Mở Compass → dán connection string vào ô
2. Bấm **Connect**
3. Thấy danh sách databases → collections → documents hiển thị dạng cây JSON

### 5.3. Những gì Compass làm được

| Chức năng | Mô tả |
|---|---|
| Xem/sửa document | Click vào field để edit trực tiếp |
| Query builder | Tạo filter bằng form, không cần gõ code |
| Aggregation builder | Kéo-thả các stage `$match`, `$group`... |
| Index management | Tạo/xóa index bằng UI |
| Schema analysis | Phân tích cấu trúc collection (kiểu dữ liệu, % null) |
| Explain plan | Xem query có dùng index không |
| Import/Export | Nhập từ JSON/CSV, xuất ra file |

> 💡 **Mẹo:** Khi viết aggregation phức tạp, dùng Compass builder để test trực quan, rồi bấm **Export to language** để lấy code Node.js/Python.

---

## 6. mongosh — Shell Dòng Lệnh

### 6.1. Kết nối

```bash
# Local (không auth)
mongosh

# Local (có auth)
mongosh -u admin -p matkhau123

# Atlas
mongosh "mongodb+srv://an:matkhau@cluster0.abc123.mongodb.net/shopdb"
```

### 6.2. Lệnh cơ bản

```js
// Xem database hiện tại
db

// Liệt kê databases
show dbs

// Chuyển database (tự tạo nếu chưa có khi insert)
use shopdb

// Liệt kê collections
show collections

// Xem thông tin server
db.version()
db.stats()
```

### 6.3. Thoát

```js
exit
// hoặc Ctrl+C hai lần
```

---

## 7. CRUD Cơ Bản

CRUD = **C**reate, **R**ead, **U**pdate, **D**elete. Đây là 4 thao tác nền tảng.

Chuẩn bị: chuyển vào database `shopdb`:

```js
use shopdb
```

### 7.1. Create — Thêm document

**Thêm 1 document:**

```js
db.products.insertOne({
  ten: "iPhone 15 Pro",
  gia: 28990000,
  hang: "Apple",
  ton_kho: 50,
  tags: ["smartphone", "flagship", "ios"]
})
```

Output:

```js
{
  acknowledged: true,
  insertedId: ObjectId('671a1b2c3d4e5f60718293a4')
}
```

**Thêm nhiều document:**

```js
db.products.insertMany([
  { ten: "Galaxy S24", gia: 22990000, hang: "Samsung", ton_kho: 30, tags: ["smartphone", "android"] },
  { ten: "Xiaomi 14", gia: 14990000, hang: "Xiaomi", ton_kho: 100, tags: ["smartphone", "android", "giá rẻ"] },
  { ten: "MacBook Air M3", gia: 27990000, hang: "Apple", ton_kho: 20, tags: ["laptop", "macos"] }
])
```

### 7.2. Read — Đọc document

**Đọc tất cả:**

```js
db.products.find()
```

**Đọc 1 document:**

```js
db.products.findOne({ hang: "Apple" })
```

**Đọc có điều kiện:**

```js
// Giá > 20 triệu
db.products.find({ gia: { $gt: 20000000 } })

// Hãng Apple HOẶC Samsung
db.products.find({ hang: { $in: ["Apple", "Samsung"] } })

// Tên chứa "iPhone" (regex, không phân biệt hoa thường)
db.products.find({ ten: { $regex: "iphone", $options: "i" } })
```

**Đếm:**

```js
db.products.countDocuments()                    // tổng số
db.products.countDocuments({ hang: "Apple" })    // theo điều kiện
db.products.estimatedDocumentCount()             // ước lượng nhanh (dùng metadata)
```

### 7.3. Update — Cập nhật

**Cập nhật 1 document:**

```js
// Giảm tồn kho iPhone xuống 45
db.products.updateOne(
  { ten: "iPhone 15 Pro" },       // filter: tìm document nào
  { $set: { ton_kho: 45 } }       // update: set field nào
)
```

**Cập nhật nhiều document:**

```js
// Giảm giá 10% tất cả sản phẩm Samsung (dùng $mul)
db.products.updateMany(
  { hang: "Samsung" },
  { $mul: { gia: 0.9 } }
)
```

**Upsert (update hoặc insert nếu chưa có):**

```js
db.products.updateOne(
  { ten: "Pixel 8" },
  { $set: { gia: 18990000, hang: "Google", ton_kho: 15 } },
  { upsert: true }   // không tìm thấy → tạo mới
)
```

### 7.4. Delete — Xóa

**Xóa 1 document:**

```js
db.products.deleteOne({ ten: "Xiaomi 14" })
```

**Xóa nhiều document:**

```js
// Xóa hết sản phẩm tồn kho = 0
db.products.deleteMany({ ton_kho: 0 })
```

> ⚠️ **Cẩn thận:** `db.products.deleteMany({})` với filter rỗng sẽ **xóa toàn bộ collection**! Luôn kiểm tra filter bằng `find()` trước khi delete.

### 7.5. Thay thế toàn bộ document

```js
// replaceOne: thay toàn bộ document (giữ _id)
db.products.replaceOne(
  { ten: "Galaxy S24" },
  { ten: "Galaxy S24 Ultra", gia: 27990000, hang: "Samsung", ton_kho: 25 }
)
```

Khác với `updateOne` + `$set` (chỉ sửa field chỉ định), `replaceOne` thay **toàn bộ** document.

---

## 8. Truy Vấn Nâng Cao

### 8.1. Projection — Chọn field trả về

```js
// Chỉ lấy tên và giá (luôn có _id mặc định)
db.products.find(
  { hang: "Apple" },
  { ten: 1, gia: 1 }
)
// → { _id: ..., ten: "iPhone 15 Pro", gia: 28990000 }

// Ẩn _id
db.products.find(
  { hang: "Apple" },
  { _id: 0, ten: 1, gia: 1 }
)
```

> ⚠️ Không thể vừa include vừa exclude trong 1 projection (trừ `_id`).

### 8.2. Sort — Sắp xếp

```js
// Giá tăng dần
db.products.find().sort({ gia: 1 })

// Giá giảm dần
db.products.find().sort({ gia: -1 })

// Sắp xếp nhiều field: hãng A-Z, rồi giá giảm dần
db.products.find().sort({ hang: 1, gia: -1 })
```

### 8.3. Limit & Skip — Phân trang

```js
// Trang 1: 10 sản phẩm đầu
db.products.find().limit(10)

// Trang 2: bỏ 10, lấy 10 tiếp
db.products.find().skip(10).limit(10)

// Trang N (page bắt đầu từ 1):
// skip = (page - 1) * limit
db.products.find().sort({ gia: -1 }).skip(20).limit(10)
```

### 8.4. Kết hợp đầy đủ

```js
db.products.find(
  { gia: { $gte: 10000000 }, ton_kho: { $gt: 0 } },  // filter
  { _id: 0, ten: 1, gia: 1, hang: 1 }                // projection
).sort({ gia: -1 }).limit(5)                          // sort + limit
```

---

## 9. Toán Tử Truy Vấn Đầy Đủ

### 9.1. So sánh

| Toán tử | Ý nghĩa | Ví dụ |
|---|---|---|
| `$eq` | Bằng | `{ gia: { $eq: 100 } }` |
| `$ne` | Khác | `{ hang: { $ne: "Apple" } }` |
| `$gt` | Lớn hơn | `{ gia: { $gt: 20000000 } }` |
| `$gte` | Lớn hơn hoặc bằng | `{ gia: { $gte: 20000000 } }` |
| `$lt` | Nhỏ hơn | `{ gia: { $lt: 15000000 } }` |
| `$lte` | Nhỏ hơn hoặc bằng | `{ gia: { $lte: 15000000 } }` |
| `$in` | Trong danh sách | `{ hang: { $in: ["Apple", "Samsung"] } }` |
| `$nin` | Không trong danh sách | `{ hang: { $nin: ["Apple"] } }` |

### 9.2. Logic

```js
// AND (mặc định khi liệt kê nhiều field)
db.products.find({ hang: "Apple", gia: { $lt: 30000000 } })

// $and tường minh
db.products.find({ $and: [{ hang: "Apple" }, { gia: { $lt: 30000000 } }] })

// OR
db.products.find({ $or: [{ hang: "Apple" }, { hang: "Samsung" }] })

// NOR (không thỏa mãn điều kiện nào)
db.products.find({ $nor: [{ hang: "Apple" }, { hang: "Samsung" }] })

// NOT
db.products.find({ gia: { $not: { $gt: 20000000 } } })
```

### 9.3. Element & Array

```js
// Field tồn tại
db.products.find({ mo_ta: { $exists: true } })

// Field có kiểu là string
db.products.find({ ten: { $type: "string" } })

// Mảng chứa phần tử
db.products.find({ tags: "android" })

// Mảng chứa TẤT CẢ các giá trị
db.products.find({ tags: { $all: ["smartphone", "android"] } })

// Mảng có đúng N phần tử
db.products.find({ tags: { $size: 2 } })

// Điều kiện trên phần tử mảng (object trong mảng)
db.products.find({ "danh_gia.diem": { $gte: 4 } })

// $elemMatch: 1 phần tử thỏa mãn NHIỀU điều kiện
db.orders.find({
  items: { $elemMatch: { ten_sp: "iPhone", so_luong: { $gte: 2 } } }
})
```

### 9.4. Regex

```js
// Tên chứa "pro" (không phân biệt hoa thường)
db.products.find({ ten: { $regex: "pro", $options: "i" } })

// Bắt đầu bằng "iPhone"
db.products.find({ ten: { $regex: "^iPhone" } })

// Kết thúc bằng "Pro"
db.products.find({ ten: { $regex: "Pro$" } })
```

> ⚠️ Regex không dùng index hiệu quả (trừ prefix `^...`). Với tìm kiếm text nghiêm túc, dùng **Atlas Search** (mục 17).

### 9.5. Truy vấn field lồng nhau (dot notation)

```js
// Document: { dia_chi: { thanh_pho: "Hà Nội", quan: "Cầu Giấy" } }
db.users.find({ "dia_chi.thanh_pho": "Hà Nội" })
db.users.find({ "dia_chi.quan": { $in: ["Cầu Giấy", "Đống Đa"] } })
```

---

## 10. Update Operators

### 10.1. Field operators

| Toán tử | Tác dụng | Ví dụ |
|---|---|---|
| `$set` | Gán giá trị | `{ $set: { gia: 100 } }` |
| `$unset` | Xóa field | `{ $unset: { mo_ta: "" } }` |
| `$inc` | Tăng/giảm số | `{ $inc: { ton_kho: -5 } }` |
| `$mul` | Nhân | `{ $mul: { gia: 0.9 } }` |
| `$rename` | Đổi tên field | `{ $rename: { ten: "ten_sp" } }` |
| `$min` / `$max` | Chỉ update nếu nhỏ/lớn hơn | `{ $max: { gia_cao_nhat: 500 } }` |
| `$currentDate` | Set thời gian hiện tại | `{ $currentDate: { cap_nhat: true } }` |

```js
// Ví dụ tổng hợp
db.products.updateOne(
  { ten: "iPhone 15 Pro" },
  {
    $set: { mo_ta: "Flagship mới nhất" },
    $inc: { luot_xem: 1 },
    $currentDate: { cap_nhat_luc: true }
  }
)
```

### 10.2. Array operators

```js
// $push: thêm phần tử vào mảng
db.products.updateOne(
  { ten: "iPhone 15 Pro" },
  { $push: { tags: "new" } }
)

// $push nhiều phần tử + sắp xếp + giới hạn (giữ 5 tags mới nhất)
db.products.updateOne(
  { ten: "iPhone 15 Pro" },
  { $push: { tags: { $each: ["hot", "sale"], $slice: -5 } } }
)

// $addToSet: thêm nếu CHƯA có (tránh trùng)
db.products.updateOne(
  { ten: "iPhone 15 Pro" },
  { $addToSet: { tags: "flagship" } }
)

// $pull: xóa phần tử khỏi mảng
db.products.updateOne(
  { ten: "iPhone 15 Pro" },
  { $pull: { tags: "sale" } }
)

// $pull theo điều kiện
db.orders.updateOne(
  { _id: ObjectId("...") },
  { $pull: { items: { so_luong: 0 } } }
)

// $pop: xóa đầu (-1) hoặc cuối (1) mảng
db.products.updateOne({ ten: "X" }, { $pop: { tags: 1 } })
```

### 10.3. Array filters — update phần tử cụ thể

```js
// Giảm giá 10% cho item "iPhone" trong mảng items của mọi order
db.orders.updateMany(
  { "items.ten_sp": "iPhone" },
  { $set: { "items.$[item].gia": 26000000 } },
  { arrayFilters: [{ "item.ten_sp": "iPhone" }] }
)
```

---

## 11. Indexes — Tối Ưu Tốc Độ

### 11.1. Index là gì?

Không có index, MongoDB phải **quét toàn bộ collection** (COLLSCAN) để tìm document — chậm khi dữ liệu lớn. Index giống mục lục sách: tìm nhanh hơn nhiều.

### 11.2. Tạo index

```js
// Index đơn trên field hang (tăng dần)
db.products.createIndex({ hang: 1 })

// Index giảm dần
db.products.createIndex({ gia: -1 })

// Compound index (nhiều field)
db.products.createIndex({ hang: 1, gia: -1 })

// Unique index (không trùng)
db.users.createIndex({ email: 1 }, { unique: true })

// Text index (tìm kiếm text)
db.products.createIndex({ ten: "text", mo_ta: "text" })

// TTL index (tự xóa sau N giây — hay dùng cho session/OTP)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })
```

### 11.3. Xem và xóa index

```js
// Liệt kê indexes
db.products.getIndexes()

// Xóa 1 index
db.products.dropIndex("hang_1")

// Xóa tất cả (trừ _id)
db.products.dropIndexes()
```

### 11.4. Kiểm tra query có dùng index không

```js
db.products.find({ hang: "Apple" }).explain("executionStats")
```

Xem output:
- `"stage": "IXSCAN"` → tốt, đang dùng index ✅
- `"stage": "COLLSCAN"` → xấu, đang quét toàn bộ ❌
- `totalDocsExamined` vs `nReturned`: chênh lệch lớn = index chưa hiệu quả

### 11.5. Quy tắc tạo index

1. **Index field hay dùng trong filter** (`find`, `update`, `delete`)
2. **Compound index:** field `equality` trước, `sort`/`range` sau (quy tắc ESR)
3. **Không lạm dụng:** mỗi index tốn disk + làm chậm write
4. **_id đã có index mặc định**, không cần tạo thêm

```js
// Query phổ biến:
db.orders.find({ user_id: 123, ngay_dat: { $gte: start } }).sort({ ngay_dat: -1 })

// Index tối ưu (ESR: Equality → Sort → Range):
db.orders.createIndex({ user_id: 1, ngay_dat: -1 })
```

---

## 12. Aggregation Pipeline

### 12.1. Pipeline là gì?

Aggregation pipeline xử lý dữ liệu qua **nhiều stage nối tiếp** — output của stage trước là input của stage sau. Mạnh hơn `find()` rất nhiều: group, join, tính toán, transform.

```
[Collection] → $match → $group → $sort → $limit → [Kết quả]
```

### 12.2. Các stage phổ biến

| Stage | Tác dụng |
|---|---|
| `$match` | Lọc document (như find) |
| `$group` | Gom nhóm + tính toán (như GROUP BY) |
| `$sort` | Sắp xếp |
| `$limit` / `$skip` | Phân trang |
| `$project` | Chọn/biến đổi field |
| `$lookup` | JOIN với collection khác |
| `$unwind` | Tách mảng thành nhiều document |
| `$addFields` | Thêm field tính toán |

### 12.3. Ví dụ 1: Doanh thu theo hãng

```js
db.orders.aggregate([
  // Stage 1: chỉ lấy order đã thanh toán
  { $match: { trang_thai: "da_thanh_toan" } },

  // Stage 2: tách mảng items thành từng document
  { $unwind: "$items" },

  // Stage 3: gom theo hãng, tính tổng
  { $group: {
      _id: "$items.hang",
      tong_doanh_thu: { $sum: { $multiply: ["$items.gia", "$items.so_luong"] } },
      so_don: { $sum: 1 }
  }},

  // Stage 4: sắp xếp giảm dần
  { $sort: { tong_doanh_thu: -1 } },

  // Stage 5: lấy top 5
  { $limit: 5 }
])
```

Output:

```js
[
  { _id: "Apple", tong_doanh_thu: 1500000000, so_don: 120 },
  { _id: "Samsung", tong_doanh_thu: 900000000, so_don: 95 },
  ...
]
```

### 12.4. Ví dụ 2: $lookup (JOIN)

```js
// Lấy orders kèm thông tin user
db.orders.aggregate([
  { $match: { trang_thai: "da_thanh_toan" } },
  { $lookup: {
      from: "users",           // collection join
      localField: "user_id",   // field của orders
      foreignField: "_id",     // field của users
      as: "thong_tin_user"     // tên field kết quả (mảng)
  }},
  { $unwind: "$thong_tin_user" },  // tách mảng thành object
  { $project: {
      _id: 0,
      ma_don: 1,
      ten_khach: "$thong_tin_user.ten",
      email: "$thong_tin_user.email",
      tong_tien: 1
  }}
])
```

### 12.5. Ví dụ 3: Thống kê theo thời gian

```js
// Doanh thu theo tháng
db.orders.aggregate([
  { $match: { trang_thai: "da_thanh_toan" } },
  { $group: {
      _id: {
        nam: { $year: "$ngay_dat" },
        thang: { $month: "$ngay_dat" }
      },
      doanh_thu: { $sum: "$tong_tien" },
      so_don: { $sum: 1 }
  }},
  { $sort: { "_id.nam": 1, "_id.thang": 1 } }
])
```

### 12.6. Toán tử hay dùng trong $group

| Toán tử | Tác dụng |
|---|---|
| `$sum: 1` | Đếm số document |
| `$sum: "$field"` | Tổng |
| `$avg: "$field"` | Trung bình |
| `$min` / `$max` | Nhỏ nhất / lớn nhất |
| `$push: "$field"` | Gom thành mảng |
| `$addToSet: "$field"` | Gom thành mảng (loại trùng) |
| `$first` / `$last` | Lấy đầu/cuối nhóm |

---

## 13. Data Modeling — Thiết Kế Schema

### 13.1. Embed vs Reference

**Embed (nhúng):** đặt dữ liệu liên quan vào cùng document.

```js
// Post nhúng comments — đọc 1 lần là có tất cả
{
  tieu_de: "Học MongoDB",
  noi_dung: "...",
  comments: [
    { user: "An", text: "Hay quá!", ngay: ISODate("...") },
    { user: "Bình", text: "Cảm ơn!", ngay: ISODate("...") }
  ]
}
```

**Reference (tham chiếu):** lưu `_id` trỏ sang collection khác.

```js
// Order tham chiếu user
{ ma_don: "DH001", user_id: ObjectId("..."), tong_tien: 500000 }
// → cần $lookup để lấy thông tin user
```

### 13.2. Quy tắc chọn

| Embed khi... | Reference khi... |
|---|---|
| Dữ liệu con thuộc về cha (comment của post) | Dữ liệu độc lập, dùng ở nhiều nơi |
| Đọc cùng nhau thường xuyên | Kích thước mảng có thể phình to vô hạn |
| Mảng nhỏ, ít thay đổi | Cần consistency mạnh |
| Muốn 1 query lấy hết | Hay update riêng lẻ |

> ⚠️ Giới hạn document: **16MB**. Mảng có thể phình vô hạn (ví dụ: lịch sử giao dịch) → dùng reference.

### 13.3. Schema Validation

Dù NoSQL linh hoạt, vẫn nên validate để tránh dữ liệu rác:

```js
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["ten", "email"],
      properties: {
        ten: { bsonType: "string", minLength: 2 },
        email: {
          bsonType: "string",
          pattern: "^.+@.+\\..+$"
        },
        tuoi: { bsonType: "int", minimum: 0, maximum: 150 }
      }
    }
  },
  validationLevel: "strict",
  validationAction: "error"
})

// Test: insert thiếu email → báo lỗi
db.users.insertOne({ ten: "An" })
// Document failed validation
```

---

## 14. Mongoose — MongoDB Với Node.js

### 14.1. Cài đặt

```bash
npm install mongoose
# hoặc
npm install mongoose dotenv
```

### 14.2. Kết nối

```js
// db.js
const mongoose = require('mongoose');

async function connectDB() {
  try {
    await mongoose.connect(process.env.MONGO_URI, {
      // options cho mongoose 7+ (hầu hết đã default)
    });
    console.log('✅ MongoDB connected');
  } catch (err) {
    console.error('❌ MongoDB connection error:', err.message);
    process.exit(1);
  }
}

module.exports = connectDB;
```

```env
# .env
MONGO_URI=mongodb+srv://an:matkhau@cluster0.abc123.mongodb.net/shopdb?retryWrites=true&w=majority
```

### 14.3. Schema & Model

```js
// models/Product.js
const mongoose = require('mongoose');

const productSchema = new mongoose.Schema(
  {
    ten: { type: String, required: [true, 'Tên là bắt buộc'], trim: true },
    gia: { type: Number, required: true, min: [0, 'Giá không âm'] },
    hang: { type: String, required: true, index: true },
    ton_kho: { type: Number, default: 0 },
    tags: [String],
    mo_ta: String,
    dang_ban: { type: Boolean, default: true },
  },
  {
    timestamps: true,  // tự thêm createdAt, updatedAt
    collection: 'products',
  }
);

// Index compound
productSchema.index({ hang: 1, gia: -1 });

module.exports = mongoose.model('Product', productSchema);
```

### 14.4. CRUD với Mongoose

```js
const Product = require('./models/Product');

// CREATE
const sp = await Product.create({
  ten: 'iPhone 15 Pro',
  gia: 28990000,
  hang: 'Apple',
  ton_kho: 50,
});

// READ
const all = await Product.find();                              // tất cả
const apple = await Product.find({ hang: 'Apple' });            // filter
const one = await Product.findOne({ ten: 'iPhone 15 Pro' });
const byId = await Product.findById('671a1b2c3d4e5f60718293a4');
const cheap = await Product.find({ gia: { $lt: 15000000 } })
  .select('ten gia')       // projection
  .sort({ gia: -1 })       // sort
  .limit(10);              // limit

// UPDATE
await Product.updateOne({ ten: 'iPhone 15 Pro' }, { $set: { ton_kho: 45 } });
const updated = await Product.findByIdAndUpdate(id, { gia: 27990000 }, { new: true });

// DELETE
await Product.deleteOne({ ten: 'Xiaomi 14' });
await Product.deleteMany({ ton_kho: 0 });
```

### 14.5. Validation & Middleware

```js
// Custom validator
productSchema.path('gia').validate(function (v) {
  return v % 1000 === 0;
}, 'Giá phải là bội số của 1000');

// Pre-save hook: tự tạo slug
productSchema.pre('save', function (next) {
  if (this.isModified('ten')) {
    this.slug = this.ten.toLowerCase().replace(/\s+/g, '-');
  }
  next();
});
```

### 14.6. Populate (thay cho JOIN)

```js
// models/Order.js
const orderSchema = new mongoose.Schema({
  user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  items: [{
    product: { type: mongoose.Schema.Types.ObjectId, ref: 'Product' },
    so_luong: Number,
    gia: Number,
  }],
  tong_tien: Number,
});

// Query kèm populate
const orders = await Order.find()
  .populate('user', 'ten email')              // chỉ lấy ten, email
  .populate('items.product', 'ten gia');     // populate nested
```

---

## 15. PyMongo — MongoDB Với Python

### 15.1. Cài đặt

```bash
pip install pymongo
# Có thể cần certifi cho Atlas trên một số môi trường
pip install certifi
```

### 15.2. Kết nối

```python
from pymongo import MongoClient
import os
from dotenv import load_dotenv

load_dotenv()

client = MongoClient(os.getenv("MONGO_URI"))
db = client["shopdb"]          # database
products = db["products"]      # collection

# Test kết nối
print(client.server_info()["version"])
```

### 15.3. CRUD

```python
from bson import ObjectId

# CREATE
result = products.insert_one({
    "ten": "iPhone 15 Pro",
    "gia": 28990000,
    "hang": "Apple",
    "ton_kho": 50,
})
print("Inserted:", result.inserted_id)

products.insert_many([
    {"ten": "Galaxy S24", "gia": 22990000, "hang": "Samsung"},
    {"ten": "Xiaomi 14", "gia": 14990000, "hang": "Xiaomi"},
])

# READ
for p in products.find({"hang": "Apple"}):
    print(p["ten"], p["gia"])

one = products.find_one({"ten": "iPhone 15 Pro"})
doc = products.find_one({"_id": ObjectId("671a1b2c3d4e5f60718293a4")})

# Projection + sort + limit
top = products.find(
    {"gia": {"$gte": 10000000}},
    {"_id": 0, "ten": 1, "gia": 1}
).sort("gia", -1).limit(5)

# UPDATE
products.update_one(
    {"ten": "iPhone 15 Pro"},
    {"$set": {"ton_kho": 45}, "$inc": {"luot_xem": 1}}
)

# DELETE
products.delete_one({"ten": "Xiaomi 14"})
```

### 15.4. Aggregation với PyMongo

```python
pipeline = [
    {"$match": {"trang_thai": "da_thanh_toan"}},
    {"$unwind": "$items"},
    {"$group": {
        "_id": "$items.hang",
        "doanh_thu": {"$sum": {"$multiply": ["$items.gia", "$items.so_luong"]}},
    }},
    {"$sort": {"doanh_thu": -1}},
]

for row in db.orders.aggregate(pipeline):
    print(row["_id"], row["doanh_thu"])
```

### 15.5. Dùng với FastAPI

```python
from fastapi import FastAPI, HTTPException
from pymongo import MongoClient
import os

app = FastAPI()
client = MongoClient(os.getenv("MONGO_URI"))
db = client["shopdb"]

@app.get("/products")
def list_products(hang: str = None, limit: int = 20):
    q = {"hang": hang} if hang else {}
    docs = list(db.products.find(q).limit(limit))
    # ObjectId không serialize JSON được → chuyển thành str
    for d in docs:
        d["_id"] = str(d["_id"])
    return docs

@app.get("/products/{product_id}")
def get_product(product_id: str):
    from bson import ObjectId
    doc = db.products.find_one({"_id": ObjectId(product_id)})
    if not doc:
        raise HTTPException(404, "Không tìm thấy sản phẩm")
    doc["_id"] = str(doc["_id"])
    return doc
```

---

## 16. Transactions

Từ MongoDB 4.0, đã hỗ trợ multi-document ACID transactions (cần replica set — Atlas mặc định đã có).

### 16.1. Ví dụ: chuyển tiền

```js
const session = await mongoose.startSession();
session.startTransaction();
try {
  await Account.updateOne(
    { _id: accA }, { $inc: { so_du: -100000 } }, { session }
  );
  await Account.updateOne(
    { _id: accB }, { $inc: { so_du: 100000 } }, { session }
  );
  await session.commitTransaction();
  console.log('✅ Chuyển tiền thành công');
} catch (err) {
  await session.abortTransaction();
  console.error('❌ Rollback:', err.message);
} finally {
  session.endSession();
}
```

### 16.2. Với mongosh

```js
const session = db.getMongo().startSession();
session.startTransaction();
try {
  const accounts = session.getDatabase("bank").accounts;
  accounts.updateOne({ _id: "A" }, { $inc: { so_du: -100 } });
  accounts.updateOne({ _id: "B" }, { $inc: { so_du: 100 } });
  session.commitTransaction();
} catch (e) {
  session.abortTransaction();
  throw e;
} finally {
  session.endSession();
}
```

---

## 17. Atlas Search & Vector Search

### 17.1. Atlas Search (full-text search)

Mạnh hơn regex rất nhiều: hỗ trợ tiếng Việt (analyzer), fuzzy search, relevance scoring.

**Bước 1:** Atlas → cluster → **Search** → Create Search Index (chọn collection).

**Bước 2:** Query:

```js
db.products.aggregate([
  { $search: {
      index: "default",
      text: { query: "iphone pro", path: "ten", fuzzy: { maxEdits: 1 } }
  }},
  { $limit: 10 },
  { $project: { _id: 0, ten: 1, gia: 1, score: { $meta: "searchScore" } } }
])
```

### 17.2. Vector Search (cho AI/RAG)

```js
// Tìm sản phẩm tương tự bằng embedding vector
db.products.aggregate([
  { $vectorSearch: {
      index: "vector_index",
      path: "embedding",
      queryVector: [0.12, -0.45, ...],  // embedding của câu query
      numCandidates: 100,
      limit: 5
  }},
  { $project: { _id: 0, ten: 1, score: { $meta: "vectorSearchScore" } } }
])
```

> 💡 Kết hợp với OpenAI/Cohere embedding để làm chatbot RAG, recommendation engine.

---

## 18. Backup & Restore

### 18.1. mongodump / mongorestore (local)

```bash
# Backup toàn bộ
mongodump --uri="mongodb://admin:matkhau@localhost:27017" --out=./backup

# Backup 1 database
mongodump --uri="..." --db=shopdb --out=./backup

# Restore
mongorestore --uri="mongodb://admin:matkhau@localhost:27017" ./backup
```

### 18.2. mongoexport / mongoimport (JSON/CSV)

```bash
# Export collection ra JSON
mongoexport --uri="..." --db=shopdb --collection=products --out=products.json

# Export ra CSV (chỉ định field)
mongoexport --uri="..." --db=shopdb --collection=products \
  --type=csv --fields=ten,gia,hang --out=products.csv

# Import
mongoimport --uri="..." --db=shopdb --collection=products --file=products.json
```

### 18.3. Atlas backup

Atlas M0 free: backup thủ công qua **... → Download** (mongodump). Các gói trả phí (M10+) có **Cloud Backup** tự động theo lịch + point-in-time restore.

---

## 19. Pricing & Các Gói

### 19.1. Atlas pricing (tóm tắt 2026)

| Gói | Giá | Storage | RAM | Phù hợp |
|---|---|---|---|---|
| **M0** | **Miễn phí** | 512 MB | Shared | Học, prototype |
| M2/M5 | ~$9/tháng | 2-5 GB | Shared | Side project nhỏ |
| M10 | ~$57/tháng | 10 GB | 2 GB dedicated | Production nhỏ |
| M20+ | từ ~$117/tháng | 20 GB+ | 4 GB+ | Production thật |

> 💡 M0 đủ cho học và demo. Khi cần production, M10 là điểm bắt đầu hợp lý.

### 19.2. Self-hosted (miễn phí)

- **Community Server:** miễn phí vĩnh viễn, tự quản lý
- Chi phí = tiền VPS (~$5-10/tháng cho VPS 2GB RAM)
- Phải tự lo backup, update, monitoring

### 19.3. Khi nào nâng cấp khỏi M0?

- Dữ liệu > 400MB (để dư buffer)
- Cần backup tự động
- Cần nhiều hơn 500 connections đồng thời
- Cần Atlas Search / VPC peering (một số tính năng chỉ có từ M10)

---

## 20. Troubleshooting

| Lỗi | Nguyên nhân | Cách fix |
|---|---|---|
| `MongoServerSelectionError` / timeout | Sai connection string, IP chưa whitelist, cluster đang sleep | Kiểm tra Network Access, user/pass, cluster status |
| `bad auth: authentication failed` | Sai username/password | Kiểm tra Database Access; encode ký tự đặc biệt trong password (`@`→`%40`) |
| `not authorized on db` | User thiếu quyền | Cấp quyền readWrite trong Database Access |
| Query chậm | Thiếu index | Dùng `.explain()`, tạo index cho field filter |
| `E11000 duplicate key error` | Trùng unique index | Kiểm tra dữ liệu trùng trước khi insert |
| `Document failed validation` | Vi phạm schema validation | Kiểm tra required field, kiểu dữ liệu |
| Mongoose `buffering timed out` | Chưa connect DB đã query | Đảm bảo `await mongoose.connect()` trước khi query |
| Atlas M0 "paused" | Không hoạt động 7 ngày | Bấm Resume trong Atlas UI |
| `Transaction numbers are only allowed on a replica set` | Local standalone không hỗ trợ transaction | Chạy `mongod --replSet` hoặc dùng Atlas |

### Chi tiết lỗi kết nối Atlas

```bash
# Test DNS (Atlas dùng SRV record)
nslookup -type=SRV _mongodb._tcp.cluster0.abc123.mongodb.net

# Test kết nối TCP
mongosh "mongodb+srv://..." --eval "db.runCommand({ping:1})"
```

---

## 21. Best Practices & Cheat Sheet

### 21.1. Best practices

1. **Luôn dùng biến môi trường** cho connection string — không hardcode
2. **Tạo index** cho field hay filter/sort
3. **Projection** chỉ lấy field cần — giảm băng thông
4. **Không lưu file lớn trong document** — dùng GridFS hoặc S3, lưu URL
5. **Dùng `ObjectId`** làm `_id` mặc định (có thứ tự thời gian, tốt cho sort)
6. **Validate input** ở application layer + schema validation ở DB
7. **Pagination:** dùng `_id`-based (cursor) thay vì `skip()` với dữ liệu lớn
8. **Connection pooling:** Mongoose/PyMongo đã tự quản lý pool, không tạo connection mới mỗi request

### 21.2. Cheat sheet mongosh

```js
// Database
show dbs                    // liệt kê DB
use mydb                    // chuyển DB
db.dropDatabase()           // xóa DB hiện tại

// Collection
show collections
db.createCollection("users")
db.users.drop()

// Insert
db.c.insertOne({...})
db.c.insertMany([{...}, {...}])

// Find
db.c.find({})                          // tất cả
db.c.find({a: 1}, {b: 1, _id: 0})      // filter + projection
db.c.findOne({a: 1})
db.c.find().sort({a: -1}).limit(10).skip(20)
db.c.countDocuments({a: 1})

// Update
db.c.updateOne({a: 1}, { $set: {b: 2} })
db.c.updateMany({}, { $inc: {count: 1} })

// Delete
db.c.deleteOne({a: 1})
db.c.deleteMany({a: 1})

// Index
db.c.createIndex({a: 1})
db.c.getIndexes()
db.c.dropIndex("a_1")

// Aggregation
db.c.aggregate([{ $match: {} }, { $group: { _id: "$a", n: { $sum: 1 } } }])
```

### 21.3. Tài liệu tham khảo

- 📘 Docs chính thức: https://www.mongodb.com/docs/
- 🎓 MongoDB University (khóa học miễn phí): https://university.mongodb.com/
- 🧭 MQL cheat sheet: https://www.mongodb.com/docs/manual/reference/
- 📊 DB-Engines ranking: https://db-engines.com/en/system/MongoDB

---

*📅 Cập nhật: 2026-10-10 — Tổng 21 phần, từ cơ bản đến production.*
