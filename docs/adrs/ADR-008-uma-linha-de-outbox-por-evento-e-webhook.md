# ADR-008: Uma linha de outbox por evento e webhook

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md)

## Status

Proposto. A reunião não decidiu este ponto. A proposta foi feita na elaboração dos documentos de design e precisa ser validada com Larissa, Bruno e Diego, na sessão de revisão que Larissa marcou antes de começar a codar ([09:50] Larissa). O ponto está aberto no [RFC](../RFC.md).

## Contexto

A reunião fechou várias peças que, juntas, deixam uma pergunta sem resposta: quantas linhas a outbox recebe quando um evento interessa a mais de um webhook. Um customer pode ter vários endpoints cadastrados; Sofia pediu o header X-Webhook-Id justamente para o cliente com vários cadastros saber qual deles recebeu o envio ([09:44] Sofia). O filtro por status é aplicado na inserção da outbox: se nenhum webhook do customer quer aquele status, nada é inserido ([09:34] Bruno), e Diego concordou ([09:34] Diego).

O X-Event-Id é um UUID gerado quando o evento entra na outbox, único por evento ([09:25] Diego). O histórico de entregas é consultado por webhook, em GET /webhooks/:id/deliveries (a criar) ([09:34] Marcos). Retry e DLQ funcionam por evento na outbox ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)), mas cada endpoint tem disponibilidade própria.

Nenhuma fala da reunião diz se a outbox guarda uma linha por evento ou uma por par de evento e webhook, nem se o X-Event-Id se repete entre endpoints. Esta ADR registra a proposta adotada nos documentos para que o FDD possa ser escrito, sem apresentá-la como decisão da reunião.

## Fatores de decisão

- Um customer pode ter vários webhooks, cada um com seu próprio endpoint e sua própria secret ([09:21] Sofia).
- O filtro por status acontece uma vez, na inserção ([09:34] Bruno).
- O X-Event-Id identifica o evento, não a tentativa nem o destino ([09:25] Diego).
- O histórico de entregas é por webhook ([09:34] Marcos).
- A DLQ guarda o payload e o motivo da falha, o que só faz sentido se a falha for de um destino concreto ([09:18] Diego).

## Alternativas Consideradas

### A. Uma linha por evento e webhook (proposta)

Para cada webhook do customer que quer aquele status, uma linha na outbox. Todas compartilham o mesmo event_id. Descrita em detalhe na seção Decisão.

### B. Uma linha por evento, com fan-out na hora do envio

A outbox guarda um único registro por mudança de status, e o worker descobre os webhooks interessados e envia para cada um ao processar. Ninguém propôs na reunião; é a leitura literal de "uma linha por evento".

- **Prós:** menos linhas na outbox; o evento aparece uma única vez.
- **Contras:** um endpoint lento ou fora do ar segura os outros, ou exige guardar sub-estado por endpoint fora da linha. DLQ e replay ficam ambíguos quando só um dos endpoints falhou. O filtro por status precisaria rodar duas vezes: na inserção ([09:34] Bruno) e de novo no envio.
- **Por que foi descartada:** estado, tentativas e DLQ deixam de caber em uma linha só, que é como a reunião desenhou a outbox ([09:08] Diego).

### C. Uma linha por evento, mais uma tabela de entregas por evento e webhook

A outbox guarda o evento, e uma segunda tabela guarda o estado de cada par de evento e webhook. Ninguém propôs na reunião.

- **Prós:** separa o evento das entregas e funciona.
- **Contras:** cria duas estruturas de fila para o worker coordenar, e se afasta do desenho da reunião, que tem uma outbox com estados e uma DLQ ([09:18] Diego).
- **Por que foi descartada:** resolve o mesmo problema que A com mais peças para manter.

## Decisão

Proposta: na inserção, para cada webhook do customer que quer aquele status, o sistema grava uma linha na outbox. Todas as linhas daquela mudança de status compartilham o mesmo event_id, porque o evento é a mudança de status ([09:25] Diego). Cada linha tem estado, contador de tentativas e caminho para a DLQ próprios.

Com isso, o filtro por status roda uma vez, na inserção ([09:34] Bruno). Um endpoint fora do ar entra em backoff sozinho, sem atrasar os outros, e o replay de uma linha da DLQ atinge só o endpoint que falhou. O cliente que tem vários webhooks recebe o mesmo X-Event-Id em cada um e distingue o destino pelo X-Webhook-Id ([09:44] Sofia).

## Consequências

### Positivas

- Isolamento entre endpoints: retry, DLQ e replay valem por webhook ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)).
- O filtro na inserção ([09:34] Bruno) continua sendo o único ponto de filtragem.
- O cliente deduplica pelo X-Event-Id em todos os endpoints dele, com a mesma regra ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)).
- O worker processa linhas independentes, o que mantém o polling simples ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)).

### Negativas

- A outbox recebe mais linhas: uma por webhook interessado. O arquivamento das linhas entregues já estava fora do escopo ([09:08] Diego), e o volume cresce mais rápido.
- O event_id deixa de ser único na outbox; a unicidade passa a ser do par evento e webhook.
- O histórico de entregas, com sucesso ou falha, payload, response e tempo de resposta ([09:34] Marcos), ainda precisa de um registro por tentativa HTTP. A reunião não nomeou essa tabela. O FDD traz uma proposta e o [RFC](../RFC.md) mantém o ponto aberto.
- Ponto em aberto: toda a proposta depende de validação com Larissa, Bruno e Diego. Está no [RFC](../RFC.md).

### Trade-off

Aceitamos mais linhas na outbox e um event_id repetido entre webhooks em troca de isolar a entrega, o retry e a DLQ de cada endpoint.

## Referências

- Transcrição: [09:08] Diego, [09:18] Diego, [09:21] Sofia, [09:25] Diego, [09:34] Bruno, [09:34] Diego, [09:34] Marcos, [09:44] Sofia, [09:50] Larissa
- Código: `prisma/schema.prisma` (onde webhook_outbox será criada)
