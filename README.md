# SQL-va-PostgreSQL-Asoslari-2
PostgreSQL-da ma'lumotlarni kiritish (INSERT), o'zgartirish (UPDATE), o'chirish (DELETE) va tranzaksiyalar (ACID tamoyillari) bilan ishlash bo'yicha so'ralgan barcha amaliy misollar quyida keltirilgan.

1. Ma'lumot kiritish (INSERT — 5 xil usul)
SQL
-- 1. Standart usul (Barcha ustunlarni ko'rsatib bitta qator kiritish)
INSERT INTO talabalar (ism, familiya, yosh, yonalish, joriy_ball) 
VALUES ('Eldor', 'Nomozov', 20, 'Kiberxavfsizlik', 75.00);

-- 2. Qisqartirilgan usul (Ustun nomlarisiz - jadvaldagi ketma-ketlik bo'yicha)
-- Diqqat: 'id' SERIAL bo'lgani uchun DEFAULT kalit so'zi ishlatiladi
INSERT INTO talabalar VALUES (DEFAULT, 'Madina', 'Nasimova', 21, 'Dasturlash', 90.00);

-- 3. Vergul orqali bir vaqtda bir nechta qatorni kiritish (Bulk Insert)
INSERT INTO talabalar (ism, familiya, yosh, yonalish, joriy_ball) VALUES 
('Shaxzod', 'Xasanov', 22, 'Sun''iy intellekt', 68.50),
('Lola', 'Karimova', 19, 'Ma''lumotlar tahlili', 84.20);

-- 4. RETURNING yordamida kiritilgan ma'lumot id-sini qaytarib olish
INSERT INTO talabalar (ism, familiya, yonalish) 
VALUES ('Rustam', 'Ganiyev', 'Dasturlash') 
RETURNING id, ism;

-- 5. Boshqa jadvaldan ma'lumot nusxalash (INSERT INTO ... SELECT)
-- Faraz qilaylik, 'arxiv_talabalar' degan jadval mavjud
INSERT INTO talabalar (ism, familiya, yonalish)
SELECT ism, familiya, yonalish FROM arxiv_talabalar WHERE yosh = 20;
2. Ma'lumotni yangilash (UPDATE — 3 xil usul)
SQL
-- 1. Oddiy UPDATE (Konkret shart asosida qiymatni o'zgartirish)
UPDATE talabalar 
SET yonalish = 'Sun''iy intellekt' 
WHERE id = 5;

-- 2. Expression (Ifoda) yordamida yangilash (Eski qiymat ustida amal bajarish)
-- Dasturlash yo'nalishidagi barcha talabalarning balini 5 ballga oshirish
UPDATE talabalar 
SET joriy_ball = joriy_ball + 5 
WHERE yonalish = 'Dasturlash';

-- 3. Korelatsion (Bog'langan) UPDATE (Boshqa jadval ma'lumotiga tayanib yangilash)
-- Talabaning balini 'imtihonlar' jadvalidagi eng oxirgi ballga qarab yangilash
UPDATE talabalar t
SET joriy_ball = (SELECT max_baho FROM imtihonlar i WHERE i.talaba_id = t.id)
WHERE EXISTS (SELECT 1 FROM imtihonlar i WHERE i.talaba_id = t.id);
3. Ma'lumotni o'chirish (DELETE — 2 xil usul)
SQL
-- 1. Shart bo'yicha o'chirish va o'chirilgan qatorlarni RETURNING bilan ko'rish
DELETE FROM talabalar 
WHERE joriy_ball < 50
RETURNING id, ism, familiya, joriy_ball;

-- 2. Murakkab shart (Subquery) orqali o'chirish
-- Hech qaysi fandan baho olmagan (baholar jadvalida mavjud bo'lmagan) talabalarni o'chirish
DELETE FROM talabalar 
WHERE id NOT IN (SELECT DISTINCT talaba_id FROM baholar)
RETURNING *;
4. Tranzaksiyalar (TCL: COMMIT, ROLLBACK, SAVEPOINT)
Tranzaksiyalar ma'lumotlar bazasining butunligini ta'minlaydi. Bir nechta amal birgalikda muvaffaqiyatli bajarilishi yoki xatolik bo'lsa barchasi bekor bo'lishi kerak.

BEGIN ... COMMIT (O'zgarishlarni doimiy saqlash)
SQL
BEGIN; -- Tranzaksiyani boshlash

UPDATE talabalar SET joriy_ball = joriy_ball + 2 WHERE yonalish = 'Kiberxavfsizlik';
INSERT INTO talabalar (ism, familiya, yonalish) VALUES ('Jasur', 'Akramov', 'Kiberxavfsizlik');

COMMIT; -- Agar xatolik bo'lmasa, barcha o'zgarishlar bazaga yoziladi
BEGIN ... ROLLBACK (O'zgarishlarni butunlay bekor qilish)
SQL
BEGIN;

DELETE FROM talabalar WHERE id = 1; -- Tasodifan noto'g'ri odam o'chirildi
UPDATE talabalar SET joriy_ball = 0; -- Yana bir xato amal bajarildi

ROLLBACK; -- Tranzaksiya ichidagi barcha amallar bekor qilinadi, jadval avvalgi holiga qaytadi
SAVEPOINT bilan qisman bekor qilish
SQL
BEGIN;

UPDATE talabalar SET joriy_ball = 99 WHERE id = 2;
SAVEPOINT nuqta_1; -- Birinchi xavfsiz nuqtani yaratamiz

DELETE FROM talabalar; -- Xatolik: hamma talabalar o'chirib yuborildi!

ROLLBACK TO nuqta_1; -- Faqat o'chirish amali bekor bo'ladi, id=2 bo'lgan yangilanish saqlanib qoladi
COMMIT; -- Tranzaksiyani yakunlaymiz
5. ON CONFLICT DO UPDATE (Upsert)
Agar kiritilayotgan ma'lumot bazada oldindan mavjud bo'lsa (birlamchi kalit yoki UNIQUE ustun bo'yicha to'qnashuv bo'lsa), xatolik bermasdan mavjud qatorni yangilash amali:

SQL
-- Faraz qilaylik 'id' ustuni to'qnashsa, yangi ma'lumot qo'shilmaydi, faqat ball yangilanadi
INSERT INTO talabalar (id, ism, familiya, joriy_ball) 
VALUES (3, 'Jasur', 'Tursunov', 85.00)
ON CONFLICT (id) 
DO UPDATE SET joriy_ball = EXCLUDED.joriy_ball; 
-- EXCLUDED kalit so'zi VALUES ichida kelgan yangi qiymatni anglatadi
6. WHERE operatorisiz UPDATE va DELETE — Katta Xavf!
SQL-da UPDATE va DELETE buyruqlariga WHERE shartini yozish majburiy emas. Ammo bu juda katta xavfni keltirib chiqaradi.

Xavf nimada?
Agar siz tasodifan shart berishni unutsangiz:

SQL
-- DAHSATLI XATO: Bazadagi BARCHA talabalarning ismi 'Xaker' bo'lib qoladi!
UPDATE talabalar SET ism = 'Xaker';

-- DAHSATLI XATO: Jadvaldagi BARCHA ma'lumotlar butunlay o'chib ketadi!
DELETE FROM talabalar;
Natija: Baza hech qanday ogohlantirishsiz jadvaldagi barcha qatorlarni yangilaydi yoki o'chiradi. Katta proyektlarda bu kompaniya uchun millionlab dollar zarar keltirishi yoki biznes faoliyatini to'xtatib qo'yishi mumkin.

Himoyalanish: 1. Har doim bunday so'rovlarni avval BEGIN; tranzaksiyasi ichida yozib, natijani tekshirib, keyin COMMIT; qilish lozim.
2. pgAdmin yoki ishlab chiqarish (Production) muhitlarida "Safe Updates" rejimini yoqib qo'yish kerak, bu rejim WHEREsiz so'rovlarni bloklaydi.
