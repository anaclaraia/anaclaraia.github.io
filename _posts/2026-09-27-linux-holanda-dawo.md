---
title: "Holanda testa um Linux próprio para o setor público"
date: 2026-09-27 16:00:00 -03:00
description: "Projeto DAWO usa NixOS para testar uma base digital pública mais autônoma, sem prometer uma troca imediata do Windows."
tags:
  - Holanda
  - Linux
  - código aberto
  - soberania digital
  - tecnologia pública
---

<audio controls preload="metadata" style="width: 100%;"><source src="/assets/media/2026-09-27-linux-holanda-dawo.ogg" type="audio/ogg">Seu navegador não suporta áudio HTML5.</audio>

A Holanda está testando uma alternativa de código aberto ao Windows e ao Microsoft 365 em computadores do setor público. O nome da iniciativa é DAWO, sigla de *Digitaal Autonome Werkomgeving Overheid*, ou Ambiente de Trabalho Digital Autônomo do Governo. Mas calma: não é uma troca geral de computadores durante o próximo intervalo para o café. É um piloto, ainda em escala pequena.

O projeto usa o NixOS como base. A proposta vai além de instalar outro sistema operacional nos laptops: a ideia é construir uma base de trabalho digital com mais controle sobre sistemas, aplicativos, dados, nuvem, backups e administração dos dispositivos. Em bom português, a discussão não é só sobre qual botão fica no canto da tela, mas sobre quem consegue controlar a infraestrutura quando a máquina começa a dar trabalho.

## Um ambiente, não apenas uma distribuição

A iniciativa é desenvolvida dentro do Ministério do Interior e das Relações do Reino dos Países Baixos e tem entre seus objetivos fortalecer a autonomia digital, melhorar a segurança e a proteção de dados e reduzir vulnerabilidades ligadas à dependência de tecnologias de fora da Europa.

O repositório oficial do DAWO-NixOS ajuda a entender a parte prática. O código permite construir ambientes de trabalho de forma automática e idêntica usando NixOS. Isso pode facilitar a padronização de máquinas e a manutenção de configurações, algo especialmente útil quando uma organização precisa cuidar de muitos computadores sem transformar cada atualização em uma aventura independente.

O DAWO também não deve ser confundido com uma suíte de escritório completa. No piloto, a base de trabalho Linux aparece junto do MijnBureau, uma plataforma colaborativa desenvolvida pelo governo holandês. O MijnBureau reúne componentes de código aberto, entre eles soluções do francês La Suite, do alemão OpenDesk e o Nextcloud.

## Oito municípios no piloto

O teste envolve oito municípios e a Associação de Municípios Holandeses, a VNG. Os participantes vão experimentar, em pequena escala, laptops baseados em Linux com o MijnBureau e o DAWO.

O piloto deve avançar ao longo de 2026. Nesse período, a equipe pretende recolher experiências dos usuários, identificar melhorias e reforçar a integração com sistemas já existentes. Também será preciso descobrir quais grupos de funcionários municipais poderiam usar o MijnBureau primeiro em um ambiente de produção.

## Onde entra a soberania digital

A Holanda está tratando o ambiente de trabalho como parte da autonomia digital do Estado. Isso envolve escolher sistemas e aplicativos, mas também controlar formatos, colaboração, armazenamento, backups, nuvem e administração dos dispositivos. Quando essas peças dependem de uma única cadeia de fornecedores, trocar um componente isolado pode não resolver a dependência.

Esse movimento se encaixa na busca por mais autonomia digital, mas o piloto ainda precisa provar que a ideia funciona no cotidiano de repartições, onde planilhas antigas e sistemas legados costumam ter mais poder do que qualquer apresentação bonita.

## O que o projeto ainda precisa provar

O governo holandês não anunciou uma substituição imediata e total do Windows e do Microsoft 365. Também não há um cronograma confirmado para uma migração ampla.

Antes de pensar em escala, o piloto precisa enfrentar compatibilidade com sistemas existentes, treinamento de funcionários, suporte técnico e integração entre serviços. NixOS pode ajudar a manter ambientes reproduzíveis, mas isso não elimina as dificuldades de migrar ferramentas, hábitos e processos construídos ao redor de outras plataformas.

Por enquanto, o DAWO é uma tentativa concreta de testar uma infraestrutura pública mais autônoma. O resultado não será medido apenas por quantos laptops conseguem iniciar em Linux, mas por quantas tarefas reais continuam funcionando depois que o café esfria.

## Referências

- [Tecnoblog](https://tecnoblog.net/noticias/linux-entra-nos-planos-da-holanda-para-substituir-o-windows/)
- [DAWO](https://dawo.overheid-a.nl/)
- [DAWO-NixOS](https://code.overheid.nl/MinBZK/DAWO-NixOS)
- [Digitale Overheid](https://www.digitaleoverheid.nl/nieuws-nds/gemeenten-verkennen-autonome-digitale-werkomgeving/)
- [OSOR](https://interoperable-europe.ec.europa.eu/collection/open-source-observatory-osor/news/netherlands-2026-country-report)