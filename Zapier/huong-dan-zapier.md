# Hướng Dẫn Toàn Tập Zapier — Tự Động Hóa Mọi Công Việc

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10

---

## Mục Lục

1. [Zapier Là Gì?](#1-zapier-là-gì)
2. [Tạo Tài Khoản](#2-tạo-tài-khoản)
3. [Khái Niệm Cốt Lõi: Zap, Trigger, Action](#3-khái-niệm-cốt-lõi)
4. [Tạo Zap Đầu Tiên (Từng Bước)](#4-tạo-zap-đầu-tiên)
5. [Các Tích Hợp Phổ Biến](#5-các-tích-hợp-phổ-biến)
6. [Multi-Step Zaps](#6-multi-step-zaps)
7. [Paths (Điều Kiện Rẽ Nhánh)](#7-paths)
8. [Filter (Bộ Lọc)](#8-filter)
9. [Formatter (Định Dạng Dữ Liệu)](#9-formatter)
10. [Webhooks](#10-webhooks)
11. [Delay, Schedule, Digest](#11-delay-schedule-digest)
12. [Tables & Interfaces](#12-tables--interfaces)
13. [Quản Lý Zaps](#13-quản-lý-zaps)
14. [Task Là Gì? Giới Hạn](#14-task-là-gì-giới-hạn)
15. [Bảng Giá Chi Tiết](#15-bảng-giá-chi-tiết)
16. [Zapier vs Đối Thủ](#16-zapier-vs-đối-thủ)
17. [Xử Lý Lỗi Thường Gặp](#17-xử-lý-lỗi-thường-gặp)
18. [Best Practices](#18-best-practices)
19. [Cheat Sheet](#19-cheat-sheet)

---

## 1. Zapier Là Gì?

### 1.1 Định Nghĩa Đơn Giản

**Zapier** là nền tảng tự động hóa "không cần code" (no-code) giúp các ứng dụng web khác nhau "nói chuyện" với nhau.

Thay vì bạn phải làm thủ công: *"có email mới → copy nội dung → dán vào Google Sheets → gửi tin nhắn Slack báo cho team"* — Zapier làm tất cả tự động, 24/7.

### 1.2 Ví Dụ Thực Tế

| Công việc thủ công | Zapier tự động |
|---|---|
| Khách điền form → bạn copy email vào danh sách | Form submit → tự thêm vào Mailchimp |
| Có đơn hàng mới → bạn nhắn tin báo team | Đơn hàng mới → tự gửi Slack + SMS |
| Bài blog mới → bạn đăng tay lên Facebook, Twitter, LinkedIn | Đăng blog → tự đăng lên mọi mạng xã hội |
| File đính kèm email → bạn tải về lưu Drive | Email có file → tự lưu vào Google Drive |
| Lead mới → bạn nhập tay vào CRM | Lead mới → tự tạo contact trong HubSpot |

### 1.3 Zapier Kết Nối Được Bao Nhiêu App?

- **7.000+ ứng dụng** (tính đến 2026): Gmail, Slack, Google Sheets, Notion, Trello, Shopify, Stripe, Facebook, Twitter/X, Discord, Telegram, Airtable, HubSpot, Salesforce...
- Nếu app bạn dùng có API, gần như chắc chắn Zapier hỗ trợ (hoặc dùng Webhooks để tự kết nối).

### 1.4 Ai Nên Dùng Zapier?

- **Freelancer / chủ shop online:** tự động hóa đơn hàng, khách hàng, marketing
- **Marketer:** đồng bộ lead giữa các nền tảng, tự đăng bài
- **Team nhỏ:** giảm việc lặp đi lặp lại, tiết kiệm hàng giờ mỗi tuần
- **Người không biết code:** mọi thứ kéo-thả, không cần lập trình

### 1.5 Zapier Không Phải Là Gì?

- ❌ Không phải công cụ lập trình (dù có Code step cho dev)
- ❌ Không thay thế được logic phức tạp (vòng lặp, xử lý dữ liệu lớn) — lúc đó cần Make, n8n, hoặc code thật
- ❌ Không miễn phí hoàn toàn cho nhu cầu lớn (giới hạn task, xem phần 14)

---

## 2. Tạo Tài Khoản

### 2.1 Đăng Ký

1. Vào **https://zapier.com**
2. Bấm **Sign up** (góc phải trên)
3. Đăng ký bằng **Google**, **Facebook**, hoặc **email + mật khẩu**
4. Trả lời vài câu hỏi khảo sát (vai trò, công ty, app hay dùng) — Zapier sẽ gợi ý Zaps phù hợp

### 2.2 Giao Diện Dashboard

Sau khi đăng nhập, bạn thấy:

| Khu vực | Chức năng |
|---|---|
| **Home** | Gợi ý Zap theo app bạn dùng, template phổ biến |
| **Zaps** | Danh sách tất cả Zap bạn đã tạo |
| **Tables** | Bảng dữ liệu lưu trữ trong Zapier |
| **Interfaces** | Tạo form/trang web mini |
| **Apps** | Quản lý các app đã kết nối |
| **History** | Lịch sử chạy của từng Zap (thành công/lỗi) |

### 2.3 Kết Nối App Lần Đầu

Khi tạo Zap, Zapier sẽ yêu cầu kết nối app:

1. Chọn app (ví dụ: Gmail)
2. Bấm **Sign in** → cửa sổ OAuth hiện ra
3. Đăng nhập tài khoản Gmail → **Cho phép** (Allow)
4. Xong! Kết nối được lưu, lần sau không cần đăng nhập lại

> 💡 **Mẹo:** Vào **Apps** (menu trái) để xem/quản lý tất cả kết nối. Có thể kết nối **nhiều tài khoản** cùng 1 app (ví dụ: 2 tài khoản Gmail khác nhau).

---

## 3. Khái Niệm Cốt Lõi

### 3.1 Zap

**Zap** = 1 quy trình tự động hóa hoàn chỉnh. Cấu trúc:

```
TRIGGER (kích hoạt) → ACTION 1 → ACTION 2 → ... (tùy chọn)
```

Ví dụ Zap "Báo đơn hàng mới":

```
[Shopify: Đơn hàng mới] → [Slack: Gửi tin nhắn #don-hang] → [Google Sheets: Thêm dòng mới]
   TRIGGER                        ACTION 1                          ACTION 2
```

### 3.2 Trigger (Điều Kiện Kích Hoạt)

Trigger là **sự kiện** khiến Zap chạy. Mỗi Zap có **đúng 1 trigger**.

Các loại trigger phổ biến:

| Loại | Ý nghĩa | Ví dụ |
|---|---|---|
| **New Item** | Có dữ liệu mới | Email mới, đơn hàng mới, dòng mới trong Sheets |
| **Updated Item** | Dữ liệu bị sửa | Contact được cập nhật trong CRM |
| **New/Updated** | Mới hoặc sửa | Task mới hoặc đổi trạng thái trong Trello |
| **Schedule** | Theo lịch | Mỗi sáng 8h, mỗi thứ 2 hàng tuần |
| **Webhook** | Nhận HTTP request | App khác gọi vào URL của Zapier |

**Polling vs Instant:**

| Loại | Cách hoạt động | Độ trễ |
|---|---|---|
| **Instant** (webhook) | App gửi tín hiệu ngay khi có sự kiện | Vài giây |
| **Polling** | Zapier "hỏi" app mỗi X phút có gì mới không | 1-15 phút (tùy gói) |

> ⚠️ Gói Free chỉ có polling 15 phút. Muốn instant (chạy ngay) cần gói trả phí.

### 3.3 Action (Hành Động)

Action là **việc Zapier làm** sau khi trigger kích hoạt. Một Zap có thể có **nhiều action** nối tiếp nhau.

Ví dụ actions của Gmail:
- Send Email (gửi email)
- Create Draft (tạo nháp)
- Add Label (gắn nhãn)

Ví dụ actions của Google Sheets:
- Create Spreadsheet Row (thêm dòng)
- Update Spreadsheet Row (sửa dòng)
- Lookup Spreadsheet Row (tìm dòng)

### 3.4 Dữ Liệu Chảy Giữa Các Bước (Mapping)

Điểm mạnh nhất của Zapier: **dữ liệu từ bước trước có thể dùng ở bước sau**.

Ví dụ:

```
Bước 1 (Trigger): Shopify — Đơn hàng mới
  → Dữ liệu có: tên khách, email, tổng tiền, sản phẩm

Bước 2 (Action): Slack — Gửi tin nhắn
  → Nội dung: "🛒 Đơn mới từ {tên khách}: {tổng tiền}đ — {sản phẩm}"
```

Khi setup action, bạn bấm vào ô nhập liệu → chọn **dữ liệu từ bước trước** (hiện dưới dạng pill màu). Zapier tự thay giá trị thật mỗi lần chạy.

### 3.5 Task

**Task** = 1 lần thực hiện thành công của 1 action step.

```
1 Zap chạy = 1 trigger + N actions = N tasks
```

Ví dụ: Zap có 1 trigger + 3 actions, chạy 10 lần/tháng = **30 tasks/tháng**.

> ⚠️ Trigger **không** tính task. Chỉ action thành công mới tính. Filter/Formatter (phần 8, 9) cũng tính task!

---

## 4. Tạo Zap Đầu Tiên (Từng Bước Chi Tiết)

**Mục tiêu:** Khi có email mới trong Gmail → tự động gửi tin nhắn vào Slack.

### Bước 1: Bấm "Create Zap"

- Ở dashboard, bấm nút **+ Create** → **Zaps** (hoặc nút **Create Zap** lớn ở trang Home)

### Bước 2: Setup Trigger

1. Ô **Trigger**: gõ `Gmail` → chọn app **Gmail**
2. **Trigger event**: chọn **New Email** (email mới)
3. **Account**: bấm **Sign in** → đăng nhập Gmail → Allow
4. **Trigger setup** (tùy chọn):
   - *Label/Mailbox*: chọn `INBOX` (hoặc label cụ thể như `Đơn hàng`)
   - *Search String*: ví dụ `subject:"đơn hàng"` (chỉ trigger với email có tiêu đề chứa "đơn hàng")
5. **Test trigger**: bấm **Test trigger** → Zapier lấy 3 email gần nhất làm mẫu
6. Chọn 1 email mẫu → **Continue with selected record**

### Bước 3: Setup Action

1. Ô **Action**: gõ `Slack` → chọn app **Slack**
2. **Action event**: chọn **Send Channel Message**
3. **Account**: Sign in Slack → chọn workspace → Allow
4. **Setup action**:
   - *Channel*: chọn kênh, ví dụ `#thong-bao`
   - *Message Text*: gõ nội dung, chèn dữ liệu từ Gmail:
     ```
     📧 Email mới từ {From Name} ({From Email})
     Tiêu đề: {Subject}
     Nội dung: {Body Plain}
     ```
     (Bấm vào ô → chọn các pill dữ liệu từ bước Gmail)
   - *Bot Name*: `Zapier Bot` (tùy chọn)
5. **Test**: bấm **Test step** → kiểm tra Slack có tin nhắn chưa

### Bước 4: Publish

1. Bấm **Publish** (góc phải trên)
2. Đặt tên Zap: ví dụ `Gmail → Slack: báo email mới`
3. Xong! Zap đang **ON** và tự chạy

### Bước 5: Kiểm Tra

- Gửi 1 email test vào Gmail
- Đợi (Free: tối đa 15 phút; trả phí: gần như ngay)
- Kiểm tra Slack có tin nhắn không
- Vào **History** để xem log chạy

---

## 5. Các Tích Hợp Phổ Biến

### 5.1 Theo Nhóm

**Email & Giao tiếp:**
| App | Trigger hay dùng | Action hay dùng |
|---|---|---|
| Gmail | New Email, New Labeled Email | Send Email, Create Draft |
| Slack | New Message, New Channel | Send Message, Create Channel |
| Discord | New Message | Send Channel Message |
| Telegram | — | Send Message |

**Bảng tính & Dữ liệu:**
| App | Trigger hay dùng | Action hay dùng |
|---|---|---|
| Google Sheets | New Row, Updated Row | Create Row, Update Row, Lookup |
| Airtable | New Record | Create/Update Record |
| Notion | New Database Item | Create Page, Update Page |

**Bán hàng & CRM:**
| App | Trigger hay dùng | Action hay dùng |
|---|---|---|
| Shopify | New Order, New Customer | Create Product |
| WooCommerce | New Order | — |
| Stripe | New Payment | Create Customer |
| HubSpot | New Contact, New Deal | Create Contact/Deal |

**Quản lý công việc:**
| App | Trigger hay dùng | Action hay dùng |
|---|---|---|
| Trello | New Card, Card Moved | Create Card, Add Comment |
| Asana | New Task | Create Task |
| Google Calendar | New Event | Create Event |
| Todoist | New Task | Create Task |

**Marketing:**
| App | Trigger hay dùng | Action hay dùng |
|---|---|---|
| Mailchimp | New Subscriber | Add Subscriber |
| Facebook Pages | New Post | Create Post |
| WordPress | New Post | Create Post |

### 5.2 Tìm App

- Vào **Apps** → gõ tên app → xem tất cả trigger/action nó hỗ trợ
- Nếu app chưa có: dùng **Webhooks** (phần 10) hoặc yêu cầu tại zapier.com/app-directory

### 5.3 Kết Nối Nhiều Tài Khoản

Ví dụ: bạn có 2 Gmail (cá nhân + công việc):

1. Khi setup step, ở phần Account bấm **+ Connect a new account**
2. Đăng nhập tài khoản thứ 2
3. Lần sau chọn tài khoản nào cũng được trong dropdown

---

## 6. Multi-Step Zaps

### 6.1 Thêm Nhiều Actions

Sau action đầu tiên, bấm **+** để thêm step tiếp theo. Dữ liệu từ **tất cả** bước trước đều dùng được.

**Ví dụ Zap 4 bước — Xử lý đơn hàng:**

```
1. [Trigger] Shopify — New Order
     ↓ (dữ liệu: tên, email, sđt, sản phẩm, tổng tiền)
2. [Action] Google Sheets — Create Row
     → Thêm vào sheet "Đơn hàng": ngày, tên, sđt, tổng tiền
3. [Action] Slack — Send Message → #don-hang
     → "🛒 {tên} vừa đặt {sản phẩm} — {tổng tiền}đ"
4. [Action] Gmail — Send Email
     → Gửi email cảm ơn tới {email khách}
```

### 6.2 Sắp Xếp Lại Steps

- Di chuột vào step → kéo thả để đổi thứ tự
- ⚠️ Cẩn thận: step sau dùng dữ liệu step trước — đổi thứ tự có thể làm hỏng mapping!

### 6.3 Nhân Bản Step

- Bấm `⋯` ở step → **Duplicate**: copy step (hữu ích khi làm nhiều action tương tự)

---

## 7. Paths (Điều Kiện Rẽ Nhánh)

**Paths** cho phép Zap đi theo hướng khác nhau tùy điều kiện (như if/else).

> 📌 Cần gói **Professional** trở lên.

### 7.1 Ví Dụ

```
[Trigger] Form — Khách điền form liên hệ
    ↓
[Paths]
  ├─ Path A: IF {loại khách} = "mua sỉ"
  │    → [Action] Gửi email báo giá sỉ
  │    → [Action] Tạo deal trong CRM (giá trị cao)
  │
  └─ Path B: IF {loại khách} = "mua lẻ"
       → [Action] Gửi email cảm ơn + mã giảm giá
```

### 7.2 Cách Setup

1. Thêm step → chọn **Paths** (trong Built-in tools)
2. Đặt tên Path A, thêm điều kiện: `{trường dữ liệu}` `Exactly matches` `mua sỉ`
3. Thêm actions vào trong Path A
4. Bấm **+ Add path** cho Path B, C...
5. **Fallback**: bật để xử lý trường hợp không khớp path nào

### 7.3 Lưu Ý

- Mỗi Zap tối đa **5 paths** (tùy gói)
- Điều kiện dùng **AND/OR** kết hợp được
- Paths lồng nhau không được (không có path trong path)

---

## 8. Filter (Bộ Lọc)

**Filter** chặn Zap chạy tiếp nếu không đủ điều kiện — tiết kiệm task!

### 8.1 Ví Dụ

```
[Trigger] Gmail — New Email (mọi email đều trigger)
    ↓
[Filter] Chỉ tiếp tục nếu {Subject} Contains "KHẨN"
    ↓ (email thường → dừng, KHÔNG tốn task cho các step sau)
[Action] Slack — Gửi tin nhắn báo động
```

Không có Filter: mọi email đều gửi Slack (ồn ào + tốn task).
Có Filter: chỉ email khẩn mới báo.

### 8.2 Cách Setup

1. Thêm step → **Filter** (Built-in)
2. Chọn trường dữ liệu, điều kiện, giá trị:

| Điều kiện | Ý nghĩa |
|---|---|
| Exactly matches | Bằng chính xác |
| Contains | Chứa chuỗi |
| Does not contain | Không chứa |
| Starts with / Ends with | Bắt đầu/kết thúc bằng |
| Greater than / Less than | So sánh số |
| Exists / Does not exist | Có dữ liệu hay không |

3. Kết hợp nhiều điều kiện bằng **AND** (tất cả đúng) / **OR** (1 cái đúng)

### 8.3 Filter vs Paths

| | Filter | Paths |
|---|---|---|
| Mục đích | Chặn hoặc cho qua | Rẽ nhiều hướng |
| Khi không khớp | Dừng Zap | Đi path khác/fallback |
| Tốn task | Có (1 task cho step filter) | Có |

> 💡 **Mẹo tiết kiệm task:** Đặt Filter **ngay sau Trigger** để Zap dừng sớm nhất có thể.

---

## 9. Formatter (Định Dạng Dữ Liệu)

**Formatter** biến đổi dữ liệu: đổi định dạng ngày, tách text, tính toán...

> Mỗi lần chạy Formatter tốn 1 task.

### 9.1 Text

| Chức năng | Ví dụ |
|---|---|
| Uppercase/Lowercase/Title Case | `nguyen van a` → `Nguyen Van A` |
| Split Text | `"An, Bình, Chi"` → tách thành 3 mục |
| Replace | Thay `http://` → `https://` |
| Truncate | Cắt còn 100 ký tự + `...` |
| Extract Email/URL/Phone | Lấy email từ đoạn text dài |
| Remove HTML | Xóa thẻ HTML khỏi nội dung |

**Ví dụ thực tế:** Tên khách từ form viết lung tung (`nGuYeN vAn A`) → Formatter Title Case → `Nguyen Van A` trước khi lưu CRM.

### 9.2 Date/Time

| Chức năng | Ví dụ |
|---|---|
| Format | `2026-10-10` → `10/10/2026` hoặc `Thứ Bảy, 10 tháng 10` |
| Add/Subtract Time | Cộng 7 ngày vào ngày đặt hàng (= hạn giao) |
| Compare Dates | Kiểm tra deadline đã qua chưa |

### 9.3 Numbers

- Thực hiện phép tính: cộng/trừ/nhân/chia
- Làm tròn, định dạng tiền tệ: `1500000` → `1.500.000đ`

### 9.4 Utilities

| Chức năng | Dùng khi nào |
|---|---|
| Lookup Table | Đổi mã thành tên: `HN` → `Hà Nội` |
| Pick from List | Lấy phần tử đầu/cuối từ danh sách |
| Line Items | Xử lý danh sách sản phẩm trong đơn hàng |

---

## 10. Webhooks

**Webhooks** giúp Zapier nhận/gửi HTTP request — kết nối với **bất kỳ** hệ thống nào có API.

### 10.1 Catch Hook (Nhận Dữ Liệu Vào Zapier)

Biến Zapier thành 1 API endpoint:

1. Trigger → chọn **Webhooks by Zapier** → **Catch Hook**
2. Zapier tạo URL duy nhất: `https://hooks.zapier.com/hooks/catch/123456/abcdef/`
3. Hệ thống khác POST dữ liệu vào URL này → Zap chạy

**Ví dụ:** Website của bạn (tự code) có form đăng ký:

```javascript
// Khi khách submit form trên website
fetch('https://hooks.zapier.com/hooks/catch/123456/abcdef/', {
  method: 'POST',
  headers: {'Content-Type': 'application/json'},
  body: JSON.stringify({
    name: 'Nguyễn Văn A',
    email: 'a@example.com',
    phone: '0901234567'
  })
});
```

→ Zap trigger → gửi email chào mừng + lưu vào Sheets + báo Slack.

### 10.2 Custom Actions (Gọi API Ra Ngoài)

Dùng khi app bạn cần chưa có trên Zapier:

1. Action → **Webhooks by Zapier** → chọn method:
   - **GET**: lấy dữ liệu
   - **POST**: tạo mới
   - **PUT/PATCH**: cập nhật
   - **DELETE**: xóa
   - **Custom Request**: tự viết toàn bộ
2. Điền URL, headers, body

**Ví dụ:** Gửi SMS qua API của nhà mạng:

```
Method: POST
URL: https://api.sms-provider.com/send
Headers:
  Authorization: Bearer YOUR_API_TOKEN
  Content-Type: application/json
Body:
{
  "to": "{số điện thoại từ trigger}",
  "message": "Cảm ơn {tên} đã đặt hàng!"
}
```

### 10.3 Lưu Ý Bảo Mật

- ⚠️ URL Catch Hook là **bí mật** — ai có URL đều trigger được Zap của bạn
- Không để URL webhook trong code public (GitHub public)
- Dùng **Catch Hook with key** hoặc kiểm tra secret trong body nếu cần bảo mật cao

---

## 11. Delay, Schedule, Digest

### 11.1 Delay (Trì Hoãn)

Chờ một khoảng thời gian trước khi chạy step tiếp theo.

| Loại Delay | Ví dụ dùng |
|---|---|
| **Delay For** | Chờ 2 giờ rồi gửi email follow-up |
| **Delay Until** | Chờ đến 9h sáng mai mới gửi (tránh làm phiền đêm) |
| **Delay After Queue** | Xếp hàng: mỗi lần chạy cách nhau 10 phút (tránh spam API) |

**Ví dụ Zap chăm sóc khách:**

```
[Trigger] Khách mua hàng
  → [Action] Gửi email cảm ơn (ngay)
  → [Delay] Chờ 3 ngày
  → [Action] Gửi email xin đánh giá
  → [Delay] Chờ 7 ngày
  → [Action] Gửi mã giảm giá mua lại
```

> 📌 Cần gói **Professional** trở lên. Delay tối đa 30 ngày.

### 11.2 Schedule (Trigger Theo Lịch)

Thay vì chờ sự kiện, Zap chạy theo lịch:

1. Trigger → **Schedule by Zapier**
2. Chọn tần suất:

| Tần suất | Ví dụ |
|---|---|
| Every Hour / Day / Week / Month | Mỗi ngày lúc 8h |
| Custom | Thứ 2, 4, 6 lúc 9h30 |

**Ví dụ dùng:**
- Mỗi sáng 8h: lấy báo cáo doanh thu hôm qua → gửi Slack
- Mỗi thứ 2: tạo task tuần mới trong Asana
- Ngày 1 hàng tháng: gửi email nhắc thanh toán

### 11.3 Digest (Gom Nhiều Thành Một)

Thay vì gửi 20 tin nhắn cho 20 đơn hàng, **gom thành 1 bản tổng hợp**.

```
[Trigger] Shopify — New Order
  → [Digest] Gom đơn hàng vào bản tin
  → [Schedule] Mỗi tối 20h
  → [Action] Slack: gửi 1 tin nhắn "Hôm nay có 20 đơn, tổng 15tr"
```

---

## 12. Tables & Interfaces

### 12.1 Tables (Bảng Dữ Liệu Của Zapier)

Lưu trữ dữ liệu ngay trong Zapier, không cần Google Sheets.

**Khi nào dùng:**
- Lưu trạng thái giữa các lần chạy Zap
- Bảng tra cứu nhỏ (mã → tên)
- Không muốn phụ thuộc Google Sheets

**Ví dụ:** Lưu `email khách → đã gửi mã giảm giá chưa` để tránh gửi trùng.

### 12.2 Interfaces (Tạo Form/Trang Web)

Tạo form nhập liệu, trang dashboard mini mà không cần code.

**Ví dụ:**
- Form "Đăng ký tư vấn" trên web → submit → trigger Zap
- Trang cho nhân viên nhập liệu → lưu vào Tables

> 📌 Tính năng nâng cao, cần gói trả phí.

---

## 13. Quản Lý Zaps

### 13.1 Bật/Tắt Zap

- Ở danh sách Zaps, gạt công tắc **ON/OFF**
- Tắt khi đang sửa để tránh chạy nhầm

### 13.2 History (Lịch Sử Chạy)

Vào **History** (menu trái) để xem:

| Cột | Ý nghĩa |
|---|---|
| Status | Success / Filtered / Error / Held |
| Zap | Zap nào chạy |
| Time | Thời gian chạy |

Bấm vào 1 dòng để xem **chi tiết từng step**: dữ liệu vào/ra — cực hữu ích khi debug!

### 13.3 Replay (Chạy Lại)

Zap bị lỗi? Vào History → chọn lần chạy lỗi → **Replay**: chạy lại với cùng dữ liệu.

### 13.4 Folder (Thư Mục)

- Tạo folder để nhóm Zaps: `Bán hàng`, `Marketing`, `Nội bộ`
- Kéo thả Zap vào folder

### 13.5 Sharing & Team

- Gói Team trở lên: chia sẻ Zap cho thành viên, phân quyền
- **Share Zap template**: tạo link template cho người khác dùng

### 13.6 Version History

- Zapier tự lưu các phiên bản khi bạn sửa
- **Zaps** → chọn Zap → **Version history** → khôi phục bản cũ nếu sửa hỏng

---

## 14. Task Là Gì? Giới Hạn

### 14.1 Cách Tính Task (Nhắc Lại)

- 1 action chạy thành công = 1 task
- Trigger, Filter chặn (không qua), Paths không tính riêng — nhưng các step Formatter, Delay, Webhooks **có tính**

**Ví dụ tính:**

```
Zap A: 1 trigger + 2 actions, chạy 100 lần/tháng
→ 200 tasks/tháng

Zap B: 1 trigger + 1 filter + 3 actions, chạy 100 lần/tháng
→ 400 tasks/tháng (filter cũng tính!)
```

### 14.2 Giới Hạn Theo Gói

| Gói | Tasks/tháng | Zaps | Steps/Zap |
|---|---|---|---|
| Free | 100 | 5 | 2 |
| Starter | 750 | 20 | Multi-step |
| Professional | 2.000+ | Không giới hạn | Không giới hạn |
| Team | 2.000+ | Không giới hạn | Không giới hạn |

### 14.3 Hết Task Thì Sao?

- Zap **tạm dừng** đến tháng sau (reset vào ngày billing)
- Hoặc mua thêm task / nâng gói
- Vào **Settings** → **Billing** để xem đã dùng bao nhiêu

### 14.4 Mẹo Tiết Kiệm Task

1. **Filter ngay sau trigger** — dừng sớm, đỡ tốn
2. **Gom actions**: 1 Formatter làm nhiều việc thay vì 3 Formatter
3. **Dùng Digest** thay vì gửi từng cái
4. **Tắt Zap không cần thiết**
5. **Polling interval**: trigger ít quan trọng thì để polling lâu (ít check = ít tốn? — thực ra polling không tốn task, chỉ action mới tốn)

---

## 15. Bảng Giá Chi Tiết

> 💲 Giá USD, có thể thay đổi — kiểm tra tại zapier.com/pricing

| Gói | Giá/tháng (năm) | Tasks | Zaps | Điểm nổi bật |
|---|---|---|---|---|
| **Free** | $0 | 100 | 5 | 2-step, polling 15 phút |
| **Starter** | ~$20 | 750 | 20 | Multi-step, polling nhanh hơn |
| **Professional** | ~$50 | 2.000 | ∞ | Instant trigger, Paths, Webhooks, Delay |
| **Team** | ~$69 | 2.000 | ∞ | + chia sẻ team, folder chung |
| **Enterprise** | Liên hệ | Tùy chỉnh | ∞ | SSO, audit log, hỗ trợ riêng |

### 15.1 Nên Chọn Gói Nào?

| Nhu cầu | Gói phù hợp |
|---|---|
| Thử nghiệm, 1-2 Zap đơn giản | Free |
| Shop nhỏ, vài chục đơn/ngày | Starter |
| Cần Webhooks, Paths, chạy ngay | Professional |
| Team nhiều người cùng quản lý | Team |

### 15.2 Mẹo Tiết Kiệm

- Trả theo **năm** rẻ hơn ~20% so với theo tháng
- Bắt đầu từ Free → khi nào hết task mới nâng cấp
- Xóa Zap thừa trước khi nâng gói

---

## 16. Zapier vs Đối Thủ

| Tiêu chí | **Zapier** | Make (Integromat) | n8n | Pipedream |
|---|---|---|---|---|
| Dễ dùng | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Số app | 7.000+ | 2.000+ | 400+ | 2.500+ |
| Logic phức tạp | Hạn chế | Tốt (visual) | Rất tốt | Tốt (code) |
| Cần biết code | Không | Không | Ít | Có (JS/Python) |
| Tự host | Không | Không | ✅ Có | Không |
| Giá rẻ nhất | $20 | $9 | Free (tự host) | Free tier rộng |
| Phù hợp | Người mới, team | Giá rẻ, logic vừa | Dev, tự host | Developer |

**Kết luận nhanh:**
- **Mới bắt đầu, không biết code** → Zapier
- **Muốn rẻ + logic phức tạp hơn** → Make
- **Dev, muốn tự host miễn phí** → n8n
- **Dev, thích viết code** → Pipedream

---

## 17. Xử Lý Lỗi Thường Gặp

| Lỗi / Vấn đề | Nguyên nhân | Cách khắc phục |
|---|---|---|
| Zap không trigger | Polling chưa đến lượt (Free: 15 phút) | Đợi, hoặc test bằng "Test trigger" |
| `Authentication failed` | Token hết hạn, đổi mật khẩu app | Vào Apps → Reconnect |
| `Required field missing` | Thiếu dữ liệu bắt buộc ở action | Kiểm tra mapping, đảm bảo trigger có dữ liệu đó |
| `Rate limit exceeded` | Gọi API app quá nhanh | Thêm Delay After Queue, hoặc giảm tần suất |
| Dữ liệu sai / trống | Mapping nhầm field | Vào History → xem data từng step → sửa mapping |
| Zap chạy 2 lần | Trigger nhận trùng (polling) | Dùng Filter loại trùng, hoặc chuyển sang instant trigger |
| Hết task giữa tháng | Dùng quá giới hạn | Tắt bớt Zap, mua thêm task, nâng gói |
| Webhook không nhận | Sai URL, sai method | Kiểm tra URL, dùng webhook.site để test |
| Formatter lỗi định dạng | Dữ liệu đầu vào không đúng kiểu | Thêm Filter kiểm tra trước, hoặc dùng "Default value" |

### 17.1 Debug Theo Quy Trình

1. Vào **History** → tìm lần chạy lỗi (màu đỏ)
2. Bấm vào → xem **từng step**, step nào đỏ là lỗi ở đó
3. Đọc **error message** — thường ghi rõ nguyên nhân
4. Xem **Data In / Data Out** để biết dữ liệu vào ra
5. Sửa Zap → **Replay** để chạy lại

---

## 18. Best Practices

### 18.1 Thiết Kế Zap

1. **Đặt tên rõ ràng:** `Gmail → Slack: báo email khẩn` (không để `My Zap #3`)
2. **Mô tả trong Notes:** ghi chú Zap làm gì, ai tạo, ngày tạo
3. **1 Zap = 1 nhiệm vụ:** đừng nhồi 10 việc vào 1 Zap — khó debug
4. **Filter sớm:** đặt ngay sau trigger
5. **Test từng step** trước khi Publish

### 18.2 Bảo Mật

1. ⚠️ Không để API key/token trong tên Zap hoặc mô tả public
2. Dùng **Secrets** cho thông tin nhạy cảm trong Webhooks
3. Kiểm tra quyền app kết nối — chỉ cho phép cái cần thiết
4. Khi nhân viên nghỉ: vào Apps → ngắt kết nối tài khoản của họ

### 18.3 Vận Hành

1. **Theo dõi History** hàng tuần — phát hiện lỗi sớm
2. **Bật email thông báo lỗi:** Settings → Notifications
3. **Backup ý tưởng:** ghi lại danh sách Zap quan trọng (nếu mất tài khoản còn làm lại được)
4. **Dọn dẹp định kỳ:** tắt/xóa Zap không dùng — đỡ tốn task, đỡ rối

### 18.4 Tối Ưu Chi Phí

1. Bắt đầu Free → đo nhu cầu thật → mới nâng gói
2. Gom nhiều việc vào ít Zap hơn (dùng Paths thay vì nhiều Zap)
3. Dùng Schedule + Digest thay vì trigger liên tục
4. Đánh giá lại mỗi quý: có Zap nào không còn cần không?

---

## 19. Cheat Sheet

### Khái niệm
```
Zap     = 1 quy trình tự động (trigger + actions)
Trigger = sự kiện kích hoạt (1 cái duy nhất)
Action  = việc cần làm (nhiều cái được)
Task    = 1 action chạy thành công (đơn vị tính tiền)
```

### Built-in Tools (không cần app ngoài)
```
Filter    → chặn/cho qua theo điều kiện
Paths     → rẽ nhánh if/else (tối đa 5)
Formatter → đổi định dạng text/date/number
Delay     → chờ (For / Until / After Queue)
Schedule  → trigger theo lịch
Digest    → gom nhiều thành 1 bản tin
Webhooks  → nhận/gửi HTTP request
```

### Phím tắt / thao tác nhanh
```
Tạo Zap mới          → nút + Create → Zaps
Test từng step       → nút "Test step" trong mỗi step
Xem lịch sử chạy     → menu History
Chạy lại khi lỗi     → History → Replay
Nhân bản Zap        → ⋯ → Duplicate
Tắt/mở Zap          → công tắc ON/OFF
```

### Công thức Zap phổ biến
```
Bán hàng:    Shopify trigger → Sheets + Slack + Email cảm ơn
Marketing:   Form submit → CRM + Mailchimp + Slack báo
Nội bộ:      Schedule 8h → lấy báo cáo → gửi Slack
Chăm sóc:    Mua hàng → Delay 3 ngày → xin đánh giá
```

---

*Cập nhật: 2026-10-10*

---

## 20. 10 Mẫu Zap Thực Tế (Copy Ý Tưởng)

### 20.1 Shop Online

```
1. Đơn hàng mới (Shopify)
   → Lưu vào Google Sheets (theo dõi)
   → Gửi Slack #don-hang (báo team)
   → Gửi email cảm ơn khách (Gmail)
   → Delay 3 ngày → Gửi email xin đánh giá
```

### 20.2 Tuyển Dụng / Nhận Lead

```
2. Form "Đăng ký tư vấn" (Google Forms / Typeform)
   → Tạo contact trong HubSpot
   → Gửi Slack #lead-moi cho sale
   → Thêm vào Mailchimp (danh sách "Lead mới")
   → Tạo task trong Trello/Asana (giao cho sale follow-up)
```

### 20.3 Content Marketing

```
3. Bài blog mới (WordPress)
   → Đăng lên Facebook Page
   → Đăng lên LinkedIn
   → Rút gọn link (Bitly) → đăng Twitter/X
   → Lưu vào Google Sheets (tracking content)
```

### 20.4 Chăm Sóc Khách Hàng

```
4. Ticket mới (Zendesk / Freshdesk)
   → Tạo task ưu tiên trong Asana
   → Gửi Slack #ho-tro (báo team)
   → Nếu quá 2h chưa xử lý → gửi email nhắc trưởng nhóm
```

### 20.5 Tài Chính / Kế Toán

```
5. Thanh toán mới (Stripe)
   → Tạo hóa đơn trong QuickBooks/Xero
   → Gửi email biên lai cho khách
   → Lưu vào Sheets "Doanh thu"
   → Cuối tháng: Digest → gửi báo cáo tổng cho kế toán
```

### 20.6 Nhân Sự

```
6. Nhân viên mới (BambooHR / Google Forms)
   → Tạo tài khoản Google Workspace
   → Gửi email chào mừng + tài liệu onboarding
   → Tạo task checklist trong Asana
   → Thêm vào Slack #chao-mung
```

### 20.7 Bất Động Sản

```
7. Khách điền form xem nhà
   → Tạo deal trong CRM
   → Gửi SMS xác nhận lịch hẹn (qua Webhooks + SMS API)
   → Tạo event Google Calendar
   → Trước hẹn 2h → gửi SMS nhắc
```

### 20.8 Giáo Dục / Khóa Học

```
8. Học viên đăng ký (Form)
   → Tạo tài khoản trên Teachable/Thinkific
   → Gửi email chào mừng + hướng dẫn
   → Thêm vào nhóm Facebook/Discord
   → Delay 7 ngày → gửi khảo sát hài lòng
```

### 20.9 Quản Lý Nội Bộ

```
9. Mỗi sáng thứ 2 (Schedule)
   → Lấy task quá hạn từ Asana/Trello
   → Gửi Slack #team: "Tuần này còn X task quá hạn"
   → Tạo Google Docs báo cáo tuần
```

### 20.10 Backup Tự Động

```
10. File mới trong Dropbox (thư mục quan trọng)
    → Copy sang Google Drive (backup chéo)
    → Ghi log vào Sheets (ngày, tên file, dung lượng)
```

---

## 21. Zapier Tables Chi Tiết

### 21.1 Tạo Bảng

1. Menu **Tables** → **+ Create table**
2. Đặt tên: ví dụ `Khách hàng VIP`
3. Thêm cột: Tên (text), Email (email), Tổng chi (number), Ngày mua cuối (date)

### 21.2 Dùng Trong Zap

**Trigger:** `Tables` → **New Record** — khi có dòng mới trong bảng → chạy Zap

**Action:** `Tables` → **Create Record / Update Record / Find Record**

**Ví dụ:** Lưu trạng thái "đã gửi mã giảm giá":

```
[Trigger] Shopify — New Order
  → [Tables] Find Record: tìm email khách trong bảng "Đã gửi mã"
  → [Filter] Chỉ tiếp tục nếu KHÔNG tìm thấy (chưa gửi bao giờ)
  → [Action] Gmail: gửi mã giảm giá
  → [Tables] Create Record: lưu email vào bảng
```

### 21.3 Khi Nào Dùng Tables vs Google Sheets?

| | Tables | Google Sheets |
|---|---|---|
| Tốc độ | Nhanh hơn | Chậm hơn (qua API) |
| Xem/sửa tay | Được (giao diện đơn giản) | Tốt hơn (quen thuộc) |
| Chia sẻ ngoài | Khó | Dễ (share link) |
| Công thức | Không | Có |
| Phù hợp | Dữ liệu nội bộ của Zap | Báo cáo, chia sẻ với người khác |

---

## 22. Zapier Interfaces Chi Tiết

### 22.1 Tạo Form

1. Menu **Interfaces** → **+ Create** → **Form**
2. Kéo thả các trường: text, email, dropdown, file upload...
3. **Publish** → được link public: `interfaces.zapier.com/abc123`

### 22.2 Nối Form Với Zap

```
[Trigger] Interfaces — New Form Submission
  → [Action] Tạo contact HubSpot
  → [Action] Gửi email xác nhận
```

### 22.3 Ví Dụ: Form Đăng Ký Sự Kiện

1. Tạo form với các trường: Họ tên, Email, SĐT, Số người tham dự
2. Submit → Zap chạy:
   - Lưu vào Google Sheets "Danh sách sự kiện"
   - Gửi email vé điện tử (kèm mã QR nếu dùng Formatter)
   - Báo Slack #su-kien

---

## 23. Code by Zapier (Cho Người Biết Code)

Nếu không code đủ linh hoạt bằng Formatter, dùng **Code step** (Python hoặc JavaScript).

> 📌 Cần gói trả phí. Mỗi lần chạy tốn 1 task.

### 23.1 Ví Dụ Python

```python
# Input: input_data = {'ten': 'nguyen van a', 'tien': '1500000'}
ten = input_data['ten'].title()           # Nguyen Van A
tien = int(input_data['tien'])
phi_ship = 30000 if tien < 500000 else 0
tong = tien + phi_ship

# Output: return dict để step sau dùng
return {'ten_dep': ten, 'tong_tien': tong, 'phi_ship': phi_ship}
```

### 23.2 Ví Dụ JavaScript

```javascript
// Tính chiết khấu theo hạng khách
const hang = inputData.hang; // 'vang', 'bac', 'dong'
const tong = parseFloat(inputData.tong);

const ty_le = { 'vang': 0.15, 'bac': 0.10, 'dong': 0.05 };
const giam = tong * (ty_le[hang] || 0);

return { so_tien_giam: giam, thanh_toan: tong - giam };
```

### 23.3 Lưu Ý

- Không có internet trong Code step (không `fetch`/`requests` ra ngoài — dùng Webhooks thay)
- Timeout 10 giây, RAM giới hạn
- Chỉ dùng khi Formatter không làm được

---

## 24. Chuyển Từ Công Cụ Khác Sang Zapier

### 24.1 Từ Làm Tay Sang Zapier

1. Liệt kê công việc lặp đi lặp lại hàng tuần
2. Với mỗi việc, ghi: "khi X xảy ra, tôi làm A, B, C"
3. X → Trigger, A/B/C → Actions
4. Bắt đầu với việc đơn giản nhất trước

### 24.2 Từ IFTTT Sang Zapier

| IFTTT | Zapier tương đương |
|---|---|
| Applet | Zap |
| If This | Trigger |
| Then That | Action |
| Chỉ 1 action | Multi-step actions |

Zapier mạnh hơn IFTTT ở: multi-step, filter, formatter, nhiều app hơn.

---

*Cập nhật: 2026-10-10*

---

## 25. Zapier Central & AI Features

### 25.1 Zapier AI (Trợ Lý Tạo Zap Bằng Lời Nói)

Zapier có tính năng AI giúp tạo Zap bằng ngôn ngữ tự nhiên:

```
Bạn gõ: "Khi có đơn hàng Shopify mới, gửi tin nhắn Slack cho team và lưu vào Google Sheets"

AI tự tạo:
  → Trigger: Shopify - New Order
  → Action 1: Slack - Send Message
  → Action 2: Google Sheets - Create Row
```

**Cách dùng:**
1. Trang Home → ô **"What would you like to automate?"**
2. Gõ mô tả bằng tiếng Anh (AI hiểu tiếng Anh tốt nhất)
3. AI đề xuất Zap → bạn chỉnh sửa → Publish

### 25.2 AI by Zapier (Trong Zap)

Thêm step **AI by Zapier** để xử lý với AI:

| Chức năng | Ví dụ |
|---|---|
| Tóm tắt text | Email dài → tóm tắt 3 dòng |
| Phân loại | Phân loại ticket: "kỹ thuật" / "thanh toán" / "khác" |
| Trích xuất | Lấy tên, sđt từ đoạn chat |
| Dịch | Dịch email khách từ tiếng Anh sang Việt |
| Viết lại | Viết lại email cho chuyên nghiệp hơn |

**Ví dụ:** Tự động phân loại email hỗ trợ:

```
[Trigger] Gmail - New Email (label: hỗ trợ)
  → [AI] Phân loại: email này thuộc loại nào?
       (kỹ thuật / thanh toán / giao hàng / khác)
  → [Paths]
      ├─ Kỹ thuật → Assign cho team kỹ thuật
      ├─ Thanh toán → Assign cho kế toán
      └─ Khác → Assign cho CSKH chung
```

---

## 26. Quản Lý Chi Phí Nâng Cao

### 26.1 Theo Dõi Task Theo Zap

1. Vào **Settings** → **Usage**
2. Xem: Zap nào tốn nhiều task nhất
3. Zap tốn nhiều mà ít giá trị → tối ưu hoặc tắt

### 26.2 Task History

- **History** → lọc theo Zap → đếm số lần chạy
- Tính: `số lần chạy × số actions` = tasks tiêu thụ

### 26.3 Cảnh Báo Hết Task

1. Settings → **Notifications** → bật **Usage alerts**
2. Nhận email khi dùng 80%, 90%, 100% quota

---

## 27. Tình Huống Nâng Cao

### 27.1 Đồng Bộ 2 Chiều (Two-Way Sync)

**Vấn đề:** Task tạo trong Trello → muốn đồng bộ sang Asana và ngược lại.

**Giải pháp:** 2 Zaps:

```
Zap 1: Trello (New Card) → Asana (Create Task)
Zap 2: Asana (New Task) → Trello (Create Card)
```

⚠️ **Cẩn thận vòng lặp vô hạn!** Thêm Filter:
- Zap 1: chỉ chạy nếu card Trello chưa có trong Asana
- Dùng custom field đánh dấu "đã đồng bộ"

### 27.2 Xử Lý File Đính Kèm

```
[Trigger] Gmail - New Email có file đính kèm
  → [Filter] Tên file chứa ".pdf" hoặc ".xlsx"
  → [Action] Google Drive - Upload File
  → [Action] Google Sheets - Ghi log (tên file, ngày, link Drive)
  → [Action] Slack - Báo "có file mới cần xử lý"
```

### 27.3 Hệ Thống Điểm Danh (Chấm Công)

```
[Trigger] Schedule - Mỗi ngày 8h sáng
  → [Action] Google Sheets - Tạo dòng mới (ngày hôm nay)
  
Nhân viên điền form điểm danh:
[Trigger] Form - New Submission
  → [Action] Sheets - Tìm dòng hôm nay → cập nhật giờ vào
  
[Trigger] Schedule - Mỗi ngày 18h
  → [Action] Sheets - Lấy những ai chưa điểm danh
  → [Action] Slack - Nhắc những người chưa điểm danh
```

---

## 28. FAQ Mở Rộng

**Zapier có làm việc với app Việt Nam không?**
Các app Việt (Haravan, Sapo, KiotViet...) chưa có tích hợp chính thức. Dùng **Webhooks** nếu app đó có API, hoặc qua trung gian (email → Zapier).

**1 tài khoản dùng cho nhiều công ty được không?**
Được, nhưng nên tách workspace/folder riêng. Gói Team hỗ trợ tốt hơn.

**Zap có chạy khi tôi tắt máy không?**
Có! Zap chạy trên server Zapier, không phụ thuộc máy bạn.

**Dữ liệu của tôi có an toàn không?**
Zapier đạt SOC 2 Type II, GDPR compliant. Nhưng đừng truyền thông tin siêu nhạy cảm (mật khẩu ngân hàng...) qua Zapier.

**Có thể tạm dừng 1 step mà không tắt cả Zap không?**
Không trực tiếp. Workaround: thêm Filter luôn-false trước step đó, hoặc duplicate Zap để test.

**Zapier có API để tự tạo Zap bằng code không?**
Có: Zapier Developer Platform (dành cho nhà phát triển app muốn đưa app lên Zapier).

---

*Cập nhật: 2026-10-10*
