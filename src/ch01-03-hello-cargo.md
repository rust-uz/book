# Salom, Cargo! (Hello, Cargo!)

Kichik dasturlar uchun `rustc main.rs` yetarli, lekin haqiqiy loyihalarda yuzlab fayllar, tashqi kutubxonalar (dependencies) va testlar bo‘ladi. Rust bularning barchasini boshqarish uchun o‘zining rasmiy qurish tizimi va paket menejeri — **Cargo** ni taqdim etadi.

Cargo nimalarga qodir?
* Loyihalar yaratish va strukturasini boshqarish;
* Kodni kompilyatsiya qilish (`cargo build`);
* Dasturni darhol ishga tushirish (`cargo run`);
* Kodda xatolar yo‘qligini tezkor tekshirish (`cargo check`);
* Tashqi kutubxonalarni avtomatik yuklab olish;
* Testlarni ishga tushirish (`cargo test`).

---

## Cargo Yordamida Yangi Loyiha Ochish

Terminalingizda yangi loyiha yaratish uchun quyidagi buyruqni bering:

```bash
cargo new salom_cargo
cd salom_cargo
```

Keling, Cargo bizga qanday fayllarni yaratib berganini ko‘ramiz:

```text
salom_cargo
├── Cargo.toml
└── src
    └── main.rs
```

Cargo avtomatik tarzda:
1. **`Cargo.toml`** konfiguratsiya faylini yaratdi. TOML (*Tom's Obvious Minimal Language*) formati loyiha haqidagi metadata va uning bog‘liqliklarini saqlaydi.
2. **`src/`** jildini va uning ichida boshlang‘ich `main.rs` faylini yaratdi.
3. Yangi Git repozitoriyasini boshladi (`.git` va `.gitignore`).

---

## `Cargo.toml` Fayli Ichida Nimalar Bor?

Keling, `Cargo.toml` faylini ko‘rib chiqaylik:

```toml
[package]
name = "salom_cargo"
version = "0.1.0"
edition = "2021"

[dependencies]
```

* `[package]` bo‘limi paket nomi, versiyasi va Rust nashri (edition) ni belgilaydi.
* `[dependencies]` bo‘limida esa loyihangiz foydalanadigan barcha tashqi kutubxonalar ro‘yxati (crates.io dan) yoziladi.

---

## Loyihani Yig‘ish va Ishga Tushirish

Cargo bilan loyihani yig‘ish juda oson:

```bash
cargo build
```

Bu buyruq kodni kompilyatsiya qiladi va `target/debug/salom_cargo` manziliga dastur faylini joylashtiradi.

Agar siz kodni ham yig‘ib, ham bir vaqtda ishga tushirmoqchi bo‘lsangiz, shunchaki quyidagini yozasiz:

```bash
cargo run
```

Ekranda:
```text
   Compiling salom_cargo v0.1.0 (/home/user/salom_cargo)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/salom_cargo`
Hello, world!
```

### Kodni Tezkor Tekshirish (`cargo check`)
Katta loyihalarda har safar binar fayl yasamasdan, faqat sintaksis va tiplar to‘g‘riligini tekshirish uchun:

```bash
cargo check
```

Bu `cargo build` dan bir necha barobar tez ishlaydi va kompyuter resurslarini tejaydi.

---

## Release Rejimida Yig‘ish

Loyihangiz tayyor bo‘lib, uni ishlab chiqarish (production) muhitiga chiqarish vaqti kelganda, maksimal optimizatsiyalash uchun quyidagi bayroqdan foydalanasiz:

```bash
cargo build --release
```

Bu buyruq barcha CPU optimizatsiyalarini yoqadi, kodni tekshirishlarni tezlashtiradi va binar hajmini kichraytiradi (`target/release/salom_cargo`).

Tabriklaymiz! Siz Rust va Cargo asoslarini to‘liq o‘zlashtirdingiz.
