# Hướng Dẫn GitHub Actions — Từ Cơ Bản Đến Nâng Cao

> GitHub Actions là nền tảng CI/CD tích hợp sẵn trong GitHub, cho phép tự động hoá build, test, deploy và mọi tác vụ lặp đi lặp lại ngay trong repository của bạn — không cần server CI riêng.

---

## Mục lục

1. [GitHub Actions là gì? Các khái niệm cốt lõi](#1-github-actions-là-gì-các-khái-niệm-cốt-lõi)
2. [Bắt đầu: repo và workflow đầu tiên](#2-bắt-đầu-repo-và-workflow-đầu-tiên)
3. [Cú pháp YAML đầy đủ](#3-cú-pháp-yaml-đầy-đủ)
4. [Triggers (sự kiện kích hoạt) chi tiết](#4-triggers-sự-kiện-kích-hoạt-chi-tiết)
5. [Runners: GitHub-hosted và self-hosted](#5-runners-github-hosted-và-self-hosted)
6. [Actions Marketplace](#6-actions-marketplace)
7. [Secrets & Variables](#7-secrets--variables)
8. [Caching dependencies](#8-caching-dependencies)
9. [Artifacts: chia sẻ dữ liệu giữa jobs](#9-artifacts-chia-sẻ-dữ-liệu-giữa-jobs)
10. [Matrix builds](#10-matrix-builds)
11. [Environments & deployments](#11-environments--deployments)
12. [OIDC với cloud (không cần long-lived secrets)](#12-oidc-với-cloud-không-cần-long-lived-secrets)
13. [Composite actions tự viết](#13-composite-actions-tự-viết)
14. [Reusable workflows](#14-reusable-workflows)
15. [Ví dụ thực tế hoàn chỉnh](#15-ví-dụ-thực-tế-hoàn-chỉnh)
16. [Pricing](#16-pricing)
17. [Troubleshooting: 15 lỗi thường gặp](#17-troubleshooting-15-lỗi-thường-gặp)
18. [Best practices checklist](#18-best-practices-checklist)
19. [FAQ + Cheat sheet](#19-faq--cheat-sheet)

---

## 1. GitHub Actions là gì? Các khái niệm cốt lõi

**GitHub Actions** là dịch vụ CI/CD (Continuous Integration / Continuous Deployment) do GitHub cung cấp, ra mắt năm 2019. Bạn định nghĩa quy trình tự động bằng file YAML đặt trong thư mục `.github/workflows/` của repo. Mỗi khi một sự kiện xảy ra (push code, mở pull request, theo lịch...), GitHub tự động chạy quy trình đó trên máy ảo gọi là **runner**.

### Các khái niệm cốt lõi

| Khái niệm | Giải thích | Ví dụ |
|---|---|---|
| **Workflow** | Một quy trình tự động hoàn chỉnh, định nghĩa trong 1 file YAML | `ci.yml` — chạy test mỗi lần push |
| **Event (Trigger)** | Sự kiện kích hoạt workflow | `push`, `pull_request`, `schedule` |
| **Job** | Một tập hợp các bước, chạy trên cùng một runner. Các job mặc định chạy **song song** | `build`, `test`, `deploy` |
| **Step** | Một tác vụ đơn lẻ trong job: hoặc chạy shell command, hoặc dùng một action | `run: npm test`, `uses: actions/checkout@v4` |
| **Action** | Đơn vị tái sử dụng: một đoạn code đóng gói sẵn một tác vụ | `actions/checkout`, `actions/setup-node` |
| **Runner** | Máy (ảo hoặc thật) thực thi workflow. GitHub cung cấp sẵn (hosted) hoặc bạn tự cài (self-hosted) | `ubuntu-latest` |

### Luồng hoạt động

```
Sự kiện (push/PR/lịch...) 
   → Workflow được kích hoạt
      → Job 1 ──→ Step 1 ──→ Step 2 ──→ ...
      → Job 2 ──→ Step 1 ──→ Step 2 ──→ ...
   (các job chạy song song, trừ khi có needs:)
```

### GitHub Actions khác gì Jenkins / GitLab CI / Travis?

| Tiêu chí | GitHub Actions | Jenkins | GitLab CI |
|---|---|---|---|
| Cần server riêng | Không (dùng runner của GitHub) | Có | Có thể dùng shared runner |
| Tích hợp với repo | Native, ngay trong GitHub | Cần plugin/webhook | Native trong GitLab |
| Marketplace | Rất lớn, hàng nghìn actions sẵn | Plugin phong phú | Ít component hơn |
| Free tier | 2.000 phút/tháng (free plan) | Miễn phí (tự host) | 400 phút/tháng |

---

## 2. Bắt đầu: repo và workflow đầu tiên

### Bước 1: Chuẩn bị repository

Bạn cần một repo trên GitHub (public hoặc private đều được). Trên máy local:

```bash
mkdir demo-actions
cd demo-actions
git init
echo "# Demo GitHub Actions" > README.md
git add .
git commit -m "init"
git branch -M main
git remote add origin https://github.com/<username>/demo-actions.git
git push -u origin main
```

### Bước 2: Tạo file workflow đầu tiên

Tạo thư mục `.github/workflows/` và file `hello.yml`:

```yaml
# .github/workflows/hello.yml
name: Hello World

on: [push]          # chạy mỗi khi có push

jobs:
  say-hello:
    runs-on: ubuntu-latest
    steps:
      - name: Chào thế giới
        run: echo "Xin chào từ GitHub Actions!"
```

Push lên GitHub:

```bash
git add .github/
git commit -m "Thêm workflow đầu tiên"
git push
```

### Bước 3: Xem kết quả

1. Mở repo trên GitHub → tab **Actions**.
2. Bạn sẽ thấy workflow "Hello World" đang chạy (chấm vàng) rồi chuyển xanh (thành công).
3. Bấm vào run → bấm vào job `say-hello` → bấm vào step để xem log chi tiết.

> **Mẹo:** Lần đầu bật Actions cho repo, GitHub có thể yêu cầu xác nhận ở tab Actions ("I understand my workflows, go ahead and enable them").

### Workflow "thật" đầu tiên: checkout + chạy test

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Lấy code
        uses: actions/checkout@v4

      - name: Cài Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Cài dependencies
        run: npm ci

      - name: Chạy test
        run: npm test
```

---

## 3. Cú pháp YAML đầy đủ

Một file workflow là một tài liệu YAML. Dưới đây là toàn bộ các key quan trọng.

### 3.1. Cấu trúc tổng quan

```yaml
name: Tên workflow (hiển thị trên tab Actions)

on: ...                 # trigger(s)

env: ...                # biến môi trường áp dụng cho TOÀN workflow

concurrency: ...        # kiểm soát chạy đồng thời

permissions: ...        # quyền của GITHUB_TOKEN

jobs:
  ten-job:
    name: Tên hiển thị (tùy chọn)
    runs-on: ubuntu-latest
    needs: [job-khac]
    if: ...
    strategy: ...
    environment: production
    env: ...
    defaults: ...
    steps:
      - name: ...
        uses: ... / run: ...
```

### 3.2. `name`

```yaml
name: CI Pipeline
```

Nếu bỏ trống, GitHub dùng tên file làm tên workflow.

### 3.3. `on` — xem chi tiết ở mục 4.

### 3.4. `jobs.<job_id>`

`job_id` chỉ gồm chữ thường, số, `-`, `_`, và phải duy nhất trong workflow.

```yaml
jobs:
  build:            # job_id = "build"
    runs-on: ubuntu-latest
    steps:
      - run: echo "build"
```

### 3.5. `runs-on` — chọn runner

```yaml
runs-on: ubuntu-latest        # GitHub-hosted
runs-on: windows-latest
runs-on: macos-latest
runs-on: [self-hosted, linux] # self-hosted (xem mục 5)
```

Có thể là string hoặc array label. Với self-hosted, runner phải có **đủ mọi label** trong array mới nhận job.

### 3.6. `steps`

Mỗi step **bắt buộc** có `uses:` hoặc `run:`, không có cả hai.

```yaml
steps:
  # Step dùng action có sẵn
  - name: Checkout code
    uses: actions/checkout@v4
    with:
      fetch-depth: 0        # lấy full history (mặc định là 1)

  # Step chạy lệnh shell
  - name: In thông tin
    run: |
      echo "Branch: $GITHUB_REF_NAME"
      echo "SHA: $GITHUB_SHA"
      pwd && ls -la

  # Step chạy trên Windows PowerShell
  - name: Lệnh Windows
    shell: pwsh
    run: Get-ChildItem
```

Các key của step:

| Key | Ý nghĩa |
|---|---|
| `name` | Tên hiển thị (nên luôn đặt) |
| `uses` | Dùng action: `owner/repo@ref` hoặc `./path/to/action` |
| `run` | Lệnh shell (mặc định bash trên Linux/macOS, PowerShell trên Windows) |
| `with` | Tham số truyền vào action |
| `env` | Biến môi trường riêng cho step |
| `if` | Điều kiện chạy step |
| `continue-on-error` | `true` → step fail vẫn cho job tiếp tục |
| `timeout-minutes` | Timeout riêng cho step |
| `shell` | `bash`, `pwsh`, `python`, `sh`... |
| `working-directory` | Thư mục chạy lệnh |

### 3.7. `with` vs `env`

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20        # tham số của ACTION (do action định nghĩa)
    cache: npm

- run: npm run build
  env:
    NODE_ENV: production    # biến môi trường của SHELL
    API_URL: ${{ vars.API_URL }}
```

Quy tắc nhớ: `with` → đầu vào của action; `env` → biến môi trường cho process chạy.

### 3.8. Contexts và expressions `${{ }}`

GitHub cung cấp các **context** (đối tượng chứa dữ liệu):

| Context | Chứa gì | Ví dụ |
|---|---|---|
| `github` | Thông tin về workflow run, repo, event | `${{ github.sha }}`, `${{ github.ref_name }}` |
| `env` | Biến môi trường | `${{ env.MY_VAR }}` |
| `vars` | Repository/organization variables | `${{ vars.API_URL }}` |
| `secrets` | Secrets | `${{ secrets.DB_PASSWORD }}` |
| `job` | Thông tin job hiện tại | `${{ job.status }}` |
| `jobs` | Kết quả các job khác | `${{ jobs.build.outputs.version }}` |
| `steps` | Output của các step | `${{ steps.build.outputs.image }}` |
| `runner` | Thông tin runner | `${{ runner.os }}`, `${{ runner.arch }}` |
| `matrix` | Giá trị matrix hiện tại | `${{ matrix.node-version }}` |
| `strategy` | Thông tin strategy | `${{ strategy.job-index }}` |
| `inputs` | Input của `workflow_dispatch` / `workflow_call` | `${{ inputs.environment }}` |
| `needs` | Kết quả các job phụ thuộc | `${{ needs.build.outputs.artifact }}` |

**Expressions** hỗ trợ toán tử và hàm:

```yaml
if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
run: echo ${{ contains(github.event.head_commit.message, '[skip ci]') }}
```

Hàm hay dùng: `contains()`, `startsWith()`, `endsWith()`, `format()`, `join()`, `toJSON()`, `fromJSON()`, `hashFiles()`, `success()`, `failure()`, `always()`, `cancelled()`.

> **Lưu ý:** Trong `if:` không cần `${{ }}` vẫn chạy được, nhưng viết đầy đủ cho rõ ràng.

### 3.9. `needs` — phụ thuộc giữa jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "build xong"

  test:
    needs: build          # chờ build xong mới chạy
    runs-on: ubuntu-latest
    steps:
      - run: echo "test"

  deploy:
    needs: [build, test]  # chờ cả hai
    if: ${{ needs.build.result == 'success' }}
    runs-on: ubuntu-latest
    steps:
      - run: echo "deploy"
```

### 3.10. `outputs` — truyền dữ liệu giữa jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.ver.outputs.version }}
    steps:
      - id: ver
        run: echo "version=$(node -p "require('./package.json').version")" >> "$GITHUB_OUTPUT"

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploy version ${{ needs.build.outputs.version }}"
```

> **Quan trọng:** Cách cũ `::set-output` đã bị GitHub **khai tử** (deprecated). Luôn dùng file `$GITHUB_OUTPUT`. Tương tự, set biến môi trường dùng `$GITHUB_ENV`:
>
> ```bash
> echo "MY_VAR=hello" >> "$GITHUB_ENV"
> ```

### 3.11. `defaults`, `timeout-minutes`, `continue-on-error`

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 20        # job fail nếu quá 20 phút (mặc định 360)
    continue-on-error: false
    defaults:
      run:
        shell: bash
        working-directory: ./app
    steps:
      - run: npm ci
```

### 3.12. `concurrency` — chống chạy chồng chéo

```yaml
# Hủy run cũ khi có push mới vào cùng branch (rất hữu ích cho deploy)
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

### 3.13. `permissions` — quyền của GITHUB_TOKEN

```yaml
permissions:
  contents: read      # chỉ đọc code
  packages: write     # được push image lên GHCR
  id-token: write     # cần cho OIDC (mục 12)
  pull-requests: write
```

Nguyên tắc **least privilege**: chỉ cấp quyền tối thiểu cần thiết. Có thể đặt ở cấp workflow hoặc cấp job.

---

## 4. Triggers (sự kiện kích hoạt) chi tiết

### 4.1. `push`

```yaml
on:
  push:
    branches: [main, develop]          # chỉ các branch này
    branches-ignore: [docs/**]         # trừ các branch này
    tags: ['v*']                       # hoặc chỉ tag v*
    paths: ['src/**', 'package.json'] # chỉ khi file trong paths đổi
    paths-ignore: ['docs/**', '**.md']
```

> `paths` lọc theo file thay đổi — tiết kiệm phút chạy CI rất nhiều cho monorepo.

### 4.2. `pull_request`

```yaml
on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review]
    branches: [main]
    paths: ['src/**']
```

Các `types` hay dùng:

| Type | Khi nào |
|---|---|
| `opened` | Mở PR mới |
| `synchronize` | Push thêm commit vào PR |
| `reopened` | Mở lại PR đã đóng |
| `ready_for_review` | Chuyển từ draft → ready |
| `closed` | Đóng/merge PR (dùng với `github.event.pull_request.merged == true` để chạy sau merge) |
| `labeled` / `unlabeled` | Gắn/gỡ label |

> **Cảnh báo bảo mật:** Với PR từ fork (người ngoài), dùng `pull_request` (chạy trong context của base branch, **không** có quyền write secrets) thay vì `pull_request_target` trừ khi bạn hiểu rõ mình làm gì.

### 4.3. `schedule` — chạy theo lịch (cron)

```yaml
on:
  schedule:
    - cron: '0 2 * * *'      # 2h sáng UTC mỗi ngày
    - cron: '*/15 * * * *'   # mỗi 15 phút
```

Cú pháp cron chuẩn 5 trường: `phút giờ ngày-của-tháng tháng thứ-trong-tuần`.

| Ví dụ | Nghĩa |
|---|---|
| `0 2 * * *` | 02:00 UTC hàng ngày |
| `0 9 * * 1` | 09:00 UTC mỗi thứ Hai |
| `30 1 * * 6,0` | 01:30 UTC thứ Bảy và Chủ nhật |
| `0 0 1 * *` | 00:00 UTC ngày 1 mỗi tháng |

Lưu ý:
- Giờ là **UTC**. Việt Nam (UTC+7): muốn chạy 9h sáng VN → `cron: '0 2 * * *'`.
- GitHub có thể delay vài phút đến vài chục phút khi hệ thống tải cao.
- `schedule` chỉ chạy trên **default branch** (thường là `main`).

### 4.4. `workflow_dispatch` — chạy thủ công

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Môi trường deploy'
        required: true
        default: 'staging'
        type: choice
        options: [staging, production]
      version:
        description: 'Version (để trống = latest)'
        required: false
        type: string
```

Sau đó bấm **Run workflow** trên tab Actions, chọn branch và điền input. Trong workflow dùng `${{ inputs.environment }}`.

### 4.5. `workflow_call` — gọi từ workflow khác (reusable)

```yaml
on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'
    secrets:
      NPM_TOKEN:
        required: true
    outputs:
      version:
        value: ${{ jobs.build.outputs.version }}
```

Xem chi tiết ở mục 14.

### 4.6. `release`

```yaml
on:
  release:
    types: [published]   # khi bấm Publish release
```

Dùng để build artifact/docker image gắn với release. Các types khác: `created`, `prereleased`, `released`, `edited`, `deleted`.

### 4.7. Các trigger khác hay dùng

```yaml
on:
  issues:
    types: [opened, labeled]     # khi có issue mới / gắn label
  issue_comment:
    types: [created]            # comment mới (dùng làm chatbot: "/deploy")
  fork:                         # khi repo bị fork
  watch:
    types: [started]            # khi ai đó star repo ⭐
  workflow_run:                 # khi một workflow khác chạy xong
    workflows: [CI]
    types: [completed]
  repository_dispatch:          # trigger từ API bên ngoài
    types: [deploy-event]
```

Trigger từ API ngoài bằng `repository_dispatch`:

```bash
curl -X POST \
  -H "Authorization: Bearer <PAT>" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<owner>/<repo>/dispatches \
  -d '{"event_type":"deploy-event","client_payload":{"version":"1.2.3"}}'
```

### 4.8. Lọc nâng cao bằng `if` ở cấp job

```yaml
jobs:
  deploy-prod:
    # chỉ deploy khi push tag v* lên main repo (không phải fork)
    if: startsWith(github.ref, 'refs/tags/v') && github.repository == 'myorg/myapp'
    runs-on: ubuntu-latest
    steps:
      - run: echo "deploy production"
```

---

## 5. Runners: GitHub-hosted và self-hosted

### 5.1. GitHub-hosted runners

GitHub cung cấp máy ảo miễn phí (trong giới hạn phút), reset sạch sau mỗi job.

| Label | OS | CPU / RAM | Ghi chú |
|---|---|---|---|
| `ubuntu-latest` (24.04) | Ubuntu | 4 vCPU / 16 GB | **Nên dùng mặc định** — nhanh, rẻ phút nhất |
| `ubuntu-22.04` / `ubuntu-24.04` | Ubuntu | 4 vCPU / 16 GB | Pin version khi cần ổn định |
| `windows-latest` | Windows Server 2025 | 4 vCPU / 16 GB | Tốn **2x phút** so với Linux |
| `macos-latest` (15) | macOS | 4 vCPU / 14 GB (ARM) | Tốn **10x phút** so với Linux |
| `macos-13` | macOS | Intel | Cho app cần chip Intel |

Phần mềm cài sẵn: git, Docker, Node.js, Python, Java, Go, Rust, .NET, kubectl, Terraform, Chrome... (danh sách đầy đủ: repo `actions/runner-images`).

> **Mẹo tiết kiệm:** Luôn ưu tiên `ubuntu-latest`. Chỉ dùng macOS khi bắt buộc (build iOS/macOS app), Windows khi build .NET Framework.

### 5.2. Self-hosted runner — khi nào cần?

- Cần phần cứng đặc biệt (GPU, RAM lớn, ARM, máy Mac vật lý).
- Cần truy cập mạng nội bộ (deploy vào server trong LAN/VPN).
- Muốn **không giới hạn phút** (dùng máy của mình).
- Cần cache/persistent disk giữa các lần chạy.

### 5.3. Cài đặt self-hosted runner step-by-step (Linux)

**Bước 1:** Trên GitHub, vào repo → **Settings → Actions → Runners → New self-hosted runner** → chọn Linux x64. Bạn sẽ được lệnh mẫu có token.

**Bước 2:** Trên máy Linux của bạn:

```bash
# Tạo thư mục và tải runner
mkdir -p ~/actions-runner && cd ~/actions-runner
curl -o actions-runner-linux-x64-2.321.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.321.0/actions-runner-linux-x64-2.321.0.tar.gz
tar xzf actions-runner-linux-x64-2.321.0.tar.gz

# Cài đặt (thay URL và TOKEN bằng lệnh GitHub đưa)
./config.sh --url https://github.com/<owner>/<repo> --token <TOKEN> \
  --name "my-runner-01" --labels linux,self-hosted,docker --unattended

# Cài dependencies
sudo ./bin/installdependencies.sh
```

**Bước 3:** Chạy runner:

```bash
# Chạy thử (foreground)
./run.sh

# Chạy như service (khuyến nghị cho server)
sudo ./svc.sh install
sudo ./svc.sh start
sudo ./svc.sh status
```

**Bước 4:** Dùng trong workflow:

```yaml
jobs:
  build:
    runs-on: [self-hosted, linux]
    steps:
      - uses: actions/checkout@v4
      - run: docker build -t myapp .
```

### 5.4. Runner groups & scale

- **Organization/Enterprise runner:** đăng ký runner ở cấp org để nhiều repo dùng chung (Settings của organization → Actions → Runners).
- **Autoscaling:** dùng [actions-runner-controller](https://github.com/actions/actions-runner-controller) trên Kubernetes, hoặc dùng dịch vụ như Ubicloud để tự scale runner theo nhu cầu.
- **Ephemeral runner:** `--ephemeral` khi config → runner tự hủy sau 1 job (an toàn hơn cho repo public).

### 5.5. Bảo mật self-hosted runner

> ⚠️ **Không dùng self-hosted runner cho repo PUBLIC** nếu workflow chạy code từ PR của người lạ — họ có thể chạy mã độc và chiếm quyền máy bạn. Nếu bắt buộc, dùng runner ephemeral + mạng cô lập.

---

## 6. Actions Marketplace

[Marketplace](https://github.com/marketplace?type=actions) có hàng nghìn actions. Quy tắc vàng: **luôn pin version** (`@v4`, tốt hơn là `@<sha-đầy-đủ>`) để tránh bị đổi code bất ngờ.

### 6.1. `actions/checkout` — lấy code

```yaml
- uses: actions/checkout@v4
  with:
    ref: 'develop'        # checkout branch/tag/SHA khác
    fetch-depth: 0        # full history (cần cho git log, tags)
    submodules: recursive # nếu dùng git submodules
    token: ${{ secrets.PAT }}  # nếu cần quyền cao hơn GITHUB_TOKEN
```

### 6.2. `actions/setup-node` — cài Node.js

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: npm             # tự động cache node_modules (npm/yarn/pnpm)
    registry-url: https://npm.pkg.github.com  # cho GitHub Packages
```

### 6.3. `actions/setup-python` — cài Python

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: pip
    cache-dependency-path: '**/requirements*.txt'
```

Tương tự có `actions/setup-go`, `actions/setup-java`, `rust-lang/rustup`, `docker/setup-buildx-action`.

### 6.4. `actions/cache` — cache thủ công

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-npm-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-npm-
```

Xem chi tiết ở mục 8.

### 6.5. `actions/upload-artifact` / `download-artifact`

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
    retention-days: 7
```

Xem chi tiết ở mục 9.

### 6.6. Các actions phổ biến khác

| Action | Dùng để |
|---|---|
| `docker/login-action` | Đăng nhập Docker Hub / GHCR |
| `docker/build-push-action` | Build & push Docker image (hỗ trợ cache, multi-arch) |
| `aws-actions/configure-aws-credentials` | Lấy credentials AWS (hỗ trợ OIDC) |
| `google-github-actions/auth` | Auth GCP (hỗ trợ OIDC) |
| `azure/login` | Đăng nhập Azure (hỗ trợ OIDC) |
| `appleboy/ssh-action` | SSH vào server chạy lệnh (deploy) |
| `appleboy/scp-action` | Copy file qua SCP |
| `peter-evans/create-pull-request` | Tự tạo PR (vd: auto-update dependencies) |
| `codecov/codecov-action` | Upload coverage report |
| `github/codeql-action` | Quét bảo mật CodeQL |

### 6.7. Đánh giá action trước khi dùng

1. Xem **số star**, lần update gần nhất, tác giả (ưu tiên actions chính chủ `actions/*`).
2. Đọc source — action chạy code của người khác với quyền trong repo bạn.
3. Pin bằng **SHA commit đầy đủ** cho workflow quan trọng:
   ```yaml
   - uses: actions/checkout@b4ffde65f46336ab88a
...[truncated 29099 chars]
---

## 11. Environments & deployments

**Environment** = một "môi trường" deploy (staging, production) với quy tắc bảo vệ riêng.

### 11.1. Tạo environment

Repo → **Settings → Environments → New environment** → đặt tên `production`. Trong workflow:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production   # gắn job với environment
    steps:
      - run: ./deploy.sh
```

### 11.2. Protection rules

Trong settings của environment, bạn có thể bật:

| Quy tắc | Ý nghĩa |
|---|---|
| **Required reviewers** | Cần N người bấm Approve mới được deploy (chống deploy nhầm production) |
| **Wait timer** | Chờ X phút trước khi deploy (cửa sổ hủy) |
| **Deployment branches** | Chỉ cho deploy từ branch/tag nhất định (vd: chỉ `main` được deploy production) |

Ví dụ workflow với reviewer:

```yaml
jobs:
  deploy-prod:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com   # hiện nút "View deployment" trên UI
    steps:
      - run: ./deploy-prod.sh
```

Khi job tới bước này, GitHub **dừng lại chờ duyệt** — người có quyền vào tab Actions bấm **Review deployments → Approve**.

### 11.3. Environment secrets/variables

Mỗi environment có secrets & variables riêng, **ghi đè** repo-level khi trùng tên:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - run: deploy --api-key "${{ secrets.API_KEY }}" --url "${{ vars.APP_URL }}"
```

`secrets.API_KEY` ở đây lấy từ environment `staging`, không phải repo secrets. Rất tiện cho multi-env deploy.

---

## 12. OIDC với cloud (không cần long-lived secrets)

Truyền thống: lưu `AWS_SECRET_ACCESS_KEY` trong GitHub Secrets — key này sống lâu, rò rỉ là nguy hiểm. **OIDC** (OpenID Connect) cho phép GitHub Actions **đổi token ngắn hạn** lấy credentials cloud, không lưu secret dài hạn nào.

### Cách hoạt động

```
GitHub Actions ──(JWT có chữ ký GitHub)──▶ AWS / GCP / Azure
                    "Tôi là repo X, workflow Y"
                        │
                        ▼ kiểm tra chữ ký + điều kiện (repo/branch)
                 ◀── credentials tạm thời (15-60 phút) ──
```

### 12.1. AWS

**Bước 1:** Trên AWS tạo IAM Role với trust policy tin tưởng GitHub OIDC provider:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com" },
    "Action": "sts:AssumeRoleWithWebIdentity",
    "Condition": {
      "StringEquals": { "token.actions.githubusercontent.com:aud": "sts.amazonaws.com" },
      "StringLike": { "token.actions.githubusercontent.com:sub": "repo:myorg/myapp:*" }
    }
  }]
}
```

**Bước 2:** Trong workflow:

```yaml
permissions:
  id-token: write   # BẮT BUỘC để lấy OIDC token
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-role
          aws-region: ap-southeast-1

      - run: aws s3 sync dist/ s3://my-bucket/ --delete
```

> Điều kiện `sub: repo:myorg/myapp:*` giới hạn chỉ repo đó được assume role — có thể siết thêm `repo:myorg/myapp:ref:refs/heads/main` chỉ cho branch main.

### 12.2. GCP

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: google-github-actions/auth@v2
        with:
          workload_identity_provider: 'projects/123456/locations/global/workloadIdentityPools/github-pool/providers/github-provider'
          service_account: 'deployer@my-project.iam.gserviceaccount.com'

      - uses: google-github-actions/deploy-cloudrun@v2
        with:
          service: my-service
          image: gcr.io/my-project/myapp:${{ github.sha }}
          region: asia-southeast1
```

### 12.3. Azure

```yaml
permissions:
  id-token: write
  contents: read

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
      - run: az webapp deploy --name myapp --resource-group myrg --src-path app.zip
```

> Với Azure, 3 ID trên **không phải secret** (là định danh công khai) — có thể để trong `vars`.

---

## 13. Composite actions tự viết

**Composite action** = gom nhiều step thành một action tái sử dụng, viết bằng YAML (không cần Docker/JS).

### Ví dụ: action "setup + test" dùng chung

Tạo repo (hoặc thư mục trong repo hiện tại) với file `action.yml`:

```yaml
# .github/actions/setup-and-test/action.yml
name: Setup and Test
description: Cài Node, install deps, chạy test

inputs:
  node-version:
    description: 'Node version'
    required: false
    default: '20'
  run-lint:
    description: 'Có chạy lint không'
    required: false
    default: 'true'

outputs:
  test-result:
    description: 'Kết quả test'
    value: ${{ steps.test.outputs.result }}

runs:
  using: composite
  steps:
    - uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: npm

    - run: npm ci
      shell: bash

    - if: ${{ inputs.run-lint == 'true' }}
      run: npm run lint
      shell: bash

    - id: test
      run: |
        npm test
        echo "result=passed" >> "$GITHUB_OUTPUT"
      shell: bash
```

> ⚠️ Trong composite action, mỗi `run:` **bắt buộc** có `shell:`.

### Sử dụng

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ./.github/actions/setup-and-test   # action local
        with:
          node-version: 20
```

Hoặc từ repo khác: `uses: myorg/shared-actions/setup-and-test@v1`.

### Khi nào dùng composite vs JS/Docker action?

| Loại | Khi nào dùng |
|---|---|
| **Composite** | Chỉ gồm các step `run`/`uses` có sẵn — đủ cho 90% nhu cầu nội bộ |
| **JavaScript action** | Cần logic phức tạp, gọi API, xử lý JSON... (chạy nhanh, cross-platform) |
| **Docker action** | Cần môi trường đặc biệt, tool hệ thống lạ (chỉ chạy trên Linux) |

---

## 14. Reusable workflows

**Reusable workflow** = cả file workflow gọi được từ workflow khác qua `workflow_call`. Khác composite ở chỗ: nó chạy như **job độc lập** (có `runs-on` riêng), phù hợp cho cả pipeline dùng chung.

### Workflow dùng chung: `.github/workflows/reusable-ci.yml`

```yaml
name: Reusable CI

on:
  workflow_call:
    inputs:
      node-version:
        required: false
        type: string
        default: '20'
      working-directory:
        required: false
        type: string
        default: '.'
    secrets:
      NPM_TOKEN:
        required: false
    outputs:
      version:
        description: 'App version'
        value: ${{ jobs.ci.outputs.version }}

jobs:
  ci:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.ver.outputs.version }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: npm
          cache-dependency-path: ${{ inputs.working-directory }}/package-lock.json

      - run: npm ci && npm run lint && npm test
        working-directory: ${{ inputs.working-directory }}
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}

      - id: ver
        run: echo "version=$(node -p "require('./package.json').version")" >> "$GITHUB_OUTPUT"
        working-directory: ${{ inputs.working-directory }}
```

### Workflow gọi nó

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]

jobs:
  call-ci:
    uses: ./.github/workflows/reusable-ci.yml   # local
    with:
      node-version: 20
    secrets:
      NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

  deploy:
    needs: call-ci
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploy version ${{ needs.call-ci.outputs.version }}"
```

Gọi từ repo khác: `uses: myorg/shared-workflows/.github/workflows/reusable-ci.yml@v1`.

### So sánh nhanh

| | Composite action | Reusable workflow |
|---|---|---|
| Đơn vị tái sử dụng | Các **steps** | Cả **workflow** (jobs) |
| Chạy ở đâu | Trong job hiện tại | Job riêng, runner riêng |
| Secrets | Kế thừa tự động | Phải truyền显式 qua `secrets:` |
| Dùng khi | Gom bước lặp lại trong job | Chuẩn hóa cả pipeline cho nhiều repo |

---

## 15. Ví dụ thực tế hoàn chỉnh

### 15.1. CI cho Node.js: test + lint + build

```yaml
# .github/workflows/node-ci.yml
name: Node.js CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  ci:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check (nếu dùng TS)
        run: npm run typecheck --if-present

      - name: Test với coverage
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          fail_ci_if_error: false

      - name: Build
        run: npm run build

      - name: Upload build artifact
        uses: actions/upload-artifact@v4
        with:
          name: dist-${{ github.sha }}
          path: dist/
          retention-days: 7
```

### 15.2. Deploy lên server qua SSH

Chuẩn bị secrets: `SSH_HOST`, `SSH_USER`, `SSH_KEY` (private key), `SSH_PORT` (tùy chọn).

```yaml
# .github/workflows/deploy-ssh.yml
name: Deploy via SSH

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci && npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/
          retention-days: 1

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist

      - name: Copy files lên server
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT || 22 }}
          source: "dist/*"
          target: "/var/www/myapp/"
          strip_components: 1

      - name: Restart service
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          port: ${{ secrets.SSH_PORT || 22 }}
          script: |
            cd /var/www/myapp
            pm2 reload ecosystem.config.js --update-env || pm2 start ecosystem.config.js
            pm2 save
```

> **An toàn hơn:** thay vì lưu private key trong secrets, dùng **Tailscale** hoặc **deploy key chỉ đọc** + webhook pull trên server.

### 15.3. Docker build + push lên GHCR

```yaml
# .github/workflows/docker.yml
name: Docker Build & Push

on:
  push:
    branches: [main]
    tags: ['v*']

permissions:
  contents: read
  packages: write   # để push lên GHCR

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Tạo tags
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=branch
            type=ref,event=tag
            type=sha,prefix=
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build và push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64
```

Image sẽ có tại `ghcr.io/<owner>/<repo>:latest`. Muốn public image: vào package settings trên GitHub → đổi visibility.

---

## 16. Pricing

### Free tier (tính đến 2026, có thể thay đổi — kiểm tra trang pricing của GitHub)

| Plan | Phút Linux/tháng | Storage (Packages/Artifacts) |
|---|---|---|
| **Free** (personal) | 2.000 | 500 MB |
| **Pro** (personal) | 3.000 | 1 GB |
| **Team** | 3.000 | 2 GB |
| **Enterprise** | 50.000 | 50 GB |

### Hệ số nhân phút

| Runner | 1 phút thực tế = |
|---|---|
| Linux (2-core) | **1** phút |
| Windows | **2** phút |
| macOS | **10** phút |
| Linux lớn hơn (4/8/16 core) | 2 / 4 / 8 phút |

Ví dụ: chạy 10 phút trên `macos-latest` = trừ **100 phút** quota.

### Vượt quota thì sao?

- Hết phút free → workflow **không chạy** (hoặc bị disable) cho đến tháng sau, trừ khi bạn bật **spending limit** và trả thêm (pay-as-you-go, khoảng $0.008/phút Linux).
- Kiểm tra mức dùng: repo/org → **Settings → Billing → Usage**.

### Mẹo tiết kiệm phút

1. Ưu tiên `ubuntu-latest`.
2. Dùng `paths:` để bỏ qua CI khi chỉ sửa docs.
3. `concurrency.cancel-in-progress: true` để hủy run cũ.
4. Cache dependencies (mục 8) — giảm thời gian `npm ci` từ phút xuống giây.
5. Gộp nhiều lệnh vào ít step hơn không tiết kiệm phút (tính theo job), nhưng giảm overhead khởi tạo step.
6. Dùng self-hosted runner cho job nặng/chạy thường xuyên.

---

## 17. Troubleshooting: 15 lỗi thường gặp

### Lỗi 1: Workflow không chạy / không hiện trong tab Actions

**Nguyên nhân:** file YAML sai vị trí, sai tên branch trigger, hoặc Actions bị tắt.
**Khắc phục:**
- File phải nằm đúng `.github/workflows/*.yml` (hoặc `.yaml`) trên **default branch**.
- Kiểm tra tab Actions có banner "Workflows aren't being run" → bấm enable.
- Với `pull_request` từ fork: kiểm tra Settings → Actions → General → "Fork pull request workflows".

### Lỗi 2: `Error: Unable to process file command 'output' successfully`

**Nguyên nhân:** dùng cú pháp `::set-output` / `::set-env` / `::save-state` đã bị khai tử.
**Khắc phục:** chuyển sang environment files:

```bash
# Sai (cũ)
echo "::set-output name=version::1.2.3"
# Đúng (mới)
echo "version=1.2.3" >> "$GITHUB_OUTPUT"
echo "MY_VAR=hello" >> "$GITHUB_ENV"
```

### Lỗi 3: `The workflow is not valid. ... Unexpected value 'xxx'`

**Nguyên nhân:** sai cú pháp YAML (thụt lề, key không tồn tại).
**Khắc phục:** dùng [YAML validator](https://www.yamllint.com/) hoặc extension "GitHub Actions" cho VS Code. Lỗi thường gặp: `uses` và `run` cùng tồn tại trong một step.

### Lỗi 4: Secrets không có giá trị (empty) trong workflow

**Nguyên nhân:**
- Secret tạo ở repo A nhưng workflow chạy ở repo B / environment khác.
- Với PR từ fork: secrets **không** được truyền vào `pull_request` runs (bảo mật).
- Gõ sai tên: `${{ secrets.MYSECRET }}` vs `MY_SECRET` (phân biệt hoa thường).

**Khắc phục:** kiểm tra Settings → Secrets and variables → Actions; debug bằng cách in độ dài (không in giá trị!):

```bash
echo "secret length: ${#SECRET_VAR}"   # 0 = secret trống/không tồn tại
```
```yaml
env:
  SECRET_VAR: ${{ secrets.MY_SECRET }}
```

### Lỗi 5: `Permission denied (publickey)` khi SSH deploy

**Nguyên nhân:** private key trong secret sai format hoặc user sai.
**Khắc phục:**
- Secret phải chứa **nguyên private key** gồm dòng `-----BEGIN OPENSSH PRIVATE KEY-----`.
- Không có passphrase trên key dùng cho CI.
- Test local trước: `ssh -i key user@host`.

### Lỗi 6: `npm ci` fail: `lock file does not match package.json`

**Nguyên nhân:** `package-lock.json` không đồng bộ (ai đó sửa `package.json` mà không chạy `npm install`).
**Khắc phục:** chạy `npm install` local rồi commit lock file. Trên CI luôn dùng `npm ci` (nghiêm ngặt) thay vì `npm install`.

### Lỗi 7: Cache không bao giờ hit (`Cache not found`)

**Nguyên nhân:** `key` thay đổi mỗi lần (vd: chứa `github.sha`), hoặc job chạy trên OS khác.
**Khắc phục:** key phải ổn định: `${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}`. Nhớ: cache chỉ hit khi **key trùng chính xác**.

### Lỗi 8: Job bị treo rồi timeout sau 6 giờ

**Nguyên nhân:** process không thoát (server dev chạy foreground, `npm start` không có `&`).
**Khắc phục:** đặt `timeout-minutes` nhỏ cho job; với lệnh chạy nền dùng `nohup ... &` hoặc chạy service thật.

### Lỗi 9: `Resource not accessible by integration` (403)

**Nguyên nhân:** `GITHUB_TOKEN` thiếu quyền.
**Khắc phục:** thêm `permissions:` phù hợp (vd: tạo PR cần `pull-requests: write`; push tag cần `contents: write`). Kiểm tra Settings → Actions → General → "Workflow permissions".

### Lỗi 10: Docker build fail: `failed to solve: permission denied`

**Nguyên nhân:** trên GitHub-hosted runner hiếm; thường do Dockerfile `USER` không có quyền ghi, hoặc buildx cache lỗi.
**Khắc phục:** thêm `cache-from/cache-to: type=gha`; nếu nghi cache hỏng, đổi key cache hoặc tắt cache để test.

### Lỗi 11: `No space left on device`

**Nguyên nhân:** runner hết disk (image Docker lớn, nhiều artifacts).
**Khắc phục:** dọn dẹp đầu job:

```yaml
- name: Dọn disk
  run: |
    df -h
    sudo rm -rf /usr/share/dotnet /opt/ghc /usr/local/share/boost
    sudo apt-get clean
    docker system prune -af
    df -h
```

### Lỗi 12: Matrix tạo ra quá nhiều job (nổ quota)

**Nguyên nhân:** matrix 3 chiều × 4 giá trị = 64 jobs.
**Khắc phục:** dùng `exclude:` loại bớt, hoặc `fail-fast: false` + giảm chiều. Cân nhắc chỉ matrix trên PR, còn push main test 1 version.

### Lỗi 13: `workflow_dispatch` inputs luôn rỗng khi trigger từ API

**Nguyên nhân:** gọi API sai endpoint. `workflow_dispatch` dùng endpoint riêng, không phải `dispatches`.
**Khắc phục:**

```bash
curl -X POST \
  -H "Authorization: Bearer <PAT>" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/repos/<owner>/<repo>/actions/workflows/<workflow-file>/dispatches \
  -d '{"ref":"main","inputs":{"environment":"staging"}}'
```

### Lỗi 14: Cron schedule không chạy

**Nguyên nhân:** workflow chưa từng chạy thành công trên default branch; file schedule nằm trên branch khác; repo bị disable Actions do không hoạt động 60 ngày (repo public).
**Khắc phục:** push workflow lên `main`, chạy tay một lần bằng `workflow_dispatch` để "kích hoạt".

### Lỗi 15: Artifacts rỗng / download sai path

**Nguyên nhân:** `path:` không khớp (glob sai, thư mục chưa được tạo vì build fail trước đó).
**Khắc phục:** luôn `ls -la` trước khi upload để xác nhận file tồn tại; dùng `if-no-files-found: error` để fail rõ ràng:

```yaml
- uses: actions/upload-artifact@v4
  with:
    name: dist
    path: dist/
    if-no-files-found: error
```

---

## 18. Best practices checklist

### Tổ chức workflow

- [ ] Mỗi workflow làm **một việc** (CI riêng, deploy riêng, release riêng).
- [ ] Đặt `name` rõ ràng cho workflow, job và **mọi step**.
- [ ] Dùng `concurrency.cancel-in-progress` cho CI chạy theo push.
- [ ] Đặt `timeout-minutes` cho mọi job (tránh treo 6 giờ).

### Bảo mật

- [ ] Pin action version (`@v4` tối thiểu; `@sha` cho workflow quan trọng).
- [ ] Đặt `permissions:` tối thiểu (least privilege) ở cấp workflow/job.
- [ ] Không bao giờ `echo` secret ra log; GitHub tự che `***` nhưng đừng ỷ lại.
- [ ] Dùng OIDC thay vì long-lived cloud keys (mục 12).
- [ ] Với repo public: không dùng self-hosted runner persistent cho PR từ fork.
- [ ] Cẩn thận với `pull_request_target` — chỉ dùng khi thật sự cần secrets cho PR fork (vd: label bot), và không checkout code PR một cách mù quáng.

### Hiệu năng & chi phí

- [ ] Cache dependencies (npm/pip/maven...).
- [ ] Dùng `paths:` / `paths-ignore:` để tránh CI không cần thiết.
- [ ] Ưu tiên `ubuntu-latest`; tránh macOS/Windows trừ khi bắt buộc.
- [ ] Gộp jobs khi thứ tự không quan trọng để giảm overhead (mỗi job mất ~10-30s khởi tạo runner).

### Độ tin cậy

- [ ] `npm ci` thay vì `npm install`; commit lock file.
- [ ] Matrix test trên PR, deploy chỉ từ `main`/tag.
- [ ] Dùng environments + required reviewers cho production.
- [ ] Lưu artifacts của bản release (không chỉ build lại từ source).
- [ ] Thêm step kiểm tra sức khỏe sau deploy (health check / smoke test).

### Maintainability

- [ ] Tách logic dùng chung thành composite actions / reusable workflows.
- [ ] Version cho reusable workflows (`@v1`) và giữ backward compatibility.
- [ ] Comment giải thích những đoạn `if:` phức tạp.

---

## 19. FAQ + Cheat sheet

### FAQ

**Hỏi: GitHub Actions có miễn phí cho repo private không?**
Đáp: Có — 2.000 phút Linux/tháng cho plan Free, áp dụng cả repo private.

**Hỏi: Workflow chạy tối đa bao lâu?**
Đáp: 6 giờ/job (mặc định 360 phút). Có thể tăng lên? Không — 6 giờ là giới hạn cứng của GitHub-hosted. Self-hosted không bị giới hạn này (do bạn tự quản).

**Hỏi: Làm sao chạy workflow chỉ khi commit message chứa từ khóa?**
Đáp:
```yaml
jobs:
  deploy:
    if: contains(github.event.head_commit.message, '[deploy]')
```

**Hỏi: Làm sao bỏ qua CI cho một commit?**
Đáp: Thêm `[skip ci]`, `[ci skip]`, `[no ci]`, `[skip actions]` hoặc `[actions skip]` vào commit message. Chỉ có tác dụng với `push` và `pull_request` (không áp dụng cho `workflow_dispatch`).

**Hỏi: `GITHUB_TOKEN` là gì? Có cần tạo không?**
Đáp: Token tự động GitHub cấp cho mỗi run, hết hạn khi run kết thúc. Dùng để gọi API, checkout, push... Không cần tạo. Quyền mặc định cấu hình ở Settings → Actions → General → Workflow permissions.

**Hỏi: Hai job chạy song song, làm sao job B đợi job A?**
Đáp: Dùng `needs:` (mục 3.9).

**Hỏi: Muốn dùng output của job A trong job B?**
Đáp: Job A khai báo `outputs:`, job B đọc qua `${{ needs.jobA.outputs.ten }}` (mục 3.10).

**Hỏi: Self-hosted runner có dùng được cho nhiều repo không?**
Đáp: Có — đăng ký runner ở cấp **organization** thay vì repo.

**Hỏi: Xóa log của một workflow run?**
Đáp: Vào run → ⋯ (góc phải) → Delete workflow run. Log tự xóa sau 90 ngày (có thể chỉnh ở Settings → Actions → General).

**Hỏi: GitHub Actions có chạy được cron mỗi phút không?**
Đáp: Về lý thuyết `* * * * *` hợp lệ, nhưng GitHub **không đảm bảo** tần suất cao — thực tế schedule thưa (≥5 phút) mới ổn định, và có thể delay.

### Cheat sheet

```yaml
# ─── Trigger cơ bản ───
on:
  push:
    branches: [main]
    paths-ignore: ['docs/**', '**.md']
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'   # 9h sáng VN mỗi ngày
  workflow_dispatch:

# ─── Job chuẩn ───
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: npm }
      - run: npm ci && npm test

# ─── Biến & secrets ───
# ${{ secrets.TEN }}  ${{ vars.TEN }}  ${{ github.sha }}
# ${{ github.ref_name }}  ${{ matrix.version }}

# ─── Ghi output / env ───
# echo "key=value" >> "$GITHUB_OUTPUT"   (có id: cho step)
# echo "VAR=x" >> "$GITHUB_ENV"

# ─── Điều kiện ───
# if: github.event_name == 'push'
# if: failure()        → chạy khi job có step fail
# if: always()         → luôn chạy
# if: cancelled()      → chạy khi bị hủy

# ─── Job phụ thuộc ───
# needs: [build]
# ${{ needs.build.outputs.version }}

# ─── Hủy run cũ ───
# concurrency:
#   group: ${{ github.workflow }}-${{ github.ref }}
#   cancel-in-progress: true

# ─── Bỏ qua CI ───  [skip ci] trong commit message
```

**Bảng tra nhanh context `github`:**

| Expression | Giá trị |
|---|---|
| `${{ github.repository }}` | `owner/repo` |
| `${{ github.ref }}` | `refs/heads/main` / `refs/tags/v1.0` |
| `${{ github.ref_name }}` | `main` / `v1.0` |
| `${{ github.sha }}` | SHA commit đầy đủ |
| `${{ github.actor }}` | Người trigger workflow |
| `${{ github.event_name }}` | `push`, `pull_request`... |
| `${{ github.run_id }}` / `run_number` | ID / số thứ tự của run |

---

*Tài liệu này bao quát GitHub Actions từ cơ bản đến nâng cao. GitHub cập nhật Actions liên tục — khi gặp hành vi lạ, kiểm tra [GitHub Docs](https://docs.github.com/actions) và [changelog](https://github.blog/changelog/label/actions/) để xác nhận.*
