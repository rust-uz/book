# 11-Bob: Avtomatlashtirilgan Testlar Yozish (Writing Automated Tests)

Rust dasturlash tilining muhim afzalliklaridan biri — dastur to‘g‘riligini tekshirish uchun testlar tizimining bevosita til va Cargo tarkibiga kiritilganligidir. Hech qanday tashqi kutubxona yoki murakkab freymvorklarni o‘rnatish shart emas.

Rust testlari uchta asosiy amalni bajaradi:
1. Kerakli ma’lumotlar yoki holatni sozlash;
2. Sinovdan o‘tkazilayotgan kodni ishga tushirish;
3. Natija biz kutganimizdek ekanligini tasdiqlash (*assert*).

---

## 1. Test Funksiyasi Anatomiyasi

Har qanday test funksiyasi uning tepasiga qo‘yiladigan **`#[test]`** atributi bilan belgilanadi. Ushbu atribut Cargo ga bu oddiy funksiya emas, balki sinov funksiyasi ekanligini bildiradi.

```rust
pub fn qosh(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn ikki_va_ikki_tort_boladi() {
        let natija = qosh(2, 2);
        assert_eq!(natija, 4);
    }
}
```

Testni ishga tushirish uchun shunchaki quyidagi buyruqni bering:

```bash
cargo test
```

### Tasdiqlash Makroslari:
- **`assert!(shart)`:** Berilgan mantiqiy ifoda `true` ekanligini tekshiradi. Agar `false` bo‘lsa, test yiqiladi (`panic`).
- **`assert_eq!(chap, ong)`:** Ikki qiymat tengligini tekshiradi (`==`). Teng bo‘lmasa, ikkala qiymatni taqqoslab ko‘rsatadi.
- **`assert_ne!(chap, ong)`:** Ikki qiymat teng emasligini tekshiradi (`!=`).
- **Maxsus xato xabarlari:** Har bir makrosga qo‘shimcha formatlangan xabar berish mumkin:
  ```rust
  assert!(natija > 0, "Qiymat musbat bo‘lishi kerak edi, lekin {natija} olindi!");
  ```

---

## 2. Kutilgan Favqulodda Holatlarni Tekshirish: `#[should_panic]`

Ba’zida funksiyaning xato vaziyatda to‘g‘ri `panic!` qilayotganini tekshirish lozim bo‘ladi. Buning uchun **`#[should_panic]`** atributidan foydalaniladi:

```rust
pub struct Taxmin {
    qiymat: i32,
}

impl Taxmin {
    pub fn new(qiymat: i32) -> Taxmin {
        if qiymat < 1 || qiymat > 100 {
            panic!("Taxmin qilingan son 1 va 100 oralig‘ida bo‘lishi shart! Siz kiritdingiz: {qiymat}");
        }

        Taxmin { qiymat }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "1 va 100 oralig‘ida bo‘lishi shart")]
    fn yuzdan_katta_qiymat_panic_berishi_kerak() {
        Taxmin::new(200);
    }
}
```
`expected` parametri aynan qaysi xato xabari chiqqanini tekshirish orqali testning boshqa tasodifiy sabab bilan yiqilmaganligiga kafolat beradi.

---

## 3. `Result<T, E>` Qaytaruvchi Testlar

Agar test ichida `panic` o‘rniga `?` operatoridan foydalanishni istasangiz, test funksiyasining qaytish turini `Result<(), String>` deb belgilashingiz mumkin:

```rust
#[test]
fn bu_ishlaydi() -> Result<(), String> {
    if 2 + 2 == 4 {
        Ok(())
    } else {
        Err(String::from("Ikki karra ikki to‘rt bo‘lmadi!"))
    }
}
```

---

## 4. Testlarni Moslashuvchan Ishga Tushirish

Cargo testlarni ishga tushirish uchun juda ko‘p qulay bayroqlarga (flags) ega:

- **Barcha testlarni parallel emas, bitta oqimda ketma-ket yurgizish:**
  ```bash
  cargo test -- --test-threads=1
  ```
  (Agar testlar umumiy fayl yoki bitta bazaga murojaat qilayotgan bo‘lsa juda qo‘l keladi).

- **Muvaffaqiyatli testlarning ham `println!` chiqishlarini ko‘rish:**
  ```bash
  cargo test -- --nocapture
  ```

- **Faqat ma’lum bir testni nomi bo‘yicha ishga tushirish:**
  ```bash
  cargo test ikki_va_ikki
  ```

- **Sekin ishlaydigan testlarni e’tiborsiz qoldirish (`#[ignore]`):**
  ```rust
  #[test]
  #[ignore]
  fn sekin_ishlovchi_katta_test() { ... }
  ```
  Faqat `ignore` qilingan testlarni ishga tushirish uchun:
  ```bash
  cargo test -- --ignored
  ```

---

## 5. Testlarni Tashkil Etish: Birlik (Unit) vs Integratsion Testlar

Rust jamoasi testlarni ikki toifaga ajratadi:

### A. Birlik Testlari (Unit Tests)
- Kichik, alohida modullarni yakkalangan holda tekshiradi.
- Bevosita o‘sha kod yozilgan faylning pastida `#[cfg(test)] mod tests { ... }` ichida joylashadi.
- **Rust ning qulay tomoni:** Birlik testlari hatto `pub` qilinmagan maxfiy (private) funksiyalarni ham to‘g‘ridan-to‘g‘ri sinovdan o‘tkaza oladi!

### B. Integratsion Testlar (Integration Tests)
- Kutubxonaning bir nechta qismlari birgalikda to‘g‘ri ishlayotganini sinaydi.
- Tashqi foydalanuvchi ko‘zi bilan faqat ommaviy (`pub`) interfeyslarga murojaat qiladi.
- Loyiha ildizidagi **`tests/`** papkasida alohida fayllar sifatida joylashadi (masalan: `tests/integration_test.rs`).

```text
mening_loyiham/
├── Cargo.toml
├── src/
│   └── lib.rs       (bu yerda Unit testlar)
└── tests/
    └── api_test.rs  (bu yerda Integratsion testlar)
```

Integratsion test fayllari mustaqil crate kabi kompilyatsiya qilinadi, bu esa loyihangiz tashqi dunyo uchun qanchalik to‘g‘ri integratsiya qilinganini sinashning eng yaxshi usulidir.
