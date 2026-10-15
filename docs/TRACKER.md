# Tracker de rastreabilidade

Este tracker liga cada item do [PRD](PRD.md), do [RFC](RFC.md), do [FDD](FDD.md) e dos [ADRs](adrs/README.md) à origem dele. A origem é uma de duas: a fala da reunião técnica ([`TRANSCRICAO.md`](../TRANSCRICAO.md)) ou um arquivo do código. Cada linha traz o documento onde o item aparece, o tipo, um resumo de uma linha e a localização da origem. Para `TRANSCRICAO`, a localização é `[hh:mm] Nome`, com o minuto e quem falou. Para `CODIGO`, é o caminho do arquivo. Um item que é item próprio em mais de um documento aparece uma vez por documento. **(proposta)** no início do resumo marca uma escolha do documento sobre um ponto que a reunião não decidiu. Nesse caso, a localização aponta a fala ou o arquivo que motivou a proposta, e não uma decisão da reunião. **(análise)** marca uma conclusão tirada a partir da fonte, que a fala citada não enuncia. Quando o item junta mais de uma fala, a localização traz a principal, e as demais aparecem no resumo, com timestamp e nome.

**O que conta como item.** Um item é uma unidade com conteúdo verificável próprio. A regra vale igual para todos os documentos:
- do PRD: problema, público, objetivos, escopo fora, requisitos, dependências, riscos, critérios de aceitação e testes;
- do RFC: cada escolha da proposta técnica, alternativas, questões em aberto, lacunas e riscos;
- de cada ADR: a decisão, cada alternativa e o trade-off;
- do FDD: objetivos técnicos, escopo, modelo de dados, passos de fluxo, contratos, erros, resiliência, observabilidade, integração, dependências e riscos.

Ficam de fora a repetição expositiva e o que só deriva de um item já rastreado:
- os cenários de uso do PRD, que ilustram os RFs;
- o escopo incluso do PRD e do FDD, que resume os RFs;
- a tabela de decisões do PRD, que remete aos ADRs;
- o contexto e o impacto do RFC, que repetem o PRD e os riscos;
- o contexto, os fatores de decisão, os prós, os contras e as consequências de cada ADR. Eles derivam da decisão e das alternativas, e o que acrescenta comportamento reaparece como item próprio no PRD ou no FDD;
- os critérios de aceite técnicos do FDD, que verificam os objetivos técnicos e os fluxos.

A conta item a item está na seção [Cobertura](#cobertura), no fim.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-OBJ-01 | `docs/PRD.md` | Objetivo | Aviso entregue em menos de 10 s a cliente disponível; o recorte no p95 é proposta do PRD | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | `docs/PRD.md` | Objetivo | Zero divergência entre mudança de status e aviso registrado; a forma de medir (teste de integração e conferência diária em uma direção) é proposta | TRANSCRICAO | [09:06] Diego |
| PRD-OBJ-03 | `docs/PRD.md` | Objetivo | Janela de novas tentativas cobre pelo menos uma manutenção de 2 h do cliente | TRANSCRICAO | [09:16] Diego |
| PRD-OBJ-04 | `docs/PRD.md` | Objetivo | Entrar em produção até o fim de novembro, prazo pedido pela Atlas | TRANSCRICAO | [09:45] Marcos |
| PRD-FORA-01 | `docs/PRD.md` | Fora de Escopo | E-mail ao cliente quando o webhook falha repetidamente fica para a próxima fase | TRANSCRICAO | [09:37] Larissa |
| PRD-FORA-02 | `docs/PRD.md` | Fora de Escopo | Painel visual para o cliente; só endpoints nesta fase | TRANSCRICAO | [09:40] Larissa |
| PRD-FORA-03 | `docs/PRD.md` | Fora de Escopo | Rate limiting de envio por cliente: observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| PRD-FORA-04 | `docs/PRD.md` | Fora de Escopo | Arquivamento dos avisos entregues fica fora desta feature | TRANSCRICAO | [09:08] Diego |
| PRD-FORA-05 | `docs/PRD.md` | Fora de Escopo | Vários workers em paralelo são problema do futuro; ordem global não existe nem com um worker ([09:13] Larissa) | TRANSCRICAO | [09:13] Diego |
| PRD-FORA-06 | `docs/PRD.md` | Fora de Escopo | Webhooks enviados pelos clientes para a plataforma; o fluxo é só de saída | TRANSCRICAO | [09:02] Marcos |
| PRD-FORA-07 | `docs/PRD.md` | Fora de Escopo | Restringir o cadastro de webhook por role fica para depois | TRANSCRICAO | [09:37] Sofia |
| PRD-RF-01 | `docs/PRD.md` | Requisito Funcional | Usuário cadastra webhook com URL e lista de status ([09:31] Marcos); o customer vem na requisição, não do JWT | TRANSCRICAO | [09:32] Larissa |
| PRD-RF-02 | `docs/PRD.md` | Requisito Funcional | A plataforma gera a secret do webhook e a devolve na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-RF-03 | `docs/PRD.md` | Requisito Funcional | Usuário edita um webhook existente | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-04 | `docs/PRD.md` | Requisito Funcional | Usuário remove um webhook; parar os pendentes e manter o histórico é comportamento proposto no FDD | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-05 | `docs/PRD.md` | Requisito Funcional | Usuário lista os webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-06 | `docs/PRD.md` | Requisito Funcional | Cada webhook recebe só os status da lista dele ([09:33] Marcos); status que ninguém quer não gera aviso | TRANSCRICAO | [09:34] Bruno |
| PRD-RF-07 | `docs/PRD.md` | Requisito Funcional | Aviso a cada mudança de status com status anterior, novo e dados básicos, sem itens; não avisar na criação do pedido é proposta | TRANSCRICAO | [09:43] Diego |
| PRD-RF-08 | `docs/PRD.md` | Requisito Funcional | Cada aviso leva assinatura que permite verificar origem e integridade | TRANSCRICAO | [09:19] Sofia |
| PRD-RF-09 | `docs/PRD.md` | Requisito Funcional | Usuário troca a secret pela API e a antiga vale por mais 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-RF-10 | `docs/PRD.md` | Requisito Funcional | Cada aviso leva identificador de evento igual em todas as tentativas, para descartar repetições | TRANSCRICAO | [09:25] Diego |
| PRD-RF-11 | `docs/PRD.md` | Requisito Funcional | Cada aviso identifica o cadastro de webhook que o recebeu | TRANSCRICAO | [09:44] Sofia |
| PRD-RF-12 | `docs/PRD.md` | Requisito Funcional | Aviso não entregue é reenviado em intervalos crescentes e, esgotadas as tentativas, fica guardado como falha definitiva | TRANSCRICAO | [09:15] Diego |
| PRD-RF-13 | `docs/PRD.md` | Requisito Funcional | Só ADMIN reprocessa aviso em falha definitiva, e a plataforma registra quem fez | TRANSCRICAO | [09:36] Sofia |
| PRD-RF-14 | `docs/PRD.md` | Requisito Funcional | Histórico das últimas 100 entregas com sucesso ou falha, conteúdo, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-RF-15 | `docs/PRD.md` | Requisito Funcional | (proposta) Usuário desativa e reativa um webhook sem removê-lo; a reunião citou só o estado ativo na configuração; a edição vem de [09:33] Bruno | TRANSCRICAO | [09:21] Bruno |
| PRD-RF-16 | `docs/PRD.md` | Requisito Funcional | Cada aviso informa o momento do envio, pensado para detectar reenvio malicioso; fora da assinatura, a proteção depende da revisão de segurança | TRANSCRICAO | [09:44] Diego |
| PRD-RNF-01 | `docs/PRD.md` | Requisito Não Funcional | Aviso entregue em menos de 10 s a um cliente disponível | TRANSCRICAO | [09:02] Marcos |
| PRD-RNF-02 | `docs/PRD.md` | Requisito Não Funcional | O envio acrescenta até 2 s de espera antes da primeira tentativa | TRANSCRICAO | [09:10] Larissa |
| PRD-RNF-03 | `docs/PRD.md` | Requisito Não Funcional | Timeout de 10 s na chamada ao cliente; depois disso conta como falha | TRANSCRICAO | [09:42] Diego |
| PRD-RNF-04 | `docs/PRD.md` | Requisito Não Funcional | URL do webhook só com HTTPS; cadastro com HTTP é recusado | TRANSCRICAO | [09:23] Sofia |
| PRD-RNF-05 | `docs/PRD.md` | Requisito Não Funcional | Aviso acima de 64 KB não é enviado nem truncado; é erro | TRANSCRICAO | [09:24] Larissa |
| PRD-RNF-06 | `docs/PRD.md` | Requisito Não Funcional | Garantia at-least-once: o cliente pode receber o mesmo aviso mais de uma vez | TRANSCRICAO | [09:24] Diego |
| PRD-RNF-07 | `docs/PRD.md` | Requisito Não Funcional | Ordem só dentro do mesmo pedido e com um único processo de envio; sem ordem global. A exceção do replay e o custo de segurar os seguintes são análise do FDD | TRANSCRICAO | [09:13] Larissa |
| PRD-RNF-08 | `docs/PRD.md` | Requisito Não Funcional | Secret única por endpoint, nunca global | TRANSCRICAO | [09:21] Sofia |
| PRD-RNF-09 | `docs/PRD.md` | Requisito Não Funcional | Cliente lento ou fora do ar não afeta a mudança de status de nenhum pedido | TRANSCRICAO | [09:04] Bruno |
| PRD-RNF-10 | `docs/PRD.md` | Requisito Não Funcional | Reinício da API não interrompe o envio de avisos | TRANSCRICAO | [09:11] Diego |
| PRD-RNF-11 | `docs/PRD.md` | Requisito Não Funcional | Nenhuma infraestrutura nova; mesmo MySQL e mesma stack vêm de [09:11] Diego | TRANSCRICAO | [09:07] Diego |
| PRD-DEP-01 | `docs/PRD.md` | Dependência | A mudança de status em `order.service.ts` precisa gravar o aviso na mesma transação | TRANSCRICAO | [09:40] Bruno |
| PRD-DEP-02 | `docs/PRD.md` | Dependência | Revisão de segurança da Sofia sobre HMAC e geração de secret, com dois dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-DEP-03 | `docs/PRD.md` | Dependência | Sessão de revisão do design com Bruno e Diego antes de codar | TRANSCRICAO | [09:50] Larissa |
| PRD-DEP-04 | `docs/PRD.md` | Dependência | Um processo novo, separado da API, sobe e é acompanhado em produção | TRANSCRICAO | [09:11] Diego |
| PRD-DEP-05 | `docs/PRD.md` | Dependência | Marcos documenta a integração no portal do desenvolvedor, com destaque para descartar repetições | TRANSCRICAO | [09:26] Marcos |
| PRD-DEP-06 | `docs/PRD.md` | Dependência | Marcos confirma o prazo com a Atlas | TRANSCRICAO | [09:47] Marcos |
| PRD-RISCO-01 | `docs/PRD.md` | Risco | Atrasar além do prazo da Atlas e perder o cliente para o concorrente | TRANSCRICAO | [09:00] Marcos |
| PRD-RISCO-02 | `docs/PRD.md` | Risco | Cliente não descarta avisos repetidos e processa a mesma mudança duas vezes | TRANSCRICAO | [09:24] Diego |
| PRD-RISCO-03 | `docs/PRD.md` | Risco | Secret vazada no lado do cliente, como já aconteceu uma vez | TRANSCRICAO | [09:22] Diego |
| PRD-RISCO-04 | `docs/PRD.md` | Risco | Usuário autenticado cadastra webhook para customer que não é o dele, porque `User` não tem vínculo com `Customer` | CODIGO | `prisma/schema.prisma` |
| PRD-RISCO-05 | `docs/PRD.md` | Risco | Rajada de mudanças de status gera rajada de avisos para um cliente | TRANSCRICAO | [09:38] Diego |
| PRD-RISCO-06 | `docs/PRD.md` | Risco | Qualquer pessoa cria conta ADMIN pelo cadastro público e reprocessa avisos; problema do código existente | CODIGO | `src/modules/auth/auth.schemas.ts` |
| PRD-TESTE-01 | `docs/PRD.md` | Restrição | Testes de integração com Vitest e Supertest, no padrão da suíte atual | CODIGO | `tests/orders.test.ts` |
| PRD-TESTE-02 | `docs/PRD.md` | Proposta | (proposta) Worker validado contra um servidor HTTP local que responde sucesso, erro, demora ou queda | TRANSCRICAO | [09:42] Diego |
| PRD-TESTE-03 | `docs/PRD.md` | Proposta | (proposta) Primeiro webhook em produção com um dos três clientes que pediram a feature, antes de abrir para os demais; a reunião não definiu piloto | TRANSCRICAO | [09:00] Marcos |
| PRD-TESTE-04 | `docs/PRD.md` | Restrição | Regressão: a suíte existente de pedidos continua passando | CODIGO | `tests/orders.test.ts` |
| PRD-TESTE-05 | `docs/PRD.md` | Restrição | Regressão: a suíte existente de autenticação continua passando | CODIGO | `tests/auth.test.ts` |
| PRD-AC-01 | `docs/PRD.md` | Critério de Aceitação | Cadastro com HTTPS e lista de status devolve a secret; a recusa de HTTP vem de [09:23] Sofia | TRANSCRICAO | [09:31] Marcos |
| PRD-AC-02 | `docs/PRD.md` | Critério de Aceitação | Status listado pelo webhook gera aviso e status não listado não gera; a escolha por status vem de [09:33] Marcos | TRANSCRICAO | [09:34] Bruno |
| PRD-AC-03 | `docs/PRD.md` | Critério de Aceitação | Cliente disponível recebe o aviso em menos de 10 s, no p95 (percentil é proposta) | TRANSCRICAO | [09:02] Marcos |
| PRD-AC-04 | `docs/PRD.md` | Critério de Aceitação | Cliente valida a assinatura do aviso com a secret recebida | TRANSCRICAO | [09:20] Sofia |
| PRD-AC-05 | `docs/PRD.md` | Critério de Aceitação | Todas as tentativas de um aviso chegam com o mesmo identificador de evento | TRANSCRICAO | [09:25] Diego |
| PRD-AC-06 | `docs/PRD.md` | Critério de Aceitação | Com o cliente fora do ar, novas tentativas nos intervalos combinados; a guarda em tabela de DLQ vem de [09:18] Diego | TRANSCRICAO | [09:17] Larissa |
| PRD-AC-07 | `docs/PRD.md` | Critério de Aceitação | Só ADMIN reprocessa, e o reprocessamento fica registrado com o autor | TRANSCRICAO | [09:36] Sofia |
| PRD-AC-08 | `docs/PRD.md` | Critério de Aceitação | Depois da troca, o cliente verifica com a secret nova ou a antiga por 24 h, sem troca seguinte encurtar o prazo (forma de assinar e recusa de nova troca são propostas do FDD) | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-09 | `docs/PRD.md` | Critério de Aceitação | Histórico mostra as últimas 100 entregas com resultado, conteúdo, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-10 | `docs/PRD.md` | Critério de Aceitação | Mudança de status continua funcionando com o cliente lento ou fora do ar | TRANSCRICAO | [09:04] Bruno |
| PRD-AC-11 | `docs/PRD.md` | Critério de Aceitação | Revisão de segurança da Sofia feita antes do deploy | TRANSCRICAO | [09:46] Sofia |
| PRD-PROB-01 | `docs/PRD.md` | Contexto | Clientes consultam GET /orders periodicamente, integração lenta e cara | TRANSCRICAO | [09:00] Marcos |
| PRD-PROB-02 | `docs/PRD.md` | Contexto | O cliente quer saber da mudança sem atualizar manualmente | TRANSCRICAO | [09:02] Marcos |
| PRD-PROB-03 | `docs/PRD.md` | Risco | A Atlas pode migrar para concorrente se a feature não sair até o fim do trimestre | TRANSCRICAO | [09:00] Marcos |
| PRD-PROB-04 | `docs/PRD.md` | Restrição | Não há mecanismo de notificação, evento ou fila no código; `src/modules` só tem auth, users, customers, products e orders | CODIGO | `src/routes/index.ts` |
| PRD-PUB-01 | `docs/PRD.md` | Contexto | Público: clientes B2B integradores, a começar por Atlas, MaxDistribuição e Nova Cargo | TRANSCRICAO | [09:00] Marcos |
| PRD-PUB-02 | `docs/PRD.md` | Contexto | Público: usuários que cadastram webhooks pela API com o JWT do sistema; o vínculo com o customer é premissa a confirmar | TRANSCRICAO | [09:32] Marcos |
| PRD-PUB-03 | `docs/PRD.md` | Contexto | Público: administradores (role ADMIN) que reprocessam avisos em falha definitiva | TRANSCRICAO | [09:36] Sofia |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa Descartada | Disparo síncrono no service de orders: transação pesada e cliente lento trava outros pedidos | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa Descartada | Fila em Redis Streams: exigiria subir mais infraestrutura | TRANSCRICAO | [09:07] Larissa |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa Descartada | Trigger do banco para acordar o worker: MySQL não notifica processo externo | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa Descartada | Retry indefinido: evento fica pendurado para sempre se o cliente sumir | TRANSCRICAO | [09:15] Diego |
| RFC-ALT-05 | `docs/RFC.md` | Alternativa Descartada | Três tentativas: pouca janela para uma manutenção de duas horas | TRANSCRICAO | [09:16] Diego |
| RFC-ALT-06 | `docs/RFC.md` | Alternativa Descartada | DLQ como status "failed" na outbox: tabela separada deixa a outbox mais limpa | TRANSCRICAO | [09:18] Diego |
| RFC-ALT-07 | `docs/RFC.md` | Alternativa Descartada | Secret global da plataforma: se vaza uma, vaza tudo | TRANSCRICAO | [09:21] Sofia |
| RFC-ALT-08 | `docs/RFC.md` | Alternativa Descartada | Garantia exactly-once: exige coordenação dos dois lados | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-09 | `docs/RFC.md` | Alternativa Descartada | Montar o payload na hora do envio: o evento deixaria de refletir o momento da mudança | TRANSCRICAO | [09:52] Larissa |
| RFC-PROP-01 | `docs/RFC.md` | Decisão | Proposta 1: outbox transacional com payload montado na inserção | TRANSCRICAO | [09:08] Larissa |
| RFC-PROP-02 | `docs/RFC.md` | Decisão | Proposta 2: worker em entry-point separada, por polling, um único worker com ordem por pedido | TRANSCRICAO | [09:10] Larissa |
| RFC-PROP-03 | `docs/RFC.md` | Decisão | Proposta 3: retry com backoff exponencial e DLQ com reprocessamento por ADMIN | TRANSCRICAO | [09:17] Larissa |
| RFC-PROP-04 | `docs/RFC.md` | Decisão | Proposta 4: HMAC-SHA256 com secret por endpoint e id de evento para deduplicação ([09:26] Larissa) | TRANSCRICAO | [09:22] Sofia |
| RFC-PROP-05 | `docs/RFC.md` | Decisão | Proposta 5: módulo novo no padrão do projeto, com erros, logger, validação e controle de role existentes | TRANSCRICAO | [09:30] Larissa |
| RFC-QA-01 | `docs/RFC.md` | Questão em Aberto | Quantos envios: Larissa disse "5 tentativas" ([09:17] Larissa) e "total 5" ([09:48] Larissa), mas cinco intervalos somam quase 15 h, o que dá 1 + 5 | TRANSCRICAO | [09:17] Diego |
| RFC-QA-02 | `docs/RFC.md` | Questão em Aberto | Rate limiting de saída ficou "observar e decidir depois" | TRANSCRICAO | [09:39] Larissa |
| RFC-QA-03 | `docs/RFC.md` | Questão em Aberto | Escalar para vários workers: particionar por order_id ou lock pessimista | TRANSCRICAO | [09:13] Diego |
| RFC-QA-04 | `docs/RFC.md` | Questão em Aberto | Endurecer o acesso ao CRUD de webhooks, hoje aberto a qualquer role autenticada | TRANSCRICAO | [09:37] Sofia |
| RFC-RESTR-01 | `docs/RFC.md` | Restrição | As roles são só ADMIN e OPERATOR, sem papel de cliente para restringir o acesso por customer | CODIGO | `src/middlewares/auth.middleware.ts` |
| RFC-QA-05 | `docs/RFC.md` | Questão em Aberto | Aviso ao cliente por e-mail quando o webhook falha, talvez na próxima fase | TRANSCRICAO | [09:37] Larissa |
| RFC-QA-06 | `docs/RFC.md` | Questão em Aberto | Arquivamento da outbox "depois de 30 dias ou assim" | TRANSCRICAO | [09:08] Diego |
| RFC-QA-07 | `docs/RFC.md` | Questão em Aberto | Onde vai o customer_id no cadastro: "no body ou no path" | TRANSCRICAO | [09:32] Larissa |
| RFC-LAC-01 | `docs/RFC.md` | Lacuna | Uma linha de outbox por evento ou uma por webhook, já que um customer pode ter vários endpoints | TRANSCRICAO | [09:44] Sofia |
| RFC-LAC-02 | `docs/RFC.md` | Lacuna | Com qual secret a plataforma assina durante as 24 h de rotação | TRANSCRICAO | [09:21] Sofia |
| RFC-LAC-03 | `docs/RFC.md` | Lacuna | (análise) X-Timestamp fora da assinatura não autentica o horário; motivado pelo header de Diego e pelo HMAC só sobre o corpo de [09:22] Sofia | TRANSCRICAO | [09:44] Diego |
| RFC-LAC-04 | `docs/RFC.md` | Lacuna | Forma de guardar a secret, que precisa ser recuperável para assinar cada envio | TRANSCRICAO | [09:21] Bruno |
| RFC-LAC-05 | `docs/RFC.md` | Lacuna | Evento na criação do pedido: `OrderService.create()` nasce em PENDING sem passar por `changeStatus` | CODIGO | `src/modules/orders/order.service.ts` |
| RFC-LAC-06 | `docs/RFC.md` | Lacuna | (análise) URL apontando para a rede interna; HTTPS não impede, e a proteção exige conferir o IP no envio | TRANSCRICAO | [09:23] Sofia |
| RFC-LAC-07 | `docs/RFC.md` | Lacuna | Cadastro público sem login aceita `role: ADMIN`, o que esvazia a restrição a ADMIN do reprocessamento | CODIGO | `src/modules/auth/auth.routes.ts` |
| RFC-LAC-08 | `docs/RFC.md` | Lacuna | (análise) CRUD liberado para qualquer role, sem vínculo `User`-`Customer`, permite acesso entre customers; consequência, não decisão | CODIGO | `prisma/schema.prisma` |
| RFC-LAC-09 | `docs/RFC.md` | Lacuna | `changeStatus` lê o pedido sem lock; duas mudanças simultâneas podem gerar `from_status` que não reflete a transição real | CODIGO | `src/modules/orders/order.service.ts` |
| RFC-LAC-10 | `docs/RFC.md` | Lacuna | Desempate da ordem: `seq` autoincremental só para ordenar, contra a preferência por UUID; vai para a sessão de revisão | TRANSCRICAO | [09:51] Larissa |
| RFC-RISCO-01 | `docs/RFC.md` | Risco | Entrega duplicada: o cliente processa o mesmo evento duas vezes | TRANSCRICAO | [09:24] Diego |
| RFC-RISCO-02 | `docs/RFC.md` | Risco | (análise) Ordem perdida com retry: evento em backoff ultrapassado pelo seguinte do mesmo pedido | TRANSCRICAO | [09:12] Diego |
| RFC-RISCO-03 | `docs/RFC.md` | Risco | (análise) Latência acima de 10 s: polling de [09:10] Larissa somado à espera por vaga com clientes lentos até o timeout | TRANSCRICAO | [09:42] Diego |
| RFC-RISCO-04 | `docs/RFC.md` | Risco | Secret em log: o redact do logger não cobre a secret | CODIGO | `src/shared/logger/index.ts` |
| RFC-RISCO-05 | `docs/RFC.md` | Risco | Prazo citado como fim do trimestre ([09:00] Marcos) e depois como fim de novembro | TRANSCRICAO | [09:45] Marcos |
| ADR-001 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | Outbox no MySQL na mesma transação de `changeStatus`; atomicidade e rollback vêm de [09:06] Diego e [09:40] Bruno | TRANSCRICAO | [09:08] Larissa |
| ADR-001-ALT-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa Descartada | Disparo síncrono no service de orders | TRANSCRICAO | [09:04] Bruno |
| ADR-001-ALT-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa Descartada | Redis Streams: Redis Cluster é overengineering para um time pequeno | TRANSCRICAO | [09:07] Diego |
| ADR-001-TO | `docs/adrs/ADR-001-outbox-no-mysql.md` | Trade-off | (análise) Transação maior e tabela sem arquivamento, em troca de consistência sem infraestrutura nova ([09:06] Diego e [09:07] Diego) | TRANSCRICAO | [09:08] Diego |
| ADR-002 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Decisão | Polling a cada 2 s fechado por Larissa; processo separado e um único worker vêm de [09:11] Diego e [09:12] Diego | TRANSCRICAO | [09:10] Larissa |
| ADR-002-ALT-01 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Alternativa Descartada | Trigger do banco para reagir sem esperar o ciclo | TRANSCRICAO | [09:09] Diego |
| ADR-002-ALT-02 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Alternativa Descartada | Worker dentro do processo da API: cai junto quando a API reinicia | TRANSCRICAO | [09:11] Diego |
| ADR-002-TO | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Trade-off | Ordem só por pedido; os 2 s de espera vêm de [09:10] Larissa e o processo a mais de [09:11] Diego | TRANSCRICAO | [09:13] Larissa |
| ADR-003 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Decisão | Retry com backoff 1m/5m/30m/2h/12h; a DLQ em tabela separada e o reprocessamento manual vêm de [09:18] Diego | TRANSCRICAO | [09:17] Larissa |
| ADR-003-ALT-01 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Alternativa Descartada | Retry indefinido com backoff | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Alternativa Descartada | Três tentativas, "mais agressivo" | TRANSCRICAO | [09:16] Bruno |
| ADR-003-ALT-03 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Alternativa Descartada | DLQ como status "failed" na própria outbox | TRANSCRICAO | [09:18] Diego |
| ADR-003-TO | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Trade-off | Entregas que podem atrasar cerca de 15 h, a depender da contagem em aberto, e uma tabela a mais ([09:18] Diego) | TRANSCRICAO | [09:17] Marcos |
| ADR-004 | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Decisão | HMAC-SHA256 sobre o corpo e secret por endpoint com 24 h de carência; a geração pela plataforma vem de [09:31] Marcos | TRANSCRICAO | [09:22] Sofia |
| ADR-004-ALT-01 | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Alternativa Descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| ADR-004-ALT-02 | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Alternativa Descartada | (proposta) Secret informada pelo cliente: alternativa plausível que ninguém defendeu; a única pista é o código citado por Bruno, e foi descartada pela regra do Marcos em [09:31] | TRANSCRICAO | [09:28] Bruno |
| ADR-004-TO | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Trade-off | (análise) Custo de gerar, guardar e trocar uma secret por endpoint, em troca de vazamento de alcance limitado | TRANSCRICAO | [09:21] Sofia |
| ADR-005 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Decisão | At-least-once com id no X-Event-Id para o cliente deduplicar; o formato UUID vem de [09:25] Diego | TRANSCRICAO | [09:26] Larissa |
| ADR-005-ALT-01 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Alternativa Descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| ADR-005-TO | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Trade-off | O cliente assume a deduplicação; as repetições vêm de [09:24] Diego e a deduplicação por id de [09:25] Diego | TRANSCRICAO | [09:25] Sofia |
| ADR-006 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Reuso máximo: módulo padrão, AppError, Pino, error middleware, schemas Zod e códigos de erro | TRANSCRICAO | [09:30] Larissa |
| ADR-006-ALT-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | (proposta) Infraestrutura própria de log e erro para o módulo: alternativa plausível que ninguém propôs, descartada pelo "não vamos botar nada novo" | TRANSCRICAO | [09:29] Bruno |
| ADR-006-ALT-02 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | (proposta) Usar os códigos genéricos, sem prefixo WEBHOOK_: alternativa plausível que ninguém propôs, descartada pelo prefixo fixado por Larissa | TRANSCRICAO | [09:29] Larissa |
| ADR-006-TO | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Trade-off | As peças existentes precisam de ajuste: classes com código fixo; Zod com código genérico (`validate.middleware.ts`); redact sem a secret (`logger/index.ts`) | CODIGO | `src/shared/errors/http-errors.ts` |
| ADR-006-RESTR-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Restrição | O middleware de erro é do Express e não cobre o processo do worker | CODIGO | `src/middlewares/error.middleware.ts` |
| ADR-007 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Decisão | Payload montado e gravado na outbox no momento da inserção | TRANSCRICAO | [09:52] Larissa |
| ADR-007-ALT-01 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Alternativa Descartada | Guardar só o order_id e montar o payload no envio | TRANSCRICAO | [09:51] Bruno |
| ADR-007-TO | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Trade-off | (análise) Linhas maiores e evento que pode descrever estado superado ([09:52] Larissa); o cliente consulta o pedido para o atual | TRANSCRICAO | [09:43] Diego |
| FDD-OT-1 | `docs/FDD.md` | Objetivo | Toda mudança de status com interessado ([09:34] Bruno) gera evento na outbox; mudança revertida não gera | TRANSCRICAO | [09:06] Diego |
| FDD-OT-2 | `docs/FDD.md` | Objetivo | Com vaga livre, a primeira tentativa começa até 2 s depois da inserção; medida pelo relógio do MySQL, desde a inserção e não do commit | TRANSCRICAO | [09:10] Larissa |
| FDD-OT-3 | `docs/FDD.md` | Objetivo | Cliente saudável recebe o evento em menos de 10 s | TRANSCRICAO | [09:02] Marcos |
| FDD-OT-4 | `docs/FDD.md` | Objetivo | Falha ou timeout ([09:42] Diego) agenda tentativas em 1 min, 5 min, 30 min, 2 h e 12 h e depois move para a DLQ ([09:18] Diego) | TRANSCRICAO | [09:17] Larissa |
| FDD-OT-5 | `docs/FDD.md` | Objetivo | Todo envio leva HMAC-SHA256 verificável com a secret do endpoint | TRANSCRICAO | [09:22] Sofia |
| FDD-OT-6 | `docs/FDD.md` | Objetivo | Eventos de um pedido para um webhook chegam na ordem de criação, com um único worker | TRANSCRICAO | [09:13] Larissa |
| FDD-OT-7 | `docs/FDD.md` | Objetivo | Nenhum payload acima de 64 KB é enviado | TRANSCRICAO | [09:24] Larissa |
| FDD-ESC-01 | `docs/FDD.md` | Proposta | (proposta) A criação do pedido não gera evento, porque `create()` não passa por `changeStatus` | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-ESC-02 | `docs/FDD.md` | Fora de Escopo | E-mail ao cliente quando o webhook falha | TRANSCRICAO | [09:37] Larissa |
| FDD-ESC-03 | `docs/FDD.md` | Fora de Escopo | Rate limiting de saída | TRANSCRICAO | [09:39] Larissa |
| FDD-ESC-04 | `docs/FDD.md` | Fora de Escopo | Dashboard visual | TRANSCRICAO | [09:40] Larissa |
| FDD-ESC-05 | `docs/FDD.md` | Fora de Escopo | Arquivamento da outbox | TRANSCRICAO | [09:08] Diego |
| FDD-ESC-06 | `docs/FDD.md` | Fora de Escopo | Vários workers em paralelo | TRANSCRICAO | [09:13] Diego |
| FDD-ESC-07 | `docs/FDD.md` | Fora de Escopo | Webhooks de entrada | TRANSCRICAO | [09:02] Marcos |
| FDD-DADOS-01 | `docs/FDD.md` | Restrição | Tabelas novas com id UUID, no padrão do resto do projeto | TRANSCRICAO | [09:51] Larissa |
| FDD-DADOS-02 | `docs/FDD.md` | Proposta | (proposta) Tabela `webhook_endpoints` com url, secret, `customerId`, ativo, statuses e campos de rotação, a partir de "url + secret + customer_id + estado ativo" | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-03 | `docs/FDD.md` | Proposta | Tabela `webhook_outbox` com estados pendente, processando, falhou e entregue e índices em status e created_at, como Diego falou; nomes de tabela e colunas são proposta | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-04 | `docs/FDD.md` | Proposta | Tabela `webhook_dead_letter` com payload, motivo e momento da falha (os nomes são proposta) | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-05 | `docs/FDD.md` | Proposta | (proposta) Tabela `webhook_deliveries` com sucesso, resposta, início e duração de cada tentativa | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-06 | `docs/FDD.md` | Proposta | (proposta) Uma linha de outbox por par evento e webhook, com o mesmo event_id | TRANSCRICAO | [09:44] Sofia |
| FDD-DADOS-07 | `docs/FDD.md` | Proposta | (proposta) `statuses` como coluna Json, como `address` de customers no schema MySQL; o Prisma não tem lista escalar no MySQL | CODIGO | `prisma/schema.prisma` |
| FDD-DADOS-08 | `docs/FDD.md` | Proposta | (proposta) FAILED encerra o ciclo automático; DLQ só para tentativas esgotadas e payload grande; webhook removido fica sem DLQ; só o replay reabre | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-09 | `docs/FDD.md` | Proposta | (proposta) Remoção lógica com `deletedAt`, mantendo outbox, histórico e DLQ consistentes | TRANSCRICAO | [09:33] Bruno |
| FDD-DADOS-10 | `docs/FDD.md` | Proposta | (proposta) FK restritiva de `webhook_endpoints.customerId` para customers, igual à de `orders.customerId` | CODIGO | `prisma/schema.prisma` |
| FDD-DADOS-11 | `docs/FDD.md` | Proposta | (proposta) O payload fica só na outbox e na DLQ; o histórico aponta para a outbox | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-12 | `docs/FDD.md` | Proposta | (proposta) `responseBody` guarda até 4 KB, lidos com limite, e `responseTruncated` marca o corte | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-13 | `docs/FDD.md` | Proposta | (proposta) Secret legível pelo worker, fora das respostas exceto criação e rotação, nunca em log; motivada pelo vazamento de secret em log de cliente | TRANSCRICAO | [09:22] Diego |
| FDD-DADOS-14 | `docs/FDD.md` | Fora de Escopo | (proposta) Sem endpoint de listagem da DLQ: o ADMIN acha o id pelo log ou pelo banco; a reunião só pediu o reprocessamento | TRANSCRICAO | [09:35] Diego |
| FDD-DADOS-15 | `docs/FDD.md` | Proposta | (proposta) Campos novos em camelCase sem `@map`, seguindo a coluna física `customerId` de orders; só a tabela usa `@@map` | CODIGO | `prisma/migrations/20260519182739_init/migration.sql` |
| FDD-DADOS-16 | `docs/FDD.md` | Proposta | (proposta) Coluna `seq` autoincremental só para ordenar, porque `createdAt` em DATETIME(3) empata; o id segue UUID e o conflito com "Tudo é uuid" de [09:51] Larissa vai para revisão | CODIGO | `prisma/schema.prisma` |
| FDD-DADOS-17 | `docs/FDD.md` | Proposta | (proposta) `failureReason` na outbox guarda o motivo de FAILED, inclusive `WEBHOOK_DELETED`, que não tem DLQ | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-18 | `docs/FDD.md` | Proposta | (proposta) Tabela de transições da outbox com estado anterior, condição, estado final e o que grava junto | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-19 | `docs/FDD.md` | Proposta | (proposta) Índice em `(status, nextAttemptAt)` para a seleção do worker | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-20 | `docs/FDD.md` | Proposta | (proposta) `replayCount` na outbox e nas tentativas separa envios depois de reprocessamento | TRANSCRICAO | [09:35] Diego |
| FDD-DADOS-21 | `docs/FDD.md` | Proposta | (proposta) `claimToken` (UUID por claim) localiza a linha na conclusão e na recuperação sem comparar datas | TRANSCRICAO | [09:12] Diego |
| FDD-DADOS-22 | `docs/FDD.md` | Proposta | (proposta) Datas em UTC: Prisma grava em UTC e o SQL usa `UTC_TIMESTAMP(3)`; contas entre API e MySQL pressupõem NTP | CODIGO | `prisma/migrations/20260519182739_init/migration.sql` |
| FDD-FLUXO-01 | `docs/FDD.md` | Integração | `changeStatus` chama `publishWebhookEvent(tx, order, fromStatus, toStatus)` dentro da transação | TRANSCRICAO | [09:41] Bruno |
| FDD-FLUXO-02 | `docs/FDD.md` | Requisito Funcional | Filtro de status na inserção; sem webhook interessado, nada é gravado | TRANSCRICAO | [09:34] Bruno |
| FDD-FLUXO-03 | `docs/FDD.md` | Proposta | (proposta) Webhook pausado recebe o evento na outbox e a linha fica PENDING até a reativação; a reunião só citou o estado ativo | TRANSCRICAO | [09:21] Bruno |
| FDD-FLUXO-04 | `docs/FDD.md` | Decisão | Payload montado uma vez na inserção, com `to_status` igual ao status novo | TRANSCRICAO | [09:52] Larissa |
| FDD-FLUXO-05 | `docs/FDD.md` | Contrato | event_id é um UUID gerado na inserção na outbox | TRANSCRICAO | [09:25] Diego |
| FDD-FLUXO-06 | `docs/FDD.md` | Proposta | (proposta) Payload acima de 64 KB entra como FAILED e vai para a DLQ com `WEBHOOK_PAYLOAD_TOO_LARGE`, sem desfazer o status | TRANSCRICAO | [09:24] Larissa |
| FDD-FLUXO-07 | `docs/FDD.md` | Proposta | (proposta) `requestId` vai inteiro para a outbox, por um quinto parâmetro opcional do publisher, para casar com o log da API | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-FLUXO-08 | `docs/FDD.md` | Decisão | Ciclo do worker a cada 2 s | TRANSCRICAO | [09:09] Diego |
| FDD-FLUXO-09 | `docs/FDD.md` | Proposta | (proposta) No máximo 10 envios simultâneos, a partir do "batch pequeno" | TRANSCRICAO | [09:08] Diego |
| FDD-FLUXO-10 | `docs/FDD.md` | Decisão | Linhas processadas na ordem de criação; a ordem de created_at citada vira ordem de `seq` (proposta), sem empate | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-11 | `docs/FDD.md` | Proposta | (proposta) Linha só é elegível sem outra de `seq` menor do mesmo webhook e pedido pendente ou em envio, via `NOT EXISTS` antes do LIMIT | TRANSCRICAO | [09:13] Larissa |
| FDD-FLUXO-12 | `docs/FDD.md` | Proposta | (proposta) Passo no início do ciclo marca FAILED com `WEBHOOK_DELETED` as pendentes de webhook removido, sem DLQ e sem envio | TRANSCRICAO | [09:33] Bruno |
| FDD-FLUXO-13 | `docs/FDD.md` | Proposta | (proposta) Claim condicional com `claimToken`, que revalida estado, vencimento e webhook ativo; mantém o desenho de um único worker | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-14 | `docs/FDD.md` | Proposta | (proposta) O ciclo preenche vagas livres e dispara cada envio sem esperar o mais lento, para clientes lentos não segurarem a meta de 10 s | TRANSCRICAO | [09:02] Marcos |
| FDD-FLUXO-15 | `docs/FDD.md` | Proposta | (proposta) POST para a URL do webhook com `fetch` nativo e timeout de 10 s; o `fetch` é escolha do FDD, o timeout veio da reunião | TRANSCRICAO | [09:42] Diego |
| FDD-FLUXO-15B | `docs/FDD.md` | Restrição | `package.json` exige Node >=20, versão que já traz `fetch` e `AbortSignal.timeout` sem dependência nova | CODIGO | `package.json` |
| FDD-FLUXO-16 | `docs/FDD.md` | Proposta | (proposta) Tabela de envios com 1 envio original mais 5 retentativas (1 min, 5 min, 30 min, 2 h, 12 h); a contagem é premissa provisória, questão 1 do RFC | TRANSCRICAO | [09:17] Diego |
| FDD-FLUXO-17 | `docs/FDD.md` | Proposta | (proposta) Intervalos de retry como constantes do módulo, sem configuração | TRANSCRICAO | [09:17] Larissa |
| FDD-FLUXO-18 | `docs/FDD.md` | Decisão | Na última falha, entra linha na DLQ com payload, motivo e `failedAt`; o estado FAILED e a transação única são proposta | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-19 | `docs/FDD.md` | Decisão | Endpoint admin de reprocessamento; o reprocessamento manual que volta o evento para pendente vem de [09:18] Diego | TRANSCRICAO | [09:35] Diego |
| FDD-FLUXO-20 | `docs/FDD.md` | Proposta | (proposta) Reprocessamento por update condicional, attempts zerado, `replayCount` + 1, mesmo event_id e nova linha na DLQ se esgotar de novo | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-21 | `docs/FDD.md` | Proposta | (proposta) No reprocessamento, payload grande demais responde 422; webhook removido ([09:33] Bruno) responde 404 | TRANSCRICAO | [09:24] Larissa |
| FDD-FLUXO-22 | `docs/FDD.md` | Requisito Funcional | Rotação move a secret atual para `previousSecret` com validade de 24 h | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-23 | `docs/FDD.md` | Proposta | (proposta) Na carência, o worker envia as duas assinaturas no mesmo header, separadas por vírgula | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-24 | `docs/FDD.md` | Proposta | (proposta) Só uma secret anterior; nova rotação durante a carência responde 409, para nenhuma secret perder as 24 h | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-25 | `docs/FDD.md` | Proposta | (proposta) Secret gerada com 32 bytes de `crypto.randomBytes`, em hex, sujeita à revisão da Sofia | TRANSCRICAO | [09:46] Sofia |
| FDD-FLUXO-26 | `docs/FDD.md` | Proposta | (proposta) `publishWebhookEvent` devolve os pares eventId e webhookId gravados, logados só depois do commit | TRANSCRICAO | [09:41] Bruno |
| FDD-FLUXO-27 | `docs/FDD.md` | Proposta | (proposta) Seleção filtra webhook ativo e não removido antes do LIMIT, para linhas pausadas não ocuparem as vagas | TRANSCRICAO | [09:21] Bruno |
| FDD-FLUXO-28 | `docs/FDD.md` | Proposta | (proposta) Conclusão de cada tentativa numa transação curta (tentativa, outbox e DLQ), que localiza a linha pelo `claimToken` com `FOR UPDATE` | TRANSCRICAO | [09:34] Marcos |
| FDD-FLUXO-29 | `docs/FDD.md` | Proposta | (proposta) Corpo da resposta lido em stream até 4 KB, resto cancelado; o 2xx nos headers decide o sucesso | TRANSCRICAO | [09:34] Marcos |
| FDD-FLUXO-30 | `docs/FDD.md` | Proposta | (proposta) Recuperação de PROCESSING grava `startedAt = lockedAt` e `durationMs` nulo | TRANSCRICAO | [09:24] Diego |
| FDD-FLUXO-31 | `docs/FDD.md` | Proposta | (proposta) Meta de 2 s vale com vaga livre; com as vagas ocupadas por clientes lentos, a espera pode chegar a cerca de 12 s | TRANSCRICAO | [09:10] Larissa |
| FDD-CONTRATO-01 | `docs/FDD.md` | Contrato | Rotas sob `/api/v1` (`app.ts`), com `Authorization: Bearer <jwt>` via `authenticate` de `auth.middleware.ts` | CODIGO | `src/app.ts` |
| FDD-CONTRATO-02 | `docs/FDD.md` | Contrato | Erros no formato `{ "error": { "code", "message", "details?" } }` do `errorMiddleware` | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-CONTRATO-03 | `docs/FDD.md` | Proposta | (proposta) `customerId` no body na criação e na query na listagem, no padrão dos schemas de orders | CODIGO | `src/modules/orders/order.schemas.ts` |
| FDD-CONTRATO-04 | `docs/FDD.md` | Restrição | CRUD de configuração aceita qualquer role autenticada, por enquanto | TRANSCRICAO | [09:37] Sofia |
| FDD-CONTRATO-05 | `docs/FDD.md` | Proposta | (proposta) Rotação de secret e histórico de entregas seguem a mesma regra de role do CRUD | TRANSCRICAO | [09:36] Marcos |
| FDD-CONTRATO-06 | `docs/FDD.md` | Contrato | `POST /api/v1/webhooks` cadastra o webhook e devolve a secret na criação; o prefixo e o 201 seguem o padrão da API | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-07 | `docs/FDD.md` | Contrato | `GET /api/v1/webhooks?customerId=` lista os webhooks de um customer; a query e a ausência de secret são proposta | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-08 | `docs/FDD.md` | Contrato | (proposta) `PATCH /api/v1/webhooks/:id` aceita só `url`, `statuses` e `active`; a reunião disse só "PATCH pra editar" | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-09 | `docs/FDD.md` | Contrato | Mudança de `statuses` vale só para eventos novos, porque o filtro roda na inserção | TRANSCRICAO | [09:34] Bruno |
| FDD-CONTRATO-10 | `docs/FDD.md` | Contrato | `DELETE /api/v1/webhooks/:id` remove o webhook; 204 sem corpo, como os outros DELETE da API | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-11 | `docs/FDD.md` | Proposta | (proposta) `POST /api/v1/webhooks/:id/secret/rotate` como caminho do endpoint de rotação | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-12 | `docs/FDD.md` | Contrato | `GET /api/v1/webhooks/:id/deliveries` devolve as 100 tentativas mais recentes, sem paginação | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-13 | `docs/FDD.md` | Contrato | `POST /api/v1/admin/webhooks/dead-letter/:id/replay`; ADMIN vem de [09:36] Larissa e o 202 é escolha do FDD | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRATO-14 | `docs/FDD.md` | Proposta | (proposta) A secret só volta na criação e na rotação, nunca em listagem ou edição | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-15 | `docs/FDD.md` | Contrato | Payload em snake_case: event_id, event_type, timestamp, order_id, order_number, from_status, to_status, customer_id, total_cents, sem items | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-16 | `docs/FDD.md` | Proposta | (proposta) `timestamp` do payload é o momento da mudança de status; o do envio vai no header | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-17 | `docs/FDD.md` | Contrato | Header `Content-Type: application/json` | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-18 | `docs/FDD.md` | Contrato | Header `X-Event-Id` com o event_id; igual em todas as tentativas por [09:25] Diego; manter no reprocessamento é proposta | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-19 | `docs/FDD.md` | Contrato | Header `X-Webhook-Id` com o id do cadastro que recebeu o envio | TRANSCRICAO | [09:44] Sofia |
| FDD-CONTRATO-20 | `docs/FDD.md` | Contrato | Header `X-Timestamp` com o momento do envio; ISO 8601 no header é proposta (Diego pediu ISO 8601 no payload) | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-21 | `docs/FDD.md` | Contrato | Header `X-Signature` com HMAC-SHA256 do corpo; o formato `sha256=<hex>` é proposta | TRANSCRICAO | [09:20] Sofia |
| FDD-CONTRATO-22 | `docs/FDD.md` | Proposta | (proposta) Resposta 2xx em até 10 s é sucesso; o resto conta como falha, sem distinguir 4xx de 5xx | TRANSCRICAO | [09:42] Diego |
| FDD-CONTRATO-23 | `docs/FDD.md` | Proposta | (proposta) `customerId` não muda depois do cadastro; corpo vazio, campo fora da lista, `statuses` vazia ou repetida respondem 400 | TRANSCRICAO | [09:32] Larissa |
| FDD-CONTRATO-24 | `docs/FDD.md` | Proposta | (proposta) Troca de `url` vale também para os pendentes, porque o worker lê a URL atual no envio | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-25 | `docs/FDD.md` | Contrato | Exemplos de requisição sem corpo para GET, DELETE, rotação e reprocessamento; 204 sem corpo no DELETE | CODIGO | `src/modules/customers/customer.controller.ts` |
| FDD-ERRO-01 | `docs/FDD.md` | Restrição | Erros do módulo com prefixo `WEBHOOK_`; estender `AppError` vem de [09:28] Bruno | TRANSCRICAO | [09:29] Larissa |
| FDD-ERRO-02 | `docs/FDD.md` | Restrição | `NotFoundError` tem código fixo `NOT_FOUND`, então os 404 do módulo estendem `AppError` direto | CODIGO | `src/shared/errors/http-errors.ts` |
| FDD-ERRO-03 | `docs/FDD.md` | Erro | `WEBHOOK_NOT_FOUND` citado por Bruno; 404 segue o padrão da API e tratar removido como inexistente é proposta | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-04 | `docs/FDD.md` | Erro | `WEBHOOK_INVALID_URL` citado por Bruno; a regra do HTTPS vem de [09:23] Sofia e o 400 segue o padrão | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-05 | `docs/FDD.md` | Erro | (proposta) `WEBHOOK_CUSTOMER_NOT_FOUND` (404): customerId inexistente no cadastro, checado com `findById` | CODIGO | `src/modules/customers/customer.repository.ts` |
| FDD-ERRO-06 | `docs/FDD.md` | Erro | (proposta) `WEBHOOK_DEAD_LETTER_NOT_FOUND` (404): item da DLQ inexistente | TRANSCRICAO | [09:35] Diego |
| FDD-ERRO-07 | `docs/FDD.md` | Erro | (proposta) `WEBHOOK_ALREADY_REPLAYED` (409): item já reprocessado ou outbox fora de FAILED | TRANSCRICAO | [09:18] Diego |
| FDD-ERRO-08 | `docs/FDD.md` | Erro | `WEBHOOK_PAYLOAD_TOO_LARGE` (422 no reprocessamento e motivo na DLQ): regra da reunião, nome proposto | TRANSCRICAO | [09:24] Larissa |
| FDD-ERRO-09 | `docs/FDD.md` | Erro | `VALIDATION_ERROR` (400): corpo, query ou params fora do schema Zod; código existente | CODIGO | `src/middlewares/validate.middleware.ts` |
| FDD-ERRO-10 | `docs/FDD.md` | Erro | `UNAUTHORIZED` e `FORBIDDEN` (401/403): sem token, ou role diferente de ADMIN no reprocessamento; códigos existentes | CODIGO | `src/middlewares/auth.middleware.ts` |
| FDD-ERRO-11 | `docs/FDD.md` | Erro | `WEBHOOK_DELIVERY_TIMEOUT`: cliente não respondeu em 10 s; regra da reunião, nome proposto | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-12 | `docs/FDD.md` | Erro | (proposta) `WEBHOOK_DELIVERY_HTTP_ERROR`: resposta fora de 2xx, inclusive redirect | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-13 | `docs/FDD.md` | Erro | (proposta) `WEBHOOK_DELIVERY_NETWORK_ERROR`: falha de DNS, conexão ou TLS com o cliente fora do ar | TRANSCRICAO | [09:15] Diego |
| FDD-ERRO-14 | `docs/FDD.md` | Erro | (proposta) `WEBHOOK_DELIVERY_INTERRUPTED`: worker caiu durante o envio | TRANSCRICAO | [09:24] Diego |
| FDD-ERRO-15 | `docs/FDD.md` | Erro | (proposta) `WEBHOOK_DELETED`: linha de webhook removido vai para FAILED sem DLQ | TRANSCRICAO | [09:33] Bruno |
| FDD-ERRO-16 | `docs/FDD.md` | Proposta | (proposta) Schema Zod valida o formato da URL e o service rejeita o que não é https, porque o `validate()` transforma todo erro Zod em `VALIDATION_ERROR` | CODIGO | `src/middlewares/validate.middleware.ts` |
| FDD-ERRO-17 | `docs/FDD.md` | Decisão | `WEBHOOK_SECRET_REQUIRED`, citado como exemplo em [09:28] Bruno, não entra na matriz porque a secret é gerada pela plataforma | TRANSCRICAO | [09:31] Marcos |
| FDD-ERRO-18 | `docs/FDD.md` | Erro | Falha ao gravar na outbox derruba a transação; o 500 vem do `errorMiddleware` para erro que não é `AppError` | TRANSCRICAO | [09:40] Bruno |
| FDD-ERRO-19 | `docs/FDD.md` | Erro | (proposta) `WEBHOOK_SECRET_ROTATION_IN_PROGRESS` (409): rotação pedida durante a carência de 24 h | TRANSCRICAO | [09:21] Sofia |
| FDD-RES-01 | `docs/FDD.md` | Restrição | Cliente lento: timeout de 10 s, conta como falha e entra no retry | TRANSCRICAO | [09:42] Diego |
| FDD-RES-02 | `docs/FDD.md` | Restrição | Cliente fora do ar: retry com 1 min, 5 min, 30 min, 2 h e 12 h e depois DLQ ([09:18] Diego) | TRANSCRICAO | [09:17] Larissa |
| FDD-RES-03 | `docs/FDD.md` | Proposta | (proposta) Redirect conta como falha e não é seguido, para não trocar https por http | TRANSCRICAO | [09:23] Sofia |
| FDD-RES-04 | `docs/FDD.md` | Restrição | Falha na gravação do evento faz rollback da mudança de status | TRANSCRICAO | [09:40] Bruno |
| FDD-RES-05 | `docs/FDD.md` | Restrição | API reinicia e o worker, outro processo, segue entregando | TRANSCRICAO | [09:11] Diego |
| FDD-RES-06 | `docs/FDD.md` | Proposta | (proposta) Linha presa em PROCESSING há mais de 60 s conta como tentativa falha; o resultado remoto é desconhecido e o cliente deduplica a repetição | TRANSCRICAO | [09:24] Diego |
| FDD-RES-07 | `docs/FDD.md` | Proposta | (proposta) Banco indisponível: o ciclo loga `webhook_worker_cycle_failed` e tenta de novo sem derrubar o processo | CODIGO | `src/shared/logger/index.ts` |
| FDD-RES-08 | `docs/FDD.md` | Proposta | (proposta) Dois workers por engano: o claim impede pegar a mesma linha ao mesmo tempo e antecipar retry; não garante ordem nem ausência de duplicata | TRANSCRICAO | [09:12] Diego |
| FDD-RES-09 | `docs/FDD.md` | Restrição | Payload acima de 64 KB não é enviado nem truncado (reunião); ir para a DLQ é proposta do FDD | TRANSCRICAO | [09:24] Larissa |
| FDD-RES-10 | `docs/FDD.md` | Fora de Escopo | Sem aviso por e-mail nesta fase; fica para a próxima | TRANSCRICAO | [09:37] Larissa |
| FDD-RES-11 | `docs/FDD.md` | Proposta | (proposta) Encerramento do worker espera os envios em andamento, ao contrário do `process.exit(0)` da API | CODIGO | `src/server.ts` |
| FDD-OBS-01 | `docs/FDD.md` | Proposta | (proposta) Logs estruturados com o Pino existente: enqueued, succeeded, failed, dead letter, replay, rotação e falha de ciclo | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | `docs/FDD.md` | Requisito Não Funcional | Log `webhook_replay_requested` com o userId de quem pediu, para auditoria | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-02B | `docs/FDD.md` | Proposta | (proposta) Log `webhook_secret_rotated` com o autor da rotação | TRANSCRICAO | [09:21] Sofia |
| FDD-OBS-02C | `docs/FDD.md` | Proposta | (proposta) Log `webhook_worker_stats` a cada 60 s com backlog, DLQ e envios por webhook, para medir sem dependência nova | TRANSCRICAO | [09:39] Larissa |
| FDD-OBS-03 | `docs/FDD.md` | Proposta | (proposta) `webhook_event_enqueued` só é logado depois que a transação resolve, para não registrar evento revertido | TRANSCRICAO | [09:06] Diego |
| FDD-OBS-04 | `docs/FDD.md` | Proposta | (proposta) Regra principal: logar só campos planos, sem objeto do webhook, headers ou erro inteiro; redact como segunda barreira, com caminhos de secret e `x-signature` | CODIGO | `src/shared/logger/index.ts` |
| FDD-OBS-05 | `docs/FDD.md` | Proposta | (proposta) O payload e o corpo da resposta do cliente não são logados, só ids | TRANSCRICAO | [09:19] Sofia |
| FDD-OBS-06 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_outbox_pending` para acompanhar o backlog da outbox | TRANSCRICAO | [09:07] Bruno |
| FDD-OBS-07 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_delivery_latency_seconds` contra a meta de 10 s, só sobre a primeira entrega com sucesso, sem retry nem replay | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-08 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_deliveries_total{result}` por sucesso, erro HTTP, timeout e rede | TRANSCRICAO | [09:42] Diego |
| FDD-OBS-09 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_dead_letter_total` de eventos que esgotaram as tentativas | TRANSCRICAO | [09:18] Diego |
| FDD-OBS-10 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_sends_per_webhook_per_minute` para decidir o rate limiting com dado | TRANSCRICAO | [09:39] Larissa |
| FDD-OBS-11 | `docs/FDD.md` | Proposta | (proposta) Coleta pelos logs e por `webhook_worker_stats` a cada 60 s, sem biblioteca de métricas nova | CODIGO | `package.json` |
| FDD-OBS-12 | `docs/FDD.md` | Proposta | (proposta) Tracing por correlação: X-Request-Id inteiro na outbox; event_id e outboxId nas tentativas, na DLQ e nos logs de falha | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-OBS-13 | `docs/FDD.md` | Proposta | (proposta) O worker loga com o nome de serviço `order-management-webhook-worker`, porque o logger fixa `order-management-api` | CODIGO | `src/shared/logger/index.ts` |
| FDD-OBS-14 | `docs/FDD.md` | Proposta | (proposta) Fórmula da latência: `startedAt + durationMs - createdAt`, primeira tentativa com sucesso, janela de 24 h, relógio do MySQL | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-15 | `docs/FDD.md` | Proposta | (proposta) Gauges de idade da linha elegível mais antiga (sem os eventos que esperam o anterior) e de falha na primeira tentativa, lidos junto com o p95 | TRANSCRICAO | [09:07] Bruno |
| FDD-OBS-16 | `docs/FDD.md` | Proposta | (proposta) Log `webhook_delivery_result_discarded` para resultado que chega depois da recuperação da linha | TRANSCRICAO | [09:24] Diego |
| FDD-OBS-17 | `docs/FDD.md` | Proposta | (proposta) Log diário `webhook_reconciliation`: avisos sem a mudança correspondente em `order_status_history`, a conferência do PRD | TRANSCRICAO | [09:06] Diego |
| FDD-OBS-18 | `docs/FDD.md` | Proposta | (proposta) O `errorMiddleware` loga o erro inteiro; o service relança sem a mensagem original as falhas nas escritas que levam a secret | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-INT-01 | `docs/FDD.md` | Integração | `order.service.ts` chama `publishWebhookEvent` na mesma transação; a posição depois do histórico é escolha da integração | TRANSCRICAO | [09:41] Bruno |
| FDD-INT-02 | `docs/FDD.md` | Integração | O snapshot usa `to`, porque o `order` lido em `changeStatus` é anterior ao update | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-INT-03 | `docs/FDD.md` | Integração | (proposta) `OrderController.changeStatus` passa também `req.id` como `requestId` | CODIGO | `src/modules/orders/order.controller.ts` |
| FDD-INT-04 | `docs/FDD.md` | Integração | `order.status.ts` sem mudança; o enum `OrderStatus` define os status aceitos, com `z.nativeEnum` como em `order.schemas.ts` | CODIGO | `src/modules/orders/order.status.ts` |
| FDD-INT-05 | `docs/FDD.md` | Integração | (proposta) Quatro modelos novos e relação `Customer` para `WebhookEndpoint` no schema Prisma | CODIGO | `prisma/schema.prisma` |
| FDD-INT-06 | `docs/FDD.md` | Integração | Uma migração nova, só aditiva, ao lado da migração inicial | CODIGO | `prisma/migrations/20260519182739_init/migration.sql` |
| FDD-INT-07 | `docs/FDD.md` | Integração | Mudança planejada: `buildControllers` passa a instanciar repository, service e controller de webhooks | CODIGO | `src/app.ts` |
| FDD-INT-08 | `docs/FDD.md` | Integração | Mudança planejada: `buildApiRouter` passa a registrar `/webhooks` e `/admin` | CODIGO | `src/routes/index.ts` |
| FDD-INT-09 | `docs/FDD.md` | Integração | `auth.middleware.ts` sem mudança; o reprocessamento usa `requireRole('ADMIN')` | TRANSCRICAO | [09:36] Larissa |
| FDD-INT-10 | `docs/FDD.md` | Integração | `validate.middleware.ts` sem mudança; o módulo usa o mesmo `validate()` | CODIGO | `src/middlewares/validate.middleware.ts` |
| FDD-INT-11 | `docs/FDD.md` | Integração | `error.middleware.ts` sem mudança, porque as classes novas estendem `AppError` | TRANSCRICAO | [09:29] Bruno |
| FDD-INT-12 | `docs/FDD.md` | Integração | `app-error.ts` e `http-errors.ts` sem mudança; as classes novas ficam no módulo | CODIGO | `src/shared/errors/app-error.ts` |
| FDD-INT-13 | `docs/FDD.md` | Integração | Logger: incluir os caminhos de secret e `x-signature` no redact e aceitar nome de serviço para o worker | CODIGO | `src/shared/logger/index.ts` |
| FDD-INT-14 | `docs/FDD.md` | Integração | `paginated()` reusado na listagem de webhooks | CODIGO | `src/shared/http/response.ts` |
| FDD-INT-15 | `docs/FDD.md` | Integração | `request-logger.middleware.ts` sem mudança; o `req.id` vai inteiro para a outbox | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-INT-16 | `docs/FDD.md` | Integração | O worker tem a própria instância do Prisma, mesmo banco e mesma DATABASE_URL; usar `createPrismaClient()` de `database.ts` é escolha da integração | TRANSCRICAO | [09:30] Bruno |
| FDD-INT-17 | `docs/FDD.md` | Integração | O worker carrega o mesmo `.env`, então `JWT_SECRET` precisa existir no ambiente dele | CODIGO | `src/config/env.ts` |
| FDD-INT-18 | `docs/FDD.md` | Integração | `src/server.ts` serve de modelo para a nova entry-point `src/worker.ts` | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-19 | `docs/FDD.md` | Integração | Script `worker` citado por Larissa; `worker:start` e os comandos completos são proposta | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-20 | `docs/FDD.md` | Integração | `customer.repository.ts` valida o `customerId` no cadastro | CODIGO | `src/modules/customers/customer.repository.ts` |
| FDD-INT-21 | `docs/FDD.md` | Integração | Fábrica `createTestWebhook` nos helpers de teste | CODIGO | `tests/helpers/factories.ts` |
| FDD-INT-21B | `docs/FDD.md` | Integração | Limpeza das quatro tabelas novas no `beforeEach`, antes de `order` | CODIGO | `tests/setup.ts` |
| FDD-INT-22 | `docs/FDD.md` | Integração | Módulo `src/modules/webhooks` (a criar) no padrão dos outros módulos; errors e processor de [09:28] Bruno, publisher de [09:41] Bruno; signer é proposta ligada ao HMAC de [09:22] Sofia | TRANSCRICAO | [09:27] Bruno |
| FDD-INT-23 | `docs/FDD.md` | Proposta | (proposta) Lógica do worker em `webhook.processor.ts`, entre os dois nomes citados na reunião | TRANSCRICAO | [09:28] Bruno |
| FDD-DEP-01 | `docs/FDD.md` | Dependência | Versões no `package.json`; `fetch` e `AbortSignal.timeout` fazem parte do Node 20; não acrescentar dependência é plano do FDD | CODIGO | `package.json` |
| FDD-DEP-02 | `docs/FDD.md` | Dependência | MySQL 8 como banco | CODIGO | `docker-compose.yml` |
| FDD-DEP-03 | `docs/FDD.md` | Dependência | (proposta) Migração aditiva; a FK nova faz a remoção de customer com webhook falhar com P2003, que o `errorMiddleware` não trata e vira 500 | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-DEP-04 | `docs/FDD.md` | Dependência | `PATCH /api/v1/orders/:id/status` mantém o contrato; muda a latência e surge uma nova causa de 500, a falha ao gravar na outbox | CODIGO | `src/modules/orders/order.routes.ts` |
| FDD-DEP-05 | `docs/FDD.md` | Dependência | Parar o worker interrompe os envios sem afetar a API; a gravação na outbox continua na transação | TRANSCRICAO | [09:11] Diego |
| FDD-DEP-06 | `docs/FDD.md` | Dependência | Revisão de segurança da Sofia com pelo menos dois dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia |
| FDD-DEP-07 | `docs/FDD.md` | Dependência | Documentação no portal do desenvolvedor pelo Marcos | TRANSCRICAO | [09:26] Marcos |
| FDD-RISCO-01 | `docs/FDD.md` | Risco | Transação de `changeStatus` mais lenta com o select de webhooks e os inserts; mitigação proposta: índice por `customerId` e medir antes e depois | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-RISCO-02 | `docs/FDD.md` | Risco | Webhook fora do ar segura os eventos seguintes do mesmo pedido, pela regra de ordem | TRANSCRICAO | [09:13] Larissa |
| FDD-RISCO-03 | `docs/FDD.md` | Risco | A contagem de envios mudar na confirmação: "total 5" de Larissa contra quase 15 h de [09:17] Diego | TRANSCRICAO | [09:48] Larissa |
| FDD-RISCO-04 | `docs/FDD.md` | Risco | Revisão de segurança mudar a assinatura, com timestamp no HMAC ou secret na carência | TRANSCRICAO | [09:46] Sofia |
| FDD-RISCO-05 | `docs/FDD.md` | Risco | Outbox e histórico crescem sem arquivamento | TRANSCRICAO | [09:08] Diego |
| FDD-RISCO-06 | `docs/FDD.md` | Risco | Rajada de mudanças gera rajada de chamadas para um cliente | TRANSCRICAO | [09:38] Diego |
| FDD-RISCO-07 | `docs/FDD.md` | Risco | (análise) Com o CRUD liberado para qualquer role ([09:37] Sofia) e sem vínculo `User`-`Customer` no schema, um autenticado pode trocar a URL de webhook de outro customer; não houve aceite explícito | TRANSCRICAO | [09:37] Sofia |
| FDD-RISCO-08 | `docs/FDD.md` | Risco | Qualquer usuário autenticado rotaciona a secret de qualquer webhook e passa a assinar envios falsos | TRANSCRICAO | [09:37] Sofia |
| FDD-RISCO-09 | `docs/FDD.md` | Risco | (análise) URL para endereço interno da rede; HTTPS e redirect bloqueado não impedem; a reunião não tratou do tema | TRANSCRICAO | [09:23] Sofia |
| FDD-RISCO-10 | `docs/FDD.md` | Risco | Evento reprocessado da DLQ chega depois dos seguintes do mesmo pedido | TRANSCRICAO | [09:18] Diego |
| FDD-RISCO-11 | `docs/FDD.md` | Risco | Cadastro público aceita ADMIN; o `requireRole` do reprocessamento confere a role, mas não basta sozinho | CODIGO | `src/modules/auth/auth.schemas.ts` |
| FDD-RISCO-12 | `docs/FDD.md` | Risco | Concorrência herdada: leitura sem lock em `changeStatus` pode gerar `from_status` incorreto; mitigação com lock fica para revisão | CODIGO | `src/modules/orders/order.service.ts` |

Total: 324 linhas. TRANSCRICAO: 263 (81,2%). CODIGO: 61.


## Cobertura

A contagem segue a regra de "O que conta como item". Cada unidade dos documentos conta como um item: linha de tabela, item de lista, passo numerado ou parágrafo com regra própria. Abaixo, cada seção lista suas unidades na ordem em que aparecem, com o ID que as cobre. Uma linha cobre mais de uma unidade quando o resumo nomeia todas, como `FDD-OBS-01` para os logs de envio. Quando o documento repete um conteúdo em outra seção, a repetição aponta para a mesma linha.

| Documento | Seção | Itens | Com linha |
| --- | --- | --- | --- |
| PRD | 2. Problema | 4 | 4 |
| PRD | 3. Público | 3 | 3 |
| PRD | 4. Objetivos (4 linhas e 2 notas) | 6 | 6 |
| PRD | 5.2 Fora de escopo | 7 | 7 |
| PRD | 6. Requisitos funcionais | 16 | 16 |
| PRD | 7. Requisitos não funcionais | 11 | 11 |
| PRD | 9. Dependências | 6 | 6 |
| PRD | 10. Riscos | 6 | 6 |
| PRD | 11. Critérios de aceitação | 11 | 11 |
| PRD | 12. Testes | 5 | 5 |
| RFC | Proposta técnica | 5 | 5 |
| RFC | Alternativas | 9 | 9 |
| RFC | Questões levantadas na reunião | 7 | 7 |
| RFC | Lacunas da análise | 10 | 10 |
| RFC | Riscos | 5 | 5 |
| ADRs | Decisão, alternativas e trade-off | 27 | 27 |
| FDD | 2. Objetivos técnicos | 7 | 7 |
| FDD | 3. Exclusões e criação do pedido | 7 | 7 |
| FDD | 4.1 Modelo de dados | 19 | 19 |
| FDD | 4.2 Criação do evento | 9 | 9 |
| FDD | 4.3 Worker | 9 | 9 |
| FDD | 4.4 Retry | 2 | 2 |
| FDD | 4.5 DLQ e reprocessamento | 5 | 5 |
| FDD | 4.6 Rotação de secret | 5 | 5 |
| FDD | 5. Regras gerais dos contratos | 7 | 7 |
| FDD | 5.1 Endpoints | 14 | 14 |
| FDD | 5.2 Envio | 10 | 10 |
| FDD | 6. Matriz de erros | 21 | 21 |
| FDD | 7. Resiliência | 14 | 14 |
| FDD | 8. Observabilidade | 26 | 26 |
| FDD | 9. Integração | 29 | 29 |
| FDD | 10. Dependências | 5 | 5 |
| FDD | 12. Riscos | 12 | 12 |
| **Total** | | **339** | **339 (100%)** |

**Unidade por unidade.**

PRD
- 2: polling caro → PRD-PROB-01; atualização manual → PROB-02; risco Atlas → PROB-03; nada para reaproveitar → PROB-04.
- 3: integradores → PRD-PUB-01; usuários que cadastram → PUB-02; administradores → PUB-03.
- 4: as quatro linhas → PRD-OBJ-01 a 04; nota da conferência em uma direção → OBJ-02; nota sobre o polling sem meta → PROB-01.
- 5.2, 6, 7, 9, 10 e 11: uma linha por unidade, na ordem do documento → PRD-FORA-01 a 07, PRD-RF-01 a 16, PRD-RNF-01 a 11, PRD-DEP-01 a 06, PRD-RISCO-01 a 05 e 06 (cadastro ADMIN), PRD-AC-01 a 11.
- 12: testes no padrão do projeto → PRD-TESTE-01; worker contra endpoint de teste → TESTE-02; revisão de segurança → PRD-DEP-02; validação com os clientes → TESTE-03; regressão → TESTE-04 e 05.

RFC
- Proposta técnica, itens 1 a 5 → RFC-PROP-01 a 05.
- Alternativas, na ordem da tabela → RFC-ALT-01 a 09.
- Questões 1 a 7 → RFC-QA-01 a 07; a restrição de roles da questão 4 também tem RFC-RESTR-01.
- Lacunas, na ordem: cardinalidade da outbox → RFC-LAC-01; secret na rotação → LAC-02; timestamp → LAC-03; armazenamento da secret → LAC-04; criação do pedido → LAC-05; desempate da ordem → LAC-10; rede interna → LAC-06; cadastro ADMIN → LAC-07; acesso entre customers → LAC-08; concorrência em `changeStatus` → LAC-09.
- Riscos, na ordem da tabela → RFC-RISCO-01 a 05.

ADRs
- Por ADR: decisão → ADR-00N; cada alternativa → ADR-00N-ALT-NN; trade-off → ADR-00N-TO. São 7 decisões, 13 alternativas e 7 trade-offs. A restrição do middleware de erro na ADR-006 também tem ADR-006-RESTR-01.

FDD
- 2: OT-1 a OT-7 → FDD-OT-1 a 7.
- 3: criação do pedido → FDD-ESC-01; as seis exclusões → ESC-02 a 07.
- 4.1: padrão UUID → FDD-DADOS-01; camelCase → DADOS-15; tabelas `webhook_endpoints`, `webhook_outbox` (com índices), `webhook_dead_letter` e `webhook_deliveries` → DADOS-02, 03 e 19, 04, 05; linha por par → DADOS-06; `seq` → DADOS-16; Json → DADOS-07; FAILED → DADOS-08 e 17; remoção lógica → DADOS-09; FK → DADOS-10; payload só na outbox e na DLQ → DADOS-11; `responseBody` → DADOS-12; `replayCount` → DADOS-20; `claimToken` → DADOS-21; relógio e fuso → DADOS-22; secret → DADOS-13; tabela de transições → DADOS-18.
- 4.2: passo 1 → FDD-FLUXO-01; passo 2 → FLUXO-02 e 03; passo 3 → FLUXO-04; passo 4 → FLUXO-05; passo 5 → FLUXO-06; passo 6 → DADOS-06; passo 7 → FLUXO-26; passo 8 → FDD-RES-04; `requestId` → FLUXO-07 e FDD-INT-03.
- 4.3: passo 1 → FLUXO-08, 09 e 14; passo 2 → FLUXO-31; passo 3 → FLUXO-10, 11 e 27; passo 4 → FLUXO-12; passo 5 → FLUXO-13; passo 6 → FLUXO-15, 15B e RES-03; passo 7 → FLUXO-29; passo 8 → FLUXO-28; passo 9 → RES-06 e FLUXO-30.
- 4.4: tabela e premissa → FLUXO-16; constantes → FLUXO-17.
- 4.5: passo 1 → FLUXO-18; passo 2 → FLUXO-19 e DADOS-14; passo 3 → FLUXO-20; passo 4 → FLUXO-21; passo 5 → FDD-OBS-02.
- 4.6: passo 1 → FDD-CONTRATO-11; passos 2 a 5 → FLUXO-22 a 25.
- 5, regras gerais: rotas e autenticação → CONTRATO-01; formato de erro → CONTRATO-02; `customerId` → CONTRATO-03; role do CRUD → CONTRATO-04; rotação e histórico → CONTRATO-05; role e acesso ao recurso → FDD-RISCO-07; limite da fronteira ADMIN → RISCO-11.
- 5.1: cadastro → CONTRATO-06; secret só na criação e na rotação → CONTRATO-14; listagem → CONTRATO-07; edição → CONTRATO-08; campos editáveis → CONTRATO-23; efeito de `statuses` → CONTRATO-09; efeito de `url` → CONTRATO-24; efeito de `active` → FLUXO-03; remoção → CONTRATO-10; semântica da remoção → FLUXO-12; exemplos sem corpo → CONTRATO-25; rotação, com o 409 → CONTRATO-11 e ERRO-19; histórico → CONTRATO-12; reprocessamento → CONTRATO-13.
- 5.2: campos do payload, lista fechada e ausência de items → CONTRATO-15; `timestamp` → CONTRATO-16; os cinco headers → CONTRATO-17 a 21; resposta esperada → CONTRATO-22.
- 6: prefixo e `AppError` → FDD-ERRO-01; classes com código fixo → ERRO-02; as nove linhas de erros da API → ERRO-03, 04, 05, 06, 07, 19, 08, 09, 10; as seis linhas de motivos → ERRO-11, 12, 13, 14, 08, 15; onde ficam os motivos → DADOS-17; onde nasce `WEBHOOK_INVALID_URL` → ERRO-16; `WEBHOOK_SECRET_REQUIRED` fora → ERRO-17; falha ao gravar na outbox → ERRO-18.
- 7, na ordem da tabela: RES-01, RES-02, RES-03, RES-04, RES-05, RES-06, FLUXO-28, FLUXO-29, FLUXO-31, RES-07, RES-08, RES-09, RES-10; encerramento → RES-11.
- 8: logs de enfileiramento → OBS-03; de sucesso, falha, DLQ e falha de ciclo → OBS-01; de descarte → OBS-16; de reprocessamento → OBS-02; de rotação → OBS-02B; de estatística → OBS-02C; de conferência → OBS-17; regra principal e redact → OBS-04 (duas unidades); payload e corpo da resposta fora do log → OBS-05 (duas unidades); erro inteiro no `errorMiddleware` → OBS-18; nome do serviço → OBS-13; as sete métricas → OBS-06, OBS-15, OBS-07, OBS-15, OBS-08, OBS-09, OBS-10; fórmula da latência → OBS-14; coleta → OBS-11; tracing → OBS-12.
- 9: as 20 linhas de arquivos existentes → FDD-INT-01 e 02 (`order.service.ts`), INT-03 a 21B, na ordem da tabela; os oito arquivos novos do módulo → INT-22; `webhook.processor.ts` → INT-23.
- 10: runtime → FDD-DEP-01 e 02; banco → DEP-03; API existente → DEP-04; desligar a feature → DEP-05; dependências humanas → DEP-06 e 07.
- 12: as doze linhas → FDD-RISCO-01 a 12. Na ordem da tabela: 01, 12, 02, 03, 04, 05, 06, 07, 08, 11, 09, 10.

**Fora da contagem.** As categorias abaixo foram excluídas pela regra do início. As quantidades estão aqui para quem quiser refazer a conta com outro critério:
- cenários de uso do PRD: 8;
- escopo incluso: 6 no PRD e 3 no FDD;
- tabela de decisões do PRD: 7;
- nos ADRs: 31 fatores de decisão, 39 marcadores de prós, contras e motivo do descarte, e 46 consequências;
- critérios de aceite técnicos do FDD: 28.
