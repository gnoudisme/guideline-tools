# Hướng Dẫn Jenkins Toàn Tập (CI/CD)

> Tài liệu tiếng Việt chi tiết từ A–Z: cài đặt, cấu hình, viết Pipeline, vận hành và khắc phục sự cố Jenkins.

---

## Mục lục

1. [Jenkins là gì? Kiến trúc](#1-jenkins-là-gì-kiến-trúc)
2. [Cài đặt Jenkins](#2-cài-đặt-jenkins)
3. [Setup Wizard lần đầu](#3-setup-wizard-lần-đầu)
4. [Job đầu tiên: Freestyle vs Pipeline](#4-job-đầu-tiên-freestyle-vs-pipeline)
5. [Declarative vs Scripted Pipeline](#5-declarative-vs-scripted-pipeline)
6. [Jenkinsfile ví dụ hoàn chỉnh](#6-jenkinsfile-ví-dụ-hoàn-chỉnh)
7. [Quản lý Plugins](#7-quản-lý-plugins)
8. [Agents / Nodes](#8-agents--nodes)
9. [Credentials](#9-credentials)
10. [Blue Ocean UI](#10-blue-ocean-ui)
11. [Shared Libraries](#11-shared-libraries)
12. [Multibranch Pipeline](#12-multibranch-pipeline)
13. [Webhooks từ GitHub / GitLab](#13-webhooks-từ-github--gitlab)
14. [Backup & Restore](#14-backup--restore)
15. [Bảo mật cơ bản](#15-bảo-mật-cơ-bản)
16. [Troubleshooting](#16-troubleshooting)
17. [Best Practices Checklist](#17-best-practices-checklist)
18. [FAQ + Cheat Sheet](#18-faq--cheat-sheet)

---

## 1. Jenkins là gì? Kiến trúc

### 1.1. Jenkins là gì?

**Jenkins** là công cụ **CI/CD (Continuous Integration / Continuous Delivery)** mã nguồn mở phổ biến nhất thế giới, viết bằng Java, ra mắt năm 2011 (fork từ Hudson). Jenkins tự động hoá toàn bộ vòng đời phần mềm:

- **Pull code** từ Git/GitHub/GitLab/Bitbucket khi có thay đổi
- **Build** (biên dịch, đóng gói): Maven, Gradle, npm, Docker…
- **Test** tự động: unit test, integration test
- **Phân tích chất lượng**: SonarQube, kiểm tra bảo mật
- **Deploy** lên môi trường dev / staging / production
- **Gửi thông báo**: email, Slack, Telegram khi build pass/fail

Thay vì dev phải làm thủ công từng bước (`git pull` → `npm install` → `npm test` → `docker build` → `kubectl apply`…), Jenkins chạy tất cả tự động theo lịch hoặc theo sự kiện (mỗi lần push code).

### 1.2. Kiến trúc Controller / Agents

Từ phiên bản 2.x, thuật ngữ chính thức là **Controller** (trước đây gọi là *master*) và **Agents** (trước đây gọi là *slaves*).

```
┌─────────────────────────────────────────────────────────┐
│                    JENKINS CONTROLLER                    │
│  ┌────────────┐  ┌──────────┐  ┌──────────┐              │
│  │  Web UI    │  │ Scheduler│  │  Plugins │              │
│  │  (:8080)   │  │ (Cron /  │  │ (Git,    │              │
│  │            │  │ Webhook) │  │ Docker… )│              │
│  └────────────┘  └──────────┘  └──────────┘              │
│  ┌────────────────────────────────────────────────┐      │
│  │  JENKINS_HOME (/var/jenkins_home)               │      │
│  │  - jobs/ (định nghĩa job + lịch sử build)       │      │
│  │  - plugins/  - users/  - credentials            │      │
│  └────────────────────────────────────────────────┘      │
└──────────────┬────────────┬────────────┬─────────────────┘
               │ TCP        │ TCP        │ TCP
        (JNLP / SSH)  (JNLP / SSH)  (JNLP / SSH)
               ▼            ▼            ▼
        ┌──────────┐  ┌──────────┐  ┌──────────┐
        │  Agent 1 │  │  Agent 2 │  │  Agent 3 │
        │ (Linux)  │  │(Windows) │  │ (macOS)  │
        │ label:   │  │ label:   │  │ label:   │
        │ docker   │  │ dotnet   │  │ ios      │
        └──────────┘  └──────────┘  └──────────┘
```

**Controller** làm gì:
- Chạy giao diện web (`http://<host>:8080`)
- Quản lý jobs, lịch trình (triggers, cron), users, permissions
- Lưu trữ toàn bộ cấu hình và lịch sử build trong `JENKINS_HOME`
- Điều phối công việc: quyết định job nào chạy ở agent nào

**Agent** làm gì:
- Là máy (vật lý / VM / container) thực thi build thật
- Nhận lệnh từ controller, checkout code, chạy `mvn test`, `docker build`…
- Kết nối về controller qua **SSH**, **JNLP** (Java Web Start / inbound agent) hoặc **Docker**
- Có thể gán **label** (nhãn) để job chọn đúng agent: ví dụ job iOS chỉ chạy trên agent macOS

**Vì sao tách Controller / Agents?**
- Giảm tải cho controller (build nặng như Docker, Maven không ăn CPU của controller)
- Cách ly môi trường: mỗi agent có thể cài toolchain khác nhau (Node 20, JDK 17, .NET…)
- Mở rộng ngang: thêm agent khi cần nhiều build song song
- An toàn: controller không chạy code lạ từ PR

**Thuật ngữ hay gặp:**

| Thuật ngữ | Nghĩa |
|---|---|
| Job / Project | Một công việc CI/CD (build, test, deploy…) |
| Build | Một lần chạy của job (build #1, #2, #3…) |
| Pipeline | Job định nghĩa bằng code (Jenkinsfile) |
| Stage | Một giai đoạn: Build → Test → Deploy |
| Step | Một lệnh cụ thể trong stage: `sh 'mvn test'` |
| Node | Controller hoặc agent — nơi thực thi pipeline |
| Executor | "Khe" chạy job trên một node |
| Workspace | Thư mục làm việc của một build trên agent |
| Artifact | File kết quả build cần lưu lại (.jar, .zip…) |

---

## 2. Cài đặt Jenkins

### 2.1. Yêu cầu hệ thống

- **Java**: Jenkins 2.x yêu cầu **Java 11 hoặc 17** (khuyến nghị Java 17 cho bản mới nhất). Java 8 đã ngừng hỗ trợ từ Jenkins 2.357+.
- **RAM**: tối thiểu 1 GB cho controller demo, **4 GB+** cho production; heap mặc định 256MB là quá ít — nên set `-Xmx2g` trở lên.
- **Disk**: 10 GB trống tối thiểu; build nhiều / artifact lớn cần hàng chục–trăm GB.
- **Port**: 8080 (web UI), 50000 (agent JNLP inbound).
- **OS**: Linux (khuyến nghị), Windows, macOS.

Kiểm tra Java trước khi cài:

```bash
java -version
# Cần thấy: openjdk version "17.x.x"
```

Nếu chưa có, cài Java 17:

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install -y openjdk-17-jdk

# RHEL/CentOS/Fedora
sudo yum install -y java-17-openjdk-devel
```

### 2.2. Cách 1 — Chạy bằng Docker (nhanh nhất, khuyên dùng)

```bash
docker run -d \
  --name jenkins \
  --restart unless-stopped \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts-jdk17
```

Giải thích tham số:

| Tham số | Ý nghĩa |
|---|---|
| `-d` | Chạy nền (detached) |
| `--restart unless-stopped` | Tự khởi động lại khi máy reboot |
| `-p 8080:8080` | Map cổng web UI |
| `-p 50000:50000` | Map cổng agent JNLP |
| `-v jenkins_home:/var/jenkins_home` | Volume lưu dữ liệu (config, jobs, plugins) — **bắt buộc**, không là mất hết khi xóa container |
| `jenkins/jenkins:lts-jdk17` | Image LTS chính thức, kèm JDK 17 |

Dùng Docker Compose (gọn hơn, dễ thêm agent về sau):

```yaml
# docker-compose.yml
version: "3.8"
services:
  jenkins:
    image: jenkins/jenkins:lts-jdk17
    restart: unless-stopped
    ports: ["8080:8080", "50000:50000"]
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock  # build Docker từ trong Jenkins
    environment:
      - JAVA_OPTS=-Xmx2g -Xms1g
volumes:
  jenkins_home:
```

```bash
docker compose up -d
docker logs -f jenkins   # xem log, lấy initial admin password
```

> **Lưu ý:** mount `/var/run/docker.sock` cho phép container Jenkins gọi Docker của host — tiện nhưng cần hiểu rủi ro bảo mật (xem mục 15).

### 2.3. Cách 2 — Cài từ package (apt / yum)

**Ubuntu/Debian:**

```bash
# Thêm key và repo chính thức
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | \
  sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install -y jenkins

# Khởi động và tự chạy khi boot
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

**RHEL/CentOS/Rocky/Fedora:**

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
  https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key

sudo yum install -y jenkins
sudo systemctl enable --now jenkins
```

Mặc định sau khi cài package:
- Service chạy dưới user `jenkins`
- `JENKINS_HOME=/var/lib/jenkins`
- Web UI: `http://<server-ip>:8080`
- Mở firewall nếu cần: `sudo ufw allow 8080` hoặc `firewall-cmd --add-port=8080/tcp --permanent`

### 2.4. Cách 3 — Chạy file .war trực tiếp

Cách cổ điển, phù hợp test nhanh hoặc môi trường không có Docker/apt:

```bash
# Tải bản LTS mới nhất
wget https://get.jenkins.io/war-stable/latest/jenkins.war

# Chạy (JENKINS_HOME mặc định là ~/.jenkins)
java -jar jenkins.war

# Tùy chỉnh port và thư mục home:
JENKINS_HOME=/opt/jenkins_home java -jar jenkins.war --httpPort=9090
```

Để chạy nền + tự khởi động, nên bọc bằng systemd:

```ini
# /etc/systemd/system/jenkins.service
[Unit]
Description=Jenkins CI
After=network.target

[Service]
Type=simple
User=jenkins
Environment="JENKINS_HOME=/opt/jenkins_home"
Environment="JAVA_OPTS=-Xmx2g"
ExecStart=/usr/bin/java -jar /opt/jenkins/jenkins.war --httpPort=8080
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now jenkins
```

### 2.5. So sánh 3 cách cài

| Tiêu chí | Docker | Package (apt/yum) | File .war |
|---|---|---|---|
| Độ dễ | Rất dễ | Dễ | Trung bình |
| Cô lập môi trường | Tốt | Kém | Kém |
| Nâng cấp | `docker pull` + recreate | `apt upgrade` | Tải war mới, restart |
| Backup | Volume `jenkins_home` | `/var/lib/jenkins` | `JENKINS_HOME` tùy chọn |
| Phù hợp | Dev, test, production nhỏ | Server Linux production | Môi trường đặc thù |

### 2.6. Cấu hình reverse proxy (Nginx)

Không nên expose :8080 trực tiếp ra internet. Dùng Nginx + HTTPS (rút gọn):

```nginx
# /etc/nginx/sites-available/jenkins
upstream jenkins {
    server 127.0.0.1:8080 fail_timeout=0;
}

server {
    listen 443 ssl;
    server_name jenkins.example.com;

    ssl_certificate     /etc/ssl/certs/jenkins.crt;
    ssl_certificate_key /etc/ssl/private/jenkins.key;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://jenkins;
        # Bắt buộc cho WebSocket (log console realtime)
        proxy_http_version 1.1;
        proxy_request_buffering off;
    }
}
```

Trong Jenkins: **Manage Jenkins → System → Jenkins URL** đặt thành `https://jenkins.example.com/` để webhook/redirect hoạt động đúng.

---

## 3. Setup Wizard lần đầu

Mở `http://<host>:8080`:

1. **Unlock**: lấy mật khẩu ở `secrets/initialAdminPassword` (Docker: `docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword`; package: `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`), dán vào → Continue.
2. **Plugins**: chọn **Install suggested plugins** (Git, Pipeline, Blue Ocean…) — cài thêm sau cũng được.
3. **Admin user**: tạo tài khoản admin riêng (không dùng "admin" nếu expose internet), mật khẩu mạnh.
4. **Jenkins URL**: giữ nguyên nếu chạy local; sửa thành domain/HTTPS nếu có reverse proxy → **Save and Finish**.

> Tài khoản tạm ở bước unlock bị vô hiệu hóa sau khi wizard xong.

---

## 4. Job đầu tiên: Freestyle vs Pipeline

### 4.1. Freestyle Project

**Freestyle** là loại job cổ điển: cấu hình hoàn toàn bằng form click trên web UI, không cần viết code.

**Tạo job Freestyle đầu tiên:**

1. Dashboard → **New Item** → nhập tên `hello-freestyle` → chọn **Freestyle project** → OK
2. Tab **Build Steps** → **Add build step** → **Execute shell**:
   ```bash
   echo "Xin chào từ Jenkins!"
   echo "Build số $BUILD_NUMBER trên node $NODE_NAME"
   ```
3. **Save** → bấm **Build Now** → vào build vừa chạy → **Console Output** xem kết quả.

**Ưu điểm Freestyle:** dễ học, click là xong, hợp job đơn giản (script backup, cron dọn dẹp…).

**Nhược điểm:** cấu hình nằm trong Jenkins (không version-control), khó review, khó copy giữa môi trường.

### 4.2. Pipeline Job

**Pipeline** định nghĩa toàn bộ quy trình CI/CD bằng **code** (Groovy DSL) trong file **Jenkinsfile** — lưu cùng repo code, review như code thường.

**Tạo Pipeline job đầu tiên:**

1. **New Item** → tên `hello-pipeline` → chọn **Pipeline** → OK
2. Kéo xuống mục **Pipeline** → **Definition: Pipeline script** → dán:

```groovy
pipeline {
    agent any
    stages {
        stage('Chao hoi') {
            steps {
                echo 'Xin chào từ Pipeline!'
                sh 'date && whoami'
            }
        }
        stage('Tinh toan') {
            steps {
                sh 'echo "2 + 2 = $((2+2))"'
            }
        }
    }
    post {
        always {
            echo 'Pipeline kết thúc (dù pass hay fail).'
        }
    }
}
```

3. **Save** → **Build Now** → xem **Console Output** và biểu đồ **Stage View**.

### 4.3. So sánh Freestyle vs Pipeline

| Tiêu chí | Freestyle | Pipeline |
|---|---|---|
| Định nghĩa | Form trên web UI | Code (Jenkinsfile, Groovy DSL) |
| Version control | Không (nằm trong Jenkins) | Có — lưu cùng repo |
| Code review | Không | Có (pull request) |
| Job phức tạp (song song, điều kiện) | Rất khó | Dễ (`parallel`, `when`) |
| Tái sử dụng | Copy job thủ công | Shared Libraries |
| Phù hợp | Job đơn giản, script một lần | Mọi dự án nghiêm túc |

> **Khuyến nghị:** học Freestyle để hiểu khái niệm, nhưng dự án thật luôn dùng **Pipeline + Jenkinsfile**.

---

## 5. Declarative vs Scripted Pipeline

Pipeline có 2 phong cách viết:

### 5.1. Declarative Pipeline (khuyên dùng)

Cú pháp khai báo, có cấu trúc chặt chẽ, dễ đọc, có kiểm tra lỗi cú pháp. Bắt đầu bằng `pipeline { }`.

```groovy
pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')   // timeout toàn pipeline
        buildDiscarder(logRotator(numToKeepStr: '10')) // giữ 10 build
        timestamps()                          // thêm timestamp vào log
    }

    environment {
        APP_NAME = 'my-app'
        VERSION  = '1.0.0'
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Nhánh cần build')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Chạy test?')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Môi trường deploy')
    }

    triggers {
        cron('H 2 * * *')        // chạy định kỳ 2h sáng
        pollSCM('H/5 * * * *')   // kiểm tra SCM mỗi 5 phút
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
        stage('Test') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                sh 'make test'
            }
            post {
                always {
                    junit 'reports/**/*.xml'   // publish kết quả test
                }
            }
        }
        stage('Deploy') {
            when {
                branch 'main'   // chỉ deploy khi nhánh main
            }
            steps {
                sh './deploy.sh ${ENV}'
            }
        }
    }

    post {
        always  { echo "Luôn chạy" }
        success { echo "Chỉ khi PASS" }
        failure { echo "Chỉ khi FAIL" }
        unstable { echo "Chỉ khi UNSTABLE (test fail nhưng build ok)" }
        changed { echo "Khi trạng thái đổi so với build trước" }
    }
}
```

### 5.2. Scripted Pipeline (linh hoạt tối đa)

Viết như code Groovy thuần, bắt đầu bằng `node { }`. Mạnh nhưng khó đọc, không kiểm tra cú pháp lúc lưu.

```groovy
node('linux') {
    try {
        stage('Checkout') {
            checkout scm
        }
        stage('Build') {
            sh 'make build'
        }
        // Logic Groovy tự do: vòng lặp, try/catch, hàm...
        def services = ['api', 'worker', 'web']
        stage('Test song song') {
            def jobs = [:]
            services.each { svc ->
                jobs[svc] = { sh "./test.sh ${svc}" }
            }
            parallel jobs
        }
    } catch (err) {
        echo "Build lỗi: ${err}"
        currentBuild.result = 'FAILURE'
        throw err
    } finally {
        echo 'Dọn dẹp...'
    }
}
```

### 5.3. Bảng cú pháp đầy đủ (Declarative)

| Khối | Vị trí | Ý nghĩa |
|---|---|---|
| `pipeline` | Ngoài cùng | Bắt buộc, bao toàn bộ |
| `agent` | Đầu pipeline / trong stage | Nơi chạy: `any`, `none`, `label 'docker'`, `docker { image 'node:20' }` |
| `stages` | Trong pipeline | Chứa danh sách `stage` |
| `stage` | Trong stages | Một giai đoạn, có `steps` |
| `steps` | Trong stage | Các lệnh: `sh`, `echo`, `bat`, `script`… |
| `post` | Trong pipeline hoặc stage | Chạy sau cùng: `always`, `success`, `failure`, `unstable`, `changed`, `aborted` |
| `environment` | Pipeline / stage | Biến môi trường `KEY = 'value'` |
| `parameters` | Pipeline | Tham số nhập khi bấm Build: `string`, `booleanParam`, `choice`, `password` |
| `triggers` | Pipeline | `cron('...')`, `pollSCM('...')`, `upstream(...)` |
| `options` | Pipeline / stage | `timeout`, `timestamps`, `buildDiscarder`, `retry(n)`, `skipDefaultCheckout()` |
| `tools` | Pipeline / stage | Tool đã cài trong Global Tool Config: `tools { maven 'M3'; jdk 'JDK17' }` |
| `when` | Trong stage | Điều kiện chạy stage: `branch`, `expression`, `environment`, `not`, `allOf`, `anyOf` |
| `input` | Trong stage | Dừng chờ người duyệt (manual approval) |
| `parallel` | Trong stage | Chạy các stage con song song |

**Ví dụ các khối hay dùng:**

```groovy
agent {
    docker { image 'maven:3.9-eclipse-temurin-17' }  // chạy trong container
}
// hoặc
agent { label 'linux && docker' }  // chọn agent theo label

options {
    timeout(time: 1, unit: 'HOURS')
    retry(2)                       // thử lại stage tối đa 2 lần khi lỗi
    skipDefaultCheckout(true)      // không checkout tự động
}

tools {
    nodejs 'NodeJS-20'             // tên trong Global Tool Configuration
}

stage('Duyệt deploy') {
    input {
        message 'Deploy lên production?'
        ok 'Đồng ý'
        submitter 'team-lead'      // chỉ user này được duyệt
    }
    steps { sh './deploy-prod.sh' }
}

stage('Build đa nền tảng') {
    parallel {
        stage('Linux')   { agent { label 'linux' };   steps { sh 'make' } }
        stage('Windows') { agent { label 'windows' }; steps { bat 'make.bat' } }
    }
}
```

### 5.4. Nên dùng loại nào?

| | Declarative | Scripted |
|---|---|---|
| Độ dễ đọc | Cao | Trung bình |
| Kiểm tra cú pháp | Có (khi lưu / qua linter) | Không |
| Logic phức tạp | Hạn chế (dùng `script { }` để chèn Groovy) | Tự do |
| Khuyến nghị | **Mặc định cho mọi dự án** | Chỉ khi Declarative không đủ |

> Mẹo: trong Declarative, cần logic Groovy phức tạp thì bọc trong `script { ... }`:
> ```groovy
> steps {
>     script {
>         def files = findFiles(glob: '**/*.jar')
>         echo "Tìm thấy ${files.size()} file jar"
>     }
> }
> ```

---

## 6. Jenkinsfile ví dụ hoàn chỉnh

### 6.1. CI cho Java / Maven

```groovy
// Jenkinsfile — dự án Java Maven
pipeline {
    agent any

    tools {
        jdk 'JDK17'        // tên cấu hình trong Manage Jenkins → Tools
        maven 'Maven-3.9'
    }

    environment {
        MAVEN_OPTS = '-Dmaven.repo.local=.m2'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '20'))
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn -B test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Phân tích SonarQube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'mvn -B sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Lưu Artifact') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }

    post {
        success {
            echo '✅ Build thành công!'
        }
        failure {
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Xem chi tiết: ${env.BUILD_URL}",
                to: 'team@example.com'
            )
        }
    }
}
```

### 6.2. CI cho Node.js

```groovy
// Jenkinsfile — dự án Node.js
pipeline {
    agent {
        docker { image 'node:20-alpine' }   // mỗi build chạy trong container sạch
    }

    environment {
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"  // cache npm trong workspace
        CI = 'true'
    }

    options {
        timeout(time: 20, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '15'))
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Cài đặt') {
            steps {
                sh 'node --version && npm --version'
                sh 'npm ci'   // nhanh + ổn định hơn npm install cho CI
            }
        }

        stage('Lint') {
            steps { sh 'npm run lint' }
        }

        stage('Test') {
            steps { sh 'npm test -- --ci --reporters=jest-junit' }
            post {
                always {
                    junit 'junit.xml'
                }
            }
        }

        stage('Build') {
            steps { sh 'npm run build' }
        }

        stage('Lưu artifact') {
            steps {
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }
    }

    post {
        always {
            echo "Build #${env.BUILD_NUMBER} kết thúc: ${currentBuild.currentResult}"
        }
    }
}
```

### 6.3. Build & Push Docker Image

```groovy
// Jenkinsfile — build Docker image và push lên registry
pipeline {
    agent any

    environment {
        REGISTRY   = 'registry.example.com'
        IMAGE_NAME = 'my-app'
        // Credential loại "Username with password", id = docker-registry
        REG_CREDS  = credentials('docker-registry')
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build image') {
            steps {
                script {
                    // Tag theo build number + short commit
                    def commit = sh(script: 'git rev-parse --short HEAD',
                                    returnStdout: true).trim()
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}-${commit}"
                    sh "docker build -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker tag ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} " +
                       "${REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Quét bảo mật (Trivy)') {
            steps {
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL " +
                   "${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} || true"
            }
        }

        stage('Push registry') {
            when { branch 'main' }
            steps {
                sh '''
                    echo "$REG_CREDS_PSW" | docker login $REGISTRY \
                        -u "$REG_CREDS_USR" --password-stdin
                    docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    docker push $REGISTRY/$IMAGE_NAME:latest
                    docker logout $REGISTRY
                '''
            }
        }

        stage('Dọn dẹp') {
            steps {
                sh "docker rmi ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} || true"
                sh 'docker image prune -f'
            }
        }
    }

    post {
        success { echo "✅ Image ${env.IMAGE_TAG} đã push thành công" }
        failure { echo "❌ Build Docker thất bại" }
    }
}
```

> **Lưu ý bảo mật:** không bao giờ `echo` password ra log. Dùng `credentials()` binding — Jenkins tự che (mask) giá trị trong console output.

---

## 7. Quản lý Plugins

Plugin là linh hồn của Jenkins: mọi tích hợp (Git, Docker, Pipeline, Blue Ocean…) đều là plugin.

### 7.1. Cài / Gỡ / Cập nhật plugin

**Trên Web UI:** **Manage Jenkins → Plugins**

| Tab | Dùng để |
|---|---|
| **Available plugins** | Tìm và cài plugin mới (tick → **Install**) |
| **Installed plugins** | Xem đã cài, tick chọn để **Uninstall** / enable-disable |
| **Updates** | Plugin có bản mới → tick → **Download now and install after restart** |
| **Advanced** | Upload file `.hpi`/`.jpi` thủ công; cấu hình update site / proxy |

Sau khi cài plugin mới, Jenkins thường yêu cầu **restart** (tick "Restart Jenkins when installation is complete and no jobs are running").

**Bằng CLI (jenkins-cli):**

```bash
# Tải CLI
wget http://localhost:8080/jnlpJars/jenkins-cli.jar

# Liệt kê plugin đã cài
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  -auth admin:matkhau list-plugins

# Cài plugin
java -jar jenkins-cli.jar -s http://localhost:8080/ \
  -auth admin:matkhau install-plugin blueocean git -restart
```

**Cài plugin vào Docker image (khuyên dùng cho IaC):**

```dockerfile
FROM jenkins/jenkins:lts-jdk17
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt
```

```
# plugins.txt
git
workflow-aggregator
blueocean
docker-workflow
credentials-binding
junit
emailext
sonar
```

```bash
docker build -t my-jenkins:lts .
```

### 7.2. Plugins nên có

| Plugin | Chức năng |
|---|---|
| **Pipeline** (`workflow-aggregator`) | Bộ plugin pipeline đầy đủ (stage, node…) |
| **Blue Ocean** | UI hiện đại cho pipeline |
| **Git** / **GitHub** / **GitLab** | Tích hợp SCM, webhook, build PR |
| **Credentials Binding** | Dùng credentials trong pipeline (`credentials()`) |
| **Docker Pipeline** (`docker-workflow`) | `agent { docker { ... } }`, build/push image |
| **JUnit** | Hiển thị kết quả test đẹp |
| **Email Extension** (`emailext`) | Gửi mail thông báo linh hoạt |
| **SonarQube Scanner** | Phân tích chất lượng code |
| **SSH Agent** | Dùng SSH key trong build |
| **Timestamper** | Thêm timestamp vào log |
| **Workspace Cleanup** | Dọn workspace sau build |
| **ThinBackup** | Backup Jenkins định kỳ |
| **Matrix Authorization Strategy** | Phân quyền chi tiết theo user |
| **Configuration as Code (JCasC)** | Cấu hình Jenkins bằng YAML |
| **Prometheus metrics** | Export metrics để giám sát |

### 7.3. Xử lý xung đột plugin

- **Không tương thích version:** plugin mới đòi Jenkins core mới hơn → nâng cấp Jenkins trước (backup JENKINS_HOME trước!).
- **Dependency conflict:** cập nhật **tất cả** plugin lên bản mới nhất cùng lúc thay vì lẻ từng cái.
- **Plugin lỗi sau update:** **Downgrade** về bản trước, hoặc gỡ nếu không bắt buộc.
- Nguyên tắc: **không cài plugin không dùng đến** — mỗi plugin là một bề mặt lỗi + rủi ro bảo mật.

---

## 8. Agents / Nodes

### 8.1. Khái niệm

- Controller mặc định cũng là một node (built-in node), có thể chạy build — nhưng **không nên** cho production (xem mục 17).
- Thêm agent để phân tải, cách ly môi trường, build đa nền tảng.
- **Label**: nhãn gán cho agent (ví dụ `linux`, `docker`, `gpu`). Job dùng `agent { label 'docker' }` để chọn đúng agent.

### 8.2. Thêm agent qua SSH (phổ biến nhất)

Chuẩn bị máy agent: cài Java 17 (cùng major version với controller), mở port 22.

Trên controller:

1. **Manage Jenkins → Nodes → New Node** → tên `agent-01` → chọn **Permanent Agent**
2. Điền:
   - **Remote root directory**: `/home/jenkins/agent` (thư mục làm việc)
   - **Labels**: `linux docker`
   - **Launch method**: **Launch agents via SSH**
     - **Host**: IP máy agent
     - **Credentials**: chọn SSH key (tạo trong mục 9)
     - **Host Key Verification Strategy**: **Manually trusted key** (lần đầu) hoặc **Known hosts file**
3. **Save** → Jenkins tự SSH sang, copy `agent.jar`, khởi chạy agent.

Kiểm tra log agent: **Nodes → agent-01 → Log** — phải thấy `Agent successfully connected`.

### 8.3. Thêm agent qua JNLP (inbound)

Phù hợp khi agent nằm sau NAT/firewall, không SSH được từ controller (agent chủ động kết nối về).

1. **New Node** → tên `agent-jnlp` → **Launch method: Launch agent by connecting it to the controller**
2. Save → mở trang node, lấy **lệnh chạy** (ví dụ):
   ```bash
   curl -sO http://jenkins:8080/jnlpJars/agent.jar
   java -jar agent.jar \
     -url http://jenkins:8080/ \
     -secret <SECRET-KEY> \
     -name agent-jnlp \
     -webSocket -workDir "/home/jenkins/agent"
   ```
3. Chạy lệnh trên máy agent → agent tự kết nối về controller.

> Từ Jenkins 2.219+, nên bật **WebSocket** cho JNLP để không cần mở port 50000 (Manage Jenkins → Security → Agents → TCP port: Disable, dùng WebSocket).

### 8.4. Agent dạng Docker (động)

Dùng plugin **Docker** để mỗi build tự tạo container agent rồi xóa — không cần máy agent cố định:

1. Cài plugin **Docker**
2. **Manage Jenkins → Nodes → Configure Clouds → Add a new cloud → Docker**
3. Docker Host URI: `unix:///var/run/docker.sock` (hoặc `tcp://...`)
4. **Docker Agent templates** → Add:
   - Image: `jenkins/inbound-agent:latest`
   - Labels: `docker-agent`
5. Trong pipeline: `agent { label 'docker-agent' }` — Jenkins tự `docker run` agent khi có build.

### 8.5. Cấu hình executor và label

- **# of executors**: số build chạy song song trên 1 agent (thường = số CPU).
- **Usage**: chọn **Only build jobs with label expressions matching this node** — agent chỉ chạy job đúng label.
- **Availability**: online liên tục, hoặc theo lịch để tiết kiệm chi phí cloud.

---

## 9. Credentials

Credentials lưu bí mật (mật khẩu, token, SSH key) tập trung, mã hóa trong Jenkins — pipeline chỉ tham chiếu bằng **ID**, không hardcode.

### 9.1. Tạo credential

**Manage Jenkins → Credentials → (domain) → Add Credentials**:

| Loại (Kind) | Dùng cho |
|---|---|
| **Username with password** | Đăng nhập Docker registry, server SSH, API basic auth |
| **Secret text** | API token, GitHub PAT, webhook secret |
| **SSH Username with private key** | SSH vào agent / server deploy (paste key hoặc từ file) |
| **Secret file** | File bí mật (kubeconfig, .p12, settings.xml…) |
| **Certificate** | Chứng chỉ client PKCS#12 |

Đặt **ID** có ý nghĩa, ví dụ: `docker-registry`, `github-token`, `deploy-ssh-key`, `kubeconfig-prod`. Tick **Treat username as secret** nếu cần.

Phạm vi (Scope):
- **Global**: dùng cho mọi job (phổ biến)
- **System**: chỉ Jenkins core/plugin dùng (ví dụ credential kết nối agent)

### 9.2. Dùng credential trong Pipeline

```groovy
pipeline {
    agent any
    environment {
        // Username with password → tạo 2 biến: _USR và _PSW
        DOCKER = credentials('docker-registry')
        // Secret text → 1 biến
        GH_TOKEN = credentials('github-token')
        // Secret file → biến chứa đường dẫn file tạm
        KUBECONFIG_FILE = credentials('kubeconfig-prod')
    }
    stages {
        stage('Deploy') {
            steps {
                // Cách 2: withCredentials (khuyên dùng, phạm vi hẹp)
                withCredentials([
                    usernamePassword(credentialsId: 'deploy-server',
                                     usernameVariable: 'USER',
                                     passwordVariable: 'PASS'),
                    string(credentialsId: 'api-token',
                           variable: 'TOKEN'),
                    sshUserPrivateKey(credentialsId: 'deploy-ssh-key',
                                      keyFileVariable: 'KEY')
                ]) {
                    sh '''
                        ssh -i "$KEY" "$USER"@server "deploy.sh"
                        curl -H "Authorization: Bearer $TOKEN" https://api...
                    '''
                }
            }
        }
    }
}
```

**Quy tắc an toàn:**
- Jenkins tự **mask** (che thành `****`) giá trị credential trong console log — nhưng chỉ khi giá trị xuất hiện nguyên vẹn. Đừng `echo` biến ra rồi xử lý chuỗi làm lộ một phần.
- Không bao giờ hardcode secret trong Jenkinsfile (vì Jenkinsfile nằm trong repo Git!).
- Xoay (rotate) token định kỳ; xóa credential không dùng.
- Hạn chế ai được xem cấu hình credential (xem mục 15).

---

## 10. Blue Ocean UI

**Blue Ocean** là giao diện hiện đại thay thế UI cổ điển cho pipeline: trực quan, đẹp, dễ theo dõi.

### 10.1. Cài đặt

1. **Manage Jenkins → Plugins → Available plugins** → tìm **Blue Ocean** → Install (nó kéo theo ~10 plugin phụ thuộc)
2. Restart Jenkins nếu được yêu cầu
3. Mở Blue Ocean: bấm **Open Blue Ocean** ở menu trái dashboard, hoặc truy cập `http://<host>:8080/blue`

### 10.2. Sử dụng

- **Dashboard Blue Ocean**: liệt kê pipeline, trạng thái build gần nhất, thời gian chạy — nhìn một phát biết branch nào đang fail.
- **Pipeline view**: từng stage hiện dạng cột ngang xanh/đỏ; click stage xem log **chỉ của stage đó**.
- **Visual pipeline editor**: chọn repo Git → thêm/sửa stage bằng kéo-thả → commit thẳng Jenkinsfile về repo. Tiện cho người mới, nhưng lâu dài nên viết Jenkinsfile bằng tay.

### 10.3. Lưu ý

- Blue Ocean **chỉ hiển thị Pipeline job** — Freestyle job không hiện.
- Một số tính năng nâng cao (quản lý users, plugins, system config) vẫn phải về UI cổ điển.
- Blue Ocean đã qua thời phát triển mạnh (Jenkins tập trung vào UI mới), nhưng vẫn là công cụ xem pipeline tốt nhất hiện tại.

---

## 11. Shared Libraries

Khi có 10–20 pipeline, bạn sẽ thấy code lặp lại: bước checkout, gửi Slack, build Docker… **Shared Library** cho phép viết một lần, dùng chung mọi pipeline.

### 11.1. Tạo Shared Library

Tạo một repo Git riêng, ví dụ `jenkins-shared-lib`, cấu trúc chuẩn:

```
jenkins-shared-lib/
├── vars/                  # Hàm gọi trực tiếp trong pipeline
│   ├── notifySlack.groovy
│   ├── buildDocker.groovy
│   └── deployK8s.groovy
├── src/com/example/       # Class Groovy (package chuẩn)
│   └── Utils.groovy
└── resources/             # File tĩnh (script shell, template...)
    └── scripts/
        └── healthcheck.sh
```

Ví dụ `vars/notifySlack.groovy`:

```groovy
// Gửi thông báo Slack — gọi: notifySlack('Build xong', 'good')
def call(String message, String color = 'good') {
    slackSend(
        channel: '#ci-cd',
        color: color,
        message: "${message}\nJob: ${env.JOB_NAME} #${env.BUILD_NUMBER}\n${env.BUILD_URL}"
    )
}
```

Ví dụ `vars/buildDocker.groovy`:

```groovy
def call(String imageName, String tag = env.BUILD_NUMBER) {
    sh "docker build -t ${imageName}:${tag} ."
    return "${imageName}:${tag}"
}
```

### 11.2. Khai báo library trong Jenkins

**Manage Jenkins → System → Global Pipeline Libraries** → Add:

- **Name**: `my-lib` (tên dùng trong pipeline)
- **Default version**: `main` (nhánh/tag/commit)
- **Retrieval method**: **Modern SCM → Git** → điền URL repo `https://github.com/org/jenkins-shared-lib.git` (+ credential nếu private)
- Tick **Load implicitly** nếu muốn mọi pipeline tự có (không cần `@Library`)

### 11.3. Dùng trong pipeline

```groovy
@Library('my-lib') _   // dòng này phải ở trên cùng, ngoài block pipeline

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def img = buildDocker('registry.example.com/my-app')
                    echo "Đã build: ${img}"
                }
            }
        }
    }
    post {
        success { notifySlack('✅ Build thành công') }
        failure { notifySlack('❌ Build thất bại', 'danger') }
    }
}
```

**Best practices cho Shared Library:**
- Version bằng tag (`v1.2.0`) thay vì `main` cho pipeline production.
- Viết unit test cho `vars/` bằng framework **JenkinsPipelineUnit**.

---

## 12. Multibranch Pipeline

**Multibranch Pipeline** tự động tạo một pipeline con cho **mỗi branch / PR** có chứa Jenkinsfile — chuẩn CI hiện đại: push branch mới → tự build → báo pass/fail ngay trên PR.

### 12.1. Tạo Multibranch Pipeline

1. **New Item** → tên `my-app-ci` → chọn **Multibranch Pipeline**
2. **Branch Sources** → **Add source** → **GitHub** (hoặc Git / GitLab):
   - **Credentials**: GitHub PAT (tạo ở mục 9)
   - **Repository HTTPS URL**: `https://github.com/org/my-app.git`
   - **Behaviours**: mặc định đã có **Discover branches** + **Discover pull requests** — giữ nguyên
3. **Build Configuration**: **by Jenkinsfile**, **Script Path**: `Jenkinsfile`
4. **Scan Multibranch Pipeline Triggers**: tick **Periodically if not otherwise run**, Interval `5 minutes` (quét branch mới; nếu đã có webhook thì không cần dày)
5. **Save** → Jenkins quét repo, liệt kê từng branch/PR thành job con.

### 12.2. Jenkinsfile thân thiện multibranch

```groovy
pipeline {
    agent any
    stages {
        stage('Build & Test') {
            steps {
                sh 'npm ci && npm test'
            }
        }
        // Chỉ deploy khi là nhánh main (không phải PR)
        stage('Deploy staging') {
            when {
                allOf {
                    branch 'main'
                    not { changeRequest() }  // changeRequest() = true nếu là PR
                }
            }
            steps { sh './deploy-staging.sh' }
        }
    }
}
```

Các biến môi trường hữu ích trong multibranch:

| Biến | Ý nghĩa |
|---|---|
| `env.BRANCH_NAME` | Tên branch đang build (`main`, `feature/x`) |
| `env.CHANGE_ID` | Số PR (ví dụ `42`), null nếu không phải PR |
| `env.CHANGE_TARGET` | Branch đích của PR |
| `env.CHANGE_BRANCH` | Branch nguồn của PR |

### 12.3. Dọn dẹp branch cũ

Trong cấu hình Multibranch → **Orphaned Item Strategy**: **Discard old items**, đặt **Days to keep** (ví dụ 7) và **Max # of items** — tự xóa job của branch đã merge/xóa, tránh rác.

---

## 13. Webhooks từ GitHub / GitLab

Thay vì Jenkins poll SCM mỗi 5 phút (tốn tài nguyên, chậm), **webhook** giúp: push code → GitHub gọi ngay vào Jenkins → build tức thì.

### 13.1. Chuẩn bị trên Jenkins

1. Cài plugin **GitHub** (cho github.com) hoặc **GitLab**
2. Job Pipeline/Multibranch: tab **Build Triggers** → tick **GitHub hook trigger for GITScm polling** (hoặc với GitLab plugin: **Build when a change is pushed to GitLab**)
3. Lấy URL webhook của Jenkins:
   - GitHub plugin: `http://<jenkins-host>/github-webhook/`
   - GitLab plugin: `http://<jenkins-host>/project/<job-name>`
   - Generic: `http://<jenkins-host>/generic-webhook-trigger/invoke?token=TOKEN`
4. Nếu Jenkins sau NAT: cần public URL (domain, hoặc ngrok/Cloudflare Tunnel lúc test).

### 13.2. Cấu hình trên GitHub

Repo → **Settings → Webhooks → Add webhook**:
- **Payload URL**: `http://<jenkins-host>/github-webhook/`
- **Content type**: `application/json`
- **Which events**: **Just the push event** (thêm **Pull requests** nếu muốn build PR)
- **Active**: tick → **Add webhook**

Push thử một commit → webhook hiện dấu tick xanh (delivery 200) → Jenkins tự trigger build.

### 13.3. Cấu hình trên GitLab

Repo → **Settings → Webhooks**: **URL** `http://<jenkins-host>/project/<job-name>`, **Secret token** trùng với token trong cấu hình job (Build Triggers → Advanced), tick **Push events** + **Merge request events** → **Add webhook** → **Test → Push events**.

### 13.4. Xử lý sự cố webhook

| Triệu chứng | Nguyên nhân / Cách sửa |
|---|---|
| GitHub báo delivery failed (timeout) | Jenkins không public — kiểm tra firewall, NAT, reverse proxy |
| 403 "No valid crumb" | CSRF protection chặn — xem mục 15 để cấp crumb hoặc dùng token |
| Webhook 200 nhưng không build | Sai URL job (GitLab), hoặc job chưa tick trigger; kiểm tra **GitHub Hook Log** trong job |
| Build trigger 2 lần | Vừa bật webhook vừa để `pollSCM` — tắt pollSCM đi |

---

## 14. Backup & Restore

Mất `JENKINS_HOME` = mất toàn bộ jobs, lịch sử, users, plugins, credentials. **Backup là bắt buộc.**

### 14.1. Cách 1 — Copy JENKINS_HOME (đơn giản, hiệu quả)

Dừng Jenkins (để file không bị ghi dở), rồi copy toàn bộ:

```bash
# Dừng Jenkins
sudo systemctl stop jenkins   # hoặc docker stop jenkins

# Backup (giữ permission, dùng rsync cho lần sau nhanh hơn)
sudo rsync -a --delete /var/lib/jenkins/ /backup/jenkins-2026-10-10/

# Nén lại cho gọn
sudo tar -czf /backup/jenkins-2026-10-10.tar.gz -C /backup jenkins-2026-10-10

# Khởi động lại
sudo systemctl start jenkins
```

**Restore:** giải nén đè vào `JENKINS_HOME` đúng vị trí, chown lại user `jenkins`, restart.

```bash
sudo systemctl stop jenkins
sudo tar -xzf /backup/jenkins-2026-10-10.tar.gz -C /tmp
sudo rsync -a --delete /tmp/jenkins-2026-10-10/ /var/lib/jenkins/
sudo chown -R jenkins:jenkins /var/lib/jenkins
sudo systemctl start jenkins
```

> Với Docker: backup volume `jenkins_home`:
> ```bash
> docker run --rm -v jenkins_home:/data -v /backup:/backup \
>   alpine tar -czf /backup/jenkins_home.tar.gz -C /data .
> ```

**Nên loại trừ khi backup để nhẹ:** `workspace/` (build xong tạo lại được), `caches/`, `*.log` lớn. Giữ lại: `jobs/`, `users/`, `plugins/`, `secrets/`, `config.xml`, `credentials.xml`, `*.key`.

### 14.2. Cách 2 — Plugin ThinBackup

1. Cài plugin **ThinBackup**
2. **Manage Jenkins → ThinBackup → Settings**:
   - **Backup directory**: `/var/backups/jenkins`
   - **Backup schedule**: `H 2 * * *` (2h sáng hàng ngày)
   - **Max number of backup sets**: `7`
   - Tick **Backup plugins**, **Backup user content**
3. Bấm **Backup Now** để chạy tay lần đầu
4. **Restore**: ThinBackup → **Restore** → chọn bản backup → restart Jenkins

Ưu điểm: chạy ngay trên UI, có lịch tự động, không cần dừng Jenkins (backup "nóng" — đủ tốt cho đa số).

### 14.3. Cách 3 — Configuration as Code (JCasC)

Mô tả toàn bộ cấu hình Jenkins bằng YAML, version-control trong Git:

```yaml
# jenkins.yaml
jenkins:
  systemMessage: "Jenkins managed by JCasC"
  numExecutors: 2
credentials:
  system:
    domainCredentials:
      - credentials:
          - usernamePassword:
              id: "docker-registry"
              username: "myuser"
              password: "${DOCKER_PASSWORD}"
```

Chạy Jenkins với `CASC_JENKINS_CONFIG=/path/to/jenkins.yaml`. Kết hợp **JCasC (cấu hình) + backup jobs/builds (dữ liệu)** = restore nhanh nhất.

### 14.4. Lịch backup khuyến nghị

| Dữ liệu | Tần suất |
|---|---|
| Toàn bộ JENKINS_HOME | Hàng ngày (giờ thấp điểm) |
| Jenkinsfile, JCasC YAML, shared lib | Mỗi lần đổi (đã có Git) |
| Kiểm tra restore | Mỗi quý — restore lên máy test, verify job chạy được |

---

## 15. Bảo mật cơ bản

Jenkins mặc định sau wizard đã bật login, nhưng production cần siết thêm.

### 15.1. Matrix-based Security (phân quyền chi tiết)

1. **Manage Jenkins → Security**:
   - **Security Realm**: **Jenkins' own user database** (+ tick **Allow users to sign up**? — **tắt** trên production)
   - **Authorization**: **Matrix-based security**
2. Bảng ma trận hiện ra — gán quyền theo user/group:

| Quyền | admin | team-lead | developer | viewer |
|---|---|---|---|---|
| Overall/Administer | ✅ | | | |
| Overall/Read | ✅ | ✅ | ✅ | ✅ |
| Job/Build | ✅ | ✅ | ✅ | |
| Job/Configure | ✅ | ✅ | | |
| Job/Create, Delete | ✅ | | | |
| Job/Workspace | ✅ | ✅ | ✅ | |
| Credential/View | ✅ | | | |
| Agent/Connect | ✅ | | | |

3. **Save**. Từ giờ user `developer` chỉ build/xem, không sửa cấu hình job.

> Có thể phân quyền **theo từng job/folder** bằng plugin **Role-based Authorization Strategy** hoặc **Folder-based** — gọn hơn matrix global khi nhiều team.

### 15.2. CSRF Protection (Crumb)

Jenkins bật sẵn **CSRF protection**: mọi POST từ ngoài phải kèm **crumb** (token chống giả mạo request).

- Webhook GitHub/GitLab plugin chính thức đã tự xử lý crumb — thường không cần làm gì.
- Nếu tự gọi API bằng script, lấy crumb trước:

```bash
CRUMB=$(curl -s -u admin:matkhau \
  'http://localhost:8080/crumbIssuer/api/json' | jq -r '.crumb')
curl -u admin:matkhau -H "Jenkins-Crumb:$CRUMB" \
  -X POST http://localhost:8080/job/my-job/build
```

- **Không tắt** CSRF ("Default Crumb Issuer" luôn bật) trừ khi bạn hiểu rõ hậu quả.

### 15.3. Checklist bảo mật nhanh

- [ ] Đổi/tắt user `admin` mặc định, mỗi người một tài khoản
- [ ] Bật HTTPS qua reverse proxy (mục 2.6), không expose :8080 trực tiếp
- [ ] Tắt **Allow users to sign up** trên production
- [ ] Agent chạy dưới user ít quyền, không chạy build trên controller (`# of executors = 0` cho built-in node)
- [ ] Secret trong credentials, không hardcode trong Jenkinsfile/log
- [ ] Cập nhật Jenkins core + plugins định kỳ (lỗ hổng RCE từng xảy ra)
- [ ] Bật **Audit Trail** plugin để log ai làm gì
- [ ] Backup JENKINS_HOME định kỳ (mục 14)
- [ ] Giới hạn ai được cấu hình Groovy/Script Console (chạy được lệnh hệ thống!)

> **Cảnh báo:** **Manage Jenkins → Script Console** cho phép chạy Groovy tùy ý = tương đương quyền root trên controller. Chỉ admin tin cậy được vào.

---

## 16. Troubleshooting

### Lỗi 1 — Agent disconnect / không kết nối được

**Triệu chứng:** node báo offline, log `Connection refused` / `UnknownHostException`.

**Xử lý:**
- SSH agent: kiểm tra Java đã cài trên agent chưa (`java -version` — phải cùng major version với controller), user SSH có quyền ghi `Remote root directory` không.
- JNLP agent: kiểm tra secret đúng không, port 50000 mở chưa (hoặc chuyển sang WebSocket).
- Kiểm tra đồng hồ 2 máy lệch nhau quá nhiều (TLS/SSH nhạy với time skew) — đồng bộ NTP.
- Xem log chi tiết: **Nodes → <agent> → Log**.

### Lỗi 2 — Xung đột plugin sau khi update

**Triệu chứng:** Jenkins không khởi động, trang báo lỗi `NoSuchMethodError` / `Plugin X failed to load`.

**Xử lý:**
1. Dừng Jenkins, vào `$JENKINS_HOME/plugins/`, xóa file `.jpi` của plugin lỗi + thư mục giải nén của nó.
2. Khởi động lại — Jenkins sẽ chạy thiếu plugin đó (job dùng nó sẽ báo lỗi cấu hình, nhưng hệ thống lên được).
3. Cài lại plugin từ **Available plugins** (bản tương thích), hoặc **Downgrade** nếu vừa update.
4. Bài học: **backup JENKINS_HOME trước mỗi lần update hàng loạt**.

### Lỗi 3 — Disk full (JENKINS_HOME hết dung lượng)

**Triệu chứng:** build fail lạ, không lưu được artifact, UI báo `No space left on device`.

**Xử lý:**
```bash
# Tìm thư mục ngốn dung lượng
du -sh /var/lib/jenkins/* | sort -rh | head
```
- Bật **Discard old builds** cho mọi job (`buildDiscarder`: giữ 10–30 build).
- Xóa workspace cũ: **Manage Jenkins → Nodes → <node> → Workspace → Wipe out**, hoặc dùng plugin Workspace Cleanup.
- Xóa artifact cũ nặng: job config → **Discard old builds → Max # of builds to keep with artifacts**.
- Dọn Docker: `docker system prune -af --volumes` (cẩn thận trên máy dùng chung).
- Về lâu dài: tách `JENKINS_HOME` ra ổ riêng, gắn cảnh báo disk > 80%.

### Lỗi 4 — Build bị treo ở stage (hanging)

**Triệu chứng:** build chạy mãi không xong, log đứng yên.

**Xử lý:**
- Luôn đặt `timeout()` ở pipeline options và từng stage (mục 5) — build treo tự abort.
- Nguyên nhân phổ biến: lệnh chờ input (`read`, `ssh` hỏi password), test mở server không tắt, `docker build` chờ network.
- Hủy build treo: mở build → **Abort** (bấm 2 lần nếu lần 1 không ăn → **force abort**).
- Nếu abort không được, restart agent hoặc kill process trên agent.

### Lỗi 5 — `permission denied` khi chạy sh trong pipeline

**Triệu chứng:** `sh './deploy.sh'` báo permission denied.

**Xử lý:** file checkout từ Git mất quyền execute. Thêm bước `sh 'chmod +x deploy.sh'` trước khi chạy, hoặc commit đúng quyền:
```bash
git update-index --chmod=+x deploy.sh
git commit -m "chmod +x deploy.sh"
```

### Lỗi 6 — Git checkout fail: `Permission denied (publickey)`

**Triệu chứng:** stage Checkout đỏ, không clone được repo private.

**Xử lý:** tạo credential **SSH Username with private key**, add public key vào GitHub/GitLab (Deploy keys), rồi chỉ định `credentialsId` trong `checkout`. Hoặc dùng HTTPS + PAT (dạng Username with password).

### Lỗi 7 — `docker: command not found` trong pipeline

**Triệu chứng:** agent báo không tìm thấy docker.

**Xử lý:**
- Cài Docker trên **agent** (không phải controller) — pipeline chạy ở agent.
- User chạy agent phải thuộc group `docker`: `sudo usermod -aG docker jenkins`.
- Nếu dùng `agent { docker { image ... } }`: agent cần plugin Docker Pipeline + Docker đã cài (để spawn container con).

### Lỗi 8 — Webhook không trigger build

Xem bảng xử lý ở [mục 13.4](#134-xử-lý-sự-cố-webhook). Thêm: kiểm tra **Jenkins URL** trong System config phải là URL public mà GitHub gọi được (không phải `localhost`).

### Lỗi 9 — OutOfMemoryError / Jenkins chậm, treo UI

**Triệu chứng:** log có `java.lang.OutOfMemoryError: Java heap space`, UI load chậm.

**Xử lý:**
- Tăng heap: `JAVA_OPTS="-Xms1g -Xmx4g"` (trong `/etc/default/jenkins`, systemd unit, hoặc docker-compose).
- Giảm số executor trên controller, chuyển build sang agent.
- Kiểm tra plugin ngốn RAM: **Manage Jenkins → System Information** / dùng plugin **Monitoring**.
- Restart định kỳ nếu rò rỉ bộ nhớ do plugin (đặt lịch restart đêm + backup trước).

### Lỗi 10 — Port 8080 đã bị chiếm

**Triệu chứng:** Jenkins không start, log `java.net.BindException: Address already in use`.

**Xử lý:**
```bash
sudo ss -tlnp | grep 8080   # xem process nào chiếm
# Đổi port Jenkins:
# - Docker: -p 9090:8080
# - Package: sửa HTTP_PORT=9090 trong /etc/default/jenkins
# - War: java -jar jenkins.war --httpPort=9090
```

### Lỗi 11 — Quên mật khẩu admin

**Xử lý:** (cần SSH vào server) dừng Jenkins → sửa `$JENKINS_HOME/config.xml`: đặt `<useSecurity>false</useSecurity>` → start lại → vào **Manage Jenkins → Users** reset mật khẩu → bật lại security.

### Lỗi 12 — Build fail chỉ trên Jenkins, local chạy được

**Xử lý:** 99% do khác môi trường — pin version tool bằng `tools { }`/Docker agent, so sánh biến `env`, chuẩn hóa `.gitattributes`, wipe workspace build lại từ đầu để xác nhận.

---

## 17. Best Practices Checklist

### Pipeline & Code
- [ ] Mọi dự án dùng **Pipeline + Jenkinsfile** trong repo (không Freestyle cho dự án thật)
- [ ] Ưu tiên **Declarative Pipeline**; `script { }` chỉ khi cần logic phức tạp
- [ ] Jenkinsfile được **code review** như code sản phẩm
- [ ] Mọi stage dài có `timeout()`; pipeline có `buildDiscarder`
- [ ] Không hardcode secret — dùng **credentials binding**
- [ ] Logic dùng chung ≥2 pipeline → tách ra **Shared Library** có version tag

### Agents & Môi trường
- [ ] Build chạy trên **agent**, controller để `# of executors = 0`
- [ ] Môi trường build **tái lập được**: Docker image pin version, hoặc `tools { }` cố định
- [ ] Agent có **label** rõ ràng; job chỉ định đúng label
- [ ] Wipe workspace định kỳ / dùng Workspace Cleanup

### Bảo mật
- [ ] HTTPS qua reverse proxy; không expose :8080 trực tiếp
- [ ] Mỗi người một tài khoản; tắt signup; phân quyền matrix tối thiểu
- [ ] CSRF luôn bật; Script Console chỉ admin
- [ ] Update Jenkins core + plugins theo lịch (sau khi backup)

### Vận hành
- [ ] **Backup JENKINS_HOME hàng ngày** + test restore mỗi quý
- [ ] Giám sát: disk, RAM, số executor bận, build fail rate (plugin Prometheus + Grafana)
- [ ] Webhook thay cho pollSCM dày đặc
- [ ] Multibranch + Quality Gate (SonarQube) cho repo chính
- [ ] Thông báo build fail về đúng kênh (Slack/email) — không spam cả team vì 1 job phụ

### Hiệu năng
- [ ] Cache dependency (`.m2`, `node_modules`, Docker layer) giữa các build
- [ ] Chia test chạy **parallel** khi suite lớn
- [ ] Artifact chỉ lưu cái cần thiết, có chính sách xóa artifact cũ

---

## 18. FAQ + Cheat Sheet

### 18.1. FAQ

**Jenkins có miễn phí không?**
Hoàn toàn miễn phí, mã nguồn mở (MIT license). Chi phí duy nhất là server chạy nó.

**Jenkins vs GitHub Actions / GitLab CI, chọn cái nào?**
- GitHub Actions: dễ nhất nếu code đã trên GitHub.
- GitLab CI: tích hợp sẵn nếu dùng GitLab.
- Jenkins: linh hoạt nhất, self-hosted hoàn toàn, 1800+ plugins, phù hợp on-premise / yêu cầu đặc thù.

**Jenkinsfile để ở đâu?**
Thư mục gốc của repo, tên đúng `Jenkinsfile` (không có đuôi). Multibranch tự tìm file này.

**Một Jenkins chạy được bao nhiêu job song song?**
Phụ thuộc executors: mỗi agent N executors. Ví dụ 3 agent × 4 executors = 12 build song song.

**Nâng cấp Jenkins có mất dữ liệu không?**
Không, nếu **backup JENKINS_HOME trước** rồi mới nâng cấp. Plugin cũ có thể không tương thích core mới — update plugin cùng lúc.

### 18.2. Cheat Sheet — Lệnh nhanh

```bash
# --- Docker ---
docker run -d --name jenkins --restart unless-stopped \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts-jdk17
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword

# --- Systemd ---
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
sudo journalctl -u jenkins -f          # xem log

# --- Backup nhanh ---
sudo tar -czf /backup/jenkins-$(date +%F).tar.gz \
  -C /var/lib jenkins --exclude=jenkins/workspace

# --- Jenkins CLI ---
java -jar jenkins-cli.jar -s http://localhost:8080 \
  -auth admin:pass list-plugins
java -jar jenkins-cli.jar -s http://localhost:8080 \
  -auth admin:pass install-plugin blueocean -restart
java -jar jenkins-cli.jar -s http://localhost:8080 \
  -auth admin:pass safe-restart

# --- Trigger build bằng API ---
CRUMB=$(curl -s -u admin:pass \
  'http://localhost:8080/crumbIssuer/api/json' | jq -r .crumb)
curl -u admin:pass -H "Jenkins-Crumb:$CRUMB" \
  -X POST 'http://localhost:8080/job/my-job/build'

# --- Agent JNLP ---
curl -sO http://jenkins:8080/jnlpJars/agent.jar
java -jar agent.jar -url http://jenkins:8080/ \
  -secret <SECRET> -name agent-01 -webSocket -workDir /home/jenkins/agent
```

### 18.3. Cheat Sheet — Pipeline syntax

```groovy
pipeline {
    agent any                                    // any | none | label 'x' | docker { image 'x' }
    tools { maven 'M3'; jdk 'JDK17' }            // tool đã khai báo trong Global Tools
    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps(); retry(2); skipDefaultCheckout(true)
    }
    environment { KEY = 'value' }                // biến môi trường
    parameters {                                 // tham số khi bấm Build
        string(name: 'BRANCH', defaultValue: 'main', description: '')
        booleanParam(name: 'TEST', defaultValue: true, description: '')
        choice(name: 'ENV', choices: ['dev','prod'], description: '')
    }
    triggers { cron('H 2 * * *') }               // pollSCM('H/5 * * * *')
    stages {
        stage('Tên stage') {
            when { branch 'main' }               // expression | not | allOf | anyOf | changeRequest()
            // input { message 'Duyệt?'; ok 'OK'; submitter 'lead' }
            steps {
                echo 'log'; sh 'cmd'; bat 'cmd'
                checkout scm
                junit 'reports/*.xml'
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
                withCredentials([string(credentialsId: 'id', variable: 'V')]) {
                    sh 'echo $V'
                }
                script { /* groovy tự do */ }
            }
            post { always { echo 'x' } }
        }
        // stage('Song song') { parallel {
        //     stage('A') { steps { sh 'a' } }
        //     stage('B') { steps { sh 'b' } }
        // } }
    }
    post {
        always  { /* luôn chạy: dọn dẹp, junit */ }
        success { /* pass */ }
        failure { /* fail: gửi mail/slack */ }
        unstable { /* test fail */ }
        changed { /* đổi trạng thái vs build trước */ }
    }
}
```

### 18.4. Cron Jenkins (giống crontab, thêm `H`)

```
H/15 * * * *   → mỗi 15 phút (H = hash, rải tải)
H 2 * * *      → 2h sáng mỗi ngày
H H * * 0      → mỗi Chủ nhật (giờ ngẫu nhiên)
0 9 * * 1-5    → 9h sáng thứ 2–6
```

| Trường | Giá trị |
|---|---|
| phút | 0–59 |
| giờ | 0–23 |
| ngày trong tháng | 1–31 |
| tháng | 1–12 |
| thứ trong tuần | 0–7 (0 và 7 = Chủ nhật) |

---

*Hết tài liệu. Chúc bạn build xanh!* 🟢
