---
title: "Jev: a IA que não escreve textos, mas decide dentro dos softwares"
date: 2026-09-23 23:10:00 -0300
description: "O Jev, da TypeSafe AI, devolve decisões tipadas, probabilidades e confiança para aplicações que precisam classificar, pontuar e encaminhar tarefas."
tags:
  - inteligência artificial
  - automação
  - tecnologia
  - software
  - agentes de IA
image: "/assets/images/jev-chatgpt-dos-softwares.png"
image_alt: "Fluxos luminosos conectam uma inteligência artificial a painéis de decisões estruturadas e sistemas de software."
---

<p><strong>Áudio do post:</strong></p>
<audio controls preload="none" aria-label="Versão em áudio desta publicação" src="{{ '/assets/media/2026-09-23-jev-modelo-de-decisoes-ia.mp3' | relative_url }}">Seu navegador não suporta áudio. <a href="{{ '/assets/media/2026-09-23-jev-modelo-de-decisoes-ia.mp3' | relative_url }}">Baixe o áudio</a>.</audio>

# Jev: a IA que não escreve textos, mas decide dentro dos softwares

**Por Tisha News**

A inteligência artificial se acostumou a falar. Pergunte, ela explica. Peça um resumo, ela entrega um resumo, talvez com uma introdução sobre a importância do resumo. O Jev, modelo da TypeSafe AI, tenta resolver outro problema: como fazer uma IA responder para um software sem obrigá-lo a interpretar um pequeno ensaio a cada decisão.

Em vez de gerar prosa, o Jev avalia um estado, que pode ser uma mensagem, um registro ou um histórico, diante de perguntas definidas pelo desenvolvedor. A resposta vem em formatos tipados, como `Choice`, `Score` e `Noul`, o equivalente a uma pergunta booleana de sim ou não. O resultado inclui probabilidades e, em alguns tipos de resposta, uma medida separada de confiança. [1] [2]

## Quando uma aplicação não quer conversar

Pense em uma caixa de entrada de suporte. Cada chamado precisa ser encaminhado para uma equipe, receber uma avaliação de gravidade e talvez ser marcado para revisão humana. Um modelo generalista pode escrever “parece ser um caso urgente de cobrança, com alta probabilidade”. Um programa, porém, prefere algo mais próximo de:

```json
{
  "department": "billing",
  "severity": 2,
  "needs_review": true
}
```

O Jev foi desenhado para esse tipo de tarefa. A aplicação define as perguntas e as opções possíveis. `Choice` escolhe uma opção de um conjunto, como `billing`, `technical` ou `account`. `Score` posiciona o caso em uma escala ordenada, como baixo, médio ou alto risco. `Noul`, nome usado na documentação da Cloudflare, responde a uma questão binária; na integração descrita pela Vercel, a mesma família aparece como resposta `boolean`. [1] [2]

A vantagem prática está no contrato da resposta. O código sabe quais campos esperar e quais alternativas existem. Não aparece uma quinta categoria inventada no meio do caminho, nem uma justificativa com um emoji onde o sistema esperava uma chave de banco de dados. Isso reduz o trabalho de transformar linguagem em instrução de software, mas não elimina a necessidade de conferir se a decisão faz sentido.

## Como o Jev funciona

O fluxo começa com um `state`, o material que será avaliado. Pode ser o texto de um chamado, um objeto com dados de uma conta ou uma combinação dos dois. Depois, o desenvolvedor descreve perguntas atômicas, cada uma com seu tipo e seus critérios.

Uma pergunta de escolha pode ser: “qual equipe deve tratar este chamado?”. Uma pergunta de pontuação pode ser: “qual é o nível de urgência, de 0 a 2?”. Uma pergunta `Noul` pode verificar: “o cliente pediu reembolso?”. O modelo avalia essas perguntas contra o mesmo estado e devolve as respostas sob os identificadores definidos pela aplicação. A documentação da Cloudflare mostra exemplos com `choice`, `score` e `noul`, acompanhados de distribuições de probabilidade; a Vercel descreve ainda o uso de confiança para respostas de escolha e pontuação. [1] [2]

Essa separação deixa as regras de negócio no código. O Jev pode identificar que alguém pediu um reembolso. A aplicação ainda precisa consultar a política, o plano do cliente, o histórico da conta e as permissões antes de aprovar qualquer pagamento. Identificar uma intenção não é autorizar uma operação. [2]

## Usos que fazem sentido

O primeiro grupo de usos é o da classificação e do encaminhamento. Chamados podem seguir para cobrança, suporte técnico ou segurança. Documentos podem ser separados por tipo. Mensagens podem ser avaliadas para decidir se entram em uma fila de atendimento ou em uma revisão manual.

Há também tarefas de pontuação. Uma empresa pode avaliar a urgência de uma solicitação, o risco aparente de uma conta ou a gravidade de um incidente. O resultado não precisa mandar no sistema sozinho. Ele pode apenas alimentar uma regra: casos com probabilidade alta seguem automaticamente; casos incertos esperam uma pessoa.

Outro uso é verificar ações propostas por agentes. Antes de um agente executar um comando, o Jev pode avaliar se a ação parece destrutiva, se toca um ambiente de produção ou se exige aprovação. A Vercel recomenda definir limites por ação, com critérios mais rigorosos para operações que podem apagar dados ou causar prejuízo. [2]

A reportagem da TechCrunch também cita a possibilidade de usar o Jev para acompanhar rastros de agentes e identificar comportamentos problemáticos, como tentativas de contornar restrições. [3] Nesse desenho, ele não é o agente que conversa com o usuário e faz tudo. É uma camada de avaliação dentro de um sistema maior.

## O que ele tem a ver com agentes de IA

Agentes precisam escolher ferramentas, interpretar resultados e decidir o próximo passo. Modelos generalistas continuam úteis para conversar, planejar e lidar com tarefas abertas. O Jev entra quando uma etapa pode ser reduzida a uma pergunta bem definida, com saídas que o software consegue usar diretamente.

Uma arquitetura possível combina os dois: o modelo generalista entende o pedido; o Jev avalia uma condição específica; as regras da aplicação determinam o caminho; e um humano assume os casos ambíguos ou sensíveis. A entrevista de Diogo Almeida ao Latent Space apresenta justamente essa visão de modelos “System One” voltados para produção, com decisões pequenas e programáveis, em vez de uma máquina que tenta ser um chatbot universal. [4]

Isso também explica por que o Jev não deve ser descrito como um chatbot menor. Ele não foi criado para escrever uma receita, conversar sobre férias ou substituir modelos generalistas. É um modelo de avaliação e decisão. A TypeSafe AI o posiciona como uma alternativa para tarefas em que a linguagem livre é uma camada desnecessária entre a análise e o código. [2] [3]

## Rápido e barato, mas em quais condições?

A promessa chamou a atenção de desenvolvedores porque uma saída sem prosa pode reduzir o volume de processamento e facilitar a integração. A TechCrunch relata um teste da Vercel em que o Jev respondeu entre cinco e 18 vezes mais rápido e com maior precisão do que o modelo usado antes para classificar comandos de segurança. A mesma reportagem cita um teste de classificação de e-mails no qual outro desenvolvedor encontrou diferença de custo em relação ao Gemini. [3]

Esses números descrevem testes específicos, não uma tabela universal de desempenho. Entrada, perguntas, critérios, provedor, infraestrutura, limites de confiança e método de avaliação mudam o resultado. A documentação da Vercel lista, para o AI Gateway, preço de US$ 0,042 por milhão de tokens de entrada e ausência de cobrança por tokens de saída, nas condições daquele serviço. [2] O dado não deve ser transformado em promessa para qualquer integração.

Também é preciso separar velocidade de qualidade. Uma resposta que chega cedo, no formato correto e com probabilidade alta ainda pode estar errada. O modelo não gera texto livre, mas continua fazendo uma avaliação probabilística.

## Saída válida não é decisão correta

Este é o freio que impede a novidade de virar piloto automático.

O esquema limita o tipo de resposta. Ele ajuda a impedir uma categoria fora da lista e torna o retorno mais previsível para o programa. Nada disso prova que a classificação esteja correta. A própria Vercel alerta que o formato tipado não garante a conclusão. [2]

Probabilidade também não é verdade. Uma aplicação precisa testar o Jev com exemplos rotulados, medir falsos positivos e falsos negativos e verificar se as probabilidades correspondem ao que acontece no mundo. Os limites devem ser escolhidos conforme o risco da ação. Mostrar uma tela errada é diferente de bloquear uma conta, cancelar um pedido ou liberar uma transferência.

Decisões automatizadas ainda exigem registro, explicação e possibilidade de contestação. Se um sistema recusar um reembolso ou marcar uma pessoa como risco, a equipe precisa conseguir reconstruir o caminho da decisão e corrigir o processo. Uma saída tipada pode ser fácil de ler por uma máquina e difícil de questionar por quem sofreu seus efeitos. Esse é um problema de governança, não de sintaxe.

## A inteligência que desaparece no fundo do sistema

O Jev aponta para uma forma menos teatral de usar IA. Em vez de uma janela chamativa, ele pode ficar escondido em uma fila de suporte, em uma regra de roteamento ou na verificação de uma ferramenta de agente. O objetivo, segundo a visão apresentada pela TypeSafe AI e discutida por Almeida, é que a inteligência se torne parte comum do software, quase como uma função que ninguém comenta quando funciona. [3] [4]

Mas invisibilidade técnica não pode significar falta de transparência. Quanto mais decisões passam para os bastidores, mais importantes ficam os registros, os testes, os limiares claros e a intervenção humana.

O Jev não representa o fim dos modelos que escrevem. Representa uma divisão de trabalho. Há tarefas em que uma resposta bem redigida é o produto. Há outras em que o melhor resultado é uma escolha, uma nota ou um “sim” acompanhado de incerteza. Para essas situações, a pergunta mais útil talvez não seja “a IA consegue conversar?”, mas “o software sabe o que fazer com a resposta, e alguém consegue perceber quando ela falha?”.

## Fontes

[1] Cloudflare AI Docs. “Jev (typesafe)”. https://developers.cloudflare.com/ai/models/typesafe/jev/

[2] Vercel. “How to classify, route, and score with Jev and AI SDK”. https://vercel.com/kb/guide/typesafe-jev-and-ai-sdk

[3] Tim Fernholz, TechCrunch. “A new kind of AI model from a ChatGPT inventor is thrilling developers”. 18 de setembro de 2026. https://techcrunch.com/2026/09/18/a-new-kind-of-ai-model-from-a-chatgpt-inventor-is-thrilling-developers/

[4] Latent Space. “Jev: System One models for Prod, not God”, entrevista com Diogo Almeida. https://www.latent.space/p/jev

## Sources

[1] https://developers.cloudflare.com/ai/models/typesafe/jev — Cloudflare AI Docs: Jev
[2] https://vercel.com/kb/guide/typesafe-jev-and-ai-sdk — Vercel: How to classify, route, and score with Jev and AI SDK
[3] https://techcrunch.com/2026/09/18/a-new-kind-of-ai-model-from-a-chatgpt-inventor-is-thrilling-developers — TechCrunch: A new kind of AI model from a ChatGPT inventor is thrilling developers
[4] https://www.latent.space/p/jev — Latent Space: Jev with Diogo Almeida
