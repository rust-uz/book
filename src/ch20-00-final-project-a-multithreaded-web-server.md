# 20-Bob: Yakuniy Loyiha: Ko‘p Oqimli Veb Server Qurish (Multithreaded Web Server)

Ushbu kitobdagi sayohatimizning eng cho‘qqisiga yetib keldik! Ushbu yakuniy bobda biz shu paytgacha o‘rgangan deyarli barcha bilimlarni — xotira boshqaruvi, aqlli ko‘rsatkichlar (`Arc`, `Mutex`, `Box`), kanallar (`mpsc`), ko‘p oqimlilik, traitlar, xatoliklarni qayta ishlash va RAII tozalash tamoyillarini bitta yirik, ishlab chiqarish darajasidagi loyihada birlashtiramiz.

Biz **haqiqiy, ko‘p oqimli HTTP veb-serverini** tashqi freymvorklarsiz, faqatgina Rust standart kutubxonasining o‘zidan foydalangan holda noldan quramiz!

---

## 1. TCP Ulanishlarini Tinglash

Veb-serverlar tarmoqda TCP (Transmission Control Protocol) protokoli orqali ishlaydi. Rust da TCP ulanishlarini qabul qilish uchun `std::net::TcpListener` ishlatiladi:

```rust,no_run
// src/main.rs
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    println!("Server http://127.0.0.1:7878 manzilida ishga tushdi...");

    for stream in listener.incoming() {
        let stream = stream.unwrap();
        println!("Yangi ulanish qabul qilindi!");
    }
}
```

Brauzerda `http://127.0.0.1:7878` manzilini ochsangiz, terminalda yangi ulanishlar paydo bo‘layotganini ko‘rasiz.

---

## 2. HTTP So‘rovini O‘qish va Javob Qaytarish

HTTP — bu matnli protokol. Brauzer serverga quyidagi ko‘rinishdagi so‘rov yuboradi:

```text
GET / HTTP/1.1
Host: 127.0.0.1:7878
User-Agent: Mozilla/5.0...
```

Server esa quyidagicha javob qaytarishi lozim:
```text
HTTP/1.1 200 OK\r\nContent-Length: <uzunlik>\r\n\r\n<HTML_MATN>
```

Keling, so‘rovni tahlil qilib, HTML sahifani qaytaruvchi kod yozamiz:

```rust,no_run
use std::fs;
use std::io::{prelude::*, BufReader};
use std::net::{TcpListener, TcpStream};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();
        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    let contents = fs::read_to_string(filename).unwrap_or_else(|_| {
        String::from("<h1>404 - Sahifa Topilmadi</h1>")
    });
    let length = contents.len();

    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

Loyihangiz papkasida `hello.html` va `404.html` fayllarini yarating. Endi serveringiz chiroyli sahifalarni uzata oladi!

---

## 3. Sekin So‘rov Muammosi (The Slow Request Problem)

Hozirgi serverimiz **bir oqimli (single-threaded)**. Agar kimdir `/sleep` manziliga kirsa va u 5 soniya vaqt olsa:

```rust,ignore
"GET /sleep HTTP/1.1" => {
    thread::sleep(Duration::from_secs(5));
    ("HTTP/1.1 200 OK", "hello.html")
}
```
Ushbu 5 soniya davomida boshqa hech bir foydalanuvchi serverga kira olmaydi — barcha so‘rovlar navbatda muzlab qoladi!

### Nega Har Bir So‘rovga Yangi Oqim Ochavermaymiz?
`thread::spawn(|| handle_connection(stream))` deb yozish mumkin edi. Ammo tajovuzkor shaxs bir vaqtning o‘zida 1 millionta so‘rov yuborsa, tizim xotirasi to‘lib, kompyuter butunlay qulab tushadi (Denial of Service - DoS).

**Yechim:** Cheklangan miqdordagi (masalan, 4 ta) doimiy oqimlardan iborat **Oqimlar Basseyni (Thread Pool)** qurish!

---

## 4. Oqimlar Basseynini Loyihalash (Thread Pool Architecture)

`ThreadPool` quyidagi arxitektura asosida ishlaydi:

```text
                +------------+
                | ThreadPool |
                +------------+
                      |
           (mpsc kanali orqali)
                      |
                      v
             [ Arc<Mutex<Rx>> ]
            /        |         \
           v         v          v
      +---------+ +---------+ +---------+
      | Worker 1| | Worker 2| | Worker 3| ...
      +---------+ +---------+ +---------+
```

1. Server so‘rovni qabul qiladi va uni `Job` (vazifa klojuri) ko‘rinishida kanalga uzatadi.
2. Basseyndagi `Worker` oqimlari umumiy qabul qiluvchi kanalni (`Arc<Mutex<Receiver>>`) kuzatib turadi.
3. Bo‘shagan oqim navbatdagi vazifani oladi va uni bajaradi.

### `src/lib.rs` Faylini Yozamiz:

```rust
use std::sync::{mpsc, Arc, Mutex};
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}

// Oqimlar bajarishi kerak bo'lgan har qanday vazifa tipi:
type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    /// Yangi ThreadPool yaratadi.
    ///
    /// # Panics
    /// Agar size <= 0 bo'lsa panic qiladi.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        // Qabul qiluvchini barcha workerlar o'rtasida xavfsiz ulashamiz:
        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);
        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    // println!("Worker {id} vazifani bajarishni boshladi...");
                    job();
                }
                Err(_) => {
                    // Kanal yopildi, oqim to'xtaydi
                    break;
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

---

## 5. Chiroyli To‘xtatish (Graceful Shutdown) va Tozalash: `Drop` Traiti

Server to‘xtatilganda barcha davom etayotgan so‘rovlar oxirigacha bajarilishi va oqimlar toza yopilishi lozim. Buning uchun `ThreadPool` ga `Drop` traitini ulaymiz:

```rust,ignore
impl Drop for ThreadPool {
    fn drop(&mut self) {
        // Avval kanal uzatuvchisini o'chiramiz, shunda barcha workerlar Err oladi va tsikldan chiqadi:
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!("Worker {} to‘xtatilmoqda...", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }

        println!("Barcha oqimlar muvaffaqiyatli to‘xtatildi.");
    }
}
```

---

## 6. Serverni Yakuniy Ishga Tushirish

Endi `src/main.rs` faylini yangilaymiz:

```rust,ignore
use std::fs;
use std::io::{prelude::*, BufReader};
use std::net::{TcpListener, TcpStream};
use std::thread;
use std::time::Duration;
use server::ThreadPool; // yoki loyihangiz nomi

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4); // 4 ta parallel oqim

    println!("⚡ Ko‘p oqimli server 7878-portda ishlamoqda!");

    // Test uchun dastlabki 2 ta so'rovdan so'ng to'xtashni tekshirish mumkin:
    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Server o‘z ishini yakunladi.");
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = match buf_reader.lines().next() {
        Some(Ok(line)) => line,
        _ => return,
    };

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(5));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    let contents = fs::read_to_string(filename).unwrap_or_else(|_| {
        String::from("<h1>Sahifa topilmadi</h1>")
    });
    let length = contents.len();

    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    let _ = stream.write_all(response.as_bytes());
}
```

Dasturni `cargo run` qilib ishga tushiring!
Bir vaqtning o‘zida ikkita brauzer tabini oching:
1. Biri orqali `http://127.0.0.1:7878/sleep` ga kiring (5 soniya kuting);
2. Darhol ikkinchi tabda `http://127.0.0.1:7878/` ni oching.

Ko‘rasizki, ikkinchi tab birinchisi tugashini kutmasdan, **lahza ichida darhol yuklanadi!** Chunki bizning ko‘p oqimli basseyndagi bo‘sh oqimlar uni mustaqil parallel ravishda qabul qilib xizmat ko‘rsatdi.

---

## 🏆 Katta Sayohat Yakuni

Siz Rust dasturlash tili bo‘yicha to‘liq o‘quv kursini muvaffaqiyatli yakunladingiz!
Siz oddiy `Salom, Dunyo!` dan boshlab, xotira modellari, qat’iy egalik va qarz olish tizimi, ilg‘or traitlar, aqlli ko‘rsatkichlar hamda to‘liq ko‘p oqimli xavfsiz tizimlar qurishgacha bo‘lgan murakkab va sharafli yo‘lni bosib o‘tdingiz.

Rust tili sizga nafaqat yuqori tezlik va xavfsizlikni, balki dasturlash arxitekturasiga mutlaqo yangicha, professional nigoh bilan qarashni taqdim etdi.

Kelajakdagi Rust loyihalaringizda omad tilaymiz! 🦀🚀
