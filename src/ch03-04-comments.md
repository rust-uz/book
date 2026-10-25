# Izohlar (Comments)

Dasturlashda izohlar (kommentlar) — kompilyator tomonidan e’tiborsiz qoldiriladigan, ammo kodni o‘qiydigan boshqa dasturchilar (yoki kelajakdagi o‘zingiz) uchun yoziladigan tushuntirish matnlaridir.

Rust da izohlarning bir necha turlari mavjud.

---

## 1. Oddiy Bir Qatorli Izohlar

Eng keng tarqalgan izoh shakli — bu ikkita qiya chiziq (`//`) bilan boshlanadigan bir qatorli izohlardir:

```rust
// Bu yerda o'zgaruvchi e'lon qilinmoqda
let x = 5;

let y = 10; // Qator oxiriga ham izoh yozish mumkin
```

---

## 2. Ko‘p Qatorli Blok Izohlar

Bir nechta qatorni birdaniga izohga olish uchun `/*` va `*/` sintaksisidan foydalaniladi:

```rust
/*
   Ushbu dastur juda murakkab matematik
   hisob-kitoblarni amalga oshiradi va
   quyidagi bosqichlardan iborat...
*/
fn hisobla() {
    // ...
}
```

Biroq, Rust amaliyotida ko‘p qatorli tushuntirishlar uchun ham har bir qator boshiga `//` qo‘yish ko‘proq tavsiya etiladi.

---

## 3. Hujjatlashtiruvchi Izohlar (Documentation Comments)

Rust ning eng sevimli xususiyatlaridan biri — bu kodning o‘zidan avtomatik HTML hujjatlar (documentation) yaratish imkoniyatidir. Buning uchun uchta qiya chiziq (**`///`**) ishlatiladi.

Ushbu izohlar ichida to‘liq **Markdown** sintaksisi qo‘llab-quvvatlanadi:

```rust
/// Berilgan songa birni qo‘shib qaytaradi.
///
/// # Misollar
///
/// ```
/// let natija = birni_qosh(5);
/// assert_eq!(natija, 6);
/// ```
pub fn birni_qosh(x: i32) -> i32 {
    x + 1
}
```

Agar terminalda quyidagi buyruqni bersangiz:

```bash
cargo doc --open
```

Rust kodingizdagi barcha `///` izohlarini chiroyli brauzer veb-sahifasiga aylantirib beradi! Bundan tashqari, izoh ichidagi ` ``` ` kod bloklari `cargo test` vaqtida avtomatik ravishda test sifatida ham tekshiriladi (*Doc-tests*).

### Butun Modul yoki Crate uchun Izoh: `//!`
Agar izoh butun fayl yoki modulning o‘ziga tegishli bo‘lsa, u faylning eng yuqori qismida `//!` orqali yoziladi:

```rust
//! # Mening Ajoyib Matematik Kutubxonam
//!
//! Ushbu crate yuqori aniqlikdagi hisoblashlar uchun mo‘ljallangan.
```
