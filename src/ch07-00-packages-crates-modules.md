# 7-Bob: Paketlar, Crate-lar va Modullar Tizimi (Packages, Crates, Modules)

Dasturlar kattalashgani sari, barcha kodni bitta faylda saqlash qiyinlashib boradi. Kodni funksionalligi bo‘yicha alohida bloklarga ajratish, detallarni yashirish (enkapsulyatsiya) va ommaviy interfeyslarni aniq belgilash zarurati paydo bo‘ladi.

Rust modullar tizimi quyidagi asosiy vositalardan iborat:
- **Paketlar (Packages):** Crate-larni yig‘ish, sinash va ulashish imkonini beruvchi Cargo funksiyasi.
- **Crate-lar:** Modullardan tashkil topgan daraxtsimon kod to‘plami (kutubxona yoki ijro etiluvchi dastur).
- **Modullar (Modules) va `use`:** Kodni tashkil etish, ko‘rinish doirasini boshqarish va maxfiylik chegaralari.
- **Yo‘llar (Paths):** Struktura, funksiya yoki modullarga murojaat qilish manzili.

---

## 1. Paketlar va Crate-lar

- **Crate** — bu Rust kompilyatori bir vaqtning o‘zida tahlil qiladigan eng kichik kod birligidir.
  - **Binar Crate (Binary Crate):** Ijro etilishi mumkin bo‘lgan dastur. U har doim `main` funksiyasiga ega bo‘ladi. Odatda `src/main.rs` faylida joylashadi.
  - **Kutubxona Crate (Library Crate):** Boshqa dasturlar foydalanishi uchun mo‘ljallangan funksiyalar va turlar to‘plami. `main` funksiyasiga ega bo‘lmaydi. Odatda `src/lib.rs` faylida joylashadi.

- **Paket (Package)** — bu bitta yoki bir nechta crate-larni o‘z ichiga olgan va ularni qanday yig‘ishni belgilovchi `Cargo.toml` fayliga ega bo‘lgan papkadir.
  - Har bir paket **istalgancha binar crate**ga ega bo‘lishi mumkin (`src/bin/` ichida).
  - Lekin har bir paketda **ko‘pi bilan faqat BITTA kutubxona crate** bo‘lishi mumkin (`src/lib.rs`).

---

## 2. Modullar Daraxti (Module Tree)

Modullar dastur kodini tartibga solish va maxfiylik chegarasini (privacy) belgilash uchun ishlatiladi:

```rust
// src/lib.rs ichida

mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}
```

Ushbu modullar tizimi quyidagi daraxtni hosil qiladi:

```text
crate (src/lib.rs)
 └── front_of_house
      ├── hosting
      │    ├── add_to_waitlist
      │    └── seat_at_table
      └── serving
           ├── take_order
           ├── serve_order
           └── take_payment
```

---

## 3. Modullarga Yo‘llar va Maxfiylik (Privacy Boundary)

Rust da **barcha narsalar (funksiyalar, metodlar, strukturalar, modullar) odatiy holatda MAXFIY (private)** bo‘ladi!

- Ota modul ichidagi kod farzand modulning shaxsiy detallarini ko‘ra olmaydi.
- Lekin farzand modul o‘zining barcha ajdodlari (ota-onalari) kontekstini ko‘ra oladi.
- Tashqi dunyoga elementni ochish uchun uning oldiga **`pub`** kalit so‘zi qo‘yiladi.

### Yo‘llar Turlari:
1. **Mutlaq yo‘l (Absolute path):** Crate ildizidan boshlanadi va `crate` so‘zi bilan yoziladi:
   `crate::front_of_house::hosting::add_to_waitlist();`
2. **Nisbiy yo‘l (Relative path):** Joriy moduldan boshlanadi yoki ota modulga murojaat qilish uchun `super` dan foydalanadi:
   `front_of_house::hosting::add_to_waitlist();`
   `super::boshqa_funksiya();`

```rust,ignore
pub fn eat_at_restaurant() {
    // Mutlaq yo'l
    crate::front_of_house::hosting::add_to_waitlist();

    // Nisbiy yo'l
    front_of_house::hosting::add_to_waitlist();
}
```

### Struktura va Enumlar Maxfiyligi:
- Agar strukturani `pub struct` deb e’lon qilsangiz, uning o‘zi ommaviy bo‘ladi, ammo **uning maydonlari hali ham maxfiy bo‘lib qoladi**. Har bir maydonga alohida `pub` qo‘yish kerak!
- Agar enumni `pub enum` deb e’lon qilsangiz, uning **barcha variantlari avtomatik tarzda ommaviy (`pub`)** bo‘ladi.

---

## 4. `use` Kalit So‘zi Bilan Yo‘lni Qisqartirish

Har safar uzun yo‘llarni yozmaslik uchun `use` orqali kerakli elementni ko‘rinish sohasiga olib kirish mumkin:

```rust,ignore
use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

### Qayta Nomlash (`as`):
Agar ikki xil moduldan bir xil nomli turlarni import qilish kerak bo‘lsa:

```rust
use std::fmt::Result;
use std::io::Result as IoResult;
```

### Qayta Eksport Qilish (`pub use`):
Ichki kod strukturasini yashirgan holda, tashqi foydalanuvchilar uchun qulay API yaratish maqsadida `pub use` ishlatiladi:

```rust,ignore
pub use crate::front_of_house::hosting;
```

---

## 5. Modullarni Alohida Fayllarga Ajratish

Loyiha kattalashganda har bir modulni o‘z fayliga chiqarish eng toza usuldir (Rust 2018+ standarti):

**`src/lib.rs`:**
```rust,ignore
mod front_of_house; // Rust avtomatik front_of_house.rs faylini qidiradi

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

**`src/front_of_house.rs`:**
```rust,ignore
pub mod hosting; // front_of_house/hosting.rs faylini qidiradi
```

**`src/front_of_house/hosting.rs`:**
```rust,ignore
pub fn add_to_waitlist() {}
```

Fayllar daraxti quyidagicha chiroyli va tartibli bo‘ladi:
```text
src/
├── lib.rs
├── front_of_house.rs
└── front_of_house/
    └── hosting.rs
```
Bu tizim katta jamoalarda yuz minglab qatorli Rust loyihalarini oson va tartibli boshqarish imkonini beradi.
