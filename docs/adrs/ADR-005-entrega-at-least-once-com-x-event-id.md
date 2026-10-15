# ADR-005: Entrega at-least-once com X-Event-Id

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md)

## Status

Aceito. Larissa fechou com "At-least-once com X-Event-Id pra dedup do lado do cliente. Decisão." ([09:26] Larissa). O resumo final repetiu "Idempotência por X-Event-Id, garantia at-least-once" ([09:48] Larissa).

## Contexto

A entrega passa por uma outbox ([ADR-001](ADR-001-outbox-no-mysql.md)), um worker que faz a chamada HTTP ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)) e um retry com janela de quase 15 horas ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)). Nesse caminho, o mesmo evento pode chegar mais de uma vez. Diego deixou isso explícito: pode acontecer de o cliente receber o mesmo evento duas vezes, e ele tem que estar preparado ([09:24] Diego).

Bruno perguntou como o cliente diferencia uma repetição de um evento novo ([09:25] Bruno). Diego respondeu com um UUID gerado quando o evento entra na outbox, enviado no header X-Event-Id, único por evento; o cliente deduplica por ele ([09:25] Diego).

Sofia comentou que isso joga responsabilidade para o cliente ([09:25] Sofia). Diego respondeu que é o padrão de mercado, citando Stripe e GitHub, e que exactly-once exigiria coordenação dos dois lados ([09:25] Diego). Marcos se ofereceu para documentar isso com destaque no portal de desenvolvedor ([09:26] Marcos).

## Fatores de decisão

- Com retry, uma entrega duplicada pode acontecer e o cliente precisa estar preparado ([09:24] Diego).
- O cliente precisa de um jeito de reconhecer a repetição ([09:25] Bruno).
- Exactly-once exige coordenação dos dois lados e é muito mais complexo ([09:25] Diego).
- At-least-once com id de evento é o padrão de mercado ([09:25] Diego).
- A responsabilidade do cliente precisa estar bem documentada ([09:26] Marcos).

## Alternativas Consideradas

### 1. Exactly-once

Garantir que cada evento chegue ao cliente uma única vez. Diego trouxe a opção para descartá-la ([09:25] Diego).

- **Prós:** o cliente não precisaria deduplicar, o que tiraria dele a responsabilidade que Sofia apontou ([09:25] Sofia).
- **Contras:** exige coordenação dos dois lados e fica muito mais complexo ([09:25] Diego).
- **Por que foi descartada:** Diego argumentou pela complexidade e pelo padrão de mercado ([09:25] Diego), e Larissa fechou por at-least-once ([09:26] Larissa).

Nenhuma outra alternativa foi discutida.

## Decisão

A garantia de entrega é at-least-once ([09:24] Diego). Cada evento recebe um UUID quando entra na outbox; esse UUID vai no header X-Event-Id, é único por evento, e o cliente deduplica por ele ([09:25] Diego). Larissa fechou a decisão ([09:26] Larissa).

A responsabilidade do cliente foi aceita como trade-off, e Marcos documenta o comportamento com destaque no portal de desenvolvedor ([09:26] Marcos). Como o id é gerado na inserção, ele é o mesmo em todas as tentativas daquele evento. O comportamento do id no reprocessamento da DLQ e com vários webhooks do mesmo customer está no FDD.

## Consequências

### Positivas

- O cliente consegue identificar uma repetição pelo X-Event-Id ([09:25] Diego).
- A plataforma pode reenviar sem coordenar com o cliente, o que torna o retry da [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md) seguro do ponto de vista do contrato (análise, não fala da reunião).
- O modelo é o mesmo de Stripe e GitHub, segundo Diego ([09:25] Diego).

### Negativas

- A deduplicação passa a ser responsabilidade do cliente ([09:25] Sofia); quem não deduplicar processa repetições.
- Uma chamada que estoura o timeout pode já ter sido processada pelo cliente, e o retry a reenvia (análise, não fala da reunião).
- O sucesso da integração depende da documentação no portal ([09:26] Marcos).

### Trade-off

Aceitamos que o cliente receba repetições e precise deduplicar em troca de uma entrega simples, sem coordenação entre os dois lados.

## Referências

- Transcrição: [09:24] Diego, [09:25] Bruno, [09:25] Diego, [09:25] Sofia, [09:26] Marcos, [09:26] Larissa, [09:48] Larissa
- Código: `package.json` (a biblioteca `uuid` já é dependência do projeto)
