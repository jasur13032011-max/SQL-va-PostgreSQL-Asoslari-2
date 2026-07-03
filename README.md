# SQL-va-PostgreSQL-Asoslari-2
Mana, siz so'ragan barcha shartlarni o'z ichiga olgan SQL ma'lumotlar bazasi sxemasi va misollari. Biz "Onlayn Kurslar va Obunalar" tizimini loyihalashtiramiz.

1. Jadval sxemalarini yaratish (CREATE TABLE)
Foydalanuvchilar jadvali (users)
Bu yerda UNIQUE, NOT NULL va DEFAULT cheklovlari ishlatilgan.

SQL
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'suspended', 'inactive'))
);
Kurslar jadvali (courses)
Bu yerda pul uchun NUMERIC(p, s) va kurs narxini tekshirish uchun CHECK ishlatilgan.

SQL
CREATE TABLE courses (
    course_id SERIAL PRIMARY KEY,
    title VARCHAR(150) NOT NULL,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    is_published BOOLEAN DEFAULT FALSE
);
Obunalar jadvali (subscriptions)
Bu yerda FOREIGN KEY bog'lanishlari, ON DELETE CASCADE / RESTRICT va Multi-column UNIQUE (bitta foydalanuvchi bitta kursga faqat bir marta obuna bo'lishi uchun) qo'llanilgan.

SQL
CREATE TABLE subscriptions (
    subscription_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    course_id INT NOT NULL,
    start_date TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    end_date TIMESTAMPTZ NOT NULL,
    
    -- Foreign Key va ON DELETE misollari
    CONSTRAINT fk_user FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE,
    CONSTRAINT fk_course FOREIGN KEY (course_id) REFERENCES courses(course_id) ON DELETE RESTRICT,
    
    -- Multi-column UNIQUE (Foydalanuvchi + Kurs kombinatsiyasi takrorlanmasligi kerak)
    CONSTRAINT unique_user_course UNIQUE (user_id, course_id)
);
2. ALTER TABLE yordamida ustun qo'shish va o'zgartirish
Jadval yaratilgandan keyin unga o'zgartirish kiritish:

SQL
-- 1. Kurslar jadvaliga yangi ustun qo'shish
ALTER TABLE courses ADD COLUMN duration_hours INT DEFAULT 0;

-- 2. Foydalanuvchilar jadvalidagi 'username' ustunining ma'lumot turini o'zgartirish
ALTER TABLE users ALTER COLUMN username TYPE VARCHAR(100);
3. Sxemada xato ma'lumot kiritishga urinish (Testlar)
Keling, yuqoridagi cheklovlar (Constraints) qanday ishlashini tekshirib ko'ramiz.

Xato 1: CHECK cheklovining buzilishi (Manfiy narx)
Kurs narxi 0 dan katta yoki teng bo'lishi kerak edi.

SQL
INSERT INTO courses (title, price) VALUES ('Sun''iy Intellekt', -50.00);
❌ Xatolik xabari: ERROR: new row for relation "courses" violates check constraint "courses_price_check"

Xato 2: UNIQUE cheklovining buzilishi (Bir xil email)
Bir xil emailli foydalanuvchi ro'yxatdan o'ta olmaydi.

SQL
INSERT INTO users (username, email) VALUES ('ali', 'ali@example.com');
-- Ikkinchi marta xuddi shu email bilan urinamiz:
INSERT INTO users (username, email) VALUES ('valijon', 'ali@example.com');
❌ Xatolik xabari: ERROR: duplicate key value violates unique constraint "users_email_key"

Xato 3: Multi-column UNIQUE buzilishi
Foydalanuvchi bitta kursga ikki marta obuna bo'la olmaydi.

SQL
-- Birinchi marta muvaffaqiyatli yoziladi:
INSERT INTO subscriptions (user_id, course_id, end_date) VALUES (1, 1, '2026-12-31 23:59:59+05');

-- Xuddi shu foydalanuvchi yana shu kursga yozilmoqchi bo'lsa:
INSERT INTO subscriptions (user_id, course_id, end_date) VALUES (1, 1, '2026-12-31 23:59:59+05');
❌ Xatolik xabari: ERROR: duplicate key value violates unique constraint "unique_user_course"

4. ON DELETE CASCADE va RESTRICT amalda
Aytaylik, tizimda ma'lumotlar bor. Endi o'chirishni sinab ko'ramiz:

ON DELETE RESTRICT (Taqiqlash)
Agar kursga kimdir obuna bo'lgan bo'lsa, u kursni o'chirib tashlay olmaymiz.

SQL
DELETE FROM courses WHERE course_id = 1;
❌ Xatolik xabari: ERROR: update or delete on table "courses" violates foreign key constraint "fk_course" on table "subscriptions"
(Izoh: Tizim kursni o'chirishga yo'l qo'ymaydi, chunki obunalar jadvali unga bog'lanib turibdi).

ON DELETE CASCADE (Zanjirli o'chirish)
Agar foydalanuvchi o'chirib yuborilsa, unga tegishli barcha obunalar avtomatik ravishda o'chib ketadi.

SQL
DELETE FROM users WHERE user_id = 1;
Natija: So'rov muvaffaqiyatli bajariladi (Query returned successfully). users jadvalidan ushbu foydalanuvchi o'chadi va subscriptions jadvalidagi unga tegishli qatorlar ham avtomatik tozalanadi.
