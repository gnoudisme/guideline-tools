# Hướng Dẫn Toàn Tập: OpenFreeMap — Tiles Bản Đồ Miễn Phí 100%

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật:** 10/10/2026
>
> 🎉 **Tóm tắt 1 dòng:** Bản đồ vector miễn phí, không cần đăng ký, không cần API key, không giới hạn request — chỉ cần 1 URL!

---

## 📑 Mục Lục

1. [OpenFreeMap Là Gì?](#1-openfreemap-là-gì)
2. [Vì Sao Nó Miễn Phí? (Mô Hình Hoạt Động)](#2-vì-sao-nó-miễn-phí-mô-hình-hoạt-động)
3. [Các Style Có Sẵn](#3-các-style-có-sẵn)
4. [Dùng Với MapLibre GL JS (5 Phút)](#4-dùng-với-maplibre-gl-js-5-phút)
5. [Dùng Với Leaflet](#5-dùng-với-leaflet)
6. [Dùng Với OpenLayers](#6-dùng-với-openlayers)
7. [Tùy Chỉnh Style](#7-tùy-chỉnh-style)
8. [Attribution — Ghi Nguồn Đúng Cách](#8-attribution--ghi-nguồn-đúng-cách)
9. [Self-host OpenFreeMap](#9-self-host-openfreemap)
10. [So Sánh Với Các Lựa Chọn Miễn Phí Khác](#10-so-sánh-với-các-lựa-chọn-miễn-phí-khác)
11. [Giới Hạn & Lưu Ý](#11-giới-hạn--lưu-ý)
12. [Xử Lý Lỗi Thường Gặp](#12-xử-lý-lỗi-thường-gặp)
13. [Best Practices & Cheat Sheet](#13-best-practices--cheat-sheet)

---

## 1. OpenFreeMap Là Gì?

**OpenFreeMap** (https://openfreemap.org) là dịch vụ cung cấp **vector tiles bản đồ miễn phí**, tương thích với MapLibre GL JS. Dữ liệu từ **OpenStreetMap**, render theo schema **OpenMapTiles**.

**Những con số ấn tượng:**
- 💰 **Giá:** 0 đồng — không free tier, không quota, không paywall
- 🔑 **API key:** Không cần — không đăng ký, không tài khoản
- 📊 **Giới hạn request:** Không giới hạn công khai
- 🍪 **Cookies:** Không — không theo dõi người dùng
- 🔄 **Cập nhật dữ liệu:** Hàng tuần từ planet OpenStreetMap
- 📜 **License:** MIT (cả code lẫn dịch vụ đều mở)

### 1.1. Ai đứng sau?

Dự án open-source do **hyperknot** khởi xướng, chạy nhờ **donate** cộng đồng, hạ tầng đặt trên **Cloudflare** (CDN toàn cầu → nhanh ở Việt Nam).

### 1.2. Phù hợp với ai?

- ✅ Dự án cá nhân, portfolio, đồ án
- ✅ Startup muốn tiết kiệm chi phí bản đồ
- ✅ App open-source (không muốn token bí mật trong repo public)
- ✅ Prototype cần bản đồ "mở lên là chạy"
- ⚠️ Cân nhắc kỹ nếu là hệ thống **sống còn** (xem mục 11)

---

## 2. Vì Sao Nó Miễn Phí? (Mô Hình Hoạt Động)

Hiểu mô hình để dùng đúng cách:

```
OpenStreetMap (dữ liệu mở)
        ↓ (weekly planet dump)
OpenFreeMap (render thành vector tiles, host trên Cloudflare)
        ↓ (miễn phí, không key)
   App của bạn (MapLibre GL JS)
```

- **Chi phí hạ tầng** được trả bằng tiền donate + sponsor
- **Không có SLA** (cam kết uptime) — dịch vụ "best effort"
- **Điều khoản sử dụng:** https://openfreemap.org/tos/ — đọc kỹ trước khi dùng production

> 💡 **Nguyên tắc vàng:** Dùng free cho những gì chịu được rủi ro downtime. Nếu bản đồ là "mạch máu" của business → tự host hoặc dùng provider có SLA.

---

## 3. Các Style Có Sẵn

| Style | URL | Mô tả | Preview |
|---|---|---|---|
| **Liberty** | `https://tiles.openfreemap.org/styles/liberty` | Đầy đủ chi tiết, phong cách OSM truyền thống — **khuyên dùng** | Đẹp, quen thuộc |
| **Bright** | `https://tiles.openfreemap.org/styles/bright` | Sáng, gọn, hiện đại | Sạch sẽ |
| **Positron** | `https://tiles.openfreemap.org/styles/positron` | Xám nhạt tối giản — nền hoàn hảo cho data viz | Tối giản |

**Thử nhanh:** Mở URL style trên trình duyệt → thấy JSON là OK.

```javascript
// Đổi style trong code
map.setStyle('https://tiles.openfreemap.org/styles/bright');
map.setStyle('https://tiles.openfreemap.org/styles/positron');
```

---

## 4. Dùng Với MapLibre GL JS (5 Phút)

Đây là combo "quốc dân" cho bản đồ miễn phí.

### 4.1. HTML thuần

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="utf-8" />
  <title>Bản đồ miễn phí với OpenFreeMap</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
  <link href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css" rel="stylesheet" />
  <style>
    body { margin: 0; font-family: sans-serif; }
    #map { width: 100%; height: 100vh; }
    .info {
      position: absolute; top: 10px; left: 10px; z-index: 1;
      background: white; padding: 10px 14px; border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,.15); font-size: 14px;
    }
  </style>
</head>
<body>
  <div id="map"></div>
  <div class="info">🗺️ Bản đồ miễn phí 100% — không token, không đăng ký!</div>
  <script>
    const map = new maplibregl.Map({
      container: 'map',
      style: 'https://tiles.openfreemap.org/styles/liberty',
      center: [105.8542, 21.0285], // Hồ Gươm [lng, lat]
      zoom: 13
    });
    map.addControl(new maplibregl.NavigationControl(), 'top-right');

    // Thêm marker chào mừng
    new maplibregl.Marker({ color: '#e11d48' })
      .setLngLat([105.8542, 21.0285])
      .setPopup(new maplibregl.Popup().setHTML('<b>Hồ Gươm</b><br>Hà Nội 🇻🇳'))
      .addTo(map);
  </script>
</body>
</html>
```

Lưu thành `map.html`, mở bằng trình duyệt → xong!

### 4.2. React

```jsx
import { useEffect, useRef } from 'react';
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';

export default function FreeMap() {
  const ref = useRef(null);

  useEffect(() => {
    const map = new maplibregl.Map({
      container: ref.current,
      style: 'https://tiles.openfreemap.org/styles/liberty',
      center: [105.8542, 21.0285],
      zoom: 13,
      attributionControl: {
        customAttribution: '© OpenMapTiles © OpenStreetMap contributors'
      }
    });
    map.addControl(new maplibregl.NavigationControl());
    return () => map.remove();
  }, []);

  return <div ref={ref} style={{ height: '500px' }} />;
}
```

### 4.3. Thêm dữ liệu của bạn lên trên

```javascript
map.on('load', () => {
  // Ví dụ: vẽ các quán cà phê
  map.addSource('cafes', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: [
        { type: 'Feature', geometry: { type: 'Point', coordinates: [105.8520, 21.0300] }, properties: { name: 'Cà phê Giảng' } },
        { type: 'Feature', geometry: { type: 'Point', coordinates: [105.8490, 21.0250] }, properties: { name: 'Cộng Cà Phê' } }
      ]
    }
  });

  map.addLayer({
    id: 'cafes-points',
    type: 'circle',
    source: 'cafes',
    paint: {
      'circle-radius': 8,
      'circle-color': '#b45309',
      'circle-stroke-width': 2,
      'circle-stroke-color': '#fff'
    }
  });

  map.addLayer({
    id: 'cafes-labels',
    type: 'symbol',
    source: 'cafes',
    layout: {
      'text-field': ['get', 'name'],
      'text-size': 12,
      'text-offset': [0, 1.4]
    }
  });

  map.on('click', 'cafes-points', (e) => {
    new maplibregl.Popup()
      .setLngLat(e.lngLat)
      .setHTML(`<b>${e.features[0].properties.name}</b> ☕`)
      .addTo(map);
  });
});
```

---

## 5. Dùng Với Leaflet

Thích Leaflet hơn? OpenFreeMap cũng có raster tiles:

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<div id="map" style="height: 500px;"></div>

<script>
  const map = L.map('map').setView([21.0285, 105.8542], 13); // Leaflet: [lat, lng]

  // Raster tiles từ OpenFreeMap
  L.tileLayer('https://tiles.openfreemap.org/styles/liberty/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '© OpenMapTiles © OpenStreetMap contributors'
  }).addTo(map);

  L.marker([21.0285, 105.8542]).addTo(map)
    .bindPopup('Hồ Gươm').openPopup();
</script>
```

> 💡 Vector (MapLibre) cho chữ sắc nét khi zoom + tùy biến style. Raster (Leaflet) đơn giản, nhẹ, hợp với bản đồ ít tương tác.

---

## 6. Dùng Với OpenLayers

```javascript
import Map from 'ol/Map';
import View from 'ol/View';
import VectorTileLayer from 'ol/layer/VectorTile';
import VectorTileSource from 'ol/source/VectorTile';
import MVT from 'ol/format/MVT';
import { applyStyle } from 'ol-mapbox-style';

const map = new Map({
  target: 'map',
  view: new View({ center: [11740720, 2388300], zoom: 13 }) // Web Mercator!
});

// Áp style MapLibre vào OpenLayers
fetch('https://tiles.openfreemap.org/styles/liberty')
  .then(r => r.json())
  .then(style => {
    const layer = new VectorTileLayer({ declutter: true });
    map.addLayer(layer);
    applyStyle(layer, style);
  });
```

> ⚠️ OpenLayers dùng tọa độ **Web Mercator (mét)**, không phải `[lng, lat]` — cần convert bằng `ol/proj.fromLonLat([lng, lat])`.

---

## 7. Tùy Chỉnh Style

### 7.1. Tải style về sửa

```bash
curl -s https://tiles.openfreemap.org/styles/liberty -o my-style.json
# Mở my-style.json, sửa màu sắc...
```

Ví dụ đổi màu nước thành xanh ngọc:

```json
{
  "id": "water",
  "type": "fill",
  "source": "openmaptiles",
  "source-layer": "water",
  "paint": {
    "fill-color": "#5ec8d8"
  }
}
```

Rồi host file JSON ở đâu đó (cùng domain, GitHub Pages, S3...) và trỏ tới:

```javascript
style: 'https://cdn-cua-ban.com/my-style.json'
```

> ⚠️ Nếu style của bạn vẫn trỏ `sources` về `tiles.openfreemap.org` thì vẫn dùng tiles free — chỉ phần "công thức vẽ" là của bạn.

### 7.2. Highlight đường phố theo tên

```javascript
map.on('load', () => {
  map.addLayer({
    id: 'highlight-pho',
    type: 'symbol',
    source: 'openmaptiles',
    'source-layer': 'transportation_name',
    filter: ['match', ['get', 'name'], ['Phố Huế', 'Bà Triệu'], true, false],
    layout: {
      'text-field': ['get', 'name'],
      'text-size': 16,
      'text-font': ['Noto Sans Regular']
    },
    paint: {
      'text-color': '#000',
      'text-halo-color': '#ffff00',
      'text-halo-width': 3
    }
  });
});
```

---

## 8. Attribution — Ghi Nguồn Đúng Cách

Bắt buộc (và cũng là phép lịch sự) khi dùng dữ liệu OSM:

```
© OpenMapTiles © OpenStreetMap contributors
```

MapLibre tự hiện attribution từ style JSON. Nếu tắt `attributionControl`, hãy tự thêm:

```html
<div style="position:absolute;bottom:5px;right:5px;background:rgba(255,255,255,.7);font-size:11px;padding:2px 6px;">
  © <a href="https://openmaptiles.org/">OpenMapTiles</a> © <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors
</div>
```

---

## 9. Self-host OpenFreeMap

Muốn tự chủ 100%? Repo chính thức: https://github.com/hyperknot/openfreemap

### 9.1. Tổng quan kiến trúc

```
Planet OSM (.osm.pbf)
   ↓ (planetiler / tilemaker)
Vector tiles (.mbtiles / .pmtiles)
   ↓ (Martin / tileserver-gl / nginx)
Style JSON + fonts + sprites (static files)
   ↓
MapLibre GL JS (client)
```

### 9.2. Cách đơn giản nhất: PMTiles + static hosting

```bash
# 1. Tải planet tiles (file lớn ~100GB+ cho cả thế giới,
#    hoặc extract riêng Việt Nam cho nhẹ)
# 2. Upload lên S3 / R2 / VPS chạy nginx
# 3. Trỏ style tới URL của bạn
```

```javascript
// pmtiles protocol với MapLibre
import { PMTiles, Protocol } from 'pmtiles';
import maplibregl from 'maplibre-gl';

const protocol = new Protocol();
maplibregl.addProtocol('pmtiles', protocol.tile);

const map = new maplibregl.Map({
  container: 'map',
  style: 'https://server-cua-ban.com/styles/liberty.json'
  // trong style.json: "url": "pmtiles://https://server-cua-ban.com/vietnam.pmtiles"
});
```

### 9.3. Khi nào nên self-host?

- ✅ App production quan trọng, cần SLA tự kiểm soát
- ✅ Yêu cầu dữ liệu không qua bên thứ ba
- ✅ Muốn tùy biến sâu tile data
- ❌ Prototype, đồ án, portfolio → dùng public instance cho nhanh

---

## 10. So Sánh Với Các Lựa Chọn Miễn Phí Khác

| Tiêu chí | OpenFreeMap | MapTiler free | CARTO free | Stadia free | Tự host |
|---|---|---|---|---|---|
| Cần đăng ký | ❌ | ✅ | ⚠️ (prod) | ⚠️ (prod) | ❌ |
| Giới hạn request | Không | 100k/tháng | 1M/tháng | 200k credits/tháng | Tự lo |
| Dùng thương mại | ✅ | ❌ (free tier) | ✅ (có quota) | ❌ (free tier) | ✅ |
| SLA/uptime cam kết | ❌ | ✅ (paid) | ✅ | ✅ | Tự lo |
| Cập nhật dữ liệu | Hàng tuần | Thường xuyên | Thường xuyên | Thường xuyên | Tự chạy |
| Chi phí | $0 | $0 → $30+ | $0 → $500+ | $0 → $20+ | Server |
| Phù hợp | Mọi thứ không критик | Dự án nhỏ | Data viz | Thử nghiệm | Production lớn |

---

## 11. Giới Hạn & Lưu Ý

### 11.1. Điều khoản sử dụng (tóm tắt)

- Đọc bản đầy đủ: https://openfreemap.org/tos/
- Dịch vụ cung cấp **"as-is"** — có thể thay đổi hoặc dừng mà không báo trước
- **Không** dùng để scrape/mirror toàn bộ tiles với mục đích tái phân phối
- Luôn ghi attribution

### 11.2. Khi nào KHÔNG nên dùng public instance?

- 🏦 Hệ thống tài chính, y tế, khẩn cấp — cần uptime đảm bảo
- 🚗 Điều hướng real-time cho đội xe — cần SLA
- 📈 App có hàng triệu user — hãy tự host hoặc dùng provider trả phí (cũng là cách "đền đáp" cộng đồng)

### 11.3. Cách dùng có trách nhiệm

- Cache tiles ở client (trình duyệt tự cache theo HTTP headers)
- Đừng F5 liên tục để "test" — mỗi lần tải lại là 1 đống request
- Nếu app lớn dần → lên kế hoạch self-host sớm

---

## 12. Xử Lý Lỗi Thường Gặp

| Lỗi / Triệu chứng | Nguyên nhân | Cách fix |
|---|---|---|
| Bản đồ trắng trơn | Style URL sai | Mở URL style trên trình duyệt, phải thấy JSON |
| `Failed to fetch` | Mất mạng / provider down | Thử lại, kiểm tra https://openfreemap.org |
| Chữ hiện ô vuông □ | Font glyphs lỗi | Dùng nguyên style gốc, đừng sửa `glyphs` bừa |
| Icon mất | Sprite URL sai | Kiểm tra `sprite` trong style JSON |
| Tiles mờ khi zoom sâu | Hết maxzoom của tiles | Giới hạn `maxZoom: 14` cho vector |
| Chậm lần đầu | Tải tiles + fonts lần đầu | Bình thường — lần sau cache sẽ nhanh |
| CORS error khi tự host style | Thiếu header CORS | Thêm `Access-Control-Allow-Origin: *` ở server |

---

## 13. Best Practices & Cheat Sheet

```javascript
// Combo quốc dân: MapLibre + OpenFreeMap
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';

const map = new maplibregl.Map({
  container: 'map',
  style: 'https://tiles.openfreemap.org/styles/liberty', // hoặc bright / positron
  center: [105.8542, 21.0285], // [lng, lat]
  zoom: 13
});
map.addControl(new maplibregl.NavigationControl());

// Đổi style
map.setStyle('https://tiles.openfreemap.org/styles/positron');

// Dọn dẹp
map.remove();
```

**Checklist:**
- [ ] Ghi attribution đầy đủ
- [ ] Không dùng `demotiles.maplibre.org` cho production
- [ ] Đọc TOS trước khi dùng cho app thương mại lớn
- [ ] Có kế hoạch B (self-host hoặc provider dự phòng) nếu app quan trọng
- [ ] Ủng hộ/donate nếu dự án của bạn kiếm được tiền từ bản đồ free ❤️

---

## 📚 Tài Liệu Tham Khảo

- Trang chủ: https://openfreemap.org
- Điều khoản: https://openfreemap.org/tos/
- GitHub (self-host): https://github.com/hyperknot/openfreemap
- MapLibre GL JS: https://maplibre.org
- OpenMapTiles schema: https://openmaptiles.org/schema/

---

## 14. Tích Hợp Với Các Framework

### 14.1. Vue 3

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
  map.addControl(new maplibregl.NavigationControl());
});

onUnmounted(() => map?.remove());
</script>

<style>
.map { width: 100%; height: 500px; }
</style>
```

### 14.2. Next.js (App Router)

```jsx
// components/FreeMap.jsx
'use client';
import { useEffect, useRef } from 'react';
import maplibregl from 'maplibre-gl';
import 'maplibre-gl/dist/maplibre-gl.css';

export default function FreeMap() {
  const ref = useRef(null);
  useEffect(() => {
    const map = new maplibregl.Map({
      container: ref.current,
      style: 'https://tiles.openfreemap.org/styles/liberty',
      center: [105.8542, 21.0285],
      zoom: 13
    });
    return () => map.remove();
  }, []);
  return <div ref={ref} style={{ height: '500px' }} />;
}
```

> 💡 Không cần biến môi trường cho token — vì không có token!

---

## 15. Ví Dụ Thực Tế: Bản Đồ Quán Ăn Hà Nội

Ví dụ hoàn chỉnh: load danh sách quán từ API, hiển thị, click xem chi tiết, tìm quán gần nhất.

```html
<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="utf-8" />
<title>Quán ngon Hà Nội</title>
<script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
<link href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css" rel="stylesheet" />
<style>
  body { margin: 0; font-family: system-ui; }
  #map { height: 100vh; }
  #panel {
    position: absolute; top: 10px; right: 10px; z-index: 1;
    background: white; border-radius: 12px; padding: 14px;
    width: 280px; box-shadow: 0 4px 16px rgba(0,0,0,.15);
  }
  #panel h3 { margin: 0 0 8px; }
  #panel button {
    width: 100%; padding: 10px; margin-top: 8px;
    background: #e11d48; color: white; border: none;
    border-radius: 8px; cursor: pointer; font-size: 14px;
  }
</style>
</head>
<body>
<div id="map"></div>
<div id="panel">
  <h3>🍜 Quán ngon Hà Nội</h3>
  <div id="count">Đang tải...</div>
  <button id="nearby">📍 Tìm quán gần tôi</button>
</div>

<script>
const QUANS = [
  { name: 'Phở Bò Nam Định', lng: 105.8542, lat: 21.0285, rating: 4.8, price: '45k' },
  { name: 'Bún Chả Hương Liên', lng: 105.8510, lat: 21.0240, rating: 4.7, price: '40k' },
  { name: 'Cà Phê Giảng', lng: 105.8520, lat: 21.0300, rating: 4.9, price: '30k' },
  { name: 'Bánh Mì Pate', lng: 105.8600, lat: 21.0320, rating: 4.6, price: '25k' }
];

const map = new maplibregl.Map({
  container: 'map',
  style: 'https://tiles.openfreemap.org/styles/liberty',
  center: [105.8542, 21.0285],
  zoom: 14
});

map.on('load', () => {
  map.addSource('quans', {
    type: 'geojson',
    data: {
      type: 'FeatureCollection',
      features: QUANS.map(q => ({
        type: 'Feature',
        geometry: { type: 'Point', coordinates: [q.lng, q.lat] },
        properties: q
      }))
    }
  });

  map.addLayer({
    id: 'quans-layer', type: 'circle', source: 'quans',
    paint: {
      'circle-radius': 10,
      'circle-color': '#e11d48',
      'circle-stroke-width': 3,
      'circle-stroke-color': '#fff'
    }
  });

  map.addLayer({
    id: 'quans-labels', type: 'symbol', source: 'quans',
    layout: { 'text-field': ['get', 'name'], 'text-size': 11, 'text-offset': [0, 1.5] }
  });

  document.getElementById('count').textContent = `Có ${QUANS.length} quán`;

  map.on('click', 'quans-layer', (e) => {
    const q = e.features[0].properties;
    new maplibregl.Popup()
      .setLngLat(e.lngLat)
      .setHTML(`<b>${q.name}</b><br>⭐ ${q.rating} · 💰 ${q.price}`)
      .addTo(map);
  });
});

// Tìm quán gần nhất (dùng công thức Haversine, không cần API!)
document.getElementById('nearby').onclick = () => {
  navigator.geolocation.getCurrentPosition((pos) => {
    const { latitude: lat, longitude: lng } = pos.coords;
    let best = null, bestD = Infinity;
    QUANS.forEach(q => {
      const d = Math.hypot(q.lng - lng, q.lat - lat);
      if (d < bestD) { bestD = d; best = q; }
    });
    map.flyTo({ center: [best.lng, best.lat], zoom: 16 });
    new maplibregl.Popup()
      .setLngLat([best.lng, best.lat])
      .setHTML(`<b>Gần bạn nhất:</b><br>${best.name} ⭐ ${best.rating}`)
      .addTo(map);
  }, () => alert('Không lấy được vị trí 😢'));
};
</script>
</body>
</html>
```

---

## 16. Câu Hỏi Thường Gặp (FAQ)

**Hỏi: OpenFreeMap có thật sự miễn phí mãi mãi không?**
Đáp: Hiện tại là vậy (donation-funded). Không ai đảm bảo "mãi mãi" — đó là lý do nên có kế hoạch B (self-host) cho app quan trọng.

**Hỏi: Dùng cho app thương mại có được không?**
Đáp: Được — không có điều khoản cấm thương mại như MapTiler free tier. Nhưng hãy donate nếu app kiếm ra tiền, và đọc kỹ TOS.

**Hỏi: Tốc độ ở Việt Nam thế nào?**
Đáp: Nhanh — chạy trên Cloudflare CDN có PoP ở Việt Nam.

**Hỏi: Có hỗ trợ tiếng Việt cho tên địa danh không?**
Đáp: Có — dữ liệu OSM có `name:vi`. Style Liberty hiển thị theo ngôn ngữ trình duyệt.

**Hỏi: Khác gì so với dùng tiles.openstreetmap.org trực tiếp?**
Đáp: `tile.openstreetmap.org` là **raster** (ảnh PNG) — cấm dùng nặng (tile usage policy nghiêm). OpenFreeMap là **vector** — nhẹ hơn, đẹp hơn, không giới hạn.

**Hỏi: Có thể dùng offline không?**
Đáp: Có — tải planet MBTiles về, self-host bằng Martin/tileserver-gl, hoặc convert sang PMTiles.
