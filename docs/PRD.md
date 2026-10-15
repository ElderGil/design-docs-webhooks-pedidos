# PRD: Sistema de Webhooks de Notificação de Pedidos

| Campo | Valor |
| --- | --- |
| Status | Em revisão |
| Data | 14/10/2026 |
| Autor | Elder Gil (redação a partir da reunião técnica) |
| Product Manager na reunião | Marcos |
| Documentos relacionados | [RFC](RFC.md) · [FDD](FDD.md) · [ADRs](adrs/README.md) · [Tracker](TRACKER.md) |

`[hh:mm] Nome` indica a fala da reunião técnica de origem ([`TRANSCRICAO.md`](../TRANSCRICAO.md)). O que é proposta deste documento está marcado como tal.

## 1. Resumo e contexto

O Order Management System passa a avisar os clientes B2B, por webhook, quando o status de um pedido muda. Hoje esses clientes descobrem mudanças consultando a API de tempos em tempos ([09:00] Marcos). Com a feature, a plataforma chama um endereço cadastrado pelo cliente, com um aviso assinado, em menos de 10 segundos ([09:02] Marcos).

O pedido veio formalmente de três clientes: Atlas Comercial, MaxDistribuição e Nova Cargo ([09:00] Marcos). O fluxo é só de saída: os clientes recebem, não enviam ([09:02] Marcos).

## 2. Problema e motivação

- **Integração lenta e cara.** Os clientes ficam batendo em GET /orders para ver se algo mudou, e isso deixa a integração lenta e cara para eles ([09:00] Marcos).
- **Atualização manual.** O cliente quer saber da mudança sem precisar "ficar atualizando manualmente" ([09:02] Marcos).
- **Risco comercial.** A Atlas sugeriu que pode migrar para um concorrente se isso não for entregue até o fim do trimestre ([09:00] Marcos).
- **Nada para reaproveitar.** O código atual não tem nenhum mecanismo de notificação, evento ou fila (`src/modules/` só tem auth, users, customers, products e orders). A feature começa do zero, dentro dos padrões do projeto.

## 3. Público-alvo e cenários de uso

**Público.**
- **Clientes B2B integradores**, a começar por Atlas, MaxDistribuição e Nova Cargo ([09:00] Marcos). Eles recebem os avisos e precisam verificar a assinatura e descartar repetições.
- **Usuários que representam o cliente** na plataforma, que cadastram e mantêm os webhooks pela API com o JWT do sistema ([09:32] Marcos). Premissa a confirmar: no código, usuário e customer não têm vínculo e as roles são só ADMIN e OPERATOR (`prisma/schema.prisma`), então hoje qualquer usuário autenticado faz esse papel (risco 4).
- **Administradores da plataforma** (role ADMIN), que reprocessam avisos que falharam de vez ([09:36] Sofia).

**Cenários de uso.**
1. **Cadastro.** O integrador cadastra um endereço HTTPS e escolhe os status que quer receber, por exemplo só SHIPPED e DELIVERED ([09:33] Marcos). Recebe uma secret gerada pela plataforma ([09:31] Marcos).
2. **Aviso.** Um operador muda o status de um pedido; em segundos, o endereço do cliente recebe o aviso com o status anterior, o novo e os dados básicos do pedido ([09:43] Diego). Para ver os itens, o cliente consulta o pedido ([09:43] Diego).
3. **Verificação.** O cliente confere a assinatura com a secret dele e descarta o aviso se não bater ([09:20] Sofia).
4. **Repetição.** O cliente recebe o mesmo aviso duas vezes e descarta a repetição pelo identificador do evento ([09:25] Diego).
5. **Indisponibilidade.** O sistema do cliente fica fora do ar por duas horas numa manutenção; os avisos chegam quando ele volta, porque a plataforma continua tentando por horas ([09:16] Diego, [09:17] Diego).
6. **Falha definitiva.** Depois de esgotar as tentativas, o aviso fica guardado; um ADMIN reprocessa quando o cliente corrige o problema ([09:18] Diego).
7. **Histórico.** O integrador consulta os últimos 100 avisos enviados para ele, com sucesso ou falha, conteúdo, resposta e tempo de resposta ([09:34] Marcos).
8. **Troca de secret.** O integrador desconfia de vazamento e pede uma secret nova; a antiga continua valendo por 24 horas enquanto ele atualiza os sistemas dele ([09:21] Sofia).

## 4. Objetivos e métricas de sucesso

| Objetivo | Métrica | Meta | Origem |
| --- | --- | --- | --- |
| Avisar em "tempo real" na visão do cliente | Tempo entre a mudança de status e a entrega do aviso a um cliente disponível, no percentil 95 (o percentil é proposta deste documento) | **Menos de 10 segundos** | [09:02] Marcos |
| Nenhuma mudança de status sem aviso, nenhum aviso sem mudança | Duas verificações, propostas por este documento. Nos testes de integração, mudança revertida com aviso gravado ou mudança confirmada sem o aviso esperado (OT-1 do FDD). Em produção, conferência diária de que todo aviso registrado tem a mudança de status correspondente no histórico do pedido | **Zero divergências** | [09:06] Diego, [09:40] Bruno |
| Sobreviver à indisponibilidade do cliente | Janela em que a plataforma continua tentando antes de considerar falha definitiva | **Cobrir pelo menos uma manutenção de 2 horas**. Com os intervalos combinados, a janela é de quase 15 horas se forem 6 envios, ou cerca de 2h36 se forem 5; a contagem está em aberto no [RFC](RFC.md#levantadas-na-reunião-e-adiadas-ou-não-decididas). Diego também citou "12 ou 24 horas" ([09:15] Diego) | [09:16] Diego, [09:17] Diego, [09:17] Marcos |
| Atender o prazo comercial | Data de entrada em produção | **Fim de novembro**, prazo pedido pela Atlas, a confirmar com ela. A estimativa do time é de três sprints com a revisão de segurança incluída | [09:45] Marcos, [09:47] Larissa, [09:47] Marcos |

A conferência em produção só olha em uma direção. Para saber se uma mudança antiga deveria ter gerado aviso, seria preciso saber quais webhooks e filtros valiam naquele momento, e o modelo não guarda o histórico de edições. A outra direção é garantida pela construção, com o aviso gravado na mesma transação da mudança ([09:40] Bruno), e é verificada nos testes.

O polling dos clientes deve cair depois da adoção, mas a reunião não trouxe número de chamadas atuais nem meta de redução. Fica como observação, sem meta.

## 5. Escopo

### 5.1 Incluso

- Cadastro, edição, remoção e listagem de webhooks por customer, com filtro de status ([09:31] Marcos, [09:33] Bruno).
- Aviso a cada mudança de status do pedido que o webhook quer receber ([09:00] Marcos).
- Assinatura de cada aviso, com secret por endpoint e troca de secret ([09:22] Sofia).
- Novas tentativas automáticas e guarda dos avisos que falharam de vez, com reprocessamento por ADMIN ([09:17] Larissa, [09:18] Diego).
- Histórico das últimas entregas por webhook ([09:34] Marcos).
- Documentação de integração no portal do desenvolvedor, a cargo do Marcos ([09:26] Marcos, [09:40] Marcos).

### 5.2 Fora de escopo

| Item | Situação na reunião | Origem |
| --- | --- | --- |
| E-mail ao cliente quando o webhook falha repetidamente | Adiado: "talvez próxima fase, depois que a gente medir o impacto" | [09:37] Larissa |
| Painel visual para o cliente ver os webhooks | Descartado nesta fase: "Só endpoints. Painel é projeto separado do time de frontend" | [09:40] Larissa |
| Rate limiting de envio por cliente | Em aberto: "observar e decidir depois" | [09:39] Larissa |
| Arquivamento dos avisos entregues | Fora desta feature | [09:08] Diego |
| Vários workers em paralelo e ordem global dos avisos | Adiado: "problema do futuro" | [09:13] Diego |
| Webhooks enviados pelos clientes para a plataforma | Fora: só saída | [09:02] Marcos |
| Restringir o cadastro por role | Adiado: "mais pra frente a gente pode endurecer" | [09:37] Sofia |

## 6. Requisitos funcionais

| ID | Requisito | Origem |
| --- | --- | --- |
| RF-01 | O usuário cadastra um webhook para um customer, informando a URL e a lista de status que quer receber; o customer é informado na requisição, não vem do JWT | [09:31] Marcos, [09:32] Larissa |
| RF-02 | A plataforma gera a secret do webhook e a devolve na criação | [09:31] Marcos |
| RF-03 | O usuário edita um webhook existente | [09:33] Bruno |
| RF-04 | O usuário remove um webhook. Avisos ainda não entregues para ele deixam de ser enviados, e o histórico continua consultável (comportamento proposto no FDD) | [09:33] Bruno |
| RF-05 | O usuário lista os webhooks de um customer | [09:33] Bruno |
| RF-06 | Cada webhook recebe só os status da lista dele; mudança para um status que nenhum webhook do customer quer não gera aviso | [09:33] Marcos, [09:34] Bruno |
| RF-07 | A cada mudança de status de um pedido, cada webhook interessado recebe um aviso com status anterior, status novo e dados básicos do pedido, sem os itens. A criação do pedido não gera aviso (proposta; lacuna no RFC) | [09:00] Marcos, [09:43] Diego |
| RF-08 | Cada aviso leva uma assinatura que permite ao cliente verificar origem e integridade | [09:19] Sofia, [09:20] Sofia |
| RF-09 | O usuário troca a secret de um webhook pela API, e a antiga continua válida por 24 horas | [09:21] Sofia |
| RF-10 | Cada aviso leva um identificador de evento, igual em todas as tentativas, para o cliente descartar repetições | [09:25] Diego |
| RF-11 | Cada aviso identifica o cadastro de webhook que o recebeu | [09:44] Sofia |
| RF-12 | Aviso não entregue é reenviado automaticamente em intervalos crescentes; esgotadas as tentativas, fica guardado como falha definitiva | [09:15] Diego, [09:17] Larissa, [09:18] Diego |
| RF-13 | Um ADMIN reprocessa um aviso guardado como falha definitiva, e a plataforma registra quem fez o reprocessamento | [09:18] Diego, [09:36] Sofia, [09:36] Larissa |
| RF-14 | O usuário consulta o histórico das últimas 100 entregas de um webhook, com sucesso ou falha, conteúdo, resposta e tempo de resposta | [09:34] Marcos |
| RF-15 | O usuário desativa e reativa um webhook sem removê-lo (proposta: a reunião citou o estado ativo na configuração, não a operação; o comportamento durante a pausa está no FDD) | [09:21] Bruno |
| RF-16 | Cada aviso informa o momento do envio, pensado para o cliente detectar reenvio malicioso. Como esse dado não entra na assinatura, a proteção que ele dá depende da revisão de segurança (lacuna no [RFC](RFC.md#lacunas-identificadas-na-análise-não-discutidas-na-reunião)) | [09:44] Diego |

Rotas, payload e headers de cada requisito estão no [FDD](FDD.md#5-contratos-públicos).

## 7. Requisitos não funcionais

| ID | Requisito | Origem |
| --- | --- | --- |
| RNF-01 | Aviso entregue em menos de 10 segundos a um cliente disponível | [09:02] Marcos |
| RNF-02 | O mecanismo de envio acrescenta até 2 segundos de espera antes da primeira tentativa, quando o envio não está ocupado com clientes lentos (o limite de envios simultâneos está no FDD) | [09:10] Larissa |
| RNF-03 | A chamada ao cliente tem timeout de 10 segundos; depois disso, conta como falha | [09:42] Diego |
| RNF-04 | URL do webhook só com HTTPS; cadastro com HTTP é recusado | [09:23] Sofia |
| RNF-05 | Aviso maior que 64 KB não é enviado nem truncado; é tratado como erro | [09:24] Larissa |
| RNF-06 | Garantia at-least-once: o cliente pode receber o mesmo aviso mais de uma vez | [09:24] Diego |
| RNF-07 | Avisos de um mesmo pedido chegam na ordem enquanto houver um único processo de envio; não há ordem entre pedidos diferentes. Exceção: um aviso reprocessado manualmente chega depois dos seguintes que já tinham sido entregues. Custo: se um aviso do pedido está esperando nova tentativa, os seguintes daquele pedido esperam também (FDD) | [09:13] Larissa |
| RNF-08 | Secret única por endpoint, nunca uma secret global | [09:21] Sofia |
| RNF-09 | Lentidão ou indisponibilidade do cliente não afeta a mudança de status de nenhum pedido | [09:04] Bruno |
| RNF-10 | Reinício da API não interrompe o envio de avisos | [09:11] Diego |
| RNF-11 | Nenhuma infraestrutura nova: mesmo banco MySQL e mesma stack | [09:07] Diego, [09:11] Diego |

## 8. Decisões e trade-offs principais

O que cada decisão técnica significa para o produto. O porquê de cada uma está no ADR.

| Para o produto | ADR |
| --- | --- |
| Nunca há mudança de status sem aviso registrado, e o aviso não atrasa a operação do pedido | [ADR-001](adrs/ADR-001-outbox-no-mysql.md) |
| O aviso pode levar até 2 s a mais para sair, e a ordem é garantida só dentro de um pedido | [ADR-002](adrs/ADR-002-worker-em-processo-separado-com-polling.md) |
| Um cliente fora do ar recebe os avisos quando volta, às vezes horas depois | [ADR-003](adrs/ADR-003-retry-com-backoff-exponencial-e-dlq.md) |
| O cliente precisa verificar a assinatura para ter a garantia de origem | [ADR-004](adrs/ADR-004-hmac-sha256-com-secret-por-endpoint.md) |
| O cliente precisa descartar avisos repetidos | [ADR-005](adrs/ADR-005-entrega-at-least-once-com-x-event-id.md) |
| A API de webhooks responde e erra no mesmo formato do resto da plataforma | [ADR-006](adrs/ADR-006-reuso-dos-padroes-do-projeto.md) |
| O aviso descreve o pedido no momento da mudança, mesmo que ele tenha mudado depois | [ADR-007](adrs/ADR-007-snapshot-do-payload-na-insercao.md) |

## 9. Dependências

- **Código existente.** A mudança de status em `src/modules/orders/order.service.ts` precisa gravar o aviso na mesma transação ([09:40] Bruno). Detalhes de integração no [FDD](FDD.md#9-integração-com-o-sistema-existente).
- **Revisão de segurança.** Sofia revisa o código de HMAC e de geração de secret, com pelo menos dois dias úteis reservados antes do deploy ([09:46] Sofia).
- **Sessão de revisão técnica.** Larissa revisa o design com Bruno e Diego antes de começar a codar ([09:50] Larissa). É o lugar natural para fechar as questões de design do [RFC](RFC.md#questões-em-aberto), como a contagem de envios. As de segurança vão para a revisão da Sofia.
- **Operação.** Um processo novo, separado da API, precisa subir e ser acompanhado em produção ([09:11] Diego).
- **Portal do desenvolvedor.** Marcos documenta a integração e destaca a necessidade de descartar repetições ([09:26] Marcos, [09:40] Marcos).
- **Confirmação do prazo.** Marcos confirma o prazo com a Atlas e atualiza os clientes ([09:47] Marcos, [09:49] Marcos).

## 10. Riscos e mitigação

Probabilidade e impacto são avaliação deste documento, com a justificativa quando há fala de apoio.

| Risco | Probabilidade | Impacto | Mitigação |
| --- | --- | --- | --- |
| Atrasar além do prazo da Atlas e perder o cliente para o concorrente ([09:00] Marcos) | Média: a estimativa de três sprints é um chute da Larissa ([09:46] Larissa) e o prazo foi citado de duas formas, fim do trimestre e fim de novembro ([09:45] Marcos) | Alto: perda de cliente | Confirmar o prazo com a Atlas ([09:47] Marcos); manter fora de escopo e-mail, painel e rate limiting; reservar a revisão de segurança no plano ([09:47] Larissa) |
| Cliente não descarta avisos repetidos e processa a mesma mudança duas vezes | Média: a garantia é at-least-once ([09:24] Diego) | Médio: efeito duplicado no sistema do cliente | Identificador de evento em todo aviso e destaque na documentação do portal ([09:26] Marcos) |
| Secret vazada no lado do cliente, como já aconteceu uma vez ([09:22] Diego) | Média | Alto: avisos forjados para aquele cliente | Secret por endpoint e troca com 24 h de convivência ([09:21] Sofia) |
| Um usuário autenticado cadastra um webhook com URL própria para um customer que não é o dele e passa a receber os avisos de pedidos desse customer | Média: a reunião liberou o cadastro para qualquer role autenticada ([09:37] Sofia), e usuário e customer não têm vínculo no sistema. O acesso entre customers é consequência dessa combinação, não algo que a reunião aceitou | Alto: vazamento de dados de pedidos de um cliente | Levar à revisão de segurança da Sofia antes do deploy; restringir exige mudança de modelo de dados (questão 4 do [RFC](RFC.md#levantadas-na-reunião-e-adiadas-ou-não-decididas)) |
| Rajada de mudanças de status gera rajada de avisos para um cliente ([09:38] Diego) | Média | Médio para o cliente | Medir os envios por webhook e decidir o rate limiting com dado ([09:39] Larissa) |
| Qualquer pessoa cria uma conta ADMIN pelo cadastro público da API e passa a reprocessar avisos, que deveria ser só de administradores | Alta, se a API estiver exposta como está: o cadastro não exige login e aceita a role ADMIN (`src/modules/auth/auth.schemas.ts`) | Alto: a restrição de ADMIN do reprocessamento, e de qualquer rota administrativa, deixa de proteger | Problema do código existente, fora do escopo desta feature. Levar à revisão de segurança da Sofia antes do deploy ([RFC](RFC.md#lacunas-identificadas-na-análise-não-discutidas-na-reunião)) |

## 11. Critérios de aceitação

- [ ] Um usuário cadastra um webhook HTTPS com uma lista de status e recebe a secret na resposta; cadastro com HTTP é recusado.
- [ ] Uma mudança de status listada pelo webhook gera um aviso; uma mudança não listada não gera.
- [ ] Um cliente disponível recebe o aviso em menos de 10 segundos, no percentil 95.
- [ ] O cliente valida a assinatura do aviso com a secret recebida.
- [ ] Todas as tentativas de um mesmo aviso chegam com o mesmo identificador de evento.
- [ ] Com o endpoint do cliente fora do ar, a plataforma tenta de novo nos intervalos combinados e, esgotadas as tentativas, guarda o aviso como falha definitiva.
- [ ] Um ADMIN reprocessa um aviso guardado; um usuário sem role ADMIN não consegue; o reprocessamento fica registrado com o autor. Enquanto o cadastro público aceitar ADMIN (risco 6), este critério prova a regra da rota, não a proteção.
- [ ] Depois de trocar a secret, o cliente consegue verificar os avisos com a secret nova ou com a antiga por 24 horas; depois disso, só com a nova. Nenhuma troca seguinte encurta essas 24 horas. A forma de assinar durante a carência e a recusa de nova troca nesse período são propostas do FDD e dependem da revisão da Sofia.
- [ ] O histórico mostra as últimas 100 entregas do webhook, com sucesso ou falha, conteúdo, resposta e tempo.
- [ ] Mudanças de status continuam funcionando com o endpoint do cliente fora do ar ou lento.
- [ ] A revisão de segurança da Sofia foi feita antes do deploy.

## 12. Estratégia de testes e validação

- **Testes automatizados no padrão do projeto.** Testes de integração com Vitest e Supertest, como em `tests/orders.test.ts`. Prioridade para:
  - o aviso gravado junto com a mudança de status e descartado no rollback;
  - o filtro por status;
  - a recusa de URL sem HTTPS;
  - o reprocessamento só por ADMIN.
- **Worker contra um endpoint de teste (proposta).** Um servidor HTTP local que responde sucesso, erro, demora ou queda, para validar timeout, novas tentativas, falha definitiva e assinatura. Os critérios técnicos estão no [FDD](FDD.md#11-critérios-de-aceite-técnicos).
- **Revisão de segurança.** Sofia revisa HMAC e geração de secret antes do deploy ([09:46] Sofia).
- **Validação com os clientes (proposta).** Primeiro webhook em produção com um dos três clientes que pediram a feature, acompanhando latência e falhas pelo histórico de entregas, antes de abrir para os demais. A reunião não definiu piloto, e ele precisa caber no prazo da Atlas; se não couber, a validação acontece com os três ao mesmo tempo. Marcos ficou de atualizar os clientes ([09:49] Marcos).
- **Regressão.** A suíte atual (`tests/orders.test.ts` e `tests/auth.test.ts`) continua passando.
