# grafico-rosca

## Descrição

Gráfico de rosca (doughnut) usando Chart.js 4.x para visualização de distribuição proporcional (ex.: despesas por categoria, composição de ativos). Inclui legenda lateral ou superior com quadrados coloridos e rótulos, e suporta um resumo textual abaixo do gráfico listando cada categoria com valor. Extraído das funções `renderContas()` e `renderBalanco()` do arquivo original.

---

## HTML

```html
<!-- Variante 1: Rosca com legenda integrada (Chart.js position:right) -->
<div class="card">
  <div class="ct">Distribuição dos custos — mês</div>
  <div class="cw" style="height: 210px">
    <canvas id="c-cat" role="img" aria-label="Distribuição dos custos por categoria"></canvas>
  </div>
</div>

<!-- Variante 2: Rosca com legenda manual em tags + resumo textual -->
<div class="card">
  <div class="ct">Distribuição por categoria</div>

  <!-- Legenda manual (gerada via JS) -->
  <div class="leg-wrap" id="leg-cp"></div>

  <!-- Canvas do gráfico -->
  <div class="cw" style="height: 200px">
    <canvas id="c-cat-cp" role="img" aria-label="Contas pagas por categoria"></canvas>
  </div>

  <!-- Resumo textual por categoria (gerado via JS) -->
  <div id="resumo-cp" style="flex: 1; margin-top: 16px;"></div>
</div>
```

---

## CSS

```css
/* ── Wrapper responsivo do canvas ── */
.cw {
  position: relative;
  width: 100%;
}

/* ── Legenda manual (wrap de tags) ── */
.leg-wrap {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 14px;
}

/* Item da legenda */
.leg-item {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 11px;
  color: var(--t2);
}

/* Quadrado colorido da legenda */
.leg-sq {
  width: 10px;
  height: 10px;
  border-radius: 2px;
  flex-shrink: 0;
}

/* ── Ponto circular de categoria (usado no resumo textual) ── */
.cat-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  display: inline-block;
  margin-right: 5px;
  flex-shrink: 0;
  vertical-align: middle;
}
```

---

## JavaScript

```js
// ── Paleta de cores dos gráficos ─────────────────────────────────
const CORES = ['#7c6ff7','#4ecdc4','#fbbf24','#f87171','#34d399','#60a5fa','#a78bfa','#fb923c'];

// Cores por categoria (para gráfico de rosca de contas pagas)
const CAT_CORES = {
  'Ativo fixo':           '#7c6ff7',
  'Compra de ativo fixo': '#7c6ff7',
  'Serviços':             '#4ecdc4',
  'Serviços contratados': '#4ecdc4',
  'Aluguel e condomínio': '#60a5fa',
  'Aluguel/Cond.':        '#60a5fa',
  'Outras despesas':      '#fbbf24',
  'Taxas':                '#9ca3af',
  'Mútuo':                '#f87171',
  'Pagamento de mútuo':   '#f87171',
  'Distribuição':         '#fb923c',
  'Distribuição de dividendos': '#fb923c',
};

// ── Cache de instâncias Chart.js ─────────────────────────────────
const charts = {};

/**
 * Destrói uma instância Chart.js existente antes de recriar.
 * Necessário ao re-renderizar o gráfico com novos dados.
 */
function dc(id) {
  if (charts[id]) { charts[id].destroy(); delete charts[id]; }
}

/**
 * Formata valor com centavos — usado no tooltip e resumo textual.
 */
function fmtF(v) {
  return 'R$ ' + Math.abs(v).toLocaleString('pt-BR', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  });
}

// ══════════════════════════════════════════════════════════════════
// VARIANTE 1 — Rosca de ativos (legenda integrada à direita)
// Usada em: Resumo (distribuição de custos) e Balanço (composição)
// ══════════════════════════════════════════════════════════════════
/**
 * @param {string}   canvasId - ID do elemento <canvas>
 * @param {string[]} labels   - Rótulos das fatias
 * @param {number[]} valores  - Valores numéricos de cada fatia
 * @param {string[]} cores    - Array de cores hex (use CORES ou cores custom)
 */
function renderRoscaComLegenda(canvasId, labels, valores, cores) {
  dc(canvasId);
  if (!valores.length) return;

  charts[canvasId] = new Chart(document.getElementById(canvasId), {
    type: 'doughnut',
    data: {
      labels,
      datasets: [{
        data: valores,
        backgroundColor: cores,
        borderWidth: 0,
      }],
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      cutout: '62%',
      plugins: {
        legend: {
          position: 'right',
          labels: {
            color: '#8e8da0',     // --t2
            font: { size: 10 },
            boxWidth: 10,
          },
        },
      },
    },
  });
}

// ══════════════════════════════════════════════════════════════════
// VARIANTE 2 — Rosca com legenda manual + resumo textual
// Usada em: Contas Pagas (distribuição por categoria)
// ══════════════════════════════════════════════════════════════════
/**
 * @param {string}   canvasId    - ID do <canvas>
 * @param {string}   legendaId   - ID do container .leg-wrap
 * @param {string}   resumoId    - ID do container de resumo textual
 * @param {Object[]} lancamentos - Array de { cat, v }
 */
function renderRoscaCategorias(canvasId, legendaId, resumoId, lancamentos) {
  // Agrupa valores por categoria
  const cats = {};
  lancamentos.forEach(p => { cats[p.cat] = (cats[p.cat] || 0) + p.v; });

  const cl = Object.keys(cats);
  const cv = cl.map(k => cats[k]);
  const cc = cl.map(k => CAT_CORES[k] || '#888');
  const total = cv.reduce((s, v) => s + v, 0);

  // Legenda manual
  document.getElementById(legendaId).innerHTML = cl.map((k, i) => `
    <span class="leg-item">
      <span class="leg-sq" style="background:${cc[i]}"></span>
      ${k} — ${fmtF(cv[i])}
    </span>`).join('');

  // Gráfico
  dc(canvasId);
  if (cv.length) {
    charts[canvasId] = new Chart(document.getElementById(canvasId), {
      type: 'doughnut',
      data: {
        labels: cl,
        datasets: [{
          data: cv,
          backgroundColor: cc,
          borderWidth: 0,
          hoverOffset: 4,
        }],
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        cutout: '65%',
        plugins: {
          legend: { display: false },
          tooltip: {
            callbacks: {
              label: ctx => ' ' + ctx.label + ': ' + fmtF(ctx.raw),
            },
          },
        },
      },
    });
  }

  // Resumo textual
  const catRows = cl.map((k, i) => `
    <div style="display:flex;justify-content:space-between;align-items:center;
                padding:8px 0;border-bottom:1px solid rgba(255,255,255,0.07)">
      <span style="display:flex;align-items:center;font-size:12px">
        <span class="cat-dot" style="background:${cc[i]}"></span>${k}
      </span>
      <span style="font-family:'DM Mono',monospace;font-size:11px;font-weight:500">
        − ${fmtF(cv[i])}
      </span>
    </div>`).join('');

  document.getElementById(resumoId).innerHTML = catRows + `
    <div style="display:flex;justify-content:space-between;padding:10px 0 0;
                font-weight:600;font-size:12px">
      <span>Total</span>
      <span style="font-family:'DM Mono',monospace;color:var(--red)">
        − ${fmtF(total)}
      </span>
    </div>`;
}
```

---

## Exemplo de uso

```html
<!-- Dependência: Chart.js 4.x -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>

<!-- Variante 1: distribuição simples com legenda à direita -->
<div class="card">
  <div class="ct">Composição dos ativos</div>
  <div class="cw" style="height:240px">
    <canvas id="c-ativo"></canvas>
  </div>
</div>

<script>
  const nomes  = ['Flipping Industrial', 'Portucale 301', 'Epic 506', 'Salamanca'];
  const totais = [450000, 4200000, 3843771, 930000];
  renderRoscaComLegenda('c-ativo', nomes, totais, CORES);
</script>

<!-- Variante 2: distribuição de contas pagas com legenda manual e resumo -->
<div class="card">
  <div class="ct">Distribuição por categoria</div>
  <div class="leg-wrap" id="leg-cp"></div>
  <div class="cw" style="height:200px"><canvas id="c-cat-cp"></canvas></div>
  <div id="resumo-cp"></div>
</div>

<script>
  const pagasDoMes = PAGAS.filter(p => parseMes(p.dt).m === mesAtual);
  renderRoscaCategorias('c-cat-cp', 'leg-cp', 'resumo-cp', pagasDoMes);
</script>
```

---

## Variáveis a substituir

| Elemento | Descrição |
|---|---|
| `id="c-cat"` / `id="c-cat-cp"` | ID único do canvas por gráfico na página |
| `id="leg-cp"` | ID do container da legenda manual |
| `id="resumo-cp"` | ID do container do resumo textual |
| `cutout: '62%'` / `'65%'` | Espessura da rosca (quanto maior, mais fino o anel) |
| `CORES` | Paleta global de cores para gráficos sem mapa por categoria |
| `CAT_CORES` | Mapa categoria → cor para gráficos de contas pagas |
| `height: 210px` / `200px` | Altura do wrapper `.cw` — ajuste conforme layout |
| `legend.position: 'right'` | Posição da legenda integrada: `'right'`, `'bottom'`, `'top'` |
