# Hướng dẫn Bitbucket Pipelines — CI/CD toàn tập

> Tài liệu tiếng Việt chi tiết về Bitbucket Pipelines: từ bật Pipelines cho repository,
> viết file `bitbucket-pipelines.yml`, cho đến deploy, Docker, OIDC, self-hosted runner
> và các mẹo xử lý lỗi thường gặp. Mọi ví dụ code đều dùng được trong thực tế.

---

## Mục lục

1. [Bitbucket Pipelines là gì](#1-bitbucket-pipelines-là-gì)
2. [So sánh với GitHub Actions và GitLab CI](#2-so-sánh-với-github-actions-và-gitlab-ci)
3. [Bật Pipelines cho repository](#3-bật-pipelines-cho-repository)
4. [File `bitbucket-pipelines.yml` — cú pháp đầy đủ](#4-file-bitbucket-pipelinesyml--cú-pháp-đầy-đủ)
5. [Step: image, script, caches, artifacts, variables](#5-step-image-script-caches-artifacts-variables)
6. [Các loại pipeline: default, branches, pull-requests, custom, tags](#6-các-loại-pipeline-default-branches-pull-requests-custom-tags)
7. [Caches và Artifacts](#7-caches-và-artifacts)
8. [Deployments & Environments](#8-deployments--environments)
9. [Repository và Workspace Variables](#9-repository-và-workspace-variables)
10. [Self-hosted Runners](#10-self-hosted-runners)
11. [Docker trong Pipelines](#11-docker-trong-pipelines)
12. [OIDC với AWS và GCP](#12-oidc-với-awsgcp)
13. [Parallel steps và Conditions](#13-parallel-steps-và-conditions)
14. [Ví dụ thực tế](#14-ví-dụ-thực-tế)
15. [Build minutes và pricing](#15-build-minutes-và-pricing)
16. [Troubleshooting](#16-troubleshooting)
17. [Best practices](#17-best-practices)
18. [Cheat sheet](#18-cheat-sheet)

---

## 1. Bitbucket Pipelines là gì

**Bitbucket Pipelines** là dịch vụ **CI/CD tích hợp sẵn** (built-in) của **Bitbucket Cloud**.
Thay vì phải tự dựng Jenkins hay nối Bitbucket với một CI server bên ngoài, bạn chỉ cần
thêm file `bitbucket-pipelines.yml` vào gốc repository — mọi push, pull request đều tự động
trigger pipeline build/test/deploy.

**Chu trình hoạt động:**

```
push code ──▶ Bitbucket phát hiện bitbucket-pipelines.yml
      │
      ▼
Chia thành các step, mỗi step chạy trong một Docker container riêng
      │
      ▼
Cài dependencies → chạy test/lint → build → (deploy)
      │
      ▼
Hiển thị kết quả (pass/fail) ngay trên commit, PR, giao diện Pipelines
```

**Khái niệm cốt lõi cần nắm:**

| Thuật ngữ | Ý nghĩa |
|---|---|
| **Pipeline** | Một lần chạy CI/CD (một run), gồm một hoặc nhiều step |
| **Step** | Một đơn vị công việc: chạy trong Docker container, thực thi các lệnh `script` |
| **Build container** | Container Docker chạy step — image do bạn chọn (Node, Python, Go…) |
| **Service container** | Container phụ (MySQL, Redis…) chạy cùng step để test cần database |
| **Cache** | Lưu dependencies (node_modules, ~/.m2…) giữa các lần chạy để build nhanh |
| **Artifact** | File tạo ra ở step này, chuyển sang step khác (file build, báo cáo test…) |
| **Deployment** | Môi trường deploy có tên (test / staging / production), có tracking lịch sử |

**Điểm khác biệt cốt lõi so với CI server tự dựng:**

- Không cần cài đặt, bảo trì server CI — Atlassian lo hết phần hạ tầng.
- Mỗi step là một container Docker sạch, cô lập hoàn toàn.
- Cấu hình bằng YAML khai báo, version-control cùng code.

> **Lưu ý:** Bitbucket Pipelines chỉ có trên **Bitbucket Cloud** (bitbucket.org).
> Bitbucket Data Center (self-hosted) không có Pipelines — phải dùng Bamboo hoặc CI khác.

---

## 2. So sánh với GitHub Actions và GitLab CI

| Tiêu chí | Bitbucket Pipelines | GitHub Actions | GitLab CI |
|---|---|---|---|
| File cấu hình | `bitbucket-pipelines.yml` (gốc repo) | `.github/workflows/*.yml` | `.gitlab-ci.yml` (gốc repo) |
| Đơn vị chạy | Step (1 container Docker/step) | Job (mỗi job có thể nhiều step) | Job (mỗi job 1 image) |
| Container mặc định | **Bắt buộc** Docker image cho mỗi step | Ubuntu runner, container tùy chọn | Image Docker tùy chọn (shared runner) |
| Free tier | 50 build minutes/tháng (Free plan) | 2.000 minutes/tháng (private repo) | 400 compute minutes/tháng |
| Parallel | Có (tối đa 100 step song song) | Có (matrix, jobs song song) | Có (parallel, DAG) |
| Environments | Có (test/staging/production) + deployment tracking | Có (environments + protection rules) | Có (environments, review apps) |
| Manual trigger | Custom pipeline (bấm nút chạy tay) | `workflow_dispatch` | Manual job (`when: manual`) |
| Secrets | Repository/workspace/deployment variables (secured) | Secrets + Environments secrets | CI/CD variables (masked/protected) |
| OIDC cloud | Có (AWS, GCP) | Có (AWS, GCP, Azure…) | Có (AWS, GCP, Azure, Vault) |
| Self-hosted | Linux runners (Docker) | Self-hosted runners (đa OS) | GitLab Runner (đa OS, đa executor) |
| Marketplace | Ít pipe chính chủ (~ Atlassian) | Rất lớn (Actions marketplace) | Ít hơn, dùng image/template |
| Reusable config | YAML anchors (`&`, `*`), definitions | Reusable workflows, composite actions | `extends`, `include`, components |
| Tích hợp sẵn | Jira, Trello, Opsgenie (cùng nhà Atlassian) | GitHub ecosystem | GitLab DevSecOps all-in-one |

**Khi nào chọn Bitbucket Pipelines?**

- Team đã dùng Bitbucket + Jira: tích hợp issue key (ví dụ `PROJ-123`) vào commit message
  thì trạng thái build hiện ngay trên ticket Jira, không cần plugin.
- Muốn CI/CD "zero-ops": không dựng server, mỗi step chạy trong container sạch.
- Workflow đơn giản đến trung bình: test → build → deploy vài môi trường.

**Khi nào cân nhắc giải pháp khác?**

- Cần hàng nghìn phút build miễn phí mỗi tháng → GitHub Actions hào phóng hơn.
- Cần Windows/macOS runner hoặc self-hosted đa nền tảng → GitHub Actions / GitLab Runner linh hoạt hơn.
- Cần marketplace action khổng lồ, copy-paste là chạy → GitHub Actions.

---

## 3. Bật Pipelines cho repository

### 3.1. Bật từ giao diện web

1. Mở repository trên bitbucket.org.
2. Vào **Repository settings** (bánh răng ở sidebar trái).
3. Chọn **Pipelines** → **Settings**.
4. Bật công tắc **Enable Pipelines**.
5. (Tùy chọn) Bật **Enable Pipelines for pull requests from forks** nếu muốn chạy CI cho PR từ fork.

Sau khi bật, Bitbucket gợi ý tạo file `bitbucket-pipelines.yml` bằng template
(**Pipelines** → **Create your first pipeline** → chọn template như Node.js, Python…).

### 3.2. Bật bằng file YAML

Thực ra chỉ cần **commit file `bitbucket-pipelines.yml` vào nhánh mặc định** (thường là `main`)
là Pipelines tự kích hoạt — không bắt buộc thao tác trên UI. Quy tắc nhận diện file:

- File phải đúng tên `bitbucket-pipelines.yml`, đặt ở **thư mục gốc** của repo.
- Áp dụng cho cả nhánh `main`/`master` và các nhánh khác có file này.

### 3.3. Kiểm tra pipeline đang chạy ở đâu

- Tab **Pipelines** trong sidebar repo: danh sách mọi lần chạy, filter theo nhánh, trạng thái, người trigger.
- Trên trang **commit** và **pull request**: badge trạng thái build (✔ / ✖ / …) ngay cạnh commit.
- Trên **Jira**: nếu commit message chứa issue key (`git commit -m "PROJ-42 fix login"`),
  trạng thái build hiện trong phần Development của ticket.

### 3.4. Tắt Pipelines

**Repository settings → Pipelines → Settings** → tắt **Enable Pipelines**.
Các pipeline đang chạy sẽ bị hủy. Muốn tắt tạm thời cho một push, thêm `[skip ci]`
hoặc `[ci skip]` vào commit message:

```bash
git commit -m "docs: cập nhật README [skip ci]"
```

---

## 4. File `bitbucket-pipelines.yml` — cú pháp đầy đủ

### 4.1. Khung xương tối thiểu

```yaml
# bitbucket-pipelines.yml — file cấu hình pipeline, đặt ở gốc repository
image: node:20  # Docker image mặc định cho mọi step (nếu step không khai báo image riêng)

pipelines:
  default:
    - step:
        name: Chào thế giới
        script:
          - echo "Xin chào từ Bitbucket Pipelines!"
          - node --version
```

Chạy pipeline này: push bất kỳ commit nào → pipeline `default` chạy → log hiện dòng chào.

### 4.2. Cấu trúc đầy đủ

```yaml
# ── Image mặc định cho toàn file ──────────────────────────────
image: node:20

# ── Định nghĩa dùng chung (definitions) ───────────────────────
definitions:
  caches:
    node-custom: ./node_modules          # cache tự định nghĩa
  services:
    postgres:
      image: postgres:15
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass

# ── Các pipeline ──────────────────────────────────────────────
pipelines:
  branches:
    main:
      - step:
          name: Test & Build
          image: node:20-alpine           # ghi đè image mặc định của file
          caches:
            - node                        # cache có sẵn của Atlassian
          script:
            - npm ci
            - npm test
            - npm run build
          artifacts:
            - dist/**                     # giữ thư mục dist cho step sau

      - step:
          name: Deploy production
          deployment: production          # đánh dấu đây là deployment
          trigger: manual                 # bấm tay mới deploy
          script:
            - echo "Deploying $BITBUCKET_BUILD_NUMBER..."

  pull-requests:
    '**':
      - step:
          name: Kiểm tra PR
          script:
            - npm ci
            - npm run lint
            - npm test

  custom:
    deploy-staging:
      - step:
          name: Deploy staging
          deployment: staging
          script:
            - ./scripts/deploy.sh staging

  tags:
    'v*':
      - step:
          name: Release
          script:
            - ./scripts/release.sh
```

### 4.3. Các key cấp cao nhất

| Key | Bắt buộc | Mô tả |
|---|---|---|
| `image` | Không | Docker image mặc định cho mọi step |
| `definitions` | Không | Khai báo `caches`, `services` dùng chung |
| `pipelines` | **Có** | Chứa `default`, `branches`, `pull-requests`, `custom`, `tags` |
| `options` | Không | Tùy chọn toàn cục: `docker: true` (bật Docker service), `max-time` |

Ví dụ `options`:

```yaml
options:
  docker: true      # cho phép dùng Docker trong mọi step (build image, docker-compose…)
  max-time: 60      # timeout toàn pipeline: 60 phút (mặc định 120)

image: python:3.12
pipelines:
  default:
    - step:
        name: Build Docker image
        script:
          - docker build -t myapp:$BITBUCKET_BUILD_NUMBER .
```

> **Lưu ý:** `options.docker: true` cấp quyền chạy Docker daemon trong container —
> cần thiết cho mọi step dùng lệnh `docker ...`. Có thể bật riêng từng step bằng
> `services: - docker` (xem mục 11).

### 4.4. YAML anchors — tái sử dụng cấu hình

Pipelines hỗ trợ YAML anchor (`&` định nghĩa, `*` tham chiếu) giúp tránh lặp code:

```yaml
definitions:
  steps:
    - step: &test-step          # đặt tên anchor "test-step"
        name: Test
        image: node:20
        caches:
          - node
        script:
          - npm ci
          - npm test

pipelines:
  branches:
    main:
      - step: *test-step         # dùng lại step đã định nghĩa
      - step:
          name: Build
          image: node:20
          caches:
            - node
          script:
            - npm ci
            - npm run build
```

---

## 5. Step: image, script, caches, artifacts, variables

Step là đơn vị công việc nhỏ nhất. Mỗi step:

1. Khởi tạo một Docker container mới từ `image` đã chọn.
2. Clone source code của commit trigger vào `/opt/atlassian/pipelines/agent/build`.
3. Khôi phục cache, chạy từng lệnh trong `script` theo thứ tự.
4. Lưu artifacts, upload cache, rồi **hủy container**.

### 5.1. `image` — chọn môi trường chạy

```yaml
pipelines:
  default:
    - step:
        name: Test Python
        image: python:3.12            # Docker Hub image
        script:
          - python --version
    - step:
        name: Test Go
        image: golang:1.22
        script:
          - go version
    - step:
        name: Dùng private registry
        image:
          name: registry.congty.vn/team/base-image:2.1
          username: $REGISTRY_USER        # biến môi trường
          password: $REGISTRY_PASSWORD    # biến secured
          email: devops@congty.vn
        script:
          - ./run.sh
```

- Có thể dùng image từ **Docker Hub**, **AWS ECR**, **GCP Artifact Registry**,
  hoặc private registry bất kỳ (cần username/password).
- Image dùng trong Pipelines nên nhỏ gọn (`-alpine`, `-slim`) để pull nhanh.

### 5.2. `script` — các lệnh thực thi

```yaml
- step:
    name: Build frontend
    image: node:20
    script:
      - npm ci                              # cài dependencies (nhanh & sạch hơn npm install)
      - npm run lint
      - npm test -- --coverage
      - npm run build
```

Quy tắc:

- Mỗi dòng là một lệnh shell chạy bằng `bash`. Lệnh nào **exit code khác 0** → step fail ngay,
  các lệnh sau không chạy.
- Muốn cho phép lệnh fail mà vẫn tiếp tục: `npm run lint || true` (cân nhắc kỹ — có thể che lỗi thật).
- Dùng biến môi trường: `$BITBUCKET_BRANCH`, `$BITBUCKET_COMMIT`, xem đầy đủ ở mục 5.5.

**Multi-line script** cho lệnh dài:

```yaml
script:
  - |
    if [ "$BITBUCKET_BRANCH" = "main" ]; then
      echo "Đang ở nhánh main — deploy production"
      ./deploy.sh production
    else
      echo "Nhánh $BITBUCKET_BRANCH — chỉ build"
    fi
```

### 5.3. `name`, `trigger`, `max-time`, `size`, `condition`

```yaml
- step:
    name: Deploy production        # tên hiển thị trên UI (nên đặt tiếng Việt/Anh rõ nghĩa)
    deployment: production          # gắn step với môi trường deploy
    trigger: manual                 # "manual" = chờ bấm nút; mặc định "automatic"
    max-time: 30                    # timeout của step: 30 phút (mặc định 120, tối đa 120)
    size: 2x                        # 1x (4GB RAM, mặc định) hoặc 2x (8GB RAM, tốn gấp đôi minutes)
    script:
      - ./deploy.sh
```

| Thuộc tính | Giá trị | Ghi chú |
|---|---|---|
| `trigger` | `automatic` (mặc định) / `manual` | Manual step hiện nút ▶ trên UI để bấm chạy tay |
| `max-time` | 1–120 (phút) | Hết giờ step bị kill, đánh fail |
| `size` | `1x` / `2x` | 2x = 8GB RAM, build minutes tính gấp đôi |
| `condition` | biểu thức `changesets` | chỉ chạy step khi file thay đổi (xem mục 13) |

### 5.4. `services` — container phụ (database, cache…)

```yaml
definitions:
  services:
    postgres:
      image: postgres:15
      variables:
        POSTGRES_DB: shopdb
        POSTGRES_USER: shop
        POSTGRES_PASSWORD: shopsecret
    redis:
      image: redis:7-alpine

pipelines:
  default:
    - step:
        name: Integration test
        image: python:3.12
        services:
          - postgres
          - redis
        script:
          - pip install -r requirements.txt
          - pytest tests/integration
```

- Service container chạy **trước** khi script bắt đầu, cùng network với build container.
- Kết nối qua hostname = tên service: `postgres://shop:shopsecret@postgres:5432/shopdb`.

### 5.5. Biến môi trường mặc định của Pipelines

| Biến | Ví dụ giá trị | Ý nghĩa |
|---|---|---|
| `BITBUCKET_BUILD_NUMBER` | `42` | Số thứ tự build, tăng dần — hay dùng tag version |
| `BITBUCKET_COMMIT` | `a1b2c3d…` | SHA đầy đủ của commit trigger |
| `BITBUCKET_BRANCH` | `main` | Tên nhánh (pipeline từ branch) |
| `BITBUCKET_TAG` | `v1.2.0` | Tên tag (pipeline từ tag) |
| `BITBUCKET_PR_ID` | `17` | ID pull request (pipeline PR) |
| `BITBUCKET_REPO_SLUG` | `shop-api` | Slug của repository |
| `BITBUCKET_REPO_FULL_NAME` | `congty/shop-api` | `workspace/repo` |
| `BITBUCKET_WORKSPACE` | `congty` | Tên workspace |
| `BITBUCKET_STEP_UUID` | `{…}` | UUID của step hiện tại |
| `BITBUCKET_DEPLOYMENT_ENVIRONMENT` | `production` | Tên môi trường deploy (step có `deployment`) |
| `BITBUCKET_PARALLEL_STEP` | `0` | Chỉ số step trong nhóm parallel |
| `BITBUCKET_PARALLEL_STEP_COUNT` | `3` | Tổng số step trong nhóm parallel |
| `CI` | `true` | Luôn `true` trong Pipelines |

Dùng trong script:

```bash
echo "Build #$BITBUCKET_BUILD_NUMBER trên nhánh $BITBUCKET_BRANCH"
docker tag myapp:latest myapp:$BITBUCKET_BUILD_NUMBER
```

---

## 6. Các loại pipeline: default, branches, pull-requests, custom, tags

```yaml
pipelines:
  default:            # chạy khi push lên nhánh KHÔNG khớp branches nào bên dưới
    - step: ...

  branches:           # khớp theo tên nhánh (glob pattern)
    main:
      - step: ...     # push lên main
    'feature/*':
      - step: ...     # push lên feature/xxx
    '**':             # mọi nhánh còn lại (đặt cuối cùng)
      - step: ...

  pull-requests:      # chạy khi tạo/cập nhật PR
    '**':
      - step: ...     # mọi PR
    main:
      - step: ...     # PR nhắm vào main

  tags:               # chạy khi push tag
    'v*':
      - step: ...     # tag v1.0.0, v2.1.3…

  custom:             # KHÔNG tự chạy — chỉ chạy tay từ UI hoặc API
    deploy-staging:
      - step: ...
    run-e2e:
      - step: ...
```

### 6.1. Thứ tự ưu tiên khi push

Khi bạn push lên nhánh `main`:

1. Nếu `branches.main` tồn tại → chạy nó.
2. Nếu không, thử khớp pattern (`feature/*`, `**`…).
3. Nếu không khớp gì → chạy `default`.

### 6.2. Chạy custom pipeline

**Từ UI:** tab **Pipelines** → nút **Run pipeline** → chọn branch/tag/commit → chọn pipeline
trong mục Custom → **Run**.

**Từ API** (cần app password hoặc API token):

```bash
curl -X POST \
  -u "ten-dang-nhap:APP_PASSWORD" \
  -H "Content-Type: application/json" \
  https://api.bitbucket.org/2.0/repositories/congty/shop-api/pipelines/ \
  -d '{
    "target": {
      "ref_type": "branch",
      "ref_name": "main",
      "type": "pipeline_ref_target",
      "selector": {
        "type": "custom",
        "pattern": "deploy-staging"
      }
    }
  }'
```

### 6.3. Ví dụ phân luồng hoàn chỉnh

```yaml
image: node:20

pipelines:
  # Mọi PR: chỉ kiểm tra nhanh, không deploy
  pull-requests:
    '**':
      - step:
          name: Lint & Unit test
          caches: [node]
          script:
            - npm ci
            - npm run lint
            - npm test

  branches:
    main:
      - step:
          name: Test
          caches: [node]
          script:
            - npm ci
            - npm test
      - step:
          name: Build
          caches: [node]
          script:
            - npm ci
            - npm run build
          artifacts: [dist/**]
      - step:
          name: Deploy production
          deployment: production
          trigger: manual            # merge xong, lead bấm nút mới deploy
          script:
            - ./scripts/deploy.sh production dist/

    'release/*':
      - step:
          name: Deploy staging
          deployment: staging
          script:
            - ./scripts/deploy.sh staging

  # Tag vX.Y.Z: đóng gói release
  tags:
    'v*':
      - step:
          name: Tạo release package
          caches: [node]
          script:
            - npm ci && npm run build
            - tar -czf release-$BITBUCKET_TAG.tar.gz dist/
          artifacts: ['release-*.tar.gz']

  custom:
    rollback:
      - step:
          name: Rollback production
          deployment: production
          script:
            - ./scripts/rollback.sh
```

---

## 7. Caches và Artifacts

### 7.1. Caches — tăng tốc build bằng cách lưu dependencies

Cache lưu thư mục dependencies **giữa các lần chạy pipeline** (tối đa 7 ngày không dùng sẽ bị xóa,
tổng dung lượng cache 1GB/repo trên gói Free, 5GB trên Standard).

**Cache có sẵn của Atlassian** (không cần khai báo `definitions`):

| Tên cache | Thư mục được cache | Dùng cho |
|---|---|---|
| `node` | `node_modules/` | Node.js |
| `composer` | `~/.composer/cache` | PHP |
| `pip` | `~/.cache/pip` | Python |
| `maven` | `~/.m2/repository` | Java Maven |
| `gradle` | `~/.gradle` | Java Gradle |
| `sbt` | `~/.sbt`, `~/.ivy2` | Scala |
| `ivy` | `~/.ivy2` | Java Ivy |
| `bazel` | output dirs | Bazel |

**Cache tự định nghĩa:**

```yaml
definitions:
  caches:
    bundler: ./vendor/bundle          # Ruby bundler
    cargo-custom: ~/.cargo            # Rust

pipelines:
  default:
    - step:
        name: Test Rust
        image: rust:1.78
        caches:
          - cargo-custom
        script:
          - cargo test
```

**Cơ chế hoạt động:**

1. Step bắt đầu → Pipelines tải cache (nếu có) về đúng thư mục.
2. Script chạy (ví dụ `npm ci` vẫn chạy nhưng nhanh hơn vì phần lớn đã có sẵn —
   thực tế nhiều team dùng `npm install` kết hợp cache để tận dụng tối đa).
3. Step thành công → thư mục cache được nén và upload, **ghi đè** bản cũ.

> **Mẹo:** cache lưu theo **nhánh** — nhánh mới tách ra kế thừa cache từ nhánh nguồn,
> giúp build nhánh mới vẫn nhanh ngay lần đầu.

### 7.2. Artifacts — truyền file giữa các step

Artifacts là file/thư mục step hiện tại tạo ra, step **sau** (trong cùng pipeline) có thể dùng.
Hết pipeline, artifacts còn tải được từ UI trong 14 ngày.

```yaml
pipelines:
  default:
    - step:
        name: Build
        image: node:20
        caches: [node]
        script:
          - npm ci
          - npm run build
          - npm run test -- --coverage --coverageReporters=cobertura
        artifacts:
          - dist/**                    # thư mục build
          - coverage/cobertura-coverage.xml

    - step:
        name: Deploy
        image: node:20
        script:
          - ls dist/                   # dist/ đã có sẵn nhờ artifacts step trước
          - ./deploy.sh
```

**Lưu ý quan trọng:**

- Artifacts chỉ truyền **xuôi** theo thứ tự step (step sau nhận của step trước),
  không truyền ngược, không truyền sang pipeline khác.
- Mỗi step có thể khai báo `artifacts` riêng; step sau nhận **cộng dồn** artifacts
  của tất cả step trước.
- Tổng artifacts mỗi step tối đa **1GB**.
- Không dùng artifacts để thay cache dependencies — cache đúng mục đích hơn.

### 7.3. So sánh nhanh

|  | Cache | Artifacts |
|---|---|---|
| Mục đích | Lưu dependencies để build nhanh | Chuyển file build giữa các step |
| Phạm vi | Giữa các lần chạy pipeline | Trong cùng một lần chạy pipeline |
| Thời gian sống | 7 ngày (không dùng thì xóa) | 14 ngày (tải tay từ UI) |
| Giới hạn | 1GB (Free) / 5GB (Standard) | 1GB / step |

---

## 8. Deployments & Environments

### 8.1. Khái niệm

**Deployment** gắn một step với một **môi trường** (`test`, `staging`, `production` —
hoặc tên tự đặt). Lợi ích:

- UI **Deployments** hiển thị lịch sử deploy từng môi trường: ai deploy, commit nào, lúc nào.
- Mỗi môi trường có **biến riêng** (deployment variables) — ví dụ API key production khác staging.
- Có thể bật **yêu cầu duyệt / restrict** ai được deploy production (gói Premium).

```yaml
pipelines:
  branches:
    main:
      - step:
          name: Deploy staging
          deployment: staging        # môi trường "staging"
          script:
            - ./deploy.sh staging
      - step:
          name: Deploy production
          deployment: production
          trigger: manual            # an toàn: bấm tay mới deploy prod
          script:
            - ./deploy.sh production
```

### 8.2. Deployment variables

Vào **Repository settings → Pipelines → Deployments** → chọn môi trường →
thêm biến. Biến ở đây **ghi đè** biến repository cùng tên khi step chạy với `deployment:` tương ứng.

Ví dụ:

| Môi trường | `API_URL` | `API_KEY` (secured) |
|---|---|---|
| staging | `https://api-staging.congty.vn` | `sk-staging-xxx` |
| production | `https://api.congty.vn` | `sk-live-yyy` |

```yaml
- step:
    name: Deploy
    deployment: production
    script:
      - echo "Deploy tới $API_URL"   # https://api.congty.vn
```

### 8.3. Ví dụ deploy thật: SCP lên VPS

```yaml
- step:
    name: Deploy lên VPS staging
    deployment: staging
    image: atlassian/default-image:3   # image mặc định có sẵn ssh, rsync
    script:
      - mkdir -p ~/.ssh
      # SSH_KEY là biến secured chứa private key
      - echo "$SSH_KEY" > ~/.ssh/id_rsa && chmod 600 ~/.ssh/id_rsa
      - ssh-keyscan -H $VPS_HOST >> ~/.ssh/known_hosts
      - rsync -avz --delete dist/ deploy@$VPS_HOST:/var/www/staging/
      - ssh deploy@$VPS_HOST "sudo systemctl reload nginx"
```

### 8.4. Ví dụ deploy thật: AWS S3 + CloudFront (static site)

```yaml
- step:
    name: Deploy S3 + CloudFront
    deployment: production
    image: amazon/aws-cli:2.15.32
    script:
      - aws s3 sync dist/ s3://$S3_BUCKET --delete
      - aws cloudfront create-invalidation --distribution-id $CF_DISTRIBUTION_ID --paths "/*"
```

(biến `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `S3_BUCKET`, `CF_DISTRIBUTION_ID`
khai báo trong deployment variables của môi trường `production`; hoặc dùng OIDC ở mục 12
để khỏi lưu access key.)

---

## 9. Repository và Workspace Variables

### 9.1. Ba cấp độ biến

| Cấp | Nơi đặt | Phạm vi |
|---|---|---|
| **Workspace variables** | Workspace settings → Pipelines → Workspace variables | Mọi repo trong workspace |
| **Repository variables** | Repository settings → Pipelines → Repository variables | Repo hiện tại |
| **Deployment variables** | Repository settings → Pipelines → Deployments → chọn môi trường | Step có `deployment:` tương ứng |

**Thứ tự ưu tiên** (cao → thấp): Deployment > Repository > Workspace.
Biến cấp cao hơn **ghi đè** biến cùng tên ở cấp thấp hơn.

### 9.2. Biến secured (bí mật)

Khi tạo biến, tick **Secured** nếu giá trị là bí mật (password, API key, private key):

- Giá trị bị che (`****`) trên UI và trong log — Pipelines tự redact cả khi bạn `echo` nhầm.
- **Chỉ dùng được trong `script`**, không dùng trong `image:` hay `services:` ở một số trường hợp —
  thực tế `image.password: $VAR` vẫn hoạt động vì được resolve trước khi chạy.
- Không thể xem lại giá trị sau khi lưu — quên thì tạo lại.

```yaml
# Dùng biến secured trong script
- step:
    name: Login registry
    script:
      - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
```

> **Mẹo an toàn:** không bao giờ `echo` biến secured ra log để debug. Nếu nghi biến sai,
> kiểm tra độ dài thay vì giá trị: `echo ${#MY_SECRET}` (in ra số ký tự, không lộ nội dung).

### 9.3. Khai báo biến ngay trong YAML (không bí mật)

```yaml
- step:
    name: Build
    script:
      - npm run build
    variables:              # chỉ dùng cho giá trị KHÔNG bí mật
      NODE_ENV: production
      API_URL: https://api.congty.vn
```

### 9.4. Checklist quản lý biến cho team

1. Secret (key, token, password) → **luôn tick Secured**.
2. Giá trị khác nhau giữa môi trường → dùng **deployment variables**, không `if` trong script.
3. Giá trị dùng chung nhiều repo (ví dụ Docker registry công ty) → **workspace variables**.
4. Xoay (rotate) secret định kỳ; sau khi đổi secret phải chạy lại pipeline để kiểm tra.
5. Không commit secret vào repo — dùng `git-secrets` hoặc pre-commit hook để chặn.

---

## 10. Self-hosted Runners

### 10.1. Runner là gì, khi nào cần

Mặc định step chạy trên hạ tầng cloud của Atlassian ("cloud runner").
**Self-hosted runner** cho phép chạy step trên máy của chính bạn (server công ty, VM nội bộ):

- Cần truy cập tài nguyên **mạng nội bộ** (database, artifact repo không public).
- Cần **phần cứng đặc biệt** (GPU, nhiều RAM/disk hơn cloud).
- Yêu cầu **bảo mật**: code và secret không rời khỏi hạ tầng công ty.
- Muốn **không tốn build minutes** của Atlassian (runner tự host không tính minutes).

> Runner Bitbucket **chỉ hỗ trợ Linux + Docker**. Không có Windows/macOS runner chính thức.

### 10.2. Tạo runner trên Bitbucket

1. **Workspace settings** (runner cấp workspace, dùng chung nhiều repo) hoặc
   **Repository settings → Pipelines → Runners** (runner riêng cho repo).
2. **Add runner** → đặt tên (ví dụ `runner-ha-noi-01`) → chọn labels
   (ví dụ `self.hosted`, `gpu`) → **Create**.
3. Bitbucket hiện **lệnh Docker** để chạy runner — copy lệnh này.

### 10.3. Cài runner trên máy Linux

```bash
# 1. Cài Docker (Ubuntu/Debian)
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER && newgrp docker

# 2. Chạy runner (lệnh copy từ Bitbucket — ví dụ)
docker run -d --restart=always \
  --name bitbucket-runner \
  -e ACCOUNT_UUID='{xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx}' \
  -e REPOSITORY_UUID='{yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy}' \
  -e RUNNER_UUID='{zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz}' \
  -e OAUTH_CLIENT_ID='ABC123' \
  -e OAUTH_CLIENT_SECRET='XYZ789' \
  -e WORKING_DIRECTORY='/tmp' \
  -v /tmp:/tmp \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/containers:/var/lib/docker/containers:ro \
  docker-public.packages.atlassian.com/sox/atlassian/bitbucket-pipelines-runner:1
```

Giải thích các mount quan trọng:

- `/var/run/docker.sock`: runner cần Docker để khởi tạo build container.
- `OAUTH_CLIENT_ID/SECRET`: runner xác thực với Bitbucket Cloud để nhận việc.

Kiểm tra runner đã online:

```bash
docker logs bitbucket-runner --tail 20
# Thấy dòng "Runner started successfully" là OK — trên UI Bitbucket cũng hiện trạng thái Online
```

### 10.4. Dùng runner trong YAML

```yaml
pipelines:
  default:
    - step:
        name: Build nội bộ
        runs-on:                       # chỉ định runner
          - 'self.hosted'              # khớp theo label
          - 'linux'
        image: node:20
        script:
          - npm ci
          - npm test
          - curl http://db-noi-bo:5432/health   # truy cập được mạng nội bộ
```

- `runs-on` nhận danh sách **labels** — runner nào có đủ labels mới nhận step.
- Step không có `runs-on` vẫn chạy trên cloud như bình thường.

### 10.5. Vận hành runner thực tế

```bash
# Cập nhật runner lên bản mới
docker pull docker-public.packages.atlassian.com/sox/atlassian/bitbucket-pipelines-runner:1
docker stop bitbucket-runner && docker rm bitbucket-runner
# chạy lại lệnh docker run như trên

# Giới hạn tài nguyên (tránh runner chiếm hết máy chủ)
docker update --memory 16g --cpus 8 bitbucket-runner
```

**Lưu ý:**

- Mỗi runner mặc định chạy **1 step tại một thời điểm** — cần song song thì chạy nhiều container runner.
- Runner không tự dọn Docker image cũ: đặt cron `docker image prune -af` hàng tuần.
- Đặt runner sau firewall: chỉ cần **outbound HTTPS** tới bitbucket.org là đủ, không cần mở inbound port.

---

## 11. Docker trong Pipelines

### 11.1. Bật Docker service

Để chạy lệnh `docker` trong step, cần Docker-in-Docker. Hai cách:

**Cách 1 — bật toàn file:**

```yaml
options:
  docker: true
```

**Cách 2 — bật từng step:**

```yaml
- step:
    name: Build image
    services:
      - docker          # service "docker" có sẵn, không cần khai báo definitions
    script:
      - docker build -t myapp .
```

### 11.2. Build & push image lên Docker Hub

```yaml
image: atlassian/default-image:3

pipelines:
  branches:
    main:
      - step:
          name: Build & Push Docker image
          services:
            - docker
          caches:
            - docker                       # cache layer Docker giữa các lần build
          script:
            - IMAGE=congty/shop-api
            - VERSION=$BITBUCKET_BUILD_NUMBER
            - docker build -t $IMAGE:$VERSION -t $IMAGE:latest .
            - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            - docker push $IMAGE:$VERSION
            - docker push $IMAGE:latest
```

Dockerfile ví dụ (multi-stage, image nhỏ):

```dockerfile
# ---- build stage ----
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# ---- runtime stage ----
FROM node:20-alpine
WORKDIR /app
ENV NODE_ENV=production
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### 11.3. Push lên AWS ECR

```yaml
- step:
    name: Push lên ECR
    image: amazon/aws-cli:2.15.32
    services:
      - docker
    script:
      - ECR=123456789012.dkr.ecr.ap-southeast-1.amazonaws.com
      - aws ecr get-login-password --region ap-southeast-1 \
          | docker login --username AWS --password-stdin $ECR
      - docker build -t $ECR/shop-api:$BITBUCKET_BUILD_NUMBER .
      - docker push $ECR/shop-api:$BITBUCKET_BUILD_NUMBER
```

> Với OIDC (mục 12), step trên không cần `AWS_ACCESS_KEY_ID` nữa — role được gán tạm thời.

### 11.4. Docker Compose trong pipeline

```yaml
- step:
    name: Test với docker-compose
    image: docker:24
    services:
      - docker
    script:
      - apk add --no-cache docker-compose-plugin 2>/dev/null || true
      - docker compose -f docker-compose.test.yml up --build --abort-on-container-exit
      - docker compose -f docker-compose.test.yml down -v
```

### 11.5. Lưu ý về cache `docker`

- Cache `docker` của Atlassian lưu `/var/lib/docker` — giúp `docker build` tái sử dụng layer.
- Kết hợp BuildKit cache mount trong Dockerfile để tối ưu hơn:

```dockerfile
RUN --mount=type=cache,target=/root/.npm npm ci
```

---

## 12. OIDC với AWS/GCP

**OIDC (OpenID Connect)** cho phép pipeline lấy **credential tạm thời** của AWS/GCP
mà **không cần lưu access key** trong variables — an toàn hơn hẳn.

### 12.1. Cơ chế

```
Pipeline ──▶ xin OIDC token từ Bitbucket (JWT, có kèm repo/branch/step info)
    │
    ▼
AWS IAM / GCP ──▶ xác thực JWT ──▶ cấp credential tạm thời (15–60 phút)
    │
    ▼
aws cli / gcloud dùng credential tạm thời — hết hạn tự mất hiệu lực
```

### 12.2. OIDC với AWS

**Bước 1 — Tạo Identity Provider trong AWS IAM:**

- Console → IAM → Identity providers → Add provider → OpenID Connect.
- Provider URL: `https://api.bitbucket.org/2.0/workspaces/congty/pipelines-config/identity/oidc`
  (thay `congty` bằng tên workspace của bạn).
- Audience: `api.bitbucket.org/2.0/workspaces/congty/pipelines-config/identity/oidc`
  (Bitbucket gợi ý chính xác URL này trong **Repository settings → Pipelines → OpenID Connect** —
  copy từ đó cho chắc).

**Bước 2 — Tạo IAM Role với trust policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/api.bitbucket.org/2.0/workspaces/congty/pipelines-config/identity/oidc"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringLike": {
          "api.bitbucket.org/2.0/workspaces/congty/pipelines-config/identity/oidc:sub":
            "congty/shop-api:*:*" 
        }
      }
    }
  ]
}
```

Điều kiện `sub` có dạng `{workspace}/{repo}:{branch}:{step}` — ví dụ chỉ cho nhánh main deploy:

```json
"StringEquals": {
  "api.bitbucket.org/2.0/workspaces/congty/pipelines-config/identity/oidc:sub":
    "congty/shop-api:main:*"
}
```

**Bước 3 — Dùng trong pipeline với pipe chính chủ:**

```yaml
- step:
    name: Deploy AWS bằng OIDC
    oidc: true                       # xin OIDC token cho step này
    image: amazon/aws-cli:2.15.32
    script:
      - pipe: atlassian/aws-ecr-push-image:2.3.0
        variables:
          AWS_OIDC_ROLE_ARN: 'arn:aws:iam::123456789012:role/bitbucket-pipelines-role'
          AWS_REGION: 'ap-southeast-1'
          IMAGE_NAME: 'shop-api'
          TAGS: '$BITBUCKET_BUILD_NUMBER latest'
```

Hoặc tự assume role bằng tay:

```bash
# Lấy OIDC token mà Bitbucket cấp cho step
OIDC_TOKEN=$(curl -s --request GET \
  --url "$BITBUCKET_STEP_OIDC_TOKEN_URL" \
  --header "Authorization: Bearer $BITBUCKET_STEP_OIDC_TOKEN" | jq -r '.token')

# Đổi token lấy credential AWS tạm thời
CREDS=$(aws sts assume-role-with-web-identity \
  --role-arn arn:aws:iam::123456789012:role/bitbucket-pipelines-role \
  --role-session-name "bb-$BITBUCKET_BUILD_NUMBER" \
  --web-identity-token "$OIDC_TOKEN" \
  --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]' \
  --output text)

export AWS_ACCESS_KEY_ID=$(echo "$CREDS" | cut -f1)
export AWS_SECRET_ACCESS_KEY=$(echo "$CREDS" | cut -f2)
export AWS_SESSION_TOKEN=$(echo "$CREDS" | cut -f3)

aws s3 sync dist/ s3://my-bucket --delete
```

### 12.3. OIDC với GCP

**Bước 1 — Tạo Workload Identity Pool trong GCP:**

```bash
# Tạo pool
gcloud iam workload-identity-pools create bitbucket-pool \
  --location=global \
  --description="Bitbucket Pipelines OIDC"

# Thêm Bitbucket làm provider
gcloud iam workload-identity-pools providers create-oidc bitbucket-provider \
  --location=global \
  --workload-identity-pool=bitbucket-pool \
  --issuer-uri="https://api.bitbucket.org/2.0/workspaces/congty/pipelines-config/identity/oidc" \
  --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.sub"
```

**Bước 2 — Gán quyền cho service account:**

```bash
gcloud iam service-accounts add-iam-policy-binding \
  deployer@du-an-123.iam.gserviceaccount.com \
  --role=roles/iam.workloadIdentityUser \
  --member="principalSet://iam.googleapis.com/projects/123/locations/global/workloadIdentityPools/bitbucket-pool/attribute.repository/congty/shop-api"
```

**Bước 3 — Dùng trong pipeline:**

```yaml
- step:
    name: Deploy GCP bằng OIDC
    oidc: true
    image: google/cloud-sdk:slim
    script:
      - pipe: atlassian/google-cloud-storage-deploy:0.4.2
        variables:
          KEY_FILE: ''   # bỏ trống khi dùng OIDC
          PROJECT: 'du-an-123'
          BUCKET: 'gs://web-congty'
          SOURCE: 'dist/'
```

---

## 13. Parallel steps và Conditions

### 13.1. Parallel — chạy nhiều step cùng lúc

Các step trong cùng một "nhóm" (liệt kê cạnh nhau) chạy **song song**:

```yaml
pipelines:
  default:
    - parallel:                    # nhóm parallel: 3 step chạy đồng thời
        - step:
            name: Unit test
            image: node:20
            caches: [node]
            script:
              - npm ci
              - npm run test:unit
        - step:
            name: Lint
            image: node:20
            caches: [node]
            script:
              - npm ci
              - npm run lint
        - step:
            name: Security scan
            image: node:20
            script:
              - npm audit --audit-level=high
    - step:
        name: Build              # chạy SAU khi cả 3 step trên xong
        image: node:20
        script:
          - npm ci && npm run build
```

Quy tắc:

- Tối đa **100 step** trong một nhóm parallel.
- Nhóm parallel **fail** nếu bất kỳ step nào fail → các step sau nhóm không chạy.
- Mỗi step parallel vẫn là container riêng — **không chia sẻ filesystem**:
  muốn dùng chung file phải qua artifacts… nhưng artifacts chỉ truyền xuôi,
  nên step parallel thường tự build độc lập từ source.
- Dùng `$BITBUCKET_PARALLEL_STEP` (0, 1, 2…) để chia việc, ví dụ chia test suite:

```yaml
- parallel:
    - step:
        name: Test nhóm A
        script:
          - npm test -- --shard=$((BITBUCKET_PARALLEL_STEP+1))/$BITBUCKET_PARALLEL_STEP_COUNT
    - step:
        name: Test nhóm B
        script:
          - npm test -- --shard=$((BITBUCKET_PARALLEL_STEP+1))/$BITBUCKET_PARALLEL_STEP_COUNT
```

### 13.2. Condition — chạy step theo file thay đổi

```yaml
pipelines:
  default:
    - step:
        name: Test backend
        condition:
          changesets:
            includePaths:
              - "backend/**"        # chỉ chạy khi có file trong backend/ thay đổi
        script:
          - cd backend && go test ./...
    - step:
        name: Test frontend
        condition:
          changesets:
            includePaths:
              - "frontend/**"
        script:
          - cd frontend && npm ci && npm test
    - step:
        name: Luôn chạy
        script:
          - echo "step này không có condition nên luôn chạy"
```

- `includePaths` hỗ trợ glob (`**`, `*`).
- Có thể thêm `excludePaths` để loại trừ (ví dụ bỏ qua `docs/**`).
- Condition đánh giá trên **diff của commit/push hiện tại** so với base.

---

## 14. Ví dụ thực tế

### 14.1. Node.js: test + build + deploy (đầy đủ)

```yaml
image: node:20

definitions:
  services:
    postgres:
      image: postgres:15
      variables:
        POSTGRES_DB: shopdb
        POSTGRES_USER: shop
        POSTGRES_PASSWORD: $DB_PASSWORD   # biến secured ở repo variables

pipelines:
  pull-requests:
    '**':
      - parallel:
          - step:
              name: Lint & Unit test
              caches: [node]
              script:
                - npm ci
                - npm run lint
                - npm run test:unit
          - step:
              name: Integration test
              caches: [node]
              services: [postgres]
              script:
                - npm ci
                - npm run test:integration
              variables:
                DATABASE_URL: postgres://shop:$DB_PASSWORD@postgres:5432/shopdb

  branches:
    main:
      - step:
          name: Test
          caches: [node]
          script:
            - npm ci
            - npm run lint
            - npm test
      - step:
          name: Build
          caches: [node]
          script:
            - npm ci
            - npm run build
          artifacts:
            - dist/**
      - step:
          name: Deploy production
          deployment: production
          trigger: manual
          image: node:20-alpine
          script:
            - apk add --no-cache openssh-client rsync
            - mkdir -p ~/.ssh
            - echo "$SSH_KEY" > ~/.ssh/id_rsa && chmod 600 ~/.ssh/id_rsa
            - ssh-keyscan -H $VPS_HOST >> ~/.ssh/known_hosts
            - rsync -avz --delete dist/ deploy@$VPS_HOST:/var/www/shop/
            - ssh deploy@$VPS_HOST "cd /var/www/shop && pm2 reload ecosystem.config.js"

  custom:
    deploy-staging:
      - step:
          name: Deploy staging
          deployment: staging
          image: node:20-alpine
          script:
            - npm ci && npm run build
            - ./scripts/deploy.sh staging
```

### 14.2. Python: test + lint + build package

```yaml
image: python:3.12

pipelines:
  default:
    - step:
        name: Lint & Test
        caches: [pip]
        script:
          - pip install -r requirements.txt
          - pip install ruff pytest pytest-cov
          - ruff check src/ tests/
          - ruff format --check src/ tests/
          - pytest tests/ --cov=src --cov-report=xml
        artifacts:
          - coverage.xml

    - step:
        name: Build package
        caches: [pip]
        script:
          - pip install build
          - python -m build
        artifacts:
          - dist/*

  tags:
    'v*':
      - step:
          name: Publish lên PyPI
          caches: [pip]
          trigger: manual
          script:
            - pip install build twine
            - python -m build
            - twine upload --username __token__ --password "$PYPI_TOKEN" dist/*
```

### 14.3. Docker: build & push image (multi-arch)

```yaml
image: atlassian/default-image:3

pipelines:
  branches:
    main:
      - step:
          name: Build & push multi-arch image
          services: [docker]
          caches: [docker]
          script:
            - IMAGE=$DOCKERHUB_USER/shop-api
            - TAG=$BITBUCKET_BUILD_NUMBER
            # Đăng nhập Docker Hub (biến secured)
            - echo "$DOCKERHUB_TOKEN" | docker login -u "$DOCKERHUB_USER" --password-stdin
            # Bật buildx để build multi-arch
            - docker buildx create --use --name multiarch || docker buildx use multiarch
            - docker buildx build --platform linux/amd64,linux/arm64 \
                -t $IMAGE:$TAG -t $IMAGE:latest \
                --push .
            - echo "Đã push $IMAGE:$TAG"
```

### 14.4. Monorepo: chỉ build service bị thay đổi

```yaml
pipelines:
  default:
    - parallel:
        - step:
            name: API service
            image: node:20
            condition:
              changesets:
                includePaths: ["services/api/**"]
            script:
              - cd services/api && npm ci && npm test && npm run build
        - step:
            name: Web service
            image: node:20
            condition:
              changesets:
                includePaths: ["services/web/**"]
            script:
              - cd services/web && npm ci && npm test && npm run build
```

### 14.5. Dùng Pipe có sẵn (atlassian pipes)

Pipes là các "action đóng gói sẵn" của Atlassian và cộng đồng:

```yaml
- step:
    name: Gửi thông báo Slack
    script:
      - pipe: atlassian/slack-notify:2.3.0
        variables:
          WEBHOOK_URL: $SLACK_WEBHOOK_URL
          MESSAGE: "Build #$BITBUCKET_BUILD_NUMBER của $BITBUCKET_REPO_SLUG: thành công ✅"

- step:
    name: Deploy lên Heroku
    deployment: production
    script:
      - pipe: atlassian/heroku-deploy:1.2.3
        variables:
          HEROKU_API_KEY: $HEROKU_API_KEY
          HEROKU_APP_NAME: 'shop-api-prod'
```

Danh sách pipe xem tại `bitbucket.org/product/features/pipelines/integrations`.

---

## 15. Build minutes và pricing

### 15.1. Build minutes là gì

- **1 build minute = 1 phút** chạy step trên cloud runner size `1x`.
- Step size `2x` (8GB RAM): **1 phút thực = 2 build minutes**.
- Self-hosted runner: **không tốn** build minutes.
- Step chạy song song: minutes tính **cộng dồn** từng step.

### 15.2. Hạn mức theo gói (Bitbucket Cloud)

| Gói | Build minutes/tháng | Storage | Ghi chú |
|---|---|---|---|
| **Free** | 50 | 1 GB | Tối đa 5 user |
| **Standard** | 2.500 | 250 GB | Tính theo user |
| **Premium** | Không giới hạn* | Không giới hạn* | Có deployment permissions, merge checks |

> *Premium: "không giới hạn" theo chính sách fair use của Atlassian.
> Số liệu có thể thay đổi — kiểm tra trang pricing chính thức của Atlassian
> trước khi lập ngân sách.

Hết minutes giữa tháng: pipeline mới **không chạy** (hàng đợi) cho đến khi sang tháng mới
hoặc nâng gói. Vào **Repository settings → Pipelines → Settings** xem mức đã dùng.

### 15.3. Mẹo tiết kiệm build minutes

1. **Dùng cache** (`node`, `pip`, `maven`…) — giảm 30–70% thời gian cài dependencies.
2. **Image nhỏ** (`-alpine`, `-slim`) — pull nhanh hơn, ít tốn minutes chờ.
3. **Condition theo file thay đổi** — monorepo không build lại service không liên quan.
4. **`[skip ci]`** cho commit docs/README.
5. **Gộp lệnh** hợp lý — mỗi step có overhead khởi tạo container ~30–60 giây;
   đừng tách pipeline thành quá nhiều step tí hon.
6. **Self-hosted runner** cho workload nặng, chạy thường xuyên.
7. Tránh `npm install` khi `npm ci` đủ dùng; tránh build lại Docker layer không đổi
   (sắp xếp Dockerfile: COPY package files → install → COPY source sau cùng).

**Ví dụ tính toán:** pipeline Node.js chạy 4 phút/step × 3 step nối tiếp = 12 minutes/lần chạy.
Ngày push 10 lần → 120 minutes/ngày → ~2.600/tháng — vượt xa gói Free,
cần Standard hoặc tối ưu (cache + parallel + skip ci).

---

## 16. Troubleshooting

### 16.1. Pipeline không chạy sau khi push

| Nguyên nhân | Cách kiểm tra / sửa |
|---|---|
| Chưa bật Pipelines | Repository settings → Pipelines → Enable Pipelines |
| File sai tên/vị trí | Phải đúng `bitbucket-pipelines.yml` ở **gốc repo**, đúng chính tả |
| YAML lỗi cú pháp | Dùng tab thay vì space; kiểm tra bằng `yamllint` hoặc online YAML validator |
| Commit có `[skip ci]` | Xóa cụm đó khỏi message và push lại |
| Nhánh không khớp pattern nào và không có `default` | Thêm `default:` hoặc pattern `**` |
| Hết build minutes | Kiểm tra usage trong Repository/Workspace settings |

### 16.2. Lỗi YAML thường gặp

```yaml
# ❌ SAI: dùng tab để indent
pipelines:
→ default:

# ✅ ĐÚNG: dùng 2 spaces
pipelines:
  default:

# ❌ SAI: script không phải list
script:
  echo "hello"

# ✅ ĐÚNG: mỗi lệnh là một item
script:
  - echo "hello"

# ❌ SAI: quên dấu - trước step
pipelines:
  default:
    step:
      script:
        - echo hi

# ✅ ĐÚNG
pipelines:
  default:
    - step:
        script:
          - echo hi

# ⚠️ CẨN THẬN: giá trị bắt đầu bằng * hoặc chứa : cần quote
variables:
  PATTERN: "*"          # phải quote, nếu không YAML hiểu là alias
  MESSAGE: "deploy: xong"  # chứa dấu : phải quote
```

### 16.3. Container / Docker

**`docker: command not found`**
→ Chưa bật Docker service. Thêm `options: { docker: true }` hoặc `services: [docker]` cho step.

**`Cannot connect to the Docker daemon`**
→ Service `docker` chưa kịp khởi động. Thêm `sleep 5` đầu script, hoặc kiểm tra image
có tương thích Docker-in-Docker không.

**Build Docker chậm mỗi lần**
→ Thêm `caches: [docker]` và tối ưu thứ tự layer trong Dockerfile.

### 16.4. Cache & artifacts

**Cache không có tác dụng (vẫn cài lại từ đầu)**
- Kiểm tra tên cache đúng (`node`, `pip`…) và thư mục khớp.
- Cache chỉ lưu khi step **thành công** — step fail thì cache không được cập nhật.
- Nhánh mới tách ra lần đầu dùng cache của nhánh nguồn — sau đó cache riêng.

**Step sau không thấy file của step trước**
→ Step trước thiếu `artifacts:`. Nhớ: artifacts chỉ đi xuôi, step parallel không
thấy artifacts của nhau.

**`Artifact too large`**
→ Mỗi step tối đa 1GB artifacts. Loại trừ file không cần bằng pattern cụ thể
thay vì `**`.

### 16.5. Variables & secrets

**Biến secured bị hiện `****` nhưng lệnh báo sai**
→ Giá trị có thể dính khoảng trắng/xuống dòng khi paste. Tạo lại biến, hoặc kiểm tra
độ dài: `echo ${#MY_VAR}`.

**Biến không có giá trị trong step**
- Kiểm tra step có `deployment:` không — deployment variables chỉ áp dụng cho đúng môi trường.
- Biến workspace/repo bị biến deployment cùng tên **ghi đè**.

**`Image pull failed: unauthorized` (private registry)**
→ Kiểm tra `image.username/password` — với biến secured, đảm bảo đã lưu đúng và không hết hạn.

### 16.6. OIDC

**`sts:AssumeRoleWithWebIdentity` báo lỗi**
- Kiểm tra `oidc: true` đã có trong step.
- Provider URL trong AWS phải **khớp từng ký tự** với URL trong
  Repository settings → Pipelines → OpenID Connect.
- Điều kiện `sub` phải khớp format `{workspace}/{repo}:{branch}:{step}` —
  test bằng `StringLike` với `*` trước, siết chặt sau.

### 16.7. Runner tự host

**Runner hiện Offline**
- Kiểm tra `docker logs bitbucket-runner` — lỗi thường do sai `OAUTH_CLIENT_SECRET`
  hoặc máy không ra internet (cần outbound HTTPS tới bitbucket.org).
- Kiểm tra đồng hồ máy chủ (clock skew) — OAuth rất nhạy với lệch giờ: `timedatectl`.

**Step treo ở trạng thái Pending mãi**
→ Không có runner nào khớp labels trong `runs-on`. Kiểm tra labels của runner trên UI.

### 16.8. Đọc log hiệu quả

1. Mở pipeline → click step bị fail → xem log từng lệnh (mỗi lệnh có exit code).
2. Lệnh fail luôn có dòng màu đỏ + `exit code: N` — đọc **20–30 dòng ngay phía trên**,
   đó mới là lỗi thật (dòng cuối thường chỉ là tóm tắt).
3. Tải **full log** (nút Download) khi cần tìm bằng Ctrl+F.
4. Bật debug tạm thời: thêm `set -x` đầu script để in từng lệnh khi chạy.

---

## 17. Best practices

### 17.1. Tốc độ & chi phí

- **Cache mọi thứ cài đi cài lại**: dependencies, Docker layers, build tools.
- **Fail fast**: chạy lint + unit test **trước**, integration test nặng sau —
  lỗi sớm thì dừng sớm, đỡ tốn minutes.
- **Parallel** cho các job độc lập (lint / unit / security scan).
- **Image nhỏ, cố định tag**: `node:20.11-alpine` thay vì `node:latest` —
  reproducible và tránh bị đổi version bất ngờ.

### 17.2. An toàn

- Secret → **secured variables** hoặc **OIDC**; không bao giờ commit vào repo.
- Production deploy → `trigger: manual` + `deployment: production`
  (+ deployment permissions nếu dùng Premium).
- Pin version của pipe: `atlassian/aws-ecr-push-image:2.3.0`, không dùng `:latest`.
- Quét secret trước khi push: pre-commit hook với `gitleaks` hoặc `git-secrets`.
- Xoay secret định kỳ; mỗi lần đổi chạy thử pipeline ngay.

### 17.3. Bảo trì file YAML

- Dùng **YAML anchors** (`&`/`*`) cho step lặp lại (test matrix nhiều version Node…).
- Đặt `name` rõ nghĩa cho mọi step — người on-call lúc 2h sáng cảm ơn bạn.
- Comment ngắn gọn bằng tiếng Việt/Anh cho đoạn logic không hiển nhiên.
- Validate YAML trước khi push: `yamllint` hoặc `python -c "import yaml,sys; yaml.safe_load(open('bitbucket-pipelines.yml'))"`.
- Giữ file dưới ~300 dòng — dài hơn thì tách script phức tạp ra file `.sh` riêng trong repo.

### 17.4. Chiến lược pipeline theo team

```
PR → pull-requests: lint + unit test (nhanh, <5 phút)
  ↓ merge
main → branches.main: test đầy đủ + build + artifacts
  ↓ bấm nút tay
production → deployment:production, trigger: manual
  ↓
tag v* → đóng gói release / publish package
```

- Quy ước commit message: `[skip ci]` cho docs; issue key Jira (`PROJ-123`) để link ticket.
- Branch protection: bật **merge check** "Minimum successful builds" để không merge PR đỏ.

---

## 18. Cheat sheet

### 18.1. Khung file đầy đủ nhất (copy-paste)

```yaml
image: node:20

definitions:
  caches:
    my-cache: ./custom-dir
  services:
    db:
      image: postgres:15
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: user
        POSTGRES_PASSWORD: pass

options:
  docker: true
  max-time: 60

pipelines:
  default:
    - step:
        name: Tên step
        image: node:20-alpine
        caches: [node]
        services: [db]
        artifacts: [dist/**]
        trigger: automatic      # hoặc manual
        deployment: staging     # hoặc test/production
        max-time: 30
        size: 1x                # hoặc 2x
        oidc: true
        runs-on: ['self.hosted']
        variables:
          NODE_ENV: production
        condition:
          changesets:
            includePaths: ["src/**"]
        script:
          - echo "lệnh 1"
          - echo "lệnh 2"

  branches:
    main: [...]
    'feature/*': [...]

  pull-requests:
    '**': [...]

  tags:
    'v*': [...]

  custom:
    ten-pipeline: [...]
```

### 18.2. Biến môi trường hay dùng

```bash
$BITBUCKET_BUILD_NUMBER        # số build: 42
$BITBUCKET_COMMIT               # SHA commit
$BITBUCKET_BRANCH               # tên nhánh
$BITBUCKET_TAG                 # tên tag
$BITBUCKET_PR_ID               # ID pull request
$BITBUCKET_REPO_SLUG           # slug repo
$BITBUCKET_WORKSPACE           # tên workspace
$BITBUCKET_DEPLOYMENT_ENVIRONMENT  # staging/production
```

### 18.3. Lệnh CLI hữu ích

```bash
# Validate YAML trước khi push
python3 -c "import yaml; yaml.safe_load(open('bitbucket-pipelines.yml')); print('YAML OK')"

# Chạy thử Docker image giống hệt môi trường pipeline
docker run --rm -it -v "$PWD":/opt/atlassian/pipelines/agent/build \
  -w /opt/atlassian/pipelines/agent/build node:20 bash

# Trigger custom pipeline bằng API
curl -X POST -u "$BB_USER:$BB_APP_PASSWORD" \
  -H "Content-Type: application/json" \
  https://api.bitbucket.org/2.0/repositories/WORKSPACE/REPO/pipelines/ \
  -d '{"target":{"ref_type":"branch","ref_name":"main","type":"pipeline_ref_target",
        "selector":{"type":"custom","pattern":"deploy-staging"}}}'

# Xem trạng thái pipeline mới nhất của nhánh main
curl -u "$BB_USER:$BB_APP_PASSWORD" \
  "https://api.bitbucket.org/2.0/repositories/WORKSPACE/REPO/pipelines/?target.branch=main&pagelen=1"
```

### 18.4. Bảng tra nhanh cú pháp step

| Muốn… | Dùng |
|---|---|
| Đổi image cho step | `image: python:3.12` |
| Chạy lệnh | `script: [- npm test]` |
| Cache dependencies | `caches: [node]` |
| Giữ file cho step sau | `artifacts: [dist/**]` |
| Chạy tay | `trigger: manual` |
| Gắn môi trường deploy | `deployment: production` |
| Tăng RAM gấp đôi | `size: 2x` |
| Giới hạn thời gian | `max-time: 30` |
| Dùng database phụ | `services: [postgres]` |
| Dùng Docker | `services: [docker]` hoặc `options: {docker: true}` |
| Bật OIDC | `oidc: true` |
| Chạy trên runner tự host | `runs-on: ['self.hosted']` |
| Chạy song song | `- parallel: [...]` |
| Chạy theo file đổi | `condition: {changesets: {includePaths: [...]}}` |
| Biến không bí mật | `variables: {KEY: value}` |

---

> **Ghi nhớ nhanh:** `bitbucket-pipelines.yml` ở gốc repo → mỗi step là một Docker container
> sạch → cache để nhanh, artifacts để truyền file, deployment để deploy có kiểm soát,
> OIDC để khỏi lưu access key, runner tự host khi cần mạng nội bộ hoặc tiết kiệm minutes.
> Hết.
