# ADR-002: Worker em processo separado com polling

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

## Status

Aceito. Larissa registrou o polling de 2 segundos como decisão ([09:10] Larissa) e fechou a ordem por order_id como limitação conhecida ([09:13] Larissa). O resumo final repetiu "worker separado em polling de 2 segundos" ([09:48] Larissa).

## Contexto

Com a outbox decidida ([ADR-001](ADR-001-outbox-no-mysql.md)), Larissa abriu o próximo ponto: como o worker lê a tabela ([09:08] Larissa). A meta de produto é o aviso chegar em menos de 10 segundos ([09:02] Marcos). O banco é MySQL, e o projeto tem hoje uma única entry-point, `src/server.ts`, que sobe a API usando o client Prisma exportado por `src/config/database.ts`.

Diego propôs polling em loop: a cada 2 segundos, o worker busca os eventos pendentes mais antigos, processa e marca ([09:09] Diego). Bruno perguntou se dava para usar trigger do banco para ser mais reativo ([09:09] Bruno). Sobre onde rodar, Diego disse que o worker não pode ficar na mesma instância da API, porque se a API reinicia, perde o worker ([09:11] Diego).

## Fatores de decisão

- O aviso precisa chegar em menos de 10 segundos, e o polling de 2 segundos atende esse limite ([09:09] Diego).
- Trigger no MySQL só executa SQL e não notifica processo externo ([09:09] Diego).
- O worker não pode cair junto quando a API reinicia ([09:11] Diego).
- Mesmo banco e mesma stack, só não o mesmo processo ([09:11] Diego).
- Os clientes não pediram ordem global, só querem saber se cada pedido mudou ([09:14] Marcos).

## Alternativas Consideradas

### 1. Trigger do banco

Usar uma trigger do MySQL para o worker reagir assim que o evento entra na outbox. Bruno propôs, para ser "mais reativo" ([09:09] Bruno).

- **Prós:** reação sem esperar o próximo ciclo de polling, que era o objetivo da pergunta ([09:09] Bruno).
- **Contras:** o MySQL não tem listener nativo, como o NOTIFY/LISTEN do Postgres. A trigger só executa SQL e não avisa processo externo, então seria preciso improvisar, escrevendo em arquivo ou chamando um endpoint ([09:09] Diego).
- **Por que foi descartada:** Diego achou a solução "esquisita" e lembrou que o polling de 2 segundos já atende o limite de 10 segundos ([09:09] Diego).

### 2. Worker dentro do processo da API

Rodar o loop de polling no mesmo processo Node da API. Ninguém defendeu essa opção; ela aparece como o que Diego vetou ([09:11] Diego).

- **Prós:** um processo a menos para subir (análise, não fala da reunião).
- **Contras:** se a API reinicia, o worker cai junto ([09:11] Diego).
- **Por que foi descartada:** Diego exigiu processo separado ([09:11] Diego), e o resumo final manteve "worker separado" ([09:48] Larissa).

## Decisão

O worker roda como processo separado da API ([09:11] Diego), com uma entry-point própria no molde de `src/server.ts`: `src/worker.ts` (a criar), iniciado por `npm run worker` (a criar) ([09:11] Larissa). Bruno confirmou a entry separada ([09:28] Bruno) e Diego concordou ([09:28] Diego). O worker usa o mesmo banco que a API, mas abre a própria conexão por ser outro processo ([09:30] Bruno); o detalhe está no FDD.

O worker faz polling a cada 2 segundos e processa os eventos pendentes mais antigos ([09:09] Diego). Larissa registrou a decisão ([09:10] Larissa) e Marcos aceitou o intervalo ([09:10] Marcos). Roda um único worker, na ordem de criação dos eventos, o que dá ordem implícita por pedido ([09:12] Diego). Larissa registrou isso como limitação conhecida: não há ordem global, só por order_id e enquanto houver um único worker ([09:13] Larissa). Escalar para vários workers, particionando por order_id ou com lock pessimista, ficou para o futuro ([09:13] Diego).

## Consequências

### Positivas

- Reinício ou deploy da API não interrompe as entregas ([09:11] Diego).
- Nenhuma infraestrutura nova: mesmo banco, mesma stack ([09:11] Diego).
- O intervalo de 2 segundos cabe no limite de 10 segundos ([09:09] Diego).
- Com um único worker, cada cliente recebe os eventos de um pedido na ordem em que entraram ([09:12] Diego).

### Negativas

- O polling acrescenta até 2 segundos antes de o worker ver o evento, custo que Larissa registrou e aceitou ([09:10] Larissa).
- O worker consulta o banco a cada ciclo, mesmo sem eventos novos (análise, não fala da reunião).
- Há mais um processo para subir e acompanhar em produção, consequência de separar o worker da API ([09:11] Diego). O custo operacional não foi discutido.
- Um único worker limita a vazão. Escalar quebra a garantia de ordem e exige particionar ou travar, o que ficou para o futuro ([09:13] Diego).
- Com retry, a ordem por pedido pode se perder mesmo com um único worker, se um evento entrar em backoff e o seguinte do mesmo pedido estiver pendente (análise, não fala da reunião). O tratamento está no FDD.

### Trade-off

Aceitamos até 2 segundos de espera pelo polling, um processo a mais em produção e ordem garantida só por pedido, com um único worker, em troca de não depender de infraestrutura nova nem de um mecanismo de notificação que o MySQL não tem.

## Referências

- Transcrição: [09:02] Marcos, [09:08] Larissa, [09:09] Diego, [09:09] Bruno, [09:10] Marcos, [09:10] Larissa, [09:11] Diego, [09:11] Larissa, [09:12] Diego, [09:13] Diego, [09:13] Larissa, [09:14] Marcos, [09:28] Bruno, [09:28] Diego, [09:30] Bruno, [09:48] Larissa
- Código: `src/server.ts`, `src/config/database.ts`
