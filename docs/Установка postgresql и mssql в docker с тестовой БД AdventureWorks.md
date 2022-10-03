# Краткая инструкция по запуску у себя на железе бд postresql или mssql с тестовыми данными из БД AdventureWorks

Итоговое время 2-10 минут на установку, и всё готово для тестирований.

# Docker

Скачиваем с [официального сайта](https://www.docker.com) на любую ОС docker, и устанавливаем.

Инструкций в интернете множество, описывать не буду

# Запускаем postgresql в docker

Открываем терминал (cmd/powershell/bash...) и вводим строку снизу, где **my_password** - это пароль для пользователя **postgres**


``` cmd
docker run -p 5432:5432 -e 'POSTGRES_PASSWORD=my_password' -d chriseaton/adventureworks:postgres
```

Ждём окончания процесса и проверяем

# Запускаем mssql в docker

Открываем терминал (cmd/powershell/bash...) и вводим строку снизу, где **my_password** - это пароль для пользователя **sa**
``` cmd
docker run -p 1433:1433 -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=my_password' -d chriseaton/adventureworks:latest
```

Ждём окончания процесса и проверяем

# Проверка

Используем ваш любимый менеджер для БД, у меня это [dbeaver](https://dbeaver.io) смотрим в бд:
