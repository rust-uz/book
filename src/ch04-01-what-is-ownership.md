# Egalik Nima? (What is Ownership?)

Rust dasturlash tilining eng muhim va o‘ziga xos xususiyati — bu **Egalik tizimi (Ownership System)**dir. U dasturchiga **Garbage Collector (axlat yig‘uvchi)siz** va xotirani qo‘lda boshqarishdagi xatoliklarsiz (dangling pointers, memory leaks) 100% xotira xavfsizligini taqdim etadi.

Ushbu tushunchani to‘liq anglash uchun, dastlab xotiraning ikkita asosiy qismi: **Stack** va **Heap** qanday ishlashini bilib olishimiz shart.

---

## Stack va Heap Xotirasi

Dastur ish vaqtida kompyuterning operativ xotirasi (RAM) asosan ikkita qismga bo‘linadi:

### 1. Stack (Stak xotirasi)
- **Tartib:** LIFO (Last In, First Out — Oxirgi kirgan birinchi chiqadi). Xuddi likopchalarni ustma-ust taxlash kabi: yangi likopcha tepaga qo‘yiladi, kerak bo‘lganda tepadan olinadi.
- **Tezlik:** Juda tezkor! Chunki xotira ajratish uchun shunchaki ko‘rsatkichni siljitish kifoya.
- **Talab:** Stack da saqlanadigan barcha ma’lumotlarning **hajmi kompilyatsiya vaqtida qat’iy aniq va o‘zgarmas** bo‘lishi shart (masalan: `i32`, `bool`, `f64`, o‘lchami qat’iy massivlar).

### 2. Heap (Uyulma xotira)
- **Xususiyat:** Hajmi oldindan noma’lum bo‘lgan yoki dastur davomida o‘sib-kamayishi mumkin bo‘lgan ma’lumotlar uchun ishlatiladi (masalan: foydalanuvchi kiritadigan matnlar, dinamik massivlar).
- **Qanday ishlaydi:** Dastur operatsion tizimdan "Menga 100 bayt joy ber" deb so‘raydi. Tizim yetarli bo‘sh joyni topadi, uni band deb belgilaydi va o‘sha joyning **boshlang‘ich manzilini (pointer)** qaytaradi.
- **Tezlik:** Stack ga nisbatan sekinroq, chunki xotira ajratuvchi (allocator) bo‘sh joy qidirishi va manzil orqali ma’lumotga murojaat qilishi kerak.

---

## `String` Turi Misolida Egalik

Biz avval o‘rgangan satr literallari (`"salom"`) dastur kodining o‘ziga biriktiriladi (hardcoded) va ular o‘zgarmasdir. Ammo agar dastur davomida foydalanuvchi kiritgan matnni o‘zgartirmoqchi bo‘lsak, Heap da saqlanuvchi **`String`** turidan foydalanamiz:

```rust
let mut s = String::from("salom");
s.push_str(", dunyo!"); // push_str satr oxiriga belgilarni qo'shadi
println!("{s}"); // "salom, dunyo!"
```

### Xotirada Bu Qanday Ko‘rinadi?

`String` Stack va Heap dan quyidagicha foydalanadi:

```text
       STACK                          HEAP
+----------+-------+           +---+---+---+---+---+
| ptr      |   *---|---------->| s | a | l | o | m |
| len      |   5   |           +---+---+---+---+---+
| capacity |   5   |
+----------+-------+
```

1. **Stack**da kichik 3 ta maydon saqlanadi (jami 24 bayt):
   - `ptr` — Heap dagi ma’lumotga ko‘rsatuvchi manzil;
   - `len` — matnning joriy baytlar soni (uzunligi);
   - `capacity` — Heap dan ajratilgan umumiy xotira sig‘imi.
2. **Heap**da esa haqiqiy matn belgilari (`'s', 'a', 'l', 'o', 'm'`) joylashadi.

---

## Xotiradan O‘chirish: RAII va `drop`

C tilida dasturchi har bir `malloc` uchun `free` yozishi shart. Agar unutsangiz — xotira sizib chiqadi (*memory leak*). Agar erta yozsangiz — *dangling pointer* yuzaga keladi.

Rust bu muammoni **RAII (Resource Acquisition Is Initialization)** tamoyili bilan hal qiladi:
> **Qachonki o‘zgaruvchi o‘z ko‘rinish sohasidan (scope `{}`) chiqib ketsa, unga tegishli barcha Heap xotirasi avtomatik tarzda bo‘shatiladi!**

Rust bu jarayonda yashirincha `drop` nomli maxsus funksiyani chaqiradi.

```rust
{
    let s = String::from("salom"); // s shu yerdan boshlab yaroqli
    // s bilan amallar...
} // Ushbu qavs yopilishi bilan Rust avtomatik s.drop() ni chaqiradi va xotira ozod etiladi!
```

---

## Ko‘chirish (Move) Semantikasi

Endi eng muhim savolga keldik: bitta o‘zgaruvchini ikkinchisiga biriktirsak nima bo‘ladi?

```rust
let s1 = String::from("salom");
let s2 = s1;
```

Boshqa ko‘plab tillarda bu shunchaki ikkinchi ko‘rsatkichni hosil qiladi (shallow copy). Agar Rust ham shunday qilganda edi, ikkala o‘zgaruvchi (`s1` va `s2`) blok tugaganda Heap dagi ayni bitta xotirani bo‘shatishga urinardi. Bunga **Double Free Error** deyiladi va u xotirani buzilishiga (memory corruption) olib keladi.

Rust bu xatoga yo‘l qo‘ymaslik uchun ajoyib qoidani qo‘llaydi:
**`let s2 = s1;` bajarilganda, `s1` ning egalik huquqi `s2` ga to‘liq o‘tadi (Move bo‘ladi), va `s1` darhol bekor qilinadi (invalid holatga o‘tadi)!**

```rust
let s1 = String::from("salom");
let s2 = s1;

// println!("{s1}"); // KOMPILYATSIYA XATOSI! `s1` endi mavjud emas!
println!("{s2}"); // Bu to'g'ri ishlaydi
```

```text
    STACK                           HEAP
s1 (bekor qilingan)
+----------+-------+           
| ptr      |   x   |           
+----------+-------+           

s2 (yangi ega)
+----------+-------+           +---+---+---+---+---+
| ptr      |   *---|---------->| s | a | l | o | m |
| len      |   5   |           +---+---+---+---+---+
| capacity |   5   |
+----------+-------+
```

Chunki `s1` bekor qilingan, blok oxirida faqatgina `s2` xotirani ozod qiladi — hech qanday double free bo‘lmaydi!

---

## Xotiradan To‘liq Nusxa Olish: `clone`

Agar bizga Heap dagi ma’lumotlarni ham to‘liq nusxalash (deep copy) kerak bo‘lsa, **`.clone()`** metodidan foydalanamiz:

```rust
let s1 = String::from("salom");
let s2 = s1.clone(); // Heap da alohida yangi xotira ajratiladi

println!("s1 = {s1}, s2 = {s2}"); // Ikkalasi ham faol!
```

---

## Stack Ma’lumotlari va `Copy` Traiti

Nega butun sonlar bilan ishlaganda bu holat yuz bermaydi?

```rust
let x = 5;
let y = x;

println!("x = {x}, y = {y}"); // Xatolik yo‘q!
```

Chunki `i32` kabi o‘lchami aniq turlar faqat **Stack**da saqlanadi. Stack da nusxa olish juda tez va arzon bo‘lgani uchun, Rust bunday turlarga **`Copy`** traitini (xislatini) beradi. 

Quyidagi turlar `Copy` ga ega:
- Barcha butun sonlar (`u32`, `i64` va h.k.);
- Mantiqiy tip (`bool`);
- Suzuvchi nuqtali sonlar (`f64`, `f32`);
- Belgilar (`char`);
- Faqat `Copy` turlardan tashkil topgan kortejlar (masalan: `(i32, i32)`).
