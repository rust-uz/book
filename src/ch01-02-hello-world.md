# Salom, Dunyo! (Hello, World!)

Endi birinchi dasturingizni yozish vaqti keldi. Dasturlash tillarini o‘rganishda terminalga *"Salom, Dunyo!"* matnini chiqarish an’anaga aylangan.

## Loyiha Jildini Yaratish

Terminalda yangi jild oching va uning ichiga kiring:

```bash
mkdir ~/projects/salom_dunyo
cd ~/projects/salom_dunyo
```

## Dastur Kodini Yozish

O‘zingiz yoqtirgan matn muharriri (VS Code, Neovim, Helix yoki Nano) yordamida `main.rs` nomli fayl yarating. Rust manba kodlari har doim `.rs` kengaytmasi bilan tugaydi.

`main.rs` fayliga quyidagi kodni yozing:

```rust
fn main() {
    println!("Salom, Dunyo!");
}
```

Faylni saqlang va terminalga qayting.

---

## Dasturni Kompilyatsiya Qilish va Ishga Tushirish

Rust — bu oldindan kompilyatsiya qilinadigan (ahead-of-time compiled) til. Bu degani siz dasturingizni kompilyatsiya qilib binar (executable) faylga aylantirasiz va ushbu faylni Rust o‘rnatilmagan boshqa kompyuterlarda ham bemalol ishga tushirishingiz mumkin!

Faylni kompilyatsiya qilish uchun:

```bash
rustc main.rs
```

Agar barcha qoidalar to‘g‘ri bajarilgan bo‘lsa, `rustc` jim ishlaydi va jildingizda yangi binar fayl paydo bo‘ladi (Linux/macOS da `main`, Windows da `main.exe`).

Endi dasturni ishga tushiring:

```bash
./main
```

Terminalingizda quyidagi yozuv chiqadi:

```text
Salom, Dunyo!
```

---

## Kod Anatomiyasi: Qatorlar Tahlili

Keling, yozgan kodimizni batafsil ko‘rib chiqaylik:

1. **`fn main() {`**: 
   * `fn` kalit so‘zi Rust da yangi funksiya e’lon qilishni bildiradi.
   * `main` funksiyasi — har qanday Rust dasturining kirish nuqtasi (entrypoint) hisoblanadi. Dastur doimo aynan shu yerdan boshlanadi.
   * Funksiya tanasi jingalak qavslar `{ ... }` ichida yoziladi.
2. **`println!("Salom, Dunyo!");`**:
   * Ushbu qator ekranga matn chiqaradi.
   * **Diqqat qiling:** `println!` oxirida undov belgisi (`!`) bor. Bu shunchaki oddiy funksiya emas, balki **Rust Makrosi (Macro)** ekanligini anglatadi. Rust da makroslar va oddiy funksiyalar boshqacha ishlaydi (buni 19-bobda batafsil ko‘ramiz).
   * Har bir ifoda oxiriga nuqta-vergul (`;`) qo‘yiladi.
