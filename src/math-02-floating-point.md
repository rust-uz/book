# 2-Modul: Haqiqiy Sonlar va Hisoblash Xatoliklari (Floating-Point Math)

Matematikada haqiqiy sonlar to'plami $\mathbb{R}$ uzluksiz (kontinuum) va cheksizdir. Har qanday ikki haqiqiy son orasida sanoqsiz ko'p boshqa sonlar mavjud. Kompyuter xotirasi esa cheklangan — bizda sonlarni ifodalash uchun bor-yo'g'i 32 yoki 64 ta binar bit berilgan. 

Ushbu cheklovni yechish uchun **IEEE-754** suzuvchi nuqtali sonlar standarti ishlab chiqilgan. Ushbu modulda suzuvchi sonlarning ichki arxitekturasi, yaxlitlash xatoliklarining matematik sababi va o'yin grafikasi tarixidagi eng mashhur Fast Inverse Square Root algoritmini tahlil qilamiz.

---

## 1. IEEE-754 Standarti: Normalizatsiya, Mantissa, Eksponenta va Bias ($2^{e - 127}$)

Haqiqiy sonni cheklangan bitlarda ilmiy notatsiya (scientific notation) ko'rinishida ifodalash mumkin:
$$\pm M \times 2^E$$

### 1.1. Matematik Isbot

IEEE-754 binar suzuvchi nuqta (binary32 / `f32`) formati 32 bitdan iborat bo'lib, uchta asosiy maydonga bo'linadi:
1. **Sign ($S$ - Ishora):** 1 bit ($b_{31}$). $S = 0$ bo'lsa musbat, $S = 1$ bo'lsa manfiy.
2. **Biased Exponent ($E$ - Siljitilgan eksponenta):** 8 bit ($b_{30} \dots b_{23}$). Qiymatlar oralig'i: $[0, 255]$.
3. **Mantissa / Fraction ($M$ - Kasr qism):** 23 bit ($b_{22} \dots b_0$).

```text
 31 30         23 22                                            0
+--+-------------+-----------------------------------------------+
|S |   E (8 bit) |                  M (23 bit)                   |
+--+-------------+-----------------------------------------------+
```

#### Normal sonlar formulasining matematik isboti:
Agar $0 < E < 255$ bo'lsa, son **normal** hisoblanadi. Normal sonning qiymati:
$$V = (-1)^S \times (1 + M) \times 2^{E - \text{Bias}}$$

bu yerda:
- **Mantissa qiymati:**
  $$M = \sum_{i=1}^{23} b_{23-i} \cdot 2^{-i} = \frac{b_{22}}{2^1} + \frac{b_{21}}{2^2} + \dots + \frac{b_0}{2^{23}}$$
  Demak, $0 \le M < 1$.
- **Yashirin bir (Implicit Leading Bit):** Normal son har doim $1.xxxx_2$ ko'rinishida yozilishi mumkinligi sababli, boshidagi $1$ raqamini xotirada saqlash shart emas! Bu bizga 23 bit maydon bilan 24 bitlik aniqlik beradi.
- **Bias (Siljish) matematik zarurati:**
  Eksponenta manfiy ham bo'lishi kerak (masalan, $2^{-5}$ kichik sonlar uchun). Agar eksponenta uchun Two's complement ishlatilsa, suzuvchi sonlarni tezkor taqqoslash qiyinlashadi. 
  IEEE-754 da eksponentaga doimiy $\text{Bias} = 2^{k-1} - 1$ qo'shiladi ($k=8$ bo'lganda $\text{Bias} = 127$).
  Haqiqiy daraja:
  $$e = E - 127$$
  Bu yechim $E$ ning doimo manfiy bo'lmagan butun son ($0 \le E \le 255$) bo'lishini ta'minlaydi. Natijada protsessor ikki suzuvchi sonni oddiy butun sonlar kabi registr darajasida to'g'ridan-to'g'ri taqqoslay oladi!

#### Subnormal (Denormalized) sonlar va Underflow:
Agar $E = 0$ va $M \neq 0$ bo'lsa:
$$V = (-1)^S \times (0 + M) \times 2^{-126}$$
Bu yerda yashirin bir $0$ ga aylanadi va eksponenta qat'iy $e = -126$ bo'ladi. Bu nolga juda yaqin sonlar uchun to'satdan nolga tushib ketmasdan, asta-sekin aniqlikni kamaytirish (gradual underflow) imkonini beradi.

#### Maxsus qiymatlar ($E = 255$):
- $M = 0$: $\pm \infty$ (cheksizlik, masalan, $1.0 / 0.0$).
- $M \neq 0$: $\text{NaN}$ (Not a Number, masalan, $0.0 / 0.0$ yoki $\sqrt{-1}$).

### 1.2. Algoritm: Haqiqiy Sonni Ikkilik IEEE-754 ga O'tkazish
Misol: $N = -13.625$ sonini `f32` ga o'tkazish:
1. **Ishora:** $N < 0 \implies S = 1$.
2. **Butun qism:** $13_{10} = 1101_2$.
3. **Kasr qism:** $0.625 \times 2 = 1.25 \to 1$; $0.25 \times 2 = 0.5 \to 0$; $0.5 \times 2 = 1.0 \to 1$. Demak: $0.625_{10} = 0.101_2$.
4. **Birlashtirish:** $13.625_{10} = 1101.101_2$.
5. **Normalizatsiya:** $1.101101_2 \times 2^3$.
6. **Eksponenta va Bias:** $e = 3 \implies E = 3 + 127 = 130 = 10000010_2$.
7. **Mantissa:** Yashirin $1$ olinadi, qolgani 23 bitgacha nollar bilan to'ldiriladi: $10110100000000000000000_2$.

### 1.3. Rust Xotira Modeli
Rust tilida `f32` 4 bayt, `f64` 8 bayt (11 bit eksponenta, $\text{bias} = 1023$, 52 bit mantissa) xotirani egallaydi.
Rust xotiradagi xom bitlarni ko'rish va o'zgartirish uchun qulay, nol-overheadli metodlarni taqdim etadi:
- `f32::to_bits()` $\to$ `u32` ga xavfsiz bitcast.
- `f32::from_bits(u32)` $\to$ `f32` ga xavfsiz bitcast.

### 1.4. Amaliy Kod

Quyidagi kod berilgan `f32` sonini uning tarkibiy matematik qismlariga ajratadi va formulani qayta hisoblab tekshiradi:

```rust
struct DecodedF32 {
    sign: u32,
    exponent_raw: u32,
    exponent_actual: i32,
    mantissa_raw: u32,
    reconstructed: f32,
}

fn decode_f32(val: f32) -> DecodedF32 {
    let bits = val.to_bits();
    let sign = (bits >> 31) & 1;
    let exponent_raw = (bits >> 23) & 0xFF;
    let mantissa_raw = bits & 0x7F_FFFF;

    let (exponent_actual, mantissa_val) = if exponent_raw == 0 {
        // Subnormal
        (-126, mantissa_raw as f64 / (1 << 23) as f64)
    } else {
        // Normal
        (
            exponent_raw as i32 - 127,
            1.0 + (mantissa_raw as f64 / (1 << 23) as f64),
        )
    };

    let sign_multiplier = if sign == 1 { -1.0 } else { 1.0 };
    let reconstructed = (sign_multiplier * mantissa_val * (2.0f64).powi(exponent_actual)) as f32;

    DecodedF32 {
        sign,
        exponent_raw,
        exponent_actual,
        mantissa_raw,
        reconstructed,
    }
}

fn main() {
    let target = -13.625f32;
    let decoded = decode_f32(target);

    assert_eq!(decoded.sign, 1);
    assert_eq!(decoded.exponent_raw, 130);
    assert_eq!(decoded.exponent_actual, 3);
    assert_eq!(decoded.reconstructed, target);

    // Maxsus qiymatlarni tekshiramiz
    let pos_inf = f32::INFINITY;
    assert_eq!((pos_inf.to_bits() >> 23) & 0xFF, 255);
    assert_eq!(pos_inf.to_bits() & 0x7F_FFFF, 0);

    let nan_val = f32::NAN;
    assert_eq!((nan_val.to_bits() >> 23) & 0xFF, 255);
    assert_ne!(nan_val.to_bits() & 0x7F_FFFF, 0);
}
```

---

## 2. Nega $0.1 + 0.2 \neq 0.3$? Mashina Epsiloni ($\varepsilon$) va Yaxlitlash Xatoliklari

Dasturchilar ko'pincha `0.1 + 0.2 == 0.3` ifodasi nega `false` qaytarishidan hayratga tushishadi. Bu kompyuterning nosozligi emas, balki sanoq tizimlari orasidagi fundamental matematik xossadir.

### 2.1. Matematik Isbot

O'nlik sanoq tizimidagi $0.1 = \frac{1}{10}$ kasrining maxrajini tub ko'paytuvchilarga ajratamiz:
$$10 = 2 \times 5$$

Ikkilik sanoq tizimida chekli kasr hosil bo'lishi uchun maxraj faqat $2$ ning darajalaridan iborat bo'lishi shart ($2^k$). Maxrajda $5$ qatnashgani sababli, $0.1$ ikkilik tizimda **cheksiz davriy kasr**ga aylanadi:
$$\frac{1}{10} = 0.00011001100110011\dots_2 = 0.00011\overline{0011}_2$$

Xuddi $1/3$ soni o'nlik tizimda $0.3333\dots$ bo'lib tugallanmagani kabi, $0.1$ ham ikkilik tizimda cheksiz davom etadi.

Ammo kompyuter xotirasidagi `f64` mantissasi atigi 53 bit (1 yashirin bit + 52 bit). Cheksiz qator ushbu 53-bitda kesiladi va eng yaqin juft songa yaxlitlanadi (round to nearest, ties to even).

Haqiqiy xotirada saqlangan qiymatlar:
$$\text{val}(0.1) = 0.1000000000000000055511151231257827021181583404541015625$$
$$\text{val}(0.2) = 0.200000000000000011102230246251565404236316680908203125$$
Ularning apparat darajasidagi yig'indisi:
$$\text{val}(0.1) + \text{val}(0.2) = 0.3000000000000000444089209850062616169452667236328125$$
Biroq, to'g'ridan-to'g'ri $0.3$ kiritilganda uning yaxlitlangan qiymati:
$$\text{val}(0.3) = 0.299999999999999988897769753748434595763683319091796875$$

Ko'rib turganimizdek:
$$\text{val}(0.1 + 0.2) - \text{val}(0.3) \approx 5.55 \times 10^{-17} \neq 0$$

### 2.2. Mashina Epsiloni ($\varepsilon$)
Mashina epsiloni — $1.0$ soniga qo'shilganda $1.0$ dan farqli natija beruvchi eng kichik musbat son:
$$1.0 + \varepsilon > 1.0$$
- `f32::EPSILON` $= 2^{-23} \approx 1.1920929 \times 10^{-7}$
- `f64::EPSILON` $= 2^{-52} \approx 2.220446049250313 \times 10^{-16}$

### 2.3. Algoritm: Suzuvchi Sonlarni Xavfsiz Taqqoslash
Suzuvchi nuqtali sonlarni hech qachon `==` bilan taqqoslab bo'lmaydi. Ular nisbiy va mutloq xatolik chegarasi (tolerance) bilan tekshiriladi:
$$|a - b| \le \varepsilon \times \max(1.0, |a|, |b|)$$

### 2.4. Rust Xotira Modeli va Tip Tizimi Falsafasi
Rust tip tizimi matematik jihatdan eng to'g'ri qarorlardan birini qabul qilgan: **`f32` va `f64` tiplari `Eq` va `Ord` traitlariga ega emas!**
Nega?
Matematikada ekvivalentlik munosabati refleksivlik aksiomasini qanoatlantirishi shart: $\forall x, x = x$.
Biroq IEEE-754 bo'yicha:
$$\text{NaN} == \text{NaN} \implies \text{false}$$
$\text{NaN}$ o'z-o'ziga teng emasligi sababli, suzuvchi sonlar qat'iy ekvivalentlikni buza oladi. Shu sababli Rust ularga faqat `PartialEq` va `PartialOrd` traitlarini bergan. Natijada `f64` ni to'g'ridan-to'g'ri `std::collections::BTreeSet` yoki `HashMap` kaliti sifatida ishlatib bo'lmaydi — bu xotira xavfsizligini ta'minlaydi.

### 2.5. Amaliy Kod

```rust
fn approx_eq(a: f64, b: f64, epsilon: f64) -> bool {
    let diff = (a - b).abs();
    let norm = a.abs().max(b.abs()).max(1.0);
    diff <= epsilon * norm
}

fn main() {
    let sum = 0.1 + 0.2;
    let expected = 0.3;

    // To'g'ridan-to'g'ri taqqoslash kutilganidek false beradi:
    assert!(sum != expected);

    // Xavfsiz taqqoslash:
    assert!(approx_eq(sum, expected, f64::EPSILON));

    // Rust tip tizimidagi NaN xatti-harakati
    let nan = f64::NAN;
    assert!(!(nan == nan)); // NaN o'ziga teng emas!
    assert!(nan.is_nan());
}
```

---

## 3. Afsonaviy Fast Inverse Square Root Algoritmi Matematikasi ($1/\sqrt{x}$ va `0x5f3759df` Sehrli Soni)

1999-yilda chiqarilgan *Quake III Arena* o'yinining dvigatelida ($Id$ Software, Jon Karmak) yorug'lik va 3D koordinatalarni hisoblash uchun millionlab marta birlik vektorlarni normalizatsiya qilish talab qilingan:
$$\mathbf{\hat{v}} = \frac{\mathbf{v}}{\|\mathbf{v}\|} = \mathbf{v} \times \frac{1}{\sqrt{x}}$$
O'sha davr protsessorlarida bo'lish (`div`) va kvadrat ildiz (`fsqrt`) o'nlab takt talab qiluvchi juda sekin operatsiyalar edi. Muhandislar analitik matematikaning aqlbovar qilmas fokusini qo'llab, bu amalni $4\times$ tezlashtirishgan.

### 3.1. Matematik Isbot

Bizga berilgan musbat $x$ soni uchun $y = \frac{1}{\sqrt{x}} = x^{-1/2}$ ni hisoblash talab etiladi.

#### 1-Qadam: Logarifm orqali soddalashtirish
Ikkala tomonni $2$ asos bo'yicha logarifmlaymiz:
$$\log_2(y) = \log_2(x^{-1/2}) = -\frac{1}{2} \log_2(x)$$

#### 2-Qadam: IEEE-754 bitlari va Logarifm bog'liqligi
IEEE-754 formatidagi normal son:
$$x = 2^{E_x - 127} (1 + M_x)$$
Logarifm olamiz:
$$\log_2(x) = E_x - 127 + \log_2(1 + M_x)$$
$M_x \in [0, 1)$ oraliqda yotgani sababli, $\log_2(1 + M_x)$ funksiyasini to'g'ri chiziq orqali chiziqli approksimatsiya (linear approximation) qilish mumkin:
$$\log_2(1 + M_x) \approx M_x + \mu$$
bu yerda $\mu \approx 0.0450466$ — oraliqdagi o'rtacha xatolikni minimallashtiruvchi koeffitsiyent.

Endi sonning 32-bitli butun son sifatida talqin qilingan bit qiymatini ($I_x$) ko'ramiz:
$$I_x = E_x \cdot 2^{23} + M_x \cdot 2^{23} = 2^{23} (E_x + M_x)$$
Ushbu tenglikdan:
$$E_x + M_x = \frac{I_x}{2^{23}}$$
Demak:
$$\log_2(x) \approx E_x + M_x - 127 + \mu = \frac{I_x}{2^{23}} - 127 + \mu$$

#### 3-Qadam: Bog'liqlik tenglamasini yechish va Sehrli Sonning kelib chiqishi
Ushbu natijani $\log_2(y) = -\frac{1}{2} \log_2(x)$ ga qo'yamiz:
$$\frac{I_y}{2^{23}} - 127 + \mu \approx -\frac{1}{2} \left( \frac{I_x}{2^{23}} - 127 + \mu \right)$$
Ikkala tomonni $2^{23}$ ga ko'paytiramiz:
$$I_y - 2^{23}(127 - \mu) \approx -\frac{1}{2} I_x + \frac{1}{2} 2^{23}(127 - \mu)$$
$$I_y \approx \frac{3}{2} 2^{23} (127 - \mu) - \frac{1}{2} I_x$$

Sehrli konstanta $K$ ni hisoblaymiz:
$$K = \frac{3}{2} \times 2^{23} \times (127 - \mu)$$
Agar $\mu \approx 0.0450466$ olinsa:
$$K = \frac{3}{2} \times 8388608 \times 126.9549534 \approx 1597463007_{10} = \mathbf{0x5F3759DF}_{16}$$

Natijada butun sonli bitlar darajasida mo''jizaviy formula hosil bo'ladi:
$$I_y = \text{0x5f3759df} - (I_x \gg 1)$$
Bitta bitwise o'ngga siljitish va bitta ayirish orqali biz $1/\sqrt{x}$ ning taxminiy qiymatini topdik!

#### 4-Qadam: Nyuton-Rafson (Newton-Raphson) Iteratsiyasi
Dastlabki taxminni yuqori aniqlikka yetkazish uchun Nyuton usuli qo'llaniladi.
Bizning funksiyamiz:
$$f(y) = \frac{1}{y^2} - x = 0$$
Uning hosilasi:
$$f'(y) = -\frac{2}{y^3}$$
Nyuton-Rafson qadami:
$$y_{n+1} = y_n - \frac{f(y_n)}{f'(y_n)} = y_n - \frac{y_n^{-2} - x}{-2 y_n^{-3}} = y_n + \frac{y_n - x y_n^3}{2} = y_n \left( \frac{3}{2} - \frac{x}{2} y_n^2 \right)$$

Birgina iteratsiyaning o'zi xatolikni $1\%$ dan kam darajaga tushiradi!

### 3.2. Rust Xotira Modeli: C dagi Undefined Behavior ga qarshi Xavfsiz Bitcast
C tilidagi mashhur kod:
```c
i = * ( long * ) &y; // Xavfli Undefined Behavior!
```
Bu C/C++ tillarida **Strict Aliasing Rule** qoidasini buzadi va zamonaviy optimizatsiyalarda xotira korrupsiyasiga olib keladi.

Rust tilida bu amal xotira xavfsiz va mutlaqo nol xarajatli (zero-cost) tarzda `f32::to_bits()` va `f32::from_bits()` yordamida bajariladi. LLVM kompilyatori buni to'g'ridan-to'g'ri protsessorning registrlararo ko'chirish instruksiyasiga aylantiradi.

### 3.3. Amaliy Kod

```rust
pub fn fast_inv_sqrt(x: f32) -> f32 {
    let xhalf = 0.5f32 * x;
    let mut i = x.to_bits(); // C dagi: *(long*)&x
    i = 0x5f3759df - (i >> 1); // Sehrli konstanta bilan hisoblash
    let mut y = f32::from_bits(i); // Bitlarni yana suzuvchi songa qaytarish

    // 1-Nyuton-Rafson iteratsiyasi: y = y * (1.5 - xhalf * y * y)
    y = y * (1.5f32 - xhalf * y * y);

    // Agar zarur bo'lsa, 2-iteratsiya orqali aniqlikni yana oshirish mumkin:
    // y = y * (1.5f32 - xhalf * y * y);

    y
}

fn main() {
    let test_values = [1.0f32, 4.0, 16.0, 0.25, 100.0, 12345.0];

    for &val in &test_values {
        let approx = fast_inv_sqrt(val);
        let exact = 1.0 / val.sqrt();

        let relative_error = ((approx - exact) / exact).abs();

        // Xatolik 1% (0.01) dan ancha kichikligini tasdiqlaymiz:
        assert!(
            relative_error < 0.002,
            "val: {}, approx: {}, exact: {}, error: {}",
            val,
            approx,
            exact,
            relative_error
        );
    }

    // 4.0 ning teskari ildizi 0.5 bo'lishi kerak:
    let inv_4 = fast_inv_sqrt(4.0);
    assert!((inv_4 - 0.5).abs() < 0.001);
}
```
