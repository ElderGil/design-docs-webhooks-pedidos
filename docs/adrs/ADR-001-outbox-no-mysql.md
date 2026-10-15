# ADR-001: Outbox no MySQL

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md), [ADR-008](ADR-008-uma-linha-de-outbox-por-evento-e-webhook.md)

## Status

Aceito. Larissa fechou a decisão com "Tá decidido então: outbox em MySQL" ([09:08] Larissa) e a repetiu no resumo final ([09:48] Larissa), confirmado por Diego ([09:49] Diego) e Bruno ([09:49] Bruno).

## Contexto

Três clientes B2B pediram para ser avisados quando o status dos pedidos deles muda. Hoje eles consultam GET /orders de tempos em tempos, e a integração fica lenta e cara ([09:00] Marcos). Para esses clientes, qualquer coisa abaixo de 10 segundos já é tempo real ([09:02] Marcos).

A mudança de status acontece em `OrderService.changeStatus`, no arquivo `src/modules/orders/order.service.ts`. O método roda dentro de `prisma.$transaction`: atualiza a order, grava em order_status_history e, conforme a transição, debita ou repõe estoque. Bruno descreveu essa transação como "já pesada" ([09:04] Bruno). A primeira pergunta da reunião foi se o webhook sairia de forma síncrona nesse service ou por alguma fila ou outbox ([09:03] Larissa).

A força principal é a consistência entre o status e o evento. Se a transação principal commitou, o evento precisa estar registrado; se ela deu rollback, o evento some junto ([09:06] Diego). Bruno reforçou que não pode existir caso de o status mudar e o evento não sair ([09:40] Bruno). A outra força é o tamanho do time: subir infraestrutura nova para isso foi chamado de overengineering ([09:07] Diego). O banco do projeto já é MySQL (`prisma/schema.prisma`, datasource `mysql`).

## Fatores de decisão

- O evento precisa ser registrado se, e somente se, a mudança de status commitar ([09:06] Diego).
- Uma chamada HTTP dentro da transação deixaria um cliente lento travar a mudança de status de outros pedidos ([09:04] Bruno).
- Cliente fora do ar não pode levar a um rollback da mudança de status ([09:04] Bruno).
- O time é pequeno e não quer operar infraestrutura nova só para isso ([09:07] Diego).
- A meta de latência é abaixo de 10 segundos, não instantânea ([09:02] Marcos).

## Alternativas Consideradas

### 1. Disparo síncrono no service de orders

Chamar o endpoint do cliente dentro de `changeStatus`, no momento da mudança. Larissa pôs essa opção na mesa ao lado da fila ou outbox ([09:03] Larissa).

- **Prós:** nenhuma tabela nova; o aviso sai no mesmo instante da mudança.
- **Contras:** a transação já é pesada, e um cliente lento trava a mudança de status de outros pedidos ([09:04] Bruno). Com o cliente fora do ar, a única saída seria desfazer a mudança de status, o que não faz sentido ([09:04] Bruno).
- **Por que foi descartada:** Bruno argumentou contra ([09:04] Bruno), Larissa concordou ([09:04] Larissa) e Diego declarou "fora de questão" ([09:06] Diego).

### 2. Redis Streams

Usar Redis Streams, ou algo parecido, como fila dos eventos. Larissa propôs e ela mesma apontou o custo: seria preciso subir mais infraestrutura ([09:07] Larissa).

- **Prós:** estrutura feita para fila, com leitura mais reativa que um polling.
- **Contras:** mais infraestrutura para um time pequeno ([09:07] Diego). A gravação no Redis também ficaria fora da transação do MySQL, o que reabre o problema de consistência que a outbox resolve (análise, não fala da reunião).
- **Por que foi descartada:** Diego chamou de overengineering subir Redis Cluster para isso e afirmou que a outbox no MySQL existente resolve ([09:07] Diego).

### 3. Injetar o repository de webhooks no OrderService

Passar ao `OrderService` um repository de webhook para gravar o evento. Bruno citou essa opção ao lado de uma função de "enqueue event" ([09:41] Bruno).

- **Prós:** segue a injeção que o `OrderService` já usa, cujo construtor recebe `OrderRepository` e `PrismaClient` (`src/modules/orders/order.service.ts`).
- **Contras:** o `OrderService` passa a depender do módulo de webhooks inteiro, e o repository ainda teria que receber o client da transação para gravar dentro dela.
- **Por que foi descartada:** Bruno preferiu uma função que recebe o client da transação ([09:41] Bruno), e Diego concordou que não precisa injetar o repository inteiro ([09:41] Diego).

## Decisão

Adotamos o padrão outbox no MySQL existente. Na mesma transação de `changeStatus`, o evento é gravado na tabela webhook_outbox (a criar) ([09:06] Diego). Se a inserção na outbox falhar, a transação inteira faz rollback e o status não muda ([09:40] Bruno). Diego classificou isso como essencial: fora da transação, a garantia se perde ([09:41] Diego). Um worker separado lê a tabela e faz as chamadas HTTP ([09:06] Diego), conforme a [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md).

O `OrderService` grava o evento chamando `publishWebhookEvent(tx, order, fromStatus, toStatus)` (a criar), uma função que recebe o client da transação atual ([09:41] Bruno). Como ela escreve no banco por esse client, participa do mesmo commit e do mesmo rollback, e o `OrderService` não precisa conhecer o repository de webhooks ([09:41] Diego). O filtro por status é aplicado nessa gravação: se nenhum webhook do customer quer aquele status, nada é inserido ([09:34] Bruno). Quantas linhas um evento gera está na [ADR-008](ADR-008-uma-linha-de-outbox-por-evento-e-webhook.md); o conteúdo gravado, na [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md).

## Consequências

### Positivas

- Não há inconsistência possível entre status e evento: o commit grava os dois e o rollback descarta os dois ([09:06] Diego).
- A mudança de status deixa de depender da latência e da disponibilidade dos clientes ([09:04] Bruno).
- Nenhuma infraestrutura nova: o mesmo MySQL e o mesmo Prisma do projeto ([09:07] Diego).
- A outbox vira a base do retry e da DLQ ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)) e da deduplicação por X-Event-Id ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)).

### Negativas

- A transação de `changeStatus` fica maior: além do insert, ela consulta os webhooks do customer para aplicar o filtro na inserção ([09:34] Bruno).
- Uma falha ao gravar na outbox passa a impedir a mudança de status. É o comportamento desejado ([09:40] Bruno), mas aumenta o número de motivos para a operação principal falhar.
- A tabela cresce com o tempo. O arquivamento das linhas entregues, "depois de 30 dias ou assim", ficou fora do escopo desta feature ([09:08] Diego).
- O desempenho do worker depende de índices e de leitura em lote pequeno, que Diego descreveu ([09:08] Diego) e o FDD detalha.
- Ponto em aberto: a criação do pedido acontece em `OrderService.create()`, fora de `changeStatus`, e a reunião não disse se ela gera evento. Está no [RFC](../RFC.md).
- Ponto em aberto: a outbox tem um estado "falhou" ([09:08] Diego) e a DLQ é uma tabela separada ([09:18] Diego). A transição entre os dois não foi definida. Ver [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md) e o [RFC](../RFC.md).

### Trade-off

Aceitamos uma transação de mudança de status maior e uma tabela que cresce sem arquivamento nesta fase em troca de consistência garantida entre status e evento, sem infraestrutura nova.

## Referências

- Transcrição: [09:00] Marcos, [09:02] Marcos, [09:03] Larissa, [09:04] Bruno, [09:04] Larissa, [09:06] Diego, [09:07] Larissa, [09:07] Diego, [09:08] Diego, [09:08] Larissa, [09:18] Diego, [09:34] Bruno, [09:40] Bruno, [09:41] Bruno, [09:41] Diego, [09:48] Larissa, [09:49] Diego, [09:49] Bruno
- Código: `src/modules/orders/order.service.ts`, `prisma/schema.prisma`
