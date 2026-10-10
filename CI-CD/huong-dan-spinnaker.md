# Hướng Dẫn Toàn Diện về Spinnaker — Nền Tảng Continuous Delivery

> Spinnaker là nền tảng Continuous Delivery đa cloud, mã nguồn mở của Netflix (open source 2015). Tài liệu này bao quát: khái niệm, kiến trúc, cài đặt, provider, pipeline, chiến lược deploy, canary analysis, RBAC và vận hành production.

## Mục lục

1. [Spinnaker là gì?](#1-spinnaker-là-gì)
2. [Lịch sử phát triển](#2-lịch-sử-phát-triển)
3. [Kiến trúc microservices](#3-kiến-trúc-microservices)
4. [Cài đặt Spinnaker](#4-cài-đặt-spinnaker)
5. [Cấu hình Providers: Kubernetes, AWS, GCP](#5-cấu-hình-providers-kubernetes-aws-gcp)
6. [Applications: Application, Cluster, Server Group, Load Balancer](#6-applications)
7. [Pipelines: Stages và Triggers](#7-pipelines-stages-và-triggers)
8. [Deployment Strategies chi tiết](#8-deployment-strategies-chi-tiết)
9. [Manual Judgments & Approvals](#9-manual-judgments--approvals)
10. [Canary Analysis với Kayenta](#10-canary-analysis-với-kayenta)
11. [Artifacts & Expected Artifacts](#11-artifacts--expected-artifacts)
12. [Notifications: Slack, Email](#12-notifications-slack-email)
13. [RBAC với Fiat](#13-rbac-với-fiat)
14. [Scaling & Production Considerations](#14-scaling--production-considerations)
15. [Troubleshooting](#15-troubleshooting)
16. [Best Practices Checklist](#16-best-practices-checklist)
17. [FAQ](#17-faq)
18. [Cheat Sheet](#18-cheat-sheet)

---

## 1. Spinnaker là gì?

**Spinnaker** là một nền tảng **Continuous Delivery (CD)** mã nguồn mở, đa cloud (multi-cloud), cho phép các team phát triển phần mềm triển khai (deploy) và quản lý vòng đời ứng dụng một cách nhanh chóng, an toàn và có kiểm soát.

### Spinnaker giải quyết vấn đề gì?

| Vấn đề | Cách Spinnaker giải quyết |
|---|---|
| Deploy thủ công, rủi ro cao | Pipeline tự động hóa với nhiều stage kiểm soát |
| Khó rollback / không biết bản mới có ổn không | Red/Black giữ bản cũ (rollback 1 click); Kayenta canary analysis tự động |
| Mỗi cloud một công cụ khác nhau | Một mặt điều khiển duy nhất cho AWS, GCP, Azure, Kubernetes... |
| Deploy "mù", không ai duyệt | Manual judgment / approval gates trong pipeline |
| Khó quản lý immutable infrastructure | Mô hình Server Group bất biến, bake image (AMI/GCE image) |

### Các khái niệm cốt lõi

- **Application**: Nhóm logic các microservice liên quan (ví dụ: `checkout-api`).
- **Cluster**: Các Server Group của một app trong một account + region/namespace.
- **Server Group**: Đơn vị deploy bất biến — tương đương Auto Scaling Group (AWS), ReplicaSet/Deployment (K8s V2). Mỗi deploy tạo Server Group **mới**, bản cũ giữ lại để rollback.
- **Load Balancer**: Điều hướng traffic tới các Server Group.
- **Pipeline**: Chuỗi stage (deploy, test, judgment, canary...) chạy tự động hoặc thủ công.
- **Artifact**: Đơn vị versioned deploy được: Docker image, AMI, manifest, Helm chart...
- **Bake**: Tạo machine image (AMI/GCE image) từ artifact + config (Rosco + Packer).

### Spinnaker KHÔNG phải là gì?

- Không phải CI tool (như Jenkins, GitLab CI): Spinnaker là CD, nó **nhận artifact từ CI** rồi deploy. Jenkins build → Spinnaker deploy là mô hình chuẩn.
- Không phải orchestrator thay thế Kubernetes: Spinnaker điều phối deploy **lên** Kubernetes/AWS/GCP.
- Không phải GitOps tool (như ArgoCD/Flux): Spinnaker là pipeline-driven, có UI điều khiển trung tâm và canary analysis tích hợp sâu.

---

## 2. Lịch sử phát triển

| Năm | Sự kiện |
|---|---|
| 2012–2014 | Netflix phát triển nội bộ hệ thống deploy thế hệ mới thay thế Asgard (công cụ deploy cũ trên AWS) |
| **11/2015** | **Netflix chính thức open source Spinnaker** tại sự kiện, với sự hỗ trợ của Google, Microsoft, Pivotal |
| 2016 | Google tích hợp hỗ trợ GCP/GKE sâu; cộng đồng đóng góp provider Azure, Cloud Foundry |
| 2017 | Ra mắt **Halyard** — công cụ CLI quản trị vòng đời Spinnaker (cài đặt, cấu hình, nâng cấp), thay thế cách deploy thủ công |
| 2018 | Kayenta (canary analysis tự động) trở thành service chính thức; hỗ trợ Kubernetes provider V2 (manifest-based) thay thế V1 (server-group-based) |
| 2019 | Spinnaker 1.14+ ổn định với Kubernetes V2 làm mặc định; cộng đồng vượt 200+ contributors |
| 2020 | Spinnaker Operator (Kubernetes Operator) ra đời, cho phép cài Spinnaker bằng CRD `SpinnakerService` |
| 2021–2024 | Các bản 1.26–1.35: plugin framework (PF4J), Deck hiện đại hóa, hỗ trợ Helm 3/Kustomize, cải thiện Fiat/RBAC |
| Nay | Dùng bởi Netflix, Google, Microsoft, Target, Adobe... — một trong các CD platform multi-cloud phổ biến nhất |

---

## 3. Kiến trúc microservices

Spinnaker gồm ~10 microservices, mỗi service đảm nhiệm một vai trò riêng, giao tiếp qua REST API và message queue (Redis pub/sub). Tất cả request từ UI (Deck) đều đi qua **Gate** (API gateway).

### Bảng chi tiết từng service

| Service | Ngôn ngữ | Vai trò |
|---|---|---|
| **Deck** | TypeScript | Giao diện web; mọi thao tác gọi API qua Gate |
| **Gate** | Java/Spring | API Gateway: xác thực, định tuyến tới service nội bộ |
| **Orca** | Java/Kotlin | **Pipeline engine**: thực thi pipeline, quản lý stage/task/trigger, retry/rollback |
| **Clouddriver** | Java/Kotlin | **Cloud abstraction**: gọi API AWS/GCP/K8s/Azure..., cache hạ tầng (service nặng nhất) |
| **Front50** | Java | Lưu metadata app/pipeline; backend S3/GCS |
| **Echo** | Java/Kotlin | Event bus + scheduler: webhook, cron, Docker/Jenkins event; gửi notification |
| **Igor** | Java | Tích hợp CI (Jenkins/Travis/Concourse) và artifact repo |
| **Rosco** | Java | **Image bakery**: dùng Packer bake AMI/GCE image |
| **Kayenta** | Java | **Canary analysis tự động**: so sánh metrics, chấm điểm pass/fail |
| **Fiat** | Java | **Phân quyền RBAC**: roles/permissions, sync từ GitHub Teams/LDAP/SAML |
| **Dinghy** | Go | **Pipeline as Code**: pipeline định nghĩa trong Git |
| **Spinnaker Operator** | Go | K8s Operator quản lý Spinnaker qua CRD (thay Halyard) |

### Luồng request điển hình (bấm "Deploy" trên UI)

1. **Deck** → **Gate** (xác thực qua Fiat) → **Orca** tạo pipeline execution, lưu Redis.
2. Orca gọi **Clouddriver** thao tác cloud (tạo server group, resize, attach LB...).
3. Clouddriver gọi API thật của AWS/GCP/K8s, cache kết quả vào Redis cho Deck hiển thị.
4. **Echo** gửi notification (Slack/email); stage canary thì Orca gọi **Kayenta** chấm điểm metrics.

### Các thành phần hạ tầng phụ trợ

| Thành phần | Dùng cho |
|---|---|
| **Redis** | Cache của Clouddriver, queue/pub-sub của Orca/Echo, lưu execution đang chạy |
| **S3 / GCS / Azure Storage / MinIO** | Persistent storage của Front50 (định nghĩa pipeline/app) |
| **MySQL/PostgreSQL** (optional) | Thay Redis/SQL cho Orca execution repository ở quy mô lớn (khuyến nghị production) |

---
## 4. Cài đặt Spinnaker

Có 3 cách phổ biến: **Halyard** (khuyến nghị, quản trị vòng đời đầy đủ), **Docker** (nhanh, thử nghiệm), **Minikube/Spinnaker Operator** (dev, học tập). Yêu cầu chung: Ubuntu 20.04/22.04, RAM tối thiểu 4GB (khuyến nghị 8GB+ cho production), Java 11 cho Halyard.

### 4.1. Cài đặt bằng Halyard trên Ubuntu (khuyến nghị)

**Bước 1 — Cài Halyard:**

```bash
# Cập nhật hệ thống và cài Java 11
sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-11-jdk curl

# Cài Halyard
curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/debian/InstallHalyard.sh
sudo bash InstallHalyard.sh

# Kiểm tra
hal --version
```

**Bước 2 — Chọn phiên bản Spinnaker và tài khoản deploy:**

```bash
# Xem các phiên bản khả dụng
hal version list

# Chọn phiên bản (ví dụ 1.33.3 — thay bằng bản mới nhất từ list)
hal config version edit --version 1.33.3

# Spinnaker sẽ chạy dưới user nào trên máy này
sudo update-alternatives --config java   # đảm bảo Java 11
hal config deploy edit --type localdebian --account-name my-spinnaker-account
# Hoặc deploy phân tán (distributed) lên Kubernetes cluster:
# hal config deploy edit --type distributed --account-name my-k8s-account --location spinnaker
```

> **Lưu ý:** `localdebian` cài tất cả service trên cùng máy (phù hợp lab/1 node). `distributed` deploy lên Kubernetes (khuyến nghị production).

**Bước 3 — Cấu hình persistent storage (bắt buộc):**

Dùng S3 (AWS), GCS (GCP), Azure Storage, hoặc MinIO (tự host, S3-compatible):

```bash
# Ví dụ với MinIO tự host (không cần cloud)
hal config storage s3 edit \
  --endpoint http://minio.internal:9000 \
  --access-key-id $MINIO_ACCESS_KEY \
  --secret-access-key \
  --bucket spinnaker-storage \
  --root-folder front50 \
  --region us-east-1 \
  --path-style-access true

hal config storage edit --type s3
```

Với GCS:

```bash
hal config storage gcs edit \
  --bucket spinnaker-storage \
  --project my-gcp-project \
  --json-path /home/spinnaker/gcs-key.json

hal config storage edit --type gcs
```


**Bước 4 — Cấu hình Redis (mặc định dùng Redis nội bộ của Halyard, production nên tách riêng):**

```bash
hal config storage redis edit --address redis://redis.internal:6379
```

**Bước 5 — Bật provider (xem chi tiết ở mục 5) — ví dụ Kubernetes:**

```bash
hal config provider kubernetes account add my-k8s \
  --provider-version v2 \
  --context minikube \
  --kubeconfig-file ~/.kube/config \
  --only-spinnaker-managed true

hal config provider kubernetes enable
```

**Bước 6 — Cấu hình địa chỉ truy cập UI/API (để dùng domain thay vì localhost):**

```bash
hal config security ui edit \
  --override-base-url http://spinnaker.example.com

hal config security api edit \
  --override-base-url http://spinnaker-api.example.com
```

**Bước 7 — Deploy:**

```bash
# Áp dụng toàn bộ cấu hình và khởi động Spinnaker
sudo hal deploy apply
```

Quá trình này mất 5–15 phút. Kiểm tra trạng thái:

```bash
# Xem log deploy
tail -f /var/log/spinnaker/halyard/halyard.log

# Liệt kê service đang chạy (localdebian)
sudo hal deploy connect   # mở tunnel tới UI/API khi chạy local
```

Sau khi deploy xong, mở UI tại `http://localhost:9000` (Deck) và API tại `http://localhost:8084` (Gate).

Các lệnh quản trị vòng đời (`hal deploy apply/diff/rollback`, `hal version list`) được tổng hợp ở [Cheat Sheet](#18-cheat-sheet).

### 4.2. Cài đặt nhanh bằng Docker (thử nghiệm)

Dùng image Halyard chính thức, mount `~/.hal` để giữ config:

```bash
mkdir -p ~/.hal ~/.kube

docker run -p 8084:8084 -p 9000:9000 \
  --name halyard --rm \
  -v ~/.hal:/root/.hal \
  -v ~/.kube:/root/.kube \
  -d docker.io/armory/halyard:stable

docker exec -it halyard bash
```

Bên trong container, chạy các lệnh `hal config ...` như mục 4.1, rồi:

```bash
hal deploy apply
```

Mở trình duyệt: `http://localhost:9000` (cách này phù hợp demo/dev).

### 4.3. Dev setup với Minikube + Spinnaker Operator

**Bước 1 — Cài kubectl + Minikube (theo docs chính thức) và khởi động cluster đủ tài nguyên:**

```bash
minikube start --cpus=4 --memory=8192 --disk-size=40g --driver=docker
```

**Bước 2 — Cài Spinnaker Operator:**

```bash
# Tạo namespace và cài operator
kubectl create ns spinnaker-operator
kubectl apply -n spinnaker-operator -f \
  https://raw.githubusercontent.com/armory/spinnaker-operator/master/deploy/operator/basic/crds.yaml
kubectl apply -n spinnaker-operator -f \
  https://raw.githubusercontent.com/armory/spinnaker-operator/master/deploy/operator/basic/operator.yaml
```

**Bước 3 — Khai báo SpinnakerService CRD:**

```yaml
# spinnakerservice.yaml
apiVersion: spinnaker.armory.io/v1alpha2
kind: SpinnakerService
metadata:
  name: spinnaker
  namespace: spinnaker
spec:
  spinnakerConfig:
    config:
      version: 1.33.3
      providers:
        kubernetes:
          enabled: true
          accounts:
            - name: minikube
              providerVersion: v2
              kubeconfigFile: /tmp/kubeconfig
              onlySpinnakerManaged: true
              namespaces:
                - default
                - staging
      storage:
        persistentStoreType: s3
        s3:
          bucket: spinnaker-storage
          endpoint: http://minio.minio-ns:9000
          accessKeyId: ENC[k8s:secret/minio-secret!data.access-key]
          secretAccessKey: ENC[k8s:secret/minio-secret!data.secret-key]
          pathStyleAccess: true
```

```bash
kubectl create ns spinnaker && kubectl apply -f spinnakerservice.yaml
kubectl -n spinnaker get spinsvc spinnaker -w   # theo dõi tiến trình
kubectl -n spinnaker port-forward svc/spin-deck 9000:9000   # mở UI Deck
```

### 4.4. Kiểm tra cài đặt thành công

```bash
curl http://localhost:8084/health        # phải trả về {"status":"UP"}
curl http://localhost:8084/credentials   # kiểm tra provider đã nhận account
```

Mở Deck ở cổng 9000 để bắt đầu tạo Application.

---
## 5. Cấu hình Providers: Kubernetes, AWS, GCP

Provider là "tài khoản cloud" mà Spinnaker dùng để deploy. Mỗi provider có thể có nhiều **account** (ví dụ: AWS prod, AWS staging; K8s cluster A, B).

### 5.1. Docker Registry (điều kiện tiên quyết cho mọi provider)

Spinnaker cần biết Docker registry để trigger pipeline khi có image mới và resolve artifact:

```bash
# Docker Hub
hal config provider docker-registry account add my-docker-hub \
  --address index.docker.io \
  --repositories myorg/api,myorg/worker \
  --username mydockeruser \
  --password-file /home/spinnaker/.docker-hub-password.txt

# AWS ECR (dùng password helper qua AWS CLI)
hal config provider docker-registry account add my-ecr \
  --address 123456789012.dkr.ecr.ap-southeast-1.amazonaws.com \
  --repositories myorg/api \
  --username AWS \
  --password-command "aws ecr get-login-password --region ap-southeast-1"

hal config provider docker-registry enable
```

> `--repositories` giới hạn repo Spinnaker theo dõi (tránh scan cả registry nặng). Có thể cập nhật sau: `hal config provider docker-registry account edit my-docker-hub --add-repository myorg/new-svc`.

### 5.2. Kubernetes Provider (V2 — manifest-based)

Kubernetes V2 deploy bằng **manifest** (YAML) thay vì mô hình server group truyền thống. Đây là provider phổ biến nhất hiện nay.

**Chuẩn bị ServiceAccount cho Spinnaker trong cluster đích:**

```yaml
# spinnaker-serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: spinnaker-serviceaccount
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: spinnaker-clusterrole
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: spinnaker-clusterrolebinding
subjects:
  - kind: ServiceAccount
    name: spinnaker-serviceaccount
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: spinnaker-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f spinnaker-serviceaccount.yaml

# Lấy token (K8s < 1.24 tự tạo secret; K8s >= 1.24 tạo thủ công)
kubectl create token spinnaker-serviceaccount -n kube-system --duration=87600h > /tmp/spinnaker-token.txt

# Lấy CA cert
kubectl config view --raw -o jsonpath='{.clusters[?(@.name=="minikube")].cluster.certificate-authority-data}' | base64 -d > /tmp/spinnaker-ca.crt
```

**Thêm account Kubernetes vào Spinnaker:**

```bash
# Cách 1: dùng kubeconfig có sẵn
hal config provider kubernetes account add my-k8s-prod \
  --provider-version v2 \
  --context prod-cluster \
  --kubeconfig-file ~/.kube/config \
  --docker-registries my-docker-hub \
  --namespaces default,production,staging \
  --omit-namespaces kube-system,kube-public \
  --only-spinnaker-managed true \
  --check-permissions true

# Cách 2: dùng token + CA (không cần kubeconfig trên máy Halyard)
hal config provider kubernetes account add my-k8s-token \
  --provider-version v2 \
  --service-account true \
  --token-file /tmp/spinnaker-token.txt \
  --configure-image-pull-secrets true

hal config provider kubernetes enable
```

Giải thích các flag quan trọng:

| Flag | Ý nghĩa |
|---|---|
| `--provider-version v2` | Dùng provider manifest-based (bắt buộc cho bản mới) |
| `--docker-registries` | Registry account để resolve image trong manifest |
| `--namespaces` | Giới hạn namespace Spinnaker quản lý |
| `--omit-namespaces` | Namespace bỏ qua khi cache |
| `--only-spinnaker-managed true` | Chỉ deploy vào namespace có annotation `spinnaker.io/managed: "true"` (an toàn) |
| `--check-permissions true` | Kiểm tra quyền ServiceAccount trước khi add |

### 5.3. AWS Provider

**Chuẩn bị IAM:** tạo role `SpinnakerManagedRole` (policy `PowerUserAccess` hoặc policy giới hạn EC2/AutoScaling/ELB) cho phép Spinnaker assume role.

```bash
# Thêm AWS account (dùng assume role thay vì access key dài hạn)
hal config provider aws account add my-aws-prod \
  --account-id 123456789012 \
  --assume-role role/SpinnakerManagedRole \
  --regions ap-southeast-1,ap-southeast-2 \
  --default-key-pair my-ec2-keypair

hal config provider aws enable
```

Kiểm tra account:

```bash
hal config provider aws account list
curl "http://localhost:8084/credentials/my-aws-prod" | python3 -m json.tool
```

### 5.4. GCP Provider

**Chuẩn bị:** tạo Service Account với các role Compute Admin + Storage Admin, tải JSON key.

```bash
hal config provider google account add my-gcp-prod \
  --project my-gcp-project-id \
  --json-path /home/spinnaker/gcp-spinnaker-key.json \
  --image-projects ubuntu-os-cloud,debian-cloud \
  --regions asia-southeast1,asia-northeast1

hal config provider google enable
```

### 5.5. Cấu hình CI — Jenkins (để trigger pipeline từ build)

```bash
hal config ci jenkins master add my-jenkins \
  --address http://jenkins.internal:8080 \
  --username spinnaker \
  --password-file /home/spinnaker/.jenkins-password.txt

hal config ci jenkins enable
```

### 5.6. Áp dụng và kiểm tra toàn bộ

```bash
# Áp dụng mọi thay đổi provider
sudo hal deploy apply

# Kiểm tra danh sách credentials Spinnaker nhìn thấy
curl -s http://localhost:8084/credentials | python3 -m json.tool
```

---
## 6. Applications

### 6.1. Tạo Application

Application là "thư mục" logic chứa mọi thứ của một service. Tạo qua UI (Deck → Applications → Create) hoặc API:

```bash
curl -X POST http://localhost:8084/applications \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "checkout-api",
    "email": "team-checkout@example.com",
    "description": "Checkout API microservice",
    "cloudProviders": ["kubernetes", "aws"],
    "instancePort": 8080,
    "dataSources": {
      "enabled": ["serverGroups", "loadBalancers", "clusters", "pipelines", "executions"]
    }
  }'
```

| Trường | Ý nghĩa |
|---|---|
| `cloudProviders` | Provider mà app này deploy lên |
| `dataSources.enabled` | Các tab hiển thị trong UI (serverGroups, loadBalancers, clusters, pipelines...) |

### 6.2. Cluster, Server Group, Load Balancer

Tạo Load Balancer cho Kubernetes qua UI: Application → Load Balancers → Create → chọn account/namespace → định nghĩa Service manifest. Ví dụ:

```yaml
# Dán vào ô "YAML" khi tạo Load Balancer (K8s provider V2)
apiVersion: v1
kind: Service
metadata:
  name: checkout-api
  namespace: production
  annotations:
    spinnaker.io/managed: "true"
spec:
  type: LoadBalancer
  selector:
    app: checkout-api
  ports:
    - port: 80
      targetPort: 8080
```

### 6.3. Tạo Server Group thủ công (AWS)

Application → Server Groups → Create: chọn account/region, AMI đã bake, instance type (ví dụ `t3.medium`), capacity Min/Max/Desired (ví dụ `2/6/3`), load balancer, health check (`ELB`), và strategy (mặc định Red/Black — xem mục 8).

---

## 7. Pipelines: Stages và Triggers

Pipeline là xương sống của Spinnaker. Một pipeline gồm **stages** chạy tuần tự/song song, được kích hoạt bởi **triggers**.

### 7.1. Tạo pipeline

UI: Application → Pipelines → Create → đặt tên (ví dụ `deploy-to-prod`). Có thể tạo pipeline bằng API (`POST /pipelines`) với JSON như ví dụ ở mục 7.3.

### 7.2. Các loại Stage phổ biến

| Stage | Công dụng |
|---|---|
| **Deploy** | Deploy server group mới (manifest/image mới) |
| **Bake** | Bake machine image từ package (Rosco + Packer) |
| **Resize Server Group** | Thay đổi capacity (scale up/down) |
| **Disable Server Group** | Gỡ khỏi LB nhưng giữ lại (để rollback) |
| **Destroy Server Group** | Xóa hẳn server group cũ |
| **Manual Judgment** | Dừng chờ người duyệt (xem mục 9) |
| **Canary Analysis** | Chạy phân tích canary tự động (Kayenta, xem mục 10) |
| **Jenkins** | Chạy job Jenkins (build/test) |
| **Run Job (K8s)** | Chạy Kubernetes Job (test, migration) |
| **Webhook** | Gọi HTTP endpoint ngoài (pre/post deploy hook) |
| **Evaluate Variables** | Tính toán biến SpEL dùng cho stage sau |
| **Check Preconditions** | Kiểm tra điều kiện (expression) trước khi chạy tiếp |

### 7.3. Ví dụ pipeline JSON hoàn chỉnh (K8s: deploy staging → judgment → deploy prod)

```json
{
  "name": "deploy-to-prod",
  "application": "checkout-api",
  "triggers": [
    {
      "type": "docker",
      "enabled": true,
      "account": "my-docker-hub",
      "organization": "myorg",
      "repository": "myorg/checkout-api",
      "tag": "^v\\d+\\.\\d+\\.\\d+$"
    }
  ],
  "stages": [
    {
      "name": "Deploy to staging",
      "type": "deploy",
      "refId": "1",
      "requisiteStageRefIds": [],
      "account": "my-k8s-prod",
      "cloudProvider": "kubernetes",
      "clusters": [
        {
          "account": "my-k8s-prod",
          "application": "checkout-api",
          "stack": "staging",
          "namespace": "staging",
          "strategy": "redblack",
          "manifests": [
            {
              "apiVersion": "apps/v1",
              "kind": "Deployment",
              "metadata": { "name": "checkout-api-staging", "namespace": "staging" },
              "spec": {
                "replicas": 2,
                "selector": { "matchLabels": { "app": "checkout-api" } },
                "template": {
                  "metadata": { "labels": { "app": "checkout-api" } },
                  "spec": {
                    "containers": [
                      { "name": "api", "image": "myorg/checkout-api:${trigger['tag']}", "ports": [{ "containerPort": 8080 }] }
                    ]
                  }
                }
              }
            }
          ]
        }
      ]
    },
    {
      "name": "Approve production deploy",
      "type": "manualJudgment",
      "refId": "2",
      "requisiteStageRefIds": ["1"],
      "instructions": "Kiểm tra staging OK rồi mới duyệt deploy production.",
      "judgmentInputs": [
        { "value": "deploy", "label": "Deploy production" },
        { "value": "rollback", "label": "Hủy, rollback staging" }
      ]
    },
    {
      "name": "Deploy to production",
      "type": "deploy",
      "refId": "3",
      "requisiteStageRefIds": ["2"],
      "stageEnabled": {
        "type": "expression",
        "expression": "${ #judgment('Approve production deploy') == 'deploy' }"
      },
      "account": "my-k8s-prod",
      "cloudProvider": "kubernetes",
      "clusters": [
        {
          "account": "my-k8s-prod",
          "application": "checkout-api",
          "stack": "prod",
          "namespace": "production",
          "strategy": "redblack",
          "manifests": [
            {
              "apiVersion": "apps/v1",
              "kind": "Deployment",
              "metadata": { "name": "checkout-api", "namespace": "production" },
              "spec": {
                "replicas": 4,
                "selector": { "matchLabels": { "app": "checkout-api" } },
                "template": {
                  "metadata": { "labels": { "app": "checkout-api" } },
                  "spec": {
                    "containers": [
                      { "name": "api", "image": "myorg/checkout-api:${trigger['tag']}", "ports": [{ "containerPort": 8080 }] }
                    ]
                  }
                }
              }
            }
          ]
        }
      ]
    }
  ],
  "limitConcurrent": true,
  "keepWaitingPipelines": false
}
```

### 7.4. Các loại Trigger

| Trigger | Cấu hình | Ví dụ sử dụng |
|---|---|---|
| **Docker** | account, repository, tag regex | Auto deploy khi có image mới |
| **Jenkins** | master, job, property file | Deploy sau khi build xong |
| **Git** | repo, branch, secret | Pipeline-as-code (Dinghy) hoặc trigger khi push |
| **Webhook** | source, payload constraints | Hệ thống ngoài kích hoạt deploy |
| **Cron** | expression | Deploy định kỳ (ví dụ nightly) |
| **Pipeline** | application, pipeline, status | Chuỗi pipeline nối tiếp nhau |

**Ví dụ các trigger trong JSON:**

```json
"triggers": [
  {
    "type": "jenkins",
    "enabled": true,
    "master": "my-jenkins",
    "job": "checkout-api/build",
    "propertyFile": "build.properties"
  },
  {
    "type": "webhook",
    "enabled": true,
    "source": "github-actions",
    "payloadConstraints": {
      "repository": "myorg/checkout-api",
      "action": "published"
    }
  },
  {
    "type": "cron",
    "enabled": true,
    "cronExpression": "0 2 * * *",
    "timezone": "Asia/Ho_Chi_Minh"
  }
]
```

### 7.5. Biểu thức SpEL trong pipeline

Spinnaker dùng Spring Expression Language (SpEL) với các helper:

```
${trigger['tag']}                          # tag docker từ trigger
${trigger['buildNumber']}                   # số build Jenkins
${execution['trigger']['user']}             # người chạy pipeline
${#judgment('Approve production deploy')}   # kết quả manual judgment
${#stage('Deploy to staging')['context']['deploy.account']}  # output stage trước
${#alphanum(8)}                             # chuỗi ngẫu nhiên 8 ký tự
${ 2 * 3 }                                  # biểu thức số học
```

---
## 8. Deployment Strategies chi tiết

Chiến lược deploy quyết định cách bản mới thay thế bản cũ. Chọn sai chiến lược = downtime hoặc rủi ro không rollback được.

### 8.1. Tổng quan so sánh

| Chiến lược | Cách hoạt động | Downtime | Rollback | Provider hỗ trợ | Khi nào dùng |
|---|---|---|---|---|---|
| **Red/Black** (Blue/Green) | Deploy bản mới (green) song song, chuyển traffic, tắt bản cũ (blue) | Không | Rất nhanh (bật lại bản cũ) | AWS, GCP, Azure, K8s | **Mặc định khuyến nghị** cho stateless service |
| **Rolling Red/Black** | Thay thế từng instance của bản cũ bằng bản mới theo đợt | Không | Nhanh | AWS, GCP | Khi không đủ capacity chạy song song 2 bản |
| **Canary** | Deploy bản mới với capacity nhỏ, tăng dần theo stage | Không | Nhanh | AWS, GCP, K8s | Muốn kiểm tra bản mới trên % traffic nhỏ trước |
| **Highlander** | Chỉ giữ 1 server group duy nhất ("there can be only one"), xóa bản cũ ngay | Không | Chậm (phải deploy lại) | K8s V2 | Job/cron, hoặc môi trường dev |
| **None** | Deploy bản mới, không động đến bản cũ | Không | Thủ công | Tất cả | Trường hợp đặc biệt, tự quản lý |
| **Rolling Push** (K8s) | Dùng native rolling update của Deployment | Không | `kubectl rollout undo` | K8s V2 | Khi muốn K8s tự quản lý rollout |

### 8.2. Red/Black (Blue/Green) — chi tiết

Luồng: tạo server group mới (green) → health check pass → attach LB → **disable** bản cũ (detach khỏi LB, giữ lại) → sau `scaleDownDelaySeconds` thì **destroy** (tùy cấu hình).

```json
{
  "name": "Deploy",
  "type": "deploy",
  "account": "my-aws-prod",
  "cloudProvider": "aws",
  "clusters": [
    {
      "application": "checkout-api",
      "stack": "prod",
      "strategy": "redblack",
      "maxRemainingAsgs": 2,
      "scaleDownDelaySeconds": 600,
      "delayBeforeDisableSeconds": 0,
      "delayBeforeScaleDownSeconds": 0,
      "rollback": {
        "onFailure": true
      },
      "capacity": { "min": 2, "max": 6, "desired": 3 },
      "image": "checkout-api-20261010",
      "instanceType": "t3.medium"
    }
  ]
}
```

| Tham số | Mặc định | Ý nghĩa |
|---|---|---|
| `maxRemainingAsgs` | 2 | Số server group cũ tối đa được giữ lại |
| `scaleDownDelaySeconds` | 30 | Đợi bao lâu trước khi scale down bản cũ |
| `delayBeforeDisableSeconds` | 0 | Đợi trước khi disable bản cũ (cho warmup) |
| `rollback.onFailure` | false | Tự động rollback nếu deploy fail → **nên bật true** |

Trên **Kubernetes V2**, Red/Black tạo ReplicaSet mới, scale bản mới lên trước rồi mới scale bản cũ về 0 (cùng tham số `strategy: redblack`).

### 8.3. Rolling Red/Black — chi tiết (AWS/GCP)

Thay thế instance theo đợt: launch instance mới → terminate instance cũ, lặp lại cho đến hết. Phù hợp khi không đủ quota/capacity để chạy 2 bản full song song.

```json
{
  "name": "Rolling deploy",
  "type": "deploy",
  "account": "my-aws-prod",
  "cloudProvider": "aws",
  "clusters": [
    {
      "application": "checkout-api",
      "stack": "prod",
      "strategy": "rollingredblack",
      "maxServerGroupSize": 4,
      "maxRemainingAsgs": 2,
      "scaleDownDelaySeconds": 300,
      "delayBeforeDisableSeconds": 60,
      "targetPercent": 100,
      "launchOrder": "oldestFirst"
    }
  ]
}
```

| Tham số | Ý nghĩa |
|---|---|
| `maxServerGroupSize` | Số instance tối đa của bản mới trong lúc rolling (giới hạn capacity tạm thời) |
| `targetPercent` | % instance của bản cũ cần thay thế (100 = thay hết) |
| `launchOrder` | `oldestFirst` hoặc `newestFirst`: thứ tự terminate instance cũ |

### 8.4. Canary Strategy — chi tiết

Canary deploy bản mới với capacity nhỏ, sau đó **tăng dần** qua nhiều stage. Khác với Canary Analysis (Kayenta, mục 10) — đây là chiến lược deploy thuần túy, thường kết hợp với Kayenta để tự động đánh giá.

Pipeline canary điển hình (AWS):

```
Stage 1: Deploy canary (capacity: 1 instance, ~5% traffic)
Stage 2: Canary Analysis (Kayenta so sánh metrics 30 phút)
Stage 3: Manual Judgment (tùy chọn)
Stage 4: Deploy remaining (scale canary lên full, disable bản cũ)
```

```json
{
  "stages": [
    {
      "name": "Deploy canary",
      "type": "deploy",
      "refId": "1",
      "requisiteStageRefIds": [],
      "account": "my-aws-prod",
      "cloudProvider": "aws",
      "clusters": [
        {
          "application": "checkout-api",
          "stack": "canary",
          "freeFormDetails": "canary",
          "strategy": "redblack",
          "capacity": { "min": 1, "max": 1, "desired": 1 },
          "image": "checkout-api-20261010"
        }
      ]
    },
    {
      "name": "Canary analysis",
      "type": "canary",
      "refId": "2",
      "requisiteStageRefIds": ["1"],
      "canaryConfig": { "id": "abc123-canary-config-id" },
      "baseline": {
        "account": "my-aws-prod",
        "cloudProvider": "aws",
        "cluster": "checkout-api-prod"
      },
      "canary": {
        "account": "my-aws-prod",
        "cloudProvider": "aws",
        "cluster": "checkout-api-canary"
      }
    }
  ]
}
```

Sau khi canary PASS, nối thêm stage **Resize Server Group** (`scale_exact`, target `current_asg_dynamic`) để promote canary lên full capacity, rồi **Disable Server Group** bản cũ.

Trên **Kubernetes V2** có thể dùng `"strategy": "canary"` với các bước `setWeight`/`pause` — cần service mesh (Istio/Linkerd) hoặc ingress hỗ trợ traffic splitting.

### 8.5. Highlander — chi tiết (Kubernetes V2)

"There can be only one" — mỗi cluster chỉ tồn tại **duy nhất 1** server group. Deploy bản mới xong, bản cũ bị **destroy ngay lập tức**.

```json
{
  "name": "Deploy batch job",
  "type": "deploy",
  "account": "my-k8s-prod",
  "cloudProvider": "kubernetes",
  "clusters": [
    {
      "application": "report-worker",
      "strategy": "highlander",
      "manifests": [
        {
          "apiVersion": "batch/v1",
          "kind": "CronJob",
          "metadata": { "name": "report-worker", "namespace": "jobs" },
          "spec": {
            "schedule": "0 1 * * *",
            "jobTemplate": {
              "spec": {
                "template": {
                  "spec": {
                    "containers": [
                      {
                        "name": "worker",
                        "image": "myorg/report-worker:${trigger['tag']}"
                      }
                    ],
                    "restartPolicy": "OnFailure"
                  }
                }
              }
            }
          }
        }
      ]
    }
  ]
}
```

| Ưu điểm | Nhược điểm |
|---|---|
| Không bao giờ rò rỉ resource cũ, gọn nhẹ | **Không rollback nhanh được** — bản cũ đã bị xóa |
| Phù hợp CronJob, Job một lần | Không dùng cho service quan trọng |

### 8.6. Chọn strategy nào?

Stateless + đủ capacity → **Red/Black** (mặc định); muốn test % traffic nhỏ → **Canary** (+ Kayenta); không đủ capacity chạy song song → **Rolling Red/Black**; CronJob/Job → **Highlander**; muốn K8s tự quản lý rollout → **Rolling Push**.

---
## 9. Manual Judgments & Approvals

Manual Judgment là "cổng kiểm soát bằng con người": pipeline dừng lại, gửi thông báo, chờ ai đó bấm **Continue** hoặc **Stop**.

### 9.1. Cấu hình cơ bản

```json
{
  "name": "Approve production deploy",
  "type": "manualJudgment",
  "refId": "3",
  "requisiteStageRefIds": ["2"],
  "instructions": "Đã kiểm tra staging:\n- Smoke test pass\n- Không có error spike\nDuyệt để deploy production.",
  "judgmentInputs": [
    { "value": "proceed", "label": "✅ Duyệt deploy" },
    { "value": "rollback", "label": "⛔ Hủy bỏ" }
  ],
  "notifications": [
    { "type": "slack", "address": "#deploys", "when": ["manualJudgment"] },
    { "type": "email", "address": "team-checkout@example.com", "when": ["manualJudgment"] }
  ],
  "failPipeline": true,
  "propagateAuthenticationContext": true
}
```

| Trường | Ý nghĩa |
|---|---|
| `instructions` | Hướng dẫn hiển thị cho người duyệt (hỗ trợ markdown) |
| `judgmentInputs` | Các nút lựa chọn (tối đa nên 2–3 nút) |
| `failPipeline` | `true`: nếu chọn Stop thì pipeline FAIL; `false`: pipeline dừng ở trạng thái STOPPED |
| `propagateAuthenticationContext` | Truyền quyền của người chạy pipeline sang các stage sau (quan trọng khi dùng Fiat) |

### 9.2. Rẽ nhánh theo kết quả judgment (SpEL)

Đặt `stageEnabled` dạng expression cho từng stage sau judgment:

- Stage "Deploy to production": `${ #judgment('Approve production deploy') == 'proceed' }`
- Stage "Rollback staging" (type `rollbackCluster`): `${ #judgment('Approve production deploy') == 'rollback' }`

### 9.3. Judgment nhiều cấp

Mẫu promotion `dev → staging → prod`: đặt judgment trước mỗi môi trường cao hơn; giới hạn người được bấm Continue bằng quyền WRITE trên application (Fiat) để chỉ lead mới duyệt được prod.

---

## 10. Canary Analysis với Kayenta

**Kayenta** là service phân tích canary tự động: so sánh metrics của bản **canary** (mới) với bản **baseline** (cũ) trong cùng khoảng thời gian, chấm điểm 0–100, quyết định pass/fail theo ngưỡng.

### 10.1. Cách Kayenta hoạt động

Orca (canary stage) → Kayenta lấy metrics từ Metrics Store (Prometheus/Datadog/...) → so sánh canary vs baseline theo từng metric → chấm điểm marginal/pass/fail → trả kết quả về Orca. Lưu ý: Kayenta **không thu thập metrics**, nó đọc từ hệ thống monitoring sẵn có đang scrape app.

### 10.2. Cấu hình Kayenta kết nối Prometheus

```bash
# Thêm Prometheus làm metrics store
hal config canary prometheus account add my-prometheus \
  --base-url http://prometheus.monitoring:9090 \
  --supported-types METRICS_STORE

# Nếu Prometheus cần basic auth
hal config canary prometheus account edit my-prometheus \
  --username prom-user \
  --password-file /home/spinnaker/.prom-password.txt

# Bật Kayenta
hal config canary enable

sudo hal deploy apply
```

### 10.3. Viết Canary Config

Canary Config định nghĩa: đo metric gì, ngưỡng pass/marginal ra sao, chạy trong bao lâu. Tạo qua UI (Delivery → Canary Configs) hoặc API:

```bash
curl -X POST http://localhost:8090/canaryConfig \
  -H 'Content-Type: application/json' \
  -d @canary-config.json
```

**Ví dụ `canary-config.json` đầy đủ (Prometheus):**

```json
{
  "name": "checkout-api-canary-config",
  "description": "Canary cho checkout-api: error rate, latency",
  "applications": ["checkout-api"],
  "metrics": [
    {
      "name": "Error Rate",
      "query": {
        "type": "prometheus",
        "metricName": "http_requests_errors_total",
        "labelBindings": {
          "app": "checkout-api",
          "scope": "${scope}"
        }
      },
      "groups": ["group1"],
      "analysisConfigurations": { "canary": { "direction": "decrease", "nanStrategy": "remove" } },
      "scopeName": "default"
    },
    {
      "name": "P99 Latency",
      "query": {
        "type": "prometheus",
        "metricName": "http_request_latency_p99_ms",
        "labelBindings": {
          "app": "checkout-api",
          "scope": "${scope}"
        }
      },
      "groups": ["group1"],
      "analysisConfigurations": { "canary": { "direction": "decrease", "nanStrategy": "remove" } },
      "scopeName": "default"
    }
  ],
  "classifier": {
    "groupWeights": { "group1": 100.0 },
    "thresholds": { "marginal": 75.0, "pass": 95.0 }
  },
  "judge": {
    "name": "NetflixACAJudge-v1.0",
    "judgeConfigurations": {}
  }
}
```

Giải thích:

| Phần | Ý nghĩa |
|---|---|
| `metrics[].query.labelBindings.scope = "${scope}"` | Kayenta tự thay bằng `canary`/`baseline` — **bắt buộc** để phân biệt 2 bản |
| `direction: "decrease"` | Metric càng thấp càng tốt (error, latency). Dùng `"increase"` cho throughput/success rate |
| `nanStrategy: "remove"` | Bỏ điểm dữ liệu NaN (tránh fail oan khi metric chưa có dữ liệu) |
| `groupWeights` | Trọng số từng nhóm metric (nên gom metric quan trọng vào nhóm weight cao) |
| `thresholds.pass: 95` | ≥95 điểm → PASS; 75–95 → MARGINAL (cần người quyết); <75 → FAIL |
| `judge: NetflixACAJudge-v1.0` | Thuật toán chấm điểm mặc định của Netflix |

### 10.4. Dùng Canary Config trong pipeline

```json
{
  "name": "Automated canary analysis",
  "type": "canary",
  "refId": "2",
  "requisiteStageRefIds": ["1"],
  "canaryConfig": {
    "id": "a1b2c3d4-canary-config-id",
    "name": "checkout-api-canary-config"
  },
  "baseline": {
    "account": "my-k8s-prod",
    "cloudProvider": "kubernetes",
    "cluster": "checkout-api-prod",
    "namespace": "production"
  },
  "canary": {
    "account": "my-k8s-prod",
    "cloudProvider": "kubernetes",
    "cluster": "checkout-api-canary",
    "namespace": "production"
  },
  "analysisConfig": {
    "beginCanaryAnalysisAfterMins": "5",
    "canaryAnalysisIntervalMins": "15",
    "lifetimeDuration": "PT2H",
    "lifetimeDurationMins": "120",
    "lookbackMins": "60",
    "metricsAccountName": "my-prometheus",
    "storageAccountName": "my-minio-storage"
  },
  "analysisType": "realTime"
}
```

| Tham số analysisConfig | Ý nghĩa |
|---|---|
| `beginCanaryAnalysisAfterMins` | Đợi 5 phút cho canary warmup trước khi đo |
| `canaryAnalysisIntervalMins` | Chấm điểm mỗi 15 phút |
| `lifetimeDurationMins` | Tổng thời gian canary: 120 phút |
| `lookbackMins` | So sánh với dữ liệu baseline 60 phút trước |
| `metricsAccountName` | Account Prometheus đã cấu hình ở 10.2 |

### 10.5. Đọc kết quả canary

Trong UI execution → stage canary → tab **Canary Report**: biểu đồ từng metric (canary vs baseline), điểm từng group, verdict PASS/MARGINAL/FAIL. Nếu FAIL, pipeline dừng hoặc rẽ nhánh rollback bằng expression kiểm tra `#stage(...)['status']`.

### 10.6. Lưu ý thực tế với Kayenta

- Metric phải có **đủ traffic** (canary traffic thấp → dữ liệu nhiễu, dễ fail oan): dùng `nanStrategy: remove` và warmup đủ dài.
- Label `scope` phải phân biệt được canary vs baseline trong Prometheus (thường dùng label `version`/`stack`).
- Bắt đầu với 3–5 metric cốt lõi (error rate, p99 latency, CPU, memory); ngưỡng pass 95 khá gắt cho lần đầu — có thể hạ xuống 80–90 rồi siết dần.

---
## 11. Artifacts & Expected Artifacts

**Artifact** là đơn vị versioned có thể deploy: Docker image, file trong Git/S3, Helm chart... Spinnaker dùng artifact để trigger pipeline và bind vào manifest (thay vì hardcode tag image).

### 11.1. Các loại artifact

| Type | Ví dụ `reference` |
|---|---|
| `docker/image` | `myorg/checkout-api:v1.2.3` |
| `helm/chart` | `stable/my-chart:1.0.0` |
| `s3/object` / `gcs/object` | `s3://bucket/config/app.yml` |
| `git/repo` | `https://github.com/myorg/config` |

### 11.2. Expected Artifacts — khai báo artifact pipeline cần

Trong pipeline, tab **Configuration → Expected Artifacts**, khai báo artifact trigger sẽ cung cấp. Stage deploy sau đó **bind** artifact này vào manifest bằng SpEL.

```json
{
  "name": "deploy-to-prod",
  "application": "checkout-api",
  "expectedArtifacts": [
    {
      "id": "checkout-api-image",
      "matchArtifact": {
        "type": "docker/image",
        "name": "myorg/checkout-api",
        "artifactAccount": "my-docker-hub"
      },
      "defaultArtifact": {
        "type": "docker/image",
        "name": "myorg/checkout-api",
        "reference": "myorg/checkout-api:latest",
        "artifactAccount": "my-docker-hub"
      },
      "usePriorArtifact": true,
      "useDefaultArtifact": true
    }
  ],
  "triggers": [
    {
      "type": "docker",
      "enabled": true,
      "account": "my-docker-hub",
      "organization": "myorg",
      "repository": "myorg/checkout-api",
      "tag": "^v\\d+\\.\\d+\\.\\d+$",
      "expectedArtifactIds": ["checkout-api-image"]
    }
  ]
}
```

| Trường | Ý nghĩa |
|---|---|
| `matchArtifact` | Điều kiện khớp artifact từ trigger (type + name) |
| `defaultArtifact` | Dùng khi chạy pipeline thủ công (không có trigger) |
| `usePriorArtifact` | Dùng artifact của lần chạy trước nếu trigger không cung cấp |
| `expectedArtifactIds` trong trigger | Liên kết trigger với expected artifact |

### 11.3. Bind artifact vào manifest (K8s)

Trong stage Deploy, đặt image là placeholder rồi bind artifact bằng SpEL:

```yaml
# Trong manifest của stage Deploy
image: "${#artifact('checkout-api-image')['reference']}"
```

```json
// Trong cluster config của stage Deploy, khai báo artifact được bind
{
  "strategy": "redblack",
  "manifestArtifactId": "checkout-api-image"
}
```

Lợi ích: cùng pipeline template dùng cho nhiều môi trường — đổi image chỉ cần đổi artifact, không sửa manifest.

---

## 12. Notifications: Slack, Email

Echo gửi notification khi pipeline/stage/manual judgment đạt trạng thái cấu hình.

### 12.1. Slack

```bash
# Tạo Slack app, lấy Bot Token (xoxb-...), mời bot vào channel #deploys
hal config notification slack edit \
  --bot-name spinnaker-bot \
  --token-file /home/spinnaker/.slack-token.txt

hal config notification slack enable
sudo hal deploy apply
```

Trong stage manual judgment (hoặc pipeline), cấu hình `notifications`:

```json
"notifications": [
  { "type": "slack", "address": "#deploys", "when": ["manualJudgment"] }
]
```

Các giá trị `when` hợp lệ: `pipeline.starting`, `pipeline.complete`, `pipeline.failed`, `pipeline.canceled`, `stage.starting`, `stage.complete`, `stage.failed`, `manualJudgment`.

### 12.2. Email

```bash
hal config notification email edit \
  --smtp \
  --host smtp.gmail.com \
  --port 587 \
  --from spinnaker@example.com \
  --username spinnaker@example.com \
  --password-file /home/spinnaker/.smtp-password.txt

hal config notification email enable
sudo hal deploy apply
```

```json
"notifications": [
  { "type": "email", "address": "team-checkout@example.com", "when": ["pipeline.failed", "manualJudgment"] }
]
```

---

## 13. RBAC với Fiat

**Fiat** là service phân quyền của Spinnaker. Mô hình:

- **Authentication** (xác thực — "bạn là ai"): do Gate xử lý (OAuth2/GitHub/Google/SAML/LDAP/x509).
- **Authorization** (phân quyền — "bạn được làm gì"): do Fiat xử lý, dựa trên **roles**.

### 13.1. Bật xác thực OAuth2 (ví dụ GitHub)

```bash
hal config security authn oauth2 edit \
  --client-id $GITHUB_CLIENT_ID \
  --client-secret \
  --provider github \
  --user-info-requirements "hd=example.com"

hal config security authn oauth2 enable
```

### 13.2. Bật Fiat với GitHub Teams làm role provider

```bash
# GitHub personal access token cần scope read:org
hal config security authz edit \
  --type github \
  --role-provider-type github \
  --base-url https://api.github.com \
  --access-token-file /home/spinnaker/.github-token.txt \
  --organization myorg

hal config security authz enable
sudo hal deploy apply
```

Các role provider khác: `--role-provider-type ldap|saml|google-groups|file`.

### 13.3. Gán quyền trên Application

```json
{
  "name": "checkout-api",
  "permissions": {
    "READ": ["team-checkout", "team-platform"],
    "WRITE": ["team-checkout"],
    "EXECUTE": ["team-checkout", "lead"]
  }
}
```

| Quyền | Được làm gì |
|---|---|
| `READ` | Xem application, pipeline, execution |
| `WRITE` | Tạo/sửa pipeline, deploy, tạo server group |
| `EXECUTE` | Chạy pipeline thủ công, bấm manual judgment |

> Nếu application **không** khai báo `permissions`, mặc định ai cũng có full quyền. Muốn "deny by default", bật:
> ```bash
> hal config security authz edit --type github   # hoặc provider đang dùng
> # và set trong gate-local.yml:
> # fiat:
> #   enabled: true
> ```

### 13.4. Phân quyền pipeline (restrict execution)

Trong pipeline JSON, giới hạn ai được chạy:

```json
{
  "name": "deploy-to-prod",
  "permissions": {
    "READ": ["team-checkout"],
    "WRITE": ["lead"],
    "EXECUTE": ["lead"]
  }
}
```

### 13.5. Sync roles

```bash
curl -X POST http://localhost:8084/fiat/sync   # đồng bộ roles thủ công
```

### 13.6. Ma trận phân quyền gợi ý

| Role | App dev | App prod | Pipeline prod |
|---|---|---|---|
| `team-checkout` (dev team) | READ+WRITE+EXECUTE | READ | EXECUTE (không WRITE) |
| `lead` | READ+WRITE+EXECUTE | READ+WRITE+EXECUTE | READ+WRITE+EXECUTE |
| `team-platform` / `auditor` | READ | READ | READ |

---
## 14. Scaling & Production Considerations

### 14.1. Tách Redis và Storage ra khỏi máy Halyard

Mặc định Halyard chạy Redis nội bộ — production phải dùng Redis riêng (HA: Redis Sentinel hoặc ElastiCache/Memorystore):

```bash
hal config storage redis edit --address redis://redis-ha.internal:6379
```

### 14.2. Dùng SQL thay Redis cho Orca execution repository

Ở quy mô lớn (hàng nghìn execution/ngày), Redis dễ đầy và mất dữ liệu khi restart. Dùng MySQL/PostgreSQL:

```bash
# Thêm dependency SQL cho Orca (qua hal config với custom profile)
mkdir -p ~/.hal/default/profiles
cat >> ~/.hal/default/profiles/orca-local.yml <<'EOF'
sql:
  enabled: true
  connectionPool:
    jdbcUrl: jdbc:mysql://mysql.internal:3306/orca?useSSL=false
    user: spinnaker
    password: ${MYSQL_PASSWORD}
    connectionTimeout: 5000
    maxPoolSize: 50
  migration:
    jdbcUrl: jdbc:mysql://mysql.internal:3306/orca?useSSL=false
    user: spinnaker
    password: ${MYSQL_PASSWORD}
executionRepository:
  sql:
    enabled: true
EOF
sudo hal deploy apply
```

### 14.3. Scale Clouddriver (service nặng nhất)

Clouddriver cache toàn bộ hạ tầng cloud — khi thêm nhiều account/region, cần tăng tài nguyên. Với deploy `distributed` trên K8s, scale replica:

```bash
kubectl -n spinnaker scale deploy spin-clouddriver --replicas=3
kubectl -n spinnaker set resources deploy spin-clouddriver \
  --limits=memory=4Gi,cpu=2000m --requests=memory=2Gi,cpu=1000m
```

### 14.4. Monitoring Spinnaker

Mỗi service expose `/metrics` (Prometheus) và `/health`:

```bash
# Bật monitoring
hal config metricstores prometheus edit --enabled true --url http://prometheus.monitoring:9090
```

Dashboard Grafana gợi ý theo dõi:

| Metric | Ngưỡng cảnh báo |
|---|---|
| `orca_pipeline_executions_failed_total` tăng đột biến | Pipeline fail hàng loạt |
| `clouddriver_cache_*_duration_seconds` > 300s | Cache chậm, cần scale Clouddriver |
| JVM heap / `redis_connected_clients` | Sắp OOM / Redis quá tải |

### 14.5. Backup & Disaster Recovery

```bash
# Backup toàn bộ config Halyard (chứa mọi hal config)
tar -czf hal-backup-$(date +%F).tar.gz ~/.hal

# Front50 data nằm trên S3/GCS — bật versioning cho bucket
aws s3api put-bucket-versioning --bucket spinnaker-storage \
  --versioning-configuration Status=Enabled

# Khôi phục: cài Halyard mới → giải nén ~/.hal → hal deploy apply
```

### 14.6. Checklist production

- [ ] Redis HA riêng, không dùng Redis nội bộ của Halyard
- [ ] Storage (S3/GCS) bật versioning + lifecycle
- [ ] Orca dùng SQL repository
- [ ] Clouddriver đủ CPU/RAM, cache interval hợp lý
- [ ] Bật authn (OAuth2/SAML) + Fiat RBAC, không để anonymous
- [ ] TLS cho Deck/Gate (qua ingress/LB ngoài)
- [ ] `--only-spinnaker-managed true` cho K8s accounts
- [ ] Giới hạn `--namespaces` / `--regions` đúng nhu cầu
- [ ] Backup `~/.hal` định kỳ + test restore
- [ ] Monitoring + alerting cho tất cả service
- [ ] Pin phiên bản Spinnaker (`hal config version edit`), nâng cấp theo lịch

---

## 15. Troubleshooting

| Triệu chứng | Nguyên nhân thường gặp | Cách fix |
|---|---|---|
| `hal deploy apply` treo ở "Waiting for service to start" | Thiếu RAM; service OOM | Kiểm tra `dmesg`/log `/var/log/spinnaker/<service>/`; tăng RAM hoặc giảm service không dùng |
| Deck trắng trang / không load | Gate chưa chạy; sai `override-base-url` | `curl localhost:8084/health`; kiểm tra `hal config security ui/api` khớp domain |
| Không thấy account trong UI | Chưa `hal deploy apply` sau khi add account | Chạy `sudo hal deploy apply`; kiểm tra `curl localhost:8084/credentials` |
| K8s account báo `permission denied` | ServiceAccount thiếu quyền | Kiểm tra ClusterRoleBinding; chạy với `--check-permissions true` để xem thiếu quyền gì |
| K8s deploy báo namespace not managed | Thiếu annotation | `kubectl annotate ns <ns> spinnaker.io/managed="true"` (khi bật `onlySpinnakerManaged`) |
| Docker trigger không kích hoạt | Igor chưa thấy image mới; sai regex tag | Kiểm tra Igor log; test regex; đảm bảo `--repositories` chứa repo đó |
| Pipeline kẹt ở "RUNNING" mãi | Orca mất kết nối Redis; task treo | Restart Orca; kiểm tra Redis `redis-cli ping`; xem execution trong Redis |
| Canary stage fail ngay lập tức | Sai `metricsAccountName`; Prometheus không có dữ liệu | Test query Prometheus thủ công với label `scope`; kiểm tra Kayenta log `/var/log/spinnaker/kayenta/` |
| Kayenta chấm điểm thấp oan | Traffic canary quá ít; warmup ngắn | Tăng `beginCanaryAnalysisAfterMins`; dùng `nanStrategy: remove`; tăng traffic canary |
| Bake fail (Rosco) | Packer template lỗi; thiếu quyền tạo AMI | Xem log Rosco; kiểm tra IAM quyền `ec2:CreateImage`; test Packer template thủ công |
| Fiat 403 dù đã login | Roles chưa sync; sai role provider | `curl -X POST localhost:8084/fiat/sync`; kiểm tra token GitHub scope `read:org` |
| Clouddriver cache chậm/miss resource | Quá nhiều region/namespace; cache thread ít | Giảm scope `--regions`/`--namespaces`; tăng resource; xem metric cache duration |

### Lệnh debug hữu ích

```bash
# Log từng service (localdebian)
tail -f /var/log/spinnaker/orca/orca.log
tail -f /var/log/spinnaker/clouddriver/clouddriver.log
tail -f /var/log/spinnaker/gate/gate.log
tail -f /var/log/spinnaker/echo/echo.log
tail -f /var/log/spinnaker/kayenta/kayenta.log
tail -f /var/log/spinnaker/fiat/fiat.log

# Trên Kubernetes (distributed)
kubectl -n spinnaker logs -l app=spin,cluster=spin-orca --tail=200 -f
kubectl -n spinnaker get pods   # xem pod nào CrashLoopBackOff
```

---

## 16. Best Practices Checklist

### Pipeline & strategy
- [ ] Mọi deploy production qua pipeline (không deploy tay, trừ khẩn cấp); có manual judgment trước prod
- [ ] `rollback.onFailure: true` (redblack), `limitConcurrent: true`, `keepWaitingPipelines: false`
- [ ] Dùng expected artifacts thay vì hardcode image tag; tách pipeline staging → promote prod
- [ ] Stateless → Red/Black; service quan trọng → Canary + Kayenta (≥3 metric); migration DB chạy Run Job riêng trước; không dùng Highlander cho service có traffic

### Bảo mật & vận hành
- [ ] Bật authn + Fiat (deny-by-default); secret qua `--*-file`, không paste vào lệnh
- [ ] K8s: `onlySpinnakerManaged=true` + giới hạn namespace; AWS: assume role thay vì access key dài hạn
- [ ] Pin version, nâng cấp ở staging trước; backup `~/.hal` + versioning bucket Front50
- [ ] Monitoring đầy đủ (mục 14.4) + alert pipeline fail; giữ `maxRemainingAsgs` nhỏ (ví dụ 2)

---

## 17. FAQ

**Hỏi: Spinnaker khác Jenkins thế nào? Có thay thế Jenkins không?**
Đáp: Không thay thế. Jenkins là CI (build/test/đóng gói artifact); Spinnaker là CD (nhận artifact → deploy đa cloud). Mô hình chuẩn: Jenkins build → trigger Spinnaker deploy.

**Hỏi: Có cần bake AMI không, hay dùng Docker là đủ?**
Đáp: Nếu deploy lên Kubernetes thì Docker image là đủ, không cần bake. Bake AMI chỉ cần cho AWS EC2/GCP VM theo mô hình immutable infrastructure truyền thống của Netflix.

**Hỏi: Spinnaker có tốn tài nguyên không?**
Đáp: Có. Full install ~10 service JVM, cần tối thiểu 4GB RAM cho lab, 8–16GB cho production nhỏ. Clouddriver là service ngốn nhất khi nhiều account.

**Hỏi: Rollback trong Spinnaker hoạt động thế nào?**
Đáp: Với Red/Black, bản cũ chỉ bị disable (không xóa) → rollback = enable lại bản cũ + disable bản mới, làm trong vài giây qua UI (Server Groups → chọn bản cũ → Rollback) hoặc stage `rollbackCluster` trong pipeline.

**Hỏi: Halyard và Spinnaker Operator khác nhau thế nào?**
Đáp: Halyard là CLI quản trị truyền thống (cài service trực tiếp hoặc render manifest). Operator là Kubernetes-native (CRD `SpinnakerService`), phù hợp GitOps. Dự án mới nên cân nhắc Operator.

**Hỏi: Nâng cấp Spinnaker có nguy hiểm không?**
Đáp: Dùng `hal config version edit --version <mới>` + `hal deploy apply`; Halyard hỗ trợ `hal deploy rollback` nếu lỗi. Luôn đọc release notes (có breaking change về provider/schema giữa các bản minor) và test ở staging trước.

---

## 18. Cheat Sheet

### Halyard — vòng đời

```bash
hal version list                          # liệt kê bản Spinnaker
hal config version edit --version 1.33.3  # pin version
sudo hal deploy apply                     # deploy/áp dụng config
hal deploy diff                           # diff config vs bản đang chạy
hal deploy rollback                       # rollback bản deploy trước
```

### Provider & account

```bash
# Docker registry
hal config provider docker-registry account add my-reg --address index.docker.io \
  --repositories myorg/api --username u --password-file pw.txt
hal config provider docker-registry enable

# Kubernetes
hal config provider kubernetes account add my-k8s --provider-version v2 \
  --kubeconfig-file ~/.kube/config --docker-registries my-reg \
  --only-spinnaker-managed true
hal config provider kubernetes enable

# AWS
hal config provider aws account add my-aws --account-id 123456789012 \
  --assume-role role/SpinnakerManagedRole --regions ap-southeast-1
hal config provider aws enable

# GCP
hal config provider google account add my-gcp --project my-proj \
  --json-path /path/key.json
hal config provider google enable
```

### Storage / CI / Notification / Canary / Auth

```bash
# Storage
hal config storage s3 edit --bucket b --region ap-southeast-1 \
  --access-key-id $AK --secret-access-key
hal config storage edit --type s3

# Jenkins
hal config ci jenkins master add my-jenkins --address http://jenkins:8080 \
  --username u --password-file pw.txt
hal config ci jenkins enable

# Slack / Email
hal config notification slack edit --bot-name spinnaker-bot --token-file tok.txt
hal config notification slack enable
hal config notification email edit --smtp --host smtp.gmail.com --port 587 \
  --from a@b.c --username a@b.c --password-file pw.txt
hal config notification email enable

# Kayenta + Prometheus
hal config canary prometheus account add my-prom \
  --base-url http://prometheus:9090 --supported-types METRICS_STORE
hal config canary enable

# Fiat RBAC (GitHub Teams)
hal config security authz edit --type github --role-provider-type github \
  --access-token-file tok.txt --organization myorg
hal config security authz enable
```

### API nhanh

```bash
G=http://localhost:8084
curl $G/health
curl $G/credentials
curl $G/applications
curl -X POST $G/pipelines -H 'Content-Type: application/json' -d @pipeline.json
curl "$G/applications/myapp/executions?statuses=RUNNING"
```

### SpEL hay dùng

```
${trigger['tag']}                         # docker tag
${#judgment('Ten stage')}                 # kết quả manual judgment
${#artifact('artifact-id')['reference']}  # reference của artifact
```

### Port mặc định các service

| Service | Port | Service | Port |
|---|---|---|---|
| Deck (UI) | 9000 | Gate (API) | 8084 |
| Orca | 8083 | Clouddriver | 7002 |
| Front50 | 8080 | Echo | 8089 |
| Igor | 8088 | Rosco | 8087 |
| Kayenta | 8090 | Fiat | 7003 |
| Dinghy | 8081 | Redis | 6379 |

---

*Hết tài liệu. Chúc bạn deploy an toàn.*
