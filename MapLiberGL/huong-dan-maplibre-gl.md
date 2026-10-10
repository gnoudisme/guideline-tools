# Hướng Dẫn Toàn Tập: MapLibre GL JS — Bản Đồ Open Source Miễn Phí

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 10/10/2026
>
> 🎉 **Điểm ăn tiền nhất:** Không cần token, không cần đăng ký, không giới hạn — mở lên là chạy!

---

## 📑 Mục Lục

1. [MapLibre GL Là Gì?](#1-maplibre-gl-là-gì)
2. [Lịch Sử: Vì Sao Có MapLibre?](#2-lịch-sử-vì-sao-có-maplibre)
3. [MapLibre vs Mapbox — So Sánh Chi Tiết](#3-maplibre-vs-mapbox--so-sánh-chi-tiết)
4. [Kiến Trúc: Thư Viện Render + Tile Provider](#4-kiến-trúc-thư-viện-render--tile-provider)
5. [Cài Đặt](#5-cài-đặt)
6. [Bản Đồ Đầu Tiên (Không Cần Token!)](#6-bản-đồ-đầu-tiên-không-cần-token)
7. [Các Style Miễn Phí Có Sẵn](#7-các-style-miễn-phí-có-sẵn)
8. [Tọa Độ [lng, lat]](#8-tọa-độ-lng-lat)
9. [Marker & Popup](#9-marker--popup)
10. [Controls](#10-controls)
11. [Layers: Source + Layer](#11-layers-source--layer)
12. [Tự Viết Style JSON](#12-tự-viết-style-json)
13. [Dùng Với Các Tile Provider](#13-dùng-với-các-tile-provider)
14. [Geocoding Miễn Phí (Nominatim)](#14-geocoding-miễn-phí-nominatim)
15. [Routing Miễn Phí (OSRM / Valhalla)](#15-routing-miễn-phí-osrm--valhalla)
16. [Self-host Tiles](#16-self-host-tiles)
17. [Migrate Từ Mapbox Sang MapLibre](#17-migrate-từ-mapbox-sang-maplibre)
18. [Tối Ưu Hiệu Suất](#18-tối-ưu-hiệu-suất)
19. [Xử Lý Lỗi Thường Gặp](#19-xử-lý-lỗi-thường-gặp)
20. [Best Practices & Cheat Sheet](#20-best-practices--cheat-sheet)

---

## 1. MapLibre GL Là Gì?

**MapLibre GL JS** là thư viện JavaScript open-source để vẽ bản đồ vector tương tác trên web bằng WebGL. Nó render bản đồ mượt mà, hỗ trợ 3D, custom style — giống hệt Mapbox GL JS.

**Điểm khác biệt cốt lõi:** MapLibre là **thư viện render thuần túy** — nó KHÔNG đi kèm dữ liệu bản đồ. Bạn tự chọn nguồn tiles (miễn phí có, trả phí có, tự host cũng được).

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  MapLibre GL JS │ ←── │   Style JSON     │ ←── │  Tile Provider  │
│  (render WebGL) │     │ (mô tả cách vẽ)  │     │ (dữ liệu bản đồ)│
└─────────────────┘     └──────────────────┘     └─────────────────┘
   maplibre.org            bạn tự chọn            OpenFreeMap /
                                                  MapTiler / tự host
```

### 1.1. Hệ sinh thái MapLibre

| Dự án | Mô tả |
|---|---|
| **MapLibre GL JS** | Thư viện web (JavaScript) |
| **MapLibre Native** | iOS, Android, macOS, Windows |
| **MapLibre React Native** | Cho React Native |
| **Martin** | Tile server viết bằng Rust (cùng tổ chức) |
| **MapLibre Tile (MLT)** | Định dạng vector tile mới, nén tốt hơn MVT |

---

## 2. Lịch Sử: Vì Sao Có MapLibre?

Câu chuyện ngắn gọn:

1. **Trước 2020:** Mapbox GL JS là open-source (BSD license), ai cũng dùng free
2. **Tháng 12/2020:** Mapbox đổi license sang **BSL** (Business Source License) từ v2.0 — không còn open-source thật, bắt buộc dùng token của Mapbox
3. **Cộng đồng phản ứng:** Fork bản v1.x cuối cùng còn BSD → đặt tên **MapLibre GL**
4. **Hiện tại:** MapLibre phát triển độc lập, cộng đồng quản trị, tài trợ bởi nhiều công ty (không phụ thuộc 1 công ty nào)

> 💡 **Bài học:** Đây là lý do nhiều dự án open-source chọn MapLibre — không lo một ngày bị đổi license hay tăng giá.

---

## 3. MapLibre vs Mapbox — So Sánh Chi Tiết

| Tiêu chí | MapLibre GL JS | Mapbox GL JS (v2+) |
|---|---|---|
| License | **BSD-3** (open source thật) | BSL (proprietary) |
| Cần token | ❌ **Không** | ✅ Bắt buộc |
| Dữ liệu bản đồ đi kèm | ❌ Tự chọn provider | ✅ Mapbox tiles |
| Giá | **Miễn phí** (trả cho tile provider nếu dùng) | Free 50k loads, sau đó trả phí |
| Mapbox Studio | ❌ Không dùng được | ✅ Có |
| Geocoding/Directions API | ❌ Tự tích hợp (Nominatim, OSRM...) | ✅ Có sẵn |
| 3D buildings/terrain | ✅ Có | ✅ Có |
| Globe projection | ✅ Có | ✅ Có |
| Cộng đồng | Cộng đồng mở | Công ty Mapbox |
| Vendor lock-in | ❌ Không | ⚠️ Có |
| Độ ổn định API | Tốt (fork từ v1, phát triển tiếp) | Tốt |

**Khi nào chọn MapLibre:**
- ✅ Muốn **miễn phí hoàn toàn**, không đăng ký tài khoản
- ✅ Dự án open-source, portfolio (clone về chạy ngay, không cần token bí mật)
- ✅ Muốn **tự chủ** dữ liệu, tự host tiles
- ✅ Không muốn phụ thuộc vào 1 công ty

**Khi nào vẫn chọn Mapbox:**
- Cần Mapbox Studio, Search API, Directions API xịn
- Team đã quen hệ sinh thái Mapbox
- Sẵn sàng trả phí khi scale

---

## 4. Kiến Trúc: Thư Viện Render + Tile Provider

Hiểu rõ kiến trúc này là chìa khóa dùng MapLibre:

### 4.1. 3 thành phần

1. **MapLibre GL JS** — thư viện render (bạn `npm install`)
2. **Style JSON** — file JSON mô tả: lấy tiles ở đâu, vẽ layer nào, màu gì, font gì
3. **Tile Provider** — server chứa tiles (OpenFreeMap, MapTiler, CARTO, tự host...)

### 4.2. So sánh các tile provider miễn phí

| Provider | Style URL mẫu | Cần key? | Giới hạn | Ghi chú |
|---|---|---|---|---|
| **OpenFreeMap** | `https://tiles.openfreemap.org/styles/liberty` | ❌ Không | Không giới hạn | Donation-funded, khuyên dùng |
| **MapLibre demo** | `https://demotiles.maplibre.org/style.json` | ❌ Không | Demo thôi | Chỉ dùng để thử, không production |
| **CARTO basemaps** | `https://basemaps.cartocdn.com/gl/positron-gl-style/style.json` | ❌ Không* | 1M/tháng (commercial) | *Production cần key free |
| **MapTiler** | `https://api.maptiler.com/maps/streets/style.json?key=KEY` | ✅ Có | 100k/tháng free | Cần đăng ký |
| **Stadia Maps** | `https://tiles.stadiamaps.com/styles/alidade_smooth.json` | ⚠️ Localhost free | 200k credits/tháng | Production cần key |
| **Tự host** | `https://server-cua-ban/style.json` | ❌ Không | Tự lo | Martin, tileserver-gl, PMTiles |

---

## 5. Cài Đặt

### 5.1. Qua CDN

```html
<script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
<link href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css" rel="stylesheet" />
```

### 5.2. Qua npm

```bash
npm install maplibre-gl
```

```javascript
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';
```

### 5.3. React

```jsx
// Map.jsx
import { useEffect, useRef } from 'react';
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';

export default function Map() {
  const mapRef = useRef(null);

  useEffect(() => {
    const map = new maplibregl.Map({
      container: mapRef.current,
      style: 'https://tiles.openfreemap.org/styles/liberty',
      center: [105.8542, 21.0285], // Hồ Gươm
      zoom: 13
    });
    map.addControl(new maplibregl.NavigationControl(), 'top-right');

    return () => map.remove();
  }, []);

  return <div ref={mapRef} style={{ width: '100%', height: '500px' }} />;
}
```

---

## 6. Bản Đồ Đầu Tiên (Không Cần Token!)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8" />
  <title>MapLibre Demo</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
  <link href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css" rel="stylesheet" />
  <style>
    body { margin: 0; }
    #map { width: 100%; height: 100vh; }
  </style>
</head>
<body>
  <div id="map"></div>
  <script>
    // KHÔNG CẦN TOKEN! 🎉
    const map = new maplibregl.Map({
      container: 'map',
      style: 'https://tiles.openfreemap.org/styles/liberty',
      center: [105.8542, 21.0285], // Hồ Gươm, Hà Nội
      zoom: 13
    });
    map.addControl(new maplibregl.NavigationControl());
  </script>
</body>
</html>
```

Mở file bằng trình duyệt → bản đồ hiện ngay, không đăng ký gì cả!

---

## 7. Các Style Miễn Phí Có Sẵn

### 7.1. OpenFreeMap styles

| Style | URL | Mô tả |
|---|---|---|
| **Liberty** | `https://tiles.openfreemap.org/styles/liberty` | Đầy đủ chi tiết, mặc định khuyên dùng |
| **Bright** | `https://tiles.openfreemap.org/styles/bright` | Sáng, gọn gàng |
| **Positron** | `https://tiles.openfreemap.org/styles/positron` | Xám nhạt, nền cho data viz |

```javascript
// Đổi style lúc chạy
map.setStyle('https://tiles.openfreemap.org/styles/bright');
```

### 7.2. CARTO basemaps (raster + vector)

```javascript
// Positron (sáng)
style: 'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json'

// Dark matter (tối)
style: 'https://basemaps.cartocdn.com/gl/dark-matter-gl-style/style.json'
```

### 7.3. MapLibre demo tiles (chỉ để thử)

```javascript
style: 'https://demotiles.maplibre.org/style.json'
```

> ⚠️ Demo tiles **không dùng cho production** — chậm và có thể bị tắt bất cứ lúc nào.

---

## 8. Tọa Độ [lng, lat]

Giống Mapbox: MapLibre dùng chuẩn GeoJSON `[kinh độ, vĩ độ]`.

```javascript
// ✅ Hồ Gươm
map.setCenter([105.8542, 21.0285]);

// ❌ Đảo ngược → bay ra biển
// map.setCenter([21.0285, 105.8542]);
```

---

## 9. Marker & Popup

API giống hệt Mapbox (vì fork từ đó):

```javascript
// Marker mặc định
new maplibregl.Marker()
  .setLngLat([105.8542, 21.0285])
  .addTo(map);

// Marker màu tùy chỉnh
new maplibregl.Marker({ color: '#ff0000' })
  .setLngLat([105.8355, 21.0285])
  .addTo(map);

// Marker HTML tùy chỉnh
const el = document.createElement('div');
el.innerHTML = '🍜';
el.style.fontSize = '28px';
new maplibregl.Marker({ element: el })
  .setLngLat([105.8342, 21.0378])
  .addTo(map);

// Popup
const popup = new maplibregl.Popup({ offset: 25 })
  .setHTML('<h3>Phở Gà</h3><p>⭐ 4.9 · 40k</p>');

new maplibregl.Marker()
  .setLngLat([105.8542, 21.0285])
  .setPopup(popup)
  .addTo(map);
```

---

## 10. Controls

```javascript
map.addControl(new maplibregl.NavigationControl(), 'top-right');
map.addControl(new maplibregl.ScaleControl(), 'bottom-left');
map.addControl(new maplibregl.GeolocateControl({
  positionOptions: { enableHighAccuracy: true },
  trackUserLocation: true
}));
map.addControl(new maplibregl.FullscreenControl());
```

---

## 11. Layers: Source + Layer

### 11.1. GeoJSON points

```javascript
map.on('load', () => {
  map.addSource('quan-an', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: [
        {
          type: 'Feature',
          geometry: { type: 'Point', coordinates: [105.8542, 21.0285] },
          properties: { name: 'Phở Bò', rating: 4.8 }
        },
        {
          type: 'Feature',
          geometry: { type: 'Point', coordinates: [105.8355, 21.0285] },
          properties: { name: 'Bún Chả', rating: 4.9 }
        }
      ]
    }
  });

  map.addLayer({
    id: 'quan-an-points',
    type: 'circle',
    source: 'quan-an',
    paint: {
      'circle-radius': [
        'interpolate', ['linear'], ['get', 'rating'],
        4.0, 6,
        5.0, 14
      ],
      'circle-color': '#ef4444',
      'circle-stroke-width': 2,
      'circle-stroke-color': '#fff'
    }
  });

  // Label
  map.addLayer({
    id: 'quan-an-labels',
    type: 'symbol',
    source: 'quan-an',
    layout: {
      'text-field': ['get', 'name'],
      'text-size': 12,
      'text-offset': [0, 1.2]
    }
  });
});
```

### 11.2. Click để xem chi tiết

```javascript
map.on('click', 'quan-an-points', (e) => {
  const p = e.features[0].properties;
  new maplibregl.Popup()
    .setLngLat(e.lngLat)
    .setHTML(`<b>${p.name}</b><br>⭐ ${p.rating}`)
    .addTo(map);
});
```

### 11.3. Vẽ đường đi

```javascript
map.addSource('duong-di', {
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
  id: 'duong-di-line',
  type: 'line',
  source: 'duong-di',
  paint: { 'line-color': '#3b82f6', 'line-width': 4 }
});
```

---

## 12. Tự Viết Style JSON

Style JSON là "công thức" vẽ bản đồ. Cấu trúc cơ bản:

```json
{
  "version": 8,
  "name": "Style của tôi",
  "sources": {
    "openmaptiles": {
      "type": "vector",
      "url": "https://tiles.openfreemap.org/planet"
    }
  },
  "sprite": "https://tiles.openfreemap.org/styles/liberty/sprite",
  "glyphs": "https://tiles.openfreemap.org/fonts/{fontstack}/{range}.pbf",
  "layers": [
    {
      "id": "background",
      "type": "background",
      "paint": { "background-color": "#f8f4f0" }
    },
    {
      "id": "water",
      "type": "fill",
      "source": "openmaptiles",
      "source-layer": "water",
      "paint": { "fill-color": "#a0c8f0" }
    }
  ]
}
```

**Các trường quan trọng:**

| Trường | Ý nghĩa |
|---|---|
| `version` | Luôn là `8` |
| `sources` | Nguồn dữ liệu (vector/raster/geojson) |
| `sprite` | URL sprite chứa icon |
| `glyphs` | URL font chữ (template `{fontstack}/{range}`) |
| `layers` | Danh sách layer vẽ theo thứ tự (dưới → trên) |

> 💡 **Mẹo:** Không cần viết từ đầu — copy style của OpenFreeMap về, sửa màu sắc theo ý thích!

```bash
# Tải style mẫu về sửa
curl https://tiles.openfreemap.org/styles/liberty -o my-style.json
# Sửa file, rồi dùng:
# style: '/my-style.json'  (host cùng domain)
```

---

## 13. Dùng Với Các Tile Provider

### 13.1. MapTiler (cần key free)

```javascript
const map = new maplibregl.Map({
  container: 'map',
  style: 'https://api.maptiler.com/maps/streets/style.json?key=YOUR_MAPTILER_KEY',
  center: [105.85, 21.02],
  zoom: 12
});
```

### 13.2. Stadia Maps

```javascript
style: 'https://tiles.stadiamaps.com/styles/alidade_smooth.json'
// Production cần thêm ?api_key=KEY
```

### 13.3. Protomaps (PMTiles — 1 file duy nhất!)

```javascript
// PMTiles: toàn bộ tiles trong 1 file, host ở đâu cũng được
map.addSource('protomaps', {
  type: 'vector',
  url: 'pmtiles://https://example.com/vietnam.pmtiles'
});
```

---

## 14. Geocoding Miễn Phí (Nominatim)

MapLibre không có Geocoding API riêng → dùng **Nominatim** (của OpenStreetMap, miễn phí):

```javascript
// Forward: địa chỉ → tọa độ
async function geocode(query) {
  const url = `https://nominatim.openstreetmap.org/search?q=${encodeURIComponent(query)}&format=json&limit=5&accept-language=vi`;
  const res = await fetch(url, {
    headers: { 'User-Agent': 'MyApp/1.0 (contact@example.com)' }
  });
  const data = await res.json();
  return data.map(d => ({
    name: d.display_name,
    lat: parseFloat(d.lat),
    lng: parseFloat(d.lon)
  }));
}

// Dùng thử
const results = await geocode('Hồ Gươm, Hà Nội');
console.log(results[0]); // { name: '...', lat: 21.028..., lng: 105.85... }
```

```javascript
// Reverse: tọa độ → địa chỉ
async function reverseGeocode(lat, lng) {
  const url = `https://nominatim.openstreetmap.org/reverse?lat=${lat}&lon=${lng}&format=json&accept-language=vi`;
  const res = await fetch(url, {
    headers: { 'User-Agent': 'MyApp/1.0 (contact@example.com)' }
  });
  const data = await res.json();
  return data.display_name;
}
```

> ⚠️ **Quy tắc Nominatim:** Tối đa 1 request/giây. Bắt buộc gửi `User-Agent` hoặc `Referer` hợp lệ. Không dùng cho app traffic lớn — khi đó nên tự host Nominatim hoặc dùng provider trả phí.

---

## 15. Routing Miễn Phí (OSRM / Valhalla)

### 15.1. OSRM demo server

```javascript
async function getRoute(start, end) {
  // start/end: [lng, lat]
  const url = `https://router.project-osrm.org/route/v1/driving/${start[0]},${start[1]};${end[0]},${end[1]}?overview=full&geometries=geojson`;
  const res = await fetch(url);
  const data = await res.json();
  return data.routes[0]; // { distance, duration, geometry }
}

// Vẽ lên bản đồ
const route = await getRoute([105.8542, 21.0285], [105.8342, 21.0378]);

map.addSource('route', { type: 'geojson', data: route.geometry });
map.addLayer({
  id: 'route-line', type: 'line', source: 'route',
  paint: { 'line-color': '#3887be', 'line-width': 5 }
});

console.log(`Cách ${(route.distance/1000).toFixed(1)}km, mất ${Math.round(route.duration/60)} phút`);
```

> ⚠️ Demo server của OSRM chỉ cho mục đích thử nghiệm. Production → tự host OSRM hoặc dùng Valhalla.

### 15.2. Các profile OSRM

| Profile | URL |
|---|---|
| Ô tô | `/route/v1/driving/...` |
| Đi bộ | `/route/v1/foot/...` |
| Xe đạp | `/route/v1/bike/...` |

---

## 16. Self-host Tiles

Muốn hoàn toàn tự chủ? Có 3 cách phổ biến:

### 16.1. Martin (Rust, nhẹ, nhanh)

```bash
# Chạy tile server từ file PMTiles
docker run -p 3000:3000 \
  -v /data:/data \
  ghcr.io/maplibre/martin \
  --connection-string "postgres://user:pass@localhost/db"
```

### 16.2. PMTiles trên static hosting

```bash
# Upload 1 file .pmtiles lên S3/R2/GitHub Pages
# Rồi trỏ MapLibre tới:
# pmtiles://https://cdn.example.com/vietnam.pmtiles
```

```javascript
import { PMTiles, Protocol } from 'pmtiles';
const protocol = new Protocol();
maplibregl.addProtocol('pmtiles', protocol.tile);
```

### 16.3. tileserver-gl

```bash
docker run --rm -it -v $(pwd):/data -p 8080:8080 maptiler/tileserver-gl
# Mở http://localhost:8080 → có sẵn style viewer
```

---

## 17. Migrate Từ Mapbox Sang MapLibre

Vì MapLibre fork từ Mapbox GL v1, API gần như giống hệt:

### 7 bước migrate

```bash
# 1. Đổi package
npm uninstall mapbox-gl
npm install maplibre-gl
```

```javascript
// 2. Đổi import
// Trước:
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';

// Sau:
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';
```

```javascript
// 3. Xóa token (không cần nữa!)
// mapboxgl.accessToken = 'pk...';  ← XÓA DÒNG NÀY

// 4. Đổi style sang provider mới
// Trước:
style: 'mapbox://styles/mapbox/streets-v12'
// Sau:
style: 'https://tiles.openfreemap.org/styles/liberty'

// 5. Đổi tên class
// Trước: new mapboxgl.Map(...)
// Sau:   new maplibregl.Map(...)
// Tương tự: Marker, Popup, NavigationControl...

// 6. Mọi thứ khác GIỮ NGUYÊN:
// map.setCenter, setZoom, flyTo, fitBounds
// map.addSource, map.addLayer
// map.on('click', ...)
// ... đều giống hệt!
```

### Những thứ KHÔNG migrate được

| Tính năng Mapbox | Thay thế ở MapLibre |
|---|---|
| `mapbox://` styles | Style JSON từ provider khác |
| Mapbox Geocoding API | Nominatim / Pelias |
| Mapbox Directions API | OSRM / Valhalla |
| Mapbox Studio | Viết style JSON tay / dùng tool open-source |

---

## 18. Tối Ưu Hiệu Suất

- **Clustering** cho điểm nhiều (giống Mapbox, API tương tự)
- **Giới hạn maxBounds** để không tải tiles thừa
- **Dùng vector thay vì raster** khi có thể
- **`map.remove()`** khi unmount
- **PMTiles** cho dataset tĩnh lớn (1 file, cache tốt)

```javascript
// Clustering với MapLibre
map.addSource('diem', {
  type: 'geojson',
  data: geojsonData,
  cluster: true,
  clusterMaxZoom: 14,
  clusterRadius: 50
});
```

---

## 19. Xử Lý Lỗi Thường Gặp

| Lỗi / Triệu chứng | Nguyên nhân | Cách fix |
|---|---|---|
| Bản đồ trắng, không hiện tiles | Style URL sai hoặc provider down | Thử mở style URL trên trình duyệt xem có JSON không |
| `Failed to fetch style` | CORS hoặc URL sai | Kiểm tra URL, thử style khác |
| Chữ không hiện (ô vuông □□□) | Thiếu glyphs URL trong style | Dùng style đầy đủ của OpenFreeMap |
| Icon không hiện | Thiếu sprite URL | Kiểm tra `sprite` trong style JSON |
| Marker sai vị trí | Nhầm `[lat, lng]` | Dùng `[lng, lat]` |
| `demotiles` chậm/không load | Demo server quá tải | Chuyển sang OpenFreeMap |
| Nominatim trả 403 | Thiếu User-Agent | Thêm header `User-Agent` |
| Nominatim chậm | Quá 1 req/s | Giảm tần suất, cache kết quả |

---

## 20. Best Practices & Cheat Sheet

```javascript
// Khởi tạo (không token!)
import maplibregl from 'maplibre-gl';
const map = new maplibregl.Map({
  container: 'map',
  style: 'https://tiles.openfreemap.org/styles/liberty',
  center: [105.8542, 21.0285], // [lng, lat]
  zoom: 13
});

// Điều hướng
map.flyTo({ center: [105.85, 21.02], zoom: 14 });
map.fitBounds([[105.8, 21.0], [105.9, 21.05]], { padding: 40 });

// Marker + popup
new maplibregl.Marker().setLngLat([lng, lat])
  .setPopup(new maplibregl.Popup().setText('Xin chào!'))
  .addTo(map);

// Layer
map.on('load', () => {
  map.addSource('id', { type: 'geojson', data: geojson });
  map.addLayer({ id: 'layer-id', type: 'circle', source: 'id', paint: {...} });
});

// Dọn dẹp
map.remove();
```

**Checklist:**
- [ ] Dùng OpenFreeMap cho production miễn phí (không dùng demotiles)
- [ ] Ghi attribution: `© OpenMapTiles © OpenStreetMap contributors`
- [ ] Thêm `map.remove()` khi unmount
- [ ] Nominatim: gửi User-Agent, tối đa 1 req/s, cache kết quả
- [ ] Nhớ `[lng, lat]`!

---

## 📚 Tài Liệu Tham Khảo

- Trang chủ: https://maplibre.org
- Docs GL JS: https://maplibre.org/maplibre-gl-js/docs/
- GitHub: https://github.com/maplibre/maplibre-gl-js
- OpenFreeMap: https://openfreemap.org
- Style spec: https://maplibre.org/maplibre-style-spec/

---

## 21. Style Expressions — Vẽ Theo Dữ Liệu

Expressions là "ngôn ngữ mini" trong style JSON để style động theo thuộc tính:

```javascript
// Màu thay đổi theo rating
'circle-color': [
  'interpolate', ['linear'], ['get', 'rating'],
  3.0, '#ef4444',  // rating 3 → đỏ
  4.0, '#f59e0b',  // rating 4 → cam
  4.5, '#22c55e'   // rating 4.5+ → xanh
]

// Kích thước theo zoom
'circle-radius': [
  'interpolate', ['linear'], ['zoom'],
  10, 4,   // zoom 10 → r=4
  15, 12   // zoom 15 → r=12
]

// Hiện/ẩn theo loại
'filter': ['==', ['get', 'type'], 'restaurant']

// Text theo điều kiện
'text-field': [
  'case',
  ['>', ['get', 'rating'], 4.5], ['concat', '⭐ ', ['get', 'name']],
  ['get', 'name']
]
```

**Các expression hay dùng:**

| Expression | Ý nghĩa | Ví dụ |
|---|---|---|
| `['get', 'prop']` | Lấy thuộc tính | `['get', 'name']` |
| `['zoom']` | Mức zoom hiện tại | `['interpolate', ..., ['zoom'], ...]` |
| `['interpolate', ...]` | Nội suy tuyến tính | Màu/size theo giá trị |
| `['step', ...]` | Bậc thang | Màu theo ngưỡng |
| `['match', ...]` | So khớp | Filter theo danh sách |
| `['case', ...]` | Điều kiện | if-else |
| `['concat', ...]` | Nối chuỗi | `'⭐ ' + name` |

---

## 22. Terrain 3D & Sky

```javascript
map.on('load', () => {
  // Thêm nguồn địa hình (dùng terrain miễn phí của AWS)
  map.addSource('terrain', {
    type: 'raster-dem',
    tiles: ['https://s3.amazonaws.com/elevation-tiles-prod/terrarium/{z}/{x}/{y}.png'],
    tileSize: 256,
    maxzoom: 15
  });

  // Bật terrain
  map.setTerrain({ source: 'terrain', exaggeration: 1.5 });

  // Thêm bầu trời
  map.setSky({
    'sky-color': '#199EF3',
    'horizon-color': '#ffffff',
    'fog-color': '#ffffff'
  });

  map.setPitch(70); // nghiêng để thấy 3D
});
```

```javascript
// Tắt terrain
map.setTerrain(null);
```

---

## 23. Tích Hợp Framework

### 23.1. Vue 3

```vue
<template>
  <div ref="mapEl" class="map"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';

const mapEl = ref(null);
let map = null;

onMounted(() => {
  map = new maplibregl.Map({
    container: mapEl.value,
    style: 'https://tiles.openfreemap.org/styles/liberty',
    center: [105.8542, 21.0285],
    zoom: 13
  });
});

onUnmounted(() => map?.remove());
</script>

<style>
.map { width: 100%; height: 500px; }
</style>
```

### 23.2. Svelte

```svelte
<script>
  import { onMount, onDestroy } from 'svelte';
  import maplibregl from 'maplibre-gl';
  import 'maplibre-gl/dist/maplibre-gl.css';

  let mapEl, map;

  onMount(() => {
    map = new maplibregl.Map({
      container: mapEl,
      style: 'https://tiles.openfreemap.org/styles/liberty',
      center: [105.8542, 21.0285],
      zoom: 13
    });
  });

  onDestroy(() => map?.remove());
</script>

<div bind:this={mapEl} style="height: 500px;"></div>
```

### 23.3. Flutter (maplibre_gl package)

```dart
import 'package:maplibre_gl/maplibre_gl.dart';

MapLibreMap(
  styleString: 'https://tiles.openfreemap.org/styles/liberty',
  initialCameraPosition: CameraPosition(
    target: LatLng(21.0285, 105.8542), // Flutter: LatLng(lat, lng)!
    zoom: 13.0,
  ),
  onMapCreated: (controller) {
    controller.addSymbol(SymbolOptions(
      geometry: LatLng(21.0285, 105.8542),
      textField: 'Hồ Gươm',
    ));
  },
)
```

> ⚠️ Flutter dùng `LatLng(lat, lng)` — ngược với JS `[lng, lat]`!

---

## 24. PMTiles Chi Tiết

PMTiles là định dạng "1 file chứa cả bản đồ" — cách mạng cho self-host.

### 24.1. Tại sao PMTiles hay?

| Cách cũ (MBTiles + server) | PMTiles |
|---|---|
| Cần server chạy 24/7 | Chỉ cần static hosting (S3, R2, GitHub Pages) |
| Nhiều file nhỏ | 1 file duy nhất |
| Tốn tiền server | Gần như miễn phí (R2 free tier) |

### 24.2. Dùng PMTiles có sẵn

```html
<script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
<script src="https://unpkg.com/pmtiles@3.0.6/dist/pmtiles.js"></script>
<script>
  const protocol = new pmtiles.Protocol();
  maplibregl.addProtocol('pmtiles', protocol.tile);

  const map = new maplibregl.Map({
    container: 'map',
    style: {
      version: 8,
      sources: {
        protomaps: {
          type: 'vector',
          url: 'pmtiles://https://demo-bucket.protomaps.com/v4.pmtiles'
        }
      },
      layers: [
        { id: 'bg', type: 'background', paint: { 'background-color': '#f0f0f0' } }
        // ... thêm layers theo schema Protomaps
      ]
    }
  });
</script>
```

### 24.3. Tự tạo PMTiles từ dữ liệu riêng

```bash
# Cài tippecanoe
sudo apt install tippecanoe

# GeoJSON → PMTiles
tippecanoe -o vietnam.pmtiles --force \
  --minimum-zoom=5 --maximum-zoom=14 \
  du-lieu.geojson

# Upload lên R2/S3 → dùng URL như trên
```

---

## 25. Offline Maps (PWA)

```javascript
// Cache tiles bằng Service Worker (ý tưởng)
// Lưu ý: kiểm tra TOS của tile provider trước khi cache hàng loạt!

// Với PMTiles: tải 1 lần, dùng mãi mãi offline
const pmtilesFile = await fetch('https://cdn.com/vietnam.pmtiles');
const blob = await pmtilesFile.blob();
// Lưu vào IndexedDB / Cache API...
```

---

## 26. So Sánh Chi Tiết Các Tile Provider Free

| Provider | Style URL | Đăng ký | Giới hạn | Tốc độ ở VN | Dùng thương mại |
|---|---|---|---|---|---|
| OpenFreeMap | `tiles.openfreemap.org/styles/liberty` | ❌ | Không | Nhanh (Cloudflare) | ✅ |
| CARTO | `basemaps.cartocdn.com/gl/positron-gl-style/style.json` | ❌* | 1M/tháng | Nhanh | ✅ |
| MapTiler | `api.maptiler.com/...?key=KEY` | ✅ | 100k/tháng | Nhanh | ❌ (free) |
| Stadia | `tiles.stadiamaps.com/styles/...` | ⚠️ | 200k credits | Trung bình | ❌ (free) |
| Esri | `server.arcgisonline.com/...` | ❌ | Không rõ | Trung bình | ⚠️ |

---

## 27. Câu Hỏi Thường Gặp (FAQ)

**Hỏi: MapLibre có hoàn toàn miễn phí không?**
Đáp: Thư viện thì 100% free (BSD). Chi phí duy nhất là tile provider bạn chọn — dùng OpenFreeMap thì $0.

**Hỏi: API của MapLibre và Mapbox khác nhau nhiều không?**
Đáp: Gần như giống hệt cho các tính năng cơ bản (Map, Marker, Popup, addSource, addLayer). Khác ở: không có `accessToken`, style URL khác, không có API riêng của Mapbox.

**Hỏi: Có thể dùng style của Mapbox cho MapLibre không?**
Đáp: Không trực tiếp — style `mapbox://` cần token Mapbox. Nhưng bạn có thể export style JSON từ Mapbox Studio (dạng URL https) rồi sửa sources.

**Hỏi: MapLibre có hỗ trợ 3D không?**
Đáp: Có — `fill-extrusion` cho buildings, `setTerrain` cho địa hình, `setSky` cho bầu trời.

**Hỏi: Nên dùng MapLibre GL JS v4 hay v5?**
Đáp: Dùng bản mới nhất (v5+) cho dự án mới. Chú ý: từ v5, UMD build (CDN script tag) có thể khác — kiểm tra docs.
