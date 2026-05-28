        # postgresql — Специализированные индексы: GIN, GiST, BRIN, hash, partial, expression

        Homework-шаблон для урока **l2_specialized_indexes** (Специализированные индексы: GIN, GiST, BRIN, hash, partial, expression) на платформе Vibe Learn.

        ## Что делать

        Дано: testcontainers PG + миграция, создающая events с 1М синтетических строк, в т.ч.
jsonb и geography (опционально через PostGIS). Реализуй на Go бенчмарк:
1) Создай 4 разных индекса (B-tree, GIN, GiST, BRIN) — каждый CONCURRENTLY.
2) Замерь латентность типового запроса под каждым индексом + размер индекса (pg_relation_size).
3) Сравни latency и место в Markdown-отчёте.
Тесты в template проверят корректность создания индексов, парсинг pg_relation_size и
отсутствие race-conditions.

## Контекст (из transfer-задачи урока)

SaaS-аналитика. Таблица `events`:

```
id          bigserial PK
tenant_id   bigint NOT NULL
user_id     bigint
event_type  text     -- 'page_view','click','purchase',... (десятки типов)
payload     jsonb    -- произвольный контекст (UTM, product_id, deviceModel, ...)
location    geography(POINT, 4326)  -- координаты, опционально
created_at  timestamptz NOT NULL DEFAULT now()
```

## Recap из урока

- **GIN — для «один документ → много значений»**: jsonb, массивы, tsvector, pg_trgm. Жирный и медленный на запись, но непобедим на своих кейсах.
- **GiST — для диапазонов и геометрии**: PostGIS, range типы (`tstzrange`, `int4range`). Поиски «пересекается с», «в радиусе».
- **BRIN — для append-only time-series**: мегабайты вместо гигабайт, почти нулевая цена записи. Работает только при корреляции колонка ↔ физический порядок.
- **Partial и Expression — модификаторы, а не отдельный метод**: можно прикрутить к любому из b-tree/gin/gist. Partial UNIQUE — мощнейший паттерн для soft-delete.
- **Выбирай индекс по типу запроса, а не «на всякий случай».** Каждый лишний индекс — penalty на каждый INSERT/UPDATE/DELETE.

        ## Как работать

        1. Платформа Vibe Learn создаёт копию этого репо в твоём GitHub-аккаунте по клику «Начать домашку» на странице урока (через GitHub `/generate`, codecrafters-pattern).
        2. Склонируй копию локально, реализуй TODO в `main.go`, прогони тесты, запушь.
        3. CI (`.github/workflows/ci.yml`) запускает `go vet` + `go test ./...` на каждый push. Платформа слушает результат через webhook от GitHub Actions и обновляет статус домашки на странице урока.

        ## Локальное окружение

        - Go 1.22+
        - Docker + docker-compose — `docker compose up -d` поднимает single-node PostgreSQL 16 на `localhost:5432` с healthcheck. DSN: `postgres://postgres:postgres@localhost:5432/postgres`. Переопределяется через env `DATABASE_URL`.

        ## Запуск

        ```bash
        # Поднять локальный PostgreSQL
        docker compose up -d

        # Прогнать тесты (интеграционный включается через PG_INTEGRATION=1)
        go test ./...
        PG_INTEGRATION=1 go test ./...

        # Запустить main (печатает marker; замени stub на реализацию)
        go run .
        ```

        ## Заметка автора

        Это baseline-шаблон, сгенерированный платформой. Бизнес-сущность задачи (что конкретно реализовать в `main.go`, какие тесты сделать строгими) расширяется по ходу итераций — параллельно с углублением теории урока.
