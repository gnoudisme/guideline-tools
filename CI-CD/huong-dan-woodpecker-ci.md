# Hướng Dẫn Woodpecker CI — Từ Cài Đặt Đến Production

> Woodpecker CI là hệ thống CI/CD mã nguồn mở (Apache-2.0), community fork của Drone CI. Chạy pipeline trong Docker container, cấu hình bằng file `.woodpecker.yml`, tích hợp sâu với Gitea/GitHub/GitLab/Bitbucket/Forgejo. Guide này đi từ cài đặt bằng Docker Compose, tích hợp forge, viết pipeline, plugin, secrets, agent, CLI, đến migration từ Drone và vận hành production.

---

## Mục Lục

1. [Woodpecker CI là gì?](#1-woodpecker-ci-là-gì)
2. [Lịch sử fork từ Drone](#2-lịch-sử-fork-từ-drone)
3. [So sánh với Drone, GitHub Actions, Jenkins, GitLab CI](#3-so-sánh-với-drone-github-actions-jenkins-gitlab-ci)
4. [Yêu cầu hệ thống](#4-yêu-cầu-hệ-thống)
5. [Cài đặt với Docker Compose (server + agent)](#5-cài-đặt-với-docker-compose-server--agent)
6. [Biến môi trường đầy đủ của server và agent](#6-biến-môi-trường-đầy-đủ-của-server-và-agent)
7. [Tích hợp forge: Gitea, GitHub, GitLab, Bitbucket, Forgejo](#7-tích-hợp-forge-gitea-github-gitlab-bitbucket-forgejo)
8. [Kích hoạt repo và chạy pipeline đầu tiên](#8-kích-hoạt-repo-và-chạy-pipeline-đầu-tiên)
9. [Cú pháp `.woodpecker.yml` đầy đủ](#9-cú-pháp-woodpeckeryml-đầy-đủ)
10. [Workflows, services, volumes, workspace](#10-workflows-services-volumes-workspace)
11. [Plugins phổ biến](#11-plugins-phổ-biến)
12. [Secrets: tạo và sử dụng](#12-secrets-tạo-và-sử-dụng)
13. [Agents: thêm agent, labels, agent từ xa](#13-agents-thêm-agent-labels-agent-từ-xa)
14. [Woodpecker CLI](#14-woodpecker-cli-woodpecker-cli)
15. [Migration từ Drone sang Woodpecker](#15-migration-từ-drone-sang-woodpecker)
16. [Ví dụ thực tế hoàn chỉnh](#16-ví-dụ-thực-tế-hoàn-chỉnh)
17. [Webhooks và badges](#17-webhooks-và-badges)
18. [Backup, upgrade, bảo mật](#18-backup-upgrade-bảo-mật)
19. [Troubleshooting](#19-troubleshooting)
20. [FAQ](#20-faq)
21. [Cheat sheet](#21-cheat-sheet)
22. [Vì sao chọn Woodpecker?](#22-vì-sao-chọn-woodpecker)

---

## 1. Woodpecker CI là gì?

**Woodpecker CI** là hệ thống **Continuous Integration / Continuous Delivery** mã nguồn mở, giấy phép **Apache License 2.0**, được cộng đồng phát triển (community-driven). Đặc điểm cốt lõi:

- **Pipeline-as-code**: mọi pipeline được định nghĩa trong file `.woodpecker.yml` (hoặc `.woodpecker.yaml`) đặt ở gốc repo.
- **Chạy trong container**: mỗi step chạy trong một Docker container riêng, môi trường build sạch, tái lập được.
- **Plugin-based**: các tác vụ phổ biến (build Docker image, push registry, gửi Slack, deploy SSH, upload S3...) được đóng gói thành plugin — bản thân plugin cũng là Docker image.
- **Tích hợp forge**: nhận webhook từ Gitea, GitHub, GitLab, Bitbucket, Forgejo; tự động clone code, chạy pipeline, báo trạng thái commit về lại forge.
- **Kiến trúc server + agent**: server quản lý web UI/API, nhận webhook, lên lịch pipeline; agent là worker thực thi pipeline. Có thể chạy nhiều agent, gắn label để định tuyến.
- **Nhẹ**: server + agent đều là binary Go nhỏ gọn, chạy tốt trên VPS 1–2 GB RAM; không cần Java như Jenkins.

Website chính thức: `woodpecker-ci.org` — repo: `github.com/woodpecker-ci/woodpecker`.

## 2. Lịch sử fork từ Drone

| Năm | Sự kiện |
|---|---|
| 2012–2019 | **Drone CI** ra đời, trở thành CI chạy-container phổ biến, giấy phép Apache-2.0. Kiến trúc server + agent, config `.drone.yml`. |
| 2020 | Harness mua lại Drone. Drone đổi giấy phép sang **PolyForm** (source-available, không còn OSI open-source thuần túy); bản enterprise trả phí. Cộng đồng lo ngại về tương lai bản open-source. |
| 2020–2021 | Cộng đồng fork code Drone (bản Apache-2.0 cuối cùng) thành **Woodpecker CI**, giữ giấy phép Apache-2.0, phát triển độc lập theo hướng community-driven, không công ty đứng sau chi phối. |
| 2021–nay | Woodpecker phát triển nhanh: hỗ trợ Gitea/Forgejo native (Drone bản mới bỏ), cải tiến UI, thêm `when` filters, matrix, multi-workflow, plugin registry riêng, CLI hoàn chỉnh. Phiên bản ổn định dòng 1.x, 2.x, 3.x (kiểm tra tag mới nhất trên GitHub releases). |

**Điểm khác biệt triết lý**: Drone (dưới Harness) tập trung enterprise; Woodpecker tập trung self-hosted, cộng đồng, hỗ trợ tốt các forge tự host như Gitea/Forgejo — đúng nhu cầu của team muốn "CI của mình, chạy trên server của mình".

## 3. So sánh với Drone, GitHub Actions, Jenkins, GitLab CI

| Tiêu chí | Woodpecker CI | Drone (Harness) | GitHub Actions | Jenkins | GitLab CI |
|---|---|---|---|---|---|
| Giấy phép | Apache-2.0 (open source thuần) | PolyForm (source-available) | Proprietary (runner OSS) | MIT | MIT (core) |
| Self-hosted | ✅ native, nhẹ | ✅ | ⚠️ self-hosted runner | ✅ | ✅ |
| Config | `.woodpecker.yml` | `.drone.yml` | `.github/workflows/*.yml` | Jenkinsfile (Groovy) | `.gitlab-ci.yml` |
| Chạy trong container | ✅ mỗi step 1 container | ✅ | ✅ (container jobs) | ⚠️ cần plugin | ✅ |
| Gitea/Forgejo native | ✅ | ❌ (bản mới) | ❌ | ⚠️ plugin | ❌ |
| Web UI | Gọn, đủ dùng | Tương tự | Tích hợp GitHub | Già cỗi, plugin | Tích hợp GitLab |
| Tài nguyên | Rất nhẹ (Go binary) | Nhẹ | N/A (cloud) | Nặng (Java) | Trung bình |
| Plugin ecosystem | Plugin Docker image, registry riêng | Tương thích phần lớn | Marketplace khổng lồ | Plugin khổng lồ | Component catalog |
| Chi phí | Free, server của bạn | Free tier / trả phí | Free quota / trả phí | Free | Free tier / trả phí |
| Vendor lock-in | Không | Có (Harness) | Cao | Không | Trung bình |

**Khi nào chọn Woodpecker**: bạn self-host Gitea/Forgejo/GitLab, muốn CI nhẹ chạy trên VPS, không muốn phụ thuộc GitHub/Microsoft hay Harness, thích config YAML đơn giản chạy trong container.

---

## 4. Yêu cầu hệ thống

**Server Woodpecker:**

| Thành phần | Tối thiểu | Khuyến nghị production |
|---|---|---|
| CPU | 1 core | 2+ cores |
| RAM | 512 MB | 2 GB |
| Disk | 5 GB | 20 GB+ (tùy log/artifacts) |
| Docker | 20.10+ | 24+ |
| Database | SQLite (mặc định) | PostgreSQL |

**Agent:**

| Thành phần | Tối thiểu | Khuyến nghị |
|---|---|---|
| CPU | 1 core | 2+ cores (tùy số pipeline song song) |
| RAM | 512 MB | 2–4 GB |
| Disk | 10 GB | 50 GB+ (Docker images, cache) |
| Docker | 20.10+ | 24+ |

**Mạng**: server cần HTTPS public (để forge gọi webhook về) — dùng reverse proxy (Nginx/Caddy/Traefik) + Let's Encrypt. Agent chỉ cần kết nối **ra** tới server qua gRPC (port 9000), không cần public IP.

---

## 5. Cài đặt với Docker Compose (server + agent)

### 5.1. Chuẩn bị

```bash
mkdir -p ~/woodpecker && cd ~/woodpecker
```

Tạo file `.env` (không commit file này):

```bash
# Domain public của Woodpecker (bắt buộc HTTPS cho webhook)
WOODPECKER_HOST=https://ci.example.com

# Secret dùng để agent xác thực với server (tự sinh chuỗi ngẫu nhiên dài)
WOODPECKER_AGENT_SECRET=$(openssl rand -hex 32)
```

> Sinh secret: `openssl rand -hex 32`. Giữ bí mật — ai có secret này có thể đăng ký agent giả vào server.

### 5.2. Docker Compose đầy đủ (server + agent + PostgreSQL)

Kiểm tra tag image mới nhất tại `github.com/woodpecker-ci/woodpecker/releases` — ví dụ dưới dùng `3.x`:

```yaml
# ~/woodpecker/docker-compose.yml
services:
  woodpecker-server:
    image: woodpeckerci/woodpecker-server:3.8.0
    container_name: woodpecker-server
    restart: unless-stopped
    ports:
      - "8000:8000"   # Web UI + API
      - "9000:9000"   # gRPC cho agent
    volumes:
      - woodpecker-data:/var/lib/woodpecker
    environment:
      # --- Bắt buộc ---
      - WOODPECKER_HOST=${WOODPECKER_HOST}
      - WOODPECKER_AGENT_SECRET=${WOODPECKER_AGENT_SECRET}
      # --- Forge: chọn 1 (ví dụ Gitea, chi tiết ở mục 7) ---
      - WOODPECKER_GITEA=true
      - WOODPECKER_GITEA_URL=https://gitea.example.com
      - WOODPECKER_GITEA_CLIENT=${GITEA_CLIENT_ID}
      - WOODPECKER_GITEA_SECRET=${GITEA_CLIENT_SECRET}
      # --- Database PostgreSQL (khuyến nghị production) ---
      - WOODPECKER_DATABASE_DRIVER=postgres
      - WOODPECKER_DATABASE_DATASOURCE=postgres://woodpecker:${DB_PASSWORD}@db:5432/woodpecker?sslmode=disable
      # --- Tùy chọn hay dùng ---
      - WOODPECKER_OPEN=false            # false = chỉ user đã đăng nhập mới thấy; true = public
      - WOODPECKER_ADMIN=admin           # username forge được quyền admin Woodpecker
      - WOODPECKER_LOG_LEVEL=info

  woodpecker-agent:
    image: woodpeckerci/woodpecker-agent:3.8.0
    container_name: woodpecker-agent
    restart: unless-stopped
    depends_on:
      - woodpecker-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WOODPECKER_SERVER=woodpecker-server:9000
      - WOODPECKER_AGENT_SECRET=${WOODPECKER_AGENT_SECRET}
      - WOODPECKER_MAX_WORKFLOWS=2       # số pipeline chạy song song
      - WOODPECKER_HEALTHCHECK=true

  db:
    image: postgres:16-alpine
    container_name: woodpecker-db
    restart: unless-stopped
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=woodpecker
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=woodpecker

volumes:
  woodpecker-data:
  postgres-data:
```

Chạy:

```bash
docker compose up -d
docker compose logs -f woodpecker-server
```

Mở `https://ci.example.com` (qua reverse proxy trỏ tới `localhost:8000`) → bấm **Login** → đăng nhập bằng tài khoản forge → xong.

### 5.3. Reverse proxy mẫu (Caddy — đơn giản nhất)

```
# /etc/caddy/Caddyfile
ci.example.com {
    reverse_proxy localhost:8000
}
```

Caddy tự cấp Let's Encrypt. Với Nginx:

```nginx
server {
    listen 443 ssl;
    server_name ci.example.com;
    ssl_certificate /etc/letsencrypt/live/ci.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/ci.example.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        # WebSocket cho log realtime
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 5.4. Bản tối giản (SQLite, 1 file, test nhanh)

```yaml
services:
  server:
    image: woodpeckerci/woodpecker-server:3.8.0
    ports: ["8000:8000", "9000:9000"]
    volumes: ["./data:/var/lib/woodpecker"]
    environment:
      - WOODPECKER_HOST=http://localhost:8000
      - WOODPECKER_AGENT_SECRET=changeme
      - WOODPECKER_GITEA=true
      - WOODPECKER_GITEA_URL=https://gitea.example.com
      - WOODPECKER_GITEA_CLIENT=xxx
      - WOODPECKER_GITEA_SECRET=yyy
      - WOODPECKER_ADMIN=admin
  agent:
    image: woodpeckerci/woodpecker-agent:3.8.0
    volumes: ["/var/run/docker.sock:/var/run/docker.sock"]
    environment:
      - WOODPECKER_SERVER=server:9000
      - WOODPECKER_AGENT_SECRET=changeme
```

---

## 6. Biến môi trường đầy đủ của server và agent

### 6.1. Server (tiền tố `WOODPECKER_`)

| Biến | Bắt buộc | Mô tả |
|---|---|---|
| `WOODPECKER_HOST` | ✅ | URL public của server, ví dụ `https://ci.example.com` |
| `WOODPECKER_AGENT_SECRET` | ✅ | Secret chia sẻ giữa server và agent |
| `WOODPECKER_ADMIN` | — | Username forge làm admin (phân cách dấu phẩy) |
| `WOODPECKER_OPEN` | — | `true` cho phép xem public không cần login (mặc định `false`) |
| `WOODPECKER_ORGS` | — | Giới hạn org được dùng, ví dụ `myorg,team-a` |
| `WOODPECKER_REPO_OWNERS` | — | Giới hạn repo owner |
| `WOODPECKER_DATABASE_DRIVER` | — | `sqlite3` (mặc định) hoặc `postgres` |
| `WOODPECKER_DATABASE_DATASOURCE` | — | Connection string |
| `WOODPECKER_LOG_LEVEL` | — | `trace/debug/info/warn/error` |
| `WOODPECKER_LOG_FILE` | — | Ghi log ra file |
| `WOODPECKER_SESSION_EXPIRES` | — | Thời gian session, mặc định `72h` |
| `WOODPECKER_PIPELINE_DEFAULT_TIMEOUT` | — | Timeout mặc định mỗi pipeline, ví dụ `60m` |
| `WOODPECKER_PIPELINE_MAX_TIMEOUT` | — | Timeout tối đa cho phép |
| `WOODPECKER_STATUS_CONTEXT` | — | Tên context status báo về forge (mặc định `ci/woodpecker`) |
| `WOODPECKER_STATUS_CONTEXT_FORMAT` | — | Format tên context |
| `WOODPECKER_WEBHOOK_HOST` | — | Host riêng cho webhook nếu khác `WOODPECKER_HOST` |
| `WOODPECKER_WEBHOOK_SKIP_VERIFY` | — | `true` để bỏ verify TLS webhook (chỉ lab) |
| `WOODPECKER_PROMETHEUS_AUTH_TOKEN` | — | Token cho endpoint `/metrics` |
| `WOODPECKER_PLUGINS_PRIVILEGED` | — | Plugin image được chạy privileged (phân cách dấu phẩy), ví dụ `woodpeckerci/plugin-docker-buildx` |
| `WOODPECKER_AUTHENTICATE_PUBLIC_REPOS` | — | Bắt buộc auth cả repo public |
| `WOODPECKER_DEFAULT_CLONE_PLUGIN` | — | Đổi plugin clone mặc định |
| `WOODPECKER_DEFAULT_CANCEL_PREVIOUS_PIPELINE_EVENTS` | — | Tự hủy pipeline cũ khi có push mới |
| `WOODPECKER_GRPC_ADDR` | — | Địa chỉ gRPC listen (mặc định `:9000`) |
| `WOODPECKER_SERVER_ADDR` | — | Địa chỉ HTTP listen (mặc định `:8000`) |

### 6.2. Agent (tiền tố `WOODPECKER_`)

| Biến | Bắt buộc | Mô tả |
|---|---|---|
| `WOODPECKER_SERVER` | ✅ | `host:9000` của server (gRPC) |
| `WOODPECKER_AGENT_SECRET` | ✅ | Khớp với server |
| `WOODPECKER_MAX_WORKFLOWS` | — | Số pipeline song song (mặc định `1`) |
| `WOODPECKER_AGENT_LABELS` / `WOODPECKER_FILTER_LABELS` | — | Label của agent, ví dụ `platform=linux/amd64,docker=true` |
| `WOODPECKER_HEALTHCHECK` | — | Bật healthcheck |
| `WOODPECKER_TIMEOUT` | — | Timeout kết nối server |
| `WOODPECKER_DOCKER_CONFIG` | — | Path docker config (chứa auth registry) |
| `WOODPECKER_BACKEND` | — | `docker` (mặc định) hoặc `local` |
| `WOODPECKER_BACKEND_DOCKER_HOST` | — | Docker host cho backend (mặc định dùng socket mount) |
| `WOODPECKER_BACKEND_DOCKER_NETWORK` | — | Network cho container pipeline |
| `WOODPECKER_BACKEND_DOCKER_VOLUMES` | — | Volume mount thêm |

---

## 7. Tích hợp forge: Gitea, GitHub, GitLab, Bitbucket, Forgejo

Nguyên tắc chung: tạo **OAuth application** trên forge → lấy Client ID + Client Secret → điền vào biến môi trường server → restart server. Callback URL luôn là:

```
https://ci.example.com/authorize
```

### 7.1. Gitea (khuyến nghị cho self-hosted)

**Bước 1 — Tạo OAuth2 Application trên Gitea:**

1. Đăng nhập Gitea → avatar → **Settings** → **Applications** → **Manage OAuth2 Applications** → **Create a new OAuth2 application**.
2. Điền:
   - **Application Name**: `Woodpecker CI`
   - **Redirect URI**: `https://ci.example.com/authorize`
3. Bấm **Create Application** → copy **Client ID** và **Client Secret**.

**Bước 2 — Cấu hình server:**

```yaml
environment:
  - WOODPECKER_GITEA=true
  - WOODPECKER_GITEA_URL=https://gitea.example.com
  - WOODPECKER_GITEA_CLIENT=${GITEA_CLIENT_ID}
  - WOODPECKER_GITEA_SECRET=${GITEA_CLIENT_SECRET}
  # Nếu Gitea dùng self-signed cert (lab):
  - WOODPECKER_GITEA_SKIP_VERIFY=true
```

**Bước 3** — Restart server, đăng nhập Woodpecker bằng tài khoản Gitea.

> Webhook: Woodpecker tự tạo webhook trên repo khi bạn bấm **Activate** (mục 8). Không cần tạo tay.

### 7.2. GitHub

**Bước 1 — Tạo OAuth App:**

1. GitHub → **Settings** → **Developer settings** → **OAuth Apps** → **New OAuth App**.
2. Điền:
   - **Application name**: `Woodpecker CI`
   - **Homepage URL**: `https://ci.example.com`
   - **Authorization callback URL**: `https://ci.example.com/authorize`
3. **Register application** → **Generate a new client secret** → copy Client ID + Secret.

**Bước 2 — Cấu hình server:**

```yaml
environment:
  - WOODPECKER_GITHUB=true
  - WOODPECKER_GITHUB_CLIENT=${GITHUB_CLIENT_ID}
  - WOODPECKER_GITHUB_SECRET=${GITHUB_CLIENT_SECRET}
  # GitHub Enterprise:
  # - WOODPECKER_GITHUB_URL=https://ghe.example.com
  # - WOODPECKER_GITHUB_API_URL=https://ghe.example.com/api/v3
```

**Lưu ý quyền**: OAuth app cần quyền `repo` (private) hoặc `public_repo` + `user:email`. Khi activate repo, Woodpecker tạo webhook — user đăng nhập phải có quyền **admin** trên repo đó.

### 7.3. GitLab

**Bước 1 — Tạo OAuth Application:**

1. GitLab → avatar → **Preferences** → **Applications** → **Add new application**.
2. Điền:
   - **Name**: `Woodpecker CI`
   - **Redirect URI**: `https://ci.example.com/authorize`
   - **Scopes**: tick `api`
3. **Save application** → copy **Application ID** và **Secret**.

**Bước 2 — Cấu hình server:**

```yaml
environment:
  - WOODPECKER_GITLAB=true
  - WOODPECKER_GITLAB_CLIENT=${GITLAB_CLIENT_ID}
  - WOODPECKER_GITLAB_SECRET=${GITLAB_CLIENT_SECRET}
  # Self-hosted GitLab:
  - WOODPECKER_GITLAB_URL=https://gitlab.example.com
```

### 7.4. Bitbucket

**Bước 1 — Tạo OAuth consumer:**

1. Bitbucket → avatar → **Workspace settings** (hoặc Personal settings) → **OAuth consumers** → **Add consumer**.
2. Điền:
   - **Name**: `Woodpecker CI`
   - **Callback URL**: `https://ci.example.com/authorize`
   - **Permissions**: `Account: Read`, `Repositories: Read, Write`, `Pull requests: Read`, `Webhooks: Read and write`
3. Copy **Key** (Client ID) và **Secret**.

**Bước 2 — Cấu hình server:**

```yaml
environment:
  - WOODPECKER_BITBUCKET=true
  - WOODPECKER_BITBUCKET_CLIENT=${BITBUCKET_CLIENT_ID}
  - WOODPECKER_BITBUCKET_SECRET=${BITBUCKET_CLIENT_SECRET}
```

### 7.5. Forgejo

Forgejo fork từ Gitea nên cách làm **giống hệt mục 7.1**, chỉ khác URL:

```yaml
environment:
  - WOODPECKER_GITEA=true   # dùng driver Gitea cho Forgejo
  - WOODPECKER_GITEA_URL=https://forgejo.example.com
  - WOODPECKER_GITEA_CLIENT=${FORGEJO_CLIENT_ID}
  - WOODPECKER_GITEA_SECRET=${FORGEJO_CLIENT_SECRET}
```

Tạo OAuth2 application trong Forgejo: **Settings → Applications → Manage OAuth2 Applications** (đường dẫn giống Gitea).

### 7.6. Dùng nhiều forge cùng lúc

Woodpecker hỗ trợ bật nhiều driver cùng lúc — chỉ cần khai báo đủ biến cho từng loại. User chọn forge khi đăng nhập.

---

## 8. Kích hoạt repo và chạy pipeline đầu tiên

**Bước 1** — Đăng nhập Woodpecker (`https://ci.example.com`) bằng tài khoản forge.

**Bước 2** — Bấm **Repositories** (hoặc vào trang cá nhân) → danh sách repo từ forge hiện ra → tìm repo → bấm **Activate**.

**Bước 3** — Cấu hình repo (màn hình Settings của repo):
- **Config file**: đường dẫn file pipeline, mặc định `.woodpecker.yml`.
- **Trusted**: bật nếu pipeline cần `privileged` (build Docker image với DinD). ⚠️ Chỉ bật cho repo bạn tin tưởng tuyệt đối.
- **Protected**: chỉ chạy pipeline cho branch/tag protected.
- **Timeout**: timeout riêng cho repo.

Khi activate, Woodpecker tự tạo webhook trên forge trỏ về `https://ci.example.com/hook`.

**Bước 4** — Thêm file `.woodpecker.yml` vào gốc repo:

```yaml
# .woodpecker.yml — pipeline đầu tiên
when:
  - event: [push, pull_request]

steps:
  hello:
    image: alpine:3.20
    commands:
      - echo "Xin chào từ Woodpecker CI!"
      - echo "Branch: $CI_COMMIT_BRANCH"
      - echo "Commit: $CI_COMMIT_SHA"
```

**Bước 5** — `git add .woodpecker.yml && git commit -m "ci: add woodpecker pipeline" && git push` → vào Woodpecker UI xem pipeline chạy, bấm vào step để xem log realtime.

---

## 9. Cú pháp `.woodpecker.yml` đầy đủ

### 9.1. Khung tổng quát

```yaml
# Biến định nghĩa ở cấp pipeline (áp dụng cho mọi step)
variables:
  GO_VERSION: "1.23"

# Bộ lọc toàn cục: pipeline chỉ chạy khi khớp
when:
  - event: [push, pull_request]
    branch: [main, develop]

# Clone code (tùy chỉnh hoặc tắt)
clone:
  git:
    image: woodpeckerci/plugin-git:latest
    settings:
      depth: 50

# Các bước thực thi, chạy TUẦN TỰ
steps:
  ten-step:
    image: golang:1.23-alpine
    commands:
      - go build ./...
    environment:
      CGO_ENABLED: "0"
    when:
      - event: push
        branch: main

# Workflow định nghĩa ở mục 10
```

### 9.2. `steps` — từng step

```yaml
steps:
  build:
    image: node:20-alpine            # Docker image chạy step (bắt buộc)
    commands:                        # lệnh shell chạy trong container
      - npm ci
      - npm run build
    environment:                     # biến môi trường riêng cho step
      NODE_ENV: production
      API_URL: "https://api.example.com"
    secrets: [npm_token]             # secrets từ Woodpecker (mục 12)
    working_dir: /app/subdir         # thư mục làm việc (mặc định: /woodpecker/src/<repo>)
    when:                            # filter riêng cho step
      - event: push
        branch: main
    depends_on: [test]               # chạy sau step test (mặc định đã tuần tự)
    failure: ignore                  # bỏ qua lỗi step này (mặc định: fail cả pipeline)
    detach: false                    # true = chạy nền (cho service)
```

| Thuộc tính | Mô tả |
|---|---|
| `image` | Image Docker của step (bắt buộc, trừ plugin có image sẵn) |
| `commands` | List lệnh shell, chạy tuần tự; lệnh nào exit ≠ 0 → step fail |
| `environment` | Map biến môi trường (static) |
| `secrets` | List tên secret lấy từ Woodpecker, expose thành biến môi trường cùng tên |
| `working_dir` | Thư mục làm việc trong container |
| `when` | Điều kiện chạy step (ghi đè/AND với when toàn cục) |
| `depends_on` | Step phụ thuộc (dùng cho chạy song song ở mục 10) |
| `failure` | `ignore` để pipeline tiếp tục khi step fail |
| `detach` | `true` để chạy detach (service nền) |
| `privileged` | `true` để chạy privileged (cần repo Trusted) |
| `dns`, `extra_hosts`, `network_mode` | Tùy chỉnh network container |

### 9.3. `when` — filters đầy đủ

```yaml
when:
  - event: [push, pull_request, tag, cron, manual, deployment]
    branch: [main, develop, feature/*]   # hỗ trợ glob
    # branch: ["!main"]                  # phủ định bằng !
    path:                                # chỉ chạy khi file đổi khớp pattern
      include: ["src/**", "go.mod"]
      exclude: ["docs/**", "*.md"]
    instance: [ci.example.com]           # filter theo instance
    repo: [myorg/myrepo]
    actor: [deploy-bot]                  # filter theo user kích hoạt
    # cron: [nightly]                    # filter theo tên cron job
```

| Event | Kích hoạt khi |
|---|---|
| `push` | Push commit lên branch |
| `pull_request` | Mở/cập nhật PR |
| `tag` | Push tag |
| `cron` | Cron job của Woodpecker |
| `manual` | Bấm chạy tay trên UI/API |
| `deployment` | Deployment event từ forge |

**Ví dụ thực tế:**

```yaml
# Chỉ build Docker image khi push tag v*
when:
  - event: tag
    branch: v*

# Bỏ qua pipeline khi chỉ sửa docs
when:
  - event: push
    path:
      exclude: ["docs/**", "*.md", "LICENSE"]
```

### 9.4. `matrix` — matrix builds

```yaml
matrix:
  GO_VERSION: ["1.22", "1.23"]
  DB: [postgres, mysql]

steps:
  test:
    image: golang:${GO_VERSION}-alpine
    commands:
      - echo "Test với Go ${GO_VERSION} + ${DB}"
```

Woodpecker tạo 2×2 = 4 pipeline con (mỗi tổ hợp một workflow run), mỗi run có biến `${GO_VERSION}`, `${DB}`.

### 9.5. Biến môi trường có sẵn trong mọi pipeline

| Biến | Nội dung |
|---|---|
| `CI` | `woodpecker` |
| `CI_REPO` | `owner/name` |
| `CI_REPO_NAME` | Tên repo |
| `CI_REPO_OWNER` | Owner |
| `CI_REPO_URL` | URL clone |
| `CI_COMMIT_SHA` | SHA commit |
| `CI_COMMIT_BRANCH` | Branch (push) |
| `CI_COMMIT_TAG` | Tag (event tag) |
| `CI_COMMIT_MESSAGE` | Message commit |
| `CI_COMMIT_AUTHOR` | Tác giả |
| `CI_PIPELINE_NUMBER` | Số thứ tự pipeline |
| `CI_PIPELINE_EVENT` | `push`, `pull_request`, `tag`... |
| `CI_PIPELINE_URL` | Link pipeline trên UI |
| `CI_STEP_NAME` | Tên step hiện tại |
| `CI_WORKSPACE` | `/woodpecker/src/<owner>/<repo>` |
| `CI_SYSTEM_HOST` | Host của server |

---

## 10. Workflows, services, volumes, workspace

### 10.1. Nhiều workflow trong 1 file (chạy song song)

Dùng `depends_on` ở cấp workflow để tạo DAG:

```yaml
variables:
  APP: myapp

# Workflow 1: test
steps:
  unit-test:
    image: golang:1.23-alpine
    commands: [go test ./...]
  lint:
    image: golang:1.23-alpine
    commands: [go vet ./...]

---
# Workflow 2: build — chạy song song với workflow 1
depends_on: []
steps:
  build:
    image: golang:1.23-alpine
    commands: [go build -o app .]

---
# Workflow 3: release — chạy sau cả 2 workflow trên
depends_on: [unit-test, build]
when:
  - event: tag
steps:
  release:
    image: alpine:3.20
    commands: [echo "Release ${CI_COMMIT_TAG}"]
```

> Mỗi khối cách nhau bằng `---` là một workflow. `depends_on: []` (hoặc bỏ trống) = chạy song song ngay từ đầu. Đặt tên workflow bằng `name:` ở đầu khối để dễ đọc.

### 10.2. Services — sidecar container (postgres, redis...)

```yaml
services:
  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: testdb
  cache:
    image: redis:7-alpine

steps:
  test:
    image: golang:1.23-alpine
    commands:
      - go test ./...
    environment:
      DB_HOST: database      # hostname = tên service
      DB_PORT: "5432"
      REDIS_HOST: cache
```

Service chạy detach, step truy cập qua hostname là tên service. Thêm `wait` nếu cần chờ service sẵn sàng:

```yaml
services:
  database:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: test
steps:
  test:
    image: golang:1.23-alpine
    commands:
      - sleep 5
      - pg_isready -h database -U postgres
      - go test ./...
```

### 10.3. Volumes

```yaml
steps:
  build:
    image: golang:1.23-alpine
    commands: [go build ./...]
    volumes:
      - /tmp/cache:/go/pkg/mod     # mount host path (cần agent cho phép)
```

Mount volume host yêu cầu agent cấu hình `WOODPECKER_BACKEND_DOCKER_VOLUMES`. Cách an toàn hơn cho cache là dùng plugin cache hoặc `volumes` dạng named do Woodpecker quản lý trong workspace.

### 10.4. Workspace

Mặc định code được clone vào `/woodpecker/src/<repo>` và **share giữa các step** trong cùng workflow qua workspace volume. Tùy chỉnh:

```yaml
workspace:
  base: /woodpecker
  path: src/myorg/myrepo
```

File tạo ra ở step trước (ví dụ binary sau `go build`) vẫn còn ở step sau — không cần artifacts thủ công trong cùng workflow.

---

## 11. Plugins phổ biến

Plugin là Docker image nhận config qua `settings:`. Danh sách chính thức: `woodpecker-ci.org/plugins`.

### 11.1. Build & publish Docker image

```yaml
steps:
  build-image:
    image: woodpeckerci/plugin-docker-buildx
    privileged: true                 # cần repo Trusted
    settings:
      repo: registry.example.com/myorg/myapp
      registry: registry.example.com
      username:
        from_secret: registry_user
      password:
        from_secret: registry_pass
      tags:
        - latest
        - ${CI_COMMIT_SHA:0:8}
        - ${CI_COMMIT_TAG}            # khi event tag
      platforms: linux/amd64,linux/arm64
      dockerfile: Dockerfile
      context: .
    when:
      - event: [push, tag]
        branch: [main, v*]
```

### 11.2. Thông báo Slack / Discord / Telegram / Email

```yaml
steps:
  notify-slack:
    image: woodpeckerci/plugin-slack
    settings:
      webhook:
        from_secret: slack_webhook
      channel: "#deploy"
      template: |
        {{#success build.status}}✅{{else}}❌{{/success}} Build {{build.number}} ({{repo.name}}@{{commit.branch}}):
        {{build.link}}
    when:
      - event: [push, tag]
        status: [success, failure]   # chỉ gửi khi xong (success/failure)

  notify-discord:
    image: woodpeckerci/plugin-discord
    settings:
      webhook:
        from_secret: discord_webhook
      message: "Deploy {{build.number}} — {{build.status}}"

  notify-email:
    image: woodpeckerci/plugin-email
    settings:
      host: smtp.example.com
      port: "587"
      username:
        from_secret: smtp_user
      password:
        from_secret: smtp_pass
      from: ci@example.com
      recipients: [team@example.com]
```

### 11.3. Upload S3 / MinIO

```yaml
steps:
  upload:
    image: woodpeckerci/plugin-s3
    settings:
      bucket: my-artifacts
      endpoint: https://s3.example.com
      path_style: true               # bắt buộc cho MinIO
      access_key:
        from_secret: s3_access
      secret_key:
        from_secret: s3_secret
      source: dist/**/*
      target: /releases/${CI_COMMIT_TAG}/
      strip_prefix: dist/
```

### 11.4. Deploy qua SSH

```yaml
steps:
  deploy:
    image: appleboy/drone-ssh      # plugin SSH tương thích Drone
    settings:
      host:
        from_secret: ssh_host
      username:
        from_secret: ssh_user
      key:
        from_secret: ssh_key       # private key
      port: "22"
      script:
        - cd /opt/myapp
        - docker compose pull && docker compose up -d
    when:
      - event: push
        branch: main
```

### 11.5. Git push / release

```yaml
steps:
  release-gitea:
    image: woodpeckerci/plugin-gitea-release
    settings:
      base_url: https://gitea.example.com
      api_key:
        from_secret: gitea_token
      files: dist/*
      title: "Release ${CI_COMMIT_TAG}"
    when:
      - event: tag
```

### 11.6. Các plugin hữu ích khác

| Plugin | Image | Dùng cho |
|---|---|---|
| Docker buildx | `woodpeckerci/plugin-docker-buildx` | Build/push multi-arch |
| Kaniko | `woodpeckerci/plugin-kaniko` | Build image không cần privileged |
| Slack | `woodpeckerci/plugin-slack` | Thông báo Slack |
| Discord | `woodpeckerci/plugin-discord` | Thông báo Discord |
| Telegram | `woodpeckerci/plugin-telegram` | Thông báo Telegram |
| Email | `woodpeckerci/plugin-email` | Gửi mail |
| S3 | `woodpeckerci/plugin-s3` | Upload S3/MinIO |
| SSH | `appleboy/drone-ssh` | Chạy lệnh qua SSH |
| SCP | `appleboy/drone-scp` | Copy file qua SCP |
| Gitea release | `woodpeckerci/plugin-gitea-release` | Tạo release Gitea |
| GitHub release | `woodpeckerci/plugin-github-release` | Tạo release GitHub |
| Helm | `woodpeckerci/plugin-helm` | Deploy Helm chart |
| Kubectl | `woodpeckerci/plugin-kubectl` | Apply Kubernetes |
| PyPI publish | `woodpeckerci/plugin-pypi-publish` | Publish package Python |
| NPM publish | `woodpeckerci/plugin-npm` | Publish package npm |
| Codecov | `woodpeckerci/plugin-codecov` | Upload coverage |

> Plugin của Drone (ảnh `drone/*`, `appleboy/*`) chạy được trên Woodpecker vì tương thích config `.drone.yml`.

---

## 12. Secrets: tạo và sử dụng

### 12.1. Tạo secret

**Cách 1 — Web UI:** vào repo → **Settings** → **Secrets** → **Add secret** → nhập Name + Value.

**Cách 2 — CLI:**

```bash
woodpecker-cli secret add --repository owner/repo --name registry_pass --value s3cr3t
woodpecker-cli secret ls --repository owner/repo
woodpecker-cli secret rm --repository owner/repo --name old_key
```

**Secret theo event** (chỉ expose cho event nhất định — tăng bảo mật):

```bash
woodpecker-cli secret add --repository owner/repo \
  --name prod_key --value xxx \
  --event push --event tag        # không expose cho pull_request
```

**Secret tổ chức (organization)**: tạo ở cấp org để dùng chung cho nhiều repo (cần quyền admin).

### 12.2. Dùng secret trong pipeline

```yaml
steps:
  deploy:
    image: alpine:3.20
    secrets: [api_token]              # expose thành $api_token
    commands:
      - curl -H "Authorization: Bearer $api_token" https://api.example.com/deploy

  docker-push:
    image: woodpeckerci/plugin-docker-buildx
    settings:
      username:
        from_secret: registry_user    # dùng trong settings của plugin
      password:
        from_secret: registry_pass
```

### 12.3. Credential filtering

Woodpecker tự động **che secret trong log**: mọi giá trị secret xuất hiện trong output đều bị thay bằng `********`. Tuy nhiên:

- ⚠️ Không `echo $secret` để debug — dù bị che, vẫn tránh.
- ⚠️ Secret có thể lộ nếu script encode nó (base64) rồi in ra — filtering chỉ khớp chuỗi gốc.
- ✅ Giới hạn secret theo event: secret deploy production chỉ cho `push`/`tag` trên `main`, không cho `pull_request` (tránh PR từ fork đánh cắp).

---

## 13. Agents: thêm agent, labels, agent từ xa

### 13.1. Thêm agent mới (cùng server)

```yaml
# agent-2/docker-compose.yml
services:
  agent:
    image: woodpeckerci/woodpecker-agent:3.8.0
    restart: unless-stopped
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WOODPECKER_SERVER=192.168.1.10:9000   # IP/hostname server
      - WOODPECKER_AGENT_SECRET=${WOODPECKER_AGENT_SECRET}
      - WOODPECKER_MAX_WORKFLOWS=4
```

Agent tự đăng ký với server qua gRPC — không cần khai báo trước trên UI.

### 13.2. Labels — định tuyến pipeline tới agent phù hợp

Gán label cho agent:

```yaml
environment:
  - WOODPECKER_SERVER=server:9000
  - WOODPECKER_AGENT_SECRET=xxx
  - WOODPECKER_AGENT_LABELS=platform=linux/arm64,gpu=true
```

Pipeline yêu cầu label:

```yaml
# .woodpecker.yml
labels:
  platform: linux/arm64

steps:
  build:
    image: alpine:3.20
    commands: [uname -m]
```

Chỉ agent có label `platform=linux/arm64` mới nhận pipeline này. Dùng cho: build ARM, máy có GPU, agent đặt trong mạng nội bộ cần deploy.

### 13.3. Chạy agent trên server khác (remote agent)

1. Mở firewall cho port **9000** trên server Woodpecker (chỉ cho IP của agent).
2. Trên máy remote, chạy agent với `WOODPECKER_SERVER=ci.example.com:9000`.
3. Nên bọc gRPC trong TLS: đặt reverse proxy hoặc dùng WireGuard/Tailscale giữa server và agent thay vì expose port 9000 ra internet.

### 13.4. Backend `local` (chạy không cần Docker)

Agent có thể chạy step trực tiếp trên host (không qua container) — hữu ích cho máy build đặc biệt (macOS, NixOS):

```yaml
environment:
  - WOODPECKER_BACKEND=local
```

Pipeline dùng backend local:

```yaml
steps:
  build:
    image: placeholder              # vẫn cần field image nhưng không dùng
    backend: local
    commands: [make build]
```

---

## 14. Woodpecker CLI (`woodpecker-cli`)

### 14.1. Cài đặt

```bash
# Linux (amd64) — kiểm tra release mới nhất trên GitHub
curl -L https://github.com/woodpecker-ci/woodpecker/releases/latest/download/woodpecker-cli_linux_amd64.tar.gz \
  | tar xz -C /tmp
sudo mv /tmp/woodpecker-cli /usr/local/bin/

# macOS
brew install woodpecker-cli

# Kiểm tra
woodpecker-cli --version
```

### 14.2. Cấu hình kết nối

```bash
export WOODPECKER_SERVER=https://ci.example.com
# Lấy token: Woodpecker UI → avatar → Personal Access Token → tạo token
export WOODPECKER_TOKEN=xxx
```

### 14.3. Lệnh thường dùng

```bash
# --- Thông tin ---
woodpecker-cli info                          # thông tin server
woodpecker-cli user ls                       # danh sách user (admin)
woodpecker-cli repo ls                       # repo đã activate
woodpecker-cli repo add owner/repo           # activate repo
woodpecker-cli repo rm owner/repo            # deactivate

# --- Pipeline ---
woodpecker-cli pipeline ls owner/repo        # liệt kê pipeline
woodpecker-cli pipeline info owner/repo 42   # chi tiết pipeline #42
woodpecker-cli pipeline logs owner/repo 42   # xem log
woodpecker-cli pipeline start owner/repo 42  # chạy lại
woodpecker-cli pipeline stop owner/repo 42   # dừng pipeline đang chạy
woodpecker-cli pipeline create owner/repo --branch main   # trigger tay

# --- Secrets ---
woodpecker-cli secret ls --repository owner/repo
woodpecker-cli secret add --repository owner/repo --name k --value v
woodpecker-cli secret rm --repository owner/repo --name k

# --- Registry credentials ---
woodpecker-cli registry ls --repository owner/repo
woodpecker-cli registry add --repository owner/repo \
  --hostname registry.example.com --username u --password p

# --- Cron ---
woodpecker-cli cron ls --repository owner/repo
woodpecker-cli cron add --repository owner/repo --name nightly \
  --schedule "0 2 * * *" --branch main

# --- Agent ---
woodpecker-cli agent ls
```

### 14.4. Lint pipeline local (không cần push)

```bash
# Kiểm tra cú pháp file pipeline
woodpecker-cli lint .woodpecker.yml
woodpecker-cli lint --repo owner/repo .woodpecker.yml
```

---

## 15. Migration từ Drone sang Woodpecker

### 15.1. Khác biệt chính

| Hạng mục | Drone (`.drone.yml`) | Woodpecker (`.woodpecker.yml`) |
|---|---|---|
| Tên file | `.drone.yml` | `.woodpecker.yml` |
| Kind header | `kind: pipeline` + `type: docker` | Không cần (mặc định docker) |
| Trigger | `trigger:` | `when:` |
| Matrix | `matrix:` | `matrix:` (giống) |
| Depends on | `depends_on:` | `depends_on:` (giống) |
| Services | `services:` | `services:` (giống) |
| Secrets | `from_secret:` | `from_secret:` (giống) |
| Clone | `clone:` | `clone:` (giống) |
| Biến môi trường | `DRONE_*` | `CI_*` (xem bảng map bên dưới) |

### 15.2. Map biến môi trường Drone → Woodpecker

| Drone | Woodpecker |
|---|---|
| `DRONE_REPO` | `CI_REPO` |
| `DRONE_REPO_NAME` | `CI_REPO_NAME` |
| `DRONE_COMMIT_SHA` | `CI_COMMIT_SHA` |
| `DRONE_COMMIT_BRANCH` | `CI_COMMIT_BRANCH` |
| `DRONE_TAG` | `CI_COMMIT_TAG` |
| `DRONE_BUILD_NUMBER` | `CI_PIPELINE_NUMBER` |
| `DRONE_BUILD_EVENT` | `CI_PIPELINE_EVENT` |
| `DRONE_BUILD_STATUS` | `CI_PIPELINE_STATUS` |
| `DRONE_STEP_NAME` | `CI_STEP_NAME` |
| `DRONE_WORKSPACE` | `CI_WORKSPACE` |

### 15.3. Script chuyển đổi nhanh

```bash
# Đổi tên file
git mv .drone.yml .woodpecker.yml

# Thay thế hàng loạt trong file
sed -i \
  -e 's/^kind:.*//' \
  -e 's/^type:.*//' \
  -e 's/trigger:/when:/' \
  -e 's/DRONE_REPO_NAME/CI_REPO_NAME/g' \
  -e 's/DRONE_REPO\b/CI_REPO/g' \
  -e 's/DRONE_COMMIT_SHA/CI_COMMIT_SHA/g' \
  -e 's/DRONE_COMMIT_BRANCH/CI_COMMIT_BRANCH/g' \
  -e 's/DRONE_TAG/CI_COMMIT_TAG/g' \
  -e 's/DRONE_BUILD_NUMBER/CI_PIPELINE_NUMBER/g' \
  -e 's/DRONE_BUILD_EVENT/CI_PIPELINE_EVENT/g' \
  .woodpecker.yml
```

### 15.4. Ví dụ chuyển đổi

**Trước (Drone):**

```yaml
kind: pipeline
type: docker
name: default

trigger:
  branch: [main]
  event: [push]

steps:
  - name: test
    image: golang:1.23
    commands:
      - go test ./...
      - echo $DRONE_COMMIT_SHA
```

**Sau (Woodpecker):**

```yaml
when:
  - event: push
    branch: main

steps:
  test:
    image: golang:1.23
    commands:
      - go test ./...
      - echo $CI_COMMIT_SHA
```

### 15.5. Checklist migration

- [ ] Dựng Woodpecker server + agent, kết nối cùng forge
- [ ] Đổi tên `.drone.yml` → `.woodpecker.yml`, sửa `trigger` → `when`, map biến `DRONE_*` → `CI_*`
- [ ] Chạy `woodpecker-cli lint` kiểm tra cú pháp
- [ ] Tạo lại secrets trên Woodpecker (secret không migrate tự động)
- [ ] Deactivate repo trên Drone, activate trên Woodpecker (tránh double webhook)
- [ ] Kiểm tra plugin: plugin `drone/*` chạy được, nhưng nên đổi sang `woodpeckerci/*` khi có
- [ ] Chạy thử 1 pipeline end-to-end trước khi xóa Drone

---

## 16. Ví dụ thực tế hoàn chỉnh

### 16.1. Node.js app: test → build → docker push → deploy

```yaml
# .woodpecker.yml
when:
  - event: [push, pull_request, tag]

variables:
  IMAGE: registry.example.com/myorg/webapp

steps:
  install-test:
    image: node:20-alpine
    commands:
      - npm ci
      - npm run lint
      - npm test -- --coverage

  build:
    image: node:20-alpine
    commands:
      - npm ci
      - npm run build
    depends_on: [install-test]

  docker:
    image: woodpeckerci/plugin-docker-buildx
    privileged: true
    settings:
      repo: registry.example.com/myorg/webapp
      registry: registry.example.com
      username:
        from_secret: registry_user
      password:
        from_secret: registry_pass
      tags:
        - ${CI_COMMIT_SHA:0:8}
        - latest
      platforms: linux/amd64,linux/arm64
    depends_on: [build]
    when:
      - event: push
        branch: main
      - event: tag

  deploy:
    image: appleboy/drone-ssh
    settings:
      host:
        from_secret: prod_host
      username:
        from_secret: prod_user
      key:
        from_secret: prod_ssh_key
      script:
        - cd /opt/webapp
        - docker compose pull
        - docker compose up -d
        - docker image prune -f
    depends_on: [docker]
    when:
      - event: push
        branch: main
    secrets: []

  notify:
    image: woodpeckerci/plugin-telegram
    settings:
      token:
        from_secret: tg_token
      to:
        from_secret: tg_chat
      message: |
        🚀 Deploy webapp #{{build.number}}: {{build.status}}
        {{repo.name}} @ {{commit.branch}} ({{commit.sha | substr 0 8}})
    when:
      - event: [push, tag]
        status: [success, failure]
```

### 16.2. Python: test với matrix + publish PyPI

```yaml
when:
  - event: [push, pull_request, tag]

matrix:
  PYTHON: ["3.11", "3.12", "3.13"]

services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: testdb

steps:
  test:
    image: python:${PYTHON}-slim
    commands:
      - pip install -e ".[test]"
      - pytest -v --cov
    environment:
      DATABASE_URL: postgresql://test:test@postgres:5432/testdb

  lint:
    image: python:3.12-slim
    commands:
      - pip install ruff
      - ruff check .
      - ruff format --check .

  publish:
    image: woodpeckerci/plugin-pypi-publish
    settings:
      username: __token__
      password:
        from_secret: pypi_token
      repository: https://upload.pypi.org/legacy/
    when:
      - event: tag
```

### 16.3. Go: build multi-platform + release Gitea

```yaml
when:
  - event: [push, pull_request, tag]

steps:
  vet-test:
    image: golang:1.23-alpine
    commands:
      - go vet ./...
      - go test -race ./...

  build-all:
    image: golang:1.23-alpine
    commands:
      - GOOS=linux GOARCH=amd64 go build -o dist/app-linux-amd64 .
      - GOOS=linux GOARCH=arm64 go build -o dist/app-linux-arm64 .
      - GOOS=darwin GOARCH=arm64 go build -o dist/app-darwin-arm64 .
      - sha256sum dist/* > dist/checksums.txt

  release:
    image: woodpeckerci/plugin-gitea-release
    settings:
      base_url: https://gitea.example.com
      api_key:
        from_secret: gitea_token
      files: dist/*
      title: "${CI_COMMIT_TAG}"
      note: "Release ${CI_COMMIT_TAG} — build #${CI_PIPELINE_NUMBER}"
    when:
      - event: tag

  docker:
    image: woodpeckerci/plugin-docker-buildx
    privileged: true
    settings:
      repo: registry.example.com/myorg/goapp
      username:
        from_secret: registry_user
      password:
        from_secret: registry_pass
      tags: [${CI_COMMIT_TAG}, latest]
      platforms: linux/amd64,linux/arm64
    when:
      - event: tag
```

### 16.4. Cron job — chạy định kỳ (nightly build)

Tạo cron trên UI (repo → Settings → Cron) hoặc CLI:

```bash
woodpecker-cli cron add --repository owner/repo \
  --name nightly --schedule "0 2 * * *" --branch main
```

Pipeline phản ứng với cron:

```yaml
when:
  - event: cron
    # cron: [nightly]   # chỉ chạy cho cron tên "nightly"

steps:
  nightly-e2e:
    image: node:20-alpine
    commands:
      - npm ci
      - npm run test:e2e
```

---

## 17. Webhooks và badges

### 17.1. Webhook từ forge

Khi activate repo, Woodpecker tự đăng ký webhook trên forge:

| Forge | Webhook URL | Events |
|---|---|---|
| Gitea/Forgejo | `https://ci.example.com/hook` | push, pull_request, tag |
| GitHub | `https://ci.example.com/hook` | push, pull_request |
| GitLab | `https://ci.example.com/hook` | push, merge_request, tag |

Nếu webhook không về: kiểm tra repo → Settings → Webhooks trên forge, xem **Recent Deliveries** có lỗi gì (thường do TLS/firewall).

### 17.2. Status check báo về forge

Sau mỗi pipeline, Woodpecker đẩy commit status về forge với context mặc định `ci/woodpecker`. Đổi tên:

```yaml
# server env
- WOODPECKER_STATUS_CONTEXT=my-ci
```

Trên Gitea/GitHub: vào **Branch protection** → yêu cầu status check `ci/woodpecker` pass mới được merge.

### 17.3. Badge trạng thái

Gắn badge vào README:

```markdown
[![status](https://ci.example.com/api/badges/owner/repo/status.svg)](https://ci.example.com/owner/repo)
```

Các loại badge: `status.svg`, `cc.xml` (CCTray), `coverage.svg` (nếu có coverage).

### 17.4. Webhook đi (outgoing)

Dùng plugin webhook hoặc curl trong step để báo sang hệ thống khác:

```yaml
steps:
  ping-monitor:
    image: alpine:3.20
    commands:
      - apk add --no-cache curl
      - curl -fsS -m 10 --retry 3 https://hc.example.com/ping/xxx
    when:
      - event: cron
```

---

## 18. Backup, upgrade, bảo mật

### 18.1. Backup

**Những gì cần backup:**

| Dữ liệu | Vị trí | Cách backup |
|---|---|---|
| Database | volume `woodpecker-data` (SQLite) hoặc Postgres | snapshot volume / `pg_dump` |
| File `.env` | `~/woodpecker/.env` | copy an toàn (chứa secret!) |
| `docker-compose.yml` | `~/woodpecker/` | git repo riêng |

```bash
# Backup SQLite (dừng server trước để file không corrupt)
docker compose stop woodpecker-server
tar czf woodpecker-backup-$(date +%F).tar.gz -C ~/woodpecker data .env docker-compose.yml
docker compose start woodpecker-server

# Backup PostgreSQL
docker exec woodpecker-db pg_dump -U woodpecker woodpecker \
  | gzip > woodpecker-db-$(date +%F).sql.gz
```

### 18.2. Upgrade

```bash
cd ~/woodpecker
# 1. Backup trước (mục 18.1)
# 2. Đọc release notes: github.com/woodpecker-ci/woodpecker/releases
#    — chú ý "Breaking changes" và migration notes
# 3. Đổi tag image
sed -i 's/woodpeckerci\/woodpecker-server:.*/woodpeckerci\/woodpecker-server:3.9.0/' docker-compose.yml
sed -i 's/woodpeckerci\/woodpecker-agent:.*/woodpeckerci\/woodpecker-agent:3.9.0/' docker-compose.yml
# 4. Upgrade server TRƯỚC, agent SAU
docker compose pull
docker compose up -d woodpecker-server
docker compose logs -f woodpecker-server   # chờ "server running", kiểm tra migration DB
docker compose up -d woodpecker-agent
```

> Luôn giữ server và agent **cùng version** (hoặc agent cũ hơn 1 minor). Đọc kỹ migration guide khi nhảy major version.

### 18.3. Security best practices

- [ ] **HTTPS bắt buộc** cho `WOODPECKER_HOST` — webhook forge yêu cầu TLS.
- [ ] `WOODPECKER_AGENT_SECRET` dài, ngẫu nhiên (`openssl rand -hex 32`), rotate định kỳ.
- [ ] Port **9000 (gRPC)** không expose ra internet — chỉ agent tin cậy được kết nối (dùng firewall/VPN).
- [ ] **Trusted repo**: chỉ bật cho repo nội bộ tin cậy — repo trusted cho phép `privileged`, có thể mount docker socket và chiếm host agent.
- [ ] Secret giới hạn theo **event**: secret production không expose cho `pull_request`.
- [ ] Không chạy agent trên cùng host với service quan trọng khác nếu chạy pipeline từ contributor ngoài.
- [ ] Pin image version trong pipeline (`node:20-alpine` thay vì `node:latest`) để build tái lập.
- [ ] Bật `WOODPECKER_OPEN=false` nếu không muốn public pipeline.
- [ ] Audit log: theo dõi ai activate repo, ai sửa secret (UI → admin panel).
- [ ] Thường xuyên `docker image prune` trên agent để tránh disk full — hoặc cron dọn dẹp.
- [ ] Dùng `WOODPECKER_PLUGINS_PRIVILEGED` whitelist thay vì bật trusted tràn lan.

---

## 19. Troubleshooting

| Triệu chứng | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| Pipeline stuck ở "pending" mãi | Không có agent nào khớp labels / agent offline | Kiểm tra agent đang chạy, `WOODPECKER_AGENT_LABELS` khớp `labels:` trong YAML |
| `cannot connect to server` ở agent | Sai `WOODPECKER_SERVER` / secret / firewall chặn 9000 | Kiểm tra secret khớp 2 phía, telnet port 9000 |
| Webhook không kích hoạt pipeline | Webhook forge lỗi / sai URL / repo chưa activate | Xem Recent Deliveries trên forge; kiểm tra `WOODPECKER_HOST` đúng public URL |
| Login OAuth báo redirect_uri_mismatch | Redirect URI đăng ký sai | Phải đúng `https://ci.example.com/authorize` |
| Step báo `image not found` | Sai tên/tag image, registry private chưa auth | Kiểm tra tên image; thêm registry credentials (UI/CLI) |
| Build Docker trong step lỗi `permission denied` | Thiếu `privileged: true` / repo chưa Trusted | Bật Trusted cho repo, thêm `privileged: true` |
| Secret không có trong step | Quên khai `secrets:` / sai tên / secret giới hạn event | Khai báo `secrets: [ten]`, kiểm tra event filter của secret |
| Log hiện `********` | Bình thường — credential filtering | Không phải lỗi |
| `when: path` không trigger | Pattern sai / file đổi không khớp | Test pattern glob, nhớ `path` chỉ áp dụng cho event push/PR |
| Matrix tạo quá nhiều job | Tổ hợp bùng nổ | Giảm trục matrix, dùng `when` loại bớt |
| Disk agent đầy | Image/layer Docker tồn đọng | `docker system prune -af --volumes` (cẩn thận), đặt cron dọn |
| Pipeline fail ngay step clone | Repo private / token hết hạn / LFS | Re-login Woodpecker để refresh token; kiểm tra deploy key |
| UI load chậm, DB phình | SQLite quá tải / log pipeline lớn | Chuyển PostgreSQL; giảm retention log |
| Plugin `from_secret` báo empty | Secret chưa tạo ở đúng repo/org | Tạo secret đúng scope repo, đúng tên |
| Cron không chạy | Sai cron expression / agent offline lúc đó | Test expression tại crontab.guru; kiểm tra cron list |

---

## 20. FAQ

**Hỏi: Woodpecker có miễn phí hoàn toàn không?**
Đáp: Có. Apache-2.0, không có bản enterprise trả phí, không giới hạn số repo/pipeline/user. Chi phí duy nhất là server bạn tự chạy.

**Hỏi: Chạy được trên Raspberry Pi / ARM không?**
Đáp: Được. Image official có build `linux/arm64`. Gắn label `platform=linux/arm64` cho agent Pi và pipeline yêu cầu label đó.

**Hỏi: Có chạy pipeline local để debug không?**
Đáp: Có 2 cách: (1) `woodpecker-cli lint` kiểm tra cú pháp; (2) chạy agent backend `local` trên máy dev. Không có chế độ "dry-run" full như `act` của GitHub Actions — cách thực tế nhất là push lên branch test.

**Hỏi: Một repo dùng nhiều file pipeline được không?**
Đáp: Được. Vào repo Settings → đổi **Config file**, hoặc dùng nhiều workflow trong 1 file cách nhau bằng `---`.

**Hỏi: Woodpecker có hỗ trợ monorepo không?**
Đáp: Có, bằng `when: path:` — mỗi service một workflow, filter theo thư mục đổi. Ví dụ workflow A chỉ chạy khi `services/a/**` đổi.

**Hỏi: Phân biệt `event: manual` và bấm "Restart"?**
Đáp: `manual` là trigger pipeline mới từ UI/API (event = manual). Restart là chạy lại pipeline cũ giữ nguyên event ban đầu.

**Hỏi: Có thể dùng Woodpecker mà không cần Docker trên agent?**
Đáp: Có, backend `local` — nhưng mất lợi ích isolation của container. Chỉ nên dùng cho trường hợp đặc biệt.

**Hỏi: Pipeline của PR từ fork có an toàn không?**
Đáp: Mặc định secret **không** expose cho PR từ fork nếu bạn giới hạn event của secret. Nên bật thêm "Protected" hoặc yêu cầu approve cho repo public.

**Hỏi: Woodpecker scale được bao nhiêu agent?**
Đáp: Không giới hạn cứng — thêm agent là tăng throughput. Server Go nhẹ, hàng chục agent trên 1 server 2 CPU vẫn ổn.

**Hỏi: Khác gì khi dùng plugin Drone cũ?**
Đáp: Chạy được ngay vì tương thích config. Nhưng plugin `woodpeckerci/*` được maintain tích cực hơn, nên ưu tiên.

---

## 21. Cheat sheet

### File `.woodpecker.yml` tối thiểu

```yaml
when:
  - event: [push, pull_request]
steps:
  test:
    image: alpine:3.20
    commands: [echo hello]
```

### Cấu trúc đầy đủ nhớ nhanh

```yaml
variables: {}        # biến toàn cục
when: []             # filter toàn cục (event/branch/path/...)
clone: {}            # tùy chỉnh git clone
services: {}         # sidecar containers
workspace: {}        # tùy chỉnh workspace
matrix: {}           # build matrix
labels: {}           # yêu cầu agent labels
depends_on: []       # DAG giữa workflows (ngăn cách bởi ---)
steps:               # các step tuần tự
  ten-step:
    image:           # bắt buộc
    commands: []     # lệnh shell
    environment: {}  # env tĩnh
    secrets: []      # secret từ Woodpecker
    when: []         # filter riêng step
    depends_on: []   # phụ thuộc step
    failure: ignore  # bỏ qua lỗi
    privileged: true # cần repo Trusted
    detach: true     # chạy nền
```

### CLI nhớ nhanh

```bash
export WOODPECKER_SERVER=https://ci.example.com WOODPECKER_TOKEN=xxx
woodpecker-cli repo ls
woodpecker-cli pipeline ls owner/repo
woodpecker-cli pipeline logs owner/repo 42
woodpecker-cli secret add --repository owner/repo --name k --value v
woodpecker-cli lint .woodpecker.yml
```

### Docker Compose nhớ nhanh

```bash
docker compose up -d
docker compose logs -f woodpecker-server
docker compose pull && docker compose up -d   # upgrade
```

### Biến `CI_*` hay dùng

`CI_REPO` · `CI_COMMIT_SHA` · `CI_COMMIT_BRANCH` · `CI_COMMIT_TAG` · `CI_COMMIT_MESSAGE` · `CI_PIPELINE_NUMBER` · `CI_PIPELINE_EVENT` · `CI_PIPELINE_URL` · `CI_STEP_NAME` · `CI_WORKSPACE`

---

## 22. Vì sao chọn Woodpecker?

1. **Thực sự open source** — Apache-2.0, cộng đồng điều hành, không công ty nào có thể đổi giấy phép hay khóa tính năng sau lưng bạn (bài học từ Drone → PolyForm).
2. **Nhẹ đến khó tin** — server + agent là binary Go vài chục MB, chạy ngon trên VPS 1 GB RAM; không Java, không daemon nặng.
3. **Bạn của self-hosted forge** — hỗ trợ Gitea/Forgejo native tốt nhất trong các CI hiện nay, đúng triết lý "hạ tầng của mình".
4. **Pipeline-as-code đơn giản** — YAML dễ đọc hơn Jenkinsfile Groovy, chạy trong container nên môi trường sạch và tái lập.
5. **Tương thích hệ sinh thái Drone** — plugin Drone cũ chạy được ngay, migration chỉ là đổi tên file + map biến.
6. **Không vendor lock-in** — config YAML chuẩn, agent tự host, muốn dời đi đâu cũng được; không bị khóa vào cloud của ai.
7. **Chi phí = 0 đồng license** — trả duy nhất tiền server bạn đã có.

> Tóm lại: nếu bạn self-host code trên Gitea/Forgejo và muốn một CI nhẹ, trung thực với tinh thần open source, chạy pipeline trong Docker container bằng file YAML gọn gàng — Woodpecker CI là lựa chọn hợp lý nhất hiện nay.

---

*Tài liệu tham khảo chính: woodpecker-ci.org/docs · github.com/woodpecker-ci/woodpecker (releases, plugin registry). Kiểm tra version mới nhất trước khi áp dụng vì cú pháp có thể thay đổi giữa các major release.*
