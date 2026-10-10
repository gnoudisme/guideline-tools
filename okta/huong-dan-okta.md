# Hướng Dẫn Toàn Tập: Okta

> **Cấp độ:** Người mới bắt đầu → tích hợp đăng nhập được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật lần cuối:** 10/10/2026

---

## Mục Lục

1. [Okta là gì? Identity là gì?](#1-okta-là-gì-identity-là-gì)
2. [So sánh Okta với Auth0, Firebase Auth, Keycloak](#2-so-sánh-okta-với-auth0-firebase-auth-keycloak)
3. [Tạo tài khoản Developer miễn phí](#3-tạo-tài-khoản-developer-miễn-phí)
4. [Tổng quan Admin Console](#4-tổng-quan-admin-console)
5. [OAuth 2.0 & OpenID Connect — hiểu trong 10 phút](#5-oauth-20--openid-connect--hiểu-trong-10-phút)
6. [Tạo Application đầu tiên](#6-tạo-application-đầu-tiên)
7. [Thêm đăng nhập vào web (JavaScript)](#7-thêm-đăng-nhập-vào-web-javascript)
8. [Thêm đăng nhập vào React](#8-thêm-đăng-nhập-vào-react)
9. [Backend: xác thực token (Node.js)](#9-backend-xác-thực-token-nodejs)
10. [Backend: xác thực token (Python)](#10-backend-xác-thực-token-python)
11. [Social login (Google, Facebook)](#11-social-login-google-facebook)
12. [MFA — xác thực đa yếu tố](#12-mfa--xác-thực-đa-yếu-tố)
13. [Quản lý người dùng & Groups](#13-quản-lý-người-dùng--groups)
14. [Authorization: phân quyền theo role](#14-authorization-phân-quyền-theo-role)
15. [Okta API](#15-okta-api)
16. [Universal Directory & Lifecycle](#16-universal-directory--lifecycle)
17. [SSO cho doanh nghiệp (SAML)](#17-sso-cho-doanh-nghiệp-saml)
18. [Bảo mật nâng cao](#18-bảo-mật-nâng-cao)
19. [Pricing](#19-pricing)
20. [Troubleshooting](#20-troubleshooting)
21. [Best practices & Cheat sheet](#21-best-practices--cheat-sheet)

---

## 1. Okta Là Gì? Identity Là Gì?

### 1.1. Identity — "chứng minh thư" của thế giới số

Mỗi khi bạn bấm "Đăng nhập bằng Google", có 1 hệ thống đứng sau xác minh bạn là ai — đó là **Identity Provider (IdP)**.

**Identity & Access Management (IAM)** gồm 2 phần:
- **Authentication (xác thực):** bạn là ai? (đăng nhập, MFA)
- **Authorization (phân quyền):** bạn được làm gì? (admin/user, đọc/ghi)

### 1.2. Okta là gì?

**Okta** là nền tảng Identity-as-a-Service hàng đầu thế giới — công ty dùng Okta để:
- Nhân viên đăng nhập 1 lần vào mọi app công ty (SSO)
- Khách hàng đăng ký/đăng nhập app (Customer Identity)
- Quản lý ai được truy cập cái gì

Okta mua lại **Auth0** năm 2021 ($6.5 tỷ) — giờ là 2 sản phẩm trong 1 công ty:
- **Okta** → cho nhân viên doanh nghiệp (Workforce Identity)
- **Auth0** → cho developer tích hợp login vào app (Customer Identity)

> 💡 Guide này tập trung vào **Okta Developer** — tích hợp đăng nhập vào app của bạn.

### 1.3. Tại sao không tự code đăng nhập?

```
Tự code:                          Dùng Okta:
- Hash password đúng cách?        - Xong sẵn, chuẩn bảo mật
- Quên mật khẩu?                   - Có sẵn flow
- MFA?                             - Bật 1 click
- Social login?                    - Tích hợp sẵn Google/FB/Apple
- Chống brute force?               - Tự động
- GDPR/compliance?                 - Đạt chuẩn
→ 3 tháng code, vẫn có lỗ hổng     → 1 buổi chiều xong
```

---

## 2. So Sánh Okta Với Auth0, Firebase Auth, Keycloak

| Tiêu chí | Okta | Auth0 (của Okta) | Firebase Auth | Keycloak |
|----------|------|------------------|---------------|----------|
| Đối tượng | Doanh nghiệp lớn | Developer, startup | App mobile/web nhỏ | Tự host |
| Free tier | 100 MAU | 25,000 MAU | Không giới hạn* | Miễn phí hoàn toàn |
| Social login | ✅ | ✅ | ✅ | ✅ |
| MFA | Mạnh nhất | ✅ | Cơ bản | ✅ |
| SAML SSO | ✅ chuẩn doanh nghiệp | ✅ | ❌ | ✅ |
| Tự host | ❌ (cloud) | ❌ (cloud) | ❌ | ✅ |
| Độ khó tích hợp | Trung bình | Dễ nhất | Dễ | Khó |
| Giá scale lớn | Đắt | Vừa phải | Rẻ | Chỉ tốn server |

*Firebase Auth free không giới hạn số user nhưng giới hạn SMS OTP.

**Khi nào chọn Okta:**
- Công ty cần SSO cho nhân viên (SAML, SCIM)
- Cần compliance nghiêm ngặt (HIPAA, SOC2)

**Khi nào chọn Auth0:**
- Startup/dev muốn tích hợp login nhanh nhất
- Free 25k MAU hào phóng

**Khi nào chọn Firebase Auth:**
- App đã dùng Firebase ecosystem

**Khi nào chọn Keycloak:**
- Muốn tự host, không phụ thuộc vendor
- Có team DevOps maintain

---

## 3. Tạo Tài Khoản Developer Miễn Phí

1. Vào https://developer.okta.com/signup
2. Điền: email công việc, tên, công ty (ghi cá nhân cũng được)
3. Chọn region gần nhất (khuyên `Asia Pacific` nếu ở VN)
4. Xác nhận email
5. Bạn nhận được **Okta domain** riêng, dạng:
   ```
   https://dev-12345678.okta.com
   ```
6. Đăng nhập vào domain đó → vào **Admin Console**

> 💡 Gói developer miễn phí: **100 Monthly Active Users**, đầy đủ tính năng core — đủ để học và làm side project.

---

## 4. Tổng Quan Admin Console

Sau khi đăng nhập `dev-xxx.okta.com/admin`:

| Menu | Chức năng |
|------|-----------|
| **Directory → People** | Quản lý người dùng |
| **Directory → Groups** | Quản lý nhóm |
| **Applications** | Tạo/quản lý app tích hợp |
| **Security** | MFA, policies, network zones |
| **Workflow** | Tự động hóa lifecycle |
| **Reports** | Log đăng nhập, audit |

### 4.1. Tạo user thử nghiệm

**Directory** → **People** → **Add person**:
- First name: `Test`, Last name: `User`
- Username: `test@example.com` (dạng email)
- Password: đặt tạm `Test123!` (bỏ check "User must change password")

→ Dùng user này để test đăng nhập sau này.

---

## 5. OAuth 2.0 & OpenID Connect — Hiểu Trong 10 Phút

### 5.1. OAuth 2.0 là gì?

**OAuth 2.0** là chuẩn "ủy quyền" — cho phép app A truy cập dữ liệu ở app B **mà không cần mật khẩu** của user.

Ví dụ: bạn bấm "Đăng nhập bằng Google" vào 1 web lạ:
1. Web chuyển bạn sang Google
2. Bạn đăng nhập Google, bấm "Cho phép"
3. Google trả về **mã code** cho web
4. Web đổi code lấy **access token**
5. Web dùng token để lấy thông tin bạn (tên, email)

→ Web **không bao giờ thấy mật khẩu Google** của bạn!

### 5.2. Các khái niệm

| Thuật ngữ | Nghĩa | Ví dụ |
|-----------|-------|-------|
| Resource Owner | Chủ dữ liệu (bạn) | Bạn |
| Client | App muốn truy cập | Web lạ |
| Authorization Server | Nơi xác thực | Google / Okta |
| Access Token | "Vé" truy cập | Chuỗi JWT |
| Scope | Phạm vi quyền | `openid profile email` |

### 5.3. OpenID Connect (OIDC) là gì?

**OIDC = OAuth 2.0 + Authentication**. OAuth chỉ ủy quyền, OIDC thêm lớp "xác thực danh tính":
- Trả về **ID Token** (JWT) chứa thông tin user: tên, email, avatar
- Có endpoint chuẩn `/.well-known/openid-configuration` để app tự khám phá

### 5.4. Luồng Authorization Code (dùng nhiều nhất)

```
1. User bấm "Đăng nhập"
   → App chuyển hướng đến:
   https://dev-xxx.okta.com/oauth2/default/v1/authorize?
     client_id=ABC&
     redirect_uri=https://myapp.com/callback&
     response_type=code&
     scope=openid+profile+email&
     state=xyz

2. User đăng nhập Okta, bấm đồng ý
   → Okta chuyển về:
   https://myapp.com/callback?code=AUTH_CODE&state=xyz

3. Backend đổi code lấy token (server-to-server):
   POST https://dev-xxx.okta.com/oauth2/default/v1/token
   { code, client_id, client_secret, grant_type: "authorization_code" }

4. Nhận về:
   {
     "access_token": "eyJhbG...",
     "id_token": "eyJhbG...",
     "token_type": "Bearer",
     "expires_in": 3600
   }

5. Giải mã ID Token (JWT) → biết user là ai:
   { "sub": "00u123", "name": "Test User", "email": "test@example.com" }
```

### 5.5. ID Token vs Access Token

| | ID Token | Access Token |
|---|---|---|
| Mục đích | Cho **frontend** biết user là ai | Cho **backend** gọi API |
| Định dạng | JWT (đọc được) | JWT hoặc opaque |
| Chứa | name, email, picture | scopes, permissions |
| Gửi cho ai | Không gửi đi đâu | Gửi kèm mỗi API request |

---

## 6. Tạo Application Đầu Tiên

### 6.1. Tạo app trên Okta

1. **Applications** → **Create App Integration**
2. **Sign-in method:** `OIDC - OpenID Connect`
3. **Application type:** `Web Application`
4. Điền:
   - **App name:** `My Web App`
   - **Sign-in redirect URIs:** `http://localhost:3000/callback`
   - **Sign-out redirect URIs:** `http://localhost:3000`
   - **Assignments:** chọn `Allow everyone in your organization` (cho test)
5. **Save** → nhận được:
   - **Client ID:** `0oa1b2c3d4e5f6g7h8i9`
   - **Client secret:** bấm **Show** để xem (giữ bí mật!)

### 6.2. Lấy Issuer URL

**Security** → **API** → **Authorization Servers** → `default`:
```
Issuer: https://dev-xxx.okta.com/oauth2/default
```

Metadata đầy đủ:
```
https://dev-xxx.okta.com/oauth2/default/.well-known/openid-configuration
```

---

## 7. Thêm Đăng Nhập Vào Web (JavaScript)

Dùng thư viện `@okta/okta-auth-js` — không cần tự code luồng OAuth phức tạp.

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <title>Đăng nhập với Okta</title>
  <script src="https://global.oktacdn.com/okta-auth-js/7.5.0/okta-auth-js.min.js"></script>
</head>
<body>
  <div id="chua-dang-nhap">
    <h1>Chào mừng!</h1>
    <button onclick="dangNhap()">Đăng nhập bằng Okta</button>
  </div>
  <div id="da-dang-nhap" style="display:none">
    <h1>Xin chào, <span id="ten"></span>!</h1>
    <p>Email: <span id="email"></span></p>
    <button onclick="dangXuat()">Đăng xuất</button>
  </div>

  <script>
    const authClient = new OktaAuth({
      issuer: 'https://dev-XXX.okta.com/oauth2/default',
      clientId: 'CLIENT_ID_CUA_BAN',
      redirectUri: window.location.origin + '/callback',
      scopes: ['openid', 'profile', 'email'],
      pkce: true,   // bắt buộc cho SPA
    });

    // Xử lý khi quay về từ Okta (trang /callback)
    async function xuLyCallback() {
      if (window.location.pathname === '/callback') {
        await authClient.token.parseFromUrl();
        window.location.href = '/';
      }
    }

    async function dangNhap() {
      await authClient.token.getWithRedirect({
        responseType: ['id_token', 'token'],
      });
    }

    async function dangXuat() {
      await authClient.signOut();
      location.reload();
    }

    async function kiemTra() {
      const session = await authClient.session.exists();
      if (session) {
        const user = await authClient.token.getUserInfo();
        document.getElementById('chua-dang-nhap').style.display = 'none';
        document.getElementById('da-dang-nhap').style.display = 'block';
        document.getElementById('ten').textContent = user.name;
        document.getElementById('email').textContent = user.email;
      }
    }

    xuLyCallback().then(kiemTra);
  </script>
</body>
</html>
```

> ⚠️ Thay `dev-XXX` và `CLIENT_ID_CUA_BAN` bằng giá trị thật từ mục 6.

---

## 8. Thêm Đăng Nhập Vào React

```bash
npm install @okta/okta-react @okta/okta-auth-js react-router-dom
```

```jsx
// src/App.jsx
import { BrowserRouter, Routes, Route, useNavigate } from 'react-router-dom';
import { Security, LoginCallback, useOktaAuth } from '@okta/okta-react';
import { OktaAuth } from '@okta/okta-auth-js';

const oktaAuth = new OktaAuth({
  issuer: 'https://dev-XXX.okta.com/oauth2/default',
  clientId: 'CLIENT_ID_CUA_BAN',
  redirectUri: window.location.origin + '/login/callback',
  scopes: ['openid', 'profile', 'email'],
  pkce: true,
});

function TrangChu() {
  const { oktaAuth, authState } = useOktaAuth();
  const navigate = useNavigate();

  if (!authState) return <div>Đang tải...</div>;

  return authState.isAuthenticated ? (
    <div>
      <h1>Xin chào!</h1>
      <button onClick={() => oktaAuth.signOut()}>Đăng xuất</button>
    </div>
  ) : (
    <button onClick={() => navigate('/login')}>Đăng nhập</button>
  );
}

function DangNhap() {
  const { oktaAuth } = useOktaAuth();
  // Tự chuyển sang Okta
  oktaAuth.signInWithRedirect();
  return null;
}

export default function App() {
  return (
    <BrowserRouter>
      <Security oktaAuth={oktaAuth}>
        <Routes>
          <Route path="/" element={<TrangChu />} />
          <Route path="/login" element={<DangNhap />} />
          <Route path="/login/callback" element={<LoginCallback />} />
        </Routes>
      </Security>
    </BrowserRouter>
  );
}
```

### Bảo vệ route (chỉ user đăng nhập mới vào được)

```jsx
import { useOktaAuth } from '@okta/okta-react';
import { Navigate } from 'react-router-dom';

function RouteBaoVe({ children }) {
  const { authState } = useOktaAuth();
  if (!authState) return <div>Đang tải...</div>;
  return authState.isAuthenticated
    ? children
    : <Navigate to="/login" />;
}

// Dùng:
<Route path="/admin" element={
  <RouteBaoVe><TrangAdmin /></RouteBaoVe>
} />
```

---

## 9. Backend: Xác Thực Token (Node.js)

Frontend gửi access token lên → backend phải **xác thực** token thật từ Okta (không tin blind!).

```bash
npm install @okta/jwt-verifier express
```

```javascript
// server.js
const express = require('express');
const OktaJwtVerifier = require('@okta/jwt-verifier');

const verifier = new OktaJwtVerifier({
  issuer: 'https://dev-XXX.okta.com/oauth2/default',
  clientId: 'CLIENT_ID_CUA_BAN',
});

const app = express();

// Middleware: kiểm tra token
async function kiemTraToken(req, res, next) {
  try {
    const auth = req.headers.authorization || '';
    const token = auth.replace('Bearer ', '');
    if (!token) return res.status(401).json({ loi: 'Thiếu token' });

    const jwt = await verifier.verifyAccessToken(token, 'api://default');
    req.user = jwt.claims;  // { sub, email, ... }
    next();
  } catch (err) {
    res.status(401).json({ loi: 'Token không hợp lệ' });
  }
}

// API công khai
app.get('/api/cong-khai', (req, res) => {
  res.json({ message: 'Ai cũng xem được' });
});

// API cần đăng nhập
app.get('/api/rieng-tu', kiemTraToken, (req, res) => {
  res.json({
    message: `Chào user ${req.user.sub}!`,
    email: req.user.email,
  });
});

app.listen(3001, () => console.log('API chạy ở :3001'));
```

```javascript
// Frontend gọi API kèm token
const { accessToken } = await oktaAuth.tokenManager.get('accessToken');
const res = await fetch('http://localhost:3001/api/rieng-tu', {
  headers: { Authorization: `Bearer ${accessToken.accessToken}` },
});
```

---

## 10. Backend: Xác Thực Token (Python)

```bash
pip install flask pyjwt cryptography requests
```

```python
# app.py
import jwt
import requests
from flask import Flask, request, jsonify
from functools import wraps

ISSUER = "https://dev-XXX.okta.com/oauth2/default"
CLIENT_ID = "CLIENT_ID_CUA_BAN"

# Lấy public keys của Okta (cache lại)
JWKS_URL = f"{ISSUER}/v1/keys"
jwks = requests.get(JWKS_URL, timeout=10).json()

app = Flask(__name__)

def kiem_tra_token(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        auth = request.headers.get("Authorization", "")
        token = auth.replace("Bearer ", "")
        if not token:
            return jsonify({"loi": "Thiếu token"}), 401
        try:
            # Lấy key phù hợp từ JWKS
            header = jwt.get_unverified_header(token)
            key = next(k for k in jwks["keys"] if k["kid"] == header["kid"])
            public_key = jwt.algorithms.RSAAlgorithm.from_jwk(key)

            claims = jwt.decode(
                token, public_key,
                algorithms=["RS256"],
                issuer=ISSUER,
                audience="api://default",
            )
            request.user = claims
        except Exception as e:
            return jsonify({"loi": f"Token không hợp lệ: {e}"}), 401
        return f(*args, **kwargs)
    return decorated

@app.get("/api/cong-khai")
def cong_khai():
    return {"message": "Ai cũng xem được"}

@app.get("/api/rieng-tu")
@kiem_tra_token
def rieng_tu():
    return {
        "message": f"Chào user {request.user['sub']}!",
        "email": request.user.get("email"),
    }

app.run(port=5000)
```

---

## 11. Social Login (Google, Facebook)

Cho phép user đăng nhập bằng Google/Facebook thay vì tạo tài khoản mới.

### 11.1. Thêm Google làm IdP

1. **Security** → **Identity Providers** → **Add identity provider** → **Google**
2. Cần **Client ID** và **Client Secret** từ Google Cloud Console:
   - https://console.cloud.google.com → tạo OAuth client
   - Authorized redirect URI: `https://dev-XXX.okta.com/oauth2/v1/authorize/callback`
3. Dán vào Okta → **Add**
4. Vào **Applications** → app của bạn → **Sign On** → thêm Google vào **Identity Provider Routing**

Giờ trang đăng nhập Okta sẽ hiện nút **"Đăng nhập bằng Google"**!

### 11.2. Các IdP hỗ trợ

| IdP | Cần setup |
|-----|-----------|
| Google | OAuth client ID/secret |
| Facebook | App ID/secret |
| Apple | Service ID, key |
| Microsoft | Application ID/secret |
| GitHub | OAuth App |
| SAML tùy chỉnh | Metadata XML |

---

## 12. MFA — Xác Thực Đa Yếu Tố

### 12.1. Bật MFA

**Security** → **Multifactor** → bật các phương thức:

| Phương thức | Bảo mật | Ghi chú |
|-------------|---------|---------|
| Okta Verify (app) | Cao | App của Okta, push notification |
| Google Authenticator | Cao | TOTP, dùng chung được |
| SMS | Trung bình | Dễ bị SIM swap |
| Email | Thấp | Chỉ nên backup |
| WebAuthn (vân tay/USB key) | **Cao nhất** | Chuẩn FIDO2 |

### 12.2. Tạo policy yêu cầu MFA

**Security** → **Authentication Policies** → tạo policy:

```
IF: user thuộc group "Admin"
THEN: yêu cầu MFA mỗi lần đăng nhập

IF: đăng nhập từ IP lạ
THEN: yêu cầu MFA

ELSE: chỉ cần password
```

### 12.3. Test MFA

1. Đăng nhập bằng user test
2. Okta yêu cầu enroll MFA → quét QR bằng Google Authenticator
3. Lần sau đăng nhập: password + mã 6 số

---

## 13. Quản Lý Người Dùng & Groups

### 13.1. Tạo user hàng loạt (CSV import)

**Directory** → **People** → **More Actions** → **Import from CSV**:

```csv
firstName,lastName,login,email
An,Nguyen,an@congty.com,an@congty.com
Bình,Trần,binh@congty.com,binh@congty.com
```

### 13.2. Groups

**Directory** → **Groups** → **Add Group**:
- `admin` — quản trị viên
- `nhan-vien` — nhân viên thường
- `khach-hang` — khách hàng

Gán user vào group → phân quyền theo group (mục 14).

### 13.3. Group rules (tự động)

Tự động cho user vào group theo điều kiện:

```
IF user.department == "Kỹ thuật"
THEN add to group "dev-team"
```

**Directory** → **Groups** → chọn group → **Rules** → **Add Rule**.

---

## 14. Authorization: Phân Quyền Theo Role

### 14.1. Thêm groups vào token (claims)

Mặc định token không chứa groups. Cấu hình để có:

1. **Security** → **API** → **Authorization Servers** → `default` → **Claims**
2. **Add Claim**:
   - Name: `groups`
   - Include in token type: `ID Token`, `Access Token`
   - Value type: `Groups`
   - Filter: `Matches regex` → `.*`

Giờ token sẽ có:
```json
{
  "sub": "00u123",
  "email": "test@example.com",
  "groups": ["admin", "nhan-vien"]
}
```

### 14.2. Kiểm tra quyền ở backend

```javascript
function canAdmin(req, res, next) {
  const groups = req.user.groups || [];
  if (!groups.includes('admin')) {
    return res.status(403).json({ loi: 'Cần quyền admin' });
  }
  next();
}

app.delete('/api/users/:id', kiemTraToken, canAdmin, (req, res) => {
  // chỉ admin xóa được user
});
```

### 14.3. Kiểm tra quyền ở frontend

```jsx
function TrangAdmin() {
  const { authState } = useOktaAuth();
  const groups = authState?.idToken?.claims.groups || [];

  if (!groups.includes('admin')) {
    return <div>⛔ Bạn không có quyền truy cập trang này.</div>;
  }
  return <div>Chào admin!</div>;
}
```

---

## 15. Okta API

Mọi thứ trên dashboard đều gọi được qua API — để tự động hóa.

### 15.1. Tạo API token

**Security** → **API** → **Tokens** → **Create Token** → đặt tên → copy token (chỉ hiện 1 lần!).

### 15.2. Quản lý users

```bash
export OKTA_DOMAIN="dev-XXX.okta.com"
export TOKEN="00Txxx"

# Liệt kê users
curl -H "Authorization: SSWS $TOKEN" \
  "https://$OKTA_DOMAIN/api/v1/users?limit=10" | jq '.[].profile.email'

# Tạo user mới
curl -X POST -H "Authorization: SSWS $TOKEN" \
  -H "Content-Type: application/json" \
  "https://$OKTA_DOMAIN/api/v1/users?activate=true" \
  -d '{
    "profile": {
      "firstName": "An",
      "lastName": "Nguyễn",
      "email": "an@congty.com",
      "login": "an@congty.com"
    }
  }'

# Vô hiệu hóa user
curl -X POST -H "Authorization: SSWS $TOKEN" \
  "https://$OKTA_DOMAIN/api/v1/users/USER_ID/lifecycle/deactivate"

# Xóa user (phải deactivate trước)
curl -X DELETE -H "Authorization: SSWS $TOKEN" \
  "https://$OKTA_DOMAIN/api/v1/users/USER_ID"
```

### 15.3. Python SDK

```bash
pip install okta
```

```python
from okta.client import Client as OktaClient

config = {
    "orgUrl": "https://dev-XXX.okta.com",
    "token": "00Txxx",
}
client = OktaClient(config)

# Liệt kê users
users, _, _ = await client.list_users()
for u in users:
    print(u.profile.email)

# Tạo user
from okta.models import UserProfile, CreateUserRequest
profile = UserProfile()
profile.first_name = "Bình"
profile.last_name = "Trần"
profile.email = "binh@congty.com"
profile.login = "binh@congty.com"
req = CreateUserRequest({"profile": profile})
user, _, _ = await client.create_user(req, {"activate": "true"})
print("Tạo user:", user.id)
```

---

## 16. Universal Directory & Lifecycle

### 16.1. Profile mappings

Khi user đăng nhập bằng Google lần đầu, Okta tự tạo profile. Tùy chỉnh mapping:

**Directory** → **Profile Editor** → chọn app → **Mappings**:
```
Google "given_name" → Okta "firstName"
Google "family_name" → Okta "lastName"
```

### 16.2. Lifecycle events

| Sự kiện | Tự động làm gì |
|---------|----------------|
| User được tạo | Gửi email kích hoạt |
| User bị deactivate | Thu hồi mọi session, token |
| User bị xóa | Xóa khỏi mọi app |
| Password hết hạn | Yêu cầu đổi |

### 16.3. Workflows (tự động hóa)

**Workflow** → tạo flow không cần code, ví dụ "Nhân viên mới":

```
Khi: user được thêm vào group "nhan-vien"
Thì:
  1. Tạo tài khoản Google Workspace
  2. Thêm vào Slack workspace
  3. Gửi email chào mừng
  4. Tạo ticket IT cấp laptop
```

---

## 17. SSO Cho Doanh Nghiệp (SAML)

SAML là chuẩn SSO cũ nhưng vẫn phổ biến trong doanh nghiệp (dùng cho Salesforce, AWS, Office 365...).

### 17.1. Okta làm IdP (cho nhân viên đăng nhập app công ty)

1. **Applications** → **Create App Integration** → **SAML 2.0**
2. Điền **Single sign-on URL** và **Audience URI** của app (lấy từ docs app đó)
3. **Attribute Statements**: map `user.email` → `email`
4. Gán cho group nhân viên
5. Tải **Metadata XML** đưa cho bên app

### 17.2. So sánh SAML vs OIDC

| | SAML 2.0 | OIDC |
|---|---|---|
| Định dạng | XML | JSON/JWT |
| Dùng cho | App doanh nghiệp cũ | App hiện đại, mobile |
| Độ phức tạp | Cao | Thấp hơn |
| Khuyên dùng | Khi app chỉ hỗ trợ SAML | Mọi trường hợp khác |

---

## 18. Bảo Mật Nâng Cao

### 18.1. Password policy

**Security** → **Authentication Policies** → **Password**:
- Độ dài tối thiểu: 12 ký tự (khuyên dùng)
- Yêu cầu: chữ hoa, chữ thường, số, ký tự đặc biệt
- Không cho dùng password đã lộ (check HaveIBeenPwned)
- Lịch sử: không lặp lại 5 password gần nhất

### 18.2. Network zones

**Security** → **Networks** → **Zones**:
- Định nghĩa IP công ty → policy riêng (không cần MFA trong công ty)
- Chặn IP từ quốc gia không liên quan
- Phát hiện đăng nhập từ IP lạ → yêu cầu MFA

### 18.3. ThreatInsight

Tự động chặn IP đã biết độc hại (botnet, brute force):
**Security** → **ThreatInsight** → **Enable**.

### 18.4. API token best practices

- Mỗi integration 1 token riêng (đừng dùng chung)
- Đặt tên rõ ràng: `ci-deploy`, `user-sync`
- Rotate định kỳ 90 ngày
- Token lộ → revoke ngay lập tức

---

## 19. Pricing

### 19.1. Okta Workforce (nhân viên)

| Gói | Giá | Tính năng |
|-----|-----|-----------|
| Starter | $2/user/tháng | SSO cơ bản, MFA |
| SSO | $3/user/tháng | + SAML, SCIM |
| MFA | $3/user/tháng | + Adaptive MFA |
| Enterprise | Liên hệ | Full |

### 19.2. Customer Identity (Auth0)

| Gói | Giá |
|-----|-----|
| Free | 25,000 MAU |
| Professional | Từ $35/tháng (1,000 MAU) |
| Enterprise | Liên hệ |

### 19.3. Developer (học tập)

**Miễn phí:** 100 MAU, đầy đủ tính năng core — đủ để học và demo.

---

## 20. Troubleshooting

| Vấn đề | Nguyên nhân | Cách khắc phục |
|--------|-------------|----------------|
| `redirect_uri mismatch` | URI callback chưa đăng ký | Thêm đúng URI vào app settings (kể cả port!) |
| `invalid_client` | Sai Client ID/secret | Kiểm tra lại, secret có khoảng trắng thừa không |
| Không nhận được groups trong token | Chưa cấu hình claims | Làm theo mục 14.1 |
| CORS error | Origin chưa cho phép | **Security** → **API** → **Trusted Origins** → thêm domain |
| Token hết hạn nhanh | Mặc định 1 giờ | Tăng lifetime ở Authorization Server → Access Policies |
| User không đăng nhập được | Chưa assign app cho user | Applications → app → Assignments → gán user/group |
| MFA loop | Policy conflict | Kiểm tra Authentication Policies |
| `401` khi gọi Okta API | Token sai/hết hạn | Tạo token mới; kiểm tra header `SSWS` (không phải Bearer) |

### Debug JWT token

Dán token vào https://jwt.io để xem claims bên trong — kiểm tra `iss`, `aud`, `exp`, `groups`.

---

## 21. Best Practices & Cheat Sheet

### 21.1. Best practices

1. **Không bao giờ** để Client Secret ở frontend — chỉ backend dùng
2. **Dùng PKCE** cho SPA/mobile (không cần client secret)
3. **Validate token ở backend** mọi request — đừng tin frontend
4. **Scopes tối thiểu** — chỉ xin quyền cần thiết
5. **Rotate secrets** định kỳ
6. **Bật MFA** cho admin Okta (bắt buộc!)
7. **Trusted Origins** chỉ thêm domain thật
8. **Test với user thường** — đừng chỉ test bằng admin

### 21.2. Cheat sheet

```
# Okta domain
https://dev-XXX.okta.com

# OIDC Discovery (metadata)
https://dev-XXX.okta.com/oauth2/default/.well-known/openid-configuration

# Authorize endpoint
https://dev-XXX.okta.com/oauth2/default/v1/authorize

# Token endpoint
https://dev-XXX.okta.com/oauth2/default/v1/token

# Userinfo
https://dev-XXX.okta.com/oauth2/default/v1/userinfo

# Luồng login
GET /authorize?client_id=...&redirect_uri=...&response_type=code&scope=openid+profile+email&state=...

# Đổi code lấy token
POST /token { grant_type: authorization_code, code, redirect_uri }
+ Basic Auth: client_id:client_secret

# Gọi Okta API (dùng SSWS, không phải Bearer!)
curl -H "Authorization: SSWS $TOKEN" https://dev-XXX.okta.com/api/v1/users
```

### 21.3. Tài liệu tham khảo

- Developer docs: https://developer.okta.com/docs
- Auth0 docs: https://auth0.com/docs
- OIDC spec: https://openid.net/connect
- OAuth 2.0: https://oauth.net/2

---

*Cập nhật: 10/10/2026*

---

## 22. Okta Sign-In Widget — Form Đăng Nhập Nhúng

Thay vì chuyển hướng sang trang Okta, nhúng form đăng nhập ngay trong web của bạn:

```bash
npm install @okta/okta-signin-widget
```

```jsx
// src/Login.jsx
import { useEffect, useRef } from 'react';
import OktaSignIn from '@okta/okta-signin-widget';
import '@okta/okta-signin-widget/dist/css/okta-sign-in.min.css';
import { useOktaAuth } from '@okta/okta-react';

export default function Login() {
  const { oktaAuth } = useOktaAuth();
  const widgetRef = useRef();

  useEffect(() => {
    const widget = new OktaSignIn({
      baseUrl: 'https://dev-XXX.okta.com',
      clientId: 'CLIENT_ID_CUA_BAN',
      redirectUri: window.location.origin + '/login/callback',
      authParams: {
        issuer: 'https://dev-XXX.okta.com/oauth2/default',
        scopes: ['openid', 'profile', 'email'],
      },
      // Tùy chỉnh giao diện
      colors: { brand: '#00297a' },
      language: 'vi',  // tiếng Việt!
    });

    widget.renderEl(
      { el: widgetRef.current },
      () => {},  // thành công -> tự redirect
      (err) => console.error('Lỗi đăng nhập:', err),
    );

    return () => widget.remove();
  }, [oktaAuth]);

  return <div ref={widgetRef} />;
}
```

Widget tự có: form đăng nhập, quên mật khẩu, đăng ký, MFA, social login buttons.

---

## 23. Inline Hooks — Tùy Biến Luồng Auth

Inline Hooks cho phép gọi API của bạn **trong lúc** Okta xử lý — để thêm logic tùy chỉnh.

### 23.1. Registration hook (khi user đăng ký)

```javascript
// API của bạn: POST /okta-hooks/register
app.post('/okta-hooks/register', (req, res) => {
  const { email } = req.body.data.userProfile;

  // Chặn email rác
  if (email.endsWith('@spam.com')) {
    return res.json({
      commands: [{
        type: 'com.okta.action.update',
        value: { registration: 'DENY' },
      }],
    });
  }

  // Tự gán group theo domain email
  const group = email.endsWith('@congty.com') ? 'nhan-vien' : 'khach-hang';
  res.json({
    commands: [{
      type: 'com.okta.user.group.assign',
      value: { group },
    }],
  });
});
```

Đăng ký hook: **Workflow** → **Inline Hooks** → **Add** → dán URL endpoint.

### 23.2. Token hook (thêm claims động)

```javascript
// Thêm "is_vip" vào token dựa trên database của bạn
app.post('/okta-hooks/token', async (req, res) => {
  const userId = req.body.data.context.user.id;
  const isVip = await db.users.isVip(userId);

  res.json({
    commands: [{
      type: 'com.okta.identity.patch',
      value: [{
        op: 'add',
        path: '/claims/is_vip',
        value: isVip,
      }],
    }],
  });
});
```

---

## 24. SCIM Provisioning — Đồng Bộ User Tự Động

SCIM tự động tạo/sửa/xóa user ở app bên thứ 3 khi có thay đổi trên Okta.

Ví dụ: nhân viên mới vào công ty → Okta tự tạo tài khoản Slack, Google Workspace cho họ. Nghỉ việc → tự thu hồi.

### Cấu hình cơ bản

1. **Applications** → app (ví dụ Slack) → **Provisioning** → **Configure API Integration** → bật
2. Điền API credentials của app đích
3. **To App** → bật:
   - Create Users
   - Update User Attributes
   - Deactivate Users
4. Gán group → user tự động được provision!

---

## 25. Ví Dụ Hoàn Chỉnh: Todo App Có Đăng Nhập

### Frontend (React)

```jsx
// src/App.jsx — tóm tắt
import { Security, useOktaAuth } from '@okta/okta-react';
import { OktaAuth } from '@okta/okta-auth-js';

const oktaAuth = new OktaAuth({
  issuer: 'https://dev-XXX.okta.com/oauth2/default',
  clientId: 'CLIENT_ID',
  redirectUri: window.location.origin + '/login/callback',
  scopes: ['openid', 'profile', 'email'],
  pkce: true,
});

function TodoList() {
  const { oktaAuth, authState } = useOktaAuth();
  const [todos, setTodos] = useState([]);

  useEffect(() => {
    (async () => {
      const { accessToken } = await oktaAuth.tokenManager.get('accessToken');
      const res = await fetch('/api/todos', {
        headers: { Authorization: `Bearer ${accessToken.accessToken}` },
      });
      setTodos(await res.json());
    })();
  }, []);

  return (
    <ul>{todos.map(t => <li key={t.id}>{t.title}</li>)}</ul>
  );
}
```

### Backend (Express)

```javascript
// Mỗi user chỉ thấy todo của mình (dựa vào sub trong token)
const todos = {};  // { userId: [todos] }

app.get('/api/todos', kiemTraToken, (req, res) => {
  const userId = req.user.sub;
  res.json(todos[userId] || []);
});

app.post('/api/todos', kiemTraToken, (req, res) => {
  const userId = req.user.sub;
  const todo = { id: Date.now(), title: req.body.title };
  todos[userId] = [...(todos[userId] || []), todo];
  res.status(201).json(todo);
});
```

---

## 26. Migration: Từ Tự Code Sang Okta

### Bước 1: Export users hiện tại

```javascript
// Script migrate 1 lần
const bcrypt = require('bcrypt');

for (const user of usersCu) {
  // Okta API không nhận password hash bcrypt trực tiếp,
  // nên tạo user với password ngẫu nhiên + bắt đổi khi login đầu
  await oktaClient.createUser({
    profile: {
      firstName: user.ten,
      lastName: '',
      email: user.email,
      login: user.email,
    },
    credentials: {
      // Nếu dùng cùng thuật toán hash, có thể import hook
      password: { value: generateRandom() },
    },
  });
}
```

### Bước 2: Password import hook (giữ nguyên password cũ)

Okta hỗ trợ **password import hook**: khi user đăng nhập lần đầu, Okta gọi API của bạn để kiểm tra password cũ:

```javascript
app.post('/okta-hooks/password-import', async (req, res) => {
  const { login, password } = req.body.data.context.credential;
  const userCu = await db.users.findByEmail(login);

  const dung = await bcrypt.compare(password, userCu.password_hash);
  if (!dung) {
    return res.json({ commands: [{
      type: 'com.okta.action.update',
      value: { credential: 'UNVERIFIED' },
    }]});
  }

  // Đúng → Okta lưu password mới (hash theo chuẩn Okta)
  res.json({ commands: [{
    type: 'com.okta.action.update',
    value: {
      credential: 'VERIFIED',
      password: { value: password },
    },
  }]});
});
```

→ User **không cần đổi mật khẩu**, migrate trong suốt!

---

## 27. So Sánh Chi Tiết: Khi Nào Dùng Gì

| Tình huống | Giải pháp khuyên dùng |
|------------|----------------------|
| Side project, muốn nhanh | Auth0 free (25k MAU) |
| App mobile + Firebase sẵn | Firebase Auth |
| Công ty 100+ nhân viên, cần SSO | Okta Workforce |
| Không muốn phụ thuộc vendor | Keycloak tự host |
| Chỉ cần "Đăng nhập bằng Google" đơn giản | Google Identity Services (không cần Okta!) |
| Cần HIPAA/SOC2 compliance | Okta Enterprise |

---

## 28. Câu Hỏi Thường Gặp (FAQ)

**Okta và Auth0 khác nhau thế nào?**
Cùng công ty mẹ (Okta mua Auth0). Auth0 dễ dùng hơn cho developer, free hào phóng hơn. Okta mạnh về doanh nghiệp (SAML, lifecycle).

**Tôi chỉ cần nút "Đăng nhập bằng Google", có cần Okta không?**
Không! Dùng trực tiếp **Google Identity Services** đơn giản hơn nhiều. Chỉ dùng Okta khi cần thêm: nhiều IdP, MFA, phân quyền, quản lý user tập trung.

**Access token nên lưu ở đâu ở frontend?**
- **Không** lưu ở `localStorage` (dễ bị XSS đánh cắp)
- Okta SDK mặc định lưu ở memory + dùng **refresh token rotation** với PKCE — an toàn nhất
- Nếu cần "nhớ đăng nhập": bật refresh token trong app settings

**ID token hết hạn thì sao?**
Okta SDK tự refresh bằng refresh token (nếu bật). Bật ở: **Applications** → app → **General** → **Refresh Token** →勾选.

**Có dùng Okta cho mobile app (Flutter/React Native) không?**
Có. Dùng luồng **Authorization Code + PKCE** (không cần client secret). Có SDK: `okta-react-native`, hoặc dùng `AppAuth` (chuẩn OAuth cho mobile).

**Okta có hỗ trợ tiếng Việt không?**
Sign-In Widget hỗ trợ `language: 'vi'`. Trang hosted login của Okta cũng có tiếng Việt.

---

*Cập nhật: 10/10/2026 — Tổng 28 phần*
