# 18-Bob: Pattern va Matching (Patterns & Pattern Matching)

**Andazalar (Patterns)** — bu Rust tilidagi eng kuchli va o‘ziga xos mexanizmlardan biri bo‘lib, ular ma’lumotlarning tuzilishini tekshirish va ulardan kerakli qismlarni sug‘urib olish uchun xizmat qiladi.

Andazalar quyidagilardan tashkil topishi mumkin:
- Literallar (masalan, `1`, `'a'`);
- Qismlarga ajratilgan (destructured) massivlar, kortejlar, strukturalar yoki enumlar;
- O‘zgaruvchilar va oraliqlar (`1..=5`);
- Qo‘shimcha shartlar (Match Guards).

---

## 1. Andazalar Qayerlarda Ishlatiladi?

Andazalar nafaqat `match` iboralarida, balki Rust tilining deyarli barcha qismlarida qo‘llaniladi:

### A. `match` Shoxobchalarida
```rust
# let qiymat = 1;
match qiymat {
    1 => println!("Bir"),
    2 | 3 => println!("Ikki yoki Uch"),
    _ => println!("Boshqa"),
}
```

### B. `if let` va `while let` Iboralarida
```rust
let mut navbat = vec![1, 2, 3];

// To to'plam bo'shaguncha elementlarni olish:
while let Some(yuqori) = navbat.pop() {
    println!("{yuqori}");
}
```

### C. `for` Tsikllarida
```rust
let nuqtalar = vec![(0, 0), (1, 5), (10, 20)];

for (x, y) in &nuqtalar {
    println!("x = {x}, y = {y}");
}
```

### D. `let` Ko‘rsatmalarida
Biz har doim ishlatadigan `let` ko‘rsatmasi ham aslida andazadir!
```rust
let (x, y, z) = (1, 2, 3);
```

### E. Funksiya Parametrlarida
```rust
fn nuqtani_chop_et(&(x, y): &(i32, i32)) {
    println!("Koordinata: ({x}, {y})");
}
```

---

## 2. Inkor Etiluvchanlik (Refutability)

Rust da andazalar ikkita toifaga bo‘linadi:
1. **Inkor etilmaydigan (Irrefutable):** Berilgan har qanday qiymatga so‘zsiz mos keladigan andazalar. Masalan: `let x = 5;` — `x` har doim mos keladi.
2. **Inkor etilishi mumkin bo‘lgan (Refutable):** Ba’zi qiymatlarga mos kelmay qolishi mumkin bo‘lgan andazalar. Masalan: `Some(x)` — chunki qiymat `None` bo‘lib qolishi mumkin.

- `let` va funksiya parametrlari faqat **irrefutable** andazalarni qabul qiladi.
- `if let` va `while let` esa **refutable** andazalar bilan ishlash uchun yaratilgan.

---

## 3. Andazalar Sintaksisining Ilg‘or Imkoniyatlari

### A. Bir Nechta Andazalar (`|`) va Oraliqlar (`..=`)
```rust
let x = 3;

match x {
    1 | 2 => println!("1 yoki 2"),
    3..=5 => println!("3 dan 5 gacha bo‘lgan son"),
    _ => println!("Boshqa son"),
}

let harf = 'c';
match harf {
    'a'..='j' => println!("Alifboning boshidagi harf"),
    _ => println!("Boshqa harf"),
}
```

### B. Struktura va Kortejlarni Destrukturizatsiya Qilish
```rust
struct Point {
    x: i32,
    y: i32,
}

let p = Point { x: 0, y: 7 };

match p {
    Point { x, y: 0 } => println!("X o‘qida joylashgan: {x}"),
    Point { x: 0, y } => println!("Y o‘qida joylashgan: {y}"),
    Point { x, y } => println!("({x}, {y}) koordinatasida"),
}
```

### C. Keraksiz Qiymatlarni E’tiborsiz Qoldirish (`_` va `..`)
- `_` — bitta qiymatni tashlab ketish;
- `_nom` — ishlatilmagan o‘zgaruvchi haqidagi ogohlantirishni (warning) o‘chirish;
- `..` — qolgan barcha ko‘p sonli maydonlarni birdaniga e’tiborsiz qoldirish:

```rust
struct Nuqta3D {
    x: i32,
    y: i32,
    z: i32,
}

let nuqta = Nuqta3D { x: 1, y: 2, z: 3 };

match nuqta {
    Nuqta3D { x, .. } => println!("Faqat x muhim: {x}"),
}
```

### D. Qo‘shimcha Shartlar: Match Guards (`if`)
Andazaga qo‘shimcha erkin mantiqiy shart qo‘shish mumkin:

```rust
let son = Some(4);

match son {
    Some(x) if x % 2 == 0 => println!("{x} — juft son!"),
    Some(x) => println!("{x} — toq son!"),
    None => (),
}
```

### E. Qiymatni Bog‘lash: `@` Operatori
`@` operatori andaza mos kelganligini tekshirish bilan bir vaqtda uning qiymatini o‘zgaruvchiga saqlab olish imkonini beradi:

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello {
        id: id_ozgaruvchisi @ 3..=7,
    } => println!("ID 3 va 7 oralig‘ida topildi: {id_ozgaruvchisi}"),
    Message::Hello { id: 10..=12 } => {
        println!("Oraliqda, lekin ID qiymati alohida olinmadi");
    }
    Message::Hello { id } => println!("Boshqa ID: {id}"),
}
```

Rust dagi pattern matching mexanizmi kodni bir vaqtning o‘zida ham ixcham, ham barcha mumkin bo‘lgan holatlarni qamrab oluvchi xatosiz holga keltiradi.
