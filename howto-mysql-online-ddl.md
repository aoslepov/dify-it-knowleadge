---
type: how-to
db_type: mysql
tags: [alter, indexing, online-ddl, ddl, locking]
difficulty: intermediate
mysql_version: "8.0+"
date_verified: 2026-01-01
---

# Создание индекса, добавление и удаление поля без блокировок в MySQL

## Проблема
Стандартный ALTER TABLE и CREATE INDEX блокирует таблицу на запись

## Решение

```sql
-- Добавление поля
-- используем вместо
ALTER TABLE invoices_items_leads ADD affiliate_id INT UNSIGNED DEFAULT NULL after lead_id;

-- запись
ALTER TABLE invoices_items_leads ADD affiliate_id INT UNSIGNED DEFAULT NULL after lead_id, ALGORITHM=INPLACE, LOCK=NONE;
 
-- Добавляем индекс
-- используем вместо
CREATE INDEX idx_affiliate_id ON invoices_items_leads (affiliate_id);

-- такую запись
alter table invoices_items_leads add index idx_affiliate_id (affiliate_id), ALGORITHM=INPLACE, LOCK=NONE
```

# Ограничение
```sql
-- поддерживает ALGORITHM=COPY с блокировкой
-- для ускорения желательно снять траффик с записи на таблицу
ALTER TABLE invoices_items_leads CHANGE affiliate_id affiliate_id BIGINT UNSIGNED DEFAULT NULL after lead_id;
```



---
type: how-to
db_type: mysql
tags: [alter, indexing, online-ddl, ddl, locking]
difficulty: intermediate
mysql_version: "8.0+"
date_verified: 2026-01-01
---

# Создание индекса, добавление и удаление поля без блокировок в MySQL

## Проблема
Стандартный `ALTER TABLE` и `CREATE INDEX` блокируют таблицу на запись, что приводит к простою сервиса.

## Решение

### Добавление поля без блокировки

```sql
-- ❌ Блокирующий вариант
ALTER TABLE table_name ADD field INT UNSIGNED DEFAULT NULL AFTER existing_field;

-- ✅ Неблокирующий вариант (ONLINE DDL)
ALTER TABLE table_name ADD new_field INT UNSIGNED DEFAULT NULL AFTER existing_field, ALGORITHM=INPLACE, LOCK=NONE;
```

### Добавление индекса без блокировки

```sql
-- ❌ Блокирующий вариант
CREATE INDEX idx_new_field ON table_name (new_field);

-- ✅ Неблокирующий вариант (ONLINE DDL)
ALTER TABLE table_name ADD INDEX idx_new_field (new_field), ALGORITHM=INPLACE, LOCK=NONE
```

### Ограничения
```sql
-- Изменение типа данных НЕ поддерживает INPLACE
-- Только ALGORITHM=COPY с полной блокировкой таблицы
-- Необходимо снять траффик с таблицы на запись перед выполнением
ALTER TABLE orders CHANGE new_field new_field BIGINT UNSIGNED DEFAULT NULL AFTER existing_field, ALGORITHM=COPY;
```

### Важные примечания
- данные операции нужно выполнять в tmux/screen
- ALGORITHM=INPLACE — изменяет таблицу на месте, без создания копии
- LOCK=NONE — разрешает параллельные чтение и запись
- Перед выполнением убедитесь в отсутствии длительных транзакций
- Для таблиц > 10GB операция может занять минуты, но без блокировки
- Для больших таблиц убедитесь что места на диске 2x(размер таблицы) хватит
- Операции нужно выполнять по одному и дожидаться выполнения как на мастере, так и на всех репликах

### Мониторинг ddl
```sql
-- текущие операции ddl(alter)
SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED FROM  performance_schema.events_stages_current;

-- история ddl(alter)
SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED FROM performance_schema.events_stages_history;

-- если не выводит информацию во время альтра, то включаем инструмент
UPDATE performance_schema.setup_instruments SET ENABLED = 'YES'  WHERE NAME LIKE 'stage/innodb/alter%';
UPDATE performance_schema.setup_consumers SET ENABLED = 'YES'  WHERE NAME LIKE '%stages%';
```

### Мониторинг блокировок на бд
```sql
select * from performance_schema.metadata_locks;
```
