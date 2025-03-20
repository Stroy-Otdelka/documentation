# Реализованные маркетплейсы
* OZON
* WildBerries (WB)

# Схема архитектуры проекта
https://lucid.app/lucidchart/1991848a-b63e-4de5-b3c2-2e2ca063f2ad/edit?invitationId=inv_ebd0a390-2c63-4ef7-814e-fde9a6a1cbb9&page=jTL~579UgVUh#

## Где запущено
* Ozon - свой сервер
* WB - google облако (VM: bot_training)

## Актуальные сервисы:
* **Сервис отзывов** (`/feedback_service`, `/feedback_bot`)
* **gRPC**
* **auth_service** (Аутентификация в монолите - тг айдишники запросов)
* **api_gateway** (Прослойка, кому запросы кидать)

## Установка всех зависимостей
```commandline
poetry install
```

## Как запустить локально

- Нужна Kafka
- На всякий случай отключать на сервере процессы

```commandline
poetry run <директория и т.д.>
```

# Структура .env файла
`.env.example`