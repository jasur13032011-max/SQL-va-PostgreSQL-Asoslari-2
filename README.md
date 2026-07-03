# SQL-va-PostgreSQL-Asoslari-2
PostgreSQL-da jadvallarni o'zaro bog'lash (JOIN) relyatsion ma'lumotlar bazalari bilan ishlashning eng muhim qismidir. Misollarni ko'rishdan oldin, bazamizda 3 ta jadval bor deb tasavvur qilamiz: talabalar, fanlar va ularni bog'laydigan baholar jadvali.

1. Jadvallarni o'zaro bog'lash (JOIN turlari)
INNER JOIN bilan 3 ta jadvalni bog'lash
Faqat baho olgan talabalarni, ularning qaysi fandan qanday baho olganini ko'rish uchun uchta jadvalni zanjir simon bog'laymiz:

SQL
SELECT t.ism, t.familiya, f.fan_nomi, b.baho
FROM talabalar t
INNER JOIN baholar b ON t.id = b.talaba_id
INNER JOIN fanlar f ON b.fan_id = f.id;
LEFT JOIN bilan bahosizlarni saqlash
Baho olgan yoki umuman bahosi yo'q (imtihonga kirmagan) barcha talabalarni ro'yxatda saqlab qolish uchun LEFT JOIN ishlatiladi:

SQL
SELECT t.ism, t.familiya, b.baho
FROM talabalar t
LEFT JOIN baholar b ON t.id = b.talaba_id;
Bahosi yo'q talabalarni topish (WHERE ... IS NULL)
LEFT JOIN qilinganda, bahosi yo'q talabalarning baho ustunlariga NULL yoziladi. Faqat ularni ajratib olish so'rovi:

SQL
SELECT t.ism, t.familiya
FROM talabalar t
LEFT JOIN baholar b ON t.id = b.talaba_id
WHERE b.id IS NULL; -- Baholar jadvalidan hech qanday mos qator topilmaganlar
2. Murakkab JOIN va Analitik So'rovlar
Har talabaning o'rtacha bahosi (LEFT JOIN + GROUP BY)
Baho olmagan talabalarni ham hisobotdan tushirib qoldirmaslik uchun LEFT JOIN va GROUP BY birga ishlatiladi:

SQL
SELECT t.ism, t.familiya, ROUND(AVG(b.baho), 2) AS ortacha_baho
FROM talabalar t
LEFT JOIN baholar b ON t.id = b.talaba_id
GROUP BY t.id, t.ism, t.familiya;
Har fan bo'yicha eng yuqori ball olgan talaba
Buning uchun subquery yoki PostgreSQL-ning DISTINCT ON operatoridan foydalanish eng qulay yechim hisoblanadi:

SQL
SELECT DISTINCT ON (f.id) f.fan_nomi, t.ism, t.familiya, b.baho
FROM fanlar f
INNER JOIN baholar b ON f.id = b.fan_id
INNER JOIN talabalar t ON b.talaba_id = t.id
ORDER BY f.id, b.baho DESC;
Self JOIN misoli (Bitta jadvalni o'ziga bog'lash)
Bitta jadvalda o'zaro bog'liqlikni tekshirish. Masalan, bir xil yo'nalishda o'qiydigan kursdoshlarni juftlik qilib chiqarish:

SQL
SELECT t1.ism AS talaba_1, t2.ism AS talaba_2, t1.yonalish
FROM talabalar t1
INNER JOIN talabalar t2 ON t1.yonalish = t2.yonalish AND t1.id < t2.id; 
-- t1.id < t2.id sharti bitta juftlik ikki marta takrorlanmasligi (A va B, keyin B va A) uchun qo'yiladi.
3. Ataylab xato: Implicit Join va Dekart ko'paytmasi (Cartesian Product)
SQL-ning eski standartlarida (SQL-92 gacha) JOIN so'zi o'rniga jadvallar shunchaki vergul bilan ajratib yozilgan. Bu Implicit Join deyiladi.

Xato / Xavfli kod:
SQL
-- Agarda WHERE sharti esdan chiqsa yoki noto'g'ri yozilsa:
SELECT talabalar.ism, fanlar.fan_nomi
FROM talabalar, fanlar; 
Natija va Sababi (Cartesian / CROSS JOIN):
Agarda siz ON yoki WHERE orqali bog'lanish shartini bermasangiz, SQL har ikkala jadvaldagi barcha qatorlarni bir-biriga ko'paytirib chiqadi (Dekart ko'paytmasi).

Masalan, talabalar jadvalida 10 ta qator, fanlar jadvalida 5 ta qator bo'lsa, natijada hech qanday mantiqsiz 50 ta qator (10×5) hosil bo'ladi.

Agar katta bazalarda (millionlab qatorli jadvallarda) tasodifan mana shunday so'rov yozib yuborilsa, server xotirasi to'lib, baza butunlay qulashi (crash bo'lishi) mumkin. Shuning uchun har doim zamonaviy va xavfsiz INNER JOIN ... ON ... sintaksisidan foydalanish shart.
