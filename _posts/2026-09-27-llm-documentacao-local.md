---
title: "LLM local reconstrói documentação a partir do código"
date: 2026-09-27 15:30:00 -03:00
description: "Um teste prático mostra como um LLM local pode transformar arquivos de código em documentação, com bons resultados e limites claros."
image: /assets/media/2026-09-27-llm-documentacao-local.png
tags:
  - inteligência artificial
  - LLM local
  - programação
  - documentação
---

<audio controls preload="metadata" style="width: 100%;"><source src="/assets/media/2026-09-27-llm-documentacao-local.ogg" type="audio/ogg">Seu navegador não suporta áudio HTML5.</audio>

# LLM local reconstrói documentação a partir do código

Documentar um projeto costuma ficar para depois. Quando o software cresce, esse “depois” pode virar um problema: quem chega ao código precisa descobrir sozinho o que cada arquivo faz, como as partes se conectam e quais caminhos existem para manutenção.

O programador Rich Edmonds decidiu testar se um LLM rodando no próprio computador poderia ajudar nessa tarefa. Em vez de fazer perguntas gerais ao modelo, ele entregou arquivos de projetos reais e pediu uma documentação baseada no código disponível. O modelo usado foi o Qwen 3.6:27b-q4_K_M, executado localmente.

O resultado foi bom nos projetos pequenos, mas não deve ser confundido com uma garantia de que qualquer software será documentado perfeitamente. A qualidade depende do tamanho do projeto, do contexto que cabe na execução e da revisão de quem conhece o sistema.

## Um plugin de um único arquivo

O primeiro teste envolveu um plugin para MyBB voltado a links de afiliados. A extensão administra varejistas e substituições de links em uma área própria. Quando alguém publica uma resposta, o plugin procura URLs de domínios cadastrados e acrescenta os parâmetros de afiliado antes da exibição. O sistema também usa cache para evitar impacto desnecessário no desempenho.

Como o plugin inteiro está concentrado em um arquivo, o modelo conseguiu entender o contexto e descrever seu funcionamento. A documentação produzida ficou próxima de um texto pronto para ser incorporado ao repositório, segundo o relato do autor.

Esse caso mostra uma situação favorável para a automação: pouco código, uma finalidade bem definida e relações que podem ser acompanhadas dentro do próprio arquivo. Ainda assim, a documentação gerada precisa ser conferida. Um modelo pode interpretar de forma errada uma regra de negócio ou deixar passar uma dependência que não aparece no trecho analisado.

## Um pequeno projeto ligado ao Home Assistant

O segundo experimento usou um projeto PHP que consulta o site da autoridade local para acompanhar a coleta de lixo. Os dados aparecem em uma página da aplicação e também podem ser acessados remotamente pelo Home Assistant, usado pela família para organizar tarefas da casa.

O projeto tinha quatro arquivos e menos de 15 KB no total: index.php, sync.php, api.php e admin.php. Para essa etapa, Edmonds usou uma Radeon RX 7900 XT com 20 GB de VRAM. O modelo não ficou sem espaço de contexto e, de acordo com o teste, não inventou funções que não estavam presentes nos arquivos.

A documentação identificou corretamente o painel principal, as visualizações, a legenda e a ligação entre as partes. Também descreveu o portal de administração, incluindo gerenciamento de usuários, datas de referência, substituições manuais e opções de configuração. O endpoint JSON foi reconhecido como uma parte voltada a automações do Home Assistant.

Nesse projeto, a documentação tinha uma utilidade que ia além do próprio autor. Embora o código fosse pequeno e simples, um texto explicando seu funcionamento poderia ajudar outra pessoa da casa a fazer uma alteração ou entender o sistema sem precisar ler cada arquivo.

## Quando o projeto fica maior

O terceiro teste envolveu um jogo web em PHP com 16 arquivos, entre controladores, serviços e arquivos da aplicação pública carregados no navegador. O modelo conseguiu interpretar boa parte das funções do backend, inclusive mudanças recentes relacionadas à mineração e aos cálculos feitos quando o jogador está offline.

Mas também perdeu alguns detalhes. Essa diferença é importante: o resultado continuou útil, porém já não tinha a mesma precisão observada no projeto menor. Quanto mais arquivos e relações existirem, maior a chance de alguma informação ficar fora do contexto disponível ou de uma conexão entre componentes ser descrita de maneira incompleta.

É por isso que a documentação gerada por LLM deve ser tratada como uma primeira versão. Ela pode economizar tempo ao organizar funções, entradas, saídas e fluxos que já estão no código. Não substitui a revisão humana, sobretudo quando o texto será usado para manutenção, segurança ou tomada de decisões sobre o sistema.

## O ganho e o risco de usar um modelo local

A vantagem mais direta é transformar código sem documentação em um ponto de partida legível. Isso pode facilitar a entrada de novos colaboradores, a manutenção de projetos antigos e a explicação de sistemas pessoais que nunca receberam comentários suficientes.

Rodar o modelo localmente também oferece uma camada de privacidade. Os arquivos podem permanecer no computador do desenvolvedor, sem serem enviados a um serviço externo. Essa característica não elimina todos os riscos, mas pode ser relevante para código proprietário, dados internos ou projetos que não devem sair do ambiente de trabalho.

O risco está em confiar demais em um texto que parece seguro. Uma documentação alucinada pode atribuir ao sistema uma função inexistente, omitir uma limitação ou explicar errado uma regra. Quanto mais importante o projeto, menos aceitável é aceitar o resultado sem comparação com o código e com o comportamento real da aplicação.

O teste de Edmonds sugere um uso prático e moderado: deixar o LLM local fazer o trabalho inicial, conferir o que foi escrito e ajustar os pontos que dependem de conhecimento humano. Para projetos pequenos, isso pode ser bastante eficiente. Em bases maiores, a escala e o contexto disponível continuam sendo limites concretos.

## Referências

- [Relato original sobre o experimento](https://www.xda-developers.com/deleted-documentation-for-project-asked-local-llm-to-reconstruct-it/)
