# 9-Bob: Xatolar Bilan Ishlash (Error Handling)

Rust dasturlash tilining asosiy falsafasi — **ishonchlilik (reliability)**. Tizimli dasturlashda kutilmagan xatoliklar yuz berishi tabiiy, ammo Rust ularni yashirishga yoki e’tiborsiz qoldirishga aslo yo‘l qo‘ymaydi.

Ko‘pgina tillarda xatolar istisnolar (*Exceptions*: `try...catch`) orqali boshqariladi. Bu ko‘pincha qaysi funksiya xato tashlashi mumkinligini bilishni qiyinlashtiradi.
Rust da esa **istisnolar (exceptions) yo‘q!** Buning o‘rniga xatolar ikki toifaga bo‘linadi:

1. **Qaytarib bo‘lmaydigan xatolar (Unrecoverable Errors):** Dasturning davom etishi mumkin emas (masalan, massiv chegarasidan chiqib ketish). Bular **`panic!`** makrosi orqali hal qilinadi.
2. **Qaytarib bo‘ladigan xatolar (Recoverable Errors):** Odatdagi kutilgan xatolar (masalan, fayl topilmadi yoki tarmoq uzildi). Bular **`Result<T, E>`** enumi orqali ifodalanadi.

---

## 1. Qaytarib Bo‘lmaydigan Xatolar: `panic!`

Dasturda kutilmagan favqulodda holat yuz berganda `panic!` makrosi ishga tushadi:

```rust
fn main() {
    panic!("Katta avariya yuz berdi! Tizim to‘xtatildi.");
}
```

Dastur `panic!` qilganda:
1. Xato haqida xabar ekranga chiqariladi;
2. Dastur stakni yechadi (*unwinding the stack*) — ya’ni barcha xotiradagi o‘zgaruvchilar tozalanadi;
3. Dastur o‘z faoliyatini to‘xtatadi.

### Backtrace ni Ko‘rish
Agar xato qayerda sodir bo‘lganini aniqlamoqchi bo‘lsangiz, terminalda `RUST_BACKTRACE=1` muhit o‘zgaruvchisi bilan dasturni ishga tushirishingiz mumkin:

```bash
RUST_BACKTRACE=1 cargo run
```
Bu sizga chaqiruvlar stakini (Call Stack) to‘liq ko‘rsatib beradi.

---

## 2. Qaytarib Bo‘ladigan Xatolar: `Result<T, E>`

Ko‘p hollarda xatolik yuz berganda dasturni butunlay o‘ldirish shart emas. Masalan, agar kerakli fayl topilmasa, dastur yangi fayl yaratishi yoki foydalanuvchidan boshqa nom kiritishni so‘rashi mumkin.

Standart kutubxonada buning uchun `Result` tipi mavjud:

```rust
enum Result<T, E> {
    Ok(T),  // Muvaffaqiyatli qiymat
    Err(E), // Xatolik tafsiloti
}
```

### Fayl Ochish Misolida `match` Bilan Ishlash

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let fayl_natijasi = File::open("salom.txt");

    let fayl = match fayl_natijasi {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("salom.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Faylni yaratib bo‘lmadi: {e:?}"),
            },
            boshqa_xato => {
                panic!("Faylni ochishda xatolik: {boshqa_xato:?}");
            }
        },
    };
}
```

---

## 3. Qisqartmalar: `unwrap` va `expect`

Har safar `match` yozish kodni cho‘zib yuborishi mumkin. Sinov yoki prototiplar yozayotganda quyidagi qisqa metodlardan foydalanish qulay:

- **`.unwrap()`:** Agar natija `Ok` bo‘lsa qiymatni qaytaradi; agar `Err` bo‘lsa, dasturni avtomatik `panic!` qilib to‘xtatadi.
- **`.expect("Xabar")`:** Xuddi `unwrap` kabi ishlaydi, ammo `panic` bo‘lganda siz yozgan maxsus xabarni ko‘rsatadi. Bu ishlab chiqarish (production) kodida xatoni tez topish uchun tavsiya etiladi.

```rust
use std::fs::File;

let f = File::open("salom.txt").expect("salom.txt fayli ochilmadi! Iltimos, fayl mavjudligini tekshiring.");
```

---

## 4. Xatolarni Yuqoriga Uzatish: `?` Operatori (Error Propagation)

Agar funksiya ichida xato yuz bersa, uni shu yerning o‘zida to‘xtatmasdan, xatoni chaqirgan yuqori funksiyaga qaytarish odatiy amaliyotdir. Rust da buning uchun mo‘jizaviy **`?`** operatori bor!

Keling, fayldan matnni o‘qib qaytaruvchi funksiyani ko‘raylik:

```rust
use std::fs::File;
use std::io::{self, Read};

fn foydalanuvchi_ismini_oqish() -> Result<String, io::Error> {
    let mut fayl = File::open("ism.txt")?;
    let mut ism = String::new();
    fayl.read_to_string(&mut ism)?;
    Ok(ism)
}
```

### `?` Qanday Ishlaydi?
Agar operatsiya `Ok(qiymat)` qaytarsa, `?` operatori qiymatni ochib beradi va dastur keyingi satrga o‘tadi.
Agar `Err(xato)` qaytarsa, butun funksiya shu zahoti o‘z faoliyatini to‘xtatadi va o‘sha `Err` qiymatini funksiyadan chaqiruvchiga qaytaradi!

Buni zanjir uslubida (method chaining) yanada qisqartirish mumkin:

```rust
use std::fs::File;
use std::io::{self, Read};

fn foydalanuvchi_ismini_oqish() -> Result<String, io::Error> {
    let mut ism = String::new();
    File::open("ism.txt")?.read_to_string(&mut ism)?;
    Ok(ism)
}
```

Hatto standart kutubxonada bu vazifa uchun bir satrlik yechim ham bor:
```rust
use std::fs;
use std::io;

fn foydalanuvchi_ismini_oqish() -> Result<String, io::Error> {
    fs::read_to_string("ism.txt")
}
```

---

## 5. Qachon `panic!`, Qachon `Result` Ishlatish Kerak?

- **`Result` ishlatilishi kerak:** Xatolik kutilishi mumkin bo‘lgan har qanday holatda (tarmoq uzilishi, noto‘g‘ri kiritilgan ma’lumot, fayl tizimi xatolari). Siz funksiyangizdan foydalanuvchiga xatoni qanday hal qilish bo‘yicha tanlov berishingiz lozim.
- **`panic!` ishlatilishi mumkin:**
  - Misollar, testlar va tezkor prototiplar yozayotganda (`unwrap`, `expect`);
  - Dasturingiz qat’iy invariantlarga tayansa va bu qoida buzilsa (masalan, mantiqiy imkonsiz holatga tushib qolsangiz va bu dasturchining koddagi xatosi - bug ekanligini bildirsa).
