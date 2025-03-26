# Adidas Webstore Sales Analysis

<h2>📌Опис</h2>
Цей проєкт спрямований на аналіз даних про продажі онлайн-магазину Adidas. Основна мета – дослідити тенденції продажів, виявити популярні моделі взуття та визначити ключові фактори, що впливають на продажі

<h2>🎯Мета проєкту</h2>

<h3>Основні завдання аналізу:</h3>

**🔹 Визначити, які моделі взуття продаються найкраще**  

**🔹 Порівняти обсяги продажів між різними країнами**  

**🔹 Проаналізувати зміни обсягів продажів залежно від місяця/року**  

**🔹 Дослідити, які типи взуття продаються краще (жіноче/чоловіче)**  

**🔹 Виявити кореляцію між кольором взуття та рівнем продажів**  

<h2>📂 Структура проєкту</h2>
<h3>Проєкт складається з кількох етапів:</h3>

1. **Об'єднання таблиць**: використання LEFT JOIN для з'єднання таблиць з різними аспектами продажів взуття компанії Adidas
2. **Фільтрація даних**: відбір лише необхідних стовпців для аналізу
3. **Очищення даних**: перевірка на дублікати, пропущені значення та аномалії
4. **Групування даних**: створення загальних категорій для аналізу, таких як "Тренування", "Біг", "Теніс" тощо
5. **Аналіз і візуалізація**: використання аналітичних запитів для вивчення трендів, цінових змін і попиту на певні категорії товарів

<h2>🗂 Датасет</h2>
Проєкт базується на даних на датасеті Adidas Webstore Shoe Data з Kaggle, посилання: <br>
https://www.kaggle.com/datasets/tamsnd/adidas-webstore-shoe-data?select=shoes_dim.csv <br>

<h3>Датасет містить кілька таблиць, включаючи:</h3>

- **shoes_fact**: основна інформація про продажі, включаючи ціну, дату та країну
- **country_dim**: дані про країни, включаючи валюту
- **shoes_dim**: деталі взуття, включаючи назву, категорію, колір та інші характеристики

<h2>🛠Технології</h2> 

- **SQL**: для обробки та маніпуляцій з даними
- **PostgreSQL**: для зберігання та обробки даних
- **pgAdmin 4**: для керування базою даних PostgreSQL
- **Power BI/Tableau**: для візуалізації результатів аналізу (можна додати цей пункт, якщо плануєш використовувати візуалізацію)

<details>
  <summary><h2>🚀Перебіг роботи</h2></summary>

  **1. Об'єднання таблиць**<br>  
   На першому етапі я об'єднала три таблиці з різними аспектами продажу, використовуючи SQL-запити з **LEFT JOIN**. Це дозволило зібрати всі необхідні дані в одному запиті для подальшого аналізу.

```sql
    SELECT * 
    FROM shoes_fact
    LEFT JOIN country_dim USING (country_code)
    LEFT JOIN shoes_dim USING (id);
```

 **2. Використання Common Table Expression (CTE)**<br>  
Для зручності і ефективності я обгорнула попередній запит в Common Table Expression (CTE). Це домогло оптимізувати процес отримання потрібних даних з бази данних.

```sql
    WITH common_table AS (
        SELECT * 
        FROM shoes_fact
        LEFT JOIN country_dim USING (country_code)
        LEFT JOIN shoes_dim USING (id)
    )
    SELECT * 
    FROM common_table;
```

**3. Фільтрація необхідних стовпців для аналізу**<br>  
Щоб зробити запит більш ефективним і зручним, я розвинула попередній запит з метою збереження лише тих стовпців, які потрібні для подальшого аналізу продажів. Це дозволяє зберегти лише релевантні дані, виключивши непотрібні стовпці, такі як: **image_url**, **x**, **size**, **shoe_metric**, **availability**, **sub_color1**, **sub_color2**. Ось як виглядає оновлений SQL-запит:

```sql
WITH filtered_common_table AS (
    SELECT shoes_fact.id
          ,shoes_fact.country_code
          ,shoes_fact.price
          ,shoes_fact.category
          ,shoes_fact.date
          ,country_dim.currency
          ,shoes_dim.name
          ,shoes_dim.best_for_wear
          ,shoes_dim.gender
          ,shoes_dim.dominant_color
    FROM shoes_fact 
    LEFT JOIN country_dim USING (country_code)
    LEFT JOIN shoes_dim USING (id)
)
SELECT * 
FROM filtered_common_table;
```


**4. Очищення даних: перевірка на дублікати, пропущені значення та аномалії**<br>
Для покращення якості даних я оновила попередній CTE-запит, додавши кілька важливих етапів очистки даних. Моя мета — виключити дублікати та рядки з пропущеними значеннями.

- **Запобігання дублікатам:** Я використала INNER JOIN, щоб об'єднати таблиці таким чином, щоб дані містили тільки ті записи, які точно мають відповідні значення у всіх таблицях. Це дозволяє уникнути дублювання інформації.

- **Фільтрація порожніх значень**: Застосувала умови в WHERE-клаузі, щоб видалити всі рядки, де хоча б одне значення є порожнім (NULL). Це важливо, оскільки наявність пропущених даних може вплинути на точність аналізу.

- **Аномалії**:
Під час перевірки на наявність аномалій у даних я не виявивила жодних суттєвих відхилень або некоректних значень.
<br>Оновлений SQL-запит:
```sql
WITH filtered_common_table AS (
    SELECT DISTINCT 
        shoes_fact.id
        ,shoes_dim.name
        ,shoes_fact.price
        ,country_dim.currency
        ,shoes_fact.country_code
        ,shoes_fact.date
        ,shoes_fact.category
        ,shoes_dim.best_for_wear 
        ,shoes_dim.gender
        ,shoes_dim.dominant_color
    FROM shoes_fact
    INNER JOIN country_dim
        ON shoes_fact.country_code = country_dim.country_code
    INNER JOIN shoes_dim
        ON shoes_fact.id = shoes_dim.id
    WHERE 
        shoes_fact.id IS NOT NULL
        AND shoes_dim.name IS NOT NULL
        AND shoes_fact.price IS NOT NULL
        AND country_dim.currency IS NOT NULL
        AND shoes_fact.country_code IS NOT NULL
        AND shoes_fact.date IS NOT NULL
        AND shoes_fact.category IS NOT NULL
        AND shoes_dim.best_for_wear IS NOT NULL
        AND shoes_dim.gender IS NOT NULL
        AND shoes_dim.dominant_color IS NOT NULL
)

SELECT *
FROM filtered_common_table;
```
- **Робота з текстом**:
  
1.Видалила дефіси з назв category
  ```sql
UPDATE shoes_fact
SET category = REPLACE(category, '-', ' ');
```


2.Видалила частку us/... так як не бачу в ній практичного використання, коли є country code
  ```sql
UPDATE shoes_fact
SET category = REPLACE(category, 'us/', '');
```
3.замінила _ на пробіли
  ```sql
UPDATE filtered_common_table
SET category = REPLACE(category, '_', ' ');
```
4.була проблема з athleticsneakers. Рішення
  ```sql
UPDATE shoes_fact
SET category = REPLACE(category, 'athleticsneakers', 'athletic sneakers')
WHERE category = 'athleticsneakers';


```

видалення часточок shoes та sneakers зі значень колонки category 

```sql
UPDATE shoes_fact
SET category = REPLACE(category, ' shoes', '')
WHERE category LIKE '% shoes';
```

UPDATE shoes_fact
SET category = REPLACE(category, ' sneakers', '')
WHERE category LIKE '%sneakers';

**Трансформування даних**<br>
Зміна типу данних
Для того щоб мати можливість працювати з значенням price(робити розрахунки, тощо)
 я змінила тип данних з text на decimal

```sql
ALTER TABLE your_table_name
ALTER COLUMN price TYPE numeric USING price::numeric;
```

Перейменування колонки date
```sql
ALTER TABLE shoes_fact
RENAME COLUMN date TO order_date;
```
**5. Перейменування стовпця**<br>
Щоб зробити дані зрозумілішими, я вирішила перейменувати стовпець best_for_wear на use_purpose. Це дозволило краще відобразити значення цього стовпця, адже **use_purpose** чіткіше показує мету використання взуття, ніж попередня назва.
```sql
ALTER TABLE shoes_dim
RENAME COLUMN best_for_wear TO use_purpose;
```

**6. Групування параметрів**<br>
Для того, щоб полегшити аналіз і краще організувати дані, я вирішив згрупувати різні параметри з категорії best_for_wear у більш загальні групи. Для цього я створив новий стовпець purpose_category, де кожен параметр було перерозподілено на основі його призначення. Замість конкретних значень для кожної пари, я об'єднав їх у такі більш загальні категорії:
Для простоти розуміння з вирішив згрупувати параметри з use_purpose у більш загальні групи. Спочатку для цього створив окремий стовпчик під назвою purpose_category: 
```sql
ALTER TABLE shoes_dim
ADD COLUMN purpose_category VARCHAR(255);
```
Потім згрупував усі значення у більш загальні категорії ось так:
```sql
UPDATE shoes_dim
SET category = 'Training and Fitness'
WHERE best_for_wear IN ('Train', 'Strength', 'HIIT', 'Workout', 'Accuracy', 'Speed', 'Staying Cool', 'Staying Warm');

UPDATE shoes_dim
SET category = 'Running'
WHERE best_for_wear IN ('Run', 'Long Distance', 'Trail Run', 'Marathon');

UPDATE shoes_dim
SET category = 'Tennis and Racquet Sports'
WHERE best_for_wear IN ('Padel Tennis', 'Pickleball', 'On-Court', 'Off-Court');

UPDATE shoes_dim
SET category = 'Mountain Sports'
WHERE best_for_wear IN ('Skiing', 'Hiking & Trekking', 'Trekking', 'Day Hiking', 'All Mountain');

UPDATE shoes_dim
SET category = 'Cycling Activities'
WHERE best_for_wear IN ('Race', 'Racing', 'Flat Pedal', 'All-rounder', 'Speed');

UPDATE shoes_dim
SET category = 'Urban Conditions'
WHERE best_for_wear IN ('City', 'Stadium', 'Inside', 'Cage');

UPDATE shoes_dim
SET category = 'Outdoor / Nature'
WHERE best_for_wear IN ('Outside', 'Natural', 'Terrain', 'Dirt / Casual', 'Rugged Terrain', 'Everyday', 'Walking');

UPDATE shoes_dim
SET category = 'Triathlon'
WHERE best_for_wear = 'Triathlon';

UPDATE shoes_dim
SET category = 'Speed and Dynamics'
WHERE best_for_wear = 'Fast';

UPDATE shoes_dim
SET category = 'General'
WHERE best_for_wear IN ('Comfort', 'Agility', 'Gravity', 'Stability', 'Control', 'Neutral', 'Accuracy');
```
**Питання до данних**<br>
10 моделей взуття, які продаються найкраще?
  ```sql
SELECT name, SUM(price) AS total_sales
FROM filtered_common_table
GROUP BY name
ORDER BY total_sales DESC
LIMIT 10;
```
Продажі по країнам: 
 ```sql
SELECT 
	CASE country_code 
	WHEN 'DE' THEN 'Germany' 
	WHEN 'UK' THEN 'United Kingdom'
	WHEN 'BE' THEN 'Belgium'
	WHEN 'US' THEN 'United States'
	ELSE 'Undifiended' END AS country,
	ROUND(SUM(price),1) AS total_sales
	,ROUND(100.0 * SUM(price) / SUM(SUM(price)) OVER (), 2) AS sales_percentage

FROM filtered_common_table
GROUP BY country,currency
ORDER BY total_sales DESC;
```

Продажі по дням (датасет зовсім новий, є лише дані за січень 2025-го року)
 ```sql
SELECT 
    date,
    ROUND(SUM(price),2) AS total_sales
FROM filtered_common_table
GROUP BY date
ORDER BY date;
```
Проценти з загальної кількості продажів по країнам
 ```sql
SELECT 
    date,
    ROUND(SUM(price), 2) AS total_sales,
    ROUND(100.0 * SUM(CASE WHEN country_code = 'DE' THEN price ELSE 0 END) / SUM(price), 2) AS "DE",
    ROUND(100.0 * SUM(CASE WHEN country_code = 'UK' THEN price ELSE 0 END) / SUM(price), 2) AS "UK",
    ROUND(100.0 * SUM(CASE WHEN country_code = 'BE' THEN price ELSE 0 END) / SUM(price), 2) AS "BE",
	ROUND(100.0 * SUM(CASE WHEN country_code = 'US' THEN price ELSE 0 END) / SUM(price),  2) AS "US"
FROM filtered_common_table
GROUP BY date
ORDER BY date;
```
Показати обсяг продажів для кожної категорії взуття:
 ```sql
SELECT 
    category AS "Тип взуття", 
    ROUND(SUM(price), 2) AS "Total Sales", 
    COUNT(id) AS "Number of Transactions"
FROM filtered_common_table
GROUP BY category
ORDER BY "Total Sales" DESC;
```
додати розподіл за статтю (gender), щоб побачити різницю між продажами для чоловіків і жінок:
 ```sql
SELECT 
    category AS "Тип взуття", 
    gender AS "Стать",
    ROUND(SUM(price), 2) AS "Total Sales", 
    COUNT(id) AS "Number of Transactions"
FROM filtered_common_table
GROUP BY category, gender
ORDER BY "Total Sales" DESC;
```
кількість проданих пар взуття по певній категорії та кількістю куплених моделей в кожному кольорі
 ```sql
SELECT 
    category , 
    dominant_color,
    ROUND(SUM(price), 2) AS "Total Sales",
    COUNT(id) AS "Number of Transactions"
FROM filtered_common_table
WHERE category='running-shoes'
GROUP BY category, dominant_color
ORDER BY "Total Sales" DESC;
```

кількість проданих пар взуття в загальному для кожного з кольорів
 ```sql
SELECT 
    dominant_color AS "Основний колір",
    ROUND(SUM(price), 2) AS "Total Sales",
    COUNT(id) AS "Number of Transactions"
FROM filtered_common_table
GROUP BY dominant_color
ORDER BY "Total Sales" DESC;
```

</details>

<h2>Результати роботи</h2>

