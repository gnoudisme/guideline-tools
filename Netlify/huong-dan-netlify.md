# Hướng Dẫn Toàn Tập: Netlify

> **Cấp độ:** Người mới bắt đầu → deploy được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật lần cuối:** 10/10/2026

---

## Mục Lục

1. [Netlify là gì?](#1-netlify-là-gì)
2. [So sánh Netlify với Vercel, GitHub Pages](#2-so-sánh-netlify-với-vercel-github-pages)
3. [Tạo tài khoản](#3-tạo-tài-khoản)
4. [Deploy web tĩnh bằng kéo-thả](#4-deploy-web-tĩnh-bằng-kéo-thả)
5. [Deploy từ GitHub (cách chính)](#5-deploy-từ-github-cách-chính)
6. [Cài đặt build](#6-cài-đặt-build)
7. [Deploy Preview & Branch Deploy](#7-deploy-preview--branch-deploy)
8. [Custom domain](#8-custom-domain)
9. [HTTPS miễn phí](#9-https-miễn-phí)
10. [Biến môi trường](#10-biến-môi-trường)
11. [Redirects & Rewrites](#11-redirects--rewrites)
12. [Headers tùy chỉnh](#12-headers-tùy-chỉnh)
13. [Netlify Forms — nhận form không cần backend](#13-netlify-forms--nhận-form-không-cần-backend)
14. [Serverless Functions](#14-serverless-functions)
15. [Edge Functions](#15-edge-functions)
16. [Netlify CLI](#16-netlify-cli)
17. [Rollback & deploy history](#17-rollback--deploy-history)
18. [Tối ưu hiệu suất](#18-tối-ưu-hiệu-suất)
19. [Pricing](#19-pricing)
20. [Troubleshooting](#20-troubleshooting)
21. [Best practices & Cheat sheet](#21-best-practices--cheat-sheet)

---

## 1. Netlify Là Gì?

**Netlify** là nền tảng deploy website/app frontend — bạn push code lên GitHub, Netlify tự build và đưa lên mạng trong vài phút, miễn phí.

### 1.1. Netlify làm được gì?

- Host **web tĩnh** (HTML/CSS/JS thuần)
- Host **SPA** (React, Vue, Angular, Svelte)
- Host **Static Site Generator** (Next.js static export, Astro, Hugo, Jekyll, Gatsby)
- Chạy **serverless functions** (backend nhẹ không cần server)
- Nhận **form** không cần backend
- **Preview** từng pull request tự động

### 1.2. Quy trình deploy điển hình

```
Bạn push code → GitHub
       ↓
Netlify phát hiện → chạy lệnh build (npm run build)
       ↓
Lấy thư mục output (dist/, build/, .next/out)
       ↓
Đưa lên CDN toàn cầu → có URL https://ten-cua-ban.netlify.app
```

Mỗi lần push mới → Netlify tự build lại. Không cần động tay!

---

## 2. So Sánh Netlify Với Vercel, GitHub Pages

| Tiêu chí | Netlify | Vercel | GitHub Pages |
|----------|---------|--------|--------------|
| Giá free | 100GB bandwidth/tháng | 100GB bandwidth/tháng | Không giới hạn (public repo) |
| Build minutes | 300 phút/tháng | 6000 phút/tháng | 2000 phút (Actions) |
| Framework tối ưu | Đa dạng | **Next.js tốt nhất** | Chỉ web tĩnh |
| Serverless functions | ✅ (AWS Lambda) | ✅ (Edge + Serverless) | ❌ |
| Forms | ✅ có sẵn | ❌ | ❌ |
| Deploy preview | ✅ | ✅ | ❌ |
| Custom domain + HTTPS | ✅ miễn phí | ✅ miễn phí | ✅ miễn phí |
| Analytics | Trả phí | Có free cơ bản | ❌ |

**Khi nào chọn Netlify:**
- Web tĩnh, SPA, blog, landing page
- Cần Forms không backend
- Cần functions nhẹ
- Thích giao diện đơn giản

**Khi nào chọn Vercel:**
- Dùng Next.js (tối ưu nhất)
- Cần Edge functions mạnh

---

## 3. Tạo Tài Khoản

1. Vào https://www.netlify.com → **Sign up**
2. Chọn **Sign up with GitHub** (khuyên dùng — để deploy từ GitHub sau này)
3. Xác nhận email
4. Xong! Bạn có ngay 1 site miễn phí

---

## 4. Deploy Web Tĩnh Bằng Kéo-Thả

Cách nhanh nhất để đưa web lên mạng — không cần GitHub!

### 4.1. Chuẩn bị

Tạo thư mục chứa web của bạn, ví dụ:

```
web-cua-toi/
├── index.html
├── style.css
└── script.js
```

### 4.2. Kéo thả deploy

1. Đăng nhập Netlify → vào https://app.netlify.com/drop
2. **Kéo thả** thư mục `web-cua-toi` vào vùng "Drag and drop"
3. Đợi vài giây → có ngay URL `https://ten-ngau-nhien.netlify.app`
4. Bấm vào tên site → **Site settings** → **Change site name** để đổi tên đẹp

### 4.3. Cập nhật web

Kéo thả lại thư mục mới → Netlify tự cập nhật. (Mỗi lần kéo thả là 1 deploy mới.)

> 💡 Cách này hợp cho web đơn giản, demo nhanh. Dự án thật nên dùng GitHub (mục 5).

---

## 5. Deploy Từ GitHub (Cách Chính)

### 5.1. Chuẩn bị repo

Repo GitHub của bạn cần có:
- Code web (ví dụ React app)
- File `package.json` với script `build`

Ví dụ cấu trúc React + Vite:

```
my-app/
├── src/
├── index.html
├── package.json     ← có "scripts": { "build": "vite build" }
└── vite.config.js
```

### 5.2. Kết nối GitHub với Netlify

1. Netlify Dashboard → **Add new site** → **Import an existing project**
2. Chọn **Deploy with GitHub** → xác nhận quyền
3. Chọn repo của bạn
4. Cài đặt build:
   - **Branch to deploy:** `main`
   - **Build command:** `npm run build`
   - **Publish directory:** `dist` (Vite) / `build` (Create React App) / `out` (Next.js export)
5. Bấm **Deploy site**
6. Đợi 1-3 phút → có URL!

### 5.3. Bảng publish directory theo framework

| Framework | Build command | Publish directory |
|-----------|---------------|-------------------|
| HTML thuần | (để trống) | `.` (thư mục gốc) |
| Vite | `npm run build` | `dist` |
| Create React App | `npm run build` | `build` |
| Vue CLI | `npm run build` | `dist` |
| Angular | `npm run build` | `dist/ten-project` |
| Next.js (static) | `npm run build` | `out` |
| Astro | `npm run build` | `dist` |
| Hugo | `hugo` | `public` |
| Jekyll | `jekyll build` | `_site` |
| Gatsby | `npm run build` | `public` |

### 5.4. File `netlify.toml` (khuyên dùng)

Thay vì cài đặt trên web, khai báo trong file `netlify.toml` ở gốc repo — version control được, ai clone cũng deploy giống nhau:

```toml
[build]
  command = "npm run build"
  publish = "dist"

# Biến môi trường cho production
[build.environment]
  NODE_VERSION = "20"
  NPM_FLAGS = "--legacy-peer-deps"

# Redirect cho SPA (React Router etc.)
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

---

## 6. Cài Đặt Build

### 6.1. Chỉ định phiên bản Node.js

Netlify mặc định dùng Node LTS mới nhất. Để cố định version (tránh build lỗi khi Node update):

**Cách 1 — file `.nvmrc`:**
```
20
```

**Cách 2 — biến môi trường:**
```
NODE_VERSION=20
```

**Cách 3 — netlify.toml:**
```toml
[build.environment]
  NODE_VERSION = "20"
```

### 6.2. Cache dependencies

Netlify tự cache `node_modules` giữa các lần build — build lần 2+ sẽ nhanh hơn. Không cần làm gì thêm.

### 6.3. Build plugins

**Site settings** → **Build & deploy** → **Plugins** — một số plugin hữu ích:
- **Lighthouse** — chấm điểm hiệu suất mỗi deploy
- **Sitemap** — tự tạo sitemap.xml

### 6.4. Xem log build

Mỗi deploy có log chi tiết: **Deploys** → bấm vào deploy → **Deploy log**. Khi build lỗi, đọc log từ dưới lên để tìm nguyên nhân.

---

## 7. Deploy Preview & Branch Deploy

### 7.1. Deploy Preview (cho Pull Request)

Mỗi khi bạn mở **Pull Request** trên GitHub, Netlify tự tạo 1 URL preview riêng:

```
https://deploy-preview-42--ten-site.netlify.app
```

→ Reviewer bấm vào xem thử thay đổi **trước khi merge**. Cực kỳ hữu ích khi làm nhóm!

### 7.2. Branch Deploy

Mỗi branch (ngoài `main`) cũng được deploy tự động:

```
https://ten-branch--ten-site.netlify.app
```

Bật/tắt ở: **Site settings** → **Build & deploy** → **Branch deploys**

### 7.3. Deploy notification

**Site settings** → **Build & deploy** → **Deploy notifications** — nhận thông báo qua:
- Email
- Slack
- Webhook tùy chỉnh

---

## 8. Custom Domain

### 8.1. Thêm domain riêng

1. Mua domain ở nhà cung cấp (Porkbun, Cloudflare, Namecheap...)
2. Netlify → **Site settings** → **Domain management** → **Add custom domain**
3. Nhập domain: `vidu.com`
4. Netlify hiện hướng dẫn trỏ DNS

### 8.2. Cấu hình DNS

**Cách A — dùng Netlify DNS (khuyên dùng):**
- Đổi nameserver ở nhà cung cấp domain thành 4 nameserver của Netlify
- Netlify tự quản lý DNS, HTTPS tự động

**Cách B — trỏ thủ công:**
| Loại | Host | Giá trị |
|------|------|---------|
| A | `@` | `75.2.60.5` |
| CNAME | `www` | `ten-site.netlify.app` |

> ⚠️ IP `75.2.60.5` là IP load balancer của Netlify — kiểm tra lại docs mới nhất vì có thể thay đổi.

### 8.3. Subdomain

Thêm `blog.vidu.com`, `app.vidu.com` — mỗi subdomain có thể trỏ về 1 site Netlify khác nhau.

---

## 9. HTTPS Miễn Phí

Netlify tự cấp chứng chỉ **Let's Encrypt** miễn phí cho mọi domain:

1. Thêm custom domain (mục 8)
2. Đợi DNS propagate (vài phút đến vài giờ)
3. Netlify tự động cấp SSL → bấm vào site đã có `https://`
4. Bật **Force HTTPS**: tự redirect http → https

Kiểm tra: **Site settings** → **Domain management** → **HTTPS** → phải hiện "Certificate active".

---

## 10. Biến Môi Trường

### 10.1. Thêm biến môi trường

**Site settings** → **Environment variables** → **Add a variable**:

| Key | Value | Scope |
|-----|-------|-------|
| `VITE_API_URL` | `https://api.vidu.com` | Production |
| `API_SECRET` | `abc123...` | Production |

> ⚠️ Biến dùng ở **frontend** (Vite: `VITE_*`, CRA: `REACT_APP_*`) sẽ lộ trong code JS — đừng để secret ở đó! Secret chỉ dùng trong Functions.

### 10.2. Dùng trong code

```javascript
// Vite
const apiUrl = import.meta.env.VITE_API_URL;

// Create React App
const apiUrl = process.env.REACT_APP_API_URL;
```

### 10.3. Khai báo trong netlify.toml

```toml
[context.production.environment]
  VITE_API_URL = "https://api.vidu.com"

[context.deploy-preview.environment]
  VITE_API_URL = "https://staging-api.vidu.com"

[context.branch-deploy.environment]
  VITE_API_URL = "https://dev-api.vidu.com"
```

### 10.4. Biến môi trường cho build

```toml
[build.environment]
  NODE_VERSION = "20"
```

---

## 11. Redirects & Rewrites

### 11.1. File `_redirects` (đơn giản)

Tạo file `_redirects` trong thư mục publish:

```
# Redirect cũ sang mới
/blog-cu/*  /blog-moi/:splat  301

# SPA fallback (React Router, Vue Router)
# PHẢI CÓ nếu dùng client-side routing!
/*  /index.html  200

# Redirect theo điều kiện
/admin/*  /dang-nhap  302  Role=admin
```

### 11.2. Khai báo trong netlify.toml (mạnh hơn)

```toml
# SPA fallback
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

# Redirect 301
[[redirects]]
  from = "/cu"
  to = "/moi"
  status = 301

# Proxy API (tránh CORS!)
[[redirects]]
  from = "/api/*"
  to = "https://api.backend.com/:splat"
  status = 200
  force = true

# Redirect theo quốc gia
[[redirects]]
  from = "/"
  to = "/vi/"
  status = 302
  conditions = { Country = ["vn"] }
```

### 11.3. Proxy API — mẹo hay nhất

Frontend gọi `/api/users` → Netlify proxy sang backend thật → **không lo CORS**, không lộ URL backend:

```toml
[[redirects]]
  from = "/api/*"
  to = "https://backend-cua-ban.com/:splat"
  status = 200
```

```javascript
// Frontend chỉ cần gọi relative path
fetch('/api/users').then(r => r.json())
```

---

## 12. Headers Tùy Chỉnh

### 12.1. File `_headers`

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin

/static/*
  Cache-Control: public, max-age=31536000, immutable

/*.html
  Cache-Control: public, max-age=0, must-revalidate
```

### 12.2. Trong netlify.toml

```toml
[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    Referrer-Policy = "strict-origin-when-cross-origin"

[[headers]]
  for = "/static/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"
```

### 12.3. Bảo mật cơ bản nên có

| Header | Tác dụng |
|--------|----------|
| `X-Frame-Options: DENY` | Chống clickjacking |
| `X-Content-Type-Options: nosniff` | Chống MIME sniffing |
| `Referrer-Policy` | Kiểm soát referrer gửi đi |
| `Content-Security-Policy` | Giới hạn nguồn tài nguyên (nâng cao) |

---

## 13. Netlify Forms — Nhận Form Không Cần Backend

### 13.1. Cách dùng (chỉ 1 thuộc tính!)

Thêm `netlify` vào thẻ `<form>`:

```html
<form name="lien-he" method="POST" netlify>
  <input type="text" name="ten" placeholder="Tên của bạn" required />
  <input type="email" name="email" placeholder="Email" required />
  <textarea name="loi-nhan" placeholder="Lời nhắn"></textarea>
  <button type="submit">Gửi</button>
</form>
```

Khi deploy, Netlify tự phát hiện form → submissions hiện ở **Forms** trong dashboard. Nhận thông báo qua email!

### 13.2. Chống spam

```html
<form name="lien-he" method="POST" netlify netlify-honeypot="bot-field">
  <input type="hidden" name="form-name" value="lien-he" />
  <p style="display:none">
    <input name="bot-field" />  <!-- bẫy bot -->
  </p>
  <!-- các field thật -->
</form>
```

Thêm reCAPTCHA:

```html
<form name="lien-he" method="POST" netlify netlify-recaptcha>
  <!-- fields -->
  <div netlify-recaptcha></div>
  <button type="submit">Gửi</button>
</form>
```

### 13.3. Trang cảm ơn tùy chỉnh

```html
<form name="lien-he" method="POST" netlify action="/cam-on">
  <!-- fields -->
</form>
```

Tạo trang `/cam-on.html` → sau khi submit, chuyển đến đó.

### 13.4. Giới hạn free

- 100 submissions/tháng (free)
- Vượt → nâng cấp hoặc dùng Formspree/Getform

---

## 14. Serverless Functions

### 14.1. Tạo function đầu tiên

```
my-site/
├── netlify/
│   └── functions/
│       └── hello.js
└── netlify.toml
```

```javascript
// netlify/functions/hello.js
exports.handler = async (event, context) => {
  const ten = event.queryStringParameters.ten || "bạn";
  return {
    statusCode: 200,
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ message: `Xin chào ${ten}!` }),
  };
};
```

Gọi: `https://ten-site.netlify.app/.netlify/functions/hello?ten=Dương`

### 14.2. Function với POST + JSON

```javascript
// netlify/functions/tinh-tong.js
exports.handler = async (event) => {
  if (event.httpMethod !== "POST") {
    return { statusCode: 405, body: "Chỉ nhận POST" };
  }
  const { a, b } = JSON.parse(event.body);
  return {
    statusCode: 200,
    body: JSON.stringify({ tong: a + b }),
  };
};
```

```javascript
// Frontend gọi
const res = await fetch('/.netlify/functions/tinh-tong', {
  method: 'POST',
  body: JSON.stringify({ a: 5, b: 3 }),
});
const { tong } = await res.json();  // 8
```

### 14.3. Dùng biến môi trường trong function

```javascript
// netlify/functions/weather.js
exports.handler = async (event) => {
  const API_KEY = process.env.WEATHER_API_KEY;  // secret, an toàn!
  const city = event.queryStringParameters.city || "Hanoi";
  const res = await fetch(
    `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric`
  );
  const data = await res.json();
  return {
    statusCode: 200,
    body: JSON.stringify({
      city: data.name,
      nhiet_do: data.main.temp,
      mo_ta: data.weather[0].description,
    }),
  };
};
```

### 14.4. Cấu hình functions

```toml
[functions]
  directory = "netlify/functions"
  node_bundler = "esbuild"
```

### 14.5. Giới hạn free

- 125,000 invocations/tháng
- 100 giờ chạy/tháng
- Timeout 10 giây (free), 26 giây (Pro)

---

## 15. Edge Functions

Edge Functions chạy ở **CDN edge gần người dùng nhất** — nhanh hơn serverless thường.

```typescript
// netlify/edge-functions/chao.ts
export default async (request: Request) => {
  const url = new URL(request.url);
  const ten = url.searchParams.get("ten") || "bạn";
  return new Response(`Xin chào ${ten} từ edge!`, {
    headers: { "content-type": "text/plain" },
  });
};

export const config = { path: "/chao" };
```

Gọi: `https://ten-site.netlify.app/chao?ten=Dương`

**Khi nào dùng Edge vs Serverless:**

| | Serverless Functions | Edge Functions |
|---|---|---|
| Chạy ở đâu | 1 region (US) | Edge gần user nhất |
| Độ trễ | Cao hơn | Thấp nhất |
| Runtime | Node.js | Deno |
| Timeout | 10-26s | 50ms CPU (free) |
| Dùng cho | API, xử lý nặng | Redirect, A/B test, auth check |

---

## 16. Netlify CLI

### 16.1. Cài đặt & đăng nhập

```bash
npm install -g netlify-cli
netlify login
```

### 16.2. Lệnh hay dùng

```bash
# Khởi tạo site mới
netlify init

# Deploy thử (draft)
netlify deploy

# Deploy production
netlify deploy --prod

# Mở site trên trình duyệt
netlify open

# Xem log functions
netlify functions:list
netlify functions:invoke ten-function

# Quản lý biến môi trường
netlify env:list
netlify env:set TEN_BIEN "gia-tri"

# Chạy dev server local (giả lập Netlify)
netlify dev
```

### 16.3. `netlify dev` — dev local như production

```bash
netlify dev
```

Lệnh này chạy web của bạn ở `localhost:8888` **kèm** redirects, functions, env vars — giống hệt production. Không còn cảnh "local chạy được, deploy lỗi"!

---

## 17. Rollback & Deploy History

### 17.1. Xem lịch sử deploy

**Deploys** → danh sách mọi lần deploy, kèm:
- Commit message
- Thời gian
- Trạng thái (thành công/thất bại)
- Deploy preview URL

### 17.2. Rollback (quay về bản cũ)

Deploy mới bị lỗi? 1 click để quay lại:

1. **Deploys** → tìm bản cũ ổn định
2. Bấm `⋮` → **Publish deploy**
3. Xong! Site quay về bản cũ ngay lập tức, không cần build lại

### 17.3. Lock deploy

**Deploys** → **Stop auto publishing** — tạm dừng tự động publish khi push code (để test kỹ trước).

---

## 18. Tối Ưu Hiệu Suất

### 18.1. Netlify tự làm cho bạn

- ✅ CDN toàn cầu (site serve từ edge gần user)
- ✅ Nén Brotli/Gzip tự động
- ✅ HTTP/2
- ✅ Cache headers hợp lý cho assets

### 18.2. Bạn nên làm thêm

1. **Tối ưu ảnh:** dùng định dạng WebP/AVIF, lazy loading
2. **Code splitting:** chia bundle JS (Vite/Webpack làm tự động)
3. **Preconnect** tới domain API:
   ```html
   <link rel="preconnect" href="https://api.vidu.com">
   ```
4. **Cache immutable** cho file có hash:
   ```
   /assets/*
     Cache-Control: public, max-age=31536000, immutable
   ```

### 18.3. Đo hiệu suất

Cài plugin **Lighthouse** → mỗi deploy tự chấm điểm Performance/SEO. Hoặc dùng https://pagespeed.web.dev

---

## 19. Pricing

### 19.1. Gói Free ($0)

| Tài nguyên | Giới hạn |
|------------|----------|
| Bandwidth | 100 GB/tháng |
| Build minutes | 300 phút/tháng |
| Sites | Không giới hạn |
| Serverless invocations | 125,000/tháng |
| Forms submissions | 100/tháng |
| Team members | 1 |

**Đủ cho:** blog cá nhân, portfolio, landing page, side project.

### 19.2. Gói Pro ($19/thành viên/tháng)

- Bandwidth 1TB, build 25,000 phút
- Analytics, password protection
- Background functions (timeout 15 phút)

### 19.3. Khi nào hết free?

100GB bandwidth ≈ **1 triệu pageview** web nhẹ. Hầu hết site cá nhân không bao giờ hết. Nếu hết, Netlify báo trước — không tự trừ tiền.

---

## 20. Troubleshooting

| Vấn đề | Nguyên nhân | Cách khắc phục |
|--------|-------------|----------------|
| Build failed | Lệnh build sai / thiếu dependency | Đọc deploy log từ dưới lên; chạy `npm run build` ở local trước |
| Page not found khi refresh | SPA thiếu redirect | Thêm `/* /index.html 200` vào `_redirects` |
| 404 sau deploy | Sai publish directory | Kiểm tra bảng mục 5.3 |
| Biến môi trường không có tác dụng | Quên rebuild sau khi thêm | Trigger deploy mới (Clear cache and deploy) |
| Custom domain không vào được | DNS chưa propagate | Đợi 1-24h; kiểm tra bằng `nslookup` |
| HTTPS lỗi | DNS chưa đúng | Đảm bảo A/CNAME trỏ đúng, đợi propagate rồi mới cấp cert |
| Form không nhận submission | Thiếu `netlify` hoặc `name` | Kiểm tra thẻ form có đủ thuộc tính |
| Function 502 | Lỗi code / timeout | Xem function log; tăng timeout nếu cần |
| Deploy preview không tạo | PR từ fork chưa bật | Site settings → cho phép fork PR |

### Clear cache and deploy

Khi nghi cache cũ gây lỗi: **Deploys** → **Trigger deploy** → **Clear cache and deploy site**.

---

## 21. Best Practices & Cheat Sheet

### 21.1. Best practices

1. **Dùng `netlify.toml`** thay vì cài đặt trên web — version control được
2. **SPA luôn thêm** `/* /index.html 200` redirect
3. **Secret** chỉ để trong Functions, không để ở biến `VITE_*`/`REACT_APP_*`
4. **Test bằng Deploy Preview** trước khi merge PR
5. **Đặt `NODE_VERSION`** cố định để build ổn định
6. **Dùng proxy `/api/*`** thay vì gọi backend trực tiếp (tránh CORS)

### 21.2. Cheat sheet

```bash
# CLI
npm i -g netlify-cli
netlify login
netlify init
netlify deploy --prod
netlify dev                    # dev local như production
netlify env:set KEY "value"
netlify open
```

```toml
# netlify.toml mẫu đầy đủ
[build]
  command = "npm run build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "20"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[redirects]]
  from = "/api/*"
  to = "https://backend.com/:splat"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"

[functions]
  directory = "netlify/functions"
```

```
# _redirects mẫu
/*  /index.html  200
/blog-cu/*  /blog-moi/:splat  301
/api/*  https://backend.com/:splat  200
```

### 21.3. Tài liệu tham khảo

- Docs: https://docs.netlify.com
- Blog: https://www.netlify.com/blog
- Templates: https://www.netlify.com/templates
- Cộng đồng: https://answers.netlify.com

---

*Cập nhật: 10/10/2026*

---

## 22. Deploy Từng Framework Chi Tiết

### 22.1. Next.js

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = ".next"

[[plugins]]
  package = "@netlify/plugin-nextjs"
```

Cài plugin:
```bash
npm install -D @netlify/plugin-nextjs
```

> Lưu ý: Next.js trên Netlify chạy tốt nhất ở chế độ **static export** hoặc dùng plugin trên. Nếu cần SSR phức tạp, Vercel tối ưu hơn.

### 22.2. Astro (rất hợp Netlify)

```bash
npm create astro@latest
# Chọn template, xong push lên GitHub là Netlify tự nhận
```

```toml
[build]
  command = "npm run build"
  publish = "dist"
```

### 22.3. Hugo (blog tĩnh siêu nhanh)

```toml
[build]
  command = "hugo --gc --minify"
  publish = "public"

[build.environment]
  HUGO_VERSION = "0.121.0"
```

### 22.4. Monorepo (nhiều app trong 1 repo)

```
repo/
├── apps/
│   ├── web/        ← deploy site 1
│   └── admin/      ← deploy site 2
└── packages/
    └── ui/
```

Tạo **2 site** trên Netlify, mỗi site trỏ cùng repo nhưng khác **Base directory**:
- Site 1: Base = `apps/web`, Publish = `apps/web/dist`
- Site 2: Base = `apps/admin`, Publish = `apps/admin/dist`

---

## 23. Netlify Identity — Auth Có Sẵn

Netlify Identity cho phép thêm đăng nhập/đăng ký vào web tĩnh **không cần backend**.

### 23.1. Bật Identity

**Site settings** → **Identity** → **Enable Identity**

### 23.2. Thêm widget đăng nhập

```html
<script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>

<div data-netlify-identity-menu></div>       <!-- menu login/logout -->
<div data-netlify-identity-button>Đăng nhập</div>

<script>
  netlifyIdentity.on('login', user => {
    console.log('Đã đăng nhập:', user.email);
  });
  netlifyIdentity.on('logout', () => {
    console.log('Đã đăng xuất');
  });
</script>
```

### 23.3. Bảo vệ route bằng role

```toml
# Chỉ user đăng nhập mới vào được /admin
[[redirects]]
  from = "/admin/*"
  to = "/admin/index.html"
  status = 200
  conditions = { Role = ["admin"] }
```

Gán role trong dashboard: **Identity** → chọn user → thêm role `admin`.

### 23.4. Giới hạn free

- 1,000 active users/tháng (free)
- Vượt → $99/tháng cho 5,000 users

---

## 24. Split Testing (A/B Test)

Chia traffic giữa 2 branch để test:

1. **Site settings** → **Split Testing** → **Start new test**
2. Chọn branch A (production) và branch B (thử nghiệm)
3. Đặt tỷ lệ: ví dụ 90% → A, 10% → B
4. Netlify tự chia traffic, đo conversion

---

## 25. Large Media (Quản Lý File Lớn)

Netlify Large Media dùng Git LFS để quản lý ảnh/video lớn:

```bash
# Cài plugin
netlify plugins:install netlify-lm-plugin
netlify lm:install

# Track file lớn
git lfs track "*.mp4" "*.psd"
git add .gitattributes
```

Ảnh tự động optimize, resize theo query params:
```
https://ten-site.netlify.app/anh.jpg?nf_resize=fit&w=800
```

---

## 26. So Sánh Chi Tiết Netlify vs Đối Thủ

### Netlify vs Vercel (kỹ hơn)

| | Netlify | Vercel |
|---|---|---|
| Build free | 300 phút | 6000 phút |
| Functions free | 125k invocations | 1M invocations (hobby) |
| Edge | Có (Deno) | Mạnh hơn (Vercel Edge) |
| Next.js | Chạy được (qua plugin) | **Native, tốt nhất** |
| Forms | ✅ Có sẵn | ❌ |
| Identity/Auth | ✅ Có sẵn | Phải tự làm (Auth.js) |
| Giá Pro | $19/user | $20/user |

### Netlify vs Cloudflare Pages

| | Netlify | Cloudflare Pages |
|---|---|---|
| Bandwidth free | 100GB | **Không giới hạn** |
| Build free | 300 phút | 500 builds/tháng |
| Functions | AWS Lambda | Cloudflare Workers |
| Forms | ✅ | ❌ |
| Dễ dùng | Rất dễ | Dễ |

**Kết luận nhanh:** Blog/portfolio/landing → Netlify hoặc Cloudflare Pages đều tốt. Next.js nặng → Vercel.

---

## 27. Ví Dụ Thực Tế: Deploy Blog Astro Trong 5 Phút

```bash
# 1. Tạo project
npm create astro@latest blog-cua-toi
cd blog-cua-toi
npm install

# 2. Chạy thử local
npm run dev   # mở http://localhost:4321

# 3. Push lên GitHub
git init
git add .
git commit -m "Blog đầu tiên"
gh repo create blog-cua-toi --public --source=. --push

# 4. Lên Netlify
# - Add new site → Import from GitHub → chọn repo
# - Build command: npm run build (tự điền)
# - Publish: dist (tự điền)
# - Deploy!

# 5. Mỗi lần viết bài mới:
# - Thêm file .md vào src/content/
# - git add, commit, push → Netlify tự rebuild (2 phút)
```

---

## 28. Câu Hỏi Thường Gặp (FAQ)

**Netlify có thật sự miễn phí không?**
Có. Gói free đủ cho hầu hết site cá nhân. Netlify chỉ tính tiền khi vượt giới hạn (và báo trước).

**Tôi có cần thẻ tín dụng để dùng free không?**
Không. Đăng ký là dùng ngay.

**Site .netlify.app có bị Google phạt SEO không?**
Không. Nhưng nên dùng custom domain cho chuyên nghiệp.

**Deploy private repo được không?**
Được. Netlify đọc được cả repo private (khi bạn cấp quyền GitHub).

**Xóa site như thế nào?**
Site settings → General → Danger zone → Delete this site.

**Netlify có hỗ trợ backend như Express không?**
Không trực tiếp. Dùng Serverless Functions cho API nhẹ, hoặc deploy backend riêng (Render, Railway, Fly.io) rồi proxy qua `/api/*`.

**Build quá 15 phút bị timeout?**
Gói free giới hạn 15 phút/build. Tối ưu: giảm dependencies, dùng cache, chia nhỏ repo.

---

*Cập nhật: 10/10/2026 — Tổng 28 phần*

---

## 29. Deploy Notifications Chi Tiết

### 29.1. Thông báo qua Slack

1. **Site settings** → **Build & deploy** → **Deploy notifications** → **Add notification** → **Slack**
2. Chọn sự kiện:
   - `Deploy succeeded` — deploy thành công
   - `Deploy failed` — deploy lỗi (quan trọng nhất!)
   - `Deploy preview ready` — preview PR xong
3. Dán **Incoming Webhook URL** của Slack
4. Từ nay deploy lỗi → Slack báo ngay

### 29.2. Thông báo qua Email

Mặc định Netlify đã gửi email khi deploy fail cho owner. Thêm người nhận:
**Team settings** → **Notifications**.

### 29.3. Webhook tùy chỉnh

```toml
# Gọi webhook sau mỗi deploy (để trigger task khác)
# Cài trong dashboard: Deploy notifications → Outgoing webhook
```

Ví dụ payload Netlify gửi:
```json
{
  "name": "my-site",
  "state": "ready",
  "branch": "main",
  "commit_url": "https://github.com/user/repo/commit/abc123"
}
```

---

## 30. Netlify Analytics

Không cần chèn script tracking — Netlify đo từ server:

**Site settings** → **Analytics** → **Enable** ($9/tháng/site, dùng thử 30 ngày free)

Đo được:
- Pageview thật (không bị adblock chặn như Google Analytics)
- Nguồn traffic, quốc gia
- Top pages, bandwidth

> 💡 Nếu muốn free: dùng **Cloudflare Web Analytics** (free, chèn 1 script) hoặc **Plausible** self-host.

---

## 31. Password Protection (Gói Pro)

Bảo vệ site staging bằng mật khẩu:

**Site settings** → **Access control** → **Password protection** → đặt mật khẩu.

Hoặc bảo vệ theo branch trong `netlify.toml`:
```toml
[context.staging]
  # staging branch yêu cầu password (cài trong dashboard)
```

---

## 32. Prerendering Cho SPA (SEO)

Google bot đôi khi không render JS tốt. Bật prerendering:

**Site settings** → **Build & deploy** → **Prerendering** → **Enable**

Netlify sẽ serve bản HTML render sẵn cho bot. (Dịch vụ beta, có thể thay đổi.)

Giải pháp tốt hơn cho SEO: dùng **SSG** (Astro, Hugo, Next.js static export) — HTML có sẵn từ lúc build.

---

## 33. Background Functions (Gói Pro)

Functions thường timeout 10s. Task lâu (xử lý video, gửi 1000 email) → dùng Background Functions:

```javascript
// netlify/functions/xu-ly-lau.js
exports.handler = async (event) => {
  // Chạy nền, trả về 202 ngay
  await xuLyNang(event.body);  // mất 5 phút cũng được
  return { statusCode: 202 };
};
```

Đặt file trong `netlify/functions/` với tên kết thúc `-background.js`:
```
netlify/functions/gui-email-background.js
```
Gọi: `/.netlify/functions/gui-email-background`

---

## 34. Migrate Từ Nền Tảng Khác

### Từ Vercel sang Netlify
1. Import repo như bình thường
2. Chuyển `vercel.json` → `netlify.toml`:
   ```toml
   # vercel.json: { "rewrites": [{ "source": "/api/:p", "destination": "..." }] }
   # thành:
   [[redirects]]
     from = "/api/:p"
     to = "https://backend/:p"
     status = 200
   ```
3. Env vars: copy từ Vercel dashboard sang Netlify

### Từ GitHub Pages sang Netlify
1. Import repo
2. Build command để trống (nếu đã có HTML sẵn), publish = `.` hoặc thư mục docs
3. Lợi ngay: preview deploy, forms, functions, redirect dễ

### Từ Heroku (frontend) sang Netlify
Heroku khai tử free tier từ 2022 — Netlify là nơi đến lý tưởng cho frontend. Backend chuyển sang Render/Railway/Fly.io.

---

## 35. Checklist Trước Khi Production

- [ ] `netlify.toml` đã commit vào repo
- [ ] `NODE_VERSION` cố định
- [ ] SPA redirect `/* /index.html 200` đã có
- [ ] Security headers đã thêm
- [ ] Biến môi trường production đã set
- [ ] Custom domain + HTTPS active + Force HTTPS
- [ ] Deploy preview test 1 PR thử
- [ ] Lighthouse score > 90
- [ ] Form test thử submit
- [ ] 404 page tùy chỉnh (`404.html`)

### Tạo trang 404 đẹp

Tạo file `404.html` trong publish directory — Netlify tự serve khi không tìm thấy trang.

---

*Cập nhật: 10/10/2026 — Tổng 35 phần*

---

## 36. Làm Việc Nhóm

### 36.1. Thêm thành viên

**Team settings** → **Members** → **Invite** — nhập email.

Phân quyền:
| Role | Quyền |
|------|-------|
| Owner | Mọi thứ, thanh toán |
| Collaborator | Deploy, cài đặt site |
| Viewer | Chỉ xem (Pro) |

> ⚠️ Gói Pro tính tiền **theo số thành viên** ($19/người). Gói free chỉ 1 member.

### 36.2. Quy trình nhóm chuẩn

```
1. Dev tạo branch feature/login
2. Push → Netlify tạo Branch Deploy: https://feature-login--site.netlify.app
3. Mở PR → Deploy Preview: https://deploy-preview-15--site.netlify.app
4. Reviewer test trên preview URL, comment
5. Merge → auto deploy production
6. Lỗi? Rollback 1 click (mục 17)
```

### 36.3. Protected deploys

**Site settings** → **Build & deploy** → **Deploy contexts** → yêu cầu review trước khi deploy production (gói Pro).

---

## 37. Netlify API — Tự Động Hóa

Mọi thứ làm trên dashboard đều gọi được qua API.

```bash
# Lấy token: User settings → Applications → New access token
export NETLIFY_TOKEN="nfp_xxx"

# Liệt kê sites
curl -H "Authorization: Bearer $NETLIFY_TOKEN" \
  https://api.netlify.com/api/v1/sites | jq '.[].name'

# Trigger deploy mới (build hook)
curl -X POST "https://api.netlify.com/build_hooks/xxx"

# Liệt kê deploys của 1 site
curl -H "Authorization: Bearer $NETLIFY_TOKEN" \
  https://api.netlify.com/api/v1/sites/SITE_ID/deploys | jq '.[0].state'
```

### Build hooks — rebuild theo lịch

**Site settings** → **Build & deploy** → **Build hooks** → tạo hook → dùng với cron:

```bash
# Rebuild blog mỗi sáng 6h (để cập nhật nội dung từ API)
0 6 * * * curl -X POST https://api.netlify.com/build_hooks/xxx
```

---

## 38. Snippet Theo Framework

### React + Vite + React Router

```toml
[build]
  command = "npm run build"
  publish = "dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Nuxt 3 (static)

```toml
[build]
  command = "npm run generate"
  publish = ".output/public"
```

### SvelteKit (static adapter)

```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-static';
export default { kit: { adapter: adapter() } };
```

```toml
[build]
  command = "npm run build"
  publish = "build"
```

### Docusaurus (docs)

```toml
[build]
  command = "npm run build"
  publish = "build"

[[redirects]]
  from = "/docs"
  to = "/docs/intro"
  status = 302
```

---

## 39. So Sánh Tốc Độ Build

| Framework | Build time điển hình | Mẹo tăng tốc |
|-----------|---------------------|--------------|
| HTML thuần | < 30s | Không cần build |
| Vite React | 1-3 phút | Dùng `npm ci`, cache tốt |
| Next.js | 3-8 phút | Giảm page, dùng ISR |
| Gatsby | 5-15 phút | Chuyển sang Astro! |
| Hugo | < 1 phút | Nhanh nhất trong SSG |

---

*Cập nhật: 10/10/2026 — Tổng 39 phần*

---

## 40. Xử Lý Ảnh/Asset Tối Ưu

### Netlify Image CDN (mới)

Resize ảnh động qua URL:

```
https://ten-site.netlify.app/.netlify/images?url=/anh-goc.jpg&w=800&q=80
```

Tham số:
| Param | Ý nghĩa |
|-------|---------|
| `w` | Chiều rộng (px) |
| `h` | Chiều cao (px) |
| `q` | Chất lượng 1-100 |
| `fm` | Định dạng: `webp`, `avif` |

```html
<!-- Ảnh responsive tự động -->
<img
  src="/.netlify/images?url=/anh.jpg&w=800&fm=webp&q=80"
  srcset="/.netlify/images?url=/anh.jpg&w=400&fm=webp 400w,
          /.netlify/images?url=/anh.jpg&w=800&fm=webp 800w,
          /.netlify/images?url=/anh.jpg&w=1200&fm=webp 1200w"
  alt="Mô tả ảnh"
  loading="lazy"
/>
```

---

## 41. Tóm Tắt Lộ Trình Học Netlify

```
Tuần 1: Deploy web tĩnh bằng kéo-thả → đổi tên site → thêm custom domain
Tuần 2: Kết nối GitHub → deploy React/Vite → thêm _redirects cho SPA
Tuần 3: Thử Netlify Forms (form liên hệ) → viết 1 serverless function
Tuần 4: Netlify CLI + netlify dev → Deploy Preview workflow với PR
Nâng cao: Edge Functions, Identity, Split Testing, API automation
```

**Sau 1 tháng:** bạn deploy web chuyên nghiệp như senior — push code là lên mạng, có preview, rollback, HTTPS, domain riêng.

---

*Cập nhật: 10/10/2026 — Tổng 41 phần — Hết! 🎉*
