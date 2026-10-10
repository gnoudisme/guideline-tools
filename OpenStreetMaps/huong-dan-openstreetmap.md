# Hướng Dẫn Toàn Tập: OpenStreetMap

> **Cấp độ:** Người mới bắt đầu → dùng được thật
> **Ngôn ngữ:** Tiếng Việt
> **Cập nhật lần cuối:** 10/10/2026

---

## Mục Lục

1. [OpenStreetMap là gì?](#1-openstreetmap-là-gì)
2. [So sánh OSM với Google Maps, Mapbox](#2-so-sánh-osm-với-google-maps-mapbox)
3. [Dùng website openstreetmap.org](#3-dùng-website-openstreetmaporg)
4. [Tạo tài khoản](#4-tạo-tài-khoản)
5. [Đóng góp dữ liệu: chỉnh sửa bản đồ](#5-đóng-góp-dữ-liệu-chỉnh-sửa-bản-đồ)
6. [iD Editor — sửa bản đồ trên trình duyệt](#6-id-editor--sửa-bản-đồ-trên-trình-duyệt)
7. [JOSM — editor nâng cao](#7-josm--editor-nâng-cao)
8. [Tags — ngôn ngữ của OSM](#8-tags--ngôn-ngữ-của-osm)
9. [Overpass API — truy vấn dữ liệu](#9-overpass-api--truy-vấn-dữ-liệu)
10. [Nominatim — geocoding miễn phí](#10-nominatim--geocoding-miễn-phí)
11. [Hiển thị bản đồ với Leaflet](#11-hiển-thị-bản-đồ-với-leaflet)
12. [Tile server & các kiểu bản đồ](#12-tile-server--các-kiểu-bản-đồ)
13. [License ODbL — điều cần biết](#13-license-odbl--điều-cần-biết)
14. [Troubleshooting](#14-troubleshooting)
15. [Best practices & Cheat sheet](#15-best-practices--cheat-sheet)

---

## 1. OpenStreetMap Là Gì?

### 1.1. Định nghĩa

**OpenStreetMap (OSM)** là bản đồ **miễn phí, mở, do cộng đồng xây dựng** — người ta hay gọi là "Wikipedia của bản đồ". Ra đời năm 2004 tại Anh, đến nay OSM có hơn **10 triệu người đóng góp** và dữ liệu bao phủ toàn thế giới, chi tiết đến từng con hẻm, cây ATM, ghế đá công viên.

Khác với Google Maps (dữ liệu độc quyền của Google), **dữ liệu OSM thuộc về cộng đồng** và ai cũng được dùng miễn phí theo giấy phép ODbL.

### 1.2. Dữ liệu OSM gồm 3 loại đối tượng

| Loại | Tiếng Anh | Ví dụ |
|------|-----------|-------|
| Điểm | **Node** | 1 quán cà phê, 1 cây ATM, 1 trạm xe buýt |
| Đường | **Way** | 1 con đường, 1 con sông, ranh giới 1 tòa nhà |
| Quan hệ | **Relation** | Tuyến xe buýt (gồm nhiều đoạn đường), ranh giới tỉnh |

Mỗi đối tượng gắn **tags** — cặp `key=value` mô tả nó là gì. Ví dụ quán phở:

```
name=Phở Hòa Pasteur
amenity=restaurant
cuisine=pho
opening_hours=06:00-22:00
phone=+84 28 3829 7943
```

### 1.3. Ai dùng OSM?

- **Facebook/Meta**, **Apple Maps** (1 phần), **Uber**, **Grab** — đều dùng dữ liệu OSM
- **Strava, Komoot** — app thể thao
- **Humanitarian OpenStreetMap Team (HOT)** — cứu trợ thiên tai
- Hàng nghìn website/app nhúng bản đồ miễn phí qua Leaflet + OSM tiles

---

## 2. So Sánh OSM Với Google Maps, Mapbox

| Tiêu chí | OpenStreetMap | Google Maps Platform | Mapbox |
|----------|---------------|---------------------|--------|
| Giá dữ liệu | **Miễn phí 100%** | $200 credit/tháng, sau đó trả tiền | Free tier 50k request, sau đó trả tiền |
| Giấy phép | ODbL (mở) | Độc quyền Google | Độc quyền Mapbox |
| Độ chi tiết VN | Rất tốt ở thành phố lớn, hẻm nhỏ | Tốt nhất tổng thể | Dựa trên OSM + dữ liệu riêng |
| Chỉ đường | Có (qua OSRM, GraphHopper) | Tốt nhất, realtime traffic | Tốt |
| Geocoding | Nominatim (miễn phí, giới hạn 1 req/s) | Tốt nhất | Tốt |
| Vệ tinh | Không có (chỉ bản đồ vector/raster) | Có | Có |
| Offline | Tải dữ liệu raw thoải mái | Hạn chế | Có (mobile SDK) |
| Sửa dữ liệu | **Ai cũng sửa được** | Báo cáo, chờ duyệt | Không |

**Khi nào chọn OSM:**
- Muốn miễn phí hoàn toàn, không lo hóa đơn
- Cần tùy biến style bản đồ
- Làm app cộng đồng, phi lợi nhuận, nghiên cứu
- Cần dữ liệu thô để phân tích (routing, GIS)

**Khi nào chọn Google Maps:**
- Cần traffic realtime, địa điểm kinh doanh đầy đủ nhất
- Khách hàng doanh nghiệp, sẵn sàng trả tiền

---

## 3. Dùng Website openstreetmap.org

### 3.1. Tìm kiếm địa điểm

1. Mở https://www.openstreetmap.org
2. Gõ vào ô tìm kiếm, ví dụ: `Hồ Gươm, Hà Nội`
3. Kết quả hiện ra — bấm vào để xem chi tiết, tọa độ

### 3.2. Các lớp bản đồ (Layers)

Bấm nút **Lớp** (bên phải màn hình) để đổi kiểu hiển thị:

| Lớp | Mô tả |
|-----|-------|
| Standard | Bản đồ mặc định |
| Cycle Map | Nhấn mạnh đường xe đạp, địa hình |
| Transport Map | Nhấn mạnh giao thông công cộng |
| Hot | Kiểu của Humanitarian team |

### 3.3. Lấy tọa độ 1 điểm

1. Chuột phải vào điểm trên bản đồ → **Hiển thị địa chỉ**
2. Hoặc nhìn URL: `https://www.openstreetmap.org/#map=15/21.0285/105.8542`
   - `15` = mức zoom, `21.0285` = vĩ độ, `105.8542` = kinh độ

### 3.4. Đo khoảng cách

1. Chuột phải → **Đo khoảng cách**
2. Click các điểm trên bản đồ → hiện tổng km

### 3.5. Chia sẻ vị trí

Bấm nút **Chia sẻ** → copy link, hoặc lấy code HTML nhúng:

```html
<iframe width="425" height="350"
  src="https://www.openstreetmap.org/export/embed.html?bbox=105.84%2C21.02%2C105.87%2C21.04&layer=mapnik"
  style="border: 1px solid black">
</iframe>
```

---

## 4. Tạo Tài Khoản

1. Vào https://www.openstreetmap.org → **Đăng ký**
2. Điền email, tên hiển thị, mật khẩu
3. Xác nhận email
4. Xong! Giờ bạn có thể sửa bản đồ, lưu địa điểm yêu thích

> 💡 **Mẹo:** Tên hiển thị sẽ gắn với mọi chỉnh sửa của bạn — chọn tên nghiêm túc nếu muốn đóng góp lâu dài.

---

## 5. Đóng Góp Dữ Liệu: Chỉnh Sửa Bản Đồ

### 5.1. Tại sao nên đóng góp?

- Quán ăn gần nhà chưa có trên bản đồ → thêm vào, mọi người cùng dùng
- Đường mới mở chưa ai vẽ → bạn vẽ đầu tiên
- Dữ liệu VN còn thiếu ở nông thôn → bạn giúp cải thiện

### 5.2. Quy tắc vàng khi sửa bản đồ

1. **Không copy từ Google Maps** — vi phạm bản quyền! Chỉ dùng nguồn mở hoặc tự khảo sát
2. **Vẽ đúng thực tế** — đừng vẽ đường tưởng tượng
3. **Tag đầy đủ** — càng nhiều thông tin càng tốt
4. **Chia nhỏ changeset** — mỗi lần sửa 1 khu vực, ghi chú rõ ràng
5. **Không xóa bừa** — nếu không chắc, hỏi cộng đồng trước

### 5.3. Nguồn ảnh vệ tinh được phép dùng

Trong editor, bạn được dùng các lớp ảnh:
- **Bing aerial** — ảnh vệ tinh Bing
- **Esri World Imagery** — ảnh Esri
- **Maxar** — ảnh độ phân giải cao

> ⚠️ **Tuyệt đối không** đồ lại từ Google Maps / Google Earth — dữ liệu Google có bản quyền.

---

## 6. iD Editor — Sửa Bản Đồ Trên Trình Duyệt

iD là editor mặc định, chạy ngay trên web, dễ nhất cho người mới.

### 6.1. Bắt đầu sửa

1. Đăng nhập openstreetmap.org
2. Zoom vào khu vực muốn sửa
3. Bấm nút **Sửa** (hoặc phím `E`)
4. iD editor mở ra

### 6.2. Thêm 1 địa điểm (điểm POI)

**Ví dụ: thêm quán cà phê mới mở gần nhà**

1. Bấm nút **Điểm** (hoặc phím `1`)
2. Click vào vị trí quán trên bản đồ
3. Panel bên trái hiện ra → tìm kiếm loại địa điểm: gõ `cafe`
4. Chọn **Quán cà phê**
5. Điền thông tin:
   - Tên: `Cộng Cà Phê`
   - Giờ mở cửa: `07:00-23:00`
   - Số điện thoại, website (nếu biết)
6. Bấm **Lưu** → ghi chú changeset: `Thêm quán Cộng Cà Phê mới mở`
7. Bấm **Tải lên**

### 6.3. Vẽ 1 con đường mới

1. Bấm nút **Đường** (hoặc phím `2`)
2. Click từng điểm dọc theo con đường (theo ảnh vệ tinh)
3. Click đúp để kết thúc
4. Panel trái → chọn loại đường: `Đường nhỏ`, `Đường chính`...
5. Điền tên đường nếu biết
6. **Lưu** → ghi chú → **Tải lên**

### 6.4. Vẽ 1 tòa nhà

1. Bấm nút **Vùng** (hoặc phím `3`)
2. Click các góc của tòa nhà, khép kín lại
3. Chọn loại: `Tòa nhà`
4. Có thể bấm `S` để vuông góc các góc (tòa nhà thường vuông)
5. Điền tên, số tầng, địa chỉ
6. **Lưu** → **Tải lên**

### 6.5. Sửa thông tin có sẵn

1. Click vào đối tượng trên bản đồ
2. Panel trái hiện thông tin → sửa trực tiếp
3. **Lưu** → **Tải lên**

### 6.7. Phím tắt iD editor

| Phím | Chức năng |
|------|-----------|
| `1` / `2` / `3` | Thêm điểm / đường / vùng |
| `S` | Vuông góc hình vẽ |
| `C` | Nối 2 đường |
| `X` | Cắt đường tại điểm |
| `Ctrl+Z` | Hoàn tác |
| `Esc` | Hủy thao tác |

---

## 7. JOSM — Editor Nâng Cao

JOSM (Java OpenStreetMap Editor) là phần mềm cài trên máy, mạnh hơn iD nhiều — dành cho người đóng góp thường xuyên.

### 7.1. Cài đặt

```
# Ubuntu/Debian
sudo apt install josm

# Hoặc tải từ https://josm.openstreetmap.de
# Cần Java 11+ đã cài sẵn
java -version
```

### 7.2. Khi nào dùng JOSM thay vì iD?

| iD (web) | JOSM (desktop) |
|----------|----------------|
| Sửa nhanh vài điểm | Sửa hàng loạt, import dữ liệu |
| Người mới | Người có kinh nghiệm |
| Không cần cài | Cần cài Java + JOSM |
| Ít plugin | Hàng trăm plugin (building_tools, utilsplugin2...) |

### 7.3. Quy trình cơ bản

1. Mở JOSM → **File** → **Download from OSM** → chọn vùng
2. Thêm lớp ảnh: **Imagery** → **Bing aerial**
3. Vẽ/sửa như iD nhưng nhanh hơn nhiều (phím tắt mạnh)
4. **Upload** → ghi chú changeset → gửi

### 7.4. Plugin nên cài

- **building_tools** — vẽ tòa nhà 1 click
- **utilsplugin2** — hàng loạt công cụ tiện ích
- **Todo list** — quản lý danh sách việc cần sửa

---

## 8. Tags — Ngôn Ngữ Của OSM

### 8.1. Tags phổ biến nhất

**Địa điểm ăn uống:**
```
amenity=restaurant + cuisine=pho        → quán phở
amenity=cafe                            → quán cà phê
amenity=fast_food + cuisine=burger      → quán burger
amenity=bar                             → quán bar
```

**Mua sắm:**
```
shop=convenience + name=Circle K       → cửa hàng tiện lợi
shop=supermarket                       → siêu thị
shop=mall                              → trung tâm thương mại
amenity=atm                            → cây ATM
amenity=bank                           → ngân hàng
```

**Giao thông:**
```
highway=bus_stop + name=Bến xe Mỹ Đình → trạm xe buýt
amenity=parking                        → bãi đỗ xe
amenity=fuel                           → cây xăng
aeroway=aerodrome + name=Nội Bài       → sân bay
```

**Y tế, giáo dục:**
```
amenity=hospital                       → bệnh viện
amenity=pharmacy                       → nhà thuốc
amenity=school                         → trường học
amenity=university                     → đại học
```

**Du lịch:**
```
tourism=hotel                          → khách sạn
tourism=attraction                     → điểm tham quan
tourism=museum                         → bảo tàng
leisure=park                           → công viên
```

### 8.2. Tags địa chỉ (rất quan trọng ở VN)

```
addr:housenumber=123
addr:street=Phố Huế
addr:city=Hà Nội
addr:postcode=100000
```

### 8.3. Tra cứu tag

- Wiki chính: https://wiki.openstreetmap.org/wiki/Map_features
- Công cụ Taginfo: https://taginfo.openstreetmap.org — xem tag nào được dùng nhiều nhất

---

## 9. Overpass API — Truy Vấn Dữ Liệu

Overpass API cho phép truy vấn dữ liệu OSM bằng ngôn ngữ Overpass QL — cực mạnh cho phân tích.

### 9.1. Dùng thử nhanh

Vào https://overpass-turbo.eu → dán query → bấm **Run**.

### 9.2. Ví dụ 1: Tìm tất cả quán cà phê ở Quận 1, TP.HCM

```overpass
[out:json][timeout:25];
area["name"="Quận 1"]["admin_level"=9]->.q1;
(
  node["amenity"="cafe"](area.q1);
  way["amenity"="cafe"](area.q1);
);
out center;
```

### 9.3. Ví dụ 2: Tìm cây ATM trong bán kính 1km quanh Hồ Gươm

```overpass
[out:json][timeout:25];
node["amenity"="atm"](around:1000, 21.0285, 105.8542);
out;
```

### 9.4. Ví dụ 3: Tất cả bệnh viện ở Hà Nội

```overpass
[out:json][timeout:60];
area["name"="Hà Nội"]->.hn;
(
  node["amenity"="hospital"](area.hn);
  way["amenity"="hospital"](area.hn);
);
out center;
```

### 9.5. Ví dụ 4: Đếm số quán phở ở Đà Nẵng

```overpass
[out:json][timeout:25];
area["name"="Đà Nẵng"]->.dn;
nwr["amenity"="restaurant"]["cuisine"="pho"](area.dn);
out count;
```

### 9.6. Gọi Overpass từ Python

```python
import requests

query = """
[out:json][timeout:25];
node["amenity"="cafe"](around:500, 21.0285, 105.8542);
out;
"""

resp = requests.get(
    "https://overpass-api.de/api/interpreter",
    params={"data": query},
    timeout=60,
)
data = resp.json()

for el in data["elements"]:
    name = el.get("tags", {}).get("name", "không tên")
    print(f"{name}: {el['lat']}, {el['lon']}")
```

**Output mẫu:**
```
Cộng Cà Phê: 21.0291, 105.8538
Highlands Coffee: 21.0279, 105.8549
...
```

### 9.7. Giới hạn sử dụng

- Không gửi quá nhiều request liên tục (bị ban IP tạm thời)
- Query nặng: tăng `[timeout:60]` nhưng đừng lạm dụng
- Cần dữ liệu lớn → tải file `.osm.pbf` cả nước thay vì query

---

## 10. Nominatim — Geocoding Miễn Phí

Nominatim là dịch vụ geocoding (địa chỉ ↔ tọa độ) chạy trên dữ liệu OSM.

### 10.1. Geocoding: địa chỉ → tọa độ

```python
import requests

def geocode(dia_chi):
    resp = requests.get(
        "https://nominatim.openstreetmap.org/search",
        params={"q": dia_chi, "format": "json", "limit": 1,
                "countrycodes": "vn"},
        headers={"User-Agent": "ung-dung-cua-toi/1.0"},
        timeout=10,
    )
    r = resp.json()
    return (float(r[0]["lat"]), float(r[0]["lon"])) if r else None

print(geocode("Hồ Gươm, Hà Nội"))
# (21.028511, 105.854174)
```

### 10.2. Reverse geocoding: tọa độ → địa chỉ

```python
import requests

def reverse(lat, lon):
    resp = requests.get(
        "https://nominatim.openstreetmap.org/reverse",
        params={"lat": lat, "lon": lon, "format": "json"},
        headers={"User-Agent": "ung-dung-cua-toi/1.0"},
        timeout=10,
    )
    return resp.json().get("display_name")

print(reverse(21.0285, 105.8542))
# "Hồ Hoàn Kiếm, Hàng Bạc, Hoàn Kiếm, Hà Nội, Việt Nam"
```

### 10.3. Quy tắc sử dụng (bắt buộc)

| Quy tắc | Chi tiết |
|---------|----------|
| User-Agent | **Bắt buộc** gửi header `User-Agent` định danh app của bạn |
| Tốc độ | Tối đa **1 request/giây** |
| Cache | Nên cache kết quả, đừng query lại địa chỉ đã có |
| Không dùng nặng | App production lớn → tự host Nominatim hoặc dùng dịch vụ trả phí |

> ⚠️ Không gửi User-Agent hoặc spam request → IP bị chặn!

---

## 11. Hiển Thị Bản Đồ Với Leaflet

Leaflet là thư viện JavaScript nhẹ nhất để nhúng bản đồ OSM vào website — miễn phí, không cần API key.

### 11.1. Bản đồ cơ bản

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Bản đồ với OSM + Leaflet</title>
  <link rel="stylesheet"
    href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>
    #map { height: 100vh; }
    body { margin: 0; }
  </style>
</head>
<body>
  <div id="map"></div>
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    // Tạo bản đồ, tâm ở Hồ Gươm, zoom 15
    const map = L.map('map').setView([21.0285, 105.8542], 15);

    // Thêm lớp tiles OSM chuẩn
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
    }).addTo(map);

    // Cắm marker
    L.marker([21.0285, 105.8542])
      .addTo(map)
      .bindPopup('<b>Hồ Gươm</b><br>Trái tim Hà Nội')
      .openPopup();
  </script>
</body>
</html>
```

### 11.2. Thêm nhiều marker từ dữ liệu

```html
<script>
  const map = L.map('map').setView([21.0285, 105.8542], 14);
  L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '© OpenStreetMap contributors'
  }).addTo(map);

  // Danh sách quán cà phê
  const cafes = [
    { name: 'Cộng Cà Phê', lat: 21.0291, lon: 105.8538 },
    { name: 'Highlands',   lat: 21.0279, lon: 105.8549 },
    { name: 'The Coffee House', lat: 21.0302, lon: 105.8521 },
  ];

  cafes.forEach(c => {
    L.marker([c.lat, c.lon])
      .addTo(map)
      .bindPopup(`<b>${c.name}</b>`);
  });
</script>
```

### 11.3. Vẽ đường đi (polyline)

```html
<script>
  // Tuyến đường mẫu
  const route = [
    [21.0285, 105.8542],  // Hồ Gươm
    [21.0278, 105.8520],  // đi bộ qua phố
    [21.0245, 105.8412],  // Văn Miếu
  ];

  L.polyline(route, { color: 'blue', weight: 5 })
    .addTo(map)
    .bindPopup('Tuyến đi bộ: Hồ Gươm → Văn Miếu');
</script>
```

### 11.4. Vẽ vùng (polygon)

```html
<script>
  // Khoanh vùng công viên
  const park = [
    [21.0300, 105.8500],
    [21.0320, 105.8500],
    [21.0320, 105.8550],
    [21.0300, 105.8550],
  ];

  L.polygon(park, { color: 'green', fillOpacity: 0.3 })
    .addTo(map)
    .bindPopup('Công viên');
</script>
```

### 11.5. Đổi kiểu bản đồ (tile khác)

```javascript
// Bản đồ giao thông (cần ghi attribution tương ứng)
L.tileLayer('https://tile.openstreetmap.fr/hot/{z}/{x}/{y}.png', {
  attribution: '© OpenStreetMap contributors, Tiles style by HOT'
}).addTo(map);

// Bản đồ xe đạp
L.tileLayer('https://tile-cyclosm.openstreetmap.fr/cyclosm/{z}/{x}/{y}.png', {
  attribution: '© OpenStreetMap contributors, CyclOSM'
}).addTo(map);
```

### 11.6. Quy tắc dùng tile.openstreetmap.org

| Quy tắc | Chi tiết |
|---------|----------|
| Ghi attribution | **Bắt buộc** hiển thị `© OpenStreetMap contributors` |
| Không dùng nặng | Tile server chính chỉ cho demo/app nhỏ |
| App lớn | Tự host tile server hoặc dùng nhà cung cấp (Mapbox, Stadia...) |

---

## 12. Tile Server & Các Kiểu Bản Đồ

### 12.1. Tile là gì?

Bản đồ web được cắt thành ô vuông nhỏ 256×256 pixel (gọi là **tile**), trình duyệt tải các ô cần thiết theo mức zoom. URL dạng:

```
https://tile.openstreetmap.org/{z}/{x}/{y}.png
```

- `z` = mức zoom (0: cả thế giới, 19: từng tòa nhà)
- `x`, `y` = tọa độ ô

### 12.2. Các tile server miễn phí phổ biến

| Tile server | URL | Đặc điểm |
|-------------|-----|----------|
| OSM Standard | `tile.openstreetmap.org` | Mặc định, chỉ demo nhẹ |
| HOT | `tile.openstreetmap.fr/hot` | Kiểu nhân đạo, màu ấm |
| CyclOSM | `tile-cyclosm.openstreetmap.fr/cyclosm` | Cho xe đạp |
| CartoDB Positron | `basemaps.cartocdn.com/light_all` | Sáng, tối giản |
| CartoDB Dark Matter | `basemaps.cartocdn.com/dark_all` | Tối, hợp dashboard |
| Stadia Maps | cần API key miễn phí | Nhiều style đẹp |

### 12.3. Tự host tile server (nâng cao)

Khi app lớn, nên tự host:
1. Tải file `.osm.pbf` Việt Nam từ https://download.geofabrik.de/asia/vietnam.html (~300MB)
2. Dùng **TileServer GL** để serve vector tiles:

```bash
docker run --rm -it -v $(pwd):/data -p 8080:80 \
  maptiler/tileserver-gl --file /data/vietnam.osm.pbf
```

3. Mở `http://localhost:8080` → có ngay tile server riêng!

---

## 13. License ODbL — Điều Cần Biết

### 13.1. ODbL là gì?

Dữ liệu OSM dùng giấy phép **Open Database License (ODbL)**. Tóm tắt dễ hiểu:

| Được phép | Điều kiện |
|-----------|-----------|
| Dùng miễn phí, kể cả thương mại | ✅ |
| Sửa đổi, kết hợp dữ liệu | ✅ |
| Giữ kín app của bạn | ✅ (không bắt open-source code) |

**Nhưng bắt buộc:**
1. **Ghi công (Attribution):** hiển thị `© OpenStreetMap contributors` + link về openstreetmap.org/copyright
2. **Share-Alike:** nếu bạn **cải thiện dữ liệu bản đồ** (ví dụ sửa đường, thêm địa điểm) rồi phân phối, bạn phải chia sẻ lại cải thiện đó dưới ODbL

### 13.2. Ví dụ thực tế

- ✅ **App giao đồ ăn** dùng bản đồ OSM nền → chỉ cần ghi attribution, không cần share code app
- ✅ **Vẽ thêm 100 quán ăn** vào OSM để app mình đẹp hơn → nên đóng góp lại lên OSM (đúng tinh thần share-alike)
- ❌ Lấy dữ liệu OSM, xóa attribution, bảo là dữ liệu của mình → vi phạm

### 13.3. Attribution chuẩn

```
© OpenStreetMap contributors
```

Kèm link: https://www.openstreetmap.org/copyright

Trong Leaflet, attribution đã có sẵn trong code mẫu ở mục 11.

---

## 14. Troubleshooting

| Vấn đề | Nguyên nhân | Cách khắc phục |
|--------|-------------|----------------|
| Sửa xong không thấy hiện | Tile cache chưa cập nhật | Đợi vài phút đến vài giờ; bấm Ctrl+F5 |
| Overpass query timeout | Vùng quá lớn / query nặng | Thu hẹp vùng, thêm `[timeout:60]`, chia nhỏ query |
| Nominatim 403 Forbidden | Thiếu User-Agent | Thêm header `User-Agent` định danh app |
| Nominatim bị chặn IP | Gửi quá 1 req/s | Giảm tốc độ, thêm cache |
| Leaflet bản đồ xám | Sai URL tile / mất mạng | Kiểm tra URL, F12 xem lỗi network |
| Marker không hiện | Sai thứ tự lat/lon | Leaflet dùng `[lat, lon]` — đừng đảo! |
| iD editor không tải được | Mạng chậm / zoom quá xa | Zoom gần hơn, thử trình duyệt khác |
| Không upload được changeset | Chưa đăng nhập / conflict | Đăng nhập lại; tải lại dữ liệu vùng đó |

---

## 15. Best Practices & Cheat Sheet

### 15.1. Best practices

1. **Luôn ghi attribution** khi dùng dữ liệu/tiles OSM
2. **Cache** kết quả Nominatim/Overpass, đừng query lặp lại
3. **Tôn trọng giới hạn** 1 req/s của Nominatim và Overpass
4. **Đóng góp lại** khi sửa dữ liệu — đó là cách OSM tồn tại
5. **Không copy từ Google** — chỉ dùng nguồn mở hoặc tự khảo sát
6. **App production** → tự host tile/Nominatim hoặc dùng dịch vụ trả phí

### 15.2. Cheat sheet

```
# Website OSM                    https://www.openstreetmap.org
# Sửa bản đồ (web)               bấm nút Sửa (phím E)
# Overpass Turbo                 https://overpass-turbo.eu
# Tra cứu tag                    https://taginfo.openstreetmap.org
# Wiki tags                      https://wiki.openstreetmap.org/wiki/Map_features
# Tải dữ liệu VN (.pbf)          https://download.geofabrik.de/asia/vietnam.html

# Overpass: quán cafe quanh 1 điểm (bán kính 500m)
node["amenity"="cafe"](around:500, LAT, LON); out;

# Nominatim geocoding
https://nominatim.openstreetmap.org/search?q=Hồ+Gươm&format=json

# Nominatim reverse
https://nominatim.openstreetmap.org/reverse?lat=21.0285&lon=105.8542&format=json

# Leaflet cơ bản
L.map('map').setView([lat, lon], zoom);
L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
  attribution: '© OpenStreetMap contributors'
}).addTo(map);
```

### 15.3. Tài liệu tham khảo

- Trang chủ: https://www.openstreetmap.org
- Wiki: https://wiki.openstreetmap.org
- Cộng đồng VN: tìm "OpenStreetMap Vietnam" trên Facebook/Telegram
- LearnOSM (học từng bước): https://learnosm.org
- Overpass docs: https://wiki.openstreetmap.org/wiki/Overpass_API
- Leaflet docs: https://leafletjs.com

---

## 16. Chỉ Đường Với OSRM (Miễn Phí)

OSRM (Open Source Routing Machine) là engine chỉ đường chạy trên dữ liệu OSM — có demo public miễn phí.

### 16.1. Gọi API chỉ đường

```python
import requests

def chi_duong(lat1, lon1, lat2, lon2):
    url = (f"https://router.project-osrm.org/route/v1/driving/"
           f"{lon1},{lat1};{lon2},{lat2}")
    r = requests.get(url, params={
        "overview": "full", "geometries": "geojson", "steps": "true"
    }, timeout=15).json()
    route = r["routes"][0]
    km = route["distance"] / 1000
    phut = route["duration"] / 60
    return km, phut, route["geometry"]["coordinates"]

# Hồ Gươm -> Sân bay Nội Bài
km, phut, _ = chi_duong(21.0285, 105.8542, 21.2144, 105.8042)
print(f"Quãng đường: {km:.1f} km, thời gian: {phut:.0f} phút")
# Quãng đường: 27.3 km, thời gian: 35 phút
```

### 16.2. Các profile di chuyển

| Profile | URL | Dùng cho |
|---------|-----|----------|
| `driving` | `/route/v1/driving/` | Ô tô |
| `walking` | `/route/v1/walking/` | Đi bộ |
| `cycling` | `/route/v1/cycling/` | Xe đạp |

### 16.3. Vẽ tuyến đường lên Leaflet

```html
<script>
// Lấy tuyến đường từ OSRM rồi vẽ
async function veTuyenDuong() {
  const url = 'https://router.project-osrm.org/route/v1/driving/' +
    '105.8542,21.0285;105.8042,21.2144' +
    '?overview=full&geometries=geojson';
  const data = await (await fetch(url)).json();
  const coords = data.routes[0].geometry.coordinates
    .map(([lon, lat]) => [lat, lon]);  // OSRM trả [lon,lat]

  L.polyline(coords, { color: 'blue', weight: 5 }).addTo(map);
  const km = (data.routes[0].distance / 1000).toFixed(1);
  alert(`Quãng đường: ${km} km`);
}
veTuyenDuong();
</script>
```

### 16.4. Giới hạn demo public

- Demo `router.project-osrm.org` chỉ cho dùng nhẹ, thử nghiệm
- App thật → tự host OSRM bằng Docker:

```bash
# Tải dữ liệu VN
wget https://download.geofabrik.de/asia/vietnam-latest.osm.pbf

# Xử lý dữ liệu (mất vài phút)
docker run -t -v "${PWD}:/data" osrm/osrm-backend \
  osrm-extract -p /opt/car.lua /data/vietnam-latest.osm.pbf
docker run -t -v "${PWD}:/data" osrm/osrm-backend \
  osrm-partition /data/vietnam-latest.osrm
docker run -t -v "${PWD}:/data" osrm/osrm-backend \
  osrm-customize /data/vietnam-latest.osrm

# Chạy server
docker run -t -i -p 5000:5000 -v "${PWD}:/data" osrm/osrm-backend \
  osrm-routed --algorithm mld /data/vietnam-latest.osrm
```

Truy cập `http://localhost:5000` — có ngay server chỉ đường riêng!

---

## 17. Humanitarian Mapping — Đóng Góp Ý Nghĩa

### 17.1. HOT Tasking Manager

Khi có thiên tai (bão lũ ở miền Trung chẳng hạn), tổ chức **Humanitarian OpenStreetMap Team** tạo "task" để cộng đồng cùng vẽ bản đồ vùng bị ảnh hưởng — giúp đội cứu trợ biết đường đi, nhà dân.

1. Vào https://tasks.hotosm.org
2. Đăng nhập bằng tài khoản OSM
3. Chọn 1 project (ví dụ: vẽ nhà ở Quảng Bình sau bão)
4. Chọn 1 ô vuông → bấm **Start mapping**
5. Vẽ nhà/đường theo ảnh vệ tinh → **Submit**

### 17.2. Mapillary — đóng góp ảnh đường phố

Tải app **Mapillary** (của Meta), vừa đi vừa chụp ảnh đường phố — ảnh giúp người khác vẽ bản đồ chính xác hơn. Dữ liệu mở, tích hợp vào iD/JOSM.

---

## 18. Dữ Liệu Thô & Phân Tích GIS

### 18.1. Tải dữ liệu cả nước

```bash
# File .osm.pbf Việt Nam (~300MB)
wget https://download.geofabrik.de/asia/vietnam-latest.osm.pbf

# Xem thông tin file
osmium fileinfo vietnam-latest.osm.pbf
```

### 18.2. Lọc dữ liệu bằng Osmium

```bash
# Cài osmium
sudo apt install osmium-tool

# Lọc tất cả bệnh viện
osmium tags-filter vietnam-latest.osm.pbf \
  nwr/amenity=hospital -o benh_vien.osm.pbf

# Lọc đường giao thông ở Hà Nội (cần file đã cắt vùng trước)
osmium extract -b 105.7,20.9,106.0,21.1 \
  vietnam-latest.osm.pbf -o hanoi.osm.pbf
```

### 18.3. Chuyển sang GeoJSON để phân tích

```bash
# Cài osmtogeojson
npm install -g osmtogeojson

# Overpass query -> GeoJSON
curl "https://overpass-api.de/api/interpreter?data=[out:json];node['amenity'='hospital'](20.9,105.7,21.1,106.0);out;" \
  | osmtogeojson > benh_vien_hanoi.geojson
```

```python
# Phân tích bằng Python
import geopandas as gpd

gdf = gpd.read_file("benh_vien_hanoi.geojson")
print(f"Số bệnh viện: {len(gdf)}")
print(gdf[["name"]].head())

# Vẽ bản đồ
gdf.plot(figsize=(10, 10), color="red", markersize=20)
```

---

## 19. OSM Trên Mobile

| App | Nền tảng | Chức năng |
|-----|----------|-----------|
| **OsmAnd** | Android/iOS | Bản đồ offline, chỉ đường, ghi track GPS |
| **Organic Maps** | Android/iOS | Nhẹ, offline, mã nguồn mở |
| **StreetComplete** | Android | Đóng góp OSM kiểu "trả lời câu hỏi" — rất vui! |
| **Every Door** | Android/iOS | Thêm/sửa POI nhanh khi đang đi ngoài đường |
| **Vespucci** | Android | Editor đầy đủ trên điện thoại |

### StreetComplete — cách đóng góp vui nhất

1. Cài app, đăng nhập OSM
2. App hiện các "nhiệm vụ" quanh bạn: "Quán này mở cửa đến mấy giờ?", "Đường này có vỉa hè không?"
3. Trả lời bằng cách bấm chọn → tự động cập nhật lên OSM
4. Vừa đi dạo vừa đóng góp — không cần biết gì về tag!

---

## 20. So Sánh Thư Viện Bản Đồ Web

| Thư viện | Ngôn ngữ | Độ khó | Khi nào dùng |
|----------|----------|--------|--------------|
| **Leaflet** | JS | Dễ | Web đơn giản, nhẹ (~40KB) |
| **OpenLayers** | JS | Trung bình | Cần GIS nâng cao |
| **MapLibre GL** | JS | Trung bình | Bản đồ vector đẹp, 3D |
| **Folium** | Python | Dễ | Vẽ bản đồ từ Jupyter/notebook |

### Ví dụ Folium (Python)

```python
# pip install folium
import folium

m = folium.Map(location=[21.0285, 105.8542], zoom_start=15)

folium.Marker(
    [21.0285, 105.8542],
    popup="<b>Hồ Gươm</b>",
    tooltip="Click để xem",
).add_to(m)

# Vẽ vòng tròn bán kính 500m
folium.Circle(
    [21.0285, 105.8542], radius=500,
    color="blue", fill=True, fill_opacity=0.2,
    popup="Bán kính 500m",
).add_to(m)

m.save("ban_do.html")
print("Đã lưu ban_do.html — mở bằng trình duyệt!")
```

---

## 21. Câu Hỏi Thường Gặp (FAQ)

**Dữ liệu OSM có chính xác không?**
Ở thành phố lớn VN khá chính xác nhờ cộng đồng đóng góp. Vùng sâu vùng xa có thể thiếu — bạn có thể tự bổ sung!

**Dùng OSM cho app thương mại được không?**
Được 100%. Chỉ cần ghi attribution `© OpenStreetMap contributors`. Không mất phí.

**Tại sao bản đồ của tôi khác với openstreetmap.org?**
Các app/website cập nhật dữ liệu theo lịch riêng (hàng ngày/tuần/tháng). openstreetmap.org cập nhật gần như realtime sau khi bạn sửa.

**Tôi sửa sai thì sao?**
Mọi chỉnh sửa đều có lịch sử — ai cũng có thể revert (hoàn tác). Đừng sợ sai, cộng đồng sẽ giúp sửa.

**Overpass query của tôi bị lỗi 429?**
Bạn gửi quá nhiều request. Giãn cách ra, hoặc dùng mirror khác: `overpass.kumi.systems`, `overpass.nchc.org.tw`.

**Muốn xóa tài khoản OSM?**
Vào cài đặt tài khoản → xóa. Lưu ý: các chỉnh sửa đã đóng góp vẫn ở lại (thuộc về cộng đồng theo ODbL).

---

*Cập nhật: 10/10/2026*

---

## 22. Hướng Dẫn Chi Tiết: Thêm Địa Chỉ Nhà Lên Bản Đồ

Tình huống thực tế: nhà bạn ở hẻm nhỏ, shipper hay lạc. Hãy thêm địa chỉ lên OSM!

### Bước 1: Chuẩn bị
1. Đăng nhập openstreetmap.org
2. Zoom đến khu nhà bạn (zoom 18-19)
3. Bật lớp ảnh vệ tinh: **Lớp nền** → **Bing aerial** (để thấy rõ nhà)

### Bước 2: Vẽ tòa nhà
1. Bấm **Sửa** → chọn công cụ **Vùng** (phím `3`)
2. Click 4 góc nhà bạn, khép kín
3. Bấm `S` để vuông góc
4. Panel trái → **Tòa nhà** → điền:
   - `Tên`: (để trống nếu là nhà riêng)
   - `Số tầng`: ví dụ `4`
   - `Địa chỉ`: số nhà + tên đường

### Bước 3: Thêm điểm địa chỉ
1. Chọn công cụ **Điểm** (phím `1`)
2. Click ngay cổng nhà
3. Tìm `địa chỉ` → điền đầy đủ:
   ```
   addr:housenumber=45
   addr:street=Hẻm 123 Nguyễn Trãi
   addr:city=TP. Hồ Chí Minh
   ```
4. **Lưu** → ghi chú: `Thêm địa chỉ nhà số 45 hẻm 123 Nguyễn Trãi` → **Tải lên**

Sau vài phút, địa chỉ của bạn sẽ tìm được trên Nominatim và mọi app dùng OSM!

---

## 23. Overpass QL Nâng Cao

### Toán tử hay dùng

| Cú pháp | Ý nghĩa | Ví dụ |
|---------|---------|-------|
| `nwr[...]` | node + way + relation | `nwr["amenity"="cafe"]` |
| `around:R,lat,lon` | Trong bán kính R mét | `(around:1000, 21.0, 105.8)` |
| `["key"~"regex"]` | Tag khớp regex | `["name"~"Phở"]` |
| `out center` | Trả tọa độ trung tâm (cho way) | — |
| `out count` | Chỉ đếm số lượng | — |

### Ví dụ 5: Tìm quán ăn mở cửa sau 22h ở Hà Nội

```overpass
[out:json][timeout:30];
area["name"="Hà Nội"]->.hn;
nwr["amenity"~"restaurant|cafe|fast_food"]["opening_hours"](area.hn);
out center;
```

> Lưu ý: query này trả về tất cả rồi bạn lọc `opening_hours` bằng code — Overpass không so sánh giờ trực tiếp được.

```python
# Lọc bằng Python
import re

def mo_sau_22h(opening_hours):
    # Đơn giản: tìm giờ đóng cửa > 22
    m = re.findall(r'(\d{2}):\d{2}-(\d{2}):\d{2}', opening_hours or '')
    return any(int(dong) >= 22 or int(dong) < 6 for _, dong in m)

# Giả sử elements là kết quả Overpass
dem = [e for e in elements
       if mo_sau_22h(e.get("tags", {}).get("opening_hours"))]
print(f"Có {len(dem)} quán mở sau 22h")
```

### Ví dụ 6: Xuất CSV danh sách trường học

Dùng overpass-turbo → chạy query → bấm **Xuất** → **CSV**:

```overpass
[out:csv(name, "addr:street")][timeout:25];
area["name"="Đà Nẵng"]->.dn;
nwr["amenity"="school"](area.dn);
out;
```

---

## 24. Tích Hợp OSM Vào App Mobile

### React Native (dùng react-native-maps + OSM tiles)

```bash
npx expo install react-native-maps
```

```jsx
import MapView, { UrlTile, Marker } from 'react-native-maps';

export default function BanDo() {
  return (
    <MapView
      style={{ flex: 1 }}
      initialRegion={{
        latitude: 21.0285, longitude: 105.8542,
        latitudeDelta: 0.02, longitudeDelta: 0.02,
      }}
    >
      {/* Dùng tiles OSM thay vì Google */}
      <UrlTile
        urlTemplate="https://tile.openstreetmap.org/{z}/{x}/{y}.png"
        maximumZ={19}
      />
      <Marker
        coordinate={{ latitude: 21.0285, longitude: 105.8542 }}
        title="Hồ Gươm"
      />
    </MapView>
  );
}
```

### Flutter (dùng flutter_map)

```yaml
# pubspec.yaml
dependencies:
  flutter_map: ^6.0.0
  latlong2: ^0.9.0
```

```dart
import 'package:flutter_map/flutter_map.dart';
import 'package:latlong2/latlong.dart';

FlutterMap(
  options: MapOptions(
    initialCenter: LatLng(21.0285, 105.8542),
    initialZoom: 15,
  ),
  children: [
    TileLayer(
      urlTemplate: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
    ),
    MarkerLayer(markers: [
      Marker(
        point: LatLng(21.0285, 105.8542),
        child: Icon(Icons.location_pin, color: Colors.red, size: 40),
      ),
    ]),
  ],
)
```

---

*Cập nhật: 10/10/2026 — Tổng 24 phần*

---

## 25. Dự Án Thực Tế: Web Tìm Quán Cà Phê Gần Nhất

Kết hợp mọi thứ đã học: Overpass + Nominatim + Leaflet.

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tìm quán cà phê gần bạn</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>
    body { margin: 0; font-family: sans-serif; }
    #map { height: 70vh; }
    #panel { padding: 16px; }
    button {
      padding: 12px 24px; font-size: 16px; cursor: pointer;
      background: #6f4e37; color: white; border: none; border-radius: 8px;
    }
    #ketqua { margin-top: 12px; }
    .quan { padding: 8px; border-bottom: 1px solid #eee; }
  </style>
</head>
<body>
  <div id="map"></div>
  <div id="panel">
    <button onclick="timQuan()">☕ Tìm quán cà phê gần tôi</button>
    <div id="ketqua"></div>
  </div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    const map = L.map('map').setView([21.0285, 105.8542], 15);
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
    }).addTo(map);
    let layerQuan = L.layerGroup().addTo(map);

    async function timQuan() {
      document.getElementById('ketqua').innerHTML = '⏳ Đang tìm...';

      // 1. Lấy vị trí người dùng
      const pos = await new Promise((res, rej) =>
        navigator.geolocation.getCurrentPosition(res, rej));
      const lat = pos.coords.latitude, lon = pos.coords.longitude;
      map.setView([lat, lon], 16);
      L.marker([lat, lon]).addTo(map).bindPopup('📍 Bạn ở đây');

      // 2. Query Overpass: quán cafe trong 1km
      const q = `[out:json][timeout:25];
        (node["amenity"="cafe"](around:1000,${lat},${lon});
         way["amenity"="cafe"](around:1000,${lat},${lon}););
        out center 20;`;
      const data = await (
        await fetch('https://overpass-api.de/api/interpreter?data=' +
          encodeURIComponent(q))).json();

      // 3. Hiển thị
      layerQuan.clearLayers();
      let html = `<b>Tìm thấy ${data.elements.length} quán:</b>`;
      data.elements.slice(0, 20).forEach(el => {
        const t = el.tags || {};
        const name = t.name || 'Quán không tên';
        const la = el.lat || el.center.lat, lo = el.lon || el.center.lon;
        L.marker([la, lo]).addTo(layerQuan)
          .bindPopup(`<b>☕ ${name}</b>`);
        html += `<div class="quan">☕ ${name}</div>`;
      });
      document.getElementById('ketqua').innerHTML = html;
    }
  </script>
</body>
</html>
```

**Cách dùng:**
1. Lưu thành `tim-cafe.html`
2. Mở bằng trình duyệt (cho phép truy cập vị trí)
3. Bấm nút → bản đồ hiện các quán cà phê quanh bạn, dữ liệu 100% từ OSM!

> Lưu ý: file này dùng `geolocation` nên cần mở qua `http://` (dùng `npx serve` hoặc Live Server của VS Code), không mở trực tiếp `file://`.

---

## 26. Bảng So Sánh Nhanh Các Dịch Vụ OSM

| Dịch vụ | Chức năng | URL | Giới hạn free |
|---------|-----------|-----|---------------|
| openstreetmap.org | Xem & sửa bản đồ | openstreetmap.org | Không giới hạn |
| Overpass API | Truy vấn dữ liệu | overpass-api.de | Dùng vừa phải |
| Overpass Turbo | Query bằng giao diện | overpass-turbo.eu | Như trên |
| Nominatim | Geocoding | nominatim.openstreetmap.org | 1 req/s |
| OSRM demo | Chỉ đường | router.project-osrm.org | Demo nhẹ |
| Geofabrik | Tải dữ liệu .pbf | download.geofabrik.de | Miễn phí |
| uMap | Tạo bản đồ riêng | umap.openstreetmap.fr | Miễn phí |

---

*Cập nhật: 10/10/2026 — Tổng 26 phần*
