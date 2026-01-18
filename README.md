# Производственная практика (3 семестр): глоссарий терминов ВКР (gRPC + Protobuf)

Тема ВКР: **«Сравнительный анализ технологии Web Components и современных фронтенд‑фреймворков для разработки пользовательских интерфейсов»**.

В репозитории реализован мини‑проект по образцу «2 сервиса (рекомендация книг)», но вместо рекомендателя — **глоссарий терминов**.

## 1. Архитектура

**Сервис 1 — `glossary` (gRPC)**
- Хранит термины (JSON‑хранилище в volume).
- Предоставляет CRUD и поиск.
- Отдает семантический граф (узлы + ребра по `related_ids`).

**Сервис 2 — `web` (HTTP)**
- HTTP‑шлюз на FastAPI.
- Рендерит простую HTML‑страницу со списком/поиском.
- Общается с `glossary` по gRPC.

Стек: **Python + gRPC + Protocol Buffers + Docker + docker‑compose**.

## 2. API (protobuf)

Файл: [`protobufs/glossary.proto`](protobufs/glossary.proto)

Методы:
- `UpsertTerm` — создать/обновить термин
- `GetTerm` — получить термин по ID
- `DeleteTerm` — удалить термин
- `ListTerms` — постраничный список
- `SearchTerms` — поиск по подстроке в термине/описании/тегах
- `GetGraph` — экспорт графа (nodes + edges)

## 3. Запуск локально (Docker)

### 3.1. Сборка и старт

```bash
docker-compose build
docker-compose up
```

Сервисы:
- gRPC: `localhost:50051`
- Web UI: `http://localhost:8000`

### 3.2. Наполнение тестовыми терминами

В отдельном терминале:

```bash
# (опционально) локально в venv
python -m pip install grpcio grpcio-tools

python tools/seed.py --host localhost --port 50051
```

После этого откройте `http://localhost:8000` и проверьте поиск.

## 4. Проверка работы (примеры)

### 4.1. Через Web UI
- Список терминов: `http://localhost:8000/`
- Поиск: `http://localhost:8000/?q=shadow`
- Просмотр термина: `http://localhost:8000/term/<id>`
- Граф: `http://localhost:8000/graph`

### 4.2. Через HTTP API шлюза
- `GET /api/terms` — список (JSON)
- `GET /api/terms/{id}` — термин
- `GET /api/search?q=...` — поиск
- `GET /api/graph` — граф

## 5. Структура репозитория

```
.
├── docker-compose.yml
├── protobufs/
│   └── glossary.proto
├── glossary_service/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── datastore.py
│   └── server.py
├── web_gateway/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app.py
└── tools/
    └── seed.py
```

## 6. Развертывание в интернете (опционально)

### Вариант A: Fly.io / Render / Railway
1. Публикуете репозиторий на GitHub.
2. Выбираете деплой «Docker».
3. Экспортируете наружу только HTTP‑шлюз (порт 8000).
4. `glossary` оставляете как приватный сервис в одной сети.

> Для продакшена желательно заменить JSON на БД (SQLite/PostgreSQL) и добавить аутентификацию.

## 7. Скриншоты

В отчет (README) обычно добавляют:
- скрин главной страницы со списком терминов
- скрин поиска
- скрин графа

Добавьте изображения в папку `docs/` и вставьте в README, например:

```md
![Web UI](docs/ui.png)
```
