# Microblog API

REST API для социальной сети: пользователи публикуют посты, комментируют, 
подписываются друг на друга и объединяются в группы.

## О проекте

Backend-часть социальной сети Yatube. Реализует REST API для работы с 
постами, группами, комментариями и подписками. Поддерживает 
JWT-аутентификацию, фильтрацию, поиск и разграничение прав доступа.

Проект демонстрирует:
- проектирование REST API на Django REST Framework;
- JWT-аутентификацию через `djangorestframework-simplejwt`;
- кастомные permissions (`IsAuthorOrReadOnly`);
- фильтрацию (`django-filter`) и поиск (`SearchFilter`);
- вложенную маршрутизацию (комментарии к постам);
- покрытие API тестами на `pytest`.

##  Стек

- **Backend:** Python 3.11, Django 4.2, Django REST Framework
- **Аутентификация:** SimpleJWT (Bearer-токены)
- **Фильтрация:** django-filter
- **БД:** PostgreSQL 16 / SQLite (для локальной разработки)
- **Инфраструктура:** Docker, Docker Compose, Gunicorn
- **Тесты:** pytest, pytest-django
- **Документация API:** ReDoc, Postman Collection

##  Запуск

### Через Docker 

1. Клонировать репозиторий:  
    git clone https://github.com/endlesslessness/microblog-api.git  
    cd api_final_yatube  

2. Создать .env в корне проекта:
    env  
    DJANGO_SECRET_KEY=your-secret-key-here  
    DJANGO_DEBUG=False  
    DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1  
  
    POSTGRES_DB=yatube  
    POSTGRES_USER=yatube  
    POSTGRES_PASSWORD=yatube  
    DB_HOST=db  
    DB_PORT=5432  
    Запустить:  
  
3. Запуск
    docker-compose up --build  
    API будет доступно по адресу: http://localhost:8000/api/  

###  Локально (без Docker)
    python3 -m venv venv
    source venv/bin/activate       # Linux/macOS
    # venv\Scripts\activate        # Windows

    pip install -r requirements.txt
    python manage.py migrate
    python manage.py runserver


##  Основные эндпоинты
Метод	Эндпоинт	Описание	Доступ  
POST	/api/token/	Получить JWT-токен	Все  
POST	/api/token/refresh/	Обновить JWT-токен	Все  
POST	/api/register/	Регистрация пользователя	Все  
GET	/api/posts/	Список постов (пагинация 10)	Все  
POST	/api/posts/	Создать пост	Авторизованные  
GET	/api/posts/{id}/	Детали поста	Все  
PATCH	/api/posts/{id}/	Обновить пост	Автор  
DELETE	/api/posts/{id}/	Удалить пост	Автор  
GET	/api/posts/?group={id}	Фильтр постов по группе	Все  
GET	/api/groups/	Список групп	Все  
GET	/api/posts/{post_id}/comments/	Комментарии к посту	Все  
POST	/api/posts/{post_id}/comments/	Добавить комментарий	Авторизованные  
GET	/api/follow/	Мои подписки	Авторизованные  
POST	/api/follow/	Подписаться	Авторизованные  
GET	/api/follow/?search={username}	Поиск по подпискам	Авторизованные  

##  Примеры запросов
Получение JWT-токена
curl -X POST http://localhost:8000/api/token/ \
  -H "Content-Type: application/json" \
  -d '{"username": "user", "password": "pass12345"}'

Ответ:
{
  "refresh": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "access": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

Создание поста
curl -X POST http://localhost:8000/api/posts/ \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{"text": "Мой первый пост", "group": 1}'

Подписка на пользователя
curl -X POST http://localhost:8000/api/follow/ \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{"following": "other_user"}'

##  Архитектурные решения
JWT вместо сессий
Аутентификация через djangorestframework-simplejwt. Токен передаётся в
заголовке Authorization: Bearer <token>. Это стандарт для SPA и мобильных
клиентов, где cookie-сессии неудобны.

Кастомный permission IsAuthorOrReadOnly
Читать посты и комментарии могут все, но редактировать и удалять — только
автор. Реализовано через has_object_permission с проверкой
SAFE_METHODS (GET, HEAD, OPTIONS).

Вложенная маршрутизация комментариев
Комментарии доступны по /api/posts/{post_id}/comments/. Это явно
показывает связь «пост → комментарии» и упрощает клиенту навигацию.

Фильтрация и поиск
Посты фильтруются по группе (?group={id}), подписки — по имени
пользователя (?search={username}). Реализовано через DjangoFilterBackend
и SearchFilter.

Валидация подписок
Нельзя подписаться на самого себя (проверка в validate_following) и
нельзя подписаться дважды (UniqueTogetherValidator).

Пагинация
Список постов отдаётся постранично (10 записей). Список групп — без
пагинации, так как групп обычно немного.

##  Тесты
pytest
Покрыто:

test_post.py — CRUD постов, права доступа;

test_comment.py — комментарии к постам;

test_group.py — группы;

test_follow.py — подписки, валидация;

test_jwt.py — аутентификация через JWT.

##  Структура проекта  
api_final_yatube/  
├── tests/                    # Тесты на pytest  
│   ├── fixtures/             # Фикстуры  
│   ├── conftest.py  
│   ├── test_comment.py  
│   ├── test_follow.py  
│   ├── test_group.py  
│   ├── test_jwt.py  
│   └── test_post.py  
├── yatube_api/  
│   ├── api/                  # API-приложение  
│   │   ├── permissions.py    # Кастомные права  
│   │   ├── serializers.py    # Сериализаторы  
│   │   ├── urls.py           # Маршруты API  
│   │   └── views.py          # ViewSet'ы  
│   ├── posts/                # Модели: Post, Group, Comment, Follow  
│   ├── static/redoc.yaml     # OpenAPI-схема  
│   ├── manage.py  
│   └── yatube_api/           # Настройки проекта  
├── postman_collection/       # Postman-коллекция для тестирования  
├── .env.example  
├── .gitignore  
├── pytest.ini  
├── requirements.txt  
├── setup.cfg  
└── README.md  
  
##  Что можно улучшить
- Добавить Swagger UI в дополнение к ReDoc
- Настроить CI/CD (GitHub Actions: линтер + тесты)
- Покрыть тестами регистрацию и обновление постов
- Добавить throttling для защиты от брутфорса

## Контакты
Telegram: @endlesslessness
Email: yan.lejn@mail.ru
