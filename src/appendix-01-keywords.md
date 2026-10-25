# Ilova A: Kalit So‘zlar (Keywords)

Kalit so‘zlar — bu Rust tili tomonidan maxsus sintaktik vazifalar uchun band qilingan so‘zlardir. Ushbu so‘zlarni o‘zgaruvchilar, funksiyalar, strukturalar yoki modullar nomi sifatida ishlatib bo‘lmaydi (agar maxsus xom identifikator `r#so‘z` sintaksisi qo‘llanilmasa).

---

## 1. Amaldagi Kalit So‘zlar

Quyidagi jadvalda hozirgi kunda Rust tilida faol ishlatilayotgan barcha asosiy kalit so‘zlar jamlangan:

| Kalit So‘z | Vazifasi va Tavsifi |
| :--- | :--- |
| **`as`** | Birlamchi turlarni bir-biriga o‘girish (type casting) yoki importlarni qayta nomlash. |
| **`async`** | Asinxron blok yoki funksiya e’lon qilish (`Future` qaytaradi). |
| **`await`** | Asinxron `Future` natijasini kutish. |
| **`break`** | Tsikldan darhol chiqib ketish (va ixtiyoriy ravishda qiymat qaytarish). |
| **`const`** | Kompilyatsiya vaqtida hisoblanuvchi o‘zgarmas qiymatlarni aniqlash. |
| **`continue`** | Tsiklning joriy qadamini to‘xtatib, keyingi iteratsiyaga o‘tish. |
| **`crate`** | Modul yo‘llarida crate ildiziga murojaat qilish. |
| **`dyn`** | Trait obyektlariga dinamik jo‘natish (dynamic dispatch) prefiksi. |
| **`else`** | `if` yoki `if let` sharti qanoatlantirilmaganda bajariladigan blok. |
| **`enum`** | Sanam (Enumeration) turini e’lon qilish. |
| **`extern`** | Tashqi tillardagi C-kutubxonalarni (FFI) ulash. |
| **`false`** | Mantiqiy (boolean) yolg‘on qiymat literali. |
| **`fn`** | Yangi funksiya yoki funksiya ko‘rsatkichini e’lon qilish. |
| **`for`** | To‘plamlar ustida iteratsiya qilish yoki trait implementatsiyasini bog‘lash (`impl Trait for Type`). |
| **`if`** | Shartli tarmoqlanish ifodasi. |
| **`impl`** | Struktura, enum yoki traitlarga metodlar bog‘lash bloki. |
| **`in`** | `for` tsikli sintaksisining bir qismi. |
| **`let`** | Yangi o‘zgaruvchini e’lon qilish va unga qiymat biriktirish. |
| **`loop`** | Cheksiz shartsiz takrorlanish tsikli. |
| **`match`** | Qiymatni andazalarga moslashtirish (pattern matching). |
| **`mod`** | Yangi modul e’lon qilish. |
| **`move`** | Klojurlarda atrofdagi o‘zgaruvchilar egaligini to‘liq o‘zlashtirish. |
| **`mut`** | O‘zgaruvchi yoki havolalarning qiymatini o‘zgartirishga ruxsat berish. |
| **`pub`** | Modul, funksiya yoki maydonlarni ommaviy (tashqi kodga ochiq) qilish. |
| **`ref`** | Andazalarda qiymatni ko‘chirmasdan havola sifatida tutib olish. |
| **`return`** | Funksiyadan muddatidan oldin qiymat bilan chiqish. |
| **`self`** | Metod ichida joriy obyekt nusxasiga murojaat qilish. |
| **`Self`** | Joriy amalga oshirilayotgan tip nomini ifodalovchi tur taxallusi. |
| **`static`** | Dasturning butun umri davomida yashaydigan global o‘zgaruvchi. |
| **`struct`** | Yangi struktura ma’lumot turini yaratish. |
| **`super`** | Modul yo‘llarida ota (yuqori) modulga murojaat qilish. |
| **`trait`** | Yangi xislat (interfeys) e’lon qilish. |
| **`true`** | Mantiqiy (boolean) rost qiymat literali. |
| **`type`** | Yangi tur taxallusi (type alias) yaratish yoki bog‘liq turni e’lon qilish. |
| **`unsafe`** | Kompilyator xavfsizlik cheklovlaridan tashqari maxsus kod bloki. |
| **`use`** | Tashqi elementlarni joriy ko‘rinish sohasiga (scope) olib kirish. |
| **`where`** | Generiklar uchun murakkab Trait Bound cheklovlarini kiritish bloki. |
| **`while`** | Shartli takrorlanish tsikli. |

---

## 2. Kelajak Uchun Zahiralangan Kalit So‘zlar

Quyidagi so‘zlar hozirda faol sintaksisga ega bo‘lmasligi mumkin, ammo Rust tilining kelgusi versiyalariga kutilmagan ziddiyatlarsiz yangi funksiyalar qo‘shish maqsadida zahiraga olingan:

- `abstract`
- `become`
- `box`
- `do`
- `final`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

### Xom Identifikatorlar (`r#`)
Agar siz boshqa tilda yozilgan tashqi kutubxonadagi Rust kalit so‘zi bilan mos tushuvchi nomga (masalan, `match` nomli funksiyaga) murojaat qilishingiz kerak bo‘lsa, **`r#match()`** ko‘rinishidagi xom identifikatorlardan foydalanishingiz mumkin.
