# SQL-va-PostgreSQL-Asoslari-2
PostgreSQL-da berilgan vazifalarni bajarish uchun quyidagi SQL so'rovlaridan foydalanishingiz mumkin. Bularni pgAdmin (Query Tool) yoki psql konsoli orqali ishga tushirasiz.

1. Jadvalni yaratish (talabalar)
Kamida 5 ta ustundan iborat va birlamchi kalitga (PRIMARY KEY) ega jadval yaratamiz:

SQL
CREATE TABLE talabalar (
    id SERIAL PRIMARY KEY,
    ism VARCHAR(50) NOT NULL,
    familiya VARCHAR(50) NOT NULL,
    yosh INT,
    yonalish VARCHAR(100),
    joriy_ball NUMERIC(5, 2)
);
2. Jadvalga ma'lumot kiritish (10+ qator)
Jadvalni test ma'lumotlari bilan to'ldiramiz:

SQL
INSERT INTO talabalar (ism, familiya, yosh, yonalish, joriy_ball) VALUES
('Anvar', 'Karimov', 20, 'Dasturlash', 85.50),
('Malika', 'Aliyeva', 19, 'Ma''lumotlar tahlili', 92.00),
('Jasur', 'Tursunov', 21, 'Kiberxavfsizlik', 78.30),
('Diyora', 'Umarova', 22, 'Dasturlash', 95.00),
('Bekzod', 'Sultonov', 20, 'Sun''iy intellekt', 64.50),
('Zilola', 'Ahmedova', 19, 'Dasturlash', 88.00),
('Otabek', 'Madaminov', 23, 'Kiberxavfsizlik', 71.20),
('Nigora', 'Raximova', 21, 'Ma''lumotlar tahlili', 89.90),
('Sardor', 'Ismoilov', 20, 'Sun''iy intellekt', 55.00),
('Kamola', 'Hikmatova', 22, 'Dasturlash', 99.10);
3. SELECT So'rovlari (5 xil variant)
Variant 1: Barcha ma'lumotlarni chiqarish (*)
SQL
SELECT * FROM talabalar;
Variant 2: Ma'lum ustunlarni tanlab chiqarish
SQL
SELECT ism, yonalish, joriy_ball FROM talabalar;
Variant 3: Ustun nomiga taxallus (AS) berish
SQL
SELECT ism AS talaba_ismi, joriy_ball AS imtihon_bali FROM talabalar;
Variant 4: Matnlarni birlashtirish (||) operatori orqali to'liq ismni chiqarish
SQL
SELECT ism || ' ' || familiya AS toliq_ism, yonalish FROM talabalar;
Variant 5: Filtrlash va saralash bilan birga matn birlashtirish
SQL
SELECT 'Talaba: ' || ism || ' (' || yonalish || ')' AS malumot 
FROM talabalar 
WHERE joriy_ball >= 80;
4. Hisoblangan ustun bilan so'rov (ball * 1.1)
Talabalarning ballarini 10% ga oshirilgan holatda hisoblab chiqarish (masalan, bonus ball):

SQL
SELECT 
    ism, 
    familiya, 
    joriy_ball, 
    (joriy_ball * 1.1) AS yangi_bonus_ball 
FROM talabalar;
5. Hisobotni .sql fayl sifatida saqlash
pgAdmin orqali: Yuqoridagi barcha kodlarni pgAdmin Query Tool oynasiga yozing. Oynaning yuqori panelidagi Save (disketa belgisi) tugmasini bosing yoki Ctrl + S kombinatsiyasidan foydalanib, faylni talabalar_hisobot.sql nomi bilan saqlang.

Bog'lanish screenshotini qo'shish: pgAdmin-ga kirganingizda chap tomondagi Servers -> PostgreSQL -> Databases qismida o'z ma'lumotlar bazangizga bog'langan holatingizni ekran tasviriga (screenshot) olib, hisobotingizga ilova qiling.
