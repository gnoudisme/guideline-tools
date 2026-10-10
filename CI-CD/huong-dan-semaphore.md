# 🚦 Hướng Dẫn Toàn Tập: Semaphore CI/CD

> **Cấp độ:** Người mới bắt đầu → cấu hình được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật lần cuối:** 10/10/2026
>
> ⚠️ **Ghi chú:** Semaphore là cloud CI/CD, nổi tiếng với tốc độ nhanh và pricing đơn giản. Phiên bản trong guide: Semaphore 2.0.

---

## 📑 Mục Lục

1. [Semaphore Là Gì?](#1-semaphore-là-gì)
2. [Semaphore vs GitHub Actions vs CircleCI](#2-semaphore-vs-github-actions-vs-circleci)
3. [Đăng Ký & Kết Nối GitHub](#3-đăng-ký--kết-nối-github)
4. [Tạo Project Đầu Tiên](#4-tạo-project-đầu-tiên)
5. [.semaphore/semaphore.yml: Cấu Trúc](#5-semaphoresemaphoreyml-cấu-trúc)
6. [Pipelines, Blocks & Jobs](#6-pipelines-blocks--jobs)
7. [Run Blocks: Khi Nào Chạy](#7-run-blocks-khi-nào-chạy)
8. [Promotions: Manual & Automatic](#8-promotions-manual--automatic)
9. [Secrets: Quản Lý An Toàn](#9-secrets-quản-lý-an-toàn)
10. [Environment Variables](#10-environment-variables)
11. [Caches: Tăng Tốc Build](#11-caches-tăng-tốc-build)
12. [Artifacts: Lưu & Tải](#12-artifacts-lưu--tải)
13. [Test Reports & Flaky Tests](#13-test-reports--flaky-tests)
14. [Monorepo: Changed-based Pipelines](#14-monorepo-changed-based-pipelines)
15. [Self-hosted Agents](#15-self-hosted-agents)
16. [Docker Trong Semaphore](#16-docker-trong-semaphore)
17. [Dự Án Thực Tế: Node.js Full Pipeline](#17-dự-án-thực-tế-nodejs-full-pipeline)
18. [Dự Án Thực Tế: Python + Docker](#18-dự-án-thực-tế-python--docker)
19. [Scheduled Pipelines (Cron)](#19-scheduled-pipelines-cron)
20. [Notifications: Slack, Email](#20-notifications-slack-email)
21. [Semaphore CLI (sem)](#21-semaphore-cli-sem)
22. [API](#22-api)
23. [Pricing](#23-pricing)
24. [Troubleshooting](#24-troubleshooting)
25. [Best Practices Checklist](#25-best-practices-checklist)
26. [Cheat Sheet](#26-cheat-sheet)
27. [FAQ](#27-faq)

---

## 1. Semaphore Là Gì?

**Semaphore** (Semaphore 2.0) là nền tảng CI/CD trên cloud, được biết đến với:

- ⚡ **Tốc độ cực nhanh** — parallel jobs mạnh mẽ, machine khởi động nhanh
- 🎯 **Cú pháp đơn giản** — file `.semaphore/semaphore.yml` dễ đọc
- 🔄 **Promotions** — khái niệm độc đáo: pipeline nối pipeline
- 📊 **Test reports** — tự động phát hiện flaky tests

### Luồng hoạt động

```
Push code → GitHub webhook → Semaphore
                                │
                                ▼
                    ┌───────────────────────┐
                    │ Pipeline: CI          │
                    │ ┌─────┐ ┌─────┐       │
                    │ │Block│ │Block│       │
                    │ │Lint │ │Test │       │
                    │ └──┬──┘ └──┬──┘       │
                    │    └────┬───┘         │
                    │         ▼             │
                    │    ┌─────────┐        │
                    │    │Promotion│──▶ Pipeline: Deploy
                    │    │(manual) │        │
                    │    └─────────┘        │
                    └───────────────────────┘
```

### Điểm mạnh

| Đặc điểm | Giải thích |
|----------|------------|
| **Tốc độ** | Một trong những CI nhanh nhất (parallel + machine mạnh) |
| **Promotions** | Tách CI và CD thành pipelines riêng, nối bằng promotion |
| **Secrets native** | Quản lý secrets ngay trong UI, mã hóa |
| **Monorepo support** | Change-based pipeline: chỉ chạy phần bị thay đổi |
| **Self-hosted agents** | Chạy agent trên hạ tầng của bạn nếu cần |

### Khi nào nên dùng Semaphore?

- Muốn CI cloud nhanh, không maintain server
- Pipeline phức tạp cần tách CI/CD rõ ràng
- Monorepo cần chạy selective builds
- Team vừa và nhỏ muốn setup nhanh

### Khi nào KHÔNG nên dùng?

- Muốn free hoàn toàn không giới hạn → Drone/Jenkins self-hosted
- Cần Windows builds (Semaphore chủ yếu Linux, macOS có nhưng đắt)
- Đã đầu tư sâu vào GitHub Actions ecosystem

---

## 2. Semaphore vs GitHub Actions vs CircleCI

| Tiêu chí | Semaphore | GitHub Actions | CircleCI |
|----------|-----------|----------------|----------|
| Mô hình | Cloud | Cloud | Cloud |
| Cấu hình | `.semaphore/*.yml` | `.github/workflows/*.yml` | `.circleci/config.yml` |
| Tốc độ | Rất nhanh | Nhanh | Rất nhanh |
| Free tier | $20 credit/tháng | 2.000 phút/tháng | 6.000 phút/tháng |
| Parallel jobs | Mạnh (tùy plan) | Matrix | Mạnh |
| Khái niệm đặc biệt | Promotions | Reusable workflows | Orbs |
| Self-hosted | Có (agents) | Có (runners) | Có (runners) |
| Độ khó | Dễ | Dễ | Trung bình |

---

## 3. Đăng Ký & Kết Nối GitHub

### Bước 1: Đăng ký

1. Vào [semaphoreci.com](https://semaphoreci.com) → **Start Free**
2. Đăng ký bằng **GitHub** (khuyên dùng) hoặc Bitbucket
3. Authorize Semaphore truy cập repos

### Bước 2: Tạo Organization

1. Sau khi đăng nhập → tạo organization (ví dụ: `my-company`)
2. Chọn plan (bắt đầu với **Free** — $20 credit/tháng)

### Bước 3: Cấp quyền repo

1. GitHub → **Settings** → **Applications** → **Semaphore**
2. Cấp quyền cho organization/repos cần CI

---

## 4. Tạo Project Đầu Tiên

### Cách 1: Qua Web UI (khuyên dùng cho lần đầu)

1. Vào Semaphore → **Create New Project**
2. Chọn **GitHub** → chọn repo
3. Semaphore hỏi: **"How do you want to set up?"**
   - Chọn **"I want to add a custom pipeline"** (tự viết YAML)
   - Hoặc chọn template (Node.js, Python, Go...)
4. Nếu chọn custom → Semaphore tạo file `.semaphore/semaphore.yml` mẫu
5. Commit file vào repo → push → pipeline tự chạy!

### Cách 2: Tạo file thủ công

Tạo file `.semaphore/semaphore.yml` trong repo:

```yaml
# .semaphore/semaphore.yml
version: v1.0
name: Hello Semaphore

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

blocks:
  - name: "Hello"
    run:
      when: true
    task:
      jobs:
        - name: Say hello
          commands:
            - echo "Semaphore hoạt động rồi!"
            - sem-version
```

Push → vào Semaphore xem pipeline chạy!

### Machine types

| Type | CPU | RAM | Dùng cho |
|------|-----|-----|----------|
| `e1-standard-2` | 2 | 4 GB | Builds thông thường |
| `e1-standard-4` | 4 | 8 GB | Builds nặng |
| `e1-standard-8` | 8 | 16 GB | Builds rất nặng |
| `a1-standard-4` | 4 (ARM) | 8 GB | ARM builds |

---

## 5. .semaphore/semaphore.yml: Cấu Trúc

### Cấu trúc đầy đủ

```yaml
version: v1.0                    # version (bắt buộc)
name: My Pipeline                # tên pipeline

agent:                           # máy chạy
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

# Biến môi trường toàn cục
env_vars:
  - name: NODE_ENV
    value: production

# Tự động hủy build cũ khi có push mới (khuyên bật)
auto_cancel:
  running:
    when: "true"
  queued:
    when: "true"

# Thất bại nhanh: dừng pipeline khi 1 job fail
fail_fast:
  stop:
    when: "true"

# Các khối công việc
blocks:
  - name: "Build"
    run:
      when: true
    task:
      jobs:
        - name: Build app
          commands:
            - npm ci
            - npm run build

# Pipeline tiếp theo (CD)
promotions:
  - name: Deploy to staging
    pipeline_file: staging-deploy.yml
    auto_promote:
      when: "result = 'passed' and branch = 'main'"
```

### Các trường quan trọng

| Trường | Ý nghĩa |
|--------|---------|
| `version` | Luôn là `v1.0` |
| `agent.machine.type` | Loại máy (CPU/RAM) |
| `agent.machine.os_image` | OS image (`ubuntu2204`) |
| `blocks` | Danh sách các block (chạy tuần tự) |
| `blocks[].task.jobs` | Các job trong block (chạy song song) |
| `promotions` | Pipeline tiếp theo |
| `auto_cancel` | Tự hủy build cũ |
| `fail_fast` | Dừng ngay khi fail |

---

## 6. Pipelines, Blocks & Jobs

### Quan hệ: Pipeline → Block → Job

```
Pipeline: "CI"
├── Block: "Install" (chạy đầu tiên)
│   └── Job: "npm ci"
│
├── Block: "Test" (chạy sau Install)
│   ├── Job: "Unit tests"      ← chạy song song
│   ├── Job: "Lint"            ← chạy song song
│   └── Job: "Type check"      ← chạy song song
│
└── Block: "Build" (chạy sau Test)
    └── Job: "Build app"
```

- **Blocks** chạy **tuần tự** (block sau đợi block trước xong)
- **Jobs** trong cùng block chạy **song song**

### Ví dụ chi tiết

```yaml
version: v1.0
name: Node.js CI

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

blocks:
  # Block 1: Cài đặt
  - name: "Install dependencies"
    task:
      jobs:
        - name: npm ci
          commands:
            - npm ci
            # Cache node_modules cho blocks sau
            - cache store node-modules-$SEMAPHORE_GIT_SHA node_modules

  # Block 2: Kiểm tra (3 jobs song song)
  - name: "Checks"
    task:
      jobs:
        - name: Lint
          commands:
            - cache restore node-modules-$SEMAPHORE_GIT_SHA
            - npm run lint

        - name: Unit tests
          commands:
            - cache restore node-modules-$SEMAPHORE_GIT_SHA
            - npm test -- --ci

        - name: Type check
          commands:
            - cache restore node-modules-$SEMAPHORE_GIT_SHA
            - npm run typecheck

  # Block 3: Build
  - name: "Build"
    run:
      when: "branch = 'main'"
    task:
      jobs:
        - name: Build app
          commands:
            - cache restore node-modules-$SEMAPHORE_GIT_SHA
            - npm run build
```

### Job với nhiều commands

```yaml
jobs:
  - name: Full check
    commands:
      - echo "Bắt đầu..."
      - npm ci
      - npm run lint
      - npm test
      - echo "Xong!"
```

---

## 7. Run Blocks: Khi Nào Chạy

### Điều kiện chạy block

```yaml
blocks:
  - name: "Deploy"
    run:
      when: "branch = 'main' and result = 'passed'"
    task:
      jobs:
        - name: Deploy
          commands:
            - ./deploy.sh
```

### Các điều kiện phổ biến

```yaml
# Chỉ chạy trên main
run:
  when: "branch = 'main'"

# Chỉ chạy cho pull request
run:
  when: "pull_request != ''"

# Chỉ chạy khi tag
run:
  when: "tag =~ '^v.*'"

# Chạy khi file thay đổi
run:
  when: "change_in('/frontend/', {default_branch: 'main'})"

# Kết hợp
run:
  when: "branch = 'main' and change_in('/backend/')"

# Không bao giờ chạy tự động (chỉ manual)
run:
  when: "false"
```

### Biến có sẵn trong điều kiện

| Biến | Ý nghĩa |
|------|---------|
| `branch` | Tên branch hiện tại |
| `tag` | Tên tag (nếu là tag build) |
| `pull_request` | Số PR (rỗng nếu không phải PR) |
| `result` | Kết quả pipeline (`passed`/`failed`) |
| `change_in(path)` | `true` nếu file trong path thay đổi |

---

## 8. Promotions: Manual & Automatic

**Promotion** là cách nối pipeline CI với pipeline CD — khái niệm đặc trưng của Semaphore.

### Cấu trúc

```
.semaphore/
├── semaphore.yml          # Pipeline CI (chạy khi push)
├── staging-deploy.yml     # Pipeline deploy staging
└── production-deploy.yml  # Pipeline deploy production
```

### Tự động promote (auto_promote)

```yaml
# .semaphore/semaphore.yml (CI pipeline)
promotions:
  - name: Deploy to staging
    pipeline_file: staging-deploy.yml
    auto_promote:
      when: "result = 'passed' and branch = 'main'"
```

→ Khi CI pass trên main → tự động chạy staging-deploy.yml

### Manual promote (chờ bấm nút)

```yaml
promotions:
  - name: Deploy to production
    pipeline_file: production-deploy.yml
    # Không có auto_promote → chờ bấm "Promote" thủ công trong UI
```

### Truyền parameters giữa pipelines

```yaml
# Trong semaphore.yml:
promotions:
  - name: Deploy
    pipeline_file: deploy.yml
    auto_promote:
      when: "result = 'passed'"
    parameter:
      - name: IMAGE_TAG
        value: "$SEMAPHORE_WORKFLOW_ID"
      - name: DEPLOY_ENV
        value: "staging"
```

```yaml
# Trong deploy.yml, dùng parameter:
blocks:
  - name: "Deploy"
    task:
      jobs:
        - name: Deploy app
          commands:
            - echo "Deploying image tag: $IMAGE_TAG to $DEPLOY_ENV"
```

### Chuỗi promotions

```yaml
# semaphore.yml → staging-deploy.yml → production-deploy.yml

# staging-deploy.yml:
promotions:
  - name: Promote to production
    pipeline_file: production-deploy.yml
    # Manual: chờ bấm nút sau khi kiểm tra staging
```

---

## 9. Secrets: Quản Lý An Toàn

### Tạo secret trong UI

1. Project → **Secrets** (sidebar) → **New Secret**
2. Đặt tên: ví dụ `docker-hub`
3. Thêm các biến:
   - `DOCKER_USERNAME` = `myuser`
   - `DOCKER_PASSWORD` = `xxx` (đánh dấu là secret)
4. Bấm **Create**

### Dùng secret trong pipeline

```yaml
blocks:
  - name: "Docker push"
    task:
      # Gắn secret vào task
      secrets:
        - name: docker-hub
      jobs:
        - name: Push image
          commands:
            - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            - docker push myuser/myapp:latest
```

### Tạo secret bằng CLI

```bash
# Cài sem CLI (xem phần 21)
sem create secret docker-hub \
  -e DOCKER_USERNAME=myuser \
  -e DOCKER_PASSWORD=xxx
```

### Lưu ý bảo mật

- Secrets được **mã hóa** khi lưu trữ
- Giá trị secrets **không hiện** trong log (che thành `[hidden]`)
- Chỉ project owner/admin mới xem/sửa secrets
- **Không** commit secrets vào `.semaphore/*.yml`!

---

## 10. Environment Variables

### 3 cấp độ

```yaml
# 1. Pipeline-level (áp dụng mọi job)
env_vars:
  - name: NODE_ENV
    value: production
  - name: API_URL
    value: https://api.example.com

blocks:
  - name: "Build"
    task:
      # 2. Task-level (áp dụng mọi job trong task)
      env_vars:
        - name: BUILD_MODE
          value: release
      jobs:
        - name: Build
          # 3. Job-level (chỉ job này)
          env_vars:
            - name: JOB_SPECIFIC
              value: "true"
          commands:
            - echo $NODE_ENV $BUILD_MODE $JOB_SPECIFIC
```

### Biến có sẵn của Semaphore

| Biến | Ý nghĩa |
|------|---------|
| `SEMAPHORE_PROJECT_NAME` | Tên project |
| `SEMAPHORE_BRANCH` | Branch hiện tại |
| `SEMAPHORE_SHA` | Commit SHA |
| `SEMAPHORE_WORKFLOW_ID` | ID của workflow |
| `SEMAPHORE_JOB_ID` | ID của job |
| `SEMAPHORE_GIT_URL` | URL của repo |
| `SEMAPHORE_PULL_REQUEST_NUMBER` | Số PR |

---

## 11. Caches: Tăng Tốc Build

### Cache dependencies

```yaml
blocks:
  - name: "Install"
    task:
      jobs:
        - name: npm ci
          commands:
            # Thử restore cache trước
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH || npm ci
            - cache store node-modules-$SEMAPHORE_GIT_BRANCH node_modules
```

### Cache key strategies

```bash
# Cache theo branch (đơn giản)
cache store my-cache-$SEMAPHORE_GIT_BRANCH node_modules

# Cache theo lock file (chính xác hơn)
CHECKSUM=$(sha256sum package-lock.json | cut -d' ' -f1)
cache restore node-modules-$CHECKSUM || npm ci
cache store node-modules-$CHECKSUM node_modules

# Cache nhiều thư mục
cache store my-cache dir1,dir2,dir3
```

### Xóa cache

```bash
# Trong job:
cache delete node-modules-main

# Hoặc qua CLI:
sem cache delete --project my-project node-modules-main
```

---

## 12. Artifacts: Lưu & Tải

### Upload artifacts

```yaml
blocks:
  - name: "Build"
    task:
      jobs:
        - name: Build
          commands:
            - npm run build
          # Tự động upload sau khi job xong
          artifacts:
            - name: dist
              path: dist/
```

### Download trong job khác

```yaml
blocks:
  - name: "Deploy"
    task:
      jobs:
        - name: Deploy
          commands:
            - artifact pull dist/
            - ls dist/
```

### Artifact retention

Mặc định artifacts lưu **30 ngày**. Cấu hình trong Project Settings.

---

## 13. Test Reports & Flaky Tests

### JUnit test reports

```yaml
blocks:
  - name: "Test"
    task:
      jobs:
        - name: Run tests
          commands:
            # Tạo JUnit XML report
            - npm test -- --reporters=default --reporters=jest-junit
          artifacts:
            - name: test-results
              path: junit.xml
```

Semaphore tự parse JUnit XML và hiển thị:
- ✅/❌ từng test case
- ⏱️ Thời gian chạy từng test
- 🔄 **Flaky test detection** — tự động phát hiện test lúc pass lúc fail

### Xem test reports

Vào build → tab **Tests** → xem danh sách test cases, filter theo status.

---

## 14. Monorepo: Changed-based Pipelines

### Chỉ chạy khi thư mục thay đổi

```yaml
blocks:
  - name: "Frontend"
    run:
      when: "change_in('/frontend/', {default_branch: 'main'})"
    task:
      jobs:
        - name: Build frontend
          commands:
            - cd frontend && npm ci && npm run build

  - name: "Backend"
    run:
      when: "change_in('/backend/', {default_branch: 'main'})"
    task:
      jobs:
        - name: Test backend
          commands:
            - cd backend && npm ci && npm test
```

### change_in với nhiều options

```yaml
run:
  when: "change_in(['/frontend/', '/shared/'], {default_branch: 'main', on_tags: false})"
```

---

## 15. Self-hosted Agents

Chạy agent trên hạ tầng của bạn (khi cần phần cứng đặc biệt hoặc compliance).

### Cài đặt agent

```bash
# 1. Tạo agent type trong UI:
#    Organization → Self-hosted Agents → New Agent Type
#    Đặt tên: my-agents → copy registration token

# 2. Trên máy của bạn:
curl -L https://github.com/semaphoreci/agent/releases/latest/download/agent_Linux_x86_64.tar.gz | tar xz

# 3. Cấu hình
cat > config.yaml <<EOF
endpoint: "wss://agent.semaphoreci.com"
token: "your-registration-token"
agent_type: "my-agents"
EOF

# 4. Chạy
./agent start --config config.yaml
```

### Dùng self-hosted agent trong pipeline

```yaml
agent:
  machine:
    type: my-agents  # ← tên agent type đã tạo
```

---

## 16. Docker Trong Semaphore

### Build Docker image

```yaml
blocks:
  - name: "Docker build"
    task:
      secrets:
        - name: docker-hub
      jobs:
        - name: Build and push
          commands:
            - docker build -t myuser/myapp:$SEMAPHORE_WORKFLOW_ID .
            - docker tag myuser/myapp:$SEMAPHORE_WORKFLOW_ID myuser/myapp:latest
            - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            - docker push myuser/myapp:$SEMAPHORE_WORKFLOW_ID
            - docker push myuser/myapp:latest
```

### Docker Compose cho integration test

```yaml
blocks:
  - name: "Integration test"
    task:
      jobs:
        - name: Test with DB
          commands:
            - docker compose -f docker-compose.test.yml up -d
            - sleep 10  # đợi DB sẵn sàng
            - npm run test:integration
            - docker compose -f docker-compose.test.yml down -v
```

---

## 17. Dự Án Thực Tế: Node.js Full Pipeline

### Kịch bản

App Express. Pipeline CI: **Install → Lint + Test (song song) → Build**. Promote: **Staging (auto) → Production (manual)**.

### .semaphore/semaphore.yml

```yaml
version: v1.0
name: Node.js CI

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

auto_cancel:
  running:
    when: "true"
  queued:
    when: "true"

fail_fast:
  stop:
    when: "true"

blocks:
  - name: "Install"
    task:
      jobs:
        - name: npm ci
          commands:
            - checkout
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json) || npm ci
            - cache store node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json) node_modules

  - name: "Checks"
    task:
      jobs:
        - name: ESLint
          commands:
            - checkout
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json)
            - npm run lint

        - name: Unit tests
          commands:
            - checkout
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json)
            - npm test -- --ci --reporters=default --reporters=jest-junit
          artifacts:
            - name: junit
              path: junit.xml

        - name: Type check
          commands:
            - checkout
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json)
            - npm run typecheck

  - name: "Build"
    run:
      when: "branch = 'main'"
    task:
      jobs:
        - name: Build app
          commands:
            - checkout
            - cache restore node-modules-$SEMAPHORE_GIT_BRANCH-$(checksum package-lock.json)
            - npm run build
          artifacts:
            - name: dist
              path: dist/

promotions:
  - name: Deploy to staging
    pipeline_file: staging-deploy.yml
    auto_promote:
      when: "result = 'passed' and branch = 'main'"
```

### .semaphore/staging-deploy.yml

```yaml
version: v1.0
name: Deploy to Staging

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

blocks:
  - name: "Deploy"
    task:
      secrets:
        - name: staging-secrets
      jobs:
        - name: Deploy to staging
          commands:
            - checkout
            - artifact pull dist/
            - ./scripts/deploy.sh staging

promotions:
  - name: Deploy to production
    pipeline_file: production-deploy.yml
    # Manual: chờ bấm nút trong UI
```

### .semaphore/production-deploy.yml

```yaml
version: v1.0
name: Deploy to Production

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

blocks:
  - name: "Pre-flight checks"
    task:
      jobs:
        - name: Health check staging
          commands:
            - curl -f https://staging.example.com/health

  - name: "Deploy"
    task:
      secrets:
        - name: production-secrets
      jobs:
        - name: Deploy to production
          commands:
            - checkout
            - ./scripts/deploy.sh production

  - name: "Verify"
    task:
      jobs:
        - name: Smoke test
          commands:
            - curl -f https://example.com/health
            - npm run test:smoke
```

---

## 18. Dự Án Thực Tế: Python + Docker

### .semaphore/semaphore.yml

```yaml
version: v1.0
name: Python CI

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

blocks:
  - name: "Lint & Test"
    task:
      jobs:
        - name: Ruff + Pytest
          commands:
            - checkout
            - python -m pip install -r requirements.txt
            - python -m pip install ruff pytest
            - ruff check src/
            - pytest tests/ -v --junitxml=junit.xml
          artifacts:
            - name: junit
              path: junit.xml

  - name: "Docker Build"
    run:
      when: "branch = 'main'"
    task:
      secrets:
        - name: docker-hub
      jobs:
        - name: Build & Push
          commands:
            - checkout
            - docker build -t myuser/fastapi:$SEMAPHORE_WORKFLOW_ID .
            - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            - docker push myuser/fastapi:$SEMAPHORE_WORKFLOW_ID

promotions:
  - name: Deploy
    pipeline_file: deploy.yml
    auto_promote:
      when: "result = 'passed' and branch = 'main'"
```

---

## 19. Scheduled Pipelines (Cron)

### Tạo schedule

1. Project → **Settings** → **Schedules** → **New**
2. Điền:
   - **Name:** `Nightly E2E`
   - **Cron:** `0 2 * * *` (2h sáng)
   - **Branch:** `main`
   - **Pipeline file:** `semaphore.yml` (hoặc file riêng)

### Phân biệt scheduled vs normal

```yaml
blocks:
  - name: "Nightly E2E"
    run:
      when: "true"  # luôn chạy, nhưng pipeline file riêng
    task:
      jobs:
        - name: E2E tests
          commands:
            - npm run test:e2e
```

> 💡 Tạo file `.semaphore/nightly.yml` riêng cho scheduled pipeline để tách biệt.

---

## 20. Notifications: Slack, Email

### Slack

1. Project → **Settings** → **Notifications** → **Add Slack**
2. Nhập Slack webhook URL
3. Chọn events: build passed/failed

Hoặc trong pipeline:

```yaml
# Cuối semaphore.yml
# (Semaphore 2.0 dùng UI để cấu hình notifications)
```

### Gửi Slack từ job

```yaml
blocks:
  - name: "Notify"
    run:
      when: "result = 'failed'"
    task:
      secrets:
        - name: slack
      jobs:
        - name: Send alert
          commands:
            - |
              curl -X POST "$SLACK_WEBHOOK" \
                -H "Content-Type: application/json" \
                -d "{\"text\":\"❌ Build failed: $SEMAPHORE_PROJECT_NAME #$SEMAPHORE_WORKFLOW_ID\"}"
```

---

## 21. Semaphore CLI (sem)

### Cài đặt

```bash
# Linux
curl -L https://github.com/semaphoreci/cli/releases/latest/download/sem_Linux_x86_64.tar.gz | tar xz
sudo mv sem /usr/local/bin/

# macOS
brew install semaphoreci/tap/sem

# Kết nối
sem connect https://my-org.semaphoreci.com your-api-token
```

### Lệnh thường dùng

```bash
# Xem projects
sem get projects

# Xem pipelines
sem get pipelines --project my-project

# Xem workflows (builds)
sem get workflows --project my-project

# Trigger pipeline thủ công
sem create workflow --project my-project --branch main

# Quản lý secrets
sem create secret my-secret -e KEY=value --project my-project
sem get secrets --project my-project
sem delete secret my-secret --project my-project

# Xem logs
sem logs <workflow-id>

# Dừng workflow
sem stop workflow <workflow-id>
```

---

## 22. API

```bash
# Tạo API token: Organization → Settings → API

# List projects
curl -H "Authorization: Token $API_TOKEN" \
  "https://my-org.semaphoreci.com/api/v1alpha/projects"

# Trigger pipeline
curl -X POST \
  -H "Authorization: Token $API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"branch":"main","pipeline_file":".semaphore/semaphore.yml"}' \
  "https://my-org.semaphoreci.com/api/v1alpha/pipelines"
```

---

## 23. Pricing

| Gói | Giá | Bao gồm |
|-----|-----|---------|
| **Free** | $0 | $20 credit/tháng (~400 phút e1-standard-2) |
| **Starter** | $29/tháng | Thêm credits, nhiều parallel hơn |
| **Team** | $199/tháng | Dành cho team vừa |
| **Enterprise** | Liên hệ | SLA, SSO, dedicated support |

### Ước tính chi phí

```
e1-standard-2: ~$0.05/phút

Project nhỏ (10 phút/build, 5 builds/ngày):
  10 × 5 × 30 = 1.500 phút/tháng ≈ $75/tháng

Free tier ($20): đủ cho ~400 phút/tháng
```

> 💡 **Mẹo tiết kiệm:** Dùng `auto_cancel` để hủy build cũ, cache dependencies, chọn machine type vừa đủ.

---

## 24. Troubleshooting

| Vấn đề | Nguyên nhân | Cách sửa |
|--------|-------------|----------|
| Pipeline không trigger | Webhook chưa cấu hình | Kiểm tra GitHub webhook, reconnect repo |
| `checkout` fail | Thiếu quyền GitHub | Re-authorize Semaphore trên GitHub |
| Secret không có giá trị | Sai tên secret | Kiểm tra tên trong Secrets vs `secrets:` |
| Cache restore fail | Cache key sai | Kiểm tra `checksum` và branch name |
| Job timeout | Mặc định 1 giờ | Chia job nhỏ hơn, hoặc liên hệ support tăng limit |
| `docker: command not found` | Machine không có Docker | Docker có sẵn trên mọi machine Linux |
| Promotion không chạy | Sai điều kiện `auto_promote` | Kiểm tra `result` và `branch` |
| Artifact pull fail | Sai tên artifact | Kiểm tra `artifacts:` trong job upload |
| Flaky test không detect | Thiếu JUnit XML | Đảm bảo tạo `junit.xml` và upload artifact |
| Self-hosted agent không nhận job | Sai agent type | Kiểm tra `agent.machine.type` khớp với agent type |

### Debug

```bash
# Xem log chi tiết trong UI: bấm vào job → xem từng command

# SSH vào machine đang chạy (debug trực tiếp)
# Trong job đang chạy: bấm "Debug" → mở SSH session

# Kiểm tra YAML syntax
sem validate .semaphore/semaphore.yml
```

---

## 25. Best Practices Checklist

- [ ] Dùng `auto_cancel` để hủy build cũ khi push mới
- [ ] Dùng `fail_fast` để dừng sớm khi fail
- [ ] Cache dependencies (npm, pip, Go modules)
- [ ] Chia blocks hợp lý: install → test (song song) → build
- [ ] Secrets qua Semaphore Secrets, không hardcode
- [ ] Tách CI và CD bằng promotions
- [ ] Production deploy luôn manual promote
- [ ] JUnit XML cho test reports + flaky detection
- [ ] `change_in()` cho monorepo
- [ ] Machine type vừa đủ (đừng dùng e1-standard-8 cho `npm ci`)
- [ ] Artifacts cho build outputs quan trọng
- [ ] Scheduled pipeline cho E2E/nightly tests

---

## 26. Cheat Sheet

### semaphore.yml template

```yaml
version: v1.0
name: CI

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

auto_cancel:
  running: { when: "true" }
  queued: { when: "true" }

blocks:
  - name: "Install"
    task:
      jobs:
        - name: Install deps
          commands:
            - checkout
            - npm ci

  - name: "Test"
    task:
      jobs:
        - name: Lint
          commands: [npm run lint]
        - name: Test
          commands: [npm test]

  - name: "Build"
    run: { when: "branch = 'main'" }
    task:
      jobs:
        - name: Build
          commands: [npm run build]

promotions:
  - name: Deploy to staging
    pipeline_file: staging.yml
    auto_promote:
      when: "result = 'passed' and branch = 'main'"
```

### Điều kiện hay dùng

```yaml
# Branch main
when: "branch = 'main'"

# Pull request
when: "pull_request != ''"

# Tag
when: "tag =~ '^v.*'"

# File thay đổi
when: "change_in('/frontend/')"

# Kết hợp
when: "branch = 'main' and result = 'passed'"
```

### sem CLI nhanh

```bash
sem connect <org-url> <token>
sem get workflows --project <name>
sem create secret <name> -e KEY=value --project <name>
sem validate .semaphore/semaphore.yml
```

### Biến môi trường

| Biến | Ý nghĩa |
|------|---------|
| `SEMAPHORE_GIT_BRANCH` | Branch hiện tại |
| `SEMAPHORE_GIT_SHA` | Commit SHA |
| `SEMAPHORE_WORKFLOW_ID` | Workflow ID |
| `SEMAPHORE_PROJECT_NAME` | Tên project |

---

## 27. FAQ

**Hỏi: Semaphore có miễn phí không?**
Đáp: Có, $20 credit/tháng (~400 phút). Đủ cho project nhỏ/cá nhân.

**Hỏi: Khác gì GitHub Actions?**
Đáp: Semaphore nhanh hơn, có khái niệm Promotions độc đáo, UI tập trung vào pipeline visualization. GitHub Actions tích hợp sâu với GitHub hơn.

**Hỏi: Có hỗ trợ Windows/macOS không?**
Đáp: Linux là chính. macOS có (đắt hơn). Windows không hỗ trợ chính thức — dùng self-hosted agent nếu cần.

**Hỏi: Promotion khác gì với block?**
Đáp: Block là bước trong cùng pipeline. Promotion là cầu nối sang pipeline **hoàn toàn riêng** (file YAML khác), phù hợp tách CI/CD.

**Hỏi: Có chạy trên ARM được không?**
Đáp: Có, machine type `a1-standard-4` (ARM).

**Hỏi: Migration từ CircleCI có khó không?**
Đáp: Khá dễ. Cấu trúc blocks/jobs tương tự CircleCI jobs/workflows. Chủ yếu là đổi cú pháp YAML.

---

## 28. Phụ Lục: Tích Hợp Nâng Cao

### Tích Hợp SonarQube

```yaml
blocks:
  - name: "Code Analysis"
    task:
      secrets:
        - name: sonar
      jobs:
        - name: SonarQube scan
          commands:
            - checkout
            - |
              docker run --rm \
                -e SONAR_HOST_URL="$SONAR_URL" \
                -e SONAR_LOGIN="$SONAR_TOKEN" \
                -v "$PWD:/usr/src" \
                sonarsource/sonar-scanner-cli
```

### Deploy Lên Kubernetes

```yaml
# .semaphore/production-deploy.yml
blocks:
  - name: "Deploy to K8s"
    task:
      secrets:
        - name: k8s-secrets
      jobs:
        - name: Rolling update
          commands:
            - echo "$KUBECONFIG_DATA" | base64 -d > kubeconfig
            - export KUBECONFIG=kubeconfig
            - kubectl set image deployment/myapp myapp=myuser/myapp:$SEMAPHORE_WORKFLOW_ID -n production
            - kubectl rollout status deployment/myapp -n production --timeout=300s
```

### Blue-Green Deploy

```yaml
blocks:
  - name: "Blue-Green Deploy"
    task:
      secrets:
        - name: deploy-secrets
      jobs:
        - name: Deploy green
          commands:
            - ./scripts/deploy-green.sh $SEMAPHORE_WORKFLOW_ID
            - ./scripts/smoke-test.sh https://green.example.com
            - ./scripts/switch-traffic.sh green
```

### Database Migration Trong Pipeline

```yaml
blocks:
  - name: "Migrate DB"
    run:
      when: "branch = 'main'"
    task:
      secrets:
        - name: db-secrets
      jobs:
        - name: Run migrations
          commands:
            - checkout
            # Backup trước khi migrate
            - pg_dump "$DATABASE_URL" > backup-$SEMAPHORE_WORKFLOW_ID.sql
            - npm run migrate:up
            # Verify
            - npm run migrate:status
```

### Slack Notification Chi Tiết

```yaml
blocks:
  - name: "Notify"
    run:
      when: "true"  # luôn chạy
    task:
      secrets:
        - name: slack
      jobs:
        - name: Send notification
          commands: |
            if [ "$SEMAPHORE_PIPELINE_RESULT" = "passed" ]; then
              EMOJI="✅"
            else
              EMOJI="❌"
            fi
            curl -X POST "$SLACK_WEBHOOK" \
              -H "Content-Type: application/json" \
              -d "{
                \"text\": \"$EMOJI *$SEMAPHORE_PROJECT_NAME* - Pipeline $SEMAPHORE_PIPELINE_RESULT\",
                \"attachments\": [{
                  \"fields\": [
                    {\"title\": \"Branch\", \"value\": \"$SEMAPHORE_GIT_BRANCH\", \"short\": true},
                    {\"title\": \"Commit\", \"value\": \"${SEMAPHORE_GIT_SHA:0:8}\", \"short\": true}
                  ]
                }]
              }"
```

---

## 29. Phụ Lục: Dự Án Thực Tế Bổ Sung

### Go Pipeline

```yaml
version: v1.0
name: Go CI

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

blocks:
  - name: "Vet & Test"
    task:
      jobs:
        - name: Go vet and test
          commands:
            - checkout
            - cache restore go-mod-$(checksum go.sum) || go mod download
            - cache store go-mod-$(checksum go.sum) /home/semaphore/go/pkg/mod
            - go vet ./...
            - go test -race -coverprofile=coverage.out ./...
            - go tool cover -func=coverage.out

  - name: "Build"
    run:
      when: "branch = 'main'"
    task:
      jobs:
        - name: Build binary
          commands:
            - checkout
            - CGO_ENABLED=0 go build -ldflags="-s -w" -o app .
          artifacts:
            - name: binary
              path: app
```

### Rust Pipeline

```yaml
version: v1.0
name: Rust CI

agent:
  machine:
    type: e1-standard-4  # Rust cần nhiều RAM để compile
    os_image: ubuntu2204

blocks:
  - name: "Check & Test"
    task:
      jobs:
        - name: Cargo check
          commands:
            - checkout
            - cache restore cargo-$SEMAPHORE_GIT_BRANCH-$(checksum Cargo.lock) || true
            - cargo check --all-targets
            - cache store cargo-$SEMAPHORE_GIT_BRANCH-$(checksum Cargo.lock) ~/.cargo

        - name: Clippy
          commands:
            - checkout
            - cargo clippy -- -D warnings

        - name: Tests
          commands:
            - checkout
            - cargo test --release
```

### Flutter Pipeline

```yaml
version: v1.0
name: Flutter CI

agent:
  machine:
    type: e1-standard-4
    os_image: ubuntu2204

blocks:
  - name: "Setup Flutter"
    task:
      jobs:
        - name: Install Flutter
          commands:
            - |
              if [ ! -d "/opt/flutter" ]; then
                git clone https://github.com/flutter/flutter.git -b stable /opt/flutter
              fi
              export PATH="$PATH:/opt/flutter/bin"
              flutter --version
              # Lưu vào cache để dùng lại
              cache store flutter-stable /opt/flutter

  - name: "Analyze & Test"
    task:
      jobs:
        - name: Flutter analyze
          commands:
            - cache restore flutter-stable
            - export PATH="$PATH:/opt/flutter/bin"
            - checkout
            - flutter analyze

        - name: Flutter test
          commands:
            - cache restore flutter-stable
            - export PATH="$PATH:/opt/flutter/bin"
            - checkout
            - flutter test --coverage

  - name: "Build APK"
    run:
      when: "tag =~ '^v.*'"
    task:
      jobs:
        - name: Build release APK
          commands:
            - cache restore flutter-stable
            - export PATH="$PATH:/opt/flutter/bin"
            - checkout
            - flutter build apk --release
          artifacts:
            - name: apk
              path: build/app/outputs/flutter-apk/app-release.apk
```

### Terraform Pipeline (Infrastructure as Code)

```yaml
version: v1.0
name: Terraform CI/CD

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

blocks:
  - name: "Validate"
    task:
      jobs:
        - name: Terraform validate
          commands:
            - checkout
            - terraform init -backend=false
            - terraform validate
            - terraform fmt -check -recursive

  - name: "Plan"
    task:
      secrets:
        - name: aws-secrets
      jobs:
        - name: Terraform plan
          commands:
            - checkout
            - terraform init
            - terraform plan -out=tfplan
          artifacts:
            - name: tfplan
              path: tfplan

promotions:
  - name: Apply infrastructure
    pipeline_file: terraform-apply.yml
    # Manual promote: kiểm tra plan kỹ trước khi apply!
```

```yaml
# .semaphore/terraform-apply.yml
version: v1.0
name: Terraform Apply

agent:
  machine:
    type: e1-standard-2
    os_image: ubuntu2204

blocks:
  - name: "Apply"
    task:
      secrets:
        - name: aws-secrets
      jobs:
        - name: Terraform apply
          commands:
            - checkout
            - artifact pull tfplan/
            - terraform init
            - terraform apply tfplan
```

### Security Scanning Pipeline

```yaml
blocks:
  - name: "Security"
    task:
      jobs:
        - name: Trivy scan
          commands:
            - checkout
            - |
              docker run --rm \
                -v /var/run/docker.sock:/var/run/docker.sock \
                aquasec/trivy:latest image \
                --severity HIGH,CRITICAL \
                --exit-code 1 \
                myuser/myapp:latest

        - name: npm audit
          commands:
            - checkout
            - npm audit --audit-level=high

        - name: Gitleaks (quét secrets)
          commands:
            - checkout
            - |
              docker run --rm \
                -v "$PWD:/repo" \
                zricethezav/gitleaks:latest \
                detect --source /repo --verbose
```

### Performance Testing

```yaml
blocks:
  - name: "Performance"
    run:
      when: "branch = 'main'"
    task:
      jobs:
        - name: K6 load test
          commands:
            - checkout
            - |
              docker run --rm \
                -v "$PWD/tests/performance:/scripts" \
                grafana/k6:latest run /scripts/load-test.js
```

```javascript
// tests/performance/load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 50 },   // tăng lên 50 users
    { duration: '1m', target: 50 },    // giữ 50 users
    { duration: '30s', target: 0 },    // giảm về 0
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% requests < 500ms
  },
};

export default function () {
  const res = http.get('https://staging.example.com/api/health');
  check(res, { 'status is 200': (r) => r.status === 200 });
  sleep(1);
}
```

---

*Hết guide Semaphore. Chúc cậu build xanh lè!* 🚦💚
