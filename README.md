# test-data

## Задание 1: Bash-скрипт для работы с таблицей событий подписок

Описание задачи:
	1.	Пересоздавать таблицу events, которая хранит информацию о подписках пользователей.
	2.	Выгружать данные таблицы в CSV-файл.
	3.	Выполнять вставку данных из CSV-файла в таблицу.

1. Скрипт для пересоздания таблицы, экспорта данных в CSV и импорта обратно

Вот bash-скрипт, который реализует пересоздание таблицы, экспорт в CSV и импорт из CSV:

```bash
#!/bin/bash

# Настройки базы данных
DB_NAME="testdb"
DB_USER="testuser"
DB_PASS="testpassword"
DB_HOST="localhost"
DB_PORT="5432"
TABLE_NAME="events"
CSV_FILE="events_data.csv"

# Функция для пересоздания таблицы
recreate_table() {
    echo "Пересоздание таблицы $TABLE_NAME..."

    PGPASSWORD=$DB_PASS psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME -c "
    DROP TABLE IF EXISTS $TABLE_NAME;
    CREATE TABLE $TABLE_NAME (
        user_id INT,
        product_identifier VARCHAR(255),
        start_time TIMESTAMP,
        end_time TIMESTAMP,
        price_in_usd NUMERIC
    );"
    
    if [ $? -eq 0 ]; then
        echo "Таблица успешно пересоздана."
    else
        echo "Ошибка при пересоздании таблицы."
        exit 1
    fi
}

# Функция для вставки данных в таблицу (пример данных)
insert_data() {
    echo "Вставка тестовых данных в таблицу $TABLE_NAME..."

    PGPASSWORD=$DB_PASS psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME -c "
    INSERT INTO $TABLE_NAME (user_id, product_identifier, start_time, end_time, price_in_usd)
    VALUES 
    (1, 'prod_1', '2023-01-01 10:00:00', '2023-12-31 23:59:59', 9.99),
    (2, 'prod_2', '2023-02-01 11:00:00', '2023-12-31 23:59:59', 19.99),
    (3, 'prod_3', '2023-03-01 12:00:00', '2023-12-31 23:59:59', 29.99);
    "
    
    if [ $? -eq 0 ]; then
        echo "Тестовые данные успешно вставлены."
    else
        echo "Ошибка при вставке данных."
        exit 1
    fi
}

# Функция для выгрузки данных в CSV-файл
export_to_csv() {
    echo "Выгрузка данных в CSV-файл..."

    PGPASSWORD=$DB_PASS psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME -c "\copy $TABLE_NAME TO '$CSV_FILE' CSV HEADER"
    
    if [ $? -eq 0 ]; then
        echo "Данные успешно выгружены в $CSV_FILE."
    else
        echo "Ошибка при выгрузке данных."
        exit 1
    fi
}

# Функция для вставки данных из CSV-файла
insert_from_csv() {
    echo "Вставка данных из CSV-файла в таблицу..."

    PGPASSWORD=$DB_PASS psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME -c "\copy $TABLE_NAME FROM '$CSV_FILE' CSV HEADER"
    
    if [ $? -eq 0 ]; then
        echo "Данные успешно вставлены в таблицу $TABLE_NAME."
    else
        echo "Ошибка при вставке данных."
        exit 1
    fi
}

# Основной процесс
recreate_table
insert_data
export_to_csv
insert_from_csv
```


## Задание 2: Работа с таблицами операций депозитов и выплат

Описание задачи:
	1.	Создать две таблицы deposits и withdrawals, содержащие данные о депозитах и снятиях средств пользователей.
	2.	Вставить тестовые данные в эти таблицы.
	3.	Создать материализованное представление, которое будет агрегировать суммы депозитов и снятий для каждого пользователя по дням.
	4.	Объяснить и предложить методы обновления материализованного представления.

1. Создание таблиц

Для создания таблиц deposits и withdrawals в PostgreSQL необходимо выполнить следующий SQL-код:

```sql
-- Создаем таблицу deposits
CREATE TABLE deposits (
    id INT PRIMARY KEY,
    user_id INT NOT NULL,
    amount FLOAT NOT NULL,
    created_at TIMESTAMP NOT NULL
);

-- Создаем таблицу withdrawals
CREATE TABLE withdrawals (
    id INT PRIMARY KEY,
    user_id INT NOT NULL,
    amount FLOAT NOT NULL,
    created_at TIMESTAMP NOT NULL
);
```

2. Вставка данных


```sql
-- Вставляем данные в таблицу deposits
INSERT INTO deposits (id, user_id, amount, created_at) VALUES
(1, 1, 100.0, '2023-01-01 10:00:00'),
(2, 1, 150.0, '2023-01-02 12:30:00'),
(3, 2, 200.0, '2023-01-01 11:00:00'),
(4, 3, 300.5, '2023-01-03 15:45:00');

-- Вставляем данные в таблицу withdrawals
INSERT INTO withdrawals (id, user_id, amount, created_at) VALUES
(1, 1, 50.0, '2023-01-01 14:00:00'),
(2, 2, 100.0, '2023-01-02 10:00:00'),
(3, 3, 150.5, '2023-01-04 17:15:00');
```

3. Создание Materialized View

```sql
CREATE MATERIALIZED VIEW daily_financial_summary AS
SELECT
    d.user_id,
    DATE(d.created_at) AS operation_day,
    COALESCE(SUM(d.amount), 0) AS total_deposits,
    COALESCE(SUM(w.amount), 0) AS total_withdrawals,
    (COALESCE(SUM(d.amount), 0) - COALESCE(SUM(w.amount), 0)) AS balance_difference
FROM
    deposits d
LEFT JOIN
    withdrawals w ON d.user_id = w.user_id AND DATE(d.created_at) = DATE(w.created_at)
GROUP BY
    d.user_id, DATE(d.created_at);
```

4. Обновление Materialized View

```sql
REFRESH MATERIALIZED VIEW daily_financial_summary;
```

## Задание 3:

```bash
#!/bin/bash

# Параметры подключения к базе данных
DB_NAME="your_db_name"
DB_USER="your_db_user"
DB_PASS="your_db_password"
DB_HOST="localhost"
DB_PORT="5432"
CSV_FILE="diagnostics.csv"

# Шаг 1: Создание таблицы station_errors, если она еще не существует
PGPASSWORD=$DB_PASS psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME <<EOF
CREATE TABLE IF NOT EXISTS station_errors (
    id INT PRIMARY KEY,
    date DATE NOT NULL,
    station VARCHAR(50) NOT NULL,
    msg VARCHAR(255) NOT NULL,
    status VARCHAR(50)
);
EOF

# Шаг 2: Создание временной таблицы для загрузки данных из CSV
PGPASSWORD=$DB_PASS psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME <<EOF
DROP TABLE IF EXISTS temp_station_errors;
CREATE TABLE temp_station_errors (
    id INT,
    date DATE,
    station VARCHAR(50),
    msg VARCHAR(255)
);
\copy temp_station_errors(id, date, station, msg) FROM '$CSV_FILE' WITH CSV HEADER;
EOF

# Шаг 3: Перенос данных из временной таблицы в таблицу station_errors, игнорируя "not finished"
PGPASSWORD=$DB_PASS psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME <<EOF
INSERT INTO station_errors (id, date, station, msg, status)
SELECT id, date, station, msg, NULL
FROM temp_station_errors
WHERE msg != 'not finished';
EOF

# Шаг 4: Присвоение статусов ошибкам (new, serious, critical)
PGPASSWORD=$DB_PASS psql -h $DB_HOST -p $DB_PORT -U $DB_USER -d $DB_NAME <<EOF
WITH ranked_errors AS (
    SELECT 
        id, 
        date, 
        station, 
        msg,
        LAG(date) OVER (PARTITION BY station ORDER BY date) AS prev_date,
        ROW_NUMBER() OVER (PARTITION BY station ORDER BY date) AS error_rank
    FROM 
        station_errors
)
UPDATE station_errors se
SET status = CASE
    -- Если не было предыдущей ошибки или прошло больше 1 дня - статус new
    WHEN ranked_errors.prev_date IS NULL OR ranked_errors.date - ranked_errors.prev_date > 1 THEN 'new'
    -- Если ошибка второй день подряд - serious
    WHEN ranked_errors.date - ranked_errors.prev_date = 1 AND ranked_errors.error_rank = 2 THEN 'serious'
    -- Если ошибка 3 и более дней подряд - critical
    WHEN ranked_errors.date - ranked_errors.prev_date = 1 AND ranked_errors.error_rank >= 3 THEN 'critical'
    ELSE 'new'
END
FROM ranked_errors
WHERE se.id = ranked_errors.id;
EOF

echo "Данные успешно загружены и статусы присвоены."
```
