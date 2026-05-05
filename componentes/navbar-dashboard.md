# navbar-dashboard

## Descrição

Barra de navegação superior fixa do dashboard. Contém: logo "Oikos BPO", separador, nome e CNPJ do cliente, conjunto de abas de navegação entre páginas (Resumo, Contas pagas, A pagar, Contratos, Mútuos, DRE, Balanço), seletor de mês com setas anterior/próximo, exibição do mês ativo e botão de sair. Extraído diretamente do `<div class="topbar">` e das funções `sp()` e `cm()` do arquivo original.

---

## HTML

```html
<div class="topbar">
  <!-- Logo -->
  <span class="logo">Oikos <span>BPO</span></span>

  <!-- Separador vertical -->
  <div class="sep"></div>

  <!-- Nome e CNPJ do cliente -->
  <span class="cli" id="cli-nome">TO Family Holding &mdash; CNPJ 63.307.198/0001-70</span>

  <!-- Abas de navegação -->
  <div class="nav">
    <button class="nb on" onclick="sp('resumo', this)">Resumo</button>
    <button class="nb"    onclick="sp('contas', this)">Contas pagas</button>
    <button class="nb"    onclick="sp('apagar', this)">A pagar</button>
    <button class="nb"    onclick="sp('contratos', this)">Contratos</button>
    <button class="nb"    onclick="sp('mutuos', this)">Mútuos</button>
    <button class="nb"    onclick="sp('dre', this)">DRE</button>
    <button class="nb"    onclick="sp('balanco', this)">Balanço</button>
  </div>

  <!-- Separador + seletor de mês -->
  <div class="sep"></div>
  <div class="mrow-inline">
    <button class="mbtn" onclick="cm(-1)" title="Mês anterior">&#8249;</button>
    <span class="mlabel" id="ml-nav"></span>
    <button class="mbtn" onclick="cm(1)"  title="Próximo mês">&#8250;</button>
  </div>

  <!-- Botão sair -->
  <button class="btn-sair" onclick="sair()">Sair</button>
</div>
```

> **Nota:** O HTML original tem o seletor de mês repetido dentro de cada `.page`. A variante acima centraliza o seletor na topbar, o que é recomendado para novos portais. Caso prefira o padrão original (seletor por página), remova `mrow-inline` e `btn-sair` da topbar e use o bloco `.mrow` dentro de cada página.

---

## CSS

```css
/* ── Topbar ── */
.topbar {
  background: var(--bg1);
  border-bottom: 1px solid rgba(255,255,255,0.07);
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 0 20px;
  position: relative;
  z-index: 10;
  height: 52px;
  flex-shrink: 0;
}

/* ── Logo ── */
.logo {
  font-size: 13px;
  font-weight: 600;
  letter-spacing: .04em;
  white-space: nowrap;
  color: var(--t1);
}
.logo span {
  color: var(--accent); /* --ac no original: #7c6ff7 */
}

/* ── Separador ── */
.sep {
  width: 1px;
  height: 18px;
  background: rgba(255,255,255,0.13);
  flex-shrink: 0;
}

/* ── Nome do cliente ── */
.cli {
  font-size: 11px;
  color: var(--t2);
  white-space: nowrap;
}

/* ── Navegação ── */
.nav {
  display: flex;
  gap: 2px;
  margin-left: auto;
}

/* Botão de aba */
.nb {
  padding: 6px 13px;
  border-radius: 8px;
  border: none;
  background: none;
  color: var(--t2);
  font-size: 11px;
  font-family: 'DM Sans', sans-serif;
  cursor: pointer;
  transition: all .15s;
  white-space: nowrap;
}
.nb:hover { background: var(--bg3, #22222c); color: var(--t1); }
.nb.on    { background: var(--accent); color: #fff; font-weight: 500; }

/* ── Seletor de mês inline na topbar ── */
.mrow-inline {
  display: flex;
  align-items: center;
  gap: 6px;
  flex-shrink: 0;
}

/* Versão original: seletor por página (dentro de .page) */
.mrow {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 18px;
}

/* Botão de seta */
.mbtn {
  background: var(--bg2);
  border: 1px solid rgba(255,255,255,0.07);
  color: var(--t2);
  width: 28px;
  height: 28px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all .15s;
  flex-shrink: 0;
}
.mbtn:hover { background: var(--bg3, #22222c); color: var(--t1); }

/* Label do mês */
.mlabel {
  font-size: 15px;
  font-weight: 500;
  min-width: 110px;
  text-align: center;
  color: var(--t1);
}

/* ── Botão sair ── */
.btn-sair {
  padding: 5px 12px;
  border-radius: 8px;
  border: 1px solid rgba(255,255,255,0.07);
  background: none;
  color: var(--t2);
  font-size: 11px;
  font-family: 'DM Sans', sans-serif;
  cursor: pointer;
  transition: all .15s;
  white-space: nowrap;
  flex-shrink: 0;
}
.btn-sair:hover { background: var(--red); color: #fff; border-color: var(--red); }

/* ── Responsivo ── */
@media (max-width: 900px) {
  .cli { display: none; }
  .sep { display: none; }
  .nav { gap: 1px; }
  .nb  { padding: 5px 8px; font-size: 10px; }
}
```

---

## JavaScript

```js
// ── Navegação entre páginas ──────────────────────────────────────
/**
 * Alterna a página visível e destaca a aba ativa.
 * @param {string} id  - ID da seção (sem prefixo "page-"), ex.: 'resumo'
 * @param {HTMLElement} btn - Elemento botão clicado
 */
function sp(id, btn) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('on'));
  document.querySelectorAll('.nb').forEach(b => b.classList.remove('on'));
  document.getElementById('page-' + id).classList.add('on');
  btn.classList.add('on');
  document.querySelector('.content').scrollTop = 0;
}

// ── Navegação de mês ─────────────────────────────────────────────
let mesAtual  = 4;    // mês inicial (1–12)
let anoAtual  = 2026; // ano inicial

const M = ['Jan','Fev','Mar','Abr','Mai','Jun','Jul','Ago','Set','Out','Nov','Dez'];

function lbMes(m, a) { return M[m - 1] + ' ' + a; }

/**
 * Avança ou recua o mês e re-renderiza todas as páginas.
 * @param {number} d - Direção: -1 para mês anterior, +1 para próximo
 */
function cm(d) {
  mesAtual += d;
  if (mesAtual > 12) { mesAtual = 1;  anoAtual++; }
  if (mesAtual < 1)  { mesAtual = 12; anoAtual--; }
  renderAll();
}

/**
 * Atualiza todos os labels de mês na tela.
 * IDs esperados: ml-r, ml-c, ml-a, ml-d, ml-nav (topbar)
 */
function atualizarLabels() {
  ['r', 'c', 'a', 'd', 'nav'].forEach(id => {
    const el = document.getElementById('ml-' + id);
    if (el) el.textContent = lbMes(mesAtual, anoAtual);
  });
}

// ── Botão sair ───────────────────────────────────────────────────
function sair() {
  sessionStorage.clear();
  window.location.href = '/login.html'; // ajuste para a URL de login do cliente
}
```

---

## Exemplo de uso

```html
<!-- Shell do dashboard -->
<div class="shell">
  <!-- Topbar (navbar) -->
  <div class="topbar">
    <span class="logo">Oikos <span>BPO</span></span>
    <div class="sep"></div>
    <span class="cli" id="cli-nome">TO Family Holding &mdash; CNPJ 63.307.198/0001-70</span>
    <div class="nav">
      <button class="nb on" onclick="sp('resumo', this)">Resumo</button>
      <button class="nb"    onclick="sp('contas', this)">Contas pagas</button>
      <!-- ... demais abas ... -->
    </div>
    <div class="sep"></div>
    <div class="mrow-inline">
      <button class="mbtn" onclick="cm(-1)">&#8249;</button>
      <span class="mlabel" id="ml-nav"></span>
      <button class="mbtn" onclick="cm(1)">&#8250;</button>
    </div>
    <button class="btn-sair" onclick="sair()">Sair</button>
  </div>

  <!-- Conteúdo das páginas abaixo -->
  <div class="content"> ... </div>
</div>
```

---

## Variáveis a substituir

| Elemento | Descrição |
|---|---|
| `#cli-nome` (texto) | Nome do cliente e CNPJ exibido na topbar |
| Textos dos botões `.nb` | Nomes das abas (podem ser adicionadas/removidas conforme módulos do cliente) |
| Atributos `onclick="sp('ID', this)"` | IDs das seções devem corresponder a `id="page-ID"` no HTML |
| `mesAtual` / `anoAtual` | Mês e ano inicial exibidos ao carregar o portal |
| `/login.html` na função `sair()` | URL de redirecionamento pós-logout |
