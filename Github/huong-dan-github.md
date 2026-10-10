# Hướng Dẫn Git & GitHub Toàn Tập (Web UI + Git)

> Chào cậu! Cẩm nang từ A–Z cho người mới bắt đầu với **Git** và **GitHub** — giải thích như tâm sự với bạn bè, mỗi lệnh đều có ví dụ thật kèm output. Đọc xong là tự tạo repo, push code, làm nhóm, đóng góp open source ngon lành. 🦫

## Mục Lục

1. [Git là gì? GitHub là gì?](#1-git-là-gì-github-là-gì)
2. [Tại sao nên dùng Git?](#2-tại-sao-nên-dùng-git)
3. [Tạo tài khoản GitHub](#3-tạo-tài-khoản-github)
4. [Tạo repository mới trên web](#4-tạo-repository-mới-trên-web)
5. [Cài đặt Git & cấu hình lần đầu](#5-cài-đặt-git--cấu-hình-lần-đầu)
6. [git init, git clone](#6-git-init-git-clone)
7. [git status, git add, git diff](#7-git-status-git-add-git-diff)
8. [git commit](#8-git-commit)
9. [git log, git show](#9-git-log-git-show)
10. [git push, git pull, git fetch](#10-git-push-git-pull-git-fetch)
11. [Branch](#11-branch--nhánh-làm-việc)
12. [Merge vs Rebase](#12-merge-vs-rebase)
13. [Xử lý conflict](#13-xử-lý-conflict)
14. [Pull Request](#14-pull-request)
15. [Fork](#15-fork)
16. [Issues](#16-issues)
17. [.gitignore](#17-gitignore)
18. [README.md](#18-readmemd)
19. [Xác thực: PAT & SSH key](#19-xác-thực-pat--ssh-key)
20. [GitHub Actions cơ bản](#20-github-actions-cơ-bản)
21. [Workflow thực tế: fork → PR → merge](#21-workflow-thực-tế-fork--pr--merge)
22. [Lệnh "cứu nguy"](#22-lệnh-cứu-nguy-stash-reset-revert-cherry-pick)
23. [Troubleshooting](#23-troubleshooting)
24. [Best practices](#24-best-practices)
25. [Cheat sheet](#25-cheat-sheet)

---

## 1. Git Là Gì? GitHub Là Gì?

### Git là gì?

**Git** là **hệ thống quản lý phiên bản phân tán** — một công cụ cài trên máy cậu, giúp lưu "ảnh chụp" của dự án tại từng thời điểm, quay ngược thời gian, so sánh thay đổi, và thử nghiệm trên nhánh riêng. Do **Linus Torvalds** (cha đẻ Linux) tạo ra năm 2005. Điểm mấu chốt: **Git chạy offline hoàn toàn**, mọi lịch sử nằm trong thư mục ẩn `.git` ngay trên máy cậu.

### GitHub là gì?

**GitHub** là **website lưu trữ các dự án Git trên đám mây** — "Google Drive + mạng xã hội cho code". Cậu push dự án lên đó để backup, chia sẻ, cộng tác: người khác review code, gửi Pull Request, báo lỗi qua Issues. (Đối thủ tương tự: GitLab, Bitbucket.)

### Phân biệt nhanh

| | **Git** | **GitHub** |
|---|---|---|
| Là gì | Phần mềm cài trên máy | Website/dịch vụ đám mây |
| Cần mạng? | Không | Có |
| Việc chính | Quản lý phiên bản, lịch sử | Lưu trữ, chia sẻ, cộng tác |
| Không có nó thì sao | Git vẫn chạy tốt trên máy | Không dùng được dịch vụ này |

**Ví dụ đời thường:** Git như **máy ảnh** (tự chụp, tự lưu, xem lúc nào cũng được); GitHub như **Instagram** (đăng lên để chia sẻ, bạn bè xem và bình luận).

---

## 2. Tại Sao Nên Dùng Git?

**1. Cỗ máy thời gian cho code.** Thay vì `app_v1_final_FINAL_chot.py` 😅, Git nhớ giúp mọi thay đổi trong một thư mục duy nhất. Muốn quay về bản "ngày xưa còn chạy ngon"? Một lệnh. Muốn biết ai sửa dòng này, khi nào? Một giây.

**2. Làm nhóm không giẫm chân nhau.** Mỗi người làm trên nhánh riêng, xong gộp lại — Git tự ghép chỗ không xung đột, chỗ xung đột thật thì báo rõ để con người quyết.

**3. Backup + chia sẻ.** Push lên GitHub = có bản sao trên mây. Mất laptop vẫn còn code. Đồng thời đó là portfolio sống cho nhà tuyển dụng xem.

**4. Chuẩn công nghiệp.** Hầu hết tin tuyển dụng dev đều yêu cầu biết Git.

---

## 3. Tạo Tài Khoản GitHub

**Bước 1 — Đăng ký:** vào **github.com** → **Sign up** → nhập email → đặt mật khẩu mạnh → chọn **username** (tên công khai, một phần URL `github.com/ten-cua-cau` — chọn kỹ vì đổi sau sẽ hỏng link cũ) → giải puzzle chống robot → **Create account**.

**Bước 2 — Xác minh email:** mở email GitHub gửi → bấm **Verify**.

**Bước 3 — Cài đặt profile:** avatar → **Settings** → **Public profile**. Điền tên, bio 1–2 câu (VD: "Sinh viên IT, thích Python 🌱"), upload avatar. Trang cá nhân là bộ mặt với nhà tuyển dụng — chăm chút một chút.

**Bước 4 — Bật 2FA (bắt buộc):** **Settings** → **Password and authentication** → **Enable two-factor authentication**. Khuyên dùng app Authenticator (Google/Microsoft Authenticator, Authy): quét mã QR → nhập mã 6 số. **Lưu ngay recovery codes** vào chỗ an toàn — mất điện thoại + mất mã này = mất tài khoản vĩnh viễn.

> ⚠️ Từ 2021, GitHub **không cho dùng mật khẩu** để push qua dòng lệnh — phải dùng **Personal Access Token** hoặc **SSH key** (mục 19).

---

## 4. Tạo Repository Mới Trên Web

**Repository (repo)** là "thư mục dự án" trên GitHub, chứa code + toàn bộ lịch sử Git.

**Các bước:** bấm **+** (góc phải) → **New repository** (hoặc `github.com/new`) → điền:

- **Repository name:** viết thường, dùng `-` thay khoảng trắng (VD: `quan-ly-ban-hang`).
- **Description:** mô tả ngắn (nên có).
- **Public / Private:**

| | Public | Private |
|---|---|---|
| Ai xem được | Cả thế giới | Chỉ cậu + người được mời |
| Miễn phí? | ✅ | ✅ (không giới hạn) |
| Dùng khi nào | Open source, portfolio | Dự án chưa muốn công khai, code công ty |

- **Add a README file:** nên tick ✅.
- **Add .gitignore:** chọn template theo ngôn ngữ (mục 17).
- **Choose a license:** giấy phép sử dụng code. Phổ biến: **MIT** (thoải mái nhất — dùng/sửa/bán đều được, giữ dòng ghi công), **Apache 2.0** (như MIT + điều khoản bằng sáng chế), **GPL v3** (ai dùng thì dự án họ cũng phải mở mã nguồn). Không chọn = mặc định "all rights reserved", người khác xem được nhưng không có quyền dùng.

> ⚠️ Đừng bao giờ push **mật khẩu, API key, token** lên repo public — bot quét GitHub "nhặt" trong vài giây.

---

## 5. Cài Đặt Git & Cấu Hình Lần Đầu

**Cài đặt:**
- **Windows:** tải tại **git-scm.com/download/win**, chạy file `.exe`, cứ Next. Dùng **Git Bash** để gõ lệnh.
- **macOS:** mở Terminal gõ `git` → cài Xcode Command Line Tools; hoặc `brew install git`.
- **Linux:** `sudo apt update && sudo apt install git`

Kiểm tra: `git --version` → hiện số version là OK.

**Cấu hình bắt buộc** (Git cần biết cậu là ai để ghi vào lịch sử commit):

```bash
$ git config --global user.name "An"
$ git config --global user.email "gnoudisme@gmail.com"

$ git config --list
user.name=bạn
user.email=gnoudisme@gmail.com
```

- `--global`: áp dụng mọi repo trên máy (lưu ở `~/.gitconfig`). Bỏ `--global` = chỉ repo hiện tại.
- `user.email` **nên trùng email đăng ký GitHub** để commit hiện đúng avatar/thống kê của cậu. Muốn giấu email thật: dùng email ẩn `xxx+username@users.noreply.github.com` ở **Settings → Emails**.

Cấu hình thêm (không bắt buộc):

```bash
git config --global init.defaultBranch main   # nhánh mặc định tên "main"
git config --global color.ui auto             # output có màu cho dễ đọc
```

---

## 6. git init, git clone

Hai cách để có repo Git trên máy: tạo mới (`git init`) hoặc tải từ GitHub về (`git clone`).

### 6.1. `git init` — biến thư mục thường thành repo Git

```bash
git init [ten-thu-muc]        # không tên = thư mục hiện tại; có tên = tạo thư mục mới
git init --initial-branch=main
```

```bash
$ mkdir du-an-moi && cd du-an-moi
$ git init
Initialized empty Git repository in /home/cau/du-an-moi/.git/
```

Git tạo thư mục ẩn `.git/` — "bộ não" lưu toàn bộ lịch sử. Từ giờ mọi thay đổi trong đây đều theo dõi được. (Đừng xóa tay `.git/` trừ khi muốn hủy hết lịch sử.)

### 6.2. `git clone` — tải repo từ GitHub về

```bash
git clone <url> [ten-thu-muc-dich]
```

`clone` làm 3 việc: tải code + toàn bộ lịch sử + tự thiết lập remote tên `origin` trỏ về repo gốc. Lấy URL ở nút xanh **<> Code** trên trang repo.

**HTTPS vs SSH:**

| | HTTPS | SSH |
|---|---|---|
| URL dạng | `https://github.com/user/repo.git` | `git@github.com:user/repo.git` |
| Xác thực | **Personal Access Token** | **SSH key** (cài 1 lần, push không hỏi lại) |
| Phù hợp | Người mới, dùng nhiều máy | Máy cố định, dùng thường xuyên |

```bash
$ git clone https://github.com/octocat/Hello-World.git
Cloning into 'Hello-World'...
remote: Enumerating objects: 13, done.
Receiving objects: 100% (13/13), done.

$ git clone git@github.com:octocat/Hello-World.git ten-khac   # SSH + đặt tên thư mục khác
```

---

## 7. git status, git add, git diff

Bộ 3 dùng hàng chục lần mỗi ngày. Trước hết, hiểu **mô hình 3 vùng** của Git:

```
Working Directory --git add--> Staging Area --git commit--> Repository (.git)
  (file đang sửa)                (khay chờ)                    (lịch sử chính thức)
```

Quy trình chuẩn: **sửa file → `git add` → `git commit`**.

### 7.1. `git status` — xem trạng thái (chạy trước mọi thao tác!)

```bash
git status
git status -s     # dạng ngắn gọn
```

```bash
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:          # đã git add -> sẵn sàng commit
        modified:   app.py

Changes not staged for commit:    # đã sửa nhưng chưa add
        modified:   README.md

Untracked files:                  # file mới, Git chưa biết
        notes.txt
```

Dạng ngắn:

```bash
$ git status -s
M  app.py        # M cột đầu: đã staged
 M README.md     # M cột sau: sửa ở working dir, chưa staged
?? notes.txt     # ??: untracked
```

### 7.2. `git add` — đưa thay đổi vào staging

```bash
git add <file>...   # file cụ thể
git add .           # mọi thay đổi từ thư mục hiện tại trở xuống
git add -A          # mọi thay đổi trong TOÀN repo (kể cả file đã xóa)
git add -u          # chỉ file đã từng được theo dõi (bỏ qua file mới)
git add -p          # chọn từng phần nhỏ để add (tương tác)
```

**Ví dụ 1 — add file cụ thể:**

```bash
$ git add app.py README.md
$ git status -s
M  app.py
M  README.md
?? notes.txt        # chưa add nên vẫn ??
```

**Ví dụ 2 — `git add -p`** (sửa 2 việc trong 1 file, muốn tách 2 commit):

```bash
$ git add -p app.py
(1/2) Stage this hunk [y,n,q,a,d,s,e,?]? y   # y = đưa phần này vào staging
(2/2) Stage this hunk [y,n,q,a,d,s,e,?]? n   # n = để lại commit sau
```

Phím: `y` yes, `n` no, `s` chẻ nhỏ hơn, `q` thoát.

### 7.3. `git diff` — xem đã thay đổi gì

```bash
git diff            # working dir vs staging (chỗ CHƯA add)
git diff --staged   # staging vs commit gần nhất (chỗ sắp commit)
git diff HEAD       # tất cả thay đổi so với commit gần nhất
git diff --stat     # chỉ thống kê
```

```bash
$ git diff
diff --git a/app.py b/app.py
--- a/app.py
+++ b/app.py
@@ -1,3 +1,4 @@
 def login(username, password):
-    if password == "123456":
+    if check_password(password):
         return True
```

Dòng `-` (đỏ): bị xóa/sửa đi. Dòng `+` (xanh): mới thêm. Không dấu: giữ nguyên (ngữ cảnh).

> 💡 Thói quen tốt: trước khi commit luôn chạy `git diff --staged` đọc lại — tránh commit nhầm mật khẩu, code debug.

---

## 8. git commit

`git commit` "chụp ảnh" staging area thành một điểm trong lịch sử.

```bash
git commit -m "message"        # message viết trực tiếp
git commit                    # mở editor để viết message dài
git commit -a -m "message"    # tự add file CŨ rồi commit (file mới vẫn phải add tay)
git commit --amend             # sửa commit gần nhất
```

```bash
$ git commit -m "Sửa lỗi đăng nhập và cập nhật README"
[main a3f9c21] Sửa lỗi đăng nhập và cập nhật README
 2 files changed, 6 insertions(+), 2 deletions(-)
```

`[main a3f9c21]`: commit vào nhánh `main`, mã hash `a3f9c21`.

### Viết commit message tốt

```bash
# ❌ Dở
git commit -m "fix"   /  git commit -m "update code"   /  git commit -m "asdf"

# ✅ Tốt — đọc là hiểu
git commit -m "Sửa lỗi crash khi đăng nhập bằng email trống"
git commit -m "Thêm chức năng tìm kiếm sản phẩm theo tên"
```

Quy ước **Conventional Commits** (nhiều công ty dùng):

```bash
git commit -m "feat: thêm trang giỏ hàng"
git commit -m "fix: sửa lỗi không lưu được avatar"
git commit -m "docs: cập nhật hướng dẫn API"
git commit -m "refactor: tách hàm xử lý thanh toán"
```

### `git commit --amend` — sửa commit vừa xong (chưa push)

```bash
$ git add file-quen-mat.py
$ git commit --amend --no-edit        # nhét file vào commit vừa rồi, giữ message cũ
$ git commit --amend -m "message mới" # sửa message gõ sai
```

> ⚠️ `--amend` viết lại lịch sử — chỉ dùng cho commit **chưa push**. Đã push thì dùng `git revert` (mục 22).

---

## 9. git log, git show

### 9.1. `git log` — xem lịch sử

```bash
git log                              # đầy đủ, từng commit
git log --oneline                    # gọn: 1 dòng/commit (dùng hàng ngày)
git log --oneline --graph --all      # + vẽ sơ đồ nhánh
git log -3                           # 3 commit gần nhất
git log --stat / -p                  # kèm thống kê / kèm diff chi tiết
git log --author="Tên" --since="2026-10-01" --grep="fix"   # lọc
git log --oneline -- app.py          # lịch sử của 1 file
```

```bash
$ git log --oneline
a3f9c21 (HEAD -> main, origin/main) Sửa lỗi đăng nhập và cập nhật README
7b2e4d0 Thêm trang giỏ hàng
c1a88f3 Khởi tạo dự án
```

`HEAD` = "cậu đang đứng đây". `main` = nhánh local, `origin/main` = nhánh trên GitHub.

```bash
$ git log --oneline --graph --all
*   5f1a2b3 (HEAD -> main) Merge branch 'them-gio-hang'
|\
| * 9c8d7e6 Thêm nút xóa sản phẩm
| * 2b4a6c8 Thêm trang giỏ hàng
|/
* a3f9c21 Sửa lỗi đăng nhập
```

### 9.2. `git show` — chi tiết 1 commit

```bash
git show            # commit gần nhất
git show 7b2e4d0    # commit cụ thể (vài ký tự đầu của hash là đủ)
```

```bash
$ git show 7b2e4d0
commit 7b2e4d0c1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d
Author: bạn <gnoudisme@gmail.com>
Date:   Fri Oct 9 15:30:00 2026 +0700

    Thêm trang giỏ hàng

diff --git a/cart.py b/cart.py
new file mode 100644
+class Cart:
+    def __init__(self):
+        self.items = []
```

---

## 10. git push, git pull, git fetch

**Thuật ngữ:** remote = repo trên GitHub (mặc định tên `origin`). **push** = đẩy lên, **pull** = kéo về + gộp, **fetch** = kéo về nhưng chưa gộp (xem trước).

### 10.1. `git push`

```bash
git remote add origin <url>   # liên kết repo GitHub (làm 1 lần)
git push -u origin main       # push lần đầu, -u = nhớ upstream, lần sau chỉ cần git push
git push                      # các lần sau
```

```bash
$ git push -u origin main
To https://github.com/cau/du-an-moi.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

Lần đầu push bằng HTTPS, Git hỏi username + **PAT** (không phải mật khẩu!). Dùng SSH thì không hỏi gì (mục 19).

### 10.2. `git pull`

```bash
git pull                  # kéo từ origin về nhánh hiện tại
git pull origin main
```

```bash
$ git pull
From https://github.com/cau/du-an-moi
   a3f9c21..9d2b3e4  main -> origin/main
Updating a3f9c21..9d2b3e4
Fast-forward
 cart.py | 20 ++++++++++++++++++++
```

### 10.3. `git fetch` — xem trước, chưa gộp

```bash
$ git fetch
From https://github.com/cau/du-an-moi
   a3f9c21..9d2b3e4  main -> origin/main
# code trên máy KHÔNG đổi — chỉ cập nhật cái Git "biết" về remote

$ git log --oneline main..origin/main   # remote hơn local những gì?
$ git merge origin/main                 # ưng thì gộp tay
```

| | `git pull` | `git fetch` |
|---|---|---|
| Tải tin mới từ remote | ✅ | ✅ |
| Tự gộp vào code hiện tại | ✅ (có thể conflict bất ngờ) | ❌ (cậu chủ động gộp sau) |
| Khi nào dùng | Muốn cập nhật nhanh | Muốn **xem trước** rồi mới quyết định |

`git pull` = `git fetch` + `git merge`.

> 💡 Mỗi sáng làm việc: `git pull`. Trước khi push cũng `git pull` trước để tránh bị rejected (mục 23).

---

## 11. Branch — Nhánh Làm Việc

Code như cái cây: thân chính (`main`) là bản ổn định đang chạy thật. Muốn làm tính năng mới → mọc **cành (branch)** riêng, thoải mái thử nghiệm. Xong → gộp vào thân. Cành hỏng → chặt bỏ, thân không hề hấn.

**Không dùng branch thì sao?** Đang code dở tính năng A trên `main` thì cần sửa gấp lỗi production — kẹt, vì không thể push `main` còn code dở. Có branch: A nằm yên trên nhánh riêng, về `main` sạch để fix gấp.

> 💡 Quy tắc vàng: `main` **luôn phải chạy được**. Mọi thử nghiệm trên branch riêng.

### 11.1. `git branch` — quản lý nhánh

```bash
git branch                  # xem nhánh local (* = đang đứng)
git branch -a               # xem cả nhánh remote
git branch <ten>            # tạo nhánh mới (CHƯA chuyển qua)
git branch -d <ten>         # xóa nhánh đã gộp (an toàn)
git branch -D <ten>         # xóa bắt buộc (chưa gộp — cẩn thận!)
git branch -m <cu> <moi>    # đổi tên
```

```bash
$ git branch
* main
$ git branch them-gio-hang
$ git branch
* main               # vẫn ở main! tạo ≠ chuyển
  them-gio-hang
$ git branch -d them-gio-hang
Deleted branch them-gio-hang (was 9c8d7e6).
```

Chưa gộp mà đòi xóa → Git cản: `error: The branch 'x' is not fully merged.` Chắc chắn thì dùng `-D`.

### 11.2. `git switch` — chuyển nhánh

(Git 2.23+ tách `checkout` cũ thành `switch` = chuyển nhánh, `restore` = khôi phục file.)

```bash
git switch <ten>        # chuyển sang nhánh có sẵn
git switch -c <ten>     # tạo nhánh mới VÀ chuyển luôn
git switch -           # quay lại nhánh vừa đứng
```

```bash
$ git switch -c them-gio-hang
Switched to a new branch 'them-gio-hang'

$ echo "code" > cart.py && git add cart.py
$ git commit -m "Thêm trang giỏ hàng"
[them-gio-hang 2b4a6c8] Thêm trang giỏ hàng

$ git switch main
$ ls                      # cart.py "biến mất" — nó chỉ tồn tại trên nhánh kia
app.py  README.md
$ git switch them-gio-hang
$ ls                      # lại hiện ra
app.py  cart.py  README.md
```

Git hiện đúng file của nhánh đang đứng — đó là sức mạnh của branch. Đặt tên theo quy ước: `feature/dang-nhap-google`, `fix/loi-crash`, `hotfix/loi-thanh-toan`.

### 11.3. Workflow chuẩn với branch

```bash
1. git switch main && git pull        # về main, lấy mới nhất
2. git switch -c feature/ten-tinh-nang
3. (code... commit nhiều lần...)
4. git switch main && git merge feature/ten-tinh-nang
5. git push
6. git branch -d feature/ten-tinh-nang
```

---

## 12. Merge vs Rebase

Làm xong trên nhánh riêng, muốn đưa vào `main` — có 2 cách gộp.

**Bối cảnh ví dụ:** `main` có A-B; nhánh `them-gio-hang` tách từ B, có thêm C-D.

### 12.1. `git merge` — gộp, giữ nguyên lịch sử

```bash
git switch main              # đứng ở nhánh MUỐN NHẬN code
git merge them-gio-hang
```

```bash
$ git merge them-gio-hang
Merge made by the 'ort' strategy.
 cart.py | 20 ++++++++++++++++++++

$ git log --oneline --graph
*   5f1a2b3 (HEAD -> main) Merge branch 'them-gio-hang'
|\
| * 9c8d7e6 Thêm nút xóa sản phẩm
| * 2b4a6c8 Thêm trang giỏ hàng
|/
* a3f9c21 Sửa lỗi đăng nhập
```

Git tạo **merge commit** `5f1a2b3` nối 2 dòng lịch sử. Đặc điểm: giữ nguyên sự thật (ai làm gì, nhánh nào), an toàn, nhưng graph có thể rối nếu merge nhiều.

**Biến thể:**

```bash
git merge --no-ff them-gio-hang    # ép tạo merge commit (graph rõ "đây là 1 tính năng")
git merge --ff-only them-gio-hang  # chỉ gộp nếu fast-forward được
```

**Fast-forward:** nếu `main` không có commit mới từ lúc tách nhánh, merge chỉ "tua" `main` tới D, không cần merge commit: `Updating a3f9c21..9d2b3e4 / Fast-forward`.

### 12.2. `git rebase` — dời nhánh lên đầu, lịch sử thẳng tắp

```bash
git switch them-gio-hang     # đứng ở nhánh MUỐN DỜI
git rebase main
```

Git nhấc C, D ra, đặt lại lên đầu `main` thành C', D' (nội dung giống, hash khác). Lịch sử thành đường thẳng, không merge commit:

```bash
$ git rebase main
Successfully rebased and updated refs/heads/them-gio-hang.
$ git log --oneline
d4e5f6a (them-gio-hang) Thêm nút xóa sản phẩm
c3d4e5f Thêm trang giỏ hàng
a3f9c21 Sửa lỗi đăng nhập
```

Conflict khi rebase → sửa xong `git add .` → `git rebase --continue` (hoặc `--abort` để hủy).

### 12.3. Khi nào dùng cái nào?

| | Merge | Rebase |
|---|---|---|
| Lịch sử | Giữ nguyên, có merge commit | Thẳng tắp, sạch |
| Viết lại commit cũ? | Không | Có (đổi hash) |
| Nhánh public nhiều người dùng | ✅ An toàn | ❌ Nguy hiểm |

**Quy tắc vàng:**
1. Nhánh **public/shared** (main, develop): **chỉ merge**, không bao giờ rebase.
2. Nhánh **cá nhân** (chỉ mình dùng): rebase thoải mái cho lịch sử sạch.
3. Cập nhật nhánh tính năng theo main mới: `git rebase main` (thay vì `git merge main`) → gọn hơn.

> Tóm 1 câu: **merge** = gộp 2 dòng lịch sử, giữ nguyên quá khứ. **Rebase** = viết lại quá khứ cho gọn, như thể mình làm sau cùng từ đầu.

---

## 13. Xử Lý Conflict

Conflict xảy ra khi Git không tự quyết được — thường là hai người **cùng sửa cùng một dòng** một file. Đừng sợ, đó là chuyện bình thường khi làm nhóm.

### Ví dụ thật — từng bước

File `app.py` có `PRICE = 100`. Cậu (nhánh `them-gio-hang`) sửa thành `120`; đồng đội (trên `main`, đã push) sửa thành `150`.

```bash
$ git switch main && git pull
$ git merge them-gio-hang
Auto-merging app.py
CONFLICT (content): Merge conflict in app.py
Automatic merge failed; fix conflicts and then commit the result.
```

**Bước 1 — tìm file conflict:** `git status` → mục `Unmerged paths` → `both modified: app.py`.

**Bước 2 — mở file, đọc marker:**

```python
<<<<<<< HEAD
PRICE = 150
=======
PRICE = 120
>>>>>>> them-gio-hang
```

| Marker | Ý nghĩa |
|---|---|
| `<<<<<<< HEAD` | Code của **nhánh hiện tại** (nơi cậu đứng) |
| `=======` | Vạch phân cách |
| `>>>>>>> them-gio-hang` | Code của **nhánh đang gộp vào** |

**Bước 3 — quyết định:** sửa tay thành đáp án đúng (hỏi đồng đội nếu cần), **xóa sạch** các dòng marker:

```python
PRICE = 135
```

**Bước 4 — đánh dấu xong và commit:**

```bash
$ git add app.py
$ git commit -m "Gộp nhánh them-gio-hang, chốt PRICE = 135"
[main 8f2a1c3] Gộp nhánh them-gio-hang, chốt PRICE = 135
```

Muốn bỏ cuộc: `git merge --abort`. VS Code còn hiện nút `Accept Current/Incoming/Both Change` ngay trên chỗ conflict — bấm là xong.

**Tránh conflict:** `git pull` mỗi sáng; chia việc rõ (đừng cùng sửa 1 file); commit nhỏ, merge sớm; nói chuyện với nhau trước khi động vào file người khác đang làm.

---

## 14. Pull Request

**Pull Request (PR)** là "đơn xin gộp code": làm xong trên nhánh riêng → tạo PR → mọi người review → OK thì merge. PR là trung tâm cộng tác trên GitHub.

### 14.1. Tạo PR (trên web)

```bash
$ git switch them-gio-hang
$ git push -u origin them-gio-hang
```

1. Vào repo → banner vàng **"them-gio-hang had recent pushes"** → **Compare & pull request**. (Không thấy banner: tab **Pull requests** → **New pull request**, chọn `base: main` ← `compare: them-gio-hang`.)
2. Điền **Title** ("Thêm trang giỏ hàng") + **Description** (làm gì, vì sao, test thế nào — checklist `- [x]` rất được ưa). Có thể gán Reviewers, Labels.
3. **Create pull request**.

### 14.2. Review

- Tab **Files changed**: xem diff từng file; di chuột vào dòng code → dấu **+** → comment từng dòng.
- Nút **Review changes**: **Comment** (góp ý), **Approve** ✅ (đồng ý merge), **Request changes** ❌ (chưa đạt, sửa rồi review lại).
- Tác giả push thêm commit lên nhánh → PR **tự cập nhật**, không cần tạo mới. Trả lời comment → **Resolve conversation**.

### 14.3. Merge PR — 3 kiểu

| Kiểu | Làm gì | Khi nào dùng |
|---|---|---|
| **Create a merge commit** | Gộp + giữ nguyên từng commit | Muốn giữ đầy đủ lịch sử từng bước |
| **Squash and merge** | Nén mọi commit thành **1 commit** | PR nhiều commit linh tinh ("sửa", "sửa tiếp" 😅) |
| **Rebase and merge** | Dời từng commit lên đầu main, không merge commit | Muốn lịch sử thẳng, giữ từng commit riêng |

Sau merge → **Delete branch** trên web, rồi dọn local:

```bash
$ git switch main && git pull
$ git branch -d them-gio-hang
```

> 💡 PR cũng có thể bị **Close** (từ chối) nếu ý tưởng không phù hợp — bình thường trong open source, đừng buồn.

---

## 15. Fork

**Fork** = sao chép repo public của người khác về tài khoản mình. Cậu tự do sửa trên bản sao, không ảnh hưởng repo gốc. Đây là cửa ngõ **đóng góp open source**: fork → sửa → gửi PR cho tác giả.

**Bước 1 — Fork (web):** vào repo người ta → **Fork** → **Create fork** → có `github.com/cau/cool-project`.

**Bước 2 — Clone + thêm remote `upstream`** (trỏ về repo GỐC — bước nhiều người quên!):

```bash
$ git clone https://github.com/cau/cool-project.git && cd cool-project
$ git remote add upstream https://github.com/someone/cool-project.git
$ git remote -v
origin    https://github.com/cau/cool-project.git (fetch/push)
upstream  https://github.com/someone/cool-project.git (fetch/push)
```

**Bước 3 — Làm việc, push lên fork của mình:**

```bash
$ git switch -c fix/loi-chinh-ta
# ... sửa code ...
$ git add . && git commit -m "fix: sửa lỗi chính tả trong README"
$ git push -u origin fix/loi-chinh-ta
```

**Bước 4 — Tạo PR về repo gốc:** vào fork của cậu → **Compare & pull request** (base = `someone/cool-project:main` ← head = `cau/cool-project:fix/loi-chinh-ta`) → tác giả review → merge. 🎉

**Sync fork** (repo gốc có code mới, bản fork cũ đi):

```bash
$ git fetch upstream
$ git switch main && git merge upstream/main
$ git push origin main
```

Hoặc trên web: vào fork → **Sync fork** → **Update branch**.

---

## 16. Issues

**Issues** là "phiếu công việc": báo lỗi, đề xuất tính năng, giao việc. Mỗi issue có số thứ tự (`#12`) để tham chiếu khắp nơi.

**Tạo issue:** tab **Issues** → **New issue** → Title + Description chi tiết (mô tả, các bước tái hiện, mong đợi, môi trường). Gán **Labels** (`bug`, `enhancement`, `good first issue`...), **Assignees**, **Milestone** nếu cần.

**Đóng issue:** bấm **Close issue** khi xong (có thể Reopen). Hay hơn — **đóng tự động qua PR**: trong mô tả PR viết `Fixes #12` → khi PR được merge, issue #12 tự đóng. (`Closes`, `Resolves` cũng được.)

Gõ `#12` ở bất kỳ đâu (comment, commit message) → GitHub tự link tới issue đó.

**Vòng làm việc chuẩn:** Issue (mô tả việc) → Branch (làm) → PR (review, ghi `Fixes #12`) → Merge (tự đóng issue).

---

## 17. .gitignore

File `.gitignore` (đặt ở thư mục gốc repo) liệt kê những gì Git nên **lờ đi**: file tạm, log, `node_modules/`, file chứa mật khẩu...

```gitignore
# Comment bắt đầu bằng #
secret-key.txt        # 1 file cụ thể
*.log                 # mọi file .log
node_modules/         # cả thư mục (dấu / cuối)
__pycache__/
**/.env               # file .env ở bất kỳ đâu
*.log
!important.log        # ngoại lệ: đừng bỏ qua file này
/config-local.json    # chỉ ở thư mục gốc
```

**Mẫu nhanh:**

```gitignore
# Python
__pycache__/  *.py[cod]  .venv/  venv/  dist/  build/  *.egg-info/
.pytest_cache/  .coverage  .env

# Node.js
node_modules/  dist/  build/  *.log  .env  .env.local  .DS_Store

# Java
*.class  target/  build/  out/  *.jar  *.war  .idea/  *.iml
```

> 💡 Khi tạo repo trên web, mục **Add .gitignore** có sẵn template chuẩn theo ngôn ngữ. Lưu ý: đổi `.gitignore` không tác dụng với file **đã commit** — phải `git rm --cached <file>` trước. Kiểm tra: `git check-ignore -v <file>`.

---

## 18. README.md

README là thứ người ta đọc đầu tiên khi vào repo. Viết bằng **Markdown**:

```markdown
# Tiêu đề
## Tiêu đề cấp 2
**đậm** *nghiêng* `code`
- gạch đầu dòng
1. danh sách số
[link](https://example.com)  ![ảnh](demo.png)
> trích dẫn
- [x] việc xong  - [ ] việc chưa xong
| Cột 1 | Cột 2 |
|-------|-------|
| A     | B     |
```python
def hello():
    print("Xin chào!")
```
```

**Cấu trúc README chuẩn:** tên dự án + badge + mô tả 1 dòng → ✨ Tính năng → 🚀 Cài đặt → 📖 Cách dùng → 🖼️ Ảnh demo → 🤝 Đóng góp → 📄 Giấy phép.

**Badge** (nhãn màu xinh): lấy ở **shields.io**:

```markdown
![License](https://img.shields.io/badge/license-MIT-green)
![Python](https://img.shields.io/badge/python-3.11-blue)
```

> 💡 **Profile README:** tạo repo tên **trùng username** (VD: `cau/cau`), thêm `README.md` → nội dung hiện ngay trên trang cá nhân GitHub của cậu như một CV động.

---

## 19. Xác Thực: PAT & SSH Key

Từ 2021, GitHub **không chấp nhận mật khẩu** khi push/pull qua dòng lệnh. Hai lựa chọn:

### 19.1. Personal Access Token (PAT) — dùng với HTTPS

**Tạo token:** **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** → **Generate new token (classic)**. Điền Note ("Laptop cá nhân"), Expiration (30–90 ngày), tick scope **`repo`** (cần để push; chỉ tick quyền tối thiểu). **Copy ngay** chuỗi `ghp_xxx...` — đóng trang là mất, phải tạo mới.

**Dùng:** khi `git push` hỏi Password → **dán PAT** (không phải mật khẩu!). Để khỏi nhập mỗi lần:

```bash
git config --global credential.helper store   # nhập 1 lần, máy nhớ
```

> ⚠️ Coi PAT như mật khẩu: không commit vào code, không khoe. Lỡ lộ → vào trang token **Delete** ngay → tạo mới.

### 19.2. SSH Key — push không hỏi gì

**Bước 1 — tạo key:**

```bash
$ ssh-keygen -t ed25519 -C "email-cua-cau@example.com"
# bấm Enter hết (dùng mặc định); passphrase để trống hoặc đặt thêm
```

**Bước 2 — copy khóa CÔNG KHAI** (`cat ~/.ssh/id_ed25519.pub`, cả dòng `ssh-ed25519 AAAA...`). Không bao giờ chia sẻ file `id_ed25519` (khóa riêng!).

**Bước 3 — thêm vào GitHub:** **Settings** → **SSH and GPG keys** → **New SSH key** → dán public key.

**Bước 4 — kiểm tra & dùng:**

```bash
$ ssh -T git@github.com
Hi cau! You've successfully authenticated...   # thành công 🎉

$ git clone git@github.com:cau/repo.git        # từ giờ dùng URL SSH
$ git remote set-url origin git@github.com:cau/repo.git   # repo cũ chuyển sang SSH
```

Lỗi `Permission denied (publickey)`? Checklist: đã thêm key lên GitHub chưa → `ssh-add ~/.ssh/id_ed25519` (key chưa vào agent) → thử lại `ssh -T git@github.com`.

### PAT vs SSH — chọn cái nào?

| | PAT (HTTPS) | SSH key |
|---|---|---|
| Cài đặt | Dễ, tạo trên web là xong | Tạo key + thêm vào GitHub |
| Mỗi lần push | Nhập 1 lần rồi máy nhớ | Không hỏi gì |
| Lỡ lộ | Thu hồi trên web 10 giây | Xóa key trên web |

Người mới → PAT. Dùng thường xuyên trên máy cố định → SSH. Dùng cả hai cũng được.

---

## 20. GitHub Actions Cơ Bản

**GitHub Actions** = tự động hóa: push code → GitHub tự chạy test/build/deploy trên máy ảo miễn phí, báo ✅/❌. Workflow định nghĩa bằng file YAML trong `.github/workflows/`.

**Ví dụ: chạy test Python mỗi khi push/PR vào main** — file `.github/workflows/test.yml`:

```yaml
name: Python Test

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4        # tải code về

      - name: Cài Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Cài thư viện
        run: pip install -r requirements.txt && pip install pytest

      - name: Chạy test
        run: pytest -v
```

Đọc file: `on:` = khi nào chạy; `jobs:` = làm gì (mỗi job 1 máy ảo `runs-on`); `steps:` = từng bước — dùng action có sẵn (`uses:`) hoặc chạy lệnh (`run:`). Push file lên → tab **Actions** xem log từng step; trên PR hiện ✅/❌ để reviewer biết test có pass không.

**Ví dụ: test Node trên nhiều version (matrix):**

```yaml
name: Node Test
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

Tài khoản miễn phí có **2,000 phút/tháng** — thoải mái cho dự án cá nhân.

---

## 21. Workflow Thực Tế: Fork → PR → Merge

Một vòng đầy đủ ngoài đời: cậu muốn sửa lỗi chính tả README của repo open source `awesome-dev/cool-app`.

```bash
# 1. Fork trên web: github.com/awesome-dev/cool-app -> Fork -> Create fork

# 2. Clone bản fork + thêm upstream về repo gốc
$ git clone https://github.com/cau/cool-app.git && cd cool-app
$ git remote add upstream https://github.com/awesome-dev/cool-app.git

# 3. Đồng bộ main mới nhất từ repo gốc
$ git fetch upstream && git switch main && git merge upstream/main

# 4. Tạo nhánh, sửa code, commit
$ git switch -c fix/sua-loi-chinh-ta
$ # ... sửa README.md: "chức nag" -> "chức năng" ...
$ git add README.md && git commit -m "fix: sửa lỗi chính tả trong README"

# 5. Push lên fork của mình
$ git push -u origin fix/sua-loi-chinh-ta

# 6. Tạo PR trên web: base = awesome-dev/cool-app:main
#    <- head = cau/cool-app:fix/sua-loi-chinh-ta -> Create pull request
# 7. Tác giả review, cậu push thêm commit nếu cần (PR tự cập nhật),
#    tác giả Approve -> Squash and merge -> Delete branch

# 8. Dọn dẹp local
$ git switch main
$ git fetch upstream && git merge upstream/main   # lấy code đã merge
$ git push origin main                            # cập nhật fork
$ git branch -d fix/sua-loi-chinh-ta
```

**Vòng đời tóm tắt:** Fork (1 lần) → Sync upstream → Branch → Commit → Push → PR → Review → Merge → Dọn nhánh.

---

## 22. Lệnh "Cứu Nguy": stash, reset, revert, cherry-pick

### 22.1. `git stash` — cất tạm việc đang dở

Đang sửa dở (chưa muốn commit) mà cần chuyển nhánh gấp? `stash` gom thay đổi cất vào "ngăn kéo", trả thư mục về sạch. Xong việc lấy ra làm tiếp.

```bash
git stash -m "mô tả"    # cất (+ mô tả cho dễ nhớ)
git stash list          # xem các ngăn đã cất
git stash pop           # lấy ngăn mới nhất ra (xóa khỏi list)
git stash apply         # lấy ra nhưng GIỮ trong list
git stash drop / clear  # xóa 1 ngăn / xóa hết
```

```bash
$ git status -s
 M app.py
 M README.md
$ git stash -m "đang sửa giỏ hàng dở"
Saved working directory and index state On main: đang sửa giỏ hàng dở
$ git status -s          # sạch trơn!
# ... đi fix gấp xong ...
$ git stash pop
Dropped refs/stash@{0}
$ git status -s
 M app.py
 M README.md             # việc dở đã trở lại
```

> ⚠️ `stash` mặc định không cất file untracked — thêm `-u` để cất cả file mới.

### 22.2. `git reset` — quay ngược (hiểu kỹ trước khi dùng!)

`reset` lùi HEAD về commit cũ hơn. 3 chế độ (giả sử muốn từ C về B):

| | `--soft` | `--mixed` (mặc định) | `--hard` |
|---|---|---|---|
| HEAD | về B | về B | về B |
| Staging | **giữ** (thay đổi của C) | về B | về B |
| Working dir | **giữ** | **giữ** | về B (**mất hết!**) |

```bash
git reset --soft HEAD~1    # bỏ commit cuối, thay đổi nằm trong staging (commit lại khi muốn)
git reset --mixed HEAD~1   # bỏ commit cuối, thay đổi rớt xuống working dir
git reset --hard a1b2c3d   # XÓA SẠCH về commit a1b2c3d — không khôi phục được!
git reset <file>           # bỏ file khỏi staging, giữ nguyên nội dung sửa
```

> ⚠️ `git reset --hard` **xóa vĩnh viễn** thay đổi chưa commit — chắc chắn 100% hẵng dùng. Không reset commit **đã push** rồi push đè khi làm nhóm — dùng `git revert` thay thế.

### 22.3. `git revert` — hoàn tác AN TOÀN (cho commit đã push)

Không xóa commit cũ — tạo **commit mới** làm ngược lại. Lịch sử minh bạch, push bình thường, đồng đội không ảnh hưởng:

```bash
$ git revert c3d4e5f
[main 9f8e7d6] Revert "Thêm tính năng gửi mail (bị lỗi)"
$ git log --oneline
9f8e7d6 Revert "Thêm tính năng gửi mail (bị lỗi)"   # commit mới undo
c3d4e5f Thêm tính năng gửi mail (bị lỗi)            # commit cũ vẫn còn
```

> Quy tắc: **chưa push** → `reset`/`amend` thoải mái. **Đã push** → `revert`.

### 22.4. `git cherry-pick` — gắp 1 commit sang nhánh khác

Chỉ muốn lấy đúng commit fix lỗi từ nhánh A sang `main`, không merge cả nhánh:

```bash
$ git switch main
$ git cherry-pick f1a2b3c
[main 7a8b9c0] Fix lỗi crash khi email trống
```

Conflict → sửa như bình thường → `git cherry-pick --continue` (hoặc `--abort` để hủy).

| Lệnh | Dùng khi | Viết lại lịch sử? |
|---|---|---|
| `stash` | Cất tạm việc dở | Không |
| `reset --soft/mixed` | Bỏ commit **chưa push**, giữ thay đổi | Có (chỉ local) |
| `reset --hard` | Xóa sạch về commit cũ | Có — **mất dữ liệu** |
| `revert` | Hoàn tác commit **đã push** | Không (commit mới) |
| `cherry-pick` | Lấy 1 commit từ nhánh khác | Không |

---

## 23. Troubleshooting

### `rejected (non-fast-forward)` — push bị từ chối

```bash
$ git push
 ! [rejected]  main -> main (non-fast-forward)
```

**Nguyên nhân:** GitHub có commit mới mà máy cậu chưa có. **Fix:** `git pull` (xử lý conflict nếu có) → `git push` lại. Đừng `push --force` khi làm nhóm — sẽ xóa commit của người khác.

### `Permission denied (publickey)` — lỗi SSH

Checklist: đã thêm public key lên GitHub chưa (mục 19.2) → key đã vào agent chưa:

```bash
$ ssh-add -l                              # không có gì? -> thêm vào:
$ eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519
$ ssh -T git@github.com                   # Hi cau! là OK
```

### `Authentication failed` — lỗi HTTPS

99% là nhập **mật khẩu** thay vì **PAT**. Tạo PAT (scope `repo`, mục 19.1) → dán vào chỗ Password → `git config --global credential.helper store` để máy nhớ.

### Detached HEAD

```bash
$ git checkout a1b2c3d
You are in 'detached HEAD' state...
```

Cậu đang đứng ở 1 commit, không gắn nhánh nào — commit ở đây dễ bị mất. Chỉ xem thì `git switch main` để về. Muốn giữ thay đổi: `git switch -c nhanh-moi` tạo nhánh mới từ vị trí hiện tại.

### Lỡ commit file chứa secret

1. **Thu hồi key ngay** trên trang nhà cung cấp (quan trọng nhất — xóa khỏi Git không đủ vì lịch sử còn).
2. Gỡ khỏi Git, thêm vào `.gitignore`:

```bash
$ git rm --cached .env
$ echo ".env" >> .gitignore
$ git add .gitignore && git commit -m "Gỡ .env, thêm vào .gitignore" && git push
```

Phòng bệnh: cho `.env` vào `.gitignore` từ commit đầu tiên.

### Lỗi nhỏ khác

| Lỗi | Fix |
|---|---|
| `fatal: not a git repository` | `cd` vào đúng thư mục repo |
| `Your branch is ahead of 'origin/main'` | Không phải lỗi — `git push` đi |
| `nothing to commit, working tree clean` | Không phải lỗi — không có gì thay đổi |
| Conflict khi `git stash pop` | Xử lý như merge conflict → `git add` → `git stash drop` |

---

## 24. Best Practices

1. **Commit nhỏ, message rõ:** 1 commit = 1 việc. Đừng dồn 3 việc vào 1 commit khổng lồ.
2. **Pull trước khi push:** `git pull` mỗi sáng + trước mỗi lần push — tránh 90% lỗi rejected.
3. **Không push secret:** `.env`/key/token vào `.gitignore` từ ngày đầu.
4. **`main` luôn chạy được:** thử nghiệm trên branch riêng, merge qua PR + review + test xanh.
5. **Branch ngắn ngày:** 1–3 ngày rồi merge. Branch sống cả tháng = bom nổ chậm (conflict to).
6. **Đọc lại trước khi commit:** `git status -s` + `git diff --staged` — 30 giây cứu nhiều pha quê.
7. **Dọn nhánh đã xong:** `git branch -d` local + `git push origin --delete` remote.
8. **Viết README tử tế:** repo không README như món ăn không tên.

---

## 25. Cheat Sheet

```bash
# Cấu hình
git config --global user.name "Tên Cậu"
git config --global user.email "email@example.com"
git config --global init.defaultBranch main
git config --list

# Bắt đầu
git init
git clone <url> [ten-thu-muc]

# Hàng ngày
git status -s
git diff                  # chưa add
git diff --staged         # đã add (đọc trước khi commit)
git add <file> | git add . | git add -p
git commit -m "message"
git commit -a -m "message"    # add file cũ + commit luôn
git commit --amend            # sửa commit gần nhất (chưa push)

# Lịch sử
git log --oneline
git log --oneline --graph --all
git log -3 | git log -- <file>
git show <hash>

# Remote
git remote add origin <url> | git remote -v
git push -u origin main    # lần đầu
git push | git pull | git fetch

# Nhánh
git branch [-a] | git branch <ten> | git branch -d/-D <ten> | git branch -m <cu> <moi>
git switch -c <ten> | git switch <ten> | git switch -

# Gộp
git merge <nhanh> | git merge --no-ff <nhanh> | git merge --ff-only <nhanh>
git rebase main                       # nhánh cá nhân
git rebase --continue | git rebase --abort | git merge --abort

# Cứu nguy
git stash -m "mô tả" | git stash pop | git stash list
git reset <file>                      # bỏ khỏi staging
git reset --soft/--mixed/--hard HEAD~1
git revert <hash>                     # hoàn tác đã push (an toàn)
git cherry-pick <hash>

# Fork
git remote add upstream <url-repo-goc>
git fetch upstream && git merge upstream/main
```

---

## Lời Kết

Đi hết 25 mục là cậu đã nắm toàn bộ vòng đời Git/GitHub: `init` → commit → branch → merge → conflict → PR → fork → Actions.

**Lộ trình luyện tập:** tuần 1 tạo repo, commit/push mỗi ngày → tuần 2 tập branch + merge + tự tạo conflict rồi resolve → tuần 3 fork repo open source nhỏ, gửi PR đầu tiên 🎉 → sau đó viết README đẹp, thêm Actions chạy test.

Git là kỹ năng càng dùng càng nhuần. Đừng sợ gõ sai — với `stash`, `revert` trong tay, hầu như mọi tai nạn đều cứu được. Cứ thử, cứ sai, cứ học. Chúc cậu push commit đầu tiên thật vui! 🦫💻

*Gặp lệnh lạ: `git <lệnh> --help`, hoặc tra [docs.github.com](https://docs.github.com) và [git-scm.com/doc](https://git-scm.com/doc).*
