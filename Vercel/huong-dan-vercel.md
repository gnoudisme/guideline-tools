# Hướng Dẫn Toàn Diện Vercel

> Tài liệu tiếng Việt chi tiết về mọi tính năng và cách sử dụng Vercel — từ deploy web đầu tiên đến production.
> Cập nhật: 2026-10-10

---

## Mục Lục

1. [Vercel Là Gì?](#1-vercel-là-gì)
2. [Tạo Tài Khoản](#2-tạo-tài-khoản)
3. [Kết Nối GitHub](#3-kết-nối-github)
4. [Deploy Dự Án Đầu Tiên](#4-deploy-dự-án-đầu-tiên)
5. [Framework Presets](#5-framework-presets)
6. [Biến Môi Trường (Environment Variables)](#6-biến-môi-trường-environment-variables)
7. [Custom Domain — Gắn Tên Miền Riêng](#7-custom-domain--gắn-tên-miền-riêng)
8. [Preview Deployments](#8-preview-deployments)
9. [Serverless Functions](#9-serverless-functions)
10. [Vercel CLI — Deploy Từ Terminal](#10-vercel-cli--deploy-từ-terminal)
11. [Redirects & Rewrites](#11-redirects--rewrites)
12. [Analytics & Speed Insights](#12-analytics--speed-insights)
13. [Quản Lý Dự Án & Team](#13-quản-lý-dự-án--team)
14. [Xử Lý Lỗi Thường Gặp](#14-xử-lý-lỗi-thường-gặp)
15. [Vercel Pro & Giới Hạn Gói Free](#15-vercel-pro--giới-hạn-gói-free)
16. [Bảo Mật & Best Practices](#16-bảo-mật--best-practices)
17. [Tài Liệu Tham Khảo](#17-tài-liệu-tham-khảo)

---

## 1. Vercel Là Gì?

### 1.1 Định Nghĩa

**Vercel** là nền tảng cloud chuyên để **deploy website và web app** — đặc biệt mạnh với frontend (React, Next.js, Vue...). Bạn push code lên GitHub, Vercel tự động build và đưa web lên mạng với URL công khai.

**Slogan:** "Develop. Preview. Ship." — Viết code, xem trước, xuất bản.

### 1.2 Tại Sao Dùng Vercel?

| Ưu điểm | Giải thích |
|---------|-----------|
| **Miễn phí hào phóng** | Deploy không giới hạn dự án cá nhân, 100GB bandwidth/tháng |
| **Tự động từ Git** | Push code → tự build → tự deploy, không cần thao tác tay |
| **Preview URL** | Mỗi pull request có link xem trước riêng |
| **CDN toàn cầu** | Web load nhanh ở mọi nơi trên thế giới |
| **HTTPS tự động** | Chứng chỉ SSL miễn phí, tự gia hạn |
| **Zero config** | Nhận diện framework tự động, không cần cấu hình phức tạp |

### 1.3 Vercel vs Các Nền Tảng Khác

| Tiêu chí | Vercel | Netlify | GitHub Pages | VPS tự quản |
|----------|--------|---------|--------------|-------------|
| Deploy từ Git | ✅ Tự động | ✅ Tự động | ✅ Tự động | ❌ Thủ công |
| Serverless Functions | ✅ | ✅ | ❌ | Tự cài |
| Framework detection | ✅ Rất tốt | ✅ Tốt | ❌ | ❌ |
| Free tier | Hào phóng | Hào phóng | Giới hạn | Trả phí |
| Phù hợp Next.js | ⭐ Tốt nhất | Tốt | Không | Tự config |

> 💡 Vercel được tạo bởi cùng team phát triển **Next.js** — nếu dùng Next.js thì Vercel là lựa chọn số 1.

---

## 2. Tạo Tài Khoản

### 2.1 Đăng Ký

1. Vào https://vercel.com
2. Bấm **Sign Up**
3. Chọn 1 trong 3 cách:
   - **Continue with GitHub** (khuyên dùng — để deploy từ GitHub)
   - Continue with GitLab
   - Continue with Bitbucket

### 2.2 Các Loại Tài Khoản

| Loại | Giá | Dành cho |
|------|-----|----------|
| **Hobby** | Miễn phí | Cá nhân, project phi thương mại |
| **Pro** | $20/tháng/user | Freelancer, team nhỏ, thương mại |
| **Enterprise** | Liên hệ | Công ty lớn |

> ⚠️ **Lưu ý quan trọng:** Gói Hobby **không cho phép dùng cho mục đích thương mại** (theo điều khoản mới của Vercel). Nếu web kiếm tiền → cần lên Pro.

### 2.3 Dashboard Tổng Quan

Sau khi đăng nhập, bạn thấy:
- **Overview:** Danh sách dự án đã deploy
- **Activity:** Lịch sử deploy
- **Domains:** Quản lý tên miền
- **Usage:** Xem mức sử dụng (bandwidth, build minutes)
- **Settings:** Cài đặt tài khoản, team

---

## 3. Kết Nối GitHub

### 3.1 Tại Sao Kết Nối GitHub?

Kết nối GitHub cho phép Vercel:
- Tự động phát hiện repo của bạn
- Auto-deploy mỗi khi bạn push code
- Tạo preview URL cho mỗi pull request
- Hiển thị trạng thái deploy ngay trên GitHub

### 3.2 Cách Kết Nối

**Nếu đăng ký bằng GitHub:** Đã tự động kết nối.

**Nếu đăng ký bằng email/GitLab:**
1. Vào **Settings** → **Connected Git Accounts**
2. Bấm **Connect** bên cạnh GitHub
3. Cấp quyền cho Vercel truy cập repo

### 3.3 Phân Quyền Repo

Khi kết nối, bạn chọn:
- **All repositories:** Vercel thấy tất cả repo (tiện nhất)
- **Only select repositories:** Chỉ cho phép 1 số repo cụ thể (an toàn hơn)

> 🔒 **Khuyên dùng:** Chọn "Only select repositories" và chỉ cấp quyền cho repo cần deploy.

---

## 4. Deploy Dự Án Đầu Tiên

### 4.1 Chuẩn Bị

Bạn cần:
1. Tài khoản Vercel (đã kết nối GitHub)
2. Một repo GitHub chứa code web
3. Code có file build được (ví dụ: `package.json` với script `build`)

### 4.2 Các Bước Deploy

**Bước 1:** Vào https://vercel.com/new (hoặc bấm **Add New...** → **Project**)

**Bước 2:** Chọn repo từ danh sách → bấm **Import**

**Bước 3:** Cấu hình dự án:

| Mục | Giải thích | Ví dụ |
|-----|-----------|-------|
| **Project Name** | Tên dự án (thành URL) | `my-awesome-web` → `my-awesome-web.vercel.app` |
| **Framework Preset** | Vercel tự nhận diện | Next.js, Vite, Create React App... |
| **Root Directory** | Thư mục chứa code (nếu repo có nhiều project) | `./` (mặc định) hoặc `frontend/` |
| **Build Command** | Lệnh build | `npm run build` (tự điền) |
| **Output Directory** | Thư mục chứa file sau build | `dist`, `.next`, `build` (tự điền) |
| **Install Command** | Lệnh cài dependencies | `npm install` (tự điền) |

**Bước 4:** Bấm **Deploy** → đợi 1-3 phút → xong!

**Bước 5:** Vercel cấp URL: `https://ten-du-an.vercel.app` — bấm vào để xem web!

### 4.2.1 Deploy Dự Án HTML/CSS/JS Thuần (Không Framework)

Nếu dự án chỉ có file `index.html` (không cần build):

1. Import repo như bình thường
2. Framework Preset → chọn **Other**
3. **Build Command:** để trống (hoặc `echo "no build"`)
4. **Output Directory:** `./` (thư mục gốc)
5. Deploy!

### 4.3 Auto-Deploy Sau Này

Từ nay, mỗi khi bạn:
```bash
git add .
git commit -m "Sửa giao diện"
git push origin main
```

→ Vercel **tự động** build và deploy phiên bản mới trong 1-3 phút. Bạn không cần làm gì thêm!

### 4.4 Xem Lịch Sử Deploy

Vào dự án → tab **Deployments**:
- Mỗi dòng là 1 lần deploy
- Hiện: commit message, người push, thời gian, trạng thái
- **Building** (đang build) → **Ready** (xong) hoặc **Error** (lỗi)
- Bấm vào từng deployment để xem log chi tiết

### 4.5 Rollback — Quay Về Bản Cũ

Nếu bản mới bị lỗi:
1. Vào **Deployments**
2. Tìm bản cũ còn **Ready**
3. Bấm `⋯` → **Promote to Production**
4. Xác nhận → web quay về bản cũ ngay lập tức!

---

## 5. Framework Presets

Vercel tự nhận diện framework và điền sẵn cấu hình build. Dưới đây là các preset phổ biến:

### 5.1 Next.js (Khuyên Dùng Nhất Trên Vercel)

| Mục | Giá trị tự động |
|-----|----------------|
| Build Command | `next build` (hoặc `prisma generate && next build`) |
| Output Directory | `.next` |
| Install Command | `npm install` / `yarn install` / `pnpm install` |
| Dev Command | `next dev` |

**Lưu ý đặc biệt:**
- Vercel hỗ trợ **mọi tính năng** Next.js: SSR, ISR, API Routes, Middleware, Image Optimization
- Không cần config gì thêm — deploy là chạy

### 5.2 Vite (React, Vue, Svelte...)

| Mục | Giá trị tự động |
|-----|----------------|
| Build Command | `npm run build` (chạy `vite build`) |
| Output Directory | `dist` |
| Install Command | `npm install` |

**File `vite.config.js` mẫu:**
```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  // Không cần config gì đặc biệt cho Vercel
})
```

### 5.3 Create React App (CRA)

| Mục | Giá trị tự động |
|-----|----------------|
| Build Command | `npm run build` |
| Output Directory | `build` |

### 5.4 Vue / Nuxt

**Vue (Vite):**
- Build: `npm run build`
- Output: `dist`

**Nuxt 3:**
- Build: `npm run build`
- Output: `.output/public`

### 5.5 Static Site (Hugo, Jekyll, Astro...)

**Astro:**
- Build: `npm run build`
- Output: `dist`

**Hugo:**
- Build: `hugo --gc --minify`
- Output: `public`

### 5.6 Tự Định Nghĩa (Other)

Nếu framework không có trong danh sách:
1. Chọn **Other**
2. Tự điền Build Command và Output Directory
3. Tham khảo tài liệu framework để biết lệnh build đúng

---

## 6. Biến Môi Trường (Environment Variables)

### 6.1 Biến Môi Trường Là Gì?

Là các giá trị cấu hình **không nên ghi cứng vào code**, ví dụ:
- API keys (OpenAI, Google Maps...)
- Database URL
- Secret tokens

```js
// ❌ SAI: ghi cứng vào code
const API_KEY = "sk-abc123xyz";

// ✅ ĐÚNG: đọc từ biến môi trường
const API_KEY = process.env.NEXT_PUBLIC_API_KEY;
```

### 6.2 Thêm Biến Môi Trường

1. Vào dự án → **Settings** → **Environment Variables**
2. Điền:
   - **Key:** Tên biến (ví dụ: `DATABASE_URL`)
   - **Value:** Giá trị (ví dụ: `postgres://user:pass@host/db`)
   - **Environments:** Chọn môi trường áp dụng:
     - ✅ Production (web chính)
     - ✅ Preview (link preview)
     - ⬜ Development (chạy local)
3. Bấm **Save**
4. **Redeploy** để biến có hiệu lực!

### 6.3 Quy Tắc Đặt Tên

| Framework | Biến dùng ở client (trình duyệt) | Biến chỉ dùng ở server |
|-----------|----------------------------------|------------------------|
| Next.js | Phải bắt đầu bằng `NEXT_PUBLIC_` | Tên tùy ý |
| Vite | Phải bắt đầu bằng `VITE_` | Tên tùy ý |
| CRA | Phải bắt đầu bằng `REACT_APP_` | Tên tùy ý |

```js
// Next.js
// ✅ Client thấy được:
const key = process.env.NEXT_PUBLIC_MAPS_KEY;
// 🔒 Chỉ server thấy (an toàn cho secret):
const secret = process.env.DATABASE_URL;
```

> ⚠️ **CẢNH BÁO:** Biến có prefix `NEXT_PUBLIC_` / `VITE_` / `REACT_APP_` sẽ **lộ ra trình duyệt** — ai cũng xem được! Không bao giờ để secret key vào đó!

### 6.4 Dùng File `.env` Local

Khi chạy local, tạo file `.env.local` (không push lên GitHub!):

```
# .env.local
DATABASE_URL=postgres://localhost/mydb
NEXT_PUBLIC_API_URL=http://localhost:3000
```

Thêm vào `.gitignore`:
```
.env.local
.env*.local
```

### 6.5 Kéo Biến Từ Vercel Về Local

```bash
# Cài Vercel CLI trước: npm i -g vercel

# Kéo biến môi trường về file .env.local
vercel env pull .env.local
```

---

## 7. Custom Domain — Gắn Tên Miền Riêng

### 7.1 Tại Sao Cần?

Thay vì `my-web.vercel.app`, bạn muốn `myweb.com` — chuyên nghiệp hơn!

### 7.2 Mua Tên Miền Ở Đâu?

- **Trong Vercel:** Vào dự án → **Settings** → **Domains** → **Buy** (giá từ ~$10/năm)
- **Ngoài:** Namecheap, GoDaddy, Cloudflare Registrar...

### 7.3 Gắn Domain Đã Mua

**Bước 1:** Vào dự án → **Settings** → **Domains** → nhập domain → **Add**

**Bước 2:** Vercel hiện hướng dẫn DNS. Bạn có 2 cách:

**Cách A — Dùng Nameserver của Vercel (khuyên dùng):**
1. Vào trang quản lý domain (nơi bạn mua)
2. Đổi nameserver thành:
   ```
   ns1.vercel-dns.com
   ns2.vercel-dns.com
   ```
3. Đợi 5-60 phút → tự động xong!

**Cách B — Thêm bản ghi DNS thủ công:**

| Type | Name | Value |
|------|------|-------|
| A | `@` | `76.76.21.21` |
| CNAME | `www` | `cname.vercel-dns.com` |

### 7.4 HTTPS Tự Động

Vercel tự cấp chứng chỉ SSL miễn phí (Let's Encrypt) cho domain của bạn. Không cần làm gì thêm!

### 7.5 Domain `www` và Non-`www`

Sau khi thêm `myweb.com`, nên thêm cả `www.myweb.com`:
1. Thêm domain `www.myweb.com`
2. Vào domain chính → **Edit** → bật **Redirect** từ `www` về non-`www` (hoặc ngược lại)

### 7.6 Kiểm Tra DNS

```bash
# Kiểm tra domain đã trỏ đúng chưa
nslookup myweb.com
dig myweb.com +short
```

---

## 8. Preview Deployments

### 8.1 Preview Deployment Là Gì?

Mỗi khi bạn tạo **Pull Request** trên GitHub, Vercel tự động deploy một bản **xem trước** với URL riêng:

```
https://my-web-git-feature-login-ten.vercel.app
```

→ Bạn (và team) xem thử thay đổi **trước khi merge** vào main!

### 8.2 Quy Trình Làm Việc Với Preview

```bash
# 1. Tạo nhánh mới
git checkout -b feature/dark-mode

# 2. Code...

# 3. Push nhánh
git push origin feature/dark-mode

# 4. Tạo Pull Request trên GitHub
# → Vercel tự comment link preview vào PR!

# 5. Team xem preview, review code

# 6. Merge PR → Vercel deploy lên production
```

### 8.3 Comment Tự Động Trên PR

Vercel bot sẽ comment vào PR:
```
✅ Preview: https://my-web-git-feature-dark-mode-ten.vercel.app
🔍 Inspect: https://vercel.com/ten/du-an/abc123
```

### 8.4 Bảo Vệ Preview Bằng Password

Nếu không muốn ai cũng xem được preview:
1. **Settings** → **Deployment Protection**
2. Bật **Password Protection** (gói Pro) hoặc **Vercel Authentication**
3. Người xem phải đăng nhập Vercel mới xem được

### 8.5 Xóa Preview Cũ

Preview tự động bị xóa khi:
- PR được merge hoặc đóng
- Quá 30 ngày không dùng (gói Hobby)

Xóa tay: **Deployments** → chọn preview → `⋯` → **Delete**

---

## 9. Serverless Functions

### 9.1 Serverless Function Là Gì?

Là **backend API** chạy trên Vercel mà không cần quản lý server. Bạn viết function, Vercel tự chạy khi có request.

### 9.2 Tạo API Route (Next.js)

Tạo file trong `app/api/`:

```typescript
// app/api/hello/route.ts
export async function GET() {
  return Response.json({
    message: "Xin chào từ Vercel!",
    time: new Date().toISOString(),
  });
}
```

→ API có tại: `https://my-web.vercel.app/api/hello`

### 9.3 Xử Lý POST Request

```typescript
// app/api/users/route.ts
export async function POST(request: Request) {
  const body = await request.json();
  const { name, email } = body;

  // Xử lý... (lưu database, gửi email...)

  return Response.json(
    { success: true, user: { name, email } },
    { status: 201 }
  );
}
```

### 9.4 Dynamic Routes

```typescript
// app/api/users/[id]/route.ts
export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  const userId = params.id;
  // Lấy user theo ID...
  return Response.json({ id: userId, name: "Nguyễn Văn A" });
}
```

→ `GET /api/users/123` trả về user có id=123

### 9.5 Giới Hạn (Gói Hobby)

| Giới hạn | Hobby | Pro |
|----------|-------|-----|
| Thời gian chạy/function | 10 giây | 60 giây (config được 800s) |
| Kích thước function | 50 MB | 50 MB |
| Số lần gọi | 100k/tháng | 1000k/tháng |

> ⚠️ Function phải trả về trong 10 giây (Hobby)! Nếu xử lý lâu → dùng **Edge Functions** hoặc nâng cấp Pro.

### 9.6 Biến Môi Trường Trong Function

```typescript
// app/api/secret/route.ts
export async function GET() {
  // Đọc từ Environment Variables (đã set ở dashboard)
  const apiKey = process.env.MY_SECRET_KEY;

  const res = await fetch("https://api.example.com/data", {
    headers: { Authorization: `Bearer ${apiKey}` },
  });
  const data = await res.json();

  return Response.json(data);
}
```

---

## 10. Vercel CLI — Deploy Từ Terminal

### 10.1 Cài Đặt

```bash
# Cài global
npm install -g vercel

# Kiểm tra
vercel --version
```

### 10.2 Đăng Nhập

```bash
vercel login
# → Mở trình duyệt → xác nhận → xong!
```

### 10.3 Deploy Dự Án

```bash
# Vào thư mục dự án
cd my-web

# Deploy (lần đầu sẽ hỏi cấu hình)
vercel

# Deploy thẳng lên production (bỏ qua preview)
vercel --prod
```

### 10.4 Các Lệnh Thường Dùng

| Lệnh | Chức năng |
|------|-----------|
| `vercel` | Deploy preview |
| `vercel --prod` | Deploy production |
| `vercel ls` | Liệt kê deployments |
| `vercel inspect <url>` | Xem chi tiết deployment |
| `vercel logs <url>` | Xem log |
| `vercel rm <url>` | Xóa deployment |
| `vercel env ls` | Liệt kê biến môi trường |
| `vercel env add` | Thêm biến môi trường |
| `vercel env pull` | Kéo biến về file local |
| `vercel domains ls` | Liệt kê domains |
| `vercel --help` | Xem tất cả lệnh |

### 10.5 Deploy Thư Mục Tĩnh Nhanh

```bash
# Deploy 1 thư mục HTML tĩnh trong 10 giây
mkdir quick-site && cd quick-site
echo "<h1>Hello Vercel!</h1>" > index.html
vercel --prod
# → Có ngay link public!
```

---

## 11. Redirects & Rewrites

### 11.1 Redirects (Chuyển Hướng URL)

Khi đổi URL cũ sang mới, dùng redirect để không mất SEO:

**Cách 1 — File `vercel.json`:**
```json
{
  "redirects": [
    {
      "source": "/blog-cu/:slug",
      "destination": "/blog-moi/:slug",
      "permanent": true
    },
    {
      "source": "/cu",
      "destination": "https://trang-moi.com",
      "permanent": false
    }
  ]
}
```

**Cách 2 — Trong `next.config.js` (Next.js):**
```js
module.exports = {
  async redirects() {
    return [
      {
        source: '/blog-cu/:slug',
        destination: '/blog-moi/:slug',
        permanent: true,  // 308 redirect
      },
    ];
  },
};
```

### 11.2 Rewrites (Viết Lại URL Ẩn)

Rewrite đổi URL hiển thị mà **không đổi URL thật** (user không biết):

```json
{
  "rewrites": [
    {
      "source": "/api/:path*",
      "destination": "https://backend-rieng.com/:path*"
    }
  ]
}
```
→ User gọi `/api/users` nhưng thực tế request tới `backend-rieng.com/users`. Hữu ích để **giấu backend URL**!

### 11.3 Headers Tùy Chỉnh

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" }
      ]
    }
  ]
}
```

---

## 12. Analytics & Speed Insights

### 12.1 Vercel Analytics

Theo dõi lượt truy cập, không cần Google Analytics:

1. Vào dự án → **Analytics** tab
2. Bấm **Enable**
3. Thêm package (Next.js):
```bash
npm install @vercel/analytics
```

```tsx
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

**Xem được:** Page views, visitors, top pages, top referrers, devices, countries.

### 12.2 Speed Insights

Đo hiệu năng web (Core Web Vitals):

```bash
npm install @vercel/speed-insights
```

```tsx
import { SpeedInsights } from '@vercel/speed-insights/next';

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <SpeedInsights />
      </body>
    </html>
  );
}
```

**Xem được:** LCP, FID/INP, CLS — điểm tốc độ từng trang.

> 💡 Gói Hobby: Analytics và Speed Insights có giới hạn events/tháng. Đủ dùng cho web cá nhân.

---

## 13. Quản Lý Dự Án & Team

### 13.1 Tạo Team

1. Bấm avatar → **Create Team**
2. Đặt tên team → **Create**
3. Mời thành viên bằng email

### 13.2 Phân Quyền

| Role | Quyền |
|------|-------|
| **Owner** | Toàn quyền (xóa team, thanh toán) |
| **Member** | Deploy, sửa settings dự án |
| **Viewer** | Chỉ xem |

### 13.3 Chuyển Dự Án Sang Team

Dự án → **Settings** → **Transfer Project** → chọn team.

### 13.4 Monorepo (Nhiều App Trong 1 Repo)

```
my-monorepo/
├── apps/
│   ├── web/          → Deploy thành web.vercel.app
│   └── docs/         → Deploy thành docs.vercel.app
├── packages/
│   └── ui/           → Shared components
```

Mỗi app tạo 1 Vercel project riêng, set **Root Directory** tương ứng (`apps/web`, `apps/docs`).

---

## 14. Xử Lý Lỗi Thường Gặp

### 14.1 Build Failed

**Nguyên nhân phổ biến:**

| Lỗi | Cách fix |
|-----|----------|
| `Module not found` | Chạy `npm install` local xem có lỗi không; kiểm tra tên import đúng case-sensitive |
| `Command "npm run build" exited with 1` | Chạy `npm run build` ở local để xem lỗi chi tiết |
| Thiếu biến môi trường | Thêm vào **Settings** → **Environment Variables** → redeploy |
| Node version sai | Set trong **Settings** → **General** → **Node.js Version** |

**Xem log chi tiết:** Deployments → bấm vào deployment lỗi → xem **Build Logs**.

### 14.2 Trang Trắng / 404 Sau Deploy

- Kiểm tra **Output Directory** đúng chưa (`dist`, `build`, `.next`)
- Với SPA (React Router): thêm `vercel.json`:
```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

### 14.3 API Route Không Chạy

- Kiểm tra file đặt đúng `app/api/.../route.ts` (Next.js 13+)
- Xem **Runtime Logs**: Deployments → deployment → tab **Functions**
- Kiểm tra timeout (Hobby: 10s)

### 14.4 "This deployment is queued"

Hết build minutes (Hobby: 6000 phút/tháng). Đợi sang tháng mới hoặc nâng cấp.

### 14.5 Domain Không Hoạt Động

1. Kiểm tra DNS: `nslookup tenmien.com`
2. Đợi DNS propagate (tối đa 48h, thường 5-60 phút)
3. Kiểm tra trong **Settings** → **Domains** có báo lỗi gì không

---

## 15. Vercel Pro & Giới Hạn Gói Free

### 15.1 Giới Hạn Gói Hobby (Miễn Phí)

| Tài nguyên | Giới hạn/tháng |
|------------|----------------|
| Bandwidth | 100 GB |
| Build minutes | 6,000 phút |
| Serverless function calls | 100,000 |
| Image optimization | 1,000 ảnh |
| Team members | 1 (cá nhân) |

### 15.2 Khi Nào Cần Lên Pro ($20/tháng)?

- ✅ Web thương mại (bán hàng, dịch vụ thu phí)
- ✅ Cần nhiều bandwidth (>100GB)
- ✅ Cần password-protect preview
- ✅ Team nhiều người
- ✅ Function chạy lâu hơn 10s

### 15.3 Theo Dõi Usage

Vào **Settings** → **Usage** để xem:
- Bandwidth đã dùng
- Build minutes đã dùng
- Số function calls

> 💡 Vượt giới hạn → web vẫn chạy nhưng có thể bị tính phí hoặc tạm dừng. Theo dõi thường xuyên!

---

## 16. Bảo Mật & Best Practices

### 16.1 Checklist Bảo Mật

- [ ] Không commit file `.env` lên GitHub (thêm vào `.gitignore`)
- [ ] Secret key chỉ để ở **Environment Variables** (server-side)
- [ ] Không dùng prefix `NEXT_PUBLIC_` cho secret
- [ ] Bật **Deployment Protection** cho preview (nếu có thông tin nhạy cảm)
- [ ] Dùng **Custom Domain** với HTTPS (tự động)
- [ ] Thêm security headers trong `vercel.json`

### 16.2 `.gitignore` Chuẩn

```
# Dependencies
node_modules/

# Build output
.next/
dist/
build/
out/

# Env files
.env
.env.local
.env*.local

# Vercel
.vercel

# OS
.DS_Store
```

### 16.3 Tối Ưu Hiệu Năng

- Dùng `next/image` để tối ưu ảnh tự động
- Bật **Edge Caching** cho nội dung tĩnh
- Giảm kích thước bundle: `npm run build` → kiểm tra output
- Dùng **ISR** (Incremental Static Regeneration) thay vì SSR toàn trang

---

## 17. Tài Liệu Tham Khảo

- 🏠 Trang chủ: https://vercel.com
- 📖 Docs: https://vercel.com/docs
- 💻 Vercel CLI docs: https://vercel.com/docs/cli
- 🎓 Learn Next.js: https://nextjs.org/learn
- 💬 Community: https://github.com/vercel/vercel/discussions
- 📊 Trạng thái hệ thống: https://www.vercel-status.com

---

