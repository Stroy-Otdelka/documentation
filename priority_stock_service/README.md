# Сервис Priority Stock Service

**Priority Stock Service** — это сервис для проверки остатков приоритетных товаров из таблиц Google Sheets, реализованный как две облачные функции Google Cloud Functions: `check_tags_stock` и `check_listing_stock`. Он интегрируется с API для получения остатков, использует Redis для предотвращения дублирования проверок и публикует события в Google Cloud Pub/Sub при нулевых остатках.

## Функциональные возможности

- **Проверка остатков тегов**: Анализ товаров с тегами (родительских и дочерних) из Google Sheets, публикация событий в топик `out_stock_tags_wb` при нулевых остатках.
- **Проверка остатков листинга**: Анализ приоритетных артикулов Wildberries и Ozon из Google Sheets, публикация событий в топик `out_stock_listing` при нулевых остатках.
- **Интеграция с API**: Использование внешнего API `get-stocks` для получения текущих остатков по SKU.
- **Google Sheets**: Чтение данных из листов "Лист1" (теги) и "LISTING" (артикулы WB/Ozon).
- **Redis**: Кэширование проверок с TTL 1 день для исключения повторных запросов.
- **Pub/Sub**: Публикация событий о нулевых остатках с повторными попытками при ошибках.

## Начало работы

Для запуска сервиса необходимо настроить окружение, установить зависимости и задеплоить функции в Google Cloud Functions.

### Настройка локально

1. **Создание виртуального окружения**:
   Создайте виртуальное окружение с помощью Poetry:
   ```bash
   poetry env use python3.11
   ```
   
2. **Установка зависимостей: Установите зависимости через Poetry:**
    ```
   poetry install
   ```
   
3. **Настройка переменных окружения: Скопируйте файл .env-example в .env и заполните его:**
    ```
   cp .env-example .env
   ```
   
4. **Локальный запуск функций для тестирования:**
    - Для check_tags_stock:
   ```
   functions-framework --target check_tags_stock --port 8080
   ```
   - Отправьте тестовый запрос:
   ```
   curl -X POST http://localhost:8080 -H "Content-Type: application/json"
   ```
   - Для check_listing_stock:
   ```
   functions-framework --target check_listing_stock --port 8081
   ```
   
## Деплой в Google Cloud Functions
1. **Настройка Google Cloud SDK: Установите и настройте Google Cloud SDK:**
    ```
   gcloud init
   ```
   
2. **Деплой функций:**
    - Для check_tags_stock:
   ```
   gcloud functions deploy check_tags_stock \
    --gen2 \
    --trigger-http \
    --runtime python311 \
    --entry-point check_tags_stock \
    --region europe-west1 \
    --source . \
    --set-env-vars PROJECT_ID=$(gcloud config get-value project) \
    --vpc-connector product-card \
    --timeout 240
   ```
   - Для check_listing_stock:
   ```
   gcloud functions deploy check_listing_stock \
    --gen2 \
    --trigger-http \
    --runtime python311 \
    --entry-point check_listing_stock \
    --region europe-west1 \
    --source . \
    --set-env-vars PROJECT_ID=$(gcloud config get-value project) \
    --vpc-connector product-card \
    --timeout 240
   ```

3. **Добавление дополнительных переменных окружения: Если требуется, добавьте переменные в команду деплоя (например, SHEETS_ID, REDIS_URL):**
    ```
   gcloud functions deploy check_tags_stock \
    --gen2 \
    --trigger-http \
    --runtime python311 \
    --entry-point check_tags_stock \
    --region europe-west1 \
    --source . \
    --set-env-vars PROJECT_ID=$(gcloud config get-value project),SHEETS_ID=... \
    --vpc-connector product-card \
    --timeout 240
   ```
   
## Дополнительная информация
Google Sheets:
- Лист "Лист1": Содержит данные о товарах с тегами (родители в столбце C, дочерние в D).
- Лист "LISTING": Содержит пары артикулов Wildberries (столбец A) и Ozon (столбец B).

Redis: Кэширует проверенные SKU с TTL 24 часа для предотвращения повторных запросов.

Pub/Sub: Публикует события в топики out_stock_tags_wb и out_stock_listing с повторными попытками (до 40 секунд).
