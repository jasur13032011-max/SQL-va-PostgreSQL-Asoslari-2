# SQL-va-PostgreSQL-Asoslari-2
vMana, SQL-da subquery (ichki so'rovlar), CTE (Common Table Expressions) va Oyna funksiyalari (Window Functions) bilan ishlash bo'yicha batafsil qo'llanma.

1. Subquery (Ichki so'rov) turlari
A. Scalar Subquery
Faqatgina bitta ustun va bitta qator (yagona qiymat) qaytaradigan ichki so'rov. Ustunlar orasida yoki WHERE shartida oddiy o'zgaruvchidek ishlatish mumkin.

SQL
-- Har bir xodimning maoshi va kompaniyadagi o'rtacha maosh farqi
SELECT 
    name, 
    salary,
    (SELECT AVG(salary) FROM employees) AS avg_company_salary,
    salary - (SELECT AVG(salary) FROM employees) AS difference
FROM employees;
B. IN Subquery
Ichki so'rov bitta ustun va bir nechta qatorlar ro'yxatiniaytaradi. Tashqi so'rov shu ro'yxat ichidan moslikni qidiradi.

SQL
-- Kamida bitta buyurtma bergan mijozlarni topish
SELECT customer_id, name 
FROM customers 
WHERE customer_id IN (SELECT DISTINCT customer_id FROM orders);
C. EXISTS Subquery
Ichki so'rovda ma'lumot bor yoki yo'qligini tekshiradi (TRUE/FALSE). INga qaraganda yirik jadvallarda ancha tezroq ishlaydi, chunki moslik topilishi bilan qidirishni to'xtatadi.

SQL
-- Omborida mahsuloti qolmagan kategoriyalarni topish
SELECT category_name 
FROM categories c
WHERE EXISTS (
    SELECT 1 FROM products p 
    WHERE p.category_id = c.category_id AND p.stock_quantity = 0
);
2. CTE (Common Table Expressions)
CTE so'rovlarni o'qishni osonlashtiradigan va murakkab hisobotlarni bo'laklarga bo'lishga yordam beradigan vaqtinchalik natijalar to'plamidir.

Bir nechta CTE birgalikda (WITH a AS ..., b AS ...)
Keling, do'kondagi mijozlarning o'rtacha xaridini va eng ko'p sotilgan mahsulotlarni alohida hisoblab, yakuniy hisobotda birlashtiramiz:

SQL
WITH customer_spending AS (
    -- Har bir mijozning umumiy xarajatini hisoblaymiz
    SELECT customer_id, SUM(total_amount) AS total_spent
    FROM orders
    GROUP BY customer_id
),
top_products AS (
    -- Top 5 ta eng ko'p sotilgan mahsulotni topamiz
    SELECT product_id, COUNT(*) as sales_count
    FROM order_items
    GROUP BY product_id
    ORDER BY sales_count DESC
    LIMIT 5
)
-- Yakuniy oson o'qiladigan hisobot
SELECT 
    u.username, 
    cs.total_spent,
    CASE WHEN tp.product_id IS NOT NULL THEN 'Top Mahsulot Xaridori' ELSE 'Oddiy Xaridor' END as buyer_status
FROM users u
JOIN customer_spending cs ON u.user_id = cs.customer_id
LEFT JOIN order_items oi ON u.user_id = oi.order_id
LEFT JOIN top_products tp ON oi.product_id = tp.product_id;
Recursive CTE (Iyerarxiya uchun)
Kategoriyalar iyerarxiyasini (Ota-bola munosabatlarini) ildizidan boshlab daraxt shaklida yoyib chiqish misoli:

SQL
WITH RECURSIVE category_tree AS (
    -- Ankor qismi: Eng yuqori darajadagi (ota-onasi yo'q) kategoriyalar
    SELECT category_id, category_name, parent_id, 1 AS level
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Rekursiv qismi: Bolalarini ota-onalariga bog'laymiz
    SELECT c.category_id, c.category_name, c.parent_id, ct.level + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.category_id
)
SELECT * FROM category_tree ORDER BY level, category_name;
3. Oyna funksiyalari (Window Functions)
A. ROW_NUMBER yordamida har sinfdan TOP-1
ROW_NUMBER() har bir guruh ichida qatorlarga qat'iy ketma-ket raqam beradi.

SQL
WITH ranked_students AS (
    SELECT 
        student_name, 
        class_id, 
        score,
        ROW_NUMBER() OVER (PARTITION BY class_id ORDER BY score DESC) AS rn
    FROM students
)
-- Har bir sinf (class_id) bo'yicha eng yuqori ball olgan 1 ta talabani saralab olish
SELECT student_name, class_id, score 
FROM ranked_students 
WHERE rn = 1;
B. RANK vs DENSE_RANK farqi
Agar ikkita talaba bir xil ball olsa, bu funksiyalar turlicha raqamlaydi:

SQL
SELECT 
    student_name, score,
    RANK() OVER (ORDER BY score DESC) AS rnk,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rnk
FROM students;
Natija farqi jadvali:

Student	Score	RANK()	DENSE_RANK()	Izoh
Ali	100	1	1	O'rinlar bir xil
Vali	95	2	2	O'rinlar bir xil
Anvar	95	2	2	Ikkalasi ham 2-o'rin
Sardor	90	4	3	RANK() 3-o'rinni tashlab ketadi, DENSE_RANK() esa zichlikni saqlaydi.
C. LAG yordamida oldingi qiymat bilan taqqoslash
LAG() joriy qatordan oldingi qator qiymatini ko'rishga imkon beradi. Oylik sotuvlar o'sishi yoki kamayishini hisoblashda juda qulay.

SQL
SELECT 
    month, 
    monthly_sales,
    LAG(monthly_sales, 1) OVER (ORDER BY month) AS previous_month_sales,
    monthly_sales - LAG(monthly_sales, 1) OVER (ORDER BY month) AS sales_growth
FROM monthly_revenue;
D. Kumulyativ SUM (O'sib boruvchi yakun)
SUM() OVER ichida ORDER BY ishlatilsa, ma'lumotlar o'zidan oldingi qatorlar yig'indisini qo'shib, o'sib borish tartibida hisoblanadi.

SQL
SELECT 
    ordered_at, 
    amount,
    SUM(amount) OVER (ORDER BY ordered_at) AS cumulative_revenue
FROM orders;
E. Ulush (Foiz) hisoblash
Joriy qator qiymatini oyna funksiyasi orqali hisoblangan umumiy yig'indiga bo'lib, foiz chiqarish:

SQL
SELECT 
    product_name, 
    category_id, 
    revenue,
    -- Butun do'kondagi umumiy savdoga nisbatan foizi
    ROUND(revenue * 100.0 / SUM(revenue) OVER (), 2) AS percent_of_total_sales,
    
    -- Faqat o'zining kategoriyasidagi savdoga nisbatan foizi
    ROUND(revenue * 100.0 / SUM(revenue) OVER (PARTITION BY category_id), 2) AS percent_of_category_sales
FROM products;
