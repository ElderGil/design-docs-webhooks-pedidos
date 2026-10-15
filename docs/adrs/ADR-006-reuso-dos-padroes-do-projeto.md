# ADR-006: Reuso dos padrões do projeto

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md)

## Status

Aceito. Larissa fechou com "Decisão: reuso máximo do que já existe" ([09:30] Larissa) e repetiu os itens no resumo final ([09:48] Larissa).

## Contexto

A codebase tem um padrão claro: cada domínio é um módulo em `src/modules` com controller, service, repository, routes e schemas ([09:27] Bruno). O módulo `src/modules/orders/` segue esse desenho e serve de referência. Os erros de negócio herdam de `AppError` (`src/shared/errors/app-error.ts`), e as classes específicas, como `InsufficientStockError` e `InvalidStatusTransitionError`, usam códigos como INSUFFICIENT_STOCK e INVALID_STATUS_TRANSITION ([09:28] Bruno).

O logger é Pino e já está no projeto inteiro (`src/shared/logger/index.ts`), e o middleware de erro centralizado já trata AppError, Zod e Prisma ([09:29] Bruno). A autorização por role existe em `requireRole`, no `src/middlewares/auth.middleware.ts`, e hoje é usada com ADMIN nas rotas de usuários.

A pergunta era se o módulo de webhooks seguiria esse padrão ou criaria peças próprias. O time é pequeno ([09:07] Diego) e a estimativa é de três sprints ([09:47] Larissa), o que pesa a favor de reaproveitar.

## Fatores de decisão

- O padrão de módulos já é claro e usado por todos os domínios ([09:27] Bruno).
- Os erros seguem uma convenção de classe e código que vale estender ([09:28] Bruno).
- Logger e middleware de erro já cobrem o que o módulo precisa, sem mudança ([09:29] Bruno).
- O replay da DLQ precisa de role ADMIN, e o controle de role já existe ([09:36] Larissa).
- O projeto usa UUID como id, e a outbox segue o mesmo padrão ([09:51] Larissa).

## Alternativas Consideradas

### 1. Infraestrutura própria para o módulo de webhooks

Criar logger, classes de erro ou tratamento de erro específicos para webhooks. Ninguém propôs; a opção aparece só na negativa de Bruno.

- **Prós:** liberdade para desenhar o módulo sem as limitações das classes existentes.
- **Contras:** duas formas de logar e de responder erro no mesmo projeto, e mais código para o time manter.
- **Por que foi descartada:** "Não vamos botar nada novo" ([09:29] Bruno), e Larissa fechou por reuso máximo ([09:30] Larissa). Pelo contexto, a frase de Bruno fala do logger; a reunião não disse se ela também proíbe dependências novas, como um cliente HTTP.

### 2. Lógica do worker fora do módulo

Colocar o processamento da outbox em uma pasta própria, separada de `src/modules/webhooks` (a criar). Diego perguntou onde o worker ficaria ([09:28] Diego).

- **Prós:** separa o código que roda no processo do worker do código que roda na API.
- **Contras:** espalha o domínio de webhooks por dois lugares.
- **Por que foi descartada:** Bruno pôs só a entry-point fora, em `src/worker.ts` (a criar), e a lógica dentro do módulo, em `webhook.worker.ts` ou `webhook.processor.ts` (a criar) ([09:28] Bruno). Diego concordou ([09:28] Diego). O nome do arquivo não foi escolhido.

## Decisão

O sistema de webhooks vira um módulo como os outros: `src/modules/webhooks` (a criar), com controller, service, repository, routes e schemas ([09:27] Bruno). Os erros usam `AppError` e classes específicas ([09:28] Bruno), e todos os códigos do módulo têm prefixo WEBHOOK_ ([09:29] Larissa). O logger é o Pino existente, e o middleware de erro centralizado trata os erros novos sem mudança ([09:29] Bruno). Larissa fechou o conjunto: AppError, Pino, error middleware, padrão de módulos, padrão de schemas Zod e padrão de códigos de erro ([09:30] Larissa).

Duas decisões com origem separada completam o reuso. O replay da DLQ reaproveita o `requireRole` existente para exigir ADMIN ([09:36] Larissa). O id da outbox é UUID, porque "tudo é uuid" no projeto ([09:51] Larissa); a pergunta de Diego foi entre id auto incremental e UUID ([09:51] Diego).

## Consequências

### Positivas

- Quem conhece um módulo do projeto entende o de webhooks sem curva nova ([09:27] Bruno).
- Respostas de erro com o mesmo formato da API inteira, via `src/middlewares/error.middleware.ts` ([09:29] Bruno).
- Logs do módulo saem no mesmo formato e com a mesma configuração do resto ([09:29] Bruno).
- Nenhuma dependência nova de infraestrutura de código ([09:29] Bruno).

### Negativas

- `NotFoundError` e `ValidationError` têm código fixo (NOT_FOUND e VALIDATION_ERROR) em `src/shared/errors/http-errors.ts`. Códigos como WEBHOOK_NOT_FOUND ([09:28] Bruno) pedem subclasse nova de `AppError`.
- A validação Zod do `src/middlewares/validate.middleware.ts` devolve VALIDATION_ERROR genérico, então erros de schema do módulo não saem com prefixo WEBHOOK_ sem ajuste.
- O `redact` do logger em `src/shared/logger/index.ts` cobre senha e tokens, mas não um campo `secret`. Sem ajuste, a secret do webhook ([ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md)) pode aparecer em log.
- O middleware de erro é do Express e não cobre o processo do worker ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)). O tratamento de erro do worker não foi discutido.
- "Tudo é uuid" ([09:51] Larissa) não é literal no código: `OrderNumberSequence` usa id inteiro em `prisma/schema.prisma`.

### Trade-off

Aceitamos as limitações das classes de erro, da validação e do logger atuais, que pedem pequenos ajustes, em troca de um módulo que segue o padrão do projeto e não introduz infraestrutura de código nova.

## Referências

- Transcrição: [09:07] Diego, [09:27] Bruno, [09:28] Diego, [09:28] Bruno, [09:29] Larissa, [09:29] Bruno, [09:30] Larissa, [09:36] Larissa, [09:47] Larissa, [09:48] Larissa, [09:51] Diego, [09:51] Larissa
- Código: `src/modules/orders/`, `src/shared/errors/http-errors.ts`, `src/middlewares/error.middleware.ts`, `src/middlewares/auth.middleware.ts`, `src/shared/logger/index.ts`
