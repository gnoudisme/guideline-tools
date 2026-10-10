# Hướng Dẫn Toàn Tập Supabase — Backend Hoàn Chỉnh Không Cần Server

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10

---

## Mục Lục

1. [Supabase Là Gì?](#1-supabase-là-gì)
2. [Tạo Tài Khoản & Project](#2-tạo-tài-khoản--project)
3. [Tổng Quan Dashboard](#3-tổng-quan-dashboard)
4. [Database — PostgreSQL](#4-database--postgresql)
5. [Table Editor](#5-table-editor)
6. [SQL Editor](#6-sql-editor)
7. [Authentication](#7-authentication)
8. [Row Level Security (RLS)](#8-row-level-security-rls)
9. [Storage — Lưu File](#9-storage--lưu-file)
10. [Realtime — Dữ Liệu Thời Gian Thực](#10-realtime--dữ-liệu-thời-gian-thực)
11. [Edge Functions](#11-edge-functions)
12. [JavaScript Client — supabase-js](#12-javascript-client--supabase-js)
13. [Dự Án Thực Tế: App Ghi Chú Full-Stack](#13-dự-án-thực-tế-app-ghi-chú-full-stack)
14. [Pricing Chi Tiết](#14-pricing-chi-tiết)
15. [So Sánh Với Firebase & Đối Thủ](#15-so-sánh-với-firebase--đối-thủ)
16. [Troubleshooting](#16-troubleshooting)
17. [Best Practices](#17-best-practices)
18. [FAQ](#18-faq)

---

## 1. Supabase Là Gì?

### 1.1. Định Nghĩa

**Supabase** là nền tảng **Backend-as-a-Service (BaaS)** mã nguồn mở — được mệnh danh là **"Firebase alternative open-source"**. Bạn có ngay:

```
Đăng ký Supabase → Có ngay:
  ✅ PostgreSQL database (mạnh nhất thế giới open-source)
  ✅ Authentication (đăng nhập email, Google, GitHub...)
  ✅ Storage (lưu ảnh, file như S3)
  ✅ Realtime (WebSocket tự động)
  ✅ Edge Functions (chạy code serverless)
  ✅ Auto-generated API (REST + GraphQL tự sinh từ DB!)
```

**Không cần viết backend.** Frontend gọi trực tiếp vào Supabase!

### 1.2. Kiến Trúc

```
┌──────────────┐         ┌─────────────────────────────────┐
│              │         │           SUPABASE                │
│  Frontend    │────────▶│  ┌─────────┐  ┌──────────────┐   │
│  (React/Vue/ │  HTTPS  │  │ PostgREST│  │ Auth         │   │
│   Flutter)   │         │  │ (auto   │  │ (GoTrue)     │   │
│              │         │  │  API)   │  └──────────────┘   │
└──────────────┘         │  └─────────┘  ┌──────────────┐   │
                         │               │ Storage      │   │
                         │  ┌─────────┐  └──────────────┘   │
                         │  │Realtime │  ┌──────────────┐   │
                         │  │         │  │ Edge Funcs   │   │
                         │  └─────────┘  └──────────────┘   │
                         │  ┌─────────────────────────┐   │
                         │  │  PostgreSQL 16          │   │
                         │  └─────────────────────────┘   │
                         └─────────────────────────────────┘
```

### 1.3. So Sánh Với Firebase

| Tiêu chí | Supabase | Firebase |
|----------|----------|----------|
| Database | **PostgreSQL** (SQL quan hệ) | Firestore (NoSQL) |
| Mã nguồn mở | ✅ Có (tự host được) | ❌ Đóng |
| Query phức tạp | ✅ SQL mạnh (JOIN, aggregate...) | ⚠️ Hạn chế |
| Auth | ✅ Email, OAuth, phone | ✅ Tương tự |
| Realtime | ✅ Có | ✅ Có |
| Pricing | Rẻ hơn, PostgreSQL thật | Đắt khi scale |
| Vendor lock-in | Thấp (Postgres chuẩn) | Cao |
| Học SQL | Cần biết SQL cơ bản | Không cần |

**Chọn Supabase khi:** Cần quan hệ dữ liệu phức tạp, thích SQL, muốn open-source, muốn rẻ.
**Chọn Firebase khi:** Đã quen hệ sinh thái Google, cần NoSQL đơn giản, không muốn học SQL.

---

## 2. Tạo Tài Khoản & Project

### 2.1. Đăng Ký

1. Vào https://supabase.com → **Start your project**
2. Đăng ký bằng **GitHub** (khuyên dùng)
3. Vào Dashboard

### 2.2. Tạo Project

1. **New Project**
2. Điền:

| Trường | Gợi ý |
|--------|-------|
| **Name** | `my-app` |
| **Database Password** | Mật khẩu mạnh (lưu lại! dùng để kết nối trực tiếp) |
| **Region** | `Southeast Asia (Singapore)` — gần VN nhất! |
| **Plan** | Free |

3. **Create new project** → đợi 2-3 phút khởi tạo

### 2.3. Lấy API Keys

Project → **Settings** (⚙️) → **API**:

| Key | Dùng ở đâu | Bảo mật |
|-----|-----------|---------|
| **Project URL** | `https://xyz.supabase.co` — public được | ✅ Public |
| **anon key** | Dùng ở frontend — public được | ✅ Public (bị giới hạn bởi RLS!) |
| **service_role key** | Chỉ dùng ở backend — **TUYỆT ĐỐI KHÔNG** để lộ ra frontend! | 🔒 Secret |

> ⚠️ **service_role key bypass mọi RLS** — ai có key này là toàn quyền database! Chỉ dùng trong server/edge function.

---

## 3. Tổng Quan Dashboard

```
┌──────────┬──────────────────────────────────────┐
│ 🏠 Home  │  Project: my-app                     │
│          │                                      │
│ 🗄️ Table │  Quick start, project health         │
│   Editor │                                      │
│          │                                      │
│ 📝 SQL   │                                      │
│   Editor │                                      │
│          │                                      │
│ 🔐 Auth  │                                      │
│          │                                      │
│ 🗂️ Storage                                      │
│          │                                      │
│ ⚡ Edge  │                                      │
│   Functions                                     │
│          │                                      │
│ ⚙️ Settings                                     │
└──────────┴──────────────────────────────────────┘
```

| Menu | Chức năng |
|------|-----------|
| **Table Editor** | Xem/sửa bảng như Excel |
| **SQL Editor** | Chạy query SQL trực tiếp |
| **Authentication** | Quản lý users, providers |
| **Storage** | Quản lý buckets, files |
| **Edge Functions** | Deploy serverless functions |
| **Settings** | API keys, database settings |

---

## 4. Database — PostgreSQL

### 4.1. Tại Sao PostgreSQL?

PostgreSQL là database open-source mạnh nhất:
- Hỗ trợ JSON, full-text search, GIS...
- ACID đầy đủ, đáng tin cậy
- Cộng đồng lớn, tài liệu nhiều

Supabase cho bạn **PostgreSQL 16 thật**, không phải bản "giả lập".

### 4.2. Kết Nối Trực Tiếp (Ngoài Supabase Client)

**Settings** → **Database** → **Connection string**:

```
# Dùng với psql, DBeaver, TablePlus...
postgresql://postgres:[PASSWORD]@db.xyz.supabase.co:5432/postgres
```

**Connection pooling** (cho app nhiều connections):
```
postgresql://postgres.xyz:[PASSWORD]@aws-0-ap-southeast-1.pooler.supabase.com:6543/postgres
```
Dùng port **6543** (pooler) thay vì 5432 cho app production!

### 4.3. Extensions Có Sẵn

Supabase bật sẵn nhiều extension hữu ích:

| Extension | Chức năng |
|-----------|-----------|
| `uuid-ossp` | Sinh UUID |
| `pgcrypto` | Mã hóa, hash password |
| `pg_trgm` | Tìm kiếm gần đúng (fuzzy search) |
| `postgis` | Dữ liệu địa lý (bản đồ) |
| `pg_cron` | Chạy job theo lịch trong DB |

Bật thêm: **Database** → **Extensions** → bật cái cần.

---

## 5. Table Editor

### 5.1. Tạo Bảng Bằng UI

**Table Editor** → **+ New Table**:

| Trường | Điền |
|--------|------|
| **Name** | `tasks` |
| **Description** | Bảng công việc |
| **Enable RLS** | ✅ Bật (xem mục 8) |

Thêm cột:

| Tên cột | Kiểu | Mặc định | Ghi chú |
|---------|------|----------|---------|
| `id` | uuid | `gen_random_uuid()` | Primary key |
| `created_at` | timestamptz | `now()` | Tự động |
| `title` | text | — | NOT NULL |
| `is_done` | bool | `false` | |
| `user_id` | uuid | — | Foreign key → auth.users |

Bấm **Save** → bảng được tạo!

### 5.2. Các Kiểu Dữ Liệu Thường Dùng

| Kiểu | Dùng cho | Ví dụ |
|------|----------|-------|
| `text` | Chuỗi dài | Tên, mô tả |
| `varchar(n)` | Chuỗi giới hạn | Mã (10 ký tự) |
| `int4` / `int8` | Số nguyên | Tuổi, số lượng |
| `numeric` | Số thập phân chính xác | Tiền tệ |
| `bool` | Đúng/sai | is_active |
| `timestamptz` | Thời gian có timezone | created_at |
| `date` | Ngày | birth_date |
| `uuid` | ID duy nhất | id |
| `jsonb` | JSON linh hoạt | metadata, settings |
| `text[]` | Mảng text | tags |

### 5.3. Quan Hệ (Foreign Keys)

**Ví dụ:** Bảng `tasks` thuộc về `user`:

Trong Table Editor → cột `user_id` → **Add foreign key relation**:
- **Reference table:** `auth.users`
- **Reference column:** `id`
- **Action on delete:** `CASCADE` (xóa user → xóa luôn tasks của họ)

### 5.4. Thao Tác Dữ Liệu

- **Xem:** Bấm vào bảng → dữ liệu hiện như Excel
- **Thêm:** **Insert** → **Insert row** → điền → Save
- **Sửa:** Click đúp vào ô → sửa → Enter
- **Xóa:** Chọn dòng → Delete
- **Lọc:** Bấm **Filter** → chọn cột, điều kiện
- **Sắp xếp:** Bấm vào tên cột

---

## 6. SQL Editor

### 6.1. Chạy Query

**SQL Editor** → **+ New Query**:

```sql
-- Tạo bảng tasks
create table tasks (
  id uuid default gen_random_uuid() primary key,
  created_at timestamptz default now(),
  title text not null,
  is_done boolean default false,
  user_id uuid references auth.users(id) on delete cascade
);

-- Thêm RLS (xem mục 8)
alter table tasks enable row level security;

-- Policy: user chỉ thấy task của mình
create policy "Users can manage own tasks"
  on tasks for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);
```

Bấm **Run** (hoặc `Ctrl+Enter`).

### 6.2. Query Thường Dùng

```sql
-- Xem 10 tasks mới nhất
select * from tasks
order by created_at desc
limit 10;

-- Đếm task theo trạng thái
select is_done, count(*)
from tasks
group by is_done;

-- Tìm kiếm với full-text (tiếng Việt cần config thêm)
select * from tasks
where title ilike '%họp%';

-- Join với bảng users
select t.title, u.email
from tasks t
join auth.users u on t.user_id = u.id
where t.is_done = false;
```

### 6.3. Snippets Có Sẵn

SQL Editor có sẵn snippets: bấm **+ New Query** → chọn từ **Templates**:
- Create table
- Enable RLS
- Create policy...

---

## 7. Authentication

### 7.1. Bật Auth Providers

**Authentication** → **Providers**:

| Provider | Cách bật |
|----------|----------|
| **Email** | Mặc định bật — đăng ký/đăng nhập bằng email+password |
| **Google** | Nhập Client ID + Secret (tạo ở Google Cloud Console) |
| **GitHub** | Nhập Client ID + Secret (GitHub → Settings → OAuth Apps) |
| **Facebook** | Tương tự |
| **Phone (SMS)** | Cần cấu hình Twilio/MessageBird (tốn phí) |
| **Anonymous** | Cho phép user ẩn danh (demo) |

### 7.2. Đăng Ký / Đăng Nhập Bằng JS

```javascript
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  'https://xyz.supabase.co',
  'eyJhbGciOi...anon-key...'
);

// Đăng ký
const { data, error } = await supabase.auth.signUp({
  email: 'an@example.com',
  password: 'mat-khau-manh-123',
});
if (error) console.error('Lỗi đăng ký:', error.message);
else console.log('Đăng ký xong! Kiểm tra email để xác nhận.');

// Đăng nhập
const { data: loginData, error: loginError } =
  await supabase.auth.signInWithPassword({
    email: 'an@example.com',
    password: 'mat-khau-manh-123',
  });

// Đăng nhập bằng Google
await supabase.auth.signInWithOAuth({
  provider: 'google',
  options: { redirectTo: 'https://myapp.com/welcome' }
});

// Đăng xuất
await supabase.auth.signOut();

// Lấy user hiện tại
const { data: { user } } = await supabase.auth.getUser();
console.log('User:', user?.email);
```

### 7.3. Theo Dõi Trạng Thái Đăng Nhập

```javascript
// Tự động cập nhật khi login/logout
supabase.auth.onAuthStateChange((event, session) => {
  console.log('Event:', event); // SIGNED_IN, SIGNED_OUT, TOKEN_REFRESHED
  if (session) {
    console.log('Đã đăng nhập:', session.user.email);
    // Chuyển hướng hoặc cập nhật UI
  } else {
    console.log('Đã đăng xuất');
  }
});
```

### 7.4. Quản Lý Users

**Authentication** → **Users**:
- Xem danh sách users
- Xem chi tiết: email, lần đăng nhập cuối, metadata
- **Xóa user**, **ban user**
- **Thêm user tay** (cho testing)

### 7.5. Email Templates

**Authentication** → **Email Templates**: tùy chỉnh email xác nhận, reset password:
- **Confirm signup:** email khi đăng ký
- **Reset password:** email quên mật khẩu
- **Magic link:** đăng nhập không cần password

Có thể dùng biến `{{ .ConfirmationURL }}`, `{{ .Email }}` trong template.

### 7.6. Đặt Lại Mật Khẩu

```javascript
// Gửi email reset
await supabase.auth.resetPasswordForEmail('an@example.com', {
  redirectTo: 'https://myapp.com/reset-password',
});

// Trang reset-password: đặt mật khẩu mới
await supabase.auth.updateUser({ password: 'mat-khau-moi-456' });
```

---

## 8. Row Level Security (RLS)

### 8.1. RLS Là Gì? Tại Sao Quan Trọng?

**Row Level Security** là "tường lửa" ở tầng database: quy định **ai được thấy/sửa dòng nào**.

```
KHÔNG có RLS:
  User A gọi API → thấy được task của User B! 🔴 NGUY HIỂM

CÓ RLS:
  User A gọi API → chỉ thấy task của chính mình ✅
  (dù dùng anon key public ở frontend!)
```

> ⚠️ **Đây là tính năng QUAN TRỌNG NHẤT của Supabase.** Bật RLS + viết policies đúng = frontend gọi trực tiếp DB mà vẫn an toàn!

### 8.2. Cách Hoạt Động

Mỗi khi có query, PostgreSQL kiểm tra **policies** của bảng:

```sql
-- Policy: "user chỉ được đọc task của mình"
create policy "Users can read own tasks"
  on tasks
  for select                    -- áp dụng cho lệnh SELECT
  using (auth.uid() = user_id); -- điều kiện: id user đang login = user_id của dòng
```

`auth.uid()` = ID của user đang đăng nhập (lấy từ JWT token).

### 8.3. Các Loại Policy

| Loại | Áp dụng cho | Ví dụ |
|------|-------------|-------|
| `for select` | Đọc | User chỉ đọc task của mình |
| `for insert` | Thêm | User chỉ thêm task cho chính mình |
| `for update` | Sửa | User chỉ sửa task của mình |
| `for delete` | Xóa | Chỉ admin được xóa |
| `for all` | Tất cả | Gộp 4 loại trên |

### 8.4. Policies Mẫu Cho Mọi Tình Huống

**1. Private: user chỉ thao tác dữ liệu của mình (phổ biến nhất):**
```sql
-- Bật RLS
alter table tasks enable row level security;

-- 1 policy cho mọi thao tác
create policy "Users manage own tasks"
  on tasks for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);
```
> `using` kiểm tra khi đọc/sửa/xóa, `with check` kiểm tra khi thêm/sửa.

**2. Public read, private write (blog, sản phẩm):**
```sql
-- Ai cũng đọc được
create policy "Public read"
  on posts for select
  using (true);

-- Chỉ chủ sở hữu được viết
create policy "Owner can write"
  on posts for insert
  with check (auth.uid() = user_id);

create policy "Owner can update"
  on posts for update
  using (auth.uid() = user_id);

create policy "Owner can delete"
  on posts for delete
  using (auth.uid() = user_id);
```

**3. Chỉ admin:**
```sql
-- Giả sử có bảng user_roles(user_id, role)
create policy "Admin only"
  on secret_data for all
  using (
    exists (
      select 1 from user_roles
      where user_id = auth.uid()
      and role = 'admin'
    )
  );
```

**4. Theo team/organization:**
```sql
-- User thấy dữ liệu của team mình
create policy "Team members can read"
  on documents for select
  using (
    team_id in (
      select team_id from team_members
      where user_id = auth.uid()
    )
  );
```

### 8.5. Test Policies

Trong **Table Editor**, bấm **...** → **View policies** → **Test** với user khác nhau để đảm bảo đúng.

**Test bằng SQL:**
```sql
-- Giả lập user cụ thể (trong SQL Editor, dùng service_role)
set request.jwt.claims to '{"sub": "user-id-abc"}';
select * from tasks;  -- chỉ thấy task của user-id-abc
```

### 8.6. Lỗi Thường Gặp Với RLS

| Lỗi | Nguyên nhân | Fix |
|-----|-------------|-----|
| Query trả về rỗng dù có data | Chưa tạo policy / policy sai | Kiểm tra policies của bảng |
| `new row violates row-level security` | `with check` không pass khi insert | Đảm bảo `user_id` = `auth.uid()` khi insert |
| Admin cũng không thấy gì | RLS bật nhưng không có policy cho admin | Thêm policy cho admin hoặc dùng service_role key |

> 💡 **Mẹo debug:** Tạm tắt RLS (`alter table tasks disable row level security;`) để xem có phải do RLS không. Nhớ bật lại!

---

## 9. Storage — Lưu File

### 9.1. Khái Niệm

**Storage** = nơi lưu file (ảnh đại diện, tài liệu, video...) như AWS S3.

| Khái niệm | Giải thích |
|-----------|-----------|
| **Bucket** | "Thư mục gốc" chứa files. Có 2 loại: public và private |
| **Policy** | Quy định ai được upload/xem/xóa (giống RLS!) |

### 9.2. Tạo Bucket

**Storage** → **+ New Bucket**:
- **Name:** `avatars`
- **Public:** ✅ (nếu muốn ai cũng xem được, ví dụ ảnh đại diện)

### 9.3. Upload File Bằng JS

```javascript
// Upload ảnh đại diện
const file = document.querySelector('input[type=file]').files[0];
const fileName = `${user.id}/avatar.png`;

const { data, error } = await supabase.storage
  .from('avatars')
  .upload(fileName, file, {
    upsert: true,  // ghi đè nếu đã tồn tại
    contentType: 'image/png'
  });

if (error) console.error(error);
else console.log('Upload xong!');
```

### 9.4. Lấy URL File

```javascript
// Bucket public: URL trực tiếp
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl(`${user.id}/avatar.png`);
console.log(data.publicUrl);
// → https://xyz.supabase.co/storage/v1/object/public/avatars/xxx/avatar.png
// Dùng <img src="..."> luôn được!
```

```javascript
// Bucket private: URL có thời hạn (signed URL)
const { data, error } = await supabase.storage
  .from('documents')
  .createSignedUrl('hop-dong.pdf', 3600); // hiệu lực 1 giờ
console.log(data.signedUrl);
```

### 9.5. Storage Policies (RLS Cho File)

Giống RLS cho bảng! **Storage** → bucket → **Policies**:

```sql
-- User chỉ upload vào thư mục của mình
create policy "Users upload own avatar"
  on storage.objects for insert
  with check (
    bucket_id = 'avatars'
    and (storage.foldername(name))[1] = auth.uid()::text
  );

-- Ai cũng xem được avatar
create policy "Public read avatars"
  on storage.objects for select
  using (bucket_id = 'avatars');

-- User chỉ xóa file của mình
create policy "Users delete own files"
  on storage.objects for delete
  using (
    bucket_id = 'avatars'
    and (storage.foldername(name))[1] = auth.uid()::text
  );
```

### 9.6. Các Thao Tác Khác

```javascript
// Liệt kê files
const { data } = await supabase.storage.from('avatars').list(user.id);

// Xóa file
await supabase.storage.from('avatars').remove([`${user.id}/avatar.png`]);

// Di chuyển/đổi tên
await supabase.storage.from('avatars')
  .move('old-name.png', 'new-name.png');

// Download file
const { data: blob } = await supabase.storage
  .from('documents').download('bao-cao.pdf');
```

### 9.7. Giới Hạn Free Tier

| Giới hạn | Free |
|----------|------|
| Storage | 1GB |
| Bandwidth | 2GB/tháng |
| Kích thước file | 50MB/file |

> 💡 File lớn (video) nên dùng Cloudflare R2 hoặc S3 thay vì Supabase Storage free.

---

## 10. Realtime — Dữ Liệu Thời Gian Thực

### 10.1. Realtime Là Gì?

Khi database thay đổi, Supabase **tự động** đẩy update đến frontend qua WebSocket — không cần refresh!

```
User A thêm task  →  Database  →  WebSocket  →  User B thấy ngay! ⚡
```

Dùng cho: chat app, thông báo, dashboard live, game...

### 10.2. Bật Realtime Cho Bảng

**Database** → **Replication** → bật cho bảng `tasks` (hoặc chạy SQL):

```sql
-- Cho phép realtime trên bảng tasks
alter publication supabase_realtime add table tasks;
```

### 10.3. Lắng Nghe Thay Đổi Bằng JS

```javascript
// Lắng nghe mọi thay đổi của bảng tasks
const channel = supabase
  .channel('tasks-changes')
  .on(
    'postgres_changes',
    { event: '*', schema: 'public', table: 'tasks' },
    (payload) => {
      console.log('Thay đổi:', payload.eventType); // INSERT, UPDATE, DELETE
      console.log('Dữ liệu mới:', payload.new);
      console.log('Dữ liệu cũ:', payload.old);
      // Cập nhật UI...
    }
  )
  .subscribe();
```

**Lọc theo event:**
```javascript
// Chỉ nghe INSERT
.on('postgres_changes',
  { event: 'INSERT', schema: 'public', table: 'messages' },
  (payload) => {
    appendMessage(payload.new); // thêm tin nhắn mới vào UI
  })
```

**Lọc theo điều kiện:**
```javascript
// Chỉ nghe task của user hiện tại
.on('postgres_changes',
  {
    event: '*', schema: 'public', table: 'tasks',
    filter: `user_id=eq.${user.id}`
  },
  handleChange)
```

### 10.4. Ví Dụ: Chat App Realtime

```javascript
// Gửi tin nhắn
async function sendMessage(text) {
  await supabase.from('messages').insert({
    text,
    user_id: user.id,
    room_id: 'general'
  });
  // Không cần làm gì thêm — realtime tự đẩy cho mọi người!
}

// Nhận tin nhắn realtime
supabase
  .channel('chat-room')
  .on('postgres_changes',
    { event: 'INSERT', schema: 'public', table: 'messages' },
    (payload) => {
      const msg = payload.new;
      document.getElementById('chat').innerHTML +=
        `<div><b>${msg.user_id}:</b> ${msg.text}</div>`;
    })
  .subscribe();
```

### 10.5. Presence (Ai Đang Online)

```javascript
const channel = supabase.channel('online-users');

// Theo dõi user online
channel
  .on('presence', { event: 'sync' }, () => {
    const state = channel.presenceState();
    console.log('Đang online:', Object.keys(state).length);
  })
  .subscribe(async (status) => {
    if (status === 'SUBSCRIBED') {
      await channel.track({ user_id: user.id, online_at: new Date() });
    }
  });
```

### 10.6. Broadcast (Gửi Event Tùy Ý)

```javascript
// Gửi event (không qua database)
channel.send({
  type: 'broadcast',
  event: 'typing',
  payload: { user_id: user.id }
});

// Nhận event
channel.on('broadcast', { event: 'typing' }, (payload) => {
  showTypingIndicator(payload.user_id); // "An đang nhập..."
});
```

---

## 11. Edge Functions

### 11.1. Edge Functions Là Gì?

**Edge Functions** = chạy code TypeScript/Deno ở server của Supabase (gần user). Dùng khi:
- Cần dùng `service_role` key (bypass RLS) an toàn
- Gọi API bên thứ 3 với secret
- Xử lý webhook (Stripe, GitHub...)
- Logic phức tạp không làm được bằng RLS

### 11.2. Cài Supabase CLI

```bash
# macOS/Linux
curl -fsSL https://supabase.com/install.sh | sh

# Hoặc npm
npm i -g supabase

# Đăng nhập
supabase login

# Link với project
supabase link --project-ref xyz
```

### 11.3. Tạo Function Đầu Tiên

```bash
# Tạo function
supabase functions new hello

# Cấu trúc:
# supabase/functions/hello/index.ts
```

`supabase/functions/hello/index.ts`:
```typescript
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";

serve(async (req) => {
  const { name } = await req.json();
  return new Response(
    JSON.stringify({ message: `Xin chào ${name}! 👋` }),
    { headers: { "Content-Type": "application/json" } }
  );
});
```

### 11.4. Deploy & Gọi

```bash
# Deploy
supabase functions deploy hello
```

```javascript
// Gọi từ frontend
const { data, error } = await supabase.functions.invoke('hello', {
  body: { name: 'An' }
});
console.log(data.message); // "Xin chào An! 👋"
```

### 11.5. Ví Dụ Thực Tế: Xử Lý Thanh Toán Stripe

```typescript
// supabase/functions/create-payment/index.ts
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import Stripe from "https://esm.sh/stripe@13.0.0";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

const stripe = new Stripe(Deno.env.get("STRIPE_SECRET_KEY")!, {
  apiVersion: "2023-10-16",
});

serve(async (req) => {
  // Verify user từ JWT
  const supabase = createClient(
    Deno.env.get("SUPABASE_URL")!,
    Deno.env.get("SUPABASE_ANON_KEY")!,
    { global: { headers: { Authorization: req.headers.get("Authorization")! } } }
  );
  const { data: { user } } = await supabase.auth.getUser();
  if (!user) return new Response("Unauthorized", { status: 401 });

  const { amount, productId } = await req.json();

  // Tạo Stripe payment intent (dùng SECRET key an toàn ở server)
  const paymentIntent = await stripe.paymentIntents.create({
    amount, // tính bằng cent: 100000đ = 10000000
    currency: "vnd",
    metadata: { user_id: user.id, product_id: productId },
  });

  return new Response(
    JSON.stringify({ clientSecret: paymentIntent.client_secret }),
    { headers: { "Content-Type": "application/json" } }
  );
});
```

```bash
# Set secret (không commit vào code!)
supabase secrets set STRIPE_SECRET_KEY=sk_test_xxx
supabase functions deploy create-payment
```

### 11.6. Secrets Cho Functions

```bash
# Xem secrets
supabase secrets list

# Thêm secret
supabase secrets set API_KEY=xxx

# Xóa secret
supabase secrets unset API_KEY
```

---

## 12. JavaScript Client — supabase-js

### 12.1. Cài Đặt & Khởi Tạo

```bash
npm install @supabase/supabase-js
```

```javascript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = 'https://xyz.supabase.co';
const supabaseAnonKey = 'eyJhbGciOi...';

export const supabase = createClient(supabaseUrl, supabaseAnonKey);
```

### 12.2. CRUD Đầy Đủ

```javascript
// ===== CREATE =====
const { data, error } = await supabase
  .from('tasks')
  .insert({ title: 'Học Supabase', user_id: user.id })
  .select();  // trả về dòng vừa tạo

// Insert nhiều
await supabase.from('tasks').insert([
  { title: 'Task 1', user_id: user.id },
  { title: 'Task 2', user_id: user.id },
]);

// ===== READ =====
// Tất cả
const { data: tasks } = await supabase.from('tasks').select('*');

// Chọn cột cụ thể
const { data } = await supabase.from('tasks').select('id, title, is_done');

// Lọc
const { data } = await supabase.from('tasks')
  .select('*')
  .eq('is_done', false)                    // bằng
  .neq('user_id', 'xxx')                   // khác
  .gt('created_at', '2026-01-01')          // lớn hơn
  .like('title', '%họp%')                  // giống (case-sensitive)
  .ilike('title', '%HỌP%');                // giống (không phân biệt hoa thường)

// Sắp xếp + giới hạn + phân trang
const { data } = await supabase.from('tasks')
  .select('*')
  .order('created_at', { ascending: false })
  .range(0, 9);  // 10 items đầu (page 1)

// Đếm
const { count } = await supabase.from('tasks')
  .select('*', { count: 'exact', head: true });

// ===== UPDATE =====
const { data } = await supabase.from('tasks')
  .update({ is_done: true })
  .eq('id', taskId)
  .select();

// ===== DELETE =====
await supabase.from('tasks').delete().eq('id', taskId);

// ===== UPSERT (có thì update, không thì insert) =====
await supabase.from('profiles').upsert(
  { id: user.id, username: 'an123' },
  { onConflict: 'id' }
);
```

### 12.3. Join (Lấy Dữ Liệu Liên Quan)

```javascript
// Lấy tasks kèm thông tin user (dựa vào foreign key)
const { data } = await supabase.from('tasks')
  .select(`
    id,
    title,
    is_done,
    profiles (username, avatar_url)
  `);
// Kết quả: [{ id, title, is_done, profiles: { username, avatar_url } }]
```

### 12.4. Gọi RPC (Postgres Function)

```sql
-- Tạo function trong SQL Editor
create or replace function get_task_stats(user_uuid uuid)
returns table (total bigint, done bigint, pending bigint)
language sql as $$
  select
    count(*),
    count(*) filter (where is_done = true),
    count(*) filter (where is_done = false)
  from tasks
  where user_id = user_uuid;
$$;
```

```javascript
// Gọi từ JS
const { data } = await supabase.rpc('get_task_stats', {
  user_uuid: user.id
});
console.log(data); // [{ total: 10, done: 7, pending: 3 }]
```

---

## 13. Dự Án Thực Tế: App Ghi Chú Full-Stack

### 13.1. Kiến Trúc

```
┌─────────────┐         ┌──────────────┐
│  React App  │────────▶│   Supabase   │
│  (Vite)     │  HTTPS  │  - Auth      │
│             │         │  - Postgres  │
└─────────────┘         │  - Realtime  │
                        └──────────────┘
     Không cần backend riêng!
```

### 13.2. Setup Database

```sql
-- Bảng notes
create table notes (
  id uuid default gen_random_uuid() primary key,
  created_at timestamptz default now(),
  updated_at timestamptz default now(),
  title text not null,
  content text,
  is_pinned boolean default false,
  user_id uuid references auth.users(id) on delete cascade not null
);

-- RLS
alter table notes enable row level security;

create policy "Users manage own notes"
  on notes for all
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);

-- Bật realtime
alter publication supabase_realtime add table notes;

-- Tự động cập nhật updated_at
create or replace function update_updated_at()
returns trigger as $$
begin
  new.updated_at = now();
  return new;
end;
$$ language plpgsql;

create trigger set_updated_at
  before update on notes
  for each row execute function update_updated_at();
```

### 13.3. Frontend React (Vite)

```bash
npm create vite@latest notes-app -- --template react
cd notes-app
npm install
npm install @supabase/supabase-js
```

`.env`:
```
VITE_SUPABASE_URL=https://xyz.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOi...
```

`src/supabase.js`:
```javascript
import { createClient } from '@supabase/supabase-js';

export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL,
  import.meta.env.VITE_SUPABASE_ANON_KEY
);
```

`src/App.jsx`:
```jsx
import { useState, useEffect } from 'react';
import { supabase } from './supabase';

export default function App() {
  const [user, setUser] = useState(null);
  const [notes, setNotes] = useState([]);
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');

  // Auth state
  useEffect(() => {
    supabase.auth.getSession().then(({ data: { session } }) => {
      setUser(session?.user ?? null);
    });
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (_event, session) => setUser(session?.user ?? null)
    );
    return () => subscription.unsubscribe();
  }, []);

  // Load notes + realtime
  useEffect(() => {
    if (!user) return;

    const fetchNotes = async () => {
      const { data } = await supabase
        .from('notes')
        .select('*')
        .order('is_pinned', { ascending: false })
        .order('updated_at', { ascending: false });
      setNotes(data || []);
    };
    fetchNotes();

    // Realtime: tự cập nhật khi có thay đổi
    const channel = supabase
      .channel('notes')
      .on('postgres_changes',
        { event: '*', schema: 'public', table: 'notes' },
        () => fetchNotes()
      )
      .subscribe();

    return () => supabase.removeChannel(channel);
  }, [user]);

  const addNote = async (e) => {
    e.preventDefault();
    if (!title.trim()) return;
    await supabase.from('notes').insert({
      title, content, user_id: user.id
    });
    setTitle(''); setContent('');
  };

  const togglePin = async (note) => {
    await supabase.from('notes')
      .update({ is_pinned: !note.is_pinned })
      .eq('id', note.id);
  };

  const deleteNote = async (id) => {
    if (!confirm('Xóa ghi chú này?')) return;
    await supabase.from('notes').delete().eq('id', id);
  };

  const signInWithGoogle = async () => {
    await supabase.auth.signInWithOAuth({ provider: 'google' });
  };

  if (!user) {
    return (
      <div style={{ textAlign: 'center', marginTop: 100 }}>
        <h1>📝 Notes App</h1>
        <button onClick={signInWithGoogle}>Đăng nhập bằng Google</button>
      </div>
    );
  }

  return (
    <div style={{ maxWidth: 600, margin: '0 auto', padding: 20 }}>
      <h1>📝 Ghi chú của {user.email}</h1>
      <button onClick={() => supabase.auth.signOut()}>Đăng xuất</button>

      <form onSubmit={addNote} style={{ margin: '20px 0' }}>
        <input
          value={title}
          onChange={e => setTitle(e.target.value)}
          placeholder="Tiêu đề..."
          style={{ width: '100%', padding: 8, marginBottom: 8 }}
        />
        <textarea
          value={content}
          onChange={e => setContent(e.target.value)}
          placeholder="Nội dung..."
          rows={3}
          style={{ width: '100%', padding: 8 }}
        />
        <button type="submit">+ Thêm ghi chú</button>
      </form>

      {notes.map(note => (
        <div key={note.id} style={{
          border: '1px solid #ddd', borderRadius: 8,
          padding: 12, marginBottom: 8,
          background: note.is_pinned ? '#fffbe6' : 'white'
        }}>
          <h3>{note.is_pinned && '📌 '}{note.title}</h3>
          <p>{note.content}</p>
          <button onClick={() => togglePin(note)}>
            {note.is_pinned ? 'Bỏ ghim' : 'Ghim'}
          </button>
          <button onClick={() => deleteNote(note.id)}>Xóa</button>
        </div>
      ))}
    </div>
  );
}
```

### 13.4. Deploy

```bash
# Build
npm run build

# Deploy lên Vercel/Render static site
# Nhớ set VITE_SUPABASE_URL và VITE_SUPABASE_ANON_KEY!
```

**Tổng kết:** 1 file SQL + 1 file React = app full-stack hoàn chỉnh với auth, database, realtime. Không viết 1 dòng backend nào! 🎉

---

## 14. Pricing Chi Tiết

### 14.1. Các Gói

| Tính năng | Free | Pro ($25/tháng) | Team ($599/tháng) |
|-----------|------|-----------------|-------------------|
| Database | 500MB | 8GB (+$0.125/GB) | 8GB |
| Bandwidth | 5GB/tháng | 250GB (+$0.09/GB) | 250GB |
| File storage | 1GB | 100GB (+$0.021/GB) | 100GB |
| Edge Functions | 500K invocations | 2M (+$2/1M) | 2M |
| Realtime | 200 concurrent | 500 | 500 |
| Auth users (MAU) | 50,000 | 100,000 (+$0.00325/user) | 100,000 |
| Tạm dừng khi idle | ✅ (sau 7 ngày không dùng) | ❌ | ❌ |
| Backup hàng ngày | ❌ | ✅ 7 ngày | ✅ |

### 14.2. Free Tier Đủ Cho Gì?

- ✅ Side project, học tập, prototype
- ✅ App nhỏ < 50K users/tháng
- ⚠️ Project **bị pause sau 7 ngày không hoạt động** → vào dashboard bấm Resume là chạy lại
- ⚠️ 500MB database — đủ cho app nhỏ, không đủ cho app lưu nhiều ảnh (dùng storage riêng)

### 14.3. Khi Nào Nâng Lên Pro?

- App production thật, cần uptime 24/7 (không bị pause)
- Cần backup tự động
- Vượt giới hạn free (bandwidth, storage...)

> 💡 **Mẹo:** Free tier của Supabase hào phóng hơn Firebase free tier nhiều. Hầu hết side project không bao giờ cần trả tiền!

---

## 15. So Sánh Với Firebase & Đối Thủ

### 15.1. Supabase vs Firebase (Chi Tiết)

| Tiêu chí | Supabase | Firebase |
|----------|----------|----------|
| Database | PostgreSQL (SQL) | Firestore (NoSQL) |
| Query phức tạp | ✅ JOIN, aggregate, full-text | ⚠️ Rất hạn chế |
| Realtime | ✅ | ✅ |
| Auth | ✅ Tương đương | ✅ Tương đương |
| Storage | ✅ | ✅ |
| Functions | ✅ Edge Functions (Deno) | ✅ Cloud Functions (Node) |
| Pricing free | 500MB DB, 5GB bandwidth | 1GB Firestore, 10GB hosting |
| Pricing khi scale | Rẻ hơn đáng kể | Đắt nhanh |
| Open source | ✅ Tự host được | ❌ |
| Learning curve | Cần biết SQL cơ bản | Dễ hơn nếu chưa biết SQL |

### 15.2. Supabase vs Neon vs Turso vs PlanetScale

| Tiêu chí | Supabase | Neon | Turso | PlanetScale |
|----------|----------|------|-------|-------------|
| Loại | BaaS full (DB+Auth+Storage) | Serverless Postgres | SQLite phân tán | Serverless MySQL |
| Database | PostgreSQL | PostgreSQL | SQLite (libSQL) | MySQL (Vitess) |
| Auth có sẵn | ✅ | ❌ | ❌ | ❌ |
| Branching DB | ✅ (Pro) | ✅ Nổi bật | ✅ | ✅ |
| Free tier | 500MB | 512MB | 9GB! | 5GB |
| Phù hợp | App full-stack nhanh | Chỉ cần Postgres tốt | Edge, SQLite | App MySQL |

> 💡 Chỉ cần database thuần (không cần auth/storage) → Neon/Turso có thể hợp hơn. Cần full backend nhanh → Supabase.

---

## 16. Troubleshooting

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `Failed to fetch` / CORS | Sai URL/key | Kiểm tra Project URL và anon key trong Settings → API |
| Query trả về `[]` dù có data | RLS chặn | Kiểm tra policies, test với service_role key để xác nhận |
| `new row violates row-level security` | Policy `with check` fail khi insert | Đảm bảo insert đúng `user_id = auth.uid()` |
| Auth: `Email not confirmed` | Chưa bật/tắt confirm email | Settings → Auth → tắt "Confirm email" cho dev |
| Storage: `403 Forbidden` | Storage policy sai | Kiểm tra policies của bucket |
| Realtime không hoạt động | Chưa bật replication | `alter publication supabase_realtime add table xxx;` |
| `JWT expired` | Token hết hạn | supabase-js tự refresh, kiểm tra `onAuthStateChange` |
| Edge Function 500 | Lỗi code / thiếu secret | Xem logs: `supabase functions logs`, kiểm tra secrets |
| Project bị paused | Free tier idle 7 ngày | Vào dashboard → Resume |
| `password authentication failed` | Sai DB password | Reset trong Settings → Database |
| Quá 500MB database | Free tier đầy | Xóa data cũ hoặc nâng Pro |

### Debug RLS Nhanh

```sql
-- 1. Xem policies hiện tại
select * from pg_policies where tablename = 'tasks';

-- 2. Test với role anon (giả lập frontend)
set role anon;
select * from tasks;  -- xem anon thấy gì

-- 3. Reset về
reset role;
```

---

## 17. Best Practices

### 17.1. Database

- ✅ **Luôn bật RLS** cho bảng chứa dữ liệu user
- ✅ Dùng `uuid` cho primary key (khó đoán hơn `int`)
- ✅ Đặt `on delete cascade` cho foreign key phù hợp
- ✅ Tạo index cho cột hay query: `create index on tasks (user_id);`
- ✅ Dùng connection pooler (port 6543) cho production
- ❌ Đừng select `*` khi chỉ cần vài cột — tốn bandwidth

### 17.2. Auth

- ✅ Bật **Confirm email** cho production
- ✅ Cấu hình **Site URL** đúng domain production (Auth → URL Configuration)
- ✅ Dùng PKCE flow cho mobile app
- ❌ Không bao giờ để lộ `service_role` key ở frontend

### 17.3. Storage

- ✅ Đặt tên file theo `user_id/filename` để dễ viết policy
- ✅ Validate loại file ở frontend trước khi upload
- ✅ Dùng `upsert: true` cho avatar (ghi đè thay vì tạo mới)
- ❌ Không lưu file nhạy cảm ở bucket public

### 17.4. Hiệu Năng

- ✅ Dùng `.select('id, title')` thay vì `.select('*')`
- ✅ Phân trang với `.range()` thay vì lấy hết
- ✅ Cache ở frontend cho dữ liệu ít đổi
- ✅ Realtime: unsubscribe khi component unmount!

```javascript
// React: nhớ cleanup!
useEffect(() => {
  const channel = supabase.channel('x').on(...).subscribe();
  return () => supabase.removeChannel(channel);  // ← quan trọng!
}, []);
```

### 17.5. Bảo Mật Checklist

- [ ] RLS bật cho mọi bảng có dữ liệu user
- [ ] Policies test kỹ với nhiều user khác nhau
- [ ] `service_role` key chỉ ở server, không trong code frontend
- [ ] Storage policies đúng (private cho file nhạy cảm)
- [ ] Site URL cấu hình đúng domain
- [ ] Không commit `.env` lên Git

---

## 18. FAQ

**Hỏi: Supabase có miễn phí thật không?**
Đáp: Có. Free tier 500MB DB + 50K auth users/tháng, không cần thẻ. Đủ cho hầu hết side project.

**Hỏi: Project bị pause là sao?**
Đáp: Free project không hoạt động 7 ngày sẽ tự pause để tiết kiệm tài nguyên. Vào dashboard bấm Resume là chạy lại ngay, dữ liệu không mất.

**Hỏi: Có dùng được với Flutter/React Native không?**
Đáp: Có! Có package `supabase_flutter`. Auth + realtime hoạt động tốt trên mobile.

**Hỏi: Supabase có thay thế hoàn toàn backend không?**
Đáp: Với 80% app thông thường: có. 20% còn lại (logic phức tạp, tích hợp bên thứ 3) dùng Edge Functions.

**Hỏi: Migrate từ Firebase sang Supabase có khó không?**
Đáp: Cần chuyển NoSQL → SQL (thiết kế lại schema). Auth và storage migrate được. Có tool hỗ trợ nhưng vẫn cần làm tay phần lớn.

**Hỏi: Self-host Supabase được không?**
Đáp: Được — có docker-compose chính thức. Nhưng tự host mất đi sự tiện lợi (phải tự lo backup, update...). Khuyên dùng cloud trừ khi có lý do đặc biệt.

**Hỏi: Supabase có hỗ trợ tiếng Việt không?**
Đáp: UI tiếng Anh. Nhưng full-text search tiếng Việt cần cấu hình thêm (dùng `pg_trgm` cho tìm gần đúng là đơn giản nhất).

**Hỏi: Nên học Supabase hay Firebase?**
Đáp: Nếu biết SQL hoặc muốn học SQL → Supabase (tương lai rộng hơn, rẻ hơn). Nếu cần ra sản phẩm cực nhanh và không muốn đụng SQL → Firebase.

---

## Phụ Lục: Cheat Sheet

```javascript
// Khởi tạo
import { createClient } from '@supabase/supabase-js';
const supabase = createClient(URL, ANON_KEY);

// Auth
await supabase.auth.signUp({ email, password });
await supabase.auth.signInWithPassword({ email, password });
await supabase.auth.signInWithOAuth({ provider: 'google' });
await supabase.auth.signOut();
const { data: { user } } = await supabase.auth.getUser();

// CRUD
await supabase.from('tasks').select('*');
await supabase.from('tasks').insert({ title });
await supabase.from('tasks').update({ done: true }).eq('id', id);
await supabase.from('tasks').delete().eq('id', id);

// Lọc
supabase.from('t').select().eq('a', 1).neq('b', 2)
  .gt('c', 3).like('d', '%x%').order('e').range(0, 9);

// Realtime
supabase.channel('x')
  .on('postgres_changes', { event: '*', schema: 'public', table: 't' }, cb)
  .subscribe();

// Storage
await supabase.storage.from('b').upload(path, file);
supabase.storage.from('b').getPublicUrl(path);
await supabase.storage.from('b').createSignedUrl(path, 3600);

// Functions
await supabase.functions.invoke('fn-name', { body: {...} });
```

```sql
-- RLS mẫu
alter table tasks enable row level security;
create policy "own" on tasks for all
  using (auth.uid() = user_id) with check (auth.uid() = user_id);

-- Bật realtime
alter publication supabase_realtime add table tasks;
```

| Thao tác | Vị trí Dashboard |
|----------|-----------------|
| Lấy API keys | Settings → API |
| Tạo bảng | Table Editor → New Table |
| Chạy SQL | SQL Editor |
| Quản lý users | Authentication → Users |
| Bật OAuth provider | Authentication → Providers |
| Tạo bucket | Storage → New Bucket |
| Deploy function | `supabase functions deploy <name>` |
| Xem logs function | `supabase functions logs` |

**Links:**
- Docs: https://supabase.com/docs
- Discord VN: tìm "Supabase Vietnam" trên Facebook/Discord

---
