# 13-Bob: Funksional Xususiyatlar: Klojurlar va Iteratorlar (Closures & Iterators)

Rust tili faqat tizimli dasturlash imkoniyatlari bilan cheklanib qolmaydi — u funksional dasturlash (Functional Programming) tamoyillarini ham o‘zida chuqur mujassam etgan.

Ushbu bobda ikkita asosiy funksional qurolni o‘rganamiz:
1. **Klojurlar (Closures):** O‘zgaruvchida saqlanishi va atrofdagi muhitdan qiymatlarni tutib olishi mumkin bo‘lgan anonim funksiyalar.
2. **Iteratorlar (Iterators):** Ketma-ket elementlar to‘plamini samarali va nafis qayta ishlash usuli.

---

## 1. Klojurlar (Closures)

Klojurlar — bu o‘zgaruvchiga biriktirish yoki boshqa funksiyalarga argument sifatida uzatish mumkin bo‘lgan nomsiz funksiyalardir. Funksiyalardan (`fn`) farqli o‘laroq, klojurlar **o‘zlari e’lon qilingan ko‘rinish sohasidagi (scope) o‘zgaruvchilarni tutib olish (capture qilish)** xususiyatiga ega.

### Klojur Sintaksisi
Parametrlar ikkita vertikal chiziq (`| |`) ichida yoziladi:

```rust
let qoshish = |x: i32| x + 1;

// Tipni yozish majburiy emas (Rust o'zi tushunib oladi):
let qisqa_qoshish = |x| x + 1;

let natija = qisqa_qoshish(5); // 6
```

### Muhitni Tutib Olish Usullari
Klojurlar atrofdagi qiymatlarni uch xil usulda tutib olishi mumkin (bu bevosita 3 ta traitga mos keladi):

1. **`Fn` (O‘zgarmas havola orqali):** Qiymatni faqat o‘qish uchun oladi (`&T`).
2. **`FnMut` (O‘zgaruvchan havola orqali):** Qiymatni o‘zgartirish huquqi bilan qarz oladi (`&mut T`).
3. **`FnOnce` (Egalikni ko‘chirish orqali):** Qiymatning butun egaligini o‘zlashtiradi va shuning uchun kamida bir marta chaqirilishi mumkin.

```rust
let mut royxat = vec![1, 2, 3];

// FnMut misoli:
let mut qosh = || royxat.push(7);
qosh();
println!("{royxat:?}"); // [1, 2, 3, 7]
```

### `move` Kalit So‘zi Bilan Egalikni O‘tkazish
Agar klojurning boshqa oqimga (thread) o‘tishini yoki qarz olingan ma’lumotning egasiga aylanishini istasangiz, klojurning oldiga **`move`** so‘zini qo‘shasiz:

```rust
let royxat = vec![1, 2, 3];

let ega_klojur = move || {
    println!("Men egalik qilaman: {royxat:?}");
};

ega_klojur();
// println!("{royxat:?}"); // XATO: royxat klojur ichiga ko'chib ketgan!
```

---

## 2. Iteratorlar (Iterators)

Rust da iterator — bu ketma-ketlikdagi har bir element bo‘yicha navbatma-navbat harakatlanish imkonini beruvchi andazadir. 

Rust ning barcha iteratorlari standart kutubxonadagi `Iterator` traitini amalga oshiradi:

```rust
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```

Har safar `.next()` chaqirilganda, u `Some(element)` qaytaradi, to‘plam tugagach esa `None` beradi.

```rust
let v1 = vec![1, 2, 3];
let mut v1_iter = v1.iter();

assert_eq!(v1_iter.next(), Some(&1));
assert_eq!(v1_iter.next(), Some(&2));
assert_eq!(v1_iter.next(), Some(&3));
assert_eq!(v1_iter.next(), None);
```

### Iterator Hosil Qilishning 3 Ta Usuli:
- **`.iter()`:** Elementlarning o‘zgarmas havolalari (`&T`) ustida aylanadi.
- **`.iter_mut()`:** O‘zgaruvchan havolalar (`&mut T`) ustida aylanadi.
- **`.into_iter()`:** To‘plamning butun egaligini o‘zlashtirib (`T`), elementlarni iste’mol qiladi.

---

## 3. Iterator Adapterlari (Iterator Adapters)

Iteratorlar juda kuchli metodlarga ega. Ular ikkiga bo‘linadi:

### A. Iste’molchi Adapterlar (Consuming Adaptors)
Iteratorni to‘liq aylanib chiqib, yagona natija qaytaradi (masalan, `.sum()`):

```rust
let v1 = vec![1, 2, 3];
let jami: i32 = v1.iter().sum();
assert_eq!(jami, 6);
```

### B. O‘zgartiruvchi Adapterlar (Iterator Adaptors)
Mavjud iterator asosida yangi iterator hosil qiladi. Ular **dangasa (lazy)** bo‘lib, to natija talab qilinmaguncha (`.collect()` chaqirilmaguncha) hisob-kitob bajarilmaydi:

```rust
let v1: Vec<i32> = vec![1, 2, 3];

// Har bir elementga 1 qo'shish va yangi vektorga yig'ish:
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
assert_eq!(v2, vec![2, 3, 4]);

// Faqat juft sonlarni filtrlash:
let juftlar: Vec<_> = v1.into_iter().filter(|x| x % 2 == 0).collect();
assert_eq!(juftlar, vec![2]);
```

---

## 4. `minigrep` Loyihasini Iteratorlar Bilan Qayta Yozish

Oldingi bobdagi `search` funksiyamizni eslang. Uni imperativ `for` tsikli o‘rniga deklarativ iteratorlar bilan qanchalik nafis yozish mumkinligiga qarang:

```rust
// Eski usul (ch012):
// pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
//     let mut results = Vec::new();
//     for line in contents.lines() {
//         if line.contains(query) {
//             results.push(line);
//         }
//     }
//     results
// }

// Zamonaviy funksional usul:
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents
        .lines()
        .filter(|line| line.contains(query))
        .collect()
}
```
Kod 8 qatordan bor-yo‘g‘i 4 qatorga qisqardi va uning nima qilayotgani bir qarashdayoq tushunarli bo‘ldi!

---

## 5. Tezlik Tahlili: Tsikllar vs Iteratorlar (Zero-cost Abstraction)

Ko‘pchilik dasturchilar "Iteratorlar yuqori darajadagi abstraksiya bo‘lgani uchun oddiy `for` tsiklidan sekinroq ishlasa kerak" deb o‘ylashadi.

**Ammo Rust da bu mutlaqo unday emas!**
Bjarne Stroustrup ta’riflaganidek:
> *"Siz ishlatmagan narsangiz uchun pul to‘lamaysiz. Va siz qo‘lda yozishingiz mumkin bo‘lgan koddan ham yaxshiroq natijaga erishasiz."*

Rust kompilyatori iterator zanjirlarini tahlil qilib, ularni barcha qo‘shimcha tekshiruvlardan xoli bo‘lgan to‘g‘ridan-to‘g‘ri assemblerdagi bitta zich mashina tsikliga aylantiradi (hatto massiv chegarasi tekshiruvlarini olib tashlaydi). Shu sababli Rust da iteratorlar ko‘pincha oddiy indeksli tsikldan ham **tezroq** ishlaydi!
