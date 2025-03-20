# Сервис Designer App Backend

**Designer App Backend** — это бэкенд для приложения дизайнеров, состоящий из набора облачных функций Google Cloud Functions и скрипта-демона для рекурсивного сканирования файловой системы. Сервис интегрируется с Elasticsearch для поиска изображений, Google Cloud Storage для хранения файлов, Redis для кэширования и использует Leonardo AI и ChatGPT для генерации и анализа изображений.

## Функциональные возможности

- **Поиск изображений**: Поиск по тегам и запросам через Elasticsearch с поддержкой автодополнения.
- **Генерация изображений**: Расширение промптов через ChatGPT и генерация изображений через Leonardo AI с последующим анализом.
- **Хранение данных**: Загрузка изображений в Google Cloud Storage и индексация метаданных в Elasticsearch.
- **Кэширование**: Использование Redis для кэширования результатов поиска и тегов.
- **Сканирование файлов**: Рекурсивное сканирование локальных директорий для обработки изображений (JPG, PNG, ARW и др.).
- **Обработка изображений**: Сжатие изображений в WebP и генерация тегов через внешний сервис OpenAI.

## Начало работы

Для запуска сервиса необходимо настроить окружение, установить зависимости и задеплоить функции в Google Cloud Functions или запустить скрипт-демон локально.

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
   poetry install --with crawler,google
   ```
   - Для Windows дополнительно установите:
   ```
   pip install pywin32==308
   ```
   
3. **Настройка переменных окружения: Скопируйте файл .env-example в .env:**
    ```
   cp .env-example .env
   ```

**Укажите в .env:**
- PATH_TO_START
- INDEX_NAME
- CLOUD_ID
- ELASTIC_API_KEY
- BUCKET_NAME
- REDIS_URL
- CHATGPT_FUNCTION_URL
- LEONARDO_FUNCTION_URL

4. **Локальный запуск демона: Запустите скрипт для сканирования и обработки изображений:**
    ```
   python main.py
   ```
   
5. **Локальный запуск функций для тестирования: Запустите одну из функций через functions-framework:**
    ```
   functions-framework --target search_images --port 8080
   ```
   - Отправьте тестовый запрос:
   ```
   curl -X POST http://localhost:8080 \
    -H "Content-Type: application/json" \
    -d '{"query": "кухня", "keywords": ["белые стены"]}'
   ```


## Деплой в Google Cloud Functions
1. **Настройка Google Cloud SDK: Установите и настройте Google Cloud SDK:**
    ```
   gcloud init
   ```

2. **Деплой функций: Выполните команды для деплоя каждой функции:**
    - Поиск изображений:
   ```
   gcloud functions deploy search_images \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point search_images \
    --region europe-west1 \
    --source .
   ```
   
    - Обновление изображения:
   ```
   gcloud functions deploy update_image \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point update_image \
    --region europe-west1 \
    --source .
   ```
   
    - Автодополнение тегов:
   ```
   gcloud functions deploy suggest_image_search \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point suggest \
    --region europe-west1 \
    --source .
   ```
   
    - Генерация изображений:
    ```
   gcloud functions deploy generate_leonardo \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point generate_handler \
    --region europe-west1 \
    --source .
   ```
   
    - Получение уникальных тегов:
    ```
   gcloud functions deploy get_unique_tags \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point get_unique_tags \
    --region europe-west1 \
    --source .
   ```
   
3. **Передача переменных окружения: Добавьте переменные окружения в команду деплоя:**
    ```
   gcloud functions deploy search_images \
    --gen2 \
    --trigger-http \
    --runtime python312 \
    --entry-point search_images \
    --region europe-west1 \
    --source . \
    --set-env-vars "INDEX_NAME=your_index,CLOUD_ID=your_cloud_id,ELASTIC_API_KEY=your_key,REDIS_URL=your_redis_url,BUCKET_NAME=your_bucket"
   ```
   - **Повторите для каждой функции.**

## Дополнительная информация
- Elasticsearch: Используется для хранения метаданных и поиска с кастомными анализаторами для тегов (русский стемминг, edge_ngram).
- Google Cloud Storage: Хранит сжатые изображения в формате WebP.
- Redis: Кэширует результаты поиска и тегов (TTL 4-10 часов).
- Сканирование: Поддерживает форматы JPG, PNG, ARW и др., с разрешением .lnk (Windows) и симлинков.
- Генерация тегов: Через внешний сервис OpenAI с заданным списком категорий.