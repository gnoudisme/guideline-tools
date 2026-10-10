# Hướng Dẫn Toàn Diện OpenRouter 🔀

> Tài liệu tiếng Việt chi tiết về OpenRouter — dùng hàng trăm model AI qua 1 API duy nhất.
> Tác giả: Capybara 🦫 cho Tiểu Dươngg

---

## Mục Lục

1. [OpenRouter Là Gì?](#1-openrouter-là-gì)
2. [Tạo Tài Khoản & API Key](#2-tạo-tài-khoản--api-key)
3. [Model Miễn Phí (:free)](#3-model-miễn-phí-free)
4. [Cách Gọi API Cơ Bản](#4-cách-gọi-api-cơ-bản)
5. [Ví Dụ Python](#5-ví-dụ-python)
6. [Ví Dụ JavaScript](#6-ví-dụ-javascript)
7. [Ví Dụ cURL](#7-ví-dụ-curl)
8. [Chọn Model Phù Hợp](#8-chọn-model-phù-hợp)
9. [Bảng Giá & Pricing](#9-bảng-giá--pricing)
10. [Rate Limits](#10-rate-limits)
11. [Streaming — Trả Lời Từng Chữ](#11-streaming--trả-lời-từng-chữ)
12. [Xử Lý Lỗi](#12-xử-lý-lỗi)
13. [Tính Năng Nâng Cao](#13-tính-năng-nâng-cao)
14. [Tích Hợp Với Công Cụ](#14-tích-hợp-với-công-cụ)
15. [Best Practices](#15-best-practices)
16. [Tài Liệu Tham Khảo](#16-tài-liệu-tham-khảo)

---

## 1. OpenRouter Là Gì?

### 1.1 Định Nghĩa

**OpenRouter** (https://openrouter.ai) là "chợ" API cho các model AI — **1 API key dùng được hàng trăm model** từ OpenAI, Anthropic, Google, Meta, Mistral...

```
Thay vì:
  OpenAI API key → chỉ dùng GPT
  Anthropic key  → chỉ dùng Claude
  Google key     → chỉ dùng Gemini

Dùng OpenRouter:
  1 API key → GPT + Claude + Gemini + Llama + Mistral + ...
```

### 1.2 Tại Sao Dùng OpenRouter?

| Lợi ích | Giải thích |
|---------|------------|
| **1 key cho tất cả** | Không cần đăng ký từng nhà cung cấp |
| **Model miễn phí** | Nhiều model `:free` không tốn tiền |
| **Tự động fallback** | Model A lỗi → tự chuyển sang Model B |
| **So sánh giá** | Xem giá từng model, chọn rẻ nhất |
| **OpenAI-compatible** | Code viết cho OpenAI chạy được ngay |
| **Không cần thẻ** | Model free không cần nạp tiền |

### 1.3 So Sánh Với Gọi Trực Tiếp

| Tiêu chí | Gọi trực tiếp OpenAI | OpenRouter |
|----------|---------------------|------------|
| Số model | 1 nhà cung cấp | 300+ model |
| API key | Mỗi nơi 1 key | 1 key duy nhất |
| Model free | Ít | Nhiều (`:free`) |
| Đổi model | Sửa code + key mới | Chỉ đổi tên model |
| Giá | Giá gốc | Giá gốc + phí nhỏ |

---

## 2. Tạo Tài Khoản & API Key

### 2.1 Đăng Ký

1. Vào https://openrouter.ai
2. Bấm **Sign In** → đăng nhập bằng Google/GitHub
3. Xong! Không cần thẻ tín dụng để bắt đầu.

### 2.2 Tạo API Key

1. Vào https://openrouter.ai/settings/keys
2. Bấm **Create Key**
3. Đặt tên (ví dụ: `my-app`)
4. **Copy key ngay** — dạng `sk-or-v1-abc...xyz`
5. (Tùy chọn) Set giới hạn chi tiêu cho key

> ⚠️ **Bảo mật:** Key như mật khẩu! Không chia sẻ, không commit lên GitHub.

### 2.3 Nạp Credit (Tùy Chọn)

- Model `:free` → **không cần nạp tiền**
- Model trả phí → nạp tối thiểu **$5** qua thẻ
- Vào https://openrouter.ai/settings/credits → **Add Credits**

### 2.4 Kiểm Tra Key Có Hoạt Động Không

```bash
curl https://openrouter.ai/api/v1/auth/key \
  -H "Authorization: Bearer sk-or-v1-YOUR_KEY"
```

Trả về JSON với thông tin key và số dư:
```json
{
  "data": {
    "label": "my-app",
    "limit": null,
    "usage": 0.12,
    "is_free_tier": true
  }
}
```

---

## 3. Model Miễn Phí (:free)

### 3.1 Model Free Là Gì?

Model có hậu tố `:free` — **gọi không tốn tiền**, không cần nạp credit!

### 3.2 Danh Sách Model Free Phổ Biến (2026)

| Model | ID đầy đủ | Điểm mạnh |
|-------|-----------|-----------|
| Llama 3.2 3B | `meta-llama/llama-3.2-3b-instruct:free` | Nhẹ, nhanh |
| Qwen 2.5 7B | `qwen/qwen-2.5-7b-instruct:free` | Tốt tiếng Việt |
| Mistral 7B | `mistralai/mistral-7b-instruct:free` | Chất lượng tốt |
| Gemma 2 9B | `google/gemma-2-9b-it:free` | Google, ổn định |
| DeepSeek R1 | `deepseek/deepseek-r1:free` | Reasoning mạnh |
| Phi-4 | `microsoft/phi-4:free` | Microsoft, nhỏ gọn |

> 💡 Danh sách thay đổi theo thời gian. Xem đầy đủ tại: https://openrouter.ai/models?max_price=0

### 3.3 Tìm Model Free

1. Vào https://openrouter.ai/models
2. Lọc **Max price = $0**
3. Sắp xếp theo **Most popular**

### 3.4 Giới Hạn Của Model Free

- **Rate limit thấp:** ~20 requests/phút
- **Tốc độ chậm hơn** bản trả phí
- **Có thể hết quota** giờ cao điểm
- **Không ưu tiên** khi server bận

> 💡 Đủ dùng cho học tập, test, project cá nhân. Production nên dùng bản trả phí.

---

## 4. Cách Gọi API Cơ Bản

### 4.1 Endpoint

```
POST https://openrouter.ai/api/v1/chat/completions
```

### 4.2 Headers Bắt Buộc

```
Authorization: Bearer sk-or-v1-YOUR_KEY
Content-Type: application/json
```

### 4.3 Headers Khuyến Nghị (Tùy Chọn)

```
HTTP-Referer: https://your-app.com    # URL app của bạn
X-Title: Ten App Cua Ban              # Tên app (hiện trên dashboard)
```

> OpenRouter dùng thông tin này để thống kê. Một số model free **yêu cầu** có Referer.

### 4.4 Body Request (Chuẩn OpenAI)

```json
{
  "model": "qwen/qwen-2.5-7b-instruct:free",
  "messages": [
    {"role": "system", "content": "Bạn là trợ lý AI thân thiện."},
    {"role": "user", "content": "Xin chào!"}
  ],
  "temperature": 0.7,
  "max_tokens": 500
}
```

### 4.5 Response

```json
{
  "id": "gen-abc123",
  "model": "qwen/qwen-2.5-7b-instruct:free",
  "choices": [
    {
      "message": {
        "role": "assistant",
        "content": "Xin chào! Tôi có thể giúp gì cho bạn?"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 15,
    "completion_tokens": 12,
    "total_tokens": 27
  }
}
```

---

## 5. Ví Dụ Python

### 5.1 Dùng `requests` (Không Cần Cài Gì Thêm)

```python
import requests
import json
import getpass

# Nhập key an toàn
API_KEY = getpass.getpass("Nhập OpenRouter API key: ")

def chat(prompt, model="qwen/qwen-2.5-7b-instruct:free"):
    response = requests.post(
        url="https://openrouter.ai/api/v1/chat/completions",
        headers={
            "Authorization": f"Bearer {API_KEY}",
            "Content-Type": "application/json",
            "HTTP-Referer": "https://my-app.com",
            "X-Title": "My App",
        },
        data=json.dumps({
            "model": model,
            "messages": [
                {"role": "user", "content": prompt}
            ],
        })
    )
    result = response.json()
    return result['choices'][0]['message']['content']

# Dùng thử
print(chat("Viết 1 câu thơ về mùa thu"))
```

### 5.2 Dùng Thư Viện `openai` (Khuyến Nghị!)

OpenRouter tương thích 100% với thư viện OpenAI — chỉ cần đổi `base_url`!

```bash
pip install openai
```

```python
from openai import OpenAI
import getpass

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=getpass.getpass("API key: "),
)

# Gọi như OpenAI bình thường!
completion = client.chat.completions.create(
    model="qwen/qwen-2.5-7b-instruct:free",
    messages=[
        {"role": "system", "content": "Bạn là trợ lý AI."},
        {"role": "user", "content": "Giải thích AI là gì trong 2 câu."}
    ],
    extra_headers={
        "HTTP-Referer": "https://my-app.com",
        "X-Title": "My App",
    }
)

print(completion.choices[0].message.content)
print(f"Tokens: {completion.usage.total_tokens}")
```

### 5.3 Chat Nhiều Lượt (Giữ Ngữ Cảnh)

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key="sk-or-v1-YOUR_KEY",
)

messages = [
    {"role": "system", "content": "Bạn là Capybara, trợ lý thân thiện."}
]

while True:
    user_input = input("Bạn: ")
    if user_input.lower() in ['quit', 'exit', 'thoát']:
        break

    messages.append({"role": "user", "content": user_input})

    completion = client.chat.completions.create(
        model="qwen/qwen-2.5-7b-instruct:free",
        messages=messages,
    )

    reply = completion.choices[0].message.content
    print(f"AI: {reply}\n")

    messages.append({"role": "assistant", "content": reply})
```

### 5.4 Đổi Model Dễ Dàng

```python
# Chỉ cần đổi tên model!
MODELS = {
    "nhanh": "meta-llama/llama-3.2-3b-instruct:free",
    "tieng_viet": "qwen/qwen-2.5-7b-instruct:free",
    "manh": "deepseek/deepseek-r1:free",
    "tra_phi": "anthropic/claude-3.5-sonnet",  # cần credit
}

def chat_voi_model(prompt, ten_model="nhanh"):
    completion = client.chat.completions.create(
        model=MODELS[ten_model],
        messages=[{"role": "user", "content": prompt}],
    )
    return completion.choices[0].message.content

print(chat_voi_model("Xin chào", "nhanh"))
print(chat_voi_model("Xin chào", "manh"))
```

---

## 6. Ví Dụ JavaScript

### 6.1 Dùng `fetch` (Browser / Node.js)

```javascript
const API_KEY = 'sk-or-v1-YOUR_KEY';

async function chat(prompt) {
  const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${API_KEY}`,
      'Content-Type': 'application/json',
      'HTTP-Referer': 'https://my-app.com',
      'X-Title': 'My App',
    },
    body: JSON.stringify({
      model: 'qwen/qwen-2.5-7b-instruct:free',
      messages: [
        { role: 'user', content: prompt }
      ],
    }),
  });

  const data = await response.json();
  return data.choices[0].message.content;
}

// Dùng thử
chat('Viết 1 câu đùa').then(console.log);
```

### 6.2 Dùng Thư Viện OpenAI (Node.js)

```bash
npm install openai
```

```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: process.env.OPENROUTER_API_KEY,
});

const completion = await client.chat.completions.create({
  model: 'qwen/qwen-2.5-7b-instruct:free',
  messages: [
    { role: 'user', content: 'Xin chào!' }
  ],
});

console.log(completion.choices[0].message.content);
```

### 6.3 Trong React

```jsx
import { useState } from 'react';

function ChatApp() {
  const [input, setInput] = useState('');
  const [reply, setReply] = useState('');
  const [loading, setLoading] = useState(false);

  const sendMessage = async () => {
    setLoading(true);
    try {
      const res = await fetch('https://openrouter.ai/api/v1/chat/completions', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${import.meta.env.VITE_OR_KEY}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          model: 'qwen/qwen-2.5-7b-instruct:free',
          messages: [{ role: 'user', content: input }],
        }),
      });
      const data = await res.json();
      setReply(data.choices[0].message.content);
    } catch (e) {
      setReply('Lỗi: ' + e.message);
    }
    setLoading(false);
  };

  return (
    <div>
      <input value={input} onChange={e => setInput(e.target.value)} />
      <button onClick={sendMessage} disabled={loading}>
        {loading ? 'Đang nghĩ...' : 'Gửi'}
      </button>
      <p>{reply}</p>
    </div>
  );
}
```

> ⚠️ **Bảo mật:** Không để API key trong code frontend! Dùng backend proxy hoặc biến môi trường.

---

## 7. Ví Dụ cURL

```bash
# Chat đơn giản
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-or-v1-YOUR_KEY" \
  -H "HTTP-Referer: https://my-app.com" \
  -d '{
    "model": "qwen/qwen-2.5-7b-instruct:free",
    "messages": [
      {"role": "user", "content": "Xin chào!"}
    ]
  }'
```

```bash
# Với system prompt và temperature
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-or-v1-YOUR_KEY" \
  -d '{
    "model": "meta-llama/llama-3.2-3b-instruct:free",
    "messages": [
      {"role": "system", "content": "Bạn là nhà thơ."},
      {"role": "user", "content": "Viết thơ về biển"}
    ],
    "temperature": 0.9,
    "max_tokens": 200
  }'
```

```bash
# Lấy danh sách model
curl https://openrouter.ai/api/v1/models \
  -H "Authorization: Bearer sk-or-v1-YOUR_KEY" | python3 -m json.tool | head -50
```

---

## 8. Chọn Model Phù Hợp

### 8.1 Theo Nhu Cầu

| Nhu cầu | Model gợi ý | Giá |
|---------|-------------|-----|
| Test, học tập | `qwen/qwen-2.5-7b-instruct:free` | Miễn phí |
| Tiếng Việt tốt | `qwen/qwen-2.5-7b-instruct:free` | Miễn phí |
| Reasoning/suy luận | `deepseek/deepseek-r1:free` | Miễn phí |
| Nhanh nhất | `meta-llama/llama-3.2-3b-instruct:free` | Miễn phí |
| Chất lượng cao | `anthropic/claude-3.5-sonnet` | ~$3/1M tokens |
| Code tốt | `anthropic/claude-3.5-sonnet` | ~$3/1M tokens |
| Rẻ mà tốt | `google/gemini-flash-1.5` | ~$0.07/1M tokens |

### 8.2 Đọc Thông Số Model

Vào https://openrouter.ai/models → bấm vào model → xem:
- **Context length:** Độ dài tối đa (VD: 128K tokens)
- **Prompt price:** Giá cho 1M tokens input
- **Completion price:** Giá cho 1M tokens output
- **Throughput:** Tốc độ (tokens/giây)

### 8.3 Model Router Tự Động

Để OpenRouter tự chọn model rẻ nhất/nhanh nhất:

```python
# Dùng "auto" router
completion = client.chat.completions.create(
    model="openrouter/auto",  # tự chọn!
    messages=[{"role": "user", "content": "Xin chào"}],
)
print(f"Model được chọn: {completion.model}")
```

---

## 9. Bảng Giá & Pricing

### 9.1 Cách Tính Giá

Giá tính theo **token** (đơn vị text). 1 token ≈ 0.75 từ tiếng Anh, ≈ 1-2 ký tự tiếng Việt.

```
Chi phí = (prompt_tokens × giá_input + completion_tokens × giá_output) / 1,000,000
```

**Ví dụ:**
- Gửi 1000 tokens, nhận 500 tokens
- Model giá $1/1M input, $2/1M output
- Chi phí = (1000×1 + 500×2)/1M = $0.002

### 9.2 Ước Tính Chi Phí

```python
# Tính chi phí sau mỗi lần gọi
completion = client.chat.completions.create(...)

usage = completion.usage
# Giả sử giá: $0.5/1M input, $1/1M output
cost = (usage.prompt_tokens * 0.5 + usage.completion_tokens * 1) / 1_000_000
print(f"Chi phí: ${cost:.6f}")
print(f"Prompt: {usage.prompt_tokens}, Completion: {usage.completion_tokens}")
```

### 9.3 Mẹo Tiết Kiệm

1. **Dùng model `:free`** khi test/học
2. **Giảm `max_tokens`** nếu không cần câu trả lời dài
3. **Rút gọn system prompt** — prompt dài = tốn tiền
4. **Cache** câu trả lời cho câu hỏi lặp lại
5. **Dùng model nhỏ** cho task đơn giản

---

## 10. Rate Limits

### 10.1 Giới Hạn Theo Gói

| Gói | Requests/phút | Requests/ngày |
|-----|---------------|---------------|
| Free (chưa nạp tiền) | ~20 | Giới hạn |
| Đã nạp $10+ | ~200 | Cao |
| Đã nạp $50+ | ~500 | Rất cao |

### 10.2 Xử Lý Rate Limit Trong Code

```python
import time
from openai import OpenAI, RateLimitError

client = OpenAI(base_url="https://openrouter.ai/api/v1", api_key=KEY)

def chat_with_retry(prompt, max_retries=3):
    for i in range(max_retries):
        try:
            return client.chat.completions.create(
                model="qwen/qwen-2.5-7b-instruct:free",
                messages=[{"role": "user", "content": prompt}],
            )
        except RateLimitError:
            wait = 2 ** i  # 1s, 2s, 4s...
            print(f"Rate limit! Đợi {wait}s...")
            time.sleep(wait)
    raise Exception("Thử quá nhiều lần")
```

---

## 11. Streaming — Trả Lời Từng Chữ

### 11.1 Streaming Là Gì?

Thay vì đợi model trả lời xong mới hiện, **streaming** hiện từng chữ ngay khi có — như ChatGPT.

### 11.2 Python Streaming

```python
from openai import OpenAI

client = OpenAI(base_url="https://openrouter.ai/api/v1", api_key=KEY)

stream = client.chat.completions.create(
    model="qwen/qwen-2.5-7b-instruct:free",
    messages=[{"role": "user", "content": "Kể 1 câu chuyện ngắn"}],
    stream=True,  # bật streaming!
)

# In từng chunk ngay khi nhận được
for chunk in stream:
    content = chunk.choices[0].delta.content
    if content:
        print(content, end="", flush=True)
print()  # xuống dòng khi xong
```

### 11.3 JavaScript Streaming

```javascript
async function chatStream(prompt) {
  const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      model: 'qwen/qwen-2.5-7b-instruct:free',
      messages: [{ role: 'user', content: prompt }],
      stream: true,
    }),
  });

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const text = decoder.decode(value);
    // Parse SSE: mỗi dòng bắt đầu bằng "data: "
    for (const line of text.split('\n')) {
      if (line.startsWith('data: ') && line !== 'data: [DONE]') {
        const json = JSON.parse(line.slice(6));
        const content = json.choices[0]?.delta?.content;
        if (content) process.stdout.write(content);
      }
    }
  }
}
```

---

## 12. Xử Lý Lỗi

### 12.1 Các Lỗi Thường Gặp

| Mã lỗi | Ý nghĩa | Cách fix |
|--------|---------|----------|
| `401` | API key sai/thiếu | Kiểm tra key |
| `402` | Hết credit | Nạp thêm tiền |
| `404` | Model không tồn tại | Kiểm tra tên model |
| `429` | Rate limit | Đợi rồi thử lại |
| `500` | Lỗi server model | Thử model khác |
| `502` | Model đang load | Đợi 20s thử lại |

### 12.2 Code Xử Lý Lỗi Đầy Đủ

```python
from openai import OpenAI, APIError, RateLimitError, AuthenticationError
import time

client = OpenAI(base_url="https://openrouter.ai/api/v1", api_key=KEY)

def safe_chat(prompt, model="qwen/qwen-2.5-7b-instruct:free"):
    try:
        completion = client.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
        )
        return completion.choices[0].message.content

    except AuthenticationError:
        return "❌ API key sai! Kiểm tra lại key."

    except RateLimitError:
        return "⏳ Quá nhiều request! Đợi 1 phút rồi thử lại."

    except APIError as e:
        if e.status_code == 402:
            return "💳 Hết credit! Nạp thêm tại openrouter.ai/settings/credits"
        elif e.status_code == 404:
            return f"❌ Model '{model}' không tồn tại!"
        else:
            return f"❌ Lỗi API: {e}"

    except Exception as e:
        return f"❌ Lỗi không xác định: {e}"

# Dùng thử
print(safe_chat("Xin chào!"))
```

### 12.3 Fallback Khi Model Lỗi

```python
MODELS_FALLBACK = [
    "qwen/qwen-2.5-7b-instruct:free",
    "meta-llama/llama-3.2-3b-instruct:free",
    "google/gemma-2-9b-it:free",
]

def chat_with_fallback(prompt):
    for model in MODELS_FALLBACK:
        try:
            completion = client.chat.completions.create(
                model=model,
                messages=[{"role": "user", "content": prompt}],
            )
            print(f"✅ Dùng model: {model}")
            return completion.choices[0].message.content
        except Exception as e:
            print(f"⚠️ {model} lỗi: {e}, thử model tiếp theo...")
            continue
    return "❌ Tất cả model đều lỗi!"

print(chat_with_fallback("Xin chào!"))
```

---

## 13. Tính Năng Nâng Cao

### 13.1 Vision — Gửi Ảnh Cho AI

```python
import base64

# Đọc ảnh thành base64
with open("anh.jpg", "rb") as f:
    img_b64 = base64.b64encode(f.read()).decode()

completion = client.chat.completions.create(
    model="qwen/qwen-2.5-vl-7b-instruct:free",  # model hỗ trợ vision
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "Mô tả ảnh này"},
                {
                    "type": "image_url",
                    "image_url": {"url": f"data:image/jpeg;base64,{img_b64}"}
                }
            ]
        }
    ],
)
print(completion.choices[0].message.content)
```

### 13.2 Function Calling / Tools

```python
tools = [
    {
        "type": "function",
        "function": {
            "name": "xem_thoi_tiet",
            "description": "Xem thời tiết của 1 thành phố",
            "parameters": {
                "type": "object",
                "properties": {
                    "thanh_pho": {"type": "string", "description": "Tên thành phố"}
                },
                "required": ["thanh_pho"]
            }
        }
    }
]

completion = client.chat.completions.create(
    model="qwen/qwen-2.5-7b-instruct",  # model trả phí hỗ trợ tốt hơn
    messages=[{"role": "user", "content": "Thời tiết Hà Nội hôm nay?"}],
    tools=tools,
)

# Kiểm tra AI có muốn gọi function không
tool_call = completion.choices[0].message.tool_calls
if tool_call:
    print(f"AI muốn gọi: {tool_call[0].function.name}")
    print(f"Tham số: {tool_call[0].function.arguments}")
```

### 13.3 JSON Mode

```python
completion = client.chat.completions.create(
    model="qwen/qwen-2.5-7b-instruct:free",
    messages=[
        {"role": "user", "content": "Liệt kê 3 món phở ngon ở Hà Nội"}
    ],
    response_format={"type": "json_object"},  # ép trả về JSON!
)

import json
data = json.loads(completion.choices[0].message.content)
print(data)
```

### 13.4 Điều Chỉnh Temperature & Tham Số

```python
completion = client.chat.completions.create(
    model="qwen/qwen-2.5-7b-instruct:free",
    messages=[{"role": "user", "content": "Viết truyện"}],
    temperature=0.9,      # 0.0-2.0: càng cao càng sáng tạo
    top_p=0.9,            # lọc từ có xác suất cao
    max_tokens=500,       # giới hạn độ dài
    frequency_penalty=0.5, # phạt lặp từ (-2.0 đến 2.0)
    presence_penalty=0.5,  # khuyến khích chủ đề mới
)
```

| Tham số | Thấp | Cao |
|---------|------|-----|
| `temperature` | Trả lời nhất quán, an toàn | Sáng tạo, bất ngờ |
| `max_tokens` | Câu trả lời ngắn | Câu trả lời dài |

---

## 14. Tích Hợp Với Công Cụ

### 14.1 LangChain

```bash
pip install langchain-openai
```

```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key="sk-or-v1-YOUR_KEY",
    model="qwen/qwen-2.5-7b-instruct:free",
)

response = llm.invoke("Xin chào!")
print(response.content)
```

### 14.2 LlamaIndex

```bash
pip install llama-index-llms-openai
```

```python
from llama_index.llms.openai import OpenAI

llm = OpenAI(
    api_base="https://openrouter.ai/api/v1",
    api_key="sk-or-v1-YOUR_KEY",
    model="qwen/qwen-2.5-7b-instruct:free",
)

print(llm.complete("Giải thích RAG là gì?"))
```

---

## 15. Best Practices

### 15.1 Bảo Mật API Key
```python
# ✅ ĐÚNG: dùng biến môi trường
import os
api_key = os.environ.get("OPENROUTER_API_KEY")

# ✅ ĐÚNG: dùng getpass
import getpass
api_key = getpass.getpass("API key: ")

# ❌ SAI: hardcode
api_key = "sk-or-v1-abc123..."  # ai cũng thấy!
```

```bash
# Set biến môi trường (Linux/Mac)
export OPENROUTER_API_KEY="sk-or-v1-YOUR_KEY"

# Dùng file .env
echo 'OPENROUTER_API_KEY=sk-or-v1-YOUR_KEY' >> .env
```

```python
# Đọc từ .env
from dotenv import load_dotenv
import os
load_dotenv()
api_key = os.getenv("OPENROUTER_API_KEY")
```

### 15.2 Luôn Set Referer
```python
extra_headers={
    "HTTP-Referer": "https://my-app.com",  # giúp OpenRouter thống kê
    "X-Title": "My App",
}
```

### 15.3 Giới Hạn Chi Tiêu
1. Vào https://openrouter.ai/settings/keys
2. Bấm vào key → **Edit**
3. Set **Credit limit** (VD: $5)
→ Key tự dừng khi hết giới hạn, tránh bị trừ tiền bất ngờ!

### 15.4 Cache Kết Quả
```python
import hashlib, json, os

CACHE_DIR = "./cache"
os.makedirs(CACHE_DIR, exist_ok=True)

def cached_chat(prompt, model="qwen/qwen-2.5-7b-instruct:free"):
    # Tạo key từ prompt
    key = hashlib.md5(f"{model}:{prompt}".encode()).hexdigest()
    path = f"{CACHE_DIR}/{key}.json"

    # Đọc cache nếu có
    if os.path.exists(path):
        print("📂 Dùng cache")
        return json.load(open(path))

    # Gọi API nếu chưa có
    print("🌐 Gọi API")
    result = chat(prompt, model)
    json.dump(result, open(path, "w"))
    return result
```

---

## 16. Tài Liệu Tham Khảo

- 🏠 Trang chủ: https://openrouter.ai
- 📚 Docs API: https://openrouter.ai/docs
- 🤖 Danh sách model: https://openrouter.ai/models
- 💰 Bảng giá: https://openrouter.ai/models (cột giá)
- 🔑 Quản lý key: https://openrouter.ai/settings/keys
- 💳 Nạp credit: https://openrouter.ai/settings/credits
- 📊 Thống kê dùng: https://openrouter.ai/activity
- 💬 Discord: https://discord.gg/openrouter

---

*📝 Viết bởi **Capybara** 🦫 cho **Tiểu Dươngg** — 2026-10-10*
