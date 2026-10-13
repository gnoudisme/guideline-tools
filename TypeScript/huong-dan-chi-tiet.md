# Hướng dẫn TypeScript chi tiết

> Tài liệu học TypeScript từ cơ bản đến type system nâng cao — giải thích chi tiết từng khái niệm. Nên học JavaScript trước, hoặc học song song.

## Mục lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Cài đặt](#2-cài-đặt)
3. [Chương trình đầu tiên](#3-chương-trình-đầu-tiên)
4. [Kiểu dữ liệu cơ bản](#4-kiểu-dữ-liệu-cơ-bản)
5. [Type annotations và Inference](#5-type-annotations-và-inference)
6. [Interface vs Type](#6-interface-vs-type)
7. [Union và Intersection Types](#7-union-và-intersection-types)
8. [Literal Types và Enums](#8-literal-types-và-enums)
9. [Functions](#9-functions)
10. [Array và Tuple](#10-array-và-tuple)
11. [Generics](#11-generics)
12. [Type Narrowing](#12-type-narrowing)
13. [Utility Types](#13-utility-types)
14. [Classes](#14-classes)
15. [Modules](#15-modules)
16. [Type Guards nâng cao](#16-type-guards-nâng-cao)
17. [Decorators](#17-decorators)
18. [tsconfig.json](#18-tsconfigjson)
19. [Best practices](#19-best-practices)

---

## 1. Giới thiệu

**TypeScript** do Microsoft phát triển (Anders Hejlsberg — cha đẻ C#), ra mắt 2012. Là **superset của JavaScript**: mọi code JS hợp lệ đều là code TS hợp lệ, cộng thêm **hệ thống kiểu tĩnh** (static typing).

**Vấn đề TypeScript giải quyết:**
```javascript
// JavaScript: lỗi chỉ phát hiện khi CHẠY
function cong(a, b) { return a + b; }
cong("1", 2);  // "12" (nối chuỗi!) — không báo lỗi, bug âm thầm
```

```typescript
// TypeScript: lỗi phát hiện khi VIẾT CODE (trong IDE)
function cong(a: number, b: number): number { return a + b; }
cong("1", 2);  // LỖI BIÊN DỊCH: Argument of type 'string' is not assignable to 'number'
```

**TypeScript biên dịch ra JavaScript** — trình duyệt/Node chỉ chạy JS. TS chỉ tồn tại lúc phát triển để bắt lỗi sớm.

**Khi nào dùng TS?** Project vừa và lớn, team nhiều người, code sống lâu. Project nhỏ, prototype nhanh → JS thuần cũng được.

---

## 2. Cài đặt

```bash
# Cài TypeScript compiler
npm install -g typescript

# Kiểm tra
tsc --version

# Khởi tạo project
mkdir my-app && cd my-app
npm init -y
npx tsc --init  # tạo tsconfig.json
```

---

## 3. Chương trình đầu tiên

`hello.ts`:
```typescript
const loiChao: string = "Xin chào TypeScript!";
console.log(loiChao);
```

```bash
tsc hello.ts    # biên dịch → sinh ra hello.js
node hello.js   # chạy bằng Node
```

**Quy trình:** viết `.ts` → `tsc` kiểm tra kiểu + biên dịch → ra `.js` → chạy như JS bình thường. Nếu có lỗi kiểu, `tsc` báo và (mặc định) vẫn sinh file JS — dùng `--noEmitOnError` để chặn.

---

## 4. Kiểu dữ liệu cơ bản

```typescript
let ten: string = "Dương";
let tuoi: number = 20;        // không phân biệt int/float, tất cả là number
let dau: boolean = true;
let giDo: any = "gì cũng được";  // any: tắt kiểm tra kiểu (TRÁNH DÙNG!)
let khong: undefined = undefined;
let rong: null = null;

let soVoHan: unknown = 42;   // unknown: như any nhưng an toàn hơn (phải kiểm tra trước khi dùng)
let khongBaoGio: never = (() => { throw new Error(); })();  // never: không bao giờ có giá trị
```

**`any` vs `unknown`:**
- `any`: cho làm gì cũng được, không kiểm tra → mất hết lợi ích của TS
- `unknown`: cũng "chưa biết kiểu", nhưng bắt phải kiểm tra/narrow trước khi dùng → an toàn

```typescript
let a: any = "hello";
a.toFixed(2);  // OK lúc biên dịch, CRASH lúc chạy (string không có toFixed)

let u: unknown = "hello";
// u.toFixed(2);  // LỖI BIÊN DỊCH: phải kiểm tra kiểu trước
if (typeof u === "string") {
    console.log(u.toUpperCase());  // ok, đã narrow
}
```

**Quy tắc:** `any` là "kẻ thù", chỉ dùng khi bất đắc dĩ (lib JS cũ không có type). Mặc định dùng `unknown`.

---

## 5. Type annotations và Inference

```typescript
// Ghi rõ kiểu (annotation)
let x: number = 42;

// Type inference: TS tự suy luận, khỏi ghi cũng được
let y = 42;        // TS hiểu y: number
let z = "hello";   // TS hiểu z: string
// z = 123;        // LỖI: đã suy là string

// Khi nào CẦN ghi rõ?
let duLieu: string[] = [];  // nếu không ghi, TS suy là never[] (mảng rỗng vô dụng)
function tinhThue(luong: number): number {  // tham số hàm BẮT BUỘC ghi kiểu
    return luong * 0.1;
}
```

---

## 6. Interface vs Type

Hai cách định nghĩa "hình dạng" của object:

```typescript
// Interface
interface SinhVien {
    ten: string;
    tuoi: number;
    diem?: number;        // ?: optional (có thể không có)
    readonly mssv: string; // readonly: chỉ đọc, không gán lại
}

// Type alias
type SinhVien2 = {
    ten: string;
    tuoi: number;
};
```

**Khác nhau khi nào?**
- `interface` có thể **merge** (khai báo 2 lần cùng tên → gộp lại) và **extends** (kế thừa). Dùng cho object/class shape.
- `type` linh hoạt hơn: dùng được cho union, tuple, primitive. Không merge được.

```typescript
interface Nguoi { ten: string; }
interface Nguoi { tuoi: number; }  // merge → { ten, tuoi }

interface SinhVien extends Nguoi {  // kế thừa
    mssv: string;
}

type ID = string | number;  // type làm được, interface không
```

**Khuyên dùng:** object shape → `interface`; union/complex → `type`.

---

## 7. Union và Intersection Types

### Union (`|`) — "hoặc"

```typescript
let id: string | number;
id = "abc";  // ok
id = 123;    // ok
// id = true;  // LỖI

function inRa(x: string | number) {
    // Chỉ được gọi method CHUNG của cả hai kiểu
    console.log(x.toString());  // ok (cả hai đều có)
    // console.log(x.toFixed(2));  // LỖI: string không có toFixed
}
```

### Intersection (`&`) — "và" (gộp)

```typescript
type Nguoi = { ten: string };
type NhanVien = { maNV: string };
type NhanVienVanPhong = Nguoi & NhanVien;  // phải có CẢ ten VÀ maNV

const nv: NhanVienVanPhong = { ten: "Dương", maNV: "NV01" };
```

### Discriminated Union — pattern cực mạnh

```typescript
type ThanhCong = { ok: true; duLieu: string };
type ThatBai = { ok: false; loi: string };
type KetQua = ThanhCong | ThatBai;

function xuLy(kq: KetQua) {
    if (kq.ok) {
        console.log(kq.duLieu);  // TS tự biết đây là ThanhCong!
    } else {
        console.log(kq.loi);     // và đây là ThatBai
    }
}
```

Trường `ok` là "discriminant" — TS dựa vào nó để narrow kiểu. Pattern này thay thế hoàn hảo cho việc ném exception trong nhiều trường hợp.

---

## 8. Literal Types và Enums

```typescript
// Literal: giá trị cụ thể cũng là một kiểu!
let huong: "bac" | "nam" | "dong" | "tay";
huong = "bac";   // ok
// huong = "tren";  // LỖI

// Enum
enum Mau {
    Do = "DO",
    Xanh = "XANH",
    Vang = "VANG",
}
let m: Mau = Mau.Do;

// Const enum: tối ưu hơn (không sinh object lúc chạy)
const enum TrangThai {
    Bat,
    Tat,
}
```

**Khi nào dùng gì?** Union của literal (`"bac" | "nam"`) nhẹ và hiện đại hơn enum trong hầu hết trường hợp. Enum chỉ cần khi muốn nhóm hằng số có ý nghĩa đặc biệt.

---

## 9. Functions

```typescript
// Đầy đủ annotation
function cong(a: number, b: number): number {
    return a + b;
}

// Arrow function
const nhan = (a: number, b: number): number => a * b;

// Tham số optional và default
function chao(ten: string, loiChao: string = "Xin chào"): string {
    return `${loiChao}, ${ten}!`;
}
chao("Dương");           // "Xin chào, Dương!"
chao("Dương", "Hi");     // "Hi, Dương!"

// Rest parameters
function tong(...so: number[]): number {
    return so.reduce((a, b) => a + b, 0);
}
tong(1, 2, 3, 4);  // 10

// Function type (kiểu của hàm)
let phepTinh: (a: number, b: number) => number;
phepTinh = cong;  // ok
```

### Overloads — một hàm nhiều chữ ký

```typescript
function xuLy(x: string): string;
function xuLy(x: number): number;
function xuLy(x: any): any {  // implementation (không gọi trực tiếp được)
    if (typeof x === "string") return x.toUpperCase();
    return x * 2;
}
xuLy("hello");  // string
xuLy(42);       // number
```

---

## 10. Array và Tuple

```typescript
// Array
let so: number[] = [1, 2, 3];
let ten: Array<string> = ["a", "b"];  // tương đương

// Tuple: mảng với kiểu cố định từng vị trí
let diem: [string, number] = ["Toán", 9];
// diem = [9, "Toán"];  // LỖI: sai thứ tự kiểu

// Tuple có tên (đọc dễ hơn)
type ToaDo = [x: number, y: number];
const p: ToaDo = [10, 20];
```

---

## 11. Generics

Viết code dùng được cho nhiều kiểu mà vẫn an toàn kiểu:

```typescript
// Hàm generic
function dauTien<T>(arr: T[]): T | undefined {
    return arr[0];
}
dauTien([1, 2, 3]);        // T = number, trả về number | undefined
dauTien(["a", "b"]);       // T = string

// Ràng buộc generic (constraint)
function daiNhat<T extends { length: number }>(a: T, b: T): T {
    return a.length > b.length ? a : b;
}
daiNhat("hello", "hi");    // ok (string có length)
daiNhat([1,2], [1]);       // ok (array có length)
// daiNhat(1, 2);          // LỖI: number không có length

// Generic với interface/class
interface Hop<T> {
    chua: T;
    lay(): T;
}
const hopSo: Hop<number> = { chua: 42, lay() { return this.chua; } };

// Giá trị mặc định cho generic
type ApiResponse<T = any> = {
    data: T;
    status: number;
};
```

---

## 12. Type Narrowing

Thu hẹp kiểu từ rộng thành hẹp — TS làm tự động trong nhiều trường hợp:

```typescript
function xuLy(x: string | number) {
    // typeof guard
    if (typeof x === "string") {
        console.log(x.toUpperCase());  // ở đây x: string
    } else {
        console.log(x.toFixed(2));     // ở đây x: number
    }
}

// instanceof guard (cho class)
class Cho { sua() {} }
class Meo { keu() {} }
function nghe(dv: Cho | Meo) {
    if (dv instanceof Cho) dv.sua();
    else dv.keu();
}

// truthiness guard
function inTen(ten?: string) {
    if (ten) {
        console.log(ten.toUpperCase());  // ten: string (không phải undefined)
    }
}
```

---

## 13. Utility Types

TypeScript có sẵn các "công cụ" biến đổi kiểu — cực kỳ hữu ích:

```typescript
interface User {
    id: number;
    ten: string;
    email: string;
    tuoi?: number;
}

// Partial: tất cả thành optional
type UserUpdate = Partial<User>;  // { id?: number; ten?: string; ... }

// Required: tất cả thành bắt buộc
type UserDayDu = Required<User>;

// Pick: chỉ lấy một số field
type UserTomTat = Pick<User, "id" | "ten">;  // { id: number; ten: string }

// Omit: bỏ một số field
type UserMoi = Omit<User, "id">;  // không có id (để DB tự sinh)

// Record: dict với key kiểu K, value kiểu V
type DiemSo = Record<string, number>;  // { [key: string]: number }
const diem: DiemSo = { "Toán": 9, "Văn": 8 };

// ReturnType: lấy kiểu trả về của hàm
function taoUser(): User { ... }
type KqTao = ReturnType<typeof taoUser>;  // User
```

---

## 14. Classes

```typescript
class Nguoi {
    // Property declarations
    private biMat: string = "secret";  // chỉ trong class
    protected giaDinh: string = "";    // class + class con
    readonly id: number;               // chỉ gán 1 lần

    constructor(
        public ten: string,   // parameter properties: vừa khai báo vừa gán!
        tuoi: number,
        id: number
    ) {
        this.id = id;
    }

    chao(): string {
        return `Chào, tôi là ${this.ten}`;
    }
}

class SinhVien extends Nguoi {
    constructor(ten: string, public mssv: string) {
        super(ten, 20, 1);  // gọi constructor cha
    }

    override chao(): string {  // override: ghi đè (TS 4.3+)
        return super.chao() + `, MSSV ${this.mssv}`;
    }
}

// Abstract class: không tạo object trực tiếp được
abstract class Hinh {
    abstract dienTich(): number;  // bắt class con implement
}
```

---

## 15. Modules

```typescript
// utils.ts
export function cong(a: number, b: number): number {
    return a + b;
}
export const PI = 3.14;
export default class MayTinh { ... }

// main.ts
import { cong, PI } from "./utils";
import MayTinh from "./utils";
import * as U from "./utils";  // U.cong(...)
```

---

## 16. Type Guards nâng cao

Tự định nghĩa hàm kiểm tra kiểu:

```typescript
// Type predicate: x is string
function laChuoi(x: unknown): x is string {
    return typeof x === "string";
}

function xuLy(x: unknown) {
    if (laChuoi(x)) {
        console.log(x.toUpperCase());  // x: string
    }
}

// Exhaustiveness checking với never
type Hinh = "vuong" | "tron" | "tamgiac";
function dienTich(h: Hinh): number {
    switch (h) {
        case "vuong": return 1;
        case "tron": return 2;
        case "tamgiac": return 3;
        default:
            const kiemTra: never = h;  // nếu thêm variant mà quên case → LỖI ở đây
            throw new Error("Chưa xử lý");
    }
}
```

---

## 17. Decorators

(Thử nghiệm, cần bật `experimentalDecorators` trong tsconfig)

```typescript
function Log(target: any, key: string, descriptor: PropertyDescriptor) {
    const goc = descriptor.value;
    descriptor.value = function (...args: any[]) {
        console.log(`Gọi ${key} với`, args);
        return goc.apply(this, args);
    };
}

class TinhToan {
    @Log
    cong(a: number, b: number) {
        return a + b;
    }
}
```

---

## 18. tsconfig.json

File cấu hình quan trọng nhất — nên hiểu các option chính:

```json
{
    "compilerOptions": {
        "target": "ES2020",        // JS đầu ra dùng chuẩn nào
        "module": "commonjs",      // hệ module đầu ra
        "strict": true,            // BẬT strict mode (KHUYẾN NGHỊ!)
        "outDir": "./dist",        // thư mục chứa JS đầu ra
        "rootDir": "./src",        // thư mục chứa TS nguồn
        "esModuleInterop": true,   // import lib CommonJS dễ hơn
        "skipLibCheck": true,      // bỏ qua kiểm tra file .d.ts (build nhanh)
        "forceConsistentCasingInFileNames": true,
        "noEmitOnError": true      // có lỗi kiểu thì không sinh JS
    },
    "include": ["src/**/*"],
    "exclude": ["node_modules"]
}
```

**`"strict": true` bật 4 kiểm tra nghiêm ngặt:**
- `strictNullChecks`: `null`/`undefined` không gán được cho kiểu khác
- `noImplicitAny`: cấm `any` ngầm định
- `strictFunctionTypes`, `strictPropertyInitialization`, v.v.

Luôn bật `strict` cho project mới!

---

## 19. Best practices

1. **Bật `"strict": true`** — không thỏa hiệp
2. **Tránh `any`**, dùng `unknown` + narrowing khi chưa biết kiểu
3. **Ưu tiên `interface`** cho object shape, `type` cho union/complex
4. **Dùng union literal** (`"a" | "b"`) thay vì enum khi có thể
5. **Tận dụng type inference** — đừng ghi kiểu thừa (`let x: number = 5` → `let x = 5`)
6. **Dùng Utility Types** (`Partial`, `Pick`, `Omit`) thay vì định nghĩa lại
7. **Discriminated union** cho kết quả thành công/thất bại thay vì throw lung tung
8. **Viết `.d.ts`** khi dùng lib JS không có type (hoặc tìm `@types/...` trên npm)
9. **Dùng `readonly`** cho dữ liệu không đổi
10. **Chạy `tsc --noEmit`** trong CI để bắt lỗi kiểu trước khi merge

---

*TypeScript = JavaScript + kỷ luật. Học TS là học cách để JS không còn "bất ngờ" nữa.*
