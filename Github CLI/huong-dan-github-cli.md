# Hướng dẫn GitHub CLI (`gh`) — Từ A tới Z

> Làm chủ GitHub ngay trong terminal: tạo repo, mở PR, xem issue, chạy workflow, gọi API...
> mà không cần mở trình duyệt. Càng chi tiết càng tốt — mỗi lệnh đều có **cú pháp đầy đủ**,
> **giải thích từng option quan trọng** và **ít nhất 1 ví dụ chạy được**.

## Mục lục

1. [GitHub CLI là gì? So sánh với git thường và web UI](#1-github-cli-gh-là-gì)
2. [Cài đặt trên mọi nền tảng](#2-cài-đặt)
3. [`gh auth` — Đăng nhập & quản lý xác thực](#3-gh-auth--đăng-nhập--quản-lý-xác-thực)
4. [`gh repo` — Quản lý repository](#4-gh-repo--quản-lý-repository)
5. [`gh pr` — Pull Request](#5-gh-pr--pull-request)
6. [`gh issue` — Quản lý issue](#6-gh-issue--quản-lý-issue)
7. [`gh release` — Release & assets](#7-gh-release--release--assets)
8. [`gh run` + `gh workflow` — GitHub Actions](#8-gh-run--gh-workflow--github-actions)
9. [`gh gist` — Đoạn code chia sẻ nhanh](#9-gh-gist--đoạn-code-chia-sẻ-nhanh)
10. [`gh api` — Gọi REST API của GitHub](#10-gh-api--gọi-rest-api-của-github)
11. [`gh alias` — Bí danh lệnh](#11-gh-alias--bí-danh-lệnh)
12. [`gh extension` — Mở rộng sức mạnh](#12-gh-extension--mở-rộng-sức-mạnh)
13. [`gh search` — Tìm kiếm](#13-gh-search--tìm-kiếm)
14. [`gh config` — Cấu hình](#14-gh-config--cấu-hình)
15. [`gh secret` — Secrets cho Actions](#15-gh-secret--secrets-cho-actions)
16. [`gh variable` — Variables cho Actions](#16-gh-variable--variables-cho-actions)
17. [Xử lý lỗi thường gặp](#17-xử-lý-lỗi-thường-gặp)
18. [Best practices & Cheat sheet tổng hợp](#18-best-practices--cheat-sheet-tổng-hợp)

---

## 1. GitHub CLI (`gh`) là gì?

**GitHub CLI** (gọi tắt là `gh`) là công cụ dòng lệnh chính thức của GitHub, cho phép bạn làm
gần như mọi việc trên GitHub — tạo repo, mở pull request, xem issue, trigger workflow, tải
release... — ngay trong terminal, không cần mở trình duyệt.

### So sánh: `git` thường vs `gh` vs web UI

| Tiêu chí | `git` thường | `gh` (GitHub CLI) | Web UI (github.com) |
|---|---|---|---|
| Phạm vi | Chỉ thao tác với Git (commit, push, pull, branch) | Git **+** mọi tính năng GitHub (PR, issue, release, Actions, API...) | Đầy đủ tính năng nhất |
| Tạo repo mới trên GitHub | Không làm được (phải tạo tay trên web rồi add remote) | `gh repo create` — 1 lệnh xong | Vài cú click |
| Mở Pull Request | Không làm được | `gh pr create` — ngay trong terminal | Vài cú click |
| Xem trạng thái CI | Không | `gh pr checks`, `gh run watch` | Tab Actions / Checks |
| Tốc độ với dev dùng terminal | Nhanh | **Nhanh nhất** (không rời terminal) | Chậm hơn (chuyển tab, chờ load) |
| Gọi GitHub REST API | Phải tự gắn token vào curl | `gh api` — tự dùng token đã login | Không |
| Scripting / tự động hoá | Khó với tác vụ GitHub | Rất dễ (`--json`, `-q` xuất JSON) | Không |
| Học curve | Cần biết Git | Cần biết Git + vài lệnh `gh` | Dễ nhất cho người mới |

**Kết luận nhanh:**

- `git` = quản lý **mã nguồn** (local).
- `gh` = quản lý **dự án trên GitHub** (remote: PR, issue, release, CI...).
- Hai đứa **bổ sung cho nhau**, không thay thế nhau. Một dev dùng terminal hiệu quả thường
  gõ `git` cho commit/push và `gh` cho mọi thứ còn lại.

**Ví dụ workflow điển hình khi đã có `gh`:**

```bash
git checkout -b feature/dark-mode      # git: tạo nhánh
# ... code, commit ...
git push -u origin feature/dark-mode   # git: push nhánh
gh pr create --fill                    # gh: mở PR, tiêu đề + mô tả tự lấy từ commit
gh pr checks --watch                   # gh: ngồi chờ CI chạy xong ngay trong terminal
gh pr merge --squash --delete-branch   # gh: merge xong xóa nhánh
```

Cả quy trình không cần rời terminal. Đó chính là lý do `gh` đáng học.

---

## 2. Cài đặt

Kiểm tra đã cài chưa:

```bash
gh --version
# gh version 2.74.0 (2025-...) — nếu báo "command not found" thì cài theo HD dưới
```

### Windows

**Cách 1 — winget (khuyên dùng, có sẵn trên Windows 10/11):**

```powershell
winget install --id GitHub.cli
```

**Cách 2 — Chocolatey:**

```powershell
choco install gh
```

**Cách 3 — Scoop:**

```powershell
scoop install gh
```

> Sau khi cài, **đóng terminal cũ, mở terminal mới** (hoặc `refreshenv` nếu dùng choco)
> rồi chạy `gh --version` để kiểm tra.

### macOS

```bash
brew install gh
```

Nâng cấp sau này:

```bash
brew upgrade gh
```

### Linux

**Debian / Ubuntu / Raspberry Pi OS (apt):**

```bash
# Thêm key + repo chính thức của GitHub CLI
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
  | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null

sudo apt update
sudo apt install gh
```

**Fedora / RHEL / CentOS (dnf):**

```bash
sudo dnf install dnf-command(config-manager)
sudo dnf config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo
sudo dnf install gh
```

**Arch Linux / Manjaro (pacman):**

```bash
sudo pacman -S github-cli
```

### Android / Termux

```bash
pkg update && pkg install gh
```

> Trên Termux, khi `gh auth login` chọn **paste token** (vì không mở được trình duyệt
> trên máy tính để lấy code một chạm thì vẫn được — xem mục 3).

### Kiểm tra sau cài đặt

```bash
gh --version
# Ví dụ output:
# gh version 2.74.2 (2026-01-15)
# https://github.com/cli/cli/releases/tag/v2.74.2
```

---
## 3. `gh auth` — Đăng nhập & quản lý xác thực

Mọi lệnh `gh` (trừ vài lệnh public read-only) đều cần bạn đăng nhập để lấy token.
Token được lưu an toàn trong máy (Linux/macOS: keyring hệ thống; Windows: Credential Manager).

### 3.1. `gh auth login` — Đăng nhập

**Cú pháp:**

```
gh auth login [flags]
```

**Các option quan trọng:**

| Option | Giải thích |
|---|---|
| `--hostname <host>` | Đăng nhập vào GitHub Enterprise Server (mặc định là `github.com`) |
| `--with-token` | Đọc token từ stdin thay vì mở trình duyệt (dùng cho script/CI) |
| `--git-protocol <https\|ssh>` | Chọn giao thức khi clone repo sau này |
| `--scopes <scope>` | Xin thêm quyền cho token, cách nhau bằng dấu phẩy (vd: `repo,workflow,read:org`) |
| `--web` | Ép dùng flow mở trình duyệt (mặc định) |

**Ví dụ 1 — Đăng nhập tương tác (cách phổ biến nhất):**

```bash
gh auth login
```

Luồng hỏi đáp sẽ hiện ra, bạn trả lời từng bước:

```
? What account do you want to log into?  [Use arrows to move, type to filter]
> GitHub.com
  GitHub Enterprise Server

? What is your preferred protocol for Git operations on this host?
> HTTPS
  SSH

? How would you like to authenticate GitHub CLI?
> Login with a web browser        ← chọn cái này nếu máy có trình duyệt
  Paste an authentication token

! First copy your one-time code: XXXX-XXXX
Press Enter to open github.com in your browser...
✓ Authentication complete.
- gh config set -h github.com git_protocol https
✓ Configured git protocol
✓ Logged in as gnoudisme
```

Giải thích từng bước chọn:

1. **GitHub.com vs GitHub Enterprise Server**: hầu hết mọi người chọn `GitHub.com`.
   Chỉ chọn Enterprise khi công ty bạn tự host GitHub riêng (vd: `github congty.internal`).
2. **HTTPS vs SSH**: chọn `HTTPS` cho đơn giản (không cần tạo SSH key).
   Chọn `SSH` nếu bạn đã có SSH key add vào GitHub và muốn clone/push qua SSH.
3. **Login with a web browser vs Paste token**: chọn browser cho nhanh — nó cho bạn
   1 mã code, bạn mở link, nhập code, bấm Authorize là xong. Chọn paste token khi
   đang ở máy không có trình duyệt (Termux, server remote) — lúc đó tự tạo token tại
   `github.com/settings/tokens` rồi paste vào.

**Ví dụ 2 — Đăng nhập bằng token có sẵn (cho Termux / server / CI):**

```bash
# Cách an toàn: đọc token từ file, KHÔNG paste token trực tiếp lên dòng lệnh
# (vì lịch sử shell sẽ lưu lại token)
cat ~/my-token.txt | gh auth login --with-token
# Output: ✓ Logged in as gnoudisme
```

Hoặc với biến môi trường (dùng trong CI):

```bash
echo "$GH_TOKEN" | gh auth login --with-token --hostname github.com
```

**Ví dụ 3 — Đăng nhập và xin thêm quyền `workflow` (để trigger/disable workflow):**

```bash
gh auth login --scopes "repo,workflow,read:org"
```

> Mặc định `gh auth login` đã xin quyền `repo`, `read:org`, `gist`. Thêm `workflow`
> khi bạn cần thao tác với GitHub Actions.

### 3.2. `gh auth status` — Kiểm tra trạng thái đăng nhập

**Cú pháp:**

```
gh auth status [--hostname <host>] [--show-token]
```

| Option | Giải thích |
|---|---|
| `--hostname` | Kiểm tra host cụ thể (nếu login nhiều host) |
| `--show-token` | **Hiện token ra màn hình** — chỉ dùng khi debug, cẩn thận lộ token |

**Ví dụ:**

```bash
gh auth status
# Output:
# github.com
#   ✓ Logged in to github.com account gnoudisme (keyring)
#   - Active account: true
#   - Git operations protocol: https
#   - Token: gho_*******************
#   - Token scopes: 'repo', 'read:org', 'gist', 'workflow'
```

Dòng `Token scopes` cho biết token của bạn có quyền gì — nếu thiếu quyền nào
(vd thiếu `workflow`), xem mục `gh auth refresh` bên dưới.

### 3.3. `gh auth refresh` — Làm mới / thêm quyền cho token

**Cú pháp:**

```
gh auth refresh [--hostname <host>] [--scopes <scopes>]
```

Dùng khi: token hết hạn, hoặc bạn cần thêm scope mới mà không muốn login lại từ đầu.

**Ví dụ — thêm quyền `delete_repo` để được phép xóa repo:**

```bash
gh auth refresh --scopes "repo,delete_repo,workflow"
# Mở trình duyệt xác nhận lại, xong:
# ✓ Authentication complete.
```

**Ví dụ — token hết hạn (thường gặp với Enterprise / SSO):**

```bash
gh auth refresh -h github.com
# Output: ✓ Authentication complete. (token mới đã được lưu đè)
```

### 3.4. `gh auth logout` — Đăng xuất

**Cú pháp:**

```
gh auth logout [--hostname <host>]
```

**Ví dụ:**

```bash
gh auth logout
# ? Are you sure you want to log out of account gnoudisme on github.com? Yes
# ✓ Logged out of github.com account gnoudisme
```

> Lệnh này **xóa token khỏi máy bạn** và thu hồi nó trên GitHub. Dùng khi đổi tài khoản
> hoặc dùng máy chung. Lưu ý: nó không xóa cấu hình git local của repo.

---

## 4. `gh repo` — Quản lý repository

### 4.1. `gh repo create` — Tạo repo mới

**Cú pháp:**

```
gh repo create [<name>] [flags]
```

**Các option quan trọng:**

| Option | Giải thích |
|---|---|
| `--public` / `--private` / `--internal` | Chế độ hiển thị của repo (mặc định hỏi tương tác) |
| `--source <dir>` | Tạo repo từ thư mục local có sẵn (mặc định là `.`) |
| `--remote <name>` | Tên remote git sẽ add (mặc định `origin`) |
| `--push` | Tự động push nhánh hiện tại lên sau khi tạo |
| `--description <text>` | Mô tả repo |
| `--homepage <url>` | URL trang chủ hiển thị trên repo |
| `--clone` | Clone repo về ngay sau khi tạo |
| `--template <repo>` | Tạo từ 1 template repository |
| `--include-all-branches` | (khi fork) copy toàn bộ nhánh |
| `-y` / `--confirm` | Bỏ qua mọi câu hỏi xác nhận |
| `--disable-issues` / `--disable-wiki` | Tắt Issues / Wiki ngay từ đầu |

**Ví dụ 1 — Tạo repo public từ thư mục hiện tại + push code lên luôn:**

```bash
cd ~/my-app
git init -b main
git add . && git commit -m "init"
gh repo create my-app --public --source=. --remote=origin --push
# Output:
# ✓ Created repository gnoudisme/my-app on github.com
#   https://github.com/gnoudisme/my-app
# ✓ Added remote git@github.com:gnoudisme/my-app.git
```

**Ví dụ 2 — Tạo repo private, chỉ tạo remote trống (chưa có code local):**

```bash
gh repo create my-secret-project --private --description "Dự án bí mật 🤫"
# Output:
# ✓ Created repository gnoudisme/my-secret-project on github.com
#   https://github.com/gnoudisme/my-secret-project
```

**Ví dụ 3 — Tạo repo từ template:**

```bash
gh repo create my-blog --template gnoudisme/blog-template --public --clone
# Tạo repo mới copy cấu trúc từ blog-template rồi clone về máy luôn
```

### 4.2. `gh repo clone` — Clone repo về máy

**Cú pháp:**

```
gh repo clone <owner/repo> [<directory>] [-- --depth 1]
```

| Option | Giải thích |
|---|---|
| `--` | Mọi thứ sau `--` được chuyển thẳng cho `git clone` (vd `-- --depth 1`) |

**Ví dụ 1 — Clone repo của mình:**

```bash
gh repo clone gnoudisme/my-app
# = git clone https://github.com/gnoudisme/my-app.git (tự dùng đúng protocol đã chọn)
```

**Ví dụ 2 — Clone shallow (chỉ lấy commit mới nhất, nhanh, nhẹ):**

```bash
gh repo clone gnoudisme/my-app -- --depth 1
```

**Ví dụ 3 — Clone vào thư mục tên khác:**

```bash
gh repo clone gnoudisme/my-app app-v2
```

### 4.3. `gh repo fork` — Fork repo người khác

**Cú pháp:**

```
gh repo fork <owner/repo> [flags]
```

| Option | Giải thích |
|---|---|
| `--clone` | Clone fork về máy ngay (mặc định `true` — hỏi tương tác) |
| `--remote` | Add thêm remote trỏ về repo gốc (mặc định tên `upstream`) |
| `--org <org>` | Fork vào organization thay vì tài khoản cá nhân |
| `--default-branch-only` | Chỉ fork nhánh mặc định |

**Ví dụ — Fork + clone + add remote upstream (workflow đóng góp open source chuẩn):**

```bash
gh repo fork cli/cli --clone --remote
# Output:
# ✓ Created fork gnoudisme/cli
# ✓ Added remote upstream -> https://github.com/cli/cli.git
# ✓ Cloned fork to ~/cli
```

Sau đó muốn cập nhật fork theo repo gốc:

```bash
git fetch upstream
git merge upstream/main
```

### 4.4. `gh repo view` — Xem thông tin repo

**Cú pháp:**

```
gh repo view [<owner/repo>] [flags]
```

| Option | Giải thích |
|---|---|
| `--web` / `-w` | Mở repo trên trình duyệt |
| `--branch <name>` | Xem ở nhánh cụ thể |

**Ví dụ 1 — Xem repo hiện tại (đang đứng trong thư mục repo):**

```bash
gh repo view
# Output:
# gnoudisme/my-app
# Dự án demo của tớ
# -- Homepage: https://my-app.vercel.app
# -- Stars: 12 | Forks: 3 | Issues: 2 | PRs: 1
```

**Ví dụ 2 — Xem repo người khác + mở web:**

```bash
gh repo view cli/cli --web   # mở https://github.com/cli/cli trên trình duyệt
```

### 4.5. `gh repo list` — Liệt kê repo

**Cú pháp:**

```
gh repo list [<owner>] [flags]
```

| Option | Giải thích |
|---|---|
| `--limit <n>` / `-L` | Số repo tối đa (mặc định 30) |
| `--visibility <public\|private\|internal>` | Lọc theo chế độ hiển thị |
| `--fork` / `--source` | Chỉ hiện fork / chỉ hiện repo gốc |
| `--language <lang>` | Lọc theo ngôn ngữ |
| `--no-archived` | Ẩn repo đã archive |
| `--json <fields>` | Xuất JSON với các field chỉ định (để script) |

**Ví dụ 1 — Liệt kê repo của mình:**

```bash
gh repo list
# Output (dạng bảng):
# NAME                  TYPE     UPDATED
# gnoudisme/my-app      public   2026-10-09
# gnoudisme/my-secret   private  2026-10-08
```

**Ví dụ 2 — Liệt kê repo public viết bằng Python của người khác:**

```bash
gh repo list torvalds --visibility public --language python --limit 10
```

**Ví dụ 3 — Xuất JSON để script (vd: lấy tên tất cả repo private):**

```bash
gh repo list --visibility private --json name,url -q '.[].name'
# -q là jq query: lấy field name của từng phần tử
```

### 4.6. Đổi tên repo

`gh` không có lệnh `rename` riêng — dùng `gh api` (xem mục 10) hoặc `gh repo edit`:

```bash
gh repo edit gnoudisme/old-name --name new-name
# Output: ✓ Renamed repository
```

`gh repo edit` còn sửa được nhiều thứ khác:

```bash
gh repo edit --description "Mô tả mới" \
            --homepage "https://example.com" \
            --add-topic "cli,devtools" \
            --enable-issues=false
```

### 4.7. `gh repo delete` — Xóa repo ⚠️

**Cú pháp:**

```
gh repo delete [<owner/repo>] [--yes]
```

| Option | Giải thích |
|---|---|
| `--yes` | Bỏ qua xác nhận (nguy hiểm — cân nhắc khi dùng trong script) |

**Ví dụ:**

```bash
gh repo delete gnoudisme/old-prototype
# ? Type gnoudisme/old-prototype to confirm deletion: gnoudisme/old-prototype
# ✓ Deleted repository gnoudisme/old-prototype
```

> ⚠️ **Nguy hiểm:** xóa repo là **không thể hoàn tác**. Token cần scope `delete_repo`
> (xem `gh auth refresh`). Mẹo an toàn: luôn gõ lệnh không có `--yes` để nó bắt bạn
> nhập lại tên repo xác nhận.

---

## 5. `gh pr` — Pull Request

Đây là nhóm lệnh "đáng tiền" nhất của `gh`: toàn bộ vòng đời PR gói gọn trong terminal.

### 5.1. `gh pr create` — Mở PR mới

**Cú pháp:**

```
gh pr create [flags]
```

**Các option quan trọng:**

| Option | Giải thích |
|---|---|
| `--title <t>` / `-t` | Tiêu đề PR |
| `--body <b>` / `-b` | Mô tả PR (dùng `-` để đọc từ stdin) |
| `--fill` | Tự điền title/body từ commit messages (nhanh nhất) |
| `--fill-first` | Title/body lấy từ **commit đầu tiên** của nhánh |
| `--draft` / `-d` | Mở dưới dạng **draft PR** (chưa sẵn sàng review) |
| `--base <branch>` | Nhánh đích (mặc định: nhánh mặc định của repo, thường là `main`) |
| `--head <branch>` | Nhánh nguồn (mặc định: nhánh hiện tại) |
| `--reviewer <user>` / `-r` | Chỉ định reviewer |
| `--assignee <user>` / `-a` | Chỉ định người thực hiện |
| `--label <label>` / `-l` | Gắn label |
| `--milestone <name>` / `-m` | Gắn milestone |
| `--project <title>` | Thêm vào project board |
| `--web` / `-w` | Mở form tạo PR trên web thay vì terminal |

**Ví dụ 1 — Mở PR tương tác (hỏi từng bước):**

```bash
git checkout -b feature/dark-mode
# ... code ...
git push -u origin feature/dark-mode
gh pr create
# Nó sẽ hỏi lần lượt: Title? Body? Base branch? Rồi mở PR.
```

**Ví dụ 2 — Mở PR siêu nhanh với `--fill` (khuyên dùng cho PR cá nhân):**

```bash
gh pr create --fill
# Output:
# Creating pull request for feature/dark-mode into main in gnoudisme/my-app
# ✓ Created pull request #42 (https://github.com/gnoudisme/my-app/pull/42)
```

`--fill` lấy tiêu đề PR từ commit message đầu tiên và body từ các commit còn lại —
phù hợp khi bạn commit message cẩn thận.

**Ví dụ 3 — Mở draft PR, gắn reviewer + label đầy đủ:**

```bash
gh pr create --draft \
  --title "WIP: dark mode cho trang chủ" \
  --body "Còn thiếu phần toggle, chưa merge vội nhé" \
  --reviewer lee123 \
  --label "enhancement,ui" \
  --base main
# Output: ✓ Created draft pull request #43 (...)
```

Draft PR = "tớ đang làm dở, mọi người xem chơi thôi, đừng merge". Khi xong thì:

```bash
gh pr ready 43   # chuyển draft -> ready for review
```

**Ví dụ 4 — Viết body dài bằng editor:**

```bash
gh pr create --title "Thêm dark mode" --body -
# rồi paste/dán nội dung nhiều dòng, Ctrl+D để kết thúc (đọc từ stdin)
```

### 5.2. `gh pr list` — Liệt kê PR

**Cú pháp:**

```
gh pr list [flags]
```

| Option | Giải thích |
|---|---|
| `--state <open\|closed\|merged\|all>` / `-s` | Lọc trạng thái (mặc định `open`) |
| `--limit <n>` / `-L` | Số PR tối đa (mặc định 30) |
| `--author <user>` / `-A` | Lọc theo tác giả |
| `--assignee <user>` | Lọc theo người được assign |
| `--label <label>` / `-l` | Lọc theo label |
| `--search <query>` / `-S` | Tìm kiếm full-text (cú pháp như GitHub search) |
| `--json <fields>` | Xuất JSON |

**Ví dụ 1 — Xem PR đang mở:**

```bash
gh pr list
# Output:
# Showing 2 of 2 open pull requests in gnoudisme/my-app
#
# ID   TITLE                       BRANCH              CREATED AT
# #43  WIP: dark mode cho trang chủ feature/dark-mode   about 1 hour ago
# #42  Sửa lỗi crash khi login      fix/login-crash     about 3 days ago
```

**Ví dụ 2 — Lọc PR của mình đang chờ review:**

```bash
gh pr list --author "@me" --label "needs-review"
```

**Ví dụ 3 — Tìm PR liên quan "dark mode" (cả open lẫn closed):**

```bash
gh pr list --search "dark mode" --state all
```

### 5.3. `gh pr view` — Xem chi tiết PR

**Cú pháp:**

```
gh pr view [<number>|<url>|<branch>] [flags]
```

| Option | Giải thích |
|---|---|
| `--comments` / `-c` | Hiện cả comments |
| `--web` / `-w` | Mở trên trình duyệt |
| `--json <fields>` | Xuất JSON |

**Ví dụ 1 — Xem PR của nhánh hiện tại:**

```bash
gh pr view
```

**Ví dụ 2 — Xem PR #42 kèm comments:**

```bash
gh pr view 42 --comments
# Output:
# WIP: dark mode cho trang chủ #43
# Open • gnoudisme wants to merge 3 commits into main from feature/dark-mode
# ...
# -- Comments --
# lee123: Nhìn ổn đấy, nhưng chỗ toggle hơi lệch 2px 😄
```

### 5.4. `gh pr checkout` — Checkout PR về máy để review/sửa

**Cú pháp:**

```
gh pr checkout <number>
```

**Ví dụ — Lấy code của PR #42 về máy:**

```bash
gh pr checkout 42
# Output:
# From https://github.com/gnoudisme/my-app
#  * [new ref]         refs/pull/42/head -> feature/dark-mode
# Switched to branch 'feature/dark-mode'
```

> Đây là cách nhanh nhất để review code của người khác: 1 lệnh là có code chạy thử
> ngay, không cần add remote fork lằng nhằng.

### 5.5. `gh pr diff` — Xem diff của PR

**Cú pháp:**

```
gh pr diff [<number>] [--name-only] [--patch]
```

**Ví dụ:**

```bash
gh pr diff 42
# In ra full diff ngay trong terminal (dùng pager less, bấm q để thoát)

gh pr diff 42 --name-only
# Chỉ liệt kê file nào bị đổi:
# src/App.jsx
# src/styles.css
```

### 5.6. `gh pr checks` — Xem trạng thái CI của PR

**Cú pháp:**

```
gh pr checks [<number>] [flags]
```

| Option | Giải thích |
|---|---|
| `--watch` | **Đứng chờ** cho đến khi CI chạy xong (rất tiện) |
| `--fail-fast` | Thoát ngay khi có check fail |
| `--interval <n>` | Số giây giữa mỗi lần poll (mặc định 10) |

**Ví dụ 1 — Xem nhanh CI pass hay fail:**

```bash
gh pr checks 42
# Output:
# Some checks were not successful
# 1 failing, 1 successful, 1 pending
#
# NAME          STATUS  DURATION
# lint          fail    1m12s
# test          pass    2m03s
# build         pending -
```

**Ví dụ 2 — Ngồi chờ CI chạy xong (khỏi F5 trình duyệt):**

```bash
gh pr checks 42 --watch
# ... đợi ...
# All checks were successful
# 3 successful, 0 failing
```

### 5.7. `gh pr review` — Review PR

**Cú pháp:**

```
gh pr review [<number>] --approve | --request-changes | --comment [flags]
```

| Option | Giải thích |
|---|---|
| `--approve` | ✅ Approve |
| `--request-changes` | ❌ Yêu cầu sửa |
| `--comment` | 💬 Comment thường (không approve cũng không reject) |
| `--body <text>` / `-b` | Nội dung review |
| `--require-last-push-approval` | Bắt reviewer approve lại nếu có push mới (repo rule) |

**Ví dụ 1 — Approve:**

```bash
gh pr review 42 --approve --body "LGTM! Code sạch, test đủ 👍"
# Output: ✓ Approved pull request #42
```

**Ví dụ 2 — Request changes:**

```bash
gh pr review 42 --request-changes --body "Chỗ toggle bị lệch 2px, sửa giúp tớ nhé"
```

**Ví dụ 3 — Comment không vote:**

```bash
gh pr review 42 --comment --body "Hỏi ngu tí: sao chỗ này không dùng useMemo?"
```

### 5.8. `gh pr merge` — Merge PR

**Cú pháp:**

```
gh pr merge [<number>] [flags]
```

| Option | Giải thích |
|---|---|
| `--merge` | Merge commit thường (giữ toàn bộ lịch sử commit) |
| `--squash` | **Squash**: gộp tất cả commit thành 1 commit duy nhất (sạch history) |
| `--rebase` | Rebase từng commit lên base (giữ từng commit, history tuyến tính) |
| `--delete-branch` / `-d` | Xóa nhánh head sau khi merge |
| `--auto` | Tự động merge **khi đủ điều kiện** (CI pass + đủ approve) |
| `--body <text>` | Ghi chú merge commit |

**Ví dụ 1 — Squash merge + xóa nhánh (combo phổ biến nhất):**

```bash
gh pr merge 42 --squash --delete-branch
# Output:
# ✓ Squashed and merged pull request #42
# ✓ Deleted branch feature/dark-mode (locally and remotely)
```

**Ví dụ 2 — Đặt auto-merge: CI pass là tự merge, khỏi canh:**

```bash
gh pr merge 42 --squash --auto --delete-branch
# Output: ✓ Set pull request #42 to auto-merge (squash)
```

**Khi nào dùng kiểu merge nào?**

| Kiểu | Khi nào dùng |
|---|---|
| `--squash` | PR có nhiều commit lặt vặt ("fix typo", "wip") — gộp thành 1 cho sạch |
| `--merge` | Muốn giữ nguyên lịch sử từng commit (vd: PR từ release branch) |
| `--rebase` | Team thích history tuyến tính, từng commit đều có ý nghĩa |

### 5.9. `gh pr status` — Tổng quan PR liên quan đến bạn

```bash
gh pr status
# Output chia 3 nhóm rất tiện:
#
# Relevant pull requests in gnoudisme/my-app
#
# Current branch
#   #43  WIP: dark mode cho trang chủ [feature/dark-mode]
#   ✓ Checks passing - Review required
#
# Created by you
#   #42  Sửa lỗi crash khi login [fix/login-crash]
#   × 1 check failing - Review required
#
# Requesting a code review from you
#   (trống — không ai nhờ bạn review)
```

> Chạy `gh pr status` mỗi sáng để biết mình đang có PR nào cần xử lý — nhanh hơn
> mở 3 tab GitHub.

---

## 6. `gh issue` — Quản lý issue

Issue = nơi báo bug, đề xuất tính năng, thảo luận. Cú pháp `gh issue` rất giống `gh pr`.

### 6.1. `gh issue create` — Tạo issue

**Cú pháp:**

```
gh issue create [flags]
```

| Option | Giải thích |
|---|---|
| `--title <t>` / `-t` | Tiêu đề issue |
| `--body <b>` / `-b` | Mô tả (dùng `-` để đọc từ stdin) |
| `--label <l>` / `-l` | Gắn label (vd: `bug`, `enhancement`) |
| `--assignee <user>` / `-a` | Assign cho ai |
| `--milestone <name>` / `-m` | Gắn milestone |
| `--project <title>` | Thêm vào project board |
| `--web` / `-w` | Mở form trên web |

**Ví dụ 1 — Tạo issue báo bug đầy đủ:**

```bash
gh issue create \
  --title "Crash khi bấm nút đăng nhập trên Android" \
  --body "## Các bước tái hiện
1. Mở app
2. Bấm Đăng nhập
3. App crash

**Thiết bị:** Samsung A55, Android 15
**Bản app:** 2.3.1" \
  --label "bug,android" \
  --assignee lee123
# Output: ✓ Created issue #17 (https://github.com/gnoudisme/my-app/issues/17)
```

**Ví dụ 2 — Tạo nhanh từ template có sẵn của repo:**

```bash
gh issue create --title "Đề xuất: thêm dark mode" --label "enhancement"
# Nếu repo có nhiều issue template, nó sẽ hỏi bạn chọn template nào.
```

### 6.2. `gh issue list` — Liệt kê issue

**Cú pháp:**

```
gh issue list [flags]
```

| Option | Giải thích |
|---|---|
| `--state <open\|closed\|all>` / `-s` | Lọc trạng thái (mặc định `open`) |
| `--limit <n>` / `-L` | Số issue tối đa |
| `--label <l>` / `-l` | Lọc theo label |
| `--author <user>` / `-A` | Lọc theo người tạo |
| `--assignee <user>` | Lọc theo người được assign |
| `--milestone <name>` / `-m` | Lọc theo milestone |
| `--search <query>` / `-S` | Tìm kiếm full-text |
| `--json <fields>` | Xuất JSON |

**Ví dụ:**

```bash
gh issue list --label "bug" --state open
# Output:
# Showing 3 of 3 open issues in gnoudisme/my-app with label "bug"
#
# ID   TITLE                                          CREATED AT
# #17  Crash khi bấm nút đăng nhập trên Android        about 2 hours ago
# #15  Mất tiếng khi gọi video trên iOS               about 1 day ago
# #9   Lỗi font chữ trên màn hình cài đặt             about 5 days ago

# Issue được assign cho chính bạn:
gh issue list --assignee "@me"
```

### 6.3. `gh issue view` — Xem chi tiết issue

**Cú pháp:**

```
gh issue view <number> [flags]
```

| Option | Giải thích |
|---|---|
| `--comments` / `-c` | Hiện comments |
| `--web` / `-w` | Mở trên trình duyệt |

**Ví dụ:**

```bash
gh issue view 17 --comments
# Output:
# Crash khi bấm nút đăng nhập trên Android #17
# Open • gnoudisme opened this issue about 2 hours ago • 2 comments
# Labels: bug, android   Assignees: lee123
# ...
```

### 6.4. `gh issue close` / `gh issue reopen` — Đóng / mở lại issue

**Cú pháp:**

```
gh issue close <number> [--reason <completed|not planned>]
gh issue reopen <number>
```

| Option (close) | Giải thích |
|---|---|
| `--reason completed` | Đóng vì đã xong (mặc định) |
| `--reason "not planned"` | Đóng vì không làm (won't fix) |
| `--comment <text>` | Để lại comment khi đóng |

**Ví dụ:**

```bash
gh issue close 17 --comment "Đã fix ở PR #42, chờ release 2.3.2 nhé 🎉"
# Output: ✓ Closed issue #17

# Ôi fix chưa triệt để, mở lại:
gh issue reopen 17
# Output: ✓ Reopened issue #17
```

### 6.5. `gh issue comment` — Bình luận vào issue

**Cú pháp:**

```
gh issue comment <number> [--body <text>]
```

**Ví dụ:**

```bash
gh issue comment 17 --body "Tớ reproduce được rồi, đang fix..."
# Không có --body thì nó mở editor cho bạn gõ.
```

> 💡 Mẹo: `gh pr comment` hoạt động y hệt cho PR. Và bạn có thể dùng số PR/issue
> lẫn lộn — GitHub đánh số chung một dãy nên `gh issue view 42` cũng mở được PR #42.

---

## 7. `gh release` — Release & assets

Release = đóng gói 1 phiên bản để người dùng tải về (file .zip, .apk, binary...).

### 7.1. `gh release create` — Tạo release

**Cú pháp:**

```
gh release create <tag> [<files>...] [flags]
```

| Option | Giải thích |
|---|---|
| `--title <t>` / `-t` | Tên release (mặc định = tên tag) |
| `--notes <n>` / `-n` | Ghi chú release (dùng `-` để đọc stdin) |
| `--notes-file <file>` / `-F` | Đọc release notes từ file |
| `--generate-notes` | **Tự sinh** release notes từ PR đã merge (tiện!) |
| `--draft` / `-d` | Tạo draft release (chưa publish) |
| `--prerelease` / `-p` | Đánh dấu bản prerelease (beta/RC) |
| `--target <branch>` | Tag trỏ vào branch/commit nào (mặc định nhánh mặc định) |
| `--latest` | Đánh dấu đây là bản "latest" (khi repo có nhiều dòng release) |

**Ví dụ 1 — Tạo release kèm file binary:**

```bash
# Build ra file trước, rồi:
gh release create v2.3.2 dist/my-app.apk dist/my-app.zip \
  --title "v2.3.2 — Sửa crash login" \
  --notes "## Có gì mới
- Sửa crash khi bấm đăng nhập trên Android (#17)
- Tối ưu tốc độ khởi động 20%"
# Output:
# ✓ Created release v2.3.2 (https://github.com/gnoudisme/my-app/releases/tag/v2.3.2)
```

**Ví dụ 2 — Release notes tự động từ PR (khuyên dùng):**

```bash
gh release create v2.4.0 --generate-notes
# GitHub tự liệt kê: "## What's Changed" + danh sách PR + người đóng góp.
```

**Ví dụ 3 — Draft release để up file dần:**

```bash
gh release create v3.0.0-beta --draft --prerelease --title "v3.0.0 Beta"
# Sau này up thêm file rồi bấm Publish trên web, hoặc:
gh release edit v3.0.0-beta --draft=false
```

### 7.2. `gh release list` — Liệt kê release

```bash
gh release list --limit 5
# Output:
# TITLE                  TYPE      TAG NAME   PUBLISHED
# v2.3.2 — Sửa crash     Latest    v2.3.2     about 1 day ago
# v2.3.1                 Release   v2.3.1     about 7 days ago
# v3.0.0 Beta            Pre-release v3.0.0-beta about 2 days ago
# v2.3.0                 Release   v2.3.0     about 14 days ago
```

### 7.3. `gh release view` — Xem chi tiết release

**Cú pháp:**

```
gh release view [<tag>] [flags]
```

| Option | Giải thích |
|---|---|
| `--web` / `-w` | Mở trên trình duyệt |

**Ví dụ:**

```bash
gh release view v2.3.2
# Output:
# v2.3.2 — Sửa crash login
# Latest • gnoudisme released this about 1 day ago
# ## Có gì mới ...
# ## Assets
# my-app.apk (18.2 MB)
# my-app.zip (21.7 MB)

# Không ghi tag = xem bản latest:
gh release view
```

### 7.4. `gh release upload` — Upload thêm asset vào release có sẵn

**Cú pháp:**

```
gh release upload <tag> <files>... [--clobber]
```

| Option | Giải thích |
|---|---|
| `--clobber` | Ghi đè nếu file trùng tên đã tồn tại |

**Ví dụ:**

```bash
gh release upload v2.3.2 dist/checksums.txt --clobber
# Output: ✓ Uploaded checksums.txt
```

### 7.5. `gh release download` — Tải asset về

**Cú pháp:**

```
gh release download [<tag>] [flags]
```

| Option | Giải thích |
|---|---|
| `--pattern <glob>` / `-p` | Chỉ tải file khớp pattern (vd `*.apk`) |
| `--dir <dir>` | Thư mục lưu file |
| `--clobber` | Ghi đè file đã tồn tại |

**Ví dụ 1 — Tải toàn bộ asset của bản latest:**

```bash
gh release download --dir ./downloads
```

**Ví dụ 2 — Chỉ tải file .apk của bản v2.3.2:**

```bash
gh release download v2.3.2 --pattern "*.apk" --dir ./apk
# Output: ✓ Downloaded my-app.apk
```

### 7.6. `gh release delete` — Xóa release / tag ⚠️

**Cú pháp:**

```
gh release delete <tag> [flags]
```

| Option | Giải thích |
|---|---|
| `--yes` | Bỏ qua xác nhận |
| `--cleanup-tag` | Xóa luôn cả git tag (mặc định chỉ xóa release, giữ tag) |

**Ví dụ:**

```bash
gh release delete v3.0.0-beta --cleanup-tag --yes
# Xóa release + tag v3.0.0-beta
```

### 7.7. `gh release edit` — Sửa release

```bash
gh release edit v2.3.2 --title "v2.3.2 (bản vá nóng)" --prerelease=false
gh release edit v3.0.0-beta --draft=false   # publish draft release
```

---

## 8. `gh run` + `gh workflow` — GitHub Actions

Xem và điều khiển CI/CD ngay trong terminal — khỏi F5 tab Actions.

### 8.1. `gh run list` — Liệt kê các lần chạy workflow

**Cú pháp:**

```
gh run list [flags]
```

| Option | Giải thích |
|---|---|
| `--workflow <name>` / `-w` | Chỉ hiện workflow cụ thể (tên file hoặc tên hiển thị) |
| `--branch <b>` / `-b` | Lọc theo nhánh |
| `--status <s>` | `completed`, `in_progress`, `queued`, `failure`... |
| `--limit <n>` / `-L` | Số run tối đa (mặc định 20) |
| `--json <fields>` | Xuất JSON |

**Ví dụ:**

```bash
gh run list --limit 5
# Output:
# STATUS      TITLE                    WORKFLOW   BRANCH   EVENT  ID
# ✓ success   Sửa lỗi crash khi login  CI         main     push   12345678
# × failure   WIP: dark mode           CI         feature  push   12345600
# ● in_progress Bump version           Release    main     push   12345555

# Chỉ xem run bị fail của workflow CI:
gh run list --workflow CI --status failure
```

### 8.2. `gh run view` — Xem chi tiết 1 run

**Cú pháp:**

```
gh run view [<run-id>] [flags]
```

| Option | Giải thích |
|---|---|
| `--log` | In **toàn bộ log** của run |
| `--log-failed` | Chỉ in log của các step **bị fail** (debug nhanh!) |
| `--job <job-id>` | Xem 1 job cụ thể |
| `--web` | Mở trên trình duyệt |

**Ví dụ 1 — Xem tổng quan run mới nhất:**

```bash
gh run view
# Output:
# × CI · 12345600
# Triggered via push about 10 minutes ago
#
# JOBS
# ✓ build in 1m22s (ID 111)
# × test in 2m05s (ID 222)
# - deploy skipped
```

**Ví dụ 2 — Debug: chỉ xem log của step fail (khỏi kéo log dài):**

```bash
gh run view 12345600 --log-failed
# Output (đoạn log lỗi):
# FAIL src/login.test.js
#   ● Login › crash khi bấm nút
#     TypeError: Cannot read properties of null (reading 'token')
#     ...
```

### 8.3. `gh run watch` — Theo dõi run đang chạy (live)

**Cú pháp:**

```
gh run watch [<run-id>] [--exit-status] [--interval <n>]
```

| Option | Giải thích |
|---|---|
| `--exit-status` | Thoát với **exit code ≠ 0** nếu run fail (tiện cho script/CI lồng nhau) |
| `--interval <n>` | Giây giữa mỗi lần cập nhật (mặc định 3) |

**Ví dụ:**

```bash
gh run watch 12345600 --exit-status
# Vẽ tiến trình live trong terminal:
# ✓ build · 1m22s
# ● test · running...
# ... xong:
# ✓ All jobs passed!  (exit code 0)
```

### 8.4. Các lệnh `gh run` hữu ích khác

```bash
# Chạy lại run bị fail (không cần push commit mới):
gh run rerun 12345600
# Chỉ chạy lại các job bị fail (nhanh hơn rerun toàn bộ):
gh run rerun 12345600 --failed

# Hủy run đang chạy:
gh run cancel 12345600

# Tải log của run về file:
gh run download 12345600 --dir ./logs

# Xóa run (dọn dẹp lịch sử):
gh run delete 12345600
```

### 8.5. `gh workflow list` — Liệt kê workflow

```bash
gh workflow list
# Output:
# NAME     STATE    ID
# CI       active   111111
# Release  active   222222
# Nightly  disabled 333333
```

### 8.6. `gh workflow run` — Kích hoạt workflow thủ công

**Cú pháp:**

```
gh workflow run <workflow> [flags]
```

| Option | Giải thích |
|---|---|
| `--ref <branch>` / `-r` | Chạy trên nhánh nào (mặc định nhánh mặc định) |
| `--field <key=value>` / `-f` | Truyền **input** cho workflow (`workflow_dispatch` inputs) |

**Ví dụ 1 — Chạy workflow "Release" trên nhánh main:**

```bash
gh workflow run Release --ref main
# Output: ✓ Created workflow_dispatch event for Release at main
```

**Ví dụ 2 — Workflow có input (vd: chọn môi trường deploy):**

Giả sử workflow định nghĩa:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Môi trường deploy'
        required: true
```

Thì chạy:

```bash
gh workflow run Deploy -f environment=staging --ref main
```

### 8.7. `gh workflow enable` / `disable` / `view`

```bash
# Tắt workflow Nightly (đỡ tốn phút Actions):
gh workflow disable Nightly

# Bật lại:
gh workflow enable Nightly

# Xem định nghĩa workflow:
gh workflow view CI
```

> ⚠️ `gh workflow run` / `enable` / `disable` cần token có scope **`workflow`**.
> Nếu báo lỗi 403, chạy `gh auth refresh --scopes "repo,workflow"`.

---

## 9. `gh gist` — Đoạn code chia sẻ nhanh

Gist = "pastebin của GitHub": chia sẻ nhanh 1 đoạn code, log lỗi, config... có version control.

### 9.1. `gh gist create` — Tạo gist

**Cú pháp:**

```
gh gist create [<filename>... | -] [flags]
```

| Option | Giải thích |
|---|---|
| `--public` / `-p` | Gist public (mặc định là **secret** — chỉ ai có link mới xem) |
| `--secret` | Gist secret (mặc định) |
| `--desc <text>` / `-d` | Mô tả gist |
| `--filename <name>` | Đặt tên file khi tạo từ stdin |
| `-` | Đọc nội dung từ stdin |

**Ví dụ 1 — Tạo gist từ file có sẵn:**

```bash
gh gist create fix-login.js --desc "Cách fix crash login Android"
# Output:
# - Creating gist with contents:
#   fix-login.js
# ✓ Created secret gist https://gist.github.com/gnoudisme/abc123def456
```

**Ví dụ 2 — Tạo gist public từ nhiều file:**

```bash
gh gist create --public index.html style.css app.js --desc "Landing page demo"
```

**Ví dụ 3 — Pipe output của lệnh vào gist (chia sẻ log lỗi cực nhanh):**

```bash
npm test 2>&1 | gh gist create --filename test-log.txt --desc "Log test bị fail"
# Output: ✓ Created secret gist https://gist.github.com/...
# → gửi link này cho bạn bè/mentor xem giúp, khỏi copy-paste cả đống log
```

### 9.2. `gh gist list` — Liệt kê gist

**Cú pháp:**

```
gh gist list [flags]
```

| Option | Giải thích |
|---|---|
| `--public` | Chỉ hiện public |
| `--secret` | Chỉ hiện secret |
| `--limit <n>` / `-L` | Số gist tối đa |

**Ví dụ:**

```bash
gh gist list --limit 5
# Output:
# ID                                DESCRIPTION              FILES  VISIBILITY  UPDATED
# abc123def456                      Cách fix crash login    1      secret      about 1 hour ago
```

### 9.3. `gh gist view` — Xem gist

**Cú pháp:**

```
gh gist view <id> [flags]
```

| Option | Giải thích |
|---|---|
| `--filename <name>` | Chỉ xem 1 file trong gist |
| `--raw` | In nội dung thô (không highlight, tiện để pipe) |
| `--web` | Mở trên trình duyệt |

**Ví dụ:**

```bash
gh gist view abc123def456
gh gist view abc123def456 --raw > fix-login.js   # tải nội dung về file
```

### 9.4. Các lệnh gist khác

```bash
# Clone gist về như 1 repo git (để sửa rồi push cập nhật):
gh gist clone abc123def456 my-gist

# Sửa mô tả / thêm file vào gist có sẵn:
gh gist edit abc123def456 --desc "Mô tả mới" --add new-file.js

# Xóa gist:
gh gist delete abc123def456
```

---

## 10. `gh api` — Gọi REST API của GitHub

`gh api` là "vũ khí bí mật": mọi thứ GitHub API làm được, bạn gọi được — mà **không cần
tự quản lý token** (nó dùng token từ `gh auth login`).

**Cú pháp:**

```
gh api <endpoint> [flags]
```

**Các option quan trọng:**

| Option | Giải thích |
|---|---|
| `-X` / `--method <m>` | HTTP method: `GET` (mặc định), `POST`, `PUT`, `PATCH`, `DELETE` |
| `-f` / `--field <k=v>` | Gửi field dạng **string** (tự convert số/bool nếu hợp lệ) |
| `-F` / `--raw-field <k=v>` | Gửi field dạng **string thô** (không convert — dùng cho text có số 0 ở đầu, v.v.) |
| `--input <file>` | Đọc body từ file JSON |
| `-H` / `--header <k:v>` | Thêm HTTP header |
| `--paginate` | **Tự động lật hết các trang** (API phân trang) rồi gộp kết quả |
| `-q` / `--jq <expr>` | Lọc output bằng jq expression |
| `--template <t>` | Format output bằng Go template |
| `--verbose` | Hiện chi tiết request/response (debug) |

> **Nhớ nhanh:** `-f` = field thường (thông minh), `-F` = field thô (nguyên văn).
> 99% trường hợp dùng `-f` là đủ.

### 10.1. GET — Đọc dữ liệu

**Ví dụ 1 — Xem thông tin repo (tương đương `gh repo view` nhưng ra JSON):**

```bash
gh api repos/gnoudisme/my-app
# Output (JSON rút gọn):
# {
#   "id": 123456789,
#   "name": "my-app",
#   "full_name": "gnoudisme/my-app",
#   "private": false,
#   "stargazers_count": 12,
#   ...
# }
```

**Ví dụ 2 — Chỉ lấy field cần thiết bằng `-q` (jq):**

```bash
gh api repos/gnoudisme/my-app -q '{stars: .stargazers_count, forks: .forks_count, lang: .language}'
# Output:
# {"stars":12,"forks":3,"lang":"TypeScript"}
```

**Ví dụ 3 — Lấy danh sách contributors:**

```bash
gh api repos/gnoudisme/my-app/contributors -q '.[].login'
# Output:
# gnoudisme
# lee123
```

**Ví dụ 4 — Lấy TẤT CẢ issue (kể cả nhiều trang) với `--paginate`:**

```bash
gh api repos/gnoudisme/my-app/issues --paginate -q '.[].title'
```

### 10.2. POST — Tạo mới (ví dụ thật: tạo issue qua API)

**Ví dụ 1 — Tạo issue bằng API (tương đương `gh issue create`):**

```bash
gh api repos/gnoudisme/my-app/issues -X POST \
  -f title="Bug: mất tiếng khi gọi video" \
  -f body="Mô tả chi tiết..." \
  -f labels='["bug","ios"]' \
  -f assignees='["lee123"]'
# Output: JSON của issue vừa tạo, trong đó có "number": 18
```

> 💡 Array/object thì truyền dưới dạng **chuỗi JSON**: `-f labels='["bug"]'`.

**Ví dụ 2 — Star một repo (thả ⭐ bằng terminal!):**

```bash
gh api user/starred/cli/cli -X PUT
# (không có output = thành công, HTTP 204)

# Bỏ star:
gh api user/starred/cli/cli -X DELETE
```

**Ví dụ 3 — Tạo comment vào issue:**

```bash
gh api repos/gnoudisme/my-app/issues/17/comments -X POST \
  -f body="Tớ reproduce được rồi, đang fix 🚧"
```

### 10.3. PATCH/PUT — Cập nhật

**Ví dụ — Sửa tiêu đề issue #17:**

```bash
gh api repos/gnoudisme/my-app/issues/17 -X PATCH \
  -f title="[Android] Crash khi bấm nút đăng nhập"
```

**Ví dụ — Thêm topics cho repo:**

```bash
gh api repos/gnoudisme/my-app/topics -X PUT \
  -f names='["android","kotlin","devtools"]'
```

### 10.4. DELETE — Xóa

**Ví dụ — Xóa comment:**

```bash
gh api repos/gnoudisme/my-app/issues/comments/999999 -X DELETE
```

**Ví dụ — Xóa 1 label khỏi repo:**

```bash
gh api repos/gnoudisme/my-app/labels/wontfix -X DELETE
```

### 10.5. Mẹo `gh api` nâng cao

```bash
# Đọc body lớn từ file JSON:
gh api repos/gnoudisme/my-app/issues -X POST --input new-issue.json

# Truyền preview header (khi dùng API beta):
gh api -H "Accept: application/vnd.github+json" repos/gnoudisme/my-app

# Kết hợp --paginate + -q để đếm tổng số PR từng mở:
gh api repos/gnoudisme/my-app/pulls --paginate -q 'length' --state all
# (API mặc định trả 30 item/trang; --paginate lật hết rồi mới đếm)

# Xem rate limit còn lại:
gh api rate_limit -q '.resources.core | {limit, remaining}'
# Output: {"limit":5000,"remaining":4987}
```

> 📚 Tra endpoint nào tồn tại: docs chính thức tại `docs.github.com/rest`.
> Quy tắc endpoint: bỏ phần `https://api.github.com/` đi, giữ lại path.
> Vd: `GET https://api.github.com/repos/{owner}/{repo}` → `gh api repos/{owner}/{repo}`.

---

## 11. `gh alias` — Bí danh lệnh

Alias = đặt tên ngắn cho lệnh dài bạn gõ hoài. Lưu trong file config của `gh`,
dùng được trên mọi repo.

### 11.1. `gh alias set` — Tạo alias

**Cú pháp:**

```
gh alias set <tên> "<lệnh gh đầy đủ>" [--shell]
```

| Option | Giải thích |
|---|---|
| `--shell` | Alias chạy **lệnh shell** thay vì lệnh gh (bắt đầu bằng `!`) |

**Ví dụ 1 — Các alias kinh điển (nên đặt ngay):**

```bash
gh alias set co "pr checkout"
gh alias set il "issue list"
gh alias set pl "pr list"
gh alias set pv "pr view"
gh alias set pc "pr create --fill"

# Dùng:
gh co 42        # = gh pr checkout 42
gh il           # = gh issue list
gh pc           # = gh pr create --fill
```

**Ví dụ 2 — Alias có tham số cố định:**

```bash
# Xem PR đang mở của chính mình:
gh alias set mine "pr list --author @me --state open"
gh mine

# Merge nhanh kiểu squash + xóa nhánh:
gh alias set done "pr merge --squash --delete-branch"
gh done 42
```

**Ví dụ 3 — Alias shell (mạnh nhất, chạy được nhiều lệnh):**

```bash
# Alias "sync": pull main + prune nhánh đã xóa trên remote
gh alias set --shell sync '!git fetch --prune && git pull'

# Alias "prv": xem PR hiện tại kèm comments
gh alias set --shell prv '!gh pr view --comments | less'

# Dùng:
gh sync
gh prv
```

> Alias shell **phải** bắt đầu bằng `!`. Bên trong là shell script thu nhỏ —
> bạn dùng được `$1`, `$2` cho tham số truyền vào.

### 11.2. `gh alias list` / `delete`

```bash
gh alias list
# Output:
# co:     pr checkout
# il:     issue list
# sync:   !git fetch --prune && git pull  (shell)

gh alias delete co
# Output: ✓ Deleted alias co
```

> 💡 Muốn alias "xịn" hơn (có logic, nhiều file)? Xem `gh extension` ở mục 12 —
> alias phù hợp lệnh 1 dòng, extension phù hợp công cụ hoàn chỉnh.

---

## 12. `gh extension` — Mở rộng sức mạnh

Extension = chương trình con do cộng đồng viết, gắn vào `gh` như lệnh chính thức
(vd: `gh dash` sau khi cài extension `gh-dash`).

### 12.1. `gh extension search` — Tìm extension

**Cú pháp:**

```
gh extension search [<từ khóa>]
```

**Ví dụ:**

```bash
gh extension search dashboard
# Output (rút gọn):
# NAME              REPO
# dash              dlvhdr/gh-dash        ← dashboard PR/issue trong terminal
# ...
```

### 12.2. `gh extension install` — Cài extension

**Cú pháp:**

```
gh extension install <owner/repo>
```

**Ví dụ 1 — Cài `gh-dash` (dashboard TUI đẹp nhất cho PR/issue):**

```bash
gh extension install dlvhdr/gh-dash
# Output: ✓ Installed extension dlvhdr/gh-dash

# Dùng:
gh dash
# → Mở giao diện text-based: xem PR cần review, issue được assign,
#   bấm phím tắt để checkout/merge ngay. Thoát bằng q.
```

**Ví dụ 2 — Vài extension đáng cài:**

```bash
gh extension install seachicken/gh-poi
# gh poi: xóa các nhánh local đã merge (dọn repo sạch sẽ)
# Dùng: gh poi

gh extension install k1LoW/gh-grep
# gh-grep: grep code trên toàn bộ repo GitHub (không cần clone)
# Dùng: gh grep "TODO" --owner gnoudisme

gh extension install github/gh-copilot
# Hỏi Copilot ngay trong terminal:
# Dùng: gh copilot suggest "undo last commit"
```

### 12.3. `gh extension list` / `remove` / `upgrade`

```bash
gh extension list
# Output:
# dlvhdr/gh-dash         v0.0.0  (local)

# Cập nhật tất cả extension:
gh extension upgrade --all

# Gỡ extension:
gh extension remove dlvhdr/gh-dash
```

> ⚠️ Extension là code của bên thứ ba — chỉ cài từ repo uy tín (nhiều star,
> tác giả rõ ràng). `gh` sẽ hỏi xác nhận trước khi cài.

---

## 13. `gh search` — Tìm kiếm

Tìm repo, issue, PR trên toàn GitHub bằng cú pháp search quen thuộc.

### 13.1. `gh search repos` — Tìm repository

**Cú pháp:**

```
gh search repos [<query>] [flags]
```

| Option | Giải thích |
|---|---|
| `--language <lang>` / `-l` | Lọc ngôn ngữ |
| `--stars <n>` | Lọc theo số star (vd `>1000`, `100..5000`) |
| `--limit <n>` / `-L` | Số kết quả (tối đa 1000) |
| `--sort <stars|forks|updated>` | Sắp xếp |
| `--order <asc|desc>` | Thứ tự |
| `--json <fields>` | Xuất JSON |

**Ví dụ 1 — Tìm repo làm terminal dashboard bằng Go:**

```bash
gh search repos "terminal dashboard" --language go --stars ">500" --limit 5
# Output:
# NAME              DESCRIPTION              STARS  UPDATED
# charmbracelet/... ...
```

**Ví dụ 2 — Tìm repo Việt Nam viết về chủ đề "chat":**

```bash
gh search repos chat --language typescript --sort stars | head -20
```

### 13.2. `gh search issues` — Tìm issue

**Cú pháp:**

```
gh search issues [<query>] [flags]
```

| Option | Giải thích |
|---|---|
| `--state <open\|closed>` | Trạng thái |
| `--label <l>` | Lọc label |
| `--author <user>` | Lọc tác giả |
| `--repo <owner/repo>` | Giới hạn trong 1 repo |

**Ví dụ — Tìm issue mở label "good first issue" trong repo cli/cli (tìm việc đóng góp!):**

```bash
gh search issues "good first issue" --repo cli/cli --state open --label "good first issue"
```

### 13.3. `gh search prs` — Tìm pull request

**Cú pháp:** giống `gh search issues`, thêm `--merged`, `--draft`.

**Ví dụ — Tìm PR đã merge liên quan "dark mode" trong repo của mình:**

```bash
gh search prs "dark mode" --repo gnoudisme/my-app --merged
```

**Ví dụ — Xem PR của người khác để học cách họ fix bug:**

```bash
gh search prs "fix crash" --merged --language kotlin --limit 5 --sort updated
```

> 💡 Cú pháp query trong dấu ngoặc kép tuân theo
> [GitHub search syntax](https://docs.github.com/search-github): dùng được
> `stars:>1000`, `language:python`, `created:>2024-01-01`, v.v.

---

## 14. `gh config` — Cấu hình

Quản lý file cấu hình của `gh` (`~/.config/gh/config.yml` trên Linux/macOS,
`%APPDATA%\GitHub CLI\config.yml` trên Windows).

### 14.1. `gh config set` — Đặt cấu hình

**Cú pháp:**

```
gh config set <key> <value> [--host <hostname>]
```

**Các key hay dùng:**

| Key | Giải thích | Ví dụ |
|---|---|---|
| `git_protocol` | Giao thức clone: `https` hoặc `ssh` | `gh config set git_protocol ssh` |
| `editor` | Editor mở khi soạn PR/issue | `gh config set editor "code --wait"` |
| `prompt` | `enabled` (hỏi tương tác) / `disabled` (chế độ script) | `gh config set prompt disabled` |
| `browser` | Trình duyệt dùng cho `--web` | `gh config set browser firefox` |
| `pager` | Pager xem output dài | `gh config set pager "less -R"` |

**Ví dụ 1 — Đổi sang clone bằng SSH mặc định:**

```bash
gh config set git_protocol ssh
# Từ giờ gh repo clone / gh repo fork --clone đều dùng SSH
```

**Ví dụ 2 — Dùng VS Code làm editor soạn thảo:**

```bash
gh config set editor "code --wait"
# --wait: đợi bạn đóng tab mới chạy tiếp (quan trọng!)
```

**Ví dụ 3 — Tắt prompt tương tác (dùng trong script CI):**

```bash
gh config set prompt disabled
# Mọi lệnh thiếu tham số sẽ báo lỗi thay vì hỏi — tránh script treo vì chờ nhập
```

### 14.2. `gh config get` / `list`

```bash
gh config get git_protocol
# Output: https

gh config list
# Output:
# hosts.yml
# config.yml
#   git_protocol: https
#   editor:
#   prompt: enabled
#   pager:
#   browser:
```

### 14.3. `gh config clear-cache`

```bash
gh config clear-cache
# Xóa cache API của gh (khi nghi output cũ / muốn lấy dữ liệu mới nhất)
```

---

## 15. `gh secret` — Secrets cho Actions

Secret = biến môi trường **mã hóa**, chỉ workflow mới đọc được — dùng để lưu API key,
token, mật khẩu... **Không bao giờ** hardcode secret vào code!

Phạm vi: secret có thể đặt ở 3 cấp — **repo**, **organization**, **environment**.

### 15.1. `gh secret set` — Tạo/cập nhật secret

**Cú pháp:**

```
gh secret set <tên> [flags]
```

| Option | Giải thích |
|---|---|
| `--body <value>` / `-b` | Giá trị secret (truyền trực tiếp) |
| `--env-file <file>` | Đọc từ file `.env` (nhiều secret 1 lúc) |
| `--org <org>` | Đặt ở cấp organization |
| `--env <environment>` | Đặt ở cấp environment (vd `production`) |
| `--repos <list>` | (kèm `--org`) chỉ định repo nào được dùng secret org |
| `--visibility <all\|private\|selected>` | (kèm `--org`) phạm vi hiển thị |

**Ví dụ 1 — Đặt secret tương tác (an toàn nhất — không lưu vào history):**

```bash
gh secret set API_KEY
# ? Paste your secret: ********** (gõ/paste, nó che đi)
# ✓ Set Actions secret API_KEY for gnoudisme/my-app
```

**Ví dụ 2 — Đặt từ biến môi trường (dùng trong script):**

```bash
gh secret set API_KEY --body "$PROD_API_KEY"
```

**Ví dụ 3 — Import cả file .env:**

```bash
# File .env:
# API_KEY=abc123
# DB_PASSWORD=s3cr3t
gh secret set --env-file .env
# ✓ Set 2 secrets
```

**Ví dụ 4 — Secret cấp organization (dùng chung nhiều repo):**

```bash
gh secret set NPM_TOKEN --org my-company --visibility selected --repos "my-app,my-web"
```

### 15.2. `gh secret list` — Liệt kê (chỉ thấy TÊN, không thấy giá trị!)

```bash
gh secret list
# Output:
# NAME        UPDATED
# API_KEY     about 1 hour ago
# DB_PASSWORD about 2 days ago
```

> 🔒 GitHub **không bao giờ** cho đọc lại giá trị secret — quên thì chỉ có set đè.
> Đây là thiết kế bảo mật, không phải thiếu tính năng.

### 15.3. `gh secret delete` — Xóa secret

```bash
gh secret delete DB_PASSWORD
# Output: ✓ Deleted secret DB_PASSWORD
```

### 15.4. Dùng secret trong workflow

```yaml
# .github/workflows/deploy.yml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
        env:
          API_KEY: ${{ secrets.API_KEY }}   # ← secret bạn vừa set
```

---

## 16. `gh variable` — Variables cho Actions

Variable giống secret nhưng **không mã hóa** — dùng cho giá trị không nhạy cảm:
tên môi trường, region, version... Cú pháp **y hệt** `gh secret`.

### 16.1. `gh variable set` / `list` / `delete`

**Cú pháp:**

```
gh variable set <tên> [flags]   # flags giống hệt gh secret set
gh variable list
gh variable delete <tên>
```

| Option | Giải thích |
|---|---|
| `--body <value>` / `-b` | Giá trị variable |
| `--env-file <file>` | Đọc từ file |
| `--org <org>` / `--env <environment>` | Phạm vi org / environment |

**Ví dụ:**

```bash
# Đặt variable cho repo hiện tại:
gh variable set DEPLOY_REGION --body "ap-southeast-1"
gh variable set NODE_VERSION --body "20"

gh variable list
# Output:
# NAME          VALUE            UPDATED
# DEPLOY_REGION ap-southeast-1  just now
# NODE_VERSION  20               just now

# Khác secret: variable ĐỌC ĐƯỢC giá trị (vì không nhạy cảm) ✓
```

**Dùng trong workflow:**

```yaml
steps:
  - run: echo "Deploy tới ${{ vars.DEPLOY_REGION }}"
```

**So sánh nhanh secret vs variable:**

|  | `gh secret` | `gh variable` |
|---|---|---|
| Mã hóa | ✅ Có | ❌ Không |
| Đọc lại giá trị | ❌ Không được | ✅ Được |
| Dùng cho | API key, token, password | Region, version, tên môi trường |
| Trong workflow | `${{ secrets.TEN }}` | `${{ vars.TEN }}` |

---

## 17. Xử lý lỗi thường gặp

### 17.1. Auth hết hạn / token không còn hiệu lực

**Triệu chứng:**

```
! Failed to authenticate: the token is expired or revoked
```

**Cách fix:**

```bash
# Cách 1: refresh token (giữ nguyên scopes cũ)
gh auth refresh

# Cách 2: login lại từ đầu
gh auth login
```

> Thường gặp khi: đổi mật khẩu GitHub, bật/tắt SSO, token Enterprise hết hạn,
> hoặc admin thu hồi token. `gh auth status` để kiểm tra trước.

### 17.2. Thiếu quyền (403 / "Resource not accessible")

**Triệu chứng:**

```
! HTTP 403: Resource not accessible by integration
# hoặc khi xóa repo:
! HTTP 403: Must have admin rights to Repository
```

**Nguyên nhân & fix:**

| Tình huống | Fix |
|---|---|
| Muốn `gh workflow run` mà báo 403 | Token thiếu scope `workflow` → `gh auth refresh --scopes "repo,workflow"` |
| Muốn `gh repo delete` mà báo 403 | Token thiếu scope `delete_repo` → `gh auth refresh --scopes "repo,delete_repo"` |
| Không phải collaborator của repo | Xin quyền từ owner, hoặc fork về rồi làm PR |
| Branch protection chặn merge | Đủ approve + CI pass trước, hoặc dùng `--auto` |

### 17.3. Not found (404)

**Triệu chứng:**

```
! HTTP 404: Not Found
```

**Checklist:**

```bash
# 1. Gõ sai tên repo? Kiểm tra lại:
gh repo view <owner/repo>

# 2. Repo private mà bạn chưa được cấp quyền? → xin access từ owner

# 3. Sai số PR/issue? Liệt kê lại:
gh pr list --state all --limit 5
gh issue list --state all --limit 5

# 4. Đang đứng nhầm repo local? Kiểm tra remote:
git remote -v
```

### 17.4. Lệnh hỏi tương tác làm treo script CI

**Triệu chứng:** script chạy mãi không xong, log đứng yên.

**Fix:** tắt prompt + truyền đủ tham số:

```bash
gh config set prompt disabled   # hoặc export GH_PROMPT_DISABLED=1
# và luôn truyền --yes / -y, --title, --body... đầy đủ, không để thiếu
gh pr create --fill --draft     # có --fill nên không hỏi title/body
```

### 17.5. `gh` dùng sai tài khoản (login nhiều account)

```bash
# Xem đang login những ai:
gh auth status

# Chuyển account active:
gh auth switch --hostname github.com
# ? What account do you want to switch to? → chọn

# Hoặc chỉ định user cho 1 lệnh:
gh pr list --author other-account
```

### 17.6. Output JSON bị cắt / muốn debug request

```bash
# Xem request/response thô:
gh api repos/gnoudisme/my-app --verbose

# Kiểm tra rate limit (tránh bị chặn khi gọi API nhiều):
gh api rate_limit -q '.resources.core.remaining'
# Nếu về 0 → đợi reset (field .resources.core.reset là timestamp)
```

---

## 18. Best practices & Cheat sheet tổng hợp

### Best practices (kinh nghiệm xương máu)

1. **Commit message cẩn thận → `gh pr create --fill` viết PR giúp bạn.**
   Title/body PR đẹp hay không phụ thuộc 90% vào commit message.
2. **`--draft` trước, `gh pr ready` sau.** Mở draft sớm để CI chạy + mọi người
   góp ý sớm, đừng để code xong mới mở PR.
3. **Squash merge cho PR cá nhân/feature nhỏ**, merge commit cho release branch.
   History sạch = `git log` đọc được như truyện.
4. **Secret thì `gh secret set` (gõ tay, không `--body` trên máy chung)** —
   token lọt vào shell history là toang.
5. **Script/CI thì `prompt disabled` + `--yes` đầy đủ.** Không bao giờ để script
   treo vì chờ người nhập.
6. **`--json` + `-q` là bạn thân của automation.** Mọi dashboard/report đều
   build được từ output JSON của `gh`.
7. **Đặt alias cho 5 lệnh gõ nhiều nhất** (`co`, `il`, `pl`, `pv`, `pc`) —
   tiết kiệm hàng trăm lần gõ mỗi tuần.
8. **Trước khi `gh repo delete`, `git remote -v` + đọc lại tên repo 2 lần.**
   Xóa nhầm repo là không cứu được.
9. **Dùng `gh run watch --exit-status` trong deploy script** để pipeline dừng
   đúng lúc CI fail thay vì deploy mù.
10. **Token scope tối thiểu.** Chỉ `gh auth refresh` thêm scope khi cần
    (`workflow`, `delete_repo`) — đừng xin `admin:org` cho vui.

### Cheat sheet — 1 trang tra nhanh

```bash
# ── AUTH ──────────────────────────────────────────────
gh auth login                          # đăng nhập (tương tác)
gh auth login --with-token < token.txt # đăng nhập bằng token
gh auth status                         # kiểm tra đăng nhập + scopes
gh auth refresh --scopes "repo,workflow"  # thêm quyền / làm mới token
gh auth logout                         # đăng xuất

# ── REPO ──────────────────────────────────────────────
gh repo create my-app --public --source=. --push   # tạo + push
gh repo clone owner/repo                # clone
gh repo fork owner/repo --clone --remote           # fork + clone + upstream
gh repo view [--web]                    # xem info / mở web
gh repo list --limit 10                 # liệt kê
gh repo edit --description "..." --add-topic "x"   # sửa info / đổi tên (--name)
gh repo delete owner/repo                # XÓA (cẩn thận!)

# ── PULL REQUEST ──────────────────────────────────────
gh pr create --fill                     # mở PR nhanh từ commit msg
gh pr create --draft -r user -l bug     # draft + reviewer + label
gh pr ready 42                          # draft -> ready
gh pr list --author @me                 # PR của mình
gh pr status                            # tổng quan PR liên quan đến bạn
gh pr view 42 --comments                # xem chi tiết + comments
gh pr diff 42 [--name-only]             # xem diff
gh pr checkout 42                       # lấy code PR về review
gh pr checks 42 --watch                 # xem/đợi CI
gh pr review 42 --approve -b "LGTM"     # approve / --request-changes / --comment
gh pr merge 42 --squash -d              # squash merge + xóa nhánh
gh pr merge 42 --squash --auto -d       # auto-merge khi CI pass

# ── ISSUE ─────────────────────────────────────────────
gh issue create -t "..." -b "..." -l bug -a user   # tạo issue
gh issue list --label bug               # liệt kê
gh issue view 17 --comments             # xem chi tiết
gh issue close 17 --reason "not planned"           # đóng
gh issue reopen 17                      # mở lại
gh issue comment 17 -b "..."            # bình luận

# ── RELEASE ───────────────────────────────────────────
gh release create v1.0.0 file.zip --generate-notes  # tạo release + notes tự động
gh release list                         # liệt kê
gh release view [v1.0.0]                # xem chi tiết / bản latest
gh release upload v1.0.0 file2.zip --clobber        # up thêm asset
gh release download v1.0.0 -p "*.apk" --dir ./apk   # tải asset
gh release edit v1.0.0 --draft=false    # publish draft
gh release delete v1.0.0 --cleanup-tag -y            # xóa (+ tag)

# ── ACTIONS ───────────────────────────────────────────
gh run list --workflow CI --status failure  # liệt kê run fail
gh run view --log-failed                # xem log lỗi
gh run watch --exit-status              # theo dõi live
gh run rerun 123 --failed              # chạy lại job fail
gh run cancel 123                       # hủy run
gh workflow list                        # liệt kê workflow
gh workflow run Deploy -f env=staging   # kích hoạt thủ công + input
gh workflow disable Nightly / enable Nightly

# ── GIST ──────────────────────────────────────────────
gh gist create file.js -d "mô tả"       # tạo gist secret
gh gist create --public a.js b.css       # tạo gist public
cmd 2>&1 | gh gist create --filename log.txt  # pipe output vào gist
gh gist list / view ID [--raw] / clone ID / delete ID

# ── API ───────────────────────────────────────────────
gh api repos/{owner}/{repo}             # GET
gh api repos/{o}/{r}/issues -X POST -f title="..." -f body="..."
gh api user/starred/{o}/{r} -X PUT      # star repo
gh api ... --paginate -q '.[].title'    # lật hết trang + lọc jq
gh api rate_limit -q '.resources.core.remaining'  # xem quota

# ── ALIAS / EXTENSION / SEARCH / CONFIG ──────────────
gh alias set co "pr checkout"           # tạo alias
gh alias set --shell sync '!git fetch --prune && git pull'
gh alias list / delete co
gh extension install dlvhdr/gh-dash     # cài extension
gh extension search <keyword> / list / upgrade --all / remove <ext>
gh search repos "query" -l python --stars ">1000"
gh search issues "query" --repo o/r --state open
gh search prs "query" --merged
gh config set git_protocol ssh / editor "code --wait" / prompt disabled
gh config list

# ── SECRET / VARIABLE (Actions) ──────────────────────
gh secret set API_KEY                   # nhập tương tác (an toàn)
gh secret set API_KEY -b "$VAL"         # từ biến môi trường
gh secret set --env-file .env           # import nhiều từ file
gh secret list / delete API_KEY         # (list chỉ thấy TÊN)
gh variable set REGION -b "ap-southeast-1"
gh variable list / delete REGION        # variable đọc được giá trị
# Trong workflow: ${{ secrets.API_KEY }} / ${{ vars.REGION }}
```

### Lộ trình luyện tập 7 ngày (mỗi ngày 15 phút)

| Ngày | Làm gì |
|---|---|
| 1 | Cài `gh`, `gh auth login`, `gh auth status`, tạo repo test bằng `gh repo create` |
| 2 | Clone repo, tạo nhánh, push, `gh pr create --fill`, `gh pr checks --watch` |
| 3 | `gh pr checkout` PR của bạn cùng team, `gh pr diff`, `gh pr review --approve` |
| 4 | `gh issue create/list/close`, `gh release create --generate-notes` |
| 5 | `gh run list/view/watch`, `gh workflow run` workflow của repo mình |
| 6 | `gh api` 5 endpoint, đặt 5 alias, cài `gh-dash` |
| 7 | Viết 1 script bash dùng `gh ... --json -q` để báo cáo PR open của team |

---

*Viết bởi Capybara 🦫 cho Tiểu Dươngg — 2026-10-10*
