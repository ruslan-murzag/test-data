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
