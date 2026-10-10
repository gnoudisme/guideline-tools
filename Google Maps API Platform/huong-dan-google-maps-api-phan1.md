# 🗺️ Hướng Dẫn Toàn Tập: Google Maps Platform API

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật lần cuối:** 10/10/2026
>
> ⚠️ **CẢNH BÁO VỀ KEY:** Bài này dùng placeholder `YOUR_API_KEY` trong mọi ví dụ.
> **Tuyệt đối KHÔNG** commit API key thật lên GitHub/Git — lộ key là người khác xài chùa tiền của cậu đấy!

---

## 📑 Mục Lục

1. [Google Maps Platform là gì? Các nhóm API](#1-google-maps-platform-là-gì)
2. [Tạo project trên Google Cloud Console (từng bước click)](#2-tạo-project-trên-google-cloud-console)
3. [Enable API — bật từng API cần dùng](#3-enable-api)
4. [Tạo API key](#4-tạo-api-key)
5. [Bảo mật key — bước quan trọng nhất](#5-bảo-mật-api-key)
6. [Maps JavaScript API — nhúng bản đồ vào web](#6-maps-javascript-api)
7. [Geocoding API — địa chỉ ↔ tọa độ](#7-geocoding-api)
8. [Places API (New) — tìm kiếm địa điểm](#8-places-api-new)
9. [Directions API — chỉ đường A → B](#9-directions-api)
10. [Distance Matrix API — ma trận khoảng cách](#10-distance-matrix-api)
11. [Bảng giá & $200 free credit mỗi tháng](#11-pricing--free-credit)
12. [Billing setup — bắt buộc bật mới dùng được](#12-billing-setup)
13. [Code ví dụ tổng hợp — web mini hoàn chỉnh](#13-code-ví-dụ-tổng-hợp)
14. [Xử lý lỗi thường gặp](#14-xử-lý-lỗi-thường-gặp)
15. [Best practices & Cheat sheet](#15-best-practices--cheat-sheet)

---

## 1. Google Maps Platform Là Gì?

**Google Maps Platform (GMP)** là bộ dịch vụ bản đồ trả phí của Google dành cho lập trình viên: nhúng bản đồ vào web/app, chuyển địa chỉ thành tọa độ, tìm quán ăn gần đó, chỉ đường, tính ma trận khoảng cách... Tất cả qua API, trả tiền theo số request.

GMP chia thành **4 nhóm sản phẩm chính**:

| Nhóm | Dùng để làm gì | Các API tiêu biểu |
|------|---------------|-------------------|
| 🗺️ **Maps** | Hiển thị bản đồ, đánh dấu vị trí | Maps JavaScript API, Maps SDK for Android/iOS, Static Maps, Street View |
| 🛣️ **Routes** | Chỉ đường, tính khoảng cách/thời gian | Directions API, Distance Matrix API, Roads API, Routes API |
| 📍 **Places** | Tìm địa điểm, chi tiết, ảnh, review | Places API (New), Geocoding API, Geolocation API, Time Zone API |
| 🌱 **Environment** | Dữ liệu môi trường | Air Quality API, Pollen API, Solar API |

> 💡 Hầu hết app chỉ cần **Maps + Routes + Places**. Nhóm Environment là mảng mới, dùng cho app thời tiết/năng lượng mặt trời.

### So sánh Google Maps vs Mapbox vs OpenStreetMap

| Tiêu chí | Google Maps Platform | Mapbox | OpenStreetMap (OSM) |
|----------|---------------------|--------|---------------------|
| Dữ liệu bản đồ | Chi tiết nhất, cập nhật liên tục, dữ liệu địa điểm (quán ăn, review) vô địch | Đẹp, tùy biến style mạnh, dữ liệu tốt | Miễn phí, cộng đồng đóng góp, dữ liệu địa điểm ở VN còn thưa |
| Giá | $200 credit/tháng miễn phí, sau đó trả theo request | 50.000 map loads/tháng miễn phí | **Miễn phí** (nhưng tự host tile server thì tốn server) |
| Tìm kiếm địa điểm (Places) | **Tốt nhất** — autocomplete tiếng Việt rất chuẩn | Khá, nhưng ở VN thua Google | Nominatim miễn phí nhưng giới hạn 1 req/giây, dữ liệu thưa |
| Chỉ đường | Rất tốt, có real-time traffic | Tốt | OSRM/Valhalla tự host được, miễn phí |
| Custom style | Có (map styling) nhưng hạn chế hơn Mapbox Studio | **Mạnh nhất** — Studio kéo thả | Tự do tuyệt đối |
| Cần thẻ tín dụng? | **Bắt buộc** bật billing (dù có $200 free) | Không bắt buộc ngay | Không cần |
| Phù hợp | App thương mại cần dữ liệu địa điểm chuẩn | App cần bản đồ đẹp, style độc | Dự án 0 đồng, tự host, chấp nhận dữ liệu thưa |

**Kết luận nhanh:**
- Muốn **tìm quán ăn gần đây, autocomplete địa chỉ tiếng Việt chuẩn** → Google Maps (không đối thủ ở VN).
- Muốn **bản đồ đẹp, style theo brand** → Mapbox.
- Muốn **miễn phí hoàn toàn** → OSM + tự host (nhưng đừng mong dữ liệu địa điểm dày như Google).

---

## 2. Tạo Project Trên Google Cloud Console

Mọi thứ của Google Maps Platform đều chạy qua **Google Cloud Console**. Làm theo từng bước:

**Bước 1 — Mở Console**
Truy cập 👉 `console.cloud.google.com` và đăng nhập bằng tài khoản Google của cậu.

**Bước 2 — Tạo project mới**
1. Click vào ô **chọn project** ở góc trái thanh top bar (mặc định ghi "Select a project").
2. Trong popup, click nút **"New Project"** (Dự án mới) ở góc phải trên.
3. Điền:
   - **Project name:** ví dụ `ban-do-cua-to` (đặt tên gì cũng được, tiếng Việt có dấu cũng OK).
   - **Organization:** để mặc định (nếu không có tổ chức thì nó hiện "No organization").
   - **Location:** để mặc định.
4. Click **"Create"** → chờ vài giây cho Google tạo xong.
5. Click lại ô chọn project → chọn project vừa tạo. Từ giờ mọi thao tác đều diễn ra trong project này.

> 💡 Mỗi project có **quota và billing riêng**. Nên tách project theo app (app A, app B) để dễ quản lý chi phí và key.

**Bước 3 — Ghi nhớ Project ID**
Vào **IAM & Admin → Settings**, cậu sẽ thấy **Project ID** (dạng `ban-do-cua-to-123456`) và **Project number** (dãy số). Có lúc support hoặc API yêu cầu 2 thông tin này.

---

## 3. Enable API

Mặc định project mới **chưa bật API nào**. Key tạo ra sẽ báo lỗi nếu API chưa được enable. Bật như sau:

**Cách bật (áp dụng cho từng API):**
1. Trên Cloud Console, mở menu ☰ → **"APIs & Services"** → **"Library"** (Thư viện).
2. Gõ tên API vào ô tìm kiếm. Ví dụ gõ `Maps JavaScript API`.
3. Click vào kết quả → click nút **"Enable"** (Bật).
4. Chờ vài giây → nút chuyển thành **"Manage"** là xong.

**5 API cần bật cho bài hướng dẫn này:**

| # | Tên API (gõ đúng tên này để tìm) | Dùng cho |
|---|----------------------------------|----------|
| 1 | **Maps JavaScript API** | Nhúng bản đồ tương tác vào web |
| 2 | **Geocoding API** | Địa chỉ ↔ tọa độ |
| 3 | **Places API (New)** | Tìm kiếm địa điểm, chi tiết, ảnh, autocomplete |
| 4 | **Directions API** | Chỉ đường A → B |
| 5 | **Distance Matrix API** | Ma trận khoảng cách nhiều điểm |

> ⚠️ Chú ý: có **2 Places API** — bản cũ "Places API" (Legacy) và bản mới "Places API (New)". Từ 2025 Google khuyến khích dùng **bản New** (endpoint `places.googleapis.com`), bản cũ vẫn chạy nhưng không có tính năng mới. Bài này dùng bản **New**.

**Kiểm tra API đã bật chưa:** menu ☰ → **APIs & Services → Dashboard** — danh sách hiện ra chính là các API đang bật của project.

---

## 4. Tạo API Key

**Từng bước click:**

1. Menu ☰ → **"APIs & Services"** → **"Credentials"** (Thông tin xác thực).
2. Trên thanh công cụ phía trên, click **"＋ Create Credentials"** → chọn **"API key"**.
3. Một popup hiện ra, hiển thị key dạng `AIzaSy...` → click **"Copy"** (biểu tượng 2 tờ giấy) để copy.
4. Trong popup đó click **"Restrict key"** để đi thẳng sang phần bảo mật (mục 5 bên dưới) — **đừng bao giờ để key không giới hạn!**

> 📝 Key có dạng `AIzaSy` + 33 ký tự nữa, ví dụ: `AIzaSyXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX`.
> Trong bài này mọi ví dụ đều ghi `YOUR_API_KEY` — khi chạy thật thì cậu thay bằng key của mình.

**Quản lý key về sau:** quay lại **APIs & Services → Credentials**, cậu sẽ thấy danh sách key. Click vào tên key để xem, sửa restriction, hoặc regenerate.

---

## 5. Bảo Mật API Key

> 🔴 **Đây là mục quan trọng nhất bài.** Key không giới hạn = ai nhặt được cũng xài được = **tiền trong thẻ của cậu bay theo request của người ta**. Có không ít vụ lộ key trên GitHub rồi bị charge hàng nghìn đô.

### 5.1. Các loại restriction (giới hạn)

Mở key lên (**Credentials → click tên key**), cậu sẽ thấy 2 nhóm giới hạn:

**A. Application restrictions — "key này chỉ được gọi từ đâu"**

| Loại | Dùng khi nào | Điền gì |
|------|--------------|---------|
| **HTTP referrers (web sites)** | Key nhúng trong web frontend | Domain của web, ví dụ `*.tieuduongg.vercel.app/*`, `localhost:3000/*` |
| **IP addresses (web servers...)** | Key dùng ở backend/server gọi API trực tiếp | IP public của server, ví dụ `203.0.113.10` |
| **Android apps** | App Android | Package name + SHA-1 certificate fingerprint |
| **iOS apps** | App iOS | Bundle identifier |

Ví dụ cho web: thêm 2 dòng vào HTTP referrers:
```
*.tieuduongg.vercel.app/*
http://localhost:*/ *
```
> 💡 Dấu `*` là wildcard. Nhớ thêm cả bản localhost để dev local chạy được, và cả `https://` lẫn `http://` nếu cần.

**B. API restrictions — "key này chỉ được gọi API nào"**

Chọn **"Restrict key"** → tick chọn đúng các API trong mục 3:
- Maps JavaScript API
- Geocoding API
- Places API (New)
- Directions API
- Distance Matrix API

→ Kể cả ai đó lấy được key, họ cũng chỉ gọi được đúng 5 API này, không phá được các dịch vụ Cloud khác trong project của cậu.

### 5.2. Checklist bảo mật key

- [ ] **KHÔNG** bao giờ commit key vào git. Dùng biến môi trường (`.env`) + thêm `.env` vào `.gitignore`.
- [ ] Key frontend (Maps JS) → bắt buộc HTTP referrer restriction.
- [ ] Key backend (Geocoding/Directions gọi từ server) → IP restriction + **KHÔNG** bao giờ để lộ ra frontend. Gọi API từ server, frontend chỉ gọi server của mình.
- [ ] API restriction: chỉ tick API nào dùng, không tick "Don't restrict key".
- [ ] Đặt quota limit theo ngày cho từng API (xem mục 12) để dù lộ key thì thiệt hại cũng có trần.
- [ ] Tách key theo môi trường: 1 key cho dev (localhost), 1 key cho production.

### 5.3. Lỡ lộ key thì xoay key thế nào?

**Xoay key (rotate)** = tạo key mới → đổi app sang key mới → xóa key cũ.

1. **Credentials → Create Credentials → API key** → tạo key mới, đặt restriction đầy đủ ngay từ đầu.
2. Cập nhật app/web sang dùng key mới (đổi biến môi trường, redeploy).
3. Xác nhận app chạy ổn với key mới.
4. Quay lại Credentials → click key cũ → **"Delete"** (hoặc "Regenerate key" nếu muốn giữ nguyên tên).
5. Kiểm tra **APIs & Services → Metrics** xem còn request nào đổ về key cũ không.

> ⚠️ Nếu key bị lộ trên GitHub public: GitHub có thể tự báo cho Google và Google có thể **vô hiệu hóa key đó luôn**. Đừng hoảng — đó là cơ chế bảo vệ cậu. Cứ tạo key mới theo bước trên.

### 5.4. Mẹo: 2 key — 1 public, 1 secret

Pattern chuẩn mà dev hay dùng:

| Key | Đặt ở | Restriction |
|-----|-------|-------------|
| **Public key** (Maps JS) | Nhúng trong HTML/JS frontend — ai view-source cũng thấy, chấp nhận được | HTTP referrers (domain của cậu) + chỉ tick Maps JavaScript API |
| **Secret key** (server-side) | Chỉ nằm trên server, trong biến môi trường | IP restriction (IP server) + tick Geocoding/Directions/Distance Matrix/Places |

Frontend **không bao giờ** gọi Geocoding/Directions trực tiếp bằng secret key. Frontend gọi → server của cậu → server gọi Google bằng secret key → trả kết quả về. Vừa giấu key, vừa cache được kết quả (xem mục 15).

---

## 6. Maps JavaScript API

Nhúng bản đồ tương tác vào trang web. Đây là API "mặt tiền" — cái user nhìn thấy.

### 6.1. Bản đồ cơ bản — code chạy được ngay

Tạo file `map.html`, thay `YOUR_API_KEY` bằng key của cậu, mở bằng trình duyệt:

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Bản đồ Hà Nội</title>
  <style>
    /* Bản đồ phải có chiều cao cụ thể, không thì nó "tàng hình" */
    #map { height: 100vh; width: 100%; }
    body { margin: 0; }
  </style>
</head>
<body>
  <div id="map"></div>

  <script>
    // Google gọi hàm này sau khi thư viện tải xong (callback=initMap)
    function initMap() {
      // Tọa độ Hồ Gươm, Hà Nội
      const hoGuom = { lat: 21.0285, lng: 105.8542 };

      const map = new google.maps.Map(document.getElementById("map"), {
        center: hoGuom,   // tâm bản đồ
        zoom: 15,          // mức zoom: 1 (cả thế giới) → 20 (từng tòa nhà)
      });

      // Cắm 1 cái ghim
      new google.maps.Marker({
        position: hoGuom,
        map: map,
        title: "Hồ Gươm",
      });
    }
  </script>

  <!-- Nạp thư viện Maps JS: key + hàm callback + v=weekly (bản mới nhất) -->
  <script async
    src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap&v=weekly">
  </script>
</body>
</html>
```

**Giải thích tham số quan trọng:**
- `center`: `{ lat, lng }` — vĩ độ, kinh độ của tâm bản đồ.
- `zoom`: số càng lớn càng gần. 12 ≈ cả thành phố, 15 ≈ vài con phố, 18 ≈ từng tòa nhà.
- `callback=initMap`: Google tải xong thư viện sẽ tự gọi hàm `initMap()`. Hàm này **phải** tồn tại ở phạm vi global.
- `v=weekly`: dùng bản cập nhật hàng tuần. Muốn ổn định tuyệt đối thì dùng `v=quarterly`.

> 🐞 **Lỗi hay gặp:** mở file lên thấy trang trắng. 99% là do `#map` không có `height`, hoặc key chưa enable Maps JavaScript API, hoặc sai referrer restriction. Mở DevTools (F12) → tab Console để đọc lỗi.

### 6.2. Marker + Info Window (bong bóng thông tin)

```js
function initMap() {
  const map = new google.maps.Map(document.getElementById("map"), {
    center: { lat: 10.7756, lng: 106.7019 }, // Chợ Bến Thành, TP.HCM
    zoom: 14,
  });

  const places = [
    { name: "Chợ Bến Thành", lat: 10.7725, lng: 106.6983, desc: "Chợ nổi tiếng nhất Sài Gòn, mở từ 1914." },
    { name: "Nhà thờ Đức Bà", lat: 10.7798, lng: 106.6990, desc: "Nhà thờ cổ hơn 140 năm tuổi, đang trùng tu." },
    { name: "Dinh Độc Lập",  lat: 10.7772, lng: 106.6953, desc: "Di tích lịch sử quốc gia đặc biệt." },
  ];

  // 1 InfoWindow dùng chung — click marker nào thì hiện nội dung marker đó
  const infoWindow = new google.maps.InfoWindow();

  places.forEach((p) => {
    const marker = new google.maps.Marker({
      position: { lat: p.lat, lng: p.lng },
      map,
      title: p.name,
    });

    marker.addListener("click", () => {
      infoWindow.setContent(`<h3>${p.name}</h3><p>${p.desc}</p>`);
      infoWindow.open(map, marker);
    });
  });
}
```

### 6.3. Custom style — đổi màu bản đồ

Muốn bản đồ tối (dark mode) hay pastel? Truyền `styles` vào options. Style viết tay rất cực — dùng công cụ **Google Maps Platform Styling Wizard** (`mapstyle.withgoogle.com`) kéo thả rồi copy JSON:

```js
const map = new google.maps.Map(document.getElementById("map"), {
  center: { lat: 21.0285, lng: 105.8542 },
  zoom: 13,
  styles: [
    { elementType: "geometry", stylers: [{ color: "#1a1a2e" }] },          // nền tối
    { elementType: "labels.text.fill", stylers: [{ color: "#e0e0e0" }] }, // chữ sáng
    { featureType: "water", elementType: "geometry", stylers: [{ color: "#16213e" }] },
    { featureType: "road", elementType: "geometry", stylers: [{ color: "#2d4059" }] },
  ],
});
```

### 6.4. Sự kiện click — lấy tọa độ điểm user bấm

```js
map.addListener("click", (e) => {
  const lat = e.latLng.lat();
  const lng = e.latLng.lng();
  console.log("User bấm vào:", lat, lng);

  // Cắm marker tạm tại điểm vừa bấm
  new google.maps.Marker({ position: e.latLng, map });
});
```

Kết hợp với **Reverse Geocoding** (mục 7) là cậu có ngay tính năng "bấm vào bản đồ → hiện địa chỉ".

### 6.5. Các options hay dùng khác

```js
const map = new google.maps.Map(document.getElementById("map"), {
  center: { lat: 16.0471, lng: 108.2062 }, // Đà Nẵng
  zoom: 13,
  mapTypeId: "roadmap",      // roadmap | satellite | hybrid | terrain
  disableDefaultUI: false,   // true = ẩn hết nút zoom, fullscreen...
  zoomControl: true,         // chỉ hiện nút +/-
  streetViewControl: false,  // ẩn hình người Street View
  fullscreenControl: true,
  gestureHandling: "cooperative", // mobile: phải dùng 2 ngón mới zoom (đỡ bị kẹt scroll trang)
});
```

---


## 7. Geocoding API

**Geocoding** = địa chỉ (chữ) → tọa độ (số).
**Reverse geocoding** = tọa độ (số) → địa chỉ (chữ).

Endpoint: `https://maps.googleapis.com/maps/api/geocode/json`

### 7.1. Địa chỉ → tọa độ (Geocoding)

**Request bằng curl** (chạy trong terminal):

```bash
curl "https://maps.googleapis.com/maps/api/geocode/json?address=Ho+Gươm,+Hà+Nội&key=YOUR_API_KEY"
```

Nên thêm `region=vn` và `language=vi` để kết quả ưu tiên Việt Nam, trả về tiếng Việt:

```bash
curl "https://maps.googleapis.com/maps/api/geocode/json?address=Chợ+Bến+Thành,+TP.HCM&region=vn&language=vi&key=YOUR_API_KEY"
```

**Response JSON thật** (đã rút gọn):

```json
{
  "results": [
    {
      "formatted_address": "Chợ Bến Thành, Lê Lợi, Phường Bến Thành, Quận 1, Hồ Chí Minh, Việt Nam",
      "geometry": {
        "location": { "lat": 10.7724609, "lng": 106.6979816 },
        "location_type": "GEOMETRIC_CENTER",
        "viewport": {
          "northeast": { "lat": 10.7738099, "lng": 106.6993306 },
          "southwest": { "lat": 10.7711119, "lng": 106.6966326 }
        }
      },
      "place_id": "ChIJd9p6Q5YpdTER...",
      "types": ["point_of_interest", "establishment"]
    }
  ],
  "status": "OK"
}
```

**Đọc các field quan trọng:**

| Field | Nghĩa |
|-------|-------|
| `status` | `OK` = thành công. Luôn kiểm tra field này trước! |
| `results[0].geometry.location` | **Tọa độ** `{lat, lng}` — cái cậu cần |
| `results[0].formatted_address` | Địa chỉ đầy đủ dạng chuẩn của Google |
| `results[0].place_id` | ID duy nhất của địa điểm — dùng để gọi Place Details (mục 8) |
| `results[0].types` | Loại địa điểm (`street_address`, `locality`, `point_of_interest`...) |
| `geometry.location_type` | Độ chính xác: `ROOFTOP` (chính xác đến nóc nhà) > `GEOMETRIC_CENTER` > `APPROXIMATE` |

**Request bằng fetch() trong JS:**

```js
async function geocode(address) {
  const params = new URLSearchParams({
    address: address,
    region: "vn",
    language: "vi",
    key: "YOUR_API_KEY", // ⚠️ demo thôi — production gọi qua server của cậu!
  });

  const res = await fetch(
    `https://maps.googleapis.com/maps/api/geocode/json?${params}`
  );
  const data = await res.json();

  if (data.status !== "OK") {
    throw new Error(`Geocoding thất bại: ${data.status}`);
  }

  const best = data.results[0];
  return {
    lat: best.geometry.location.lat,
    lng: best.geometry.location.lng,
    address: best.formatted_address,
    placeId: best.place_id,
  };
}

// Dùng thử:
geocode("Phố cổ Hội An, Quảng Nam").then(console.log);
```

### 7.2. Tọa độ → địa chỉ (Reverse Geocoding)

```bash
curl "https://maps.googleapis.com/maps/api/geocode/json?latlng=21.0285,105.8542&language=vi&key=YOUR_API_KEY"
```

Response trả về mảng `results` — phần tử đầu tiên là địa chỉ **chi tiết nhất** (số nhà, đường), các phần tử sau càng lúc càng chung chung (phường → quận → thành phố → quốc gia).

```js
async function reverseGeocode(lat, lng) {
  const params = new URLSearchParams({
    latlng: `${lat},${lng}`,
    language: "vi",
    key: "YOUR_API_KEY",
  });
  const res = await fetch(
    `https://maps.googleapis.com/maps/api/geocode/json?${params}`
  );
  const data = await res.json();

  if (data.status !== "OK" || !data.results.length) {
    throw new Error(`Reverse geocoding thất bại: ${data.status}`);
  }
  return data.results[0].formatted_address;
}
```

### 7.3. Xử lý lỗi — bảng `status`

| `status` | Nghĩa | Xử lý |
|----------|-------|-------|
| `OK` | Thành công | Đọc `results` |
| `ZERO_RESULTS` | Không tìm thấy địa chỉ | **Không phải lỗi** — báo user nhập lại/sửa địa chỉ |
| `OVER_QUERY_LIMIT` | Hết quota | Giảm tần suất gọi, kiểm tra quota, bật billing |
| `REQUEST_DENIED` | Key sai / API chưa enable / restriction chặn | Kiểm tra key, enable API, xem lại referrer/IP restriction |
| `INVALID_REQUEST` | Thiếu tham số (quên `address` hoặc `latlng`) | Kiểm tra lại query string |
| `UNKNOWN_ERROR` | Lỗi phía Google | Thử lại sau vài giây (retry với backoff) |

> 💡 Mẹo: `ZERO_RESULTS` hay xảy ra với địa chỉ tiếng Việt viết tắt ("Q1", "P. Bến Thành"). Thử viết đầy đủ ("Quận 1", "Phường Bến Thành") hoặc thêm `region=vn`.

---


---

📄 *Tiếp tục đọc [Phần 2](huong-dan-google-maps-api-phan2.md): Places API, Directions, Distance Matrix, Pricing...*
