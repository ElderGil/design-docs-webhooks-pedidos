# ADR-004: HMAC-SHA256 com secret por endpoint

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md), [ADR-008](ADR-008-uma-linha-de-outbox-por-evento-e-webhook.md)

## Status

Aceito. Sofia fechou com "Decidido: HMAC-SHA256 sobre o corpo do request, secret por endpoint, suporte a rotação com grace period de 24h" ([09:22] Sofia). O resumo final repetiu a decisão ([09:48] Larissa).

## Contexto

Os webhooks levam dados de pedidos para endpoints fora da infraestrutura da plataforma. O cliente precisa conseguir validar que a requisição veio da plataforma e que ninguém adulterou o payload no caminho ([09:19] Sofia). A feature é só de saída: os clientes recebem, não enviam ([09:03] Sofia).

Cada customer cadastra os próprios endpoints, e a configuração de cada um guarda url, secret, customer_id e estado ativo ([09:21] Bruno), com confirmação de Sofia ([09:21] Sofia). Um customer pode ter vários endpoints, tanto que Sofia pediu o header X-Webhook-Id para o cliente saber qual cadastro recebeu cada envio ([09:44] Sofia). Diego lembrou que um cliente já vazou secret em log da própria aplicação ([09:22] Diego), o que torna a rotação uma necessidade concreta.

Quem gera a secret ficou definido na lista de requisitos: a plataforma gera e devolve na criação ([09:31] Marcos). Duas falas apontam no mesmo sentido: o cliente "pede nova secret" na rotação ([09:21] Sofia), e Sofia vai revisar "HMAC e geração de secret" antes do deploy ([09:46] Sofia). A única pista em sentido contrário é o código de erro WEBHOOK_SECRET_REQUIRED, citado como exemplo por Bruno antes dessa regra ([09:28] Bruno).

## Fatores de decisão

- O cliente precisa verificar origem e integridade de cada requisição ([09:19] Sofia).
- O algoritmo precisa ser padrão de mercado, com biblioteca disponível para os clientes ([09:20] Sofia).
- O vazamento de uma secret não pode comprometer os outros clientes ([09:21] Sofia).
- Já houve cliente que vazou secret em log, então trocar a secret tem que ser possível ([09:22] Diego).
- O cliente precisa de tempo para migrar os sistemas dele durante a troca ([09:21] Sofia).

## Alternativas Consideradas

### 1. Secret global da plataforma

Uma única secret para assinar os envios de todos os clientes. Sofia citou a opção como contraponto ([09:21] Sofia).

- **Prós:** uma só chave para gerar, guardar e rotacionar.
- **Contras:** se vaza uma, vaza tudo ([09:21] Sofia); qualquer cliente que conhece a secret consegue forjar envios para os outros.
- **Por que foi descartada:** Sofia exigiu secret única por endpoint ([09:21] Sofia).

### 2. Secret informada pelo cliente

O cliente escolhe a secret e a envia no cadastro. Ninguém defendeu essa opção. A única pista é o código WEBHOOK_SECRET_REQUIRED, citado como exemplo por Bruno ([09:28] Bruno) antes de Marcos listar os requisitos.

- **Prós:** o cliente pode reaproveitar o próprio gerenciamento de segredos.
- **Contras:** permite secret fraca ou reaproveitada entre sistemas, e a plataforma perde o controle sobre a qualidade da chave.
- **Por que foi descartada:** contradiz a regra de que a secret é gerada pela plataforma e devolvida na criação ([09:31] Marcos).

### 3. Modelo híbrido

O cliente informa a secret ou, se não informar, a plataforma gera. Ninguém propôs essa opção na reunião. Fica registrada como descartada por não ter origem na reunião e por somar os riscos da alternativa 2 ao fluxo de geração.

## Decisão

Cada envio é assinado com HMAC-SHA256 sobre o corpo do request, e a assinatura vai no header X-Signature ([09:20] Sofia). O cliente verifica a assinatura do lado dele ([09:20] Sofia). O SHA-256 foi escolhido por ser o padrão de mercado: todo cliente sério tem biblioteca para isso ([09:20] Sofia).

A secret é única por endpoint de webhook, nunca global ([09:21] Sofia). Ela é gerada pela plataforma e devolvida na criação do webhook ([09:31] Marcos). A secret é rotacionável pela API: o cliente pede uma nova, e a antiga continua válida por 24 horas em paralelo antes de deixar de valer ([09:21] Sofia). Sofia vai revisar o código de HMAC e de geração de secret, com pelo menos dois dias úteis reservados antes do deploy ([09:46] Sofia).

HTTPS obrigatório na URL ([09:23] Sofia) e o limite de 64 KB de payload ([09:24] Larissa) foram tratados na reunião como validação e requisito não funcional, não como decisão arquitetural. Ver PRD e FDD.

## Consequências

### Positivas

- O cliente consegue rejeitar requisições forjadas ou alteradas no caminho ([09:19] Sofia).
- Um vazamento atinge um único endpoint, não a base de clientes ([09:21] Sofia).
- Rotação com 24 horas de carência deixa o cliente trocar a secret sem janela de rejeição ([09:21] Sofia).
- Secret gerada pela plataforma evita chave fraca escolhida pelo cliente ([09:31] Marcos).

### Negativas

- O cliente precisa implementar a verificação do HMAC; sem isso, a assinatura não protege nada ([09:20] Sofia).
- A plataforma passa a guardar e gerenciar uma secret por endpoint, mais a antiga durante a carência ([09:21] Sofia).
- O `redact` do logger em `src/shared/logger/index.ts` não cobre um campo `secret` (ver [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)).
- Ponto em aberto: a reunião não disse com qual secret a plataforma assina durante as 24 horas de rotação (a nova, a antiga ou as duas). Está no [RFC](../RFC.md).
- Ponto em aberto: o X-Timestamp foi pensado para o cliente detectar replay attack ([09:44] Diego), mas a assinatura cobre só o corpo ([09:22] Sofia). Sem o timestamp dentro do HMAC, quem capturar uma requisição pode trocar o header e reenviar, o que enfraquece essa detecção. Está no [RFC](../RFC.md).
- Ponto em aberto: o armazenamento da secret não foi discutido. Como ela precisa ser recuperável para assinar cada envio, não pode ficar só como hash. Está no [RFC](../RFC.md).

### Trade-off

Aceitamos o custo de gerar, guardar e rotacionar uma secret por endpoint, e de exigir verificação do lado do cliente, em troca de autenticidade e integridade por endpoint, com vazamento de alcance limitado.

## Referências

- Transcrição: [09:03] Sofia, [09:19] Sofia, [09:20] Sofia, [09:21] Sofia, [09:21] Bruno, [09:22] Diego, [09:22] Sofia, [09:23] Sofia, [09:24] Larissa, [09:28] Bruno, [09:31] Marcos, [09:44] Diego, [09:44] Sofia, [09:46] Sofia, [09:48] Larissa
- Código: `src/shared/logger/index.ts`, `prisma/schema.prisma` (onde a configuração de webhook será criada)
