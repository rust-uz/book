# 4-Modul: Kriptografik Matematika (Ferris Vault asosi)

Zamonaviy axborot xavfsizligi va kriptografiya poydevori sonlar nazariyasi, modulli arifmetika va abstrakt algebraning eng nozik natijalariga tayanadi. Kriptografik kod yozish oddiy amaliy dasturlashdan tubdan farq qiladi: bu yerda nafaqat matematik to'g'rilik, balki apparat darajasidagi **Side-Channel (Vaqt o'lchash)** hujumlariga chidamlilik va xotiradagi maxfiy ma'lumotlarni xavfsiz tozalash talab etiladi.

Ushbu modulda ochiq kalitli kriptografiya (RSA/Diffie-Hellman) asosi bo'lgan modulli arifmetika teoremalari, simmetrik shifrlashning eng xavfsiz standarti bo'lgan **ChaCha20 da ARX primitivlari** hamda zamonaviy **BLAKE3 daraxt xeshlashi** matematikasi va Rust xotira modelini o'rganamiz.

---

## 1. Modulli Arifmetika: Eyler Teoremasi, Kichik Fermat Teoremasi va Katta Tub Sonlar

Ochiq kalitli kriptografiya (asymmetric cryptography) "bir tomonlama funksiyalar" (one-way trapdoor functions) ustiga qurilgan: to'g'ri yo'nalishda hisoblash oson, ammo maxfiy kalitsiz teskari hisoblash amalda imkonsizdir.

### 1.1. Matematik Isbot

$m \in \mathbb{Z}^+$ bo'lsin. Ikki son $a$ va $b$ ning modulli taqqoslamasi:
$$a \equiv b \pmod m \iff m \mid (a - b)$$

#### Eyler Totient Funksiyasi $\varphi(n)$:
$n$ dan oshmaydigan va $n$ bilan o'zaro tub bo'lgan sonlar miqdori $\varphi(n)$ deb ataladi.
- Agar $p$ tub son bo'lsa:
  $$\varphi(p) = p - 1$$
- Agar $n = p \cdot q$ ikkita tub son ko'paytmasi bo'lsa:
  $$\varphi(pq) = (p - 1)(q - 1)$$

#### Kichik Fermat Teoremasi (Fermat's Little Theorem):
Agar $p$ tub son va $\gcd(a, p) = 1$ bo'lsa:
$$a^{p-1} \equiv 1 \pmod p \implies a^p \equiv a \pmod p$$

#### Isbot:
$\{1, 2, 3, \dots, p-1\}$ to'plamini olamiz. Har bir elementni $a$ ga ko'paytiramiz:
$$S = \{1a, 2a, 3a, \dots, (p-1)a\}$$
Ushbu to'plamdagi hech qaysi ikki element $\pmod p$ bo'yicha teng bo'la olmaydi. Chunki agar $ia \equiv ja \pmod p$ bo'lsa, $(i - j)a \equiv 0 \pmod p$. $\gcd(a, p) = 1$ bo'lgani uchun $p \mid (i - j)$, bu esa $1 \le i, j < p$ oraliqda faqat $i = j$ bo'lgandagina mumkin.
Demak, $S$ to'plamining $\pmod p$ qoldiqlari aynan $\{1, 2, \dots, p-1\}$ to'plamining o'rin almashtirishidan (permutatsiyasidan) iborat!

Barcha elementlarni o'zaro ko'paytiramiz:
$$(1a) \cdot (2a) \cdot (3a) \dots ((p-1)a) \equiv 1 \cdot 2 \cdot 3 \dots (p-1) \pmod p$$
$$a^{p-1} \cdot (p-1)! \equiv (p-1)! \pmod p$$
$\gcd((p-1)!, p) = 1$ bo'lgani sababli, ikkala tomonni $(p-1)!$ ga bo'lish mumkin:
$$a^{p-1} \equiv 1 \pmod p$$
Q.E.D.

#### Eyler Teoremasi (Euler's Totient Theorem):
Ixtiyoriy $n \in \mathbb{N}$ va $\gcd(a, n) = 1$ uchun:
$$a^{\varphi(n)} \equiv 1 \pmod n$$
Ushbu teorema Kichik Fermat teoremasining ixtiyoriy $n$ soni uchun umumlashtirilgan ko'rinishidir.

#### Katta Tub Sonlar va Ularning Taqsimoti:
Ochiq kalitli shifrlashda (RSA, ECC, Diffie-Hellman) xavfsizlik 1024, 2048 yoki 4096 bitli ulkan tub sonlarni generatsiya qilishga asoslanadi.

1. **Tub Sonlar Taqsimoti Teoremasi (Prime Number Theorem):**
   $x$ gacha bo'lgan tub sonlar soni $\pi(x)$ asimptotik ravishda:
   $$\pi(x) \sim \frac{x}{\ln x}$$
   Demak, $x$ atrofidagi tasodifiy toq sonning tub bo'lish ehtimoli taxminan $\frac{2}{\ln x}$ ga teng. Masalan, 2048-bitli sonlar oralig'ida ($\ln(2^{2048}) \approx 1419.5$) har 710 ta toq sondan bittasi tub sondir! Bu shuni anglatadiki, tasodifiy toq son olib, uni tublikka tekshirish orqali bir necha yuzta urinishda katta tub son topish kafolatlanadi.

2. **Tublikni Tekshirish (Primality Testing):**
   Ulkan sonlarni bo'lib ko'rish (Trial Division, $O(\sqrt{n})$) amalda imkonsiz. Shu sababli ehtimollik algoritmlari qo'llaniladi:

   - **Fermat Tublik Testi va Karmaykl Sonlari (Carmichael Numbers):**
     Kichik Fermat teoremasiga ko'ra, agar $a^{n-1} \not\equiv 1 \pmod n$ bo'lsa, $n$ aniq murakkab son. Biroq aksincha xulosa har doim to'g'ri emas! Ba'zi murakkab sonlar mavjudki (masalan, $561 = 3 \times 11 \times 17$, $1105$, $1729$), ular barcha $\gcd(a, n) = 1$ asoslar uchun Fermat tengligini qanoatlantiradi. Bular **Karmaykl sonlari** deb ataladi va Fermat testini alday oladi.

   - **Miller-Rabin Algoritmi (Matematik Isboti):**
     Ushbu zaiflikni yengish uchun algebraik lemmaga tayanamiz: agar $p$ tub son bo'lsa, $x^2 \equiv 1 \pmod p$ tenglama faqat ikkita trivial yechimga ega: $x \equiv 1$ yoki $x \equiv -1 \pmod p$. Agar biror sonning $1$ ga teng kvadratida oraliq ildiz $\pm 1$ bo'lmasa, son murakkabdir!

     $n - 1$ juft sonini $n - 1 = 2^s \cdot d$ ko'rinishida yozamiz ($d$ toq).
     Ixtiyoriy $a$ asosi uchun quyidagi shartlardan kamida biri bajarilishi shart:
     1. $a^d \equiv 1 \pmod n$
     2. Biror $0 \le r < s$ uchun $a^{2^r d} \equiv n - 1 \equiv -1 \pmod n$

     Agar ikkala shart ham bajarilmasa, $n$ qat'iy **murakkab son** (composite) hisoblanadi. $k$ ta mustaqil tasodifiy asoslar bilan tekshirilganda, murakkab sonning testdan o'tib ketish ehtimoli $\le 4^{-k}$ bo'ladi. Masalan, $k = 40$ bo'lganda xato ehtimoli $2^{-80} < 10^{-24}$ bo'lib, amalda mutlaq aniqlikni kafolatlaydi.

#### Kriptografik Qo'llanilishi (RSA Shifrlash):
Kalitlar generatsiyasi:
1. Ikkita katta tub son $p$ va $q$ tanlanadi. $n = p \cdot q$, $\varphi(n) = (p-1)(q-1)$.
2. Ochiq ko'rsatkich $e$ tanlanadi ($\gcd(e, \varphi(n)) = 1$).
3. Maxfiy ko'rsatkich $d$ modulli teskari element sifatida hisoblanadi:
   $$e \cdot d \equiv 1 \pmod{\varphi(n)}$$
4. Shifrlash: $c \equiv m^e \pmod n$.
5. Deshifrlash: $c^d \equiv (m^e)^d \equiv m^{e \cdot d} \equiv m^{k \cdot \varphi(n) + 1} \equiv (m^{\varphi(n)})^k \cdot m \equiv 1^k \cdot m \equiv m \pmod n$.

### 1.2. Algoritm: Ikkilik Darajaga Ko'tarish (Binary Exponentiation)
$a^b \bmod m$ ni hisoblashda oddiy $b$ marta ko'paytirish $O(b)$ vaqt oladi ($b \approx 2^{2048}$ bo'lganda olam oxirigacha hisoblab bo'lmaydi!).
**Square-and-Multiply** algoritmi darajaning ikkilik bitlariga qarab amalni $O(\log b)$ qadamga tushiradi:
- Har bir qadamda son o'z-o'ziga ko'paytiriladi (Square: $x \leftarrow x^2 \bmod m$).
- Agar joriy bit $1$ bo'lsa, asosga ko'paytiriladi (Multiply: $x \leftarrow x \cdot a \bmod m$).

### 1.3. Rust Xotira Modeli: Constant-Time va Zeroize

Kriptografiyada dasturning nafaqat natijasi, balki u qancha vaqt ishlashi ham maxfiy kalitni fosh qilishi mumkin (**Timing Attack**).
Agar `if (bit == 1) { multiply }` kabi tarmoqlanish yozilsa, CPU ning Branch Predictor apparati va instruction cache o'rtasidagi farq xakerga maxfiy kalit bitlarini mikrosoniyalarda aniqlashga yo'l ochadi!

#### Xotirani Tozalash (Zeroize):
Operativ xotira (Stack va Heap) xavfsizligi: o'zgaruvchi o'z ko'lamidan (scope) chiqib ketganda (`drop`), Rust xotirani qaytaradi, lekin undagi baytlarni avtomatik tozalamaydi. Maxfiy kalitlar xotirada qolib ketsa, "Heartbleed" kabi xurujlar ularni o'qib olishi mumkin.
Kompilyator optimizatsiyasi ham ayyor: agar siz shunchaki massivga nol yozsangiz, kompilyator *"Bu o'zgaruvchi boshqa ishlatilmaydi"* deb, yozish amalini Dead Store Elimination orqali o'chirib yuboradi!
Shuning uchun kriptografik dasturlarda `volatile` yoki `compiler_fence` orqali tozalash kafolatlanishi shart.

### 1.4. Amaliy Kod

Quyidagi dastur modulli darajaga ko'tarish, Kengaytirilgan Evklid algoritmi (Extended GCD), modulli teskari sonni hisoblash hamda Miller-Rabin tublik testini to'liq amalga oshiradi:

```rust
/// Kengaytirilgan Evklid algoritmi: ax + by = gcd(a, b)
/// Qaytaradi: (gcd, x, y)
pub fn extended_gcd(a: i64, b: i64) -> (i64, i64, i64) {
    if b == 0 {
        (a, 1, 0)
    } else {
        let (gcd, x1, y1) = extended_gcd(b, a % b);
        let x = y1;
        let y = x1 - (a / b) * y1;
        (gcd, x, y)
    }
}

/// Modulli teskari elementni hisoblash: (a * x) = 1 (mod m)
pub fn mod_inverse(a: i64, m: i64) -> Option<i64> {
    let (gcd, x, _) = extended_gcd(a, m);
    if gcd != 1 {
        None // gcd(a, m) != 1 bo'lsa teskari element mavjud emas
    } else {
        Some((x % m + m) % m)
    }
}

/// Square-and-Multiply (Binary Exponentiation): (base^exp) % m
pub fn mod_exp(mut base: u64, mut exp: u64, modulus: u64) -> u64 {
    if modulus == 1 {
        return 0;
    }
    let mut result = 1u64;
    base %= modulus;

    while exp > 0 {
        if exp & 1 == 1 {
            result = ((result as u128 * base as u128) % modulus as u128) as u64;
        }
        base = ((base as u128 * base as u128) % modulus as u128) as u64;
        exp >>= 1;
    }

    result
}

/// Miller-Rabin deterministik tublik testi (u64 chegarasida kichik asoslar bilan)
pub fn is_prime_miller_rabin(n: u64) -> bool {
    if n < 2 {
        return false;
    }
    if n == 2 || n == 3 {
        return true;
    }
    if n % 2 == 0 {
        return false;
    }

    // n - 1 = 2^s * d (d toq)
    let mut d = n - 1;
    let mut s = 0;
    while d % 2 == 0 {
        d /= 2;
        s += 1;
    }

    // 2^64 gacha bo'lgan barcha sonlar uchun yetarli deterministik asoslar:
    let test_bases: &[u64] = &[2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37];

    for &a in test_bases {
        if n <= a {
            break;
        }

        let mut x = mod_exp(a, d, n);
        if x == 1 || x == n - 1 {
            continue;
        }

        let mut composite = true;
        for _ in 1..s {
            x = ((x as u128 * x as u128) % n as u128) as u64;
            if x == n - 1 {
                composite = false;
                break;
            }
        }

        if composite {
            return false; // Aniq murakkab son
        }
    }

    true // Tub son
}

fn main() {
    // Kichik Fermat teoremasini tekshiramiz: a^(p-1) = 1 (mod p)
    // p = 17 (tub son), a = 5
    let p = 17u64;
    let a = 5u64;
    assert_eq!(mod_exp(a, p - 1, p), 1);

    // Eyler teoremasi: 7^(phi(10)) = 1 (mod 10). phi(10) = 4
    assert_eq!(mod_exp(7, 4, 10), 1);

    // Modulli teskari sonni topish: 3 * x = 1 (mod 11)
    let inv = mod_inverse(3, 11).expect("Teskari element mavjud");
    assert_eq!(inv, 4); // Chunki 3 * 4 = 12 = 1 (mod 11)
    assert_eq!((3 * inv) % 11, 1);

    // Tub sonlarni tekshirish (Miller-Rabin testi)
    assert!(is_prime_miller_rabin(17));
    assert!(is_prime_miller_rabin(104_729)); // 10,000-tub son
    assert!(!is_prime_miller_rabin(100));

    // Karmaykl soni (Carmichael number): 561 = 3 * 11 * 17
    // Fermat testini aldaydi, lekin Miller-Rabin uni to'g'ri murakkab deb topadi!
    assert!(!is_prime_miller_rabin(561));
}
```

---

## 2. Simmetrik Shifrlar Arifmetikasi: ChaCha20 da ARX (Add-Rotate-Xor) Primitivlari

Simmetrik kriptografiyada Klod Shennon tomonidan asos solingan ikki asosiy talab mavjud:
1. **Confusion (Chalkashtirish):** Shifrlangan matn va kalit o'rtasidagi bog'liqlikni imkon qadar chiziqsiz qilish.
2. **Diffusion (Tarqatish):** Ochiq matndagi bitta bit o'zgarsa, shifrlangan matnning o'rtacha 50% bitlari o'zgarishi kerak (qor ko'chishi effekti).

### 2.1. Matematik Isbot: Nega ARX?

Klassik AES shifri **S-Box** (Substitution Box) deb ataladigan jadvallardan foydalanadi. Ammo zamonaviy kiberxavfsizlikda ma'lum bo'lishicha, S-Box lardan foydalanish xavfli kesh xurujlariga (**Cache-Timing Attacks**) sabab bo'ladi: xakerlar CPU keshiga kirish vaqtini o'lchab, maxfiy kalitni tiklay oladilar.

Professor Daniel J. Bernstein (djb) tomonidan yaratilgan **ChaCha20** shifri ushbu muammoni tubdan hal qildi. U hech qanday jadvallarsiz, faqat uchta sodda **ARX** amallaridan foydalanadi:
1. **Modular Addition ($\boxplus$):** $a \boxplus b = (a + b) \bmod 2^{32}$. Chiziqsizlik (Confusion) kiritadi.
2. **Rotation ($\lll$):** Bitlarni aylanma siljitish ($a \lll k$). Tezkor tarqalishni (Diffusion) ta'minlaydi.
3. **XOR ($\oplus$):** Bitlarni chiziqli aralashtirish va simmetrik teskarilash.

#### ChaCha20 Quarter Round (QR) Formulalari:
ChaCha20 ning butun kuchi 4 ta 32-bitli so'z ustida bajariladigan bitta elementar matematik funksiyaga — **Quarter Round** ga asoslanadi:

$$\begin{aligned}
a &\leftarrow a \boxplus b; & d &\leftarrow (d \oplus a) \lll 16; \\
c &\leftarrow c \boxplus d; & b &\leftarrow (b \oplus c) \lll 12; \\
a &\leftarrow a \boxplus b; & d &\leftarrow (d \oplus a) \lll 8; \\
c &\leftarrow c \boxplus d; & b &\leftarrow (b \oplus c) \lll 7;
\end{aligned}$$

Nega aynan $\{16, 12, 8, 7\}$ siljitish konstanta qiymatlari tanlangan?
Matematik tahlillar shuni ko'rsatadiki, bu burchaklar 32-bitli so'z ichida har bir bitning boshqa barcha bitlar bilan o'zaro chatishishini eng tez vaqtda ta'minlaydi (optimal diffusion).

### 2.2. Algoritm: ChaCha20 Holat Matritsasi
ChaCha20 ning ichki holati $4 \times 4$ o'lchamli 16 ta 32-bitli so'zlardan iborat matritsadir (jami 512 bit = 64 bayt):

$$\begin{pmatrix}
c_0 & c_1 & c_2 & c_3 \\
k_0 & k_1 & k_2 & k_3 \\
k_4 & k_5 & k_6 & k_7 \\
b & n_0 & n_1 & n_2
\end{pmatrix}$$

- $c_0 \dots c_3$: O'zgarmas doimiylar (`"expand 32-byte k"` satrining ASCII kodlari).
- $k_0 \dots k_7$: 256-bitli maxfiy kalit (8 ta so'z).
- $b$: 32-bitli blok hisoblagich (counter).
- $n_0 \dots n_2$: 96-bitli bir martalik raqam (nonce).

Shifrlash 20 ta raunddan (10 ta ustun va diagonal juftliklari) iborat bo'lib, uning natijasida hosil bo'lgan 64-baytli oqim ochiq matn bilan oddiygina XOR qilinadi!

### 2.3. Rust Xotira Modeli
ChaCha20 holati stack xotirasida tekis `[u32; 16]` massivi sifatida joylashadi:
- Umumiy o'lchami: $16 \times 4 = 64$ bayt.
- Zamonaviy CPU larning kesh liniyasi (cache line) ham aynan **64 bayt**!
- Demak, butun ChaCha20 holati bitta L1 kesh liniyasiga to'liq sig'adi.
- Rustdagi `wrapping_add` va `rotate_left` bevosita protsessorning apparat instruksiyalariga (`add` va `rol`) aylanadi va hech qanday shartli sakrashlarsiz (branchless), konstanta vaqtda ($O(1)$ constant-time) bajariladi.

### 2.4. Amaliy Kod

Quyidagi kod rasmiy **RFC 8439** standartidagi test vektorlari bilan tekshirilgan to'liq ChaCha20 Quarter Round implementatsiyasidir:

```rust
/// ChaCha20 Quarter Round amali (RFC 8439 standarti bo'yicha)
pub fn quarter_round(a: &mut u32, b: &mut u32, c: &mut u32, d: &mut u32) {
    // 1-bosqich:
    *a = a.wrapping_add(*b);
    *d = (*d ^ *a).rotate_left(16);

    // 2-bosqich:
    *c = c.wrapping_add(*d);
    *b = (*b ^ *c).rotate_left(12);

    // 3-bosqich:
    *a = a.wrapping_add(*b);
    *d = (*d ^ *a).rotate_left(8);

    // 4-bosqich:
    *c = c.wrapping_add(*d);
    *b = (*b ^ *c).rotate_left(7);
}

fn main() {
    // RFC 8439 (Section 2.1.1) rasmiy test vektorlari:
    let mut a: u32 = 0x1111_1111;
    let mut b: u32 = 0x0102_0304;
    let mut c: u32 = 0x9b8d_6f43;
    let mut d: u32 = 0x0123_4567;

    quarter_round(&mut a, &mut b, &mut c, &mut d);

    // RFC dagi rasmiy kutilgan natijalar bilan solishtiramiz:
    assert_eq!(a, 0xea2a_92f4);
    assert_eq!(b, 0xcb1c_f8ce);
    assert_eq!(c, 0x4581_472e);
    assert_eq!(d, 0x5881_c4bb);
}
```

---

## 3. Xesh-Funksiyalar: Merkle-Damgård vs Sponge Konstruksiyalari (BLAKE3 Daraxt Xeshi)

Kriptografik xesh-funksiya har qanday ixtiyoriy hajmdagi ma'lumotni qat'iy belgilangan o'lchamdagi (masalan, 256 bit) xesh-qiymatga (digest) akslantiradi.

### 3.1. Matematik Isbot va Konstruksiyalar Solishtirmasi

Kriptografik xesh-funksiyalarga qo'yiladigan uchta asosiy aksioma:
1. **Pre-image resistance (Bir tomonlamalik):** Berilgan $h$ uchun $H(m) = h$ bo'lgan $m$ ni topish qiyinligi $O(2^n)$.
2. **Second pre-image resistance:** Berilgan $m_1$ uchun $H(m_1) = H(m_2)$ bo'lgan $m_2 \neq m_1$ ni topish qiyinligi $O(2^n)$.
3. **Collision resistance (Kolliziyaga chidamlilik):** $H(m_1) = H(m_2)$ bo'lgan ixtiyoriy ikki xabarni topish qiyinligi Tug'ilgan kun paradoksi (Birthday Paradox) bo'yicha $O(2^{n/2})$.

```text
1. Merkle-Damgård (MD5, SHA-1, SHA-256):
   IV -> [ f ] -> [ f ] -> [ f ] -> Hash
           ^        ^        ^
           |        |        |
          M_0      M_1      M_2
   (Zaiflik: Length Extension Attack!)

2. Sponge Konstruksiyasi (SHA-3 / Keccak):
   Kirish:     M_0      M_1
                |        |
   Holat: [r | c] -> [r | c] -> [r | c] -> Chiqish (Digest)
             Absorbing             Squeezing

3. Merkle Daraxt Xeshi (BLAKE3):
                 [ Root Hash ]
                 /           \
           [ Node 0 ]      [ Node 1 ]
            /      \        /      \
          Leaf0   Leaf1   Leaf2   Leaf3  <-- Parallel hisoblash!
```

#### 1. Merkle-Damgård Konstruksiyasi va Uning Kamchiligi:
Xabar qat'iy bloklarga bo'linadi va siqish funksiyasi (compression function) orqali ketma-ket hisoblanadi:
$$H_i = f(H_{i-1}, M_i)$$
**Fundamental kamchilik (Length Extension Attack):**
Agar tizim $H(\text{secret} \parallel \text{message})$ orqali autentifikatsiya qilsa, tajovuzkor maxfiy kalitni bilmagan holda xabarning oxiriga qo'shimcha ma'lumot ulab ($M'$), uning qonuniy xeshini hisoblay oladi!

#### 2. Sponge (Gubka) Konstruksiyasi (SHA-3 / Keccak):
Ichki holat kengligi $b = r + c$ (Rate $r$ va Capacity $c$).
- **Absorbing (Yutish fazasi):** Kiruvchi xabar bo'laklari $r$ qismi bilan XOR qilinadi va f-permutatsiya ishga tushadi.
- **Squeezing (Siqib chiqarish fazasi):** Natijaviy xesh $r$ qismidan o'qib olinadi. Maxfiy $c$ qismi hech qachon tashqariga chiqmagani sababli, Length Extension hujumlariga qarshi 100% tabiiy immunitetga ega.

#### 3. BLAKE3 Daraxt Xeshlashi (Tree Hashing):
An'anaviy xeshlar (SHA-256, SHA-3) **qat'iy ketma-ket (sequential)** ishlaydi. Ular 100 GB hajmdagi katta faylni xeshlashda faqat bitta CPU yadrosidan foydalana oladi.

BLAKE3 esa ma'lumotni 1024 baytli (1 KB) bo'laklarga bo'ladi va **Merkle Daraxti** shaklida quradi:
- Har bir 1 KB bo'lak mustaqil ravishda bir vaqtda parallel xeshlanadi.
- Hosil bo'lgan xeshlar juftma-juft birlashtirilib, daraxt bo'ylab yuqoriga ko'tariladi.
- Natijada ko'p yadroli protsessorlarda va SIMD vektor registrlarida sekundiga 5-10 Gigabayt tezlikka erishiladi!

### 3.2. Rust Xotira Modeli: Parallelizm va Rayon Integratsiyasi
Merkle daraxti xeshlash modeli Rust tilining "Qo'rqmas Paralellik" (Fearless Concurrency) paradigmasiga ideal mos tushadi:
- Har bir barg xotirada mustaqil slice (`&[u8]`) sifatida olinadi.
- Bir-biriga bog'liq bo'lmagan xotira bo'laklari `rayon::join` yoki SIMD intrinsics orqali turli oqimlarga tarqatiladi.
- Xotirada hech qanday lock yoki mutex talab etilmaydi.

### 3.3. Amaliy Kod

Quyida soddalashtirilgan, lekin qat'iy matematik qonuniyatlarga asoslangan Merkle Tree xesh hisoblagichi keltirilgan:

```rust
/// Kriptografik primitiv: 64-bitli soddalashtirilgan xesh-funksiya
fn fnv1a_hash(data: &[u8]) -> u64 {
    let mut hash = 0xcbf2_9ce4_8422_2325u64;
    for &byte in data {
        hash ^= byte as u64;
        hash = hash.wrapping_mul(0x0000_0100_0000_01b3u64);
    }
    hash
}

/// Ikki bolaning xeshlarini birlashtiruvchi ichki tugun xeshi
fn combine_hashes(left: u64, right: u64) -> u64 {
    let mut buf = [0u8; 16];
    buf[..8].copy_from_slice(&left.to_le_bytes());
    buf[8..].copy_from_slice(&right.to_le_bytes());
    fnv1a_hash(&buf)
}

/// BLAKE3 uslubidagi Merkle Tree ildiz (root) xeshini hisoblash
pub fn compute_merkle_root(chunks: &[&[u8]]) -> u64 {
    if chunks.is_empty() {
        return 0;
    }

    // 1-qadam: Barcha barglarning (leaf) xeshlarini parallel/ketma-ket hisoblaymiz
    let mut current_level: Vec<u64> = chunks.iter().map(|&chunk| fnv1a_hash(chunk)).collect();

    // 2-qadam: Daraxt bo'ylab ildizgacha birlashtirib chiqamiz
    while current_level.len() > 1 {
        let mut next_level = Vec::with_capacity((current_level.len() + 1) / 2);

        for pair in current_level.chunks(2) {
            if pair.len() == 2 {
                next_level.push(combine_hashes(pair[0], pair[1]));
            } else {
                // Agar toq element qolsa, o'zini yuqoriga o'tkazamiz
                next_level.push(pair[0]);
            }
        }
        current_level = next_level;
    }

    current_level[0]
}

fn main() {
    let data1 = b"Ferris Vault - Birinchi blok ma'lumotlari";
    let data2 = b"Ferris Vault - Ikkinchi blok ma'lumotlari";
    let data3 = b"Ferris Vault - Uchinchi blok ma'lumotlari";
    let data4 = b"Ferris Vault - To'rtinchi blok ma'lumotlari";

    let root1 = compute_merkle_root(&[data1, data2, data3, data4]);
    assert_ne!(root1, 0);

    // Xuddi shu ma'lumotlar yana hisoblansa bir xil root chiqadi (Determinizm):
    let root2 = compute_merkle_root(&[data1, data2, data3, data4]);
    assert_eq!(root1, root2);

    // Bitta belgi o'zgarsa butun ildiz xeshi o'zgaradi (Diffusion / Avalanche effect):
    let data2_tampered = b"Ferris Vault - Ikkinchi blok ma'Lumotlari";
    let root_tampered = compute_merkle_root(&[data1, data2_tampered, data3, data4]);
    assert_ne!(root1, root_tampered);
}
```
