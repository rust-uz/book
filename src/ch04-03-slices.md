# Kesmalar Turi (The Slice Type)

**Kesma (Slice)** — bu to‘plamning (masalan, matn yoki massiv) barchasiga emas, balki uning ketma-ket joylashgan ma’lum bir qismiga ko‘rsatuvchi havoladir. Kesmalar ham xuddi oddiy havolalar kabi egalik huquqiga ega emas.

---

## Muammo: Alohida Indekslar bilan Ishlash

Tasavvur qiling, matn ichidagi birinchi so‘zning oxirgi indeksini topuvchi funksiya yozmoqchimiz:

```rust
fn birinchi_soz_oxiri(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

Bu funksiya `usize` (masalan, `5`) qaytaradi. Ammo keyinchalik `s.clear()` orqali matnni tozalab tashlasak nima bo‘ladi? Bizdagi `5` indeksi mutlaqo o‘z ma’nosini yo‘qotadi, lekin u o‘zgaruvchida saqlanib qolaveradi! Dasturda bunday nomuvofiqliklar jiddiy xatoliklarni keltirib chiqaradi.

Rust bu muammoni hal qilish uchun **Satr Kesmalari (String Slices)**ni taqdim etadi.

---

## Satr Kesmalari (`&str`)

Satr kesmasi butun bir `String` ning qismiga havola hisoblanadi:

```rust
let s = String::from("salom dunyo");

let salom = &s[0..5];
let dunyo = &s[6..11];
```

Sintaksis: `[boshlangich_indeks..oxirgi_indeks]`. Oxirgi indeks oraliqqa kirmaydi.

```text
       STACK                          HEAP
s:
+----------+-------+           +---+---+---+---+---+---+---+---+---+---+---+
| ptr      |   *---|---------->| s | a | l | o | m |   | d | u | n | y | o |
| len      |  11   |           +---+---+---+---+---+---+---+---+---+---+---+
| capacity |  11   |                 ^                   ^
+----------+-------+                 |                   |
                                     |                   |
salom (&str):                        |                   |
+-----+-------+                      |                   |
| ptr |   *---|----------------------+                   |
| len |   5   |                                          |
+-----+-------+                                          |
                                                         |
dunyo (&str):                                            |
+-----+-------+                                          |
| ptr |   *---|------------------------------------------+
| len |   5   |
+-----+-------+
```

Xotirada kesma juda ixcham: u faqatgina boshlang‘ich baytning manzili (`ptr`) va uzunlik (`len`) dan iborat bo‘ladi.

### Qisqartirilgan Sintaksis:
- Boshidan boshlash: `&s[..2]` xuddi `&s[0..2]` bilan teng.
- Oxirigacha olish: `&s[3..]` xuddi `&s[3..len]` bilan teng.
- Butun satr: `&s[..]`.

---

## Muammoning Xavfsiz Yechimi

Endi birinchi so‘zni to‘g‘ridan-to‘g‘ri kesma ko‘rinishida qaytaruvchi funksiya yozamiz:

```rust
fn birinchi_soz(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let mut s = String::from("salom dunyo");

    let soz = birinchi_soz(&s);

    // s.clear(); // KOMPILYATSIYA XATOSI! 
    // Chunki `soz` o'zgarmas havola (&s) sifatida mavjud bo'lib turganda, 
    // `clear()` o'zgaruvchan havola (&mut s) olishga urinadi!

    println!("Birinchi so‘z: {soz}");
}
```

Rust ning borrow checkeri dasturimizni xatodan mukammal darajada himoya qildi!

---

## Satr Literallari Aslida Kesmalardir

Endi siz satr literallari qanday ishlashini to‘liq tushunishingiz mumkin:

```rust
let s = "Salom, dunyo!";
```

Bu yerda `s` ning tipi aslida **`&str`** dir! U dasturning binar kodi (binary executable) ichiga yozilgan matnga ko‘rsatuvchi o‘zgarmas kesmadir.

### Eng Yaxshi Amaliyot (Best Practice)
Funksiyalarga parametr yozayotganda `&String` o‘rniga `&str` dan foydalanish ancha professional yondashuvdir:
```rust,ignore
fn birinchi_soz(s: &str) -> &str { ... }
```
Chunki bunday funksiyaga ham `String` ning kesmasini (`&my_string[..]`), ham oddiy satr literalini (`"salom"`) birdek uzatish mumkin bo‘ladi.

---

## Boshqa Kesmalar: Massiv Kesmalari

Kesmalar faqat satrlar uchun emas, balki massivlar uchun ham amal qiladi:

```rust
let a = [1, 2, 3, 4, 5];

let kesma: &[i32] = &a[1..3];

assert_eq!(kesma, &[2, 3]);
```
`&[i32]` tipi massivning bir qismiga havola bo‘lib, xuddi satr kesmasi kabi ko‘rsatkich va uzunlikdan iborat bo‘ladi.
