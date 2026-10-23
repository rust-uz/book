# 4-Bob: Egalik Tizimini Tushunish (Understanding Ownership)

**Egalik (Ownership)** — bu Rust tilining eng o‘ziga xos va inqilobiy xususiyati bo‘lib, u dasturchiga Garbage Collector (axlat yig‘uvchi)siz xotira xavfsizligini ta’minlash imkonini beradi.

## Xotira Boshqaruvi Tarixi

Dasturlash tillarida xotira (RAM) bilan ishlashning asosan ikkita usuli bor edi:
1. **Garbage Collector (GC):** Go, Java, Python, JavaScript kabi tillarda dastur orqa fonda xotirani doimiy kuzatib boradi va kerak bo‘lmagan joylarni tozalaydi. Kamchiligi: kutilmagan to‘xtashlar (stop-the-world pauses) va katta resurs sarfi.
2. **Qo‘lda boshqarish (Manual Allocation):** C va C++ kabi tillarda dasturchi `malloc`/`free` orqali xotirani o‘zi ajratadi va o‘zi bo‘shatadi. Kamchiligi: insoniy xatolar (Dangling Pointers, Double Free, Buffer Overflows, Use-After-Free) natijasida kiberxavfsizlik teshiklarining 70% dan ortig‘i aynan shu sababli kelib chiqadi.

## Rust Yechimi: Egalik Qoidalari (Ownership Rules)

Rust uchinchi va mutlaqo yangi yo‘lni tanladi: **Xotira kompilyatsiya vaqtida qat’iy qoidalar to‘plami asosida boshqariladi!**

Kompilyator barcha qoidalarni dastur yig‘ilayotganda tekshiradi. Agar qoidalardan biri buzilsa, dastur hatto kompilyatsiya ham bo‘lmaydi!

### Egalikning 3 ta Asosiy Qoidasi:
1. **Rust da har bir qiymatning o‘z *Egasi (Owner)* bo‘ladi.**
2. **Bir vaqtning o‘zida faqat bitta Ega bo‘lishi mumkin.**
3. **Qachonki Ega o‘z ko‘rinish sohasidan (scope) chiqib ketsa, qiymat avtomatik ravishda xotiradan o‘chiriladi (`drop` qilinadi).**

---

## O‘zgaruvchilar Doirasi (Scope)

Ko‘rinish sohasi (scope) — bu dastur ichida o‘zgaruvchi amal qiladigan oraliqdir.

```rust
{                      // s bu yerda hali mavjud emas
    let s = "salom";   // s shu qatordan boshlab mavjud va faol

    // s bilan amallar bajaramiz
}                      // Bu blok tugadi! s ning umri tugadi va u xotiradan tozalandi
```

Keyingi darslarda biz Heap va Stack xotira turlari, `Move` semantikasi hamda `Borrowing` (Qarz olish) qanday ishlashini ko‘rib chiqamiz.
