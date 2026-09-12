# Guia rápido de atualização do blog

Tudo pode ser alterado pelo navegador, sem instalar programas:

1. Abra o repositório `anaclaraia/anaclaraia.github.io` no GitHub.
2. Entre no arquivo desejado e clique no ícone de lápis **Edit this file**.
3. Faça a alteração e clique em **Commit changes**.
4. Aguarde o GitHub Pages concluir a publicação.

## O que editar

- **Cores, fonte, tamanhos e bordas:** `assets/css/theme.css`
- **Estrutura e conteúdo da página inicial:** `_layouts/home.html`
- **Links do menu:** `_data/navigation.yml`
- **Nome, descrição, domínio e WhatsApp:** `_config.yml`
- **Página Sobre:** `sobre.html`
- **Novo artigo:** copie `templates/post.md` para `_posts/AAAA-MM-DD-slug.md`
- **Imagens:** envie para `assets/images/`
- **Áudios:** envie para `assets/media/`

## Tokens visuais

O início de `assets/css/theme.css` concentra toda a identidade visual:

```css
--font-twilio: "Whitney SSm", "Helvetica Neue", Helvetica, Arial, sans-serif;
--twilio-ink: #000d25;
--twilio-red: #db132a;
--twilio-blue: #1866ee;
--radius: 4px;
```

Altere uma variável para atualizar todos os componentes que a utilizam.

## Fonte Whitney SSm

Os arquivos licenciados estão hospedados em `assets/fonts/` e as regras `@font-face` ficam em `assets/css/whitney-ssm.css`. O arquivo é carregado no `<head>` antes de `style.css` e `theme.css`.

Para trocar ou atualizar as fontes:

1. envie os novos arquivos autorizados para `assets/fonts/`;
2. atualize os caminhos em `assets/css/whitney-ssm.css`;
3. mantenha `font-display: swap` para preservar o desempenho;
4. confirme a licença antes de adicionar ou substituir arquivos comerciais.

## Tamanhos dos títulos

Desktop:

- H1: 48 px / linha de 60 px
- H2: 40 px / linha de 52 px
- H3: 28 px / linha de 36 px
- H4: 20 px / linha de 32 px
- H5: 18 px / linha de 28 px
- H6: 16 px / linha de 24 px

Celular:

- H1: 32 px / linha de 44 px
- H2: 28 px / linha de 36 px
- H3: 24 px / linha de 32 px
- H4: 18 px / linha de 28 px
- H5: 16 px / linha de 24 px
- H6: 14 px / linha de 20 px

## Conferência antes de salvar

- Não apague os delimitadores `---` dos arquivos Markdown.
- Use nomes de arquivos sem espaços ou acentos.
- Confira links e caminhos de imagens.
- Não envie chaves, senhas ou tokens ao repositório.
- Verifique a publicação em `https://anaclaraia.github.io` após o commit.
