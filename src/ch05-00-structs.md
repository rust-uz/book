# 5-Bob: Strukturalar Yordamida Bog‘liq Ma’lumotlarni Tuzish (Structs)

**Struktura (Struct)** — bu bir nechta o‘zaro bog‘liq qiymatlarni bitta ma’noli guruhga birlashtirish imkonini beruvchi maxsus ma’lumot turidir. Agar siz obyektga yo‘naltirilgan tillar (masalan, C++, Java, Python) bilan ishlagan bo‘lsangiz, structlar obyektning atributlariga (xususiyatlariga) o‘xshaydi.

Rust da strukturalarning uch turi mavjud:
1. Klassik nomlangan maydonli strukturalar;
2. Kortej strukturalar (Tuple Structs);
3. Maydoni bo‘lmagan birlik strukturalar (Unit-Like Structs).

---

## 1. Struktura E’lon Qilish va Yaratish

Klassik strukturani e’lon qilish uchun `struct` kalit so‘zi va har bir maydonning nomi hamda tipi yoziladi:

```rust
struct Foydalanuvchi {
    faol: bool,
    foydalanuvchi_nomi: String,
    email: String,
    kirishlar_soni: u64,
}

fn main() {
    let mut foydalanuvchi1 = Foydalanuvchi {
        faol: true,
        foydalanuvchi_nomi: String::from("iskurama"),
        email: String::from("rustuz@example.com"),
        kirishlar_soni: 1,
    };

    // Maydonga murojaat qilish va o'zgartirish (nuqta orqali):
    foydalanuvchi1.email = String::from("yangi_email@example.com");
}
```

> **Eslatma:** Rust da butun instansiya yoki to‘liq o‘zgaruvchan (`let mut`), yoki to‘liq o‘zgarmas bo‘ladi. Alohida bitta maydonning o‘zini `mut` qilib belgilash imkonsizdir.

### Maydon Nomini Qisqartirish (Field Init Shorthand)
Agar o‘zgaruvchi nomi bilan struktura maydonining nomi bir xil bo‘lsa, uni qisqartirib yozish mumkin:

```rust
fn foydalanuvchi_yarat(email: String, foydalanuvchi_nomi: String) -> Foydalanuvchi {
    Foydalanuvchi {
        faol: true,
        foydalanuvchi_nomi, // foydalanuvchi_nomi: foydalanuvchi_nomi o'rniga
        email,              // email: email o'rniga
        kirishlar_soni: 1,
    }
}
```

### Strukturalarni Yangilash Sintaksisi (Struct Update Syntax)
Mavjud struktura asosida yangi nusxa yaratish va faqat ma’lum maydonlarni o‘zgartirish uchun `..` sintaksisidan foydalaniladi:

```rust
let foydalanuvchi2 = Foydalanuvchi {
    email: String::from("boshqa@example.com"),
    ..foydalanuvchi1 // Qolgan barcha maydonlar foydalanuvchi1 dan ko'chiriladi
};
```
*Diqqat:* Ushbu amaldan keyin `foydalanuvchi1` ning Heap dagi maydonlari (`foydalanuvchi_nomi`) yangi strukturaga ko‘chadi (Move bo‘ladi). Shuning uchun `foydalanuvchi1` dan boshqa foydalanib bo‘lmaydi.

---

## 2. Kortej Strukturalar va Birlik Strukturalar

### Kortej Strukturalar (Tuple Structs)
Maydonlarga alohida nom bermasdan, faqat ularning turlari muhim bo‘lgan hollarda ishlatiladi:

```rust
struct Rang(i32, i32, i32);
struct Nuqta(i32, i32, i32);

fn main() {
    let qora = Rang(0, 0, 0);
    let nuqta = Nuqta(0, 0, 0);

    println!("Qizil rang qiymati: {}", qora.0);
}
```
Garchi `Rang` va `Nuqta` bir xil tiplardan (`i32, i32, i32`) iborat bo‘lsa-da, ular mutlaqo alohida turlardir — birining o‘rniga ikkinchisini uzatib bo‘lmaydi!

### Maydoni Bo‘lmagan Birlik Strukturalar (Unit-Like Structs)
Hech qanday ma’lumot saqlamaydigan, ammo ma’lum bir xatti-harakatni (trait) ifodalash uchun xizmat qiladigan strukturalar:

```rust
struct DoimiyBirlik;
```

---

## 3. Amaliy Misol: To‘g‘ri To‘rtburchak Yuzini Hisoblash

Keling, strukturalarning afzalligini geometrik shakl misolida ko‘rib chiqaylik:

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1: {rect1:#?}");
    println!("Yuzasi: {} piksel", area(&rect1));
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

### `#[derive(Debug)]` va `dbg!`
Strukturalarni konsolga chop etish uchun uning ustiga `#[derive(Debug)]` atributi qo‘shiladi va `println!("{rect1:#?}")` orqali chiroyli formatda ko‘rish mumkin.
Shuningdek, qiymatni tekshirish uchun qulay `dbg!(&rect1)` makrosi mavjud bo‘lib, u kod qatorini va natijasini to‘g‘ridan-to‘g‘ri terminalga chiqaradi.

---

## 4. Metodlar Sintaksisi (Method Syntax)

Metodlar xuddi funksiyalarga o‘xshaydi, lekin ular **struktura doirasida** (`impl` blokida) e’lon qilinadi va ularning birinchi parametri har doim **`self`** bo‘ladi.

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // Struktura metodlari:
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }

    // Bog'langan funksiya (Konstruktor - self qabul qilmaydi):
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let kvadrat = Rectangle::square(25);

    println!("rect1 yuzasi: {}", rect1.area());
    println!("rect1 rect2 ni o‘z ichiga sig‘dira oladimi? {}", rect1.can_hold(&rect2));
    println!("Kvadrat: {kvadrat:#?}");
}
```

### `&self` Nimani Anglatadi?
- `&self` — bu `self: &Self` ning qisqartmasi bo‘lib, strukturadan o‘zgarmas havola oladi.
- Agar metod struktura qiymatlarini o‘zgartirishi kerak bo‘lsa, `&mut self` ishlatiladi.
- Agar metod strukturani butunlay o‘zlashtirishi (iste’mol qilishi) kerak bo‘lsa, `self` (egalik o‘tadi) yoziladi.

### Bog‘langan Funksiyalar (Associated Functions)
`self` parametriga ega bo‘lmagan funksiyalarga bog‘langan funksiyalar deyiladi (masalan, `Rectangle::square`). Ular odatda yangi obyektlarni yaratuvchi konstruktor vazifasini bajaradi va ularga murojaat qilishda ikki nuqta (`::`) ishlatiladi.
