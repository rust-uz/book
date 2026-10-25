# 8-Bob: Umumiy Kolleksiyalar (Vectors, Strings, HashMaps)

Oddiy massivlar va kortejlardan farqli o‘laroq, Rust standart kutubxonasidagi **kolleksiyalar (collections)** o‘z ichiga bir nechta qiymatlarni oladi va ularning ma’lumotlari **Heap** xotirasida saqlanadi. Bu degani ularning hajmi dastur davomida dinamik tarzda kattalashishi yoki kichrayishi mumkin.

Ushbu bobda eng ko‘p qo‘llaniladigan uchta asosiy kolleksiyani ko‘rib chiqamiz:
1. **Vektor (`Vec<T>`):** Bir xil turdagi qiymatlarni ketma-ket saqlovchi dinamik massiv.
2. **Matn (`String`):** UTF-8 formatida kodlangan belgilar to‘plami.
3. **Xesh-jadval (`HashMap<K, V>`):** Kalitlarni qiymatlarga moslashtiruvchi assotsiativ massiv.

---

## 1. Vektorlar (`Vec<T>`)

Vektor barcha elementlarini xotirada bir-birining yonida ketma-ket saqlaydi.

### Yaratish va Element Qo‘shish
```rust
// Bo'sh vektor yaratish:
let mut v: Vec<i32> = Vec::new();

// Elementlar qo'shish:
v.push(1);
v.push(2);
v.push(3);

// vec! makrosi orqali boshlang'ich qiymatlar bilan yaratish:
let v2 = vec![1, 2, 3, 4, 5];
```

### Vektor Elementlarini O‘qish
Vektor elementlariga murojaat qilishning ikkita usuli bor:

```rust
let v = vec![1, 2, 3, 4, 5];

// 1. Indeks orqali (to'g'ridan-to'g'ri havola):
let uchinchi: &i32 = &v[2];
println!("Uchinchi element: {uchinchi}");

// 2. get metodi orqali (Option<&T> qaytaradi):
let uchinchi_variant: Option<&i32> = v.get(2);
match uchinchi_variant {
    Some(qiymat) => println!("Topildi: {qiymat}"),
    None => println!("Bunday indeks mavjud emas!"),
}
```

> **Farqi:** Agar siz `&v[100]` deb mavjud bo‘lmagan indeksga murojaat qilsangiz, dastur `panic!` qilib to‘xtaydi. Ammo `.get(100)` usulidan foydalansangiz, u xavfsizgina `None` qaytaradi va dastur qulamasdan davom etadi.

### Egalik va Vektorlar
Vektor ustida iteratsiya qilayotganda borrow checker xavfsizlikni qattiq nazorat qiladi:

```rust
let mut v = vec![1, 2, 3, 4, 5];

let birinchi = &v[0]; // O'zgarmas havola olinmoqda

// v.push(6); // KOMPILYATSIYA XATOSI!

println!("Birinchi: {birinchi}");
```
*Nega bu xato?* Chunki `push(6)` yangi element qo‘shayotganda xotira sig‘imi to‘lib qolgan bo‘lsa, vektor Heap dagi butun massivni boshqa yangi va kattaroq xotira maydoniga ko‘chirishi mumkin. Agar shunday bo‘lsa, `birinchi` ko‘rsatkichi eski bo‘shatilgan xotiraga ko‘rsatib qolardi (*dangling pointer*). Rust bu xatoni ildizi bilan yo‘q qiladi!

### Vektor Ustida Tsikl Bilan Aylanish
```rust
let mut v = vec![100, 32, 57];

for i in &mut v {
    *i += 50; // * orqali dereference qilib qiymatni o'zgartiramiz
}
```

---

## 2. Satrlar (`String` va `&str`)

Rust da satrlar boshqa tillarga nisbatan murakkabroq ko‘rinishi mumkin, chunki Rust satrlarning **UTF-8 xavfsizligini** birinchi o‘ringa qo‘yadi.

### Yaratish va Yangilash
```rust
let mut s = String::from("Salom");
s.push_str(", dunyo"); // kesma qo'shish
s.push('!');           // bitta belgi qo'shish

// format! makrosi orqali satrlarni birlashtirish:
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{s1}-{s2}-{s3}"); // s1, s2, s3 ning egaligi saqlanib qoladi
```

### Nega Rust Satrlarni Indekslashga (`s[0]`) Ruxsat Bermaydi?

Ko‘plab dasturchilar nima uchun `let h = s[0];` deb yoza olmasliklariga hayron bo‘lishadi.
Sababi: Rust dagi barcha `String` lar **UTF-8** da saqlanadi!
- Inglizcha harflar (ASCII) xotirada 1 bayt joy egallaydi.
- O‘zbekcha `o‘` yoki kirill, arab harflari 2 bayt joy egallaydi.
- Emojilar va xitoycha belgilar 3 yoki 4 bayt joy egallaydi.

Agar matn `String::from("Salom")` bo‘lsa, u 5 bayt. Ammo `String::from("Здравствуйте")` bo‘lsa, u 12 ta harf bo‘lishiga qaramay, xotirada 24 bayt joy oladi! Agar siz `s[0]` qilsangiz, butun harfni emas, harfning yarim baytini olib qo‘ygan bo‘lardingiz. Bu esa ma’lumotlarning buzilishiga olib keladi.

### Satrlar Bo‘ylab Qanday Yurish Kerak?
Aniq nima kerakligiga qarab:

1. **Har bir harf (Unicode Skalyar Qiymat) bo‘yicha:**
```rust
for c in "Зд".chars() {
    println!("{c}"); // 'З', keyin 'д'
}
```

2. **Har bir xom bayt bo‘yicha:**
```rust
for b in "Зд".bytes() {
    println!("{b}"); // 208, 151, 208, 180...
}
```

---

## 3. Xesh-Jadval (`HashMap<K, V>`)

`HashMap<K, V>` ma’lumotlarni kalit (`K`) va unga mos qiymat (`V`) juftligi ko‘rinishida saqlaydi. Kalit orqali qiymatni tezkor $O(1)$ qidirib topish uchun xeshlash funksiyasidan foydalanadi.

### Foydalanish Asoslari
`HashMap` prelude da mavjud emas, shuning uchun uni import qilamiz:

```rust
use std::collections::HashMap;

fn main() {
    let mut ballar = HashMap::new();

    // Ma'lumot qo'shish:
    ballar.insert(String::from("Moviy"), 10);
    ballar.insert(String::from("Sariq"), 50);

    // Qiymatni olish:
    let jamoa_nomi = String::from("Moviy");
    let ball = ballar.get(&jamoa_nomi).copied().unwrap_or(0);
    println!("Moviy jamoa bali: {ball}");

    // Barcha elementlarni aylanib chiqish:
    for (kalit, qiymat) in &ballar {
        println!("{kalit}: {qiymat}");
    }
}
```

### Faqat Mavjud Bo‘lmasa Qo‘shish (`entry`)
`entry` API si orqali kalit mavjudligini tekshirib, agar mavjud bo‘lmasa yangi qiymat kiritish juda qulay:

```rust
# use std::collections::HashMap;
let mut ballar = HashMap::new();
ballar.insert(String::from("Moviy"), 10);

ballar.entry(String::from("Sariq")).or_insert(50);
ballar.entry(String::from("Moviy")).or_insert(50); // Bu o'zgarmaydi, chunki Moviy allaqachon bor!
```

### Amaliy Misol: Matndagi So‘zlar Soni
```rust
use std::collections::HashMap;

let matn = "olma anor olma nok anor olma";
let mut hisob = HashMap::new();

for soz in matn.split_whitespace() {
    let sanagich = hisob.entry(soz).or_insert(0);
    *sanagich += 1;
}

println!("{hisob:#?}");
// Natija: {"olma": 3, "anor": 2, "nok": 1}
```
Bu uchta kolleksiya Rust dasturlashining kundalik amaliyotida eng ko‘p qo‘llaniladigan poydevor tuzilmalardir.
