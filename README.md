# goit-rdb-fp

1. Завантажте дані:

Створіть схему pandemic у базі даних за допомогою SQL-команди.
Оберіть її як схему за замовчуванням за допомогою SQL-команди.
Імпортуйте дані за допомогою Import wizard так, як ви вже робили це у темі 3.
Продивіться дані, щоб бути у контексті.
💡 Як бачите, атрибути Entity та Code постійно повторюються. Позбудьтеся цього за допомогою нормалізації даних.
CREATE SCHEMA pandemic;
USE pandemic;
SELECT * FROM infectious_cases LIMIT 10;
<img width="1450" height="655" alt="image" src="https://github.com/user-attachments/assets/8f9eff45-9d66-44a0-8842-76fa22397517" />
--  Перевірити, що Entity і Code повторюються
SELECT Entity, Code, COUNT(*) AS cnt
FROM infectious_cases
GROUP BY Entity, Code
ORDER BY cnt DESC;
<img width="933" height="568" alt="image" src="https://github.com/user-attachments/assets/8d58b66d-4f00-442e-802f-d076252683cd" />

2. Нормалізуйте таблицю infectious_cases до 3ї нормальної форми. Збережіть у цій же схемі дві таблиці з нормалізованими даними.

Виконайте запит SELECT COUNT(*) FROM infectious_cases , щоб ментор міг зрозуміти, скільки записів ви завантажили у базу даних із файла.
--  Крок 2.1. Перевірити кількість записів
SELECT COUNT(*) FROM infectious_cases;

<img width="540" height="340" alt="image" src="https://github.com/user-attachments/assets/b97408fb-9e38-413e-bb78-d2147544d15e" />
--  Крок 2.2. Створити довідник entity (уникальні Entity + Code)
CREATE TABLE entity (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255),
    code VARCHAR(50)
);
INSERT INTO entity (name, code)
SELECT DISTINCT Entity, Code FROM infectious_cases;

--  Крок 2.3. Створити нормалізовану таблицю infectious_case_data
CREATE TABLE infectious_case_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity_id INT,
    Year INT,
    Number_yaws TEXT,
    polio_cases INT,
    cases_guinea_worm INT,
    Number_rabies TEXT,
    FOREIGN KEY (entity_id) REFERENCES entity(id)
);

-- Крок 2.4. Заповнити infectious_case_data з JOIN по entity_id
INSERT INTO infectious_case_data (
    entity_id, Year, Number_yaws, polio_cases, cases_guinea_worm, Number_rabies
)
SELECT 
    e.id, 
    i.Year, 
    i.Number_yaws, 
    i.polio_cases, 
    i.cases_guinea_worm, 
    i.Number_rabies
FROM infectious_cases i
JOIN entity e ON i.Entity = e.name AND i.Code = e.code;
 -- Перевірити вставлення
SELECT COUNT(*) FROM infectious_case_data;
SELECT * FROM infectious_case_data LIMIT 10;
<img width="998" height="716" alt="image" src="https://github.com/user-attachments/assets/c5ab324a-3cfb-4a51-a0ec-f01f041182fa" />

3. Проаналізуйте дані:

Для кожної унікальної комбінації Entity та Code або їх id порахуйте середнє, мінімальне, максимальне значення та суму для атрибута Number_rabies.
💡 Врахуйте, що атрибут Number_rabies може містити порожні значення ‘’ — вам попередньо необхідно їх відфільтрувати.

Результат відсортуйте за порахованим середнім значенням у порядку спадання.
Оберіть тільки 10 рядків для виведення на екран.
-- Завдання 3: Аналіз Number_rabies
SELECT 
    e.name AS Entity,
    e.code AS Code,
    AVG(CAST(ic.Number_rabies AS UNSIGNED)) AS avg_rabies,
    MIN(CAST(ic.Number_rabies AS UNSIGNED)) AS min_rabies,
    MAX(CAST(ic.Number_rabies AS UNSIGNED)) AS max_rabies,
    SUM(CAST(ic.Number_rabies AS UNSIGNED)) AS sum_rabies
FROM infectious_case_data ic
JOIN entity e ON ic.entity_id = e.id
WHERE ic.Number_rabies != ''
GROUP BY e.name, e.code
ORDER BY avg_rabies DESC
LIMIT 10;
<img width="1026" height="585" alt="image" src="https://github.com/user-attachments/assets/1515642c-71d1-4313-920b-b2891a9a3f52" />

4. Побудуйте колонку різниці в роках.

Для оригінальної або нормованої таблиці для колонки Year побудуйте з використанням вбудованих SQL-функцій:

атрибут, що створює дату першого січня відповідного року,
💡 Наприклад, якщо атрибут містить значення ’1996’, то значення нового атрибута має бути ‘1996-01-01’.
атрибут, що дорівнює поточній даті,
атрибут, що дорівнює різниці в роках двох вищезгаданих колонок.
💡 Перераховувати всі інші атрибути, такі як Number_malaria, не потрібно.
-- Завдання 4: Дата 1 січня, поточна дата та різниця в роках
SELECT 
    Year,
    TIMESTAMPDIFF(
        YEAR,
        STR_TO_DATE(CONCAT(Year, '-01-01'), '%Y-%m-%d'),
        CURDATE()
    ) AS years_diff
FROM infectious_case_data
LIMIT 5;
<img width="584" height="415" alt="image" src="https://github.com/user-attachments/assets/e4dd70c2-3ccd-4e21-9e1d-736b2ce84c29" />
5. Побудуйте власну функцію.

Створіть і використайте функцію, що будує такий же атрибут, як і в попередньому завданні: функція має приймати на вхід значення року, а повертати різницю в роках між поточною датою та датою, створеною з атрибута року (1996 рік → ‘1996-01-01’).
💡 Якщо ви не виконали попереднє завдання, то можете побудувати іншу функцію — функцію, що рахує кількість захворювань за певний період. Для цього треба поділити кількість захворювань на рік на певне число: 12 — для отримання середньої кількості захворювань на місяць, 4 — на квартал або 2 — на півріччя. Таким чином, функція буде приймати два параметри: кількість захворювань на рік та довільний дільник. Ви також маєте використати її — запустити на даних. Оскільки не всі рядки містять число захворювань, вам необхідно буде відсіяти ті, що не мають чисельного значення (≠ ‘’).
-- Завдання 5: Створення та використання функції years_since()
-- Крок 5.1. Створити функцію
DELIMITER //

CREATE FUNCTION years_since(input_year INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE start_date DATE;
    SET start_date = STR_TO_DATE(CONCAT(input_year, '-01-01'), '%Y-%m-%d');
    RETURN TIMESTAMPDIFF(YEAR, start_date, CURDATE());
END //

DELIMITER ;

-- Крок 5.2. Використати функцію
SELECT 
    Year,
    years_since(Year) AS year_difference
FROM infectious_case_data
LIMIT 10;
<img width="1100" height="679" alt="image" src="https://github.com/user-attachments/assets/6c76d267-82f7-4969-a83c-3392aa1a15fa" />
-- Альтернативний варіант (Функція disease_per_period(disease_count TEXT, divider INT)
-- Вона ділить кількість захворювань на вказаний період (наприклад, 12 — місяців):
DELIMITER //

CREATE FUNCTION disease_per_period(disease_count TEXT, divider INT)
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE result DECIMAL(10,2);
    IF disease_count = '' OR disease_count IS NULL THEN
        RETURN NULL;
    END IF;
    SET result = CAST(disease_count AS DECIMAL(10,2)) / divider;
    RETURN result;
END //

DELIMITER ;

-- Виклик функції:
SELECT 
    Number_rabies,
    disease_per_period(Number_rabies, 12) AS rabies_per_month
FROM infectious_case_data
WHERE Number_rabies != ''
LIMIT 10;
<img width="1030" height="759" alt="image" src="https://github.com/user-attachments/assets/c93b5d9d-5919-4d8a-a71f-ebc04aa37c78" />



<img width="353" height="545" alt="image" src="https://github.com/user-attachments/assets/d5ca2a30-9da2-4e85-9902-dc54c430dfa4" />







