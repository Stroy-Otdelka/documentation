# Сервис Advertisement Info

**Advertisement Info** — это сервис для отслеживания рекламы на маркетплейсе Ozon. Он собирает данные о товарах в активных рекламных кампаниях (трафареты и продвижение в поиске), проверяет их наличие на складе и публикует события в Google Cloud Pub/Sub, если товар закончился, но продолжает рекламироваться.

## Функциональные возможности

- **Сбор данных о рекламе**: Получение списка SKU товаров в активных рекламных кампаниях Ozon через API.
- **Проверка остатков**: Проверка наличия товаров на складах Ozon (FBO и FBS).
- **Оповещения**: Публикация событий в топик `zero_stock_adv_ozon` при нулевых остатках рекламируемых товаров.
- **Асинхронная обработка**: Использование `asyncio` и `aiohttp` для параллельных запросов к API Ozon.
- **Поддержка нескольких продавцов**: Работа с данными юридических лиц (например, "Амодекор", "СтройОтделка", "Орион").
- **Логирование**: Логи уровня `INFO` и выше для отслеживания процесса и ошибок.

## Начало работы

Для запуска сервиса необходимо настроить окружение, установить зависимости и задеплоить функцию в Google Cloud Functions.

### Настройка локально

1. **Создание виртуального окружения**:
   Создайте виртуальное окружение с помощью Poetry:
   ```bash
   poetry env use python3.12
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
   
4. **Локальный запуск функции для тестирования: Запустите функцию с помощью functions-framework:**
   ```
   functions-framework --target run_adv_info --port 8080
   ```
   - Отправьте тестовый HTTP-запрос:
   ```
   curl -X POST http://localhost:8080 -H "Content-Type: application/json"
   ```
   
## Деплой в Google Cloud Functions
1. **Настройка Google Cloud SDK: Установите и настройте Google Cloud SDK:**
    ```
   gcloud init
   ```

2. **Деплой функции: Задеплойте функцию check-ozon-adv**
    ```
   gcloud functions deploy check-ozon-adv \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point run_adv_info \
    --region europe-west1 \
    --source . \
    --set-env-vars PROJECT_ID=$(gcloud config get-value project) \
    --memory 1G \
    --cpu 1 \
    --min-instances 0 \
    --max-instances 1 \
    --vpc-connector product-card \
    --timeout 600s
   ```
   
3. **Добавление API-ключей: Добавьте переменную окружения API_KEYS через Google Cloud Console или CLI:**
    ```
   gcloud functions deploy check-ozon-adv \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point run_adv_info \
    --region europe-west1 \
    --source . \
    --set-env-vars PROJECT_ID=$(gcloud config get-value project),API_KEYS='{"amodecor":{"ozon":{"client_id":"your_client_id","api_key":"your_api_key","client_adv_id":"your_adv_client_id","client_adv_secret":"your_adv_secret"}},"stroy_otdelka":{"ozon":{"client_id":"your_client_id","api_key":"your_api_key","client_adv_id":"your_adv_client_id","client_adv_secret":"your_adv_secret"}},"orion":{"ozon":{"client_id":"your_client_id","api_key":"your_api_key","client_adv_id":"your_adv_client_id","client_adv_secret":"your_adv_secret"}}}' \
    --memory 1G \
    --cpu 1 \
    --min-instances 0 \
    --max-instances 1 \
    --vpc-connector product-card \
    --timeout 600s
   ```
   
## Дополнительная информация

**API Ozon:**
- https://api-seller.ozon.ru/v3/product/list: Получение списка активных продуктов.
- https://api-seller.ozon.ru/v3/product/info/list: Получение детальной информации о продуктах.
- https://api-performance.ozon.ru/api/client/campaign: Получение списка рекламных кампаний.
- https://api-performance.ozon.ru/api/client/campaign/{id}/v2/products: Получение товаров в кампании.
- https://api-performance.ozon.ru/api/client/campaign/search_promo/v2/products: Получение товаров в продвижении в поиске.
- Pub/Sub: События публикуются в топик zero_stock_adv_ozon

**Алгоритм работы**
1. Получение списка SKU из активных рекламных кампаний для каждого продавца.
2. Проверка остатков каждого SKU через API Ozon.
3. Если остатки нулевые, создание события FoundZeroStockEvent.
4. Публикация события в топик zero_stock_adv_ozon.