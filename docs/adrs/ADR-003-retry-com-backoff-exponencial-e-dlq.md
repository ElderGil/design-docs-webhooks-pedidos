# ADR-003: Retry com backoff exponencial e DLQ

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)

## Status

Aceito. Larissa fechou com "Decidido: 5 tentativas, backoff 1m/5m/30m/2h/12h" ([09:17] Larissa). A DLQ em tabela separada foi proposta por Diego ([09:18] Diego), aceita por Bruno ([09:18] Bruno) e confirmada no resumo final ([09:48] Larissa).

## Contexto

Larissa abriu o tema com a pergunta: se o cliente está offline, o que a gente faz ([09:14] Larissa)? Diego propôs backoff exponencial: tentar de novo depois de algum tempo, aumentar o intervalo e, passado um teto de tentativas, considerar falha permanente e mover para uma DLQ ([09:15] Diego).

As forças puxam para lados opostos. De um lado, o evento não pode ficar pendurado para sempre se o cliente sumiu ([09:15] Diego). Do outro, a janela precisa cobrir indisponibilidades reais: já houve cliente com duas horas de manutenção planejada ([09:16] Diego). Marcos aceitou o limite de cerca de 15 horas: um cliente fora do ar por esse tempo já tem um problema sério dele ([09:17] Marcos).

Também era preciso decidir onde ficam os eventos que esgotaram as tentativas: numa tabela separada ou marcados como "failed" na própria outbox ([09:17] Larissa).

## Fatores de decisão

- O evento não pode ficar pendurado indefinidamente quando o cliente desaparece ([09:15] Diego).
- A janela de retentativa precisa cobrir uma manutenção planejada de duas horas ([09:16] Diego).
- Cerca de 15 horas de janela é aceitável para o produto ([09:17] Marcos).
- A leitura da outbox principal fica mais limpa com as falhas em outra tabela ([09:18] Diego).
- As falhas permanentes servem de evidência para debug e reprocessamento ([09:18] Diego).

## Alternativas Consideradas

### 1. Retry indefinido com backoff

Continuar tentando para sempre, com intervalos crescentes. A opção só foi mencionada, sem defensor na reunião: Diego disse que "algumas pessoas defendem" ([09:15] Diego).

- **Prós:** nenhum evento é abandonado enquanto o cliente puder voltar (análise, não fala da reunião).
- **Contras:** o evento fica pendurado para sempre se o cliente sumiu ([09:15] Diego).
- **Por que foi descartada:** Diego a afastou por esse risco e propôs um teto ([09:15] Diego). Não houve debate.

### 2. Três tentativas

Bruno sugeriu 3 tentativas, "mais agressivo" ([09:16] Bruno).

- **Prós:** Bruno não detalhou; a palavra usada foi "agressivo" ([09:16] Bruno).
- **Contras:** com indisponibilidade de manhã, seriam três tentativas em 30 minutos e o evento morreria, e já houve cliente com duas horas de manutenção planejada ([09:16] Diego).
- **Por que foi descartada:** Diego considerou pouco ([09:16] Diego) e Larissa ficou com cinco ([09:16] Larissa).

### 3. DLQ como status "failed" na própria outbox

Marcar o evento como "failed" na outbox em vez de movê-lo. Larissa colocou as duas opções na mesa ([09:17] Larissa).

- **Prós:** uma tabela a menos (análise, não fala da reunião).
- **Contras:** a leitura da outbox principal fica menos limpa ([09:18] Diego).
- **Por que foi descartada:** Diego preferiu tabela separada ([09:18] Diego), Bruno concordou ([09:18] Bruno) e o resumo final registrou "DLQ persistida em tabela separada" ([09:48] Larissa).

## Decisão

A entrega usa retry com backoff exponencial ([09:15] Diego): 5 tentativas, com intervalos de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas ([09:17] Diego), fechado por Larissa ([09:17] Larissa). Uma chamada sem resposta dentro do timeout conta como falha e entra no retry ([09:42] Diego); o valor do timeout e a contagem exata de envios estão no FDD. Os cinco intervalos somam 14h36, o que bate com "quase 15 horas entre primeira falha e última tentativa" ([09:17] Diego); por isso os documentos leem a decisão como 1 envio inicial mais 5 retentativas (interpretação, não fala da reunião).

Esgotadas as tentativas, a falha é permanente e o evento vai para uma tabela separada, webhook_dead_letter (a criar), com o payload, o motivo da falha e o timestamp ([09:18] Diego). O reprocessamento é manual, por um endpoint administrativo que recoloca o evento na outbox como pendente ([09:18] Diego); o contrato está no FDD.

## Consequências

### Positivas

- A janela de quase 15 horas cobre indisponibilidades longas, inclusive uma manutenção de duas horas ([09:16] Diego).
- Nenhum evento fica pendurado para sempre: há um teto claro ([09:15] Diego).
- As falhas permanentes ficam em tabela própria, com o motivo, prontas para debug e reprocessamento ([09:18] Diego).

### Negativas

- Um evento pode chegar até cerca de 15 horas depois da mudança de status ([09:17] Diego), muito além da meta de 10 segundos do caminho normal ([09:02] Marcos).
- Cada retentativa é um reenvio, o que aumenta a chance de entrega duplicada; o cliente deduplica pelo X-Event-Id ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)).
- O reprocessamento depende de ação humana ([09:18] Diego); não há reprocessamento automático.
- A outbox tem um estado "falhou" ([09:08] Diego) além da DLQ, e a reunião não definiu a transição entre os dois. O FDD propõe essa regra.

### Trade-off

Aceitamos entregas que podem atrasar até cerca de 15 horas e uma tabela a mais para operar em troca de sobreviver a indisponibilidades longas do cliente sem deixar eventos pendurados para sempre.

## Referências

- Transcrição: [09:02] Marcos, [09:08] Diego, [09:14] Larissa, [09:15] Diego, [09:16] Bruno, [09:16] Diego, [09:16] Larissa, [09:17] Diego, [09:17] Marcos, [09:17] Larissa, [09:18] Diego, [09:18] Bruno, [09:42] Diego, [09:48] Larissa
