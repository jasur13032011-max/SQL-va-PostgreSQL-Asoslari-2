# SQL-va-PostgreSQL-Asoslari-2
PostgreSQL-da murakkab statistik hisobotlar, guruhlash va jadvallarni bog'lash (JOIN) yordamida "Dashboard" (tahliliy panel) uchun ma'lumotlar tayyorlash so'rovlari quyida keltirilgan.

Har bir so'rov tepasida uning vazifasi -- izoh ko'rinishida yozilgan.

SQL
-- 1. Sinf (Yo'nalish) reytingi: Har bir yo'nalish bo'yicha jami talabalar soni, o'rtacha, maksimal va minimal ballar
SELECT 
    yonalish,
    COUNT(*) AS talabalar_soni,
    ROUND(AVG(joriy_ball), 2) AS ortacha_ball,
    MAX(joriy_ball) AS eng_yuqori_ball,
    MIN(joriy_ball) AS eng_past_ball
FROM talabalar
GROUP BY yonalish
ORDER BY ortacha_ball DESC;


-- 2. Fan bo'yicha statistika (HAVING bilan): Faqat kamida 3 ta talaba baho olgan va o'rtacha bahosi 4 dan yuqori bo'lgan fanlarni chiqarish
SELECT 
    f.fan_nomi,
    COUNT(b.id) AS baholangan_talabalar,
    ROUND(AVG(b.baho), 2) AS ortacha_baho
FROM fanlar f
INNER JOIN baholar b ON f.id = b.fan_id
GROUP BY f.id, f.fan_nomi
HAVING COUNT(b.id) >= 3 AND AVG(b.baho) > 4;


-- 3. Har fan bo'yicha eng yaxshi talaba: Har bir fandan eng yuqori baho olgan talabani aniqlash (DISTINCT ON bilan optimallashtirilgan)
SELECT DISTINCT ON (f.id) 
    f.fan_nomi,
    t.ism,
    t.familiya,
    b.baho AS eng_yuqori_baho
FROM fanlar f
INNER JOIN baholar b ON f.id = b.fan_id
INNER JOIN talabalar t ON b.talaba_id = t.id
ORDER BY f.id, b.baho DESC;


-- 4. Bahosiz talabalar (LEFT JOIN + IS NULL): Biron marta ham baho olmagan (yoki imtihonga kirmagan) talabalar ro'yxati
SELECT 
    t.id, 
    t.ism, 
    t.familiya,
    t.yonalish
FROM talabalar t
LEFT JOIN baholar b ON t.id = b.talaba_id
WHERE b.id IS NULL;


-- 5. Universal a'lochi: Barcha topshirgan fanlaridan olgan eng past bahosi ham 85 dan yuqori bo'lgan (ya'ni umuman past baho olmagan) talabalar
SELECT 
    t.ism,
    t.familiya,
    MIN(b.baho) AS eng_past_bahosi,
    ROUND(AVG(b.baho), 2) AS ortacha_bahosi
FROM talabalar t
INNER JOIN baholar b ON t.id = b.talaba_id
GROUP BY t.id, t.ism, t.familiya
HAVING MIN(b.baho) >= 85;


-- 6. Hamma talabalar uchun ortacha (LEFT JOIN): Bahosi bor yoki yo'qligidan qat'iy nazar barcha talabalar va ularning o'rtacha bahosi (bahosizlarga 0 yoki NULL chiqadi)
SELECT 
    t.ism,
    t.familiya,
    COALESCE(ROUND(AVG(b.baho), 2), 0) AS ortacha_baho
FROM talabalar t
LEFT JOIN baholar b ON t.id = b.talaba_id
GROUP BY t.id, t.ism, t.familiya
ORDER BY ortacha_baho DESC;


-- 7. Bonus — UNION ALL bilan dashboard metrikalari: Administratorlar paneli (Dashboard) uchun asosiy raqamli ko'rsatkichlarni bitta jadvalga yig'ish
SELECT 'Jami talabalar soni' AS metrika, COUNT(*)::NUMERIC AS qiymat FROM talabalar
UNION ALL
SELECT 'Mavjud fanlar soni', COUNT(*)::NUMERIC FROM fanlar
UNION ALL
SELECT 'Umumiy o''rtacha ball (Universitet bo''yicha)', ROUND(AVG(joriy_ball), 2) FROM talabalar
UNION ALL
SELECT 'Eng yuqori talaba bali', MAX(joriy_ball) FROM talabalar;
Tahlil va Maslahat:
COALESCE(..., 0) funksiyasi 6-so'rovda bahosi bo'lmagan talabalarning o'rtacha bali NULL bo'lib qolmasdan, chiroyli ko'rinishda 0 bo'lib chiqishini ta'minlaydi.

DISTINCT ON (f.id) operatori PostgreSQL-ning eng kuchli imkoniyatlaridan biri bo'lib, har bir guruhning (fanning) faqat birinchi qatorini (biz ORDER BY b.baho DESC qilganimiz uchun eng yuqori baholi qatorni) ajratib beradi.

UNION ALL yordamida yaratilgan 7-so'rov (Dashboard) turli xil jadvallardan yig'ilgan yakuniy hisobotlarni backend dasturchilar yoki biznes tahlilchilar uchun bitta umumiy jadval ko'rinishida taqdim etadi.
