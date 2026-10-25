# 17-Bob: Rust da OOP Xususiyatlari (Object-Oriented Programming)

Obyektga Yo‘naltirilgan Dasturlash (OOP) — dasturiy ta’minotni obyektlar (ma’lumotlar va ularning ustida bajariluvchi metodlar) vositasida modellashtirish usulidir.

Ko‘plab yangi kelgan dasturchilar: *"Rust OOP tilimi?"* degan savolni berishadi. Bu savolning javobi siz OOP deganda nimani tushunishingizga bog‘liq:
- **Inkapsulyatsiya:** Bor! Strukturalar va `pub` maxfiylik chegarasi orqali.
- **Polimorfizm:** Bor! Traitlar va Trait Obyektlari orqali.
- **Merosxo‘rlik (Inheritance):** **YO‘Q!** Rust klassik sinflar merosxo‘rligini qo‘llab-quvvatlamaydi va buni ataylab qilgan.

Ushbu bobda Rust an’anaviy OOP masalalarini qanday nafis va xavfsiz hal qilishini ko‘rib chiqamiz.

---

## 1. Inkapsulyatsiya (Encapsulation)

Inkapsulyatsiya obyektning ichki holatini tashqi dunyodan yashirish va faqat ruxsat berilgan metodlar orqali boshqarishni anglatadi.

Rust da bu `pub` kalit so‘zi yordamida hal qilinadi:

```rust
pub struct OrtachaKolleksiya {
    royxat: Vec<i32>,
    ortacha: f64,
}

impl OrtachaKolleksiya {
    pub fn new() -> Self {
        OrtachaKolleksiya {
            royxat: vec![],
            ortacha: 0.0,
        }
    }

    pub fn qosh(&mut self, qiymat: i32) {
        self.royxat.push(qiymat);
        self.ortachani_yangila();
    }

    pub fn ortacha(&self) -> f64 {
        self.ortacha
    }

    fn ortachani_yangila(&mut self) {
        let jami: i32 = self.royxat.iter().sum();
        self.ortacha = jami as f64 / self.royxat.len() as f64;
    }
}
```
Tashqi kod `royxat` ga to‘g‘ridan-to‘g‘ri qiymat tiqa olmaydi, shuning uchun `ortacha` qiymati doimo 100% to‘g‘ri bo‘lishi kafolatlanadi.

---

## 2. Nega Merosxo‘rlik Yo‘q?

Ko‘plab an’anaviy OOP tillarida (Java, C++) yangi sinf mavjud sinfdan meros oladi (`extends`). Ammo katta loyihalarda bu quyidagi jiddiy muammolarga olib keladi:
1. **Frangible Base Class Problem (Mo‘rt ota sinf):** Ota sinfdagi bitta kichik o‘zgarish butun avlod sinflarining kutilmaganda sinishiga olib kelishi mumkin.
2. **Haddan tashqari ortiqcha yuk:** Farzand sinf ota sinfning o‘ziga mutlaqo kerak bo‘lmagan barcha metodlarini ham meros qilib olishga majbur bo‘ladi.

Rust "Merosxo‘rlik o‘rniga kompozitsiya va traitlarni afzal ko‘ring" (*Composition over Inheritance*) qoidasiga to‘liq amal qiladi.

---

## 3. Dinamik Polimorfizm: Trait Obyektlari (`dyn Trait`)

Biz 10-bobda generiklar orqali **Statik jo‘natish (Static Dispatch)**ni ko‘rdik. U yerda kompilyator barcha tiplarni oldindan bilib, kodni monomorfizatsiya qiladi.

Ammo agar bizga bir vaqtning o‘zida turli xil tiplarni bitta to‘plamda saqlash kerak bo‘lsa-chi? Masalan, grafik interfeysda tugma (`Button`), matn maydoni (`TextField`) va rasm (`Image`) — barchasini bitta ekranda chizmoqchimiz.

Rust buning uchun **Trait Obyektlari (`dyn Trait`)** mexanizmini beradi:

```rust
pub trait Chiziluvchi {
    fn chiz(&self);
}

pub struct Ekran {
    // dyn Chiziluvchi traitini bajargan istalgan har xil tiplar to'plami:
    pub komponentlar: Vec<Box<dyn Chiziluvchi>>,
}

impl Ekran {
    pub fn ishga_tushir(&self) {
        for komponent in &self.komponentlar {
            komponent.chiz();
        }
    }
}

pub struct Tugma {
    pub matn: String,
}

impl Chiziluvchi for Tugma {
    fn chiz(&self) {
        println!("[Tugma: {}]", self.matn);
    }
}

pub struct TanlovQutisi {
    pub tanlangan: bool,
}

impl Chiziluvchi for TanlovQutisi {
    fn chiz(&self) {
        println!("[TanlovQutisi: {}]", self.tanlangan);
    }
}

fn main() {
    let ekran = Ekran {
        komponentlar: vec![
            Box::new(Tugma { matn: String::from("Tasdiqlash") }),
            Box::new(TanlovQutisi { tanlangan: true }),
        ],
    };

    ekran.ishga_tushir();
}
```

### Dinamik Jo‘natish (Dynamic Dispatch) Qanday Ishlaydi?
`dyn Chiziluvchi` ishlatilganda, Rust ish vaqtida qaysi metod chaqirilishini aniqlash uchun **vtable (virtual method table)** deb ataladigan ko‘rsatkichlar jadvalidan foydalanadi. Bu statik generiklarga nisbatan bir necha nanosekundga sekinroq, ammo juda yuqori moslashuvchanlik beradi.

---

## 4. Rust Tip Tizimi Bilan State Pattern (Typestate Pattern)

Klassik OOP kitoblarida (GoF) "Holat andazasi" (State Pattern) o‘rganiladi. Ammo Rust dasturchilari bu andazani odatda Rust ning kuchli tip tizimi orqali yanada mukammalroq — **Typestate Pattern** yordamida amalga oshiradilar:

Noto‘g‘ri harakatlarni (masalan, qoralama maqolani tasdiqlanmasdan nashr qilib yuborishni) ish vaqtidagi xato emas, balki **kompilyatsiya xatosi**ga aylantiramiz!

```rust
// Turli holatlar uchun alohida strukturalar:
pub struct Qoralama;
pub struct KoribChiqilmoqda;
pub struct NashrQilingan;

pub struct Maqola<Holat> {
    matn: String,
    holat: Holat,
}

impl Maqola<Qoralama> {
    pub fn yangi(matn: &str) -> Self {
        Maqola {
            matn: matn.to_string(),
            holat: Qoralama,
        }
    }

    pub fn tekshiruvga_yubor(self) -> Maqola<KoribChiqilmoqda> {
        Maqola {
            matn: self.matn,
            holat: KoribChiqilmoqda,
        }
    }
}

impl Maqola<KoribChiqilmoqda> {
    pub fn tasdiqlash(self) -> Maqola<NashrQilingan> {
        Maqola {
            matn: self.matn,
            holat: NashrQilingan,
        }
    }
}

impl Maqola<NashrQilingan> {
    pub fn matn(&self) -> &str {
        &self.matn
    }
}

fn main() {
    let maqola = Maqola::yangi("Rust dasturlash sirlari");
    // maqola.matn(); // KOMPILYATSIYA XATOSI! Qoralama maqolaning matnini o'qib bo'lmaydi!

    let tekshiruvda = maqola.tekshiruvga_yubor();
    let tayyor = tekshiruvda.tasdiqlash();

    println!("Nashr etildi: {}", tayyor.matn());
}
```

Bu Rust tilining haqiqiy go‘zalligidir: dastur mantig‘idagi xatolarni qidirib yurish o‘rniga, tip tizimi noto‘g‘ri kodni yozishingizga jismonan yo‘l qo‘ymaydi!
