# Architectural Decision Records

Decisões arquiteturais do Sistema de Webhooks de Notificação de Pedidos, uma por arquivo, no formato MADR e com nome `ADR-NNN-titulo-em-kebab-case.md`.

| ADR | Decisão |
| --- | --- |
| [ADR-001](ADR-001-outbox-no-mysql.md) | Outbox no MySQL |
| [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md) | Worker em processo separado com polling |
| [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md) | Retry com backoff exponencial e DLQ |
| [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md) | HMAC-SHA256 com secret por endpoint |
| [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md) | Entrega at-least-once com X-Event-Id |
| [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md) | Reuso dos padrões do projeto |
| [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md) | Snapshot do payload na inserção |
| [ADR-008](ADR-008-uma-linha-de-outbox-por-evento-e-webhook.md) | Uma linha de outbox por evento e webhook (proposto) |
