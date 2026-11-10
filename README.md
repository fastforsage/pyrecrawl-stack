# 🚀 PyreCrawl MCP Stack (Docker)

Готовый к развертыванию Docker-образ для **[PyreCrawl](https://github.com/SanggonBoy/PyreCrawl)** — мощного, полностью бесплатного и self-hosted инструмента для глубокого веб-поиска, краулинга и извлечения данных.

PyreCrawl является отличной альтернативой Firecrawl, не требует API-ключей и умеет обходить сложные защиты (включая Cloudflare), используя невидимый браузер Chromium и «умную лестницу» фолбэков (HTTP -> Stealth Browser -> LLM).

Этот образ не только запускает PyreCrawl, но и автоматически оборачивает его в **[MCPO](https://github.com/open-webui/mcpo)** (MCP-to-OpenAPI proxy). Это позволяет легко подключать сервер к веб-интерфейсам (например, **Open WebUI**) по протоколу OpenAPI.

## ✨ Возможности

*   **🕵️‍♂️ Глубокий поиск:** Переход по ссылкам, обход блокировок и парсинг динамического контента.
*   **📄 Извлечение данных:** Преобразование страниц в чистый Markdown, извлечение структурированных данных.
*   **🔓 Без API-ключей:** Работает «из коробки» (использует DuckDuckGo и встроенные парсеры).
*   **🌐 HTTP-доступ (MCPO):** Встроенный прокси-сервер транслирует MCP-инструменты в стандартный REST API (порт `8000`).
*   **🛡️ Готов к жестким сайтам:** Включает `patchright` (Stealth-версия Playwright) с собственным браузером Chromium.
*   **⚙️ CI/CD:** Автоматическая сборка образа через GitHub Actions.

## 🚀 Быстрый старт

Создайте файл `docker-compose.yml`:

```yaml
services:
  pyrecrawl:
    image: docker.io/fastforsage/pyrecrawl-mcp:latest
    container_name: pyrecrawl
    ports:
      - "8000:8000"
    environment:
      - PYRECRAWL_NO_TELEMETRY=1
    restart: unless-stopped
    volumes:
      - pyrecrawl_cache:/root/.cache/pyrecrawl
    networks:
      - crawl-net   # Используем кастомную сеть для фикса MTU

volumes:
  pyrecrawl_cache:

networks:
  crawl-net:
    driver: bridge
    driver_opts:
      # ВАЖНО: Уменьшение MTU решает проблему с зависанием TLS/SSL рукопожатия 
      # (TimeoutError) при работе внутри Docker на некоторых хостингах.
      com.docker.network.driver.mtu: 1400
```

Затем выполните:

```bash
docker compose up -d
```

🔌 Интеграция с Open WebUI

Благодаря встроенному MCPO, сервер доступен по адресу http://<IP-вашего-сервера>:8000 (или http://pyrecrawl:8000, если Open WebUI в той же сети).

Как добавить в Open WebUI:

1. Перейдите в Settings -> Admin Settings -> Integrations (или Tools).
2. Нажмите + Add Connection.
3. Type (Тип): Выберите OpenAPI (Важно: не MCP, так как MCPO отдает OpenAPI-совместимую спецификацию).
4. URL: http://pyrecrawl:8000 (если они в одной сети Docker) или http://<IP_сервера>:8000.
5. Auth (Вход): Выберите None (пусто).
6. Сохраните. Инструменты (scrape, crawl, deep_research и др.) станут доступны моделям.

🛠️ Решенные проблемы (Under the Hood)

При разработке этого образа были решены несколько нетривиальных задач:

1. Сетевой конфликт (MTU и TLS): По умолчанию Docker использует MTU 1500. На некоторых хостингах это приводит к тому, что TCP-соединение устанавливается, но TLS-рукопожатие (HTTPS) зависает с TimeoutError. В docker-compose.yml мы уменьшили MTU до 1400.
2. Отсутствующие библиотеки: В базовый образ добавлены libatomic1 и системные зависимости для корректной работы Node.js и Chromium.
3. Конфликт версий MCPO и FastAPI: Зафиксирована рабочая версия mcpo==0.0.7 (в более новых версиях есть баг с передачей аргумента ssl_certfile в FastAPI, вызывающий падение).
4. Patchright Chromium: Стандартный Playwright и patchright (Stealth-браузер) используют разные версии Chromium. В Dockerfile добавлена команда patchright install chromium для предотвращения ошибки Executable doesn't exist.

⚠️ Ограничения (Авито, Ozon и др.)

PyreCrawl отлично справляется с 90% сайтов (новости, блоги, документация, Wikipedia).

Однако, сайты с агрессивной защитой (Авито, Wildberries, Ozon, Instagram) могут блокировать запросы. Вы получите ответ HTTP 200 OK, но внутри будет страница «Доступ ограничен из-за проблем с IP». 
Это не ошибка парсера, а блокировка IP-адреса вашего сервера (дата-центра). 
Для обхода таких сайтов требуются резидентные прокси (желательно российские) или специализированные API (ScraperAPI и т.д.).

📂 Сборка образа

Образ собирается автоматически через GitHub Actions при каждом пуше в ветку main. Исходный Dockerfile находится в корне репозитория.

---

Собрано с любовью к Open Source и благодаря упорству в дебаггинге.
