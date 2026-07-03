# SQL-va-PostgreSQL-Asoslari-2
PostgreSQL-da indekslar bilan ishlash, so'rovlarni optimallashtirish va unumdorlikni tahlil qilish bo'yicha so'ralgan barcha bosqichlar va misollar:

1. generate_series yordamida test ma'lumotlarini yaratish
Keling, sinovlar o'tkazish uchun customers (mijozlar) jadvalini yaratamiz va generate_series orqali unga 100 000 ta qator ma'lumot yuklaymiz.

SQL
-- Jadval yaratish
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    status VARCHAR(20),
    country_code VARCHAR(3),
    created_at TIMESTAMPTZ
);

-- 100 000 ta qator generatsiya qilish
INSERT INTO customers (first_name, last_name, email, status, country_code, created_at)
SELECT 
    'First_' || i,
    'Last_' || i,
    'user_' || i || '@example.com',
    (ARRAY['active', 'inactive', 'pending'])[floor(random() * 3) + 1],
    (ARRAY['UZB', 'USA', 'DEU', 'KAZ'])[floor(random() * 4) + 1],
    NOW() - (random() * INTERVAL '365 days')
FROM generate_series(1, 100000) AS i;
2. Indekssiz EXPLAIN (Seq Scan)
Hali indeks yaratilmagan email ustuni bo'yicha qidiruv beramiz va rejanini ko'ramiz.

SQL
EXPLAIN SELECT * FROM customers WHERE email = 'user_50000@example.com';
Natija (Plan):

Plaintext
Seq Scan on customers  (cost=0.00..2634.00 rows=1 width=51)
  Filter: ((email)::text = 'user_50000@example.com'::text)
⚠️ Izoh: Bazada indeks yo'qligi sababli Seq Scan (Sequential Scan - ketma-ket qidiruv) sodir bo'ldi. PostgreSQL kerakli qatorni topish uchun barcha 100 000 ta qatorni birma-bir tekshirib chiqdi.

3. B-tree indeks qo'shish va EXPLAIN ANALYZE
Endi email ustuniga standart B-tree indeksini qo'shamiz va so'rov tezligini o'lchaymiz.

SQL
-- Indeks yaratish
CREATE INDEX idx_customers_email ON customers(email);

-- Tezlikni taqqoslash uchun ANALYZE bilan ishga tushirish
EXPLAIN ANALYZE SELECT * FROM customers WHERE email = 'user_50000@example.com';
Natija (Plan & Time):

Plaintext
Index Scan using idx_customers_email on customers  (cost=0.29..8.30 rows=1 width=51) (actual time=0.042..0.043 ms)
  Index Cond: ((email)::text = 'user_50000@example.com'::text)
Planning Time: 0.088 ms
Execution Time: 0.061 ms
⚡ Vaqt taqqoslovi: Indekssiz holatda so'rov to'liq jadvalni aylanib chiqishga millisekundlab vaqt sarflagan bo'lsa (yirikroq bazalarda soniyalarga aylanadi), indeks qo'yilgach 0.061 ms (sekundning mingdan bir ulushi) ichida Index Scan orqali natijani qaytardi.

4. Maxsus indeks turlari va misollar
A. Multi-column (Ko'p ustunli) indeks
Agar biz ko'pincha mijozlarni ham davlati (country_code), ham holati (status) bo'yicha birga qidirsak, ushbu indeks juda foydali bo'ladi.

SQL
CREATE INDEX idx_customers_country_status ON customers(country_code, status);

-- Quyidagi so'rov ushbu indeksdan maksimal foydalanadi:
SELECT * FROM customers WHERE country_code = 'UZB' AND status = 'active';
B. Functional (Funksional) indeks
Agar qidiruvda qiymatlarni kichik harflarga o'girib (LOWER) qidirsak, oddiy indeks ishlamay qoladi. Buning uchun funksiyaning o'ziga indeks beriladi.

SQL
CREATE INDEX idx_customers_lower_last_name ON customers(LOWER(last_name));

-- So'rov misoli:
SELECT * FROM customers WHERE LOWER(last_name) = 'last_50000';
C. Partial (Qisman) indeks
Agar bizga faqat ma'lum bir shartga mos qatorlarni tez-tez qidirish kerak bo'lsa (masalan, faqat pending holatidagilar), indeks hajmini tejash uchun shartli indeks yaratamiz.

SQL
CREATE INDEX idx_customers_pending ON customers(status) WHERE status = 'pending';
Bu indeks faqat status = 'pending' bo'lgan qatorlar manzilini saqlagani uchun diskdan juda kam joy egallaydi.

5. Foreign Key (Tashqi kalit) ustuniga indeks qo'shish
💡 Muhim qoida: PostgreSQL jadval yaratilayotganda PRIMARY KEY va UNIQUE cheklovlari uchun avtomatik indeks yaratadi, lekin FOREIGN KEY (tashqi kalit) ustunlari uchun avtomatik indeks yaratmaydi.

Agar bizda buyurtmalar jadvali bo'lsa va u customersga bog'langan bo'lsa, bog'lanish ustuniga indeksni qo'lda qo'shishimiz shart:

SQL
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(customer_id), -- FK ustuni
    amount NUMERIC(10,2)
);

-- PostgreSQL buni avtomatik qilmagani uchun o'zimiz indekslaymiz:
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
Bu indeks JOIN amallari bajarilganda va mijoz o'chirilganda (ON DELETE CASCADE tekshiruvida) tizim qotib qolmasligini ta'minlaydi.

6. Indekslar narxi (Cost) haqida qisqacha hisobot
Indekslar so'rovlarni tezlashtirsa-da, ularning o'ziga yarasha "to'lovi" (narxi) bor:

Amaliyot turi	Indeksning ta'siri	Sababi
SELECT (Qidiruv)	🚀 Juda tezlashadi	B-tree daraxti bo'ylab qidiruv O(logn) vaqt oladi.
INSERT (Qo'shish)	🐌 Sekinganlashadi	Yangi qator qo'shilganda, baza nafaqat jadvalga, balki barcha indeks daraxtlariga ham ushbu ma'lumotni joylashtirishi kerak.
UPDATE (O'zgartirish)	🐌 Sekinganlashadi	Agar indekslangan ustun qiymati o'zgarsa, indeks daraxtidagi manzillar qayta tartiblanadi.
DELETE (O'chirish)	🐌 Sekinganlashadi	Qator o'chirilganda, uning indekslardagi barcha ko'rsatkichlari ham tozalanishi shart.
Disk hajmi	📁 Kattalashadi	Har bir indeks diskda qo'shimcha joy egallaydi. Ba'zan indekslar hajmi jadvalning o'zidan ham o'tib ketishi mumkin.
Xulosa: Har bir ustunga ko'r-ko'rona indeks qo'yish yaramaydi. Indekslarni faqat WHERE, JOIN, ORDER BY shartlarida ko'p ishlatiladigan ustunlar uchun yaratish maqsadga muvofiqdir.
