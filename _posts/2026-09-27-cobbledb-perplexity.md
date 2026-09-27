---
title: "CobbleDB: a Perplexity cansou de pagar pedágio"
date: 2026-09-27 15:40:00 -03:00
description: "Como a Perplexity trocou o DynamoDB por um banco próprio e derrubou a latência de leituras em lote sem fingir que toda empresa precisa fazer o mesmo."
image: /assets/media/2026-09-27-cobbledb-perplexity.png
tags:
  - Perplexity
  - CobbleDB
  - bancos de dados
  - Rust
  - engenharia de software
  - inteligência artificial
---

<audio controls preload="metadata" style="width: 100%;"><source src="/assets/media/2026-09-27-cobbledb-perplexity.ogg" type="audio/ogg">Seu navegador não suporta áudio HTML5.</audio>

# CobbleDB: a Perplexity cansou de pagar pedágio

A Perplexity olhou para a conta e para a latência do DynamoDB e tomou uma decisão que, em qualquer reunião de tecnologia, costuma provocar duas reações: “faz sentido” e “vocês fizeram o quê?”. A empresa construiu o próprio banco de dados, o CobbleDB, para atender uma carga bem específica do seu buscador com IA.

O resultado divulgado é daqueles que fazem o gráfico parecer ter tomado energético: a latência mediana das leituras em lote caiu de 31,4 ms para 5,60 ms. No p90, foi de 56,7 ms para 9,77 ms. No p99, de 123 ms para 24,2 ms. A Perplexity também estima pelo menos 20% de economia em comparação com o DynamoDB.

Mas calma. Isso não significa que o DynamoDB ficou ruim ou que toda startup deveria jogar o banco gerenciado pela janela. Significa que a Perplexity tem uma carga muito particular, em escala de gente grande, e decidiu trocar um serviço genérico por uma ferramenta feita sob medida. É a diferença entre comprar um canivete e mandar fabricar uma tesoura que só corta o seu tipo de papel.

## O problema era o tamanho da sacola

Uma consulta na Perplexity não pede apenas um título, uma URL e aquele pedacinho de texto que cabe num resultado de busca tradicional. Para montar uma resposta, o sistema precisa buscar passagens preparadas de páginas e seus embeddings, as representações vetoriais usadas no processamento da IA.

Cada consulta gera cerca de 100 a 120 chaves de páginas. O serviço divide esse pedido em lotes de aproximadamente 10 a 15 ou 10 a 20 chaves. Cada item preparado tem cerca de 50 KB. Faça isso em um tráfego de aproximadamente 200 mil requisições por segundo e a brincadeira deixa de ser “vamos consultar um banco” para virar “quem autorizou esta esteira industrial?”.

No DynamoDB, a empresa tinha a conveniência de um serviço gerenciado, com alta disponibilidade e sem precisar operar cada engrenagem de um banco distribuído. O problema apareceu no encontro entre payloads grandes, muitas leituras em lote e cobrança por volume transferido. A conta cresceu junto com o tráfego, enquanto a equipe também precisava lidar com variações de latência em leituras que estavam no caminho crítico das respostas.

## Três sistemas, cada um no seu quadrado

A arquitetura nova separa tarefas que antes ficavam mais grudadas. O Pillar cuida do estado durável dos documentos e da publicação das versões. O Lorry agrupa atualizações em lotes. O CobbleDB fica com a parte quente: entregar rapidamente os registros preparados durante a consulta.

O CobbleDB é um armazenamento distribuído key-value escrito em Rust. Ele guarda as representações já processadas das páginas, com passagens e embeddings, e foi desenhado para leituras em lote. Não é um banco genérico tentando fazer tudo ao mesmo tempo. É mais parecido com uma cozinha de restaurante que só prepara o prato mais pedido do cardápio, mas consegue mandar centenas deles sem deixar o salão esperando.

Essa separação também evita misturar a atualização pesada do acervo com a leitura dos usuários. Quando páginas são reprocessadas, o sistema pode preparar e entregar os dados em lotes, enquanto os nós de serving continuam focados em responder consultas. O Lorry organiza os pacotes, o Pillar mantém a fonte durável e o CobbleDB tenta não deixar ninguém esperando na porta.

## Os números têm asterisco, como todo número honesto

A comparação apresentada pela Perplexity é de antes e depois em produção, acompanhada de testes sintéticos. Portanto, os valores não são um benchmark universal de “Rust sempre vence DynamoDB”. Eles descrevem o comportamento dessa arquitetura, nessa carga, nesse volume e com esse padrão de leitura.

A própria escolha envolve troca. O CobbleDB tem consistência eventual, e a operação de um banco distribuído próprio deixa de ser problema de um fornecedor para virar responsabilidade da equipe. Particionamento, réplicas, falhas, observabilidade, atualizações e incidentes agora precisam de gente preparada para cuidar deles. O almoço pode sair mais barato, mas alguém passou a lavar a louça.

Segundo a Perplexity, o projeto tem cerca de 40 mil linhas de Rust e foi desenvolvido em dois meses por dois engenheiros, com apoio de agentes de IA. A empresa diz que pretende abrir o código em breve, mas ainda não há uma data garantida. Até lá, o CobbleDB fica como um caso interessante de engenharia aplicada, não como receita de bolo para qualquer sistema que esteja achando seu banco caro.

A lição é menos “construa seu próprio banco” e mais “entenda exatamente o que sua carga está pedindo”. Para a Perplexity, o gargalo tinha nome, formato, tamanho e frequência: leituras em lote de registros grandes, em escala gigantesca. Nesse cenário, fazer uma ferramenta especializada pode valer a dor de cabeça. Para muita empresa, porém, o melhor banco próprio continua sendo aquele que não precisa ser operado às três da manhã.

## Referências

- [CobbleDB: como a Perplexity substituiu o DynamoDB](https://imasters.com.br/noticia/perplexity-troca-dynamodb-por-banco-proprio-e-corta-latencia-em-5x)
- [CobbleDB: Rebuilding AI search storage for lower latency and cost](https://www.perplexity.ai/hub/blog/cobbledb)
- [Home Made CobbleDB Replaces DynamoDB at Perplexity](https://www.infoq.com/news/2026/09/cobbledb-perplexity/)
