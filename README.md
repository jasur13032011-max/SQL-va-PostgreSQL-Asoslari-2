# SQL-va-PostgreSQL-Asoslari-2
1. Agregat va Statistik Funksiyalar
COUNT(*) va COUNT(DISTINCT) farqi
COUNT(*) — jadvaldagi umumiy qatorlar sonini (shu jumladan NULL va takrorlanadigan qatorlarni) hisoblaydi.

COUNT(DISTINCT ustun) — faqat unikal (takrorlanmaydigan) va NULL bo'lmagan qiymatlar sonini hisoblaydi.

SQL
-- Talabalarning umumiy soni va yo'nalishlarning takrorsiz soni
SELECT 
    COUNT(*) AS jami_talabalar,
    COUNT(DISTINCT yonalish) AS unikal_yonalishlar
FROM talabalar;
To'liq statistika va ROUND bilan yumaloqlash
SUM, AVG, MIN, MAX funksiyalari orqali umumiy statistikani chiqarish va ROUND funksiyasi yordamida kasr qismni (masalan, verguldan keyin 2 ta raqamgacha) yumaloqlash:

SQL
SELECT 
    SUM(joriy_ball) AS umumiy_ball,
    ROUND(AVG(joriy_ball), 2) AS ortacha_ball,
    MIN(joriy_ball) AS eng_past_ball,
    MAX(joriy_ball) AS eng_yuqori_ball
FROM talabalar;
2. Matn (String) va Matematik Funksiyalar
Matn funksiyalari (UPPER, LENGTH, ||)
SQL
-- Ism-familiyani katta harflarda birlashtirish va yo'nalish matni uzunligini o'lchash
SELECT 
    UPPER(ism || ' ' || familiya) AS toliq_ism_katta,
    yonalish,
    LENGTH(yonalish) AS yonalish_harflar_soni
FROM talabalar;
Matematik funksiyalar (CEIL, FLOOR, MOD)
CEIL(x) — eng yaqin katta butun songa qarab yaxlitlaydi.

FLOOR(x) — eng yaqin kichik butun songa qarab yaxlitlaydi.

MOD(x, y) — bo'lishdan chiqqan qoldiqni hisoblaydi.

SQL
-- Ballarni yuqoriga/pastga yaxlitlash va yoshni 2 ga bo'lgandagi qoldiq (toq/juftlik)
SELECT 
    joriy_ball,
    CEIL(joriy_ball) AS yuqoriga_yaxlit,
    FLOOR(joriy_ball) AS pastga_yaxlit,
    yosh,
    MOD(yosh, 2) AS yosh_qoldig_i
FROM talabalar;
3. Sana va Vaqt Funksiyalari (NOW, CURRENT_DATE, EXTRACT)
PostgreSQL-da joriy vaqtni aniqlash va sananing ma'lum qismlarini ajratib olish:

SQL
SELECT 
    NOW() AS joriy_vaqt_va_sana,          -- To'liq vaqt (miqyosi bilan)
    CURRENT_DATE AS bugungi_sana,         -- Faqat sana (yil-oy-kun)
    EXTRACT(YEAR FROM NOW()) AS joriy_yil, -- Faqat yilni ajratib olish
    EXTRACT(MONTH FROM NOW()) AS joriy_oy  -- Faqat oyni ajratib olish
FROM talabalar 
LIMIT 1; -- Faqat 1 marta chiqishi uchun
4. Eng ko'p qilinadigan xato: WHERE ichida Agregat funksiya ishlatish
Xato kod (Ishlamaydi!):
SQL
-- XATO: O'rtacha balldan yuqori bo'lgan talabalarni topishga urinish
SELECT ism, joriy_ball 
FROM talabalar 
WHERE joriy_ball > AVG(joriy_ball); 
PostgreSQL beradigan xatolik xabari: ERROR: aggregate functions are not allowed in WHERE

Sababi nimada?
SQL so'rovlarining bajarilish tartibi (Logical Query Processing) mavjud. WHERE operatori jadvaldagi qatorlarni hali agregat funksiyalar hisoblanmasdan oldin (guruhlash va umumlashtirishdan avval) birma-bir elakdan o'tkazadi.

AVG(joriy_ball) ni hisoblash uchun esa bazaga barcha qatorlarning qiymati kerak. WHERE ishlayotgan vaqtda hali umumiy o'rtacha ball qanchaligi ma'lum bo'lmagani uchun, SQL bu so'rovni bajara olmaydi.

To'g'ri yechim (Subquery yoki HAVING):
Buning uchun Subquery (ichma-ich so'rov) ishlatish kerak:

SQL
-- TO'G'RI: Avval o'rtacha ball hisoblanadi, keyin WHERE orqali solishtiriladi
SELECT ism, joriy_ball 
FROM talabalar 
WHERE joriy_ball > (SELECT AVG(joriy_ball) FROM talabalar);
