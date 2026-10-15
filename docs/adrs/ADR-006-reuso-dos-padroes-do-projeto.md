# ADR-006: Reuso dos padrões do projeto

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md)

## Status

Aceito. Larissa fechou com "Decisão: reuso máximo do que já existe" ([09:30] Larissa) e repetiu os itens no resumo final ([09:48] Larissa).

## Contexto

A codebase tem um padrão claro: cada domínio é um módulo em `src/modules` com controller, service, repository, routes e schemas ([09:27] Bruno). O módulo `src/modules/orders/` segue esse desenho e serve de referência. Os erros de negócio herdam de `AppError` (`src/shared/errors/app-error.ts`), e as classes específicas em `src/shared/errors/http-errors.ts`, como `InsufficientStockError` e `InvalidStatusTransitionError`, usam códigos como INSUFFICIENT_STOCK e INVALID_STATUS_TRANSITION ([09:28] Bruno).

O logger é Pino e já está no projeto inteiro (`src/shared/logger/index.ts`). O middleware de erro centralizado (`src/middlewares/error.middleware.ts`) já trata AppError, Zod e Prisma ([09:29] Bruno). A autorização por role existe como `requireRole`, em `src/middlewares/auth.middleware.ts`.

## Fatores de decisão

- O padrão de módulos já é claro e usado por todos os domínios ([09:27] Bruno).
- Os erros seguem uma convenção de classe e código que vale estender ([09:28] Bruno).
- O logger e o middleware de erro já existem e pegam os erros novos sem mudança ([09:29] Bruno).
- O endpoint administrativo da DLQ precisa de role ADMIN, e o controle de role já existe ([09:36] Larissa).

## Alternativas Consideradas

### 1. Infraestrutura própria para o módulo de webhooks

Criar logger, classes de erro ou tratamento de erro específicos para webhooks. Ninguém propôs; é uma alternativa plausível que aparece só na negativa de Bruno: "Não vamos botar nada novo" ([09:29] Bruno).

- **Prós:** liberdade para desenhar o módulo sem as limitações das classes existentes (análise, não fala da reunião).
- **Contras:** duas formas de logar e de responder erro no mesmo projeto (análise, não fala da reunião).
- **Por que foi descartada:** Bruno descartou peça nova no logger ([09:29] Bruno) e Larissa fechou por reuso máximo ([09:30] Larissa).

### 2. Usar os códigos de erro genéricos existentes

Responder os erros do módulo com os códigos que já existem, como NOT_FOUND e VALIDATION_ERROR, sem prefixo próprio. Ninguém propôs; é uma alternativa plausível ao que Bruno sugeriu ([09:28] Bruno).

- **Prós:** nenhuma classe de erro nova (análise, não fala da reunião).
- **Contras:** o cliente não distingue um erro do módulo de webhooks de um erro de outro domínio (análise, não fala da reunião).
- **Por que foi descartada:** Bruno propôs seguir o padrão de códigos específicos, com nomes como WEBHOOK_NOT_FOUND ([09:28] Bruno), e Larissa fixou o prefixo WEBHOOK_ para tudo do módulo ([09:29] Larissa).

## Decisão

O sistema de webhooks vira um módulo como os outros, `src/modules/webhooks` (a criar), com controller, service, repository, routes e schemas ([09:27] Bruno). A lógica do worker também fica dentro do módulo, e só a entry-point fica fora ([09:28] Bruno). Os erros usam `AppError` e classes específicas ([09:28] Bruno), com o prefixo WEBHOOK_ em todos os códigos do módulo ([09:29] Larissa). O logger é o Pino existente, e o middleware de erro centralizado trata os erros novos ([09:29] Bruno).

Larissa fechou o conjunto: AppError, Pino, error middleware, padrão de módulos, padrão de schemas Zod e padrão de códigos de erro ([09:30] Larissa). O endpoint administrativo da DLQ reaproveita o `requireRole` existente para exigir ADMIN ([09:36] Larissa).

## Consequências

### Positivas

- Quem conhece um módulo do projeto entende o de webhooks sem curva nova ([09:27] Bruno).
- As respostas de erro saem no mesmo formato da API inteira, pelo middleware existente ([09:29] Bruno).
- Os logs do módulo saem com o mesmo formato e a mesma configuração do resto ([09:29] Bruno).

### Negativas

- As peças existentes não cobrem tudo sem ajuste. Algumas classes de erro têm código fixo, a validação Zod devolve um código genérico e o redact do logger não cobre a secret do webhook. Os ajustes estão na seção de integração do FDD.
- O middleware de erro é do Express e não cobre o processo do worker ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)); o tratamento de erro no worker está no FDD.

### Trade-off

Aceitamos as limitações das peças existentes, com ajustes pontuais, em troca de um módulo que se comporta, loga e responde erro como o resto do projeto.

## Referências

- Transcrição: [09:27] Bruno, [09:28] Bruno, [09:29] Bruno, [09:29] Larissa, [09:30] Larissa, [09:36] Larissa, [09:48] Larissa
- Código: `src/modules/orders/`, `src/shared/errors/app-error.ts`, `src/shared/errors/http-errors.ts`, `src/middlewares/error.middleware.ts`, `src/middlewares/auth.middleware.ts`
