# Hướng dẫn Harness — Nền tảng Software Delivery AI-Powered

> Hướng dẫn toàn diện từ cơ bản đến thực chiến: CI, CD, Feature Flags, Chaos Engineering,
> STO, governance và triển khai production trên Kubernetes.

## Mục lục

1. [Harness là gì](#1-harness-là-gì)
2. [Đăng ký tài khoản và thiết lập workspace](#2-đăng-ký-tài-khoản-và-thiết-lập-workspace)
3. [Pipeline đầu tiên](#3-pipeline-đầu-tiên)
4. [Connectors và Harness Delegate](#4-connectors-và-harness-delegate)
5. [Build Stage (CI)](#5-build-stage-ci)
6. [Deploy Stage (CD) lên Kubernetes](#6-deploy-stage-cd-lên-kubernetes)
7. [Chiến lược deployment nâng cao: Canary và Blue-Green](#7-chiến-lược-deployment-nâng-cao-canary-và-blue-green)
8. [Feature Flags](#8-feature-flags)
9. [Chaos Engineering (cơ bản)](#9-chaos-engineering-cơ-bản)
10. [STO — Security Testing Orchestration](#10-sto--security-testing-orchestration)
11. [Approval và Governance](#11-approval-và-governance)
12. [Dự án thực tế: CI → Canary Deploy lên Kubernetes](#12-dự-án-thực-tế-ci--canary-deploy-lên-kubernetes)
13. [Pricing](#13-pricing)
14. [Best Practices](#14-best-practices)
15. [Troubleshooting](#15-troubleshooting)
16. [FAQ và Cheat Sheet](#16-faq-và-cheat-sheet)

---

## 1. Harness là gì

**Harness** là một nền tảng software delivery (phân phối phần mềm) hiện đại, được phát triển bởi
**Harness Inc** — công ty do Jyoti Bansal (nhà sáng lập AppDynamics) thành lập năm 2017.
Harness tự định vị là **"AI-powered software delivery platform"**: gom mọi khâu trong vòng đời
phát triển phần mềm — build, test, deploy, kiểm thử bảo mật, quản lý chi phí cloud,
quản lý tính năng, kiểm thử hỗn loạn (chaos) — vào **một nền tảng thống nhất** với một
control plane chung, một hệ thống RBAC chung, một pipeline DSL chung (YAML) và một lớp
governance chung.

### 1.1 Các module chính

| Module | Tên đầy đủ | Chức năng |
|---|---|---|
| **CD** | Continuous Delivery | Deploy tự động lên Kubernetes, ECS, VM, serverless... với canary, blue-green, rolling; verify bằng AI/ML |
| **CI** | Continuous Integration | Build + test code trên infrastructure của bạn hoặc Harness Cloud; cache thông minh, test parallelism |
| **FF** | Feature Flags (và FME — Feature Management & Experimentation) | Bật/tắt tính năng theo đối tượng, rollout theo %, A/B testing, kill switch |
| **Chaos** | Chaos Engineering (tích hợp ChaosNative/Litmus) | Tiêm lỗi (fault injection) vào hệ thống để kiểm chứng khả năng chịu lỗi |
| **STO** | Security Testing Orchestration | Điều phối các công cụ scan bảo mật (SAST/DAST/SCA) ngay trong pipeline |
| **CCM** | Cloud Cost Management | Quan sát và tối ưu chi phí cloud (AWS/GCP/Azure), phát hiện tài nguyên thừa |
| **SRM** | Service Reliability Management | Theo dõi SLO, error budget |
| **IACM** | Infrastructure as Code Management | Quản lý Terraform/OpenTofu trong pipeline |
| **SEI** | Software Engineering Insights | Đo lường hiệu suất kỹ sư (DORA metrics...) |

Điểm mạnh nhất: **mọi module dùng chung một pipeline** — build (CI) → scan (STO) →
deploy canary (CD) → kiểm tra SLO (SRM) → bật flag (FF), không cần nối tool rời rạc.

### 1.2 So sánh với các công cụ khác

| Tiêu chí | Harness | Jenkins | GitHub Actions | Spinnaker |
|---|---|---|---|---|
| Mô hình | SaaS + Delegate (agent chạy trong infra của bạn) | Self-hosted, master/agent | SaaS, runner cloud hoặc self-hosted | Self-hosted, microservices |
| Viết pipeline | YAML + UI trực quan (2 chiều) | Groovy (Jenkinsfile) hoặc UI | YAML | JSON (Pipeline as Code qua API) |
| Deploy nâng cao | Canary, Blue-Green, Rolling tích hợp sẵn + AI verify | Phải tự viết script | Phải tự viết action/script | Canary mạnh nhưng cấu hình phức tạp |
| AI | AI verify deployment, AI test selection, AI SRE | Không có | Copilot riêng, không tích hợp sâu | Không có |
| Secrets | Tích hợp Vault, AWS SM, GCP SM, Azure KV | Credentials store cơ bản | GitHub Secrets | Vault |
| Feature Flags / Chaos / STO / CCM | Module chính thức trong cùng nền tảng | Plugin rời rạc | Không có (dùng tool ngoài) | Không có |
| Chi phí vận hành | Thấp (SaaS, chỉ cài Delegate) | Cao (bảo trì Jenkins master/plugins) | Thấp | Rất cao (chạy hàng chục microservices) |
| Học curve | Trung bình (khái niệm Service/Environment/Infrastructure mới) | Thấp với người quen Groovy | Thấp | Cao |

**Nên chọn Harness khi:** muốn chuẩn hoá delivery nhiều service mà không bảo trì
Jenkins/Spinnaker; cần deploy an toàn (canary/blue-green + verify + auto rollback) mà không
tự viết script; muốn gom CI/CD/scan/cost/flag vào một nơi; môi trường multi-cloud/hybrid.

**Không nên khi:** project cá nhân cực nhỏ (chỉ cần `docker build && kubectl apply`);
team bắt buộc air-gapped hoàn toàn (lúc đó dùng bản Self-Managed Platform).

### 1.3 Kiến trúc tổng quan

```
Harness SaaS (app.harness.io) — UI, API, pipeline engine, RBAC, policy, AI
        ▲  (Delegate poll task qua HTTPS outbound, không cần mở inbound)
        │
Harness Delegate (chạy trong infra của bạn: K8s / Docker / VM)
  → thực thi kubectl, helm, docker, terraform, script... tại chỗ
```

Nguyên tắc quan trọng: **Delegate chỉ gọi ra ngoài (outbound)** tới Harness SaaS qua HTTPS,
không cần mở port inbound vào cluster của bạn.

---

## 2. Đăng ký tài khoản và thiết lập workspace

### 2.1 Đăng ký

1. Truy cập **https://app.harness.io** → chọn **Sign up** (hoặc "Start for free").
2. Đăng ký bằng email công việc, Google SSO hoặc GitHub SSO.
3. Đặt tên **Account** (ví dụ: `cong-ty-abc`). Ghi nhớ **Account ID** — cần khi cài Delegate.
4. Chọn module muốn dùng (có thể bật thêm sau) — bắt đầu với **CD + CI**.

> **Mẹo:** Harness cung cấp tài khoản **Free** cho cá nhân và team nhỏ — đủ để học và chạy
> project thật (xem chi tiết ở [mục 13](#13-pricing)).

### 2.2 Cấu trúc phân cấp: Account → Organization → Project

Harness tổ chức tài nguyên theo 3 cấp:

```
Account (công ty của bạn)
└── Organization (phòng ban / business unit, ví dụ: "Fintech")
    └── Project (team / ứng dụng, ví dụ: "payments-api")
        ├── Pipelines
        ├── Services
        ├── Environments
        ├── Connectors
        ├── Secrets
        └── ...
```

- **Organization**: **Account Settings → Organizations → New** (ví dụ `fintech`).
- **Project**: **Home → Projects → New Project**, chọn Organization + module
  (ví dụ `payments_api`).

Quy tắc đặt identifier (YAML tham chiếu bằng identifier): chỉ dùng chữ, số, gạch dưới,
bắt đầu bằng chữ cái (`payments_api`); duy nhất trong phạm vi của nó.

### 2.3 Các khái niệm cốt lõi cần nắm trước khi làm pipeline

| Khái niệm | Ý nghĩa | Ví dụ |
|---|---|---|
| **Pipeline** | Chuỗi các stage thực thi theo thứ tự | `build-and-deploy` |
| **Stage** | Một pha lớn trong pipeline, có `type` riêng | `CI`, `Deployment`, `Approval`, `FeatureFlag`, `Chaos`, `Security` |
| **Step** | Đơn vị công việc nhỏ nhất trong stage | `Run` (chạy script), `BuildAndPushDockerRegistry`, `K8sRollingDeploy` |
| **Service** | Định nghĩa "cái gì được deploy": artifacts + manifests + config | `payments-api` service |
| **Environment** | Định nghĩa "deploy đi đâu": cluster + namespace + biến môi trường | `staging`, `production` |
| **Infrastructure Definition** | Cách kết nối tới environment đó (connector + namespace...) | `k8s-direct-staging` |
| **Connector** | Thông tin đăng nhập tới hệ thống ngoài | GitHub connector, Docker Hub connector, K8s connector |
| **Input Set / Trigger** | Tham số hoá pipeline và cách kích hoạt tự động | Trigger chạy khi push lên `main` |

Triết lý Harness CD: **Service (cái gì) + Environment (ở đâu) = một lần deploy**.
Cùng một Service deploy tới nhiều Environment mà không sửa lại định nghĩa.

---

## 3. Pipeline đầu tiên

### 3.1 Tạo pipeline trên UI (từng bước)

1. **Project → Pipelines → Create a Pipeline** → tên `Hello Harness`
   (sửa identifier thành `hello_harness`).
2. **Add Stage → CI** → tên `Build` → infrastructure: **Harness Cloud**
   (chưa có Delegate) hoặc **Kubernetes** + connector cluster (đã có Delegate).
3. **Add Step → Run** → nhập `echo "Xin chào từ Harness CI!"`.
4. **Save → Run** → xem log trực tiếp trên UI.

### 3.2 Pipeline YAML — nhìn dưới lớp UI

Mọi thao tác trên UI đều được lưu thành YAML — bấm **YAML** (góc trên bên phải Pipeline Studio)
để xem/sửa trực tiếp:

```yaml
pipeline:
  name: Hello Harness
  identifier: hello_harness
  projectIdentifier: my_project
  orgIdentifier: default
  tags: {}
  stages:
    - stage:
        name: Build
        identifier: Build
        description: ""
        type: CI
        spec:
          cloneCodebase: false
          infrastructure:
            type: VM
            spec:
              type: Pool
              spec:
                os: Linux
                arch: Amd64
          execution:
            steps:
              - step:
                  type: Run
                  name: Run_1
                  identifier: Run_1
                  spec:
                    shell: Sh
                    command: |-
                      echo "Xin chào từ Harness CI!"
                      uname -a
```

Giải thích từng phần:

- `pipeline`: metadata của pipeline (tên, identifier, project/org chứa nó).
- `stages`: danh sách các stage. Mỗi stage có `name` (hiển thị), `identifier` (tham chiếu),
  `type` (`CI`, `Deployment`, `Approval`...), và `spec` (cấu hình riêng theo type).
- `stage.spec.infrastructure`: chạy ở đâu. `VM` + `Pool` = Harness Cloud;
  `KubernetesDirect` = chạy trên cluster của bạn qua Delegate.
- `stage.spec.execution.steps`: các bước chạy tuần tự; step `Run` thực thi shell script.

### 3.3 Biến và biểu thức (Expressions)

Harness dùng cú pháp `<+...>` để tham chiếu biến — gọi là **expressions**:

```yaml
# Biến pipeline do người chạy nhập (runtime input)
command: echo "Deploying version <+pipeline.variables.APP_VERSION>"

# Output của step trước (ví dụ: tag image vừa build)
image: myorg/app:<+stages.Build.spec.artifacts.primary.tag>

# Biến môi trường hệ thống
command: echo "Pipeline <+pipeline.name> chạy bởi <+pipeline.triggeredBy.name>"
```

Các nguồn biến hay dùng: `<+pipeline.variables.TEN_BIEN>` (khai báo ở mục Variables) ·
| `<+input>` | Runtime input — hỏi người dùng khi bấm Run |
| `<+stages.<StageId>.spec.execution.steps.<StepId>.output.outputVariables.X>` | Output của một step |
| `<+env.name>`, `<+service.name>` | Tên environment/service trong stage CD |
| `<+pipeline.executionId>` | ID của lần chạy hiện tại (duy nhất) |
| `<+codebase.branch>`, `<+codebase.commitSha>` | Thông tin git khi stage CI clone codebase |

> Trong YAML, expression nên đặt trong quote: `tag: "<+pipeline.variables.VERSION>"`.

---

## 4. Connectors và Harness Delegate

### 4.1 Connector là gì?

**Connector** là nơi Harness lưu thông tin kết nối tới hệ thống bên ngoài: GitHub, Docker Hub,
Kubernetes cluster, AWS, GCP... Thay vì nhét username/password vào từng pipeline, bạn tạo
connector một lần rồi tham chiếu bằng `connectorRef` ở mọi nơi.

Tạo ở **Project Setup → Connectors → New Connector**. Các loại thường dùng:

| Loại | Dùng cho | Thông tin cần |
|---|---|---|
| GitHub / GitLab / Bitbucket | Clone code, lưu pipeline as code | URL + Personal Access Token (lưu trong Secret) |
| Docker Registry | Push/pull image | URL registry + username/password |
| Kubernetes Cluster | Deploy, chạy step trên K8s | **Chạy qua Delegate** (xem 4.2) |
| AWS / GCP / Azure | Cloud Cost, provision infra, ECR/GAR | Access key hoặc OIDC / service account |
| Artifactory / Nexus | Lưu artifact | URL + credentials |

Ví dụ tham chiếu connector trong pipeline:

```yaml
spec:
  connectorRef: my_dockerhub   # identifier của connector đã tạo
  repo: myorg/payments-api
```

### 4.2 Harness Delegate là gì?

**Harness Delegate** là một **worker agent** chạy trong hạ tầng của bạn (thường là một Deployment
trong Kubernetes cluster, hoặc container Docker trên VM). Mọi thao tác "chạm" vào hệ thống của
bạn — `kubectl apply`, `docker push`, `helm upgrade`, chạy script — đều do Delegate thực hiện.

Luồng: bấm **Run** → SaaS xếp task vào hàng đợi → Delegate **poll qua HTTPS outbound** →
chạy lệnh tại chỗ → stream log về SaaS.

Đặc điểm quan trọng:

- **Không cần mở firewall inbound**: Delegate chủ động kết nối ra `app.harness.io:443`.
- **Chọn Delegate theo tag**: gán `delegateSelectors` (ví dụ `prod`, `vpc-a`)
  để task chạy đúng Delegate trong môi trường tương ứng.
- **Tự động scale**: Delegate là Deployment K8s bình thường, có thể gắn HPA.
- **Nâng cấp thường xuyên**: Harness phát hành bản Delegate mới liên tục; nên upgrade
  để nhận fix bảo mật.

### 4.3 Cài Delegate lên Kubernetes bằng Helm

**Bước 1 — Lấy thông tin trên Harness UI:**

Vào **Project Setup → Delegates → New Delegate → Kubernetes** → đặt tên (ví dụ `k8s-prod`)
→ UI sẽ hiện lệnh cài đặt kèm **Delegate Token** (tạo tự động, dạng secret).

**Bước 2 — Cài bằng Helm (chạy trên máy có quyền admin cluster):**

```bash
# Thêm repo chart của Harness
helm repo add harness-delegate https://app.harness.io/storage/harness-download/delegate-helm-chart/
helm repo update

# Cài delegate vào namespace riêng
helm install k8s-prod harness-delegate/harness-delegate-ng \
  --namespace harness-delegate-ng --create-namespace \
  --set delegateName=k8s-prod \
  --set accountId=YOUR_ACCOUNT_ID \
  --set delegateToken=YOUR_DELEGATE_TOKEN \
  --set managerEndpoint=https://app.harness.io/gratis \
  --set delegateDockerImage=harness/delegate:25.11.86111
```

Thay `YOUR_ACCOUNT_ID` (xem ở **Account Settings → Overview**) và `YOUR_DELEGATE_TOKEN`
bằng giá trị UI cấp; dùng đúng tag image mà UI gợi ý.

**Bước 3 — Kiểm tra:** `kubectl -n harness-delegate-ng get pods` (phải `Running`).
Trên UI (**Delegates**), delegate hiện **Connected** sau 1–2 phút.

**Cài Delegate bằng Docker (cho VM không có K8s):** dùng image `harness/delegate`
với các biến môi trường `DELEGATE_NAME`, `ACCOUNT_ID`, `DELEGATE_TOKEN`,
`MANAGER_HOST_AND_PORT=https://app.harness.io/gratis`.

### 4.4 Tạo Kubernetes Connector (qua Delegate)

Khi Delegate đã connected: **Project Setup → Connectors → New → Kubernetes Cluster** →
tên `k8s_prod` → **"Use the credentials of a specific Harness Delegate"** → chọn delegate
`k8s-prod` → **Test Connection** → **Save**.

### 4.5 Tạo GitHub Connector và Docker Registry Connector

**GitHub:** tạo **Personal Access Token (classic)** với scope `repo` trên GitHub, rồi trên
Harness **Connectors → New → GitHub** → tên `github` → Username + Token (token lưu thành
Secret `github_pat`). Bật **Enable API access** nếu muốn Harness tự tạo webhook/trigger.

**Docker Hub:** **Connectors → New → Docker Registry** → tên `dockerhub` →
Provider Type **DockerHub** → username + Docker access token (lưu thành Secret).
Test connection → Save.

---

## 5. Build Stage (CI)

### 5.1 Cấu trúc một CI stage

```yaml
- stage:
    name: Build and Test
    identifier: build_and_test
    type: CI
    spec:
      cloneCodebase: true            # clone repo đã gắn ở pipeline
      infrastructure:
        type: KubernetesDirect       # build trên chính cluster K8s (qua Delegate)
        spec:
          connectorRef: k8s_prod
          namespace: ci-builds
          automountServiceAccountToken: true
      execution:
        steps:
          - step: { ... }            # các step chạy tuần tự
```

`cloneCodebase: true` yêu cầu pipeline phải gắn **Codebase**: khi tạo pipeline chọn
**Codebase → GitHub connector `github`** + repo + branch mặc định.

### 5.2 Ví dụ đầy đủ: build + test + push image Docker

Giả sử repo là một app Node.js có `Dockerfile`, `package.json` với script `test`.
Pipeline CI hoàn chỉnh:

```yaml
pipeline:
  name: payments-api CI
  identifier: payments_api_ci
  projectIdentifier: payments_api
  orgIdentifier: fintech
  properties:
    ci:
      codebase:
        connectorRef: github
        repoName: myorg/payments-api
        build: <+input>              # runtime input: branch / tag / PR
  stages:
    - stage:
        name: Build Test Push
        identifier: build_test_push
        type: CI
        spec:
          cloneCodebase: true
          infrastructure:
            type: KubernetesDirect
            spec:
              connectorRef: k8s_prod
              namespace: ci-builds
          execution:
            steps:
              # Cài dependencies (có cache)
              - step:
                  type: Run
                  name: Install deps
                  identifier: install_deps
                  spec:
                    connectorRef: k8s_prod
                    image: node:20-alpine
                    shell: Sh
                    command: npm ci
                    caching:
                      enabled: true
                      key: "npm-{{ checksum \"package-lock.json\" }}"
                      paths:
                        - /root/.npm

              # Chạy unit test
              - step:
                  type: Run
                  name: Unit tests
                  identifier: unit_tests
                  spec:
                    connectorRef: k8s_prod
                    image: node:20-alpine
                    shell: Sh
                    command: npm test -- --ci
                    reports:
                      type: JUnit
                      spec:
                        paths:
                          - junit.xml

              # Build và push image
              - step:
                  type: BuildAndPushDockerRegistry
                  name: Build and push image
                  identifier: build_push
                  spec:
                    connectorRef: dockerhub
                    repo: myorg/payments-api
                    tags:
                      - <+codebase.commitSha>          # tag = commit SHA (duy nhất)
                      - latest
                    dockerfile: Dockerfile
                    context: .
                    optimize: true                     # bật BuildKit
```

Giải thích:

- **Codebase as runtime input** (`build: <+input>`): mỗi lần bấm Run, Harness hỏi bạn muốn
  build branch/tag/PR nào.
- **Caching**: cache chỉ invalidate khi `package-lock.json` thay đổi. Mỗi step `Run` chạy
  trong container riêng nhưng share workspace.
- **BuildAndPushDockerRegistry**: build bằng Docker/BuildKit rồi push. Tag bằng
  `<+codebase.commitSha>` đảm bảo mỗi build có image duy nhất, truy vết được.

### 5.3 Các step CI hay dùng khác

| Step | Dùng khi |
|---|---|
| `Run` | Chạy script tuỳ ý (npm, mvn, go build, pytest...) |
| `BuildAndPushDockerRegistry` / `BuildAndPushECR` / `BuildAndPushGCR` | Build + push image |
| `RunTests` | Chạy test với parallelism (chia test ra nhiều máy) |
| `Background` | Chạy service nền (DB, Redis) cho integration test |
| `Security` (STO) | Chạy scan bảo mật — xem [mục 10](#10-sto--security-testing-orchestration) |

**Test Intelligence (AI):** Harness CI có thể chỉ chạy **những test liên quan tới code bạn sửa**
thay vì toàn bộ suite — bật ở **Project → Test Intelligence**. Với repo lớn, tính năng này
giảm thời gian CI từ 40 phút xuống còn vài phút.

### 5.4 Matrix build — build nhiều phiên bản song song

Thêm `strategy.matrix` vào **stage**/**step** để chạy song song:

```yaml
strategy:
  matrix:
    node_version: ["18", "20", "22"]
  maxConcurrency: 3
```

Hữu ích khi test thư viện trên nhiều phiên bản runtime.

---
## 6. Deploy Stage (CD) lên Kubernetes

### 6.1 Chuẩn bị: Service, Environment, Infrastructure Definition

Trước khi viết stage deploy, bạn cần 3 tài nguyên này. Có thể tạo trên UI
(**Services**, **Environments** trong Project) hoặc khai báo inline trong pipeline YAML.

**Service** — "deploy cái gì":

```yaml
service:
  name: payments-api
  identifier: payments_api
  serviceDefinition:
    type: Kubernetes
    spec:
      manifests:
        - manifest:
            identifier: k8s_manifests
            type: K8sManifest
            spec:
              store:
                type: Github                    # manifest lấy từ repo Git
                spec:
                  connectorRef: github
                  gitFetchType: Branch
                  paths:
                    - k8s/base                 # thư mục chứa deployment.yaml, service.yaml
                  branch: main
              skipResourceVersioning: false
      artifacts:
        primary:
          primaryArtifactRef: <+input>         # tag nhập lúc chạy
          sources:
            - spec:
                connectorRef: dockerhub
                imagePath: myorg/payments-api
                tag: <+input>
              identifier: payments_api_image
              type: DockerRegistry
```

Manifest K8s trong repo (`k8s/base/deployment.yaml`) — Harness tự thay image tag lúc deploy,
bạn chỉ cần dùng biến cho những giá trị muốn tham số hoá:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments-api
spec:
  replicas: {{ .Values.REPLICAS | default 2 }}
  selector:
    matchLabels:
      app: payments-api
  template:
    metadata:
      labels:
        app: payments-api
    spec:
      containers:
        - name: api
          image: myorg/payments-api:latest   # Harness tự thay tag lúc deploy
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "{{ .Values.NODE_ENV }}"
```

**Environment + Infrastructure Definition** — "deploy đi đâu":

```yaml
environment:
  name: Staging
  identifier: staging
  type: Production                             # hoặc PreProduction
infrastructure:
  environmentRef: staging
  infrastructureDefinition:
    type: KubernetesDirect
    spec:
      connectorRef: k8s_prod                   # cluster staging
      namespace: payments-staging
      releaseName: payments-api
```

### 6.2 Stage deploy Rolling Update

Rolling update là chiến lược mặc định của Kubernetes: thay pod cũ bằng pod mới từng đợt,
không downtime nếu có nhiều replica.

```yaml
- stage:
    name: Deploy to Staging
    identifier: deploy_staging
    type: Deployment
    spec:
      deploymentType: Kubernetes
      service:
        name: payments-api
        identifier: payments_api
        serviceDefinition:
          type: Kubernetes
          spec:
            manifests:
              - manifest:
                  identifier: k8s_manifests
                  type: K8sManifest
                  spec:
                    store:
                      type: Github
                      spec:
                        connectorRef: github
                        gitFetchType: Branch
                        paths:
                          - k8s/base
                        branch: main
            artifacts:
              primary:
                primaryArtifactRef: <+input>
                sources:
                  - spec:
                      connectorRef: dockerhub
                      imagePath: myorg/payments-api
                      tag: <+input>
                    identifier: payments_api_image
                    type: DockerRegistry
      environment:
        name: Staging
        identifier: staging
        type: PreProduction
      infrastructure:
        environmentRef: staging
        infrastructureDefinition:
          type: KubernetesDirect
          spec:
            connectorRef: k8s_prod
            namespace: payments-staging
            releaseName: payments-api
      execution:
        steps:
          - step:
              type: K8sRollingDeploy
              name: Rolling deploy
              identifier: rolling_deploy
              timeout: 10m
              spec:
                skipDryRun: false
                pruningEnabled: false
        rollbackSteps:
          - step:
              type: K8sRollingRollback
              name: Rolling rollback
              identifier: rolling_rollback
              timeout: 10m
              spec: {}
```

Luồng chạy: Harness render manifest (thay biến + image tag) → chạy
`kubectl apply --dry-run=server` trước (vì `skipDryRun: false`) → apply thật, chờ rollout
hoàn tất trong `timeout` → nếu step fail, tự động chạy `rollbackSteps`.

### 6.3 Verify deployment bằng AI (Continuous Verification)

Điểm khác biệt lớn của Harness CD: sau khi deploy, nó tự động **verify** bằng ML —
so sánh metrics (Prometheus/Datadog...) và log (Splunk/Elastic...) của bản mới với baseline,
chấm điểm rủi ro. Nếu error rate tăng đột biến → đánh dấu deploy "rủi ro cao" và có thể
tự rollback. Thêm step `Verify` sau deploy (cần tạo **Monitored Service** gắn data source
trước):

```yaml
- step:
    type: Verify
    name: Verify deployment
    identifier: verify
    spec:
      type: Canary
      spec:
        sensitivity: MEDIUM
        duration: 5m
        deploymentTag: <+pipeline.executionId>
```

---

## 7. Chiến lược deployment nâng cao: Canary và Blue-Green

### 7.1 Canary deployment

**Ý tưởng:** đưa bản mới ra cho một **tỷ lệ nhỏ traffic** trước (ví dụ 10%), quan sát,
rồi tăng dần (25% → 50% → 100%). Nếu có lỗi, chỉ một phần nhỏ user bị ảnh hưởng.

Trên Kubernetes, Harness triển khai canary bằng cách tạo thêm Deployment canary
chạy song song với Deployment ổn định (stable). Cách đơn giản nhất không cần service mesh:
canary theo **số pod** (1 pod canary trên 10 pod tổng ≈ 10% traffic với Service round-robin).

**Cấu hình canary trong Harness:**

```yaml
- stage:
    name: Canary to Production
    identifier: canary_prod
    type: Deployment
    spec:
      deploymentType: Kubernetes
      service: { ... }            # giống mục 6
      environment:
        name: Production
        identifier: production
        type: Production
      infrastructure:
        environmentRef: production
        infrastructureDefinition:
          type: KubernetesDirect
          spec:
            connectorRef: k8s_prod
            namespace: payments-prod
            releaseName: payments-api
      execution:
        steps:
          # Đợt 1: 1 pod canary
          - step:
              type: K8sCanaryDeploy
              name: Canary 10%
              identifier: canary_10
              spec:
                instanceUnitType: Count
                instances: 1

          # Verify tự động
          - step:
              type: Verify
              name: Verify canary
              identifier: verify_canary
              spec:
                type: Canary
                spec:
                  sensitivity: HIGH
                  duration: 10m

          # Đợt 2: 5 pod (~50%)
          - step:
              type: K8sCanaryDeploy
              name: Canary 50%
              identifier: canary_50
              spec:
                instanceUnitType: Count
                instances: 5

          - step:
              type: HarnessApproval
              name: Duyệt full rollout
              identifier: approve_rollout
              timeout: 1d
              spec:
                approvalMessage: Canary 50% ổn định. Rollout 100%?
                approvers:
                  userGroups: [release_managers]
                  minimumCount: 1

          # Đợt 3: full rollout
          - step:
              type: K8sRollingDeploy
              name: Full rollout
              identifier: full_rollout
              spec:
                skipDryRun: true
        rollbackSteps:
          - step:
              type: K8sCanaryDelete
              name: Xoá canary
              identifier: del_canary
              spec: {}
          - step:
              type: K8sRollingRollback
              name: Rollback stable
              identifier: rollback_stable
              spec: {}
```

Giải thích luồng:

1. `K8sCanaryDeploy` tạo Deployment `-canary` với số pod chỉ định, gắn chung Service
   nên nhận một phần traffic.
2. `Verify` quan sát metrics trong 10 phút → tăng canary lên 5 pod (~50%).
3. `HarnessApproval` dừng pipeline chờ người duyệt (xem [mục 11](#11-approval-và-governance)).
4. `K8sRollingDeploy` nâng stable lên bản mới (full rollout), canary được dọn tự động.
5. Nếu bất kỳ step nào fail → `rollbackSteps`: xoá canary, rollback stable.

**Canary theo % chính xác** (cần Istio/Linkerd hoặc ALB weighted targets):
dùng `instanceUnitType: Percentage`. Với người mới, canary theo pod count là đủ.

### 7.2 Blue-Green deployment

**Ý tưởng:** chạy **hai môi trường song song** — Blue (đang phục vụ user) và Green (bản mới).
Deploy bản mới lên Green, test kỹ, rồi **chuyển traffic** sang Green trong một thao tác
(swap service selector). Lỗi → swap ngược lại trong vài giây.

Ưu điểm: rollback gần như tức thì. Nhược điểm: tốn gấp đôi tài nguyên lúc deploy;
không hợp với workload có migration DB phức tạp.

**Cấu hình trong Harness:**

```yaml
execution:
  steps:
    # B1: deploy bản mới lên Green (chưa nhận traffic)
    - step:
        type: K8sBlueGreenDeploy
        name: Deploy to Green
        identifier: deploy_green
        timeout: 10m
        spec:
          skipDryRun: false
          pruningEnabled: false

    # B2: smoke test trên Green
    - step:
        type: Run
        name: Smoke test Green
        identifier: smoke_test_green
        spec:
          shell: Sh
          command: |-
            # Harness expose địa chỉ Green qua biến nội bộ
            GREEN_URL="http://payments-api-green.payments-prod.svc.cluster.local:3000"
            curl -fsS "$GREEN_URL/health" || exit 1
            curl -fsS "$GREEN_URL/api/version" | grep -q "<+input>"

    # B3: duyệt trước khi chuyển traffic
    - step:
        type: HarnessApproval
        name: Duyệt chuyển traffic
        identifier: approve_switch
        timeout: 4h
        spec:
          approvalMessage: Green đã pass smoke test. Chuyển 100% traffic?
          approvers:
            userGroups: [release_managers]
            minimumCount: 1

    # B4: swap traffic sang Green
    - step:
        type: K8sBGSwapServices
        name: Swap to Green
        identifier: swap_to_green
        spec: {}
  rollbackSteps:
    # Rollback: swap ngược về Blue
    - step:
        type: K8sBGSwapServices
        name: Swap back to Blue
        identifier: swap_back_blue
        spec:
          downsizeOldService: false
```

> **Lưu ý:** Blue-Green của Harness yêu cầu Service K8s dùng selector do Harness quản lý
> (nó tự thêm label `harness.io/track`). Hãy để Harness tạo Service ở lần deploy đầu tiên
> để tránh xung đột selector.

### 7.3 Chọn chiến lược nào?

| | Rolling | Canary | Blue-Green |
|---|---|---|---|
| Downtime | Không (nếu đủ replica) | Không | Không |
| Rủi ro với user | 100% user nhận bản mới cùng lúc | Tăng dần, kiểm soát được | 0% cho đến lúc swap, rồi 100% |
| Tài nguyên thêm | Ít (maxSurge) | Trung bình | Gấp đôi lúc deploy |
| Rollback | Chậm (rollout ngược) | Nhanh (xoá canary) | Rất nhanh (swap service) |
| Phù hợp | Service nội bộ, ít rủi ro | API public, cần quan sát dần | Release lớn, cần test kỹ pre-prod |

---

## 8. Feature Flags

### 8.1 Khái niệm

**Feature Flag** là một công tắc trong code cho phép **bật/tắt tính năng mà không cần
deploy lại**. Kết hợp với CD, bạn deploy code "tắt" lên production trước, rồi bật dần
cho từng nhóm user — gọi là **decouple deployment khỏi release**.

Các loại flag trong Harness:

| Loại | Dùng cho | Ví dụ |
|---|---|---|
| **Boolean** | Bật/tắt | `new_checkout_ui` |
| **String / Number / JSON** | Cấu hình động | `max_upload_mb = 50`, `promo_banner = {...}` |
| **Multivariate** | A/B testing | `pricing_test` → `control` / `variant_a` / `variant_b` |

### 8.2 Tạo flag trên UI

1. **Project → Feature Flags → New Flag**: tên `Giao diện thanh toán mới`,
   identifier `new_checkout_ui`, kind **Boolean**, default `false`.
2. **Targeting rules**: `email` kết thúc `@congty.vn` → `true` (nội bộ test);
   `country` = `VN` + 10% → `true` (rollout 10%); còn lại `false`.
3. Bật flag cho environment `Production`, lấy **SDK Key**
   (**Feature Flags → SDK Keys**).

### 8.3 Dùng SDK trong code

**Node.js (server-side):**

```bash
npm install @harnessio/ff-nodejs-server-sdk
```

```js
// ff.js — khởi tạo client một lần khi app start
const { CFClient } = require('@harnessio/ff-nodejs-server-sdk');

const client = CFClient.getInstance(process.env.HARNESS_FF_SDK_KEY);

async function initFF() {
  await client.waitForInitialization();
  console.log('Feature Flags sẵn sàng');
}

// Lấy giá trị flag cho một user
async function isNewCheckoutEnabled(user) {
  const target = {
    identifier: user.id,
    name: user.email,
    attributes: { email: user.email, country: user.country, plan: user.plan },
  };
  return client.boolVariation('new_checkout_ui', target, false);
}

module.exports = { initFF, isNewCheckoutEnabled };
```

```js
// routes/checkout.js — dùng flag để rẽ nhánh
const { isNewCheckoutEnabled } = require('../ff');

app.get('/checkout', async (req, res) => {
  const useNewUI = await isNewCheckoutEnabled(req.user);
  return res.render(useNewUI ? 'checkout-v2' : 'checkout-v1');
});
```

**Các SDK khác:** Java, Python, Go, .NET, Ruby, iOS (Swift), Android (Kotlin),
JavaScript client-side, React... Cách dùng tương tự: `init(SDK_KEY)` →
`boolVariation(flagKey, target, default)`.

> **Client-side:** dùng **Client SDK Key** riêng (public được), không bao giờ nhúng
> Server SDK Key vào frontend.

### 8.4 Điều khiển flag từ pipeline

Bật/tắt flag như một **stage trong pipeline** — ví dụ sau khi canary ổn định thì tự bật 100%:

```yaml
- stage:
    name: Enable new checkout
    identifier: enable_checkout_flag
    type: FeatureFlag
    spec:
      environmentRef: production
      flags:
        - identifier: new_checkout_ui
          state: "on"                 # hoặc "off"
```

Rollback: thêm stage FeatureFlag vào `rollbackSteps` để tắt flag —
**tắt tính năng lỗi trong vài giây mà không cần redeploy** ("kill switch").

### 8.5 Best practices cho Feature Flags

- Đặt tên có tiền tố: `exp_`, `ops_`, `rel_` (ví dụ `rel_new_checkout`).
- Flag tạm thời phải có ngày xoá — release xong thì xoá code + flag trong 1–2 sprint.
- Default = trạng thái an toàn (khi SDK lỗi vẫn ổn).
- Mọi thay đổi flag đều có audit log (ai đổi, khi nào).

---

## 9. Chaos Engineering (cơ bản)

### 9.1 Khái niệm

**Chaos Engineering** là phương pháp **chủ động gây sự cố có kiểm soát** (kill pod,
gây trễ mạng, đầy ổ đĩa...) trên staging để trả lời: *"Hệ thống có thật sự chịu lỗi
như thiết kế không?"* — diễn tập vào giờ hành chính với blast radius giới hạn,
thay vì đợi sự cố thật lúc 2h sáng.

Harness Chaos Engineering (xây dựng trên ChaosNative/Litmus) cho phép tạo
**chaos experiment** và chạy như một stage trong pipeline — ví dụ: sau mỗi lần deploy
staging, tự động kill 1 pod và kiểm tra service vẫn sống.

### 9.2 Cài đặt cơ bản

**Project → Chaos → Chaos Infrastructures → New** → **Kubernetes**, tên `chaos-staging`,
chọn cluster staging. Harness cài agent vào namespace `harness-chaos`. **Chaos Hub** cung cấp
sẵn các fault phổ biến (pod-delete, container-kill, pod-network-latency, disk-fill...).

### 9.3 Tạo chaos experiment đơn giản: kill pod

**Chaos → Experiments → New Experiment** → infrastructure `chaos-staging` →
**Add Fault → pod-delete**:

| Tham số | Giá trị | Ý nghĩa |
|---|---|---|
| `TARGET_PODS` | `payments-api` | Label app của pod muốn kill |
| `PODS_AFFECTED_PERC` | `50` | Kill 50% số pod (giới hạn blast radius) |
| `CHAOS_INTERVAL` | `10s` | Khoảng cách giữa các lần kill |
| `TOTAL_CHAOS_DURATION` | `60s` | Tổng thời gian thí nghiệm |
| `RAMP_TIME` | `10s` | Đợi 10s trước khi bắt đầu |

Định nghĩa **steady-state hypothesis** (giả thuyết trạng thái ổn định) — probe kiểm tra
service vẫn healthy trong lúc chaos diễn ra:

```yaml
# probe: gọi /health phải trả về 200 trong suốt experiment
type: httpProbe
name: payments-api-health
properties:
  url: http://payments-api.payments-staging.svc.cluster.local:3000/health
  method: GET
  criteria: "==200"
  responseTimeout: 5000
  interval: 5
  retry: 3
```

Nếu probe fail → experiment đánh dấu **fail** → bạn biết hệ thống chưa chịu được
việc mất 50% pod (cần tăng replica tối thiểu hoặc thêm PodDisruptionBudget).

### 9.4 Chạy chaos trong pipeline CD

```yaml
- stage:
    name: Chaos test staging
    identifier: chaos_staging
    type: Chaos
    spec:
      experimentRef: pod_delete_payments   # identifier của experiment đã tạo
      infrastructureRef: chaos-staging
```

Kịch bản thực tế: **Deploy → Chaos → Verify** — biến chaos thành gate chất lượng tự động.

> **Nguyên tắc an toàn:** không bao giờ chạy chaos experiment đầu tiên trên production.
> Luôn bắt đầu ở staging, giới hạn blast radius (namespace/label cụ thể), đặt
> `TOTAL_CHAOS_DURATION` ngắn, và đảm bảo có người trực khi chạy.

---

## 10. STO — Security Testing Orchestration

### 10.1 STO là gì?

**Security Testing Orchestration** điều phối các công cụ kiểm thử bảo mật **ngay trong
pipeline CI/CD**, thay vì scan thủ công rời rạc. STO hỗ trợ 3 nhóm scan:

| Nhóm | Tên đầy đủ | Công cụ tích hợp sẵn (ví dụ) |
|---|---|---|
| **SAST** | Static Application Security Testing — quét mã nguồn | Snyk Code, Checkmarx, SonarQube, Semgrep |
| **SCA** | Software Composition Analysis — quét thư viện phụ thuộc | Snyk Open Source, Mend (Whitesource), OWASP Dependency-Check |
| **DAST** | Dynamic Application Security Testing — quét app đang chạy | OWASP ZAP, Burp, Nikto |

Kết quả scan được **chuẩn hoá về một dashboard**, chấm severity, và bạn đặt policy
"fail pipeline nếu có lỗi Critical" — **shift-left security**.

### 10.2 Thêm Snyk scan (SCA + SAST) vào CI stage

Yêu cầu: tạo Snyk API token, lưu vào Harness Secret tên `snyk_token`.
Trên UI: trong CI stage bấm **Add Step → Security → Snyk**, điền token và chọn loại scan —
UI sẽ sinh YAML. Dạng YAML tương đương:

```yaml
- step:
    type: Security
    name: Snyk scan
    identifier: snyk_scan
    spec:
      privileged: true
      settings:
        policy_type: orchestratedScan
        fail_on_severity: CRITICAL      # fail pipeline nếu có lỗi Critical
      config:
        target:
          type: repository
          variant: nodejs
      steps:
        - name: snyk
          identifier: snyk
          type: Snyk
          spec:
            mode: orchestration
            auth:
              access_token: <+secrets.getValue("snyk_token")>
            tool:
              product_name: snyk
              product_config_name: default
```

### 10.3 Thêm OWASP ZAP scan (DAST) sau khi deploy staging

DAST cần app đang chạy — đặt step này trong pipeline **sau stage deploy staging**:

```yaml
- stage:
    name: DAST scan staging
    identifier: dast_staging
    type: SecurityTests
    spec:
      infrastructure:
        type: KubernetesDirect
        spec:
          connectorRef: k8s_prod
          namespace: sto-scans
      execution:
        steps:
          - step:
              type: Security
              name: OWASP ZAP baseline
              identifier: zap_baseline
              spec:
                privileged: true
                settings:
                  policy_type: orchestratedScan
                  fail_on_severity: HIGH
                config:
                  target:
                    type: instance         # quét app đang chạy
                    name: payments-api-staging
                    variant: baseline
                    instance:
                      # URL của app vừa deploy lên staging
                      domain: payments-api-staging.congty.vn
                steps:
                  - name: zap
                    identifier: zap
                    type: Owasp
                    spec:
                      mode: orchestration
                      tool:
                        product_name: zap
                        product_config_name: baseline
```

### 10.4 Fail pipeline khi có lỗi nghiêm trọng

Dùng `fail_on_severity` trong step STO (như ví dụ trên): pipeline fail nếu scan phát hiện
lỗi từ mức đó trở lên. Nếu một finding là false positive hoặc đã có kế hoạch fix, tạo
**exemption** có thời hạn thay vì tắt scan.

Kết hợp STO vào pipeline chuẩn: **Build → Test → Snyk (SCA/SAST) → Push image →
Deploy staging → ZAP (DAST) → Canary production**. Bảo mật trở thành gate tự động.

---
## 11. Approval và Governance

### 11.1 Manual Approval step

Dừng pipeline chờ người duyệt — dùng trước khi deploy production, chuyển traffic, bật flag:

```yaml
- step:
    type: HarnessApproval
    name: Duyệt deploy production
    identifier: approve_prod
    timeout: 24h
    spec:
      approvalMessage: Deploy bản <+pipeline.variables.VERSION> lên production?
      includePipelineExecutionHistory: true
      approvers:
        userGroups:
          - release_managers         # group tạo ở Project → Access Control → User Groups
        minimumCount: 1              # cần ít nhất 1 người trong group duyệt
        disallowPipelineExecutor: true
```

Người duyệt nhận email/notification, bấm **Approve/Reject** ngay trên UI hoặc qua API.
Mọi quyết định đều lưu audit log.

### 11.2 Policy as Code với OPA

Harness cho phép viết **policy bằng Rego (Open Policy Agent)** để chặn pipeline vi phạm
quy tắc — ví dụ: "stage production bắt buộc có approval".

Vào **Account/Org/Project → Policies → New Policy**, chọn entity áp dụng (Pipeline):

```rego
package pipeline

# Chặn deploy production nếu thiếu approval step
deny[msg] {
  some stage
  stage := input.pipeline.stages[_].stage
  stage.spec.environment.identifier == "production"
  not has_approval(stage)
  msg := sprintf("Stage '%s' deploy production nhưng thiếu approval step", [stage.name])
}

has_approval(stage) {
  some step
  step := stage.spec.execution.steps[_].step
  step.type == "HarnessApproval"
}
```

Chế độ: **Warn** (cảnh báo) hoặc **Error** (chặn). Nên bắt đầu bằng Warn 1–2 tuần rồi mới
chuyển sang Error.

---

## 12. Dự án thực tế: CI → Canary Deploy lên Kubernetes

Ghép mọi thứ thành một pipeline hoàn chỉnh cho app `payments-api`.
Thứ tự stage: **Build/Test/Push (CI)** → **Deploy Staging + DAST** → **Canary Production**.

```yaml
pipeline:
  name: payments-api full delivery
  identifier: payments_api_delivery
  projectIdentifier: payments_api
  orgIdentifier: fintech
  tags: {}
  variables:
    - name: VERSION
      type: String
      value: <+codebase.commitSha>            # mặc định = commit SHA
  properties:
    ci:
      codebase:
        connectorRef: github
        repoName: myorg/payments-api
        build: <+input>

  stages:
    # ── STAGE 1: CI ──────────────────────────────────────────
    - stage:
        name: Build Test Push
        identifier: build_test_push
        type: CI
        spec:
          cloneCodebase: true
          infrastructure:
            type: KubernetesDirect
            spec:
              connectorRef: k8s_prod
              namespace: ci-builds
          execution:
            steps:
              - step:
                  type: Run
                  name: Unit tests
                  identifier: unit_tests
                  spec:
                    connectorRef: k8s_prod
                    image: node:20-alpine
                    shell: Sh
                    command: |-
                      npm ci
                      npm test -- --ci
              - step:
                  type: BuildAndPushDockerRegistry
                  name: Build + push image
                  identifier: build_push
                  spec:
                    connectorRef: dockerhub
                    repo: myorg/payments-api
                    tags:
                      - <+pipeline.variables.VERSION>
                    dockerfile: Dockerfile

    # ── STAGE 2: Deploy staging (rolling) ────────────────────
    - stage:
        name: Deploy Staging
        identifier: deploy_staging
        type: Deployment
        spec:
          deploymentType: Kubernetes
          # Service entity `payments_api` định nghĩa một lần ở cấp Project (xem mục 6.1)
          service:
            name: payments-api
            identifier: payments_api
            serviceRef: payments_api
          environment:
            name: Staging
            identifier: staging
            type: PreProduction
          infrastructure:
            environmentRef: staging
            infrastructureDefinition:
              type: KubernetesDirect
              spec:
                connectorRef: k8s_prod
                namespace: payments-staging
                releaseName: payments-api
          execution:
            steps:
              - step:
                  type: K8sRollingDeploy
                  name: Rolling deploy staging
                  identifier: rolling_staging
                  spec: { skipDryRun: false }
            rollbackSteps:
              - step:
                  type: K8sRollingRollback
                  name: Rollback staging
                  identifier: rollback_staging
                  spec: {}

    # ── STAGE 3: Canary production ───────────────────────────
    - stage:
        name: Canary Production
        identifier: canary_production
        type: Deployment
        spec:
          deploymentType: Kubernetes
          # Dùng chung Service entity với stage Staging
          service:
            name: payments-api
            identifier: payments_api
            serviceRef: payments_api
          environment:
            name: Production
            identifier: production
            type: Production
          infrastructure:
            environmentRef: production
            infrastructureDefinition:
              type: KubernetesDirect
              spec:
                connectorRef: k8s_prod
                namespace: payments-prod
                releaseName: payments-api
          execution:
            steps:
              - step:
                  type: K8sCanaryDeploy
                  name: Canary 1 pod
                  identifier: canary_1
                  spec: { instanceUnitType: Count, instances: 1 }
              - step:
                  type: Verify
                  name: Verify canary
                  identifier: verify_canary
                  spec:
                    type: Canary
                    spec: { sensitivity: MEDIUM, duration: 10m }
              - step:
                  type: HarnessApproval
                  name: Duyệt full rollout
                  identifier: approve_full
                  timeout: 4h
                  spec:
                    approvalMessage: Canary ổn. Rollout 100%?
                    approvers:
                      userGroups: [release_managers]
                      minimumCount: 1
              - step:
                  type: K8sRollingDeploy
                  name: Full rollout
                  identifier: full_rollout
                  spec: { skipDryRun: true }
            rollbackSteps:
              - step:
                  type: K8sCanaryDelete
                  name: Xoá canary
                  identifier: del_canary
                  spec: {}
              - step:
                  type: K8sRollingRollback
                  name: Rollback production
                  identifier: rollback_prod
                  spec: {}

    # ── STAGE 4: Bật feature flag ──────────────────────────────
    - stage:
        name: Enable new checkout
        identifier: enable_flag
        type: FeatureFlag
        spec:
          environmentRef: production
          flags:
            - identifier: new_checkout_ui
              state: "on"
```

**Các bước thực hiện:**

1. Tạo Organization `fintech`, Project `payments_api` (bật CI, CD, FF).
2. Cài Delegate `k8s_prod`, tạo connector `k8s_prod` / `github` / `dockerhub`,
   Secret `github_pat` + Docker credentials, user group `release_managers`.
3. Push code app + `Dockerfile` + `k8s/base/*.yaml` lên repo `myorg/payments-api`.
4. Tạo pipeline (dựng bằng UI hoặc dán YAML trên), bấm **Run** → chọn branch →
   theo dõi từng stage.
5. Thử nghiệm rollback: cố tình push image lỗi → quan sát canary fail → rollback tự động.

---

## 13. Pricing

Harness tính phí **theo module** — bạn chỉ trả cho module mình dùng. Ba gói chính:

| Gói | Đối tượng | Đặc điểm |
|---|---|---|
| **Free** | Cá nhân, team nhỏ, học tập | Miễn phí; đủ chạy CI/CD thật cho project nhỏ |
| **Team / Essentials** | Team đang phát triển (tới ~500 user) | Khoảng **$30/user/tháng** (theo công bố); support tiêu chuẩn |
| **Enterprise** | Tổ chức lớn | Giá custom (thường **$100K–200K/năm** trở lên); chọn module tuỳ ý, premium support, dedicated CSM |

> Giá có thể thay đổi — kiểm tra trang pricing chính thức của Harness trước khi lập ngân sách.

**Giới hạn gói Free (tham khảo, có thể thay đổi):**

- Số lượng **developer** và **pipeline execution** giới hạn ở mức dùng cho team nhỏ.
- Một số tính năng nâng cao (Advanced policy, một số integration enterprise, SSO/SAML nâng cao)
  chỉ có ở gói trả phí.
- Delegate: Free vẫn cài được Delegate và chạy pipeline qua hạ tầng của bạn.

**Mẹo tiết kiệm:** chạy CI trên **Harness Cloud** cho job nhẹ; bật **Test Intelligence**
để không chạy lại toàn bộ suite; dùng **CCM** để soi chi phí cloud sớm.

---

## 14. Best Practices

### 14.1 Pipeline as Code (Git Experience)

Đừng để pipeline "sống" mỗi trên UI. Bật **Git Experience** (**Project Setup → Git Experience**),
nối repo Git — mọi pipeline/service/environment được lưu thành file YAML, review bằng
Pull Request như code thường.

### 14.2 Templates — đừng copy-paste pipeline

Khi có 20 microservice với pipeline giống nhau 90%, tạo **Template**:

- **Step Template**: gói một step dùng lại ở mọi nơi (ví dụ "build + push chuẩn công ty").
- **Stage Template**: cả stage CI chuẩn (test → scan → push).
- **Pipeline Template**: khung pipeline đầy đủ, service chỉ override biến.

Tạo ở **Project Setup → Templates**, tham chiếu bằng `templateRef` — sửa template một chỗ,
mọi pipeline dùng nó được update (có versioning):

```yaml
- step:
    name: Build chuẩn công ty
    identifier: build_chuan
    template:
      templateRef: build_and_push_docker
      versionLabel: v1
      templateInputs:
        spec:
          repo: <+service.name>
```

Sửa template một chỗ → mọi pipeline dùng nó được update (có versioning, rollback template được).

### 14.3 Secrets management

- **Không bao giờ** hardcode secret trong YAML — luôn dùng **Secrets** (`<+secrets.getValue("ten_secret")>`)
  hoặc file secret (`type: Secret File`).
- Dùng **external secret manager** (Vault, AWS Secrets Manager, GCP Secret Manager...)
  làm backend: **Project Setup → Secrets → Secret Managers → New**.
- Xoay (rotate) token định kỳ; với GitHub PAT nên dùng **fine-grained token** giới hạn repo.

### 14.4 Các nguyên tắc khác

- **Một pipeline, nhiều environment**: chung Service, khác Environment; stage production
  luôn có approval + rollback steps.
- **Tag image bằng commit SHA**, không dùng `latest` cho production.
- **Timeout cho mọi step/stage** để pipeline treo không chạy mãi.
- **Đặt tên identifier nhất quán**: `snake_case`.
- **Dùng Input Sets** lưu sẵn bộ tham số cho các kịch bản chạy thường xuyên.

---

## 15. Troubleshooting

### 15.1 Delegate offline / không kết nối

**Triệu chứng:** UI báo delegate `Disconnected`; pipeline treo ở queued.

- Pod crash → `kubectl -n harness-delegate-ng get pods` và xem log.
- Sai Account ID / Token → soát lại lúc `helm install`.
- Không ra được internet → từ pod delegate `curl -v https://app.harness.io` (cần outbound 443).
- Pod bị OOMKilled → tăng memory limit trong values helm.
- Version quá cũ → upgrade image delegate.

### 15.2 Connector test failed

- **GitHub:** PAT hết hạn hoặc thiếu scope `repo`; repo thuộc org bật SSO thì PAT phải
  được authorize cho org đó.
- **Docker:** tài khoản ẩn danh bị rate limit — đăng nhập tài khoản (kể cả free) để tăng limit.
- **K8s:** 99% do Delegate — delegate phải **Connected** và có ClusterRole đủ quyền.

### 15.3 Deployment stuck (treo)

- **RollingDeploy treo:** pod mới không Ready (probe fail, image pull lỗi, thiếu resource).
  Kiểm tra `kubectl get/describe pod`. Đặt `timeout` hợp lý (10–15m).
- **ImagePullBackOff:** sai tên image/tag hoặc thiếu imagePullSecrets cho registry private.
- **Approval treo:** người duyệt không nhận mail → kiểm tra notification settings;
  luôn đặt `timeout` để pipeline không đợi vô hạn.

### 15.4 CI build fail

- **Cache key sai:** `{{ checksum "package-lock.json" }}` phải đúng đường dẫn từ root repo.
- **Hết disk trên build pod:** tăng ephemeral-storage hoặc dọn layer Docker cũ.
- **`cloneCodebase` fail:** kiểm tra GitHub connector + quyền đọc repo của PAT.

### 15.5 Canary / Blue-Green lỗi

- **Canary không nhận traffic:** Service selector không match label pod canary —
  để Harness tự quản lý label, đừng ghi đè selector thủ công.
- **BG swap không tác dụng:** Service đang dùng selector tĩnh tự viết — để Harness tạo
  Service ở lần deploy đầu tiên.

### 15.6 Cách debug hiệu quả

1. Mở execution → bấm step fail → đọc log chi tiết của Delegate.
2. Dùng **"Retry from failed step"** thay vì chạy lại từ đầu.
3. Kiểm tra **Audit Trail** (Account → Audit Trail) khi nghi ngờ ai đó sửa pipeline/connector.


## 16. FAQ và Cheat Sheet

### 16.1 FAQ

**Hỏi: Harness có miễn phí thật không, hay chỉ trial?**
Đáp: Có gói **Free** vĩnh viễn cho cá nhân/team nhỏ, không phải trial.

**Hỏi: Dữ liệu/code của tôi có bị gửi lên cloud Harness không?**
Đáp: Không — Delegate clone code và build **trong hạ tầng của bạn**. SaaS chỉ nhận
metadata, log và kết quả. Muốn tách hẳn thì dùng bản Self-Managed.

**Hỏi: Delegate có cần mở port inbound không?**
Đáp: Không — chỉ cần **outbound HTTPS (443)** tới `app.harness.io`.

**Hỏi: Khác gì giữa Harness CD và Argo CD?**
Đáp: Argo CD là **GitOps thuần** (agent sync trạng thái từ Git). Harness CD là
**pipeline-driven** (deploy → verify → approve → promote) và tích hợp sẵn CI/FF/STO/CCM.
Nhiều team dùng cả hai: Harness orchestrate, Argo CD sync.

**Hỏi: Viết YAML tay hay dùng UI?**
Đáp: Cả hai đồng bộ 2 chiều. Dựng bằng UI cho quen, rồi sửa YAML + Git Experience.

**Hỏi: Rollback tự động khi nào?**
Đáp: Khi step deploy fail và đã khai báo `rollbackSteps`. Kết hợp step `Verify` (AI)
để tự rollback khi metrics xấu.

### 16.2 Cheat Sheet

**Cài Delegate:**

```bash
helm repo add harness-delegate https://app.harness.io/storage/harness-download/delegate-helm-chart/
helm install k8s-prod harness-delegate/harness-delegate-ng \
  --namespace harness-delegate-ng --create-namespace \
  --set delegateName=k8s-prod \
  --set accountId=ACCOUNT_ID \
  --set delegateToken=TOKEN \
  --set managerEndpoint=https://app.harness.io/gratis
```

**Expressions:** `<+pipeline.variables.X>` · `<+input>` ·
`<+codebase.branch>`/`<+codebase.commitSha>` (git) · `<+secrets.getValue("ten")>` (secret) ·
`<+pipeline.executionId>` (ID lần chạy) · `<+service.name>`/`<+env.name>` ·
`<+stages.BUILD.spec.artifacts.primary.tag>` (tag từ stage BUILD)


**Step types:** `Run` · `BuildAndPushDockerRegistry` · `Security` (STO) · `K8sRollingDeploy` ·
`K8sCanaryDeploy`/`K8sCanaryDelete` · `K8sBlueGreenDeploy`/`K8sBGSwapServices` ·
`K8sRollingRollback` · `Verify` (AI) · `HarnessApproval` · stage `FeatureFlag`

**Thứ tự pipeline khuyến nghị:**

`CI: Build → Test → STO → Push → CD Staging (rolling + DAST + Chaos) → CD Production (canary → verify → approval → full rollout) → FeatureFlag`

**Checklist production:** tag image = commit SHA · có `rollbackSteps` · stage production có
`HarnessApproval` + `disallowPipelineExecutor` · policy OPA ở chế độ Error ·
secret trong secret manager · pipeline lưu trong Git (PR review) ·
delegate production tách riêng delegate CI/staging
