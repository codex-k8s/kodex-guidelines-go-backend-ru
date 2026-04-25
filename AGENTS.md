# Инструкции для агентов

## Назначение репозитория

Этот репозиторий содержит инженерные правила для Go backend, которые можно использовать в разных проектах.

## Правила работы

- Ответы пользователю и оформление PR писать на русском языке.
- Тексты commit message писать на английском языке.
- Прямой push в `main` запрещён: изменения вносятся через отдельную ветку и PR.
- Если меняется файл в этом репозитории, проверить, нужно ли синхронизировать локальную копию в `github.com/codex-k8s/kodex` (`docs/design-guidelines/go/**`).
- Репозиторий должен оставаться самодостаточным: не добавлять обязательных ссылок на другие пакеты правил.

## Состав репозитория

Документы для Go backend.

- `check_list.md` — чек-лист перед PR для Go изменений.
- `services_design_requirements.md` — структура сервиса, домен/кастеры, repo+SQL правила, OpenAPI/AsyncAPI.
- `infrastructure_integration_requirements.md` — Postgres/Redis/секреты/миграции (goose) и запреты.
- `observability_requirements.md` — логи/трейсы/метрики (OTel/Jaeger/Prometheus).
- `protobuf_grpc_contracts.md` — правила gRPC `.proto` как транспортного контракта.
- `rest.md` — REST стек (echo + OpenAPI validation + codegen + swagger UI).
- `grpc.md` — gRPC (границы, контракты, ссылки на codegen).
- `websockets.md` — WebSocket (контракт AsyncAPI, правила сервера).
- `code_generation.md` — обязательные правила и команды кодогенерации.
- `code_commenting_rules.md` — правила комментариев в Go.
- `error_handling.md` — обязательные правила обработки ошибок в Go.
- `libraries.md` — что выносить в `libs/go/*` и как.

Текущее применение в `kodex`:
- Kubernetes интеграция только через `client-go` и адаптеры.
- Репозитории (GitHub/GitLab) только через provider-интерфейсы.
- Оркестрация процессов event/webhook-driven, без workflow-first зависимостей.
- Состояние процессов и синхронизация pod'ов — через PostgreSQL (`JSONB` + `pgvector`).
- Проектное планирование и документационная каноника задаются корневым `AGENTS.md` платформы `github.com/codex-k8s/kodex` и актуальной проектной документацией, а не этим техническим гайдом.
