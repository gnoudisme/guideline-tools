# Hướng Dẫn Toàn Diện Travis CI

> Tài liệu từ A đến Z về Travis CI: từ khái niệm, cấu hình `.travis.yml`, build lifecycle, build matrix, caching, biến môi trường mã hóa, deploy tự động, đến các ví dụ thực tế và bảng cheat sheet. Dành cho developer Việt Nam muốn hiểu và làm chủ CI/CD với Travis CI.

**Mục lục**

1. [Travis CI là gì](#1-travis-ci-là-gì)
2. [So sánh Travis CI với GitHub Actions và CircleCI](#2-so-sánh-travis-ci-với-github-actions-và-circleci)
3. [Đăng ký bằng GitHub và kích hoạt repository](#3-đăng-ký-bằng-github-và-kích-hoạt-repository)
4. [`.travis.yml` — cú pháp đầy đủ](#4-travisyml--cú-pháp-đầy-đủ)
5. [Build lifecycle](#5-build-lifecycle)
6. [Hỗ trợ ngôn ngữ lập trình](#6-hỗ-trợ-ngôn-ngữ-lập-trình)
7. [Build matrix — test nhiều phiên bản song song](#7-build-matrix--test-nhiều-phiên-bản-song-song)
8. [Caching dependencies](#8-caching-dependencies)
9. [Biến môi trường mã hóa (travis encrypt)](#9-biến-môi-trường-mã-hóa-travis-encrypt)
10. [Deploy tự động (providers)](#10-deploy-tự-động-providers)
11. [Travis CLI](#11-travis-cli)
12. [Thay đổi pricing: từ free sang credits](#12-thay-đổi-pricing-từ-free-sang-credits)
13. [Ví dụ thực tế](#13-ví-dụ-thực-tế)
14. [Troubleshooting — lỗi thường gặp](#14-troubleshooting--lỗi-thường-gặp)
15. [Best practices](#15-best-practices)
16. [Cheat sheet](#16-cheat-sheet)

---

## 1. Travis CI là gì

### 1.1. Khái niệm

**Travis CI** là một dịch vụ **Continuous Integration (CI)** chạy trên nền tảng đám mây (hosted CI), giúp tự động build và test code mỗi khi bạn push code lên GitHub (hoặc Bitbucket). Nói đơn giản: mỗi lần bạn `git push`, Travis CI sẽ tự động kéo code về một máy ảo sạch, chạy lệnh test, và báo cho bạn biết code có "xanh" (pass) hay "đỏ" (fail) — qua badge trên README, email, hoặc Slack.

Tên "Travis" được đặt theo tên nhân vật Travis trong bộ phim *The Princess Bride*? Không — thực ra nó được đặt theo tên **Travis (Trav)** một chú mèo? Theo founder Mathias Meyer, tên Travis được lấy cảm hứng từ một nhân vật hoạt hình mà đội ngũ yêu thích. Dù nguồn gốc thế nào, Travis CI đã trở thành một trong những dịch vụ CI phổ biến nhất trong giới open source giai đoạn 2012–2020.

### 1.2. Lịch sử phát triển

| Năm | Sự kiện |
|-----|---------|
| 2011 | Dự án khởi động tại Berlin (Đức) bởi Fritz Thielemann, Mathias Meyer, Sven Fuchs, Josh Kalderimis — bắt đầu như một ý tưởng trong phong trào open source Ruby |
| 2012 | Ra mắt public beta, trở thành CI mặc định cho hàng loạt gem Ruby trên GitHub |
| 2013 | Gọi vốn seed round ~1 triệu USD; hỗ trợ thêm Node.js, Python, PHP |
| 2014 | Ra mắt **travis-ci.com** (bản thương mại cho private repo), song song với **travis-ci.org** (miễn phí cho open source) |
| 2015 | Mở rộng mạnh: hỗ trợ Go, Rust, Java, C/C++, macOS builds, Docker |
| 2017 | Được **Idera, Inc.** mua lại |
| 2018 | Travis CI trở thành một trong những CI phổ biến nhất: badge "build | passing" xuất hiện trên hàng triệu README |
| 2019 | Công bố kế hoạch **sáp nhập travis-ci.org vào travis-ci.com** (thống nhất nền tảng) |
| 2020 | **Bước ngoặt pricing**: bỏ gói free không giới hạn, chuyển sang mô hình **credits** (tín dụng). Cộng đồng open source phản ứng dữ dội, làn sóng di cư sang GitHub Actions bắt đầu |
| 2021 | Hoàn tất migration: travis-ci.org ngừng nhận build mới (ngày 31/05/2021), mọi repo chuyển sang travis-ci.com |
| 2022–nay | Duy trì dịch vụ với mô hình credits; vẫn được dùng ở một số dự án legacy và doanh nghiệp |

### 1.3. travis-ci.org vs travis-ci.com — khác nhau thế nào?

Đây là điểm gây nhầm lẫn nhiều nhất cho người mới. Lịch sử có hai domain:

| Tiêu chí | travis-ci.org (cũ) | travis-ci.com (hiện tại) |
|----------|-------------------|--------------------------|
| Mục đích ban đầu | Miễn phí cho **public repo** (open source) | Trả phí cho **private repo** |
| Đăng nhập | GitHub OAuth | GitHub / Bitbucket / GitLab / Assembla OAuth |
| Pricing cũ | Free, không giới hạn (open source) | Gói trả phí theo số user đồng thời |
| Pricing hiện tại | **Đã ngừng hoạt động** (từ 31/05/2021) | Mô hình **credits** — free trial credits, hết thì mua thêm |
| Mã hóa env var | Public key riêng của .org | Public key riêng của .com — **không dùng chéo được** |

> ⚠️ **Lưu ý quan trọng:** Mọi giá trị mã hóa bằng `travis encrypt` trên `.org` đều **không dùng được** trên `.com` và ngược lại, vì mỗi bên dùng một cặp public/private key khác nhau. Khi migrate repo từ `.org` sang `.com`, bạn phải encrypt lại toàn bộ secrets.

**Hiện tại (2026):** chỉ còn **travis-ci.com**. Nếu bạn thấy tài liệu cũ nhắc đến travis-ci.org, hãy hiểu đó là lịch sử — mọi thao tác mới đều thực hiện trên travis-ci.com.

### 1.4. Travis CI hoạt động như thế nào?

```
┌──────────────┐   git push   ┌──────────────┐   webhook   ┌──────────────┐
│   GitHub     │ ───────────► │  Travis CI   │ ──────────► │  Build VM    │
│  repository  │              │  (scheduler) │             │ (Ubuntu/macOS│
└──────────────┘              └──────────────┘             │  /Windows)   │
       ▲                             │                    └──────────────┘
       │                             │                           │
       │        status: pending/      │      git clone +          │  chạy
       │        success/failure       │      .travis.yml          │  install
       └─────────────────────────────┘                           │  script
                                                                 ▼
                                                          ┌──────────────┐
                                                          │  Kết quả:    │
                                                          │  ✓ passing   │
                                                          │  ✗ failing   │
                                                          └──────────────┘
```

Quy trình chi tiết:

1. Bạn push code (hoặc mở pull request) lên GitHub.
2. GitHub gửi **webhook** sang Travis CI.
3. Travis CI đọc file `.travis.yml` ở thư mục gốc của repo (tại commit vừa push).
4. Travis cấp phát một **máy ảo (VM) hoặc container sạch**, clone code về.
5. Chạy tuần tự các phase: `install` → `before_script` → `script` → `after_success`/`after_failure` → `deploy`...
6. Gửi kết quả về GitHub (commit status: 🟢 passing / 🔴 failing / 🟡 pending) và thông báo cho bạn (email/Slack).
7. Hủy máy ảo — build tiếp theo lại bắt đầu từ môi trường sạch (đảm bảo tính lặp lại — *reproducible*).

---

## 2. So sánh Travis CI với GitHub Actions và CircleCI

Ba cái tên phổ biến nhất trong thế giới CI/CD hiện nay. Bảng so sánh dưới đây giúp bạn chọn đúng công cụ.

### 2.1. Bảng so sánh tổng quan

| Tiêu chí | **Travis CI** | **GitHub Actions** | **CircleCI** |
|----------|---------------|--------------------|--------------|
| Ra mắt | 2011 | 2019 | 2011 |
| Tích hợp Git host | GitHub, Bitbucket, GitLab, Assembla | **GitHub (native)** | GitHub, Bitbucket |
| File cấu hình | `.travis.yml` (YAML) | `.github/workflows/*.yml` | `.circleci/config.yml` |
| Cú pháp | Khai báo, đơn giản, ít dòng | YAML + expression `${{ }}`, linh hoạt | YAML + orbs, khá verbose |
| Free tier (2026) | Credits trial (giới hạn, hết phải mua) | **2.000 phút/tháng** (private repo), public repo free | 6.000 credits/tuần (~1.000 phút Linux) |
| OS hỗ trợ | Linux, macOS, Windows | Linux, macOS, Windows (+ self-hosted runner) | Linux, macOS, Windows (+ self-hosted) |
| Docker layer caching | Có (gói trả phí) | Có (qua cache action) | Có (Docker Layer Caching, trả phí) |
| Build matrix | `matrix:` — rất gọn, 1 dòng/version | `strategy.matrix` — linh hoạt hơn | `matrix:` parameters |
| Marketplace / reusable | Hạn chế | **Rất mạnh** (hàng nghìn actions có sẵn) | Orbs (tái sử dụng tốt) |
| Cộng đồng (2026) | Suy giảm sau vụ pricing 2020 | **Lớn nhất hiện nay** | Ổn định, mạnh ở enterprise |
| Phù hợp nhất | Dự án legacy đang chạy Travis; team quen YAML đơn giản | Hầu hết dự án mới trên GitHub | Team cần pipeline phức tạp, enterprise |

### 2.2. Khi nào nên chọn Travis CI?

**Nên chọn Travis CI khi:**
- Dự án của bạn **đang chạy ổn định trên Travis** và không có lý do đủ lớn để migrate (chi phí migrate > lợi ích).
- Bạn thích file cấu hình **ngắn gọn, dễ đọc** — một dự án Node.js test đơn giản chỉ cần ~10 dòng `.travis.yml`.
- Team đã quen với Travis CLI và hệ sinh thái deploy providers có sẵn (Heroku, S3, NPM... chỉ cần vài dòng).

**Nên chọn GitHub Actions khi:**
- Repo đã nằm trên GitHub (không cần cấp quyền cho bên thứ ba, webhook native nhanh hơn).
- Cần free tier hào phóng cho open source (public repo chạy free không giới hạn).
- Cần hệ sinh thái actions phong phú (deploy lên Vercel, gửi Telegram, quét bảo mật... chỉ cần `uses:` một dòng).

**Nên chọn CircleCI khi:**
- Cần pipeline phức tạp: approval thủ công, fan-in/fan-out, test splitting thông minh.
- Chạy test song song ở quy mô lớn và cần tối ưu thời gian (parallelism + test splitting).
- Doanh nghiệp cần SLA, hỗ trợ kỹ thuật và self-hosted runner.

### 2.3. Ví dụ cùng một job viết bằng 3 công cụ

Test một thư viện Node.js trên 3 phiên bản Node — xem sự khác biệt về cú pháp:

**Travis CI** (`.travis.yml`) — gọn nhất:
```yaml
language: node_js
node_js:
  - "18"
  - "20"
  - "22"
script: npm test
```

**GitHub Actions** (`.github/workflows/ci.yml`):
```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test
```

**CircleCI** (`.circleci/config.yml`):
```yaml
version: 2.1
jobs:
  test:
    parameters:
      node-version:
        type: string
    docker:
      - image: cimg/node:<< parameters.node-version >>
    steps:
      - checkout
      - run: npm ci
      - run: npm test
workflows:
  ci:
    jobs:
      - test:
          matrix:
            parameters:
              node-version: ["18", "20", "22"]
```

Nhận xét: Travis CI thắng ở độ ngắn gọn cho các case đơn giản; GitHub Actions thắng ở hệ sinh thái; CircleCI thắng ở pipeline phức tạp.

---

## 3. Đăng ký bằng GitHub và kích hoạt repository

### 3.1. Đăng ký tài khoản

Travis CI không có form "đăng ký email + mật khẩu" truyền thống — bạn đăng nhập bằng tài khoản Git provider (OAuth).

**Các bước:**

1. Truy cập **https://travis-ci.com** (lưu ý: `.com`, không phải `.org`).
2. Nhấn **Sign up with GitHub** (hoặc Bitbucket / GitLab / Assembla tùy nhu cầu).
3. GitHub hiện màn hình **Authorize Travis CI**:
   - Đọc kỹ quyền mà Travis yêu cầu: đọc repo, đọc email, ghi commit status, quản lý webhook...
   - Nhấn **Authorize travis-ci**.
4. Bạn được chuyển về dashboard của Travis CI. Lần đầu đăng nhập, Travis tặng một lượng **trial credits** (số lượng thay đổi theo chính sách từng thời điểm — xem chương 12).

> 💡 **Mẹo:** Nếu bạn là thành viên của organization trên GitHub, sau khi authorize bạn cần vào GitHub → Settings → Applications → Authorized OAuth Apps → Travis CI → **Grant** quyền cho từng organization, nếu không Travis sẽ không thấy repo của org đó.

### 3.2. Kích hoạt repository

Sau khi đăng nhập:

1. Trên dashboard Travis CI, nhấn vào avatar → **Settings** hoặc vào mục **Repositories**.
2. Nhấn **Manage repositories on GitHub** — bạn được chuyển sang trang cài đặt GitHub App của Travis.
3. Chọn phạm vi repo Travis được phép truy cập:
   - **All repositories** (tất cả repo), hoặc
   - **Only select repositories** → chọn từng repo cụ thể (khuyến nghị nếu bạn có nhiều repo cá nhân).
4. Nhấn **Install & Authorize**. GitHub sẽ tự động tạo webhook trỏ về Travis CI cho các repo được chọn.
5. Quay lại dashboard Travis CI — repo của bạn đã xuất hiện trong danh sách. Nhấn vào repo để xem trang build.

**Kích hoạt bằng cách thủ công (nếu webhook không tự tạo):**

Vào repo trên Travis CI → **Settings** → kiểm tra mục **Build pushed branches** và **Build pushed pull requests** đang bật. Nếu GitHub không nhận webhook, vào GitHub repo → Settings → Webhooks → kiểm tra có webhook `https://api.travis-ci.com` không; nếu thiếu, thêm thủ công với:
- Payload URL: `https://api.travis-ci.com`
- Content type: `application/json`
- Events: push, pull_request

### 3.3. Thêm badge trạng thái vào README

Badge là "bộ mặt" của CI — cho người xem biết build mới nhất xanh hay đỏ. Thêm vào đầu file `README.md`:

```markdown
[![Build Status](https://travis-ci.com/gnoudisme/my-project.svg?branch=main)](https://travis-ci.com/gnoudisme/my-project)
```

Thay `gnoudisme/my-project` bằng `username/repo` của bạn, `branch=main` bằng nhánh mặc định của repo.

Các biến thể badge:
```markdown
<!-- Badge cho nhánh develop -->
[![Build Status](https://travis-ci.com/gnoudisme/my-project.svg?branch=develop)](https://travis-ci.com/gnoudisme/my-project)

<!-- Badge dạng flat-square (nếu thích) - dùng shields.io -->
[![Build Status](https://img.shields.io/travis/com/gnoudisme/my-project/main.svg)](https://travis-ci.com/gnoudisme/my-project)
```

### 3.4. Cấu trúc tài khoản và phân quyền

- **Tài khoản cá nhân**: gắn với GitHub user của bạn, thấy tất cả repo bạn có quyền.
- **Organization**: Travis CI tự đồng bộ org từ GitHub. Billing tính theo org riêng.
- **Collaborator**: người được thêm vào repo GitHub sẽ tự thấy repo đó trên Travis (theo quyền GitHub).

---

## 4. `.travis.yml` — cú pháp đầy đủ

File `.travis.yml` đặt ở **thư mục gốc** của repository. Đây là "bản nhạc" mà Travis CI chơi mỗi lần build — khai báo ngôn ngữ, phiên bản, lệnh cài đặt, lệnh test, deploy...

### 4.1. File tối giản

```yaml
language: node_js
node_js:
  - "20"
script: npm test
```

Chỉ 4 dòng: Travis sẽ dùng Node.js 20, chạy `npm install` (mặc định cho Node) rồi `npm test`.

### 4.2. Toàn bộ các key phổ biến

Bảng dưới đây liệt kê các key hay dùng nhất trong `.travis.yml`:

| Key | Kiểu | Mô tả |
|-----|------|-------|
| `language` | string | Ngôn ngữ chính: `node_js`, `python`, `ruby`, `go`, `java`, `php`, `rust`, `c`, `cpp`, `generic`, ... |
| `os` | string/list | Hệ điều hành: `linux` (mặc định), `osx`, `windows` |
| `dist` | string | Distro Ubuntu cho Linux: `xenial`, `bionic`, `focal`, `jammy` |
| `arch` | string/list | Kiến trúc CPU: `amd64` (mặc định), `arm64`, `ppc64le`, `s390x` |
| `env` | list/map | Biến môi trường (xem mục 4.4) |
| `matrix` / `jobs` | map | Build matrix: chạy nhiều job song song (xem chương 7) |
| `before_install` | list | Lệnh chạy trước phase install |
| `install` | list | Lệnh cài dependencies (ghi đè mặc định của ngôn ngữ) |
| `before_script` | list | Lệnh chạy trước script chính (vd: migrate DB, build assets) |
| `script` | list | **Lệnh test chính** — fail ở đây = build fail |
| `before_cache` | list | Lệnh chạy trước khi lưu cache |
| `after_success` | list | Chạy khi build thành công (vd: gửi coverage) |
| `after_failure` | list | Chạy khi build thất bại (vd: dump log) |
| `after_script` | list | Luôn chạy sau script (dù pass hay fail) — tương tự `finally` |
| `before_deploy` / `deploy` / `after_deploy` | list/map | Deploy tự động (xem chương 10) |
| `cache` | map | Caching dependencies (xem chương 8) |
| `services` | list | Service đi kèm: `postgresql`, `mysql`, `redis`, `mongodb`, `docker`... |
| `addons` | map | Addons: `apt` packages, Chrome, PostgreSQL version... |
| `notifications` | map | Thông báo: email, Slack, webhook... |
| `branches` | map | Giới hạn nhánh nào được build (`only` / `except`) |
| `git` | map | Tùy chỉnh git: `depth`, `submodules`, `strategy` |
| `sudo` | bool | Bắt buộc `sudo: required` cho một số môi trường cũ |

### 4.3. Ví dụ file đầy đủ (tham khảo)

```yaml
# Ngôn ngữ và môi trường
language: node_js
node_js: "20"
os: linux
dist: focal
arch: amd64

# Chỉ build các nhánh này
branches:
  only:
    - main
    - develop
    - /^release\/.*$/   # regex: các nhánh release/*

# Biến môi trường
env:
  global:
    - NODE_ENV=test
    - secure: "ENCRYPTED_VALUE_1"   # mã hóa bằng travis encrypt
  jobs:
    - DB=postgres
    - DB=mysql

# Service đi kèm
services:
  - postgresql
  - redis-server

# Cài thêm gói hệ thống
addons:
  apt:
    packages:
      - postgresql-client

# Cache
cache:
  npm: true
  directories:
    - node_modules/.cache

# Lifecycle
before_install:
  - echo "Bắt đầu build..."
install:
  - npm ci
before_script:
  - npm run db:migrate
script:
  - npm run lint
  - npm test
after_success:
  - npm run coverage
after_failure:
  - echo "Build thất bại, xem log phía trên"

# Thông báo
notifications:
  email:
    recipients:
      - dev@congty.vn
    on_success: change   # chỉ gửi khi trạng thái đổi
    on_failure: always
  slack:
    rooms:
      - secure: "ENCRYPTED_SLACK_TOKEN"

# Deploy
deploy:
  provider: npm
  email: dev@congty.vn
  api_key:
    secure: "ENCRYPTED_NPM_TOKEN"
  on:
    tags: true
    branch: main
```

### 4.4. Khai báo biến môi trường (`env`)

Có 3 cách khai báo, từ đơn giản đến phức tạp:

```yaml
# Cách 1: danh sách đơn giản (áp dụng cho mọi job)
env:
  - DB=postgres
  - RAILS_ENV=test

# Cách 2: tách global (chung) và jobs (tạo matrix)
env:
  global:
    - API_URL=https://api.vi-du.vn
    - secure: "MA_HOA_1"          # biến mã hóa, dùng chung mọi job
  jobs:
    - TEST_SUITE=unit
    - TEST_SUITE=integration      # tạo 2 job song song

# Cách 3: dạng map (dễ đọc khi nhiều biến)
env:
  global:
    NODE_ENV: test
    TZ: Asia/Ho_Chi_Minh
```

> ⚠️ **Không bao giờ** ghi secret (token, password, API key) dạng plain text trong `.travis.yml` — file này nằm trong git history. Dùng `secure:` (mã hóa) hoặc khai báo trong Settings của repo trên web Travis CI (xem chương 9).

### 4.5. Giới hạn nhánh build (`branches`)

```yaml
branches:
  only:
    - main
    - develop
    - /^hotfix\/.*$/      # regex phải đặt trong /.../
  except:
    - experimental
```

Lưu ý:
- Nếu không khai báo `branches`, Travis build **mọi nhánh** có push.
- Tag: mặc định Travis **không build tag** trừ khi bạn cho phép:
  ```yaml
  branches:
    only:
      - main
      - /^v\d+\.\d+\.\d+$/   # build các tag dạng v1.2.3
  ```

### 4.6. Tùy chỉnh git clone

```yaml
git:
  depth: 10              # shallow clone 10 commit gần nhất (nhanh hơn)
  # depth: false         # clone full history (cần khi deploy dùng git log)
  submodules: true       # tự động init/update submodule
  strategy: clone        # hoặc 'tarball' (nhanh hơn nhưng không có .git)
  quiet: true            # bớt log khi clone
```

### 4.7. Validate YAML trước khi push

YAML rất nhạy với thụt đầu dòng (indentation). Một lỗi phổ biến:

```yaml
# ❌ SAI: 'script' bị thụt sai cấp
language: node_js
  script: npm test

# ✅ ĐÚNG
language: node_js
script: npm test
```

Cách kiểm tra:
1. Dùng `travis lint` (xem chương 11).
2. Dán file vào **http://lint.travis-ci.com** (web linter chính thức).
3. Chạy `python -c "import yaml,sys; yaml.safe_load(open('.travis.yml'))"` để check cú pháp YAML thuần.

---

## 5. Build lifecycle

Build lifecycle là **trình tự các phase** mà Travis chạy. Hiểu rõ thứ tự này giúp bạn đặt lệnh đúng chỗ và debug nhanh hơn.

### 5.1. Sơ đồ lifecycle đầy đủ

```
 1. before_install ─┐
 2. install          │  ← nếu lệnh nào exit != 0 → build FAILED ngay
 3. before_script   ─┘         (trừ after_* vẫn chạy)

 4. script ────────────── ← phase QUAN TRỌNG NHẤT
                           exit 0 = pass, exit != 0 = fail

 5a. after_success ──── chạy khi script pass
 5b. after_failure ──── chạy khi script fail
 5c. after_script ───── luôn chạy (pass hay fail)

 6. before_deploy ─┐
 7. deploy          │  ← chỉ chạy khi build pass (mặc định)
 8. after_deploy   ─┘

 9. after_cache ─────── lưu cache (nếu bật)
```

Chi tiết từng phase:

| # | Phase | Khi nào chạy | Fail thì sao? | Dùng để làm gì (ví dụ thực tế) |
|---|-------|--------------|---------------|-------------------------------|
| 1 | `before_install` | Trước khi cài dependencies | Build fail ngay | Cài tool hệ thống: `nvm install`, `apt-get`, cập nhật submodule |
| 2 | `install` | Cài dependencies | Build fail ngay | `npm ci`, `pip install -r requirements.txt`, `bundle install`, `go mod download` |
| 3 | `before_script` | Trước lệnh test | Build fail ngay | Chạy migration DB, build frontend assets, khởi động server test |
| 4 | `script` | **Chạy test chính** | **Build fail** | `npm test`, `pytest`, `go test ./...`, `mvn verify` |
| 5a | `after_success` | Script pass | Không ảnh hưởng kết quả | Gửi coverage lên Codecov, build docs |
| 5b | `after_failure` | Script fail | Không ảnh hưởng kết quả | Dump log, chụp debug info, gửi cảnh báo |
| 5c | `after_script` | Luôn chạy sau script | Không ảnh hưởng kết quả | Dọn dẹp, tắt server test (như `finally`) |
| 6 | `before_deploy` | Trước deploy, khi build pass | Bỏ qua deploy | Build production bundle, nén artifact |
| 7 | `deploy` | Deploy (mặc định chỉ khi pass) | Đánh dấu build "errored" | Đẩy lên NPM, Heroku, S3, GitHub Pages... |
| 8 | `after_deploy` | Sau deploy | Không ảnh hưởng | Thông báo "đã deploy xong" lên Slack |

### 5.2. Quy tắc exit code

- Mỗi lệnh trong phase `install`, `before_install`, `before_script`, `script`: nếu **exit code != 0** → dừng ngay, build **failed**.
- Các phase `after_*`: lệnh fail **không** làm đổi kết quả build (nhưng vẫn hiện log đỏ).
- Muốn một lệnh được phép fail mà không dừng build: thêm `|| true`:
  ```yaml
  script:
    - npm test
    - npm run optional-check || true   # fail cũng không sao
  ```

### 5.3. Ví dụ lifecycle cho dự án Node.js + PostgreSQL

```yaml
language: node_js
node_js: "20"

services:
  - postgresql

addons:
  postgresql: "14"

before_install:
  # Tạo database cho test
  - psql -c 'CREATE DATABASE myapp_test;' -U postgres

install:
  - npm ci

before_script:
  # Chạy migration trước khi test
  - npm run db:migrate -- --env test
  # Build frontend để test E2E có asset mới nhất
  - npm run build

script:
  # 1. Lint trước — lỗi style thì fail nhanh, khỏi chạy test lâu
  - npm run lint
  # 2. Test chính
  - npm test

after_success:
  # Gửi coverage (chỉ khi pass)
  - npx codecov --token $CODECOV_TOKEN

after_failure:
  # Dump log Postgres để debug
  - echo "=== Postgres log ==="
  - tail -n 50 /var/log/postgresql/postgresql-14-main.log || true

after_script:
  # Luôn dọn file tạm
  - rm -rf /tmp/myapp-test-*
```

### 5.4. Ví dụ lifecycle cho dự án Python

```yaml
language: python
python: "3.11"

services:
  - postgresql
  - redis-server

before_install:
  - sudo apt-get update -qq
  - sudo apt-get install -y libpq-dev

install:
  - pip install -r requirements.txt
  - pip install -r requirements-test.txt

before_script:
  - python manage.py migrate --settings=myapp.settings.test

script:
  - flake8 myapp/            # lint
  - pytest --cov=myapp       # test + coverage

after_success:
  - codecov
```

### 5.5. Bỏ qua build với `[skip ci]`

Đôi khi bạn chỉ sửa README hoặc docs, không muốn tốn credits:

```bash
git commit -m "Cập nhật README [skip ci]"
# hoặc [ci skip] — cả hai đều được Travis nhận
git push
```

Travis sẽ bỏ qua build cho commit đó. Áp dụng cho cả message của merge commit và tag.

---

## 6. Hỗ trợ ngôn ngữ lập trình

Travis CI hỗ trợ sẵn hàng chục ngôn ngữ. Mỗi ngôn ngữ có **lệnh `install` và `script` mặc định** — nếu bạn không ghi đè, Travis tự chạy chúng.

### 6.1. Bảng tổng hợp các ngôn ngữ phổ biến

| `language:` | Phiên bản khai báo | `install` mặc định | `script` mặc định |
|-------------|-------------------|--------------------|--------------------|
| `node_js` | `node_js: ["18","20"]` hoặc `.nvmrc` | `npm install` | `npm test` |
| `python` | `python: ["3.10","3.11"]` | `pip install -r requirements.txt` | `pytest` (hoặc `nosetests`) |
| `ruby` | `rvm: ["3.1","3.2"]` | `bundle install` | `bundle exec rake` |
| `go` | `go: ["1.21","1.22"]` | `go get ./...` | `go test ./...` |
| `java` | `jdk: ["openjdk17"]` | — | `mvn test -B` / `gradle test` |
| `php` | `php: ["8.1","8.2"]` | `composer install` | `phpunit` |
| `rust` | `rust: [stable, beta]` | `cargo build` | `cargo test` |
| `c` / `cpp` | `compiler: [gcc, clang]` | — | `./configure && make && make test` |
| `generic` | — | — | — (bạn tự viết hết) |
| `android` | `android: components: [...]` | sdkmanager | `gradle build` |

### 6.2. Node.js

```yaml
language: node_js

# Cách 1: liệt kê version
node_js:
  - "18"
  - "20"
  - "22"

# Cách 2: đọc từ file .nvmrc (khuyến nghị: 1 nguồn sự thật)
# node_js: "20"  # bỏ, để Travis tự đọc .nvmrc

cache: npm          # cache node_modules

install: npm ci     # nhanh + sạch hơn npm install trên CI
script:
  - npm run lint
  - npm test
```

Quản lý version chi tiết:
```yaml
language: node_js
node_js: "20"

before_install:
  - npm install -g npm@10        # ép version npm
  - node --version && npm --version
```

### 6.3. Python

```yaml
language: python
python:
  - "3.10"
  - "3.11"
  - "3.12"

cache: pip          # cache ~/.cache/pip

install:
  - pip install -U pip
  - pip install -r requirements.txt

script:
  - pytest -v
```

Dùng `tox` để test nhiều môi trường (chuẩn của cộng đồng Python):
```yaml
language: python
python:
  - "3.10"
  - "3.11"

install:
  - pip install tox

script: tox
```
```ini
# tox.ini
[tox]
envlist = py310, py311

[testenv]
deps = pytest
commands = pytest tests/
```

### 6.4. Go

```yaml
language: go
go:
  - "1.21"
  - "1.22.x"     # x = patch mới nhất

# Go module (khuyến nghị từ Go 1.11+)
# go: 1.22.x

install: go mod download

script:
  - go vet ./...
  - go test -v -race -coverprofile=coverage.out ./...

after_success:
  - bash <(curl -s https://codecov.io/bash)
```

Biến môi trường hữu ích cho Go:
```yaml
env:
  global:
    - GO111MODULE=on
    - CGO_ENABLED=0        # build static binary
```

### 6.5. Rust

```yaml
language: rust
rust:
  - stable
  - beta
  - nightly        # cho phép fail (xem mục allow_failures ở chương 7)

cache: cargo       # cache ~/.cargo và target/

script:
  - cargo build --verbose
  - cargo test --verbose
```

### 6.6. Java (Maven / Gradle)

```yaml
# Maven
language: java
jdk:
  - openjdk11
  - openjdk17

cache:
  directories:
    - $HOME/.m2          # cache Maven repo

script: mvn -B verify    # -B = batch mode, không hỏi interactive
```

```yaml
# Gradle
language: java
jdk: openjdk17

cache:
  directories:
    - $HOME/.gradle

script: ./gradlew build --no-daemon
```

### 6.7. PHP

```yaml
language: php
php:
  - "8.1"
  - "8.2"
  - "8.3"

cache:
  directories:
    - $HOME/.composer/cache

before_install:
  - composer self-update

install: composer install --no-interaction --prefer-dist

script: vendor/bin/phpunit
```

### 6.8. C / C++

```yaml
language: cpp
compiler:
  - gcc
  - clang
os:
  - linux
  - osx          # test đa compiler + đa OS

script:
  - ./configure
  - make
  - make test
```

### 6.9. Generic (ngôn ngữ không có sẵn)

Khi dùng ngôn ngữ lạ (Dart, Elixir chưa hỗ trợ sẵn, hoặc project đa ngôn ngữ), dùng `generic` và tự viết toàn bộ:

```yaml
language: generic

# Cài Dart SDK thủ công
before_install:
  - wget https://storage.googleapis.com/dart-archive/channels/stable/release/latest/sdk/dartsdk-linux-x64-release.zip
  - unzip -q dartsdk-linux-x64-release.zip
  - export PATH="$PWD/dart-sdk/bin:$PATH"

install:
  - dart pub get

script:
  - dart analyze
  - dart test
```

> 💡 `language: generic` cũng hữu ích khi bạn chỉ muốn chạy shell script, build Docker image, hoặc deploy mà không cần runtime ngôn ngữ nào.

---

## 7. Build matrix — test nhiều phiên bản song song

Build matrix là **tính năng mạnh nhất** của Travis CI: từ một file `.travis.yml`, Travis tự sinh ra nhiều job chạy **song song**, mỗi job là một tổ hợp khác nhau của (ngôn ngữ version × OS × env var...).

### 7.1. Matrix cơ bản: nhiều version ngôn ngữ

```yaml
language: node_js
node_js:
  - "18"     # → job 1
  - "20"     # → job 2
  - "22"     # → job 3
script: npm test
```

Travis tạo 3 job song song. Trên dashboard bạn thấy từng job riêng: `18`, `20`, `22`.

### 7.2. Matrix nhiều chiều (version × OS × env)

Mỗi trục thêm vào sẽ **nhân** số job theo tích Descartes:

```yaml
language: node_js
node_js:
  - "18"
  - "20"          # 2 versions
os:
  - linux
  - osx           # 2 OS
env:
  jobs:
    - DB=postgres
    - DB=mysql    # 2 env
script: npm test
# → 2 × 2 × 2 = 8 job song song
```

> ⚠️ Cẩn thận **bùng nổ tổ hợp**: 5 version × 3 OS × 4 env = 60 job — tốn credits và thời gian chờ queue. Chỉ matrix những gì thực sự cần.

### 7.3. `jobs.include` — thêm job đặc biệt

Thêm job không nằm trong ma trận (ví dụ: job lint, job build docs):

```yaml
language: node_js
node_js:
  - "18"
  - "20"

jobs:
  include:
    - name: "Lint & format check"
      node_js: "20"
      script: npm run lint
    - name: "Build docs"
      node_js: "20"
      script: npm run docs:build
```

### 7.4. `jobs.exclude` — loại tổ hợp không cần thiết

```yaml
language: node_js
node_js:
  - "18"
  - "20"
os:
  - linux
  - osx

jobs:
  exclude:
    # Không test Node 18 trên macOS (chỉ test 20)
    - node_js: "18"
      os: osx
```

### 7.5. `allow_failures` — cho phép fail không tính

Dùng cho version thử nghiệm (nightly, beta) — job đỏ nhưng build tổng vẫn xanh:

```yaml
language: rust
rust:
  - stable
  - beta
  - nightly

jobs:
  allow_failures:
    - rust: nightly
    - rust: beta

  # fast_finish: kết thúc build ngay khi các job "bắt buộc" xong,
  # không chờ job allow_failures
  fast_finish: true
```

Trên dashboard, job allow_failures fail sẽ hiện màu vàng (không đỏ), và build tổng vẫn pass nếu các job còn lại xanh.

### 7.6. Ví dụ thực tế: thư viện Python hỗ trợ nhiều version + nhiều DB

```yaml
language: python

python:
  - "3.10"
  - "3.11"
  - "3.12"

env:
  jobs:
    - DB=postgres DB_URL=postgresql://postgres@localhost/mydb
    - DB=mysql    DB_URL=mysql://root@localhost/mydb
    - DB=sqlite   DB_URL=sqlite:///test.db

services:
  - postgresql
  - mysql

jobs:
  exclude:
    # sqlite không cần service nào, nhưng vẫn chạy trên mọi python — giữ lại
    # Loại combo python 3.10 + mysql (không hỗ trợ driver cũ)
    - python: "3.10"
      env: DB=mysql    DB_URL=mysql://root@localhost/mydb

install: pip install -r requirements.txt
script: pytest tests/ -k $DB
```

### 7.7. Giới hạn và tối ưu matrix

```yaml
# Giới hạn số job chạy đồng thời (tránh nghẽn queue của team)
# Không có key native — dùng Travis CI Enterprise hoặc chia nhỏ file

# Đặt tên job cho dễ đọc trên dashboard
jobs:
  include:
    - name: "Node 20 + Postgres 14 (main test)"
      node_js: "20"
      env: DB=postgres
      script: npm test
```

**Best practices cho matrix:**
1. **Test version mới nhất + LTS**, không test mọi version từng release (vd: Node chỉ test 18, 20, 22 — bỏ 19, 21 là odd release).
2. Dùng `fast_finish: true` + `allow_failures` cho version thử nghiệm.
3. Đặt `name:` cho từng job trong `include` để dashboard dễ đọc.
4. Tránh matrix OS nếu không cần — test `osx`/`windows` đắt credits hơn `linux` nhiều (xem chương 12).

---

## 8. Caching dependencies

Mỗi build Travis chạy trên VM **sạch** — nghĩa là `npm install` phải tải lại hàng trăm MB mỗi lần. Cache giúp lưu dependencies giữa các build, giảm thời gian từ vài phút xuống vài chục giây.

### 8.1. Bật cache theo ngôn ngữ (1 dòng)

```yaml
language: node_js
cache: npm          # cache node_modules

language: python
cache: pip          # cache ~/.cache/pip

language: ruby
cache: bundler      # cache vendor/bundle

language: rust
cache: cargo        # cache ~/.cargo + target/

language: php
cache: composer     # cache ~/.composer/cache

language: go
cache:               # Go module cache (thủ công)
  directories:
    - $HOME/go/pkg/mod
```

### 8.2. Cache thư mục tùy ý

```yaml
cache:
  directories:
    - node_modules          # cache npm thủ công
    - $HOME/.m2             # Maven local repo
    - $HOME/.gradle         # Gradle cache
    - $HOME/.cache/pip      # pip cache
    - vendor/bundle         # bundler path
```

### 8.3. Cache hoạt động như thế nào?

1. **Build đầu tiên**: chạy `install` bình thường (tải dependencies), cuối build Travis nén các thư mục cache và upload lên S3.
2. **Build tiếp theo**: trước phase `install`, Travis tải cache về và giải nén — `npm ci` chỉ cần kiểm tra, không tải lại.
3. Cache được **gắn với nhánh**: nhánh `develop` tách từ `main` sẽ kế thừa cache của `main` (cache fallback theo git history), nhưng cache của `develop` không ghi đè lên `main`.

```
Build #1 (main)     → install (tải mới) → cuối build: lưu cache ─┐
Build #2 (main)     → tải cache ──→ install (nhanh) ──→ cập nhật cache
Build #3 (develop)  → dùng cache của main (fallback) ──→ lưu cache riêng
```

### 8.4. `before_cache` — dọn dẹp trước khi lưu

Cache phình to làm chậm cả tải lên lẫn tải xuống. Dọn file không cần thiết:

```yaml
cache:
  directories:
    - node_modules

before_cache:
  # Xóa log, file tạm trong node_modules trước khi nén cache
  - rm -rf node_modules/.cache
  - find node_modules -name "*.log" -delete
```

### 8.5. Tắt cache khi cần

```yaml
cache: false                    # tắt hoàn toàn

# Hoặc tắt cache cho một số nhánh (vd: nhánh release luôn build sạch)
cache:
  npm: true
  directories:
    - node_modules
branches:
  only:
    - main
```

**Khi nào nên xóa cache?** Trên web Travis CI: vào repo → **Settings** → **Caches** → **Delete all caches**. Làm vậy khi:
- Dependencies bị "kẹt" version cũ dù đã sửa `package.json`/`requirements.txt`.
- Build fail một cách khó hiểu chỉ trên CI (local vẫn pass) — nghi cache hỏng.

### 8.6. Lưu ý quan trọng

- Cache **không đảm bảo** tồn tại: Travis có thể xóa cache cũ bất cứ lúc nào. Luôn viết `install` sao cho **chạy được cả khi không có cache**.
- Không cache **artifact build** (file biên dịch, binary) nếu nó phụ thuộc vào code — dễ gây "test pass giả".
- Không bao giờ đưa **secret** vào thư mục cache.

---

## 9. Biến môi trường mã hóa (travis encrypt)

Secret (API token, password, private key) **không được** ghi plain text vào `.travis.yml`. Travis cung cấp cơ chế mã hóa bất đối xứng: bạn mã hóa bằng **public key** của repo, Travis giải mã bằng **private key** chỉ nó mới có — ngay trong lúc build.

### 9.1. Nguyên lý

```
Bạn (local)                          Travis CI
─────────────                        ─────────
travis encrypt "TOKEN=abc123"
  → mã hóa bằng PUBLIC KEY của repo
  → ra chuỗi "secure: xxxxx"
  → paste vào .travis.yml, push lên GitHub (an toàn, ai cũng đọc được nhưng không giải được)

Khi build:
  Travis dùng PRIVATE KEY (chỉ Travis giữ) giải mã
  → biến môi trường TOKEN=abc123 có mặt trong build
  → log tự động che giá trị (hiện [secure])
```

### 9.2. Cài Travis CLI và encrypt

```bash
# Cài đặt (cần Ruby)
gem install travis

# Đăng nhập (dùng GitHub token — xem chương 11)
travis login --com --github-token <GITHUB_TOKEN>

# Mã hóa một biến cho repo hiện tại
cd my-project
travis encrypt "NPM_TOKEN=npm_abc123xyz" --com --add env.global
```

Lệnh trên tự động **chèn vào `.travis.yml`**:

```yaml
env:
  global:
    - secure: "aBcDeFgHiJkLmNoPqRsTuVwXyZ1234567890..."
```

Nếu không muốn tự chèn, bỏ `--add` để chỉ in ra chuỗi mã hóa rồi paste tay.

### 9.3. Mã hóa file (cho key dài: SSH key, GPG key, JSON credential)

Biến `secure:` chỉ phù hợp chuỗi ngắn. Với file (vd: `service-account.json` của GCP, private key SSH), dùng `encrypt-file`:

```bash
# Mã hóa file deploy-key (giữ bản gốc, tạo ra deploy-key.enc)
travis encrypt-file deploy-key --com --add

# Lệnh --add tự chèn vào .travis.yml:
```

```yaml
before_install:
  # Giải mã trong lúc build (2 biến $encrypted_* do Travis tự sinh)
  - openssl aes-256-cbc -K $encrypted_0a1b2c3d_key -iv $encrypted_0a1b2c3d_iv
      -in deploy-key.enc -out deploy-key -d
  - chmod 600 deploy-key
```

> ⚠️ **Nhớ thêm `*.enc` vào git** (file đã mã hóa thì an toàn khi public), và thêm **file gốc** vào `.gitignore`:
> ```
> # .gitignore
> deploy-key
> service-account.json
> ```

### 9.4. Khai báo secret trên web UI (cách đơn giản nhất)

Nếu ngại dùng CLI, vào Travis CI web → repo → **Settings** → **Environment Variables**:
- Name: `NPM_TOKEN` | Value: `npm_abc123xyz`
- Bật **"Display value in build log" = OFF** (mặc định off — luôn để off với secret).
- Nhấn **Add**.

Ưu điểm: không cần encrypt, đổi secret không cần push code. Nhược điểm: secret không đi theo repo (fork không có), khó version-control.

### 9.5. Quy tắc an toàn với secret

1. **Pull request từ fork không được dùng secret**: Travis tự động **vô hiệu hóa** mọi biến `secure:` và env var trên web UI khi build PR đến từ fork (tránh lộ secret qua PR độc hại). Nếu deploy cần secret, chỉ chạy trên nhánh chính, không chạy trên PR:
   ```yaml
   deploy:
     provider: npm
     on:
       branch: main
       # PR từ fork không có secret → deploy sẽ fail, nên giới hạn nhánh
   ```
2. Log tự che: Travis thay giá trị secret trong log bằng `[secure]` — nhưng **đừng `echo $SECRET`** hay in ra file rồi `cat`.
3. Key `.org` vs `.com` khác nhau (đã nói ở 1.3) — encrypt lại khi migrate.
4. Xoay (rotate) token định kỳ; khi lộ token, revoke ngay trên dịch vụ gốc rồi encrypt lại.

### 9.6. Ví dụ tổng hợp

```yaml
env:
  global:
    # Biến thường
    - NODE_ENV=production
    # Biến mã hóa bằng travis encrypt
    - secure: "Y29kZUNvdmVydG9rZW4xMjM0NTY3ODkw..."
    - secure: "U2Fua0JveE1hdXRoVG9rZW4xMjM0NTY3ODkw..."

script:
  - echo "Deploy lên server staging..."
  # $DEPLOY_TOKEN đã có mặt trong môi trường build (được giải mã)
  - ./deploy.sh --token "$DEPLOY_TOKEN"
```

---

## 10. Deploy tự động (providers)

Travis CI tích hợp sẵn hàng chục **deployment provider** — chỉ cần khai báo vài dòng là tự động đẩy artifact đi sau khi build pass.

### 10.1. Cơ chế chung

```yaml
deploy:
  provider: <ten-provider>   # npm, heroku, s3, pages, ...
  # ... thông tin xác thực (nên dùng secure:)
  on:
    branch: main             # chỉ deploy từ nhánh này
    # tags: true             # hoặc chỉ deploy khi có tag
    # condition: $TEST_SUITE = integration
  run:
    - "npm run build"        # lệnh chạy trước khi deploy (tùy provider)
```

Mặc định deploy **chỉ chạy khi build pass** và **bỏ qua PR**. Muốn deploy cả khi có job allow_failures đỏ: thêm `allow_failure: true`? Không — dùng `on: { all_branches: true }` tùy case.

**Nhiều provider cùng lúc:**
```yaml
deploy:
  - provider: npm
    email: dev@vi-du.vn
    api_key:
      secure: "NPM_TOKEN_MA_HOA"
    on:
      tags: true
  - provider: s3
    access_key_id:
      secure: "AWS_KEY_MA_HOA"
    secret_access_key:
      secure: "AWS_SECRET_MA_HOA"
    bucket: "my-app-releases"
    on:
      tags: true
```

### 10.2. GitHub Pages (`pages`)

Deploy tài liệu/static site lên GitHub Pages — cực phổ biến cho docs:

```yaml
deploy:
  provider: pages
  # Cách 1: dùng GitHub token (khuyến nghị)
  github_token:
    secure: "GITHUB_TOKEN_MA_HOA"   # token cần quyền repo/public_repo
  # Cách 2: dùng username/password (không khuyến nghị)
  # username: gnoudisme
  # password:
  #   secure: "..."
  local_dir: docs/_build/html      # thư mục chứa site tĩnh (vd: output của Sphinx)
  target_branch: gh-pages          # nhánh chứa site (mặc định gh-pages)
  fqdn: docs.vi-du.vn              # (tùy chọn) custom domain → tạo file CNAME
  project_name: my-project         # (tùy chọn)
  on:
    branch: main
```

**Ví dụ đầy đủ: build Sphinx docs rồi deploy lên Pages** (xem thêm ví dụ 13.3):
```yaml
language: python
python: "3.11"

install:
  - pip install sphinx sphinx-rtd-theme

script:
  - cd docs && make html && cd ..

deploy:
  provider: pages
  github_token:
    secure: "TOKEN..."
  local_dir: docs/_build/html
  on:
    branch: main
```

Tạo GitHub token: GitHub → Settings → Developer settings → Personal access tokens → **Tokens (classic)** → Generate → tick `repo` (hoặc `public_repo` nếu repo public) → copy token → `travis encrypt GH_TOKEN=<token>`.

### 10.3. Heroku (`heroku`)

```yaml
deploy:
  provider: heroku
  api_key:
    secure: "HEROKU_API_KEY_MA_HOA"
  app: my-app-staging              # tên app trên Heroku
  # app:
  #   master: my-app-production    # deploy nhánh khác nhau → app khác nhau
  #   develop: my-app-staging
  on:
    branch: main
  run:
    - "npm run build"              # chạy trước khi đẩy
    - "npm run db:migrate"         # chạy sau khi đẩy (release phase)
```

Lấy Heroku API key: `heroku auth:token` (Heroku CLI) → encrypt.

### 10.4. NPM (`npm`)

Tự động publish package lên npmjs.com khi tạo tag:

```yaml
deploy:
  provider: npm
  email: dev@vi-du.vn
  api_key:
    secure: "NPM_TOKEN_MA_HOA"     # npm access token (classic hoặc granular)
  tag: v2.1.0                      # (tùy chọn) ghi đè version trong package.json
  on:
    tags: true                     # chỉ publish khi push tag
    branch: main
```

Quy trình release chuẩn:
```bash
npm version patch   # tăng 1.2.3 → 1.2.4, tự tạo commit + tag v1.2.4
git push --follow-tags
# → Travis build tag → test pass → tự publish lên npm
```

> 💡 Dùng **npm access token** loại "Automation" (không cần 2FA khi publish từ CI). Tạo tại npmjs.com → Access Tokens → Generate New Token → Automation.

### 10.5. Amazon S3 (`s3`)

Đẩy file build/binary lên S3:

```yaml
deploy:
  provider: s3
  access_key_id:
    secure: "AWS_ACCESS_KEY_MA_HOA"
  secret_access_key:
    secure: "AWS_SECRET_MA_HOA"
  bucket: "releases.vi-du.vn"
  region: "ap-southeast-1"          # Singapore — gần VN nhất
  local_dir: dist                   # thư mục upload
  upload-dir: "my-app/v2"           # tiền tố trên S3
  acl: public_read                  # hoặc private
  on:
    tags: true
```

### 10.6. Docker Hub

Travis không có provider "dockerhub" riêng — deploy Docker image bằng **script provider** hoặc lệnh docker trực tiếp:

```yaml
services:
  - docker

env:
  global:
    - secure: "DOCKER_USERNAME_MA_HOA"   # hoặc để plain nếu là username public
    - secure: "DOCKER_PASSWORD_MA_HOA"   # Docker Hub access token

script:
  - docker build -t gnoudisme/my-app:$TRAVIS_TAG .

deploy:
  provider: script
  script: bash scripts/docker-push.sh
  on:
    tags: true
```

```bash
# scripts/docker-push.sh
#!/usr/bin/env bash
set -euo pipefail

echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

VERSION="${TRAVIS_TAG:-latest}"
IMAGE="gnoudisme/my-app"

docker tag "$IMAGE:$TRAVIS_TAG" "$IMAGE:$VERSION"
docker tag "$IMAGE:$TRAVIS_TAG" "$IMAGE:latest"

docker push "$IMAGE:$VERSION"
docker push "$IMAGE:latest"
```

> 🔐 **Bảo mật:** luôn dùng Docker Hub **access token** thay vì password chính, và login bằng `--password-stdin` (không để password lộ trong `ps` hay log).

### 10.7. Các provider khác (tham khảo nhanh)

| Provider | `provider:` | Dùng cho |
|----------|-------------|----------|
| GitHub Releases | `releases` | Đính kèm binary vào release notes |
| Heroku | `heroku` | Deploy web app |
| NPM | `npm` | Publish package |
| PyPI | `pypi` | Publish Python package |
| S3 | `s3` | Lưu trữ file, static hosting |
| GitHub Pages | `pages` | Docs / static site |
| Firebase | `firebase` | Deploy web app lên Firebase Hosting |
| Netlify | `netlify` | Deploy static site |
| Surge | `surge` | Deploy static site nhanh |
| Cloud Foundry | `cloudfoundry` | Enterprise PaaS |
| Script tùy ý | `script` | Chạy script shell bất kỳ |

**Ví dụ: GitHub Releases** — đính kèm file binary vào release:
```yaml
deploy:
  provider: releases
  api_key:
    secure: "GITHUB_TOKEN_MA_HOA"
  file: dist/my-app-linux-amd64.tar.gz
  file_glob: true                 # hỗ trợ wildcard
  draft: false
  prerelease: false
  on:
    tags: true
```

**Ví dụ: PyPI** — publish Python package:
```yaml
deploy:
  provider: pypi
  username: __token__
  password:
    secure: "PYPI_TOKEN_MA_HOA"    # PyPI API token (pypi-...)
  distributions: "sdist bdist_wheel"
  on:
    tags: true
```

---

## 11. Travis CLI

**Travis CLI** là công cụ dòng lệnh chính thức (`travis` gem, viết bằng Ruby) — dùng để encrypt secret, kiểm tra file cấu hình, xem log, trigger build... mà không cần mở web.

### 11.1. Cài đặt

```bash
# Cách 1: qua RubyGems (cần Ruby >= 2.7)
gem install travis

# Kiểm tra
travis version
# travis 1.11.1

# Cách 2: trên macOS (Homebrew)
brew install travis

# Cách 3: nếu gặp lỗi SSL/native extension trên Linux
sudo apt-get install -y ruby-dev build-essential
gem install travis --no-document
```

> ⚠️ Nếu máy chưa có Ruby, cài Ruby trước: dùng `rbenv`/`rvm`, hoặc `sudo apt-get install ruby-full`.

### 11.2. Đăng nhập

```bash
# Đăng nhập vào travis-ci.com (mặc định mới là .com)
travis login --com

# Đăng nhập bằng GitHub token (dùng trong CI/script, không hỏi interactive)
travis login --com --github-token <GITHUB_PERSONAL_ACCESS_TOKEN>

# Kiểm tra đang login ai
travis whoami --com

# Đăng xuất
travis logout --com
```

GitHub token cần quyền tối thiểu: `repo`, `read:org`, `user:email`. Tạo tại GitHub → Settings → Developer settings → Personal access tokens.

### 11.3. Các lệnh hay dùng

| Lệnh | Tác dụng | Ví dụ |
|------|----------|-------|
| `travis lint` | Kiểm tra cú pháp `.travis.yml` | `travis lint --com` |
| `travis encrypt` | Mã hóa biến môi trường | `travis encrypt "TOKEN=abc" --com --add` |
| `travis encrypt-file` | Mã hóa file | `travis encrypt-file secret.json --com --add` |
| `travis show` | Xem build gần nhất của repo | `travis show --com` |
| `travis logs` | Xem log build | `travis logs --com -b 123` |
| `travis restart` | Chạy lại build | `travis restart 123 --com` |
| `travis cancel` | Hủy build đang chạy | `travis cancel 123 --com` |
| `travis status` | Trạng thái repo hiện tại | `travis status --com` |
| `travis branches` | Liệt kê branch + trạng thái | `travis branches --com` |
| `travis pubkey` | Lấy public key của repo (để tự mã hóa) | `travis pubkey --com -r user/repo` |
| `travis settings` | Xem/sửa settings repo | `travis settings --com` |
| `travis env` | Quản lý env var trên web UI | `travis env set FOO=bar --com` |
| `travis repos` | Liệt kê repo của bạn | `travis repos --com` |
| `travis open` | Mở trang repo trên trình duyệt | `travis open --com` |

### 11.4. `travis lint` — kiểm tra file cấu hình

```bash
cd my-project
travis lint --com
# Happy to report: mọi thứ ổn → "Hooray, .travis.yml looks valid :)"

# Nếu có lỗi, ví dụ:
#   [x] missing key "language", defaulting to "ruby"
#   [x] unexpected key "nodejs", did you mean "node_js"?
```

> 💡 `travis lint` chỉ kiểm tra **cú pháp/khóa hợp lệ**, không kiểm tra logic (ví dụ: nó không biết `npm run test:unit` có tồn tại trong `package.json` không). Luôn kết hợp với một build thử trên nhánh test.

### 11.5. `travis encrypt` nâng cao

```bash
# Mã hóa cho repo khác (không cần cd vào repo đó)
travis encrypt "TOKEN=abc" --com -r gnoudisme/my-project --add

# Mã hóa nhiều biến một lúc (mỗi biến một dòng secure riêng)
travis encrypt "USER=deploy" --com --add env.global
travis encrypt "PASS=s3cr3t" --com --add env.global

# Mã hóa và in ra (không tự sửa file) — để paste tay
travis encrypt "TOKEN=abc" --com
# "secure: \"aBcDeFg...\""

# Mã hóa chuỗi đã có sẵn trong file (vd: mã hóa lại sau khi migrate .org → .com)
travis encrypt --com --add env.global --split < secrets.txt
```

### 11.6. Xem log và debug từ terminal

```bash
# Xem log build mới nhất
travis logs --com

# Xem log của job cụ thể trong build số 123
travis logs --com -b 123 -j 123.2

# Theo dõi log trực tiếp (tail -f) khi build đang chạy
travis logs --com --tail

# Restart job bị fail do lỗi mạng (flaky)
travis restart 123.2 --com
```

### 11.7. Debug build bằng SSH (gói trả phí)

Với gói trả phí, bạn có thể bật **debug mode** để SSH vào đúng VM đang build:

```yaml
# Thêm vào .travis.yml (tạm thời, nhớ xóa sau khi debug xong)
env:
  global:
    - TRAVIS_DEBUG=true
```

Hoặc restart build ở chế độ debug từ web UI: vào build → **Restart build** → chọn **Debug build**. Travis hiện lệnh SSH trong log — bạn có ~30 phút để vào VM và chạy tay từng lệnh.

---

## 12. Thay đổi pricing: từ free sang credits

Đây là chương quan trọng nhất về mặt **chi phí** — và là lý do chính khiến cộng đồng open source rời Travis CI sau 2020.

### 12.1. Tóm tắt lịch sử pricing

| Giai đoạn | Chính sách |
|-----------|------------|
| 2011–2020 (travis-ci.org) | **Miễn phí không giới hạn** cho public repo. Đây là lý do Travis phủ sóng mọi README open source |
| 2014–2020 (travis-ci.com) | Private repo trả phí theo gói (số build đồng thời) |
| **Tháng 11/2020** | Công bố: bỏ free không giới hạn, chuyển sang **mô hình credits** cho TẤT CẢ (kể cả open source). Mỗi tài khoản mới được tặng một lượng trial credits |
| 31/05/2021 | travis-ci.org ngừng nhận build mới — ép migrate sang .com (tức là ép vào mô hình credits) |
| Hiện tại | Trial credits cho tài khoản mới; hết credits → mua thêm hoặc xin OSS credits |

### 12.2. Mô hình credits hoạt động như thế nào?

- Mỗi phút build tiêu tốn một lượng **credits** tùy loại worker:
  - **Linux**: 10 credits/phút (mức cơ sở)
  - **macOS**: 50 credits/phút (đắt gấp 5 lần — vì Apple hardware)
  - **Windows**: 20 credits/phút (gấp 2 lần)
- Ví dụ: build 10 phút trên Linux tốn 100 credits; trên macOS tốn 500 credits.
- Credits **không rollover**: gói tháng nào dùng tháng đó (tùy gói).
- Hết credits → build xếp hàng ở trạng thái **"queued" mãi mãi** cho đến khi bạn mua thêm. Đây là lỗi nhiều người mới gặp và tưởng "Travis bị treo".

### 12.3. Open Source credits — xin thế nào?

Travis CI vẫn có chính sách hỗ trợ open source: dự án open source **phi thương mại** có thể xin cấp credits định kỳ.

**Điều kiện (theo chính sách công bố):**
1. Repo **public** trên GitHub.
2. Dự án **không** phục vụ mục đích thương mại trực tiếp.
3. Có **lịch sử hoạt động**: không phải repo mới tạo để "đào" credits.
4. Đã dùng hết trial credits ban đầu.

**Cách xin:**
1. Gửi email đến **support@travis-ci.com** với tiêu đề kiểu: `[OSS Credits] Request for <username/repo>`.
2. Nội dung: giới thiệu dự án, link repo, giải thích vì sao cần CI, ước tính lượng build/tháng.
3. Chờ phản hồi (thường vài ngày làm việc). Nếu được duyệt, credits được cộng vào tài khoản theo đợt.

> 💡 **Mẹo thực tế:** Nếu dự án của bạn là open source thuần túy và chưa bị ràng buộc gì với Travis, hãy cân nhắc **GitHub Actions** — public repo được chạy free không giới hạn phút, không cần xin xỏ. Chỉ ở lại Travis khi có lý do cụ thể (pipeline legacy, deploy provider đặc thù...).

### 12.4. Tối ưu chi phí credits

| Kỹ thuật | Tiết kiệm |
|----------|-----------|
| Dùng `cache` (chương 8) để build nhanh hơn | Giảm trực tiếp số phút build |
| `[skip ci]` cho commit docs/README | Không tốn build vô ích |
| Giới hạn `branches.only` (không build mọi nhánh linh tinh) | Tránh build rác |
| Thu gọn build matrix (bỏ OS/version không cần thiết) | Giảm số job × số phút |
| Ưu tiên `linux` thay vì `osx`/`windows` khi có thể | macOS đắt gấp 5 lần |
| `fast_finish: true` + `allow_failures` cho job thử nghiệm | Không chờ job không bắt buộc |
| Gộp lint + unit test vào một job thay vì tách nhiều job | Giảm overhead khởi động VM mỗi job (~30–60s/job) |
| Dùng `git: depth: 10` (shallow clone) | Clone nhanh hơn |

### 12.5. Kiểm tra số dư credits

- Web UI: avatar → **Plan** → xem **Credits balance** và lịch sử tiêu thụ theo repo.
- Khi credits sắp hết, Travis gửi email cảnh báo. Đừng để về 0 giữa lúc đang release!

---

## 13. Ví dụ thực tế

### 13.1. Ví dụ 1: Thư viện Node.js — test nhiều version + tự động publish lên npm

**Kịch bản:** Bạn viết thư viện `chuan-hoa-tieng-viet` (chuẩn hóa tiếng Việt: bỏ dấu, chuẩn hóa khoảng trắng...). Mỗi lần push tag `v*`, Travis test trên Node 18/20/22, pass thì tự publish lên npm.

**Cấu trúc repo:**
```
chuan-hoa-tieng-viet/
├── .travis.yml
├── package.json
├── src/
│   └── index.js
└── test/
    └── index.test.js
```

**`package.json`:**
```json
{
  "name": "chuan-hoa-tieng-viet",
  "version": "1.0.0",
  "description": "Thư viện chuẩn hóa chuỗi tiếng Việt",
  "main": "src/index.js",
  "scripts": {
    "lint": "eslint src/ test/",
    "test": "jest --coverage"
  },
  "devDependencies": {
    "eslint": "^8.57.0",
    "jest": "^29.7.0"
  },
  "keywords": ["vietnamese", "normalize", "unicode"],
  "license": "MIT"
}
```

**`.travis.yml`:**
```yaml
language: node_js
node_js:
  - "18"
  - "20"
  - "22"

cache: npm

install: npm ci

script:
  - npm run lint
  - npm test

after_success:
  - npx codecov --token $CODECOV_TOKEN

# Chỉ publish 1 lần (tránh publish 3 lần cho 3 version Node!)
jobs:
  include:
    - stage: test
      node_js: "18"
      script: npm test
    - stage: test
      node_js: "20"
      script: npm test
    - stage: test
      node_js: "22"
      script: npm test
    - stage: release
      node_js: "20"
      script: npm test
      deploy:
        provider: npm
        email: dev@vi-du.vn
        api_key:
          secure: "NPM_TOKEN_DA_MA_HOA"
        on:
          tags: true

# Đảm bảo stage release chỉ chạy sau khi test xong
stages:
  - test
  - name: release
    if: tag IS present
```

**Quy trình release:**
```bash
# 1. Mã hóa npm token (chỉ làm 1 lần)
travis encrypt "NPM_TOKEN=npm_xxx" --com   # rồi paste vào file

# 2. Mỗi lần release:
npm version minor          # 1.0.0 → 1.1.0, tự tạo tag v1.1.0
git push --follow-tags

# 3. Travis: test 3 version Node → pass → publish v1.1.0 lên npm
# Kiểm tra: npm view chuan-hoa-tieng-viet version
```

> ⚠️ **Bẫy thường gặp:** nếu để `deploy` ở cấp root với matrix 3 version, Travis sẽ publish **3 lần** (mỗi job deploy một lần). Cách fix: dùng `stages` như trên, hoặc giới hạn deploy chỉ chạy trên 1 version:
```yaml
deploy:
  provider: npm
  on:
    tags: true
    node_js: "20"      # chỉ job Node 20 được deploy
```

### 13.2. Ví dụ 2: Python package — test với tox + publish lên PyPI

**Kịch bản:** Package `tinhtien-dien` — tính tiền điện theo bậc thang EVN. Test trên Python 3.10–3.12 với tox, publish lên PyPI khi có tag.

**`.travis.yml`:**
```yaml
language: python
python:
  - "3.10"
  - "3.11"
  - "3.12"

cache: pip

install:
  - pip install -U pip tox

script: tox

deploy:
  provider: pypi
  username: __token__
  password:
    secure: "PYPI_TOKEN_DA_MA_HOA"   # token dạng pypi-AgEI...
  distributions: "sdist bdist_wheel"
  on:
    tags: true
    python: "3.11"                    # chỉ publish 1 lần
```

**`tox.ini`:**
```ini
[tox]
envlist = py310, py311, py312

[testenv]
deps =
    pytest
    pytest-cov
commands =
    pytest --cov=tinhtien_dien tests/
```

**`setup.cfg` (metadata PyPI):**
```ini
[metadata]
name = tinhtien-dien
version = 1.0.0
description = Tính tiền điện sinh hoạt theo bậc thang EVN
long_description = file: README.md
long_description_content_type = text/markdown
author = Ten Cua Ban
license = MIT
classifiers =
    Programming Language :: Python :: 3
    License :: OSI Approved :: MIT License

[options]
packages = find:
python_requires = >=3.10
```

**Release:**
```bash
# Tạo PyPI token: pypi.org → Account settings → API tokens → Add token
travis encrypt "PYPI_TOKEN=pypi-xxx" --com   # paste vào .travis.yml

git tag v1.0.0
git push --tags
# → Travis test 3 version Python → publish lên PyPI
pip install tinhtien-dien   # kiểm tra
```

### 13.3. Ví dụ 3: Deploy tài liệu lên GitHub Pages

**Kịch bản:** Dự án `quan-ly-thu-vien` có tài liệu viết bằng **MkDocs** (Markdown → static site). Mỗi lần push lên `main`, Travis build docs và đẩy lên nhánh `gh-pages`.

**`.travis.yml`:**
```yaml
language: python
python: "3.11"

# Không cần test code ở đây — repo docs riêng, hoặc docs/ trong monorepo
install:
  - pip install mkdocs mkdocs-material

script:
  # Build thử để chắc chắn không lỗi Markdown
  - mkdocs build --strict    # --strict: warning cũng tính là fail

deploy:
  provider: pages
  github_token:
    secure: "GITHUB_TOKEN_DA_MA_HOA"  # token với quyền public_repo (repo public)
  local_dir: site                     # output mặc định của mkdocs build
  target_branch: gh-pages
  on:
    branch: main
```

**`mkdocs.yml`:**
```yaml
site_name: "Tài liệu Quản Lý Thư Viện"
site_url: https://gnoudisme.github.io/quan-ly-thu-vien/
theme:
  name: material
  language: vi
  palette:
    - scheme: default
      primary: teal
      accent: amber
nav:
  - Trang chủ: index.md
  - Hướng dẫn cài đặt: cai-dat.md
  - API: api.md
```

**Bật GitHub Pages:** GitHub repo → Settings → Pages → Source: **Deploy from a branch** → Branch: `gh-pages` → Save. Sau lần push đầu tiên, docs có tại `https://gnoudisme.github.io/quan-ly-thu-vien/`.

**Biến thể với Sphinx** (chuẩn Python):
```yaml
install:
  - pip install sphinx sphinx-rtd-theme
script:
  - cd docs && make html SPHINXOPTS="-W" && cd ..   # -W: warning → error
deploy:
  provider: pages
  github_token:
    secure: "GITHUB_TOKEN_DA_MA_HOA"
  local_dir: docs/_build/html
  on:
    branch: main
```

---

## 14. Troubleshooting — lỗi thường gặp

### 14.1. Bảng tra lỗi nhanh

| Triệu chứng | Nguyên nhân thường gặp | Cách fix |
|-------------|------------------------|----------|
| Build **không chạy** sau khi push | Webhook GitHub chưa được tạo / bị revoke | Travis repo Settings → kiểm tra; GitHub repo → Settings → Webhooks → thêm lại `https://api.travis-ci.com` |
| Build **queued mãi** không chạy | Hết credits | Avatar → Plan → kiểm tra Credits balance; mua thêm hoặc xin OSS credits |
| `Could not find .travis.yml` | File không ở root, sai tên (`.travis.yaml`), hoặc chưa push | Đặt đúng `/​.travis.yml` ở thư mục gốc, push lên |
| Lỗi YAML: `mapping values are not allowed` | Thụt đầu dòng sai, thiếu dấu `:` | Chạy `travis lint`, kiểm tra indent (2 spaces) |
| `npm ERR!` / `pip install` fail | Cache hỏng, version dependency trôi | Xóa cache (Settings → Caches → Delete); ghim version trong `package.json`/`requirements.txt` |
| Test pass local, fail trên CI | Khác version runtime; thiếu env var; timezone/locale khác | Log version (`node --version`); khai báo env; set `TZ: Asia/Ho_Chi_Minh`, `LANG: C.UTF-8` |
| `secure:` không giải mã được | Encrypt bằng key của `.org` nhưng build trên `.com` (hoặc ngược lại) | Encrypt lại với `--com` (xem 1.3) |
| Deploy fail: `invalid credentials` | Token hết hạn / sai quyền | Tạo token mới, encrypt lại; kiểm tra scope (npm: Automation; GitHub: repo) |
| PR từ fork không có secret | Travis cố tình chặn (bảo mật) | Đúng hành vi — chỉ deploy từ nhánh chính, không deploy từ PR fork |
| `sudo: command not found` | Môi trường container cũ không có sudo | Thêm `sudo: required` (môi trường cũ) hoặc chuyển sang `dist: focal`+ |
| Build quá 10 phút không log → bị kill | Lệnh treo chờ input (interactive) | Thêm flag non-interactive: `apt-get -y`, `composer --no-interaction`, `mvn -B` |
| Job `osx` fail, `linux` pass | Lệnh Linux-only (apt-get), path khác nhau | Tách lệnh theo OS: `if [[ "$TRAVIS_OS_NAME" == "osx" ]]; then brew install ...; fi` |
| `git clone` chậm / fail | Repo lớn, full history | `git: depth: 10` (shallow clone) |
| Badge hiện "unknown" | Sai `user/repo`, repo private, hoặc chưa có build nào | Kiểm tra URL badge; chạy 1 build thành công trước |

### 14.2. Debug theo từng bước

**Bước 1 — Đọc log từ dưới lên:** Lỗi thực sự thường nằm ở 20–50 dòng cuối của phase đỏ. Đừng đọc từ đầu.

**Bước 2 — Xác định phase nào đỏ:**
- Đỏ ở `install` → vấn đề dependencies/mạng.
- Đỏ ở `script` → vấn đề code/test.
- Đỏ ở `deploy` → vấn đề credentials/cấu hình deploy (code vẫn pass).

**Bước 3 — Tái hiện local bằng Docker:** Chạy đúng image Ubuntu mà Travis dùng:
```bash
docker run -it --rm ubuntu:focal bash
# trong container: cài runtime, clone repo, chạy tay từng lệnh trong .travis.yml
```

**Bước 4 — Bật debug chi tiết:**
```yaml
# Thêm tạm vào .travis.yml
before_install:
  - node --version; npm --version; env | sort   # in version + env để đối chiếu
script:
  - npm test -- --verbose
```

**Bước 5 — Restart job flaky:** Lỗi mạng/test chập chờn → **Restart job** (không cần push commit mới). Nếu fail >2 lần liên tiếp thì không còn là flaky — phải fix thật.

### 14.3. Các "bẫy" kinh điển của Travis CI

1. **Bẫy `npm install` vs `npm ci`**: `npm install` có thể tự cập nhật `package-lock.json` trên CI → build không lặp lại được. Luôn dùng `npm ci` trên CI.
2. **Bẫy timezone**: Server Travis dùng UTC. Test nào so sánh giờ giấc mà không set TZ sẽ fail lặt vặt:
   ```yaml
   env:
     global:
       - TZ=Asia/Ho_Chi_Minh
   ```
3. **Bẫy `language: generic` + cache**: `generic` không có cache mặc định — phải khai báo `cache.directories` tay.
4. **Bẫy deploy chạy nhiều lần**: matrix N job → deploy chạy N lần (đã nói ở 13.1). Fix bằng `stages` hoặc `on: { <version>: ... }`.
5. **Bẫy `[skip ci]` trong message merge**: GitHub tạo merge commit với message mặc định không có `[skip ci]` — nếu muốn skip phải sửa message lúc merge.

---

## 15. Best practices

### 15.1. Viết `.travis.yml` tốt

1. **Fail nhanh (fail fast):** đặt lint và unit test **trước** integration test nặng. Lỗi style thì dừng ngay, đừng tốn 10 phút chạy E2E rồi mới báo.
   ```yaml
   script:
     - npm run lint        # nhanh: 10s
     - npm run test:unit   # vừa: 2 phút
     - npm run test:e2e    # chậm: 8 phút
   ```
2. **Một nguồn sự thật cho version:** đọc version từ `.nvmrc` / `.python-version` / `go.mod` thay vì ghi cứng 2 nơi.
3. **Ghim version dependencies** (`package-lock.json`, `requirements.txt` có `==`, `Gemfile.lock`) — build hôm nay và 6 tháng sau phải giống nhau.
4. **Tách file khi quá dài:** logic phức tạp cho vào `scripts/ci-*.sh`, `.travis.yml` chỉ gọi script — dễ test local.
   ```yaml
   script: bash scripts/ci-test.sh
   ```

### 15.2. Bảo mật

5. **Không bao giờ commit secret plain text** — kể cả "chỉ test tạm rồi xóa sau" (git history không quên).
6. Dùng **token với quyền tối thiểu** (least privilege): npm Automation token, GitHub token chỉ scope `public_repo` nếu đủ, Docker access token thay password.
7. **Xoay token định kỳ** (3–6 tháng) và ngay khi nghi lộ.
8. Fork PR không có secret là **tính năng**, không phải bug — thiết kế pipeline xoay quanh điều đó.

### 15.3. Tốc độ & chi phí

9. **Bật cache** cho mọi project (chương 8) — tiết kiệm cả thời gian lẫn credits.
10. **Thu gọn matrix**: chỉ test version LTS + mới nhất; `osx`/`windows` chỉ khi thực sự cần.
11. Dùng `[skip ci]` cho commit docs/README.
12. Giới hạn `branches.only` — đừng build mọi nhánh feature linh tinh.

### 15.4. Quy trình team

13. **Bắt buộc CI xanh mới được merge:** GitHub repo → Settings → Branches → Branch protection rule → tick **Require status checks to pass** → chọn `continuous-integration/travis-ci`. Không ai (kể cả lead) được merge khi đỏ.
14. **Badge trên README** — vừa là quảng cáo "dự án này có CI", vừa là link nhanh tới trang build.
15. **Thông báo đúng nơi:** Slack cho team, email `on_failure: always` cho maintainer. Đừng spam `on_success: always` — người ta sẽ tắt thông báo và bỏ lỡ lúc fail thật.
16. **Tài liệu hóa pipeline:** ghi chú trong README/CONTRIBUTING: "mỗi PR đều chạy CI gồm lint + test; xem `.travis.yml` để biết chi tiết".

---

## 16. Cheat sheet

### 16.1. Mẫu `.travis.yml` copy-paste

**Node.js tối giản:**
```yaml
language: node_js
node_js: "20"
cache: npm
install: npm ci
script: npm test
```

**Python tối giản:**
```yaml
language: python
python: "3.11"
cache: pip
install: pip install -r requirements.txt
script: pytest
```

**Go tối giản:**
```yaml
language: go
go: "1.22.x"
script:
  - go vet ./...
  - go test ./...
```

**Rust tối giản:**
```yaml
language: rust
rust: stable
cache: cargo
script:
  - cargo build --verbose
  - cargo test --verbose
```

**Đa version + deploy npm khi có tag:**
```yaml
language: node_js
node_js: ["18", "20", "22"]
cache: npm
install: npm ci
script: npm test
deploy:
  provider: npm
  email: dev@vi-du.vn
  api_key: { secure: "PASTE_SAU_KHI_ENCRYPT" }
  on: { tags: true, node_js: "20" }
```

### 16.2. Lệnh Travis CLI hay dùng

```bash
travis login --com --github-token <TOKEN>   # đăng nhập
travis lint --com                           # kiểm tra .travis.yml
travis encrypt "KEY=val" --com --add        # mã hóa + chèn vào file
travis encrypt-file secret.json --com --add # mã hóa file
travis logs --com --tail                    # xem log trực tiếp
travis restart <build#> --com               # chạy lại build
travis status --com                         # trạng thái repo
```

### 16.3. Biến môi trường có sẵn trong build

| Biến | Giá trị | Dùng để làm gì |
|------|---------|----------------|
| `TRAVIS_BRANCH` | Tên nhánh (hoặc tag) đang build | Nhánh hiện tại |
| `TRAVIS_TAG` | Tên tag (rỗng nếu không phải tag build) | Version release |
| `TRAVIS_PULL_REQUEST` | Số PR, hoặc `"false"` | Phân biệt PR vs push thường |
| `TRAVIS_COMMIT` | SHA commit đang build | Gắn version |
| `TRAVIS_BUILD_NUMBER` | Số thứ tự build (1, 2, 3...) | Đánh số artifact |
| `TRAVIS_JOB_NUMBER` | Số job (`123.1`, `123.2`...) | Phân biệt job trong matrix |
| `TRAVIS_OS_NAME` | `linux` / `osx` / `windows` | Lệnh theo OS |
| `TRAVIS_LANGUAGE` | `node_js`, `python`... | Ngôn ngữ của job |
| `TRAVIS_BUILD_DIR` | Đường dẫn thư mục code | `cd` tuyệt đối |
| `CI` | `true` | Phát hiện "đang chạy trên CI" trong script |

**Pattern hay dùng:**
```bash
# Chỉ chạy khi KHÔNG phải PR (vd: deploy, push coverage)
if [ "$TRAVIS_PULL_REQUEST" = "false" ]; then
  npx codecov
fi

# Lấy version từ tag, fallback sang commit ngắn
VERSION="${TRAVIS_TAG:-$(echo $TRAVIS_COMMIT | cut -c1-7)}"
```

### 16.4. Webhook & API nhanh

```bash
# Trigger build mới bằng API (cần Travis API token)
curl -s -X POST \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Travis-API-Version: 3" \
  -H "Authorization: token <TRAVIS_API_TOKEN>" \
  -d '{"request": {"branch": "main"}}' \
  https://api.travis-ci.com/repo/gnoudisme%2Fmy-project/requests
```

### 16.5. Checklist trước khi push

- [ ] `travis lint` báo valid
- [ ] Không có secret plain text trong file
- [ ] `install` chạy được cả khi không có cache
- [ ] Matrix không bùng nổ tổ hợp (đếm số job)
- [ ] Deploy giới hạn đúng 1 job / đúng nhánh / đúng tag
- [ ] `[skip ci]` nếu chỉ sửa docs
- [ ] Badge đã có trên README (nếu là lần setup đầu)

---

*Hết tài liệu. Chúc bạn build xanh! 🟢*
