# Настройка отдельного хранилища логов Moodle

Эта инструкция описывает, как настроить Moodle для хранения логов в отдельной базе данных.

## Шаг 1. Создать базу данных и пользователя

1. Подключитесь к серверу MariaDB/MySQL:
   ```bash
   mysql -u root -p
   ```

2. Создайте базу данных для логов:
   ```sql
   CREATE DATABASE moodle_log DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

3. Создайте пользователя с нужными привилегиями:
   ```sql
   GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, DROP, INDEX, ALTER
   ON moodle_log.* TO 'moodleuser'@'localhost' IDENTIFIED BY 'your_password';
   FLUSH PRIVILEGES;
   ```

4. Проверьте подключение:
   ```bash
   mysql -u moodleuser -h 127.0.0.1 -p moodle_log
   ```

---

## Шаг 2. Создать таблицу для логов

Создайте таблицу `mdl_logstore_standard_log` с помощью SQL-запроса:

```sql
CREATE TABLE mdl_logstore_standard_log (
    id BIGINT(20) NOT NULL AUTO_INCREMENT,
    eventname VARCHAR(255) NOT NULL,
    component VARCHAR(100) NOT NULL,
    action VARCHAR(100) NOT NULL,
    target VARCHAR(100) NULL DEFAULT NULL,
    `objecttable` VARCHAR(50) NULL DEFAULT NULL,
    `objectid` BIGINT(20) NULL DEFAULT NULL,
    `crud` CHAR(1) NOT NULL,
    `edulevel` TINYINT(1) NOT NULL,
    contextid BIGINT(20) NOT NULL,
    contextlevel TINYINT(3) NOT NULL,
    contextinstanceid BIGINT(20) NOT NULL,
    userid BIGINT(20) NOT NULL,
    `courseid` BIGINT(20) NULL DEFAULT NULL,
    `relateduserid` BIGINT(20) NULL DEFAULT NULL,
    `anonymous` TINYINT(1) NOT NULL DEFAULT 0,
    `other` LONGTEXT NULL DEFAULT NULL,
    `timecreated` BIGINT(20) NOT NULL,
    `origin` VARCHAR(10) NULL DEFAULT NULL,
    `ip` VARCHAR(45) NULL DEFAULT NULL,
    `realuserid` BIGINT(20) NULL DEFAULT NULL,
    PRIMARY KEY (id),
    INDEX eventname_idx (eventname),
    INDEX courseid_idx (courseid),
    INDEX userid_idx (userid),
    INDEX contextid_idx (contextid),
    INDEX contextlevel_idx (contextlevel),
    INDEX contextinstanceid_idx (contextinstanceid),
    INDEX timecreated_idx (timecreated)
) ENGINE=InnoDB DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

---

## Шаг 3. Настройка Moodle для новой базы данных

1. Авторизуйтесь в Moodle как администратор.
2. Перейдите в: Администрирование сайта > Плагины > Логи > Управление логами
3. Найдите **"Стандартное хранилище логов"** и нажмите на иконку **"Редактировать"**.
4. Заполните параметры подключения:

| Поле                          | Значение                                                                                |
|-------------------------------|----------------------------------------------------------------------------------------|
| **Таблица базы данных**        | `mdl_logstore_standard_log`                                                            |
| **Постоянное соединение**      | `Да`                                                                                   |
| **Подключение через Unix-сокет** | Оставить пустым (если используется TCP/IP)                                            |
| **Порт базы данных**           | `3306`                                                                                 |
| **Хост**                       | `127.0.0.1`                                                                            |
| **Схема базы данных**          | `moodle_log`                                                                           |
| **Имя пользователя базы данных** | `moodleuser`                                                                        |
| **Пароль базы данных**         | Укажите пароль, который был задан для пользователя `moodleuser`.                       |
| **Сопоставление базы данных**  | Оставьте пустым (используется `utf8mb4_unicode_ci` по умолчанию).                       |
| **Размер буфера**              | `512` (стоит увеличить до 512KB или 1MB для серверов с высокой нагрузкой).             |
| **Формат JSON**                | `Сжатый текст` (Compressed text).                                                     |

Сохраните изменения.

---

## Шаг 4. Тестирование подключения

Moodle автоматически проверит соединение с новой базой данных. Если возникнут ошибки, проверьте настройки подключения, таблицу и права пользователя.

---

## Примечания

1. **Размер буфера:**
   - Для тестовых стендов можно оставить **50KB**.
   - Для серверов с высокой нагрузкой используйте **512KB** или **1MB**.

2. **Мониторинг и очистка логов:**
   В админке Moodle можно настроить автоматическую очистку старых логов: Администрирование сайта > Сервер > Очистка данных
3. **Диагностика подключения:**
   - Проверьте базу данных и пользователя через:
     ```sql
     SHOW GRANTS FOR 'moodleuser'@'localhost';
     ```
   - Убедитесь, что MariaDB слушает на нужном порту в файле `/etc/mysql/my.cnf`.

---

Теперь логи Moodle должны записываться в новую базу данных `moodle_log`!

Для любых вопросов или уточнений обращайтесь.😊
