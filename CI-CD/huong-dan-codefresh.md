# Hướng dẫn Codefresh từ A đến Z 🚀

> Tài liệu tiếng Việt chi tiết về Codefresh — nền tảng CI/CD chạy trên Argo: từ đăng ký tài khoản, viết `codefresh.yml`, build/push/deploy, tới GitOps với Argo CD. Đọc xong là dựng được pipeline thật và đưa app lên Kubernetes.

Chào cậu! Nếu cậu đang làm việc với Kubernetes mà thấy "CI thì một nơi, CD thì Argo CD — hai thứ rời rạc, khó quản lý", thì Codefresh chính là đáp án: **một nền tảng duy nhất gộp CI + CD + GitOps**, và phần GitOps chạy thẳng trên **Argo** (Codefresh là một trong những maintainer chính của dự án Argo mã nguồn mở).

**Chuẩn bị:** tài khoản Git (GitHub/GitLab/Bitbucket), một Kubernetes cluster để deploy thử (minikube, kind, hoặc cluster cloud), Docker Hub hoặc registry khác. Biết sơ về Docker và Kubernetes thì đọc nhanh hơn.

**Cách đọc:** đọc theo thứ tự từ trên xuống. Mọi file YAML trong bài đều chạy được — copy về repo, chỉnh tên repo/cluster cho khớp là chạy.

---

## Mục lục

1. [Codefresh là gì?](#1-codefresh-là-gì)
2. [Lịch sử: từ startup CI tới Argo, rồi về với Octopus Deploy](#2-lịch-sử)
3. [So sánh: Codefresh vs Argo CD thuần vs Jenkins X vs GitHub Actions](#3-so-sánh)
4. [Đăng ký tài khoản](#4-đăng-ký-tài-khoản)
5. [Tour giao diện](#5-tour-giao-diện)
6. [Kiến trúc tổng quan: CI + CD GitOps](#6-kiến-trúc-tổng-quan)
7. [Kết nối Git provider](#7-kết-nối-git-provider)
8. [Kết nối Kubernetes cluster](#8-kết-nối-kubernetes-cluster)
9. [Kết nối container registry](#9-kết-nối-container-registry)
10. [Cú pháp `codefresh.yml` đầy đủ](#10-cú-pháp-codefreshyml-đầy-đủ)
11. [Pipeline mẫu hoàn chỉnh](#11-pipeline-mẫu-hoàn-chỉnh)
12. [GitOps với Argo](#12-gitops-với-argo)
13. [Image enrichment & dashboards](#13-image-enrichment--dashboards)
14. [Helm deployments chi tiết](#14-helm-deployments-chi-tiết)
15. [Runners: SaaS vs Hybrid](#15-runners-saas-vs-hybrid)
16. [Environments & promotions](#16-environments--promotions)
17. [Triggers: Git, cron, registry, webhook](#17-triggers)
18. [Secrets & variables](#18-secrets--variables)
19. [Tests & test reporting](#19-tests--test-reporting)
20. [Notifications (Slack)](#20-notifications-slack)
21. [Codefresh CLI](#21-codefresh-cli)
22. [Pricing & free tier](#22-pricing--free-tier)
23. [Troubleshooting](#23-troubleshooting)
24. [Best practices](#24-best-practices)
25. [FAQ](#25-faq)
26. [Cheat sheet](#26-cheat-sheet)

---

## 1. Codefresh là gì?

**Codefresh** là nền tảng **Continuous Integration / Continuous Delivery (CI/CD)** chạy trên cloud, chuyên cho ứng dụng **container và Kubernetes**. Điểm khác biệt lớn nhất của nó so với Jenkins, GitLab CI hay GitHub Actions:

- **CI chạy trong Docker container**: mỗi step của pipeline chạy trong một container riêng, cô lập, reproducible — không còn cảnh "máy build của anh khác máy build của em".
- **CD là GitOps thật, chạy trên Argo**: Codefresh không tự chế CD riêng mà đóng gói cả bộ **Argo** (Argo CD, Argo Rollouts, Argo Workflows, Argo Events) thành sản phẩm enterprise có tên **Codefresh GitOps** (còn gọi là Argo Runtime). Nói cách khác: xài Codefresh GitOps = xài Argo nhưng có UI quản trị tập trung, dashboard, DORA metrics, quản lý nhiều cluster.
- **Image-centric**: mọi thứ xoay quanh Docker image — build xong image tự động được enrich metadata (commit, branch, build logs, kết quả scan lỗ hổng), dashboard cho thấy image đang chạy ở môi trường nào.

---

## 2. Lịch sử

| Năm | Sự kiện |
|---|---|
| **2014** | Codefresh được thành lập tại Mỹ bởi **Oleg Verhovsky** và **Raziel Tabib**, ban đầu là CI/CD "Docker-native" — một trong những nền tảng đầu tiên build pipeline mà mỗi step là một container. |
| **2019–2022** | Codefresh đầu tư mạnh vào hệ sinh thái Argo, trở thành **maintainer/contributor chính của dự án Argo** (Argo CD, Argo Rollouts, Argo Workflows, Argo Events). |
| **2021–2022** | Ra mắt **Codefresh GitOps** — nền tảng GitOps enterprise xây trên Argo, khẩu hiệu "Argo for the enterprise". Đây cũng là lúc Codefresh chuyển trọng tâm từ CI thuần sang **CD/GitOps**. |
| **27/02/2024** | **Octopus Deploy** (chuẩn công nghiệp về Continuous Delivery, công ty Úc) **mua lại Codefresh**. Thông báo chính thức: gộp CD, CI và GitOps vào một nền tảng. Sau thương vụ, Octopus + Codefresh có hơn 4.000 khách hàng doanh nghiệp, doanh thu trên 60 triệu USD/năm. |
| **2024–nay** | Codefresh tiếp tục phát triển như thương hiệu độc lập trong nhà Octopus, giữ vai trò "Argo enterprise solution": các team như GoodRx, Monday.com, Deloitte dùng Codefresh để deploy an toàn ở quy mô lớn. |

**Vì sao vụ mua lại này quan trọng:** Octopus mạnh về **CD cho VM/workload truyền thống**, Codefresh mạnh về **Kubernetes/GitOps** — gộp lại thành nền tảng CD phủ cả hai thế giới. Octopus cam kết đầu tư tiếp vào **mã nguồn mở Argo** (Codefresh vẫn là maintainer). Về pricing/packaging có thay đổi sau sáp nhập — luôn check trang pricing chính thức (xem [phần 22](#22-pricing--free-tier)).

---

## 3. So sánh

### 3.1. Bảng so sánh nhanh

| Tiêu chí | **Codefresh** | **Argo CD thuần (mã nguồn mở)** | **Jenkins X** | **GitHub Actions** |
|---|---|---|---|---|
| Bản chất | CI + CD + GitOps thương mại, built on Argo | Chỉ CD/GitOps, mã nguồn mở | CI/CD Kubernetes-native mã nguồn mở (dựa trên Jenkins + Tekton) | CI/CD gắn chặt GitHub repo |
| CI | Có, Docker-native (`codefresh.yml`) | Không có CI (phải ghép Jenkins/GitHub Actions...) | Có (Tekton pipelines) | Có, rất mạnh trong hệ GitHub |
| CD/GitOps | Argo CD + Rollouts + Workflows + Events đóng gói sẵn | Tự cài, tự vận hành Argo CD | Tự động qua GitOps (jx boot) | Không có GitOps native (phải tự cài Argo/Flux) |
| Quản lý nhiều cluster | Có, dashboard tập trung | Mỗi instance Argo CD quản lý thủ công, hoặc dùng ApplicationSet | Có, nhưng cấu hình phức tạp | N/A (không phải thế mạnh) |
| DORA metrics, image dashboard | Có sẵn | Không (phải tự build) | Hạn chế | Không |
| Vận hành | SaaS hoặc hybrid (runner trong cluster của cậu) | Tự host 100% | Tự host 100% | SaaS (hoặc self-hosted runner) |
| Giá | Có free tier, trả phí theo user/build | Miễn phí (tốn công vận hành) | Miễn phí (tốn công vận hành) | Free 2.000 phút/tháng (public repo free) |
| Độ khó ban đầu | Trung bình | Trung bình–khó (Argo concepts) | Khó (Jenkins X đã ngừng phát triển tích cực từ 2023) | Dễ |

### 3.2. Khi nào chọn cái nào?

| Hoàn cảnh của cậu | Chọn |
|---|---|
| Deploy lên Kubernetes, muốn một nơi cho cả CI lẫn GitOps, cần DORA metrics/audit | **Codefresh** |
| Team rành Argo, muốn kiểm soát 100% hạ tầng, ngân sách 0 | **Argo CD thuần** |
| Code trên GitHub, pipeline chủ yếu CI đơn giản | **GitHub Actions** (+ Codefresh GitOps cho CD nếu cần — combo rất phổ biến) |
| Dự án mới năm 2026 | **Tránh Jenkins X** (dự án đã chững lại) |

### 3.3. Codefresh CI vs Codefresh GitOps

| | **Codefresh CI (Classic)** | **Codefresh GitOps** |
|---|---|---|
| Định nghĩa pipeline | File `codefresh.yml` trong repo | Argo CD `Application`/`ApplicationSet` (GitOps) |
| Chạy cái gì | Build, test, push image | Sync manifest từ Git lên cluster, rollout strategies |
| Trigger | Git push, PR, cron, registry webhook | Thay đổi trong Git (Argo CD poll/watch) |
| UI | Pipelines, Builds | GitOps Apps, Environments, DORA |
| Quan hệ | CI build ra image → GitOps deploy image đó | CD nhận image từ CI |

**Luồng chuẩn:** Git push → Codefresh CI pipeline chạy (`codefresh.yml`: test → build → push image) → image mới được "enrich" → Argo CD (Codefresh GitOps) phát hiện image/manifest mới → sync lên cluster (có thể qua promotion flow với approval). Hai nửa này nói chuyện với nhau qua **image tag** và **Git**.

---
## 4. Đăng ký tài khoản

1. Vào **codefresh.io** → **Start for free** → đăng nhập bằng **GitHub** (khuyên dùng: vừa tạo account vừa kết nối Git provider luôn, xem [phần 7](#7-kết-nối-git-provider)), GitLab, Bitbucket hoặc email.
2. Trả lời vài câu onboarding cho qua, rồi vào dashboard chính. Tài khoản free dùng được cả hai module **CI** và **GitOps**.

**Lưu ý:** mỗi tài khoản gắn với một **Account** (tổ chức), switch ở góc trên bên phải. API token lấy ở **User Settings → API Keys** — giữ kỹ, dùng cho CLI (xem [phần 21](#21-codefresh-cli)).

---

## 5. Tour giao diện

Sau khi đăng nhập, thanh điều hướng bên trái là "bản đồ" của Codefresh:

### 5.1. Nhóm CI (Classic)

| Mục | Để làm gì |
|---|---|
| **Pipelines** | Danh sách pipeline. Mỗi pipeline gắn với một repo + file `codefresh.yml`. Bấm vào để xem lịch sử build, tạo trigger, sửa variables. |
| **Builds** | Lịch sử tất cả các lần chạy pipeline (build). Bấm vào một build để xem từng step chạy thế nào, log của từng step, image nào được build ra. |
| **Images** | **Dashboard image** — "linh hồn" của Codefresh. Mỗi image build ra hiện ở đây kèm metadata: commit, branch, ai build, lúc nào, đang deploy ở môi trường nào, kết quả scan lỗ hổng. |
| **Helm → Releases / Charts** | Quản lý Helm release đã deploy và chart repository nội bộ của Codefresh (mỗi account có một Helm repo riêng). |
| **Environments (CI)** | Board môi trường kiểu CI: xem image nào đang ở dev/staging/prod, nút **Promote** để đẩy image sang môi trường khác. |

### 5.2. Nhóm GitOps (Argo)

| Mục | Để làm gì |
|---|---|
| **GitOps Apps** | Danh sách Argo CD Applications — mỗi app là một ứng dụng đang được GitOps quản lý, kèm trạng thái Sync/Health. |
| **GitOps → Products** | Gom nhiều Application thành một "sản phẩm" để **promote đồng loạt** qua các môi trường (promotion flow). |
| **Environments (GitOps)** | Xem trạng thái từng môi trường theo góc nhìn GitOps. |
| **DORA Metrics** | Bốn chỉ số DORA: Deployment Frequency, Lead Time, Change Failure Rate, MTTR — tự động tính từ dữ liệu deploy. |

### 5.3. Nhóm cấu hình

| Mục | Để làm gì |
|---|---|
| **Clusters** | Danh sách Kubernetes cluster đã kết nối (xem [phần 8](#8-kết-nối-kubernetes-cluster)). |
| **Pipelines Settings / Account Settings → Integrations** | Kết nối Git provider, Docker registry, Slack, Jira... (xem [phần 7](#7-kết-nối-git-provider), [phần 9](#9-kết-nối-container-registry)). |
| **Shared Configuration** | Kho biến môi trường/secret dùng chung cho nhiều pipeline (xem [phần 18](#18-secrets--variables)). |
| **Runner Management** | Quản lý SaaS runner và hybrid runner (xem [phần 15](#15-runners-saas-vs-hybrid)). |
| **Audit Logs** | Ai đã làm gì, khi nào — cần cho compliance |

**Mẹo:** ô **search/command palette** (`Cmd/Ctrl + K`) — gõ tên pipeline, image, cluster là nhảy tới ngay.

---

## 6. Kiến trúc tổng quan

Sơ đồ luồng dữ liệu của một hệ thống Codefresh hoàn chỉnh:

```
┌─────────────┐   push/PR    ┌──────────────────┐   build/push   ┌────────────────┐
│  Git repo   │ ───────────▶ │  Codefresh CI    │ ──────────────▶ │ Docker Registry │
│ (app code + │              │  (codefresh.yml) │   (image:tag)   │ (Docker Hub/    │
│  codefresh  │              │  test→build→push │                 │  ECR/GCR...)    │
│  .yml)      │              └──────────────────┘                 └───────┬────────┘
└─────────────┘                                                        │ image mới
                                                                       ▼
┌─────────────┐   update tag  ┌──────────────────┐   sync        ┌────────────────┐
│ GitOps repo │ ───────────▶ │ Codefresh GitOps │ ───────────▶ │ K8s clusters   │
│ (manifest/  │  (thủ công    │ (Argo CD +       │  (kubectl     │ (dev/staging/  │
│  helm chart │   hoặc tự     │  Rollouts +      │   apply qua   │  prod)         │
│  config)    │   động)       │  Workflows)      │   Argo CD)    └────────────────┘
└─────────────┘              └──────────────────┘
```

Hai mô hình triển khai hạ tầng:

| Mô hình | Pipeline chạy ở đâu | Khi nào dùng |
|---|---|---|
| **SaaS** | Cloud của Codefresh (agent gọi ngược về cluster, không cần mở firewall inbound) | Bắt đầu nhanh, không muốn quản lý hạ tầng CI |
| **Hybrid** | Trên **runner trong cluster của cậu** (+ Argo Runtime cài trong cluster) | Yêu cầu compliance, code không được rời VPC |

---

## 7. Kết nối Git provider

Codefresh hỗ trợ **GitHub, GitLab, Bitbucket** (cả bản cloud lẫn self-hosted cho gói Enterprise).

### 7.1. Kết nối GitHub (OAuth)

**Account Settings → Integrations → Git → Configure → Add** ở mục GitHub → cấp quyền cho GitHub App của Codefresh (nên chọn "selected repositories"). Xong thì integration hiện **Connected**; Codefresh tự liệt kê repo khi tạo pipeline, đăng ký webhook, và comment kết quả build lên PR (tắt được trong pipeline settings).

### 7.2. Kết nối GitLab / Bitbucket

Tương tự: **Integrations → Git → Configure → Add**, đăng nhập OAuth vào GitLab/Bitbucket và cấp quyền. Với **GitLab self-hosted / Bitbucket Server** (gói Enterprise), cậu nhập thêm URL của server nội bộ.

### 7.3. Kiểm tra nhanh

Vào **Pipelines → Add Pipeline** → bước chọn repo phải thấy repo của mình. Không thấy: kiểm tra quyền OAuth (GitHub: Settings → Applications → Authorized OAuth Apps), hoặc bấm **Sync** trong Integrations.

---

## 8. Kết nối Kubernetes cluster

Cluster trong Codefresh dùng cho hai việc: **deploy** (CD) và **chạy hybrid runner** ([phần 15](#15-runners-saas-vs-hybrid)). Có ba cách kết nối:

### 8.1. Cách 1: bằng Codefresh CLI (nhanh nhất)

```bash
# Đăng nhập CLI (lấy API key ở User Settings → API Keys)
codefresh auth create-context --api-key <API_KEY>

# Thêm cluster hiện tại trong kubeconfig vào Codefresh
codefresh add cluster --kube-context <tên-context> --name staging-cluster
```

### 8.2. Cách 2: từ giao diện web

1. Vào **Clusters → Add Cluster**.
2. Đặt tên (ví dụ `staging`), chọn **provider** (GKE/EKS/AKS/DigitalOcean/generic...).
3. Codefresh hiện một lệnh `kubectl apply -f <url>` — chạy lệnh đó **trong cluster của cậu** (cần quyền cluster-admin). Lệnh này cài một **agent** nhẹ vào cluster, agent sẽ gọi ngược về Codefresh (outbound) nên **không cần mở firewall inbound**.
4. Vài chục giây sau, cluster hiện trạng thái **Connected** trong UI.

### 8.4. Kiểm tra

```bash
codefresh get clusters
# hoặc trong UI: Clusters → bấm vào cluster → xem namespaces, workloads
```

**Lưu ý:** Codefresh phân biệt **"cluster để deploy"** với **"runtime cluster"** (nơi cài Argo Runtime cho GitOps). Với SaaS GitOps, cài agent lên cluster rồi Codefresh quản lý Argo CD applications trỏ về đó.

---

## 9. Kết nối container registry

Pipeline `build` của Codefresh tự động push image vào **internal registry của account** (miễn phí, khỏi login). Nhưng thực tế cậu sẽ cần registry ngoài (Docker Hub, ECR, GCR...). Kết nối ở **Account Settings → Integrations → Docker Registry → Add**:

| Registry | Cần gì |
|---|---|
| **Docker Hub** | Username + password (nên dùng **access token** thay vì password chính) |
| **Amazon ECR** | AWS Access Key ID + Secret (hoặc IAM role nếu chạy hybrid trên EKS) |
| **Google GCR/GAR** | Service account JSON key |
| **Azure ACR** | Service principal / admin credentials |
| **Harbor / Nexus / registry tự host** | URL + username/password |

Sau khi thêm, bấm **Test** để Codefresh thử login — báo **Success** mới yên tâm viết pipeline. Tên integration (ví dụ `dockerhub`) sẽ dùng trong `codefresh.yml` ở trường `registry:`.

---
## 10. Cú pháp `codefresh.yml` đầy đủ

`codefresh.yml` là "trái tim" của Codefresh CI — file YAML đặt ở **thư mục gốc của repo**, mô tả toàn bộ pipeline. Cấu trúc tối giản:

```yaml
version: "1.0"          # bắt buộc, luôn là "1.0"
stages:                # (tùy chọn) nhóm các step thành giai đoạn
  - clone
  - build
  - test
  - deploy
steps:                 # (bắt buộc) các step của pipeline
  main_clone:
    title: Cloning repository
    type: git-clone
    stage: clone
    repo: '${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}'
    revision: '${{CF_REVISION}}'
    git: github
```

### 10.1. `version`, `stages`, `steps`

- **`version: "1.0"`** — bắt buộc, phải có ở đầu file.
- **`stages`** — tên các giai đoạn theo thứ tự chạy. **Các stage chạy tuần tự, các step trong cùng stage chạy song song.** Stage giúp UI hiển thị pipeline gọn thành các cột.
- **`steps`** — map các step, key là tên step (viết dạng `snake_case`), value là cấu hình step. Thứ tự khai báo trong file **không** quyết định thứ tự chạy — thứ tự chạy do `stage` và sự phụ thuộc dữ liệu quyết định.

Thuộc tính chung mọi step đều có:

| Thuộc tính | Ý nghĩa |
|---|---|
| `title` | Tên hiển thị trong UI |
| `description` | Mô tả dài hơn (hiện khi hover) |
| `type` | Loại step: `git-clone`, `freestyle`, `build`, `push`, `deploy`, `helm`, `composition`, `approval`, `parallel` |
| `stage` | Step thuộc stage nào |
| `fail_fast` | `true` = step này fail thì dừng cả pipeline ngay (mặc định `true`) |
| `when` | Điều kiện chạy step (xem 10.8) |
| `on_success` / `on_failure` / `on_termination` | Hook chạy khi step thành công/thất bại/kết thúc |
| `working_directory` | Thư mục làm việc của step (mặc định là volume share) |

### 10.2. Variables — biến

Biến dùng cú pháp **`${{TEN_BIEN}}`**. Có ba nguồn biến:

**1. Biến hệ thống `CF_*`** (Codefresh tự inject mỗi build) — những biến hay dùng nhất:

| Biến | Ý nghĩa | Ví dụ giá trị |
|---|---|---|
| `CF_BRANCH` | Nhánh đang build | `main` |
| `CF_BRANCH_TAG_NORMALIZED` | Tên nhánh đã chuẩn hóa (lowercase, thay ký tự lạ bằng `-`) — an toàn để làm Docker tag | `feature-login` |
| `CF_REVISION` | SHA đầy đủ của commit | `a3f9c1d...` |
| `CF_SHORT_REVISION` | 7 ký tự đầu của SHA | `a3f9c1d` |
| `CF_COMMIT_MESSAGE` | Message của commit | `fix: login bug` |
| `CF_REPO_OWNER` / `CF_REPO_NAME` | Owner và tên repo | `myorg` / `myapp` |
| `CF_PIPELINE_NAME` | Tên pipeline | `myapp-ci` |
| `CF_BUILD_ID` | ID duy nhất của build | `abc123` |
| `CF_PULL_REQUEST_ID` | Số PR (nếu build từ PR) | `42` |
| `CF_PULL_REQUEST_TARGET` | Nhánh đích của PR | `main` |
| `CF_TAG` | Git tag (nếu build từ tag) | `v1.2.0` |
| `CF_VOLUME_PATH` | Đường dẫn volume share giữa các step | `/codefresh/volume` |
| `CF_ACCOUNT` | Tên account | `myorg` |

**2. Biến tự định nghĩa** ở đầu file:

```yaml
version: "1.0"
variables:
  - NODE_VERSION=20
  - APP_NAME=myapp
  - IMAGE_TAG='${{CF_BRANCH_TAG_NORMALIZED}}-${{CF_SHORT_REVISION}}'
steps:
  ...
```

**3. Biến tạo trong lúc chạy** bằng `cf_export`:

```yaml
detect_env:
  title: Detect environment
  image: alpine
  commands:
    - |
      if [ '${{CF_BRANCH}}' = 'main' ]; then cf_export DEPLOY_ENV=production
      else cf_export DEPLOY_ENV=staging; fi
    # các step SAU dùng được ${{DEPLOY_ENV}}
```

`cf_export` có sẵn trong mọi freestyle step. Che giá trị khỏi log (cho secret): `cf_export MY_TOKEN=abc --mask` → log hiện `****`.

### 10.3. Step `git-clone`

Checkout code từ repo. Tài khoản mới có **implicit clone step** (tự clone repo của pipeline), nhưng viết tường minh vẫn rõ ràng hơn:

```yaml
main_clone:
  title: Cloning main repository
  type: git-clone
  stage: clone
  repo: '${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}'  # hoặc 'myorg/myapp'
  revision: '${{CF_REVISION}}'                   # commit/branch/tag cần checkout
  git: github                                   # github | gitlab | bitbucket (tên integration)
  # clone_depth: 1                              # shallow clone cho nhanh (tùy chọn)
```

**Clone nhiều repo** (app + chart repo riêng): thêm step `git-clone` thứ hai trỏ repo khác — mỗi repo checkout vào thư mục riêng trong shared volume.

### 10.4. Step `freestyle` — "xương sống" của pipeline

Freestyle step chạy **bất kỳ lệnh nào trong một Docker container** — đây là step linh hoạt nhất, dùng cho install dependencies, chạy test, lint, script tùy ý:

```yaml
unit_tests:
  title: Running unit tests
  stage: test
  image: node:20-alpine                 # image chạy step (bắt buộc)
  working_directory: '${{main_clone}}'  # chạy lệnh trong thư mục code đã clone
  commands:                            # danh sách lệnh shell
    - npm ci
    - npm run lint
    - npm test
  environment:                         # biến môi trường riêng cho step
    - NODE_ENV=test
    - CI=true
  fail_fast: true
```

Điểm mạnh của freestyle: **shared volume** — mọi freestyle step cùng mount một volume (step A `npm ci` xong, step B dùng ngay `node_modules`); và đây là cách thay thế an toàn cho `docker run` (cậu không bao giờ phải tự `docker run` trong pipeline).

### 10.5. Step `build`

Build Docker image từ Dockerfile. Đây là "cửa ngõ" vào Docker daemon của Codefresh — **không cần tự cài docker-in-docker**:

```yaml
build_app:
  title: Building Docker image
  type: build
  stage: build
  dockerfile: Dockerfile            # đường dẫn Dockerfile (mặc định: Dockerfile ở root)
  image_name: myorg/myapp           # tên image
  tag: '${{CF_BRANCH_TAG_NORMALIZED}}-${{CF_SHORT_REVISION}}'  # tag
  build_arguments:                  # --build-arg
    - NODE_ENV=production
    - APP_VERSION='${{CF_SHORT_REVISION}}'
  target: production                # multi-stage build: chỉ build tới stage này
  buildkit: true                    # bật BuildKit (khuyên dùng: nhanh hơn, cache tốt hơn)
  # no_cache: true                  # bỏ qua cache (mặc định false)
  # cache_from:                     # lấy cache từ image khác
  #   - myorg/myapp:latest
  # platform: linux/amd64,linux/arm64  # multi-arch (xem 11.3)
  registry: dockerhub               # registry integration (mặc định: internal registry)
```

**Build xong image tự động push** vào registry đã chỉ định (internal registry thì không cần `push` step riêng) và được **enrich metadata** (xem [phần 13](#13-image-enrichment--dashboards)).

### 10.6. Step `push`

Đổi tag / đẩy image đã build sang registry khác (không cần nếu chỉ dùng internal registry):

```yaml
push_latest:
  title: Tagging as latest
  type: push
  stage: build
  candidate: '${{build_app}}'   # tham chiếu image output của step build_app
  tag: latest
  registry: dockerhub           # tên integration registry
```

Một `build` step có thể đi kèm nhiều `push` step (mỗi push một tag/registry khác nhau).

### 10.7. Step `deploy`

Deploy image lên Kubernetes cluster đã kết nối — Codefresh tự patch deployment với image mới:

```yaml
deploy_staging:
  title: Deploying to staging
  type: deploy
  stage: deploy
  kind: kubernetes                    # loại deploy: kubernetes
  cluster: staging-cluster            # tên cluster đã kết nối (phần 8)
  namespace: staging
  service: myapp-deployment           # tên K8s Deployment/StatefulSet cần update
  candidate:
    image: myorg/myapp
    tag: '${{CF_BRANCH_TAG_NORMALIZED}}-${{CF_SHORT_REVISION}}'
    registry: dockerhub
  timeout: 600                        # chờ rollout xong, giây
```

> **Lưu ý:** `deploy` step là kiểu deploy truyền thống (imperative). Theo GitOps thì pipeline CI chỉ **update image tag trong GitOps repo** rồi để Argo CD sync — xem [phần 12](#12-gitops-với-argo).

### 10.8. `when` — điều kiện chạy step

```yaml
# Chỉ chạy khi build nhánh main
deploy_prod:
  type: deploy
  when:
    branch:
      only:
        - main
      # ignore:            # hoặc loại trừ
      #   - experimental

# Điều kiện phức tạp với biểu thức
notify_slack:
  image: alpine
  commands:
    - echo "Build failed!"
  when:
    condition:
      all:
        buildFailed: "'${{CF_BUILD_STATUS}}' == 'failure'"
        isMain: "'${{CF_BRANCH}}' == 'main'"

# Chỉ chạy khi build từ git tag
release_step:
  image: alpine
  commands:
    - echo "Releasing ${{CF_TAG}}"
  when:
    condition:
      all:
        isTag: "'${{CF_TAG}}' != ''"
```

Bảng tóm tắt các dạng `when`:

| Cú pháp | Ý nghĩa |
|---|---|
| `when.branch.only: [main]` | Chỉ chạy trên các nhánh liệt kê |
| `when.branch.ignore: [wip-.*]` | Chạy trên mọi nhánh trừ các nhánh liệt kê (hỗ trợ regex) |
| `when.condition.all.<tên>: "<biểu thức>"` | Chạy khi **tất cả** biểu thức đúng |
| `when.condition.any.<tên>: "<biểu thức>"` | Chạy khi **bất kỳ** biểu thức nào đúng |
| Biểu thức | So sánh chuỗi/số: `==`, `!=`, `>`, `<`, `=~` (regex), `&&`, `\|\|` |

### 10.9. Các step type còn lại

**`approval` — cổng phê duyệt thủ công:**

```yaml
approve_prod:
  title: Approve production deploy
  type: approval
  stage: deploy
  description: "Review staging trước khi đẩy lên production"

deploy_prod:
  title: Deploying to production
  type: deploy
  # step này đứng SAU approval trong cùng stage → đợi người bấm Approve mới chạy
```

Khi pipeline tới step `approval`, build chuyển trạng thái **Pending Approval** và gửi thông báo (Slack/email) — người có quyền bấm **Approve/Deny** trong UI.

**`parallel` — chạy nhóm step song song:** gom nhiều step con trong một step `type: parallel` (mỗi step con có `title`/`image`/`commands` riêng) để UI hiển thị gộp và quản lý chung.

> **Nhắc lại:** các step cùng `stage` đã chạy song song mặc định — `parallel` step chỉ cần khi cậu muốn gom nhóm có cấu trúc (và xem gộp trong UI).

**`composition` — chạy nhiều service để test** (thay thế `docker-compose`): khai báo services (app + db...), `composition_candidates` đặt tên service chính, `entry_point` chỉ service chạy lệnh test, `commands` là lệnh test có thể gọi tới các service khác.

**`helm`** — deploy/rollback Helm chart, xem chi tiết ở [phần 14](#14-helm-deployments-chi-tiết).

---
## 11. Pipeline mẫu hoàn chỉnh

### 11.1. Pipeline Node.js: clone → test → build → push → deploy

Giả sử cậu có app Node.js + Dockerfile, muốn: mỗi push lên `main` thì chạy lint + test, build image, push Docker Hub, deploy lên staging; deploy production thì cần approve thủ công.

```yaml
version: "1.0"

stages:
  - clone
  - test
  - build
  - deploy

variables:
  - IMAGE_NAME=myorg/myapp
  - IMAGE_TAG='${{CF_BRANCH_TAG_NORMALIZED}}-${{CF_SHORT_REVISION}}'

steps:
  main_clone:
    title: Cloning repository
    type: git-clone
    stage: clone
    repo: '${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}'
    revision: '${{CF_REVISION}}'
    git: github

  lint_and_unit:
    title: Lint + unit tests
    stage: test
    image: node:20-alpine
    working_directory: '${{main_clone}}'
    commands:
      - npm ci
      - npm run lint
      - npm test -- --reporter=junit --outputFile=junit.xml
    # lưu test report để UI hiển thị (xem phần 19)
    on_success:
      upload_report:
        title: Uploading test report
        image: codefresh/cf-test-reporter
        working_directory: '${{main_clone}}'
        commands:
          - cf-test-reporter junit.xml --working-directory ./

  build_image:
    title: Building Docker image
    type: build
    stage: build
    dockerfile: Dockerfile
    image_name: '${{IMAGE_NAME}}'
    tag: '${{IMAGE_TAG}}'
    buildkit: true
    build_arguments:
      - NODE_ENV=production
    registry: dockerhub

  push_latest:
    title: Tagging image as latest
    type: push
    stage: build
    candidate: '${{build_image}}'
    tag: latest
    registry: dockerhub
    when:
      branch:
        only:
          - main

  deploy_staging:
    title: Deploying to staging
    type: deploy
    stage: deploy
    kind: kubernetes
    cluster: staging-cluster
    namespace: staging
    service: myapp
    candidate:
      image: '${{IMAGE_NAME}}'
      tag: '${{IMAGE_TAG}}'
      registry: dockerhub
    timeout: 600
    when:
      branch:
        only:
          - main

  approve_prod:
    title: Approve production deploy
    type: approval
    stage: deploy
    description: "Staging đã OK. Bấm Approve để deploy lên production."
    when:
      branch:
        only:
          - main

  deploy_prod:
    title: Deploying to production
    type: deploy
    stage: deploy
    kind: kubernetes
    cluster: prod-cluster
    namespace: production
    service: myapp
    candidate:
      image: '${{IMAGE_NAME}}'
      tag: '${{IMAGE_TAG}}'
      registry: dockerhub
    timeout: 900
    when:
      branch:
        only:
          - main
```

**Giải thích luồng:** `clone` checkout code → `test` chạy lint + unit test (fail thì dừng) → `build` image tag `main-a3f9c1d` + tag `latest` (chỉ nhánh main) → `deploy` staging tự động → dừng ở `approve_prod` chờ người bấm Approve → `deploy` production.

### 11.2. Pipeline deploy Helm chart

Giả sử chart nằm trong thư mục `charts/myapp` của cùng repo (hoặc repo chart riêng đã clone):

```yaml
version: "1.0"

stages:
  - clone
  - build
  - deploy

variables:
  - IMAGE_TAG='${{CF_BRANCH_TAG_NORMALIZED}}-${{CF_SHORT_REVISION}}'

steps:
  main_clone:
    title: Cloning repository
    type: git-clone
    stage: clone
    repo: '${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}'
    revision: '${{CF_REVISION}}'
    git: github

  build_image:
    title: Building Docker image
    type: build
    stage: build
    image_name: myorg/myapp
    tag: '${{IMAGE_TAG}}'
    buildkit: true
    registry: dockerhub

  helm_deploy_staging:
    title: Deploying Helm chart to staging
    type: helm
    stage: deploy
    action: install                    # install = upgrade --install
    chart_name: charts/myapp          # đường dẫn chart trong repo
    release_name: myapp
    namespace: staging
    kube_context: staging-cluster     # tên cluster đã kết nối
    helm_version: v3.12.0
    values:
      - charts/myapp/values-staging.yaml
    set:
      - image.tag=${{IMAGE_TAG}}       # override tag vừa build
      - replicaCount=2
    wait: true                        # chờ rollout xong
    timeout: 300
    when:
      branch:
        only:
          - main

  helm_deploy_prod:
    title: Deploying Helm chart to production
    type: helm
    stage: deploy
    action: install
    chart_name: charts/myapp
    release_name: myapp
    namespace: production
    kube_context: prod-cluster
    helm_version: v3.12.0
    values:
      - charts/myapp/values-prod.yaml
    set:
      - image.tag=${{IMAGE_TAG}}
    wait: true
    timeout: 600
    when:
      condition:
        all:
          isReleaseTag: "'${{CF_TAG}}' != ''"   # chỉ deploy prod khi push git tag
```

### 11.3. Pipeline multi-arch build (amd64 + arm64)

Dùng BuildKit `platform` để build một image chạy được cả trên Intel/AMD lẫn ARM (Apple Silicon, AWS Graviton):

```yaml
version: "1.0"

stages:
  - clone
  - build

steps:
  main_clone:
    title: Cloning repository
    type: git-clone
    stage: clone
    repo: '${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}'
    revision: '${{CF_REVISION}}'
    git: github

  build_multiarch:
    title: Building multi-arch image
    type: build
    stage: build
    dockerfile: Dockerfile
    image_name: myorg/myapp
    tag: '${{CF_SHORT_REVISION}}'
    buildkit: true
    platform: linux/amd64,linux/arm64
    build_arguments:
      - NODE_ENV=production
    registry: dockerhub

  push_tags:
    title: Pushing additional tags
    type: push
    stage: build
    candidate: '${{build_multiarch}}'
    tag: latest
    registry: dockerhub
```

**Lưu ý multi-arch:** dùng base image hỗ trợ đa arch (hầu hết image official đều có); tránh `RUN` biên dịch native không tương thích (hoặc dùng `BUILDPLATFORM`/`TARGETARCH` của BuildKit); build 2 arch nên chậm hơn — `buildkit: true` để tận dụng cache.

### 11.4. Caching để pipeline chạy nhanh

Codefresh có **global Docker cache** gắn sẵn vào build node. Tận dụng thêm:

```yaml
# Tận dụng layer cache của chính image trước đó
build_app:
  type: build
  buildkit: true
  cache_from:
    - myorg/myapp:latest     # dùng image latest làm nguồn cache

# Cache dependencies giữa các build bằng shared volume
install_deps:
  title: Installing dependencies (cached)
  image: node:20-alpine
  working_directory: '${{main_clone}}'
  commands:
    # dùng thư mục cache trên volume share, key theo file lock
    - npm ci --cache ${{CF_VOLUME_PATH}}/npm-cache
```

---

## 12. GitOps với Argo

Đây là "món chính" của Codefresh hiện tại. **Codefresh GitOps = Argo CD + Argo Rollouts + Argo Workflows + Argo Events** được Codefresh đóng gói, quản lý tập trung (gọi là **Argo Runtime**), cộng thêm UI enterprise: quản lý nhiều cluster, Products, Environments, DORA metrics.

### 12.1. Tư duy GitOps (nhắc nhanh)

Git là **single source of truth** — Argo CD liên tục so sánh trạng thái Git với cluster, lệch thì **sync** cho khớp (có thể bật auto-sync + self-heal). CI ([phần 11](#11-pipeline-mẫu-hoàn-chỉnh)) chỉ lo **build image**; "đưa image mới lên cluster" = **commit tag mới vào GitOps repo** → Argo CD tự sync. Không ai `kubectl set image` bằng tay nữa.

### 12.2. Tạo GitOps Application đầu tiên

**Cách 1: từ UI** — GitOps Apps → **Add Application** → điền tên, project, source (repo Git + path chứa manifest), destination (cluster + namespace), sync policy (Manual/Automatic + Prune + Self Heal).

**Cách 2: khai báo bằng YAML** (đúng chất GitOps — Argo CD Application là một CRD):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-staging
  namespace: argocd              # namespace cài Argo CD / Runtime
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/gitops-config.git
    targetRevision: HEAD
    path: apps/myapp/staging     # thư mục chứa manifest/helm/kustomize
    helm:
      valueFiles:
        - values-staging.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
  syncPolicy:
    automated:
      prune: true                # xóa resource không còn trong Git
      selfHeal: true             # tự sửa nếu ai đó sửa tay trên cluster
    syncOptions:
      - CreateNamespace=true
```

Apply file này vào cluster có Argo Runtime (hoặc commit vào repo "app-of-apps").

### 12.3. ApplicationSet — quản lý hàng loạt app

Khi có nhiều môi trường/cluster, viết tay từng Application rất mệt. **ApplicationSet** sinh Application tự động từ template + generator:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp-all-envs
  namespace: argocd
spec:
  generators:
    - list:
        elements:
          - env: staging
            cluster: https://staging-cluster.example.com
            namespace: staging
          - env: production
            cluster: https://prod-cluster.example.com
            namespace: production
  template:
    metadata:
      name: 'myapp-{{env}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/myorg/gitops-config.git
        targetRevision: HEAD
        path: 'apps/myapp/{{env}}'
      destination:
        server: '{{cluster}}'
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

Một file duy nhất sinh ra 2 Application (`myapp-staging`, `myapp-production`). Thêm môi trường mới = thêm một element.

### 12.4. Sync waves — điều khiển thứ tự sync

Khi một Application chứa nhiều resource (DB migration → backend → frontend), dùng annotation **sync-wave** để Argo CD apply theo thứ tự:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/sync-wave: "1"   # chạy đầu tiên
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  annotations:
    argocd.argoproj.io/sync-wave: "2"   # chạy sau migration
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  annotations:
    argocd.argoproj.io/sync-wave: "3"   # chạy cuối cùng
```

Số wave nhỏ chạy trước; resource không có annotation mặc định wave `0`. Kết hợp với `argocd.argoproj.io/hook: Sync` cho Job chỉ chạy lúc sync.

### 12.5. Promotion flows — đẩy image qua các môi trường

1. **CI build** image `myapp:main-a3f9c1d` → image xuất hiện trong dashboard **Images**.
2. Trong **GitOps → Products**, tạo **Product** gom các Application của cùng app (staging + prod).
3. **Promote**: chọn image → môi trường đích → Codefresh tạo **commit/PR** update image tag trong GitOps repo của môi trường đích → Argo CD sync. Có thể yêu cầu manual approval, image đã qua scan (không CVE critical), đã đủ test ở môi trường nguồn.
4. Với **Argo Rollouts**, promotion đi kèm chiến lược **canary/blue-green** — ví dụ canary 20% traffic 5 phút → 50% → 100% (cấu hình trong Rollout manifest, Codefresh UI cho promote/pause/abort trực quan).

### 12.6. Kết nối CI với GitOps (image updater pattern)

Pipeline CI sau khi build xong không deploy trực tiếp mà **update tag trong GitOps repo** (pattern GitOps thuần khiết mà Codefresh khuyến nghị):

```yaml
update_gitops:
  title: Updating image tag in GitOps repo
  image: alpine/git
  working_directory: '${{gitops_clone}}'   # đã git-clone repo gitops-config
  commands:
    - apk add --no-cache yq
    - yq eval '.image.tag = "${{IMAGE_TAG}}"' -i apps/myapp/staging/values.yaml
    - git config user.email "ci@codefresh.io" && git config user.name "codefresh-ci"
    - git add apps/myapp/staging/values.yaml
    - git commit -m "deploy staging: ${{IMAGE_TAG}} [skip ci]"
    - git push origin HEAD:main
```

`[skip ci]` để tránh vòng lặp trigger. Argo CD (bật auto-sync) sẽ tự deploy trong 1–3 phút.

---
## 13. Image enrichment & dashboards

### 13.1. Image enrichment là gì?

Mỗi khi `build` step chạy xong, Codefresh tự gắn vào image một "hồ sơ": commit SHA, branch, tag, message, tác giả, pipeline + build ID (+ link log build), thời gian build, Dockerfile đã dùng. Trong **Images dashboard**, bấm vào image là thấy ngay *build từ commit nào, ai build, log ở đâu* — nền tảng cho traceability/audit.

### 13.2. Annotations — gắn metadata tùy ý

Ngoài metadata tự động, cậu có thể gắn thêm key-value tùy ý bằng CLI trong pipeline:

```yaml
annotate_image:
  title: Annotating image
  image: codefresh/cli
  commands:
    - codefresh annotate image myorg/myapp:${{IMAGE_TAG}} jira=PROJ-123
    - codefresh annotate image myorg/myapp:${{IMAGE_TAG}} tested-by=qa-team
```

Annotation hiện trong Images dashboard và dùng được cho filter/tìm kiếm.

### 13.3. Vulnerability scanning — quét lỗ hổng image

Codefresh tích hợp quét lỗ hổng container image: vào **Images** → bấm image → tab **Vulnerabilities** → **Scan** — liệt kê CVE theo mức Critical/High/Medium/Low kèm package bị ảnh hưởng và phiên bản vá gợi ý. Có thể cấu hình **chặn promote/deploy** nếu image có CVE critical. **Nên làm:** quét ở CI ngay sau `build`, fail pipeline nếu có CVE Critical mới — đừng để lọt tới production mới biết.

### 13.4. Đọc Images dashboard như dân pro

Mỗi image hiện: **tag** nào là latest, **created** (khi nào + pipeline nào build), **deployed** (đang chạy ở môi trường nào), **metadata** (commit/branch/PR/tác giả), **annotations** tự gắn, **vulnerabilities** (kết quả quét CVE), và cả **Dockerfile** đã dùng để build — đủ cho audit.

---

## 14. Helm deployments chi tiết

Step `helm` trong Codefresh là wrapper an toàn quanh Helm CLI — cậu không cần tự `helm upgrade` trong freestyle.

### 14.1. Tham số đầy đủ của helm step

```yaml
helm_deploy:
  title: Deploying with Helm
  type: helm
  action: install              # install | upgrade | delete | rollback | lint | test | template | push
  chart_name: charts/myapp     # đường dẫn chart trong repo, hoặc tên chart từ repo (vd: bitnami/nginx)
  chart_repo_url: https://charts.bitnami.com/bitnami   # nếu dùng chart từ repo ngoài
  release_name: myapp
  namespace: staging
  kube_context: staging-cluster   # cluster đã kết nối
  helm_version: v3.12.0
  values:                         # file values (áp dụng theo thứ tự)
    - charts/myapp/values.yaml
    - charts/myapp/values-staging.yaml
  set:                            # --set (ưu tiên cao nhất)
    - image.tag=${{IMAGE_TAG}}
    - replicaCount=3
  set_file:                       # --set-file
    - config.data=./config.json
  wait: true
  timeout: 300
  atomic: true                    # true = fail thì tự rollback về release cũ
  cleanup_on_fail: true
```

### 14.2. Các action hay dùng

| `install` | Cài mới (thực chất là `upgrade --install` — idempotent) |
| `upgrade` | Nâng cấp release đã tồn tại |
| `rollback` | Rollback về revision trước (xem 14.3) |
| `delete` | Gỡ release |
| `lint` | Kiểm tra chart lỗi syntax — nên chạy ở stage test |
| `template` | Render manifest để review, không deploy thật |
| `test` | Chạy helm test hooks |

### 14.3. Rollback

```yaml
helm_rollback:
  title: Rolling back Helm release
  type: helm
  stage: deploy
  action: rollback
  release_name: myapp
  namespace: production
  kube_context: prod-cluster
  helm_version: v3.12.0
  revision: 0              # 0 = về revision trước đó; hoặc ghi số revision cụ thể
  wait: true
  timeout: 300
  when:
    condition:
      all:
        rollbackRequested: "'${{ROLLBACK}}' == 'true'"
```

Kết hợp với trigger thủ công (manual trigger với biến `ROLLBACK=true`) là cậu có nút "rollback khẩn cấp" trong UI.

### 14.4. Quản lý chart với Codefresh Helm repo

Mỗi account có một **private Helm chart repository**: push chart lên bằng helm step (`action: push`), rồi dùng lại qua `chart_repo_url` trỏ về repo của account. Trong UI **Helm → Charts** thấy các version đã push; **Helm → Releases** thấy mọi release đang chạy trên các cluster.

---

## 15. Runners: SaaS vs Hybrid

**Runner** là nơi pipeline của cậu thực sự chạy (chạy container của các step).

### 15.1. So sánh

| | **SaaS Runner** (mặc định) | **Hybrid Runner** |
|---|---|---|
| Chạy ở đâu | Cloud của Codefresh | Trong **Kubernetes cluster của cậu** |
| Cài đặt | Không cần làm gì | Cài agent qua Helm chart từ UI **Runner Management → Add Runner** |
| Code có rời khỏi hạ tầng cậu không | Có (code được clone lên cloud Codefresh) | **Không** — code, secret, image layer ở lại trong VPC của cậu |
| Truy cập tài nguyên nội bộ | Không (trừ khi mở firewall) | Có — truy cập trực tiếp DB, registry, Vault nội bộ |
| Tùy biến tài nguyên | Theo gói | Tự chọn node size, số lượng |
| Khi nào dùng | Bắt đầu nhanh, project mã nguồn mở/nội bộ không nhạy cảm | **Yêu cầu compliance** (tài chính, y tế, chính phủ), monorepo nặng cần máy build khỏe, cần truy cập mạng nội bộ |

### 15.2. Cài hybrid runner

**Account Settings → Runner Management → Add Runner** → đặt tên (ví dụ `vpc-runner`) → copy lệnh Helm do UI sinh ra, chạy trong cluster của cậu (namespace riêng, kèm token UI cấp). Runner hiện **Connected** là xong; khi tạo/sửa pipeline, ở phần **Runner** chọn `vpc-runner` thay vì default.

### 15.3. Chọn runner cho từng pipeline

Trong pipeline settings (UI) hoặc khi chạy bằng CLI:

```bash
codefresh run myapp-ci --runner vpc-runner --branch main
```

**Khuyên dùng thực tế:** pipeline build/test thông thường → SaaS runner cho rẻ và khỏi quản lý; pipeline nào đụng tới secret nhạy cảm, mạng nội bộ, hoặc cần máy build khủng → hybrid runner.

---

## 16. Environments & promotions

### 16.1. Environment board (CI)

**Environments** là bảng kanban: mỗi cột là một môi trường (dev → staging → prod), mỗi thẻ là image đang chạy ở đó — nhìn một phát biết staging/prod đang chạy tag nào, prod có tụt hậu không.

### 16.2. Promote image giữa các môi trường

Có hai cách promote:

**Cách 1: bấm nút trong UI** — ở Environment board, kéo/bấm **Promote** trên image ở staging → chọn môi trường đích → Codefresh deploy image đó sang môi trường đích (ghi log ai promote, khi nào).

**Cách 2: promotion qua pipeline** — dùng `deploy`/`helm` step trỏ tới môi trường đích với cùng tag (như ví dụ 11.1: `deploy_prod` dùng cùng `IMAGE_TAG` đã test ở staging — **đúng image đã test** được đẩy lên prod, không build lại).

### 16.3. Promotion policies (GitOps)

Trong Codefresh GitOps, promotion được quản lý chặt hơn qua **Products**:

1. Tạo Product gom các Argo Application của cùng app.
2. Định nghĩa **promotion flow**: staging → (manual approval) → production.
3. Khi promote, Codefresh tạo commit/PR update image tag trong GitOps repo của môi trường đích (xem 12.6), có thể yêu cầu:
   - Manual approval (người bấm duyệt)
   - Image đã qua scan, không có CVE critical
   - Đã chạy đủ test ở môi trường nguồn

Luồng này cho cậu **audit trail đầy đủ**: ai approve, image nào, từ đâu tới đâu, lúc nào.

---
## 17. Triggers

Trigger quyết định **khi nào pipeline chạy** — cấu hình ở **Pipeline Settings → Triggers** (không nằm trong `codefresh.yml`):

| Loại trigger | Kích hoạt khi | Cấu hình chính |
|---|---|---|
| **Git trigger** | Push, PR opened/synchronized, tag push | Provider, repo, **events**, `branchRegex` (ví dụ `^main$`, `^release/.*`) |
| **Cron trigger** | Theo lịch | Biểu thức cron (ví dụ `0 2 * * *` = 2h sáng mỗi ngày), branch, biến |
| **Docker registry trigger** | Image mới được push lên registry | Registry integration, `imageRegex` (ví dụ `myorg/base-image:.*`) |
| **Webhook thủ công** | Gọi HTTP | Codefresh cấp URL + token; gọi kèm biến tùy ý |

**Ví dụ webhook:** lấy URL ở Pipeline Settings → Triggers → Add webhook, rồi:

```bash
curl -X POST 'https://g.codefresh.io/api/webhooks/<ID>?token=<TOKEN>' \
  -H 'Content-Type: application/json' \
  -d '{"IMAGE_TAG": "v1.2.3", "DEPLOY_ENV": "staging"}'
```

Biến gửi kèm dùng được ngay trong pipeline qua `${{IMAGE_TAG}}`. **Mẹo:** dùng `branchRegex` cho một pipeline phục vụ nhiều nhánh (`^(main|release/.*)$`); cron trigger cho nightly build/dependency check; registry trigger cho pipeline "base image" (base image mới → mọi app tự rebuild).

---

## 18. Secrets & variables

Thứ tự ưu tiên biến (cao → thấp): **trigger/event variables** → **pipeline variables (UI)** → **shared config** → **`variables:` trong YAML** → **biến hệ thống `CF_*`**.

| Cách | Dùng khi |
|---|---|
| **Pipeline variables** (Pipeline Settings → Variables) | Biến riêng của một pipeline |
| **Shared configuration** (Account Settings → Shared Configuration) | Secret/biến dùng chung nhiều pipeline (API key, token) — tạo context rồi gắn vào pipeline |
| **Encrypted variables** | Bật **Encrypt** khi tạo biến — UI chỉ hiện `****` |
| `cf_export ... --mask` | Secret tạo ra trong lúc chạy — log che thành `****` |

**Nguyên tắc:** không hardcode secret vào `codefresh.yml` (file này nằm trong Git!); rotate token định kỳ theo least privilege; với hybrid runner + Vault nội bộ, secret không qua cloud Codefresh.

---

## 19. Tests & test reporting

Chạy test trong Codefresh = freestyle step với image chứa runtime tương ứng. Điểm cộng: **test report hiển thị đẹp trong UI** thay vì mò trong log.

```yaml
unit_tests:
  title: Unit tests with JUnit report
  image: node:20-alpine
  working_directory: '${{main_clone}}'
  commands:
    - npm ci
    - npm test -- --reporter=junit --outputFile=junit.xml
  on_success:
    publish_report:
      title: Publishing test report
      image: codefresh/cf-test-reporter
      working_directory: '${{main_clone}}'
      commands:
        - cf-test-reporter junit.xml --working-directory ./
```

- `cf-test-reporter` nhận **JUnit XML** (jest, pytest, mocha, go test... đều xuất được) → tab **Tests** trong build hiện số test pass/fail, test nào fail, thời gian chạy.
- Test fail → step fail → pipeline dừng — không build image từ code hỏng. Test nặng: tách `parallel` step hoặc stage riêng; cần service phụ (DB, Redis) thì dùng `composition`.

---

## 20. Notifications (Slack)

Kết nối Slack ở **Account Settings → Integrations → Slack** (qua OAuth hoặc incoming webhook URL), rồi thêm step notify:

```yaml
notify_slack:
  title: Notify Slack
  image: codefresh/cli
  commands:
    - 'codefresh notify slack --channel "#deploys" --text "✅ *${{CF_PIPELINE_NAME}}*: `${{IMAGE_TAG}}` đã deploy lên staging"'
  when:
    branch: {only: [main]}
```

Hoặc dùng **webhook URL trực tiếp** (biến `${{SLACK_WEBHOOK_URL}}` lưu trong shared config đã encrypt):

```yaml
notify_slack_webhook:
  title: Notify via webhook
  image: curlimages/curl
  commands:
    - >
      curl -X POST -H 'Content-Type: application/json'
      --data '{"text":"Build ${{CF_BUILD_ID}} của *${{CF_REPO_NAME}}*: ${{CF_BUILD_STATUS}}"}'
      '${{SLACK_WEBHOOK_URL}}'
```

**Nên notify khi nào:** deploy production thành công/thất bại, pipeline fail trên nhánh main, approval đang chờ duyệt. Đừng notify mọi build — team sẽ mute kênh của cậu.

---

## 21. Codefresh CLI

```bash
npm install -g codefresh        # hoặc tải binary từ trang download của codefresh.io
codefresh auth create-context --api-key <API_KEY>   # lấy ở User Settings → API Keys
```

Lệnh hay dùng:

| Lệnh | Tác dụng |
|---|---|
| `codefresh get pipelines` | Liệt kê pipeline |
| `codefresh run <pipeline> --branch main` | Chạy pipeline thủ công |
| `codefresh run <pipeline> --branch main -v IMAGE_TAG=v1.2.3` | Chạy kèm biến |
| `codefresh get builds --pipeline <pipeline> --limit 5` | Xem build gần nhất |
| `codefresh logs <build-id>` | Xem log build |
| `codefresh get clusters` / `codefresh add cluster` | Quản lý cluster |
| `codefresh annotate image <image:tag> key=value` | Gắn annotation cho image |
| `codefresh create context` | Tạo shared config context |

CLI dùng được trong script, cron job, hoặc ngay trong pipeline (image `codefresh/cli`).

---

## 22. Pricing & free tier

> Giá thay đổi sau khi Octopus Deploy mua lại Codefresh — bảng dưới tổng hợp theo thông tin công khai gần nhất (2026). **Luôn kiểm tra codefresh.io/pricing trước khi chốt**, vì hãng có thể điều chỉnh.

**Codefresh CI (Classic):**

| Gói | Giá | Giới hạn chính |
|---|---|---|
| **Basic (Free)** | $0 | Tối đa 3 user, builds không giới hạn, 1 concurrent build, private Docker registry + Helm repo, lưu dữ liệu 1 tháng |
| **Pro** | ~$34/tháng (trả theo năm) | Tối đa 10 user, thêm Kubernetes/Helm dashboard, live pipeline debugging, lưu dữ liệu 6 tháng |
| **Enterprise** | Liên hệ | User/build không giới hạn, SaaS + Hybrid + On-premise, SSO/SAML/RBAC, hỗ trợ sau firewall |

**Codefresh GitOps (Argo):**

| Gói | Giá | Giới hạn chính |
|---|---|---|
| **Developer (Free)** | $0 | 1.000 MTU, 2 sources (cluster), destinations không giới hạn |
| **Team** | ~$120/tháng | 10.000 MTU, sources không giới hạn |
| **Business** | Liên hệ | MTU tùy chỉnh, replay dữ liệu lịch sử, premium support |

(MTU = đơn vị đo mức sử dụng GitOps theo cách tính của Codefresh.)

**Chọn gói thế nào:** cá nhân/học tập → free tier là đủ (CI free không giới hạn build rất hào phóng); team nhỏ → Team GitOps + Pro CI; doanh nghiệp cần compliance → Enterprise + hybrid runner.

---

## 23. Troubleshooting

| Triệu chứng | Nguyên nhân thường gặp | Cách fix |
|---|---|---|
| Pipeline không chạy khi push | Webhook chưa đăng ký / repo chưa gắn pipeline | Pipeline Settings → kiểm tra trigger; Integrations → Git → Sync; kiểm tra webhook trong repo Settings |
| `git-clone` fail: permission denied | OAuth token hết hạn / thiếu quyền repo | Re-connect Git integration; kiểm tra quyền GitHub App với repo private |
| `${{VAR}}` rỗng | Sai tên biến / `cf_export` ở step chạy sau | Kiểm tra chính tả; `cf_export` chỉ có tác dụng với các step **sau** step export |
| `build` fail: Dockerfile not found | Sai `dockerfile:` path | Kiểm tra đường dẫn tương đối từ root repo; xem log step clone |
| Build chậm bất thường | Không bật BuildKit / cache miss | `buildkit: true` + `cache_from` trỏ image trước đó |
| `push` fail: unauthorized | Credential registry sai / token hết hạn | Integrations → Docker Registry → Test lại (Docker Hub dùng access token) |
| `deploy`/`helm` fail: cluster not found | Sai tên `cluster:`/`kube_context:` | Tên phải khớp đúng trong Clusters (`codefresh get clusters` để đối chiếu) |
| Helm release stuck | Release trước fail giữa chừng | Chạy lại với `atomic: true`, hoặc rollback bằng tay |
| Argo CD không sync | GitOps repo chưa update / Application sai path | Kiểm tra commit đã vào đúng path; xem trạng thái Sync và log lỗi trong GitOps Apps |

**Debug chuẩn:** Builds → bấm build fail → xem step đỏ → đọc log từ dưới lên → sửa → **Re-run** từ step fail (không cần chạy lại từ đầu).

---

## 24. Best practices

1. **Một repo, một `codefresh.yml` ở root** — đừng rải nhiều file pipeline lung tung.
2. **Tag image bất biến:** `${{CF_BRANCH_TAG_NORMALIZED}}-${{CF_SHORT_REVISION}}` — đừng deploy tag `latest` lên production.
3. **Fail fast:** test/lint chạy trước build — code hỏng thì đừng tốn tiền build image.
4. **Tách stage rõ ràng:** `clone → test → build → deploy`; step cùng stage tự song song.
5. **Secret không vào Git:** chỉ qua shared config / encrypted variables / `--mask`.
6. **CI build, GitOps deploy:** pipeline CI update tag trong GitOps repo, để Argo CD sync.
7. **Mọi deploy production qua approval** hoặc promotion flow có người duyệt.
8. **Quét lỗ hổng image ở CI**, chặn image có CVE critical lên prod.
9. **Bật BuildKit + cache** cho mọi `build` step; **hybrid runner** cho pipeline chạm secret/mạng nội bộ.
10. **Gắn annotation** (Jira ticket, người request) cho image quan trọng — lúc incident truy vết nhanh hơn hẳn.

---

## 25. FAQ

**Hỏi: Codefresh có miễn phí không?**
Đáp: Có. CI gói Basic free (3 user, builds không giới hạn); GitOps gói Developer free (1.000 MTU). Đủ cho cá nhân và team nhỏ thử nghiệm nghiêm túc.

**Hỏi: Codefresh có thay thế được Argo CD không?**
Đáp: Không "thay thế" mà "đóng gói + mở rộng" — Codefresh GitOps chạy Argo CD bên dưới, cộng UI đa cluster, Products, promotion flows, DORA metrics. Manifest Argo của cậu vẫn tương thích.

**Hỏi: Dùng GitHub Actions cho CI rồi, có dùng Codefresh GitOps cho CD được không?**
Đáp: Được, combo rất phổ biến: GitHub Actions build/push image → update tag trong GitOps repo → Codefresh (Argo CD) sync.

**Hỏi: Pipeline của tớ cần truy cập database trong VPC, làm sao?**
Đáp: Dùng **hybrid runner** cài trong VPC — pipeline chạy trong mạng nội bộ của cậu. Ngược lại, với SaaS runner thì code được clone lên cloud Codefresh trong lúc chạy (xong thì xóa) — chính sách không cho phép thì bắt buộc hybrid.

**Hỏi: Khác gì giữa `deploy` step và GitOps?**
Đáp: `deploy` step là imperative (pipeline ra lệnh deploy trực tiếp). GitOps là declarative (pipeline chỉ update Git, Argo CD sync) — audit trail tốt hơn và là hướng Codefresh đầu tư.

---

## 26. Cheat sheet

### Biến hệ thống nhớ nhanh

```
${{CF_BRANCH}}  ${{CF_SHORT_REVISION}}  ${{CF_TAG}}  ${{CF_PULL_REQUEST_ID}}
${{CF_REPO_OWNER}}/${{CF_REPO_NAME}}    ${{CF_BUILD_ID}}    ${{CF_VOLUME_PATH}}
```

### CLI nhớ nhanh

```bash
codefresh auth create-context --api-key <KEY>
codefresh run <pipeline> --branch main
codefresh get builds --pipeline <pipeline> --limit 5
codefresh logs <build-id>
codefresh get clusters
```

### Helm deploy nhớ nhanh

```yaml
type: helm
action: install          # install | upgrade | rollback | delete | lint | test
chart_name: charts/myapp
release_name: myapp
namespace: staging
kube_context: staging-cluster
set:
  - image.tag=${{CF_SHORT_REVISION}}
wait: true
```

### GitOps Application nhớ nhanh

Xem mẫu đầy đủ ở [12.2](#122-tạo-gitops-application-đầu-tiên) — cheat: `Application` (argoproj.io/v1alpha1) với `source.repoURL` + `path`, `destination.server` + `namespace`, `syncPolicy.automated: {prune: true, selfHeal: true}`.

---

**Chúc cậu build xanh, deploy mượt! 🚀**
