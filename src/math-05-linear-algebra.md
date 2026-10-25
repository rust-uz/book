# 5-Modul: Chiziqli Algebra va Vektorlar Fazosi (Grafika va Tizimli Hisoblar)

Chiziqli algebra zamonaviy kompyuter grafikasi (Vulkan, DirectX, Metal), o'yin dvigatellari (Bevy, Unreal Engine), robototexnika hamda sun'iy intellekt (Machine Learning) tizimlarining asosiy matematik tilidir. Vektorlar va matritsalar ustidagi hisob-kitoblar kompyuter xotirasida millionlab marta takrorlanadi.

Ushbu modulda matritsalarning xotiradagi geometriyasini (Row-major vs Column-major), CPU keshlarini aldashdan saqlovchi algoritmlarni hamda 3D fazoda burchaklar qulflanishi (**Gimbal Lock**) muammosini va uni 4 o'lchovli sonlar — **Kvaternionlar (Quaternions)** orqali yechish matematikasini ko'rib chiqamiz.

---

## 1. Vektorlar, Matritsalar Ko‘paytmasi va Xotirada Joylashuvi (Row-major vs Col-major, SIMD)

Chiziqli fazodagi har qanday chiziqli almashtirish (masshtablash, siljitish, burish) matritsalar ko'paytmasiga keltiriladi.

### 1.1. Matematik Isbot

$A \in \mathbb{R}^{M \times K}$ va $B \in \mathbb{R}^{K \times N}$ matritsalari berilgan bo'lsin. Ularning ko'paytmasi $C = A \times B \in \mathbb{R}^{M \times N}$ matritsasining har bir elementi quyidagi skalyar ko'paytma bilan aniqlanadi:
$$C_{i, j} = \sum_{k=1}^K A_{i, k} \cdot B_{k, j}$$

Ushbu amallar soni $M \times N \times K$ ga teng, ya'ni kvadrat matritsalar uchun asimptotik vaqt murakkabligi:
$$T(n) = O(n^3)$$

#### Xotira Geometriyasi: Bir O'lchovli RAM
Kompyuterning asosiy xotirasi (RAM) ko'p o'lchovli emas, balki bir o'lchovli ketma-ket baytlar manzilidir ($[0, 1, 2, \dots]$).
$M \times N$ o'lchamli 2D matritsani 1D xotiraga joylashtirishning ikkita asosiy standarti mavjud:

1. **Row-Major (Qatorlar bo'yicha — Rust, C/C++, Python NumPy):**
   Qator elementlari xotirada ketma-ket yotadi.
   $$\text{index}(i, j) = i \times N + j$$
2. **Column-Major (Ustunlar bo'yicha — Fortran, MATLAB, OpenGL/WebGL):**
   Ustun elementlari xotirada ketma-ket yotadi.
   $$\text{index}(i, j) = j \times M + i$$

```text
2x2 Matritsa: [[1, 2],
               [3, 4]]

Row-Major xotirada:    [ 1 ] [ 2 ] [ 3 ] [ 4 ]   (0-qator, keyin 1-qator)
Column-Major xotirada: [ 1 ] [ 3 ] [ 2 ] [ 4 ]   (0-ustun, keyin 1-ustun)
```

#### Kesh Falokati (The Cache Miss Disaster):
Zamonaviy CPU lar xotiradan ma'lumotni bitta baytlab emas, balki **Kesh Liniyasi** (odatda 64 bayt = 16 ta `f32`) bloklari bilan o'qiydi (**Spatial Locality** — yonma-yon elementlar keshga birga yuklanadi).

Endi klassik matritsa ko'paytmasiga qaraymiz:
$$C[i][j] += A[i][k] \times B[k][j]$$
- $A[i][k]$: $k$ o'sishi bilan xotirada ketma-ket ($+1$ qadam) harakatlanadi $\implies$ L1 keshda deyarli 100% Cache Hit!
- $B[k][j]$: $k$ o'sishi bilan har qadamda $N$ ta element sakrab o'tadi ($k \times N + j$). Agar $N = 1024$ bo'lsa, har bir amal uchun boshqa kesh liniyasiga murojaat qilinadi $\implies$ **Deyarli 100% Cache Miss!** Protsessor hisoblash o'rniga yuzlab taktlar davomida RAM dan ma'lumot kutib bo'sh turadi (Memory-Stalled).

#### Matematik Yechim: Transponirlash (Transposition)
Matritsani transponirlaymiz: $B^T_{j, k} = B_{k, j}$.
U holda ko'paytirish formulasi:
$$C_{i, j} = \sum_{k=1}^K A_{i, k} \cdot B^T_{j, k}$$
Endi har ikkala matritsa ham xotiradan ketma-ket (qator bo'ylab, Stride-1) o'qiladi! Bu oddiy matematik o'zgartirish dastur tezligini $5\times - 10\times$ martagacha oshiradi.

### 1.2. Rust Xotira Modeli va SIMD Tezlashtirish
Rust tilida `[[f32; 4]; 4]` matritsasi stack xotirada yaxlit 64 baytli blok sifatida joylashadi:
- Zamonaviy x86 (AVX2/AVX-512) va ARM (NEON) protsessorlarida 128/256/512 bitli vektor registrlari mavjud.
- Bitta SIMD instruksiyasi (masalan, `_mm_mul_ps` yoki `fma` — Fused Multiply-Add) bir vaqtning o'zida 4 ta yoki 8 ta `f32` sonini parallel ko'paytiradi va qo'shadi.
- Rust kompilyatori (LLVM) xotirada ketma-ket yotgan ma'lumotlarni ko'rganda, avtomatik ravishda **Auto-Vectorization** ni yoqadi.

### 1.3. Amaliy Kod

Quyidagi Rust dasturi $4 \times 4$ matritsalar ko'paytmasini klassik usul hamda keshga moslashtirilgan (transponirlangan) usulda hisoblaydi va natijalarning aynan bir xilligini isbotlaydi:

```rust
pub type Matrix4 = [[f32; 4]; 4];

/// Matritsani transponirlash: B^T[j][i] = B[i][j]
pub fn transpose(m: &Matrix4) -> Matrix4 {
    let mut result = [[0.0; 4]; 4];
    for i in 0..4 {
        for j in 0..4 {
            result[j][i] = m[i][j];
        }
    }
    result
}

/// Kesh-do'st (Cache-friendly) matritsa ko'paytmasi
pub fn matmul_cache_friendly(a: &Matrix4, b: &Matrix4) -> Matrix4 {
    let b_t = transpose(b);
    let mut c = [[0.0; 4]; 4];

    for i in 0..4 {
        for j in 0..4 {
            // Ikkala qator ham ketma-ket xotiradan o'qiladi (Row-Row dot product)
            let mut dot = 0.0;
            for k in 0..4 {
                dot += a[i][k] * b_t[j][k];
            }
            c[i][j] = dot;
        }
    }
    c
}

/// Standart naiv ko'paytirish
pub fn matmul_naive(a: &Matrix4, b: &Matrix4) -> Matrix4 {
    let mut c = [[0.0; 4]; 4];
    for i in 0..4 {
        for j in 0..4 {
            for k in 0..4 {
                c[i][j] += a[i][k] * b[k][j];
            }
        }
    }
    c
}

fn main() {
    let a: Matrix4 = [
        [1.0, 2.0, 3.0, 4.0],
        [5.0, 6.0, 7.0, 8.0],
        [9.0, 1.0, 2.0, 3.0],
        [4.0, 5.0, 6.0, 7.0],
    ];

    let b: Matrix4 = [
        [2.0, 0.0, 1.0, 3.0],
        [1.0, 4.0, 2.0, 0.0],
        [0.0, 3.0, 5.0, 1.0],
        [4.0, 1.0, 0.0, 2.0],
    ];

    let res_naive = matmul_naive(&a, &b);
    let res_cache = matmul_cache_friendly(&a, &b);

    // Ikkala usul bir xil natija berishini tekshiramiz
    for i in 0..4 {
        for j in 0..4 {
            assert!((res_naive[i][j] - res_cache[i][j]).abs() < 1e-5);
        }
    }

    // Birlik matritsa bilan ko'paytirish
    let identity: Matrix4 = [
        [1.0, 0.0, 0.0, 0.0],
        [0.0, 1.0, 0.0, 0.0],
        [0.0, 0.0, 1.0, 0.0],
        [0.0, 0.0, 0.0, 1.0],
    ];
    let res_id = matmul_cache_friendly(&a, &identity);
    assert_eq!(a, res_id);
}
```

---

## 2. Fazoviy Aylantirishlar: Kvaternionlar (Quaternions) va Gimbal Lock Muammosi

Uch o'lchovli fazoda jismning fazoviy burilishini ifodalash uchun dastlab **Eyler burchaklari (Euler Angles: Pitch, Yaw, Roll)** qo'llangan. Ammo bu yondashuv fizik va matematik halokatga olib keladi.

### 2.1. Matematik Isbot: Gimbal Lock va Uning Bartaraf Etilishi

Eyler burchaklari orqali burish uchta ketma-ket koordinata o'qlari atrofida aylantirish matritsalari ko'paytmasidan iborat:
$$R = R_z(\psi) R_y(\theta) R_x(\phi)$$

#### Gimbal Lock (Kardan Qulfi) Matematikasi:
Agar $y$-o'qi atrofidagi burilish burchagi (Pitch) $\theta = 90^\circ$ ($\pi/2$) bo'lib qolsa:
$\cos(90^\circ) = 0$ va $\sin(90^\circ) = 1$.
Trigonometrik almashtirishlar natijasida aylantirish matritsasi quyidagi ko'rinishga keladi:
$$R = \begin{pmatrix}
0 & \sin(\psi - \phi) & \cos(\psi - \phi) \\
0 & 0 & 1 \\
-\cos(\psi - \phi) & 0 & 0
\end{pmatrix}$$
E'tibor bering: burchaklar faqat $(\psi - \phi)$ ayirmasi shaklida qoldi!
Bu shuni anglatadiki, $x$-o'qi (Roll) va $z$-o'qi (Yaw) atrofidagi harakatlar **bitta tekislikka tushib, birlashib ketdi**.
Erkinlik darajasi (Degree of Freedom) 3 dan 2 ga tushib qoladi! Tizim bitta yo'nalishdagi burchak boshqaruvini butunlay yo'qotadi. Bu tarixda mashhur kosmik kemalar (Apollo 11) va vertolyot avtopilotlarida halokatli vaziyatlarni vujudga keltirgan.

#### Uilyam Rouen Gamilton va Kvaternionlar (1843-yil):
Ushbu muammoni hal qilish uchun 4 o'lchovli giperkompleks sonlar fazosi — **Kvaternionlar ($\mathbb{H}$)** kashf etildi:
$$q = w + x\mathbf{i} + y\mathbf{j} + z\mathbf{k}$$
bu yerda $w \in \mathbb{R}$ — skalyar qism, $(x, y, z) \in \mathbb{R}^3$ — vektor qism.

Fundamental algebraik qoida:
$$\mathbf{i}^2 = \mathbf{j}^2 = \mathbf{k}^2 = \mathbf{i}\mathbf{j}\mathbf{k} = -1$$
$$\mathbf{i}\mathbf{j} = \mathbf{k}, \quad \mathbf{j}\mathbf{k} = \mathbf{i}, \quad \mathbf{k}\mathbf{i} = \mathbf{j}$$
$$\mathbf{j}\mathbf{i} = -\mathbf{k}, \quad \mathbf{k}\mathbf{j} = -\mathbf{i}, \quad \mathbf{i}\mathbf{k} = -\mathbf{j}$$
Ko'paytirish **nokommutativdir** ($q_1 q_2 \neq q_2 q_1$).

#### Gamilton Ko'paytmasi (Hamilton Product):
$$q_1 q_2 = (w_1 w_2 - \mathbf{v}_1 \cdot \mathbf{v}_2) + (w_1 \mathbf{v}_2 + w_2 \mathbf{v}_1 + \mathbf{v}_1 \times \mathbf{v}_2)$$

#### Birlik Kvaternion Bilan Vektorni Burish:
Fazodagi ixtiyoriy $\mathbf{v}$ vektorini birlik $\mathbf{u}$ o'qi atrofida $\theta$ burchakka burish uchun birlik kvaternion tuziladi:
$$q = \cos\left(\frac{\theta}{2}\right) + \mathbf{u} \sin\left(\frac{\theta}{2}\right), \quad \|q\| = 1$$
Vektor kvaternion shakliga keltiriladi: $p = 0 + \mathbf{v} = (0, \mathbf{v})$.
Burilgan yangi vektor mashhur **Sandvich Formulada** hisoblanadi:
$$p' = q \cdot p \cdot q^*$$
bu yerda $q^* = w - x\mathbf{i} - y\mathbf{j} - z\mathbf{k}$ — qo'shma (conjugate) kvaternion.

#### Nega Kvaternionda Gimbal Lock Bo'lmaydi?
Topologik nuqtai nazardan, 3D fazodagi barcha burilishlar guruhi $\mathrm{SO}(3)$ hisoblanadi. 4 o'lchovli birlik kvaternionlar esa $S^3$ sferasini tashkil qiladi.
Matematik jihatdan $S^3$ sferasi $\mathrm{SO}(3)$ guruhining **qo'shaloq universal qoplamasidir (Universal Covering Space)** ($S^3 \to \mathrm{SO}(3)$).
$S^3$ sferasida hech qanday qutblar yoki koordinata singulyarliklari (matritsa determinanti nolga teng bo'ladigan nuqtalar) mavjud emas! Har qanday ikki yo'nalish o'rtasida **SLERP (Spherical Linear Interpolation)** orqali mutlaqo silliq va o'zgarmas burchak tezligida harakatlanish mumkin.

### 2.2. Rust Xotira Modeli
Kvaternion Rust tilida quyidagicha saqlanadi:
```rust
#[repr(C)]
pub struct Quaternion {
    pub w: f32,
    pub x: f32,
    pub y: f32,
    pub z: f32,
}
```
- `repr(C)` atributi xotirada maydonlarning aniq ketma-ket yotishini ta'minlaydi ($4 \times 4 = 16$ bayt).
- **16 bayt** — bu x86-64 da bitta `__m128` (XMM) registri, ARM da esa bitta 128-bitli NEON registridir!
- Butun kvaternion CPU registriga bitta taktda yuklanadi va Gamilton ko'paytmasi apparat darajasida bir necha taktda bajariladi.

### 2.3. Amaliy Kod

Quyida kvaternionlar yordamida 3D fazoda vektorlarni buruvchi, Gamilton ko'paytmasini bajaruvchi to'liq Rust kodi keltirilgan:

```rust
use std::ops::Mul;

#[derive(Debug, Clone, Copy, PartialEq)]
pub struct Vec3 {
    pub x: f32,
    pub y: f32,
    pub z: f32,
}

impl Vec3 {
    pub fn new(x: f32, y: f32, z: f32) -> Self {
        Self { x, y, z }
    }

    pub fn length(&self) -> f32 {
        (self.x * self.x + self.y * self.y + self.z * self.z).sqrt()
    }

    pub fn normalize(&self) -> Self {
        let len = self.length();
        assert!(len > 0.0);
        Self {
            x: self.x / len,
            y: self.y / len,
            z: self.z / len,
        }
    }
}

#[repr(C)]
#[derive(Debug, Clone, Copy, PartialEq)]
pub struct Quaternion {
    pub w: f32,
    pub x: f32,
    pub y: f32,
    pub z: f32,
}

impl Quaternion {
    pub fn new(w: f32, x: f32, y: f32, z: f32) -> Self {
        Self { w, x, y, z }
    }

    /// O'q (axis) va burchak (theta radian) bo'yicha birlik kvaternion yaratish
    pub fn from_axis_angle(axis: Vec3, theta: f32) -> Self {
        let norm_axis = axis.normalize();
        let half_theta = theta * 0.5;
        let sin_half = half_theta.sin();

        Self {
            w: half_theta.cos(),
            x: norm_axis.x * sin_half,
            y: norm_axis.y * sin_half,
            z: norm_axis.z * sin_half,
        }
    }

    /// Kvaternion normasi: ||q|| = sqrt(w^2 + x^2 + y^2 + z^2)
    pub fn norm(&self) -> f32 {
        (self.w * self.w + self.x * self.x + self.y * self.y + self.z * self.z).sqrt()
    }

    /// Normalizatsiya qilib birlik kvaternionga aylantirish (||q|| = 1)
    pub fn normalize(&self) -> Self {
        let n = self.norm();
        assert!(n > 0.0);
        Self {
            w: self.w / n,
            x: self.x / n,
            y: self.y / n,
            z: self.z / n,
        }
    }

    /// Qo'shma kvaternion: q* = w - xi - yj - zk
    pub fn conjugate(&self) -> Self {
        Self {
            w: self.w,
            x: -self.x,
            y: -self.y,
            z: -self.z,
        }
    }

    /// 3D vektorni burish: p' = q * p * q* (faqat birlik kvaternionlar uchun)
    pub fn rotate_vector(&self, v: Vec3) -> Vec3 {
        let q_norm = self.normalize();
        let p = Quaternion::new(0.0, v.x, v.y, v.z);
        let p_prime = q_norm * p * q_norm.conjugate();
        Vec3::new(p_prime.x, p_prime.y, p_prime.z)
    }
}

/// Gamilton ko'paytmasi: q1 * q2
impl Mul for Quaternion {
    type Output = Self;

    fn mul(self, rhs: Self) -> Self {
        Self {
            w: self.w * rhs.w - self.x * rhs.x - self.y * rhs.y - self.z * rhs.z,
            x: self.w * rhs.x + self.x * rhs.w + self.y * rhs.z - self.z * rhs.y,
            y: self.w * rhs.y - self.x * rhs.z + self.y * rhs.w + self.z * rhs.x,
            z: self.w * rhs.z + self.x * rhs.y - self.y * rhs.x + self.z * rhs.w,
        }
    }
}

fn main() {
    // Z-o'qi atrofida 90 gradusga (PI / 2) burish kvaternioni:
    let z_axis = Vec3::new(0.0, 0.0, 1.0);
    let angle = std::f32::consts::FRAC_PI_2; // 90 gradus
    let q = Quaternion::from_axis_angle(z_axis, angle);

    // X-o'qidagi (1, 0, 0) vektorni buramiz
    let v = Vec3::new(1.0, 0.0, 0.0);
    let v_rotated = q.rotate_vector(v);

    // 90 gradus burilgach, vektor Y-o'qiga (0, 1, 0) o'tishi kerak:
    assert!((v_rotated.x - 0.0).abs() < 1e-5);
    assert!((v_rotated.y - 1.0).abs() < 1e-5);
    assert!((v_rotated.z - 0.0).abs() < 1e-5);

    assert!((q.norm() - 1.0).abs() < 1e-5);

    // Ikki marta 90 gradus burish (jami 180 gradus): (-1, 0, 0)
    let q180 = q * q;
    assert!((q180.norm() - 1.0).abs() < 1e-5);
    let v_180 = q180.rotate_vector(v);
    assert!((v_180.x - (-1.0)).abs() < 1e-5);
    assert!((v_180.y - 0.0).abs() < 1e-5);
}
```
