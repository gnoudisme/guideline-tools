# Hướng Dẫn Tekton — CI/CD Kubernetes-Native Từ A Đến Z

> Tài liệu toàn diện về Tekton: framework CI/CD chạy hoàn toàn trên Kubernetes — từ khái niệm, cài đặt, Task, Pipeline, Triggers, đến bảo mật chuỗi cung ứng và dự án thực tế.

---

## Mục lục

1. [Tekton là gì](#1-tekton-là-gì)
2. [Cài đặt](#2-cài-đặt)
3. [Task & TaskRun](#3-task--taskrun)
4. [Pipeline & PipelineRun](#4-pipeline--pipelinerun)
5. [Workspaces](#5-workspaces)
6. [Triggers](#6-triggers)
7. [Tekton Hub](#7-tekton-hub)
8. [Tekton Dashboard](#8-tekton-dashboard)
9. [Tekton Chains](#9-tekton-chains)
10. [Dự án thực tế](#10-dự-án-thực-tế-build--test--push--deploy-app-lên-k8s)
11. [Best Practices](#11-best-practices)
12. [Troubleshooting](#12-troubleshooting)
13. [FAQ + Cheat Sheet](#13-faq--cheat-sheet)

---

## 1. Tekton là gì

**Tekton** là một framework CI/CD (Continuous Integration / Continuous Delivery) mã nguồn mở, được xây dựng **native trên Kubernetes** và hiện là dự án của **CD Foundation** (tổ chức thuộc Linux Foundation, cùng nhà với Jenkins và Argo).

### 1.1. Kiến trúc cốt lõi

Tekton định nghĩa pipeline CI/CD dưới dạng các **custom resource (CRD)** của Kubernetes:

| Khái niệm | Ý nghĩa | Tương đương |
|---|---|---|
| **Task** | Đơn vị công việc nhỏ nhất — một chuỗi các **Step** | 1 job trong CI |
| **Step** | Một lệnh chạy trong 1 container | 1 bước lệnh shell |
| **TaskRun** | Lần thực thi cụ thể của một Task | 1 lần chạy job |
| **Pipeline** | Sơ đồ nối nhiều Task theo thứ tự/đồ thị DAG | 1 pipeline đầy đủ |
| **PipelineRun** | Lần thực thi cụ thể của một Pipeline | 1 lần chạy pipeline |

Điểm khác biệt căn bản: **mỗi TaskRun chạy dưới dạng một Pod Kubernetes**. Mỗi Step trong Task là một container riêng trong cùng Pod, chia sẻ filesystem qua volume. Controller của Tekton (chạy trong namespace `tekton-pipelines`) theo dõi các CRD này và điều phối Pod tương ứng.

Luồng hoạt động:

```
PipelineRun (CRD)
  └─> Controller tạo TaskRun cho từng Task
        └─> Controller tạo Pod cho từng TaskRun
              └─> Pod chạy các Step (container) theo thứ tự
                    └─> Hoàn tất → cập nhật status TaskRun → PipelineRun
```

### 1.2. Tại sao gọi là "cloud-native"

- **Không cần server CI riêng:** Jenkins cần master + agent; Tekton tái sử dụng chính cluster Kubernetes bạn đang có — không có hạ tầng CI riêng để quản trị.
- **Mọi thứ là YAML trong Git:** Task, Pipeline, Trigger đều là CRD — version được bằng Git, áp dụng bằng `kubectl apply`, quản lý bằng GitOps (Argo CD, Flux).
- **Mở rộng theo Kubernetes:** muốn tăng độ song song? Scheduler của K8s lo. Pod xong tự xóa, không để lại agent zombie.
- **Chạy ở đâu cũng được:** on-prem, GCP, AWS, Azure — miễn có Kubernetes.

### 1.3. So sánh với Jenkins và GitHub Actions

| Tiêu chí | Tekton | Jenkins | GitHub Actions |
|---|---|---|---|
| Nền tảng chạy | Kubernetes (bắt buộc) | VM/container bất kỳ | GitHub-hosted hoặc self-hosted runner |
| Định nghĩa pipeline | YAML CRD (Kubernetes-native) | Groovy (Jenkinsfile) / YAML | YAML (`.github/workflows`) |
| Khả năng mở rộng | Theo cluster K8s | Cần cấu hình agent thủ công | Tự động (giới hạn quota) |
| Quản trị hạ tầng | Thấp (dùng cluster sẵn có) | Cao (quản lý master, agent, plugin) | Rất thấp (SaaS) |
| Plugin ecosystem | Tekton Hub (Task tái dùng) | Rất lớn (1800+ plugin) | Marketplace (20000+ action) |
| Vendor lock-in | Không (chạy mọi K8s) | Không | Phụ thuộc GitHub |
| Chạy offline/on-prem | Có | Có | Cần self-hosted runner |
| Độ dốc học tập | Trung bình–cao (cần biết K8s) | Trung bình | Thấp |

**Khi nào chọn Tekton?**

- Đội đã vận hành Kubernetes và muốn pipeline sống cùng cluster.
- Muốn mọi thứ là YAML, GitOps hóa toàn bộ CI/CD.
- Cần tránh lock-in vào một nền tảng SaaS.
- Cần tùy biến sâu ở tầng container/pod (security context, node selector, resource limit...).

**Khi nào không nên?**

- Chưa có Kubernetes (dựng cả cluster chỉ để chạy CI là quá đắt).
- Đội không quen K8s — Jenkins/GitHub Actions dễ tiếp cận hơn.

---

## 2. Cài đặt

### 2.1. Yêu cầu

- Một Kubernetes cluster đang chạy (phiên bản **1.27+** khuyến nghị).
- `kubectl` đã cấu hình, có quyền cluster-admin (để cài CRD).
- Cluster có ít nhất 2 CPU / 4GB RAM trống cho controller + chạy thử pipeline.

Kiểm tra cluster trước khi cài:

```bash
kubectl version --short
kubectl cluster-info
kubectl get nodes
```

### 2.2. Cài Tekton Pipelines

Cài bản mới nhất bằng một lệnh duy nhất:

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

Muốn ghim phiên bản cụ thể (khuyến nghị cho production, ví dụ v0.60):

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/previous/v0.60.0/release.yaml
```

Lệnh này tạo namespace `tekton-pipelines`, cài CRD (Task, TaskRun, Pipeline, PipelineRun...), controller, webhook và RBAC cần thiết.

Kiểm tra cài đặt thành công:

```bash
# Xem các pod của Tekton — tất cả phải ở trạng thái Running
kubectl get pods -n tekton-pipelines

# Kết quả mong đợi:
# NAME                                           READY   STATUS    RESTARTS   AGE
# tekton-pipelines-controller-xxxxxxxxxx-xxxxx   1/1     Running   0          2m
# tekton-pipelines-webhook-xxxxxxxxxx-xxxxx     1/1     Running   0          2m

# Kiểm tra CRD đã được đăng ký
kubectl get crds | grep tekton.dev
```

### 2.3. Cài Tekton CLI (`tkn`)

`tkn` là CLI chính thức giúp tương tác với Tekton nhanh hơn `kubectl` rất nhiều (xem log, mô tả pipeline, khởi chạy...).

**Linux:**

```bash
# Cách 1: script cài đặt chính thức
curl -LO https://github.com/tektoncd/cli/releases/latest/download/tkn_0.38.0_Linux_x86_64.tar.gz
tar xvzf tkn_0.38.0_Linux_x86_64.tar.gz -C /tmp
sudo mv /tmp/tkn /usr/local/bin/
```

**macOS:**

```bash
brew install tektoncd-cli
```

**Windows (PowerShell):**

```powershell
# Dùng Chocolatey
choco install tektoncd-cli
```

Kiểm tra:

```bash
tkn version
# Client version: 0.38.0
# Pipeline version: v0.60.x   (phiên bản Tekton trên cluster)
```

### 2.4. Cài các thành phần bổ trợ (tùy chọn, làm sau)

| Thành phần | Tác dụng | Sẽ cài ở |
|---|---|---|
| Tekton Triggers | Chạy pipeline tự động khi có webhook (GitHub push...) | Mục 6 |
| Tekton Dashboard | Giao diện web xem pipeline, log | Mục 8 |
| Tekton Chains | Ký image, tạo SLSA provenance | Mục 9 |
| Tekton Hub CLI | Cài Task có sẵn từ cộng đồng | Mục 7 |

---
## 3. Task & TaskRun

**Task** là "viên gạch" cơ bản của Tekton: một danh sách các **Step** chạy tuần tự, mỗi Step là một container image thực thi một lệnh.

### 3.1. Giải phẫu một Task

```yaml
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: hello-task
spec:
  steps:
    - name: say-hello            # tên step (hiển thị trong log)
      image: ubuntu:22.04        # image chạy step này
      script: |                  # lệnh thực thi (mặc định chạy bằng /bin/sh)
        #!/usr/bin/env bash
        echo "Xin chào từ Tekton!"
```

- `steps[].image`: bất kỳ image nào trên registry (Docker Hub, GHCR, ECR...).
- `steps[].script`: đoạn script chạy trong container. Nếu không dùng `script`, có thể dùng `command` + `args` kiểu Kubernetes thuần.
- Các step **chạy tuần tự** trong cùng một Pod, theo đúng thứ tự khai báo.

Áp dụng và chạy thử ngay:

```bash
kubectl apply -f - <<'EOF'
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: hello-task
spec:
  steps:
    - name: say-hello
      image: ubuntu:22.04
      script: |
        #!/usr/bin/env bash
        echo "Xin chào từ Tekton!"
EOF
```

### 3.2. TaskRun — thực thi một Task

Task chỉ là **định nghĩa** (giống class), còn **TaskRun** là **lần chạy cụ thể** (giống object):

```yaml
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  generateName: hello-task-run-   # tự sinh tên duy nhất
spec:
  taskRef:
    name: hello-task              # trỏ tới Task đã định nghĩa
```

```bash
kubectl apply -f taskrun.yaml

# Xem log bằng tkn (tiện hơn kubectl logs rất nhiều)
tkn taskrun logs --last -f
```

### 3.3. Task đầy đủ: params, results, workspaces

Đây là 3 khái niệm làm cho Task trở nên mạnh mẽ:

- **params** — tham số đầu vào (giống argument của hàm).
- **results** — giá trị đầu ra mà Task "trả về" cho Task khác dùng.
- **workspaces** — thư mục dùng chung để trao đổi file giữa các Task.

Ví dụ Task clone repo Git đầy đủ:

```yaml
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: git-clone
spec:
  params:
    - name: url                  # URL repo cần clone
      type: string
      description: URL của Git repository
    - name: revision             # nhánh/tag/commit
      type: string
      default: "main"
      description: Nhánh hoặc tag cần checkout
    - name: depth                # clone nông để nhanh
      type: string
      default: "1"
  workspaces:
    - name: output               # workspace chứa source code sau khi clone
      description: Thư mục chứa mã nguồn
  results:
    - name: commit               # trả về SHA commit đã clone
      description: SHA của commit vừa clone
    - name: url
      description: URL repo đã clone
  steps:
    - name: clone
      image: alpine/git:2.45.2
      script: |
        #!/usr/bin/env sh
        set -e
        cd $(workspaces.output.path)
        git clone --depth $(params.depth) \
          --branch $(params.revision) \
          $(params.url) .
        # Ghi result: Tekton đọc file trong /tekton/results/
        git rev-parse HEAD > /tekton/results/commit
        echo -n "$(params.url)" > /tekton/results/url
```

Giải thích chi tiết:

| Cú pháp | Ý nghĩa |
|---|---|
| `$(params.url)` | Tham chiếu giá trị param `url` |
| `$(workspaces.output.path)` | Đường dẫn mount của workspace `output` trong container |
| `/tekton/results/<tên>` | Ghi file vào đây để tạo **result** — Tekton tự đọc và lưu vào `status.results` của TaskRun |
| `$(results.commit.path)` | (Trong Task khác) đường dẫn đọc result — nhưng thường dùng qua Pipeline (mục 4) |

Chạy TaskRun cho Task có param + workspace:

```yaml
apiVersion: tekton.dev/v1
kind: TaskRun
metadata:
  generateName: git-clone-run-
spec:
  taskRef:
    name: git-clone
  params:
    - name: url
      value: https://github.com/tektoncd/pipeline.git
    - name: revision
      value: main
  workspaces:
    - name: output
      emptyDir: {}               # dùng emptyDir cho demo; production dùng PVC (mục 5)
```

```bash
kubectl apply -f git-clone-run.yaml
tkn taskrun logs --last -f

# Xem result trả về
tkn taskrun describe --last
# ... trong phần "Results" sẽ thấy commit SHA
```

### 3.4. Task chạy unit test

```yaml
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: go-test
spec:
  params:
    - name: package
      type: string
      default: ./...
      description: Package cần test
    - name: go-version
      type: string
      default: "1.22"
  workspaces:
    - name: source
  steps:
    - name: unit-test
      image: golang:1.22
      workingDir: $(workspaces.source.path)
      script: |
        #!/usr/bin/env bash
        set -e
        go test $(params.package) -v -count=1
```

### 3.5. Task build image với kaniko (không cần Docker daemon)

Trong Kubernetes, bạn **không nên** dùng Docker-in-Docker. **Kaniko** build image trực tiếp từ Dockerfile mà không cần Docker daemon — chuẩn cho Tekton:

```yaml
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: kaniko-build
spec:
  params:
    - name: IMAGE
      type: string
      description: Tên image đầy đủ, ví dụ registry.example.com/app:v1
    - name: DOCKERFILE
      type: string
      default: ./Dockerfile
    - name: CONTEXT
      type: string
      default: ./
  workspaces:
    - name: source
    - name: dockerconfig        # chứa file config.json đăng nhập registry
      optional: true
  steps:
    - name: build-and-push
      image: gcr.io/kaniko-project/executor:v1.23.2
      workingDir: $(workspaces.source.path)
      args:
        - --dockerfile=$(params.DOCKERFILE)
        - --context=$(workspaces.source.path)/$(params.CONTEXT)
        - --destination=$(params.IMAGE)
        - --cache=true
        - --compressed-caching=false
```

> Ngoài kaniko còn có **buildah** (build image kiểu daemonless, hỗ trợ tốt trên Tekton Hub). Chọn một trong hai tùy môi trường.

### 3.6. Các lệnh `tkn` cho Task

```bash
tkn task list                      # liệt kê Task
tkn task describe git-clone        # xem chi tiết Task
tkn task start git-clone \         # chạy Task tương tác (không cần viết TaskRun YAML)
  --param url=https://github.com/tektoncd/pipeline.git \
  --workspace name=output,emptyDir= \
  --showlog
tkn taskrun list                   # liệt kê TaskRun
tkn taskrun logs <tên> -f          # xem log ( -f = follow )
tkn taskrun delete --all           # dọn dẹp TaskRun cũ
```

---

## 4. Pipeline & PipelineRun

**Pipeline** nối nhiều Task thành một đồ thị có hướng (DAG): Task nào chạy trước, Task nào chạy sau, dữ liệu chảy giữa chúng ra sao.

### 4.1. Ví dụ Pipeline hoàn chỉnh: clone → test → build → push → deploy

```yaml
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: build-test-deploy
spec:
  params:
    - name: repo-url
      type: string
    - name: revision
      type: string
      default: main
    - name: image
      type: string
      description: Image đích, ví dụ registry.example.com/myapp:latest
  workspaces:
    - name: shared-workspace     # workspace dùng chung cho cả pipeline
    - name: dockerconfig
      optional: true

  tasks:
    - name: fetch-source
      taskRef:
        name: git-clone
      params:
        - name: url
          value: $(params.repo-url)
        - name: revision
          value: $(params.revision)
      workspaces:
        - name: output
          workspace: shared-workspace

    - name: run-tests
      taskRef:
        name: go-test
      runAfter: [fetch-source]   # chạy sau khi clone xong
      workspaces:
        - name: source
          workspace: shared-workspace

    - name: build-image
      taskRef:
        name: kaniko-build
      runAfter: [run-tests]      # chỉ build khi test pass
      params:
        - name: IMAGE
          value: $(params.image)
      workspaces:
        - name: source
          workspace: shared-workspace
        - name: dockerconfig
          workspace: dockerconfig

    - name: deploy
      taskRef:
        name: kubectl-deploy
      runAfter: [build-image]
      params:
        - name: image
          value: $(params.image)
        - name: manifest-dir
          value: k8s
      workspaces:
        - name: source
          workspace: shared-workspace

  finally:                       # luôn chạy, dù các task trên thành công hay thất bại
    - name: notify
      taskRef:
        name: send-notification
      params:
        - name: message
          value: "Pipeline $(params.repo-url) hoàn tất"
```

Task `kubectl-deploy` tham chiếu ở trên:

```yaml
apiVersion: tekton.dev/v1
kind: Task
metadata:
  name: kubectl-deploy
spec:
  params:
    - name: image
      type: string
    - name: manifest-dir
      type: string
      default: k8s
  workspaces:
    - name: source
  steps:
    - name: deploy
      image: bitnami/kubectl:1.30
      workingDir: $(workspaces.source.path)
      script: |
        #!/usr/bin/env bash
        set -e
        cd $(params.manifest-dir)
        # Thay image trong manifest rồi apply
        kubectl set image -f deployment.yaml app=$(params.image) --local -o yaml | kubectl apply -f -
        kubectl rollout status deployment/myapp --timeout=120s
```

### 4.2. Chạy Pipeline: PipelineRun

```yaml
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  generateName: build-test-deploy-run-
spec:
  pipelineRef:
    name: build-test-deploy
  params:
    - name: repo-url
      value: https://github.com/example/myapp.git
    - name: image
      value: registry.example.com/myapp:v1.0.0
  workspaces:
    - name: shared-workspace
      volumeClaimTemplate:       # tự tạo PVC riêng cho mỗi lần chạy
        spec:
          accessModes: [ReadWriteOnce]
          resources:
            requests:
              storage: 1Gi
    - name: dockerconfig
      secret:
        secretName: regcred       # secret docker-registry đã tạo trước
```

```bash
kubectl apply -f pipelinerun.yaml

# Xem log toàn bộ pipeline
tkn pipelinerun logs --last -f

# Xem log của 1 task cụ thể trong pipeline
tkn pipelinerun logs --last -f --task run-tests

# Khởi chạy tương tác không cần YAML
tkn pipeline start build-test-deploy \
  --param repo-url=https://github.com/example/myapp.git \
  --param image=registry.example.com/myapp:v1.0.0 \
  --workspace name=shared-workspace,volumeClaimTemplateFile=workspace-template.yaml \
  --showlog
```

### 4.3. Truyền result giữa các Task

Task A tạo result `commit`, Task B dùng nó qua cú pháp `$(tasks.<tên-task>.results.<tên-result>)`:

```yaml
tasks:
  - name: fetch-source
    taskRef:
      name: git-clone
    # ...

  - name: tag-image
    taskRef:
      name: kaniko-build
    runAfter: [fetch-source]
    params:
      # Tag image bằng 7 ký tự đầu của commit SHA
      - name: IMAGE
        value: "registry.example.com/myapp:$(tasks.fetch-source.results.commit)"
```

Truyền result giữa các Task qua cú pháp `$(tasks.<tên-task>.results.<tên-result>)` — xem ví dụ trong pipeline ở mục 10.

### 4.4. `finally` — task luôn chạy

Mục `finally` chứa các Task **luôn chạy sau khi mọi task thường hoàn tất**, bất kể thành công hay thất bại. Dùng cho:

- Gửi thông báo (Slack, email).
- Dọn dẹp tài nguyên tạm.
- Ghi nhận trạng thái pipeline vào hệ thống ngoài.

Trong `finally`, có thể dùng biến trạng thái tổng hợp:

```yaml
finally:
  - name: notify
    taskRef:
      name: send-notification
    params:
      - name: pipeline-status
        # Tekton cung cấp biến tổng hợp trạng thái
        value: $(tasks.status)   # "Succeeded", "Failed", "Completed"...
```

### 4.5. `when` — chạy Task có điều kiện

`when` cho phép **bỏ qua** Task khi điều kiện không thỏa (giống `if` trong GitHub Actions):

```yaml
tasks:
  - name: deploy-staging
    taskRef:
      name: kubectl-deploy
    runAfter: [build-image]
    when:
      - input: $(params.environment)     # giá trị cần kiểm tra
        operator: in                      # toán tử: in | notin
        values: ["staging"]               # deploy staging chỉ khi env=staging

  - name: deploy-prod
    taskRef:
      name: kubectl-deploy
    runAfter: [build-image]
    when:
      - input: $(params.environment)
        operator: in
        values: ["production"]
      - input: $(params.revision)         # có thể kết hợp nhiều điều kiện (AND)
        operator: in
        values: ["main"]
```

Khi điều kiện sai, Task bị **Skipped** (không phải Failed) — PipelineRun vẫn thành công.

### 4.6. Chạy song song

Các Task **không có `runAfter`** và không phụ thuộc result của nhau sẽ chạy **song song**:

```yaml
tasks:
  - name: lint
    taskRef: {name: golangci-lint}
    runAfter: [fetch-source]
  - name: unit-test
    taskRef: {name: go-test}
    runAfter: [fetch-source]
  # lint và unit-test chạy song song với nhau
  - name: build-image
    taskRef: {name: kaniko-build}
    runAfter: [lint, unit-test]   # chờ cả hai xong
```

### 4.7. Timeout và retry ở cấp Pipeline

```yaml
spec:
  timeouts:
    pipeline: 1h          # toàn pipeline tối đa 1 giờ
    tasks: 20m            # mỗi task tối đa 20 phút
    finally: 5m
  taskRunTemplate:
    retries: 2            # task thất bại thì thử lại tối đa 2 lần
```

---
## 5. Workspaces

**Workspace** là cách Tekton cho các Task **chia sẻ dữ liệu** (source code, artifact build...) — vì mỗi Task chạy trên Pod riêng, filesystem không dùng chung được.

### 5.1. Ý tưởng

- Task **khai báo** workspace nó cần: `spec.workspaces[].name`.
- TaskRun/PipelineRun **cung cấp** volume thật cho workspace đó (PVC, emptyDir, ConfigMap, Secret...).
- Trong container, workspace được mount tại `$(workspaces.<tên>.path)`.

### 5.2. Các loại volume thường dùng

**a) `emptyDir` — nhanh, tạm, mất khi Pod xóa**

```yaml
workspaces:
  - name: shared-workspace
    emptyDir: {}
```

Phù hợp: demo, pipeline chạy nhanh, không cần giữ dữ liệu sau khi xong.

**b) `persistentVolumeClaim` — dùng PVC có sẵn**

```yaml
workspaces:
  - name: shared-workspace
    persistentVolumeClaim:
      claimName: myapp-workspace-pvc
```

Phù hợp: cache build (Go module cache, npm cache) giữa các lần chạy — tăng tốc đáng kể.

Tạo PVC cache dùng chung:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: build-cache-pvc
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 10Gi
```

**c) `volumeClaimTemplate` — tự tạo PVC riêng cho mỗi lần chạy**

```yaml
workspaces:
  - name: shared-workspace
    volumeClaimTemplate:
      spec:
        accessModes: [ReadWriteOnce]
        resources:
          requests:
            storage: 1Gi
```

Phù hợp: production — mỗi PipelineRun có không gian riêng, không giẫm chân nhau, xong có thể dọn bằng chính sách retention.

**d) `secret` / `configMap` — đưa credential, file cấu hình vào**

```yaml
workspaces:
  - name: dockerconfig
    secret:
      secretName: regcred        # docker-registry secret
```

Tạo secret đăng nhập registry:

```bash
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password='mypass' \
  --docker-email=myuser@example.com
```

### 5.3. SubPath — chia nhỏ workspace

Nhiều Task có thể dùng chung một workspace nhưng mỗi Task một thư mục con:

```yaml
workspaces:
  - name: source
    workspace: shared-workspace
    subPath: frontend/          # chỉ thấy thư mục frontend/
```

### 5.4. Workspace `optional`

Task có thể đánh dấu workspace không bắt buộc — PipelineRun không cung cấp cũng không lỗi:

```yaml
# Trong Task
workspaces:
  - name: dockerconfig
    optional: true
```

### 5.5. So sánh nhanh

| Loại | Tồn tại sau Pod | Dùng chung giữa các run | Tốc độ |
|---|---|---|---|
| `emptyDir` | Không | Không | Nhanh nhất |
| `persistentVolumeClaim` | Có | Có (cần quản lý dọn dẹp) | Nhanh (cache) |
| `volumeClaimTemplate` | Có (đến khi xóa PVC) | Không (riêng mỗi run) | Trung bình |
| `secret`/`configMap` | Theo object gốc | Có | Nhanh |

---

## 6. Triggers

**Tekton Triggers** là thành phần giúp pipeline **tự động chạy khi có sự kiện** — ví dụ: push code lên GitHub → webhook → PipelineRun tự khởi tạo. Không có Triggers, bạn phải chạy `tkn pipeline start` thủ công.

### 6.1. Cài đặt Triggers

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
```

Kiểm tra:

```bash
kubectl get pods -n tekton-pipelines | grep trigger
# tekton-triggers-controller-...   1/1 Running
# tekton-triggers-webhook-...      1/1 Running
# tekton-triggers-core-interceptors-... 1/1 Running
```

### 6.2. Kiến trúc: 4 mảnh ghép

```
GitHub push
   │  (webhook POST)
   ▼
EventListener  ── "tai nghe": Service nhận HTTP request
   │  dùng TriggerBinding trích xuất dữ liệu từ payload
   ▼
TriggerBinding ── "bộ lọc": JSONPath lấy field từ webhook
   │  (repo url, branch, commit...)
   ▼
TriggerTemplate ── "khuôn": mẫu PipelineRun với tham số điền sẵn
   │  (+ Interceptor kiểm tra chữ ký webhook)
   ▼
PipelineRun được tạo tự động
```

| Thành phần | Vai trò | Ví dụ |
|---|---|---|
| **EventListener** | Expose HTTP endpoint nhận webhook | `el-github-listener` |
| **TriggerBinding** | Trích xuất param từ JSON payload (`$.repository.clone_url`) | `github-push-binding` |
| **TriggerTemplate** | Template tạo PipelineRun/TaskRun từ param | `build-template` |
| **Trigger** | Nối 3 thứ trên lại + interceptor | `github-push-trigger` |
| **Interceptor** | Xử lý trước: verify chữ ký, lọc sự kiện | `github` interceptor |

### 6.3. Ví dụ đầy đủ: GitHub push → chạy Pipeline

**Bước 1 — TriggerBinding:** lấy dữ liệu từ payload GitHub:

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: github-push-binding
spec:
  params:
    - name: gitrevision
      value: $(body.head_commit.id)          # SHA commit mới nhất
    - name: gitrepositoryurl
      value: $(body.repository.clone_url)    # URL repo
    - name: gitbranch
      value: $(body.ref)                     # refs/heads/main
```

Tham khảo các field payload GitHub push event: `body.ref`, `body.repository.clone_url`, `body.head_commit.id`, `body.pusher.name`.

**Bước 2 — TriggerTemplate:** khuôn PipelineRun:

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: build-test-deploy-template
spec:
  params:
    - name: gitrevision
    - name: gitrepositoryurl
    - name: gitbranch
  resourcetemplates:
    - apiVersion: tekton.dev/v1
      kind: PipelineRun
      metadata:
        generateName: build-test-deploy-
      spec:
        pipelineRef:
          name: build-test-deploy
        params:
          - name: repo-url
            value: $(tt.params.gitrepositoryurl)
          - name: revision
            value: $(tt.params.gitrevision)
          - name: image
            # Tag image bằng SHA ngắn — xử lý trong Task nếu cần
            value: registry.example.com/myapp:$(tt.params.gitrevision)
        workspaces:
          - name: shared-workspace
            volumeClaimTemplate:
              spec:
                accessModes: [ReadWriteOnce]
                resources:
                  requests: {storage: 1Gi}
          - name: dockerconfig
            secret:
              secretName: regcred
```

**Bước 3 — Trigger + EventListener:** nối lại và verify webhook:

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: Trigger
metadata:
  name: github-push-trigger
spec:
  interceptors:
    - ref:
        name: github                        # interceptor có sẵn: verify chữ ký
      params:
        - name: secretRef
          value:
            secretName: github-webhook-secret
            secretKey: secretToken
        - name: eventTypes
          value: ["push"]                   # chỉ nhận sự kiện push
  bindings:
    - ref: github-push-binding
  template:
    ref: build-test-deploy-template
---
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: github-listener
spec:
  serviceAccountName: tekton-triggers-sa
  triggers:
    - triggerRef: github-push-trigger
```

Tạo secret chứa webhook secret (phải trùng với secret cấu hình trên GitHub):

```bash
kubectl create secret generic github-webhook-secret \
  --from-literal=secretToken='mot-chuoi-bi-mat-dai-va-ngau-nhien'
```

**Bước 4 — Expose EventListener ra ngoài:**

```bash
# Cách nhanh khi test: port-forward
kubectl port-forward svc/el-github-listener 8080:8080

# Production: tạo Ingress hoặc dùng LoadBalancer
kubectl get svc el-github-listener
```

**Bước 5 — Cấu hình webhook trên GitHub:**

1. Vào repo → *Settings → Webhooks → Add webhook*.
2. **Payload URL:** `http://<địa-chỉ-public-của-listener>:8080`.
3. **Content type:** `application/json`.
4. **Secret:** chuỗi đã dùng ở trên.
5. **Events:** chọn *Just the push event*.
6. Save → GitHub gửi ping → EventListener trả 200 là thành công.

Từ giờ, mỗi lần `git push` lên nhánh được cấu hình, một PipelineRun mới tự động xuất hiện:

```bash
tkn pipelinerun list
```

### 6.4. ServiceAccount cho EventListener

EventListener cần quyền **tạo PipelineRun**. Tạo ServiceAccount riêng:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-triggers-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: triggers-pipelinerun-creator
rules:
  - apiGroups: ["tekton.dev"]
    resources: ["pipelineruns", "taskruns", "pipelines", "tasks"]
    verbs: ["create", "get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: triggers-pipelinerun-creator-binding
subjects:
  - kind: ServiceAccount
    name: tekton-triggers-sa
roleRef:
  kind: Role
  name: triggers-pipelinerun-creator
  apiGroup: rbac.authorization.k8s.io
```

> Nguyên tắc: chỉ cấp quyền tối thiểu (xem thêm mục 11).

### 6.5. Lọc sự kiện nâng cao với CEL Interceptor

Chỉ chạy pipeline khi push lên `main` (bỏ qua các nhánh khác):

```yaml
interceptors:
  - ref: {name: github}
    params: [...]
  - ref: {name: cel}                        # CEL interceptor có sẵn
    params:
      - name: filter
        value: "body.ref == 'refs/heads/main'"
      - name: overlays
        value:
          - key: short_sha
            expression: "body.head_commit.id.truncate(7)"
```

---

## 7. Tekton Hub

**Tekton Hub** (hub.tekton.dev) là "chợ" Task/Pipeline do cộng đồng đóng góp — giống Docker Hub nhưng cho Task Tekton. Thay vì tự viết `git-clone`, `buildah`, `kubectl-deploy`, bạn cài bản đã được kiểm chứng.

### 7.1. Cài đặt từ Hub bằng `tkn`

```bash
# Tìm task
tkn hub search git-clone

# Xem thông tin chi tiết (param, workspace, ví dụ)
tkn hub info task git-clone

# Cài task vào cluster (namespace hiện tại)
tkn hub install task git-clone

# Cài phiên bản cụ thể
tkn hub install task git-clone --version 0.9

# Cài vào namespace khác
tkn hub install task buildah --namespace tekton-pipelines
```

Một số Task phổ biến nên biết:

| Task | Tác dụng |
|---|---|
| `git-clone` | Clone repo Git (bản chuẩn của Tekton) |
| `buildah` | Build image không cần Docker daemon |
| `kaniko` | Build image bằng Kaniko |
| `kubectl-deploy` / `kubernetes-actions` | Apply manifest, rollout |
| `golang-test` / `golang-build` | Build & test dự án Go |
| `npm` / `yarn` | Build & test dự án Node.js |
| `pytest` | Chạy test Python |
| `sonarqube-scanner` | Quét chất lượng code |
| `trivy-scanner` | Quét lỗ hổng image |
| `send-to-slack` / `sendmail` | Gửi thông báo |

### 7.2. Dùng Task từ Hub trong Pipeline

Sau khi `tkn hub install task git-clone`, Task `git-clone` đã có trong cluster — chỉ việc `taskRef` tới nó:

```yaml
tasks:
  - name: fetch-source
    taskRef:
      name: git-clone
      kind: Task
    params:
      - name: url
        value: $(params.repo-url)
      - name: revision
        value: $(params.revision)
    workspaces:
      - name: output
        workspace: shared-workspace
```

### 7.3. Nâng cấp và gỡ

```bash
tkn hub upgrade task git-clone --to 0.10
tkn hub uninstall task git-clone
```

> Production nên **ghim version** Task Hub trong Git (export YAML sau khi install rồi commit), tránh việc Hub cập nhật làm đổi hành vi pipeline.

---
## 8. Tekton Dashboard

**Tekton Dashboard** là giao diện web giúp xem Pipeline/Task, theo dõi PipelineRun realtime, xem log từng step mà không cần gõ `tkn`.

### 8.1. Cài đặt

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/dashboard/latest/release.yaml

# Kiểm tra
kubectl get pods -n tekton-pipelines | grep dashboard
# tekton-dashboard-xxxxxxxxxx-xxxxx   1/1 Running
```

### 8.2. Truy cập

```bash
# Cách nhanh nhất khi test local
kubectl port-forward -n tekton-pipelines svc/tekton-dashboard 9097:9097
# Mở trình duyệt: http://localhost:9097
```

Production — expose qua Ingress (ví dụ NGINX):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tekton-dashboard
  namespace: tekton-pipelines
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: dashboard-auth
    nginx.ingress.kubernetes.io/auth-realm: "Tekton Dashboard"
spec:
  rules:
    - host: tekton.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: tekton-dashboard
                port: {number: 9097}
```

> Dashboard mặc định **không có đăng nhập**. Nếu expose ra internet, bắt buộc đặt sau xác thực (basic auth, OAuth2-proxy...) hoặc giới hạn IP.

### 8.3. Những gì xem được trên Dashboard

- **Pipelines / Tasks:** danh sách định nghĩa, xem YAML, nút *Create PipelineRun* điền param trực quan.
- **PipelineRuns / TaskRuns:** trạng thái từng run (Running/Succeeded/Failed), sơ đồ DAG, thời gian chạy từng Task.
- **Logs:** xem log realtime từng step, không cần `kubectl logs`.
- **Triggers:** xem EventListener, TriggerBinding, TriggerTemplate (khi cài Triggers).

---

## 9. Tekton Chains — Bảo mật chuỗi cung ứng

**Tekton Chains** là thành phần ký số artifact và tạo chứng thực nguồn gốc (provenance) — trả lời câu hỏi: *"Image này có đúng là được build từ commit X bởi pipeline Y không, có bị chèn mã độc dọc đường không?"*

### 9.1. Cài đặt

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/chains/latest/release.yaml

kubectl get pods -n tekton-chains
# tekton-chains-controller-...   1/1 Running
```

### 9.2. Cơ chế hoạt động

1. Mỗi khi TaskRun build image (kaniko/buildah) hoàn tất, Chains **tự động** quan sát.
2. Chains tạo **SLSA provenance** — bản kê khai có chữ ký ghi lại: image digest, source repo + commit, Task/Pipeline nào đã build, thời gian, builder ID.
3. Chains **ký image** bằng **cosign** (Sigstore), lưu chữ ký lên registry cạnh image.
4. Kết quả lưu dạng **attestation** (in-toto format), có thể verify độc lập.

```
TaskRun build image xong
        │
        ▼
Tekton Chains controller
   ├─ Ký image bằng cosign (key hoặc keyless qua Fulcio)
   └─ Tạo SLSA provenance attestation
        │
        ▼
Registry: image + signature + attestation
```

### 9.3. Cấu hình ký bằng cosign (keyless — không cần giữ key)

Chế độ **keyless** dùng OIDC + Fulcio/Rekor của Sigstore — không cần tạo và bảo vệ private key:

```bash
# Cấu hình Chains dùng keyless
kubectl patch configmap chains-config -n tekton-chains \
  --type merge -p '{"data":{"signers.x509.fulcio.enabled":"true","signers.x509.fulcio.address":"https://fulcio.sigstore.dev","transparency.enabled":"true"}}'

# Khởi động lại controller để nhận cấu hình
kubectl rollout restart deployment tekton-chains-controller -n tekton-chains
```

Chế độ **dùng key riêng** (môi trường air-gapped / chính sách nội bộ):

```bash
# Tạo key cosign
cosign generate-key-pair   # tạo cosign.key + cosign.pub

# Lưu vào secret cho Chains
kubectl create secret generic signing-secrets \
  -n tekton-chains \
  --from-file=cosign.key=cosign.key \
  --from-file=cosign.pub=cosign.pub \
  --from-literal=cosign.password=''
```

### 9.4. Verify image đã ký

Sau khi pipeline chạy xong, kiểm chứng image từ máy bất kỳ:

```bash
# Verify chữ ký (keyless)
cosign verify registry.example.com/myapp:v1.0.0 \
  --certificate-identity-regexp '.*' \
  --certificate-oidc-issuer-regexp '.*'

# Xem SLSA provenance
cosign verify-attestation \
  --type slsaprovenance \
  --certificate-identity-regexp '.*' \
  --certificate-oidc-issuer-regexp '.*' \
  registry.example.com/myapp:v1.0.0
```

Kết quả cho biết chính xác: image digest nào, build từ repo/commit nào, bởi TaskRun nào, lúc nào.

### 9.5. Kết hợp quét lỗ hổng trong pipeline

Chuỗi cung ứng an toàn = **ký + quét**. Thêm Task quét Trivy trước khi ký/deploy:

```yaml
tasks:
  - name: scan-image
    taskRef:
      name: trivy-scanner        # từ Tekton Hub
    runAfter: [build-image]
    params:
      - name: IMAGE_PATH
        value: $(params.image)
      - name: SEVERITY
        value: CRITICAL,HIGH
```

> Có thể đặt chính sách "fail pipeline nếu phát hiện CVE Critical" ngay trong Task quét.

---

## 10. Dự án thực tế: Build + Test + Push + Deploy app lên K8s

Mục này dựng một pipeline **chạy được thật** cho một app mẫu (Go và Python), từ repo Git đến khi app chạy trên cluster.

### 10.1. Chuẩn bị app mẫu

**App Go** (`main.go` + `go.mod` + `Dockerfile`):

```go
// main.go
package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Xin chào từ Tekton!")
	})
	http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
		w.WriteHeader(http.StatusOK)
		fmt.Fprintln(w, "ok")
	})
	port := os.Getenv("PORT")
	if port == "" {
		port = "8080"
	}
	http.ListenAndServe(":"+port, nil)
}
```

```dockerfile
# Dockerfile (multi-stage cho Go)
FROM golang:1.22-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server .

FROM gcr.io/distroless/static:nonroot
COPY --from=builder /app/server /server
EXPOSE 8080
ENTRYPOINT ["/server"]
```

**App Python** (thay thế tương đương):

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.get("/")
def index():
    return "Xin chào từ Tekton!"

@app.get("/healthz")
def healthz():
    return "ok", 200

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

```dockerfile
# Dockerfile cho Python
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 8080
CMD ["python", "app.py"]
# requirements.txt: Flask==3.0.3
```

**Manifest K8s** (`k8s/deployment.yaml` + `k8s/service.yaml`):

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels: {app: myapp}
  template:
    metadata:
      labels: {app: myapp}
    spec:
      containers:
        - name: app
          image: registry.example.com/myapp:latest   # pipeline sẽ thay bằng tag thật
          ports: [{containerPort: 8080}]
          livenessProbe:
            httpGet: {path: /healthz, port: 8080}
            periodSeconds: 15
          readinessProbe:
            httpGet: {path: /healthz, port: 8080}
            periodSeconds: 5
          resources:
            requests: {cpu: 100m, memory: 128Mi}
            limits: {cpu: 500m, memory: 512Mi}
---
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp
spec:
  selector: {app: myapp}
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

Đẩy toàn bộ lên một Git repo (ví dụ `https://github.com/example/myapp`).

### 10.2. Bước 1 — Tạo Secret registry

```bash
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password='mat-khau-that' \
  --docker-email=myuser@example.com
```

### 10.3. Bước 2 — ServiceAccount riêng cho pipeline

```yaml
# pipeline-sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pipeline-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pipeline-deployer
rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "patch", "watch"]
  - apiGroups: [""]
    resources: ["services"]
    verbs: ["get", "list", "create", "update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pipeline-deployer-binding
subjects:
  - kind: ServiceAccount
    name: pipeline-sa
roleRef:
  kind: Role
  name: pipeline-deployer
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f pipeline-sa.yaml
```

> Giải thích: pipeline chỉ cần quyền patch Deployment và quản lý Service trong namespace của nó — không cho cluster-admin.

### 10.4. Bước 3 — Cài Task cần thiết từ Hub

```bash
tkn hub install task git-clone
tkn hub install task kaniko
```

Task `go-test` / `pytest` tự viết (mục 3.4) hoặc lấy từ Hub:

```bash
tkn hub install task golang-test
```

### 10.5. Bước 4 — Định nghĩa Pipeline

```yaml
# myapp-pipeline.yaml
apiVersion: tekton.dev/v1
kind: Pipeline
metadata:
  name: myapp-ci-cd
spec:
  params:
    - name: repo-url
      type: string
    - name: revision
      type: string
      default: main
    - name: image
      type: string
  workspaces:
    - name: shared
    - name: dockerconfig
      optional: true
  tasks:
    - name: clone
      taskRef: {name: git-clone, kind: Task}
      params:
        - {name: url, value: $(params.repo-url)}
        - {name: revision, value: $(params.revision)}
      workspaces:
        - {name: output, workspace: shared}

    - name: test
      taskRef: {name: golang-test, kind: Task}   # hoặc pytest cho app Python
      runAfter: [clone]
      workspaces:
        - {name: source, workspace: shared}

    - name: build
      taskRef: {name: kaniko, kind: Task}
      runAfter: [test]
      params:
        - {name: IMAGE, value: $(params.image)}
      workspaces:
        - {name: source, workspace: shared}
        - {name: dockerconfig, workspace: dockerconfig}

    - name: deploy
      taskRef: {name: kubectl-deploy, kind: Task}  # Task tự viết ở mục 4.1
      runAfter: [build]
      params:
        - {name: image, value: $(params.image)}
        - {name: manifest-dir, value: k8s}
      workspaces:
        - {name: source, workspace: shared}

    - name: smoke-test
      runAfter: [deploy]
      taskSpec:                      # định nghĩa Task inline, không cần tạo Task riêng
        steps:
          - name: curl-health
            image: curlimages/curl:8.9.1
            script: |
              #!/usr/bin/env sh
              set -e
              for i in $(seq 1 12); do
                if curl -sf http://myapp/healthz; then
                  echo "Smoke test PASSED"
                  exit 0
                fi
                echo "Thử lại $i/12..."
                sleep 10
              done
              echo "Smoke test FAILED"
              exit 1
```

> `taskSpec` cho phép nhúng Task trực tiếp trong Pipeline — tiện cho các bước nhỏ, một lần, không cần tái dùng.

### 10.6. Bước 5 — Chạy PipelineRun

```yaml
# myapp-pipelinerun.yaml
apiVersion: tekton.dev/v1
kind: PipelineRun
metadata:
  generateName: myapp-ci-cd-
spec:
  serviceAccountName: pipeline-sa
  pipelineRef:
    name: myapp-ci-cd
  timeouts:
    pipeline: 30m
  params:
    - name: repo-url
      value: https://github.com/example/myapp.git
    - name: image
      value: registry.example.com/myapp:v1.0.0
  workspaces:
    - name: shared
      volumeClaimTemplate:
        spec:
          accessModes: [ReadWriteOnce]
          resources:
            requests: {storage: 1Gi}
    - name: dockerconfig
      secret:
        secretName: regcred
```

```bash
kubectl apply -f myapp-pipeline.yaml
kubectl apply -f myapp-pipelinerun.yaml

# Theo dõi
tkn pipelinerun logs --last -f
```

### 10.7. Bước 6 — Kiểm tra kết quả

```bash
# PipelineRun thành công?
tkn pipelinerun describe --last

# App đã deploy?
kubectl get deployment myapp
kubectl rollout status deployment/myapp

# Gọi thử service
kubectl port-forward svc/myapp 8080:80
curl localhost:8080/          # Xin chào từ Tekton!
curl localhost:8080/healthz   # ok
```

### 10.8. Bước 7 (nâng cao) — Gắn Trigger tự động

Dùng lại bộ Trigger ở mục 6, chỉ cần đổi `TriggerTemplate` trỏ sang Pipeline `myapp-ci-cd` và param `image` dùng tag theo commit. Từ đây, mỗi `git push` → build → test → push image → deploy → smoke test **hoàn toàn tự động**.

Sơ đồ tổng thể dự án:

```
git push ──▶ EventListener ──▶ Trigger ──▶ PipelineRun (myapp-ci-cd)
                                                    │
                    ┌───────────┬───────────┬───────┴────────┬──────────┐
                    ▼           ▼           ▼                ▼          ▼
                 git-clone  golang-test   kaniko        kubectl-    smoke-test
                 (Task Hub)  (Task Hub)  (Task Hub)      deploy     (inline)
                                                    │
                                                    ▼
                                              Chains ký image
                                              (SLSA provenance)
```

---
## 11. Best Practices

### 11.1. Task nhỏ, đơn nhiệm, tái sử dụng

- Một Task chỉ làm **một việc**: clone, test, build, deploy — không nhét "build + test + push" vào một Task khổng lồ.
- Task generic nhận param (repo URL, image name...) thay vì hardcode — dùng lại được cho nhiều dự án.
- Đóng gói Task dùng chung của tổ chức thành **catalog nội bộ** (lưu trong Git repo riêng, cài bằng `kubectl apply -k`).

### 11.2. Luôn đặt timeout

Pipeline treo vì test chờ input, `curl` chờ mạng... là chuyện thường. Đặt timeout ở mọi cấp:

```yaml
# Cấp Task
spec:
  timeout: 10m
# Cấp Pipeline
spec:
  timeouts:
    pipeline: 1h
    tasks: 20m
```

### 11.3. Retry có chiến lược

```yaml
spec:
  taskRunTemplate:
    retries: 2
```

- Retry phù hợp cho lỗi **tạm thời** (mạng chập chờn, registry timeout).
- Không retry Task deploy mù quáng — deploy nửa chừng rồi chạy lại có thể gây trạng thái dở dang. Hãy làm Task deploy **idempotent** (chạy lại cho kết quả như nhau, như `kubectl apply`).

### 11.4. ServiceAccount riêng, quyền tối thiểu

- Mỗi Pipeline (hoặc mỗi team) một ServiceAccount riêng — không dùng `default`.
- Chỉ cấp đúng verb/resource cần thiết (ví dụ pipeline CI chỉ cần đọc repo + push image, không cần quyền deploy).
- Credential registry để trong Secret, mount qua workspace — **không** hardcode vào Task, không `echo` password ra log.

### 11.5. Quản lý TaskRun/PipelineRun cũ

Mỗi lần chạy tạo Pod + PVC (nếu dùng volumeClaimTemplate) — không dọn sẽ đầy cluster:

```bash
# Xóa thủ công
tkn pipelinerun delete --all
tkn taskrun delete --all
```

Tự động dọn bằng chính sách giữ lại trong PipelineRun:

```yaml
spec:
  # Giữ tối đa 5 run thành công, 3 run thất bại gần nhất
  persistentAffinity: false
---
# Hoặc dùng TTL (Tekton >= 0.44):
# pruner component hoặc cronjob chạy tkn delete định kỳ
```

> `tkn pipelinerun delete --all --keep 5`: giữ lại 5 run mới nhất, xóa phần còn lại — có thể đặt vào CronJob chạy mỗi đêm để dọn tự động.

### 11.6. Ghim phiên bản mọi thứ

- Ghim `image` kèm tag digest (`golang:1.22` → `golang:1.22.5@sha256:...`) trong Task quan trọng.
- Ghim version Tekton Pipelines/Triggers khi cài (không dùng `latest` cho production).
- Ghim version Task từ Hub (export YAML vào Git).

### 11.7. Tách namespace theo môi trường

```
team-a-dev      → pipeline dev của team A
team-a-prod     → pipeline prod của team A (RBAC chặt hơn)
tekton-pipelines → controller (không chạy workload ở đây)
```

### 11.8. Quan sát được (observability)

- Đặt `description` cho Task/Pipeline để Dashboard hiển thị rõ ràng.
- Dùng `results` để các bước sau dùng lại (commit SHA, image digest) thay vì parse log.
- Ship log Pod Tekton vào hệ thống log tập trung (Loki/ELK) như mọi workload khác — label `tekton.dev/task` rất tiện để lọc.

---

## 12. Troubleshooting

### 12.1. Pod của TaskRun ở trạng thái Pending mãi

**Triệu chứng:** `tkn taskrun describe` thấy Pod Pending, pipeline không chạy.

**Nguyên nhân & cách xử lý:**

```bash
kubectl describe pod <tên-pod-taskrun>
```

| Dòng sự kiện (Events) | Nguyên nhân | Cách sửa |
|---|---|---|
| `Insufficient cpu/memory` | Cluster hết tài nguyên | Giảm `resources.requests` của step, hoặc thêm node |
| `persistentvolumeclaim "pvc-xxx" not found` | PVC chưa bind | Kiểm tra StorageClass, `kubectl get pvc` |
| `FailedScheduling ... node(s) didn't match Pod's node affinity` | `nodeSelector`/`affinity` trong Task không khớp node nào | Sửa lại selector cho đúng label node |
| `0/3 nodes are available: 3 node(s) had taint` | Node bị taint | Thêm toleration hoặc bỏ taint |

Đặt resource hợp lý cho step nặng (kaniko/buildah cần RAM):

```yaml
steps:
  - name: build-and-push
    image: gcr.io/kaniko-project/executor:v1.23.2
    computeResources:
      requests: {memory: 1Gi, cpu: 500m}
      limits: {memory: 4Gi, cpu: 2}
```

### 12.2. Lỗi workspace binding

**Triệu chứng:** `Workspace binding failed` / `volumeClaimTemplate` lỗi.

- Kiểm tra tên workspace ở Task (`spec.workspaces[].name`) **khớp chính xác** tên khai báo ở PipelineRun (`spec.workspaces[].name`) — sai một ký tự cũng lỗi.
- Workspace `optional: true` trong Task nhưng PipelineRun không cung cấp → OK, không lỗi. Ngược lại, Task bắt buộc mà PipelineRun thiếu → lỗi.
- `volumeClaimTemplate` cần cluster có **StorageClass mặc định** hỗ trợ dynamic provisioning:

```bash
kubectl get storageclass
# Phải có 1 dòng (default). Nếu không, PVC ở trạng thái Pending vĩnh viễn.
```

### 12.3. ImagePullBackOff / ErrImagePull

```bash
kubectl describe pod <pod> | grep -A5 "Failed to pull image"
```

- **Sai tên/tag image:** kiểm tra lại `steps[].image`.
- **Registry private:** Pod TaskRun dùng ServiceAccount — phải gắn `imagePullSecrets`:

```bash
kubectl patch serviceaccount pipeline-sa \
  -p '{"imagePullSecrets": [{"name": "regcred"}]}'
```

- **Rate limit Docker Hub** (lỗi 429): dùng mirror registry hoặc đăng nhập (tạo secret `docker.io`).

### 12.4. Trigger không chạy khi push code

Checklist từng bước:

1. **EventListener có nhận request?**
   ```bash
   kubectl logs -l eventlistener=github-listener
   # Phải thấy dòng POST / với status 200/201
   ```
2. **Webhook GitHub báo gì?** Vào repo → *Settings → Webhooks → Recent Deliveries*: xem response code. 4xx/5xx → đọc body.
3. **Sai secret?** Secret trên GitHub phải trùng `secretToken` trong secret `github-webhook-secret` — interceptor `github` sẽ từ chối (403) nếu lệch.
4. **Sai event type?** Interceptor lọc `eventTypes: ["push"]` — push tag/release không phải push branch vẫn là push event, nhưng PR thì không.
5. **CEL filter loại bỏ?** Test expression `body.ref == 'refs/heads/main'` — push nhánh khác bị lọc là đúng thiết kế.
6. **ServiceAccount thiếu quyền tạo PipelineRun?** Xem log `tekton-triggers-controller` tìm `Forbidden`.

### 12.5. TaskRun Failed nhưng log trống

- Pod bị **evict** (hết tài nguyên node) trước khi log kịp ghi → `kubectl describe pod` xem Events.
- Container **OOMKilled** (vượt memory limit) → tăng `computeResources.limits.memory`.
- Lỗi ngay khi khởi tạo step (image không tồn tại) → kiểm tra `kubectl get events --sort-by=.lastTimestamp`.

### 12.6. Kaniko build thất bại

| Lỗi | Cách sửa |
|---|---|
| `UNAUTHORIZED: authentication required` | Workspace `dockerconfig` chưa đúng — kiểm tra secret `regcred`, đảm bảo Task mount đúng workspace tên `dockerconfig` |
| `no such file or directory` cho Dockerfile | `workingDir` + `CONTEXT` sai — in `ls -la` trong step debug |
| Build chậm / timeout | Bật `--cache=true` + `--cache-repo=<repo-cache>` để dùng layer cache |
| `error building image: failed to get...` do mạng | Thêm retry ở `taskRunTemplate.retries` |

### 12.7. PipelineRun treo ở Running

```bash
# Xem task nào đang kẹt
tkn pipelinerun describe --last

# Xem chi tiết taskrun con
tkn taskrun list | grep <pipelinerun-name>
kubectl describe taskrun <tên>
```

- Thường do thiếu `timeout` — đặt `spec.timeouts.pipeline`.
- Hoặc Task chờ input tương tác (script có `read`, `apt install` hỏi yes/no) — mọi script phải non-interactive (`-y`, `DEBIAN_FRONTEND=noninteractive`).

### 12.8. Lệnh debug vạn năng

```bash
# Xem events mới nhất trong namespace (rất nhiều manh mối ở đây)
kubectl get events --sort-by=.lastTimestamp | tail -30

# Mô tả TaskRun/PipelineRun xem condition chi tiết
kubectl describe pipelinerun <tên>
kubectl describe taskrun <tên>

# Log init container + step cụ thể
kubectl logs <pod-taskrun> -c step-clone
```

---

## 13. FAQ + Cheat Sheet

### 13.1. FAQ

**Hỏi: Tekton có thay thế hoàn toàn Jenkins được không?**
Đáp: Được, nếu đội đã chạy trên Kubernetes và pipeline biểu diễn được bằng Task/Pipeline YAML. Jenkins vẫn hơn ở hệ plugin khổng lồ và chạy ngoài K8s. Nhiều tổ chức chạy song song: Jenkins cho job legacy, Tekton cho workload cloud-native mới.

**Hỏi: Mỗi Step là một container — vậy biến môi trường/file giữa các step có dùng chung không?**
Đáp: **File dùng chung được** (các step mount chung volume của Task, mặc định là `/workspace`). **Biến môi trường thì không** — mỗi step là process riêng. Muốn truyền giá trị giữa step, ghi ra file trong `/workspace` rồi step sau đọc.

**Hỏi: Pipeline có gọi API / chạy trên nhiều cluster được không?**
Đáp: Một PipelineRun sống trong một cluster. Muốn multi-cluster: dùng Task gọi `kubectl`/`argocd` trỏ sang cluster khác (qua kubeconfig trong Secret), hoặc mỗi cluster một pipeline con rồi điều phối bằng công cụ bên ngoài.

**Hỏi: Tekton lưu lịch sử chạy ở đâu? Có database không?**
Đáp: Không có database riêng — mọi trạng thái nằm trong `etcd` của Kubernetes dưới dạng TaskRun/PipelineRun object. Vì vậy cần chính sách dọn run cũ (mục 11.5), nếu không etcd phình to.

**Hỏi: Có cần Docker daemon / Docker-in-Docker không?**
Đáp: Không — và không nên dùng. Dùng kaniko hoặc buildah (mục 3.5).

**Hỏi: Tekton có miễn phí không?**
Đáp: Hoàn toàn mã nguồn mở (Apache 2.0), không tốn phí license.

### 13.2. Cheat Sheet — lệnh `tkn` hay dùng

```bash
# ── Phiên bản ──────────────────────────────────────────────
tkn version

# ── Task ───────────────────────────────────────────────────
tkn task list                          # liệt kê Task
tkn task describe <tên>                # xem chi tiết
tkn task start <tên> --showlog         # chạy Task (tương tác điền param)
tkn task start <tên> \
  --param url=https://github.com/example/repo.git \
  --param revision=main \
  --workspace name=output,emptyDir= \
  --showlog
tkn task delete <tên>

# ── Pipeline ───────────────────────────────────────────────
tkn pipeline list
tkn pipeline describe <tên>
tkn pipeline start <tên> --showlog     # chạy Pipeline tương tác
tkn pipeline start <tên> \
  --param repo-url=https://github.com/example/repo.git \
  --param image=registry.example.com/app:v1 \
  --workspace name=shared,claimName=build-cache-pvc \
  --showlog
tkn pipeline delete <tên>

# ── TaskRun / PipelineRun ──────────────────────────────────
tkn taskrun list
tkn taskrun describe <tên>
tkn taskrun logs <tên> -f              # xem log, -f để follow
tkn taskrun logs --last -f             # log của run mới nhất
tkn taskrun delete --all               # xóa tất cả
tkn taskrun delete --all --keep 5      # giữ lại 5 run mới nhất

tkn pipelinerun list
tkn pipelinerun describe <tên>
tkn pipelinerun logs <tên> -f
tkn pipelinerun logs --last -f --task run-tests   # log 1 task trong pipeline
tkn pipelinerun delete --all --keep 5
tkn pipelinerun cancel <tên>           # hủy run đang chạy

# ── Hub ────────────────────────────────────────────────────
tkn hub search <từ-khóa>
tkn hub info task <tên>
tkn hub install task <tên> --version 0.9
tkn hub upgrade task <tên> --to 0.10
tkn hub uninstall task <tên>

# ── Namespace khác ─────────────────────────────────────────
tkn -n <namespace> pipeline list        # mọi lệnh tkn đều nhận -n
```

### 13.3. Cheat Sheet — `kubectl` cho Tekton

```bash
# Trạng thái controller
kubectl get pods -n tekton-pipelines

# Mọi CRD Tekton
kubectl get tasks,pipelines,taskruns,pipelineruns

# Xem YAML đầy đủ của một run (debug result, condition)
kubectl get pipelinerun <tên> -o yaml

# Xem results của TaskRun
kubectl get taskrun <tên> -o jsonpath='{.status.results}'

# Pod thực thi của TaskRun
kubectl get pods -l tekton.dev/taskRun=<tên-taskrun>

# Events mới nhất (debug nhanh mọi lỗi)
kubectl get events --sort-by=.lastTimestamp | tail -30

# Áp dụng hàng loạt file pipeline
kubectl apply -f ./tekton/            # cả thư mục
kubectl apply -k ./tekton/            # dùng kustomize
```

### 13.4. Tài liệu tham khảo

| Nguồn | Địa chỉ |
|---|---|
| Tài liệu chính thức | tekton.dev/docs |
| Tekton Hub (Task có sẵn) | hub.tekton.dev |
| GitHub tổ chức | github.com/tektoncd |
| Ví dụ pipeline mẫu | github.com/tektoncd/pipeline/tree/main/examples |
| Chuẩn SLSA | slsa.dev |
| Cosign / Sigstore | docs.sigstore.dev |

---
