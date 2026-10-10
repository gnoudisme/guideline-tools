# Hướng Dẫn Toàn Diện Kubernetes (K8s) ☸️

> Tài liệu tiếng Việt chi tiết về Kubernetes — từ khái niệm đến deploy thực tế.

---

## Mục Lục

1. [Kubernetes Là Gì?](#1-kubernetes-là-gì)
2. [Kiến Trúc Kubernetes](#2-kiến-trúc-kubernetes)
3. [Các Khái Niệm Cốt Lõi](#3-các-khái-niệm-cốt-lõi)
4. [Cài Đặt kubectl](#4-cài-đặt-kubectl)
5. [Minikube — K8s Trên Máy Local](#5-minikube--k8s-trên-máy-local)
6. [Lệnh kubectl Cơ Bản](#6-lệnh-kubectl-cơ-bản)
7. [Viết YAML Manifest](#7-viết-yaml-manifest)
8. [Pod — Đơn Vị Nhỏ Nhất](#8-pod--đơn-vị-nhỏ-nhất)
9. [Deployment — Quản Lý Pod](#9-deployment--quản-lý-pod)
10. [Service — Mạng Nội Bộ](#10-service--mạng-nội-bộ)
11. [Ingress — Truy Cập Từ Ngoài](#11-ingress--truy-cập-từ-ngoài)
12. [ConfigMap & Secret](#12-configmap--secret)
13. [Namespace — Chia Không Gian](#13-namespace--chia-không-gian)
14. [Scaling — Mở Rộng](#14-scaling--mở-rộng)
15. [Rolling Update & Rollback](#15-rolling-update--rollback)
16. [Volumes — Lưu Trữ Dữ Liệu](#16-volumes--lưu-trữ-dữ-liệu)
17. [Troubleshooting](#17-troubleshooting)
18. [Best Practices](#18-best-practices)
19. [Tài Liệu Tham Khảo](#19-tài-liệu-tham-khảo)

---

## 1. Kubernetes Là Gì?

### 1.1 Định Nghĩa

**Kubernetes** (viết tắt **K8s** — K + 8 chữ + s) là hệ thống điều phối container mã nguồn mở do Google tạo ra (2014), hiện do CNCF quản lý.

Nói đơn giản: nếu **Docker** là "đóng gói app vào container", thì **Kubernetes** là "quản lý hàng trăm container đó" — tự động deploy, mở rộng, và phục hồi khi lỗi.

### 1.2 Vấn Đề Kubernetes Giải Quyết

**Không có K8s:**
```
😰 Container chết → phải restart tay
😰 Traffic tăng → phải thêm server tay
😰 Deploy bản mới → downtime
😰 100 server → quản lý bằng tay = ác mộng
```

**Có K8s:**
```
✅ Container chết → tự restart
✅ Traffic tăng → tự thêm replica
✅ Deploy bản mới → zero downtime (rolling update)
✅ Khai báo 1 lần → K8s tự lo phần còn lại
```

### 1.3 So Sánh Với Docker Compose / Docker Swarm

| Tiêu chí | Docker Compose | Docker Swarm | Kubernetes |
|----------|----------------|--------------|------------|
| Quy mô | 1 máy | Vài máy | Hàng trăm máy |
| Tự phục hồi | ❌ | Cơ bản | ✅ Mạnh mẽ |
| Auto-scaling | ❌ | ❌ | ✅ |
| Rolling update | ❌ | Cơ bản | ✅ Nâng cao |
| Độ phức tạp | Dễ | Trung bình | Khó |
| Dùng khi nào | Dev local | Nhỏ | Production lớn |

### 1.4 Ai Dùng Kubernetes?

Google, Spotify, Airbnb, Tinder, Pinterest, và hàng nghìn công ty lớn. Nếu app của bạn cần chạy production với nhiều user → K8s là chuẩn công nghiệp.

---

## 2. Kiến Trúc Kubernetes

### 2.1 Tổng Quan

```
┌─────────────────────────────────────────────┐
│              CONTROL PLANE                   │
│  (bộ não - quyết định chạy gì ở đâu)        │
│  ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │ API      │ │Scheduler │ │ Controller   │  │
│  │ Server   │ │          │ │ Manager      │  │
│  └──────────┘ └──────────┘ └──────────────┘  │
│  ┌──────────┐                                │
│  │  etcd    │  (database lưu trạng thái)    │
│  └──────────┘                                │
└──────────────────┬──────────────────────────┘
                   │ API
        ┌──────────┼──────────┐
        ▼          ▼          ▼
   ┌─────────┐ ┌─────────┐ ┌─────────┐
   │ Worker  │ │ Worker  │ │ Worker  │
   │ Node 1  │ │ Node 2  │ │ Node 3  │
   │ ┌─────┐ │ │ ┌─────┐ │ │ ┌─────┐ │
   │ │ Pod │ │ │ │ Pod │ │ │ │ Pod │ │
   │ └─────┘ │ │ └─────┘ │ │ └─────┘ │
   │ kubelet │ │ kubelet │ │ kubelet │
   └─────────┘ └─────────┘ └─────────┘
```

### 2.2 Các Thành Phần

**Control Plane (bộ não):**
| Thành phần | Chức năng |
|------------|-----------|
| API Server | Cổng giao tiếp — mọi lệnh kubectl đều qua đây |
| etcd | Database lưu toàn bộ trạng thái cluster |
| Scheduler | Quyết định Pod chạy trên Node nào |
| Controller Manager | Đảm bảo trạng thái thực = trạng thái mong muốn |

**Worker Node (tay chân):**
| Thành phần | Chức năng |
|------------|-----------|
| kubelet | Agent trên mỗi node, quản lý Pod |
| kube-proxy | Quản lý mạng cho Pod |
| Container Runtime | Chạy container (containerd, CRI-O) |

### 2.3 Luồng Hoạt Động

```
1. Bạn: kubectl apply -f app.yaml
         ↓
2. API Server nhận yêu cầu, lưu vào etcd
         ↓
3. Scheduler chọn Node phù hợp
         ↓
4. kubelet trên Node đó tạo Pod
         ↓
5. Controller kiểm tra: đủ 3 replica chưa?
   Chưa → tạo thêm. Thừa → xóa bớt.
```

---

## 3. Các Khái Niệm Cốt Lõi

### 3.1 Bản Đồ Khái Niệm

```
Namespace (không gian tên)
  └── Deployment (quản lý Pod)
  │     └── ReplicaSet (đảm bảo số lượng Pod)
  │           └── Pod (đơn vị nhỏ nhất)
  │                 └── Container (app của bạn)
  │
  ├── Service (địa chỉ mạng ổn định)
  ├── Ingress (cổng vào từ internet)
  ├── ConfigMap (cấu hình dạng text)
  └── Secret (mật khẩu, token)
```

### 3.2 Tóm Tắt Nhanh

| Khái niệm | Là gì | Ví dụ thực tế |
|-----------|-------|---------------|
| **Pod** | 1+ container chạy cùng nhau | 1 Pod = app + sidecar log |
| **Deployment** | Quản lý Pod, update không downtime | Deploy web app v2 |
| **Service** | Địa chỉ IP ổn định cho Pod | `web-service:80` |
| **Ingress** | Route traffic từ internet | `app.com` → service A |
| **ConfigMap** | Biến cấu hình | `DB_HOST=db.local` |
| **Secret** | Dữ liệu nhạy cảm (mã hóa) | `DB_PASSWORD=***` |
| **Namespace** | Chia cluster thành vùng riêng | `dev`, `staging`, `prod` |
| **Volume** | Ổ đĩa gắn vào Pod | Lưu database |

---

## 4. Cài Đặt kubectl

`kubectl` là công cụ dòng lệnh để điều khiển Kubernetes.

### 4.1 Linux

```bash
# Tải binary mới nhất
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Cài đặt
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Kiểm tra
kubectl version --client
```

### 4.2 macOS

```bash
# Dùng Homebrew
brew install kubectl

# Kiểm tra
kubectl version --client
```

### 4.3 Windows

```powershell
# Dùng Chocolatey
choco install kubernetes-cli

# Hoặc tải trực tiếp
curl.exe -LO "https://dl.k8s.io/release/v1.28.0/bin/windows/amd64/kubectl.exe"
```

### 4.4 Cấu Hình Autocomplete (Khuyến Nghị)

```bash
# Bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc

# Zsh
echo 'source <(kubectl completion zsh)' >> ~/.zshrc
echo 'alias k=kubectl' >> ~/.zshrc
source ~/.zshrc
```

Từ giờ dùng `k` thay vì `kubectl` cho nhanh!

---

## 5. Minikube — K8s Trên Máy Local

### 5.1 Minikube Là Gì?

Minikube tạo cluster Kubernetes 1-node trên máy bạn — hoàn hảo để học và test.

### 5.2 Cài Đặt Minikube

```bash
# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# macOS
brew install minikube

# Windows
choco install minikube
```

### 5.3 Khởi Động Cluster

```bash
# Khởi động (mất 1-2 phút lần đầu)
minikube start

# Khởi động với nhiều tài nguyên hơn
minikube start --cpus=4 --memory=8192

# Kiểm tra trạng thái
minikube status

# Mở dashboard (giao diện web)
minikube dashboard
```

### 5.4 Các Lệnh Minikube Hữu Ích

```bash
minikube start          # khởi động cluster
minikube stop           # dừng cluster
minikube delete         # xóa cluster
minikube status         # xem trạng thái
minikube ip             # xem IP của cluster
minikube dashboard      # mở dashboard web
minikube ssh            # SSH vào node
minikube addons list    # xem addons có sẵn
minikube addons enable ingress  # bật ingress
```

### 5.5 Giải Pháp Thay Thế Minikube

| Công cụ | Đặc điểm |
|---------|----------|
| **kind** | Chạy K8s trong Docker, khởi động nhanh |
| **k3s** | K8s siêu nhẹ, tốt cho IoT/edge |
| **Docker Desktop** | Có sẵn K8s, bật trong settings |
| **k3d** | k3s trong Docker |

---

## 6. Lệnh kubectl Cơ Bản

### 6.1 Xem Thông Tin Cluster

```bash
# Thông tin cluster
kubectl cluster-info

# Danh sách nodes
kubectl get nodes

# Xem chi tiết 1 node
kubectl describe node minikube

# Xem tất cả resources
kubectl get all
```

### 6.2 Quản Lý Pod

```bash
# Liệt kê Pod
kubectl get pods

# Liệt kê kèm thêm thông tin (IP, node)
kubectl get pods -o wide

# Xem chi tiết 1 Pod
kubectl describe pod ten-pod

# Xem log của Pod
kubectl logs ten-pod

# Xem log realtime (follow)
kubectl logs -f ten-pod

# Xem log container cụ thể (nếu Pod có nhiều container)
kubectl logs ten-pod -c ten-container

# Chạy lệnh trong Pod
kubectl exec -it ten-pod -- /bin/bash

# Chạy lệnh 1 lần
kubectl exec ten-pod -- ls /app

# Xóa Pod
kubectl delete pod ten-pod
```

### 6.3 Lọc Theo Namespace

```bash
# Xem Pod trong namespace cụ thể
kubectl get pods -n kube-system

# Xem tất cả namespace
kubectl get pods -A
# (-A = --all-namespaces)
```

### 6.4 Format Output

```bash
# Dạng bảng (mặc định)
kubectl get pods

# Dạng YAML đầy đủ
kubectl get pod ten-pod -o yaml

# Dạng JSON
kubectl get pod ten-pod -o json

# Chỉ lấy tên
kubectl get pods -o name

# Tùy chỉnh cột
kubectl get pods -o custom-columns=NAME:.metadata.name,IP:.status.podIP

# Sắp xếp theo thời gian tạo
kubectl get pods --sort-by=.metadata.creationTimestamp
```

### 6.5 Label & Selector

```bash
# Lọc theo label
kubectl get pods -l app=web
kubectl get pods -l app=web,env=prod

# Thêm label
kubectl label pods ten-pod app=web

# Xóa label
kubectl label pods ten-pod app-
```

---

## 7. Viết YAML Manifest

### 7.1 YAML Là Gì?

Kubernetes dùng file **YAML** để khai báo "tôi muốn trạng thái như thế này". K8s sẽ tự động đạt trạng thái đó.

### 7.2 Cú Pháp YAML Cơ Bản

```yaml
# Comment bắt đầu bằng #
apiVersion: v1          # phiên bản API
kind: Pod               # loại resource
metadata:               # thông tin định danh
  name: my-pod          # tên (duy nhất trong namespace)
  labels:               # nhãn để lọc/tìm
    app: web
spec:                   # trạng thái mong muốn
  containers:
  - name: nginx
    image: nginx:1.25
```

**Quy tắc thụt đầu dòng:**
- Dùng **space**, KHÔNG dùng tab!
- Thụt đầu dòng = quan hệ cha-con
- `key: value` cho giá trị đơn
- `- item` cho danh sách

### 7.3 Áp Dụng File YAML

```bash
# Tạo/cập nhật resource từ file
kubectl apply -f app.yaml

# Áp dụng cả thư mục
kubectl apply -f ./k8s/

# Xem trước (không thực sự tạo)
kubectl apply -f app.yaml --dry-run=client

# Xóa resource trong file
kubectl delete -f app.yaml

# So sánh thay đổi
kubectl diff -f app.yaml
```

---

## 8. Pod — Đơn Vị Nhỏ Nhất

### 8.1 Pod Là Gì?

**Pod** là đơn vị triển khai nhỏ nhất trong K8s — chứa 1 hoặc nhiều container chia sẻ:
- Network (cùng IP, có thể gọi nhau qua `localhost`)
- Storage (cùng volumes)

> 💡 **Quy tắc:** 1 Pod = 1 chức năng chính. Đừng nhét nhiều app không liên quan vào 1 Pod!

### 8.2 Tạo Pod Đơn Giản

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:1.25
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f pod.yaml
kubectl get pods
```

### 8.3 Pod Với Biến Môi Trường

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    - name: DB_HOST
      value: "db.example.com"
    - name: DB_PORT
      value: "5432"
    - name: DEBUG
      value: "true"
```

### 8.4 Pod Với Resource Limit

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:1.0
    resources:
      requests:          # tối thiểu cần
        memory: "128Mi"
        cpu: "250m"      # 250 millicpu = 0.25 CPU
      limits:            # tối đa được dùng
        memory: "512Mi"
        cpu: "500m"
```

> 💡 **requests** = Scheduler dùng để chọn node. **limits** = K8s kill container nếu vượt quá.

### 8.5 Vòng Đời Pod

```
Pending → ContainerCreating → Running → Succeeded/Failed
                  ↓
              CrashLoopBackOff (lỗi, đang thử restart)
```

```bash
# Xem trạng thái Pod
kubectl get pods

# Nếu Pod bị CrashLoopBackOff, xem log:
kubectl logs ten-pod --previous
```

---

## 9. Deployment — Quản Lý Pod

### 9.1 Deployment Là Gì?

**Deployment** quản lý Pod giúp bạn:
- Đảm bảo luôn có N Pod chạy (tự tạo lại khi chết)
- Update version không downtime (rolling update)
- Rollback về bản cũ khi lỗi

> ⚠️ **Thực tế:** Hầu như không ai tạo Pod trực tiếp — luôn dùng Deployment!

### 9.2 Tạo Deployment

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web
spec:
  replicas: 3                    # muốn 3 Pod chạy
  selector:
    matchLabels:
      app: web                   # quản lý Pod có label này
  template:                      # mẫu Pod
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "128Mi"
            cpu: "250m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

```bash
kubectl apply -f deployment.yaml

# Kiểm tra
kubectl get deployments
kubectl get pods -l app=web
kubectl get replicasets
```

### 9.3 Các Lệnh Deployment

```bash
# Xem danh sách
kubectl get deployments

# Xem chi tiết
kubectl describe deployment web-app

# Xem lịch sử rollout
kubectl rollout history deployment web-app

# Trạng thái rollout
kubectl rollout status deployment web-app
```

---

## 10. Service — Mạng Nội Bộ

### 10.1 Vấn Đề Service Giải Quyết

Pod có IP thay đổi mỗi khi restart. **Service** cung cấp địa chỉ ổn định để truy cập nhóm Pod.

```
Pod 1 (IP: 10.0.0.1) ─┐
Pod 2 (IP: 10.0.0.2) ─┼─→ Service (IP: 10.96.0.10, ổn định)
Pod 3 (IP: 10.0.0.3) ─┘         ↓ load balance
                          truy cập qua tên: web-service
```

### 10.2 Các Loại Service

| Loại | Dùng khi nào |
|------|--------------|
| **ClusterIP** | Chỉ truy cập nội bộ cluster (mặc định) |
| **NodePort** | Truy cập từ ngoài qua `NodeIP:Port` |
| **LoadBalancer** | Cloud tự tạo load balancer (AWS/GCP/Azure) |
| **ExternalName** | Alias tới DNS bên ngoài |

### 10.3 Tạo Service ClusterIP

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP
  selector:
    app: web              # chọn Pod có label app=web
  ports:
  - port: 80              # port của Service
    targetPort: 80        # port của container
    protocol: TCP
```

```bash
kubectl apply -f service.yaml

# Test từ 1 Pod khác:
kubectl run test --image=busybox -it --rm -- wget -qO- http://web-service
```

### 10.4 Tạo Service NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  type: NodePort
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080       # port trên Node (30000-32767)
```

```bash
# Truy cập từ ngoài:
# http://<node-ip>:30080

# Với minikube:
minikube service web-nodeport --url
```

---

## 11. Ingress — Truy Cập Từ Ngoài

### 11.1 Ingress Là Gì?

**Ingress** route HTTP/HTTPS từ internet vào các Service — như reverse proxy.

```
Internet
   │
   │  app.com/      → web-service:80
   │  app.com/api   → api-service:8080
   ▼
┌─────────┐
│ Ingress │  (nginx, traefik...)
└─────────┘
```

### 11.2 Bật Ingress Controller (Minikube)

```bash
minikube addons enable ingress
```

### 11.3 Tạo Ingress

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
```

```bash
kubectl apply -f ingress.yaml

# Thêm vào /etc/hosts:
# <minikube-ip> myapp.local

# Truy cập: http://myapp.local
```

### 11.4 Ingress Với HTTPS (TLS)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress-tls
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-tls-secret  # chứa cert
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

---

## 12. ConfigMap & Secret

### 12.1 ConfigMap — Cấu Hình

Lưu cấu hình dạng key-value (không nhạy cảm).

**Tạo từ lệnh:**
```bash
# Từ giá trị trực tiếp
kubectl create configmap app-config \
  --from-literal=DB_HOST=db.local \
  --from-literal=DB_PORT=5432

# Từ file
kubectl create configmap app-config --from-file=config.properties

# Từ file env
kubectl create configmap app-config --from-env-file=.env
```

**Tạo từ YAML:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DB_HOST: "db.local"
  DB_PORT: "5432"
  config.properties: |
    debug=true
    timeout=30
```

**Dùng trong Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_HOST
    # Hoặc load tất cả:
    envFrom:
    - configMapRef:
        name: app-config
```

### 12.2 Secret — Dữ Liệu Nhạy Cảm

Lưu password, token (được mã hóa base64).

**Tạo từ lệnh:**
```bash
# Từ giá trị trực tiếp
kubectl create secret generic db-secret \
  --from-literal=password=matkhau123 \
  --from-literal=username=admin

# Từ file
kubectl create secret generic tls-secret \
  --from-file=tls.crt --from-file=tls.key
```

**Tạo từ YAML:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  # echo -n "matkhau123" | base64
  password: bWF0a2hhdTEyMw==
  # echo -n "admin" | base64
  username: YWRtaW4=
```

**Dùng trong Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:1.0
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

> ⚠️ **Lưu ý:** Secret chỉ mã hóa base64 (dễ giải mã!). Production nên dùng **Sealed Secrets** hoặc **Vault**.

---

## 13. Namespace — Chia Không Gian

### 13.1 Namespace Là Gì?

Chia cluster thành các vùng riêng biệt — như thư mục.

```
cluster
├── default       (mặc định)
├── kube-system   (hệ thống K8s)
├── dev           (môi trường dev)
├── staging       (môi trường staging)
└── prod          (môi trường production)
```

### 13.2 Quản Lý Namespace

```bash
# Liệt kê
kubectl get namespaces

# Tạo
kubectl create namespace dev

# Tạo từ YAML
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: staging
```

```bash
# Deploy vào namespace cụ thể
kubectl apply -f app.yaml -n dev

# Xem Pod trong namespace
kubectl get pods -n dev

# Set namespace mặc định
kubectl config set-context --current --namespace=dev

# Xóa namespace (xóa hết resource bên trong!)
kubectl delete namespace dev
```

---

## 14. Scaling — Mở Rộng

### 14.1 Scale Thủ Công

```bash
# Tăng lên 5 replica
kubectl scale deployment web-app --replicas=5

# Giảm xuống 2
kubectl scale deployment web-app --replicas=2

# Kiểm tra
kubectl get pods -l app=web
```

### 14.2 Horizontal Pod Autoscaler (Tự Động)

Tự động thêm/bớt Pod theo CPU/RAM.

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # scale khi CPU > 70%
```

```bash
kubectl apply -f hpa.yaml

# Xem trạng thái
kubectl get hpa
kubectl describe hpa web-hpa
```

> 💡 Deployment phải có `resources.requests` thì HPA mới hoạt động!

---

## 15. Rolling Update & Rollback

### 15.1 Rolling Update Là Gì?

Update từng Pod một — không downtime.

```
v1 v1 v1  →  v2 v1 v1  →  v2 v2 v1  →  v2 v2 v2
```

### 15.2 Thực Hiện Update

```bash
# Cách 1: sửa image trong YAML rồi apply
kubectl apply -f deployment-v2.yaml

# Cách 2: set image trực tiếp
kubectl set image deployment/web-app web=nginx:1.26

# Xem tiến trình
kubectl rollout status deployment web-app
```

### 15.3 Cấu Hình Rolling Update

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # tối đa tạo thêm 1 Pod mới
      maxUnavailable: 1  # tối đa 1 Pod bị down
  # ... template như bình thường
```

### 15.4 Rollback Khi Lỗi

```bash
# Xem lịch sử
kubectl rollout history deployment web-app

# Rollback về bản trước
kubectl rollout undo deployment web-app

# Rollback về bản cụ thể
kubectl rollout undo deployment web-app --to-revision=2

# Xem chi tiết 1 revision
kubectl rollout history deployment web-app --revision=3
```

---

## 16. Volumes — Lưu Trữ Dữ Liệu

### 16.1 Tại Sao Cần Volume?

Pod bị xóa → dữ liệu trong container **mất hết**. Volume giữ dữ liệu sống sót qua restart.

### 16.2 emptyDir (Tạm Thời)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:1.0
    volumeMounts:
    - mountPath: /data
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

> emptyDir: tồn tại khi Pod còn sống, mất khi Pod bị xóa.

### 16.3 hostPath (Dùng Ổ Cứng Node)

```yaml
volumes:
- name: data-volume
  hostPath:
    path: /data/myapp
    type: DirectoryOrCreate
```

### 16.4 PersistentVolumeClaim (Khuyến Nghị)

```yaml
# 1. Yêu cầu ổ đĩa
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
---
# 2. Gắn vào Pod
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  containers:
  - name: postgres
    image: postgres:15
    volumeMounts:
    - mountPath: /var/lib/postgresql/data
      name: db-storage
  volumes:
  - name: db-storage
    persistentVolumeClaim:
      claimName: db-pvc
```

---

## 17. Troubleshooting

### 17.1 Pod Không Chạy

```bash
# Bước 1: xem trạng thái
kubectl get pods

# Bước 2: xem chi tiết (Events ở cuối rất quan trọng!)
kubectl describe pod ten-pod

# Bước 3: xem log
kubectl logs ten-pod

# Log của lần chạy trước (khi bị crash)
kubectl logs ten-pod --previous
```

### 17.2 Các Lỗi Phổ Biến

| Trạng thái | Nguyên nhân | Cách fix |
|------------|-------------|----------|
| `ImagePullBackOff` | Không tải được image | Kiểm tra tên image, `imagePullSecrets` |
| `CrashLoopBackOff` | App crash liên tục | Xem log: `kubectl logs pod --previous` |
| `Pending` | Không đủ tài nguyên | Kiểm tra `kubectl describe`, thêm node/giảm requests |
| `ContainerCreating` | Đang tạo (lâu = lỗi) | Kiểm tra volume, network |
| `Error` | Container thoát với lỗi | Xem log |
| `OOMKilled` | Hết memory | Tăng `limits.memory` |

### 17.3 Debug Mạng

```bash
# Test kết nối tới Service từ Pod tạm
kubectl run test --image=busybox -it --rm -- wget -qO- http://web-service:80

# Xem DNS có resolve không
kubectl run test --image=busybox -it --rm -- nslookup web-service

# Xem endpoints của Service
kubectl get endpoints web-service

# Kiểm tra Service có chọn đúng Pod không
kubectl describe service web-service
```

### 17.4 Debug Deployment

```bash
# Xem rollout có bị kẹt không
kubectl rollout status deployment web-app

# Xem events
kubectl get events --sort-by=.metadata.creationTimestamp

# Xem events của namespace cụ thể
kubectl get events -n dev --sort-by=.metadata.creationTimestamp
```

### 17.5 Vào Pod Để Debug

```bash
# Mở shell trong Pod
kubectl exec -it ten-pod -- /bin/bash
# (nếu không có bash, thử /bin/sh)

# Chạy lệnh 1 lần
kubectl exec ten-pod -- env | sort
kubectl exec ten-pod -- cat /etc/hosts
```

---

## 18. Best Practices

### 18.1 Luôn Set Resource Limits
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "1000m"
```

### 18.2 Dùng Liveness & Readiness Probe
```yaml
containers:
- name: app
  image: my-app:1.0
  livenessProbe:          # kiểm tra app còn sống không
    httpGet:
      path: /healthz
      port: 8080
    initialDelaySeconds: 30
    periodSeconds: 10
  readinessProbe:         # kiểm tra app sẵn sàng nhận traffic chưa
    httpGet:
      path: /ready
      port: 8080
    initialDelaySeconds: 5
    periodSeconds: 5
```

### 18.3 Không Dùng `:latest`
```yaml
# ❌ Sai: không biết đang chạy bản nào
image: my-app:latest

# ✅ Đúng: rõ ràng version
image: my-app:1.2.3
```

### 18.4 Tách Môi Trường Bằng Namespace
```bash
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod
```

### 18.5 Dùng Secret Cho Dữ Liệu Nhạy Cảm
- ❌ Không ghi password vào YAML Deployment
- ✅ Dùng Secret hoặc biến môi trường

### 18.6 Label Nhất Quán
```yaml
labels:
  app: web
  env: prod
  version: "1.2.3"
  team: backend
```

### 18.7 Lưu YAML Vào Git (GitOps)
```bash
# Mọi thay đổi đều qua Git
# Dùng ArgoCD hoặc Flux để tự động sync
```

---

## 19. Tài Liệu Tham Khảo

- 🏠 Trang chủ: https://kubernetes.io
- 📚 Docs: https://kubernetes.io/docs/
- 🎓 Tutorial tương tác: https://kubernetes.io/docs/tutorials/
- 📖 kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- 🧪 Minikube: https://minikube.sigs.k8s.io/docs/
- 💬 Cộng đồng: https://kubernetes.slack.com

---

