# Prompt completo — Painel de Preventivas de Ciclo (Backbone & Acesso)

Crie um painel HTML autônomo (um único arquivo, sem backend) para acompanhamento de ciclos de preventiva de rede de **Backbone** e **Acesso**, com importação de mapas KMZ/KML. Ele deve abrir direto no navegador ou ser hospedado como página estática (GitHub Pages), no mesmo padrão de outras ferramentas "Jarvis MCLL".

## Identidade visual (obrigatório) — padrão profissional/corporativo
- Paleta: navy `#182238`, verde `#42C34B`, azul `#1476C9` (usar âmbar `#F2A93B` e vermelho `#E5484D` para alertas). Cor é sinal (status/elemento/origem), nunca decoração — nenhuma cor fora dessa paleta aparece na interface, incluindo estados de hover/foco (usar variações de opacidade/luminosidade das mesmas cores, não tons novos).
- Fonte: Calibri (fallback Segoe UI/Helvetica/Arial). Hierarquia tipográfica fixa e a mesma em todo o painel: título de seção 18–20px/600, subtítulo/rótulo de KPI 12–13px/500 (letter-spacing leve, geralmente maiúsculo), número de KPI 28–32px/700, corpo de tabela/texto 13–14px/400. Nenhum texto usa peso >700 nem itálico decorativo.
- Grid de espaçamento único (múltiplos de 4px: 4/8/12/16/24/32) aplicado a paddings, gaps e margens em todo o painel — nada de valores soltos tipo 10px/15px/22px. Alinhamento consistente: todo bloco de conteúdo alinha à mesma margem esquerda/direita da sidebar e da barra de ações.
- Componentes com linguagem única e sóbria em todo o painel (sem variar estilo por seção):
  - Bordas de 1px em cor neutra (não sombra) para separar tabelas, cards e a barra de filtros; cantos com raio pequeno e igual em todos os elementos (4–6px).
  - Botões: só duas variantes — primário (fundo azul `#1476C9`, texto branco) e secundário (contorno neutro, texto navy) — mesma altura, mesmo raio, mesmo padding horizontal em todos os botões do painel.
  - Tabelas densas: linhas com altura fixa e compacta, cabeçalho com peso 600 e fundo levemente diferenciado (não colorido), hover de linha sutil (leve mudança de fundo, sem sombra), zebra opcional só em tons neutros.
  - Estados de hover/foco/disabled definidos uma única vez e reutilizados em todos os elementos interativos (botões, links de menu, linhas de tabela, campos de filtro) — sem inconsistência entre seções.
- Rodapé fixo com o texto "Jarvis MCLL · alloha FIBRA".
- Layout **moderno e slim**: pouco padding, tipografia compacta, tabelas densas — evitar "cara de dashboard genérico de IA" (nada de sombra em todo card, nada de barrinha colorida lateral em KPI, nada de emoji decorativo em título/aba/mensagem, nada de gradiente chamativo, nada de ícone decorativo sem função, nada de animação/transição vistosa — no máximo transições de 100–150ms em hover/estado). Referência de tom: dashboard interno de NOC/operações de telecom, não landing page de produto.

## Layout / navegação
- **Menu lateral fixo (sidebar navy), recolhível** à esquerda, com:
  - Botão de recolher/ocultar (ícone, sem emoji) na barra de topo, que esconde a sidebar por completo (largura para 0, sem deixar rastro) e mostra uma aba estreita fixa na borda esquerda da tela para reabri-la. Estado (aberto/oculto) lembrado entre sessões (`localStorage`).
  - Logo/marca no topo (iniciais + nome da ferramenta + "Jarvis MCLL · alloha FIBRA").
  - Item "Mapa" (mostra só o mapa em tela cheia).
  - Item "Painel", com sub-itens sempre visíveis abaixo dele: Alertas de vencimento, Distribuição por status, % de cumprimento por UF, Trechos, Marcadores (CEO/CTO/CTOE). Clicar num sub-item enquanto está na aba Mapa já troca automaticamente para o Painel.
- Área de conteúdo (à direita da sidebar):
  - Barra fixa no topo só com os botões de ação: Backup/Restaurar, Exportar CSV, Registrar preventiva, Importar KMZ/KML. O botão "Registrar preventiva" da barra superior abre um modal que primeiro pede para buscar/selecionar o trecho (mesmo campo de busca por nome dos filtros) e então abre o mesmo modal de registro (data, técnico, ocorrências, observações) usado pela ação equivalente na linha do trecho, dentro da tabela "Trechos".
  - Abaixo: banner de aviso (dados de exemplo / bibliotecas não carregadas), barra de filtros (UF, Praça, Tipo, Status, busca por nome — sem caixa/sombra, só uma linha divisória) e o conteúdo da seção ativa.
  - No Painel: KPIs em uma única faixa horizontal com divisores finos entre colunas e números grandes coloridos (não usar cards separados por métrica nem barrinha lateral).
  - No Mapa: alternador **Colorir por status** (padrão) / **Colorir por arquivo KMZ** no cabeçalho do card do mapa, com legenda dinâmica quando o modo "arquivo KMZ" estiver ativo.

## Importação de KMZ/KML
- Aceitar `.kmz` (zip) e `.kml` puro, com drag-and-drop ou seleção de arquivo.
- **Linhas (LineString) viram trechos** de rede: nome, tipo (backbone/acesso — detectar por palavra-chave no nome ou campo customizado, default "acesso"), UF, praça, extensão em km (calcular automaticamente por geometria/Haversine se não vier no arquivo), ciclo de preventiva em dias (default 30 para backbone, 60 para acesso).
- **Pontos (Point) só viram marcadores quando identificados como CEO, CTO ou CTOE** — pela tag `<name>` do ponto ou por dados estendidos (`elemento`/`tipo_elemento`/`equipamento`/`tipo`). Se o nome/tag do ponto contiver mais de uma palavra-chave ao mesmo tempo (ex.: um ponto nomeado "CTO-CEO-014"), classificar pela palavra-chave de **maior prioridade** encontrada, na ordem **CTOE > CEO > CTO** (ex.: no caso acima, vira CEO), ignorando as demais palavras-chave presentes. **Qualquer ponto que não bata com esses três na importação é ignorado** (não vira marcador, não aparece em lugar nenhum do painel) — a importação não oferece nem cria categoria "outro". Internamente, o campo `elemento` do marcador deve aceitar qualquer valor de texto (não só os três), pois backups antigos podem conter marcadores legados fora desse conjunto — ver seção "Marcadores CEO/CTO/CTOE" sobre como esses casos são exibidos.
- Antes de confirmar a importação, mostrar uma prévia editável (tabela) de todos os trechos e marcadores detectados, permitindo corrigir tipo/UF/praça/ciclo/elemento linha a linha.
- Limite prático de importação: se um único arquivo gerar mais de 500 trechos ou 500 marcadores, mostrar um aviso (não bloqueante) na prévia informando que volumes grandes podem degradar a performance do mapa e do `localStorage`; suporte formal a arquivos maiores (paginação, etc.) fica registrado em "Fora de escopo".
- **Cada arquivo importado recebe uma cor própria** (paleta cíclica de ~12 cores), guardada em cada trecho/marcador resultante (id do lote, cor, nome do arquivo). No modo "Colorir por arquivo KMZ" do mapa, trechos e marcadores usam essa cor em vez da cor de status/elemento, com legenda dinâmica listando cada arquivo, sua cor e quantidade de itens. Dados sem origem (ex.: exemplo/manual) aparecem em cinza como "Sem origem / manual".
- Resiliência: se as bibliotecas externas (Leaflet/Chart.js/JSZip, carregadas via CDN) não carregarem — comum em rede corporativa/antivírus —, mostrar um aviso claro no topo do painel em vez de quebrar a página; o resto do painel (tabelas, KPIs) deve continuar funcionando mesmo sem mapa/gráficos.

## Marcadores CEO/CTO/CTOE
- Sempre exibidos no mapa com **rótulo colorido permanente** (não só no hover) — uma "placa" pequena com o texto do elemento (CEO roxo, CTO azul, CTOE verde-azulado).
- Se algum marcador de dado legado tiver um elemento fora desses três (situação que não deve mais ocorrer na importação, mas pode existir em backups antigos), ele deve aparecer no mapa só como **um ponto colorido sem texto** (sem placa/título) — diferenciação apenas por cor.
- Listados numa seção própria do Painel ("Marcadores CEO/CTO/CTOE"), com filtro por elemento (CEO/CTO/CTOE) e por UF/praça (reaproveitando os filtros do topo), tabela e modal de edição/exclusão.

## Mapa
- Leaflet + OpenStreetMap, com **zoom pelo scroll do mouse** habilitado.
- Trechos (linhas) e marcadores (pontos) plotados juntos, coloridos por status/elemento (padrão) ou por arquivo de origem (alternável), ambos clicáveis para abrir detalhe.

## KPIs e Painel
- KPIs: total de trechos monitorados, % em dia, km monitorados, vencendo em ≤7 dias, vencidos, ocorrências nos últimos 30 dias.
- Status de trecho: Em dia (verde) / Atenção ≤7 dias para vencer (âmbar) / Vencido (vermelho) — calculado a partir da última preventiva registrada + ciclo em dias.
- Seção "Alertas de vencimento": lista de trechos vencidos + vencendo nos próximos 14 dias.
- Seção "Distribuição por status": gráfico de rosca (Chart.js).
- Seção "% de cumprimento por UF": gráfico de barras por UF, onde % de cumprimento = (trechos com status "Em dia" ÷ total de trechos daquela UF) × 100 — trechos em "Atenção" e "Vencido" contam contra o percentual.
- Seção "Trechos": tabela completa com filtro (UF/praça/tipo/status/busca) e ordenação por coluna, exportável em CSV, com ação por linha para editar os dados cadastrais do trecho (nome/tipo/UF/praça/km/ciclo) e para excluí-lo (com confirmação, removendo também seu histórico de execuções).
- Registro de execução de preventiva: modal com data, técnico, quantidade de ocorrências (número inteiro ≥ 0, representando problemas encontrados/corrigidos naquela visita), observações (texto livre), e histórico por trecho. O KPI "ocorrências nos últimos 30 dias" soma esse número em todas as execuções registradas no período, de todos os trechos.

## Persistência
- Tudo salvo em `localStorage` do navegador (sem backend/sincronização entre computadores).
- Botão de backup (baixar JSON com trechos + execuções + marcadores + lotes de importação/cores) e de restauração (subir JSON).
- Ao abrir pela primeira vez, carregar dados de exemplo claramente marcados como "(exemplo)" (6 trechos + 4 marcadores fictícios cobrindo MA/PA/PI/AP/AM), com botão para limpar antes de importar dados reais.

## Fora de escopo por enquanto (registrar como pendência, não implementar sem confirmar)
- Anexo de fotos por execução de preventiva (limite de tamanho do localStorage).
- Definir se o ciclo é por trecho inteiro ou por segmento dentro da rota.
- SLA/tolerância de atraso antes de virar "crítico" (hoje é imediato ao passar do prazo).
- Exportação CSV dedicada para marcadores (hoje só trechos têm export).
- Paleta de cores por KMZ repete depois de ~12 arquivos importados — avisar se for cenário real de uso.
- Suporte formal a importações grandes (acima de 500 trechos/marcadores por arquivo): paginação da prévia, importação em lote assíncrona, etc. — hoje só existe o aviso não bloqueante descrito em "Importação de KMZ/KML".
