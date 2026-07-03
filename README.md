# SQL-va-PostgreSQL-Asoslari-2
PostgreSQL-da so'ralgan hisobotlar va vazifalarni bajarish uchun tayyor SQL so'rovlari quyida keltirilgan. Har bir so'rovning vazifasi tepasidagi sharhlar (-- comment) orqali tushuntirilgan.

SQL
-- 1. Top-3 a'lochi: Eng yuqori ball to'plagan 3 ta talabaning ismi va bali
SELECT ism, joriy_ball 
FROM talabalar 
ORDER BY joriy_ball DESC 
LIMIT 3;


-- 2. Filterlangan + tartiblangan ro'yxat: Dasturlash yo'nalishidagi balli 80 dan yuqori talabalarni yoshi bo'yicha o'sish tartibida chiqarish
SELECT ism, familiya, yosh, yonalish, joriy_ball 
FROM talabalar 
WHERE yonalish = 'Dasturlash' AND joriy_ball > 80 
ORDER BY yosh ASC;


-- 3. Takrorsiz sinflar (yo'nalishlar) ro'yxati: Universitetdagi barcha unikal yo'nalishlarni alifbo tartibida chiqarish
SELECT DISTINCT yonalish 
FROM talabalar 
ORDER BY yonalish ASC;


-- 4. Yo'qotgan kunlar (kiritilmagan ma'lumotlar) bo'yicha hisobot: Yoshi yoki yo'nalishi kiritilmagan (NULL bo'lgan) talabalar ro'yxati
SELECT id, ism, familiya, yosh, yonalish 
FROM talabalar 
WHERE yosh IS NULL OR yonalish IS NULL;


-- 5. Eng yosh va eng katta yoshli talabalar: UNION ALL orqali eng kichik va eng katta yoshli talabalarni bitta jadvalda birlashtirish
(SELECT 'Eng yosh talaba' AS status, ism, familiya, yosh FROM talabalar WHERE yosh IS NOT NULL ORDER BY yosh ASC LIMIT 1)
UNION ALL
(SELECT 'Eng katta yoshli talaba' AS status, ism, familiya, yosh FROM talabalar WHERE yosh IS NOT NULL ORDER BY yosh DESC LIMIT 1);


-- 6. Sahifa 2 (LIMIT/OFFSET): Har bir sahifada 3 tadan talaba chiqarilganda, 2-sahifadagi ma'lumotlarni ko'rish (OFFSET = (2-1)*3 = 3)
SELECT id, ism, familiya 
FROM talabalar 
ORDER BY id ASC 
LIMIT 3 OFFSET 3;


-- 7. Bonus: Dasturlash yo'nalishidagi (shartli ravishda eng yuqori guruh/sinf) top-5 a'lochi talabalar ro'yxati
SELECT ism, familiya, yonalish, joriy_ball 
FROM talabalar 
WHERE yonalish = 'Dasturlash' 
ORDER BY joriy_ball DESC 
LIMIT 5;
Qisqa tushuntirish:
UNION ALL operatori ikkita alohida SELECT so'rovi natijasini bitta jadvalga ketma-ket birlashtiradi. Birinchi qism eng kichik yoshni, ikkinchi qism esa eng katta yoshni topib, natijani umumlashtiradi.

LIMIT 3 OFFSET 3 so'rovi dastlabki 3 ta qatorni tashlab yuborib (OFFSET), keyingi 3 ta qatorni (LIMIT) ekranga chiqaradi, bu esa aniq 2-sahifani shakllantiradi.
