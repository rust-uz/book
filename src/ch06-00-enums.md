# 6-Bob: Sanamlar va Andaza Moslash (Enums & Pattern Matching)

**Sanamlar (Enums)** — qiymatning bir nechta mumkin bo‘lgan variantlardan biri ekanligini ifodalash imkonini beruvchi kuchli ma’lumot turidir.

Boshqa ko‘plab tillarda enumlar faqat nomlangan sonli o‘zgarmaslar ro‘yxatidan iborat bo‘lsa, Rust tilida enumlar ancha qudratli bo‘lib, **har bir variant o‘z ichida turli xil ma’lumotlarni saqlashi mumkin!**

---

## 1. Enum E’lon Qilish va Variantlariga Ma’lumot Biriktirish

Keling, tarmoq protokoli misolida enum yaratamiz:

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

fn main() {
    let uy = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));
}
```

E’tibor bering, bitta enum ichida `V4` to‘rtta `u8` sonini saqlasa, `V6` esa `String` matnini saqlaydi! Har bir variant butunlay o‘ziga xos tuzilishga ega bo‘lishi mumkin.

Hatto turli xil strukturalarni ham enum variantlari sifatida kiritish mumkin:

```rust
enum Message {
    Quit,                       // Ma'lumot yo'q
    Move { x: i32, y: i32 },    // Anonim struktura
    Write(String),              // Bitta String
    ChangeColor(i32, i32, i32), // Uchta i32
}

impl Message {
    fn call(&self) {
        // Enum uchun ham xuddi strukturalar kabi metodlar yozish mumkin!
    }
}
```

---

## 2. `null` O‘rniga `Option<T>` Turi

Dasturlash olamida `null` (qiymatning yo‘qligi) eng ko‘p xatoliklarni keltirib chiqaruvchi sabablardan biridir. Hatto uning ixtirochisi Toni Xoar (Tony Hoare) buni o‘zining **"Milliard dollarlik xatosi"** deb atagan.

Rust tilida `null` tushunchasi umuman mavjud emas! Buning o‘rniga standart kutubxonada **`Option<T>`** nomli enum taqdim etilgan:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

- Agar qiymat mavjud bo‘lsa: `Some(qiymat)`
- Agar qiymat yo‘q bo‘lsa: `None`

```rust
let ba_zi_raqam = Some(5);
let ba_zi_matn = Some("matn");
let yoq_raqam: Option<i32> = None;
```

### Nega bu `null` dan ancha xavfsiz?
Oddiy `i32` tipi har doim aniq songa ega ekanligiga 100% kafolat bor. Agar qiymat mavjud bo‘lmasligi mumkin bo‘lsa, siz uni `Option<i32>` deb e’lon qilishingiz shart. Rust sizni `Option<i32>` ni to‘g‘ridan-to‘g‘ri oddiy songa qo‘shishga yo‘l qo‘ymaydi — dastlab `None` holatini tekshirishingizni majbur qiladi!

---

## 3. `match` Boshqaruv Oqimi Operatori

`match` operatori berilgan qiymatni bir nechta andazalarga (patterns) solishtirish va mos kelgan kod blokini bajarish imkonini beradi:

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Omadli tanga!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### Qiymatlarni Bog‘lash (Binding Values)
Andazalar variant ichidagi ma’lumotlarni sug‘urib olish imkoniyatiga ega:

```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
}

enum Coin {
    Penny,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Quarter(state) => {
            println!("{state:?} shtatining chorakligi!");
            25
        }
    }
}
```

### `Option<T>` Bilan `match` Ishlatish

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let besh = Some(5);
let olti = plus_one(besh);
let hech_narsa = plus_one(None);
```

### To‘liq Qamrab Olish Majburiyati (Exhaustiveness)
Rust da `match` barcha mumkin bo‘lgan variantlarni ko‘rib chiqishi **shart**! Agar birorta variantni unutib qoldirsangiz, kod kompilyatsiya bo‘lmaydi.

Qolgan barcha holatlarni bitta qoidada qamrab olish uchun `other` yoki `_` (placeholder) belgisi ishlatiladi:

```rust
# fn fancy_hat_ber() {}
# fn shlyapani_olib_qoy() {}
# fn qayta_tashla() {}
let dice_roll = 9;
match dice_roll {
    3 => fancy_hat_ber(),
    7 => shlyapani_olib_qoy(),
    _ => qayta_tashla(), // Qolgan barcha sonlar uchun
}
```

---

## 4. `if let` Bilan Qisqartirilgan Nazorat

Agar sizga faqat bitta variant muhim bo‘lib, qolgan barcha holatlar e’tiborsiz bo‘lsa, uzun `match` o‘rniga ixcham **`if let`** sintaksisidan foydalaniladi:

```rust
let config_max = Some(3u8);

// match uslubi:
match config_max {
    Some(max) => println!("Maksimal qiymat: {max}"),
    _ => (),
}

// Yana ham ixcham if let uslubi:
if let Some(max) = config_max {
    println!("Maksimal qiymat: {max}");
} else {
    println!("Qiymat mavjud emas");
}
```
`if let` kodni toza va o‘qishli qilishga yordam beradi.
