# 16-Bob: Qo‘rqmas Paralellik (Fearless Concurrency)

Ko‘p oqimli (multithreaded) dasturlash ko‘pincha dasturchilar uchun eng qiyin va xavfli sohalardan biri hisoblanadi. Ma’lumotlar poygasi (*Data Races*), o‘lik qulflar (*Deadlocks*) va faqat ish vaqtida kamdan-kam chiqadigan sirli xatoliklar sabab ko‘pchilik undan hayiqadi.

Rust bu vaziyatni tubdan o‘zgartiradi va shuning uchun bu bob **"Qo‘rqmas Paralellik (Fearless Concurrency)"** deb ataladi. Rust ning Egalik va Tip tizimlari ma’lumotlar poygasi va xavfli xotira amallarini **kompilyatsiya paytida 100% aniqlaydi va taqiqlaydi**!

---

## 1. Oqimlarni Yaratish: `thread::spawn` va `join`

Yangi operatsion tizim oqimini (thread) yaratish uchun `std::thread::spawn` funksiyasi chaqiriladi:

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("Yangi oqimdan salom: {i}");
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("Asosiy oqimdan salom: {i}");
        thread::sleep(Duration::from_millis(1));
    }

    // handle.join() yangi oqim o'z ishini to'liq tugatishini kutadi:
    handle.join().unwrap();
}
```
Agar `handle.join()` chaqirilmasa, asosiy `main` oqimi tugashi bilan barcha yaratilgan yangi oqimlar o‘z ishini tugatmasdanoq to‘xtatib qo‘yiladi.

### Oqimlarga Ma’lumot Uzatish: `move` Klojuri
Yangi oqim qancha vaqt yashashini kompilyator oldindan bilolmaydi. Shuning uchun oqim ichiga asosiy oqimdagi o‘zgaruvchini olib kirish uchun uning **egaligini to‘liq o‘tkazish (`move`)** talab etiladi:

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Vektor qiymati: {v:?}");
    });

    handle.join().unwrap();
}
```

---

## 2. Xabarlar Almashish (Message Passing)

Mashhur Go va Rust shiorida aytilganidek:
> *"Xotirani ulashib muloqot qilmang; aksincha, muloqot qilib xotirani ulashing!"*

Rust standart kutubxonasi xabarlar almashish uchun **Kanallar (Channels)** mexanizmini taqdim etadi: `mpsc` (Multiple Producer, Single Consumer — Bir nechta jo‘natuvchi, bitta qabul qiluvchi).

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    // Kanal yaratamiz: tx - uzatuvchi (transmitter), rx - qabul qiluvchi (receiver)
    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone(); // Ikkinchi jo'natuvchi nusxasi

    // 1-oqim:
    thread::spawn(move || {
        let xabarlar = vec![
            String::from("oqim 1: birinchi"),
            String::from("oqim 1: ikkinchi"),
        ];

        for xabar in xabarlar {
            tx1.send(xabar).unwrap();
            thread::sleep(Duration::from_millis(200));
        }
    });

    // 2-oqim:
    thread::spawn(move || {
        let xabarlar = vec![
            String::from("oqim 2: salom"),
            String::from("oqim 2: dunyo"),
        ];

        for xabar in xabarlar {
            tx.send(xabar).unwrap();
            thread::sleep(Duration::from_millis(200));
        }
    });

    // Asosiy oqim xabarlarni navbat bilan qabul qiladi:
    for qabul_qilingan in rx {
        println!("Qabul qilindi: {qabul_qilingan}");
    }
}
```

### Egalik Huquqi va Kanallar Xavfsizligi
Diqqat qiling: `tx.send(val)` chaqirilishi bilan `val` ning egalik huquqi kanalga o‘tadi! Siz `send` dan so‘ng o‘sha qiymatni boshqa o‘zgartira olmaysiz yoki o‘qiy olmaysiz. Bu orqali ikki oqim ayni bir ma’lumotni buzib qo‘yishi mutlaqo imkonsiz qilinadi.

---

## 3. Ulashilgan Holat: `Mutex<T>` va `Arc<T>`

Xabarlar almashishdan tashqari, bir nechta oqim ayni bitta xotira blokini to‘g‘ridan-to‘g‘ri o‘zgartirishi kerak bo‘lgan vaziyatlar ham uchraydi.

### Mutex (Mutual Exclusion)
`Mutex<T>` ma’lumotga faqat bitta oqim bir vaqtning o‘zida kirishiga ruxsat beruvchi qulfdir.
- Oqim ma’lumotga kirish uchun dastlab `.lock()` so‘raydi;
- Rust sizga `MutexGuard` aqlli ko‘rsatkichini beradi;
- Oqim o‘z blokidan chiqib ketishi bilan (`drop`), qulf **avtomatik ravishda yechiladi**! Hech qachon qulfni qo‘lda ochishni unutib qoldira olmaysiz.

### Nega `Rc<T>` Emas, `Arc<T>`?
Oldingi bobda ko‘rgan `Rc<T>` ko‘rsatkichi oqimlararo uzatish uchun xavfsiz emas, chunki uning ichki hisoblagichi oddiy butun son bo‘lib, oqimlar bir vaqtda kirishganda hisob buzilishi mumkin.
Ko‘p oqimli tizimlar uchun **`Arc<T>` (Atomic Reference Counted)** ishlatiladi. U protsessorning apparat darajasidagi atomik amallaridan foydalanadi va 100% xavfsizdir.

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let hisoblagich = Arc::new(Mutex::new(0));
    let mut dastalar = vec![];

    for _ in 0..10 {
        let hisoblagich_nusxasi = Arc::clone(&hisoblagich);
        let dasta = thread::spawn(move || {
            let mut raqam = hisoblagich_nusxasi.lock().unwrap();
            *raqam += 1;
        });
        dastalar.push(dasta);
    }

    for dasta in dastalar {
        dasta.join().unwrap();
    }

    println!("Yakuniy hisob: {}", *hisoblagich.lock().unwrap()); // 10
}
```

---

## 4. `Send` va `Sync` Marker Traitlari

Rust kompilyatori qaysi turdagi ma’lumotlarni oqimlararo uzatish mumkinligini qanday biladi? Bu ikkita asosiy marker-trait orqali amalga oshiriladi:

1. **`Send`:** Agar tip `Send` traitiga ega bo‘lsa, uning egalik huquqini boshqa bir oqimga xavfsiz o‘tkazish mumkin. Deyarli barcha birlamchi Rust turlari `Send` dir (faqat `Rc` va xom ko‘rsatkichlar bundan mustasno).
2. **`Sync`:** Agar tip `Sync` traitiga ega bo‘lsa, unga bir nechta oqimdan havolalar (`&T`) orqali bir vaqtning o‘zida xavfsiz murojaat qilish mumkin (masalan, `Mutex<T>` har doim `Sync` dir).

Rust ning eng buyuk jihati shundaki, bu qoidalar dastur kodiga sun’iy cheklov sifatida kiritilmagan, balki tilning umumiy tip tizimi asosida ishlaydi. Natijada siz yozgan kodda hech qanday sirli data race bo‘lmaydi!
