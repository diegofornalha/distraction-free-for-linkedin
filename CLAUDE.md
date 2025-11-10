# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão Geral do Projeto

**Distraction Free for LinkedIn™** é uma extensão do Chrome (Manifest V3) que oculta elementos distrativos do LinkedIn para melhorar o foco do usuário. A extensão injeta CSS e JavaScript no site do LinkedIn para esconder feeds de notícias, anúncios, sugestões de conexões e outros elementos.

## Arquitetura

### Componentes Principais

1. **Content Script** (`extension/hider/hider.js`)
   - Injetado em todas as páginas `https://www.linkedin.com/*` via `run_at: "document_start"`
   - Responsável por:
     - Injetar stylesheet CSS tanto no frame principal quanto no preload iframe
     - Modificar o título da página (remove contadores de mensagens)
     - Substituir favicon (versão sem notificação)
     - Criar e gerenciar o toggle UI flutuante (master switch)

2. **Stylesheet** (`extension/hider/hider.css`)
   - Estratégia: usa `visibility: hidden` para elementos principais (mantém layout estável)
   - Usa `display: none` para anúncios e elementos que não devem ocupar espaço
   - Esconde: feed de notícias, sidebar direita, anúncios, sugestões de conexões, recomendações de vagas, promos de jogos do LinkedIn

3. **Popup** (`extension/popup/`)
   - UI simples usando Bulma CSS
   - Links para: reportar issues (GitHub), ver extensão no GitHub, gerenciar extensões
   - `popup.js`: gerencia cliques nos links e abre novas tabs

### Técnicas Importantes

#### Injeção de CSS em Múltiplos Frames
O código injeta CSS tanto no documento principal quanto em iframes de preload do LinkedIn:
- Frame principal: polling a cada 100ms até `document.head` existir
- Preload iframe: usa MutationObserver robusto para detectar criação/modificação de iframes

#### Master Switch Toggle
- Toggle flutuante fixo (top-left ou top-right, clicável para trocar lado)
- Estado persistido em memória (variável `showDfl`)
- Ativa/desativa stylesheets usando atributo `disabled`
- UI com animações CSS (transições suaves)
- Posição salva em `localStorage` (chave: `dfl_side`)

#### Observação de Mudanças no DOM
O código usa `MutationObserver` extensivamente para:
- Detectar quando LinkedIn remove/reconstrói partes da página (SPA)
- Re-injetar toggle se removido
- Monitorar mudanças no título e favicon

## Estrutura de Arquivos

```
extension/
├── manifest.json          # Manifest V3, permissions, content scripts
├── hider/
│   ├── hider.js          # Content script principal
│   ├── hider.css         # Regras de ocultação
│   └── favicon-no-messages.ico
├── popup/
│   ├── popup.html        # UI do popup
│   ├── popup.js          # Lógica do popup
│   ├── bulma.min.css     # Framework CSS
│   └── icons/            # SVGs para o popup
└── icons/                # Ícones da extensão (16, 32, 48, 128)

chrome-store/
└── archive/              # Screenshots para Chrome Web Store
```

## Desenvolvimento

### Carregar Extensão Localmente
1. Abra `chrome://extensions/`
2. Ative "Modo do desenvolvedor"
3. Clique em "Carregar sem compactação"
4. Selecione a pasta `extension/`

### Testar Mudanças
- **CSS**: Edite `hider.css` e recarregue a página do LinkedIn
- **JavaScript**: Edite `hider.js`, vá em `chrome://extensions/` e clique em "Atualizar" na extensão
- **Manifest**: Qualquer mudança no `manifest.json` requer recarregar a extensão

### Debugging
- Abra DevTools no LinkedIn para ver console logs e erros do content script
- Para o popup: clique-direito no ícone da extensão → "Inspecionar"
- Content script aparece em DevTools → Sources → Content Scripts

## Considerações Importantes

### Estratégia de Ocultação
- **`visibility: hidden`**: Para colunas/blocos principais (layout permanece estável)
- **`display: none`**: Para anúncios e elementos que não devem ocupar espaço

### Seletores CSS Específicos do LinkedIn
O LinkedIn muda frequentemente suas classes e estrutura. Principais padrões:
- `.scaffold-layout__aside` (sidebar direita)
- `.feed-shared-update-v2` (posts do feed)
- `[componentkey="..."]` (componentes identificados)
- `div[data-testid="mainFeed"]` (container principal do feed - atualização 2025)

### Performance
- Evite polling quando possível (preferir MutationObserver)
- Use `{ once: true }` em event listeners quando apropriado
- Intervalos de polling limitados e com `clearInterval` quando possível

### Compatibilidade
- Manifest V3 (Chrome moderno)
- Usa `@supports selector(:has(*))` para funcionalidades CSS modernas
- Fallbacks para navegadores sem suporte a `:has()`

## URLs de Referência

- **GitHub**: https://github.com/mthurmond/distraction-free-for-linkedin
- **Issues**: https://github.com/mthurmond/distraction-free-for-linkedin/issues
- **ID da Extensão**: `kigfnbfbpfpgphbocdkmeablbgdbpfke`
