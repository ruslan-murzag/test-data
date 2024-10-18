# test-data

Задание 1: Bash-скрипт для работы с таблицей событий подписок

Описание задачи:
	1.	Пересоздавать таблицу events, которая хранит информацию о подписках пользователей.
	2.	Выгружать данные таблицы в CSV-файл.
	3.	Выполнять вставку данных из CSV-файла в таблицу.

1. Скрипт для пересоздания таблицы, экспорта данных в CSV и импорта обратно

Вот bash-скрипт, который реализует пересоздание таблицы, экспорт в CSV и импорт из CSV:

```
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
