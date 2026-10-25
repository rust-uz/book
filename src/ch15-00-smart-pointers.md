# 15-Bob: Aqlli Ko‘rsatkichlar (Smart Pointers)

Ko‘rsatkich (*pointer*) — bu xotiradagi ma’lum bir manzilni o‘zida saqlovchi o‘zgaruvchidir. Rust da eng keng tarqalgan ko‘rsatkich — bu biz avval ko‘rgan oddiy **havolalar (`&`)**dir. Ular faqat ma’lumotni qarzga oladi va hech qanday qo‘shimcha xarajatga ega emas.

**Aqlli ko‘rsatkichlar (Smart Pointers)** esa oddiy havolalardan farq qiladi:
1. Ular xotira manzilidan tashqari qo‘shimcha **metama’lumotlar** va imkoniyatlarga ega;
2. Ko‘pincha ular qaratilgan ma’lumotga **to‘liq egalik (Ownership)** qiladilar;
3. Aslida ular `Deref` va `Drop` traitlarini amalga oshirgan maxsus strukturalardir (masalan, biz ko‘rgan `String` va `Vec<T>` ham aslida aqlli ko‘rsatkich hisoblanadi!).

Ushbu bobda standart kutubxonaning eng mashhur aqlli ko‘rsatkichlarini chuqur tahlil qilamiz:
- **`Box<T>`:** Ma’lumotlarni Heap da saqlash uchun;
- **`Rc<T>`:** Bir nechta egaga ega bo‘lish (Reference Counting) uchun;
- **`RefCell<T>`:** Kompilyatsiya vaqtida emas, ish vaqtida (runtime) qarz olish qoidalarini boshqarish (Interior Mutability) uchun;
- **`Weak<T>`:** Tsiklik havolalar va xotira sizib chiqishining oldini olish uchun.

---

## 1. `Deref` va `Drop` Traitlari

Har qanday aqlli ko‘rsatkich ikkita asosiy traitga tayanadi:

### A. `Deref` Traiti (Dereference Operatori `*`)
`Deref` traiti aqlli ko‘rsatkichni oddiy havola kabi ishlatishga (`*ko‘rsatkich`) imkon beradi.

```rust
use std::ops::Deref;

struct MeningBox<T>(T);

impl<T> MeningBox<T> {
    fn new(x: T) -> MeningBox<T> {
        MeningBox(x)
    }
}

impl<T> Deref for MeningBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MeningBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y); // *y aslida *(y.deref()) ga aylanadi!
}
```

#### Deref Coercion (Deref O‘girilishi)
Rust avtomatik tarzda `&MeningBox<String>` ni `&str` ga o‘girib beradi. Bu funksiyalarga argument uzatishni nihoyatda osonlashtiradi.

### B. `Drop` Traiti (Tozalash K构)
`Drop` traiti o‘zgaruvchi ko‘rinish sohasidan chiqib ketayotgan paytda nima sodir bo‘lishini boshqaradi:

```rust
struct MaxsusResurs {
    malumot: String,
}

impl Drop for MaxsusResurs {
    fn drop(&mut self) {
        println!("MaxsusResurs tozalanmoqda: {}", self.malumot);
    }
}

fn main() {
    let c = MaxsusResurs { malumot: String::from("fayl_oqimi") };
    // Blok tugashi bilan drop() avtomatik chaqiriladi!
}
```
Agar resursni muddatidan oldin bo‘shatish kerak bo‘lsa, standart funksiya `std::mem::drop(c)` chaqiriladi (`c.drop()` deb to‘g‘ridan-to‘g‘ri chaqirish taqiqlangan).

---

## 2. `Box<T>`: Heap Xotiradan Joy Ajratish

`Box<T>` eng oddiy aqlli ko‘rsatkich bo‘lib, u qiymatni Stack o‘rniga **Heap**da saqlaydi. Stack da faqatgina Heap ga ko‘rsatuvchi 1 ta manzil qoladi.

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {b}");
}
```

### Rekursiv Turlar (Masalan, Bog‘langan Ro‘yxat - Cons List)
Rust har bir tur xotirada qancha bayt egallashini kompilyatsiya paytida bilishi shart. Agar tur o‘z-o‘zini saqlasa (rekursiv bo‘lsa), uning o‘lchami cheksiz bo‘lib qoladi:

```rust
// Bu kod kompilyatsiya BO'LMAYDI:
// enum List {
//     Cons(i32, List),
//     Nil,
// }
```

Kompilyator `List` ning hajmini hisoblay olmaydi. Yechim — **`Box<T>`** ishlatish! Chunki ko‘rsatkichning hajmi har doim qat’iy (64-bit tizimda 8 bayt):

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

```text
Stack                  Heap
+-------+             +---+------+
| list  |------------>| 1 |  *---|-----+
+-------+             +---+------+     |
                                       v
                                  +---+------+
                                  | 2 |  *---|-----+
                                  +---+------+     |
                                                   v
                                              +---+-----+
                                              | 3 | Nil |
                                              +---+-----+
```

---

## 3. `Rc<T>`: Havolalarni Sanovchi Ko‘rsatkich (Reference Counted)

Rust ning asosiy qoidasiga ko‘ra, qiymatning faqat bitta egasi bo‘lishi mumkin. Ammo ba’zan bitta ma’lumotga bir nechta obyekt bir vaqtning o‘zida to‘liq egalik qilishi talab etiladi (masalan, grafikdagi bitta tugunga bir nechta qirralar ulangan bo‘lsa).

Buning uchun **`Rc<T>`** (Reference Counted) xizmat qiladi:
- U Heap dagi qiymatga nechta faol havola borligini hisoblab boradi (`strong_count`).
- Har safar `Rc::clone(&a)` chaqirilganda, yangi nusxa olinmaydi, faqat hisoblagich +1 ga oshadi.
- Hisoblagich 0 ga tenglashgandagina Heap dagi xotira tozalanadi.
- *Diqqat:* `Rc<T>` faqat bitta oqimli (single-thread) muhit uchun mo‘ljallangan!

```rust
use std::rc::Rc;

enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil))))));
    println!("a yaratilgandan so'ng hisob: {}", Rc::strong_count(&a)); // 1

    let b = Cons(3, Rc::clone(&a));
    println!("b yaratilgandan so'ng hisob: {}", Rc::strong_count(&a)); // 2

    {
        let c = Cons(4, Rc::clone(&a));
        println!("c yaratilgandan so'ng hisob: {}", Rc::strong_count(&a)); // 3
    }

    println!("c o'chgandan so'ng hisob: {}", Rc::strong_count(&a)); // 2
}
```

---

## 4. `RefCell<T>` va Ichki O‘zgaruvchanlik (Interior Mutability)

Rust da o‘zgarmas o‘zgaruvchini o‘zgartirib bo‘lmasligini bilamiz. Ammo ba’zan ma’lumot tashqi tomondan o‘zgarmas bo‘lib ko‘rinsa-da, uning ichki qismini o‘zgartirish zarurati tug‘iladi. Bunga **Interior Mutability (Ichki o‘zgaruvchanlik)** andazasi deyiladi.

`RefCell<T>` borrow checking qoidalarini kompilyatsiya paytida emas, balki **ish vaqtida (runtime)** tekshiradi:
- `.borrow()` — o‘zgarmas `Ref<T>` qarz oladi;
- `.borrow_mut()` — o‘zgaruvchan `RefMut<T>` qarz oladi.

Agar dastur ish vaqtida bir vaqtning o‘zida ikkita `borrow_mut()` olishga urinsa, kompilyatsiya xatosi emas, balki dastur ish vaqtida `panic!` qiladi!

### `Rc<T>` va `RefCell<T>` ni Birlashtirish
Ko‘pincha bir nechta egaga ega bo‘lgan va o‘zgaruvchan bo‘lgan ma’lumotlar kerak bo‘ladi: `Rc<RefCell<T>>`.

```rust
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let qiymat = Rc::new(RefCell::new(5));

    let a = Rc::clone(&qiymat);
    let b = Rc::clone(&qiymat);

    *qiymat.borrow_mut() += 10;

    println!("a orqali: {}", a.borrow()); // 15
    println!("b orqali: {}", b.borrow()); // 15
}
```

---

## 5. Tsiklik Havolalar va `Weak<T>`

Agar ikkita `Rc` bir-biriga ko‘rsatib qolsa (masalan, bola otasiga, ota esa bolasiga), ularning hisoblagichi hech qachon 0 ga tushmaydi va bu **Xotira Sizib Chiqishi (Memory Leak)**ga olib keladi.

Buning oldini olish uchun kuchsiz ko‘rsatkich — **`Weak<T>`** (`Rc::downgrade`) ishlatiladi:
- U egalik huquqiga ega emas;
- `weak_count` hisobini oshiradi, lekin bu obyektning xotiradan tozalanishiga to‘sqinlik qilmaydi;
- Ma’lumotga kirishdan oldin `weak.upgrade()` chaqirilib, qiymat hali ham xotirada bor-yo‘qligi (`Option<Rc<T>>`) tekshiriladi.

Bu daraxtsimon va grafsimon tuzilmalarda ota-bola munosabatlarini xavfsiz ifodalash uchun ayni muddaodir.
