# Funksiyalar Qanday Ishlaydi (How Functions Work)

Funksiyalar har qanday dasturning asosiy bloklaridan biridir. Rust dasturining asosiy kirish nuqtasi — bu har doim **`main`** funksiyasidir.

Yangi funksiyalarni e’lon qilish uchun **`fn`** kalit so‘zidan foydalaniladi. Rust da funksiyalar va o‘zgaruvchilar nomlari **`snake_case`** uslubida yoziladi (barcha harflar kichik va so‘zlar pastki chiziq `_` bilan ajratiladi).

```rust
fn main() {
    println!("Asosiy funksiya ishga tushdi.");
    boshqa_funksiya();
}

fn boshqa_funksiya() {
    println!("Bu boshqa funksiya!");
}
```

Rust da funksiyaning qayerda e’lon qilinishi muhim emas — uni `main` dan oldin yoki keyin yozishingiz mumkin; asosiysi, u ko‘rinish sohasida mavjud bo‘lsa kifoya.

---

## Parametrlar (Arguments & Parameters)

Funksiyalarga parametrlar uzatishingiz mumkin. Rust tilida **har bir parametrning ma’lumot turi aniq ko‘rsatilishi shart**:

```rust
fn main() {
    chop_et(5, 'm');
}

fn chop_et(qiymat: i32, birlik: char) {
    println!("Natija: {qiymat}{birlik}");
}
```

---

## Ko‘rsatmalar (Statements) va Iboralar (Expressions)

Rust — **iboralarga asoslangan til** (expression-based language). Boshqa tillardan kelgan dasturchilar uchun bu juda muhim nozik farqdir:

1. **Ko‘rsatma (Statement):** Biror amalni bajaradigan, lekin **hech qanday qiymat qaytarmaydigan** ko‘rsatma.
   - `let y = 6;` — bu statement.
   - Statementlar oxiriga har doim nuqta-vergul (`;`) qo‘yiladi.
   - Rust da siz `let x = (let y = 6);` deb yoza olmaysiz, chunki `let y = 6` hech qanday qiymat qaytarmaydi!

2. **Ibora (Expression):** Hisoblanadigan va **natijada biror qiymat beradigan** kod parchasi.
   - Matematik amallar: `5 + 6` — bu 11 qiymatini beruvchi ifoda.
   - Funksiyani chaqirish: `boshqa_funksiya()` — ifoda.
   - Yangi blok (`{}`): blokning o‘zi ham ibora bo‘lishi mumkin!

Quyidagi misolga diqqat bilan qarang:

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1 // DIQQAT: Oxirida nuqta-vergul YO‘Q!
    };

    println!("y ning qiymati: {y}"); // y = 4 bo‘ladi
}
```

> **MUHIM QOIDA:** Agar ifoda oxiriga nuqta-vergul (`;`) qo‘ysangiz, u iboradan ko‘rsatmaga (statement) aylanadi va hech qanday qiymat qaytarmaydi (aniqrog‘i, unit type `()` qaytaradi).

---

## Qiymat Qaytaruvchi Funksiyalar

Funksiya biror qiymat qaytarsa, uning qaytish tipi **`->`** (strelka) orqali e’lon qilinadi:

```rust
fn besh() -> i32 {
    5 // Yashirin qaytarish (implicit return) - nuqta-vergulsiz!
}

fn birni_qosh(x: i32) -> i32 {
    x + 1
}

fn main() {
    let x = besh();
    let y = birni_qosh(x);

    println!("y ning qiymati: {y}"); // 6
}
```

### Ochiq Qaytarish (`return`)
Odatda funksiyaning eng oxirgi ifodasi uning qaytaradigan qiymati hisoblanadi. Biroq funksiyani vaqtidan oldin to‘xtatib, qiymat bilan chiqib ketish kerak bo‘lsa, **`return`** kalit so‘zidan foydalanish mumkin:

```rust
fn tekshir(son: i32) -> bool {
    if son < 0 {
        return false; // Erta chiqish
    }

    son > 10 // Oxirgi ifoda
}
```
