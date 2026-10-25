# Ilova B: Operatorlar va Belgilar (Operators & Symbols)

Ushbu ilovada Rust dasturlash tilida qo‘llaniladigan barcha operatorlar va maxsus belgilar ularning vazifalari hamda misollari bilan tizimlashtirilgan.

---

## 1. Arifmetik va Mantiqiy Operatorlar

| Operator | Tavsifi | Misol |
| :--- | :--- | :--- |
| **`+`** | Qo‘shish amali; shuningdek trait boundlarni birlashtirish | `x + y`, `T: Clone + Debug` |
| **`-`** | Ayrish amali; inkor (manfiy son) | `x - y`, `-x` |
| **`*`** | Ko‘paytirish amali; dereference (ko‘rsatkichni ochish) | `x * y`, `*ptr` |
| **`/`** | Bo‘lish amali | `x / y` |
| **`%`** | Qoldiqli bo‘lish (modul) | `x % y` |
| **`&`** | Bit darajasidagi VA (AND); o‘zgarmas havola olish | `x & y`, `&x` |
| **`\|`** | Bit darajasidagi YOKI (OR); pattern matching varianti; klojur parametrlari chegarasi | `x \| y`, `1 \| 2`, `\|x\| x + 1` |
| **`^`** | Bit darajasidagi XOR (istisno qiluvchi YOKI) | `x ^ y` |
| **`!`** | Mantiqiy EMAS (NOT); makros belgisi; Never type | `!rost`, `println!`, `!` |
| **`<<`** | Bitlarni chapga siljitish | `x << 2` |
| **`>>`** | Bitlarni o‘ngga siljitish | `x >> 2` |
| **`&&`** | Mantiqiy qisqa tutashuvli VA (short-circuiting AND) | `a && b` |
| **`\|\|`** | Mantiqiy qisqa tutashuvli YOKI (short-circuiting OR) | `a \|\| b` |

---

## 2. Solishtirish va Biriktirish Operatorlari

| Operator | Tavsifi | Misol |
| :--- | :--- | :--- |
| **`=`** | Qiymat biriktirish; atribut qiymati | `let x = 5;`, `#[export_name = "test"]` |
| **`==`** | Tenglikni tekshirish (`PartialEq`) | `x == y` |
| **`!=`** | Teng emaslikni tekshirish | `x != y` |
| **`<`** | Kichik | `x < y` |
| **`>`** | Katta | `x > y` |
| **`<=`** | Kichik yoki teng | `x <= y` |
| **`>=`** | Katta yoki teng | `x >= y` |
| **`+=`, `-=`, `*=`, `/=`, `%=`** | Birikmali arifmetik amallar | `x += 1` |
| **`&=`, `\|=`, `^=`, `<<=`, `>>=`** | Birikmali bit amallari | `x &= 0b1111` |

---

## 3. Havola, Yo‘l va Maxsus Belgilar

| Belgisi | Tavsifi | Misol |
| :--- | :--- | :--- |
| **`&mut`** | O‘zgaruvchan havola olish yoki parametr turi | `&mut x`, `&mut String` |
| **`::`** | Modul, tip yoki enum variantiga yo‘l ajratgichi | `std::io::stdin`, `Option::Some` |
| **`.`** | Maydon yoki metodga murojaat | `foydalanuvchi.ism`, `x.len()` |
| **`..`** | Ochiq oraliq (range); strukturadagi qolgan maydonlarni to‘ldirish; andazada qolganlarni e’tiborsiz qoldirish | `1..5`, `Foydalanuvchi { ..user1 }`, `Point { x, .. }` |
| **`..=`** | Yopiq oraliq (oxirgi qiymat ham kiradi) | `1..=5` |
| **`?`** | Xatoni yuqoriga uzatish operatori; ixtiyoriy trait bound | `fayl.read()?`, `T: ?Sized` |
| **`->`** | Funksiya yoki klojur qaytish turi strelkasi | `fn hisobla() -> i32` |
| **`=>`** | `match` shoxobchasi strelkasi | `1 => println!("bir")` |
| **`@`** | Andazalarda qiymatni o‘zgaruvchiga bog‘lash (binding) | `id @ 1..=5` |
| **`_`** | E’tiborsiz qoldiriladigan andaza yoki tur inferensi | `_ => ()`, `let _: Vec<_> = ...` |
| **`'a`** | Yashash muddati (lifetime) yorlig‘i; tsikl yorlig‘i | `&'a str`, `'loop1: loop` |
| **`#`** | Ichki yoki tashqi atribut prefiksi | `#[derive(Debug)]`, `//! Hujjat` |
| **`$`** | Deklarativ makroslarda parametr o‘zgaruvchisi | `$x:expr` |
