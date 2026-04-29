# REST/HTTP в Go

Цель: единый стек, единый контракт (OpenAPI YAML), единая валидация и единая точка обработки ошибок.

## Стек (фиксирован)
- HTTP сервер: `github.com/labstack/echo/v5`
- OpenAPI (загрузка спеки + request/response validation): `github.com/getkin/kin-openapi`
- Codegen по OpenAPI: `github.com/oapi-codegen/oapi-codegen/v2/cmd/oapi-codegen@latest`
- Swagger UI (отдача спеки/доков): `github.com/swaggest/swgui/v5emb`

## Где хранится OpenAPI
- Спека сервиса: `api/server/api.yaml` (для `services/external/*` и `services/staff/*`).
- (Опционально) JSON schema/примеры можно держать рядом в `api/server/*`, но источником правды остаётся OpenAPI YAML.

## Как используем `kin-openapi`
Флоу (по смыслу):
1) На старте загружаем `api/server/api.yaml` (разрешая external refs при необходимости) и валидируем документ.
2) На входе запроса валидируем request по операции:
   - `openapi3.Loader` -> `doc.Validate(ctx)` -> `router.FindRoute(req)` -> `openapi3filter.ValidateRequest(...)`.
3) На выходе (опционально, минимум в dev/stage) валидируем response:
   - `openapi3filter.ValidateResponse(...)`.
4) Ошибки валидации нормализуем в единый безопасный HTTP контракт ошибок (см. `error_handling.md`).

Важно:
- Handler’ы не повторяют schema/type validation (это ответственность OpenAPI middleware).
- Сообщение ошибки валидатора не считается публичным контрактом; наружу отдаём безопасные поля (например, `message/loc/field`).

## Лимиты HTTP сервера

HTTP server должен иметь явные лимиты и таймауты, заданные через конфигурацию:

- `ReadTimeout`, `ReadHeaderTimeout`, `WriteTimeout`, `IdleTimeout`;
- максимальный размер request body, если endpoint принимает body;
- максимальное число in-flight requests на реплику;
- размер очереди или буфера ожидания, если проект допускает кратковременное накопление запросов;
- таймаут ожидания места в буфере и безопасный ответ при переполнении.

Middleware входящего лимита должно выполняться до дорогой валидации, обращения к БД и запуска фоновых операций.
Сервис обязан публиковать метрики активных HTTP-запросов, отказов из-за лимита, заполнения буфера
и длительности ожидания. Эти метрики должны быть пригодны для HPA или другого механизма автомасштабирования,
если проект его использует.

## Codegen (OpenAPI -> Go)
Генерация запускается через `make` (см. `code_generation.md`), а конфигурация/шаблоны — в репозитории.

Рекомендуемый подход:
- хранить конфиги/шаблоны генерации централизованно (project-wide) в `tools/codegen/openapi/`;
- генерировать отдельно types + server/client (по необходимости), не смешивая с бизнес-логикой;
- не редактировать сгенерированный файл руками.

Пример запуска:
```bash
make gen-openapi-go SVC=services/<zone>/<service>
```

## Swagger UI
Правило:
- В `external|staff` сервисах отдаём Swagger UI, который показывает `api/server/api.yaml` (и статически, и/или через эндпоинт).
- В prod это включается осознанно (по политике безопасности продукта), в dev/stage — обычно включено.
