# Go: что выносить в `libs/go/*`

Цель: уменьшать дублирование между сервисами без “god-lib” и без протечки бизнес-логики конкретного домена.

Список согласованных внешних библиотек/инструментов:
- `github.com/codex-k8s/kodex-guidelines-common-ru/external_dependencies_catalog.md`

## Когда выносить

- код нужен >= 2 сервисам;
- нужен единый стандарт поведения (логирование/метрики/otel, middleware, клиенты);
- API библиотеки можно сделать минимальным и стабильным.

## Что обычно выносим

- `libs/go/observability/*` — логгер, метрики, OTel helpers.
- `libs/go/auth/*` — OAuth/session helpers и безопасность.
- `libs/go/crypto/*` — шифрование/расшифровка секретов и токенов.
- `libs/go/db/*` — общие DB helpers (tx, pagination, jsonb/pgvector утилиты).
- `libs/go/k8s/*` — клиентские адаптеры и шаблоны работы с Kubernetes API.
- `libs/go/repo/*` — общий слой provider интерфейсов для GitHub/GitLab.
- `libs/go/mcp/*` — общий слой MCP tool contracts и helpers.

## Что запрещено выносить

- доменные правила конкретного сервиса;
- транспортные DTO продукта (для этого есть `proto/` и OpenAPI/AsyncAPI контракты);
- тяжёлые зависимости ради одной функции.

## Контракты транспорта

- gRPC правила см. `protobuf_grpc_contracts.md`.
- Ошибки см. `error_handling.md`.
