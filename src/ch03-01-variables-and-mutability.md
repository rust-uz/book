# O‘zgaruvchilar va O‘zgaruvchanlik (Variables and Mutability)

Rust tilining eng muhim xususiyatlaridan biri shundaki, barcha o‘zgaruvchilar **odatiy holatda o‘zgarmas (immutable)** qilib yaratiladi. Bu tasodifiy tanlov emas, balki xavfsiz va parallel (concurrency) dasturlashni soddalashtirish uchun kiritilgan asosiy tamoyillardan biridir.

---

## O‘zgarmaslik (Immutability) nima uchun kerak?

O‘zgaruvchi o‘zgarmas bo‘lsa, unga bir marta qiymat biriktirilgach, uni qayta o‘zgartirib bo‘lmaydi. Quyidagi misolga qarang:

```rust,compile_fail
fn main() {
    let x = 5;
    println!("x ning qiymati: {x}");
    x = 6; // Kompilyatsiya xatosi!
    println!("x ning qiymati: {x}");
}
```

Ushbu kodni `cargo run` qilsangiz, Rust kompilyatori quyidagicha aniq xato beradi:

```text
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         - first assignment to `x`
3 |     println!("x ning qiymati: {x}");
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
```

Rust kompilyatori siz o‘zgarmas deb e’lon qilgan narsani o‘zgartirmasligingizni qat’iy talab qiladi. Bu esa:
1. Dasturning boshqa qismida ushbu qiymat kutilmaganda o‘zgarib ketmasligiga 100% kafolat beradi.
2. Kodni o‘qiyotgan dasturchiga "bu qiymat hech qachon o‘zgarmaydi" degan ishonch beradi.

---

## O‘zgaruvchanlik: `mut` Kalit So‘zi

Agar qiymat dastur davomida o‘zgarishi kerak bo‘lsa, `let` dan so‘ng **`mut`** kalit so‘zini qo‘shamiz:

```rust
fn main() {
    let mut x = 5;
    println!("Boshlang‘ich qiymat: {x}");
    x = 6; // Endi ruxsat beriladi!
    println!("O‘zgargan qiymat: {x}");
}
```

`mut` kompilyatorga hamda boshqa dasturchilarga ushbu o‘zgaruvchi keyingi satrlarda yangilanishini bildiradi.

---

## O‘zgarmaslar (Constants)

O‘zgarmaslar (`const`) — har doim o‘zgarmas bo‘lgan qiymatlardir. Ular `let` bilan yaratilgan o‘zgarmas o‘zgaruvchilardan bir necha jihatdan farq qiladi:

1. `const` bilan hech qachon `mut` ishlatib bo‘lmaydi.
2. O‘zgarmasning tipi **har doim aniq yozilishi shart** (tur xulosasi - type inference ishlamaydi).
3. O‘zgarmas faqat **kompilyatsiya vaqtida hisoblanishi mumkin bo‘lgan ifoda** bo‘lishi kerak (masalan, funksiya natijasi yoki runtime da olinadigan qiymat bo‘lishi mumkin emas).
4. O‘zgarmaslar har qanday doirada (hatto global miqyosda ham) e’lon qilinishi mumkin.
5. Rust an’anasiga ko‘ra, ularning nomi KATTA_HARFLAR bilan yoziladi.

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;

fn main() {
    println!("3 soat: {THREE_HOURS_IN_SECONDS} soniya");
}
```

---

## O‘zgaruvchini Soyalash (Shadowing)

Rust da siz avval e’lon qilingan o‘zgaruvchi bilan bir xil nomda yangi o‘zgaruvchini `let` yordamida qayta e’lon qilishingiz mumkin:

```rust
fn main() {
    let x = 5;

    let x = x + 1; // 5 + 1 = 6

    {
        let x = x * 2; // 6 * 2 = 12 (ichki blokda)
        println!("Ichki blokdagi x: {x}"); // 12
    }

    println!("Tashqi blokdagi x: {x}"); // 6
}
```

### Shadowing ning `mut` dan Farqlari:
1. **O‘zgarmaslik saqlanadi:** `let` orqali yangilagandan so‘ng, yangi o‘zgaruvchi yana o‘zgarmas (immutable) bo‘lib qoladi.
2. **Tipni o‘zgartirish mumkin:** `mut` faqat qiymatni o‘zgartirishga ruxsat beradi, lekin tipni o‘zgartirib bo‘lmaydi. Shadowing da esa yangi o‘zgaruvchi yaratilayotgani sababli, ma’lumot turini o‘zgartirish mumkin!

```rust
// Bu to‘g‘ri ishlaydi (Shadowing):
let spaces = "   "; // &str tipi
let spaces = spaces.len(); // usize tipi

// Bu esa xato beradi (mut):
// let mut spaces = "   ";
// spaces = spaces.len(); // XATO: tiplar mos kelmadi!
```

Shadowing orqali biz xotirada eskisini yashirib, xuddi shu nom bilan yangi o‘zgaruvchi ochamiz. Bu kodni toza va tushunarli saqlashga yordam beradi.
