# Hướng dẫn Argo CD — Deploy Kubernetes theo chuẩn GitOps

> Argo CD là công cụ **GitOps** phổ biến nhất cho Kubernetes: toàn bộ trạng thái mong muốn của hệ thống nằm trong Git, Argo CD tự động đồng bộ (sync) Git → cluster, đảm bảo cluster luôn giống hệt những gì Git mô tả.

Mục lục:

1. [GitOps là gì](#1-gitops-là-gì)
2. [Argo CD là gì và kiến trúc](#2-argo-cd-là-gì-và-kiến-trúc)
3. [Cài đặt Argo CD](#3-cài-đặt-argo-cd)
4. [Kết nối Git repo và cluster](#4-kết-nối-git-repo-và-cluster)
5. [Application CRD](#5-application-crd)
6. [Sync policies](#6-sync-policies)
7. [Health checks](#7-health-checks)
8. [RBAC](#8-rbac)
9. [SSO](#9-sso)
10. [Notifications](#10-notifications)
11. [ApplicationSets](#11-applicationsets)
12. [Argo CD Image Updater](#12-argo-cd-image-updater)
13. [Dự án thực tế: deploy app Node.js](#13-dự-án-thực-tế-deploy-app-nodejs)
14. [Best practices](#14-best-practices)
15. [Troubleshooting](#15-troubleshooting)
16. [FAQ và cheat sheet](#16-faq-và-cheat-sheet)

---

## 1. GitOps là gì

### Định nghĩa đơn giản

GitOps là cách vận hành hạ tầng và ứng dụng mà trong đó **Git là nguồn sự thật duy nhất (single source of truth)**. Mọi thứ — manifest Kubernetes, cấu hình, Helm values — đều nằm trong Git. Một "agent" chạy trong cluster liên tục so sánh trạng thái thực tế của cluster với trạng thái mong muốn trong Git, và tự động sửa mọi khác biệt.

Quy trình chuẩn của GitOps:

```
Developer ──push──▶ Git repo ──pull/sync──▶ Argo CD ──apply──▶ Kubernetes cluster
                              (agent trong cluster tự động kéo, không cần push từ ngoài vào)
```

### Tại sao nên dùng GitOps thay vì CI/CD truyền thống

| Tiêu chí | CI/CD truyền thống (push-based) | GitOps (pull-based) |
|---|---|---|
| Cơ chế deploy | Pipeline CI chạy `kubectl apply` từ ngoài **đẩy** vào cluster | Agent trong cluster **kéo** từ Git và tự đồng bộ |
| Bảo mật | CI cần credential của cluster (kubeconfig) — lộ là nguy hiểm | Không cần mở quyền truy cập cluster cho CI; chỉ cần quyền read Git |
| Kiểm tra (audit) | Khó biết ai deploy cái gì, khi nào | Mọi thay đổi đều là commit Git: ai, khi nào, nội dung gì — rõ ràng |
| Rollback | Chạy lại pipeline cũ hoặc deploy tay | `git revert` là xong, agent tự sync lại |
| Drift (lệch trạng thái) | Ai đó `kubectl edit` tay là cluster lệch mà không ai hay | Agent phát hiện và tự sửa về đúng trạng thái Git (self-heal) |
| Disaster recovery | Dựng lại cluster thủ công, nhớ từng bước | Cluster mới + trỏ agent vào Git là tự tái tạo toàn bộ |

**Tóm lại:** CI/CD truyền thống giải quyết "build và test", GitOps giải quyết "deploy và vận hành" một cách khai báo (declarative), có thể kiểm toán, dễ rollback.

> GitOps không thay thế CI. Quy trình chuẩn: **CI build image → push image → GitOps deploy**.

---

## 2. Argo CD là gì và kiến trúc

### Argo CD là gì

Argo CD là công cụ **Continuous Delivery theo mô hình GitOps** cho Kubernetes, thuộc dự án Argo (CNCF graduated). Nó theo dõi Git repo chứa manifest (YAML thuần, Helm, Kustomize, Jsonnet...), và đảm bảo trạng thái trong cluster khớp với Git.

Điểm mạnh nổi bật:

- Hỗ trợ nhiều định dạng: plain YAML, Helm, Kustomize, Ksonnet, Jsonnet, plugin tùy biến.
- Giao diện web trực quan: nhìn thấy từng resource, trạng thái sync/health theo thời gian thực.
- Tự động sync + tự chữa (self-heal) khi có drift.
- SSO, RBAC, multi-cluster, quản lý nhiều team qua Projects.
- Là chuẩn công nghiệp: dùng rộng rãi từ startup tới enterprise.

### Kiến trúc

Argo CD chạy **trong** cluster Kubernetes, gồm 3 thành phần chính:

```
┌─────────────────────────────────────────────────────┐
│                    Argo CD (namespace argocd)        │
│                                                      │
│  ┌──────────────┐   ┌──────────────┐   ┌───────────┐  │
│  │ API Server   │   │ Repo Server  │   │Application│  │
│  │ (argocd-     │◀─▶│ (argocd-     │   │Controller │  │
│  │  server)     │   │  repo-server)│   │(argocd-   │  │
│  │              │   │              │   │ application│  │
│  │ - REST/gRPC  │   │ - Clone Git  │   │ - So sánh │  │
│  │ - Web UI     │   │ - Render     │   │   Git vs  │  │
│  │ - CLI        │   │   manifest   │   │   cluster │  │
│  │ - Auth/RBAC  │   │ - Cache      │   │ - Sync    │  │
│  └──────────────┘   └──────────────┘   │ - Health  │  │
│                                        └───────────┘  │
└─────────────────────────────────────────────────────┘
         │                       │               │
         ▼                       ▼               ▼
      Người dùng              Git repos      Kubernetes
      (UI/CLI)                               (in-cluster
                                              hoặc remote)
```

**1. API Server (`argocd-server`)**: REST/gRPC cho UI web, CLI `argocd`; xử lý xác thực (admin local, SSO qua Dex, token) và RBAC; mọi thao tác quản trị đều đi qua đây.

**2. Repo Server (`argocd-repo-server`)**: clone/pull Git, render manifest (Helm template, Kustomize build...), cache kết quả.

**3. Application Controller (`argocd-application-controller`)**: "bộ não" — vòng lặp so sánh trạng thái mong muốn (Git) với trạng thái thực tế (cluster), quyết định `Synced`/`OutOfSync` và `Healthy`/`Degraded`, thực hiện sync. Có thể chạy nhiều replica (sharding) khi quản lý hàng nghìn app.

Ngoài ra còn có **Dex** (cầu nối SSO), **Redis** (cache), **Notifications controller** (gửi thông báo).

### So sánh ngắn với các GitOps tool khác

| | **Argo CD** | **Flux CD** | **Jenkins X** |
|---|---|---|---|
| Mô hình | Pull-based, agent trong cluster | Pull-based, agent trong cluster | Push + GitOps lai |
| Giao diện | Web UI rất mạnh, trực quan | Không có UI chính thức (dùng CLI) | UI cơ bản |
| Đa cluster | Hỗ trợ native, quản lý tập trung | Hỗ trợ, cấu hình phân tán hơn | Hỗ trợ |
| Định dạng manifest | YAML, Helm, Kustomize, Jsonnet, plugin | Kustomize native, Helm qua controller riêng | Helm chủ yếu |
| Cộng đồng/hệ sinh thái | Rất lớn, nhiều extension (Image Updater, Rollouts...) | Lớn, CNCF | Nhỏ hơn |
| Độ phức tạp | Trung bình — cài 1 file YAML là chạy | Thấp hơn ở bản đơn giản, nhưng nhiều CRD rời rạc | Cao |
| Phù hợp | Team muốn UI trực quan, quản lý tập trung nhiều app/cluster | Team thích CLI/Git-native, kiến trúc micro-controller | Team đã quen Jenkins |

> Kết luận nhanh: nếu bạn muốn **nhìn thấy** toàn bộ trạng thái deploy trên một dashboard và quản lý nhiều môi trường/cluster tập trung, Argo CD là lựa chọn phổ biến nhất hiện nay.

---
## 3. Cài đặt Argo CD

### Yêu cầu

- Một Kubernetes cluster đang chạy (minikube, kind, k3s, hoặc cloud: GKE/EKS/AKS đều được). Argo CD cần khoảng **2 CPU / 4GB RAM** trống cho bản cài đầy đủ.
- `kubectl` đã cấu hình và trỏ đúng vào cluster (kiểm tra bằng `kubectl cluster-info`).
- Quyền cluster-admin trên cluster (để cài CRD và các ClusterRole).

### Bước 1: Cài đặt bằng manifest chính thức

```bash
# Tạo namespace riêng cho Argo CD
kubectl create namespace argocd

# Cài bản stable mới nhất
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

> Manifest `install.yaml` là bản đầy đủ (HA với nhiều replica). Nếu máy yếu hoặc chỉ muốn thử nghiệm, dùng bản gọn nhẹ:
>
> ```bash
> kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
> ```
>
> (Bản gọn `install.yaml` thay bằng `manifests/install.yaml` — thực tế chỉ có 1 file install chuẩn; với môi trường nhỏ, sau khi cài có thể scale các deployment về 1 replica.)

Kiểm tra các pod đã chạy (`kubectl get pods -n argocd`) — cả 7 deployment (`application-controller`, `applicationset-controller`, `dex-server`, `notifications-controller`, `redis`, `repo-server`, `server`) đều phải ở trạng thái `Running`.

### Bước 2: Lấy mật khẩu admin ban đầu

Khi cài mới, Argo CD tạo một Secret chứa mật khẩu ngẫu nhiên cho user `admin`:

```bash
# Cách 1: đọc trực tiếp từ secret (mật khẩu đã base64)
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d && echo

# Cách 2: xem log của argocd-server lúc khởi động (nếu chưa xóa secret)
```

> Sau khi đăng nhập lần đầu, **nên đổi mật khẩu ngay** (UI → User Info → Update Password, hoặc CLI `argocd account update-password`). Khi đã đổi xong, có thể xóa secret `argocd-initial-admin-secret` cho gọn:
>
> ```bash
> kubectl -n argocd delete secret argocd-initial-admin-secret
> ```

### Bước 3: Truy cập giao diện web

Mặc định `argocd-server` là ClusterIP, không truy cập được từ ngoài. Cách nhanh nhất để thử là port-forward:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Mở trình duyệt vào **https://localhost:8080** (chấp nhận cảnh báo chứng chỉ tự ký), đăng nhập:

- Username: `admin`
- Password: mật khẩu lấy ở bước 2

Môi trường thật thì expose bằng **LoadBalancer** (`kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"LoadBalancer"}}'`) hoặc **Ingress** trỏ tới `argocd-server:443` (dùng SSL passthrough vì server đã có TLS sẵn).

### Bước 4: Cài CLI `argocd`

CLI giúp thao tác nhanh từ terminal (tạo app, sync, xem log...).

```bash
# Linux (amd64)
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

# macOS (dùng Homebrew)
brew install argocd

# Kiểm tra
argocd version --client
```

Đăng nhập CLI (khi đang port-forward ở cổng 8080):

```bash
argocd login localhost:8080 --username admin --password '<mat-khau-admin>' --insecure
```

> `--insecure` bỏ qua kiểm tra chứng chỉ tự ký — chỉ dùng cho môi trường thử nghiệm. Môi trường thật nên dùng chứng chỉ hợp lệ.

Đổi mật khẩu admin bằng CLI:

```bash
argocd account update-password
# Nhập mật khẩu hiện tại, rồi nhập mật khẩu mới 2 lần
```

---

## 4. Kết nối Git repo và cluster

Trước khi tạo Application, Argo CD cần biết: **lấy code từ Git repo nào** và **deploy lên cluster nào**.

### 4.1. Thêm Git repository

#### Repo public (không cần xác thực)

```bash
argocd repo add https://github.com/my-org/my-app-config.git
```

#### Repo private qua HTTPS (username + password/token)

Với GitHub, dùng Personal Access Token (PAT) thay cho mật khẩu:

```bash
argocd repo add https://github.com/my-org/private-config.git \
  --username my-github-user \
  --password ghp_xxxxxxxxxxxxxxxxxxxx
```

#### Repo private qua SSH

Tạo SSH key không passphrase, thêm public key vào Deploy Keys của repo:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/argocd_deploy -N "" -C "argocd-deploy"
argocd repo add git@github.com:my-org/private-config.git \
  --ssh-private-key-path ~/.ssh/argocd_deploy
```

> **Nên dùng SSH hay HTTPS?** Cả hai đều ổn. SSH gọn vì không lo token hết hạn; HTTPS + token dễ xoay vòng và phù hợp chính sách của nhiều công ty. Quan trọng là key/token chỉ cần quyền **read** repo.

#### Khai báo repo bằng YAML (GitOps cho chính Argo CD!)

Thay vì gõ lệnh, bạn có thể khai báo repo dưới dạng Secret — cách này cho phép quản lý cả cấu hình Argo CD bằng Git:

```yaml
# repo-https.yaml — repo private dùng HTTPS + token
apiVersion: v1
kind: Secret
metadata:
  name: my-private-repo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: https://github.com/my-org/private-config.git
  username: my-github-user
  password: ghp_xxxxxxxxxxxxxxxxxxxx
```

```yaml
# repo-ssh.yaml — repo private dùng SSH
apiVersion: v1
kind: Secret
metadata:
  name: my-private-repo-ssh
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: git
  url: git@github.com:my-org/private-config.git
  sshPrivateKey: |
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...dán private key vào đây...
    -----END OPENSSH PRIVATE KEY-----
```

Áp dụng:

```bash
kubectl apply -f repo-https.yaml -n argocd
```

Kiểm tra repo đã kết nối:

```bash
argocd repo list
# hoặc xem trạng thái kết nối trên UI: Settings → Repositories
```

Trạng thái `Successful` nghĩa là Argo CD đã clone được repo.

#### Repository credentials template (dùng chung cho nhiều repo)

Nếu có nhiều repo cùng một GitHub org, thay vì khai báo từng repo, dùng **credential template** — Argo CD tự áp dụng cho mọi repo khớp URL pattern:

```bash
argocd repocreds add https://github.com/my-org/ \
  --username my-github-user \
  --password ghp_xxxxxxxxxxxxxxxxxxxx
```

### 4.2. Kết nối thêm cluster (multi-cluster)

Mặc định Argo CD quản lý chính cluster nó đang chạy (gọi là `in-cluster`, địa chỉ `https://kubernetes.default.svc`). Để deploy sang cluster khác:

```bash
# Chuyển kubectl sang context của cluster đích
kubectl config use-context <context-cluster-dich>

# Thêm cluster vào Argo CD (lệnh này chạy trên máy có cả 2 context)
argocd cluster add <context-cluster-dich> --name production
```

Lệnh trên sẽ tạo ServiceAccount + ClusterRole trên cluster đích và lưu credential vào Argo CD.

Khai báo bằng YAML (khi đã có bearer token của ServiceAccount trên cluster đích):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: production-cluster
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
stringData:
  name: production
  server: https://203.0.113.10:6443
  config: |
    {
      "bearerToken": "<token-cua-serviceaccount>",
      "tlsClientConfig": {
        "insecure": false,
        "caData": "<ca-crt-base64-cua-cluster-dich>"
      }
    }
```

Kiểm tra:

```bash
argocd cluster list
```

> **Lưu ý bảo mật:** Argo CD mặc định tạo cluster credential với quyền **cluster-admin**. Môi trường thật nên giới hạn: tạo ServiceAccount chỉ có quyền trong các namespace cần deploy, rồi thêm cluster với `--service-account` tùy chỉnh.

---
## 5. Application CRD

`Application` là Custom Resource trung tâm của Argo CD: nó mô tả **"lấy cái gì, từ đâu, deploy đi đâu"**. Mỗi app deploy qua Argo CD tương ứng với một resource `Application`.

### Ví dụ YAML đầy đủ

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-web-app            # Tên app hiển thị trên UI
  namespace: argocd            # Application CR luôn nằm ở namespace argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io  # (tùy chọn) xóa app thì xóa cả resource trên cluster
spec:
  project: default             # Thuộc project nào (phân quyền, giới hạn repo/cluster)

  # ── NGUỒN: lấy manifest từ đâu ──
  source:
    repoURL: https://github.com/my-org/my-app-config.git
    targetRevision: HEAD       # nhánh/tag/commit: HEAD, main, v1.2.3, hoặc SHA
    path: k8s/overlays/prod    # đường dẫn thư mục chứa manifest trong repo
    # Nếu dùng Helm thì thay path bằng:
    # helm:
    #   valueFiles:
    #     - values-prod.yaml
    #   parameters:
    #     - name: image.tag
    #       value: "v1.2.3"
    # Nếu dùng Kustomize thì không cần gì thêm, Argo CD tự nhận biết.

  # ── ĐÍCH: deploy đi đâu ──
  destination:
    server: https://kubernetes.default.svc   # cluster đích (in-cluster)
    # hoặc dùng tên cluster đã add: name: production
    namespace: my-web-app                    # namespace deploy trên cluster đích

  # ── CHÍNH SÁCH SYNC ──
  syncPolicy:
    automated:                # (tùy chọn) tự động sync khi Git thay đổi
      prune: true             # xóa resource trên cluster nếu Git không còn định nghĩa
      selfHeal: true          # tự sửa khi ai đó chỉnh tay trên cluster (chống drift)
    syncOptions:
      - CreateNamespace=true  # tự tạo namespace đích nếu chưa có
    # retry:                  # (tùy chọn) thử lại khi sync lỗi
    #   limit: 5
    #   backoff:
    #     duration: 5s
    #     factor: 2
    #     maxDuration: 3m
```

Áp dụng:

```bash
kubectl apply -f my-web-app.yaml -n argocd
```

Hoặc tạo bằng CLI:

```bash
argocd app create my-web-app \
  --repo https://github.com/my-org/my-app-config.git \
  --path k8s/overlays/prod \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace my-web-app \
  --sync-policy automated \
  --self-heal \
  --auto-prune
```

### Giải thích các trường quan trọng

| Trường | Ý nghĩa |
|---|---|
| `spec.project` | Gom app theo team/môi trường, giới hạn repo và cluster được phép dùng, gắn RBAC |
| `source.repoURL` | URL Git repo (phải khớp repo đã kết nối ở mục 4) |
| `source.targetRevision` | Nhánh (`main`), tag (`v1.2.3`), hoặc SHA commit. `HEAD` = nhánh mặc định của repo |
| `source.path` | Thư mục chứa manifest; dùng `.` nếu ở root repo |
| `destination.server` / `destination.name` | Cluster đích: dùng URL API hoặc tên cluster đã `argocd cluster add` (chỉ dùng 1 trong 2) |
| `destination.namespace` | Namespace trên cluster đích |
| `finalizers` | Có `resources-finalizer` thì xóa Application sẽ xóa luôn resource nó quản lý (cẩn thận khi dùng!) |

### Multi-source (nhiều nguồn cho 1 app)

Từ Argo CD 2.x, một Application có thể lấy manifest từ **nhiều repo** (ví dụ: Helm chart ở repo chung + values riêng của team):

```yaml
spec:
  sources:
    - repoURL: https://github.com/my-org/helm-charts.git
      targetRevision: main
      path: charts/web-app
      helm:
        valueFiles:
          - $values/values-prod.yaml
    - repoURL: https://github.com/my-org/team-values.git
      targetRevision: main
      ref: values            # đặt tên ref để chart chính tham chiếu
  destination:
    server: https://kubernetes.default.svc
    namespace: my-web-app
```

---

## 6. Sync policies

Sync là quá trình Argo CD áp manifest từ Git vào cluster. Có 2 chế độ chính:

### 6.1. Manual sync (mặc định)

Khi Git thay đổi, app chuyển `OutOfSync` và chờ bấm Sync (UI hoặc `argocd app sync my-web-app`). Nên xem diff trước khi apply: `argocd app diff my-web-app`. Phù hợp production cần kiểm soát chặt.

### 6.2. Automated sync

Argo CD tự sync ngay khi phát hiện Git thay đổi:

```yaml
syncPolicy:
  automated:
    prune: true      # Xóa resource thừa trên cluster (không còn trong Git)
    selfHeal: true   # Tự revert chỉnh sửa thủ công trên cluster
```

| Tùy chọn | Mặc định | Ý nghĩa |
|---|---|---|
| `automated` | tắt | Bật là tự sync, tắt là manual |
| `prune` | `false` | `true`: resource nào có trên cluster mà Git không còn định nghĩa sẽ bị xóa |
| `selfHeal` | `false` | `true`: ai `kubectl edit` tay làm lệch cluster → Argo CD tự apply lại theo Git |

> **Cẩn thận với `prune: true` + xóa nhầm file trong Git:** xóa file manifest khỏi Git = Argo CD xóa resource thật trên cluster. Với dữ liệu quan trọng (database...), cân nhắc kỹ hoặc dùng annotation bảo vệ (mục 14).

Bật/tắt nhanh bằng CLI:

```bash
# Bật auto-sync kèm prune + self-heal
argocd app set my-web-app --sync-policy automated --self-heal --auto-prune

# Tắt auto-sync (về manual)
argocd app set my-web-app --sync-policy none
```

### 6.3. Sync waves — thứ tự deploy

Mặc định Argo CD apply tất cả resource gần như cùng lúc. Với app cần thứ tự (database trước, app sau), dùng annotation `argocd.argoproj.io/sync-wave`:

```yaml
# 1. Deploy namespace + secret/config trước (wave -1)
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
---
# 2. Deploy database (wave 0 — mặc định nếu không ghi)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  annotations:
    argocd.argoproj.io/sync-wave: "0"
---
# 3. Deploy app sau cùng (wave 1)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  annotations:
    argocd.argoproj.io/sync-wave: "1"
```

Quy tắc:

- Wave số **âm** chạy trước, số **dương** chạy sau; wave sau chỉ bắt đầu khi wave trước **Healthy** (nếu resource có health check).
- Resource không ghi wave = wave `0`.
- Hook đặc biệt: `argocd.argoproj.io/hook: PreSync` (chạy trước sync, ví dụ backup DB), `PostSync`, `SyncFail`.

Ví dụ PreSync hook chạy job backup trước mỗi lần sync (xóa job sau khi chạy xong nhờ `hook-delete-policy`):

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-backup-before-sync
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
```

### 6.4. Sync options (tùy chọn sync)

Khai báo ở cấp Application hoặc từng resource:

```yaml
syncPolicy:
  syncOptions:
    - CreateNamespace=true      # Tự tạo namespace đích
    - PrunePropagationPolicy=foreground  # Cách xóa resource con khi prune
    - PruneLast=true            # Prune sau cùng (hữu ích khi CRD bị xóa trước CR)
    - Replace=true              # Dùng replace thay vì apply (khi apply báo lỗi immutable field)
    - ServerSideApply=true      # Dùng server-side apply
```

Hoặc annotation trên từng resource:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-options: Prune=false  # resource này không bao giờ bị prune
```

Các annotation điều khiển sync hữu ích: `argocd.argoproj.io/sync-wave` (thứ tự), `argocd.argoproj.io/hook: PreSync|PostSync|SyncFail` (hook), `argocd.argoproj.io/sync-options: Prune=false` (không prune resource này), `argocd.argoproj.io/compare-options: IgnoreExtraneous` (bỏ qua field thừa khi diff).

### 6.5. Selective sync — chỉ sync vài resource

Trên UI: vào app → chọn resource → Sync (chỉ sync resource đã chọn). Bằng CLI:

```bash
argocd app sync my-web-app --resource apps:Deployment:my-web-app
# cú pháp: --resource <group>:<kind>:<name>
```

### 6.6. So sánh 3 chiều và `ignoreDifferences`

Một số resource bị "OutOfSync giả" vì controller khác tự thêm field (ví dụ HPA sửa `replicas`). Bảo Argo CD bỏ qua field đó:

```yaml
spec:
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
    # Hoặc bỏ qua toàn bộ annotation do tool khác quản lý:
    - group: ""
      kind: Service
      jsonPointers:
        - /metadata/annotations
```

---
## 7. Health checks

Sau khi sync, Argo CD không chỉ "apply xong là thôi" — nó liên tục đánh giá **sức khỏe** của từng resource.

### 7.1. Cách Argo CD đánh giá health

Mỗi loại resource có logic health riêng (built-in), ví dụ:

| Resource | Healthy khi | Degraded khi |
|---|---|---|
| Deployment | `updatedReplicas == replicas`, rollout xong | Pod crash, image pull lỗi |
| StatefulSet | Tương tự Deployment | Pod không ready |
| Pod | Phase Running, containers ready | CrashLoopBackOff, Pending lâu |
| Service | Luôn Healthy (trừ khi có lỗi cấu hình) | — |
| Ingress | Có ít nhất 1 rule/host | — |
| Job | Hoàn thành | Thất bại |
| PVC | Bound | Pending |

Trạng thái health tổng của Application = tổng hợp health của các resource con: chỉ cần 1 resource `Degraded` là app báo `Degraded`.

Xem health bằng CLI:

```bash
argocd app get my-web-app
# cột HEALTH hiện Healthy/Degraded/Progressing/Missing...
```

### 7.2. Custom health checks bằng Lua

Với CRD của bên thứ ba (ví dụ: `Certificate` của cert-manager, `PrometheusRule`...), Argo CD không biết logic health → mặc định báo `Healthy` mù. Bạn có thể dạy Argo CD bằng script **Lua** trong ConfigMap `argocd-cm`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  # Health check cho Certificate của cert-manager
  resource.customizations.health.cert-manager.io_Certificate: |
    hs = {}
    if obj.status ~= nil then
      if obj.status.conditions ~= nil then
        for i, condition in ipairs(obj.status.conditions) do
          if condition.type == "Ready" and condition.status == "False" then
            hs.status = "Degraded"
            hs.message = condition.message
            return hs
          end
          if condition.type == "Ready" and condition.status == "True" then
            hs.status = "Healthy"
            hs.message = "Certificate is ready"
            return hs
          end
        end
      end
    end
    hs.status = "Progressing"
    hs.message = "Waiting for certificate"
    return hs
```

Giải thích:

- Key có dạng `resource.customizations.health.<group>_<Kind>`.
- Script Lua nhận biến `obj` (resource dạng JSON), phải trả về table `hs` với `hs.status` là một trong: `Healthy`, `Degraded`, `Progressing`, `Suspended`, `Missing`, `Unknown`.
- Sau khi sửa `argocd-cm`, application controller tự reload (không cần restart).

> Cộng đồng Argo CD duy trì sẵn một kho Lua health check cho hàng trăm CRD phổ biến — khi dùng CRD mới, tìm trước trong kho đó thay vì tự viết.

---

## 8. RBAC

Mặc định chỉ có user `admin` local với toàn quyền. Môi trường team cần phân quyền qua ConfigMap **`argocd-rbac-cm`**.

### Mô hình RBAC của Argo CD

- **Subject**: user, group (từ SSO), hoặc service account.
- **Resource**: `applications`, `clusters`, `repositories`, `projects`, `logs`, `exec`...
- **Action**: `get`, `create`, `update`, `delete`, `sync`, `override`...
- Phân quyền có thể giới hạn theo **project** và theo **tên object** (glob pattern).

### Ví dụ: role `dev` chỉ được xem + sync app của team mình

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  # Định nghĩa policy (cú pháp CSV, giống Casbin)
  policy.csv: |
    # --- Role: developer ---
    # Được xem applications trong project "team-a"
    p, role:dev, applications, get, team-a/*, allow
    # Được sync (nhưng KHÔNG được tạo/xóa app)
    p, role:dev, applications, sync, team-a/*, allow
    # Được xem log của app team mình
    p, role:dev, logs, get, team-a/*, allow
    # Được xem repo/cluster (read-only để UI hiển thị)
    p, role:dev, repositories, get, *, allow
    p, role:dev, clusters, get, *, allow

    # --- Role: admin ---
    p, role:admin, *, *, *, allow

    # --- Gán role cho user/group ---
    # user "duong" (đăng nhập SSO) thuộc team-a → role dev
    g, duong, role:dev
    # cả group "backend-team" từ SSO đều là dev
    g, backend-team, role:dev
    # user "root" là admin
    g, root, role:admin

  # Role mặc định cho user đăng nhập mà không thuộc group nào
  policy.default: role:dev

  # Cho phép dùng cả user local (mặc định chỉ SSO mới áp policy này)
  # policy.matchMode: glob
```

Áp dụng:

```bash
kubectl apply -f argocd-rbac-cm.yaml -n argocd
# argocd-server tự reload, không cần restart
```

### Tạo user local cho dev (không dùng SSO)

```bash
# Thêm user "dev1" có quyền API
argocd account add dev1 --role role:dev

# Cấp token cho CI/automation (hết hạn sau 720h)
argocd account generate-token --account dev1 --expires-in 720h
```

> Token của account bị giới hạn bởi RBAC của role được gán — đây là cách an toàn để CI gọi Argo CD API (ví dụ trigger sync) mà không cần admin.

### Kiểm tra quyền

```bash
# user dev1 có được xóa app team-a/web không?
argocd account can-i delete applications 'team-a/web' --as dev1
# Denied (vì role:dev không có quyền delete)
```

### Lưu ý quan trọng

- `policy.default` áp dụng cho **mọi** user đã xác thực mà không match rule nào — production nên để `policy.default: ""` (không quyền gì) rồi gán tường minh.
- Tên trong policy dạng `<project>/<app-name>`; `*` là tất cả.
- Project (`AppProject` CRD) giới hạn **repo và cluster** mà app trong project được dùng — kết hợp RBAC để team A không deploy nhầm sang cluster của team B.

Ví dụ AppProject giới hạn team-a chỉ dùng repo của team và cluster staging (kết hợp RBAC để team A không deploy nhầm sang cluster của team B):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: team-a
  namespace: argocd
spec:
  sourceRepos:
    - https://github.com/my-org/team-a-*.git
  destinations:
    - namespace: 'team-a-*'
      server: https://staging.example.com:6443
  clusterResourceWhitelist:      # cấm tạo cluster-scoped resource
    - group: ''
      kind: Namespace
```

---
## 9. SSO

Argo CD tích hợp **Dex** — một identity broker — nên cấu hình SSO (đăng nhập bằng GitHub/Google/GitLab/LDAP...) rất đơn giản: chỉ cần sửa ConfigMap `argocd-cm`.

### 9.1. SSO với GitHub

**Bước 1:** Tạo OAuth App trên GitHub (Settings → Developer settings → OAuth Apps): Homepage URL `https://argocd.example.com`, callback URL `https://argocd.example.com/api/dex/callback`. Lấy Client ID/Secret.

**Bước 2:** Cấu hình `argocd-cm`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  url: https://argocd.example.com     # URL public của Argo CD (Dex dùng để tạo callback)
  dex.config: |
    connectors:
      - type: github
        id: github
        name: GitHub
        config:
          clientID: <github-client-id>
          clientSecret: <github-client-secret>
          redirectURI: https://argocd.example.com/api/dex/callback
          orgs:
            - name: my-org            # chỉ member của org này mới đăng nhập được
              # teams:               # (tùy chọn) giới hạn theo team
              #   - backend-team
```

**Bước 3:** Gán RBAC theo GitHub org/team trong `argocd-rbac-cm`:

```yaml
data:
  policy.csv: |
    # group từ Dex có dạng "<org>:<team>", ví dụ "my-org:backend-team"
    g, my-org:backend-team, role:dev
    g, my-org:platform-team, role:admin
```

> Từ Argo CD 2.x, group trả về có dạng `org:team`. Nếu chỉ khai báo `orgs` (không có teams), group sẽ là tên org.

### 9.2. SSO với Google (OIDC)

**Bước 1:** Vào Google Cloud Console → APIs & Services → Credentials → Create OAuth client ID (Web application), thêm Authorized redirect URI: `https://argocd.example.com/api/dex/callback`.

**Bước 2:** Cấu hình tương tự GitHub nhưng dùng connector `oidc` với `issuer: https://accounts.google.com`. Có thể giới hạn theo domain công ty bằng `hostedDomains: [mycompany.com]`, rồi gán RBAC theo email trong `policy.csv` (`g, duong@mycompany.com, role:admin`).

### 9.3. Tắt user admin local (sau khi SSO chạy ổn)

Khi SSO đã hoạt động và đã có admin qua SSO, nên vô hiệu hóa tài khoản `admin` local để giảm bề mặt tấn công:

```yaml
data:
  # Trong argocd-cm
  admin.enabled: "false"
```

> **Cẩn thận:** chỉ tắt khi chắc chắn đăng nhập SSO được và đã gán quyền admin cho ít nhất một user/group SSO. Nếu bị khóa ngoài, xóa dòng này (hoặc set `"true"`) là mở lại được — nhưng cần quyền kubectl vào cluster.

---

## 10. Notifications

Argo CD có **notifications controller** gửi thông báo khi có sự kiện (sync thành công/thất bại, health chuyển Degraded...) qua Slack, Email, webhook, Opsgenie, PagerDuty...

### 10.1. Cấu hình gửi Slack

**Bước 1:** Tạo Slack Incoming Webhook (Slack → Apps → Incoming Webhooks), lấy URL dạng `https://hooks.slack.com/services/XXX/YYY/ZZZ`.

**Bước 2:** Lưu webhook vào Secret và khai báo trong `argocd-notifications-cm`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: argocd-notifications-secret
  namespace: argocd
stringData:
  slack-token: https://hooks.slack.com/services/XXX/YYY/ZZZ
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  # Định nghĩa "dịch vụ" Slack
  service.slack: |
    token: $slack-token
    # username: argocd   # (tùy chọn) tên hiển thị
```

**Bước 3:** Gắn trigger + subscription vào Application (annotation):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-web-app
  namespace: argocd
  annotations:
    # Gửi về Slack channel #deploys khi sync thất bại HOẶC app bị Degraded
    notifications.argoproj.io/subscribe.on-sync-failed.slack: deploys
    notifications.argoproj.io/subscribe.on-deployed.slack: deploys
    notifications.argoproj.io/subscribe.on-health-degraded.slack: deploys
```

Các trigger có sẵn: `on-sync-failed`, `on-sync-succeeded`, `on-deployed` (sync ok + healthy), `on-health-degraded`, `on-sync-running`...

### 10.2. Cấu hình gửi Email

```yaml
# argocd-notifications-cm
data:
  service.email: |
    host: smtp.gmail.com
    port: 587
    from: argocd@mycompany.com
    username: $email-username
    password: $email-password
---
# argocd-notifications-secret
stringData:
  email-username: argocd@mycompany.com
  email-password: <app-password-cua-gmail>
```

Gắn vào app:

```yaml
metadata:
  annotations:
    notifications.argoproj.io/subscribe.on-sync-failed.email: devops@mycompany.com
    notifications.argoproj.io/subscribe.on-health-degraded.email: devops@mycompany.com
```

### 10.3. Template thông báo tùy biến

Muốn nội dung Slack đẹp hơn mặc định, định nghĩa template riêng trong `argocd-notifications-cm` (key `template.<tên>` và `trigger.<tên>`), rồi subscribe trigger tùy biến thay vì trigger mặc định:

```yaml
metadata:
  annotations:
    notifications.argoproj.io/subscribe.on-sync-failed-custom.slack: deploys
```

> **Mẹo:** thêm `oncePer: app.status.sync.revision` trong trigger để mỗi revision lỗi chỉ báo một lần, tránh spam khi controller retry liên tục.

---
## 11. ApplicationSets

Khi có nhiều môi trường (dev/staging/prod) hoặc nhiều cluster, viết tay từng Application rất cực. **ApplicationSet** là CRD cho phép định nghĩa **một template** và tự sinh ra hàng loạt Application từ các "generator".

### 11.1. Ví dụ: multi-environment (dev/staging/prod)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-web-app
  namespace: argocd
spec:
  # Generator: danh sách các giá trị sẽ "nhân bản" template
  generators:
    - list:
        elements:
          - env: dev
            namespace: myapp-dev
            autoSync: "false"      # dev sync tay
          - env: staging
            namespace: myapp-staging
            autoSync: "true"
          - env: prod
            namespace: myapp-prod
            autoSync: "false"      # prod sync tay cho an toàn

  # Template: dùng {{env}}, {{namespace}}... như biến
  template:
    metadata:
      name: 'my-web-app-{{env}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/my-org/my-app-config.git
        targetRevision: HEAD
        path: 'k8s/overlays/{{env}}'   # mỗi env một thư mục kustomize overlay
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{namespace}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        # Chỉ bật auto-sync khi autoSync == "true" — dùng Go template:
        # (Cách đơn giản hơn: tách 2 ApplicationSet, hoặc dùng helm values)
```

> ApplicationSet template hỗ trợ Go template (bật `goTemplate: true`), nên có thể viết `if` trực tiếp trong template — ví dụ chỉ bật `automated` khi `env` là staging.

Kết quả: 1 ApplicationSet sinh ra 3 Application: `my-web-app-dev`, `my-web-app-staging`, `my-web-app-prod`.

### 11.2. Ví dụ: multi-cluster bằng Cluster generator

Tự động tạo 1 app cho **mỗi cluster** đã kết nối (có label `env=prod`):

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: monitoring-agents
  namespace: argocd
spec:
  goTemplate: true
  generators:
    - clusters:
        selector:
          matchLabels:
            env: prod
  template:
    metadata:
      name: 'prometheus-agent-{{.name}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/my-org/infra-charts.git
        targetRevision: main
        path: charts/prometheus-agent
        helm:
          parameters:
            - name: clusterName
              value: '{{.name}}'
      destination:
        server: '{{.server}}'      # URL của từng cluster
        namespace: monitoring
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```

Thêm cluster mới có label `env=prod` → ApplicationSet tự sinh app deploy agent lên đó. Xóa cluster → app tự bị xóa (nếu bật `preserveResourcesOnDeletion: false`).

### 11.3. Các generator khác

| Generator | Dùng khi |
|---|---|
| `list` | Danh sách cố định (env, region...) — đơn giản nhất |
| `clusters` | Sinh app theo cluster đã kết nối (lọc bằng label selector) |
| `git` | Quét repo: mỗi thư mục/file sinh 1 app (monorepo) |
| `pullRequest` | Mỗi PR sinh 1 app (môi trường preview) |
| `matrix` / `merge` | Kết hợp 2 generator (ví dụ: env × cluster) |

---

## 12. Argo CD Image Updater

Bài toán: CI build xong image mới (tag `v1.2.4`), làm sao Git tự cập nhật tag để Argo CD deploy? **Argo CD Image Updater** tự động kiểm tra registry, phát hiện tag mới và **commit ngược lại vào Git** (hoặc chỉ update annotation).

### Cài đặt

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/argocd-image-updater/stable/manifests/install.yaml
```

### Cấu hình cơ bản: tự update theo semver

Giả sử Application dùng Helm chart, image tag nằm ở `values.yaml`:

```yaml
# values.yaml trong Git
image:
  repository: my-registry.com/my-org/web-app
  tag: v1.2.3
```

Thêm annotation vào Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-web-app
  namespace: argocd
  annotations:
    # Kiểm tra tag mới nhất theo semver mỗi 2 phút, cho phép cả patch+minor
    argocd-image-updater.argoproj.io/image-list: app=my-registry.com/my-org/web-app
    argocd-image-updater.argoproj.io/app.update-strategy: semver
    argocd-image-updater.argoproj.io/app.helm.image-tag: image.tag
    argocd-image-updater.argoproj.io/write-back-method: git
```

Giải thích:

- `image-list`: alias `app` → image cần theo dõi.
- `update-strategy: semver`: chỉ update lên version mới hơn theo semver (không nhảy từ `v1.2.3` xuống `latest` bậy bạ).
- `app.helm.image-tag: image.tag`: ghi tag mới vào key `image.tag` của Helm values.
- `write-back-method: git`: commit trực tiếp vào Git repo (nhánh `targetRevision`).

Các chiến lược update: `semver` (theo version, có thể giới hạn bằng `allow-tags` regexp), `newest-build` (tag mới nhất theo thời gian build), `alphabetical`, `digest` (pin digest cho tag `:latest`).

### Xác thực với private registry

```bash
# Tạo credential cho registry (dùng docker login trước)
kubectl create secret docker-registry regcred \
  --docker-server=my-registry.com \
  --docker-username=robot \
  --docker-password='<token>' \
  -n argocd

# Bảo image-updater dùng secret này
kubectl patch secret regcred -n argocd \
  -p '{"metadata":{"labels":{"argocd-image-updater.argoproj.io/secret-type":"pullsecret"}}}'
```

Hoặc khai báo trực tiếp trong annotation (kém an toàn hơn):

```yaml
argocd-image-updater.argoproj.io/app.pullsecret: pullsecret:argocd/regcred
```

### Write-back methods

- `git`: commit trực tiếp vào Git repo (đơn giản, repo config riêng) — **khuyến nghị cho production**.
- `argocd`: chỉ update annotation của Application, không commit Git (thử nhanh).
- `helmvalues`: update file values.yaml riêng (khi chart dùng chung nhiều app).

> Kết hợp `write-back-method: git` với `allow-tags` dạng regexp (ví dụ chỉ nhận `v1.x`) để kiểm soát version.

Xem log image-updater để debug:

```bash
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-image-updater --tail=100
```

---
## 13. Dự án thực tế: deploy app Node.js từ Git lên Kubernetes

Mục tiêu: có một app Node.js chạy trên Kubernetes, mọi thay đổi deploy đều đi qua Git + Argo CD. Làm từng bước từ con số 0.

### Tổng quan kiến trúc

```
┌──────────────┐   docker build/push   ┌──────────────┐
│  Source repo │ ────────────────────▶ │   Registry   │
│  (code Node) │      (CI hoặc tay)    │ (Docker Hub) │
└──────────────┘                       └──────────────┘
        │ thay đổi manifest                     ▲ image mới
        ▼                                       │
┌──────────────┐   Argo CD sync   ┌──────────────┴─────┐
│ Config repo  │ ───────────────▶ │  Kubernetes cluster │
│ (k8s YAML)   │                  │  (namespace demo)   │
└──────────────┘                  └────────────────────┘
```

Dùng **2 repo** (best practice): repo code và repo config tách riêng để CI chỉ build image, GitOps chỉ lo deploy.

### Bước 1: Viết app Node.js

```bash
mkdir -p node-argo-demo && cd node-argo-demo
npm init -y
npm install express
```

`server.js`:

```js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;
const VERSION = process.env.APP_VERSION || 'v1.0.0';

app.get('/', (req, res) => {
  res.json({ message: 'Xin chào từ Argo CD!', version: VERSION });
});

app.get('/health', (req, res) => res.json({ status: 'ok' }));

app.listen(PORT, () => console.log(`Listening on ${PORT} (${VERSION})`));
```

`package.json` (thêm script start) rồi viết `Dockerfile`:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY server.js ./
EXPOSE 3000
CMD ["node", "server.js"]
```

Build và push image (thay `duong` bằng Docker Hub username của bạn):

```bash
docker build -t duong/node-argo-demo:v1.0.0 .
docker push duong/node-argo-demo:v1.0.0
```

Push code lên GitHub repo `node-argo-demo` (repo source).

### Bước 2: Viết manifest Kubernetes (repo config)

Tạo repo Git thứ hai, ví dụ `node-argo-demo-config`, cấu trúc:

```
node-argo-demo-config/
└── k8s/
    ├── deployment.yaml
    ├── service.yaml
    └── kustomization.yaml
```

`k8s/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-argo-demo
  labels:
    app: node-argo-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: node-argo-demo
  template:
    metadata:
      labels:
        app: node-argo-demo
    spec:
      containers:
        - name: app
          image: duong/node-argo-demo:v1.0.0   # ← tag image nằm ở đây
          ports:
            - containerPort: 3000
          env:
            - name: APP_VERSION
              value: "v1.0.0"
          livenessProbe:
            httpGet: { path: /health, port: 3000 }
            initialDelaySeconds: 10
            periodSeconds: 15
          readinessProbe:
            httpGet: { path: /health, port: 3000 }
            initialDelaySeconds: 5
            periodSeconds: 10
          resources:
            requests: { cpu: "100m", memory: "128Mi" }
            limits: { cpu: "500m", memory: "256Mi" }
```

`k8s/service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: node-argo-demo
  labels:
    app: node-argo-demo
spec:
  type: ClusterIP
  selector:
    app: node-argo-demo
  ports:
    - port: 80
      targetPort: 3000
```

`k8s/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```

Commit + push repo config.

### Bước 3: Tạo Application trên Argo CD

```yaml
# node-argo-demo-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: node-argo-demo
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/duong/node-argo-demo-config.git
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: demo
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

```bash
kubectl apply -f node-argo-demo-app.yaml -n argocd
```

Vì bật `automated`, Argo CD tự sync ngay. Kiểm tra:

```bash
argocd app get node-argo-demo
kubectl get pods -n demo
```

Test app:

```bash
kubectl port-forward svc/node-argo-demo -n demo 3000:80
curl localhost:3000
# {"message":"Xin chào từ Argo CD!","version":"v1.0.0"}
```

### Bước 4: Deploy phiên bản mới qua Git (quy trình chuẩn)

1. Sửa code (ví dụ đổi message), build image mới:

```bash
docker build -t duong/node-argo-demo:v1.1.0 .
docker push duong/node-argo-demo:v1.1.0
```

2. Cập nhật tag trong **repo config** (CI có thể làm tự động, hoặc dùng Image Updater ở mục 12): sửa `image: duong/node-argo-demo:v1.1.0` và `APP_VERSION: "v1.1.0"` trong `k8s/deployment.yaml`, rồi `git commit && git push`.

3. Argo CD phát hiện commit mới (mặc định poll mỗi 3 phút, hoặc webhook để nhanh hơn — xem mục 14) → tự sync → pod mới chạy `v1.1.0`.

```bash
curl localhost:3000
# {"message":"Xin chào từ Argo CD!","version":"v1.1.0"}
```

### Bước 5: Rollback khi có sự cố

Vì mọi deploy đều là commit Git, rollback = revert commit:

```bash
git revert HEAD          # revert commit "deploy v1.1.0"
git push
# Argo CD tự sync về v1.0.0
```

Hoặc rollback nhanh trên UI: app → History → Rollback (sau đó nên revert commit trong Git cho đồng bộ).

### Bước 6 (nâng cao): bật Image Updater cho demo này

Thay vì sửa tag bằng tay, để Image Updater tự làm bước 2:

```yaml
metadata:
  annotations:
    argocd-image-updater.argoproj.io/image-list: app=duong/node-argo-demo
    argocd-image-updater.argoproj.io/app.update-strategy: semver
    argocd-image-updater.argoproj.io/app.parameters: image.tag
    # Vì dùng Kustomize thuần (không phải Helm), dùng kustomize image override:
    argocd-image-updater.argoproj.io/write-back-method: git
```

Với Kustomize, image-updater tự chạy tương đương `kustomize edit set image` rồi commit. Quy trình khi đó chỉ còn: **push code → CI build/push image → image-updater commit tag mới → Argo CD sync**.

---
## 14. Best practices

### 14.1. Cấu trúc repo: app-of-apps pattern

Khi có hàng chục app, đừng tạo từng Application bằng tay. Dùng **app-of-apps**: một Application "mẹ" quản lý thư mục chứa các Application "con".

```
infra-repo/
├── bootstrap/
│   └── root-app.yaml          # Application mẹ, path trỏ vào apps/
└── apps/
    ├── api-app.yaml           # Application con: api
    ├── worker-app.yaml        # Application con: worker
    └── monitoring-app.yaml    # Application con: monitoring
```

`bootstrap/root-app.yaml`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/my-org/infra-repo.git
    targetRevision: HEAD
    path: apps          # thư mục chứa các Application con
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Chỉ cần `kubectl apply` **một lần duy nhất** file root. Sau đó thêm app mới = thêm file YAML vào `apps/` → root tự sync → app con tự xuất hiện. Muốn dựng lại toàn bộ hệ thống trên cluster mới: apply 1 file root là xong. Biến thể nâng cao: root là **ApplicationSet** dùng git generator quét thư mục `apps/`.

### 14.2. Tách repo code và repo config

- **Source repo**: code ứng dụng, Dockerfile, CI build/test.
- **Config repo**: manifest Kubernetes, Helm values, Kustomize overlays.

Lợi ích: team dev không cần quyền sửa manifest production; CI của source repo không cần credential cluster; lịch sử deploy (config repo) sạch, chỉ chứa thay đổi deploy.

### 14.3. Quản lý secret: Sealed Secrets hoặc External Secrets

**Tuyệt đối không commit secret dạng plaintext vào Git.** Hai hướng phổ biến:

**Sealed Secrets** (Bitnami): mã hóa secret bằng public key của cluster, commit file mã hóa vào Git an toàn:

```bash
# Cài controller
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.27.3/controller.yaml

# Tạo sealed secret từ secret thường
kubectl create secret generic db-pass --from-literal=password='s3cr3t' \
  --dry-run=client -o yaml | \
  kubeseal --controller-name=sealed-secrets-controller \
           --controller-namespace=kube-system \
           --format yaml > sealed-db-pass.yaml
# Commit sealed-db-pass.yaml vào Git — chỉ cluster có private key mới giải mã được
```

**External Secrets Operator (ESO)**: secret thật nằm ở Vault/AWS Secrets Manager/GCP Secret Manager, Git chỉ chứa "tham chiếu":

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-pass
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: db-pass          # Secret thật được tạo trong cluster
  data:
    - secretKey: password
      remoteRef:
        key: prod/db
        property: password
```

> Lưu ý khi dùng với Argo CD: thêm annotation `argocd.argoproj.io/compare-options: IgnoreExtraneous` hoặc `ignoreDifferences` cho Secret do ESO quản lý để tránh OutOfSync giả (ESO tự thêm field).

### 14.4. Bảo mật Argo CD

- **Tắt admin local** sau khi SSO ổn định (`admin.enabled: "false"`).
- **RBAC tối thiểu**: `policy.default` để trống, gán quyền tường minh theo project.
- **Repo credential**: chỉ cấp quyền read; dùng deploy key/token riêng cho Argo CD, xoay vòng định kỳ.
- **NetworkPolicy**: giới hạn namespace `argocd` chỉ nhận traffic cần thiết; **không expose UI ra internet** nếu không cần.
- **Ký commit / branch protection** cho config repo production: yêu cầu PR review trước khi merge vào `main`.
- **Cập nhật Argo CD thường xuyên**: theo dõi CVE (đã từng có lỗ hổng nghiêm trọng).

### 14.5. Webhook thay vì polling

Mặc định Argo CD poll Git mỗi 3 phút → deploy chậm. Cấu hình webhook để GitHub báo ngay khi có push:

1. Trong `argocd-secret`, đảm bảo có `webhook.github.secret` (chuỗi bí mật tự đặt):

```bash
kubectl -n argocd patch secret argocd-secret \
  -p '{"stringData":{"webhook.github.secret":"<chuoi-bi-mat-cua-ban>"}}'
```

2. Trên GitHub repo → Settings → Webhooks → Add webhook:
   - Payload URL: `https://argocd.example.com/api/webhook`
   - Content type: `application/json`
   - Secret: chuỗi bí mật ở bước 1
   - Events: chỉ chọn **Push**

Từ giờ push code → GitHub gọi webhook → Argo CD refresh repo ngay lập tức.

### 14.6. Tổ chức theo môi trường

```
config-repo/
├── base/                  # manifest chung
│   ├── deployment.yaml
│   └── kustomization.yaml
└── overlays/
    ├── dev/kustomization.yaml      # replicas: 1, image tag mới nhất
    ├── staging/kustomization.yaml  # replicas: 2
    └── prod/kustomization.yaml     # replicas: 5, resource lớn hơn
```

Mỗi môi trường = 1 Application trỏ vào overlay tương ứng; production nên dùng `targetRevision` là tag/release thay vì `HEAD`.

---

## 15. Troubleshooting

### App báo OutOfSync mãi không hết

**Nguyên nhân thường gặp:** diff giả do controller khác tự sửa field (HPA, ESO...) → dùng `ignoreDifferences`; PreSync hook fail → xem log job hook; Argo CD thiếu quyền tạo resource (CRD chưa cài) → xem Operation State trong `argocd app get`.

Debug:

```bash
argocd app diff <app-name>
kubectl -n argocd logs -l app.kubernetes.io/name=argocd-application-controller --tail=200
```

### Repo unreachable / Failed to fetch

```
ComparisonError: Failed to fetch ...: authentication required
```

- Kiểm tra credential: `argocd repo list` xem trạng thái.
- Token hết hạn → cập nhật lại secret repo.
- SSH key sai → test thủ công: `ssh -T git@github.com -i <key>`.
- Egress firewall chặn → Argo CD cần ra internet tới Git host (443 cho HTTPS, 22 cho SSH).
- Repo quá lớn → tăng timeout repo server hoặc dùng shallow clone (mặc định Argo CD đã shallow).

### RBAC denied

```
permission denied: applications, get, team-a/web
```

- Kiểm tra policy: `kubectl -n argocd get cm argocd-rbac-cm -o yaml`.
- Test quyền: `argocd account can-i <action> <resource> --as <user>`.
- Nhớ: user login bằng SSO mới áp `policy.csv` mặc định; muốn áp cho user local thì set `policy.matchMode` hoặc dùng `argocd-rbac-cm` với `accounts.<user>.enabled`.

### Sync fail: "is forbidden" / "is invalid"

- Argo CD ServiceAccount thiếu quyền trên cluster đích → kiểm tra ClusterRoleBinding của `argocd-application-controller`.
- Lỗi `field is immutable`: resource đã tồn tại với field không sửa được (ví dụ `spec.clusterIP` của Service) → dùng sync option `Replace=true` hoặc xóa resource cũ.

### Application kẹt ở trạng thái Progressing

Pod không ready → `kubectl describe pod` xem events (ImagePullBackOff, CrashLoopBackOff...). Health check custom sai → xem log application-controller.

### Xóa Application nhưng resource vẫn còn (hoặc ngược lại)

Mặc định xóa app **giữ lại** resource trên cluster. Muốn xóa kéo theo resource: thêm finalizer `resources-finalizer.argocd.argoproj.io` **trước** khi xóa.

- App đã xóa nhưng finalizer kẹt → edit bỏ finalizer tay: `kubectl -n argocd patch app <name> --type=json -p='[{"op":"remove","path":"/metadata/finalizers"}]'`.

### Argo CD server/UI không vào được

```bash
kubectl get pods -n argocd
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-server --tail=100
kubectl get svc -n argocd   # kiểm tra service rồi port-forward lại
```

### Repo server OOM / chậm khi render Helm chart lớn

Tăng memory limit cho `argocd-repo-server`, giảm số values file, và dùng `ignoreDifferences` cho resource render ra quá nhiều field động.

### Hai app cùng quản lý một resource

Mỗi resource chỉ thuộc về 1 app (theo annotation `argocd.argoproj.io/instance`). → Gộp vào 1 app, hoặc tách namespace/path rõ ràng.

---
## 16. FAQ và cheat sheet

### FAQ

**Q: Argo CD có thay thế Jenkins/GitHub Actions không?**
A: Không. CI (build, test, đóng image) vẫn cần Jenkins/GitHub Actions/Tekton... Argo CD chỉ đảm nhận phần **CD** — đưa manifest từ Git lên cluster. Quy trình chuẩn: CI build image → update config repo → Argo CD sync.

**Q: Nên để manifest ở cùng repo code hay repo riêng?**
A: Repo riêng (config repo) là best practice: tách quyền, lịch sử deploy sạch, CI không cần credential cluster.

**Q: Argo CD quản lý được cluster ở nhà cung cấp khác không?**
A: Được — `argocd cluster add` với kubeconfig của cluster bất kỳ (EKS, GKE, on-prem...).

**Q: Mỗi lần push Git phải đợi bao lâu Argo CD mới deploy?**
A: Mặc định poll 3 phút; cấu hình webhook (mục 14.5) thì chỉ vài giây.

**Q: Có nên bật auto-sync cho production không?**
A: Tùy khẩu vị rủi ro. Nhiều team bật auto-sync cả production (tin vào Git + PR review); team thận trọng thì để manual sync cho prod, auto cho dev/staging.

**Q: Argo CD có deploy được Helm chart không?**
A: Có, native — chỉ cần `path` trỏ vào thư mục chart, thêm `helm.valueFiles`/`helm.parameters` trong `source`.

**Q: Xóa Application có xóa luôn app trên cluster không?**
A: Mặc định **không** — resource trên cluster được giữ lại. Muốn xóa theo thì thêm finalizer `resources-finalizer.argoproj.io` trước khi xóa.

**Q: Làm sao deploy 1 app lên 10 cluster?**
A: Dùng ApplicationSet với `clusters` generator (mục 11.2) — 1 định nghĩa sinh 10 app.

**Q: Argo CD có tốn tài nguyên không?**
A: Bản đầy đủ cần ~2 CPU/4GB RAM cho control plane; cluster nhỏ có thể scale các deployment về 1 replica. Quản lý hàng nghìn app thì cần sharding application-controller.

### Cheat sheet — lệnh hay dùng

```bash
# ── Cài đặt & đăng nhập ──
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
kubectl port-forward svc/argocd-server -n argocd 8080:443
argocd login localhost:8080 --username admin --password '<pass>' --insecure

# ── Repo & cluster ──
argocd repo add https://github.com/my-org/config.git
argocd repo add git@github.com:my-org/private.git --ssh-private-key-path ~/.ssh/argocd_deploy
argocd repo list
argocd cluster add <context> --name production
argocd cluster list

# ── Application ──
argocd app create my-app --repo <url> --path k8s --dest-server https://kubernetes.default.svc --dest-namespace demo
argocd app list
argocd app get my-app
argocd app diff my-app
argocd app sync my-app
argocd app sync my-app --resource apps:Deployment:my-app   # sync 1 resource
argocd app set my-app --sync-policy automated --self-heal --auto-prune
argocd app delete my-app

# ── Theo dõi & debug ──
argocd app wait my-app --health --timeout 300
argocd app logs my-app --follow
kubectl -n argocd logs -l app.kubernetes.io/name=argocd-application-controller --tail=200
```

### Bảng tra nhanh trạng thái

| Sync status | Ý nghĩa |
|---|---|
| `Synced` | Cluster khớp hoàn toàn với Git |
| `OutOfSync` | Có khác biệt — cần sync (hoặc đang chờ sync tay) |
| `Unknown` | Không xác định được (lỗi kết nối repo/cluster) |

| Health status | Ý nghĩa |
|---|---|
| `Healthy` | Resource chạy đúng như mong đợi |
| `Progressing` | Đang triển khai, chưa ổn định |
| `Degraded` | Có lỗi (pod crash, rollout fail...) |
| `Missing` | Manifest có trong Git nhưng chưa tồn tại trên cluster |
| `Suspended` | Bị tạm dừng (ví dụ CronJob suspended) |

---

*Hết hướng dẫn. Chúc bạn deploy GitOps mượt mà!*
