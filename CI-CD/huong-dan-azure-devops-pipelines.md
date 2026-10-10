# Hướng dẫn toàn diện: Azure DevOps Pipelines

Tài liệu tiếng Việt xây dựng CI/CD bằng Azure Pipelines: từ tạo organization, viết `azure-pipelines.yml`, triggers, variables, agents, environments, deployment strategies, templates, ví dụ thực tế hoàn chỉnh, pricing, troubleshooting đến checklist best practices.

> **Cần**: tài khoản Microsoft, một repo code, kiến thức Git cơ bản.

---

## Mục lục

1. [Azure DevOps là gì? Các thành phần](#1-azure-devops-là-gì-các-thành-phần)
2. [Tạo Organization, Project và kết nối Repo](#2-tạo-organization-project-và-kết-nối-repo)
3. [azure-pipelines.yml: pipeline đầu tiên](#3-azure-pipelinesyml-pipeline-đầu-tiên)
4. [Cú pháp YAML đầy đủ](#4-cú-pháp-yaml-đầy-đủ)
5. [Triggers: CI, PR và Schedule](#5-triggers-ci-pr-và-schedule)
6. [Variables và Variable Groups](#6-variables-và-variable-groups)
7. [Service Connections](#7-service-connections)
8. [Agents: Microsoft-hosted và Self-hosted](#8-agents-microsoft-hosted-và-self-hosted)
9. [Các Task phổ biến](#9-các-task-phổ-biến)
10. [Artifacts: publish và download giữa các stages](#10-artifacts-publish-và-download-giữa-các-stages)
11. [Environments: approvals, checks và deployment history](#11-environments-approvals-checks-và-deployment-history)
12. [Deployment Strategies](#12-deployment-strategies)
13. [Templates: tái sử dụng YAML](#13-templates-tái-sử-dụng-yaml)
14. [Ví dụ thực tế hoàn chỉnh](#14-ví-dụ-thực-tế-hoàn-chỉnh)
15. [Classic Release Pipelines](#15-classic-release-pipelines)
16. [Pricing](#16-pricing)
17. [Troubleshooting: 15 lỗi thường gặp](#17-troubleshooting-15-lỗi-thường-gặp)
18. [Best Practices Checklist](#18-best-practices-checklist)
19. [FAQ và Cheat Sheet](#19-faq-và-cheat-sheet)

---

## 1. Azure DevOps là gì? Các thành phần

**Azure DevOps** là bộ công cụ phát triển phần mềm dạng SaaS của Microsoft, bao phủ toàn bộ vòng đời ứng dụng: quản lý code, CI/CD, quản lý công việc, package và kiểm thử. Hai dạng triển khai:

**Azure DevOps Services** (cloud, `dev.azure.com`) và **Azure DevOps Server** (on-premise, tiền thân là TFS).

5 thành phần chính:

| Thành phần | Biểu tượng | Chức năng chính |
|---|---|---|
| **Azure Repos** | 📦 | Git repositories không giới hạn dung lượng (private), hỗ trợ pull request, branch policies, code search |
| **Azure Pipelines** | 🚀 | CI/CD cho mọi ngôn ngữ, nền tảng; chạy trên cloud hoặc on-premise; hỗ trợ YAML và classic editor |
| **Azure Boards** | 📋 | Quản lý công việc: backlog, sprint, Kanban board, liên kết work item với commit/PR/build |
| **Azure Artifacts** | 📚 | NuGet, npm, Maven, Python (PyPI), Universal packages feed cho tổ chức; quản lý version nội bộ |
| **Azure Test Plans** | 🧪 | Test case management, manual/exploratory testing, liên kết kết quả test với pipeline |

Các thành phần liên kết chặt chẽ với nhau. Ví dụ: một work item trên Boards (ví dụ `#123`) được nhắc trong commit message thì tự động liên kết; PR trên Repos có thể gắn build validation từ Pipelines; kết quả test từ Pipelines hiển thị trên Test Plans.

**Vì sao chọn Azure Pipelines?**

- Native với hệ sinh thái Microsoft/Azure (deploy lên Azure chỉ vài dòng YAML).
- YAML lưu cùng code → version control và code review cho chính pipeline.
- Hỗ trợ mọi ngôn ngữ; agents Microsoft-hosted miễn phí cho open source, có free tier cho private repo.
- Parallel jobs, environments với approval, tích hợp sâu với GitHub.

---

## 2. Tạo Organization, Project và kết nối Repo

### 2.1. Tạo Organization

1. Đăng nhập `https://dev.azure.com` bằng tài khoản Microsoft → **Start free** → đặt tên organization (ví dụ `mycompany`, chỉ chữ/số/gạch ngang, tối đa 50 ký tự).
2. Organization chứa các **Project**; mỗi project chứa repos, pipelines, boards riêng. Lưu ý: không đổi được region sau khi tạo.

### 2.2. Tạo Project

Bấm **New project** và điền: **Project name** (`webapp-demo`), **Visibility** (`Private`, hoặc `Public` để nhận free unlimited build minutes), **Version control** (`Git`), **Work item process** (`Agile` phổ biến nhất).

### 2.3. Kết nối Repo

Khi tạo pipeline (**Pipelines → New pipeline**), chọn nơi chứa code:

| Nguồn | Ghi chú |
|---|---|
| **Azure Repos Git** | Repo trong project, không cần cấu hình thêm |
| **GitHub** | Authorize OAuth; hỗ trợ cả GitHub Enterprise |
| **GitHub Enterprise Server** | Cần service connection với PAT |
| **Bitbucket Cloud** | Qua OAuth |
| **Other Git** | Repo Git bất kỳ qua URL + credentials |

Với GitHub, Azure DevOps tự tạo webhook nhận sự kiện push/PR; file YAML có thể nằm ngay trên repo GitHub.

---

## 3. azure-pipelines.yml: pipeline đầu tiên

### 3.1. Tạo pipeline từ giao diện

**Pipelines → New pipeline → Azure Repos Git** (hoặc GitHub) → chọn repo → **Starter pipeline**:

```yaml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'

- script: |
    echo Add other tasks to build, test, and deploy your project.
    echo See https://aka.ms/yaml
  displayName: 'Run a multi-line script'
```

Bấm **Save and run** → commit lên `main` (hoặc tạo branch mới để mở PR) → pipeline chạy ngay.

### 3.2. Đọc kết quả build

Bấm vào build number để xem **Summary** (tổng quan, artifacts), **Jobs** (log chi tiết từng step theo thời gian thực, tải về được để debug). Các khái niệm variables/predefined variables xem chi tiết ở mục 4.5 và 4.8.

---
## 4. Cú pháp YAML đầy đủ

Cấu trúc phân cấp: **pipeline → stages → jobs → steps**.

### 4.1. Sơ đồ phân cấp

```yaml
trigger: ...          # CI trigger (khi nào chạy)
pr: ...               # PR trigger
schedules: ...        # lịch chạy định kỳ
parameters: ...       # tham số nhập lúc chạy tay (runtime parameters)

variables: ...        # biến cấp pipeline
  group: ...          # variable group

pool: ...             # agent pool mặc định
stages:
- stage: Build
  displayName: 'Build stage'
  dependsOn: []       # mặc định stage chạy tuần tự; [] để chạy song song
  condition: ...      # điều kiện chạy
  variables: ...      # biến cấp stage
  jobs:
  - job: BuildJob
    pool: ...         # có thể override pool theo job
    strategy: ...     # matrix / parallel
    steps:
    - script: ...
    - task: ...
    - template: ...
```

### 4.2. trigger (CI trigger)

```yaml
trigger:
  branches:
    include:
    - main
    - release/*
    exclude:
    - main-experimental
  paths:
    include:
    - src/*
    exclude:
    - docs/*
  batch: true
```

- `trigger: none` → tắt CI (chỉ chạy tay hoặc theo lịch).
- Mặc định khi không khai báo `trigger`, pipeline chạy với mọi push lên mọi branch (trừ khi repo là GitHub — mặc định cũng tương tự).

### 4.3. pool

```yaml
pool:
  vmImage: ubuntu-latest
# hoặc
pool:
  name: 'MySelfHostedPool'
  demands:
  - npm                     # yêu cầu capability
```

Các `vmImage` phổ biến: `ubuntu-latest`, `windows-latest`, `macos-latest` (có thể pin version cụ thể như `ubuntu-22.04`).

### 4.4. stages, jobs, steps

```yaml
stages:
- stage: Build
  displayName: '🏗️ Build'
  jobs:
  - job: BuildJob
    displayName: 'Build job'
    steps:
    - script: dotnet build --configuration Release
      displayName: 'Build solution'

- stage: Deploy
  displayName: '🚀 Deploy'
  dependsOn: Build
  condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
  jobs:
  - deployment: DeployWeb
    displayName: 'Deploy web app'
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo Deploying...
```

- **Stage**: nhóm logic lớn (Build → Test → Deploy), chạy tuần tự theo thứ tự khai báo.
- **Job**: đơn vị chạy trên **một agent**; các job trong cùng stage mặc định chạy **song song**.
- **Deployment job** (`deployment:`): job đặc biệt gắn với `environment`, hỗ trợ strategies (runOnce, rolling, canary).
- **Step**: lệnh nhỏ nhất, có 4 loại:

| Loại step | Cú pháp | Dùng khi |
|---|---|---|
| `script` | `- script: echo hi` | Chạy shell (bash/cmd/powershell tùy OS agent) |
| `bash` / `pwsh` / `powershell` | `- bash: ls -la` | Ép shell cụ thể, đa nền tảng rõ ràng |
| `task` | `- task: NodeTool@0` | Dùng task có sẵn của Azure DevOps |
| `template` | `- template: steps/test.yml` | Tái sử dụng YAML |

### 4.5. variables

Ba nơi khai báo, độ ưu tiên tăng dần: pipeline → stage → job.

```yaml
variables:
  buildConfiguration: 'Release'      # literal
  isMain: $[eq(variables['Build.SourceBranch'], 'refs/heads/main')]  # runtime expression
  mySecret: $(secretFromGroup)       # tham chiếu variable group

steps:
- script: echo $(buildConfiguration)
```

Ba cú pháp template expression cần phân biệt:

| Cú pháp | Tên | Tính lúc | Ví dụ |
|---|---|---|---|
| `$(var)` | Macro | Runtime, trước khi step chạy | `$(Build.BuildNumber)` |
| `${{ variables.var }}` | Template | Compile-time (lúc parse YAML) | `${{ variables.isMain }}` |
| `$[ ... ]` | Runtime expression | Runtime, trong `condition` | `$[eq(variables['Build.SourceBranch'], 'refs/heads/main')]` |

**Secret variables** (`isSecret: true` hoặc từ Key Vault): giá trị bị che (`***`) trong log.

```yaml
variables:
- name: dbPassword
  value: 'P@ssw0rd'
  isSecret: true
```

### 4.6. parameters (runtime parameters)

Cho phép nhập tham số khi bấm **Run pipeline** thủ công:

```yaml
parameters:
- name: environment
  displayName: 'Môi trường deploy'
  type: string
  default: staging
  values:
  - staging
  - production
- name: runTests
  type: boolean
  default: true
- name: deployImage
  type: string
  default: 'myapp:latest'

stages:
- stage: Deploy
  jobs:
  - job: Deploy
    steps:
    - script: echo Deploying ${{ parameters.deployImage }} to ${{ parameters.environment }}
```

> `parameters` chỉ dùng được với `${{ }}` (compile-time). Không dùng `$( )` cho parameters.

### 4.7. conditions

Điều khiển step/job/stage có chạy hay không:

```yaml
steps:
- script: echo "Chỉ chạy khi build thành công"
  condition: succeeded()                 # mặc định

- script: echo "Chạy cả khi step trước fail"
  condition: always()

- script: echo "Chạy khi có fail nhưng không bị cancel"
  condition: failed()

- script: echo "Chạy khi bị cancel"
  condition: cancelled()

- script: npm run e2e
  condition: and(succeeded(), eq('${{ parameters.runTests }}', 'true'))
```

Hàm hay dùng: `succeeded()`, `failed()`, `always()`, `cancelled()`, `and/or/not()`, `eq/ne()`, `contains()`, `startsWith()`.

### 4.8. Predefined variables hay dùng

| Variable | Ý nghĩa |
|---|---|
| `$(Build.BuildNumber)` | Số build, ví dụ `20261010.3` |
| `$(Build.BuildId)` | ID duy nhất của build |
| `$(Build.SourceBranch)` | Ref đầy đủ, ví dụ `refs/heads/main` |
| `$(Build.SourceBranchName)` | Tên branch ngắn, ví dụ `main` |
| `$(Build.SourceVersion)` | Commit SHA trigger build |
| `$(Build.ArtifactStagingDirectory)` | Thư mục staging artifact (`a` trong work dir) |
| `$(Build.SourcesDirectory)` | Thư mục chứa source code checkout (`s`) |
| `$(Build.Repository.Name)` | Tên repo |
| `$(System.DefaultWorkingDirectory)` | Giống `Build.SourcesDirectory` |
| `$(System.TeamProject)` | Tên project |
| `$(Agent.OS)` | Hệ điều hành agent (`Linux`, `Windows_NT`, `Darwin`) |

Đổi định dạng build number:

```yaml
name: $(BuildDefinitionName)_$(date:yyyyMMdd)$(rev:.r)
# ví dụ: webapp-CI_20261010.3
```

---

## 5. Triggers: CI, PR và Schedule

### 5.1. CI trigger (push)

Đã trình bày ở mục 4.2. Một số pattern nâng cao:

```yaml
# Chỉ build khi code trong src/ thay đổi, bỏ qua docs
trigger:
  branches:
    include:
    - main
    - feature/*
  paths:
    include:
    - src/
    - tests/
    exclude:
    - '*.md'

# Monorepo: pipeline riêng cho từng service
# File: pipelines/service-a.yml
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - services/service-a/*
```

> `batch: true`: gom nhiều push thành một build duy nhất khi build trước chưa xong — tránh hàng đợi dài ở repo commit dày.

Tắt CI hoàn toàn:

```yaml
trigger: none
```

### 5.2. PR trigger (pull request validation)

```yaml
pr:
  branches:
    include:
    - main
  paths:
    include:
    - src/*
    exclude:
    - docs/*
```

- Khi mở/cập nhật PR vào `main`, pipeline tự chạy và báo status check về PR.
- Kết hợp **Branch policy** (Build Validation) để bắt buộc build pass mới được merge — cách chuẩn bảo vệ branch `main`.
- `pr: none` để tắt.

Ví dụ policy hoàn chỉnh cho branch `main`:

Bật: minimum reviewers (1–2), linked work items required, và **Build validation** với pipeline `PR-Validation` ở chế độ Required.

### 5.3. Schedule trigger (cron)

```yaml
schedules:
- cron: '0 2 * * *'          # 2h sáng mỗi ngày (UTC)
  displayName: 'Nightly build'
  branches:
    include:
    - main
  always: true               # chạy cả khi không có code change
```

- Giờ cron luôn tính theo **UTC**.
- `always: false` (mặc định): chỉ chạy nếu có code change từ lần chạy trước.
- Schedule trigger chạy trên branch mặc định được liệt kê; có thể dùng kèm `trigger: none` để pipeline **chỉ** chạy theo lịch.

### 5.4. Kết hợp nhiều trigger

```yaml
trigger:
- main            # CI khi push lên main

pr:
- main            # Validate PR vào main

schedules:
- cron: '0 3 * * *'
  displayName: 'Nightly'
  branches:
    include:
    - main
```

---
## 6. Variables và Variable Groups

### 6.1. Variable trong pipeline vs Variable Group

| | Pipeline variables | Variable Group |
|---|---|---|
| Phạm vi | 1 pipeline | Nhiều pipeline dùng chung |
| Quản lý | Trong YAML | Giao diện web (Pipelines → Library) |
| Secret | `isSecret: true` | Đánh dấu 🔒 từng biến |
| Key Vault | Không | Link trực tiếp, tự đồng bộ secret |

### 6.2. Tạo Variable Group

1. **Pipelines → Library → + Variable group**.
2. Đặt tên, ví dụ `webapp-prod-vars`.
3. Thêm variables:
   - `ApiBaseUrl` = `https://api.example.com` (thường)
   - `DbPassword` = `...` → bấm 🔒 để đánh dấu secret (sau khi lưu sẽ không xem/sửa lại được giá trị, chỉ có thể xóa và tạo mới).
4. (Tùy chọn) Giới hạn **Pipeline permissions**: chỉ cho phép các pipeline cụ thể dùng group này.

Dùng trong YAML:

```yaml
variables:
- group: webapp-prod-vars        # tất cả biến trong group khả dụng qua $(TenBien)

steps:
- script: |
    echo "API: $(ApiBaseUrl)"
    echo "DB password is hidden: $(DbPassword)"   # log hiện ***
```

> Secret variable KHÔNG tự map thành environment variable. Muốn dùng trong script phải map thủ công:

```yaml
steps:
- script: ./deploy.sh --password $DB_PASSWORD
  env:
    DB_PASSWORD: $(DbPassword)     # map secret → env var
```

### 6.3. Link Variable Group với Azure Key Vault

Cách chuẩn cho production: secret nằm trong Key Vault, pipeline tự lấy lúc chạy.

**Bước 1** — Tạo Key Vault (ví dụ `kv-webapp-prod`) và thêm secret trên Azure Portal.
**Bước 2** — Cấp quyền **Get/List** secrets cho service principal của Azure DevOps (RBAC: role **Key Vault Secrets User**).
**Bước 3** — Tạo service connection loại Azure Resource Manager (mục 7).
**Bước 4** — Trong Library, tạo Variable group → bật **Link secrets from an Azure key vault as variables** → chọn subscription, Key Vault, service connection → **Add** secret → Save.

```yaml
variables:
- group: kv-webapp-prod-linked   # secret tự refresh mỗi lần pipeline chạy

steps:
- script: ./migrate.sh
  env:
    DB_PASSWORD: $(DbPassword)   # giá trị mới nhất từ Key Vault
```

Ưu điểm: xoay secret trong Key Vault không cần sửa pipeline; audit log đầy đủ bên Key Vault.

---

## 7. Service Connections

**Service connection** là "cầu nối xác thực" đã lưu giữa Azure DevOps và hệ thống ngoài (Azure, DockerHub, GitHub, Kubernetes...). Pipeline tham chiếu bằng tên, không nhúng credential vào YAML.

### 7.1. Tạo Service Connection tới Azure (ARM)

**Project settings → Service connections → New → Azure Resource Manager → Service principal (automatic)** → chọn Subscription/Resource group → đặt tên `azure-prod` → **Save** (tự tạo service principal trong Entra ID).

Dùng trong pipeline:

```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'azure-prod'     # tên service connection
    appName: 'my-webapp'
    package: '$(Pipeline.Workspace)/drop/*.zip'
```

### 7.2. Tạo Service Connection tới Docker Hub / ACR

**Docker Registry**: chọn **Docker Hub** (nhập username/token, tên `dockerhub-conn`) hoặc **Azure Container Registry** (chọn subscription/registry, tên `acr-conn`).

```yaml
- task: Docker@2
  inputs:
    containerRegistry: 'acr-conn'
    command: 'buildAndPush'
    repository: 'myapp/web'
    tags: '$(Build.BuildId)'
```

### 7.3. Service Connection tới GitHub

**Project settings → Service connections → New → GitHub** → authorize OAuth hoặc nhập PAT. Dùng khi checkout thêm repo GitHub khác, hoặc pipeline ở Azure Repos cần đọc repo GitHub:

Ví dụ checkout nhiều repo và dùng template từ repo GitHub khác xem ở mục 13.3.

### 7.4. Phân quyền và bảo mật service connection

- Mặc định cần cấp quyền từng pipeline: vào **... → Security** để cấp (tránh mở "allow all" nếu không cần).
- Tạo connection riêng cho từng môi trường (`azure-dev`/`azure-staging`/`azure-prod`).
- Có thể gắn **Approvals and checks** lên service connection để duyệt tay trước khi pipeline production dùng.

---

## 8. Agents: Microsoft-hosted và Self-hosted

**Agent** là máy thực thi các job. Azure Pipelines có hai loại:

### 8.1. Microsoft-hosted agents

Microsoft cung cấp sẵn, mỗi lần chạy là máy ảo sạch.

```yaml
pool:
  vmImage: ubuntu-latest
```

| vmImage | Ghi chú |
|---|---|
| `ubuntu-latest` (→ 24.04) | Phổ biến nhất, build đa số stack |
| `windows-latest` (→ Server 2025) | Build .NET Framework, WPF, WinForms |
| `macos-latest` | Build iOS/macOS (Xcode) |

Ưu điểm: không bảo trì, có sẵn nhiều tool (`github.com/actions/runner-images`). Nhược điểm: mỗi build cài lại dependencies, không vào được mạng nội bộ, có thể chờ queue khi hết parallel job.

> Pin version cụ thể (`ubuntu-22.04`) cho pipeline production để tránh bị "lên đời" image bất ngờ gây fail build.

### 8.2. Self-hosted agent: cài đặt step-by-step

Dùng khi cần: mạng nội bộ/VPN, phần cứng đặc biệt (GPU), cache dependencies, hoặc tiết kiệm phút build.

**Bước 1 — Tạo Personal Access Token (PAT)**

1. Vào `dev.azure.com/{org}` → bấm avatar → **Personal access tokens → New Token**.
2. Name: `agent-pool`, Organization: chọn org, Scopes: **Agent Pools → Read & manage**.
3. Copy token (chỉ hiện 1 lần).

**Bước 2 — Tạo Agent Pool**

**Project settings → Agent pools → Add pool** → đặt tên `MySelfHosted` → chọn **Self-hosted**.

**Bước 3 — Tải và cài agent (Linux)**

```bash
# 1. Tạo thư mục và tải agent (lấy URL đúng trong pool → New agent)
mkdir -p ~/azagent && cd ~/azagent
curl -fsSL https://vstsagentpackage.azureedge.net/agent/4.248.0/vsts-agent-linux-x64-4.248.0.tar.gz -o agent.tar.gz
tar zxvf agent.tar.gz

# 2. Cài dependencies
sudo ./bin/installdependencies.sh

# 3. Cấu hình (trả lời interactive hoặc truyền tham số)
./config.sh --unattended \
  --url https://dev.azure.com/mycompany \
  --auth pat --token <PAT> \
  --pool MySelfHosted \
  --agent my-linux-01 \
  --acceptTeeEula \
  --work _work \
  --runAsService

# 4. Cài service và chạy
sudo ./svc.sh install
sudo ./svc.sh start

# Kiểm tra trạng thái
sudo ./svc.sh status
```

**Trên Windows (PowerShell, chạy as Administrator):**

```powershell
mkdir C:\azagent; cd C:\azagent
# Tải vsts-agent-win-x64-*.zip và giải nén
.\config.cmd --unattended --url https://dev.azure.com/mycompany --auth pat --token <PAT> `
  --pool MySelfHosted --agent my-win-01 --acceptTeeEula --runAsService
.\svc.sh install  # thực chất: .\config.cmd đã hỏi cài service; hoặc chạy service thủ công
```

**Bước 4 — Dùng trong pipeline**

```yaml
pool:
  name: MySelfHosted
  demands:
  - Agent.OS -equals Linux     # chọn agent cụ thể nếu cần
```

**Bước 5 — Gỡ agent (khi không dùng nữa)**

```bash
sudo ./svc.sh stop
sudo ./svc.sh uninstall
./config.sh remove --auth pat --token <PAT>
```

### 8.3. So sánh nhanh

| Tiêu chí | Microsoft-hosted | Self-hosted |
|---|---|---|
| Chi phí | Tính theo phút (có free tier) | Miễn phí phút build, tốn tiền máy |
| Bảo trì | Không | Tự cập nhật OS, agent, tools |
| Mạng nội bộ | Không | Có |
| Khởi động | ~30–60s provision | Gần như tức thì (máy luôn bật) |
| Bảo mật | Môi trường sạch mỗi build | Tự chịu trách nhiệm dọn dẹp |

### 8.4. Agent diagnostics hay dùng

```yaml
steps:
- script: |
    echo "Agent: $(Agent.Name) / $(Agent.OS)"
    echo "Work dir: $(Agent.WorkFolder)"
    df -h                    # kiểm tra disk (lỗi hết disk rất phổ biến)
    env | sort               # xem biến môi trường
  displayName: 'Agent diagnostics'
```

---
## 9. Các Task phổ biến

Task là "khối xây dựng" đóng gói sẵn. Cú pháp chung:

```yaml
- task: TenTask@version
  displayName: 'Mô tả hiển thị'
  inputs:
    thamSo1: 'giá trị'
  condition: succeeded()     # tùy chọn
  continueOnError: false     # tùy chọn: true = fail vẫn chạy tiếp
```

### 9.1. UseNode@1 — cài Node.js

```yaml
- task: UseNode@1
  inputs:
    version: '20.x'          # hoặc '18.20.4', 'lts/*'
  displayName: 'Cài Node.js 20'
```

### 9.2. UsePythonVersion@0 — cài Python

```yaml
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.12'
    addToPath: true
  displayName: 'Cài Python 3.12'

- script: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
    pytest tests/ --junitxml=test-results.xml
  displayName: 'Cài deps và chạy test'
```

### 9.3. Docker@2 — build/push/login

```yaml
# Login + build + push trong 1 task
- task: Docker@2
  displayName: 'Build & push image'
  inputs:
    containerRegistry: 'acr-conn'       # service connection
    repository: 'myapp/web'
    command: 'buildAndPush'
    Dockerfile: 'src/Dockerfile'
    tags: |
      $(Build.BuildId)
      latest
```

### 9.4. PublishBuildArtifacts@1 / DownloadBuildArtifacts@1

```yaml
- task: PublishBuildArtifacts@1
  displayName: 'Publish artifact'
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'     # lưu trên Azure DevOps (mặc định)
```

```yaml
- task: DownloadBuildArtifacts@1
  displayName: 'Download artifact'
  inputs:
    buildType: 'current'            # 'current' | 'specific'
    downloadType: 'single'
    artifactName: 'drop'
    downloadPath: '$(Pipeline.Workspace)'
```

> Từ 2024, Microsoft khuyến nghị cặp task mới **`PublishPipelineArtifact@1` / `DownloadPipelineArtifact@2`** (tốc độ cao hơn, tích hợp tốt hơn với multi-stage). Xem chi tiết ở mục 10.

### 9.5. Các task build theo ngôn ngữ

```yaml
# .NET
- task: UseDotNet@2
  inputs:
    version: '8.x'
- task: DotNetCoreCLI@2
  inputs:
    command: 'build'
    projects: '**/*.csproj'
    arguments: '--configuration Release'
- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    projects: '**/*Tests.csproj'
    arguments: '--configuration Release --collect "XPlat Code Coverage"'

# npm
- task: Npm@1
  inputs:
    command: 'ci'                 # npm ci: sạch và nhanh hơn npm install
    workingDir: 'frontend'
```

### 9.6. Task tiện ích hay dùng

```yaml
# Chạy Azure CLI (đã login bằng service connection)
- task: AzureCLI@2
  inputs:
    azureSubscription: 'azure-prod'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az webapp restart --name my-webapp --resource-group rg-prod

# Publish kết quả test để hiện tab Tests
- task: PublishTestResults@2
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: '**/test-results.xml'
```

---

## 10. Artifacts: publish và download giữa các stages

Trong multi-stage pipeline, **stage Build** tạo ra file (zip, dll, image tag...) và **stage Deploy** cần dùng — cầu nối là **Pipeline Artifacts**.

### 10.1. Publish từ stage Build

```yaml
stages:
- stage: Build
  jobs:
  - job: BuildJob
    steps:
    - script: npm ci && npm run build
      workingDirectory: frontend

    - task: CopyFiles@2
      inputs:
        SourceFolder: 'frontend/dist'
        Contents: '**'
        TargetFolder: '$(Build.ArtifactStagingDirectory)/web'

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Build.ArtifactStagingDirectory)/web'
        artifact: 'webapp'          # tên artifact
```

### 10.2. Download ở stage Deploy

```yaml
- stage: Deploy
  dependsOn: Build
  jobs:
  - deployment: DeployWeb
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: DownloadPipelineArtifact@2
            inputs:
              artifact: 'webapp'
              path: '$(Pipeline.Workspace)/webapp'

          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'azure-staging'
              appName: 'my-webapp-staging'
              package: '$(Pipeline.Workspace)/webapp'
```

### 10.3. So sánh hai họ task artifact

| | Build Artifacts (cũ) | Pipeline Artifacts (mới, nên dùng) |
|---|---|---|
| Task | `PublishBuildArtifacts@1` | `PublishPipelineArtifact@1` |
| Tốc độ | Chậm hơn | Nhanh hơn (deduplication, song song) |
| Multi-stage | Dùng được nhưng cồng kềnh | Thiết kế cho multi-stage |
| Download mặc định | Phải gọi task download | Deployment job **tự download** artifact của stage trước (nếu cùng pipeline) |

> Trong deployment job, artifact từ các stage `dependsOn` được tự động tải về `$(Pipeline.Workspace)` — thường không cần gọi `DownloadPipelineArtifact` thủ công.

### 10.4. Artifact retention

Artifact giữ theo **retention policy** của project (thường 30 ngày); chỉnh ở **Project settings → Pipelines → Settings**. Bấm **Retain** để giữ vô thời hạn bản release quan trọng.

---

## 11. Environments: approvals, checks và deployment history

**Environment** là "đích deploy" logic (dev/staging/production), gom các tài nguyên thật (VM, Kubernetes namespace, App Service) và là nơi gắn **quy tắc kiểm soát** trước khi deploy.

### 11.1. Tạo Environment

1. **Pipelines → Environments → New environment** → tên `production` → Resource: None (hoặc Kubernetes/VM nếu cần).
2. Trong YAML, deployment job trỏ tới environment:

```yaml
- stage: DeployProd
  jobs:
  - deployment: DeployProd
    environment: 'production'     # tạo tự động nếu chưa có (nhưng nên tạo tay để gắn checks)
    strategy:
      runOnce:
        deploy:
          steps:
          - script: echo "Deploying to production"
```

### 11.2. Approvals and checks

Vào **Environment → ... → Approvals and checks → +** :

| Check | Tác dụng |
|---|---|
| **Approvals** | Yêu cầu 1+ người bấm Approve trước khi deploy chạy; có thể set timeout, cho phép approver tự approve build của mình hay không |
| **Branch control** | Chỉ cho deploy từ branch cụ thể (ví dụ chỉ `main` được deploy lên production) |
| **Business hours** | Chỉ deploy trong khung giờ hành chính |
| **Invoke Azure Function** | Gọi function để kiểm tra điều kiện động (ví dụ: check monitoring) rồi mới cho deploy |
| **Query Azure Monitor alerts** | Chặn deploy nếu đang có alert cháy |
| **Required template** | Bắt buộc pipeline dùng template đã duyệt |

Ví dụ YAML không cần khai báo gì thêm — checks tự áp dụng khi deployment job nhắm tới environment:

```yaml
- stage: DeployProd
  dependsOn: DeployStaging
  condition: succeeded()
  jobs:
  - deployment: Prod
    displayName: 'Deploy production (cần approve)'
    environment: 'production'     # đã gắn Approval: team-leads
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'azure-prod'
              appName: 'my-webapp'
              package: '$(Pipeline.Workspace)/webapp'
```

Luồng chạy: stage bắt đầu → pipeline **tạm dừng** chờ approve → người được chỉ định bấm **Approve** (nhận email thông báo) → deploy tiếp tục. Lịch sử approve/reject được lưu đầy đủ.

### 11.3. Deployment history

Tab **Deployments** của mỗi environment ghi lại: build nào deploy, ai approve, thời gian, trạng thái, commit — audit trail quan trọng cho compliance.

### 11.4. Kết hợp environments cho pipeline chuẩn 3 môi trường

```yaml
stages:
- stage: Build           # build + publish artifact 'webapp'
- stage: DeployDev       # environment: dev (không approval → deploy tự động)
  dependsOn: Build
- stage: DeployStaging   # environment: staging (approval: QA team)
  dependsOn: DeployDev
- stage: DeployProd      # environment: production (approval: team lead + branch control)
  dependsOn: DeployStaging
```

Ví dụ đầy đủ từng bước deploy (kèm smoke test, slot swap) xem ở mục 14.2.

---
## 12. Deployment Strategies

Khai báo trong `deployment` job, tại `strategy`. Ba chiến lược có sẵn:

### 12.1. runOnce (mặc định, đơn giản nhất)

Deploy một lần duy nhất tới toàn bộ target. Phù hợp cho dev/staging hoặc production quy mô nhỏ.

```yaml
strategy:
  runOnce:
    preDeploy:
      steps:
      - script: echo "Backup current version"
    deploy:
      steps:
      - task: AzureWebApp@1
        inputs:
          azureSubscription: 'azure-prod'
          appName: 'my-webapp'
          package: '$(Pipeline.Workspace)/webapp'
    postDeploy:
      steps:
      - script: curl -f https://my-webapp.azurewebsites.net/health
```

### 12.2. rolling (cuốn chiếu)

Triển khai lần lượt từng nhóm target (ví dụ VM trong VM scale set), mỗi nhóm xong mới sang nhóm tiếp theo. Nếu một nhóm fail → dừng lại, các nhóm chưa deploy giữ phiên bản cũ.

```yaml
jobs:
- deployment: RollingDeploy
  environment: 'prod-vms'
  strategy:
    rolling:
      maxParallel: 2
      preDeploy:
        steps:
        - script: echo "Chuẩn bị node"
      deploy:
        steps:
        - script: ./deploy-to-vm.sh
      routeTraffic:
        steps:
        - script: echo "Mở traffic cho batch vừa deploy"
      postRouteTraffic:
        steps:
        - script: ./smoke-test.sh
      on:
        failure:
          steps:
          - script: echo "Rollback batch lỗi"
        success:
          steps:
          - script: echo "Batch OK"
```

Thứ tự hook: `preDeploy` → `deploy` → `routeTraffic` → `postRouteTraffic`, kèm nhánh `on: failure/success`.

### 12.3. canary (chim hoàng yến)

Deploy phiên bản mới cho một tỉ lệ nhỏ target trước (ví dụ 10%), theo dõi, rồi mới mở rộng dần.

```yaml
strategy:
  canary:
    increments: [10, 50]
    preDeploy:
      steps:
      - script: echo "Canary: kiểm tra sức khỏe hệ thống"
    deploy:
      steps:
      - script: ./deploy-canary.sh $(increment)
    postRouteTraffic:
      steps:
      - script: ./verify-metrics.sh
    on:
      failure:
        steps:
        - script: ./rollback-canary.sh
```

> `rolling`/`canary` hiệu quả nhất với resource type **Virtual machines**. Với App Service/K8s, canary thường dùng deployment slots hoặc Argo Rollouts.

---

## 13. Templates: tái sử dụng YAML

Khi có nhiều pipeline dùng chung logic (build Node, scan bảo mật, deploy App Service...), tách thành **template** để tái sử dụng và quản lý tập trung.

### 13.1. Step template

`templates/steps/npm-build.yml`:

```yaml
parameters:
- name: workingDir
  type: string
  default: '.'
- name: nodeVersion
  type: string
  default: '20.x'

steps:
- task: UseNode@1
  inputs:
    version: '${{ parameters.nodeVersion }}'
- script: npm ci
  workingDirectory: '${{ parameters.workingDir }}'
  displayName: 'npm ci'
- script: npm run build
  workingDirectory: '${{ parameters.workingDir }}'
  displayName: 'npm run build'
```

Dùng trong pipeline:

```yaml
steps:
- template: templates/steps/npm-build.yml
  parameters:
    workingDir: 'frontend'
    nodeVersion: '20.x'
```

### 13.2. Job và stage template

`templates/jobs/deploy-appservice.yml`:

```yaml
parameters:
- name: environment
  type: string
- name: appName
  type: string
- name: serviceConnection
  type: string
- name: artifact
  type: string
  default: 'webapp'

jobs:
- deployment: Deploy_${{ parameters.environment }}
  displayName: 'Deploy ${{ parameters.environment }}'
  environment: '${{ parameters.environment }}'
  strategy:
    runOnce:
      deploy:
        steps:
        - task: AzureWebApp@1
          inputs:
            azureSubscription: '${{ parameters.serviceConnection }}'
            appName: '${{ parameters.appName }}'
            package: '$(Pipeline.Workspace)/${{ parameters.artifact }}'
```

`azure-pipelines.yml`:

```yaml
stages:
- stage: Build
  jobs:
  - job: Build
    steps:
    - template: templates/steps/npm-build.yml
      parameters:
        workingDir: 'frontend'
    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: 'frontend/dist'
        artifact: 'webapp'

- stage: DeployDev
  dependsOn: Build
  jobs:
  - template: templates/jobs/deploy-appservice.yml
    parameters:
      environment: 'dev'
      appName: 'my-webapp-dev'
      serviceConnection: 'azure-dev'

- stage: DeployProd
  dependsOn: DeployDev
  jobs:
  - template: templates/jobs/deploy-appservice.yml
    parameters:
      environment: 'production'
      appName: 'my-webapp'
      serviceConnection: 'azure-prod'
```

### 13.3. Template từ repository khác

```yaml
resources:
  repositories:
  - repository: templates
    type: git
    name: PlatformTeam/pipeline-templates   # repo Azure Repos khác trong cùng project/org
    ref: refs/tags/v1.2.0                   # pin version để ổn định

steps:
- template: steps/npm-build.yml@templates
  parameters:
    workingDir: 'frontend'
```

### 13.4. Required template check (governance)

Gắn check **Required template** lên environment: pipeline chỉ deploy được nếu `extends` đúng template đã duyệt — cách platform team kiểm soát chuẩn deploy toàn tổ chức:

```yaml
# templates/base-deploy.yml (template được duyệt)
parameters:
- name: stages
  type: stageList
stages:
- ${{ each stage in parameters.stages }}:
  - ${{ stage }}
- stage: ComplianceCheck      # bước bắt buộc mọi pipeline phải có
  jobs:
  - job: Scan
    steps:
    - script: ./security-scan.sh
```

```yaml
# azure-pipelines.yml của team
extends:
  template: templates/base-deploy.yml@templates
  parameters:
    stages:
    - stage: Build
      jobs:
      - job: Build
        steps:
        - script: npm run build
```

---
## 14. Ví dụ thực tế hoàn chỉnh

### 14.1. CI cho .NET 8 Web API

`azure-pipelines.yml` — build, test (kèm coverage), publish artifact:

```yaml
trigger:
  branches:
    include:
    - main
    - release/*
  paths:
    exclude:
    - docs/*

pr:
  branches:
    include:
    - main

name: $(BuildDefinitionName)_$(date:yyyyMMdd)$(rev:.r)

pool:
  vmImage: ubuntu-latest

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '8.x'

stages:
- stage: Build
  displayName: '🏗️ Build & Test'
  jobs:
  - job: BuildTest
    displayName: 'Build, test, publish'
    steps:
    - task: UseDotNet@2
      displayName: 'Cài .NET $(dotnetVersion)'
      inputs:
        version: '$(dotnetVersion)'

    - task: DotNetCoreCLI@2
      displayName: 'Restore'
      inputs:
        command: 'restore'
        projects: '**/*.csproj'

    - task: DotNetCoreCLI@2
      displayName: 'Build'
      inputs:
        command: 'build'
        projects: '**/*.csproj'
        arguments: '--configuration $(buildConfiguration) --no-restore'

    - task: DotNetCoreCLI@2
      displayName: 'Test + coverage'
      inputs:
        command: 'test'
        projects: '**/*Tests.csproj'
        arguments: '--configuration $(buildConfiguration) --no-build --collect "XPlat Code Coverage" --logger trx'
        publishTestResults: true

    - task: PublishCodeCoverageResults@2
      displayName: 'Publish coverage'
      inputs:
        summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'

    - task: DotNetCoreCLI@2
      displayName: 'Publish app'
      inputs:
        command: 'publish'
        projects: 'src/WebApi/WebApi.csproj'
        arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)/app --no-build'
        zipAfterPublish: true

    - task: PublishPipelineArtifact@1
      displayName: 'Publish artifact'
      inputs:
        targetPath: '$(Build.ArtifactStagingDirectory)/app'
        artifact: 'webapi'
```

### 14.2. CI Node.js + CD lên Azure App Service (3 môi trường)

```yaml
trigger:
  branches:
    include:
    - main
pr:
  branches:
    include:
    - main

pool:
  vmImage: ubuntu-latest

variables:
- group: webapp-common          # chứa NODE_ENV chung, APP_NAME_PREFIX...
- name: nodeVersion
  value: '20.x'

stages:
- stage: Build
  displayName: '🏗️ Build'
  jobs:
  - job: Build
    steps:
    - task: UseNode@1
      inputs:
        version: '$(nodeVersion)'

    - script: npm ci
      workingDirectory: frontend
      displayName: 'npm ci'

    - script: npm run lint
      workingDirectory: frontend
      displayName: 'ESLint'

    - script: npm test -- --watchAll=false --reporters=default --reporters=jest-junit
      workingDirectory: frontend
      displayName: 'Jest tests'
      continueOnError: false

    - task: PublishTestResults@2
      condition: succeededOrFailed()
      inputs:
        testResultsFormat: 'JUnit'
        testResultsFiles: 'frontend/junit.xml'

    - script: npm run build
      workingDirectory: frontend
      displayName: 'Build production'

        - task: ArchiveFiles@2
      inputs:
        rootFolderOrFile: 'frontend/dist'
        includeRootFolder: false
        archiveType: 'zip'
        archiveFile: '$(Build.ArtifactStagingDirectory)/webapp.zip'

    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: '$(Build.ArtifactStagingDirectory)/webapp.zip'
        artifact: 'webapp'

- stage: DeployDev
  displayName: '🚀 Deploy DEV'
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: DeployDev
    environment: 'dev'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'azure-dev'
              appType: 'webAppLinux'
              appName: 'my-webapp-dev'
              runtimeStack: 'NODE|20-lts'
              package: '$(Pipeline.Workspace)/webapp/webapp.zip'
              startUpCommand: 'npx serve -s . -l 8080'

- stage: DeployStaging
  displayName: '🚀 Deploy STAGING'
  dependsOn: DeployDev
  condition: succeeded()
  jobs:
  - deployment: DeployStaging
    environment: 'staging'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'azure-staging'
              appType: 'webAppLinux'
              appName: 'my-webapp-staging'
              package: '$(Pipeline.Workspace)/webapp/webapp.zip'
          - script: |
              sleep 20
              curl -f https://my-webapp-staging.azurewebsites.net/health || exit 1
            displayName: 'Smoke test staging'

- stage: DeployProd
  displayName: '🚀 Deploy PRODUCTION'
  dependsOn: DeployStaging
  condition: succeeded()
  jobs:
  - deployment: DeployProd
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureWebApp@1
            inputs:
              azureSubscription: 'azure-prod'
              appType: 'webAppLinux'
              appName: 'my-webapp'
              slotName: 'staging'
              package: '$(Pipeline.Workspace)/webapp/webapp.zip'
          - script: curl -f https://my-webapp-staging.azurewebsites.net/health
            displayName: 'Smoke test slot'
          - task: AzureAppServiceManage@0
            displayName: 'Swap to production'
            inputs:
              azureSubscription: 'azure-prod'
              action: 'Swap Slots'
              webAppName: 'my-webapp'
              sourceSlot: 'staging'
              targetSlot: 'production'
```

### 14.3. Build Docker image và push lên Azure Container Registry

```yaml
trigger:
  branches:
    include:
    - main
  paths:
    include:
    - src/
    - Dockerfile

pool:
  vmImage: ubuntu-latest

variables:
  acrServiceConnection: 'acr-conn'
  imageRepository: 'myapp/api'
  dockerfilePath: 'src/Dockerfile'

stages:
- stage: Build
  displayName: '🏗️ Build image'
  jobs:
  - job: DockerBuild
    steps:
        - script: |
        docker run --rm -i hadolint/hadolint < $(dockerfilePath)
      displayName: 'Lint Dockerfile (hadolint)'
      continueOnError: true

    - task: Docker@2
      displayName: 'Build & push to ACR'
      inputs:
        containerRegistry: '$(acrServiceConnection)'
        repository: '$(imageRepository)'
        command: 'buildAndPush'
        Dockerfile: '$(dockerfilePath)'
        buildContext: '.'
        tags: |
          $(Build.BuildId)
          $(Build.SourceBranchName)-$(date:yyyyMMdd)

        - script: |
        ACR_LOGIN=$(az acr show --name myregistry --query loginServer -o tsv)
        docker run --rm aquasec/trivy image --severity HIGH,CRITICAL \
          $ACR_LOGIN/$(imageRepository):$(Build.BuildId)
      displayName: 'Trivy scan image'
      env:
        AZURE_DEVOPS_EXT_PAT: $(System.AccessToken)
      continueOnError: true

- stage: Deploy
  displayName: '🚀 Deploy to AKS'
  dependsOn: Build
  jobs:
  - deployment: DeployAKS
    environment: 'prod-aks'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: KubernetesManifest@1
            inputs:
              action: 'deploy'
              kubernetesServiceConnection: 'aks-prod'
              namespace: 'production'
              manifests: 'k8s/deployment.yaml'
              containers: 'myregistry.azurecr.io/$(imageRepository):$(Build.BuildId)'
```

> `$(System.AccessToken)` là token OAuth mà Azure DevOps tự cấp cho mỗi build — dùng để gọi API Azure DevOps hoặc `az` CLI mà không cần PAT riêng. Cần bật **Allow scripts to access the OAuth token** trong agent job settings (mặc định bật cho YAML).

---
## 15. Classic Release Pipelines

Trước khi YAML pipeline ra đời (2018), Azure DevOps dùng **Classic editor**: build pipeline và **Release pipeline** tách rời, cấu hình bằng giao diện kéo-thả.

### 15.1. Classic Release hoạt động thế nào

**Build pipeline** tạo artifact → **Release pipeline** lấy artifact đi qua các **stages** (Dev → QA → Prod) với **pre-deployment approvals**. Khái niệm tương đương YAML: release ≈ một lần chạy multi-stage pipeline; approvals ≈ environment approvals.

### 15.2. So sánh Classic Release vs YAML multi-stage

| Tiêu chí | Classic Release | YAML multi-stage (khuyến nghị) |
|---|---|---|
| Định nghĩa | Click trên giao diện web | File `azure-pipelines.yml` trong repo |
| Version control | Không (lưu trong DB của Azure DevOps) | Có — review, blame, rollback như code |
| Tái sử dụng | Task groups (hạn chế) | Templates + `extends` mạnh mẽ |
| Audit "ai đổi pipeline" | Khó truy vết | Git history rõ ràng |
| Tính năng mới của Microsoft | Đóng băng, không phát triển thêm | Mọi tính năng mới (environments, checks, strategies) đều ở YAML |
| Độ dễ tiếp cận | Dễ cho người mới (UI trực quan) | Cần học cú pháp YAML |

### 15.3. Khi nào vẫn gặp Classic Release

- Hệ thống legacy chưa migrate.
- Một số template marketplace cũ chỉ có bản classic.

**Lộ trình migrate**: export release definition làm tài liệu tham khảo → viết lại bằng YAML multi-stage + environments → chạy song song 1–2 release để đối chiếu → tắt classic.

> **Khuyến nghị dứt khoát**: mọi pipeline mới viết bằng **YAML**. Classic chỉ để duy trì hệ cũ.

---

## 16. Pricing

Azure DevOps tính phí theo hai trục: **Pipelines (parallel jobs)** và **Artifacts (dung lượng)**. Các thành phần khác (Repos, Boards, Test Plans cơ bản) miễn phí cho team nhỏ.

### 16.1. Azure Pipelines — parallel jobs

Một **parallel job** = một pipeline được chạy đồng thời. Hết job rảnh → build xếp hàng chờ.

| Gói | Public project | Private project |
|---|---|---|
| **Miễn phí** | 10 parallel jobs, **unlimited minutes** | 1 parallel job, **1.800 phút/tháng** |
| Mua thêm | — | ~$40 / parallel job / tháng (Microsoft-hosted) |
| Self-hosted | Unlimited minutes, **1 free parallel job** | Unlimited minutes, **1 free parallel job** (mua thêm ~$15/job/tháng) |

Điểm mấu chốt:

- **Self-hosted agent không tốn phút build**. 1.800 phút/tháng ≈ 30 giờ — đủ cho team nhỏ nếu tối ưu (cache, batch). Public project gần như không giới hạn.

### 16.2. Các giới hạn free tier cần nhớ

| Tài nguyên | Giới hạn free |
|---|---|
| Build minutes (private, MS-hosted) | 1.800 phút/tháng |
| Parallel jobs (private) | 1 |
| Artifacts storage | 2 GB miễn phí, sau đó ~$2/GB/tháng |
| Test Plans (có phí) | 30 ngày trial, sau đó ~$52/user/tháng (chỉ cho tester chuyên trách) |

### 16.3. Tối ưu chi phí

1. **Cache dependencies** (`Cache@2`): giảm 50–80% thời gian build.
2. **`batch: true`** + path filters: tránh build trùng lặp.
3. **Self-hosted agent** cho build nặng/lặp lại nhiều.
4. **Pin `vmImage`**; set retention hợp lý, đừng retain vô tội vạ.
5. Test nặng (E2E) chạy theo lịch đêm thay vì mỗi PR.

Ví dụ:

```yaml
- task: Cache@2
  inputs:
    key: 'npm | "$(Agent.OS)" | frontend/package-lock.json'
    restoreKeys: |
      npm | "$(Agent.OS)"
    path: frontend/node_modules
  displayName: 'Cache node_modules'
```

---
## 17. Troubleshooting: 15 lỗi thường gặp

### Lỗi 1 — `No hosted parallelism has been purchased or granted`

**Triệu chứng**: pipeline xếp hàng mãi không chạy, báo lỗi về parallelism.
**Nguyên nhân**: hết parallel job miễn phí (thường gặp ở org mới với private project — Microsoft yêu cầu verify để cấp free tier).
**Cách fix**: kiểm tra `dev.azure.com/{org}/_settings/billing`; org mới điền form xin cấp free parallelism tại `aka.ms/azpipelines-parallelism-request` (2–3 ngày làm việc); tạm thời dùng self-hosted agent.

### Lỗi 2 — YAML parse error: `Unexpected value`, `mapping values are not allowed`

**Nguyên nhân**: sai thụt đầu dòng, thiếu `:` hoặc dùng tab thay vì space. **Fix**: dùng VS Code + extension Azure Pipelines, bấm **Validate** trước khi save; lỗi phổ biến là `steps` đặt ngang hàng `jobs` thay vì thụt vào trong.

### Lỗi 3 — `$(variable)` không thay thế / ra chuỗi rỗng

**Nguyên nhân**: nhầm `${{ parameters.x }}` với `$(variables.x)` (parameters chỉ dùng `${{ }}`); dùng `$(var)` trong `condition` phải viết `eq(variables['var'], 'value')`; secret không tự thành env var (map qua `env:`, mục 6.2).

### Lỗi 4 — `There was a resource authorization issue`

**Triệu chứng**: pipeline fail ngay khi bắt đầu, báo không được phép dùng service connection / variable group / environment.
**Fix**: vào tài nguyên → **Security / Pipeline permissions** → thêm pipeline vào danh sách cho phép (tránh mở "Allow all pipelines" nếu không cần).

### Lỗi 5 — `The pipeline is not valid. Job ...: Step ... has an invalid template`

**Nguyên nhân**: sai đường dẫn template, hoặc thiếu `resources.repositories` khi template ở repo khác. **Fix**: kiểm tra đường dẫn tương đối từ gốc repo; repo GitHub ngoài cần `endpoint`.

### Lỗi 6 — Artifact `drop` không tìm thấy ở stage Deploy

**Nguyên nhân**: stage Deploy thiếu `dependsOn` stage Build; gõ sai tên artifact; trộn hai họ task (`PublishBuildArtifacts@1` vs `DownloadPipelineArtifact@2`). **Fix**: thống nhất dùng Pipeline Artifacts, kiểm tra `dependsOn` và tab **Artifacts** trong build summary.

### Lỗi 7 — Docker build fail: `no space left on device`

**Nguyên nhân**: agent chỉ có ~10GB trống, layer cache chiếm hết. **Fix** (chạy trước khi build Docker):
```yaml
- script: |
    docker system prune -af --volumes   # dọn trước khi build
    df -h
  displayName: 'Dọn disk trước khi build Docker'
```
Lâu dài: self-hosted agent disk lớn, hoặc dùng ACR Tasks (`az acr build`).

### Lỗi 8 — `npm ci` fail: `package-lock.json` không đồng bộ

**Nguyên nhân**: `package.json` và `package-lock.json` lệch nhau. **Fix**: chạy `npm install` ở local để regenerate lock file rồi commit; trong pipeline **luôn dùng `npm ci`**.

### Lỗi 9 — .NET build fail trên `ubuntu-latest` nhưng chạy được trên máy dev (Windows)

**Nguyên nhân**: path phân biệt hoa/thường, hoặc dùng API chỉ có trên Windows. **Fix**: sửa tên file/folder khớp hoàn toàn; .NET Framework bắt buộc `vmImage: windows-latest`.

### Lỗi 10 — Deploy lên App Service báo `ERROR_FILE_IN_USE` / 409

**Nguyên nhân**: file bị process giữ (hay gặp ở App Service Windows). **Fix**: bật **TakeAppOfflineFlag** / **RenameFilesFlag**:
```yaml
- task: AzureRmWebAppDeployment@4
  inputs:
    azureSubscription: 'azure-prod'
    appType: 'webApp'
    WebAppName: 'my-webapp'
    package: '$(Pipeline.Workspace)/webapp/webapp.zip'
    TakeAppOfflineFlag: true
    RenameFilesFlag: true
```

### Lỗi 11 — Approval treo: stage chờ mãi không ai duyệt

**Nguyên nhân**: approver không nhận email hoặc đã đổi team. **Fix**: vào build → tab **Checks** xem ai đang được chờ rồi nhắc/approve hộ; set **Timeout** cho approval (ví dụ 24h) để tự reject; định kỳ rà soát danh sách approver.

### Lỗi 12 — `TF401027: You need the Git 'GenericContribute' permission`

**Nguyên nhân**: build service account thiếu quyền push/tag (hay gặp khi pipeline tự tạo tag hoặc commit version bump). **Fix**: cấp **Contribute** cho `Project Collection Build Service ({org})` trong repo Security, hoặc trong YAML:
```yaml
steps:
- checkout: self
  persistCredentials: true
```

### Lỗi 13 — Schedule trigger không chạy

**Nguyên nhân**: `always: false` + không có code change → bị skip (đúng thiết kế); cron tính theo **UTC**; branch không tồn tại. **Fix**: set `always: true` nếu muốn chạy luôn; kiểm tra giờ UTC và **Scheduled runs**.

### Lỗi 14 — Self-hosted agent offline / không nhận job

**Checklist**: `sudo ./svc.sh status`; PAT hết hạn (tối đa 1 năm) → tạo mới, `./config.sh remove` rồi config lại; agent cũ → **Update all agents**; `demands` không khớp capability; firewall chặn `*.dev.azure.com:443`.

### Lỗi 15 — Pipeline chạy chậm bất thường

**Chẩn đoán**: xem thời gian từng step, tìm step ngốn thời gian. **Fix**: thêm `Cache@2`; chạy test song song; self-hosted agent gần Azure region; tách pipeline PR (nhẹ) khỏi nightly (nặng).

---
## 18. Best Practices Checklist

### Thiết kế pipeline

- [ ] Mọi pipeline mới viết bằng **YAML**, lưu cùng repo (`azure-pipelines.yml` ở gốc hoặc thư mục `pipelines/`).
- [ ] Tách pipeline **PR validation** (nhanh: build + unit test) khỏi pipeline **CD** (đầy đủ: E2E, scan, deploy).
- [ ] Dùng **path filters** cho monorepo để tránh build không cần thiết.
- [ ] Bật `batch: true` cho repo commit dày.
- [ ] Pin `vmImage` version cụ thể cho pipeline production (`ubuntu-22.04` thay vì `ubuntu-latest`).
- [ ] Dùng **templates** cho logic lặp lại; pin template repo ngoài bằng tag.

### Bảo mật

- [ ] Secret **không bao giờ** hardcode trong YAML — dùng variable group 🔒 hoặc Key Vault.
- [ ] Map secret sang `env:` khi dùng trong script; không `echo` secret ra log.
- [ ] Service connection theo môi trường (`azure-dev`/`azure-prod`), không dùng một connection quyền rộng.
- [ ] Bật **Pipeline permissions** hạn chế cho service connections, variable groups, environments quan trọng.
- [ ] Gắn **Approvals + Branch control** cho environment `production`.
- [ ] PAT dùng cho self-hosted agent đặt scope tối thiểu (`Agent Pools: Read & manage`) và hạn ngắn, có lịch rotate.
- [ ] Quét secret rò rỉ trong repo (CredScan / GitHub Advanced Security / gitleaks trong pipeline).

### Tốc độ & chi phí

- [ ] Cache dependencies (`Cache@2`) cho npm/pip/NuGet/Maven/Gradle.
- [ ] Chạy jobs độc lập **song song**, chỉ `dependsOn` khi thật sự cần.
- [ ] Test nặng (E2E, performance) chuyển sang **schedule đêm**, không chạy mỗi PR.
- [ ] Set **retention policy** hợp lý; chỉ Retain tay bản release quan trọng.
- [ ] Cân nhắc self-hosted agent khi build > 1.800 phút/tháng hoặc cần Docker layer cache.

### Độ tin cậy

- [ ] Branch policy: bắt buộc build validation + tối thiểu 1 reviewer cho `main`.
- [ ] Deployment có **smoke test** sau deploy (health endpoint).
- [ ] Production deploy qua **slot swap** (App Service) hoặc **canary/rolling** thay vì deploy thẳng.
- [ ] Mọi stage deploy production có `condition: succeeded()` chuỗi từ stage trước.
- [ ] Log build được giữ đủ lâu để truy vết sự cố (retention ≥ 30 ngày cho `main`).

### Vận hành

- [ ] Đặt `displayName` rõ ràng cho mọi stage/job/step — log dễ đọc, dễ debug.
- [ ] Dùng `name:` cho build number có ý nghĩa (`prefix_yyyyMMdd.rev`).
- [ ] Gắn work items vào PR/commit để traceability Boards ↔ Pipelines.
- [ ] Có pipeline **nightly** chạy scan bảo mật (Trivy, CodeQL) và dependency check (Dependabot/Renovate).

---

## 19. FAQ và Cheat Sheet

### 19.1. FAQ

**Hỏi: Azure DevOps khác GitHub Actions thế nào? Nên chọn cái nào?**
Đáp: GitHub Actions gắn chặt với GitHub, marketplace khổng lồ, tốt cho open source và team đã ở GitHub. Azure DevOps mạnh ở quản trị tập trung (Boards, Test Plans, Artifacts, approvals/checks chi tiết), hợp doanh nghiệp dùng hệ sinh thái Microsoft/Azure.

**Hỏi: Chạy pipeline tay với tham số khác nhau thế nào?**
Đáp: Khai báo `parameters:` trong YAML → bấm **Run pipeline** → điền form tham số (hoặc `az pipelines run --parameters`).

**Hỏi: Làm sao deploy 1 commit cũ (rollback)?**
Đáp: Sạch nhất là **revert commit** trên `main` để CI/CD chạy lại (có audit trail). Nhanh hơn: **Run pipeline** chọn branch/tag bản cũ. Với App Service slots: swap ngược slot.

**Hỏi: Multi-repo checkout (pipeline cần 2 repo) làm thế nào?**
Đáp: Dùng `resources.repositories` + nhiều `checkout` step (mục 7.3); repo thứ hai checkout vào `$(Pipeline.Workspace)/s/<alias>`.

**Hỏi: Biến giữa các jobs/stages truyền thế nào?**
Đáp: Cùng job: `##vso[task.setvariable variable=myVar]giá trị`. Sang job khác: thêm `isOutput=true`, tham chiếu `dependencies.JobName.outputs['stepName.myVar']`. Sang stage khác: dùng `stageDependencies...` tương tự (cần `dependsOn`).

```yaml
jobs:
- job: SetVar
  steps:
  - script: echo "##vso[task.setvariable variable=version;isOutput=true]1.2.3"
    name: setver
- job: UseVar
  dependsOn: SetVar
  variables:
    appVersion: $[ dependencies.SetVar.outputs['setver.version'] ]
  steps:
  - script: echo "Version là $(appVersion)"
```

**Hỏi: `System.AccessToken` dùng được cho việc gì?**
Đáp: Gọi Azure DevOps REST API, `git push`, `az` CLI mà không cần PAT riêng. Bật trong agent job settings; siết quyền ở **Project settings → Pipelines → Settings** nếu cần.

### 19.2. Cheat Sheet

**Multi-stage chuẩn**

```yaml
stages:
- stage: Build
  jobs:
  - job: B
    steps: [ ... ]
- stage: Deploy
  dependsOn: Build
  condition: succeeded()
  jobs:
  - deployment: D
    environment: 'production'
    strategy:
      runOnce:
        deploy:
          steps: [ ... ]
```

**Biểu thức hay dùng**

| Mục đích | Cú pháp |
|---|---|
| Tham chiếu biến | `$(tenBien)` |
| Tham chiếu parameter | `${{ parameters.tenParam }}` |
| Condition: branch main | `eq(variables['Build.SourceBranch'], 'refs/heads/main')` |
| Condition: PR build | `eq(variables['Build.Reason'], 'PullRequest')` |
| Condition: chạy tay | `eq(variables['Build.Reason'], 'Manual')` |
| Chỉ chạy khi stage trước OK | `condition: succeeded()` |
| Luôn chạy (cleanup) | `condition: always()` |

**Logging commands (`##vso`)**

| Lệnh | Tác dụng |
|---|---|
| `##vso[task.setvariable variable=x]y` | Set biến `x=y` cho các step sau |
| `##vso[task.setvariable variable=x;issecret=true]y` | Set secret |
| `##vso[task.logissue type=error]msg` | Ghi lỗi vào summary |
| `##vso[task.logissue type=warning]msg` | Ghi cảnh báo |
| `##vso[task.complete result=Failed]` | Đánh fail task hiện tại |
| `##vso[build.addbuildtag]hotfix` | Gắn tag cho build |

**Azure CLI nhanh**

```bash
# Chạy pipeline tay với branch chỉ định
az pipelines run --org https://dev.azure.com/myorg --project myproject \
  --name my-pipeline --branch main

# Tạo variable group
az pipelines variable-group create --org https://dev.azure.com/myorg \
  --project myproject --name my-vars --variables a=1 b=2
```
