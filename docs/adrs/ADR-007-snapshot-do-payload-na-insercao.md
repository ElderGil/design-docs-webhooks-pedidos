# ADR-007: Snapshot do payload na inserção

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)

## Status

Aceito. Larissa propôs o snapshot ([09:52] Larissa), Diego concordou ([09:52] Diego) e Bruno fechou com "Beleza, snapshot. Decidido." ([09:52] Bruno). A decisão veio depois do resumo final, quando Marcos e Sofia já tinham saído da call (às 09:50, segundo a transcrição).

## Contexto

O evento é gravado na outbox dentro da transação de `changeStatus` ([ADR-001](ADR-001-outbox-no-mysql.md)) e enviado depois por um worker ([ADR-002](ADR-002-worker-em-processo-separado-com-polling.md)), com retry que pode se estender por quase 15 horas ([09:17] Diego). Entre a gravação e o envio, o pedido pode mudar de novo.

Bruno perguntou se a outbox guarda o payload já renderizado ou só o order_id, para renderizar na hora do envio ([09:51] Bruno). Larissa preferiu o payload pronto na inserção: se o pedido mudar depois, o evento ainda reflete o estado de quando o status mudou; do contrário, "tem caso esquisito" ([09:52] Larissa).

O conteúdo do payload foi descrito por Diego ([09:43] Diego) e está no FDD. Esta ADR trata só do momento em que ele é montado.

## Fatores de decisão

- O evento precisa refletir o estado do pedido no momento da mudança de status ([09:52] Larissa).
- Entre a gravação e o envio pode passar muito tempo por causa do retry ([09:17] Diego).
- Uma mudança posterior no pedido não pode alterar o que um evento antigo diz ([09:52] Larissa).

## Alternativas Consideradas

### 1. Guardar só o order_id e montar o payload no envio

A outbox guarda a referência ao pedido, e o worker lê o pedido e monta o payload na hora de enviar. Bruno trouxe a opção em forma de pergunta ([09:51] Bruno).

- **Prós:** linha menor na outbox (análise, não fala da reunião).
- **Contras:** o evento pode sair com dados posteriores à mudança que ele descreve, o "caso esquisito" de Larissa ([09:52] Larissa). Um exemplo: um evento de PAID para PROCESSING enviado depois de o pedido já estar em SHIPPED (exemplo nosso, não da reunião).
- **Por que foi descartada:** Larissa preferiu o snapshot ([09:52] Larissa), e Diego e Bruno concordaram ([09:52] Diego; [09:52] Bruno).

Nenhuma outra alternativa foi discutida.

## Decisão

O payload é montado e gravado na outbox no momento da inserção ([09:52] Larissa), ou seja, dentro da transação de `changeStatus` ([ADR-001](ADR-001-outbox-no-mysql.md)). O worker envia o conteúdo gravado, sem montar de novo. O motivo é que o evento reflete o estado de quando o status mudou, mesmo que o pedido mude depois ([09:52] Larissa).

## Consequências

### Positivas

- Cada evento conta o que aconteceu naquele momento, mesmo que o pedido mude depois ([09:52] Larissa).
- Todas as tentativas e o reprocessamento da DLQ enviam o mesmo conteúdo, o que combina com a deduplicação por X-Event-Id ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)) (análise, não fala da reunião).
- O worker não precisa ler a tabela de pedidos para enviar (análise, não fala da reunião).

### Negativas

- A linha da outbox fica maior, porque guarda o payload inteiro em vez de só uma referência (análise, não fala da reunião).
- Com retry de até cerca de 15 horas ([09:17] Diego), o cliente pode receber um evento que descreve um estado já superado. Para o estado atual, ele consulta o pedido, como Diego previu para quem quiser detalhes ([09:43] Diego).
- A decisão foi tomada sem Marcos e Sofia. Vale validar com eles, em especial com Sofia, porque a assinatura HMAC é calculada sobre esse corpo ([ADR-004](ADR-004-hmac-sha256-com-secret-por-endpoint.md)).

### Trade-off

Aceitamos linhas maiores na outbox e eventos que podem chegar descrevendo um estado já superado em troca de eventos fiéis ao momento da mudança e iguais em todas as tentativas.

## Referências

- Transcrição: [09:17] Diego, [09:43] Diego, [09:51] Bruno, [09:52] Larissa, [09:52] Diego, [09:52] Bruno
- Código: `src/modules/orders/order.service.ts`
