# Ma’lumotlar Turlari (Data Types)

Rust — **statik tipli til** (statically typed language). Bu degani, kompilyatsiya vaqtida dasturdagi har bir qiymatning aniq tipi ma’lum bo‘lishi shart.

Aksariyat hollarda Rust kompilyatori qiymatga qarab uning tipini o‘zi avtomatik aniqlay oladi (*type inference*). Ammo bir nechta tip mos kelishi mumkin bo‘lgan vaziyatlarda tipni aniq ko‘rsatishimiz kerak:

```rust
let guess: u32 = "42".parse().expect("Raqam emas!");
```

Rust ma’lumotlar turlari ikkita asosiy toifaga bo‘linadi:
1. **Skalyar turlar (Scalar Types)** — yagona qiymatni ifodalovchi turlar.
2. **Kompozit turlar (Compound Types)** — bir nechta qiymatni birlashtiruvchi turlar.

---

## 1. Skalyar Turlar (Scalar Types)

Rust da 4 ta asosiy skalyar tip mavjud:
- Butun sonlar (Integers)
- Suzuvchi nuqtali sonlar (Floating-point numbers)
- Mantiqiy qiymat (Boolean)
- Belgilar (Characters)

### A. Butun Sonlar (Integers)

Butun sonlar kasr qismi bo‘lmagan sonlardir. Ular ishorali (`i` harfi bilan boshlanadi — manfiy va musbat) yoki ishorasiz (`u` harfi bilan boshlanadi — faqat musbat) bo‘lishi mumkin:

| Uzunlik | Ishorali (Signed) | Ishorasiz (Unsigned) | Qiymatlar Oralig‘i (Ishorasiz) |
| :--- | :--- | :--- | :--- |
| **8-bit** | `i8` | `u8` | 0 dan 255 gacha |
| **16-bit** | `i16` | `u16` | 0 dan 65,535 gacha |
| **32-bit** | `i32` | `u32` | 0 dan 4,294,967,295 gacha |
| **64-bit** | `i64` | `u64` | 0 dan 18,446,744,073,709,551,615 gacha |
| **128-bit** | `i128` | `u128` | Juda katta sonlar |
| **arch** | `isize` | `usize` | Tizim arxitekturasiga bog‘liq (64-bit yoki 32-bit) |

> **Standart tanlov:** Agar aniq talab bo‘lmasa, Rust dasturchilari odatda **`i32`** turidan foydalanadilar. To‘plamlar va massivlarni indekslash uchun esa har doim **`usize`** ishlatiladi.

#### Butun Sonlarning To‘lib Ketishi (Integer Overflow)
Agar siz `u8` (0 dan 255 gacha) ga 256 qiymatini yozmoqchi bo‘lsangiz:
- **Debug rejimida:** Rust dasturni kompilyatsiya qiladi va ish vaqtida dastur `panic!` qilib to‘xtaydi.
- **Release rejimida (`--release`):** Rust xatolik bermaydi, balki ikki darajali modul bo‘yicha "aylanish" (two's complement wrapping) sodir bo‘ladi (masalan, 256 qiymati 0 ga aylanadi).

### B. Suzuvchi Nuqtali Sonlar (Floating-Point Numbers)

Rust kasrli sonlar uchun IEEE-754 standarti bo‘yicha ikkita asosiy turni taqdim etadi:
- `f32` — 32-bitli (yagona aniqlik - single precision);
- `f64` — 64-bitli (ikki karra aniqlik - double precision). Standart holatda `f64` ishlatiladi, chunki zamonaviy protsessorlarda u deyarli bir xil tezlikda ishlaydi va ancha aniqroqdir.

```rust
let x = 2.0; // f64 (standart)
let y: f32 = 3.0; // f32
```

### C. Mantiqiy Tip (Boolean)

Rust da boolean tipi `bool` bo‘lib, u faqat ikkita qiymatdan birini oladi: `true` yoki `false`. Xotirada 1 bayt joy egallaydi.

```rust
let t = true;
let f: bool = false; // aniq tur bilan
```

### D. Belgilar Turi (Character)

Rust da `char` bittalik belgini ifodalaydi va bir tirnoq (`'`) bilan yoziladi (ikkitalik tirnoq matnlar uchun ishlatiladi).

```rust
let c = 'z';
let z: char = 'ℤ';
let heart_eyed_cat = '😻';
```

Rust dagi `char` **4 bayt (32-bit)** hajmga ega bo‘lib, u **Unicode Skalyar Qiymatlarini** ifodalaydi. Bu degani siz unda nafaqat ingliz harflari, balki o‘zbekcha `o‘`, xitoy iyerogliflari, emojilar va aksentli harflarni to‘g‘ridan-to‘g‘ri saqlashingiz mumkin!

---

## 2. Kompozit Turlar (Compound Types)

Kompozit turlar bir nechta qiymatni bitta tur ichida birlashtirishga imkon beradi. Rust da ikkita asosiy birlamchi kompozit tur mavjud: **Kortejlar (Tuples)** va **Massivlar (Arrays)**.

### A. Kortejlar (Tuples)

Kortej turli xil ma’lumot turlariga ega bo‘lgan qiymatlarni bitta guruhga birlashtirish imkonini beradi. Kortejlar yaratilgandan so‘ng o‘lchami o‘zgarmaydi.

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);

    // Kortejni ochish (Destructuring):
    let (x, y, z) = tup;
    println!("y ning qiymati: {y}");

    // Yoki indeks orqali to‘g‘ridan-to‘g‘ri murojaat qilish:
    let besh_yuz = tup.0;
    let olti_butun_tort = tup.1;
    let bir = tup.2;
}
```

Hech qanday qiymatga ega bo‘lmagan bo‘sh kortej `()` — **Unit type** deb ataladi. Agar funksiya hech qanday qiymat qaytarmasa, u yashirincha `()` qaytaradi.

### B. Massivlar (Arrays)

Kortejdan farqli o‘laroq, massivdagi barcha elementlarning **tipi bir xil** bo‘lishi shart. Shuningdek, Rust dagi oddiy massivlarning uzunligi qat’iy (fixed) bo‘ladi.

```rust
let a = [1, 2, 3, 4, 5];
let b: [i32; 5] = [1, 2, 3, 4, 5]; // [tip; soni]
let c = [3; 5]; // [3, 3, 3, 3, 3] ni bildiradi
```

#### Xotirada Joylashishi:
Massivlar to‘liq **Stack** xotirasida ketma-ket bitta uzluksiz blok sifatida joylashadi:

```text
Stack:
+---+---+---+---+---+
| 1 | 2 | 3 | 4 | 5 |  -> [i32; 5] (jami 20 bayt)
+---+---+---+---+---+
```

#### Massiv Elementlariga Murojaat va Xavfsizlik:
```rust
let a = [10, 20, 30, 40, 50];
let first = a[0]; // 10
let second = a[1]; // 20
```

Agar siz mavjud bo‘lmagan indeksga (masalan, `a[10]`) murojaat qilishga urinsangiz, Rust dasturni xotira xavfsizligini himoya qilish uchun `panic!` orqali to‘xtatadi. C/C++ tillarida bu "Undefined Behavior" (kutilmagan xatti-harakat va xavfsizlik teshigi) ga olib keladi, lekin Rust da bu mutlaqo imkonsizdir!

> **Eslatma:** Agar sizga uzunligi dinamik ravishda o‘sib-kamayadigan to‘plam kerak bo‘lsa, massiv o‘rniga **Vektor (`Vec<T>`)** ishlatiladi (buni 8-bobda ko‘rib chiqamiz).
