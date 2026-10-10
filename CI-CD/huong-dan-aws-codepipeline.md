# Hướng dẫn AWS CodePipeline — CI/CD trên AWS từ A đến Z

> Hướng dẫn chi tiết xây dựng pipeline CI/CD bằng AWS CodePipeline: khái niệm, kiến trúc, tạo pipeline bằng Console và CloudFormation, cấu hình từng stage, approval thủ công, artifact S3, IAM least privilege, cross-account, ví dụ thực tế, pricing, troubleshooting, best practices và cheat sheet.

---

## Mục lục

1. [AWS CodePipeline là gì](#1-aws-codepipeline-là-gì)
2. [So sánh với GitHub Actions và Jenkins](#2-so-sánh-với-github-actions-và-jenkins)
3. [Kiến trúc: stages, actions, artifacts](#3-kiến-trúc-stages-actions-artifacts)
4. [Tạo pipeline bằng Console (từng bước)](#4-tạo-pipeline-bằng-console-từng-bước)
5. [Tạo pipeline bằng CloudFormation](#5-tạo-pipeline-bằng-cloudformation)
6. [Source stage: CodeCommit, GitHub, S3, ECR](#6-source-stage-codecommit-github-s3-ecr)
7. [Build stage: CodeBuild và buildspec.yml](#7-build-stage-codebuild-và-buildspecyml)
8. [Deploy stage: CodeDeploy, ECS, Lambda, S3, CloudFormation](#8-deploy-stage-codedeploy-ecs-lambda-s3-cloudformation)
9. [Manual approval actions](#9-manual-approval-actions)
10. [Artifacts lưu trong S3](#10-artifacts-lưu-trong-s3)
11. [IAM roles & policies (least privilege)](#11-iam-roles--policies-least-privilege)
12. [Cross-account pipelines](#12-cross-account-pipelines)
13. [Ví dụ thực tế](#13-ví-dụ-thực-tế)
14. [Pricing (giá cả)](#14-pricing-giá-cả)
15. [Troubleshooting lỗi thường gặp](#15-troubleshooting-lỗi-thường-gặp)
16. [Best practices](#16-best-practices)
17. [Cheat sheet](#17-cheat-sheet)

---

## 1. AWS CodePipeline là gì

**AWS CodePipeline** là dịch vụ CI/CD được quản lý hoàn toàn (fully managed) của AWS, tự động hoá quy trình phát hành phần mềm: mỗi khi code thay đổi, pipeline tự build, test và deploy theo các stage đã định nghĩa.

CodePipeline nằm trong bộ **AWS Code* services** (còn gọi là "AWS Developer Tools"):

| Dịch vụ | Vai trò | Ví dụ tương đương |
|---|---|---|
| **AWS CodeCommit** | Git repository (quản lý source code) | GitHub / GitLab |
| **AWS CodeBuild** | Build + test code | GitHub Actions runners / Jenkins agents |
| **AWS CodeDeploy** | Deploy lên EC2 / On-premise / Lambda / ECS | ArgoCD / Ansible |
| **AWS CodePipeline** | Điều phối CI/CD: nối Source → Build → Test → Deploy thành pipeline | GitHub Actions workflows / Jenkins pipelines |

**CodePipeline không tự build, không tự deploy** — nó là "nhạc trưởng" điều phối CodeBuild, CodeDeploy và các dịch vụ khác. Ngoài ra, pipeline còn tích hợp được với nhiều tool bên ngoài: GitHub, S3, ECR, Jenkins, Lambda, CloudFormation...

### Các khái niệm cơ bản

- **Pipeline**: toàn bộ quy trình CI/CD; được kích hoạt (trigger) mỗi khi source thay đổi.
- **Stage**: bước lớn (`Source`, `Build`, `Deploy`) — chạy **tuần tự**.
- **Action**: công việc cụ thể trong stage — chạy **song song** hoặc tuần tự (tùy `runOrder`).
- **Artifact**: dữ liệu trung gian giữa các action, lưu trong **artifact bucket** (S3).
- **Execution**: một lần chạy pipeline.

### Khi nào nên dùng CodePipeline?

- Hạ tầng đã nằm trên AWS, muốn CI/CD ăn khớp với IAM, CloudWatch, S3, ECS...
- Cần audit/compliance: mọi action log vào CloudTrail, artifact mã hoá trong S3.
- Muốn dịch vụ managed, không tự vận hành server CI; cần pipeline cross-account (dev → staging → prod ở các account khác nhau).

---

## 2. So sánh với GitHub Actions và Jenkins

| Tiêu chí | AWS CodePipeline | GitHub Actions | Jenkins |
|---|---|---|---|
| **Mô hình** | Dịch vụ managed, điều phối qua stages/actions | Workflow YAML chạy trên runner | Server tự quản, pipeline Groovy |
| **Định nghĩa pipeline** | Console / CloudFormation / API | File `.github/workflows/*.yml` | `Jenkinsfile` hoặc cấu hình UI |
| **Nơi chạy** | AWS-managed (CodeBuild, Lambda...) | GitHub-hosted hoặc self-hosted runner | Agent do bạn tự quản |
| **Build/Test** | Uỷ quyền cho CodeBuild | Chạy trên runner | Chạy trên agent |
| **Chi phí cơ bản** | $1/active pipeline/tháng (pipeline active đầu tiên miễn phí) | 2000 phút/tháng miễn phí (repo public không giới hạn) | Miễn phí (tự trả chi phí server) |
| **Tích hợp AWS** | Rất sâu (IAM, S3, ECS, Lambda, CloudFormation native) | Qua OIDC + AWS CLI / action cộng đồng | Qua plugin |
| **Artifacts** | S3 artifact bucket tự động, có versioning + mã hoá | `actions/upload-artifact` (lưu 90 ngày) | Tự cấu hình lưu trữ |
| **Approval thủ công** | Native (Manual action + SNS) | Environment protection rules | `input` step |
| **Cross-account** | Hỗ trợ native (IAM role + KMS key policy) | Qua OIDC assume role | Qua credential |
| **Điểm yếu** | Ít action cộng đồng; giá tăng theo số pipeline | Ràng buộc với GitHub | Tự vận hành, vá bảo mật, scale agent |

**Kết luận nhanh:** hạ tầng AWS là chính + cần compliance → **CodePipeline**; code ở GitHub + thích hệ sinh thái action → **GitHub Actions**; cần tự chủ hoàn toàn/plugin đặc thù → **Jenkins**. Nhiều team dùng kết hợp (ví dụ: trigger CodePipeline từ GitHub, build bằng CodeBuild, deploy EKS bằng action Lambda tuỳ chỉnh).

---

## 3. Kiến trúc: stages, actions, artifacts

### 3.1. Sơ đồ tổng quan

`Source` (CodeCommit/GitHub/S3/ECR) → `Build` (CodeBuild + buildspec.yml) → `Test` (tùy chọn) → `Deploy` (CodeDeploy/ECS/Lambda/S3/CloudFormation). Mọi artifact trung gian lưu trong **Artifact Store** (S3 bucket).

### 3.2. Cách dữ liệu chảy qua pipeline (artifacts)

Mỗi action có **input artifacts** và **output artifacts**. CodePipeline tự zip/unzip artifact giữa các action qua S3 artifact bucket. Ví dụ pipeline web app điển hình:

| Stage | Action | Input | Output |
|---|---|---|---|
| Source | `Source` (GitHub) | — | `SourceArtifact` |
| Build | `Build` (CodeBuild) | `SourceArtifact` | `BuildArtifact` |
| Deploy | `Deploy` (S3) | `BuildArtifact` | — |

Tên artifact do bạn đặt; action sau tham chiếu đúng tên ở `inputArtifacts`.

### 3.3. Execution flow và runOrder

- **Stage chạy tuần tự**: `Deploy` chỉ bắt đầu khi `Build` thành công; action nào fail → stage fail → execution dừng.
- **Action trong cùng stage** chạy song song theo mặc định; đặt `runOrder: 2` để chạy sau action `runOrder: 1`.
- Mỗi trigger tạo một **execution** mới (ID dạng `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`).

### 3.4. Trigger (cách pipeline được kích hoạt)

| Loại trigger | Áp dụng cho source | Ghi chú |
|---|---|---|
| **Push / commit** | CodeCommit, GitHub (CodeStar Connection) | Dùng CloudWatch Events / EventBridge rule tự động tạo |
| **PollForSourceChanges** | CodeCommit, S3 | Tự poll định kỳ (cũ, nên tránh) |
| **S3 event** | S3 source | EventBridge rule bắt `PutObject` |
| **ECR image push** | ECR source | EventBridge rule bắt `PutImage` |
| **Manual** | — | Bấm "Release change" trên Console hoặc `aws codepipeline start-pipeline-execution` |

> Từ cuối 2023, CodePipeline hỗ trợ **pipeline trigger theo Git tag** và **lọc theo branch/file path** (pipeline trigger filters) — xem mục 6.2.

### 3.5. Pipeline V1 vs V2

Khi tạo pipeline, chọn **pipeline type**:

| | **V1** | **V2** |
|---|---|---|
| Trigger | EventBridge / polling | EventBridge + trigger filters (branch, tag, file path) |
| Giá | $1/active pipeline/tháng | $1/active pipeline/tháng (giống V1) |
| Biến pipeline | Hạn chế | Hỗ trợ **pipeline variables** đầy đủ |
| Queued executions | Không | Có (execution xếp hàng khi pipeline đang chạy) |

**Khuyến nghị: luôn dùng V2** cho pipeline mới.

---
## 4. Tạo pipeline bằng Console (từng bước)

Ví dụ: deploy web React từ GitHub lên S3 + CloudFront.

### Bước 1: Pipeline settings

Console → **CodePipeline** → **Create pipeline**:
- **Pipeline name**: `web-tinh-pipeline`; **Pipeline type**: **V2**; **Execution mode**: **Queued**.
- **Service role**: **New service role** (tinh chỉnh least privilege sau — mục 11).
- **Artifact store**: **Custom location**, chọn bucket riêng (gắn KMS key cho production) → **Next**.

### Bước 2: Source stage

- **Source provider**: **GitHub (via GitHub App)** → **Connect to GitHub**, authorize và cài app vào repo.
- **Repository**: `TieuDuongg/web-tinh-demo`, **Branch**: `main`.
- **Output artifact format**: **CodePipeline default** (chọn **Full clone** nếu CodeBuild cần `.git` để lấy commit hash/tag).
- **Trigger**: bật, filter branch `main` + file paths `frontend/**` (V2).
- **Output artifact name**: `SourceArtifact` → **Next**.

### Bước 3: Build stage

- **Build provider**: **AWS CodeBuild** → **Create project**:
  - **Project name**: `web-tinh-build`; **Environment image**: Managed, Amazon Linux 2023, `aws/codebuild/amazonlinux2023-x86_64-standard:5.0` (bản mới nhất).
  - **Service role**: New service role; **Buildspec**: **Use a buildspec file**.
  - **Continue to CodePipeline**, chọn project vừa tạo.
- **Input artifacts**: `SourceArtifact`; **Output**: `BuildArtifact` → **Next**.

### Bước 4: Deploy stage

- **Deploy provider**: **Amazon S3**; **Bucket**: `web-tinh-demo-prod`.
- **Input artifacts**: `BuildArtifact`; tích **Extract file before deploy** → **Next** → **Review** → **Create pipeline**.

### Bước 5: Chạy thử

Bấm **Release change** (hoặc push code để trigger tự động), bấm vào từng stage xem log CodeBuild và kiểm tra file trên S3.

| Thao tác | Vị trí |
|---|---|
| Lịch sử execution | Pipeline → **View history** |
| Chạy thủ công / Dừng execution | **Release change** / Execution → **Stop execution** |
| Retry stage fail | Stage fail → **Retry stage** |
| Sửa / Xoá pipeline | **Edit** / **Delete pipeline** (không xoá artifact bucket — xoá tay) |

---

## 5. Tạo pipeline bằng CloudFormation

Template **đầy đủ, chạy được**: GitHub (CodeStar Connection) → CodeBuild → S3.

> **Lưu ý:** CodeStar Connection tạo **thủ công một lần** trên Console (authorize GitHub cần bấm tay), rồi tham chiếu ARN trong template.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "CodePipeline: GitHub -> CodeBuild -> S3 (web tinh)"

Parameters:
  GitHubConnectionArn:
    Type: String
  GitHubRepo:
    Type: String
    Default: "TieuDuongg/web-tinh-demo"
  GitHubBranch:
    Type: String
    Default: "main"
  DeployBucketName:
    Type: String
    Default: "web-tinh-demo-prod"
  ArtifactBucketName:
    Type: String
    Default: "web-tinh-pipeline-artifacts"

Resources:
  ArtifactBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref ArtifactBucketName
      VersioningConfiguration:
        Status: Enabled
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true
      LifecycleConfiguration:
        Rules:
          - Id: XoaArtifactCu
            Status: Enabled
            ExpirationInDays: 90
            NoncurrentVersionExpirationInDays: 30

  PipelineRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub "${AWS::StackName}-pipeline-role"
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: codepipeline.amazonaws.com
            Action: sts:AssumeRole
      Policies:
        - PolicyName: pipeline-policy
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action:
                  - s3:GetObject
                  - s3:GetObjectVersion
                  - s3:PutObject
                  - s3:ListBucket
                Resource:
                  - !Sub "${ArtifactBucket.Arn}"
                  - !Sub "${ArtifactBucket.Arn}/*"
              - Effect: Allow
                Action:
                  - codebuild:StartBuild
                  - codebuild:BatchGetBuilds
                Resource: !GetAtt BuildProject.Arn
              - Effect: Allow
                Action:
                  - codestar-connections:UseConnection
                Resource: !Ref GitHubConnectionArn
              - Effect: Allow
                Action:
                  - s3:PutObject
                  - s3:GetObject
                  - s3:ListBucket
                Resource:
                  - !Sub "arn:aws:s3:::${DeployBucketName}"
                  - !Sub "arn:aws:s3:::${DeployBucketName}/*"

  BuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      Name: !Sub "${AWS::StackName}-build"
      ServiceRole: !GetAtt BuildRole.Arn
      Artifacts:
        Type: CODEPIPELINE
      Environment:
        Type: LINUX_CONTAINER
        ComputeType: BUILD_GENERAL1_SMALL
        Image: aws/codebuild/amazonlinux2023-x86_64-standard:5.0
      Source:
        Type: CODEPIPELINE
        BuildSpec: buildspec.yml

  BuildRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub "${AWS::StackName}-build-role"
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: Allow
            Principal:
              Service: codebuild.amazonaws.com
            Action: sts:AssumeRole
      Policies:
        - PolicyName: build-policy
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
              - Effect: Allow
                Action:
                  - logs:CreateLogGroup
                  - logs:CreateLogStream
                  - logs:PutLogEvents
                Resource: !Sub "arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:/aws/codebuild/*"
              - Effect: Allow
                Action:
                  - s3:GetObject
                  - s3:GetObjectVersion
                  - s3:PutObject
                Resource:
                  - !Sub "${ArtifactBucket.Arn}/*"

  Pipeline:
    Type: AWS::CodePipeline::Pipeline
    Properties:
      Name: !Sub "${AWS::StackName}-pipeline"
      RoleArn: !GetAtt PipelineRole.Arn
      PipelineType: V2
      ExecutionMode: QUEUED
      ArtifactStore:
        Type: S3
        Location: !Ref ArtifactBucket
      Triggers:
        - ProviderType: CodeStarSourceConnection
          GitConfiguration:
            SourceActionName: Source
            Push:
              - Branches:
                  Includes:
                    - main
                FilePaths:
                  Includes:
                    - "frontend/**"
      Stages:
        - Name: Source
          Actions:
            - Name: Source
              ActionTypeId:
                Category: Source
                Owner: AWS
                Provider: CodeStarSourceConnection
                Version: "1"
              OutputArtifacts:
                - Name: SourceArtifact
              Configuration:
                ConnectionArn: !Ref GitHubConnectionArn
                FullRepositoryId: !Ref GitHubRepo
                BranchName: !Ref GitHubBranch
                OutputArtifactFormat: CODE_ZIP
        - Name: Build
          Actions:
            - Name: Build
              ActionTypeId:
                Category: Build
                Owner: AWS
                Provider: CodeBuild
                Version: "1"
              InputArtifacts:
                - Name: SourceArtifact
              OutputArtifacts:
                - Name: BuildArtifact
              Configuration:
                ProjectName: !Ref BuildProject
        - Name: Deploy
          Actions:
            - Name: Deploy
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: S3
                Version: "1"
              InputArtifacts:
                - Name: BuildArtifact
              Configuration:
                BucketName: !Ref DeployBucketName
                Extract: "true"

Outputs:
  PipelineName:
    Value: !Ref Pipeline
  PipelineUrl:
    Value: !Sub "https://console.aws.amazon.com/codesuite/codepipeline/pipelines/${Pipeline}/view?region=${AWS::Region}"
```

```bash
aws cloudformation deploy --stack-name web-tinh-pipeline --template-file pipeline.yaml \
  --parameter-overrides GitHubConnectionArn=<connection-arn> GitHubRepo=TieuDuongg/web-tinh-demo \
  --capabilities CAPABILITY_NAMED_IAM
```

---
## 6. Source stage: CodeCommit, GitHub, S3, ECR

Source stage lấy code/tài nguyên đầu vào. Stage Source có thể chứa nhiều source action song song, mỗi action cho ra một output artifact riêng (ví dụ: một action lấy code app, một action lấy repo chứa template CloudFormation).

### 6.1. AWS CodeCommit

Git repo do AWS quản lý; trigger qua **EventBridge** (tự tạo rule khi tạo pipeline trên Console).

```yaml
# Action trong CloudFormation - Source tu CodeCommit
- Name: Source
  ActionTypeId:
    Category: Source
    Owner: AWS
    Provider: CodeCommit
    Version: "1"
  OutputArtifacts:
    - Name: SourceArtifact
  Configuration:
    RepositoryName: web-tinh-demo
    BranchName: main
    PollForSourceChanges: "false"   # false = dung EventBridge (khuyen nghi)
    OutputArtifactFormat: CODE_ZIP  # hoac CODEBUILD_CLONE_REF
```

> Pipeline role cần quyền `codecommit:GetBranch`, `GetCommit`, `UploadArchive`, `GetUploadArchiveStatus`, `CancelUploadArchive` trên ARN repo.

### 6.2. GitHub (qua CodeStar Connection) — cách nên dùng

Thay thế OAuth token cũ (đã deprecated). **Tạo connection 1 lần:** Console → **Developer Tools** → **Connections** → **Create connection** → provider **GitHub** → **Connect to GitHub**, authorize và **Install GitHub App** vào repo → chờ trạng thái **Available** → copy ARN.

Action trong pipeline:

```yaml
- Name: Source
  ActionTypeId:
    Category: Source
    Owner: AWS
    Provider: CodeStarSourceConnection
    Version: "1"
  OutputArtifacts:
    - Name: SourceArtifact
  Configuration:
    ConnectionArn: arn:aws:codestar-connections:ap-southeast-1:123456789012:connection/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    FullRepositoryId: TieuDuongg/web-tinh-demo
    BranchName: main
    OutputArtifactFormat: CODE_ZIP   # hoac FULL_CLONE (can .git)
```

> Pipeline role cần quyền `codestar-connections:UseConnection` trên ARN connection.

**Trigger filters (pipeline V2):**

```yaml
Triggers:
  - ProviderType: CodeStarSourceConnection
    GitConfiguration:
      SourceActionName: Source
      Push:
        - Branches:
            Includes: ["main", "release/*"]
            Excludes: ["wip-*"]
          FilePaths:
            Includes: ["src/**", "Dockerfile"]
            Excludes: ["docs/**", "*.md"]
          Tags:
            Includes: ["v*"]
```

### 6.3. Amazon S3

Dùng khi source là file zip có sẵn trong S3 (artifact do hệ thống khác đẩy lên).

```yaml
- Name: Source
  ActionTypeId:
    Category: Source
    Owner: AWS
    Provider: S3
    Version: "1"
  OutputArtifacts:
    - Name: SourceArtifact
  Configuration:
    S3Bucket: nguon-build-demo
    S3ObjectKey: source/app-v1.2.3.zip
    PollForSourceChanges: "false"   # dung EventBridge rule bat s3:ObjectCreated:*
```

> S3 source yêu cầu file **zip**; muốn trigger tự động thì tạo EventBridge rule bắt `PutObject`.

### 6.4. Amazon ECR (container image)

Mỗi khi có image mới push lên ECR → trigger pipeline deploy container.

```yaml
- Name: Source
  ActionTypeId:
    Category: Source
    Owner: AWS
    Provider: ECR
    Version: "1"
  OutputArtifacts:
    - Name: SourceArtifact   # chua file imagedefinitions.json
  Configuration:
    RepositoryName: web-api
    ImageTag: latest
    PollForSourceChanges: "false"
```

Artifact output của ECR source là file **`imageDetail.json`** chứa thông tin image (repository, tag, digest, registry ID). Deploy action (ECS) sẽ đọc file này. Ví dụ nội dung:

```json
{
  "ImageURI": "123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/web-api:latest",
  "ImageDigest": "sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08",
  "RegistryId": "123456789012",
  "RepositoryName": "web-api",
  "ImageTag": "latest"
}
```

---

## 7. Build stage: CodeBuild và buildspec.yml

CodeBuild là dịch vụ build managed: bạn đưa source + file **`buildspec.yml`**, AWS lo phần chạy. Buildspec định nghĩa **phases** (giai đoạn), **artifacts** (file đầu ra), **cache** và **reports**.

### 7.1. Cấu trúc buildspec.yml đầy đủ

```yaml
version: 0.2

env:
  variables:
    NODE_ENV: production
    APP_NAME: web-tinh-demo
  parameter-store:
    API_KEY: /prod/web-tinh/api-key
  secrets-manager:
    DB_PASSWORD: prod/db:password
  exported-variables:
    - BUILD_VERSION
    - IMAGE_TAG

phases:
  install:
    runtime-versions:
      nodejs: 20
      python: 3.11
    commands:
      - echo "Cai dat dependencies..."
      - npm ci
  pre_build:
    commands:
      - echo "Kiem tra chat luong code..."
      - npm run lint
      - npm run test:unit
      - |
        export BUILD_VERSION=$(git describe --tags --always --dirty)
        export IMAGE_TAG=${BUILD_VERSION}-$(date +%Y%m%d%H%M%S)
        echo "BUILD_VERSION=$BUILD_VERSION"
  build:
    commands:
      - echo "Build ung dung..."
      - npm run build
  post_build:
    commands:
      - echo "Hoan tat build $BUILD_VERSION"
      - ls -la dist/

artifacts:
  files:
    - "**/*"
  base-directory: dist        # chi lay file trong thu muc dist
  discard-paths: no           # giu nguyen cau truc thu muc (yes = gom phang)
  name: build-output-$(date +%Y-%m-%d)

cache:
  paths:
    - node_modules/**/*
    - /root/.npm/**/*

reports:
  unit-tests:
    files:
      - "test-results.xml"
    base-directory: reports
    file-format: JUNITXML
  coverage:
    files:
      - "coverage.xml"
    base-directory: reports
    file-format: COBERTURAXML
```

### 7.2. Giải thích từng phase

| Phase | Dùng để làm gì |
|---|---|
| `install` | Cài runtime (`runtime-versions`), `npm ci` |
| `pre_build` | Lint, unit test, chuẩn bị biến, login ECR |
| `build` | `npm run build`, `docker build`, `go build`... |
| `post_build` | Push image, tạo `imagedefinitions.json`, thông báo |

**Lưu ý quan trọng:**

- Lệnh nào exit code ≠ 0 → phase fail → build fail → pipeline dừng (muốn bỏ qua: `command || true`).
- `env.exported-variables`: export biến sang CodePipeline, action sau dùng qua `#{Build.BUILD_VERSION}`.
- Docker build trong CodeBuild: bật **Privileged mode** (mục Environment).

### 7.3. buildspec.yml cho Docker image + push ECR

```yaml
version: 0.2

env:
  variables:
    AWS_REGION: ap-southeast-1
    ECR_REPO: 123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/web-api

phases:
  pre_build:
    commands:
      - echo "Dang nhap ECR..."
      - aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
      - IMAGE_TAG=${CODEBUILD_RESOLVED_SOURCE_VERSION:0:8}
      - echo "IMAGE_TAG=$IMAGE_TAG"
  build:
    commands:
      - echo "Build Docker image..."
      - docker build -t $ECR_REPO:$IMAGE_TAG -t $ECR_REPO:latest .
  post_build:
    commands:
      - echo "Push image len ECR..."
      - docker push $ECR_REPO:$IMAGE_TAG
      - docker push $ECR_REPO:latest
      - printf '[{"name":"web-api","imageUri":"%s:%s"}]' "$ECR_REPO" "$IMAGE_TAG" > imagedefinitions.json
      - cat imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

> `CODEBUILD_RESOLVED_SOURCE_VERSION` là biến môi trường CodeBuild tự inject = commit SHA của source. `imagedefinitions.json` là "hợp đồng" giữa build và ECS deploy action.

### 7.4. Cấu hình CodeBuild project (CloudFormation)

```yaml
BuildProject:
  Type: AWS::CodeBuild::Project
  Properties:
    Name: web-api-build
    ServiceRole: !GetAtt BuildRole.Arn
    Artifacts:
      Type: CODEPIPELINE
    Environment:
      Type: LINUX_CONTAINER
      ComputeType: BUILD_GENERAL1_MEDIUM   # SMALL | MEDIUM | LARGE | XL
      Image: aws/codebuild/amazonlinux2023-x86_64-standard:5.0
      PrivilegedMode: true                 # bat buoc neu docker build
      EnvironmentVariables:
        - Name: AWS_REGION
          Value: ap-southeast-1
    Source:
      Type: CODEPIPELINE
      BuildSpec: buildspec.yml             # hoac nhung truc tiep YAML
    TimeoutInMinutes: 30
    Cache:
      Type: S3
      Location: !Sub "${ArtifactBucket}/codebuild-cache"
```

### 7.5. Test action (stage riêng)

Tách test thành action riêng để build và test chạy **song song**:

```yaml
- Name: Test
  Actions:
    - Name: UnitTest
      ActionTypeId:
        Category: Test
        Owner: AWS
        Provider: CodeBuild
        Version: "1"
      InputArtifacts:
        - Name: SourceArtifact
      Configuration:
        ProjectName: !Ref TestProject   # buildspec chi chay lint + test, khong sinh artifact
      RunOrder: 1
```

---

## 8. Deploy stage: CodeDeploy, ECS, Lambda, S3, CloudFormation

### 8.1. AWS CodeDeploy → EC2 / On-premise

CodeDeploy deploy code lên EC2 (hoặc server on-premise có cài CodeDeploy agent) theo kiểu **in-place** (deploy đè lên server cũ) hoặc **blue/green**.

**Chuẩn bị:** EC2 có **IAM instance profile** + **CodeDeploy agent**; repo có **`appspec.yml`** + scripts hooks.

`appspec.yml` mẫu:

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/web-api
    overwrite: true
permissions:
  - object: /var/www/web-api
    owner: ec2-user
    group: ec2-user
    mode: 755
hooks:
  BeforeInstall:
    - location: scripts/stop-server.sh
      timeout: 60
  AfterInstall:
    - location: scripts/install-deps.sh
      timeout: 300
  ApplicationStart:
    - location: scripts/start-server.sh
      timeout: 60
  ValidateService:
    - location: scripts/health-check.sh
      timeout: 120
```

Action trong pipeline:

```yaml
- Name: Deploy
  ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: CodeDeploy
    Version: "1"
  InputArtifacts:
    - Name: BuildArtifact   # phai chua appspec.yml o thu muc goc
  Configuration:
    ApplicationName: web-api-codedeploy
    DeploymentGroupName: web-api-prod-dg
```

> **Bẫy:** `BuildArtifact` phải chứa `appspec.yml` ở gốc zip — nếu `base-directory: dist` thì copy `appspec.yml` vào `dist` trước.

> Pipeline role cần thêm quyền CodeDeploy (`CreateDeployment`, `GetDeployment`, `RegisterApplicationRevision`...) — xem mẫu least privilege ở mục 11.1.

### 8.2. Amazon ECS (deploy container)

Có 2 cách: **ECS standard** (rolling update) và **CodeDeploy blue/green**. Build stage phải sinh `imagedefinitions.json`:

```json
[
  {
    "name": "web-api",
    "imageUri": "123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/web-api:a1b2c3d4"
  }
]
```

Trong đó `name` = **container name** trong task definition. Action:

```yaml
- Name: Deploy
  ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: ECS
    Version: "1"
  InputArtifacts:
    - Name: BuildArtifact   # chua imagedefinitions.json
  Configuration:
    ClusterName: prod-cluster
    ServiceName: web-api-service
    FileName: imagedefinitions.json
```

**Blue/Green qua CodeDeploy** — cần thêm `appspec.yaml` + `taskdef.json`:

`appspec.yaml` (cho ECS):

```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>
        LoadBalancerInfo:
          ContainerName: web-api
          ContainerPort: 8080
```

Action dùng provider `CodeDeployToECS`:

```yaml
- Name: Deploy
  ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: CodeDeployToECS
    Version: "1"
  InputArtifacts:
    - Name: BuildArtifact
  Configuration:
    ApplicationName: web-api-ecs-app
    DeploymentGroupName: web-api-ecs-dg
    TaskDefinitionTemplateArtifact: BuildArtifact
    TaskDefinitionTemplatePath: taskdef.json
    AppSpecTemplateArtifact: BuildArtifact
    AppSpecTemplatePath: appspec.yaml
    Image1ArtifactName: BuildArtifact
    Image1ContainerName: IMAGE1_NAME
```

Trong `taskdef.json`, chỗ image ghi placeholder `"image": "<IMAGE1_NAME>"` — CodeDeploy thay bằng image URI từ artifact `Image1ArtifactName` (file `imageDetail.json` do ECR source hoặc build sinh ra).

### 8.3. AWS Lambda

Update function code (zip) — dùng provider **Lambda**, hoặc **CodeDeploy** nếu cần canary/linear traffic shifting.

```yaml
# Deploy don gian: update function code
- Name: Deploy
  ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: Lambda
    Version: "1"
  InputArtifacts:
    - Name: BuildArtifact   # file zip chua code, ten file tuy y
  Configuration:
    FunctionName: xu-ly-anh-thumbnail
    UserParameters: '{"bucket":"anh-upload-prod"}'   # tuy chon, truyen vao
```

`BuildArtifact` là file zip (`function.zip`) chứa code ở thư mục gốc. Dùng SAM thì deploy qua **CloudFormation action** (mục 8.5).

### 8.4. Amazon S3 (static website)

Deploy web tĩnh: upload file lên S3 bucket (đã bật static website hosting hoặc dùng CloudFront + OAC).

```yaml
- Name: Deploy
  ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: S3
    Version: "1"
  InputArtifacts:
    - Name: BuildArtifact
  Configuration:
    BucketName: web-tinh-demo-prod
    Extract: "true"          # giai nen zip truoc khi upload
    CacheControl: "max-age=31536000"   # tuy chon: cache cho file tinh
```

> Invalidate CloudFront sau deploy: thêm action **Lambda** gọi `create-invalidation` (ví dụ đầy đủ ở mục 13.1).

### 8.5. AWS CloudFormation (deploy infrastructure / SAM)

Deploy stack CloudFormation — hợp cho serverless (SAM) hoặc cập nhật hạ tầng cùng lúc với code. Thường dùng 2 action: `CreateChangeSet` (runOrder 1) → manual approval → `ExecuteChangeSet` (runOrder 3).

```yaml
- Name: DeployInfra
  Actions:
    - Name: CreateChangeSet
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CloudFormation
        Version: "1"
      InputArtifacts:
        - Name: BuildArtifact   # chua template.yaml + sam config
      Configuration:
        ActionMode: CHANGE_SET_REPLACE
        StackName: web-api-prod
        ChangeSetName: pipeline-changeset
        TemplatePath: BuildArtifact::template.yaml
        Capabilities: CAPABILITY_IAM,CAPABILITY_NAMED_IAM
        RoleArn: arn:aws:iam::123456789012:role/cloudformation-deploy-role
        ParameterOverrides: |
          {
            "Environment": "prod",
            "ImageUri": { "Fn::GetArtifactAtt": ["BuildArtifact", "ImageUri"] }
          }
      RunOrder: 1
    - Name: ApproveChangeSet
      ActionTypeId:
        Category: Approval
        Owner: AWS
        Provider: Manual
        Version: "1"
      Configuration:
        NotificationArn: arn:aws:sns:ap-southeast-1:123456789012:deploy-approve
        CustomData: "Kiem tra change set truoc khi deploy len PROD"
      RunOrder: 2
    - Name: ExecuteChangeSet
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CloudFormation
        Version: "1"
      Configuration:
        ActionMode: CHANGE_SET_EXECUTE
        StackName: web-api-prod
        ChangeSetName: pipeline-changeset
      RunOrder: 3
```

> `ParameterOverrides` hỗ trợ `Fn::GetArtifactAtt` để "nối" build output (ví dụ image URI) vào infra template.

---

## 9. Manual approval actions

Manual approval chèn "người kiểm duyệt" vào pipeline — pipeline **dừng chờ** cho tới khi có người bấm Approve/Reject (Console, CLI hoặc link trong email SNS).

### 9.1. Cấu hình cơ bản

```yaml
- Name: ApproveProd
  ActionTypeId:
    Category: Approval
    Owner: AWS
    Provider: Manual
    Version: "1"
  Configuration:
    NotificationArn: arn:aws:sns:ap-southeast-1:123456789012:pipeline-approve
    CustomData: "Deploy ban build #{Build.BUILD_VERSION} len PROD?"
    ExternalEntityLink: https://dashboard.internal/acme/deploys   # link tuy chon
```

Khi pipeline tới bước này, SNS gửi email/SMS cho người duyệt. Người duyệt bấm link trong email → trang review → **Approve** hoặc **Reject** (kèm comment).

### 9.2. Vị trí đặt approval

Mô hình phổ biến: `Source` → `Build` → `DeployStaging` → **`Approval`** → `DeployProd`

```yaml
# Stage chen giua DeployStaging va DeployProd:
- Name: Approval
  Actions:
    - Name: ApproveProd
      ActionTypeId: {Category: Approval, Owner: AWS, Provider: Manual, Version: "1"}
      Configuration:
        NotificationArn: !Ref ApproveTopicArn
        CustomData: "Staging OK? Duyet de deploy len PROD"
```

### 9.3. Approve/Reject bằng AWS CLI

```bash
# Lay token cua approval action dang cho
aws codepipeline get-pipeline-state \
  --name web-api-pipeline \
  --query "stageStates[?stageName=='Approval'].actionStates[?actionName=='ApproveProd'].latestExecution.token" \
  --output text

# Duyet
aws codepipeline put-approval-result \
  --pipeline-name web-api-pipeline \
  --stage-name Approval \
  --action-name ApproveProd \
  --token <token> \
  --result status=Approved,summary="Staging test OK, duyet len prod"

# Tu choi
aws codepipeline put-approval-result \
  --pipeline-name web-api-pipeline \
  --stage-name Approval \
  --action-name ApproveProd \
  --token <token> \
  --result status=Rejected,summary="Phat hien loi UI tren staging, can fix"
```

> IAM user/role bấm Approve cần quyền `codepipeline:PutApprovalResult`; token chỉ hiệu lực khi action đang chờ.

---
## 10. Artifacts lưu trong S3

### 10.1. Artifact bucket là gì?

Mỗi pipeline cần một **artifact store** — S3 bucket lưu mọi artifact trung gian. Có thể dùng chung một bucket cho nhiều pipeline:

```
s3://web-tinh-pipeline-artifacts/
  web-tinh-pipeline/
    SourceArtif/
      a1B2c3D4.zip        # output cua Source action (execution 1)
    BuildArtif/
      e5F6g7H8.zip        # output cua Build action
```

### 10.2. Tạo và cấu hình bucket đúng chuẩn

```yaml
ArtifactBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: cong-ty-pipeline-artifacts
    VersioningConfiguration:
      Status: Enabled                      # bat buoc: CodePipeline can versioning
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: aws:kms
            KMSMasterKeyID: !Ref ArtifactKmsKey   # dung KMS CMK rieng (khuyen nghi)
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
    LifecycleConfiguration:
      Rules:
        - Id: XoaArtifactCu
          Status: Enabled
          ExpirationInDays: 90                     # artifact cu tu xoa sau 90 ngay
          NoncurrentVersionExpirationInDays: 30
```

**Mã hoá:** **SSE-S3 (AES256)** cho mặc định đơn giản; **SSE-KMS (CMK riêng)** cho production/cross-account (kiểm soát key policy, audit qua CloudTrail).

> **Bắt buộc với cross-account:** artifact bucket phải dùng **KMS CMK** (không dùng AWS managed key `aws/s3`), và key policy phải cho phép account kia `kms:Encrypt/kms:Decrypt`.

### 10.3. Bucket policy cho cross-account

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::999988887777:role/prod-deploy-role" },
      "Action": ["s3:GetObject", "s3:GetObjectVersion", "s3:PutObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::cong-ty-pipeline-artifacts",
        "arn:aws:s3:::cong-ty-pipeline-artifacts/*"
      ]
    }
  ]
}
```

### 10.4. Lưu ý về dung lượng và chi phí

- Artifact là file **zip** do CodePipeline tự đóng gói; artifact quá lớn → build/deploy chậm.
- **Lifecycle rule** là bắt buộc: pipeline chạy hàng trăm lần/tháng, không xoá artifact cũ thì bucket phình rất nhanh.
- Không để artifact bucket public; không dùng chung với bucket chứa code production.

---

## 11. IAM roles & policies (least privilege)

Pipeline dùng **3 role chính**:

```
┌──────────────────┐   assume   ┌──────────────────┐
│ Pipeline Role    │ ─────────▶ │ codepipeline.    │
│ (service role)   │            │ amazonaws.com    │
└──────────────────┘            └──────────────────┘
        │ gọi CodeBuild, CodeDeploy, S3...
        ▼
┌──────────────────┐   assume   ┌──────────────────┐
│ Build Role       │ ─────────▶ │ codebuild.       │
│                  │            │ amazonaws.com    │
└──────────────────┘            └──────────────────┘
        │ push ECR, đọc SSM...
        ▼
┌──────────────────┐   assume   ┌──────────────────┐
│ Deploy Role      │ ─────────▶ │ codedeploy /     │
│ (VD: CFN role)   │            │ cloudformation.  │
└──────────────────┘            │ amazonaws.com    │
                               └──────────────────┘
```

### 11.1. Pipeline service role (least privilege)

Thay vì dùng policy AWS tạo sẵn (`AWSCodePipelineServiceRole` full quyền), hãy giới hạn theo tài nguyên cụ thể:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:GetObjectVersion", "s3:PutObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::cong-ty-pipeline-artifacts",
        "arn:aws:s3:::cong-ty-pipeline-artifacts/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": ["kms:Encrypt", "kms:Decrypt", "kms:GenerateDataKey", "kms:DescribeKey"],
      "Resource": "arn:aws:kms:ap-southeast-1:123456789012:key/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    },
    {
      "Effect": "Allow",
      "Action": ["codebuild:StartBuild", "codebuild:BatchGetBuilds", "codebuild:StopBuild"],
      "Resource": [
        "arn:aws:codebuild:ap-southeast-1:123456789012:project/web-api-build",
        "arn:aws:codebuild:ap-southeast-1:123456789012:project/web-api-test"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "codedeploy:CreateDeployment",
        "codedeploy:GetDeployment",
        "codedeploy:GetApplicationRevision",
        "codedeploy:RegisterApplicationRevision"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Project": "web-api"
        }
      }
    },
    {
      "Effect": "Allow",
      "Action": "codestar-connections:UseConnection",
      "Resource": "arn:aws:codestar-connections:ap-southeast-1:123456789012:connection/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::web-tinh-demo-prod",
        "arn:aws:s3:::web-tinh-demo-prod/*"
      ]
    }
  ]
}
```

Trust policy (ai được assume role này):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "codepipeline.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### 11.2. CodeBuild service role (least privilege)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents"],
      "Resource": "arn:aws:logs:ap-southeast-1:123456789012:log-group:/aws/codebuild/web-api-build:*"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:GetObjectVersion", "s3:PutObject"],
      "Resource": "arn:aws:s3:::cong-ty-pipeline-artifacts/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload",
        "ecr:PutImage"
      ],
      "Resource": "arn:aws:ecr:ap-southeast-1:123456789012:repository/web-api"
    },
    {
      "Effect": "Allow",
      "Action": ["ssm:GetParameters", "secretsmanager:GetSecretValue"],
      "Resource": [
        "arn:aws:ssm:ap-southeast-1:123456789012:parameter/prod/web-api/*",
        "arn:aws:secretsmanager:ap-southeast-1:123456789012:secret:prod/web-api/*"
      ]
    }
  ]
}
```

> **Lưu ý:** `ecr:GetAuthorizationToken` bắt buộc `Resource: "*"` (ECR không hỗ trợ giới hạn resource cho action này). Nếu policy validator báo lỗi, tách riêng statement với `"Resource": "*"`.

### 11.3. Nguyên tắc least privilege — checklist

- [ ] Không `Action: "*"` / `Resource: "*"` trừ khi AWS bắt buộc (như `ecr:GetAuthorizationToken`).
- [ ] Giới hạn S3 đúng bucket + prefix; tách role riêng cho pipeline / build / deploy.
- [ ] Secret qua **SSM Parameter Store / Secrets Manager**, không hardcode trong buildspec.
- [ ] Bật **CloudTrail** audit; định kỳ review bằng **IAM Access Analyzer**.

---

## 12. Cross-account pipelines

Pipeline chạy ở account **CI** (`111122223333`), deploy sang account **Prod** (`999988887777`) — mô hình tách môi trường bằng account.

### 12.1. Tổng quan kiến trúc

```
Account CI (111122223333)                 Account PROD (999988887777)
┌─────────────────────┐                   ┌──────────────────────┐
│ CodePipeline        │                   │ ECS service (prod)   │
│ Artifact bucket     │ ──cross-account──▶│ KMS key (prod)       │
│ (KMS CMK của CI)    │   deploy action   │ Deploy role (prod)   │
└─────────────────────┘                   └──────────────────────┘
```

### 12.2. Các bước cấu hình

**Bước 1 — Account CI:** artifact bucket dùng KMS CMK; key policy cho phép account PROD dùng:

```json
{
  "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::999988887777:root" },
  "Action": [
    "kms:Encrypt",
    "kms:Decrypt",
    "kms:ReEncrypt*",
    "kms:GenerateDataKey*",
    "kms:DescribeKey"
  ],
  "Resource": "*"
}
```

**Bước 2 — Account CI:** bucket policy cho phép deploy role PROD đọc/ghi (mẫu mục 10.3, thay Principal bằng role PROD).

**Bước 3 — Account PROD:** tạo deploy role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::111122223333:root" },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "pipeline-deploy-prod"
        }
      }
    }
  ]
}
```

> Dùng **ExternalId** để chống confused deputy attack.

**Bước 4 — Account CI:** pipeline role được phép assume deploy role

```json
{
  "Effect": "Allow",
  "Action": "sts:AssumeRole",
  "Resource": "arn:aws:iam::999988887777:role/prod-deploy-role"
}
```

**Bước 5 — Deploy action khai báo role cross-account:**

```yaml
- Name: DeployToProd
  ActionTypeId:
    Category: Deploy
    Owner: AWS
    Provider: ECS
    Version: "1"
  InputArtifacts:
    - Name: BuildArtifact
  RoleArn: arn:aws:iam::999988887777:role/prod-deploy-role   # <-- role ben account PROD
  Configuration:
    ClusterName: prod-cluster
    ServiceName: web-api-service
    FileName: imagedefinitions.json
```

### 12.3. Những điểm dễ sai

| Vấn đề | Cách fix |
|---|---|
| `AccessDenied` khi decrypt artifact | Key policy của KMS CMK chưa cho account PROD `kms:Decrypt` |
| Dùng AWS managed key `aws/s3` | Không sửa được key policy → bắt buộc tạo CMK riêng |
| Deploy action báo không assume được role | Thiếu `sts:AssumeRole` ở pipeline role, hoặc trust policy bên PROD sai ExternalId |
| ECS deploy cross-region + cross-account | `RoleArn` hỗ trợ cả hai, nhưng artifact bucket nên ở cùng region với pipeline |

---

## 13. Ví dụ thực tế

### 13.1. Ví dụ 1: Web app — GitHub → CodeBuild → S3 + CloudFront

**Kịch bản:** web React (Vite) ở GitHub `acme-corp/landing-page`, deploy lên S3 + CloudFront; push lên `main` tự build và deploy.

**`buildspec.yml`** (`VITE_API_URL` truyen qua bien moi truong):

```yaml
version: 0.2

env:
  variables:
    VITE_API_URL: https://api.acme.vn

phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - npm ci
  pre_build:
    commands:
      - npm run lint
      - npm run test -- --run
  build:
    commands:
      - npm run build
      - ls -la dist/
  post_build:
    commands:
      - echo "Build xong, san sang deploy len S3"

artifacts:
  files:
    - "**/*"
  base-directory: dist
  discard-paths: no

cache:
  paths:
    - node_modules/**/*
```

**CloudFormation — pipeline + CloudFront invalidation bằng Lambda action:**

Sau S3 deploy, cần invalidate CloudFront để user thấy bản mới. Thêm action Lambda:

```yaml
  DeployStage:
    Type: AWS::CodePipeline::Pipeline
    # ... (Source, Build giong muc 5)
```

Action Lambda trong stage Deploy (chạy sau S3 deploy, `RunOrder: 2`):

```yaml
- Name: InvalidateCloudFront
  ActionTypeId:
    Category: Invoke
    Owner: AWS
    Provider: Lambda
    Version: "1"
  InputArtifacts:
    - Name: BuildArtifact
  Configuration:
    FunctionName: invalidate-cloudfront-fn
    UserParameters: '{"distributionId":"E1A2B3C4D5E6F7"}'
  RunOrder: 2
```

**Lambda function `invalidate-cloudfront-fn` (Python):**

```python
import json
import time
import boto3

cp = boto3.client("codepipeline")

def lambda_handler(event, context):
    job = event["CodePipeline.job"]
    job_id = job["id"]
    cfg = job["data"]["actionConfiguration"]["configuration"]
    dist_id = json.loads(cfg["UserParameters"])["distributionId"]
    try:
        boto3.client("cloudfront").create_invalidation(
            DistributionId=dist_id,
            InvalidationBatch={
                "Paths": {"Quantity": 1, "Items": ["/*"]},
                "CallerReference": f"{job_id}-{int(time.time())}",
            },
        )
        cp.put_job_success_result(jobId=job_id)
    except Exception as e:
        cp.put_job_failure_result(
            jobId=job_id,
            failureDetails={"message": str(e)[:255], "type": "JobFailed"},
        )
```

> Lambda được CodePipeline invoke **phải** gọi `put_job_success_result` / `put_job_failure_result` — nếu không action treo tới timeout.

**Pipeline hoàn chỉnh:** `Source` (GitHub) → `Build` (CodeBuild: npm build) → `Deploy` (S3) → `Invoke` (Lambda invalidate CloudFront).

### 13.2. Ví dụ 2: Container app — ECR → CodeBuild → ECS (Fargate)

**Kịch bản:** API Node.js trên ECS Fargate: GitHub → CodeBuild (docker build + push ECR) → ECS rolling deploy; staging tự động, prod cần approval.

**`Dockerfile`:**

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/src ./src
COPY --from=builder /app/package.json ./
EXPOSE 8080
USER node
CMD ["node", "src/index.js"]
```

**`buildspec.yml`** (đã có ở mục 7.3 — build, push ECR, sinh `imagedefinitions.json`).

**Pipeline CloudFormation (trích phần Stages):**

```yaml
Stages:
  - Name: Source
    Actions:
      - Name: Source
        ActionTypeId:
          Category: Source
          Owner: AWS
          Provider: CodeStarSourceConnection
          Version: "1"
        OutputArtifacts:
          - Name: SourceArtifact
        Configuration:
          ConnectionArn: !Ref GitHubConnectionArn
          FullRepositoryId: acme-corp/web-api
          BranchName: main
          OutputArtifactFormat: CODE_ZIP
  - Name: Build
    Actions:
      - Name: BuildAndPush
        ActionTypeId:
          Category: Build
          Owner: AWS
          Provider: CodeBuild
          Version: "1"
        InputArtifacts:
          - Name: SourceArtifact
        OutputArtifacts:
          - Name: BuildArtifact   # chua imagedefinitions.json
        Configuration:
          ProjectName: !Ref BuildProject
  - Name: DeployStaging
    Actions:
      - Name: DeployStaging
        ActionTypeId:
          Category: Deploy
          Owner: AWS
          Provider: ECS
          Version: "1"
        InputArtifacts:
          - Name: BuildArtifact
        Configuration:
          ClusterName: acme-staging
          ServiceName: web-api-service
          FileName: imagedefinitions.json
  - Name: Approval
    Actions:
      - Name: ApproveProd
        ActionTypeId:
          Category: Approval
          Owner: AWS
          Provider: Manual
          Version: "1"
        Configuration:
          NotificationArn: !Ref ApproveTopicArn
          CustomData: "Staging da deploy image #{BuildAndPush.IMAGE_TAG}. Duyet de len PROD?"
  - Name: DeployProd
    Actions:
      - Name: DeployProd
        ActionTypeId:
          Category: Deploy
          Owner: AWS
          Provider: ECS
          Version: "1"
        InputArtifacts:
          - Name: BuildArtifact
        Configuration:
          ClusterName: acme-prod
          ServiceName: web-api-service
          FileName: imagedefinitions.json
```

> `#{BuildAndPush.IMAGE_TAG}`: biến `IMAGE_TAG` export từ buildspec (`env.exported-variables`); cú pháp `#{TênAction.TênBiến}`.

**Kiểm tra sau deploy:** `aws ecs describe-services --cluster acme-prod --services web-api-service --query "services[0].deployments"` và `aws logs tail /ecs/web-api --follow`.

---

## 14. Pricing (giá cả)

### 14.1. Giá CodePipeline

| Hạng mục | Giá (region ap-southeast-1 / Singapore) |
|---|---|
| **Active pipeline** | **$1.00** / pipeline / tháng |
| **Free tier** | **1 active pipeline miễn phí** mỗi tháng (12 tháng đầu cho tài khoản mới) |

**"Active pipeline" là gì?** Pipeline có **ít nhất 1 execution trong 30 ngày qua**. Pipeline nằm im không chạy → không tính tiền.

Ví dụ: bạn có 5 pipeline, tháng này 3 pipeline có chạy → trả $3 (trừ 1 free nếu còn trong free tier).

### 14.2. Chi phí liên quan (thường tốn hơn tiền pipeline)

| Dịch vụ | Giá tham khảo | Ghi chú |
|---|---|---|
| **CodeBuild** | $0.005/phút (small, Linux) | Build 10 phút/ngày × 22 ngày ≈ $1.1/tháng |
| **S3 artifact** | $0.025/GB/tháng (Singapore) | Artifact zip thường vài chục MB — không đáng kể nếu có lifecycle rule |
| **CodeDeploy** | Miễn phí deploy lên EC2/Lambda | Deploy lên on-premise: $0.02/instance/update |
| **SNS** (approval notify) | 1000 email đầu miễn phí | Gần như không tốn |
| **CloudWatch Logs** | $0.60/GB ingest | Log build nhiều có thể tốn — đặt retention hợp lý |

### 14.3. Ví dụ tính chi phí thực tế / tháng

Ví dụ: 3 pipeline, mỗi pipeline build ~8 phút, deploy ~20 lần/tháng:

| Hạng mục | Tính toán | Chi phí |
|---|---|---|
| CodePipeline | 3 active − 1 free = 2 × $1 | $2.00 |
| CodeBuild | 3 × 20 × 8 phút × $0.005 | $2.40 |
| S3 artifact | ~5 GB × $0.025 | $0.13 |
| **Tổng** | | **≈ $4.5/tháng** |

> Khi số pipeline tăng (mỗi microservice một pipeline), $1/pipeline cộng dồn — 50 pipeline = $50/tháng chưa kể build.

---

## 15. Troubleshooting lỗi thường gặp

### 15.1. Lỗi permissions

**Triệu chứng:** Action báo `AccessDenied`, `is not authorized to perform`, pipeline dừng ở stage.

| Lỗi | Cách fix |
|---|---|
| `... not authorized: s3:PutObject` (pipeline role) | Thêm `s3:PutObject` đúng bucket ARN (mục 11.1) |
| `... not authorized: codebuild:StartBuild` | Thêm ARN project vào policy, kiểm tra đúng region |
| `... not authorized: codestar-connections:UseConnection` | Thêm quyền với đúng ARN connection |
| Build: `AccessDenied ... ecr:GetAuthorizationToken` | Action này bắt buộc `Resource: "*"` |
| Build: `AccessDenied ... secretsmanager:GetSecretValue` | Thêm ARN secret vào policy; KMS key của secret phải cho role decrypt |
| Deploy: `... not authorized: lambda:UpdateFunctionCode` | Thêm `lambda:UpdateFunctionCode`, `lambda:GetFunction` đúng function ARN |

**Debug nhanh:** `aws iam list-role-policies --role-name <role>` để xem policy; `aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=StartBuild` để xem call nào bị từ chối.

### 15.2. Lỗi artifacts

| Triệu chứng | Cách fix |
|---|---|
| `artifact not found` / `BuildArtifact does not exist` | `OutputArtifacts` → `InputArtifacts` phải khớp từng ký tự (phân biệt hoa thường) |
| CodeDeploy: `appspec.yml not found` | Copy `appspec.yml` vào `base-directory` trước khi đóng gói; kiểm tra bằng `unzip -l artifact.zip` |
| S3 deploy sai cấu trúc thư mục | Muốn giữ cấu trúc: `discard-paths: no` + `Extract: "true"` |
| ECS: `imagedefinitions.json not found` | Thêm file vào `artifacts.files`; `cat` ở `post_build` để xác nhận |
| Artifact quá lớn, build/deploy chậm | Chỉ lấy `dist/`, loại trừ `node_modules`, `.git` |

**Kiểm tra artifact thực tế:** tìm S3 key trong artifact bucket rồi `aws s3 cp s3://<bucket>/<key>.zip /tmp/a.zip && unzip -l /tmp/a.zip`.

### 15.3. Lỗi buildspec

| Triệu chứng | Cách fix |
|---|---|
| `YAML_FILE_ERROR` | Sai indent — dùng 2 spaces, kiểm tra bằng `yamllint` |
| `COMMAND_EXECUTION_ERROR ... exit code 1` | Một lệnh fail — xem log CloudWatch, reproduce local |
| `Cannot connect to the Docker daemon` | Bật **Privileged** (Project → Environment) |
| `exported-variables` không sang pipeline | `export` biến trong `pre_build`/`build`; tên phải khớp |
| `npm ci` fail: lock file out of sync | Chạy `npm install` local rồi commit lại lock file |

### 15.4. Lỗi trigger, source & deploy

| Triệu chứng | Cách fix |
|---|---|
| Push code mà pipeline không chạy | Connections → kiểm tra trạng thái **Available**; bấm **Update pending connection** nếu cần |
| Trigger 2 lần cho 1 push | Đặt `PollForSourceChanges: "false"` (đang dùng cả EventBridge lẫn polling) |
| Trigger filter không ăn | `GitConfiguration.SourceActionName` phải khớp **tên action** Source |
| CodeDeploy treo ở hook | `chmod +x scripts/*.sh` rồi commit (git giữ execute bit) |
| CodeDeploy: `Health check failed` | SSH vào instance xem `/opt/codedeploy-agent/deployment-root/.../logs/scripts.log` |
| ECS deploy xong nhưng task cũ vẫn chạy | `name` trong `imagedefinitions.json` phải khớp **container name** trong task definition |
| CloudFormation: `No updates are to be performed` | Không phải lỗi — template không đổi; muốn ép thay đổi thì đổi parameter |

---

## 16. Best practices

### Thiết kế pipeline

1. **Một pipeline một mục đích**: tách theo service, trigger theo `FilePaths` (`services/api/**`).
2. **Fail fast**: lint + unit test ở `pre_build` hoặc action Test song song.
3. **Promotion theo môi trường**: `dev → staging (tự động) → approval → prod`; **immutable artifact** — build 1 lần, đi qua mọi môi trường.

### Bảo mật

4. **Least privilege** cho cả 3 role (mục 11); secret qua **SSM/Secrets Manager**, không hardcode.
5. **KMS CMK riêng** cho artifact bucket production; bật CloudTrail audit.
6. Deploy prod ở **account riêng** (cross-account, mục 12) + bắt buộc manual approval.

### Vận hành

7. **Lifecycle rule** cho artifact bucket (xoá sau 30–90 ngày).
8. **CloudWatch alarm** khi pipeline fail; **đặt tên nhất quán** `{service}-{env}-pipeline`.
9. **Version hoá pipeline bằng CloudFormation/CDK** — không click tay cho production.
10. **Rollback plan**: ECS giữ task definition cũ; S3 bật versioning; CodeDeploy auto-rollback khi health check fail.

### Tối ưu chi phí & tốc độ

11. Dùng **cache** CodeBuild (`node_modules`, Docker layer cache); chọn `ComputeType` vừa đủ.
12. Pipeline không dùng → **xoá** (hoặc để 30 ngày không execution để khỏi tính $1).

---

## 17. Cheat sheet

### CLI

```bash
aws codepipeline list-pipelines

aws codepipeline get-pipeline-state --name web-api-pipeline

aws codepipeline start-pipeline-execution --name web-api-pipeline

aws codepipeline stop-pipeline-execution \
  --pipeline-name web-api-pipeline \
  --pipeline-execution-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
  --reason "Dung de kiem tra loi build"

aws codepipeline retry-stage-execution \
  --pipeline-name web-api-pipeline \
  --stage-name Build \
  --pipeline-execution-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx \
  --retry-mode FAILED_ACTIONS

aws codepipeline list-pipeline-executions \
  --pipeline-name web-api-pipeline --max-items 10

aws codepipeline get-pipeline-execution \
  --pipeline-name web-api-pipeline \
  --pipeline-execution-id xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx

aws codepipeline get-pipeline --name web-api-pipeline > pipeline.json

aws codepipeline update-pipeline --cli-input-json file://pipeline.json

aws codepipeline put-approval-result \
  --pipeline-name web-api-pipeline --stage-name Approval \
  --action-name ApproveProd --token <token> \
  --result status=Approved,summary="OK"

aws codepipeline delete-pipeline --name web-api-pipeline-cu
```

### Cấu trúc action trong CloudFormation (mẫu nhớ nhanh)

```yaml
- Name: TenAction
  ActionTypeId:
    Category: Source | Build | Test | Deploy | Approval | Invoke
    Owner: AWS
    Provider: CodeStarSourceConnection | CodeBuild | CodeDeploy | ECS | Lambda | S3 | CloudFormation | Manual
    Version: "1"
  InputArtifacts:            # khong co o Source / Approval
    - Name: SourceArtifact
  OutputArtifacts:           # khong co o Deploy / Approval
    - Name: BuildArtifact
  Configuration:
    ProjectName: ten-project
  RoleArn: arn:...            # cross-account
  RunOrder: 1                # tang dan de chay tuan tu
  Region: ap-southeast-1     # khi action o region khac
```

### Biến pipeline

```
#{Source.CommitId}              # bien mac dinh cua source action (GitHub/Connection)
#{Source.BranchName}
#{Source.RepositoryName}
#{Build.BUILD_VERSION}          # bien export tu buildspec (env.exported-variables)
#{codepipeline.pipelineExecutionId}   # bien he thong cua pipeline
```

Dùng trong `CustomData` của approval, `UserParameters` của Lambda, `ParameterOverrides` của CloudFormation.

### buildspec.yml tối thiểu (nhớ nhanh)

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 20
    commands:
      - npm ci
  build:
    commands:
      - npm run build
artifacts:
  files:
    - "**/*"
  base-directory: dist
```

### Ma trận provider hay dùng

| Muốn làm gì | Category | Provider | Artifact vào |
|---|---|---|---|
| Lấy code GitHub | Source | CodeStarSourceConnection | → `SourceArtifact` |
| Lấy code CodeCommit | Source | CodeCommit | → `SourceArtifact` |
| Lấy image ECR | Source | ECR | → `imageDetail.json` |
| Build/test | Build/Test | CodeBuild | `SourceArtifact` → `BuildArtifact` |
| Deploy EC2 | Deploy | CodeDeploy | zip có `appspec.yml` |
| Deploy ECS rolling | Deploy | ECS | `imagedefinitions.json` |
| Deploy ECS blue/green | Deploy | CodeDeployToECS | `taskdef.json` + `appspec.yaml` |
| Deploy Lambda | Deploy | Lambda | zip code |
| Deploy web tĩnh | Deploy | S3 | zip web |
| Deploy stack/SAM | Deploy | CloudFormation | template |
| Chờ người duyệt | Approval | Manual | — |
| Chạy hàm tuỳ chỉnh | Invoke | Lambda | tuỳ ý |

### Checklist trước khi xong pipeline

- [ ] Push code → tự trigger; tên artifact `Input`/`Output` khớp nhau
- [ ] IAM least privilege; artifact bucket có versioning + mã hoá + lifecycle rule
- [ ] Secret từ SSM/Secrets Manager; có approval trước khi deploy prod
- [ ] CloudWatch alarm khi fail; pipeline bằng CloudFormation/CDK; đã test rollback

---

*Hết hướng dẫn. Chúc bạn deploy mượt mà, không lỗi!* 🚀
