# 🗺️ Google Maps Platform API — Phần 2

> **Nội dung:** Places API, Directions API, Distance Matrix, Pricing, Billing, Code tổng hợp, Xử lý lỗi
> 📄 *Phần 1: [Setup & Maps JavaScript API](huong-dan-google-maps-api.md)*

---

## 8. Places API (New)

Bộ API tìm kiếm địa điểm: quán cà phê gần đây, chi tiết 1 quán (giờ mở cửa, review, ảnh), gợi ý khi user gõ.

> ⚠️ Bản **New** khác bản Legacy ở 2 điểm lớn:
> 1. Endpoint mới: `https://places.googleapis.com/v1/...` (không phải `maps.googleapis.com/maps/api/place/...`).
> 2. **Key truyền qua header** `X-Goog-Api-Key`, và phải gửi header **`X-Goog-FieldMask`** liệt kê field muốn lấy (không gửi = lỗi!). FieldMask giúp cậu chỉ trả tiền cho dữ liệu mình cần.

### 8.1. Text Search — tìm địa điểm bằng câu chữ

```bash
curl -X POST "https://places.googleapis.com/v1/places:searchText" \
  -H "Content-Type: application/json" \
  -H "X-Goog-Api-Key: YOUR_API_KEY" \
  -H "X-Goog-FieldMask: places.displayName,places.formattedAddress,places.location,places.rating,places.userRatingCount" \
  -d '{
    "textQuery": "cà phê view đẹp ở Đà Lạt",
    "languageCode": "vi",
    "maxResultCount": 5
  }'
```

**Response** (rút gọn 1 kết quả):

```json
{
  "places": [
    {
      "displayName": { "text": "Tiệm Cà Phê Túi Mơ To", "languageCode": "vi" },
      "formattedAddress": "Hẻm 31 Sào Nam, Phường 11, Đà Lạt, Lâm Đồng, Việt Nam",
      "location": { "latitude": 11.9404, "longitude": 108.4278 },
      "rating": 4.4,
      "userRatingCount": 3120
    }
  ]
}
```

| Field | Nghĩa |
|-------|-------|
| `displayName.text` | Tên địa điểm |
| `formattedAddress` | Địa chỉ đầy đủ |
| `location` | Tọa độ (`latitude`, `longitude` — chú ý khác kiểu `{lat,lng}` của Maps JS) |
| `rating` / `userRatingCount` | Điểm đánh giá trung bình / số lượt đánh giá |

### 8.2. Nearby Search — tìm quanh 1 tọa độ

Tìm "quán phở trong bán kính 1km quanh Hồ Gươm":

```bash
curl -X POST "https://places.googleapis.com/v1/places:searchNearby" \
  -H "Content-Type: application/json" \
  -H "X-Goog-Api-Key: YOUR_API_KEY" \
  -H "X-Goog-FieldMask: places.displayName,places.location,places.rating,places.types" \
  -d '{
    "includedTypes": ["restaurant"],
    "maxResultCount": 10,
    "locationRestriction": {
      "circle": {
        "center": { "latitude": 21.0285, "longitude": 105.8542 },
        "radius": 1000.0
      }
    },
    "languageCode": "vi"
  }'
```

- `includedTypes`: lọc loại địa điểm (`restaurant`, `cafe`, `gas_station`, `atm`, `hospital`, `park`...). Danh sách đầy đủ xem trong tài liệu "Place Types".
- `radius`: mét, tối đa 50.000 (50km).
- Có thể loại trừ bằng `excludedTypes`.

### 8.3. Place Details — chi tiết 1 địa điểm

Cần `placeId` (lấy từ Text/Nearby Search, hoặc từ Geocoding):

```bash
curl "https://places.googleapis.com/v1/places/ChIJd9p6Q5YpdTERxxx \
  -H 'X-Goog-Api-Key: YOUR_API_KEY' \
  -H 'X-Goog-FieldMask: displayName,formattedAddress,internationalPhoneNumber,websiteUri,rating,regularOpeningHours,reviews'"
```

Response chứa `regularOpeningHours` (giờ mở cửa từng ngày), `reviews` (mảng review: tác giả, rating, text, thời gian), `websiteUri`, `internationalPhoneNumber`...

### 8.4. Place Photo — lấy ảnh địa điểm

Place Details trả về `photos[]`, mỗi photo có `name` dạng `places/XXX/photos/YYY`. Lấy ảnh thật:

```bash
curl "https://places.googleapis.com/v1/places/XXX/photos/YYY/media?maxHeightPx=400&key=YOUR_API_KEY" -o anh.jpg
```

Dùng trực tiếp trong thẻ `<img>`: `<img src="https://places.googleapis.com/v1/places/XXX/photos/YYY/media?maxHeightPx=400&key=YOUR_API_KEY">`.

> ⚠️ Ảnh qua API vẫn tính tiền theo SKU Place Photos. Đừng load ảnh full-size khi chỉ cần thumbnail — chỉnh `maxHeightPx`/`maxWidthPx` nhỏ lại.

### 8.5. Autocomplete — gợi ý khi user gõ

```bash
curl -X POST "https://places.googleapis.com/v1/places:autocomplete" \
  -H "Content-Type: application/json" \
  -H "X-Goog-Api-Key: YOUR_API_KEY" \
  -d '{
    "input": "phở hòa",
    "languageCode": "vi",
    "locationBias": {
      "circle": { "center": { "latitude": 10.7769, "longitude": 106.7009 }, "radius": 20000.0 }
    }
  }'
```

Response:

```json
{
  "suggestions": [
    {
      "placePrediction": {
        "placeId": "ChIJ...",
        "text": { "text": "Phở Hòa Pasteur, Quận 3, Hồ Chí Minh", "matches": [...] }
      }
    }
  ]
}
```

**Quy tắc vàng của Autocomplete:**
1. **Debounce** — chỉ gọi API khi user ngừng gõ ~300ms (xem mục 15). Mỗi ký tự 1 request = đốt tiền.
2. **Dùng session token** — 1 phiên gõ (từ lúc gõ đến lúc chọn) dùng chung 1 `sessionToken`, Google tính tiền theo session rẻ hơn nhiều so với tính từng request.
3. `locationBias` giúp gợi ý ưu tiên gần user, nhưng không bắt buộc — user ở Hà Nội vẫn tìm được quán ở Đà Lạt.

> 💡 Trên web, cách dễ nhất là dùng widget có sẵn của Maps JS: `google.maps.places.Autocomplete` gắn vào `<input>` — Google lo hết UI + session token cho cậu. Chỉ cần load thêm library: `&libraries=places` trong URL script.

---

## 9. Directions API

Chỉ đường từ A đến B, trả về lộ trình chi tiết từng bước rẽ.

Endpoint: `https://maps.googleapis.com/maps/api/directions/json`

### 9.1. Request thật

Đi từ **Sân bay Nội Bài** đến **Hồ Gươm** bằng ô tô:

```bash
curl "https://maps.googleapis.com/maps/api/directions/json?origin=Sân+bay+Nội+Bài&destination=Hồ+Gươm,+Hà+Nội&mode=driving&language=vi&region=vn&key=YOUR_API_KEY"
```

`origin`/`destination` nhận cả địa chỉ chữ lẫn tọa độ (`21.0285,105.8542`).

### 9.2. Đọc response — cấu trúc routes → legs → steps

```json
{
  "status": "OK",
  "routes": [
    {
      "summary": "Võ Nguyên Giáp - Cầu Nhật Tân",
      "legs": [
        {
          "distance": { "text": "27,5 km", "value": 27500 },
          "duration": { "text": "35 phút", "value": 2100 },
          "duration_in_traffic": { "text": "48 phút", "value": 2880 },
          "start_address": "Sân bay Quốc tế Nội Bài, ...",
          "end_address": "Hồ Hoàn Kiếm, ...",
          "steps": [
            {
              "distance": { "text": "0,3 km", "value": 300 },
              "duration": { "text": "1 phút", "value": 60 },
              "html_instructions": "Đi về hướng <b>Đông Nam</b>...",
              "travel_mode": "DRIVING"
            }
          ]
        }
      ],
      "overview_polyline": { "points": "encoded_string..." }
    }
  ]
}
```

| Field | Nghĩa |
|-------|-------|
| `routes[]` | Mảng lộ trình (nhiều route khi `alternatives=true`) |
| `routes[].legs[]` | Chặng — 1 leg = 1 đoạn giữa 2 waypoints liên tiếp. Đi thẳng A→B chỉ có 1 leg |
| `leg.distance` | `text` để hiển thị ("27,5 km"), `value` để tính toán (mét) |
| `leg.duration` | Thời gian không kẹt xe; `duration_in_traffic` = có tính kẹt xe (chỉ khi có `departure_time=now`) |
| `leg.steps[]` | Từng bước rẽ chi tiết — `html_instructions` chứa thẻ HTML, nhớ xử lý khi render |
| `overview_polyline.points` | Chuỗi polyline mã hóa — **vẽ lộ trình lên bản đồ** (giải mã bằng `google.maps.geometry.encoding.decodePath`, cần `&libraries=geometry`) |

### 9.3. Các mode di chuyển

| `mode=` | Nghĩa | Ghi chú |
|---------|-------|---------|
| `driving` | Ô tô (mặc định) | Có `duration_in_traffic` nếu kèm `departure_time=now` |
| `walking` | Đi bộ | |
| `bicycling` | Xe đạp | Không khả dụng ở mọi khu vực |
| `transit` | Phương tiện công cộng | Cần thêm `departure_time` hoặc `arrival_time` (timestamp). Ở VN dữ liệu transit còn hạn chế |

```bash
# Đi bộ từ Chợ Bến Thành đến Nhà thờ Đức Bà
curl "https://maps.googleapis.com/maps/api/directions/json?origin=Chợ+Bến+Thành&destination=Nhà+thờ+Đức+Bà,+TP.HCM&mode=walking&language=vi&key=YOUR_API_KEY"
```

### 9.4. Waypoints & alternatives

```bash
curl "https://maps.googleapis.com/maps/api/directions/json?origin=Hà+Nội&destination=TP.HCM&waypoints=Huế|Đà+Nẵng&alternatives=true&key=YOUR_API_KEY"
```

- `waypoints`: các điểm dừng giữa đường, cách nhau bằng `|`. Thêm `optimize:true|` ở đầu để Google tự sắp xếp thứ tự tối ưu (bài toán travelling salesman mini).
- `alternatives=true`: trả về nhiều lộ trình để user chọn.
- Giới hạn: tối đa 25 waypoints (có billing) — đủ cho tour xuyên Việt rồi 😄

### 9.5. Vẽ lộ trình lên bản đồ (kết hợp Maps JS)

```js
async function drawRoute(map, origin, destination) {
  const res = await fetch(
    `https://maps.googleapis.com/maps/api/directions/json?` +
    new URLSearchParams({ origin, destination, language: "vi", key: "YOUR_API_KEY" })
  );
  const data = await res.json();
  if (data.status !== "OK") throw new Error(data.status);

  // Giải mã polyline → vẽ đường màu xanh lên bản đồ
  const path = google.maps.geometry.encoding.decodePath(
    data.routes[0].overview_polyline.points
  );
  new google.maps.Polyline({ path, map, strokeColor: "#1a73e8", strokeWeight: 5 });

  const leg = data.routes[0].legs[0];
  return { distance: leg.distance.text, duration: leg.duration.text };
}
// ⚠️ Nhớ load thư viện geometry: .../api/js?key=...&libraries=geometry&callback=initMap
```

> 💡 Cách "lười" hơn: dùng `google.maps.DirectionsService` + `DirectionsRenderer` của Maps JS — 2 class này gọi và vẽ hộ cậu, không cần fetch tay:
>
> ```js
> const svc = new google.maps.DirectionsService();
> const renderer = new google.maps.DirectionsRenderer({ map });
> svc.route(
>   { origin: "Sân bay Nội Bài", destination: "Hồ Gươm, Hà Nội", travelMode: "DRIVING" },
>   (result, status) => { if (status === "OK") renderer.setDirections(result); }
> );
> ```

---

## 10. Distance Matrix API

Tính **khoảng cách + thời gian** giữa NHIỀU điểm xuất phát và NHIỀU điểm đến trong 1 request. Ví dụ: "từ kho hàng đến 5 địa chỉ giao hàng, mỗi cặp mất bao lâu?"

Endpoint: `https://maps.googleapis.com/maps/api/distancematrix/json`

### 10.1. Request thật

```bash
curl "https://maps.googleapis.com/maps/api/distancematrix/json?origins=Hồ+Gươm,+Hà+Nội|Chợ+Bến+Thành,+TP.HCM&destinations=Sân+bay+Nội+Bài|Phố+cổ+Hội+An&mode=driving&language=vi&key=YOUR_API_KEY"
```

- `origins`: các điểm đi, cách nhau bằng `|`.
- `destinations`: các điểm đến, cách nhau bằng `|`.
- Giới hạn: tối đa 25 origins × 25 destinations = 625 cặp/request.

### 10.2. Đọc response — rows × elements

```json
{
  "status": "OK",
  "origin_addresses": ["Hồ Hoàn Kiếm, Hà Nội, Việt Nam", "Chợ Bến Thành, ..."],
  "destination_addresses": ["Sân bay Quốc tế Nội Bài, ...", "Phố cổ Hội An, ..."],
  "rows": [
    {
      "elements": [
        { "status": "OK",
          "distance": { "text": "27,5 km", "value": 27500 },
          "duration": { "text": "35 phút", "value": 2100 } },
        { "status": "OK",
          "distance": { "text": "797 km", "value": 797000 },
          "duration": { "text": "14 giờ 20 phút", "value": 51600 } }
      ]
    },
    { "elements": [ /* origin thứ 2 × 2 destinations */ ] }
  ]
}
```

**Cách đọc ma trận:** `rows[i].elements[j]` = cặp (origin thứ i, destination thứ j).

| Field | Nghĩa |
|-------|-------|
| `origin_addresses` / `destination_addresses` | Địa chỉ chuẩn mà Google hiểu cho từng điểm (theo đúng thứ tự cậu gửi) |
| `rows[i].elements[j].status` | `OK` / `NOT_FOUND` (không tìm thấy điểm) / `ZERO_RESULTS` (không có đường đi) — **kiểm tra từng element**, đừng chỉ nhìn status tổng! |
| `distance.value` | Mét (số) — dùng để tính toán, sắp xếp |
| `duration.value` | Giây (số) — dùng để tính toán |

### 10.3. Ví dụ JS: tìm điểm đến gần nhất

```js
async function nearestDestination(origin, destinations) {
  const params = new URLSearchParams({
    origins: origin,
    destinations: destinations.join("|"),
    mode: "driving",
    language: "vi",
    key: "YOUR_API_KEY",
  });
  const data = await (
    await fetch(`https://maps.googleapis.com/maps/api/distancematrix/json?${params}`)
  ).json();

  if (data.status !== "OK") throw new Error(data.status);

  let best = null;
  data.rows[0].elements.forEach((el, j) => {
    if (el.status === "OK" && (!best || el.duration.value < best.seconds)) {
      best = { name: destinations[j], seconds: el.duration.value, text: el.duration.text };
    }
  });
  return best; // { name: "...", seconds: 2100, text: "35 phút" }
}

nearestDestination("Hồ Gươm, Hà Nội", ["Sân bay Nội Bài", "Phố cổ Hội An"])
  .then(console.log);
```

> 💡 Distance Matrix tính tiền **theo số element** (cặp origin×destination), không phải theo request. 1 request 10×10 = 100 elements = trả tiền 100 lần. Gom request khôn ngoan (mục 15) để đỡ tốn.

---

## 11. Pricing & Free Credit

> ⚠️ Giá dưới đây là mức phổ biến tại thời điểm viết bài (10/2026). Google có thể điều chỉnh — luôn kiểm tra trang chính thức: `developers.google.com/maps/billing-and-pricing/pricing`.

### 11.1. $200 free credit mỗi tháng nghĩa là gì?

Mỗi tháng, Google **tặng $200** vào tài khoản billing của cậu. Tiền thật trong thẻ chỉ bị trừ khi chi phí **vượt quá $200** trong tháng đó. Credit **không cộng dồn** — hết tháng là reset.

Quy đổi ra số request miễn phí mỗi tháng (làm tròn):

| API / SKU | Giá (USD) | $200 mua được |
|-----------|-----------|---------------|
| Dynamic Maps (Maps JS load) | $7 / 1.000 loads | **~28.500 lượt tải bản đồ** |
| Static Maps (ảnh bản đồ) | $2 / 1.000 | ~100.000 ảnh |
| Geocoding | $5 / 1.000 | **~40.000 request** |
| Directions | $5 / 1.000 | ~40.000 request |
| Distance Matrix (basic) | $5 / 1.000 elements | ~40.000 cặp điểm |
| Places Text Search | $32 / 1.000 | **~6.250 lượt tìm** |
| Places Nearby Search | $32 / 1.000 | ~6.250 lượt tìm |
| Place Details | $17 / 1.000 | ~11.700 lượt xem chi tiết |
| Place Photos | $7 / 1.000 | ~28.500 ảnh |
| Autocomplete (per session) | $2.83 / 1.000 sessions | ~70.600 phiên gõ |

> 💡 Với app cá nhân / demo / vài nghìn user: **$200/tháng thường là đủ**, thực tế không mất đồng nào. Chỉ khi app lên hàng chục nghìn user hoạt động mạnh mới cần lo.

### 11.2. Ví dụ tính chi phí thực tế

**Kịch bản 1 — Web du lịch nhỏ:**
- 10.000 lượt tải bản đồ/tháng: 10.000 × $7/1.000 = **$70**
- 5.000 Text Search: 5.000 × $32/1.000 = **$160**
- 2.000 Directions: 2.000 × $5/1.000 = **$10**
- **Tổng: $240** → trừ $200 free = **trả $40/tháng** (~1 triệu VNĐ).

**Kịch bản 2 — App giao hàng (gọi từ server, cache tốt):**
- 30.000 Geocoding: 30.000 × $5/1.000 = **$150**
- 8.000 Distance Matrix elements: 8.000 × $5/1.000 = **$40**
- **Tổng: $190** → trong $200 free = **trả $0** 🎉

**Bài học:** Text Search là SKU **đắt nhất** trong nhóm phổ biến ($32/1.000). Nếu app tìm kiếm nhiều, hãy cache kết quả (mục 15) — tìm "phở Hà Nội" 1 lần rồi lưu lại, đừng gọi lại mỗi lần user mở app.

### 11.3. Xem mình đã xài bao nhiêu

- **APIs & Services → Dashboard** → click từng API → tab **Metrics**: biểu đồ request theo ngày.
- **Billing → Reports**: xem chi phí quy ra tiền, lọc theo API/SKU/project.
- Đặt **budget alert** (mục 12) để nhận email khi chạm ngưỡng 50%, 90%, 100%.

---

## 12. Billing Setup

> 🔴 **Bắt buộc:** Google Maps Platform **không chạy** nếu project chưa bật billing — kể cả trong hạn mức $200 free. Đây là lý do số 1 khiến người mới báo "key không chạy".

### 12.1. Bật billing — từng bước

1. Menu ☰ → **"Billing"**.
2. Click **"Link a billing account"** (hoặc "Manage billing accounts" → "Create account" nếu chưa có).
3. Chọn quốc gia **Việt Nam**, điền thông tin thuế (cá nhân thì điền tên + địa chỉ, mã số thuế có thể bỏ qua).
4. **Thêm thẻ thanh toán:** Google chấp nhận **thẻ tín dụng/ghi nợ quốc tế** (Visa/Mastercard). Thẻ ATM nội địa (NAPAS-only) **không dùng được**.
5. Xác minh: Google trừ thử ~$1 (hoặc 25.000đ) rồi hoàn lại — để kiểm tra thẻ thật.
6. Link billing account vào project → xong. Quay lại gọi API thử, lỗi `REQUEST_DENIED` vì billing sẽ biến mất.

> 💡 Kinh nghiệm từ thực tế: nhiều bạn ở VN không có thẻ quốc tế nên kẹt ở bước này — đó đúng là rào cản lớn nhất của Google Maps Platform. Khi nào có thẻ thì quay lại làm tiếp, các bước khác trong bài vẫn đọc hiểu được.

### 12.2. Đặt budget alert — chuông báo cháy túi

1. **Billing → Budgets & alerts** → **"Create budget"**.
2. Đặt tên, chọn project, đặt **số tiền** (ví dụ $10, $50).
3. Ở phần **Alert thresholds**, thêm các ngưỡng: **50%, 90%, 100%** (và 120% nếu muốn).
4. Nhập email nhận cảnh báo → **Save**.

Từ giờ, chi phí chạm ngưỡng là cậu nhận email ngay. **Budget không tự chặn chi tiêu** — nó chỉ báo. Muốn chặn cứng thì xem mục tiếp theo.

### 12.3. Đặt quota limit — trần cứng chống "đốt tiền"

Đây mới là cái **ngăn charge bất ngờ** thật sự:

1. **APIs & Services → Dashboard** → click 1 API (ví dụ Directions API).
2. Tab **"Quotas & System Limits"** (Hạn mức).
3. Tìm dòng quota theo ngày (ví dụ "Requests per day") → click biểu tượng ✏️ **Edit**.
4. Đặt con số trần, ví dụ **1.000 requests/ngày** → Save.

Vượt trần → API trả lỗi `OVER_QUERY_LIMIT`, **không tốn thêm tiền**. Với app mới, đặt quota ngày cho từng API là thói quen nên có từ ngày đầu.

**Gợi ý quota khởi đầu an toàn:**

| API | Quota/ngày gợi ý |
|-----|------------------|
| Maps JavaScript API | 1.000 loads |
| Geocoding API | 1.000 |
| Places API (New) | 500 |
| Directions API | 500 |
| Distance Matrix API | 500 |

Hết quota giữa chừng? Vào tăng lên — mất 30 giây, còn hơn mất tiền oan.

---

## 13. Code Ví Dụ Tổng Hợp

Một trang web mini hoàn chỉnh: **bản đồ + ô tìm kiếm địa điểm (autocomplete) + bấm để chỉ đường từ vị trí hiện tại**. HTML/JS thuần, 1 file duy nhất.

> ⚠️ Demo này gọi Directions/Geocoding **trực tiếp từ trình duyệt** cho đơn giản. App thật thì chuyển các call tốn tiền này về **server** (xem mục 5.4).

```html
<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Mini Maps — Tìm đường</title>
  <style>
    * { box-sizing: border-box; font-family: system-ui, sans-serif; }
    body { margin: 0; display: flex; flex-direction: column; height: 100vh; }
    #toolbar {
      display: flex; gap: 8px; padding: 12px;
      background: #fff; box-shadow: 0 2px 8px rgba(0,0,0,.15); z-index: 10;
    }
    #search { flex: 1; padding: 10px 14px; font-size: 16px;
      border: 1px solid #ccc; border-radius: 8px; }
    #goBtn { padding: 10px 20px; font-size: 16px; border: none;
      border-radius: 8px; background: #1a73e8; color: #fff; cursor: pointer; }
    #goBtn:disabled { background: #aaa; cursor: not-allowed; }
    #info { padding: 8px 14px; font-size: 14px; color: #333; background: #f8f9fa; }
    #map { flex: 1; }
  </style>
</head>
<body>
  <div id="toolbar">
    <input id="search" type="text" placeholder="Tìm địa điểm… (vd: phở Hòa Pasteur)" />
    <button id="goBtn" disabled>🧭 Chỉ đường</button>
  </div>
  <div id="info">📍 Đang lấy vị trí của bạn…</div>
  <div id="map"></div>

  <script>
    const API_KEY = "YOUR_API_KEY"; // ⚠️ demo — production để key ở server!
    let map, userPos = null, destPlace = null;
    let directionsRenderer, userMarker, destMarker;

    function initMap() {
      // Mặc định mở ở TP.HCM nếu chưa có vị trí
      map = new google.maps.Map(document.getElementById("map"), {
        center: { lat: 10.7769, lng: 106.7009 },
        zoom: 13,
      });
      directionsRenderer = new google.maps.DirectionsRenderer({ map });

      // 1. Lấy vị trí hiện tại của user
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          (pos) => {
            userPos = { lat: pos.coords.latitude, lng: pos.coords.longitude };
            map.setCenter(userPos);
            map.setZoom(14);
            userMarker = new google.maps.Marker({
              position: userPos, map, title: "Vị trí của bạn",
              icon: { path: google.maps.SymbolPath.CIRCLE, scale: 8,
                      fillColor: "#1a73e8", fillOpacity: 1, strokeColor: "#fff", strokeWeight: 2 },
            });
            document.getElementById("info").textContent = "📍 Đã lấy vị trí. Hãy tìm điểm đến!";
            document.getElementById("goBtn").disabled = false;
          },
          () => {
            document.getElementById("info").textContent =
              "⚠️ Không lấy được vị trí — vẫn tìm địa điểm được, nhưng nút Chỉ đường cần vị trí.";
          }
        );
      }

      // 2. Autocomplete gắn vào ô tìm kiếm (Google lo UI + session token)
      const autocomplete = new google.maps.places.Autocomplete(
        document.getElementById("search"),
        { componentRestrictions: { country: "vn" }, fields: ["place_id", "name", "geometry"] }
      );
      autocomplete.addListener("place_changed", () => {
        const place = autocomplete.getPlace();
        if (!place.geometry) {
          document.getElementById("info").textContent = "❌ Không tìm thấy địa điểm này.";
          return;
        }
        destPlace = place;
        if (destMarker) destMarker.setMap(null);
        destMarker = new google.maps.Marker({
          position: place.geometry.location, map, title: place.name,
        });
        map.panTo(place.geometry.location);
        map.setZoom(15);
        document.getElementById("info").textContent = `📌 Đã chọn: ${place.name} — bấm "Chỉ đường" nhé!`;
      });

      // 3. Nút chỉ đường
      document.getElementById("goBtn").addEventListener("click", () => {
        if (!userPos || !destPlace) {
          document.getElementById("info").textContent = "⚠️ Cần cả vị trí của bạn và điểm đến.";
          return;
        }
        new google.maps.DirectionsService().route(
          {
            origin: userPos,
            destination: { placeId: destPlace.place_id },
            travelMode: google.maps.TravelMode.DRIVING,
          },
          (result, status) => {
            if (status !== "OK") {
              document.getElementById("info").textContent = `❌ Không tìm được đường: ${status}`;
              return;
            }
            directionsRenderer.setDirections(result);
            const leg = result.routes[0].legs[0];
            document.getElementById("info").textContent =
              `🛣️ ${leg.distance.text} — khoảng ${leg.duration.text} (${leg.start_address} → ${leg.end_address})`;
          }
        );
      });
    }
  </script>

  <script async
    src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=places&callback=initMap&v=weekly">
  </script>
</body>
</html>
```

**App này dùng những gì đã học:**
- Mục 6: `google.maps.Map`, `Marker`, `DirectionsRenderer`.
- Mục 8: `places.Autocomplete` (gợi ý khi gõ, giới hạn trong VN).
- Mục 9: `DirectionsService.route()` chỉ đường, đọc `legs[0].distance/duration`.
- Mục 5: ⚠️ key đang lộ trong HTML — production phải tách key server-side!

Lưu file là `mini-maps.html`, thay key, mở trình duyệt → gõ "phở hòa" → chọn → bấm Chỉ đường. Chạy được ngay! 🎉

---

## 14. Xử Lý Lỗi Thường Gặp

### 14.1. Bảng tra lỗi nhanh

| Lỗi / `status` | Thấy ở đâu | Nguyên nhân | Cách sửa |
|----------------|-----------|-------------|----------|
| `REQUEST_DENIED` | Mọi API | (1) Key sai; (2) API chưa enable; (3) Billing chưa bật; (4) Restriction chặn (sai domain/IP) | Kiểm tra theo thứ tự: key đúng chưa → API đã enable chưa → billing đã bật chưa → restriction có chặn nhầm không |
| `OVER_QUERY_LIMIT` | Mọi API | Hết quota (ngày/giây) hoặc vượt trần đã đặt | Giảm tần suất gọi; kiểm tra Quotas; tăng quota nếu cần; bật billing |
| `INVALID_REQUEST` | Geocoding/Directions | Thiếu/thừa tham số bắt buộc | Geocoding cần `address` HOẶC `latlng`; Directions cần cả `origin` và `destination` |
| `ZERO_RESULTS` | Geocoding/Directions | Không tìm thấy / không có đường đi | Không phải lỗi hệ thống — báo user nhập lại, viết đầy đủ tên địa chỉ |
| `NOT_FOUND` | Distance Matrix (từng element) | 1 trong các điểm không xác định được | Kiểm tra lại điểm đó, thử geocode riêng từng điểm |
| `ApiNotActivatedMapError` | Maps JS (console) | Maps JavaScript API chưa enable | Vào Library bật API (mục 3) |
| `InvalidKeyMapError` | Maps JS (console) | Key sai hoặc đã bị xóa/vô hiệu hóa | Tạo key mới, kiểm tra lại chuỗi key |
| `RefererNotAllowedMapError` | Maps JS (console) | Domain hiện tại không nằm trong HTTP referrers | Thêm domain vào restriction của key (mục 5.1) |
| Bản đồ trắng trơn, không lỗi | Maps JS | `#map` không có height | CSS `#map { height: ... }` — lỗi quốc dân 😄 |
| Key lộ trên GitHub | — | Commit nhầm key thật | Xoay key ngay (mục 5.3); xóa commit chứa key (hoặc coi như key đã chết); kiểm tra Metrics xem có request lạ không |

### 14.2. Template xử lý lỗi trong code

Đừng bao giờ `fetch` mà không kiểm tra status — viết 1 hàm wrapper dùng cho mọi API:

```js
async function callGoogleApi(url) {
  const res = await fetch(url);
  if (!res.ok) throw new Error(`HTTP ${res.status} — kiểm tra mạng/key`);
  const data = await res.json();

  switch (data.status) {
    case "OK": return data;
    case "ZERO_RESULTS":
      throw new Error("Không tìm thấy kết quả — thử nhập địa chỉ đầy đủ hơn.");
    case "OVER_QUERY_LIMIT":
      throw new Error("Hết quota API trong ngày — thử lại mai hoặc tăng quota.");
    case "REQUEST_DENIED":
      throw new Error("Key bị từ chối — kiểm tra key, billing, API đã enable chưa.");
    case "INVALID_REQUEST":
      throw new Error("Thiếu tham số — kiểm tra lại query gửi đi.");
    default:
      throw new Error(`Lỗi không xác định: ${data.status}`);
  }
}
```

### 14.3. Quy trình debug khi "tự nhiên không chạy"

Làm theo đúng thứ tự này, 90% ca bệnh khỏi ở 3 bước đầu:

1. **Mở Console (F12)** — đọc lỗi chính xác, đừng đoán.
2. **API đã enable chưa?** → APIs & Services → Dashboard.
3. **Billing đã bật chưa?** → Billing — chưa bật là `REQUEST_DENIED`.
4. **Key đúng không?** — copy lại từ Credentials, coi chừng dính khoảng trắng.
5. **Restriction có chặn không?** — test bằng cách tạm nới lỏng, chạy được thì siết lại đúng.
6. **Quota còn không?** — Dashboard → Quotas & System Limits.
7. Vẫn không được → **Metrics** xem request có đến được Google không (phân biệt lỗi ở client hay ở Google).

---

## 15. Best Practices & Cheat Sheet

### 15.1. Best practices — xài API khôn ngoan, đỡ tốn tiền

**1. Cache kết quả — quy tắc số 1 tiết kiệm tiền**
Tọa độ của "Chợ Bến Thành" hôm nay và 1 năm sau vẫn vậy. Lưu kết quả Geocoding/Place Details vào database/cache (Redis, hoặc bảng SQL đơn giản) theo key là địa chỉ/place_id. Lần sau user hỏi lại → đọc cache, **không gọi Google**.

```js
// Pseudocode: cache-aside cho geocoding
async function geocodeCached(address) {
  const hit = await db.get(`geo:${address}`);
  if (hit) return hit;                       // trúng cache → $0
  const fresh = await geocodeFromGoogle(address);
  await db.set(`geo:${address}`, fresh, { ttl: "30d" });
  return fresh;
}
```
> 📜 Điều khoản Google cho phép cache **với điều kiện** hiển thị đúng attribution và refresh định kỳ — đọc kỹ "Maps Platform Terms" trước khi cache lâu dài.

**2. Debounce autocomplete (mục 8.5 nhắc rồi, nhắc lại vì quan trọng)**
```js
let timer;
searchInput.addEventListener("input", () => {
  clearTimeout(timer);
  timer = setTimeout(() => callAutocomplete(searchInput.value), 300);
});
```

**3. Lazy load bản đồ**
Đừng tải Maps JS ngay khi mở trang nếu user chưa cần thấy bản đồ. Dùng `IntersectionObserver` hoặc chỉ load khi user bấm nút "Xem bản đồ". Mỗi lần load thư viện = 1 Dynamic Maps load = tiền.

**4. Giới hạn quota từ ngày đầu** (mục 12.3) — trần cứng chống cháy túi.

**5. Gom request Distance Matrix** — thay vì 100 request 1×1, gom thành 1 request 10×10 (nhớ là tính tiền theo element, nhưng đỡ overhead và đỡ chạm rate limit/giây).

**6. FieldMask tối giản (Places API New)** — chỉ xin field nào dùng. Vừa rẻ hơn, vừa nhanh hơn.

**7. Tách key public/secret** (mục 5.4) — key frontend chỉ tick Maps JavaScript API.

**8. Theo dõi Metrics hàng tuần** — vào Dashboard xem API nào "ngốn" nhất, tối ưu chỗ đó trước.

### 15.2. 📋 Cheat sheet — tóm tắt 1 trang

**Endpoints:**
```
Maps JS:      https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&callback=initMap&v=weekly
Geocoding:    https://maps.googleapis.com/maps/api/geocode/json?address=...&key=YOUR_API_KEY
Reverse:      https://maps.googleapis.com/maps/api/geocode/json?latlng=LAT,LNG&key=YOUR_API_KEY
Text Search:  POST https://places.googleapis.com/v1/places:searchText      (header X-Goog-Api-Key + X-Goog-FieldMask)
Nearby:       POST https://places.googleapis.com/v1/places:searchNearby
Details:      GET  https://places.googleapis.com/v1/places/{placeId}
Photo:        GET  https://places.googleapis.com/v1/{photoName}/media?maxHeightPx=400&key=YOUR_API_KEY
Autocomplete: POST https://places.googleapis.com/v1/places:autocomplete
Directions:   https://maps.googleapis.com/maps/api/directions/json?origin=...&destination=...&key=YOUR_API_KEY
Dist Matrix:  https://maps.googleapis.com/maps/api/distancematrix/json?origins=...&destinations=...&key=YOUR_API_KEY
```

**Tham số hay dùng:** `language=vi` · `region=vn` · `mode=driving|walking|bicycling|transit` · `alternatives=true` · `waypoints=optimize:true|A|B`

**Tọa độ mẫu VN để test:**
| Địa điểm | lat | lng |
|----------|-----|-----|
| Hồ Gươm, Hà Nội | 21.0285 | 105.8542 |
| Chợ Bến Thành, TP.HCM | 10.7725 | 106.6983 |
| Cầu Rồng, Đà Nẵng | 16.0471 | 108.2062 |
| Phố cổ Hội An | 15.8801 | 108.3380 |
| Sân bay Nội Bài | 21.2148 | 105.8035 |

**$200 free ≈** 28.500 map loads · 40.000 geocode · 40.000 directions · 6.250 text search · 70.600 autocomplete sessions

**Checklist trước khi production:**
- [ ] Billing đã bật
- [ ] Đủ 5 API đã enable
- [ ] Key có HTTP referrer / IP restriction
- [ ] Key chỉ tick đúng API cần dùng
- [ ] Quota ngày đã đặt trần
- [ ] Budget alert 50/90/100% đã bật
- [ ] Không có key thật trong git (`.env` + `.gitignore`)
- [ ] Autocomplete có debounce + session token
- [ ] Kết quả ít thay đổi đã cache

**Lỗi → tra nhanh:** `REQUEST_DENIED` = key/billing/API/restriction · `OVER_QUERY_LIMIT` = hết quota · `ZERO_RESULTS` = không tìm thấy (nhập lại) · `INVALID_REQUEST` = thiếu tham số · bản đồ trắng = quên CSS height.

---

