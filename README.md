# 🚀 PyreCrawl MCP Stack (Docker)

Готовый к развертыванию Docker-образ для **[PyreCrawl](https://github.com/SanggonBoy/PyreCrawl)** — мощного, полностью бесплатного и self-hosted инструмента для глубокого веб-поиска, краулинга и извлечения данных.

PyreCrawl является отличной альтернативой Firecrawl, не требует API-ключей и умеет обходить сложные защиты (включая Cloudflare), используя невидимый браузер Chromium с «умной лестницей» фолбэков.

Этот образ не только запускает PyreCrawl, но и автоматически оборачивает его в **[MCPO](https://github.com/open-webui/mcpo)** (MCP-to-OpenAPI proxy). Это позволяет подключать сервер к веб-интерфейсам (например, **Open WebUI**) по HTTP, а не только через локальный stdio.

## ✨ Возможности

*   **🕵️‍♂️ Глубокий поиск:** Переход по ссылкам, обход блокировок и парсинг динамического контента.
*   **📄 Извлечение данных:** Преобразование страниц в чистый Markdown, извлечение структурированных данных (таблиц, списков) по схеме.
*   **🔓 Без API-ключей:** Работает «из коробки» без необходимости регистрироваться в платных сервисах.
*   **🌐 HTTP-доступ (MCPO):** Встроенный прокси-сервер транслирует MCP-инструменты в стандартный REST API (порт `8000`).
*   **🐳 Docker-ready:** Полная изоляция зависимостей и браузера.

## 🚀 Быстрый старт

Для запуска вам понадобится только установленный Docker. Образ уже собран и доступен на Docker Hub.

**Вариант 1: Запуск одной командой (Docker CLI)**

```bash
docker run -d \
  --name pyrecrawl \
  -p 8000:8000 \
  -e PYRECRAWL_NO_TELEMETRY=1 \
  -v pyrecrawl_cache:/root/.cache/pyrecrawl \
  --restart unless-stopped \
  fastforsage/pyrecrawl-mcp:latest
```

**Вариант 2: Запуск через Docker Compose**

Создайте файл `docker-compose.yml`:

```yaml
services:
  pyrecrawl:
    image: fastforsage/pyrecrawl-mcp:latest
    container_name: pyrecrawl
    ports:
      - "8000:8000"
    environment:
      - PYRECRAWL_NO_TELEMETRY=1
    restart: unless-stopped
    volumes:
      - pyrecrawl_cache:/root/.cache/pyrecrawl

volumes:
  pyrecrawl_cache:
```

Затем выполните:

```bash
docker compose up -d
```

## 🔌 Интеграция с Open WebUI (и другими клиентами)

Благодаря встроенному MCPO, сервер доступен по адресу `http://<IP-вашего-сервера>:8000`.

**Для добавления в Open WebUI:**
1. Перейдите в **Settings** -> **Admin Settings** -> **Integrations**.
2. Нажмите **+ Add Server** (добавить MCP-сервер).
3. Укажите URL: `http://localhost:8000` (если Open WebUI и PyreCrawl в одной сети Docker).
4. Сохраните. Инструменты (`scrape`, `crawl`, `extract`, `deep_research` и другие) станут доступны вашим моделям.

## 📂 О образе

*   **Базовый образ:** `ghcr.io/astral-sh/uv:python3.12-bookworm-slim`
*   **Установленные компоненты:** `pyrecrawl`, `mcpo`, браузер `Chromium` и все системные зависимости.
*   **Размер:** ~741.4 MB
*   **Docker Hub:** [fastforsage/pyrecrawl-mcp](https://hub.docker.com/r/fastforsage/pyrecrawl-mcp)

## 🛡️ Безопасность и конфиденциальность

Все запросы выполняются с вашего сервера. Стек полностью автономен, телеметрия PyreCrawl отключена по умолчанию (`PYRECRAWL_NO_TELEMETRY=1`). Кэш результатов сохраняется в Docker volume `pyrecrawl_cache` для ускорения повторных запросов.

---
*Собрано с любовью к Open Source.*
```
