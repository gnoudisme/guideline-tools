# Hướng dẫn Bamboo — CI/CD của Atlassian

> Bamboo là máy chủ CI/CD của **Atlassian** (cùng nhà Jira, Bitbucket, Confluence): mỗi khi developer đẩy code, Bamboo tự động build, chạy test, đóng gói và triển khai.
>
> Hướng dẫn này đi từ khái niệm cơ bản tới cài đặt, Build Plan đầu tiên, quản trị agent, deployment, Specs as Code, tích hợp Jira/Bitbucket, backup/upgrade và xử lý sự cố.

**Đối tượng:** developer, QA, DevOps trong hệ sinh thái Atlassian. **Tham chiếu:** Bamboo 9.x.

---

## Mục lục

1. [Bamboo là gì?](#1-bamboo-là-gì)
2. [So sánh: Bamboo vs Jenkins vs GitHub Actions vs GitLab CI](#2-so-sánh-bamboo-vs-jenkins-vs-github-actions-vs-gitlab-ci)
3. [Kiến trúc tổng quan](#3-kiến-trúc-tổng-quan)
4. [Cài đặt Bamboo](#4-cài-đặt-bamboo)
5. [Tour giao diện: Dashboard, Projects, Plans](#5-tour-giao-diện-dashboard-projects-plans)
6. [Build Plan đầu tiên (step-by-step)](#6-build-plan-đầu-tiên-step-by-step)
7. [Stages, Jobs, Tasks — chi tiết](#7-stages-jobs-tasks--chi-tiết)
8. [Triggers — kích hoạt build](#8-triggers--kích-hoạt-build)
9. [Variables & Bamboo Specs](#9-variables--bamboo-specs)
10. [Agents — Local, Remote, Elastic](#10-agents--local-remote-elastic)
11. [Artifacts — tạo, chia sẻ, tải về](#11-artifacts--tạo-chia-sẻ-tải-về)
12. [Deployment Projects — Dev/Staging/Prod](#12-deployment-projects--devstagingprod)
13. [Specs as Code — YAML và Java](#13-specs-as-code--yaml-và-java)
14. [Tích hợp Bitbucket, Jira, Confluence](#14-tích-hợp-bitbucket-jira-confluence)
15. [Notifications — thông báo](#15-notifications--thông-báo)
16. [Branch Plans & Pull Request Automation](#16-branch-plans--pull-request-automation)
17. [Permissions & Security](#17-permissions--security)
18. [Backup / Restore / Upgrade](#18-backup--restore--upgrade)
19. [Pricing — Server / Data Center / Cloud](#19-pricing--server--data-center--cloud)
20. [Troubleshooting](#20-troubleshooting)
21. [Best Practices](#21-best-practices)
22. [FAQ](#22-faq)
23. [Cheat Sheet](#23-cheat-sheet)

---

## 1. Bamboo là gì?

**Bamboo** là sản phẩm CI/CD server của **Atlassian** (cùng nhà với Jira, Bitbucket, Confluence, Trello). Ra mắt từ năm 2007, Bamboo thuộc thế hệ CI server đời đầu cùng với Jenkins/Hudson, nhưng điểm khác biệt lớn nhất là **tích hợp sâu, liền mạch với hệ sinh thái Atlassian**:

- Ticket Jira nào dẫn tới commit nào → build nào → deploy lên môi trường nào: tất cả hiển thị ngay trong Jira (mục *Releases* / *Development panel*).
- Pull request trên Bitbucket tự động trigger build kiểm tra; kết quả build hiển thị ngay trên PR (✅/❌), có thể chặn merge khi build đỏ.
- Tài liệu Confluence có thể nhúng trạng thái build mới nhất.

### CI/CD là gì? (nhắc nhanh cho người mới)

| Thuật ngữ | Ý nghĩa |
|---|---|
| **CI (Continuous Integration)** | Mỗi commit được tự động build + test. Phát hiện lỗi sớm, tránh "trên máy tôi chạy được". |
| **CD — Continuous Delivery** | Sau CI pass, artifact (file .jar, image Docker...) sẵn sàng deploy chỉ với 1 cú click. |
| **CD — Continuous Deployment** | Đi xa hơn: tự động deploy lên production sau khi mọi check pass, không cần click. |

### Bamboo quản lý cái gì?

| Khái niệm | Giải thích nôm na |
|---|---|
| **Project** | Nhóm các Plan lại, ví dụ project "Website bán hàng". |
| **Plan** | Một "dây chuyền" CI: định nghĩa *khi nào* build và *build cái gì*. |
| **Stage** | Giai đoạn trong Plan (ví dụ: Build → Test → Release). Chạy tuần tự. |
| **Job** | Công việc trong Stage (ví dụ: job "Unit test", job "Integration test"). Các job trong cùng stage chạy song song. |
| **Task** | Bước nhỏ nhất trong Job: checkout code, chạy `mvn test`, chạy script shell... Chạy tuần tự. |
| **Build** | Một lần chạy (run) của Plan. Build #1, #2, #3... |
| **Agent** | "Công nhân" thực thi build: một tiến trình Java chạy trên máy nào đó. |
| **Artifact** | Sản phẩm đầu ra: file `.jar`, `.war`, image Docker, báo cáo test... |
| **Deployment Project** | Dây chuyền CD: đưa artifact lên các môi trường Dev → Staging → Prod. |

Sơ đồ quan hệ:

```
Project "ShopApp"
 └── Plan "ShopApp-Backend-CI"  (build mỗi commit)
 │    ├── Stage "Build"
 │    │    └── Job "Compile" ── Tasks: [Checkout] → [mvn package]
 │    ├── Stage "Test"
 │    │    ├── Job "Unit Test" ── Tasks: [mvn test]
 │    │    └── Job "API Test"  ── Tasks: [newman run ...]
 │    └── Stage "Release"
 │         └── Job "Dockerize" ── Tasks: [docker build] → [docker push]
 └── Deployment Project "ShopApp-Deploy"
      ├── Environment "Dev"     (auto deploy)
      ├── Environment "Staging" (auto deploy)
      └── Environment "Prod"    (manual + approval)
```

---

## 2. So sánh: Bamboo vs Jenkins vs GitHub Actions vs GitLab CI

| Tiêu chí | **Bamboo** | **Jenkins** | **GitHub Actions** | **GitLab CI** |
|---|---|---|---|---|
| Nhà phát triển | Atlassian (thương mại) | Open source (CD Foundation) | GitHub/Microsoft | GitLab |
| Mô hình chạy | Self-hosted (Server/DC) | Self-hosted | Cloud (runner) + self-hosted runner | Cloud + self-hosted runner |
| Định nghĩa pipeline | UI click + **Specs as Code** (YAML/Java) | `Jenkinsfile` (Groovy) / UI | YAML (`.github/workflows`) | YAML (`.gitlab-ci.yml`) |
| Tích hợp Jira/Bitbucket | ⭐⭐⭐⭐⭐ Native, 2 chiều | Qua plugin | Qua app/marketplace | Tích hợp sẵn GitLab |
| Deployment/CD | ⭐ Có **Deployment Project** riêng, approval, environment | Phải tự dựng bằng pipeline | Có Environments + approvals | Có Environments, review apps |
| Plugin ecosystem | ~200+ app trên Marketplace | ⭐⭐⭐⭐⭐ ~1800+ plugin | ~20k+ actions | Tích hợp sẵn, ít plugin |
| Chi phí | Trả phí theo agent (xem mục 19) | Miễn phí (tốn công vận hành) | Trả theo phút build | Trả theo user/seat |
| Độ khó setup | Trung bình (wizard cài đặt) | Trung bình–khó | Dễ | Dễ–trung bình |
| Phù hợp khi | Team đã dùng Jira+Bitbucket, cần CD chuẩn enterprise, self-hosted | Muốn miễn phí + tùy biến tối đa | Code đã ở GitHub, muốn nhanh gọn | Dùng GitLab làm trung tâm |

**Khi nào chọn Bamboo?** ✅ Team đã dùng Jira + Bitbucket, cần CD chuẩn enterprise (approval, audit) và bắt buộc self-hosted. ❌ Team nhỏ code trên GitHub/GitLab → Actions/GitLab CI nhanh rẻ hơn; muốn miễn phí hoàn toàn → Jenkins.

---

## 3. Kiến trúc tổng quan

```
┌─────────────┐      push/PR       ┌──────────────────┐
│  Bitbucket  │ ─────────────────▶ │                  │
└─────────────┘                    │   BAMBOO SERVER  │
┌─────────────┐   issue key        │  (Tomcat + Java) │
│    Jira     │ ◀────────────────▶ │                  │
└─────────────┘                    │  ┌─────────────┐ │
                                   │  │  Database   │ │── PostgreSQL/MySQL/MSSQL
┌─────────────┐                    │  │  (shared)   │ │
│ Confluence  │                    │  └─────────────┘ │
└─────────────┘                    │  ┌─────────────┐ │
                                   │  │ Home dir    │ │── plans, artifacts, logs
                                   │  └─────────────┘ │
                                   └────────┬─────────┘
                                            │ phân việc (JMS)
                    ┌───────────────────────┼────────────────────────┐
                    ▼                       ▼                        ▼
            ┌──────────────┐        ┌──────────────┐          ┌──────────────┐
            │ Local Agent  │        │ Remote Agent │          │Elastic Agent │
            │(cùng máy Bamboo)│      │(máy build riêng)│       │(EC2 tự sinh) │
            └──────────────┘        └──────────────┘          └──────────────┘
                    chạy job: checkout → build → test → artifact → trả kết quả
```

- **Bamboo Server**: ứng dụng Java (chạy trên Tomcat nhúng), giữ toàn bộ cấu hình, lịch sử build, điều phối job.
- **Agent**: tiến trình Java nhận job từ server qua JMS, thực thi task, gửi kết quả + log về.
- **Shared home (Data Center)**: các node dùng chung thư mục home qua NFS.

---

## 4. Cài đặt Bamboo

### 4.1. Yêu cầu hệ thống (Bamboo 9.x)

| Thành phần | Yêu cầu tối thiểu | Khuyến nghị production |
|---|---|---|
| OS | Linux (RHEL/Ubuntu), Windows, macOS | Linux 64-bit |
| Java | OpenJDK 17 (Bamboo 9.x) | OpenJDK 17, heap 2–4GB |
| Database | PostgreSQL 13–15, MySQL 8, MSSQL, Oracle | **PostgreSQL** (Atlassian khuyến nghị) |
| RAM | 4 GB | 8–16 GB |
| Disk | 20 GB (home dir) | SSD, tách riêng home dir |
| Browser | Chrome/Firefox/Edge mới | — |

> ⚠️ **Lưu ý quan trọng:** Bamboo 8.x dùng Java 11; Bamboo 9.x dùng **Java 17**. Cài sai version Java là lỗi phổ biến nhất khiến Bamboo không khởi động.

### 4.2. Cài đặt Bamboo Server trên Linux (Ubuntu 22.04) — step by step

**Bước 1 — Cài Java 17 và PostgreSQL:**

```bash
sudo apt update
sudo apt install -y openjdk-17-jdk postgresql postgresql-contrib
java -version   # phải ra openjdk "17.0.x"
sudo systemctl enable --now postgresql
```

**Bước 2 — Tạo database (UTF-8, user riêng):**

```bash
sudo -u postgres psql <<'EOF'
CREATE USER bamboo WITH PASSWORD 'DoiMatKhauManh123!';
CREATE DATABASE bamboodb OWNER bamboo ENCODING 'UTF8';
GRANT ALL PRIVILEGES ON DATABASE bamboodb TO bamboo;
EOF
```

**Bước 3 — Tải, giải nén, tạo user `bamboo` (không chạy bằng root):**

```bash
cd /opt
sudo wget https://www.atlassian.com/software/bamboo/downloads/binary/atlassian-bamboo-9.6.0.tar.gz
sudo tar -xzf atlassian-bamboo-9.6.0.tar.gz && sudo mv atlassian-bamboo-9.6.0 bamboo
sudo useradd --create-home --shell /bin/bash bamboo
sudo mkdir -p /var/atlassian/bamboo-home
sudo chown -R bamboo:bamboo /opt/bamboo /var/atlassian/bamboo-home

# Trỏ Bamboo về home dir (tách data khỏi thư mục cài đặt)
echo "bamboo.home=/var/atlassian/bamboo-home" | sudo tee /opt/bamboo/atlassian-bamboo/WEB-INF/classes/bamboo-init.properties
sudo chown bamboo:bamboo /opt/bamboo/atlassian-bamboo/WEB-INF/classes/bamboo-init.properties
```

**Bước 4 — Tăng heap (sửa `bin/setenv.sh`):**

```bash
JVM_MINIMUM_MEMORY="1g"
JVM_MAXIMUM_MEMORY="4g"
```

**Bước 5 — Mở firewall, khởi động, hoàn tất trên browser:**

```bash
sudo ufw allow 8085/tcp
sudo -u bamboo /opt/bamboo/bin/start-bamboo.sh
tail -f /opt/bamboo/logs/catalina.out   # chờ "Server startup"
```

Mở `http://<server-ip>:8085` → wizard: license key → Database **PostgreSQL** (`localhost`/`bamboodb`/`bamboo`) → **Test Connection** → tạo tài khoản **admin** → Finish. Dừng: `stop-bamboo.sh`.

### 4.3. Chạy Bamboo như systemd service (tự khởi động cùng máy)

Tạo file `/etc/systemd/system/bamboo.service`:

```ini
[Unit]
Description=Atlassian Bamboo
After=network.target postgresql.service

[Service]
Type=forking
User=bamboo
Group=bamboo
ExecStart=/opt/bamboo/bin/start-bamboo.sh
ExecStop=/opt/bamboo/bin/stop-bamboo.sh
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now bamboo
sudo systemctl status bamboo
```

### 4.4. Cài đặt bằng Docker

**docker-compose.yml — Bamboo + PostgreSQL:**

```yaml
version: "3.8"

services:
  bamboo-db:
    image: postgres:15-alpine
    container_name: bamboo-db
    environment:
      POSTGRES_DB: bamboodb
      POSTGRES_USER: bamboo
      POSTGRES_PASSWORD: DoiMatKhauManh123!
    volumes:
      - bamboo-db-data:/var/lib/postgresql/data
    restart: unless-stopped

  bamboo:
    image: atlassian/bamboo:9.6.0-jdk17
    container_name: bamboo
    depends_on:
      - bamboo-db
    ports:
      - "8085:8085"
    environment:
      - JVM_MINIMUM_MEMORY=1g
      - JVM_MAXIMUM_MEMORY=4g
    volumes:
      - bamboo-home:/var/atlassian/application-data/bamboo
    restart: unless-stopped

volumes:
  bamboo-db-data:
  bamboo-home:
```

```bash
docker compose up -d
docker logs -f bamboo   # chờ "Server startup"
# Mở http://localhost:8085 → wizard, database host điền: bamboo-db
```

### 4.5. Bamboo Data Center — tổng quan

Khi một server không đủ tải, Bamboo **Data Center** chạy **cluster nhiều node** active-active sau load balancer + database chung + shared home (NFS): high availability, scale ngang, **zero-downtime upgrade** (rolling từng node). License tính theo số remote agent, giá cao hơn Server (mục 19). Team < 50 dev thường single-node là đủ.

### 4.6. Cấu hình reverse proxy (Nginx) + HTTPS

Production nên đặt Nginx trước Bamboo:

```nginx
server {
    listen 443 ssl;
    server_name bamboo.congty.vn;

    ssl_certificate     /etc/ssl/certs/bamboo.crt;
    ssl_certificate_key /etc/ssl/private/bamboo.key;

    location / {
        proxy_pass              http://127.0.0.1:8085;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Trong Bamboo: **⚙️ → General configuration → Base URL** → đổi thành `https://bamboo.congty.vn` (base URL sai khiến link trong email thông báo bị hỏng — lỗi hay gặp).

---

## 5. Tour giao diện: Dashboard, Projects, Plans

Đăng nhập xong, bạn sẽ thấy:

### 5.1. Dashboard (trang chủ)

- **My Bamboo**: plan bạn theo dõi, build của bạn, build đang chạy.
- **All Plans**: mỗi dòng = 1 plan: trạng thái build mới nhất (✅/❌/🟡), thời gian chạy, người trigger. Có filter theo project/label.
- **Build activity**: build đang chạy realtime.

### 5.2. Projects

**Projects → All projects**: mỗi Project gom nhiều Plan + Deployment Project. Ví dụ project `SHOP` gồm `SHOP-BE`, `SHOP-FE` và Deployment Project "Shop Deploy". Tạo mới: **Create → Create project** (nhập *Project name* + *Project key* viết hoa, ví dụ `SHOP`).

### 5.3. Màn hình Plan & Build result

Tab trong Plan: **Summary** (trạng thái + biểu đồ), **Builds** (lịch sử + log từng build), **Branches** (branch plan), **Issues** (Jira issue từ commit message), **Reports** (tỉ lệ pass, thời gian build).

Click vào 1 build: tab **Tests** (test nào fail + stack trace), **Artifacts** (tải file), **Logs** (debug ở đây), **Commits/Issues** (link Bitbucket/Jira).

### 5.4. Administration (⚙️)

**Agents** (thêm/xem agent), **Elastic Bamboo** (AWS), **Repositories**, **Global variables**, **Security/Permissions**, **Backup**, **Audit log**, **System** (version, license).

---

## 6. Build Plan đầu tiên (step-by-step)

Mục tiêu: tạo Plan build một project **Node.js** đơn giản (repo có `package.json` với script `test`) mỗi khi có push.

### Bước 1 — Tạo Plan

1. Trên thanh top: **Create → Create plan**.
2. Điền:
   - *Project*: chọn project có sẵn hoặc tạo mới (`DEMO`).
   - *Plan name*: `Hello Bamboo`.
   - *Plan key*: `HB` (tự sinh từ tên, key này dùng trong URL/API).
   - ✅ *Link a new repository to this plan*.
3. **Configure repository**:
   - *Repository type*: Git.
   - *Display name*: `hello-bamboo`.
   - *Repository URL*: `https://bitbucket.org/team/hello-bamboo.git` (hoặc GitHub).
   - *Authentication*: Username + password / SSH key / Access token.
   - *Branch*: `main`.
   - Bấm **Test** → báo *Connection successful* → **Continue**.
4. Bấm **Configure plan** (chưa vội Create vội — ta sẽ thêm task).

### Bước 2 — Thêm Tasks cho Job mặc định

Bamboo đã tạo sẵn Stage "Default Stage" → Job "Default Job" (có task *Source Code Checkout*). Vào **Actions → Configure plan → Stages** → click **Default Job** → tab **Tasks** → **Add task**:

1. **NPM** — *Task description*: `npm install`, *Command*: `install`.
2. **NPM** — *Task description*: `npm test`, *Command*: `run`, *Arguments*: `test`.

Thứ tự tasks: 1. Source Code Checkout → 2. npm install → 3. npm test.

### Bước 3 — Bật Trigger (build khi có push)

1. Tab **Triggers** → **Add trigger** → **Bitbucket Server repository triggered** (nếu dùng Bitbucket) hoặc **Polling the repository for changes**.
   - Với GitHub/Git chung: dùng *Polling* → *Polling frequency*: `180` giây.
2. **Create** (góc dưới) → Plan đã sẵn sàng.

### Bước 4 — Chạy thử và xem kết quả

1. **Actions → Run plan** (hoặc **Run** trên thanh plan).
2. Build #1 xuất hiện, trạng thái 🟡 đang chạy → click vào xem log realtime.
3. Xong: ✅ xanh là thành công. Thử push 1 commit hỏng test lên repo → vài phút sau Bamboo tự build và báo ❌ đỏ.

🎉 **Chúc mừng!** Bạn vừa có pipeline CI đầu tiên. Các mục tiếp theo sẽ đi sâu từng khái niệm.

---

## 7. Stages, Jobs, Tasks — chi tiết

### 7.1. Quy tắc chạy (hiểu để thiết kế pipeline đúng)

```
Plan
 ├── Stage 1 "Build"          ◀── chạy ĐẦU TIÊN
 │    ├── Job A ──┐
 │    └── Job B ──┘── chạy SONG SONG (mỗi job chiếm 1 agent)
 ├── Stage 2 "Test"           ◀── chỉ chạy khi TẤT CẢ job của Stage 1 pass
 │    └── Job C
 └── Stage 3 "Deploy"         ◀── chỉ chạy khi Stage 2 pass
      └── Job D (manual stage: cần bấm tay mới chạy)
```

- **Stage chạy tuần tự**, stage sau chỉ chạy khi stage trước **thành công toàn bộ**.
- **Job trong cùng stage chạy song song** trên các agent khác nhau → tận dụng để chia test cho nhanh.
- **Task trong job chạy tuần tự** trên cùng 1 agent, cùng workspace (thư mục checkout code).
- Có thể đặt stage là **Manual**: build dừng lại chờ người bấm "tiếp tục" (dùng cho deploy staging/prod trong plan thường).

> ⚠️ Mỗi **Job chiếm 1 agent**. Plan có 3 job chạy song song = cần 3 agent rảnh. Thiết kế job song song mà chỉ có 1 agent → build xếp hàng chờ, rất chậm.

### 7.2. Các loại Task phổ biến

Vào **Configure plan → <Job> → Tasks → Add task**, Bamboo có sẵn:

#### Source Code Checkout (mặc định)

- Checkout repo đã link ở plan (hoặc thêm repo khác ở cấp job).
- Tùy chọn: *Checkout directory* (thư mục con), *Clean working directory* sau build.

#### Script Task (linh hoạt nhất)

Chạy lệnh shell/batch/Python trực tiếp — "cứu cánh" cho mọi thứ không có task chuyên dụng.

| Field | Ví dụ |
|---|---|
| Interpreter | `Shell` (Linux/macOS) / `Windows PowerShell` |
| Script body | xem ví dụ dưới |

```bash
#!/bin/bash
set -e  # dừng ngay khi có lệnh lỗi
echo "Build version: ${bamboo.buildNumber}"

# Cài deps và build
npm ci
npm run build

# Chạy test, xuất báo cáo JUnit để Bamboo parse
npm test -- --reporters=junit --outputFile=test-results/junit.xml
```

> 💡 Biến `${bamboo.buildNumber}` là **Bamboo variable** (mục 9) — Bamboo tự thay giá trị khi chạy.

#### Maven 3.x / Gradle / NPM

- **Maven**: *Goal* `clean package`, chọn JDK trong dropdown, tick *Will produce test results* để có tab Tests đẹp.
- **Gradle**: *Tasks* `build`, tick *Use Gradle Wrapper* (không phụ thuộc Gradle trên agent).
- **NPM**: *Command* `install`/`ci`/`run`/`test` — trên CI dùng `ci` thay vì `install` (nhanh + đúng lockfile).

#### Docker Task

*Command* build/push/run image — hoặc dùng Script task khi cần kiểm soát nhiều (ví dụ dưới).

#### Các task khác hay dùng

**JUnit Parser** (parse XML test từ tool lạ), **Artifact download/upload** (mục 11), **SSH/SCP** (chạy lệnh, copy file lên server), **Inject Bamboo variables** (mục 9), **Command** (chạy 1 executable nhẹ hơn Script).

### 7.3. Job requirements — "chỉ chạy trên agent có..."

Mặc định job chạy trên **bất kỳ agent nào rảnh**. Nếu job cần Docker/JDK 17/Node 20, khai báo ở **Job → Requirements**, ví dụ `system.jdk.JDK 17` *exists* (khớp với capability cùng tên trên agent — xem mục 10.6).

> ⚠️ Không agent nào thỏa requirement → build **kẹt ở Queued mãi**. Thiết kế job song song mà thiếu agent cũng gây xếp hàng dài.

### 7.4. Ví dụ: Plan CI cho Spring Boot (3 stage)

- **Stage "Build"** → Job "Compile": Checkout → Maven `clean package -DskipTests` (JDK 17) → Artifact `app-jar` (`target/*.jar`, shared).
- **Stage "Test"** → 2 job song song: "Unit Test" (Maven `test`) và "Integration Test" (Script: `docker compose up -d postgres` → Maven `verify -Pintegration`).
- **Stage "Release"** (manual): Job "Dockerize" — download `app-jar` → Script `docker build/push` với tag = buildNumber.

---

## 8. Triggers — kích hoạt build

Vào **Configure plan → Triggers → Add trigger**:

| Trigger | Khi nào build chạy | Dùng khi |
|---|---|---|
| **Manual** | Chỉ khi bấm *Run* | Deploy prod, build đặc biệt |
| **Single daily build** | 1 lần/ngày, giờ cố định (cron) | Nightly build, regression test đêm |
| **Scheduled (cron)** | Theo biểu thức cron linh hoạt | Build định kỳ phức tạp |
| **Polling the repository** | Bamboo hỏi repo mỗi N giây có commit mới không | Repo Git chung (GitHub/GitLab qua polling) |
| **Bitbucket Server repository triggered** | Bitbucket **đẩy webhook** sang Bamboo khi có push | Bitbucket + Bamboo (nhanh, không tốn polling) |
| **Remote trigger** | Gọi URL/API từ ngoài | Trigger từ hệ thống khác |
| **Dependency (child/parent)** | Build sau khi plan khác build xong | Chuỗi: Lib build xong → App build |

### 8.1. Ví dụ cron trigger

Build nightly lúc 2h sáng mỗi ngày, chỉ thứ 2–6:

```
0 0 2 ? * MON-FRI
```

Cú pháp cron của Bamboo (Quartz): `giây phút giờ ngày-của-tháng tháng ngày-của-tuần [năm]`

| Ví dụ | Ý nghĩa |
|---|---|
| `0 0/15 * * * ?` | Mỗi 15 phút |
| `0 0 8 * * ?` | 8h sáng mỗi ngày |
| `0 30 18 ? * FRI` | 18:30 thứ 6 hàng tuần |

### 8.2. Polling vs Webhook

- **Webhook** (Bitbucket Server trigger): Bitbucket đẩy sự kiện sang Bamboo ngay khi push — nhanh (vài giây), ít tải. Yêu cầu 2 máy gọi được nhau.
- **Polling**: Bamboo hỏi repo mỗi N giây — đơn giản, dùng được với mọi Git repo (GitHub/GitLab), nhưng delay = chu kỳ poll.

### 8.3. Quiet period & trigger cho deployment

**Quiet period** (ví dụ 60s): Bamboo chờ sau commit cuối mới build → gom nhiều commit dồn dập vào 1 build, đỡ tốn agent. Deployment Project có trigger riêng: *After successful build plan*, *Scheduled*, *Manual* (xem mục 12).

---

## 9. Variables & Bamboo Specs

### 9.1. Bamboo Variables — biến dùng trong build

Biến giúp tham số hóa plan: cùng 1 plan chạy cho nhiều môi trường/nhánh mà không sửa task.

**3 phạm vi (scope):**

| Scope | Khai báo ở | Ví dụ truy cập |
|---|---|---|
| **Plan variable** | Configure plan → Variables | `${bamboo.myVar}` |
| **Job variable** | Ghi đè ở cấp job | `${bamboo.myVar}` (ưu tiên hơn plan) |
| **Global variable** | ⚙️ → Global variables | Dùng chung mọi plan |

**Biến có sẵn (không cần khai báo)** — hay dùng nhất:

| Biến | Giá trị |
|---|---|
| `${bamboo.buildNumber}` | Số build: 1, 2, 3... |
| `${bamboo.buildKey}` | Key build: `SHOP-BE-123` |
| `${bamboo.planKey}` | Key plan: `SHOP-BE` |
| `${bamboo.planRepository.branchName}` | Tên nhánh: `main`, `feature/x` |
| `${bamboo.repository.revision.number}` | Commit hash |
| `${bamboo.build.working.directory}` | Thư mục workspace của job |
| `${bamboo.agentId}` / `${bamboo.agent.name}` | Agent đang chạy job |

Dùng trong Script task: `${bamboo.planKey} #${bamboo.buildNumber}` → Bamboo tự thay giá trị khi chạy. Đặt tên biến chứa `password`/`secret`/`token` để Bamboo tự che `****` trong log.

### 9.2. Inject variables & biến mật

Task **Inject Bamboo variables** đọc file `.properties` thành biến build:

```properties
# version.properties (do task trước sinh ra)
app.version=2.4.1
```

> ⚠️ **Bảo mật:** không `echo` biến chứa password/token ra log. Tên biến chứa `password`/`secret`/`token` sẽ bị che `****` tự động. Tốt nhất: lưu credential ở **Shared credentials** (⚙️) rồi chọn trong task thay vì biến text.

---

## 10. Agents — Local, Remote, Elastic

Agent là "công nhân" chạy job. Hiểu agent = hiểu 80% vận hành Bamboo.

### 10.1. Các loại agent

| Loại | Chạy ở đâu | Đặc điểm |
|---|---|---|
| **Local agent** | Cùng máy với Bamboo Server | Có sẵn sau cài đặt, mặc định **disabled**. Dùng cho test nhanh, không nên dùng production (tranh tài nguyên với server) |
| **Remote agent** | Máy riêng (VM/physical/Docker) | Khuyên dùng production. Cài bao nhiêu tùy ý (tốn license theo số agent) |
| **Elastic agent** | EC2 trên AWS, tự tạo/hủy theo nhu cầu | Trả tiền AWS theo giờ dùng. Phù hợp build bùng nổ theo đợt |

**Quy tắc vàng:** 1 agent tại 1 thời điểm chỉ chạy **1 job**. Có 10 job song song mà 2 agent → 8 job xếp hàng.

### 10.2. Bật Local Agent (test nhanh)

**⚙️ → Agents** → tab *Local agents* → `Default Agent` → **Enable**. Production nên **disable** local agent, dùng remote agent để build không "ăn" tài nguyên server.

### 10.3. Cài đặt Remote Agent — step by step (Linux)

**Bước 1 — Chuẩn bị máy agent** (cùng Java version với server — Java 17 cho Bamboo 9.x — và tool build cần thiết):

```bash
sudo apt update && sudo apt install -y openjdk-17-jdk git maven nodejs npm docker.io
```

**Bước 2 — Tải installer từ server:** Bamboo **⚙️ → Agents → Install remote agents** → copy lệnh, hoặc tải trực tiếp `http://<bamboo-server>:8085/agentServer/atlassian-bamboo-agent-installer.jar`.

**Bước 3 — Chạy trên máy agent:**

```bash
mkdir -p /opt/bamboo-agent && cd /opt/bamboo-agent
wget http://192.168.1.10:8085/agentServer/atlassian-bamboo-agent-installer.jar
java -jar atlassian-bamboo-agent-installer.jar http://192.168.1.10:8085/agentServer/
# Chọn Yes khi hỏi cài như service
```

**Bước 4 — Duyệt trên server:** **⚙️ → Agents** → agent mới ở *Pending approval* → **Approve** → **Enable**. 🎉

### 10.4. Remote Agent bằng Docker

```bash
docker run -d --name bamboo-agent \
  -e BAMBOO_SERVER=http://192.168.1.10:8085/agentServer \
  -v bamboo-agent-data:/home/bamboo/bamboo-agent-home \
  --restart unless-stopped \
  atlassian/bamboo-agent-base
```

### 10.5. Elastic Agents (AWS EC2)

Cho nhu cầu build bùng nổ theo đợt: Bamboo tự bật/tắt EC2 theo độ dài queue.

1. **⚙️ → Elastic Bamboo → Configuration**: nhập AWS Access Key/Secret, chọn Region (gần VN nhất: `ap-southeast-1` Singapore).
2. **Image configurations**: chọn AMI của Atlassian hoặc AMI tự build.
3. Đặt số instance tối đa và loại instance (`t3.medium`...).

> 💡 Chi phí = tiền EC2 trả cho AWS. Rẻ hơn nuôi dàn remote agent cố định nếu build theo đợt.

### 10.6. Capabilities & Requirements — "ghép đôi" job với agent

**Capability** = "agent làm được gì" (agent tự báo + admin thêm tay); **Requirement** = "job cần gì" (khai báo ở job). Ví dụ: capability `system.jdk.JDK 17` trên agent ↔ requirement `system.jdk.JDK 17` *exists* ở job. Xem/sửa tại **⚙️ → Agents → <tên agent> → Capabilities**.

**Tình huống hay gặp:** build **stuck ở "Queued" mãi** → kiểm tra job requirements có gì mà không agent online nào thỏa (⚙️ → Agents xem capability từng agent).

### 10.7. Dedicated Agent — agent riêng cho plan quan trọng

**Configure plan → <Job> → Miscellaneous** → *Dedicated agent*: gán agent chỉ chạy plan chỉ định → plan release luôn có agent trống, không xếp hàng.

> ⚠️ Lạm dụng gây lãng phí: agent ngồi chờ plan "của mình" trong khi plan khác xếp hàng dài.

---

## 11. Artifacts — tạo, chia sẻ, tải về

Artifact là "thành phẩm" của build: file `.jar`, `.zip`, image, báo cáo... Có 2 mục đích chính:

1. **Chia sẻ giữa các job/stage** (job Build tạo `.jar` → job Deploy lấy `.jar` đó dùng).
2. **Lưu trữ để deploy / tải về** (deployment project lấy artifact của build để đẩy lên server).

### 11.1. Tạo Artifact

**Configure plan → <Job tạo file> → Artifacts → Create artifact:**

| Field | Ví dụ |
|---|---|
| *Name* | `app-jar` |
| *Location* | `target` (tương đối từ working dir) |
| *Copy pattern* | `*.jar` (Ant-style; `**/*` = mọi file) |
| ✅ *Shared* | **Bắt buộc** nếu job khác / deployment cần dùng |

### 11.2. Dùng Artifact ở Job khác

**Job "Deploy" → Tasks → Add task → Artifact download:** chọn artifact `app-jar`, *Source plan* (có thể là plan khác!), *Destination path*. Đây là cầu nối CI → CD.

### 11.3. Artifact trong Deployment & quản lý dung lượng

Khi tạo Deployment Project, Bamboo hỏi lấy artifact nào làm **release** (mục 12). ⚠️ Artifact rất tốn disk: đặt **expiry** cho mọi artifact (ví dụ giữ 30 ngày / 50 build) tại **Configure plan → Artifacts**, và chỉ share file thật sự cần (đừng artifact `node_modules`).

---

## 12. Deployment Projects — Dev/Staging/Prod

Đây là tính năng CD "xịn" nhất của Bamboo, thứ Jenkins phải dựng tay bằng pipeline. Deployment Project trả lời: **"Đưa bản build #123 lên môi trường nào, khi nào, ai được duyệt?"**

### 12.1. Tạo Deployment Project

**Chuẩn bị:** Plan `SHOP-BE` đã có artifact shared `app-jar`. **Create → Create deployment project** → *Name*: `Shop Backend Deploy` → *Source plan*: `SHOP-BE` → **Release versioning**: dùng build number (`release-123`). Bamboo tạo environment đầu tiên — ta đổi thành 3 môi trường dưới đây.

### 12.2. Thiết lập 3 môi trường Dev → Staging → Prod

Mỗi **Environment** = 1 đích deploy, gồm: **Tasks** (làm gì để deploy) + **Triggers** + **Approvals** + **Variables**.

#### Environment 1: Development (tự động)

- **Tasks:**
  1. *Artifact download*: lấy `app-jar` từ plan `SHOP-BE`.
  2. *Script/SSH task*: copy + restart app trên server dev:

```bash
scp deploy/app.jar dev-user@dev.congty.vn:/opt/shop/app.jar
ssh dev-user@dev.congty.vn "sudo systemctl restart shop-be"
```

- **Triggers → Add trigger**: *After successful build plan* → mỗi build xanh là tự deploy dev ngay.
- **Approvals**: không cần (dev deploy tự do).

#### Environment 2: Staging (tự động, sau Dev)

- **Tasks**: tương tự, đích là server staging + smoke test (`curl -sf https://staging.congty.vn/api/health`).
- **Triggers**: *After successful deployment* → environment **Development**.
- **Approvals**: không cần (hoặc QA approve).

#### Environment 3: Production (thủ công + phê duyệt)

- **Tasks**: đích là server prod — backup bản đang chạy, deploy rolling từng node, health check:

```bash
#!/bin/bash
set -e
ssh prod-user@prod.congty.vn "cp /opt/shop/app.jar /opt/shop/backup/app-$(date +%Y%m%d-%H%M).jar"
for node in prod-1 prod-2; do
  scp deploy/app.jar prod-user@$node.congty.vn:/opt/shop/app.jar
  ssh prod-user@$node.congty.vn "sudo systemctl restart shop-be && sleep 20"
  curl -sf https://$node.congty.vn/api/health || exit 1
done
```

- **Triggers**: **Manual only** — ai đó phải bấm *Deploy*.
- **Approvals → Add approver**: nhóm `release-managers` (có thể yêu cầu 2 người duyệt) → Bamboo **chặn** deploy tới khi đủ approve.

Sơ đồ luồng:

```
Build #123 ✅
   │ (trigger: after successful build)
   ▼
┌─────────┐   deploy OK    ┌──────────┐   manual+approve   ┌───────┐
│   DEV   │ ─────────────▶ │ STAGING  │ ────────────────▶ │ PROD  │
│ auto    │  (trigger:     │ auto     │   (bấm Deploy,    │manual │
│         │   after Dev)   │          │    chờ duyệt)     │       │
└─────────┘                └──────────┘                   └───────┘
```

### 12.3. Release & rollback

Mỗi lần deploy từ 1 build, Bamboo tạo **Release** (`release-123`). Tab **Releases** cho thấy release nào đang ở môi trường nào — và cho **deploy lại release cũ (rollback) chỉ với 1 click**.

### 12.4. Deployment Variables & Permissions

Mỗi environment có **Variables** riêng (`db.host`, `api.url`...) — task dùng `${bamboo.db.host}` → cùng 1 task deploy chạy đúng config từng môi trường. **Environment permissions** giới hạn ai được deploy/approve từng môi trường (dev deploy Dev, chỉ lead deploy Prod).

### 12.5. Ví dụ deploy bằng Docker/Kubernetes

Team dùng container thì task deploy là Script task + `kubectl` (agent cần có kubectl + kubeconfig):

```bash
#!/bin/bash
set -e
IMAGE="registry.congty.vn/shop-be:${bamboo.deploy.release}"
kubectl --kubeconfig /etc/kube/${bamboo.deploy.environment}.kubeconfig \
  set image deployment/shop-be shop-be="$IMAGE" -n shop
kubectl rollout status deployment/shop-be -n shop --timeout=300s
```

`${bamboo.deploy.release}` và `${bamboo.deploy.environment}` là biến Bamboo tự cung cấp trong deployment.

---

## 13. Specs as Code — YAML và Java

Click chuột trên UI tiện lúc đầu, nhưng khi có 50 plan thì "click-ops" thành ác mộng: không review được, không version control, khó nhân bản. **Bamboo Specs** giải quyết: định nghĩa plan bằng **code**, lưu trong repo Git, Bamboo tự đọc và tạo/cập nhật plan.

Có 2 loại:

| | **YAML Specs** | **Java Specs** |
|---|---|---|
| Viết bằng | YAML | Java (dùng thư viện `bamboo-specs`) |
| Độ phức tạp | Đơn giản, đủ 90% nhu cầu | Mạnh nhất, làm được mọi thứ UI làm được |
| Chạy ở đâu | Bamboo tự quét file `bamboo.yaml`/`bamboo.yml` trong repo | Chạy class Java (qua Maven) để "đẩy" cấu hình lên Bamboo |
| Phù hợp | Team mới, plan tiêu chuẩn | Plan phức tạp, logic điều kiện, template |

### 13.1. YAML Specs — ví dụ đầy đủ

**Bước 1:** Bật quét YAML: **Configure plan → Specs → YAML specs** → ✅ *Enable YAML Specs* (hoặc bật toàn cục ở ⚙️).

**Bước 2:** Đặt file `bamboo.yaml` ở **thư mục gốc repo**:

```yaml
---
version: 2

plan:
  project-key: SHOP
  key: BE
  name: Shop Backend CI

variables:
  maven.goals: clean package
  docker.registry: registry.congty.vn/shop-be

repositories:
  - shop-be:
      type: git
      url: https://bitbucket.org/team/shop-be.git
      branch: main

triggers:
  - bitbucket-server-trigger: {}   # webhook từ Bitbucket, không cần polling
  - polling:
      period: 180

stages:
  - Build:
      jobs:
        - Compile:
            tasks:
              - checkout:
                  description: Checkout code
                  repository: shop-be
              - maven:
                  description: Build with Maven
                  executable: Maven 3
                  jdk: JDK 17
                  goal: ${bamboo.maven.goals} -DskipTests
            artifacts:
              - name: app-jar
                location: target
                pattern: "*.jar"
                shared: true
            requirements:
              - capability: system.jdk.JDK 17

  - Test:
      jobs:
        - Unit Test:
            tasks:
              - checkout:
                  repository: shop-be
              - maven:
                  description: Run unit tests
                  executable: Maven 3
                  jdk: JDK 17
                  goal: test
                  test-parsers:
                    - junit:
                        test-results: "**/surefire-reports/*.xml"

  - Release:
      manual: true
      jobs:
        - Dockerize:
            tasks:
              - artifact-download:
                  description: Get app jar
                  source-plan: SHOP-BE
                  artifacts:
                    - name: app-jar
                      destination: deploy
              - script:
                  description: Build and push Docker image
                  interpreter: SHELL
                  scripts:
                    - docker build -t ${bamboo.docker.registry}:${bamboo.buildNumber} .
                    - docker push ${bamboo.docker.registry}:${bamboo.buildNumber}
```

**Bước 3:** Push file lên repo → Bamboo tự phát hiện (sau vài phút hoặc khi trigger chạy) → tạo/cập nhật plan `SHOP-BE`. Từ nay **sửa YAML = sửa plan**, có lịch sử Git và review qua pull request.

### 13.2. Java Specs — ví dụ đầy đủ

**Bước 1:** Tạo project Maven với dependency `bamboo-specs`:

```xml
<!-- pom.xml -->
<dependencies>
  <dependency>
    <groupId>com.atlassian.bamboo</groupId>
    <artifactId>bamboo-specs</artifactId>
    <version>9.6.0</version>
  </dependency>
</dependencies>
```

**Bước 2:** Viết class định nghĩa plan (dùng builder API — ví dụ rút gọn):

```java
package com.congty.specs;

import com.atlassian.bamboo.specs.api.BambooSpec;
import com.atlassian.bamboo.specs.api.builders.plan.*;
import com.atlassian.bamboo.specs.api.builders.repository.VcsRepository;
import com.atlassian.bamboo.specs.api.builders.task.*;
import com.atlassian.bamboo.specs.api.builders.trigger.*;
import com.atlassian.bamboo.specs.api.builders.requirement.Requirement;
import com.atlassian.bamboo.specs.api.model.BambooKey;
import com.atlassian.bamboo.specs.util.BambooServer;

@BambooSpec
public class ShopBackendPlan {

    public Plan plan() {
        return new Plan(new BambooKey("SHOP", "BE"), "Shop Backend CI")
            .description("CI cho backend Spring Boot")
            .planRepositories(new VcsRepository()
                .name("shop-be").type("git")
                .url("https://bitbucket.org/team/shop-be.git")
                .branch("main"))
            .triggers(new BitbucketServerTrigger(),
                new ScheduledTrigger().name("Nightly")
                    .cronExpression("0 0 2 ? * MON-FRI"))
            .stages(
                new Stage("Build").jobs(
                    new Job("Compile", new BambooKey("COMPILE"))
                        .tasks(
                            new CheckoutTask().description("Checkout code")
                                .checkoutItems(new CheckoutItem().repository("shop-be")),
                            new MavenTask().description("Build with Maven")
                                .executableLabel("Maven 3").jdk("JDK 17")
                                .goal("clean package -DskipTests"))
                        .artifacts(new com.atlassian.bamboo.specs.api.builders.Artifact("app-jar")
                            .location("target").copyPattern("*.jar").shared(true))
                        .requirements(new Requirement("system.jdk.JDK 17"))),
                new Stage("Test").jobs(
                    new Job("Unit Test", new BambooKey("UNIT"))
                        .tasks(
                            new CheckoutTask()
                                .checkoutItems(new CheckoutItem().repository("shop-be")),
                            new MavenTask().description("Run unit tests")
                                .executableLabel("Maven 3").jdk("JDK 17")
                                .goal("test"))),
                new Stage("Release").manual(true).jobs(
                    new Job("Dockerize", new BambooKey("DOCKER"))
                        .tasks(
                            new ScriptTask().description("Build and push image")
                                .interpreter(ScriptTaskProperties.Interpreter.SHELL)
                                .inlineBody(
                                    "docker build -t registry.congty.vn/shop-be:${bamboo.buildNumber} .\n" +
                                    "docker push registry.congty.vn/shop-be:${bamboo.buildNumber}"))));
    }

    public static void main(String[] args) {
        BambooServer server = new BambooServer("http://bamboo.congty.vn:8085");
        new ShopBackendPlan().plan().publish(server);
    }
}
```

**Bước 3:** Chạy để đẩy cấu hình lên Bamboo (dùng API token thay vì password):

```bash
mvn -q compile exec:java \
  -Dexec.mainClass=com.congty.specs.ShopBackendPlan \
  -Dbamboo.username=admin -Dbamboo.password=xxx
# Hoặc dùng token: -Dbamboo.token=xxx
```

> 💡 Java Specs mạnh ở chỗ dùng được **vòng lặp, điều kiện, hàm** — ví dụ sinh 10 plan cho 10 microservice từ 1 template, điều YAML không làm được.

### 13.3. Nên chọn loại nào?

**YAML** cho plan đơn giản (nhanh, dễ đọc); **Java** khi cần template/logic (sinh 10 plan từ 1 mẫu bằng vòng lặp — YAML không làm được). Quy tắc team: **mọi plan production phải có Specs trong repo**, cấm sửa tay trên UI.

---

## 14. Tích hợp Bitbucket, Jira, Confluence

### 14.1. Liên kết Bamboo ↔ Bitbucket (Application Link)

Bamboo **⚙️ → Application links** → *Create link* → nhập URL Bitbucket → wizard OAuth. Sau khi link, webhook Bamboo tự xuất hiện ở Bitbucket (*Repository settings → Webhooks*).

| Ở Bitbucket | Thấy gì |
|---|---|
| Pull request | Build status ✅/❌ của branch; có thể **chặn merge khi build đỏ** (*Branch permissions → Checks → Require passing builds*) |
| Commit / Branch list | Icon + trạng thái build mới nhất cạnh mỗi commit |

### 14.2. Tích hợp Jira — truy vết từ ticket tới production

1. **⚙️ → Application links** → link tới Jira.
2. Quy ước commit message: ghi **issue key** vào commit (`SHOP-123 Thêm API giỏ hàng`).

Kết quả: trong Jira issue, panel *Development* hiện commit/branch/PR/build ✅/❌; tab *Releases* cho biết issue đã deploy lên môi trường nào. Ngược lại, tab *Issues* của build liệt kê issue liên quan, click để nhảy sang Jira.

### 14.3. Confluence & Smart Commits

- Nhúng trạng thái build mới nhất vào trang tài liệu release bằng macro Bamboo/Jira trong Confluence.
- **Smart Commits** — commit message điều khiển cả Jira + Bamboo: `SHOP-123 #comment Đã fix lỗi #time 2h` (bình luận + log work 2 giờ vào issue).

---

## 15. Notifications — thông báo

### 15.1. Cấu hình ở cấp Plan

**Configure plan → Notifications → Add notification.** Event hay dùng: *Failed builds and first successful* (build đỏ + build xanh đầu tiên sau chuỗi đỏ — khuyên dùng), *All builds completed* (ồn, ít dùng).

**Cấu hình khuyên dùng cho mọi plan** — Event: *Failed builds and first successful*; Recipient: **Committer** (ai làm hỏng người đó nhận mail) + group team. Build đỏ → mail ngay; build xanh lại → mail "đã hồi phục".

### 15.2. Notification ở Deployment

**Deployment Project → <Environment> → Notifications**: *Deployment started / finished / failed*, *approved* — gửi cho release manager/on-call. Ví dụ: deploy Prod thất bại → mail/SMS ngay.

### 15.3. Webhook (Slack/Teams)

Recipient type **Webhook** → Bamboo POST JSON tới URL bạn định nghĩa (ví dụ Slack Incoming Webhook):

```json
{
  "planKey": "SHOP-BE",
  "buildNumber": "123",
  "buildResult": "FAILED",
  "buildUrl": "https://bamboo.congty.vn/browse/SHOP-BE-123"
}
```

→ Dùng Slack Workflow render tin nhắn đẹp, hoặc cài app "Bamboo for Slack" trên Marketplace.

---

## 16. Branch Plans & Pull Request Automation

Vấn đề: team 10 dev, mỗi người 1 branch `feature/*`. Nếu chỉ build `main`, lỗi chỉ lòi ra khi merge — quá muộn. **Branch plan** = Bamboo tự tạo plan con cho mỗi branch, build riêng từng branch.

### 16.1. Bật Branch Plan

**Configure plan → Branches → Create branch plan:**

| Tùy chọn | Khuyên dùng |
|---|---|
| *Branch pattern* | `(feature\|bugfix\|hotfix)/.*` — chỉ build branch theo quy ước |
| *Trigger* | Bitbucket Server repository triggered |
| ✅ *Branch merging* | Tự merge thử branch vào main trước khi build — phát hiện conflict sớm |

Kết quả: push lên `feature/gio-hang` → Bamboo tạo plan con → build → trạng thái hiện ngay trên **Pull Request** của Bitbucket (✅ cho merge / ❌ chặn merge nếu bật branch permission).

> ⚠️ Bật **Branches → Branch cleanup** (xóa plan sau 7 ngày inactive + khi branch bị xóa) — nếu không dashboard sẽ đầy plan con "ma".

---

## 17. Permissions & Security

### 17.1. Phân quyền theo cấp

| Cấp | Ở đâu | Quyền gồm |
|---|---|---|
| **Global** | ⚙️ → Security → Global permissions | Admin hệ thống, tạo plan, truy cập toàn cục |
| **Project** | Project → Actions → Permissions | View/Edit/Admin project, tạo plan trong project |
| **Plan** | Configure plan → Permissions | View/Build/Edit/Admin/Clone plan |
| **Deployment Environment** | Environment → Permissions | View/Deploy/Approve từng môi trường |
| **Agent** | ⚙️ → Agents → Agent permissions | Ai được gán dedicated agent |

**Ma trận khuyên dùng:**

| Vai trò | Project | Plan CI | Deploy Dev/Staging | Deploy Prod |
|---|---|---|---|---|
| Developer | View | View + Build | Deploy | View (không deploy) |
| QA | View | View | Deploy | View |
| Tech Lead | Edit | Edit + Build | Deploy + Approve | Approve |
| Release Manager | Admin | Admin | Deploy | Deploy + Approve |
| Khách hàng (stakeholder) | View | View | — | — |

### 17.2. Xác thực & SSO

- **Mặc định:** user/password trong Bamboo (user directory nội bộ).
- **LDAP/Active Directory:** ⚙️ → Security → User directories → *Add directory* → kết nối LDAP công ty → user đăng nhập bằng tài khoản công ty, phân nhóm theo LDAP group.
- **SSO (Crowd / SAML):** dùng **Atlassian Crowd** hoặc app SAML SSO trên Marketplace để 1 tài khoản cho Jira+Bitbucket+Bamboo.
- **API token:** user tạo token ở *Profile → API tokens* → dùng cho script/CI gọi REST API thay vì password.

### 17.3. Bảo mật cần làm ngay sau cài đặt

- [ ] Base URL HTTPS + redirect HTTP→HTTPS; tắt public signup (⚙️ → Security).
- [ ] Secret: dùng **Shared credentials**, không hardcode password trong script.
- [ ] Agent ↔ Server qua VPN nếu khác mạng; agent phải approve tay.
- [ ] Backup chứa credential → lưu nơi an toàn, mã hóa.
- [ ] Ít người có quyền Admin global; đăng ký mail Atlassian Security để nhận advisory.

### 17.4. Audit log

**⚙️ → Audit log** ghi ai đã làm gì (tạo/xóa plan, đổi permission, deploy prod...). Khi có sự cố "ai deploy bản này?", đây là nơi trả lời — không tắt.

---

## 18. Backup / Restore / Upgrade

### 18.1. Backup

**Cách 1 — Backup từ UI (đơn giản):**

1. **⚙️ → Backup** → *Create backup*.
2. Bamboo xuất file `.zip` gồm: database + cấu hình (không gồm artifacts — artifacts backup riêng).
3. Tải về / copy vào nơi an toàn.

**Cách 2 — Backup tự động theo lịch:**

**⚙️ → Scheduled backups** → đặt cron (ví dụ 2h sáng hàng ngày), chọn thư mục lưu:

```bash
# Thư mục backup trên server
/var/atlassian/bamboo-backup/
```

**Cách 3 — Backup "thủ công chuẩn DevOps" (khuyên dùng production):**

```bash
#!/bin/bash
# /opt/scripts/bamboo-backup.sh — chạy 2h sáng mỗi ngày qua cron
set -e
DATE=$(date +%Y%m%d)
BACKUP_DIR=/backup/bamboo

/opt/bamboo/bin/stop-bamboo.sh   # cold backup cho nhất quán
pg_dump -U bamboo -h localhost bamboodb | gzip > $BACKUP_DIR/bamboodb-$DATE.sql.gz
tar -czf $BACKUP_DIR/bamboo-home-$DATE.tar.gz /var/atlassian/bamboo-home
/opt/bamboo/bin/start-bamboo.sh
find $BACKUP_DIR -mtime +14 -delete   # giữ 14 ngày
echo "Backup xong: $DATE"
```

```bash
0 2 * * * /opt/scripts/bamboo-backup.sh >> /var/log/bamboo-backup.log 2>&1
```

**Checklist backup đầy đủ:**

- [ ] Database dump (quan trọng nhất)
- [ ] `bamboo-home/` (chứa `bamboo.cfg.xml`, specs cache...)
- [ ] Artifacts (nếu lưu local và cần giữ)
- [ ] File `bamboo-init.properties`, `setenv.sh` (cấu hình custom)
- [ ] Test restore định kỳ (backup không test restore = chưa backup)

### 18.2. Restore

1. Cài Bamboo **cùng version** với bản backup trên máy mới.
2. Restore DB dump vào database trống, giải nén `bamboo-home` đè lên home dir mới (sửa `bamboo.cfg.xml` nếu đổi đường dẫn).
3. Khởi động → kiểm tra plan/agent chạy thử.

### 18.3. Upgrade — quy trình an toàn

> ⚠️ Không upgrade thẳng production mà không backup + test.

1. **Backup đầy đủ**, đọc **Upgrade notes** của version mới (có đổi Java version không? Bamboo 8→9 cần Java 11→17).
2. Dừng Bamboo, giải nén bản mới ra thư mục mới (giữ bản cũ), copy `setenv.sh` + `bamboo-init.properties` sang.
3. Start bản mới → Bamboo tự migrate DB → login kiểm tra, chạy thử 1 plan.
4. Lỗi → rollback: dừng bản mới, trỏ lại bản cũ, restore DB.

**Zero-downtime upgrade** chỉ có ở Data Center (rolling từng node).

---

## 19. Pricing — Server / Data Center / Cloud

> ⚠️ Giá Atlassian thay đổi theo thời gian. Bảng dưới là **mức tham khảo** tại thời điểm viết (2026), tính theo USD/năm. Luôn kiểm tra giá mới nhất tại `atlassian.com/software/bamboo/pricing`.

**Cách tính license Bamboo: theo số REMOTE AGENT** (local agent không tính, unlimited plan/user/build).

| Gói | Remote agent | Giá tham khảo/năm | Ghi chú |
|---|---|---|---|
| **Bamboo Server** Starter | 1 | ~$10 | Team rất nhỏ, trial thực tế |
| Bamboo Server | 5 | ~$1,200 | Team nhỏ |
| Bamboo Server | 10–25 | ~$2,400–$4,800 | Team vừa |
| **Bamboo Data Center** | theo agent tier | cao hơn Server ~1.5–2x | HA, rolling upgrade |

- **Không có Bamboo Cloud** riêng lẻ — Atlassian dồn CI/CD cloud vào **Bitbucket Cloud Pipelines**. Bamboo = self-hosted (Server/DC).
- So nhanh: Jenkins miễn phí; GitHub Actions ~$0.008/phút; GitLab ~$29/user/tháng. Bamboo rẻ khi team đông + build nhiều (không tính theo phút/user).

**Mẹo tiết kiệm:** đếm đúng agent cần (job song song tối đa + 20% dự phòng); dùng elastic agent cho đợt cao điểm thay vì mua thêm remote agent cố định.

---

## 20. Troubleshooting

| Triệu chứng | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| Bamboo không start, `catalina.out` báo lỗi Java | Sai version Java (Bamboo 9 cần Java 17) | `java -version` → cài đúng JDK, sửa `JAVA_HOME` trong `setenv.sh` |
| `Database connection failed` lúc setup | Sai host/user/pass; DB chưa tạo; firewall chặn port 5432 | Test `psql -h host -U bamboo bamboodb`; kiểm tra `pg_hba.conf` |
| Port 8085 đã dùng | App khác chiếm port | `ss -tlnp \| grep 8085` → đổi port trong `conf/server.xml` |
| Build **Queued mãi** không chạy | Không agent nào thỏa requirement; agent offline/disabled; hết license agent | ⚙️ → Agents kiểm tra capability + trạng thái; xem *Agent Matrix* ở plan |
| Build fail `command not found: mvn/node/docker` | Agent thiếu tool | Cài tool lên máy agent → kiểm tra Capabilities |
| `OutOfMemoryError` khi build | Heap quá nhỏ | Tăng `JVM_MAXIMUM_MEMORY` (server); build Java thêm `MAVEN_OPTS=-Xmx2g` |
| Webhook Bitbucket không trigger | Bitbucket không gọi được sang Bamboo; base URL sai | Test `curl` từ máy Bitbucket; kiểm tra Base URL |
| Artifact download báo "not found" | Artifact chưa tick **Shared** | Kiểm tra job tạo artifact có Shared không |
| Deploy Prod bị chặn dù đã approve | Thiếu quyền ở environment | Environment → Permissions: cần cả quyền Deploy + Approve |
| Disk đầy | Artifacts/logs không expiry | ⚙️ → System → Storage; đặt artifact expiry |

**Câu thần chú debug build fail:** Mở build → **Logs** → tìm dòng đỏ đầu tiên từ dưới lên (`ERROR`, `FAILURE`, `BUILD FAILED`). 90% vấn đề nằm ở 10 dòng quanh đó.

---

## 21. Best Practices

### Thiết kế pipeline

1. **Fail fast:** stage đầu chạy compile + unit test nhanh (< 5 phút). Test nặng (E2E, performance) để stage sau.
2. **Song song hóa:** chia test thành nhiều job chạy song song — nhưng đếm agent cho đủ.
3. **Build deterministic:** dùng lockfile, Docker image tag cụ thể — cấm `latest` trong CI.
5. **Manual stage cho bước nguy hiểm:** deploy staging/prod trong plan thường → để manual.

### Vận hành

5. **Specs as Code cho mọi plan production:** cấm sửa tay trên UI, review qua PR.
6. **Agent:** production dùng remote agent, disable local agent.
7. **Artifact expiry:** đặt cho mọi artifact (30 ngày / 50 build) — disk đầy là sự cố "ngớ ngẩn" nhất.
8. **Backup + test restore** định kỳ hàng tháng; backup tự động hàng ngày.
9. **Tách server và agent** ra máy khác nhau; server sau reverse proxy + HTTPS.

### Bảo mật & quy trình

10. **Secret không vào repo/log:** dùng Shared credentials + biến secret.
11. **Approve cho Prod:** ít nhất 1 người duyệt; người deploy không tự duyệt.
12. **Rollback trong 5 phút:** mọi deployment phải deploy lại được bản cũ (giữ N release gần nhất).
13. **Branch protection:** chặn merge PR khi build đỏ + commit message có issue key.

### Văn hóa team

14. **"Ai làm đỏ, người đó sửa":** build đỏ là việc ưu tiên số 1.
15. **Build < 10 phút:** build chậm → dev gom commit to → càng dễ đỏ.

---

## 22. FAQ

**Hỏi: Bamboo khác gì Bitbucket Pipelines?**
Đáp: Pipelines là CI/CD **cloud** tính tiền theo phút build; Bamboo là **self-hosted**, mạnh về deployment/approval, hợp chính sách dữ liệu nội bộ.

**Hỏi: Có dùng Bamboo miễn phí được không?**
Đáp: Có bản **trial 30 ngày** đầy đủ tính năng. Hết trial phải mua license. Muốn miễn phí dài hạn → Jenkins.

**Hỏi: 1 agent chạy được mấy job cùng lúc?**
Đáp: **1 job tại 1 thời điểm.** Muốn song song → thêm agent (mỗi agent tốn 1 license remote agent).

**Hỏi: Bamboo có chạy được pipeline dạng code như Jenkinsfile không?**
Đáp: Có — **Bamboo Specs** (YAML hoặc Java), lưu trong repo, Bamboo tự đồng bộ (mục 13).

**Hỏi: Làm sao deploy 1 bản cũ (rollback)?**
Đáp: Vào Deployment Project → **Releases** → chọn release cũ → **Deploy** vào môi trường → xong trong 1 click (miễn là task deploy hỗ trợ).

**Hỏi: Bamboo có hỗ trợ Docker/Kubernetes không?**
Đáp: Có. Task Docker build/push có sẵn; deploy K8s thường dùng Script task + `kubectl` (ví dụ mục 12.5). Agent cần có Docker/Kubernetes CLI.

**Hỏi: Nên dùng polling hay webhook trigger?**
Đáp: Dùng Bitbucket + Bamboo cùng hệ thống → **webhook** (Bitbucket trigger) nhanh hơn. Repo Git ngoài (GitHub...) → **polling**.

**Hỏi: Quên mật khẩu admin thì sao?**
Đáp: Cách sạch nhất: restore từ backup còn nhớ pass. Hoặc dùng công cụ password recovery của Atlassian (xem doc "Restore passwords").

**Hỏi: Bamboo Server và Data Center khác nhau gì?**
Đáp: Data Center = chạy **cluster nhiều node** (HA, scale ngang, zero-downtime upgrade), giá cao hơn. Tính năng pipeline như nhau. Team < 100 dev thường Server là đủ.

---

## 23. Cheat Sheet

### Vòng đời nhanh

```bash
/opt/bamboo/bin/start-bamboo.sh   # khởi động
/opt/bamboo/bin/stop-bamboo.sh    # dừng
tail -f /opt/bamboo/logs/catalina.out
cd bamboo-agent-home/bin && ./bamboo-agent.sh start   # remote agent
```

### Biến hay dùng

```
${bamboo.buildNumber}              # số build
${bamboo.buildKey}                 # SHOP-BE-123
${bamboo.planRepository.branchName}# tên nhánh
${bamboo.repository.revision.number} # commit hash
${bamboo.deploy.release}           # tên release (trong deployment)
${bamboo.deploy.environment}       # tên môi trường (trong deployment)
```

### Cron trigger (Quartz)

```
0 0/15 * * * ?     mỗi 15 phút
0 0 2 ? * MON-FRI  2h sáng thứ 2-6
0 0 8 * * ?        8h sáng mỗi ngày
```

### REST API nhanh (cần API token)

```bash
BASE="https://bamboo.congty.vn/rest/api/latest"
TOKEN="xxx"

# Trigger build
curl -X POST -H "Authorization: Bearer $TOKEN" \
  "$BASE/queue/SHOP-BE"

# Xem trạng thái build mới nhất
curl -s -H "Authorization: Bearer $TOKEN" \
  "$BASE/result/SHOP-BE/latest.json" | head -c 500

# Liệt kê agent
curl -s -H "Authorization: Bearer $TOKEN" \
  "$BASE/agent?online=true"
```

### Checklist ra mắt Bamboo cho team mới

- [ ] Bamboo cài xong: HTTPS + base URL đúng, PostgreSQL, backup tự động
- [ ] Remote agent ≥ 2 (đủ JDK/Node/Docker), local agent disabled
- [ ] Plan CI đầu tiên chạy xanh, trigger webhook/polling hoạt động
- [ ] Notification: committer nhận mail khi build đỏ
- [ ] Branch plan cho `feature/*` + cleanup 7 ngày; chặn merge khi build đỏ
- [ ] Deployment: Dev auto → Staging auto → Prod manual + approve
- [ ] Jira: commit có issue key; permission theo ma trận; tắt public signup
- [ ] Test restore backup 1 lần

---

> **Tham khảo:** `docs.atlassian.com/bamboo` · `docs.atlassian.com/bamboo-specs` · `developer.atlassian.com/server/bamboo/rest`
