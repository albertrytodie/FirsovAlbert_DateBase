# Лабораторные работы по базам данных, выполнил Фирсов Альберт

## Лабораторная работа №1

### Задание:
Учет товаров на складах и их потребности на торговых точках.

Имеются:
- **Товары**: регистрационный номер, наименование, единица измерения, стоимость единицы
- **Склады**: номер, ФИО кладовщика  
- **Торговые точки**: наименование, адрес

Товары поступают на склады в определенном количестве и в определенную дату. Товары запрашиваются в определенную торговую точку в определенном количестве.

**Выходные документы:**
1. Выдать список товаров на каждом складе, отсортированный по наименованиям товаров с подсчетом стоимости каждого товара
2. Для заданной торговой точки выдать список запрашиваемых товаров с указанием их количества, упорядоченный по наименованиям товаров и по номерам складов

### ER-диаграмма:
![ER-диаграмма базы данных](https://github.com/user-attachments/assets/ced39e89-9708-476c-9d09-ddd1f8070d20)

### Обоснование нормализации:

#### 1НФ (атомарность значений)
Все значения в таблицах атомарны. Нет составных полей или массивов.

#### 2НФ (нет частичной зависимости от составного ключа)
- В таблицах `admissions` и `requests` все неключевые атрибуты (`admission_date`, `quantity`) полностью зависят от целого первичного ключа (`admission_id`, `request_id`)
- В таблице `inventory` неключевые атрибуты зависят от всей пары (`warehouse_id`, `product_id`)

#### 3НФ (нет транзитивных зависимостей)
- В таблице `products` атрибуты `product_name`, `measure_unit`, `unit_cost` зависят напрямую от `product_id`
- В других таблицах неключевые поля ссылаются только на первичный ключ своей таблицы (через foreign key)
## Лабораторная работа №2
## 1. Логическая модель базы данных
### 1.1 Сущности и атрибуты

**ТОВАР (PRODUCT)**
- ID товара (PK, автоинкремент)
- Наименование товара (обязательно)
- Единица измерения (обязательно)
- Стоимость единицы (≥ 0)

**СКЛАД (WAREHOUSE)**
- ID склада (PK, автоинкремент)
- ФИО кладовщика (обязательно)

**МАГАЗИН (SHOP)**
- ID магазина (PK, автоинкремент)
- Наименование магазина (обязательно)
- Адрес (обязательно)

**ПОСТУПЛЕНИЕ (ADMISSION)**
- ID поступления (PK, автоинкремент)
- Дата поступления (обязательно)
- ID товара (FK → PRODUCT)
- ID склада (FK → WAREHOUSE)
- Количество (> 0)

**ЗАПРОС (REQUEST)**
- ID запроса (PK, автоинкремент)
- Дата запроса (обязательно)
- ID товара (FK → PRODUCT)
- ID склада (FK → WAREHOUSE)
- ID магазина (FK → SHOP)
- Количество (> 0)
- Статус (pending/shipped/completed)

**ОСТАТОК (INVENTORY)**
- ID записи остатка (PK, автоинкремент)
- ID склада (FK → WAREHOUSE)
- ID товара (FK → PRODUCT)
- Количество (≥ 0)
- Уникальный ключ: (склад, товар)

### 1.2 Связи между сущностями
- Товар → Поступление (1:M)
- Склад → Поступление (1:M)
- Товар → Запрос (1:M)
- Склад → Запрос (1:M)
- Магазин → Запрос (1:M)
- Товар → Остаток (1:M)
- Склад → Остаток (1:M)

### 2. DDL-скрипты создания таблиц
```
-- Установка схемы по умолчанию
SET search_path TO firsov2272;

-- Таблица товаров
CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    measure_unit VARCHAR(20) NOT NULL,
    unit_cost NUMERIC(10,2) NOT NULL CHECK (unit_cost >= 0)
);

-- Таблица складов
CREATE TABLE warehouses (
    warehouse_id SERIAL PRIMARY KEY,
    manager_name VARCHAR(100) NOT NULL
);

-- Таблица магазинов
CREATE TABLE shops (
    shop_id SERIAL PRIMARY KEY,
    shop_name VARCHAR(100) NOT NULL,
    address VARCHAR(255) NOT NULL
);

-- Таблица поступлений
CREATE TABLE admissions (
    admission_id SERIAL PRIMARY KEY,
    admission_date DATE NOT NULL DEFAULT CURRENT_DATE,
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    warehouse_id INTEGER NOT NULL REFERENCES warehouses(warehouse_id),
    quantity INTEGER NOT NULL CHECK (quantity > 0)
);

-- Таблица запросов
CREATE TABLE requests (
    request_id SERIAL PRIMARY KEY,
    request_date DATE NOT NULL DEFAULT CURRENT_DATE,
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    warehouse_id INTEGER NOT NULL REFERENCES warehouses(warehouse_id),
    shop_id INTEGER NOT NULL REFERENCES shops(shop_id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    status VARCHAR(20) DEFAULT 'pending' 
        CHECK (status IN ('pending', 'shipped', 'completed'))
);

-- Таблица остатков
CREATE TABLE inventory (
    inventory_id SERIAL PRIMARY KEY,
    warehouse_id INTEGER NOT NULL REFERENCES warehouses(warehouse_id),
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    quantity INTEGER NOT NULL DEFAULT 0 CHECK (quantity >= 0),
    UNIQUE(warehouse_id, product_id)
);
```
### 3. Заполнение таблиц данными
```
-- Очистка таблиц перед заполнением
TRUNCATE TABLE admissions, inventory, requests, shops, warehouses, products CASCADE;

-- Заполнение таблицы товаров
INSERT INTO products (product_name, measure_unit, unit_cost) VALUES
('Молоко "Простоквашино"', 'литр', 85.50),
('Хлеб "Бородинский"', 'шт', 45.00),
('Яблоки "Голден"', 'кг', 120.00),
('Сахар песок', 'кг', 75.00),
('Масло сливочное', 'кг', 450.00),
('Кофе молотый', 'кг', 800.00);

-- Заполнение таблицы складов
INSERT INTO warehouses (manager_name) VALUES
('Иванов Александр Сергеевич'),
('Петрова Мария Константиновна'),
('Сидоров Владимир Викторович'),
('Кузнецова Ольга Петровна');

-- Заполнение таблицы магазинов
INSERT INTO shops (shop_name, address) VALUES
('Продукты №1', 'г. Москва, ул. Ленина, 10'),
('Супермаркет "Весна"', 'г. Москва, пр. Мира, 25'),
('Магазин "У дома"', 'г. Москва, ул. Садовая, 5'),
('Мини-маркет "Уют"', 'г. Москва, ул. Центральная, 17');

-- Заполнение таблицы поступлений
INSERT INTO admissions (admission_date, product_id, warehouse_id, quantity) VALUES
('2024-01-10', 1, 1, 100),
('2024-01-11', 2, 1, 200),
('2024-01-12', 3, 2, 150),
('2024-01-13', 4, 2, 80),
('2024-01-14', 5, 3, 50),
('2024-01-15', 6, 4, 30);

-- Заполнение таблицы запросов
INSERT INTO requests (request_date, product_id, warehouse_id, shop_id, quantity, status) VALUES
('2024-01-16', 1, 1, 1, 20, 'completed'),
('2024-01-16', 2, 1, 1, 30, 'completed'),
('2024-01-17', 3, 2, 2, 25, 'shipped'),
('2024-01-17', 4, 2, 2, 15, 'pending'),
('2024-01-18', 5, 3, 3, 10, 'completed'),
('2024-01-18', 6, 4, 4, 5, 'pending');

-- Инициализация остатков на основе поступлений
INSERT INTO inventory (warehouse_id, product_id, quantity)
SELECT warehouse_id, product_id, SUM(quantity)
FROM admissions
GROUP BY warehouse_id, product_id;

-- Корректировка остатков по выполненным запросам
UPDATE inventory i
SET quantity = i.quantity - r.quantity
FROM requests r
WHERE i.warehouse_id = r.warehouse_id 
  AND i.product_id = r.product_id 
  AND r.status = 'completed';
  ```
### 4. Выполнение содержательных SELECT-запросов с JOIN 2-3 таблиц
4.1 Запрос 1: Список товаров на каждом складе с подсчетом стоимости
Выходной документ 1: "выдать список товаров на каждом складе, отсортированный по наименованиям товаров с подсчетом стоимости каждого товара"
```
  SELECT 
    w.warehouse_id AS "Номер склада",
    w.manager_name AS "Кладовщик",
    p.product_name AS "Наименование товара",
    i.quantity AS "Количество",
    p.measure_unit AS "Единица измерения",
    p.unit_cost AS "Стоимость единицы",
    ROUND((i.quantity * p.unit_cost), 2) AS "Общая стоимость"
FROM inventory i
JOIN warehouses w ON i.warehouse_id = w.warehouse_id
JOIN products p ON i.product_id = p.product_id
WHERE i.quantity > 0
ORDER BY w.warehouse_id, p.product_name;
```
Результат:
<img width="985" height="201" alt="image" src="https://github.com/user-attachments/assets/9f223afa-8481-47ab-aa63-8b00f62c6a1c" />
4.2 Запрос 2: Список запрашиваемых товаров для заданной торговой точки
Выходной документ 2: "для заданной торговой точки выдать список запрашиваемых товаров с указанием их количества, упорядоченный по наименованиям товаров и по номерам складов"
```
SELECT 
    s.shop_name AS "Торговая точка",
    r.request_date AS "Дата запроса",
    p.product_name AS "Наименование товара",
    r.quantity AS "Запрашиваемое количество",
    p.measure_unit AS "Единица измерения",
    w.warehouse_id AS "Номер склада",
    r.status AS "Статус запроса"
FROM requests r
JOIN shops s ON r.shop_id = s.shop_id
JOIN products p ON r.product_id = p.product_id
JOIN warehouses w ON r.warehouse_id = w.warehouse_id
WHERE s.shop_id = 1  -- ID торговой точки "Продукты №1"
ORDER BY p.product_name, w.warehouse_id;
```
Результат:
<img width="1100" height="98" alt="image" src="https://github.com/user-attachments/assets/770a708d-af7c-4a4f-8ff4-e90db1bdffae" />
### Лабораторная работа №3
Представление для первого выходного документа
```
CREATE OR REPLACE VIEW firsov2272.warehouse_stock_report AS
SELECT 
    w.warehouse_id AS "Номер склада",
    w.manager_name AS "Кладовщик",
    p.product_name AS "Наименование товара",
    i.quantity AS "Количество",
    p.measure_unit AS "Единица измерения",
    p.unit_cost AS "Стоимость единицы",
    ROUND((i.quantity * p.unit_cost), 2) AS "Общая стоимость"
FROM firsov2272.inventory i
JOIN firsov2272.warehouses w ON i.warehouse_id = w.warehouse_id
JOIN firsov2272.products p ON i.product_id = p.product_id
WHERE i.quantity > 0
ORDER BY w.warehouse_id, p.product_name;
```
Проверка
```
SELECT * FROM firsov2272.warehouse_stock_report;
```
Результат
<img width="1087" height="197" alt="pred1" src="https://github.com/user-attachments/assets/1560742c-82b4-4ecc-a0bb-04d6cd64fec5" />

Функция для этого представления
```
CREATE OR REPLACE FUNCTION firsov2272.get_warehouse_stock_func(
    p_warehouse_id INTEGER DEFAULT NULL,
    p_min_quantity INTEGER DEFAULT 0,
    p_min_total_cost NUMERIC DEFAULT 0
)
RETURNS TABLE(
    warehouse_number INTEGER,
    warehouse_manager VARCHAR,
    product_name VARCHAR,
    quantity INTEGER,
    measure_unit VARCHAR,
    unit_cost NUMERIC,
    total_value NUMERIC
)
LANGUAGE SQL
AS $$
    SELECT 
        "Номер склада",
        "Кладовщик", 
        "Наименование товара",
        "Количество",
        "Единица измерения",
        "Стоимость единицы",
        "Общая стоимость"
    FROM firsov2272.warehouse_stock_report
    WHERE (p_warehouse_id IS NULL OR "Номер склада" = p_warehouse_id)
      AND "Количество" >= p_min_quantity
      AND "Общая стоимость" >= p_min_total_cost
    ORDER BY "Номер склада", "Наименование товара";
$$;
```
Вывод товаров только на первом складе
```
SELECT * FROM firsov2272.get_warehouse_stock_func(1);
```
<img width="937" height="113" alt="image" src="https://github.com/user-attachments/assets/62663d13-30b8-45c4-a324-842581b39f74" />

Вывод товаров дороже 5000 на втором складе
```
SELECT * FROM firsov2272.get_warehouse_stock_func(2, 0, 5000);
```
<img width="919" height="105" alt="image" src="https://github.com/user-attachments/assets/08467041-3942-4eb0-85f7-3afcf5d2cf51" />

### Лабораторная работа №4
## 1. Создание генератора данных
```
SET search_path TO firsov2272;

TRUNCATE TABLE requests, inventory, products, warehouses, shops RESTART IDENTITY CASCADE;

INSERT INTO products (product_name, measure_unit, unit_cost)
SELECT 'Товар_'||i, 'шт', (i*5)::numeric(10,2)
FROM generate_series(1, 100) i;

INSERT INTO warehouses (manager_name)
SELECT 'Склад_'||i FROM generate_series(1, 20) i;

INSERT INTO shops (shop_name, address)
SELECT 'Магазин_'||i, 'Адрес_'||i FROM generate_series(1, 30) i;

WITH combinations AS (
    SELECT 
        (i % 20) + 1 as warehouse_id,
        (i % 100) + 1 as product_id,
        (i * 7) % 1000 as quantity
    FROM generate_series(1, 500) i
)
INSERT INTO inventory (warehouse_id, product_id, quantity)
SELECT DISTINCT warehouse_id, product_id, quantity
FROM combinations
ON CONFLICT (warehouse_id, product_id) DO NOTHING;

INSERT INTO requests (request_date, product_id, warehouse_id, shop_id, quantity, status)
SELECT 
    CURRENT_DATE - (i % 30),
    (i % 100) + 1,
    (i % 20) + 1,
    (i % 30) + 1,
    (i % 10) + 1,
    CASE (i % 3)
        WHEN 0 THEN 'pending'
        WHEN 1 THEN 'shipped'
        ELSE 'completed'
    END
FROM generate_series(1, 1000) i;
```
Описание:
Создал 100 товаров, 20 складов, 30 магазинов. Создал остатки на складах и 1000 запросов товаров
Проверка данных:
```
SELECT 
    'products' as "Таблица", 
    COUNT(*) as "Записей",
    MIN(product_id) as "Min ID", 
    MAX(product_id) as "Max ID"
FROM products
UNION ALL
SELECT 'warehouses', COUNT(*), MIN(warehouse_id), MAX(warehouse_id) FROM warehouses
UNION ALL
SELECT 'shops', COUNT(*), MIN(shop_id), MAX(shop_id) FROM shops
UNION ALL
SELECT 'inventory', COUNT(*), MIN(inventory_id), MAX(inventory_id) FROM inventory
UNION ALL
SELECT 'requests', COUNT(*), MIN(request_id), MAX(request_id) FROM requests;

SELECT 
    status as "Статус",
    COUNT(*) as "Количество",
    ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 1) as "Процент"
FROM requests
GROUP BY status;
-- Примеры товаров на складах
SELECT 
    w.manager_name as "Склад",
    p.product_name as "Товар",
    i.quantity as "Количество",
    p.unit_cost as "Цена",
    (i.quantity * p.unit_cost) as "Стоимость"
FROM inventory i
JOIN warehouses w ON i.warehouse_id = w.warehouse_id
JOIN products p ON i.product_id = p.product_id
WHERE i.quantity > 0
ORDER BY i.quantity DESC
LIMIT 10;
```
Результат проверки
<img width="553" height="247" alt="image" src="https://github.com/user-attachments/assets/bb347a30-0816-4f07-b060-5d17de3c3f54" />


## 2. Анализ планов выполнения запросов 
```
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT 
    w.warehouse_id,
    w.manager_name,
    p.product_name,
    COUNT(r.request_id) as request_count,
    SUM(r.quantity) as total_requested,
    AVG(p.unit_cost) as avg_price,
    SUM(i.quantity) as current_stock,
    SUM(i.quantity * p.unit_cost) as stock_value
FROM inventory i
JOIN warehouses w ON i.warehouse_id = w.warehouse_id
JOIN products p ON i.product_id = p.product_id
LEFT JOIN requests r ON i.product_id = r.product_id 
    AND i.warehouse_id = r.warehouse_id
    AND r.request_date >= CURRENT_DATE - INTERVAL '90 days'
WHERE i.quantity > 10
    AND p.unit_cost BETWEEN 100 AND 500
    AND w.warehouse_id IN (1, 2, 3, 4, 5)
GROUP BY w.warehouse_id, w.manager_name, p.product_name
HAVING COUNT(r.request_id) > 0
    AND SUM(i.quantity) > 100
ORDER BY stock_value DESC, request_count DESC
LIMIT 20;
```
```
"Limit  (cost=50.72..50.73 rows=1 width=323) (actual time=3.345..3.355 rows=20 loops=1)"
"  Output: w.warehouse_id, w.manager_name, p.product_name, (count(r.request_id)), (sum(r.quantity)), (avg(p.unit_cost)), (sum(i.quantity)), (sum(((i.quantity)::numeric * p.unit_cost)))"
"  Buffers: shared hit=510"
"  ->  Sort  (cost=50.72..50.73 rows=1 width=323) (actual time=3.344..3.350 rows=20 loops=1)"
"        Output: w.warehouse_id, w.manager_name, p.product_name, (count(r.request_id)), (sum(r.quantity)), (avg(p.unit_cost)), (sum(i.quantity)), (sum(((i.quantity)::numeric * p.unit_cost)))"
"        Sort Key: (sum(((i.quantity)::numeric * p.unit_cost))) DESC, (count(r.request_id)) DESC"
"        Sort Method: quicksort  Memory: 27kB"
"        Buffers: shared hit=510"
"        ->  GroupAggregate  (cost=50.62..50.71 rows=1 width=323) (actual time=3.135..3.321 rows=20 loops=1)"
"              Output: w.warehouse_id, w.manager_name, p.product_name, count(r.request_id), sum(r.quantity), avg(p.unit_cost), sum(i.quantity), sum(((i.quantity)::numeric * p.unit_cost))"
"              Group Key: w.warehouse_id, p.product_name"
"              Filter: ((count(r.request_id) > 0) AND (sum(i.quantity) > 100))"
"              Buffers: shared hit=510"
"              ->  Sort  (cost=50.62..50.63 rows=2 width=252) (actual time=3.106..3.127 rows=200 loops=1)"
"                    Output: w.warehouse_id, p.product_name, w.manager_name, r.request_id, r.quantity, p.unit_cost, i.quantity"
"                    Sort Key: w.warehouse_id, p.product_name"
"                    Sort Method: quicksort  Memory: 53kB"
"                    Buffers: shared hit=510"
"                    ->  Nested Loop  (cost=19.15..50.61 rows=2 width=252) (actual time=0.311..2.862 rows=200 loops=1)"
"                          Output: w.warehouse_id, p.product_name, w.manager_name, r.request_id, r.quantity, p.unit_cost, i.quantity"
"                          Inner Unique: true"
"                          Buffers: shared hit=510"
"                          ->  Hash Join  (cost=19.01..50.04 rows=2 width=238) (actual time=0.241..1.990 rows=250 loops=1)"
"                                Output: i.quantity, i.product_id, w.warehouse_id, w.manager_name, r.request_id, r.quantity"
"                                Inner Unique: true"
"                                Hash Cond: (i.warehouse_id = w.warehouse_id)"
"                                Buffers: shared hit=10"
"                                ->  Hash Right Join  (cost=3.75..34.51 rows=100 width=20) (actual time=0.204..1.581 rows=1000 loops=1)"
"                                      Output: i.quantity, i.warehouse_id, i.product_id, r.request_id, r.quantity"
"                                      Inner Unique: true"
"                                      Hash Cond: ((r.product_id = i.product_id) AND (r.warehouse_id = i.warehouse_id))"
"                                      Buffers: shared hit=9"
"                                      ->  Seq Scan on firsov2272.requests r  (cost=0.00..25.50 rows=1000 width=16) (actual time=0.039..0.574 rows=1000 loops=1)"
"                                            Output: r.request_id, r.request_date, r.product_id, r.warehouse_id, r.shop_id, r.quantity, r.status"
"                                            Filter: (r.request_date >= (CURRENT_DATE - '90 days'::interval))"
"                                            Buffers: shared hit=8"
"                                      ->  Hash  (cost=2.25..2.25 rows=100 width=12) (actual time=0.152..0.153 rows=100 loops=1)"
"                                            Output: i.quantity, i.warehouse_id, i.product_id"
"                                            Buckets: 1024  Batches: 1  Memory Usage: 13kB"
"                                            Buffers: shared hit=1"
"                                            ->  Seq Scan on firsov2272.inventory i  (cost=0.00..2.25 rows=100 width=12) (actual time=0.022..0.077 rows=100 loops=1)"
"                                                  Output: i.quantity, i.warehouse_id, i.product_id"
"                                                  Filter: (i.quantity > 10)"
"                                                  Buffers: shared hit=1"
"                                ->  Hash  (cost=15.20..15.20 rows=5 width=222) (actual time=0.024..0.024 rows=5 loops=1)"
"                                      Output: w.warehouse_id, w.manager_name"
"                                      Buckets: 1024  Batches: 1  Memory Usage: 9kB"
"                                      Buffers: shared hit=1"
"                                      ->  Seq Scan on firsov2272.warehouses w  (cost=0.00..15.20 rows=5 width=222) (actual time=0.011..0.019 rows=5 loops=1)"
"                                            Output: w.warehouse_id, w.manager_name"
"                                            Filter: (w.warehouse_id = ANY ('{1,2,3,4,5}'::integer[]))"
"                                            Rows Removed by Filter: 15"
"                                            Buffers: shared hit=1"
"                          ->  Index Scan using products_pkey on firsov2272.products p  (cost=0.14..0.28 rows=1 width=22) (actual time=0.003..0.003 rows=1 loops=250)"
"                                Output: p.product_name, p.unit_cost, p.product_id"
"                                Index Cond: (p.product_id = i.product_id)"
"                                Filter: ((p.unit_cost >= '100'::numeric) AND (p.unit_cost <= '500'::numeric))"
"                                Rows Removed by Filter: 0"
"                                Buffers: shared hit=500"
"Planning:"
"  Buffers: shared hit=7"
"Planning Time: 1.466 ms"
"Execution Time: 3.479 ms"
```

Выявленные проблемы:
- Отсутствие индексов на часто фильтруемых полях (quantity, request_date)
- Полное сканирование таблиц вместо использования индексов
- Hash Join может быть заменен на более эффективные соединения

## 3. Оптимизация
```
-- 1. Индекс для фильтрации по quantity
CREATE INDEX idx_inventory_quantity ON inventory(quantity) WHERE quantity > 10;

-- 2. Индекс для фильтрации по дате в requests
CREATE INDEX idx_requests_date ON requests(request_date);

-- 3. Составной индекс для JOIN inventory ↔ requests
CREATE INDEX idx_inventory_product_warehouse ON inventory(product_id, warehouse_id, quantity);

-- 4. Индекс для фильтрации по цене в products
CREATE INDEX idx_products_price ON products(unit_cost) WHERE unit_cost BETWEEN 100 AND 500;

-- 5. Индекс для warehouses (уже есть PK, но для IN clause)
CREATE INDEX idx_warehouses_id ON warehouses(warehouse_id);
```
- Создание частичных индексов для часто используемых диапазонов
- Составные индексы для часто JOIN-соединений
- Индексы для полей в WHERE условиях
### 4. План выполнения после оптимизации
```
"QUERY PLAN"
"Limit  (cost=40.67..40.67 rows=2 width=323) (actual time=2.342..2.354 rows=20 loops=1)"
"  Output: w.warehouse_id, w.manager_name, p.product_name, (count(r.request_id)), (sum(r.quantity)), (avg(p.unit_cost)), (sum(i.quantity)), (sum(((i.quantity)::numeric * p.unit_cost)))"
"  Buffers: shared hit=11"
"  ->  Sort  (cost=40.67..40.67 rows=2 width=323) (actual time=2.340..2.347 rows=20 loops=1)"
"        Output: w.warehouse_id, w.manager_name, p.product_name, (count(r.request_id)), (sum(r.quantity)), (avg(p.unit_cost)), (sum(i.quantity)), (sum(((i.quantity)::numeric * p.unit_cost)))"
"        Sort Key: (sum(((i.quantity)::numeric * p.unit_cost))) DESC, (count(r.request_id)) DESC"
"        Sort Method: quicksort  Memory: 27kB"
"        Buffers: shared hit=11"
"        ->  HashAggregate  (cost=40.26..40.66 rows=2 width=323) (actual time=2.267..2.306 rows=20 loops=1)"
"              Output: w.warehouse_id, w.manager_name, p.product_name, count(r.request_id), sum(r.quantity), avg(p.unit_cost), sum(i.quantity), sum(((i.quantity)::numeric * p.unit_cost))"
"              Group Key: w.warehouse_id, p.product_name"
"              Filter: ((count(r.request_id) > 0) AND (sum(i.quantity) > 100))"
"              Batches: 1  Memory Usage: 48kB"
"              Buffers: shared hit=11"
"              ->  Hash Join  (cost=8.66..39.81 rows=20 width=252) (actual time=0.279..1.933 rows=200 loops=1)"
"                    Output: w.warehouse_id, p.product_name, w.manager_name, r.request_id, r.quantity, p.unit_cost, i.quantity"
"                    Inner Unique: true"
"                    Hash Cond: (i.product_id = p.product_id)"
"                    Buffers: shared hit=11"
"                    ->  Hash Join  (cost=5.14..36.22 rows=25 width=238) (actual time=0.127..1.696 rows=250 loops=1)"
"                          Output: i.quantity, i.product_id, w.warehouse_id, w.manager_name, r.request_id, r.quantity"
"                          Inner Unique: true"
"                          Hash Cond: (i.warehouse_id = w.warehouse_id)"
"                          Buffers: shared hit=10"
"                          ->  Hash Right Join  (cost=3.75..34.51 rows=100 width=20) (actual time=0.103..1.372 rows=1000 loops=1)"
"                                Output: i.quantity, i.warehouse_id, i.product_id, r.request_id, r.quantity"
"                                Inner Unique: true"
"                                Hash Cond: ((r.product_id = i.product_id) AND (r.warehouse_id = i.warehouse_id))"
"                                Buffers: shared hit=9"
"                                ->  Seq Scan on firsov2272.requests r  (cost=0.00..25.50 rows=1000 width=16) (actual time=0.011..0.543 rows=1000 loops=1)"
"                                      Output: r.request_id, r.request_date, r.product_id, r.warehouse_id, r.shop_id, r.quantity, r.status"
"                                      Filter: (r.request_date >= (CURRENT_DATE - '90 days'::interval))"
"                                      Buffers: shared hit=8"
"                                ->  Hash  (cost=2.25..2.25 rows=100 width=12) (actual time=0.080..0.081 rows=100 loops=1)"
"                                      Output: i.quantity, i.warehouse_id, i.product_id"
"                                      Buckets: 1024  Batches: 1  Memory Usage: 13kB"
"                                      Buffers: shared hit=1"
"                                      ->  Seq Scan on firsov2272.inventory i  (cost=0.00..2.25 rows=100 width=12) (actual time=0.007..0.039 rows=100 loops=1)"
"                                            Output: i.quantity, i.warehouse_id, i.product_id"
"                                            Filter: (i.quantity > 10)"
"                                            Buffers: shared hit=1"
"                          ->  Hash  (cost=1.32..1.32 rows=5 width=222) (actual time=0.016..0.017 rows=5 loops=1)"
"                                Output: w.warehouse_id, w.manager_name"
"                                Buckets: 1024  Batches: 1  Memory Usage: 9kB"
"                                Buffers: shared hit=1"
"                                ->  Seq Scan on firsov2272.warehouses w  (cost=0.00..1.32 rows=5 width=222) (actual time=0.006..0.012 rows=5 loops=1)"
"                                      Output: w.warehouse_id, w.manager_name"
"                                      Filter: (w.warehouse_id = ANY ('{1,2,3,4,5}'::integer[]))"
"                                      Rows Removed by Filter: 15"
"                                      Buffers: shared hit=1"
"                    ->  Hash  (cost=2.50..2.50 rows=82 width=22) (actual time=0.113..0.114 rows=81 loops=1)"
"                          Output: p.product_name, p.unit_cost, p.product_id"
"                          Buckets: 1024  Batches: 1  Memory Usage: 13kB"
"                          Buffers: shared hit=1"
"                          ->  Seq Scan on firsov2272.products p  (cost=0.00..2.50 rows=82 width=22) (actual time=0.024..0.072 rows=81 loops=1)"
"                                Output: p.product_name, p.unit_cost, p.product_id"
"                                Filter: ((p.unit_cost >= '100'::numeric) AND (p.unit_cost <= '500'::numeric))"
"                                Rows Removed by Filter: 19"
"                                Buffers: shared hit=1"
"Planning:"
"  Buffers: shared hit=140 read=6 dirtied=2"
"Planning Time: 2.821 ms"
"Execution Time: 2.503 ms"
```
Последствия оптимизации:
- Индексы уменьшили использование памяти (Buffers hit ↓98%)
- Время выполнения уменьшилось на 28%
- Для маленьких таблиц PostgreSQL часто выбирает Seq Scan, даже если есть индексы
- Planning Time увеличился — планировщику нужно больше времени для выбора из большего количества вариантов
