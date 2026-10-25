# 12-Bob: I/O Loyiha: Komandalar Qatori Dasturi (minigrep)

Ushbu bobda biz shu paytgacha o‘rgangan barcha asosiy tushunchalarni (o‘zgaruvchilar, funksiyalar, xatolarni boshqarish, modullar, testlar) birlashtirgan holda to‘liq amaliy loyiha yaratamiz.

Biz klassik GNU vositasi bo‘lgan `grep` (Global Regular Expression Print) ning kichik analogi — **`minigrep`** dasturini quramiz. U berilgan matnli fayl ichidan qidirilayotgan satrni topib, natijalarni terminalga chiqaradi.

---

## 1. Komandalar Qatori Argumentlarini Qabul Qilish

Dastlab `cargo new minigrep` orqali yangi loyiha ochamiz.
Terminaldan kiritilgan argumentlarni o‘qish uchun `std::env::args()` funksiyasidan foydalanamiz:

```rust
// src/main.rs
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    if args.len() < 3 {
        eprintln!("Foydalanish: cargo run -- <qidiruv_sozi> <fayl_yoli>");
        std::process::exit(1);
    }

    let query = &args[1];
    let file_path = &args[2];

    println!("Qidirilmoqda: '{query}'");
    println!("Fayl manzili: '{file_path}'");
}
```

---

## 2. Faylni O‘qish

Fayl ichidagi barcha matnni bitta `String` ga o‘qib olish uchun `std::fs::read_to_string` funksiyasi xizmat qiladi:

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let file_path = &args[2];

    let contents = fs::read_to_string(file_path)
        .expect("Faylni o‘qishda kutilmagan xatolik yuz berdi!");

    println!("Fayl matni:\n{contents}");
}
```

---

## 3. Kodni Modullarga Ajratish (Refactoring)

`main.rs` fayli faqat dasturni ishga tushirish uchun xizmat qilishi, barcha asosiy biznes-mantiq esa `src/lib.rs` kutubxonasida jamlanishi Rust dasturlashining eng yaxshi amaliyotidir.

Keling, konfiguratsiyani saqlovchi `Config` strukturasi va xatolarni chiroyli qaytaruvchi `Config::build` metodini yaratamiz.

### `src/lib.rs` Fayli:
```rust
use std::error::Error;
use std::fs;

pub struct Config {
    pub query: String,
    pub file_path: String,
    pub ignore_case: bool,
}

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("Argumentlar yetarli emas! Talab: <qidiruv_sozi> <fayl_yoli>");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();
        let ignore_case = std::env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}

pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    let results = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };

    for line in results {
        println!("{line}");
    }

    Ok(())
}
```

---

## 4. TDD (Test-Driven Development) Bilan Qidiruv Funksiyasini Yozish

Rust da yangi funksionallikni test orqali boshlash (TDD) juda qulay. Dastlab sinov testini yozamiz:

```rust
// src/lib.rs ning eng pastki qismiga qo'shamiz:

pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}

pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn bitta_natija_topilishi_kerak() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }

    #[test]
    fn harf_registriga_bogliq_bolmagan_qidiruv() {
        let query = "rUsT";
        let contents = "\
Rust:
safe, fast, productive.
Trust me.";

        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}
```

---

## 5. Standart Xatolar Oqimi: `eprintln!`

`src/main.rs` faylimizni toza va nafis qilib bog‘laymiz:

```rust
// src/main.rs
use std::env;
use std::process;
use minigrep::Config;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Dasturni ishga tushirishda xatolik: {err}");
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Dastur xatosi: {e}");
        process::exit(1);
    }
}
```

### Nega `println!` emas, `eprintln!`?
Ko‘pgina terminal dasturlari standart chiqish (`stdout`) va xatolar chiqishi (`stderr`) oqimlariga ega.
Agar siz oddiy `println!` ishlatsangiz va foydalanuvchi natijani faylga yo‘naltirsa (`cargo run -- soz fayl.txt > natija.txt`), hatto xato xabarlari ham `natija.txt` fayliga yozilib qoladi.
**`eprintln!`** esa xatolarni alohida standart xato oqimiga (`stderr`) yuboradi, shunda ekranda xato aniq ko‘rinadi va fayl tozalanadi.

### Dasturni Sinab Ko‘rish:
```bash
cargo run -- to poem.txt
```
Katta-kichik harflarni e’tiborsiz qoldirish uchun:
```bash
IGNORE_CASE=1 cargo run -- to poem.txt
```

Tabriklaymiz! Siz Rust tilida to‘liq ishlaydigan, xavfsiz va xatoliklar chiroyli qayta ishlangan haqiqiy CLI (Command Line Interface) dasturini yaratdingiz!
