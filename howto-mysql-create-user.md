---
type: how-to
db_type: mysql
tags: [users, quotas]
mysql_version: "8.0+"
date_verified: 2026-01-01
---

# Создание пользователей

## Проблема
Создание безопастных пользователей для сервисов и сотрудников

## Решение
- планин caching_sha2_password пароль хранит в шифрованном виде
- плагин mysql_native_password хранит пароль в виде хеша - стараемся не использовать
- REQUIRE SSL - использование асинхронного шифрования при коннекте
- подключение к бд 
```sql
mysqli://USER:PASS@HOST:PORT/DB?serverVersion=8.4&ssl-mode=required
```

### Создание пользователя для сервиса
```sql
-- создаём пользователя
create user '<USER>'@'%' identified with caching_sha2_password by '<PASS>' REQUIRE SSL;

-- даём доступы на чтение/запись 
grant select,insert,update,delete on DB.* to '<USER>'@'%';

-- даём доступы на чтение
grant select on DB.* to '<USER>'@'%';
```

### Создание пользователя для сотрудника
```sql
-- создаём пользователя
create user '<USER>'@'46.148.226.57' identified with caching_sha2_password by '<PASS>' REQUIRE SSL;
create user '<USER>'@'46.148.226.62' identified with caching_sha2_password by '<PASS>' REQUIRE SSL;
create user '<USER>'@'49.13.26.6' identified with caching_sha2_password by '<PASS>' REQUIRE SSL;

-- даём доступы на чтение
grant select on DB.* to '<USER>'@'46.148.226.57';
grant select on DB.* to '<USER>'@'46.148.226.62';
grant select on DB.* to '<USER>'@'49.13.26.6';
```

### Квоты для пользователей и сервисов
```
-- для сервисов квоты квоту ставить исходя из количества коннектов
alter user '<USER>'@'%' WITH MAX_USER_CONNECTIONS 50;

-- для пользователей
alter user '<USER>'@'46.148.226.57' WITH MAX_USER_CONNECTIONS 5;
alter user '<USER>'@'46.148.226.62' WITH MAX_USER_CONNECTIONS 5;
alter user '<USER>'@'49.13.26.6' WITH MAX_USER_CONNECTIONS 5;
```

