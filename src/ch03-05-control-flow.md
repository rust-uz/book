# Boshqaruv Oqimi (Control Flow)

Dasturning shartlarga qarab qaysi yo‘nalishda davom etishini yoki biror amalni necha marta takrorlashini belgilash **boshqaruv oqimi** deb ataladi.

Rust da boshqaruv oqimining asosiy mexanizmlari quyidagilardir:
1. `if` shartli iboralari;
2. Tsikllar: `loop`, `while` va `for`.

---

## 1. `if` Iboralari (If Expressions)

Rust da shart har doim **aniq `bool`** (mantiqiy) qiymat bo‘lishi shart. JavaScript yoki Python kabi tillardan farqli ravishda, Rust da `0` yoki bo‘sh matn avtomatik ravishda `false` ga aylanmaydi (hech qanday "truthy/falsy" tushunchasi yo‘q).

```rust
fn main() {
    let son = 7;

    if son < 5 {
        println!("Shart to‘g‘ri bajarildi!");
    } else {
        println!("Shart bajarilmadi!");
    }
}
```

Bir nechta shartlarni `else if` orqali tekshirish mumkin:

```rust
let son = 6;

if son % 4 == 0 {
    println!("Son 4 ga bo‘linadi");
} else if son % 3 == 0 {
    println!("Son 3 ga bo‘linadi");
} else {
    println!("Son 4 ga ham, 3 ga ham bo‘linmaydi");
}
```

### `let` Bilan `if` ifodasini Ishlatish
Rust da `if` — bu ko‘rsatma emas, balki **ibora (expression)** bo‘lgani sababli, uning natijasini to‘g‘ridan-to‘g‘ri o‘zgaruvchiga biriktirish mumkin:

```rust
let shart = true;
let son = if shart { 5 } else { 6 };

println!("son qiymati: {son}"); // 5
```

> **Eslatma:** Har ikkala shoxobchadan (`if` va `else`) qaytadigan qiymatlarning tipi bir xil bo‘lishi shart! Masalan, biridan son, ikkinchisidan matn qaytarish kompilyatsiya xatosiga sabab bo‘ladi.

---

## 2. Tsikllar bilan Takrorlash (Loops)

Rust takroriy amallar uchun uch turdagi tsikllarni taqdim etadi: `loop`, `while` va `for`.

### A. Cheksiz Tsikl: `loop`
`loop` kalit so‘zi dasturni to to‘xtatish buyrug‘i (`break`) berilmaguncha cheksiz takrorlaydi:

```rust
fn main() {
    let mut sanagich = 0;

    loop {
        sanagich += 1;
        println!("Sanagich: {sanagich}");

        if sanagich == 3 {
            break; // Tsikldan chiqish
        }
    }
}
```

#### `loop` dan Qiymat Qaytarish
`break` so‘zidan keyin ifoda yozish orqali tsikl natijasini o‘zgaruvchiga qaytarish mumkin:

```rust
let mut sanagich = 0;

let natija = loop {
    sanagich += 1;

    if sanagich == 10 {
        break sanagich * 2; // natija 20 bo‘ladi
    }
};

println!("Natija: {natija}");
```

#### Tsikl Yorliqlari (Loop Labels)
Agar ichma-ich joylashgan tsikllar bo‘lsa, `break` yoki `continue` eng ichki tsiklga ta’sir qiladi. Agar tashqi tsikldan chiqmoqchi bo‘lsangiz, tsikllarga bittalik tirnoq bilan yorliq (label) berishingiz mumkin:

```rust
fn main() {
    let mut sanagich = 0;

    'tashqi_tsikl: loop {
        println!("Tashqi tsikl: {sanagich}");
        let mut qolgan = 10;

        loop {
            println!("  Ichki tsikl: {qolgan}");
            if qolgan == 9 {
                break; // faqat ichki tsiklni to‘xtatadi
            }
            if sanagich == 2 {
                break 'tashqi_tsikl; // Tashqi tsiklni butunlay to‘xtatadi!
            }
            qolgan -= 1;
        }

        sanagich += 1;
    }
}
```

### B. Shartli Tsikl: `while`
Shart `true` bo‘lib turguncha kod blokini takrorlash uchun `while` ishlatiladi:

```rust
let mut son = 3;

while son != 0 {
    println!("{son}!");
    son -= 1;
}

println!("START! 🚀");
```

### C. To‘plamlar Bo‘ylab Yurish: `for`
Rust da eng ko‘p ishlatiladigan va eng xavfsiz tsikl — bu `for` dir.

Massiv yoki kolleksiyalar elementlarini aylanib chiqishda `while` o‘rniga `for` ishlatish xavfsizlikni (indeks chegarasidan chiqib ketish xavfini yo‘qotadi) va yuqori tezlikni kafolatlaydi:

```rust
let massiv = [10, 20, 30, 40, 50];

for element in massiv {
    println!("Qiymat: {element}");
}
```

Raqamlar oralig‘i bo‘yicha yurish uchun `Range` dan foydalaniladi:

```rust
// 1 dan 4 gacha (4 kirmaydi), teskari tartibda:
for son in (1..4).rev() {
    println!("{son}!");
}
println!("START! 🚀");
```
