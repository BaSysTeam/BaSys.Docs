# Начало работы

Для установки приложения **BaSYS** требуется Docker и одна из поддерживаемых СУБД:
- PostgreSQL
- MS SQL

> **Примечание:** В настоящее время тестирование в основном проводится на PostgreSQL. Поэтому для ознакомления рекомендуется использовать именно эту СУБД.

## Создание файла `docker-compose.yml`

Создайте файл docker-compose.yml, согласно следующего шаблона:

```yml
version: '3'

services:
  basys:
    container_name: basys.app
    image: basysteam/basys:latest
    restart: always
    ports:
      - 13080:8080
    environment:
      # CurrentApp
      InitAppSettings:CurrentApp:Id: main
      InitAppSettings:CurrentApp:Title: Main

      # Sa. Параметры для создания системной базы данных.
      # Логин супер администратора.
      InitAppSettings:Sa:Login: <super_admin_login>
      # Пароль супер администратора.
      InitAppSettings:Sa:Password: <super_admin_password>
      # Вид СУБД: 0 - MS SQL, 1 - PG SQL.
      InitAppSettings:Sa:DbKind: 1
      # Строка подключения к системной базе данных.
      InitAppSettings:Sa:ConnectionString: <super_admin_base_connections_string>

      # MainDb. Параметры для создания рабочей базы данных.
      # Имя базы данных (может содержать латинские буквы в нижнем регистре, цифры и символ подчеркивания)
      InitAppSettings:MainDb:Name: <db_name>
      # Вид СУБД: 0 - MS SQL, 1 - PG SQL
      InitAppSettings:MainDb:DbKind: 1
      # Строка подключения к базе данных.
      InitAppSettings:MainDb:ConnectionString: <work_base_connection_string>
      # Логин администратора.
      InitAppSettings:MainDb:AdminLogin: <admin_login>
      # Пароль администратора. 
      InitAppSettings:MainDb:AdminPassword: <admin_password>
      # Язык по умолчанию для администратора: ru | en
      InitAppSettings:MainDb:Culture: <admin_culture>
    extra_hosts:
      - "host.docker.internal:host-gateway"
```

## Пример файла docker-compose.yml

Пример конфигурации для локальной установки BaSYS с использованием PostgreSQL. Обратите внимание, что имя сервера host.docker.internal соответствует локально установленной СУБД. Не забудьте заменить \<your_sql_server_password\> на пароль вашей базы данных.

```yml
version: '3'

services:
  basys:
    container_name: basys.app
    image: basysteam/basys:latest
    restart: always
    ports:
      - 13080:8080
    environment:
      # CurrentApp
      InitAppSettings:CurrentApp:Id: main
      InitAppSettings:CurrentApp:Title: Main

      # Sa. Параметры для создания системной базы данных.
      # Логин супер администратора.
      InitAppSettings:Sa:Login: sa@mydomain.com
      # Пароль супер администратора.
      InitAppSettings:Sa:Password: 111111
      # Вид СУБД: 0 - MS SQL, 1 - PG SQL.
      InitAppSettings:Sa:DbKind: 1
      # Строка подключения к системной базе данных.
      InitAppSettings:Sa:ConnectionString: Server=host.docker.internal;Port=5432;Database=basys_system;User ID=postgres;Password=<your_sql_server_password>;Timeout=60;

      # MainDb. Параметры для создания рабочей базы данных.
      # Имя базы данных (может содержать латинские буквы в нижнем регистре, цифры и символ подчеркивания)
      InitAppSettings:MainDb:Name: basys_work_1
      # Вид СУБД: 0 - MS SQL, 1 - PG SQL
      InitAppSettings:MainDb:DbKind: 1
      # Строка подключения к базе данных.
      InitAppSettings:MainDb:ConnectionString: Server=host.docker.internal;Port=5432;Database=basys_work_1;User ID=postgres;Password=<your_sql_server_password>;Timeout=60;
      # Логин администратора.
      InitAppSettings:MainDb:AdminLogin: admin@mydomain.com
      # Пароль администратора. 
      InitAppSettings:MainDb:AdminPassword: 111111
      # Язык по умолчанию для администратора: ru | en
      InitAppSettings:MainDb:Culture: ru
    extra_hosts:
      - "host.docker.internal:host-gateway"

```

## Запуск приложения

1. Перейдите в папку, содержащую созданный файл docker-compose.yml.
2. Выполните следующую команду:

```
docker-compose up -d
```

После выполнения команды приложение **BaSYS** будет доступно браузере по адресу http://localhost:13080/ .