# Ilova C: O‘zbekcha-Inglizcha Rust Atamalar Lug‘ati (Glossary)

Ushbu lug‘at O‘zbekiston Rust hamjamiyati tomonidan Rust dasturlash tili bo‘yicha ilmiy va amaliy adabiyotlarda chalkashliklarning oldini olish hamda yagona standartni ta’minlash maqsadida tasdiqlangan.

---

| Inglizcha Termin | O‘zbekcha Standart | Ta’rifi va Misol |
| :--- | :--- | :--- |
| **Ownership** | *Egalik tizimi / Egalik huquqi* | Rust da xotira xavfsizligini ta’minlovchi resurs boshqaruvi qoidalari. |
| **Borrowing** | *Qarz olish* | O‘zgaruvchining egalik huquqini bermasdan, unga havola (`&` yoki `&mut`) orqali vaqtincha murojaat qilish. |
| **Reference** | *Havola* | Boshqa bir o‘zgaruvchining xotiradagi manziliga ko‘rsatuvchi xavfsiz mexanizm. |
| **Lifetime** | *Yashash muddati* | Kompilyator (`borrow checker`) tomonidan havola qancha vaqt davomida yaroqli ekanligini kafolatlovchi parametr (`'a`). |
| **Trait** | *Xislat (Trait)* | Muayyan tip amalga oshirishi (implement qilishi) kerak bo‘lgan metodlar to‘plami (interfeys). |
| **Pattern Matching** | *Andaza moslash* | Ma’lumotlar strukturasini shartlar asosida tekshirish va ajratib olish (`match`). |
| **Slice** | *Kesma* | To‘plamning (masalan, massiv yoki String) ma’lum bir qismiga uzluksiz havola (`&[T]`, `&str`). |
| **Smart Pointer** | *Aqlli ko‘rsatkich* | Oddiy xotira manzilidan tashqari qo‘shimcha funksionallik va avtomatik tozalashga ega tip (`Box`, `Rc`, `Arc`). |
| **Zero-cost Abstraction** | *Nol-xarajatli abstraksiya* | Yuqori darajadagi sintaktik qulaylik, lekin dastur ishlash tezligiga (runtime) hech qanday salbiy ta’sir ko‘rsatmaydi. |
| **Concurrency** | *Paralellik / Ko‘p oqimlilik* | Dasturda bir vaqtning o‘zida bir nechta vazifalarni xavfsiz bajarish. |
| **Data Race** | *Ma’lumotlar poygasi* | Ikki yoki undan ortiq oqim bir vaqtda xotiraning bitta nuqtasiga murojaat qilib, ulardan biri yozmoqchi bo‘lgandagi xatolik. Rust da bu kompilyatsiya vaqtida 100% oldi olinadi. |
| **Immutable** | *O‘zgarmas* | Qiymati yaratilgandan so‘ng o‘zgartirilishi mumkin bo‘lmagan o‘zgaruvchi (default Rust holati). |
| **Mutable** | *O‘zgaruvchan* | Qiymati keyinchalik o‘zgartirilishi mumkin bo‘lgan o‘zgaruvchi (`mut`). |
