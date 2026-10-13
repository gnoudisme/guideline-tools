# Hướng dẫn ReactJS chi tiết

> Tài liệu học React từ component, hooks đến state management — giải thích chi tiết tư duy "React way", kèm ví dụ chạy được thật.

## Mục lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Cài đặt & tạo project](#2-cài-đặt--tạo-project)
3. [Tư duy React](#3-tư-duy-react)
4. [JSX](#4-jsx)
5. [Component](#5-component)
6. [Props](#6-props)
7. [State và useState](#7-state-và-usestate)
8. [Xử lý sự kiện](#8-xử-lý-sự-kiện)
9. [Render danh sách & key](#9-render-danh-sách--key)
10. [Conditional rendering](#10-conditional-rendering)
11. [Form và controlled component](#11-form-và-controlled-component)
12. [useEffect](#12-useeffect)
13. [useRef](#13-useref)
14. [useMemo và useCallback](#14-usememo-và-usecallback)
15. [useContext](#15-usecontext)
16. [useReducer](#16-usereducer)
17. [Custom hooks](#17-custom-hooks)
18. [Lifting state & composition](#18-lifting-state--composition)
19. [Gọi API](#19-gọi-api)
20. [React Router](#20-react-router)
21. [State management](#21-state-management)
22. [Best practices](#22-best-practices)

---

## 1. Giới thiệu

**React** là thư viện JavaScript do Facebook (Meta) phát triển (2013, bởi Jordan Walke), dùng để xây dựng **giao diện người dùng** (UI), đặc biệt là SPA (Single Page Application).

**Ý tưởng cốt lõi:**
- **Component-based**: UI là tập hợp các component độc lập, tái sử dụng được
- **Declarative**: mô tả UI "trông như thế nào" theo state, React tự lo việc cập nhật DOM
- **Virtual DOM**: React so sánh (diff) cây UI ảo với cây cũ, chỉ cập nhật phần DOM thật sự thay đổi → nhanh

**So sánh tư duy:**
```javascript
// Imperative (jQuery): ra lệnh từng bước
$("#btn").click(() => {
    $("#text").text("Đã bấm");  // tự thao tác DOM
    $("#text").addClass("active");
});

// Declarative (React): mô tả theo state
const [daBam, setDaBam] = useState(false);
return <p className={daBam ? "active" : ""} onClick={() => setDaBam(true)}>
    {daBam ? "Đã bấm" : "Chưa bấm"}
</p>;
// React tự cập nhật DOM khi daBam đổi — bạn không đụng DOM trực tiếp
```

---

## 2. Cài đặt & tạo project

```bash
# Cách hiện đại nhất: Vite (nhanh, nhẹ)
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev

# Hoặc với TypeScript
npm create vite@latest my-app -- --template react-ts
```

Cấu trúc:
```
my-app/
  index.html
  src/
    main.jsx       # điểm vào
    App.jsx        # component gốc
    index.css
```

---

## 3. Tư duy React

Trước khi code, nắm 3 tư duy này:

1. **UI = f(state)**: giao diện là hàm của state. State đổi → UI tự vẽ lại. Đừng nghĩ "làm sao cập nhật DOM", hãy nghĩ "state nào mô tả đúng tình huống này".

2. **Dữ liệu chảy một chiều** (one-way data flow): cha truyền xuống con qua props. Con muốn "nói" với cha → cha truyền hàm callback xuống.

3. **Chia component theo trách nhiệm**, không theo kỹ thuật: một component = một việc (hiển thị danh sách, form nhập, nút bấm...).

---

## 4. JSX

JSX là cú pháp mở rộng cho phép viết HTML trong JavaScript:

```jsx
const el = <h1>Xin chào!</h1>;
const el2 = (
    <div className="box">  {/* class → className (class là từ khóa JS) */}
        <p>Nội dung</p>
    </div>
);
```

**Quy tắc JSX:**
- Phải có **một thẻ cha duy nhất** (dùng `<div>` hoặc fragment `<>...</>`)
- Mọi thẻ phải **đóng**: `<img />`, `<br />` (self-closing)
- `class` → `className`, `for` → `htmlFor` (tránh trùng từ khóa JS)
- Nhúng biểu thức JS bằng `{}`:
```jsx
const ten = "Dương";
const tuoi = 20;
<p>Chào {ten}, năm sau bạn {tuoi + 1} tuổi</p>
<p>{tuoi >= 18 ? "Người lớn" : "Trẻ em"}</p>
```

**JSX không phải HTML trong JS** — nó được biên dịch thành `React.createElement()` calls. Hiểu vậy để khỏi bất ngờ khi debug.

---

## 5. Component

Component là hàm trả về JSX (function component — cách hiện đại, dùng hooks):

```jsx
// Định nghĩa
function Chao(props) {
    return <h1>Xin chào, {props.ten}!</h1>;
}

// Sử dụng (như thẻ HTML tự chế!)
function App() {
    return (
        <div>
            <Chao ten="Dương" />
            <Chao ten="Lan" />
        </div>
    );
}
```

**Quy tắc đặt tên:** component phải viết hoa chữ đầu (`Chao`, không phải `chao`) — React phân biệt component tự chế với thẻ HTML thường bằng chữ hoa.

---

## 6. Props

Props = tham số truyền từ cha xuống con. **Read-only**: con không được sửa props!

```jsx
function SinhVien({ ten, tuoi = 18, diem }) {  // destructuring + default
    return <p>{ten}, {tuoi} tuổi, điểm {diem}</p>;
}

<SinhVien ten="Dương" tuoi={20} diem={8.5} />
```

**Children prop** — nội dung lồng bên trong thẻ:
```jsx
function Khung({ children }) {
    return <div className="khung">{children}</div>;
}

<Khung>
    <p>Nội dung bên trong</p>  {/* = children */}
</Khung>
```

---

## 7. State và useState

State = dữ liệu nội bộ của component, khi đổi → component **render lại**.

```jsx
import { useState } from "react";

function Dem() {
    const [dem, setDem] = useState(0);  // [giá trị, hàm cập nhật] = useState(giá trị đầu)

    return (
        <button onClick={() => setDem(dem + 1)}>
            Đã bấm {dem} lần
        </button>
    );
}
```

**Những điều PHẢI nhớ về setState:**

1. **Bất đồng bộ**: gọi `setDem` không đổi ngay, React gom lại rồi render một lần
```jsx
setDem(dem + 1);
console.log(dem);  // vẫn giá trị CŨ!
```

2. **Dùng updater function khi giá trị mới phụ thuộc giá trị cũ:**
```jsx
// Sai (có thể mất update khi gọi dồn):
setDem(dem + 1); setDem(dem + 1);  // chỉ tăng 1!

// Đúng:
setDem(d => d + 1); setDem(d => d + 1);  // tăng 2
```

3. **Không mutate trực tiếp**, luôn tạo mới:
```jsx
const [ds, setDs] = useState([1, 2, 3]);
// setDs(ds.push(4));  // SAI: push sửa mảng gốc, React không phát hiện đổi!
setDs([...ds, 4]);     // ĐÚNG: tạo mảng mới

const [sv, setSv] = useState({ ten: "Dương", tuoi: 20 });
// sv.tuoi = 21;  // SAI
setSv({ ...sv, tuoi: 21 });  // ĐÚNG: spread tạo object mới
```

**Tại sao?** React so sánh state cũ/mới bằng `Object.is` (so sánh nông). Mutate trực tiếp → reference không đổi → React tưởng không có gì đổi → không render lại.

---

## 8. Xử lý sự kiện

```jsx
function Nut() {
    function bam() {
        alert("Đã bấm!");
    }
    // Truyền THAM CHIẾU hàm, không gọi: onClick={bam}, KHÔNG phải onClick={bam()}

    // Cần tham số: bọc trong arrow function
    function chao(ten) { alert(`Chào ${ten}`); }

    return (
        <>
            <button onClick={bam}>Bấm</button>
            <button onClick={() => chao("Dương")}>Chào Dương</button>
            <button onClick={(e) => {
                e.preventDefault();  // chặn hành vi mặc định
                console.log(e.target);  // element được bấm
            }}>Log</button>
        </>
    );
}
```

**Tên sự kiện trong React:** camelCase (`onClick`, `onChange`, `onSubmit`), không phải `onclick` thường.

---

## 9. Render danh sách & key

```jsx
const ds = ["Táo", "Cam", "Chuối"];

<ul>
    {ds.map((traiCay, i) => (
        <li key={i}>{traiCay}</li>
    ))}
</ul>
```

**`key` là gì và tại sao bắt buộc?** Khi list đổi (thêm/xóa/sắp xếp), React dùng `key` để biết phần tử nào là phần tử nào → chỉ cập nhật đúng chỗ, không vẽ lại cả list. **Dùng id ổn định, đừng dùng index** nếu list có thể đổi thứ tự (dùng index khi reorder sẽ gây bug hiển thị sai, state lẫn lộn).

```jsx
// Tốt: id từ data
{users.map(u => <li key={u.id}>{u.ten}</li>)}
```

---

## 10. Conditional rendering

```jsx
function TrangThai({ dangNhap }) {
    // 1. if/else thường (ngoài return)
    // 2. Toán tử ba ngôi
    return <p>{dangNhap ? "Xin chào!" : "Hãy đăng nhập"}</p>;

    // 3. && cho "có thì hiện, không thì thôi"
    // {dangNhap && <button>Đăng xuất</button>}

    // 4. Tách component con cho logic phức tạp
}
```

**Bẫy `&&`:** `{dem && <p>...</p>}` — nếu `dem = 0`, React render ra số `0`! Vì `0 && ...` = `0` (falsy nhưng vẫn là giá trị render được). Sửa: `{dem > 0 && <p>...</p>}`.

---

## 11. Form và controlled component

Trong React, form input nên là **controlled**: giá trị input nằm trong state.

```jsx
function Form() {
    const [ten, setTen] = useState("");
    const [tuoi, setTuoi] = useState("");

    function gui(e) {
        e.preventDefault();  // chặn reload trang!
        alert(`Chào ${ten}, ${tuoi} tuổi`);
    }

    return (
        <form onSubmit={gui}>
            <input
                value={ten}
                onChange={e => setTen(e.target.value)}  // mỗi phím gõ → state đổi → render lại
                placeholder="Tên"
            />
            <input
                type="number"
                value={tuoi}
                onChange={e => setTuoi(e.target.value)}
                placeholder="Tuổi"
            />
            <button type="submit">Gửi</button>
        </form>
    );
}
```

**Controlled vs uncontrolled:** controlled = React nắm giá trị (qua state). Uncontrolled = DOM nắm (dùng `ref` để đọc khi cần). Controlled phổ biến hơn vì dễ validate realtime.

**Form nhiều field gọn:**
```jsx
const [form, setForm] = useState({ ten: "", tuoi: "" });
function doi(e) {
    setForm({ ...form, [e.target.name]: e.target.value });  // computed key
}
<input name="ten" value={form.ten} onChange={doi} />
<input name="tuoi" value={form.tuoi} onChange={doi} />
```

---

## 12. useEffect

`useEffect` chạy **side effect** sau khi render: gọi API, đăng ký listener, timer...

```jsx
import { useState, useEffect } from "react";

function Users() {
    const [users, setUsers] = useState([]);

    useEffect(() => {
        // Chạy SAU lần render đầu tiên
        fetch("https://api.example.com/users")
            .then(r => r.json())
            .then(setUsers);
    }, []);  // [] = chỉ chạy 1 lần (như componentDidMount)

    return <ul>{users.map(u => <li key={u.id}>{u.ten}</li>)}</ul>;
}
```

**Dependency array — phần quan trọng nhất:**

```jsx
useEffect(() => {
    console.log(`Đếm = ${dem}`);
}, [dem]);  // chạy lại mỗi khi `dem` đổi

useEffect(() => {
    // không có array: chạy sau MỌI lần render (hiếm khi cần)
});

useEffect(() => {
    // []: chạy 1 lần duy nhất
});
```

**Cleanup function** — dọn dẹp khi component unmount hoặc trước khi effect chạy lại:
```jsx
useEffect(() => {
    const timer = setInterval(() => console.log("tick"), 1000);
    return () => clearInterval(timer);  // dọn timer, tránh rò rỉ!
}, []);

useEffect(() => {
    const sub = api.subscribe(...);
    return () => sub.unsubscribe();
}, []);
```

**Bẫy phổ biến:**
1. Quên dependency → effect dùng giá trị cũ (stale closure)
2. Thiếu cleanup → rò rỉ timer/listener, setState sau unmount
3. `useEffect` async trực tiếp: `useEffect(async () => ...)` — SAI (effect phải trả về cleanup hoặc undefined, không phải Promise). Đúng:
```jsx
useEffect(() => {
    async function lay() {
        const data = await fetch(...).then(r => r.json());
        setData(data);
    }
    lay();
}, []);
```

---

## 13. useRef

`useRef` giữ giá trị **không gây render lại** khi đổi — dùng cho DOM node, timer id, giá trị "nháp".

```jsx
import { useRef } from "react";

function Form() {
    const inputRef = useRef(null);  // trỏ tới DOM node

    function focus() {
        inputRef.current.focus();  // đụng DOM trực tiếp (hiếm khi cần)
    }

    return (
        <>
            <input ref={inputRef} />
            <button onClick={focus}>Focus</button>
        </>
    );
}
```

**`useRef` vs `useState`:** cần render lại khi đổi → `useState`. Chỉ cần "nhớ" giá trị qua các lần render → `useRef`.

---

## 14. useMemo và useCallback

Tối ưu hiệu năng: tránh tính toán/render lại không cần thiết.

```jsx
import { useMemo, useCallback } from "react";

// useMemo: nhớ KẾT QUẢ tính toán đắt
const dsLoc = useMemo(() => {
    return ds.filter(x => x.ten.includes(tuKhoa));  // chỉ tính lại khi ds/tuKhoa đổi
}, [ds, tuKhoa]);

// useCallback: nhớ HÀM (để không tạo hàm mới mỗi render → con không render lại vô ích)
const xuLy = useCallback((id) => {
    console.log(id);
}, []);  // [] = hàm không bao giờ đổi
```

**Khi nào cần?** Đừng tối ưu sớm! Chỉ dùng khi:
- Tính toán thật sự đắt (lọc/sắp xếp mảng lớn)
- Truyền callback xuống component con đã `memo` hóa

```jsx
const NutNho = React.memo(function NutNho({ onClick }) {
    // chỉ render lại khi onClick đổi (so sánh nông)
    return <button onClick={onClick}>Bấm</button>;
});
```

---

## 15. useContext

Truyền dữ liệu qua nhiều tầng component mà không cần prop drilling (truyền tay từng tầng).

```jsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext();

function App() {
    const [theme, setTheme] = useState("sang");
    return (
        <ThemeContext.Provider value={{ theme, setTheme }}>
            <TrangChu />
        </ThemeContext.Provider>
    );
}

function NutDoiTheme() {
    const { theme, setTheme } = useContext(ThemeContext);  // lấy trực tiếp!
    return <button onClick={() => setTheme(theme === "sang" ? "toi" : "sang")}>
        Đang: {theme}
    </button>;
}
```

**Khi nào dùng Context?** Theme, ngôn ngữ, user đăng nhập — dữ liệu "toàn cục", nhiều component cần. **Đừng dùng cho mọi state** — Context không tối ưu cho state đổi liên tục (mỗi lần đổi, mọi consumer render lại).

---

## 16. useReducer

Quản lý state phức tạp với nhiều action — như Redux thu nhỏ.

```jsx
import { useReducer } from "react";

function reducer(state, action) {
    switch (action.type) {
        case "tang": return { dem: state.dem + 1 };
        case "giam": return { dem: state.dem - 1 };
        case "dat": return { dem: action.giaTri };
        default: return state;
    }
}

function Dem() {
    const [state, dispatch] = useReducer(reducer, { dem: 0 });
    return (
        <>
            <p>{state.dem}</p>
            <button onClick={() => dispatch({ type: "tang" })}>+</button>
            <button onClick={() => dispatch({ type: "giam" })}>-</button>
            <button onClick={() => dispatch({ type: "dat", giaTri: 100 })}>Đặt 100</button>
        </>
    );
}
```

**`useState` vs `useReducer`:** state đơn giản (số, chuỗi, boolean) → `useState`. State phức tạp (object nhiều field, logic chuyển đổi phức tạp, nhiều action) → `useReducer`.

---

## 17. Custom hooks

Tách logic dùng lại được thành hàm riêng (bắt đầu bằng `use`):

```jsx
// useLocalStorage.js — hook tự viết
import { useState, useEffect } from "react";

export function useLocalStorage(key, giaTriDau) {
    const [val, setVal] = useState(() => {
        const luu = localStorage.getItem(key);
        return luu ? JSON.parse(luu) : giaTriDau;
    });

    useEffect(() => {
        localStorage.setItem(key, JSON.stringify(val));
    }, [key, val]);

    return [val, setVal];
}

// Dùng:
function App() {
    const [ten, setTen] = useLocalStorage("ten", "");
    return <input value={ten} onChange={e => setTen(e.target.value)} />;
}
```

**Quy tắc hooks:**
1. Chỉ gọi ở **top level** của component/hook (không trong if, for, hàm lồng)
2. Chỉ gọi trong **function component** hoặc **custom hook**
3. Tên custom hook phải bắt đầu bằng `use`

ESLint plugin `eslint-plugin-react-hooks` tự bắt vi phạm 2 quy tắc đầu.

---

## 18. Lifting state & composition

### Lifting state up — nâng state lên cha chung

Khi 2 component anh em cần dùng chung state → đặt state ở component cha gần nhất, truyền xuống qua props.

```jsx
function App() {
    const [nhietDo, setNhietDo] = useState(25);  // state chung
    return (
        <>
            <HienThi nhietDo={nhietDo} />
            <DieuChinh nhietDo={nhietDo} setNhietDo={setNhietDo} />
        </>
    );
}
```

### Composition — ghép component linh hoạt

Thay vì kế thừa (React không dùng kế thừa), dùng composition:

```jsx
// Thay vì: class NutDo extends Nut (KHÔNG làm vậy)
// Dùng: truyền component qua props/children
function Khung({ tieuDe, children }) {
    return (
        <div className="khung">
            <h2>{tieuDe}</h2>
            {children}
        </div>
    );
}

<Khung tieuDe="Thông báo">
    <p>Nội dung bất kỳ</p>
    <button>OK</button>
</Khung>
```

---

## 19. Gọi API

Pattern chuẩn với loading/error state:

```jsx
function DanhSachUser() {
    const [users, setUsers] = useState([]);
    const [dangTai, setDangTai] = useState(true);
    const [loi, setLoi] = useState(null);

    useEffect(() => {
        let huy = false;  // flag tránh setState sau unmount
        async function tai() {
            try {
                const res = await fetch("https://api.example.com/users");
                if (!res.ok) throw new Error(`HTTP ${res.status}`);
                const data = await res.json();
                if (!huy) setUsers(data);
            } catch (e) {
                if (!huy) setLoi(e.message);
            } finally {
                if (!huy) setDangTai(false);
            }
        }
        tai();
        return () => { huy = true; };
    }, []);

    if (dangTai) return <p>Đang tải...</p>;
    if (loi) return <p>Lỗi: {loi}</p>;
    return <ul>{users.map(u => <li key={u.id}>{u.ten}</li>)}</ul>;
}
```

Trong project thật, dùng thư viện như **React Query (TanStack Query)** hoặc **SWR** — tự lo caching, refetch, loading state.

---

## 20. React Router

Điều hướng giữa các "trang" trong SPA:

```bash
npm install react-router-dom
```

```jsx
import { BrowserRouter, Routes, Route, Link, useParams, useNavigate } from "react-router-dom";

function App() {
    return (
        <BrowserRouter>
            <nav>
                <Link to="/">Trang chủ</Link>
                <Link to="/gioi-thieu">Giới thiệu</Link>
            </nav>
            <Routes>
                <Route path="/" element={<TrangChu />} />
                <Route path="/gioi-thieu" element={<GioiThieu />} />
                <Route path="/user/:id" element={<UserChiTiet />} />
                <Route path="*" element={<p>404</p>} />
            </Routes>
        </BrowserRouter>
    );
}

function UserChiTiet() {
    const { id } = useParams();  // lấy :id từ URL
    const navigate = useNavigate();
    return <button onClick={() => navigate("/")}>Về trang chủ</button>;
}
```

---

## 21. State management

| Quy mô | Giải pháp |
|--------|-----------|
| 1 component | `useState` |
| Vài component liên quan | Lifting state + props |
| Toàn app, ít đổi | `useContext` |
| State phức tạp | `useReducer` + Context |
| App lớn, nhiều async | Zustand / Redux Toolkit / TanStack Query |

**Zustand** (gọn, hiện đại):
```bash
npm install zustand
```
```jsx
import { create } from "zustand";

const useStore = create((set) => ({
    dem: 0,
    tang: () => set((s) => ({ dem: s.dem + 1 })),
}));

function Nut() {
    const { dem, tang } = useStore();
    return <button onClick={tang}>{dem}</button>;
}
```

---

## 22. Best practices

1. **Một component, một việc** — component quá 200 dòng thì tách
2. **Không mutate state** — luôn tạo object/array mới
3. **Dùng updater function** `setX(prev => ...)` khi phụ thuộc giá trị cũ
4. **key ổn định** cho list (id từ data, không dùng index khi reorder được)
5. **Đặt logic gọi API vào custom hook** (`useUsers()`), component chỉ render
6. **Tách container/presentational**: component lo data riêng, component hiển thị riêng
7. **Dùng `useEffect` đúng**: đủ dependency, có cleanup
8. **Đừng tối ưu sớm** — `useMemo`/`useCallback` chỉ khi đo thấy chậm
9. **Đặt tên component rõ nghĩa**, file một component
10. **Xử lý đủ 3 trạng thái** khi gọi API: loading, error, success

---

*React = tư duy "UI là hàm của state". Nắm chắc hooks cơ bản (useState, useEffect, useContext) là đủ xây 90% app thực tế.*
