# tabela-lancamentos

## Descrição

Tabela completa de lançamentos financeiros com colunas Data / Favorecido / Descrição / Categoria / Valor. Suporta entradas e saídas com cores distintas, badges de categoria com cor customizada por tipo, linha de total ao final e estado vazio quando não há lançamentos no mês. Extraída das funções `renderContas()` e `renderResumo()` do arquivo original.

---

## HTML

```html
<!-- Wrapper do card -->
<div class="card">
  <div class="ct">Lançamentos do mês</div>

  <!-- Wrapper com scroll horizontal em telas pequenas -->
  <div class="tw">
    <table id="t-lancamentos">
      <!-- Thead e tbody são gerados via JS -->
    </table>
  </div>
</div>
```

Estrutura HTML completa gerada pelo JS (referência estática):

```html
<table>
  <thead>
    <tr>
      <th>Data</th>
      <th>Favorecido</th>
      <th>Descrição</th>
      <th>Categoria</th>
      <th style="text-align:right">Valor</th>
    </tr>
  </thead>
  <tbody>

    <!-- Linha de saída (exemplo) -->
    <tr>
      <td style="color:var(--t2);font-family:'DM Mono',monospace;font-size:11px;white-space:nowrap">
        10/04
      </td>
      <td>José Adailton Pinheiro</td>
      <td style="color:var(--t2)">Portucale</td>
      <td>
        <span style="display:inline-flex;align-items:center">
          <span class="cat-dot" style="background:#7c6ff7"></span>
          <span class="bdg" style="background:#7c6ff722;color:#7c6ff7">Compra de ativo fixo</span>
        </span>
      </td>
      <td style="text-align:right;font-weight:500;font-family:'DM Mono',monospace">
        − R$ 75.000,00
      </td>
    </tr>

    <!-- Linha de entrada (exemplo — cor verde) -->
    <tr>
      <td style="color:var(--t2);font-family:'DM Mono',monospace;font-size:11px">07/04</td>
      <td>T.O. Medicina Especializada</td>
      <td style="color:var(--t2)">Recebimento de dividendos</td>
      <td>
        <span class="bdg bc">Recebimento de Dividendos</span>
      </td>
      <td style="text-align:right;font-weight:500;color:var(--green);font-family:'DM Mono',monospace">
        + R$ 100.000,00
      </td>
    </tr>

    <!-- Linha de total -->
    <tr class="tot-row">
      <td colspan="4">Total</td>
      <td style="text-align:right;color:var(--red);font-family:'DM Mono',monospace">
        − R$ 432.174,29
      </td>
    </tr>

    <!-- Estado vazio (quando não há lançamentos) -->
    <tr>
      <td colspan="5" style="color:var(--t2);text-align:center;padding:20px">
        Nenhum lançamento neste mês
      </td>
    </tr>

  </tbody>
</table>
```

---

## CSS

```css
/* ── Wrapper com scroll horizontal ── */
.tw {
  overflow-x: auto;
}

/* ── Table base ── */
table {
  width: 100%;
  border-collapse: collapse;
  font-size: 12px;
}

/* ── Cabeçalho ── */
thead th {
  color: var(--t2);             /* --t3 original: #4e4d60 */
  font-weight: 500;
  text-align: left;
  padding: 5px 8px;
  border-bottom: 1px solid rgba(255,255,255,0.07);
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: .05em;
}

/* ── Células ── */
tbody td {
  padding: 9px 8px;
  border-bottom: 1px solid rgba(255,255,255,0.07);
  color: var(--t1);
  vertical-align: middle;
}
tbody tr:last-child td {
  border-bottom: none;
}
tbody tr:hover td {
  background: var(--bg2);
}

/* ── Linha de total ── */
.tot-row td {
  border-top: 1px solid rgba(255,255,255,0.13) !important;
  border-bottom: none !important;
  font-weight: 600;
}

/* ── Badge de categoria ── */
.bdg {
  display: inline-block;
  padding: 2px 7px;
  border-radius: 20px;
  font-size: 10px;
  font-weight: 500;
  white-space: nowrap;
}

/* Classes de badge por tipo */
.bg { background: rgba(74,222,128,.12);  color: var(--green);  }  /* verde */
.ba { background: rgba(251,191,36,.12);  color: var(--yellow); }  /* amarelo */
.bb { background: rgba(124,111,247,.15); color: #b0a9f9;       }  /* roxo claro */
.bc { background: rgba(78,205,196,.15);  color: #4ecdc4;       }  /* teal */
.br { background: rgba(248,113,113,.12); color: var(--red);    }  /* vermelho */

/* ── Ponto colorido de categoria ── */
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
// ── Mapa de cores por categoria ──────────────────────────────────
const CAT_CORES = {
  'Ativo fixo':           '#7c6ff7',   // roxo (accent)
  'Compra de ativo fixo': '#7c6ff7',
  'Serviços':             '#4ecdc4',   // teal
  'Serviços contratados': '#4ecdc4',
  'Aluguel e condomínio': '#60a5fa',   // azul
  'Aluguel/Cond.':        '#60a5fa',
  'Outras despesas':      '#fbbf24',   // amarelo
  'Taxas':                '#9ca3af',   // cinza
  'Mútuo':                '#f87171',   // vermelho
  'Pagamento de mútuo':   '#f87171',
  'Distribuição':         '#fb923c',   // laranja
  'Distribuição de dividendos': '#fb923c',
  'Pagto. empréstimo':    '#9ca3af',
  'Mútuo/Distrib.':       '#f87171',
  'Rendimento aplicações':'#4ade80',   // verde claro
  'Recebimento de Dividendos': '#4ecdc4',
};

/**
 * Formata valor monetário com centavos.
 * Ex.: 75000 → "R$ 75.000,00"
 */
function fmtF(v) {
  return 'R$ ' + Math.abs(v).toLocaleString('pt-BR', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  });
}

/**
 * Renderiza a tabela de lançamentos no elemento informado.
 *
 * @param {string}  tblId    - ID do elemento <table>
 * @param {Array}   saidas   - Array de objetos { dt, nm, desc, cat, v }
 * @param {Array}   entradas - Array de objetos { dt, nm, desc, cat, v } (opcional)
 * @param {Object}  opts     - Opções { mostrarTotal: bool, corTotal: '--red'|'--yellow' }
 */
function renderTabelaLancamentos(tblId, saidas, entradas = [], opts = {}) {
  const { mostrarTotal = true, corTotal = 'var(--red)' } = opts;
  const tbl = document.getElementById(tblId);

  // Cabeçalho
  tbl.innerHTML = `
    <thead>
      <tr>
        <th>Data</th>
        <th>Favorecido</th>
        <th>Descrição</th>
        <th>Categoria</th>
        <th style="text-align:right">Valor</th>
      </tr>
    </thead>`;

  const tb = document.createElement('tbody');

  // Estado vazio
  if (!saidas.length && !entradas.length) {
    const tr = document.createElement('tr');
    tr.innerHTML = `<td colspan="5" style="color:var(--t2);text-align:center;padding:20px">
      Nenhum lançamento neste mês
    </td>`;
    tb.appendChild(tr);
    tbl.appendChild(tb);
    return;
  }

  // Combina e ordena por data
  const todos = [
    ...saidas.map(p  => ({ ...p, tipo: 'saida' })),
    ...entradas.map(p => ({ ...p, tipo: 'entrada' })),
  ].sort((a, b) => a.dt.localeCompare(b.dt));

  todos.forEach(p => {
    const cor = CAT_CORES[p.cat] || '#888';
    const isEntrada = p.tipo === 'entrada';
    const tr = document.createElement('tr');

    const sinal    = isEntrada ? '+ ' : '− ';
    const corValor = isEntrada ? 'color:var(--green)' : '';
    const badgeCls = isEntrada ? 'bc' : '';

    tr.innerHTML = `
      <td style="color:var(--t2);font-family:'DM Mono',monospace;font-size:11px;white-space:nowrap">
        ${p.dt.slice(8)}/${p.dt.slice(5,7)}/${p.dt.slice(0,4)}
      </td>
      <td>${p.nm}</td>
      <td style="color:var(--t2)">${p.desc}</td>
      <td>
        <span style="display:inline-flex;align-items:center">
          <span class="cat-dot" style="background:${cor}"></span>
          <span class="bdg ${badgeCls}" style="${badgeCls ? '' : `background:${cor}22;color:${cor}`}">
            ${p.cat}
          </span>
        </span>
      </td>
      <td style="text-align:right;font-weight:500;${corValor};font-family:'DM Mono',monospace">
        ${sinal}${fmtF(p.v)}
      </td>`;
    tb.appendChild(tr);
  });

  // Linha de total (apenas saídas)
  if (mostrarTotal && saidas.length) {
    const total = saidas.reduce((s, p) => s + p.v, 0);
    const trTot = document.createElement('tr');
    trTot.className = 'tot-row';
    trTot.innerHTML = `
      <td colspan="4">Total</td>
      <td style="text-align:right;color:${corTotal};font-family:'DM Mono',monospace">
        − ${fmtF(total)}
      </td>`;
    tb.appendChild(trTot);
  }

  tbl.appendChild(tb);
}
```

---

## Exemplo de uso

```html
<!-- HTML -->
<div class="card">
  <div class="ct">Lançamentos do mês</div>
  <div class="tw">
    <table id="t-lancamentos"></table>
  </div>
</div>
```

```js
// JS — após filtrar os dados do mês:
const saidas  = PAGAS.filter(p => parseMes(p.dt).m === mesAtual && parseMes(p.dt).a === anoAtual);
const entradas = RECEBIDAS.filter(p => parseMes(p.dt).m === mesAtual && parseMes(p.dt).a === anoAtual);

renderTabelaLancamentos('t-lancamentos', saidas, entradas, {
  mostrarTotal: true,
  corTotal: 'var(--red)',
});

// Apenas saídas sem total (ex.: tabela A Pagar):
renderTabelaLancamentos('t-apagar', aPagarDoMes, [], {
  mostrarTotal: true,
  corTotal: 'var(--yellow)',
});
```

---

## Variáveis a substituir

| Elemento | Descrição |
|---|---|
| `id="t-lancamentos"` | ID único da tabela por seção do dashboard |
| `CAT_CORES` | Mapa categoria → cor hex; adicione/remova categorias conforme o cliente |
| `corTotal` | Cor da linha de total: `var(--red)` para saídas, `var(--yellow)` para previstos |
| Colunas do `<thead>` | Ajuste os títulos conforme contexto (ex.: "Venc." em vez de "Data" para A Pagar) |
| Texto do estado vazio | "Nenhum lançamento neste mês" — personalize por contexto |
