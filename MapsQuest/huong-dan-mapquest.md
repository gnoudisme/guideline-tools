# Hướng Dẫn Toàn Tập MapQuest — Bản Đồ & Định Vị

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 2026-10-10

---

## Mục Lục

1. [MapQuest Là Gì?](#1-mapquest-là-gì)
2. [Tạo Tài Khoản & Lấy API Key](#2-tạo-tài-khoản--lấy-api-key)
3. [Tổng Quan Các API](#3-tổng-quan-các-api)
4. [Geocoding API — Địa Chỉ ↔ Tọa Độ](#4-geocoding-api)
5. [Reverse Geocoding — Tọa Độ → Địa Chỉ](#5-reverse-geocoding)
6. [Directions API — Chỉ Đường](#6-directions-api)
7. [Route Matrix — Ma Trận Khoảng Cách](#7-route-matrix)
8. [Static Map API — Ảnh Bản Đồ](#8-static-map-api)
9. [JavaScript SDK — Bản Đồ Tương Tác](#9-javascript-sdk)
10. [Search API — Tìm Địa Điểm](#10-search-api)
11. [Traffic API — Giao Thông](#11-traffic-api)
12. [Bảng Giá](#12-bảng-giá)
13. [MapQuest vs Đối Thủ](#13-mapquest-vs-đối-thủ)
14. [Xử Lý Lỗi Thường Gặp](#14-xử-lý-lỗi-thường-gặp)
15. [Best Practices](#15-best-practices)
16. [Cheat Sheet](#16-cheat-sheet)

---

## 1. MapQuest Là Gì?

### 1.1 Định Nghĩa

**MapQuest** là một trong những nền tảng bản đồ trực tuyến lâu đời nhất (ra đời 1996, trước cả Google Maps!). Ngày nay, MapQuest cung cấp **bộ API bản đồ** cho developers:

- Chuyển địa chỉ thành tọa độ (geocoding)
- Chỉ đường, tính khoảng cách
- Hiển thị bản đồ trên web/app
- Dữ liệu giao thông

### 1.2 Điểm Đặc Biệt Của MapQuest

1. **Dùng dữ liệu OpenStreetMap:** bản đồ nền từ OSM (mở, miễn phí, cộng đồng cập nhật)
2. **Free tier rộng rãi:** 15.000 requests/tháng miễn phí
3. **Không cần thẻ tín dụng** để bắt đầu
4. **Đơn giản:** API REST dễ dùng, không phức tạp như Google Maps Platform

### 1.3 Ai Nên Dùng MapQuest?

- **Dự án nhỏ/vừa:** cần bản đồ mà không muốn trả tiền Google Maps
- **Học tập:** tìm hiểu về geocoding, routing API
- **App nội bộ:** dashboard theo dõi vị trí, quản lý giao hàng
- **Thay thế Google Maps:** khi muốn tránh phụ thuộc vào Google

### 1.4 Hạn Chế

- Dữ liệu bản đồ OSM có thể **kém chi tiết** hơn Google ở một số khu vực
- Ít tính năng nâng cao (không có Street View, Places chi tiết như Google)
- Cộng đồng và tài liệu nhỏ hơn Google Maps Platform

---

## 2. Tạo Tài Khoản & Lấy API Key

### 2.1 Đăng Ký

1. Vào **https://developer.mapquest.com**
2. Bấm **Sign Up** (góc phải trên)
3. Điền: tên, email, mật khẩu, công ty (tùy chọn)
4. Xác nhận email → đăng nhập

### 2.2 Tạo API Key

1. Sau khi đăng nhập → vào **My Applications** (hoặc Dashboard)
2. Bấm **+ Create a New Key**
3. Điền:
   - **App Name:** `ung-dung-cua-toi`
   - **Callback URL:** (tùy chọn, cho web app)
4. **Create** → copy key (dạng `AbCdEfGh12345678`)

### 2.3 Quản Lý Key

- Dashboard hiển thị: số requests đã dùng / giới hạn
- **Regenerate:** tạo key mới nếu key cũ bị lộ
- **Delete:** xóa key không dùng

> ⚠️ **Bảo mật:** Không để API key trong code public (GitHub public). Dùng biến môi trường!

```javascript
// ❌ SAI: hardcode key
const KEY = "AbCdEfGh12345678";

// ✅ ĐÚNG: dùng biến môi trường
const KEY = process.env.MAPQUEST_KEY;
```

---

## 3. Tổng Quan Các API

| API | Chức năng | Endpoint |
|---|---|---|
| **Geocoding** | Địa chỉ → tọa độ | `geocoding/v1/address` |
| **Reverse Geocoding** | Tọa độ → địa chỉ | `geocoding/v1/reverse` |
| **Directions** | Chỉ đường A → B | `directions/v2/route` |
| **Route Matrix** | Ma trận khoảng cách nhiều điểm | `directions/v2/routematrix` |
| **Static Map** | Ảnh bản đồ tĩnh | `staticmap/v5/map` |
| **Search** | Tìm địa điểm | `search/v4/place` |
| **Traffic** | Tình hình giao thông | `traffic/v2/incidents` |

**Base URL chung:** `https://www.mapquestapi.com/`

**Xác thực:** tất cả API đều cần `?key=YOUR_KEY`

---

## 4. Geocoding API — Địa Chỉ ↔ Tọa Độ

### 4.1 Geocoding Là Gì?

Chuyển **địa chỉ dạng chữ** thành **tọa độ** (vĩ độ, kinh độ).

```
"1 Tràng Tiền, Hà Nội"  →  { lat: 21.0245, lng: 105.8542 }
```

### 4.2 Request Cơ Bản

```bash
curl "https://www.mapquestapi.com/geocoding/v1/address?key=YOUR_KEY&location=1+Trang+Tien,Ha+Noi,Vietnam"
```

### 4.3 Response

```json
{
  "info": { "statuscode": 0 },
  "results": [
    {
      "locations": [
        {
          "street": "1 Tràng Tiền",
          "adminArea5": "Hà Nội",
          "adminArea1": "VN",
          "latLng": {
            "lat": 21.0245,
            "lng": 105.8542
          },
          "geocodeQuality": "ADDRESS",
          "geocodeQualityCode": "P1AAA"
        }
      ]
    }
  ]
}
```

### 4.4 Tham Số

| Tham số | Ý nghĩa | Ví dụ |
|---|---|---|
| `location` | Địa chỉ cần geocode | `Hà Nội, Vietnam` |
| `key` | API key | `AbCdEfGh...` |
| `maxResults` | Số kết quả tối đa | `5` |
| `thumbMaps` | Kèm ảnh thumbnail | `true` |

### 4.5 Ví Dụ JavaScript (Fetch)

```javascript
const KEY = 'YOUR_KEY';

async function geocode(address) {
  const url = `https://www.mapquestapi.com/geocoding/v1/address` +
    `?key=${KEY}&location=${encodeURIComponent(address)}`;

  const res = await fetch(url);
  const data = await res.json();

  const loc = data.results[0].locations[0];
  return {
    lat: loc.latLng.lat,
    lng: loc.latLng.lng,
    address: `${loc.street}, ${loc.adminArea5}, ${loc.adminArea1}`,
    quality: loc.geocodeQuality
  };
}

// Dùng
geocode('1 Tràng Tiền, Hà Nội, Vietnam')
  .then(r => console.log(r));
// → { lat: 21.0245, lng: 105.8542, ... }
```

### 4.6 Ví Dụ Python

```python
import requests

KEY = 'YOUR_KEY'

def geocode(address):
    url = 'https://www.mapquestapi.com/geocoding/v1/address'
    params = {'key': KEY, 'location': address}
    res = requests.get(url, params=params).json()
    loc = res['results'][0]['locations'][0]
    return {
        'lat': loc['latLng']['lat'],
        'lng': loc['latLng']['lng'],
        'quality': loc['geocodeQuality']
    }

print(geocode('1 Tràng Tiền, Hà Nội, Vietnam'))
```

### 4.7 Geocode Nhiều Địa Chỉ (Batch)

```bash
curl -X POST "https://www.mapquestapi.com/geocoding/v1/batch?key=YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "locations": [
      "Hà Nội, Vietnam",
      "TP.HCM, Vietnam",
      "Đà Nẵng, Vietnam"
    ]
  }'
```

> 💡 Batch tiết kiệm requests: 50 địa chỉ = 1 request (thay vì 50)!

### 4.8 Đánh Giá Chất Lượng (geocodeQuality)

| Giá trị | Ý nghĩa |
|---|---|
| `POINT` | Chính xác đến điểm |
| `ADDRESS` | Chính xác đến địa chỉ |
| `STREET` | Chỉ đến đường |
| `CITY` | Chỉ đến thành phố |
| `STATE` / `COUNTRY` | Mơ hồ — cần địa chỉ chi tiết hơn |

---

## 5. Reverse Geocoding — Tọa Độ → Địa Chỉ

### 5.1 Dùng Khi Nào?

- App có GPS → hiển thị "Bạn đang ở đâu" dạng chữ
- Chấm công: nhân viên check-in → ghi lại địa chỉ

### 5.2 Request

```bash
curl "https://www.mapquestapi.com/geocoding/v1/reverse?key=YOUR_KEY&location=21.0245,105.8542"
```

### 5.3 Response

```json
{
  "results": [
    {
      "locations": [
        {
          "street": "1 Tràng Tiền",
          "adminArea5": "Hoàn Kiếm",
          "adminArea4": "Hà Nội",
          "adminArea1": "VN",
          "postalCode": "100000"
        }
      ]
    }
  ]
}
```

### 5.4 Ví Dụ JavaScript

```javascript
async function reverseGeocode(lat, lng) {
  const url = `https://www.mapquestapi.com/geocoding/v1/reverse` +
    `?key=${KEY}&location=${lat},${lng}`;
  const data = await (await fetch(url)).json();
  const loc = data.results[0].locations[0];
  return `${loc.street}, ${loc.adminArea4}, ${loc.adminArea1}`;
}

reverseGeocode(21.0245, 105.8542)
  .then(addr => console.log(addr));
// → "1 Tràng Tiền, Hà Nội, VN"
```

---

## 6. Directions API — Chỉ Đường

### 6.1 Request Cơ Bản

```bash
curl "https://www.mapquestapi.com/directions/v2/route?key=YOUR_KEY&from=Ha+Noi,Vietnam&to=Hai+Phong,Vietnam"
```

Hoặc dùng tọa độ:

```bash
curl "https://www.mapquestapi.com/directions/v2/route?key=YOUR_KEY&from=21.0245,105.8542&to=20.8449,106.6881"
```

### 6.2 Response (Rút Gọn)

```json
{
  "route": {
    "distance": 119.5,
    "formattedTime": "02:15:30",
    "legs": [
      {
        "maneuvers": [
          { "narrative": "Đi thẳng 500m", "distance": 0.5 },
          { "narrative": "Rẽ phải vào đường Lê Lợi", "distance": 2.3 }
        ]
      }
    ],
    "options": {
      "routeType": "fastest"
    }
  }
}
```

### 6.3 Tham Số Quan Trọng

| Tham số | Giá trị | Ý nghĩa |
|---|---|---|
| `routeType` | `fastest` / `shortest` / `pedestrian` / `bicycle` | Loại đường |
| `unit` | `k` (km) / `m` (dặm) | Đơn vị khoảng cách |
| `avoids` | `Toll road`, `Highway`, `Ferry`... | Tránh loại đường |
| `locale` | `vi_VN` | Ngôn ngữ hướng dẫn |

### 6.4 Ví Dụ: Tính Phí Ship

```javascript
async function tinhPhiShip(fromLat, fromLng, toLat, toLng) {
  const url = `https://www.mapquestapi.com/directions/v2/route` +
    `?key=${KEY}&from=${fromLat},${fromLng}` +
    `&to=${toLat},${toLng}&unit=k&routeType=fastest`;

  const data = await (await fetch(url)).json();
  const km = data.route.distance;

  // Tính phí: 15k cho 3km đầu + 5k mỗi km tiếp theo
  const phi = km <= 3 ? 15000 : 15000 + Math.ceil(km - 3) * 5000;

  return {
    khoang_cach_km: km.toFixed(1),
    thoi_gian: data.route.formattedTime,
    phi_ship: phi
  };
}

// Kho HN → khách ở Hải Phòng
tinhPhiShip(21.0245, 105.8542, 20.8449, 106.6881)
  .then(console.log);
```

### 6.5 Nhiều Điểm Dừng (Waypoints)

```bash
curl "https://www.mapquestapi.com/directions/v2/route?key=YOUR_KEY&from=A&to=C&via=B1|B2"
```

Ví dụ shipper giao 3 đơn 1 chuyến:
```
Kho → Khách 1 → Khách 2 → Khách 3
```

### 6.6 Tối Ưu Thứ Tự (Route Optimization)

```bash
curl -X POST "https://www.mapquestapi.com/directions/v2/optimizedroute?key=YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "locations": ["Kho hàng", "Khách A", "Khách B", "Khách C"]
  }'
```

→ MapQuest tự sắp xếp thứ tự tối ưu nhất (tiết kiệm xăng!)

---

## 7. Route Matrix — Ma Trận Khoảng Cách

### 7.1 Dùng Khi Nào?

Cần biết khoảng cách từ **nhiều điểm** đến **nhiều điểm**:
- Tìm chi nhánh gần khách nhất
- Tính phí ship từ nhiều kho

### 7.2 Request

```bash
curl -X POST "https://www.mapquestapi.com/directions/v2/routematrix?key=YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "locations": ["Hà Nội", "Hải Phòng", "Đà Nẵng"],
    "options": { "allToAll": true, "unit": "k" }
  }'
```

### 7.3 Response

```json
{
  "distance": [0, 119.5, 764.2, 119.5, 0, 880.1, 764.2, 880.1, 0],
  "time": [0, 8100, 45000, 8100, 0, 50000, 45000, 50000, 0]
}
```

Đọc: `distance[1]` = 119.5km = khoảng cách từ điểm 0 (Hà Nội) → điểm 1 (Hải Phòng)

---

## 8. Static Map API — Ảnh Bản Đồ

### 8.1 Tạo Ảnh Bản Đồ Đơn Giản

```
https://www.mapquestapi.com/staticmap/v5/map?key=YOUR_KEY&center=HaNoi,VN&size=600,400&zoom=12
```

Mở URL trên trình duyệt → thấy ảnh bản đồ Hà Nội!

### 8.2 Thêm Ghim (Markers)

```
https://www.mapquestapi.com/staticmap/v5/map?key=YOUR_KEY
  &center=21.0245,105.8542
  &size=600,400&zoom=14
  &locations=21.0245,105.8542|marker-red
  &locations=21.0285,105.8542|marker-blue
```

### 8.3 Vẽ Đường Đi

```
https://www.mapquestapi.com/staticmap/v5/map?key=YOUR_KEY
  &size=600,400
  &route=21.0245,105.8542|20.8449,106.6881
```

### 8.4 Tham Số

| Tham số | Ý nghĩa |
|---|---|
| `center` | Tâm bản đồ |
| `size` | Kích thước ảnh (rộng,cao) |
| `zoom` | Mức zoom 0-20 |
| `type` | `map` / `sat` (vệ tinh) / `hyb` (kết hợp) |
| `locations` | Các điểm đánh dấu |
| `route` | Vẽ đường đi |

### 8.5 Dùng Trong HTML

```html
<!-- Hiện bản đồ tĩnh trong web -->
<img src="https://www.mapquestapi.com/staticmap/v5/map?key=YOUR_KEY&center=HaNoi,VN&size=600,400&zoom=12"
     alt="Bản đồ Hà Nội">
```

> 💡 Static map **nhẹ** hơn bản đồ tương tác nhiều — phù hợp cho email, PDF, trang cần load nhanh.

---

## 9. JavaScript SDK — Bản Đồ Tương Tác

### 9.1 Setup

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script src="https://www.mapquestapi.com/sdk/leaflet/v2.2/mq-map.js?key=YOUR_KEY"></script>
  <style>
    #map { height: 500px; width: 100%; }
  </style>
</head>
<body>
  <div id="map"></div>
  <script>
    // Tạo bản đồ tại Hà Nội
    const map = L.map('map').setView([21.0245, 105.8542], 13);

    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap'
    }).addTo(map);

    // Thêm ghim
    L.marker([21.0245, 105.8542])
      .addTo(map)
      .bindPopup('Hồ Gươm, Hà Nội')
      .openPopup();
  </script>
</body>
</html>
```

### 9.2 Thêm Chỉ Đường Lên Bản Đồ

```javascript
// Vẽ đường đi từ A đến B
MQ.routing.directions()
  .route({
    locations: [
      'Hà Nội, Vietnam',
      'Hải Phòng, Vietnam'
    ]
  });

// Hiển thị narrative (hướng dẫn từng bước)
const directionsLayer = MQ.routing.directionsLayer({
  directions: MQ.routing.directions(),
  fitBounds: true
});
map.addLayer(directionsLayer);
```

### 9.3 Tìm Kiếm Địa Điểm Trên Bản Đồ

```javascript
// Geocode + zoom đến
MQ.geocoding.geocode()
  .search('1 Tràng Tiền, Hà Nội')
  .on('success', function(e) {
    const latlng = e.result.best.latlng;
    map.setView(latlng, 16);
    L.marker(latlng).addTo(map);
  });
```

---

## 10. Search API — Tìm Địa Điểm

### 10.1 Tìm Quán Ăn Gần Đây

```bash
curl "https://www.mapquestapi.com/search/v4/place?key=YOUR_KEY&location=21.0245,105.8542&sort=distance&feedback=false&circle=21.0245,105.8542,1000&q=pho"
```

### 10.2 Tham Số

| Tham số | Ý nghĩa |
|---|---|
| `q` | Từ khóa tìm kiếm |
| `location` | Tọa độ trung tâm |
| `circle` | `lat,lng,bán_kính_m` |
| `sort` | `distance` / `relevance` |
| `limit` | Số kết quả (tối đa 50) |

### 10.3 Response

```json
{
  "results": [
    {
      "name": "Phở Thìn",
      "displayString": "Phở Thìn, 13 Lò Đúc, Hà Nội",
      "place": {
        "geometry": { "coordinates": [105.8542, 21.0245] }
      }
    }
  ]
}
```

---

## 11. Traffic API — Giao Thông

### 11.1 Xem Sự Cố Giao Thông

```bash
curl "https://www.mapquestapi.com/traffic/v2/incidents?key=YOUR_KEY&boundingBox=21.2,105.6,20.8,106.0&filters=construction,incidents"
```

### 11.2 Tham Số boundingBox

```
boundingBox = lat_trên,trái_dưới,lng_phải
```

Ví dụ Hà Nội: `21.2,105.6,20.8,106.0`

### 11.3 Response

```json
{
  "incidents": [
    {
      "type": "Construction",
      "severity": 2,
      "shortDesc": "Đang thi công đường",
      "lat": 21.0245,
      "lng": 105.8542,
      "startTime": "2026-10-01T08:00:00"
    }
  ]
}
```

---

## 12. Bảng Giá

> 💲 Kiểm tra tại developer.mapquest.com/plans

| Gói | Giá | Requests/tháng | Phù hợp |
|---|---|---|---|
| **Free** | $0 | 15.000 | Học tập, dự án nhỏ, thử nghiệm |
| **Basic** | ~$49 | 500.000 | App vừa, vài nghìn users |
| **Pro** | ~$199 | 2.000.000 | Sản phẩm thương mại |
| **Enterprise** | Liên hệ | Không giới hạn | SLA, hỗ trợ riêng |

### 12.1 Tính Requests

- 1 geocode = 1 request
- 1 directions = 1 request
- 1 batch 50 địa chỉ = 1 request (rẻ!)
- Static map mỗi lần load = 1 request

### 12.2 Mẹo Tiết Kiệm

1. **Cache kết quả:** địa chỉ đã geocode → lưu DB, lần sau không gọi lại
2. **Dùng batch:** gom 50 địa chỉ thành 1 request
3. **Static map thay vì interactive:** khi chỉ cần hiển thị
4. **Theo dõi usage:** dashboard hiện số requests đã dùng

---

## 13. MapQuest vs Đối Thủ

| Tiêu chí | **MapQuest** | Google Maps | Mapbox | OpenRouteService |
|---|---|---|---|---|
| Free tier | 15.000/tháng | $200 credit (~28.000 loads) | 50.000 loads | 2.000/ngày |
| Cần thẻ tín dụng | ❌ Không | ✅ Có | ❌ Không | ❌ Không |
| Dữ liệu bản đồ | OpenStreetMap | Google (riêng) | OSM + riêng | OSM |
| Độ chi tiết VN | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| Dễ dùng | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Tính năng nâng cao | Ít | Nhiều nhất | Nhiều | Trung bình |
| Phù hợp | Dự án nhỏ, học tập | Sản phẩm lớn | Bản đồ đẹp, tùy chỉnh | Hoàn toàn miễn phí |

**Kết luận:**
- **Miễn phí, đơn giản, không cần thẻ** → MapQuest
- **Cần tốt nhất, sẵn sàng trả tiền** → Google Maps
- **Bản đồ đẹp, tùy chỉnh cao** → Mapbox
- **Hoàn toàn miễn phí, open source** → OpenRouteService

---

## 14. Xử Lý Lỗi Thường Gặp

| Lỗi / Vấn đề | Nguyên nhân | Cách khắc phục |
|---|---|---|
| `403 Forbidden` | Key sai hoặc hết quota | Kiểm tra key, xem dashboard usage |
| `400 Bad Request` | Thiếu tham số bắt buộc | Kiểm tra `key`, `location`/`from`/`to` |
| Không tìm thấy địa chỉ VN | OSM thiếu dữ liệu | Thử địa chỉ tiếng Anh, hoặc tọa độ gần đúng |
| `OVER_QUERY_LIMIT` | Vượt giới hạn | Đợi reset tháng sau, nâng gói, hoặc cache |
| Static map trắng | Sai center/zoom | Kiểm tra tọa độ, zoom 0-20 |
| Directions không có đường | 2 điểm quá xa hoặc không có đường bộ | Kiểm tra lại tọa độ |
| Kết quả sai vị trí | Địa chỉ mơ hồ | Thêm chi tiết (quận, thành phố, quốc gia) |
| CORS error (web) | Gọi API trực tiếp từ browser | Dùng backend proxy, hoặc JSONP nếu hỗ trợ |

### 14.1 Debug Checklist

```
1. Key đúng chưa? → test bằng curl đơn giản
2. Còn quota không? → xem dashboard
3. Tham số đúng format? → location=lat,lng (không có khoảng trắng)
4. URL encode chưa? → dùng encodeURIComponent() cho địa chỉ có dấu
```

---

## 15. Best Practices

### 15.1 Hiệu Suất

1. **Cache mọi thứ:** geocode 1 lần, lưu DB, dùng lại
2. **Batch:** gom nhiều địa chỉ thành 1 request
3. **Debounce:** ô tìm kiếm → đợi user ngừng gõ 500ms mới gọi API
4. **Lazy load:** bản đồ chỉ load khi user cuộn tới

### 15.2 Bảo Mật

1. **Không để key trong frontend public** — dùng backend proxy:

```javascript
// Backend (Node.js) — key ở server, client không thấy
app.get('/api/geocode', async (req, res) => {
  const { address } = req.query;
  const url = `https://www.mapquestapi.com/geocoding/v1/address?key=${process.env.MQ_KEY}&location=${address}`;
  const data = await (await fetch(url)).json();
  res.json(data);
});
```

2. **Giới hạn referrer** (nếu MapQuest hỗ trợ cho key của bạn)
3. **Rotate key** định kỳ

### 15.3 UX

1. Hiển thị **loading** khi đang gọi API
2. Xử lý **không tìm thấy**: gợi ý địa chỉ gần đúng
3. **Fallback:** nếu MapQuest lỗi → thử Nominatim (OSM miễn phí)

---

## 16. Cheat Sheet

### Endpoints
```
Geocode:     GET  /geocoding/v1/address?key=K&location=...
Reverse:     GET  /geocoding/v1/reverse?key=K&location=lat,lng
Batch:       POST /geocoding/v1/batch?key=K
Directions:  GET  /directions/v2/route?key=K&from=A&to=B
Optimized:   POST /directions/v2/optimizedroute?key=K
Matrix:      POST /directions/v2/routematrix?key=K
Static map:  GET  /staticmap/v5/map?key=K&center=...&size=600,400&zoom=12
Search:      GET  /search/v4/place?key=K&location=lat,lng&q=...
Traffic:     GET  /traffic/v2/incidents?key=K&boundingBox=...
```

### Tính phí ship nhanh
```javascript
const km = (await route(from, to)).distance;
const phi = km <= 3 ? 15000 : 15000 + Math.ceil(km - 3) * 5000;
```

### Dashboard
```
developer.mapquest.com → My Applications → xem usage
```

---

*Cập nhật: 2026-10-10*

---

## 17. Ví Dụ Ứng Dụng Hoàn Chỉnh

### 17.1 Web Tìm Quán Ăn Gần Đây

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tìm quán ăn gần đây</title>
  <style>
    body { font-family: system-ui; max-width: 600px; margin: 20px auto; padding: 0 15px; }
    input, button { padding: 10px; font-size: 16px; margin: 5px 0; }
    input { width: 70%; }
    .quan { border: 1px solid #ddd; padding: 10px; margin: 10px 0; border-radius: 8px; }
    .quan h3 { margin: 0 0 5px 0; }
  </style>
</head>
<body>
  <h1>🍜 Tìm quán ăn gần bạn</h1>
  <input id="tukhoa" placeholder="Nhập món ăn: phở, bún chả..." value="phở">
  <button onclick="timKiem()">Tìm</button>
  <div id="ketqua"></div>

  <script>
    const KEY = 'YOUR_KEY'; // ⚠️ demo thôi, production dùng backend!

    // Lấy vị trí hiện tại
    function layViTri() {
      return new Promise((resolve, reject) => {
        navigator.geolocation.getCurrentPosition(
          pos => resolve([pos.coords.latitude, pos.coords.longitude]),
          reject
        );
      });
    }

    async function timKiem() {
      const tukhoa = document.getElementById('tukhoa').value;
      const [lat, lng] = await layViTri().catch(() => [21.0245, 105.8542]); // fallback HN

      const url = `https://www.mapquestapi.com/search/v4/place` +
        `?key=${KEY}&location=${lat},${lng}` +
        `&circle=${lat},${lng},2000` +
        `&q=${encodeURIComponent(tukhoa)}` +
        `&sort=distance&limit=10`;

      const data = await (await fetch(url)).json();
      const div = document.getElementById('ketqua');

      if (!data.results || data.results.length === 0) {
        div.innerHTML = '<p>Không tìm thấy quán nào 😢</p>';
        return;
      }

      div.innerHTML = data.results.map(q => `
        <div class="quan">
          <h3>${q.name}</h3>
          <p>📍 ${q.displayString}</p>
        </div>
      `).join('');
    }

    // Tự tìm khi mở trang
    timKiem();
  </script>
</body>
</html>
```

### 17.2 Dashboard Theo Dõi Shipper (Python + Flask)

```python
from flask import Flask, render_template_string, jsonify
import requests

app = Flask(__name__)
KEY = 'YOUR_KEY'

# Vị trí shipper (giả lập - thực tế lấy từ GPS)
shippers = [
    {'id': 1, 'name': 'Anh A', 'lat': 21.0245, 'lng': 105.8542},
    {'id': 2, 'name': 'Chị B', 'lat': 21.0285, 'lng': 105.8600},
]

HTML = '''
<!DOCTYPE html>
<html><head><title>Theo dõi shipper</title></head>
<body>
<h1>🛵 Vị trí shipper</h1>
<div id="list"></div>
<script>
async function load() {
  const data = await (await fetch('/api/shippers')).json();
  document.getElementById('list').innerHTML = data.map(s => `
    <div style="border:1px solid #ddd;padding:10px;margin:5px">
      <b>${s.name}</b><br>
      📍 ${s.address}<br>
      <img src="${s.map_url}" width="300">
    </div>`).join('');
}
load();
setInterval(load, 30000); // refresh mỗi 30s
</script>
</body></html>
'''

@app.route('/')
def index():
    return render_template_string(HTML)

@app.route('/api/shippers')
def api_shippers():
    result = []
    for s in shippers:
        # Reverse geocode để lấy địa chỉ
        geo = requests.get(
            'https://www.mapquestapi.com/geocoding/v1/reverse',
            params={'key': KEY, 'location': f"{s['lat']},{s['lng']}"}
        ).json()
        loc = geo['results'][0]['locations'][0]
        address = f"{loc.get('street','')}, {loc.get('adminArea5','')}"

        # Static map cho từng shipper
        map_url = (
            f"https://www.mapquestapi.com/staticmap/v5/map?key={KEY}"
            f"&center={s['lat']},{s['lng']}&size=300,200&zoom=15"
            f"&locations={s['lat']},{s['lng']}|marker-red"
        )
        result.append({**s, 'address': address, 'map_url': map_url})
    return jsonify(result)

if __name__ == '__main__':
    app.run(port=5000, debug=True)
```

### 17.3 Tính Phí Ship Theo Quận (E-commerce)

```python
import requests

KEY = 'YOUR_KEY'
KHO = (21.0245, 105.8542)  # Kho ở Hà Nội

# Bảng giá theo khoảng cách
def bang_gia(km):
    if km <= 3: return 15000
    if km <= 10: return 25000
    if km <= 30: return 40000
    return 60000

def phi_ship(dia_chi_khach):
    # 1. Geocode địa chỉ khách
    geo = requests.get(
        'https://www.mapquestapi.com/geocoding/v1/address',
        params={'key': KEY, 'location': dia_chi_khach}
    ).json()
    loc = geo['results'][0]['locations'][0]
    lat, lng = loc['latLng']['lat'], loc['latLng']['lng']

    # 2. Tính khoảng cách từ kho
    route = requests.get(
        'https://www.mapquestapi.com/directions/v2/route',
        params={
            'key': KEY,
            'from': f"{KHO[0]},{KHO[1]}",
            'to': f"{lat},{lng}",
            'unit': 'k'
        }
    ).json()

    km = route['route']['distance']
    return {
        'khoang_cach': round(km, 1),
        'thoi_gian': route['route']['formattedTime'],
        'phi_ship': bang_gia(km)
    }

# Test
print(phi_ship('123 Nguyễn Trãi, Thanh Xuân, Hà Nội'))
print(phi_ship('456 Lê Lợi, Hải Phòng'))
```

---

## 18. So Sánh Chi Tiết Với Nominatim (OSM Miễn Phí)

MapQuest dùng dữ liệu OSM. Có thể dùng **Nominatim** (API miễn phí của OSM) thay thế:

| | MapQuest | Nominatim |
|---|---|---|
| Giá | 15k free/tháng, sau đó trả phí | Hoàn toàn miễn phí |
| Giới hạn | Rõ ràng theo gói | 1 request/giây (nghiêm ngặt!) |
| Directions | ✅ Có | ❌ Không (dùng OpenRouteService) |
| Static map | ✅ Có | ❌ Không |
| Độ ổn định | Cao (thương mại) | Trung bình |
| Phù hợp | Sản phẩm thật | Thử nghiệm, dự án cá nhân |

**Ví dụ Nominatim:**
```bash
# Geocode miễn phí (không cần key!)
curl "https://nominatim.openstreetmap.org/search?q=Ha+Noi&format=json&limit=1" \
  -H "User-Agent: ung-dung-cua-toi"
```

> ⚠️ Nominatim **bắt buộc** gửi `User-Agent`, tối đa 1 request/giây. Dùng cho production sẽ bị chặn!

---

## 19. Tích Hợp Với Framework Phổ Biến

### 19.1 React

```jsx
import { useState } from 'react';

const KEY = process.env.REACT_APP_MQ_KEY;

function BanDoTinh() {
  const [diaChi, setDiaChi] = useState('');
  const [mapUrl, setMapUrl] = useState('');

  const xemBanDo = async () => {
    // Geocode trước
    const geo = await fetch(
      `https://www.mapquestapi.com/geocoding/v1/address?key=${KEY}&location=${encodeURIComponent(diaChi)}`
    ).then(r => r.json());

    const { lat, lng } = geo.results[0].locations[0].latLng;

    // Tạo static map URL
    setMapUrl(
      `https://www.mapquestapi.com/staticmap/v5/map?key=${KEY}` +
      `&center=${lat},${lng}&size=600,400&zoom=15` +
      `&locations=${lat},${lng}|marker-red`
    );
  };

  return (
    <div>
      <input value={diaChi} onChange={e => setDiaChi(e.target.value)}
             placeholder="Nhập địa chỉ..." />
      <button onClick={xemBanDo}>Xem bản đồ</button>
      {mapUrl && <img src={mapUrl} alt="Bản đồ" />}
    </div>
  );
}
```

### 19.2 Vue 3

```vue
<template>
  <div>
    <input v-model="diaChi" placeholder="Nhập địa chỉ..." />
    <button @click="xemBanDo">Xem bản đồ</button>
    <img v-if="mapUrl" :src="mapUrl" alt="Bản đồ" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
const KEY = import.meta.env.VITE_MQ_KEY;
const diaChi = ref('');
const mapUrl = ref('');

async function xemBanDo() {
  const geo = await fetch(
    `https://www.mapquestapi.com/geocoding/v1/address?key=${KEY}&location=${encodeURIComponent(diaChi.value)}`
  ).then(r => r.json());
  const { lat, lng } = geo.results[0].locations[0].latLng;
  mapUrl.value = `https://www.mapquestapi.com/staticmap/v5/map?key=${KEY}&center=${lat},${lng}&size=600,400&zoom=15&locations=${lat},${lng}|marker-red`;
}
</script>
```

---

## 20. Câu Hỏi Thường Gặp (FAQ)

**MapQuest có dùng được ở Việt Nam không?**
Có. Dữ liệu OSM ở các thành phố lớn VN (HN, HCM, Đà Nẵng) khá đầy đủ. Vùng sâu vùng xa có thể thiếu.

**Có cần thẻ tín dụng để dùng free không?**
Không. Đăng ký là dùng ngay 15.000 requests/tháng.

**15.000 requests đủ cho app nhỏ không?**
Đủ. Ví dụ: 500 users/ngày × 1 geocode = 15.000/tháng vừa khít. Nhớ cache!

**MapQuest có app mobile SDK không?**
Có hỗ trợ qua Leaflet (web). Native iOS/Android thì dùng Mapbox hoặc Google Maps SDK tốt hơn.

**Dữ liệu bản đồ cập nhật thường xuyên không?**
Dữ liệu OSM do cộng đồng cập nhật liên tục. MapQuest đồng bộ định kỳ.

**Có hỗ trợ tiếng Việt không?**
API trả về tên địa danh theo dữ liệu OSM (có tiếng Việt ở nhiều nơi). Tài liệu API chỉ có tiếng Anh.

---

*Cập nhật: 2026-10-10*

---

## 21. Xử Lý Lỗi Nâng Cao & Retry Logic

### 21.1 Retry Khi Gọi API (JavaScript)

```javascript
async function goiAPIvoiRetry(url, soLanThu = 3) {
  for (let i = 0; i < soLanThu; i++) {
    try {
      const res = await fetch(url);
      if (res.status === 429) {
        // Hết quota tạm thời → đợi rồi thử lại
        const doi = Math.pow(2, i) * 1000; // 1s, 2s, 4s
        console.log(`Bị giới hạn, đợi ${doi}ms rồi thử lại...`);
        await new Promise(r => setTimeout(r, doi));
        continue;
      }
      if (!res.ok) throw new Error(`HTTP ${res.status}`);
      return await res.json();
    } catch (e) {
      if (i === soLanThu - 1) throw e;
      console.log(`Lỗi, thử lại lần ${i + 2}...`);
    }
  }
}

// Dùng
const data = await goiAPIvoiRetry(
  `https://www.mapquestapi.com/geocoding/v1/address?key=${KEY}&location=HaNoi`
);
```

### 21.2 Retry (Python)

```python
import requests
import time

def goi_api_voi_retry(url, params, so_lan=3):
    for i in range(so_lan):
        try:
            res = requests.get(url, params=params, timeout=10)
            if res.status_code == 429:
                doi = (2 ** i) * 2
                print(f"Bị giới hạn, đợi {doi}s...")
                time.sleep(doi)
                continue
            res.raise_for_status()
            return res.json()
        except requests.RequestException as e:
            if i == so_lan - 1:
                raise
            print(f"Lỗi: {e}, thử lại...")
            time.sleep(2)
    return None
```

---

## 22. Cache Với Redis (Production)

```python
import redis
import json
import hashlib

# Kết nối Redis
r = redis.Redis(host='localhost', port=6379, decode_responses=True)

def geocode_co_cache(dia_chi):
    # Tạo key từ địa chỉ
    key = "geocode:" + hashlib.md5(dia_chi.encode()).hexdigest()

    # Kiểm tra cache
    cached = r.get(key)
    if cached:
        print("📂 Lấy từ cache")
        return json.loads(cached)

    # Gọi API
    print("🌐 Gọi MapQuest API")
    res = requests.get(
        'https://www.mapquestapi.com/geocoding/v1/address',
        params={'key': KEY, 'location': dia_chi}
    ).json()

    loc = res['results'][0]['locations'][0]
    ket_qua = {'lat': loc['latLng']['lat'], 'lng': loc['latLng']['lng']}

    # Lưu cache 30 ngày
    r.setex(key, 30 * 24 * 3600, json.dumps(ket_qua))
    return ket_qua

# Lần 1: gọi API
print(geocode_co_cache('1 Tràng Tiền, Hà Nội'))
# Lần 2: lấy từ cache (không tốn request!)
print(geocode_co_cache('1 Tràng Tiền, Hà Nội'))
```

---

## 23. Giám Sát & Cảnh Báo Quota

### 23.1 Kiểm Tra Usage Tự Động

```python
import requests

def kiem_tra_quota():
    # MapQuest không có API kiểm tra quota trực tiếp
    # Workaround: theo dõi số requests trong DB của bạn
    pass

# Tự đếm requests trong app của bạn
class DemRequests:
    def __init__(self):
        self.dem = 0
        self.gioi_han = 15000

    def goi_api(self, url, params):
        if self.dem >= self.gioi_han * 0.9:
            print("⚠️ CẢNH BÁO: đã dùng 90% quota!")
            # Gửi email/Slack báo cho admin
        self.dem += 1
        return requests.get(url, params=params).json()
```

### 23.2 Dashboard Đơn Giản

```python
from flask import Flask, jsonify
from datetime import datetime

app = Flask(__name__)
thong_ke = {'requests_hom_nay': 0, 'requests_thang_nay': 0}

@app.route('/admin/usage')
def usage():
    return jsonify({
        'hom_nay': thong_ke['requests_hom_nay'],
        'thang_nay': thong_ke['requests_thang_nay'],
        'gioi_han': 15000,
        'con_lai': 15000 - thong_ke['requests_thang_nay'],
        'phan_tram': round(thong_ke['requests_thang_nay'] / 15000 * 100, 1)
    })
```

---

## 24. So Sánh Chi Phí Thực Tế

### 24.1 Kịch Bản: App Giao Đồ Ăn Nhỏ

```
Giả định: 1.000 đơn/ngày, mỗi đơn cần:
  - 1 geocode (địa chỉ khách)
  - 1 directions (tính khoảng cách)
  = 2.000 requests/ngày = 60.000/tháng
```

| Nền tảng | Chi phí/tháng |
|---|---|
| MapQuest Free | ❌ Không đủ (15k < 60k) |
| MapQuest Basic (~$49) | ✅ Đủ (500k) |
| Google Maps ($200 credit) | ~$50-100 (tùy API) |
| Tự cache 80% | MapQuest Free có thể đủ! |

### 24.2 Khi Nào Nên Trả Phí?

```
Dùng free + cache tốt → đủ cho 90% dự án nhỏ
Trả phí khi:
  - Requests > 15k/tháng liên tục 3 tháng
  - Cần SLA đảm bảo uptime
  - Cần hỗ trợ kỹ thuật ưu tiên
```

---

*Cập nhật: 2026-10-10*

---

## 25. Bảo Mật Nâng Cao

### 25.1 Backend Proxy Pattern (Chi Tiết)

**Vấn đề:** Key trong JavaScript frontend → ai cũng xem được source → lấy key xài chùa.

**Giải pháp:** Tất cả gọi API qua backend của bạn.

```javascript
// Backend: server.js (Node.js + Express)
const express = require('express');
const fetch = require('node-fetch');
const app = express();

const MQ_KEY = process.env.MAPQUEST_KEY; // key chỉ ở server!

// Rate limiting: mỗi IP tối đa 100 requests/giờ
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({ windowMs: 60*60*1000, max: 100 });
app.use('/api/', limiter);

// Proxy geocode
app.get('/api/geocode', async (req, res) => {
  const { address } = req.query;
  if (!address) return res.status(400).json({ error: 'Thiếu address' });

  const url = `https://www.mapquestapi.com/geocoding/v1/address` +
    `?key=${MQ_KEY}&location=${encodeURIComponent(address)}`;
  const data = await (await fetch(url)).json();
  res.json(data);
});

// Proxy directions
app.get('/api/directions', async (req, res) => {
  const { from, to } = req.query;
  const url = `https://www.mapquestapi.com/directions/v2/route` +
    `?key=${MQ_KEY}&from=${encodeURIComponent(from)}` +
    `&to=${encodeURIComponent(to)}&unit=k`;
  const data = await (await fetch(url)).json();
  res.json(data);
});

app.listen(3000, () => console.log('Server chạy ở port 3000'));
```

```javascript
// Frontend: chỉ gọi backend của mình, không thấy key!
async function geocode(diaChi) {
  const res = await fetch(`/api/geocode?address=${encodeURIComponent(diaChi)}`);
  return res.json();
}
```

### 25.2 Validate Input

```javascript
// Chặn input độc hại trước khi gọi API
function diaChiHopLe(diaChi) {
  if (!diaChi || typeof diaChi !== 'string') return false;
  if (diaChi.length > 200) return false;           // quá dài
  if (/<script|javascript:/i.test(diaChi)) return false; // XSS
  return true;
}
```

---

## 26. Migration: Từ Google Maps Sang MapQuest

### 26.1 Bảng Tương Đương API

| Google Maps | MapQuest tương đương |
|---|---|
| Geocoding API | Geocoding API |
| Directions API | Directions API |
| Distance Matrix API | Route Matrix API |
| Maps JavaScript API | Leaflet + MQ SDK |
| Places API | Search API |
| Static Maps API | Static Map API |

### 26.2 Checklist Migration

```
[ ] Liệt kê tất cả chỗ dùng Google Maps API trong code
[ ] Đăng ký MapQuest, lấy key
[ ] Thay endpoint từng API (theo bảng trên)
[ ] Sửa format response (cấu trúc JSON khác nhau!)
[ ] Test kỹ từng tính năng
[ ] Chạy song song 2 tuần
[ ] Tắt Google Maps, hủy billing
```

### 26.3 Khác Biệt Response Format

```javascript
// Google: results[0].geometry.location.lat
// MapQuest: results[0].locations[0].latLng.lat

function chuanHoa(googleRes, mapquestRes, dungCaiNao) {
  if (dungCaiNao === 'google') {
    const loc = googleRes.results[0].geometry.location;
    return { lat: loc.lat, lng: loc.lng };
  } else {
    const loc = mapquestRes.results[0].locations[0].latLng;
    return { lat: loc.lat, lng: loc.lng };
  }
}
```

---

*Cập nhật: 2026-10-10*
