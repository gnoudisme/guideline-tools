# Hướng Dẫn Toàn Diện npm (Node Package Manager)

> Tài liệu tiếng Việt chi tiết về npm — công cụ quản lý thư viện không thể thiếu khi làm web/app với JavaScript.
> Cập nhật: 2026-10-10

---

## Mục Lục

1. [npm Là Gì?](#1-npm-là-gì)
2. [Cài Đặt Node.js & npm](#2-cài-đặt-nodejs--npm)
3. [File package.json — Trái Tim Dự Án](#3-file-packagejson--trái-tim-dự-án)
4. [Các Lệnh Cơ Bản](#4-các-lệnh-cơ-bản)
5. [Semantic Versioning — Hiểu Số Phiên Bản](#5-semantic-versioning--hiểu-số-phiên-bản)
6. [npx — Chạy Package Không Cần Cài](#6-npx--chạy-package-không-cần-cài)
7. [Scripts — Tự Động Hóa Tác Vụ](#7-scripts--tự-động-hóa-tác-vụ)
8. [Cài Global vs Local](#8-cài-global-vs-local)
9. [package-lock.json](#9-package-lockjson)
10. [Quản Lý Dependencies Nâng Cao](#10-quản-lý-dependencies-nâng-cao)
11. [Tạo & Xuất Bản Package](#11-tạo--xuất-bản-package)
12. [File .npmrc — Cấu Hình npm](#12-file-npmrc--cấu-hình-npm)
13. [npm vs Yarn vs pnpm](#13-npm-vs-yarn-vs-pnpm)
14. [Xử Lý Lỗi Thường Gặp](#14-xử-lý-lỗi-thường-gặp)
15. [Bảo Mật](#15-bảo-mật)
16. [Best Practices](#16-best-practices)
17. [Tài Liệu Tham Khảo](#17-tài-liệu-tham-khảo)

---

## 1. npm Là Gì?

### 1.1 Định Nghĩa

**npm** (Node Package Manager) là công cụ giúp bạn:
- **Tải thư viện** JavaScript từ kho chung (npm registry) về dùng
- **Quản lý** các thư viện dự án đang dùng (phiên bản nào, cập nhật...)
- **Chia sẻ** thư viện của bạn cho cả thế giới

**npm registry** là kho thư viện JS lớn nhất thế giới — hơn **2 triệu packages**!

### 1.2 Ví Dụ Thực Tế

Muốn dùng thư viện `axios` để gọi API? Chỉ 1 lệnh:

```bash
npm install axios
```

Xong! Giờ dùng được ngay trong code:

```js
import axios from 'axios';
const { data } = await axios.get('https://api.example.com/users');
```

Không npm, bạn phải tự tải file JS, copy vào dự án, tự quản lý phiên bản — rất cực!

### 1.3 npm Đi Kèm Node.js

npm được cài **tự động** cùng Node.js. Không cần cài riêng.

```bash
node --version   # v20.x.x
npm --version    # 10.x.x
```

---

## 2. Cài Đặt Node.js & npm

### 2.1 Tải Node.js

**Windows / macOS:**
1. Vào https://nodejs.org
2. Tải bản **LTS** (ổn định nhất)
3. Cài đặt như phần mềm bình thường

**Linux (Ubuntu/Debian):**
```bash
# Cách 1: Dùng NodeSource (khuyên dùng - bản mới nhất)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Cách 2: Dùng apt (bản cũ hơn nhưng đơn giản)
sudo apt update
sudo apt install nodejs npm
```

**Dùng nvm (quản lý nhiều phiên bản Node):**
```bash
# Cài nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Cài Node 20
nvm install 20
nvm use 20

# Đặt mặc định
nvm alias default 20
```

> 💡 **Khuyên dùng nvm** nếu bạn làm nhiều dự án cần Node version khác nhau.

### 2.2 Kiểm Tra Cài Đặt

```bash
node --version
# v20.11.0

npm --version
# 10.2.4
```

### 2.3 Cập Nhật npm

```bash
# Cập nhật npm lên bản mới nhất
npm install -g npm@latest

# Kiểm tra lại
npm --version
```

---

## 3. File package.json — Trái Tim Dự Án

### 3.1 package.json Là Gì?

File `package.json` mô tả **mọi thứ** về dự án: tên, phiên bản, thư viện đang dùng, lệnh chạy...

### 3.2 Tạo package.json

```bash
# Tạo thư mục dự án
mkdir my-project && cd my-project

# Khởi tạo (hỏi từng mục)
npm init

# Khởi tạo nhanh (chấp nhận mặc định)
npm init -y
```

### 3.3 Cấu Trúc Chi Tiết

```json
{
  "name": "my-awesome-app",
  "version": "1.0.0",
  "description": "Mô tả ngắn về dự án",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "start": "node index.js",
    "dev": "vite",
    "build": "vite build",
    "test": "jest"
  },
  "keywords": ["web", "app"],
  "author": "Tên Bạn <email@example.com>",
  "license": "MIT",
  "dependencies": {
    "react": "^18.2.0",
    "axios": "~1.6.0"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "eslint": "^8.55.0"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### 3.4 Giải Thích Từng Trường

| Trường | Ý nghĩa |
|--------|---------|
| `name` | Tên package (viết thường, không dấu cách) |
| `version` | Phiên bản (theo semver: `1.0.0`) |
| `description` | Mô tả ngắn |
| `main` | File chính khi người khác `require` package này |
| `type` | `"module"` = dùng `import` / bỏ trống = dùng `require` |
| `scripts` | Các lệnh tắt (xem phần 7) |
| `dependencies` | Thư viện cần khi **chạy** app |
| `devDependencies` | Thư viện chỉ cần khi **phát triển** (test, build...) |
| `engines` | Phiên bản Node yêu cầu |
| `license` | Giấy phép (MIT, ISC...) |

### 3.5 dependencies vs devDependencies

```bash
# Thư viện chạy production (React, axios, express...)
npm install react

# Thư viện chỉ dùng khi dev (vite, eslint, jest...)
npm install --save-dev vite
# hoặc viết tắt:
npm install -D vite
```

**Tại sao phân biệt?**
- Khi deploy production: `npm install --production` chỉ cài `dependencies` → nhẹ hơn, nhanh hơn
- `devDependencies` không cần trên server chạy thật

---

## 4. Các Lệnh Cơ Bản

### 4.1 Cài Đặt Package

```bash
# Cài 1 package (mới nhất)
npm install axios

# Cài nhiều package cùng lúc
npm install react react-dom

# Cài phiên bản cụ thể
npm install axios@1.6.0

# Cài bản beta / tag đặc biệt
npm install react@beta
npm install react@next

# Viết tắt
npm i axios
```

### 4.2 Gỡ Package

```bash
# Gỡ 1 package
npm uninstall axios

# Gỡ nhiều
npm uninstall react react-dom

# Viết tắt
npm un axios
```

### 4.3 Cập Nhật Package

```bash
# Cập nhật 1 package lên bản mới nhất (trong phạm vi semver)
npm update axios

# Cập nhật tất cả
npm update

# Kiểm tra package nào đã cũ
npm outdated
```

Output của `npm outdated`:
```
Package    Current  Wanted  Latest
axios      1.5.0    1.6.7   1.6.7
react      18.2.0   18.2.0  18.3.1
```
- **Current:** Đang cài
- **Wanted:** Bản mới nhất trong phạm vi `^`/`~`
- **Latest:** Bản mới nhất trên registry

### 4.4 Xem Thông Tin Package

```bash
# Xem thông tin trên registry
npm view axios

# Xem phiên bản cụ thể
npm view axios version

# Xem tất cả phiên bản
npm view axios versions

# Xem dependencies của package
npm view react dependencies
```

### 4.5 Liệt Kê Package Đã Cài

```bash
# Cây dependencies (có thể rất dài)
npm list

# Chỉ hiện top-level (không hiện dependencies của dependencies)
npm list --depth=0

# Xem package global
npm list -g --depth=0
```

### 4.6 Tìm Kiếm Package

```bash
# Tìm trên terminal
npm search "http client"

# Hoặc tìm trên web: https://www.npmjs.com
```

### 4.7 Dọn Dẹp

```bash
# Xóa cache
npm cache clean --force

# Kiểm tra và sửa vấn đề
npm doctor

# Gỡ package không dùng
npm prune
```

---

## 5. Semantic Versioning — Hiểu Số Phiên Bản

### 5.1 Cấu Trúc: MAJOR.MINOR.PATCH

```
1.6.7
│ │ │
│ │ └── PATCH: sửa lỗi nhỏ (1.6.7 → 1.6.8) — an toàn
│ └──── MINOR: thêm tính năng mới, tương thích ngược (1.6.7 → 1.7.0) — thường an toàn
└────── MAJOR: thay đổi lớn, có thể breaking (1.6.7 → 2.0.0) — cẩn thận!
```

### 5.2 Ký Hiệu `^` và `~`

Khi cài package, npm tự thêm ký hiệu vào `package.json`:

| Ký hiệu | Ý nghĩa | Ví dụ `"axios": "^1.6.0"` |
|---------|---------|---------------------------|
| `^` (caret) | Cho phép MINOR + PATCH mới | Cài được `1.6.0` → `1.99.99` (không lên `2.0.0`) |
| `~` (tilde) | Chỉ cho phép PATCH mới | `"axios": "~1.6.0"` → chỉ `1.6.0` → `1.6.99` |
| Không ký hiệu | Phiên bản chính xác | `"axios": "1.6.0"` → đúng `1.6.0` |
| `*` / `latest` | Bản mới nhất | Không khuyên dùng (dễ vỡ) |

```json
{
  "dependencies": {
    "react": "^18.2.0",
    "axios": "~1.6.0",
    "lodash": "4.17.21"
  }
}
```

### 5.3 Khi Nào Dùng Cái Nào?

- **`^` (mặc định):** Hầu hết trường hợp — nhận tính năng mới + sửa lỗi tự động
- **`~`:** Khi muốn ổn định tuyệt đối, chỉ nhận sửa lỗi
- **Chính xác:** Khi cần đảm bảo 100% giống nhau (production quan trọng)

---

## 6. npx — Chạy Package Không Cần Cài

### 6.1 npx Là Gì?

**npx** chạy package trực tiếp từ registry **mà không cần cài đặt** vào dự án. Dùng xong là thôi!

### 6.2 Ví Dụ

```bash
# Tạo app React (không cần cài create-react-app global)
npx create-react-app my-app

# Tạo app Vite
npx create-vite my-app

# Tạo app Expo
npx create-expo-app MyApp

# Chạy TypeScript compiler 1 lần
npx tsc --init

# Kiểm tra package có lỗ hổng không
npx snyk test
```

### 6.3 npx vs npm

| | `npm` | `npx` |
|---|---|---|
| Mục đích | **Cài** package vào dự án | **Chạy** package 1 lần |
| Ví dụ | `npm install vite` | `npx vite` |
| Khi dùng | Package dùng lâu dài | Tool dùng 1 lần (scaffold, CLI...) |

### 6.4 Chạy Script Trong node_modules

```bash
# Thay vì:
./node_modules/.bin/vite

# Dùng npx (tự tìm trong node_modules trước):
npx vite
```

---

## 7. Scripts — Tự Động Hóa Tác Vụ

### 7.1 Scripts Là Gì?

Là các **lệnh tắt** định nghĩa trong `package.json`. Thay vì gõ lệnh dài, chỉ cần `npm run ten-lenh`.

### 7.2 Ví Dụ Cơ Bản

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "test": "jest",
    "lint": "eslint src/"
  }
}
```

```bash
npm run dev      # chạy "vite"
npm run build    # chạy "vite build"
npm start        # đặc biệt: không cần chữ "run"
npm test         # đặc biệt: không cần chữ "run"
```

> 💡 `npm start` và `npm test` là 2 script đặc biệt — không cần gõ `run`.

### 7.3 Scripts Nâng Cao

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",

    "prebuild": "npm run lint",
    "postbuild": "echo 'Build xong!'",
    "lint": "eslint src/ --fix",
    "format": "prettier --write src/",
    "clean": "rm -rf dist node_modules",
    "deploy": "npm run build && vercel --prod"
  }
}
```

**`pre` và `post`:** npm tự chạy `prebuild` trước và `postbuild` sau khi chạy `npm run build`!

### 7.4 Chạy Nhiều Lệnh

```bash
# Cài concurrently để chạy song song
npm install -D concurrently
```

```json
{
  "scripts": {
    "dev": "concurrently \"npm run server\" \"npm run client\"",
    "server": "node server.js",
    "client": "vite"
  }
}
```

### 7.5 Truyền Tham Số

```bash
# Truyền tham số vào script (dùng --)
npm run test -- --watch
# → chạy: jest --watch
```

---

## 8. Cài Global vs Local

### 8.1 Local (Mặc Định — Khuyên Dùng)

```bash
npm install axios
# → vào node_modules/ của dự án hiện tại
# → ghi vào package.json dependencies
```

**Ưu điểm:**
- Mỗi dự án có version riêng, không conflict
- Người khác clone về chạy `npm install` là có đủ

### 8.2 Global (Dùng `-g`)

```bash
npm install -g vercel
npm install -g typescript
npm install -g nodemon
# → vào thư mục global, dùng được ở mọi nơi
```

**Khi nào dùng global:**
- ✅ CLI tools dùng ở mọi dự án (`vercel`, `typescript`, `nodemon`, `pm2`)
- ❌ Thư viện code dùng trong dự án (`react`, `axios`) — phải cài local!

### 8.3 Xem & Gỡ Global

```bash
# Liệt kê
npm list -g --depth=0

# Gỡ
npm uninstall -g vercel

# Xem thư mục global ở đâu
npm root -g
```

### 8.4 Vấn Đề Quyền (Linux/macOS)

Nếu `npm install -g` báo lỗi `EACCES` (permission denied):

```bash
# Cách 1: Đổi thư mục global về home (khuyên dùng)
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'

# Thêm vào ~/.bashrc hoặc ~/.zshrc:
export PATH=~/.npm-global/bin:$PATH

# Load lại
source ~/.bashrc
```

```bash
# Cách 2: Dùng sudo (không khuyên dùng)
sudo npm install -g vercel
```

---

## 9. package-lock.json

### 9.1 Là Gì?

File `package-lock.json` ghi lại **chính xác** phiên bản của mọi package (kể cả dependencies của dependencies) đã cài.

```json
// package.json (linh hoạt)
"axios": "^1.6.0"

// package-lock.json (chính xác)
"axios": "1.6.7",
  "dependencies": {
    "follow-redirects": "1.15.6",
    "form-data": "4.0.0"
  }
```

### 9.2 Tại Sao Quan Trọng?

- Đảm bảo mọi người (và server) cài **đúng** phiên bản giống nhau
- Tránh lỗi "máy tôi chạy được, máy bạn không"

### 9.3 Có Nên Commit Không?

**CÓ!** Luôn commit `package-lock.json` lên GitHub (trừ khi là thư viện publish lên npm).

```bash
# Clone dự án về và cài đúng version:
git clone https://github.com/user/repo.git
cd repo
npm ci    # cài chính xác theo package-lock.json (nhanh hơn npm install)
```

### 9.4 `npm install` vs `npm ci`

| | `npm install` | `npm ci` |
|---|---|---|
| Đọc từ | `package.json` | `package-lock.json` |
| Cập nhật lock file | ✅ Có thể | ❌ Không |
| Tốc độ | Chậm hơn | Nhanh hơn |
| Dùng khi | Dev local | CI/CD, production |

---

## 10. Quản Lý Dependencies Nâng Cao

### 10.1 Peer Dependencies

Một số package yêu cầu bạn tự cài thêm package khác:

```bash
npm install eslint
# ⚠️ warning: eslint-plugin-react@"*" requires eslint as peer
npm install --save-dev eslint-plugin-react
```

### 10.2 Optional Dependencies

```bash
npm install --save-optional fsevents
```

### 10.3 Overrides (Ghi Đè Version)

Khi 2 package cần version khác nhau của cùng 1 dependency:

```json
{
  "overrides": {
    "lodash": "4.17.21"
  }
}
```
→ Mọi package đều dùng lodash 4.17.21, bất kể chúng yêu cầu gì.

### 10.4 Workspaces (Monorepo)

Quản lý nhiều package trong 1 repo:

```json
// package.json ở root
{
  "name": "my-monorepo",
  "workspaces": ["packages/*"]
}
```

```
my-monorepo/
├── package.json
├── packages/
│   ├── frontend/
│   │   └── package.json
│   └── backend/
│       └── package.json
```

```bash
# Cài tất cả workspaces
npm install

# Chạy script ở workspace cụ thể
npm run build --workspace=frontend

# Thêm dependency vào workspace cụ thể
npm install axios --workspace=backend
```

---

## 11. Tạo & Xuất Bản Package

### 11.1 Tạo Package Đơn Giản

```bash
mkdir my-utils && cd my-utils
npm init -y
```

```js
// index.js
function chaoHoi(ten) {
  return `Xin chào, ${ten}!`;
}

function tinhTong(a, b) {
  return a + b;
}

module.exports = { chaoHoi, tinhTong };
```

### 11.2 Chuẩn Bị package.json Để Publish

```json
{
  "name": "@tentaikhoan/my-utils",
  "version": "1.0.0",
  "description": "Các hàm tiện ích của tôi",
  "main": "index.js",
  "keywords": ["utils", "vietnamese"],
  "author": "Tên Bạn",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/tentaikhoan/my-utils.git"
  }
}
```

> 💡 Tên có `@tentaikhoan/` là **scoped package** — không sợ trùng tên với người khác!

### 11.3 Đăng Nhập npm

```bash
npm login
# Nhập username, password, email
# Xác thực 2FA nếu có
```

### 11.4 Xuất Bản

```bash
# Kiểm tra trước khi publish
npm publish --dry-run

# Publish thật
npm publish

# Với scoped package (mặc định private, cần --access public cho free)
npm publish --access public
```

### 11.5 Cập Nhật Phiên Bản

```bash
# Tăng version tự động
npm version patch   # 1.0.0 → 1.0.1 (sửa lỗi)
npm version minor   # 1.0.0 → 1.1.0 (thêm tính năng)
npm version major   # 1.0.0 → 2.0.0 (breaking change)

# Publish bản mới
npm publish
```

### 11.6 Gỡ Package (Hạn Chế)

```bash
# Chỉ gỡ được trong 72h sau khi publish
npm unpublish @tentaikhoan/my-utils@1.0.0
```

> ⚠️ Sau 72h không gỡ được nữa! Hãy publish version mới để sửa thay vì gỡ.

---

## 12. File .npmrc — Cấu Hình npm

### 12.1 .npmrc Là Gì?

File cấu hình npm. Có thể đặt ở:
- Dự án: `./.npmrc`
- User: `~/.npmrc`
- Global: `/usr/etc/npmrc`

### 12.2 Các Cấu Hình Thường Dùng

```ini
# Đổi registry (ví dụ dùng mirror nhanh hơn)
registry=https://registry.npmjs.org/

# Dùng registry của công ty
# registry=https://npm.mycompany.com/

# Auth token cho private registry
//npm.mycompany.com/:_authToken=abc123

# Không lưu prefix ^ khi cài (dùng version chính xác)
save-exact=true

# Tự động cài devDependencies
omit=

# Hiển thị tiến trình chi tiết
loglevel=info
```

### 12.3 Xem & Sửa Config

```bash
# Xem tất cả config
npm config list

# Xem 1 giá trị
npm config get registry

# Đặt giá trị
npm config set registry https://registry.npmjs.org/

# Xóa giá trị
npm config delete registry
```

---

## 13. npm vs Yarn vs pnpm

### 13.1 So Sánh

| Tiêu chí | npm | Yarn Classic | Yarn Berry | pnpm |
|----------|-----|--------------|------------|------|
| Tốc độ cài | Trung bình | Nhanh | Nhanh | **Nhanh nhất** |
| Dung lượng disk | Nhiều (duplicate) | Nhiều | Ít | **Ít nhất** (symlink) |
| Lock file | `package-lock.json` | `yarn.lock` | `yarn.lock` | `pnpm-lock.yaml` |
| Đi kèm Node | ✅ Có sẵn | ❌ Cài thêm | ❌ Cài thêm | ❌ Cài thêm |
| Phổ biến | **Nhất** | Phổ biến | Ít hơn | Đang tăng |

### 13.2 Khi Nào Dùng Cái Nào?

- **npm:** Mặc định, đủ tốt cho 90% dự án
- **pnpm:** Dự án lớn, muốn tiết kiệm disk và cài nhanh
- **Yarn:** Team đã quen Yarn, hoặc cần tính năng đặc biệt

### 13.3 Chuyển Đổi

```bash
# Cài pnpm
npm install -g pnpm

# Dùng pnpm thay npm (lệnh tương tự)
pnpm install
pnpm add axios
pnpm remove axios
pnpm run dev
```

---

## 14. Xử Lý Lỗi Thường Gặp

### 14.1 Lỗi Cài Đặt

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `EACCES: permission denied` | Không có quyền ghi thư mục global | Đổi prefix về `~/.npm-global` (xem 8.4) |
| `ENOTFOUND` | Không có mạng / sai registry | Kiểm tra mạng, `npm config get registry` |
| `E404: not found` | Sai tên package | Kiểm tra tên trên npmjs.com |
| `ERESOLVE unable to resolve` | Conflict version dependencies | Thử `npm install --legacy-peer-deps` |
| `EBADENGINE` | Node version không tương thích | Dùng nvm đổi Node version |

### 14.2 Lỗi Chạy

| Lỗi | Cách fix |
|-----|----------|
| `command not found: vite` | Chưa cài: `npm install` trước |
| `Cannot find module 'x'` | `rm -rf node_modules && npm install` |
| Port đã dùng | Đổi port hoặc kill process cũ |

### 14.3 Nuclear Option — Xóa Sạch Cài Lại

Khi mọi thứ đều lỗi, thử cách này:

```bash
# Xóa node_modules và lock file
rm -rf node_modules package-lock.json

# Xóa cache
npm cache clean --force

# Cài lại từ đầu
npm install
```

### 14.4 Kiểm Tra Sức Khỏe

```bash
# npm tự kiểm tra
npm doctor

# Kiểm tra lỗ hổng bảo mật
npm audit

# Tự động sửa lỗ hổng
npm audit fix
```

---

## 15. Bảo Mật

### 15.1 Kiểm Tra Lỗ Hổng

```bash
# Quét dependencies
npm audit

# Output mẫu:
# found 3 vulnerabilities (1 low, 2 high)
#   run `npm audit fix` to fix them
```

### 15.2 Không Bao Giờ

- ❌ Chạy `sudo npm install` (trừ khi biết rõ mình làm gì)
- ❌ Cài package không rõ nguồn gốc
- ❌ Commit file `.env` chứa secret
- ❌ Dùng package đã 5 năm không update cho dự án quan trọng

### 15.3 Kiểm Tra Package Trước Khi Cài

Trước khi `npm install ten-package`, kiểm tra:
1. **Lượt tải:** Trên npmjs.com — package tốt thường có hàng nghìn lượt/tuần
2. **Cập nhật gần nhất:** Không quá 1 năm
3. **GitHub:** Có repo, có issues được trả lời
4. **Dependencies:** Không quá nhiều (mỗi dep là 1 rủi ro)

```bash
# Xem thông tin nhanh
npm view ten-package time.modified
npm view ten-package maintainers
```

---

## 16. Best Practices

### 16.1 Checklist Dự Án Chuẩn

- [ ] Có `package.json` đầy đủ thông tin
- [ ] Commit `package-lock.json`
- [ ] `.gitignore` có `node_modules/`
- [ ] Phân biệt `dependencies` và `devDependencies`
- [ ] Dùng `engines` để chỉ định Node version
- [ ] Scripts rõ ràng: `dev`, `build`, `test`, `lint`
- [ ] Chạy `npm audit` định kỳ

### 16.2 .gitignore Chuẩn Cho Node.js

```
# Dependencies
node_modules/

# Build output
dist/
build/

# Logs
npm-debug.log*
*.log

# Env
.env
.env.local

# OS
.DS_Store

# IDE
.vscode/
.idea/
```

### 16.3 Tối Ưu Tốc Độ Cài

```bash
# Dùng ci trên server (nhanh hơn)
npm ci

# Bỏ qua audit khi cài (nhanh hơn chút)
npm install --no-audit

# Bỏ qua fund messages
npm install --no-fund
```

---

## 17. Tài Liệu Tham Khảo

- 🏠 npmjs.com (tìm package): https://www.npmjs.com
- 📖 npm docs: https://docs.npmjs.com
- 📦 Node.js: https://nodejs.org
- 🔄 nvm: https://github.com/nvm-sh/nvm
- 📊 Trạng thái npm: https://status.npmjs.org
- 🆚 So sánh package: https://npmtrends.com

---

