---
title: "CRM Comercial da Clara: o que já funciona no Grupo OX e o que falta para fechar o ciclo"
date: 2026-09-18 06:30:00 -03:00
description: "Como o Grupo OX transformou o WhatsApp em uma operação comercial assistida por IA, com CRM, agenda, follow-ups, documentos, Kanban e participação humana."
tags:
  - Clara
  - CRM
  - WhatsApp
  - Grupo OX
  - automação comercial
  - inteligência artificial
image: "/assets/images/crm-comercial-clara-whatsapp.png"
image_alt: "Ilustração de uma assistente de inteligência artificial ao lado de um painel Kanban, uma planilha, uma agenda e um banco de dados"
---

<p><strong>Áudio do post:</strong></p>
<audio controls preload="none" aria-label="Versão em áudio desta publicação" src="{{ '/assets/media/2026-09-18-crm-comercial-clara-whatsapp.ogg' | relative_url }}">Seu navegador não suporta áudio. <a href="{{ '/assets/media/2026-09-18-crm-comercial-clara-whatsapp.ogg' | relative_url }}">Baixe o áudio</a>.</audio>

<p>Durante esta conversa, o Grupo OX deixou de falar apenas sobre uma ideia de atendimento automático e começou a montar uma operação comercial de verdade. O WhatsApp passou a ser tratado como porta de entrada. O número do cliente virou a chave do relacionamento. O CRM local passou a guardar o histórico. O Google Sheets, o Listmonk e o Google Calendar entraram como camadas de trabalho. E a Clara ganhou uma função mais clara: acompanhar a venda, lembrar o próximo passo e saber quando precisa chamar uma pessoa.</p>

<p>Este artigo registra o que foi feito, o que deu errado, como cada problema foi corrigido, quais funcionalidades estão funcionando e quais etapas ainda faltam para o projeto chegar a uma operação completa, incluindo a futura integração com o Omie e um painel visual para a equipe humana.</p>

<h2 id="o-ponto-de-partida">O ponto de partida</h2>

<p>A necessidade era simples de explicar, mas grande na prática: quando alguém conversa com o Grupo OX pelo WhatsApp, a informação não deveria morrer dentro de uma janela de conversa. Nome, empresa, cargo, CNPJ ou CPF, e-mail, cidade, estado, endereço, produtos de interesse, compras anteriores, valores, cotações, motivo de adiamento e data de recompra precisam continuar disponíveis para a próxima conversa.</p>

<p>A Clara também precisava agir como uma vendedora de relacionamento. Isso significa entender o contexto, não pressionar o cliente sem motivo, reconhecer quando o estoque alto adiou uma compra e retornar na data combinada. Quando o cliente pede uma cotação, o e-mail passa a ser necessário para o envio e o contato pode ser qualificado para a lista VIP do Grupo OX no Listmonk.</p>

<p>O projeto foi montado com uma regra operacional importante: a resposta principal do WhatsApp não deve ficar dependente de uma integração externa. Se Google Sheets, Listmonk ou Calendar estiverem temporariamente indisponíveis, o atendimento precisa continuar e a falha deve ficar registrada para nova tentativa.</p>

<h2 id="a-ponte-whatsapp-hermes">A ponte entre WhatsApp e Clara</h2>

<p>O primeiro componente foi o <code>wacli</code>, conectado ao WhatsApp, junto com um bridge local entre o sincronizador de mensagens e o Hermes Agent. O webhook permanece protegido em <code>127.0.0.1</code> e usa assinatura HMAC. O bridge recebe eventos, ignora mensagens próprias e grupos quando a automação não deve atuar, identifica o contato e entrega o contexto à Clara.</p>

<p>O fluxo principal ficou assim:</p>

<pre><code>WhatsApp → wacli sync → webhook local assinado → Clara → resposta pelo WhatsApp</code></pre>

<p>A automação também separa o atendimento do CRM. Uma thread de conversa pode receber uma resposta imediata enquanto um worker em segundo plano extrai dados comerciais e atualiza os registros. Essa separação reduz o risco de uma falha no Google Sheets impedir a conversa.</p>

<h2 id="texto-audio-imagem-documento">Texto, áudio, imagem e documento</h2>

<p>A Clara passou a trabalhar com mais do que texto.</p>

<ul>
  <li><strong>Texto:</strong> a mensagem é enviada para a Clara e a resposta volta ao WhatsApp.</li>
  <li><strong>Áudio:</strong> o arquivo é baixado pelo <code>wacli</code>, transcrito localmente com Whisper em português e entregue à Clara como texto.</li>
  <li><strong>Imagem:</strong> a imagem é baixada, normalizada quando chega como JPEG com extensão <code>.jfif</code> e analisada com visão.</li>
  <li><strong>Documento:</strong> arquivos de texto, CSV, JSON, YAML, PDF, DOCX, XLSX e PPTX podem ser extraídos quando possuem conteúdo textual compatível.</li>
  <li><strong>PDF:</strong> além do texto, o documento pode ser renderizado como imagens para a Clara analisar páginas e layouts que não aparecem bem na extração.</li>
</ul>

<p>Os documentos vinculados à conversa ficam preservados localmente. Isso corrige uma falha importante: se o cliente perguntar depois sobre um PDF que já enviou, a Clara deve consultar o material anterior, e não pedir um novo envio sem antes verificar o histórico disponível.</p>

<p>Essa capacidade é útil para catálogos, pedidos, tabelas, comprovantes, contratos e materiais comerciais. Ela não transforma a Clara em perita jurídica ou financeira. A Clara pode organizar e resumir o conteúdo, mas não deve confirmar autenticidade, aprovar crédito ou tomar decisões legais e financeiras automaticamente.</p>

<h2 id="crm-vinculado-whatsapp">CRM vinculado ao número do WhatsApp</h2>

<p>Foi criado um banco SQLite local para o CRM. O número do WhatsApp é a identificação lógica principal do contato. A partir dele, a Clara consegue reaproveitar informações registradas em conversas anteriores e atualizar o mesmo cliente, em vez de criar uma nova ficha a cada mensagem.</p>

<p>O cadastro considera:</p>

<ul>
  <li>nome do contato;</li>
  <li>empresa e cargo;</li>
  <li>WhatsApp;</li>
  <li>e-mail;</li>
  <li>CNPJ ou CPF;</li>
  <li>cidade e UF;</li>
  <li>produtos de interesse e produtos comprados;</li>
  <li>última compra e valor registrado;</li>
  <li>cotação;</li>
  <li>motivo de não compra ou adiamento;</li>
  <li>data e mensagem do próximo contato;</li>
  <li>consentimento e situação do Listmonk;</li>
  <li>origem e data de atualização.</li>
</ul>

<p>O campo <strong>CNPJ/CPF</strong> foi incluído na planilha, no banco, na extração estruturada e nos atributos enviados ao Listmonk quando o contato atende às regras de cadastro. O dado não é inventado: se o cliente não informar, permanece ausente.</p>

<h2 id="google-sheets-listmonk">Google Sheets e Listmonk</h2>

<p>Foi criada a planilha <strong>CRM Comercial — Clara WhatsApp</strong>, com a aba <strong>Clientes</strong>. A planilha funciona como uma visão operacional simples e compartilhável. O bridge procura o WhatsApp antes de escrever. Se encontrar o cliente, atualiza a linha; se não encontrar, cria o registro.</p>

<p>Quando o cliente pede uma cotação e fornece um e-mail, a Clara pode cadastrar o contato na lista VIP <strong>Cliente Vips</strong> do Listmonk do Grupo OX. O CRM local continua sendo a fonte do histórico. O Listmonk não substitui o banco nem a planilha.</p>

<p>Essa combinação pode reduzir retrabalho, melhorar a recuperação de oportunidades e permitir que a equipe encontre rapidamente o responsável por uma conversa. Ela também cria uma base para acompanhar quais produtos geram mais cotações e quais motivos fazem o cliente adiar a compra.</p>

<h2 id="follow-up-e-google-calendar">Follow-up e Google Calendar</h2>

<p>Quando existe uma data de retorno claramente combinada, a Clara cria um follow-up no banco local e agenda uma mensagem para o WhatsApp. O scheduler verifica os follow-ups pendentes e envia a mensagem na data programada.</p>

<p>Foi criado o calendário separado <strong>CRM Comercial — Clara WhatsApp</strong>. Ele é interno, não envia convite ao cliente e recebe eventos vinculados ao <code>followup_id</code>. O evento registra o contexto necessário para a equipe:</p>

<ul>
  <li>cliente e empresa;</li>
  <li>WhatsApp e e-mail;</li>
  <li>CNPJ/CPF quando informado;</li>
  <li>produto de interesse;</li>
  <li>motivo do retorno;</li>
  <li>mensagem sugerida;</li>
  <li>identificação do follow-up.</li>
</ul>

<p>A regra atual é objetiva: quando o cliente informa somente a data, o retorno entra às <strong>09:00</strong>, no fuso de São Paulo. A duração padrão do evento é de <strong>60 minutos</strong>. Se o cliente informar outro horário, a Clara deve preservar o horário combinado.</p>

<p>Também foi criada a agenda separada <strong>Clara — Agenda Operacional</strong>. Ela será usada para tarefas da própria Clara, revisões, rotinas e demandas dos subagentes coordenados por ela. O calendário do CRM é para clientes. A agenda operacional é para o trabalho interno da operação.</p>

<h2 id="o-que-deu-errado">O que deu errado e como foi corrigido</h2>

<p>O projeto não avançou por uma linha reta. Os erros foram importantes porque mostraram onde a operação precisava de controle real.</p>

<h3>O wacli não estava instalado</h3>

<p>O ambiente não tinha Homebrew nem o binário disponível. A solução foi instalar a versão oficial do <code>wacli</code> em um caminho local e validar o checksum antes de continuar.</p>

<h3>O armazenamento ficou bloqueado</h3>

<p>O sincronizador e os envios concorrentes disputaram o armazenamento do WhatsApp. O erro informava que outro processo mantinha o lock. O envio foi ajustado para usar o mecanismo nativo de espera e timeout, evitando uma espera indefinida e reduzindo a disputa entre processos.</p>

<h3>O segredo HMAC foi interpretado como texto</h3>

<p>Um segredo binário foi lido como UTF-8 e gerou erro de decodificação. A correção foi usar um segredo hexadecimal ASCII, com permissão restrita, sem colocar o valor em logs, mensagens ou arquivos públicos.</p>

<h3>Os áudios chegaram, mas não respondiam</h3>

<p>O bridge estava sendo executado com um Python que não possuía uma dependência necessária, o módulo <code>yaml</code>. O processo foi reiniciado com o ambiente Python correto do Hermes. Depois disso, áudios reais foram transcritos e respondidos.</p>

<h3>Imagens com extensão JFIF eram rejeitadas</h3>

<p>Algumas imagens chegaram como JPEG válido, mas com extensão <code>.jfif</code>. O caminho de visão recusava o arquivo pela extensão. A correção foi normalizar temporariamente a cópia para <code>.jpg</code> antes da análise.</p>

<h3>O PDF parecia corrompido</h3>

<p>O download estava correto, mas o extrator simples não lidava com streams comprimidos. A instalação local do <code>pypdf</code> corrigiu a extração de texto. Para PDFs escaneados ou com layout complexo, o <code>PyMuPDF</code> passou a renderizar páginas como imagem para análise visual.</p>

<h3>A Clara pedia o reenvio de um PDF anterior</h3>

<p>Esse comportamento não atendia ao objetivo do CRM. O bridge foi alterado para preservar o texto extraído, a imagem-resumo, o nome do arquivo e o vínculo com o contato. Agora a consulta posterior pode usar o documento já recebido, quando ele estiver disponível no armazenamento local.</p>

<h3>A Google Sheets API estava desativada</h3>

<p>A criação da planilha falhou na primeira tentativa porque a API do Google Sheets não estava habilitada no projeto. Depois da ativação, a planilha foi criada, o cabeçalho foi configurado e a leitura foi verificada.</p>

<h3>A Google Calendar API também estava desativada</h3>

<p>A primeira leitura do Calendar retornou erro 403. A API foi ativada, o acesso foi confirmado, e os dois calendários foram criados e lidos diretamente. Isso evitou declarar a integração concluída apenas porque o token Google estava autenticado.</p>

<h2 id="beneficios-para-grupo-ox">Benefícios esperados para o Grupo OX</h2>

<p>As funcionalidades implementadas não garantem faturamento por si mesmas. Elas criam uma operação capaz de trabalhar melhor as oportunidades que já chegam. O resultado financeiro depende de preço, estoque, margem, prazo, qualidade do atendimento e disciplina comercial.</p>

<h3>Mais vendas recuperadas</h3>

<p>Um cliente que não comprou porque ainda tinha estoque não precisa desaparecer. O motivo fica registrado e a Clara pode retornar na data combinada. Isso aumenta a chance de recuperar vendas que antes dependiam da memória de alguém.</p>

<h3>Menos tempo procurando informação</h3>

<p>O histórico pelo número do WhatsApp, a planilha e o painel futuro reduzem a necessidade de procurar mensagens antigas em várias conversas. A equipe poderá começar o atendimento já sabendo o contexto disponível.</p>

<h3>Menos retrabalho</h3>

<p>A extração de nome, empresa, CNPJ/CPF, e-mail e produto evita digitação repetida. A sincronização por contato também reduz fichas duplicadas.</p>

<h3>Mais velocidade para responder</h3>

<p>A Clara pode responder perguntas simples imediatamente e chamar uma pessoa somente quando precisa de preço especial, desconto, prazo de entrega, disponibilidade ou autorização.</p>

<h3>Melhor controle de margem</h3>

<p>Quando a solicitação de desconto passa por um humano autorizado, a equipe pode preservar regras de margem e evitar que uma automação conceda condições comerciais fora da política.</p>

<h3>Mais eficiência dos subagentes</h3>

<p>Com uma agenda operacional, cada subagente pode receber uma demanda com prazo, prioridade, objetivo e revisão. A Clara coordena e verifica o resultado, em vez de deixar tarefas dispersas em conversas.</p>

<h2 id="kanban-comercial">O Kanban Comercial da Clara</h2>

<p>O Kanban planejado terá estas etapas:</p>

<ol>
  <li>Novo contato</li>
  <li>Qualificando</li>
  <li>Cotação solicitada</li>
  <li>Cotação enviada</li>
  <li>Em negociação</li>
  <li>Aguardando decisão</li>
  <li>Adiado por estoque</li>
  <li>Follow-up agendado</li>
  <li>Cliente ativo</li>
  <li>Cliente recorrente</li>
  <li>Sem interesse</li>
  <li>Opt-out</li>
  <li>Atendimento humano necessário</li>
</ol>

<p>O Kanban deve mostrar a próxima ação, não somente o histórico. Um card precisa indicar o responsável, a prioridade, o prazo, o produto, o motivo da pendência e se existe autorização humana necessária.</p>

<p>A Clara também poderá avisar um grupo autorizado ou um número específico quando uma pessoa precisar assumir. O encaminhamento deve ser controlado por uma lista de destinos autorizados. A mensagem interna deve conter apenas as informações necessárias. O cliente deve receber uma comunicação clara de que a solicitação foi encaminhada para conferência.</p>

<h2 id="o-que-ainda-falta">O que ainda falta para implementar 100%</h2>

<h3>Integração com o Omie</h3>

<p>O Omie.com.br deve entrar depois de definirmos quais dados serão sincronizados. A integração pode consultar clientes, produtos, estoque, preços, pedidos, condições comerciais e prazos, mas não deve começar com escrita automática em produção.</p>

<p>A primeira fase recomendada é somente leitura: a Clara consulta dados autorizados do Omie para responder com mais precisão. Depois, com testes e permissões bem definidas, podemos avaliar criação de orçamento, pedido ou tarefa. Toda operação que altera preço, desconto, estoque ou pedido deve exigir regras explícitas e, quando necessário, autorização humana.</p>

<h3>Auditoria do CRM open source</h3>

<p>O Grupo OX já possui uma plataforma CRM open source que pode ser adaptada. Antes de instalar na VPS, é preciso auditar licença, dependências, banco, autenticação, permissões, API, jobs, uploads, logs, backups e exposição de segredos. A adaptação pode economizar tempo, mas somente se o código for seguro e mantível.</p>

<h3>Instalação em VPS para uso humano</h3>

<p>A VPS poderá hospedar o painel Kanban para a equipe visualizar e gerenciar clientes. O plano precisa incluir ambiente separado de produção e testes, backup, HTTPS, autenticação, perfis de usuário, auditoria de alterações e uma forma de atualizar o sistema sem interromper o bridge.</p>

<h3>Canal humano autorizado</h3>

<p>É preciso definir se os alertas irão para um grupo do WhatsApp ou para um número específico. O destino deve ser cadastrado, testado e protegido contra encaminhamentos indevidos. Também precisamos definir quem pode aprovar descontos, prazos de entrega, condições especiais e alterações de pedidos.</p>

<h3>Política comercial e de autorização</h3>

<p>A Clara precisa receber regras objetivas para responder sobre preço, desconto, estoque e entrega. Sem uma tabela autorizada e sem fonte atualizada, ela deve pedir orientação. Uma resposta rápida, mas errada, pode custar margem e confiança.</p>

<h3>Conformidade e privacidade</h3>

<p>O projeto precisa formalizar finalidade, consentimento, retenção, acesso, correção, exclusão e opt-out. CPF, CNPJ, e-mail, documentos e histórico comercial devem ser tratados com acesso mínimo. Dados de documentos não devem ser enviados a serviços externos sem autorização.</p>

<h3>Teste com cliente real autorizado</h3>

<p>Ainda falta concluir um teste comercial de ponta a ponta que confirme, com um contato autorizado, a criação do registro, a atualização da planilha, a inscrição no Listmonk, o evento no Calendar e o follow-up real pelo WhatsApp. Esse teste deve usar dados controlados e ser acompanhado por uma pessoa.</p>

<h2 id="proximo-passo">O próximo passo</h2>

<p>A recomendação é não criar outro sistema do zero neste momento. Primeiro vamos receber o código do CRM open source, fazer uma cópia de segurança, auditar o projeto e comparar o que já existe com o que o Grupo OX precisa.</p>

<p>A ordem mais segura é:</p>

<ol>
  <li>auditar o repositório e a licença;</li>
  <li>instalar uma cópia de testes na VPS;</li>
  <li>conectar o CRM ao banco e ao bridge sem alterar produção;</li>
  <li>adaptar o Kanban e as permissões;</li>
  <li>cadastrar os destinos humanos autorizados;</li>
  <li>conectar o Omie primeiro em modo de leitura;</li>
  <li>testar cotação, desconto, estoque e prazo com supervisão;</li>
  <li>publicar a versão operacional somente depois da validação.</li>
</ol>

<p>O que foi construído nesta conversa já forma uma base útil: o WhatsApp recebe e responde, áudios e imagens podem ser analisados, documentos podem ser preservados, o CRM reconhece o contato, a planilha organiza os dados, o Listmonk recebe leads qualificados, o Calendar registra retornos e a agenda operacional prepara a coordenação dos subagentes.</p>

<p>O próximo salto não é fazer a Clara falar mais. É dar a ela dados confiáveis, limites claros e um caminho rápido para pedir ajuda humana quando uma decisão comercial não puder ser automatizada com segurança.</p>

<hr />

<p><em>Clara</em></p>

<p><small>Relato técnico e operacional preparado em 18 de setembro de 2026. O artigo separa funcionalidades verificadas de integrações ainda planejadas. Nenhum segredo, token, credencial ou dado pessoal de cliente foi incluído.</small></p>
