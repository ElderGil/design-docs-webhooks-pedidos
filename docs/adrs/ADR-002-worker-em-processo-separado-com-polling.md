# ADR-002: Worker em processo separado com polling

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-001](ADR-001-outbox-no-mysql.md), [ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

## Status

Aceito. Larissa registrou o polling de 2 segundos como decisão ([09:10] Larissa) e fechou a ordem por order_id como limitação conhecida ([09:13] Larissa). O resumo final repetiu "worker separado em polling de 2 segundos" ([09:48] Larissa).

## Contexto

Com a outbox decidida ([ADR-001](ADR-001-outbox-no-mysql.md)), faltava definir como o worker descobre os eventos novos e onde ele roda ([09:08] Larissa). A meta de produto é entregar o aviso em menos de 10 segundos ([09:02] Marcos). O banco é MySQL, e o projeto tem hoje uma única entry-point, `src/server.ts`, que sobe a API, usa o `prisma` exportado por `src/config/database.ts` e chama `prisma.$disconnect()` ao receber SIGINT ou SIGTERM.

Diego propôs polling em loop: a cada 2 segundos o worker busca os eventos pendentes mais antigos, processa e marca ([09:09] Diego). Bruno perguntou se dava para usar trigger do banco para reagir mais rápido ([09:09] Bruno). Sobre onde rodar, Diego disse que o worker não pode ficar na mesma instância da API: se a API reinicia, o worker cai junto ([09:11] Diego).

Houve uma correção sobre a conexão com o banco. Bruno disse primeiro que o worker usaria "o mesmo Prisma client" ([09:11] Bruno). Quando Diego perguntou se seria o mesmo PrismaClient ou um separado ([09:29] Diego), Bruno respondeu "Separado": PrismaClient é por processo, então o worker usa o mesmo banco e a mesma DATABASE_URL com uma instância própria ([09:30] Bruno). Vale a segunda fala.

## Fatores de decisão

- O aviso precisa chegar em menos de 10 segundos, e 2 segundos de polling cabem com folga ([09:09] Diego).
- MySQL não notifica processo externo, ao contrário do LISTEN/NOTIFY do Postgres ([09:09] Diego).
- O worker não pode cair junto quando a API reinicia ([09:11] Diego).
- Mesmo banco e mesma stack, sem infraestrutura nova ([09:11] Diego).
- Os clientes não pediram ordem global, só querem saber se cada pedido mudou ([09:14] Marcos).

## Alternativas Consideradas

### 1. Trigger do banco ou LISTEN/NOTIFY

Usar uma trigger do banco para o worker reagir assim que o evento entra na outbox. Bruno propôs ([09:09] Bruno).

- **Prós:** reação mais rápida que um intervalo fixo de polling.
- **Contras:** MySQL não tem listener nativo como o NOTIFY/LISTEN do Postgres; a trigger só executa SQL e não avisa processo externo. Para acordar o worker seria preciso improvisar, escrevendo em arquivo ou chamando um endpoint ([09:09] Diego).
- **Por que foi descartada:** Diego considerou a solução "esquisita" e lembrou que o polling de 2 segundos já atende o requisito de menos de 10 segundos ([09:09] Diego).

### 2. Worker dentro do processo da API

Rodar o loop de polling no mesmo processo Node que serve a API. Ninguém propôs essa opção; ela aparece só como o que Diego negou ([09:11] Diego).

- **Prós:** um processo a menos para subir e monitorar.
- **Contras:** se a API reinicia, o worker cai junto ([09:11] Diego), e a carga de envio passa a disputar recursos com as requisições.
- **Por que foi descartada:** Diego exigiu processo separado ([09:11] Diego), e o resumo final manteve "worker separado" ([09:48] Larissa).

## Decisão

O worker roda como processo separado da API ([09:11] Diego), com uma entry-point nova no molde de `src/server.ts`: `src/worker.ts` (a criar) e o script `npm run worker` (a criar) ([09:11] Larissa). Bruno confirmou `src/worker.ts` como entry separada ([09:28] Bruno) e Diego concordou ([09:28] Diego). O worker usa o mesmo banco e a mesma DATABASE_URL, com um PrismaClient próprio por ser outro processo ([09:30] Bruno). A lógica de processamento fica dentro do módulo de webhooks ([ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)).

O worker faz polling a cada 2 segundos, busca os eventos pendentes mais antigos, processa e marca ([09:09] Diego); Larissa registrou a decisão ([09:10] Larissa) e Marcos aceitou o intervalo ([09:10] Marcos). Roda um único worker, processando na ordem de created_at da outbox, o que dá ordem implícita por order_id ([09:12] Diego). Larissa registrou isso como limitação conhecida: não há ordem global, só por order_id e enquanto houver um único worker ([09:13] Larissa). Escalar para vários workers, particionando por order_id ou com lock pessimista, ficou para o futuro ([09:13] Diego).

## Consequências

### Positivas

- Reinício ou deploy da API não interrompe as entregas ([09:11] Diego).
- Nenhuma infraestrutura nova: mesmo banco, mesma stack ([09:11] Diego).
- O polling é simples de implementar e de testar, e cabe no limite de 10 segundos ([09:09] Diego).
- Com um único worker, cada cliente recebe os eventos de um pedido na ordem em que entraram na outbox ([09:12] Diego).

### Negativas

- O polling acrescenta até 2 segundos de espera antes de o worker ver o evento. Larissa registrou e aceitou esse custo ([09:10] Larissa); a frase original fala em "latência mínima" e "pior caso" ao mesmo tempo.
- O banco recebe uma consulta a cada 2 segundos mesmo sem eventos novos ([09:09] Diego).
- Há mais um processo para subir, monitorar e reiniciar em produção, consequência direta de separar o worker da API ([09:11] Diego). O custo operacional não foi discutido na reunião.
- Um único worker limita a vazão. Escalar quebra a garantia de ordem e exige particionar ou travar, o que ficou para o futuro ([09:13] Diego).
- O worker vai importar a configuração da aplicação. Hoje `src/config/env.ts` exige DATABASE_URL e JWT_SECRET para qualquer processo que a carregue.
- Ponto em aberto: a ordem por order_id pode se perder mesmo com um único worker. Se um evento de um pedido entra em backoff ([ADR-003](ADR-003-retry-com-backoff-exponencial-e-dlq.md)) e o evento seguinte do mesmo pedido já está pendente, o worker, que busca os pendentes mais antigos ([09:09] Diego), pode entregar o segundo antes do primeiro. Isto é análise, não fala da reunião, e está no [RFC](../RFC.md).

### Trade-off

Aceitamos até 2 segundos de espera pelo polling, um processo a mais em produção e ordem garantida só por pedido com um único worker em troca de não depender de infraestrutura nova nem de mecanismos de notificação que o MySQL não tem.

## Referências

- Transcrição: [09:02] Marcos, [09:08] Larissa, [09:09] Diego, [09:09] Bruno, [09:10] Marcos, [09:10] Larissa, [09:11] Diego, [09:11] Larissa, [09:11] Bruno, [09:12] Diego, [09:13] Diego, [09:13] Larissa, [09:14] Marcos, [09:28] Bruno, [09:28] Diego, [09:29] Diego, [09:30] Bruno, [09:48] Larissa
- Código: `src/server.ts`, `src/config/database.ts`, `src/config/env.ts`, `package.json`
