API YAMDB
====
![example workflow](https://github.com/SKhnykin/yamdb_final/actions/workflows/yamdb_workflow.yml/badge.svg)

API для получения информации и обсуждения наиболее интересных произведений.
Для автоматизации развертывания на боевых серверах используется среда виртуализации Docker, а также Docker-compose - инструмент для запуска многоконтейнерных приложений.

## Стек технологий:
- Python 3
- DRF (Django REST framework)
- Django ORM
- Docker
- Gunicorn
- nginx
- Яндекс Облако(Ubuntu)
- Django
- PostgreSQL
- GIT

## О проекте:
Реализована регистрация с кодом подтверждения и дальнейшая авторизация с использованием JWT токена, при отправке запроса к API.

Проект **YaMDb** собирает **отзывы** (**Review**) пользователей на **произведения** (**Titles**). Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Список **категорий** (**Category**) может быть расширен администратором (например, можно добавить категорию «Изобразительное искусство» или «Ювелирка»).

В каждой категории есть **произведения**: книги, фильмы или музыка. Например, в категории «Книги» могут быть произведения «Винни-Пух и все-все-все» и «Марсианские хроники», а в категории «Музыка» — песня «Давеча» группы «Насекомые» и вторая сюита Баха.

Произведению может быть присвоен **жанр** (**Genre**) из списка предустановленных (например, «Сказка», «Рок» или «Артхаус»). Новые жанры может создавать только администратор.

Благодарные или возмущённые пользователи оставляют к произведениям текстовые **отзывы** (**Review**) и ставят произведению оценку в диапазоне от одного до десяти (целое число); из пользовательских оценок формируется усреднённая оценка произведения — **рейтинг** (целое число). На одно произведение пользователь может оставить только один отзыв.

### Документация и возможности API:
К проекту подключен redoc. Для просмотра документации используйте эндпойнт `redoc/`

## Шаблон наполнения .env
```
# указываем, с какой БД работаем
DB_ENGINE=django.db.backends.postgresql
# имя базы данных
DB_NAME=
# логин для подключения к базе данных
POSTGRES_USER=
# пароль для подключения к БД (установите свой)
POSTGRES_PASSWORD=
# название сервиса (контейнера)
DB_HOST=
# порт для подключения к БД
DB_PORT=
```

## Автоматизация развертывания серверного ПО

Ниже представлен Dockerfile - файл с инструкцией по разворачиванию Docker-контейнера веб-приложения:
```Dockerfile
FROM python:3.7-slim

RUN mkdir /app

WORKDIR /app

COPY requirements.txt /app

RUN pip3 install -r /app/requirements.txt --no-cache-dir

COPY ./ /app

CMD ["gunicorn", "api_yamdb.wsgi:application", "--bind", "0:8000" ]
```
В файле «docker-compose.yml» описываются запускаемые контейнеры: веб-приложения, СУБД PostgreSQL и сервера Nginx.
```sh
version: '3.8'

services:
  db:
    image: postgres:13.0-alpine
    
    volumes:
      - /var/lib/postgresql/data/
    env_file:
      - ./.env
  web:
    build: ../api_yamdb
    restart: always
    volumes:
      - static_value:/app/static/
      - media_value:/app/media/
    depends_on:
      - db
    env_file:
      - ./.env    

  nginx:
    image: nginx:1.21.3-alpine

    ports:
      - "80:80"

    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf

      - static_value:/var/html/static/

      - media_value:/var/html/media/

    depends_on:
      - web

volumes:
  static_value:
  media_value:
```

## Описание команд для запуска приложения в контейнерах
Для запуска проекта в контейнерах используем **docker-compose** : ```docker-compose up -d --build```, находясь в директории (infra_sp2) с ```docker-compose.yaml```

После сборки контейнеров выполяем:
```bash
# Выполняем миграции
docker-compose exec web python manage.py migrate
# Создаем суперппользователя
docker-compose exec web python manage.py createsuperuser
# Собираем статику со всего проекта
docker-compose exec web python manage.py collectstatic --no-input
# Для дампа данных из БД
docker-compose exec web python manage.py dumpdata > dump.json
```
