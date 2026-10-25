# 19-Bob: Ilg‘or Xususiyatlar: Unsafe, FFI va Makroslar (Advanced Features)

Rust tilining ko‘p qismi kompilyatsiya vaqtidagi qat’iy xavfsizlik kafolatlari ustiga qurilgan. Ammo eng quyi tizimlar (operatsion tizim yadrolari, qurilmalar drayverlari, apparat ta’minoti bilan to‘g‘ridan-to‘g‘ri ishlash) yoki boshqa tillar (masalan, C/C++) bilan integratsiya qilishda tilning eng chuqur va ilg‘or imkoniyatlaridan foydalanish talab etiladi.

Ushbu bobda quyidagi mavzularni o‘rganamiz:
1. **Xavfsiz Bo‘lmagan Rust (Unsafe Rust);**
2. **Ilg‘or Xislatlar (Advanced Traits);**
3. **Ilg‘or Tiplar (Advanced Types);**
4. **Ilg‘or Funksiyalar va Klojurlar;**
5. **Makroslar Tizimi (Macros).**

---

## 1. Xavfsiz Bo‘lmagan Rust (Unsafe Rust)

Kompyuter apparaturasi mohiyatan o‘zida xavfsizlik kafolatiga ega emas. Agar Rust 100% xavfsiz bo‘lish bilan cheklanib qolganda, unda operatsion tizim yoki qurilma drayverini yozib bo‘lmas edi.

Shu sababli Rust ichida ikkinchi, yashirin til mavjud: **Unsafe Rust**.
Unsafe Rust borrow checkerni o‘chirib qo‘ymaydi, balki sizga **5 ta qo‘shimcha super-kuch (superpowers)** beradi:

1. **Xom ko‘rsatkichlarni (Raw Pointers) dereference qilish;**
2. **Xavfsiz bo‘lmagan (unsafe) funksiya yoki metodlarni chaqirish;**
3. **O‘zgaruvchan statik o‘zgaruvchilarni (`static mut`) o‘qish yoki o‘zgartirish;**
4. **Unsafe traitlarni amalga oshirish;**
5. **`union` maydonlariga murojaat qilish.**

### Xom Ko‘rsatkichlar (Raw Pointers: `*const T` va `*mut T`)
Oddiy havolalardan farqli ravishda:
- Ular borrow checker qoidalariga bo‘ysunmaydi (bir vaqtda ham o‘qish, ham yozish mumkin);
- Ular `null` bo‘lishi mumkin;
- Xotiradan o‘chib ketgan manzilga ko‘rsatishi mumkin.

Xom ko‘rsatkichni yaratish xavfsiz, ammo uning manzilidagi ma’lumotni ochish (`*`) faqat `unsafe` blok ichida amalga oshirilishi mumkin:

```rust
fn main() {
    let mut raqam = 5;

    let r1 = &raqam as *const i32;
    let r2 = &mut raqam as *mut i32;

    unsafe {
        println!("r1 manzildagi qiymat: {}", *r1);
        *r2 = 10;
        println!("r2 orqali yangilandi: {}", *r2);
    }
}
```

### Xavfsiz Bo‘lmagan Kod Ustiga Xavfsiz Abstraksiya Qurish
Rust ning eng yaxshi amaliyoti — unsafe kodni iloji boricha kichik hajmda saqlash va uning ustiga qulay, 100% xavfsiz API qurishdir.
Bunga standart kutubxonadagi `split_at_mut` funksiyasi ajoyib misol bo‘la oladi:

```rust
use std::slice;

fn massivni_bol<'a>(kesma: &'a mut [i32], oraliq: usize) -> (&'a mut [i32], &'a mut [i32]) {
    let uzunlik = kesma.len();
    let ptr = kesma.as_mut_ptr();

    assert!(oraliq <= uzunlik);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, oraliq),
            slice::from_raw_parts_mut(ptr.add(oraliq), uzunlik - oraliq),
        )
    }
}
```

### FFI (Foreign Function Interface): C Kutubxonalarini Chaqirish
Rust boshqa tillardagi (masalan, C) funksiyalarni to‘g‘ridan-to‘g‘ri chaqirish imkonini beradi:

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("C tilidagi abs(-3): {}", abs(-3));
    }
}
```

---

## 2. Ilg‘or Xislatlar (Advanced Traits)

### Bog‘liq Turlar (Associated Types)
Trait ichida aniq tur nomini generik parametr qilmasdan, trait bilan birga aniqlanadigan tur:

```rust
pub trait Iterator {
    type Item; // Bog'liq tur
    fn next(&mut self) -> Option<Self::Item>;
}
```

### Operatorlarni Qayta Yuklash (Operator Overloading)
Rust da `+`, `-`, `*` kabi operatorlarni o‘z strukturalaringiz uchun qayta aniqlashingiz mumkin (`std::ops` modullari orqali):

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Nuqta {
    x: i32,
    y: i32,
}

impl Add for Nuqta {
    type Output = Nuqta;

    fn add(self, other: Nuqta) -> Nuqta {
        Nuqta {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    let p1 = Nuqta { x: 1, y: 0 };
    let p2 = Nuqta { x: 2, y: 3 };
    assert_eq!(p1 + p2, Nuqta { x: 3, y: 3 });
}
```

---

## 3. Ilg‘or Tiplar (Advanced Types)

- **Type Aliases (Tur Taxallusi):** Uzun tur nomlarini qisqartirish:
  ```rust
  type Kilometr = i32;
  type Thunk = Box<dyn Fn() + Send + 'static>;
  ```
- **Never Type (`!`):** Hech qachon qiymat qaytarmaydigan funksiyalar (masalan, `panic!` yoki cheksiz `loop`):
  ```rust
  fn cheksiz() -> ! {
      loop {}
  }
  ```
- **Dinamik O‘lchamli Turlar (DST) va `Sized` Traiti:** Rust da deyarli barcha turlar kompilyatsiya paytida o‘z hajmiga ega (`Sized`). Havola orqali ishlatiladigan `str` yoki `dyn Trait` kabi turlar esa DST bo‘lib, ular har doim ko‘rsatkich orqali (`&str`, `Box<dyn Trait>`) ishlatiladi.

---

## 4. Makroslar Tizimi (Macros)

Makroslar — bu **kod generatsiya qiluvchi kod** (metadasturlash) dir. Biz tez-tez ishlatadigan `println!`, `vec!`, `format!` aynan makroslardir.

### Nega Funksiya Emas, Makros?
1. Makroslar ixtiyoriy sondagi parametrlarni qabul qila oladi (`println!("a")`, `println!("a", 1, 2)`);
2. Makroslar kompilyator kodni tahlil qilishidan oldin kengaytiriladi (expand bo‘ladi), shuning uchun ular yangi turlar va traitlarni hosil qila oladi.

### A. Deklarativ Makroslar (`macro_rules!`)
Andazalarga asoslangan makroslar:

```rust
#[macro_export]
macro_rules! mening_vektorim {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

fn main() {
    let v = mening_vektorim![1, 2, 3];
    println!("{v:?}");
}
```

### B. Protsedura Makroslari (Procedural Macros)
Kodni AST (Abstract Syntax Tree) ko‘rinishida qabul qilib, uni o‘zgartirib yangi Rust kodini qaytaruvchi murakkab makroslar:
1. **Custom Derive:** `#[derive(Serialize)]`
2. **Attribute-like:** `#[get("/")]`
3. **Function-like:** `sql!("SELECT * FROM users")`

Ushbu ilg‘or xususiyatlar Rust tilini dunyodagi eng qudratli va cheksiz imkoniyatlarga ega tizimli dasturlash vositasiga aylantiradi.
