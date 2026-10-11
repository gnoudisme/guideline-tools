# Hướng Dẫn Go Từ A-Z Cho Tiểu Dươngg 🐹

> Go (Golang) — ngôn ngữ do Google tạo ra, nhanh như C, viết dễ như Python!

---

## 1. Go Là Gì?

- Do **Google** tạo (2009), open source
- **Biên dịch** ra 1 file binary duy nhất → ném lên server là chạy
- Có **goroutine**: chạy hàng chục nghìn tác vụ song song cực nhẹ
- Cú pháp gọn, dễ đọc, khó viết sai

**Chạy thử không cần cài:** mở `go.dev/play` trên điện thoại là chạy được ngay!

---

## 2. Chương Trình Đầu Tiên

```go
package main          // mọi chương trình chạy được đều bắt đầu bằng package main

import "fmt"          // nhập thư viện in ra màn hình

func main() {         // hàm main = điểm bắt đầu
    fmt.Println("Chào cậu, tớ là Dươngg!")
}
```

- `package main`: khai báo đây là chương trình chính
- `import "fmt"`: dùng thư viện format (in ấn)
- `func main()`: hàm đầu tiên được gọi khi chạy

---

## 3. Biến (Variables)

```go
// Cách 1: khai báo rõ kiểu
var ten string = "Dươngg"
var tuoi int = 18

// Cách 2: tự suy kiểu (ngắn gọn, dùng nhiều nhất)
ten := "Dươngg"       // := vừa khai báo vừa gán, chỉ dùng trong hàm
tuoi := 18

// Cách 3: khai báo nhiều biến
var (
    a = 1
    b = "hai"
    c = true
)

// Hằng số (không đổi được)
const PI = 3.14
```

**Kiểu dữ liệu chính:**

| Kiểu | Ý nghĩa | Ví dụ |
|------|---------|-------|
| `int` | số nguyên | `42`, `-7` |
| `float64` | số thực | `3.14` |
| `string` | chuỗi | `"hello"` |
| `bool` | đúng/sai | `true`, `false` |
| `byte` | ký tự đơn | `'a'` |

---

## 4. Điều Kiện

```go
// if / else
diem := 8
if diem >= 9 {
    fmt.Println("Giỏi!")
} else if diem >= 5 {
    fmt.Println("Đạt")
} else {
    fmt.Println("Chưa đạt")
}

// switch (không cần break!)
thu := 3
switch thu {
case 2:
    fmt.Println("Thứ hai")
case 3:
    fmt.Println("Thứ ba")
default:
    fmt.Println("Ngày khác")
}
```

---

## 5. Vòng Lặp (chỉ có `for`!)

Go **không có** `while` — mọi vòng lặp đều dùng `for`:

```go
// Kiểu 1: như for truyền thống
for i := 0; i < 5; i++ {
    fmt.Println(i)
}

// Kiểu 2: như while
n := 0
for n < 3 {
    fmt.Println(n)
    n++
}

// Kiểu 3: lặp vô hạn (cẩn thận!)
for {
    // ...
    break // thoát
}

// Kiểu 4: duyệt mảng/slice
ds := []string{"Táo", "Cam", "Chuối"}
for i, v := range ds {
    fmt.Println(i, v)  // i = chỉ số, v = giá trị
}
```

---

## 6. Hàm (Functions)

```go
// Hàm cơ bản
func chao(ten string) string {
    return "Chào " + ten
}

// Nhiều tham số, nhiều giá trị trả về
func chia(a, b int) (int, int) {
    return a / b, a % b
}

// Dùng:
loi := chao("Dươngg")       // "Chào Dươngg"
thuong, du := chia(7, 3)    // thuong=2, du=1

// Bỏ qua giá trị không cần bằng _
_, du2 := chia(7, 3)        // chỉ lấy số dư
```

---

## 7. Mảng, Slice, Map

```go
// Mảng: kích thước CỐ ĐỊNH
var arr [3]int = [3]int{1, 2, 3}

// Slice: mảng ĐỘNG (dùng nhiều nhất!)
s := []int{1, 2, 3}
s = append(s, 4)            // thêm phần tử → [1 2 3 4]
fmt.Println(len(s))       // độ dài = 4

// Map: từ điển key-value
tuoi := map[string]int{
    "Dươngg": 18,
    "Capybara": 3,
}
tuoi["Mới"] = 20            // thêm
fmt.Println(tuoi["Dươngg"]) // 18
delete(tuoi, "Mới")         // xoá
```

---

## 8. Struct (giống object/class)

```go
// Định nghĩa
type Nguoi struct {
    Ten  string
    Tuoi int
}

// Dùng
d := Nguoi{Ten: "Dươngg", Tuoi: 18}
fmt.Println(d.Ten)   // Dươngg
d.Tuoi = 19          // đổi được

// Method: hàm gắn với struct
func (n Nguoi) GioiThieu() string {
    return "Tớ là " + n.Ten
}
fmt.Println(d.GioiThieu())
```

---

## 9. Con Trỏ (Pointer)

```go
x := 10
p := &x              // p = địa chỉ của x
fmt.Println(*p)      // *p = giá trị tại địa chỉ đó = 10
*p = 20              // đổi giá trị qua con trỏ
fmt.Println(x)       // 20!

// Dùng trong hàm để đổi thật (không copy)
func tang(n *int) {
    *n = *n + 1
}
y := 5
tang(&y)
fmt.Println(y)       // 6
```

---

## 10. Xử Lý Lỗi (không có try/catch!)

Go trả lỗi như **giá trị bình thường**:

```go
import "errors"

func chia(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("không chia được cho 0!")
    }
    return a / b, nil  // nil = không lỗi
}

kq, err := chia(10, 0)
if err != nil {
    fmt.Println("Lỗi:", err)
} else {
    fmt.Println(kq)
}
```

**Quy tắc vàng:** luôn kiểm tra `err != nil` ngay sau khi gọi hàm!

---

## 11. Goroutine — Linh Hồn Của Go ⚡

Thêm chữ `go` trước hàm là nó chạy **song song**:

```go
package main

import (
    "fmt"
    "time"
)

func dem(ten string) {
    for i := 1; i <= 3; i++ {
        fmt.Println(ten, i)
        time.Sleep(200 * time.Millisecond)
    }
}

func main() {
    go dem("A")   // chạy nền
    go dem("B")   // chạy nền
    dem("C")      // chạy chính
    time.Sleep(time.Second) // đợi goroutine xong
}
```

Kết quả: A, B, C in **xen kẽ** nhau — không chờ nhau!

**Tại sao nhẹ?** 1 goroutine chỉ tốn ~2KB RAM. Mở 100.000 goroutine vẫn mượt!

---

## 12. Channel — Goroutine Nói Chuyện Với Nhau

```go
package main

import "fmt"

func tinh(n int, ch chan int) {
    ch <- n * n   // gửi kết quả vào channel
}

func main() {
    ch := make(chan int)

    go tinh(5, ch)
    go tinh(7, ch)

    fmt.Println(<-ch)  // nhận: 25 hoặc 49
    fmt.Println(<-ch)  // nhận: cái còn lại
}
```

- `ch <- value`: **gửi** vào channel
- `<-ch`: **nhận** từ channel
- Gửi/nhận sẽ **đợi nhau** → tự đồng bộ, không cần lock!

---

## 13. WaitGroup — Đợi Goroutine Xong

```go
package main

import (
    "fmt"
    "sync"
)

func viec(id int, wg *sync.WaitGroup) {
    defer wg.Done()  // báo xong khi hàm kết thúc
    fmt.Println("Việc", id, "xong!")
}

func main() {
    var wg sync.WaitGroup
    for i := 1; i <= 5; i++ {
        wg.Add(1)           // +1 việc
        go viec(i, &wg)     // chạy song song
    }
    wg.Wait()               // đợi TẤT CẢ xong
    fmt.Println("Hoàn thành!")
}
```

---

## 14. HTTP Server — Backend Thật Chỉ 10 Dòng!

```go
package main

import (
    "fmt"
    "net/http"
)

func chao(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Chào cậu! Server Go đang chạy!")
}

func main() {
    http.HandleFunc("/", chao)
    fmt.Println("Server chạy ở :8080")
    http.ListenAndServe(":8080", nil)
}
```

Chạy → mở trình duyệt vào `localhost:8080` là thấy! Đây chính là cách tớ viết data plane cho MeshTrust đó!

---

## 15. Lộ Trình Học Tiếp

1. ✅ Cơ bản (bài này)
2. → Đọc code thật: mở `~/workspace/meshtrust/services/data-plane-go/` xem tớ viết
3. → Tự viết 1 API nhỏ: CRUD danh sách việc cần làm
4. → Học thêm: `go mod`, test (`go test`), context, mutex

**Tài liệu chính chủ:** `go.dev/tour` (tour tương tác, có tiếng Việt!)

---

*Viết bởi Capybara cho Tiểu Dươngg — 2026-10-11* 🦫
