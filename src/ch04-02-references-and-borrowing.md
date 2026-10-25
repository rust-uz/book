# Qarz Olish va Havolalar (References and Borrowing)

Oldingi bo‘limda ko‘rganimizdek, agar biz o‘zgaruvchini funksiyaga argument sifatida uzatsak, uning egalik huquqi funksiyaga o‘tib ketadi (Move bo‘ladi). Funksiya tugagach esa qiymat o‘chib ketadi.

Agar biz qiymatning egalik huquqini bermasdan, undan faqat vaqtincha foydalanmoqchi bo‘lsak-chi? Har safar qiymatni qaytarib olish uchun uni `return` qilish juda noqulay bo‘lardi.

Rust buning uchun **Havolalar (References)** va **Qarz olish (Borrowing)** mexanizmini taqdim etadi.

---

## Havolalar (References)

Havola — bu xotirada boshqa bir o‘zgaruvchining manziliga ko‘rsatuvchi, lekin unga egalik qilmaydigan vositadir. Havola yaratish uchun o‘zgaruvchi oldiga **`&`** belgisi qo‘yiladi:

```rust
fn main() {
    let s1 = String::from("salom");

    let uzunlik = uzunlikni_hisobla(&s1);

    println!("'{s1}' ning uzunligi: {uzunlik}");
}

fn uzunlikni_hisobla(s: &String) -> usize {
    s.len()
} // s ko'rinish sohasidan chiqadi, lekin u s1 ga egalik qilmagani uchun xotira tozalanmaydi!
```

```text
  &s1 (Havola)               s1 (Ega)                      HEAP
+-----+-------+        +----------+-------+         +---+---+---+---+---+
| ptr |   *---|------->| ptr      |   *---|-------->| s | a | l | o | m |
+-----+-------+        | len      |   5   |         +---+---+---+---+---+
                       | capacity |   5   |
                       +----------+-------+
```

Funksiya parametr sifatida havolani qabul qilishiga Rust da **Qarz olish (Borrowing)** deyiladi. Xuddi hayotda biror narsani qarzga olganingiz kabi, ishlatib bo‘lgach uni egasiga qaytarasiz.

### Qarz Olingan Narsani O‘zgartirib Bo‘lmaydi!
Odatiy holatda barcha havolalar o‘zgarmas (immutable) bo‘ladi:

```rust
fn o_zgartir(s: &String) {
    s.push_str(", dunyo"); // XATO: cannot borrow `*s` as mutable
}
```

---

## O‘zgaruvchan Havolalar (Mutable References)

Agar qarz olingan ma’lumotni o‘zgartirish zarur bo‘lsa, **`&mut`** sintaksisidan foydalanamiz:

```rust
fn main() {
    let mut s = String::from("salom");

    o_zgartir(&mut s);

    println!("{s}"); // "salom, dunyo"
}

fn o_zgartir(s: &mut String) {
    s.push_str(", dunyo");
}
```

---

## Qarz Olishning Oltin Qoidasi (The Golden Rule)

Rust da xotira bilan bog‘liq xatoliklarni 100% yo‘qotuvchi juda qat’iy qoida bor:

> **Ixtiyoriy vaqtda ma’lum bir ma’lumotga nisbatan:**
> 1. YOKI istalgancha **o‘zgarmas havola (`&T`)** bo‘lishi mumkin;
> 2. YOKI faqat bittagina **o‘zgaruvchan havola (`&mut T`)** bo‘lishi mumkin.
> 
> **Ammo ikkalasi BIR VAQTDA mavjud bo‘lishi MUMKIN EMAS!**

### Nega bir vaqtda ikkita `&mut` bo‘lishi mumkin emas?

Agar bitta ma’lumotga bir vaqtning o‘zida ikkita ko‘rsatkich o‘zgartirish kirita olsa, bu **Ma’lumotlar Poygasi (Data Race)**ga olib keladi:
- Ikki yoki undan ortiq ko‘rsatkich ayni bir vaqtda bitta xotira bo‘lagiga kirishadi;
- Ulardan kamida bittasi ma’lumotni yozmoqda;
- Ko‘rsatkichlar o‘rtasida hech qanday sinxronizatsiya mexanizmi yo‘q.

Rust bu xatolikni hali dastur ishga tushmasidanoq, kompilyatsiya jarayonida to‘xtatadi!

```rust
let mut s = String::from("salom");

let r1 = &mut s;
let r2 = &mut s; // KOMPILYATSIYA XATOSI! cannot borrow `s` as mutable more than once at a time

println!("{r1}, {r2}");
```

### O‘zgarmas va O‘zgaruvchan Havolalarni Aralashtirish
Xuddi shuningdek, kimdir ma’lumotni o‘qiyotgan paytda boshqa birov uni o‘zgartirib yuborishi mumkin emas:

```rust
let mut s = String::from("salom");

let r1 = &s; // muammo yo'q (o'qish)
let r2 = &s; // muammo yo'q (o'qish)
let r3 = &mut s; // KOMPILYATSIYA XATOSI! cannot borrow `s` as mutable because it is also borrowed as immutable

println!("{r1}, {r2}, {r3}");
```

---

## Muallaq Havolalar (Dangling References)

C va C++ kabi tillarda xotiradan o‘chib ketgan o‘zgaruvchining manzilini saqlab qolish oson. Bunga **Dangling Pointer (Muallaq ko‘rsatkich)** deyiladi.

Rust da kompilyator hech qachon muallaq havola yaratilishiga yo‘l qo‘ymaydi:

```rust
fn main() {
    let havola = muallaq();
}

fn muallaq() -> &String { // XATO: bu funksiya o'chib ketadigan ma'lumotga havola qaytarmoqchi!
    let s = String::from("salom");

    &s
} // s xotiradan o'chadi, binobarin &s bo'shliqqa ko'rsatib qolardi!
```

Kompilyator ushbu xatoni ko‘rganda: *"this function's return type contains a borrowed value, but there is no value for it to be borrowed from"* deya ogohlantiradi. To‘g‘ri yechim — havolani emas, to‘g‘ridan-to‘g‘ri `String` ning o‘zini (egalikni) qaytarishdir.
