# Сервис Advertising Campaigns

**Advertising Campaigns** — это сервис для управления рекламными кампаниями на маркетплейсе Wildberries. Он предоставляет функциональность для синхронизации данных о кампаниях, управления ставками и бюджетами, а также автоматизации их обновления. Сервис работает как набор бессерверных функций Google Cloud Functions, интегрируется с базой данных PostgreSQL через SQLAlchemy и использует API Wildberries для получения актуальной информации.

## Функциональные возможности

- **Синхронизация кампаний**: Получение и обновление данных о рекламных кампаниях (авто и аукционные) через API Wildberries.
- **Управление ставками**: Понижение или восстановление ставок кампаний до заданных значений.
- **Обновление бюджетов**: Автоматическое пополнение бюджетов кампаний с низким остатком.
- **Запуск приостановленных кампаний**: Включение остановленных кампаний с предварительным пополнением бюджета.
- **Хранение данных**: Сохранение информации о кампаниях, товарах и юридических лицах в PostgreSQL.
- **Асинхронная обработка**: Использование `asyncio` и `aiohttp` для параллельных запросов к API.
- **Логирование и мониторинг**: Логи уровня `INFO` и интеграция с Sentry для отслеживания ошибок.

## Начало работы

Для запуска сервиса необходимо настроить окружение, установить зависимости и задеплоить функции в Google Cloud Functions.

### Настройка локально

1. **Создание виртуального окружения**:
   Создайте виртуальное окружение с помощью Poetry:
   ```bash
   poetry env use python3.11
   ```
   - Активируйте его:
   ```
   poetry shell
   ```

2. **Установка зависимостей: Установите зависимости через Poetry:**
    ```
   poetry install
   ```
   
3. **Настройка Google Cloud SDK: Установите и настройте Google Cloud SDK:**
    ```
    gcloud init
    gcloud auth application-default login
   ```
   
4. **Настройка переменных окружения: Скопируйте файл .env-example в .env и заполните его:**
    ```
   cp .env-example .env
   ```
   
5. **Инициализация базы данных: Примените миграции с помощью Alembic:**
    ```
   poetry run alembic upgrade head
   ```
   
## Деплой в Google Cloud Functions
1. **Деплой функций: Задеплойте каждую функцию в Google Cloud Functions. Пример для entrypoint_update_adv_status:**
    ```
   gcloud functions deploy entrypoint_update_adv_status \
    --gen2 \
    --trigger-http \
    --runtime python311 \
    --entry-point entrypoint_update_adv_status \
    --region europe-west1 \
    --source . \
    --set-env-vars PROJECT_ID=$(gcloud config get-value project) \
    --memory 1G \
    --cpu 1 \
    --min-instances 1 \
    --vpc-connector product-card \
    --timeout 240s
   ```

## Дополнительная информация
**API Wildberries:**
- https://advert-api.wildberries.ru/adv/v1/promotion/adverts: Получение списка кампаний.
- https://advert-api.wildberries.ru/adv/v2/fullstats: Статистика кампаний.
- https://advert-api.wildberries.ru/adv/v0/cpm: Обновление ставок.
- https://advert-api.wildberries.ru/adv/v1/budget: Получение бюджета.
- https://advert-api.wildberries.ru/adv/v1/budget/deposit: Пополнение бюджета.
- https://advert-api.wildberries.ru/adv/v0/start|pause: Управление статусом кампаний.

**Функции сервиса:**
- adjustment-ad-bid: Управление ставками.
- sync-advertising-campaigns: Синхронизация кампаний.
- update_adv_info: Обновление информации о кампаниях из внешних данных.
- auto_budget_update: Автоматическое обновление бюджетов.
- get_adv_campaigns: Получение списка кампаний.
- users_adv_campaigns: Управление пользователями.
- entrypoint_update_adv_status: Обновление статуса кампаний по юр. лицу.

**База данных: PostgreSQL с таблицами:**
- legal_entities: Юридические лица.
- advertising_campaigns: Рекламные кампании.
- products: Товары.
- products_advertising_campaigns_associations: Связь товаров и кампаний.
- api_keys: API-ключи для Wildberries.
- users: Пользователи системы.

**Алгоритм работы**
- Синхронизация данных о кампаниях через API Wildberries.
- Обновление ставок и бюджетов на основе заданных условий.
- Сохранение изменений в базе данных.
- Автоматическое управление статусом кампаний (включение/выключение).
- Логирование всех операций и ошибок.