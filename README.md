# SQL-va-PostgreSQL-Asoslari-2
PostgreSQL-da ma'lumotlarni filtrlash va shartlar bilan ishlash uchun quyidagi so'rovlar va tushuntirishlardan foydalanishingiz mumkin. Barcha misollar yuqorida yaratilgan talabalar jadvali ma'lumotlariga asoslangan.

1. 5 ta turli WHERE shartlari
Quyida jadvaldan ma'lumotlarni turli operatorlar yordamida filtrlash so'rovlari keltirilgan:

SQL
-- 1. Tenglik operatori (=) - Dasturlash yo'nalishidagi talabalar
SELECT * FROM talabalar WHERE yonalish = 'Dasturlash';

-- 2. Katta operatori (>) - Balli 80 dan yuqori bo'lgan talabalar
SELECT * FROM talabalar WHERE joriy_ball > 80;

-- 3. BETWEEN operatori - Yoshi 20 va 22 oralig'ida bo'lgan talabalar (chegaralar kiradi)
SELECT * FROM talabalar WHERE yosh BETWEEN 20 AND 22;

-- 4. IN operatori - Ko'rsatilgan yo'nalishlardan birida o'qiydigan talabalar
SELECT * FROM talabalar WHERE yonalish IN ('Kiberxavfsizlik', 'Sun''iy intellekt');

-- 5. LIKE operatori - Familiyasi 'K' harfi bilan boshlanadigan talabalar
SELECT * FROM talabalar WHERE familiya LIKE 'K%';

-- 6. IS NULL operatori - Yo'nalishi kiritilmagan talabalarni topish
SELECT * FROM talabalar WHERE yonalish IS NULL;
2. AND / OR / NOT bilan murakkab shart
Mantiqiy operatorlar va qavslar yordamida ustuvorlikni belgilagan holda murakkab shart yozamiz:

SQL
-- Dasturlash yoki Kiberxavfsizlik yo'nalishida o'qiydigan, 
-- yoshi 20 dan katta bo'lgan va balli 70 dan past bo'lmagan talabalar
SELECT * FROM talabalar 
WHERE (yonalish = 'Dasturlash' OR yonalish = 'Kiberxavfsizlik') 
  AND yosh > 20 
  AND NOT joriy_ball < 70;
3. LIKE va ILIKE farqi
LIKE: Katta-kichik harflarga sezgir (Case-sensitive). 'A%' sharti 'anvar' ni topolmaydi.

ILIKE: Katta-kichik harflarga sezgir emas (Case-insensitive). Faqat PostgreSQL-ga xos operator.

SQL
-- Faqat Katta 'A' bilan boshlanadigan ismlarni topadi (Masalan: Anvar)
SELECT * FROM talabalar WHERE ism LIKE 'A%';

-- Ham 'A', ham 'a' bilan boshlanadigan ismlarni topadi (Masalan: Anvar, asror)
SELECT * FROM talabalar WHERE ism ILIKE 'a%';
4. IS NULL va = NULL farqi
SQL-da NULL bu "qiymat yo'q" yoki "noma'lum" degan ma'noni anglatadi. Uni hech qanday qiymat bilan (hatto boshqa NULL bilan ham) solishtirib bo'lmaydi.

= NULL: Har doim UNKNOWN (yolg'on variant) qaytaradi va hech qanday qatorni topmaydi.

IS NULL: Maxsus operator bo'lib, katak haqiqatdan ham bo'sh yoki yo'qligini to'g'ri tekshiradi.

SQL
-- XATO: Bu so'rov hech qanday natija qaytarmaydi (hatto bazada bo'sh kataklar bo'lsa ham)
SELECT * FROM talabalar WHERE yonalish = NULL;

-- TO'G'RI: Bo'sh kataklarni aniqlash uchun faqat shu usul ishlaydi
SELECT * FROM talabalar WHERE yonalish IS NULL;
5. Hisoblangan shart bilan WHERE (ball + 5 > 80)
WHERE operatori ichida ustunlar ustida matematik amallar bajarib, so'ng filtrlash mumkin:

SQL
-- Joriy baliga 5 ball qo'shilganda 80 dan oshadigan talabalarni saralash
SELECT ism, familiya, joriy_ball, (joriy_ball + 5) AS taxminiy_ball
FROM talabalar 
WHERE (joriy_ball + 5) > 80;
6. So'rov natijalari hisoboti (8+ xil natijaviy ko'rinish)
Quyidagi jadvalda yuqoridagi so'rovlar ishga tushirilganda konsolda yoki pgAdmin-da qanday natijalar chiqishi sxematik ko'rsatilgan:

№	So'rov turi	Kutilgan natija / Qatorlar soni	Izoh
1	WHERE yonalish = 'Dasturlash'	4 ta qator (Anvar, Diyora, Zilola, Kamola)	Faqat to'liq mos kelganlar
2	WHERE joriy_ball > 80	5 ta qator (Anvar, Malika, Diyora, Zilola, Nigora, Kamola)	80 dan katta ballar
3	WHERE yosh BETWEEN 20 AND 22	7 ta qator (Anvar, Jasur, Diyora, Bekzod, Nigora, Sardor, Kamola)	20, 21 va 22 yoshdagilar
4	WHERE yonalish IN (...)	4 ta qator (Jasur, Bekzod, Otabek, Sardor)	Ro'yxatdagi yo'nalishlar
5	WHERE familiya LIKE 'K%'	2 ta qator (Karimov, Kamola - agar familiyasi mos bo'lsa)	Faqat 'K' harfi
6	Murakkab shart (AND/OR/NOT)	1 ta qator (Diyora - 22 yosh, Dasturlash, ball > 70)	Bir nechta süzgich kesishmasi
7	ILIKE 'a%'	1 ta qator (Anvar)	Katta-kichik harf farqsiz qidiruv
8	Hisoblangan WHERE (ball + 5) > 80	7 ta qator (Asl bali 75 dan yuqori bo'lgan barcha talabalar)	Matematik hisobga ko'ra filtr
Ushbu SQL kodlarini ham avvalgi .sql faylingizga qo'shib, pgAdmin-da F5 tugmasi orqali tekshirib ko'rishingiz mumkin.
