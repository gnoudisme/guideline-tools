# Hướng Dẫn Toàn Tập HubSpot — CRM Miễn Phí Cho Mọi Doanh Nghiệp

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10

---

## Mục Lục

1. [HubSpot Là Gì?](#1-hubspot-là-gì)
2. [Tạo Tài Khoản & Setup](#2-tạo-tài-khoản--setup)
3. [Tổng Quan Giao Diện](#3-tổng-quan-giao-diện)
4. [Contacts — Quản Lý Khách Hàng](#4-contacts--quản-lý-khách-hàng)
5. [Companies — Quản Lý Công Ty](#5-companies--quản-lý-công-ty)
6. [Deals — Pipeline Bán Hàng](#6-deals--pipeline-bán-hàng)
7. [Tickets — Chăm Sóc Khách Hàng](#7-tickets--chăm-sóc-khách-hàng)
8. [Email Marketing](#8-email-marketing)
9. [Forms — Tạo Form Thu Lead](#9-forms--tạo-form-thu-lead)
10. [Landing Pages & Website](#10-landing-pages--website)
11. [Live Chat & Chatbot](#11-live-chat--chatbot)
12. [Workflows — Tự Động Hóa](#12-workflows--tự-động-hóa)
13. [Reports — Báo Cáo](#13-reports--báo-cáo)
14. [HubSpot API Cơ Bản](#14-hubspot-api-cơ-bản)
15. [Tích Hợp Với Công Cụ Khác](#15-tích-hợp-với-công-cụ-khác)
16. [Bảng Giá Chi Tiết](#16-bảng-giá-chi-tiết)
17. [HubSpot vs Đối Thủ](#17-hubspot-vs-đối-thủ)
18. [Xử Lý Lỗi Thường Gặp](#18-xử-lý-lỗi-thường-gặp)
19. [Best Practices](#19-best-practices)
20. [Cheat Sheet](#20-cheat-sheet)

---

## 1. HubSpot Là Gì?

### 1.1 Định Nghĩa

**HubSpot** là nền tảng CRM (Customer Relationship Management — Quản lý quan hệ khách hàng) tất cả-trong-một. Nó giúp doanh nghiệp:

- **Lưu trữ** thông tin khách hàng ở một nơi duy nhất
- **Theo dõi** toàn bộ lịch sử tương tác (email, cuộc gọi, meeting, chat)
- **Tự động hóa** marketing, bán hàng, chăm sóc khách hàng
- **Đo lường** hiệu quả qua báo cáo

### 1.2 5 "Hub" Chính

HubSpot chia thành 5 trung tâm (Hub), mỗi Hub phục vụ 1 bộ phận:

| Hub | Dành cho | Chức năng chính |
|---|---|---|
| **Marketing Hub** | Team marketing | Email marketing, landing page, SEO, social media, ads |
| **Sales Hub** | Team bán hàng | Pipeline deals, email tracking, meeting scheduler, quotes |
| **Service Hub** | Team CSKH | Tickets, knowledge base, live chat, surveys |
| **CMS Hub** | Team web/content | Xây website, blog, landing page |
| **Operations Hub** | Team vận hành | Đồng bộ dữ liệu, tự động hóa, data quality |

> 💡 **Tin vui:** Bản **Free** đã bao gồm tính năng cơ bản của TẤT CẢ 5 Hub!

### 1.3 Tại Sao HubSpot Phổ Biến?

1. **Miễn phí thật:** CRM free không giới hạn số contacts (lên đến 1 triệu!), không cần thẻ tín dụng
2. **Dễ dùng:** giao diện trực quan, không cần đào tạo nhiều
3. **Tất cả trong một:** không cần mua 5 công cụ riêng lẻ
4. **Mở rộng dần:** bắt đầu free → nâng cấp từng Hub khi lớn lên

### 1.4 Ai Nên Dùng HubSpot?

- **Startup / shop online:** quản lý khách hàng, đơn hàng miễn phí
- **Agency:** quản lý nhiều khách hàng, chiến dịch marketing
- **Doanh nghiệp B2B:** pipeline bán hàng phức tạp, nhiều giai đoạn
- **Freelancer:** theo dõi lead, gửi proposal chuyên nghiệp

---

## 2. Tạo Tài Khoản & Setup

### 2.1 Đăng Ký

1. Vào **https://www.hubspot.com** → bấm **Get started free**
2. Nhập: tên, email công việc, số điện thoại, tên công ty, website (nếu có)
3. Chọn quy mô công ty, vai trò của bạn
4. ✅ Không cần thẻ tín dụng!

### 2.2 Setup Wizard (Lần Đầu)

HubSpot sẽ hướng dẫn từng bước:

1. **Import contacts:** upload file CSV/Excel danh sách khách hàng hiện có (hoặc bỏ qua)
2. **Kết nối email:** kết nối Gmail/Outlook để tracking email (rất nên làm!)
3. **Cài đặt công ty:** logo, múi giờ, đơn vị tiền tệ (VND)
4. **Mời team:** thêm email đồng nghiệp

### 2.3 Cài Đặt Cơ Bản Nên Làm Ngay

**Settings (bánh răng góc phải dưới) → General:**

| Cài đặt | Nên để |
|---|---|
| Timezone | `(GMT+07:00) Bangkok, Hanoi, Jakarta` |
| Currency | `VND - Vietnamese Dong` |
| Language | English (chưa có tiếng Việt đầy đủ) |
| Date format | `DD/MM/YYYY` |

**Kết nối email cá nhân:**
- Settings → General → Email → **Connect personal email**
- Chọn Gmail/Outlook → đăng nhập → Allow
- Từ nay: gửi email từ HubSpot = gửi từ email thật của bạn
- **Email tracking:** biết khi nào khách mở email, click link!

---

## 3. Tổng Quan Giao Diện

### 3.1 Thanh Điều Hướng Trên

| Menu | Chứa gì |
|---|---|
| **CRM** | Contacts, Companies, Deals, Tickets, Calls |
| **Marketing** | Email, Social, Ads, Landing Pages, Forms, Workflows |
| **Sales** | Deals, Tasks, Meetings, Quotes, Playbooks |
| **Service** | Tickets, Knowledge Base, Live Chat, Surveys |
| **Automation** | Workflows |
| **Reporting** | Dashboards, Reports |
| **Library** | Templates, Files, Documents |

### 3.2 Tìm Kiếm Nhanh (Rất Hữu Ích!)

- Bấm **/** (dấu gạch chéo) hoặc bấm ô search trên cùng
- Gõ tên contact, công ty, deal → nhảy đến ngay
- Gõ tên tính năng → mở nhanh (như Spotlight trên Mac)

---

## 4. Contacts — Quản Lý Khách Hàng

### 4.1 Contact Là Gì?

**Contact** = 1 người (khách hàng, lead, đối tác). Mỗi contact lưu:

- Thông tin cơ bản: tên, email, sđt, chức danh, công ty
- **Timeline:** toàn bộ lịch sử tương tác (email đã gửi/nhận, cuộc gọi, meeting, note, task)
- **Properties:** các trường tùy chỉnh (xem 4.4)

### 4.2 Tạo Contact

**Cách 1 — Tạo tay:**
1. CRM → **Contacts** → **Create contact**
2. Nhập: email (bắt buộc), tên, sđt, công ty...
3. **Create**

**Cách 2 — Import hàng loạt:**
1. CRM → Contacts → **Import**
2. Chọn **Start an import** → **File from computer**
3. Upload CSV/Excel → map cột (cột "Họ tên" → First Name...)
4. **Finish import**

> 💡 **Mẫu CSV:**
> ```csv
> Email,First Name,Last Name,Phone,Company
> an@example.com,An,Nguyễn,0901234567,Công ty ABC
> binh@example.com,Bình,Trần,0912345678,Công ty XYZ
> ```

**Cách 3 — Tự động:** từ Form, Chat, API, tích hợp (xem các phần sau)

### 4.3 Xem & Quản Lý Contacts

**Danh sách contacts:**
- CRM → Contacts → thấy bảng tất cả contacts
- **Lọc:** bấm **Add filter** — ví dụ: `City is Hà Nội` + `Lifecycle stage is Customer`
- **Lưu view:** sau khi lọc → **Save view** → đặt tên `Khách Hà Nội` (lần sau bấm 1 cái là ra)
- **Cột hiển thị:** bấm **Edit columns** để chọn cột muốn thấy

**Trang chi tiết 1 contact:**
- Bấm vào tên contact → thấy:
  - **About:** thông tin chi tiết
  - **Timeline:** dòng thời gian tương tác (email, call, note...)
  - Bên phải: công ty, deals, tickets liên quan

**Thao tác nhanh trên timeline:**
- **Log call:** ghi lại cuộc gọi (thời gian, nội dung)
- **Log email:** ghi email gửi ngoài HubSpot
- **Create note:** ghi chú nội bộ
- **Create task:** giao việc follow-up
- **Schedule meeting:** đặt lịch hẹn (gửi link đặt lịch cho khách)

### 4.4 Custom Properties (Trường Tùy Chỉnh)

Mặc định có: tên, email, sđt... Nhưng bạn cần thêm: "Nguồn lead", "Ngân sách", "Sở thích"...

1. Settings → Properties → **Contact properties** → **Create property**
2. Điền:
   - **Label:** `Nguồn lead` (tên hiển thị)
   - **Field type:** chọn kiểu:

| Kiểu | Dùng khi nào |
|---|---|
| Single-line text | Text ngắn (mã khách hàng) |
| Multi-line text | Ghi chú dài |
| Number | Số (ngân sách, tuổi) |
| Date picker | Ngày (sinh nhật, ngày mua) |
| Dropdown select | Chọn 1 (nguồn lead: Facebook/Google/Giới thiệu) |
| Multiple checkboxes | Chọn nhiều (sở thích) |
| Radio select | Chọn 1 kiểu nút tròn |
| Checkbox | Có/Không (đồng ý nhận tin) |

3. **Create** → property xuất hiện trong form tạo/sửa contact

### 4.5 Lifecycle Stages (Giai Đoạn Vòng Đời)

HubSpot tự phân loại contact theo hành trình:

```
Subscriber → Lead → Marketing Qualified Lead → Sales Qualified Lead
    → Opportunity → Customer → Evangelist
```

- **Subscriber:** mới đăng ký nhận tin
- **Lead:** đã để lại thông tin
- **MQL:** marketing đánh giá tiềm năng
- **SQL:** sale xác nhận đáng follow
- **Opportunity:** đang đàm phán (gắn với deal)
- **Customer:** đã mua hàng
- **Evangelist:** khách trung thành, giới thiệu người khác

> 💡 Dùng lifecycle để lọc: "gửi email ưu đãi cho tất cả Customer", "sale chỉ follow SQL"...

### 4.6 Xóa / Gộp Contacts Trùng

**Gộp trùng:**
1. Mở contact → **Actions** → **Merge**
2. Chọn contact trùng → chọn dữ liệu nào giữ lại
3. **Merge** → timeline 2 contact gộp thành 1

**Xóa:**
- Chọn contact → **Actions** → **Delete**
- Hoặc chọn nhiều → **Delete** hàng loạt

---

## 5. Companies — Quản Lý Công Ty

### 5.1 Company Là Gì?

**Company** = 1 tổ chức. Một company có thể có **nhiều contacts** (nhiều người cùng công ty).

Ví dụ:
```
Công ty ABC (Company)
  ├── Nguyễn Văn A — Giám đốc (Contact)
  ├── Trần Thị B — Kế toán (Contact)
  └── Lê Văn C — Nhân viên mua hàng (Contact)
```

### 5.2 Tạo & Liên Kết

- **Tự động:** khi tạo contact có điền "Company name", HubSpot tự tạo company và liên kết
- **Tay:** CRM → Companies → Create company
- **Liên kết tay:** mở contact → bên phải → **Add company** → tìm tên

### 5.3 Thông Tin Company

- Tên công ty, website, ngành, quy mô, địa chỉ
- **Tự động enrich:** nhập domain website → HubSpot tự điền thông tin công ty (logo, ngành, quy mô)!
- Timeline riêng: tất cả tương tác với mọi người trong công ty

---

## 6. Deals — Pipeline Bán Hàng

### 6.1 Deal Là Gì?

**Deal** = 1 cơ hội bán hàng đang theo đuổi. Mỗi deal có:
- **Giá trị** (số tiền)
- **Giai đoạn** (đang ở bước nào trong quy trình bán)
- **Liên kết** với contact(s) và company

### 6.2 Pipeline (Đường Ống Bán Hàng)

Pipeline mặc định có các giai đoạn:

```
Appointment Scheduled → Qualified to Buy → Presentation Scheduled
    → Decision Maker Bought-In → Contract Sent → Closed Won / Closed Lost
```

**Tùy chỉnh pipeline cho shop online:**

1. Sales → Deals → **Board actions** → **Edit stages**
2. Sửa thành:

```
Mới (lead mới vào)
  → Đang tư vấn (đang chat/gọi)
    → Đã báo giá (gửi báo giá rồi)
      → Chờ thanh toán (khách đồng ý, chưa chuyển tiền)
        → Thành công (Closed Won)
        → Thất bại (Closed Lost)
```

### 6.3 Tạo Deal

1. Sales → **Deals** → **Create deal**
2. Nhập:
   - **Deal name:** `Đơn hàng - Nguyễn Văn A - iPhone 15`
   - **Amount:** `25000000` (VND)
   - **Pipeline & Stage:** chọn giai đoạn hiện tại
   - **Associated contacts/companies:** liên kết khách hàng
   - **Close date:** ngày dự kiến chốt
3. **Create**

### 6.4 Kéo Thả Trên Board

- Deals hiển thị dạng **Kanban** (cột theo giai đoạn)
- **Kéo deal** từ cột này sang cột khác = chuyển giai đoạn
- Tổng giá trị mỗi cột hiện trên đầu → biết ngay pipeline đang có bao nhiêu tiền

### 6.5 Tự Động Hóa Deals

- Deal vào stage "Chờ thanh toán" → tự tạo task "Nhắc thanh toán sau 2 ngày"
- Deal "Closed Won" → tự tạo ticket "Giao hàng" cho team vận hành
- (Dùng Workflows — xem phần 12)

---

## 7. Tickets — Chăm Sóc Khách Hàng

### 7.1 Ticket Là Gì?

**Ticket** = 1 yêu cầu hỗ trợ từ khách hàng. Pipeline mặc định:

```
New → Waiting on contact → Waiting on us → Closed
```

### 7.2 Tạo Ticket

1. Service → **Tickets** → **Create ticket**
2. Nhập: tiêu đề (`Lỗi đăng nhập app`), mô tả, mức ưu tiên, liên kết contact
3. **Create**

**Tự động tạo ticket từ:**
- Email gửi đến `hotro@congty.com` → tự tạo ticket
- Form "Liên hệ hỗ trợ" → tự tạo ticket
- Live chat → chuyển thành ticket

### 7.3 Quản Lý

- Board Kanban như deals
- Gán cho nhân viên (Assignee)
- Đặt SLA: ticket ưu tiên cao phải trả lời trong 2h
- Khách trả lời email → tự động cập nhật vào ticket

---

## 8. Email Marketing

### 8.1 Tạo Email Chiến Dịch

1. Marketing → **Email** → **Create email**
2. Chọn loại:
   - **Regular:** gửi 1 lần cho danh sách
   - **Automated:** gửi tự động qua workflow
3. Chọn template → kéo thả chỉnh sửa (drag & drop editor)
4. Viết tiêu đề, nội dung

### 8.2 Cá Nhân Hóa (Personalization)

Chèn thông tin từng người nhận:

```
Chào {{ contact.firstname }},

Cảm ơn bạn đã mua hàng tại {{ company.name }}.
Mã giảm giá dành riêng cho bạn: GIAM10
```

Mỗi người nhận thấy tên **của chính họ** — tăng tỷ lệ mở email đáng kể!

### 8.3 Gửi & Theo Dõi

1. **Send to:** chọn danh sách (list) người nhận
2. **Review** → **Send**
3. Sau khi gửi, xem báo cáo:
   - **Open rate:** % người mở email
   - **Click rate:** % người click link
   - Ai mở, ai click, ai unsubscribe

### 8.4 A/B Testing

- Tạo 2 phiên bản tiêu đề khác nhau
- HubSpot tự gửi mỗi bản cho 1 nửa danh sách
- Bản thắng (mở nhiều hơn) → tự gửi cho nửa còn lại

> 📌 A/B testing cần gói Marketing Starter trở lên.

---

## 9. Forms — Tạo Form Thu Lead

### 9.1 Tạo Form

1. Marketing → **Lead Capture** → **Forms** → **Create form**
2. Kéo thả các trường: Họ tên, Email, SĐT, dropdown "Nhu cầu"...
3. **Style:** chỉnh màu sắc, bo góc
4. **Options:**
   - Gửi email thông báo khi có người điền
   - Chuyển hướng đến trang cảm ơn sau khi submit
5. **Publish** → lấy code nhúng hoặc link

### 9.2 Nhúng Vào Website

**Cách 1 — Embed code:**
```html
<!-- Dán vào website của bạn -->
<script charset="utf-8" src="//js.hsforms.net/forms/embed/v2.js"></script>
<script>
  hbspt.forms.create({
    portalId: "12345678",
    formId: "abcdef12-3456-7890-abcd-ef1234567890"
  });
</script>
```

**Cách 2 — Link trực tiếp:** HubSpot tạo link form độc lập để chia sẻ

**Cách 3 — WordPress:** cài plugin HubSpot → form tự đồng bộ

### 9.3 Dữ Liệu Chảy Đi Đâu?

Người điền form → **tự động:**
1. Tạo contact mới (hoặc cập nhật contact cũ)
2. Ghi nguồn: `Lead source = Form XYZ`
3. Kích hoạt workflow (nếu có): gửi email chào mừng...

---

## 10. Landing Pages & Website

### 10.1 Tạo Landing Page

1. Marketing → **Website** → **Landing Pages** → **Create**
2. Chọn template theo mục đích: thu lead, quảng cáo sản phẩm, đăng ký sự kiện...
3. Kéo thả chỉnh sửa: text, ảnh, form, nút CTA, video
4. **Publish** → có URL: `congty.hubspotpagebuilder.com/trang-khuyen-mai`

### 10.2 Kết Nối Domain Riêng

- Settings → Domains & URLs → **Connect a domain**
- Nhập `www.congty.com` → làm theo hướng dẫn DNS
- Landing page chạy trên domain chính chủ, chuyên nghiệp hơn

> 📌 Cần gói CMS Starter trở lên để dùng domain riêng.

### 10.3 Blog

- Marketing → Website → **Blog** → viết bài như WordPress
- Tích hợp sẵn form thu lead, CTA
- SEO tools: gợi ý từ khóa, meta description

---

## 11. Live Chat & Chatbot

### 11.1 Cài Đặt Live Chat

1. Service → **Chatflows** → **Create chatflow**
2. Chọn **Website**
3. Tùy chỉnh: màu sắc, avatar, lời chào
4. Lấy code nhúng → dán vào website:

```html
<!-- HubSpot tracking + chat -->
<script id="hs-script-loader" async defer
  src="//js.hs-scripts.com/12345678.js"></script>
```

### 11.2 Chatbot Tự Động

Tạo chatbot hỏi đáp tự động:

```
Bot: Chào bạn! Bạn cần hỗ trợ gì?
  ├─ [Xem sản phẩm] → Bot gửi link catalogue
  ├─ [Báo giá] → Bot hỏi: "Bạn quan tâm sản phẩm nào?"
  │                → Khách trả lời → Bot: "Để lại SĐT, sale sẽ gọi!"
  │                → Tạo contact + ticket tự động
  └─ [Gặp nhân viên] → Chuyển cho nhân viên trực
```

### 11.3 Quy Tắc Hiển Thị

- Chỉ hiện chat ở trang `/san-pham` (không hiện ở trang chủ)
- Chỉ hiện giờ hành chính (8h-17h), ngoài giờ hiện form để lại lời nhắn
- Ẩn với khách đã mua hàng (dựa vào lifecycle stage)

---

## 12. Workflows — Tự Động Hóa

**Workflows** là "Zapier bên trong HubSpot" — tự động hóa dựa trên dữ liệu CRM.

> 📌 Cần gói Starter trở lên (bản Free không có Workflows).

### 12.1 Tạo Workflow Đầu Tiên

**Mục tiêu:** Khách điền form → gửi email chào mừng → sau 3 ngày gửi email giới thiệu sản phẩm.

1. Automation → **Workflows** → **Create workflow** → **From scratch**
2. **Enrollment trigger:** `Form submission` → chọn form "Đăng ký tư vấn"
3. Thêm action 1: **Send email** → chọn email "Chào mừng"
4. Thêm **Delay:** `3 days`
5. Thêm action 2: **Send email** → chọn email "Giới thiệu sản phẩm"
6. **Review and publish** → **Turn on**

### 12.2 Trigger Phổ Biến

| Trigger | Ví dụ |
|---|---|
| Form submission | Điền form → chạy |
| Contact property changed | Lifecycle thành Customer → chạy |
| Deal stage changed | Deal vào "Chờ thanh toán" → nhắc |
| Email engagement | Mở email 3 lần mà chưa mua → báo sale |
| Page view | Xem trang pricing 2 lần → chatbot mời chat |

### 12.3 Action Phổ Biến

| Action | Ví dụ |
|---|---|
| Send email | Gửi email tự động |
| Create task | Giao task cho sale |
| Set property value | Đổi lifecycle thành SQL |
| Add to list | Thêm vào danh sách |
| Send Slack notification | Báo team |
| Create deal/ticket | Tạo deal/ticket tự động |
| Webhook | Gọi API bên ngoài |

### 12.4 Ví Dụ Workflows Thực Tế

**1. Nuôi dưỡng lead (Lead Nurturing):**
```
Form submit
  → Gửi email chào mừng (ngay)
  → Delay 2 ngày → Gửi email "Câu chuyện khách hàng"
  → Delay 3 ngày → Gửi email "Ưu đãi đặc biệt"
  → Nếu mở email ưu đãi → Tạo task cho sale gọi điện
```

**2. Chăm sóc sau bán:**
```
Deal → Closed Won
  → Gửi email cảm ơn + hướng dẫn sử dụng
  → Delay 7 ngày → Gửi khảo sát hài lòng
  → Nếu đánh giá 5 sao → Gửi email "Giới thiệu bạn bè được thưởng"
  → Nếu đánh giá < 3 sao → Tạo ticket ưu tiên cao
```

**3. Giỏ hàng bỏ quên (cho e-commerce):**
```
Contact xem trang checkout nhưng không mua (tích hợp web)
  → Delay 1 giờ → Gửi email "Bạn quên gì nè!"
  → Delay 1 ngày → Gửi email + mã giảm 10%
```

---

## 13. Reports — Báo Cáo

### 13.1 Dashboard Có Sẵn

Reporting → **Dashboards** → xem các dashboard mẫu:
- **Sales:** doanh thu theo tháng, pipeline hiện tại, tỷ lệ chốt
- **Marketing:** traffic web, lead theo nguồn, email performance
- **Service:** số ticket, thời gian xử lý trung bình, CSAT

### 13.2 Tạo Báo Cáo Tùy Chỉnh

1. Reporting → **Reports** → **Create report**
2. Chọn loại:
   - **Single object:** 1 loại dữ liệu (ví dụ: deals)
   - **Cross-object:** kết hợp (contacts + deals: "doanh thu theo nguồn lead")
3. Kéo thả: trục X (tháng), trục Y (tổng tiền), lọc (năm 2026)
4. Chọn kiểu biểu đồ: cột, đường, tròn, bảng
5. **Save** → thêm vào dashboard

### 13.3 Báo Cáo Nên Có

| Báo cáo | Cho ai |
|---|---|
| Doanh thu theo tháng | Sếp |
| Pipeline hiện tại (theo stage) | Trưởng sale |
| Lead theo nguồn (Facebook/Google/Web...) | Marketing |
| Tỷ lệ chốt deal (won/total) | Sale |
| Email open/click rate | Marketing |
| Ticket tồn đọng theo nhân viên | CSKH |

---

## 14. HubSpot API Cơ Bản

Dùng API khi cần tích hợp sâu (website/app tự code).

### 14.1 Tạo API Key

1. Settings → Integrations → **Private Apps** → **Create private app**
2. Đặt tên: `Website Integration`
3. Tab **Scopes:** tick quyền cần (ví dụ: `crm.objects.contacts.write`)
4. **Create app** → copy **Access Token** (dạng `pat-xxx`)

> ⚠️ Token chỉ hiện 1 lần — lưu ngay!

### 14.2 Tạo Contact Qua API

```bash
curl -X POST https://api.hubapi.com/crm/v3/objects/contacts \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "email": "khach@example.com",
      "firstname": "An",
      "lastname": "Nguyễn",
      "phone": "0901234567"
    }
  }'
```

### 14.3 Lấy Danh Sách Contacts

```bash
curl "https://api.hubapi.com/crm/v3/objects/contacts?limit=10&properties=email,firstname,lastname" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### 14.4 Tạo Deal Qua API

```bash
curl -X POST https://api.hubapi.com/crm/v3/objects/deals \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "properties": {
      "dealname": "Đơn hàng website #1234",
      "amount": "2500000",
      "dealstage": "appointmentscheduled",
      "pipeline": "default"
    }
  }'
```

### 14.5 Webhooks (Nhận Sự Kiện)

1. Settings → Integrations → **Webhooks** (cần gói Operations)
2. Đăng ký URL của bạn → HubSpot POST khi có sự kiện (contact mới, deal đổi stage...)

### 14.6 Rate Limits

| Gói | Giới hạn |
|---|---|
| Free | 100 requests / 10 giây |
| Starter | 100 / 10 giây |
| Professional | 150 / 10 giây |

---

## 15. Tích Hợp Với Công Cụ Khác

### 15.1 Tích Hợp Có Sẵn (App Marketplace)

Vào **App Marketplace** (biểu tượng lưới trên thanh top) → 1.500+ tích hợp:

| App | Tích hợp làm gì |
|---|---|
| Gmail / Outlook | Gửi/track email, đồng bộ lịch |
| Slack | Báo deal mới, ticket mới vào Slack |
| Zoom / Google Meet | Tự tạo link meeting khi đặt lịch |
| Stripe | Tạo deal khi có thanh toán |
| Shopify / WooCommerce | Đồng bộ đơn hàng, khách hàng |
| Facebook Ads / Google Ads | Đồng bộ lead từ form quảng cáo |
| Zapier / Make | Kết nối với 7.000+ app khác |
| WordPress | Form, popup, tracking |

### 15.2 Kết Nối Slack (Ví Dụ)

1. App Marketplace → tìm **Slack** → **Install**
2. Đăng nhập Slack → chọn workspace → Allow
3. Settings → chọn kênh nhận thông báo:
   - `#sales`: báo deal mới/thắng
   - `#support`: báo ticket mới

### 15.3 Kết Nối Qua Zapier (Khi Không Có Tích Hợp Sẵn)

```
Ví dụ: Đơn hàng từ Haravan (chưa có tích hợp HubSpot)
[Haravan: Đơn mới] → [HubSpot: Tạo contact + deal]
```

---

## 16. Bảng Giá Chi Tiết

> 💲 Giá USD/tháng, có thể thay đổi — kiểm tra tại hubspot.com/pricing

### 16.1 CRM Free (Mãi Mãi $0)

| Tính năng | Giới hạn |
|---|---|
| Contacts | 1.000.000 (1 triệu!) |
| Users | Không giới hạn |
| Email marketing | 2.000 emails/tháng |
| Forms, live chat | ✅ |
| Deals, tickets | ✅ |
| Gmail/Outlook integration | ✅ |

### 16.2 Marketing Hub

| Gói | Giá từ | Nổi bật |
|---|---|---|
| Free | $0 | 2.000 emails/tháng, forms, ads |
| Starter | ~$20 | 1.000 contacts marketing, A/B test, loại bỏ logo HubSpot |
| Professional | ~$800 | Workflows, smart content, A/B, custom reporting |
| Enterprise | ~$3.600 | AI, SSO, phân quyền chi tiết |

### 16.3 Sales Hub

| Gói | Giá từ | Nổi bật |
|---|---|---|
| Free | $0 | Deals, tasks, meeting scheduler, Gmail tracking |
| Starter | ~$20 | 2 sales seats, eSignatures đơn giản |
| Professional | ~$90/seat | Sequences (email tự động nhiều bước), quotes, forecasting |
| Enterprise | ~$150/seat | Playbooks, SSO, custom objects |

### 16.4 Service Hub

| Gói | Giá từ | Nổi bật |
|---|---|---|
| Free | $0 | Tickets, live chat, knowledge base cơ bản |
| Starter | ~$20 | 2 seats, ticket automation đơn giản |
| Professional | ~$90/seat | SLA, surveys (CSAT/NPS), automation |
| Enterprise | ~$150/seat | SSO, phân quyền, custom objects |

### 16.5 Nên Bắt Đầu Từ Đâu?

```
Mới bắt đầu → CRM Free (đủ dùng rất lâu!)
  → Cần gửi email nhiều → Marketing Starter ($20)
  → Cần automation → Marketing/Sales Professional
  → Team lớn → Enterprise
```

> 💡 **Mẹo:** Dùng Free 3-6 tháng cho quen, đo nhu cầu thật rồi mới nâng cấp. Đừng mua Professional ngay từ đầu nếu chưa cần!

---

## 17. HubSpot vs Đối Thủ

| Tiêu chí | **HubSpot** | Salesforce | Zoho CRM | Pipedrive |
|---|---|---|---|---|
| Bản free | ⭐ Rất tốt (1M contacts) | Hạn chế (trial) | Tốt (3 users) | Không (trial 14 ngày) |
| Dễ dùng | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| Marketing tích hợp | ✅ Mạnh | Cần Pardot (mua thêm) | ✅ Có | ❌ Yếu |
| Giá Professional | ~$800 (Marketing) | ~$1.700+ | ~$50/user | ~$50/user |
| Phù hợp | SMB, startup | Enterprise lớn | Giá rẻ | Sale đơn giản |
| Tiếng Việt | ❌ Chưa có | ❌ | ❌ | ❌ |

**Kết luận:**
- **Mới bắt đầu / ngân sách thấp** → HubSpot Free
- **Tập đoàn lớn, quy trình phức tạp** → Salesforce
- **Muốn rẻ nhất** → Zoho
- **Chỉ cần quản lý sale đơn giản** → Pipedrive

---

## 18. Xử Lý Lỗi Thường Gặp

| Vấn đề | Nguyên nhân | Cách khắc phục |
|---|---|---|
| Không nhận được email test | Vào spam, hoặc email chưa verify | Kiểm tra spam; Settings → verify domain gửi |
| Form không lưu contact | Thiếu trường email (bắt buộc) | Đảm bảo form có trường Email |
| Workflow không chạy | Chưa bật (Turn on), hoặc enrollment không khớp | Kiểm tra workflow ON; test enrollment |
| Email vào spam của khách | Chưa xác thực domain (SPF/DKIM) | Settings → Domains → authenticate domain |
| Contact bị trùng | Import 2 lần, form điền trùng email | Dùng Merge; bật "deduplication" khi import |
| Không thấy deal trong board | Lọc sai pipeline | Kiểm tra dropdown pipeline ở đầu board |
| API trả 401 | Token sai/hết hạn | Tạo private app mới, copy token mới |
| API trả 429 | Vượt rate limit | Thêm delay, giảm tần suất gọi |
| Tích hợp Slack không báo | Chưa chọn kênh / chưa map sự kiện | Vào integration settings kiểm tra lại |

---

## 19. Best Practices

### 19.1 Dữ Liệu Sạch

1. **Quy ước đặt tên:** Deal `DH-2026-001 - Tên khách`, không để `deal mới`
2. **Bắt buộc trường quan trọng:** email, sđt, nguồn lead — không để trống
3. **Dọn định kỳ:** hàng tháng kiểm tra contact trùng, thông tin cũ
4. **Dùng dropdown** thay vì text tự do cho trường phân loại (tránh "HN", "Hà Nội", "Ha Noi" 3 kiểu)

### 19.2 Quy Trình

1. **Mọi tương tác đều log:** gọi điện xong → log call ngay (đừng để mai)
2. **Task có deadline:** không để task "khi nào rảnh làm"
3. **Deal cập nhật thường xuyên:** deal nào 30 ngày không động → xem lại
4. **Đào tạo team:** viết 1 trang hướng dẫn nội bộ (quy ước dùng HubSpot)

### 19.3 Bảo Mật

1. Bật **2FA** cho tất cả users (Settings → Security)
2. Phân quyền: sale chỉ thấy deals của mình (cần gói Pro)
3. Khi nhân viên nghỉ: **deactivate** ngay, chuyển deals cho người khác
4. API token: không commit lên Git public, rotate định kỳ

---

## 20. Cheat Sheet

### Điều hướng nhanh
```
/              → tìm kiếm toàn hệ thống
CRM → Contacts → danh sách khách hàng
Sales → Deals  → pipeline bán hàng
Service → Tickets → hỗ trợ khách hàng
Automation → Workflows → tự động hóa
```

### Vòng đời contact
```
Subscriber → Lead → MQL → SQL → Opportunity → Customer → Evangelist
```

### Quy trình bán hàng mẫu
```
Tạo contact → Tạo deal → Kéo qua các stage
  → Closed Won → Gửi email cảm ơn → Tạo ticket giao hàng
```

### API nhanh
```bash
# Tạo contact
POST /crm/v3/objects/contacts
# Lấy contacts
GET /crm/v3/objects/contacts?properties=email,firstname
# Tạo deal
POST /crm/v3/objects/deals
# Header luôn cần:
Authorization: Bearer YOUR_PRIVATE_APP_TOKEN
```

### Khi cần giúp đỡ
```
Help (góc phải trên ?) → Knowledge Base (tiếng Anh, rất chi tiết)
Community: community.hubspot.com
Academy: academy.hubspot.com (khóa học miễn phí + chứng chỉ!)
```

> 🎓 **Mẹo cuối:** HubSpot Academy có khóa học miễn phí + cấp chứng chỉ (Inbound Marketing, Sales...). Học xong vừa dùng giỏi vừa đẹp CV!

---

*Cập nhật: 2026-10-10*

---

## 21. Sequences — Email Bán Hàng Tự Động Nhiều Bước

**Sequences** (Sales Hub Pro+) khác Workflows ở chỗ: dành cho **sale gửi tay từng email** nhưng được tự động hóa follow-up.

### 21.1 Tạo Sequence

1. Sales → **Sequences** → **Create sequence**
2. Thêm các bước:

```
Bước 1: Gửi email giới thiệu (ngay khi enroll)
  → Delay 2 ngày
Bước 2: Nếu chưa trả lời → Gửi email follow-up #1
  → Delay 3 ngày
Bước 3: Nếu chưa trả lời → Tạo task "Gọi điện"
  → Delay 2 ngày
Bước 4: Nếu chưa trả lời → Gửi email "breakup" (email chia tay)
  → Tự động unenroll
```

3. **Tự dừng khi:** khách trả lời email, đặt meeting, hoặc deal thắng

### 21.2 Sequences vs Workflows

| | Sequences | Workflows |
|---|---|---|
| Gửi từ | Email cá nhân của sale | Email công ty (marketing@) |
| Dừng khi trả lời | ✅ Tự động | Phải setup tay |
| Cá nhân hóa | Cao (như sale tự gửi) | Trung bình |
| Phù hợp | B2B, deal giá trị cao | B2C, số lượng lớn |

---

## 22. Quotes & eSignatures — Báo Giá Chuyên Nghiệp

### 22.1 Tạo Quote

1. Mở deal → tab **Quotes** → **Create quote**
2. Thêm **line items** (sản phẩm):

| Sản phẩm | Số lượng | Đơn giá | Thành tiền |
|---|---|---|---|
| iPhone 15 Pro | 1 | 28.000.000 | 28.000.000 |
| Ốp lưng | 1 | 500.000 | 500.000 |
| Giảm giá 10% | | | -2.850.000 |
| **Tổng** | | | **25.650.000** |

3. Thêm điều khoản thanh toán, hiệu lực báo giá
4. **Publish** → gửi link cho khách

### 22.2 Khách Ký Online

- Khách mở link → xem báo giá → bấm **Accept & Sign**
- Ký tên trên màn hình (chuột/ngón tay)
- Bạn nhận thông báo → deal tự chuyển sang "Closed Won" (nếu setup workflow)

> 📌 Cần Sales Starter trở lên.

---

## 23. Meeting Scheduler — Đặt Lịch Không Cần Nhắn Qua Nhắn Lại

### 23.1 Tạo Link Đặt Lịch

1. Sales → **Meetings** → **Create scheduling page**
2. Cài đặt:
   - Tên: `Tư vấn 30 phút`
   - Thời lượng: 30 phút
   - Giờ làm việc: Thứ 2-6, 8h-17h
   - Buffer: nghỉ 15 phút giữa các cuộc hẹn
3. **Copy link:** `meetings.hubspot.com/ten-ban/tu-van`

### 23.2 Khách Đặt Lịch

- Khách bấm link → chọn giờ trống → điền tên/email → xong
- **Tự động:** tạo contact (nếu chưa có), tạo meeting trong CRM, gửi email xác nhận + link Zoom, nhắc trước 1h

### 23.3 Đồng Bộ Lịch

- Kết nối Google Calendar / Outlook → HubSpot tự tránh giờ bận
- Meeting tạo trong HubSpot → hiện trong Google Calendar và ngược lại

---

## 24. Lists — Phân Đoạn Khách Hàng

### 24.1 Active List vs Static List

| | Active List | Static List |
|---|---|---|
| Cập nhật | Tự động (ai đủ điều kiện → vào, không đủ → ra) | Tay (thêm/xóa thủ công) |
| Ví dụ | "Khách mua trong 30 ngày qua" | "Khách mời sự kiện 10/10" |

### 24.2 Tạo List

1. CRM → **Lists** → **Create list** → **Active**
2. Thêm điều kiện:

```
List "Khách VIP cần chăm sóc":
  Lifecycle stage is Customer
  AND Total revenue > 50.000.000
  AND Last activity date > 30 days ago (lâu chưa tương tác)
```

3. **Save** → dùng list này để gửi email ưu đãi

### 24.3 Dùng List Để Làm Gì?

- Gửi email marketing đúng đối tượng
- Làm điều kiện trigger workflow
- Báo cáo: "có bao nhiêu khách VIP?"
- Xuất ra để chạy quảng cáo (upload lên Facebook Ads)

---

## 25. Ads — Quản Lý Quảng Cáo Trong HubSpot

### 25.1 Kết Nối Tài Khoản Quảng Cáo

Marketing → **Ads** → **Connect account** → chọn:
- Facebook / Instagram Ads
- Google Ads
- LinkedIn Ads

### 25.2 Đồng Bộ Lead

- Lead từ **Facebook Lead Ads** → tự tạo contact trong HubSpot (không cần Zapier!)
- Biết chính xác lead nào đến từ chiến dịch nào → tính ROI

### 25.3 Tạo Audience

- Từ list HubSpot → **Create audience** → đẩy sang Facebook/Google
- Chạy remarketing: "những người xem pricing mà chưa mua"

---

## 26. Knowledge Base — Trung Tâm Trợ Giúp

### 26.1 Tạo Bài Viết Hướng Dẫn

1. Service → **Knowledge Base** → **Create article**
2. Viết: "Cách đổi mật khẩu", "Chính sách bảo hành"...
3. Phân loại: Tài khoản / Thanh toán / Sản phẩm...
4. **Publish** → trang help.congty.com

### 26.2 Lợi Ích

- Khách tự tìm câu trả lời → giảm ticket 30-50%
- Chatbot trích dẫn bài viết khi trả lời
- Nhân viên mới đọc để đào tạo

---

## 27. Surveys — Khảo Sát CSAT / NPS

### 27.1 Tạo Khảo Sát

Service → **Feedback Surveys** → chọn loại:

| Loại | Câu hỏi | Khi nào dùng |
|---|---|---|
| **CSAT** | "Bạn hài lòng với hỗ trợ này không?" (1-5 sao) | Sau mỗi ticket đóng |
| **NPS** | "Bạn giới thiệu chúng tôi cho bạn bè không?" (0-10) | Mỗi quý, cho khách hàng |
| **CES** | "Giải quyết vấn đề có dễ dàng không?" | Sau tương tác hỗ trợ |

### 27.2 Tự Động Hóa

```
Ticket → Closed
  → Gửi khảo sát CSAT (ngay)
  → Nếu 1-2 sao → Tạo ticket mới "Khách không hài lòng" + báo quản lý
  → Nếu 5 sao → Gửi email "Cảm ơn! Giới thiệu bạn bè nhé"
```

---

## 28. Import/Export Dữ Liệu Nâng Cao

### 28.1 Import Đúng Cách

**Chuẩn bị file:**
```csv
Email,First Name,Last Name,Phone,Lifecycle Stage,Nguồn lead
an@example.com,An,Nguyễn,0901111111,Lead,Facebook
```

**Quy trình:**
1. CRM → Import → Upload file
2. **Map cột:** kéo cột file → property HubSpot
3. **Deduplication:** chọn `Email` làm khóa → contact trùng email sẽ **cập nhật** thay vì tạo mới
4. **Review** → Import

### 28.2 Export

- Chọn contacts → **Export** → CSV/Excel
- Dùng để backup, hoặc chuyển sang hệ thống khác

---

## 29. Mobile App HubSpot

### 29.1 Cài Đặt

- Tải **HubSpot** trên App Store / Google Play
- Đăng nhập → đầy đủ CRM trên điện thoại

### 29.2 Dùng Khi Nào

- Sale gặp khách: mở contact xem lịch sử ngay
- Log call sau khi gọi điện xong
- Chụp danh thiếp → app tự nhận diện → tạo contact (business card scanner!)
- Nhận thông báo: deal thắng, email được mở, ticket mới

---

*Cập nhật: 2026-10-10*

---

## 30. Playbooks — Kịch Bản Bán Hàng

**Playbooks** (Sales Pro+) là kịch bản hướng dẫn sale từng bước khi gọi điện/gặp khách.

### 30.1 Tạo Playbook

1. Sales → **Playbooks** → **Create playbook**
2. Viết kịch bản:

```markdown
# Kịch bản gọi cho lead mới

## Mở đầu (1 phút)
- Chào hỏi, giới thiệu tên + công ty
- "Anh/chị có 5 phút không ạ?"

## Tìm hiểu nhu cầu (3 phút)
- [ ] Hỏi: hiện tại đang dùng giải pháp gì?
- [ ] Hỏi: khó khăn lớn nhất là gì?
- [ ] Hỏi: ngân sách dự kiến?

## Giới thiệu giải pháp (3 phút)
- Nêu 3 lợi ích chính phù hợp với nhu cầu vừa nghe
- Đừng liệt kê tính năng!

## Chốt (2 phút)
- [ ] Hỏi: "Anh/chị muốn bắt đầu khi nào?"
- [ ] Đặt lịch demo / gửi báo giá
- [ ] Ghi chú vào HubSpot ngay sau cuộc gọi
```

### 30.2 Dùng Playbook

- Mở contact/deal → tab **Playbooks** → chọn kịch bản
- Tick từng bước khi làm xong → HubSpot ghi log

---

## 31. Forecasting — Dự Báo Doanh Thu

### 31.1 Xem Forecast

Sales → **Forecast** → thấy:

| Cột | Ý nghĩa |
|---|---|
| Pipeline | Tổng giá trị deals đang mở |
| Weighted | Giá trị × xác suất đóng (theo stage) |
| Closed | Đã chốt tháng này |
| Goal | Mục tiêu tháng |

### 31.2 Xác Suất Theo Stage (Ví Dụ)

| Stage | Xác suất |
|---|---|
| Mới | 10% |
| Đang tư vấn | 30% |
| Đã báo giá | 60% |
| Chờ thanh toán | 90% |
| Closed Won | 100% |

Deal 100tr ở stage "Đã báo giá" → đóng góp 60tr vào forecast.

---

## 32. Custom Objects (Nâng Cao)

Ngoài Contacts/Companies/Deals/Tickets, bạn có thể tạo **đối tượng tùy chỉnh**.

> 📌 Cần gói Enterprise.

**Ví dụ:** Công ty cho thuê xe cần quản lý "Xe":

```
Custom Object: Xe
  - Biển số (text)
  - Loại xe (dropdown: 4 chỗ / 7 chỗ / 16 chỗ)
  - Trạng thái (dropdown: sẵn sàng / đang thuê / bảo trì)
  - Liên kết với: Deals (đơn thuê xe), Contacts (khách thuê)
```

---

## 33. Tính Năng AI Của HubSpot (Breeze)

### 33.1 Breeze Copilot

Trợ lý AI trong HubSpot — hỏi bằng tiếng Anh:

```
"Cho tôi xem 5 deals giá trị cao nhất tháng này"
"Tóm tắt lịch sử với contact Nguyễn Văn A"
"Viết email follow-up cho deal XYZ"
```

### 33.2 AI Trong Soạn Email

- **Tạo tiêu đề:** AI gợi ý 5 tiêu đề dựa trên nội dung
- **Viết lại:** chọn đoạn text → AI viết lại cho chuyên nghiệp/ngắn gọn/thân thiện
- **Cá nhân hóa hàng loạt:** AI tự điền thông tin phù hợp từng người nhận

### 33.3 Lead Scoring Tự Động

HubSpot AI tự chấm điểm lead dựa trên:
- Mở email bao nhiêu lần
- Xem pricing page chưa
- Chức danh, quy mô công ty
- Tương tác với chatbot

→ Sale ưu tiên gọi những lead điểm cao trước.

---

## 34. Di Chuyển Từ Hệ Thống Khác Sang HubSpot

### 34.1 Từ Excel/Google Sheets

1. Chuẩn hóa file: 1 dòng = 1 contact, cột rõ ràng
2. CRM → Import → upload
3. Map cột → chọn deduplication key (email)
4. Import → kiểm tra mẫu 10 contacts đầu

### 34.2 Từ CRM Khác (Salesforce, Zoho...)

1. Export từ CRM cũ ra CSV
2. Import vào HubSpot như trên
3. **Quan trọng:** map đúng lifecycle stage, deal stage
4. Chạy song song 2 hệ thống 2 tuần → đảm bảo không mất dữ liệu → mới tắt cũ

### 34.3 Checklist Sau Migration

- [ ] Số contacts khớp (cũ vs mới)
- [ ] Deals còn nguyên giá trị và stage
- [ ] Team đăng nhập được hết
- [ ] Email templates đã chuyển
- [ ] Workflows quan trọng đã tạo lại
- [ ] Tích hợp (Gmail, Slack...) đã kết nối lại

---

*Cập nhật: 2026-10-10*
