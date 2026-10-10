# 🚁 Hướng Dẫn Toàn Tập: Drone CI

> **Cấp độ:** Người mới bắt đầu → cấu hình được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật lần cuối:** 10/10/2026
>
> ⚠️ **Ghi chú:** Drone là open source (Apache 2.0), self-hosted hoàn toàn. Phiên bản trong guide: Drone 2.x / 3.x.

---

## 📑 Mục Lục

1. [Drone CI Là Gì?](#1-drone-ci-là-gì)
2. [Drone vs GitHub Actions vs Jenkins](#2-drone-vs-github-actions-vs-jenkins)
3. [Kiến Trúc: Server, Runner, Docker](#3-kiến-trúc-server-runner-docker)
4. [Cài Đặt Drone Server (Docker Compose)](#4-cài-đặt-drone-server-docker-compose)
5. [GitHub OAuth Setup](#5-github-oauth-setup)
6. [Kích Hoạt Repository](#6-kích-hoạt-repository)
7. [.drone.yml: Cú Pháp Cơ Bản](#7-droneyml-cú-pháp-cơ-bản)
8. [Pipelines, Steps & Services](#8-pipelines-steps--services)
9. [Trigger: Branch, Event, Path](#9-trigger-branch-event-path)
10. [Secrets: Quản Lý An Toàn](#10-secrets-quản-lý-an-toàn)
11. [Drone CLI](#11-drone-cli)
12. [Runner: Docker, SSH, Kubernetes](#12-runner-docker-ssh-kubernetes)
13. [Autoscaling Runners](#13-autoscaling-runners)
14. [Plugins: Hệ Sinh Thái Drone](#14-plugins-hệ-sinh-thái-drone)
15. [Dự Án Thực Tế: Node.js Pipeline](#15-dự-án-thực-tế-nodejs-pipeline)
16. [Dự Án Thực Tế: Docker Build & Push](#16-dự-án-thực-tế-docker-build--push)
17. [Dự Án Thực Tế: Deploy Lên VPS](#17-dự-án-thực-tế-deploy-lên-vps)
18. [Monorepo & Matrix Builds](#18-monorepo--matrix-builds)
19. [Convert Từ GitHub Actions / GitLab CI](#19-convert-từ-github-actions--gitlab-ci)
20. [Backup & Maintenance](#20-backup--maintenance)
21. [Pricing](#21-pricing)
22. [Troubleshooting](#22-troubleshooting)
23. [Best Practices Checklist](#23-best-practices-checklist)
24. [Cheat Sheet](#24-cheat-sheet)
25. [FAQ](#25-faq)

---

## 1. Drone CI Là Gì?

**Drone** là nền tảng CI/CD **container-native** — mọi build step đều chạy trong Docker container. Được viết bằng Go, open source từ 2012, hiện thuộc sở hữu của Harness.

### Triết lý cốt lõi

> "Mọi thứ đều là container"

Khác với Jenkins (chạy lệnh trực tiếp trên agent) hay GitHub Actions (VM + container lai), Drone chạy **mọi step** trong container Docker riêng biệt:

```yaml
# .drone.yml
steps:
  - name: test
    image: node:20-alpine    # ← step này chạy trong container node:20
    commands:
      - npm ci
      - npm test

  - name: build
    image: golang:1.22      # ← step này chạy trong container golang:1.22
    commands:
      - go build -o app .
```

### Điểm mạnh

| Đặc điểm | Giải thích |
|----------|------------|
| **Container-native** | Môi trường build sạch, reproducible 100% |
| **Nhẹ** | Server + runner chỉ là vài container Docker |
| **YAML đơn giản** | `.drone.yml` dễ đọc hơn Jenkinsfile |
| **Plugins = Docker images** | Muốn thêm tính năng? Chỉ cần dùng image khác |
| **Multi-platform** | Hỗ trợ x86, ARM (Raspberry Pi OK!) |
| **Self-hosted** | Chạy hoàn toàn trong hạ tầng của bạn |

### Khi nào nên dùng Drone?

- Muốn self-hosted CI nhẹ, không muốn maintain Jenkins nặng nề
- Team đã quen Docker
- Cần chạy CI trên ARM (Raspberry Pi, Apple Silicon server)
- Muốn kiểm soát hoàn toàn dữ liệu (không gửi code ra ngoài)

### Khi nào KHÔNG nên dùng?

- Muốn zero-setup → dùng GitHub Actions / CircleCI
- Cần Windows builds phức tạp (Drone hỗ trợ nhưng ít phổ biến)
- Team không biết Docker

---

## 2. Drone vs GitHub Actions vs Jenkins

| Tiêu chí | Drone | GitHub Actions | Jenkins |
|----------|-------|----------------|---------|
| Ngôn ngữ | Go | (closed source) | Java |
| License | Apache 2.0 (open) | Proprietary | MIT |
| Self-hosted | ✅ (chính) | Có (runner) | ✅ (chính) |
| Cloud | ❌ (tự host) | ✅ (chính) | ❌ |
| Cấu hình | `.drone.yml` | `.github/workflows/*.yml` | Jenkinsfile (Groovy) |
| Môi trường build | Docker container | VM / container | Agent trực tiếp |
| Resource usage | Rất nhẹ | N/A (cloud) | Nặng (Java) |
| Plugin system | Docker images | Marketplace actions | ~1.800 plugins |
| Miễn phí | Không giới hạn | 2.000 phút/tháng | Không giới hạn |

---

## 3. Kiến Trúc: Server, Runner, Docker

```
┌─────────────────────────────────────────┐
│           DRONE SERVER                   │
│  ┌──────────┐  ┌────────────────────┐   │
│  │ Web UI   │  │ Scheduler          │   │
│  │ :80      │  │ (chia việc)        │   │
│  └──────────┘  └────────────────────┘   │
│  ┌────────────────────────────────┐    │
│  │ SQLite / PostgreSQL            │    │
│  │ (lưu build history)            │    │
│  └────────────────────────────────┘    │
└─────────────────────────────────────────┘
                  │ RPC (gRPC)
     ┌────────────┼────────────┐
     ▼            ▼            ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│ Runner  │ │ Runner  │ │ Runner  │
│ Docker  │ │ Docker  │ │ K8s     │
└─────────┘ └─────────┘ └─────────┘
     │ Mỗi step = 1 Docker container
     ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│node:20  │ │golang:  │ │plugins/ │
│container│ │container│ │docker   │
└─────────┘ └─────────┘ └─────────┘
```

- **Server:** web UI, webhook receiver, scheduler. Không chạy build!
- **Runner:** nhận việc từ server, chạy container Docker thật
- **Database:** SQLite mặc định (đủ cho team nhỏ), PostgreSQL cho lớn

---

## 4. Cài Đặt Drone Server (Docker Compose)

### Bước 1: Tạo Docker Compose

```yaml
# docker-compose.yml
version: "3.8"

services:
  drone-server:
    image: drone/drone:2
    container_name: drone-server
    ports:
      - "80:80"
    volumes:
      - drone-data:/data
    environment:
      # URL public của Drone server (quan trọng!)
      - DRONE_SERVER_HOST=drone.example.com
      - DRONE_SERVER_PROTO=https

      # GitHub OAuth (xem phần 5)
      - DRONE_GITHUB_CLIENT_ID=xxx
      - DRONE_GITHUB_CLIENT_SECRET=xxx

      # RPC secret: server và runner dùng chung để xác thực
      # Tạo bằng: openssl rand -hex 16
      - DRONE_RPC_SECRET=your-super-secret-rpc-key-here

      # Database (mặc định SQLite trong /data)
      - DRONE_DATABASE_DRIVER=sqlite3
      - DRONE_DATABASE_DATASOURCE=/data/database.sqlite

      # User admin đầu tiên (username GitHub)
      - DRONE_USER_CREATE=username:your-github-username,admin:true

      # Timezone
      - TZ=Asia/Ho_Chi_Minh
    restart: unless-stopped

  drone-runner:
    image: drone/drone-runner-docker:1
    container_name: drone-runner
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_HOST=drone-server
      - DRONE_RPC_SECRET=your-super-secret-rpc-key-here
      # Số pipeline chạy song song
      - DRONE_RUNNER_CAPACITY=4
      - TZ=Asia/Ho_Chi_Minh
    depends_on:
      - drone-server
    restart: unless-stopped

volumes:
  drone-data:
```

### Bước 2: Tạo RPC Secret

```bash
# Tạo secret ngẫu nhiên
openssl rand -hex 16
# Ví dụ output: a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6

# Thay vào cả DRONE_RPC_SECRET của server và runner
```

### Bước 3: Chạy

```bash
docker compose up -d

# Xem log
docker compose logs -f drone-server
```

### Bước 4: Reverse Proxy (Nginx + HTTPS)

Drone cần HTTPS để GitHub webhook hoạt động:

```nginx
# /etc/nginx/sites-available/drone
server {
    listen 80;
    server_name drone.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name drone.example.com;

    ssl_certificate /etc/letsencrypt/live/drone.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/drone.example.com/privkey.pem;

    location / {
        proxy_pass http://localhost:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support (cho log realtime)
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

```bash
# Cấp chứng chỉ Let's Encrypt
sudo certbot --nginx -d drone.example.com
```

### Yêu cầu hệ thống

| Thành phần | CPU | RAM | Disk |
|------------|-----|-----|------|
| Server | 1 core | 1 GB | 10 GB |
| Runner (mỗi) | 2 cores | 4 GB | 50 GB (cho Docker images) |

> 💡 Server Drone rất nhẹ — chạy tốt trên VPS $5/tháng!

---

## 5. GitHub OAuth Setup

Drone dùng GitHub để đăng nhập và nhận webhook.

### Bước 1: Tạo GitHub OAuth App

1. GitHub → **Settings** → **Developer settings** → **OAuth Apps** → **New OAuth App**
2. Điền:
   - **Application name:** `Drone CI`
   - **Homepage URL:** `https://drone.example.com`
   - **Authorization callback URL:** `https://drone.example.com/login`
3. Bấm **Register application**
4. Copy **Client ID** → Generate **Client Secret**

### Bước 2: Cấu hình Drone

```yaml
environment:
  - DRONE_GITHUB_CLIENT_ID=<client-id>
  - DRONE_GITHUB_CLIENT_SECRET=<client-secret>
```

```bash
docker compose up -d --force-recreate drone-server
```

### Bước 3: Đăng nhập

1. Mở `https://drone.example.com`
2. Bấm **Login with GitHub**
3. Authorize → vào dashboard

### Các SCM khác được hỗ trợ

| SCM | Biến môi trường |
|-----|-----------------|
| GitHub | `DRONE_GITHUB_CLIENT_ID/SECRET` |
| GitLab | `DRONE_GITLAB_SERVER`, `DRONE_GITLAB_CLIENT_ID/SECRET` |
| Gitea | `DRONE_GITEA_SERVER`, `DRONE_GITEA_CLIENT_ID/SECRET` |
| Bitbucket | `DRONE_BITBUCKET_CLIENT_ID/SECRET` |

---

## 6. Kích Hoạt Repository

1. Đăng nhập Drone → bấm **+** hoặc vào **Repositories**
2. Tìm repo cần CI → bấm **Activate**
3. Cấu hình:
   - **Protected:** chỉ chạy build trên branch protected (khuyên bật)
   - **Trusted:** cho phép mount Docker socket (cần cho Docker-in-Docker, chỉ bật cho repo tin cậy!)
4. Drone tự tạo webhook trên GitHub

### Tạo file .drone.yml

Tạo file `.drone.yml` ở root của repo:

```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: hello
    image: alpine:3.19
    commands:
      - echo "Drone hoạt động rồi!"
```

Push → Drone tự chạy build!

---

## 7. .drone.yml: Cú Pháp Cơ Bản

### Cấu trúc tối thiểu

```yaml
kind: pipeline        # loại pipeline (bắt buộc)
type: docker         # chạy trên Docker runner (bắt buộc)
name: default        # tên pipeline (bắt buộc)

steps:               # các bước (bắt buộc)
  - name: build      # tên step (bắt buộc)
    image: node:20   # Docker image (bắt buộc)
    commands:        # lệnh chạy trong container
      - npm ci
      - npm run build
```

### Các trường quan trọng

| Trường | Cấp độ | Ý nghĩa |
|--------|--------|---------|
| `kind` | top | `pipeline` hoặc `secret` |
| `type` | top | `docker`, `kubernetes`, `ssh`, `exec` |
| `name` | top | Tên pipeline |
| `steps` | top | Danh sách các bước |
| `services` | top | Container phụ (database, redis...) |
| `trigger` | top | Điều kiện chạy |
| `depends_on` | top | Phụ thuộc pipeline khác |
| `node` | top | Chọn runner theo label |
| `image` | step | Docker image |
| `commands` | step | Lệnh shell |
| `environment` | step | Biến môi trường |
| `when` | step | Điều kiện chạy step |

### Ví dụ đầy đủ

```yaml
kind: pipeline
type: docker
name: ci

# Chỉ chạy khi push lên main hoặc PR
trigger:
  branch:
    - main
    - develop
  event:
    - push
    - pull_request

steps:
  - name: install
    image: node:20-alpine
    commands:
      - npm ci

  - name: lint
    image: node:20-alpine
    commands:
      - npm run lint
    depends_on: [install]

  - name: test
    image: node:20-alpine
    commands:
      - npm test
    depends_on: [install]

  - name: build
    image: node:20-alpine
    commands:
      - npm run build
    depends_on: [lint, test]
```

---

## 8. Pipelines, Steps & Services

### Multiple Pipelines trong 1 file

```yaml
---
kind: pipeline
type: docker
name: frontend

steps:
  - name: build-frontend
    image: node:20-alpine
    commands:
      - cd frontend && npm ci && npm run build

---
kind: pipeline
type: docker
name: backend

steps:
  - name: build-backend
    image: golang:1.22-alpine
    commands:
      - cd backend && go build -o app .

---
kind: pipeline
type: docker
name: notify

# Chạy sau khi cả frontend và backend xong
depends_on: [frontend, backend]

steps:
  - name: slack
    image: plugins/slack
    settings:
      webhook:
        from_secret: slack_webhook
      channel: ci-alerts
```

### Services (container phụ)

Dùng database/redis trong lúc test:

```yaml
kind: pipeline
type: docker
name: test-with-db

services:
  - name: postgres
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: testdb

  - name: redis
    image: redis:7-alpine

steps:
  - name: test
    image: node:20-alpine
    environment:
      DATABASE_URL: postgres://test:test@postgres:5432/testdb
      REDIS_URL: redis://redis:6379
    commands:
      - npm ci
      - npm test
```

> 💡 Services có thể gọi nhau bằng **tên** (ví dụ `postgres`, `redis`) như hostname.

### Volumes (chia sẻ file giữa steps)

```yaml
steps:
  - name: build
    image: node:20-alpine
    volumes:
      - name: cache
        path: /app/node_modules
    commands:
      - npm ci

  - name: test
    image: node:20-alpine
    volumes:
      - name: cache
        path: /app/node_modules
    commands:
      - npm test

volumes:
  - name: cache
    temp: {}
```

---

## 9. Trigger: Branch, Event, Path

### Trigger theo branch

```yaml
trigger:
  branch:
    # Chỉ chạy trên main
    - main

    # Chạy trên main và mọi branch feature/*
    - main
    - feature/*

    # Trừ branch experiment
    exclude:
      - experiment/*
```

### Trigger theo event

```yaml
trigger:
  event:
    - push           # push code
    - pull_request   # mở/cập nhật PR
    - tag            # push tag (dùng cho release)
    - cron           # chạy định kỳ
    - manual         # chạy thủ công từ UI
```

### Trigger theo path (monorepo)

```yaml
# Chỉ chạy khi file trong frontend/ thay đổi
trigger:
  paths:
    include:
      - frontend/**
    exclude:
      - frontend/**/*.md
```

### Kết hợp

```yaml
trigger:
  branch:
    - main
  event:
    - push
  paths:
    include:
      - src/**
```

### Cron jobs

1. Trong Drone UI → repo → **Settings** → **Cron Jobs** → Add
2. Đặt tên + schedule (cron expression) + branch
3. Trong `.drone.yml`:

```yaml
trigger:
  event:
    - cron

steps:
  - name: nightly-test
    image: node:20-alpine
    commands:
      - npm run test:e2e
```

---

## 10. Secrets: Quản Lý An Toàn

**TUYỆT ĐỐI KHÔNG** ghi secrets trực tiếp vào `.drone.yml`!

### Tạo secret

**Cách 1: Drone UI**
1. Repo → **Settings** → **Secrets** → Add
2. Name: `docker_password`, Value: nhập secret

**Cách 2: Drone CLI**
```bash
drone secret add --repository octocat/hello-world \
  --name docker_password \
  --data supersecret123
```

### Dùng secret trong pipeline

```yaml
steps:
  - name: docker-login
    image: plugins/docker
    settings:
      repo: myapp
      username:
        from_secret: docker_username
      password:
        from_secret: docker_password
```

### Biến môi trường từ secret

```yaml
steps:
  - name: deploy
    image: alpine:3.19
    environment:
      API_KEY:
        from_secret: api_key
    commands:
      - echo "Deploying with key $API_KEY"
```

### Organization secrets (dùng chung)

```bash
# Tạo secret cho cả organization
drone orgsecret add --namespace myorg \
  --name shared_token \
  --data xxx
```

### Lưu ý bảo mật

- Secrets **không** hiện trong log (tự che thành `******`)
- PR từ fork **không** được truy cập secrets (mặc định)
- Muốn cho PR dùng secrets: bật **Trusted** (chỉ cho repo tin cậy!)

---

## 11. Drone CLI

### Cài đặt

```bash
# Linux
curl -L https://github.com/harness/drone-cli/releases/latest/download/drone_linux_amd64.tar.gz | tar zx
sudo install -t /usr/local/bin drone

# macOS
brew install drone-cli

# Kiểm tra
drone --version
```

### Cấu hình

```bash
export DRONE_SERVER=https://drone.example.com
export DRONE_TOKEN=your-api-token  # lấy trong Drone UI → Account Settings

# Lưu vĩnh viễn
echo 'export DRONE_SERVER=https://drone.example.com' >> ~/.bashrc
echo 'export DRONE_TOKEN=xxx' >> ~/.bashrc
```

### Lệnh thường dùng

```bash
# Xem build gần nhất
drone build ls octocat/hello-world

# Xem chi tiết build
drone build info octocat/hello-world 42

# Xem log
drone build logs octocat/hello-world 42

# Trigger build thủ công
drone build create octocat/hello-world

# Dừng build đang chạy
drone build stop octocat/hello-world 42

# Restart build
drone build restart octocat/hello-world 42

# Quản lý secrets
drone secret add --repository octocat/hello-world --name my_secret --data xxx
drone secret ls --repository octocat/hello-world
drone secret rm --repository octocat/hello-world --name my_secret

# Validate file .drone.yml (không cần server!)
drone lint
drone jsonnet  # nếu dùng jsonnet

# Chạy pipeline local (cần drone-runner-exec)
drone exec --pipeline=default
```

### Chạy pipeline local (không cần server)

```bash
# Cài exec runner
# Thêm vào .drone.yml rồi chạy:
drone exec
```

> 💡 `drone lint` rất hữu ích để kiểm tra syntax trước khi push!

---

## 12. Runner: Docker, SSH, Kubernetes

### Docker Runner (phổ biến nhất)

Đã cấu hình ở phần 4. Mỗi step = 1 container.

```yaml
# docker-compose.yml - thêm runner thứ 2
  drone-runner-2:
    image: drone/drone-runner-docker:1
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_HOST=drone-server
      - DRONE_RPC_SECRET=xxx
      - DRONE_RUNNER_CAPACITY=2
      - DRONE_RUNNER_NAME=runner-2
```

### SSH Runner (chạy lệnh trên server remote)

```yaml
# docker-compose.yml
  drone-runner-ssh:
    image: drone/drone-runner-ssh:1
    environment:
      - DRONE_RPC_PROTO=http
      - DRONE_RPC_HOST=drone-server
      - DRONE_RPC_SECRET=xxx
```

```yaml
# .drone.yml
kind: pipeline
type: ssh
name: deploy

server:
  host:
    from_secret: ssh_host
  user: deploy
  ssh_key:
    from_secret: ssh_key

steps:
  - name: deploy
    commands:
      - cd /opt/myapp
      - git pull
      - docker compose up -d --build
```

### Kubernetes Runner

```yaml
# .drone.yml
kind: pipeline
type: kubernetes
name: default

steps:
  - name: test
    image: node:20-alpine
    commands:
      - npm ci
      - npm test
```

### Exec Runner (chạy trực tiếp trên host)

Dùng khi cần chạy không qua Docker (ví dụ: build iOS trên macOS):

```bash
# Cài trên máy macOS
# Tải binary drone-runner-exec
```

```yaml
kind: pipeline
type: exec
name: ios-build

platform:
  os: darwin
  arch: arm64

steps:
  - name: build
    commands:
      - xcodebuild -scheme MyApp build
```

---

## 13. Autoscaling Runners

Khi nhiều build cùng lúc, cần tự động thêm runner.

### AWS Autoscaler

```yaml
# docker-compose.yml
  drone-autoscaler:
    image: drone/drone-autoscaler:1
    environment:
      - DRONE_SERVER_PROTO=http
      - DRONE_SERVER_HOST=drone-server
      - DRONE_SERVER_TOKEN=xxx  # API token

      # AWS config
      - DRONE_AMAZON_INSTANCE=t3.medium
      - DRONE_AMAZON_REGION=ap-southeast-1
      - DRONE_AMAZON_AMI=ami-xxx  # AMI có Docker + runner
      - DRONE_AMAZON_KEY_FILE=/root/.ssh/aws.pem

      # Giới hạn
      - DRONE_AGENT_MIN_COUNT=1
      - DRONE_AGENT_MAX_COUNT=10
      # Tự tắt máy khi không có việc sau 10 phút
      - DRONE_AGENT_IDLE_TIMEOUT=600
    volumes:
      - ~/.ssh/aws.pem:/root/.ssh/aws.pem:ro
```

### Hetzner Autoscaler (rẻ hơn AWS)

Dùng plugin community `drone-hetzner-autoscaler` — tương tự nhưng dùng Hetzner Cloud (rẻ hơn nhiều cho team nhỏ).

---

## 14. Plugins: Hệ Sinh Thái Drone

Plugins trong Drone chỉ là **Docker images** — bạn có thể tự viết plugin bằng bất kỳ ngôn ngữ nào!

### Plugins chính thức phổ biến

| Plugin | Image | Dùng để |
|--------|-------|---------|
| Docker | `plugins/docker` | Build & push Docker image |
| Slack | `plugins/slack` | Gửi thông báo Slack |
| Email | `plugins/email` | Gửi email |
| S3 | `plugins/s3` | Upload lên S3 |
| Gitea Release | `plugins/gitea-release` | Tạo release |
| GitHub Release | `plugins/github-release` | Tạo GitHub release |
| SSH | `plugins/ssh` | Chạy lệnh qua SSH |
| Kaniko | `plugins/kaniko` | Build Docker không cần privileged |

### Ví dụ: Docker plugin

```yaml
steps:
  - name: build-and-push
    image: plugins/docker
    settings:
      repo: registry.example.com/myapp
      tags:
        - latest
        - ${DRONE_COMMIT_SHA:0:8}
      username:
        from_secret: docker_username
      password:
        from_secret: docker_password
      # Build multi-arch
      platform: linux/amd64,linux/arm64
```

### Ví dụ: Slack plugin

```yaml
steps:
  - name: notify
    image: plugins/slack
    settings:
      webhook:
        from_secret: slack_webhook
      channel: ci-alerts
      template: |
        {{#success build.status}}
          ✅ Build #{{build.number}} thành công!
        {{else}}
          ❌ Build #{{build.number}} thất bại!
        {{/success}}
        Repo: {{repo.name}}
        Branch: {{build.branch}}
        Commit: {{build.message}}
    when:
      status: [success, failure]
```

### Tự viết plugin

```dockerfile
# Dockerfile của plugin tự viết
FROM alpine:3.19
COPY my-script.sh /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/my-script.sh"]
```

```yaml
steps:
  - name: my-plugin
    image: my-registry/my-plugin:latest
    settings:
      my_option: value
```

Plugin nhận settings qua biến môi trường `PLUGIN_MY_OPTION`.

---

## 15. Dự Án Thực Tế: Node.js Pipeline

### Kịch bản

App Node.js + Express + PostgreSQL. Pipeline: **Test → Build → Push Docker → Deploy**

### .drone.yml hoàn chỉnh

```yaml
kind: pipeline
type: docker
name: ci

trigger:
  branch:
    - main
  event:
    - push
    - pull_request

services:
  - name: postgres
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: testdb

steps:
  - name: install
    image: node:20-alpine
    commands:
      - npm ci

  - name: lint
    image: node:20-alpine
    commands:
      - npm run lint
    depends_on: [install]

  - name: test
    image: node:20-alpine
    environment:
      DATABASE_URL: postgres://test:test@postgres:5432/testdb
      NODE_ENV: test
    commands:
      - npm test -- --ci
    depends_on: [install]

  - name: build
    image: node:20-alpine
    commands:
      - npm run build
    depends_on: [lint, test]

---
kind: pipeline
type: docker
name: cd

# Chỉ chạy khi CI xong và ở branch main (không chạy cho PR)
trigger:
  branch:
    - main
  event:
    - push

depends_on: [ci]

steps:
  - name: build-and-push
    image: plugins/docker
    settings:
      repo: registry.example.com/myapp
      tags:
        - latest
        - ${DRONE_COMMIT_SHA:0:8}
      username:
        from_secret: docker_username
      password:
        from_secret: docker_password

  - name: deploy
    image: appleboy/drone-ssh
    settings:
      host:
        from_secret: ssh_host
      username: deploy
      key:
        from_secret: ssh_key
      script:
        - docker pull registry.example.com/myapp:latest
        - docker stop myapp || true
        - docker rm myapp || true
        - docker run -d --name myapp -p 3000:3000 registry.example.com/myapp:latest

  - name: notify-success
    image: plugins/slack
    settings:
      webhook:
        from_secret: slack_webhook
      channel: deployments
      template: "🚀 Deployed {{repo.name}}#{{build.number}} thành công!"
    when:
      status: [success]

  - name: notify-failure
    image: plugins/slack
    settings:
      webhook:
        from_secret: slack_webhook
      channel: ci-alerts
      template: "❌ Deploy {{repo.name}}#{{build.number}} thất bại!"
    when:
      status: [failure]
```

---

## 16. Dự Án Thực Tế: Docker Build & Push

### Multi-arch build (AMD64 + ARM64)

```yaml
kind: pipeline
type: docker
name: docker-multiarch

trigger:
  event:
    - tag

steps:
  - name: build
    image: plugins/docker
    settings:
      repo: myuser/myapp
      tags:
        - ${DRONE_TAG}
        - latest
      platform: linux/amd64,linux/arm64
      username:
        from_secret: docker_username
      password:
        from_secret: docker_password
```

### Dùng Kaniko (không cần privileged Docker)

```yaml
steps:
  - name: build
    image: plugins/kaniko
    settings:
      repo: myuser/myapp
      tags: ${DRONE_COMMIT_SHA:0:8}
      dockerfile: Dockerfile
      username:
        from_secret: docker_username
      password:
        from_secret: docker_password
```

---

## 17. Dự Án Thực Tế: Deploy Lên VPS

### Cách 1: SSH + Docker Compose

```yaml
kind: pipeline
type: docker
name: deploy-vps

trigger:
  branch: [main]
  event: [push]

steps:
  - name: deploy
    image: appleboy/drone-ssh
    settings:
      host:
        from_secret: vps_host
      username:
        from_secret: vps_user
      key:
        from_secret: vps_ssh_key
      port: 22
      script:
        - cd /opt/myapp
        - git pull origin main
        - docker compose pull
        - docker compose up -d --build
        - docker image prune -f
```

### Cách 2: Webhook deploy

```yaml
steps:
  - name: trigger-deploy
    image: curlimages/curl
    commands:
      - |
        curl -X POST https://deploy.example.com/hook \
          -H "Authorization: Bearer $DEPLOY_TOKEN" \
          -d '{"image": "myapp:${DRONE_COMMIT_SHA:0:8}"}'
    environment:
      DEPLOY_TOKEN:
        from_secret: deploy_token
```

---

## 18. Monorepo & Matrix Builds

### Monorepo: trigger theo path

```yaml
---
kind: pipeline
type: docker
name: frontend

trigger:
  paths:
    include: [frontend/**]

steps:
  - name: build
    image: node:20-alpine
    commands:
      - cd frontend && npm ci && npm run build

---
kind: pipeline
type: docker
name: backend

trigger:
  paths:
    include: [backend/**]

steps:
  - name: build
    image: golang:1.22-alpine
    commands:
      - cd backend && go build -o app .
```

### Matrix builds (test nhiều version)

```yaml
kind: pipeline
type: docker
name: test-matrix

matrix:
  NODE_VERSION:
    - 18
    - 20
    - 22
  DATABASE:
    - postgres:15
    - postgres:16

services:
  - name: database
    image: ${DATABASE}
    environment:
      POSTGRES_PASSWORD: test

steps:
  - name: test
    image: node:${NODE_VERSION}-alpine
    commands:
      - npm ci
      - npm test
```

→ Tạo 3 × 2 = 6 builds song song!

---

## 19. Convert Từ GitHub Actions / GitLab CI

### Từ GitHub Actions

```yaml
# .github/workflows/ci.yml (cũ)
name: CI
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: {node-version: 20}
      - run: npm ci
      - run: npm test
```

```yaml
# .drone.yml (mới) — tương đương
kind: pipeline
type: docker
name: default

steps:
  - name: test
    image: node:20-alpine
    commands:
      - npm ci
      - npm test
```

> 💡 Drone tự checkout code — không cần step checkout riêng!

### Từ GitLab CI

```yaml
# .gitlab-ci.yml (cũ)
test:
  image: node:20
  script:
    - npm ci
    - npm test
```

```yaml
# .drone.yml (mới)
kind: pipeline
type: docker
name: default

steps:
  - name: test
    image: node:20
    commands:
      - npm ci
      - npm test
```

### Bảng mapping

| GitHub Actions | Drone |
|----------------|-------|
| `runs-on: ubuntu-latest` | `image: ubuntu:22.04` |
| `actions/checkout` | Tự động (không cần) |
| `actions/setup-node` | `image: node:20` |
| `run: npm test` | `commands: [npm test]` |
| `env:` | `environment:` |
| `if: github.ref == 'main'` | `trigger: branch: [main]` |
| `needs: [build]` | `depends_on: [build]` |
| secrets | `from_secret:` |

---

## 20. Backup & Maintenance

### Backup

```bash
# 1. Backup database SQLite
docker compose stop drone-server
cp ~/drone/drone-data/database.sqlite ~/backup/drone-$(date +%Y%m%d).sqlite
docker compose start drone-server

# 2. Backup docker-compose.yml và .env
cp docker-compose.yml ~/backup/
```

### Dọn dẹp Docker images cũ trên runner

```bash
# Chạy định kỳ (cron) trên máy runner
docker image prune -af --filter "until=168h"  # xóa image cũ hơn 7 ngày
docker builder prune -af
```

### Update Drone

```bash
# 1. Backup trước!
# 2. Pull image mới
docker compose pull

# 3. Restart
docker compose up -d

# 4. Kiểm tra version
# Vào UI → xem version ở footer
```

### Monitoring

```bash
# Xem số build đang chạy
drone build ls --status running

# Xem queue
# Vào UI → Settings → Queue
```

---

## 21. Pricing

| | Chi tiết |
|---|----------|
| **Drone Server** | Miễn phí, open source (Apache 2.0) |
| **Drone Runner** | Miễn phí, không giới hạn |
| **Harness Cloud** | Có bản cloud trả phí (từ Harness) |
| **Chi phí thực tế** | Chỉ tiền VPS: ~$5-20/tháng |

> 💡 Drone là một trong những CI **rẻ nhất** vì self-hosted hoàn toàn, không giới hạn build minutes.

---

## 22. Troubleshooting

| Vấn đề | Nguyên nhân | Cách sửa |
|--------|-------------|----------|
| Webhook không trigger build | Sai callback URL hoặc secret | Kiểm tra GitHub webhook → Recent Deliveries |
| `cannot connect to Docker daemon` | Runner không mount docker.sock | Thêm `-v /var/run/docker.sock:/var/run/docker.sock` |
| Build stuck ở Pending | Không có runner hoặc sai `type` | Kiểm tra runner đang chạy, `type: docker` khớp |
| Secret không có giá trị | Sai tên secret | Kiểm tra tên trong UI vs `from_secret:` |
| `image not found` | Sai tên image hoặc không pull được | Kiểm tra tên image, thử `docker pull` thủ công |
| PR không trigger | Chưa bật PR trong repo settings | Repo → Settings → bật Pull Requests |
| `drone.yml: yaml: line X` | Lỗi syntax YAML | Chạy `drone lint` để kiểm tra |
| Services không kết nối được | Sai hostname | Dùng **tên service** làm hostname (ví dụ `postgres`) |
| Build chậm | Không cache dependencies | Dùng volumes hoặc image có sẵn dependencies |
| `trusted` build bị chặn | Repo chưa bật Trusted | Chỉ bật cho repo tin cậy tuyệt đối! |
| OAuth login fail | Sai Client ID/Secret | Kiểm tra lại GitHub OAuth App settings |
| Runner báo `unauthorized` | Sai RPC secret | Đảm bảo server và runner dùng cùng `DRONE_RPC_SECRET` |

### Debug chi tiết

```bash
# Xem log server
docker compose logs -f drone-server

# Xem log runner
docker compose logs -f drone-runner

# Bật debug mode
# Thêm vào environment của server:
# - DRONE_LOGS_DEBUG=true
# - DRONE_LOGS_TRACE=true
```

### Kiểm tra webhook GitHub

1. GitHub repo → **Settings** → **Webhooks**
2. Bấm vào webhook của Drone → **Recent Deliveries**
3. Xem response: phải là `200 OK`
4. Nếu `4xx/5xx` → xem payload và response body

---

## 23. Best Practices Checklist

- [ ] Dùng `drone lint` kiểm tra YAML trước khi push
- [ ] Secrets qua `from_secret`, **không** hardcode
- [ ] Chia pipeline CI và CD riêng (dùng `depends_on`)
- [ ] Trigger giới hạn branch cần thiết (tránh build thừa)
- [ ] Dùng `paths` trigger cho monorepo
- [ ] Services cho database test (không dùng DB thật)
- [ ] Multi-stage: test trước, build/push sau
- [ ] Tag Docker image bằng commit SHA (traceable)
- [ ] Dọn Docker images cũ trên runner định kỳ
- [ ] Backup database SQLite định kỳ
- [ ] Bật HTTPS cho Drone server (webhook yêu cầu)
- [ ] Giới hạn `trusted` repos (nguy cơ bảo mật)
- [ ] Dùng specific image tags (`node:20.11-alpine`), tránh `latest`

---

## 24. Cheat Sheet

### .drone.yml template

```yaml
kind: pipeline
type: docker
name: default

trigger:
  branch: [main]
  event: [push, pull_request]

services:
  - name: postgres
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: test

steps:
  - name: test
    image: node:20-alpine
    environment:
      DATABASE_URL: postgres://postgres:test@postgres:5432/postgres
    commands:
      - npm ci
      - npm test

  - name: build-push
    image: plugins/docker
    settings:
      repo: myuser/myapp
      tags: [latest, ${DRONE_COMMIT_SHA:0:8}]
      username: { from_secret: docker_username }
      password: { from_secret: docker_password }
    when:
      branch: [main]
      event: [push]
```

### Biến môi trường có sẵn

| Biến | Ý nghĩa |
|------|---------|
| `DRONE_COMMIT_SHA` | Commit hash đầy đủ |
| `DRONE_COMMIT_BRANCH` | Tên branch |
| `DRONE_COMMIT_MESSAGE` | Commit message |
| `DRONE_BUILD_NUMBER` | Số build |
| `DRONE_BUILD_EVENT` | push / pull_request / tag / cron |
| `DRONE_TAG` | Tên tag (khi event=tag) |
| `DRONE_REPO` | Tên repo (org/name) |
| `DRONE_PULL_REQUEST` | Số PR |

### Drone CLI nhanh

```bash
drone lint                                    # kiểm tra YAML
drone build ls <repo>                         # list builds
drone build logs <repo> <number>              # xem log
drone build create <repo>                     # trigger thủ công
drone secret add --repository <repo> --name X --data Y
```

### Docker Compose nhanh

```bash
docker compose up -d              # chạy server + runner
docker compose logs -f            # xem log
openssl rand -hex 16              # tạo RPC secret
```

---

## 25. FAQ

**Hỏi: Drone có miễn phí hoàn toàn không?**
Đáp: Có! Open source Apache 2.0, không giới hạn builds, agents, users. Chỉ tốn tiền VPS.

**Hỏi: Drone có bản cloud không?**
Đáp: Harness (công ty sở hữu Drone) có cung cấp bản cloud, nhưng bản open source self-hosted vẫn được maintain.

**Hỏi: Có hỗ trợ Windows không?**
Đáp: Có, qua Windows runner (`drone/drone-runner-windows`), nhưng ít phổ biến hơn Docker runner.

**Hỏi: Khác gì GitHub Actions?**
Đáp: Drone nhẹ hơn, self-hosted hoàn toàn, mọi step là container. GitHub Actions dễ setup hơn nhưng tốn tiền khi hết free minutes.

**Hỏi: Có chạy trên Raspberry Pi được không?**
Đáp: Được! Drone có image ARM. Nhiều người chạy Drone server + runner trên Pi 4.

**Hỏi: Migration từ Jenkins có khó không?**
Đáp: Khá dễ. Jenkinsfile phức tạp → viết lại thành `.drone.yml` đơn giản hơn nhiều vì mỗi step chỉ là image + commands.

**Hỏi: Làm sao chạy pipeline mà không push code?**
Đáp: Dùng `drone exec` để chạy local, hoặc `drone build create` để trigger thủ công từ CLI.

---

## 26. Phụ Lục: Nâng Cao

### Jsonnet (viết pipeline bằng code)

Khi `.drone.yml` quá dài, dùng Jsonnet để generate:

```jsonnet
// .drone.jsonnet
local testNode(version) = {
  name: 'test-node-' + version,
  image: 'node:' + version + '-alpine',
  commands: ['npm ci', 'npm test'],
};

[
  {
    kind: 'pipeline',
    type: 'docker',
    name: 'test',
    steps: [
      testNode('18'),
      testNode('20'),
      testNode('22'),
    ],
  },
]
```

```bash
# Convert sang YAML
drone jsonnet --stream > .drone.yml
```

### Starlark (alternative)

```python
# .drone.star
def main(ctx):
    return {
        'kind': 'pipeline',
        'type': 'docker',
        'name': 'default',
        'steps': [
            {
                'name': 'test',
                'image': 'node:20-alpine',
                'commands': ['npm ci', 'npm test'],
            }
        ],
    }
```

### Signature (ký file .drone.yml)

Ngăn người khác sửa pipeline trái phép:

```bash
# Tạo key
openssl rand -hex 16  # DRONE_YAML_SIGN

# Ký file
drone sign octocat/hello-world
```

### Cron với timezone Việt Nam

```bash
# Trong docker-compose.yml của server:
environment:
  - TZ=Asia/Ho_Chi_Minh
```

### Tích hợp SonarQube

```yaml
steps:
  - name: sonar
    image: sonarsource/sonar-scanner-cli
    environment:
      SONAR_TOKEN:
        from_secret: sonar_token
    commands:
      - sonar-scanner -Dsonar.host.url=https://sonar.example.com
```

---

## 27. Phụ Lục: Dự Án Thực Tế Bổ Sung

### Python + FastAPI Pipeline

```yaml
kind: pipeline
type: docker
name: python-ci

trigger:
  branch: [main, develop]
  event: [push, pull_request]

services:
  - name: postgres
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: testdb
  - name: redis
    image: redis:7-alpine

steps:
  - name: lint
    image: python:3.12-slim
    commands:
      - pip install ruff
      - ruff check src/
      - ruff format --check src/

  - name: typecheck
    image: python:3.12-slim
    commands:
      - pip install mypy
      - mypy src/

  - name: test
    image: python:3.12-slim
    environment:
      DATABASE_URL: postgresql://test:test@postgres:5432/testdb
      REDIS_URL: redis://redis:6379
    commands:
      - pip install -r requirements.txt
      - pip install pytest pytest-cov
      - pytest tests/ --cov=src --cov-report=xml

  - name: build-image
    image: plugins/docker
    settings:
      repo: myuser/fastapi-app
      dockerfile: Dockerfile
      tags: ${DRONE_COMMIT_SHA:0:8}
      username: { from_secret: docker_username }
      password: { from_secret: docker_password }
    when:
      branch: [main]
      event: [push]
```

### Go Pipeline với Cache

```yaml
kind: pipeline
type: docker
name: go-ci

steps:
  - name: vet-and-test
    image: golang:1.22-alpine
    volumes:
      - name: go-cache
        path: /go/pkg/mod
    commands:
      - go vet ./...
      - go test -race -coverprofile=coverage.out ./...

  - name: build
    image: golang:1.22-alpine
    volumes:
      - name: go-cache
        path: /go/pkg/mod
    commands:
      - CGO_ENABLED=0 go build -ldflags="-s -w" -o app .

volumes:
  - name: go-cache
    temp: {}
```

### Flutter Pipeline

```yaml
kind: pipeline
type: docker
name: flutter-ci

steps:
  - name: analyze-and-test
    image: ghcr.io/cirruslabs/flutter:stable
    commands:
      - flutter analyze
      - flutter test --coverage

  - name: build-apk
    image: ghcr.io/cirruslabs/flutter:stable
    commands:
      - flutter build apk --release
    when:
      event: [tag]
```

### Approval Gate (Manual Approval)

```yaml
---
kind: pipeline
type: docker
name: staging-deploy
depends_on: [ci]

trigger:
  branch: [main]
  event: [push]

steps:
  - name: deploy-staging
    image: alpine:3.19
    commands:
      - echo "Deploying to staging..."

---
kind: pipeline
type: docker
name: production-deploy
depends_on: [staging-deploy]

trigger:
  branch: [main]
  event: [push]

# Chờ approve thủ công trong UI trước khi chạy
steps:
  - name: wait-approval
    image: alpine:3.19
    commands:
      - echo "Approved! Deploying to production..."
```

> Trong Drone UI, bấm **Approve** để cho pipeline production-deploy chạy tiếp.

### Notification Matrix

```yaml
steps:
  - name: notify-telegram
    image: appleboy/drone-telegram
    settings:
      token: { from_secret: telegram_token }
      to: { from_secret: telegram_chat_id }
      message: |
        {{#success build.status}}✅{{else}}❌{{/success}}
        {{repo.name}} #{{build.number}}
        {{build.message}}
    when:
      status: [success, failure]

  - name: notify-discord
    image: appleboy/drone-discord
    settings:
      webhook_id: { from_secret: discord_webhook_id }
      webhook_token: { from_secret: discord_webhook_token }
      message: "Build {{build.number}}: {{build.status}}"
    when:
      status: [failure]
```

### Bảo Mật Nâng Cao

| Biện pháp | Cách làm |
|-----------|----------|
| Không cho fork PR chạy secrets | Mặc định đã chặn, đừng bật Trusted bừa |
| Giới hạn image được dùng | Dùng private registry + allowlist |
| Quét lỗ hổng image | Thêm step `aquasec/trivy` |
| Ký commit | Bật required signed commits trên GitHub |
| Audit log | Xem trong Drone UI → Activity |

```yaml
# Thêm security scan vào pipeline
steps:
  - name: trivy-scan
    image: aquasec/trivy:latest
    commands:
      - trivy image --severity HIGH,CRITICAL myuser/myapp:${DRONE_COMMIT_SHA:0:8}
```

---

*Hết guide Drone CI. Chúc cậu build xanh lè!* 🚁💚
