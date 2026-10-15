# Tracker de rastreabilidade

Este tracker liga cada item do [PRD](PRD.md), do [RFC](RFC.md), do [FDD](FDD.md) e dos [ADRs](adrs/README.md) à origem dele. A origem é uma de duas: a fala da reunião técnica ([`TRANSCRICAO.md`](../TRANSCRICAO.md)) ou um arquivo do código. Cada linha traz o documento onde o item aparece, o tipo, um resumo de uma linha e a localização da origem. Para `TRANSCRICAO`, a localização é `[hh:mm] Nome`, com o minuto e quem falou. Para `CODIGO`, é o caminho do arquivo. Um item que é item próprio em mais de um documento aparece uma vez por documento. **(proposta)** no início do resumo marca uma escolha do documento sobre um ponto que a reunião não decidiu. Nesse caso, a localização aponta a fala ou o arquivo que motivou a proposta, e não uma decisão da reunião.

**O que conta como item.** Entram no tracker os itens com conteúdo próprio de cada documento:
- do PRD: requisitos, objetivos, escopo fora, riscos, dependências, critérios de aceitação e testes;
- do RFC: alternativas, questões em aberto, lacunas e riscos;
- de cada ADR: a decisão, cada alternativa e o trade-off;
- do FDD: objetivos técnicos, modelo de dados, passos de fluxo, contratos, erros, resiliência, observabilidade, integração, dependências e riscos.

Ficam de fora o que deriva de um item já rastreado: os cenários de uso do PRD, que ilustram os RFs; o escopo incluso, que resume os RFs; os prós, contras e consequências de cada ADR, que derivam da decisão e das alternativas; e os critérios de aceite técnicos do FDD, que verificam os objetivos técnicos e os fluxos.

| ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
| --- | --- | --- | --- | --- | --- |
| PRD-OBJ-01 | `docs/PRD.md` | Objetivo | Aviso entregue em menos de 10 s a cliente disponível; o recorte no p95 é proposta do PRD | TRANSCRICAO | [09:02] Marcos |
| PRD-OBJ-02 | `docs/PRD.md` | Objetivo | Zero divergência entre mudança de status e aviso registrado; a conciliação diária como forma de medir é proposta | TRANSCRICAO | [09:06] Diego |
| PRD-OBJ-03 | `docs/PRD.md` | Objetivo | Janela de novas tentativas cobre pelo menos uma manutenção de 2 h do cliente | TRANSCRICAO | [09:16] Diego |
| PRD-OBJ-04 | `docs/PRD.md` | Objetivo | Entrar em produção até o fim de novembro, prazo pedido pela Atlas | TRANSCRICAO | [09:45] Marcos |
| PRD-FORA-01 | `docs/PRD.md` | Fora de Escopo | E-mail ao cliente quando o webhook falha repetidamente fica para a próxima fase | TRANSCRICAO | [09:37] Larissa |
| PRD-FORA-02 | `docs/PRD.md` | Fora de Escopo | Painel visual para o cliente; só endpoints nesta fase | TRANSCRICAO | [09:40] Larissa |
| PRD-FORA-03 | `docs/PRD.md` | Fora de Escopo | Rate limiting de envio por cliente: observar e decidir depois | TRANSCRICAO | [09:39] Larissa |
| PRD-FORA-04 | `docs/PRD.md` | Fora de Escopo | Arquivamento dos avisos entregues fica fora desta feature | TRANSCRICAO | [09:08] Diego |
| PRD-FORA-05 | `docs/PRD.md` | Fora de Escopo | Vários workers em paralelo e ordem global dos avisos são problema do futuro | TRANSCRICAO | [09:13] Diego |
| PRD-FORA-06 | `docs/PRD.md` | Fora de Escopo | Webhooks enviados pelos clientes para a plataforma; o fluxo é só de saída | TRANSCRICAO | [09:02] Marcos |
| PRD-FORA-07 | `docs/PRD.md` | Fora de Escopo | Restringir o cadastro de webhook por role fica para depois | TRANSCRICAO | [09:37] Sofia |
| PRD-RF-01 | `docs/PRD.md` | Requisito Funcional | Usuário cadastra webhook com URL e lista de status; o customer vem na requisição, não do JWT | TRANSCRICAO | [09:32] Larissa |
| PRD-RF-02 | `docs/PRD.md` | Requisito Funcional | A plataforma gera a secret do webhook e a devolve na criação | TRANSCRICAO | [09:31] Marcos |
| PRD-RF-03 | `docs/PRD.md` | Requisito Funcional | Usuário edita um webhook existente | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-04 | `docs/PRD.md` | Requisito Funcional | Usuário remove um webhook; parar os pendentes e manter o histórico é comportamento proposto no FDD | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-05 | `docs/PRD.md` | Requisito Funcional | Usuário lista os webhooks de um customer | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-06 | `docs/PRD.md` | Requisito Funcional | Cada webhook recebe só os status da lista dele; status que ninguém quer não gera aviso | TRANSCRICAO | [09:34] Bruno |
| PRD-RF-07 | `docs/PRD.md` | Requisito Funcional | Aviso a cada mudança de status com status anterior, novo e dados básicos, sem itens; não avisar na criação do pedido é proposta | TRANSCRICAO | [09:43] Diego |
| PRD-RF-08 | `docs/PRD.md` | Requisito Funcional | Cada aviso leva assinatura que permite verificar origem e integridade | TRANSCRICAO | [09:19] Sofia |
| PRD-RF-09 | `docs/PRD.md` | Requisito Funcional | Usuário troca a secret pela API e a antiga vale por mais 24 h | TRANSCRICAO | [09:21] Sofia |
| PRD-RF-10 | `docs/PRD.md` | Requisito Funcional | Cada aviso leva identificador de evento igual em todas as tentativas, para descartar repetições | TRANSCRICAO | [09:25] Diego |
| PRD-RF-11 | `docs/PRD.md` | Requisito Funcional | Cada aviso identifica o cadastro de webhook que o recebeu | TRANSCRICAO | [09:44] Sofia |
| PRD-RF-12 | `docs/PRD.md` | Requisito Funcional | Aviso não entregue é reenviado em intervalos crescentes e, esgotadas as tentativas, fica guardado como falha definitiva | TRANSCRICAO | [09:15] Diego |
| PRD-RF-13 | `docs/PRD.md` | Requisito Funcional | Só ADMIN reprocessa aviso em falha definitiva, e a plataforma registra quem fez | TRANSCRICAO | [09:36] Sofia |
| PRD-RF-14 | `docs/PRD.md` | Requisito Funcional | Histórico das últimas 100 entregas com sucesso ou falha, conteúdo, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-RF-15 | `docs/PRD.md` | Requisito Funcional | (proposta) Usuário desativa e reativa um webhook sem removê-lo, pelo PATCH, usando o estado ativo da configuração | TRANSCRICAO | [09:33] Bruno |
| PRD-RF-16 | `docs/PRD.md` | Requisito Funcional | Cada aviso informa o momento do envio, para o cliente detectar reenvio malicioso | TRANSCRICAO | [09:44] Diego |
| PRD-RNF-01 | `docs/PRD.md` | Requisito Não Funcional | Aviso entregue em menos de 10 s a um cliente disponível | TRANSCRICAO | [09:02] Marcos |
| PRD-RNF-02 | `docs/PRD.md` | Requisito Não Funcional | O envio acrescenta até 2 s de espera antes da primeira tentativa | TRANSCRICAO | [09:10] Larissa |
| PRD-RNF-03 | `docs/PRD.md` | Requisito Não Funcional | Timeout de 10 s na chamada ao cliente; depois disso conta como falha | TRANSCRICAO | [09:42] Diego |
| PRD-RNF-04 | `docs/PRD.md` | Requisito Não Funcional | URL do webhook só com HTTPS; cadastro com HTTP é recusado | TRANSCRICAO | [09:23] Sofia |
| PRD-RNF-05 | `docs/PRD.md` | Requisito Não Funcional | Aviso acima de 64 KB não é enviado nem truncado; é erro | TRANSCRICAO | [09:24] Larissa |
| PRD-RNF-06 | `docs/PRD.md` | Requisito Não Funcional | Garantia at-least-once: o cliente pode receber o mesmo aviso mais de uma vez | TRANSCRICAO | [09:24] Diego |
| PRD-RNF-07 | `docs/PRD.md` | Requisito Não Funcional | Ordem só dentro do mesmo pedido e com um único processo de envio; sem ordem global | TRANSCRICAO | [09:13] Larissa |
| PRD-RNF-08 | `docs/PRD.md` | Requisito Não Funcional | Secret única por endpoint, nunca global | TRANSCRICAO | [09:21] Sofia |
| PRD-RNF-09 | `docs/PRD.md` | Requisito Não Funcional | Cliente lento ou fora do ar não afeta a mudança de status de nenhum pedido | TRANSCRICAO | [09:04] Bruno |
| PRD-RNF-10 | `docs/PRD.md` | Requisito Não Funcional | Reinício da API não interrompe o envio de avisos | TRANSCRICAO | [09:11] Diego |
| PRD-RNF-11 | `docs/PRD.md` | Requisito Não Funcional | Nenhuma infraestrutura nova: mesmo MySQL e mesma stack | TRANSCRICAO | [09:07] Diego |
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
| PRD-TESTE-01 | `docs/PRD.md` | Restrição | Testes de integração com Vitest e Supertest, no padrão da suíte atual | CODIGO | `tests/orders.test.ts` |
| PRD-TESTE-02 | `docs/PRD.md` | Proposta | (proposta) Worker validado contra um servidor HTTP local que responde sucesso, erro, demora ou queda | TRANSCRICAO | [09:42] Diego |
| PRD-TESTE-03 | `docs/PRD.md` | Proposta | (proposta) Primeiro webhook em produção com um dos três clientes que pediram a feature, antes de abrir para os demais; a reunião não definiu piloto | TRANSCRICAO | [09:00] Marcos |
| PRD-TESTE-04 | `docs/PRD.md` | Restrição | Regressão: a suíte existente de pedidos continua passando | CODIGO | `tests/orders.test.ts` |
| PRD-TESTE-05 | `docs/PRD.md` | Restrição | Regressão: a suíte existente de autenticação continua passando | CODIGO | `tests/auth.test.ts` |
| PRD-AC-01 | `docs/PRD.md` | Critério de Aceitação | Cadastro com HTTPS e lista de status devolve a secret; HTTP é recusado | TRANSCRICAO | [09:31] Marcos |
| PRD-AC-02 | `docs/PRD.md` | Critério de Aceitação | Status listado pelo webhook gera aviso; status não listado não gera | TRANSCRICAO | [09:34] Bruno |
| PRD-AC-03 | `docs/PRD.md` | Critério de Aceitação | Cliente disponível recebe o aviso em menos de 10 s, no p95 (percentil é proposta) | TRANSCRICAO | [09:02] Marcos |
| PRD-AC-04 | `docs/PRD.md` | Critério de Aceitação | Cliente valida a assinatura do aviso com a secret recebida | TRANSCRICAO | [09:20] Sofia |
| PRD-AC-05 | `docs/PRD.md` | Critério de Aceitação | Todas as tentativas de um aviso chegam com o mesmo identificador de evento | TRANSCRICAO | [09:25] Diego |
| PRD-AC-06 | `docs/PRD.md` | Critério de Aceitação | Com o cliente fora do ar, novas tentativas nos intervalos combinados e guarda como falha definitiva | TRANSCRICAO | [09:17] Larissa |
| PRD-AC-07 | `docs/PRD.md` | Critério de Aceitação | Só ADMIN reprocessa, e o reprocessamento fica registrado com o autor | TRANSCRICAO | [09:36] Sofia |
| PRD-AC-08 | `docs/PRD.md` | Critério de Aceitação | Depois da troca, o cliente verifica com a secret nova ou a antiga por 24 h (forma de assinar é proposta do FDD) | TRANSCRICAO | [09:21] Sofia |
| PRD-AC-09 | `docs/PRD.md` | Critério de Aceitação | Histórico mostra as últimas 100 entregas com resultado, conteúdo, resposta e tempo | TRANSCRICAO | [09:34] Marcos |
| PRD-AC-10 | `docs/PRD.md` | Critério de Aceitação | Mudança de status continua funcionando com o cliente lento ou fora do ar | TRANSCRICAO | [09:04] Bruno |
| PRD-AC-11 | `docs/PRD.md` | Critério de Aceitação | Revisão de segurança da Sofia feita antes do deploy | TRANSCRICAO | [09:46] Sofia |
| RFC-ALT-01 | `docs/RFC.md` | Alternativa Descartada | Disparo síncrono no service de orders: transação pesada e cliente lento trava outros pedidos | TRANSCRICAO | [09:04] Bruno |
| RFC-ALT-02 | `docs/RFC.md` | Alternativa Descartada | Fila em Redis Streams: exigiria subir mais infraestrutura | TRANSCRICAO | [09:07] Larissa |
| RFC-ALT-03 | `docs/RFC.md` | Alternativa Descartada | Trigger do banco para acordar o worker: MySQL não notifica processo externo | TRANSCRICAO | [09:09] Diego |
| RFC-ALT-04 | `docs/RFC.md` | Alternativa Descartada | Retry indefinido: evento fica pendurado para sempre se o cliente sumir | TRANSCRICAO | [09:15] Diego |
| RFC-ALT-05 | `docs/RFC.md` | Alternativa Descartada | Três tentativas: pouca janela para uma manutenção de duas horas | TRANSCRICAO | [09:16] Diego |
| RFC-ALT-06 | `docs/RFC.md` | Alternativa Descartada | DLQ como status "failed" na outbox: tabela separada deixa a outbox mais limpa | TRANSCRICAO | [09:18] Diego |
| RFC-ALT-07 | `docs/RFC.md` | Alternativa Descartada | Secret global da plataforma: se vaza uma, vaza tudo | TRANSCRICAO | [09:21] Sofia |
| RFC-ALT-08 | `docs/RFC.md` | Alternativa Descartada | Garantia exactly-once: exige coordenação dos dois lados | TRANSCRICAO | [09:25] Diego |
| RFC-ALT-09 | `docs/RFC.md` | Alternativa Descartada | Montar o payload na hora do envio: o evento deixaria de refletir o momento da mudança | TRANSCRICAO | [09:52] Larissa |
| RFC-QA-01 | `docs/RFC.md` | Questão em Aberto | Quantos envios: "5 tentativas" contra cinco intervalos que somam quase 15 h, o que dá 1 + 5 | TRANSCRICAO | [09:17] Diego |
| RFC-QA-02 | `docs/RFC.md` | Questão em Aberto | Rate limiting de saída ficou "observar e decidir depois" | TRANSCRICAO | [09:39] Larissa |
| RFC-QA-03 | `docs/RFC.md` | Questão em Aberto | Escalar para vários workers: particionar por order_id ou lock pessimista | TRANSCRICAO | [09:13] Diego |
| RFC-QA-04 | `docs/RFC.md` | Questão em Aberto | Endurecer o acesso ao CRUD de webhooks, hoje aberto a qualquer role autenticada | TRANSCRICAO | [09:37] Sofia |
| RFC-RESTR-01 | `docs/RFC.md` | Restrição | As roles são só ADMIN e OPERATOR, sem papel de cliente para restringir o acesso por customer | CODIGO | `src/middlewares/auth.middleware.ts` |
| RFC-QA-05 | `docs/RFC.md` | Questão em Aberto | Aviso ao cliente por e-mail quando o webhook falha, talvez na próxima fase | TRANSCRICAO | [09:37] Larissa |
| RFC-QA-06 | `docs/RFC.md` | Questão em Aberto | Arquivamento da outbox "depois de 30 dias ou assim" | TRANSCRICAO | [09:08] Diego |
| RFC-QA-07 | `docs/RFC.md` | Questão em Aberto | Onde vai o customer_id no cadastro: "no body ou no path" | TRANSCRICAO | [09:32] Larissa |
| RFC-LAC-01 | `docs/RFC.md` | Lacuna | Uma linha de outbox por evento ou uma por webhook, já que um customer pode ter vários endpoints | TRANSCRICAO | [09:44] Sofia |
| RFC-LAC-02 | `docs/RFC.md` | Lacuna | Com qual secret a plataforma assina durante as 24 h de rotação | TRANSCRICAO | [09:21] Sofia |
| RFC-LAC-03 | `docs/RFC.md` | Lacuna | X-Timestamp fora da assinatura permite trocar o valor e burlar a detecção de replay | TRANSCRICAO | [09:44] Diego |
| RFC-LAC-04 | `docs/RFC.md` | Lacuna | Forma de guardar a secret, que precisa ser recuperável para assinar cada envio | TRANSCRICAO | [09:21] Bruno |
| RFC-LAC-05 | `docs/RFC.md` | Lacuna | Evento na criação do pedido: `OrderService.create()` nasce em PENDING sem passar por `changeStatus` | CODIGO | `src/modules/orders/order.service.ts` |
| RFC-RISCO-01 | `docs/RFC.md` | Risco | Entrega duplicada: o cliente processa o mesmo evento duas vezes | TRANSCRICAO | [09:24] Diego |
| RFC-RISCO-02 | `docs/RFC.md` | Risco | Ordem perdida com retry: evento em backoff ultrapassado pelo seguinte do mesmo pedido (análise) | TRANSCRICAO | [09:12] Diego |
| RFC-RISCO-03 | `docs/RFC.md` | Risco | Latência acima de 10 s: polling somado a um envio lento até o timeout | TRANSCRICAO | [09:42] Diego |
| RFC-RISCO-04 | `docs/RFC.md` | Risco | Secret em log: o redact do logger não cobre a secret | CODIGO | `src/shared/logger/index.ts` |
| RFC-RISCO-05 | `docs/RFC.md` | Risco | Prazo citado como fim do trimestre e depois como fim de novembro | TRANSCRICAO | [09:45] Marcos |
| ADR-001 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Decisão | Outbox no MySQL, gravada na mesma transação de `changeStatus`; falha na gravação faz rollback | TRANSCRICAO | [09:08] Larissa |
| ADR-001-ALT-01 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa Descartada | Disparo síncrono no service de orders | TRANSCRICAO | [09:04] Bruno |
| ADR-001-ALT-02 | `docs/adrs/ADR-001-outbox-no-mysql.md` | Alternativa Descartada | Redis Streams: Redis Cluster é overengineering para um time pequeno | TRANSCRICAO | [09:07] Diego |
| ADR-001-TO | `docs/adrs/ADR-001-outbox-no-mysql.md` | Trade-off | Transação maior e tabela sem arquivamento nesta fase, em troca de consistência sem infraestrutura nova | TRANSCRICAO | [09:08] Diego |
| ADR-002 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Decisão | Worker em processo separado, polling a cada 2 s, um único worker com ordem por pedido | TRANSCRICAO | [09:10] Larissa |
| ADR-002-ALT-01 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Alternativa Descartada | Trigger do banco para reagir sem esperar o ciclo | TRANSCRICAO | [09:09] Diego |
| ADR-002-ALT-02 | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Alternativa Descartada | Worker dentro do processo da API: cai junto quando a API reinicia | TRANSCRICAO | [09:11] Diego |
| ADR-002-TO | `docs/adrs/ADR-002-worker-em-processo-separado-com-polling.md` | Trade-off | Até 2 s de espera, um processo a mais e ordem só por pedido, sem infraestrutura nova | TRANSCRICAO | [09:13] Larissa |
| ADR-003 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Decisão | Retry com backoff 1m/5m/30m/2h/12h e DLQ em tabela separada com reprocessamento manual | TRANSCRICAO | [09:17] Larissa |
| ADR-003-ALT-01 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Alternativa Descartada | Retry indefinido com backoff | TRANSCRICAO | [09:15] Diego |
| ADR-003-ALT-02 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Alternativa Descartada | Três tentativas, "mais agressivo" | TRANSCRICAO | [09:16] Bruno |
| ADR-003-ALT-03 | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Alternativa Descartada | DLQ como status "failed" na própria outbox | TRANSCRICAO | [09:18] Diego |
| ADR-003-TO | `docs/adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md` | Trade-off | Entregas que podem atrasar cerca de 15 h e uma tabela a mais, em troca de sobreviver a quedas longas | TRANSCRICAO | [09:17] Marcos |
| ADR-004 | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Decisão | HMAC-SHA256 sobre o corpo, secret por endpoint gerada pela plataforma, rotação com 24 h de carência | TRANSCRICAO | [09:22] Sofia |
| ADR-004-ALT-01 | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Alternativa Descartada | Secret global da plataforma | TRANSCRICAO | [09:21] Sofia |
| ADR-004-ALT-02 | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Alternativa Descartada | (proposta) Secret informada pelo cliente: alternativa plausível que ninguém defendeu; a única pista é o código citado por Bruno, e foi descartada pela regra do Marcos em [09:31] | TRANSCRICAO | [09:28] Bruno |
| ADR-004-TO | `docs/adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md` | Trade-off | Custo de gerar, guardar e trocar uma secret por endpoint, em troca de vazamento de alcance limitado | TRANSCRICAO | [09:21] Sofia |
| ADR-005 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Decisão | Entrega at-least-once com UUID no X-Event-Id para o cliente deduplicar | TRANSCRICAO | [09:26] Larissa |
| ADR-005-ALT-01 | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Alternativa Descartada | Exactly-once | TRANSCRICAO | [09:25] Diego |
| ADR-005-TO | `docs/adrs/ADR-005-entrega-at-least-once-com-x-event-id.md` | Trade-off | O cliente recebe repetições e precisa deduplicar | TRANSCRICAO | [09:25] Sofia |
| ADR-006 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Decisão | Reuso máximo: módulo padrão, AppError, Pino, error middleware, schemas Zod e códigos de erro | TRANSCRICAO | [09:30] Larissa |
| ADR-006-ALT-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | (proposta) Infraestrutura própria de log e erro para o módulo: alternativa plausível que ninguém propôs, descartada pelo "não vamos botar nada novo" | TRANSCRICAO | [09:29] Bruno |
| ADR-006-ALT-02 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Alternativa Descartada | (proposta) Usar os códigos genéricos, sem prefixo WEBHOOK_: alternativa plausível que ninguém propôs, descartada pelo prefixo fixado por Larissa | TRANSCRICAO | [09:29] Larissa |
| ADR-006-TO | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Trade-off | As peças existentes precisam de ajuste: classes com código fixo, Zod com código genérico, redact sem a secret | CODIGO | `src/shared/errors/http-errors.ts` |
| ADR-006-RESTR-01 | `docs/adrs/ADR-006-reuso-dos-padroes-do-projeto.md` | Restrição | O middleware de erro é do Express e não cobre o processo do worker | CODIGO | `src/middlewares/error.middleware.ts` |
| ADR-007 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Decisão | Payload montado e gravado na outbox no momento da inserção | TRANSCRICAO | [09:52] Larissa |
| ADR-007-ALT-01 | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Alternativa Descartada | Guardar só o order_id e montar o payload no envio | TRANSCRICAO | [09:51] Bruno |
| ADR-007-TO | `docs/adrs/ADR-007-snapshot-do-payload-na-insercao.md` | Trade-off | Linhas maiores e evento que pode descrever estado superado; o cliente consulta o pedido para o atual | TRANSCRICAO | [09:43] Diego |
| FDD-OT-1 | `docs/FDD.md` | Objetivo | Toda mudança de status com interessado gera evento na outbox; mudança revertida não gera | TRANSCRICAO | [09:06] Diego |
| FDD-OT-2 | `docs/FDD.md` | Objetivo | O worker pega o evento pendente em até 2 s após o commit | TRANSCRICAO | [09:10] Larissa |
| FDD-OT-3 | `docs/FDD.md` | Objetivo | Cliente saudável recebe o evento em menos de 10 s | TRANSCRICAO | [09:02] Marcos |
| FDD-OT-4 | `docs/FDD.md` | Objetivo | Falha ou timeout agenda tentativas em 1 min, 5 min, 30 min, 2 h e 12 h e depois move para a DLQ | TRANSCRICAO | [09:17] Larissa |
| FDD-OT-5 | `docs/FDD.md` | Objetivo | Todo envio leva HMAC-SHA256 verificável com a secret do endpoint | TRANSCRICAO | [09:22] Sofia |
| FDD-OT-6 | `docs/FDD.md` | Objetivo | Eventos de um pedido para um webhook chegam na ordem de criação, com um único worker | TRANSCRICAO | [09:13] Larissa |
| FDD-OT-7 | `docs/FDD.md` | Objetivo | Nenhum payload acima de 64 KB é enviado | TRANSCRICAO | [09:24] Larissa |
| FDD-ESC-01 | `docs/FDD.md` | Proposta | (proposta) A criação do pedido não gera evento, porque `create()` não passa por `changeStatus` | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-DADOS-01 | `docs/FDD.md` | Restrição | Tabelas novas com id UUID, no padrão do resto do projeto | TRANSCRICAO | [09:51] Larissa |
| FDD-DADOS-02 | `docs/FDD.md` | Proposta | (proposta) Tabela `webhook_endpoints` com url, secret, customer_id, ativo, statuses e campos de rotação | TRANSCRICAO | [09:21] Bruno |
| FDD-DADOS-03 | `docs/FDD.md` | Proposta | Tabela `webhook_outbox` com estados pendente, processando, falhou e entregue e índices em status e created_at (os nomes da tabela e das colunas são proposta) | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-04 | `docs/FDD.md` | Proposta | Tabela `webhook_dead_letter` com payload, motivo e momento da falha (os nomes são proposta) | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-05 | `docs/FDD.md` | Proposta | (proposta) Tabela `webhook_deliveries` com sucesso, resposta, início e duração de cada tentativa | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-06 | `docs/FDD.md` | Proposta | (proposta) Uma linha de outbox por par evento e webhook, com o mesmo event_id | TRANSCRICAO | [09:44] Sofia |
| FDD-DADOS-07 | `docs/FDD.md` | Proposta | (proposta) `statuses` como coluna Json, porque o Prisma não tem lista escalar no MySQL | CODIGO | `prisma/schema.prisma` |
| FDD-DADOS-08 | `docs/FDD.md` | Proposta | (proposta) FAILED é o estado terminal da outbox, gravado na mesma transação da linha na DLQ | TRANSCRICAO | [09:08] Diego |
| FDD-DADOS-09 | `docs/FDD.md` | Proposta | (proposta) Remoção lógica com `deleted_at`, mantendo outbox, histórico e DLQ consistentes | TRANSCRICAO | [09:33] Bruno |
| FDD-DADOS-10 | `docs/FDD.md` | Proposta | (proposta) FK restritiva de `webhook_endpoints.customer_id` para customers, igual à de orders | CODIGO | `prisma/schema.prisma` |
| FDD-DADOS-11 | `docs/FDD.md` | Proposta | (proposta) O payload fica só na outbox e na DLQ; o histórico aponta para a outbox | TRANSCRICAO | [09:18] Diego |
| FDD-DADOS-12 | `docs/FDD.md` | Proposta | (proposta) `response_body` guarda só os primeiros 4 KB da resposta | TRANSCRICAO | [09:34] Marcos |
| FDD-DADOS-13 | `docs/FDD.md` | Proposta | (proposta) Secret legível pelo worker, fora das respostas exceto criação e rotação, nunca em log; motivada pelo vazamento de secret em log de cliente | TRANSCRICAO | [09:22] Diego |
| FDD-DADOS-14 | `docs/FDD.md` | Fora de Escopo | Não há endpoint de listagem da DLQ; o ADMIN acha o id pelo log ou pelo banco, porque a reunião só pediu o replay | TRANSCRICAO | [09:35] Diego |
| FDD-FLUXO-01 | `docs/FDD.md` | Integração | `changeStatus` chama `publishWebhookEvent(tx, order, fromStatus, toStatus)` dentro da transação | TRANSCRICAO | [09:41] Bruno |
| FDD-FLUXO-02 | `docs/FDD.md` | Requisito Funcional | Filtro de status na inserção; sem webhook interessado, nada é gravado | TRANSCRICAO | [09:34] Bruno |
| FDD-FLUXO-03 | `docs/FDD.md` | Proposta | (proposta) Webhook pausado recebe o evento na outbox, e a linha fica PENDING até a reativação | TRANSCRICAO | [09:21] Bruno |
| FDD-FLUXO-04 | `docs/FDD.md` | Decisão | Payload montado uma vez na inserção, com `to_status` igual ao status novo | TRANSCRICAO | [09:52] Larissa |
| FDD-FLUXO-05 | `docs/FDD.md` | Contrato | event_id é um UUID gerado na inserção na outbox | TRANSCRICAO | [09:25] Diego |
| FDD-FLUXO-06 | `docs/FDD.md` | Proposta | (proposta) Payload acima de 64 KB entra como FAILED e vai para a DLQ com `WEBHOOK_PAYLOAD_TOO_LARGE`, sem desfazer o status | TRANSCRICAO | [09:24] Larissa |
| FDD-FLUXO-07 | `docs/FDD.md` | Proposta | (proposta) `request_id` vem do `req.id` e é cortado em 64 caracteres, porque o middleware aceita X-Request-Id sem limite | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-FLUXO-08 | `docs/FDD.md` | Decisão | Ciclo do worker a cada 2 s | TRANSCRICAO | [09:09] Diego |
| FDD-FLUXO-09 | `docs/FDD.md` | Proposta | (proposta) Lote de até 10 linhas por ciclo, a partir do "batch pequeno" | TRANSCRICAO | [09:08] Diego |
| FDD-FLUXO-10 | `docs/FDD.md` | Decisão | Linhas pendentes processadas em ordem de created_at | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-11 | `docs/FDD.md` | Proposta | (proposta) Linha só é elegível sem outra mais antiga do mesmo webhook e pedido pendente, via `NOT EXISTS` | TRANSCRICAO | [09:13] Larissa |
| FDD-FLUXO-12 | `docs/FDD.md` | Proposta | (proposta) Webhook removido: linhas vão para FAILED com `WEBHOOK_DELETED`, sem DLQ e sem envio | TRANSCRICAO | [09:33] Bruno |
| FDD-FLUXO-13 | `docs/FDD.md` | Proposta | (proposta) Claim condicional linha a linha antes do envio, mantendo o desenho de um único worker | TRANSCRICAO | [09:12] Diego |
| FDD-FLUXO-14 | `docs/FDD.md` | Proposta | (proposta) Envio em paralelo dentro do lote, para dez clientes lentos não estourarem a meta de 10 s | TRANSCRICAO | [09:02] Marcos |
| FDD-FLUXO-15 | `docs/FDD.md` | Proposta | (proposta) POST para a URL do webhook com `fetch` nativo e timeout de 10 s; o `fetch` é escolha do FDD, o timeout veio da reunião | TRANSCRICAO | [09:42] Diego |
| FDD-FLUXO-15B | `docs/FDD.md` | Restrição | `fetch` e `AbortSignal.timeout` disponíveis sem dependência nova porque o projeto exige Node >=20 | CODIGO | `package.json` |
| FDD-FLUXO-16 | `docs/FDD.md` | Proposta | (proposta) Tabela de envios com 1 envio original mais 5 retentativas (1 min, 5 min, 30 min, 2 h, 12 h); a contagem é premissa provisória, questão 1 do RFC | TRANSCRICAO | [09:17] Diego |
| FDD-FLUXO-17 | `docs/FDD.md` | Proposta | (proposta) Intervalos de retry como constantes do módulo, sem configuração | TRANSCRICAO | [09:17] Larissa |
| FDD-FLUXO-18 | `docs/FDD.md` | Decisão | Na última falha, outbox vai para FAILED e entra linha na DLQ com payload, motivo e failed_at | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-19 | `docs/FDD.md` | Decisão | Reprocessamento manual por endpoint admin recoloca o evento na outbox como pendente | TRANSCRICAO | [09:35] Diego |
| FDD-FLUXO-20 | `docs/FDD.md` | Proposta | (proposta) Reprocessamento por update condicional, attempts zerado, mesmo event_id e nova linha na DLQ se esgotar de novo | TRANSCRICAO | [09:18] Diego |
| FDD-FLUXO-21 | `docs/FDD.md` | Proposta | (proposta) Webhook removido responde 404 e payload grande demais responde 422 no reprocessamento | TRANSCRICAO | [09:24] Larissa |
| FDD-FLUXO-22 | `docs/FDD.md` | Requisito Funcional | Rotação move a secret atual para `previous_secret` com validade de 24 h | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-23 | `docs/FDD.md` | Proposta | (proposta) Na carência, o worker envia as duas assinaturas no mesmo header, separadas por vírgula | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-24 | `docs/FDD.md` | Proposta | (proposta) Só uma secret anterior; uma segunda rotação em 24 h descarta a mais antiga | TRANSCRICAO | [09:21] Sofia |
| FDD-FLUXO-25 | `docs/FDD.md` | Proposta | (proposta) Secret gerada com 32 bytes de `crypto.randomBytes`, em hex, sujeita à revisão da Sofia | TRANSCRICAO | [09:46] Sofia |
| FDD-CONTRATO-01 | `docs/FDD.md` | Contrato | Rotas sob `/api/v1` com `Authorization: Bearer <jwt>` via `authenticate` | CODIGO | `src/app.ts` |
| FDD-CONTRATO-02 | `docs/FDD.md` | Contrato | Erros no formato `{ "error": { "code", "message", "details?" } }` do `errorMiddleware` | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-CONTRATO-03 | `docs/FDD.md` | Proposta | (proposta) `customerId` no body na criação e na query na listagem, no padrão dos schemas de orders | CODIGO | `src/modules/orders/order.schemas.ts` |
| FDD-CONTRATO-04 | `docs/FDD.md` | Restrição | CRUD de configuração aceita qualquer role autenticada, por enquanto | TRANSCRICAO | [09:37] Sofia |
| FDD-CONTRATO-05 | `docs/FDD.md` | Proposta | (proposta) Rotação de secret e histórico de entregas seguem a mesma regra de role do CRUD | TRANSCRICAO | [09:36] Marcos |
| FDD-CONTRATO-06 | `docs/FDD.md` | Contrato | `POST /api/v1/webhooks` cadastra o webhook e devolve a secret na criação (201) | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-07 | `docs/FDD.md` | Contrato | `GET /api/v1/webhooks?customerId=` lista os webhooks de um customer, sem secret | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-08 | `docs/FDD.md` | Contrato | `PATCH /api/v1/webhooks/:id` edita statuses, url ou active | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-09 | `docs/FDD.md` | Contrato | Mudança de `statuses` vale só para eventos novos, porque o filtro roda na inserção | TRANSCRICAO | [09:34] Bruno |
| FDD-CONTRATO-10 | `docs/FDD.md` | Contrato | `DELETE /api/v1/webhooks/:id` remove o webhook (204) | TRANSCRICAO | [09:33] Bruno |
| FDD-CONTRATO-11 | `docs/FDD.md` | Proposta | (proposta) `POST /api/v1/webhooks/:id/secret/rotate` como caminho do endpoint de rotação | TRANSCRICAO | [09:21] Sofia |
| FDD-CONTRATO-12 | `docs/FDD.md` | Contrato | `GET /api/v1/webhooks/:id/deliveries` devolve as 100 tentativas mais recentes, sem paginação | TRANSCRICAO | [09:34] Marcos |
| FDD-CONTRATO-13 | `docs/FDD.md` | Contrato | `POST /api/v1/admin/webhooks/dead-letter/:id/replay` exige ADMIN e responde 202 | TRANSCRICAO | [09:35] Diego |
| FDD-CONTRATO-14 | `docs/FDD.md` | Proposta | (proposta) A secret só volta na criação e na rotação, nunca em listagem ou edição | TRANSCRICAO | [09:31] Marcos |
| FDD-CONTRATO-15 | `docs/FDD.md` | Contrato | Payload em snake_case: event_id, event_type, timestamp, order_id, order_number, from_status, to_status, customer_id, total_cents, sem items | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-16 | `docs/FDD.md` | Proposta | (proposta) `timestamp` do payload é o momento da mudança de status; o do envio vai no header | TRANSCRICAO | [09:43] Diego |
| FDD-CONTRATO-17 | `docs/FDD.md` | Contrato | Header `Content-Type: application/json` | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-18 | `docs/FDD.md` | Contrato | Header `X-Event-Id` com o event_id, igual em todas as tentativas e no reprocessamento | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-19 | `docs/FDD.md` | Contrato | Header `X-Webhook-Id` com o id do cadastro que recebeu o envio | TRANSCRICAO | [09:44] Sofia |
| FDD-CONTRATO-20 | `docs/FDD.md` | Contrato | Header `X-Timestamp` com o momento do envio em ISO 8601 | TRANSCRICAO | [09:44] Diego |
| FDD-CONTRATO-21 | `docs/FDD.md` | Contrato | Header `X-Signature` com HMAC-SHA256 do corpo; o formato `sha256=<hex>` é proposta | TRANSCRICAO | [09:20] Sofia |
| FDD-CONTRATO-22 | `docs/FDD.md` | Proposta | (proposta) Resposta 2xx em até 10 s é sucesso; o resto conta como falha, sem distinguir 4xx de 5xx | TRANSCRICAO | [09:42] Diego |
| FDD-ERRO-01 | `docs/FDD.md` | Restrição | Erros do módulo estendem `AppError` com prefixo `WEBHOOK_` | TRANSCRICAO | [09:29] Larissa |
| FDD-ERRO-02 | `docs/FDD.md` | Restrição | `NotFoundError` tem código fixo `NOT_FOUND`, então os 404 do módulo estendem `AppError` direto | CODIGO | `src/shared/errors/http-errors.ts` |
| FDD-ERRO-03 | `docs/FDD.md` | Erro | `WEBHOOK_NOT_FOUND` (404): webhook inexistente ou removido | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-04 | `docs/FDD.md` | Erro | `WEBHOOK_INVALID_URL` (400): URL sem https | TRANSCRICAO | [09:28] Bruno |
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
| FDD-ERRO-17 | `docs/FDD.md` | Decisão | `WEBHOOK_SECRET_REQUIRED`, citado como exemplo por Bruno, não entra na matriz porque a secret é gerada pela plataforma ([09:31] Marcos) | TRANSCRICAO | [09:28] Bruno |
| FDD-ERRO-18 | `docs/FDD.md` | Erro | Falha ao gravar na outbox derruba a transação e responde `500 INTERNAL_SERVER_ERROR` | TRANSCRICAO | [09:40] Bruno |
| FDD-RES-01 | `docs/FDD.md` | Restrição | Cliente lento: timeout de 10 s, conta como falha e entra no retry | TRANSCRICAO | [09:42] Diego |
| FDD-RES-02 | `docs/FDD.md` | Restrição | Cliente fora do ar: retry com 1 min, 5 min, 30 min, 2 h e 12 h e depois DLQ | TRANSCRICAO | [09:17] Larissa |
| FDD-RES-03 | `docs/FDD.md` | Proposta | (proposta) Redirect conta como falha e não é seguido, para não trocar https por http | TRANSCRICAO | [09:23] Sofia |
| FDD-RES-04 | `docs/FDD.md` | Restrição | Falha na gravação do evento faz rollback da mudança de status | TRANSCRICAO | [09:40] Bruno |
| FDD-RES-05 | `docs/FDD.md` | Restrição | API reinicia e o worker, outro processo, segue entregando | TRANSCRICAO | [09:11] Diego |
| FDD-RES-06 | `docs/FDD.md` | Proposta | (proposta) Linha presa em PROCESSING há mais de 60 s conta como tentativa falha; o cliente deduplica a repetição | TRANSCRICAO | [09:24] Diego |
| FDD-RES-07 | `docs/FDD.md` | Proposta | (proposta) Banco indisponível: o ciclo loga `webhook_worker_cycle_failed` e tenta de novo sem derrubar o processo | CODIGO | `src/shared/logger/index.ts` |
| FDD-RES-08 | `docs/FDD.md` | Proposta | (proposta) Dois workers por engano: o claim condicional evita envio duplicado, sem garantir ordem | TRANSCRICAO | [09:12] Diego |
| FDD-RES-09 | `docs/FDD.md` | Restrição | Payload acima de 64 KB não é enviado nem truncado (reunião); ir para a DLQ é proposta do FDD | TRANSCRICAO | [09:24] Larissa |
| FDD-RES-10 | `docs/FDD.md` | Fora de Escopo | Sem fallback por outro canal; o aviso por e-mail ficou para a próxima fase | TRANSCRICAO | [09:37] Larissa |
| FDD-RES-11 | `docs/FDD.md` | Proposta | (proposta) Encerramento do worker espera os envios em andamento, ao contrário do `process.exit(0)` da API | CODIGO | `src/server.ts` |
| FDD-OBS-01 | `docs/FDD.md` | Proposta | (proposta) Logs estruturados com o Pino existente: enqueued, succeeded, failed, dead letter, replay, rotação e falha de ciclo | TRANSCRICAO | [09:29] Bruno |
| FDD-OBS-02 | `docs/FDD.md` | Requisito Não Funcional | Log `webhook_replay_requested` com o userId de quem pediu, para auditoria | TRANSCRICAO | [09:36] Sofia |
| FDD-OBS-02B | `docs/FDD.md` | Proposta | (proposta) Log `webhook_secret_rotated` com o autor da rotação | TRANSCRICAO | [09:21] Sofia |
| FDD-OBS-02C | `docs/FDD.md` | Proposta | (proposta) Log `webhook_worker_stats` a cada 60 s com backlog, DLQ e envios por webhook, para medir sem dependência nova | TRANSCRICAO | [09:39] Larissa |
| FDD-OBS-03 | `docs/FDD.md` | Proposta | (proposta) `webhook_event_enqueued` só é logado depois que a transação resolve, para não registrar evento revertido | TRANSCRICAO | [09:06] Diego |
| FDD-OBS-04 | `docs/FDD.md` | Proposta | (proposta) `secret`, `previousSecret` e X-Signature entram no redact do logger | CODIGO | `src/shared/logger/index.ts` |
| FDD-OBS-05 | `docs/FDD.md` | Proposta | (proposta) O payload não é logado, só ids | TRANSCRICAO | [09:19] Sofia |
| FDD-OBS-06 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_outbox_pending` para acompanhar o backlog da outbox | TRANSCRICAO | [09:07] Bruno |
| FDD-OBS-07 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_delivery_latency_seconds` contra a meta de 10 s | TRANSCRICAO | [09:02] Marcos |
| FDD-OBS-08 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_deliveries_total{result}` por sucesso, erro HTTP, timeout e rede | TRANSCRICAO | [09:42] Diego |
| FDD-OBS-09 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_dead_letter_total` de eventos que esgotaram as tentativas | TRANSCRICAO | [09:18] Diego |
| FDD-OBS-10 | `docs/FDD.md` | Proposta | (proposta) Métrica `webhook_sends_per_webhook_per_minute` para decidir o rate limiting com dado | TRANSCRICAO | [09:39] Larissa |
| FDD-OBS-11 | `docs/FDD.md` | Proposta | (proposta) Coleta pelos logs e por `webhook_worker_stats` a cada 60 s, sem biblioteca de métricas nova | CODIGO | `package.json` |
| FDD-OBS-12 | `docs/FDD.md` | Proposta | (proposta) Tracing por correlação: X-Request-Id na outbox, event_id nas tentativas e na DLQ | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-INT-01 | `docs/FDD.md` | Integração | `order.service.ts` chama `publishWebhookEvent` depois do insert no histórico, na mesma transação | TRANSCRICAO | [09:41] Bruno |
| FDD-INT-02 | `docs/FDD.md` | Integração | O snapshot usa `to`, porque o `order` lido em `changeStatus` é anterior ao update | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-INT-03 | `docs/FDD.md` | Integração | (proposta) `OrderController.changeStatus` passa também `req.id` como `requestId` | CODIGO | `src/modules/orders/order.controller.ts` |
| FDD-INT-04 | `docs/FDD.md` | Integração | `order.status.ts` sem mudança; o enum `OrderStatus` define os status aceitos no filtro | CODIGO | `src/modules/orders/order.status.ts` |
| FDD-INT-05 | `docs/FDD.md` | Integração | (proposta) Quatro modelos novos e relação `Customer` para `WebhookEndpoint` no schema Prisma | CODIGO | `prisma/schema.prisma` |
| FDD-INT-06 | `docs/FDD.md` | Integração | Uma migração nova, só aditiva, ao lado da migração inicial | CODIGO | `prisma/migrations/20260519182739_init/migration.sql` |
| FDD-INT-07 | `docs/FDD.md` | Integração | `buildControllers` instancia repository, service e controller de webhooks | CODIGO | `src/app.ts` |
| FDD-INT-08 | `docs/FDD.md` | Integração | `buildApiRouter` registra `/webhooks` e `/admin` | CODIGO | `src/routes/index.ts` |
| FDD-INT-09 | `docs/FDD.md` | Integração | `auth.middleware.ts` sem mudança; o reprocessamento usa `requireRole('ADMIN')` | TRANSCRICAO | [09:36] Larissa |
| FDD-INT-10 | `docs/FDD.md` | Integração | `validate.middleware.ts` sem mudança; o módulo usa o mesmo `validate()` | CODIGO | `src/middlewares/validate.middleware.ts` |
| FDD-INT-11 | `docs/FDD.md` | Integração | `error.middleware.ts` sem mudança, porque as classes novas estendem `AppError` | TRANSCRICAO | [09:29] Bruno |
| FDD-INT-12 | `docs/FDD.md` | Integração | `app-error.ts` e `http-errors.ts` sem mudança; as classes novas ficam no módulo | CODIGO | `src/shared/errors/app-error.ts` |
| FDD-INT-13 | `docs/FDD.md` | Integração | Logger: incluir os caminhos da secret no redact e aceitar nome de serviço para o worker | CODIGO | `src/shared/logger/index.ts` |
| FDD-INT-14 | `docs/FDD.md` | Integração | `paginated()` reusado na listagem de webhooks | CODIGO | `src/shared/http/response.ts` |
| FDD-INT-15 | `docs/FDD.md` | Integração | `request-logger.middleware.ts` sem mudança; o `req.id` vai para a outbox | CODIGO | `src/middlewares/request-logger.middleware.ts` |
| FDD-INT-16 | `docs/FDD.md` | Integração | O worker chama `createPrismaClient()` no próprio processo, mesmo banco e mesma DATABASE_URL | TRANSCRICAO | [09:30] Bruno |
| FDD-INT-17 | `docs/FDD.md` | Integração | O worker carrega o mesmo `.env`, então `JWT_SECRET` precisa existir no ambiente dele | CODIGO | `src/config/env.ts` |
| FDD-INT-18 | `docs/FDD.md` | Integração | `src/server.ts` serve de modelo para a nova entry-point `src/worker.ts` | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-19 | `docs/FDD.md` | Integração | Novos scripts `worker` e `worker:start` no `package.json` | TRANSCRICAO | [09:11] Larissa |
| FDD-INT-20 | `docs/FDD.md` | Integração | `customer.repository.ts` valida o `customerId` no cadastro | CODIGO | `src/modules/customers/customer.repository.ts` |
| FDD-INT-21 | `docs/FDD.md` | Integração | Fábrica `createTestWebhook` nos helpers de teste | CODIGO | `tests/helpers/factories.ts` |
| FDD-INT-21B | `docs/FDD.md` | Integração | Limpeza das quatro tabelas novas no `beforeEach`, antes de `order` | CODIGO | `tests/setup.ts` |
| FDD-INT-22 | `docs/FDD.md` | Integração | Módulo `src/modules/webhooks` (a criar) com routes, controller, service, repository e schemas no padrão dos outros módulos; errors, publisher e signer saem de [09:28] e [09:41] Bruno | TRANSCRICAO | [09:27] Bruno |
| FDD-INT-23 | `docs/FDD.md` | Proposta | (proposta) Lógica do worker em `webhook.processor.ts`, entre os dois nomes citados na reunião | TRANSCRICAO | [09:28] Bruno |
| FDD-DEP-01 | `docs/FDD.md` | Dependência | Node >=20 para `fetch` e `AbortSignal.timeout`; Prisma e `uuid` já no projeto; nenhuma dependência nova | CODIGO | `package.json` |
| FDD-DEP-02 | `docs/FDD.md` | Dependência | MySQL 8 como banco | CODIGO | `docker-compose.yml` |
| FDD-DEP-03 | `docs/FDD.md` | Dependência | Migração aditiva; customer com webhook não pode ser removido e hoje esse caso responde 500 por falta de tratamento do P2003 | CODIGO | `src/middlewares/error.middleware.ts` |
| FDD-DEP-04 | `docs/FDD.md` | Dependência | `PATCH /api/v1/orders/:id/status` mantém contrato e resposta; só a latência muda | CODIGO | `src/modules/orders/order.routes.ts` |
| FDD-DEP-05 | `docs/FDD.md` | Dependência | Parar o worker interrompe os envios sem afetar a API | TRANSCRICAO | [09:11] Diego |
| FDD-DEP-06 | `docs/FDD.md` | Dependência | Revisão de segurança da Sofia com pelo menos dois dias úteis antes do deploy | TRANSCRICAO | [09:46] Sofia |
| FDD-DEP-07 | `docs/FDD.md` | Dependência | Documentação no portal do desenvolvedor pelo Marcos | TRANSCRICAO | [09:26] Marcos |
| FDD-RISCO-01 | `docs/FDD.md` | Risco | Transação de `changeStatus` mais lenta com o select de webhooks e os inserts | CODIGO | `src/modules/orders/order.service.ts` |
| FDD-RISCO-02 | `docs/FDD.md` | Risco | Webhook fora do ar segura os eventos seguintes do mesmo pedido, pela regra de ordem | TRANSCRICAO | [09:13] Larissa |
| FDD-RISCO-03 | `docs/FDD.md` | Risco | A contagem de envios mudar na confirmação, 5 contra 6 | TRANSCRICAO | [09:48] Larissa |
| FDD-RISCO-04 | `docs/FDD.md` | Risco | Revisão de segurança mudar a assinatura, com timestamp no HMAC ou secret na carência | TRANSCRICAO | [09:46] Sofia |
| FDD-RISCO-05 | `docs/FDD.md` | Risco | Outbox e histórico crescem sem arquivamento | TRANSCRICAO | [09:08] Diego |
| FDD-RISCO-06 | `docs/FDD.md` | Risco | Rajada de mudanças gera rajada de chamadas para um cliente | TRANSCRICAO | [09:38] Diego |
| FDD-RISCO-07 | `docs/FDD.md` | Risco | Qualquer usuário autenticado gerencia webhooks de qualquer customer | TRANSCRICAO | [09:37] Sofia |
| FDD-RISCO-08 | `docs/FDD.md` | Risco | Qualquer usuário autenticado rotaciona a secret de qualquer webhook e passa a assinar envios falsos | TRANSCRICAO | [09:37] Sofia |
| FDD-RISCO-09 | `docs/FDD.md` | Risco | URL cadastrada aponta para endereço interno da rede, tema que a reunião não tratou | TRANSCRICAO | [09:23] Sofia |
| FDD-RISCO-10 | `docs/FDD.md` | Risco | Evento reprocessado da DLQ chega depois dos seguintes do mesmo pedido | TRANSCRICAO | [09:18] Diego |

Total: 274 linhas. TRANSCRICAO: 226 (82,5%). CODIGO: 48.
