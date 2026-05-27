# /arch-docs — Gera documentação interativa de arquitetura

Argumento opcional: `$ARGUMENTS` (pode ser vazio ou conter instruções extras do usuário, como "foco no backend" ou "inclua o fluxo de deploy").

## Objetivo

Gerar um **arquivo HTML único e auto-contido** com um diagrama interativo de arquitetura do projeto atual, usando Cytoscape.js. O resultado deve ser um artefato visual profissional, com:

- Grafo interativo com ícones reais das tecnologias
- Filtros por categoria
- Busca de nós
- Animação de fluxos
- Modo apresentação com slides
- Tema claro/escuro
- Export PNG
- Minimap
- Atalhos de teclado
- Painel de detalhes por nó

## Processo

### Fase 1 — Análise do projeto

Antes de gerar qualquer código, **analise o repositório atual** para entender a arquitetura real:

1. **Leia os arquivos de configuração**: `package.json`, `docker-compose.yml`, `Dockerfile`, `.env.example`, `tsconfig.json`, `Cargo.toml`, `go.mod`, `requirements.txt`, `pyproject.toml`, `Gemfile`, `pom.xml`, `build.gradle`, `Makefile`, etc.
2. **Identifique os serviços/módulos**: pastas em `apps/`, `packages/`, `services/`, `src/`, monorepo workspaces, microserviços
3. **Identifique dependências externas**: bancos de dados, caches, filas, APIs externas, serviços de terceiros, provedores de auth, gateways de pagamento, CDNs
4. **Mapeie conexões**: quem chama quem, relações HTTP/gRPC/WebSocket/queue, imports entre módulos
5. **Identifique fluxos**: fluxos de usuário principais (login, checkout, CRUD principal, deploy, CI/CD)
6. **Leia READMEs e docs existentes** para contexto adicional

**IMPORTANTE**: Baseie-se APENAS em fatos verificáveis no código. Não invente serviços, endpoints ou fluxos que não existem. Se algo não está claro, pergunte ao usuário.

### Fase 2 — Apresente o plano ao usuário

Antes de gerar o HTML, apresente um resumo ao usuário:

- Lista de nós (serviços/módulos) identificados, com categoria
- Lista de conexões entre eles
- Fluxos que pretende documentar
- Slides da apresentação
- Pergunte se falta algo ou se algo está errado

Só prossiga após confirmação.

### Fase 3 — Gerar o HTML

Gere o arquivo `{nome-do-projeto}-docs.html` no diretório raiz do projeto, seguindo **exatamente** o template abaixo.

## Template de referência

O arquivo gerado deve seguir esta estrutura (arquivo único, ~1500-2000 linhas):

### Dependências CDN (copiar exatamente)

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/styles/atom-one-dark.min.css">
<script src="https://cdn.jsdelivr.net/npm/cytoscape@3.28.1/dist/cytoscape.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/11.9.0/highlight.min.js"></script>
```

### Design system CSS (copiar e adaptar cores)

```css
:root {
  --bg: #06070d;
  --surface: rgba(14,17,30,0.82);
  --text: #e2e8f0;
  --muted: #7888a8;
  --blue: #60a5fa; --blue-bg: rgba(96,165,250,0.12);
  --green: #34d399; --green-bg: rgba(52,211,153,0.12);
  --yellow: #fbbf24; --yellow-bg: rgba(251,191,36,0.12);
  --purple: #a78bfa; --purple-bg: rgba(167,139,250,0.12);
  --pink: #f472b6; --pink-bg: rgba(244,114,182,0.12);
  --orange: #fb923c; --orange-bg: rgba(251,146,60,0.12);
  --r: 12px; --rs: 8px;
  --detail-w: 400px;
  --toolbar-h: 52px;
}
```

### Categorias de nós

Defina categorias baseadas no tipo de componente. Categorias padrão:

| Categoria   | Cor (dark)  | Cor (light) | Uso típico |
|-------------|-------------|-------------|------------|
| `frontend`  | `#1e3a5f`   | `#dbeafe`   | Apps web, mobile, SPAs |
| `api`       | `#1a3a2a`   | `#dcfce7`   | APIs, BFFs, gateways |
| `worker`    | `#3a2e1a`   | `#fef9c3`   | Workers, jobs, filas |
| `infra`     | `#2a1f4a`   | `#ede9fe`   | DBs, cache, storage |
| `external`  | `#3a1a2e`   | `#fce7f3`   | APIs externas, SaaS |

Adapte/adicione categorias se necessário (ex: `mobile`, `devops`, `ml`).

### Modelo de dados dos nós

```javascript
const D = {
  'node-id': {
    icon: '🔷',              // emoji representativo
    label: 'Nome do Serviço',
    sub: 'Descrição curta',
    cat: 'api',              // categoria (determina cor)
    badge: '🟢',             // status badge (opcional)
    badgeLabel: 'Ativo',
    desc: '<p>Descrição HTML do serviço...</p>',
    tech: ['Node.js', 'Express', 'TypeScript'],
    connects: ['outro-node-id'],  // lista de adjacência (gera edges)
    routes: [                     // rotas/endpoints (opcional)
      { m: 'GET', p: '/api/users', d: 'Lista usuários' }
    ],
    envs: [                       // variáveis de ambiente (opcional)
      { k: 'DATABASE_URL', d: 'Connection string do PostgreSQL' }
    ],
    jobs: [],       // jobs/cron (opcional)
    collections: [],// collections/tabelas (opcional, para DBs)
    queues: [],     // filas (opcional, para workers)
  },
};
```

### Ícones das tecnologias

Use ícones reais do Simple Icons (via SVG inline com base64):

```javascript
function _svg(s){ return 'data:image/svg+xml;base64,'+btoa(s); }
function _si(fill, d){ return _svg(`<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" fill="${fill}">${d}</svg>`); }
function _cx(d){ return _svg(`<svg viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">${d}</svg>`); }
```

Para cada tecnologia, busque o path SVG correto no Simple Icons (https://simpleicons.org/). Ícones comuns que você deve conhecer:

- **Next.js** (fill white em dark, #111827 em light)
- **NestJS** (#e0234e)
- **React** (#61DAFB)
- **Vue.js** (#4FC08D)
- **Angular** (#0F0F11)
- **Node.js** (#5FA04E)
- **Python** (#3776AB)
- **Go** (#00ADD8)
- **Rust** (#000000)
- **PostgreSQL** (#4169E1)
- **MongoDB** (#47A248)
- **Redis** (#FF4438)
- **Docker** (#2496ED)
- **Kubernetes** (#326CE5)
- **AWS** (#232F3E)
- **Google Cloud** (#4285F4)
- **Stripe** (#635BFF)
- **OpenAI** (fill white em dark, #111827 em light)

**IMPORTANTE sobre ícones brancos**: tecnologias com ícone branco (Next.js, OpenAI, etc.) precisam de um set alternativo para o tema claro. Defina `lightIconOverrides` com fill `#111827` para esses ícones.

### Posições dos nós

```javascript
const positions = {
  'node-id': { x: 110, y: 195 },
  // ...
};
```

Organize em colunas lógicas:
- **Coluna 1 (x~110)**: Frontend / clientes
- **Coluna 2 (x~410)**: APIs / backend
- **Coluna 3 (x~690)**: Infra / dados
- **Coluna 4 (x~950)**: Serviços externos

Espaçamento vertical: ~100-120px entre nós na mesma coluna.

### Configuração do Cytoscape

```javascript
const cy = cytoscape({
  container: document.getElementById('cy'),
  elements: elements,
  layout: { name: 'preset' },
  style: [
    { selector: 'node', style: {
      'width': 64, 'height': 64,
      'background-color': n => { /* function that reads theme */ },
      'border-width': 2,
      'border-color': n => { /* function that reads theme */ },
      'label': 'data(label)',
      'font-family': "'JetBrains Mono', monospace",
      'font-size': 11,
      'color': () => document.body.classList.contains('light') ? '#4a5578' : '#7888a8',
      'text-valign': 'bottom', 'text-margin-y': 8,
      'background-image': 'data(icon)',
      'background-fit': 'none',
      'background-width': '62%', 'background-height': '62%',
      'transition-property': 'opacity border-color background-color',
      'transition-duration': '0.25s',
    }},
    { selector: 'edge', style: {
      'width': 1.5,
      'line-color': '#1c2138',
      'target-arrow-color': '#1c2138',
      'target-arrow-shape': 'triangle',
      'curve-style': 'bezier',
      'arrow-scale': 0.7,
      'opacity': 0.6,
    }},
    // Classes de estado:
    { selector: '.hi', style: { 'border-color': '#60a5fa', 'border-width': 3 }},
    { selector: '.dim', style: { 'opacity': 0.15 }},
    { selector: '.hover-dim', style: { 'opacity': 0.12 }},
    { selector: '.flow-on', style: { 'border-color': '#fbbf24', 'border-width': 3 }},
    { selector: '.pres-dim', style: { 'opacity': 0.10 }},
    { selector: 'edge.flow-on', style: {
      'line-color': '#fbbf24', 'target-arrow-color': '#fbbf24',
      'width': 2.5, 'opacity': 1, 'z-index': 10
    }},
    { selector: 'edge.dim', style: { 'opacity': 0.06 }},
  ],
  minZoom: 0.3, maxZoom: 3,
  wheelSensitivity: 0.18,
  boxSelectionEnabled: false,
});
```

### PNG pre-render (OBRIGATÓRIO para zoom estável)

```javascript
(async () => {
  function renderSet(iconMap) {
    const out = {};
    return Promise.all(Object.entries(iconMap).map(([id, uri]) =>
      new Promise(done => {
        const img = new Image();
        img.onload = () => {
          const c = document.createElement('canvas');
          c.width = c.height = 256;
          c.getContext('2d').drawImage(img, 0, 0, 256, 256);
          out[id] = c.toDataURL('image/png');
          done(null);
        };
        img.onerror = done;
        img.src = uri;
      })
    )).then(() => out);
  }

  const [pngDark, pngLightExtra] = await Promise.all([
    renderSet(nodeIcons),
    renderSet(lightIconOverrides),
  ]);

  window.pngIconsDark  = pngDark;
  window.pngIconsLight = {...pngDark, ...pngLightExtra};

  const icons = document.body.classList.contains('light') ? window.pngIconsLight : window.pngIconsDark;
  cy.nodes().forEach(n => {
    const png = icons[n.id()];
    if (png) n.style('background-image', png);
  });
})();
```

### Features obrigatórias

Todas estas features devem estar no arquivo gerado:

1. **Filtros por categoria** — pills no toolbar, toggle `.dim` nos nós da categoria
2. **Busca** — input no toolbar, filtra nós por id e label
3. **Dropdown de fluxos** — dropdown customizado (NÃO `<select>` nativo), cada fluxo anima step-by-step com flowcard
4. **Flowcard** — card glassmórfico flutuante no bottom-center do grafo, mostra step atual com prev/next
5. **Painel de detalhes** — painel direito fixo (400px), tabs (Info, Rotas, Env, etc.), aparece ao clicar num nó
6. **Tooltip** — segue o mouse, mostra label + sub do nó
7. **Hover dim** — ao passar o mouse, destaca nó + vizinhos, escurece o resto
8. **Tema claro/escuro** — toggle completo, inclui swap de ícones brancos
9. **Zoom** — botões +/−/fit no canto inferior esquerdo
10. **Export PNG** — botão que baixa o grafo como PNG 2x
11. **Minimap** — preview do grafo no canto inferior direito, clicável para navegar
12. **Modo apresentação** — painel lateral que desliza da direita (52% width), slides com highlight de nós no grafo, auto-play com barra de progresso
13. **Atalhos de teclado** — `F`=fit, `T`=tema, `+/-`=zoom, `/`=busca, `Escape`=fechar, setas+espaço na apresentação

### Elementos HTML principais

```
body
├── #toolbar (fixed top, 52px)
│   ├── .brand (nome do projeto)
│   ├── .filter-pill[data-cat="..."] (um por categoria)
│   ├── #search-box > input#search
│   ├── #flow-dd (custom dropdown, NÃO <select>)
│   │   ├── #fdd-btn
│   │   └── #fdd-menu > .fdd-item (um por fluxo)
│   ├── #pres-enter-btn
│   ├── #theme-btn
│   └── #export-btn
├── #main (flex row)
│   ├── #graph-area (flex:1)
│   │   ├── #cy (Cytoscape container)
│   │   ├── #zctrl (zoom buttons, absolute bottom-left)
│   │   ├── #minimap (absolute bottom-right)
│   │   │   └── #minimap-vp (viewport indicator)
│   │   └── #flowcard (absolute bottom-center, glassmorphic)
│   │       ├── .fc-close
│   │       ├── #fp-title, #fp-step, #fp-desc
│   │       └── #fp-prev, #fp-next
│   └── #detail (400px fixed)
│       ├── #detail-scroll
│       │   ├── (welcome state — nav-items para cada nó)
│       │   └── (node detail state — tabs + panels)
├── #tip (tooltip, fixed position)
└── #pres-modal (presentation panel, fixed right)
    ├── .pres-header (title + close + auto-play)
    ├── #pres-slide-title, #pres-slide-sub
    ├── #pres-slide-body (desc + code block)
    ├── .pres-nav (prev/next + step indicator)
    └── .pres-progress > .pres-bar
```

### toggleTheme (IMPORTANTE)

```javascript
function toggleTheme(){
  const light = document.body.classList.toggle('light');
  document.getElementById('theme-btn').textContent = light ? '● Tema' : '◐ Tema';
  const edgeC = light ? '#c4cfe8' : '#1c2138';
  cy.edges().not('.flow-on').style({'line-color':edgeC,'target-arrow-color':edgeC});
  cy.nodes().updateStyle();
  document.getElementById('cy').style.background = light ? '#eef1f8' : '';
  const icons = light ? window.pngIconsLight : window.pngIconsDark;
  if (icons) cy.nodes().forEach(n => { const p = icons[n.id()]; if(p) n.style('background-image', p); });
}
```

### Cytoscape node style functions (IMPORTANTE)

As funções de estilo dos nós DEVEM ler o tema dinamicamente:

```javascript
'background-color': n => {
  const lt = document.body.classList.contains('light');
  return (lt ? catBgLight : catBg)[n.data('cat')] || (lt ? '#eef1f8' : '#0c0e1a');
},
'border-color': n => {
  const lt = document.body.classList.contains('light');
  return (lt ? catColorLight : catColor)[n.data('cat')];
},
'color': () => document.body.classList.contains('light') ? '#4a5578' : '#7888a8',
```

### Custom dropdown (NÃO usar <select> nativo)

```javascript
function fddToggle(){ document.getElementById('flow-dd').classList.toggle('open'); }
function fddSelect(val, label){
  document.getElementById('fdd-label').textContent = label;
  document.getElementById('flow-dd').classList.remove('open');
  startFlow(val);
}
document.addEventListener('click', e => {
  if(!e.target.closest('#flow-dd')) document.getElementById('flow-dd').classList.remove('open');
});
```

### Glassmorphism pattern

```css
.component {
  background: var(--surface);
  backdrop-filter: blur(24px);
  -webkit-backdrop-filter: blur(24px);
  border: 1px solid rgba(255,255,255,0.06);
  border-radius: var(--r);
}
```

## Regras

1. **Um único arquivo HTML** — sem dependências locais, tudo inline ou via CDN
2. **ZERO frameworks JS** — vanilla JS puro
3. **Baseado em fatos** — só documente o que existe no código; se não tem certeza, pergunte
4. **Ícones reais** — use Simple Icons SVG paths para cada tecnologia; NUNCA use emojis como ícones dos nós no grafo
5. **Tema completo** — dark e light, incluindo swap de ícones brancos
6. **Glassmorphism** — todos os componentes flutuantes
7. **Custom dropdown** — NUNCA `<select>` nativo
8. **PNG pre-render** — OBRIGATÓRIO para estabilidade de zoom
9. **Nomes em português** — labels, descrições, botões, tudo em PT-BR
10. **Use o arquivo de referência** `/home/mithrandir/zuke/zuke-docs.html` como base. Em caso de dúvida sobre implementação de qualquer feature, leia o arquivo de referência para copiar o padrão exato.

## Saída esperada

Arquivo `{nome-do-projeto}-docs.html` no diretório raiz do projeto, pronto para abrir no browser.
