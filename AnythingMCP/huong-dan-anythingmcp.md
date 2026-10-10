# Hướng Dẫn Toàn Tập AnythingMCP — Biến Mọi API Thành Công Cụ Cho AI

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10

---

## Mục Lục

1. [MCP Là Gì?](#1-mcp-là-gì)
2. [AnythingMCP Là Gì?](#2-anythingmcp-là-gì)
3. [Cài Đặt](#3-cài-đặt)
4. [Tổng Quan Giao Diện](#4-tổng-quan-giao-diện)
5. [Connectors — Trái Tim Của AnythingMCP](#5-connectors--trái-tim-của-anythingmcp)
6. [REST Connector Chi Tiết](#6-rest-connector-chi-tiết)
7. [SOAP/WSDL Connector](#7-soapwsdl-connector)
8. [GraphQL Connector](#8-graphql-connector)
9. [Database Connector](#9-database-connector)
10. [262 Connectors Có Sẵn](#10-262-connectors-có-sẵn)
11. [Tạo MCP Server & API Key](#11-tạo-mcp-server--api-key)
12. [Kết Nối Với Claude](#12-kết-nối-với-claude)
13. [Kết Nối Với ChatGPT](#13-kết-nối-với-chatgpt)
14. [Kết Nối Với Copilot & Cursor](#14-kết-nối-với-copilot--cursor)
15. [Authentication Modes](#15-authentication-modes)
16. [RBAC — Phân Quyền Chi Tiết](#16-rbac--phân-quyền-chi-tiết)
17. [Audit Log](#17-audit-log)
18. [Response Shaping](#18-response-shaping)
19. [Read-Only Safeguards](#19-read-only-safeguards)
20. [Backup & Restore](#20-backup--restore)
21. [Dự Án Thực Tế: Nối ERP Vào Claude](#21-dự-án-thực-tế-nối-erp-vào-claude)
22. [So Sánh Với Giải Pháp Khác](#22-so-sánh-với-giải-pháp-khác)
23. [Troubleshooting](#23-troubleshooting)
24. [Best Practices](#24-best-practices)
25. [FAQ](#25-faq)

---

## 1. MCP Là Gì?

### 1.1. Định Nghĩa Đơn Giản

**MCP (Model Context Protocol)** là chuẩn mở do **Anthropic** tạo ra, giúp AI kết nối với "thế giới bên ngoài" — database, API, file, công cụ... một cách thống nhất.

Hãy tưởng tượng thế này:

```
Trước MCP:
  Claude ──plugin riêng──▶ Database A
  Claude ──plugin riêng──▶ API B
  Claude ──plugin riêng──▶ Tool C
  (mỗi cái viết code riêng, mệt!)

Sau MCP:
  Claude ──MCP──▶ MCP Server A ──▶ Database A
         ──MCP──▶ MCP Server B ──▶ API B
         ──MCP──▶ MCP Server C ──▶ Tool C
  (1 chuẩn chung cho tất cả!)
```

MCP giống như **"USB-C cho AI"** — một cổng chuẩn duy nhất, cắm thiết bị nào cũng được.

### 1.2. Các Khái Niệm Cơ Bản

| Khái niệm | Giải thích đơn giản |
|-----------|---------------------|
| **MCP Client** | Ứng dụng AI: Claude, ChatGPT, Cursor, Copilot... |
| **MCP Server** | Chương trình "phục vụ" công cụ cho AI gọi |
| **Tool** | 1 chức năng AI có thể gọi, ví dụ: `get_orders`, `search_customers` |
| **Resource** | Dữ liệu AI có thể đọc, ví dụ: file, nội dung database |
| **Prompt** | Template câu lệnh có sẵn |

### 1.3. Luồng Hoạt Động

```
1. Bạn hỏi Claude: "Đơn hàng #12345 đang ở đâu?"
         ↓
2. Claude thấy có tool "track_parcel" từ MCP server
         ↓
3. Claude gọi: track_parcel(tracking_id="12345")
         ↓
4. MCP server gọi API thật của đơn vị vận chuyển
         ↓
5. Trả kết quả về → Claude diễn giải bằng tiếng Việt cho bạn
```

### 1.4. Vấn Đề MCP Giải Quyết

Trước đây, muốn AI gọi được API của công ty bạn, bạn phải:
1. Viết MCP server bằng code (TypeScript/Python)
2. Tự lo authentication, logging, phân quyền
3. Mỗi hệ thống (ERP, CRM, database...) viết 1 server riêng

→ **Mất hàng tuần cho mỗi hệ thống.** AnythingMCP sinh ra để xóa bỏ công việc này.

---

## 2. AnythingMCP Là Gì?

### 2.1. Định Nghĩa

**AnythingMCP** là **MCP gateway tự host** (self-hosted), mã nguồn mở (source-available), biến **bất kỳ API nào** — REST, SOAP, GraphQL, OData, SQL — thành **MCP tools** cho Claude, ChatGPT, Gemini, Copilot, Cursor dùng.

**Không cần viết code. Không cần SDK.** Trỏ vào API → cấu hình → kết nối.

- **Repo:** https://github.com/HelpCode-ai/anythingmcp
- **Website:** https://anythingmcp.com
- **Cloud:** https://cloud.anythingmcp.com

### 2.2. Nguồn Gốc

AnythingMCP được tách ra từ hệ thống production thật của một tập đoàn công nghiệp Đức — nơi họ cần nối **15+ hệ thống legacy** (ERP, CRM, SOAP, database on-premise) với AI agents. Sau khi chạy ổn định trong production, họ open-source nó.

### 2.3. AnythingMCP Khác Gì Viết MCP Server Tay?

| Tiêu chí | AnythingMCP | Tự viết MCP server |
|----------|-------------|-------------------|
| Viết code | ❌ Không cần | ✅ Phải viết |
| Import OpenAPI/Postman/WSDL | ✅ Tự động | ❌ Tự parse tay |
| Authentication | ✅ Có sẵn nhiều loại | ❌ Tự làm |
| RBAC phân quyền | ✅ Có sẵn | ❌ Tự làm |
| Audit log | ✅ Tự động | ❌ Tự làm |
| Visual editor | ✅ | ❌ |
| Thời gian cho 1 API | ~15 phút | Vài ngày - vài tuần |

### 2.4. Giấy Phép (License)

AnythingMCP dùng **Business Source License 1.1 (BSL-1.1)** — giống mô hình của Sentry, MariaDB, Terraform:

- ✅ Đọc source, fork, sửa đổi, tự host: **được**
- ✅ Dùng nội bộ công ty, cá nhân, học thuật: **miễn phí**
- ❌ Bán lại dưới dạng dịch vụ SaaS cho bên thứ ba: **cần license thương mại**
- 📅 Ngày **2030-03-04**, license tự động chuyển thành **Apache 2.0** (open-source hoàn toàn)

---

## 3. Cài Đặt

### 3.1. Yêu Cầu Hệ Thống

| Yêu cầu | Chi tiết |
|---------|----------|
| Docker | Version 24+ |
| bash | Có sẵn trên Linux/macOS |
| openssl | Để sinh key mã hóa |
| RAM | Tối thiểu 2GB (khuyên 4GB+) |
| Disk | Tối thiểu 10GB |

Trên macOS: mở **Docker Desktop** trước khi cài.

### 3.2. Cài Bằng Docker Compose (Khuyên Dùng)

```bash
# 1. Clone repo
git clone https://github.com/HelpCode-ai/anythingmcp.git
cd anythingmcp

# 2. Chạy script setup (tự sinh key, tạo config)
./setup.sh
```

Script `setup.sh` sẽ:
1. Kiểm tra Docker, bash, openssl
2. Sinh `ENCRYPTION_KEY` ngẫu nhiên (dùng mã hóa credentials)
3. Tạo file `.env` và `docker-compose.yml`
4. Pull image và start container

```bash
# 3. Mở trình duyệt
# http://localhost:3000
```

### 3.3. Đăng Ký Tài Khoản Admin Đầu Tiên

> ⚠️ **CỰC KỲ QUAN TRỌNG:** Tài khoản **đầu tiên** đăng ký sẽ tự động thành **Admin**. Hãy đăng ký ngay sau khi setup xong!

Nếu server của bạn mở ra internet trong lúc setup, hãy:
- Cấu hình firewall chỉ cho IP của bạn truy cập, hoặc
- Bind UI về `127.0.0.1` tạm thời

đến khi tạo xong tài khoản admin.

### 3.4. Các Cách Deploy Khác

| Cách | Khi nào dùng |
|------|--------------|
| **Docker Compose** (trên) | VPS, server công ty — phổ biến nhất |
| **Cloud** (cloud.anythingmcp.com) | Dùng thử nhanh, không muốn tự host |
| **Railway** (1-click) | Deploy lên Railway trong 2 phút |
| **DigitalOcean** (Marketplace) | Dùng hạ tầng DigitalOcean |

**Deploy lên Railway:**
1. Bấm nút "Deploy on Railway" trong README của repo
2. Railway tự tạo service + PostgreSQL
3. **Nhớ lưu `ENCRYPTION_KEY`** — mất key này là mất toàn bộ credentials đã mã hóa!

### 3.5. Cấu Trúc Docker Compose

```yaml
# docker-compose.yml (rút gọn)
services:
  anythingmcp:
    image: helpcodeai/anythingmcp:latest
    ports:
      - "3000:3000"
    environment:
      - ENCRYPTION_KEY=${ENCRYPTION_KEY}  # ⚠️ KHÔNG BAO GIỜ để lộ/mất
      - DATABASE_URL=postgres://...
    volumes:
      - ./data:/app/data
    depends_on:
      - postgres

  postgres:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
```

### 3.6. Cập Nhật Version Mới

```bash
cd anythingmcp
git pull                    # lấy code mới
docker compose pull         # pull image mới
docker compose up -d        # restart với image mới
# Dữ liệu trong PostgreSQL volume không mất
```

---

## 4. Tổng Quan Giao Diện

Sau khi đăng nhập `http://localhost:3000`, bạn thấy:

```
┌──────────────────────────────────────────────────┐
│  AnythingMCP                    [User] [Settings]  │
├──────────┬───────────────────────────────────────┤
│          │                                       │
│ 📡       │   MCP Servers                         │
│ Connectors│   ┌─────────┐ ┌─────────┐              │
│          │   │ ERP     │ │ CRM     │  [+ New]     │
│ 🔑       │   │ ● Live  │ │ ● Live  │              │
│ API Keys │   └─────────┘ └─────────┘              │
│          │                                       │
│ 👥       │   Connectors                          │
│ Users    │   ┌────┐ ┌────┐ ┌────┐                 │
│          │   │REST│ │SOAP│ │SQL │  [+ New]        │
│ 📋       │   └────┘ └────┘ └────┘                 │
│ Audit Log│                                       │
│          │                                       │
│ ⚙️       │                                       │
│ Settings │                                       │
└──────────┴───────────────────────────────────────┘
```

| Menu | Chức năng |
|------|-----------|
| **MCP Servers** | Các MCP server đã tạo — mỗi server là 1 tập hợp tools để AI kết nối vào |
| **Connectors** | Kết nối đến từng hệ thống (REST API, SOAP, DB...) |
| **API Keys** | Key để AI client xác thực khi gọi MCP server |
| **Users** | Quản lý user, roles |
| **Audit Log** | Lịch sử mọi tool call |
| **Settings** | Cấu hình chung, SSO, backup |

**Luồng làm việc cơ bản:**
```
1. Tạo Connector (nối vào API/DB của bạn)
        ↓
2. Tạo MCP Server (gom 1 hoặc nhiều connectors)
        ↓
3. Mint API Key (cho AI client dùng)
        ↓
4. Kết nối Claude/ChatGPT/Cursor vào MCP server
        ↓
5. Hỏi AI → AI gọi tools → trả lời bạn
```

---

## 5. Connectors — Trái Tim Của AnythingMCP

### 5.1. Connector Là Gì?

**Connector** là "cầu nối" từ AnythingMCP đến **một hệ thống** của bạn (1 REST API, 1 SOAP service, 1 database...). Mỗi connector sinh ra một tập **tools** mà AI có thể gọi.

```
Hệ thống của bạn          AnythingMCP              AI Client
┌──────────────┐    ┌──────────────────┐    ┌──────────────┐
│ REST API     │───▶│ REST Connector   │───▶│              │
│ (bán hàng)   │    │ → tools:         │    │   Claude     │
│              │    │   - get_orders   │    │              │
│              │    │   - create_order │    │  ChatGPT     │
├──────────────┤    ├──────────────────┤    │              │
│ PostgreSQL   │───▶│ SQL Connector    │───▶│   Cursor     │
│ (kho hàng)   │    │ → tools:         │    │              │
│              │    │   - query_stock  │    │   Copilot    │
└──────────────┘    └──────────────────┘    └──────────────┘
```

### 5.2. Các Loại Connector

| Loại | Dùng cho | Cách tạo |
|------|----------|----------|
| **REST** | Mọi REST API | Import OpenAPI/Swagger, Postman Collection, cURL, hoặc cấu hình tay |
| **SOAP** | Hệ thống legacy dùng SOAP/WSDL | Import file WSDL — tự parse operations |
| **GraphQL** | API GraphQL | Nhập endpoint + introspection |
| **Database** | PostgreSQL, MySQL, MariaDB, MSSQL, Oracle, MongoDB, SQLite | Nhập connection string |
| **MCP Bridge** | MCP server có sẵn khác | Nhập URL của MCP server → gom vào gateway |

### 5.3. Tạo Connector Mới — Các Bước Chung

1. Sidebar → **Connectors** → **+ New Connector**
2. Chọn loại: REST / SOAP / GraphQL / Database / MCP Bridge
3. Điền thông tin kết nối + authentication
4. **Test connection** — đảm bảo nối được
5. **Import/định nghĩa tools** — chọn operations nào expose thành MCP tools
6. **Save** → connector sẵn sàng

---

## 6. REST Connector Chi Tiết

### 6.1. Các Cách Import

| Cách | Khi nào dùng |
|------|--------------|
| **OpenAPI/Swagger** | API có file `openapi.json`/`swagger.yaml` — tốt nhất, tự sinh đầy đủ |
| **Postman Collection** | Team đã có Postman collection sẵn |
| **cURL command** | Chỉ cần import 1-2 endpoints nhanh |
| **Thủ công** | API không có spec — tự định nghĩa từng tool |

### 6.2. Import Từ OpenAPI (Ví Dụ)

Giả sử công ty bạn có API bán hàng với spec tại `https://api.congty.vn/openapi.json`:

**Bước 1:** New Connector → **REST** → **Import OpenAPI** → dán URL.

**Bước 2:** AnythingMCP parse và hiện danh sách endpoints:

```
✅ GET    /orders            → tool: list_orders
✅ GET    /orders/{id}       → tool: get_order
✅ POST   /orders            → tool: create_order
✅ PATCH  /orders/{id}      → tool: update_order
✅ DELETE /orders/{id}       → tool: delete_order
✅ GET    /customers         → tool: list_customers
✅ GET    /products          → tool: list_products
```

**Bước 3:** Bỏ chọn những endpoints không muốn expose (ví dụ `delete_order` nếu muốn read-only).

**Bước 4:** Cấu hình authentication (xem mục 15).

**Bước 5:** Save → test thử từng tool trong UI.

### 6.3. Visual Tool Editor

Mỗi tool có thể chỉnh trong visual editor:

```
Tool: get_order
┌─────────────────────────────────────────────┐
│ Path:   GET /orders/{id}                    │
│                                             │
│ Parameters:                                 │
│  ┌──────────┬────────┬──────────────────┐  │
│  │ Tên      │ Vị trí │ Bắt buộc │ Mô tả  │  │
│  ├──────────┼────────┼──────────┼────────┤  │
│  │ id       │ path   │ ✅       │ ID đơn │  │
│  │ include  │ query  │ ❌       │ ...    │  │
│  └──────────┴────────┴──────────┴────────┘  │
│                                             │
│ Headers: Authorization: Bearer {{API_TOKEN}} │
│                                             │
│ Response mapping: [cấu hình] (mục 18)       │
│ Annotations: ☑ readOnlyHint                 │
└─────────────────────────────────────────────┘
```

**Biến môi trường trong connector:** Dùng cú pháp `{{VAR}}` — ví dụ `{{API_TOKEN}}`. Giá trị biến được cấu hình riêng, **AI không bao giờ thấy** giá trị thật.

### 6.4. Import Từ cURL

Copy cURL từ DevTools hoặc docs:

```bash
curl -X GET "https://api.congty.vn/orders?status=pending" \
  -H "Authorization: Bearer abc123" \
  -H "Content-Type: application/json"
```

Paste vào AnythingMCP → nó tự tách thành:
- Method: GET
- URL: https://api.congty.vn/orders
- Query param: status=pending
- Header: Authorization, Content-Type

### 6.5. Ví Dụ Thực Tế: Nối API Giao Hàng

```yaml
# Connector: Giao Hàng Nhanh (tưởng tượng)
Base URL: https://api.ghn.vn

Tools:
  - create_shipping_order:
      POST /v2/shipping-order/create
      Body: { to_name, to_phone, to_address, weight, cod_amount }
      Auth: Header "Token: {{GHN_TOKEN}}"

  - track_order:
      POST /v2/shipping-order/detail
      Body: { order_code }
      → AI dùng khi user hỏi "đơn hàng của tôi đâu rồi?"

  - calculate_fee:
      POST /v2/shipping-order/fee
      Body: { from_district, to_district, weight }
      → AI dùng khi user hỏi "ship từ HN vào SG bao nhiêu?"
```

---

## 7. SOAP/WSDL Connector

### 7.1. Vì Sao SOAP Vẫn Quan Trọng?

Nhiều hệ thống doanh nghiệp "cổ" (ERP, ngân hàng, bảo hiểm, chính phủ) vẫn dùng **SOAP** — giao thức ra đời từ 1998. Viết MCP server cho SOAP bằng tay cực khổ vì phải parse XML/WSDL. AnythingMCP làm tự động.

### 7.2. Import Từ WSDL

**Bước 1:** New Connector → **SOAP** → nhập URL WSDL:
```
https://erp.congty.vn/services/OrderService?wsdl
```

**Bước 2:** AnythingMCP parse WSDL, liệt kê operations:

```
✅ GetOrderStatus(orderId: string)
✅ CreateSalesOrder(order: OrderType)
✅ GetCustomerInfo(customerId: string)
✅ UpdateInventory(sku: string, quantity: int)
```

**Bước 3:** Cấu hình auth — SOAP thường dùng:
- **WS-Security** (username token trong SOAP header)
- **Client certificate** (file .p12/.pem)
- **Basic Auth** trên HTTP

**Bước 4:** Test từng operation → Save.

### 7.3. Ví Dụ: Nối SAP Business One (Qua SOAP)

```
Connector: SAP B1
WSDL: https://sap.congty.vn:50000/b1s/v1/$metadata... (qua SOAP adapter)

Tools sinh ra:
  - b1_get_orders      → lấy đơn hàng
  - b1_get_items       → tra cứu mặt hàng
  - b1_create_order    → tạo đơn (có thể tắt nếu muốn read-only)
```

> 💡 AnythingMCP còn có **262 connectors có sẵn** (mục 10) — trong đó có sẵn SAP S/4HANA và SAP Business One, không cần tự import WSDL!

---

## 8. GraphQL Connector

### 8.1. Tạo GraphQL Connector

1. New Connector → **GraphQL**
2. Nhập **Endpoint**: `https://api.congty.vn/graphql`
3. Cấu hình auth (thường là Bearer token)
4. Bấm **Introspect** — AnythingMCP tự query schema, liệt kê queries/mutations
5. Chọn queries/mutations muốn expose

### 8.2. Ví Dụ

```graphql
# Query có sẵn trong schema
query GetProduct($id: ID!) {
  product(id: $id) {
    name
    price
    stock
  }
}
```

→ AnythingMCP tạo tool `get_product(id)` mà AI gọi được.

**Mutations** (ghi dữ liệu) có thể tắt riêng nếu muốn read-only:
```
✅ query: get_product, list_products, search_products
❌ mutation: create_product, delete_product  (tắt)
```

---

## 9. Database Connector

### 9.1. Các Engine Hỗ Trợ

| Engine | Connection string mẫu |
|--------|----------------------|
| PostgreSQL | `postgresql://user:pass@host:5432/db` |
| MySQL / MariaDB | `mysql://user:pass@host:3306/db` |
| MSSQL | `sqlserver://user:pass@host:1433;database=db` |
| Oracle | `oracle://user:pass@host:1521/service` |
| MongoDB | `mongodb://user:pass@host:27017/db` |
| SQLite | `file:/path/to/db.sqlite` |

### 9.2. Cách Hoạt Động

1. Nhập connection string → **Test**
2. AnythingMCP **introspect schema**: đọc danh sách bảng, cột, kiểu dữ liệu
3. Tự sinh tools:
   - `list_tables` — liệt kê bảng
   - `describe_table(table)` — xem cấu trúc bảng
   - `query(sql)` — chạy SELECT (bị chặn INSERT/UPDATE/DELETE!)

### 9.3. Read-Only Mặc Định

> 🔒 Database connector **chỉ cho phép SELECT**. Mọi câu lệnh ghi (INSERT, UPDATE, DELETE, DROP) và stacked statements (nhiều lệnh cách nhau bằng `;`) đều bị chặn.

```
AI gọi: query("SELECT * FROM orders WHERE status = 'pending' LIMIT 10")
✅ Chạy được

AI gọi: query("DELETE FROM orders WHERE id = 1")
❌ Bị chặn: "Write operations are not allowed"
```

Đây là thiết kế an toàn — AI đọc được dữ liệu nhưng không thể phá.

### 9.4. Ví Dụ Thực Tế

```
Connector: DB Kho Hàng (PostgreSQL)

AI chat:
  Bạn: "Trong kho còn bao nhiêu áo thun size M?"
  Claude:
    → gọi describe_table("products") để biết cấu trúc
    → gọi query("SELECT SUM(quantity) FROM inventory
                 WHERE product_name ILIKE '%áo thun%' AND size = 'M'")
    → trả lời: "Còn 1,247 chiếc áo thun size M ạ!"
```

---

## 10. 262 Connectors Có Sẵn

### 10.1. Catalog Là Gì?

Thay vì tự cấu hình từ đầu, bạn vào **Catalog**, chọn connector có sẵn, điền credentials → xong trong 1 phút.

### 10.2. Các Nhóm Nổi Bật

| Nhóm | Ví dụ connectors |
|------|-----------------|
| **ERP** | SAP S/4HANA, SAP Business One, weclapp, Xentral |
| **Vận chuyển** | DHL, DPD, GLS, Deutsche Bahn |
| **E-commerce** | Shopware 6, WooCommerce, Etsy |
| **Nhân sự** | Personio, HRWorks |
| **Kế toán** | DATEV, Billomat |
| **Tra cứu** | Handelsregister (đăng ký kinh doanh Đức), VIES VAT (thuế EU), OpenPLZ |
| **Địa lý** | HERE Geocoding |
| **Nhắn tin** | WhatsApp |

### 10.3. Cài Connector Có Sẵn (Ví Dụ DHL)

1. **Catalog** → tìm "DHL" → **Install**
2. Điền **API key** của DHL
3. Connector tự tạo tools:
   - `dhl_track(shipment_id)` — tra cứu vận đơn
   - `dhl_get_quote(...)` — báo giá
4. Thêm connector vào MCP server → AI dùng được ngay

```
Bạn hỏi Claude: "Kiện hàng 1234567890 của DHL đang ở đâu?"
Claude → gọi dhl_track("1234567890") → "Kiện hàng đang ở trung tâm phân loại Hà Nội, dự kiến giao ngày mai."
```

---

## 11. Tạo MCP Server & API Key

### 11.1. MCP Server Là Gì (Trong AnythingMCP)?

**MCP Server** trong AnythingMCP là "gói" gom một hoặc nhiều connectors lại, expose thành **một endpoint duy nhất** cho AI client kết nối vào.

```
                    ┌─────────────────────────────┐
                    │   MCP Server: "Bán hàng"      │
┌──────────┐        │                             │
│ REST     │──┐     │  Tools:                     │
│ (API bán)│  ├────▶│   - list_orders             │
└──────────┘  │     │   - create_order            │
              │     │   - get_customer            │
┌──────────┐  │     │   - query_stock (từ SQL)    │
│ SQL      │──┘     │                             │
│ (DB kho) │        │  URL: https://.../mcp/sales │
└──────────┘        └─────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                 Claude              Cursor
```

### 11.2. Tạo MCP Server

1. Sidebar → **MCP Servers** → **+ New**
2. Đặt tên: ví dụ `sales` (slug sẽ là `sales`)
3. **Chọn connectors** muốn gom vào
4. **Chọn tools** cụ thể (có thể bỏ bớt tools nhạy cảm)
5. **Save**

Endpoint của server sẽ là:
```
https://your-domain/mcp/sales
```

### 11.3. Mint API Key

Mỗi AI client (hoặc mỗi user) nên có key riêng để dễ quản lý và thu hồi:

1. Mở MCP server → tab **API Keys** → **Mint new key**
2. Đặt tên gợi nhớ: `claude-desktop-cua-an`, `cursor-cua-binh`
3. Copy key — **chỉ hiện 1 lần!**
4. Key có dạng: `amp_xxxxxxxxxxxxxxxx`

**Quản lý key:**
| Thao tác | Khi nào |
|----------|---------|
| **Revoke** | Nhân viên nghỉ việc, key bị lộ |
| **Xem usage** | Biết key nào gọi bao nhiêu lần |
| **Giới hạn tools** | Key chỉ được dùng subset tools |

---

## 12. Kết Nối Với Claude

### 12.1. Claude Desktop (Custom Connector)

1. Mở **Claude Desktop** → **Settings** → **Connectors** (hoặc **Integrations**)
2. **Add custom connector**
3. Điền:
   - **Name:** `AnythingMCP Sales`
   - **URL:** `https://your-domain/mcp/sales`
   - **API Key:** `amp_xxx...` (key đã mint)
4. **Save** → Claude hiện tools mới trong cuộc chat

### 12.2. Sử Dụng

```
Bạn: "Cho tôi xem 5 đơn hàng mới nhất"

Claude (tự động):
  🔧 Gọi tool: list_orders(limit=5, sort="created_at desc")
  📦 Nhận: [{id: 1001, customer: "Nguyễn Văn A", total: 2500000, ...}, ...]

Claude trả lời:
  "5 đơn hàng mới nhất:
   1. #1001 — Nguyễn Văn A — 2,500,000đ — đang giao
   2. #1002 — Trần Thị B — 1,200,000đ — chờ xác nhận
   ..."
```

### 12.3. Claude Code (Cho Developer)

```bash
# Thêm MCP server vào Claude Code
claude mcp add --transport http anythingmcp-sales \
  https://your-domain/mcp/sales \
  --header "Authorization: Bearer amp_xxx..."
```

Sau đó trong Claude Code, AI tự dùng tools khi bạn yêu cầu liên quan đến dữ liệu công ty.

---

## 13. Kết Nối Với ChatGPT

### 13.1. Điều Kiện

ChatGPT hỗ trợ MCP qua tính năng **Custom Connectors** (tài khoản Plus/Team/Enterprise, tùy thời điểm OpenAI mở).

### 13.2. Các Bước

1. ChatGPT → **Settings** → **Connectors** (hoặc **Apps**)
2. **Add custom connector** → chọn **MCP**
3. Điền URL + API key của AnythingMCP server
4. Trong cuộc chat mới, bật connector lên
5. Hỏi như bình thường — ChatGPT tự gọi tools

### 13.3. Lưu Ý

- ChatGPT có thể hỏi xác nhận trước khi gọi tool nhạy cảm (ví dụ tool ghi dữ liệu) — đây là hành vi tốt, đừng tắt
- Nếu tool không hiện, kiểm tra API key còn hiệu lực không

---

## 14. Kết Nối Với Copilot & Cursor

### 14.1. Cursor

**Cách 1 — Qua Settings:**
1. Cursor → **Settings** (`Ctrl+,`) → **MCP**
2. **Add new MCP server**:
```json
{
  "mcpServers": {
    "anythingmcp-sales": {
      "url": "https://your-domain/mcp/sales",
      "headers": {
        "Authorization": "Bearer amp_xxx..."
      }
    }
  }
}
```

**Cách 2 — File config trực tiếp:**
Mở `~/.cursor/mcp.json` (hoặc `.cursor/mcp.json` trong project), thêm như trên.

### 14.2. GitHub Copilot

Copilot hỗ trợ MCP server qua cấu hình trong VS Code:

`.vscode/mcp.json`:
```json
{
  "servers": {
    "anythingmcp-sales": {
      "type": "http",
      "url": "https://your-domain/mcp/sales",
      "headers": {
        "Authorization": "Bearer amp_xxx..."
      }
    }
  }
}
```

### 14.3. Gemini

Trong **Gemini** (AI Studio hoặc app), vào phần Extensions/Tools → thêm MCP server URL + key tương tự.

> 💡 **Nguyên tắc chung:** Bất kỳ client nào "nói được MCP" đều kết nối được với AnythingMCP. Một connector cấu hình 1 lần → dùng được ở mọi client.

---

## 15. Authentication Modes

### 15.1. Các Loại Auth Hỗ Trợ

Khi connector gọi API của bạn, nó cần xác thực. AnythingMCP hỗ trợ:

| Loại | Khi nào dùng | Ví dụ |
|------|--------------|-------|
| **API Key** | Đơn giản nhất | Header `X-API-Key: {{KEY}}` |
| **Bearer <redacted>** | JWT, OAuth2 access token | Header `Authorization: Bearer {{TOKEN}}` |
| **Basic Auth** | Username + password | Tự encode Base64 |
| **OAuth2 Client Credentials** | Server-to-server | Tự lấy token, tự refresh |
| **OAuth2 PKCE** | User login qua trình duyệt | Flow redirect |
| **OAuth 1.0a** | API cũ (Twitter API v1...) | Ký request bằng HMAC-SHA1 |
| **HMAC Signing** | API yêu cầu ký request | Ký mỗi request bằng secret |
| **WS-Security** | SOAP enterprise | UsernameToken trong SOAP header |
| **Client Certificate** | mTLS | File .p12/.pem |
| **LOGIN_TOKEN** | Hệ thống tự chế | Custom flow của AnythingMCP |

### 15.2. Ví Dụ: OAuth2 Client Credentials

Nhiều API enterprise dùng flow này:

```
1. AnythingMCP gọi POST https://auth.congty.vn/token
   Body: grant_type=client_credentials
         client_id={{CLIENT_ID}}
         client_secret={{CLIENT_SECRET}}

2. Nhận: { "access_token": "eyJ...", "expires_in": 3600 }

3. Mọi API call sau tự gắn:
   Authorization: Bearer eyJ...

4. Gần hết hạn → tự lấy token mới (bạn không cần làm gì)
```

Cấu hình trong connector:
```
Auth type: OAuth2 Client Credentials
Token URL: https://auth.congty.vn/token
Client ID: {{OAUTH_CLIENT_ID}}
Client Secret: {{OAUTH_CLIENT_SECRET}}
Scope: read write (nếu cần)
```

### 15.3. Credentials Được Mã Hóa

> 🔒 Mọi credentials lưu trong AnythingMCP được mã hóa bằng **AES-256-GCM** với `ENCRYPTION_KEY` của bạn.

Điều này có nghĩa:
- Kẻ có được database cũng không đọc được password/API key nếu không có `ENCRYPTION_KEY`
- **Mất `ENCRYPTION_KEY` = mất toàn bộ credentials** → phải nhập lại từ đầu
- Backup `ENCRYPTION_KEY` ở nơi an toàn (password manager), tách khỏi backup database!

---

## 16. RBAC — Phân Quyền Chi Tiết

### 16.1. RBAC Là Gì?

**RBAC (Role-Based Access Control)**: gán quyền theo vai trò, không gán cho từng người.

```
Roles:                    Được dùng tools:
┌──────────────┐          ✅ list_orders, get_order
│ Nhân viên    │────────▶ ✅ list_customers
│ bán hàng     │          ❌ delete_order
└──────────────┘          ❌ query (DB trực tiếp)

┌──────────────┐          ✅ Tất cả tools bán hàng
│ Quản lý      │────────▶ ✅ create_order, update_order
└──────────────┘          ❌ delete_order (chỉ admin)

┌──────────────┐          ✅ Mọi thứ
│ Admin        │────────▶ ✅ Quản lý user, xem audit log
└──────────────┘
```

### 16.2. Tạo Role & Gán Quyền

1. **Users** → **Roles** → **+ New Role**
2. Đặt tên: `sales-staff`
3. Chọn MCP server → **tick chọn tools** được phép:
```
MCP Server "sales":
  ☑ list_orders
  ☑ get_order
  ☑ list_customers
  ☐ create_order
  ☐ delete_order
```
4. **Users** → gán user vào role `sales-staff`

### 16.3. Tool Whitelisting Per API Key

Ngoài role, mỗi API key cũng có thể giới hạn tools:

```
API key "cursor-cua-thuc-tap-sinh":
  Chỉ được: list_orders, get_order, list_products
  (không thấy các tools khác tồn tại)
```

> 💡 **Best practice:** Bắt đầu với **read-only** cho mọi người. Chỉ mở quyền ghi khi thực sự cần, và chỉ cho role cụ thể.

---

## 17. Audit Log

### 17.1. Audit Log Ghi Những Gì?

**Mọi** tool call đều được ghi lại:

| Trường | Ví dụ |
|--------|-------|
| Thời gian | 2026-10-10 14:32:11 |
| User / API key | `cursor-cua-an` |
| MCP server | `sales` |
| Tool | `get_order` |
| Input | `{ "id": 1001 }` |
| Output (rút gọn) | `{ "id": 1001, "total": 2500000 }` |
| Full upstream response | (bản đầy đủ mà AI không thấy — xem mục 18) |
| Duration | 234ms |
| Status | ✅ success / ❌ error |

### 17.2. Xem & Tìm Kiếm

**Audit Log** trong sidebar:
- Lọc theo user, server, tool, khoảng thời gian
- Lọc chỉ hiện lỗi
- Click vào 1 dòng để xem chi tiết input/output

### 17.3. Dùng Audit Log Để Làm Gì?

- **Debug:** "Sao AI trả lời sai?" → xem nó đã gọi tool nào với input gì
- **Bảo mật:** "Ai đã xem dữ liệu khách hàng X?" → lọc theo tool + thời gian
- **Tối ưu chi phí:** Tool nào được gọi nhiều nhất? Có tool nào không ai dùng?
- **Compliance:** Xuất log cho kiểm toán (GDPR, ISO 27001...)

### 7.4. Lưu Trữ Log

Log lưu trong PostgreSQL của chính AnythingMCP. Cấu hình retention trong **Settings**:
- Giữ 30 ngày / 90 ngày / 1 năm / vĩnh viễn
- Log cũ tự xóa theo lịch (tránh DB phình to)

---

## 18. Response Shaping

### 18.1. Vấn Đề

API thường trả về **quá nhiều** field — trong đó có dữ liệu nhạy cảm mà AI không cần thấy:

```json
// API trả về (12,172 bytes)
{
  "id": 1001,
  "customer": {
    "name": "Nguyễn Văn A",
    "phone": "0901234567",
    "iban": "DE89370400440532013000",      // ← nhạy cảm!
    "taxId": "1234567890"                  // ← nhạy cảm!
  },
  "items": [... 50 dòng ...],
  "internal_notes": "Khách này hay bom hàng",  // ← nội bộ!
  "amounts": { "gross": 2500000, "net": 2272727 }
}
```

### 18.2. Giải Pháp: Response Mapping

Trong tool editor, cấu hình **transform**:

```json
{
  "transform": {
    "mode": "select",
    "select": {
      "order": "$.id",
      "customer_name": "$.customer.name",
      "total": "$.amounts.gross",
      "status": "$.state"
    },
    "exclude": ["customer.iban", "customer.taxId"],
    "fallbackToRaw": false
  }
}
```

Kết quả AI chỉ thấy (1,072 bytes — **giảm 91%**):
```json
{
  "order": 1001,
  "customer_name": "Nguyễn Văn A",
  "total": 2500000,
  "status": "shipping"
}
```

**Lợi ích kép:**
1. 🔒 **Bảo mật:** IBAN, tax ID, ghi chú nội bộ không bao giờ đến được model
2. 💰 **Tiết kiệm:** Ít token hơn = rẻ hơn (đặc biệt với API trả tiền theo token)

### 18.3. `fallbackToRaw`

| Giá trị | Ý nghĩa |
|---------|---------|
| `true` (mặc định) | Nếu mapping lỗi → trả raw response (AI vẫn làm việc được) |
| `false` | Nếu mapping lỗi → **call thất bại luôn** |

> ⚠️ Với tool xử lý dữ liệu nhạy cảm, đặt `"fallbackToRaw": false` — thà lỗi còn hơn để lộ dữ liệu!

### 18.4. Xem Before/After

Trong tool editor, AnythingMCP cho **test với response thật** và hiện so sánh before/after — bạn thấy chính xác AI sẽ nhận được gì trước khi publish.

---

## 19. Read-Only Safeguards

### 19.1. MCP Annotations

Mỗi tool mang **annotations** cho AI biết tính chất:

| Annotation | Ý nghĩa | Ví dụ |
|------------|---------|-------|
| `readOnlyHint: true` | Tool chỉ đọc, không thay đổi gì | `list_orders`, `get_customer` |
| `destructiveHint: true` | Tool có thể xóa/phá dữ liệu | `delete_order` |
| `idempotentHint: true` | Gọi nhiều lần vẫn an toàn | `get_order` |
| `openWorldHint: true` | Tool tương tác với thế giới bên ngoài | `send_email` |

AnythingMCP **tự suy ra** annotations từ HTTP method (`GET` → readOnly), bạn có thể override từng tool.

### 19.2. Tại Sao Quan Trọng?

AI client (như Claude) đọc annotations để quyết định:
- Tool `readOnly` → gọi luôn, không hỏi
- Tool `destructive` → **hỏi xác nhận user trước khi gọi**

```
Bạn: "Xóa đơn hàng #1001 giúp tôi"

Claude:
  ⚠️ Tool "delete_order" được đánh dấu destructive.
  "Bạn có chắc muốn xóa đơn hàng #1001 không? Hành động này không thể hoàn tác."

Bạn: "Chắc chắn"
Claude: → mới gọi tool
```

### 19.3. Database: Chỉ SELECT

Như đã nói ở mục 9.3, database connector chặn cứng mọi lệnh ghi ở tầng gateway — kể cả khi AI cố tình "lách":

```sql
-- Bị chặn:
INSERT INTO users VALUES (...);
UPDATE orders SET total = 0;
DELETE FROM customers;
DROP TABLE products;
SELECT * FROM orders; DELETE FROM orders;  -- stacked statements

-- Được phép:
SELECT id, name FROM products WHERE stock < 10 LIMIT 20;
```

---

## 20. Backup & Restore

### 20.1. Cần Backup Những Gì?

| Thành phần | Chứa gì | Cách backup |
|------------|---------|-------------|
| **PostgreSQL** | Connectors, tools, users, roles, API keys (đã mã hóa), audit log | `pg_dump` |
| **ENCRYPTION_KEY** | Key giải mã credentials | Lưu trong password manager |
| **docker-compose.yml / .env** | Cấu hình deploy | Lưu trong Git (trừ secret!) |

> ⚠️ Backup database mà **không** có `ENCRYPTION_KEY` = vô dụng — không giải mã được credentials!

### 20.2. Backup PostgreSQL

```bash
# Backup thủ công
docker exec anythingmcp-postgres-1 pg_dump -U postgres anythingmcp \
  > backup-anythingmcp-$(date +%Y%m%d).sql

# Backup tự động mỗi đêm (crontab)
cat > ~/backup-mcp.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d)
docker exec anythingmcp-postgres-1 pg_dump -U postgres anythingmcp > ~/backups/mcp-$DATE.sql
gzip ~/backups/mcp-$DATE.sql
# Giữ 14 bản gần nhất
ls -t ~/backups/mcp-*.sql.gz | tail -n +15 | xargs -r rm
# ⚠️ ENCRYPTION_KEY đã lưu riêng trong password manager!
EOF
chmod +x ~/backup-mcp.sh
(crontab -l 2>/dev/null; echo "0 3 * * * ~/backup-mcp.sh") | crontab -
```

### 20.3. Restore

```bash
# 1. Dừng AnythingMCP
cd anythingmcp && docker compose down

# 2. Xóa DB cũ, tạo DB mới
docker compose up -d postgres
sleep 5

# 3. Restore (CẦN đúng ENCRYPTION_KEY trong .env!)
gunzip -c ~/backups/mcp-20261010.sql.gz | \
  docker exec -i anythingmcp-postgres-1 psql -U postgres anythingmcp

# 4. Start lại
docker compose up -d
```

### 20.4. Checklist Backup

- [ ] `pg_dump` tự động hàng ngày
- [ ] `ENCRYPTION_KEY` lưu trong password manager (1Password, Bitwarden...)
- [ ] Test restore 1 lần/quý (backup không test = không backup!)
- [ ] Backup lưu ở nơi khác với server chính

---

## 21. Dự Án Thực Tế: Nối ERP Vào Claude

### 21.1. Kịch Bản

Công ty sản xuất dùng **weclapp ERP** (có sẵn trong catalog). Giám đốc muốn hỏi Claude về tồn kho, đơn hàng mà không cần mở ERP.

### 21.2. Các Bước

**Bước 1 — Cài đặt (15 phút):**
```bash
git clone https://github.com/HelpCode-ai/anythingmcp.git
cd anythingmcp && ./setup.sh
# Mở http://localhost:3000, đăng ký admin
```

**Bước 2 — Cài connector weclapp (2 phút):**
1. Catalog → tìm "weclapp" → Install
2. Điền API key của weclapp
3. Tools tự sinh: `weclapp_get_stock`, `weclapp_get_orders`, `weclapp_get_customers`...

**Bước 3 — Response shaping (10 phút):**
- Tool `weclapp_get_customers`: exclude `bankAccount`, `taxNumber`
- Đặt `fallbackToRaw: false` cho các tool chứa dữ liệu nhạy cảm

**Bước 4 — Tạo MCP server (3 phút):**
- Tên: `erp`
- Chọn connector weclapp, bỏ tool `delete_*` (chỉ giữ read)

**Bước 5 — Phân quyền (5 phút):**
- Role `giam-doc`: full tools của server `erp`
- Role `nhan-vien-kho`: chỉ `weclapp_get_stock`
- Mint API key riêng cho từng người

**Bước 6 — Kết nối Claude (2 phút/người):**
- Claude Desktop → Add custom connector → URL + key

**Tổng:** ~40 phút từ 0 đến chatbot ERP chạy được!

### 21.3. Hội Thoại Mẫu

```
Giám đốc: "Tồn kho sản phẩm X còn bao nhiêu?"

Claude:
  🔧 weclapp_get_stock(sku="SP-X")
  📦 { sku: "SP-X", quantity: 342, warehouse: "Kho HN" }

  "Sản phẩm X còn 342 chiếc tại kho Hà Nội ạ."

Giám đốc: "Đơn hàng nào chưa giao trong tuần này?"

Claude:
  🔧 weclapp_get_orders(status="pending", week="current")
  📦 [... 15 đơn ...]

  "Có 15 đơn chưa giao, tổng giá trị 1.2 tỷ. 3 đơn quá hạn 2 ngày:
   #5012, #5015, #5021. Anh có muốn xem chi tiết đơn nào không?"
```

### 21.4. Đo Lường

| Metric | Trước | Sau |
|--------|-------|-----|
| Thời gian tra cứu tồn kho | 5 phút (mở ERP, tìm...) | 10 giây (hỏi Claude) |
| Người cần đào tạo ERP | Tất cả nhân viên | Chỉ admin |
| Rủi ro lộ dữ liệu nhạy cảm | Cao (ai cũng vào ERP) | Thấp (RBAC + response shaping) |

---

## 22. So Sánh Với Giải Pháp Khác

### 22.1. Bảng So Sánh Chi Tiết

| Tiêu chí | AnythingMCP | ContextForge | Docker MCP Gateway | MetaMCP | Composio |
|----------|-------------|--------------|-------------------|---------|----------|
| **Mô hình** | Self-hosted gateway | Self-hosted gateway | Docker-native gateway | MCP aggregator | SaaS integration platform |
| **No-code setup** | ✅ Visual editor | ⚠️ Config files | ⚠️ CLI config | ✅ UI | ✅ No-code |
| **SOAP/WSDL** | ✅ Built-in | ❌ Hiếm | ❌ | ❌ | ❌ |
| **Database connector** | ✅ 7 engines | ⚠️ Hạn chế | ❌ | ❌ | ⚠️ Một số |
| **Connectors có sẵn** | ✅ 262 | ⚠️ Ít | ❌ | ⚠️ Ít | ✅ Nhiều (SaaS) |
| **RBAC** | ✅ Chi tiết | ✅ | ⚠️ Cơ bản | ⚠️ | ✅ |
| **Audit log on-prem** | ✅ | ✅ | ⚠️ | ❌ | ❌ (cloud) |
| **Response shaping** | ✅ Mạnh | ❌ | ❌ | ❌ | ❌ |
| **Credentials ở đâu** | Server bạn (AES-256) | Server bạn | Server bạn | Server bạn | Cloud Composio |
| **SSO/SCIM** | ✅ | ✅ | ❌ | ❌ | ✅ |
| **License** | BSL-1.1 → Apache 2030 | Apache 2.0 | Apache 2.0 | MIT | Proprietary |
| **Điểm mạnh nhất** | Legacy/SOAP/ERP | Linh hoạt cho dev | Gọn nhẹ Docker | Gom nhiều MCP server | SaaS integrations nhanh |

### 22.2. Khi Nào Chọn Cái Nào?

| Tình huống | Nên chọn |
|------------|----------|
| Công ty có ERP/CRM legacy, SOAP, database on-prem | **AnythingMCP** — sinh ra cho việc này |
| Team dev muốn tự code gateway linh hoạt | **ContextForge** |
| Chỉ cần gom nhiều MCP servers lại 1 chỗ | **MetaMCP** hoặc **Docker MCP Gateway** |
| Cần nối nhanh SaaS phổ biến (Gmail, Slack, Notion...) không cần tự host | **Composio** |
| Muốn tất cả: no-code + legacy + governance | **AnythingMCP** |

### 22.3. AnythingMCP vs Tự Viết MCP Server

Đã so sánh ở mục 2.3 — tóm lại: AnythingMCP tiết kiệm **hàng tuần** công sức cho mỗi hệ thống, đổi lại bạn ít tùy chỉnh sâu hơn code tay. Với 90% nhu cầu doanh nghiệp (expose API/DB cho AI dùng), AnythingMCP là đủ.

---

## 23. Troubleshooting

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `./setup.sh` báo thiếu openssl | Chưa cài openssl | `sudo apt install openssl` (Ubuntu) |
| Không mở được localhost:3000 | Container chưa chạy / port bị chiếm | `docker compose ps`, `docker compose logs` |
| Connector test connection fail | Sai URL/credentials, firewall chặn | Test bằng `curl` từ server trước |
| WSDL parse ra 0 operations | WSDL sai format, cần auth để tải | Tải WSDL về file, import từ file |
| AI gọi tool bị 401 | Token hết hạn, sai credentials | Kiểm tra auth config, test lại connection |
| OAuth2 không refresh token | Sai token URL / client secret | Xem log connector, kiểm tra response từ IdP |
| Response mapping làm mất hết data | JSONPath sai | Dùng chế độ test before/after trong editor để debug |
| Claude không thấy tools | Sai URL/key, server chưa add connector | Kiểm tra API key còn hiệu lực, server có tools không |
| Chậm khi AI gọi tool | API upstream chậm, chunk lớn | Kiểm tra duration trong audit log để xác định bottleneck |
| Mất credentials sau restore | Sai ENCRYPTION_KEY | Đảm bảo key trong .env khớp với lúc backup |
| Database connector bị từ chối | User DB không có quyền SELECT | Cấp `GRANT SELECT ON ...` cho user |

### Xem Log Chi Tiết

```bash
# Log tất cả services
cd anythingmcp && docker compose logs -f

# Log riêng app
docker compose logs -f anythingmcp

# Log PostgreSQL
docker compose logs -f postgres
```

---

## 24. Best Practices

### 24.1. Khởi Đầu An Toàn

1. ✅ Bắt đầu với **read-only**: chỉ expose GET/SELECT trước
2. ✅ Dùng **response shaping** loại bỏ field nhạy cảm ngay từ đầu
3. ✅ Đặt `fallbackToRaw: false` cho tool nhạy cảm
4. ✅ Tạo role riêng cho từng nhóm user, đừng cho tất cả làm admin
5. ✅ Mint API key riêng cho từng người/máy

### 24.2. Vận Hành

- ✅ Backup DB + giữ `ENCRYPTION_KEY` riêng, test restore định kỳ
- ✅ Theo dõi audit log hàng tuần: tool nào lỗi nhiều? ai gọi bất thường?
- ✅ Update version mới sau khi đọc changelog (đừng update production ngay ngày release)
- ✅ Đặt retention cho audit log (tránh DB phình)
- ❌ Đừng expose AnythingMCP UI ra internet không có firewall/auth

### 24.3. Hiệu Năng

- ✅ Response shaping giảm 90%+ dung lượng → AI nhanh hơn, rẻ hơn
- ✅ Giới hạn số tools mỗi MCP server (< 30 tools là đẹp — quá nhiều AI "rối")
- ✅ Database: tạo index cho cột hay query, giới hạn LIMIT mặc định
- ✅ Dùng connector có sẵn trong catalog thay vì tự cấu hình khi có thể

### 24.4. Bảo Mật Checklist

- [ ] Đổi mật khẩu admin, bật SSO nếu có
- [ ] Credentials mã hóa AES-256 (mặc định đã bật)
- [ ] RBAC: mỗi role chỉ thấy tools cần thiết
- [ ] API key riêng từng user, revoke khi nghỉ việc
- [ ] HTTPS cho production (Nginx + Let's Encrypt)
- [ ] Audit log bật và xem định kỳ
- [ ] `ENCRYPTION_KEY` backup riêng khỏi database

---

## 25. FAQ

**Hỏi: AnythingMCP có miễn phí không?**
Đáp: Có — miễn phí cho dùng cá nhân, nội bộ công ty, dev, học thuật. Chỉ cần license thương mại nếu bạn bán nó dưới dạng dịch vụ SaaS cho người khác. Đến 2030 tự thành Apache 2.0.

**Hỏi: Khác gì với AnythingLLM?**
Đáp: Hai project khác nhau hoàn toàn! AnythingLLM = chatbot chat với tài liệu (RAG). AnythingMCP = gateway biến API thành MCP tools cho AI gọi. Tên giống nhau vì cùng họ "Anything", nhưng mục đích khác nhau.

**Hỏi: Có cần biết code không?**
Đáp: Không. Toàn bộ cấu hình qua UI: import spec, chọn tools, cấu hình auth.

**Hỏi: Dữ liệu có rời khỏi server tôi không?**
Đáp: Không — mọi thứ chạy trên hạ tầng của bạn. Chỉ có nội dung AI cần (đã qua response shaping) mới được gửi đến model (ví dụ OpenAI/Anthropic) khi AI gọi tool.

**Hỏi: Hỗ trợ tiếng Việt không?**
Đáp: UI chủ yếu tiếng Anh. Nhưng AI (Claude/ChatGPT) trả lời tiếng Việt bình thường vì tools chỉ là "tay chân", ngôn ngữ do model quyết định.

**Hỏi: Một connector dùng cho nhiều AI client được không?**
Đáp: Được — đó chính là điểm mạnh. Cấu hình 1 lần, Claude + ChatGPT + Cursor + Copilot dùng chung.

**Hỏi: Có chạy được trên Raspberry Pi / NAS không?**
Đáp: Được nếu đủ RAM (2GB+) và chạy được Docker. Dùng cho homelab rất hợp.

**Hỏi: Sao không dùng Composio cho nhanh?**
Đáp: Composio là SaaS — credentials của bạn nằm trên cloud họ. AnythingMCP tự host — credentials ở server bạn, phù hợp khi cần kiểm soát dữ liệu (ngân hàng, y tế, chính phủ...).

---

## Phụ Lục: Cheat Sheet

```bash
# Cài đặt
git clone https://github.com/HelpCode-ai/anythingmcp.git
cd anythingmcp && ./setup.sh
# Mở http://localhost:3000, đăng ký admin ngay!

# Xem log
docker compose logs -f

# Update
git pull && docker compose pull && docker compose up -d

# Backup DB
docker exec anythingmcp-postgres-1 pg_dump -U postgres anythingmcp > backup-$(date +%Y%m%d).sql

# Restore
gunzip -c backup-20261010.sql.gz | docker exec -i anythingmcp-postgres-1 psql -U postgres anythingmcp
```

| Thao tác | Vị trí trong UI |
|----------|----------------|
| Tạo connector | Connectors → + New |
| Import OpenAPI | Connector REST → Import OpenAPI |
| Cài connector có sẵn | Catalog → chọn → điền credentials |
| Tạo MCP server | MCP Servers → + New → chọn connectors |
| Mint API key | MCP server → API Keys → Mint new key |
| Phân quyền | Users → Roles → chọn tools |
| Xem audit log | Sidebar → Audit Log |
| Response shaping | Tool editor → Response mapping |
| Backup | (làm ngoài bằng pg_dump) |

**Links:**
- Repo: https://github.com/HelpCode-ai/anythingmcp
- Docs: https://anythingmcp.com
- Cloud: https://cloud.anythingmcp.com

---
