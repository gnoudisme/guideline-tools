# Hướng Dẫn Command Line Interface (CLI) / Terminal — Từ Zero Đến Hero

> 📖 Dành cho người mới bắt đầu hoàn toàn. Đọc từ đầu đến cuối, làm theo từng ví dụ, bạn sẽ tự tin dùng Terminal như một lập trình viên thực thụ.
>
> Cập nhật: 2026-10-10

---

## Mục lục

1. [CLI/Terminal là gì? Tại sao phải học?](#1-cliterminal-là-gì-tại-sao-phải-học)
2. [Mở Terminal trên các hệ điều hành](#2-mở-terminal-trên-các-hệ-điều-hành)
3. [Cấu trúc lệnh cơ bản](#3-cấu-trúc-lệnh-cơ-bản)
4. [Điều hướng: pwd, ls, cd](#4-điều-hướng-pwd-ls-cd)
5. [Thao tác file/thư mục](#5-thao-tác-filethư-mục)
6. [Xem nội dung file](#6-xem-nội-dung-file)
7. [Tìm kiếm file](#7-tìm-kiếm-file)
8. [Xử lý text: grep, sed, awk, sort, uniq, cut, tr](#8-xử-lý-text)
9. [Pipe (|) và Redirection](#9-pipe--và-redirection)
10. [Quyền file trên Linux](#10-quyền-file-trên-linux)
11. [Biến môi trường](#11-biến-môi-trường)
12. [Bash scripting cơ bản](#12-bash-scripting-cơ-bản)
13. [Quản lý process](#13-quản-lý-process)
14. [Quản lý package với apt](#14-quản-lý-package-với-apt)
15. [Networking: ping, curl, wget, ssh, scp](#15-networking)
16. [Nén và giải nén](#16-nén-và-giải-nén)
17. [Lịch sử lệnh](#17-lịch-sử-lệnh)
18. [Phím tắt Terminal quan trọng](#18-phím-tắt-terminal-quan-trọng)
19. [Troubleshooting: các lỗi thường gặp](#19-troubleshooting-các-lỗi-thường-gặp)
20. [Best practices & Cheat sheet](#20-best-practices--cheat-sheet)

---

## 1. CLI/Terminal là gì? Tại sao phải học?

### 1.1. Định nghĩa đơn giản

**CLI (Command Line Interface)** là cách điều khiển máy tính bằng cách *gõ lệnh văn bản*, thay vì *click chuột vào nút, menu* như bình thường.

**Terminal** (còn gọi là Console, Command Prompt) là cái cửa sổ đen đen mà bạn gõ lệnh vào. Bên trong nó có chương trình gọi là **Shell** (phổ biến nhất là **Bash** trên Linux/macOS, **PowerShell**/`cmd` trên Windows) — chính shell là thứ đọc và thực thi lệnh bạn gõ.

**Prompt** là dòng `user@may-tinh:~$` — cho biết bạn đang đăng nhập với tài khoản nào (`user`), trên máy nào, đang ở thư mục nào (`~` = thư mục home), và `$` nghĩa là user thường (`#` = admin/root).

### 1.2. GUI vs CLI — so sánh

| Tiêu chí | GUI (Giao diện đồ họa) | CLI (Dòng lệnh) |
|---|---|---|
| Cách dùng | Click chuột, kéo thả | Gõ lệnh bằng bàn phím |
| Học ban đầu | Dễ, nhìn là đoán được | Khó hơn, phải nhớ lệnh |
| Tốc độ | Chậm với việc lặp lại | **Nhanh gấp nhiều lần** khi quen |
| Tự động hóa | Không thể (phải làm tay) | **Viết script chạy tự động** |
| Làm việc từ xa | Cần remote desktop nặng | SSH nhẹ, chỉ cần chữ |
| Server | Thường không có GUI | **Chỉ có CLI** |
| Xử lý hàng loạt | Đổi tên 1000 file = ác mộng | 1 lệnh duy nhất |

**Ví dụ:** đổi tên 500 file ảnh `IMG_001.jpg` → `anh-001.jpg`. GUI: click từng file mất cả buổi. CLI — 1 lệnh, 1 giây:

```bash
for f in IMG_*.jpg; do mv "$f" "anh-${f#IMG_}"; done
```

### 1.3. Tại sao BẠN phải học CLI?

1. **Lập trình bắt buộc dùng:** cài thư viện, chạy server, dùng git, deploy web — tất cả đều qua terminal.
2. **Server không có màn hình:** 99% server (AWS, Google Cloud, VPS) chỉ có CLI.
3. **Nhanh hơn khi đã quen:** tìm file, xử lý text — gõ nhanh hơn click nhiều.
4. **Tự động hóa:** viết script một lần, máy tự chạy mỗi ngày.
5. **Hiểu máy tính sâu hơn:** file, quyền, process, mạng hoạt động ra sao — không còn là "hộp đen".

> 💡 **Mẹo:** Đừng cố nhớ hết mọi lệnh. Nhớ ~20 lệnh core trong tài liệu này là đủ dùng 90% công việc hàng ngày. Còn lại thì tra Google hoặc gõ `man <lệnh>`.

---

## 2. Mở Terminal trên các hệ điều hành

### 2.1. Linux (Ubuntu/Debian...)
- **Phím tắt:** `Ctrl + Alt + T` (nhanh nhất, nhớ cái này!)
- **Menu:** Applications → tìm "Terminal"
- **Chuột phải** vào thư mục → "Open in Terminal"

### 2.2. macOS
- **Spotlight:** `Cmd + Space` → gõ `Terminal` → Enter
- Hoặc cài **iTerm2** (miễn phí) — terminal đẹp và mạnh hơn hẳn
- 📝 macOS mới dùng shell **zsh** thay vì bash — 95% lệnh trong tài liệu này giống hệt nhau.

### 2.3. Windows
- **Windows Terminal** (đẹp): click phải nút Start → "Terminal", hoặc `Win + X`
- **Cũ:** `Win + R` → gõ `cmd` → Enter
- **Dân lập trình nên cài WSL2** — Ubuntu thật chạy trong Windows (`wsl --install` trong PowerShell Admin). 99% tài liệu lập trình viết cho Linux/macOS, có WSL là chạy được hết. Hoặc cài **Git Bash** cho các lệnh cơ bản.

### 2.4. Android — Termux

Điện thoại Android cũng chạy được Linux thật qua app **Termux** (tải trên F-Droid):

```bash
pkg update && pkg upgrade          # cập nhật lần đầu
pkg install git python nodejs openssh
```
Trong Termux bạn gõ `ls`, `cd`, `ssh`... y hệt Linux — tiện để học CLI mọi lúc mọi nơi.

---

## 3. Cấu trúc lệnh cơ bản

Mọi lệnh CLI đều tuân theo một cấu trúc chung:

```
command  [options]  [arguments]
lệnh     [tùy chọn] [đối số]
```

- **command**: tên lệnh, ví dụ `ls`, `cd`, `cp`
- **options** (tùy chọn/cờ): bắt đầu bằng `-` hoặc `--`, dùng để thay đổi cách lệnh hoạt động. Ví dụ `ls -l` (hiển thị chi tiết), `ls --all` (hiện cả file ẩn)
- **arguments** (đối số): thứ mà lệnh sẽ tác động lên, thường là tên file/thư mục. Ví dụ `ls /home` (liệt kê thư mục /home)

**Ví dụ phân tích:**

```bash
ls -l -h /home/user
│  │  │  └──────────── argument: thư mục cần liệt kê
│  │  └─────────────── option: -h (human readable, hiển thị dung lượng dễ đọc)
│  └────────────────── option: -l (long format, hiển thị chi tiết)
└───────────────────── command: ls (list)
```

**Quy tắc viết option:**

```bash
ls -l -a        # viết riêng từng option — OK
ls -la          # gộp lại — OK, tương đương
ls --all        # option dài, dùng 2 gạch — OK
```

**Phân biệt lệnh thành công/thất bại:** shell lưu mã thoát vào `$?` (`0` = thành công):

```bash
ls /tmp && echo "OK" || echo "Lỗi"
```

---

## 4. Điều hướng: pwd, ls, cd

Đây là 3 lệnh bạn sẽ dùng **nhiều nhất**, mỗi ngày, hàng trăm lần. Phải thuộc lòng.

### 4.1. `pwd` — Bạn đang ở đâu?

`pwd` = **p**rint **w**orking **d**irectory — in ra thư mục hiện tại.

```bash
pwd
```
Output mẫu:
```
/home/user/du-an-web
```

**Khi nào dùng:** Mỗi khi bạn "lạc đường" không biết mình đang ở thư mục nào — gõ `pwd` là biết ngay. Dùng rất nhiều trước khi chạy lệnh nguy hiểm như `rm`.

### 4.2. `ls` — Liệt kê file/thư mục

`ls` = **l**i**s**t — liệt kê nội dung thư mục hiện tại.

```bash
ls
```
Output mẫu:
```
Documents  Downloads  anh-dep.jpg  ghi-chu.txt  du-an-web
```

#### Các option quan trọng (học kỹ!)

| Option | Tên đầy đủ | Tác dụng |
|---|---|---|
| `-l` | `--long` | Hiển thị chi tiết: quyền, owner, dung lượng, ngày giờ |
| `-a` | `--all` | Hiện cả file ẩn (tên bắt đầu bằng `.`) |
| `-h` | `--human-readable` | Dung lượng dạng dễ đọc (K, M, G) — dùng kèm `-l` |
| `-t` | | Sắp xếp theo thời gian sửa đổi (mới nhất trước) |
| `-r` | `--reverse` | Đảo ngược thứ tự sắp xếp |
| `-R` | `--recursive` | Liệt kê đệ quy cả thư mục con |
| `-S` | | Sắp xếp theo dung lượng (lớn nhất trước) |
| `-1` | | Mỗi file một dòng |

**Ví dụ 1 — `ls -l` (xem chi tiết):**
```bash
ls -l
```
Output mẫu:
```
total 24
drwxr-xr-x 2 user user 4096 Oct 10 09:15 Documents
drwxr-xr-x 5 user user 4096 Oct 10 08:30 Downloads
-rw-r--r-- 1 user user  532 Oct 10 09:00 ghi-chu.txt
-rw-r--r-- 1 user user 2048 Oct  9 22:11 anh-dep.jpg
```

Giải thích từng cột (đọc từ trái sang):
1. `drwxr-xr-x` — loại + quyền (`d` đầu = thư mục, `-` = file thường). Chi tiết ở mục 10.
2. `user` / `user` — chủ sở hữu và nhóm
3. `4096` — dung lượng (byte)
4. `Oct 10 09:15` — thời gian sửa đổi lần cuối
5. `Documents` — tên file/thư mục

**Ví dụ 2 — `ls -lh` (chi tiết + dung lượng dễ đọc):**
```bash
ls -lh
```
Output mẫu:
```
total 24K
drwxr-xr-x 2 user user 4.0K Oct 10 09:15 Documents
-rw-r--r-- 1 user user  532 Oct 10 09:00 ghi-chu.txt
```
`4096` → `4.0K`, dễ đọc hơn hẳn. **Mẹo:** luôn dùng `lh` đi kèm nhau.

**Ví dụ 3 — `ls -la` (hiện cả file ẩn):**
```bash
ls -la
```
Output mẫu (rút gọn):
```
drwxr-xr-x 5 user user 4096 Oct 10 09:15 .
drwxr-xr-x 3 root root 4096 Oct  1 10:00 ..
-rw-r--r-- 1 user user 3771 Oct  1 10:00 .bashrc
-rw-r--r-- 1 user user  807 Oct  1 10:00 .profile
drwxr-xr-x 2 user user 4096 Oct 10 09:15 Documents
```
- File/thư mục bắt đầu bằng `.` là file ẩn (thường là file cấu hình).
- `.` = thư mục hiện tại, `..` = thư mục cha.

**Ví dụ 4 — kết hợp nhiều option:**
```bash
ls -lath        # chi tiết + ẩn + sắp theo thời gian mới nhất + dễ đọc
ls -lSrh        # chi tiết + sắp theo dung lượng + dễ đọc
ls -R ~/du-an   # liệt kê đệ quy toàn bộ cây thư mục
```

### 4.3. `cd` — Di chuyển giữa các thư mục

`cd` = **c**hange **d**irectory — chuyển thư mục làm việc.

```bash
cd /home/user/Documents     # đường dẫn TUYỆT ĐỐI (bắt đầu bằng /)
cd Documents                # đường dẫn TƯƠNG ĐỐI (từ chỗ đang đứng)
cd ..                       # lên 1 cấp (thư mục cha)
cd ../..                    # lên 2 cấp
cd ~                        # về thư mục home (~/ = /home/user)
cd                          # không argument = về home luôn
cd -                        # quay lại thư mục TRƯỚC ĐÓ (rất tiện!)
```

**Tuyệt đối vs tương đối:**
- **Tuyệt đối:** bắt đầu bằng `/`, đường đi đầy đủ từ gốc. Ví dụ `/home/user/Documents` — đứng ở đâu cũng đến đúng chỗ.
- **Tương đối:** tính từ thư mục hiện tại. Đang ở `/home/user` thì `cd Documents` = vào `/home/user/Documents`.

**Ký hiệu đặc biệt:**

| Ký hiệu | Ý nghĩa | Ví dụ |
|---|---|---|
| `~` | Thư mục home của user | `cd ~/Downloads` |
| `.` | Thư mục hiện tại | `./chay-script.sh` |
| `..` | Thư mục cha | `cd ..` |
| `-` | Thư mục trước đó | `cd -` |
| `/` | Thư mục gốc của cả hệ thống | `cd /` |

**Ví dụ thực tế:**
```bash
pwd                  # /home/user
cd ~/du-an-web       # vào /home/user/du-an-web
cd /tmp              # nhảy sang /tmp
cd -                 # quay lại chỗ trước đó (tiện!)
```

> ⚠️ **Lưu ý:** Tên file/thư mục có dấu cách thì phải bọc trong ngoặc kép: `cd "My Documents"`. Hoặc dùng dấu `\`: `cd My\ Documents`.

---

## 5. Thao tác file/thư mục

### 5.1. `touch` — Tạo file rỗng / cập nhật thời gian

```bash
touch ghi-chu-moi.txt          # tạo file rỗng
touch file1.txt file2.txt      # tạo nhiều file một lúc
touch ghi-chu-moi.txt          # nếu file đã tồn tại → chỉ cập nhật thời gian sửa đổi
```

**Dùng để làm gì:** tạo nhanh file rỗng để viết code, ghi chú. Lập trình viên dùng `touch index.html` mỗi ngày.

### 5.2. `mkdir` — Tạo thư mục

`mkdir` = **m**a**k**e **dir**ectory.

```bash
mkdir du-an-moi              # tạo 1 thư mục
mkdir anh video tai-lieu     # tạo nhiều thư mục cùng lúc
mkdir -p du-an/src/components
# -p (--parents): tạo cả thư mục cha nếu chưa có.
# Không có -p mà du-an/ chưa tồn tại → báo lỗi.
```

**Ví dụ:**
```bash
mkdir -p ~/project/my-app/{src,public,docs}
# Tạo my-app/src, my-app/public, my-app/docs trong 1 lệnh!
# Dấu {} là "brace expansion" của bash — rất tiện.
```

### 5.3. `rmdir` — Xóa thư mục RỖNG

```bash
rmdir thu-muc-rong     # chỉ xóa được khi RỖNG; có file thì dùng rm -r
```

### 5.4. `rm` — Xóa file/thư mục ⚠️ LỆNH NGUY HIỂM NHẤT

`rm` = **r**e**m**ove. **Xóa là mất vĩnh viễn, KHÔNG có thùng rác!**

```bash
rm file-cu.txt              # xóa 1 file
rm file1.txt file2.txt      # xóa nhiều file
rm -r thu-muc/              # -r (--recursive): xóa thư mục + toàn bộ bên trong
rm -f file.txt              # -f (--force): xóa không hỏi, không báo lỗi nếu file không tồn tại
rm -rf thu-muc/             # combo nguy hiểm nhất: xóa thư mục, không hỏi gì cả
```

| Option | Tác dụng |
|---|---|
| `-r` / `-R` | Xóa đệ quy (thư mục + nội dung) |
| `-f` | Ép xóa, bỏ qua file không tồn tại, không hỏi xác nhận |
| `-i` | Hỏi xác nhận từng file trước khi xóa (an toàn) |

```bash
rm -i *.log        # hỏi từng file trước khi xóa (an toàn)
```

> 🔥 **CẢNH BÁO SỐNG CÒN:**
>
> ```bash
> rm -rf /
> ```
> Lệnh này xóa **TOÀN BỘ hệ thống**. Đừng bao giờ gõ, đừng bao giờ đùa.
>
> **3 quy tắc vàng trước khi `rm`:**
> 1. `pwd` trước — chắc chắn mình đang ở đúng thư mục
> 2. `ls` những gì sắp xóa — nhìn tận mắt
> 3. Với `rm -rf`, gõ đường dẫn đầy đủ, KHÔNG dùng wildcard `*` gần `/`
>
> Lệnh `rm file*` khi bạn định gõ `rm file *`... một dấu cách có thể xóa nhầm cả thư mục. Cẩn thận!

### 5.5. `cp` — Copy file/thư mục

`cp` = **c**o**p**y. Cú pháp: `cp <nguồn> <đích>`

```bash
cp ghi-chu.txt ghi-chu-backup.txt       # copy file, đổi tên
cp ghi-chu.txt ~/Documents/             # copy vào thư mục khác (giữ nguyên tên)
cp -r du-an/ du-an-backup/              # -r: copy cả thư mục (bắt buộc với thư mục!)
cp -i file.txt thu-muc/                 # -i: hỏi trước khi ghi đè file đã tồn tại
cp -v file.txt thu-muc/                 # -v: hiện chi tiết đang copy gì (verbose)
```

| `-r` / `-R` | Copy đệ quy (bắt buộc với thư mục) |
| `-i` | Hỏi trước khi ghi đè |
| `-v` | Hiện quá trình (verbose) |

**Ví dụ:**
```bash
cp -rv ~/du-an/ /tmp/du-an-backup/   # -v: hiện từng file đang copy
```

### 5.6. `mv` — Di chuyển / Đổi tên

`mv` = **m**o**v**e. Một lệnh làm 2 việc: **di chuyển** và **đổi tên**.

```bash
mv cu.txt moi.txt                    # ĐỔI TÊN
mv file.txt ~/Documents/             # DI CHUYỂN vào thư mục khác
mv -i file.txt thu-muc/              # -i: hỏi trước khi ghi đè
mv -n file.txt thu-muc/              # -n: KHÔNG ghi đè nếu đã tồn tại
```
**Mẹo:** `mv` trên cùng ổ đĩa gần như tức thì (chỉ đổi tên trong hệ thống file).

### 5.7. Đổi tên hàng loạt với `rename`

```bash
rename 's/IMG_/anh-/' *.jpg       # IMG_001.jpg → anh-001.jpg
rename 'y/A-Z/a-z/' *             # viết thường hết tên file
```
> 📝 Có 2 phiên bản `rename` khác nhau. Nếu trên không chạy, thử `rename cu moi *.txt`.

---

## 6. Xem nội dung file

### 6.1. `cat` — In toàn bộ file ra màn hình

`cat` = con**cat**enate — nối và in nội dung.

```bash
cat ghi-chu.txt
```
Output mẫu:
```
Việc cần làm hôm nay:
- Học CLI
- Viết báo cáo
- Gọi cho mẹ
```

```bash
cat -n ghi-chu.txt     # -n: đánh số dòng
cat file1.txt file2.txt  # nối nhiều file lại in một lượt
cat > moi.txt          # gõ nội dung mới vào file (Ctrl+D để kết thúc + lưu)
```

| Option | Tác dụng |
|---|---|
| `-n` | Đánh số dòng |
| `-s` | Gộp nhiều dòng trống liên tiếp thành 1 |

**Khi nào dùng:** file ngắn (vài chục dòng). File dài thì dùng `less`.

### 6.2. `less` — Xem file dài, cuộn từng trang

```bash
less file-dai.txt
```

**Phím điều khiển trong `less`:** `Space`/`f` xuống trang, `b` lên trang, `/từ-khóa` tìm kiếm (`n`/`N` kết quả tiếp/trước), `g`/`G` đầu/cuối file, `q` thoát.

> 💡 `less` là bản nâng cấp của `more`. Học `less` là đủ.

### 6.3. `head` / `tail` — Xem đầu/cuối file

```bash
head -n 20 file.txt    # 20 dòng ĐẦU (mặc định 10)
tail -n 20 file.txt    # 20 dòng CUỐI
tail -n 50 /var/log/syslog   # 50 dòng log mới nhất của hệ thống
```

#### `tail -f` — THEO DÕI file theo thời gian thực ⭐

```bash
tail -f /var/log/app.log
```

Lệnh này **không thoát** — nó đứng đó và in ra ngay khi có dòng log mới được ghi vào file. Cực kỳ hữu ích khi debug server: mở 1 terminal chạy `tail -f`, mở terminal khác thao tác, nhìn log nhảy realtime.

```bash
tail -F app.log    # như -f nhưng tự theo file mới nếu file bị xoay vòng (logrotate)
```
Thoát bằng `Ctrl + C`.

### 6.4. `wc` — Đếm dòng/từ/ký tự

`wc` = **w**ord **c**ount.

```bash
wc bao-cao.txt
```
Output mẫu:
```
  120   850  5200 bao-cao.txt
   │     │     │
   │     │     └── số byte
   │     └──────── số từ (words)
   └────────────── số dòng (lines)
```

| Option | Tác dụng |
|---|---|
| `-l` | Chỉ đếm dòng (dùng nhiều nhất!) |
| `-w` | Chỉ đếm từ |
| `-c` | Đếm byte |
| `-m` | Đếm ký tự |

```bash
wc -l *.txt              # đếm dòng của mọi file .txt (+ dòng total)
ls ~/du-an | wc -l       # đếm có bao nhiêu file trong thư mục
```

---

## 7. Tìm kiếm file

### 7.1. `find` — Tìm kiếm MẠNH NHẤT (học kỹ!)

Cú pháp: `find <nơi-tìm> <điều-kiện> <hành-động>`

**Tìm theo tên:**
```bash
find . -name "bao-cao*"            # tìm file/thư mục tên bắt đầu bằng "bao-cao" từ thư mục hiện tại
find /home/user -name "*.jpg"     # mọi file .jpg trong /home/user
find . -iname "*.JPG"             # -iname: không phân biệt HOA/thường
```

**Tìm theo loại:**
```bash
find . -type f                    # chỉ file thường (f = file)
find . -type d                    # chỉ thư mục (d = directory)
find . -type f -name "*.log"      # file thường có đuôi .log
```

**Tìm theo thời gian và kích thước:**
```bash
find ~/Downloads -mtime -1        # sửa trong 24h qua
find . -type f -mtime +365        # không đụng đến hơn 1 năm (dọn dẹp!)
find . -size +100M                # file lớn hơn 100MB
find . -size -10k                # file nhỏ hơn 10KB
```
`-mtime -7` = trong 7 ngày qua, `+30` = hơn 30 ngày trước, `-mmin -60` = trong 60 phút qua. Đơn vị size: `c` (byte), `k`, `M`, `G`.

**Hành động với kết quả tìm được:**

```bash
find . -name "*.tmp" -delete          # XÓA luôn các file tìm được (cẩn thận!)
find . -name "*.log" -exec rm {} \;  # -exec: chạy lệnh với từng file ({} = tên file)
find . -type f -name "*.txt" -exec wc -l {} \;
# Đếm dòng từng file .txt tìm được
```

Giải thích `-exec rm {} \;`: với mỗi file tìm được, thay `{}` bằng tên file rồi chạy lệnh; `\;` báo kết thúc.

```bash
# Tìm file .jpg > 5MB, sửa trong 30 ngày qua, liệt kê chi tiết
find ~/Pictures -type f -name "*.jpg" -size +5M -mtime -30 -exec ls -lh {} \;
```

### 7.2. `locate` — Tìm NHANH bằng database

```bash
locate bao-cao.txt        # tìm file có tên chứa "bao-cao.txt" trên TOÀN hệ thống — cực nhanh
```

`locate` nhanh vì nó tra trong database đã index sẵn, không quét ổ đĩa như `find`. Nhược điểm: database có thể cũ (file mới tạo chưa có).

```bash
sudo updatedb    # cập nhật database (chạy khi thấy locate không tìm ra file mới)
```

### 7.3. `which` / `whereis` — Tìm lệnh nằm ở đâu

```bash
which python3        # /usr/bin/python3 — lệnh này thực thi file nào
which -a python      # -a: liệt kê TẤT CẢ vị trí tìm thấy
whereis python3      # liệt kê cả binary, source, trang manual
```
**Dùng khi nào:** kiểm tra công cụ đã cài chưa, hoặc đang dùng bản nào trong nhiều bản.

---

## 8. Xử lý text

Đây là "siêu năng lực" của CLI: xử lý văn bản nhanh hơn bất kỳ trình soạn thảo nào.

### 8.1. `grep` — Tìm kiếm BÊN TRONG file ⭐⭐⭐

`grep` = **g**lobal **r**egular **e**xpression **p**rint. Lệnh quan trọng bậc nhất khi làm việc với log và code.

Cú pháp: `grep [options] "mẫu-tìm" [file...]`

```bash
grep "lỗi" app.log              # tìm mọi dòng chứa chữ "lỗi" trong app.log
```

Output mẫu:
```
2026-10-10 09:01:22 lỗi kết nối database
2026-10-10 09:05:47 lỗi timeout khi gọi API thanh toán
```

#### Các option phải thuộc:

| Option | Tác dụng | Ví dụ |
|---|---|---|
| `-i` | Không phân biệt HOA/thường | `grep -i "error" log.txt` |
| `-r` | Tìm đệ quy trong cả thư mục | `grep -r "TODO" ~/du-an/` |
| `-n` | Hiện số dòng | `grep -n "lỗi" app.log` |
| `-v` | ĐẢO NGƯỢC: hiện dòng KHÔNG khớp | `grep -v "^#" config.txt` |
| `-c` | Chỉ đếm số dòng khớp | `grep -c "ERROR" app.log` |
| `-l` | Chỉ hiện TÊN file chứa mẫu | `grep -rl "password" ~/du-an/` |
| `-A/-B/-C 3` | Hiện thêm 3 dòng sau/trước/cả hai | `grep -C 3 "ERROR" app.log` |

**Ví dụ thực tế:**

```bash
# 2. Tìm chữ "TODO" trong toàn bộ source code
grep -rn "TODO" ~/du-an-web/src/

# 3. Đếm số request lỗi 500 trong access log
grep -c " 500 " access.log

# 4. Lọc bỏ dòng trống và dòng comment trong file cấu hình
grep -v "^#" config.conf | grep -v "^$"
```

#### Regex cơ bản với grep

`grep` hỗ trợ **regex** (biểu thức chính quy) — mẫu tìm kiếm nâng cao:

| Ký tự | Ý nghĩa | Ví dụ |
|---|---|---|
| `^` | Bắt đầu dòng | `grep "^ERROR" log.txt` — dòng bắt đầu bằng ERROR |
| `$` | Kết thúc dòng | `grep "xong$"` — dòng kết thúc bằng "xong" |
| `.` | Bất kỳ ký tự nào | `grep "a.c"` khớp "abc", "axc" |
| `*` | Lặp 0+ lần ký tự trước | `grep "ab*c"` khớp "ac", "abc", "abbbc" |
| `[abc]` | Một trong các ký tự | `grep "[0-9]"` — dòng chứa số |
| `[^abc]` | KHÔNG phải các ký tự này | `grep "[^0-9]"` |
| `\.` | Dấu chấm thật (escape) | `grep "\.txt$"` — file kết thúc bằng .txt |

```bash
grep -E "^[0-9]{4}-[0-9]{2}-[0-9]{2}" log.txt   # dòng bắt đầu bằng ngày YYYY-MM-DD
grep -E "(ERROR|CRITICAL)" app.log              # chứa ERROR hoặc CRITICAL
```

### 8.2. `sed` — Thay thế/biến đổi text

`sed` = **s**tream **ed**itor. Dùng nhiều nhất để **tìm và thay thế**: `sed 's/tìm/thay/cờ' file` (`s` = substitute).

**Ví dụ thực tế:**

```bash
# 1. Thay TẤT CẢ "foo" → "bar" (cờ g = global)
sed 's/foo/bar/g' code.txt

# 2. Sửa TRỰC TIẾP vào file (-i) + giữ bản backup
sed -i.bak 's/localhost/192.168.1.10/g' config.env

# 3. Xóa dòng chứa "debug" / chỉ in dòng 10–20
sed '/debug/d' app.log
sed -n '10,20p' file-dai.txt

# 4. Mẫu chứa dấu / thì dùng phân cách khác
sed 's#/home/cu#/home/moi#g' paths.txt
```

> ⚠️ Luôn test `sed` KHÔNG có `-i` trước (in ra màn hình xem đúng chưa), rồi mới thêm `-i` để sửa file thật.

### 8.3. `awk` — Xử lý text theo CỘT ⭐

`awk` coi mỗi dòng gồm nhiều **cột** (cách nhau bởi khoảng trắng/tab). `$1` = cột 1, `$2` = cột 2, `$0` = cả dòng.

```bash
# File diem.txt:
# An 8 9 7
# Binh 6 5 8

awk '{ print $1, $3 }' diem.txt    # in cột 1 (tên) và cột 3
```
Output:
```
An 9
Binh 5
```

**Ví dụ thực tế:**

```bash
# 1. Tính tổng dung lượng file (cột 5 của ls -l)
ls -l | awk '{ sum += $5 } END { print "Tổng:", sum, "bytes" }'

# 2. Lấy PID + %CPU + tên lệnh của process
ps aux | awk '{ print $2, $3"%", $11 }' | head -5

# 3. Đổi phân cách cột (-F), ví dụ file CSV
awk -F',' '{ print $1 " - " $3 }' data.csv

# 4. Chỉ in dòng thỏa điều kiện (cột 2 > 8)
awk '$2 > 8 { print $1 ": giỏi" }' diem.txt

# 5. Đếm số dòng (NR = số dòng đã đọc)
awk 'END { print NR }' file.txt
```

### 8.4. `sort` — Sắp xếp dòng

```bash
sort ten.txt              # sắp xếp A-Z (mặc định theo alphabet)
sort -r ten.txt           # -r: đảo ngược (Z-A)
sort -n so.txt            # -n: sắp theo SỐ (không có -n thì "10" đứng trước "2"!)
sort -u ten.txt           # -u: sắp + loại trùng (unique)
sort -t',' -k2 data.csv   # -t: phân cách cột là dấu phẩy, -k2: sắp theo cột 2
```

**Ví dụ:**
```bash
cat so.txt
# 10
# 2
# 30
sort so.txt      # 10, 2, 30  (sai! so theo chuỗi)
sort -n so.txt   # 2, 10, 30  (đúng!)
```

### 8.5. `uniq` — Lọc dòng trùng (đi kèm sort)

```bash
uniq ten.txt           # bỏ dòng trùng LIÊN TIẾP (nhớ sort trước!)
sort ten.txt | uniq -c  # -c: đếm số lần xuất hiện
sort ten.txt | uniq -d  # -d: chỉ hiện dòng BỊ TRÙNG
```

### 8.6. `cut` — Cắt cột/ký tự

```bash
# File /etc/passwd dùng dấu : phân cách, cột 1 = username
cut -d':' -f1 /etc/passwd     # -d: delimiter, -f: field (cột)
# root
# user
# ...

cut -c1-10 file.txt           # cắt ký tự 1-10 của mỗi dòng
echo "2026-10-10" | cut -d'-' -f2   # 10 (lấy tháng)
```

### 8.7. `tr` — Chuyển đổi ký tự

```bash
echo "Xin Chao" | tr 'A-Z' 'a-z'     # viết thường hết → "xin chao"
echo "Xin Chao" | tr 'a-z' 'A-Z'     # VIẾT HOA HẾT → "XIN CHAO"
echo "a   b    c" | tr -s ' '        # -s: gộp khoảng trắng liên tiếp → "a b c"
tr -d '0-9' < file.txt               # -d: XÓA hết chữ số
```

---

## 9. Pipe (|) và Redirection

Đây là phần **QUAN TRỌNG NHẤT** để bạn tư duy như dân CLI: **ghép nhiều lệnh nhỏ thành pipeline xử lý mạnh mẽ.**

### 9.1. Pipe `|` — Nối đầu ra lệnh này vào đầu vào lệnh kia

```
lệnh1 | lệnh2 | lệnh3
```

Output của `lệnh1` không in ra màn hình mà chảy thẳng vào `lệnh2` như input.

**Ví dụ từ đơn giản đến thực tế:**

```bash
# 1. Liệt kê file rồi đếm có bao nhiêu file
ls | wc -l

# 2. Tìm process chrome đang chạy
ps aux | grep chrome

# 3. Top 5 file lớn nhất (dòng đầu là "total" nên lấy 6)
ls -lhS | head -6

# 4. Đếm số lượng từng loại file trong thư mục
ls | sed 's/.*\.//' | sort | uniq -c | sort -nr
# ls → sed giữ lại đuôi file → đếm mỗi loại → sắp giảm dần
# Output mẫu:
#       15 jpg
#        8 txt

# 5. Tìm 10 dòng log lỗi mới nhất chứa "timeout"
grep "timeout" app.log | tail -10
```

> 💡 **Tư duy pipeline:** Mỗi lệnh làm MỘT việc nhỏ thật tốt, rồi nối chúng lại. Đây là triết lý Unix: "Do one thing and do it well".

### 9.2. Redirection — Chuyển hướng vào/ra file

Mỗi lệnh có 3 "luồng": **stdin** (0, đầu vào — bàn phím), **stdout** (1, đầu ra — màn hình), **stderr** (2, lỗi — màn hình).

#### `>` — Ghi đè output vào file

```bash
ls -lh > danh-sach.txt       # kết quả ls được LƯU vào file (không in ra màn hình)
echo "xin chào" > chao.txt   # tạo file chao.txt chứa "xin chào"
```
> ⚠️ `>` **GHI ĐÈ** — file cũ mất hết nội dung! Muốn giữ nội dung cũ thì dùng `>>`.

#### `>>` — Ghi NỐI TIẾP vào cuối file

```bash
echo "dòng 1" > nhat-ky.txt    # tạo mới
echo "dòng 2" >> nhat-ky.txt   # nối thêm vào cuối
cat nhat-ky.txt
# dòng 1
# dòng 2
```

#### `<` — Lấy input từ file

```bash
wc -l < bao-cao.txt      # đếm dòng, input lấy từ file
sort < ten.txt > ten-sap-xep.txt   # đọc từ ten.txt, sắp xếp, ghi vào file mới
```

#### `2>` — Chuyển hướng THÔNG BÁO LỖI

```bash
find / -name "*.conf" 2> loi.txt
# Lỗi "Permission denied" không hiện lên màn hình nữa mà vào file loi.txt
# Màn hình chỉ hiện kết quả tìm được — sạch!

ls file-khong-ton-tai 2> /dev/null
# Lỗi bị ném vào /dev/null = "hố đen", biến mất hoàn toàn
```

#### `2>&1` — Gộp stderr vào stdout

```bash
./script.sh > output.log 2>&1
# Cả output THƯỜNG và output LỖI đều ghi vào output.log
# Thứ tự quan trọng: phải viết > trước, 2>&1 sau!
```

#### `/dev/null` — "Hố đen" của Linux

```bash
./script.sh > /dev/null 2>&1    # chạy nhưng vứt bỏ hết output
```

**Bảng tổng hợp redirection:**

| Cú pháp | Tác dụng |
|---|---|
| `>` | Ghi stdout vào file (ghi đè) |
| `>>` | Ghi stdout nối vào cuối file |
| `<` | Đọc stdin từ file |
| `2>` | Ghi stderr vào file (ghi đè) |
| `2>>` | Ghi stderr nối vào cuối file |
| `2>&1` | Gộp stderr vào stdout |
| `> file 2>&1` | Gộp cả stdout + stderr vào file |
| `&> file` | Viết gọn của `> file 2>&1` |
| `>/dev/null 2>&1` | Vứt bỏ hết output |

**Ví dụ tổng hợp — script backup mini:**
```bash
# Backup thư mục, ghi log cả thành công lẫn lỗi, ẩn output spam ra màn hình
tar -czf backup.tar.gz ~/du-an > backup.log 2>&1
echo "Backup xong lúc $(date)" >> backup.log
```

---

## 10. Quyền file trên Linux

Trên Linux, **mọi file đều có quyền** quy định ai được làm gì. Hiểu quyền là bắt buộc nếu bạn code/deploy.

### 10.1. Đọc chuỗi quyền `rwxr-xr--`

```bash
ls -l script.sh
# -rwxr-xr-- 1 user user 532 Oct 10 09:00 script.sh
#  ▲└┬─┘└┬─┘└┬─┘
#  │ │   │   └── OTHERS (người lạ): r--
#  │ │   └────── GROUP (nhóm): r-x
#  │ └────────── USER/OWNER (chủ file): rwx
#  └──────────── loại: - (file), d (thư mục), l (link)
```

| Ký tự | Tên | Ý nghĩa với FILE | Ý nghĩa với THƯ MỤC |
|---|---|---|---|
| `r` | read | Được đọc nội dung | Được liệt kê file bên trong (`ls`) |
| `w` | write | Được sửa/xóa nội dung | Được tạo/xóa file bên trong |
| `x` | execute | Được CHẠY (script/chương trình) | Được `cd` vào |

**Ví dụ đọc quyền:**
- `-rwxr-xr--`: chủ được đọc+ghi+chạy; nhóm và người lạ chỉ được đọc
- `drwxr-xr-x`: thư mục, chủ toàn quyền; nhóm/người lạ được vào xem nhưng không tạo file
- `-rw-------`: chỉ chủ được đọc/ghi (file riêng tư, ví dụ SSH key!)

### 10.2. `chmod` — Đổi quyền

Có 2 cách: **dạng số** (nhanh) và **dạng ký tự** (dễ hiểu).

#### Dạng số (octal) — mỗi nhóm quyền = 1 số

| Số | Quyền | Giải thích |
|---|---|---|
| 7 | `rwx` | 4+2+1: đọc + ghi + chạy |
| 6 | `rw-` | 4+2: đọc + ghi |
| 5 | `r-x` | 4+1: đọc + chạy |
| 4 | `r--` | 4: chỉ đọc |
| 0 | `---` | không gì cả |

(r=4, w=2, x=1 — cộng lại)

```bash
chmod 755 script.sh      # rwxr-xr-x: script chạy được, ai cũng đọc/chạy được
chmod 644 tai-lieu.txt   # rw-r--r--: file văn bản thông thường
chmod 600 khoa-bi-mat    # rw-------: chỉ mình đọc được (dùng cho SSH key!)
chmod 777 file.txt       # rwxrwxrwx: ai cũng làm được mọi thứ ⚠️ NGUY HIỂM
```

**Các số hay dùng (thuộc lòng):**

| Số | Dùng cho | Ý nghĩa |
|---|---|---|
| `755` | Script, chương trình, thư mục | Chủ toàn quyền, người khác đọc + chạy |
| `644` | File văn bản, ảnh, code | Chủ đọc/ghi, người khác chỉ đọc |
| `600` | File nhạy cảm (key, password) | Chỉ chủ được đọc/ghi |
| `700` | Thư mục riêng tư (`~/.ssh`) | Chỉ chủ được vào |

> 🔥 **Đừng bao giờ `chmod 777` bừa bãi!** Nó cho phép BẤT KỲ AI cũng sửa/xóa/chạy file — lỗ hổng bảo mật nghiêm trọng. Chỉ dùng khi debug tạm thời và hiểu rõ mình làm gì.

#### Dạng ký tự — `u/g/o` + `+/-` + `rwx`

```bash
chmod u+x script.sh       # thêm quyền chạy cho chủ file
chmod g-w tai-lieu.txt    # bỏ quyền ghi của nhóm
chmod o-r secret.txt      # bỏ quyền đọc của người lạ
chmod u=rw,go=r file.txt  # gán chính xác = 644
chmod -R 755 ~/du-an/     # -R: đệ quy cả thư mục con
```
`u`/`g`/`o`/`a` = user/group/others/all; `+` thêm, `-` bỏ, `=` gán chính xác.

### 10.3. `chown` / `chgrp` — Đổi chủ sở hữu

```bash
sudo chown user2 file.txt           # đổi chủ file sang user2
sudo chown user2:nhom2 file.txt     # đổi cả chủ và nhóm
sudo chown -R user2:nhom2 thu-muc/  # đệ quy
sudo chgrp nhom2 file.txt           # chỉ đổi nhóm
```

(Cần `sudo` vì đổi chủ là việc của admin.)

### 10.4. `umask` — Quyền mặc định khi tạo file mới

```bash
umask         # xem: thường là 0022
```
`umask 022` nghĩa là: file mới tạo = `666 - 022` = `644`, thư mục mới = `777 - 022` = `755`.

```bash
umask 077     # file mới tạo ra sẽ là 600 (riêng tư) — dùng khi làm việc nhạy cảm
```

---

## 11. Biến môi trường

### 11.1. Biến môi trường là gì?

Là các **giá trị cấu hình toàn cục** mà mọi chương trình trong terminal đều đọc được: tên user, thư mục home, nơi tìm lệnh...

```bash
env                    # liệt kê TẤT CẢ biến môi trường
echo $HOME             # /home/user — thư mục home
echo $USER             # tên đăng nhập
echo $PWD              # thư mục hiện tại
echo $PATH             # nơi shell tìm lệnh (xem 11.2)
```
Trong bash, lấy giá trị biến thì thêm `$` phía trước (`$HOME`); gán thì không (`TEN="giá trị"`).

### 11.2. `PATH` — Vì sao gõ `ls` là chạy được?

Khi bạn gõ `ls`, shell tìm file thực thi `ls` ở các thư mục liệt kê trong `$PATH`:

```bash
echo $PATH
# /usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
# Các thư mục cách nhau bằng dấu :
```

Shell tìm theo thứ tự từ trái sang phải, thấy đầu tiên thì chạy. Đó là lý do bạn gõ `python3` thay vì `/usr/bin/python3` — và muốn script tự viết gọi được mọi nơi thì cho vào `/usr/local/bin/` hoặc thêm thư mục của nó vào PATH:

```bash
export PATH="$HOME/cong-cu-cua-toi:$PATH"
```

### 11.3. `export` — Tạo biến môi trường

```bash
TEN="An"       # biến thường — chỉ shell hiện tại dùng
echo $TEN               # bạn

export TEN="An"  # biến MÔI TRƯỜNG — chương trình con cũng thấy
export API_URL="https://api.example.com"
export PORT=3000
```

**Biến chỉ tồn tại trong phiên terminal hiện tại.** Đóng terminal là mất. Muốn **lưu vĩnh viễn** → ghi vào file cấu hình shell.

### 11.4. `~/.bashrc` — File cấu hình shell (rất quan trọng!)

Mỗi khi mở terminal mới, bash đọc file `~/.bashrc`. Đây là nơi để lưu biến môi trường, alias... vĩnh viễn:

```bash
nano ~/.bashrc
# Thêm vào cuối file:
export PATH="$HOME/bin:$PATH"
export EDITOR=nano
alias ll='ls -lh'
alias la='ls -la'
alias ..='cd ..'
alias capnhat='sudo apt update && sudo apt upgrade -y'

source ~/.bashrc    # nạp lại ngay, không cần mở terminal mới
```

| File | Khi nào đọc |
|---|---|
| `~/.bashrc` | Mỗi lần mở terminal mới |
| `~/.bash_profile` / `~/.profile` | Khi đăng nhập |
| `~/.zshrc` | Nếu dùng zsh (macOS mặc định) |

---

## 12. Bash scripting cơ bản

**Script** = file text chứa nhiều lệnh, chạy một phát là thực hiện cả loạt. Đây là bước từ "dùng terminal" lên "tự động hóa".

### 12.1. Script đầu tiên

Tạo file `chao.sh`:
```bash
#!/bin/bash
# Dòng trên gọi là SHEBANG — báo cho hệ thống biết chạy file này bằng bash

echo "Xin chào, $USER!"
echo "Hôm nay là $(date +%d/%m/%Y)"
echo "Bạn đang ở: $PWD"
```

Chạy nó:
```bash
chmod +x chao.sh     # cấp quyền chạy (1 lần duy nhất)
./chao.sh            # ./ = "file ở thư mục hiện tại"
# Xin chào, user!
# Hôm nay là 10/10/2026
# Bạn đang ở: /home/user
```

> 📝 **Shebang** `#!/bin/bash` phải là dòng ĐẦU TIÊN — báo cho hệ thống biết chạy file này bằng bash.

### 12.2. Biến trong script

```bash
#!/bin/bash
TEN="Dươngg"              # gán: KHÔNG có dấu cách quanh dấu =
echo "Tên tôi là $TEN"    # lấy giá trị: thêm $ phía trước

# $(...): lấy output của lệnh gán vào biến
HOM_NAY=$(date +%A)
SO_FILE=$(ls | wc -l)
echo "Hôm nay $HOM_NAY, thư mục này có $SO_FILE file"

# Tham số truyền vào: ./script.sh Anh Binh
# $1=Anh, $2=Binh, $0=tên script, $#=số tham số, $@=tất cả
echo "Chào $1 và $2! (tổng $# tham số)"
```

### 12.3. `if/else` — Rẽ nhánh

```bash
#!/bin/bash
read -p "Nhập tuổi của bạn: " TUOI   # read: hỏi user nhập, lưu vào biến

if [ "$TUOI" -ge 18 ]; then
    echo "Bạn đã trưởng thành 🎉"
elif [ "$TUOI" -ge 13 ]; then
    echo "Bạn là teen 😎"
else
    echo "Bạn còn nhỏ 👶"
fi
```

**Toán tử so sánh số:** `-eq` (bằng), `-ne` (khác), `-gt` (lớn hơn), `-lt` (nhỏ hơn), `-ge`/`-le` (lớn/nhỏ hơn hoặc bằng).

**Kiểm tra file/chuỗi:** `-f` (là file), `-d` (là thư mục), `-e` (tồn tại), `-z` (chuỗi rỗng), `-n` (chuỗi không rỗng), `=` (hai chuỗi bằng nhau).

```bash
if [ -f "backup.tar.gz" ]; then
    echo "File backup đã tồn tại, bỏ qua"
else
    tar -czf backup.tar.gz ~/du-an && echo "Đã tạo backup ✅"
fi
```

### 12.4. Vòng lặp `for`

```bash
#!/bin/bash
for TEN in An Binh Chi; do
    echo "Xin chào $TEN!"
done

for F in *.jpg; do
    echo "Đang nén $F..." && gzip "$F"
done

for i in {1..5}; do
    echo "Lần $i"
done
```

### 12.5. Vòng lặp `while`

```bash
#!/bin/bash
DEM=1
while [ $DEM -le 5 ]; do
    echo "Đếm: $DEM"
    DEM=$((DEM + 1))
done

# Đọc từng dòng file
while IFS= read -r DONG; do
    echo ">> $DONG"
done < danh-sach.txt
```

### 12.6. Function — Hàm

```bash
#!/bin/bash
chao_hoi() {
    local TEN=$1
    echo "Xin chào $TEN! 👋"
}
chao_hoi "Dươngg"
chao_hoi "Bạn"

cong() { echo $(($1 + $2)); }   # "return" bằng echo + $()
echo "5 + 3 = $(cong 5 3)"
```

### 12.7. Ví dụ script thực tế — Dọn file tải về

```bash
#!/bin/bash
# don-dep-downloads.sh — phân loại file trong ~/Downloads theo đuôi file

cd "$HOME/Downloads" || exit 1

for FILE in *; do
    [ -f "$FILE" ] || continue
    case "${FILE##*.}" in             # ${FILE##*.} = lấy đuôi file
        jpg|jpeg|png|gif)  THU_MUC="Hinh-anh" ;;
        mp4|mkv|avi)       THU_MUC="Video" ;;
        mp3|wav|flac)      THU_MUC="Nhac" ;;
        pdf|doc|docx|txt)  THU_MUC="Tai-lieu" ;;
        zip|tar|gz|rar)    THU_MUC="Nen" ;;
        *)                 continue ;;
    esac
    mkdir -p "$THU_MUC"
    mv "$FILE" "$THU_MUC/"
    echo "Đã chuyển $FILE → $THU_MUC/"
done
echo "✅ Dọn dẹp xong!"
```

Chạy thử:
```bash
chmod +x don-dep-downloads.sh
./don-dep-downloads.sh
# Đã chuyển anh-001.jpg → Hinh-anh/
# Đã chuyển bao-cao.pdf → Tai-lieu/
# ✅ Dọn dẹp xong!
```

---

## 13. Quản lý process

**Process** (tiến trình) = một chương trình đang chạy. Mỗi process có một số ID gọi là **PID**.

### 13.1. `ps` — Liệt kê process đang chạy

```bash
ps aux | grep chrome                 # tìm process chrome
ps aux --sort=-%cpu | head -10       # 10 process ngốn CPU nhất
```
Cột quan trọng: `PID` (ID process), `%CPU`/`%MEM`, `STAT` (R=đang chạy, S=đang ngủ, Z=zombie), `COMMAND`.

### 13.2. `top` / `htop` — Theo dõi realtime

```bash
top       # bảng process cập nhật liên tục, có sẵn mọi máy
htop      # bản đẹp hơn (cài: sudo apt install htop)
```
**Phím trong `top`:** `q` thoát, `M` sắp theo RAM, `P` sắp theo CPU, `k` kill process (nhập PID).

### 13.3. `kill` / `killall` / `pkill` — Dừng process

```bash
kill 5678            # gửi tín hiệu "kết thúc nhẹ nhàng" (SIGTERM) tới PID 5678
kill -9 5678         # SIGKILL: ép dừng NGAY LẬP TỨC (process treo cứng mới dùng)
killall chrome       # kill theo TÊN (tất cả process tên chrome)
pkill -f "python app.py"  # kill theo mẫu trong command line đầy đủ
```

| Tín hiệu | Số | Tác dụng |
|---|---|---|
| SIGTERM | 15 | Yêu cầu dừng lịch sự (mặc định của `kill`) — process được dọn dẹp |
| SIGKILL | 9 | Ép dừng ngay, không dọn dẹp — chỉ dùng khi SIGTERM không ăn |
| SIGHUP | 1 | Thường dùng để reload config (ví dụ `kill -HUP <pid-nginx>`) |

> ⚠️ Luôn thử `kill <PID>` trước. Chỉ dùng `kill -9` khi process treo cứng, vì nó không cho process lưu dữ liệu/dọn dẹp.

```bash
ps aux | grep "node server.js"   # tìm PID, ví dụ 9876
kill 9876          # thử nhẹ nhàng trước
kill -9 9876       # không chết mới ép
```

### 13.4. `jobs`, `fg`, `bg`, `&` — Chạy nền trong terminal

**Chạy lệnh dưới nền với `&`:**
```bash
./server.sh &        # chạy nền, terminal vẫn gõ lệnh khác được
# [1] 12345  → [số job] PID
```

**Quản lý job:**
```bash
sleep 100 &          # job 1
jobs                 # liệt kê job đang chạy
fg %1                # đưa job 1 ra foreground (Ctrl+Z để tạm dừng)
bg %1                # cho job 1 chạy tiếp dưới nền
kill %1              # kill job 1 (dùng %số-job thay vì PID)
```

**`Ctrl+Z` vs `Ctrl+C`:**
| Phím | Tác dụng |
|---|---|
| `Ctrl + C` | Dừng hẳn process đang chạy |
| `Ctrl + Z` | TẠM DỪNG (suspend) — dùng `fg` để tiếp tục hoặc `bg` cho chạy nền |

### 13.5. `nohup` — Chạy kể cả khi đóng terminal

```bash
nohup ./server.sh > server.log 2>&1 &
# "no hang up" — miễn nhiễm với tín hiệu đóng terminal
```

---

## 14. Quản lý package với apt

Trên Ubuntu/Debian, **apt** là "chợ ứng dụng" dòng lệnh. Quy trình chuẩn:

```bash
sudo apt update        # B1: cập nhật DANH SÁCH package mới nhất
sudo apt install git   # B2: cài package
```
(`sudo` = chạy với quyền admin; hệ thống sẽ hỏi mật khẩu.)

### Các lệnh apt chi tiết:

| Lệnh | Tác dụng | Ví dụ |
|---|---|---|
| `sudo apt update` | Cập nhật danh sách package (không cài gì cả) | Chạy trước khi cài, và mỗi vài ngày |
| `sudo apt upgrade` | Nâng cấp TẤT CẢ package đã cài lên bản mới | `sudo apt upgrade -y` |
| `sudo apt install <tên>` | Cài package mới | `sudo apt install htop vim curl` |
| `sudo apt remove <tên>` | Gỡ package (giữ lại file cấu hình) | `sudo apt remove firefox` |
| `sudo apt purge <tên>` | Gỡ SẠCH (cả file cấu hình) | `sudo apt purge nginx` |
| `sudo apt autoremove` | Gỡ package "mồ côi" không còn ai dùng | Dọn dẹp định kỳ |
| `apt search <từ-khóa>` | Tìm package theo tên/mô tả | `apt search "video editor"` |
| `apt show <tên>` | Xem thông tin chi tiết package | `apt show git` |
| `apt list --installed` | Liệt kê package đã cài | `apt list --installed \| grep python` |

```bash
sudo apt update && sudo apt install -y git curl wget htop vim
# -y: tự động trả lời "yes" cho mọi câu hỏi xác nhận
```

> 📝 Các distro khác dùng package manager khác: Fedora/CentOS dùng `dnf`/`yum`, Arch dùng `pacman`, macOS dùng `brew`. Lệnh khác nhau nhưng ý tưởng giống hệt.

---

## 15. Networking

### 15.1. `ping` — Kiểm tra kết nối tới máy khác

```bash
ping -c 4 google.com    # -c: chỉ ping 4 lần rồi tự dừng
```
- `time=25.3 ms`: độ trễ — càng thấp càng tốt (<50ms ngon, >300ms lag)
- `0% packet loss`: không mất gói tin — mạng ổn định
- Nhấn `Ctrl+C` để dừng (ping chạy mãi không tự dừng trên Linux!)

**Dùng khi nào:** mạng chậm/không vào được web → ping thử xem có phải do mạng mình không.

### 15.2. `curl` — Gọi HTTP/API, tải file ⭐ (dân lập trình dùng mỗi ngày)

`curl` = **c**lient **URL** — công cụ Swiss Army knife cho HTTP.

**GET đơn giản:**
```bash
curl https://api.github.com/users/gnoudisme
```
Output (JSON):
```json
{
  "login": "gnoudisme",
  "id": 12345678,
  ...
}
```

**Các option quan trọng:**

| Option | Tác dụng |
|---|---|
| `-o <file>` / `-O` | Lưu vào file (đặt tên / giữ tên gốc) |
| `-L` | Theo redirect (rất quan trọng!) |
| `-i` / `-I` | Hiện response header / chỉ lấy header |
| `-X POST` + `-d` | Gửi POST với data |
| `-H` | Thêm header (ví dụ Authorization) |
| `-s` | Im lặng (dùng trong script) |

**Ví dụ thực tế:**

```bash
# Tải file
curl -L -o ubuntu.iso https://releases.ubuntu.com/22.04/ubuntu.iso

# POST JSON lên API
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"ten": "Duongg", "tuoi": 20}'

# Gọi API kèm token
curl -H "Authorization: Bearer $TOKEN" https://api.example.com/me

# Kiểm tra web có sống không (chỉ lấy status code)
curl -s -o /dev/null -w "%{http_code}" https://tieuduongg.vercel.app
# 200  → web sống!

# Xem IP public của mình
curl ifconfig.me
```

### 15.3. `wget` — Tải file (đơn giản hơn curl cho việc download)

```bash
wget https://example.com/file.zip                 # tải vào thư mục hiện tại
wget -O ten-moi.zip https://example.com/file.zip  # đổi tên khi lưu
wget -c https://example.com/file-lon.iso          # -c: tiếp tục tải dở (resume!)
```

> 💡 **curl vs wget:** `curl` mạnh hơn cho API (POST, header...), `wget` tiện hơn cho tải đệ quy website (`-r`).

### 15.4. `ssh` — Điều khiển máy/server từ xa ⭐⭐⭐

`ssh` = **s**ecure **sh**ell — mở terminal trên máy khác qua mạng, mọi thứ mã hóa.

```bash
ssh user@192.168.1.100          # kết nối vào server
ssh -p 2222 user@192.168.1.100  # -p: port khác 22 mặc định
```
Lần đầu kết nối gõ `yes`, nhập mật khẩu → bạn đang ở trong terminal của server! Thoát bằng `exit` hoặc `Ctrl+D`.

#### SSH key — đăng nhập không cần mật khẩu (nên làm!)

```bash
# B1: Tạo cặp key trên máy mình (chỉ làm 1 lần) — Enter 3 lần
ssh-keygen -t ed25519 -C "may-tinh-cua-toi"

# B2: Copy public key lên server (nhập mật khẩu lần cuối)
ssh-copy-id user@192.168.1.100

# B3: Từ giờ ssh không cần mật khẩu nữa! 🎉
```

`ssh-keygen` tạo 2 file trong `~/.ssh/`: `id_ed25519` (**private key** — giữ bí mật tuyệt đối!) và `id_ed25519.pub` (public key — đưa lên server thoải mái).

```bash
chmod 600 ~/.ssh/id_ed25519    # private key PHẢI là 600
```

### 15.5. `scp` — Copy file qua SSH

`scp` = **s**ecure **c**o**p**y — cú pháp như `cp` nhưng thêm `user@host:`:

```bash
scp bao-cao.pdf user@192.168.1.100:/home/user/  # lên server
scp user@192.168.1.100:/home/user/data.csv ./   # về máy mình
scp -r ~/du-an-web/ user@192.168.1.100:/var/www/ # cả thư mục (-r)
```

### 15.6. `ss` / `netstat` — Xem kết nối mạng và port

```bash
ss -tuln
# -t: TCP, -u: UDP, -l: đang listen, -n: hiện số
```
Output mẫu:
```
tcp   LISTEN 0      0.0.0.0:80         0.0.0.0:*
tcp   LISTEN 0      0.0.0.0:22         0.0.0.0:*
tcp   LISTEN 0      127.0.0.1:3000     0.0.0.0:*
```
Đọc: máy đang nghe port 80 (web), 22 (SSH), 3000 (app dev, chỉ local).

```bash
ss -tuln | grep 3000      # kiểm tra app đã chạy port 3000 chưa
sudo ss -tlnp             # -p: xem process NÀO đang giữ port
```

> 📝 `netstat` là lệnh cũ, trên Linux mới đã thay bằng `ss` (nhanh hơn). Dùng `ss` nhé.

---

## 16. Nén và giải nén

### 16.1. `tar` — Đóng gói/gỡ gói (phổ biến nhất trên Linux)

`tar` = **t**ape **ar**chive. Thường kết hợp nén gzip → file `.tar.gz` (hoặc `.tgz`).

**Nhớ 2 lệnh này là đủ sống:**

```bash
# NÉN: -czf (create + gzip + file)
tar -czf backup.tar.gz ~/du-an/
#  c = create (tạo), z = gzip (nén), f = file (tên file sau đó)

# GIẢI NÉN: -xzf (extract + gzip + file)
tar -xzf backup.tar.gz
#  x = extract (giải nén)

# Xem bên trong có gì mà không cần giải nén
tar -tzf backup.tar.gz
#  t = list (liệt kê)
```

| Option | Tác dụng |
|---|---|
| `-c` | Tạo archive mới (create) |
| `-x` | Giải nén (extract) |
| `-t` | Liệt kê nội dung (list) |
| `-z` | Nén/giải nén bằng gzip (`.tar.gz`) |
| `-j` | Dùng bzip2 (`.tar.bz2` — nén chặt hơn, chậm hơn) |
| `-J` | Dùng xz (`.tar.xz` — chặt nhất) |
| `-f` | Chỉ định tên file (luôn đứng CUỐI cụm option) |
| `-v` | Hiện chi tiết từng file |
| `-C <thư-mục>` | Giải nén VÀO thư mục chỉ định |

**Ví dụ thực tế:**
```bash
tar -czvf web.tar.gz ~/du-an-web/      # nén (v = hiện từng file)
tar -xzf web.tar.gz -C /tmp/          # giải nén vào /tmp
tar -xzf backup.tar.gz du-an/index.html  # chỉ giải nén 1 file
```

### 16.2. `gzip` / `gunzip` — Nén 1 file đơn

```bash
gzip bao-cao.txt         # → bao-cao.txt.gz (file gốc BỊ XÓA, thay bằng bản nén)
gunzip bao-cao.txt.gz    # giải nén về bao-cao.txt

gzip -k bao-cao.txt      # -k: GIỮ lại file gốc
gzip -9 file.txt         # -9: nén mạnh nhất (chậm hơn); -1: nhanh nhất
```

### 16.3. `zip` / `unzip` — Định dạng quen thuộc với Windows

```bash
zip -r web.zip ~/du-an-web/          # -r: nén cả thư mục
unzip backup.zip                     # giải nén
unzip -l backup.zip                  # xem bên trong (không giải nén)
unzip backup.zip -d /tmp/restore/    # giải nén vào thư mục chỉ định
```

**Khi nào dùng cái nào?**

| Tình huống | Dùng |
|---|---|
| Gửi file cho dân Linux | `tar.gz` |
| Gửi file cho người dùng Windows | `zip` |
| Nén 1 file log lớn nhanh | `gzip` |
| Nén chặt nhất để lưu trữ lâu | `tar -cJf` (xz) |

---

## 17. Lịch sử lệnh

Bash **tự động lưu** mọi lệnh bạn đã gõ. Khai thác nó để khỏi gõ lại!

### 17.1. `history` — Xem lại lệnh đã gõ

```bash
history
```
Output mẫu:
```
  498  ls -lh
  499  cd ~/du-an-web
  500  git status
  501  npm run dev
```

```bash
history | grep git       # tìm lệnh git đã từng gõ
history | tail -20       # 20 lệnh gần nhất
```

### 17.2. Chạy lại lệnh cũ

```bash
!!         # chạy lại lệnh VỪA RỒI
sudo !!    # ⭐ quên sudo? gõ cái này = sudo <lệnh-vừa-rồi>
!500       # chạy lại lệnh số 500 trong history
!git       # chạy lại lệnh gần nhất bắt đầu bằng "git"
```

### 17.3. `Ctrl+R` — Tìm kiếm lệnh siêu nhanh ⭐

Nhấn `Ctrl + R`, gõ vài ký tự → bash tìm lệnh cũ khớp nhất. Nhấn `Ctrl+R` tiếp để xem kết quả cũ hơn, `Enter` để chạy, `Esc` để sửa trước khi chạy, `Ctrl+C` để hủy.

### 17.4. Xóa lịch sử

```bash
history -c    # xóa toàn bộ lịch sử phiên hiện tại
```

---

## 18. Phím tắt Terminal quan trọng

### Điều khiển process & màn hình
| Phím | Tác dụng |
|---|---|
| `Ctrl + C` | Dừng process đang chạy |
| `Ctrl + Z` | Tạm dừng process (tiếp tục bằng `fg`/`bg`) |
| `Ctrl + D` | Kết thúc input / thoát shell (= `exit`) |
| `Ctrl + L` | Xóa màn hình (= `clear`) |
| `Ctrl + S` / `Ctrl + Q` | Đóng băng / mở băng output |

> ⚠️ **Bẫy kinh điển:** màn hình "đơ" không nhận phím? Bạn đã vô tình bấm `Ctrl+S` → bấm `Ctrl+Q` là hết!

### Soạn lệnh
| Phím | Tác dụng |
|---|---|
| `Ctrl + A` / `Ctrl + E` | Về đầu / cuối dòng |
| `Ctrl + U` / `Ctrl + K` | Xóa từ con trỏ về đầu / đến cuối dòng |
| `Ctrl + W` | Xóa 1 từ phía trước con trỏ |
| `Ctrl + Y` | Dán lại thứ vừa xóa |

### Lịch sử & tự hoàn thành
| Phím | Tác dụng |
|---|---|
| `↑` / `↓` | Lệnh trước / sau trong history |
| `Ctrl + R` | Tìm kiếm ngược trong history ⭐ |
| `Tab` | **Tự hoàn thành** tên file/lệnh — bấm liên tục! |
| `Tab` `Tab` | Hiện tất cả gợi ý |

**Mẹo Tab completion — dùng LIÊN TỤC:**
```bash
cd ~/Doc<Tab>        # → tự thành cd ~/Documents/
ls /usr/sh<Tab><Tab> # → hiện: share/  shutdown...
ssh user@192.168.1.<Tab>  # thậm chí gợi ý từ history/known_hosts
```

---

## 19. Troubleshooting: các lỗi thường gặp

### 19.1. `command not found`

```
bash: htop: command not found
```
**Nguyên nhân:** chưa cài, hoặc gõ sai tên.
**Sửa:**
```bash
# Kiểm tra gõ đúng chưa
which htop

# Chưa có thì cài
sudo apt update && sudo apt install htop
```

### 19.2. `Permission denied`

- `bash: ./script.sh: Permission denied` → file chưa có quyền chạy: `chmod +x script.sh`
- `E: Could not open lock file... Permission denied` → quên `sudo`: thêm `sudo` phía trước lệnh

### 19.3. `No such file or directory`

Nguyên nhân thường gặp: đang đứng sai thư mục (`pwd` + `ls` để kiểm tra), gõ sai tên (Linux phân biệt HOA/thường — dùng `Tab` để tự hoàn thành), nhầm đường dẫn tương đối/tuyệt đối.

### 19.4. Lệnh treo không thoát được

Thử `Ctrl + C`. Không được → mở terminal khác, `ps aux | grep <tên-lệnh>` lấy PID rồi `kill -9 <PID>`. Màn hình "đơ" sau khi bấm nhầm `Ctrl+S` → bấm `Ctrl+Q`.

### 19.5. `rm` nhầm file

**Phòng bệnh hơn chữa bệnh:** dùng `rm -i` khi không chắc; tạo alias an toàn trong `~/.bashrc`:
```bash
alias rm='rm -i'    # rm luôn hỏi trước khi xóa
```
Và backup định kỳ (script ở mục 12!). **Lỡ xóa rồi:** ngừng ghi dữ liệu lên ổ đó ngay, thử công cụ `testdisk`/`photorec` — tỉ lệ cứu không cao, nên backup vẫn là vua.

### 19.6. SSH: `Permission denied (publickey)`

**Checklist:** private key đúng quyền 600 chưa (`chmod 600 ~/.ssh/id_ed25519`)? Public key đã lên server chưa (`ssh-copy-id` lại)? Đúng username chưa? Xem log chi tiết: `ssh -v user@host`.

### 19.7. `curl` tải về được file rỗng / HTML lạ

**Nguyên nhân:** URL bị redirect mà thiếu `-L`.
**Sửa:** `curl -L -o file.zip <url>`

### 19.8. Script báo `bad interpreter` hoặc `^M`

**Nguyên nhân:** file viết trên Windows, xuống dòng kiểu CRLF thay vì LF.
**Sửa:** `sed -i 's/\r$//' script.sh` (hoặc `sudo apt install dos2unix && dos2unix script.sh`).

### 19.9. Quy trình debug chung

1. **Đọc kỹ thông báo lỗi** — 90% đáp án nằm trong đó
2. `pwd` + `ls` — chắc chắn mình đang ở đúng chỗ
3. Copy nguyên văn lỗi lên Google — 99% đã có người gặp trước bạn

---

## 20. Best practices & Cheat sheet

### 20.1. Best practices — 10 thói quen tốt

1. **Dùng `Tab` mọi lúc** — vừa nhanh vừa tránh gõ sai tên file.
2. **`pwd` + `ls` trước `rm -rf`** — 3 giây kiểm tra cứu được cả dự án.
3. **Đừng `chmod 777` bừa** — dùng `755`/`644`.
4. **Dùng `>>` thay vì `>`** khi ghi log — tránh ghi đè mất dữ liệu.
5. **Test `sed` không `-i` trước** — xem output đúng rồi mới sửa file thật.
6. **Đặt alias** cho lệnh dài trong `~/.bashrc`.
7. **`Ctrl+R` thay vì gõ lại** lệnh dài đã từng dùng.
8. **Viết script** cho việc lặp lại quá 3 lần.
9. **Quote biến `"$VAR"`** — tránh lỗi khi giá trị có dấu cách.
10. **`--help` trước khi Google** — `ls --help` cho tóm tắt nhanh mọi option.

### 20.2. Cheat sheet tổng hợp

#### Điều hướng & file cơ bản
```bash
pwd                         # đang ở đâu
ls -lh                      # liệt kê chi tiết, dễ đọc
ls -la                      # kèm file ẩn
cd <thư-mục> | cd .. | cd ~ | cd -   # di chuyển
touch <file>                # tạo file rỗng
mkdir -p <a/b/c>            # tạo thư mục (kèm cha)
cp -r <nguồn> <đích>        # copy
mv <cũ> <mới>               # di chuyển / đổi tên
rm <file> | rm -rf <thư-mục>  # xóa (CẨN THẬN!)
```

#### Xem & tìm kiếm
```bash
cat <file>                  # xem toàn bộ
less <file>                 # xem từng trang (q thoát)
head -n 20 <file>           # 20 dòng đầu
tail -n 20 -f <file>        # 20 dòng cuối + theo dõi realtime
grep -rn "mẫu" <thư-mục>    # tìm trong file
find . -name "*.log"        # tìm file theo tên
```

#### Xử lý text
```bash
lệnh1 | lệnh2               # pipe
> file  |  >> file          # ghi đè / nối thêm
sort | uniq -c              # sắp xếp + đếm trùng
awk '{print $1}'            # lấy cột 1
sed 's/cũ/mới/g'            # thay thế
cut -d':' -f1               # cắt cột
```

#### Quyền & biến
```bash
chmod 755 <script>          # quyền chạy script
chmod 644 <file>            # quyền file thường
export TEN="giá trị"        # tạo biến môi trường
echo $PATH                  # xem PATH
source ~/.bashrc            # nạp lại cấu hình
```

#### Process
```bash
ps aux | grep <tên>         # tìm process
kill <PID> | kill -9 <PID>  # dừng process
lệnh &                      # chạy nền
jobs | fg | bg              # quản lý job
nohup lệnh &                # chạy kể cả đóng terminal
```

#### Package (Ubuntu/Debian)
```bash
sudo apt update && sudo apt install <tên>   # cài
sudo apt remove <tên> | sudo apt purge <tên>  # gỡ
apt search <từ-khóa>        # tìm package
```

#### Mạng
```bash
ping -c 4 google.com        # kiểm tra mạng
curl -L -o <file> <url>     # tải / gọi API
ssh user@host               # vào server
scp <file> user@host:<đường-dẫn>  # copy qua SSH
ss -tuln                    # xem port đang nghe
```

#### Nén
```bash
tar -czf <ra>.tar.gz <vào>  # nén
tar -xzf <file>.tar.gz      # giải nén
zip -r <ra>.zip <vào>       # nén zip
unzip <file>.zip            # giải nén zip
```

#### Lịch sử & phím tắt
```bash
history                     # xem lịch sử
sudo !!                     # chạy lại lệnh vừa rồi với sudo
# Ctrl+R: tìm lệnh cũ | Tab: tự hoàn thành
# Ctrl+C: dừng | Ctrl+Z: tạm dừng | Ctrl+L: xóa màn hình
# Ctrl+A/E: đầu/cuối dòng | Ctrl+U/K: xóa đầu/cuối dòng
```

---

## 🎓 Lời kết

Chúc mừng bạn đã đi hết tài liệu! 🎉 Giờ bạn đã nắm: điều hướng + thao tác file, tìm kiếm + xử lý text (grep/sed/awk), pipe + redirection, phân quyền, biến môi trường, viết script tự động hóa, quản lý process, cài phần mềm, và làm việc với server qua SSH.

**Bước tiếp theo:**
1. **Dùng terminal mỗi ngày** — ép mình làm mọi việc bằng CLI trong 1 tuần
2. **Học Git** (`clone`/`commit`/`push`) — công cụ không thể thiếu của dev
3. **Học Vim hoặc Nano** — soạn thảo ngay trong terminal
4. **Thuê 1 VPS rẻ** — vọc server thật là cách học nhanh nhất


*Hết tài liệu. Happy hacking!* ⌨️🚀
