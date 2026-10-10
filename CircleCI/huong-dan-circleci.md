# Hướng Dẫn Toàn Tập: CircleCI

> **Cấp độ:** Người mới bắt đầu → setup CI/CD được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật lần cuối:** 10/10/2026

---

## Mục Lục

1. [CircleCI là gì? CI/CD là gì?](#1-circleci-là-gì-cicd-là-gì)
2. [So sánh CircleCI với GitHub Actions, Jenkins](#2-so-sánh-circleci-với-github-actions-jenkins)
3. [Tạo tài khoản & kết nối GitHub](#3-tạo-tài-khoản--kết-nối-github)
4. [Pipeline đầu tiên trong 5 phút](#4-pipeline-đầu-tiên-trong-5-phút)
5. [Cấu trúc file config.yml](#5-cấu-trúc-file-configyml)
6. [Jobs — đơn vị công việc](#6-jobs--đơn-vị-công-việc)
7. [Executors: Docker, Machine, macOS](#7-executors-docker-machine-macos)
8. [Steps — từng bước trong job](#8-steps--từng-bước-trong-job)
9. [Workflows — điều phối nhiều job](#9-workflows--điều-phối-nhiều-job)
10. [Orbs — package tái sử dụng](#10-orbs--package-tái-sử-dụng)
11. [Caching — tăng tốc build](#11-caching--tăng-tốc-build)
12. [Biến môi trường & Contexts](#12-biến-môi-trường--contexts)
13. [Artifacts & Test results](#13-artifacts--test-results)
14. [Chạy test song song (Parallelism)](#14-chạy-test-song-song-parallelism)
15. [Deploy tự động](#15-deploy-tự-động)
16. [Ví dụ thực tế theo ngôn ngữ](#16-ví-dụ-thực-tế-theo-ngôn-ngữ)
17. [CircleCI CLI — chạy local](#17-circleci-cli--chạy-local)
18. [Insights & tối ưu](#18-insights--tối-ưu)
19. [Pricing](#19-pricing)
20. [Troubleshooting](#20-troubleshooting)
21. [Best practices & Cheat sheet](#21-best-practices--cheat-sheet)

---

## 1. CircleCI Là Gì? CI/CD Là Gì?

### 1.1. CI/CD giải thích đơn giản

- **CI (Continuous Integration):** mỗi lần bạn push code, máy tự động **chạy test** — biết ngay code có làm hỏng gì không, thay vì đợi đến lúc release mới phát hiện.
- **CD (Continuous Delivery/Deployment):** test xong → tự động **deploy** lên staging/production.

Không CI/CD:
```
Dev push code → ... im lặng ... → cuối tuần deploy → 💥 sập production
→ cả team debug cả đêm
```

Có CI/CD:
```
Dev push code → CircleCI chạy test (5 phút) → ❌ báo lỗi ngay
→ dev fix ngay khi còn nóng → merge an toàn
```

### 1.2. CircleCI là gì?

**CircleCI** là nền tảng CI/CD chạy trên cloud — bạn viết file cấu hình `.circleci/config.yml`, mỗi lần push code là nó tự chạy pipeline (test, build, deploy).

Ra đời 2011, là một trong những CI/CD lâu đời và ổn định nhất. Được dùng bởi **Spotify, Facebook, Coinbase, Kickstarter**.

### 1.3. Thuật ngữ cơ bản

| Thuật ngữ | Nghĩa |
|-----------|-------|
| **Pipeline** | Toàn bộ quy trình chạy 1 lần (khi push code / mở PR) |
| **Workflow** | Cách sắp xếp các job (chạy song song / nối tiếp) |
| **Job** | 1 đơn vị công việc (ví dụ: `test`, `build`, `deploy`) |
| **Step** | Từng bước trong job (ví dụ: checkout code, chạy `npm test`) |
| **Executor** | Môi trường chạy job (Docker container, máy ảo...) |
| **Orb** | Package cấu hình tái sử dụng (như npm package cho CI) |

---

## 2. So Sánh CircleCI Với GitHub Actions, Jenkins

| Tiêu chí | CircleCI | GitHub Actions | Jenkins |
|----------|----------|----------------|---------|
| Nơi chạy | Cloud CircleCI | Cloud GitHub | **Tự host** (máy của bạn) |
| Cài đặt | Đăng nhập GitHub là xong | Có sẵn trong repo | Cài server, config phức tạp |
| Free tier | 6,000 build minutes/tháng | 2,000 phút/tháng | Miễn phí (tốn tiền server) |
| Tốc độ | Nhanh, cache tốt | Nhanh | Tùy máy bạn |
| Docker layer caching | Có (gói trả phí) | Có (hạn chế) | Tự config |
| macOS build | ✅ (iOS app) | ✅ | Tự setup Mac |
| Độ phức tạp | Trung bình | Dễ (nếu đã dùng GitHub) | Cao |
| Dùng khi nào | Team cần CI mạnh, ổn định | Project trên GitHub, muốn đơn giản | Công ty cần tự host, custom sâu |

**Khi nào chọn CircleCI:**
- Muốn CI/CD chuyên nghiệp, config 1 lần chạy ổn định
- Cần build iOS (macOS executor)
- Team đã quen, cần insights chi tiết

**Khi nào chọn GitHub Actions:**
- Code đã trên GitHub, muốn "all in one"
- Muốn free đơn giản, marketplace actions khổng lồ

---

## 3. Tạo Tài Khoản & Kết Nối GitHub

### 3.1. Đăng ký

1. Vào https://circleci.com → **Start Building for Free**
2. Chọn **Sign up with GitHub** (hoặc Bitbucket)
3. Cấp quyền cho CircleCI đọc repo
4. Xong!

### 3.2. Thêm project đầu tiên

1. Dashboard → **Projects** → tìm repo của bạn
2. Bấm **Set Up Project**
3. Chọn **Fastest** (dùng config mẫu) hoặc **Set up manually**
4. Nếu repo chưa có `.circleci/config.yml` → CircleCI gợi ý commit file mẫu

### 3.3. Cấu trúc repo sau khi setup

```
my-app/
├── .circleci/
│   └── config.yml    ← file cấu hình CI/CD
├── src/
├── package.json
└── ...
```

---

## 4. Pipeline Đầu Tiên Trong 5 Phút

File `.circleci/config.yml` đơn giản nhất — chạy test mỗi lần push:

```yaml
version: 2.1

jobs:
  test:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout                    # tải code về
      - run:
          name: Cài dependencies
          command: npm ci
      - run:
          name: Chạy test
          command: npm test

workflows:
  version: 2
  test-va-build:
    jobs:
      - test
```

Giải thích từng dòng:
- `version: 2.1` — phiên bản cú pháp CircleCI
- `jobs.test` — định nghĩa 1 job tên `test`
- `docker: - image: cimg/node:20.11` — chạy trong container Node 20 (image chính chủ CircleCI)
- `steps` — các bước: checkout code → `npm ci` → `npm test`
- `workflows` — bảo CircleCI chạy job `test` mỗi pipeline

Commit + push → vào dashboard CircleCI → thấy pipeline đang chạy → xanh là pass! 🎉

---

## 5. Cấu Trúc File config.yml

```yaml
version: 2.1              # bắt buộc

orbs:                    # package tái sử dụng (tùy chọn)
  node: circleci/node@5.1.0

executors:               # định nghĩa môi trường chạy (tùy chọn)
  node-executor:
    docker:
      - image: cimg/node:20.11

jobs:                    # BẮT BUỘC: định nghĩa các job
  test:
    executor: node-executor
    steps:
      - checkout
      - node/install-packages:
          pkg-manager: npm
      - run: npm test

  build:
    executor: node-executor
    steps:
      - checkout
      - run: npm run build

workflows:               # BẮT BUỘC: điều phối job
  version: 2
  pipeline-chinh:
    jobs:
      - test
      - build:
          requires:
            - test       # build chỉ chạy khi test pass
```

### 5.1. Quy tắc YAML cần nhớ

```yaml
# Thụt đầu dòng bằng SPACE (2 space), KHÔNG dùng tab!
jobs:
  test:              # 2 space
    docker:          # 4 space
      - image: xxx   # 6 space, "- " cho list

# String có dấu : phải để trong ngoặc
command: "echo hello: world"

# Nhiều dòng dùng |
command: |
  echo "dòng 1"
  echo "dòng 2"
```

> 💡 Lỗi phổ biến nhất của người mới: **sai thụt đầu dòng YAML**. Dùng https://yamlchecker.com để kiểm tra.

---

## 6. Jobs — Đơn Vị Công Việc

### 6.1. Job cơ bản đầy đủ

```yaml
jobs:
  kiem-tra-code:
    docker:
      - image: cimg/node:20.11
    working_directory: ~/app          # thư mục làm việc (mặc định ~/project)
    steps:
      - checkout                       # bước 1: tải code
      - run:                           # bước 2
          name: Cài đặt                 # tên hiển thị
          command: npm ci
      - run:
          name: Lint
          command: npm run lint
      - run:
          name: Test
          command: npm test
```

### 6.2. Job với nhiều Docker image (ví dụ: app + database)

```yaml
jobs:
  test-api:
    docker:
      - image: cimg/node:20.11
      - image: cimg/postgres:16.2
        environment:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test123
          POSTGRES_DB: testdb
    steps:
      - checkout
      - run: npm ci
      - run:
          name: Đợi Postgres sẵn sàng
          command: dockerize -wait tcp://localhost:5432 -timeout 30s
      - run:
          name: Chạy test API
          command: npm test
          environment:
            DATABASE_URL: postgres://test:test123@localhost:5432/testdb
```

### 6.3. Chia job hợp lý

```
❌ 1 job làm tất cả (lint + test + build + deploy) — chậm, khó debug
✅ Mỗi job 1 việc:
   - lint      (2 phút)
   - test-unit (5 phút)
   - test-e2e  (10 phút, chạy song song với test-unit)
   - build     (chạy sau khi test pass)
   - deploy    (chạy sau khi build pass, chỉ ở branch main)
```

---

## 7. Executors: Docker, Machine, macOS

### 7.1. Docker executor (dùng nhiều nhất)

Chạy job trong Docker container — nhanh, nhẹ.

```yaml
jobs:
  test:
    docker:
      - image: cimg/node:20.11       # image chính chủ CircleCI
    steps:
      - checkout
      - run: npm test
```

**Image `cimg/*` nên dùng** (do CircleCI maintain, cập nhật đều):

| Image | Dùng cho |
|-------|----------|
| `cimg/node:20.11` | Node.js |
| `cimg/python:3.12` | Python |
| `cimg/go:1.22` | Go |
| `cimg/openjdk:17.0` | Java |
| `cimg/ruby:3.3` | Ruby |
| `cimg/postgres:16.2` | Database phụ |

### 7.2. Machine executor (cần Docker trong Docker)

Khi cần chạy `docker build` trong CI:

```yaml
jobs:
  build-image:
    machine:
      image: ubuntu-2204:current
    steps:
      - checkout
      - run: docker build -t my-app .
      - run: docker run my-app npm test
```

### 7.3. macOS executor (build iOS)

```yaml
jobs:
  build-ios:
    macos:
      xcode: 15.2.0
    steps:
      - checkout
      - run: bundle install
      - run: bundle exec fastlane test
```

> 💡 macOS executor tốn credits nhiều hơn Linux — chỉ dùng khi cần build iOS/macOS.

### 7.4. So sánh

| Executor | Tốc độ khởi động | Dùng khi nào |
|----------|-----------------|--------------|
| Docker | ~10 giây | Hầu hết mọi thứ |
| Machine | ~30-60 giây | Cần Docker daemon, VM đầy đủ |
| macOS | ~2-5 phút | Build iOS/macOS |
| Windows | ~2-5 phút | Build .NET Windows |

---

## 8. Steps — Từng Bước Trong Job

### 8.1. Các step đặc biệt

```yaml
steps:
  - checkout                    # tải code từ GitHub về

  - run:                        # chạy lệnh (dạng ngắn)
      npm test

  - run:                        # chạy lệnh (dạng đầy đủ)
      name: Chạy test với coverage
      command: npm test -- --coverage
      working_directory: ~/app/packages/frontend

  - setup_remote_docker         # bật Docker daemon (cho machine executor)
```

### 8.2. Step có điều kiện (when)

```yaml
steps:
  - run:
      name: Deploy production
      command: ./deploy.sh
      # Chỉ chạy khi ở branch main
```

Điều kiện branch thường đặt ở workflow (mục 9) thay vì step.

### 8.3. Lưu biến giữa các step

```yaml
steps:
  - run:
      name: Tạo version
      command: echo "export APP_VERSION=1.2.3" >> $BASH_ENV
  - run:
      name: Dùng version
      command: echo "Deploy version $APP_VERSION"
```

`$BASH_ENV` là file đặc biệt — biến ghi vào đó sẽ có ở các step sau.

---

## 9. Workflows — Điều Phối Nhiều Job

### 9.1. Job nối tiếp (requires)

```yaml
workflows:
  version: 2
  build-test-deploy:
    jobs:
      - test
      - build:
          requires:
            - test          # build đợi test xong
      - deploy:
          requires:
            - build         # deploy đợi build xong
          filters:
            branches:
              only: main    # chỉ deploy khi ở branch main
```

Sơ đồ:
```
test → build → deploy (chỉ main)
```

### 9.2. Job song song

```yaml
workflows:
  version: 2
  pipeline:
    jobs:
      - lint
      - test-unit
      - test-e2e
      # cả 3 chạy song song, không đợi nhau
      - build:
          requires:
            - lint
            - test-unit
            - test-e2e    # build đợi cả 3 xong
```

Sơ đồ:
```
lint ─┐
test-unit ─┼→ build → deploy
test-e2e ─┘
```

### 9.3. Filter theo branch và tag

```yaml
workflows:
  version: 2
  pipeline:
    jobs:
      - test:
          filters:
            branches:
              ignore: gh-pages    # không chạy cho branch gh-pages
      - deploy:
          requires: [test]
          filters:
            branches:
              only: main
            tags:
              only: /^v.*/      # hoặc khi push tag v1.0.0
```

### 9.4. Manual approval (duyệt tay trước khi deploy)

```yaml
workflows:
  version: 2
  pipeline:
    jobs:
      - test
      - build:
          requires: [test]
      - giu-de-duyet:           # job đặc biệt: đợi người bấm nút
          type: approval
          requires: [build]
      - deploy-production:
          requires: [giu-de-duyet]
          filters:
            branches:
              only: main
```

Trên dashboard sẽ hiện nút **Approve** — bấm mới deploy tiếp. Hợp cho production quan trọng.

### 9.5. Chạy theo lịch (scheduled pipeline)

**Project Settings** → **Triggers** → **Add Trigger** → chọn:
- Chạy mỗi đêm lúc 2h sáng
- Branch: main

Dùng cho: test định kỳ, dependency check, backup.

Hoặc khai báo trong config (API v2):
```yaml
workflows:
  nightly:
    triggers:
      - schedule:
          cron: "0 2 * * *"
          filters:
            branches:
              only: main
    jobs:
      - test-toan-dien
```

---

## 10. Orbs — Package Tái Sử Dụng

Orbs là package cấu hình có sẵn — thay vì viết 20 dòng, chỉ cần 3 dòng.

### 10.1. Ví dụ: dùng orb Node

```yaml
version: 2.1

orbs:
  node: circleci/node@5.1.0

jobs:
  test:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - node/install-packages:
          pkg-manager: npm        # tự cache node_modules!
      - run: npm test
```

`node/install-packages` tự động: cài đúng version Node, `npm ci`, cache `node_modules`.

### 10.2. Orbs phổ biến

| Orb | Chức năng |
|-----|-----------|
| `circleci/node` | Cài Node, npm/yarn |
| `circleci/python` | Cài Python, pip |
| `circleci/docker` | Build/publish Docker image |
| `circleci/aws-cli` | Cài AWS CLI |
| `circleci/aws-ecr` | Push image lên AWS ECR |
| `circleci/heroku` | Deploy lên Heroku |
| `circleci/slack` | Gửi thông báo Slack |
| `circleci/browser-tools` | Cài Chrome cho test E2E |

### 10.3. Ví dụ: gửi Slack khi deploy xong

```yaml
version: 2.1

orbs:
  slack: circleci/slack@4.12.5

jobs:
  deploy:
    docker:
      - image: cimg/base:current
    steps:
      - checkout
      - run: ./deploy.sh
      - slack/notify:
          event: pass
          template: success_tagged_deploy_1
      - slack/notify:
          event: fail
          template: basic_fail_1

workflows:
  version: 2
  pipeline:
    jobs:
      - deploy:
          context: slack-secrets    # chứa SLACK_ACCESS_TOKEN
```

### 10.4. Tìm orb

Vào https://circleci.com/developer/orbs → tìm theo từ khóa.

---

## 11. Caching — Tăng Tốc Build

Không cache: mỗi lần build cài lại `node_modules` (3-5 phút).
Có cache: dùng lại từ lần trước (30 giây).

### 11.1. Cache dependencies Node.js

```yaml
jobs:
  test:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - restore_cache:                    # bước 1: khôi phục cache
          keys:
            - npm-v1-{{ checksum "package-lock.json" }}
            - npm-v1-                     # fallback: dùng cache gần nhất
      - run: npm ci
      - save_cache:                       # bước 2: lưu cache mới
          key: npm-v1-{{ checksum "package-lock.json" }}
          paths:
            - ~/.npm
            - node_modules
      - run: npm test
```

Logic:
1. `restore_cache` tìm cache có key khớp checksum của `package-lock.json`
2. Nếu `package-lock.json` đổi → checksum đổi → không khớp → cài mới
3. `save_cache` lưu cache với key mới

### 11.2. Cache Python pip

```yaml
- restore_cache:
    keys:
      - pip-v1-{{ checksum "requirements.txt" }}
- run: pip install -r requirements.txt
- save_cache:
    key: pip-v1-{{ checksum "requirements.txt" }}
    paths:
      - ~/.cache/pip
```

### 11.3. Cache Docker layers (gói trả phí)

```yaml
jobs:
  build:
    machine:
      image: ubuntu-2204:current
    steps:
      - checkout
      - setup_remote_docker:
          docker_layer_caching: true    # cần gói Performance trở lên
      - run: docker build -t my-app .
```

---

## 12. Biến Môi Trường & Contexts

### 12.1. Biến môi trường trong config

```yaml
jobs:
  test:
    docker:
      - image: cimg/node:20.11
        environment:
          NODE_ENV: test
          DATABASE_URL: postgres://test:test@localhost/testdb
    environment:
      API_URL: https://staging.api.com
    steps:
      - run: npm test
```

### 12.2. Project settings (cho secret)

**Project Settings** → **Environment Variables** → thêm:
- `AWS_SECRET_KEY`
- `DOCKER_PASSWORD`

Dùng trong config như biến thường: `$AWS_SECRET_KEY`. Giá trị bị che trong log.

### 12.3. Contexts (chia sẻ giữa nhiều project)

Khi có nhiều repo dùng chung secret (ví dụ cùng deploy lên 1 server):

1. **Organization Settings** → **Contexts** → tạo context `production-secrets`
2. Thêm biến vào context
3. Trong `config.yml`:

```yaml
workflows:
  version: 2
  pipeline:
    jobs:
      - deploy:
          context: production-secrets
```

> 🔒 Chỉ admin mới tạo/sửa context. Bảo mật hơn để secret ở project settings.

---

## 13. Artifacts & Test Results

### 13.1. Lưu artifacts (file build ra)

```yaml
jobs:
  build:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - run: npm run build
      - store_artifacts:
          path: dist/               # lưu thư mục dist
          destination: web-build    # tên hiển thị
```

Sau khi chạy xong, vào dashboard → **Artifacts** → tải về hoặc xem trực tiếp.

### 13.2. Lưu kết quả test (hiện đẹp trên dashboard)

```yaml
jobs:
  test:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - run:
          name: Chạy test (xuất JUnit XML)
          command: npm test -- --reporters=jest-junit
      - store_test_results:
          path: test-results/       # CircleCI parse và hiện bảng đẹp
```

### 13.3. Test insights

**Dashboard** → **Insights** → xem:
- Job nào chậm nhất
- Test nào fail nhiều nhất
- Xu hướng thời gian build

---

## 14. Chạy Test Song Song (Parallelism)

Có 1000 test chạy 20 phút? Chia ra 4 container chạy song song → 5 phút!

```yaml
jobs:
  test:
    parallelism: 4                    # chia thành 4 container
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - run: npm ci
      - run:
          name: Chia test đều cho 4 container
          command: |
            TESTS=$(circleci tests glob "tests/**/*.test.js" \
              | circleci tests split --split-by=timings)
            npm test $TESTS
      - store_test_results:
          path: test-results/
```

`circleci tests split --split-by=timings` — chia test dựa trên thời gian chạy lịch sử (đều nhất). Lần đầu chưa có lịch sử thì chia đều theo số lượng.

---

## 15. Deploy Tự Động

### 15.1. Deploy lên Netlify

```yaml
jobs:
  deploy:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - run: npm ci && npm run build
      - run:
          name: Deploy lên Netlify
          command: npx netlify-cli deploy --prod --dir=dist --auth=$NETLIFY_TOKEN --site=$NETLIFY_SITE_ID

workflows:
  version: 2
  pipeline:
    jobs:
      - test
      - deploy:
          requires: [test]
          filters: { branches: { only: main } }
```

### 15.2. Deploy Docker image lên Docker Hub

```yaml
version: 2.1
orbs:
  docker: circleci/docker@2.4.0

workflows:
  version: 2
  build-va-push:
    jobs:
      - docker/build:
          image: username/my-app
          tag: latest,$CIRCLE_SHA1
      - docker/push:
          image: username/my-app
          tag: latest,$CIRCLE_SHA1
          requires: [docker/build]
          filters: { branches: { only: main } }
```

### 15.3. Deploy lên AWS (S3 + CloudFront)

```yaml
version: 2.1
orbs:
  aws-cli: circleci/aws-cli@4.1.0

jobs:
  deploy-s3:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - run: npm ci && npm run build
      - aws-cli/setup
      - run:
          name: Upload lên S3
          command: aws s3 sync dist/ s3://my-bucket --delete
      - run:
          name: Xóa cache CloudFront
          command: |
            aws cloudfront create-invalidation \
              --distribution-id $CF_DISTRIBUTION_ID \
              --paths "/*"
```

### 15.4. Biến môi trường CircleCI có sẵn

| Biến | Ý nghĩa |
|------|---------|
| `$CIRCLE_BRANCH` | Tên branch đang chạy |
| `$CIRCLE_SHA1` | Commit hash |
| `$CIRCLE_TAG` | Tag (nếu chạy từ tag) |
| `$CIRCLE_BUILD_NUM` | Số thứ tự build |
| `$CIRCLE_PULL_REQUEST` | URL PR (nếu có) |
| `$CIRCLE_USERNAME` | Người trigger |

---

## 16. Ví Dụ Thực Tế Theo Ngôn Ngữ

### 16.1. Python (pytest)

```yaml
version: 2.1
orbs:
  python: circleci/python@2.1.1

jobs:
  test:
    docker:
      - image: cimg/python:3.12
    steps:
      - checkout
      - python/install-packages:
          pkg-manager: pip
      - run:
          name: Chạy pytest
          command: pytest --junitxml=test-results/junit.xml
      - store_test_results:
          path: test-results/
```

### 16.2. Go

```yaml
version: 2.1

jobs:
  test:
    docker:
      - image: cimg/go:1.22
    steps:
      - checkout
      - run: go mod download
      - run: go vet ./...
      - run: go test ./... -v

workflows:
  version: 2
  pipeline:
    jobs: [test]
```

### 16.3. Rust

```yaml
version: 2.1

jobs:
  test:
    docker:
      - image: cimg/rust:1.75
    steps:
      - checkout
      - restore_cache:
          keys: [cargo-v1-{{ checksum "Cargo.lock" }}]
      - run: cargo test --all
      - save_cache:
          key: cargo-v1-{{ checksum "Cargo.lock" }}
          paths: ["~/.cargo", "target/"]
```

### 16.4. Flutter (mobile)

```yaml
version: 2.1

jobs:
  test:
    docker:
      - image: cirrusci/flutter:stable
    steps:
      - checkout
      - run: flutter pub get
      - run: flutter analyze
      - run: flutter test
```

### 16.5. Docker đa nền tảng (multi-arch)

```yaml
version: 2.1

jobs:
  build-multiarch:
    machine:
      image: ubuntu-2204:current
    steps:
      - checkout
      - setup_remote_docker
      - run: |
          docker buildx create --use
          docker buildx build --platform linux/amd64,linux/arm64 \
            -t username/app:latest --push .
```

---

## 17. CircleCI CLI — Chạy Local

Test config ở máy local trước khi push — đỡ phải "push rồi fix, push rồi fix".

```bash
# Cài CLI
curl -fLSs https://raw.githubusercontent.com/CircleCI-Public/circleci-cli/master/install.sh | bash

# Kiểm tra config hợp lệ
circleci config validate .circleci/config.yml
# ✅ Config file is valid.

# Chạy job ở local (cần Docker)
circleci local execute --job test
```

> ⚠️ `local execute` không hỗ trợ workflows, orbs một số loại — nhưng đủ để test job đơn.

---

## 18. Insights & Tối Ưu

### 18.1. Xem insights

Dashboard → **Insights** → chọn project:
- **Thời gian chạy trung bình** từng workflow/job
- **Tỷ lệ thành công** — job nào fail nhiều nhất?
- **Credits tiêu thụ** — job nào tốn nhất?

### 18.2. Tối ưu build chậm

| Vấn đề | Cách khắc phục |
|--------|----------------|
| Cài dependencies lâu | Dùng cache (mục 11) |
| Test lâu | Chia parallelism (mục 14) |
| Docker build lâu | Dùng DLC / multi-stage build |
| Job đợi nhau không cần thiết | Cho chạy song song (mục 9.2) |
| Image Docker nặng | Dùng `cimg/*` thay vì image full |

### 18.3. Resource class (tăng CPU/RAM)

```yaml
jobs:
  test-nang:
    docker:
      - image: cimg/node:20.11
    resource_class: large    # 4 CPU, 8GB RAM (mặc định: medium)
    steps:
      - checkout
      - run: npm test
```

| Class | CPU | RAM | Tốn credits |
|-------|-----|-----|-------------|
| small | 1 | 2GB | 5/phút |
| medium (mặc định) | 2 | 4GB | 10/phút |
| large | 4 | 8GB | 20/phút |
| xlarge | 8 | 16GB | 40/phút |

---

## 19. Pricing

### 19.1. Gói Free ($0)

| Tài nguyên | Giới hạn |
|------------|----------|
| Build minutes | 6,000 phút/tháng |
| Users | 5 |
| Docker Layer Caching | ❌ |
| Support | Community |

6,000 phút ≈ **100 giờ build/tháng** — đủ cho team nhỏ/side project.

### 19.2. Gói Performance ($15/user/tháng)

- 20,000+ credits
- Docker Layer Caching
- Insights nâng cao
- Support ưu tiên

### 19.3. Tính credits

Mỗi phút build tốn credits tùy executor:
- Docker `medium`: 10 credits/phút
- Machine `medium`: 10 credits/phút
- macOS `medium`: 50 credits/phút (đắt gấp 5 lần!)

**Mẹo tiết kiệm:**
- Dùng Docker thay vì Machine khi có thể
- Cache dependencies
- Chỉ chạy test cần thiết cho PR (test full ở main)

---

## 20. Troubleshooting

| Vấn đề | Nguyên nhân | Cách khắc phục |
|--------|-------------|----------------|
| `Config file is not valid` | Sai YAML / sai cú pháp | Chạy `circleci config validate` ở local |
| Job stuck ở "Queued" | Hết credits / hết concurrency | Kiểm tra Plan Usage; nâng cấp hoặc đợi |
| `Cannot connect to Docker daemon` | Quên `setup_remote_docker` | Thêm step `setup_remote_docker` |
| Cache không hit | Key sai / checksum đổi | Kiểm tra key, dùng fallback key |
| Test pass local, fail trên CI | Môi trường khác nhau | Dùng đúng image version; kiểm tra biến môi trường |
| `Out of memory` | Job cần RAM lớn | Tăng `resource_class` |
| Deploy chạy nhầm branch | Sai filter | Kiểm tra `filters.branches.only` |
| Secret lộ trong log | Echo biến secret | Không `echo $SECRET`; CircleCI tự che nhưng đừng chủ quan |
| Orb version cũ | Orb đã có bản mới | Vào registry xem version mới nhất |

### Debug bằng SSH

Job fail khó hiểu? SSH vào container đang chạy:

```yaml
jobs:
  test:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - run: npm test
```

1. Vào dashboard → job đang chạy → **Rerun** → **Rerun job with SSH**
2. Copy lệnh SSH hiện ra, chạy ở terminal của bạn
3. Vào trong container debug trực tiếp!

> ⚠️ Nhớ tắt sau khi xong — container SSH tốn credits.

---

## 21. Best Practices & Cheat Sheet

### 21.1. Best practices

1. **Validate config ở local** trước khi push (`circleci config validate`)
2. **Fail fast:** cho lint chạy trước, song song với test
3. **Cache mọi thứ** có thể (dependencies, Docker layers)
4. **Chia job nhỏ** — dễ debug, chạy song song được
5. **Deploy chỉ từ `main`** (dùng filters)
6. **Secret** để ở Project Settings/Contexts, không hardcode
7. **Pin version** image Docker (`cimg/node:20.11` thay vì `cimg/node:latest`)
8. **Dùng orbs** thay vì viết lại từ đầu

### 21.2. Cheat sheet

```yaml
# .circleci/config.yml mẫu đầy đủ
version: 2.1

orbs:
  node: circleci/node@5.1.0
  slack: circleci/slack@4.12.5

jobs:
  lint:
    docker: [{ image: cimg/node:20.11 }]
    steps:
      - checkout
      - node/install-packages: { pkg-manager: npm }
      - run: npm run lint

  test:
    parallelism: 2
    docker: [{ image: cimg/node:20.11 }]
    steps:
      - checkout
      - node/install-packages: { pkg-manager: npm }
      - run:
          command: |
            TESTS=$(circleci tests glob "tests/**/*.js" \
              | circleci tests split --split-by=timings)
            npm test $TESTS
      - store_test_results: { path: test-results/ }

  deploy:
    docker: [{ image: cimg/node:20.11 }]
    steps:
      - checkout
      - run: npm ci && npm run build
      - run: ./deploy.sh

workflows:
  version: 2
  pipeline:
    jobs:
      - lint
      - test
      - deploy:
          requires: [lint, test]
          filters: { branches: { only: main } }
          context: production-secrets
```

```bash
# CLI
circleci config validate .circleci/config.yml
circleci local execute --job test
```

### 21.3. Tài liệu tham khảo

- Docs: https://circleci.com/docs
- Orbs registry: https://circleci.com/developer/orbs
- Config reference: https://circleci.com/docs/configuration-reference
- Cộng đồng: https://discuss.circleci.com

---

*Cập nhật: 10/10/2026*

---

## 22. Matrix Jobs — Chạy Nhiều Phiên Bản

Test trên nhiều version Node/Python cùng lúc:

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
      - run: npm ci && npm test

workflows:
  version: 2
  pipeline:
    jobs:
      - test:
          matrix:
            parameters:
              node-version: ["18.19", "20.11", "21.6"]
```

→ Tạo 3 job song song: test trên Node 18, 20, 21. Biết ngay code có tương thích mọi version không!

### Matrix nhiều chiều

```yaml
- test:
    matrix:
      parameters:
        node-version: ["20.11", "21.6"]
        os: ["linux", "macos"]   # kết hợp: 2x2 = 4 job
```

---

## 23. Pipeline Parameters — Tùy Biến Khi Trigger

```yaml
version: 2.1

parameters:
  chay-test-e2e:
    type: boolean
    default: false
  moi-truong:
    type: string
    default: "staging"

jobs:
  deploy:
    docker:
      - image: cimg/base:current
    steps:
      - run: ./deploy.sh << pipeline.parameters.moi-truong >>

workflows:
  version: 2
  pipeline:
    when: << pipeline.parameters.chay-test-e2e >>
    jobs:
      - test-e2e
    # ...
```

Trigger qua API với parameters:

```bash
curl -X POST https://circleci.com/api/v2/project/gh/user/repo/pipeline \
  -H "Circle-Token: $CIRCLE_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "branch": "main",
    "parameters": {
      "chay-test-e2e": true,
      "moi-truong": "production"
    }
  }'
```

---

## 24. Dynamic Config — Config Sinh Động

Thay vì 1 file config tĩnh, dùng **setup workflow** để sinh config động (ví dụ: chỉ test những package bị thay đổi trong monorepo).

```yaml
# .circleci/config.yml
version: 2.1
setup: true

orbs:
  path-filtering: circleci/path-filtering@1.0.0

workflows:
  version: 2
  setup:
    jobs:
      - path-filtering/filter:
          base-revision: main
          config-path: .circleci/continued-config.yml
          mapping: |
            apps/web/.*    chay-web true
            apps/api/.*    chay-api true
```

```yaml
# .circleci/continued-config.yml
version: 2.1
parameters:
  chay-web: { type: boolean, default: false }
  chay-api: { type: boolean, default: false }

jobs:
  test-web:
    docker: [{ image: cimg/node:20.11 }]
    steps: [checkout, run: npm test]

workflows:
  version: 2
  web:
    when: << pipeline.parameters.chay-web >>
    jobs: [test-web]
  api:
    when: << pipeline.parameters.chay-api >>
    jobs: [test-api]
```

→ Chỉ sửa `apps/web` thì chỉ chạy test web. Tiết kiệm credits!

---

## 25. So Sánh Chi Tiết Với GitHub Actions

| | CircleCI | GitHub Actions |
|---|---|---|
| File config | `.circleci/config.yml` | `.github/workflows/*.yml` |
| Chạy local | `circleci local execute` | `act` (bên thứ 3) |
| Reusable | Orbs | Composite actions / Reusable workflows |
| Debug SSH | ✅ Có sẵn | ❌ (dùng tmate action) |
| Test splitting | ✅ `circleci tests split` | Tự làm |
| Insights | Chi tiết | Cơ bản |
| Free minutes | 6,000 | 2,000 |
| Matrix | Có | Có (mạnh hơn) |

**Migrate từ GitHub Actions sang CircleCI:**

```yaml
# .github/workflows/test.yml (GitHub Actions)
name: Test
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npm test
```

```yaml
# .circleci/config.yml (CircleCI - tương đương)
version: 2.1
jobs:
  test:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - run: npm ci
      - run: npm test
workflows:
  version: 2
  pipeline:
    jobs: [test]
```

---

## 26. Tích Hợp Với Công Cụ Khác

### 26.1. SonarQube (chất lượng code)

```yaml
jobs:
  scan:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - run: npm ci
      - run:
          name: Quét SonarQube
          command: |
            npx sonar-scanner \
              -Dsonar.projectKey=my-project \
              -Dsonar.host.url=$SONAR_URL \
              -Dsonar.login=$SONAR_TOKEN
```

### 26.2. Snyk (quét bảo mật dependencies)

```yaml
version: 2.1
orbs:
  snyk: snyk/snyk@1.7.0

jobs:
  bao-mat:
    docker:
      - image: cimg/node:20.11
    steps:
      - checkout
      - snyk/scan:
          fail-on-issues: true
```

### 26.3. Codecov (báo cáo coverage)

```yaml
- run:
    name: Upload coverage
    command: |
      curl -Os https://uploader.codecov.io/latest/linux/codecov
      chmod +x codecov
      ./codecov -t $CODECOV_TOKEN
```

---

## 27. Ví Dụ Hoàn Chỉnh: Node.js API + Postgres + Deploy

```yaml
version: 2.1

orbs:
  node: circleci/node@5.1.0
  slack: circleci/slack@4.12.5

executors:
  node-pg:
    docker:
      - image: cimg/node:20.11
      - image: cimg/postgres:16.2
        environment:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: app_test

jobs:
  lint:
    executor: node-pg
    steps:
      - checkout
      - node/install-packages: { pkg-manager: npm }
      - run: npm run lint

  test:
    parallelism: 2
    executor: node-pg
    steps:
      - checkout
      - node/install-packages: { pkg-manager: npm }
      - run:
          name: Đợi Postgres
          command: dockerize -wait tcp://localhost:5432 -timeout 30s
      - run:
          name: Migrate + test
          command: |
            TESTS=$(circleci tests glob "tests/**/*.test.js" \
              | circleci tests split)
            npm run migrate:test && npm test $TESTS
          environment:
            DATABASE_URL: postgres://test:test@localhost:5432/app_test
      - store_test_results: { path: test-results/ }

  build-image:
    machine: { image: ubuntu-2204:current }
    steps:
      - checkout
      - setup_remote_docker
      - run: |
          docker build -t myapp:$CIRCLE_SHA1 .
          echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
          docker push myapp:$CIRCLE_SHA1

  deploy-staging:
    docker: [{ image: cimg/base:current }]
    steps:
      - run: ./scripts/deploy.sh staging $CIRCLE_SHA1

workflows:
  version: 2
  ci-cd:
    jobs:
      - lint
      - test
      - build-image:
          requires: [lint, test]
          filters: { branches: { only: [main, develop] } }
      - duyet-production:
          type: approval
          requires: [build-image]
          filters: { branches: { only: main } }
      - deploy-staging:
          requires: [build-image]
          filters: { branches: { only: develop } }
      - slack/notify:
          event: fail
          template: basic_fail_1
```

---

## 28. Câu Hỏi Thường Gặp (FAQ)

**CircleCI có free thật không?**
Có. 6,000 build minutes/tháng, không cần thẻ tín dụng.

**Tôi dùng GitLab/Bitbucket được không?**
Được. CircleCI hỗ trợ GitHub, Bitbucket, GitLab.

**Config của tôi private được không?**
File `config.yml` nằm trong repo — repo private thì config private theo.

**Làm sao chạy CI cho fork PR?**
**Project Settings** → **Advanced** → bật **Build forked pull requests**.

**CircleCI có thay thế Jenkins được không?**
Được cho hầu hết trường hợp. Chỉ giữ Jenkins khi cần tự host vì lý do bảo mật/pháp lý.

**Credit hết giữa tháng thì sao?**
Pipeline dừng. Mua thêm credits hoặc đợi reset tháng sau. Gói Performance có thể mua thêm.

---

*Cập nhật: 10/10/2026 — Tổng 28 phần*
