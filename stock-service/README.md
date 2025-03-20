# Сервис Stock Service

**Stock Service** — это сервис для мониторинга и управления запасами товаров на маркетплейсах Wildberries и Ozon. Он реализован как облачная функция Google Cloud Functions, интегрируется с базой данных через SQLAlchemy, использует Redis для кэширования и Google Cloud Pub/Sub для публикации событий. Сервис отслеживает остатки, продажи и уведомляет о низких запасах или изменениях статуса FBO/FBS.

## Функциональные возможности

- **Мониторинг остатков**: Проверка текущих запасов (FBO/FBS) через API Wildberries и Ozon.
- **Анализ продаж**: Расчет продаж и доступности товаров за период (фактические и теоретические данные).
- **Уведомления**: Генерация событий при низких остатках, отсутствии или поступлении FBO через Pub/Sub.
- **Хранение данных**: Сохранение информации о товарах, SKU, продажах и уведомлениях в базе данных.
- **Статусы товаров**: Классификация товаров как "ordinary", "new", "absolute_new" на основе истории продаж.
- **Асинхронность**: Использование `asyncio` и ограничение параллелизма для обработки запросов.

## Начало работы

Для запуска сервиса необходимо настроить окружение, установить зависимости и задеплоить функцию в Google Cloud Functions.

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
   
3. **Настройка переменных окружения: Скопируйте файл .env-example в .env и заполните его:**
    ```
   cp .env-example .env
   ```
   
4. Локальный запуск функции для тестирования: Запустите функцию через functions-framework:
    ```
   functions-framework --target check_stocks --port 8080
   ```
   
    - Отправьте тестовый запрос:
   ```
   curl -X POST http://localhost:8080?legal_entity=ООО%20Амодекор \
    -H "Content-Type: application/json"
   ```
   
## Деплой в Google Cloud Functions
1. Настройка Google Cloud SDK: Установите и настройте Google Cloud SDK:
    ```
   gcloud init
   ```
   
2. Деплой функции: Выполните команду для деплоя:
    ```
   gcloud functions deploy stock-service \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point check_stocks \
    --region=europe-west1 \
    --source=. \
    --set-env-vars PROJECT_ID=$(gcloud config get-value project) \
    --vpc-connector=product-card \
    --max-instances=5 \
    --cpu=2 \
    --memory=2G \
    --timeout=3600s \
    --allow-unauthenticated
   ```
   
3. Передача переменных окружения: Добавьте дополнительные переменные окружения в команду деплоя, если требуется:
    ```
   gcloud functions deploy stock-service \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point check_stocks \
    --region=europe-west1 \
    --source=. \
    --set-env-vars PROJECT_ID=$(gcloud config get-value project),DATABASE_URL=your_db_url,REDIS_URL=your_redis_url \
    --vpc-connector=product-card \
    --max-instances=5 \
    --cpu=2 \
    --memory=2G \
    --timeout=3600s \
    --allow-unauthenticated
   ```
   
## Дополнительная информация
- База данных: Использует SQLAlchemy с таблицами api_keys, skus, products, sales, notifications.
- Redis: Кэширует уведомления с TTL 15 часов для предотвращения дублирования.
- Pub/Sub: Публикует события LowOnStock, StockFBOOver, StockFBOReceived в соответствующие топики.
- События: Обрабатываются асинхронно через шину событий (EVENT_HANDLERS и COMMAND_HANDLERS).
- Логика уведомлений:
- Уведомление о низких остатках (LowOnStock) отправляется при соотношении продаж к запасам выше порогов (например, 12.2 или 6.2).
- Уведомления об FBO (StockFBOOver/StockFBOReceived) отправляются при изменении остатков FBO ниже/выше 10.