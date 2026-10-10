# Hướng dẫn Docker từ A đến Z 🐳

> Tài liệu tiếng Việt chi tiết cho người mới bắt đầu — đọc xong là chạy được Docker ngay.

Chào cậu! Nếu cậu từng nghe câu huyền thoại của dân lập trình:

> *"Lạ nhỉ, trên máy tớ chạy ngon mà!"*

...thì Docker chính là "vị cứu tinh" cho câu nói đó. Bài này đưa cậu đi từ con số 0: Docker là gì, cài thế nào, dùng ra sao, đến khi tự build được image và chạy cả stack web + database + Redis bằng một lệnh duy nhất.

**Chuẩn bị:** một máy Linux (Ubuntu/Debian là dễ nhất), hoặc WSL2/macOS; biết mở terminal và gõ lệnh cơ bản (`cd`, `ls`). Không cần biết gì về Docker trước đó.

**Cách đọc:** đọc theo thứ tự từ trên xuống. Mỗi lệnh đều có cú pháp + giải thích từng option + ví dụ thật kèm output mẫu. Cứ mở terminal gõ theo là được.

---

## Mục lục

1. [Docker là gì? Container là gì?](#1-docker-là-gì-container-là-gì)
2. [Tại sao nên dùng Docker?](#2-tại-sao-nên-dùng-docker)
3. [Cài đặt Docker](#3-cài-đặt-docker)
4. [Khái niệm cốt lõi: Image, Container, Dockerfile, Registry](#4-khái-niệm-cốt-lõi)
5. [Lệnh Docker cơ bản](#5-lệnh-docker-cơ-bản)
6. [Ví dụ 1: Chạy web server Nginx](#6-ví-dụ-1-chạy-web-server-nginx)
7. [Ví dụ 2: Chạy PostgreSQL](#7-ví-dụ-2-chạy-postgresql)
8. [Ví dụ 3: Chạy Redis](#8-ví-dụ-3-chạy-redis)
9. [Dockerfile: tự build image](#9-dockerfile-tự-build-image)
10. [Volumes: lưu trữ bền vững](#10-volumes-lưu-trữ-bền-vững)
11. [Networks: container giao tiếp với nhau](#11-networks)
12. [Biến môi trường](#12-biến-môi-trường)
13. [Docker Compose](#13-docker-compose)
14. [Best practices](#14-best-practices)
15. [Troubleshooting](#15-troubleshooting)
16. [Dọn dẹp ổ cứng](#16-dọn-dẹp)
17. [Cheat sheet](#17-cheat-sheet)

---

## 1. Docker là gì? Container là gì?

### 1.1. Định nghĩa ngắn gọn

- **Docker** là nền tảng giúp cậu **đóng gói** ứng dụng cùng toàn bộ những thứ nó cần (thư viện, runtime, config, code...) thành một "hộp" chuẩn, rồi **chạy** cái hộp đó ở bất cứ đâu: laptop, máy đồng nghiệp, server thật — chạy đâu cũng giống hệt nhau.
- **Container** là cái "hộp" đó khi đang chạy: một tiến trình bị **cô lập** (filesystem, network, process riêng) nhưng vẫn dùng chung **kernel** của máy host.

Ví von:

- **Image** = công thức nấu ăn + nguyên liệu đóng gói sẵn.
- **Container** = món ăn đã nấu xong từ công thức đó, đang bày trên bàn.
- Một công thức (image) nấu được nhiều phần ăn (container) giống hệt nhau.

### 1.2. Container hoạt động thế nào?

Linux kernel có hai "phép thuật":

| Công nghệ | Tác dụng | Nói nôm na |
|---|---|---|
| **Namespaces** | Cô lập: mỗi container tưởng mình là máy riêng (PID, network, mount riêng) | Mỗi container có "phòng riêng" |
| **Cgroups** | Giới hạn tài nguyên CPU, RAM, I/O | Mỗi phòng có "định mức điện nước" |

Nhờ đó container **nhẹ** hơn máy ảo rất nhiều vì không cần cài cả hệ điều hành khách — xài ké kernel của máy host luôn.

### 1.3. So sánh chi tiết: Container vs Virtual Machine (VM)

**Kiến trúc VM** — mỗi VM mang theo **nguyên một hệ điều hành** (vài GB):

```
┌─────────────────────────────────┐
│         Máy vật lý (Host)       │
│  ┌───────────────────────────┐  │
│  │ Hypervisor (VirtualBox...)│  │
│  ├─────────────┬─────────────┤  │
│  │    VM 1     │    VM 2     │  │
│  │ ┌─────────┐ │ ┌─────────┐ │  │
│  │ │Guest OS │ │ │Guest OS │ │  │
│  │ ├─────────┤ │ ├─────────┤ │  │
│  │ │Libs+App │ │ │Libs+App │ │  │
│  │ └─────────┘ │ └─────────┘ │  │
│  └─────────────┴─────────────┘  │
│        Host OS (kernel)         │
└─────────────────────────────────┘
```

**Kiến trúc Container** — không có guest OS, dùng chung kernel host:

```
┌─────────────────────────────────┐
│         Máy vật lý (Host)       │
│  ┌───────────────────────────┐  │
│  │       Docker Engine       │  │
│  ├─────────────┬─────────────┤  │
│  │ Container 1 │ Container 2 │  │
│  │ ┌─────────┐ │ ┌─────────┐ │  │
│  │ │Libs+App │ │ │Libs+App │ │  │
│  │ └─────────┘ │ └─────────┘ │  │
│  └─────────────┴─────────────┘  │
│   Host OS (kernel dùng chung)  │
└─────────────────────────────────┘
```

**Bảng so sánh:**

| Tiêu chí | Container (Docker) | Virtual Machine |
|---|---|---|
| Ảo hóa ở mức | Hệ điều hành (OS-level) | Phần cứng (Hardware-level) |
| Guest OS | Không có — dùng chung kernel host | Có — mỗi VM một OS đầy đủ |
| Dung lượng | Vài MB – vài trăm MB | Vài GB – vài chục GB |
| Thời gian khởi động | Mili-giây đến vài giây | Hàng chục giây đến vài phút |
| Mật độ trên 1 host | Hàng chục đến hàng trăm | Vài đến hàng chục |
| Hiệu năng | Gần như native | Có overhead do hypervisor |
| Cách ly | Vừa (dùng chung kernel) | Mạnh (cách ly hoàn toàn) |
| Chạy OS khác kernel | Không (container Linux cần host Linux) | Có (chạy Windows trên host Linux) |
| Tính di động | Rất cao — image chạy đâu cũng được | Nặng nề, file VM to |
| Use case | Microservices, CI/CD, deploy app | Chạy OS khác, cách ly bảo mật mạnh |

**Tóm lại:** VM là "giả lập cả cái máy tính", container là "giả lập môi trường chạy app". Với 90% nhu cầu deploy app hiện đại, container vừa đủ dùng vừa nhẹ hơn rất nhiều.

---

## 2. Tại sao nên dùng Docker?

### 2.1. Vấn nạn "trên máy tớ chạy ngon mà!"

Kịch bản kinh điển: cậu code trên laptop (Python 3.11, lib X bản 2.3) → đưa bạn chạy thử (Python 3.9, lib X bản 1.8) → **toang** → deploy lên server (Ubuntu khác bản, thiếu lib hệ thống) → **toang tiếp**. Nguyên nhân: **môi trường chạy không nhất quán**.

Docker đóng gói TẤT CẢ (code + runtime + thư viện + config) vào một image duy nhất. Image đó chạy ở đâu cũng cho ra đúng một môi trường. Hết chuyện lệch môi trường.

### 2.2. Lợi ích cụ thể

| Lợi ích | Giải thích |
|---|---|
| Môi trường nhất quán | Dev, test, production dùng chung một image |
| Cài đặt trong 1 lệnh | Cần PostgreSQL? `docker run postgres` — khỏi cài lằng nhằng, khỏi xung đột version |
| Cô lập | Mỗi app một container, thư viện không "đánh nhau"; chạy 3 bản Python khác nhau trên cùng 1 máy dễ dàng |
| Deploy nhanh | Build image một lần, server chỉ việc `docker pull` + `docker run` |
| Dễ scale | Cần 10 bản copy web server? Chạy 10 container, mỗi cái khởi động trong 1 giây |
| Dọn dẹp sạch | Xóa container là sạch bong, không rác config vương vãi như cài tay |
| CI/CD thân thiện | Pipeline test chạy trong container giống hệt production |

### 2.3. Khi nào KHÔNG nên dùng Docker?

- App cần giao diện đồ họa (GUI) phức tạp → Docker hỗ trợ kém.
- Cần cách ly bảo mật mức cao nhất (multi-tenant không tin nhau) → VM an toàn hơn.
- App cần kernel module riêng hoặc phần cứng đặc biệt → container bị hạn chế.

---

## 3. Cài đặt Docker

Hướng dẫn chi tiết trên **Ubuntu/Debian**. macOS/Windows thì cài Docker Desktop từ trang chủ là xong.

### 3.1. Cài Docker Engine trên Ubuntu/Debian (từng bước)

Gỡ bản cũ trước (nếu có):

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

**Bước 1 — Cài gói phụ thuộc:**

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg
```

**Bước 2 — Thêm GPG key chính thức của Docker:**

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

(apt dùng key này để xác thực "gói đúng là của Docker, không bị tráo".)

**Bước 3 — Thêm repository của Docker:**

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

(Dùng Debian thì thay `ubuntu` bằng `debian` trong 2 URL trên.)

**Bước 4 — Cài Docker:**

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

| Gói | Là gì |
|---|---|
| `docker-ce` | Docker Engine (daemon chạy nền) |
| `docker-ce-cli` | Lệnh `docker` trên terminal |
| `containerd.io` | Runtime thực thi container |
| `docker-buildx-plugin` | Build image nâng cao |
| `docker-compose-plugin` | Docker Compose (mục 13) |

**Bước 5 — Kiểm tra cài đặt:**

```bash
sudo docker run hello-world
```

```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Thấy `Hello from Docker!` là thành công. 🎉 Chú ý dòng đầu: Docker không thấy image trên máy → **tự tải từ mạng về** rồi mới chạy — cơ chế này sẽ gặp lại suốt.

### 3.2. Kiểm tra version và service

```bash
docker version          # version client + server
docker info             # thông tin tổng quan: số container/image, driver...

sudo systemctl status docker   # service có đang chạy không?
sudo systemctl enable docker   # tự chạy khi boot máy
```

### 3.3. Chạy Docker KHÔNG cần sudo (rất nên làm)

Mặc định lệnh `docker` cần root vì nó nói chuyện với daemon qua socket `/var/run/docker.sock`. Thêm user vào group `docker` một lần cho xong:

```bash
sudo groupadd docker          # thường đã có sẵn, báo exists thì kệ
sudo usermod -aG docker $USER # -a: append, giữ nguyên group cũ (quên -a là bay hết group!)
newgrp docker                 # áp dụng ngay, khỏi logout
```

Test:

```bash
docker run hello-world   # không cần sudo nữa
```

(Vẫn báo permission denied? Logout/login lại một lần là được.)

> 🔒 User trong group `docker` có quyền tương đương root (mount được cả ổ cứng vào container). Máy cá nhân thì thoải mái, server dùng chung thì cân nhắc.

---

## 4. Khái niệm cốt lõi

4 khái niệm nền tảng — hiểu rõ là nắm được 80% Docker.

### 4.1. Từng khái niệm

**📦 Image** — template **chỉ đọc**, chứa mọi thứ để chạy app: thư viện hệ thống, runtime (Python/Node...), source code, config. Image xếp thành **nhiều lớp (layers)** chồng lên nhau; các layer được **chia sẻ** giữa các image (10 image cùng `FROM ubuntu:22.04` thì lớp Ubuntu chỉ lưu 1 lần). Image **không chạy** — muốn chạy phải tạo container từ nó. Tên image dạng `[registry/][namespace/]tên[:tag]`, vd `nginx:latest`, `postgres:16`.

**🏃 Container** — instance đang chạy (hoặc đã dừng) tạo ra từ image. Khi tạo container, Docker thêm một **lớp đọc-ghi** mỏng lên trên các layer chỉ-đọc; mọi thay đổi file trong container nằm ở lớp này. **Xóa container = mất hết thay đổi** (trừ khi dùng volume — mục 10). Vòng đời: tạo → chạy → dừng → xóa.

**📝 Dockerfile** — file text chứa các instruction (`FROM`, `COPY`, `RUN`, `CMD`...) mô tả cách build ra image từng bước. Viết Dockerfile → `docker build` → ra image.

**🌐 Registry** — kho lưu trữ/phân phối image ("GitHub cho Docker image"). Phổ biến nhất là **Docker Hub** (hub.docker.com): `nginx`, `postgres`, `redis`, `python`... Khác: `ghcr.io`, AWS ECR, hoặc tự host. Luồng: `docker pull` (tải về) → `docker run` (chạy) → `docker push` (đẩy image mình build lên).

### 4.2. Mối quan hệ

```
Dockerfile ──docker build──▶ Image ──docker run──▶ Container
                                  ↕ pull / push
                                Registry (Docker Hub...)
Một image ──▶ chạy được NHIỀU container giống hệt nhau, độc lập nhau
```

Ví dụ một vòng đời đầy đủ:

```bash
docker build -t myapp:v1 .              # từ Dockerfile ra image
docker push myregistry.com/myapp:v1    # đẩy lên registry
docker pull myregistry.com/myapp:v1    # trên server: kéo về
docker run -d --name app1 myregistry.com/myapp:v1
docker run -d --name app2 myregistry.com/myapp:v1   # 2 container từ 1 image
```

### 4.3. Tag là gì?

Tag là "nhãn version" sau dấu `:`, vd `postgres:16`, `python:3.12-slim`. Không ghi tag → mặc định `latest`. **Cẩn thận:** `latest` chỉ nghĩa là "bản build mới nhất", không phải "ổn định nhất" — hôm nay pull và ngày mai pull có thể ra 2 bản khác nhau. Trong production luôn ghi tag cụ thể (`postgres:16.4`).

---

## 5. Lệnh Docker cơ bản

Mỗi lệnh: **cú pháp → giải thích option → ví dụ thật + output mẫu**.

### 5.1. `docker pull` — tải image từ registry

```bash
docker pull [OPTIONS] TÊN_IMAGE[:TAG]
```

| Option | Tác dụng |
|---|---|
| `--platform linux/amd64` | Ép tải image cho kiến trúc CPU cụ thể (vd Mac ARM muốn chạy image amd64) |

```bash
docker pull nginx
```
```
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete
Status: Downloaded newer image for nginx:latest
```

```bash
docker pull postgres:16        # nên ghi tag cụ thể
docker pull redis:7-alpine
```

> 💡 `docker run` sẽ **tự pull** nếu image chưa có, nên `pull` tay thường chỉ để kiểm tra "image này có tồn tại không".

### 5.2. `docker images` — liệt kê image

```bash
docker images [OPTIONS] [TÊN_IMAGE[:TAG]]
```

| Option | Tác dụng |
|---|---|
| `-a` | Hiện cả layer trung gian (hiếm khi cần) |
| `-q` | Chỉ in IMAGE ID (để xóa hàng loạt) |
| `--digests` | Hiện thêm digest (hash duy nhất của image) |

```bash
docker images
```
```
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    39286ab8a5e1   3 weeks ago    188MB
postgres     16        5e06c0a3ebc7   4 weeks ago    432MB
redis        7-alpine  8f2b3c4d5e6f   2 months ago   43.8MB
```

```bash
docker images nginx   # lọc theo tên
docker images -q      # chỉ lấy ID, dùng trong script
```

### 5.3. `docker rmi` — xóa image

```bash
docker rmi [OPTIONS] IMAGE [IMAGE...]
```

| Option | Tác dụng |
|---|---|
| `-f` | Ép xóa (khi image đang bị container dùng — cẩn thận!) |

```bash
docker rmi hello-world:latest
docker rmi d2c94e258dcb        # xóa theo ID, chỉ cần vài ký tự đầu
docker rmi nginx:latest redis:7-alpine   # xóa nhiều cái
```
```
Untagged: hello-world:latest
Deleted: sha256:d2c94e258dcb023bacb24e684d0f3fe3b0d7b2b26c44a3289c91c...
```

> ⚠️ Image đang có container dùng (dù đã dừng) thì không xóa được:
> `conflict: unable to delete ... (must be forced) - image is being used by stopped container`.
> Fix: `docker rm` container trước, hoặc `docker rmi -f`.

### 5.4. `docker run` — lệnh quan trọng NHẤT

`docker run` làm 3 việc trong 1 lệnh: **pull image** (nếu chưa có) → **tạo container** → **chạy container**.

```bash
docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARGS...]
```

**Các option phải thuộc:**

| Option | Tác dụng | Ví dụ |
|---|---|---|
| `-d` | Chạy nền (detach), không chiếm terminal | `docker run -d nginx` |
| `-p` | Map port `host:container` | `-p 8080:80` |
| `--name` | Đặt tên container | `--name web` |
| `-v` | Mount volume/bind mount | `-v data:/var/lib/postgresql/data` |
| `-e` | Set biến môi trường | `-e POSTGRES_PASSWORD=secret` |
| `--rm` | Tự xóa container khi nó dừng | dùng cho việc chạy 1 lần |
| `-it` | Interactive + terminal (để "chui" vào container) | `-it ubuntu bash` |
| `--network` | Gắn vào network nào | `--network mynet` |
| `--restart` | Tự restart khi crash/boot máy | `--restart unless-stopped` |

**Giải thích kỹ từng option:**

**`-d` (detach):** Không có `-d`, container chiếm luôn terminal — log in thẳng ra màn hình, `Ctrl+C` là container dừng theo. Có `-d`, container chạy nền, muốn xem log thì `docker logs`.

**`-p 8080:80`:** Container có network riêng, port 80 *bên trong* không ai ngoài host với tới. `-p 8080:80` = "port 8080 của host → chuyển tiếp vào port 80 của container". Nhớ thứ tự: **host trước, container sau** (`-p HOST:CONTAINER`). Map nhiều port: `-p 8080:80 -p 8443:443`.

**`--name web`:** Mặc định Docker đặt tên ngẫu nhiên kiểu `nostalgic_edison` (vui nhưng khó nhớ). Đặt tên để `docker stop web` thay vì `docker stop 8f3a2c...`.

**`-v`:** Không có `-v`, data trong container **mất khi xóa container**. Database mà không mount volume thì xóa container là mất dữ liệu! (Chi tiết mục 10.)

**`-e`:** Truyền config vào container. Nhiều image official yêu cầu biến bắt buộc, vd `postgres` đòi `POSTGRES_PASSWORD`:
```bash
docker run -d --name db -e POSTGRES_PASSWORD=mysecretpassword postgres:16
```

**`--rm`:** Chạy xong tự biến mất, không để lại "xác" container stopped. Hợp cho việc chạy 1 lần:
```bash
docker run --rm ubuntu echo "hello"   # in hello rồi tự xóa
```

**`-it`:** Gộp 2 option: `-i` (giữ stdin mở để gõ lệnh) + `-t` (cấp terminal cho đẹp output). Dùng khi muốn "chui vào" container vọc:
```bash
docker run -it ubuntu bash
# root@8f3a2c1d9e4f:/#  <- đang ở trong container, gõ exit để thoát
```
⚠️ `-it` và `-d` thường không đi cùng nhau: `-it` để tương tác trực tiếp, `-d` để chạy nền.

**Ví dụ tổng hợp:**

```bash
docker run -d \
  --name myweb \
  -p 8080:80 \
  -e NGINX_HOST=example.com \
  -v webdata:/usr/share/nginx/html \
  --restart unless-stopped \
  nginx:latest
```
```
8f3a2c1d9e4f2b7a6c5d4e3f2a1b0c9d8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2
```
Output của `docker run -d` là **container ID**. Đọc lệnh trên: chạy nền, tên `myweb`, map port 8080→80, set env, mount volume, tự restart khi crash, dùng image `nginx:latest`.

### 5.5. `docker ps` — liệt kê container

```bash
docker ps [OPTIONS]
```

| Option | Tác dụng |
|---|---|
| *(không)* | Chỉ container **đang chạy** |
| `-a` | **Tất cả**, kể cả đã dừng — dùng nhiều nhất! |
| `-q` | Chỉ in container ID |
| `-n 5` | 5 container tạo gần nhất |

```bash
docker ps
```
```
CONTAINER ID   IMAGE         STATUS       PORTS                  NAMES
8f3a2c1d9e4f   nginx:latest  Up 2 hours   0.0.0.0:8080->80/tcp   myweb
```
- **STATUS**: `Up 2 hours` (đang chạy) hoặc `Exited (0) 5 minutes ago` (đã dừng).
- **PORTS**: `0.0.0.0:8080->80/tcp` = port 8080 host map vào port 80 container.

```bash
docker ps -a   # xem cả container đã dừng để dọn "xác"
```

### 5.6. `docker stop` / `start` / `restart`

```bash
docker stop CONTAINER...     # dừng "lịch sự": SIGTERM, chờ 10s rồi SIGKILL
docker start CONTAINER...    # chạy lại container đã dừng
docker restart CONTAINER...  # stop + start
```

```bash
docker stop myweb     # dùng tên
docker stop 8f3a2c1d  # hoặc vài ký tự đầu của ID
docker start myweb
docker restart myweb
docker stop -t 30 myweb   # chờ 30s thay vì 10s trước khi kill
```

> `docker kill` gửi SIGKILL ngay (gắt, có thể mất data đang ghi dở). `stop` cho app thời gian tự dọn dẹp — luôn ưu tiên `stop`.

### 5.7. `docker rm` — xóa container

```bash
docker rm [OPTIONS] CONTAINER...
```

| Option | Tác dụng |
|---|---|
| `-f` | Ép xóa cả container đang chạy |
| `-v` | Xóa luôn volume ẩn danh gắn với container |

```bash
docker rm test-ubuntu        # xóa container đã dừng
docker rm -f myweb           # ép xóa container đang chạy
docker rm $(docker ps -aq)   # xóa TẤT CẢ container đã dừng
docker container prune       # tương đương lệnh trên, gọn hơn
```

> ⚠️ `docker rm` chỉ xóa container, không xóa image. Xóa container = mất data ở lớp writable (trừ data trong volume).

### 5.8. `docker logs` — xem log (vũ khí debug số 1)

App trong container in gì ra stdout/stderr thì `docker logs` hiện cái đó.

```bash
docker logs [OPTIONS] CONTAINER
```

| Option | Tác dụng |
|---|---|
| `-f` | Xem live (như `tail -f`); `Ctrl+C` chỉ thoát xem, container vẫn chạy |
| `--tail 50` | Chỉ 50 dòng cuối |
| `--since 10m` | Log từ 10 phút trước |
| `-t` | Hiện timestamp mỗi dòng |

```bash
docker logs myweb --tail 5
```
```
172.17.0.1 - - [10/Oct/2026:03:00:12 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.81.0"
```

```bash
docker logs -f -t db          # xem live log postgres có timestamp
```

Thấy `database system is ready to accept connections` trong log postgres = DB đã sẵn sàng.

### 5.9. `docker exec` — chạy lệnh TRONG container đang chạy

Khác `docker run` (tạo container mới), `docker exec` là "chui vào" container đang chạy.

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARGS...]
```

| Option | Tác dụng |
|---|---|
| `-it` | Mở terminal tương tác |
| `-u` | Chạy với user nào, vd `-u root`, `-u postgres` |
| `-w` | Thư mục làm việc |

```bash
docker exec -it myweb bash          # mở shell trong container
docker exec -it mycontainer sh      # image alpine không có bash thì dùng sh
docker exec myweb nginx -v          # chạy 1 lệnh duy nhất
# nginx version: nginx/1.27.2
docker exec -it db psql -U postgres  # mở psql trong container postgres
```

> Phân biệt: `docker run -it ubuntu bash` = tạo container **mới**; `docker exec -it myweb bash` = chui vào container **đang chạy**.

### 5.10. `docker inspect` — xem "tất tần tật" (JSON)

```bash
docker inspect CONTAINER|IMAGE
docker inspect --format '{{ .NetworkSettings.IPAddress }}' myweb
# 172.17.0.2
docker inspect --format '{{ .Mounts }}' myweb
docker inspect --format '{{ .Config.Env }}' db
```

### 5.11. `docker stats` — Task Manager cho container

```bash
docker stats              # live, refresh mỗi 2s, Ctrl+C để thoát
docker stats --no-stream  # chụp 1 lần rồi thôi (cho vào script)
docker stats db           # chỉ xem container db
```
```
CONTAINER ID   NAME    CPU %   MEM USAGE / LIMIT     MEM %   NET I/O
8f3a2c1d9e4f   myweb   0.05%   12.5MiB / 3.8GiB      0.32%   1.2kB / 2.4kB
c7d8e9f0a1b2   db      0.30%   45.2MiB / 3.8GiB      1.16%   5.6kB / 3.1kB
```

### 5.12. `docker cp` — copy file host ↔ container

```bash
docker cp myweb:/etc/nginx/nginx.conf ./nginx.conf   # container → host
docker cp ./index.html myweb:/usr/share/nginx/html/   # host → container
```

> Tiện để móc file config/log ra xem nhanh. Đừng lạm dụng để deploy code — cách chuẩn là build lại image hoặc mount volume.

### 5.13. Tóm tắt lệnh cơ bản

```bash
docker pull nginx && docker images && docker rmi nginx   # image: tải, liệt kê, xóa

docker run -d --name web -p 8080:80 nginx   # chạy nền
docker ps -a                                 # liệt kê
docker stop web && docker start web && docker restart web
docker rm web        # xóa (đã dừng)
docker rm -f web     # ép xóa

docker logs -f web        # log live
docker exec -it web bash  # chui vào container
docker inspect web        # chi tiết JSON
docker stats              # CPU/RAM live
```

---

## 6. Ví dụ 1: Chạy web server Nginx

Ví dụ end-to-end: chạy Nginx trong Docker và truy cập từ trình duyệt.

```bash
# Bước 1: tải image
docker pull nginx

# Bước 2: chạy container, map port 8080 (host) -> 80 (container)
docker run -d --name myweb -p 8080:80 nginx
# 8f3a2c1d9e4f2b7a6c5d4e3f2a1b0c9d8e7f6a5b4c3d2e1f0a9b8c7d6e5f4a3b2

# Bước 3: kiểm tra
docker ps
```
```
CONTAINER ID   IMAGE     STATUS       PORTS                  NAMES
8f3a2c1d9e4f   nginx     Up 5 seconds 0.0.0.0:8080->80/tcp   myweb
```

**Bước 4:** mở trình duyệt vào **http://localhost:8080** → thấy trang "Welcome to nginx!". Hoặc test bằng curl:

```bash
curl http://localhost:8080
```

**Bước 5 — đổi nội dung trang web:**

```bash
echo "<h1>Chào cậu! Web chạy trong Docker 🐳</h1>" > index.html
docker cp ./index.html myweb:/usr/share/nginx/html/index.html
# Refresh trình duyệt -> thấy dòng chữ của cậu
```

**Bước 6 — xem log truy cập và dọn dẹp:**

```bash
docker logs myweb --tail 3
# 172.17.0.1 - - [10/Oct/2026:03:00:12 +0000] "GET / HTTP/1.1" 200 612 ...

docker stop myweb && docker rm myweb
```

Xong! Deploy web server mà không cần cài nginx lên máy, không lo config gì cả.

---

## 7. Ví dụ 2: Chạy PostgreSQL

```bash
docker run -d \
  --name mydb \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_DB=mydb \
  -v pgdata:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:16
```

| Phần | Ý nghĩa |
|---|---|
| `-e POSTGRES_PASSWORD=...` | **Bắt buộc.** Không có là container báo lỗi và tắt ngay |
| `-e POSTGRES_USER=myuser` | Tạo thêm user (mặc định là `postgres`) |
| `-e POSTGRES_DB=mydb` | Tạo sẵn database lúc khởi động |
| `-v pgdata:/var/lib/postgresql/data` | **Quan trọng!** Named volume giữ data sống sót khi xóa container |
| `-p 5432:5432` | Để DBeaver/code trên host kết nối vào `localhost:5432` |

Kiểm tra đã sẵn sàng chưa:

```bash
docker logs mydb
# ... LOG:  database system is ready to accept connections
```
Lần đầu postgres cần vài giây khởi tạo — thấy dòng trên mới kết nối.

**Kết nối bằng psql trong container:**

```bash
docker exec -it mydb psql -U myuser -d mydb
```
```sql
mydb=# CREATE TABLE users (id SERIAL PRIMARY KEY, name TEXT);
mydb=# INSERT INTO users (name) VALUES ('Tiểu Dươngg'), ('Capybara');
mydb=# SELECT * FROM users;
 id |    name
----+-------------
  1 | Tiểu Dươngg
  2 | Capybara
mydb=# \q
```

**Kết nối từ host** (DBeaver, pgAdmin, Python `psycopg2`...): host `localhost`, port `5432`, user `myuser`, password `mysecretpassword`, db `mydb`.

**Kiểm chứng volume giữ data:**

```bash
docker rm -f mydb   # xóa container, data trong volume KHÔNG mất

docker run -d --name mydb2 \
  -e POSTGRES_PASSWORD=mysecretpassword -e POSTGRES_USER=myuser -e POSTGRES_DB=mydb \
  -v pgdata:/var/lib/postgresql/data -p 5432:5432 postgres:16

docker exec -it mydb2 psql -U myuser -d mydb -c "SELECT * FROM users;"
#  1 | Tiểu Dươngg
#  2 | Capybara        <- data vẫn còn nguyên!
```

Dọn dẹp:

```bash
docker stop mydb2 && docker rm mydb2
docker volume rm pgdata   # xóa luôn data nếu không cần nữa
```

---

## 8. Ví dụ 3: Chạy Redis

```bash
# Chạy Redis bản alpine siêu nhẹ (~44MB)
docker run -d --name myredis -p 6379:6379 redis:7-alpine

docker logs myredis --tail 3
# 1:C 10 Oct 2026 03:10:00.123 * Ready to accept connections

# Kết nối bằng redis-cli trong container
docker exec -it myredis redis-cli
```
```
127.0.0.1:6379> SET ten "Tiểu Dươngg"
OK
127.0.0.1:6379> GET ten
"Tiểu Dươngg"
127.0.0.1:6379> INCR dem
(integer) 1
127.0.0.1:6379> EXPIRE ten 60
(integer) 1
127.0.0.1:6379> exit
```

**Đặt mật khẩu** (phần sau tên image là `[COMMAND]` ghi đè lệnh mặc định):

```bash
docker rm -f myredis
docker run -d --name myredis -p 6379:6379 \
  redis:7-alpine redis-server --requirepass myredispassword

docker exec -it myredis redis-cli -a myredispassword PING
# PONG
```

**Lưu data bền vững** (Redis mặc định chỉ lưu RAM — restart là mất):

```bash
docker rm -f myredis
docker run -d --name myredis -p 6379:6379 \
  -v redisdata:/data \
  redis:7-alpine redis-server --appendonly yes --requirepass myredispassword
# -v redisdata:/data : image official lưu file vào /data
# --appendonly yes   : bật AOF, restart vẫn khôi phục được data
```

Dọn: `docker rm -f myredis && docker volume rm redisdata`

---

## 9. Dockerfile: tự build image

Quy trình: viết `Dockerfile` → `docker build -t tên:tag .` → ra image → `docker run` → container.

### 9.1. Từng instruction

Mỗi instruction (viết IN HOA theo convention) tạo **một layer** trong image.

**`FROM`** — image nền, bắt buộc, dòng đầu tiên:
```dockerfile
FROM python:3.12-slim
```
Chọn image nền phù hợp: `python:3.12-slim` (Python), `node:20-alpine` (Node)... Các biến thể: `python:3.12` full (~1GB), `python:3.12-slim` gọn (~150MB), `python:3.12-alpine` siêu nhẹ (~50MB).

**`WORKDIR`** — thư mục làm việc (như `cd`), chưa có thì tự tạo:
```dockerfile
WORKDIR /app
```
Dùng đường dẫn tuyệt đối. Đừng dùng `RUN cd ...` vì mỗi `RUN` chạy shell riêng — `cd` dòng này không ảnh hưởng dòng sau!

**`COPY`** — chép file từ host vào image:
```dockerfile
COPY requirements.txt .
COPY . .
```
Cú pháp `COPY <nguồn> <đích>`, nguồn tương đối so với build context, `.` ở đích = `WORKDIR` hiện tại.

**`RUN`** — chạy lệnh **lúc build** (cài đặt, biên dịch...), kết quả đóng băng thành layer:
```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```
Mỗi `RUN` = 1 layer → gộp lệnh liên quan bằng `&&`:
```dockerfile
# ❌ 3 layers, cache apt kẹt lại tốn dung lượng
RUN apt-get update
RUN apt-get install -y curl
RUN rm -rf /var/lib/apt/lists/*
# ✅ 1 layer, sạch
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
```
⚠️ `RUN` chạy lúc **build**, `CMD` chạy lúc **run container** — khác nhau hoàn toàn!

**`EXPOSE`** — khai báo port, **chỉ mang tính tài liệu**, không tự mở port:
```dockerfile
EXPOSE 8000
```
Muốn truy cập từ host vẫn phải `-p` khi `docker run`.

**`ENV`** — biến môi trường mặc định trong image (user ghi đè được bằng `-e`):
```dockerfile
ENV APP_ENV=production
```

**`CMD` vs `ENTRYPOINT`** — lệnh chạy khi container khởi động. Cặp dễ nhầm nhất:

| | `CMD` | `ENTRYPOINT` |
|---|---|---|
| Vai trò | Lệnh **mặc định**, dễ bị ghi đè | Lệnh **cố định** |
| `docker run img <lệnh>` | Lệnh mới **thay thế** hoàn toàn | Lệnh mới thành **tham số** gắn vào sau |

```dockerfile
CMD ["python", "app.py"]
```
```bash
docker run myapp                  # -> python app.py
docker run myapp python test.py   # -> python test.py (ghi đè CMD)
```
```dockerfile
ENTRYPOINT ["python", "app.py"]
```
```bash
docker run myapp --debug          # -> python app.py --debug
```

**Dạng shell vs dạng exec:** `CMD python app.py` (qua `/bin/sh -c`) vs `CMD ["python", "app.py"]` (chạy trực tiếp). **Luôn ưu tiên dạng exec** — process nhận signal (SIGTERM khi `docker stop`) trực tiếp, dừng "sạch".

Kết hợp cả hai (pattern phổ biến):
```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8000"]
# mặc định: python app.py --port 8000
# docker run myapp --port 9000  ->  python app.py --port 9000
```

### 9.2. Ví dụ build image thật: Python Flask app

File `app.py`:
```python
from flask import Flask, jsonify
import os

app = Flask(__name__)

@app.route("/")
def home():
    return jsonify(message="Chào cậu! API chạy trong Docker 🐳",
                   env=os.getenv("APP_ENV", "development"))

@app.route("/health")
def health():
    return jsonify(status="ok")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)   # 0.0.0.0 là BẮT BUỘC trong Docker!
```
> Flask mặc định chỉ nghe `127.0.0.1` (localhost *bên trong* container) → ngoài host không vào được dù đã map port.

File `requirements.txt`:
```
flask==3.0.3
```

File `Dockerfile`:
```dockerfile
FROM python:3.12-slim
WORKDIR /app

# Copy riêng requirements trước để tận dụng layer caching:
# chỉ khi requirements.txt đổi mới phải pip install lại
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .
EXPOSE 8000
ENV APP_ENV=production
CMD ["python", "app.py"]
```

**Mẹo caching:** Docker cache từng layer theo thứ tự. Sửa `app.py` rồi build lại → Docker thấy `requirements.txt` không đổi → dùng lại layer `pip install` cũ, chỉ build lại từ `COPY . .`. Nếu copy cả project ngay từ đầu thì sửa 1 dòng cũng phải pip lại từ đầu!

Build và chạy:
```bash
docker build -t myapp:v1 .
```
```
[+] Building 12.3s (10/10) FINISHED
 => [1/4] FROM docker.io/library/python:3.12-slim
 => [2/4] WORKDIR /app
 => [3/4] COPY requirements.txt .
 => [4/4] RUN pip install --no-cache-dir -r requirements.txt
 => naming to docker.io/library/myapp:v1
```
- `-t myapp:v1`: đặt tên cho image. `.`: build context (thư mục gửi cho daemon để build).

```bash
docker run -d --name myapi -p 8000:8000 myapp:v1
curl http://localhost:8000/
# {"env":"production","message":"Chào cậu! API chạy trong Docker 🐳"}
curl http://localhost:8000/health
# {"status":"ok"}
```

Sửa code rồi build lại để thấy cache phát huy:
```bash
docker build -t myapp:v2 .
```
```
[+] Building 2.1s (10/10) FINISHED    # nhanh hơn hẳn!
 => CACHED [4/4] RUN pip install ...  # dùng cache, không pip lại
```

### 9.3. Ví dụ: Node.js app

`package.json`: `{"name":"mynodeapp","dependencies":{"express":"^4.19.2"}}`

`server.js`:
```js
const express = require("express");
const app = express();
app.get("/", (req, res) => res.json({ message: "Hello từ Node.js trong Docker! 🚀" }));
app.listen(3000, "0.0.0.0", () => console.log("Listening on 3000"));
```

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production   # npm ci: đúng version lock file, chuẩn cho Docker
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```
```bash
docker build -t mynodeapp:v1 . && docker run -d --name nodeapi -p 3000:3000 mynodeapp:v1
curl http://localhost:3000/
# {"message":"Hello từ Node.js trong Docker! 🚀"}
```

### 9.4. Multi-stage build — bí kíp image siêu nhẹ

Vấn đề: build app cần toolchain nặng (compiler, node_modules...), nhưng image chạy thật chỉ cần file binary/static cuối. Multi-stage: nhiều `FROM`, stage đầu để build, stage cuối chỉ copy kết quả.

**Ví dụ Go:**
```dockerfile
# ---- Stage 1: build ----
FROM golang:1.22-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app/server .

# ---- Stage 2: chạy ----
FROM alpine:3.20
WORKDIR /app
COPY --from=builder /app/server .   # chỉ copy binary, bỏ cả toolchain ~500MB
EXPOSE 8080
ENTRYPOINT ["./server"]
```
Kết quả: image cuối ~15MB thay vì ~600MB.

**Ví dụ frontend (Vite/React):**
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build          # ra thư mục dist/

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
Pattern này là **chuẩn công nghiệp** cho mọi app cần compile/build.

### 9.5. Options hay dùng của `docker build`

```bash
docker build [OPTIONS] ĐƯỜNG_DẪN_CONTEXT
```
| Option | Tác dụng | Ví dụ |
|---|---|---|
| `-t` | Đặt tên:tag | `-t myapp:v1` (dùng nhiều lần để gắn nhiều tag) |
| `-f` | Dockerfile khác tên/vị trí | `-f Dockerfile.prod .` |
| `--no-cache` | Build lại từ đầu, bỏ cache | Khi nghi cache "dởm" |
| `--build-arg` | Truyền biến lúc build (với `ARG`) | `--build-arg VERSION=1.2` |
| `--target` | Chỉ build đến stage nào (multi-stage) | `--target builder` để debug |

**`ARG` vs `ENV`:**
```dockerfile
ARG APP_VERSION=1.0      # chỉ tồn tại LÚC BUILD
ENV APP_ENV=production   # tồn tại cả lúc build VÀ lúc container chạy
```
```bash
docker build --build-arg APP_VERSION=2.0 -t myapp:v2 .
```

---

## 10. Volumes: lưu trữ bền vững

### 10.1. Tại sao cần?

File container tạo ra nằm ở **lớp writable** gắn với vòng đời container — `docker rm` là mất sạch. Với database, file upload của user... mất data = thảm họa. **Volume** mount một vùng lưu trữ ngoài vào container; data ghi vào đó **sống độc lập với vòng đời container**: xóa container, tạo container mới mount cùng volume → data vẫn còn (đã demo ở mục 7).

### 10.2. Ba loại mount

| Loại | Cú pháp | Data nằm ở đâu | Khi nào dùng |
|---|---|---|---|
| **Named volume** | `-v ten-volume:/path/container` | Docker quản lý (`/var/lib/docker/volumes/...`) | Database, data bền vững — **nên dùng nhất** |
| **Bind mount** | `-v /path/host:/path/container` | Thư mục cụ thể trên host | Dev (sửa code thấy ngay), mount file config |
| **tmpfs** | `--tmpfs /path` | RAM (mất khi dừng) | Data tạm nhạy cảm, không muốn ghi đĩa |

### 10.3. Named volume

```bash
docker volume create mydata
docker volume ls
# DRIVER    VOLUME NAME
# local     mydata

docker volume inspect mydata   # xem data thực sự nằm đâu trên host

docker run -d --name db2 -e POSTGRES_PASSWORD=secret \
  -v mydata:/var/lib/postgresql/data postgres:16

docker volume rm mydata              # xóa (khi không còn container nào dùng)
docker volume ls -f dangling=true    # tìm volume "mồ côi"
```

### 10.4. Bind mount — ví dụ dev workflow

Sửa file trên host là container thấy ngay, khỏi build lại image mỗi lần sửa 1 dòng:

```bash
docker run -d --name devweb -p 8080:80 \
  -v ~/myweb:/usr/share/nginx/html nginx:alpine
# Sửa ~/myweb/index.html -> refresh trình duyệt thấy ngay
```

**Bind mount file config (nên thêm `:ro` = read-only):**
```bash
docker run -d --name web2 -p 8081:80 \
  -v $(pwd)/my-nginx.conf:/etc/nginx/nginx.conf:ro nginx:alpine
```

> ⚠️ Bind mount ghi đè toàn bộ nội dung đích trong container — mount nhầm thư mục hệ thống là toang. Trên Docker Desktop (Mac/Windows), bind mount hơi chậm với project nhiều file nhỏ.

**Tóm tắt chọn loại:**
```
Data bền vững (DB, uploads) -> named volume
Dev, sửa code thấy ngay      -> bind mount
File config                  -> bind mount + :ro
Data tạm, nhạy cảm           -> tmpfs
```

---

## 11. Networks

### 11.1. Các driver

| Driver | Ý nghĩa | Khi nào dùng |
|---|---|---|
| **bridge** | Network ảo riêng — **mặc định** | Hầu hết mọi trường hợp trên 1 host |
| **host** | Container dùng chung network stack với host | Cần hiệu năng tối đa |
| **none** | Không có network | Job offline, cô lập hoàn toàn |
| **overlay** | Xuyên nhiều host (Swarm/K8s) | Nâng cao |

Mặc định container vào network `bridge`, ở đó container **chỉ gọi nhau bằng IP** (IP đổi mỗi lần tạo lại — bất tiện). **Best practice: tự tạo network riêng** — Docker bật DNS nội bộ, container gọi nhau bằng **tên container**.

```bash
docker network create mynet
docker network ls
docker network inspect mynet   # xem container nào đang gắn, subnet...
```

### 11.2. Ví dụ: app + database gọi nhau bằng tên

```bash
docker network create appnet

docker run -d --name db --network appnet \
  -e POSTGRES_PASSWORD=secret -e POSTGRES_DB=mydb \
  -v pgdata:/var/lib/postgresql/data postgres:16
```

App Python kết nối tới hostname `db` (tên container — Docker DNS tự resolve, không cần biết IP):

```python
import psycopg2, time
for i in range(10):
    try:
        conn = psycopg2.connect(host="db", port=5432, user="postgres",
                                password="secret", dbname="mydb")
        break
    except Exception as e:
        print("Chờ DB...", e); time.sleep(2)
cur = conn.cursor(); cur.execute("SELECT version();")
print("OK:", cur.fetchone()[0][:50]); conn.close()
```
```dockerfile
FROM python:3.12-slim
WORKDIR /app
RUN pip install --no-cache-dir psycopg2-binary
COPY dbapp.py .
CMD ["python", "dbapp.py"]
```
```bash
docker build -t dbapp:v1 .
docker run --rm --network appnet dbapp:v1
# OK: PostgreSQL 16.4 on x86_64-pc-linux-gnu, compiled by gcc
```

Thử ping bằng tên:
```bash
docker run --rm --network appnet alpine ping -c 2 db
# PING db (172.18.0.2): 56 data bytes ... 0.123 ms
```

> 💡 Trong cùng user-defined network, container thấy nhau qua **mọi port app đang nghe** — không cần `-p`. `-p` chỉ cần khi **host/ngoài internet** cần truy cập vào.

### 11.3. Lệnh network hay dùng

```bash
docker network connect mynet con       # gắn container đang chạy vào thêm network
docker network disconnect mynet con    # gỡ ra
docker network rm mynet                # xóa (gỡ hết container trước)
docker network prune                   # dọn network không dùng
```

**`--network host`:** container dùng thẳng network của host, không cần `-p`:
```bash
docker run -d --network host nginx   # nginx nghe thẳng port 80 của host
```
Ưu: hiệu năng cao nhất. Nhược: mất cô lập, dễ xung đột port, **chỉ đầy đủ trên Linux**.

---

## 12. Biến môi trường

Cách chuẩn để truyền config vào container mà không build lại image: mật khẩu DB, API key, dev/prod...

### 12.1. Ba cách set

**`-e` (trực tiếp):**
```bash
docker run -d --name db -e POSTGRES_PASSWORD=supersecret -e POSTGRES_USER=admin postgres:16
```

**`--env-file` (từ file — gọn và an toàn hơn):**
```ini
# file .env (đừng commit lên git nếu có secret!)
POSTGRES_PASSWORD=supersecret
POSTGRES_USER=admin
POSTGRES_DB=myappdb
APP_DEBUG=false
```
```bash
docker run -d --name db --env-file .env postgres:16
```

**`ENV` trong Dockerfile** (giá trị mặc định của image):
```dockerfile
ENV APP_ENV=production
```

**Thứ tự ưu tiên** (sau ghi đè trước): `ENV` trong Dockerfile < `--env-file` < `-e` dòng lệnh.

### 12.2. Kiểm tra & bảo mật

```bash
docker exec mycontainer env | sort
docker exec db printenv POSTGRES_PASSWORD
```

- `-e` không hiện trong `docker ps` nhưng hiện trong `docker inspect` — ai có quyền docker đều xem được. Secret production thật sự thì dùng Docker secrets (Swarm), Vault, hoặc mount file `:ro`.
- **Đừng hardcode password/API key bằng `ENV` trong Dockerfile** nếu image push lên registry public — ai pull về cũng `docker inspect` ra được!

---

## 13. Docker Compose

### 13.1. Vấn đề Compose giải quyết

Dựng stack "web + postgres + redis" bằng `docker run` thuần: 5-6 lệnh dài, phải đúng thứ tự, khó chia sẻ ("cậu chạy lại đúng 5 lệnh này nhé..."). **Docker Compose** gom tất cả vào **một file YAML**, một lệnh `docker compose up` là dựng cả stack.

> Compose V2 dùng `docker compose` (không gạch ngang). `docker-compose` (có gạch) là V1 cũ đã deprecated.

### 13.2. File `docker-compose.yml` — ví dụ full stack

```yaml
services:
  web:
    build: .                  # build từ Dockerfile ở thư mục hiện tại
    # image: myregistry/web:v1  # hoặc dùng image có sẵn
    container_name: myweb
    ports:
      - "8000:8000"
    environment:
      - APP_ENV=production
      - DB_HOST=db            # tên service = hostname trong network!
      - DB_PASSWORD=${DB_PASSWORD}   # lấy từ file .env
      - REDIS_HOST=cache
    depends_on:
      - db
      - cache
    restart: unless-stopped
    networks:
      - appnet

  db:
    image: postgres:16
    container_name: mydb
    environment:
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=myappdb
    volumes:
      - pgdata:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - appnet
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7-alpine
    container_name: mycache
    command: redis-server --appendonly yes
    volumes:
      - redisdata:/data
    restart: unless-stopped
    networks:
      - appnet

volumes:
  pgdata:
  redisdata:

networks:
  appnet:
    driver: bridge
```

File `.env` cùng thư mục:
```ini
DB_PASSWORD=supersecret123
```

| Key | Ý nghĩa |
|---|---|
| `build: .` / `image:` | Build từ Dockerfile hoặc dùng image có sẵn |
| `ports: ["8000:8000"]` | Map port, như `-p` |
| `environment:` | Biến môi trường, như `-e` |
| `${DB_PASSWORD}` | Lấy từ file `.env` hoặc biến shell |
| `depends_on:` | Thứ tự khởi động (chỉ đảm bảo "đã start", chưa chắc "đã sẵn sàng" — muốn chắc dùng `healthcheck`) |
| `volumes:` | Mount volume, như `-v` |
| `restart: unless-stopped` | Tự restart khi crash |
| `command:` | Ghi đè CMD của image |
| `healthcheck:` | Kiểm tra service đã "khỏe" chưa |

### 13.3. Các lệnh Compose

```bash
docker compose up -d            # dựng cả stack nền (tạo network/volume, chạy container)
```
```
[+] Running 8/8
 ✔ Network myapp_appnet      Created
 ✔ Volume myapp_pgdata       Created
 ✔ Container mydb            Started
 ✔ Container mycache         Started
 ✔ Container myweb           Started
```
```bash
docker compose ps               # trạng thái các service
```
```
NAME      SERVICE   STATUS              PORTS
mycache   cache     Up 2 minutes
mydb      db        Up 2 minutes (healthy)
myweb     web       Up 2 minutes        0.0.0.0:8000->8000/tcp
```
```bash
docker compose logs -f          # log tất cả, live
docker compose logs -f web      # chỉ service web
docker compose logs --tail 50 db

docker compose stop             # dừng (giữ container — start lại được)
docker compose start            # chạy lại sau stop
docker compose down             # dừng + XÓA container, network (giữ volume)
docker compose down -v          # + xóa LUÔN volumes (mất data! cẩn thận)

docker compose build            # build lại image
docker compose up -d --build    # build + up trong 1 lệnh

docker compose exec db psql -U postgres -d myappdb   # như docker exec
docker compose run --rm web python -c "print('test')"  # chạy 1 lần rồi xóa
docker compose up -d --scale web=3   # scale: 3 bản copy của web
```

---

## 14. Best practices

**1. Dùng image nền nhỏ:**
```dockerfile
FROM python:3.12          # ❌ ~1GB
FROM python:3.12-slim    # ✅ ~150MB
FROM python:3.12-alpine  # ✅ ~50MB (đôi khi thiếu lib, cần test)
```
Image nhỏ = pull/deploy nhanh, ít lỗ hổng bảo mật.

**2. Luôn có `.dockerignore`** (như `.gitignore` — loại khỏi build context, tránh image phình và lộ secret):
```
.git
__pycache__
*.pyc
.venv/
node_modules/
.env
*.pem
*.key
.DS_Store
.vscode/
Dockerfile
.dockerignore
```

**3. Không chạy bằng root:**
```dockerfile
RUN useradd --create-home appuser && chown -R appuser:appuser /app
USER appuser   # mọi lệnh sau (kể cả CMD) dùng user này
```
Lưu ý: non-root không ghi được thư mục của root (phải `chown` trước) và không bind được port < 1024 → app nên nghe port > 1024.

**4. Tận dụng layer caching — ít thay đổi để lên trên:**
```dockerfile
# ✅ requirements ít đổi -> copy + install trước (cache lâu)
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
# ❌ copy code trước -> sửa 1 dòng cũng pip lại từ đầu
```

**5. Gộp `RUN` và dọn trong cùng layer** (xóa ở layer sau vẫn tốn dung lượng ở layer trước!):
```dockerfile
RUN apt-get update \
    && apt-get install -y --no-install-recommends gcc \
    && rm -rf /var/lib/apt/lists/*
```

**6. Tag version cụ thể:** `postgres:16.4` thay vì `postgres:latest` — hôm nay pull và tháng sau pull không được ra 2 bản khác nhau.

**7. Một container — một nhiệm vụ:** web riêng, db riêng, redis riêng. Cần nhiều thứ → Compose nhiều container, đừng nhét tất cả vào 1 container.

**8. Bonus — `HEALTHCHECK`:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1
```
`docker ps` sẽ hiện `(healthy)`/`(unhealthy)` — orchestrator dùng để biết service thật sự sẵn sàng.

**Checklist trước khi build:**
```
[ ] Image nền slim/alpine, tag version cụ thể
[ ] Có .dockerignore (loại .git, .env, node_modules, __pycache__)
[ ] COPY requirements trước, COPY code sau
[ ] RUN gộp lệnh + dọn cache cùng layer
[ ] CMD dạng exec ["..."], không hardcode secret
[ ] Chạy non-root user (production)
```

---

## 15. Troubleshooting

### 15.1. Port đã được sử dụng
```
Bind for 0.0.0.0:8080 failed: port is already allocated.
```
```bash
docker ps --format '{{.Names}} {{.Ports}}' | grep 8080  # ai đang chiếm?
sudo ss -tlnp | grep 8080
docker stop <ten-container-kia>   # fix 1: dừng nó
docker run -d -p 8081:80 nginx    # fix 2: dùng port khác
```

### 15.2. Permission denied
```
Got permission denied while trying to connect to the Docker daemon socket
```
User chưa vào group `docker` → xem mục 3.3: `sudo usermod -aG docker $USER && newgrp docker` (không được thì logout/login lại).

### 15.3. Image không tìm thấy
```
pull access denied for myapp, repository does not exist or may require 'docker login'
```
| Nguyên nhân | Fix |
|---|---|
| Gõ sai tên | Kiểm tra lại trên Docker Hub |
| Registry private, chưa login | `docker login` |
| Tag không tồn tại | Ghi đúng tag, vd `postgres:16` |
| Mất mạng | `ping 8.8.8.8` |

### 15.4. Container exit ngay sau khi chạy (lỗi gặp NHIỀU NHẤT)

`docker ps` không thấy, `docker ps -a` thấy `Exited (1)`. Nguyên nhân gốc: **tiến trình chính (PID 1) kết thúc → container dừng**.

```bash
docker logs <ten> --tail 50   # bước 1: xem vì sao chết
```

| Log / tình huống | Nguyên nhân | Fix |
|---|---|---|
| `database is uninitialized and superuser password is not set` | Thiếu `POSTGRES_PASSWORD` | Thêm `-e POSTGRES_PASSWORD=...` |
| `executable file not found` | Sai lệnh `CMD` | Sửa CMD |
| Container `ubuntu` exit ngay | Không có tiến trình nền để chạy | `docker run -d ubuntu sleep infinity` hoặc `-it` |
| `Exited (137)` | Hết RAM (OOM) hoặc bị stop | `docker stats` kiểm tra, tăng RAM |
| App crash | Sai config/port | Xem log app, kiểm tra app có nghe `0.0.0.0` không |

### 15.5. Không kết nối được tới app trong container

```bash
docker ps | grep <ten>                              # 1. container có chạy?
docker ps --format '{{.Names}} {{.Ports}}'          # 2. port map đúng? (host:container)
docker logs <ten>                                   # 3. app nghe ở đâu?
docker exec <ten> curl http://localhost:80          # 4. test từ TRONG container trước
curl -v http://localhost:8080                       # 5. rồi mới test từ host
```
Nếu trong container còn không curl được → lỗi app, không phải lỗi Docker. Lỗi phổ biến nhất: app chỉ nghe `127.0.0.1` thay vì `0.0.0.0`.

### 15.6. Hết dung lượng ổ cứng

```bash
docker system df   # xem Docker chiếm bao nhiêu
```
→ Xem mục 16 dọn dẹp ngay bên dưới.

### 15.7. Lệnh "khám bệnh" tổng hợp

```bash
docker ps -a                    # trạng thái container
docker logs <ten> --tail 50     # vì sao chết / log gì
docker inspect <ten>            # config: env, mounts, network...
docker stats --no-stream        # có ngốn RAM/CPU không
docker system df                # ổ cứng
```

---

## 16. Dọn dẹp

Docker dễ "phình": mỗi lần build thử thêm image, mỗi `docker run` quên `--rm` thêm container stopped, mỗi volume test xong quên xóa thêm data rác. Vài tuần là vài chục GB.

```bash
docker system df
```
```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          25        8         6.2GB     4.1GB (66%)
Containers      10        3         250MB     180MB (72%)
Local Volumes   5         2         800MB     300MB (37%)
Build Cache     40                  2.1GB     2.1GB
```

**Dọn từng loại:**
```bash
docker container prune   # container đã dừng
docker image prune       # image dangling (không container nào dùng)
docker image prune -a    # + cả image có tag nhưng không dùng
docker network prune     # network không dùng
docker volume prune      # volume mồ côi (CẨN THẬN: mất data!)
docker builder prune     # build cache
```

**Dọn tất cả một phát:**
```bash
docker system prune
# WARNING! This will remove:
#   - all stopped containers
#   - all networks not used by at least one container
#   - all dangling images
#   - all build cache
# Total reclaimed space: 3.5GB

docker system prune -a --volumes   # bản MẠNH: + image có tag + volumes
```

> ⚠️ `docker system prune -a --volumes` xóa cả **volume không dùng** → **data database có thể mất**. Kiểm tra `docker volume ls` trước khi chạy.

**Thói quen để khỏi phải dọn nhiều:**
```bash
docker run --rm ubuntu echo hello   # việc 1 lần -> tự xóa
docker system df                    # kiểm tra định kỳ
```

---

## 17. Cheat sheet

**Image:**
```bash
docker pull nginx:1.27
docker images
docker rmi nginx:1.27
docker build -t myapp:v1 .
docker tag myapp:v1 myreg.com/myapp:v1
docker push myreg.com/myapp:v1
docker login / docker logout
```

**Container — vòng đời:**
```bash
docker run -d --name web -p 8080:80 nginx
docker run -it --rm ubuntu bash
docker ps / docker ps -a
docker stop web / docker start web / docker restart web
docker rm web / docker rm -f web
```

**Container — quan sát & tương tác:**
```bash
docker logs -f --tail 50 web
docker exec -it web bash
docker exec web ls /app
docker inspect web
docker stats
docker cp web:/app/file ./file
docker cp ./file web:/app/file
```

**`docker run` options:**
```
-d                 chạy nền
-it                interactive + terminal
--name <tên>       đặt tên
-p <host>:<con>    map port (vd -p 8080:80)
-v <src>:<dst>     volume/bind mount
-e KEY=val         biến môi trường
--env-file .env    biến môi trường từ file
--rm               tự xóa khi dừng
--network <net>    gắn network
--restart unless-stopped   tự restart
```

**Volume / Network:**
```bash
docker volume create mydata / docker volume ls / docker volume rm mydata
docker network create mynet / docker network ls / docker network rm mynet
docker network connect mynet con / docker network disconnect mynet con
```

**Compose:**
```bash
docker compose up -d / docker compose up -d --build
docker compose down / docker compose down -v   # -v: xóa cả volumes!
docker compose ps / docker compose logs -f [service]
docker compose stop / docker compose start
docker compose exec db psql -U postgres
```

**Dọn dẹp:**
```bash
docker system df
docker system prune
docker system prune -a --volumes   # mạnh, cẩn thận mất data
```

**Dockerfile:**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
ENV APP_ENV=production
CMD ["python", "app.py"]
```

---

## Lời kết

Chúc mừng cậu đã đi hết bài! 🎉 Giờ cậu đã:

- ✅ Hiểu Docker/container/image, khác VM thế nào
- ✅ Cài Docker, chạy không cần sudo
- ✅ Thành thạo `run/ps/logs/exec/inspect/stats/cp...`
- ✅ Chạy được nginx, PostgreSQL, Redis
- ✅ Tự viết Dockerfile, build image, multi-stage build
- ✅ Hiểu volume, network, biến môi trường
- ✅ Dựng cả stack bằng Docker Compose
- ✅ Biết best practices, troubleshoot, dọn dẹp

**Bước tiếp theo:** (1) Dockerize một project thật của cậu; (2) `docker push` image lên registry rồi pull về máy khác; (3) tìm hiểu Kubernetes — "Compose phiên bản siêu cấp" cho nhiều server; (4) CI/CD: GitHub Actions tự build + push image mỗi lần push code.

> 💡 Quy tắc vàng: **data quan trọng → luôn có volume + backup** · **image production → luôn tag version cụ thể** · **không hardcode secret vào image**.

Học Docker là học bằng tay — mở terminal lên và gõ theo từng ví dụ trong bài này nhé. Chúc cậu vui! 🐳
