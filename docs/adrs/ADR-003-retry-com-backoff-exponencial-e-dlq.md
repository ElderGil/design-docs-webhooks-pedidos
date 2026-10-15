# ADR-003: Retry com backoff exponencial e DLQ

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-002](ADR-002-worker-em-processo-separado-com-polling.md), [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-007](ADR-007-snapshot-do-payload-na-insercao.md), [ADR-008](ADR-008-uma-linha-de-outbox-por-evento-e-webhook.md)

## Status

Aceito. Larissa fechou com "Decidido: 5 tentativas, backoff 1m/5m/30m/2h/12h" ([09:17] Larissa). A DLQ em tabela separada foi proposta por Diego ([09:18] Diego), aceita por Bruno ([09:18] Bruno) e confirmada no resumo final ([09:48] Larissa).

## Contexto

O endpoint do cliente pode estar fora do ar ou lento quando o worker tenta entregar ([09:14] Larissa). Diego propôs backoff exponencial: tentar de novo depois de um tempo, aumentar o intervalo e, passado um teto de tentativas, tratar como falha permanente e mover para uma DLQ ([09:15] Diego). Uma chamada sem resposta em 10 segundos também conta como falha e vai para retry ([09:42] Diego).

As forças são duas e puxam para lados opostos. De um lado, o evento não pode ficar pendurado para sempre se o cliente sumir ([09:15] Diego). Do outro, a janela de retentativa precisa cobrir indisponibilidades reais: já houve cliente com duas horas de manutenção planejada ([09:16] Diego). Marcos aceitou como limite um cliente fora do ar por cerca de 15 horas: nesse caso, o problema é dele ([09:17] Marcos).

Também era preciso decidir onde ficam os eventos que esgotaram as tentativas: numa tabela separada ou marcados como "failed" na própria outbox ([09:17] Larissa). A outbox já tinha um estado "falhou" na descrição de Diego ([09:08] Diego).

## Fatores de decisão

- Evento não pode ficar pendurado indefinidamente quando o cliente desaparece ([09:15] Diego).
- A janela de retentativa precisa cobrir uma manutenção planejada de duas horas ([09:16] Diego).
- Cerca de 15 horas de janela é aceitável para o produto ([09:17] Marcos).
- A leitura da outbox principal deve ficar limpa, só com o que ainda está em trânsito ([09:18] Diego).
- As falhas permanentes precisam servir de evidência para debug e reprocessamento ([09:18] Diego).

## Alternativas Consideradas

### 1. Retry indefinido com backoff

Continuar tentando para sempre, com intervalos crescentes. Só foi mencionada, sem defensor na reunião: Diego disse que "algumas pessoas defendem" essa opção ([09:15] Diego).

- **Prós:** nenhum evento é abandonado enquanto o cliente puder voltar.
- **Contras:** o evento fica pendurado para sempre se o cliente sumiu ([09:15] Diego).
- **Por que foi descartada:** Diego a afastou pelo risco do evento eterno e propôs um teto ([09:15] Diego). Não houve debate.

### 2. Três tentativas

Bruno sugeriu 3 tentativas, "mais agressivo" ([09:16] Bruno).

- **Prós:** falha permanente detectada mais cedo e menos carga de retentativa.
- **Contras:** três tentativas cobririam só uns 30 minutos, e já houve cliente com duas horas de indisponibilidade em manutenção planejada ([09:16] Diego).
- **Por que foi descartada:** Diego considerou pouco ([09:16] Diego) e Larissa ficou com cinco ([09:16] Larissa).

### 3. DLQ como status "failed" na própria outbox

Marcar o evento como "failed" na outbox em vez de movê-lo. Larissa pôs as duas opções na mesa ([09:17] Larissa).

- **Prós:** uma tabela a menos e nenhuma movimentação de dados entre tabelas.
- **Contras:** a outbox principal mistura eventos em trânsito com falhas permanentes, e a leitura fica menos limpa ([09:18] Diego).
- **Por que foi descartada:** Diego preferiu tabela separada ([09:18] Diego), Bruno concordou ([09:18] Bruno) e o resumo final registrou "DLQ persistida em tabela separada" ([09:48] Larissa).

## Decisão

A entrega usa retry com backoff exponencial ([09:15] Diego), com 5 tentativas e progressão de 1 minuto, 5 minutos, 30 minutos, 2 horas e 12 horas ([09:17] Diego), fechado por Larissa ([09:17] Larissa). Timeout de 10 segundos na chamada HTTP conta como falha e entra no retry ([09:42] Diego), com aprovação de Sofia ([09:42] Sofia).

Esgotadas as tentativas, a falha é permanente e o evento vai para uma tabela separada, webhook_dead_letter (a criar), com o payload, o motivo da falha e o timestamp ([09:18] Diego). A tabela separada deixa a leitura da outbox mais limpa e guarda evidência para debug e reprocessamento ([09:18] Diego). Existe replay manual da DLQ por um endpoint administrativo que recoloca o evento na outbox como pendente ([09:18] Diego), restrito à role ADMIN ([09:36] Larissa). O detalhe desse endpoint está no FDD.

## Consequências

### Positivas

- A janela de quase 15 horas cobre indisponibilidades longas, inclusive uma manutenção de duas horas ([09:17] Diego).
- Nenhum evento fica pendurado para sempre: há um teto claro ([09:15] Diego).
- A outbox guarda só o que ainda está em trânsito; as falhas permanentes ficam em tabela própria, prontas para análise ([09:18] Diego).
- O replay manual permite recuperar eventos depois que o cliente corrige o problema ([09:18] Diego).

### Negativas

- Um evento pode chegar até cerca de 15 horas depois da mudança de status ([09:17] Diego), muito além da meta de 10 segundos do caminho normal ([09:02] Marcos).
- Cada retentativa é um reenvio, o que aumenta a chance de entrega duplicada; o cliente deduplica pelo X-Event-Id ([ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md)).
- O replay depende de ação humana de um ADMIN ([09:36] Larissa); não há reprocessamento automático.
- Ponto em aberto (contagem de tentativas): a reunião disse "5 tentativas" e listou 5 intervalos. A soma 1m + 5m + 30m + 2h + 12h dá 14h36min, o que bate com "quase 15 horas entre primeira falha e última tentativa" ([09:17] Diego). Essa conta leva a 1 envio inicial mais 5 retentativas, mas a reunião não disse isso explicitamente. Está no [RFC](../RFC.md) como questão em aberto.
- Ponto em aberto (estado "falhou" e DLQ): a outbox tem um estado "falhou" ([09:08] Diego) e a DLQ é uma tabela separada ([09:18] Diego). A reunião não definiu a transição entre os dois, nem o que a linha da outbox guarda depois que o evento vai para a DLQ. Está no [RFC](../RFC.md).
- Ponto em aberto: não se discutiu como o worker respeita o horário da próxima tentativa, já que ele busca "os eventos pendentes mais antigos" ([09:09] Diego), nem o que acontece com eventos presos em "processando" se o worker cair. Está no [RFC](../RFC.md).

### Trade-off

Aceitamos entregas que podem atrasar até cerca de 15 horas e uma tabela a mais para operar em troca de sobreviver a indisponibilidades longas do cliente sem deixar eventos pendurados para sempre.

## Referências

- Transcrição: [09:02] Marcos, [09:08] Diego, [09:09] Diego, [09:14] Larissa, [09:15] Diego, [09:16] Bruno, [09:16] Diego, [09:16] Larissa, [09:17] Diego, [09:17] Marcos, [09:17] Larissa, [09:18] Diego, [09:18] Bruno, [09:36] Larissa, [09:42] Diego, [09:42] Sofia, [09:48] Larissa
- Código: `prisma/schema.prisma` (onde webhook_dead_letter será criada)
