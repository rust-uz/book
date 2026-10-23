# O‘rnatish (Installation)

Rust bilan ishlashning birinchi qadami — uni kompyuteringizga o‘rnatishdir. Biz Rust dasturchilari jamoasi tomonidan taqdim etilgan rasmiy o‘rnatuvchi — **`rustup`** vositasidan foydalanamiz.

## Linux va macOS Tizimlarida O‘rnatish

Agar siz Linux (masalan, Arch Linux, Ubuntu, Debian) yoki macOS tizimidan foydalanayotgan bo‘lsangiz, terminalni oching va quyidagi buyruqni kiriting:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Ushbu skript avtomatik ravishda operatsion tizimingizni aniqlaydi, so‘nggi barqaror (stable) Rust kompilyatori (`rustc`), standart kutubxona va `cargo` menejerini yuklab oladi.

Ekrandagi ko‘rsatmalarga amal qilib, **1** (default installation) variantini tanlang. O‘rnatish tugagach, o‘zgarishlar kuchga kirishi uchun terminalingizni qayta ishga tushiring yoki quyidagini bajaring:

```bash
source "$HOME/.cargo/env"
```

### C Kompilyatorini O‘rnatish (Linker talabi)
Rust dasturlarini yig‘ish (link qilish) uchun tizimda C kompilyatori bo‘lishi lozim:

* **Arch Linux:**
  ```bash
  sudo pacman -S base-devel
  ```
* **Ubuntu / Debian:**
  ```bash
  sudo apt update && sudo apt install build-essential
  ```
* **macOS:**
  ```bash
  xcode-select --install
  ```

---

## Windows Tizimida O‘rnatish

Windows foydalanuvchilari [https://www.rust-lang.org/tools/install](https://www.rust-lang.org/tools/install) sahifasiga kirib, **`rustup-init.exe`** dasturini yuklab olishlari va ishga tushirishlari lozim.

O‘rnatish jarayonida **Visual Studio C++ Build Tools** talab qilinishi mumkin.

---

## O‘rnatishni Tekshirish

Rust to‘g‘ri o‘rnatilganligini bilish uchun terminalda quyidagi buyruqni kiriting:

```bash
rustc --version
```

Agar siz quyidagiga o‘xshash javobni ko‘rsangiz, demak hammasi muvaffaqiyatli yakunlandi:

```text
rustc 1.85.0 (4d91de4e4 2026-02-17)
```

Rust ni doimiy so‘nggi versiyaga yangilab turish uchun esa oddiygina quyidagi buyruq kifoya:

```bash
rustup update
```
