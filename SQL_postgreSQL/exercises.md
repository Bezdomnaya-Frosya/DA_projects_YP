*Найдите количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».*\
SELECT COUNT(id)\
FROM stackoverflow.posts\
WHERE (score > 300 OR favorites_count >= 100) AND post_type_id = 1
____________________________________________________________
*Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 включительно? Результат округлите до целого числа.*\
WITH i AS\
        (SELECT CAST(DATE_TRUNC('day', posts.creation_date) AS date) as date,\
                COUNT(posts.id)\
         FROM stackoverflow.posts\
         WHERE post_type_id = 1\
         GROUP BY CAST(DATE_TRUNC('day', posts.creation_date) AS date)\
         HAVING CAST(DATE_TRUNC('day', posts.creation_date) AS date) BETWEEN '2008-11-01' AND '2008-11-18')\
\
SELECT ROUND(AVG(count))\
FROM i\
_____________________________________________________________
*Сколько пользователей получили значки сразу в день регистрации? Выведите количество уникальных пользователей.*\
SELECT COUNT(DISTINCT(users.id))\
FROM stackoverflow.users\
JOIN stackoverflow.badges ON users.id = badges.user_id\
WHERE CAST(DATE_TRUNC('day', users.creation_date) AS date) = CAST(DATE_TRUNC('day', badges.creation_date) AS date)\
_____________________________________________________________
*Сколько уникальных постов пользователя с именем Joel Coehoorn получили хотя бы один голос?*\
SELECT COUNT(DISTINCT(posts.id))\
FROM stackoverflow.posts\
JOIN stackoverflow.users ON posts.user_id=users.id\
RIGHT JOIN stackoverflow.votes ON votes.post_id=posts.id\
WHERE users.display_name LIKE 'Joel Coehoorn'\
_______________________________________________________________
*Выгрузите все поля таблицы vote_types. Добавьте к таблице поле rank, в которое войдут номера записей в обратном порядке. Таблица должна быть отсортирована по полю id*\
SELECT *,\
       ROW_NUMBER() OVER (ORDER BY id DESC) AS rank\
FROM stackoverflow.vote_types\
ORDER BY id\
__________________________________________________________________
*Отберите 10 пользователей, которые поставили больше всего голосов типа Close. Отобразите таблицу из двух полей: идентификатором пользователя и количеством голосов. Отсортируйте данные сначала по убыванию количества голосов, потом по убыванию значения идентификатора пользователя.*\
SELECT v.user_id, \
       COUNT(v.id)\
FROM stackoverflow.votes v\
WHERE vote_type_id = 6\
GROUP BY v.user_id\
ORDER BY COUNT(v.id) DESC, v.user_id DESC\
LIMIT 10\
_______________________________________________________________________
*Отберите 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно.
Отобразите несколько полей:
идентификатор пользователя;
число значков;
место в рейтинге — чем больше значков, тем выше рейтинг.
Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге.
Отсортируйте записи по количеству значков по убыванию, а затем по возрастанию значения идентификатора пользователя.*\
SELECT b.user_id, \
       COUNT(b.id),\
       DENSE_RANK() OVER (ORDER BY COUNT(b.id) DESC)\
FROM stackoverflow.badges b\
WHERE CAST(creation_date AS date) BETWEEN '2008-11-15' AND '2008-12-15'\
GROUP BY b.user_id\
ORDER BY COUNT(b.id) DESC, b.user_id\
LIMIT 10\
________________________________________________________________________
*Сколько в среднем очков получает пост каждого пользователя?
Сформируйте таблицу из следующих полей:
заголовок поста;
идентификатор пользователя;
число очков поста;
среднее число очков пользователя за пост, округлённое до целого числа.
Не учитывайте посты без заголовка, а также те, что набрали ноль очков.*\
SELECT p.title, \
        p.user_id, \
        p.score,\
        ROUND(AVG(p.score) OVER (PARTITION BY p.user_id))\
FROM stackoverflow.posts p\
WHERE p.score <> 0 AND p.title IS NOT NULL\
___________________________________________________________________________
*Отобразите заголовки постов, которые были написаны пользователями, получившими более 1000 значков. Посты без заголовков не должны попасть в список.*\
SELECT p.title\
FROM stackoverflow.posts p\
WHERE p.user_id IN\
                    (SELECT user_id FROM stackoverflow.badges GROUP BY user_id HAVING COUNT(id)>1000)\
AND p.title IS NOT NULL\
__________________________________________________________________________
*Напишите запрос, который выгрузит данные о пользователях из Канады (англ. Canada). Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
пользователям с числом просмотров меньше 100 — группу 3.
Отобразите в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу. Пользователи с нулевым количеством просмотров не должны войти в итоговую таблицу.*\
SELECT id,\ 
        views,\
        CASE\
            WHEN views <100 THEN 3\
            WHEN views <350 THEN 2\
            WHEN views>=350 THEN 1\
         END\
FROM stackoverflow.users\
WHERE location LIKE '%Canada%' AND views > 0\
______________________________________________________
*Дополните предыдущий запрос. Отобразите лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе. Выведите поля с идентификатором пользователя, группой и количеством просмотров. Отсортируйте таблицу по убыванию просмотров, а затем по возрастанию значения идентификатора.*\
WITH \
view_groups AS (SELECT id,\
                            views,\
                                 CASE\
                                     WHEN views >= 350 THEN 1\
                                     WHEN views >= 100 THEN 2\
                                     ELSE 3\
                                  END as group_num\
                     FROM stackoverflow.users\
                     WHERE location LIKE '%Canada%' AND views > 0)\
SELECT vg.id,\
       vg.group_num,\
       vg.views\
FROM view_groups vg\
JOIN \
(SELECT group_num,\
        MAX(views) as max_views\
FROM view_groups\
GROUP BY group_num\
) max_views ON vg.group_num = max_views.group_num AND vg.views = max_views.max_views\
ORDER BY vg.views DESC, vg.id ASC;\
______________________________________________________________
*Для каждого пользователя, который написал хотя бы один пост, найдите интервал между регистрацией и временем создания первого поста. Отобразите:
идентификатор пользователя;
разницу во времени между регистрацией и первым постом.*\
WITH p AS \
(SELECT user_id, creation_date,\
RANK() OVER (PARTITION BY user_id ORDER BY creation_date)  AS first_pub\
FROM stackoverflow.posts \

ORDER BY user_id)\

SELECT user_id, p.creation_date - u.creation_date FROM p\
JOIN stackoverflow.users u ON p.user_id = u.id\
WHERE first_pub = 1 \
_________________________________________________________________
