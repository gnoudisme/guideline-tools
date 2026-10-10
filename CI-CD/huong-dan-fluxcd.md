# Hướng dẫn toàn diện về Flux CD

Flux CD là bộ công cụ GitOps cho Kubernetes: khai báo trạng thái mong muốn của hạ tầng và ứng dụng bằng Git, rồi để Flux tự động đồng bộ vào cluster.

---

## Mục lục

1. [Flux là gì](#1-flux-là-gì)
2. [So sánh Flux vs Argo CD](#2-so-sánh-flux-vs-argo-cd)
3. [Cài đặt](#3-cài-đặt)
4. [GitRepository CRD](#4-gitrepository-crd)
5. [Kustomization CRD](#5-kustomization-crd)
6. [HelmRelease CRD](#6-helmrelease-crd)
7. [Image Automation](#7-image-automation)
8. [Notifications](#8-notifications)
9. [Multi-tenancy](#9-multi-tenancy)
10. [Quản lý Secrets với SOPS](#10-quản-lý-secrets-với-sops)
11. [Dự án thực tế: bootstrap Flux + deploy app](#11-dự-án-thực-tế-bootstrap-flux--deploy-app)
12. [Migration từ Flux v1 sang v2](#12-migration-từ-flux-v1-sang-v2)
13. [Best practices](#13-best-practices)
14. [Troubleshooting](#14-troubleshooting)
15. [FAQ và Cheat sheet](#15-faq-và-cheat-sheet)

---

## 1. Flux là gì

**Flux CD** (thường gọi tắt là Flux) là một bộ công cụ GitOps mã nguồn mở dành cho Kubernetes. Dự án được CNCF (Cloud Native Computing Foundation) công nhận **graduated** — mức cao nhất, ngang hàng với Kubernetes, Prometheus, Argo. "Graduated" nghĩa là dự án đã chứng minh được độ ổn định, cộng đồng lớn, được dùng rộng rãi trong production.

### 1.1. GitOps theo cách của Flux

Ý tưởng cốt lõi của GitOps rất đơn giản:

1. **Git là nguồn chân lý duy nhất (single source of truth)**: mọi manifest Kubernetes, cấu hình Helm, policy... đều nằm trong Git repo.
2. **Pull-based**: không phải CI/CD push thay đổi vào cluster, mà là Flux *chạy bên trong cluster*, liên tục pull từ Git và áp dụng vào cluster.
3. **Tự động reconciliation**: Flux so sánh trạng thái mong muốn (trong Git) với trạng thái thực tế (trong cluster) theo chu kỳ, tự động sửa sai lệch (drift).

Lợi ích: Git cung cấp version control, audit log, rollback dễ dàng; cluster không cần mở port inbound cho CI/CD; mọi thay đổi đều qua pull request.

### 1.2. Kiến trúc Flux v2

Flux v2 là một tập hợp các **controller** (operator) chạy trong namespace `flux-system`, mỗi controller phụ trách một loại tài nguyên (CRD):

| Controller | CRD quản lý | Nhiệm vụ |
|---|---|---|
| **source-controller** | GitRepository, OCIRepository, HelmRepository, HelmChart, Bucket | Tải artifact từ nguồn (Git, OCI registry, Helm repo, S3 bucket) và lưu dưới dạng tarball nội bộ |
| **kustomize-controller** | Kustomization | Áp dụng manifest Kustomize/YAML vào cluster, chạy health check, prune tài nguyên thừa |
| **helm-controller** | HelmRelease | Render và cài đặt/nâng cấp Helm chart theo spec HelmRelease |
| **notification-controller** | Provider, Alert, Receiver | Gửi thông báo (Slack, Discord, Telegram...) về sự kiện; nhận webhook từ Git provider để trigger sync ngay |
| **image-reflector-controller** | ImageRepository, ImagePolicy | Quét container registry, trích xuất danh sách tag, áp policy chọn tag (vd: tag semver mới nhất) |
| **image-automation-controller** | ImageUpdateAutomation | Tự động cập nhật file YAML trong Git repo khi có image tag mới thỏa policy, rồi commit + push |

Luồng dữ liệu điển hình:

```
Git repo ──(1)──▶ GitRepository ──(2)──▶ Kustomization ──(3)──▶ resources trong cluster
                                     │
                                     └──▶ HelmRelease ──▶ Helm chart ──▶ resources

Container registry ──▶ ImageRepository ──▶ ImagePolicy ──▶ ImageUpdateAutomation ──▶ commit vào Git repo
```

(1) source-controller clone/pull Git repo theo `interval`.
(2) kustomize-controller lấy artifact, build Kustomize, `kubectl apply` vào cluster.
(3) helm-controller nhận spec HelmRelease, chạy `helm upgrade --install` tương đương.

Tất cả controller đều tuân theo mô hình reconciliation: vòng lặp `fetch → so sánh → áp dụng → báo trạng thái` chạy theo `interval` định sẵn. Trạng thái được ghi vào trường `.status` của CR, xem được bằng `kubectl describe` hoặc `flux get`.

### 1.3. Flux hoạt động như thế nào trong thực tế

1. Developer push code → CI build image → push image lên registry với tag mới.
2. image-reflector-controller phát hiện tag mới → image-automation-controller cập nhật tag trong Git repo và push commit.
3. source-controller thấy commit mới → tải artifact mới.
4. kustomize-controller / helm-controller áp dụng thay đổi vào cluster, chạy health check.
5. notification-controller gửi thông báo thành công/thất bại vào Slack.

Toàn bộ quá trình không cần ai chạy `kubectl apply` thủ công.

---

## 2. So sánh Flux vs Argo CD

Hai công cụ GitOps phổ biến nhất cho Kubernetes. Bảng so sánh chi tiết:

| Tiêu chí | Flux CD | Argo CD |
|---|---|---|
| **Giao diện UI** | Không có UI chính thức (chỉ CLI + `flux` dashboard cộng đồng như Weave GitOps đã dừng phát triển). Quan sát qua `kubectl`, `flux get`, Prometheus metrics | Có Web UI đầy đủ, đẹp: cây tài nguyên trực quan, xem diff, sync thủ công, xem log. Đây là điểm mạnh lớn nhất của Argo CD |
| **Cách cài đặt** | `flux bootstrap` — một lệnh duy nhất cài controller + tạo cấu trúc GitOps repo + commit vào Git. Rất nhanh | Cài manifest vào cluster (`kubectl apply`), sau đó cấu hình Application/ AppProject thủ công hoặc qua UI/CLI |
| **Triết lý GitOps** | Thuần GitOps: mọi thứ khai báo trong Git, kể cả cấu hình Flux. CLI chỉ để bootstrap và debug | Cho phép thao tác qua UI/CLI (sync thủ công, sửa trực tiếp), dễ "lệch" khỏi Git nếu không kỷ luật |
| **Multi-tenancy** | Hỗ trợ native rất mạnh: mỗi tenant có ServiceAccount + RBAC riêng, giới hạn namespace, Kustomization impersonation. Thiết kế cho nhiều team dùng chung cluster | Có AppProject để giới hạn, nhưng RBAC phức tạp hơn, thường cần cấu hình thêm |
| **Hỗ trợ Helm** | HelmRelease CRD native, helm-controller quản lý vòng đời chart như tài nguyên K8s | Hỗ trợ qua Application, quản lý vòng đời Helm (rollback, test) kém native hơn |
| **Image automation** | Có sẵn: ImageRepository + ImagePolicy + ImageUpdateAutomation tự động bump tag trong Git | Không có sẵn, phải dùng Argo Image Updater (component riêng) |
| **Kustomize** | Kustomization CRD native, hỗ trợ sâu (patches, dependsOn, health check) | Hỗ trợ trong Application, ít tùy chọn tinh chỉnh |
| **Notifications** | notification-controller native: Provider/Alert cho Slack, Discord, Telegram, PagerDuty, webhook... | Có notification engine riêng (argocd-notifications), cấu hình qua ConfigMap, hơi rườm rà |
| **Độ phức tạp** | Nhẹ, ít component, dễ vận hành. Không có UI nên quan sát qua CLI | Nặng hơn (server, repo-server, controller, redis...), nhưng dễ tiếp cận nhờ UI |
| **Mô hình triển khai** | Pull-only. Cluster chủ động pull, không cần mở inbound | Pull (mặc định) + hỗ trợ push qua webhook; UI server cần expose nếu dùng ngoài cluster |
| **Hệ sinh thái CNCF** | Graduated, tích hợp sâu với SOPS, Kustomize, Helm | Graduated, hệ sinh thái plugin phong phú |

**Khi nào chọn Flux:** muốn GitOps thuần khiết (mọi thứ trong Git), nhiều team dùng chung cluster, cần image automation và SOPS có sẵn, ưa CLI/khai báo, không cần UI.

**Khi nào chọn Argo CD:** cần Web UI trực quan để quan sát/debug/demo, muốn sync thủ công từng app và xem diff trước khi apply, hoặc team đã quen hệ sinh thái Argo (Argo Rollouts cho progressive delivery).

---

## 3. Cài đặt

### 3.1. Yêu cầu

- Một Kubernetes cluster đang chạy (phiên bản được hỗ trợ: kiểm tra ma trận tương thích trên trang release của Flux, thường là 3 phiên bản K8s mới nhất). Có thể dùng kind, k3s, minikube để thử nghiệm.
- `kubectl` đã cấu hình, có quyền cluster-admin.
- GitHub/GitLab account (cho `flux bootstrap`), hoặc một Git server bất kỳ (cho bootstrap generic).
- Flux CLI — cài đặt theo hệ điều hành:

```bash
# macOS
brew install fluxcd/tap/flux

# Linux
curl -s https://fluxcd.io/install.sh | sudo bash

# Windows (winget)
winget install FluxCD.Flux
```

Kiểm tra phiên bản:

```bash
flux --version
# flux version 2.x.x
```

### 3.2. Kiểm tra tiền điều kiện: `flux check --pre`

Trước khi cài, luôn chạy kiểm tra:

```bash
flux check --pre
```

Ví dụ output khi đạt:

```
► checking prerequisites
✔ Kubernetes 1.29.x >=1.25.0-0
✔ prerequisites checks passed
```

### 3.3. Bootstrap với GitHub (khuyên dùng)

`flux bootstrap github` làm tất cả trong một lệnh: cài các controller vào cluster, tạo (hoặc dùng) GitHub repo, tạo deploy key, commit file cấu hình Flux vào repo, và thiết lập sync từ repo đó.

**Bước 1: Tạo GitHub Personal Access Token (PAT)**

Vào GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token. Quyền cần thiết:
- `repo` (đầy đủ: repo:status, repo_deploy, public_repo, repo:invite, security_events)

> Nếu dùng fine-grained token: cần quyền Contents (read/write), Pull requests (read/write), Administration (read/write cho deploy keys), Metadata (read-only).

Export token ra biến môi trường (không paste token vào lịch sử lệnh một cách lộ liễu — dùng biến môi trường):

```bash
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-github-username>
```

**Bước 2: Chạy bootstrap**

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production \
  --personal   # repo cá nhân; bỏ flag nếu repo thuộc organization
```

`--path` cho phép 1 repo quản lý nhiều cluster (`clusters/staging`, `clusters/production`...).

**Bước 3: Xác minh**

```bash
# Xem các component Flux đã cài
flux get all

# Kiểm tra trạng thái sync
flux get sources git
flux get kustomizations
```

Sau bootstrap, cấu trúc repo `fleet-infra` sẽ như sau:

```
fleet-infra/
├── clusters/
│   └── production/
│       ├── flux-system/          # do flux bootstrap tự sinh: gotk-components.yaml, gotk-sync.yaml, kustomization.yaml
│       └── ...
├── apps/
└── infrastructure/
```

File `clusters/production/flux-system/gotk-sync.yaml` chứa GitRepository + Kustomization trỏ về chính repo này — tức là Flux tự quản lý chính nó bằng GitOps (self-managed).

> **Lưu ý quan trọng:** không sửa tay file trong `flux-system/` trên cluster. Mọi thay đổi cấu hình Flux (ví dụ đổi interval) đều thực hiện bằng cách sửa file trong Git rồi push — Flux sẽ tự áp dụng.

### 3.4. Bootstrap với GitLab

```bash
export GITLAB_TOKEN=<your-token>   # token cần quyền api
flux bootstrap gitlab \
  --owner=<group-hoặc-username> \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/production \
  --personal   # bỏ nếu là group
```

### 3.5. Bootstrap với Git server bất kỳ (generic)

Dùng khi Git server là self-hosted (Gitea, Bitbucket...), hoặc muốn tự quản lý SSH key:

```bash
# Tạo SSH key riêng cho Flux (nếu chưa có)
ssh-keygen -t ed25519 -C "flux" -f ./flux-key -N ""

flux bootstrap git \
  --url=ssh://git@gitea.example.com:2222/platform/fleet-infra.git \
  --branch=main \
  --path=clusters/production \
  --private-key-file=./flux-key
```

Sau khi chạy, Flux in ra **public key** — thêm nó vào Git server với quyền read/write (deploy key), Flux sẽ chờ rồi tiếp tục.

Với HTTPS + token:

```bash
flux bootstrap git \
  --url=https://gitea.example.com/platform/fleet-infra.git \
  --branch=main \
  --path=clusters/production \
  --username=git \
  --password=$GIT_TOKEN
```

---

## 4. GitRepository CRD

`GitRepository` (do source-controller quản lý) định nghĩa *nguồn* Flux sẽ pull: URL repo, nhánh/tag/commit, chu kỳ pull, và secret chứa credential.

Ví dụ đầy đủ:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: fleet-infra
  namespace: flux-system
spec:
  # Chu kỳ pull: kiểm tra commit mới mỗi 1 phút
  interval: 1m0s
  # URL repo Git
  url: https://github.com/example-org/fleet-infra
  # Secret chứa credential (tạo bằng: kubectl create secret generic ...)
  secretRef:
    name: fleet-infra-auth
  # Ref muốn theo dõi: nhánh, tag, semver hoặc commit cụ thể
  ref:
    branch: main
    # tag: v1.2.3
    # semver: ">=1.0.0 <2.0.0"
    # commit: abc123def456
  # Bỏ qua file không liên quan khi tính checksum artifact
  ignore: |
    *.tmp
```

Các trường quan trọng:

| Trường | Ý nghĩa |
|---|---|
| `spec.interval` | Chu kỳ pull repo. Ngắn (30s–1m) cho môi trường dev, dài (5m–10m) cho production để giảm tải Git server |
| `spec.url` | HTTPS hoặc SSH URL của repo |
| `spec.secretRef.name` | Secret chứa `username`/`password` (HTTPS) hoặc `identity`/`identity.pub`/`known_hosts` (SSH). Với public repo thì không cần |
| `spec.ref.branch` / `tag` / `semver` / `commit` | Theo dõi nhánh (môi trường), ghim tag (production), range semver, hoặc đúng 1 commit (debug) |
| `spec.timeout` | Timeout cho thao tác Git (mặc định 60s) |

Kiểm tra trạng thái:

```bash
flux get sources git
kubectl -n flux-system describe gitrepository fleet-infra
```

Trường `status.artifact.revision` cho biết commit hiện tại mà Flux đã tải. Nếu `READY=False`, đọc `MESSAGE` để biết lỗi (sai credential, sai URL...).

**Tạo secret cho private repo (HTTPS):**

```bash
kubectl -n flux-system create secret generic fleet-infra-auth \
  --from-literal=username=git \
  --from-literal=password=$GITHUB_TOKEN
```

**Tạo secret cho SSH:**

```bash
flux create secret git fleet-infra-auth \
  --namespace=flux-system \
  --url=ssh://git@gitea.example.com/platform/fleet-infra.git \
  --private-key-file=./flux-key
```

Lệnh `flux create secret git` tự động sinh `known_hosts` bằng cách scan host — tiện và an toàn hơn làm tay.

---

## 5. Kustomization CRD

`Kustomization` (do kustomize-controller quản lý) định nghĩa *cái gì* trong Git repo sẽ được áp dụng vào cluster và *áp dụng như thế nào*: đường dẫn, prune, validation, health check, thứ tự phụ thuộc.

Ví dụ đầy đủ:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  # Chu kỳ reconciliation: so sánh Git vs cluster mỗi 10 phút
  interval: 10m0s
  # Trỏ tới GitRepository nguồn
  sourceRef:
    kind: GitRepository
    name: fleet-infra
  # Đường dẫn trong repo chứa kustomization.yaml (hoặc plain YAML)
  path: ./apps/production
  # Xóa tài nguyên trong cluster nhưng không còn trong Git (chống rác)
  prune: true
  # Chờ tài nguyên "khỏe" mới coi là sync thành công
  wait: true
  # Timeout cho mỗi lần reconciliation
  timeout: 5m0s
  # Kiểm tra manifest trước khi apply: client (chỉ parse), server (dry-run lên API server)
  validation: server
  # Force apply khi có immutable field thay đổi (vd: selector của Deployment) → xóa rồi tạo lại
  force: false
  # Thứ tự: chỉ chạy sau khi Kustomization 'infrastructure' xong
  dependsOn:
    - name: infrastructure
  # Health check cho các workload không có sẵn check mặc định
  healthChecks:
    - apiVersion: apps/v1
      kind: Deployment
      name: api
      namespace: production
  # Biến môi trường thay thế trong manifest (${VAR} trong YAML)
  postBuild:
    substitute:
      CLUSTER_ENV: production
      DOMAIN: example.com
    substituteFrom:
      - kind: ConfigMap
        name: cluster-vars
```

Các trường quan trọng:

| Trường | Ý nghĩa |
|---|---|
| `spec.interval` | Chu kỳ đối chiếu. Dev: 1m; production: 5m–10m |
| `spec.path` | Đường dẫn tương đối trong Git repo |
| `spec.prune` | `true` = xóa tài nguyên không còn trong Git. **Luôn bật** để tránh drift |
| `spec.wait` | Chờ health check pass. Nên bật cho app quan trọng |
| `spec.validation` | `none` / `client` / `server`. Production nên dùng `server` (dry-run) |
| `spec.force` | Chỉ bật khi cần thay đổi immutable field; bình thường để `false` |
| `spec.dependsOn` | Khai báo phụ thuộc giữa các Kustomization |
| `spec.healthChecks` | Thêm health check cho CRD/custom resource Flux chưa biết cách check |
| `spec.suspend` | Tạm dừng reconciliation (hữu ích khi debug) |
| `spec.postBuild.substitute` | Thay biến `${VAR}` trong manifest — dùng chung manifest cho nhiều môi trường |

Xem trạng thái:

```bash
flux get kustomizations
flux logs --kind=Kustomization --name=apps --follow
```

---

## 6. HelmRelease CRD

`HelmRelease` (do helm-controller quản lý) mô tả một Helm release như một tài nguyên Kubernetes: chart nào, version nào, values ra sao, chiến lược upgrade thế nào.

### 6.1. HelmRepository — khai báo nguồn chart

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 10m0s
  url: https://charts.bitnami.com/bitnami
  # type: oci  # nếu là OCI registry: url dạng oci://...
```

Với private Helm repo, thêm `secretRef` như GitRepository.

### 6.2. HelmRelease — deploy chart

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: redis
  namespace: production
spec:
  interval: 10m0s
  # Chart lấy từ HelmRepository nào
  chart:
    spec:
      chart: redis
      version: "18.x"          # semver range: tự lên patch/minor trong 18.x
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
      interval: 10m0s
  # Ghim đúng version (thay cho range ở trên nếu muốn bất biến)
  # version: 18.4.0
  # Số bản release giữ lại trong history (để rollback, mặc định 10)
  values:
    architecture: standalone
    auth:
      enabled: true
      password: "change-me-in-production"  # thực tế nên dùng SOPS, xem mục 10
    master:
      persistence:
        enabled: true
        size: 8Gi
  # Values từ ConfigMap/Secret có sẵn (tiện khi values lớn)
  valuesFrom:
    - kind: ConfigMap
      name: redis-values
      valuesKey: values.yaml
      optional: false
    - kind: Secret
      name: redis-secret-values
      valuesKey: values.yaml
      optional: true
  # Chiến lược cài đặt
  install:
    createNamespace: true
    remediation:
      retries: 3              # thử lại 3 lần nếu install fail
  # Chiến lược upgrade
  upgrade:
    remediation:
      retries: 3
      remediateLastFailure: true
    cleanupOnFail: true
  # Test sau khi install/upgrade (helm test)
  test:
    enable: true
  # Rollback tự động về release trước nếu upgrade fail
  rollback:
    timeout: 5m0s
    cleanupOnFail: true
  # Chờ resources sẵn sàng
  timeout: 5m0s
```

Các trường quan trọng:

| Trường | Ý nghĩa |
|---|---|
| `chart.spec.version` | Semver range (`18.x`) cho auto-minor, hoặc version cố định cho production nghiêm ngặt |
| `values` / `valuesFrom` | Ghi đè values; `valuesFrom` đọc từ ConfigMap/Secret — Secret phù hợp cho giá trị nhạy cảm (kết hợp SOPS) |
| `install/upgrade.remediation` | Tự thử lại khi fail — giảm can thiệp tay lúc mạng chập chờn |
| `rollback` | Tự rollback khi upgrade fail — an toàn cho production |
| `dependsOn` | Phụ thuộc HelmRelease/Kustomization khác |
| `suspend` | Tạm dừng release này |

Kiểm tra:

```bash
flux get helmreleases -A
# NAMESPACE   NAME   REVISION  SUSPENDED  READY  MESSAGE
# production  redis  18.4.0   False      True   Release reconciliation succeeded
```

---

## 7. Image Automation

Bộ ba CRD giúp **tự động cập nhật image tag trong Git** khi có image mới — khép kín vòng lặp CI → CD mà không cần pipeline push vào cluster.

### 7.1. ImageRepository — quét registry

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImageRepository
metadata:
  name: api
  namespace: flux-system
spec:
  # Image cần theo dõi
  image: ghcr.io/example-org/api
  # Chu kỳ quét tag mới
  interval: 5m0s
  # Secret pull image nếu là private registry
  secretRef:
    name: ghcr-credentials
  # Loại trừ tag không muốn (vd: latest, sha-...)
  exclusionList:
    - "^.*\\.sig$"     # bỏ qua signature của cosign
```

Tạo secret cho private registry:

```bash
kubectl -n flux-system create secret docker-registry ghcr-credentials \
  --docker-server=ghcr.io \
  --docker-username=$GITHUB_USER \
  --docker-password=$GITHUB_TOKEN
```

### 7.2. ImagePolicy — chọn tag theo policy

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta2
kind: ImagePolicy
metadata:
  name: api
  namespace: flux-system
spec:
  imageRepositoryRef:
    name: api
  policy:
    # semver: chọn version mới nhất trong range
    semver:
      range: ">=1.0.0 <2.0.0"
    # alphabetical.order: asc/desc — cho tag dạng 20240101-abcdef
    # numerical — sắp xếp theo số
  # Chỉ xét tag khớp regex, extract phần version
  filterTags:
    pattern: "^v(?P<version>.*)$"
    extract: "$version"
```

Các policy phổ biến:
- `semver.range: 1.0.x` — chỉ nhận patch mới (an toàn nhất cho production).
- `semver.range: ">=1.0.0 <2.0.0"` — nhận cả minor mới.
- Kết hợp `filterTags.pattern` để bỏ qua tag `latest`, `dev`, `rc`...

Xem tag đang được chọn:

```bash
flux get images policy -A
# NAMESPACE    NAME  LATEST IMAGE
# flux-system  api   ghcr.io/example-org/api:v1.4.2
```

### 7.3. ImageUpdateAutomation — tự động commit vào Git

```yaml
apiVersion: image.toolkit.fluxcd.io/v1beta1
kind: ImageUpdateAutomation
metadata:
  name: fleet-images
  namespace: flux-system
spec:
  interval: 10m0s
  # Git repo sẽ được checkout, sửa file, commit, push
  sourceRef:
    kind: GitRepository
    name: fleet-infra
  # Git config cho commit
  git:
    checkout:
      ref:
        branch: main
    commit:
      author:
        name: fluxcdbot
        email: fluxcdbot@example.com
      messageTemplate: |
        chore: update image {{range .Updated.Images}}{{println .}}{{end}}
    push:
      branch: main
  # Chiến lược cập nhật file YAML trong repo
  update:
    path: ./apps/production
    strategy: Setters   # dùng kustomize setters (vd: //$imagepolicy...)
```

Để automation biết *chỗ nào* trong YAML cần thay tag, đánh dấu bằng comment đặc biệt (kustomize setter):

```yaml
# apps/production/api/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  template:
    spec:
      containers:
        - name: api
          image: ghcr.io/example-org/api:v1.4.1 # {"$imagepolicy": "flux-system:api"}
```

Comment `# {"$imagepolicy": "flux-system:api"}` nói với automation: "dòng image này do ImagePolicy `api` trong namespace `flux-system` quản lý". Khi policy chọn được tag mới (v1.4.2), automation sẽ sửa dòng này, commit và push. Sau đó luồng GitOps bình thường (source → kustomize) sẽ deploy image mới vào cluster.

Luồng hoàn chỉnh:

```
CI build & push image v1.4.2
        ▼
ImageRepository quét thấy tag mới → ImagePolicy chọn v1.4.2 (khớp semver range)
        ▼
ImageUpdateAutomation sửa deployment.yaml, commit, push lên Git
        ▼
GitRepository pull → Kustomization apply → Deployment rollout image mới
        ▼
Notification gửi Slack: "api đã lên v1.4.2"
```

> **Mẹo an toàn:** production nên dùng policy `1.0.x` (chỉ patch tự động); minor/major tạo PR thủ công để review. Có thể cấu hình automation push lên nhánh riêng để tạo PR thay vì commit thẳng vào main.

---

## 8. Notifications

notification-controller gồm 3 CRD:
- **Provider**: đích gửi thông báo (Slack, Discord, Telegram, PagerDuty, webhook generic...).
- **Alert**: quy tắc "khi tài nguyên X có sự kiện Y thì gửi tới Provider Z".
- **Receiver**: nhận webhook từ Git provider (GitHub/GitLab...) để trigger reconciliation ngay lập tức thay vì chờ interval.

### 8.1. Provider — ví dụ Slack, Discord, Telegram

Tạo secret chứa webhook URL/token trước (dùng `kubectl create secret generic`, key `address` chứa URL; Telegram cần thêm key `token` và `channel`):

```yaml
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Provider
metadata:
  name: slack
  namespace: flux-system
spec:
  type: slack
  channel: "#flux-alerts"
  address: https://hooks.slack.com/services/T000/B000/XXXX
  # Hoặc tham chiếu secret (khuyên dùng để không lộ URL trong Git):
  # secretRef:
  #   name: slack-webhook
---
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Provider
metadata:
  name: discord
  namespace: flux-system
spec:
  type: discord
  address: https://discord.com/api/webhooks/xxx/yyy
  # secretRef:
  #   name: discord-webhook
---
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Provider
metadata:
  name: telegram
  namespace: flux-system
spec:
  type: telegram
  secretRef:
    name: telegram-bot
```

Các `type` được hỗ trợ: `slack`, `discord`, `telegram`, `msteams`, `pagerduty`, `opsgenie`, `lark`, `webex`, `alertmanager`, `grafana`, `github`, `gitlab`, `webhook` (generic), `smtp`...

### 8.2. Alert — gửi thông báo khi sync lỗi (hoặc thành công)

```yaml
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Alert
metadata:
  name: apps-errors
  namespace: flux-system
spec:
  # Chỉ gửi khi có sự kiện với severity này trở lên
  eventSeverity: error
  # Nguồn sự kiện: mọi Kustomization và HelmRelease trong mọi namespace
  eventSources:
    - kind: Kustomization
      name: "*"             # wildcard: tất cả
    - kind: HelmRelease
      name: "*"
    - kind: GitRepository
      name: "*"
  providerRef:
    name: slack
  # Tóm tắt: gom nhiều sự kiện trong 5 phút thành 1 tin nhắn
  summary: "Flux sync errors"
---
# Alert riêng cho deploy thành công của app quan trọng
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Alert
metadata:
  name: api-deployments
  namespace: flux-system
spec:
  eventSeverity: info
  eventSources:
    - kind: HelmRelease
      name: api
      namespace: production
    - kind: Kustomization
      name: apps
  providerRef:
    name: telegram
  summary: "API deployment status"
```

`eventSeverity` có các mức: `info`, `error`. Dùng `error` để chỉ nhận cảnh báo lỗi (tránh spam), `info` khi muốn theo dõi deploy.

Loại trừ sự kiện ồn ào:

```yaml
spec:
  eventSeverity: info
  eventSources:
    - kind: Kustomization
      name: apps
  exclusionList:
    - ".*no changes.*"
  providerRef:
    name: discord
```

### 8.3. Receiver — webhook từ GitHub để sync ngay

Thay vì chờ `interval`, GitHub có thể báo cho Flux ngay khi có push:

```yaml
apiVersion: notification.toolkit.fluxcd.io/v1beta3
kind: Receiver
metadata:
  name: github-webhook
  namespace: flux-system
spec:
  type: github
  events:
    - "push"
  secretRef:
    name: webhook-token      # token xác thực webhook (HMAC)
  resources:
    - kind: GitRepository
      name: fleet-infra
```

```bash
kubectl -n flux-system create secret generic webhook-token \
  --from-literal=token=$(openssl rand -hex 32)

# Lấy URL receiver để cấu hình trên GitHub (Settings → Webhooks)
kubectl -n flux-system get receiver github-webhook -o jsonpath='{.status.url}'
```

---

## 9. Multi-tenancy

Flux được thiết kế để nhiều team dùng chung một cluster mà vẫn cô lập lẫn nhau. Cơ chế gồm 3 lớp:

### 9.1. Mô hình tenant trong Flux

Nguyên tắc:
1. **Platform team** giữ repo gốc, định nghĩa tenant nào được deploy cái gì.
2. Mỗi tenant có **ServiceAccount riêng** + RBAC giới hạn trong namespace của mình.
3. Kustomization của tenant dùng `serviceAccountName` để **impersonate** — Flux chỉ được phép làm những gì ServiceAccount đó được phép (manifest cố tạo tài nguyên ngoài namespace sẽ bị API server từ chối).

### 9.2. Ví dụ: tạo tenant team-a

**Bước 1: Namespace + RBAC cho tenant**

```yaml
# tenants/team-a/ns.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: team-a
  labels:
    tenant: team-a
---
# tenants/team-a/rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: team-a-deployer
  namespace: flux-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: team-a-admin
  namespace: team-a
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]   # toàn quyền TRONG namespace team-a
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: team-a-binding
  namespace: team-a
subjects:
  - kind: ServiceAccount
    name: team-a-deployer
    namespace: flux-system
roleRef:
  kind: Role
  name: team-a-admin
  apiGroup: rbac.authorization.k8s.io
```

**Bước 2: GitRepository + Kustomization riêng của tenant** (xem mẫu đầy đủ ở mục 4–5; điểm mấu chốt là `serviceAccountName` bên dưới)

```yaml
# tenants/team-a/sync.yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: team-a-apps
  namespace: flux-system
spec:
  interval: 2m0s
  url: https://github.com/example-org/team-a-apps   # repo RIÊNG của team A
  secretRef:
    name: team-a-repo-auth
  ref:
    branch: main
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: team-a-apps
  namespace: flux-system
spec:
  interval: 5m0s
  sourceRef:
    kind: GitRepository
    name: team-a-apps
  path: ./deploy
  prune: true
  # QUAN TRỌNG: impersonate ServiceAccount của tenant
  serviceAccountName: team-a-deployer
  # Kết hợp với RBAC ở bước 1: ServiceAccount chỉ có quyền trong namespace team-a,
  # nên manifest của team A không thể tạo tài nguyên ở namespace khác.
```

### 9.3. Bảng phân quyền gợi ý

| Vai trò | Được làm gì | Không được làm gì |
|---|---|---|
| Platform team | Quản lý `flux-system`, tạo tenant, định nghĩa policy chung | Không sửa app của tenant (trừ khi khẩn cấp) |
| Tenant (team A) | Deploy trong namespace `team-a`, quản lý repo riêng, xem log của mình | Tạo tài nguyên ngoài namespace, xem secret của tenant khác, sửa cấu hình Flux |
| Read-only (audit) | `flux get`, xem status | Mọi thao tác ghi |

> Kết hợp với policy engine (Kyverno/OPA Gatekeeper) để enforce thêm: ví dụ "mọi Deployment phải có resource limit", "không được dùng image latest".

---

## 10. Quản lý Secrets với SOPS

Vấn đề: GitOps yêu cầu mọi thứ trong Git, nhưng secret (password, token, key) không thể để plaintext trong Git. Giải pháp chuẩn của Flux là **SOPS** (Secrets OPerationS) của Mozilla: mã hóa file YAML bằng **age** (hiện đại, khuyên dùng) hoặc **GPG**, commit file đã mã hóa lên Git, Flux tự giải mã trong cluster bằng private key.

### 10.1. Cài đặt công cụ

```bash
# SOPS
# macOS
brew install sops
# Linux: tải binary từ GitHub releases mozilla/sops

# age (khuyên dùng thay GPG vì đơn giản)
# macOS
brew install age
# Linux
sudo apt install age   # Debian/Ubuntu
```

### 10.2. Tạo age key pair

```bash
# Tạo key
age-keygen -o age.agekey
# Output:
# # created: 2026-10-10T...
# # public key: age1ql3z7hj432v2jl2z8alunwwun8hm4s4h6d7qqd5p3v9x8f9s8abcd
# AGE-SECRET-KEY-1QL3Z7HJ432V2JL2Z8ALUNWWUN8HM4S4H6D7QQD5P3V9X8F9S8ABCD

# Public key dùng để mã hóa (cho vào .sops.yaml)
# Private key (AGE-SECRET-KEY-...) đưa vào cluster
```

Đưa private key vào cluster để Flux giải mã:

```bash
cat age.agekey |
kubectl -n flux-system create secret generic sops-age \
  --from-file=age.agekey=/dev/stdin
```

> **Bảo mật:** file `age.agekey` chứa private key — KHÔNG commit lên Git, lưu trữ an toàn (password manager, vault). Mất private key = không giải mã được secret; lộ private key = ai cũng đọc được secret.

### 10.3. Cấu hình `.sops.yaml` trong repo

File này đặt ở root repo GitOps, quy định file nào mã hóa bằng key nào:

```yaml
# .sops.yaml
creation_rules:
  # Mã hóa mọi file secret trong thư mục clusters/
  - path_regex: clusters/.*/.*\.sops\.yaml$
    age: age1ql3z7hj432v2jl2z8alunwwun8hm4s4h6d7qqd5p3v9x8f9s8abcd
  # Mã hóa secret của app
  - path_regex: apps/.*/.*\.sops\.yaml$
    age: age1ql3z7hj432v2jl2z8alunwwun8hm4s4h6d7qqd5p3v9x8f9s8abcd
```

Quy ước: file secret mã hóa đặt tên `*.sops.yaml` để dễ nhận biết.

### 10.4. Mã hóa và sử dụng secret

```bash
# Tạo file secret plaintext TẠM THỜI
cat > apps/production/api/db-secret.sops.yaml <<'EOF'
apiVersion: v1
kind: Secret
metadata:
  name: api-db
  namespace: production
stringData:
  username: api_user
  password: "siêu-bí-mật-đổi-đi-nhé"
  connection: "postgres://api_user:siêu-bí-mật-đổi-đi-nhé@postgres:5432/appdb"
EOF

# Mã hóa tại chỗ (file gốc bị thay bằng bản mã hóa)
sops --encrypt --in-place apps/production/api/db-secret.sops.yaml

# Kiểm tra: file giờ chỉ toàn ciphertext
head -20 apps/production/api/db-secret.sops.yaml

# Xóa bản plaintext khỏi lịch sử nếu lỡ commit trước đó (dùng git filter-repo)
```

File sau khi mã hóa trông như:

```yaml
apiVersion: v1
kind: Secret
metadata:
    name: api-db
    namespace: production
stringData:
    username: ENC[AES256_GCM,data:...,iv:...,tag:...,type:str]
    password: ENC[AES256_GCM,data:...,iv:...,tag:...,type:str]
    connection: ENC[AES256_GCM,data:...,iv:...,tag:...,type:str]
sops:
    age:
        - recipient: age1ql3z7hj432v2jl2z8alunwwun8hm4s4h6d7qqd5p3v9x8f9s8abcd
          enc: |
            -----BEGIN AGE ENCRYPTED FILE-----
            ...
    lastmodified: "2026-10-10T10:00:00Z"
    mac: ENC[AES256_GCM,data:...,iv:...,tag:...,type:str]
    version: 3.9.0
```

Để Flux tự giải mã, thêm `decryption` vào Kustomization:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 10m0s
  sourceRef:
    kind: GitRepository
    name: fleet-infra
  path: ./apps/production
  prune: true
  decryption:
    provider: sops
    # secretRef mặc định là 'sops-age' trong cùng namespace — khai báo rõ để chắc chắn
    secretRef:
      name: sops-age
```

kustomize-controller sẽ dùng private key trong secret `sops-age` để giải mã mọi file SOPS trước khi apply. Secret trong cluster (etcd) vẫn được lưu bình thường — SOPS chỉ bảo vệ secret *trong Git*.

### 10.5. Dùng với GPG (phương án thay thế)

Nếu đã có hạ tầng GPG, bạn có thể dùng GPG thay cho age:

```bash
gpg --full-generate-key
gpg --export-secret-keys --armor <FINGERPRINT> |
kubectl -n flux-system create secret generic sops-gpg \
  --from-file=sops.asc=/dev/stdin
```

`.sops.yaml` dùng `pgp: "<FINGERPRINT>"` thay vì `age:`, và Kustomization dùng `secretRef.name: sops-gpg`. age vẫn là lựa chọn khuyên dùng vì đơn giản hơn.

### 10.6. Quy trình làm việc với secret mã hóa

```bash
# Sửa secret đã mã hóa (sops tự giải mã vào editor, lưu là mã hóa lại)
sops apps/production/api/db-secret.sops.yaml

# Xem nội dung giải mã (không lưu)
sops --decrypt apps/production/api/db-secret.sops.yaml

# Xoay key (re-encrypt với key mới)
sops --encrypt --in-place --age <public-key-mới> apps/production/api/db-secret.sops.yaml
```

> **Lưu ý:** SOPS mã hóa *giá trị*, không mã hóa *cấu trúc* — tên secret, tên key vẫn đọc được trong Git. Nếu cần giấu cả tên, đặt tên file/key một cách trung tính.

---

## 11. Dự án thực tế: bootstrap Flux + deploy app

Mục tiêu: từ cluster trống, bootstrap Flux, deploy một app demo gồm:
- **Kustomize**: app `hello` (Deployment + Service đơn giản).
- **Helm**: Redis từ Bitnami chart làm dependency.

### 11.1. Chuẩn bị

```bash
# 1. Cluster (ví dụ kind)
kind create cluster --name flux-demo

# 2. Cài Flux CLI (xem mục 3.1)

# 3. Kiểm tra
flux check --pre

# 4. Export GitHub credential
export GITHUB_TOKEN=<token-có-quyền-repo>
export GITHUB_USER=<username>
```

### 11.2. Bootstrap

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fleet-demo \
  --branch=main \
  --path=clusters/demo \
  --personal
```

Clone repo về máy để làm việc:

```bash
git clone https://github.com/$GITHUB_USER/fleet-demo.git
cd fleet-demo
```

Cấu trúc repo lúc này:

```
fleet-demo/
└── clusters/
    └── demo/
        └── flux-system/   # gotk-components.yaml, gotk-sync.yaml, kustomization.yaml
```

### 11.3. Tạo cấu trúc thư mục

```bash
mkdir -p apps/demo/base infrastructure/demo
```

Cấu trúc mục tiêu:

```
fleet-demo/
├── .sops.yaml                    # (tùy chọn, nếu dùng SOPS)
├── clusters/
│   └── demo/
│       ├── flux-system/          # do bootstrap sinh
│       ├── apps.yaml             # Kustomization trỏ tới apps/demo
│       └── infrastructure.yaml   # Kustomization trỏ tới infrastructure/demo
├── apps/
│   └── demo/
│       ├── namespace.yaml
│       ├── hello-deployment.yaml
│       ├── hello-service.yaml
│       └── kustomization.yaml
└── infrastructure/
    └── demo/
        ├── namespace.yaml
        ├── redis-helmrelease.yaml
        └── kustomization.yaml
```

### 11.4. App Kustomize: hello

`apps/demo/namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo
```

`apps/demo/hello-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello
  namespace: demo
  labels:
    app: hello
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: hello
          # Image demo trả về "Hello" — thay bằng image thật của bạn
          image: hashicorp/http-echo:0.2.3 # {"$imagepolicy": "flux-system:hello"}
          args:
            - "-text=Hello from Flux!"
            - "-listen=:8080"
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /
              port: 8080
            initialDelaySeconds: 3
            periodSeconds: 5
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
```

`apps/demo/hello-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello
  namespace: demo
spec:
  selector:
    app: hello
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

`apps/demo/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: demo
resources:
  - namespace.yaml
  - hello-deployment.yaml
  - hello-service.yaml
```

### 11.5. Infrastructure Helm: Redis

`infrastructure/demo/namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: demo-infra
```

`infrastructure/demo/redis-helmrelease.yaml`:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  interval: 30m0s
  url: https://charts.bitnami.com/bitnami
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: redis
  namespace: demo-infra
spec:
  interval: 30m0s
  chart:
    spec:
      chart: redis
      version: "18.x"
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
      interval: 30m0s
  values:
    architecture: standalone
    auth:
      enabled: false   # demo only — production bật auth + SOPS
    master:
      persistence:
        enabled: false # demo trên kind — production bật persistence
  install:
    createNamespace: true
```

`infrastructure/demo/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - redis-helmrelease.yaml
```

### 11.6. Kustomization "gốc" trong clusters/demo

`clusters/demo/infrastructure.yaml` — deploy hạ tầng trước:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: infrastructure
  namespace: flux-system
spec:
  interval: 10m0s
  sourceRef:
    kind: GitRepository
    name: flux-system
  path: ./infrastructure/demo
  prune: true
  wait: true
  timeout: 5m0s
```

`clusters/demo/apps.yaml` — app chạy sau hạ tầng:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 10m0s
  sourceRef:
    kind: GitRepository
    name: flux-system   # GitRepository do flux bootstrap tạo sẵn
  path: ./apps/demo
  prune: true
  wait: true
  timeout: 5m0s
  dependsOn:
    - name: infrastructure   # app chỉ deploy sau khi Redis xong
```

### 11.7. Push và kiểm tra

```bash
git add -A
git commit -m "feat: add demo app (kustomize) + redis (helm)"
git push origin main
```

Theo dõi quá trình sync:

```bash
flux get helmreleases -A
flux logs --level=error --follow   # xem log nếu có vấn đề
```

Kiểm tra app chạy:

```bash
kubectl -n demo get pods
kubectl -n demo port-forward svc/hello 8080:80 &
curl localhost:8080   # Hello from Flux!
```

### 11.9. Dọn dẹp (tùy chọn)

```bash
# Gỡ app khỏi Git → Flux tự prune
# Hoặc gỡ Flux khỏi cluster (giữ lại resource đã deploy):
flux uninstall --keep-namespace
```

---

## 12. Migration từ Flux v1 sang v2

Flux v1 (dự án `fluxcd/flux`, đã ngừng phát triển) và Flux v2 (bộ toolkit hiện tại) khác nhau căn bản. Nếu bạn còn chạy v1, đây là lộ trình migration.

### 12.1. Khác biệt chính

| | Flux v1 | Flux v2 |
|---|---|---|
| Kiến trúc | 1 binary monolith (`fluxd`) + memcached | Nhiều controller chuyên biệt |
| Cấu hình | Annotation trên workload (`fluxcd.io/tag.*`) | CRD khai báo (GitRepository, Kustomization, HelmRelease...) |
| Helm | Tích hợp qua `flux-helm-operator` riêng | helm-controller native |
| Image automation | Tự động update image theo annotation | ImageRepository/ImagePolicy/ImageUpdateAutomation |
| CLI | `fluxctl` | `flux` (viết lại hoàn toàn) |

### 12.2. Các bước migration

**Bước 1: Kiểm kê v1 hiện tại** — liệt kê workload có annotation `fluxcd.io/automated=true`, ghi lại image policy, repo, nhánh đang dùng.

**Bước 2: Cài Flux v2 song song (không gỡ v1 ngay)**

```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fleet-infra-v2 \
  --branch=main \
  --path=clusters/production \
  --personal
```

Dùng repo/nhánh riêng để v1 và v2 không giẫm chân nhau.

**Bước 3: Chuyển đổi cấu hình**

- Annotation `fluxcd.io/tag.<container>` / `fluxcd.io/automated: "true"` → `ImagePolicy` + `ImageUpdateAutomation` (mục 7).
- HelmRelease của flux-helm-operator (`helm.fluxcd.io/v1`) → `helm.toolkit.fluxcd.io/v2` (mục 6). **apiVersion và schema khác nhau — phải viết lại và test trên staging.**
- `.flux.yaml` (cấu hình repo v1) → `GitRepository` + `Kustomization`.

**Bước 4: Chuyển từng workload theo đợt** — tắt automation v1 cho namespace đó trước (tránh double-write: v1 và v2 cùng commit gây conflict), rồi để v2 tiếp quản; đi từ namespace ít quan trọng đến quan trọng nhất.

**Bước 5: Gỡ v1**

Khi mọi workload đã chạy ổn định trên v2 ít nhất một chu kỳ release:

```bash
# Gỡ flux v1 (tên namespace/helm release tùy cách bạn cài v1)
helm uninstall flux --namespace flux
# hoặc
kubectl delete namespace flux
```

### 12.3. Bẫy thường gặp khi migration

- **Double-write**: quên tắt v1 automation → v1 và v2 cùng commit vào Git gây conflict. Luôn tắt v1 trước khi bật v2 cho cùng workload.
- **HelmRelease v1 không tương thích v2**: phải viết lại hoàn toàn, test trên staging trước.
- **Image tag format**: v1 dùng regex trong annotation, v2 dùng `filterTags.pattern` + `extract` — kiểm tra bằng `flux get images policy`.

---

## 13. Best practices

### 13.1. Cấu trúc repo

Mô hình được khuyên dùng (monorepo, nhiều cluster):

```
fleet-infra/
├── clusters/
│   ├── staging/
│   │   ├── flux-system/        # bootstrap sinh
│   │   ├── apps.yaml           # Kustomization: path ./apps/staging
│   │   └── infrastructure.yaml # Kustomization: path ./infrastructure/staging
│   └── production/
│       ├── flux-system/
│       ├── apps.yaml
│       └── infrastructure.yaml
├── apps/
│   ├── base/                   # manifest dùng chung
│   │   └── api/
│   ├── staging/                # overlay staging (patch, values riêng)
│   │   └── api/
│   └── production/
│       └── api/
├── infrastructure/
│   ├── base/                   # controller chung: ingress-nginx, cert-manager...
│   ├── staging/
│   └── production/
└── tenants/                    # (nếu multi-tenant)
    └── team-a/
```

Nguyên tắc:
- **Tách `apps` và `infrastructure`**: hạ tầng (ingress, cert-manager, monitoring) deploy trước, app deploy sau qua `dependsOn`.
- **Base + overlay**: dùng chung manifest ở `base`, khác biệt môi trường nằm ở overlay — tránh copy-paste.
- **Một repo cho fleet, repo riêng cho từng tenant** (nếu multi-tenant, xem mục 9).

### 13.2. Interval hợp lý

| Môi trường | GitRepository | Kustomization/HelmRelease |
|---|---|---|
| Development | 30s–1m | 1m–2m |
| Staging | 1m–2m | 5m |
| Production | 5m–10m | 10m–30m |
| HelmRepository (public) | 30m–1h | — |

Interval quá ngắn → Git server/registry bị spam request, nguy cơ rate-limit. Kết hợp **Receiver webhook** (mục 8.3) để vừa phản ứng nhanh khi push, vừa giữ interval dài.

### 13.3. dependsOn — thứ tự deploy

```yaml
# infrastructure.yaml (không dependsOn — chạy đầu tiên)
# apps.yaml
spec:
  dependsOn:
    - name: infrastructure
# monitoring.yaml (cần app có mặt để scrape metric)
spec:
  dependsOn:
    - name: apps
```

Lưu ý:
- `dependsOn` chỉ đảm bảo *thứ tự bắt đầu*, muốn đảm bảo hạ tầng *sẵn sàng* thì bật thêm `wait: true` ở Kustomization hạ tầng.
- Tránh dependsOn quá sâu (>3 tầng) — khó debug khi có sự cố dây chuyền.
- **Không tạo vòng tròn** (A dependsOn B, B dependsOn A) — Flux sẽ báo dependency cycle và cả hai đều không chạy (xem troubleshooting).

### 13.4. Các best practices khác

- **Luôn bật `prune: true`** (trừ Kustomization quản lý tài nguyên dùng chung với hệ thống khác).
- **Dùng `validation: server`** ở production để dry-run trước khi apply.
- **Ghim version ở production**: `ref.tag`/semver chặt cho GitRepository; version cố định cho Helm chart.
- **Bật `wait: true`** cho workload quan trọng; thêm `healthChecks` cho CRD custom.
- **Mọi secret trong Git đều qua SOPS** (mục 10) — không bao giờ commit plaintext secret, kể cả môi trường dev.
- **Tách quyền**: dùng `serviceAccountName` cho tenant (mục 9); controller chạy với quyền tối thiểu.
- **Alert bắt buộc**: ít nhất một Alert `eventSeverity: error` tới kênh chat của team (mục 8).
- **Test thay đổi trên staging trước**; dùng `flux diff kustomization` để xem trước.
- **Giữ Flux updated**: `flux check` định kỳ; upgrade staging trước khi upgrade production.

---

## 14. Troubleshooting

### 14.1. Cách tiếp cận chung

```bash
flux get all -A                                    # cái nào không READY?
kubectl -n flux-system describe kustomization <name>   # message lỗi chi tiết
flux logs --kind=Kustomization --name=<name> --follow
flux logs --level=error --follow                   # mọi lỗi từ mọi controller
```

### 14.2. Lỗi thường gặp

#### Reconciliation failed / Apply failed

**Triệu chứng:** `READY=False`, message `Reconciliation failed` / `apply failed`.

- YAML sai cú pháp → validate local: `kubeconform` hoặc `kubectl apply --dry-run=server`.
- Thiếu CRD (deploy CR trước khi operator chạy) → thêm `dependsOn` vào Kustomization chứa operator.
- Immutable field (vd: `spec.selector` của Deployment) → `kubectl delete` resource cũ để Flux tạo lại, hoặc `force: true` (cẩn thận: delete + recreate).
- Hết quota/RBAC → xem event của resource lỗi.

#### Suspended — Kustomization/HelmRelease bị tạm dừng

**Triệu chứng:** cột `SUSPENDED=True`, không có reconciliation nào chạy.

**Nguyên nhân:** ai đó (hoặc automation) đã chạy `flux suspend`. Kiểm tra:

```bash
flux get kustomizations  # xem cột SUSPENDED
```

**Xử lý:**

```bash
flux resume kustomization <name>
flux resume helmrelease <name> -n <namespace>
flux resume source git <name>
```

> Nếu bị suspend mà không ai làm, kiểm tra xem có cronjob/script nào gọi `flux suspend` không.

#### Dependency cycle

**Triệu chứng:** message `dependency cycle detected: a -> b -> a`, cả hai Kustomization đều không chạy.

**Xử lý:** vẽ lại đồ thị `dependsOn`, phá vòng tròn — thường bằng cách tách tài nguyên chung ra một Kustomization "base" mà cả hai cùng phụ thuộc vào.

```bash
# Tìm dependsOn của mọi Kustomization
kubectl -n flux-system get kustomizations -o json | \
  jq -r '.items[] | "\(.metadata.name) -> \(.spec.dependsOn // [] | map(.name) | join(","))"'
```

#### GitRepository không pull được (READY=False)

- **Sai credential / token hết hạn** (`authentication failed`) → xoay token, update secret, `flux reconcile source git <name>`.
- **Sai URL/nhánh** → kiểm tra `spec.url`, `spec.ref`.
- **Repo quá lớn** → tăng `spec.timeout`, dùng `spec.ignore` loại file lớn không liên quan.

#### HelmRelease fail: chart không tìm thấy / version không tồn tại

```bash
flux get sources helm   # kiểm tra HelmRepository READY chưa
flux get helmreleases -A
```

- HelmRepository `READY=False` → kiểm tra URL, credential.
- Sai version range → kiểm tra chart version thực có trên repo: `helm search repo <chart> --versions` (cần add repo local để check).
- Values sai schema → `helm template` local với cùng values để debug.

#### Image automation không update tag

1. `flux get images repository -A` — ImageRepository `READY=True` và liệt kê được tag? Nếu không: sai `spec.image`, sai secret registry.
2. `flux get images policy -A` — cột `LATEST IMAGE` đúng tag mong đợi? Nếu không: xem lại `semver.range` và `filterTags.pattern`.
3. ImageUpdateAutomation push được không? `flux logs --kind=ImageUpdateAutomation --name=<name>`. Lỗi thường gặp: token Git hết hạn, nhánh bị protect, `path` sai.

#### SOPS decryption failed

**Triệu chứng:** `decryption failed: ... cannot decrypt`.

- Secret `sops-age`/`sops-gpg` không tồn tại hoặc sai namespace → tạo lại (mục 10.2).
- Public key trong `.sops.yaml` không khớp private key trong cluster → kiểm tra fingerprint.
- File mã hóa bằng key cũ sau khi xoay key → re-encrypt lại toàn bộ file `.sops.yaml` bằng key mới.
- `decryption.secretRef` sai tên trong Kustomization.

#### Notification không gửi

- Provider `READY=False` → kiểm tra `address`/`secretRef`; test webhook bằng `curl` thủ công.
- Alert sai `eventSources` (sai kind/name/namespace) → không match sự kiện nào.
- `eventSeverity: error` nhưng sự kiện là info → không gửi (đúng thiết kế).

### 14.3. Lệnh "cấp cứu"

```bash
# Reconcile ngay thay vì chờ interval
flux reconcile source git flux-system
flux reconcile kustomization apps --with-source
flux reconcile helmrelease redis -n demo-infra

# Tạm dừng toàn bộ sync của một Kustomization để điều tra (không mất cấu hình)
flux suspend kustomization apps
# ... điều tra xong ...
flux resume kustomization apps

# Xem thay đổi sẽ apply mà chưa apply (dry-run diff)
flux diff kustomization apps --path ./apps/demo

# Export trạng thái để gửi cho người hỗ trợ
flux get all -A -o yaml > flux-state.yaml
```

---

## 15. FAQ và Cheat sheet

### 15.1. FAQ

**Flux có bắt buộc dùng Kustomize không?**
Không. Kustomization apply được cả plain YAML. Kustomize chỉ là lớp tiện ích (patch, overlay, substitute); Helm chart thì dùng HelmRelease.

**Flux có UI không?**
Không có UI chính thức. Quan sát bằng `flux get`, `kubectl`, Grafana (Flux expose Prometheus metrics), hoặc công cụ cộng đồng. Nếu UI là yêu cầu cứng, cân nhắc Argo CD (mục 2).

**Một cluster có thể dùng nhiều Git repo không?**
Có — tạo nhiều GitRepository, mỗi Kustomization trỏ tới `sourceRef` khác nhau. Mô hình phổ biến: 1 repo hạ tầng + 1 repo (hoặc nhiều repo) cho app/tenant.

**Xóa file YAML khỏi Git thì resource trong cluster có bị xóa không?**
Có, nếu Kustomization bật `prune: true` (khuyên dùng). Nếu `prune: false`, resource bị bỏ lại — phải xóa tay.

**Đổi interval của Flux thế nào cho đúng cách?**
Sửa file YAML trong Git, commit, push. Tuyệt đối không `kubectl edit` trực tiếp trên cluster — lần reconcile sau sẽ ghi đè.

### 15.2. Cheat sheet — lệnh flux hay dùng

**Cài đặt & kiểm tra:**

```bash
flux check --pre              # kiểm tra tiền điều kiện trước khi cài
flux check                    # kiểm tra các component Flux đang chạy
flux --version                # phiên bản CLI
flux install --version=v2.x.x # cài Flux vào cluster (không qua bootstrap)
flux uninstall                # gỡ Flux
flux uninstall --keep-namespace  # gỡ nhưng giữ namespace và resource đã deploy
```

**Bootstrap:**

```bash
flux bootstrap github --owner=USER --repository=REPO --branch=main --path=clusters/prod --personal
flux bootstrap gitlab --owner=GROUP --repository=REPO --branch=main --path=clusters/prod
flux bootstrap git --url=ssh://git@host/repo.git --branch=main --path=clusters/prod --private-key-file=./key
```

**Quan sát trạng thái:**

```bash
flux get all                          # tất cả Flux resources
flux get all -A                       # mọi namespace
flux get sources git                  # GitRepository
flux get sources helm                 # HelmRepository
flux get sources chart                # HelmChart
flux get kustomizations               # Kustomization
flux get helmreleases -A              # HelmRelease
flux get images repository -A         # ImageRepository
flux get images policy -A             # ImagePolicy (+ tag đang chọn)
flux get alerts -A                    # Alert
flux get receivers -A                 # Receiver
```

**Reconcile thủ công (không chờ interval):**

```bash
flux reconcile source git flux-system
flux reconcile kustomization apps --with-source   # pull source mới nhất rồi apply
flux reconcile helmrelease redis -n demo-infra
flux reconcile image repository api              # quét tag mới ngay
flux reconcile image update fleet-images         # chạy automation ngay
```

**Suspend / Resume:**

```bash
flux suspend kustomization apps
flux resume kustomization apps
flux suspend helmrelease redis -n demo-infra
flux resume helmrelease redis -n demo-infra
flux suspend source git fleet-infra
flux resume source git fleet-infra
```

**Tạo tài nguyên nhanh bằng CLI:**

```bash
# GitRepository public
flux create source git webapp \
  --url=https://github.com/org/webapp \
  --branch=main --interval=1m

# HelmRepository
flux create source helm bitnami \
  --url=https://charts.bitnami.com/bitnami --interval=30m

# Kustomization
flux create kustomization webapp \
  --source=GitRepository/webapp \
  --path="./deploy/prod" \
  --prune=true --wait=true --interval=5m \
  --validation=server

# HelmRelease
flux create helmrelease redis \
  --source=HelmRepository/bitnami.flux-system \
  --chart=redis --chart-version="18.x" \
  --target-namespace=demo-infra --create-target-namespace=true \
  --interval=10m
```

**Image automation:**

```bash
flux create image repository api --image=ghcr.io/org/api --interval=5m
flux create image policy api --image-ref=api --select-semver=">=1.0.0 <2.0.0"
flux create image update fleet-images \
  --source=GitRepository/fleet-infra \
  --path="./apps/prod" --checkout-branch=main \
  --author-name=fluxcdbot --author-email=fluxcdbot@example.com \
  --push-branch=main --interval=10m
```

**Debug & log:**

```bash
flux logs --level=error --follow                  # mọi lỗi từ mọi controller
flux logs --kind=Kustomization --name=apps        # log 1 Kustomization
flux logs --kind=HelmRelease --name=redis -n demo-infra --follow
flux diff kustomization apps                      # xem trước thay đổi (dry-run)
flux tree kustomization apps                      # cây tài nguyên do Kustomization quản lý
flux stats                                        # thống kê nhanh
```

**SOPS:**

```bash
sops --encrypt --in-place <file>.sops.yaml   # mã hóa
sops <file>.sops.yaml                        # mở sửa (tự giải mã/mã hóa)
sops --decrypt <file>.sops.yaml              # xem bản giải mã
```

---

## Kết luận

Flux CD mang triết lý GitOps thuần khiết vào Kubernetes: Git là nguồn chân lý duy nhất, cluster tự pull và tự chữa drift. Lộ trình gợi ý:

1. Dựng cluster thử nghiệm (kind/k3s), chạy `flux bootstrap` theo mục 3.
2. Làm theo dự án thực tế ở mục 11 — deploy app Kustomize + Helm Redis.
3. Thêm SOPS (mục 10) để quản lý secret đúng cách ngay từ đầu.
4. Cấu hình Alert tới Slack/Telegram (mục 8) — đừng vận hành "mù".
5. Mở rộng sang image automation (mục 7) và multi-tenancy (mục 9) khi đã quen.

Chúc bạn GitOps vui vẻ!
