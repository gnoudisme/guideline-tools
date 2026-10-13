# Hướng dẫn Python chi tiết

> Tài liệu học Python từ cơ bản đến OOP, decorator, virtualenv — giải thích chi tiết từng khái niệm kèm ví dụ chạy được thật.

## Mục lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Cài đặt](#2-cài-đặt)
3. [Chương trình đầu tiên](#3-chương-trình-đầu-tiên)
4. [Biến và kiểu dữ liệu](#4-biến-và-kiểu-dữ-liệu)
5. [Toán tử](#5-toán-tử)
6. [Nhập xuất](#6-nhập-xuất)
7. [Điều kiện](#7-điều-kiện)
8. [Vòng lặp](#8-vòng-lặp)
9. [Hàm](#9-hàm)
10. [List](#10-list)
11. [Tuple](#11-tuple)
12. [Dict](#12-dict)
13. [Set](#13-set)
14. [String nâng cao](#14-string-nâng-cao)
15. [Comprehension](#15-comprehension)
16. [Xử lý lỗi](#16-xử-lý-lỗi)
17. [File I/O](#17-file-io)
18. [Module và Package](#18-module-và-package)
19. [Virtualenv và pip](#19-virtualenv-và-pip)
20. [OOP trong Python](#20-oop-trong-python)
21. [Decorator](#21-decorator)
22. [Generator và Iterator](#22-generator-và-iterator)
23. [*args và **kwargs](#23-args-và-kwargs)
24. [Lambda, map, filter](#24-lambda-map-filter)
25. [Type hints](#25-type-hints)
26. [Best practices](#26-best-practices)

---

## 1. Giới thiệu

**Python** do Guido van Rossum tạo ra năm 1991, tên đặt theo nhóm hài Monty Python. Là ngôn ngữ **thông dịch** (interpreted), **động kiểu** (dynamically typed), nổi tiếng vì cú pháp gần tiếng Anh, dễ đọc.

**Đặc điểm:**
- **Dễ học nhất** trong các ngôn ngữ phổ biến — cú pháp gọn, ít boilerplate
- **Đa năng**: web (Django/FastAPI), AI/ML (PyTorch, TensorFlow), data science (pandas), automation, scripting
- **Thư viện khổng lồ**: PyPI có hàng trăm nghìn package
- **Nhược điểm**: chạy chậm hơn ngôn ngữ biên dịch (nhưng đủ nhanh cho 95% công việc, phần nặng đã có lib C tối ưu)

**Triết lý** (gõ `import this` trong Python để xem "Zen of Python"):
- "Beautiful is better than ugly" — code đẹp hơn code xấu
- "Readability counts" — khả năng đọc hiểu là quan trọng
- "There should be one obvious way to do it" — mỗi việc nên có một cách rõ ràng

---

## 2. Cài đặt

```bash
# Termux
pkg install python
python --version

# Linux
sudo apt install python3 python3-pip

# Kiểm tra
python3 --version  # nên là 3.9+
```

**Chạy code Python có 2 cách:**
```bash
# 1. Chạy file
python3 hello.py

# 2. REPL (gõ lệnh tương tác, học rất tốt)
python3
>>> print("hello")
```

---

## 3. Chương trình đầu tiên

```python
print("Xin chào Python!")
```

Một dòng duy nhất! Không cần `#include`, không cần `main()`, không cần `;`. Đây là lý do Python được chọn để dạy lập trình.

**Thụt đầu dòng (indentation) là cú pháp bắt buộc** — thay cho `{}` của ngôn ngữ khác:
```python
if True:
    print("dòng này thụt vào 4 spaces")  # thuộc khối if
print("dòng này không thụt")  # ngoài khối if
```

Dùng **4 spaces** (không dùng tab). Trộn tab và space → lỗi.

---

## 4. Biến và kiểu dữ liệu

### Không cần khai báo kiểu!

```python
tuoi = 20          # int
diem = 8.5         # float
ten = "Duong"     # str
dau = True        # bool (viết hoa T!)
chua = None       # "không có gì" (như null)
```

Biến trong Python là **cái nhãn dán lên object**, không phải "hộp chứa giá trị" như C++. Gán `a = b` là dán thêm nhãn vào cùng object.

### Kiểm tra kiểu

```python
print(type(tuoi))  # <class 'int'>
```

### Ép kiểu

```python
x = int("42")      # "42" -> 42
y = float("3.14")  # -> 3.14
z = str(100)       # -> "100"
t = bool(1)        # -> True (0, "", [], None là falsy, còn lại truthy)
```

### Hằng số (quy ước)

Python không có `const` thật sự. Quy ước: tên VIẾT_HOA là hằng số (mọi người tự giác không đổi):
```python
PI = 3.14159
MAX_USER = 100
```

---

## 5. Toán tử

```python
a, b = 10, 3
print(a + b)   # 13
print(a - b)   # 7
print(a * b)   # 30
print(a / b)   # 3.333... (chia thực LUÔN, khác C++!)
print(a // b)  # 3 (chia lấy phần nguyên)
print(a % b)   # 1
print(a ** b)  # 1000 (lũy thừa!)

# So sánh
print(a == b, a != b, a > b, a < b, a >= b, a <= b)

# Logic (dùng từ tiếng Anh, không dùng && ||)
print(a > 5 and b < 5)  # True
print(a > 5 or b > 5)   # True
print(not (a > 5))      # False

# So sánh chuỗi trong khoảng (độc đáo của Python!)
x = 5
print(1 < x < 10)  # True (không cần viết 1 < x and x < 10)
```

**Falsy values** (coi như False): `0`, `0.0`, `""`, `[]`, `{}`, `None`, `False`. Còn lại là truthy.

---

## 6. Nhập xuất

```python
ten = input("Nhập tên: ")       # luôn trả về str!
tuoi = int(input("Nhập tuổi: ")) # ép sang int

# In với f-string (Python 3.6+, cách hiện đại nhất)
print(f"Xin chào {ten}, bạn {tuoi} tuổi")
print(f"Năm sau bạn {tuoi + 1} tuổi")  # tính toán trong {}

# Các cách cũ (biết để đọc code người khác)
print("Xin chào {}, bạn {} tuổi".format(ten, tuoi))
print("Xin chào %s, bạn %d tuổi" % (ten, tuoi))
```

---

## 7. Điều kiện

```python
diem = 85
if diem >= 90:
    print("Xuất sắc")
elif diem >= 70:    # elif (không phải else if!)
    print("Khá")
elif diem >= 50:
    print("Trung bình")
else:
    print("Yếu")

# Toán tử ba ngôi (viết ngược so với C++)
lon = a if a > b else b
```

---

## 8. Vòng lặp

### for — duyệt trực tiếp, không cần index

```python
# Duyệt list
for x in [1, 2, 3]:
    print(x)

# range(): sinh dãy số
for i in range(5):      # 0..4
    print(i)
for i in range(2, 10, 2):  # từ 2 đến <10, bước 2: 2 4 6 8
    print(i)

# Vừa index vừa giá trị
for i, x in enumerate(["a", "b", "c"]):
    print(i, x)  # 0 a, 1 b, 2 c

# Duyệt dict
for key, val in {"a": 1}.items():
    print(key, val)
```

### while

```python
i = 0
while i < 5:
    print(i)
    i += 1
```

### break / continue / else của vòng lặp

```python
for i in range(10):
    if i == 3:
        continue  # bỏ qua
    if i == 7:
        break     # thoát
    print(i)
else:
    print("Vòng lặp chạy hết không break")  # else của for: chạy nếu KHÔNG break
```

`for...else` là tính năng độc đáo của Python: khối `else` chạy khi vòng lặp kết thúc tự nhiên (không bị `break`).

---

## 9. Hàm

```python
def cong(a, b):
    """Cộng hai số."""  # docstring: tài liệu của hàm
    return a + b

def in_chao(ten="bạn"):  # tham số mặc định
    print(f"Chào {ten}")

in_chao()        # "Chào bạn"
in_chao("Dương") # "Chào Dương"

# Gọi với tên tham số (keyword argument)
def dang_ky(ten, tuoi):
    print(ten, tuoi)
dang_ky(tuoi=20, ten="Dương")  # không cần đúng thứ tự!
```

### Hàm là object hạng nhất

```python
def binh_phuong(x):
    return x * x

f = binh_phuong  # gán hàm cho biến (không có ())
print(f(5))      # 25
```

---

## 10. List

List là cấu trúc dữ liệu dùng nhiều nhất trong Python — mảng động chứa được nhiều kiểu.

```python
ds = [1, 2, 3, "hello", 4.5]  # nhiều kiểu trong 1 list cũng được!
print(ds[0])     # 1
print(ds[-1])    # 4.5 (index âm: từ cuối lên!)
print(ds[1:3])   # [2, 3] (slice: từ 1 đến <3)
print(ds[::-1])  # đảo ngược!

# Thêm/xóa
ds.append(6)       # thêm cuối
ds.insert(0, 0)    # chèn ở vị trí 0
ds.remove("hello") # xóa theo giá trị
x = ds.pop()       # lấy ra + xóa phần tử cuối
del ds[0]          # xóa theo index

# Thông tin
print(len(ds))     # độ dài
print(3 in ds)     # kiểm tra tồn tại: True/False

# Sắp xếp
so = [3, 1, 2]
so.sort()                    # sắp xếp tại chỗ: [1, 2, 3]
so2 = sorted(so, reverse=True) # trả về list mới: [3, 2, 1]
```

**Bẫy list lồng nhau:**
```python
a = [[0] * 3] * 2  # [[0,0,0], [0,0,0]] nhưng 2 hàng CÙNG trỏ 1 list!
a[0][0] = 99
print(a)  # [[99,0,0], [99,0,0]] — cả 2 hàng cùng đổi!
# Đúng: a = [[0]*3 for _ in range(2)]
```

---

## 11. Tuple

Tuple giống list nhưng **bất biến** (immutable) — dùng cho dữ liệu cố định.

```python
diem = (8.5, 9.0, 7.5)
print(diem[0])  # 8.5
# diem[0] = 10  # LỖI: không sửa được

# Unpacking: gán nhiều biến một lúc
x, y = 10, 20
ten, tuoi = ("Dương", 20)

# Đổi giá trị không cần biến tạm!
a, b = b, a

# Hàm trả về nhiều giá trị (thật ra là tuple)
def chia_lay_du(a, b):
    return a // b, a % b
q, r = chia_lay_du(10, 3)  # q=3, r=1
```

---

## 12. Dict

Dict (từ điển) lưu cặp key-value, tra cứu cực nhanh (hash table).

```python
sv = {"ten": "Dương", "tuoi": 20, "diem": 8.5}
print(sv["ten"])              # "Dương"
sv["lop"] = "A1"             # thêm key mới
sv["tuoi"] = 21              # sửa

# Truy cập an toàn
print(sv.get("diem"))        # 8.5
print(sv.get("truong", "N/A"))  # key không có → "N/A" (không lỗi!)

# Duyệt
for k, v in sv.items():
    print(f"{k}: {v}")

# Xóa
del sv["lop"]
tuoi = sv.pop("tuoi")  # lấy ra + xóa
```

**Dict trong Python 3.7+ giữ thứ tự chèn** — duyệt sẽ ra đúng thứ tự bạn thêm vào.

---

## 13. Set

Tập hợp không trùng lặp, hỗ trợ toán tử tập hợp.

```python
s = {1, 2, 3, 3, 2}  # {1, 2, 3} (tự loại trùng)
s.add(4)
s.remove(1)

a = {1, 2, 3}
b = {3, 4, 5}
print(a | b)  # hợp: {1,2,3,4,5}
print(a & b)  # giao: {3}
print(a - b)  # hiệu: {1,2}
```

Dùng set để loại trùng trong list siêu nhanh: `list(set([1,2,2,3]))` → `[1,2,3]`

---

## 14. String nâng cao

```python
s = "  Hello Python  "
print(s.strip())        # "Hello Python" (xóa khoảng trắng 2 đầu)
print(s.lower())        # chữ thường
print(s.upper())        # chữ HOA
print(s.replace("Python", "World"))

# Tách/gộp
cau = "a,b,c"
ds = cau.split(",")     # ["a","b","c"]
print("-".join(ds))     # "a-b-c"

# Kiểm tra
print("hello".startswith("he"))  # True
print("123".isdigit())           # True

# f-string nâng cao
ten, diem = "Dương", 8.567
print(f"{ten:>10}")      # căn phải 10 ký tự
print(f"{diem:.2f}")     # 8.57 (2 số thập phân)
print(f"{1000000:,}")    # 1,000,000 (phân cách hàng nghìn)
```

---

## 15. Comprehension

Cách viết gọn "vòng lặp sinh dữ liệu" — rất Pythonic.

```python
# List comprehension
binh_phuong = [x**2 for x in range(10)]           # [0,1,4,...,81]
chan = [x for x in range(10) if x % 2 == 0]       # [0,2,4,6,8]

# Dict comprehension
binh = {x: x**2 for x in range(5)}  # {0:0, 1:1, 2:4, 3:9, 4:16}

# Set comprehension
s = {x % 3 for x in range(10)}  # {0, 1, 2}

# So sánh: viết thường cần 3 dòng, comprehension 1 dòng
# Thường:          Gọn:
# kq = []          kq = [x*2 for x in range(5)]
# for x in range(5):
#     kq.append(x*2)
```

---

## 16. Xử lý lỗi

```python
try:
    x = int(input("Nhập số: "))
    print(10 / x)
except ValueError:
    print("Bạn nhập không phải số!")
except ZeroDivisionError:
    print("Không chia được cho 0!")
except Exception as e:  # bắt mọi lỗi khác
    print(f"Lỗi lạ: {e}")
else:
    print("Không lỗi, chạy khối này")  # chạy nếu try thành công
finally:
    print("Luôn chạy, dù lỗi hay không")  # dọn dẹp tài nguyên
```

**Tự ném lỗi:**
```python
def chia(a, b):
    if b == 0:
        raise ValueError("b không được bằng 0")
    return a / b
```

---

## 17. File I/O

```python
# Ghi file (with tự đóng file, khỏi lo quên)
with open("data.txt", "w", encoding="utf-8") as f:
    f.write("Xin chào\n")
    f.write("Dòng 2\n")

# Đọc file
with open("data.txt", "r", encoding="utf-8") as f:
    noi_dung = f.read()       # đọc hết
    # f.readline()            # đọc 1 dòng
    # f.readlines()           # đọc hết thành list các dòng

for dong in open("data.txt", encoding="utf-8"):  # duyệt từng dòng (tiết kiệm RAM)
    print(dong.strip())
```

**Luôn dùng `with`** để file tự đóng ngay cả khi lỗi. Luôn chỉ định `encoding="utf-8"` để tránh lỗi font tiếng Việt trên Windows.

---

## 18. Module và Package

```python
# utils.py
def cong(a, b):
    return a + b

PI = 3.14
```

```python
# main.py
import utils                    # dùng: utils.cong(1, 2)
from utils import cong, PI      # dùng trực tiếp: cong(1, 2)
from utils import *             # (tránh: không rõ nguồn gốc)
import utils as u               # bí danh: u.cong(1, 2)
```

**`if __name__ == "__main__"`** — code chỉ chạy khi file được chạy trực tiếp, không chạy khi bị import:
```python
# utils.py
def cong(a, b):
    return a + b

if __name__ == "__main__":
    print(cong(1, 2))  # chỉ chạy khi: python utils.py
```

---

## 19. Virtualenv và pip

**Vấn đề:** mỗi project cần version thư viện khác nhau. Cài chung vào hệ thống → xung đột.

**Giải pháp:** môi trường ảo riêng cho từng project.

```bash
# Tạo môi trường ảo
python3 -m venv venv

# Kích hoạt
source venv/bin/activate      # Linux/macOS/Termux
# venv\Scripts\activate       # Windows

# Giờ pip cài vào venv, không đụng hệ thống
pip install requests

# Lưu danh sách thư viện
pip freeze > requirements.txt

# Người khác cài lại đúng version
pip install -r requirements.txt

# Thoát venv
deactivate
```

---

## 20. OOP trong Python

```python
class SinhVien:
    # Biến của class (chung cho mọi object)
    truong = "ĐH Mở"

    # Constructor
    def __init__(self, ten, tuoi):
        self.ten = ten    # self = this, BẮT BUỘC tham số đầu
        self.tuoi = tuoi

    def gioi_thieu(self):
        return f"Tôi là {self.ten}, {self.tuoi} tuổi"

    def __str__(self):  # như toString(): print(sv) gọi hàm này
        return f"SinhVien({self.ten})"

sv = SinhVien("Dương", 20)
print(sv.gioi_thieu())
print(sv)  # SinhVien(Dương)
```

### Kế thừa

```python
class Nguoi:
    def __init__(self, ten):
        self.ten = ten
    def chao(self):
        return f"Chào, tôi là {self.ten}"

class SinhVien(Nguoi):  # kế thừa
    def __init__(self, ten, mssv):
        super().__init__(ten)  # gọi constructor cha
        self.mssv = mssv

    def chao(self):  # ghi đè
        return super().chao() + f", MSSV {self.mssv}"
```

Python không có `private` thật sự. Quy ước: `_ten` = "đừng đụng từ ngoài" (nhắc nhở), `__ten` = name mangling (khó đụng hơn).

---

## 21. Decorator

Decorator "bọc" thêm chức năng cho hàm mà không sửa code hàm gốc.

```python
def do_thoi_gian(func):
    import time
    def wrapper(*args, **kwargs):
        bat_dau = time.time()
        kq = func(*args, **kwargs)  # gọi hàm gốc
        print(f"Chạy mất {time.time() - bat_dau:.2f}s")
        return kq
    return wrapper

@do_thoi_gian  # = tinh = do_thoi_gian(tinh)
def tinh(n):
    return sum(range(n))

tinh(1000000)  # in thêm thời gian chạy
```

Decorator có sẵn hay dùng: `@property`, `@staticmethod`, `@classmethod`.

```python
class HinhTron:
    def __init__(self, r):
        self.r = r

    @property
    def dien_tich(self):  # gọi như thuộc tính: ht.dien_tich (không cần ())
        return 3.14 * self.r ** 2
```

---

## 22. Generator và Iterator

Generator sinh giá trị **lười biếng** (lazy) — từng cái một, không tốn RAM.

```python
def dem_vo_han():
    n = 0
    while True:
        yield n  # yield = "trả về tạm, lần sau chạy tiếp từ đây"
        n += 1

gen = dem_vo_han()
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 2

# Generator expression (như list comprehension nhưng ())
tong = sum(x**2 for x in range(10**8))  # không tốn RAM như list!
```

Dùng generator khi xử lý dữ liệu lớn (đọc file hàng GB, stream).

---

## 23. *args và **kwargs

```python
def tong(*args):  # nhận bao nhiêu số cũng được → tuple
    return sum(args)

print(tong(1, 2, 3, 4))  # 10

def in_thong_tin(**kwargs):  # nhận key=value → dict
    for k, v in kwargs.items():
        print(f"{k}: {v}")

in_thong_tin(ten="Dương", tuoi=20)

# Kết hợp
def ham(a, *args, b=10, **kwargs):
    ...
```

---

## 24. Lambda, map, filter

```python
binh_phuong = lambda x: x**2  # hàm ẩn danh 1 dòng
print(binh_phuong(5))  # 25

ds = [1, 2, 3, 4, 5]
print(list(map(binh_phuong, ds)))       # [1,4,9,16,25] (áp hàm cho mọi phần tử)
print(list(filter(lambda x: x%2==0, ds)))  # [2,4] (lọc)

# Nhưng người Python thích comprehension hơn:
print([x**2 for x in ds])
print([x for x in ds if x % 2 == 0])
```

---

## 25. Type hints

Python động kiểu, nhưng có thể **gợi ý kiểu** để IDE bắt lỗi và code rõ ràng hơn:

```python
def cong(a: int, b: int) -> int:
    return a + b

ten: str = "Dương"
ds: list[int] = [1, 2, 3]           # Python 3.9+
tuoi_hoac_khong: int | None = None  # Python 3.10+
```

Type hints **không bắt buộc lúc chạy** (Python vẫn không kiểm tra), nhưng công cụ như `mypy` sẽ kiểm tra giúp bạn:
```bash
pip install mypy
mypy main.py
```

---

## 26. Best practices

1. **Tuân thủ PEP 8**: 4 spaces, tên hàm/biến `snake_case`, class `PascalCase`, dòng < 79 ký tự (dùng `black` để auto-format)
2. **Dùng virtualenv cho mọi project**
3. **Dùng `with` cho file và tài nguyên**
4. **Ưu tiên comprehension** thay vì vòng lặp append
5. **Dùng f-string** thay vì `%` hay `.format()`
6. **Viết docstring** cho hàm public
7. **Dùng `get()` cho dict** thay vì `[]` khi key có thể không tồn tại
8. **Không dùng `except:` trần** — luôn chỉ rõ loại lỗi
9. **Dùng `enumerate()`** thay vì `range(len(...))`
10. **Đọc "Zen of Python"** (`import this`) mỗi khi phân vân cách viết

---

*Tài liệu viết cho người mới, nhấn mạnh "Pythonic way" — cách viết đúng chất Python chứ không phải dịch từ ngôn ngữ khác sang.*
