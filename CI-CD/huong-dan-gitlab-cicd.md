# Hướng Dẫn GitLab CI/CD Toàn Tập

> Từ zero đến hero: pipeline, runner, deploy, Container Registry, Auto DevOps — tất cả trong một file.

---

## Mục lục

1. [GitLab CI/CD là gì?](#1-gitlab-cicd-là-gì)
2. [Khái niệm cốt lõi: Pipeline, Stages, Jobs, Runners](#2-khái-niệm-cốt-lõi-pipeline-stages-jobs-runners)
3. [Bắt đầu: tạo project và file `.gitlab-ci.yml` đầu tiên](#3-bắt-đầu-tạo-project-và-file-gitlab-ciyml-đầu-tiên)
4. [Cú pháp YAML đầy đủ](#4-cú-pháp-yaml-đầy-đủ)
5. [Triggers: cách kích hoạt pipeline](#5-triggers-cách-kích-hoạt-pipeline)
6. [Runners: shared, specific, cài đặt và đăng ký](#6-runners-shared-specific-cài-đặt-và-đăng-ký)
7. [Variables: biến môi trường trong CI](#7-variables-biến-môi-trường-trong-ci)
8. [Artifacts & Cache](#8-artifacts--cache)
9. [Environments: staging, production, review apps](#9-environments-staging-production-review-apps)
10. [Rules / only / except: điều kiện chạy job](#10-rules--only--except-điều-kiện-chạy-job)
11. [DAG với `needs`: chạy song song, phụ thuộc có hướng](#11-dag-với-needs-chạy-song-song-phụ-thuộc-có-hướng)
12. [Merge trains & pipeline cho Merge Request](#12-merge-trains--pipeline-cho-merge-request)
13. [Auto DevOps](#13-auto-devops)
14. [Container Registry: build & push Docker image](#14-container-registry-build--push-docker-image)
15. [Ví dụ thực tế hoàn chỉnh](#15-ví-dụ-thực-tế-hoàn-chỉnh)
16. [Pricing & giới hạn free tier](#16-pricing--giới-hạn-free-tier)
17. [Troubleshooting: 12 lỗi thường gặp](#17-troubleshooting-12-lỗi-thường-gặp)
18. [Best practices checklist](#18-best-practices-checklist)
19. [FAQ](#19-faq)
20. [Cheat sheet](#20-cheat-sheet)

---

## 1. GitLab CI/CD là gì?

**GitLab CI/CD** là hệ thống tích hợp liên tục (Continuous Integration) và triển khai liên tục (Continuous Delivery/Deployment) **tích hợp sẵn** trong GitLab. Không như GitHub cần thêm Actions hay Jenkins cần cài đặt riêng, với GitLab bạn chỉ cần:

1. Một repository Git trên GitLab (gitlab.com hoặc GitLab self-hosted).
2. Một file `.gitlab-ci.yml` đặt ở thư mục gốc của repo.

Mỗi lần bạn push code, GitLab tự động đọc file này và chạy pipeline.

### CI vs CD

| Khái niệm | Ý nghĩa |
|---|---|
| **CI (Continuous Integration)** | Mỗi lần commit, code được build + test tự động để phát hiện lỗi sớm |
| **CD (Continuous Delivery)** | Sau khi pass CI, code được đóng gói và sẵn sàng deploy — vẫn cần bấm nút hoặc merge để deploy thật |
| **CD (Continuous Deployment)** | Sau khi pass CI, code **tự động** deploy lên môi trường production không cần can thiệp tay |

### Vì sao chọn GitLab CI/CD?

- **All-in-one**: repo, issue, MR, CI/CD, registry, environments cùng một chỗ.
- **Không cần cài server CI**: GitLab.com cho sẵn shared runners miễn phí.
- **Cấu hình bằng YAML** đơn giản, version-control cùng với code.
- **Free tier hào phóng**: 400 CI minutes/tháng (tính đến 2026), Container Registry miễn phí.
- **Mạnh cho DevOps**: environments, review apps, merge trains, Auto DevOps.

---

## 2. Khái niệm cốt lõi: Pipeline, Stages, Jobs, Runners

### Pipeline

**Pipeline** là một lần chạy CI/CD hoàn chỉnh — tập hợp các jobs được thực thi theo thứ tự bạn định nghĩa. Một pipeline được tạo ra (triggered) bởi một sự kiện: push code, mở merge request, lịch định kỳ, bấm nút tay, hoặc gọi API.

Trạng thái của pipeline:

| Trạng thái | Ý nghĩa |
|---|---|
| `created` | Đã tạo, chờ runner nhận |
| `pending` | Chờ tài nguyên |
| `running` | Đang chạy |
| `success` | Tất cả jobs thành công |
| `failed` | Có job thất bại |
| `canceled` | Bị hủy |
| `skipped` | Bỏ qua (do rules) |
| `manual` | Đang chờ bấm nút chạy tay |

### Stages

**Stage** là một *giai đoạn* trong pipeline. Các stages chạy **tuần tự** (stage sau chỉ chạy khi stage trước thành công). Quy ước phổ biến: `build` → `test` → `deploy`.

### Jobs

**Job** là đơn vị công việc nhỏ nhất. Các jobs **trong cùng một stage chạy song song** (nếu có đủ runner). Mỗi job:

- Chạy trên một **runner** riêng biệt.
- Thực thi các lệnh trong `script`.
- Có thể dùng Docker image riêng (`image:`).
- Trả về `success` hoặc `failed`.

### Runners

**Runner** là agent (máy/process) thực sự chạy jobs. Có thể là máy của GitLab (shared runners) hoặc máy của bạn (self-hosted).

```
┌─────────────────────────────────────────────────┐
│                    PIPELINE                      │
│                                                  │
│  ┌─────────────┐  ┌─────────────┐  ┌───────────┐ │
│  │ Stage: build │→ │ Stage: test │→ │Stage:     │ │
│  │  job: build │  │ job: unit   │  │ deploy    │ │
│  │             │  │ job: lint   │  │ job: prod │ │
│  └─────────────┘  └─────────────┘  └───────────┘ │
│        │                │                │       │
│        └────────────────┴────────────────┘       │
│                         │                        │
│                    RUNNERS chạy jobs             │
└─────────────────────────────────────────────────┘
```

**Mối quan hệ**: 1 pipeline → N stages (tuần tự) → mỗi stage N jobs (song song) → mỗi job chạy trên 1 runner.

---

## 3. Bắt đầu: tạo project và file `.gitlab-ci.yml` đầu tiên

### Bước 1: Tạo project

1. Đăng nhập [gitlab.com](https://gitlab.com) (đăng ký miễn phí bằng email hoặc GitHub/Google).
2. Bấm **New project** → **Create blank project**.
3. Đặt tên (ví dụ `hello-gitlab-ci`), chọn visibility (**Private** khuyến nghị), bấm **Create project**.

### Bước 2: Thêm file `.gitlab-ci.yml`

Ngay trên trang project, bấm **+** → **New file**, đặt tên `.gitlab-ci.yml`, dán nội dung:

```yaml
stages:
  - build
  - test

build-job:
  stage: build
  script:
    - echo "Đang build..."
    - mkdir -p output
    - echo "Hello GitLab CI" > output/hello.txt

test-job:
  stage: test
  script:
    - echo "Đang test..."
    - cat output/hello.txt
```

Bấm **Commit changes** → GitLab tự động tạo pipeline đầu tiên. Vào **Build → Pipelines** để xem.

### Bước 3: Xem kết quả

- Pipeline hiện trạng thái `success` với 2 stages xanh.
- Bấm vào từng job để xem log chi tiết từng dòng lệnh.

> **Lưu ý quan trọng:** hai jobs trên chạy trên **runner khác nhau**, nên file `output/hello.txt` job build tạo ra **không tồn tại** ở job test! Muốn chia sẻ file giữa jobs phải dùng **artifacts** (xem mục 8).

### Cách xem pipeline

| Vị trí | Xem gì |
|---|---|
| **Build → Pipelines** | Danh sách pipelines, trạng thái, ai trigger, commit nào |
| Bấm vào 1 pipeline | Sơ đồ DAG: stages và jobs, thời gian chạy từng job |
| Bấm vào 1 job | Log chi tiết, artifacts tải về, nút retry/cancel |

---

## 4. Cú pháp YAML đầy đủ

### 4.1. `stages` — định nghĩa các giai đoạn

```yaml
stages:
  - build
  - test
  - deploy
```

- Nếu không khai báo `stages`, GitLab dùng mặc định: `build` → `test` → `deploy`.
- Job nào không khai báo `stage` sẽ rơi vào stage `test`.

### 4.2. `image` — Docker image chạy job

```yaml
# image toàn cục: áp dụng cho mọi job
image: python:3.12

# hoặc image riêng cho từng job
test-node:
  image: node:20-alpine
  script:
    - node --version
    - npm test
```

GitLab dùng Docker executor để pull image và chạy job bên trong container.

### 4.3. `script` — lệnh chính của job

```yaml
deploy-job:
  stage: deploy
  script:
    - echo "Bắt đầu deploy"
    - ./deploy.sh production
    - echo "Deploy xong"
```

- Mỗi dòng chạy trong **cùng một shell** (mặc định `sh`, có thể đổi bằng `before_script`... thực tế mỗi dòng là một lệnh shell riêng nhưng cùng working directory).
- Job **thất bại** ngay khi một dòng trả về exit code ≠ 0 (trừ khi dùng `allow_failure`).

### 4.4. `before_script` và `after_script`

```yaml
default:
  before_script:
    - echo "Chạy trước MỌI job"
    - apt-get update -qq && apt-get install -y -qq curl

after_script:
  - echo "Chạy sau MỌI job, kể cả khi job fail"
```

| Keyword | Khi nào chạy | Job fail có chạy không |
|---|---|---|
| `before_script` | Trước `script` | Không (nếu before_script fail thì job fail luôn) |
| `script` | Lệnh chính | — |
| `after_script` | Sau `script` | **Có, luôn chạy** (dùng để dọn dẹp, gửi thông báo) |

Ghi đè trong job riêng:

```yaml
special-job:
  before_script:
    - echo "Ghi đè before_script toàn cục"
  script:
    - echo "job chính"
```

### 4.5. `variables` — biến môi trường

```yaml
variables:
  NODE_ENV: production
  APP_VERSION: "1.2.3"

build:
  script:
    - echo "Deploy version $APP_VERSION lên $NODE_ENV"
```

- Biến toàn cục: áp dụng cho mọi job.
- Biến trong job: ghi đè biến toàn cục.
- Biến có thể dùng trong `script`, `image:`, `rules:`, v.v.

### 4.6. `default` — cấu hình chung

```yaml
default:
  image: alpine:3.19
  before_script:
    - apk add --no-cache curl
  retry: 2

job-a:
  script: [echo "dùng image alpine, retry 2 lần"]
job-b:
  image: node:20   # ghi đè default
  script: [echo "dùng image node"]
```

### 4.7. Các keyword quan trọng khác (tổng quan)

| Keyword | Tác dụng |
|---|---|
| `extends` | Kế thừa cấu hình từ job ẩn (tái sử dụng) |
| `include` | Nhúng file YAML khác (local, remote, template) |
| `workflow` | Điều khiển khi nào pipeline được tạo |
| `rules` | Điều kiện chạy/bỏ job (thay thế `only/except`) |
| `needs` | Phụ thuộc DAG, chạy sớm không cần chờ cả stage |
| `dependencies` | Chọn artifacts lấy từ jobs nào |
| `artifacts` | Lưu file output của job |
| `cache` | Cache dependencies giữa các lần chạy |
| `environment` | Gắn job với môi trường deploy |
| `services` | Chạy thêm container phụ (database, redis...) |
| `tags` | Chỉ chạy trên runner có tag tương ứng |
| `allow_failure` | Job fail vẫn cho pipeline tiếp tục |
| `retry` | Tự chạy lại khi fail (số lần / điều kiện) |
| `timeout` | Giới hạn thời gian job (mặc định 60 phút) |
| `when` | `on_success` / `on_failure` / `manual` / `always` / `delayed` |
| `parallel` | Chia job thành nhiều instance chạy song song (matrix) |
| `resource_group` | Khóa tài nguyên — deploy production không chạy chồng nhau |

### 4.8. `extends` và job ẩn (tái sử dụng cấu hình)

Job bắt đầu bằng `.` là **job ẩn** — không chạy, chỉ để kế thừa:

```yaml
.base-test:
  image: python:3.12
  before_script:
    - pip install -r requirements.txt

test-unit:
  extends: .base-test
  script: [pytest tests/unit]

test-integration:
  extends: .base-test
  script: [pytest tests/integration]
```

### 4.9. `include` — chia nhỏ file cấu hình

```yaml
include:
  - local: 'ci/build.yml'        # file trong cùng repo
  - local: 'ci/test.yml'
  - remote: 'https://example.com/ci/deploy.yml'  # URL ngoài
  - project: 'my-group/ci-templates'             # repo khác
    file: '/templates/docker.yml'
  - template: Security/SAST.gitlab-ci.yml        # template có sẵn của GitLab
```

### 4.10. `services` — container phụ (database...)

```yaml
test-db:
  image: python:3.12
  services:
    - name: postgres:16
      alias: db
  variables:
    POSTGRES_DB: testdb
    POSTGRES_USER: runner
    POSTGRES_PASSWORD: secret
    DATABASE_URL: "postgresql://runner:secret@db:5432/testdb"
  script:
    - pytest
```

Job kết nối tới Postgres qua hostname `db`.

### 4.11. `parallel: matrix` — chạy ma trận

```yaml
test-matrix:
  parallel:
    matrix:
      - PYTHON: ["3.10", "3.11", "3.12"]
        OS: [linux]
  image: python:$PYTHON
  script:
    - pytest
```

Tạo 3 jobs song song, mỗi job một version Python. Biến `$PYTHON` có sẵn trong job.

---

## 5. Triggers: cách kích hoạt pipeline

### 5.1. Push (mặc định)

Mọi `git push` lên branch đều tạo pipeline (trừ khi `workflow:rules` chặn).

### 5.2. Merge Request pipelines

```yaml
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH   # push branch thường
```

Pipeline chạy **trên kết quả merge** (merged result) chứ không phải trên branch nguồn — test đúng code sẽ merge vào.

### 5.3. Schedule (lịch định kỳ)

**Build → Pipeline schedules → New schedule**: đặt cron (ví dụ `0 2 * * *` chạy 2h sáng), chọn branch, thêm variables.

Dùng cho: nightly build, test định kỳ, dọn dẹp, backup.

```yaml
nightly-cleanup:
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule"
  script:
    - ./cleanup-old-images.sh
```

### 5.4. Manual (bấm tay)

```yaml
deploy-prod:
  stage: deploy
  script: [./deploy.sh prod]
  when: manual          # hiện nút "Play" để bấm chạy
  allow_failure: false
```

Kết hợp `environment: production` để cần quyền deploy.

### 5.5. API trigger (trigger token)

1. **Settings → CI/CD → Pipeline triggers → Add trigger** → lấy token.
2. Gọi API để kích hoạt:

```bash
curl -X POST \
  -F token=TOKEN_CUA_BAN \
  -F ref=main \
  -F "variables[DEPLOY_ENV]=staging" \
  https://gitlab.com/api/v4/projects/PROJECT_ID/trigger/pipeline
```

Trong YAML nhận biết:

```yaml
deploy-external:
  rules:
    - if: $CI_PIPELINE_SOURCE == "trigger"
  script: [echo "Được trigger từ bên ngoài, env=$DEPLOY_ENV"]
```

### 5.6. Bảng `$CI_PIPELINE_SOURCE`

| Giá trị | Pipeline tạo bởi |
|---|---|
| `push` | git push |
| `merge_request_event` | Mở/cập nhật MR |
| `schedule` | Lịch định kỳ |
| `manual` (web) | Bấm "Run pipeline" trên web |
| `trigger` | API trigger token |
| `api` | Gọi API trực tiếp |
| `webide` | Commit từ Web IDE |

---

## 6. Runners: shared, specific, cài đặt và đăng ký

### 6.1. Các loại runner

| Loại | Ai quản lý | Dùng cho |
|---|---|---|
| **Shared runners** | GitLab.com | Mọi project — dùng ngay không cần cài, free tier giới hạn minutes |
| **Group runners** | Admin group | Mọi project trong group |
| **Project runners (specific)** | Bạn | Chỉ project của bạn — máy riêng, không tốn minutes của GitLab |

### 6.2. Runner hoạt động thế nào?

1. Runner **đăng ký** với GitLab instance (gitlab.com hoặc self-hosted) bằng token.
2. Runner **poll** (hỏi liên tục) GitLab: "có job nào cho tôi không?"
3. Có job phù hợp (đúng tags) → runner nhận, chạy, gửi log về.

### 6.3. Cài đặt gitlab-runner trên Linux (step-by-step)

```bash
# 1. Thêm repo chính thức của GitLab
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash

# 2. Cài đặt
sudo apt-get install gitlab-runner

# 3. Kiểm tra
gitlab-runner --version
```

### 6.4. Cài đặt bằng Docker

```bash
# 1. Tạo volume lưu config
docker volume create gitlab-runner-config

# 2. Chạy runner
docker run -d --name gitlab-runner --restart always \
  -v gitlab-runner-config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

> Mount `docker.sock` để runner có thể spawn container Docker (executor `docker`).

### 6.5. Đăng ký runner với token

Lấy token: **Settings → CI/CD → Runners → New project runner** (GitLab 16+ dùng *registration token* theo flow mới; bản cũ: **Settings → CI/CD → Runners → registration token**).

```bash
sudo gitlab-runner register \
  --url https://gitlab.com \
  --token glrt-xxxxxxxxxxxxxxxx \
  --executor docker \
  --docker-image alpine:3.19 \
  --description "docker-runner-vps1" \
  --tag-list "docker,linux" \
  --run-untagged="true" \
  --locked="false"
```

Với Docker (container runner đã chạy ở trên):

```bash
docker exec -it gitlab-runner gitlab-runner register \
  --url https://gitlab.com \
  --token glrt-xxxxxxxxxxxxxxxx \
  --executor docker \
  --docker-image alpine:3.19 \
  --description "docker-runner" \
  --tag-list "docker,linux" \
  --non-interactive
```

Giải thích các flag:

| Flag | Ý nghĩa |
|---|---|
| `--executor docker` | Mỗi job chạy trong container Docker riêng |
| `--docker-image` | Image mặc định khi job không khai báo `image:` |
| `--tag-list` | Tags — job phải khai báo `tags:` trùng mới chạy trên runner này |
| `--run-untagged` | Có nhận job không khai báo tags không |
| `--locked` | `false` = runner dùng được cho project khác (nếu là shared/group) |

Các executor khác: `shell` (chạy trực tiếp trên máy host), `ssh`, `virtualbox`, `kubernetes`, `docker-autoscaler`.

### 6.6. Dùng tags để chọn runner

```yaml
build-on-my-vps:
  tags: [docker, linux]   # chỉ runner có đủ 2 tags này mới nhận job
  script: [echo "Chạy trên VPS của tôi"]
```

### 6.7. Quản lý runner

```bash
# Xem danh sách runner đã đăng ký
sudo gitlab-runner list

# Chạy runner dạng service (tự khởi động cùng máy)
sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
sudo gitlab-runner start

# Gỡ đăng ký một runner
sudo gitlab-runner unregister --name "docker-runner-vps1"
```

Trên GitLab web: **Settings → CI/CD → Runners** → bật/tắt, pause, xóa runner.

---

## 7. Variables: biến môi trường trong CI

### 7.1. Thứ tự ưu tiên (cao → thấp)

1. **Trigger variables / schedule variables / manual run variables** (truyền lúc chạy)
2. **Project variables** (Settings → CI/CD → Variables)
3. **Group variables**
4. **Variables trong `.gitlab-ci.yml`** (job > global)
5. **Predefined variables** của GitLab

### 7.2. Project / Group variables

**Settings → CI/CD → Variables → Add variable**:

| Thuộc tính | Ý nghĩa |
|---|---|
| **Key / Value** | Tên và giá trị biến |
| **Protected** | Chỉ expose cho pipeline chạy trên branch/tag protected (main...) — dùng cho secret production |
| **Masked** | Che giá trị trong log (phải thỏa: ≥8 ký tự, không chứa khoảng trắng... theo quy tắc GitLab) |
| **Environment scope** | Chỉ có hiệu lực với environment nhất định (`*`, `production`, `staging`) |

> **Best practice**: secret (API key, password, token) luôn bật **Masked** + **Protected**. Không bao giờ hardcode secret vào `.gitlab-ci.yml`.

### 7.3. Predefined variables (biến có sẵn)

GitLab tự inject hàng chục biến. Hay dùng nhất:

| Biến | Giá trị |
|---|---|
| `$CI` | `true` — nhận biết đang chạy trong CI |
| `$CI_PROJECT_PATH` | `username/project` |
| `$CI_PROJECT_ID` | ID số của project |
| `$CI_COMMIT_SHA` | SHA đầy đủ của commit |
| `$CI_COMMIT_SHORT_SHA` | 8 ký tự đầu của SHA |
| `$CI_COMMIT_BRANCH` | Tên branch (pipeline branch) |
| `$CI_COMMIT_TAG` | Tên tag (pipeline tag) |
| `$CI_COMMIT_MESSAGE` | Commit message |
| `$CI_PIPELINE_ID` / `$CI_PIPELINE_IID` | ID pipeline |
| `$CI_PIPELINE_SOURCE` | Nguồn trigger (push, schedule, trigger...) |
| `$CI_JOB_NAME` / `$CI_JOB_ID` | Tên/ID job hiện tại |
| `$CI_JOB_STAGE` | Stage của job |
| `$CI_REGISTRY` | `registry.gitlab.com` |
| `$CI_REGISTRY_IMAGE` | `registry.gitlab.com/username/project` |
| `$CI_REGISTRY_USER` / `$CI_REGISTRY_PASSWORD` | Credential push image (job token) |
| `$CI_MERGE_REQUEST_IID` | IID của MR (chỉ trong MR pipeline) |
| `$CI_ENVIRONMENT_NAME` | Tên environment job deploy tới |
| `$GITLAB_USER_LOGIN` | Username người trigger |

Ví dụ dùng:

```yaml
build:
  script:
    - echo "Build commit $CI_COMMIT_SHORT_SHA trên branch $CI_COMMIT_BRANCH"
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
```

### 7.4. Truyền biến lúc chạy tay

**Build → Pipelines → Run pipeline** → điền **Variables** → chạy. Hoặc qua API trigger (`variables[KEY]=value`).

---

## 8. Artifacts & Cache

### 8.1. Vấn đề

Mỗi job chạy trên runner/container **riêng biệt** → file job A tạo ra, job B không thấy. Hai cơ chế chia sẻ:

| | **Artifacts** | **Cache** |
|---|---|---|
| Mục đích | Chia sẻ **output** giữa jobs/stages (file build, báo cáo test) | Lưu **dependencies** để tải nhanh lần sau (`node_modules`, pip cache) |
| Phạm vi | Job → các job stage sau (hoặc job `needs`) | Giữa các lần chạy pipeline |
| Lưu ở đâu | GitLab server (tải về được) | Runner (hoặc S3 nếu cấu hình) |
| Hết hạn | Có (`expire_in`) | Không tự xóa |

### 8.2. Artifacts cơ bản

```yaml
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/            # lưu thư mục dist
    expire_in: 1 week    # mặc định 30 ngày
    name: "build-$CI_COMMIT_SHORT_SHA"

deploy:
  stage: deploy
  script:
    - ls dist/           # thấy được file từ job build
  dependencies:
    - build              # chỉ lấy artifacts của job build
```

### 8.3. Artifacts nâng cao

```yaml
test:
  script: [pytest --junitxml=report.xml]
  artifacts:
    when: always              # lưu cả khi job fail (mặc định: on_success)
    reports:
      junit: report.xml       # hiện kết quả test trên UI GitLab
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
    paths: [report.xml]
```

Các loại `reports` đặc biệt GitLab hiểu: `junit` (test), `coverage_report`, `dotenv` (xuất biến cho job sau!), `terraform`, `dependency_scanning`, `sast`...

**Truyền biến qua artifacts dotenv**:

```yaml
generate-version:
  script:
    - echo "APP_VERSION=1.2.$CI_PIPELINE_IID" > version.env
  artifacts:
    reports:
      dotenv: version.env

use-version:
  needs: [generate-version]
  script:
    - echo "Deploy $APP_VERSION"   # biến có sẵn!
```

### 8.4. Cache dependencies

```yaml
# cache toàn cục
cache:
  key:
    files: [package-lock.json]   # key đổi khi file lock đổi
  paths: [node_modules/]

install:
  script: [npm ci]

test:
  script: [npm test]   # node_modules đã có sẵn từ cache
```

Cache Python:

```yaml
cache:
  key:
    files: [requirements.txt]
  paths: [.cache/pip/]

test:
  before_script:
    - pip install --cache-dir .cache/pip -r requirements.txt
  script: [pytest]
```

> `cache:key:files` — khi `package-lock.json` thay đổi, key đổi → cache mới, tránh dùng nhầm dependencies cũ.

---

## 9. Environments: staging, production, review apps

### 9.1. Khai báo environment

```yaml
deploy-staging:
  stage: deploy
  script: [./deploy.sh staging]
  environment:
    name: staging
    url: https://staging.example.com

deploy-prod:
  stage: deploy
  script: [./deploy.sh production]
  environment:
    name: production
    url: https://example.com
  when: manual
```

Vào **Operate → Environments** thấy dashboard: environment nào, deploy lần cuối khi nào, ai deploy, nút rollback/redeploy.

### 9.2. Review Apps — môi trường tự động cho mỗi MR

Mỗi MR mở ra → tự tạo một môi trường preview riêng → reviewer bấm vào xem thử → merge xong tự xóa.

```yaml
review-app:
  stage: deploy
  script:
    - ./deploy-review.sh $CI_MERGE_REQUEST_IID
  environment:
    name: review/$CI_MERGE_REQUEST_IID
    url: https://review-$CI_MERGE_REQUEST_IID.example.com
    on_stop: stop-review-app
    auto_stop_in: 1 week
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

stop-review-app:
  stage: deploy
  script: [./teardown-review.sh $CI_MERGE_REQUEST_IID]
  environment:
    name: review/$CI_MERGE_REQUEST_IID
    action: stop
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual
```

### 9.3. Bảo vệ environment production

**Settings → CI/CD → Protected environments**: chỉ role Maintainer (hoặc user cụ thể) mới được deploy lên `production` — job manual của người không có quyền sẽ không chạy được.

Kết hợp `resource_group` để deploy production không chạy chồng:

```yaml
deploy-prod:
  resource_group: production
  environment: { name: production, url: https://example.com }
```

---

## 10. Rules / only / except: điều kiện chạy job

### 10.1. `rules` (khuyến nghị — thay thế only/except)

```yaml
job-build:
  rules:
    - if: $CI_COMMIT_BRANCH == "main"                    # chỉ chạy trên main
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"   # hoặc trên MR
      when: manual                                        # nhưng phải bấm tay
    - when: never                                         # còn lại: không chạy
```

Các clause của rule:

| Clause | Ý nghĩa |
|---|---|
| `if:` | Điều kiện (so sánh biến, regex `=~`) |
| `changes:` | Chỉ chạy khi file paths thay đổi |
| `exists:` | Chỉ chạy khi file tồn tại trong repo |
| `when:` | `on_success` / `on_failure` / `manual` / `always` / `never` / `delayed` |
| `allow_failure:` | Gắn với rule |

Ví dụ thực tế:

```yaml
# Chỉ test khi code thay đổi
test-backend:
  rules:
    - changes: [backend/**/*]
  script: [pytest backend/]

# Deploy production: chỉ tag semver, bấm tay
deploy-prod:
  rules:
    - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
      when: manual
    - when: never
  script: [./deploy.sh prod]

# Job dọn dẹp chạy khi job trước fail
notify-failure:
  rules:
    - if: $CI_PIPELINE_SOURCE == "push"
      when: on_failure
  script: [curl -X POST $SLACK_WEBHOOK -d "Build fail!"]
```

### 10.2. `only` / `except` (cũ, vẫn dùng được nhưng nên chuyển sang rules)

```yaml
job-cu:
  only:
    - main
    - tags
  except:
    - schedules
  script: [echo "chạy trên main và tags, trừ schedule"]
```

> GitLab khuyến nghị dùng `rules` — `only/except` không kết hợp được logic phức tạp và sẽ không được phát triển thêm.

### 10.3. `workflow:rules` — điều khiển cả pipeline

```yaml
workflow:
  rules:
    - if: $CI_COMMIT_MESSAGE =~ /\[skip ci\]/
      when: never                       # commit ghi [skip ci] thì không chạy pipeline
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == "main"
```

---

## 11. DAG với `needs`: chạy song song, phụ thuộc có hướng

Mặc định: stage sau chờ **tất cả** jobs stage trước xong. Với `needs`, job có thể chạy ngay khi **job cụ thể** xong — không cần chờ cả stage → pipeline nhanh hơn (Directed Acyclic Graph).

```yaml
stages: [build, test, deploy]

build-app:
  stage: build
  script: [make build]

unit-test:
  stage: test
  needs: [build-app]        # chỉ cần build-app xong là chạy
  script: [make test-unit]

lint:
  stage: test
  needs: []                 # không cần chờ gì — chạy ngay từ đầu!
  script: [make lint]

deploy:
  stage: deploy
  needs: [unit-test]        # không cần chờ lint
  script: [make deploy]
```

Sơ đồ thực thi:

```
build-app ──→ unit-test ──→ deploy
    │
lint (chạy song song ngay từ đầu, không chờ build)
```

### Artifacts với needs

Mặc định job `needs` job nào thì tự động tải artifacts của job đó. Tắt đi nếu không cần:

```yaml
job-nhanh:
  needs:
    - job: build-app
      artifacts: false     # không tải artifacts, chỉ chờ xong
  script: [echo "chỉ cần biết build xong"]
```

### `needs` xuyên stage và `optional`

```yaml
deploy:
  needs:
    - job: optional-test
      optional: true        # job này có thể không tồn tại (bị rules loại) — không lỗi
```

---

## 12. Merge trains & pipeline cho Merge Request

### 12.1. Merge Request pipelines

Đã đề cập ở mục 5.2 — pipeline chạy trên **merged result**. Bật trong **Settings → Merge requests →** chọn **Pipelines must succeed** để chặn merge khi pipeline đỏ.

### 12.2. Merge trains (đoàn tàu merge)

Vấn đề: 2 MR cùng pass pipeline riêng, nhưng merge liên tiếp có thể conflict ngầm (MR2 test trên code chưa có MR1).

Merge train giải quyết: xếp MR vào hàng đợi, GitLab tự tạo pipeline trên **kết quả merge tích lũy** của cả đoàn tàu. MR đầu fail → các MR sau tự động rebase lại.

Bật: **Settings → Merge requests → Merge method: Merge train** (cần Premium; Free dùng **Merge commit / Fast-forward** + "Pipelines must succeed").

```yaml
# Job chỉ chạy trong merge train
train-check:
  rules:
    - if: $CI_MERGE_REQUEST_EVENT_TYPE == "merge_train"
  script: [./final-check.sh]
```

---

## 13. Auto DevOps

**Auto DevOps** tự động phát hiện ngôn ngữ/framework của repo và sinh pipeline build → test → review → deploy hoàn chỉnh mà **không cần viết `.gitlab-ci.yml`**.

### Bật Auto DevOps

**Settings → CI/CD → Auto DevOps →** tick **Default to Auto DevOps pipeline**.

Nó tự làm:

| Giai đoạn | Tự động |
|---|---|
| Build | Heroku buildpacks phát hiện ngôn ngữ, build Docker image |
| Test | Chạy test của framework (nếu có) |
| Code Quality | CodeClimate |
| SAST / Dependency Scanning | Quét bảo mật |
| Review | Tạo Review App cho mỗi MR |
| Deploy | Deploy lên Kubernetes (cần gắn cluster) |

### Tùy biến

Tạo `.gitlab-ci.yml` với biến để bật/tắt từng phần:

```yaml
include:
  - template: Auto-DevOps.gitlab-ci.yml

variables:
  AUTO_DEVOPS_DOMAIN: example.com
  PRODUCTION_REPLICAS: 2
  STAGING_ENABLED: "1"
  REVIEW_DISABLED: "1"   # tắt review apps
```

> Auto DevOps tiện để demo/prototype. Project nghiêm túc nên viết pipeline riêng để kiểm soát hoàn toàn.

---

## 14. Container Registry: build & push Docker image

Mỗi project GitLab có sẵn **Container Registry** miễn phí: `registry.gitlab.com/<username>/<project>`.

### 14.1. Pipeline build & push

```yaml
stages: [build]

build-image:
  stage: build
  image: docker:24
  services:
    - docker:24-dind        # Docker-in-Docker
  variables:
    DOCKER_HOST: tcp://docker:2375
    DOCKER_TLS_CERTDIR: ""
  before_script:
    - docker login $CI_REGISTRY -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker build -t $CI_REGISTRY_IMAGE:latest .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

Giải thích:

- `docker:24-dind` — service Docker daemon riêng cho job (Docker-in-Docker).
- `$CI_REGISTRY_USER` / `$CI_REGISTRY_PASSWORD` — credential tạm GitLab cấp, có quyền push registry của chính project.
- Tag image bằng `$CI_COMMIT_SHORT_SHA` để trace được image ↔ commit.

### 14.2. Kaniko (không cần DinD, an toàn hơn)

```yaml
build-kaniko:
  stage: build
  image:
    name: gcr.io/kaniko-project/executor:debug
    entrypoint: [""]
  script:
    - /kaniko/executor
      --context $CI_PROJECT_DIR
      --dockerfile $CI_PROJECT_DIR/Dockerfile
      --destination $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
      --cache=true
```

Kaniko build image **không cần Docker daemon** — chạy được trên shared runners bị chặn DinD và trên Kubernetes.

### 14.3. Xem image đã push

**Deploy → Container Registry**: danh sách tags, size, nút xóa tag cũ. Dọn tag cũ tự động bằng **Cleanup policies** trong cùng trang.

---

## 15. Ví dụ thực tế hoàn chỉnh

### 15.1. CI Python với pytest

```yaml
stages: [test, build]

variables:
  PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"

cache:
  key:
    files: [requirements.txt]
  paths: [.cache/pip/]

lint:
  stage: test
  image: python:3.12
  before_script: [pip install ruff]
  script: [ruff check .]

pytest:
  stage: test
  image: python:3.12
  before_script:
    - pip install -r requirements.txt
    - pip install pytest pytest-cov
  script:
    - pytest --cov=src --cov-report=xml:coverage.xml --junitxml=report.xml
  artifacts:
    when: always
    reports:
      junit: report.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
  coverage: '/TOTAL.*\s+(\d+%)$/'

build-wheel:
  stage: build
  image: python:3.12
  script:
    - pip install build
    - python -m build
  artifacts:
    paths: [dist/]
    expire_in: 1 week
  needs: [pytest]
```

### 15.2. CI Node.js

```yaml
stages: [install, test, build]

variables:
  NPM_CONFIG_CACHE: "$CI_PROJECT_DIR/.npm"

cache:
  key:
    files: [package-lock.json]
  paths: [.npm/]

install-deps:
  stage: install
  image: node:20-alpine
  script: [npm ci]
  artifacts:
    paths: [node_modules/]
    expire_in: 1 hour

lint-test:
  stage: test
  image: node:20-alpine
  needs: [install-deps]
  script:
    - npm run lint
    - npm test -- --coverage

build:
  stage: build
  image: node:20-alpine
  needs: [install-deps]
  script: [npm run build]
  artifacts:
    paths: [dist/]
```

### 15.3. Build Docker image + deploy lên server qua SSH

Cần chuẩn bị:

1. Tạo SSH key pair: `ssh-keygen -t ed25519 -f deploy_key` (không passphrase).
2. Copy public key lên server: `ssh-copy-id -i deploy_key.pub user@server-ip`.
3. Trên GitLab **Settings → CI/CD → Variables** thêm:
   - `SSH_PRIVATE_KEY` = nội dung `deploy_key` (Masked ✓, Protected ✓)
   - `SERVER_IP`, `SERVER_USER`

```yaml
stages: [build, deploy]

build-image:
  stage: build
  image: docker:24
  services: [docker:24-dind]
  variables:
    DOCKER_HOST: tcp://docker:2375
    DOCKER_TLS_CERTDIR: ""
  before_script:
    - docker login $CI_REGISTRY -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

deploy-server:
  stage: deploy
  image: alpine:3.19
  before_script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh && chmod 700 ~/.ssh
    - ssh-keyscan $SERVER_IP >> ~/.ssh/known_hosts
  script:
    - |
      ssh $SERVER_USER@$SERVER_IP "
        docker login $CI_REGISTRY -u $CI_REGISTRY_USER -p $CI_JOB_TOKEN &&
        docker pull $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA &&
        docker stop myapp || true &&
        docker rm myapp || true &&
        docker run -d --name myapp --restart always -p 80:8000 \
          $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
      "
  environment:
    name: production
    url: http://$SERVER_IP
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

> Dùng `$CI_JOB_TOKEN` thay vì `$CI_REGISTRY_PASSWORD` khi login từ server — token ngắn hạn, an toàn hơn.

### 15.4. Deploy static site lên GitLab Pages

```yaml
stages: [deploy]

pages:
  stage: deploy
  image: node:20-alpine
  script:
    - npm ci
    - npm run build
    - mv dist public        # Pages yêu cầu thư mục tên "public"
  artifacts:
    paths: [public]
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

- Job **phải tên là `pages`**, artifacts **phải là thư mục `public`**.
- Sau khi chạy: **Deploy → Pages** → lấy URL `https://username.gitlab.io/project`.
- Custom domain: **Settings → Pages → New Domain** + trỏ DNS.

---

## 16. Pricing & giới hạn free tier

### 16.1. Các gói (giá tham khảo 2026, chưa VAT)

| Gói | Giá/user/tháng | Điểm chính |
|---|---|---|
| **Free** | $0 | 400 CI minutes/tháng, 5GB storage, shared runners |
| **Premium** | ~$29 | 10.000 CI minutes, merge trains, code owners, bảo mật nâng cao |
| **Ultimate** | ~$99 | 50.000 CI minutes, security dashboard full, compliance |

> Giá thay đổi theo thời gian — kiểm tra [about.gitlab.com/pricing](https://about.gitlab.com/pricing/).

### 16.2. CI minutes là gì?

- Mỗi phút job chạy trên **shared runners** của GitLab.com trừ vào quota.
- Runner tự cài (specific) → **không tốn** CI minutes.
- Job chạy trên runner Linux x86: hệ số 1. Windows/macOS hoặc runner lớn hơn: hệ số cao hơn.

### 16.3. Cách tiết kiệm CI minutes

1. Dùng `rules:changes` — chỉ chạy job khi file liên quan đổi.
2. Dùng `needs: []` + cache để job nhanh hơn.
3. Gộp jobs nhỏ, tránh spin container nhiều lần.
4. Tự host runner cho project chạy nhiều.
5. `interruptible: true` — pipeline mới tự hủy pipeline cũ đang chạy dở trên cùng branch:

```yaml
test:
  interruptible: true
  script: [pytest]
```

### 16.4. Quota khác của Free tier

| Tài nguyên | Free tier |
|---|---|
| CI/CD minutes | 400/tháng |
| Storage (repo + registry + artifacts) | 5GB/project |
| Shared runner concurrency | Giới hạn theo namespace |
| GitLab Pages | Có, miễn phí |
| Container Registry | Có, miễn phí |

---

## 17. Troubleshooting: 12 lỗi thường gặp

### Lỗi 1: Pipeline không chạy sau khi push

**Nguyên nhân**: thiếu file `.gitlab-ci.yml` ở root, YAML sai cú pháp, hoặc `workflow:rules` chặn.
**Sửa**: vào **Build → Pipelines** xem có pipeline không; dùng **CI Lint** (**Build → Pipeline editor → Validate**) để kiểm tra YAML.

### Lỗi 2: `jobs:build-job:script: should be...` — lỗi cú pháp YAML

**Nguyên nhân**: sai indent, thiếu dấu `-`, dùng tab thay vì space.
**Sửa**: YAML **cấm tab**, indent 2 spaces. Validate bằng Pipeline editor trước khi commit.

### Lỗi 3: Job stuck ở trạng thái `pending` mãi

**Nguyên nhân**: không có runner nào nhận — sai `tags`, runner offline, hết CI minutes, hoặc project chưa bật shared runners (**Settings → CI/CD → Runners → Enable shared runners**).
**Sửa**: kiểm tra tags của job khớp tags runner; xem runner status; kiểm tra quota minutes.

### Lỗi 4: `ERROR: Job failed: exit code 1` không rõ lý do

**Nguyên nhân**: một lệnh trong `script` fail.
**Sửa**: đọc log từ dưới lên — dòng đỏ cuối cùng thường là nguyên nhân. Thêm `set -x` hoặc `echo` debug. Dùng `allow_failure: true` tạm để pipeline chạy tiếp khi debug.

### Lỗi 5: File job trước tạo ra, job sau không thấy

**Nguyên nhân**: jobs chạy trên container riêng — không chia sẻ filesystem.
**Sửa**: dùng `artifacts:paths` ở job tạo file (xem mục 8).

### Lỗi 6: `docker: command not found` trong job

**Nguyên nhân**: image của job không có Docker client.
**Sửa**: dùng `image: docker:24` + service `docker:24-dind` (mục 14.1), hoặc build bằng Kaniko.

### Lỗi 7: Không push được image — `denied: access forbidden`

**Nguyên nhân**: sai credential hoặc job token hết quyền.
**Sửa**: login đúng cú pháp `docker login $CI_REGISTRY -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD`; kiểm tra **Settings → CI/CD → Token Access** cho phép CI job token.

### Lỗi 8: Biến Masked không có tác dụng / lộ secret trong log

**Nguyên nhân**: giá trị masked phải ≥8 ký tự và đáp ứng quy tắc của GitLab; `echo $SECRET` vẫn in ra nhưng GitLab che bằng `[MASKED]`.
**Sửa**: đặt lại biến đủ dài, bật Masked; không bao giờ `echo` secret ở dạng decode (base64...).

### Lỗi 9: Cache không hoạt động — `node_modules` vẫn cài lại từ đầu

**Nguyên nhân**: sai `key`, sai `paths`, hoặc job dùng image khác nhau không share cache.
**Sửa**: dùng `key:files` trỏ đúng lock file; `paths` đúng thư mục; các job share cache nên dùng chung key.

### Lỗi 10: `rules` khiến job bị skip ngoài ý muốn

**Nguyên nhân**: quên rule fallback `- when: never` / `- when: on_success`, hoặc biến `if:` sai tên.
**Sửa**: dùng **Pipeline editor → View merged YAML** và **Simulate pipeline** để xem rules đánh giá thế nào; in biến ra bằng job debug `script: [env | sort]`.

### Lỗi 11: SSH deploy fail — `Host key verification failed`

**Nguyên nhân**: thiếu `known_hosts`.
**Sửa**: thêm `ssh-keyscan $SERVER_IP >> ~/.ssh/known_hosts` trong `before_script` (như ví dụ 15.3). Hoặc dùng `ssh -o StrictHostKeyChecking=accept-new`.

### Lỗi 12: Pipeline chạy quá lâu / timeout

**Nguyên nhân**: mặc định job timeout 60 phút; project timeout mặc định.
**Sửa**: tăng `timeout: 2h` cho job nặng; tối ưu bằng cache, `needs`, image alpine nhỏ gọn; tách job lớn thành nhiều job song song.

---

## 18. Best practices checklist

### Cấu trúc & bảo trì

- [ ] Chia file CI thành nhiều file nhỏ bằng `include:local` (build.yml, test.yml, deploy.yml).
- [ ] Dùng job ẩn (`.base`) + `extends` để tránh lặp cấu hình.
- [ ] Validate YAML bằng **Pipeline editor → CI Lint** trước khi merge.
- [ ] Ghim version image (`python:3.12` thay vì `python:latest`) để build ổn định, tái lập được.

### Tốc độ

- [ ] Dùng `cache` cho dependencies với `key:files` trỏ lock file.
- [ ] Dùng `needs` (DAG) thay vì chờ cả stage khi có thể.
- [ ] Dùng `rules:changes` để skip job không liên quan.
- [ ] Dùng image alpine/slim nhỏ gọn; `interruptible: true` cho branch pipeline.
- [ ] Dùng `artifacts:expire_in` ngắn cho artifacts trung gian (tránh đầy storage).

### Bảo mật

- [ ] Secret để trong **Settings → CI/CD → Variables** (Masked + Protected), không hardcode vào YAML.
- [ ] Bật **Protected environments** cho production; deploy production dùng `when: manual`.
- [ ] Dùng `$CI_JOB_TOKEN` thay vì Personal Access Token trong script khi có thể.
- [ ] Hạn chế **Token Access** (CI_JOB_TOKEN scope) chỉ cho project cần thiết.
- [ ] Quét image dependencies định kỳ (Dependency Scanning / Container Scanning).

### Quy trình

- [ ] Bật **Pipelines must succeed** trước khi cho merge MR.
- [ ] Deploy production có `resource_group` chống chạy chồng.
- [ ] Tag Docker image bằng `$CI_COMMIT_SHORT_SHA` (traceability).
- [ ] Có job `notify` với `when: on_failure` báo về Slack/Telegram khi pipeline đỏ.
- [ ] Review Apps cho MR của web app để reviewer xem trước.

---

## 19. FAQ

**Hỏi: GitLab CI/CD khác gì GitHub Actions?**
Đáp: Về khái niệm tương đương (pipeline/workflow, job, runner). GitLab CI tích hợp sâu hơn (environments, review apps, merge trains có sẵn), cấu hình tập trung một file `.gitlab-ci.yml`. GitHub Actions mạnh ở marketplace actions khổng lồ. Chọn theo nơi đặt repo của bạn.

**Hỏi: Dùng GitLab.com có mất phí không?**
Đáp: Không — Free tier đủ cho cá nhân/small team: 400 CI minutes/tháng, shared runners, registry, Pages miễn phí. Hết minutes thì tự host runner là không tốn nữa.

**Hỏi: Runner tự host có an toàn không?**
Đáp: Runner chỉ pull job từ GitLab (outbound), không mở port inbound. Đặt runner trong mạng nội bộ/VPS riêng, dùng executor `docker` để cô lập job. Không chạy runner shared cho code không tin cậy.

**Hỏi: Làm sao chạy pipeline chỉ khi có tag release?**
Đáp:
```yaml
release:
  rules:
    - if: $CI_COMMIT_TAG
  script: [./release.sh $CI_COMMIT_TAG]
```

**Hỏi: Muốn bỏ qua pipeline cho commit sửa README thì sao?**
Đáp: Ghi `[skip ci]` hoặc `[ci skip]` vào commit message — kết hợp `workflow:rules` ở mục 10.3.

**Hỏi: Một job muốn dùng output (biến) của job khác?**
Đáp: Dùng `artifacts:reports:dotenv` (mục 8.3) — job trước ghi file `.env`, job sau (có `needs`) đọc được biến.

**Hỏi: Chạy job theo lịch cron mỗi đêm?**
Đáp: **Build → Pipeline schedules** tạo schedule + rule `if: $CI_PIPELINE_SOURCE == "schedule"` (mục 5.3).

**Hỏi: Giới hạn số job chạy song song?**
Đáp: `resource_group` để serialize (mục 9.3); concurrency của shared runners do GitLab quản lý; runner tự host chỉnh `concurrent = N` trong `/etc/gitlab-runner/config.toml`.

**Hỏi: Xem log job cũ đã bị xóa?**
Đáp: Log job lưu theo thời gian giữ artifacts/log của GitLab (mặc định khá lâu trên gitlab.com). Không khôi phục được nếu đã bị xóa — quan trọng thì forward log ra hệ thống ngoài.

**Hỏi: Self-hosted GitLab có dùng được hết tính năng không?**
Đáp: Có — bản Community (miễn phí) đã gồm pipeline, runners, registry, environments, pages. Một số tính năng (merge trains, protected environments nâng cao) cần Premium/Ultimate license.

---

## 20. Cheat sheet

### Vòng đời nhanh

```bash
# 1. Viết .gitlab-ci.yml ở root repo
# 2. git add .gitlab-ci.yml && git commit -m "Add CI" && git push
# 3. Vào Build → Pipelines xem kết quả
```

### Khung YAML tối thiểu nhớ trong đầu

```yaml
stages: [build, test, deploy]

job-name:
  stage: test
  image: alpine:3.19
  variables: { KEY: value }
  before_script: [echo "trước"]
  script: [echo "chính"]
  after_script: [echo "sau, luôn chạy"]
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  needs: [job-khac]
  artifacts: { paths: [dist/], expire_in: 1 week }
  cache: { key: { files: [lock] }, paths: [deps/] }
  tags: [docker]
  when: manual
  allow_failure: true
  retry: 2
  timeout: 30m
  environment: { name: production, url: https://... }
```

### Biến hay dùng

```
$CI_COMMIT_SHORT_SHA   $CI_COMMIT_BRANCH   $CI_COMMIT_TAG
$CI_PIPELINE_SOURCE     $CI_MERGE_REQUEST_IID
$CI_REGISTRY_IMAGE     $CI_REGISTRY_USER / $CI_REGISTRY_PASSWORD
$CI_JOB_TOKEN          $GITLAB_USER_LOGIN
```

### Rules mẫu copy-paste

```yaml
# Chỉ main
rules: [{ if: $CI_COMMIT_BRANCH == "main" }]
# Chỉ MR
rules: [{ if: $CI_PIPELINE_SOURCE == "merge_request_event" }]
# Chỉ tag semver, bấm tay
rules:
  - if: $CI_COMMIT_TAG =~ /^v\d+\.\d+\.\d+$/
    when: manual
  - when: never
# Chỉ khi file đổi
rules: [{ changes: [src/**/*] }]
# Chỉ schedule
rules: [{ if: $CI_PIPELINE_SOURCE == "schedule" }]
# Bỏ qua khi commit có [skip ci]
workflow:
  rules:
    - if: $CI_COMMIT_MESSAGE =~ /\[skip ci\]/
      when: never
    - when: always
```

### Lệnh runner hay dùng

```bash
gitlab-runner register --url https://gitlab.com --executor docker
gitlab-runner list
sudo gitlab-runner start | stop | restart
# Docker:
docker exec -it gitlab-runner gitlab-runner register --non-interactive \
  --url https://gitlab.com --token TOKEN --executor docker \
  --docker-image alpine:3.19 --tag-list "docker,linux"
```

### Trigger pipeline bằng API

```bash
curl -X POST -F token=TRIGGER_TOKEN -F ref=main \
  -F "variables[ENV]=staging" \
  https://gitlab.com/api/v4/projects/PROJECT_ID/trigger/pipeline
```

### Debug nhanh

```yaml
debug-env:
  script: [env | sort]   # in tất cả biến để kiểm tra
  rules: [{ when: manual }]
```

---

*Hết guide. Chúc bạn build pipeline xanh mướt!* 🚀
