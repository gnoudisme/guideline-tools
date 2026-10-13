# Hướng dẫn C++ chi tiết

> Tài liệu học C++ từ cơ bản đến nâng cao, giải thích chi tiết từng khái niệm kèm ví dụ chạy được thật.

## Mục lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Cài đặt môi trường](#2-cài-đặt-môi-trường)
3. [Chương trình đầu tiên](#3-chương-trình-đầu-tiên)
4. [Biến và kiểu dữ liệu](#4-biến-và-kiểu-dữ-liệu)
5. [Toán tử](#5-toán-tử)
6. [Nhập xuất dữ liệu](#6-nhập-xuất-dữ-liệu)
7. [Câu lệnh điều kiện](#7-câu-lệnh-điều-kiện)
8. [Vòng lặp](#8-vòng-lặp)
9. [Hàm](#9-hàm)
10. [Mảng](#10-mảng)
11. [Chuỗi ký tự](#11-chuỗi-ký-tự)
12. [Con trỏ](#12-con-trỏ)
13. [Tham chiếu](#13-tham-chiếu)
14. [Struct và Enum](#14-struct-và-enum)
15. [Lập trình hướng đối tượng](#15-lập-trình-hướng-đối-tượng)
16. [Kế thừa và đa hình](#16-kế-thừa-và-đa-hình)
17. [Template](#17-template)
18. [STL - Thư viện chuẩn](#18-stl---thư-viện-chuẩn)
19. [Xử lý ngoại lệ](#19-xử-lý-ngoại-lệ)
20. [Quản lý bộ nhớ](#20-quản-lý-bộ-nhớ)
21. [File I/O](#21-file-io)
22. [C++ hiện đại (11/14/17/20)](#22-c-hiện-đại-11141720)
23. [Best practices](#23-best-practices)

---

## 1. Giới thiệu

**C++** là ngôn ngữ lập trình biên dịch (compiled), được Bjarne Stroustrup phát triển từ năm 1979 như một bản mở rộng của C ("C with Classes"). C++ cho phép lập trình ở mức thấp (gần phần cứng như C) đồng thời hỗ trợ lập trình hướng đối tượng và lập trình generic.

**Đặc điểm nổi bật:**
- **Hiệu năng cao**: biên dịch ra mã máy trực tiếp, không qua máy ảo
- **Kiểm soát bộ nhớ thủ công**: con trỏ, cấp phát động
- **Đa mô hình**: hướng thủ tục, hướng đối tượng, generic, hàm
- **Tương thích ngược với C**: hầu hết code C chạy được trong C++

**Ứng dụng thực tế:**
- Game engine (Unreal Engine, Unity core)
- Hệ điều hành (Windows, phần lớn Linux kernel modules)
- Trình duyệt (Chrome, Firefox)
- Cơ sở dữ liệu (MySQL, MongoDB)
- Phần mềm nhúng, tài chính high-frequency trading

---

## 2. Cài đặt môi trường

### Trên Termux (Android)

```bash
pkg install clang
clang++ --version
```

### Trên Linux (Ubuntu/Debian)

```bash
sudo apt install g++ build-essential
g++ --version
```

### Trên Windows

Cài [MSYS2](https://www.msys2.org/) hoặc Visual Studio Community (kèm workload "Desktop development with C++").

### Biên dịch chương trình

```bash
# Biên dịch file hello.cpp thành chương trình hello
g++ -std=c++17 -O2 -Wall hello.cpp -o hello

# Chạy
./hello
```

**Giải thích các flag:**
- `-std=c++17`: dùng chuẩn C++17 (khuyên dùng, hoặc c++20 nếu compiler hỗ trợ)
- `-O2`: tối ưu mã máy ở mức 2 (chạy nhanh hơn)
- `-Wall`: bật tất cả cảnh báo (giúp phát hiện lỗi tiềm ẩn)
- `-o hello`: đặt tên file output là `hello`

---

## 3. Chương trình đầu tiên

```cpp
#include <iostream>  // thư viện nhập/xuất chuẩn

int main() {
    std::cout << "Xin chào C++!" << std::endl;
    return 0;
}
```

**Giải thích chi tiết từng dòng:**

- `#include <iostream>`: chỉ thị tiền xử lý (preprocessor directive). Nó bảo compiler "chép" nội dung file `iostream` vào đây trước khi biên dịch. File này chứa khai báo của `std::cout`, `std::cin`.
- `int main()`: hàm chính — điểm bắt đầu của mọi chương trình C++. Kiểu trả về `int` (số nguyên), `return 0` nghĩa là "chương trình kết thúc thành công" (quy ước của hệ điều hành).
- `std::cout`: đối tượng output chuẩn (màn hình). `std::` là namespace — vùng tên để tránh trùng tên. Toán tử `<<` đẩy dữ liệu ra output.
- `std::endl`: xuống dòng + xả bộ đệm (flush).
- Dấu `;` kết thúc mỗi câu lệnh — bắt buộc trong C++.

**Namespace là gì?** Hãy tưởng tượng hai thư viện khác nhau đều có hàm tên `sort`. Namespace giúp phân biệt: `std::sort` vs `thuvienA::sort`. Dùng `using namespace std;` để khỏi gõ `std::` mỗi lần, nhưng trong code lớn nên tránh vì dễ trùng tên.

---

## 4. Biến và kiểu dữ liệu

### Khai báo biến

```cpp
int tuoi = 20;              // số nguyên
double diem = 8.5;          // số thực độ chính xác kép
char grade = 'A';           // ký tự đơn (ngoặc đơn!)
bool dau = true;            // đúng/sai
std::string ten = "Duong";  // chuỗi (cần #include <string>)
```

### Các kiểu số nguyên

| Kiểu | Kích thước (thường) | Miền giá trị |
|------|---------------------|--------------|
| `char` | 1 byte | -128 đến 127 |
| `short` | 2 bytes | -32,768 đến 32,767 |
| `int` | 4 bytes | ~±2.1 tỷ |
| `long` | 4-8 bytes | tùy nền tảng |
| `long long` | 8 bytes | ~±9.2×10¹⁸ |

Thêm `unsigned` phía trước để chỉ chứa số không âm (miền giá trị dương gấp đôi):
```cpp
unsigned int x = 4000000000;  // ok, vượt quá int thường
```

### Kiểu số thực

```cpp
float  f = 3.14f;   // 4 bytes, ~7 chữ số thập phân (nhớ hậu tố f)
double d = 3.14159; // 8 bytes, ~15 chữ số (mặc định nên dùng double)
```

**Tại sao nên dùng `double` thay vì `float`?** `double` chính xác gấp đôi mà trên CPU hiện đại tốc độ tính toán tương đương. Chỉ dùng `float` khi cần tiết kiệm bộ nhớ (mảng hàng triệu phần tử, GPU).

### `auto` — suy luận kiểu tự động (C++11)

```cpp
auto x = 42;        // compiler tự hiểu x là int
auto y = 3.14;      // double
auto s = "hello";   // const char*
```

### Hằng số

```cpp
const double PI = 3.14159;   // không thể thay đổi sau khi gán
constexpr int MAX = 100;      // hằng số tính tại thời điểm biên dịch (C++11)
```

**`const` vs `constexpr`:** `const` là "không đổi khi chạy", `constexpr` là "tính được ngay khi biên dịch". Dùng `constexpr` khi giá trị biết trước để compiler tối ưu.

---

## 5. Toán tử

### Số học

```cpp
int a = 10, b = 3;
int cong = a + b;    // 13
int tru  = a - b;    // 7
int nhan = a * b;    // 30
int chia = a / b;    // 3  (chia nguyên! phần dư bị cắt)
int du   = a % b;    // 1  (toán tử modulo - lấy phần dư)
double c = 10.0 / 3; // 3.333... (một toán hạng là double → chia thực)
```

**Bẫy thường gặp:** `10 / 3` cho ra `3` chứ không phải `3.33`, vì cả hai đều là `int`. Muốn chia thực: `10.0 / 3` hoặc `static_cast<double>(10) / 3`.

### So sánh và logic

```cpp
bool b1 = (a == b);  // bằng nhau?
bool b2 = (a != b);  // khác nhau?
bool b3 = (a > b);   // lớn hơn? (còn <, >=, <=)
bool b4 = (a > 5 && b < 5);  // VÀ logic
bool b5 = (a > 5 || b > 5);  // HOẶC logic
bool b6 = !(a > 5);          // PHỦ ĐỊNH
```

`&&` và `||` có **đoản mạch (short-circuit)**: `false && ...` dừng ngay không xét vế sau; `true || ...` cũng vậy. Tận dụng để viết code an toàn:
```cpp
if (p != nullptr && p->x > 0) { ... }  // nếu p null, vế sau không chạy → không crash
```

### Tăng/giảm và gán kết hợp

```cpp
int i = 5;
i++;      // i = 6 (hậu tố: dùng giá trị cũ rồi mới tăng)
++i;      // i = 7 (tiền tố: tăng trước rồi dùng)
i += 3;   // i = i + 3  (tương tự -=, *=, /=, %=)
```

**Khác nhau giữa `i++` và `++i`:** trong biểu thức phức tạp chúng khác nhau:
```cpp
int i = 5;
int a = i++;  // a = 5, i = 6 (gán trước, tăng sau)
int b = ++i;  // i = 7, b = 7 (tăng trước, gán sau)
```

---

## 6. Nhập xuất dữ liệu

```cpp
#include <iostream>
#include <string>

int main() {
    std::string ten;
    int tuoi;

    std::cout << "Nhap ten: ";
    std::getline(std::cin, ten);  // đọc cả dòng (kể cả dấu cách)

    std::cout << "Nhap tuoi: ";
    std::cin >> tuoi;             // đọc số (dừng ở khoảng trắng)

    std::cout << "Xin chao " << ten << ", ban " << tuoi << " tuoi.\n";
    return 0;
}
```

**`cin >>` vs `getline`:** `cin >> ten` chỉ đọc đến khoảng trắng đầu tiên ("Nguyen Van" → chỉ lấy "Nguyen"). `getline(cin, ten)` đọc cả dòng. **Bẫy:** sau khi dùng `cin >> tuoi`, bộ đệm còn ký tự `\n`, gọi `getline` ngay sẽ đọc phải dòng trống. Khắc phục: `std::cin.ignore()` trước `getline`.

**Định dạng số với `<iomanip>`:**
```cpp
#include <iomanip>
double pi = 3.14159265;
std::cout << std::fixed << std::setprecision(2) << pi;  // 3.14
```

---

## 7. Câu lệnh điều kiện

### if / else if / else

```cpp
int diem = 85;
if (diem >= 90) {
    std::cout << "Xuat sac";
} else if (diem >= 70) {
    std::cout << "Kha";
} else if (diem >= 50) {
    std::cout << "Trung binh";
} else {
    std::cout << "Yeu";
}
```

### switch

```cpp
int ngay = 3;
switch (ngay) {
    case 1: std::cout << "Chu nhat"; break;
    case 2: std::cout << "Thu hai"; break;
    case 3: std::cout << "Thu ba"; break;
    default: std::cout << "Ngay khac";
}
```

**Bẫy `switch`:** quên `break` thì code "rơi xuyên" (fallthrough) xuống case tiếp theo. Đôi khi cố ý dùng, nhưng thường là lỗi.

### Toán tử ba ngôi (ternary)

```cpp
int a = 10, b = 20;
int lon = (a > b) ? a : b;  // = 20
// cú pháp: (điều kiện) ? (giá trị nếu đúng) : (giá trị nếu sai)
```

---

## 8. Vòng lặp

### for

```cpp
for (int i = 0; i < 5; ++i) {
    std::cout << i << " ";  // 0 1 2 3 4
}
```

### Range-based for (C++11) — duyệt container cực gọn

```cpp
int arr[] = {1, 2, 3, 4, 5};
for (int x : arr) {
    std::cout << x << " ";
}
for (int& x : arr) {  // tham chiếu → sửa được phần tử gốc
    x *= 2;
}
```

### while và do-while

```cpp
int i = 0;
while (i < 5) {          // kiểm tra trước, có thể không chạy lần nào
    std::cout << i++;
}

int j = 0;
do {                     // chạy ít nhất 1 lần rồi mới kiểm tra
    std::cout << j++;
} while (j < 5);
```

### break và continue

```cpp
for (int i = 0; i < 10; ++i) {
    if (i == 3) continue;  // bỏ qua lần này, nhảy sang i=4
    if (i == 7) break;     // thoát vòng lặp ngay
    std::cout << i << " "; // 0 1 2 4 5 6
}
```

---

## 9. Hàm

```cpp
// Khai báo + định nghĩa
int cong(int a, int b) {
    return a + b;
}

// Hàm không trả về gì
void in_chao(std::string ten) {
    std::cout << "Chao " << ten << "\n";
}

// Tham số mặc định
double luy_thua(double x, int n = 2) {  // n mặc định = 2
    double kq = 1;
    for (int i = 0; i < n; ++i) kq *= x;
    return kq;
}
luy_thua(3.0);    // = 9.0 (dùng n mặc định)
luy_thua(2.0, 3); // = 8.0
```

### Nạp chồng hàm (function overloading)

Cùng tên hàm, khác danh sách tham số — compiler tự chọn bản phù hợp:

```cpp
int cong(int a, int b) { return a + b; }
double cong(double a, double b) { return a + b; }
std::string cong(std::string a, std::string b) { return a + b; }
```

### Đệ quy

```cpp
long long giai_thua(int n) {
    if (n <= 1) return 1;       // điểm dừng — BẮT BUỘC
    return n * giai_thua(n - 1); // gọi lại chính mình
}
```

**Lưu ý:** mỗi lần gọi đệ quy tốn một khung stack. Đệ quy quá sâu → stack overflow. Ví dụ `giai_thua(100000)` sẽ crash.

### Lambda (hàm ẩn danh, C++11)

```cpp
auto binh_phuong = [](int x) { return x * x; };
std::cout << binh_phuong(5);  // 25

// Dùng với STL algorithm
std::vector<int> v = {3, 1, 4, 1, 5};
std::sort(v.begin(), v.end(), [](int a, int b) { return a > b; });  // giảm dần
```

---

## 10. Mảng

### Mảng tĩnh

```cpp
int arr[5] = {1, 2, 3, 4, 5};
int arr2[5] = {1, 2};        // {1, 2, 0, 0, 0} - phần còn lại = 0
int arr3[] = {1, 2, 3};      // compiler tự đếm: kích thước 3

std::cout << arr[0];  // 1 (chỉ số bắt đầu từ 0!)
arr[2] = 99;          // gán
```

**Bẫy nguy hiểm:** C++ **không kiểm tra biên mảng**. `arr[100] = 5` biên dịch được nhưng ghi đè bộ nhớ lung tung → undefined behavior, crash khó lường. Dùng `std::vector` hoặc `std::array` để an toàn hơn.

### Mảng đa chiều

```cpp
int matrix[2][3] = {{1, 2, 3}, {4, 5, 6}};
std::cout << matrix[1][2];  // 6
```

### `std::array` (C++11) — mảng an toàn hơn

```cpp
#include <array>
std::array<int, 5> a = {1, 2, 3, 4, 5};
std::cout << a.size();   // 5
std::cout << a.at(2);    // 3, có kiểm tra biên (ném ngoại lệ nếu sai)
```

---

## 11. Chuỗi ký tự

Có 2 loại: C-style string (`char[]`) và `std::string`. **Luôn dùng `std::string`** trừ khi có lý do đặc biệt.

```cpp
#include <string>
std::string s = "Hello";
s += " World";              // nối chuỗi
std::cout << s.length();    // 11 (hoặc .size())
std::cout << s[0];          // 'H'
std::string sub = s.substr(6, 5);  // "World" (từ vị trí 6, lấy 5 ký tự)

// Tìm kiếm
size_t pos = s.find("World");
if (pos != std::string::npos) std::cout << "Tim thay o " << pos;

// So sánh (đừng dùng == với char*)
if (s == "Hello World") { ... }  // đúng với std::string
```

---

## 12. Con trỏ

Con trỏ là biến chứa **địa chỉ bộ nhớ** của biến khác. Đây là khái niệm quan trọng nhất (và đáng sợ nhất) của C/C++.

```cpp
int x = 42;
int* p = &x;   // p trỏ tới x. & = "lấy địa chỉ của"

std::cout << p;   // in địa chỉ, VD: 0x7ffd4a3b2c
std::cout << *p;  // 42. * = "lấy giá trị tại địa chỉ" (dereference)

*p = 100;         // đổi x thành 100 thông qua con trỏ!
std::cout << x;   // 100
```

**Sơ đồ tư duy:**
```
x:   [ 42 ]  ở địa chỉ 0x1000
p:   [0x1000]  (p chứa địa chỉ của x)
*p:  đi tới 0x1000, đọc giá trị → 42
```

### Con trỏ null

```cpp
int* p = nullptr;  // không trỏ đi đâu (C++11, thay cho NULL)
if (p != nullptr) { *p = 5; }  // luôn kiểm tra trước khi dùng!
```

Dereference con trỏ null → crash ngay. Đây là lỗi phổ biến nhất.

### Con trỏ và mảng

```cpp
int arr[3] = {10, 20, 30};
int* p = arr;  // tên mảng = địa chỉ phần tử đầu (không cần &)
std::cout << *p;       // 10
std::cout << *(p + 1); // 20 (số học con trỏ: +1 = nhảy 1 phần tử = 4 bytes với int)
p[2];  // = 30 (cú pháp mảng dùng được với con trỏ)
```

### Con trỏ tới con trỏ, con trỏ hàm

```cpp
int x = 5;
int* p = &x;
int** pp = &p;  // con trỏ tới con trỏ
std::cout << **pp;  // 5

// Con trỏ hàm: trỏ tới hàm để gọi gián tiếp (callback)
int cong(int a, int b) { return a + b; }
int (*fp)(int, int) = cong;
std::cout << fp(3, 4);  // 7
```

---

## 13. Tham chiếu

Tham chiếu (`&`) là "bí danh" (alias) của biến — giống con trỏ nhưng an toàn và dễ dùng hơn.

```cpp
int x = 10;
int& r = x;  // r là bí danh của x
r = 20;
std::cout << x;  // 20 (x cũng đổi theo!)
```

**Khác con trỏ ở:** tham chiếu không thể null, không thể "trỏ sang chỗ khác" sau khi gán, không cần `*` để dùng.

**Dùng chính:** truyền tham số cho hàm để hàm sửa được biến gốc (mà không copy tốn kém):

```cpp
void tang_gap_doi(int& n) { n *= 2; }  // tham chiếu → sửa biến gốc
void tang_gap_doi2(int n) { n *= 2; }  // tham trị → chỉ sửa bản copy

int a = 5;
tang_gap_doi(a);   // a = 10
tang_gap_doi2(a);  // a vẫn 10
```

**Quy tắc vàng:** truyền object lớn vào hàm thì dùng `const&` — vừa nhanh (không copy) vừa an toàn (không sửa được):
```cpp
void in(const std::string& s) { std::cout << s; }
```

---

## 14. Struct và Enum

### struct — gom nhiều dữ liệu liên quan

```cpp
struct SinhVien {
    std::string ten;
    int tuoi;
    double diem;
};

SinhVien sv;
sv.ten = "Duong";
sv.tuoi = 20;
sv.diem = 8.5;

// Khởi tạo nhanh (C++11)
SinhVien sv2 = {"Lan", 19, 9.0};
```

### enum — tập hằng số có tên

```cpp
enum class Mau { Do, Xanh, Vang };  // enum class: an toàn kiểu (C++11)

Mau m = Mau::Do;
if (m == Mau::Do) { ... }

// Đừng dùng enum thường (không có class) trong code mới:
// enum Mau { Do, Xanh }; → Do "rò rỉ" ra scope ngoài, dễ trùng tên
```

---

## 15. Lập trình hướng đối tượng

### Class cơ bản

```cpp
#include <string>
#include <iostream>

class HinhChuNhat {
private:  // chỉ code trong class được đụng
    double dai, rong;

public:   // ai cũng đụng được
    // Constructor: chạy khi tạo object
    HinhChuNhat(double d, double r) : dai(d), rong(r) {}

    // Constructor mặc định
    HinhChuNhat() : dai(0), rong(0) {}

    double dien_tich() const {  // const: hàm này không sửa object
        return dai * rong;
    }

    void set_dai(double d) {
        if (d > 0) dai = d;  // validation: đóng gói dữ liệu
    }

    double get_dai() const { return dai; }
};

int main() {
    HinhChuNhat h(5, 3);
    std::cout << h.dien_tich();  // 15
    // h.dai = -5;  // LỖI: dai là private
    h.set_dai(10);  // ok, qua setter có kiểm tra
}
```

**4 tính chất OOP:**
1. **Đóng gói (Encapsulation)**: giấu dữ liệu (`private`), chỉ lộ interface (`public`)
2. **Kế thừa (Inheritance)**: class con dùng lại code class cha
3. **Đa hình (Polymorphism)**: cùng một lời gọi, hành vi khác nhau tùy object thật
4. **Trừu tượng (Abstraction)**: chỉ quan tâm "làm gì", không cần biết "làm thế nào"

### Constructor / Destructor

```cpp
class File {
    FILE* f;
public:
    File(const char* path) { f = fopen(path, "r"); }  // mở khi tạo
    ~File() { if (f) fclose(f); }  // destructor: đóng khi object chết
};
```

**RAII** (Resource Acquisition Is Initialization) — triết lý cốt lõi của C++: tài nguyên (file, bộ nhớ, lock) gắn với vòng đời object. Object chết → tài nguyên tự giải phóng. Nhờ vậy C++ không cần garbage collector mà vẫn ít rò rỉ nếu code đúng.

### `this`

```cpp
class A {
    int x;
public:
    void set(int x) {
        this->x = x;  // this = con trỏ tới object hiện tại
    }
};
```

---

## 16. Kế thừa và đa hình

```cpp
class DongVat {
public:
    virtual std::string keu() const { return "..."; }  // virtual: cho phép ghi đè
    virtual ~DongVat() = default;  // destructor ảo: QUAN TRỌNG khi kế thừa
};

class Cho : public DongVat {
public:
    std::string keu() const override { return "Gau gau!"; }  // override
};

class Meo : public DongVat {
public:
    std::string keu() const override { return "Meo meo!"; }
};

int main() {
    DongVat* a = new Cho();
    DongVat* b = new Meo();
    std::cout << a->keu() << "\n";  // "Gau gau!" (đa hình: gọi đúng bản của Cho)
    std::cout << b->keu() << "\n";  // "Meo meo!"
    delete a; delete b;
}
```

**Tại sao cần `virtual`?** Không có `virtual`, `a->keu()` sẽ gọi bản của `DongVat` (tĩnh, quyết định lúc biên dịch). Có `virtual`, quyết định lúc chạy dựa trên object thật → đa hình.

**Tại sao destructor phải virtual?** `delete a` với `a` là `DongVat*` trỏ tới `Cho`: nếu destructor không virtual, chỉ destructor của `DongVat` chạy → phần của `Cho` không được dọn → rò rỉ. Quy tắc: **class nào có hàm virtual thì destructor cũng phải virtual**.

### Pure virtual — class trừu tượng

```cpp
class Hinh {
public:
    virtual double dien_tich() const = 0;  // = 0: bắt class con phải implement
    virtual ~Hinh() = default;
};
// Không thể tạo object Hinh trực tiếp, chỉ dùng làm "bản thiết kế"
```

---

## 17. Template

Viết code một lần, dùng cho nhiều kiểu dữ liệu:

```cpp
template <typename T>
T lon_nhat(T a, T b) {
    return (a > b) ? a : b;
}

lon_nhat(3, 5);        // T = int
lon_nhat(3.5, 2.1);    // T = double
lon_nhat("a"s, "b"s);  // T = std::string
```

Class template:
```cpp
template <typename T>
class Hop {
    T data;
public:
    Hop(T d) : data(d) {}
    T lay() const { return data; }
};

Hop<int> h1(42);
Hop<std::string> h2("hello");
```

Đây là nền tảng của STL: `std::vector<int>`, `std::map<std::string, int>` đều là template.

---

## 18. STL - Thư viện chuẩn

### `vector` — mảng động (dùng nhiều nhất)

```cpp
#include <vector>
std::vector<int> v;           // rỗng
v.push_back(10);              // thêm cuối
v.push_back(20);
std::cout << v[0];            // 10
std::cout << v.size();        // 2
v.pop_back();                 // xóa cuối

for (int x : v) std::cout << x << " ";
```

### `string` — đã học ở mục 11

### `map` / `unordered_map` — từ điển key-value

```cpp
#include <map>
#include <unordered_map>

std::map<std::string, int> tuoi;  // sắp xếp theo key (cây đỏ-đen)
tuoi["Duong"] = 20;
tuoi["Lan"] = 19;
std::cout << tuoi["Duong"];  // 20

std::unordered_map<std::string, int> m2;  // hash table, nhanh hơn, không sắp xếp
```

### `set` — tập hợp không trùng

```cpp
#include <set>
std::set<int> s = {3, 1, 3, 2};
for (int x : s) std::cout << x << " ";  // 1 2 3 (tự sắp xếp + loại trùng)
```

### Thuật toán `<algorithm>`

```cpp
#include <algorithm>
std::vector<int> v = {5, 2, 8, 1};
std::sort(v.begin(), v.end());                    // 1 2 5 8
std::reverse(v.begin(), v.end());                 // 8 5 2 1
auto it = std::find(v.begin(), v.end(), 5);       // tìm
int dem = std::count(v.begin(), v.end(), 8);      // đếm
```

---

## 19. Xử lý ngoại lệ

```cpp
#include <stdexcept>

double chia(double a, double b) {
    if (b == 0) throw std::runtime_error("Chia cho 0!");
    return a / b;
}

int main() {
    try {
        std::cout << chia(10, 0);
    } catch (const std::runtime_error& e) {
        std::cout << "Loi: " << e.what();
    } catch (...) {
        std::cout << "Loi khong xac dinh";
    }
}
```

**Khi nào dùng exception?** Cho lỗi "ngoại lệ" thật sự (file không tồn tại, mạng đứt). Đừng dùng cho luồng điều khiển thông thường (như kiểm tra input người dùng) — tốn hiệu năng và khó đọc.

---

## 20. Quản lý bộ nhớ

### Cấp phát động

```cpp
int* p = new int(42);      // cấp 1 số nguyên
delete p;                  // GIẢI PHÓNG — quên là rò rỉ!

int* arr = new int[10];    // cấp mảng 10 phần tử
delete[] arr;              // nhớ dùng delete[] cho mảng!
```

### Smart pointer (C++11) — nên dùng thay cho `new`/`delete` thô

```cpp
#include <memory>

// unique_ptr: sở hữu duy nhất, tự giải phóng khi ra khỏi scope
auto p1 = std::make_unique<int>(42);
// không cần delete!

// shared_ptr: nhiều chủ sở hữu, đếm tham chiếu, hết người dùng mới giải phóng
auto p2 = std::make_shared<int>(100);
auto p3 = p2;  // cùng trỏ tới 100, đếm = 2
```

**Quy tắc hiện đại:** gần như không bao giờ dùng `new`/`delete` thô nữa. Dùng `make_unique`/`make_shared`, hoặc để object trên stack.

---

## 21. File I/O

```cpp
#include <fstream>
#include <string>

// Ghi file
std::ofstream out("data.txt");
out << "Hello\n" << 42 << "\n";
out.close();

// Đọc file
std::ifstream in("data.txt");
std::string line;
while (std::getline(in, line)) {
    std::cout << line << "\n";
}
in.close();
```

Luôn kiểm tra file có mở được không: `if (!in) { std::cerr << "Khong mo duoc file!"; }`

---

## 22. C++ hiện đại (11/14/17/20)

| Tính năng | Chuẩn | Dùng để làm gì |
|-----------|-------|----------------|
| `auto` | 11 | Suy luận kiểu |
| Range-based for | 11 | Duyệt container gọn |
| Smart pointer | 11 | Quản lý bộ nhớ an toàn |
| Lambda | 11 | Hàm ẩn danh |
| `nullptr` | 11 | Thay NULL |
| Move semantics (`&&`) | 11 | Tránh copy tốn kém |
| `constexpr` | 11 | Tính lúc biên dịch |
| Structured binding | 17 | `auto [a, b] = pair;` |
| `std::optional` | 17 | Giá trị có thể "không có" |
| `std::string_view` | 17 | Nhìn chuỗi không copy |
| Concepts | 20 | Ràng buộc template |
| Ranges | 20 | Xử lý dãy dữ liệu kiểu mới |
| Coroutine | 20 | Bất đồng bộ |

**Ví dụ structured binding (C++17):**
```cpp
std::map<std::string, int> m = {{"a", 1}};
for (auto [key, val] : m) {
    std::cout << key << ": " << val;
}
```

---

## 23. Best practices

1. **Luôn biên dịch với `-Wall -Wextra`**: để compiler bắt lỗi giúp bạn
2. **Dùng `const` triệt để**: tham số không sửa → `const&`, hàm không sửa object → `const`
3. **Ưu tiên `std::vector`/`std::string` thay vì mảng thô/con trỏ char**
4. **Dùng smart pointer thay vì `new`/`delete` thô**
5. **Destructor virtual nếu class có hàm virtual**
6. **RAII**: tài nguyên gắn với vòng đời object
7. **Không `using namespace std;` trong header file**
8. **Khởi tạo biến ngay khi khai báo**, dùng `{}` (uniform init): `int x{5};`
9. **Tránh `new` nếu object sống được trên stack**
10. **Đặt tên rõ nghĩa**: `so_sinh_vien` thay vì `n`, `tinh_diem_trung_binh()` thay vì `f()`

---

*Tài liệu được viết cho người mới bắt đầu, giải thích từ gốc. Hãy gõ lại từng ví dụ và chạy thử để hiểu sâu.*
