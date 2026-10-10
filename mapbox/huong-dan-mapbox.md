# Hướng Dẫn Toàn Tập: Mapbox — Bản Đồ Vector Tương Tác

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 10/10/2026
>
> ⚠️ **LƯU Ý VỀ TOKEN:** Mọi ví dụ dùng placeholder `YOUR_MAPBOX_TOKEN`. **Tuyệt đối KHÔNG** commit token thật lên GitHub — lộ token là người khác xài chùa quota của bạn!

---

## 📑 Mục Lục

1. [Mapbox Là Gì?](#1-mapbox-là-gì)
2. [So Sánh Mapbox Với Các Đối Thủ](#2-so-sánh-mapbox-với-các-đối-thủ)
3. [Đăng Ký Tài Khoản & Lấy Access Token](#3-đăng-ký-tài-khoản--lấy-access-token)
4. [Hiểu Về Token: Public vs Secret](#4-hiểu-về-token-public-vs-secret)
5. [Cài Đặt Mapbox GL JS](#5-cài-đặt-mapbox-gl-js)
6. [Bản Đồ Đầu Tiên — Hello Map](#6-bản-đồ-đầu-tiên--hello-map)
7. [Tọa Độ: Cái Bẫy [lng, lat]](#7-tọa-độ-cái-bẫy-lng-lat)
8. [Điều Khiển Bản Đồ (Controls)](#8-điều-khiển-bản-đồ-controls)
9. [Marker — Ghim Địa Điểm](#9-marker--ghim-địa-điểm)
10. [Popup — Cửa Sổ Thông Tin](#10-popup--cửa-sổ-thông-tin)
11. [Styles — Thay Áo Cho Bản Đồ](#11-styles--thay-áo-cho-bản-đồ)
12. [Layers — Vẽ Dữ Liệu Lên Bản Đồ](#12-layers--vẽ-dữ-liệu-lên-bản-đồ)
13. [Geocoding API — Địa Chỉ ↔ Tọa Độ](#13-geocoding-api--địa-chỉ--tọa-độ)
14. [Directions API — Chỉ Đường](#14-directions-api--chỉ-đường)
15. [Mapbox Studio — Tự Thiết Kế Style](#15-mapbox-studio--tự-thiết-kế-style)
16. [Tối Ưu Hiệu Suất](#16-tối-ưu-hiệu-suất)
17. [Pricing & Free Tier Chi Tiết](#17-pricing--free-tier-chi-tiết)
18. [Bảo Mật Token](#18-bảo-mật-token)
19. [Xử Lý Lỗi Thường Gặp](#19-xử-lý-lỗi-thường-gặp)
20. [Best Practices & Cheat Sheet](#20-best-practices--cheat-sheet)

---

## 1. Mapbox Là Gì?

**Mapbox** là nền tảng bản đồ dành cho developer, cung cấp:

| Thành phần | Mô tả |
|---|---|
| **Mapbox GL JS** | Thư viện JavaScript vẽ bản đồ vector tương tác trên web (WebGL) |
| **Mapbox Studio** | Công cụ thiết kế style bản đồ kéo-thả trực quan |
| **Tilesets** | Dữ liệu bản đồ vector (đường, nhà, địa danh...) do Mapbox host |
| **Geocoding API** | Chuyển địa chỉ ↔ tọa độ |
| **Directions API** | Tính lộ trình, chỉ đường turn-by-turn |
| **Static Images API** | Xuất ảnh bản đồ tĩnh (thumbnail, OG image) |

### 1.1. Vì sao developer thích Mapbox?

- **Vector tiles + WebGL:** Zoom mượt mà, chữ không vỡ, render bằng GPU
- **Tùy biến sâu:** Đổi màu từng loại đường, từng tòa nhà qua style
- **Hiệu suất:** Vẽ 10.000+ điểm vẫn mượt (dùng layer thay vì DOM marker)
- **3D sẵn có:** Tòa nhà 3D, địa hình terrain, chế độ globe

### 1.2. Khi nào KHÔNG nên dùng Mapbox?

- Cần **Street View** → Google Maps thắng
- App thương mại đặc thù (xe cộ, bất động sản, BI...) → kiểm tra lại license
- Muốn hoàn toàn miễn phí, không đăng ký → xem OpenFreeMap + MapLibre GL

---

## 2. So Sánh Mapbox Với Các Đối Thủ

| Tiêu chí | Mapbox | Google Maps | MapLibre + OpenFreeMap |
|---|---|---|---|
| License thư viện | BSL (không fully open-source từ v2) | Proprietary | BSD-3 (open source thật) |
| Cần token/key | ✅ Có (pk.) | ✅ Có | ❌ Không |
| Free tier (map loads) | 50.000/tháng | ~$200 credit/tháng | Không giới hạn |
| Custom style | ✅ Mapbox Studio rất mạnh | Hạn chế | ✅ Tự host style JSON |
| 3D buildings/terrain | ✅ Có sẵn | Có (giới hạn) | ✅ Có (tự cấu hình) |
| Street View | ❌ Không | ✅ Có | ❌ Không |
| Places API | Cơ bản | ✅ Rất mạnh | Dùng Nominatim (miễn phí) |
| Học ban đầu | Trung bình | Dễ | Trung bình |
| Phù hợp | App cần style đẹp, data viz | App cần Places/Street View | App open-source, tiết kiệm chi phí |

**Quy tắc chọn nhanh:**
- Muốn đẹp + mạnh + sẵn sàng trả phí khi scale → **Mapbox**
- Cần Places/Street View, team quen Google → **Google Maps**
- Muốn free hoàn toàn, tự chủ → **MapLibre GL + OpenFreeMap**

---

## 3. Đăng Ký Tài Khoản & Lấy Access Token

### 3.1. Đăng ký (không cần thẻ tín dụng cho free tier)

1. Vào https://account.mapbox.com/auth/signup/
2. Nhập email, username, password → **Sign up**
3. Xác nhận email
4. Đăng nhập → vào Dashboard

### 3.2. Lấy token mặc định

1. Vào https://account.mapbox.com/access-tokens/
2. Token **Default public token** đã có sẵn (bắt đầu bằng `pk.`)
3. Bấm vào để copy

### 3.3. Tạo token mới với quyền hạn chế

1. Bấm **Create a token**
2. Đặt tên gợi nhớ: ví dụ `web-du-an-ban-do`
3. Chọn **scopes** (quyền):
   - ✅ `styles:read` — đọc style bản đồ (bắt buộc)
   - ✅ `fonts:read` — đọc font chữ trên bản đồ (bắt buộc)
   - ⬜ `datasets:read` — chỉ tick nếu dùng Datasets API
   - ⬜ `geocoding:read` — nếu gọi Geocoding từ client
4. **URL restrictions** (khuyên dùng):
   - Thêm `https://ten-mien-cua-ban.com/*`
   - Thêm `http://localhost:3000/*` để dev local
5. Bấm **Create token** → copy ngay (chỉ hiện 1 lần!)

### 3.4. Xoay vòng token khi bị lộ

Token public bị lộ trong source JS là bình thường (nó nằm ở client), nhưng nếu thấy traffic lạ:
1. Vào trang tokens → **Delete** token cũ
2. Tạo token mới với URL restrictions chặt hơn
3. Cập nhật biến môi trường

---

## 4. Hiểu Về Token: Public vs Secret

| Loại | Tiền tố | Dùng ở đâu | Quyền |
|---|---|---|---|
| **Public token** | `pk.` | Trình duyệt (client-side) | Chỉ đọc: styles, fonts, tiles, geocoding tạm thời |
| **Secret token** | `sk.` | Server (backend) | Full quyền: ghi, xóa, Directions, Matrix... |

> 🔴 **NGUYÊN TẮC VÀNG:** Không bao giờ để `sk.` lộ ra frontend, không commit vào Git. Secret token chỉ sống trong biến môi trường server.

```javascript
// ✅ ĐÚNG — public token ở client (qua biến môi trường)
mapboxgl.accessToken = process.env.NEXT_PUBLIC_MAPBOX_TOKEN; // pk.ey...

// ❌ SAI — secret token ở client
mapboxgl.accessToken = 'sk.ey...'; // LỘ LÀ TOANG
```

**Backend gọi API cần secret:**

```javascript
// Node.js — gọi Directions API từ server
const token = process.env.MAPBOX_SECRET_TOKEN; // sk.ey...
const url = `https://api.mapbox.com/directions/v5/mapbox/driving/105.85,21.02;105.86,21.03?access_token=${token}&geometries=geojson`;
const res = await fetch(url);
const data = await res.json();
console.log(data.routes[0].distance, 'mét');
```

---

## 5. Cài Đặt Mapbox GL JS

### 5.1. Qua CDN (nhanh nhất để thử)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Mapbox Demo</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://api.mapbox.com/mapbox-gl-js/v3.6.0/mapbox-gl.js"></script>
  <link href="https://api.mapbox.com/mapbox-gl-js/v3.6.0/mapbox-gl.css" rel="stylesheet" />
  <style>
    body { margin: 0; }
    #map { width: 100%; height: 100vh; }
  </style>
</head>
<body>
  <div id="map"></div>
  <script>
    mapboxgl.accessToken = 'YOUR_MAPBOX_TOKEN';
    const map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v12',
      center: [105.8542, 21.0285], // Hồ Gươm, Hà Nội
      zoom: 13
    });
  </script>
</body>
</html>
```

### 5.2. Qua npm (dự án thật)

```bash
npm install mapbox-gl
```

```javascript
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

mapboxgl.accessToken = import.meta.env.VITE_MAPBOX_TOKEN;

const map = new mapboxgl.Map({
  container: document.getElementById('map'),
  style: 'mapbox://styles/mapbox/streets-v12',
  center: [105.8542, 21.0285],
  zoom: 13
});
```

### 5.3. React (ví dụ với Vite)

```bash
npm install mapbox-gl
```

```jsx
// Map.jsx
import { useEffect, useRef } from 'react';
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

export default function Map() {
  const mapRef = useRef(null);

  useEffect(() => {
    mapboxgl.accessToken = import.meta.env.VITE_MAPBOX_TOKEN;
    const map = new mapboxgl.Map({
      container: mapRef.current,
      style: 'mapbox://styles/mapbox/streets-v12',
      center: [105.8542, 21.0285],
      zoom: 13
    });
    map.addControl(new mapboxgl.NavigationControl());

    return () => map.remove(); // dọn dẹp khi unmount
  }, []);

  return <div ref={mapRef} style={{ width: '100%', height: '500px' }} />;
}
```

```
// .env
VITE_MAPBOX_TOKEN=pk.ey...
```

---

## 6. Bản Đồ Đầu Tiên — Hello Map

```javascript
mapboxgl.accessToken = 'YOUR_MAPBOX_TOKEN';

const map = new mapboxgl.Map({
  container: 'map',                        // id của div
  style: 'mapbox://styles/mapbox/streets-v12', // style
  center: [105.8542, 21.0285],             // [kinh độ, vĩ độ]
  zoom: 13,                                // 0 (cả thế giới) → 22 (từng tòa nhà)
  pitch: 0,                                // nghiêng 3D (độ)
  bearing: 0                               // xoay bản đồ (độ)
});
```

**Các tham số quan trọng:**

| Tham số | Ý nghĩa | Ví dụ |
|---|---|---|
| `container` | id div chứa bản đồ | `'map'` |
| `style` | style bản đồ | `'mapbox://styles/mapbox/streets-v12'` |
| `center` | tâm bản đồ `[lng, lat]` | `[105.8542, 21.0285]` |
| `zoom` | mức zoom | `13` |
| `minZoom` / `maxZoom` | giới hạn zoom | `3` / `18` |
| `pitch` | góc nghiêng 3D | `60` |
| `bearing` | góc xoay | `45` |
| `projection` | phép chiếu | `'globe'` (quả địa cầu) |

**Chế độ quả địa cầu (globe):**

```javascript
const map = new mapboxgl.Map({
  container: 'map',
  style: 'mapbox://styles/mapbox/satellite-streets-v12',
  projection: 'globe', // 🌍
  zoom: 2,
  center: [105.85, 21.02]
});
```

---

## 7. Tọa Độ: Cái Bẫy [lng, lat]

> ⚠️ **Mapbox dùng `[kinh độ, vĩ độ]`** (chuẩn GeoJSON). Google Maps/Leaflet dùng `[vĩ độ, kinh độ]`. Nhầm là bản đồ bay sang chỗ khác!

```javascript
// ✅ ĐÚNG — Hồ Gươm: kinh độ 105.85, vĩ độ 21.02
map.setCenter([105.8542, 21.0285]);

// ❌ SAI — đảo ngược sẽ bay ra biển
map.setCenter([21.0285, 105.8542]);
```

**Mẹo nhớ:** "lng trước, lat sau" — theo alphabet l < ... à không, theo chuẩn GeoJSON: x (kinh độ) trước, y (vĩ độ) sau.

**Chuyển đổi nhanh:**

```javascript
// Từ Google Maps {lat, lng} sang Mapbox [lng, lat]
const gmaps = { lat: 21.0285, lng: 105.8542 };
const mapbox = [gmaps.lng, gmaps.lat];
```

---

## 8. Điều Khiển Bản Đồ (Controls)

```javascript
// Zoom + xoay + la bàn
map.addControl(new mapboxgl.NavigationControl(), 'top-right');

// Tỷ lệ bản đồ
map.addControl(new mapboxgl.ScaleControl({ maxWidth: 100 }), 'bottom-left');

// Vị trí của tôi (đòi quyền geolocation trình duyệt)
map.addControl(new mapboxgl.GeolocateControl({
  positionOptions: { enableHighAccuracy: true },
  trackUserLocation: true
}), 'top-right');

// Fullscreen
map.addControl(new mapboxgl.FullscreenControl(), 'top-right');
```

**Vị trí đặt:** `'top-left'`, `'top-right'`, `'bottom-left'`, `'bottom-right'`.

---

## 9. Marker — Ghim Địa Điểm

### 9.1. Marker mặc định

```javascript
const marker = new mapboxgl.Marker()
  .setLngLat([105.8542, 21.0285])
  .addTo(map);
```

### 9.2. Marker màu tùy chỉnh

```javascript
new mapboxgl.Marker({ color: '#ff0000', scale: 1.2 })
  .setLngLat([105.8542, 21.0285])
  .addTo(map);
```

### 9.3. Marker HTML tùy chỉnh (đẹp nhất)

```javascript
// Tạo element HTML
const el = document.createElement('div');
el.className = 'custom-marker';
el.innerHTML = '📍';
el.style.fontSize = '32px';
el.style.cursor = 'pointer';

new mapboxgl.Marker(el)
  .setLngLat([105.8542, 21.0285])
  .addTo(map);
```

### 9.4. Nhiều marker từ danh sách

```javascript
const places = [
  { name: 'Hồ Gươm', lng: 105.8542, lat: 21.0285 },
  { name: 'Văn Miếu', lng: 105.8355, lat: 21.0285 },
  { name: 'Lăng Bác', lng: 105.8342, lat: 21.0378 }
];

places.forEach(p => {
  new mapboxgl.Marker({ color: '#3b82f6' })
    .setLngLat([p.lng, p.lat])
    .setPopup(new mapboxgl.Popup().setText(p.name))
    .addTo(map);
});
```

> 💡 **Hiệu suất:** Dưới 100 marker dùng `Marker` thoải mái. Trên 500 marker → dùng **Symbol layer** (mục 12) nhanh hơn 10-100 lần vì render bằng WebGL thay vì DOM.

### 9.5. Marker kéo thả được

```javascript
const marker = new mapboxgl.Marker({ draggable: true })
  .setLngLat([105.8542, 21.0285])
  .addTo(map);

marker.on('dragend', () => {
  const { lng, lat } = marker.getLngLat();
  console.log('Vị trí mới:', lng, lat);
});
```

---

## 10. Popup — Cửa Sổ Thông Tin

```javascript
// Popup đơn giản
new mapboxgl.Popup({ closeOnClick: true })
  .setLngLat([105.8542, 21.0285])
  .setHTML('<h3>Hồ Gươm</h3><p>Trái tim của Hà Nội ❤️</p>')
  .addTo(map);
```

```javascript
// Popup gắn với marker
const popup = new mapboxgl.Popup({ offset: 25 })
  .setHTML(`
    <div style="font-family: sans-serif;">
      <h3 style="margin: 0;">Phở Bò Tái Chín</h3>
      <p>⭐ 4.8 · 💰 45k</p>
      <button onclick="alert('Đặt bàn!')">Đặt bàn</button>
    </div>
  `);

new mapboxgl.Marker()
  .setLngLat([105.8542, 21.0285])
  .setPopup(popup)
  .addTo(map);
```

**Tùy chọn Popup:**

| Option | Ý nghĩa |
|---|---|
| `closeButton: false` | Ẩn nút X |
| `closeOnClick: false` | Không đóng khi click ra ngoài |
| `offset: 25` | Dịch popup khỏi marker (px) |
| `maxWidth: '300px'` | Giới hạn chiều rộng |
| `anchor: 'bottom'` | Neo popup ở hướng nào |

---

## 11. Styles — Thay Áo Cho Bản Đồ

### 11.1. Style có sẵn của Mapbox

| Style ID | Mô tả |
|---|---|
| `mapbox://styles/mapbox/streets-v12` | Đường phố chuẩn |
| `mapbox://styles/mapbox/outdoors-v12` | Địa hình, đi bộ đường dài |
| `mapbox://styles/mapbox/light-v11` | Sáng, tối giản |
| `mapbox://styles/mapbox/dark-v11` | Tối, tối giản |
| `mapbox://styles/mapbox/satellite-v9` | Ảnh vệ tinh |
| `mapbox://styles/mapbox/satellite-streets-v12` | Vệ tinh + tên đường |

```javascript
// Đổi style lúc chạy
map.setStyle('mapbox://styles/mapbox/dark-v11');
```

> ⚠️ `setStyle()` sẽ **xóa hết** layer/marker tùy chỉnh đã thêm → phải thêm lại trong sự kiện `styledata` hoặc `style.load`.

```javascript
map.on('style.load', () => {
  // thêm lại layer tùy chỉnh ở đây
  addMyLayers();
});
```

### 11.2. Style JSON tùy chỉnh

Style là 1 file JSON mô tả sources + layers. Bạn có thể viết tay hoặc xuất từ Mapbox Studio:

```javascript
const myStyle = {
  version: 8,
  sources: {
    'osm-tiles': {
      type: 'raster',
      tiles: ['https://tile.openstreetmap.org/{z}/{x}/{y}.png'],
      tileSize: 256,
      attribution: '© OpenStreetMap contributors'
    }
  },
  layers: [{ id: 'osm', type: 'raster', source: 'osm-tiles' }]
};

const map = new mapboxgl.Map({ container: 'map', style: myStyle });
```

---

## 12. Layers — Vẽ Dữ Liệu Lên Bản Đồ

Layer là cách mạnh nhất của Mapbox: vẽ hàng nghìn điểm/đường/vùng bằng GPU.

### 12.1. Thêm GeoJSON source + layer

```javascript
map.on('load', () => {
  // 1. Thêm nguồn dữ liệu
  map.addSource('places', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: [
        { type: 'Feature', geometry: { type: 'Point', coordinates: [105.8542, 21.0285] }, properties: { name: 'Hồ Gươm' } },
        { type: 'Feature', geometry: { type: 'Point', coordinates: [105.8355, 21.0285] }, properties: { name: 'Văn Miếu' } }
      ]
    }
  });

  // 2. Thêm layer vẽ các điểm
  map.addLayer({
    id: 'places-points',
    type: 'circle',
    source: 'places',
    paint: {
      'circle-radius': 8,
      'circle-color': '#3b82f6',
      'circle-stroke-width': 2,
      'circle-stroke-color': '#ffffff'
    }
  });
});
```

### 12.2. Vẽ đường (route)

```javascript
map.addSource('route', {
  type: 'geojson',
  data: {
    type: 'Feature',
    geometry: {
      type: 'LineString',
      coordinates: [
        [105.8542, 21.0285],
        [105.8442, 21.0300],
        [105.8342, 21.0378]
      ]
    }
  }
});

map.addLayer({
  id: 'route-line',
  type: 'line',
  source: 'route',
  paint: {
    'line-color': '#ff0000',
    'line-width': 4,
    'line-dasharray': [2, 2] // nét đứt
  }
});
```

### 12.3. Vẽ vùng (polygon)

```javascript
map.addSource('area', {
  type: 'geojson',
  data: {
    type: 'Feature',
    geometry: {
      type: 'Polygon',
      coordinates: [[
        [105.85, 21.02], [105.86, 21.02],
        [105.86, 21.03], [105.85, 21.03],
        [105.85, 21.02]
      ]]
    }
  }
});

map.addLayer({
  id: 'area-fill',
  type: 'fill',
  source: 'area',
  paint: { 'fill-color': '#00ff00', 'fill-opacity': 0.3 }
});
```

### 12.4. Symbol layer — label + icon cho nhiều điểm

```javascript
map.addLayer({
  id: 'place-labels',
  type: 'symbol',
  source: 'places',
  layout: {
    'text-field': ['get', 'name'],   // lấy tên từ properties
    'text-font': ['Open Sans Bold'],
    'text-size': 12,
    'text-offset': [0, 1.5],
    'icon-image': 'marker-15'        // icon có sẵn của Mapbox
  },
  paint: { 'text-color': '#111111' }
});
```

### 12.5. Heatmap

```javascript
map.addLayer({
  id: 'heatmap',
  type: 'heatmap',
  source: 'places',
  paint: {
    'heatmap-weight': 1,
    'heatmap-intensity': 1,
    'heatmap-color': [
      'interpolate', ['linear'], ['heatmap-density'],
      0, 'rgba(0,0,255,0)',
      0.5, 'rgba(255,255,0,0.8)',
      1, 'rgba(255,0,0,0.9)'
    ]
  }
});
```

### 12.6. 3D Buildings

```javascript
map.on('load', () => {
  map.setPitch(60); // nghiêng để thấy 3D
  map.addLayer({
    id: '3d-buildings',
    source: 'composite',
    'source-layer': 'building',
    type: 'fill-extrusion',
    minzoom: 14,
    paint: {
      'fill-extrusion-color': '#aaa',
      'fill-extrusion-height': ['get', 'height'],
      'fill-extrusion-opacity': 0.6
    }
  });
});
```

### 12.7. Click vào feature để hiện popup

```javascript
map.on('click', 'places-points', (e) => {
  const props = e.features[0].properties;
  new mapboxgl.Popup()
    .setLngLat(e.lngLat)
    .setHTML(`<b>${props.name}</b>`)
    .addTo(map);
});

// Đổi con trỏ khi hover
map.on('mouseenter', 'places-points', () => {
  map.getCanvas().style.cursor = 'pointer';
});
map.on('mouseleave', 'places-points', () => {
  map.getCanvas().style.cursor = '';
});
```

---

## 13. Geocoding API — Địa Chỉ ↔ Tọa Độ

### 13.1. Forward geocoding (địa chỉ → tọa độ)

```javascript
const query = encodeURIComponent('Hồ Gươm, Hà Nội');
const url = `https://api.mapbox.com/geocoding/v5/mapbox.places/${query}.json?access_token=YOUR_MAPBOX_TOKEN&language=vi&limit=3`;

const res = await fetch(url);
const data = await res.json();

data.features.forEach(f => {
  console.log(f.place_name, '→', f.center); // [lng, lat]
});
```

**Tham số hữu ích:**

| Tham số | Ý nghĩa | Ví dụ |
|---|---|---|
| `language=vi` | Trả tên tiếng Việt | `language=vi` |
| `limit=5` | Số kết quả | `limit=5` |
| `bbox=...` | Giới hạn vùng tìm | `bbox=105.7,20.9,106.0,21.1` |
| `proximity=105.85,21.02` | Ưu tiên gần điểm này | `proximity=lng,lat` |
| `types=address,poi` | Loại địa điểm | `types=poi` |

### 13.2. Reverse geocoding (tọa độ → địa chỉ)

```javascript
const [lng, lat] = [105.8542, 21.0285];
const url = `https://api.mapbox.com/geocoding/v5/mapbox.places/${lng},${lat}.json?access_token=YOUR_MAPBOX_TOKEN&language=vi`;

const data = await (await fetch(url)).json();
console.log(data.features[0].place_name);
```

### 13.3. Search box tự động hoàn thành

```javascript
const input = document.getElementById('search');
let timer;

input.addEventListener('input', () => {
  clearTimeout(timer);
  timer = setTimeout(async () => {
    const q = encodeURIComponent(input.value);
    const url = `https://api.mapbox.com/geocoding/v5/mapbox.places/${q}.json?access_token=YOUR_MAPBOX_TOKEN&autocomplete=true&limit=5&language=vi`;
    const data = await (await fetch(url)).json();
    // render data.features thành dropdown
    renderSuggestions(data.features);
  }, 300); // debounce 300ms
});
```

> 💡 **Tiết kiệm quota:** Dùng `session_token` (UUID) cho cả phiên search để Mapbox tính là 1 session thay vì từng request.

---

## 14. Directions API — Chỉ Đường

### 14.1. Lấy lộ trình cơ bản

```javascript
const start = [105.8542, 21.0285]; // Hồ Gươm
const end = [105.8342, 21.0378];   // Lăng Bác

const url = `https://api.mapbox.com/directions/v5/mapbox/driving/${start[0]},${start[1]};${end[0]},${end[1]}?geometries=geojson&access_token=YOUR_MAPBOX_TOKEN`;

const data = await (await fetch(url)).json();
const route = data.routes[0];

console.log('Khoảng cách:', (route.distance / 1000).toFixed(2), 'km');
console.log('Thời gian:', Math.round(route.duration / 60), 'phút');

// Vẽ lên bản đồ
map.addSource('route', { type: 'geojson', data: route.geometry });
map.addLayer({
  id: 'route', type: 'line', source: 'route',
  paint: { 'line-color': '#3887be', 'line-width': 5 }
});
```

### 14.2. Các profile di chuyển

| Profile | Ý nghĩa |
|---|---|
| `driving` | Ô tô |
| `driving-traffic` | Ô tô + tính kẹt xe (cần token đặc biệt) |
| `walking` | Đi bộ |
| `cycling` | Xe đạp |

```javascript
// Đi bộ thay vì lái xe
const url = `https://api.mapbox.com/directions/v5/mapbox/walking/${start};${end}?geometries=geojson&access_token=TOKEN`;
```

### 14.3. Hướng dẫn từng bước (turn-by-turn)

```javascript
const url = `...?steps=true&language=vi&access_token=TOKEN`;
const data = await (await fetch(url)).json();

data.routes[0].legs[0].steps.forEach(s => {
  console.log(s.maneuver.instruction); // "Rẽ phải vào phố..."
});
```

---

## 15. Mapbox Studio — Tự Thiết Kế Style

1. Vào https://studio.mapbox.com/
2. **New Style** → chọn style gốc (ví dụ Streets)
3. Tùy chỉnh:
   - **Layers:** bật/tắt, đổi màu từng loại (đường, nước, công viên...)
   - **Fonts:** đổi font label
   - **3D:** bật buildings
4. **Publish** → copy style URL dạng `mapbox://styles/username/abc123`
5. Dùng trong code:

```javascript
const map = new mapboxgl.Map({
  container: 'map',
  style: 'mapbox://styles/username/abc123', // style của bạn
  center: [105.85, 21.02],
  zoom: 12
});
```

---

## 16. Tối Ưu Hiệu Suất

### 16.1. Dùng layer thay vì Marker khi nhiều điểm

| Số điểm | Cách nên dùng |
|---|---|
| < 100 | `new mapboxgl.Marker()` |
| 100 – 1.000 | Symbol layer |
| > 1.000 | Symbol layer + clustering |
| > 10.000 | Vector tiles tự host |

### 16.2. Clustering

```javascript
map.addSource('places', {
  type: 'geojson',
  data: geojsonData,
  cluster: true,
  clusterMaxZoom: 14,
  clusterRadius: 50
});

// Layer cụm
map.addLayer({
  id: 'clusters', type: 'circle', source: 'places',
  filter: ['has', 'point_count'],
  paint: {
    'circle-color': ['step', ['get', 'point_count'], '#51bbd6', 100, '#f1f075', 750, '#f28cb1'],
    'circle-radius': ['step', ['get', 'point_count'], 20, 100, 30, 750, 40]
  }
});

// Số lượng trong cụm
map.addLayer({
  id: 'cluster-count', type: 'symbol', source: 'places',
  filter: ['has', 'point_count'],
  layout: { 'text-field': '{point_count_abbreviated}', 'text-size': 12 }
});

// Điểm lẻ
map.addLayer({
  id: 'unclustered', type: 'circle', source: 'places',
  filter: ['!', ['has', 'point_count']],
  paint: { 'circle-color': '#11b4da', 'circle-radius': 6 }
});
```

### 16.3. Các mẹo khác

- Đặt `trackResize: false` nếu container không đổi kích thước
- Dùng `map.remove()` khi unmount component (tránh rò rỉ)
- Giới hạn `maxBounds` để không tải tiles vô ích
- Dùng `cooperativeGestures: true` trên mobile để tránh cuộn trang bị kẹt

```javascript
const map = new mapboxgl.Map({
  // ...
  maxBounds: [[105.5, 20.8], [106.2, 21.2]], // giới hạn Hà Nội
  cooperativeGestures: true
});
```

---

## 17. Pricing & Free Tier Chi Tiết

### 17.1. Free tier (không cần thẻ)

| Sản phẩm | Free/tháng |
|---|---|
| Map loads (GL JS) | **50.000** |
| Static images | 50.000 |
| Geocoding tạm thời | 100.000 |
| Directions | 100.000 |
| Tilequery | 100.000 |

### 17.2. Giá khi vượt free tier (tham khảo)

| Sản phẩm | Giá vượt quota |
|---|---|
| Map loads | ~$5 / 1.000 loads |
| Geocoding tạm thời | ~$0.75 / 1.000 |
| Geocoding lưu trữ (permanent) | ~$5 / 1.000 |
| Directions | ~$0.50 / 1.000 |

> ⚠️ Giá có thể thay đổi — luôn kiểm tra https://www.mapbox.com/pricing trước khi chốt.

### 17.3. Ví dụ tính tiền

- Web 100.000 map loads/tháng → (100.000 − 50.000) × $5/1000 = **$250/tháng**
- App 10.000 loads + 5.000 geocoding → **$0** (trong free tier)

### 17.4. Kiểm soát chi phí

1. Đặt **URL restrictions** cho token → tránh bị đánh cắp quota
2. Theo dõi usage tại https://account.mapbox.com → **Statistics**
3. Cache kết quả Geocoding ở backend (đừng gọi lại cùng 1 địa chỉ)
4. Dùng `session_token` cho search autocomplete

---

## 18. Bảo Mật Token

```javascript
// ✅ .env (không commit!)
VITE_MAPBOX_TOKEN=pk.ey...

// .gitignore
// .env
```

```javascript
// ✅ Kiểm tra domain ở backend trước khi trả token (nâng cao)
// Server chỉ trả public token cho domain hợp lệ
app.get('/api/mapbox-token', (req, res) => {
  const referer = req.get('referer') || '';
  if (!referer.startsWith('https://ten-mien-cua-ban.com')) {
    return res.status(403).send('Forbidden');
  }
  res.json({ token: process.env.MAPBOX_PUBLIC_TOKEN });
});
```

**Checklist bảo mật:**
- [ ] Token public có URL restrictions
- [ ] Không commit `.env` lên Git
- [ ] Secret token (`sk.`) chỉ ở server
- [ ] Xoay vòng token định kỳ / khi nghi lộ

---

## 19. Xử Lý Lỗi Thường Gặp

| Lỗi / Triệu chứng | Nguyên nhân | Cách fix |
|---|---|---|
| Bản đồ xám, không hiện | Sai token hoặc hết quota | Kiểm tra token tại account.mapbox.com |
| `401 Unauthorized` | Token sai / bị xóa | Tạo token mới |
| `403 Forbidden` | URL restrictions chặn domain | Thêm domain vào token settings |
| Bản đồ trắng trơn | Thiếu CSS `height` cho container | `#map { height: 500px; }` |
| `setStyle` xong mất layer | Style mới xóa layer cũ | Thêm lại layer trong `style.load` |
| Marker bay sai chỗ | Nhầm thứ tự `[lat, lng]` | Dùng `[lng, lat]` |
| Chữ bị vỡ khi zoom | Dùng raster thay vì vector | Dùng style vector của Mapbox |
| `Failed to load glyphs` | Thiếu quyền `fonts:read` | Thêm scope vào token |
| Chậm khi nhiều marker | Dùng DOM marker quá nhiều | Chuyển sang symbol layer + cluster |
| Mobile cuộn trang bị kẹt | Map chiếm gesture | `cooperativeGestures: true` |

---

## 20. Best Practices & Cheat Sheet

### Cheat sheet nhanh

```javascript
// Khởi tạo
mapboxgl.accessToken = 'TOKEN';
const map = new mapboxgl.Map({ container, style, center: [lng, lat], zoom });

// Điều hướng
map.setCenter([lng, lat]); map.setZoom(12);
map.flyTo({ center: [lng, lat], zoom: 14 }); // bay mượt
map.fitBounds([[lng1, lat1], [lng2, lat2]], { padding: 50 });

// Marker + popup
new mapboxgl.Marker().setLngLat([lng, lat]).setPopup(new mapboxgl.Popup().setText('Hi')).addTo(map);

// Sự kiện
map.on('load', () => {});
map.on('click', (e) => console.log(e.lngLat));
map.on('moveend', () => {});

// Dọn dẹp
map.remove();
```

### Best practices

1. **Luôn dọn dẹp:** `map.remove()` khi unmount
2. **Đợi `load`:** Thêm source/layer trong `map.on('load', ...)`
3. **Token qua env:** Không hardcode
4. **Layer > Marker:** Khi điểm nhiều
5. **Debounce search:** Geocoding autocomplete phải debounce
6. **Cache backend:** Kết quả geocoding/directions nên cache
7. **[lng, lat]:** Nhớ thứ tự!

---

## 📚 Tài Liệu Tham Khảo

- Trang chủ: https://www.mapbox.com
- Docs: https://docs.mapbox.com
- Mapbox Studio: https://studio.mapbox.com
- Pricing: https://www.mapbox.com/pricing
- Ví dụ: https://docs.mapbox.com/mapbox-gl-js/example/

---

## 21. Matrix API — Ma Trận Khoảng Cách

Khi cần tính khoảng cách giữa NHIỀU điểm (ví dụ: tìm quán gần nhất trong 20 quán):

```javascript
// Tối đa 25 điểm (gói free), mỗi request tính all-pairs
const coords = [
  [105.8542, 21.0285], // bạn đang ở đây
  [105.8355, 21.0285], // quán A
  [105.8342, 21.0378], // quán B
  [105.8600, 21.0200]  // quán C
].map(c => c.join(',')).join(';');

const url = `https://api.mapbox.com/directions-matrix/v1/mapbox/driving/${coords}?annotations=distance,duration&access_token=YOUR_MAPBOX_TOKEN`;
const data = await (await fetch(url)).json();

// data.durations[0] = thời gian từ điểm 0 tới các điểm khác (giây)
// data.distances[0] = khoảng cách (mét)
data.durations[0].forEach((sec, i) => {
  if (i === 0) return;
  console.log(`Quán ${i}: ${(data.distances[0][i]/1000).toFixed(1)}km, ${Math.round(sec/60)} phút`);
});
```

**Ứng dụng thực tế:** "Tìm 5 quán phở gần nhất" — gọi 1 request Matrix thay vì 20 request Directions!

---

## 22. Static Images API — Ảnh Bản Đồ Tĩnh

Dùng cho thumbnail, ảnh đại diện địa điểm, OG image khi share link.

```
https://api.mapbox.com/styles/v1/mapbox/streets-v12/static/
  pin-s+ff0000(105.8542,21.0285)/       ← marker đỏ tại Hồ Gươm
  105.8542,21.0285,14/                  ← tâm + zoom
  600x400                               ← kích thước
  ?access_token=YOUR_MAPBOX_TOKEN
```

```html
<img
  src="https://api.mapbox.com/styles/v1/mapbox/streets-v12/static/pin-s+ff0000(105.8542,21.0285)/105.8542,21.0285,14/600x400?access_token=YOUR_MAPBOX_TOKEN"
  alt="Bản đồ Hồ Gươm"
/>
```

**Tham số overlay:**

| Cú pháp | Ý nghĩa |
|---|---|
| `pin-s+ff0000(lng,lat)` | Ghim nhỏ màu đỏ |
| `pin-l+0000ff(lng,lat)` | Ghim lớn màu xanh |
| `path-5+ff0000-0.8(lng1,lat1;lng2,lat2)` | Vẽ đường (dày 5, đỏ, opacity 0.8) |

---

## 23. Sự Kiện (Events) Nâng Cao

```javascript
// Click lấy tọa độ
map.on('click', (e) => {
  console.log('Click tại:', e.lngLat.lng, e.lngLat.lat);
});

// Kéo xong mới load dữ liệu (tiết kiệm API)
map.on('moveend', async () => {
  const bounds = map.getBounds();
  console.log('Vùng nhìn thấy:', bounds.toArray());
  // gọi API load quán trong bounds...
});

// Zoom thay đổi
map.on('zoomend', () => {
  console.log('Zoom hiện tại:', map.getZoom());
});

// Hover hiện tooltip theo chuột
const tooltip = new mapboxgl.Popup({ closeButton: false, closeOnClick: false });
map.on('mousemove', 'places-points', (e) => {
  tooltip.setLngLat(e.lngLat)
    .setHTML(`<b>${e.features[0].properties.name}</b>`)
    .addTo(map);
});
map.on('mouseleave', 'places-points', () => tooltip.remove());
```

**Danh sách events hay dùng:**

| Event | Khi nào bắn |
|---|---|
| `load` | Bản đồ tải xong lần đầu |
| `click` | Click vào bản đồ |
| `dblclick` | Double click |
| `mousemove` | Di chuột |
| `movestart` / `moveend` | Bắt đầu/kết thúc di chuyển |
| `zoomstart` / `zoomend` | Bắt đầu/kết thúc zoom |
| `dragstart` / `dragend` | Kéo marker (với draggable marker) |
| `style.load` | Style mới load xong |
| `sourcedata` | Dữ liệu source thay đổi |

---

## 24. Mapbox Studio — Walkthrough Chi Tiết

### 24.1. Tạo style đầu tiên

1. Vào https://studio.mapbox.com/ → **New style** → **Streets** → **Create**
2. Bạn thấy danh sách **Components** bên trái (tương ứng các layer)

### 24.2. Đổi màu nước thành xanh ngọc

1. Tìm component **Water** → bấm vào
2. Tab **Style** → **Color** → chọn `#5ec8d8`
3. Bản đồ preview cập nhật ngay

### 24.3. Ẩn/hiện layer

1. Tìm **POI labels** (nhãn địa điểm)
2. Bấm con mắt 👁️ để ẩn → bản đồ gọn hơn

### 24.4. Thêm layer dữ liệu riêng

1. Tab **Tilesets** → **Upload** → upload file GeoJSON của bạn
2. Quay lại Style → **+** → **Add layer** → chọn tileset vừa upload
3. Chọn kiểu: `Circle`, `Symbol`, `Fill`...

### 24.5. Publish và lấy style URL

1. Bấm **Publish** (góc phải trên) → **Publish**
2. Bấm **Share** → copy **Style URL**: `mapbox://styles/username/abc123xyz`
3. Dùng trong code như style thường

---

## 25. Mapbox Cho Mobile (Tổng Quan)

| SDK | Ngôn ngữ | Ghi chú |
|---|---|---|
| **Maps SDK for iOS** | Swift | Native iOS |
| **Maps SDK for Android** | Kotlin/Java | Native Android |
| **Mapbox Maps Flutter** | Dart | Plugin chính thức |
| **React Native** | JS | `@rnmapbox/maps` (community) |

```dart
// Flutter ví dụ (cần token)
MapWidget(
  key: ValueKey("mapWidget"),
  resourceOptions: ResourceOptions(accessToken: 'YOUR_TOKEN'),
  cameraOptions: CameraOptions(
    center: Point(coordinates: Position(105.8542, 21.0285)),
    zoom: 13.0,
  ),
  styleUri: MapboxStyles.MAPBOX_STREETS,
)
```

> 💡 Nếu muốn free cho mobile → dùng **MapLibre Native** + OpenFreeMap (không cần token).

---

## 26. Tích Hợp Tìm Kiếm Địa Điểm Hoàn Chỉnh

Ví dụ đầy đủ: ô search → gợi ý → click → bay tới + ghim marker:

```html
<input id="search" placeholder="Tìm địa điểm..." style="position:absolute;top:10px;left:10px;z-index:1;padding:10px;width:300px;" />
<ul id="suggestions" style="position:absolute;top:50px;left:10px;z-index:1;background:white;list-style:none;padding:0;width:320px;"></ul>
<div id="map" style="height:100vh;"></div>

<script>
mapboxgl.accessToken = 'YOUR_MAPBOX_TOKEN';
const map = new maplibregl.Map({ // dùng mapboxgl.Map nếu dùng Mapbox
  container: 'map',
  style: 'mapbox://styles/mapbox/streets-v12',
  center: [105.8542, 21.0285],
  zoom: 12
});

const input = document.getElementById('search');
const list = document.getElementById('suggestions');
let timer;
const sessionToken = crypto.randomUUID(); // 1 session cho cả phiên search

input.addEventListener('input', () => {
  clearTimeout(timer);
  timer = setTimeout(async () => {
    const q = input.value.trim();
    if (q.length < 3) { list.innerHTML = ''; return; }
    const url = `https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(q)}.json?access_token=${mapboxgl.accessToken}&autocomplete=true&limit=5&language=vi&session_token=${sessionToken}&proximity=105.8542,21.0285`;
    const data = await (await fetch(url)).json();
    list.innerHTML = '';
    data.features.forEach(f => {
      const li = document.createElement('li');
      li.textContent = f.place_name;
      li.style.cssText = 'padding:8px;cursor:pointer;border-bottom:1px solid #eee;';
      li.onclick = () => {
        map.flyTo({ center: f.center, zoom: 15 });
        new mapboxgl.Marker().setLngLat(f.center)
          .setPopup(new mapboxgl.Popup().setText(f.place_name))
          .addTo(map);
        list.innerHTML = '';
        input.value = f.place_name;
      };
      list.appendChild(li);
    });
  }, 300);
});
</script>
```

---

## 27. Câu Hỏi Thường Gặp (FAQ)

**Hỏi: Dùng token public ở frontend có an toàn không?**
Đáp: An toàn ở mức chấp nhận được NẾU đã set URL restrictions. Token public bắt buộc phải lộ ở client — đó là thiết kế của Mapbox. Kẻ xấu lấy được cũng chỉ dùng được trên domain bạn cho phép.

**Hỏi: 50.000 map loads là gì? Tính thế nào?**
Đáp: Mỗi lần `new mapboxgl.Map()` khởi tạo = 1 map load. User F5 10 lần = 10 loads. Pan/zoom không tính thêm.

**Hỏi: Có cách nào dùng Mapbox mà không tốn tiền khi traffic lớn?**
Đáp: (1) Cache tiles qua CDN của bạn, (2) Giảm số lần init map (SPA chỉ init 1 lần), (3) Cân nhắc chuyển sang MapLibre + self-host khi bill quá cao.

**Hỏi: Mapbox có hỗ trợ tiếng Việt không?**
Đáp: Geocoding có `language=vi`. Style mặc định hiển thị tên theo ngôn ngữ địa phương (ở Việt Nam hiện tên tiếng Việt). Muốn ép ngôn ngữ khác → dùng Mapbox Studio chỉnh `text-field`.

**Hỏi: Khác nhau giữa `streets-v11` và `streets-v12`?**
Đáp: v12 là bản mới hơn (dữ liệu + style cải tiến). Luôn dùng bản mới nhất trừ khi cần tương thích code cũ.

---

## 28. Checklist Trước Khi Production

- [ ] Token public đã set URL restrictions (domain production + staging)
- [ ] `.env` đã thêm vào `.gitignore`
- [ ] Secret token chỉ ở backend, không lộ ra client
- [ ] Đã test với `map.remove()` khi unmount (không rò rỉ memory)
- [ ] Geocoding search có debounce + session_token
- [ ] Kết quả geocoding/directions được cache ở backend
- [ ] Đã bật billing alerts ở Mapbox dashboard
- [ ] Attribution hiển thị đúng (Mapbox tự thêm)
- [ ] Test trên mobile (touch, cooperativeGestures)
- [ ] Fallback khi API lỗi (hiện thông báo thân thiện)
