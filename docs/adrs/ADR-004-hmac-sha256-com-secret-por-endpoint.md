# ADR-004: HMAC-SHA256 com secret por endpoint

- **Data:** reunião técnica de quinta-feira, 09:00 (a transcrição não registra a data)
- **ADRs relacionadas:** [ADR-005](ADR-005-entrega-at-least-once-com-x-event-id.md), [ADR-006](ADR-006-reuso-dos-padroes-do-projeto.md)

## Status

Aceito. Sofia fechou com "Decidido: HMAC-SHA256 sobre o corpo do request, secret por endpoint, suporte a rotação com grace period de 24h" ([09:22] Sofia). O resumo final repetiu a decisão ([09:48] Larissa).

## Contexto

Os webhooks levam dados de pedidos para endpoints fora da infraestrutura da plataforma. O cliente tem que conseguir validar que a requisição veio da plataforma e que ninguém adulterou o payload no meio do caminho ([09:19] Sofia).

A configuração de cada webhook guarda url, secret, customer_id e estado ativo ([09:21] Bruno), o que Sofia confirmou ([09:21] Sofia). Um customer pode ter vários endpoints, tanto que Sofia pediu um header com o id do webhook para o cliente saber qual cadastro recebeu cada envio ([09:44] Sofia). Diego lembrou que um cliente já vazou secret em log da própria aplicação ([09:22] Diego).

Quem gera a secret ficou definido na lista de requisitos: a plataforma gera e devolve na criação ([09:31] Marcos). Isso é coerente com o cliente "pedir nova secret" na rotação ([09:21] Sofia) e com a revisão que Sofia vai fazer da "geração de secret" ([09:46] Sofia). A única pista em sentido contrário é o código de erro WEBHOOK_SECRET_REQUIRED, dado como exemplo por Bruno antes dessa regra ([09:28] Bruno).

## Fatores de decisão

- O cliente precisa verificar origem e integridade de cada requisição ([09:19] Sofia).
- O algoritmo precisa ser padrão de mercado, com biblioteca disponível para os clientes ([09:20] Sofia).
- O vazamento de uma secret não pode comprometer os outros clientes ([09:21] Sofia).
- Já houve cliente que vazou secret em log ([09:22] Diego), então a troca de secret precisa existir.
- O cliente precisa de tempo para migrar os sistemas dele durante a troca ([09:21] Sofia).

## Alternativas Consideradas

### 1. Secret global da plataforma

Uma única secret para assinar os envios de todos os clientes. Sofia citou a opção para descartá-la ([09:21] Sofia).

- **Prós:** uma chave só para gerar, guardar e trocar (análise, não fala da reunião).
- **Contras:** "se vaza uma, vaza tudo" ([09:21] Sofia).
- **Por que foi descartada:** Sofia exigiu secret única por endpoint ([09:21] Sofia).

### 2. Secret informada pelo cliente

O cliente escolhe a secret e a envia no cadastro. Ninguém defendeu essa opção; a única pista é o código WEBHOOK_SECRET_REQUIRED, citado como exemplo por Bruno ([09:28] Bruno) antes de Marcos listar os requisitos.

- **Prós:** o cliente reaproveitaria o próprio gerenciamento de segredos (análise, não fala da reunião).
- **Contras:** a plataforma perde o controle sobre a qualidade da chave (análise, não fala da reunião).
- **Por que foi descartada:** contradiz a regra de que a secret é gerada pela plataforma e devolvida na criação ([09:31] Marcos). Por isso o código WEBHOOK_SECRET_REQUIRED não entra na matriz de erros do FDD.

## Decisão

Cada envio é assinado com HMAC-SHA256 sobre o corpo do request, e a assinatura vai num header de assinatura que o cliente verifica do lado dele ([09:20] Sofia). O SHA-256 foi escolhido por ser o padrão de mercado ([09:20] Sofia).

A secret é única por endpoint de webhook, nunca global ([09:21] Sofia). Ela é gerada pela plataforma e devolvida na criação do webhook ([09:31] Marcos). A secret é rotacionável pela API: o cliente pede uma nova, e a antiga continua válida por 24 horas em paralelo; depois disso, a antiga morre ([09:21] Sofia). Nomes de headers, rotas e o armazenamento da secret estão no FDD.

## Consequências

### Positivas

- O cliente consegue rejeitar requisições forjadas ou adulteradas no caminho ([09:19] Sofia).
- Um vazamento atinge um único endpoint, não a base de clientes ([09:21] Sofia).
- A carência de 24 horas dá tempo ao cliente para migrar os sistemas dele na troca ([09:21] Sofia).

### Negativas

- O cliente precisa implementar a verificação do HMAC; sem isso, a assinatura não protege nada (análise, não fala da reunião).
- A plataforma passa a guardar uma secret por endpoint, mais a antiga durante a carência, e ela precisa ser recuperável para assinar cada envio (análise, não fala da reunião).
- A reunião não definiu com qual secret a plataforma assina durante as 24 horas de carência, nem se o timestamp do envio entra na assinatura. Os dois pontos estão nas questões em aberto do [RFC](../RFC.md), para a revisão de segurança de Sofia ([09:46] Sofia).

### Trade-off

Aceitamos o custo de gerar, guardar e trocar uma secret por endpoint, e de exigir verificação do lado do cliente, em troca de autenticidade e integridade por endpoint, com vazamento de alcance limitado.

## Referências

- Transcrição: [09:19] Sofia, [09:20] Sofia, [09:21] Sofia, [09:21] Bruno, [09:22] Diego, [09:22] Sofia, [09:28] Bruno, [09:31] Marcos, [09:44] Sofia, [09:46] Sofia, [09:48] Larissa
