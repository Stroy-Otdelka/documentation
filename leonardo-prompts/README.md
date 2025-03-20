# Сервис Leonardo Prompts

Сервис **Leonardo Prompts** предоставляет API для генерации изображений с использованием модели Leonardo AI. Он реализован как облачная функция Google Cloud Function, принимает текстовые запросы (prompts) и возвращает URL сгенерированного изображения. Сервис поддерживает асинхронную обработку и включает утилиту для скачивания ранее сгенерированных изображений.

## Функциональные возможности

- **Генерация изображений**: Создание изображений на основе текстового prompt'а через API Leonardo AI.
- **Асинхронная обработка**: Использование `asyncio` и `aiohttp` для асинхронных запросов.
- **Облачная функция**: Деплой в Google Cloud Functions с HTTP-триггером.
- **Скачивание изображений**: Утилита для массовой загрузки сгенерированных изображений с аккаунта Leonardo AI.

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
   
2. **Установка зависимостей:**
    - Установите зависимости через Poetry:
   ```
   poetry install
   ```
   
3. **Настройка переменных окружения:**
    - Скопируйте файл .env-example в .env:
   ```
   cp .env-example .env
   ```
   
    Укажите в .env:

    - LEONARDO_KEY: API-ключ для Leonardo AI.
    - OPENAI_KEY: API-ключ для OpenAI (опционально, если используется).

4. **Локальный запуск для тестирования:**
    - Запустите функцию локально с помощью functions-framework:
   ```
   functions-framework --target entrypoint --port 8080
   ```
   - Отправьте тестовый запрос через curl:
   ```
   curl -X POST http://localhost:8080 \
    -H "Content-Type: application/json" \
    -d '{"prompt": "A futuristic city at night"}'
   ```
   
5. **Скачивание изображений:**
    - Запустите утилиту для загрузки изображений:
   ```
   python src/saving_images.py
   ```
   - Изображения будут сохранены в папку downloaded_images.

## Деплой в Google Cloud Functions
1. **Настройка Google Cloud SDK:**
    - Установите и настройте Google Cloud SDK:
   ```
   gcloud init
   ```

2. **Деплой функции:**
    - Выполните команду для деплоя в Google Cloud Functions:
   ```
   gcloud functions deploy leonardo-prompts-service \
    --gen2 \
    --trigger-http \
    --runtime python311 \
    --entry-point entrypoint \
    --region europe-west1 \
    --source .
   ```
    - --gen2: Использует второе поколение Cloud Functions.
    - --trigger-http: Запуск по HTTP-запросу.
    - --runtime python311: Версия Python.
    - --entry-point entrypoint: Точка входа (функция в main.py).
    - --region europe-west1: Регион деплоя.
    - --source .: Исходный код из текущей директории.

3. **Передача переменных окружения:**
    - Настройте переменные окружения через Google Cloud Console или добавьте их в команду деплоя:
    ```
   gcloud functions deploy leonardo-prompts-service \
    --gen2 \
    --trigger-http \
    --runtime python311 \
    --entry-point entrypoint \
    --region europe-west1 \
    --source . \
    --set-env-vars "LEONARDO_KEY=your_leonardo_key"
   ```
   
## Дополнительная информация
- Скачивание изображений: Утилита saving_images.py загружает все изображения пользователя, сортирует их по дате создания и сохраняет в формате JPEG.
- Обработка ошибок: Логируются предупреждения при сбоях генерации или скачивания.