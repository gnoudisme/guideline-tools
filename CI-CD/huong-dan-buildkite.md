# 🪁 Hướng Dẫn Toàn Tập: Buildkite

> **Cấp độ:** Người mới bắt đầu → cấu hình được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật lần cuối:** 10/10/2026
>
> ⚠️ **Ghi chú:** Buildkite là mô hình **hybrid** — control plane trên cloud, build agents chạy trên hạ tầng của bạn.

---

## 📑 Mục Lục

1. [Buildkite Là Gì?](#1-buildkite-là-gì)
2. [Mô Hình Hybrid: Hiểu Đúng](#2-mô-hình-hybrid-hiểu-đúng)
3. [Buildkite vs GitHub Actions vs Jenkins](#3-buildkite-vs-github-actions-vs-jenkins)
4. [Đăng Ký & Tạo Organization](#4-đăng-ký--tạo-organization)
5. [Cài Đặt Buildkite Agent](#5-cài-đặt-buildkite-agent)
6. [Tạo Pipeline Đầu Tiên](#6-tạo-pipeline-đầu-tiên)
7. [pipeline.yml: Cú Pháp Chi Tiết](#7-pipelineyml-cú-pháp-chi-tiết)
8. [Command Steps](#8-command-steps)
9. [Wait, Block & Input Steps](#9-wait-block--input-steps)
10. [Trigger Steps (Nối Pipeline)](#10-trigger-steps-nối-pipeline)
11. [Artifacts: Upload & Download](#11-artifacts-upload--download)
12. [Annotations (Hiển Thị Đẹp Trong UI)](#12-annotations-hiển-thị-đẹp-trong-ui)
13. [Parallelism & Matrix](#13-parallelism--matrix)
14. [Agent Targeting: Queues & Tags](#14-agent-targeting-queues--tags)
15. [Elastic CI Stack cho AWS](#15-elastic-ci-stack-cho-aws)
16. [Test Analytics](#16-test-analytics)
17. [Secrets & Environment Variables](#17-secrets--environment-variables)
18. [Docker Trong Buildkite](#18-docker-trong-buildkite)
19. [Dự Án Thực Tế: Node.js Pipeline](#19-dự-án-thực-tế-nodejs-pipeline)
20. [Dự Án Thực Tế: Monorepo](#20-dự-án-thực-tế-monorepo)
21. [Scheduled Builds & Cron](#21-scheduled-builds--cron)
22. [REST & GraphQL API](#22-rest--graphql-api)
23. [Pricing](#23-pricing)
24. [Troubleshooting](#24-troubleshooting)
25. [Best Practices Checklist](#25-best-practices-checklist)
26. [Cheat Sheet](#26-cheat-sheet)
27. [FAQ](#27-faq)

---

## 1. Buildkite Là Gì?

**Buildkite** là nền tảng CI/CD theo mô hình **hybrid** độc đáo:

- **Control plane** (lên lịch, UI, quản lý) → chạy trên cloud của Buildkite
- **Build agents** (chạy build thật) → chạy trên **máy của bạn** (VPS, AWS, on-premise, laptop cũng được!)

```
┌─────────────────────────┐         ┌─────────────────────────┐
│   BUILDKITE CLOUD       │         │   HẠ TẦNG CỦA BẠN       │
│   (buildkite.com)       │         │                         │
│                         │  poll   │  ┌─────────────────┐    │
│  ┌───────────────────┐  │ ◄────── │  │ Agent 1         │    │
│  │ Scheduler         │  │  jobs   │  │ (VPS của bạn)   │    │
│  │ Web UI            │  │         │  └─────────────────┘    │
│  │ Build history     │  │ ──────▶ │  ┌─────────────────┐    │
│  │ Pipeline config   │  │ results │  │ Agent 2         │    │
└─────────────────────────┘         │  │ (AWS EC2)       │    │
                                    │  └─────────────────┘    │
  Bạn KHÔNG cần mở port!            │  ┌─────────────────┐    │
  Agent chủ động poll về cloud.     │  │ Agent 3         │    │
                                    │  │ (Mac Mini)      │    │
                                    │  └─────────────────┘    │
                                    └─────────────────────────┘
```

### Điểm mạnh

| Đặc điểm | Giải thích |
|----------|------------|
| **Không cần mở firewall** | Agent poll về cloud, không cần inbound connection |
| **Chạy ở đâu cũng được** | VPS, AWS, GCP, on-premise, Mac Mini, Raspberry Pi |
| **Tốc độ** | Agent gần code/data của bạn → checkout nhanh, cache tốt |
| **Bảo mật** | Code không rời khỏi hạ tầng của bạn |
| **Elastic scaling** | Tự động thêm/bớt agent trên AWS |

### Khi nào nên dùng Buildkite?

- Cần kiểm soát hạ tầng build (compliance, bảo mật)
- Đã có sẵn AWS/GCP và muốn tận dụng
- Cần build trên phần cứng đặc biệt (Mac cho iOS, GPU cho ML)
- Muốn UI đẹp + không maintain server CI

### Khi nào KHÔNG nên dùng?

- Muốn free hoàn toàn → GitHub Actions / Drone
- Không có hạ tầng để chạy agent
- Project nhỏ, không cần phức tạp

---

## 2. Mô Hình Hybrid: Hiểu Đúng

### Luồng hoạt động

1. Bạn push code → GitHub gửi webhook → **Buildkite Cloud**
2. Cloud tạo **build** từ `pipeline.yml` trong repo
3. Cloud xếp **jobs** vào queue
4. **Agent** của bạn (đang poll) nhận job phù hợp
5. Agent checkout code, chạy lệnh, gửi kết quả về cloud
6. Bạn xem log realtime trên buildkite.com

### Cái gì ở cloud, cái gì ở local?

| Ở Buildkite Cloud | Ở Agent của bạn |
|-------------------|-----------------|
| Pipeline definitions | Source code |
| Build history & logs | Build artifacts (trước khi upload) |
| Scheduling & queues | Secrets (nếu dùng local) |
| User management | Docker images, caches |
| Notifications | |

> 💡 **Quan trọng:** Source code của bạn **không bao giờ** đi qua server Buildkite (trừ khi bạn upload artifacts). Agent checkout trực tiếp từ GitHub.

---

## 3. Buildkite vs GitHub Actions vs Jenkins

| Tiêu chí | Buildkite | GitHub Actions | Jenkins |
|----------|-----------|----------------|---------|
| Mô hình | Hybrid | Cloud (+ self-hosted) | Self-hosted |
| Maintain server | Không cần | Không cần | Cần |
| Agent infra | Của bạn | Của GitHub (hoặc bạn) | Của bạn |
| Cấu hình | `pipeline.yml` | YAML workflows | Groovy / UI |
| Miễn phí | Không giới hạn agents | 2.000 phút/tháng | Không giới hạn |
| Tốc độ setup | Nhanh | Rất nhanh | Chậm |
| Phù hợp | Team có infra riêng | Mọi quy mô | Enterprise legacy |

---

## 4. Đăng Ký & Tạo Organization

### Bước 1: Đăng ký

1. Vào [buildkite.com](https://buildkite.com) → **Sign up**
2. Đăng ký bằng GitHub / Email
3. Tạo **Organization** (ví dụ: `my-company`)

### Bước 2: Tạo Agent Token

1. Vào Organization → **Agents** → **Reveal Token**
2. Copy token (dạng `xxx...`) — dùng để cài agent

### Bước 3: Kết nối GitHub

1. Organization Settings → **Repository Providers** → **GitHub**
2. Bấm **Connect** → Authorize trên GitHub
3. Chọn organization/repo được phép

---

## 5. Cài Đặt Buildkite Agent

### Cách 1: Linux (systemd)

```bash
# 1. Thêm repo Buildkite
curl -fsSL https://keys.openpgp.org/vks/v1/by-fingerprint/32A37959C2FA5C3C99EFBC32A79AA38B107996 | \
  sudo gpg --dearmor -o /usr/share/keyrings/buildkite-agent.gpg

echo "deb [signed-by=/usr/share/keyrings/buildkite-agent.gpg] https://apt.buildkite.com/buildkite-agent stable main" | \
  sudo tee /etc/apt/sources.list.d/buildkite-agent.list

# 2. Cài đặt
sudo apt update && sudo apt install -y buildkite-agent

# 3. Cấu hình token
sudo sed -i "s/xxx/PASTE-YOUR-AGENT-TOKEN-HERE/" /etc/buildkite-agent/buildkite-agent.cfg

# 4. Chạy agent
sudo systemctl enable buildkite-agent
sudo systemctl start buildkite-agent

# 5. Kiểm tra
sudo systemctl status buildkite-agent
buildkite-agent --version
```

### Cách 2: Docker

```bash
docker run -d --name buildkite-agent \
  -e BUILDKITE_AGENT_TOKEN=your-agent-token \
  -e BUILDKITE_AGENT_TAGS="queue=default,os=linux" \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME/.docker:/root/.docker" \
  buildkite/agent:latest
```

### Cách 3: macOS (cho iOS builds)

```bash
# Cài bằng Homebrew
brew install buildkite/buildkite/buildkite-agent

# Cấu hình
# Sửa file: /usr/local/etc/buildkite-agent/buildkite-agent.cfg
# token="your-agent-token"

# Chạy
brew services start buildkite-agent
```

### Cấu hình agent (buildkite-agent.cfg)

```ini
# Token (bắt buộc)
token="your-agent-token-here"

# Tên agent (mặc định là hostname)
name="%hostname-%spawn"

# Tags để targeting (xem phần 14)
tags="queue=default,os=linux,docker=true"

# Thư mục build (mặc định ~/builds)
build-path="/var/lib/buildkite-agent/builds"

# Số job chạy song song trên 1 agent
spawn=2
```

### Kiểm tra agent đã kết nối

1. Vào buildkite.com → Organization → **Agents**
2. Thấy agent của bạn với trạng thái **Connected** ✅

---

## 6. Tạo Pipeline Đầu Tiên

### Bước 1: Tạo pipeline trên web

1. Vào Organization → **Pipelines** → **New Pipeline**
2. Điền:
   - **Name:** `my-app`
   - **Repository:** chọn repo GitHub đã kết nối
   - **Steps:** chọn **Read steps from repository** (khuyên dùng)
3. Bấm **Create Pipeline**

### Bước 2: Tạo .buildkite/pipeline.yml

```yaml
# .buildkite/pipeline.yml
steps:
  - label: ":wave: Hello"
    command: echo "Buildkite hoạt động rồi!"
```

### Bước 3: Push và xem build

```bash
mkdir -p .buildkite
# tạo file pipeline.yml như trên
git add .buildkite/pipeline.yml
git commit -m "Add Buildkite pipeline"
git push
```

→ Buildkite tự trigger build → xem log realtime trên web!

---

## 7. pipeline.yml: Cú Pháp Chi Tiết

### Cấu trúc cơ bản

```yaml
# .buildkite/pipeline.yml
steps:
  - label: "Tên step (hỗ trợ emoji :rocket:)"
    command: "lệnh shell chạy"
    agents:
      queue: default        # chọn agent queue
    env:
      MY_VAR: value         # biến môi trường
    artifact_paths:         # file thu thập sau build
      - "dist/**/*"
```

### Các loại step

| Loại | Key | Dùng để |
|------|-----|---------|
| Command | `command` | Chạy lệnh shell (phổ biến nhất) |
| Wait | `wait` | Đợi các step trước xong |
| Block | `block` | Dừng chờ approve thủ công |
| Input | `input` | Hỏi thông tin từ user |
| Trigger | `trigger` | Trigger pipeline khác |
| Group | `group` | Nhóm các step lại |

### Ví dụ tổng hợp

```yaml
steps:
  - label: ":npm: Install"
    command: npm ci
    key: install

  - wait: ~

  - label: ":lint: Lint"
    command: npm run lint
    depends_on: install

  - label: ":jest: Test"
    command: npm test
    depends_on: install

  - wait: ~

  - label: ":docker: Build image"
    command: docker build -t myapp:$BUILDKITE_BUILD_NUMBER .

  - block: ":rocket: Deploy to production?"
    prompt: "Chắc chắn muốn deploy bản này lên production?"

  - label: ":rocket: Deploy"
    command: ./deploy.sh
    branches: main
```

---

## 8. Command Steps

### Cơ bản

```yaml
steps:
  - label: "Run tests"
    command: npm test
```

### Nhiều lệnh

```yaml
steps:
  - label: "Build"
    commands:
      - npm ci
      - npm run build
      - echo "Build xong!"
```

### Multi-line script

```yaml
steps:
  - label: "Complex build"
    command: |
      set -e
      echo "Bắt đầu build..."
      npm ci
      npm run build
      echo "Build xong!"
```

### Chọn shell

```yaml
steps:
  - label: "Bash script"
    command: ./build.sh
    shell: /bin/bash -e -c

  - label: "PowerShell (Windows)"
    command: .\build.ps1
    shell: powershell
```

### Timeout

```yaml
steps:
  - label: "Long test"
    command: npm run test:e2e
    timeout_in_minutes: 60  # mặc định 10 phút
```

### Retry khi fail

```yaml
steps:
  - label: "Flaky test"
    command: npm test
    retry:
      automatic:
        - exit_status: 1
          limit: 3    # retry tối đa 3 lần khi exit code = 1
```

### Skip

```yaml
steps:
  - label: "Optional step"
    command: npm run optional
    skip: "Chưa cần chạy"
    # hoặc điều kiện:
    # skip: true
```

---

## 9. Wait, Block & Input Steps

### Wait (đồng bộ)

```yaml
steps:
  - label: "Test A"
    command: npm run test:a

  - label: "Test B"
    command: npm run test:b

  - wait: ~          # ← đợi A và B xong mới chạy tiếp

  - label: "Deploy"
    command: ./deploy.sh
```

```yaml
# Wait có điều kiện: tiếp tục ngay cả khi fail
  - wait: ~
    continue_on_failure: true
```

### Block (chờ approve thủ công)

```yaml
steps:
  - label: "Build"
    command: npm run build

  - block: ":rocket: Deploy to production?"
    prompt: "Nhập ghi chú release:"
    fields:
      - text: "Release notes"
        key: release-notes
        hint: "Mô tả những gì thay đổi trong bản này"
        required: false
      - select: "Môi trường"
        key: environment
        options:
          - label: "Staging"
            value: staging
          - label: "Production"
            value: production
        default: staging

  - label: "Deploy"
    command: ./deploy.sh $BUILDKITE_ENVIRONMENT
```

### Input (thu thập thông tin, không block)

```yaml
steps:
  - input: "Thông tin release"
    fields:
      - text: "Version"
        key: version
        default: "1.0.0"
        required: true
```

---

## 10. Trigger Steps (Nối Pipeline)

Trigger pipeline khác từ pipeline hiện tại:

```yaml
# Pipeline A: .buildkite/pipeline.yml
steps:
  - label: ":hammer: Build"
    command: npm run build

  - trigger: deploy-pipeline    # ← tên pipeline B
    label: ":rocket: Trigger deploy"
    build:
      message: "Deploy ${BUILDKITE_BUILD_NUMBER}"
      commit: ${BUILDKITE_COMMIT}
      branch: ${BUILDKITE_BRANCH}
      env:
        ARTIFACT_VERSION: ${BUILDKITE_BUILD_NUMBER}
```

### Truyền metadata giữa pipelines

```yaml
# Pipeline A: lưu metadata
steps:
  - label: "Build"
    command: |
      VERSION=$(cat package.json | grep version)
      buildkite-agent meta-data set "app-version" "$VERSION"

# Pipeline B: đọc metadata
steps:
  - label: "Deploy"
    command: |
      VERSION=$(buildkite-agent meta-data get "app-version")
      echo "Deploying version $VERSION"
```

---

## 11. Artifacts: Upload & Download

### Upload artifacts

```yaml
steps:
  - label: "Build"
    command: npm run build
    artifact_paths:
      - "dist/**/*"           # tất cả file trong dist/
      - "coverage/**/*"
      - "logs/*.log"
```

### Download trong step khác

```yaml
steps:
  - label: "Build"
    command: npm run build
    artifact_paths: "dist/**/*"

  - label: "Deploy"
    command: |
      buildkite-agent artifact download "dist/**/*" .
      ./deploy.sh
```

### Download từ build khác

```bash
buildkite-agent artifact download "dist/**/*" . \
  --build $BUILD_NUMBER \
  --step "Build"
```

### Artifact với build khác (cross-pipeline)

```yaml
steps:
  - trigger: deploy-pipeline
    build:
      env:
        SOURCE_BUILD: ${BUILDKITE_BUILD_NUMBER}

# Trong deploy-pipeline:
steps:
  - label: "Download artifacts"
    command: |
      buildkite-agent artifact download "dist/**/*" . \
        --build $SOURCE_BUILD
```

---

## 12. Annotations (Hiển Thị Đẹp Trong UI)

Annotations cho phép hiển thị Markdown/HTML đẹp trong trang build:

```yaml
steps:
  - label: "Test with report"
    command: |
      npm test -- --reporters=default --reporters=jest-junit
      # Tạo annotation từ kết quả
      cat <<EOF | buildkite-agent annotate --style "success" --context "test-results"
      ## 📊 Kết quả Test

      | Suite | Passed | Failed |
      |-------|--------|--------|
      | Unit  | 145    | 0      |
      | E2E   | 23     | 0      |

      **Tổng:** 168/168 passed ✅
      EOF
```

### Annotation styles

| Style | Màu | Dùng khi |
|-------|-----|----------|
| `success` | Xanh lá | Thành công |
| `warning` | Vàng | Cảnh báo |
| `error` | Đỏ | Lỗi |
| `info` | Xanh dương | Thông tin |

### Ví dụ: Coverage report

```bash
# Trong step test:
COVERAGE=$(npm run coverage:summary | grep "All files")

cat <<EOF | buildkite-agent annotate --style "info" --context "coverage"
## 📈 Code Coverage

\`\`\`
$COVERAGE
\`\`\`
EOF
```

---

## 13. Parallelism & Matrix

### Parallelism (chia job thành N phần)

```yaml
steps:
  - label: "Test (shard %n/%N)"
    command: npm test -- --shard=%n/%N
    parallelism: 4    # ← chạy 4 jobs song song
```

Buildkite tự set `%n` (index) và `%N` (tổng số).

### Matrix (nhiều combination)

```yaml
steps:
  - label: "Test Node {{matrix.node}} + DB {{matrix.db}}"
    command: npm test
    env:
      DB_IMAGE: "{{matrix.db}}"
    matrix:
      setup:
        node: ["18", "20", "22"]
        db: ["postgres:15", "postgres:16"]
      adjustments:
        - with: "18"
          skip: "Node 18 không hỗ trợ feature X"
```

→ Tạo 3 × 2 = 6 jobs (trừ 1 bị skip = 5 jobs).

---

## 14. Agent Targeting: Queues & Tags

### Queues (hàng đợi riêng)

```ini
# Agent 1: queue default
tags="queue=default"

# Agent 2: queue cho Docker builds
tags="queue=docker"

# Agent 3: queue cho macOS/iOS
tags="queue=ios,os=macos"
```

```yaml
steps:
  - label: "Build Docker"
    command: docker build -t myapp .
    agents:
      queue: docker    # ← chỉ agent có queue=docker mới nhận

  - label: "Build iOS"
    command: xcodebuild build
    agents:
      queue: ios
```

### Tags chi tiết

```yaml
steps:
  - label: "GPU Training"
    command: python train.py
    agents:
      queue: ml
      gpu: "true"       # ← agent phải có tag gpu=true
      memory: "32gb"
```

```ini
# Agent GPU:
tags="queue=ml,gpu=true,memory=32gb"
```

---

## 15. Elastic CI Stack cho AWS

Tự động scale agents trên AWS theo nhu cầu.

### Cài đặt

1. Vào [buildkite.com/elastic-ci](https://buildkite.com/docs/agent/elastic-ci-aws/elastic-ci-stack)
2. Bấm **Launch Stack** → chọn region (ap-southeast-1)
3. Điền parameters:
   - `BuildkiteAgentToken`: token của bạn
   - `BuildkiteQueue`: `default`
   - `InstanceType`: `t3.medium`
   - `MinSize`: `0` (scale về 0 khi không dùng → tiết kiệm tiền!)
   - `MaxSize`: `10`
4. Bấm **Create Stack**

### Cách hoạt động

```
Build queue trống → 0 instances (không tốn tiền)
     │
     ▼ có build mới
Auto Scaling Group launch EC2 instances
     │
     ▼ agent khởi động, poll job
Chạy builds...
     │
     ▼ hết việc 10 phút
Tự terminate instances
```

### Tiết kiệm chi phí

| Cấu hình | Chi phí ước tính |
|----------|------------------|
| 0-2 agents, t3.medium, spot | ~$10-30/tháng |
| 0-5 agents, t3.large, on-demand | ~$50-150/tháng |

> 💡 Dùng **Spot Instances** để giảm 70% chi phí!

---

## 16. Test Analytics

Buildkite Test Analytics thu thập và phân tích kết quả test.

### Cài đặt

1. Vào Organization → **Test Analytics** → **New Suite**
2. Copy API token

### Tích hợp với Jest

```bash
npm install --save-dev jest-buildkite-reporter
```

```javascript
// jest.config.js
module.exports = {
  reporters: [
    'default',
    ['jest-buildkite-reporter', {
      apiToken: process.env.BUILDKITE_ANALYTICS_TOKEN,
    }],
  ],
};
```

```yaml
steps:
  - label: "Test"
    command: npm test
    env:
      BUILDKITE_ANALYTICS_TOKEN:
        # set trong pipeline settings
```

### Những gì bạn nhận được

- 📊 Test nào chậm nhất
- 🔄 Test nào flaky (lúc pass lúc fail)
- 📈 Xu hướng thời gian test theo thời gian
- 🎯 Tự động phát hiện test mới fail

---

## 17. Secrets & Environment Variables

### Pipeline-level env

Trong Pipeline Settings trên web → **Environment Variables**:
```
DOCKER_REGISTRY=registry.example.com
NODE_ENV=production
```

### Step-level env

```yaml
steps:
  - label: "Deploy"
    command: ./deploy.sh
    env:
      DEPLOY_ENV: production
      API_URL: https://api.example.com
```

### Secrets: dùng Elastic Stack secrets

```bash
# Lưu secret vào AWS SSM Parameter Store
aws ssm put-parameter \
  --name "/buildkite/my-pipeline/docker-password" \
  --value "supersecret" \
  --type SecureString
```

```yaml
steps:
  - label: "Docker push"
    command: |
      echo "$DOCKER_PASSWORD" | docker login -u myuser --password-stdin
    env:
      DOCKER_PASSWORD: ""  # sẽ được inject từ SSM
```

### Dùng plugin secrets

```yaml
steps:
  - label: "Deploy"
    command: ./deploy.sh
    plugins:
      - secrets#v1.0.0:
          variables:
            - AWS_ACCESS_KEY_ID
            - AWS_SECRET_ACCESS_KEY
```

### TUYỆT ĐỐI KHÔNG

```yaml
# ❌ KHÔNG BAO GIỜ làm thế này:
steps:
  - label: "Bad"
    command: echo "password123"  # lộ trong log!
    env:
      PASSWORD: "password123"    # lộ trong pipeline.yml!
```

---

## 18. Docker Trong Buildkite

### Cách 1: Docker plugin

```yaml
steps:
  - label: ":docker: Build"
    plugins:
      - docker#v5.11.0:
          image: node:20-alpine
          workdir: /app
          volumes:
            - ./:/app
          command: ["npm", "ci"]
```

### Cách 2: Docker Compose plugin

```yaml
steps:
  - label: ":docker: Integration test"
    plugins:
      - docker-compose#v5.3.0:
          run: app
          config: docker-compose.test.yml
          command: ["npm", "test"]
```

### Cách 3: Chạy Docker trực tiếp (agent có Docker)

```yaml
steps:
  - label: "Docker build"
    command: |
      docker build -t myapp:$BUILDKITE_BUILD_NUMBER .
      docker push myapp:$BUILDKITE_BUILD_NUMBER
    agents:
      queue: docker
```

---

## 19. Dự Án Thực Tế: Node.js Pipeline

### Kịch bản

App Express + PostgreSQL. Pipeline: **Install → Lint + Test (song song) → Build → Docker Push → Deploy**

### .buildkite/pipeline.yml

```yaml
# .buildkite/pipeline.yml
env:
  NODE_ENV: test
  DOCKER_REGISTRY: registry.example.com

steps:
  - label: ":npm: Install"
    command: npm ci
    key: install
    artifact_paths:
      - "node_modules/**/*"  # cache cho steps sau (không khuyên dùng, chỉ ví dụ)

  - wait: ~

  - group: ":mag: Kiểm tra"
    steps:
      - label: ":lint: ESLint"
        command: npm run lint
        depends_on: install

      - label: ":jest: Unit tests"
        command: npm test -- --ci --coverage
        depends_on: install
        artifact_paths:
          - "coverage/**/*"

      - label: ":lock: Security audit"
        command: npm audit --audit-level=high
        depends_on: install
        allow_failure: true  # không fail build nếu có warning

  - wait: ~

  - label: ":hammer: Build"
    command: npm run build
    artifact_paths:
      - "dist/**/*"

  - wait: ~
    continue_on_failure: false

  - label: ":docker: Build & Push"
    command: |
      docker build -t $DOCKER_REGISTRY/myapp:$BUILDKITE_BUILD_NUMBER .
      docker tag $DOCKER_REGISTRY/myapp:$BUILDKITE_BUILD_NUMBER $DOCKER_REGISTRY/myapp:latest
      docker push $DOCKER_REGISTRY/myapp:$BUILDKITE_BUILD_NUMBER
      docker push $DOCKER_REGISTRY/myapp:latest
    agents:
      queue: docker
    branches: main

  - block: ":rocket: Deploy lên production?"
    prompt: "Xác nhận deploy bản $BUILDKITE_BUILD_NUMBER"
    branches: main

  - label: ":rocket: Deploy"
    command: ./scripts/deploy.sh $BUILDKITE_BUILD_NUMBER
    branches: main
```

---

## 20. Dự Án Thực Tế: Monorepo

### Cấu trúc repo

```
my-monorepo/
├── frontend/          # React app
├── backend/           # Node.js API
├── mobile/            # React Native
└── .buildkite/
    └── pipeline.yml
```

### pipeline.yml với dynamic pipeline

```yaml
# .buildkite/pipeline.yml
steps:
  - label: ":pipeline: Generate pipeline"
    command: .buildkite/generate-pipeline.sh | buildkite-agent pipeline upload
```

```bash
#!/bin/bash
# .buildkite/generate-pipeline.sh
set -e

echo "steps:"

# Kiểm tra thay đổi trong từng thư mục
if git diff --name-only HEAD~1 | grep -q "^frontend/"; then
  cat <<YAML
  - label: ":react: Frontend build"
    command: "cd frontend && npm ci && npm run build"
    artifact_paths: "frontend/dist/**/*"
YAML
fi

if git diff --name-only HEAD~1 | grep -q "^backend/"; then
  cat <<YAML
  - label: ":node: Backend test"
    command: "cd backend && npm ci && npm test"
YAML
fi

if git diff --name-only HEAD~1 | grep -q "^mobile/"; then
  cat <<YAML
  - label: ":iphone: Mobile build"
    command: "cd mobile && npm ci && npm run build:android"
    agents:
      queue: "mobile"
YAML
fi
```

> 💡 **Dynamic pipeline** là tính năng mạnh: generate pipeline YAML bằng script!

---

## 21. Scheduled Builds & Cron

### Tạo schedule trên web

1. Pipeline → **Settings** → **Schedules** → **New Schedule**
2. Điền:
   - **Label:** `Nightly E2E`
   - **Cron:** `0 2 * * *` (2h sáng mỗi ngày)
   - **Branch:** `main`
   - **Message:** `Nightly E2E test`

### Phân biệt build thường vs scheduled

```yaml
steps:
  - label: "E2E Test"
    command: npm run test:e2e
    if: build.source == "schedule"    # chỉ chạy khi là scheduled build

  - label: "Unit Test"
    command: npm test
    if: build.source != "schedule"    # không chạy khi scheduled
```

### Conditional đầy đủ

```yaml
steps:
  - label: "Deploy"
    command: ./deploy.sh
    if: build.branch == "main" && build.source == "webhook"
```

---

## 22. REST & GraphQL API

### REST API

```bash
# Tạo API token: User Settings → API Access Tokens

# List builds
curl -H "Authorization: Bearer $TOKEN" \
  "https://api.buildkite.com/v2/organizations/my-org/pipelines/my-app/builds"

# Tạo build mới
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"commit":"HEAD","branch":"main","message":"Manual trigger"}' \
  "https://api.buildkite.com/v2/organizations/my-org/pipelines/my-app/builds"

# Hủy build
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  "https://api.buildkite.com/v2/organizations/my-org/pipelines/my-app/builds/123/cancel"
```

### GraphQL API

```graphql
# Lấy builds gần nhất
{
  organization(slug: "my-org") {
    pipelines(first: 10) {
      edges {
        node {
          name
          builds(first: 5) {
            edges {
              node {
                number
                state
                message
              }
            }
          }
        }
      }
    }
  }
}
```

---

## 23. Pricing

| Gói | Giá | Chi tiết |
|-----|-----|----------|
| **Free** | $0 | Không giới hạn builds, users, agents. Chỉ thiếu 1 số features enterprise |
| **Team** | $15/user/tháng | Thêm SSO, audit log |
| **Enterprise** | Liên hệ | SLA, dedicated support |

> 💡 Buildkite **không tính tiền theo build minutes** — bạn chỉ trả tiền hạ tầng agent của mình!

### So sánh chi phí thực tế

```
Buildkite Free:           $0/tháng (platform)
+ 1 VPS chạy agent (2GB): ~$10/tháng
+ AWS Elastic (scale 0-5): ~$20-50/tháng (tùy dùng)
─────────────────────────────────
Tổng: $10-50/tháng cho team vừa
```

---

## 24. Troubleshooting

| Vấn đề | Nguyên nhân | Cách sửa |
|--------|-------------|----------|
| Agent không kết nối | Sai token hoặc firewall | Kiểm tra token, agent cần outbound HTTPS (443) |
| Build stuck "Waiting" | Không có agent phù hợp | Kiểm tra `agents.queue` khớp với agent tags |
| `command not found` | PATH khác trên agent | Dùng đường dẫn đầy đủ hoặc set PATH trong step |
| Artifact upload fail | File không tồn tại | Kiểm tra `artifact_paths` pattern |
| Docker permission denied | Agent không có quyền Docker | Thêm user vào group `docker` |
| Pipeline không trigger | Webhook chưa cấu hình | Kiểm tra GitHub webhook → phải có event push |
| Block step không hiện | Chưa đến lượt | Block chỉ hiện sau khi các step trước xong |
| `if` condition không chạy | Sai cú pháp | Dùng `build.branch`, `build.source` (không phải env) |
| Agent disk đầy | Artifacts/builds cũ | Dọn `build-path` định kỳ, dùng cron |
| Log quá lớn | Output quá nhiều | Giảm verbosity, dùng `--quiet` |

### Debug agent

```bash
# Xem log agent
sudo journalctl -u buildkite-agent -f

# Chạy agent ở foreground để debug
buildkite-agent start --debug

# Kiểm tra agent config
buildkite-agent --help
```

### Kiểm tra pipeline YAML

```bash
# Validate YAML syntax
python3 -c "import yaml; yaml.safe_load(open('.buildkite/pipeline.yml'))"

# Dùng buildkite agent để validate
buildkite-agent pipeline upload --dry-run < .buildkite/pipeline.yml
```

---

## 25. Best Practices Checklist

- [ ] Pipeline YAML trong repo (`.buildkite/pipeline.yml`), không config trên web
- [ ] Chia steps nhỏ, đặt `label` rõ ràng với emoji
- [ ] Dùng `key` + `depends_on` thay vì chỉ `wait` (linh hoạt hơn)
- [ ] Secrets qua SSM/vault plugin, không hardcode
- [ ] Dùng `branches:` để giới hạn deploy chỉ trên main
- [ ] Block step trước khi deploy production
- [ ] Artifacts cho build outputs quan trọng
- [ ] Annotations cho test reports, coverage
- [ ] Agent queues riêng cho workload đặc biệt (Docker, iOS, GPU)
- [ ] Timeout hợp lý cho từng step
- [ ] Retry cho test flaky (với limit)
- [ ] Dọn disk agent định kỳ
- [ ] Dùng Elastic CI Stack để auto-scale (tiết kiệm tiền)

---

## 26. Cheat Sheet

### pipeline.yml template

```yaml
steps:
  - label: ":npm: Install"
    command: npm ci
    key: install

  - wait: ~

  - label: ":jest: Test"
    command: npm test
    depends_on: install
    parallelism: 4
    artifact_paths: "coverage/**/*"

  - wait: ~

  - label: ":docker: Build"
    command: docker build -t myapp:$BUILDKITE_BUILD_NUMBER .
    agents: { queue: docker }
    branches: main

  - block: ":rocket: Deploy?"
    branches: main

  - label: ":rocket: Deploy"
    command: ./deploy.sh
    branches: main
```

### Biến môi trường có sẵn

| Biến | Ý nghĩa |
|------|---------|
| `BUILDKITE_BUILD_NUMBER` | Số build |
| `BUILDKITE_COMMIT` | Commit SHA |
| `BUILDKITE_BRANCH` | Tên branch |
| `BUILDKITE_MESSAGE` | Commit message |
| `BUILDKITE_BUILD_URL` | URL của build |
| `BUILDKITE_AGENT_NAME` | Tên agent đang chạy |
| `BUILDKITE_PULL_REQUEST` | Số PR (nếu có) |
| `BUILDKITE_TAG` | Tên tag (nếu có) |

### buildkite-agent CLI

```bash
buildkite-agent start                    # chạy agent
buildkite-agent meta-data set K V        # lưu metadata
buildkite-agent meta-data get K          # đọc metadata
buildkite-agent artifact upload "dist/*" # upload artifact
buildkite-agent artifact download "*" .  # download artifact
buildkite-agent annotate --style success # tạo annotation
buildkite-agent pipeline upload          # upload dynamic pipeline
```

### Conditional

```yaml
if: build.branch == "main"
if: build.source == "schedule"
if: build.pull_request.id != null
```

---

## 27. FAQ

**Hỏi: Buildkite có miễn phí không?**
Đáp: Có, gói Free không giới hạn builds/users/agents. Bạn chỉ trả tiền hạ tầng agent của mình.

**Hỏi: Khác gì GitHub Actions?**
Đáp: Buildkite là hybrid — bạn kiểm soát hoàn toàn máy chạy build. GitHub Actions chạy trên cloud của GitHub (trừ self-hosted runner).

**Hỏi: Có cần mở port firewall không?**
Đáp: Không! Agent chủ động kết nối ra cloud (outbound HTTPS), không cần inbound.

**Hỏi: Chạy trên Raspberry Pi được không?**
Đáp: Được! Có agent binary cho ARM.

**Hỏi: Build iOS thì sao?**
Đáp: Chạy agent trên Mac Mini/Mac Studio, dùng queue riêng `queue=ios`.

**Hỏi: Tốc độ so với GitHub Actions?**
Đáp: Thường nhanh hơn vì agent gần code, cache tốt, không phải chờ queue chung.

**Hỏi: Có hỗ trợ Windows không?**
Đáp: Có, agent chạy trên Windows, dùng PowerShell.

---

## 28. Phụ Lục: Nâng Cao

### Cài Agent Trên Nhiều Nền Tảng

**Ubuntu/Debian (đã có ở phần 5), thêm CentOS/RHEL:**

```bash
# CentOS/RHEL/Fedora
sudo dnf config-manager --add-repo https://download.buildkite.com/buildkite-agent/el/8/stable/buildkite-agent.repo
sudo dnf install -y buildkite-agent
sudo sed -i "s/xxx/YOUR-TOKEN/" /etc/buildkite-agent/buildkite-agent.cfg
sudo systemctl enable --now buildkite-agent
```

**Windows:**

```powershell
# Tải installer từ buildkite.com
# Hoặc dùng Chocolatey:
choco install buildkite-agent

# Cấu hình: C:\buildkite-agent\buildkite-agent.cfg
# token="your-token"

# Chạy như service:
nssm install buildkite-agent "C:\buildkite-agent\bin\buildkite-agent.exe" "start"
nssm start buildkite-agent
```

**Raspberry Pi (ARM):**

```bash
# Tải binary ARM
curl -L https://github.com/buildkite/agent/releases/latest/download/buildkite-agent-linux-arm64 -o buildkite-agent
chmod +x buildkite-agent
sudo mv buildkite-agent /usr/local/bin/

# Tạo config
mkdir -p ~/.buildkite-agent
echo 'token="your-token"' > ~/.buildkite-agent/buildkite-agent.cfg
echo 'tags="queue=default,arch=arm64"' >> ~/.buildkite-agent/buildkite-agent.cfg

# Chạy
buildkite-agent start
```

### Lifecycle Hooks (tùy chỉnh agent)

Agent hỗ trợ hooks — script chạy ở các thời điểm nhất định:

```
~/.buildkite-agent/hooks/
├── pre-checkout          # trước khi checkout code
├── post-checkout         # sau khi checkout
├── pre-command           # trước khi chạy command
├── post-command          # sau khi chạy command
├── pre-artifact          # trước khi upload artifact
└── post-artifact         # sau khi upload artifact
```

**Ví dụ: tự động dọn Docker sau mỗi build**

```bash
#!/bin/bash
# ~/.buildkite-agent/hooks/post-command
set -e
# Xóa container/image không dùng sau mỗi build
docker system prune -f --filter "until=24h"
```

**Ví dụ: thông báo Telegram sau mỗi build**

```bash
#!/bin/bash
# ~/.buildkite-agent/hooks/post-command
set -e

STATUS="✅"
if [ "$BUILDKITE_COMMAND_EXIT_STATUS" != "0" ]; then
  STATUS="❌"
fi

curl -s -X POST "https://api.telegram.org/bot$TELEGRAM_TOKEN/sendMessage" \
  -d "chat_id=$TELEGRAM_CHAT_ID" \
  -d "text=$STATUS Build #$BUILDKITE_BUILD_NUMBER ($BUILDKITE_LABEL) - $BUILDKITE_BUILD_URL"
```

### Clustered Agents (nhiều agent trên 1 máy)

```ini
# Chạy 4 agents trên 1 máy mạnh
# /etc/buildkite-agent/buildkite-agent.cfg
spawn=4
```

Hoặc chạy nhiều process riêng:

```bash
for i in 1 2 3 4; do
  BUILDKITE_AGENT_NAME="agent-$i" \
  buildkite-agent start &
done
```

### Pipeline Templates (tái sử dụng)

Tạo template dùng chung cho nhiều repo:

```yaml
# Template: common-steps.yml
steps:
  - label: ":broom: Cleanup"
    command: rm -rf node_modules dist
```

```yaml
# .buildkite/pipeline.yml trong từng repo
steps:
  - label: ":pipeline: Load common steps"
    command: buildkite-agent pipeline upload .buildkite/common-steps.yml
```

### Tích Hợp SonarQube

```yaml
steps:
  - label: ":sonar: Code analysis"
    command: |
      sonar-scanner \
        -Dsonar.projectKey=my-project \
        -Dsonar.sources=src \
        -Dsonar.host.url=$SONAR_URL \
        -Dsonar.login=$SONAR_TOKEN
    env:
      SONAR_URL: https://sonar.example.com
```

### Tích Hợp Snyk

```yaml
steps:
  - label: ":lock: Snyk scan"
    command: |
      npm install -g snyk
      snyk test --severity-threshold=high
    allow_failure: true
```

### Deploy Lên Kubernetes

```yaml
steps:
  - label: ":kubernetes: Deploy to K8s"
    command: |
      # Cập nhật image trong deployment
      kubectl set image deployment/myapp \
        myapp=$DOCKER_REGISTRY/myapp:$BUILDKITE_BUILD_NUMBER \
        -n production

      # Đợi rollout xong
      kubectl rollout status deployment/myapp -n production --timeout=300s

      # Annotate để hiện thông tin
      cat <<EOF | buildkite-agent annotate --style "success"
      ## 🚀 Deployed to Kubernetes

      - **Image:** \`$DOCKER_REGISTRY/myapp:$BUILDKITE_BUILD_NUMBER\`
      - **Namespace:** production
      - **Time:** $(date)
      EOF
    branches: main
    agents:
      queue: deploy
```

### Blue-Green Deploy Script

```bash
#!/bin/bash
# scripts/blue-green-deploy.sh
set -e

VERSION=$1
NAMESPACE="production"

# Deploy bản mới với label version mới (green)
kubectl set image deployment/myapp-green \
  myapp=$DOCKER_REGISTRY/myapp:$VERSION \
  -n $NAMESPACE

kubectl rollout status deployment/myapp-green -n $NAMESPACE --timeout=300s

# Smoke test
curl -f https://green.myapp.com/health || exit 1

# Chuyển traffic sang green
kubectl patch service myapp -n $NAMESPACE \
  -p '{"spec":{"selector":{"version":"'$VERSION'"}}}'

echo "✅ Blue-green deploy thành công: $VERSION"
```

### Test Splitting (chia test thông minh)

```yaml
steps:
  - label: "Test"
    command: |
      # Chia test files thành N phần bằng nhau
      TEST_FILES=$(find tests -name "*.test.js" | sort)
      TOTAL=$(echo "$TEST_FILES" | wc -l)
      PER_JOB=$(( (TOTAL + BUILDKITE_PARALLEL_JOB_COUNT - 1) / BUILDKITE_PARALLEL_JOB_COUNT ))
      START=$(( BUILDKITE_PARALLEL_JOB * PER_JOB + 1 ))
      echo "$TEST_FILES" | sed -n "${START},$((START + PER_JOB - 1))p" | xargs npm test --
    parallelism: 8
```

### Cache Dependencies Hiệu Quả

```bash
#!/bin/bash
# Trong step đầu tiên của pipeline

# Cache key dựa trên lock file
CACHE_KEY="npm-$(sha256sum package-lock.json | cut -d' ' -f1)"

if [ -d "/var/cache/buildkite/$CACHE_KEY" ]; then
  echo "--- :package: Restoring cache"
  cp -r "/var/cache/buildkite/$CACHE_KEY/node_modules" ./node_modules
else
  echo "--- :package: Installing (no cache)"
  npm ci
  mkdir -p "/var/cache/buildkite/$CACHE_KEY"
  cp -r ./node_modules "/var/cache/buildkite/$CACHE_KEY/"
fi
```

---

### Emergency Rollback

```yaml
steps:
  - label: ":rotating_light: Rollback"
    command: |
      # Rollback về bản trước
      PREVIOUS=$(kubectl rollout history deployment/myapp -n production | tail -2 | head -1 | awk '{print $1}')
      kubectl rollout undo deployment/myapp -n production --to-revision=$PREVIOUS
      kubectl rollout status deployment/myapp -n production --timeout=300s

      cat <<EOF | buildkite-agent annotate --style "warning"
      ## ⚠️ Đã rollback về revision $PREVIOUS
      EOF
    branches: main
```

### Pipeline Cho Pull Request (an toàn)

```yaml
steps:
  - label: ":test_tube: PR Checks"
    command: |
      npm ci
      npm run lint
      npm test
    # Không cho PR từ fork chạy deploy steps
    if: build.pull_request.id == null || build.pull_request.repository.fork == false

  - label: ":shield: Security scan (internal only)"
    command: ./security-scan.sh
    if: build.pull_request.repository.fork == false
```

### Tích Hợp Với GitHub Status

Buildkite tự động báo trạng thái về GitHub (✅/❌ trên commit/PR) nếu đã kết nối GitHub App. Kiểm tra:

1. Organization Settings → **Repository Providers** → GitHub
2. Đảm bảo **Build Status** được bật
3. Mỗi build sẽ hiện trạng thái ngay trên GitHub PR

### Notifications Nâng Cao

```yaml
# Trong Pipeline Settings → Notifications (web UI)
# Hoặc dùng notify block trong pipeline.yml:

notify:
  - slack: "#ci-alerts"
    if: build.state == "failed"
  - email: "team@example.com"
    if: build.branch == "main" && build.state == "failed"
  - webhook: "https://hooks.example.com/buildkite"
```

### Buildkite Packages (quản lý artifacts như registry)

Buildkite Packages cho phép lưu trữ packages (npm, Docker, Python...) ngay trong Buildkite:

```yaml
steps:
  - label: ":package: Publish npm package"
    command: |
      echo "//registry.buildkite.com/:_authToken=$BUILDKITE_PACKAGE_TOKEN" > .npmrc
      npm publish --registry https://registry.buildkite.com/my-org/npm
    env:
      BUILDKITE_PACKAGE_TOKEN: "token-cua-ban"
```

---

*Hết guide Buildkite. Chúc cậu build xanh lè!* 🪁💚
