# Hướng Dẫn Toàn Tập: MapTiler — Bản Đồ Đẹp, SDK Dễ Dùng

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 10/10/2026
>
> ⚠️ **LƯU Ý VỀ KEY:** Mọi ví dụ dùng placeholder `YOUR_MAPTILER_KEY`. **Không** commit key thật lên Git!

---

## 📑 Mục Lục

1. [MapTiler Là Gì?](#1-maptiler-là-gì)
2. [MapTiler vs Mapbox vs Google Maps](#2-maptiler-vs-mapbox-vs-google-maps)
3. [Đăng Ký & Lấy API Key](#3-đăng-ký--lấy-api-key)
4. [Tổng Quan Các Sản Phẩm](#4-tổng-quan-các-sản-phẩm)
5. [MapTiler SDK JS — Bản Đồ Đầu Tiên](#5-maptiler-sdk-js--bản-đồ-đầu-tiên)
6. [Các Map Style Có Sẵn](#6-các-map-style-có-sẵn)
7. [Marker & Popup](#7-marker--popup)
8. [Geocoding API](#8-geocoding-api)
9. [Static Maps API — Ảnh Bản Đồ Tĩnh](#9-static-maps-api--ảnh-bản-đồ-tĩnh)
10. [Dùng Với MapLibre GL Thuần](#10-dùng-với-maplibre-gl-thuần)
11. [Dùng Với Leaflet](#11-dùng-với-leaflet)
12. [Ngôn Ngữ Bản Đồ (Tiếng Việt)](#12-ngôn-ngữ-bản-đồ-tiếng-việt)
13. [Tự Host (Self-hosting)](#13-tự-host-self-hosting)
14. [Pricing Chi Tiết](#14-pricing-chi-tiết)
15. [Bảo Mật API Key](#15-bảo-mật-api-key)
16. [Xử Lý Lỗi Thường Gặp](#16-xử-lý-lỗi-thường-gặp)
17. [Best Practices & Cheat Sheet](#17-best-practices--cheat-sheet)

---

## 1. MapTiler Là Gì?

**MapTiler** là công ty (Thụy Sĩ) cung cấp bản đồ nền (tiles) và API địa lý, xây dựng trên dữ liệu **OpenStreetMap**. Điểm đặc biệt:

- **SDK dựa trên MapLibre:** MapTiler SDK JS thực chất là MapLibre GL được "đóng gói" sẵn + tích hợp key — chuyển từ MapLibre thuần sang MapTiler chỉ vài dòng
- **Style đẹp sẵn:** Hàng chục style thiết kế chuyên nghiệp, xem trước trực quan
- **Không lock-in render:** Vì SDK là MapLibre, bạn có thể đổi provider bất cứ lúc nào
- **Tự host được:** Bán cả phần mềm/tile data để chạy trên server riêng

### 1.1. Các sản phẩm chính

| Sản phẩm | Mô tả |
|---|---|
| **MapTiler Cloud** | Host tiles + API (SaaS, tính theo request) |
| **MapTiler SDK JS** | Thư viện JS (dựa trên MapLibre) |
| **MapTiler Desktop** | App desktop tạo tiles từ dữ liệu riêng |
| **MapTiler Server** | Tự host tiles trên server của bạn |
| **Tiles a la carte** | Mua tile data 1 lần (không thuê bao) |

---

## 2. MapTiler vs Mapbox vs Google Maps

| Tiêu chí | MapTiler | Mapbox | Google Maps |
|---|---|---|---|
| Nền tảng dữ liệu | OpenStreetMap | Dữ liệu riêng + OSM | Dữ liệu riêng Google |
| SDK JS dựa trên | **MapLibre** (open-source) | Mapbox GL (BSL) | Proprietary |
| Free tier | **100.000 requests/tháng** | 50.000 map loads/tháng | $200 credit/tháng |
| Cần thẻ tín dụng (free) | ❌ Không | ❌ Không | ⚠️ Có (để vượt free) |
| Custom style | ✅ Customize tool | ✅ Studio mạnh nhất | Hạn chế |
| Geocoding | ✅ Có | ✅ Có | ✅ Mạnh nhất |
| Directions/Routing | ⚠️ Cơ bản | ✅ Mạnh | ✅ Mạnh nhất |
| Self-host | ✅ Có (Server/Desktop) | ❌ Không | ❌ Không |
| Giá khi scale | Từ ~$30/tháng | Từ ~$250/tháng (100k loads) | Tính theo SKU |
| Phù hợp | Muốn đẹp + rẻ + linh hoạt | Data viz cao cấp | Cần Places/Street View |

**Khi nào chọn MapTiler:**
- Muốn bản đồ **đẹp sẵn** mà không tự thiết kế
- Muốn **rẻ hơn Mapbox** khi scale
- Thích SDK dễ dùng nhưng vẫn muốn "cửa thoát" (vì là MapLibre bên trong)
- Có nhu cầu **tự host** sau này

---

## 3. Đăng Ký & Lấy API Key

### 3.1. Đăng ký

1. Vào https://cloud.maptiler.com/auth/signup/
2. Nhập email, password → **Sign up** (không cần thẻ)
3. Xác nhận email → đăng nhập vào **MapTiler Cloud**

### 3.2. Lấy API key

1. Vào https://cloud.maptiler.com/account/keys/
2. Key mặc định đã có sẵn → bấm **Copy**
3. Hoặc **Create new key** để tạo key riêng cho từng dự án

### 3.3. Quản lý key

- Đặt tên gợi nhớ: `web-ban-hang`, `app-mobile`
- **URL restrictions:** giới hạn domain được dùng key (khuyên dùng)
- **Xoay vòng:** nghi lộ → tạo key mới, xóa key cũ

```javascript
// Dùng key qua biến môi trường
const key = import.meta.env.VITE_MAPTILER_KEY;
```

---

## 4. Tổng Quan Các Sản Phẩm

### 4.1. URL pattern chung

Mọi endpoint MapTiler Cloud đều dùng `?key=API_KEY`:

```
https://api.maptiler.com/maps/{style}/{z}/{x}/{y}.png?key=KEY        → raster tiles
https://api.maptiler.com/maps/{style}/style.json?key=KEY             → vector style JSON
https://api.maptiler.com/geocoding/{query}.json?key=KEY              → geocoding
https://api.maptiler.com/geocoding/{lon},{lat}.json?key=KEY          → reverse geocoding
https://api.maptiler.com/geolocation/ip.json?key=KEY                 → IP geolocation
https://api.maptiler.com/maps/static/{lon},{lat},{zoom}/{w}x{h}.png?key=KEY → static map
```

### 4.2. Sơ đồ chọn sản phẩm

```
Cần bản đồ tương tác trên web?
├── Muốn nhanh nhất → MapTiler SDK JS (mục 5)
├── Đã dùng MapLibre → style.json URL (mục 10)
├── Đã dùng Leaflet → plugin @maptiler/leaflet-maptilersdk (mục 11)
└── Chỉ cần ảnh tĩnh → Static Maps API (mục 9)

Cần địa chỉ → tọa độ? → Geocoding API (mục 8)
Cần tiles tự host? → MapTiler Server (mục 13)
```

---

## 5. MapTiler SDK JS — Bản Đồ Đầu Tiên

MapTiler SDK là cách dễ nhất — đã tích hợp sẵn key, style, controls.

### 5.1. Cài đặt

```bash
npm install @maptiler/sdk
```

```html
<!-- Hoặc CDN -->
<link href="https://cdn.maptiler.com/maptiler-sdk-js/v2.3.0/maptiler-sdk.css" rel="stylesheet" />
<script src="https://cdn.maptiler.com/maptiler-sdk-js/v2.3.0/maptiler-sdk.umd.min.js"></script>
<link rel="stylesheet" href="https://cdn.maptiler.com/maptiler-sdk-js/v2.3.0/maptiler-sdk.css" />
```

### 5.2. Hello map

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>MapTiler Demo</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://cdn.maptiler.com/maptiler-sdk-js/v2.3.0/maptiler-sdk.umd.min.js"></script>
  <link href="https://cdn.maptiler.com/maptiler-sdk-js/v2.3.0/maptiler-sdk.css" rel="stylesheet" />
  <style>
    body { margin: 0; }
    #map { width: 100%; height: 100vh; }
  </style>
</head>
<body>
  <div id="map"></div>
  <script>
    maptilersdk.config.apiKey = 'YOUR_MAPTILER_KEY';

    const map = new maptilersdk.Map({
      container: 'map',
      style: maptilersdk.MapStyle.STREETS,
      center: [105.8542, 21.0285], // Hồ Gươm [lng, lat]
      zoom: 13
    });
  </script>
</body>
</html>
```

### 5.3. Với npm + React

```jsx
import { useEffect, useRef } from 'react';
import * as maptilersdk from '@maptiler/sdk';
import '@maptiler/sdk/dist/maptiler-sdk.css';

export default function Map() {
  const ref = useRef(null);

  useEffect(() => {
    maptilersdk.config.apiKey = import.meta.env.VITE_MAPTILER_KEY;
    const map = new maptilersdk.Map({
      container: ref.current,
      style: maptilersdk.MapStyle.STREETS,
      center: [105.8542, 21.0285],
      zoom: 13
    });
    return () => map.remove();
  }, []);

  return <div ref={ref} style={{ width: '100%', height: '500px' }} />;
}
```

---

## 6. Các Map Style Có Sẵn

```javascript
// Dùng enum có sẵn — không cần nhớ URL!
maptilersdk.MapStyle.STREETS        // đường phố chuẩn
maptilersdk.MapStyle.BASIC         // cơ bản, nhẹ
maptilersdk.MapStyle.BRIGHT        // sáng, hiện đại
maptilersdk.MapStyle.DATAVIZ       // tối, nền cho data visualization
maptilersdk.MapStyle.DATAVIZ.LIGHT // sáng, nền cho data viz
maptilersdk.MapStyle.DATAVIZ.DARK  // tối, nền cho data viz
maptilersdk.MapStyle.OUTDOOR       // địa hình, đi bộ đường dài
maptilersdk.MapStyle.SATELLITE     // ảnh vệ tinh
maptilersdk.MapStyle.HYBRID        // vệ tinh + nhãn
maptilersdk.MapStyle.TOPO          // bản đồ địa hình
maptilersdk.MapStyle.WINTER        // mùa đông ❄️
```

```javascript
// Đổi style lúc chạy
map.setStyle(maptilersdk.MapStyle.SATELLITE);
```

### 6.1. Xem trước tất cả style

Vào https://cloud.maptiler.com/maps/ → bấm từng style để xem trực quan, copy ID.

### 6.2. Style URL thủ công

```javascript
// Nếu không dùng SDK, dùng URL trực tiếp với MapLibre:
style: 'https://api.maptiler.com/maps/streets-v2/style.json?key=YOUR_MAPTILER_KEY'
```

---

## 7. Marker & Popup

Vì SDK dựa trên MapLibre nên API giống hệt:

```javascript
// Marker
const marker = new maptilersdk.Marker({ color: '#ff0000' })
  .setLngLat([105.8542, 21.0285])
  .addTo(map);

// Popup
const popup = new maptilersdk.Popup({ offset: 25 })
  .setHTML('<h3>Hồ Gươm</h3><p>Hà Nội, Việt Nam</p>');

marker.setPopup(popup);

// Marker kéo thả
const dragMarker = new maptilersdk.Marker({ draggable: true })
  .setLngLat([105.8355, 21.0285])
  .addTo(map);

dragMarker.on('dragend', () => {
  console.log(dragMarker.getLngLat());
});
```

---

## 8. Geocoding API

### 8.1. Forward geocoding

```javascript
async function geocode(query) {
  const url = `https://api.maptiler.com/geocoding/${encodeURIComponent(query)}.json?key=YOUR_MAPTILER_KEY&language=vi&limit=5`;
  const res = await fetch(url);
  const data = await res.json();
  return data.features.map(f => ({
    name: f.place_name,
    lng: f.center[0],
    lat: f.center[1]
  }));
}

const results = await geocode('Hồ Gươm');
console.log(results);
```

**Tham số:**

| Tham số | Ý nghĩa | Ví dụ |
|---|---|---|
| `language=vi` | Ngôn ngữ kết quả | `language=vi` |
| `limit=5` | Số kết quả | `limit=5` |
| `bbox=...` | Giới hạn vùng | `bbox=105.7,20.9,106.0,21.1` |
| `proximity=lng,lat` | Ưu tiên gần điểm | `proximity=105.85,21.02` |
| `types=...` | Loại: `address`, `poi`, `place` | `types=poi` |

### 8.2. Reverse geocoding

```javascript
async function reverseGeocode(lng, lat) {
  const url = `https://api.maptiler.com/geocoding/${lng},${lat}.json?key=YOUR_MAPTILER_KEY&language=vi`;
  const data = await (await fetch(url)).json();
  return data.features[0]?.place_name;
}

console.log(await reverseGeocode(105.8542, 21.0285));
```

### 8.3. IP Geolocation

```javascript
// Lấy vị trí gần đúng từ IP người dùng
const res = await fetch('https://api.maptiler.com/geolocation/ip.json?key=YOUR_MAPTILER_KEY');
const loc = await res.json();
console.log(loc.city, loc.country, loc.latitude, loc.longitude);
// → { city: 'Hanoi', country: 'Vietnam', ... }
```

### 8.4. Dùng SDK client (gọn hơn)

```javascript
import { geocoding } from '@maptiler/sdk';

const result = await geocoding.forward('Hồ Gươm', {
  apiKey: 'YOUR_MAPTILER_KEY',
  language: ['vi'],
  limit: 5
});
console.log(result.features);
```

---

## 9. Static Maps API — Ảnh Bản Đồ Tĩnh

Dùng cho: thumbnail, ảnh OG (chia sẻ mạng xã hội), email, PDF.

```
https://api.maptiler.com/maps/streets-v2/static/105.8542,21.0285,14/600x400.png?key=YOUR_MAPTILER_KEY&markers=105.8542,21.0285,red
```

**Tham số:**

| Tham số | Ý nghĩa |
|---|---|
| `static/{lng},{lat},{zoom}` | Tâm + zoom |
| `/{width}x{height}.png` | Kích thước ảnh |
| `&markers=lng,lat,color` | Ghim marker |
| `&path=...` | Vẽ đường đi |

```html
<!-- Dùng trong thẻ img -->
<img src="https://api.maptiler.com/maps/streets-v2/static/105.8542,21.0285,14/600x400.png?key=YOUR_MAPTILER_KEY" alt="Bản đồ Hồ Gươm" />
```

> ⚠️ Static Maps thường yêu cầu gói trả phí (không nằm trong free tier cơ bản) — kiểm tra pricing trước khi dùng nhiều.

---

## 10. Dùng Với MapLibre GL Thuần

Không thích SDK? Dùng MapLibre thuần + style URL của MapTiler:

```javascript
import maplibregl from 'maplibre-gl';

const map = new maplibregl.Map({
  container: 'map',
  style: 'https://api.maptiler.com/maps/streets-v2/style.json?key=YOUR_MAPTILER_KEY',
  center: [105.8542, 21.0285],
  zoom: 13
});
```

**Khi nào dùng cách này:**
- Đã có code MapLibre sẵn, chỉ muốn đổi tile provider
- Muốn kiểm soát hoàn toàn, không phụ thuộc SDK

---

## 11. Dùng Với Leaflet

```bash
npm install leaflet @maptiler/leaflet-maptilersdk
```

```javascript
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import { MaptilerLayer } from '@maptiler/leaflet-maptilersdk';

const map = L.map('map').setView([21.0285, 105.8542], 13); // Leaflet dùng [lat, lng]!

const mtLayer = new MaptilerLayer({
  apiKey: 'YOUR_MAPTILER_KEY',
  style: 'streets-v2'
}).addTo(map);

L.marker([21.0285, 105.8542])
  .bindPopup('Hồ Gươm')
  .addTo(map)
  .openPopup();
```

> ⚠️ Leaflet dùng `[lat, lng]` — ngược với MapLibre/Mapbox!

---

## 12. Ngôn Ngữ Bản Đồ (Tiếng Việt)

MapTiler hỗ trợ đa ngôn ngữ cho tên địa danh:

```javascript
// Với SDK
const map = new maptilersdk.Map({
  container: 'map',
  style: maptilersdk.MapStyle.STREETS,
  center: [105.85, 21.02],
  zoom: 12
});

// Đổi ngôn ngữ label sang tiếng Việt
map.setLanguage(maptilersdk.Language.VIETNAMESE);
```

```javascript
// Với style URL thuần, thêm tham số
'https://api.maptiler.com/maps/streets-v2/style.json?key=KEY'
// → style mặc định đã đa ngôn ngữ theo browser;
// để ép tiếng Việt, dùng SDK setLanguage như trên
```

**Các ngôn ngữ phổ biến:** `VIETNAMESE`, `ENGLISH`, `JAPANESE`, `KOREAN`, `CHINESE`, `FRENCH`, `GERMAN`, `SPANISH`, `AUTO` (theo trình duyệt).

---

## 13. Tự Host (Self-hosting)

### 13.1. Khi nào cần tự host?

- Yêu cầu dữ liệu không ra khỏi hạ tầng nội bộ
- Muốn trả phí 1 lần thay vì thuê bao
- Cần offline hoàn toàn

### 13.2. Các lựa chọn

| Sản phẩm | Mô tả | Giá |
|---|---|---|
| **MapTiler Server** | Server chạy tiles + hosting style | License theo server |
| **MapTiler Desktop** | App tạo tiles từ GeoTIFF, ảnh... | License theo user |
| **Tiles a la carte** | Mua tile data 1 lần | Theo khu vực/dung lượng |

### 13.3. Ví dụ chạy MapTiler Server bằng Docker

```bash
docker run -d \
  --name maptiler-server \
  -p 3650:3650 \
  -v /data/tiles:/data \
  maptiler/server:latest
# Mở http://localhost:3650
```

> 💡 Nếu chỉ cần self-host đơn giản, cân nhắc **Martin** (open-source, Rust) hoặc **tileserver-gl** — miễn phí.

---

## 14. Pricing Chi Tiết

### 14.1. Free tier

| Tài nguyên | Free/tháng |
|---|---|
| Tile requests | **100.000** |
| Map sessions | 5.000 |
| Storage | 100 MB |
| Thẻ tín dụng | ❌ Không cần |

> ⚠️ Free tier **không cho dùng thương mại** (non-commercial only).

### 14.2. Gói trả phí (tham khảo)

| Gói | Giá | Quota |
|---|---|---|
| **Flex** | ~$30/tháng | 500.000 requests, sau đó ~$0.15/1.000 |
| **Unlimited** | Liên hệ | Không giới hạn |

> ⚠️ Giá có thể thay đổi — kiểm tra https://www.maptiler.com/cloud/pricing/

### 14.3. Ví dụ tính

- Blog cá nhân 30.000 views/tháng → **$0** (trong free tier)
- Web thương mại 300.000 requests → cần gói Flex **~$30/tháng**

---

## 15. Bảo Mật API Key

```javascript
// ✅ .env
VITE_MAPTILER_KEY=abc123xyz

// .gitignore → thêm .env
```

1. **URL restrictions:** Vào dashboard → key settings → giới hạn domain
2. **Key riêng từng môi trường:** 1 key cho dev, 1 key cho production
3. **Không hardcode:** Luôn qua biến môi trường
4. **Theo dõi usage:** Dashboard → Statistics → phát hiện traffic lạ

---

## 16. Xử Lý Lỗi Thường Gặp

| Lỗi / Triệu chứng | Nguyên nhân | Cách fix |
|---|---|---|
| `401 Unauthorized` | Key sai / bị xóa | Kiểm tra key ở dashboard |
| `403 Forbidden` | Key bị giới hạn domain/IP | Thêm domain vào key settings |
| Hết quota (429) | Vượt 100k requests/tháng | Nâng cấp gói hoặc đợi reset |
| Bản đồ trắng | Sai style URL hoặc key | Thử mở style.json URL trên trình duyệt |
| `setStyle` mất layer | Đổi style xóa layer | Thêm lại trong `styledata` event |
| Label tiếng Anh | Chưa set language | `map.setLanguage(maptilersdk.Language.VIETNAMESE)` |
| SDK không load (CDN) | Sai version URL | Kiểm tra version mới nhất ở docs |

---

## 17. Best Practices & Cheat Sheet

```javascript
// Setup
import * as maptilersdk from '@maptiler/sdk';
maptilersdk.config.apiKey = import.meta.env.VITE_MAPTILER_KEY;

// Map
const map = new maptilersdk.Map({
  container: 'map',
  style: maptilersdk.MapStyle.STREETS,
  center: [105.8542, 21.0285], // [lng, lat]
  zoom: 13
});

// Đổi style / ngôn ngữ
map.setStyle(maptilersdk.MapStyle.SATELLITE);
map.setLanguage(maptilersdk.Language.VIETNAMESE);

// Marker + popup
new maptilersdk.Marker()
  .setLngLat([105.85, 21.02])
  .setPopup(new maptilersdk.Popup().setText('Xin chào!'))
  .addTo(map);

// Geocoding (SDK)
import { geocoding } from '@maptiler/sdk';
const r = await geocoding.forward('Hồ Gươm', { language: ['vi'] });

// Dọn dẹp
map.remove();
```

**Checklist:**
- [ ] Key qua biến môi trường, có URL restrictions
- [ ] Theo dõi quota hàng tháng
- [ ] Free tier chỉ dùng phi thương mại
- [ ] Set ngôn ngữ tiếng Việt nếu app cho người Việt
- [ ] `map.remove()` khi unmount

---

## 📚 Tài Liệu Tham Khảo

- Trang chủ: https://www.maptiler.com
- Cloud dashboard: https://cloud.maptiler.com
- SDK docs: https://docs.maptiler.com/sdk-js/
- Pricing: https://www.maptiler.com/cloud/pricing/
- GitHub: https://github.com/maptiler

---

## 18. MapTiler SDK — Tham Khảo Chi Tiết

### 18.1. Các class chính

| Class | Tương đương MapLibre | Mô tả |
|---|---|---|
| `maptilersdk.Map` | `maplibregl.Map` | Bản đồ chính |
| `maptilersdk.Marker` | `maplibregl.Marker` | Ghim |
| `maptilersdk.Popup` | `maplibregl.Popup` | Cửa sổ thông tin |
| `maptilersdk.NavigationControl` | — | Điều khiển zoom/xoay |
| `maptilersdk.GeolocateControl` | — | Vị trí người dùng |
| `maptilersdk.ScaleControl` | — | Thang tỷ lệ |
| `maptilersdk.FullscreenControl` | — | Toàn màn hình |

### 18.2. Config toàn cục

```javascript
import * as maptilersdk from '@maptiler/sdk';

// Bắt buộc: set key 1 lần
maptilersdk.config.apiKey = 'YOUR_KEY';

// Tùy chọn
maptilersdk.config.primaryLanguage = maptilersdk.Language.VIETNAMESE;
```

### 18.3. Events (giống MapLibre)

```javascript
map.on('load', () => console.log('Bản đồ sẵn sàng'));
map.on('click', (e) => console.log(e.lngLat));
map.on('moveend', () => { /* load dữ liệu mới */ });
```

---

## 19. Vẽ Dữ Liệu Nâng Cao Với SDK

Vì SDK là MapLibre bên trong, mọi API layer đều dùng được:

```javascript
map.on('load', () => {
  // GeoJSON: các chi nhánh cửa hàng
  map.addSource('stores', {
    type: 'geojson',
    data: '/api/stores.geojson', // hoặc object GeoJSON
    cluster: true,
    clusterMaxZoom: 14,
    clusterRadius: 50
  });

  map.addLayer({
    id: 'clusters',
    type: 'circle',
    source: 'stores',
    filter: ['has', 'point_count'],
    paint: {
      'circle-color': ['step', ['get', 'point_count'], '#51bbd6', 100, '#f1f075', 750, '#f28cb1'],
      'circle-radius': ['step', ['get', 'point_count'], 20, 100, 30, 750, 40]
    }
  });

  map.addLayer({
    id: 'cluster-count',
    type: 'symbol',
    source: 'stores',
    filter: ['has', 'point_count'],
    layout: { 'text-field': '{point_count_abbreviated}', 'text-size': 12 }
  });

  map.addLayer({
    id: 'unclustered-point',
    type: 'circle',
    source: 'stores',
    filter: ['!', ['has', 'point_count']],
    paint: { 'circle-color': '#11b4da', 'circle-radius': 8, 'circle-stroke-width': 1, 'circle-stroke-color': '#fff' }
  });

  // Click vào cluster để zoom
  map.on('click', 'clusters', async (e) => {
    const features = map.queryRenderedFeatures(e.point, { layers: ['clusters'] });
    const clusterId = features[0].properties.cluster_id;
    const source = map.getSource('stores');
    const zoom = await source.getClusterExpansionZoom(clusterId);
    map.easeTo({ center: features[0].geometry.coordinates, zoom });
  });
});
```

---

## 20. Geocoding Nâng Cao — Autocomplete UI Hoàn Chỉnh

```html
<div style="position:relative;max-width:400px;margin:10px;">
  <input id="geo-input" placeholder="Nhập địa chỉ..." style="width:100%;padding:10px;" />
  <div id="geo-results" style="position:absolute;width:100%;background:white;box-shadow:0 2px 8px rgba(0,0,0,.2);z-index:10;"></div>
</div>
<div id="map" style="height:90vh;"></div>

<script type="module">
import * as maptilersdk from 'https://cdn.maptiler.com/maptiler-sdk-js/v2.3.0/maptiler-sdk.module.js';
maptilersdk.config.apiKey = 'YOUR_MAPTILER_KEY';

const map = new maptilersdk.Map({
  container: 'map',
  style: maptilersdk.MapStyle.STREETS,
  center: [105.8542, 21.0285],
  zoom: 12
});

const input = document.getElementById('geo-input');
const results = document.getElementById('geo-results');
let timer, marker;

input.addEventListener('input', () => {
  clearTimeout(timer);
  timer = setTimeout(async () => {
    const q = input.value.trim();
    if (q.length < 3) { results.innerHTML = ''; return; }
    const r = await maptilersdk.geocoding.forward(q, {
      language: ['vi'], limit: 5,
      proximity: [105.8542, 21.0285], // ưu tiên gần Hà Nội
      types: ['address', 'poi']
    });
    results.innerHTML = '';
    r.features.forEach(f => {
      const div = document.createElement('div');
      div.style.cssText = 'padding:8px 10px;cursor:pointer;border-bottom:1px solid #eee;';
      div.textContent = f.place_name;
      div.onmouseenter = () => div.style.background = '#f5f5f5';
      div.onmouseleave = () => div.style.background = '';
      div.onclick = () => {
        const [lng, lat] = f.center;
        map.flyTo({ center: [lng, lat], zoom: 16 });
        if (marker) marker.remove();
        marker = new maptilersdk.Marker({ color: '#e11d48' })
          .setLngLat([lng, lat])
          .setPopup(new maptilersdk.Popup().setHTML(`<b>${f.place_name}</b>`))
          .addTo(map);
        results.innerHTML = '';
      };
      results.appendChild(div);
    });
  }, 300);
});
</script>
```

---

## 21. So Sánh Chi Tiết Các Style MapTiler

| Style | ID | Phù hợp |
|---|---|---|
| Streets v2 | `streets-v2` | App chung, đầy đủ chi tiết |
| Basic v2 | `basic-v2` | Nhẹ, load nhanh |
| Bright v2 | `bright-v2` | Hiện đại, màu tươi |
| Dataviz | `dataviz` | Nền tối cho dashboard dữ liệu |
| Dataviz Light | `dataviz-light` | Nền sáng cho dashboard |
| Outdoor v2 | `outdoor-v2` | Leo núi, đường mòn, đường đồng mức |
| Satellite | `satellite` | Ảnh vệ tinh thuần |
| Hybrid | `hybrid` | Vệ tinh + nhãn đường |
| Topo v2 | `topo-v2` | Địa hình chi tiết |
| Winter v2 | `winter-v2` | Phong cách mùa đông |

```javascript
// Xem trực quan: https://cloud.maptiler.com/maps/
map.setStyle(maptilersdk.MapStyle.OUTDOOR);
```

---

## 22. Câu Hỏi Thường Gặp (FAQ)

**Hỏi: MapTiler SDK và MapLibre GL khác nhau thế nào?**
Đáp: SDK = MapLibre GL + tích hợp sẵn key/style/helpers của MapTiler. Dùng SDK thì code ngắn hơn; dùng MapLibre thuần thì linh hoạt hơn (đổi provider dễ).

**Hỏi: Free tier 100k requests tính thế nào?**
Đáp: Mỗi tile request, geocoding request đều tính. 1 lần load bản đồ thường tốn 10-30 tile requests. 100k ≈ 3.000-10.000 lượt xem bản đồ/tháng.

**Hỏi: Có thể dùng MapTiler mà không lộ key ở frontend?**
Đáp: Key MapTiler bắt buộc ở client cho tiles. Giảm rủi ro bằng URL restrictions. Với geocoding nhạy cảm, gọi qua backend proxy.

**Hỏi: Muốn bản đồ tiếng Việt hoàn toàn?**
Đáp: `map.setLanguage(maptilersdk.Language.VIETNAMESE)` — tên địa danh có bản dịch tiếng Việt sẽ hiện tiếng Việt.

**Hỏi: Chuyển từ Google Maps sang MapTiler có khó không?**
Đáp: Logic tương tự (map, marker, geocoding). Khác chính: tọa độ `[lng, lat]`, style URL thay vì API key Google, không có Places API mạnh bằng.
