# Hướng dẫn JavaScript chi tiết

> Tài liệu học JavaScript từ cơ bản đến async/await, closure, prototype — giải thích chi tiết từng khái niệm, đặc biệt những "bẫy" nổi tiếng của JS.

## Mục lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Chạy JavaScript ở đâu](#2-chạy-javascript-ở-đâu)
3. [Biến: var, let, const](#3-biến-var-let-const)
4. [Kiểu dữ liệu](#4-kiểu-dữ-liệu)
5. [Toán tử và ép kiểu](#5-toán-tử-và-ép-kiểu)
6. [Điều kiện và vòng lặp](#6-điều-kiện-và-vòng-lặp)
7. [Hàm](#7-hàm)
8. [Arrow function](#8-arrow-function)
9. [Scope và Hoisting](#9-scope-và-hoisting)
10. [Closure](#10-closure)
11. [Object](#11-object)
12. [Array](#12-array)
13. [Destructuring](#13-destructuring)
14. [Spread và Rest](#14-spread-và-rest)
15. [String methods](#15-string-methods)
16. [this](#16-this)
17. [Prototype và Class](#17-prototype-và-class)
18. [Async: callback, promise, async/await](#18-async-callback-promise-asyncawait)
19. [Fetch API](#19-fetch-api)
20. [Module](#20-module)
21. [JSON](#21-json)
22. [Error handling](#22-error-handling)
23. [Các bẫy nổi tiếng](#23-các-bẫy-nổi-tiếng)
24. [Best practices](#24-best-practices)

---

## 1. Giới thiệu

**JavaScript** do Brendan Eich tạo ra năm 1995 trong **10 ngày** tại Netscape. Ban đầu chỉ để làm web tương tác, giờ chạy khắp nơi: trình duyệt, server (Node.js), mobile (React Native), desktop (Electron).

**Đặc điểm:**
- **Thông dịch, động kiểu**: không cần biên dịch, biến không cần khai báo kiểu
- **Đơn luồng + bất đồng bộ**: event loop xử lý async cực tốt
- **First-class function**: hàm là giá trị, truyền được như biến
- **Prototype-based OOP**: khác class-based của Java/C++

**Lưu ý tên gọi:** JavaScript **không liên quan gì đến Java** — tên do marketing thời đó. Đừng nhầm!

---

## 2. Chạy JavaScript ở đâu

**Trên trình duyệt:** mở DevTools (F12) → tab Console → gõ code chạy ngay.

**Với Node.js:**
```bash
node --version
node app.js        # chạy file
node               # REPL tương tác
```

**File HTML nhúng JS:**
```html
<script>
  console.log("Hello từ trình duyệt!");
</script>
<script src="app.js"></script>
```

---

## 3. Biến: var, let, const

```javascript
var cu = "cách cũ, đừng dùng";
let ten = "Dương";   // có thể gán lại
const PI = 3.14;     // không gán lại được
```

**Tại sao bỏ `var`?** `var` có 2 vấn đề lớn:
1. **Function scope** thay vì block scope → biến "rò" ra ngoài khối
2. **Hoisting kỳ lạ** → dùng trước khai báo được nhưng giá trị `undefined`

```javascript
if (true) {
    var a = 1;
    let b = 2;
}
console.log(a);  // 1 (rò ra ngoài! với var)
console.log(b);  // LỖI: b chỉ sống trong khối if
```

**Quy tắc:** mặc định dùng `const`, khi nào cần gán lại mới dùng `let`, **không bao giờ dùng `var`** trong code mới.

**Lưu ý `const` với object:** `const` chỉ khóa **gán lại**, không khóa **sửa bên trong**:
```javascript
const sv = { ten: "Dương" };
sv.ten = "Lan";  // OK! (sửa property)
sv.tuoi = 20;    // OK! (thêm property)
// sv = {};      // LỖI: gán lại object mới
```

---

## 4. Kiểu dữ liệu

**7 kiểu nguyên thủy (primitive):**
```javascript
let s = "hello";      // string
let n = 42;           // number (không phân int/float!)
let f = 3.14;         // number
let b = true;         // boolean
let u;                // undefined (khai báo chưa gán)
let nl = null;        // null (chủ đích "không có gì")
let sym = Symbol("id"); // symbol (ES6, key duy nhất)
let big = 123n;       // bigint (số nguyên cực lớn, hậu tố n)
```

**Kiểu tham chiếu:** `object`, `array`, `function`.

```javascript
typeof "hello"  // "string"
typeof 42       // "number"
typeof true     // "boolean"
typeof undefined // "undefined"
typeof null     // "object" (BUG lịch sử của JS! null không phải object)
typeof []       // "object"
typeof {}       // "object"
typeof function(){} // "function"
```

**`undefined` vs `null`:** `undefined` = "chưa có giá trị" (mặc định), `null` = "cố ý đặt là trống". Khi so sánh: `null == undefined` là `true`, nhưng `null === undefined` là `false`.

---

## 5. Toán tử và ép kiểu

### `==` vs `===` — bẫy kinh điển

```javascript
// == : so sánh LỎNG (tự ép kiểu trước khi so)
0 == "0"        // true (string "0" ép thành số 0)
0 == []         // true (!!)
"" == false     // true
null == undefined // true

// === : so sánh CHẶT (kiểm tra cả kiểu, không ép)
0 === "0"       // false
0 === []        // false
null === undefined // false
```

**Quy tắc sắt:** **luôn dùng `===` và `!==`**, không bao giờ dùng `==`/`!=` trừ khi bạn cố ý muốn ép kiểu (hiếm).

### Ép kiểu tường minh

```javascript
Number("42")    // 42
String(42)      // "42"
Boolean(0)      // false
parseInt("42px") // 42 (đọc được số đầu chuỗi)
parseFloat("3.14abc") // 3.14
```

### Toán tử logic trả về giá trị, không phải boolean!

```javascript
// || trả về toán hạng ĐẦU TIÊN truthy
let ten = "" || "Khách";  // "Khách" (dùng làm giá trị mặc định!)

// && trả về toán hạng đầu tiên falsy, hoặc cuối cùng nếu tất cả truthy
let x = 5 && 10;  // 10

// ?? (nullish coalescing, ES2020): chỉ dùng mặc định khi null/undefined
let a = 0 ?? "mặc định";  // 0 (0 không phải null nên giữ 0!)
let b = 0 || "mặc định";  // "mặc định" (0 là falsy!)
// Khác nhau: || coi 0,"",false là "trống", ?? chỉ coi null/undefined là "trống"

// ?. (optional chaining): truy cập an toàn
let user = null;
console.log(user?.ten);  // undefined (không crash!)
// thay vì: user && user.ten
```

---

## 6. Điều kiện và vòng lặp

```javascript
// if/else
let diem = 85;
if (diem >= 90) console.log("Xuất sắc");
else if (diem >= 70) console.log("Khá");
else console.log("Trung bình");

// Toán tử ba ngôi
let lon = a > b ? a : b;

// switch
switch (ngay) {
    case 1: console.log("CN"); break;
    case 2: console.log("T2"); break;
    default: console.log("khác");
}

// for
for (let i = 0; i < 5; i++) console.log(i);

// for...of (duyệt GIÁ TRỊ)
for (let x of [1, 2, 3]) console.log(x);

// for...in (duyệt KEY của object — cẩn thận với array!)
for (let k in {a: 1, b: 2}) console.log(k);  // "a", "b"

// while / do-while
let i = 0;
while (i < 5) { console.log(i); i++; }
```

**`for...of` vs `for...in`:** `for...of` duyệt giá trị (dùng cho array, string, Map, Set). `for...in` duyệt key (dùng cho object). **Đừng dùng `for...in` với array** — nó duyệt cả key kế thừa, thứ tự không đảm bảo.

---

## 7. Hàm

```javascript
// Function declaration (được hoist)
function cong(a, b) {
    return a + b;
}

// Function expression (không hoist)
const tru = function(a, b) {
    return a - b;
};

// Tham số mặc định
function chao(ten = "bạn") {
    return `Chào ${ten}`;
}

// Hàm là giá trị: truyền làm tham số (callback)
function xuLy(ds, fn) {
    const kq = [];
    for (let x of ds) kq.push(fn(x));
    return kq;
}
xuLy([1, 2, 3], x => x * 2);  // [2, 4, 6]
```

---

## 8. Arrow function

```javascript
// Dạng đầy đủ
const cong = (a, b) => {
    return a + b;
};

// 1 biểu thức: bỏ {} và return
const nhan = (a, b) => a * b;

// 1 tham số: bỏ ()
const gapDoi = x => x * 2;

// Không tham số: giữ ()
const chao = () => "hello";

// Trả về object: bọc trong ()
const taoSV = (ten) => ({ ten: ten });  // không có () sẽ hiểu {} là block!
```

**Khác biệt quan trọng với `function`:** arrow function **không có `this` riêng** — nó dùng `this` của scope bên ngoài (lexical this). Chi tiết ở mục 16.

---

## 9. Scope và Hoisting

### Scope chain

```javascript
let global = "toàn cục";

function ngoai() {
    let a = 1;
    function trong() {
        let b = 2;
        console.log(a, b, global);  // thấy hết: scope lồng nhau
    }
    trong();
    // console.log(b);  // LỖI: b chỉ sống trong hàm trong()
}
```

Biến được tìm từ scope trong ra ngoài. Hàm "nhớ" scope nơi nó được định nghĩa (không phải nơi được gọi) — đây là nền tảng của closure.

### Hoisting

```javascript
console.log(x);  // undefined (không lỗi!)
var x = 5;
// → JS "kéo" khai báo var x lên đầu, nhưng gán = 5 vẫn ở dưới

console.log(y);  // LỖI: Cannot access 'y' before initialization
let y = 5;
// let/const cũng hoist nhưng vào "temporal dead zone" — dùng trước là lỗi
```

---

## 10. Closure

Closure = hàm "nhớ" các biến ở scope bên ngoài ngay cả khi scope đó đã kết thúc.

```javascript
function taoDem() {
    let dem = 0;  // biến "riêng tư"
    return function() {
        dem++;
        return dem;
    };
}

const dem1 = taoDem();
console.log(dem1());  // 1
console.log(dem1());  // 2 (nhớ dem!)
console.log(dem1());  // 3

const dem2 = taoDem();  // bộ đếm ĐỘC LẬP
console.log(dem2());  // 1
```

**Ứng dụng thực tế:**
- Tạo biến private (trước khi có class field `#`)
- Factory function, currying
- Event handler giữ state
- Debounce/throttle

```javascript
// Debounce: chỉ chạy sau khi ngừng gọi một lúc
function debounce(fn, ms) {
    let timer;
    return function(...args) {
        clearTimeout(timer);
        timer = setTimeout(() => fn(...args), ms);
    };
}
const timKiem = debounce((q) => fetch(`/api?q=${q}`), 300);
```

---

## 11. Object

```javascript
const sv = {
    ten: "Dương",
    tuoi: 20,
    "que-quan": "Phú Thọ",  // key có gạch ngang phải dùng ngoặc kép
    chao() {                 // method viết gọn (ES6)
        return `Chào, tôi là ${this.ten}`;
    }
};

console.log(sv.ten);       // "Dương" (dot notation)
console.log(sv["ten"]);    // "Dương" (bracket notation)
console.log(sv["que-quan"]); // phải dùng bracket vì key có gạch ngang

sv.diem = 8.5;             // thêm property
delete sv.tuoi;            // xóa property

// Key động
let key = "ten";
console.log(sv[key]);  // "Dương"
```

### Object methods hay dùng

```javascript
Object.keys(sv)    // ["ten", "que-quan", ...] (mảng các key)
Object.values(sv)  // mảng các value
Object.entries(sv) // [["ten","Dương"], ...] (cặp key-value)
Object.assign({}, sv, { tuoi: 21 })  // gộp object (shallow copy)
```

---

## 12. Array

```javascript
let arr = [1, 2, 3];
arr.push(4);        // thêm cuối → [1,2,3,4]
arr.pop();          // lấy ra cuối → [1,2,3]
arr.unshift(0);     // thêm đầu → [0,1,2,3]
arr.shift();        // lấy ra đầu → [1,2,3]
console.log(arr.length);  // 3
console.log(arr.includes(2));  // true
console.log(arr.indexOf(3));   // 2
```

### Các method biến đổi (rất quan trọng!)

```javascript
let so = [1, 2, 3, 4, 5];

// map: biến đổi từng phần tử → mảng mới
so.map(x => x * 2);           // [2,4,6,8,10]

// filter: lọc → mảng mới
so.filter(x => x % 2 === 0);  // [2,4]

// reduce: gộp thành 1 giá trị
so.reduce((tong, x) => tong + x, 0);  // 15

// find / findIndex: tìm phần tử đầu thỏa mãn
so.find(x => x > 3);          // 4
so.findIndex(x => x > 3);     // 3

// some / every: kiểm tra
so.some(x => x > 4);          // true (có ít nhất 1)
so.every(x => x > 0);         // true (tất cả)

// forEach: duyệt (không trả về gì)
so.forEach(x => console.log(x));

// sort (CẨN THẬN: sort tại chỗ + mặc định sort theo CHUỖI!)
[10, 2, 30].sort();                    // [10, 2, 30] (sai! vì "10" < "2")
[10, 2, 30].sort((a, b) => a - b);     // [2, 10, 30] (đúng)

// slice: cắt (không sửa gốc) | splice: cắt + chèn (SỬA gốc!)
so.slice(1, 3);   // [2,3]
so.splice(1, 2);  // xóa 2 phần tử từ index 1, so còn [1,4,5]
```

**`map`/`filter`/`reduce` không sửa mảng gốc** — chúng trả về mảng/giá trị mới. Đây là phong cách lập trình hàm, code dễ đoán hơn.

---

## 13. Destructuring

"Bóc" giá trị từ object/array ra biến — gọn và đọc dễ.

```javascript
// Object
const sv = { ten: "Dương", tuoi: 20, diem: 8.5 };
const { ten, tuoi } = sv;           // ten="Dương", tuoi=20
const { ten: hoTen } = sv;          // đổi tên: hoTen="Dương"
const { lop = "A1" } = sv;          // giá trị mặc định nếu không có

// Array
const [a, b] = [1, 2];              // a=1, b=2
const [dau, ...conLai] = [1,2,3,4]; // dau=1, conLai=[2,3,4]

// Đổi giá trị không cần biến tạm
let x = 1, y = 2;
[x, y] = [y, x];  // x=2, y=1

// Trong tham số hàm (rất hay dùng với React props!)
function chao({ ten, tuoi }) {
    return `Chào ${ten}, ${tuoi} tuổi`;
}
chao(sv);
```

---

## 14. Spread và Rest

Cùng cú pháp `...` nhưng ngược nhau tùy vị trí:

```javascript
// SPREAD: "bung" ra
const a = [1, 2];
const b = [...a, 3, 4];       // [1,2,3,4] (copy mảng)
const c = [...a];             // copy nông (shallow copy)

const o1 = { x: 1 };
const o2 = { ...o1, y: 2 };   // {x:1, y:2} (gộp object)

// REST: "gom" lại (trong khai báo)
function tong(...so) {        // gom tham số thành mảng
    return so.reduce((a, b) => a + b, 0);
}
tong(1, 2, 3);  // 6

const [dau, ...duoi] = [1, 2, 3];  // dau=1, duoi=[2,3]
```

---

## 15. String methods

```javascript
let s = "  Hello World  ";
s.trim();              // "Hello World"
s.toLowerCase();       // "  hello world  "
s.toUpperCase();       // "  HELLO WORLD  "
s.includes("World");   // true
s.startsWith("  He");  // true
s.endsWith("  ");      // true
s.replace("World", "JS");  // thay thế (chỉ cái đầu tiên!)
s.replaceAll("l", "L");    // thay tất cả (ES2021)
s.split(" ");          // tách thành mảng
s.slice(2, 7);         // "Hello" (cắt từ 2 đến <7)
s.padStart(20, "0");   // đệm đầu cho đủ 20 ký tự

// Template literal (dấu backtick `): nội suy + nhiều dòng
let ten = "Dương";
let html = `
  <div>
    <p>Chào ${ten}!</p>
    <p>${2 + 3}</p>
  </div>
`;
```

---

## 16. `this`

`this` trong JS **phụ thuộc vào CÁCH GỌI**, không phải nơi định nghĩa (trừ arrow function).

```javascript
const nguoi = {
    ten: "Dương",
    chao() {
        console.log(`Chào, tôi là ${this.ten}`);  // this = nguoi
    }
};
nguoi.chao();  // "Chào, tôi là Dương"

const fn = nguoi.chao;
fn();  // "Chào, tôi là undefined" (this mất! gọi như hàm thường → this = undefined/global)
```

**4 quy tắc xác định `this`:**
1. `obj.method()` → `this = obj`
2. `fn()` → `this = undefined` (strict) hoặc `global`
3. `new Fn()` → `this` = object mới tạo
4. `fn.call(obj)` / `fn.apply(obj)` / `fn.bind(obj)` → `this = obj` (ép buộc)

```javascript
function chao() { console.log(this.ten); }
const a = { ten: "A" };
chao.call(a);              // "A"
const chaoA = chao.bind(a);
chaoA();                   // "A" (bind tạo hàm mới với this cố định)
```

**Arrow function:** không có `this` riêng, dùng `this` của scope ngoài:
```javascript
const obj = {
    ten: "Dương",
    dem: function() {
        setTimeout(() => {
            console.log(this.ten);  // "Dương" (arrow giữ this của dem)
        }, 100);
    }
    // Nếu dùng function() thường trong setTimeout, this sẽ mất!
};
```

---

## 17. Prototype và Class

### Prototype — cơ chế kế thừa gốc của JS

Mỗi object có một "nguyên mẫu" (prototype). Truy cập property không có → JS tìm tiếp trên prototype chain.

```javascript
function Cho(ten) {
    this.ten = ten;
}
Cho.prototype.sua = function() {  // method chung cho mọi Cho
    return `${this.ten}: Gâu!`;
};

const c = new Cho("Mực");
console.log(c.sua());  // "Mực: Gâu!"
```

### Class (ES6) — cú pháp gọn của prototype

```javascript
class DongVat {
    constructor(ten) {
        this.ten = ten;
    }
    keu() {
        return "...";
    }
    static loai() {  // static: gọi qua class, không cần object
        return "Động vật";
    }
}

class Cho extends DongVat {  // kế thừa
    constructor(ten) {
        super(ten);  // gọi constructor cha
    }
    keu() {  // ghi đè
        return "Gâu!";
    }
    #biMat = 123;  // private field (ES2022, dấu #)
}

const c = new Cho("Mực");
console.log(c.keu());  // "Gâu!"
console.log(DongVat.loai());  // "Động vật"
```

**Bản chất:** `class` chỉ là "đường syntactic" của prototype — bên dưới vẫn là prototype chain. Hiểu prototype giúp debug những lỗi kỳ lạ.

---

## 18. Async: callback, promise, async/await

JS đơn luồng nhưng xử lý bất đồng bộ (I/O, timer, network) qua **event loop**.

### Callback (cách cũ, dễ "callback hell")

```javascript
layUser(1, (user) => {
    layBaiViet(user.id, (baiviet) => {
        layBinhLuan(baiviet.id, (bl) => {
            console.log(bl);  // lồng nhau sâu hun hút!
        });
    });
});
```

### Promise (ES6)

```javascript
function doi(ms) {
    return new Promise((resolve, reject) => {
        setTimeout(() => resolve("xong!"), ms);
    });
}

doi(1000)
    .then(kq => { console.log(kq); return doi(500); })  // chuỗi
    .then(kq => console.log(kq))
    .catch(loi => console.log("Lỗi:", loi))  // bắt lỗi cả chuỗi
    .finally(() => console.log("luôn chạy"));

// Chạy song song
Promise.all([doi(1000), doi(500)]).then(kq => console.log(kq));  // đợi TẤT CẢ
Promise.race([doi(1000), doi(500)]).then(kq => console.log(kq)); // lấy cái XONG ĐẦU
```

**Promise có 3 trạng thái:** pending → fulfilled (resolve) hoặc rejected. Một khi đã settled thì không đổi nữa.

### async/await (ES2017) — cách hiện đại nhất

```javascript
async function main() {
    try {
        const a = await doi(1000);  // đợi xong mới đi tiếp, code như đồng bộ!
        console.log(a);
        const b = await doi(500);
        console.log(b);
    } catch (loi) {
        console.log("Lỗi:", loi);  // try/catch bắt lỗi async
    }
}
```

**Quy tắc:**
- `await` chỉ dùng trong hàm `async` (hoặc module top-level)
- Hàm `async` luôn trả về Promise
- `await` cho code "trông như đồng bộ" nhưng không block thread

---

## 19. Fetch API

```javascript
// GET
const res = await fetch("https://api.example.com/users");
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const data = await res.json();  // parse JSON
console.log(data);

// POST
const res2 = await fetch("https://api.example.com/users", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ ten: "Dương", tuoi: 20 }),
});
const moi = await res2.json();
```

**Lưu ý:** `fetch` chỉ reject khi lỗi mạng. Lỗi HTTP 404/500 vẫn resolve → phải tự kiểm tra `res.ok`.

---

## 20. Module

```javascript
// utils.js
export function cong(a, b) { return a + b; }
export const PI = 3.14;
export default function tru(a, b) { return a - b; }

// main.js
import tru, { cong, PI } from "./utils.js";
import * as U from "./utils.js";  // U.cong(...)
```

**ES Module vs CommonJS:**
- ES Module (`import`/`export`): chuẩn JS hiện đại, dùng ở trình duyệt + Node mới
- CommonJS (`require`/`module.exports`): cách cũ của Node
- Trong Node, file `.mjs` hoặc `"type": "module"` trong package.json → dùng ES Module

---

## 21. JSON

```javascript
const obj = { ten: "Dương", tuoi: 20 };
const chuoi = JSON.stringify(obj);  // object → chuỗi JSON
// '{"ten":"Dương","tuoi":20}'

const obj2 = JSON.parse(chuoi);     // chuỗi JSON → object

// JSON.stringify bỏ qua: function, undefined, Symbol
JSON.stringify({ a: 1, f: () => {} });  // '{"a":1}'
```

---

## 22. Error handling

```javascript
try {
    const data = JSON.parse("không phải json");
} catch (e) {
    console.log("Lỗi:", e.message);
} finally {
    console.log("Luôn chạy");
}

// Tự ném lỗi
function chia(a, b) {
    if (b === 0) throw new Error("Không chia được cho 0");
    return a / b;
}

// Tạo lỗi tùy chỉnh
class LoiNhapLieu extends Error {
    constructor(msg) {
        super(msg);
        this.name = "LoiNhapLieu";
    }
}
```

---

## 23. Các bẫy nổi tiếng

```javascript
// 1. == vs === (đã nói ở mục 5): luôn dùng ===

// 2. Cộng số với chuỗi
1 + "2"     // "12" (số thành chuỗi!)
"5" - 2     // 3 (trừ thì chuỗi thành số!)
"5" * "2"   // 10

// 3. Số thực không chính xác
0.1 + 0.2   // 0.30000000000000004 (!!)
0.1 + 0.2 === 0.3  // false
// Khắc phục: Math.round((0.1+0.2)*100)/100, hoặc dùng thư viện decimal

// 4. typeof null === "object" (bug lịch sử)

// 5. NaN không bằng chính nó!
NaN === NaN  // false
Number.isNaN(NaN)  // true (dùng cái này để kiểm tra)

// 6. Array rỗng + object
[] + []     // "" (chuỗi rỗng!)
[] + {}     // "[object Object]"
{} + []     // 0 (trong một số ngữ cảnh!)

// 7. var trong vòng lặp + closure
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}  // in 3, 3, 3 (không phải 0,1,2! vì var chung 1 biến)
// Sửa: dùng let (mỗi lần lặp có scope riêng) → in 0,1,2
```

---

## 24. Best practices

1. **Luôn dùng `===`**, không dùng `==`
2. **Dùng `const` mặc định**, `let` khi cần, **không dùng `var`**
3. **Dùng arrow function** cho callback ngắn
4. **Dùng `async/await`** thay vì chuỗi `.then()` dài
5. **Dùng optional chaining (`?.`)** và **nullish coalescing (`??`)** thay vì `&&` dài dòng
6. **Dùng destructuring** để code gọn
7. **Ưu tiên `map`/`filter`/`reduce`** thay vì vòng lặp `for` khi biến đổi mảng
8. **Xử lý lỗi `fetch`**: kiểm tra `res.ok`, bọc `try/catch` quanh `await`
9. **Đặt tên rõ nghĩa**, dùng camelCase cho biến/hàm
10. **Khi project lớn dần → chuyển sang TypeScript**

---

*JavaScript "dễ học, khó giỏi". Nắm chắc closure, `this`, prototype và async là bạn đã hơn 80% người viết JS rồi.*
