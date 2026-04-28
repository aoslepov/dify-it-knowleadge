# MySQL: Работа с кластером

## Общие правила

- **Альтеры и индексы** выполняем в `screen`/`tmux` **по одной штуке на кластер**.
- После выполнения запроса **дожидаемся его доезда на реплики** (пропадёт `alter table` в processlist на репликах).
- Все таблицы **обязаны иметь PRIMARY KEY**.

---

## Просмотр медленных запросов

### Через innotop
```bash
innotop -p<pass>
```
Внутри программы используй: d 1, затем Shift+Q

### Через консоль MySQL
```sql
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;
```

## Добавление/удаление полей (без блокировки)
Важно: Выполнять в screen/tmux

```sql
ALTER TABLE invoices_items_leads ADD affiliate_id INT UNSIGNED DEFAULT NULL AFTER lead_id,  ALGORITHM=INPLACE, LOCK=NONE;
```

## Добавление индексов
Синтаксис с блокировкой 
```sql
CREATE INDEX idx_affiliate_id ON invoices_items_leads (affiliate_id); 
```

Синтаксис ALTER TABLE (без блокировки)
```sql
ALTER TABLE invoices_items_leads ADD INDEX idx_affiliate_id (affiliate_id), ALGORITHM=INPLACE, LOCK=NONE;
```

## Создание пользователей

## Для сервиса (префикс svc_)
```sql
CREATE USER '<USER>'@'%' IDENTIFIED WITH caching_sha2_password BY '<PASS>'   REQUIRE SSL;
```

Далее выдаём гранты пользователю.

Для сотрудника (привязка к IP)
```sql
CREATE USER '<USER>'@'46.148.226.57' IDENTIFIED WITH caching_sha2_password BY '<PASS>' REQUIRE SSL;
CREATE USER '<USER>'@'46.148.226.62' IDENTIFIED WITH caching_sha2_password BY '<PASS>' REQUIRE SSL;
CREATE USER '<USER>'@'49.13.26.6'   IDENTIFIED WITH caching_sha2_password BY '<PASS>' REQUIRE SSL;
```
Далее выдаём гранты каждой паре юзер/хост.

Примечание: Пользователей с плагином mysql_native_password стараемся не создавать.

## Квоты для пользователей
```sql
ALTER USER '<USER>'@'192.168.0.1' WITH MAX_USER_CONNECTIONS 5;
ALTER USER '<USER>'@'192.168.0.2' WITH MAX_USER_CONNECTIONS 5;
ALTER USER '<USER>'@'192.168.0.3' WITH MAX_USER_CONNECTIONS 5;
```

## Создание полей типа LONGTEXT
⚠️ Предупреждение: Перед созданием подумать, нужно ли это!
```
Тип	Максимальная длина
TINYTEXT	255 (2⁸−1) байт
TEXT	65 535 (2¹⁶−1) байт = 64 КиБ
MEDIUMTEXT	16 777 215 (2²⁴−1) байт = 16 МиБ
LONGTEXT	4 294 967 295 (2³²−1) байт = 4 ГиБ
```

## Статус кластера
```sql
SELECT * FROM performance_schema.replication_group_members;
```

## Выгрузка данных из БД
Можно делать через DBeaver (увеличив socket timeout). Выгрузку выполняем на реплике в screen/tmux.

```sql
SELECT order_id, product_name, qty
FROM orders
INTO OUTFILE '/var/lib/mysql-files/orders.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```
