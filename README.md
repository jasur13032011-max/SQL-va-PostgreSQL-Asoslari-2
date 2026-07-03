# SQL-va-PostgreSQL-Asoslari-2
PostgreSQL-da ma'lumotlarni saralash (tartiblash), takrorlanadigan qiymatlarni yo'qotish va sahifalash (pagination) amallarini bajarish uchun quyidagi so'rovlar va tushuntirishlardan foydalanishingiz mumkin.

1. Saralash va unikal qiymatlar bilan ishlash
ASC / DESC bilan tartiblash
Ma'lumotlarni o'sish (ASC) yoki kamayish (DESC) tartibida saralash:

SQL
-- Talabalarni ballari bo'yicha kamayish tartibida saralash (eng yuqori ball birinchi)
SELECT * FROM talabalar ORDER BY joriy_ball DESC;
Multi-column (Ko'p ustunli) tartiblash
Bir nechta ustun bo'yicha ketma-ket saralash. Agar birinchi ustundagi qiymatlar teng bo'lsa, tartiblash ikkinchi ustun bo'yicha davom etadi:

SQL
-- Avval yo'nalish bo'yicha alifbo tartibida, keyin bir xil yo'nalishdagilarni yoshi bo'yicha kamayish tartibida saralash
SELECT yonalish, yosh, ism, familiya 
FROM talabalar 
ORDER BY yonalish ASC, yosh DESC;
NULLS LAST bilan tartiblash
Agar jadvalda NULL (bo'sh) qiymatlar bo'lsa, o'sish tartibida saralaganda ular eng birinchi chiqib qoladi. Ularni majburiy ravishda ro'yxat oxiriga o'tkazish uchun NULLS LAST ishlatiladi:

SQL
-- Yoshi bo'yicha o'sish tartibida saralash, yoshi kiritilmaganlarni oxiriga qo'yish
SELECT ism, familiya, yosh 
FROM talabalar 
ORDER BY yosh ASC NULLS LAST;
DISTINCT bilan unikal qiymatlar
Jadvalda takrorlangan qatorlarni olib tashlab, faqat yagona (unikal) qiymatlarni chiqaradi:

SQL
-- Universitetda mavjud barcha yo'nalishlar ro'yxatini (takrorlarsiz) chiqarish
SELECT DISTINCT yonalish FROM talabalar;
COUNT(DISTINCT) ishlatish
Unikal qiymatlarning umumiy sonini hisoblash:

SQL
-- Bazada jami nechta turli xil yo'nalish borligini hisoblash
SELECT COUNT(DISTINCT yonalish) AS unikal_yonalishlar_soni FROM talabalar;
2. LIMIT va OFFSET bilan sahifalash (Pagination)
Katta hajmdagi ma'lumotlarni qismlarga bo'lib (sahifalab) ko'rsatish uchun LIMIT va OFFSET ishlatiladi. Faraz qilaylik, har bir sahifada 3 tadan talaba ko'rsatmoqchimiz.

3 ta sahifa uchun so'rovlar:
SQL
-- 1-Sahifa (Dastlabki 3 ta talaba)
SELECT id, ism, familiya FROM talabalar ORDER BY id ASC LIMIT 3 OFFSET 0;

-- 2-Sahifa (Keyingi 3 ta talaba)
SELECT id, ism, familiya FROM talabalar ORDER BY id ASC LIMIT 3 OFFSET 3;

-- 3-Sahifa (Undan keyingi 3 ta talaba)
SELECT id, ism, familiya FROM talabalar ORDER BY id ASC LIMIT 3 OFFSET 6;
3. Sahifalash formulasi tushuntirishi
Dasturlashda (backend yoki veb-saytlarda) dinamik sahifalashni amalga oshirish uchun har safar OFFSET qiymati matematik formula orqali hisoblanadi.

Asosiy qoida:

LIMIT — Bir sahifada nechta qator (ma'lumot) ko'rinishi lozimligi.

OFFSET — So'rov boshidan nechta qatorni tashlab yuborish (sakrab o'tish) kerakligi.

Formula:
OFFSET=(Sahifa raqami−1)×Sahifadagi elementlar soni (LIMIT)
Misol ustida tahlil:
Agar siz har bir sahifada 5 tadan ma'lumot ko'rsatmoqchi bo'lsangiz (LIMIT = 5):

1-sahifa uchun: (1−1)×5=0→OFFSET 0 (Hech narsani tashlamaydi)

2-sahifa uchun: (2−1)×5=5→OFFSET 5 (Dastlabki 5 tasini tashlab, 6-sidan boshlaydi)

3-sahifa uchun: (3−1)×5=10→OFFSET 10 (Dastlabki 10 tasini tashlab, 11-sidan boshlaydi)
