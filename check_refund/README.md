# Сервис Check Refund

**Check Refund** — это сервис для мониторинга возвратов товаров на Wildberries, реализованный как облачная функция Google Cloud Functions. Он собирает данные о возвратах через API Wildberries и записывает их в Google Sheets для дальнейшего анализа. Сервис поддерживает асинхронные запросы, кэширование учетных данных Google и повторные попытки при ошибках.

## Функциональные возможности

- **Сбор данных о возвратах**: Получение информации о возвратах за последние 30 дней через API Wildberries.
- **Запись в Google Sheets**: Обновление двух листов — "Текущие" (детализация по SKU) и "Прошлые" (общие данные за период).
- **Асинхронность**: Использование `aiohttp` для параллельных запросов к API.
- **Авторизация**: Кэширование Google credentials с помощью `GoogleAuth` и повторные попытки при сбоях.
- **Обработка ошибок**: Повторные попытки запросов с задержкой при возникновении ошибок.

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
   
4. **Локальный запуск функции для тестирования: Запустите функцию через functions-framework:**
    ```
   functions-framework --target entrypoint --port 8080
   ```
   - Отправьте тестовый запрос:
   ```
   curl -X POST http://localhost:8080 \
    -H "Content-Type: application/json"
   ```
   
## Деплой в Google Cloud Functions
1. **Настройка Google Cloud SDK: Установите и настройте Google Cloud SDK:**
    ```
   gcloud init
   ```
   
2. **Деплой функции: Выполните команду для деплоя:**
    ```
   gcloud functions deploy check-refund \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point entrypoint \
    --region europe-west1 \
    --source . \
    --set-env-vars "API_KEYS={\"amodecor\":{\"wb\":\"your_token\"}},AUTH_URL=https://europe-west1-yellduck.cloudfunctions.net/get-credentials,SHEET_ID=1gKXFhFt_BoHshZHenkmTk9PPVRCNc6p7mEEq0sMh9TY,SHEET_SECRET=SHEET_SERVICE_INFO" \
    --allow-unauthenticated
   ```
   
    - **Замените your_token на действительный API-ключ Wildberries.**

# Дополнительная информация

API Wildberries: Используется эндпоинт https://seller-analytics-api.wildberries.ru/api/v1/analytics/goods-return для получения данных о возвратах.

Google Sheets:
- Лист "Текущие": Детализация возвратов по SKU (юрлицо, артикул, количество возвратов).
- Лист "Прошлые": Общие данные за период (юрлицо, период, общее количество возвратов).

GoogleAuth: Кэширует credentials на 30 минут для уменьшения запросов к сервису авторизации. 

Повторные попытки: При ошибках API выполняется до 5 попыток с задержкой 60 секунд, при ошибках авторизации — до 10 попыток с рандомной задержкой 5-20 секунд.