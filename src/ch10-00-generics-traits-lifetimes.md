# 10-Bob: Umumiy Tiplar, Xislatlar (Traits) va Yashash Muddati (Lifetimes)

Dasturlashda kod takrorlanishining oldini olish va umumiy yechimlar yaratish eng muhim mahoratlardan biridir. Rust ushbu maqsad uchun bir-birini to‘ldiruvchi uchta kuchli vositani taqdim etadi:

1. **Umumiy Tiplar (Generics):** Turli xil ma’lumot turlari uchun yagona mantiq yaratish.
2. **Xislatlar (Traits):** Turli tiplar bajarishi mumkin bo‘lgan umumiy xatti-harakatlarni (interfeyslarni) aniqlash.
3. **Yashash Muddati (Lifetimes):** Kompilyatorga havolalar qancha vaqt yaroqli ekanligini ko‘rsatib beruvchi mexanizm.

---

## 1. Umumiy Tiplar (Generics)

Tasavvur qiling, biz massivdagi eng katta elementni topmoqchimiz. `i32` uchun bitta funksiya, `char` uchun alohida funksiya yozish o‘rniga, umumiy **`T`** parametri orqali yagona funksiya yaratamiz:

```rust
fn eng_katta<T: std::cmp::PartialOrd>(royxat: &[T]) -> &T {
    let mut eng = &royxat[0];

    for element in royxat {
        if element > eng {
            eng = element;
        }
    }

    eng
}

fn main() {
    let sonlar = vec![34, 50, 25, 100, 65];
    println!("Eng katta son: {}", eng_katta(&sonlar));

    let belgilar = vec!['y', 'm', 'a', 'q'];
    println!("Eng katta belgi: {}", eng_katta(&belgilar));
}
```

### Struktura va Enum Ta’riflarida Generiklar
Biz avval ko‘rgan `Option<T>` va `Result<T, E>` turlari aynan generiklar yordamida qurilgan:

```rust
struct Nuqta<T, U> {
    x: T,
    y: U,
}

let butun_va_kasr = Nuqta { x: 5, y: 4.0 };
```

### Monomorfizatsiya (Nol-xarajatli Abstraksiya)
Rust da generiklardan foydalanish dasturning ishlash tezligiga **hech qanday salbiy ta’sir ko‘rsatmaydi (Zero-cost Abstraction)**!

Kompilyatsiya paytida Rust siz qaysi aniq tiplar bilan generikni chaqirganingizni aniqlaydi va har bir tip uchun alohida optimallashtirilgan mashina kodini ishlab chiqadi (**Monomorphization**). Dastur ish vaqtida go‘yo siz har bir tip uchun qo‘lda alohida kod yozgandek maksimal tezlikda ishlaydi!

---

## 2. Xislatlar: Umumiy Xatti-Harakatni Aniqlash (Traits)

**Xislat (Trait)** — ma’lum bir tip taqdim etishi shart bo‘lgan metodlar to‘plamidir. Bu boshqa tillardagi **interfeyslar (interfaces)**ga juda o‘xshab ketadi.

### Trait Yaratish va Uni Amalga Oshirish (Implement)

```rust
// Trait e'lon qilamiz:
pub trait Xulosa {
    fn xulosani_ol(&self) -> String;
}

pub struct Yangilik {
    pub sarlavha: String,
    pub muallif: String,
    pub matn: String,
}

// Yangilik strukturasiga Xulosa traitini ulaymiz:
impl Xulosa for Yangilik {
    fn xulosani_ol(&self) -> String {
        format!("{}, muallif: {}", self.sarlavha, self.muallif)
    }
}

pub struct Tvitt {
    pub foydalanuvchi: String,
    pub xabar: String,
}

impl Xulosa for Tvitt {
    fn xulosani_ol(&self) -> String {
        format!("{}: {}", self.foydalanuvchi, self.xabar)
    }
}
```

### Trait Parametr Sifatida (Trait Bounds)
Funksiyalarga "faqat ushbu traitni bajargan har qanday tip qabul qilinsin" deb shart qo‘yishimiz mumkin:

```rust
// Oddiy uslub:
pub fn xabar_ber(element: &impl Xulosa) {
    println!("Yangi xulosa: {}", element.xulosani_ol());
}

// Aniqroq Trait Bound sintaksisi:
pub fn xabar_ber_kengaytirilgan<T: Xulosa>(element: &T) {
    println!("Yangi xulosa: {}", element.xulosani_ol());
}
```

Bir nechta trait talab qilish (`+` orqali) va `where` bloklari:
```rust
fn murakkab_funksiya<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // ...
}
```

---

## 3. Yashash Muddati (Lifetimes)

Rust dagi barcha havolalar o‘z yashash muddatiga (*lifetime*) ega. Ko‘pincha kompilyator buni o‘zi avtomatik hisoblab chiqadi (*Lifetime Elision Rules*). Ammo funksiya bir nechta havolalarni qabul qilib, ulardan birini qaytarayotganda, qaysi biri qaytishini kompilyator oldindan bila olmaydi.

### Muammoning Ko‘rinishi

```rust
// Bu kod kompilyatsiya BO'LMAYDI:
// fn eng_uzun(x: &str, y: &str) -> &str {
//     if x.len() > y.len() { x } else { y }
// }
```

Kompilyator quyidagi haqli savolni beradi:
> *"Qaytadigan havola `x` ga bog‘liqmi yoki `y` gami? Agar ulardan biri xotiradan o‘chib ketsa, qaytgan natija xavfli bo‘lib qolmaydimi?"*

### Yashash Muddati Sintaksisi (`'a`)

Yashash muddatini ko‘rsatish uchun apostrof (`'`) bilan boshlanuvchi kichik nomlar (odatda `'a`) ishlatiladi:

```rust
fn eng_uzun<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### Bu Nimani Anglatadi?
- `'a` parametri havolalarning qancha yashashini **o‘zgartirmaydi**.
- U shunchaki kompilyatorga quyidagi munosabatni kafolatlaydi:
  > *"Qaytgan havola `x` va `y` ning eng qisqa umr ko‘radiganidan ko‘proq yashay olmaydi!"*

Keling, amalda ko‘raylik:

```rust
fn main() {
    let matn1 = String::from("uzun matn bu yerda");

    {
        let matn2 = String::from("qisqa");
        let natija = eng_uzun(matn1.as_str(), matn2.as_str());
        println!("Eng uzun: {natija}"); // Bu to'g'ri ishlaydi!
    }

    // Ammo agar `natija` ni shu yerda ishlatmoqchi bo'lsak, 
    // kompilyator darhol xato beradi, chunki `matn2` allaqachon o'chib ketgan!
}
```

### Strukturalarda Yashash Muddati
Agar struktura o‘z ichida boshqa ma’lumotga havolani saqlasa, strukturaning o‘zi o‘sha ma’lumotdan ko‘proq yashamasligi uchun unga ham lifetime belgilanadi:

```rust
struct MuhimQism<'a> {
    qism: &'a str,
}

fn main() {
    let gap = String::from("Rust tili ajoyib. Tez va xavfsiz.");
    let birinchi_gap = gap.split('.').next().unwrap();

    let i = MuhimQism {
        qism: birinchi_gap,
    };
}
```

### Statik Yashash Muddati: `'static`
`'static` maxsus yashash muddati bo‘lib, ma’lumot dasturning butun ishlash davri mobaynida xotirada saqlanishini bildiradi. Barcha satr literallari (`"salom"`) avtomatik ravishda `'static` hisoblanadi.

Generiklar, Traitlar va Yashash muddati birgalikda Rust tilini ham haddan tashqari xavfsiz, ham cheksiz darajada moslashuvchan qiladi!
