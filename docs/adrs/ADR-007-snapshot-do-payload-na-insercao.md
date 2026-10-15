# ADR-007: Snapshot do payload na inserção

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-008](ADR-008-uma-linha-de-outbox-por-evento-e-webhook.md)

## Status

Aceito. Larissa propôs o snapshot ([09:52] Larissa), Diego concordou ([09:52] Diego) e Bruno fechou com "Beleza, snapshot. Decidido." ([09:52] Bruno). A decisão veio depois do resumo final, quando Marcos e Sofia já tinham saído da call às 09:50, segundo a transcrição.

## Contexto

O evento é gravado na outbox dentro da transação de `changeStatus` ([ADR-001](ADR-001-outbox-no-mysql.md)) e enviado depois por um worker ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)), com retry que pode se estender por quase 15 horas ([09:17] Diego). Entre a gravação e o envio, o pedido pode mudar de novo.

Bruno perguntou se a outbox guarda o payload já montado ou só o order_id, para montar o conteúdo na hora do envio ([09:51] Bruno). Larissa preferiu o payload pronto na inserção: se o pedido mudar depois, o evento ainda reflete o estado de quando o status mudou; do contrário, "tem caso esquisito" ([09:52] Larissa).

O conteúdo do payload, um JSON enxuto sem items, foi descrito por Diego ([09:43] Diego) e aprovado por Bruno ([09:44] Bruno). Esse formato é assunto do FDD; esta ADR trata só do momento em que ele é montado.

## Fatores de decisão

- O evento precisa refletir o estado do pedido no momento da mudança de status ([09:52] Larissa).
- Entre a gravação e o envio pode passar muito tempo por causa do retry ([09:17] Diego).
- Uma mudança posterior no pedido não pode alterar o que um evento antigo diz ([09:52] Larissa).
- O payload é enxuto, então guardá-lo pronto ocupa pouco espaço ([09:43] Diego).

## Alternativas Consideradas

### 1. Guardar só o order_id e montar o payload no envio

A outbox guarda a referência ao pedido, e o worker lê o pedido e monta o payload na hora de enviar. Bruno pôs a opção na mesa em forma de pergunta ([09:51] Bruno).

- **Prós:** linha menor na outbox; o cliente recebe os dados mais recentes do pedido.
- **Contras:** um evento "PAID para PROCESSING" poderia sair com dados de um pedido que já está em SHIPPED, o caso esquisito que Larissa citou ([09:52] Larissa). Um retry depois de horas mandaria um conteúdo diferente da primeira tentativa, com o mesmo X-Event-Id ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)).
- **Por que foi descartada:** Larissa preferiu o snapshot ([09:52] Larissa), e Diego e Bruno concordaram ([09:52] Diego; [09:52] Bruno).

Nenhuma outra alternativa foi discutida.

## Decisão

O payload é montado e gravado na outbox na hora da inserção ([09:52] Larissa), ou seja, dentro da transação de `changeStatus` ([ADR-001](ADR-001-outbox-no-mysql.md)). O worker envia o conteúdo gravado, sem consultar o pedido de novo. O motivo é que o evento reflete o estado de quando o status mudou, mesmo que o pedido mude depois ([09:52] Larissa).

Em `src/modules/orders/order.service.ts`, o pedido lido no início de `changeStatus` é anterior ao update de status. A função `publishWebhookEvent(tx, order, fromStatus, toStatus)` (a criar) recebe esse pedido ([09:41] Bruno), então o snapshot usa o status novo da transição, e não o campo de status do pedido lido.

## Consequências

### Positivas

- Cada evento conta o que aconteceu naquele momento, mesmo que o pedido mude depois ([09:52] Larissa).
- Todas as tentativas de um evento enviam o mesmo conteúdo, o que combina com a deduplicação por X-Event-Id ([09:25] Diego).
- O replay da DLQ ([09:18] Diego) reenvia exatamente o mesmo conteúdo da tentativa original.
- O worker não precisa ler a tabela de pedidos para enviar.

### Negativas

- Com retry de até cerca de 15 horas ([09:17] Diego), a diferença entre o snapshot e o estado atual do pedido pode ser grande. É exatamente isso que o snapshot preserva, mas o cliente precisa saber que o evento descreve o passado e, se quiser o estado atual, consultar o pedido ([09:43] Diego).
- A linha da outbox fica maior, porque guarda o payload inteiro em vez de só uma referência.
- A decisão foi tomada sem Marcos e Sofia, que já tinham saído da call às 09:50. Vale validar com eles, em especial com Sofia, porque a assinatura HMAC é calculada sobre esse corpo ([ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md)).
- Ponto em aberto: o limite de 64 KB de payload ([09:24] Larissa) pode ser checado na inserção, o que faria a mudança de status falhar, ou no envio. A reunião não definiu. Está no [RFC](../RFC.md).

### Trade-off

Aceitamos linhas maiores na outbox e eventos que podem estar desatualizados em relação ao pedido em troca de um registro fiel do momento da mudança, igual em todas as tentativas.

## Referências

- Transcrição: [09:17] Diego, [09:18] Diego, [09:24] Larissa, [09:25] Diego, [09:41] Bruno, [09:43] Diego, [09:44] Bruno, [09:51] Bruno, [09:52] Larissa, [09:52] Diego, [09:52] Bruno
- Código: `src/modules/orders/order.service.ts`
