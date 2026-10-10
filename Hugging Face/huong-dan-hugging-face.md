# Hướng Dẫn Toàn Diện Hugging Face 🤗

> Tài liệu tiếng Việt chi tiết về Hugging Face — từ tạo tài khoản đến fine-tune model AI.
> Tác giả: Capybara 🦫 cho Tiểu Dươngg

---

## Mục Lục

1. [Hugging Face Là Gì?](#1-hugging-face-là-gì)
2. [Tạo Tài Khoản & Token](#2-tạo-tài-khoản--token)
3. [Cài Đặt Thư Viện](#3-cài-đặt-thư-viện)
4. [Transformers — Thư Viện Cốt Lõi](#4-transformers--thư-viện-cốt-lõi)
5. [Pipeline — Cách Dễ Nhất Dùng Model](#5-pipeline--cách-dễ-nhất-dùng-model)
6. [AutoModel & AutoTokenizer](#6-automodel--autotokenizer)
7. [Datasets — Thư Viện Dữ Liệu](#7-datasets--thư-viện-dữ-liệu)
8. [huggingface_hub — Upload/Download](#8-huggingface_hub--uploaddownload)
9. [Spaces — Deploy Demo Miễn Phí](#9-spaces--deploy-demo-miễn-phí)
10. [Inference API — Gọi Model Qua HTTP](#10-inference-api--gọi-model-qua-http)
11. [Fine-Tuning Cơ Bản](#11-fine-tuning-cơ-bản)
12. [Model Cards — Tài Liệu Model](#12-model-cards--tài-liệu-model)
13. [Xử Lý Lỗi Thường Gặp](#13-xử-lý-lỗi-thường-gặp)
14. [Best Practices](#14-best-practices)
15. [Tài Liệu Tham Khảo](#15-tài-liệu-tham-khảo)

---

## 1. Hugging Face Là Gì?

### 1.1 Định Nghĩa

**Hugging Face** (biểu tượng 🤗) là nền tảng lớn nhất thế giới cho AI/ML mã nguồn mở — được gọi là **"GitHub của AI"**. Nơi đây lưu trữ:

- **500,000+ model** AI (text, ảnh, âm thanh, video)
- **100,000+ dataset** (dữ liệu train)
- **Spaces** — deploy demo AI miễn phí

### 1.2 Các Model Nổi Tiếng Trên HF

| Model | Tác giả | Dùng để làm gì |
|-------|---------|----------------|
| BERT | Google | Hiểu văn bản, phân loại |
| GPT-2 | OpenAI | Sinh văn bản |
| LLaMA 2/3 | Meta | Chatbot, trợ lý AI |
| Mistral | Mistral AI | LLM hiệu quả cao |
| Stable Diffusion | Stability AI | Vẽ ảnh từ text |
| Whisper | OpenAI | Chuyển giọng nói → text |
| CLIP | OpenAI | Hiểu ảnh + text |
| ViT | Google | Phân loại ảnh |

### 1.3 Hệ Sinh Thái

```
┌─────────────────────────────────────────┐
│           huggingface.co (website)       │
├──────────────┬──────────────┬───────────┤
│   Models     │   Datasets   │  Spaces   │
│  (500K+)     │   (100K+)    │ (demo AI) │
└──────────────┴──────────────┴───────────┘
         ↓              ↓            ↓
┌─────────────────────────────────────────┐
│  Thư viện Python                        │
│  • transformers (dùng model)            │
│  • datasets (tải dữ liệu)              │
│  • huggingface_hub (upload/download)    │
│  • diffusers (sinh ảnh)                 │
│  • tokenizers (xử lý text)              │
└─────────────────────────────────────────┘
```

---

## 2. Tạo Tài Khoản & Token

### 2.1 Đăng Ký Tài Khoản

1. Vào https://huggingface.co/join
2. Điền username, email, password
3. Xác nhận email
4. Xong! Bạn có profile tại `huggingface.co/username`

### 2.2 Tạo Access Token

Token dùng để upload model, gọi API riêng tư.

1. Vào **Settings** → **Access Tokens** (hoặc https://huggingface.co/settings/tokens)
2. Bấm **New token**
3. Đặt tên (ví dụ: `colab-token`)
4. Chọn quyền:
   - **Read:** Chỉ đọc/tải model (an toàn nhất)
   - **Write:** Upload model, tạo repo
5. Bấm **Generate** → **Copy token ngay** (chỉ hiện 1 lần!)

> ⚠️ **Bảo mật:** Token như mật khẩu — không chia sẻ, không ghi vào code public!

### 2.3 Đăng Nhập Bằng CLI

```bash
# Cài đặt
pip install -U huggingface_hub

# Đăng nhập (nhập token khi được hỏi)
huggingface-cli login
```

### 2.4 Đăng Nhập Bằng Python

```python
from huggingface_hub import login
import getpass

# Nhập token an toàn (không hiện ký tự)
token = getpass.getpass("Nhập HF token: ")
login(token=token)
print("✅ Đăng nhập thành công!")
```

### 2.5 Đăng Nhập Trên Google Colab

```python
# Cách 1: dùng login()
from huggingface_hub import login
import getpass
login(token=getpass.getpass("HF Token: "))

# Cách 2: dùng biến môi trường
import os
os.environ["HF_TOKEN"] = getpass.getpass("HF Token: ")
```

### 2.6 Đăng Xuất

```bash
huggingface-cli logout
```

```python
from huggingface_hub import logout
logout()
```

---

## 3. Cài Đặt Thư Viện

### 3.1 Cài Đặt Cơ Bản

```bash
# Thư viện chính (bắt buộc)
pip install transformers

# Tải dataset
pip install datasets

# Upload/download model
pip install huggingface_hub

# Sinh ảnh (Stable Diffusion)
pip install diffusers

# Tăng tốc (khuyến nghị)
pip install accelerate
```

### 3.2 Cài Đầy Đủ Cho Từng Nhu Cầu

```bash
# NLP (xử lý ngôn ngữ)
pip install transformers datasets tokenizers

# Computer Vision
pip install transformers datasets Pillow torchvision

# Audio
pip install transformers datasets librosa soundfile

# Sinh ảnh
pip install diffusers transformers accelerate safetensors

# Train nhanh trên GPU
pip install accelerate bitsandbytes
```

### 3.3 Kiểm Tra Cài Đặt

```python
import transformers
import datasets
import huggingface_hub

print(f"transformers: {transformers.__version__}")
print(f"datasets: {datasets.__version__}")
print(f"huggingface_hub: {huggingface_hub.__version__}")

# Kiểm tra GPU
import torch
print(f"PyTorch: {torch.__version__}")
print(f"CUDA: {torch.cuda.is_available()}")
```

---

## 4. Transformers — Thư Viện Cốt Lõi

### 4.1 Transformers Là Gì?

Thư viện Python để **tải và chạy** các model AI từ Hugging Face Hub. Hỗ trợ:
- **PyTorch**, TensorFlow, JAX
- 500K+ model có sẵn
- API thống nhất cho mọi loại model

### 4.2 Kiến Trúc Chung

Mọi model trên HF gồm 2 phần:

```
Model = Tokenizer + Weights
         ↓              ↓
   Chuyển text     Tham số neural
   thành số       network đã train
```

```python
# Luồng xử lý cơ bản:
# Text → Tokenizer → Số (input_ids) → Model → Kết quả → Giải mã → Text
```

### 4.3 Tìm Model Trên Hub

1. Vào https://huggingface.co/models
2. Lọc theo:
   - **Task:** text-classification, text-generation, image-classification...
   - **Library:** PyTorch, TensorFlow
   - **Language:** Vietnamese, English...
   - **License:** apache-2.0, mit, openrail...
3. Xem số lượt tải (downloads) và like để đánh giá chất lượng

**Model tiếng Việt phổ biến:**

| Model | Task | Link |
|-------|------|------|
| `vinai/phobert-base` | Hiểu tiếng Việt | huggingface.co/vinai/phobert-base |
| `bkai-foundation-models/vietnamese-bi-encoder` | Embedding TV | huggingface.co/bkai-foundation-models/... |
| `VietAI/vit5-base` | Sinh văn bản TV | huggingface.co/VietAI/vit5-base |

---

## 5. Pipeline — Cách Dễ Nhất Dùng Model

### 5.1 Pipeline Là Gì?

`pipeline()` là hàm "tất cả trong một" — tự động tải tokenizer + model, xử lý input/output. **Chỉ cần 2 dòng code!**

### 5.2 Phân Tích Cảm Xúc (Sentiment Analysis)

```python
from transformers import pipeline

# Tạo pipeline (lần đầu sẽ tải model ~500MB)
classifier = pipeline("sentiment-analysis")

# Dùng ngay!
result = classifier("Tôi rất thích Hugging Face!")
print(result)
# [{'label': 'POSITIVE', 'score': 0.9998}]

# Nhiều câu cùng lúc
results = classifier([
    "Hôm nay trời đẹp quá!",
    "Tôi buồn quá.",
    "Món ăn này bình thường."
])
for r in results:
    print(f"{r['label']}: {r['score']:.4f}")
```

### 5.3 Các Pipeline Phổ Biến

```python
from transformers import pipeline

# 1. Sinh văn bản
generator = pipeline("text-generation", model="gpt2")
print(generator("Hôm nay tôi đi", max_length=30))

# 2. Dịch thuật
translator = pipeline("translation_en_to_vi",
                      model="Helsinki-NLP/opus-mt-en-vi")
print(translator("Hello, how are you?"))

# 3. Tóm tắt văn bản
summarizer = pipeline("summarization")
text = "Đoạn văn dài..." * 10
print(summarizer(text, max_length=50))

# 4. Trả lời câu hỏi
qa = pipeline("question-answering")
context = "Hugging Face được thành lập năm 2016 tại New York."
print(qa(question="HF thành lập năm nào?", context=context))

# 5. Nhận diện thực thể (NER)
ner = pipeline("ner", aggregation_strategy="simple")
print(ner("Tôi sống ở Hà Nội, Việt Nam."))

# 6. Phân loại ảnh
img_classifier = pipeline("image-classification")
print(img_classifier("anh_meo.jpg"))

# 7. Chuyển giọng nói thành text
asr = pipeline("automatic-speech-recognition",
               model="openai/whisper-small")
print(asr("audio.mp3"))

# 8. Sinh ảnh từ text
img_gen = pipeline("text-to-image",
                   model="stabilityai/stable-diffusion-2-1")
image = img_gen("Một chú mèo đang ngủ trên ghế sofa")
image[0].save("meo.png")
```

### 5.4 Bảng Pipeline Đầy Đủ

| Tên pipeline | Task | Input | Output |
|--------------|------|-------|--------|
| `sentiment-analysis` | Phân tích cảm xúc | Text | POSITIVE/NEGATIVE + score |
| `text-generation` | Sinh văn bản | Prompt | Văn bản tiếp theo |
| `text-classification` | Phân loại text | Text | Label + score |
| `token-classification` | NER | Text | Entities |
| `question-answering` | Hỏi đáp | Question + context | Answer |
| `summarization` | Tóm tắt | Văn bản dài | Văn bản ngắn |
| `translation` | Dịch | Text | Text ngôn ngữ khác |
| `image-classification` | Phân loại ảnh | Ảnh | Label + score |
| `object-detection` | Phát hiện vật thể | Ảnh | Boxes + labels |
| `image-segmentation` | Phân vùng ảnh | Ảnh | Masks |
| `automatic-speech-recognition` | Speech-to-text | Audio | Text |
| `text-to-speech` | Text-to-speech | Text | Audio |
| `text-to-image` | Vẽ ảnh | Prompt | Ảnh |
| `zero-shot-classification` | Phân loại không cần train | Text + labels | Label phù hợp nhất |

### 5.5 Zero-Shot Classification (Không Cần Train!)

```python
from transformers import pipeline

classifier = pipeline("zero-shot-classification",
                      model="facebook/bart-large-mnli")

text = "Hôm nay tôi ăn phở rất ngon"
labels = ["ẩm thực", "thể thao", "công nghệ"]

result = classifier(text, labels)
print(result['labels'])  # ['ẩm thực', ...]
print(result['scores'])  # [0.95, ...]
```

### 5.6 Chỉ Định Thiết Bị (CPU/GPU)

```python
# Dùng GPU (device=0)
classifier = pipeline("sentiment-analysis", device=0)

# Dùng CPU (device=-1, mặc định)
classifier = pipeline("sentiment-analysis", device=-1)

# Tự động chọn
import torch
device = 0 if torch.cuda.is_available() else -1
classifier = pipeline("sentiment-analysis", device=device)
```

---

## 6. AutoModel & AutoTokenizer

### 6.1 Khi Nào Dùng Thay Vì Pipeline?

Pipeline tiện nhưng thiếu linh hoạt. Dùng `AutoModel` khi cần:
- Tùy chỉnh chi tiết input/output
- Fine-tune model
- Kết hợp nhiều model
- Tối ưu hiệu suất

### 6.2 Cách Dùng Cơ Bản

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

# 1. Tải tokenizer và model
model_name = "distilbert-base-uncased-finetuned-sst-2-english"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)

# 2. Tokenize input
text = "Tôi yêu thích lập trình!"
inputs = tokenizer(text, return_tensors="pt")
print(f"Input IDs: {inputs['input_ids']}")

# 3. Chạy model
with torch.no_grad():  # không cần gradient khi inference
    outputs = model(**inputs)

# 4. Xử lý output
logits = outputs.logits
predicted = torch.argmax(logits, dim=-1)
print(f"Logits: {logits}")
print(f"Predicted class: {predicted.item()}")
print(f"Label: {model.config.id2label[predicted.item()]}")
```

### 6.3 Tokenizer Chi Tiết

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

# Tokenize
text = "Hello, how are you?"
tokens = tokenizer.tokenize(text)
print(f"Tokens: {tokens}")
# ['hello', ',', 'how', 'are', 'you', '?']

# Chuyển thành IDs
ids = tokenizer.convert_tokens_to_ids(tokens)
print(f"IDs: {ids}")

# Encode (tokenize + thêm special tokens)
encoded = tokenizer(text)
print(f"Encoded: {encoded}")
# {'input_ids': [101, 7592, 1010, ...], 'attention_mask': [...]}

# Decode (IDs → text)
decoded = tokenizer.decode(encoded['input_ids'])
print(f"Decoded: {decoded}")

# Xử lý batch (nhiều câu)
texts = ["Hello world", "How are you?", "I love AI"]
batch = tokenizer(texts, padding=True, truncation=True, return_tensors="pt")
print(f"Batch shape: {batch['input_ids'].shape}")
```

### 6.4 Các Loại AutoModel

```python
from transformers import (
    AutoModel,                          # model cơ bản (embeddings)
    AutoModelForSequenceClassification, # phân loại câu
    AutoModelForTokenClassification,    # NER
    AutoModelForQuestionAnswering,      # hỏi đáp
    AutoModelForCausalLM,               # sinh text (GPT)
    AutoModelForSeq2SeqLM,              # dịch, tóm tắt (T5, BART)
    AutoModelForMaskedLM,               # BERT
    AutoModelForImageClassification,    # phân loại ảnh
)

# Ví dụ: model sinh text
from transformers import AutoTokenizer, AutoModelForCausalLM

tokenizer = AutoTokenizer.from_pretrained("gpt2")
model = AutoModelForCausalLM.from_pretrained("gpt2")

inputs = tokenizer("Hôm nay trời", return_tensors="pt")
outputs = model.generate(**inputs, max_length=30, do_sample=True)
print(tokenizer.decode(outputs[0]))
```

### 6.5 Lưu & Tải Model Local

```python
# Lưu model và tokenizer ra thư mục
model.save_pretrained("./my-model")
tokenizer.save_pretrained("./my-model")

# Tải lại từ thư mục
from transformers import AutoModel, AutoTokenizer
model = AutoModel.from_pretrained("./my-model")
tokenizer = AutoTokenizer.from_pretrained("./my-model")
```

---

## 7. Datasets — Thư Viện Dữ Liệu

### 7.1 Datasets Là Gì?

Thư viện để tải và xử lý dataset từ HF Hub — **100K+ dataset** có sẵn, tải bằng 1 dòng code.

### 7.2 Tải Dataset

```python
from datasets import load_dataset

# Tải dataset
dataset = load_dataset("squad")  # dataset hỏi đáp nổi tiếng
print(dataset)
# DatasetDict({
#     train: Dataset({...}),
#     validation: Dataset({...})
# })

# Xem mẫu
print(dataset["train"][0])

# Xem thông tin
print(f"Số mẫu train: {len(dataset['train'])}")
print(f"Cột: {dataset['train'].column_names}")
```

### 7.3 Dataset Tiếng Việt

```python
from datasets import load_dataset

# PhoBERT dataset
ds = load_dataset("vilm/vlsp2013 POS")  # gán nhãn từ loại TV

# Xem các dataset tiếng Việt
# Vào https://huggingface.co/datasets?language=language:vi
```

### 7.4 Xử Lý Dataset

```python
from datasets import load_dataset

ds = load_dataset("imdb")  # review phim

# Lọc
positive = ds["train"].filter(lambda x: x["label"] == 1)
print(f"Số review tích cực: {len(positive)}")

# Chọn cột
texts = ds["train"].select_columns(["text"])

# Đổi tên cột
ds = ds.rename_column("text", "review")

# Map (áp dụng hàm cho từng mẫu)
def clean_text(example):
    example["text"] = example["text"].lower().strip()
    return example

ds = ds.map(clean_text)

# Chia train/test
split = ds["train"].train_test_split(test_size=0.2)
print(split)
```

### 7.5 Tokenize Dataset Hàng Loạt

```python
from datasets import load_dataset
from transformers import AutoTokenizer

ds = load_dataset("imdb")
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

def tokenize_function(examples):
    return tokenizer(examples["text"], padding="max_length", truncation=True)

# Tokenize toàn bộ dataset (nhanh, dùng nhiều CPU)
tokenized = ds.map(tokenize_function, batched=True)
print(tokenized["train"][0].keys())
```

### 7.6 Lưu & Tải Dataset Local

```python
# Lưu ra disk
ds.save_to_disk("./my-dataset")

# Tải lại
from datasets import load_from_disk
ds = load_from_disk("./my-dataset")
```

### 7.7 Tạo Dataset Từ File Của Bạn

```python
from datasets import load_dataset

# Từ CSV
ds = load_dataset("csv", data_files="du_lieu.csv")

# Từ JSON
ds = load_dataset("json", data_files="du_lieu.json")

# Từ text (mỗi dòng 1 mẫu)
ds = load_dataset("text", data_files="van_ban.txt")

# Từ pandas DataFrame
import pandas as pd
from datasets import Dataset
df = pd.DataFrame({"text": ["a", "b"], "label": [0, 1]})
ds = Dataset.from_pandas(df)
```

---

## 8. huggingface_hub — Upload/Download

### 8.1 Tải Model Từ Hub

```python
from huggingface_hub import snapshot_download

# Tải toàn bộ repo model về local
path = snapshot_download(
    repo_id="bert-base-uncased",
    local_dir="./bert-local"
)
print(f"Đã tải về: {path}")
```

```python
# Tải 1 file cụ thể
from huggingface_hub import hf_hub_download

path = hf_hub_download(
    repo_id="bert-base-uncased",
    filename="config.json"
)
print(path)
```

### 8.2 Tạo Repository Mới

```python
from huggingface_hub import create_repo

# Tạo repo model
create_repo("my-awesome-model", private=False)

# Tạo repo dataset
create_repo("my-dataset", repo_type="dataset", private=True)

# Tạo repo Space
create_repo("my-demo", repo_type="space", space_sdk="gradio")
```

### 8.3 Upload Model Lên Hub

```python
from transformers import AutoModel, AutoTokenizer

# Tải model local
model = AutoModel.from_pretrained("./my-model")
tokenizer = AutoTokenizer.from_pretrained("./my-model")

# Đẩy lên Hub (cần đăng nhập trước)
model.push_to_hub("username/my-awesome-model")
tokenizer.push_to_hub("username/my-awesome-model")
print("✅ Đã upload!")
```

```python
# Upload file bất kỳ
from huggingface_hub import upload_file

upload_file(
    path_or_fileobj="./ket_qua.zip",
    path_in_repo="ket_qua.zip",
    repo_id="username/my-repo",
)
```

```python
# Upload cả thư mục
from huggingface_hub import upload_folder

upload_folder(
    folder_path="./my-model-folder",
    repo_id="username/my-awesome-model",
)
```

### 8.4 Liệt Kê Model Của Bạn

```python
from huggingface_hub import list_models

# Model của 1 user
for model in list_models(author="username"):
    print(model.id)

# Tìm model theo task
for model in list_models(filter="text-classification", sort="downloads", limit=5):
    print(f"{model.id}: {model.downloads} downloads")
```

### 8.5 Xóa File Trên Hub

```python
from huggingface_hub import delete_file

delete_file(
    path_in_repo="file_cu.txt",
    repo_id="username/my-repo",
)
```

---

## 9. Spaces — Deploy Demo Miễn Phí

### 9.1 Spaces Là Gì?

**Spaces** là nơi deploy demo AI miễn phí — như "Vercel cho AI". Bạn viết app Gradio/Streamlit, push lên, có link public ngay.

### 9.2 Tạo Space Bằng Web

1. Vào https://huggingface.co/spaces
2. Bấm **Create new Space**
3. Điền tên, chọn SDK:
   - **Gradio:** Dễ nhất cho demo ML
   - **Streamlit:** App data linh hoạt
   - **Static:** Web HTML/CSS/JS thuần
4. Bấm **Create Space**

### 9.3 Tạo Space Bằng Code

```python
from huggingface_hub import create_repo

create_repo(
    "my-sentiment-demo",
    repo_type="space",
    space_sdk="gradio",  # hoặc "streamlit", "static"
)
```

### 9.4 Viết App Gradio Đơn Giản

```python
# app.py
import gradio as gr
from transformers import pipeline

# Tải model
classifier = pipeline("sentiment-analysis")

def phan_tich_cam_xuc(text):
    result = classifier(text)[0]
    nhan = "😊 Tích cực" if result['label'] == 'POSITIVE' else "😢 Tiêu cực"
    return f"{nhan} (độ tin cậy: {result['score']:.2%})"

# Tạo giao diện
demo = gr.Interface(
    fn=phan_tich_cam_xuc,
    inputs=gr.Textbox(label="Nhập câu tiếng Anh", placeholder="I love this!"),
    outputs=gr.Textbox(label="Kết quả"),
    title="Phân Tích Cảm Xúc 🤗",
    description="Nhập câu tiếng Anh để phân tích cảm xúc"
)

demo.launch()
```

**File `requirements.txt`:**
```
gradio
transformers
torch
```

### 9.5 Push Lên Space

```bash
# Clone Space repo
git clone https://huggingface.co/spaces/username/my-sentiment-demo
cd my-sentiment-demo

# Copy file vào
cp app.py requirements.txt .

# Push
git add .
git commit -m "Add sentiment demo"
git push
```

→ Sau ~1 phút, demo chạy tại `huggingface.co/spaces/username/my-sentiment-demo`!

### 9.6 Cấu Hình Phần Cứng

Tạo file `.gitattributes` hoặc vào **Settings** của Space:

| Hardware | Giá | Dùng khi nào |
|----------|-----|--------------|
| CPU basic | Miễn phí | Demo nhẹ |
| CPU upgrade | $ | Xử lý nặng hơn |
| T4 small | $$ | Cần GPU nhỏ |
| A10G | $$$ | Model lớn |

---

## 10. Inference API — Gọi Model Qua HTTP

### 10.1 Inference API Là Gì?

Gọi bất kỳ model nào trên Hub qua HTTP request — **không cần tải model về!**

### 10.2 Gọi Bằng Python (requests)

```python
import requests
import getpass

# Token (cần cho rate limit cao hơn)
TOKEN = getpass.getpass("HF Token: ")
headers = {"Authorization": f"Bearer {TOKEN}"}

# Gọi API
API_URL = "https://api-inference.huggingface.co/models/distilbert-base-uncased-finetuned-sst-2-english"

response = requests.post(
    API_URL,
    headers=headers,
    json={"inputs": "I love Hugging Face!"}
)
print(response.json())
# [{'label': 'POSITIVE', 'score': 0.9998}]
```

### 10.3 Gọi Bằng cURL

```bash
curl https://api-inference.huggingface.co/models/gpt2 \
  -X POST \
  -d '{"inputs": "Hôm nay trời"}' \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### 10.4 Dùng huggingface_hub (Khuyến Nghị)

```python
from huggingface_hub import InferenceClient
import getpass

client = InferenceClient(token=getpass.getpass("HF Token: "))

# Text classification
result = client.text_classification(
    "Tôi thích AI!",
    model="distilbert-base-uncased-finetuned-sst-2-english"
)
print(result)

# Text generation
text = client.text_generation(
    "Hôm nay tôi",
    model="gpt2",
    max_new_tokens=30
)
print(text)

# Chat (dùng model chat)
messages = [{"role": "user", "content": "Xin chào!"}]
response = client.chat_completion(
    messages,
    model="meta-llama/Llama-3.2-3B-Instruct"
)
print(response.choices[0].message.content)
```

### 10.5 Rate Limit

| Loại tài khoản | Requests/phút |
|----------------|---------------|
| Không token | Rất thấp |
| Free (có token) | ~thấp |
| Pro ($9/tháng) | Cao hơn |

> 💡 Model đang "cold" (chưa ai dùng) cần ~20s để load lần đầu.

---

## 11. Fine-Tuning Cơ Bản

### 11.1 Fine-Tuning Là Gì?

Lấy model có sẵn → train thêm trên dữ liệu của bạn → model hiểu task cụ thể.

```
Model gốc (biết tiếng Anh chung)
    ↓ + dữ liệu review phim của bạn
Model fine-tuned (giỏi phân tích review phim)
```

### 11.2 Chuẩn Bị Dữ Liệu

```python
from datasets import load_dataset

# Dùng dataset IMDB (review phim)
dataset = load_dataset("imdb")
print(dataset["train"][0])
# {'text': '...', 'label': 0}
```

### 11.3 Tokenize

```python
from transformers import AutoTokenizer

tokenizer = AutoTokenizer.from_pretrained("distilbert-base-uncased")

def tokenize_function(examples):
    return tokenizer(examples["text"], padding="max_length",
                     truncation=True, max_length=256)

tokenized = dataset.map(tokenize_function, batched=True)
```

### 11.4 Train Với Trainer API

```python
from transformers import (
    AutoModelForSequenceClassification,
    TrainingArguments,
    Trainer
)
import numpy as np
from datasets import load_metric

# 1. Model
model = AutoModelForSequenceClassification.from_pretrained(
    "distilbert-base-uncased",
    num_labels=2
)

# 2. Metric đánh giá
metric = load_metric("accuracy")
def compute_metrics(eval_pred):
    logits, labels = eval_pred
    predictions = np.argmax(logits, axis=-1)
    return metric.compute(predictions=predictions, references=labels)

# 3. Cấu hình train
args = TrainingArguments(
    output_dir="./ket-qua",
    evaluation_strategy="epoch",  # đánh giá mỗi epoch
    learning_rate=2e-5,           # tốc độ học
    per_device_train_batch_size=16,
    per_device_eval_batch_size=16,
    num_train_epochs=3,           # số vòng train
    weight_decay=0.01,
    save_total_limit=2,           # chỉ giữ 2 checkpoint
    push_to_hub=False,            # True để auto upload
)

# 4. Trainer
trainer = Trainer(
    model=model,
    args=args,
    train_dataset=tokenized["train"].shuffle().select(range(1000)),
    eval_dataset=tokenized["test"].select(range(200)),
    tokenizer=tokenizer,
    compute_metrics=compute_metrics,
)

# 5. Train!
trainer.train()
print("✅ Train xong!")
```

### 11.5 Đánh Giá & Lưu

```python
# Đánh giá
results = trainer.evaluate()
print(f"Accuracy: {results['eval_accuracy']:.4f}")

# Lưu model
trainer.save_model("./model-finetuned")
tokenizer.save_pretrained("./model-finetuned")

# Đẩy lên Hub
trainer.push_to_hub("username/my-finetuned-model")
```

### 11.6 Dùng Model Đã Fine-Tune

```python
from transformers import pipeline

classifier = pipeline(
    "sentiment-analysis",
    model="./model-finetuned"
)
print(classifier("Bộ phim này tuyệt vời!"))
```

### 11.7 Fine-Tune LLM Với LoRA (Tiết Kiệm VRAM)

```bash
pip install -q peft trl bitsandbytes accelerate
```

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training

model_name = "meta-llama/Llama-3.2-1B"

# Tải model dạng 4-bit (tiết kiệm VRAM)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    load_in_4bit=True,
    device_map="auto"
)
model = prepare_model_for_kbit_training(model)

# Cấu hình LoRA (chỉ train 1% tham số!)
lora_config = LoraConfig(
    r=16,                    # rank
    lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
    task_type="CAUSAL_LM"
)
model = get_peft_model(model, lora_config)
model.print_trainable_parameters()
# trainable params: ~1M / 1B (0.1%!)
```

---

## 12. Model Cards — Tài Liệu Model

### 12.1 Model Card Là Gì?

File `README.md` trong mỗi repo model — mô tả model làm gì, train thế nào, dùng ra sao.

### 12.2 Đọc Model Card

Khi chọn model, đọc các phần:
- **Model Description:** Model này làm gì?
- **Intended Use:** Dùng cho task nào?
- **Limitations:** Hạn chế gì? (bias, ngôn ngữ...)
- **Training Data:** Train trên dữ liệu nào?
- **How to Use:** Code mẫu

### 12.3 Viết Model Card Cho Model Của Bạn

```markdown
---
language: vi
license: apache-2.0
tags:
- sentiment-analysis
- vietnamese
datasets:
- my-dataset
---

# My Vietnamese Sentiment Model

## Mô tả
Model phân tích cảm xúc tiếng Việt, fine-tune từ PhoBERT.

## Cách dùng
\`\`\`python
from transformers import pipeline
clf = pipeline("sentiment-analysis", model="username/my-model")
clf("Tôi rất vui!")
\`\`\`

## Kết quả
| Metric | Score |
|--------|-------|
| Accuracy | 0.94 |
| F1 | 0.93 |

## Hạn chế
- Chỉ tốt với review sản phẩm
- Không hiểu tốt tiếng lóng
```

---

## 13. Xử Lý Lỗi Thường Gặp

| Lỗi | Nguyên nhân | Cách fix |
|-----|-------------|----------|
| `401 Unauthorized` | Token sai/hết hạn | Tạo token mới, login lại |
| `404 Model not found` | Sai tên model | Kiểm tra tên trên hub |
| `CUDA out of memory` | Model quá lớn | Dùng model nhỏ hơn, `device_map="auto"` |
| `Token indices out of range` | Text quá dài | Thêm `truncation=True`, giảm `max_length` |
| `ConnectionError` | Mất mạng / HF down | Thử lại, kiểm tra https://status.huggingface.co |
| `OSError: Can't load tokenizer` | Thiếu file tokenizer | Dùng `trust_remote_code=True` nếu model yêu cầu |
| `ValueError: num_labels` | Số label không khớp | Set `num_labels` đúng khi tải model |

```python
# Fix: model yêu cầu trust_remote_code
tokenizer = AutoTokenizer.from_pretrained(
    "model-la",
    trust_remote_code=True
)
model = AutoModel.from_pretrained(
    "model-la",
    trust_remote_code=True
)
```

```python
# Fix: hết VRAM khi tải model lớn
model = AutoModelForCausalLM.from_pretrained(
    "model-lon",
    device_map="auto",       # tự chia qua GPU/CPU
    load_in_8bit=True,       # nén 8-bit
    # hoặc load_in_4bit=True # nén 4-bit (cần bitsandbytes)
)
```

---

## 14. Best Practices

### 14.1 Bảo Mật Token
- ✅ Dùng `getpass` để nhập token
- ✅ Lưu token trong biến môi trường
- ✅ Dùng token `read` khi chỉ cần tải
- ❌ Không hardcode token vào code
- ❌ Không commit token lên GitHub

### 14.2 Tiết Kiệm Tài Nguyên
```python
# Dùng torch.no_grad() khi inference
import torch
with torch.no_grad():
    outputs = model(inputs)

# Dọn VRAM sau khi xong
del model
torch.cuda.empty_cache()
```

### 14.3 Cache Model
```python
# Chỉ định thư mục cache (tránh tải lại)
from transformers import AutoModel

model = AutoModel.from_pretrained(
    "bert-base-uncased",
    cache_dir="/content/drive/MyDrive/hf_cache"
)
```

### 14.4 Chọn Model Phù Hợp
1. Xem **downloads** cao = được tin dùng
2. Đọc **model card** kỹ
3. Kiểm tra **license** (có được dùng thương mại không?)
4. Test trên dữ liệu của bạn trước khi dùng production

### 14.5 Version Pinning
```bash
# Ghim phiên bản để tránh lỗi khi update
pip install transformers==4.40.0 datasets==2.19.0
```

---

## 15. Tài Liệu Tham Khảo

- 🏠 Trang chủ: https://huggingface.co
- 📚 Docs Transformers: https://huggingface.co/docs/transformers
- 📊 Docs Datasets: https://huggingface.co/docs/datasets
- 🤗 Docs Hub: https://huggingface.co/docs/huggingface_hub
- 🎓 Khóa học miễn phí: https://huggingface.co/learn/nlp-course
- 💬 Forum: https://discuss.huggingface.co
- 📺 YouTube: Hugging Face channel

---

*📝 Viết bởi **Capybara** 🦫 cho **Tiểu Dươngg** — 2026-10-10*
