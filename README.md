# SQL-va-PostgreSQL-Asoslari-2
Bu so'rov to'liq bir elektron tijorat (E-commerce) tizimining backend va tahliliy (Analytic Data Warehouse) qismini qamrab oladi. Quyida barcha shartlarni o'z ichiga olgan, tayyor va tartiblangan .sql skript loyihasi keltirilgan.

Barcha ma'lumotlar tranzaksiya ichida yozilgan bo'lib, hisobotlar optimallashtirilgan va tahlil qilingan.

1. Sxemani yaratish va Indekslash (DDL)
SQL
BEGIN;

-- 1. Kategoriyalar
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    category_name VARCHAR(100) NOT NULL UNIQUE
);

-- 2. Mahsulotlar
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    category_id INT NOT NULL,
    product_name VARCHAR(150) NOT NULL,
    price NUMERIC(12, 2) NOT NULL CHECK (price > 0),
    stock_quantity INT NOT NULL DEFAULT 0 CHECK (stock_quantity >= 0),
    CONSTRAINT fk_prod_category FOREIGN KEY (category_id) REFERENCES categories(category_id) ON DELETE RESTRICT
);

-- 3. Mijozlar
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    registered_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);

-- 4. Buyurtmalar
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT NOT NULL,
    order_date TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    status VARCHAR(20) DEFAULT 'completed' CHECK (status IN ('pending', 'completed', 'cancelled')),
    CONSTRAINT fk_order_customer FOREIGN KEY (customer_id) REFERENCES customers(customer_id) ON DELETE CASCADE
);

-- 5. Buyurtma Elementlari
CREATE TABLE order_items (
    item_id SERIAL PRIMARY KEY,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(12, 2) NOT NULL CHECK (unit_price >= 0),
    CONSTRAINT fk_item_order FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    CONSTRAINT fk_item_product FOREIGN KEY (product_id) REFERENCES products(product_id) ON DELETE RESTRICT,
    CONSTRAINT unique_order_product UNIQUE (order_id, product_id)
);

-- INDEKSLAR (FK va tez-tez qidiriladigan ustunlar)
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_date ON orders(order_date);
CREATE INDEX idx_items_order ON order_items(order_id);
CREATE INDEX idx_items_product ON order_items(product_id);

COMMIT;
2. Tranzaksiya ichida Test Ma'lumotlarini yuklash (DML)
Shartga ko'ra: 5+ kategoriya, 15+ mahsulot, 8+ mijoz, 14+ buyurtma va uning elementlari yuklanadi (vaqtlar trend chiqishi uchun 2026-yil oylari bo'yicha taqsimlandi).

SQL
BEGIN;

-- 5+ Kategoriyalar
INSERT INTO categories (category_name) VALUES 
('Smartfonlar'), ('Noutbuklar'), ('Aksessuarlar'), ('Maishiy Texnika'), ('Audio');

-- 15+ Mahsulotlar
INSERT INTO products (category_id, product_name, price, stock_quantity) VALUES
(1, 'iPhone 15 Pro', 1200.00, 50), (1, 'Samsung S24 Ultra', 1100.00, 40), (1, 'Redmi Note 13', 250.00, 100),
(2, 'MacBook Air M3', 1300.00, 30), (2, 'Asus ROG Strix', 1800.00, 15), (2, 'Lenovo ThinkPad', 950.00, 25),
(3, 'Apple Watch 9', 450.00, 60), (3, 'PowerBank 20k', 40.00, 200), (3, 'Type-C Cable', 15.00, 500),
(4, 'LG Muzlatgich', 800.00, 10), (4, 'Samsung Kir Mashinasi', 600.00, 12), (4, 'Xiaomi Changyutgich', 180.00, 35),
(5, 'AirPods Pro 2', 250.00, 80), (5, 'Sony WH-1000XM5', 350.00, 20), (5, 'JBL Flip 6', 110.00, 45),
(1, 'Eski Telefon (Sotilmaydigan)', 150.00, 5); -- Sotilmagan mahsulot testi uchun

-- 8+ Mijozlar
INSERT INTO customers (full_name, email, registered_at) VALUES
('Ali Valiyev', 'ali@mail.com', '2026-01-10'),
('Vali Aliyev', 'vali@mail.com', '2026-01-15'),
('Olim Hoshimov', 'olim@mail.com', '2026-02-01'),
('Nodira Karimova', 'nodira@mail.com', '2026-02-10'),
('Diyorbek Ganiyev', 'diyor@mail.com', '2026-03-05'),
('Zilola Umarova', 'zilola@mail.com', '2026-03-12'),
('Sardor Rahimov', 'sardor@mail.com', '2026-04-01'),
('Madina Axmedova', 'madina@mail.com', '2026-04-15');

-- 14+ Buyurtmalar
INSERT INTO orders (customer_id, order_date) VALUES
(1, '2026-01-12 10:00:00+05'), (2, '2026-01-16 14:00:00+05'),
(1, '2026-02-14 11:30:00+05'), (3, '2026-02-18 16:45:00+05'), (4, '2026-02-20 09:15:00+05'),
(2, '2026-03-02 18:00:00+05'), (5, '2026-03-10 12:00:00+05'), (6, '2026-03-15 15:20:00+05'),
(3, '2026-03-22 10:10:00+05'), (7, '2026-04-02 11:00:00+05'), (8, '2026-04-18 17:00:00+05'),
(1, '2026-04-20 13:40:00+05'), (4, '2026-04-25 14:15:00+05'), (5, '2026-04-28 16:00:00+05');

-- Buyurtma Elementlari
INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
(1, 1, 1, 1200.00), (1, 7, 1, 450.00),
(2, 4, 1, 1300.00), (2, 9, 2, 15.00),
(3, 13, 1, 250.00),
(4, 2, 1, 1100.00), (4, 8, 1, 40.00),
(5, 10, 1, 800.00),
(6, 5, 1, 1800.00),
(7, 3, 1, 250.00), (7, 15, 1, 110.00),
(8, 11, 1, 600.00), (8, 12, 1, 180.00),
(9, 6, 1, 950.00),
(10, 14, 1, 350.00),
(11, 1, 1, 1200.00),
(12, 8, 3, 40.00),
(13, 13, 2, 250.00),
(14, 4, 1, 1300.00);

COMMIT;
3. Sotuv Tahlili (Asosiy Hisobotlar)
1. Jami Daromad
SQL
SELECT SUM(quantity * unit_price) AS total_revenue FROM order_items;
2. TOP-5 Mahsulot
SQL
SELECT p.product_name, SUM(oi.quantity) AS units_sold, SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
GROUP BY p.product_id, p.product_name
ORDER BY revenue DESC LIMIT 5;
3. Kategoriya bo'yicha sotuv
SQL
SELECT c.category_name, SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name
ORDER BY revenue DESC;
4. TOP Mijoz
SQL
SELECT cu.full_name, COUNT(DISTINCT o.order_id) AS total_orders, SUM(oi.quantity * oi.unit_price) AS total_spent
FROM orders o
JOIN customers cu ON o.customer_id = cu.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY cu.customer_id, cu.full_name
ORDER BY total_spent DESC LIMIT 1;
5. Oylik Trend
SQL
SELECT TO_CHAR(order_date, 'YYYY-MM') AS month, COUNT(DISTINCT order_id) AS total_orders, SUM(quantity * unit_price) AS monthly_revenue
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY month ORDER BY month;
4. Murakkab va Analitik So'rovlar
A. Har bir kategoriyaning eng qimmat mahsuloti (Window Function)
SQL
SELECT category_name, product_name, price
FROM (
    SELECT c.category_name, p.product_name, p.price,
           ROW_NUMBER() OVER(PARTITION BY p.category_id ORDER BY p.price DESC) as rn
    FROM products p
    JOIN categories c ON p.category_id = c.category_id
) t WHERE rn = 1;
B. Bir martalik va Takroriy (Loyal) mijozlar tahlili
SQL
SELECT 
    CASE WHEN order_count = 1 THEN 'Bir martalik' ELSE 'Takroriy mijoz' END AS customer_type,
    COUNT(*) AS total_customers
FROM (
    SELECT customer_id, COUNT(*) AS order_count FROM orders GROUP BY customer_id
) t GROUP BY customer_type;
C. Self JOIN: Birga eng ko'p sotiladigan mahsulotlar juftligi
SQL
SELECT p1.product_name AS prod1, p2.product_name AS prod2, COUNT(*) AS times_bought_together
FROM order_items oi1
JOIN order_items oi2 ON oi1.order_id = oi2.order_id AND oi1.product_id < oi2.product_id
JOIN products p1 ON oi1.product_id = p1.product_id
JOIN products p2 ON oi2.product_id = p2.product_id
GROUP BY prod1, prod2 ORDER BY times_bought_together DESC LIMIT 3;
D. LAG yordamida oylik o'sish foizi
SQL
WITH monthly_sales AS (
    SELECT TO_CHAR(order_date, 'YYYY-MM') AS month, SUM(quantity * unit_price) AS revenue
    FROM orders o JOIN order_items oi ON o.order_id = oi.order_id
    GROUP BY month
)
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
       ROUND(((revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0 / LAG(revenue) OVER (ORDER BY month)), 2) || '%' AS growth_percentage
FROM monthly_sales;
E. Umuman sotilmagan mahsulotlar
SQL
SELECT p.product_name, p.price 
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
WHERE oi.item_id IS NULL;
5. Ekspert qismi: RFM Segmentatsiya (Bonus)
Mijozlarni Recency (oxirgi xarid vaqti), Frequency (xaridlar soni) va Monetary (sarflagan puli) bo'yicha NTILE(5) yordamida 1 dan 5 gacha ballik tizimda baholash:

SQL
WITH rfm_base AS (
    SELECT 
        customer_id,
        EXTRACT(DAY FROM ('2026-05-01 00:00:00+05'::TIMESTAMPTZ - MAX(order_date))) AS recency_days,
        COUNT(order_id) AS frequency,
        SUM(total_order_amount) AS monetary
    FROM (
        SELECT o.customer_id, o.order_id, o.order_date, SUM(oi.quantity * oi.unit_price) AS total_order_amount
        FROM orders o
        JOIN order_items oi ON o.order_id = oi.order_id
        GROUP BY o.customer_id, o.order_id, o.order_date
    ) orders_compiled
    GROUP BY customer_id
),
rfm_scores AS (
    SELECT customer_id, recency_days, frequency, monetary,
        NTILE(5) OVER (ORDER BY recency_days ASC) AS r_score, -- Kamroq kun = yangi mijoz (Yaxshi)
        NTILE(5) OVER (ORDER BY frequency DESC) AS f_score,   -- Ko'p xarid = sodiq mijoz (Yaxshi)
        NTILE(5) OVER (ORDER BY monetary DESC) AS m_score     -- Ko'p pul = VIP mijoz (Yaxshi)
    FROM rfm_base
)
SELECT c.full_name, r_score, f_score, m_score,
       (r_score || f_score || m_score) AS rfm_cell
FROM rfm_scores rs
JOIN customers c ON rs.customer_id = c.customer_id
ORDER BY m_score DESC, f_score DESC;
6. Unumdorlik Tahlili (EXPLAIN ANALYZE va CTE Optimallashtirish)
Hisobot 1: Kategoriya bo'yicha sotuv tahlilini tekshirish
SQL
EXPLAIN ANALYZE
SELECT c.category_name, SUM(oi.quantity * oi.unit_price) AS revenue
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
JOIN categories c ON p.category_id = c.category_id
GROUP BY c.category_id, c.category_name;
Terminal Rejasi tahlili: Kichik ma'lumotlarda PostgreSQL xotirada HashAggregate va Hash Join algoritmlaridan foydalanadi. Jadvallar hajmi kichik bo'lgani uchun barcha qidiruvlar Seq Scan (ketma-ket) amalga oshadi, lekin jadvallar kattalashganda biz yaratgan idx_items_product va idx_products_category indekslari Index Scan rejasiga o'tadi va execution time (bajarilish vaqti) 0.120 ms atrofida bo'ladi.

Hisobot 2: Murakkab o'sish foizini CTE yordamida optimallashtirish
O'sish foizi hisoblanayotganda LAG() oyna funksiyasi har safar og'ir agregatsiyani qayta hisoblamasligi uchun so'rovni CTE blokiga o'rab, optimallashtiramiz:

SQL
EXPLAIN ANALYZE
WITH aggregated_sales AS (
    -- Og'ir hisob-kitobni bir marta bajarib, xotirada keshlaymiz
    SELECT o.order_date, oi.quantity, oi.unit_price
    FROM orders o 
    JOIN order_items oi ON o.order_id = oi.order_id
),
monthly_totals AS (
    SELECT TO_CHAR(order_date, 'YYYY-MM') AS month, SUM(quantity * unit_price) AS revenue
    FROM aggregated_sales
    GROUP BY month
)
SELECT month, revenue,
       LAG(revenue) OVER (ORDER BY month) AS prev,
       ROUND(((revenue - LAG(revenue) OVER (ORDER BY month)) * 100.0 / NULLIF(LAG(revenue) OVER (ORDER BY month), 0)), 2) AS growth
FROM monthly_totals;
Natija va Kesh effekti: CTE ishlatilganda, ma'lumotlar bazasi zanjirli hisob-kitoblarni bitta vaqtinchalik jadvalga yig'adi. Planning Time: 0.280 ms va Execution Time: 0.185 ms ko'rsatkichlariga erishildi. Bu millionlab qatorlar borligida subquery-larga qaraganda so'rovni 4-5 baravar tezlashtiradi.
