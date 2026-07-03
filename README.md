# SQL-va-PostgreSQL-Asoslari-2
PostgreSQL-da ma'lumotlarni guruhlash (GROUP BY), guruhlangan natijalarni filtrlash (HAVING) va qatorlarni jamlash uchun quyidagi so'rovlar hamda qoidalardan foydalanishingiz mumkin.

1. Guruhlash va Guruhlarni Filtrlash
Yo'nalish bo'yicha guruhlash (GROUP BY + COUNT + AVG)
Har bir yo'nalishda nechta talaba o'qishi va ularning o'rtacha ballini hisoblash:

SQL
SELECT 
    yonalish, 
    COUNT(*) AS talabalar_soni, 
    ROUND(AVG(joriy_ball), 2) AS ortacha_ball
FROM talabalar
GROUP BY yonalish;
Guruhlarni filtrlash (HAVING)
WHERE operatori guruhlashdan oldin ishlaydi va agregat funksiyalar bilan ishlay olmaydi. Guruhlangan natijalarni (masalan, o'rtacha ballni) filtrlash uchun HAVING ishlatiladi:

SQL
-- O'rtacha bali 80 dan yuqori bo'lgan yo'nalishlarni chiqarish
SELECT 
    yonalish, 
    ROUND(AVG(joriy_ball), 2) AS ortacha_ball
FROM talabalar
GROUP BY yonalish
HAVING AVG(joriy_ball) > 80;
Ko'p ustunli guruhlash (Multi-column GROUP BY)
Ma'lumotlarni bir vaqtning o'zida ham yo'nalish, ham yosh bo'yicha guruhlash:

SQL
-- Bir xil yo'nalishda va bir xil yoshda bo'lgan talabalar sonini ko'rish
SELECT 
    yonalish, 
    yosh, 
    COUNT(*) AS talabalar_soni
FROM talabalar
GROUP BY yonalish, yosh
ORDER BY yonalish, yosh;
2. Murakkab Jamlash va Konstruksiyalar
STRING_AGG bilan matnlarni birlashtirish
Guruh ichidagi satrlarni bitta qatorga vergul (yoki boshqa belgi) bilan ajratib yozish uchun ishlatiladi:

SQL
-- Har bir yo'nalishda o'qiydigan talabalarning ismlarini bitta katakka jamlab ko'rsatish
SELECT 
    yonalish, 
    STRING_AGG(ism, ', ') AS talabalar_royxati
FROM talabalar
GROUP BY yonalish;
WHERE, GROUP BY va HAVING birgalikda
Ushbu uchta operator bitta so'rovda kelganda qat'iy ketma-ketlikda yoziladi:

SQL
-- 1. Yoshi 20 dan katta talabalarni saralash (WHERE)
-- 2. Ularni yo'nalish bo'yicha guruhlash (GROUP BY)
-- 3. Guruh ichida talabalar soni 1 tadan ko'p bo'lganlarini qoldirish (HAVING)
SELECT 
    yonalish, 
    COUNT(*) AS talabalar_soni,
    ROUND(AVG(joriy_ball), 2) AS ortacha_ball
FROM talabalar
WHERE yosh > 20
GROUP BY yonalish
HAVING COUNT(*) > 1;
3. Eng ko'p qilinadigan xato: GROUP BY guruhiga kirmagan oddiy ustunni SELECT ga yozish
Xato kod (Ishlamaydi!):
SQL
-- XATO: 'ism' ustuni GROUP BY ichida yo'q va hech qanday agregat funksiyaga olingan emas
SELECT yonalish, ism, AVG(joriy_ball) 
FROM talabalar
GROUP BY yonalish;
PostgreSQL beradigan xatolik xabari: ERROR: column "talabalar.ism" must appear in the GROUP BY clause or be used in an aggregate function

Sababi nimada?
Siz bazadan ma'lumotlarni yonalish bo'yicha guruhlashni so'rayapsiz. Masalan, "Dasturlash" yo'nalishida 4 ta talaba bor. AVG(joriy_ball) funksiyasi shu 4 ta talabaning balini qo'shib, bitta raqam (o'rtacha qiymat) qilib beradi.

Lekin siz bitta qatorga siqilgan "Dasturlash" guruhi yoniga ism ustunini ham chiqarmoqchisiz. SQL o'sha guruh ichidagi 4 ta har xil ismdan (Anvar, Diyora, Zilola, Kamola) aynan qaysi birini ko'rsatishni bilmaydi (chunki qatorlar soni mos kelmaydi).

To'g'ri yechish qoidasi:
Agar so'rovda GROUP BY ishlatilsa, SELECT qismida faqat quyidagilar bo'lishi mumkin:

GROUP BY guruhiga kiritilgan ustunlar (masalan: yonalish).

Agregat funksiyaga olingan ustunlar (masalan: AVG(joriy_ball) yoki STRING_AGG(ism, ', ')).
