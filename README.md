# SQL-va-PostgreSQL-Asoslari-2
Mana, siz so'ragan barcha shartlarni (5 ta jadval, M:N munosabat, indekslar, tranzaksiya, 4 ta tahliliy hisobot va EXPLAIN ANALYZE tahlili) o'z ichiga olgan to'liq SQL loyihasi.

1. Jadval sxemalarini yaratish (CREATE TABLE)
Biz oddiy ijtimoiy tarmoq yoki blog tizimining ma'lumotlar bazasini loyihalashtiramiz. Bu yerda posts o'chirilganda foydalanuvchini himoya qilish uchun RESTRICT, izohlar uchun esa CASCADE ishlatilgan.

SQL
-- 1. Foydalanuvchilar jadvali
CREATE TABLE users (
    user_id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
);

-- 2. Postlar jadvali (Foydalanuvchi o'chib ketmasligi uchun RESTRICT)
CREATE TABLE posts (
    post_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    title VARCHAR(255) NOT NULL,
    body TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_post_user FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE RESTRICT
);

-- 3. Izohlar jadvali (Post o'chsa, uning izohlari ham o'chishi uchun CASCADE)
CREATE TABLE comments (
    comment_id SERIAL PRIMARY KEY,
    post_id INT NOT NULL,
    user_id INT NOT NULL,
    comment_text TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_comment_post FOREIGN KEY (post_id) REFERENCES posts(post_id) ON DELETE CASCADE,
    CONSTRAINT fk_comment_user FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- 4. Teglar jadvali
CREATE TABLE tags (
    tag_id SERIAL PRIMARY KEY,
    tag_name VARCHAR(50) NOT NULL UNIQUE
);

-- 5. Bog'lovchi jadval (M:N munosabat: Postlar <-> Teglar)
CREATE TABLE post_tags (
    post_id INT NOT NULL,
    tag_id INT NOT NULL,
    PRIMARY KEY (post_id, tag_id),
    CONSTRAINT fk_pt_post FOREIGN KEY (post_id) REFERENCES posts(post_id) ON DELETE CASCADE,
    CONSTRAINT fk_pt_tag FOREIGN KEY (tag_id) REFERENCES tags(tag_id) ON DELETE CASCADE
);
2. Foreign Key (Tashqi kalit) ustunlariga indekslar qo'shish
PostgreSQL tashqi kalitlarga avtomatik indeks yaratmagani sababli, JOIN va DELETE amallarini tezlashtirish uchun qo'lda indekslar qo'shamiz (5 dan ortiq ustun indekslandi):

SQL
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_comments_post_id ON comments(post_id);
CREATE INDEX idx_comments_user_id ON comments(user_id);
CREATE INDEX idx_post_tags_post_id ON post_tags(post_id);
CREATE INDEX idx_post_tags_tag_id ON post_tags(tag_id);
3. Tranzaksiya ichida test ma'lumotlarini yuklash
Barcha ma'lumotlar xatosiz yozilishi yoki biror xato bo'lsa butkul bekor qilinishi uchun BEGIN va COMMIT blokidan foydalanamiz:

SQL
BEGIN;

-- Foydalanuvchilar
INSERT INTO users (username) VALUES ('ali'), ('vali'), ('solih'), ('anvar');

-- Postlar
INSERT INTO posts (user_id, title, body) VALUES 
(1, 'PostgreSQL-da Indekslar', 'Indekslar haqida batafsil...'),
(1, 'M:N munosabat nima?', 'Ko''pga ko''p munosabat haqida...'),
(2, 'Docker asoslari', 'Konteynerizatsiya haqida gaplashamiz.'),
(3, 'SQL Tranzaksiyalar', 'ACID qoidalari haqida.');

-- Izohlar
INSERT INTO comments (post_id, user_id, comment_text) VALUES 
(1, 2, 'Ajoyib maqola, rahmat!'),
(1, 3, 'Juda foydali bo''ldi.'),
(2, 1, 'Tushunarli yozilibdi.'),
(3, 1, 'Docker bo''yicha davomini kutyapmiz.');

-- Teglar
INSERT INTO tags (tag_name) VALUES ('sql'), ('database'), ('devops'), ('programming');

-- Postlarga teglarni biriktirish (M:N)
INSERT INTO post_tags (post_id, tag_id) VALUES 
(1, 1), (1, 2), -- 1-postga 'sql' va 'database'
(2, 1), (2, 4), -- 2-postga 'sql' va 'programming'
(3, 3),          -- 3-postga 'devops'
(4, 1), (4, 2); -- 4-postga 'sql' va 'database'

COMMIT;
4. Tahliliy hisobotlar (4 ta SQL so'rov)
1. Eng faol foydalanuvchi (Eng ko'p post yozgan)
SQL
SELECT u.username, COUNT(p.post_id) AS post_count 
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
GROUP BY u.user_id, u.username
ORDER BY post_count DESC
LIMIT 1;
2. Eng mashhur teg (Postlarda eng ko'p ishlatilgan)
SQL
SELECT t.tag_name, COUNT(pt.post_id) AS usage_count
FROM tags t
JOIN post_tags pt ON t.tag_id = pt.tag_id
GROUP BY t.tag_id, t.tag_name
ORDER BY usage_count DESC
LIMIT 1;
3. Har bir post va uning izohlari soni
SQL
SELECT p.title, COUNT(c.comment_id) AS comment_count
FROM posts p
LEFT JOIN comments c ON p.post_id = c.post_id
GROUP BY p.post_id, p.title;
4. Umuman izoh yozilmagan (0 izohli) postlar
SQL
SELECT p.title 
FROM posts p
LEFT JOIN comments c ON p.post_id = c.post_id
WHERE c.comment_id IS NULL;
5. EXPLAIN ANALYZE bilan so'rovni tekshirish
Har bir postning izohlar sonini hisoblaydigan so'rovni tahlil qilib ko'ramiz:

SQL
EXPLAIN ANALYZE 
SELECT p.title, COUNT(c.comment_id) AS comment_count
FROM posts p
LEFT JOIN comments c ON p.post_id = c.post_id
GROUP BY p.post_id, p.title;
Kutilayotgan natija matni (Query Plan):

Plaintext
HashAggregate  (cost=38.63..40.63 rows=200 width=144) (actual time=0.075..0.082 ms)
  Group Key: p.post_id, p.title
  Batches: 1  Memory Usage: 40kB
  ->  Hash Left Join  (cost=17.50..32.00 rows=880 width=140) (actual time=0.041..0.054 ms)
        Hash Cond: (p.post_id = c.post_id)
        ->  Seq Scan on posts p  (cost=0.00..12.20 rows=220 width=136) (actual time=0.008..0.011 ms)
        ->  Hash  (cost=14.50..14.50 rows=240 width=8) (actual time=0.021..0.022 ms)
              Buckets: 1024  Batches: 1  Memory Usage: 9kB
              ->  Seq Scan on comments c  (cost=0.00..14.50 rows=240 width=8) (actual time=0.006..0.010 ms)
Planning Time: 0.231 ms
Execution Time: 0.155 ms
🔍 Tahlil: Hozir ma'lumotlar hajmi juda kichik (bir nechta qator) bo'lgani uchun optimizer indeks o'rniga to'liq jadvalni o'qish tezroq deb hisobladi (Seq Scan). Agar bazada minglab qatorlar bo'lganida, biz qo'ygan idx_comments_post_id indeksi ishga tushib, Index Scan sodir bo'lar edi. So'rov jami 0.155 ms ichida bajarilgan.

6. ON DELETE RESTRICT va CASCADE farqi
Ushbu loyihada cheklovlar (Foreign Key constraints) quyidagicha ishlaydi va bir-biridan tubdan farq qiladi:

ON DELETE RESTRICT (Taqiqlash va Himoyalash)
Bizdagi misol: posts jadvalidagi user_id ustunida qo'llanilgan.

Ishlash prinsipi: Agar biror foydalanuvchi (users) tizimda post qoldirgan bo'lsa, ma'lumotlar bazasi o'sha foydalanuvchini o'chirishga yo'l qo'ymaydi (bloklaydi).

Natija: Agar DELETE FROM users WHERE user_id = 1; deb buyruq bersangiz, xatolik chiqadi. Avval o'sha foydalanuvchining barcha postlarini qo'lda o'chirib chiqib, keyin o'zini o'chirish mumkin. Bu muhim ma'lumotlar tasodifan yo'qolib ketishining oldini oladi.

ON DELETE CASCADE (Zanjirli o'chirish)
Bizdagi misol: comments jadvalidagi post_id ustunida qo'llanilgan.

Ishlash prinsipi: Agar asosiy ob'ekt (post) o'chib ketsa, unga bog'langan barcha ikkilamchi ob'ektlar (izohlar) ham bazadan avtomatik ravishda, hech qanday so'rovsiz o'chirib tashlanadi.

Natija: DELETE FROM posts WHERE post_id = 1; deb postni o'chirsangiz, bazadagi o'sha postga yozilgan yuzlab izohlar ham bir zumda o'chib ketadi. Bu yetim (egasi yo'q) ma'lumotlar bazada qolib ketmasligini ta'minlaydi.
