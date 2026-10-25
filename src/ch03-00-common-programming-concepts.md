# 3-Bob: Dasturlashning Umumiy Tushunchalari

Har bir dasturlash tili o‘ziga xos asosiy qurilish bloklariga ega: o‘zgaruvchilar, ma’lumot turlari, funksiyalar va boshqaruv oqimi. Bu tushunchalar sizga boshqa tillardan tanish bo‘lishi mumkin, ammo Rust ularni o‘zgacha falsafa — **xotira xavfsizligi**, **aniqlik** va **nol qo‘shimcha xarajat (zero-cost abstraction)** tamoyillari asosida qayta talqin qiladi.

Rust da yozilgan har qanday kod ortida quyidagi savol turadi:
> *"Ushbu amal xotirada qanday aks etadi va kompilyatsiya paytida xatoliklarni qanday oldini olish mumkin?"*

Ushbu bobda biz quyidagi asosiy mavzularni chuqur va amaliy misollar bilan o‘rganamiz:

1. **[O‘zgaruvchilar va O‘zgaruvchanlik (mut)](ch03-01-variables-and-mutability.md):** Nega Rust da barcha narsa odatiy holatda o‘zgarmas (immutable) bo‘ladi? `mut`, `const` va `shadowing` ning tub farqlari nimada?
2. **[Ma’lumotlar Turlari (Data Types)](ch03-02-data-types.md):** Skalyar (butun sonlar, suzuvchi nuqtali sonlar, boolean, belgilar) va Kompozit (kortejlar va massivlar) turlar, ularning xotira (Stack) ko‘rinishi.
3. **[Funksiyalar Qanday Ishlaydi](ch03-03-how-functions-work.md):** Parametrlar, qiymat qaytarish, hamda Ko‘rsatmalar (Statements) va Iboralar (Expressions) o‘rtasidagi muhim tafovut.
4. **[Izohlar (Comments)](ch03-04-comments.md):** Oddiy kod izohlari va Rust ning hujjatlashtirish tizimi (`///`).
5. **[Boshqaruv Oqimi (Control Flow)](ch03-05-control-flow.md):** `if` iborasi, cheksiz `loop` tsikllari, `while` va `for` orqali to‘plamlar ustida xavfsiz aylanish.

Ushbu bobdagi tushunchalar keyingi boblarda o‘rganiladigan Rust ning eng mashhur xususiyati — **Egalik tizimi (Ownership)**ni tushunish uchun mustahkam poydevor vazifasini o‘taydi.
