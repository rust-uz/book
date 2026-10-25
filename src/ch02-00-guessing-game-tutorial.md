# 2-Bob: Taxmin Qilish O‘yini Dasturlash (Guessing Game)

Rust tilini o‘rganishning eng yaxshi usuli — amaliy dastur tuzishdir! Ushbu bobda biz kompyuter o‘ylagan sirli sonni topishga asoslangan qiziqarli terminal o‘yinini noldan birgalikda yaratamiz.

Bu loyiha orqali siz Rust tilining bir qancha fundamental tushunchalari bilan yaqindan tanishasiz:
- `let`, `mut` va o‘zgaruvchilar;
- Standart kutubxona bilan ishlash (`std::io`);
- Havolalar (`references`) va xotira xavfsizligi;
- Tashqi paketlar (crates) qo‘shish va boshqarish;
- Pattern matching (`match`) orqali xatoliklar va shartlarni tekshirish;
- Tsikllar (`loop`) va dastur oqimini nazorat qilish.

---

## 1. Yangi Loyiha Yaratish

Dastlab, o‘yinimiz uchun yangi Cargo loyihasini ochamiz. Terminalda quyidagi buyruqni bajaring:

```bash
cargo new guessing_game
cd guessing_game
```

`Cargo.toml` va `src/main.rs` fayllari yaratiladi. `src/main.rs` faylini oching va uning ichidagi kodni quyidagiga almashtiring:

```rust
use std::io;

fn main() {
    println!("Raqamni toping o‘yiniga xush kelibsiz!");

    println!("Iltimos, taxminingizni kiriting:");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Satrni o‘qib bo‘lmadi!");

    println!("Siz kiritgan taxmin: {guess}");
}
```

Dasturni ishga tushirib ko‘ring:

```bash
cargo run
```

Dastur sizdan son kiritishni so‘raydi va kiritgan matningizni qaytarib ko‘rsatadi.

---

## 2. Kodni Qatorma-Qator Tahlil Qilamiz

Keling, yozgan har bir satrimiz orqasida nimalar yotganini batafsil tushunib olaylik.

### Standart Kirish/Chiqish Kutubxonasi
```rust
use std::io;
```
Rust dasturga faqat eng zarur tushunchalarni avtomatik tarzda yuklaydi (bunga **prelude** deyiladi). Agar prelude ro‘yxatida bo‘lmagan funksionallik (masalan, foydalanuvchidan ma’lumot o‘qish) kerak bo‘lsa, uni `use` kalit so‘zi orqali aniq ko‘rsatamiz. `std::io` — kirish va chiqish bilan ishlovchi standart moduldir.

### O‘zgaruvchilar va O‘zgaruvchanlik (Mutability)
```rust
let mut guess = String::new();
```
Rust tilida barcha o‘zgaruvchilar **odatiy holatda o‘zgarmas (immutable)** bo‘ladi! Agar o‘zgaruvchining qiymatini keyinchalik o‘zgartirmoqchi bo‘lsak, uning oldiga **`mut`** kalit so‘zini qo‘yishimiz shart.

* `let apples = 5;` // o‘zgarmas (immutable)
* `let mut bananas = 5;` // o‘zgaruvchan (mutable)

`String::new()` — bo‘sh va dinamik o‘suvchi yangi matn (String) obyektini yaratadi. `::new` sintaksisi `String` turining bog‘langan funksiyasi (associated function yoki konstruktori) ekanligini bildiradi. 

#### Xotira Modeli:
`String` turi Stack va Heap xotiralaridan foydalanadi:
- **Stack**da ko‘rsatkich (pointer), uzunlik (`len: 0`), va sig‘im (`capacity: 0`) saqlanadi.
- **Heap**da esa foydalanuvchi kiritadigan belgilarning haqiqiy baytlari joylashadi.

```text
Stack                      Heap
+----------+-------+      +---+---+---+---+
| ptr      |   *---|----->|   |   |   |   | ...
| len      |   0   |      +---+---+---+---+
| capacity |   0   |
+----------+-------+
```

### Foydalanuvchidan Matn O‘qish
```rust,ignore
io::stdin()
    .read_line(&mut guess)
    .expect("Satrni o‘qib bo‘lmadi!");
```
- `io::stdin()` — terminalning standart kirish oqimiga havolani beradi.
- `.read_line(&mut guess)` — foydalanuvchi klaviaturada kiritgan matnni oladi va uni `guess` o‘zgaruvchisiga yozadi.
- **`&` (Havola / Reference):** Ushbu belgi ma’lumotdan nusxa olmasdan, xotiradagi mavjud manzilga xavfsiz murojaat qilish imkonini beradi. Havolalar ham odatda o‘zgarmas bo‘lgani uchun, `&guess` emas, balki **`&mut guess`** deb yoziladi.

### Xatoliklar bilan Ishlash (`Result` va `expect`)
`read_line` metodi `Result` enum turini qaytaradi (`Ok` yoki `Err`).
Agar operatsiya muvaffaqiyatsiz bo‘lsa (masalan, operatsion tizim kirish oqimini uzib qo‘ysa), `Result` `Err` holatida bo‘ladi.
`.expect(...)` metodi agar natija `Err` bo‘lsa, dasturni to‘xtatadi (`panic!`) va parametrda berilgan xabarni ekranga chiqaradi. Agar `expect`ni chaqirmasangiz, Rust kompilyatori sizga ogohlantirish (warning) beradi: Rust hech qachon xatoliklarni e’tiborsiz qoldirishga yo‘l qo‘ymaydi!

---

## 3. Tasodifiy Son Yaratish (`rand` Crate)

Rust standart kutubxonasida tasodifiy sonlar generatori yo‘q, chunki Rust jamoasi standart kutubxonani imkon qadar yengil va barqaror saqlashni maqsad qilgan. Buning o‘rniga hamjamiyat tomonidan tayyorlangan rasmiy **`rand`** paketidan (crate) foydalanamiz.

`Cargo.toml` faylidagi `[dependencies]` bo‘limiga quyidagilarni yozing:

```toml
[dependencies]
rand = "0.8.5"
```

Endi `src/main.rs` faylimizni yangilaymiz:

```rust,ignore
use std::io;
use rand::Rng;

fn main() {
    println!("Raqamni toping o‘yiniga xush kelibsiz!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("Sirli son: {secret_number}"); // Test uchun ko‘rib turamiz

    println!("Iltimos, taxminingizni kiriting:");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Satrni o‘qib bo‘lmadi!");

    println!("Siz kiritgan taxmin: {guess}");
}
```

`rand::thread_rng()` oqimga xos bo‘lgan generatorni ishga tushiradi, `gen_range(1..=100)` esa 1 dan 100 gacha (100 ham kiradi) oraliqdagi tasodifiy sonni hosil qiladi.

---

## 4. Taxminni Sirli Son Bilan Solishtirish

Endi foydalanuvchi kiritgan son bilan sirli sonni solishtirishimiz lozim. Buning uchun Rust ning kuchli vositasi — **`match`** va `std::cmp::Ordering` dan foydalanamiz.

```rust,ignore
use std::cmp::Ordering;
use std::io;
use rand::Rng;

fn main() {
    println!("Raqamni toping o‘yiniga xush kelibsiz!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("Iltimos, taxminingizni kiriting:");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Satrni o‘qib bo‘lmadi!");

    // Satrni (String) butun songa (u32) aylantiramiz (Shadowing)
    let guess: u32 = guess.trim().parse().expect("Iltimos, faqat musbat son kiriting!");

    println!("Siz kiritgan taxmin: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Kattaroq son ayting!"),
        Ordering::Greater => println!("Kichikroq son ayting!"),
        Ordering::Equal => println!("Tabriklaymiz, siz yutdingiz! 🎉"),
    }
}
```

### O‘zgaruvchini Soyalash (Shadowing)
E’tibor bering, bizda avval `guess` nomli `String` o‘zgaruvchi bor edi. Keyin esa:
```rust,ignore
let guess: u32 = guess.trim().parse().expect(...);
```
orqali yana `guess` nomli yangi o‘zgaruvchi yaratdik. Rust bu jarayonga **Shadowing** (soyalash) deb ataydi. Bu o‘zgaruvchilar nomini `guess_str` yoki `guess_num` deb ko‘paytirmasdan, bir xil mantiqiy nomni qayta ishlatish imkonini beradi.
- `.trim()` — satr oxiridagi bo‘sh joylar va `\n` (Enter belgisi)ni olib tashlaydi.
- `.parse()` — matnni ko‘rsatilgan tipga (`u32`) o‘giradi.

---

## 5. Cheksiz Tsikl va G‘alaba Bilan Chiqish

O‘yin bir martagina so‘rab to‘xtab qolmasligi, to foydalanuvchi to‘g‘ri topmaguncha davom etishi kerak. Buning uchun `loop` va `break` dan foydalanamiz. Shuningdek, agar foydalanuvchi harf kiritib qo‘ysa, dastur qulab tushmasdan (`panic` bo‘lmasdan), xatolikni e’tiborsiz qoldirib, yangi son so‘rashi uchun `parse()` natijasini `match` bilan tekshiramiz.

Mana to‘liq va yakuniy kodimiz:

```rust,ignore
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("========================================");
    println!("🎯 RAQAMNI TOPING O‘YINIGA XUSH KELIBSIZ!");
    println!("1 dan 100 gacha bo‘lgan son o‘ylandi.");
    println!("========================================");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("\nIltimos, o‘ylagan soningizni kiriting:");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Satrni o‘qib bo‘lmadi!");

        // Xatolik yuz bersa, dastur to‘xtamasdan tsiklni qayta boshlaydi
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                println!("⚠️  Iltimos, faqat raqam kiriting!");
                continue;
            }
        };

        println!("Sizning taxminingiz: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("⬆️  Kattaroq son ayting!"),
            Ordering::Greater => println!("⬇️  Kichikroq son ayting!"),
            Ordering::Equal => {
                println!("🏆 TABRIKLAYMIZ! Siz g‘alaba qozondingiz! 🥳");
                break;
            }
        }
    }
}
```

---

## Bob Xulosasi

Ushbu amaliy bobda siz:
1. `let` va `mut` bilan o‘zgarmas va o‘zgaruvchan qiymatlarni farqlashni;
2. `String::new()` va `io::stdin()` orqali xotirada dinamik matn bilan ishlashni;
3. `&` va `&mut` orqali havolalar berish tamoyilini;
4. Tashqi kutubxonalarni (`rand`) `Cargo.toml` orqali ulab foydalanishni;
5. `match` va `Ordering` yordamida nafis shartli mantiq qurishni;
6. `loop`, `continue` va `break` orqali tsikllarni boshqarishni o‘rgandingiz.

Keyingi boblarda biz ushbu tushunchalarni yanada chuqurlashtirib, Rust tilining ichki mexanizmlarini qadamma-qadam tahlil qilamiz.
