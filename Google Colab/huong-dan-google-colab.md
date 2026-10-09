# Hướng Dẫn Toàn Diện Google Colab

> Tài liệu tiếng Việt chi tiết về mọi tính năng và cách sử dụng Google Colab — từ cơ bản đến nâng cao.
> Cập nhật: 2026-10-10 | Tác giả: Capybara 🦫

---

## Mục Lục

1. [Google Colab Là Gì?](#1-google-colab-là-gì)
2. [Bắt Đầu Sử Dụng](#2-bắt-đầu-sử-dụng)
3. [Giao Diện Chi Tiết](#3-giao-diện-chi-tiết)
4. [Cell — Đơn Vị Cơ Bản](#4-cell--đơn-vị-cơ-bản)
5. [Runtime — Sức Mạnh Tính Toán](#5-runtime--sức-mạnh-tính-toán)
6. [Quản Lý File & Thư Mục](#6-quản-lý-file--thư-mục)
7. [Cài Đặt Package & Thư Viện](#7-cài-đặt-package--thư-viện)
8. [Magic Commands](#8-magic-commands)
9. [Phím Tắt](#9-phím-tắt)
10. [Vẽ Biểu Đồ & Hiển Thị Dữ Liệu](#10-vẽ-biểu-đồ--hiển-thị-dữ-liệu)
11. [Form Tương Tác & Widget](#11-form-tương-tác--widget)
12. [Làm Việc Với Dữ Liệu](#12-làm-việc-với-dữ-liệu)
13. [Chia Sẻ & Cộng Tác](#13-chia-sẻ--cộng-tác)
14. [Mẹo & Thủ Thuật Nâng Cao](#14-mẹo--thủ-thuật-nâng-cao)
15. [Xử Lý Lỗi Thường Gặp](#15-xử-lý-lỗi-thường-gặp)
16. [Colab Pro & So Sánh Gói](#16-colab-pro--so-sánh-gói)
17. [Bảo Mật & Lưu Ý](#17-bảo-mật--lưu-ý)
18. [Tài Liệu Tham Khảo](#18-tài-liệu-tham-khảo)

---

## 1. Google Colab Là Gì?

### 1.1 Định Nghĩa

**Google Colaboratory** (Colab) là môi trường phát triển Jupyter Notebook miễn phí chạy trên cloud của Google. Bạn viết code Python trên trình duyệt và chạy trên máy chủ của Google — không cần cài đặt Python, không cần máy mạnh.

### 1.2 Ai Nên Dùng Colab?

- **Học lập trình Python:** Không cần cài đặt, mở trình duyệt là code được ngay
- **Học Machine Learning / AI:** Miễn phí GPU để train model
- **Phân tích dữ liệu:** Có sẵn pandas, numpy, matplotlib
- **Thử nghiệm nhanh:** Test thư viện mới mà không làm bẩn máy local
- **Dạy học:** Chia sẻ notebook cho học viên dễ dàng

### 1.3 So Sánh Với Jupyter Local

| Tiêu chí | Google Colab | Jupyter Local |
|----------|--------------|---------------|
| Cài đặt | Không cần | Cần cài Python + Jupyter |
| GPU miễn phí | ✅ Có (T4) | ❌ Dùng GPU máy mình |
| Chia sẻ | Link như Google Docs | Phải gửi file |
| Lưu trữ | Google Drive | Ổ cứng máy |
| Internet | Bắt buộc | Không cần |
| Giới hạn thời gian | ~12h/session | Không giới hạn |

---

## 2. Bắt Đầu Sử Dụng

### 2.1 Truy Cập

1. Mở trình duyệt → https://colab.research.google.com
2. Đăng nhập tài khoản Google
3. Bấm **Notebook mới** để tạo notebook trắng

### 2.2 Mở Notebook Có Sẵn

**Tệp** → **Mở notebook** → Chọn 1 trong 4 tab:
- **Ví dụ:** Notebook mẫu của Google (học theo rất tốt)
- **Google Drive:** Notebook bạn đã lưu trên Drive
- **GitHub:** Mở notebook từ bất kỳ repo GitHub nào (dán URL)
- **Tải lên:** Upload file `.ipynb` từ máy

### 2.3 Lưu Notebook

- Tự động lưu vào Google Drive (thư mục `Colab Notebooks`)
- **Tệp** → **Lưu** (Ctrl + S)
- **Tệp** → **Lưu bản sao trong Drive**: tạo bản copy
- **Tệp** → **Lưu bản sao trong GitHub**: push trực tiếp lên repo

### 2.4 Đổi Tên Notebook

Bấm vào tên ở góc trên bên trái (mặc định là `Untitled.ipynb`) → nhập tên mới.

---

## 3. Giao Diện Chi Tiết

### 3.1 Thanh Menu Trên Cùng

#### Menu Tệp
- **Notebook mới:** Tạo notebook trắng
- **Mở notebook:** Mở từ Drive/GitHub/upload
- **Lưu / Lưu bản sao:** Lưu notebook
- **Tải xuống:** Xuất thành `.ipynb` hoặc `.py`
- **In:** In notebook ra giấy/PDF
- **Chia sẻ:** Mở dialog chia sẻ

#### Menu Chỉnh Sửa
- **Hoàn tác / Làm lại:** Ctrl+Z / Ctrl+Y
- **Tìm và thay thế:** Ctrl+H
- **Xóa mọi output:** Dọn sạch kết quả chạy

#### Menu Chế Độ Xem
- **Mục lục:** Hiện/ẩn outline bên trái
- **Ẩn mã / Ẩn output:** Thu gọn để trình bày

#### Menu Chèn
- **Ô mã:** Thêm cell code (phím tắt: Ctrl+M B)
- **Ô văn bản:** Thêm cell Markdown

#### Menu Thời Gian Chạy (Quan Trọng Nhất)
- **Chạy tất cả:** Chạy mọi cell từ trên xuống
- **Chạy trước / Chạy sau:** Chạy từ đầu đến cell hiện tại / từ cell hiện tại đến cuối
- **Ngắt thực thi:** Dừng cell đang chạy (như Ctrl+C)
- **Khởi động lại thời gian chạy:** Xóa hết biến, bắt đầu lại
- **Thay đổi loại thời gian chạy:** Đổi CPU/GPU/TPU
- **Quản lý phiên:** Xem các runtime đang chạy

#### Menu Công Cụ
- **Lệnh:** Tìm kiếm lệnh bằng tên (Ctrl+Shift+P)
- **Cài đặt:** Font, theme, editor
- **Phím tắt:** Xem/sửa phím tắt

### 3.2 Thanh Bên Trái

| Biểu tượng | Chức năng |
|------------|-----------|
| 📁 **Tệp** | Duyệt file, upload/download |
| 🔍 **Tìm kiếm** | Tìm trong notebook |
| 📊 **Biến** | Xem biến đang có trong RAM |
| 🧩 **Đoạn mã** | Snippet mẫu (import data, ML, visualization) |

### 3.3 Thanh Trạng Thái (Góc Phải)

- **RAM:** Bấm để xem dung lượng đã dùng / tổng
- **Đĩa:** Dung lượng ổ cứng còn lại
- Hiển thị dạng `1.2 / 12.7 GB` — khi gần đầy, cần dọn dẹp

---

## 4. Cell — Đơn Vị Cơ Bản

### 4.1 Cell Code

Cell code chứa code Python. Mỗi cell chạy độc lập nhưng chia sẻ biến với nhau.

```python
# Cell 1: định nghĩa biến
x = 10
y = 20
```

```python
# Cell 2: dùng biến từ cell 1
print(x + y)  # Output: 30
```

**Cách chạy:**
- **Shift + Enter:** Chạy và nhảy xuống cell tiếp theo
- **Ctrl + Enter:** Chạy và ở nguyên cell đó
- **Alt + Enter:** Chạy và chèn cell mới bên dưới
- Bấm nút **▶** bên trái cell

**Trạng thái cell:**
- `[ ]`: Chưa chạy
- `[*]`: Đang chạy (bấm ■ để dừng)
- `[1]`, `[2]`: Đã chạy, số là thứ tự thực thi

> ⚠️ **Lưu ý quan trọng:** Thứ tự số `[n]` là thứ tự bạn bấm chạy, KHÔNG phải thứ tự từ trên xuống! Nếu chạy cell dưới trước cell trên, biến có thể chưa tồn tại → lỗi `NameError`.

### 4.2 Lệnh Shell (Bắt Đầu Bằng `!`)

Thêm `!` ở đầu dòng để chạy lệnh Linux/terminal:

```
!pwd                  # xem thư mục hiện tại
!ls -la               # liệt kê file
!cat file.txt         # xem nội dung file
!mkdir thu_muc_moi    # tạo thư mục
!rm file_cu.txt       # xóa file
!cp a.txt b.txt       # copy file
!mv a.txt b.txt       # đổi tên/di chuyển
```

**Dùng biến Python trong lệnh shell:**

```python
ten_file = "du_lieu.csv"
```

```
!ls -lh {ten_file}        # dùng {biến} để chèn giá trị
!wc -l {ten_file}
```

**Chạy nhiều lệnh:**

```
!cd /content && ls && echo "xong"
```

### 4.3 Cell Text (Markdown)

Cell text dùng để viết tài liệu, giải thích code. Hỗ trợ đầy đủ Markdown:

```markdown
# Tiêu đề cấp 1
## Tiêu đề cấp 2
### Tiêu đề cấp 3

**Chữ đậm** và *chữ nghiêng* và ***vừa đậm vừa nghiêng***
~~Gạch ngang~~

- Mục không số 1
- Mục không số 2
  - Mục con

1. Mục có số 1
2. Mục có số 2

[Link tới Google](https://google.com)

`code ngắn` trong câu

```python
# code dài nhiều dòng
def hello():
    print("Xin chào!")
```

> Trích dẫn

| Cột 1 | Cột 2 |
|-------|-------|
| A     | B     |

![Mô tả ảnh](https://example.com/anh.jpg)

---
```

**Cách dùng:** Bấm đúp vào cell text để sửa, **Shift+Enter** để render.

### 4.4 Thêm / Xóa / Di Chuyển Cell

- **Thêm:** Bấm **+ Code** / **+ Text** trên thanh công cụ, hoặc di chuột giữa 2 cell → hiện nút **+**
- **Xóa:** Chọn cell → **Chỉnh sửa** → **Xóa ô**, hoặc bấm `D` 2 lần (ở chế độ lệnh)
- **Di chuyển:** Kéo thả bằng chuột, hoặc `Ctrl+M K` (lên) / `Ctrl+M J` (xuống)
- **Nhân bản:** `Ctrl+M Shift+D`

### 4.5 Ẩn / Hiện Code và Output

Khi trình bày hoặc chia sẻ:
- **Chế độ xem** → **Ẩn mã**: Ẩn tất cả code, chỉ hiện kết quả
- Bấm `⋮` ở góc cell → **Ẩn ô**: Ẩn cell cụ thể
- Hữu ích khi làm báo cáo cho người không code

---

## 5. Runtime — Sức Mạnh Tính Toán

### 5.1 Runtime Là Gì?

Runtime là máy ảo trên cloud Google nơi code của bạn thực sự chạy. Mỗi notebook kết nối tới 1 runtime riêng.

**Thông số runtime miễn phí:**
- CPU: 2 core Intel Xeon
- RAM: ~12.7 GB
- Disk: ~107 GB
- GPU (nếu bật): NVIDIA Tesla T4, 16GB VRAM

### 5.2 Các Loại Runtime

#### CPU (Mặc định)
- Dùng cho code Python thông thường
- Xử lý dữ liệu nhỏ, học cú pháp
- Không tốn tài nguyên GPU quý giá

#### GPU (Graphics Processing Unit)
- **Khi nào dùng:** Train neural network, xử lý ảnh/video, tính toán ma trận lớn
- **Loại GPU miễn phí:** NVIDIA Tesla T4 (16GB)
- **Tốc độ:** Nhanh hơn CPU 10-50 lần cho deep learning

#### TPU (Tensor Processing Unit)
- Chip AI do Google thiết kế riêng
- **Khi nào dùng:** Train model rất lớn (BERT, Transformer)
- **Lưu ý:** Code phải viết riêng cho TPU (dùng `torch_xla` hoặc `tf.distribute`)

### 5.3 Cách Đổi Loại Runtime

1. **Thời gian chạy** → **Thay đổi loại thời gian chạy**
2. Chọn **Bộ tăng tốc phần cứng:** Không có / GPU / TPU
3. Bấm **Lưu**
4. Runtime sẽ khởi động lại (mất hết biến!)

### 5.4 Kiểm Tra GPU Có Hoạt Động Không

**Cách 1 — Lệnh shell:**
```
!nvidia-smi
```
Nếu hiện bảng thông tin GPU → có GPU. Nếu báo lỗi → đang dùng CPU.

**Cách 2 — PyTorch:**
```python
import torch
print("Có GPU:", torch.cuda.is_available())
if torch.cuda.is_available():
    print("Tên GPU:", torch.cuda.get_device_name(0))
    print("VRAM:", torch.cuda.get_device_properties(0).total_memory / 1e9, "GB")
```

**Cách 3 — TensorFlow:**
```python
import tensorflow as tf
print("GPU:", tf.config.list_physical_devices('GPU'))
```

### 5.5 Quản Lý Runtime

| Thao tác | Cách làm |
|----------|----------|
| Kết nối | Bấm nút **Kết nối** góc phải trên |
| Ngắt kết nối | Thời gian chạy → Ngắt kết nối và xóa thời gian chạy |
| Khởi động lại | Thời gian chạy → Khởi động lại thời gian chạy |
| Xem phiên đang chạy | Thời gian chạy → Quản lý phiên |

### 5.6 Vòng Đời Runtime

```
Kết nối → Chạy code → Không hoạt động 90 phút → Tự ngắt
                                    ↓
                          File /content bị xóa!
```

> ⚠️ **CỰC KỲ QUAN TRỌNG:**
> - Mọi file trong `/content` (trừ Drive) sẽ **bị xóa** khi runtime ngắt!
> - Biến trong RAM cũng mất hết
> - Muốn giữ file lâu dài → lưu vào Google Drive hoặc GitHub
> - Muốn giữ model đã train → download về hoặc push lên GitHub/Hugging Face

### 5.7 Giới Hạn Bản Miễn Phí

| Giới hạn | Chi tiết |
|----------|----------|
| Thời gian tối đa | ~12 giờ liên tục |
| Tự ngắt khi idle | ~90 phút không tương tác |
| GPU | Không đảm bảo luôn có (hết GPU phải đợi) |
| RAM | ~12.7 GB |
| Background run | Không hỗ trợ (tắt tab là có thể ngắt) |

---

## 6. Quản Lý File & Thư Mục

### 6.1 Panel File (Thanh Bên Trái)

Bấm biểu tượng 📁 để mở panel quản lý file:

- **Upload:** Kéo thả file từ máy vào, hoặc bấm biểu tượng upload
- **Download:** Chuột phải vào file → **Tải xuống**
- **Xóa:** Chuột phải → **Xóa**
- **Đổi tên:** Chuột phải → **Đổi tên**
- **Tạo thư mục:** Bấm biểu tượng thư mục có dấu +
- **Refresh:** Bấm biểu tượng ↻ nếu file mới chưa hiện

### 6.2 Upload File Bằng Code

**Cách 1 — Dialog chọn file:**
```python
from google.colab import files

uploaded = files.upload()
# Hiện dialog → chọn file từ máy → file nằm trong /content
```

**Cách 2 — Upload nhiều file:**
```python
from google.colab import files

for name, data in files.upload().items():
    print(f"Đã upload: {name} ({len(data)} bytes)")
```

### 6.3 Download File Bằng Code

```python
from google.colab import files

files.download('ket_qua.csv')      # tải 1 file
```

```python
# Nén nhiều file rồi tải
!zip ket_qua.zip ket_qua1.csv ket_qua2.csv model.pt
from google.colab import files
files.download('ket_qua.zip')
```

### 6.4 Kết Nối Google Drive

**Bước 1 — Mount Drive:**
```python
from google.colab import drive
drive.mount('/content/drive')
```
→ Bấm link xác thực → Copy mã → Dán vào ô → Enter

**Bước 2 — Truy cập file:**
```python
# File của bạn nằm ở:
# /content/drive/MyDrive/

!ls "/content/drive/MyDrive/"
```

**Bước 3 — Đọc/ghi file Drive:**
```python
import pandas as pd

# Đọc từ Drive
df = pd.read_csv('/content/drive/MyDrive/du_lieu.csv')

# Ghi vào Drive (không mất khi runtime ngắt!)
df.to_csv('/content/drive/MyDrive/ket_qua.csv', index=False)
```

**Ngắt kết nối Drive:**
```python
from google.colab import drive
drive.flush_and_unmount()
```

### 6.5 Clone GitHub Repository

**Repo public:**
```
!git clone https://github.com/username/ten-repo.git
!ls ten-repo/
```

**Repo private (dùng Personal Access Token):**
```python
import getpass

# Nhập PAT (không hiện trên màn hình)
TOKEN = getpass.getpass('Nhập GitHub PAT: ')
USERNAME = 'ten_github_cua_ban'
REPO = 'ten-repo-private'

!git clone https://{TOKEN}@github.com/{USERNAME}/{REPO}.git
print("Clone xong!")
```

> 🔒 **Bảo mật:** Không bao giờ ghi PAT trực tiếp vào code! Luôn dùng `getpass` để nhập.

**Push code từ Colab lên GitHub:**
```
%cd ten-repo
!git config user.email "email@example.com"
!git config user.name "Ten Cua Ban"
!git add .
!git commit -m "Update từ Colab"
!git push https://{TOKEN}@github.com/{USERNAME}/{REPO}.git main
```

### 6.6 Tải File Từ URL

```python
# File nhỏ
!wget https://example.com/file.zip

# File lớn (hiện progress bar)
!wget --progress=bar:force https://example.com/bigfile.zip

# Dùng curl
!curl -L -o ten_file.zip https://example.com/file.zip

# Tải với tên khác
!wget -O ten_moi.zip https://example.com/file.zip
```

### 6.7 Giải Nén File

```
!unzip file.zip                    # giải nén zip
!unzip file.zip -d thu_muc/        # giải nén vào thư mục
!tar -xzf file.tar.gz              # giải nén tar.gz
!tar -xzf file.tar.gz -C thu_muc/  # giải nén vào thư mục
!7z x file.7z                      # cần cài p7zip trước
```

---

## 7. Cài Đặt Package & Thư Viện

### 7.1 Cài Package Python (pip)

```
# Cài 1 package
!pip install numpy

# Cài nhiều package
!pip install pandas matplotlib seaborn

# Cài phiên bản cụ thể
!pip install torch==2.0.0

# Cài từ GitHub
!pip install git+https://github.com/username/repo.git

# Cài im lặng (không hiện log dài)
!pip install -q transformers datasets
```

> 💡 **Mẹo:** Thêm `-q` (quiet) để log ngắn gọn, dễ đọc trên điện thoại.

### 7.2 Cài Package Hệ Thống (apt)

```
# Cập nhật danh sách package
!apt-get update -qq

# Cài package
!apt-get install -y -qq ffmpeg        # xử lý video/audio
!apt-get install -y -qq tesseract-ocr # OCR đọc chữ trong ảnh
!apt-get install -y -qq poppler-utils # xử lý PDF
```

### 7.3 Kiểm Tra Package Đã Cài

```
!pip list | grep torch          # tìm package
!pip show numpy                 # xem chi tiết package
!python -c "import torch; print(torch.__version__)"
```

### 7.4 Package Có Sẵn (Không Cần Cài)

Colab đã cài sẵn rất nhiều thư viện phổ biến:

| Lĩnh vực | Package có sẵn |
|----------|----------------|
| Tính toán | numpy, scipy |
| Dữ liệu | pandas |
| Vẽ biểu đồ | matplotlib, seaborn, plotly |
| ML cơ bản | scikit-learn |
| Deep Learning | torch, tensorflow, keras |
| Xử lý ảnh | PIL, opencv-python |
| Jupyter | ipython, jupyter |

```python
# Kiểm tra tất cả package có sẵn
!pip list
```

### 7.5 Lưu Ý Khi Cài Package

1. **Cài lại sau mỗi lần restart runtime** — package không được lưu vĩnh viễn
2. Đặt cell `!pip install` ở **đầu notebook** để chạy 1 lần duy nhất
3. Một số package cần **restart runtime** sau khi cài → Colab sẽ báo nút "Khởi động lại"

---

## 8. Magic Commands

Magic commands là lệnh đặc biệt của IPython, bắt đầu bằng `%` hoặc `%%`.

### 8.1 Line Magic (1 Dòng — Dùng `%`)

| Lệnh | Chức năng | Ví dụ |
|------|-----------|-------|
| `%lsmagic` | Liệt kê tất cả magic | `%lsmagic` |
| `%time` | Đo thời gian chạy 1 dòng | `%time sum(range(10**7))` |
| `%timeit` | Đo nhiều lần, lấy trung bình | `%timeit sum(range(1000))` |
| `%matplotlib inline` | Hiện biểu đồ trong notebook | `%matplotlib inline` |
| `%cd` | Đổi thư mục | `%cd /content/drive` |
| `%pwd` | Xem thư mục hiện tại | `%pwd` |
| `%ls` | Liệt kê file | `%ls` |
| `%env` | Xem/set biến môi trường | `%env MY_VAR=hello` |
| `%who` | Liệt kê biến đã định nghĩa | `%who` |
| `%whos` | Liệt kê biến + kiểu + giá trị | `%whos` |
| `%reset` | Xóa hết biến | `%reset -f` |
| `%history` | Xem lịch sử lệnh | `%history -n 1-10` |

### 8.2 Cell Magic (Cả Cell — Dùng `%%`)

| Lệnh | Chức năng |
|------|-----------|
| `%%time` | Đo thời gian chạy cả cell |
| `%%timeit` | Đo nhiều lần cả cell |
| `%%bash` | Chạy cả cell như bash script |
| `%%writefile` | Ghi nội dung cell thành file |
| `%%html` | Render cell như HTML |
| `%%javascript` | Chạy JavaScript |

**Ví dụ chi tiết:**

```python
# Đo thời gian
%%time
import time
time.sleep(2)
print("Xong!")
```

```python
# Ghi file Python
%%writefile train.py
import torch
print("Training...")

# Sau đó chạy file
!python train.py
```

```bash
%%bash
echo "Hello từ bash"
ls -la
pwd
```

```html
%%html
<h1 style="color: blue;">Tiêu đề màu xanh</h1>
<p>Đoạn văn bản</p>
```

### 8.3 Biến Môi Trường

```python
# Set biến môi trường
%env API_KEY=sk-abc123
%env MODEL_NAME=gpt-4
```

```
# Dùng trong shell
!echo $API_KEY
!echo "Model: $MODEL_NAME"
```

```python
# Dùng trong Python
import os
print(os.environ['API_KEY'])
```

---

## 9. Phím Tắt

### 9.1 Hai Chế Độ

Colab có 2 chế độ, bấm **Esc** để chuyển qua lại:

- **Chế độ lệnh** (viền xám): Thao tác với cell (thêm, xóa, di chuyển)
- **Chế độ chỉnh sửa** (viền xanh): Gõ code trong cell

### 9.2 Chế Độ Lệnh (Bấm Esc Trước)

#### Thêm / Xóa Cell

| Phím | Chức năng |
|------|-----------|
| `A` | Thêm cell code **phía trên** |
| `B` | Thêm cell code **phía dưới** |
| `D` `D` | Xóa cell (bấm D 2 lần liên tiếp) |
| `Z` | Khôi phục cell vừa xóa |

#### Đổi Loại Cell

| Phím | Chức năng |
|------|-----------|
| `M` | Đổi thành cell **Text** (Markdown) |
| `Y` | Đổi thành cell **Code** |

#### Di Chuyển & Sắp Xếp

| Phím | Chức năng |
|------|-----------|
| `K` / `↑` | Chọn cell phía trên |
| `J` / `↓` | Chọn cell phía dưới |
| `Shift + K` | Mở rộng chọn lên trên |
| `Shift + J` | Mở rộng chọn xuống dưới |
| `Shift + M` | Gộp cell đang chọn với cell dưới |

#### Copy / Paste Cell

| Phím | Chức năng |
|------|-----------|
| `C` | Copy cell |
| `X` | Cắt cell |
| `V` | Dán cell phía dưới |
| `Shift + V` | Dán cell phía trên |

### 9.3 Chế Độ Chỉnh Sửa (Đang Gõ Code)

| Phím | Chức năng |
|------|-----------|
| `Tab` | Gợi ý / tự động hoàn thành code |
| `Shift + Tab` | Xem tài liệu hàm (tooltip) |
| `Ctrl + ]` | Thụt đầu dòng |
| `Ctrl + [` | Bỏ thụt đầu dòng |
| `Ctrl + /` | Comment / bỏ comment dòng |
| `Ctrl + D` | Xóa cả dòng |
| `Esc` | Thoát về chế độ lệnh |

### 9.4 Chạy Code

| Phím | Chức năng |
|------|-----------|
| `Shift + Enter` | Chạy cell → nhảy xuống cell tiếp theo |
| `Ctrl + Enter` | Chạy cell → ở nguyên cell đó |
| `Alt + Enter` | Chạy cell → chèn cell mới bên dưới |
| `Ctrl + M` `I` | Ngắt cell đang chạy |

### 9.5 Khác

| Phím | Chức năng |
|------|-----------|
| `Ctrl + M` `H` | Mở bảng phím tắt đầy đủ |
| `Ctrl + Space` | Gợi ý code |
| `Ctrl + S` | Lưu notebook |
| `Ctrl + M` `P` | Mở bảng lệnh (command palette) |

> 💡 **Tùy chỉnh:** Vào **Công cụ** → **Phím tắt** để xem và sửa theo ý thích.

---

## 10. Vẽ Biểu Đồ & Hiển Thị Dữ Liệu

### 10.1 Matplotlib (Cơ Bản Nhất)

```python
import matplotlib.pyplot as plt

# Biểu đồ đường
x = [1, 2, 3, 4, 5]
y = [1, 4, 9, 16, 25]
plt.plot(x, y)
plt.title("Biểu đồ đường")
plt.xlabel("Trục X")
plt.ylabel("Trục Y")
plt.show()
```

```python
# Biểu đồ cột
ten = ['A', 'B', 'C', 'D']
diem = [8, 9, 7, 10]
plt.bar(ten, diem, color='skyblue')
plt.title("Điểm số")
plt.show()
```

```python
# Biểu đồ tròn
plt.pie([30, 25, 20, 15, 10],
        labels=['A', 'B', 'C', 'D', 'E'],
        autopct='%1.1f%%')
plt.title("Tỷ lệ")
plt.show()
```

```python
# Nhiều biểu đồ con
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

axes[0].plot([1, 2, 3], [1, 4, 9])
axes[0].set_title("Biểu đồ 1")

axes[1].bar(['A', 'B'], [5, 8])
axes[1].set_title("Biểu đồ 2")

plt.tight_layout()
plt.show()
```

### 10.2 Seaborn (Đẹp Hơn)

```
!pip install -q seaborn
```

```python
import seaborn as sns
import matplotlib.pyplot as plt

# Heatmap (ma trận tương quan)
import numpy as np
data = np.random.rand(5, 5)
sns.heatmap(data, annot=True, cmap='coolwarm')
plt.title("Heatmap")
plt.show()
```

```python
# Biểu đồ phân phối
sns.histplot(data=[1, 2, 2, 3, 3, 3, 4, 4, 5], kde=True)
plt.title("Phân phối điểm")
plt.show()
```

### 10.3 Hiển Thị Ảnh

```python
# Cách 1: PIL
from PIL import Image
img = Image.open('anh.jpg')
img  # tự động hiển thị
```

```python
# Cách 2: IPython display (chỉnh kích thước)
from IPython.display import Image as IPImage
IPImage('anh.jpg', width=400)
```

```python
# Cách 3: Matplotlib (kèm tiêu đề)
import matplotlib.pyplot as plt
import matplotlib.image as mpimg

img = mpimg.imread('anh.jpg')
plt.imshow(img)
plt.axis('off')
plt.title("Ảnh mẫu")
plt.show()
```

### 10.4 Hiển Thị Video & Audio

```python
from IPython.display import Video
Video('video.mp4', width=600)
```

```python
from IPython.display import Audio
Audio('nhac.mp3')
```

```python
# Nhúng YouTube
from IPython.display import YouTubeVideo
YouTubeVideo('dQw4w9WgXcQ')
```

### 10.5 Hiển Thị HTML Tùy Chỉnh

```python
from IPython.display import HTML

HTML('''
<div style="background: #f0f0f0; padding: 20px; border-radius: 10px;">
    <h2 style="color: #333;">Thông báo</h2>
    <p>Train xong với <b>accuracy 95%</b>!</p>
</div>
''')
```

### 10.6 Progress Bar

```python
from tqdm import tqdm
import time

# Vòng lặp có progress bar
for i in tqdm(range(100), desc="Đang xử lý"):
    time.sleep(0.02)
```

```python
# Với pandas
import pandas as pd
tqdm.pandas()

df = pd.DataFrame({'a': range(1000)})
df['b'] = df['a'].progress_apply(lambda x: x * 2)
```

### 10.7 Hiển Thị DataFrame Đẹp

```python
import pandas as pd

df = pd.DataFrame({
    'Tên': ['An', 'Bình', 'Chi'],
    'Điểm': [9, 8, 10],
    'Lớp': ['A1', 'A2', 'A1']
})

df  # tự động hiển thị bảng đẹp
```

```python
# Tô màu theo điều kiện
df.style.background_gradient(cmap='Greens')
df.style.highlight_max(color='yellow')
```

---

## 11. Form Tương Tác & Widget

### 11.1 Form Đơn Giản Với `@param`

Thêm comment đặc biệt `# @param` để biến thành form nhập liệu:

```python
# @title ⚙️ Cài đặt Model

# Dropdown chọn
model_name = "resnet50"  # @param ["resnet50", "vgg16", "efficientnet_b0", "vit_base"]

# Checkbox
use_gpu = True  # @param {type:"boolean"}

# Slider
learning_rate = 0.001  # @param {type:"slider", min:0.0001, max:0.1, step:0.0001}

# Ô nhập số
batch_size = 32  # @param {type:"integer", min:8, max:256}

# Ô nhập text
dataset_path = "/content/data"  # @param {type:"string"}

print(f"Model: {model_name}")
print(f"GPU: {use_gpu}")
print(f"LR: {learning_rate}")
print(f"Batch: {batch_size}")
print(f"Data: {dataset_path}")
```

### 11.2 Ẩn Code, Chỉ Hiện Form

Thêm `# @title` ở dòng đầu để thu gọn cell thành form:

```python
# @title 🚀 Train Model (bấm để mở rộng)

epochs = 10  # @param {type:"slider", min:1, max:100}
# ... code train ở đây ...
print(f"Train {epochs} epochs xong!")
```

### 11.3 Input Từ Bàn Phím

```python
ten = input("Nhập tên của bạn: ")
tuoi = int(input("Nhập tuổi: "))
print(f"Chào {ten}, bạn {tuoi} tuổi!")
```

### 11.4 Mật Khẩu (Không Hiện Ký Tự)

```python
import getpass

password = getpass.getpass("Nhập mật khẩu: ")
print("Đã nhập xong (không hiện mật khẩu)")
```

Dùng cho API key, PAT GitHub:
```python
import getpass
import os

# Nhập API key an toàn
api_key = getpass.getpass("Nhập OpenAI API key: ")
os.environ["OPENAI_API_KEY"] = api_key
print("✅ Đã lưu API key vào biến môi trường")
```

---

## 12. Làm Việc Với Dữ Liệu

### 12.1 Đọc File CSV

```python
import pandas as pd

# Từ file local
df = pd.read_csv('du_lieu.csv')
print(df.head())       # 5 dòng đầu
print(df.info())       # thông tin cột
print(df.describe())   # thống kê
```

```python
# Từ URL
url = "https://raw.githubusercontent.com/.../data.csv"
df = pd.read_csv(url)
```

```python
# Từ Google Drive
df = pd.read_csv('/content/drive/MyDrive/du_lieu.csv')
```

### 12.2 Đọc File Excel

```
!pip install -q openpyxl
```

```python
df = pd.read_excel('bao_cao.xlsx', sheet_name='Sheet1')
print(df.head())
```

### 12.3 Đọc File JSON

```python
import json

with open('config.json') as f:
    config = json.load(f)
print(config)
```

```python
# JSON thành DataFrame
df = pd.read_json('du_lieu.json')
```

### 12.4 Lưu Dữ Liệu

```python
# Lưu CSV
df.to_csv('ket_qua.csv', index=False)

# Lưu Excel
df.to_excel('bao_cao.xlsx', index=False)

# Lưu JSON
df.to_json('ket_qua.json', orient='records', indent=2)

# Lưu numpy array
import numpy as np
np.save('mang.npy', mang_numpy)
mang = np.load('mang.npy')
```

### 12.5 Làm Việc Với Ảnh

```python
from PIL import Image
import numpy as np

# Mở ảnh
img = Image.open('anh.jpg')
print(f"Kích thước: {img.size}, Mode: {img.mode}")

# Resize
img_nho = img.resize((224, 224))
img_nho.save('anh_nho.jpg')

# Chuyển thành numpy array (để đưa vào model AI)
arr = np.array(img)
print(f"Shape: {arr.shape}")  # (cao, rộng, kênh màu)
```

### 12.6 Tải Dataset Mẫu

```python
# Dùng seaborn (có sẵn nhiều dataset)
import seaborn as sns
df = sns.load_dataset('titanic')
print(df.head())
```

```python
# Dùng sklearn
from sklearn.datasets import load_iris
iris = load_iris()
print(iris.data[:5])
```

---

## 13. Chia Sẻ & Cộng Tác

### 13.1 Chia Sẻ Như Google Docs

1. Bấm nút **Chia sẻ** (góc phải trên)
2. Chọn quyền:
   - **Người xem:** Chỉ đọc, không chạy code
   - **Người nhận xét:** Thêm comment
   - **Người chỉnh sửa:** Sửa và chạy code
3. Copy link → gửi cho người khác

> 💡 Người được chia sẻ có thể chạy code trên runtime riêng của họ.

### 13.2 Lưu Lên GitHub

**Cách 1 — Qua menu:**
1. **Tệp** → **Lưu bản sao trong GitHub**
2. Chọn repository, nhánh
3. Nhập commit message → **OK**

**Cách 2 — Bằng lệnh git:**
```python
import getpass
TOKEN = getpass.getpass("GitHub PAT: ")

!git clone https://{TOKEN}@github.com/username/repo.git
%cd repo
!cp /content/notebook.ipynb .
!git add notebook.ipynb
!git commit -m "Update notebook"
!git push https://{TOKEN}@github.com/username/repo.git main
```

### 13.3 Xuất File

| Định dạng | Cách xuất |
|-----------|-----------|
| `.ipynb` | Tệp → Tải xuống → .ipynb |
| `.py` | Tệp → Tải xuống → .py |
| PDF | Tệp → In → Lưu thành PDF |
| HTML | `!jupyter nbconvert --to html notebook.ipynb` |

### 13.4 Nhúng Vào Website/Blog

Sau khi lưu lên GitHub, dùng:
- **nbviewer:** https://nbviewer.org → dán link GitHub
- **Google Colab badge:** Thêm vào README:

```markdown
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/username/repo/blob/main/notebook.ipynb)
```

---

## 14. Mẹo & Thủ Thuật Nâng Cao

### 14.1 Chạy Lệnh Nền (Background)

```python
import subprocess
import time

# Chạy script nền
proc = subprocess.Popen(
    ['python', 'train.py'],
    stdout=open('train.log', 'w'),
    stderr=subprocess.STDOUT
)
print(f"PID: {proc.pid}")

# Kiểm tra còn chạy không
print("Đang chạy:", proc.poll() is None)

# Xem log
!tail -20 train.log

# Dừng process
# proc.terminate()
```

### 14.2 TensorBoard

```
%load_ext tensorboard
```

```python
# Sau khi train với logging
%tensorboard --logdir logs/
```

### 14.3 SSH Vào Colab (Nâng Cao)

```
# Cài colab-ssh
!pip install -q colab-ssh

from colab_ssh import launch_ssh_cloudflared
launch_ssh_cloudflared(password="matkhau123")
```

→ Cho phép SSH từ máy local vào Colab runtime.

### 14.4 Chạy Web Server

```python
# Chạy Flask app
!pip install -q flask

%%writefile app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return '<h1>Hello từ Colab!</h1>'

app.run(port=5000)
```

```
# Expose ra internet bằng ngrok
!pip install -q pyngrok
from pyngrok import ngrok
ngrok.set_auth_token("YOUR_NGROK_TOKEN")
url = ngrok.connect(5000)
print(url)
```

### 14.5 Dùng Nhiều GPU

```python
import torch

# Kiểm tra số GPU
print(f"Số GPU: {torch.cuda.device_count()}")

# Dùng DataParallel cho multi-GPU
model = torch.nn.DataParallel(model)
```

### 14.6 Tiết Kiệm RAM

```python
# Xóa biến không dùng
del bien_lon
import gc
gc.collect()

# PyTorch: dọn VRAM
import torch
torch.cuda.empty_cache()

# Kiểm tra RAM còn lại
!free -h
```

### 14.7 Tải Model Lớn (Hugging Face)

```python
# Đăng nhập Hugging Face
from huggingface_hub import login
import getpass
login(token=getpass.getpass("HF Token: "))
```

```python
# Tải model
from transformers import AutoModel, AutoTokenizer

model_name = "bert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModel.from_pretrained(model_name)
print("Tải xong!")
```

### 14.8 Cache Để Không Tải Lại

```python
import os

CACHE_DIR = "/content/drive/MyDrive/cache"
os.makedirs(CACHE_DIR, exist_ok=True)

# Tải model vào Drive (lần sau không cần tải lại)
from transformers import AutoModel
model = AutoModel.from_pretrained(
    "bert-base-uncased",
    cache_dir=CACHE_DIR
)
```

### 14.9 Chụp Màn Hình Output

```python
# Lưu biểu đồ thành file
import matplotlib.pyplot as plt
plt.plot([1, 2, 3], [1, 4, 9])
plt.savefig('/content/bieu_do.png', dpi=150, bbox_inches='tight')
plt.show()

# Tải về máy
from google.colab import files
files.download('/content/bieu_do.png')
```

---

## 15. Xử Lý Lỗi Thường Gặp

### 15.1 Lỗi Python

| Lỗi | Nguyên nhân | Cách khắc phục |
|-----|-------------|----------------|
| `ModuleNotFoundError: No module named 'X'` | Chưa cài package | `!pip install X` |
| `NameError: name 'x' is not defined` | Chưa chạy cell định nghĩa biến | Chạy cell phía trên trước |
| `TypeError` | Sai kiểu dữ liệu | Kiểm tra `type(bien)` |
| `IndexError: list index out of range` | Truy cập index không tồn tại | Kiểm tra `len(list)` |
| `KeyError: 'ten_cot'` | Sai tên cột trong DataFrame | Kiểm tra `df.columns` |
| `FileNotFoundError` | Sai đường dẫn file | Kiểm tra bằng `!ls` |

### 15.2 Lỗi GPU / CUDA

| Lỗi | Cách khắc phục |
|-----|----------------|
| `CUDA out of memory` | Giảm `batch_size`, dùng `torch.cuda.empty_cache()`, giảm kích thước model |
| `No CUDA GPUs are available` | Chưa bật GPU → Đổi runtime sang GPU |
| `CUDA error: device-side assert` | Thường do label sai trong classification → kiểm tra label |

```python
# Giảm batch size khi hết VRAM
batch_size = 16  # thử giảm từ 32 xuống 16, 8, 4...

# Dọn VRAM
import torch
torch.cuda.empty_cache()
import gc
gc.collect()
```

### 15.3 Lỗi Runtime

| Vấn đề | Cách khắc phục |
|--------|----------------|
| Runtime tự ngắt | Kết nối lại; code phải chạy lại từ đầu |
| `Session crashed` (hết RAM) | Giảm kích thước data, dùng `del` + `gc.collect()` |
| Không có GPU available | Đợi 1 lúc rồi thử lại, hoặc dùng lúc ít người (sáng sớm) |
| Chạy quá 12h bị ngắt | Chia nhỏ task, lưu checkpoint vào Drive |

### 15.4 Lỗi Kết Nối Drive

```python
# Nếu mount Drive bị lỗi, thử:
from google.colab import drive
drive.flush_and_unmount()  # ngắt trước
drive.mount('/content/drive', force_remount=True)  # mount lại
```

### 15.5 Lỗi Cài Package

```
# Nếu pip install bị lỗi version conflict:
!pip install --upgrade pip
!pip install package --no-deps    # bỏ qua dependencies

# Nếu cần restart sau khi cài:
# Colab sẽ hiện nút "KHỞI ĐỘNG LẠI" → bấm vào
```

---

## 16. Colab Pro & So Sánh Gói

### 16.1 Bảng So Sánh

| Tính năng | Miễn phí | Pro ($10/tháng) | Pro+ ($50/tháng) |
|-----------|----------|-----------------|-------------------|
| GPU | T4 (khi có sẵn) | T4/V100 (ưu tiên) | A100/V100 (ưu tiên cao) |
| RAM tối đa | ~12.7 GB | ~25 GB | ~52 GB |
| Thời gian chạy | ~12 giờ | 24 giờ | 24 giờ |
| Chạy nền | ❌ | ✅ | ✅ |
| Terminal | ❌ | ✅ | ✅ |
| Ưu tiên GPU | Thấp | Cao | Rất cao |

### 16.2 Có Nên Mua Pro?

**Nên mua nếu:**
- Train model thường xuyên, cần GPU ổn định
- Cần RAM lớn (>12GB)
- Muốn chạy code khi tắt máy (background)

**Không cần nếu:**
- Chỉ học Python cơ bản
- Thỉnh thoảng mới dùng GPU
- Có thể chờ khi hết GPU

---

## 17. Bảo Mật & Lưu Ý

### 17.1 Không Bao Giờ Làm

- ❌ **Ghi API key / password trực tiếp vào code**
- ❌ **Chia sẻ notebook chứa thông tin nhạy cảm** ở chế độ công khai
- ❌ **Chạy code không rõ nguồn gốc** (có thể chứa mã độc)
- ❌ **Để PAT/token trong cell** rồi chia sẻ notebook

### 17.2 Nên Làm

- ✅ Dùng `getpass.getpass()` để nhập thông tin nhạy cảm
- ✅ Dùng biến môi trường (`os.environ`) cho API key
- ✅ Kiểm tra code trước khi chạy notebook của người khác
- ✅ Xóa output chứa thông tin nhạy cảm trước khi chia sẻ
- ✅ Thu hồi token sau khi dùng xong (nếu không cần nữa)

### 17.3 Ví Dụ Đúng

```python
# ✅ ĐÚNG: nhập ẩn
import getpass
api_key = getpass.getpass("Nhập API key: ")
```

```python
# ❌ SAI: ghi trực tiếp
api_key = "sk-abc123xyz"  # ai cũng thấy được!
```

### 17.4 Xóa Thông Tin Nhạy Cảm Trước Khi Chia Sẻ

1. **Chỉnh sửa** → **Xóa mọi output** (xóa kết quả chạy)
2. Kiểm tra không còn API key trong code
3. Mới bấm **Chia sẻ**

---

## 18. Tài Liệu Tham Khảo

### 18.1 Tài Liệu Chính Thức

- 🏠 Trang chủ Colab: https://colab.research.google.com
- ❓ FAQ: https://research.google.com/colaboratory/faq.html
- 📓 Notebook mẫu: Mở Colab → **Tệp** → **Mở notebook** → Tab **Ví dụ**

### 18.2 Học Thêm

- 📚 Stack Overflow: https://stackoverflow.com/questions/tagged/google-colaboratory
- 🎥 YouTube: Tìm "Google Colab tutorial"
- 📖 Jupyter docs: https://jupyter.org/documentation

### 18.3 Cộng Đồng Việt Nam

- Group Facebook: Tìm "Python Việt Nam", "AI Việt Nam"
- Diễn đàn: VOZ, Tinhte (box lập trình)

---

## Phụ Lục: Cheat Sheet Nhanh

```
# Chạy lệnh shell          !ls -la
# Cài package              !pip install -q ten_package
# Kiểm tra GPU             !nvidia-smi
# Upload file              from google.colab import files; files.upload()
# Download file            from google.colab import files; files.download('f.txt')
# Mount Drive              from google.colab import drive; drive.mount('/content/drive')
# Nhập mật khẩu ẩn         import getpass; getpass.getpass()
# Đo thời gian             %time code_1_dong  /  %%time (cả cell)
# Chạy cell                Shift+Enter
# Dừng cell                Ctrl+M I
# Thêm cell trên/dưới      A / B (chế độ lệnh)
# Xóa cell                 D D (chế độ lệnh)
```

---

*📝 Viết bởi **Capybara** 🦫 cho **Tiểu Dươngg***
*📅 Cập nhật lần cuối: 2026-10-10*
*💡 Thấy thiếu gì thì bảo tớ bổ sung nhé!*
