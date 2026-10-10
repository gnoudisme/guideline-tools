# Hướng dẫn Google Cloud Build từ A đến Z

> Tài liệu tổng hợp đầy đủ về Google Cloud Build: dịch vụ CI/CD serverless của Google Cloud — từ khái niệm cơ bản, viết file cấu hình, tạo trigger, cache, deploy cho tới tối ưu chi phí và xử lý lỗi. Tất cả ví dụ đều có thể chạy thật trên Google Cloud.

## Mục lục

1. [Google Cloud Build là gì](#1-google-cloud-build-là-gì)
2. [So sánh với GitHub Actions và Cloud Deploy](#2-so-sánh-với-github-actions-và-cloud-deploy)
3. [Bật Cloud Build API, tạo project, cài đặt gcloud CLI](#3-bật-cloud-build-api-tạo-project-cài-đặt-gcloud-cli)
4. [cloudbuild.yaml — cú pháp đầy đủ](#4-cloudbuildyaml--cú-pháp-đầy-đủ)
5. [Build steps: chạy lệnh và dùng community builders](#5-build-steps-chạy-lệnh-và-dùng-community-builders)
6. [Triggers: GitHub, Cloud Source Repos, manual, Pub/Sub, webhook](#6-triggers-github-cloud-source-repos-manual-pubsub-webhook)
7. [Substitutions: biến mặc định và biến tùy chỉnh](#7-substitutions-biến-mặc-định-và-biến-tùy-chỉnh)
8. [Caching với Kaniko (Docker layer cache)](#8-caching-với-kaniko-docker-layer-cache)
9. [Artifacts: push lên Artifact Registry, lưu build logs](#9-artifacts-push-lên-artifact-registry-lưu-build-logs)
10. [Deploy lên Cloud Run và GKE](#10-deploy-lên-cloud-run-và-gke)
11. [Private pools: build trong VPC](#11-private-pools-build-trong-vpc)
12. [IAM permissions và service account](#12-iam-permissions-và-service-account)
13. [Ví dụ thực tế](#13-ví-dụ-thực-tế)
14. [Pricing: bảng giá và free tier](#14-pricing-bảng-giá-và-free-tier)
15. [Troubleshooting: lỗi thường gặp](#15-troubleshooting-lỗi-thường-gặp)
16. [Best practices](#16-best-practices)
17. [Cheat sheet](#17-cheat-sheet)

---

## 1. Google Cloud Build là gì

**Google Cloud Build** (viết tắt: GCB) là dịch vụ **CI/CD serverless** chạy trên hạ tầng Google Cloud. Bạn chỉ cần mô tả "build làm những bước gì" trong một file cấu hình (thường là `cloudbuild.yaml`), Google sẽ tự lo việc cấp phát máy ảo, chạy từng bước, rồi giải phóng tài nguyên.

### Cách nó hoạt động

Một **build** trong Cloud Build gồm:

1. **Source** — mã nguồn lấy từ GitHub, Cloud Source Repositories, Bitbucket, GitLab, hoặc một file `.tar.gz` bạn upload lên Cloud Storage.
2. **Steps** — danh sách các bước chạy tuần tự (hoặc song song). Mỗi bước là **một container Docker** chạy một lệnh nào đó (ví dụ: `npm test`, `go build`, `docker build`).
3. **Artifacts** — kết quả sinh ra: Docker image push lên Artifact Registry, file `.jar` lưu vào Cloud Storage, v.v.
4. **Logs** — toàn bộ log build được lưu vào Cloud Logging, xem trực tiếp trên Console.

```text
push code ──▶ trigger ──▶ Cloud Build ──▶ step 1 (test) ──▶ step 2 (build image)
                                                      ──▶ step 3 (push image)
                                                      ──▶ step 4 (deploy Cloud Run)
```

### Vì sao là "serverless"

| Đặc điểm | Giải thích |
|---|---|
| Không quản lý server | Không cần dựng Jenkins server, GitLab runner hay agent nào cả |
| Tự động scale | Chạy 1 hay 100 build song song đều được (mặc định tối đa 10 build đồng thời) |
| Trả tiền theo phút | Chỉ tính số phút build thực chạy, làm tròn lên theo phút |
| Mỗi step là container | Dùng bất kỳ image Docker nào — `node`, `golang`, `python`, `ubuntu`, image của chính Google, hoặc image tự build |

### Khi nào nên dùng Cloud Build

- Dự án chạy trên Google Cloud (Cloud Run, GKE, App Engine) — tích hợp sâu với IAM, Artifact Registry, Secret Manager.
- Muốn CI/CD mà không vận hành thêm server CI nào.
- Build Docker image thường xuyên — có sẵn Docker daemon và Kaniko cache.

### Khi nào nên cân nhắc công cụ khác

- Team chỉ dùng GitHub, không dùng GCP — GitHub Actions gọn hơn.
- Cần runner macOS/Windows (build app mobile/desktop) — Cloud Build chỉ chạy Linux.
- Cần pipeline CD phức tạp (canary, phê duyệt nhiều tầng, rollback tự động) — kết hợp thêm Cloud Deploy.

---

## 2. So sánh với GitHub Actions và Cloud Deploy

### Cloud Build vs GitHub Actions

| Tiêu chí | Cloud Build | GitHub Actions |
|---|---|---|
| Nơi chạy | Hạ tầng Google Cloud (serverless) | GitHub-hosted runners hoặc self-hosted |
| File cấu hình | `cloudbuild.yaml` đặt trong repo | Workflow `.yml` trong `.github/workflows/` |
| Đơn vị chạy | **Step = container Docker** riêng lẻ | **Job = VM**, step chạy trực tiếp trên VM |
| Tích hợp GCP | Rất sâu: IAM, Secret Manager, Artifact Registry, Cloud Run | Phải cấu hình credentials (Workload Identity Federation) |
| Docker build | Docker daemon sẵn, hỗ trợ Kaniko cache | `docker/build-push-action`, cache qua registry |
| Runner OS | Chỉ Linux | Linux, Windows, macOS |
| Giá free tier | **120 build-minutes/ngày** | 2.000 phút/tháng (repo public không giới hạn) |
| Giá sau free | $0.003/phút (máy mặc định e2-medium) | $0.008/phút (Linux) |

**Kết luận nhanh:** dùng **Cloud Build** khi pipeline "sống" trên GCP (build image → push Artifact Registry → deploy Cloud Run/GKE, IAM kiểm soát chặt). Dùng **GitHub Actions** khi repo ở GitHub và muốn code + CI cùng một chỗ, hoặc cần build đa nền tảng (macOS/Windows). Hai công cụ **không loại trừ nhau**: nhiều team dùng GitHub Actions chạy test/lint, rồi gọi Cloud Build để build image và deploy lên GCP.

### Cloud Build vs Cloud Deploy

Đây là hai sản phẩm **bổ sung** cho nhau, không phải đối thủ:

| Tiêu chí | Cloud Build | Cloud Deploy |
|---|---|---|
| Vai trò | **CI**: build, test, đóng gói artifact | **CD**: đưa artifact ra các môi trường theo pipeline |
| Mô hình | Chạy các step tuần tự | Pipeline dev → staging → prod, có phê duyệt |
| Chiến lược deploy | Tự viết step `gcloud run deploy` | Canary, blue/green, rollback một nút bấm |
| Artifact đầu vào | Source code | Container image đã build (thường do Cloud Build tạo) |

**Luồng chuẩn của Google:**

```text
GitHub push ──▶ Cloud Build (test + build image + push AR)
                                    │
                                    ▼
                          Cloud Deploy pipeline
                    dev ──▶ staging ──▶ (approve) ──▶ prod (canary 25% → 100%)
```

> Tóm lại: Cloud Build trả lời câu hỏi "build cái gì, build như thế nào", Cloud Deploy trả lời "đưa lên môi trường nào, theo chiến lược gì".

---

## 3. Bật Cloud Build API, tạo project, cài đặt gcloud CLI

### 3.1. Tạo project và bật billing

Cloud Build yêu cầu project đã gắn billing account (kể cả khi chỉ dùng free tier).

```bash
# Đăng nhập, tạo project, gắn billing account
gcloud auth login
gcloud projects create du-an-demo-2026 --name="Dự án demo Cloud Build"
gcloud config set project du-an-demo-2026

gcloud billing accounts list
gcloud billing projects link du-an-demo-2026 \
  --billing-account=0X0X0X-0X0X0X-0X0X0X
```

Trên Console: https://console.cloud.google.com → chọn project → **Billing** → **Link a billing account**.

### 3.2. Bật các API cần thiết

```bash
# API chính + các API thường đi kèm
gcloud services enable cloudbuild.googleapis.com
gcloud services enable \
  artifactregistry.googleapis.com \
  run.googleapis.com \
  secretmanager.googleapis.com \
  iam.googleapis.com
```

### 3.3. Cài đặt và cấu hình gcloud CLI

Cài đặt: Ubuntu/Debian `sudo apt-get install google-cloud-cli` (thêm repo Google trước), macOS `brew install --cask google-cloud-sdk`, Windows tải installer tại `cloud.google.com/sdk/docs/install`.

```bash
# Khởi tạo: đăng nhập + chọn project + region mặc định (nên chọn gần người dùng)
gcloud init
gcloud config set run/region asia-southeast1
gcloud config set builds/region asia-southeast1

# Kiểm tra tài khoản đang dùng
gcloud auth list
gcloud config list
```

> **Mẹo:** Cloud Build mặc định chạy ở region `global`. Nếu muốn build chạy ở region cụ thể (ví dụ để gần Artifact Registry ở `asia-southeast1`), truyền `--region=asia-southeast1` khi submit build hoặc khi tạo trigger.

### 3.4. Chạy thử build đầu tiên (không cần repo)

```bash
gcloud builds submit --config=- <<'EOF'
steps:
- name: 'ubuntu:22.04'
  args: ['echo', 'Xin chào Cloud Build!']
EOF
```

Thấy `BUILD SUCCESS` là môi trường đã sẵn sàng.

---
## 4. cloudbuild.yaml — cú pháp đầy đủ

File `cloudbuild.yaml` là "kịch bản" của một build. Dưới đây là toàn bộ các trường được hỗ trợ, kèm ví dụ cho từng trường.

### 4.1. Khung tổng quát

```yaml
# cloudbuild.yaml — khung đầy đủ (xem bảng bên dưới để hiểu từng trường)
steps:
- name: 'gcr.io/cloud-builders/docker'
  id: 'build-image'
  entrypoint: 'bash'
  args: ['-c', 'docker build -t my-app .']
  dir: 'src/backend'
  env:
  - 'NODE_ENV=production'
  secretEnv: ['API_TOKEN']
  volumes:
  - name: 'cache-vol'
    path: '/cache'
  waitFor: ['-']
  timeout: '600s'

- name: 'gcr.io/cloud-builders/docker'
  id: 'push-image'
  args: ['push', 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$SHORT_SHA']
  waitFor: ['build-image']

images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$SHORT_SHA'

artifacts:
  objects:
    location: 'gs://my-build-artifacts/$BUILD_ID/'
    paths: ['dist/*.zip', 'coverage/lcov.info']

options:
  machineType: 'E2_HIGHCPU_8'
  diskSizeGb: 100
  logging: CLOUD_LOGGING_ONLY
  substitutionOption: ALLOW_LOOSE
  dynamicSubstitutions: true
  defaultLogsBucketBehavior: REGIONAL_USER_OWNED_BUCKET

availableSecrets:
  secretManager:
  - versionName: 'projects/$PROJECT_ID/secrets/api-token/versions/latest'
    env: 'API_TOKEN'

substitutions:
  _DEPLOY_ENV: 'staging'

timeout: '1200s'
tags: ['backend', 'staging']
serviceAccount: 'projects/my-project/serviceAccounts/builder@my-project.iam.gserviceaccount.com'
```

### 4.2. Bảng chi tiết từng trường

#### `steps[]` — danh sách các bước

| Trường | Bắt buộc | Mô tả |
|---|---|---|
| `name` | ✅ | Image Docker chạy step (`'ubuntu'`, `'node:20'`, `'gcr.io/cloud-builders/docker'`) |
| `args` | ✅ | Tham số truyền cho entrypoint của image |
| `entrypoint` | ❌ | Ghi đè entrypoint mặc định (ví dụ `'bash'`) |
| `id` | ❌ | Tên định danh để step khác `waitFor` |
| `waitFor` | ❌ | Id các step phải xong trước; mặc định chờ step ngay trước nó; `['-']` = chạy ngay |
| `dir` | ❌ | Thư mục làm việc (mặc định `/workspace`) |
| `env` | ❌ | Biến môi trường dạng `'KEY=value'` |
| `secretEnv` | ❌ | Biến môi trường từ Secret Manager (khai báo ở `availableSecrets`) |
| `volumes` | ❌ | Mount volume dùng chung giữa các step |
| `timeout` | ❌ | Timeout riêng cho step |

> **Lưu ý về `waitFor`:** mặc định các step chạy **tuần tự** (step sau chờ step trước). Muốn song song, đặt `waitFor: ['-']` cho các step độc lập.

```yaml
steps:
# Hai step test chạy SONG SONG
- name: 'node:20'
  id: 'test-unit'
  waitFor: ['-']
  args: ['npm', 'run', 'test:unit']
- name: 'node:20'
  id: 'test-lint'
  waitFor: ['-']
  args: ['npm', 'run', 'lint']
# Step build chờ CẢ HAI step trên xong
- name: 'node:20'
  id: 'build'
  waitFor: ['test-unit', 'test-lint']
  args: ['npm', 'run', 'build']
```

#### Các trường cấp build

| Trường | Mô tả |
|---|---|
| `images` | Image **tự động push** lên registry khi build thành công |
| `artifacts.objects` | File lưu vào Cloud Storage: `location` + `paths` (hỗ trợ glob `*`) |
| `substitutions` | Khai báo biến tùy chỉnh `$_TEN_BIEN` và giá trị mặc định |
| `options.machineType` | `E2_MEDIUM` (mặc định), `E2_HIGHCPU_8`, `E2_HIGHCPU_32`, `N1_HIGHCPU_8`, `N1_HIGHCPU_32` |
| `options.diskSizeGb` | Dung lượng đĩa (mặc định 100GB, tối đa 1000GB) |
| `options.logging` | `CLOUD_LOGGING_ONLY` (khuyến nghị), `GCS_ONLY`, `LEGACY`, `NONE` |
| `options.substitutionOption` | `MUST_MATCH` (mặc định) hoặc `ALLOW_LOOSE` |
| `options.dynamicSubstitutions` | `true` cho phép `${_VAR}` lồng nhau và `${VAR:-default}` |
| `options.defaultLogsBucketBehavior` | `REGIONAL_USER_OWNED_BUCKET` để log cùng region (tránh egress) |
| `timeout` | Timeout toàn build, tối đa `86400s` (24h); mặc định `600s` |
| `tags` | Tag để lọc build trên Console / API |
| `serviceAccount` | Service account chạy build (mặc định là Cloud Build SA của project) |
| `logsBucket` | Bucket GCS tự quản để lưu log |
| `queueTtl` | Thời gian build được chờ trong hàng đợi |
| `availableSecrets` | Khai báo secret từ Secret Manager, map vào biến môi trường |

#### Ví dụ `volumes` — chia sẻ dữ liệu giữa các step

Mỗi step là một container riêng, nhưng tất cả đều mount chung `/workspace`. Cần chia sẻ thêm thư mục (ví dụ cache `node_modules`) thì dùng `volumes`:

```yaml
steps:
- name: 'node:20'
  id: 'install'
  args: ['npm', 'ci']
  volumes:
  - name: 'nm-cache'
    path: '/usr/src/app/node_modules'

- name: 'node:20'
  id: 'build'
  waitFor: ['install']
  args: ['npm', 'run', 'build']
  volumes:
  - name: 'nm-cache'
    path: '/usr/src/app/node_modules'
```

#### Ví dụ `availableSecrets` — dùng secret an toàn

```yaml
steps:
- name: 'gcr.io/cloud-builders/gcloud'
  args:
  - 'run'
  - 'deploy'
  - 'my-service'
  - '--image=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$SHORT_SHA'
  - '--set-env-vars=API_TOKEN=$$API_TOKEN'   # $$ để escape, tránh Cloud Build substitute
  secretEnv: ['API_TOKEN']

availableSecrets:
  secretManager:
  - versionName: 'projects/$PROJECT_ID/secrets/api-token/versions/latest'
    env: 'API_TOKEN'
```

> **Quy tắc escape:** Trong `cloudbuild.yaml`, `$VAR` sẽ bị Cloud Build thay thế (substitution). Muốn giữ nguyên ký tự `$` cho shell (ví dụ `$API_TOKEN` của shell), viết `$$API_TOKEN`.

---

## 5. Build steps: chạy lệnh và dùng community builders

### 5.1. Nguyên tắc: mỗi step là một container

Khi step chạy, Cloud Build pull image `name`, mount source code vào `/workspace`, chạy `entrypoint` + `args`. Mọi thay đổi trong `/workspace` được giữ lại cho step tiếp theo.

```yaml
steps:
# Step 1: image node có sẵn npm
- name: 'node:20-slim'
  args: ['npm', 'ci']

# Step 2: ghi đè entrypoint để chạy nhiều lệnh shell
- name: 'node:20-slim'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    npm run lint
    npm run test -- --coverage
    npm run build
```

### 5.2. Cloud builders chính thức của Google

Google duy trì sẵn các image builder tại `gcr.io/cloud-builders/`:

| Builder | Image | Dùng để |
|---|---|---|
| Docker | `gcr.io/cloud-builders/docker` | `docker build/push` |
| Gcloud | `gcr.io/cloud-builders/gcloud` | Mọi lệnh `gcloud` (deploy Cloud Run, GKE...) |
| Git | `gcr.io/cloud-builders/git` | Clone repo, thao tác git |
| Gsutil | `gcr.io/cloud-builders/gsutil` | Upload/download Cloud Storage |
| Kubectl | `gcr.io/cloud-builders/kubectl` | `kubectl apply` lên GKE |
| Maven/Gradle | `gcr.io/cloud-builders/mvn`, `.../gradle` | Build Java/Kotlin |
| Go | `gcr.io/cloud-builders/go` | `go build/test` |

Ví dụ build Docker image rồi push (dùng trường `images:` thì không cần step `docker push` — Cloud Build tự push khi build thành công):

```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args:
  - 'build'
  - '-t'
  - 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'
  - '.'
- name: 'gcr.io/cloud-builders/docker'
  args:
  - 'push'
  - 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'

images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$COMMIT_SHA'
```

### 5.3. Community builders

Cộng đồng đóng góp thêm builder tại repo `GoogleCloudPlatform/cloud-builders-community`:

| Builder | Dùng để |
|---|---|
| `wget` | Tải file |
| `npm` / `yarn` | Quản lý package Node.js |
| `gcs-fetcher` | Tải artifact từ GCS về `/workspace` |
| `slack` | Gửi thông báo Slack khi build xong/thất bại |
| `terraform` / `packer` | `terraform plan/apply`, build VM image |
| `kaniko` | Build Docker image có cache (mục 8) |

Cách dùng: build image builder từ source của community rồi push vào registry của bạn, sau đó dùng như image thường (ví dụ builder `slack` để gửi thông báo khi build xong):

```bash
git clone https://github.com/GoogleCloudPlatform/cloud-builders-community.git
cd cloud-builders-community/slack
gcloud builds submit --tag=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/slack .
```

```yaml
steps:
# ... các step build/test ...
- name: 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/slack'
  args: ['--webhook-url=$$SLACK_WEBHOOK', '--message=Build $BUILD_ID thành công!']
  secretEnv: ['SLACK_WEBHOOK']

availableSecrets:
  secretManager:
  - versionName: 'projects/$PROJECT_ID/secrets/slack-webhook/versions/latest'
    env: 'SLACK_WEBHOOK'
```

### 5.4. Viết step đa lệnh gọn gàng

```yaml
steps:
# Cách 1: entrypoint bash -c (phổ biến nhất)
- name: 'ubuntu:22.04'
  entrypoint: 'bash'
  args: ['-c', 'apt-get update && apt-get install -y curl && curl --version']

# Cách 2: block scalar YAML cho script dài, hoặc gọi script trong repo
- name: 'python:3.12-slim'
  entrypoint: 'bash'
  args: ['-c', 'pip install -r requirements.txt && pytest tests/ -v && python -m build']

- name: 'bash:5'
  args: ['./scripts/ci.sh']
```

---
## 6. Triggers: GitHub, Cloud Source Repos, manual, Pub/Sub, webhook

**Trigger** là "cò súng": khi có sự kiện (push code, tạo pull request, ấn nút tay, message Pub/Sub...), Cloud Build tự động chạy một build từ `cloudbuild.yaml`.

### 6.1. Tổng quan các loại trigger

| Loại | Nguồn sự kiện | Dùng khi |
|---|---|---|
| GitHub | Push/PR trên GitHub | Phổ biến nhất |
| Bitbucket / GitLab | Push/MR | Repo ở Bitbucket, GitLab |
| Manual | Ấn nút / API / gcloud | Chạy tay (hotfix, build thử) |
| Pub/Sub | Message tới topic | Nối chuỗi pipeline |
| Webhook | HTTP POST | Tích hợp hệ thống ngoài |

### 6.2. Trigger GitHub — từng bước cấu hình

Đây là loại trigger phổ biến nhất, có 2 thế hệ: **1st gen** (cài GitHub App "Google Cloud Build" vào từng repo — đơn giản) và **2nd gen** (kết nối qua **Cloud Build Repositories**, hỗ trợ GitHub/GitLab/Bitbucket, quản lý tập trung — khuyến nghị cho team).

#### Cách A: 1st gen qua Console (nhanh nhất cho người mới)

Vào **Cloud Build → Triggers → Connect repository** → chọn **GitHub**, cài app **Google Cloud Build** cho repo (ví dụ `tieu-duongg/my-api`), rồi **Create trigger** với: **Event** = *Push to a branch*, branch `^main$`, **Configuration** = file `cloudbuild.yaml`. Push code lên `main` → build tự chạy, xem ở **Cloud Build → History**.

#### Cách B: 2nd gen bằng gcloud (khuyến nghị cho team)

```bash
# 1. Tạo host connection tới GitHub (làm 1 lần; PAT lưu trong Secret Manager trước)
gcloud builds connections create github my-github-conn \
  --region=asia-southeast1 \
  --authorizer-token-secret-version=projects/$PROJECT_ID/secrets/github-pat/versions/latest

# 2. Link repo
gcloud builds repositories create my-api \
  --remote-uri=https://github.com/tieu-duongg/my-api.git \
  --connection=my-github-conn \
  --region=asia-southeast1

# 3. Trigger chạy khi push lên main (+ trigger PR chỉ chạy test)
REPO="projects/$PROJECT_ID/locations/asia-southeast1/connections/my-github-conn/repositories/my-api"
gcloud builds triggers create github \
  --name=my-api-push-main \
  --repository=$REPO \
  --branch-pattern='^main$' \
  --build-config=cloudbuild.yaml \
  --region=asia-southeast1

gcloud builds triggers create github \
  --name=my-api-pr-check \
  --repository=$REPO \
  --pull-request-pattern='^main$' \
  --build-config=cloudbuild.ci.yaml \
  --region=asia-southeast1 \
  --substitutions=_DEPLOY_ENV=none
```

### 6.3. Trigger Cloud Source Repositories

```bash
gcloud source repos create my-api

gcloud builds triggers create cloud-source-repositories \
  --name=my-api-csr-main \
  --repo=my-api \
  --branch-pattern='^main$' \
  --build-config=cloudbuild.yaml
```

> **Lưu ý:** Google đã ngừng phát triển Cloud Source Repositories và không nhận khách hàng mới. Dự án mới nên dùng GitHub/Bitbucket/GitLab qua 2nd gen triggers.

### 6.4. Trigger manual — chạy bằng tay

```bash
# Chạy build từ source ở thư mục hiện tại (+ substitution tùy chỉnh)
gcloud builds submit --config=cloudbuild.yaml .
gcloud builds submit --config=cloudbuild.yaml \
  --substitutions=_DEPLOY_ENV=staging,_IMAGE_TAG=v1.2.3 .

# Chạy lại trigger có sẵn trên nhánh main
gcloud builds triggers run my-api-push-main --branch=main
```

Tạo trigger manual để đồng đội ấn nút trên Console (ví dụ deploy tay lên production sau khi kiểm tra staging):

```bash
gcloud builds triggers create manual \
  --name=deploy-prod-manual \
  --repo=my-api \
  --branch-pattern='^main$' \
  --build-config=cloudbuild.deploy.yaml \
  --substitutions=_DEPLOY_ENV=prod
```

### 6.5. Trigger Pub/Sub — nối chuỗi pipeline

Kịch bản: build "frontend" xong → bắn message → trigger build "e2e-test". Cuối `cloudbuild.yaml` của frontend thêm step publish message, rồi tạo trigger lắng nghe topic:

```bash
gcloud pubsub topics create build-events

gcloud builds triggers create pubsub \
  --name=e2e-after-frontend \
  --topic=projects/$PROJECT_ID/topics/build-events \
  --repo=my-api \
  --branch-pattern='^main$' \
  --build-config=cloudbuild.e2e.yaml
```

```yaml
# Step cuối trong cloudbuild.yaml của frontend: bắn message khi xong
- name: 'gcr.io/cloud-builders/gcloud'
  args: ['pubsub', 'topics', 'publish', 'build-events',
         '--message={"event":"frontend-done","sha":"$COMMIT_SHA"}']
```

### 6.6. Trigger webhook — nhận HTTP POST từ bên ngoài

Tạo secret chứa webhook key, tạo trigger, rồi gọi URL từ hệ thống ngoài:

```bash
# 1. Tạo secret chứa webhook key (chuỗi ngẫu nhiên)
openssl rand -hex 32 | gcloud secrets create webhook-key --data-file=-

# 2. Tạo trigger webhook (CMS sẽ gọi URL này khi editor ấn Publish)
gcloud builds triggers create webhook \
  --name=cms-publish-hook \
  --repo=my-api \
  --branch-pattern='^main$' \
  --build-config=cloudbuild.yaml \
  --webhook-config-secret=projects/$PROJECT_ID/secrets/webhook-key/versions/latest

# 3. Gọi webhook từ hệ thống ngoài
WEBHOOK_URL=$(gcloud builds triggers describe cms-publish-hook \
  --format='value(webhookConfig.url)' --region=asia-southeast1)
WEBHOOK_KEY=$(gcloud secrets versions access latest --secret=webhook-key)

curl -X POST "$WEBHOOK_URL" -H "Content-Type: application/json" \
  -d "{\"webhookKey\": \"$WEBHOOK_KEY\", \"branch\": \"main\"}"
```

### 6.7. Lọc trigger nâng cao

```bash
# Monorepo: chỉ chạy khi file trong backend/ thay đổi
gcloud builds triggers create github \
  --name=backend-only \
  --repository=... \
  --branch-pattern='^main$' \
  --build-config=backend/cloudbuild.yaml \
  --included-files='backend/**'

# Bỏ qua build khi chỉ sửa docs
gcloud builds triggers update backend-only \
  --ignored-files='**.md,docs/**'
```

| Tùy chọn | Ý nghĩa |
|---|---|
| `--included-files` | Chỉ chạy khi file đổi khớp pattern |
| `--ignored-files` | Không chạy khi TẤT CẢ file đổi đều khớp pattern |
| `--branch-pattern` / `--tag-pattern` | Regex khớp branch/tag (ví dụ `^release-.*$`) |
| `--pull-request-pattern` | Regex khớp branch đích PR |

---

## 7. Substitutions: biến mặc định và biến tùy chỉnh

**Substitution** là cơ chế thay thế biến trong `cloudbuild.yaml` **trước khi build chạy**. Nhờ đó một file YAML dùng được cho nhiều môi trường.

### 7.1. Biến mặc định (default substitutions)

Cloud Build tự cung cấp các biến sau, dùng ngay không cần khai báo:

| Biến | Ý nghĩa | Ví dụ giá trị |
|---|---|---|
| `$PROJECT_ID` | ID project GCP | `du-an-demo-2026` |
| `$PROJECT_NUMBER` | Số project | `123456789012` |
| `$BUILD_ID` | ID duy nhất của build | `a1b2c3d4-...` |
| `$REPO_NAME` | Tên repo | `my-api` |
| `$REPO_FULL_NAME` | `owner/repo` (GitHub) | `tieu-duongg/my-api` |
| `$BRANCH_NAME` | Tên branch | `main` |
| `$TAG_NAME` | Tên tag (nếu build từ tag) | `v1.2.3` |
| `$COMMIT_SHA` | SHA đầy đủ của commit | `9f86d08188...` |
| `$SHORT_SHA` | 7 ký tự đầu của SHA | `9f86d08` |
| `$REVISION_ID` | Revision ID (CSR) | `...` |
| `$TRIGGER_NAME` | Tên trigger đã kích hoạt build | `my-api-push-main` |
| `$LOCATION` | Region chạy build | `asia-southeast1` |

```yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args:
  - 'build'
  - '-t'
  - 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$SHORT_SHA'
  - '-t'
  - 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-app:$BRANCH_NAME'
  - '.'
```

### 7.2. Biến tùy chỉnh (custom substitutions)

Biến tự định nghĩa **phải bắt đầu bằng dấu gạch dưới**: `$_TEN_BIEN`. Có 3 nơi đặt giá trị (độ ưu tiên tăng dần): `substitutions:` trong YAML (mặc định) → `--substitutions` khi submit tay → cấu hình trigger.

```yaml
# cloudbuild.yaml
substitutions:
  _DEPLOY_ENV: 'staging'      # giá trị mặc định
  _IMAGE_TAG: 'latest'
  _RUN_TESTS: 'true'

steps:
- name: 'node:20'
  args: ['npm', 'run', 'test']
  env: ['RUN_TESTS=$_RUN_TESTS']

- name: 'gcr.io/cloud-builders/gcloud'
  args:
  - 'run'
  - 'deploy'
  - 'my-api-$_DEPLOY_ENV'
  - '--image=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$_IMAGE_TAG'
```

```bash
# Ghi đè khi chạy tay: deploy production
gcloud builds submit --config=cloudbuild.yaml \
  --substitutions=_DEPLOY_ENV=prod,_IMAGE_TAG=v1.2.3 .
```

### 7.3. Dynamic substitutions — biến lồng nhau

Mặc định, Cloud Build **không** thay thế biến nằm trong giá trị của biến khác. Bật `dynamicSubstitutions: true` để dùng `${_VAR}` lồng trong chuỗi và cú pháp bash `${VAR:-default}` (dùng `default` khi VAR rỗng), `${VAR:+alt}`:

```yaml
options:
  dynamicSubstitutions: true

substitutions:
  _ENV: 'staging'
  _SERVICE_NAME: 'my-api'

steps:
- name: 'gcr.io/cloud-builders/gcloud'
  args:
  - 'run'
  - 'deploy'
  - '${_SERVICE_NAME}-${_ENV}'     # => my-api-staging
```

### 7.4. Những bẫy thường gặp

```yaml
# ❌ SAI: $API_TOKEN bị Cloud Build substitute thành rỗng
- name: 'ubuntu'
  args: ['echo', '$API_TOKEN']

# ✅ ĐÚNG: $$ để giữ lại $ cho shell
- name: 'ubuntu'
  entrypoint: 'bash'
  args: ['-c', 'echo $$API_TOKEN']
  secretEnv: ['API_TOKEN']
```

```yaml
# ❌ SAI: biến custom thiếu gạch dưới
substitutions:
  DEPLOY_ENV: 'staging'

# ✅ ĐÚNG
substitutions:
  _DEPLOY_ENV: 'staging'
```

> **Bảo mật:** Không bao giờ đặt secret (mật khẩu, token, key) vào `substitutions` — giá trị substitution hiện trong log và Console dưới dạng plain text. Secret phải đi qua `availableSecrets` + `secretEnv` (mục 4.2).

---
## 8. Caching với Kaniko (Docker layer cache)

Mỗi build Cloud Build chạy trên máy ảo **mới tinh** — Docker layer cache của lần build trước **không** được giữ lại. Hậu quả: `docker build` lần nào cũng chạy lại từ đầu, build Node.js/Java có thể mất 10–20 phút mỗi lần.

**Kaniko** giải quyết vấn đề này: build Docker image **không cần Docker daemon**, và có thể lưu/tải layer cache từ registry.

### 8.1. So sánh các cách cache

| Cách | Khi nào dùng |
|---|---|
| Không cache | Build nhanh sẵn (< 2 phút) |
| Kaniko `--cache=true` | **Khuyến nghị** cho hầu hết Dockerfile |
| Pull image cũ làm `--cache-from` | Muốn giữ `docker build` truyền thống |
| Cache thủ công qua GCS | Cache `node_modules`, Go module, Maven repo |

### 8.2. Dùng Kaniko executor

```yaml
steps:
# Build bằng Kaniko, bật cache, push 2 tag
- name: 'gcr.io/kaniko-project/executor:latest'
  args:
  - '--destination=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
  - '--destination=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:latest'
  - '--cache=true'
  - '--cache-copy-layers=true'
  - '--dockerfile=Dockerfile'
  - '--context=dir:///workspace'

images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
```

| Flag | Ý nghĩa |
|---|---|
| `--destination` | Image đích (lặp lại flag để push nhiều tag) |
| `--cache=true` | Bật layer cache: dùng lại layer đã có từ registry |
| `--cache-copy-layers=true` | Cache cả các layer `COPY` |
| `--cache-ttl=168h` | Layer cache hết hạn sau bao lâu (mặc định 2 tuần) |
| `--context=dir:///workspace` | Context là thư mục source Cloud Build đã checkout |
| `--build-arg=KEY=value` | Truyền build-arg như `docker build` |
| `--target=stage-name` | Build multi-stage tới stage chỉ định |

> **Lưu ý:** nên **pin version cụ thể** (ví dụ `:v1.23.2`) thay vì `:latest` để build ổn định.

### 8.3. Dockerfile viết để tận dụng cache tốt nhất

Cache chỉ hiệu quả khi Dockerfile sắp xếp layer từ **ít đổi → hay đổi**:

```dockerfile
# ✅ TỐT: dependency (ít đổi) copy trước, source (hay đổi) copy sau
FROM node:20-slim AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev

FROM node:20-slim AS runner
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY dist ./dist
CMD ["node", "dist/index.js"]
```

```dockerfile
# ❌ TỆ: COPY . . trước RUN npm ci → sửa 1 dòng code là mất cache npm
FROM node:20-slim
WORKDIR /app
COPY . .
RUN npm ci
CMD ["node", "dist/index.js"]
```

### 8.4. Cache `--cache-from` với docker build truyền thống

Nếu vẫn muốn dùng `docker build` (ví dụ Dockerfile dùng BuildKit feature Kaniko chưa hỗ trợ):

```yaml
steps:
# Pull image lần trước làm cache (bỏ qua lỗi nếu chưa có image nào)
- name: 'gcr.io/cloud-builders/docker'
  entrypoint: 'bash'
  args: ['-c', 'docker pull asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:latest || true']

# Build với cache-from
- name: 'gcr.io/cloud-builders/docker'
  args:
  - 'build'
  - '--cache-from=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:latest'
  - '-t'
  - 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
  - '-t'
  - 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:latest'
  - '.'

images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:latest'
```

### 8.5. Cache dependency không phải Docker (npm, Go, Maven...)

Dùng GCS làm "kho cache" thủ công:

```yaml
steps:
# Tải cache về (bỏ qua lỗi lần đầu chưa có cache)
- name: 'gcr.io/cloud-builders/gsutil'
  entrypoint: 'bash'
  args: ['-c', 'gsutil cp gs://my-build-cache/node-modules.tar.gz /tmp/c.tar.gz && tar -xzf /tmp/c.tar.gz -C /workspace || echo "chua co cache"']

- name: 'node:20'
  entrypoint: 'bash'
  args: ['-c', 'npm ci && npm run build']

# Nén và đẩy cache mới lên GCS
- name: 'gcr.io/cloud-builders/gsutil'
  entrypoint: 'bash'
  args: ['-c', 'tar -czf /tmp/c.tar.gz -C /workspace node_modules && gsutil cp /tmp/c.tar.gz gs://my-build-cache/node-modules.tar.gz']
```

---

## 9. Artifacts: push lên Artifact Registry, lưu build logs

### 9.1. Tạo Artifact Registry repository

**Artifact Registry** là nơi lưu Docker image và package (npm, Maven, Python, Go...) — đã thay thế Container Registry (gcr.io) cũ.

```bash
# Tạo repo Docker ở region asia-southeast1
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=asia-southeast1 \
  --description="Repo Docker cho dự án demo"

# Xem danh sách image trong repo
gcloud artifacts docker images list \
  asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo
```

Địa chỉ image có dạng `<region>-docker.pkg.dev/<project-id>/<repo>/<image>:<tag>`, ví dụ:

```text
asia-southeast1-docker.pkg.dev/du-an-demo-2026/my-repo/my-api:v1.2.3
```

### 9.2. Push image từ Cloud Build (3 cách)

```yaml
# Cách 1: trường images: — tự động push khi build thành công (gọn nhất)
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$SHORT_SHA', '.']
images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$SHORT_SHA'

# Cách 2: step docker push tường minh (khi cần push nhiều tag có điều kiện)
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$SHORT_SHA', '.']
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', '--all-tags', 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api']

# Cách 3: Kaniko --destination (xem mục 8.2)
```

### 9.3. Lưu file artifact vào Cloud Storage

Dùng khi build sinh ra file: bản `.zip` frontend, báo cáo test, file `.jar`:

```yaml
steps:
- name: 'node:20'
  args: ['npm', 'run', 'build']   # sinh ra thư mục dist/

artifacts:
  objects:
    location: 'gs://my-build-artifacts/$BUILD_ID/'
    paths: ['dist/**', 'coverage.zip']
```

Sau build, file nằm ở `gs://my-build-artifacts/<build-id>/dist/...`.

### 9.4. Build logs — xem và lưu trữ

Mặc định log xem ở **Cloud Build → History → từng step**. Tùy chọn lưu log (`options.logging`): `CLOUD_LOGGING_ONLY` (**khuyến nghị** — xem nhanh, filter theo step), `GCS_ONLY`, `LEGACY`, `NONE`.

```yaml
options:
  logging: CLOUD_LOGGING_ONLY
  defaultLogsBucketBehavior: REGIONAL_USER_OWNED_BUCKET
```

```bash
# Xem log build mới nhất / stream log build đang chạy
gcloud builds log $(gcloud builds list --limit=1 --format='value(id)')
gcloud builds log $BUILD_ID --stream
```

> **Lưu ý chi phí:** log lưu trong Cloud Logging tính vào quota Cloud Logging (50GB/tháng free). Build chạy thường xuyên với log dài nên đặt retention hợp lý hoặc dùng `GCS_ONLY` với lifecycle xóa file cũ.

---

## 10. Deploy lên Cloud Run và GKE

### 10.1. Deploy lên Cloud Run — ví dụ đầy đủ

```yaml
# cloudbuild.yaml — build + push + deploy Cloud Run
substitutions:
  _SERVICE_NAME: 'my-api'
  _REGION: 'asia-southeast1'
  _AR_REPO: 'my-repo'

steps:
- name: 'gcr.io/kaniko-project/executor:v1.23.2'
  id: 'build'
  args:
  - '--destination=$_REGION-docker.pkg.dev/$PROJECT_ID/$_AR_REPO/$_SERVICE_NAME:$COMMIT_SHA'
  - '--destination=$_REGION-docker.pkg.dev/$PROJECT_ID/$_AR_REPO/$_SERVICE_NAME:latest'
  - '--cache=true'
  - '--cache-copy-layers=true'
  - '--context=dir:///workspace'

- name: 'gcr.io/cloud-builders/gcloud'
  id: 'deploy'
  waitFor: ['build']
  args:
  - 'run'
  - 'deploy'
  - '$_SERVICE_NAME'
  - '--image=$_REGION-docker.pkg.dev/$PROJECT_ID/$_AR_REPO/$_SERVICE_NAME:$COMMIT_SHA'
  - '--region=$_REGION'
  - '--allow-unauthenticated'
  - '--port=8080'
  - '--memory=512Mi'
  - '--min-instances=0'
  - '--max-instances=10'
  - '--set-env-vars=NODE_ENV=production'
  - '--set-secrets=API_TOKEN=api-token:latest'

images:
- '$_REGION-docker.pkg.dev/$PROJECT_ID/$_AR_REPO/$_SERVICE_NAME:$COMMIT_SHA'
- '$_REGION-docker.pkg.dev/$PROJECT_ID/$_AR_REPO/$_SERVICE_NAME:latest'

options:
  logging: CLOUD_LOGGING_ONLY
timeout: '1200s'
```

Giải thích các flag `gcloud run deploy`:

| Flag | Ý nghĩa |
|---|---|
| `--allow-unauthenticated` | Cho phép truy cập public (bỏ flag nếu service nội bộ) |
| `--port=8080` | Port container lắng nghe |
| `--min-instances=0` / `--max-instances=10` | Giới hạn scale (0 = scale về 0 khi không có traffic) |
| `--set-env-vars` | Biến môi trường thường |
| `--set-secrets=NAME=secret:version` | Biến môi trường lấy từ Secret Manager |

> **Quyền cần thiết:** service account chạy build cần role `roles/run.admin` và `roles/iam.serviceAccountUser` trên service account của Cloud Run service (xem mục 12).

### 10.2. Deploy nhiều môi trường (staging/prod) trong một file

Dùng biến `$_DEPLOY_ENV` (mặc định `staging` trong YAML, trigger prod override thành `prod`):

```yaml
substitutions:
  _DEPLOY_ENV: 'staging'

steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA', '.']

- name: 'gcr.io/cloud-builders/gcloud'
  args:
  - 'run'
  - 'deploy'
  - 'my-api-$_DEPLOY_ENV'     # => my-api-staging hoặc my-api-prod
  - '--image=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
  - '--region=asia-southeast1'
  - '--allow-unauthenticated'

images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
```

### 10.3. Deploy lên GKE

```yaml
# cloudbuild.yaml — build image + kubectl set image lên GKE
substitutions:
  _GKE_CLUSTER: 'my-cluster'
  _GKE_ZONE: 'asia-southeast1-a'
  _NAMESPACE: 'default'

steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA', '.']

- name: 'gcr.io/cloud-builders/gcloud'
  entrypoint: 'bash'
  args: ['-c', 'gcloud container clusters get-credentials $_GKE_CLUSTER --zone=$_GKE_ZONE --project=$PROJECT_ID']

- name: 'gcr.io/cloud-builders/kubectl'
  args: ['set', 'image', 'deployment/my-api',
          'my-api=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA',
          '-n', '$_NAMESPACE']
  env:
  - 'CLOUDSDK_COMPUTE_ZONE=$_GKE_ZONE'
  - 'CLOUDSDK_CONTAINER_CLUSTER=$_GKE_CLUSTER'

images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
```

Cách khác — render manifest bằng `envsubst` rồi `kubectl apply`:

```yaml
- name: 'gcr.io/cloud-builders/gcloud'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    gcloud container clusters get-credentials $_GKE_CLUSTER --zone=$_GKE_ZONE
    export IMAGE="asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA"
    envsubst < k8s/deployment.yaml | kubectl apply -n $_NAMESPACE -f -
```

```yaml
# k8s/deployment.yaml (trong repo, $IMAGE là placeholder được envsubst thay)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-api
spec:
  replicas: 3
  selector:
    matchLabels: {app: my-api}
  template:
    metadata:
      labels: {app: my-api}
    spec:
      containers:
      - name: my-api
        image: $IMAGE
        ports: [{containerPort: 8080}]
```

---
## 11. Private pools: build trong VPC

Mặc định, build chạy trên hạ tầng public của Google — **không** truy cập được tài nguyên trong VPC của bạn (Cloud SQL private IP, Memorystore, VM nội bộ...). **Private pool** cho phép chạy build trên worker nằm trong VPC của bạn.

### 11.1. Khi nào cần private pool

- Build cần kết nối Cloud SQL qua private IP, hoặc pull image từ registry nội bộ trong VPC.
- Chính sách bảo mật yêu cầu build không đi qua internet public.
- Cần máy build mạnh hơn mức public pool cho phép.

### 11.2. Tạo private pool — từng bước

```bash
# 1. Tạo network và subnet (nếu chưa có)
gcloud compute networks create build-vpc --subnet-mode=custom
gcloud compute networks subnets create build-subnet \
  --network=build-vpc --range=10.10.0.0/24 --region=asia-southeast1

# 2. Cấp quyền cho Cloud Build service agent dùng network
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:service-$PROJECT_NUMBER@gcp-sa-cloudbuild.iam.gserviceaccount.com" \
  --role="roles/compute.networkUser"

# 3. Tạo private pool
gcloud builds worker-pools create my-private-pool \
  --region=asia-southeast1 \
  --peered-network=projects/$PROJECT_ID/global/networks/build-vpc \
  --worker-machine-type=e2-standard-4 \
  --worker-disk-size=100
```

### 11.3. Chạy build trên private pool

Chỉ định pool trong YAML hoặc khi submit tay:

```yaml
options:
  pool:
    name: 'projects/$PROJECT_ID/locations/asia-southeast1/workerPools/my-private-pool'
```

```bash
gcloud builds submit --config=cloudbuild.yaml \
  --worker-pool=projects/$PROJECT_ID/locations/asia-southeast1/workerPools/my-private-pool .
```

### 11.4. Ví dụ: build cần kết nối Cloud SQL private IP

```yaml
# cloudbuild.yaml — chạy migration DB qua private IP (Cloud SQL)
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$SHORT_SHA', '.']

- name: 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$SHORT_SHA'
  entrypoint: 'bash'
  args: ['-c', 'npm run db:migrate']
  env:
  - 'DATABASE_URL=postgres://app-user:$$DB_PASS@10.20.0.5:5432/appdb'
  secretEnv: ['DB_PASS']

availableSecrets:
  secretManager:
  - versionName: 'projects/$PROJECT_ID/secrets/db-password/versions/latest'
    env: 'DB_PASS'

options:
  pool:
    name: 'projects/$PROJECT_ID/locations/asia-southeast1/workerPools/my-private-pool'

images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$SHORT_SHA'
```

> **Lưu ý giá:** private pool tính tiền theo worker tồn tại 24/7 (kể cả lúc không build), đắt hơn public pool — chỉ dùng khi thật sự cần vào VPC.

---

## 12. IAM permissions và service account

### 12.1. Hai service account liên quan

| Service account | Dạng | Vai trò |
|---|---|---|
| **Cloud Build SA** | `<project-number>@cloudbuild.gserviceaccount.com` | Identity mặc định chạy mọi build |
| **Service agent** | `service-<project-number>@gcp-sa-cloudbuild.iam.gserviceaccount.com` | Do Google quản lý: tạo worker pool, đọc trigger... |
| **Custom SA** | `builder@<project>.iam.gserviceaccount.com` | Tự tạo để tách quyền theo team/môi trường |

Cloud Build SA mặc định có dạng `<project-number>@cloudbuild.gserviceaccount.com`.

### 12.2. Cấp quyền tối thiểu theo tác vụ

| Tác vụ trong build | Role cần cấp cho SA chạy build |
|---|---|
| Push image lên Artifact Registry | `roles/artifactregistry.writer` |
| Deploy Cloud Run | `roles/run.admin` + `roles/iam.serviceAccountUser` |
| `kubectl` lên GKE | `roles/container.developer` |
| Đọc secret từ Secret Manager | `roles/secretmanager.secretAccessor` (trên từng secret) |
| Ghi file lên GCS | `roles/storage.objectAdmin` (trên bucket) |
| Publish Pub/Sub | `roles/pubsub.publisher` (trên topic) |
| Chạy trên private pool | `roles/cloudbuild.workerPoolUser` |

```bash
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
CB_SA="$PROJECT_NUMBER@cloudbuild.gserviceaccount.com"

# Cho phép push Artifact Registry + deploy Cloud Run
for role in artifactregistry.writer run.admin iam.serviceAccountUser; do
  gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:$CB_SA" \
    --role="roles/$role"
done
```

> **Nguyên tắc least privilege:** đừng cấp `roles/editor` hay `roles/owner` cho Cloud Build SA. Liệt kê đúng tác vụ build cần (bảng trên) và cấp từng role. Build bị hack/token lộ thì thiệt hại giới hạn trong các quyền đó.

### 12.3. Dùng custom service account cho từng môi trường

```bash
# Tạo SA riêng cho deploy production và cho phép Cloud Build SA giả danh nó
gcloud iam service-accounts create builder-prod \
  --display-name="Cloud Build - Production"

gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:builder-prod@$PROJECT_ID.iam.gserviceaccount.com" \
  --role="roles/run.admin"

gcloud iam service-accounts add-iam-policy-binding \
  builder-prod@$PROJECT_ID.iam.gserviceaccount.com \
  --member="serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com" \
  --role="roles/iam.serviceAccountTokenCreator"
```

```yaml
# cloudbuild.yaml cho production — chạy dưới SA riêng
serviceAccount: 'projects/$PROJECT_ID/serviceAccounts/builder-prod@$PROJECT_ID.iam.gserviceaccount.com'
steps:
- name: 'gcr.io/cloud-builders/gcloud'
  args: ['run', 'deploy', 'my-api-prod', '--image=...', '--region=asia-southeast1']
```

### 12.4. Quyền cho người dùng (ai được tạo/xem trigger và build)

| Role | Được làm gì |
|---|---|
| `roles/cloudbuild.builds.editor` | Tạo, chạy, hủy, xem build |
| `roles/cloudbuild.builds.viewer` | Chỉ xem build và log |
| `roles/cloudbuild.builds.approver` | Phê duyệt build đang chờ duyệt |

---

## 13. Ví dụ thực tế

### 13.1. Ví dụ 1: Node.js API — build + test + deploy Cloud Run

Repo gồm `cloudbuild.yaml`, `Dockerfile`, `package.json`, `src/index.js`.

```dockerfile
# Dockerfile — multi-stage, tối ưu cache
FROM node:20-slim AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev

FROM node:20-slim AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=deps /app/node_modules ./node_modules
COPY src ./src
EXPOSE 8080
CMD ["node", "src/index.js"]
```

```yaml
# cloudbuild.yaml
substitutions:
  _SERVICE_NAME: 'my-api'
  _REGION: 'asia-southeast1'
  _AR_REPO: 'my-repo'

steps:
# 1. Cài deps + lint + test
- name: 'node:20-slim'
  id: 'install'
  entrypoint: 'bash'
  args: ['-c', 'npm ci']

- name: 'node:20-slim'
  id: 'lint-test'
  waitFor: ['install']
  entrypoint: 'bash'
  args: ['-c', 'npm run lint && npm test -- --coverage']

# 2. Build image bằng Kaniko (có cache), chỉ chạy khi test pass
- name: 'gcr.io/kaniko-project/executor:v1.23.2'
  id: 'build-image'
  waitFor: ['lint-test']
  args:
  - '--destination=$_REGION-docker.pkg.dev/$PROJECT_ID/$_AR_REPO/$_SERVICE_NAME:$COMMIT_SHA'
  - '--destination=$_REGION-docker.pkg.dev/$PROJECT_ID/$_AR_REPO/$_SERVICE_NAME:latest'
  - '--cache=true'
  - '--cache-copy-layers=true'
  - '--context=dir:///workspace'

# 3. Deploy lên Cloud Run
- name: 'gcr.io/cloud-builders/gcloud'
  id: 'deploy'
  waitFor: ['build-image']
  args:
  - 'run'
  - 'deploy'
  - '$_SERVICE_NAME'
  - '--image=$_REGION-docker.pkg.dev/$PROJECT_ID/$_AR_REPO/$_SERVICE_NAME:$COMMIT_SHA'
  - '--region=$_REGION'
  - '--allow-unauthenticated'
  - '--port=8080'
  - '--memory=512Mi'
  - '--set-env-vars=NODE_ENV=production'

images:
- '$_REGION-docker.pkg.dev/$PROJECT_ID/$_AR_REPO/$_SERVICE_NAME:$COMMIT_SHA'

options:
  logging: CLOUD_LOGGING_ONLY

timeout: '1200s'
```

Chạy thử và tạo trigger:

```bash
gcloud builds submit --config=cloudbuild.yaml .

REPO="projects/$PROJECT_ID/locations/asia-southeast1/connections/my-github-conn/repositories/my-api"
gcloud builds triggers create github \
  --name=my-api-prod \
  --repository=$REPO \
  --branch-pattern='^main$' \
  --build-config=cloudbuild.yaml \
  --region=asia-southeast1
```

### 13.2. Ví dụ 2: Docker image → Artifact Registry (không deploy)

Kịch bản: team data build image `etl-job` mỗi đêm bằng trigger Pub/Sub kích hoạt từ Cloud Scheduler — chỉ push image, không deploy.

```yaml
# cloudbuild.yaml
steps:
- name: 'gcr.io/kaniko-project/executor:v1.23.2'
  args:
  - '--destination=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/etl-job:$SHORT_SHA'
  - '--destination=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/etl-job:latest'
  - '--cache=true'
  - '--context=dir:///workspace/etl'

images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/etl-job:$SHORT_SHA'
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/etl-job:latest'

options:
  logging: CLOUD_LOGGING_ONLY
```

```bash
# Scheduler bắn message 2h sáng giờ VN -> trigger Pub/Sub chạy build
gcloud scheduler jobs create pubsub etl-nightly \
  --schedule="0 2 * * *" \
  --topic=projects/$PROJECT_ID/topics/build-events \
  --message-body='{"job":"etl-build"}' \
  --time-zone="Asia/Ho_Chi_Minh"

gcloud builds triggers create pubsub \
  --name=etl-nightly-build \
  --topic=projects/$PROJECT_ID/topics/build-events \
  --repo=etl-repo \
  --branch-pattern='^main$' \
  --build-config=cloudbuild.yaml
```

### 13.3. Ví dụ 3: Monorepo — frontend + backend build độc lập

```text
monorepo/
├── frontend/cloudbuild.yaml   # build web tĩnh -> GCS
└── backend/cloudbuild.yaml    # build API -> Cloud Run
```

`frontend/cloudbuild.yaml` — build web tĩnh, upload lên GCS:

```yaml
steps:
- name: 'node:20-slim'
  dir: 'frontend'
  entrypoint: 'bash'
  args: ['-c', 'npm ci && npm run build']

- name: 'gcr.io/cloud-builders/gsutil'
  args: ['-m', 'rsync', '-r', '-d', 'frontend/dist', 'gs://my-web-bucket/']

artifacts:
  objects:
    location: 'gs://my-build-artifacts/$BUILD_ID/'
    paths: ['frontend/dist/**']

options:
  logging: CLOUD_LOGGING_ONLY
```

`backend/cloudbuild.yaml` — như ví dụ 13.1 nhưng `dir` trỏ vào `backend`:

```yaml
steps:
- name: 'gcr.io/kaniko-project/executor:v1.23.2'
  args:
  - '--destination=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
  - '--cache=true'
  - '--dockerfile=backend/Dockerfile'
  - '--context=dir:///workspace/backend'

- name: 'gcr.io/cloud-builders/gcloud'
  args:
  - 'run'
  - 'deploy'
  - 'my-api'
  - '--image=asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
  - '--region=asia-southeast1'
  - '--allow-unauthenticated'

images:
- 'asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:$COMMIT_SHA'
```

Hai trigger lọc theo thư mục thay đổi — sửa frontend không trigger build backend:

```bash
gcloud builds triggers create github \
  --name=monorepo-frontend --repository=... \
  --branch-pattern='^main$' \
  --build-config=frontend/cloudbuild.yaml \
  --included-files='frontend/**' \
  --region=asia-southeast1

gcloud builds triggers create github \
  --name=monorepo-backend --repository=... \
  --branch-pattern='^main$' \
  --build-config=backend/cloudbuild.yaml \
  --included-files='backend/**' \
  --region=asia-southeast1
```

---
## 14. Pricing: bảng giá và free tier

### 14.1. Free tier

| Hạng mục | Free |
|---|---|
| Build minutes | **120 build-minutes/ngày** (reset mỗi ngày, tính theo billing account) |
| Concurrent builds | Tối đa 10 build chạy đồng thời |

> **Điều kiện:** 120 phút free chỉ áp dụng cho máy mặc định (`e2-medium`); chọn máy mạnh hơn thì mọi phút đều tính tiền. Free tier chia sẻ cho tất cả project trong cùng billing account.

### 14.2. Giá sau free tier (public pool, chưa VAT)

| Machine type | Giá/phút | Phù hợp |
|---|---|---|
| `E2_MEDIUM` (mặc định) | $0.003 | Build nhỏ, test unit |
| `E2_HIGHCPU_8` | $0.016 | Build Docker/Go/Java vừa |
| `E2_HIGHCPU_32` | $0.064 | Build lớn, cần tốc độ |

Thời gian build **làm tròn lên theo phút**: chạy 1 phút 10 giây tính thành 2 phút.

**Ví dụ:** 20 build/ngày × 6 phút = 120 phút/ngày trên máy mặc định → vừa hết free tier → **$0**. Vượt 60 phút/ngày: `60 × $0.003 = $0.18/ngày ≈ $5.4/tháng`.

### 14.3. Private pool pricing

Private pool tính tiền theo worker tồn tại 24/7 (ví dụ `e2-medium` ~$0.06/giờ, `e2-standard-4` ~$0.13/giờ) — kể cả lúc không build. Build thưa mà dùng private pool có thể tốn hơn public pool nhiều.

### 14.4. Mẹo tiết kiệm chi phí

1. **Bật Kaniko cache** (mục 8): build 15 phút → 3 phút là chuyện thường.
2. **Chạy step song song** (`waitFor: ['-']`) để giảm tổng thời gian build.
3. **Đừng chọn máy mạnh quá mức cần**: test `E2_MEDIUM` trước, chỉ nâng khi timeout hoặc quá chậm.
4. **Lọc trigger** bằng `--included-files`: sửa docs không trigger build 10 phút.
5. **Đặt `timeout` hợp lý** cho build và step nặng.
6. **Dọn Artifact Registry**: cleanup policy xóa image untagged quá 7 ngày:

```bash
gcloud artifacts repositories set-cleanup-policies my-repo \
  --location=asia-southeast1 --policy=- <<'EOF'
- name: xoa-image-rac
  action: DELETE
  condition:
    tagState: UNTAGGED
    olderThan: 604800s
EOF
```

---

## 15. Troubleshooting: lỗi thường gặp

### 15.1. Build fail — đọc log đúng cách

```bash
# Liệt kê build gần nhất kèm trạng thái
gcloud builds list --limit=5 --format='table(id,status,createTime)'

# Xem log build fail / stream log build đang chạy (thay bằng ID build thật)
BUILD_ID=your-build-id
gcloud builds log $BUILD_ID
gcloud builds log $BUILD_ID --stream
```

### 15.2. Bảng lỗi thường gặp và cách sửa

| Lỗi / Triệu chứng | Nguyên nhân | Cách sửa |
|---|---|---|
| `403` khi `gcloud builds submit` | Chưa bật API hoặc thiếu quyền | `gcloud services enable cloudbuild.googleapis.com`; kiểm tra IAM |
| Trigger không chạy khi push | GitHub App chưa cài / repo chưa connect / sai branch pattern | Kiểm tra **Triggers** trên Console; test regex `^main$` |
| `denied: ... artifactregistry...uploadArtifacts` | SA thiếu quyền push AR | Cấp `roles/artifactregistry.writer` (mục 12.2) |
| `403` khi `gcloud run deploy` | Thiếu `roles/run.admin` hoặc `roles/iam.serviceAccountUser` | Cấp đủ 2 role cho SA chạy build |
| `secret "API_TOKEN" not found` | `secretEnv` dùng biến chưa khai báo | Khai báo trong `availableSecrets.secretManager` |
| `$VAR` thành chuỗi rỗng | Quên `$$` escape, hoặc biến chưa khai báo (`MUST_MATCH`) | Dùng `$$VAR` cho shell; khai báo `$_VAR` trong `substitutions:` |
| Kaniko `exit code 1` / không đọc được cache | Lỗi Dockerfile, sai `--context`, hoặc SA thiếu quyền đọc AR | Chạy `docker build` local để tái hiện; cấp `roles/artifactregistry.reader` |
| `Cannot connect to the Docker daemon` | Gọi lệnh docker bằng image thường | Step docker phải dùng `gcr.io/cloud-builders/docker` |
| `DEADLINE_EXCEEDED` | Build quá `timeout` (mặc định 600s) | Tăng `timeout`; bật Kaniko cache; nâng `machineType` |
| `Quota exceeded` | Vượt quota số build/ngày | Xin tăng quota ở **IAM & Admin → Quotas** |
| Cloud Run vẫn chạy bản cũ | Deploy tag `latest` bị cache / sai region | Deploy bằng tag `$COMMIT_SHA`; kiểm tra `--region` |
| Trigger Pub/Sub không kích hoạt | Sai tên topic | Kiểm tra tên topic (Pub/Sub topic là global) |
| Webhook trả về 403 | Sai `webhookKey` | Lấy lại: `gcloud secrets versions access latest --secret=webhook-key` |

### 15.3. Debug step bị lỗi nhanh

Thêm step debug tạm để "nhìn" vào `/workspace` khi build fail (xóa trước khi merge — đừng để `env | sort` lộ biến nhạy cảm):

```yaml
steps:
- name: 'ubuntu:22.04'
  entrypoint: 'bash'
  args:
  - '-c'
  - |
    ls -la /workspace
    test -f /workspace/package.json && echo "OK: package.json" || echo "THIEU: package.json"
```

### 15.4. Build treo không kết thúc

Hủy build: `gcloud builds cancel <BUILD_ID>`. Nguyên nhân phổ biến: lệnh chờ input (`apt-get install` thiếu `-y`), test mở server không tắt, `npm` chờ network. Khắc phục: thêm `timeout` cho từng step và flag non-interactive (`-y`, `--no-input`, `CI=true`).

---

## 16. Best practices

### Cấu trúc và cấu hình

1. **Một repo, một `cloudbuild.yaml` ở root** cho pipeline chính; monorepo thì mỗi service một file + trigger lọc `--included-files`.
2. **Tách file theo mục đích**: `cloudbuild.ci.yaml` (test cho PR, không deploy), `cloudbuild.yaml` (build + deploy staging), `cloudbuild.deploy-prod.yaml` (deploy prod, trigger manual).
3. **Pin version image builder** (`:v1.23.2` thay vì `:latest`) để build ổn định theo thời gian.
4. **Luôn đặt `timeout`** cho build và step nặng.

### Tốc độ

5. **Bật Kaniko cache** cho mọi Dockerfile (mục 8).
6. **Chạy song song** các step độc lập (`waitFor: ['-']`).
7. **Sắp xếp Dockerfile** theo nguyên tắc ít-đổi-trước, hay-đổi-sau (mục 8.3).
8. **Dùng image nhỏ**: `node:20-slim` thay vì `node:20` — pull nhanh hơn nhiều.

### Bảo mật

9. **Không hardcode secret** trong YAML — dùng `availableSecrets` + Secret Manager.
10. **Least privilege cho service account** (mục 12): đúng role theo tác vụ, không cấp `editor`; tách SA theo môi trường (staging không deploy được prod).
11. **Review `cloudbuild.yaml` như review code**: mọi sửa pipeline đều qua pull request.

### Ổn định và quan sát

12. **Tag image bằng `$COMMIT_SHA`** (giữ thêm tag `latest` cho tiện); deploy production luôn dùng SHA cụ thể để rollback chính xác.
13. **Thông báo khi build fail**: builder `slack` (mục 5.3) hoặc Pub/Sub → Cloud Monitoring alert.
14. **Lưu artifact quan trọng** vào GCS (`artifacts.objects`); **dọn dẹp định kỳ** (cleanup policy AR, lifecycle bucket log/cache).

### Mẫu pipeline chuẩn cho team nhỏ

```text
PR mở ──▶ trigger cloudbuild.ci.yaml ──▶ lint + unit test (+ comment kết quả lên PR)
                                    │
merge vào main ──▶ trigger cloudbuild.yaml ──▶ test lại ──▶ Kaniko build ──▶ push AR
                                                              ──▶ deploy staging (Cloud Run)
                                                              ──▶ smoke test
Ấn nút tay ──▶ trigger manual ──▶ deploy prod bằng đúng image SHA đã test ở staging
```

---

## 17. Cheat sheet

### Lệnh gcloud hay dùng

```bash
# Chạy build từ thư mục hiện tại (+ substitution, + region cụ thể)
gcloud builds submit --config=cloudbuild.yaml .
gcloud builds submit --config=cloudbuild.yaml --substitutions=_DEPLOY_ENV=prod .
gcloud builds submit --config=cloudbuild.yaml --region=asia-southeast1 .

# Xem / stream log / hủy build (thay BUILD_ID bằng ID build thật)
gcloud builds list --limit=10
gcloud builds log $BUILD_ID --stream
gcloud builds cancel $BUILD_ID

# Trigger
gcloud builds triggers list
gcloud builds triggers run <TRIGGER_NAME> --branch=main
gcloud builds triggers delete <TRIGGER_NAME> --region=asia-southeast1

# Worker pool
gcloud builds worker-pools list --region=asia-southeast1

# Artifact Registry
gcloud artifacts docker images list asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo
gcloud artifacts docker images delete \
  asia-southeast1-docker.pkg.dev/$PROJECT_ID/my-repo/my-api:old-tag --delete-tags
```

### Mẫu cloudbuild.yaml tối giản (copy-paste được)

```yaml
# Build Docker image + push Artifact Registry + deploy Cloud Run
substitutions:
  _SERVICE: 'my-api'
  _REGION: 'asia-southeast1'
  _REPO: 'my-repo'

steps:
- name: 'gcr.io/kaniko-project/executor:v1.23.2'
  args:
  - '--destination=$_REGION-docker.pkg.dev/$PROJECT_ID/$_REPO/$_SERVICE:$COMMIT_SHA'
  - '--cache=true'
  - '--context=dir:///workspace'

- name: 'gcr.io/cloud-builders/gcloud'
  args:
  - 'run'
  - 'deploy'
  - '$_SERVICE'
  - '--image=$_REGION-docker.pkg.dev/$PROJECT_ID/$_REPO/$_SERVICE:$COMMIT_SHA'
  - '--region=$_REGION'
  - '--allow-unauthenticated'

images:
- '$_REGION-docker.pkg.dev/$PROJECT_ID/$_REPO/$_SERVICE:$COMMIT_SHA'
```

### Biến substitution nhớ nhanh

```text
$PROJECT_ID   $BUILD_ID   $COMMIT_SHA   $SHORT_SHA
$BRANCH_NAME  $TAG_NAME   $REPO_NAME    $TRIGGER_NAME
$_TEN_BIEN    (custom — phải khai báo trong substitutions:, bắt đầu bằng _)
$$VAR         (escape: giữ nguyên $ cho shell, không substitute)
```

### Checklist trước khi merge pipeline mới

- [ ] `gcloud builds submit` chạy tay thành công từ đầu đến cuối
- [ ] Image builder đã pin version cụ thể (không `:latest`)
- [ ] Secret đi qua Secret Manager, không hardcode
- [ ] `timeout` đã đặt; trigger có `--included-files` nếu là monorepo
- [ ] SA chạy build chỉ có đúng role cần thiết
- [ ] Deploy production dùng tag `$COMMIT_SHA`, không dùng `latest`

---

> **Tài liệu tham khảo:** `cloud.google.com/build/docs` — khi có mâu thuẫn với docs chính thức, hãy theo docs chính thức.
