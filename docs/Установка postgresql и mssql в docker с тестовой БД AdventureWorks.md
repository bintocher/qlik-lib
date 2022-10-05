# Краткая инструкция по запуску у себя на железе бд postresql или mssql с тестовыми данными из БД AdventureWorks

Итоговое время 2-10 минут на установку, и всё готово для тестирований.

# Docker

Скачиваем с [официального сайта](https://www.docker.com) на любую ОС docker, и устанавливаем.

Инструкций в интернете множество, описывать не буду

# Запускаем postgresql в docker

Открываем терминал (cmd/powershell/bash...) и вводим строку снизу, где **my_password** - это пароль для пользователя **postgres**


``` cmd
docker run -p 5432:5432 -e 'POSTGRES_PASSWORD=my_password' -d chriseaton/adventureworks:postgres

REM Как подсказали, возможно, нужно убрать кавычки чтобы сработало. Вероятно это зависит от версии докера

docker run -p 5432:5432 -e POSTGRES_PASSWORD=my_password -d chriseaton/adventureworks:postgres

```

![image](https://user-images.githubusercontent.com/8188055/193648493-0bc9411d-15d0-41e1-a47c-db9b98b8333a.png)


Ждём окончания процесса и проверяем

# Запускаем mssql в docker

Открываем терминал (cmd/powershell/bash...) и вводим строку снизу, где **my_password** - это пароль для пользователя **sa**
``` cmd
docker run -p 1433:1433 -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=my_password' -d chriseaton/adventureworks:latest
```

![image](https://user-images.githubusercontent.com/8188055/193648473-777aeb3a-3965-4ab6-911a-9a35e79d542e.png)


Ждём окончания процесса и проверяем

# Проверка

Используем ваш любимый менеджер для БД, у меня это [dbeaver](https://dbeaver.io) смотрим в бд:

![image](https://user-images.githubusercontent.com/8188055/193648615-02d2558b-d3aa-4656-bbf0-3984fa5a4e41.png)

![image](https://user-images.githubusercontent.com/8188055/193648806-5f124d1c-77f6-47b2-8a4d-c15e6f864882.png)
