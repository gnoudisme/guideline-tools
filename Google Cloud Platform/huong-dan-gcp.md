# Hướng Dẫn Google Cloud Platform (GCP) — Từ A Đến Z

> Cẩm nang tiếng Việt dành cho người mới bắt đầu: tạo tài khoản, dùng Compute Engine, Cloud Storage, BigQuery, Cloud Run, IAM, gcloud CLI… đủ để tự tay deploy một ứng dụng thật lên cloud.
>
> *Viết bởi Capybara 🦫 cho Tiểu Dươngg — 2026-10-10*

---

## Mục Lục

1. [GCP là gì? So sánh với AWS / Azure](#1-gcp-là-gì-so-sánh-với-aws--azure)
2. [Tạo tài khoản GCP](#2-tạo-tài-khoản-gcp)
3. [Tạo và tổ chức Project](#3-tạo-và-tổ-chức-project)
4. [Thiết lập Billing: tài khoản thanh toán, budget alert, hóa đơn](#4-thiết-lập-billing)
5. [Tham quan Cloud Console](#5-tham-quan-cloud-console)
6. [Compute Engine — máy chủ ảo (VM)](#6-compute-engine--máy-chủ-ảo-vm)
7. [Cloud Storage — lưu trữ file/object](#7-cloud-storage--lưu-trữ-fileobject)
8. [Cloud Functions — serverless theo sự kiện](#8-cloud-functions--serverless-theo-sự-kiện)
9. [Cloud Run — container serverless](#9-cloud-run--container-serverless)
10. [BigQuery — kho dữ liệu phân tích SQL](#10-bigquery--kho-dữ-liệu-phân-tích-sql)
11. [IAM & Phân quyền](#11-iam--phân-quyền)
12. [gcloud CLI — điều khiển GCP từ terminal](#12-gcloud-cli--điều-khiển-gcp-từ-terminal)
13. [Cách tính tiền (Pricing)](#13-cách-tính-tiền-pricing)
14. [Free Tier: $300 credit và Always Free](#14-free-tier-300-credit-và-always-free)
15. [Xử lý lỗi thường gặp](#15-xử-lý-lỗi-thường-gặp)
16. [Best practices & Cheat sheet gcloud](#16-best-practices--cheat-sheet-gcloud)

---

## 1. GCP là gì? So sánh với AWS / Azure

**Google Cloud Platform (GCP)** là nền tảng điện toán đám mây của Google: thuê máy chủ ảo, lưu trữ file, cơ sở dữ liệu, AI/ML… qua internet, trả tiền theo mức dùng. Bạn không cần mua máy vật lý, lắp đặt tủ rack — chỉ cần vài cú click là có hạ tầng chạy được ở hàng chục trung tâm dữ liệu (region) trên toàn cầu.

### So sánh nhanh "big three"

| Tiêu chí | GCP (Google) | AWS (Amazon) | Azure (Microsoft) |
|---|---|---|---|
| Quy mô | Top 3 thế giới | Lớn nhất | Top 2 |
| Điểm mạnh | AI/ML (Vertex AI, Gemini), BigQuery, mạng nhanh, tính tiền theo giây | Hệ sinh thái rộng nhất, nhiều dịch vụ nhất | Tích hợp tốt với Windows/.NET/Office |
| Free trial | $300 credit / 90 ngày + Always Free | 12 tháng Free Tier | $200 credit / 30 ngày |
| Đơn giá VM thường | Rẻ hơn ~5–10% cho workload ổn định | Cao hơn chút | Tương đương AWS |
| Mạng backbone | Dùng chính mạng Google (cực nhanh, global) | Mạng riêng mạnh | Mạng riêng mạnh |
| Học cho người mới | Console gọn, tài liệu dễ đọc | Nhiều lựa chọn dễ "ngợp" | Hơi rối vì nhiều tên gọi |

**Tóm lại:** nếu bạn thích AI, phân tích dữ liệu (BigQuery), hoặc muốn bắt đầu với cloud một cách gọn nhẹ — GCP là lựa chọn rất hợp lý.

### Các dịch vụ chính của GCP

**Tính toán (Compute)**
- **Compute Engine** — máy chủ ảo (VM) Linux/Windows, bạn tự quản lý hoàn toàn
- **Google Kubernetes Engine (GKE)** — chạy container trên Kubernetes được quản lý
- **Cloud Run** — chạy container serverless, không cần lo server
- **Cloud Functions** — chạy đoạn code theo sự kiện, serverless hoàn toàn
- **App Engine** — PaaS "cũ nhưng bền", deploy code là chạy

**Lưu trữ (Storage)**
- **Cloud Storage** — lưu file/object (ảnh, video, backup…), rẻ và bền
- **Persistent Disk** — ổ đĩa gắn vào VM
- **Filestore** — file share NFS

**Dữ liệu (Data & Database)**
- **BigQuery** — data warehouse phân tích SQL, truy vấn petabyte trong vài giây
- **Cloud SQL** — MySQL / PostgreSQL / SQL Server được quản lý
- **Firestore** — NoSQL document, realtime, hợp cho app mobile/web
- **Cloud Spanner** — SQL phân tán toàn cầu, mạnh nhưng đắt

**AI/ML**
- **Vertex AI** — nền tảng ML end-to-end: train, deploy model
- **Gemini API** — gọi model Gemini của Google qua API
- **Cloud Vision / Speech / Translate** — API có sẵn cho từng tác vụ

**Mạng & khác**
- **VPC** — mạng riêng ảo, firewall, subnet
- **Cloud CDN, Cloud DNS, Load Balancing**
- **Cloud Build** — CI/CD build container tự động
- **Secret Manager** — lưu key, mật khẩu an toàn

> Trong bài này ta đi sâu vào các dịch vụ phổ biến nhất cho người mới: Compute Engine, Cloud Storage, Cloud Functions, Cloud Run, BigQuery, IAM và gcloud CLI.

---

## 2. Tạo tài khoản GCP

### Bước 1: Chuẩn bị

- Một **tài khoản Google** (Gmail) — nên dùng email riêng cho cloud, không nên dùng email cá nhân chính nếu có thể.
- Một **thẻ thanh toán quốc tế** (Visa/Mastercard debit hoặc credit). Google yêu cầu thẻ để xác minh danh tính, chống gian lận.

### Bước 2: Đăng ký

1. Truy cập **https://cloud.google.com** → bấm **"Get started for free"** (Bắt đầu miễn phí).
2. Đăng nhập tài khoản Google của bạn.
3. Chọn quốc gia (Việt Nam), đọc và đồng ý điều khoản.
4. Nhập thông tin thẻ:
   - Google sẽ **tạm giữ 1 USD (hoặc tương đương)** để xác minh thẻ — khoản này được hoàn lại, không phải phí.
   - Thẻ debit của các ngân hàng Việt Nam (Vietcombank, Techcombank, VIB…) thường dùng được nếu đã bật thanh toán quốc tế.
5. Bấm **"Start free"** — xong! Bạn nhận ngay **$300 credit dùng trong 90 ngày**.

### Bước 3: Xác minh tài khoản

- Google có thể gửi email xác minh hoặc yêu cầu xác minh số điện thoại qua SMS.
- Kiểm tra mục **Billing** trong Console để chắc chắn credit $300 đã về (thường hiện ngay sau khi đăng ký).

### Lưu ý quan trọng về thẻ tín dụng

- **Trong thời gian trial, Google KHÔNG tự động trừ tiền** khi hết $300 hoặc hết 90 ngày — tài khoản trial chỉ đơn giản là hết credit và bị "đóng băng" dịch vụ trả phí. Bạn phải **chủ động nâng cấp** (upgrade) sang tài khoản trả phí thì mới bị tính tiền.
- Nếu thẻ bị từ chối: kiểm tra xem thẻ đã bật **giao dịch quốc tế / online** chưa; một số thẻ ảo (virtual card) hoặc thẻ nội địa (không có Visa/Mastercard) sẽ không được chấp nhận.
- Giữ thông tin thẻ an toàn: nếu lộ key/service account của GCP mà không giới hạn quyền, kẻ xấu có thể "đào coin" bằng tài khoản của bạn → xem [mục 11 (IAM)](#11-iam--phân-quyền) để bảo vệ tài khoản.

---

## 3. Tạo và tổ chức Project

Mọi thứ trên GCP đều sống trong một **project**. Project là "hộp" chứa VM, bucket, database… và cũng là đơn vị tính tiền, phân quyền.

- **Project name** — tên hiển thị, ví dụ `Web Bán Hàng`. Có thể trùng, có thể đổi.
- **Project ID** — định danh duy nhất toàn cầu, ví dụ `web-ban-hang-412301`. **Không đổi được sau khi tạo**, dùng trong lệnh gcloud, URL, API.
- **Project number** — số tự sinh, dùng nội bộ.

### Cách tạo project

**Trên Console:**
1. Bấm vào dropdown chọn project ở thanh trên cùng → **New Project**.
2. Nhập **Project name** → Google gợi ý **Project ID** (bạn có thể bấm Edit để tự đặt, chỉ gồm chữ thường, số, dấu gạch ngang).
3. Chọn **Organization** (nếu có) hoặc để "No organization".
4. Bấm **Create**.

**Bằng gcloud:**

```bash
gcloud projects create web-ban-hang-412301 \
  --name="Web Bán Hàng"
```

Kiểm tra danh sách project:

```bash
$ gcloud projects list
PROJECT_ID            NAME          PROJECT_NUMBER
web-ban-hang-412301   Web Bán Hàng  987654321012
my-first-project      My Project    123456789098
```

Đặt project mặc định cho các lệnh sau:

```bash
gcloud config set project web-ban-hang-412301
```

### Organization, Folder và cách tổ chức nhiều project

Khi dùng GCP cho công ty/team, cấu trúc phân cấp là:

```
Organization (công ty)
├── Folder "production"
│   ├── project web-prod
│   └── project api-prod
├── Folder "staging"
│   └── project web-staging
└── Folder "dev"
    ├── project web-dev-duong
    └── project experiment-ml
```

- **Organization**: nút gốc, thường gắn với domain công ty (xác minh qua Google Workspace).
- **Folder**: nhóm các project theo môi trường (prod/staging/dev) hoặc theo team — để gán chính sách, budget chung.
- Người dùng cá nhân thường **không cần Organization**, chỉ cần đặt tên project rõ ràng:

| Quy ước đặt tên | Ví dụ |
|---|---|
| `<dự-án>-<môi-trường>` | `shop-prod`, `shop-staging`, `shop-dev` |
| `<team>-<mục-đích>` | `data-analytics`, `ml-experiment` |

> **Best practice:** tách project theo môi trường (dev/staging/prod). Lỡ tay xóa nhầm ở dev thì prod vẫn an toàn, và hóa đơn tách bạch dễ kiểm soát.

---

## 4. Thiết lập Billing

### Liên kết Billing Account với project

Mỗi project muốn dùng dịch vụ trả phí phải gắn vào một **Billing Account** (tài khoản thanh toán).

1. Vào **Billing** trong Console → nếu chưa có, tạo billing account và nhập thẻ.
2. Vào **Billing → My projects** → bấm **Change billing** ở project cần gắn → chọn billing account → **Set account**.
3. Kiểm tra: project nào chưa gắn billing sẽ báo lỗi `Billing not enabled` khi tạo tài nguyên trả phí (cách xử lý xem [mục 15](#15-xử-lý-lỗi-thường-gặp)).

Bằng gcloud:

```bash
# Liệt kê billing account
gcloud billing accounts list

# Gắn billing vào project
gcloud billing projects link web-ban-hang-412301 \
  --billing-account=0X0X0X-0X0X0X-0X0X0X
```

### Cảnh báo ngân sách (Budget Alerts)

**Việc đầu tiên sau khi gắn billing: tạo budget alert.** Đây là "phao cứu sinh" chống hóa đơn bất ngờ.

1. Vào **Billing → Budgets & alerts → Create budget**.
2. Chọn project (hoặc cả billing account).
3. Đặt **số tiền**, ví dụ **$10/tháng**.
4. Đặt **ngưỡng cảnh báo**: 50%, 90%, 100% (mặc định đã có 50/90/100).
5. Nhập email nhận cảnh báo → **Finish**.

Bạn sẽ nhận email kiểu: *"Budget 'web-prod' đã đạt 90% ($9.00 / $10.00)"* — đủ sớm để vào kiểm tra và tắt tài nguyên thừa.

> Lưu ý: budget alert **chỉ cảnh báo, không tự tắt** tài nguyên. Muốn tự động tắt khi vượt ngưỡng, phải viết thêm Cloud Function (nâng cao, ngoài phạm vi bài này).

Bằng gcloud (cần Billing Budgets API bật):

```bash
gcloud billing budgets create \
  --billing-account=0X0X0X-0X0X0X-0X0X0X \
  --display-name="web-prod-monthly" \
  --budget-amount=10 \
  --threshold-rule=percent=50 \
  --threshold-rule=percent=90 \
  --threshold-rule=percent=100
```

### Xem hóa đơn và chi phí

- **Billing → Overview**: tổng quan chi phí tháng này, biểu đồ theo dịch vụ.
- **Billing → Reports**: lọc chi phí theo project, dịch vụ, SKU, khoảng thời gian — cực hữu ích khi hóa đơn tăng đột biến.
- **Billing → Transactions / Documents**: xem và tải hóa đơn (invoice) PDF.
- Mẹo: trong Reports, nhóm theo **SKU** để biết chính xác dòng tiền nào đang "đốt" (ví dụ `E2 Instance Core` vs `Network Egress`).

---

## 5. Tham quan Cloud Console

**Cloud Console** là giao diện web tại **https://console.cloud.google.com** — nơi bạn click-chuột làm mọi thứ.

### Dashboard

Trang chủ sau khi đăng nhập:
- **Project info**: tên, ID, số project đang dùng.
- **Resources**: tóm tắt số VM, bucket, database đang chạy.
- **Billing**: chi phí ước tính tháng này.
- **Thẻ "Try"**: gợi ý các dịch vụ nên thử.

### Navigation menu (☰)

Bấm icon **☰** góc trái trên để mở menu đầy đủ, nhóm theo:
- **Compute**: Compute Engine, GKE, Cloud Run, Cloud Functions, App Engine
- **Storage**: Cloud Storage, Filestore
- **Databases**: Cloud SQL, Firestore, Spanner, BigQuery (BigQuery cũng nằm nhóm Analytics)
- **Analytics**: BigQuery, Dataflow, Pub/Sub
- **AI**: Vertex AI
- **Networking**: VPC, Load balancing, Cloud DNS
- **Security**: IAM, Secret Manager
- **Operations**: Logging, Monitoring

Mẹo: **ghim (pin)** dịch vụ hay dùng lên đầu menu để truy cập nhanh.

### Search bar (thanh tìm kiếm)

Thanh search ở giữa phía trên là "siêu năng lực":
- Gõ `vm` → ra ngay **Compute Engine / VM instances**.
- Gõ `bucket` → ra **Cloud Storage / Buckets**.
- Gõ tên tài nguyên cụ thể (ví dụ `web-server-1`) → nhảy thẳng tới nó.
- Hỗ trợ cả lệnh nhanh: gõ `?` để xem gợi ý.

### Cloud Shell — terminal miễn phí ngay trên trình duyệt

Bấm icon **`>_`** góc phải trên để mở **Cloud Shell**: một máy ảo Debian tạm thời (5GB home persistent), đã cài sẵn `gcloud`, `gsutil`, `bq`, `kubectl`, `docker`, `git`, `python`…

```bash
# Trong Cloud Shell, bạn đã đăng nhập sẵn — thử ngay:
$ gcloud auth list
   Credentialed Accounts
ACTIVE  ACCOUNT
*       duongg@gmail.com

$ gcloud config list
[core]
project = web-ban-hang-412301
```

- Cloud Shell **miễn phí**, nhưng session sẽ timeout sau ~1 giờ không hoạt động.
- Dùng editor tích hợp (bấm **Open Editor**) để sửa file như VS Code mini.
- Upload/download file qua menu **⋮** của Cloud Shell.

---

## 6. Compute Engine — máy chủ ảo (VM)

Compute Engine cho bạn thuê **máy chủ ảo (VM instance)**: chọn CPU/RAM, hệ điều hành, vùng đặt máy — vài phút sau là có server để SSH vào cài đặt.

### Tạo VM từng bước trên Console

1. Vào **Compute Engine → VM instances → Create instance**.
2. **Name**: `web-server-1` (chữ thường, số, gạch ngang).
3. **Region / Zone**: chọn region gần người dùng. Ở Việt Nam thường chọn **`asia-southeast1` (Singapore)** — ping thấp nhất. Zone ví dụ `asia-southeast1-a`.
4. **Machine configuration**:
   - Series: **E2** (rẻ, đa dụng) cho web/app thông thường.
   - Machine type: **`e2-micro`** (2 vCPU, 1GB RAM — nằm trong Always Free!) hoặc `e2-small`, `e2-medium` tùy nhu cầu.
5. **OS and storage / Boot disk**: bấm **Change** → chọn OS:
   - **Ubuntu 24.04 LTS** (khuyên dùng cho người mới), hoặc Debian, Rocky Linux, Windows Server…
   - Boot disk type: **Balanced persistent disk**, size 10–30GB (mặc định 10GB là đủ thử).
6. **Firewall**: tick **Allow HTTP traffic** (mở port 80) và **Allow HTTPS traffic** (443) nếu chạy web.
7. Bấm **Create** — khoảng 1–2 phút sau VM chuyển trạng thái **Running** là xong.

### Các khái niệm cần nhớ

| Khái niệm | Giải thích |
|---|---|
| Region | Vùng địa lý gồm nhiều zone, ví dụ `asia-southeast1` (Singapore) |
| Zone | Trung tâm dữ liệu cụ thể, ví dụ `asia-southeast1-a` |
| Machine type | Cấu hình CPU/RAM: `e2-micro`, `e2-medium`, `n2-standard-4`… |
| Preemptible / Spot VM | VM giá rẻ (giảm tới 60–91%) nhưng Google có thể thu hồi bất cứ lúc nào khi cần tài nguyên — hợp cho batch job, CI, task chịu gián đoạn |
| Snapshot | Ảnh chụp ổ đĩa tại một thời điểm — để backup hoặc clone VM |
| Static IP | IP tĩnh giữ nguyên khi restart (mặc định IP là ephemeral — đổi khi stop/start) |

### Tạo VM bằng gcloud

```bash
gcloud compute instances create web-server-1 \
  --zone=asia-southeast1-a \
  --machine-type=e2-micro \
  --image-family=ubuntu-2404-lts-amd64 \
  --image-project=ubuntu-os-cloud \
  --boot-disk-size=20GB \
  --tags=http-server,https-server
```

Liệt kê VM:

```bash
$ gcloud compute instances list
NAME          ZONE               MACHINE_TYPE  INTERNAL_IP  EXTERNAL_IP     STATUS
web-server-1  asia-southeast1-a  e2-micro      10.148.0.2   34.87.123.45    RUNNING
```

### SSH vào VM

**Cách 1 — bấm nút SSH trên Console:** ở cột Connect bấm **SSH** → mở cửa sổ terminal ngay trên trình duyệt. Google tự lo key.

**Cách 2 — từ máy của bạn:**

```bash
gcloud compute ssh web-server-1 --zone=asia-southeast1-a
```

Lần đầu gcloud sẽ tự tạo SSH key và thêm vào project. Vào được rồi thì cài đặt như server Linux bình thường:

```bash
duongg@web-server-1:~$ sudo apt update && sudo apt install -y nginx
duongg@web-server-1:~$ curl localhost
# ... trang chào mừng của nginx
```

### Start / Stop / Delete

```bash
# Dừng VM (không tính tiền CPU/RAM nữa, vẫn tính tiền ổ đĩa)
gcloud compute instances stop web-server-1 --zone=asia-southeast1-a

# Bật lại
gcloud compute instances start web-server-1 --zone=asia-southeast1-a

# Xóa hẳn (kèm ổ đĩa nếu chọn)
gcloud compute instances delete web-server-1 --zone=asia-southeast1-a
```

> **Tiết kiệm tiền:** VM tắt (STOPPED) thì không mất tiền CPU/RAM nhưng **vẫn mất tiền Persistent Disk**. Không dùng nữa thì **delete** luôn.

### Snapshot — backup ổ đĩa

```bash
# Tạo snapshot từ disk của VM (không cần tắt VM)
gcloud compute disks snapshot web-server-1 \
  --zone=asia-southeast1-a \
  --snapshot-names=web-server-1-backup-20261010

# Liệt kê snapshot
gcloud compute snapshots list
```

Trên Console: **Compute Engine → Snapshots → Create snapshot** → chọn source disk. Snapshot lưu incremental (chỉ phần thay đổi) nên rẻ hơn bạn tưởng.

### Spot VM (Preemptible) — rẻ tới 91%

```bash
gcloud compute instances create batch-worker-1 \
  --zone=asia-southeast1-a \
  --machine-type=e2-medium \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

- `--provisioning-model=SPOT`: xin VM spot giá rẻ.
- Google có thể thu hồi trong **30 giây** báo trước — code của bạn nên checkpoint thường xuyên hoặc thiết kế chịu được restart.

---

## 7. Cloud Storage — lưu trữ file/object

**Cloud Storage** là nơi cất file (ảnh, video, backup, file tĩnh của web…): bền vững 99.999999999% (11 số 9), truy cập qua URL/API, giá rẻ.

### Bucket là gì?

- **Bucket** = "thư mục gốc" chứa các **object** (file). Tên bucket phải **duy nhất toàn cầu** (ví dụ `shop-duongg-avatars`).
- Object không có thư mục thật — `images/2026/photo.jpg` chỉ là tên object có dấu `/`, Console hiển thị như folder cho dễ nhìn.

### Tạo bucket

**Console:** Storage → Buckets → Create → đặt tên → chọn region (ví dụ `asia-southeast1`) → chọn storage class → chọn kiểm soát truy cập → Create.

**gsutil / gcloud:**

```bash
# Tạo bucket ở Singapore, storage class STANDARD
gcloud storage buckets create gs://shop-duongg-avatars \
  --location=asia-southeast1 \
  --default-storage-class=STANDARD
```

### Storage class — chọn theo tần suất truy cập

| Class | Khi nào dùng | Giá lưu trữ | Ví dụ |
|---|---|---|---|
| **Standard** | Truy cập thường xuyên | Cao nhất | Ảnh avatar, file web |
| **Nearline** | Truy cập ~1 lần/tháng | Rẻ hơn | Backup tháng |
| **Coldline** | Truy cập ~1 lần/quý | Rẻ nữa | Backup quý |
| **Archive** | Hiếm khi đụng tới | Rẻ nhất | Lưu trữ pháp lý, 7 năm |

Quy tắc: class càng "lạnh" thì **tiền lưu trữ càng rẻ nhưng tiền đọc ra (retrieval) càng đắt**. Đừng để file web truy cập liên tục vào Archive — tiền retrieval sẽ "cắn" lại.

Đổi class cho object có sẵn:

```bash
gcloud storage objects update gs://shop-duongg-avatars/backup.zip \
  --storage-class=NEARLINE
```

### Upload / download file

**Trên Console:** mở bucket → **Upload files** / **Upload folder**; bấm vào object → **Download**.

**Bằng gcloud storage (khuyên dùng, thay cho gsutil cũ):**

```bash
# Upload 1 file
gcloud storage cp ./logo.png gs://shop-duongg-avatars/

# Upload cả thư mục (đệ quy)
gcloud storage cp -r ./images/ gs://shop-duongg-avatars/images/

# Download về máy
gcloud storage cp gs://shop-duongg-avatars/logo.png ./logo-download.png

# Liệt kê object
$ gcloud storage ls gs://shop-duongg-avatars/
gs://shop-duongg-avatars/logo.png
gs://shop-duongg-avatars/images/
```

> `gsutil` vẫn dùng được nhưng Google đang chuyển sang `gcloud storage` — cú pháp gần như giống hệt.

### Phân quyền public / private

Mặc định object là **private** (chỉ người có quyền mới đọc được).

**Cho phép public đọc 1 file** (ví dụ ảnh hiển thị trên web):

```bash
gcloud storage objects update gs://shop-duongg-avatars/logo.png \
  --add-acl-grant=entity=AllUsers,role=READER
```

**Cho public cả bucket** (cẩn thận!): vào Console → bucket → **Permissions** → Add principal `allUsers` với role **Storage Object Viewer**. Hoặc dùng Uniform bucket-level access + IAM.

**Thu hồi public:**

```bash
gcloud storage objects update gs://shop-duongg-avatars/logo.png \
  --remove-acl-grant=entity=AllUsers
```

> Nguyên tắc: mặc định private, chỉ public những gì thật sự cần. Bucket chứa dữ liệu nhạy cảm mà public là "tai nạn" kinh điển trên cloud.

### Lifecycle policy — tự động dọn dẹp / chuyển class

Ví dụ: file log sau 30 ngày chuyển sang Nearline, sau 90 ngày chuyển Coldline, sau 365 ngày xóa hẳn. Tạo file `lifecycle.json`:

```json
{
  "rule": [
    {
      "action": { "type": "SetStorageClass", "storageClass": "NEARLINE" },
      "condition": { "age": 30 }
    },
    {
      "action": { "type": "SetStorageClass", "storageClass": "COLDLINE" },
      "condition": { "age": 90 }
    },
    {
      "action": { "type": "Delete" },
      "condition": { "age": 365 }
    }
  ]
}
```

Áp dụng:

```bash
gcloud storage buckets update gs://shop-duongg-avatars \
  --lifecycle-file=lifecycle.json
```

---

## 8. Cloud Functions — serverless theo sự kiện

**Serverless** = bạn chỉ viết code, Google lo toàn bộ server: không provision VM, tự scale theo tải, trả tiền theo số lần chạy + thời gian chạy. Không ai gọi thì không tốn tiền (gần như).

**Cloud Functions** chạy **một hàm** để đáp ứng **một sự kiện**: HTTP request, file mới upload lên bucket, message mới trên Pub/Sub…

### Tạo function Node.js (trigger HTTP) trên Console

1. Vào **Cloud Functions → Create function**.
2. **Environment**: chọn **2nd gen** (mới, chạy trên Cloud Run).
3. **Function name**: `hello-duongg`; **Region**: `asia-southeast1`.
4. **Trigger**: HTTPS → chọn **Allow unauthenticated invocations** (để ai cũng gọi được — demo thôi nhé).
5. **Runtime**: Node.js 20; **Entry point**: `helloWorld`.
6. Sửa code mẫu trong editor:

```javascript
const functions = require('@google-cloud/functions-framework');

functions.http('helloWorld', (req, res) => {
  const name = req.query.name || 'Dươngg';
  res.send(`Chào ${name}, đây là Cloud Functions! 🦫`);
});
```

7. Bấm **Deploy** — vài phút sau bạn nhận được **URL** kiểu `https://asia-southeast1-web-ban-hang-412301.cloudfunctions.net/hello-duongg`.

Thử gọi:

```bash
$ curl "https://asia-southeast1-web-ban-hang-412301.cloudfunctions.net/hello-duongg?name=Tiểu%20Dươngg"
Chào Tiểu Dươngg, đây là Cloud Functions! 🦫
```

### Tạo function Python bằng gcloud

Chuẩn bị thư mục `my-func/`:

`main.py`:
```python
import functions_framework

@functions_framework.http
def hello_world(request):
    name = request.args.get("name", "Dươngg")
    return f"Chào {name}, đây là Cloud Functions Python! 🦫"
```

`requirements.txt`:
```
functions-framework==3.*
```

Deploy:

```bash
cd my-func
gcloud functions deploy hello-python \
  --gen2 \
  --runtime=python312 \
  --region=asia-southeast1 \
  --source=. \
  --entry-point=hello_world \
  --trigger-http \
  --allow-unauthenticated
```

Kết quả trả về có `url:` — dùng ngay:

```bash
$ curl "https://asia-southeast1-web-ban-hang-412301.cloudfunctions.net/hello-python?name=Capybara"
Chào Capybara, đây là Cloud Functions Python! 🦫
```

### Xem log

```bash
# Log của function vừa deploy (Gen 2 đọc từ Cloud Logging)
gcloud functions logs read hello-python \
  --gen2 \
  --region=asia-southeast1 \
  --limit=20
```

Kết quả mẫu:

```
2026-10-10T02:10:31.123Z  hello-python  Function execution started
2026-10-10T02:10:31.456Z  hello-python  Function execution took 12 ms, finished with status code: 200
```

Hoặc xem trực quan: **Cloud Functions → bấm vào function → tab Logs**.

---

## 9. Cloud Run — container serverless

**Cloud Run** cũng serverless, nhưng đơn vị deploy là **container** (Docker image) thay vì một hàm. Bạn đóng gói app (web API, backend…) thành image, Cloud Run chạy và tự scale — kể cả **scale về 0** khi không có traffic (không tốn tiền lúc idle).

### So sánh Cloud Run vs Cloud Functions

| Tiêu chí | Cloud Functions | Cloud Run |
|---|---|---|
| Đơn vị deploy | 1 function | 1 container (bất kỳ ngôn ngữ/framework nào) |
| Dùng khi | Xử lý sự kiện nhỏ, webhook, cron | Web app, API, microservice |
| Port | Không cần quan tâm | Container phải lắng nghe `$PORT` |
| Timeout tối đa | 60 phút (Gen 2) | 60 phút |
| Scale về 0 | Có | Có |
| Ngôn ngữ | Giới hạn runtime hỗ trợ | Bất kỳ thứ gì đóng được Docker |

Quy tắc chọn nhanh: **việc nhỏ theo sự kiện → Functions; app/web service → Cloud Run.**

### Ví dụ: deploy app Node.js đơn giản lên Cloud Run

`package.json`:
```json
{
  "name": "hello-run",
  "version": "1.0.0",
  "scripts": { "start": "node index.js" },
  "dependencies": { "express": "^4.19.2" }
}
```

`index.js`:
```javascript
const express = require('express');
const app = express();
const port = process.env.PORT || 8080;

app.get('/', (req, res) => {
  res.send('Chào Tiểu Dươngg từ Cloud Run! 🦫');
});

app.listen(port, () => console.log(`Listening on ${port}`));
```

`Dockerfile`:
```dockerfile
FROM node:20-slim
WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev
COPY . .
CMD ["npm", "start"]
```

**Cách 1 — deploy từ source (Google tự build, không cần viết Dockerfile):**

```bash
gcloud run deploy hello-run \
  --source=. \
  --region=asia-southeast1 \
  --allow-unauthenticated
```

**Cách 2 — deploy từ container image có sẵn:**

```bash
# Build image bằng Cloud Build rồi deploy
gcloud run deploy hello-run \
  --image=asia-southeast1-docker.pkg.dev/web-ban-hang-412301/my-repo/hello-run:latest \
  --region=asia-southeast1 \
  --allow-unauthenticated
```

Sau khi deploy xong, terminal in ra **Service URL**:

```
Service URL: https://hello-run-abc123xyz-as.a.run.app
```

```bash
$ curl https://hello-run-abc123xyz-as.a.run.app
Chào Tiểu Dươngg từ Cloud Run! 🦫
```

### Autoscaling

Cloud Run tự scale số container theo request. Bạn có thể giới hạn:

```bash
gcloud run services update hello-run \
  --region=asia-southeast1 \
  --min-instances=0 \
  --max-instances=5 \
  --concurrency=80
```

- `--min-instances=0`: không traffic → tắt hết → **không tốn tiền**.
- `--max-instances=5`: chặn scale quá đà → chặn hóa đơn quá đà.
- `--concurrency=80`: mỗi container xử lý tối đa 80 request đồng thời.

Xem trạng thái service:

```bash
$ gcloud run services list --region=asia-southeast1
SERVICE     REGION           URL                                          READY
hello-run   asia-southeast1  https://hello-run-abc123xyz-as.a.run.app     True
```

---

## 10. BigQuery — kho dữ liệu phân tích SQL

**Data warehouse** là "kho dữ liệu khổng lồ tối ưu cho phân tích": bạn đổ dữ liệu vào, rồi chạy **SQL** truy vấn hàng tỷ dòng trong vài giây — không cần lo server, index, shard. BigQuery tính tiền theo **lượng dữ liệu quét** (per query) hoặc theo slot.

### Tạo dataset và table

**Console:** BigQuery → chọn project → **Create dataset** → đặt `Dataset ID` = `shop_analytics`, location `asia-southeast1` → Create. Rồi trong dataset → **Create table**.

**Bằng bq CLI:**

```bash
# Tạo dataset
bq mk --location=asia-southeast1 shop_analytics

# Tạo table từ file CSV local (tự đoán schema)
bq load --autodetect --source_format=CSV \
  shop_analytics.orders ./orders.csv
```

Tạo table bằng SQL:

```sql
CREATE TABLE `web-ban-hang-412301.shop_analytics.orders` (
  order_id STRING,
  customer STRING,
  amount NUMERIC,
  created_at TIMESTAMP
);
```

### Chạy query SQL thật trên public dataset

Google cung cấp nhiều **public dataset miễn phí** để vọc. Ví dụ: dữ liệu COVID-19 của New York Times.

Mở BigQuery Studio, chạy:

```sql
-- Top 10 quốc gia có số ca COVID cao nhất (dữ liệu public của NYT)
SELECT
  country_region,
  SUM(confirmed) AS total_confirmed
FROM `bigquery-public-data.covid19_nyt.us_confirmed`
GROUP BY country_region
ORDER BY total_confirmed DESC
LIMIT 10;
```

Một ví dụ khác — phân tích tên trẻ em ở Mỹ:

```sql
-- 10 cái tên phổ biến nhất năm 2020
SELECT name, SUM(number) AS total
FROM `bigquery-public-data.usa_names.usa_1910_2013`
WHERE year = 2020
GROUP BY name
ORDER BY total DESC
LIMIT 10;
```

Kết quả mẫu:

```
+---------+--------+
|  name   | total  |
+---------+--------+
| Olivia  |  17535 |
| Emma    |  15581 |
| Ava     |  13084 |
...
```

Chạy bằng bq CLI:

```bash
bq query --use_legacy_sql=false \
'SELECT name, SUM(number) AS total
 FROM `bigquery-public-data.usa_names.usa_1910_2013`
 WHERE year = 2020
 GROUP BY name
 ORDER BY total DESC
 LIMIT 10'
```

> **Mẹo tiết kiệm tiền:** trước khi bấm Run, nhìn góc phải editor — BigQuery báo **"This query will process X GB"**. Dùng `LIMIT`, chọn đúng cột cần thiết, và tránh `SELECT *` trên table lớn.

### Lưu kết quả query

Sau khi query chạy xong → bấm **Save results** → chọn:
- **CSV / JSON local**: tải về máy.
- **Google Sheets**: mở thẳng trong Sheets.
- **BigQuery table**: lưu thành table mới để query tiếp (có tính phí lưu trữ).

```bash
# Lưu kết quả vào table mới
bq query --use_legacy_sql=false \
  --destination_table=shop_analytics.top_names_2020 \
'SELECT name, SUM(number) AS total
 FROM `bigquery-public-data.usa_names.usa_1910_2013`
 WHERE year = 2020
 GROUP BY name
 ORDER BY total DESC
 LIMIT 100'
```

### Scheduled query — chạy query theo lịch

1. Trong editor sau khi viết query xong → bấm **Schedule** → **Create new scheduled query**.
2. Đặt tên, chọn lịch (ví dụ **Every day at 07:00**).
3. Chọn **destination table** + chế độ ghi (overwrite / append).
4. **Save** — BigQuery sẽ tự chạy query mỗi ngày, kết quả nằm gọn trong table.

Ví dụ: mỗi sáng tổng hợp doanh thu hôm qua vào table `daily_revenue` — mở dashboard là có số liệu mới.

---

## 11. IAM & Phân quyền

**IAM (Identity and Access Management)** trả lời câu hỏi: **ai** được **làm gì** trên **tài nguyên nào**.

### 3 khái niệm cốt lõi

- **Principal** — "ai": tài khoản Google (user), nhóm (group), hoặc **service account**.
- **Role** — "được làm gì": tập hợp các quyền (permissions). Ví dụ role `roles/storage.objectViewer` chỉ cho đọc object trong bucket.
- **Policy** — "trên tài nguyên nào": gắn principal + role vào project/folder/bucket cụ thể.

### Các loại role

| Loại | Ví dụ | Khi dùng |
|---|---|---|
| **Basic** (nguyên thủy) | `Owner`, `Editor`, `Viewer` | Rộng, áp cả project — **hạn chế dùng** |
| **Predefined** (định sẵn) | `roles/compute.instanceAdmin`, `roles/bigquery.dataViewer` | Dùng cho nhu cầu cụ thể — **khuyên dùng** |
| **Custom** (tự tạo) | `projects/.../roles/myCustomRole` | Khi predefined quá rộng/quá hẹp |

> `Editor` cho phép làm gần như mọi thứ trừ quản lý billing/IAM — đừng cấp Editor cho cả team "cho tiện". Đó là cách nhanh nhất để ai đó vô tình xóa production.

### Gán role cho user

**Console:** IAM & Admin → IAM → **Grant access** → nhập email → chọn role → Save.

**gcloud:**

```bash
# Cho bạn A chỉ được xem (viewer) project
gcloud projects add-iam-policy-binding web-ban-hang-412301 \
  --member=user:ban-a@gmail.com \
  --role=roles/viewer

# Cho bạn B quản trị VM nhưng không đụng tới billing/storage
gcloud projects add-iam-policy-binding web-ban-hang-412301 \
  --member=user:ban-b@gmail.com \
  --role=roles/compute.instanceAdmin
```

Xem ai đang có quyền gì:

```bash
gcloud projects get-iam-policy web-ban-hang-412301
```

### Service account là gì?

**Service account** là "tài khoản robot" cho ứng dụng/máy móc — không phải người. Ví dụ: app chạy trên VM cần đọc bucket → tạo service account, gán role đọc bucket, app dùng nó để xác thực.

**Tạo service account + key:**

```bash
# 1. Tạo service account
gcloud iam service-accounts create backend-reader \
  --display-name="Backend bucket reader"

# 2. Gán role chỉ-đọc object trong project
gcloud projects add-iam-policy-binding web-ban-hang-412301 \
  --member=serviceAccount:backend-reader@web-ban-hang-412301.iam.gserviceaccount.com \
  --role=roles/storage.objectViewer

# 3. Tạo key JSON (để app ở ngoài GCP dùng)
gcloud iam service-accounts keys create ./backend-reader-key.json \
  --iam-account=backend-reader@web-ban-hang-412301.iam.gserviceaccount.com
```

App dùng key qua biến môi trường:

```bash
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/backend-reader-key.json
```

> **Cảnh báo:** file key JSON là "chìa khóa" — ai có nó là có toàn bộ quyền của service account. **Không commit lên GitHub, không gửi qua chat.** Lộ key → vào Console **xóa key ngay** (IAM → Service accounts → Keys → Delete).

### Best practice: least privilege (quyền tối thiểu)

1. **Chỉ cấp đúng quyền cần thiết**, không hơn. Cần đọc bucket thì cho `storage.objectViewer`, đừng cho `Editor`.
2. Gán quyền ở **mức tài nguyên hẹp nhất** có thể (bucket cụ thể thay vì cả project).
3. **Ưu tiên predefined role** hơn basic role.
4. Dùng **service account riêng cho từng ứng dụng**, không xài chung một con.
5. **Xoay (rotate) key định kỳ**, xóa key không dùng.
6. Bật **IAM recommender** (trong Console) để Google gợi ý thu hẹp quyền thừa.

---

## 12. gcloud CLI — điều khiển GCP từ terminal

`gcloud` là "điều khiển từ xa" cho toàn bộ GCP: tạo VM, deploy function, xem log… không cần mở trình duyệt.

### Cài đặt

**Windows:**
1. Tải installer tại **https://cloud.google.com/sdk/docs/install** → chạy file `.exe`.
2. Hoặc bằng winget: `winget install Google.CloudSDK`

**macOS:**
```bash
# Cách 1: tải archive từ trang docs, giải nén, chạy ./install.sh
# Cách 2: Homebrew (cask)
brew install --cask google-cloud-sdk
```

**Linux (Debian/Ubuntu):**
```bash
# Thêm repo Google rồi cài
sudo apt-get install -y apt-transport-https ca-certificates gnupg
echo "deb https://packages.cloud.google.com/apt cloud-sdk main" \
  | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg \
  | sudo apt-key add -
sudo apt-get update && sudo apt-get install -y google-cloud-sdk
```

Kiểm tra:

```bash
$ gcloud --version
Google Cloud SDK 520.0.0
```

### Khởi tạo: gcloud init & auth login

```bash
# Chạy wizard: chọn tài khoản, project mặc định, region/zone mặc định
gcloud init
```

Wizard sẽ:
1. Mở trình duyệt để bạn đăng nhập (`gcloud auth login` làm đúng việc này nếu muốn riêng lẻ).
2. Hỏi chọn project mặc định.
3. Hỏi chọn **region/zone mặc định** (ví dụ `asia-southeast1` / `asia-southeast1-a`) — để sau này khỏi gõ `--zone` mỗi lệnh.

Các lệnh auth/config hay dùng:

```bash
# Đăng nhập (lại)
gcloud auth login

# Liệt kê tài khoản đã đăng nhập
gcloud auth list

# Xem config hiện tại
gcloud config list

# Đổi project / region / zone mặc định
gcloud config set project web-ban-hang-412301
gcloud config set compute/region asia-southeast1
gcloud config set compute/zone asia-southeast1-a
```

### Các lệnh hay dùng theo nhóm (kèm ví dụ thực tế)

**Compute Engine:**

```bash
# Tạo VM (đã có ví dụ ở mục 6)
gcloud compute instances create web-server-1 --zone=asia-southeast1-a \
  --machine-type=e2-micro --image-family=ubuntu-2404-lts-amd64 \
  --image-project=ubuntu-os-cloud

# SSH vào VM
gcloud compute ssh web-server-1 --zone=asia-southeast1-a

# Xem địa chỉ IP
gcloud compute addresses list

# Tạo firewall rule mở port 8080
gcloud compute firewall-rules create allow-8080 \
  --allow=tcp:8080 --source-ranges=0.0.0.0/0 \
  --description="Allow app port 8080"
```

**Cloud Storage:**

```bash
gcloud storage buckets create gs://shop-duongg-backup --location=asia-southeast1
gcloud storage cp -r ./data/ gs://shop-duongg-backup/
gcloud storage ls gs://shop-duongg-backup/
gcloud storage du -s gs://shop-duongg-backup/   # xem dung lượng bucket
```

**Cloud Functions:**

```bash
gcloud functions list --region=asia-southeast1
gcloud functions describe hello-python --gen2 --region=asia-southeast1
gcloud functions logs read hello-python --gen2 --region=asia-southeast1 --limit=20
gcloud functions delete hello-python --gen2 --region=asia-southeast1
```

**Cloud Run:**

```bash
gcloud run deploy hello-run --source=. --region=asia-southeast1 --allow-unauthenticated
gcloud run services list --region=asia-southeast1
gcloud run services describe hello-run --region=asia-southeast1
gcloud run services delete hello-run --region=asia-southeast1
```

**Logging (xem log mọi dịch vụ):**

```bash
# Log của VM web-server-1 trong 1 giờ qua
gcloud logging read \
  'resource.type="gce_instance" AND resource.labels.instance_id="web-server-1"' \
  --limit=20 --format=json

# Log lỗi của Cloud Run service hello-run
gcloud logging read \
  'resource.type="cloud_run_revision" AND severity>=ERROR' \
  --limit=20
```

**Linh tinh hữu ích:**

```bash
# Xem project đang dùng + quota
gcloud projects describe web-ban-hang-412301
gcloud compute project-info describe --project=web-ban-hang-412301  # xem quota

# Bật API cho dịch vụ (ví dụ Cloud Build)
gcloud services enable cloudbuild.googleapis.com
gcloud services list --enabled   # liệt kê API đang bật
```

---

## 13. Cách tính tiền (Pricing)

Nguyên tắc chung: **trả theo mức dùng (pay-as-you-go)**, tính theo giây/phút, không có phí "mở tài khoản".

| Dịch vụ | Tính tiền theo | Ví dụ giá (tham khảo, region Singapore) |
|---|---|---|
| Compute Engine | vCPU + RAM theo giây VM chạy + ổ đĩa + IP tĩnh | e2-micro ~$6.11/tháng chạy 24/7; e2-medium ~$24/tháng |
| Cloud Storage Standard | GB lưu trữ/tháng + số request + egress | ~$0.02/GB/tháng |
| Cloud Storage Nearline/Coldline/Archive | Rẻ dần ($0.01 → $0.004 → $0.0012/GB) + phí đọc ra | Archive ~$0.0012/GB/tháng |
| Cloud Functions Gen2 | Số lần gọi + GB-giây + CPU-giây | 2 triệu request/tháng miễn phí, sau đó ~$0.40/triệu request |
| Cloud Run | vCPU-giây + RAM GB-giây + request | 2 triệu request + 360k vCPU-giây miễn phí/tháng |
| BigQuery | Dữ liệu quét ($5/TB sau 1TB free) + lưu trữ ($0.02/GB) | Query 100GB ~ $0.50 |
| Cloud SQL | Instance theo giờ + storage + backup | db-f1-micro ~$7–9/tháng |
| Network egress | GB truyền ra internet (vào thì miễn phí) | ~$0.12/GB (giảm dần theo bậc) |

> Giá thay đổi theo region và thời điểm — luôn kiểm tra trang pricing chính thức trước khi ước tính cho dự án thật.

### Pricing Calculator — ước tính trước khi làm

1. Vào **https://cloud.google.com/products/calculator**.
2. Bấm **Add to estimate** → chọn dịch vụ (ví dụ Compute Engine).
3. Nhập cấu hình (machine type, region, thời gian chạy) → ra số tiền/tháng ước tính.
4. Có thể **lưu/share estimate** dưới dạng link gửi cho team.

Mẹo thực tế:
- VM chạy 24/7 thì xem thêm **Committed Use Discount** (cam kết 1–3 năm, giảm tới 57%).
- Workload ngắt quãng → **Spot VM** rẻ hơn nhiều.
- BigQuery: bật **cost controls** (giới hạn bytes quét tối đa/query) cho tài khoản team mới.

---

## 14. Free Tier: $300 credit và Always Free

### $300 credit — 90 ngày

- Nhận ngay khi đăng ký, dùng cho **hầu hết dịch vụ GCP**.
- Hết 90 ngày **hoặc** hết $300 (điều kiện nào đến trước) → trial kết thúc.
- **Không tự trừ tiền thẻ** khi hết trial — bạn phải chủ động **Upgrade** mới bị tính phí.

### Always Free — miễn phí vĩnh viễn (không cần credit)

Những món "xài hoài không tốn tiền" (giới hạn theo tháng, vượt thì tính phí phần vượt):

| Dịch vụ | Hạn mức Always Free |
|---|---|
| Compute Engine | 1 VM **e2-micro** ở region Mỹ (us-west1/us-central1/us-east1) + 30GB disk + 1GB egress |
| Cloud Storage | **5GB** Standard + 5,000 Class A + 50,000 Class B operations |
| BigQuery | **1TB** dữ liệu quét + 10GB lưu trữ |
| Cloud Functions Gen2 | **2 triệu** invocations + 400k GB-giây |
| Cloud Run | **2 triệu** requests + 360k vCPU-giây + 720k GiB-giây |
| Firestore | 1GB lưu trữ + 50k reads + 20k writes + 20k deletes/ngày |
| Pub/Sub | 10GB message/tháng |
| Cloud Build | 120 phút build/ngày |

> Lưu ý: e2-micro free chỉ áp dụng ở **3 region Mỹ** — chạy e2-micro ở Singapore thì vẫn tính tiền (dù rẻ). Muốn free VM thì đặt ở `us-central1`.

### Cách tránh bị charge tiền ngoài ý muốn

1. **Tạo budget alert ngay** sau khi gắn billing ([mục 4](#4-thiết-lập-billing)).
2. **Dùng region free** cho VM thử nghiệm (`us-central1-a`).
3. **Tắt/xóa VM khi không dùng** — đừng để chạy qua đêm "cho vui".
4. **Để ý BigQuery**: luôn xem "query sẽ quét bao nhiêu GB" trước khi Run; đặt **custom quota** giới hạn bytes/ngày nếu share project cho người mới.
5. **Đừng public bucket chứa dữ liệu lớn** — ai đó tải nhiều là bạn trả tiền egress.
6. **Xóa static IP không dùng**: IP tĩnh giữ mà không gắn VM nào vẫn tính phí (~$7/tháng).
7. Cuối mỗi buổi vọc: vào **Billing → Reports** liếc qua xem có dòng tiền lạ không.

---

## 15. Xử lý lỗi thường gặp

### ❌ Quota exceeded

**Thông báo mẫu:**
```
Quota 'CPUS' exceeded. Limit: 8.0 in region asia-southeast1.
```

**Nguyên nhân:** project mới bị giới hạn quota thấp (ví dụ 8 vCPU/region) để chống gian lận.

**Cách xử lý:**
1. Vào **IAM & Admin → Quotas** → lọc metric `CPUs` → chọn region → **Edit quotas** → xin nâng (thường được duyệt trong vài phút–vài giờ).
2. Tạm thời: dùng machine type nhỏ hơn, hoặc chuyển sang region khác còn quota.
3. Xóa VM không dùng để giải phóng quota.

```bash
# Xem quota CPU hiện tại
gcloud compute project-info describe --project=web-ban-hang-412301 \
  --format="yaml(quotas)"
```

### ❌ Permission denied (IAM)

**Thông báo mẫu:**
```
ERROR: (gcloud.compute.instances.create) Could not fetch resource:
- Required 'compute.instances.create' permission for 'projects/web-ban-hang-412301'
```

**Nguyên nhân:** tài khoản/service account thiếu role.

**Cách xử lý:**
1. Kiểm tra mình đang đăng nhập bằng ai: `gcloud auth list`.
2. Kiểm tra role của mình: **IAM & Admin → IAM** → tìm email → xem cột Role.
3. Nhờ Owner/Editor gán thêm role cần thiết (ví dụ `roles/compute.instanceAdmin`).
4. Nếu dùng service account key: kiểm tra key có đúng file không, `GOOGLE_APPLICATION_CREDENTIALS` có trỏ đúng không.

### ❌ Billing not enabled

**Thông báo mẫu:**
```
Billing account not configured. Visit https://console.developers.google.com/billing to enable billing.
```

**Nguyên nhân:** project chưa gắn billing account (dịch vụ trả phí yêu cầu billing kể cả khi còn credit $300).

**Cách xử lý:**
1. Vào **Billing** → tạo/chọn billing account (nhập thẻ nếu chưa có).
2. **Billing → My projects** → gắn billing account vào project.
3. Chạy lại lệnh — lỗi biến mất ngay.

### Các lỗi khác hay gặp

| Lỗi | Nguyên nhân thường gặp | Cách xử lý nhanh |
|---|---|---|
| `Bucket name already exists` | Tên bucket phải duy nhất toàn cầu | Thêm hậu tố riêng: `shop-duongg-xyz` |
| `Zone ... does not have enough resources` | Zone hết tài nguyên loại máy đó | Đổi zone khác trong cùng region |
| `API ... has not been used` | Chưa bật API của dịch vụ | `gcloud services enable <api>.googleapis.com` |
| SSH timeout vào VM | Firewall chặn port 22 | Kiểm tra firewall rule `default-allow-ssh` còn không |
| `image not found` khi tạo VM | Sai image family/project | Dùng `gcloud compute images list` để xem tên đúng |

---

## 16. Best practices & Cheat sheet gcloud

### Best practices — thói quen tốt trên GCP

**Đặt tên & tổ chức**
- Tên tài nguyên: chữ thường, có quy ước `<dự-án>-<môi-trường>-<loại>-<số>` — ví dụ `shop-prod-vm-01`, `shop-prod-avatars`.
- Tách project theo môi trường: `shop-prod`, `shop-staging`, `shop-dev`.
- Dùng **Folder** khi có nhiều project/team.

**Tag & Label — "dán nhãn" mọi thứ**
- Label là cặp key=value gắn vào VM, bucket, disk…: `env=prod`, `team=backend`, `owner=duongg`.
- Dùng label để **lọc trong Billing Reports** → biết team nào đang đốt tiền.
- Tag mạng (network tag như `http-server`) để gắn firewall rule.

```bash
# Gắn label cho VM
gcloud compute instances add-labels web-server-1 \
  --zone=asia-southeast1-a --labels=env=prod,team=backend

# Lọc VM theo label
gcloud compute instances list --filter="labels.env=prod"
```

**Dọn dẹp tài nguyên không dùng (định kỳ hàng tuần)**
- VM nào STOPPED quá 7 ngày mà không ai nhận → xóa.
- Snapshot cũ, disk "mồ côi" (không gắn VM nào), static IP không dùng → xóa.
- Dùng **Billing → Reports** nhóm theo SKU để phát hiện "vòi tiền" bất thường.

**Budget alert cho mọi project quan trọng** ([mục 4](#4-thiết-lập-billing)) — ngưỡng 50/90/100%.

**Bảo mật**
- Least privilege ([mục 11](#11-iam--phân-quyền)): không cấp Editor "cho tiện".
- Key service account: không commit lên git, xoay định kỳ, xóa key thừa.
- Bật **2FA** cho tài khoản Google gắn với GCP.

### Cheat sheet gcloud — lệnh tra nhanh

```bash
# ── Khởi tạo & config ─────────────────────────────
gcloud init                                   # wizard khởi tạo
gcloud auth login                             # đăng nhập
gcloud auth list                              # xem tài khoản
gcloud config list                            # xem config
gcloud config set project PROJECT_ID          # đặt project mặc định
gcloud config set compute/zone asia-southeast1-a

# ── Project & billing ─────────────────────────────
gcloud projects list                          # liệt kê project
gcloud projects create PID --name="Tên"        # tạo project
gcloud billing accounts list                  # xem billing account
gcloud billing projects link PID --billing-account=ACC

# ── Compute Engine ────────────────────────────────
gcloud compute instances list
gcloud compute instances create NAME --zone=Z --machine-type=e2-micro \
  --image-family=ubuntu-2404-lts-amd64 --image-project=ubuntu-os-cloud
gcloud compute ssh NAME --zone=Z
gcloud compute instances start|stop|delete NAME --zone=Z
gcloud compute disks snapshot DISK --zone=Z --snapshot-names=SNAP
gcloud compute firewall-rules create allow-8080 --allow=tcp:8080

# ── Cloud Storage ─────────────────────────────────
gcloud storage buckets create gs://BUCKET --location=asia-southeast1
gcloud storage ls gs://BUCKET/
gcloud storage cp ./file gs://BUCKET/          # upload
gcloud storage cp -r ./dir/ gs://BUCKET/dir/  # upload thư mục
gcloud storage cp gs://BUCKET/file ./         # download
gcloud storage du -s gs://BUCKET/             # dung lượng
gcloud storage rm gs://BUCKET/file            # xóa object

# ── Cloud Functions ───────────────────────────────
gcloud functions deploy NAME --gen2 --runtime=python312 \
  --region=asia-southeast1 --source=. --entry-point=FUNC --trigger-http \
  --allow-unauthenticated
gcloud functions list --region=asia-southeast1
gcloud functions logs read NAME --gen2 --region=asia-southeast1 --limit=20
gcloud functions delete NAME --gen2 --region=asia-southeast1

# ── Cloud Run ─────────────────────────────────────
gcloud run deploy SVC --source=. --region=asia-southeast1 --allow-unauthenticated
gcloud run services list --region=asia-southeast1
gcloud run services describe SVC --region=asia-southeast1
gcloud run services update SVC --region=asia-southeast1 \
  --min-instances=0 --max-instances=5
gcloud run services delete SVC --region=asia-southeast1

# ── BigQuery ──────────────────────────────────────
bq mk --location=asia-southeast1 DATASET
bq ls
bq query --use_legacy_sql=false 'SELECT 1'
bq load --autodetect --source_format=CSV DATASET.TABLE ./file.csv

# ── IAM ───────────────────────────────────────────
gcloud projects get-iam-policy PROJECT_ID
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:email@gmail.com --role=roles/viewer
gcloud iam service-accounts create NAME --display-name="Mô tả"
gcloud iam service-accounts keys create ./key.json \
  --iam-account=NAME@PROJECT_ID.iam.gserviceaccount.com

# ── Logging & API ─────────────────────────────────
gcloud logging read 'resource.type="gce_instance"' --limit=20
gcloud services enable cloudbuild.googleapis.com
gcloud services list --enabled
```

---

> **Tổng kết:** bạn đã đi qua toàn bộ vòng đời dùng GCP — từ tạo tài khoản, project, billing, tới deploy VM, lưu file, chạy serverless, phân tích dữ liệu bằng BigQuery, phân quyền IAM và điều khiển mọi thứ bằng gcloud. Bước tiếp theo: chọn một ý tưởng nhỏ (ví dụ deploy web cá nhân lên Cloud Run + lưu ảnh trên Cloud Storage) và làm thật — vọc thật nhớ lâu hơn đọc mười bài hướng dẫn. Chúc vui! 🦫

*Viết bởi Capybara 🦫 cho Tiểu Dươngg — 2026-10-10*
