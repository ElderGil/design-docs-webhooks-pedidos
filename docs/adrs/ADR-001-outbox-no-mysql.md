# ADR-001: Outbox no MySQL

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md)

## Status

Aceito. Larissa fechou a decisão com "Tá decidido então: outbox em MySQL" ([09:08] Larissa) e a repetiu no resumo final ([09:48] Larissa), confirmado por Diego ([09:49] Diego) e Bruno ([09:49] Bruno).

## Contexto

Três clientes B2B pediram para ser avisados quando o status dos pedidos deles muda; hoje eles consultam GET /orders de tempos em tempos ([09:00] Marcos). A primeira pergunta técnica da reunião foi se o webhook sairia de forma síncrona no service de orders ou por algum tipo de fila ou outbox ([09:03] Larissa).

A mudança de status acontece em `OrderService.changeStatus`, no arquivo `src/modules/orders/order.service.ts`. O método roda dentro de uma transação do Prisma: atualiza o pedido, grava o histórico de status e, conforme a transição, debita ou repõe estoque. Bruno descreveu essa transação como "já pesada" ([09:04] Bruno).

A força principal é a consistência entre o status e o evento. Se a transação principal commitou, o evento tem que estar registrado; se deu rollback, o evento some junto ([09:06] Diego). Bruno reforçou que não pode existir caso de o status mudar e o evento não sair ([09:40] Bruno). A outra força é o tamanho do time: subir infraestrutura nova para isso foi chamado de overengineering ([09:07] Diego). O banco do projeto já é MySQL (`prisma/schema.prisma`).

## Fatores de decisão

- O evento precisa existir se, e somente se, a mudança de status commitar ([09:06] Diego).
- Uma chamada HTTP dentro da transação deixaria um cliente lento travar a mudança de status de outros pedidos ([09:04] Bruno).
- Cliente fora do ar não pode obrigar um rollback da mudança de status ([09:04] Bruno).
- O time é pequeno e não quer operar infraestrutura nova só para isso ([09:07] Diego).

## Alternativas Consideradas

### 1. Disparo síncrono no service de orders

Chamar o endpoint do cliente dentro de `changeStatus`, no momento da mudança. Larissa colocou essa opção na mesa ([09:03] Larissa).

- **Prós:** nenhuma tabela nova e nenhum processo novo (análise, não fala da reunião).
- **Contras:** a transação já é pesada e um cliente lento trava a mudança de status de outros pedidos ([09:04] Bruno). Com o cliente fora do ar, não dá para dar rollback na mudança de status ([09:04] Bruno).
- **Por que foi descartada:** Bruno argumentou contra ([09:04] Bruno), Larissa concordou ([09:04] Larissa) e Diego declarou síncrono "fora de questão" ([09:06] Diego).

### 2. Redis Streams

Usar Redis Streams, "ou alguma coisa parecida", como fila dos eventos ([09:07] Larissa).

- **Prós:** ninguém defendeu a opção na reunião.
- **Contras:** exigiria subir mais infraestrutura ([09:07] Larissa). Para um time pequeno, subir Redis Cluster para isso é overengineering ([09:07] Diego). Além disso, a gravação no Redis ficaria fora da transação do MySQL, o que reabre o problema de consistência (análise, não fala da reunião).
- **Por que foi descartada:** Diego afirmou que a outbox no MySQL existente resolve ([09:07] Diego).

## Decisão

Adotamos o padrão outbox no MySQL existente. Na mesma transação de `changeStatus`, o evento é gravado numa tabela de outbox, webhook_outbox (a criar) ([09:06] Diego). Se a gravação na outbox falhar, a transação inteira faz rollback e o status não muda ([09:40] Bruno); Diego classificou isso como essencial, porque fora da transação a garantia se perde ([09:41] Diego). Um worker separado lê a tabela e faz as chamadas HTTP ([09:06] Diego), conforme a [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md).

A gravação é feita por uma função do módulo de webhooks que recebe o client da transação atual, em vez de injetar o repository de webhooks no `OrderService` ([09:41] Bruno), solução com que Diego concordou ([09:41] Diego). Só entram na outbox os eventos que algum webhook do customer quer receber ([09:34] Bruno). A assinatura da função, a regra do filtro e a estrutura da tabela estão no FDD; o conteúdo gravado está na [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md).

## Consequências

### Positivas

- Não há inconsistência possível entre status e evento: o commit grava os dois e o rollback descarta os dois ([09:06] Diego).
- A mudança de status deixa de depender da latência e da disponibilidade dos clientes ([09:04] Bruno).
- Nenhuma infraestrutura nova: o mesmo MySQL do projeto ([09:07] Diego).
- A outbox vira a base do retry e da DLQ ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)) e da deduplicação por X-Event-Id ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)).

### Negativas

- A transação de `changeStatus` fica maior, porque passa a gravar o evento e aplicar o filtro dentro dela (análise, não fala da reunião).
- Uma falha ao gravar na outbox passa a impedir a mudança de status. É o comportamento desejado ([09:40] Bruno), mas soma um motivo a mais para a operação principal falhar.
- A tabela cresce com o tempo. O arquivamento das linhas entregues, "depois de 30 dias ou assim", ficou fora do escopo desta feature ([09:08] Diego).

### Trade-off

Aceitamos uma transação de mudança de status maior e uma tabela que cresce sem arquivamento nesta fase em troca de consistência garantida entre status e evento, sem infraestrutura nova.

## Referências

- Transcrição: [09:00] Marcos, [09:03] Larissa, [09:04] Bruno, [09:04] Larissa, [09:06] Diego, [09:07] Larissa, [09:07] Diego, [09:08] Diego, [09:08] Larissa, [09:34] Bruno, [09:40] Bruno, [09:41] Bruno, [09:41] Diego, [09:48] Larissa, [09:49] Diego, [09:49] Bruno
- Código: `src/modules/orders/order.service.ts`, `prisma/schema.prisma`
