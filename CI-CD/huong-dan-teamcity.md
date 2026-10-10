# 🏗️ Hướng Dẫn Toàn Tập: TeamCity (CI/CD của JetBrains)

> **Cấp độ:** Người mới bắt đầu → cấu hình được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật lần cuối:** 10/10/2026
>
> ⚠️ **Ghi chú:** TeamCity có bản miễn phí (Professional) giới hạn 3 build agent và 100 build configuration. Phiên bản trong guide: TeamCity 2024.x.

---

## 📑 Mục Lục

1. [TeamCity Là Gì?](#1-teamcity-là-gì)
2. [TeamCity vs GitHub Actions vs Jenkins](#2-teamcity-vs-github-actions-vs-jenkins)
3. [Kiến Trúc: Server, Agent, Database](#3-kiến-trúc-server-agent-database)
4. [Cài Đặt TeamCity Server](#4-cài-đặt-teamcity-server)
5. [Lần Chạy Đầu Tiên & License](#5-lần-chạy-đầu-tiên--license)
6. [Tạo Project Đầu Tiên](#6-tạo-project-đầu-tiên)
7. [Build Configuration Chi Tiết](#7-build-configuration-chi-tiết)
8. [Build Steps Phổ Biến](#8-build-steps-phổ-biến)
9. [VCS Trigger & Schedule Trigger](#9-vcs-trigger--schedule-trigger)
10. [Cài Đặt Build Agent](#10-cài-đặt-build-agent)
11. [Kotlin DSL: settings.kts](#11-kotlin-dsl-settingskts)
12. [Artifacts & Dependencies](#12-artifacts--dependencies)
13. [Build Chains (Phụ Thuộc Giữa Các Build)](#13-build-chains-phụ-thuộc-giữa-các-build)
14. [Parameters: Configuration, System, Env](#14-parameters-configuration-system-env)
15. [Secrets & Password Parameters](#15-secrets--password-parameters)
16. [Clean Checkout & Build Caching](#16-clean-checkout--build-caching)
17. [Branch Builds & Pull Request](#17-branch-builds--pull-request)
18. [Thông Báo: Slack, Email, Webhook](#18-thông-báo-slack-email-webhook)
19. [REST API & TeamCity CLI](#19-rest-api--teamcity-cli)
20. [Dự Án Thực Tế: Node.js Full Pipeline](#20-dự-án-thực-tế-nodejs-full-pipeline)
21. [Dự Án Thực Tế: Docker Build & Push](#21-dự-án-thực-tế-docker-build--push)
22. [Pricing & License](#22-pricing--license)
23. [Troubleshooting](#23-troubleshooting)
24. [Best Practices Checklist](#24-best-practices-checklist)
25. [Cheat Sheet](#25-cheat-sheet)
26. [FAQ](#26-faq)

---

## 1. TeamCity Là Gì?

**TeamCity** là nền tảng CI/CD (Continuous Integration / Continuous Delivery) do **JetBrains** phát triển — cùng công ty làm ra IntelliJ IDEA, PyCharm, WebStorm. Ra mắt từ năm 2006, TeamCity là một trong những CI server lâu đời và ổn định nhất.

### TeamCity làm gì?

Mỗi khi bạn push code lên Git:

1. TeamCity **phát hiện** commit mới (qua VCS trigger)
2. **Checkout** code về một build agent
3. **Chạy** các bước bạn định nghĩa: install → lint → test → build
4. **Thu thập** artifacts (file build ra) và test reports
5. **Thông báo** kết quả (Slack/email)
6. Nếu thành công → **deploy** tiếp (build chain)

### Điểm mạnh đặc trưng

| Đặc điểm | Giải thích |
|----------|------------|
| **Kotlin DSL** | Toàn bộ cấu hình là code Kotlin trong `.teamcity/settings.kts` — version control được, review được |
| **Build Chains** | Nối nhiều build configuration thành pipeline phức tạp (build → test → deploy) |
| **Smart test running** | Chỉ chạy test bị ảnh hưởng bởi thay đổi (test history) |
| **On-the-fly config** | Sửa cấu hình trực tiếp trên UI, TeamCity tự sinh Kotlin DSL |
| **JetBrains ecosystem** | Tích hợp sâu với IntelliJ: chạy build từ IDE, xem kết quả ngay |

### Khi nào nên dùng TeamCity?

- Team đã quen hệ sinh thái JetBrains
- Cần **self-hosted** CI chạy trong mạng nội bộ (on-premise)
- Pipeline phức tạp nhiều bước phụ thuộc nhau
- Muốn cấu hình bằng code (Kotlin DSL) nhưng vẫn có UI trực quan

### Khi nào KHÔNG nên dùng?

- Project nhỏ, open source → GitHub Actions miễn phí và đơn giản hơn
- Muốn zero-maintenance → dùng cloud CI (CircleCI, Semaphore)
- Team không ai biết Kotlin (dù DSL khá dễ đọc)

---

## 2. TeamCity vs GitHub Actions vs Jenkins

| Tiêu chí | TeamCity | GitHub Actions | Jenkins |
|----------|----------|----------------|---------|
| Nhà phát triển | JetBrains | GitHub/Microsoft | Open source (CDF) |
| Self-hosted | ✅ (chính) | Có (self-hosted runner) | ✅ (chính) |
| Cloud hosted | ✅ (TeamCity Cloud) | ✅ (chính) | ❌ |
| Cấu hình | UI + Kotlin DSL | YAML | UI + Groovy (Jenkinsfile) |
| Miễn phí | 3 agents, 100 configs | 2.000 phút/tháng | Không giới hạn |
| Độ khó setup | Trung bình | Dễ | Khó (nhiều plugin) |
| Build chains | Rất mạnh | Matrix + needs | Pipeline stages |
| Tích hợp IDE | Tốt nhất (JetBrains) | Tốt (VS Code) | Trung bình |
| Plugin ecosystem | Vừa (~400) | Rất lớn (marketplace) | Khổng lồ (~1.800) |

---

## 3. Kiến Trúc: Server, Agent, Database

```
┌─────────────────────────────────────────────────┐
│              TEAMCITY SERVER                     │
│  ┌──────────┐  ┌──────────┐  ┌───────────────┐   │
│  │ Web UI   │  │ Build    │  │ VCS polling   │   │
│  │ :8111    │  │ Queue    │  │ (Git/SVN/...) │   │
│  └──────────┘  └──────────┘  └───────────────┘   │
│         │              │                         │
│         ▼              ▼                         │
│  ┌─────────────────────────────────────┐        │
│  │ Database (HSQLDB/PostgreSQL/MySQL)  │        │
│  │ Build history, artifacts metadata   │        │
│  └─────────────────────────────────────┘        │
└─────────────────────────────────────────────────┘
         │                    │
    ┌────┴────┐          ┌────┴────┐
    ▼         ▼          ▼         ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│Agent 1 │ │Agent 2 │ │Agent 3 │ │Agent N │
│Linux   │ │Windows │ │macOS   │ │Docker  │
└────────┘ └────────┘ └────────┘ └────────┘
  Chạy build thật: checkout code, chạy steps
```

- **Server:** điều phối, lưu lịch sử, giao diện web (port 8111)
- **Agent:** máy chạy build thật. Mỗi agent chạy 1 build tại 1 thời điểm (có thể cấu hình nhiều build runner song song)
- **Database:** lưu cấu hình, lịch sử build. Mặc định dùng HSQLDB embedded (chỉ cho test), production nên dùng **PostgreSQL**

---

## 4. Cài Đặt TeamCity Server

### Cách 1: Docker (khuyên dùng để thử)

```bash
# Tạo thư mục lưu dữ liệu (để không mất khi xóa container)
mkdir -p ~/teamcity/{datadir,logs}

# Chạy TeamCity Server
docker run -d --name teamcity-server \
  -v ~/teamcity/datadir:/data/teamcity_server/datadir \
  -v ~/teamcity/logs:/opt/teamcity/logs \
  -p 8111:8111 \
  jetbrains/teamcity-server:latest
```

Mở trình duyệt: `http://localhost:8111` → làm theo wizard setup.

### Cách 2: Docker Compose (server + PostgreSQL)

```yaml
# docker-compose.yml
version: "3.8"

services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: teamcity
      POSTGRES_USER: teamcity
      POSTGRES_PASSWORD: teamcity_secret_123
    volumes:
      - pgdata:/var/lib/postgresql/data
    restart: unless-stopped

  teamcity:
    image: jetbrains/teamcity-server:latest
    ports:
      - "8111:8111"
    volumes:
      - teamcity-data:/data/teamcity_server/datadir
      - teamcity-logs:/opt/teamcity/logs
    environment:
      TEAMCITY_DATA_PATH: /data/teamcity_server/datadir
    depends_on:
      - postgres
    restart: unless-stopped

volumes:
  pgdata:
  teamcity-data:
  teamcity-logs:
```

```bash
docker compose up -d
```

### Cách 3: Cài trực tiếp trên Linux

```bash
# 1. Cài Java 17+ (TeamCity cần JVM)
sudo apt update && sudo apt install -y openjdk-17-jre-headless

# 2. Tải TeamCity
wget https://download.jetbrains.com/teamcity/TeamCity-2024.07.tar.gz
tar xzf TeamCity-2024.07.tar.gz
cd TeamCity

# 3. Chạy server
bin/teamcity-server.sh start

# 4. Mở http://<server-ip>:8111
```

### Yêu cầu hệ thống (server)

| Quy mô | CPU | RAM | Disk |
|--------|-----|-----|------|
| Thử nghiệm (< 10 projects) | 2 cores | 4 GB | 50 GB |
| Vừa (10–50 projects) | 4 cores | 8 GB | 200 GB |
| Lớn (50+ projects) | 8+ cores | 16+ GB | 500 GB+ |

> 💡 **Lưu ý quan trọng:** Database mặc định (HSQLDB) chỉ dùng để thử. Production **bắt buộc** chuyển sang PostgreSQL hoặc MySQL, nếu không sẽ mất dữ liệu khi server crash.

### Cấu hình PostgreSQL cho TeamCity

1. Tải driver JDBC PostgreSQL, copy vào `<TeamCity Data Dir>/lib/jdbc/`
2. Restart server
3. Trong wizard setup → chọn **PostgreSQL** → nhập:
   - Host: `postgres` (hoặc IP)
   - Database: `teamcity`
   - User/Password: như đã tạo

---

## 5. Lần Chạy Đầu Tiên & License

1. Mở `http://localhost:8111`
2. **Database Setup:** chọn database (HSQLDB cho thử nhanh, PostgreSQL cho thật)
3. **License Agreement:** đọc và Accept
4. **Administrator Account:** tạo user admin đầu tiên
   - Username: `admin`
   - Đặt mật khẩu mạnh
5. Xong → vào dashboard chính

### Kích hoạt license miễn phí

- **Professional Server:** miễn phí, không cần key — giới hạn 3 build agents, 100 build configurations
- **Enterprise:** cần license key mua từ JetBrains — không giới hạn agent

Vào **Administration → License Keys** để xem trạng thái.

---

## 6. Tạo Project Đầu Tiên

### Bước 1: Tạo Project

1. Trên dashboard → **Create project** (hoặc Administration → Projects → Create project)
2. **From a repository URL:** dán URL Git repo, ví dụ:
   ```
   https://github.com/your-org/your-app.git
   ```
3. TeamCity tự detect repo, hỏi username/password (hoặc SSH key cho private repo)
4. Đặt tên project → **Create**

### Bước 2: TeamCity tự gợi ý Build Configuration

TeamCity sẽ scan repo và gợi ý build steps dựa trên file nó tìm thấy:

| File tìm thấy | Gợi ý |
|---------------|-------|
| `package.json` | Node.js: npm install → npm test |
| `pom.xml` | Maven: mvn test |
| `build.gradle` | Gradle: gradle build |
| `Dockerfile` | Docker build |
| `Makefile` | make |

Bạn có thể **dùng gợi ý** hoặc **tạo thủ công** (khuyên dùng thủ công để hiểu rõ).

### Bước 3: Chạy build đầu tiên

Bấm **Run** (nút ▶️) → TeamCity xếp vào queue → agent nào rảnh sẽ nhận → xem log realtime.

---

## 7. Build Configuration Chi Tiết

**Build Configuration** là đơn vị cơ bản nhất: một tập hợp các bước (steps) chạy trên code từ VCS.

### Các thành phần của một Build Configuration

```
Build Configuration: "MyApp - Build & Test"
├── General Settings (tên, mô tả, artifact paths)
├── Version Control Settings (VCS root nào, branch nào)
├── Build Steps (chạy lệnh gì, theo thứ tự)
├── Triggers (khi nào chạy: VCS trigger, schedule...)
├── Dependencies (phụ thuộc build config khác)
├── Parameters (biến môi trường, secrets)
├── Failure Conditions (khi nào coi là fail)
└── Build Features (thông báo, coverage, ...)
```

### Tạo thủ công (khuyên dùng)

1. Vào project → **Create build configuration** → **Manually**
2. Đặt tên: ví dụ `Build and Test`
3. **Version Control Settings** → Attach VCS Root (chọn repo đã kết nối)
4. **Build Steps** → Add build step → chọn Runner type

### Runner types phổ biến

| Runner | Dùng cho |
|--------|----------|
| Command Line | Chạy shell script / lệnh bất kỳ |
| Node.js | npm/yarn/pnpm scripts |
| Maven | `mvn` goals |
| Gradle | `gradle` tasks |
| Docker | build/push image |
| .NET | dotnet CLI |
| Python | chạy script Python |
| PowerShell | script Windows |

---

## 8. Build Steps Phổ Biến

### Ví dụ: Node.js project (3 steps)

**Step 1: Install dependencies**
- Runner type: `Node.js`
- Command: `npm`
- Arguments: `ci` (dùng `npm ci` thay vì `npm install` cho CI — nhanh và deterministic)

**Step 2: Lint**
- Runner type: `Node.js`
- Command: `npm`
- Arguments: `run lint --if-present`

**Step 3: Test**
- Runner type: `Node.js`
- Command: `npm`
- Arguments: `test`

### Ví dụ: Command Line (universal)

- Runner type: `Command Line`
- Run: `Custom script`
- Script:
  ```bash
  #!/bin/bash
  set -e  # dừng ngay khi có lệnh fail
  echo "Building..."
  npm ci
  npm run build
  ```

### Thứ tự chạy & điều kiện

Mỗi step có **Execute step**: 
- `Only if all previous steps were successful` (mặc định)
- `Even if some of the previous steps failed`
- `Only if build status is successful`

### Ví dụ: Python project

```
Step 1 (Command Line):
  python -m pip install -r requirements.txt

Step 2 (Command Line):
  python -m pytest tests/ --junitxml=test-results.xml

Step 3 (Command Line):
  python -m flake8 src/
```

---

## 9. VCS Trigger & Schedule Trigger

### VCS Trigger (chạy khi có commit mới)

1. Vào Build Configuration → **Triggers** → **Add new trigger**
2. Chọn **VCS Trigger**
3. Cấu hình:
   - **Branch filter:** `+:*` (tất cả branch) hoặc `+:main` (chỉ main)
   - **Quiet period:** đợi 60 giây sau commit cuối (gom nhiều commit push liên tiếp thành 1 build)

```
Branch Filter ví dụ:
  +:main              → chỉ branch main
  +:*                → mọi branch
  +:pull/*           → mọi branch pull/*
  -:experiment/*     → trừ branch experiment/*
```

### Schedule Trigger (chạy theo lịch)

- Chọn **Schedule Trigger**
- Cron expression, ví dụ:
  ```
  0 2 * * *    → 2h sáng mỗi ngày (nightly build)
  0 9 * * 1    → 9h sáng thứ 2 hàng tuần
  ```

### Trigger khác

| Trigger | Dùng khi |
|---------|----------|
| VCS Trigger | Có commit mới |
| Schedule Trigger | Chạy định kỳ |
| Finish Build Trigger | Chạy sau khi build config khác xong (nối chuỗi) |
| Retry Build Trigger | Tự retry khi build fail |

---

## 10. Cài Đặt Build Agent

Build agent là máy chạy build thật. Server mặc định KHÔNG chạy build — phải có ít nhất 1 agent.

### Cách 1: Agent qua Docker (nhanh nhất)

```bash
docker run -d --name teamcity-agent \
  -e SERVER_URL="http://<server-ip>:8111" \
  -v ~/agent-conf:/data/teamcity_agent/conf \
  jetbrains/teamcity-agent:latest
```

Agent sẽ tự kết nối về server → vào **Agents** → **Unauthorized** → bấm **Authorize**.

### Cách 2: Cài agent trực tiếp

```bash
# 1. Tải agent từ server (trong UI: Agents → Install Build Agents)
# Hoặc tải zip:
wget http://<server-ip>:8111/update/buildAgent.zip
unzip buildAgent.zip -d build-agent
cd build-agent

# 2. Cấu hình server URL
# Sửa file conf/buildAgent.properties:
# serverUrl=http://<server-ip>:8111

# 3. Chạy agent
bin/agent.sh start
```

### Agent requirements (điều kiện để agent nhận build)

Mỗi agent có thể khai báo **Configuration Parameters**, ví dụ:
```
system.agent.name = my-linux-agent
env.DOCKER = /usr/bin/docker
system.node.version = 20.11.0
```

Trong Build Configuration → **Agent Requirements** → thêm điều kiện:
```
Exists: env.DOCKER        → chỉ agent có Docker mới chạy build này
Equals: system.os.name = Linux
```

### Authorize agent

Agent mới kết nối ở trạng thái **Unauthorized** → admin phải bấm **Authorize** thì mới được nhận build (chống agent lạ).

---

## 11. Kotlin DSL: settings.kts

Đây là **tính năng mạnh nhất** của TeamCity: toàn bộ cấu hình lưu dưới dạng code Kotlin.

### Bật Versioned Settings

1. Vào Project Settings → **Versioned Settings**
2. **Synchronization enabled:** ✅
3. **Build settings in VCS:** chọn VCS Root
4. TeamCity sẽ commit file `.teamcity/settings.kts` vào repo

Từ nay, sửa cấu hình trên UI → TeamCity tự commit Kotlin DSL vào Git. Và ngược lại: sửa file `.teamcity/settings.kts` → push → TeamCity tự áp dụng!

### Ví dụ settings.kts cơ bản

```kotlin
// .teamcity/settings.kts
import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.buildSteps.script
import jetbrains.buildServer.configs.kotlin.triggers.vcs

version = "2024.07"

project {
    buildType(BuildAndTest)
}

object BuildAndTest : BuildType({
    name = "Build and Test"

    vcs {
        root(DslContext.settingsRoot)
    }

    steps {
        script {
            name = "Install dependencies"
            scriptContent = "npm ci"
        }
        script {
            name = "Run tests"
            scriptContent = "npm test"
        }
        script {
            name = "Build"
            scriptContent = "npm run build"
        }
    }

    triggers {
        vcs {
            branchFilter = "+:*"
        }
    }
})
```

### Ví dụ: Node.js với Docker

```kotlin
import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.buildSteps.dockerCommand
import jetbrains.buildServer.configs.kotlin.buildSteps.script

version = "2024.07"

project {
    buildType(DockerBuild)
}

object DockerBuild : BuildType({
    name = "Docker Build & Push"

    params {
        param("env.DOCKER_REGISTRY", "registry.example.com")
        param("env.IMAGE_NAME", "myapp")
        password("env.REGISTRY_PASSWORD", "credentialsJSON:docker-hub-password")
    }

    vcs {
        root(DslContext.settingsRoot)
    }

    steps {
        dockerCommand {
            name = "Build image"
            commandType = build {
                source = path {
                    path = "Dockerfile"
                }
                namesAndTags = "%env.DOCKER_REGISTRY%/%env.IMAGE_NAME%:%build.number%"
            }
        }
        dockerCommand {
            name = "Push image"
            commandType = push {
                namesAndTags = "%env.DOCKER_REGISTRY%/%env.IMAGE_NAME%:%build.number%"
            }
        }
    }
})
```

### Lợi ích của Kotlin DSL

- **Code review:** mọi thay đổi CI đều qua pull request
- **Tái sử dụng:** viết template một lần, dùng cho nhiều project
- **IDE support:** autocomplete, refactor trong IntelliJ
- **Lịch sử:** git blame biết ai sửa CI lúc nào

---

## 12. Artifacts & Dependencies

### Artifacts (file build tạo ra)

Trong Build Configuration → **General Settings** → **Artifact paths:**

```
# Cú pháp: <source> => <destination>
dist/** => dist.zip          # nén thư mục dist thành dist.zip
build/*.jar => jars/        # copy jar vào thư mục jars
coverage/** => coverage/    # báo cáo coverage
```

Ví dụ thực tế:
```
# Node.js
dist/** => frontend-dist.zip

# Java
target/*.jar => .

# Python wheel
dist/*.whl => .
```

Sau khi build xong, artifacts hiện trong tab **Artifacts** của build → tải về được.

### Artifact Dependencies (dùng artifact của build khác)

Build B cần file build ra từ Build A:

1. Vào Build B → **Dependencies** → **Add new artifact dependency**
2. Chọn Build A, nhập artifact rules:
   ```
   dist.zip => ./frontend/
   ```
3. TeamCity tự tải artifact của build A mới nhất về trước khi chạy build B

### Snapshot Dependencies

Đảm bảo build B chạy trên **đúng commit** mà build A đã chạy (không bị lệch code giữa 2 build).

---

## 13. Build Chains (Phụ Thuộc Giữa Các Build)

Build Chain nối nhiều Build Configuration thành pipeline:

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Compile  │───▶│   Test   │───▶│ Package  │───▶│  Deploy  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
  Build A         Build B         Build C         Build D
```

### Cách tạo

1. Build B → **Dependencies** → **Add new snapshot dependency** → chọn Build A
2. Build C → snapshot dependency → Build B
3. Build D → snapshot dependency → Build C

Khi chạy Build D, TeamCity tự chạy A → B → C → D theo thứ tự.

### Composite Build (build tổng hợp)

Tạo một Build Configuration kiểu **Composite** để gom nhóm:
- Không chạy steps thật, chỉ trigger các build con
- Hiển thị tổng quan cả chain trong một view

```kotlin
object FullPipeline : BuildType({
    name = "Full Pipeline"
    type = BuildTypeSettings.Type.COMPOSITE

    dependencies {
        snapshot(Compile) { onDependencyFailure = FailureAction.FAIL_TO_START }
        snapshot(Test) { onDependencyFailure = FailureAction.FAIL_TO_START }
        snapshot(Deploy) { onDependencyFailure = FailureAction.FAIL_TO_START }
    }
})
```

---

## 14. Parameters: Configuration, System, Env

TeamCity có 3 loại parameter:

| Loại | Tiền tố | Truyền vào build như | Ví dụ |
|------|---------|----------------------|-------|
| Configuration | (không) | `%param.name%` | `%env.NODE_ENV%` |
| System | `system.` | `-D` (Java) / system property | `%system.build.number%` |
| Environment | `env.` | Biến môi trường | `%env.API_URL%` |

### Định nghĩa parameters

Trong Build Configuration → **Parameters** → Add:

```
env.NODE_ENV = production
env.API_URL = https://api.example.com
system.docker.registry = registry.example.com
```

### Dùng trong build steps

```bash
# Trong script, dùng như biến môi trường thông thường
echo "Deploying to $API_URL"
docker build -t %system.docker.registry%/myapp:%build.number% .
```

### Parameter references (tham chiếu chéo)

```
env.FULL_IMAGE = %system.docker.registry%/myapp:%build.number%
```

### Predefined parameters hữu ích

| Parameter | Ý nghĩa |
|-----------|---------|
| `%build.number%` | Số build (tự tăng) |
| `%build.vcs.number%` | Commit hash |
| `%teamcity.build.branch%` | Tên branch |
| `%teamcity.build.checkoutDir%` | Thư mục checkout code |
| `%vcsroot.url%` | URL của repo |

---

## 15. Secrets & Password Parameters

**TUYỆT ĐỐI KHÔNG** ghi password/token trực tiếp vào script!

### Cách 1: Password parameter

1. Parameters → Add → Type: **Password**
2. Name: `env.DOCKER_PASSWORD`
3. Value: nhập password thật
4. TeamCity **che** giá trị trong log (hiện `******`)

### Cách 2: SSH Keys / Tokens trong Project

1. Project Settings → **SSH Keys** → Upload private key
2. Trong VCS Root → chọn SSH key đã upload

### Cách 3: Connections (khuyên dùng)

1. Project Settings → **Connections** → Add
2. Chọn loại: Docker Registry / AWS / GitHub / Slack...
3. Nhập credentials một lần → dùng lại cho nhiều build config

```kotlin
// Dùng connection trong DSL
features {
    dockerRegistryConnections {
        loginToRegistry = on {
            dockerRegistryId = "PROJECT_EXT_1"  // ID của connection
        }
    }
}
```

---

## 16. Clean Checkout & Build Caching

### Clean checkout

Mặc định TeamCity **không xóa** thư mục checkout giữa các build (để nhanh). Nếu build bị "bẩn" do file cũ:

1. Build Configuration → **Version Control Settings**
2. **Clean checkout:** ✅ (xóa hết và checkout lại từ đầu)

Hoặc bấm **... → Clean and run** khi chạy build thủ công.

### Build caching

**Node.js:**
```bash
# Cache node_modules giữa các build (trong agent)
# TeamCity không cache tự động như GitHub Actions
# Giải pháp: dùng agent có SSD nhanh, hoặc:
npm ci --prefer-offline --no-audit
```

**Gradle:**
```bash
# Bật Gradle build cache
./gradlew build --build-cache
```

**Docker:**
```bash
# Dùng layer cache
docker build --cache-from registry.example.com/myapp:latest .
```

---

## 17. Branch Builds & Pull Request

### Build mọi branch

1. VCS Trigger → Branch filter: `+:*`
2. TeamCity tự tạo build cho mỗi branch có commit mới
3. Trên UI có tab **Branches** để lọc theo branch

### Pull Request builds (GitHub)

1. Cài plugin **Commit Status Publisher** (có sẵn)
2. Build Features → Add → **Commit Status Publisher**
   - Publisher: GitHub
   - Chọn GitHub connection (token)
3. Mỗi PR sẽ hiện ✅/❌ ngay trên GitHub

### Branch specification (cho VCS Root)

```
+:refs/heads/*       → tất cả branches
+:refs/pull/*/head   → tất cả PRs (GitHub)
```

---

## 18. Thông Báo: Slack, Email, Webhook

### Email

1. Administration → **Email Notifier** → cấu hình SMTP
2. Mỗi user vào **My Settings** → chọn khi nào nhận email (build fail, build fixed...)

### Slack

1. Cài plugin **Slack Notifier** (JetBrains official)
2. Project Settings → **Connections** → Add Slack
3. Build Configuration → **Build Features** → Add **Slack Notifications**
   - Channel: `#ci-alerts`
   - Notify on: build failed, build successful

### Webhook (tùy chỉnh)

Dùng **WebHooks** plugin hoặc gọi curl trong step cuối:

```bash
# Gửi thông báo về webhook riêng
curl -X POST https://hooks.example.com/ci \
  -H "Content-Type: application/json" \
  -d "{\"build\": \"%build.number%\", \"status\": \"%teamcity.build.status%\"}"
```

---

## 19. REST API & TeamCity CLI

### REST API

```bash
# Lấy danh sách builds
curl -u "username:password" \
  "http://localhost:8111/app/rest/builds?locator=buildType:MyProject_Build"

# Trigger build mới
curl -u "username:password" -X POST \
  -H "Content-Type: application/xml" \
  -d "<build><buildType id='MyProject_Build'/></build>" \
  "http://localhost:8111/app/rest/buildQueue"

# Lấy trạng thái build
curl -u "username:password" \
  "http://localhost:8111/app/rest/builds/id:1234/status"
```

Dùng **Access Token** thay vì password (an toàn hơn):
1. User avatar → **Access Tokens** → Generate
2. Dùng header: `Authorization: Bearer <token>`

### TeamCity CLI (tcc)

```bash
# Cài đặt
# Tải từ: https://www.jetbrains.com/teamcity/download

# Đăng nhập
tcc login --server http://localhost:8111

# Xem builds
tcc builds list --build-type MyProject_Build

# Trigger build
tcc build start --build-type MyProject_Build
```

---

## 20. Dự Án Thực Tế: Node.js Full Pipeline

### Kịch bản

Repo Node.js + Express, pipeline: **Install → Lint → Test → Build → Docker Push**

### settings.kts hoàn chỉnh

```kotlin
// .teamcity/settings.kts
import jetbrains.buildServer.configs.kotlin.*
import jetbrains.buildServer.configs.kotlin.buildFeatures.commitStatusPublisher
import jetbrains.buildServer.configs.kotlin.buildSteps.dockerCommand
import jetbrains.buildServer.configs.kotlin.buildSteps.nodeJS
import jetbrains.buildServer.configs.kotlin.triggers.vcs

version = "2024.07"

project {
    buildType(BuildTest)
    buildType(DockerPush)

    params {
        param("env.REGISTRY", "registry.example.com")
        param("env.IMAGE", "my-node-app")
    }
}

object BuildTest : BuildType({
    name = "1. Build & Test"

    artifactRules = "dist/** => dist.zip"

    vcs {
        root(DslContext.settingsRoot)
        cleanCheckout = true
    }

    steps {
        nodeJS {
            name = "Install"
            shellScript = "npm ci"
        }
        nodeJS {
            name = "Lint"
            shellScript = "npm run lint"
        }
        nodeJS {
            name = "Test"
            shellScript = "npm test -- --ci --reporters=default --reporters=jest-teamcity"
        }
        nodeJS {
            name = "Build"
            shellScript = "npm run build"
        }
    }

    triggers {
        vcs {
            branchFilter = "+:*"
        }
    }

    features {
        commitStatusPublisher {
            publisher = github {
                githubUrl = "https://api.github.com"
                authType = personalToken {
                    token = "credentialsJSON:github-token"
                }
            }
        }
    }
})

object DockerPush : BuildType({
    name = "2. Docker Build & Push"

    // Chỉ chạy khi Build & Test thành công
    dependencies {
        snapshot(BuildTest) {
            onDependencyFailure = FailureAction.FAIL_TO_START
        }
        artifacts(BuildTest) {
            artifactRules = "dist.zip => ."
        }
    }

    vcs {
        root(DslContext.settingsRoot)
    }

    steps {
        dockerCommand {
            name = "Build image"
            commandType = build {
                source = path { path = "Dockerfile" }
                namesAndTags = "%env.REGISTRY%/%env.IMAGE%:%build.number%"
            }
        }
        dockerCommand {
            name = "Push image"
            commandType = push {
                namesAndTags = "%env.REGISTRY%/%env.IMAGE%:%build.number%"
            }
        }
    }

    // Chỉ push image khi ở branch main
    triggers {
        vcs {
            branchFilter = "+:main"
        }
    }
})
```

---

## 21. Dự Án Thực Tế: Docker Build & Push

### Dockerfile mẫu

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --omit=dev
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Build Configuration (UI)

| Step | Runner | Cấu hình |
|------|--------|----------|
| 1. Docker Build | Docker | `build` với tag `%env.REGISTRY%/%env.IMAGE%:%build.number%` |
| 2. Docker Push | Docker | `push` cùng tag |
| 3. Cleanup | Command Line | `docker rmi %env.REGISTRY%/%env.IMAGE%:%build.number%` |

### Agent Requirements

```
Exists: env.DOCKER    → agent phải có Docker
```

---

## 22. Pricing & License

| Gói | Giá | Giới hạn |
|-----|-----|----------|
| **Professional** | Miễn phí | 3 build agents, 100 build configurations |
| **Enterprise** | Liên hệ JetBrains (tính theo agent) | Không giới hạn agents/configs |
| **TeamCity Cloud** | Từ ~$45/tháng (tính theo credits) | Cloud-hosted, không cần maintain server |

> 💡 **Mẹo:** Bản Professional miễn phí đủ cho team nhỏ (< 10 người, ít project). Khi cần thêm agent thì mua thêm.

### So sánh chi phí self-hosted

```
Server (VPS 4GB RAM):     ~$20/tháng
3 agents (VPS 2GB mỗi):    ~$18/tháng
PostgreSQL (managed):     ~$15/tháng hoặc tự host miễn phí
─────────────────────────────────
Tổng: ~$35-50/tháng cho team nhỏ
```

---

## 23. Troubleshooting

| Vấn đề | Nguyên nhân | Cách sửa |
|--------|-------------|----------|
| Agent không kết nối được server | Sai `serverUrl` hoặc firewall | Kiểm tra `conf/buildAgent.properties`, mở port 8111 |
| Build stuck ở "Waiting for agent" | Không có agent compatible | Kiểm tra Agent Requirements, authorize agent |
| `OutOfMemoryError` trên server | Heap mặc định quá nhỏ | Tăng `TEAMCITY_SERVER_MEM_OPTS=-Xmx4g` |
| VCS checkout chậm | Repo lớn, không dùng shallow | Bật **Shallow clone** trong VCS Root |
| Build fail nhưng log không rõ | Thiếu `--verbose` | Thêm flag verbose vào lệnh build |
| Kotlin DSL không sync | Lỗi syntax trong settings.kts | Xem **Versioned Settings → Show diff**, sửa lỗi compile |
| Artifact quá lớn | Upload timeout | Tăng `teamcity.artifacts.streaming.thresholdMb` |
| Database HSQLDB corrupt | Server crash đột ngột | **Chuyển sang PostgreSQL ngay** |
| Agent unauthorized sau restart | Mất file config | Kiểm tra `conf/buildAgent.properties` còn không |
| Docker build "cannot connect" | Agent không mount docker.sock | Thêm `-v /var/run/docker.sock:/var/run/docker.sock` |

### Debug build fail

1. Vào build → tab **Overview** → xem step nào đỏ
2. Bấm vào step → xem **Build Log** chi tiết
3. Dùng **... → Re-run** để chạy lại
4. SSH vào agent (nếu self-hosted) để reproduce thủ công

---

## 24. Best Practices Checklist

- [ ] Dùng **Kotlin DSL** + Versioned Settings (mọi thay đổi CI đều qua Git)
- [ ] Database production dùng **PostgreSQL**, không dùng HSQLDB
- [ ] Secrets dùng **Password parameters** hoặc **Connections**, không hardcode
- [ ] Chia pipeline thành **Build Chain** (build → test → deploy riêng)
- [ ] Dùng **Snapshot dependencies** để đảm bảo cùng commit
- [ ] Bật **Commit Status Publisher** để hiện ✅/❌ trên GitHub/GitLab
- [ ] Agent có **requirements** rõ ràng (Docker, Node version...)
- [ ] Dùng `npm ci` thay vì `npm install` trong CI
- [ ] **Clean checkout** cho build release (tránh file cũ)
- [ ] Backup **Data Directory** + Database định kỳ
- [ ] Giới hạn **artifact size** (không upload node_modules!)
- [ ] Đặt **timeout** cho build (tránh build treo mãi)

---

## 25. Cheat Sheet

### Kotlin DSL nhanh

```kotlin
// Build step: script
script {
    name = "My step"
    scriptContent = "echo hello"
}

// Trigger: VCS
triggers {
    vcs { branchFilter = "+:main" }
}

// Trigger: schedule (cron)
triggers {
    schedule {
        schedulingPolicy = daily { hour = 2 }
        branchFilter = "+:main"
    }
}

// Parameter
params {
    param("env.MY_VAR", "value")
    password("env.SECRET", "credentialsJSON:my-secret-id")
}

// Artifact
artifactRules = "dist/** => dist.zip"

// Agent requirement
requirements {
    exists("env.DOCKER")
}
```

### REST API nhanh

```bash
# Trigger build
curl -u user:pass -X POST -H "Content-Type: application/xml" \
  -d "<build><buildType id='Project_Build'/></build>" \
  http://server:8111/app/rest/buildQueue

# Lấy build status
curl -u user:pass \
  http://server:8111/app/rest/builds/id:123/status
```

### Docker nhanh

```bash
# Server
docker run -d -p 8111:8111 -v tc-data:/data/teamcity_server/datadir \
  jetbrains/teamcity-server

# Agent
docker run -d -e SERVER_URL="http://server:8111" \
  jetbrains/teamcity-agent
```

---

## 26. FAQ

**Hỏi: TeamCity có miễn phí không?**
Đáp: Có, bản Professional miễn phí với 3 agents và 100 build configurations — đủ cho team nhỏ.

**Hỏi: Khác gì GitHub Actions?**
Đáp: TeamCity mạnh về self-hosted, Kotlin DSL, build chains phức tạp. GitHub Actions đơn giản hơn, tích hợp sẵn với GitHub, nhưng YAML kém expressive hơn Kotlin.

**Hỏi: Có chạy trên ARM (Apple Silicon) được không?**
Đáp: Được. Dùng image Docker multi-arch hoặc cài agent trực tiếp trên máy ARM.

**Hỏi: Mất bao lâu để setup xong?**
Đáp: Docker: ~15 phút có server chạy. Pipeline đầu tiên: ~30 phút nếu đã quen.

**Hỏi: Backup TeamCity như thế nào?**
Đáp: Backup 2 thứ: (1) **Data Directory** (`<TeamCity Data Dir>`), (2) **Database** (pg_dump nếu dùng PostgreSQL). Vào Administration → Backup để backup từ UI.

**Hỏi: Có tích hợp với GitLab không?**
Đáp: Có. VCS Root hỗ trợ GitLab, Commit Status Publisher hỗ trợ GitLab merge request.

---

## 27. Phụ Lục: Cấu Hình Nâng Cao

### Cloud Agents (tự động scale trên AWS/GCP)

Thay vì duy trì agent cố định, TeamCity có thể tự tạo agent trên cloud khi có build:

1. Administration → **Agent Clouds** → Create new cloud profile
2. Chọn **Amazon EC2**:
   - AMI ID: image có sẵn Docker + build tools
   - Instance type: `t3.medium`
   - Max instances: `5`
   - Region: `ap-southeast-1` (Singapore, gần VN nhất)
3. Khi queue có build chờ > 2 phút → TeamCity tự launch EC2 instance
4. Hết build → tự terminate (tiết kiệm tiền)

```kotlin
// Giới hạn cloud agent chỉ chạy build nặng
requirements {
    equals("system.agent.cloud", "true")
    exists("env.DOCKER")
}
```

### Build Queue Optimization

| Cấu hình | Ở đâu | Tác dụng |
|----------|-------|----------|
| Quiet period | VCS Trigger | Gom commit liên tiếp thành 1 build |
| Max running builds | Agent config | Giới hạn build song song trên 1 agent |
| Build queue priorities | Project settings | Ưu tiên project quan trọng |
| Personal builds | IDE plugin | Chạy thử trước khi commit (pre-tested commit) |

### Pre-tested Commit (Remote Run)

Chạy build **trước khi** commit lên Git:

1. Cài plugin TeamCity cho IntelliJ
2. Trong IDE: **Remote Run** → chọn changed files
3. TeamCity chạy build trên code của bạn (chưa commit)
4. Xanh → mới commit thật

```kotlin
// Cho phép personal build trong DSL
allowPersonalBuilds = true
```

### Test Intelligence

TeamCity tự động:
- **Phát hiện test mới fail** (so với lịch sử)
- **Mute** test hay fail flaky (tạm thời bỏ qua)
- **Chỉ chạy test liên quan** đến file thay đổi (dựa vào coverage history)

Bật trong: Build Configuration → **Build Features** → Add **Test Metadata**

### Docker Wrapper & Docker Compose

Chạy mỗi build step trong container riêng (môi trường sạch):

```kotlin
steps {
    script {
        name = "Test in container"
        scriptContent = "npm test"
        dockerImage = "node:20-alpine"
        dockerImagePlatform = ScriptBuildStep.ImagePlatform.Linux
    }
}
```

Hoặc dùng Docker Compose cho integration test:

```yaml
# docker-compose.test.yml
services:
  app:
    build: .
    depends_on:
      - postgres
      - redis
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: test
  redis:
    image: redis:7-alpine
```

```bash
# Trong build step:
docker compose -f docker-compose.test.yml up --abort-on-container-exit
docker compose -f docker-compose.test.yml down -v
```

### Multi-node Setup (High Availability)

Cho team lớn cần HA:

```
┌──────────────┐     ┌──────────────┐
│ Main Server  │────▶│  Secondary   │
│ (read-write) │     │  (read-only) │
└──────────────┘     └──────────────┘
       │                     │
       └────────┬────────────┘
                ▼
       ┌──────────────┐
       │  PostgreSQL  │  (shared database)
       │  + NFS/S3    │  (shared artifacts)
       └──────────────┘
```

1. Secondary node chỉ đọc (xem build, không trigger)
2. Khi main die → promote secondary lên main
3. Artifacts lưu trên S3/NFS share giữa 2 node

### Monitoring TeamCity

**Metrics quan trọng:**
- Build queue length (nếu > 5 liên tục → cần thêm agent)
- Average build duration (tăng đột biến → có vấn đề)
- Agent disk usage (agent đầy disk → build fail)

**Prometheus integration:**
1. Cài plugin **Prometheus Metrics**
2. Scrape endpoint: `http://server:8111/app/metrics`
3. Grafana dashboard mẫu có sẵn trong plugin docs

### Migration từ Jenkins

| Jenkins | TeamCity tương đương |
|---------|---------------------|
| Jenkinsfile (Groovy) | settings.kts (Kotlin DSL) |
| Pipeline stages | Build Chain (snapshot dependencies) |
| Jenkins agent/node | Build Agent |
| Credentials | Password parameters / Connections |
| Cron trigger | Schedule Trigger |
| `archiveArtifacts` | Artifact paths |
| Blue Ocean | TeamCity UI (có sẵn, đẹp hơn) |

**Quy trình migrate:**
1. Export Jenkinsfile → viết lại thành settings.kts
2. Tạo project mới trong TeamCity, bật Versioned Settings
3. Chạy song song 2 hệ thống 1-2 tuần
4. So sánh kết quả → cutover

### Tích Hợp Với Các Tool Khác

| Tool | Cách tích hợp |
|------|---------------|
| **SonarQube** | Build Feature "SonarQube" hoặc chạy `sonar-scanner` trong step |
| **Snyk** | Chạy `snyk test` trong Command Line step |
| **Jira** | Plugin "Jira Integration" — link issue trong commit message |
| **YouTrack** | Native integration (cùng nhà JetBrains) |
| **Kubernetes** | Deploy bằng `kubectl` trong step, hoặc ArgoCD |

### Ví Dụ: Tích Hợp SonarQube

```kotlin
steps {
    script {
        name = "SonarQube Analysis"
        scriptContent = """
            sonar-scanner \
              -Dsonar.projectKey=my-project \
              -Dsonar.sources=src \
              -Dsonar.host.url=%env.SONAR_URL% \
              -Dsonar.login=%env.SONAR_TOKEN%
        """.trimIndent()
    }
}
```

---

*Hết guide TeamCity. Chúc cậu build xanh lè!* 🟢
