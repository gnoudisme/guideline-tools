# Hướng dẫn Rust chi tiết

> Tài liệu học Rust từ cơ bản đến ownership/borrowing — phần khó nhất nhưng cũng là linh hồn của Rust. Giải thích chi tiết từng khái niệm.

## Mục lục

1. [Giới thiệu](#1-giới-thiệu)
2. [Cài đặt](#2-cài-đặt)
3. [Chương trình đầu tiên & Cargo](#3-chương-trình-đầu-tiên--cargo)
4. [Biến và kiểu dữ liệu](#4-biến-và-kiểu-dữ-liệu)
5. [Hàm](#5-hàm)
6. [Điều kiện và vòng lặp](#6-điều-kiện-và-vòng-lặp)
7. [Ownership — linh hồn của Rust](#7-ownership--linh-hồn-của-rust)
8. [Borrowing và References](#8-borrowing-và-references)
9. [Slice](#9-slice)
10. [Struct](#10-struct)
11. [Enum và Pattern Matching](#11-enum-và-pattern-matching)
12. [Option và Result](#12-option-và-result)
13. [Collections](#13-collections)
14. [Error handling](#14-error-handling)
15. [Generics](#15-generics)
16. [Traits](#16-traits)
17. [Lifetime](#17-lifetime)
18. [Smart Pointers](#18-smart-pointers)
19. [Concurrency](#19-concurrency)
20. [Module và Crate](#20-module-và-crate)
21. [Testing](#21-testing)
22. [Best practices](#22-best-practices)

---

## 1. Giới thiệu

**Rust** là ngôn ngữ hệ thống được Graydon Hoaré khởi xướng năm 2006, Mozilla tài trợ từ 2009, bản 1.0 ra mắt 2015. Compiler đầu tiên viết bằng OCaml, sau đó tự viết lại bằng Rust (bootstrapping).

**Triết lý cốt lõi:** hiệu năng của C++ nhưng **an toàn bộ nhớ mà không cần garbage collector**. Rust đạt điều này bằng hệ thống **ownership** được kiểm tra ngay lúc biên dịch.

**Điểm đặc biệt:**
- **Memory safety**: không null pointer dereference, không data race, không use-after-free — compiler bắt hết
- **Zero-cost abstractions**: abstraction không tốn hiệu năng runtime
- **Fearless concurrency**: viết đa luồng an toàn
- Được bình chọn "ngôn ngữ được yêu thích nhất" trên Stack Overflow nhiều năm liên tiếp

**Ứng dụng:** Firefox components, Deno, Polkadot, Discord backend, Cloudflare, Linux kernel (driver thử nghiệm), GhostCircuit honeypot của chúng ta 🦀

---

## 2. Cài đặt

```bash
# Linux/macOS/Termux
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Kiểm tra
rustc --version
cargo --version
```

`rustup` là toolchain manager (như nvm của Node). `rustc` là compiler, `cargo` là build tool + package manager (như npm + Makefile gộp lại).

---

## 3. Chương trình đầu tiên & Cargo

```bash
cargo new hello
cd hello
cargo run
```

Cấu trúc project:
```
hello/
  Cargo.toml    # như package.json: tên, version, dependencies
  src/
    main.rs     # code chính
```

`src/main.rs`:
```rust
fn main() {
    println!("Xin chào, Rust!");
}
```

**Giải thích:**
- `fn main()`: hàm chính, không cần kiểu trả về (mặc định `()`)
- `println!`: đây là **macro** (có dấu `!`), không phải hàm. Macro chạy lúc biên dịch, sinh code. `println!` tự xuống dòng, `print!` thì không.
- Mọi câu lệnh kết thúc bằng `;`. Không có `;` ở cuối block = giá trị trả về (sẽ học ở mục hàm).

**Các lệnh cargo:**
```bash
cargo build          # biên dịch (debug, nhanh)
cargo build --release # biên dịch tối ưu (chạy nhanh, build lâu)
cargo run            # build + chạy
cargo check          # kiểm tra lỗi nhanh (không sinh binary)
cargo test           # chạy test
cargo fmt            # format code tự động
cargo clippy         # linter: gợi ý code tốt hơn
```

---

## 4. Biến và kiểu dữ liệu

### Biến: mặc định BẤT BIẾN (immutable)

Đây là điểm sốc đầu tiên với người từ ngôn ngữ khác:

```rust
let x = 5;
// x = 6;  // LỖI: cannot assign twice to immutable variable

let mut y = 5;  // mut = mutable (cho phép đổi)
y = 6;          // ok
```

**Tại sao mặc định immutable?** Để compiler dễ suy luận, tránh bug do đổi giá trị lung tung, và giúp concurrency an toàn. Khi bạn viết `mut`, bạn đang nói rõ "tôi chủ đích cho đổi" — code tự tài liệu hóa ý định.

### Shadowing — khai báo đè

```rust
let x = 5;
let x = x + 1;      // x mới = 6, x cũ bị "che"
let x = "hello";    // thậm chí đổi kiểu được!

// Khác với mut: shadowing tạo biến MỚI, mut là sửa biến CŨ
```

### Kiểu số

```rust
let a: i32 = -42;    // số nguyên 32-bit có dấu (mặc định cho int)
let b: u64 = 100;    // số nguyên 64-bit không dấu
let c: f64 = 3.14;   // số thực 64-bit (mặc định cho float)
let d = 42u8;        // hậu tố kiểu trực tiếp
```

| Kiểu | Kích thước | Kiểu | Kích thước |
|------|-----------|------|-----------|
| `i8/u8` | 8-bit | `f32` | 32-bit float |
| `i16/u16` | 16-bit | `f64` | 64-bit float |
| `i32/u32` | 32-bit | `isize/usize` | theo kiến trúc (dùng cho index) |
| `i64/u64` | 64-bit | `char` | 4 bytes (Unicode scalar!) |
| `i128/u128` | 128-bit | `bool` | 1 byte |

**`char` trong Rust là 4 bytes** (chứa được mọi ký tự Unicode như '🦀'), khác C chỉ 1 byte. Chuỗi UTF-8 dùng `String`/`&str`.

### Ép kiểu

```rust
let a: i32 = 42;
let b: i64 = a as i64;  // dùng `as`, tường minh
```

Rust **không tự ép kiểu ngầm** giữa số (trừ một số trường hợp). Tường minh để tránh bug mất mát dữ liệu.

---

## 5. Hàm

```rust
fn cong(a: i32, b: i32) -> i32 {
    a + b  // KHÔNG có ; → đây là giá trị trả về
}

fn in_chao(ten: &str) {
    println!("Chào {}", ten);  // không trả về gì = trả về ()
}

fn main() {
    let kq = cong(3, 4);
    println!("3 + 4 = {}", kq);
}
```

**Quy tắc:** biểu thức cuối không có `;` sẽ là giá trị trả về. Có `;` thì thành "statement" trả về `()` (unit type — kiểu "không có gì", giống `void`).

```rust
fn vi_du() -> i32 {
    let x = 5;
    x + 1   // trả về 6
    // x + 1;  // nếu có ; thì hàm trả về () → LỖI kiểu
}
```

---

## 6. Điều kiện và vòng lặp

### if — là biểu thức (trả về giá trị!)

```rust
let diem = 85;
let xep_loai = if diem >= 90 {
    "Xuất sắc"
} else if diem >= 70 {
    "Khá"
} else {
    "Trung bình"
};
// Không cần ngoặc tròn quanh điều kiện!
```

Vì `if` là biểu thức nên gán trực tiếp được — gọn hơn C++ nhiều.

### loop / while / for

```rust
// loop: lặp vô hạn (dùng break để thoát)
let mut dem = 0;
loop {
    dem += 1;
    if dem == 5 { break; }
}

// while
let mut i = 0;
while i < 5 {
    println!("{}", i);
    i += 1;
}

// for với range
for i in 0..5 {       // 0,1,2,3,4 (không bao gồm 5)
    println!("{}", i);
}
for i in 0..=5 {      // 0..5 (bao gồm 5)
    println!("{}", i);
}

// break trả về giá trị từ loop!
let kq = loop {
    dem += 1;
    if dem == 10 { break dem * 2; }  // break với giá trị
};
```

---

## 7. Ownership — linh hồn của Rust

Đây là phần quan trọng nhất. Hiểu ownership là hiểu Rust.

### Ba quy tắc vàng

1. **Mỗi giá trị có đúng một owner** (biến sở hữu nó)
2. **Khi owner ra khỏi scope, giá trị bị hủy** (drop)
3. **Chỉ có một owner tại một thời điểm**

```rust
{
    let s = String::from("hello");  // s là owner, cấp phát trên heap
    println!("{}", s);
}  // s ra khỏi scope → bộ nhớ tự giải phóng. Không cần free(), không GC!
```

### Move — chuyển quyền sở hữu

```rust
let s1 = String::from("hello");
let s2 = s1;  // QUYỀN SỞ HỮU chuyển từ s1 sang s2 (move)
// println!("{}", s1);  // LỖI: s1 đã bị move, không dùng được nữa!
println!("{}", s2);     // ok
```

**Tại sao?** `String` gồm con trỏ tới heap + metadata trên stack. Nếu cho cả `s1` và `s2` cùng trỏ tới một vùng heap, khi cả hai ra khỏi scope sẽ **double free** (giải phóng 2 lần → crash/bảo mật). Rust ngăn bằng cách: gán = move, biến cũ mất hiệu lực ngay lúc biên dịch.

**Với kiểu đơn giản (i32, bool...):** chúng implement trait `Copy` nên gán là **copy** chứ không move:
```rust
let x = 5;
let y = x;  // copy, x vẫn dùng được
println!("{} {}", x, y);  // ok
```

### Hàm và ownership

```rust
fn lay_ownership(s: String) {  // s được move vào hàm
    println!("{}", s);
}  // s ra khỏi scope → giải phóng

fn tra_ownership(s: String) -> String {
    s  // trả lại quyền sở hữu
}

fn main() {
    let s = String::from("hello");
    lay_ownership(s);  // s bị move vào hàm, không dùng được nữa
    // println!("{}", s);  // LỖI

    let s2 = String::from("world");
    let s3 = tra_ownership(s2);  // nhận lại quyền sở hữu
    println!("{}", s3);  // ok
}
```

---

## 8. Borrowing và References

Move mỗi lần gọi hàm thì bất tiện quá. Giải pháp: **borrow** (mượn) bằng reference.

```rust
fn tinh_do_dai(s: &String) -> usize {  // &String: mượn, không lấy ownership
    s.len()
}  // s là reference, ra khỏi scope không giải phóng gì

fn main() {
    let s = String::from("hello");
    let len = tinh_do_dai(&s);  // cho mượn
    println!("{} dài {} ký tự", s, len);  // s vẫn dùng được!
}
```

### Mutable reference — mượn để sửa

```rust
fn them_chuoi(s: &mut String) {
    s.push_str(" world");
}

fn main() {
    let mut s = String::from("hello");
    them_chuoi(&mut s);
    println!("{}", s);  // "hello world"
}
```

### Quy tắc borrowing (compiler bắt chặt)

- **Hoặc** có nhiều immutable reference (`&T`)
- **Hoặc** có đúng một mutable reference (`&mut T`)
- **Không bao giờ** có cả hai loại cùng lúc

```rust
let mut s = String::from("hello");
let r1 = &s;     // ok
let r2 = &s;     // ok (nhiều immutable)
// let r3 = &mut s;  // LỖI: đang có immutable borrow
println!("{} {}", r1, r2);
```

**Tại sao quy tắc này tồn tại?** Nó loại bỏ **data race** ngay lúc biên dịch: không thể vừa đọc vừa ghi cùng một dữ liệu từ hai nơi. Đây là lý do Rust "fearless concurrency".

### Dangling reference — không bao giờ xảy ra

```rust
// fn tao_chuoi() -> &String {  // LỖI BIÊN DỊCH
//     let s = String::from("hello");
//     &s  // s chết khi hàm kết thúc → reference treo!
// }
```

Compiler từ chối. Trong C++ đây là bug chạy được nhưng sai lung tung.

---

## 9. Slice

Slice là "cửa sổ nhìn" vào một dãy liên tiếp, không sở hữu dữ liệu:

```rust
let s = String::from("hello world");
let phan1 = &s[0..5];   // "hello"
let phan2 = &s[6..];    // "world" (tới hết)
let tat_ca = &s[..];    // cả chuỗi

let arr = [1, 2, 3, 4, 5];
let lat_cat = &arr[1..4];  // [2, 3, 4]
```

**`&str` vs `String`:** `String` sở hữu dữ liệu (growable, trên heap), `&str` chỉ mượn (string literal `"hello"` có kiểu `&'static str`). Quy tắc: tham số hàm nên nhận `&str` (linh hoạt nhận cả `String` lẫn literal):
```rust
fn in_ra(s: &str) { println!("{}", s); }
in_ra("literal");                    // ok
in_ra(&String::from("owned"));       // ok
```

---

## 10. Struct

```rust
struct SinhVien {
    ten: String,
    tuoi: u32,
    diem: f64,
}

fn main() {
    let sv = SinhVien {
        ten: String::from("Duong"),
        tuoi: 20,
        diem: 8.5,
    };
    println!("{} - {}", sv.ten, sv.diem);

    // struct không cần tên field (tuple struct)
    struct DiemMau(u8, u8, u8);
    let do_ = DiemMau(255, 0, 0);
}
```

### Method với `impl`

```rust
impl SinhVien {
    // associated function (như static method)
    fn moi(ten: String, tuoi: u32) -> Self {
        SinhVien { ten, tuoi, diem: 0.0 }
    }

    // method: tham số đầu là self
    fn gioi_thieu(&self) {  // &self: mượn, không sửa
        println!("Tôi là {}, {} tuổi", self.ten, self.tuoi);
    }

    fn tang_diem(&mut self, them: f64) {  // &mut self: được sửa
        self.diem += them;
    }
}

let mut sv = SinhVien::moi(String::from("Duong"), 20);
sv.gioi_thieu();
sv.tang_diem(1.5);
```

---

## 11. Enum và Pattern Matching

Enum trong Rust mạnh hơn C rất nhiều — mỗi variant có thể mang dữ liệu khác nhau:

```rust
enum ThongDiep {
    Thoat,                          // không dữ liệu
    DiChuyen { x: i32, y: i32 },    // struct-like
    Viet(String),                    // tuple-like
    DoiMau(u8, u8, u8),
}
```

### `match` — pattern matching toàn diện

```rust
let msg = ThongDiep::Viet(String::from("hello"));

match msg {
    ThongDiep::Thoat => println!("Thoát"),
    ThongDiep::DiChuyen { x, y } => println!("Đi tới ({}, {})", x, y),
    ThongDiep::Viet(s) => println!("Viết: {}", s),
    ThongDiep::DoiMau(r, g, b) => println!("Màu {},{},{}", r, g, b),
}
```

**Compiler bắt phải exhaustive** (xử lý hết mọi trường hợp). Quên một variant → lỗi biên dịch. Không còn bug "quên case" như switch C++.

```rust
let x = 5;
match x {
    1 => println!("một"),
    2 | 3 => println!("hai hoặc ba"),  // nhiều pattern
    4..=10 => println!("từ 4 đến 10"), // range
    _ => println!("khác"),             // _ = còn lại (bắt buộc nếu chưa đủ)
}
```

### `if let` — gọn khi chỉ quan tâm một case

```rust
let so = Some(5);
if let Some(v) = so {
    println!("Có số: {}", v);
}
```

---

## 12. Option và Result

Rust **không có null**. Thay vào đó:

### `Option<T>` — giá trị có thể không tồn tại

```rust
enum Option<T> {
    Some(T),
    None,
}

fn tim(arr: &[i32], x: i32) -> Option<usize> {
    for (i, v) in arr.iter().enumerate() {
        if *v == x { return Some(i); }
    }
    None
}

match tim(&[1, 2, 3], 2) {
    Some(i) => println!("Tìm thấy ở {}", i),
    None => println!("Không thấy"),
}

// Các cách dùng gọn
let x: Option<i32> = Some(5);
println!("{}", x.unwrap_or(0));  // 5, nếu None thì 0
let y = x.map(|v| v * 2);        // Some(10)
```

**Tại sao tốt hơn null?** Compiler bắt bạn xử lý cả hai trường hợp. Không còn "NullPointerException lúc 3 giờ sáng".

### `Result<T, E>` — thành công hoặc lỗi

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

use std::fs::File;
let f: Result<File, std::io::Error> = File::open("data.txt");
match f {
    Ok(file) => println!("Mở được!"),
    Err(e) => println!("Lỗi: {}", e),
}
```

---

## 13. Collections

```rust
// Vec: mảng động (như vector C++)
let mut v = vec![1, 2, 3];
v.push(4);
for x in &v { println!("{}", x); }

// String: đã học
let mut s = String::from("hello");
s.push_str(" world");
s.push('!');

// HashMap
use std::collections::HashMap;
let mut diem = HashMap::new();
diem.insert("Toán", 9);
diem.insert("Văn", 8);
println!("{:?}", diem.get("Toán"));  // Some(9)

// Truy cập an toàn: get() trả về Option, không panic như []
```

---

## 14. Error handling

### Toán tử `?` — lan truyền lỗi cực gọn

```rust
use std::fs::File;
use std::io::{self, Read};

fn doc_file(path: &str) -> Result<String, io::Error> {
    let mut s = String::new();
    File::open(path)?.read_to_string(&mut s)?;  // ? = nếu Err thì return luôn
    Ok(s)
}
```

`?` tương đương với match dài dòng nhưng gọn hơn 90%. Đây là cách viết idiomatic trong Rust.

### `panic!` — khi lỗi không thể phục hồi

```rust
// panic = crash có chủ đích, in stack trace
// dùng cho: bug lập trình, assert
let v = vec![1, 2, 3];
let x = v[10];  // panic: index out of bounds
```

**Quy tắc:** lỗi dự kiến được (file không tồn tại, mạng đứt) → `Result`. Lỗi lập trình (bug) → `panic!`/assert.

---

## 15. Generics

```rust
fn lon_nhat<T: PartialOrd>(a: T, b: T) -> T {
    if a > b { a } else { b }
}

lon_nhat(3, 5);      // i32
lon_nhat(3.5, 2.1);  // f64
```

`<T: PartialOrd>` là trait bound: T phải implement trait `PartialOrd` (so sánh được). Rust kiểm tra lúc biên dịch, không có runtime cost.

```rust
struct Hop<T> {
    do_vat: T,
}

impl<T> Hop<T> {
    fn new(do_vat: T) -> Self { Hop { do_vat } }
}
```

---

## 16. Traits

Trait = interface: định nghĩa hành vi chung mà nhiều kiểu cùng implement.

```rust
trait Keu {
    fn keu(&self) -> String;
}

struct Cho;
struct Meo;

impl Keu for Cho {
    fn keu(&self) -> String { String::from("Gâu!") }
}
impl Keu for Meo {
    fn keu(&self) -> String { String::from("Meo!") }
}

// Hàm nhận bất kỳ kiểu nào implement Keu
fn cho_keu(dv: &impl Keu) {
    println!("{}", dv.keu());
}
```

**Trait chuẩn hay dùng:** `Debug` (in debug với `{:?}`), `Clone`, `Copy`, `PartialEq`, `Display` (in với `{}`), `Iterator`.

```rust
#[derive(Debug, Clone)]  // tự sinh code implement
struct Diem { x: i32, y: i32 }

let d = Diem { x: 1, y: 2 };
println!("{:?}", d);  // Diem { x: 1, y: 2 }
```

---

## 17. Lifetime

Lifetime đảm bảo reference không outlive dữ liệu nó trỏ tới:

```rust
fn dai_nhat<'a>(a: &'a str, b: &'a str) -> &'a str {
    if a.len() > b.len() { a } else { b }
}
```

`'a` nghĩa là: reference trả về sống không quá reference ngắn nhất trong hai đầu vào. Compiler tự suy luận hầu hết trường hợp (elision), chỉ cần ghi khi nó không suy được.

**Đừng sợ lifetime:** 90% code không cần ghi tường minh. Khi compiler báo lỗi lifetime, thường là code đang có vấn đề thật (reference treo).

---

## 18. Smart Pointers

```rust
// Box<T>: cấp phát trên heap, sở hữu duy nhất
let b = Box::new(5);

// Rc<T>: đếm tham chiếu (single-thread)
use std::rc::Rc;
let a = Rc::new(5);
let b = Rc::clone(&a);  // đếm = 2

// Arc<T>: như Rc nhưng thread-safe (atomic)
// RefCell<T>: kiểm tra borrow lúc CHẠY (thay vì lúc biên dịch)
// Dùng khi cần interior mutability
```

---

## 19. Concurrency

```rust
use std::thread;

let handle = thread::spawn(|| {
    println!("Chạy trên thread mới!");
});
handle.join().unwrap();  // đợi thread xong

// Message passing với channel (khuyên dùng hơn share memory)
use std::sync::mpsc;
let (tx, rx) = mpsc::channel();
thread::spawn(move || {
    tx.send("hello").unwrap();
});
println!("{}", rx.recv().unwrap());
```

**Triết lý Rust:** "don't communicate by sharing memory; share memory by communicating". Ownership system đảm bảo không data race lúc biên dịch.

---

## 20. Module và Crate

```rust
// src/lib.rs hoặc main.rs
mod utils;  // nạp từ src/utils.rs

// src/utils.rs
pub fn cong(a: i32, b: i32) -> i32 { a + b }  // pub: công khai
fn rieng_tu() {}  // mặc định private

// Dùng:
fn main() {
    println!("{}", utils::cong(1, 2));
    // hoặc: use crate::utils::cong;
}
```

**Crate** = đơn vị biên dịch (một package). **Module** = tổ chức code trong crate.

---

## 21. Testing

```rust
fn cong(a: i32, b: i32) -> i32 { a + b }

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_cong() {
        assert_eq!(cong(2, 3), 5);
    }

    #[test]
    #[should_panic]
    fn test_panic() {
        panic!("cố ý");
    }
}
```
```bash
cargo test  # chạy tất cả test
```

---

## 22. Best practices

1. **Để compiler dẫn đường**: lỗi Rust dài nhưng đọc kỹ sẽ chỉ đúng chỗ sửa
2. **Dùng `cargo clippy` và `cargo fmt`** thường xuyên
3. **Ưu tiên borrow (`&`) thay vì clone** khi chỉ cần đọc
4. **Dùng `?` thay vì `unwrap()`** trong code production (`unwrap` chỉ cho prototype/test)
5. **Tham số hàm nhận `&str` thay vì `&String`**, nhận `&[T]` thay vì `&Vec<T>` (linh hoạt hơn)
6. **Đặt tên theo snake_case** (hàm, biến), **UpperCamelCase** (struct, enum, trait)
7. **Viết test** ngay từ đầu với `#[test]`
8. **Đọc lỗi từ trên xuống**: lỗi đầu tiên thường là nguyên nhân, các lỗi sau là hệ quả
9. **Dùng `Result` cho lỗi dự kiến, `panic!` cho bug**
10. **Học ownership bằng thực hành**: viết code, để compiler mắng, sửa, lặp lại — đó là cách nhanh nhất

---

*Tài liệu được viết cho người mới, đặc biệt nhấn mạnh ownership — phần khó nhất. Hãy gõ lại từng ví dụ, cố tình viết sai để xem compiler nói gì, bạn sẽ học nhanh gấp đôi.*
