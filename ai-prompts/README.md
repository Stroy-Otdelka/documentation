# Сервис AI Prompts

Сервис **AI Prompts** предоставляет API для генерации текстовых ответов с использованием модели *GPT-4-Turbo* от OpenAI. Он реализован как облачная функция Google Cloud Function, принимает текстовые запросы (prompts) и опционально ссылки на изображения, возвращая сгенерированный ответ. Сервис поддерживает асинхронную обработку и интеграцию с OpenAI.

## Функциональные возможности

- **Генерация текста**: Создание текстовых ответов на основе переданного prompt'а с использованием *GPT-4-Turbo*.
- **Анализ изображений**: Возможность передачи ссылок на изображения для анализа и генерации текста на их основе.
- **Асинхронная обработка**: Использование `asyncio` и `AsyncOpenAI` для асинхронных запросов к OpenAI.
- **Облачная функция**: Деплой в Google Cloud Functions с HTTP-триггером.
- **Конфигурация через окружение**: Использование `.env` для хранения API-ключей и настроек.

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

    - OPENAI_KEY: API-ключ для OpenAI.
    - PROJECT_ID: ID проекта Google Cloud.

4. **Локальный запуск для тестирования:**
    - Запустите функцию локально с помощью functions-framework:
    ```
   functions-framework --target entrypoint --port 8080
   ```
   
    - Отправьте тестовый запрос через curl:
    ```
   curl -X POST http://localhost:8080 \
    -H "Content-Type: application/json" \
    -d '{"prompt": "Привет, как дела?", "image_urls": ["https://example.com/image.jpg"]}'
   ```
   
## Деплой в Google Cloud Functions
1. **Настройка Google Cloud SDK:**

    - Установите и настройте Google Cloud SDK:
   ```
   gcloud init
   ```
   
2. **Деплой функции:**
    - Выполните команду для деплоя в Google Cloud Functions:
   ```
   gcloud functions deploy ai-prompts-service \
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
   gcloud functions deploy ai-prompts-service \
    --gen2 \
    --trigger-http \
    --runtime python311 \
    --entry-point entrypoint \
    --region europe-west1 \
    --source . \
    --set-env-vars "OPENAI_KEY=your_openai_key,PROJECT_ID=your_project_id"
   ```
   
## Дополнительная информация
- Модель: Используется GPT-4-Turbo с параметрами: max_tokens=1000, temperature=0.4, top_p=0.5.
- Логи доступны в Google Cloud Console:
    ```
    gcloud functions logs read ai-prompts-service
    ```