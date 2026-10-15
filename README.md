# Design docs do Sistema de Webhooks de Notificação de Pedidos

Desafio 04 do MBA em Engenharia de Software com IA da Full Cycle. Este repositório é um fork de [devfullcycle/mba-ia-desafio-design-docs-com-ia](https://github.com/devfullcycle/mba-ia-desafio-design-docs-com-ia), onde está o enunciado original.

## Sobre o desafio

Uma empresa com um Order Management System em produção fez uma reunião de 55 minutos para decidir como avisar clientes B2B quando o status de um pedido muda. Três clientes pediram, e um deles já falou em ir para o concorrente se não tiver isso até o fim do trimestre. A decisão foi tomada na call. Só que nada foi registrado além da transcrição.

O trabalho aqui é transformar essa transcrição e o código existente (Node.js, TypeScript, Prisma e MySQL) em um pacote de design docs que o time consiga pegar e começar a implementar: PRD, RFC, FDD, ADRs e um tracker que liga cada item à origem. A regra é simples: nada inventado. Se uma linha não tem por trás um `[hh:mm] Nome` da reunião ou um arquivo do código, ela não entra. O código da aplicação não foi alterado; ele é contexto e referência.

## Ferramentas de IA utilizadas

- **Claude Code com Claude Opus 5.5.** Ferramenta principal, para ler o código e a transcrição, fazer a triagem fala a fala e apoiar a escrita e as correções dos documentos.
- **Claude Sonnet 5.5 como revisor independente.** Um subagente só de leitura, no papel de advogado do diabo, revisando cada documento sem saber o que eu achava. O modelo diferente foi de propósito: quem escreve não aprova o próprio texto.
- **Revisão final com outro agente e outro modelo.** Com o pacote pronto, uma revisão completa, fora do fluxo de escrita. Ela leu as 274 linhas do tracker uma a uma, confrontou o desenho com o código e conferiu os critérios do enunciado.
- **Material do curso.** O prompt de FDD do módulo de Design Docs deu o esqueleto do FDD. O agente `adr-generator`, do plugin `adrs-management` do curso, deu o formato das ADRs. A aula "Principais Seções em um PRD de Feature" orientou o PRD.
- **Scripts Python de conferência.** Fiz scripts pequenos para checar o que dá para checar sem julgamento: se cada `[hh:mm] Nome` existe na transcrição com aquele falante, se cada caminho de código existe, se os links e as âncoras abrem, e as porcentagens do tracker.

## Workflow adotado

Segui a ordem que o enunciado sugere, com uma etapa a mais no começo:

1. **Triagem da transcrição.** Antes de qualquer documento, classifiquei cada fala: decisão, requisito, restrição, alternativa descartada, fora de escopo, adiado, em aberto, detalhe ou gancho com o código. Saíram 209 linhas, e foi dessa matriz que saíram os timestamps de todos os documentos.
2. **ADRs**, depois **RFC**, depois **FDD**, depois **PRD** e, por último, o **tracker**.
3. **README**.

Cada documento passou pelo mesmo ciclo:

- escrevo a partir da transcrição, do código e da triagem;
- scripts conferem citações, caminhos e links;
- um revisor de outro modelo ataca o conteúdo;
- confiro cada achado na fonte antes de aceitar;
- corrijo e faço o commit.

Duas regras valeram para os cinco documentos:

- Três marcas. `[hh:mm] Nome` quer dizer que veio da reunião. **(proposta)** quer dizer que é escolha do documento sobre algo que a reunião não decidiu. **(a criar)** quer dizer que o arquivo ainda não existe no código. O FDD precisa decidir coisa que a reunião não fechou, como onde vai o `customer_id` ou o que acontece com um evento quando o worker cai. Sem a marca, isso viraria "decisão da reunião" inventada.
- Cada ponto no documento certo. O que a reunião levantou e não fechou vai para "Questões em aberto" no RFC. O que eu achei e a reunião nem discutiu vai para "Lacunas identificadas na análise", também no RFC. O detalhe de implementação vai para o FDD, como proposta.

Algumas decisões que tomei no caminho, depois de pesar as opções:

- A secret do webhook é gerada pela plataforma. O Bruno cita um código `WEBHOOK_SECRET_REQUIRED`, mas o Marcos e a Sofia falam em secret gerada e devolvida.
- Uma linha de outbox por evento e por webhook, para cada endpoint ter retry e DLQ próprios. Entrou como proposta, porque a reunião não decidiu.
- O snapshot do payload virou ADR, porque foi decidido na reunião e tem trade-off real.
- A ADR-008 saiu. Tinha começado como ADR "Proposta". Saiu porque ADR registra decisão fechada, e essa não estava.

## Prompts customizados

**Triagem da transcrição.** Escrevi para este desafio. É a base do tracker e o que mais segurou os documentos contra informação inventada.

```text
Você é um analista de requisitos fazendo a triagem de uma reunião técnica.
Leia TRANSCRICAO.md inteira, fala por fala, e o código em src/ e prisma/schema.prisma.

Para cada afirmação com conteúdo, gere uma linha:
| # | Timestamp | Falante | Categoria | Conteúdo (uma linha) | Status | Observação |

Categorias (só estas): DECISAO, RF, RNF, RESTRICAO, ALTERNATIVA (quem rejeitou e por quê),
FORA_ESCOPO, ADIADO, ABERTO, CONTEXTO, DETALHE, GANCHO_CODIGO.
Status: fechado, proposto, corrigido (cite a fala que substitui), descartado.

Regras:
1. Um timestamp e um falante por linha, exatamente como na transcrição.
2. Não interprete além do texto. Ambiguidade vira ABERTO, com a explicação na Observação.
3. Fala que contradiz uma anterior: marque a anterior como corrigido.
4. Para cada GANCHO_CODIGO, confira no código e diga o caminho real ou "NÃO EXISTE (a criar)".
5. Nada descartado, adiado ou só cogitado vira requisito.

Depois da tabela: decisões fechadas com a fala que fecha; alternativas com o trade-off;
fora de escopo, adiado e aberto; ambiguidades e contradições; contagem por categoria.
Prefiro 120 linhas corretas a 40 resumidas.
```

**Advogado do diabo.** Usei a mesma estrutura nos cinco documentos, trocando só o alvo e a lista de ataques. Este é o do FDD.

```text
Você é o advogado do diabo de um FDD. NÃO edite arquivos; só leia.
Ache problemas; se um item estiver certo, escreva "sem achado".

Leia o enunciado, a TRANSCRICAO.md, o FDD, o RFC e as ADRs, e o código em src/, prisma/ e tests/.
Confira CADA afirmação que o FDD faz sobre o código existente.

O FDD usa três marcas: [hh:mm] Nome = veio da reunião; (proposta) = escolha do FDD;
(a criar) = arquivo novo. O enunciado proíbe inventar requisitos, decisões ou restrições
sem origem e reprova se um arquivo citado como existente não existir.

Ataque:
1. Origem: há escolha do FDD apresentada como decisão da reunião, sem a marca de proposta?
   A pessoa citada disse aquilo naquele minuto?
2. Código: alguma afirmação sobre arquivos existentes é falsa? Algum arquivo novo sem "(a criar)"?
3. Critérios do enunciado, ao pé da letra.
4. Furos de lógica no desenho (ordem, recuperação, remoção, reprocessamento).
5. Contradição com RFC e ADRs.
6. Excesso: proposta que aumenta a superfície de invenção sem ser exigida.

Saída: achados numerados com a frase exata do FDD e a evidência (fala ou arquivo:linha).
Depois, "bloqueia a entrega" e "melhoria".
```

**Tracker.** Trecho principal do prompt que montou a primeira versão do tracker.

```text
Escreva docs/TRACKER.md lendo inteiros a transcrição, o PRD, o RFC, o FDD e os ADRs.
Formato obrigatório: | ID | Documento | Tipo | Conteúdo (resumo) | Fonte | Localização |
Fonte só TRANSCRICAO ou CODIGO. Localização: um [hh:mm] Nome que exista na transcrição
e cuja fala sustente o item, ou um caminho de arquivo real.
Cubra pelo menos 85% dos itens; pelo menos 72% das linhas com TRANSCRICAO e 8 com CODIGO.
Se a origem real é o código, a fonte é CODIGO: não force.
Proposta do documento: aponte a fala ou o arquivo que MOTIVOU e comece o resumo com "(proposta)".
Nunca deixe parecer que a reunião decidiu o que é proposta.
Confira cada localização antes de escrever; errar o falante invalida a linha.
```

## Iterações e ajustes

Foram oito ciclos principais de geração e revisão: o plano, a triagem e um por documento, com as ADRs passando por dois. Depois veio uma revisão final do pacote inteiro. Em cada etapa, o revisor de outro modelo achou algo que tinha passado na versão anterior.

### 1. O plano tinha erro de atribuição antes de existir documento

Pedi uma segunda opinião sobre o plano. O Sonnet achou o `requireRole` atribuído à fala errada e uma contradição do Bruno que eu não tinha visto. Ele diz "mesmo Prisma client" em [09:11] Bruno e "Separado" em [09:30] Bruno. Achou também ambiguidades que acabaram no RFC: uma linha de outbox por evento ou por webhook, qual secret assina durante a rotação, e o `X-Timestamp` fora da assinatura.

### 2. As ADRs tinham alternativa que ninguém propôs

A primeira versão passou em todos os scripts: as 392 citações existiam na transcrição. O problema? O script confere se o par `[hh:mm] Nome` existe, não se a pessoa disse aquilo. O advogado do diabo achou três tipos de problema:

- alternativas inventadas: um modelo "híbrido" de secret e um "LISTEN/NOTIFY" que o Diego só citou para dizer que o MySQL não tem;
- prós e contras sem dono;
- uma ADR inteira, a 008, registrando uma decisão que a reunião não tomou.

Removi a ADR-008, marquei toda inferência como "(análise, não fala da reunião)" e tirei das ADRs o que era detalhe de FDD. Também mudei o processo: a partir dali, nenhum documento chegou a mim sem passar pelo revisor de outro modelo.

### 3. O RFC "resolvia" uma ambiguidade chamando de leitura literal

A Larissa fecha "5 tentativas, backoff 1m/5m/30m/2h/12h". Só que são cinco intervalos, e o Diego diz que dão "quase 15 horas", o que só fecha com seis envios. Eu tinha registrado 1 + 5 como "leitura literal da transcrição". Não é. Literal é "total 5 tentativas" ([09:48] Larissa). Virou a questão em aberto número 1, e o FDD usa 1 + 5 como premissa provisória. O revisor também mostrou que três das minhas "questões em aberto" nunca foram levantadas na reunião. Elas foram para uma subseção de lacunas da análise, separada do que veio da reunião.

### 4. O FDD tinha furo de lógica, além de origem

Três exemplos:

- "pausar o webhook com `active: false`" não pausava nada, porque o worker não olhava esse campo;
- o `request_id` gravado na outbox exigia mudar a assinatura do `changeStatus`, enquanto o próprio FDD dizia que ela não mudava;
- `statuses` como lista não existe no Prisma com MySQL.

Cada um foi corrigido e virou critério de aceite técnico.

### 5. No tracker, o formato estava certo e o sentido nem sempre

O tracker tinha 257 linhas e passou em todos os scripts. Na amostra de 73 linhas que o revisor leu na fonte, 60 sustentavam o item por inteiro, 9 em parte e 4 não sustentavam. Uma citava "eu atualizo os clientes hoje à tarde" como origem de um piloto que a reunião nunca definiu. Corrigi as 13, troquei para CODIGO o que vinha do código, e não da fala, e declarei no cabeçalho a regra do que conta como item.

### 6. A revisão final achou contradição que eu tinha marcado como proposta

Com o pacote publicado, passei tudo por mais uma revisão, com outro agente e outro modelo. Ela leu as 274 linhas do tracker uma a uma e confrontou o desenho com o código. Os achados que mais mudaram os documentos:

- Na rotação de secret, uma segunda troca descartava a secret antiga na hora e quebrava as 24 h que a Sofia definiu. Estava marcado como "(proposta)". O problema? Marcar como proposta não resolve contradição com requisito fechado. Agora uma nova troca durante a carência é recusada com 409.
- O diagrama dizia "busca webhooks ativos", e o texto dizia que webhook pausado acumula eventos. Quem seguisse o diagrama perderia os eventos da pausa.
- O cadastro público da API aceita `role: ADMIN` sem login. O reprocessamento "só por ADMIN" confere a role, mas qualquer um consegue essa role. Não é bug da feature, e o desafio não mexe no código. Virou lacuna no RFC e risco no PRD e no FDD.
- `created_at` com milissegundo empata, e o MySQL não garante a ordem de linhas empatadas. A ordem por pedido passou a usar uma coluna `seq`.
- O FDD escrevia as colunas em snake_case, mas o projeto usa camelCase (`customerId` em `orders`). Num `$queryRaw`, isso vira erro.
- A cobertura de 80% não estava demonstrada. As 274 linhas não dizem quantos itens os documentos têm. O tracker ganhou um inventário por seção, com numerador, denominador e o que ficou fora da conta.

O worker também ficou mais preciso no que promete:
- os 2 s de espera valem quando há vaga livre;
- cada tentativa fecha numa transação só;
- o corpo da resposta do cliente é lido até 4 KB, não inteiro para cortar depois.

Duas coisas eu não fiz. Não troquei a arquitetura, que a revisão manteve. E não corrigi o código herdado.

Antes de publicar as correções, passei o diff pelo Sonnet de novo. Ele pegou o que eu tinha deixado escapar na correção. O mais importante: a coluna `seq` usa autoincremento, e a Larissa tinha respondido "Tudo é uuid" quando o Diego perguntou sobre id autoincremental ([09:51] Larissa). A `seq` não é id, mas o conflito existe. Ficou como proposta, com lacuna no RFC para a sessão de revisão. Ele também mostrou que a métrica de fila parada contaria eventos que só esperavam o anterior do mesmo pedido, e que a cobertura precisava de um mapeamento unidade por unidade para alguém conseguir refazer a conta.

Choque de realidade: script garante formato, só leitura garante sentido. O que mais pegou erro neste desafio foi pôr um segundo modelo para discordar do primeiro e conferir cada discordância na transcrição. E a revisão final mostrou que vale trocar de revisor: ela achou coisa em trecho que já tinha passado por outra revisão.

## Como navegar a entrega

Para ler, sugiro a ordem do mais alto para o mais detalhado. Ela é diferente da ordem em que os documentos foram produzidos.

| Ordem | Arquivo | O que responde |
| --- | --- | --- |
| 1 | [`docs/PRD.md`](docs/PRD.md) | Por que e o quê: problema, público, escopo e métricas |
| 2 | [`docs/RFC.md`](docs/RFC.md) | O que propomos, quais alternativas ficaram na mesa e o que ainda está em aberto |
| 3 | [`docs/adrs/`](docs/adrs/) | Por que cada decisão foi tomada assim, uma por arquivo (sete ADRs) |
| 4 | [`docs/FDD.md`](docs/FDD.md) | Como construir: fluxos, contratos, erros, observabilidade e integração com o código |
| 5 | [`docs/TRACKER.md`](docs/TRACKER.md) | De onde veio cada item, na transcrição ou no código |
| | [`TRANSCRICAO.md`](TRANSCRICAO.md) | A reunião original, sem alteração |

Ao ler qualquer documento, `[hh:mm] Nome` remete à fala na transcrição, **(proposta)** marca o que a reunião não decidiu e **(a criar)** marca o que ainda não existe no código.
