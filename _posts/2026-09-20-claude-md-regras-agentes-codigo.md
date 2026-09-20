---
title: "O arquivo CLAUDE.md que tenta deixar os agentes de código menos impulsivos"
date: 2026-09-20 16:18:00 -03:00
description: "Um arquivo de texto inspirado nas observações de Andrej Karpathy reúne quatro regras para reduzir suposições, excesso de engenharia e alterações fora do escopo em agentes de programação."
tags:
  - inteligência artificial
  - programação
  - Claude Code
  - agentes de IA
  - GitHub
---

Agentes de programação conseguem escrever código em poucos segundos. O problema é que velocidade não resolve tudo. Às vezes, a ferramenta escolhe uma interpretação sem perguntar, cria uma solução maior do que o necessário e ainda altera arquivos que nem faziam parte do pedido.

Um arquivo chamado **CLAUDE.md** tenta reduzir esse tipo de comportamento com uma ideia simples: colocar regras de trabalho dentro do próprio projeto para que o agente leia essas orientações antes de começar.

## O arquivo não foi criado por Karpathy

A primeira correção importante é de autoria. O arquivo não foi publicado diretamente por Andrej Karpathy no repositório `github.com/karpathy`. O projeto está no repositório [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) e foi criado a partir de observações públicas de Karpathy sobre os problemas recorrentes dos modelos de linguagem ao programar.

Karpathy foi diretor de inteligência artificial e Autopilot na Tesla, mas isso não transforma todo material inspirado em suas ideias em uma publicação oficial dele. A distinção parece pequena, mas é importante quando um projeto começa a circular rapidamente nas redes sociais.

Também há outra correção de nome: o arquivo é `CLAUDE.md`, não `cloud.md`. O formato é um arquivo Markdown comum, colocado na raiz de um projeto para orientar o Claude Code. A ideia também pode ser adaptada a outras ferramentas que leem instruções do repositório.

## Quatro problemas que o arquivo tenta evitar

### 1. Suposições silenciosas

Quando o pedido é ambíguo, um agente pode escolher uma interpretação e seguir em frente como se ela fosse óbvia. O resultado pode até parecer bem escrito, mas resolver o problema errado.

A regra propõe que o agente declare as suposições, mostre alternativas quando existirem e pare para pedir esclarecimentos se a dúvida mudar a implementação.

Na prática, isso evita situações como escolher PostgreSQL quando o projeto usa SQLite, criar uma rota nova quando já existe um padrão no sistema ou assumir que uma função deve ser síncrona sem verificar o restante da aplicação.

### 2. Excesso de engenharia

Outro comportamento comum é transformar uma tarefa simples em uma pequena arquitetura. O usuário pede uma validação e recebe uma camada de abstração, uma configuração genérica, uma fábrica e vários arquivos novos.

A orientação do projeto é direta: use o mínimo de código que resolve o pedido. Não crie flexibilidade que ninguém solicitou. Se uma solução de 50 linhas resolve o problema, não há mérito em transformá-la em 200 linhas apenas para parecer mais completa.

Isso não significa rejeitar arquitetura. Significa fazer a arquitetura acompanhar a necessidade real, em vez de tentar prever todos os futuros possíveis antes de existir um segundo caso de uso.

### 3. Alterações fora do escopo

Agentes também costumam aproveitar uma tarefa para reorganizar o código ao redor. Um pedido para corrigir um erro pode terminar com renomeação de variáveis, reformatação de arquivos, remoção de comentários e mudanças em funções que não estavam quebradas.

A regra de **mudanças cirúrgicas** tenta limitar esse impulso. O agente deve tocar apenas no que for necessário para o pedido e limpar somente os resíduos criados pela própria alteração.

O teste sugerido é simples: cada linha modificada precisa ter uma relação clara com a solicitação. Se uma alteração não puder ser explicada dessa forma, ela provavelmente deve ficar para outro trabalho.

### 4. Falta de verificação

Dizer “corrija o bug” é um objetivo fraco. Um agente pode alterar o código e declarar vitória sem demonstrar que o erro foi reproduzido ou que deixou de acontecer.

A quarta regra transforma pedidos vagos em critérios verificáveis. Corrigir um bug deve significar criar ou executar um teste que falhe antes da correção e passe depois. Adicionar validação deve incluir casos inválidos. Refatorar uma função deve preservar os testes antes e depois da mudança.

Essa orientação troca a aparência de conclusão por uma evidência concreta de que o trabalho foi feito.

## É um arquivo pequeno, não uma instalação mágica

A proposta chama atenção porque não exige uma biblioteca, um servidor ou um modelo novo. Em um projeto compatível com Claude Code, o arquivo pode ser colocado na raiz para funcionar como instrução de contexto.

O repositório oferece, entre outras opções, este comando para baixar o arquivo:

```bash
curl -o CLAUDE.md https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md
```

Se o projeto já tiver um `CLAUDE.md`, não se deve sobrescrever o arquivo sem ler o conteúdo atual. As regras gerais podem ser combinadas com as instruções específicas do projeto, desde que não criem contradições.

Também é importante entender o limite da técnica. Um arquivo de instruções não garante que um agente seguirá tudo perfeitamente, não substitui revisão humana, não impede erros de API e não cria credenciais que não existem. Ele apenas torna algumas expectativas explícitas no momento em que o agente começa a trabalhar.

## O que muda para quem programa com IA?

A mudança mais útil talvez não esteja nas quatro regras isoladamente, mas no modo de formular a tarefa. Em vez de escrever apenas “faça funcionar”, o usuário passa a definir o que precisa ser comprovado:

- qual comportamento deve mudar;
- quais arquivos podem ser alterados;
- quais partes devem permanecer intactas;
- qual teste precisa passar;
- qual resultado encerra o trabalho.

Isso também melhora a revisão. Um diff pequeno, acompanhado de testes claros, é mais fácil de analisar do que uma grande reescrita apresentada como melhoria geral.

O arquivo não corrige o Claude no sentido literal. Ele não muda o modelo por dentro e não elimina alucinações. O que ele faz é estabelecer um contrato de trabalho local. Quando o agente respeita esse contrato, a conversa fica menos baseada em adivinhação e mais baseada em objetivo, limite e verificação.

Para quem já usa agentes de código, é uma intervenção barata e fácil de testar. Crie uma cópia do projeto, aplique as regras, observe os diffs e compare o resultado com o fluxo anterior. Se a ferramenta continuar assumindo demais ou mexendo fora do escopo, o problema não estará resolvido apenas porque existe um arquivo Markdown na raiz.

A melhor regra continua sendo a mais simples: não confundir uma instrução escrita com uma prova de que o trabalho foi executado corretamente.

Acompanhe as próximas análises no canal da Ana Clara no WhatsApp: https://www.whatsapp.com/channel/0029VaiPYBPLo4heVf0U3u2d

### Fontes

- [Repositório andrej-karpathy-skills no GitHub](https://github.com/forrestchang/andrej-karpathy-skills)
- [Arquivo CLAUDE.md](https://raw.githubusercontent.com/forrestchang/andrej-karpathy-skills/main/CLAUDE.md)
- [Perfil de Andrej Karpathy no GitHub](https://github.com/karpathy)
