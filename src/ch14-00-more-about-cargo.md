# 14-Bob: Cargo va Crates.io ga Chuqurroq (More About Cargo & Crates.io)

Shu paytgacha biz Cargo vositasidan kodimizni yig‘ish (`build`), ishga tushirish (`run`) va testlash (`test`) uchun foydalandik. Ammo Cargo — shunchaki kompilyator yordamchisi emas, u zamonaviy dasturiy ta’minotni tarqatish, versiyalash va yirik loyihalarni boshqarishning butun bir ekotizimidir.

Ushbu bobda quyidagi ilg‘or imkoniyatlarni o‘rganamiz:
- Yig‘ish profillarini sozlash (Release Profiles);
- O‘z kutubxonangizni Crates.io ga joylashtirish;
- Cargo Workspaces (Ish maydonlari) orqali yirik monorepolarni boshqarish;
- Binar dasturlarni o‘rnatish va maxsus Cargo buyruqlarini kengaytirish.

---

## 1. Yig‘ish Profillari (Release Profiles)

Cargo ikkita asosiy profilga ega:
1. **`dev` profili:** `cargo build` buyrug‘ida qo‘llaniladi. Dasturchi kodni tez yozib, darhol tekshirishi uchun kodni optimallashtirishga kam vaqt sarflaydi va to‘liq debug ma’lumotlarini saqlaydi.
2. **`release` profili:** `cargo build --release` buyrug‘ida qo‘llaniladi. Ishlab chiqarish (production) uchun maksimal darajada optimallashtiriladi va eng yuqori ishlash tezligini beradi.

`Cargo.toml` faylida profillarning xususiyatlarini o‘zgartirishingiz mumkin:

```toml
[profile.dev]
opt-level = 0 # 0 dan 3 gacha (0 - optimallashtirishsiz, tez yig'iladi)

[profile.release]
opt-level = 3 # maksimal tezlikka optimallashtirish
lto = true     # Link-Time Optimization (binarni yanada kichraytiradi va tezlashtiradi)
```

---

## 2. O‘z Paketingizni Crates.io ga Joylash

[crates.io](https://crates.io) — butun dunyo Rust dasturchilari o‘z kutubxonalari (crate)ni ochiq manbali tarzda ulashadigan markaziy repozitoriydir.

Paketingizni boshqalar qulay ishlatishi uchun birinchi navbatda ajoyib hujjat (documentation) bo‘lishi zarur.

### Hujjatlashtirish Bo‘limlari Standarti:
Rust hamjamiyatida `///` izohlari ichida quyidagi bo‘limlar bo‘lishi odat tusiga kirgan:
- **`# Misollar (# Examples)`:** Kod qanday ishlatilishini ko‘rsatuvchi misollar (bular `cargo test` vaqtida avtomatik tekshiriladi!).
- **`# Panics`:** Funksiya qaysi holatlarda dasturni `panic!` qilishini ochiq ko‘rsatish.
- **`# Errors`:** Agar funksiya `Result` qaytarsa, qanday xatoliklar kelib chiqishi mumkinligini tushuntirish.

```rust
/// Berilgan songa birni qo‘shadi.
///
/// # Misollar
///
/// ```
/// use mening_kutubxonam::birni_qosh;
///
/// let natija = birni_qosh(5);
/// assert_eq!(natija, 6);
/// ```
pub fn birni_qosh(x: i32) -> i32 {
    x + 1
}
```

### `Cargo.toml` Metama’lumotlarini To‘ldirish
Crates.io ga yuklashdan oldin `Cargo.toml` ga quyidagi ma’lumotlarni kiritish shart:

```toml
[package]
name = "rust_uz_super_kutubxona"
version = "0.1.0"
edition = "2021"
authors = ["Rust O‘zbekiston Hamjamiyati <info@ferris.uz>"]
description = "O'zbek tilidagi qulay va tezkor yordamchi vositalar"
license = "MIT OR Apache-2.0"
repository = "https://github.com/rust-uz/super-kutubxona"
readme = "README.md"
```

### Chop Etish:
1. `crates.io` dan API token olasiz;
2. Terminalda tizimga kirasiz: `cargo login <TOKEN>`;
3. Chop etasiz: `cargo publish`.

> **Muhim:** Crates.io dagi kodlar doimiydir! Hech kim mavjud versiyadagi kodni o‘chirib tashlay olmaydi, bu esa boshqa dasturlarning sinib qolishini oldini oladi. Agar versiyani eskirgan deb belgilamoqchi bo‘lsangiz, `cargo yank --vers 0.1.0` buyrug‘idan foydalanasiz.

---

## 3. Cargo Ish Maydonlari (Workspaces)

Loyiha juda katta bo‘lib ketganda, uni bir nechta o‘zaro bog‘liq alohida crate-larga bo‘lish tavsiya etiladi. Bunga **Workspace (Ish maydoni)** deyiladi. Barcha paketlar umumiy bitta `Cargo.lock` va bitta umumiy `target/` papkasidan foydalanadi.

Ildizdagi `Cargo.toml`:
```toml
[workspace]
members = [
    "server",
    "klient",
    "umumiy_modellar",
]
```

Daraxt tuzilishi:
```text
mening_ish_maydonim/
├── Cargo.toml
├── Cargo.lock
├── target/
├── server/
│   ├── Cargo.toml
│   └── src/main.rs
├── klient/
│   ├── Cargo.toml
│   └── src/main.rs
└── umumiy_modellar/
    ├── Cargo.toml
    └── src/lib.rs
```
`server` va `klient` crate-lari o‘z `Cargo.toml` fayllarida `umumiy_modellar = { path = "../umumiy_modellar" }` orqali bir-biridan oson foydalana oladi.

---

## 4. Binar Dasturlarni O‘rnatish: `cargo install`

Rust da yozilgan CLI vositalarni to‘g‘ridan-to‘g‘ri kompyuteringizga o‘rnatib, global buyruq sifatida ishlatishingiz mumkin:

```bash
cargo install ripgrep
```
Bu buyruq dasturning manba kodini yuklab oladi, optimallashtirib kompilyatsiya qiladi va `$HOME/.cargo/bin` papkasiga joylashtiradi.

### Maxsus Cargo Buyruqlarini Kengaytirish
Agar siz tizimingizga `cargo-birnarsa` nomli binar dasturni o‘rnatsangiz, uni to‘g‘ridan-to‘g‘ri `cargo birnarsa` ko‘rinishida ishga tushirish imkoniga ega bo‘lasiz! Bu Cargo vositasini cheksiz kengaytirish imkonini beradi.
