# Hướng dẫn Tailscale — VPN mesh kết nối mọi thiết bị

> Tailscale là dịch vụ VPN dựa trên WireGuard, giúp các thiết bị của bạn (điện thoại, máy tính, VPS)
> kết nối với nhau thành một mạng riêng an toàn, như đang chung một WiFi — dù chúng ở bất kỳ đâu
> trên thế giới.

---

## 1. Tailscale là gì?

- **Mesh VPN**: mỗi thiết bị kết nối trực tiếp với nhau (peer-to-peer), không qua server trung gian
  làm chậm (trừ khi bị NAT chặn thì relay qua DERP).
- **Dựa trên WireGuard**: giao thức VPN hiện đại, nhanh, bảo mật.
- **Zero-config**: không cần mở port, không cần IP tĩnh, không cần cấu hình firewall phức tạp.
- **Miễn phí** cho cá nhân: gói Personal free cho tối đa 100 thiết bị + 3 user.

### Khi nào nên dùng?

- SSH vào VPS từ điện thoại mà không mở port 22 ra internet (an toàn hơn).
- Truy cập web/app chạy local trên VPS (dashboard, admin panel...) từ xa.
- Chia sẻ file giữa các máy như trong mạng LAN.
- Chơi game LAN online với bạn bè.

---

## 2. Đăng ký tài khoản

1. Vào [tailscale.com](https://tailscale.com) → **Get started**.
2. Đăng nhập bằng tài khoản có sẵn: **Google, Microsoft, GitHub, Apple...**
   (khuyên dùng Google cho tiện).
3. Vào trang quản trị: [login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines).

---

## 3. Cài đặt

### 3.1. Android

1. Mở **CH Play** → tìm **Tailscale** → cài đặt.
2. Mở app → đăng nhập cùng tài khoản đã đăng ký.
3. Bật công tắc **VPN** trong app (Android sẽ hỏi xác nhận kết nối VPN → bấm OK).
4. Xong! Máy đã vào mạng Tailscale, có IP dạng `100.x.y.z`.

> Mẹo: vào phần **Settings** trong app để xem IP Tailscale của máy.

### 3.2. Ubuntu / Debian (VPS)

Chạy từng lệnh:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Xong rồi khởi động và đăng nhập:

```bash
sudo tailscale up
```

Lệnh này sẽ in ra một đường link — mở link đó trên trình duyệt (điện thoại cũng được),
đăng nhập để xác thực máy. Quay lại terminal, máy đã vào mạng.

Kiểm tra IP:

```bash
tailscale ip -4
# ví dụ: 100.101.102.103
```

### 3.3. Windows / macOS

1. Tải app từ [tailscale.com/download](https://tailscale.com/download).
2. Cài đặt → đăng nhập → bật.

---

## 4. Sử dụng cơ bản

### 4.1. Xem danh sách máy

- Trên web: [login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines)
- Trên VPS: `tailscale status`

Mỗi máy có:
- **Tên**: ví dụ `vps-1`, `dien-thoai` (đổi tên được trên web).
- **IP Tailscale**: dạng `100.x.y.z` — dùng IP này để kết nối.

### 4.2. SSH vào VPS từ điện thoại (không mở port)

Giả sử VPS có IP Tailscale là `100.101.102.103`:

1. Trên điện thoại: cài app **Termux** (CH Play) hoặc **JuiceSSH**.
2. Trong Termux:
   ```bash
   pkg install openssh -y
   ssh root@100.101.102.103
   ```
3. Nhập mật khẩu/root key như bình thường.

> Port 22 **không cần mở** ra internet — lưu lượng đi qua tunnel mã hóa của Tailscale.

### 4.3. Mở web chạy trên VPS từ điện thoại

Giả sử trên VPS có dashboard chạy ở `localhost:3000`:

- Từ điện thoại, mở trình duyệt → vào `http://100.101.102.103:3000`
- Không cần domain, không cần HTTPS public (trong mạng Tailscale đã mã hóa).

### 4.4. Ping kiểm tra kết nối

```bash
# trên VPS
tailscale ping 100.x.y.z   # IP Tailscale của điện thoại
```

---

## 5. Tính năng nâng cao

### 5.1. MagicDNS — gọi tên thay vì nhớ IP

Mặc định Tailscale đã bật MagicDNS: mỗi máy có tên miền dạng
`<ten-may>.<ten-mang>.ts.net`.

Ví dụ: thay vì `ssh root@100.101.102.103`, dùng:

```bash
ssh root@vps-1
```

Bật/tắt tại: Admin console → **DNS** → Enable MagicDNS.

### 5.2. Tailscale Serve — chia sẻ web local

Muốn cho máy khác trong mạng xem web đang chạy local trên VPS:

```bash
# chia sẻ cổng 3000
tailscale serve --bg 3000
```

Máy khác vào `https://vps-1.<ten-mang>.ts.net` là xem được (có HTTPS tự động!).

Tắt: `tailscale serve --bg off` (hoặc `tailscale serve reset`).

### 5.3. Tailscale Funnel — public ra internet (cẩn thận)

Muốn cho **người ngoài** (không trong mạng Tailscale) truy cập web của bạn:

```bash
tailscale funnel 3000
```

Lệnh này cấp một URL public `https://<ten-may>.<ten-mang>.ts.net`.
Chỉ dùng khi thật sự cần — vì lúc này web đã lộ ra internet.

### 5.4. Exit node — dùng VPS làm lối ra internet

Cho điện thoại đi internet qua VPS (đổi IP ra ngoài):

1. Trên VPS cho phép làm exit node:
   ```bash
   sudo tailscale up --advertise-exit-node
   ```
2. Trên web admin → Machines → VPS → **Edit route settings** → bật **Exit node**.
3. Trên app điện thoại → chọn **Exit node** → chọn VPS.

### 5.5. ACL — phân quyền truy cập (gói free có giới hạn)

Mặc định mọi máy thấy nhau. Muốn giới hạn (ví dụ máy A không SSH được vào VPS),
vào Admin console → **Access controls** → sửa policy JSON. Gói Personal free
dùng được ACL cơ bản.

---

## 6. Bảo mật cần biết

- **Key xác thực**: khi `tailscale up` trên VPS, link xác thực hết hạn sau vài phút —
  mỗi lần cài lại phải xác thực lại (an toàn).
- **Tắt key expiry** (mặc định key máy hết hạn sau 180 ngày): vào Machines →
  chọn máy → **Disable key expiry** cho VPS (kẻo đang dùng thì rớt mạng).
- **Không chia sẻ** link `tailscale up` cho người khác.
- Tailscale **không thấy** nội dung lưu lượng của bạn (mã hóa end-to-end bằng WireGuard),
  họ chỉ điều phối kết nối (control plane).

---

## 7. Gỡ cài đặt

**Android**: tắt VPN trong app → gỡ app như bình thường.

**Ubuntu**:

```bash
sudo tailscale logout
sudo apt remove tailscale -y
```

---

## 8. Lỗi thường gặp

| Lỗi | Cách xử lý |
|-----|-----------|
| App báo "VPN permission denied" (Android) | Vào Cài đặt → cho phép app tạo kết nối VPN |
| `tailscale up` báo link hết hạn | Chạy lại `sudo tailscale up` để lấy link mới |
| Không ping được máy khác | Kiểm tra cả 2 máy đều **online** trong admin console; thử `tailscale status` |
| Tốc độ chậm | Có thể đang relay qua DERP do NAT chặn UDP — vẫn dùng được, hoặc thử bật UPnP trên router |
| Máy bị "key expired" | Vào admin console → Machines → bật lại máy, hoặc **Disable key expiry** |

---

## 9. Tóm tắt nhanh (cheatsheet)

```bash
# Cài (Ubuntu)
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up            # xác thực qua link

# Kiểm tra
tailscale status             # danh sách máy
tailscale ip -4              # IP của máy này
tailscale ping <ip-hoac-ten> # test kết nối

# Serve web local cho mạng Tailscale
tailscale serve --bg 3000
```

**Luồng dùng chuẩn cho người mới:**
1. Đăng ký tailscale.com (bằng Google).
2. Cài app trên điện thoại → đăng nhập → bật VPN.
3. Thuê VPS → cài Tailscale → `tailscale up` → xác thực.
4. Từ điện thoại SSH vào VPS bằng IP `100.x.y.z` — không cần mở port!
