# anaclaraia.github.io
em breve

---

## Ana Clara • Blog

Blog editorial em português do Brasil, feito com Jekyll para o GitHub Pages nativo. O título e a linha acima são o conteúdo original deste README, preservado.

### Publicação

O repositório está preparado para **Deploy from a branch → main → / (root)** em Settings → Pages. Não é necessário GitHub Actions, tema remoto ou serviço de build adicional. Nenhum commit ou push é realizado por estes arquivos.

URL: https://anaclaraia.github.io. Caso mude de domínio, atualize `url` em `_config.yml`; em um site de projeto, configure também `baseurl`.

### Executar localmente

Requer Ruby e Bundler compatíveis com o pacote `github-pages`.

```sh
bundle install
bundle exec jekyll serve
```

Abra http://127.0.0.1:4000. Para validar sem servidor:

```sh
bundle exec jekyll build --strict_front_matter
```

A build é gravada em `_site/`, ignorada pelo Git. `Gemfile.lock` também é ignorado para acompanhar a resolução do ambiente Pages; se precisar de reprodutibilidade estrita, revise essa política e versione o lock gerado pelo seu ambiente compatível.

### Adicionar uma publicação

1. Crie a pasta `_posts/`, se ainda não existir.
2. Copie `templates/post.md` para `_posts/AAAA-MM-DD-slug.md`.
3. Ajuste o título, a data real, a descrição e os temas. Escreva e revise o conteúdo, remova instruções do modelo e confira fontes e direitos de imagens.
4. Confirme os caminhos de mídia e valide a build antes de publicar.

Front matter suficiente (o layout `post` já é aplicado por padrão):

```yaml
title: "Título da publicação"
date: 2026-01-01 09:00:00 -0300
description: "Descrição breve e factual."
tags: [clara-news, inteligência artificial]
# image: /assets/images/nome-da-imagem.jpg
```

Envolva o YAML em delimitadores `---` no arquivo. Não é preciso definir `layout`, autor ou categorias. `image` é opcional; o campo adicional `image_alt` pode ser usado para uma descrição acessível mais específica. Sem ele, usa-se o título. Use caminhos locais iniciados em `/assets/` ou URLs HTTPS para imagens externas.

Os temas de `tags` são exibidos como **Categorias no rodapé visível do artigo**, depois do conteúdo. Se `categories` também existir, seus valores serão reunidos e deduplicados. A tag literal `clara-news` faz a publicação aparecer tanto na seção da homepage quanto em `/clara-news/`. Clara News é uma seção, não uma atribuição automática de autoria. Nenhum autor é inventado ou preenchido pelo layout.

O blog nasce sem artigos publicados. O modelo fica fora da build por `exclude`; não existe notícia ou áudio de demonstração publicado. Datas futuras e rascunhos não aparecem na build normal. Datas no feed usam RFC 822; na interface, dia/mês/ano. O fuso do site é `America/Sao_Paulo`.

### Áudio HTML e WhatsApp

Markdown e HTML podem coexistir nos posts. O modelo inclui um bloco comentado com `<audio controls preload="none">`, fonte MP3, download e transcrição. Só ative com arquivo real em `assets/audio/` (crie a pasta quando necessário), transcrição revisada e direitos de uso. Não há autoplay, player externo nem geração automática de áudio. Não há arquivos de áudio neste projeto.

O rodapé de cada post já oferece o canal no WhatsApp. Para links adicionais no corpo, use `{{ site.whatsapp_url }}`. O endereço é configurado em `_config.yml`:

https://www.whatsapp.com/channel/0029VaiPYBPLo4heVf0U3u2d

### Estrutura

- `_config.yml`: identidade, URLs, fuso, defaults e exclusões.
- `_layouts/default.html`, `home.html`, `post.html`: estrutura sem JavaScript obrigatório.
- `_includes/post-card.html`: listagem reutilizável.
- `index.html`, `sobre.html`, `clara-news.html`, `404.html`: páginas públicas.
- `assets/css/style.css`: estrutura dos componentes, responsividade, foco e impressão.
- `assets/css/whitney-ssm.css`: fontes Whitney SSm licenciadas, Twilio Sans Mono e Buffalo, servidas localmente.
- `assets/css/theme.css`: tokens fáceis de editar para fonte, paleta Twilio, escala H1–H6 e raio de 4 px.
- `assets/css/newsletter.css`: apresentação responsiva do formulário Listmonk da Clara News.
- `_data/navigation.yml`: links do menu principal, editáveis sem tocar no HTML.
- `assets/images/monogram.svg`, `connections.svg`, `ana-clara-portrait.svg`, `clara-news-continuity.svg`: SVGs originais locais.
- `GUIA-DE-ATUALIZACAO.md`: instruções para atualizar tudo diretamente pelo GitHub.
- `robots.txt`, `sitemap.xml`, `llms.txt`: descoberta por buscadores, redes sociais e agentes de IA.
- `feed.xml`: RSS 2.0 em Liquid, até 20 publicações, sem plugin extra.
- `templates/post.md`: modelo editorial excluído da build.

### Acessibilidade e privacidade

HTML semântico, idioma pt-BR, link de pular conteúdo, foco visível, mídia responsiva, redução de movimento e indicações de nova aba. Os arquivos licenciados Whitney SSm são servidos pelo próprio domínio, com Helvetica Neue, Helvetica e Arial como fallbacks. O site usa Google Analytics e um JavaScript local mínimo para o menu móvel. O canal WhatsApp é um link externo e segue as políticas do próprio serviço.

### Validação desta entrega

A build Jekyll é validada pelo GitHub Pages após cada publicação. O estado da build e a página pública devem ser conferidos antes de considerar uma alteração concluída.
