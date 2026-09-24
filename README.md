# Проект Kittygram
## Описание
«Киттиграм» — социальная сеть для любителей кошек (SPA-приложение), в которой пользователи могут публиковать карточки своих питомцев, просматривать чужих котиков и делиться их достижениями. Доступ к контенту сайта ограничен: добавлять питомцев и управлять ими могут только авторизованные пользователи, а редактировать или удалять карточки разрешено исключительно их владельцам.

## Технологии и библиотеки
*   **Язык программирования:** Python 3.12
*   **Основные библиотеки:**
    *   Django
    *   Django REST Framework
    *   Djoser
    *   Webcolors
    *   Pillow
    *   Docker, Docker Compose
    *   Flake8
    *   Nginx
    *   PostgreSQL 13 (Production)
    *   Pytest / Pytest-django
    *   PyYAML

## Установка и запуск проекта (Локально)
1.  Клонируйте репозиторий:
    ```bash
    git clone git@github.com:Igor-Izhbiakov/kittygram
    cd kittygram
    ```
2.  Создайте и активируйте виртуальное окружение:
    *   **Для Windows:**
        ```bash
        python -m venv venv
        source venv/Scripts/activate
        ```
    *   **Для macOS / Linux:**
        ```bash
        python3 -m venv venv
        source venv/bin/activate
        ```
3.  Установите зависимости бэкенда из файла requirements.txt:
    ```bash
    pip install --upgrade pip
    pip install -r ./backend/requirements.txt
    ```
4.  Выполните миграции для создания структуры базы данных:
    ```bash
    cd backend
    python manage.py makemigrations
    python manage.py migrate
    ```
5.  Запустите сервер разработки:
    ```bash
    python manage.py runserver
    ```
    Проект будет доступен по адресу: [http://127.0.0.1:8000/](http://127.0.0.1:8000/) 

## Алгоритм регистрация и аутентификация пользователей
В проекте используется Token-аутентификация на базе библиотеки Djoser:
1.  **Регистрация пользователя:** Пользователь отправляет POST-запрос со своими данными (email, username, first_name, last_name, password) на эндпоинт `/api/users/`.
2.  **Получение токена:** Пользователь отправляет POST-запрос с параметрами `email` и `password` на эндпоинт `/api/auth/token/login/`. В ответе возвращается авторизационный токен (`auth_token`).
3.  **Инициализация сессии и авторизация:** Получив токен, SPA автоматически отправляет GET-запрос на `/api/users/me/` для получения `id` текущего пользователя. При каждом следующем запросе к защищенным эндпоинтам токен передается в заголовке `Authorization` в формате:
    ```http
    Authorization: Token <значение_токена>
    ```
4.  **Выход из системы:** Для удаления токена отправляется POST-запрос на эндпоинт `/api/auth/token/logout/`.

## Пользовательские роли и права доступа
*   **Анонимный пользователь:** Имеет доступ только к интерфейсам регистрации и аутентификации. Доступ к просмотру карточек котиков ограничен.
*   **Аутентифицированный пользователь:** Обладает полными правами внутри системы:
    *   Просматривать общий список котиков на сайте.
    *   Создавать новые карточки питомцев (с обязательным указанием имени, года рождения, цвета в формате HEX и возможностью загрузить фото / добавить достижения).
    *   Редактировать и удалять **только своих** котиков. Карточки чужих питомцев доступны исключительно для просмотра.
    *   Искать и выбирать существующие достижения, а также создавать новые достижения в системе на лету при сохранении карточки кота.
*   **Администратор / Суперпользователь Django:** Обладает полными правами на управление всем контентом и пользователями проекта через панель администратора.

## Примеры запросов к API

### Получение списка котиков (Только для авторизованных, поддерживает пагинацию до 10 объектов)
**Запрос:** `GET /api/cats/`  
**Ответ (200 OK):**
```json
{
  "count": 1,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "Матроскин",
      "color": "gray",
      "birth_year": 2022,
      "achievements": [
        {
          "id": 1,
          "achievement_name": "Поймал мышь"
        }
      ],
      "owner": 2,
      "age": 4,
      "image_url": "/media/cats/images/matroskin.png"
    }
  ]
}
```

### Добавление новой карточки котика (Только для авторизованных)
*Примечание: Если переданного достижения нет в базе, система автоматически создаст его.*  
**Запрос:** `POST /api/cats/`  
**Тело запроса:**
```json
{
  "name": "Барсик",
  "color": "#FFFFFF",
  "birth_year": 2024,
  "achievements": [
    {
      "achievement_name": "Гроза сметаны"
    }
  ],
  "image": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABAgMAAABieywaAAAACVBMVEUAAAD///9fX1/S0ecCAAAACXBIWXMAAA7EAAAOxAGVKw4bAAAACklEQVQImWNoAAAAggCByxOyYQAAAABJRU5ErkJggg=="
}
```
**Ответ (201 Created):**
```json
{
  "id": 2,
  "name": "Барсик",
  "color": "white",
  "birth_year": 2024,
  "achievements": [
    {
      "id": 2,
      "achievement_name": "Гроза сметаны"
    }
  ],
  "owner": 2,
  "age": 2,
  "image_url": "/media/cats/images/temp.png"
}
```

## Развертывание проекта на удаленном сервере (Production)

### Архитектура сети на сервере
На удаленном сервере реализована многопроектная архитектура изолированных контейнеров, управляемая через Docker Compose, которая включает в себя 4 сервиса и работает на внешнем порту **9000**:
*   **Контейнер базы данных PostgreSQL (db):** Использует образ `postgres:13`, полностью изолирован во внутренней сети Docker. Данные сохраняются в постоянный том (`pg_data`).
*   **Контейнер бэкенда Django (backend):** Запускает приложение бэкенда, связывается со статикой и медиафайлами через тома. Зависит от успешного старта контейнера базы данных.
*   **Контейнер фронтенда React (frontend):** Собирает статические файлы пользовательского интерфейса (SPA) и копирует их в общий разделяемый том статики.
*   **Контейнер шлюза Nginx (gateway):** Принимает внешний трафик на порту `9000`, перенаправляет запросы к бэкенду или раздает собранную статику фронтенда/бэкенда и медиафайлы из общих томов.

### Инструкция по деплою проекта
1.  **Подготовка папки проекта на сервере:**
    ```bash
    mkdir -p kittygram && cd kittygram
    ```
2.  **Настройка переменных окружения:** Внутри папки `kittygram/` создайте файл `.env` (`nano .env`) и заполните его боевыми данными:
    ```env
    POSTGRES_DB=kittygram
    POSTGRES_USER=kittygram_user
    POSTGRES_PASSWORD=password
    DB_HOST=db
    DB_PORT=5432
    SECRET_KEY='your_production_secret_key'
    ALLOWED_HOSTS=localhost,127.0.0.1,ваш_ip_сервера,yourkittygram.duckdns.org
    ```
3.  **Запуск контейнеров через CI/CD:** Файл `docker-compose.production.yml` автоматически копируется на сервер, а сборка образов, тестирование (`flake8`, `django tests`, `npm run test` на Node.js 18) и перезапуск контейнеров происходят автоматически при каждом пуше в ветку `main` благодаря настроенному GitHub Actions. 
    *(Для ручного перезапуска на сервере используется команда: `sudo docker compose -f docker-compose.production.yml up -d --build`)*

4.  **Первоначальная настройка базы данных внутри Docker:** При первом запуске проекта выполните по очереди команды внутри контейнера бэкенда для применения миграций и сбора статики:
    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic --no-input
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py createsuperuser
    ```

## Авторы проекта
Разработчик бэкенда: [Igor Izhbiakov](https://github.com)