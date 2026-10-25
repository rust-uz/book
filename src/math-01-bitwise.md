# 1-Modul: Protsessor Arifmetikasi va Ikkilik Mantiq (Bitwise Math)

Tizimli dasturlash va past darajali tizimlar (kernel, drayver, kriptografik dvigatellar) poydevori bevosita protsessor arifmetikasi hamda ikkilik mantiqqa tayanadi. Yuqori darajali abstraksiyalar ostida kompyuter apparaturasi faqat bitlar ($0$ va $1$) hamda mantiqiy klapanlar (AND, OR, XOR, NOT) bilan ishlaydi. 

Ushbu modulda biz ikkilik sanoq tizimining qat'iy matematik qonuniyatlarini, integer overflow fenomenini va bitwise amallar yordamida $O(1)$ vaqtda ishlovchi algoritmlarni o'rganamiz.

---

## 1. Butun sonlar: Ikkilik to‘ldiruvchi (Two's complement) matematik isboti

Kompyuter apparaturasida manfiy sonlarni ifodalash dastlab bir qancha muammolarni keltirib chiqargan. Dastlabki hisoblash mashinalarida **Sign-Magnitude** (eng yuqori bit faqat ishora uchun) yoki **Ones' Complement** (barcha bitlarni invertirlash) usullari qo'llangan. Ammo bu usullarda ikkita nol muammosi ($+0$ va $-0$) mavjud bo'lib, apparat darajasida ayirish va qo'shish uchun alohida murakkab mantiqiy sxemalar talab etilar edi.

Ushbu muammoni hal qilgan universal matematik yechim — **Two's complement** (ikkilik to'ldiruvchi) tizimidir.

### 1.1. Matematik Isbot

$n$-bitli so'z uzunligiga ega bo'lgan kompyuter arxitekturasida barcha butun sonli amallar qoldiqli arifmetika halqasida (ring) bajariladi:
$$\mathbb{Z}_{2^n} = \mathbb{Z} / 2^n \mathbb{Z}$$

Bu yerda har qanday son $2^n$ moduli bo'yicha hisoblanadi.

#### Teorema:
Ixtiyoriy $n$-bitli son $x$ ning additiv teskarisi (ya'ni $-x$) modulli arifmetikada uning bitwise inversiyasi ($\sim x$) ga $1$ qo'shish orqali hosil qilinadi:
$$-x \equiv (\sim x + 1) \pmod{2^n}$$

#### Isbot:
1. $n$-bitli sanoq tizimida barcha bitlari $1$ ga teng bo'lgan son:
   $$\sum_{i=0}^{n-1} 2^i = 2^n - 1$$
   Ikkilik ko'rinishda bu $\underbrace{111\dots 111_2}_{n \text{ ta bir}}$.

2. Ixtiyoriy $x$ soni va uning bitma-bit inkor qilingan qiymati $\sim x$ (bit-wise NOT) ni olamiz. Har bir pozitsiyadagi bit uchun $b_i + (1 - b_i) = 1$ bo'lganligi sababli, ularning arifmetik yig'indisi har doim barcha bitlari $1$ bo'lgan sonni hosil qiladi:
   $$x + (\sim x) = 2^n - 1$$

3. Ikkala tomonga $1$ qo'shamiz:
   $$x + (\sim x) + 1 = 2^n$$

4. Ushbu ifodani $\pmod{2^n}$ halqasida ko'rib chiqamiz:
   $$x + [(\sim x) + 1] \equiv 2^n \equiv 0 \pmod{2^n}$$

5. Modulli arifmetikada agar $a + b \equiv 0 \pmod m$ bo'lsa, $b$ element $a$ ning additiv teskarisi, ya'ni $-a$ hisoblanadi:
   $$-x \equiv \sim x + 1 \pmod{2^n}$$
   Q.E.D. (Isbotlandi).

#### Yagona Nol va Assimetrik Diapazon:
- **Yagona nol:** $0$ soni uchun $\sim 0 = 2^n - 1$. Unga $1$ qo'shsak: $(2^n - 1) + 1 = 2^n \equiv 0 \pmod{2^n}$. Nolning manfiy ko'rinishi yana $0$ ning o'ziga teng! Ikkita nol anomaliyasi yo'qoladi.
- **Diapazon:** $n$-bitli ishorali butun sonning diapazoni:
  $$[-2^{n-1}, \; 2^{n-1} - 1]$$
  Masalan, 8-bitli `i8` uchun: $[-128, \; 127]$. Manfiy tomonda musbat tomonga qaraganda bitta ko'p son mavjud, chunki $0$ musbatlar kabi $0$-boshlanuvchi bitlar maydonidan joy oladi.

### 1.2. Algoritm: Apparat Darajasida Ayirish
Two's complement tufayli arifmetik-mantiqiy qurilma (ALU) ga alohida ayirgich (subtractor) sxemasi kerak emas. Ayirish amali quyidagicha qo'shishga keltiriladi:
$$A - B = A + (-B) = A + (\sim B + 1)$$
Buni bajarish uchun ALU ikkinchi operandning bitlarini teskarisiga o'giradi (NOT darvozalari orqali) va qo'shgichning birinchi kirishiga kiritiluvchi Carry-In ($C_{in}$) qiymatiga $1$ beradi.

### 1.3. Rust Xotira Modeli
Rust tilida butun sonlar stack yoki registrlarda aniq o'lchamga ega bo'lgan baytlar ketma-ketligi sifatida saqlanadi:
- `i8` (1 bayt, 8 bit)
- `i16` (2 bayt, 16 bit)
- `i32` (4 bayt, 32 bit)
- `i64` (8 bayt, 64 bit)
- `i128` (16 bayt, 128 bit)
- `isize` (maqsadli arxitekturaning pointer o'lchami: 32 yoki 64 bit)

Zamonaviy protsessorlar (x86_64 va ARM64) xotirada **Little-Endian** tartibidan foydalanadi: sonning eng quyi ahamiyatli bayti (LSB - Least Significant Byte) eng kichik xotira manzilida joylashadi.

```text
Xotira manzili:      0x00     0x01     0x02     0x03
i32 = 0x12345678    [0x78]   [0x56]   [0x34]   [0x12]  (Little-Endian)
```

### 1.4. Amaliy Kod

Quyidagi Rust dasturi Two's complement xossalarini, bayt tartibini va bitwise inkor orqali manfiy son hosil bo'lishini matematik jihatdan tekshiradi:

```rust
fn twos_complement_negate(val: i8) -> i8 {
    // !val — Rustda bitwise NOT (inversiya)
    // wrapping_add(1) — 1 qo'shish
    (!val).wrapping_add(1)
}

fn main() {
    let positive: i8 = 42;
    let negative: i8 = twos_complement_negate(positive);

    // 42 ning ikkilik ko'rinishi:  00101010
    // !42 (inversiya):            11010101
    // !42 + 1:                    11010110 (-42)
    assert_eq!(negative, -42);
    assert_eq!(positive.wrapping_add(negative), 0);

    // Nolning inkorini tekshiramiz
    assert_eq!(twos_complement_negate(0), 0);

    // Little-endian xotira tartibini ko'ramiz
    let num: i32 = -42;
    let le_bytes = num.to_le_bytes();
    // i32 da -42 = 0xFFFFFFD6
    assert_eq!(le_bytes, [0xD6, 0xFF, 0xFF, 0xFF]);
}
```

---

## 2. Integer Overflow: Nega kompyuterda 127 + 1 = -128 bo‘ladi?

Dasturlashda eng ko'p uchraydigan xatolardan biri to'lib ketish (overflow) hodisasidir. 8-bitli signed turlar uchun $127 + 1 = -128$ bo'lishi sehr emas, balki qat'iy ikkilik arifmetika natijasidir.

### 2.1. Matematik Isbot

8-bitli `i8` sonining eng yuqori biti (7-indeksdagi bit $b_7$) ishora biti (MSB - Most Significant Bit) hisoblanadi. Two's complement sistemasida har bir bitning vazni quyidagicha aniqlanadi:
$$V(b) = -b_7 \cdot 2^7 + \sum_{i=0}^6 b_i \cdot 2^i$$

Endi $127$ sonini qaraymiz:
$$127_{10} = 01111111_2$$
Vaznlar bo'yicha:
$$V = -0 \cdot 2^7 + (2^6 + 2^5 + 2^4 + 2^3 + 2^2 + 2^1 + 2^0) = 0 + 127 = 127$$

Ushbu songa arifmetik $1 = 00000001_2$ ni qo'shamiz:
$$\begin{array}{r@{\quad}l}
  01111111_2 & (127) \\
+ 00000001_2 & (1) \\
\hline
  10000000_2 & (?)
\end{array}$$

Hosil bo'lgan $10000000_2$ kodini Two's complement formulasi bo'yicha hisoblaymiz:
$$V = -1 \cdot 2^7 + \sum_{i=0}^6 0 \cdot 2^i = -128 + 0 = -128$$

Modulli arifmetika nuqtai nazaridan:
$$127 + 1 = 128 \equiv 128 - 256 = -128 \pmod{256}$$
Demak, 8-bitli halqada $128$ soni mavjud emas, uning o'rnini unga ekvivalent bo'lgan $-128$ egallaydi.

### 2.2. Algoritm: ALU Bayroqlari (Flags)
Protsessorning ALU qismi har bir arifmetik amal bajarganda holat registriga (Flags register) maxsus bitlarni o'rnatadi:
1. **Carry Flag (CF):** Unsigned arifmetikada eng yuqori bitdan tashqariga toshib ketish bo'lganini bildiradi.
2. **Overflow Flag (OF):** Signed arifmetikada natijaning ishorasi noto'g'ri bo'lib qolganini bildiradi.

#### Overflow sharti:
Ikkita bir xil ishorali son qo'shilganda qarama-qarshi ishorali natija hosil bo'lsa, to'lib ketish ro'y beradi:
$$\text{OF} = (A_{\text{sign}} == B_{\text{sign}}) \land (A_{\text{sign}} \neq S_{\text{sign}})$$
Mantiqiy eshiklar darajasida bu eng yuqori bitga kiruvchi carry ($C_{in}$) va undan chiquvchi carry ($C_{out}$) ning XOR amali bilan aniqlanadi:
$$\text{OF} = C_{n-1} \oplus C_n$$

### 2.3. Rust Xotira Modeli va Xavfsizlik Strategiyasi
C va C++ tillarida signed integer overflow rasman **Undefined Behavior (UB)** hisoblanadi. Kompilyator bunday holatda kodni optimizatsiya niqobi ostida butunlay buzib yuborishi mumkin.

Rust tili tizimli xavfsizlikka alohida e'tibor qaratadi:
- **Debug rejimida (`cargo build`):** Kompilyator har bir arifmetik amal ketidan apparat darajasidagi overflow tekshiruvlarini qo'shadi. Agar overflow yuz bersa, dastur darhol `panic!` chaqiradi.
- **Release rejimida (`cargo build --release`):** Tezlikni maksimal qilish uchun tekshiruvlar olib tashlanadi va Two's complement wrapping qonuni bo'yicha hisob-kitob davom etadi.

Rust muhandisga o'z niyatini kodda aniq ifodalash uchun 4 ta eksplitsit metodlar oilasini taqdim etadi:
1. `wrapping_*`: Qat'iy ikki to'ldiruvchi moduli bo'yicha wrapping bajaradi (hech qachon panic qilmaydi).
2. `checked_*`: Xavfsiz `Option<T>` qaytaradi (`None` agar overflow bo'lsa).
3. `overflowing_*`: `(T, bool)` kortejini qaytaradi (natija va to'lib ketish bayrog'i).
4. `saturating_*`: Chegaralarga yopishadi (minimum yoki maksimum qiymatda to'xtaydi, audio va grafik signallarni qayta ishlashda ideal).

### 2.4. Amaliy Kod

```rust
fn main() {
    let max_val: i8 = 127;

    // 1. Wrapping: 127 + 1 = -128
    let wrapped = max_val.wrapping_add(1);
    assert_eq!(wrapped, -128);

    // 2. Checked: Xatolikni Option orqali xavfsiz boshqarish
    let checked = max_val.checked_add(1);
    assert_eq!(checked, None);

    // 3. Overflowing: Natija va apparatdagi OF flag analogi
    let (res, has_overflowed) = max_val.overflowing_add(1);
    assert_eq!(res, -128);
    assert!(has_overflowed);

    // 4. Saturating: Yuqori chegarada qotib qolish (clipping)
    let saturated = max_val.saturating_add(10);
    assert_eq!(saturated, 127);

    // Minimum chegara bilan tekshirish
    let min_val: i8 = -128;
    assert_eq!(min_val.saturating_sub(1), -128);
    assert_eq!(min_val.wrapping_sub(1), 127);
}
```

---

## 3. Bitwise amallar orqali $O(1)$ tezlikda matematik fokuslar (XOR xossalari, Hamming weight)

Bitwise amallar protsessorning bitta taktida (single CPU cycle) hech qanday tarmoqlanishlarsiz (branchless) bajariladigan eng tezkor amallardir.

### 3.1. XOR ($\oplus$) ning Matematik Xossalari
XOR amali Abel guruhini tashkil etadi:
1. **Kommutativlik:** $a \oplus b = b \oplus a$
2. **Assotsiativlik:** $a \oplus (b \oplus c) = (a \oplus b) \oplus c$
3. **Neytral element:** $a \oplus 0 = a$
4. **O'ziga teskarilik (Involutsiya):** $a \oplus a = 0$

#### Klassik Teorema (Yolg'iz elementni topish):
Agar massivda bitta sondan tashqari barcha sonlar juft marta (masalan, 2 martadan) qatnashgan bo'lsa, massivning barcha elementlari ketma-ket XOR qilinsa, natija aynan o'sha yolg'iz son bo'ladi.
$$S = (x_1 \oplus x_1) \oplus (x_2 \oplus x_2) \oplus \dots \oplus y = 0 \oplus 0 \oplus \dots \oplus y = y$$

#### XOR Swap Isboti:
Qo'shimcha xotirasiz ikki sonning o'rnini almashtirish:
1. $x' = x \oplus y$
2. $y' = x' \oplus y = (x \oplus y) \oplus y = x \oplus (y \oplus y) = x \oplus 0 = x$
3. $x'' = x' \oplus y' = (x \oplus y) \oplus x = (x \oplus x) \oplus y = 0 \oplus y = y$

### 3.2. Hamming Weight (Population Count) Isboti
Hamming weight — berilgan binar so'zdagi birlar ($1$) sonidir.

#### Brian Kernighan Algoritmi Matematikasi:
Har bir qadamda $x \ \& \ (x - 1)$ amali $x$ ning eng o'ngdagi (eng kichik ahamiyatli) $1$ bitini $0$ ga aylantiradi.

**Isbot:**
Ixtiyoriy sonni ikkilik ko'rinishda tasvirlaymiz:
$$x = A \, 1 \, \underbrace{00\dots 0}_{k \text{ ta nol}}$$
Bu yerda $A$ — sonning yuqori bitlari, $1$ — eng o'ngdagi o'rnatilgan bit, undan keyin $k$ ta nol bor.
Endi $x - 1$ amalini bajaramiz:
$$x - 1 = A \, 0 \, \underbrace{11\dots 1}_{k \text{ ta bir}}$$
Ularni bitma-bit AND ($\&$) qilamiz:
$$x \ \& \ (x - 1) = (A \, 1 \, 00\dots 0) \ \& \ (A \, 0 \, 11\dots 1) = A \, 0 \, \underbrace{00\dots 0}_{k \text{ ta nol}}$$
Ko'rib turganimizdek, eng o'ngdagi $1$ nolga aylandi, qolgan $A$ bitlari esa zarracha o'zgarmadi. Ushbu algoritm faqat $1$-bitlar soniga teng miqdorda qadam bajaradi ($O(k)$, bu yerda $k \le \text{bitlar soni}$).

#### Eng Quyi O'rnatilgan Bitni Ajratish (Isolate Lowest Set Bit):
$$L(x) = x \ \& \ (-x)$$
Two's complement qoidasi bo'yicha $-x = \sim x + 1$:
$$x = A \, 1 \, 00\dots 0$$
$$\sim x = \sim A \, 0 \, 11\dots 1$$
$$-x = \sim x + 1 = \sim A \, 1 \, 00\dots 0$$
Endi ularni AND qilamiz:
$$x \ \& \ (-x) = (A \, 1 \, 00\dots 0) \ \& \ (\sim A \, 1 \, 00\dots 0) = 00\dots 0 \, 1 \, 00\dots 0$$
Faqatgina eng quyi $1$ biti saqlanib qoldi!

#### Sonning 2 ning Darajasi Ekanligini Tekshirish:
Agar $x > 0$ soni 2 ning darajasi bo'lsa ($x = 2^k$), uning ikkilik ko'rinishida faqat bitta $1$ biti bo'ladi. Demak, $x \ \& \ (x - 1) == 0$.

### 3.3. Parallel SWAR (SIMD Within A Register) Popcount
Popcountni $O(1)$ vaqtda, hech qanday sikllarsiz hisoblash algoritmi divide-and-conquer usuliga asoslanadi:

```text
32-bit:  [b31 b30] [b29 b28] ... [b3 b2] [b1 b0]
1-qadam: Har 2-bitli juftlikdagi birlarni o'z ichida qo'shish (maska: 0x55555555)
2-qadam: Har 4-bitli guruhdagi birlarni qo'shish (maska: 0x33333333)
3-qadam: Har 8-bitli baytdagi birlarni qo'shish (maska: 0x0F0F0F0F)
4-qadam: Barcha baytlarni 0x01010101 ga ko'paytirib, yuqori baytga yig'ish va siljitish
```

### 3.4. Rust Xotira Modeli va CPU Instruksiyalari
Zamonaviy protsessorlarda bu operatsiyalar maxsus apparat instruksiyalariga ega:
- x86: `POPCNT`, `TZCNT` (Trailing Zero Count), `LZCNT` (Leading Zero Count).
- ARM64: `VCNT`, `CLZ`, `RBIT`.

Rust standart kutubxonasidagi `count_ones()`, `trailing_zeros()`, `leading_zeros()` funksiyalari to'g'ridan-to'g'ri LLVM intrinsics orqali ushbu CPU apparat buyruqlariga aylanadi va $1$ takt ichida bajariladi.

### 3.5. Amaliy Kod

```rust
/// Brian Kernighan algoritmi orqali 1-bitlar sonini sanash
fn brian_kernighan_popcount(mut x: u32) -> u32 {
    let mut count = 0;
    while x != 0 {
        x &= x - 1; // Eng quyi 1-bitni o'chirish
        count += 1;
    }
    count
}

/// 32-bitli SWAR O(1) Popcount algoritmi
fn swar_popcount(mut x: u32) -> u32 {
    x = x - ((x >> 1) & 0x5555_5555);
    x = (x & 0x3333_3333) + ((x >> 2) & 0x3333_3333);
    x = (x + (x >> 4)) & 0x0F0F_0F0F;
    x = x.wrapping_mul(0x0101_0101);
    x >> 24
}

/// Son 2 ning darajasi ekanligini O(1) tekshirish
fn is_power_of_two(x: u32) -> bool {
    x != 0 && (x & (x - 1)) == 0
}

/// Eng quyi o'rnatilgan bitni ajratib olish
fn isolate_lowest_bit(x: i32) -> i32 {
    x & -x
}

/// Massivdagi bitta yolg'iz (juftsiz) elementni topish
fn find_single_number(nums: &[u32]) -> u32 {
    nums.iter().fold(0, |acc, &val| acc ^ val)
}

fn main() {
    let n: u32 = 0b1011_0010_1111_0000; // 8 ta bir
    assert_eq!(brian_kernighan_popcount(n), 8);
    assert_eq!(swar_popcount(n), 8);
    assert_eq!(n.count_ones(), 8); // Rust intrinsics

    // 2 ning darajasini tekshirish
    assert!(is_power_of_two(1024));
    assert!(is_power_of_two(1));
    assert!(!is_power_of_two(1023));

    // Eng quyi bitni ajratish
    // 12 = 0b1100 => eng quyi bit 4 = 0b0100
    assert_eq!(isolate_lowest_bit(12), 4);
    assert_eq!(isolate_lowest_bit(-12), 4);

    // XOR orqali yolg'iz sonni topish
    let data = [4, 7, 2, 9, 2, 4, 7];
    assert_eq!(find_single_number(&data), 9);
}
```
