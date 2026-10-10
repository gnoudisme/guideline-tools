# Hướng Dẫn Toàn Tập LinkedIn — Từ Profile Đến API Cho Developer

> **Cấp độ:** Người mới → xây thương hiệu cá nhân & dùng LinkedIn API
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10
>
> ⚠️ LinkedIn là mạng xã hội nghề nghiệp lớn nhất thế giới (1 tỷ+ thành viên). Bài này bao phủ cả dùng cá nhân lẫn lập trình với LinkedIn API.

---

## Mục Lục

**Phần A — Dùng LinkedIn cá nhân:**
1. [LinkedIn Là Gì?](#1-linkedin-là-gì)
2. [Tạo Tài Khoản](#2-tạo-tài-khoản)
3. [Tối Ưu Profile — Checklist Chi Tiết](#3-tối-ưu-profile--checklist-chi-tiết)
4. [Viết Headline & About Ấn Tượng](#4-viết-headline--about-ấn-tượng)
5. [Kinh Nghiệm & Kỹ Năng (Skills)](#5-kinh-nghiệm--kỹ-năng-skills)
6. [Kết Nối & Mở Rộng Network](#6-kết-nối--mở-rộng-network)
7. [Tìm Việc Trên LinkedIn](#7-tìm-việc-trên-linkedin)
8. [Content Strategy — Đăng Gì Để Nổi Bật](#8-content-strategy--đăng-gì-để-nổi-bật)
9. [LinkedIn Cho Freelancer](#9-linkedin-cho-freelancer)
10. [Company Page — Trang Công Ty](#10-company-page--trang-công-ty)
11. [LinkedIn Premium — Có Đáng Tiền?](#11-linkedin-premium--có-đáng-tiền)

**Phần B — Cho developer:**
12. [Tổng Quan LinkedIn API](#12-tổng-quan-linkedin-api)
13. [Tạo App & Lấy Access Token (OAuth 2.0)](#13-tạo-app--lấy-access-token-oauth-20)
14. [API Lấy Thông Tin Profile](#14-api-lấy-thông-tin-profile)
15. [API Đăng Bài (Share API)](#15-api-đăng-bài-share-api)
16. [Ví Dụ Code Hoàn Chỉnh (Node.js + Python)](#16-ví-dụ-code-hoàn-chỉnh-nodejs--python)
17. [Giới Hạn & Lưu Ý API](#17-giới-hạn--lưu-ý-api)

**Phần C:**
18. [Bảo Mật & Quyền Riêng Tư](#18-bảo-mật--quyền-riêng-tư)
19. [Troubleshooting](#19-troubleshooting)
20. [Best Practices & Cheat Sheet](#20-best-practices--cheat-sheet)

---

## PHẦN A — DÙNG LINKEDIN CÁ NHÂN

---

## 1. LinkedIn Là Gì?

### 1.1. Định nghĩa

**LinkedIn** là mạng xã hội **nghề nghiệp** lớn nhất thế giới với hơn 1 tỷ thành viên (2024). Khác với Facebook/TikTok (giải trí), LinkedIn tập trung vào:

- **CV online:** Profile = CV sống, nhà tuyển dụng tìm thấy bạn
- **Networking:** Kết nối đồng nghiệp, mentor, đối tác
- **Tìm việc:** Hàng triệu tin tuyển dụng, apply trực tiếp
- **Học hỏi:** Bài viết chuyên môn, khóa học LinkedIn Learning
- **Thương hiệu cá nhân:** Chia sẻ kiến thức → được biết đến trong ngành

### 1.2. Ai nên dùng?

| Đối tượng | Dùng để làm gì |
|---|---|
| Sinh viên / mới ra trường | Tìm internship, job đầu tiên, học hỏi |
| Developer | Khoe project, tìm job remote, networking |
| Freelancer | Tìm khách hàng, xây uy tín |
| Founder | Tìm co-founder, nhà đầu tư, khách hàng |
| Recruiter/HR | Tìm ứng viên |

### 1.3. LinkedIn ở Việt Nam

- Cộng đồng tech VN trên LinkedIn rất active (dev, PM, founder)
- Nhiều công ty VN và quốc tế tuyển qua LinkedIn
- Viết tiếng Anh → tiếp cận job global/remote; viết tiếng Việt → tiếp cận cộng đồng local

---

## 2. Tạo Tài Khoản

1. Vào https://www.linkedin.com → **Join now**
2. Nhập email + password (hoặc đăng nhập bằng Google)
3. Điền: họ tên, vị trí hiện tại (ví dụ: "Frontend Developer tại Công ty ABC")
4. Xác thực email
5. **Tạo URL cá nhân:** Profile → **Edit public profile & URL** → đổi thành `linkedin.com/in/ten-cua-ban` (ngắn gọn, chuyên nghiệp)

> 💡 URL đẹp dùng để gắn vào CV, portfolio, chữ ký email.

---

## 3. Tối Ưu Profile — Checklist Chi Tiết

Profile đầy đủ = xuất hiện nhiều hơn trong tìm kiếm của recruiter. Checklist từng phần:

### 3.1. Ảnh đại diện

- ✅ Ảnh chân dung rõ mặt, nền đơn giản
- ✅ Nhìn thẳng camera, cười nhẹ
- ✅ Kích thước: 400x400px trở lên
- ❌ Không dùng ảnh du lịch, ảnh nhóm, avatar hoạt hình (trừ khi bạn làm creative)

### 3.2. Ảnh bìa (Background)

- Kích thước: 1584x396px
- Dùng để thể hiện cá tính: project bạn làm, tech stack, câu slogan
- Tool miễn phí: Canva (có template LinkedIn banner sẵn)

### 3.3. Các phần bắt buộc

| Phần | Yêu cầu |
|---|---|
| Headline | Xem mục 4 — quan trọng nhất sau tên |
| About | 2-3 đoạn, xem mục 4 |
| Experience | Mỗi job: tên công ty, chức danh, thời gian, 3-5 gạch đầu dòng thành tích |
| Education | Trường, ngành, năm tốt nghiệp |
| Skills | Ít nhất 10 skills liên quan (mục 5) |
| Featured | Ghim 3-5 thứ nổi bật: project, bài viết, link portfolio |

### 3.4. Open To Work

- Bật **Open to Work** (khung xanh quanh avatar) khi đang tìm việc
- Cài đặt: loại job mong muốn, địa điểm (có thể chọn Remote), ai được thấy (recruiter only hoặc tất cả)

---

## 4. Viết Headline & About Ấn Tượng

### 4.1. Headline (dòng dưới tên)

Có 220 ký tự — đừng chỉ ghi chức danh!

❌ **Dở:** `Nhân viên IT tại Công ty ABC`

✅ **Hay:** Công thức = Vai trò + Chuyên môn + Giá trị

```
Frontend Developer | React & TypeScript | Xây UI cho 1M+ users | Open to remote
```

```
Backend Developer (Node.js/PostgreSQL) | Từng scale hệ thống 100k req/s | Viết về system design
```

```
Sinh viên năm 3 CNTT | Yêu thích AI/ML | Tìm internship Summer 2027
```

### 4.2. About (giới thiệu)

Cấu trúc 3 đoạn:

**Đoạn 1 — Bạn là ai:**
> Tôi là Frontend Developer với 3 năm kinh nghiệm xây dựng web app bằng React/TypeScript...

**Đoạn 2 — Bạn làm được gì (con số cụ thể):**
> - Tối ưu performance giúp giảm 40% thời gian tải trang
> - Dẫn dắt team 4 người rebuild dashboard cho 50k users
> - Tech stack: React, TypeScript, Next.js, TailwindCSS

**Đoạn 3 — Call to action:**
> 📩 Đang tìm cơ hội remote — inbox tôi nếu bạn cần Frontend dev!
> 🔗 Portfolio: yoursite.com | 📧 email@example.com

> 💡 Dùng **emoji vừa phải** và **gạch đầu dòng** cho dễ đọc. Recruiter chỉ lướt 10 giây!

---

## 5. Kinh Nghiệm & Kỹ Năng (Skills)

### 5.1. Viết Experience đúng cách

Mỗi công việc gồm:

```
Chức danh: Frontend Developer
Công ty: Công ty ABC
Thời gian: 01/2024 – Hiện tại
Địa điểm: Remote

Mô tả (dùng công thức STAR + con số):
• Xây dựng dashboard analytics bằng React + TypeScript cho 50k users
• Tối ưu bundle size từ 2MB → 800KB, giảm 40% thời gian tải
• Viết 200+ unit tests, coverage đạt 85%
• Skills: React · TypeScript · Vite · TailwindCSS
```

> ⚠️ Tránh mô tả chung chung kiểu "Tham gia phát triển dự án". Luôn có **con số** và **kết quả**.

### 5.2. Skills & Endorsements

- Thêm ít nhất **10-15 skills** liên quan đến ngành
- Top 3 skills hiển thị đầu — sắp xếp skill quan trọng nhất lên trước
- Nhờ đồng nghiệp **endorse** (xác nhận) skills cho bạn
- **Skill quiz:** LinkedIn có bài test ngắn cho một số skill (JS, Python...) — pass sẽ có badge

### 5.3. Recommendations

- Xin **recommendation** (lời giới thiệu) từ sếp/đồng nghiệp cũ
- Cách xin lịch sự: "Anh/chị có thể viết vài dòng recommendation giúp em trên LinkedIn được không ạ?"
- 2-3 recommendations chất lượng > 10 cái chung chung

---

## 6. Kết Nối & Mở Rộng Network

### 6.1. Gửi lời mời kết nối

- **Luôn kèm note** (tin nhắn giới thiệu) — tỷ lệ chấp nhận cao gấp 3 lần:

```
Chào anh/chị,
Em là An, Frontend Developer. Em thấy anh/chị chia sẻ nhiều về React
performance rất hay. Em muốn kết nối để học hỏi thêm ạ!
```

- Mỗi tuần gửi 20-30 lời mời có chọn lọc (đừng spam)

### 6.2. Ai nên kết nối?

1. Đồng nghiệp hiện tại & cũ
2. Người cùng ngành (dev, designer, PM)
3. Recruiter trong lĩnh vực của bạn
4. Diễn giả, tác giả bài viết bạn thích
5. Alumni trường bạn

### 6.3. Tương tác để được nhớ đến

- **Like + comment có chiều sâu** vào bài viết của người khác (đừng chỉ "Hay quá!")
- Comment hay = mini-bài viết, nhiều người sẽ vào profile bạn
- Chúc mừng work anniversary, job mới của connections

### 6.4. Follow vs Connect

| | Follow | Connect |
|---|---|---|
| Thấy bài viết của họ | ✅ | ✅ |
| Họ thấy bài của bạn | ❌ | ✅ |
| Nhắn tin | ❌ (trừ Premium) | ✅ |
| Cần họ đồng ý | Không | Có |

> 💡 Với influencer/CEO: **Follow** là đủ. Với đồng nghiệp/recruiter: **Connect**.

---

## 7. Tìm Việc Trên LinkedIn

### 7.1. Tìm kiếm hiệu quả

1. Tab **Jobs** → nhập từ khóa (ví dụ: `Frontend Developer`) + địa điểm (`Vietnam`, `Remote`)
2. **Bộ lọc quan trọng:**
   - Date posted: Past week (job mới)
   - Experience level: Entry/Associate/Mid-Senior
   - Remote: On-site / Hybrid / Remote
   - Easy Apply: nộp nhanh không cần qua web công ty
3. **Job alerts:** Bật thông báo cho từ khóa → LinkedIn gửi job mới mỗi ngày

### 7.2. Tối ưu để recruiter tìm thấy bạn

Recruiter tìm ứng viên bằng từ khóa — profile bạn cần chứa đúng từ khóa:

- Skills đầy đủ (React, Node.js, AWS...)
- Headline có chức danh chuẩn
- About nhắc lại tech stack
- Bật **Open to Work** + điền job preferences

### 7.3. Apply đúng cách

- **Easy Apply:** nhanh nhưng cạnh tranh cao → nên kèm cover note ngắn
- **Apply qua web công ty:** tốn thời gian hơn nhưng nghiêm túc hơn
- **Cách tốt nhất:** Tìm hiring manager/recruiter của công ty đó trên LinkedIn → nhắn tin trực tiếp kèm CV

```
Chào anh/chị,
Em thấy công ty đang tuyển Frontend Developer.
Em có 3 năm kinh nghiệm React/TypeScript, từng làm [dự án].
Em gửi CV đính kèm, mong được trao đổi thêm ạ!
```

### 7.4. Chuẩn bị phỏng vấn

- Xem profile người phỏng vấn trước (biết background họ)
- Xem **Company page** để hiểu văn hóa, sản phẩm
- Đọc review trên các trang đánh giá công ty

---

## 8. Content Strategy — Đăng Gì Để Nổi Bật

### 8.1. Tại sao nên đăng bài?

- Recruiter và khách hàng tìm đến bạn (inbound)
- Xây uy tín chuyên môn
- Học sâu hơn khi phải diễn đạt lại kiến thức

### 8.2. Ý tưởng content cho developer

| Loại | Ví dụ |
|---|---|
| Bài học | "3 điều tôi học được khi optimize React app" |
| So sánh | "REST vs GraphQL: khi nào dùng cái nào?" |
| Behind the scenes | "Cách tôi debug bug mất 3 ngày" |
| Tóm tắt | "Tóm tắt sách Clean Code trong 5 phút" |
| Project | "Tôi vừa build xong app X bằng [stack]" |
| Quan điểm | "Junior dev có nên học AI không?" |

### 8.3. Công thức bài viết viral

1. **Hook (dòng đầu):** gây tò mò — "Tôi đã mất việc vì không biết điều này..."
2. **Xuống dòng nhiều:** mỗi ý 1-2 dòng, dễ đọc trên điện thoại
3. **Kết thúc bằng câu hỏi:** "Bạn nghĩ sao? Comment bên dưới nhé!"
4. **Đăng giờ vàng:** 8-9h sáng hoặc 12h trưa giờ VN (giờ nghỉ)
5. **Tần suất:** 2-3 bài/tuần là đẹp

### 8.4. Định dạng

- **Text post:** reach tốt nhất hiện tại
- **Carousel (PDF nhiều trang):** engagement cao, hợp tutorial
- **Video:** LinkedIn đang đẩy mạnh
- **Poll:** dễ làm, nhiều tương tác

---

## 9. LinkedIn Cho Freelancer

### 9.1. Tối ưu profile freelancer

- Headline: `Freelance [Skill] | Đã làm việc với X+ khách hàng | [Kết quả nổi bật]`
- Featured: ghim portfolio, testimonial khách hàng
- Services: bật **LinkedIn Services** (mục Services) — khách hàng có thể request trực tiếp

### 9.2. Tìm khách hàng

1. Đăng case study: "Cách tôi giúp khách hàng tăng 200% conversion"
2. Kết nối với founder, marketing manager (người ra quyết định thuê)
3. Tham gia group ngành nghề
4. Dùng Sales Navigator (trả phí) để tìm lead chính xác

---

## 10. Company Page — Trang Công Ty

### 10.1. Tạo trang công ty

1. Bấm **For Business** (góc phải) → **Create a Company Page**
2. Điền: tên công ty, logo (300x300), tagline, website, ngành, quy mô
3. Thêm **Life page** (văn hóa công ty) nếu có

### 10.2. Dùng để làm gì?

- Đăng tin tuyển dụng chính thức
- Xây thương hiệu tuyển dụng (employer branding)
- Nhân viên gắn company vào Experience → tự động link về page
- Chạy quảng cáo tuyển dụng / sản phẩm

---

## 11. LinkedIn Premium — Có Đáng Tiền?

| Tính năng | Free | Premium (~$30-60/tháng) |
|---|---|---|
| Xem ai vào profile | 5 người gần nhất | Tất cả |
| InMail | Không | 5-15/tháng (nhắn người lạ) |
| Job insights | Cơ bản | Chi tiết (bạn rank bao nhiêu trong ứng viên) |
| LinkedIn Learning | Không | Full khóa học |
| Salary insights | Không | Có |

**Đáng tiền khi:**
- Đang tích cực tìm việc (1-2 tháng)
- Làm sales/recruit cần InMail

**Không cần khi:** chỉ dùng để networking và đọc bài viết.

> 💡 **Mẹo:** LinkedIn thường cho **dùng thử Premium 1 tháng miễn phí** — nhớ hủy trước khi hết hạn nếu không muốn bị trừ tiền.

---

## PHẦN B — CHO DEVELOPER (LINKEDIN API)

---

## 12. Tổng Quan LinkedIn API

### 12.1. Có những API nào?

| API | Dùng để làm gì | Ai được dùng |
|---|---|---|
| **Share API** | Đăng bài lên LinkedIn hộ user | App đã duyệt |
| **Profile API** (`r_liteprofile`) | Lấy tên, ảnh đại diện user | App đã duyệt |
| **Sign In with LinkedIn** | Đăng nhập bằng LinkedIn (OAuth) | Mở rộng |
| **Marketing API** | Quảng cáo, analytics | Cần partner |
| **Talent API** | Tuyển dụng | Cần partner |

> ⚠️ LinkedIn API **không mở tự do** như Twitter API v1. Hầu hết cần tạo app và xin quyền (products). API công khai bị giới hạn nhiều.

### 12.2. Điều kiện

1. Tài khoản LinkedIn thật
2. Tạo app tại https://developer.linkedin.com/
3. Thêm **Products** cho app (ví dụ: "Share on LinkedIn", "Sign In with LinkedIn")
4. App ở chế độ Development → chỉ test được với chính bạn; muốn public phải qua **review**

---

## 13. Tạo App & Lấy Access Token (OAuth 2.0)

### 13.1. Tạo app

1. Vào https://developer.linkedin.com/ → **Create app**
2. Điền: tên app, LinkedIn Page liên kết, logo
3. Vào tab **Products** → thêm:
   - ✅ **Share on LinkedIn** (để đăng bài)
   - ✅ **Sign In with LinkedIn using OpenID Connect** (để lấy profile)
4. Tab **Auth** → copy **Client ID** và **Client Secret**
5. Thêm **Redirect URL** (ví dụ: `http://localhost:3000/callback`)

### 13.2. OAuth 2.0 flow (3 bước)

**Bước 1 — Đưa user đến trang authorize:**

```
https://www.linkedin.com/oauth/v2/authorization
  ?response_type=code
  &client_id=YOUR_CLIENT_ID
  &redirect_uri=http://localhost:3000/callback
  &scope=openid%20profile%20w_member_social
  &state=random123
```

**Bước 2 — Đổi code lấy access token:**

```bash
curl -X POST https://www.linkedin.com/oauth/v2/accessToken \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=authorization_code" \
  -d "code=AUTH_CODE_TREN" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "redirect_uri=http://localhost:3000/callback"
```

Response:

```json
{
  "access_token": "AQV8...",
  "expires_in": 5184000
}
```

**Bước 3 — Dùng token gọi API:**

```bash
curl -H "Authorization: Bearer AQV8..." \
  https://api.linkedin.com/v2/userinfo
```

### 13.3. Scopes phổ biến

| Scope | Quyền |
|---|---|
| `openid` | Định danh user |
| `profile` | Tên, ảnh đại diện |
| `email` | Email |
| `w_member_social` | Đăng bài hộ user |

---

## 14. API Lấy Thông Tin Profile

```bash
# Lấy thông tin cơ bản (OpenID Connect)
curl -H "Authorization: Bearer ACCESS_TOKEN" \
  https://api.linkedin.com/v2/userinfo
```

Response:

```json
{
  "sub": "abc123",
  "name": "Nguyễn Văn An",
  "given_name": "An",
  "family_name": "Nguyễn",
  "picture": "https://media.linkedin.com/...",
  "email": "an@example.com",
  "email_verified": true
}
```

> ⚠️ API profile đầy đủ (kinh nghiệm, học vấn) yêu cầu quyền partner — app thường không lấy được.

---

## 15. API Đăng Bài (Share API)

### 15.1. Lấy person URN

```bash
curl -H "Authorization: Bearer ACCESS_TOKEN" \
  https://api.linkedin.com/v2/userinfo
# → "sub": "abc123" → author = "urn:li:person:abc123"
```

### 15.2. Đăng bài text

```bash
curl -X POST https://api.linkedin.com/v2/ugcPosts \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "author": "urn:li:person:abc123",
    "lifecycleState": "PUBLISHED",
    "specificContent": {
      "com.linkedin.ugc.ShareContent": {
        "shareCommentary": { "text": "Hello từ API! 🚀" },
        "shareMediaCategory": "NONE"
      }
    },
    "visibility": { "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC" }
  }'
```

### 15.3. Đăng bài kèm link

```json
{
  "author": "urn:li:person:abc123",
  "lifecycleState": "PUBLISHED",
  "specificContent": {
    "com.linkedin.ugc.ShareContent": {
      "shareCommentary": { "text": "Bài viết mới của tôi!" },
      "shareMediaCategory": "ARTICLE",
      "media": [{
        "status": "READY",
        "originalUrl": "https://myblog.com/bai-viet"
      }]
    }
  },
  "visibility": { "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC" }
}
```

### 15.4. Đăng bài kèm ảnh (2 bước)

**Bước 1 — Đăng ký upload:**

```bash
curl -X POST https://api.linkedin.com/v2/assets?action=registerUpload \
  -H "Authorization: Bearer ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "registerUploadRequest": {
      "recipes": ["urn:li:digitalmediaRecipe:feedshare-image"],
      "owner": "urn:li:person:abc123",
      "serviceRelationships": [{
        "relationshipType": "OWNER",
        "identifier": "urn:li:userGeneratedContent"
      }]
    }
  }'
```

**Bước 2 — Upload binary + đăng bài** (dùng uploadUrl từ bước 1, rồi tạo ugcPost với `shareMediaCategory: "IMAGE"`).

---

## 16. Ví Dụ Code Hoàn Chỉnh (Node.js + Python)

### 16.1. Node.js — Đăng bài tự động

```bash
npm install express axios dotenv
```

```js
// server.js
require('dotenv').config();
const express = require('express');
const axios = require('axios');
const app = express();

const CLIENT_ID = process.env.LINKEDIN_CLIENT_ID;
const CLIENT_SECRET = process.env.LINKEDIN_CLIENT_SECRET;
const REDIRECT_URI = 'http://localhost:3000/callback';

// Bước 1: redirect user đến LinkedIn
app.get('/auth', (req, res) => {
  const url =
    'https://www.linkedin.com/oauth/v2/authorization' +
    `?response_type=code&client_id=${CLIENT_ID}` +
    `&redirect_uri=${encodeURIComponent(REDIRECT_URI)}` +
    `&scope=${encodeURIComponent('openid profile w_member_social')}` +
    `&state=xyz123`;
  res.redirect(url);
});

// Bước 2: nhận code, đổi lấy token, đăng bài
app.get('/callback', async (req, res) => {
  const { code } = req.query;
  try {
    // Đổi code → token
    const tokenRes = await axios.post(
      'https://www.linkedin.com/oauth/v2/accessToken',
      new URLSearchParams({
        grant_type: 'authorization_code',
        code,
        client_id: CLIENT_ID,
        client_secret: CLIENT_SECRET,
        redirect_uri: REDIRECT_URI,
      })
    );
    const accessToken = tokenRes.data.access_token;

    // Lấy person id
    const me = await axios.get('https://api.linkedin.com/v2/userinfo', {
      headers: { Authorization: `Bearer ${accessToken}` },
    });
    const personUrn = `urn:li:person:${me.data.sub}`;

    // Đăng bài
    await axios.post(
      'https://api.linkedin.com/v2/ugcPosts',
      {
        author: personUrn,
        lifecycleState: 'PUBLISHED',
        specificContent: {
          'com.linkedin.ugc.ShareContent': {
            shareCommentary: { text: 'Đăng tự động từ Node.js! 🚀' },
            shareMediaCategory: 'NONE',
          },
        },
        visibility: {
          'com.linkedin.ugc.MemberNetworkVisibility': 'PUBLIC',
        },
      },
      { headers: { Authorization: `Bearer ${accessToken}` } }
    );

    res.send('✅ Đăng bài thành công!');
  } catch (e) {
    res.status(500).send('❌ Lỗi: ' + e.response?.data?.message || e.message);
  }
});

app.listen(3000, () => console.log('Mở http://localhost:3000/auth'));
```

### 16.2. Python — Đăng bài

```bash
pip install requests python-dotenv
```

```python
import os
import requests
from dotenv import load_dotenv
from urllib.parse import urlencode

load_dotenv()

CLIENT_ID = os.getenv("LINKEDIN_CLIENT_ID")
CLIENT_SECRET = os.getenv("LINKEDIN_CLIENT_SECRET")
REDIRECT_URI = "http://localhost:3000/callback"


def get_auth_url():
    params = {
        "response_type": "code",
        "client_id": CLIENT_ID,
        "redirect_uri": REDIRECT_URI,
        "scope": "openid profile w_member_social",
        "state": "xyz123",
    }
    return "https://www.linkedin.com/oauth/v2/authorization?" + urlencode(params)


def exchange_code_for_token(code: str) -> str:
    r = requests.post(
        "https://www.linkedin.com/oauth/v2/accessToken",
        data={
            "grant_type": "authorization_code",
            "code": code,
            "client_id": CLIENT_ID,
            "client_secret": CLIENT_SECRET,
            "redirect_uri": REDIRECT_URI,
        },
    )
    r.raise_for_status()
    return r.json()["access_token"]


def post_update(access_token: str, text: str):
    # Lấy person URN
    me = requests.get(
        "https://api.linkedin.com/v2/userinfo",
        headers={"Authorization": f"Bearer {access_token}"},
    ).json()
    author = f"urn:li:person:{me['sub']}"

    # Đăng bài
    r = requests.post(
        "https://api.linkedin.com/v2/ugcPosts",
        headers={
            "Authorization": f"Bearer {access_token}",
            "Content-Type": "application/json",
        },
        json={
            "author": author,
            "lifecycleState": "PUBLISHED",
            "specificContent": {
                "com.linkedin.ugc.ShareContent": {
                    "shareCommentary": {"text": text},
                    "shareMediaCategory": "NONE",
                }
            },
            "visibility": {
                "com.linkedin.ugc.MemberNetworkVisibility": "PUBLIC"
            },
        },
    )
    r.raise_for_status()
    print("✅ Đăng bài thành công:", r.json().get("id"))


if __name__ == "__main__":
    print("1. Mở URL này:\n", get_auth_url())
    code = input("2. Dán code từ callback URL: ").strip()
    token = exchange_code_for_token(code)
    post_update(token, "Hello từ Python! 🐍")
```

---

## 17. Giới Hạn & Lưu Ý API

| Giới hạn | Chi tiết |
|---|---|
| Rate limit | Tùy endpoint, thường ~100-500 req/ngày cho app dev |
| App review | Muốn public cho nhiều user phải qua review của LinkedIn |
| Không có API đọc feed | Không thể lấy bài viết của người khác tự do |
| Không automation spam | Đăng bài hàng loạt = bị ban app |
| Token hết hạn | Access token ~60 ngày, cần refresh |

> ⚠️ **Tuân thủ:** Không scrape LinkedIn (vi phạm ToS, có thể bị kiện — đã có tiền lệ pháp lý). Chỉ dùng API chính thức.

---

## PHẦN C

---

## 18. Bảo Mật & Quyền Riêng Tư

### 18.1. Cài đặt quyền riêng tư nên chỉnh

1. **Settings & Privacy** → **Visibility:**
   - Tắt "Share profile updates" khi đang thử nghiệm chỉnh profile (tránh spam connections)
   - "Profile viewing options": chọn hiện tên hoặc ẩn danh khi xem profile người khác
2. **Đừng công khai:** số điện thoại, địa chỉ nhà, CMND
3. **Cẩn thận với tin nhắn lạ:** nhiều scam "tuyển dụng" yêu cầu chuyển tiền/phí

### 18.2. Nhận biết scam tuyển dụng

- 🚩 Yêu cầu đóng phí / đặt cọc
- 🚩 Phỏng vấn chỉ qua chat, không video call
- 🚩 Email không phải domain công ty
- 🚩 Lương cao bất thường cho việc nhẹ

---

## 19. Troubleshooting

| Vấn đề | Cách fix |
|---|---|
| Không ai xem profile | Thêm từ khóa vào headline/skills, đăng bài đều đặn |
| Gửi invite bị giới hạn | LinkedIn giới hạn ~100 invite/tuần — gửi có chọn lọc kèm note |
| Tài khoản bị restricted | Thường do spam invite/message — kháng cáo qua Help Center |
| API `403 Forbidden` | Thiếu product/scope cho app — kiểm tra tab Products |
| API `401 Unauthorized` | Token hết hạn — làm lại OAuth flow |
| Không lấy được email qua API | Cần scope `email` và user đồng ý |

---

## 20. Best Practices & Cheat Sheet

### 20.1. Checklist profile 5 phút mỗi tuần

- [ ] Đăng 1 bài hoặc comment 3 bài có chiều sâu
- [ ] Gửi 5-10 lời mời kết nối có chọn lọc
- [ ] Cập nhật project/skill mới học được
- [ ] Tương tác với bài của connections quan trọng

### 20.2. Công thức phát triển nhanh cho dev mới

1. **Tháng 1-2:** Hoàn thiện profile 100%, kết nối 200+ người cùng ngành
2. **Tháng 3-4:** Đăng 2 bài/tuần về thứ đang học
3. **Tháng 5-6:** Viết 1-2 bài "pillar" sâu (tutorial, so sánh) → dễ viral
4. **Ongoing:** Duy trì, trả lời comment, giúp đỡ người mới

### 20.3. Tài liệu tham khảo

- 📘 LinkedIn Help: https://www.linkedin.com/help/linkedin
- 🔌 API Docs: https://learn.microsoft.com/linkedin/
- 🎓 LinkedIn Learning: https://www.linkedin.com/learning/
- 👥 Developer community: https://developer.linkedin.com/

---

*📅 Cập nhật: 2026-10-10 — Tổng 20 phần: từ xây profile cá nhân đến lập trình với LinkedIn API.*

---

## 21. LinkedIn Cho Sinh Viên & Người Mới Bắt Đầu

### 21.1. Bắt đầu từ đâu khi chưa có kinh nghiệm?

Nhiều bạn nghĩ "chưa đi làm thì LinkedIn có gì để ghi" — sai! Bạn có thể ghi:

| Thay vì để trống | Hãy ghi |
|---|---|
| Experience trống | Project cá nhân, freelance nhỏ, CLB |
| Không có skills | Môn học giỏi, chứng chỉ online, tech đã học |
| About trống | Mục tiêu nghề nghiệp + thứ đang học |

### 21.2. Biến project cá nhân thành Experience

```
Chức danh: Frontend Developer (Personal Project)
Thời gian: 06/2026 – Hiện tại

• Xây dựng web bán hàng bằng React + TailwindCSS
• Tích hợp thanh toán demo, giỏ hàng, tìm kiếm
• Deploy lên Vercel: link-demo.vercel.app
• GitHub: github.com/ban/project
```

### 21.3. Chứng chỉ miễn phí nên lấy

- **freeCodeCamp:** Responsive Web Design, JavaScript
- **Google:** Google IT Support, Data Analytics (Coursera audit free)
- **LinkedIn Learning:** (cần Premium, nhưng có trial)
- **HackerRank:** Problem Solving, Python (có badge)

Thêm vào mục **Licenses & Certifications** kèm link verify.

### 21.4. Tìm internship

1. Bật Open to Work → chọn "Internship"
2. Follow công ty bạn thích → bật job alert
3. Nhắn trực tiếp cho HR/hiring manager (kèm CV + project)
4. Tham gia group "Việc làm IT Việt Nam"...

---

## 22. LinkedIn Analytics — Đo Lường Hiệu Quả

### 22.1. Xem ai vào profile

Profile → **Analytics** (chỉ hiện với Creator mode hoặc Premium):
- Số lượt xem profile 7/30/90 ngày
- Họ làm ở công ty nào, chức danh gì
- Từ khóa nào dẫn họ đến bạn

### 22.2. Analytics bài viết

Mỗi bài viết hiện:
- **Impressions:** số lần hiển thị
- **Reactions/comments/reposts:** tương tác
- **Click-through:** nếu có link

Bài nào impressions cao → phân tích tại sao (hook? giờ đăng? chủ đề?) → lặp lại công thức.

### 22.3. Creator Mode

Bật tại **Settings → Creator mode: On**:
- Nút **Follow** thay vì Connect làm mặc định (tốt cho người xây thương hiệu)
- Hiện số follower trên profile
- Thêm hashtags chủ đề (tối đa 5) vào profile
- Mở khóa LinkedIn Live, Newsletter

---

## 23. Tự Động Hóa & Tool Hỗ Trợ

### 23.1. Lên lịch đăng bài

LinkedIn cho **schedule post** trực tiếp:
1. Soạn bài → bấm đồng hồ ⏰ → chọn ngày giờ
2. Hoặc dùng tool: Buffer, Hootsuite, Typefully

### 23.2. Tool viết content

- **Canva:** design carousel, banner
- **Typefully:** soạn + schedule + analytics
- **Shield Analytics:** analytics sâu cho LinkedIn

### 23.3. Tự động hóa hợp lệ vs spam

| Được phép | Bị cấm (risk ban) |
|---|---|
| Schedule bài viết của chính bạn | Auto-connect hàng loạt |
| Dùng API chính thức | Scrape profile người khác |
| Chatbot trả lời inbox của bạn | Spam InMail hàng loạt |

---

## 24. Xây Newsletter Trên LinkedIn

### 24.1. Tại sao newsletter?

- Subscriber nhận email mỗi khi bạn ra bài mới (không phụ thuộc algorithm)
- Xây audience trung thành
- Cần Creator Mode để bật

### 24.2. Cách tạo

1. Bật Creator Mode
2. **Write article** → **Create a newsletter**
3. Đặt tên, mô tả, logo, tần suất (hàng tuần/hàng tháng)
4. Viết số đầu tiên → Publish

### 24.3. Ý tưởng newsletter cho dev

- "Tuần này học gì" — tóm tắt kiến thức mới
- "Code review" — phân tích 1 đoạn code hay/dở
- "Dev news VN" — tin tức tech Việt Nam

---

## 25. Câu Hỏi Thường Gặp (FAQ)

**Hỏi: Mới tạo tài khoản có nên kết nối ồ ạt không?**
Đáp: Không — LinkedIn giới hạn ~100 invite/tuần, tài khoản mới spam dễ bị restrict. Gửi 10-20/tuần kèm note.

**Hỏi: Nên viết tiếng Việt hay tiếng Anh?**
Đáp: Cả hai đều được. Tiếng Anh → job global, tiếng Việt → cộng đồng local gần gũi hơn. Có thể viết song ngữ cho bài quan trọng.

**Hỏi: Bao lâu nên đăng 1 bài?**
Đáp: 2-3 bài/tuần là tốt. Quan trọng là đều đặn, không phải số lượng.

**Hỏi: Profile có cần 100% tiếng Anh không?**
Đáp: Nếu target job quốc tế/remote → tiếng Anh. Nếu chỉ làm công ty VN → song ngữ hoặc tiếng Việt cũng OK.

**Hỏi: Có nên mua follower/connections không?**
Đáp: Không bao giờ. Follower ảo = engagement thấp = algorithm bóp reach. Chất lượng > số lượng.

**Hỏi: API có lấy được danh sách connections không?**
Đáp: Không với app thường — cần partner access. Đây là giới hạn cố ý của LinkedIn.

---

*Cập nhật: đã bổ sung sinh viên, analytics, automation, newsletter, FAQ — tổng 25 phần.*
