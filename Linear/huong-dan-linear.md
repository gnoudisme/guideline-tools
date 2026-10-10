# Hướng Dẫn Toàn Tập Linear — Quản Lý Công Việc Cho Team Hiện Đại

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10

---

## Mục Lục

1. [Linear Là Gì?](#1-linear-là-gì)
2. [Tạo Tài Khoản & Workspace](#2-tạo-tài-khoản--workspace)
3. [Tổng Quan Giao Diện](#3-tổng-quan-giao-diện)
4. [Issues — Đơn Vị Công Việc](#4-issues--đơn-vị-công-việc)
5. [Triage — Sàng Lọc Việc Mới](#5-triage--sàng-lọc-việc-mới)
6. [Projects — Quản Lý Dự Án](#6-projects--quản-lý-dự-án)
7. [Cycles — Làm Việc Theo Sprint](#7-cycles--làm-việc-theo-sprint)
8. [Views — Chế Độ Xem](#8-views--chế-độ-xem)
9. [Labels, Priority, Estimate](#9-labels-priority-estimate)
10. [Phím Tắt (Keyboard Shortcuts)](#10-phím-tắt)
11. [Lệnh Nhanh (Command Menu)](#11-lệnh-nhanh)
12. [Tích Hợp GitHub](#12-tích-hợp-github)
13. [Tích Hợp Slack](#13-tích-hợp-slack)
14. [Tích Hợp Khác](#14-tích-hợp-khác)
15. [Linear API](#15-linear-api)
16. [Notifications — Thông Báo](#16-notifications--thông-báo)
17. [Bảng Giá](#17-bảng-giá)
18. [Linear vs Đối Thủ](#18-linear-vs-đối-thủ)
19. [Xử Lý Lỗi Thường Gặp](#19-xử-lý-lỗi-thường-gặp)
20. [Best Practices](#20-best-practices)
21. [Cheat Sheet](#21-cheat-sheet)

---

## 1. Linear Là Gì?

### 1.1 Định Nghĩa

**Linear** là công cụ quản lý issues/tasks và theo dõi dự án, được thiết kế riêng cho **team phát triển phần mềm**. Nó như Jira nhưng **nhanh hơn, đẹp hơn, đơn giản hơn** rất nhiều.

### 1.2 Triết Lý Của Linear

1. **Tốc độ là tính năng:** mọi thao tác đều có phím tắt, không cần chuột
2. **Đơn giản nhưng mạnh:** ít nút bấm hơn Jira nhưng đủ dùng cho team lớn
3. **Tích hợp sâu với dev workflow:** GitHub, GitLab, Slack là công dân hạng nhất
4. **Opinionated:** Linear "ép" bạn làm theo cách tốt nhất (cycles, triage...) thay vì để bạn tự mò

### 1.3 Ai Nên Dùng Linear?

- **Team dev** (2-50 người): quản lý bugs, features, tasks
- **Startup công nghệ:** cần quy trình gọn nhẹ, không rườm rà như Jira
- **Freelancer dev:** quản lý việc cá nhân chuyên nghiệp
- **Product manager:** theo dõi roadmap, ưu tiên tính năng

### 1.4 Linear Không Phù Hợp Khi Nào?

- Team non-tech hoàn toàn (marketing, sale thuần) → dùng Asana/Trello dễ hơn
- Cần quản lý tài liệu phức tạp → Linear chỉ quản lý issues, không phải wiki
- Cần Gantt chart chi tiết → Linear không có (dùng Projects timeline đơn giản)

---

## 2. Tạo Tài Khoản & Workspace

### 2.1 Đăng Ký

1. Vào **https://linear.app**
2. Bấm **Sign up** → đăng ký bằng **Google** hoặc **email**
3. Nhập tên workspace: ví dụ `cong-ty-abc` → URL sẽ là `linear.app/cong-ty-abc`

### 2.2 Workspace Là Gì?

**Workspace** = không gian làm việc của cả công ty/team. Mỗi workspace có:
- Nhiều **teams** (team Frontend, Backend, Design...)
- Mỗi team có **issues** riêng, **cycles** riêng

### 2.3 Tạo Team

1. Settings (góc trái dưới) → **Teams** → **New team**
2. Điền:
   - **Name:** `Engineering` / `Thiết kế` / `Marketing`
   - **Key:** `ENG` / `DES` (3 ký tự viết tắt — dùng trong ID issue: `ENG-123`)
   - **Icon & màu:** chọn cho dễ phân biệt
3. **Create team**

> 💡 **Gợi ý:** Bắt đầu với 1 team duy nhất. Khi nào > 10 người mới tách team.

### 2.4 Mời Thành Viên

1. Settings → **Members** → **Invite**
2. Nhập email → chọn role:

| Role | Quyền |
|---|---|
| **Admin** | Mọi thứ (billing, xóa workspace) |
| **Member** | Tạo/sửa issues, projects |
| **Guest** | Chỉ xem issues được share (cần gói trả phí) |
| **Viewer** | Chỉ xem (cần gói trả phí) |

3. Người được mời nhận email → bấm link → vào workspace

---

## 3. Tổng Quan Giao Diện

### 3.1 Sidebar Trái

| Mục | Chức năng |
|---|---|
| **Inbox** | Thông báo: ai mention, assign việc cho bạn |
| **My Issues** | Tất cả việc được giao cho bạn |
| **Teams** | Danh sách teams → bấm vào xem issues |
| **Projects** | Danh sách dự án |
| **Cycles** | Sprint hiện tại và sắp tới |
| **Views** | Các view đã lưu |

### 3.2 Thanh Tìm Kiếm

- Bấm **Cmd/Ctrl + K** → tìm issue, project, người, hoặc chạy lệnh
- Gõ `#123` → nhảy đến issue số 123
- Gõ tên người → xem profile

### 3.3 Tốc Độ

Linear nổi tiếng **nhanh như chớp**: chuyển trang gần như tức thì, không loading spinner. Đây là lý do nhiều team bỏ Jira sang Linear.

---

## 4. Issues — Đơn Vị Công Việc

### 4.1 Issue Là Gì?

**Issue** = 1 đơn vị công việc: bug, feature, task, cải tiến... Mỗi issue có ID duy nhất: `ENG-123`, `DES-45`.

### 4.2 Tạo Issue

**Cách 1 — Phím tắt (nhanh nhất):**
- Bấm **C** ở bất kỳ đâu → popup tạo issue hiện ra
- Gõ tiêu đề → Enter → xong!

**Cách 2 — Nút:**
- Bấm **New issue** (góc phải trên)

**Cách 3 — Từ Slack/GitHub:** (xem phần tích hợp)

### 4.3 Các Trường Của Issue

| Trường | Ý nghĩa | Ví dụ |
|---|---|---|
| **Title** | Tên việc (ngắn gọn, rõ ràng) | `Sửa lỗi đăng nhập bằng Google` |
| **Description** | Chi tiết (hỗ trợ Markdown) | Các bước tái hiện lỗi, ảnh chụp màn hình |
| **Team** | Thuộc team nào | Engineering |
| **Status** | Trạng thái | Backlog → Todo → In Progress → Done |
| **Assignee** | Ai làm | Nguyễn Văn A |
| **Priority** | Độ ưu tiên | Urgent / High / Medium / Low / No priority |
| **Labels** | Nhãn phân loại | `bug`, `frontend`, `api` |
| **Estimate** | Điểm ước lượng | 1, 2, 3, 5, 8 (story points) |
| **Cycle** | Sprint nào | Cycle 12 |
| **Project** | Dự án nào | `App Mobile v2` |
| **Due date** | Hạn chót | 15/10/2026 |
| **Parent/Sub-issues** | Việc cha/con | Chia task lớn thành nhỏ |

### 4.4 Viết Tiêu Đề Issue Tốt

```
❌ Tệ:   "lỗi"
❌ Tệ:   "fix bug"
✅ Tốt:  "Sửa lỗi không đăng nhập được bằng Google trên iOS 17"
✅ Tốt:  "[API] Thêm endpoint lấy danh sách đơn hàng theo ngày"
```

**Quy ước gợi ý:**
- Bug: `Sửa lỗi [mô tả ngắn]`
- Feature: `[Module] Thêm [tính năng]`
- Bắt đầu bằng động từ: Sửa, Thêm, Cập nhật, Xóa, Tối ưu...

### 4.5 Description Với Markdown

```markdown
## Mô tả
Nút đăng nhập bằng Google không phản hồi trên iOS 17.

## Các bước tái hiện
1. Mở app trên iPhone iOS 17
2. Bấm "Đăng nhập bằng Google"
3. Không có gì xảy ra

## Mong đợi
Mở trình duyệt để xác thực Google

## Ảnh chụp
(dán ảnh trực tiếp vào đây)

## Môi trường
- iOS 17.0, iPhone 14
- App version 2.3.1
```

### 4.6 Sub-Issues (Chia Nhỏ)

Task lớn → chia thành sub-issues:

```
ENG-100: Xây dựng trang thanh toán [Parent]
  ├── ENG-101: Thiết kế UI trang thanh toán
  ├── ENG-102: Tích hợp API thanh toán VNPay
  ├── ENG-103: Xử lý callback sau thanh toán
  └── ENG-104: Viết test cho luồng thanh toán
```

- Trong issue cha → **Add sub-issues** → tạo hoặc link issue có sẵn
- Thanh tiến trình hiện: `2/4 sub-issues done`

### 4.7 Quan Hệ Giữa Các Issues

| Quan hệ | Ý nghĩa |
|---|---|
| **Blocked by** | Việc này chờ việc kia xong mới làm được |
| **Blocking** | Việc này đang chặn việc khác |
| **Related** | Liên quan (không chặn) |
| **Duplicate** | Trùng lặp — đóng cái này, giữ cái kia |

---

## 5. Triage — Sàng Lọc Việc Mới

### 5.1 Triage Là Gì?

**Triage** = hàng đợi việc mới chưa được phân loại. Khi ai đó tạo issue mà không gán team/status rõ ràng, nó rơi vào Triage.

### 5.2 Quy Trình Triage (Nên Làm Mỗi Sáng)

1. Vào **Triage** (trong team)
2. Với mỗi issue:
   - Đọc tiêu đề + mô tả
   - Gán **priority** (khẩn không?)
   - Gán **assignee** (ai làm?)
   - Chuyển status → **Todo** (hoặc Backlog nếu chưa cần)
   - Thêm vào **Cycle** hiện tại (nếu làm ngay)
3. Triage trống = team đã sàng lọc xong

> 💡 **Mẹo:** Chỉ định 1 người "trực triage" mỗi tuần để sàng lọc.

---

## 6. Projects — Quản Lý Dự Án

### 6.1 Project Là Gì?

**Project** = nhóm issues phục vụ 1 mục tiêu lớn, có **thời hạn**.

Ví dụ:
- `App Mobile v2.0` (deadline: 31/12/2026)
- `Tích hợp thanh toán VNPay`
- `Redesign trang chủ`

### 6.2 Tạo Project

1. Sidebar → **Projects** → **New project**
2. Điền:
   - **Name:** `App Mobile v2.0`
   - **Team:** Engineering
   - **Target date:** 31/12/2026
   - **Description:** mục tiêu, phạm vi
   - **Lead:** người chịu trách nhiệm
3. **Create**

### 6.3 Thêm Issues Vào Project

- Mở issue → trường **Projects** → chọn project
- Hoặc trong project → **Add issues** → chọn nhiều issue

### 6.4 Theo Dõi Tiến Độ

Project có:
- **Overview:** mô tả, timeline, tiến trình tổng
- **Issues:** danh sách issues (lọc theo status)
- **Progress bar:** `15/40 issues done (38%)`
- **Burndown:** biểu đồ tiến độ theo thời gian

### 6.5 Project Updates

- Viết **update** hàng tuần: đã làm gì, còn gì, có blocker không
- Team và sếp theo dõi mà không cần họp

---

## 7. Cycles — Làm Việc Theo Sprint

### 7.1 Cycle Là Gì?

**Cycle** = sprint theo phương pháp Agile: 1 khoảng thời gian cố định (thường 1-2 tuần) để hoàn thành 1 nhóm issues.

### 7.2 Setup Cycle

1. Team settings → **Cycles** → bật
2. Cài đặt:
   - **Duration:** 2 tuần (phổ biến nhất)
   - **Start day:** Thứ 2
   - **Auto-add:** tự động thêm issues quá hạn từ cycle trước

### 7.3 Quy Trình Cycle

```
Đầu cycle (planning):
  1. Xem backlog → chọn issues cho cycle này
  2. Kéo vào cycle hiện tại
  3. Ước lượng effort (estimate points)

Trong cycle:
  - Mỗi người tự lấy việc, chuyển status
  - Daily: xem board cycle

Cuối cycle:
  - Issues chưa xong → tự chuyển sang cycle tiếp (nếu bật auto)
  - Xem velocity: team làm được bao nhiêu points
```

### 7.4 Cycle vs Project

| | Cycle | Project |
|---|---|---|
| Là gì | Khoảng thời gian (2 tuần) | Mục tiêu lớn (có deadline) |
| Issues | Việc làm trong 2 tuần này | Tất cả việc của dự án |
| Lặp lại | Có (cycle 1, 2, 3...) | Không (1 lần) |
| Ví dụ | "Sprint 12: 14-28/10" | "App Mobile v2.0" |

Một issue có thể vừa thuộc **Project** (mục tiêu dài hạn) vừa thuộc **Cycle** (làm trong sprint này).

---

## 8. Views — Chế Độ Xem

### 8.1 Các Kiểu Hiển Thị

| Kiểu | Phù hợp |
|---|---|
| **List** | Xem nhanh danh sách, sắp xếp/lọc |
| **Board** | Kanban theo status (kéo thả) |

### 8.2 Lọc (Filters)

Bấm **Filter** → thêm điều kiện:

```
Status is In Progress
AND Assignee is me
AND Priority is High or Urgent
AND Label is bug
```

### 8.3 Lưu View

Sau khi lọc → **Save view** → đặt tên `Bug khẩn của tôi`
- Lần sau bấm 1 cái là ra
- Chia sẻ view cho team

### 8.4 Sắp Xếp & Nhóm

- **Sort:** theo priority, created date, updated...
- **Group by:** status, assignee, label, priority, cycle...

---

## 9. Labels, Priority, Estimate

### 9.1 Labels (Nhãn)

Tạo nhãn để phân loại:

| Label | Màu gợi ý | Dùng cho |
|---|---|---|
| `bug` | 🔴 đỏ | Lỗi cần sửa |
| `feature` | 🟢 xanh lá | Tính năng mới |
| `frontend` | 🔵 xanh dương | Liên quan giao diện |
| `backend` | 🟣 tím | Liên quan server/API |
| `urgent-fix` | 🟠 cam | Sửa gấp |
| `good-first-issue` | ⚪ xám | Việc dễ cho người mới |

Team settings → **Labels** → tạo/sửa/xóa.

### 9.2 Priority (Độ Ưu Tiên)

| Mức | Icon | Khi nào dùng |
|---|---|---|
| **Urgent** | 🔴 | Sập production, mất tiền — làm NGAY |
| **High** | 🟠 | Quan trọng, làm trong cycle này |
| **Medium** | 🟡 | Bình thường (mặc định) |
| **Low** | 🔵 | Khi nào rảnh làm |
| **No priority** | ⚪ | Chưa đánh giá |

### 9.3 Estimate (Ước Lượng)

Dùng **story points** (Fibonacci): `1, 2, 3, 5, 8`

| Points | Ý nghĩa |
|---|---|
| 1 | Việc nhỏ, < 2 giờ |
| 2 | Việc vừa, nửa ngày |
| 3 | Việc lớn, 1-2 ngày |
| 5 | Việc rất lớn, ~1 tuần |
| 8 | Quá lớn — nên chia nhỏ! |

> 💡 Không cần chính xác tuyệt đối. Mục đích là để tính **velocity** (team làm được bao nhiêu points/cycle).

---

## 10. Phím Tắt (Quan Trọng Nhất!)

Linear được thiết kế để **không cần chuột**. Học phím tắt = nhanh gấp 3 lần.

### 10.1 Tạo & Tìm Kiếm

| Phím | Chức năng |
|---|---|
| **C** | Tạo issue mới |
| **Cmd/Ctrl + K** | Tìm kiếm / lệnh nhanh |
| **/** | Tìm kiếm trong trang |

### 10.2 Điều Hướng

| Phím | Chức năng |
|---|---|
| **J** / **K** | Xuống / lên (chọn issue) |
| **Enter** | Mở issue đang chọn |
| **Esc** | Đóng / quay lại |
| **G then I** | Về Inbox |
| **G then M** | Về My Issues |
| **G then P** | Về Projects |
| **G then C** | Về Cycles |

### 10.3 Thao Tác Issue Đang Mở

| Phím | Chức năng |
|---|---|
| **A** | Gán người (assignee) |
| **S** | Đổi status |
| **P** | Đổi priority |
| **L** | Thêm label |
| **E** | Đổi estimate |
| **D** | Đặt due date |
| **O** | Mở/tạo sub-issue |
| **Cmd/Ctrl + Enter** | Lưu comment |

### 10.4 Xem Tất Cả

- Bấm **?** (dấu hỏi) ở bất kỳ đâu → bảng phím tắt đầy đủ
- Hoặc **Cmd/Ctrl + K** → gõ `shortcuts`

> 🎯 **Thử thách:** Dùng Linear 1 tuần chỉ bằng phím tắt, không chạm chuột. Bạn sẽ không muốn quay lại!

---

## 11. Lệnh Nhanh (Command Menu)

Bấm **Cmd/Ctrl + K** → gõ lệnh:

| Gõ | Làm gì |
|---|---|
| `new issue` | Tạo issue |
| `my issues` | Xem việc của tôi |
| `triage` | Vào triage |
| `create project` | Tạo project |
| `go to settings` | Mở cài đặt |
| `change theme` | Đổi giao diện sáng/tối |
| Tên người | Tìm người |
| `ENG-123` | Nhảy đến issue |

---

## 12. Tích Hợp GitHub

Tích hợp quan trọng nhất cho team dev!

### 12.1 Kết Nối

1. Settings → **Integrations** → **GitHub** → **Connect**
2. Đăng nhập GitHub → chọn organization/repo → **Install**

### 12.2 Tính Năng

**Từ Linear → GitHub:**
- Tạo issue Linear → tự tạo branch `eng-123-ten-issue` khi bấm "Create branch"
- Issue hiển thị PRs liên quan

**Từ GitHub → Linear (bằng cú pháp trong commit/PR):**

| Cú pháp trong commit message | Kết quả |
|---|---|
| `Fixes ENG-123` | Đóng issue khi PR merge |
| `Closes ENG-123` | Đóng issue khi PR merge |
| `Refs ENG-123` | Link PR vào issue (không đóng) |
| `ENG-123` trong tên branch | Tự link branch với issue |

**Ví dụ commit:**
```
git commit -m "Sửa lỗi đăng nhập Google

Fixes ENG-123"
```

→ Khi PR merge → issue ENG-123 tự chuyển sang **Done**! 🎉

### 12.3 GitHub PR Status Trong Linear

- Issue hiển thị: PR đang open, đã review, đã merge
- Reviewer, CI status (pass/fail)

---

## 13. Tích Hợp Slack

### 13.1 Kết Nối

1. Settings → **Integrations** → **Slack** → **Add to Slack**
2. Chọn workspace → Allow

### 13.2 Tạo Issue Từ Slack

Trong Slack, gõ:
```
/linear new Sửa lỗi trang thanh toán
```

→ Tạo issue ngay trong Slack!

Hoặc: **chuột phải vào tin nhắn** → **Create Linear issue** — nội dung tin nhắn thành mô tả issue.

### 13.3 Thông Báo

Chọn events muốn báo vào Slack:
- Issue mới được tạo
- Issue được assign cho bạn
- Issue chuyển sang Done
- Có comment mới mention bạn

### 13.4 Slack Notifications Cá Nhân

- Kết nối tài khoản Slack cá nhân (không chỉ workspace chung)
- Nhận DM khi có việc liên quan đến bạn

---

## 14. Tích Hợp Khác

| Tích hợp | Làm gì |
|---|---|
| **GitLab** | Như GitHub nhưng cho GitLab |
| **Sentry** | Bug từ Sentry tự tạo Linear issue |
| **Figma** | Link design vào issue, xem preview |
| **Zendesk / Intercom** | Ticket hỗ trợ → tạo issue dev |
| **Zapier** | Kết nối với 7.000+ app khác |
| **Google Calendar** | Hiện due date issues trên lịch |

---

## 15. Linear API

### 15.1 Tạo API Key

1. Settings → **API** → **Create key**
2. Đặt tên → copy key (chỉ hiện 1 lần!)

### 15.2 GraphQL API

Linear dùng **GraphQL** (không phải REST):

```bash
# Lấy danh sách issues
curl -X POST https://api.linear.app/graphql \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "{ issues(first: 10) { nodes { id title state { name } } } }"
  }'
```

### 15.3 Tạo Issue Qua API

```bash
curl -X POST https://api.linear.app/graphql \
  -H "Authorization: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "mutation { issueCreate(input: { teamId: \"TEAM_ID\", title: \"Bug từ hệ thống giám sát\" }) { issue { id identifier } } }"
  }'
```

### 15.4 Webhooks

1. Settings → **API** → **Webhooks** → **New webhook**
2. Nhập URL → chọn events (issue created, updated...)
3. Hệ thống của bạn nhận POST khi có sự kiện

### 15.5 Lấy Team ID

```graphql
{ teams { nodes { id name key } } }
```

---

## 16. Notifications — Thông Báo

### 16.1 Inbox

- **Inbox** (sidebar) = trung tâm thông báo
- Hiện: được assign, bị mention, issue thay đổi status...
- Bấm **Mark all as read** để dọn

### 16.2 Cài Đặt Thông Báo

Settings → **Notifications** → chọn kênh:

| Kênh | Khi nào |
|---|---|
| **Linear inbox** | Luôn có |
| **Email** | Tổng hợp hàng ngày (digest) |
| **Slack DM** | Realtime cho việc quan trọng |
| **Mobile push** | App Linear trên điện thoại |

### 16.3 Đừng Để Bị Spam

- Tắt thông báo cho team bạn không liên quan
- Chỉ bật cho: được assign, bị mention, issue mình tạo
- Dùng **Snooze**: ẩn thông báo đến mai

---

## 17. Bảng Giá

> 💲 Giá USD, kiểm tra tại linear.app/pricing

| Gói | Giá | Nổi bật |
|---|---|---|
| **Free** | $0 | 250 issues, 10MB file, 1 team? (giới hạn nhẹ) |
| **Standard** | $8/user/tháng | Issues không giới hạn, cycles, guest accounts |
| **Plus** | $14/user/tháng | + private teams, SSO Google, SLA |
| **Enterprise** | Liên hệ | SSO SAML, audit log, hỗ trợ riêng |

### 17.1 Free Có Đủ Dùng Không?

**Đủ cho:** team nhỏ < 10 người, < 250 issues đang mở.

**Không đủ khi:** cần cycles, projects nhiều, guest accounts cho khách hàng.

> 💡 Linear free **rộng rãi hơn Jira free** nhiều. Hầu hết team nhỏ dùng free là đủ.

---

## 18. Linear vs Đối Thủ

| Tiêu chí | **Linear** | Jira | Asana | Trello | ClickUp |
|---|---|---|---|---|---|
| Tốc độ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Dễ dùng | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Cho team dev | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| Phím tắt | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| Tích hợp Git | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐ | ⭐⭐⭐ |
| Giá (10 users) | $80/tháng | ~$80 | ~$110 | ~$100 | ~$70 |
| Phù hợp | Team dev hiện đại | Enterprise lớn | Đa năng | Đơn giản | All-in-one |

**Kết luận:**
- **Team dev, thích tốc độ** → Linear
- **Công ty lớn, quy trình phức tạp** → Jira
- **Team đa năng (cả non-tech)** → Asana
- **Cần đơn giản nhất** → Trello

---

## 19. Xử Lý Lỗi Thường Gặp

| Vấn đề | Cách khắc phục |
|---|---|
| Không tạo được issue | Kiểm tra đã chọn team chưa |
| GitHub không tự đóng issue | Kiểm tra cú pháp `Fixes ENG-123` (đúng ID, đúng từ khóa) |
| Không nhận thông báo Slack | Kiểm tra integration Slack còn kết nối, kiểm tra notification settings |
| Cycle không tự chuyển issue | Bật "Auto-add incomplete issues" trong team cycle settings |
| Mất issue | Kiểm tra archive (issues đóng lâu tự archive), dùng search |
| Không mời được guest | Cần gói Standard trở lên |
| API 401 | API key sai hoặc đã bị xóa → tạo key mới |
| Tích hợp GitHub mất kết nối | Settings → Integrations → GitHub → Reconnect |

---

## 20. Best Practices

### 20.1 Quy Ước Team

1. **Tiêu đề rõ ràng:** động từ + mô tả cụ thể
2. **Mọi bug đều có:** các bước tái hiện + môi trường + ảnh
3. **Estimate trước khi vào cycle:** không estimate = không vào sprint
4. **1 issue = 1 việc:** không gộp "sửa 5 lỗi linh tinh" thành 1 issue
5. **Đóng issue khi xong thật:** đã merge + đã test, không phải "code xong"

### 20.2 Quy Trình Đề Xuất (Cho Team 3-10 Người)

```
Thứ 2 sáng (30 phút):
  → Planning: chọn issues cho cycle 2 tuần
  → Gán người, estimate

Hàng ngày:
  → Mỗi người tự cập nhật status
  → Blocker → báo ngay trong Slack, link issue

Thứ 6 (15 phút):
  → Review: demo những gì xong
  → Retro nhanh: tuần này tốt/chưa tốt gì

Liên tục:
  → Bug mới → vào Triage → người trực phân loại mỗi sáng
```

### 20.3 Vệ Sinh Định Kỳ

- **Cuối mỗi cycle:** đóng issues đã xong nhưng quên chuyển Done
- **Hàng tháng:** archive issues cũ, xóa labels không dùng
- **Hàng quý:** review lại quy ước team có còn phù hợp không

---

## 21. Cheat Sheet

### Phím tắt cốt lõi
```
C              → tạo issue mới
Cmd/Ctrl + K   → tìm kiếm / lệnh
J / K          → chọn issue dưới / trên
A / S / P / L  → assignee / status / priority / label
?              → xem tất cả phím tắt
```

### Đóng issue từ GitHub
```
Fixes ENG-123    → đóng khi PR merge
Closes ENG-123   → đóng khi PR merge
```

### Quy trình issue
```
Triage → Todo → In Progress → In Review → Done
              (Backlog nếu chưa cần làm)
```

### Ưu tiên
```
🔴 Urgent → 🟠 High → 🟡 Medium → 🔵 Low → ⚪ No priority
```

### Story points
```
1 (< 2h) → 2 (nửa ngày) → 3 (1-2 ngày) → 5 (~1 tuần) → 8 (chia nhỏ!)
```

---

*Cập nhật: 2026-10-10*

---

## 22. Initiatives — Quản Lý Mục Tiêu Lớn (Roadmap)

### 22.1 Initiative Là Gì?

**Initiative** = mục tiêu chiến lược lớn, bao gồm **nhiều projects**.

```
Initiative: "Ra mắt App Mobile" (Q4 2026)
  ├── Project: App iOS v1.0
  ├── Project: App Android v1.0
  └── Project: Backend API cho mobile
```

> 📌 Cần gói Standard trở lên.

### 22.2 Tạo Initiative

1. Sidebar → **Initiatives** → **New initiative**
2. Điền: tên, mô tả, target date, owner
3. Thêm projects liên quan
4. Theo dõi tiến độ tổng: `2/3 projects xong`

### 22.3 Khi Nào Dùng Initiative vs Project?

| | Initiative | Project |
|---|---|---|
| Quy mô | Rất lớn (nhiều team, nhiều tháng) | Vừa (1 team, vài tuần-tháng) |
| Ví dụ | "Mở rộng sang thị trường Thái Lan" | "Dịch app sang tiếng Thái" |
| Ai quản lý | C-level, Head | Team lead |

---

## 23. Documents — Viết Tài Liệu Trong Linear

Linear có trình soạn thảo tài liệu tích hợp (như Notion đơn giản).

### 23.1 Tạo Document

1. Trong project → tab **Documents** → **New document**
2. Viết bằng Markdown/WYSIWYG

### 23.2 Dùng Để Làm Gì?

- **Spec kỹ thuật:** mô tả chi tiết tính năng trước khi code
- **Quyết định kiến trúc (ADR):** tại sao chọn giải pháp A thay vì B
- **Hướng dẫn onboarding:** người mới đọc để hiểu dự án

### 23.3 Link Issue Trong Document

- Gõ `#` + số issue → tạo link: `ENG-123`
- Issue tự hiển thị document liên quan

---

## 24. Insights — Phân Tích Dữ Liệu Team

### 24.1 Xem Insights

Team → **Insights** → các biểu đồ:

| Biểu đồ | Cho biết |
|---|---|
| **Velocity** | Team làm được bao nhiêu points/cycle |
| **Cycle time** | Trung bình 1 issue mất bao lâu từ Todo → Done |
| **Burndown** | Tiến độ cycle hiện tại |
| **Workload** | Ai đang quá tải, ai đang rảnh |

### 24.2 Dùng Insights Để Cải Tiến

- **Velocity giảm 3 cycle liên tiếp** → team đang gặp vấn đề (họp retro tìm nguyên nhân)
- **Cycle time tăng** → issues quá lớn, cần chia nhỏ hơn
- **1 người workload gấp đôi người khác** → phân việc lại

---

## 25. Tự Động Hóa Trong Linear

### 25.1 Auto-Close

- PR merge với `Fixes ENG-123` → issue tự Done (đã nói ở phần 12)

### 25.2 Auto-Archive

- Settings → issues **Done quá 6 tháng** → tự archive (cho gọn)

### 25.3 Template Issues

Tạo mẫu cho việc lặp đi lặp lại:

1. Tạo issue mẫu: `Template: Bug report`
2. Điền sẵn description với các mục cần điền
3. Khi cần: **Duplicate** issue mẫu → sửa nội dung

**Ví dụ template bug:**
```markdown
## Môi trường
- OS:
- Browser/App version:

## Các bước tái hiện
1.
2.
3.

## Kết quả thực tế


## Kết quả mong đợi


## Ảnh/video

```

### 25.4 Tự Động Gán (Auto-assign)

- Dùng **Triage responsibility**: mỗi tuần 1 người trực triage
- Hoặc tích hợp với Round-robin qua API

---

## 26. Linear Mobile App

### 26.1 Cài Đặt & Dùng

- Tải **Linear** trên App Store / Google Play
- Đầy đủ: xem issues, comment, chuyển status, tạo issue nhanh

### 26.2 Khi Nào Dùng Mobile

- Sếp hỏi "việc X xong chưa?" → mở app kiểm tra ngay
- Đang đi đường nảy ra ý tưởng → tạo issue nhanh
- Nhận push khi được mention

---

## 27. Import Từ Công Cụ Khác

### 27.1 Từ Jira

1. Settings → **Import** → **Jira**
2. Đăng nhập Jira → chọn projects → **Import**
3. Issues, comments, attachments được chuyển sang (status map tự động)

### 27.2 Từ Trello / Asana / GitHub Issues

- Tương tự: Settings → Import → chọn nguồn
- Kiểm tra lại labels và assignees sau import (có thể lệch)

### 27.3 Từ CSV/Excel

1. Chuẩn bị file CSV: `Title,Description,Priority,Assignee`
2. Settings → Import → **CSV**
3. Map cột → Import

---

## 28. Quyền Riêng Tư & Bảo Mật

### 28.1 Private Teams

- Tạo team **private**: chỉ thành viên team mới thấy issues
- Dùng cho: lương thưởng, kế hoạch sa thải, ý tưởng bí mật

> 📌 Cần gói Plus trở lên.

### 28.2 2FA

- Settings cá nhân → **Security** → bật **Two-factor authentication**
- Nên bắt buộc cho cả workspace (Admin settings)

### 28.3 Audit Log

- Xem ai đã làm gì: tạo/xóa issue, đổi settings...
- Cần gói Enterprise

---

## 29. Tình Huống Thực Tế & Cách Xử Lý

### 29.1 "Bug production đang cháy!"

```
1. Tạo issue: bấm C → "SỰ CỐ: thanh toán lỗi 500" → Priority: Urgent
2. Assign cho người trực (hoặc @channel trong Slack)
3. Tạo sub-issues:
   - Tìm nguyên nhân (assign: A)
   - Fix tạm thời / rollback (assign: B)
   - Viết post-mortem sau (assign: team lead)
4. Khi xong: chuyển Done → viết comment tóm tắt nguyên nhân + cách fix
```

### 29.2 "Sếp hỏi tiến độ dự án"

```
1. Mở Project → xem progress bar + burndown
2. Chụp màn hình hoặc share link project
3. Nếu sếp không dùng Linear: export issues ra CSV gửi
```

### 29.3 "Team 20 người, việc lộn xộn"

```
1. Tách thành teams nhỏ (Frontend, Backend, Mobile...)
2. Mỗi team có triage riêng + người trực
3. Dùng labels chung: bug, feature + labels riêng từng team
4. Họp planning 2 tuần/lần cho từng team
5. Dùng Initiative để sếp nhìn tổng thể
```

### 29.4 "Dev quên cập nhật status"

```
1. Nhắc nhở nhẹ trong retro (không phạt)
2. Tích hợp GitHub: commit/PR tự cập nhật
3. Đặt quy ước: "trước khi nghỉ, cập nhật status 30 giây"
4. Dùng Slack reminder hàng ngày (tích hợp)
```

---

## 30. Tài Nguyên Học Thêm

| Tài nguyên | Link |
|---|---|
| Tài liệu chính thức | linear.app/docs |
| Phím tắt đầy đủ | Bấm `?` trong Linear |
| Changelog (tính năng mới) | linear.app/changelog |
| Blog (cách Linear làm sản phẩm) | linear.app/blog |
| Cộng đồng | Slack community của Linear |

---

*Cập nhật: 2026-10-10*

---

## 31. Workflow Mẫu Cho Từng Loại Team

### 31.1 Team Startup 3 Người

```
Teams: 1 team duy nhất "Product"
Cycles: 1 tuần (ngắn, linh hoạt)
Labels: bug, feature, chore
Quy trình:
  - Sáng thứ 2: planning 30 phút, chọn việc cho tuần
  - Mỗi ngày: cập nhật status trước khi nghỉ
  - Chiều thứ 6: demo + retro 30 phút
```

### 31.2 Team Agency (Nhiều Khách Hàng)

```
Teams: 1 team "Delivery"
Projects: mỗi khách hàng = 1 project
  - "Website Công ty A"
  - "App Mobile Công ty B"
Labels: thêm label tên khách để lọc
Views: lưu view "Việc của khách A", "Việc của khách B"
```

### 31.3 Team Product Lớn (20+ Người)

```
Teams: Frontend, Backend, Mobile, Design, QA
Initiatives: mục tiêu quý
Projects: tính năng lớn xuyên team
Cycles: 2 tuần, đồng bộ ngày bắt đầu
Triage: mỗi team có người trực riêng
```

---

## 32. Tích Hợp Linear Với CI/CD

### 32.1 GitHub Actions Tự Động

```yaml
# .github/workflows/linear-notify.yml
name: Notify Linear on Deploy
on:
  push:
    branches: [main]

jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - name: Comment on Linear issues
        run: |
          # Lấy issue IDs từ commit messages và comment "Đã deploy"
          for issue in $(git log -1 --pretty=%B | grep -oE '[A-Z]+-[0-9]+'); do
            echo "Deployed fix for $issue"
          done
```

### 32.2 Tự Động Tạo Issue Từ Crash Report

```javascript
// Khi app crash, tự tạo Linear issue
const fetch = require('node-fetch');

async function baoLoiLinear(loi) {
  const query = `
    mutation {
      issueCreate(input: {
        teamId: "TEAM_ID_CUA_BAN",
        title: "🚨 Crash: ${loi.message}",
        description: "Stack trace:\n```\n${loi.stack}\n```",
        labelIds: ["LABEL_BUG_ID"]
      }) {
        issue { identifier }
      }
    }`;

  const res = await fetch('https://api.linear.app/graphql', {
    method: 'POST',
    headers: {
      'Authorization': process.env.LINEAR_API_KEY,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ query })
  });

  return res.json();
}
```

---

## 33. So Sánh Chi Tiết: Linear vs Jira

### 33.1 Tốc Độ Thực Tế

| Thao tác | Linear | Jira |
|---|---|---|
| Mở issue | < 100ms | 1-3 giây |
| Chuyển status | 1 phím tắt | 3-4 clicks |
| Tạo issue | 1 phím (C) | 2-3 clicks + form dài |
| Tìm kiếm | Tức thì | 2-5 giây |

### 33.2 Chi Phí Ẩn

| | Linear | Jira |
|---|---|---|
| Thời gian setup | 30 phút | Vài ngày (cấu hình workflow) |
| Đào tạo team mới | 1 giờ | 1-2 ngày |
| Admin duy trì | Gần như không | Cần Jira admin chuyên trách |
| Plugin cần mua | Ít (tích hợp sẵn đủ) | Nhiều (Tempo, ScriptRunner...) |

### 33.3 Khi Nào Vẫn Nên Dùng Jira?

- Công ty > 500 người với quy trình phê duyệt phức tạp
- Cần tích hợp sâu với Confluence, Bitbucket (hệ sinh thái Atlassian)
- Yêu cầu compliance/audit nghiêm ngặt
- Team đã quen Jira 10 năm, chi phí chuyển đổi cao

---

*Cập nhật: 2026-10-10*

---

## 34. Linear Cho Non-Developers

### 34.1 Designer Dùng Linear

```
Team: Design
Issue types:
  - "Thiết kế màn hình đăng nhập" (gắn Figma link)
  - "Review UI trang thanh toán"

Workflow:
  Backlog → Designing → Review → Handoff → Done
  (tùy chỉnh status cho team design)
```

### 34.2 Marketing Dùng Linear

```
Team: Marketing
Projects: "Chiến dịch Tết 2026"
Issues:
  - "Viết bài blog giới thiệu tính năng mới"
  - "Thiết kế banner quảng cáo"
  - "Lên lịch đăng Facebook tuần này"
```

### 34.3 QA / Tester Dùng Linear

```
Quy trình báo bug chuẩn:
1. Tạo issue với template bug (môi trường, steps, expected/actual)
2. Priority: Urgent nếu sập production
3. Assign cho dev phụ trách module đó
4. Khi dev fix xong → QA verify → mới chuyển Done
5. Nếu vẫn lỗi → reopen với comment
```

---

## 35. Đo Lường Hiệu Suất Team

### 35.1 Metrics Nên Theo Dõi

| Metric | Cách tính | Mục tiêu |
|---|---|---|
| **Velocity** | Points hoàn thành / cycle | Ổn định hoặc tăng dần |
| **Cycle time** | Ngày từ Todo → Done (trung bình) | < 5 ngày |
| **Bug rate** | Số bug / tổng issues | < 20% |
| **Carryover** | Issues chưa xong chuyển cycle sau | < 20% |
| **Triage time** | Thời gian từ tạo → được phân loại | < 24h |

### 35.2 Đọc Velocity Đúng Cách

```
Cycle 1: 20 points ✅
Cycle 2: 25 points ✅
Cycle 3: 15 points ⚠️ (có người nghỉ phép?)
Cycle 4: 30 points ✅

→ Velocity trung bình: ~22 points/cycle
→ Planning cycle tiếp: nhận ~20 points (thận trọng)
```

⚠️ **Đừng dùng velocity để so sánh người với người!** Nó đo năng lực team, không phải cá nhân.

---

## 36. Mẫu Template Issues

### 36.1 Template Bug Report

```markdown
## 🐛 Mô tả lỗi
<!-- Mô tả ngắn gọn lỗi là gì -->

## 🔁 Các bước tái hiện
1.
2.
3.

## ✅ Kết quả mong đợi
<!-- Đáng lẽ phải như thế nào -->

## ❌ Kết quả thực tế
<!-- Đang bị như thế nào -->

## 🖼️ Ảnh / Video
<!-- Dán ảnh hoặc link video -->

## 💻 Môi trường
- OS:
- Browser / App version:
- Tài khoản test:

## 📝 Ghi chú thêm
```

### 36.2 Template Feature Request

```markdown
## 💡 Tính năng đề xuất
<!-- Mô tả tính năng -->

## 🎯 Vấn đề giải quyết
<!-- Người dùng đang gặp khó khăn gì? -->

## 📋 Tiêu chí hoàn thành (Acceptance Criteria)
- [ ]
- [ ]
- [ ]

## 🎨 Thiết kế
<!-- Link Figma nếu có -->

## ⚠️ Rủi ro / Lưu ý
```

### 36.3 Template Task Kỹ Thuật

```markdown
## 📝 Mô tả
<!-- Cần làm gì -->

## 🔧 Phương án kỹ thuật
<!-- Định làm như thế nào -->

## ✅ Definition of Done
- [ ] Code xong + self-review
- [ ] Viết unit test
- [ ] PR được approve
- [ ] Deploy staging + test
- [ ] Cập nhật tài liệu (nếu cần)
```

---

*Cập nhật: 2026-10-10*
