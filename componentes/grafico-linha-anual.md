# grafico-linha-anual

## Descrição

Gráfico de linha anual usando Chart.js 4.x com segmentação visual entre meses realizados (linha sólida) e meses projetados (linha tracejada). Implementado na aba "A Pagar" do dashboard original para mostrar a evolução de desembolsos ao longo do ano, com até três datasets sobrepostos (ativo fixo, despesas operacionais, mútuo/distribuição). A diferença realizado vs. projetado é controlada por um flag `isPast` em cada ponto de dado usando a API `segment` do Chart.js 4.

---

## HTML

```html
<div class="card">
  <div class="ct">Evolução mensal 2026 — projeção de desembolsos</div>

  <!-- Legenda manual acima do gráfico -->
  <div class="leg-wrap">
    <span class="leg-item">
      <span class="leg-sq" style="background: var(--accent)"></span>Ativo fixo
    </span>
    <span class="leg-item">
      <span class="leg-sq" style="background: #4ecdc4"></span>Despesas operacionais
    </span>
    <span class="leg-item">
      <span class="leg-sq" style="background: var(--yellow)"></span>Mútuo / distribuição
    </span>
  </div>

  <!-- Canvas do gráfico -->
  <div class="cw" style="height: 250px">
    <canvas id="c-evo2026" role="img" aria-label="Evolução mensal 2026 de desembolsos"></canvas>
  </div>
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

/* ── Legenda manual ── */
.leg-wrap {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin-bottom: 14px;
}

.leg-item {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 11px;
  color: var(--t2);
}

.leg-sq {
  width: 10px;
  height: 10px;
  border-radius: 2px;
  flex-shrink: 0;
}
```

---

## JavaScript

```js
// ── Dependência: Chart.js 4.x ────────────────────────────────────
// <script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>

const M = ['Jan','Fev','Mar','Abr','Mai','Jun','Jul','Ago','Set','Out','Nov','Dez'];
const charts = {};

function dc(id) {
  if (charts[id]) { charts[id].destroy(); delete charts[id]; }
}

/**
 * Renderiza gráfico de linha anual com segmento realizado (sólido) vs projetado (tracejado).
 *
 * Fluxo:
 * 1. Para cada mês de Jan a Dez do ano, combina lançamentos pagos e a pagar.
 * 2. Calcula totais por categoria: ativos, opex, mútuo/distribuição.
 * 3. Marca o mês como realizado (isPast = true) se já existem pagamentos registrados.
 * 4. Usa a API `segment` do Chart.js 4 para aplicar borderDash nos segmentos futuros.
 *
 * @param {string}   canvasId  - ID do <canvas>
 * @param {number}   ano       - Ano a renderizar (ex.: 2026)
 * @param {number}   mesAtual  - Mês corrente para separar realizado vs projetado
 * @param {Array}    pagas     - Array de lançamentos pagos { dt, cat, v }
 * @param {Array}    aPagar    - Array de lançamentos a pagar { dt, cat, v }
 */
function renderGraficoLinhaAnual(canvasId, ano, mesAtual, pagas, aPagar) {
  // ── Filtra lançamentos por mês e ano ──────────────────────────
  function doMes(arr, m, a) {
    return arr.filter(p => {
      const d = new Date(p.dt + 'T12:00:00');
      return d.getMonth() + 1 === m && d.getFullYear() === a;
    });
  }

  // ── Categorias de classificação ───────────────────────────────
  const isAtivo = cat => ['Ativo fixo', 'Compra de ativo fixo'].includes(cat);
  const isOpex  = cat => ['Serviços', 'Serviços contratados', 'Aluguel e condomínio',
                          'Aluguel/Cond.', 'Outras despesas', 'Taxas'].includes(cat);
  const isMut   = cat => ['Mútuo', 'Pagamento de mútuo', 'Distribuição',
                          'Distribuição de dividendos', 'Pagto. empréstimo'].includes(cat);

  // ── Monta dados para cada mês ─────────────────────────────────
  const meses = [];
  for (let m = 1; m <= 12; m++) {
    const pg_m  = doMes(pagas,  m, ano);
    const ap_m  = doMes(aPagar, m, ano);
    const all   = [...pg_m, ...ap_m];

    // Mês realizado = já passou OU é o mês atual com pagamentos registrados
    const isPast = (m < mesAtual) || (m === mesAtual && pg_m.length > 0);

    const soma = (arr, fn) => arr.filter(p => fn(p.cat)).reduce((s, p) => s + p.v, 0) || null;

    meses.push({
      lb: M[m - 1],
      isPast,
      ativos: soma(all, isAtivo),
      opex:   soma(all, isOpex),
      mut:    soma(all, isMut),
    });
  }

  // ── Cria o gráfico ────────────────────────────────────────────
  dc(canvasId);
  charts[canvasId] = new Chart(document.getElementById(canvasId), {
    type: 'line',
    data: {
      labels: meses.map(m => m.lb),
      datasets: [
        // Dataset 1: Ativo fixo — sólido quando realizado, tracejado quando projetado
        {
          label: 'Ativo fixo',
          data: meses.map(m => m.ativos),
          borderColor: '#7c6ff7',                        // --accent
          backgroundColor: 'rgba(124,111,247,0.08)',
          borderWidth: 2,
          pointRadius: 4,
          pointBackgroundColor: '#7c6ff7',
          fill: true,
          tension: 0.3,
          // API segment do Chart.js 4: aplica borderDash nos segmentos futuros
          segment: {
            borderDash: ctx => !meses[ctx.p1DataIndex]?.isPast ? [5, 4] : undefined,
          },
        },
        // Dataset 2: Despesas operacionais
        {
          label: 'Despesas operacionais',
          data: meses.map(m => m.opex),
          borderColor: '#4ecdc4',
          backgroundColor: 'rgba(78,205,196,0.05)',
          borderWidth: 2,
          pointRadius: 4,
          pointBackgroundColor: '#4ecdc4',
          fill: true,
          tension: 0.3,
          segment: {
            borderDash: ctx => !meses[ctx.p1DataIndex]?.isPast ? [5, 4] : undefined,
          },
        },
        // Dataset 3: Mútuo / distribuição — sempre tracejado (projeção)
        {
          label: 'Mútuo / distrib.',
          data: meses.map(m => m.mut),
          borderColor: '#fbbf24',                        // --yellow
          backgroundColor: 'rgba(251,191,36,0)',
          borderWidth: 2,
          borderDash: [5, 4],
          pointRadius: 4,
          pointBackgroundColor: '#fbbf24',
          fill: false,
          tension: 0.3,
        },
      ],
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      interaction: { mode: 'index', intersect: false },
      plugins: {
        legend: { display: false }, // usa legenda manual no HTML
        tooltip: {
          backgroundColor: '#111118',                    // --bg1
          borderColor: 'rgba(255,255,255,0.1)',
          borderWidth: 1,
          padding: 10,
          callbacks: {
            label: ctx => ctx.raw
              ? ' ' + ctx.dataset.label + ': R$ ' + Math.round(ctx.raw / 1000) + 'k'
              : null,
          },
        },
      },
      scales: {
        x: {
          ticks:  { color: '#4e4d60', font: { size: 10 }, autoSkip: false },
          grid:   { color: 'rgba(255,255,255,0.04)' },
        },
        y: {
          ticks: {
            color: '#4e4d60',
            font:  { size: 10 },
            callback: v => v >= 1_000_000
              ? 'R$' + (v / 1_000_000).toFixed(1) + 'M'
              : 'R$' + Math.round(v / 1_000) + 'k',
          },
          grid: { color: 'rgba(255,255,255,0.04)' },
        },
      },
    },
  });
}
```

---

## Exemplo de uso

```html
<!-- Dependência -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>

<!-- HTML -->
<div class="card">
  <div class="ct">Evolução mensal 2026 — projeção de desembolsos</div>
  <div class="leg-wrap">
    <span class="leg-item"><span class="leg-sq" style="background:#7c6ff7"></span>Ativo fixo</span>
    <span class="leg-item"><span class="leg-sq" style="background:#4ecdc4"></span>Despesas operacionais</span>
    <span class="leg-item"><span class="leg-sq" style="background:#fbbf24"></span>Mútuo / distribuição</span>
  </div>
  <div class="cw" style="height:250px">
    <canvas id="c-evo2026"></canvas>
  </div>
</div>

<!-- JS -->
<script>
  // mesAtual e anoAtual são as variáveis globais do dashboard
  renderGraficoLinhaAnual('c-evo2026', 2026, mesAtual, PAGAS, A_PAGAR);
</script>
```

---

## Variáveis a substituir

| Elemento | Descrição |
|---|---|
| `id="c-evo2026"` | ID único do canvas |
| `ano` (parâmetro) | Ano do gráfico — altere para o ano corrente do cliente |
| `mesAtual` | Variável global do dashboard; define a fronteira realizado/projetado |
| `PAGAS` / `A_PAGAR` | Arrays de dados do cliente; substitua pelas fontes de dados reais |
| Cores dos datasets | `#7c6ff7` (accent), `#4ecdc4` (teal), `#fbbf24` (yellow) — alinhados com `--accent`, `--yellow` |
| Categorias em `isAtivo` / `isOpex` / `isMut` | Ajuste os nomes de categoria conforme as categorizações do cliente |
| `height: 250px` | Altura do wrapper `.cw` |
| Textos da legenda manual | Atualize os labels conforme os datasets exibidos |
