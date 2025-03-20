# Сервис Purchases Service

Сервис **Purchases Service** отвечает за сбор данных о заказах с маркетплейса *Wildberries*, их обработку и запись результатов в Google Sheets. Сервис реализован как облачная функция Google Cloud Function, которая запускается по HTTP-запросу, извлекает данные из MongoDB и обрабатывает их с использованием асинхронных запросов.

## Функциональные возможности

- **Сбор данных из MongoDB**: Получение заказов Wildberries из базы данных за последние 31 день.
- **Обработка заказов**: Подсчет количества продаж по артикулам (SKU) с разбивкой на заказы с количеством = 1, > 1 и > 3.
- **Запись в Google Sheets**: Асинхронная загрузка обработанных данных в таблицу Google Sheets.
- **Интеграция с Google Cloud**: Использование Google Cloud Functions для деплоя и Google Auth для авторизации.

## Начало работы

Для запуска сервиса необходимо создать виртуальное окружение, установить зависимости и задеплоить его в Google Cloud Functions.

### Настройка локально

1. **Создание нового виртуального окружения**:

   Создайте новое виртуальное окружение с помощью Poetry:
   ```bash
   poetry env use python3.11
   ```
   
    - Активируйте окружение:
   ```
   poetry shell
   ```
   
2. **Установка зависимостей:**
    - Установите зависимости, необходимые для работы сервиса:
   ```
   pip install \
    functions-framework==3.8.2 \
    python-dotenv==1.0.1 \
    aiohttp==3.11.12 \
    google-api-python-client==2.161.0 \
    motor==3.7.0 \
    pymongo==4.11.1 \
    google-auth-oauthlib==1.2.1
   ```
   
3. **Копирование и настройка переменных окружения:**
    - Скопируйте файл окружения-примера .env-example в .env и заполните его:
   ```
   cp .env-example .env
   ```

4. **Локальный запуск для тестирования:**
    - Запустите сервис локально с помощью functions-framework:
   ```
   functions-framework --target purchases_response --port 8080
   ```
    - Отправьте тестовый HTTP-запрос (например, через curl или Postman):
   ```
   curl -X GET http://localhost:8080
   ```

## Деплой в Google Cloud Functions
1. Настройка Google Cloud SDK:
    - Убедитесь, что у вас установлен и настроен Google Cloud SDK:
   ```
   gcloud init
   ```
2. Деплой функции:
    - Выполните команду для деплоя сервиса в Google Cloud Functions:
   ```
   gcloud functions deploy order-purchases-service \
    --gen2 \
    --trigger-http \
    --runtime python311 \
    --entry-point purchases_response \
    --region europe-west1 \
    --source .
   ```
    - --gen2: Использует второе поколение Cloud Functions.
    - --trigger-http: Запуск по HTTP-запросу.
    - --runtime python311: Указывает версию Python.
    - --entry-point purchases_response: Точка входа (функция в main.py).
    - --region europe-west1: Регион деплоя.
    - --source .: Исходный код из текущей директории.


3. Передача переменных окружения:
    - Настройте переменные окружения в Google Cloud Console после деплоя или добавьте их в команду деплоя через --set-env-vars:
    ```
   gcloud functions deploy order-purchases-service \
    --gen2 \
    --trigger-http \
    --runtime python311 \
    --entry-point purchases_response \
    --region europe-west1 \
    --source . \
    --set-env-vars "MONGO_HOST=your_host,MONGO_PORT=27017,MONGO_USER=your_user,MONGO_PASS=your_pass,MONGO_DB=your_db,SHEET_ID=your_sheet_id,CREDS_URL=your_creds_url"
   ```
   
## Дополнительная информация
- MongoDB: Сервис извлекает заказы из коллекции, настроенной в MONGO_ORDERS_COLLECTION. Убедитесь, что данные о заказах Wildberries уже загружены (например, через data-orchestrator).
- Google Sheets: Результаты записываются в лист 'Продажи' с колонками: "Юр Лицо", "Артикул", "Количество продаж = 1", "Количество продаж > 1", "Количество продаж > 3".
- Авторизация: Используется Google Auth через внешнюю Cloud Function (CREDS_URL) для получения временных credentials с 30-минутным кэшированием.
- Обработка ошибок: При ошибках сервис возвращает HTTP-статус 500 с сообщением {"error": "Internal server error"}.

Если возникли проблемы, проверьте логи в Google Cloud Console (gcloud functions logs read order-purchases-service).