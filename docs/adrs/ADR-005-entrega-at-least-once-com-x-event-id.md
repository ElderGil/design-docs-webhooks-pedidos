# ADR-005: Entrega at-least-once com X-Event-Id

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md), [ADR-008](ADR-008-uma-linha-de-outbox-por-evento-e-webhook.md)

## Status

Aceito. Larissa fechou com "At-least-once com X-Event-Id pra dedup do lado do cliente. Decisão." ([09:26] Larissa). O resumo final repetiu "Idempotência por X-Event-Id, garantia at-least-once" ([09:48] Larissa).

## Contexto

A entrega passa por uma outbox ([ADR-001](ADR-001-outbox-no-mysql.md)), um worker que faz a chamada HTTP ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)) e um retry com até cerca de 15 horas de janela ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)). Nesse caminho, o mesmo evento pode chegar mais de uma vez ao cliente. Diego deixou isso explícito: o cliente pode receber o mesmo evento duas vezes e tem que estar preparado ([09:24] Diego).

Bruno perguntou como o cliente diferencia uma repetição de um evento novo ([09:25] Bruno). A resposta de Diego foi um identificador por evento, enviado no header X-Event-Id, que o cliente usa para deduplicar ([09:25] Diego).

Sofia objetou que isso joga responsabilidade para o cliente ([09:25] Sofia). Diego respondeu que é o padrão de mercado, citando Stripe e GitHub, e que exactly-once exigiria coordenação dos dois lados ([09:25] Diego). Marcos se ofereceu para documentar o comportamento com destaque no portal de desenvolvedor ([09:26] Marcos).

## Fatores de decisão

- Com retry, uma entrega duplicada pode acontecer e precisa ser tratável ([09:24] Diego).
- O cliente precisa de um jeito simples de reconhecer a repetição ([09:25] Bruno).
- Exactly-once exige coordenação dos dois lados e é muito mais complexo ([09:25] Diego).
- At-least-once com id de evento é o padrão de mercado ([09:25] Diego).
- A responsabilidade extra do cliente precisa estar bem documentada ([09:26] Marcos).

## Alternativas Consideradas

### 1. Exactly-once

Garantir que cada evento chegue ao cliente uma única vez. Diego trouxe a opção para descartá-la ([09:25] Diego).

- **Prós:** o cliente não precisaria deduplicar, o que atende a objeção de Sofia ([09:25] Sofia).
- **Contras:** exige coordenação dos dois lados e fica muito mais complexo ([09:25] Diego). Sem um protocolo de confirmação com o cliente, a plataforma não tem como saber se uma chamada que estourou o timeout foi processada.
- **Por que foi descartada:** Diego argumentou pela complexidade e pelo padrão de mercado ([09:25] Diego), e Larissa fechou a decisão por at-least-once ([09:26] Larissa).

Nenhuma outra alternativa foi discutida.

## Decisão

A garantia de entrega é at-least-once ([09:24] Diego). Cada evento recebe um UUID gerado quando entra na outbox; esse UUID vai no header X-Event-Id, é único por evento, e o cliente deduplica por ele ([09:25] Diego). Larissa fechou a decisão ([09:26] Larissa).

A objeção de Sofia ([09:25] Sofia) foi aceita como trade-off: o cliente arca com a deduplicação, e Marcos documenta isso com destaque no portal de desenvolvedor ([09:26] Marcos). O evento é a mudança de status, então, quando vários webhooks do mesmo customer querem aquele status, todos recebem o mesmo X-Event-Id ([ADR-008](ADR-008-uma-linha-de-outbox-por-evento-e-webhook.md), proposta). O pacote `uuid` já está no `package.json` do projeto.

## Consequências

### Positivas

- O cliente sempre consegue identificar uma repetição pelo X-Event-Id ([09:25] Diego).
- A plataforma pode reenviar sem medo: retry ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)) e timeout de 10 segundos ([09:42] Diego) não exigem coordenação com o cliente.
- O modelo é o mesmo que Stripe e GitHub usam, segundo Diego ([09:25] Diego).
- Como o id nasce na inserção da outbox ([09:25] Diego), ele é o mesmo em todas as tentativas daquele evento.

### Negativas

- O cliente precisa guardar os event_ids já processados e checar cada chegada; quem não fizer isso processa duplicatas ([09:25] Sofia).
- Um timeout de 10 segundos ([09:42] Diego) pode acontecer depois de o cliente já ter processado a chamada, e o retry reenvia o mesmo evento. Isto é análise do fluxo, não fala da reunião.
- O sucesso da integração passa a depender da documentação no portal ([09:26] Marcos).
- Ponto em aberto: a reunião não disse se o replay da DLQ mantém o event_id original; Diego descreveu só que o evento volta à outbox como pendente ([09:18] Diego). Se o id mudar, a deduplicação do cliente deixa de funcionar para eventos reprocessados. Está no [RFC](../RFC.md).
- Ponto em aberto: o X-Event-Id compartilhado entre webhooks do mesmo customer depende da [ADR-008](ADR-008-uma-linha-de-outbox-por-evento-e-webhook.md), que ainda é proposta e está no [RFC](../RFC.md).

### Trade-off

Aceitamos que o cliente receba duplicatas e tenha que deduplicar pelo X-Event-Id em troca de um mecanismo de entrega simples, sem coordenação entre os dois lados.

## Referências

- Transcrição: [09:18] Diego, [09:24] Diego, [09:25] Bruno, [09:25] Diego, [09:25] Sofia, [09:26] Marcos, [09:26] Larissa, [09:42] Diego, [09:48] Larissa
- Código: `package.json` (dependência `uuid`)
