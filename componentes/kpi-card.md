# kpi-card

## Descrição

Card de indicador-chave de performance (KPI) usado em todas as páginas do dashboard. Exibe um label descritivo (título pequeno em uppercase), um valor principal formatado e um subtítulo opcional. A cor do valor é customizável via `style` inline ou classe utilitária. Os cards são dispostos em grid responsivo usando a classe `.kgrid`.

---

## HTML

```html
<!-- Container grid (coloque todos os cards dentro) -->
<div class="kgrid" id="kpi-resumo">

  <!-- Card 1: valor neutro (cor padrão) -->
  <div class="kpi">
    <div class="kl">Total saídas</div>
    <div class="kv">R$ 432k</div>
    <div class="kd">Abr 2026</div>
  </div>

  <!-- Card 2: valor positivo (verde) -->
  <div class="kpi">
    <div class="kl">Entradas</div>
    <div class="kv" style="color: var(--green)">R$ 295k</div>
    <div class="kd">dividendos</div>
  </div>

  <!-- Card 3: valor condicional (verde se positivo, vermelho se negativo) -->
  <div class="kpi">
    <div class="kl">Resultado líquido</div>
    <div class="kv" style="color: var(--red)">− R$ 137k</div>
    <!-- sem subtítulo -->
  </div>

  <!-- Card 4: valor com cor accent (roxo) -->
  <div class="kpi">
    <div class="kl">Compra de ativos</div>
    <div class="kv" style="color: var(--accent)">R$ 167k</div>
    <!-- sem subtítulo -->
  </div>

  <!-- Card 5: valor amarelo (alerta/previsto) -->
  <div class="kpi">
    <div class="kl">Despesas operac.</div>
    <div class="kv" style="color: var(--yellow)">R$ 19k</div>
    <!-- sem subtítulo -->
  </div>

</div>
```

> O conteúdo dos cards é gerado dinamicamente via JavaScript no projeto original (ver seção JS abaixo). O HTML acima mostra a estrutura estática equivalente para facilitar reuso.

---

## CSS

```css
/* ── Grid de KPIs ── */
.kgrid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 10px;
  margin-bottom: 18px;
}

/* ── Card individual ── */
.kpi {
  background: var(--bg2);
  border: 1px solid rgba(255,255,255,0.07); /* --border original */
  border-radius: 12px;                       /* --r original */
  padding: 14px;
}

/* ── Label (título) ── */
.kl {
  font-size: 10px;
  color: var(--t2);           /* --t3 no original: #4e4d60 — ajustado para --t2 para maior legibilidade */
  text-transform: uppercase;
  letter-spacing: .06em;
  margin-bottom: 6px;
}

/* ── Valor principal ── */
.kv {
  font-size: 20px;
  font-weight: 500;
  font-variant-numeric: tabular-nums;
  color: var(--t1); /* cor padrão; sobrescreva com style inline para --green, --red, --accent, --yellow */
}

/* ── Subtítulo ── */
.kd {
  font-size: 10px;
  margin-top: 4px;
  color: var(--t2);
}
```

---

## JavaScript

```js
// ── Funções utilitárias de formatação (usadas em todos os cards) ──

/**
 * Formata valor monetário em formato compacto (k / M).
 * Ex.: 295000 → "R$ 295k" | 1500000 → "R$ 1,500M"
 */
function fmt(v) {
  if (!v && v !== 0) return '—';
  const a = Math.abs(v);
  if (a >= 1_000_000) return 'R$ ' + (v / 1_000_000).toFixed(3).replace(/\.?0+$/, '').replace('.', ',') + 'M';
  if (a >= 1_000)     return 'R$ ' + (v / 1_000).toFixed(0) + 'k';
  return 'R$ ' + Math.round(v).toLocaleString('pt-BR');
}

/**
 * Formata valor monetário com centavos (para tabelas e totais).
 * Ex.: 3110.44 → "R$ 3.110,44"
 */
function fmtF(v) {
  return 'R$ ' + Math.abs(v).toLocaleString('pt-BR', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2,
  });
}

// ── Exemplo: geração dinâmica de cards ──────────────────────────
/**
 * Renderiza o grid de KPIs do Resumo.
 * @param {number} total    - Total de saídas do mês
 * @param {number} totalRec - Total de entradas do mês
 * @param {number} ativos   - Valor investido em ativos fixos
 * @param {number} opex     - Despesas operacionais
 * @param {string} mesLabel - Ex.: "Abr 2026"
 */
function renderKpiResumo(total, totalRec, ativos, opex, mesLabel) {
  const saldo = totalRec - total;

  document.getElementById('kpi-resumo').innerHTML = `
    <div class="kpi">
      <div class="kl">Total saídas</div>
      <div class="kv">${fmt(total)}</div>
      <div class="kd">${mesLabel}</div>
    </div>
    <div class="kpi">
      <div class="kl">Entradas</div>
      <div class="kv" style="color:var(--green)">${fmt(totalRec)}</div>
      <div class="kd">dividendos</div>
    </div>
    <div class="kpi">
      <div class="kl">Resultado líquido</div>
      <div class="kv" style="color:${saldo >= 0 ? 'var(--green)' : 'var(--red)'}">${fmt(saldo)}</div>
    </div>
    <div class="kpi">
      <div class="kl">Compra de ativos</div>
      <div class="kv" style="color:var(--accent)">${fmt(ativos)}</div>
    </div>
    <div class="kpi">
      <div class="kl">Despesas operac.</div>
      <div class="kv" style="color:var(--yellow)">${fmt(opex)}</div>
    </div>
  `;
}
```

---

## Exemplo de uso

```html
<!-- 1. Adicione o container no HTML -->
<div class="kgrid" id="kpi-resumo"></div>

<!-- 2. No JS, chame após calcular os valores do mês -->
<script>
  const totalSaidas  = 432000;
  const totalEntradas = 295000;
  const ativos       = 167000;
  const opex         = 19000;

  renderKpiResumo(totalSaidas, totalEntradas, ativos, opex, 'Abr 2026');
</script>
```

```html
<!-- Uso estático (sem JS) — apenas HTML/CSS -->
<div class="kgrid">
  <div class="kpi">
    <div class="kl">Saldo devedor total</div>
    <div class="kv" style="color: var(--yellow)">R$ 7,778M</div>
    <div class="kd">portfólio imobiliário</div>
  </div>
  <div class="kpi">
    <div class="kl">Total desembolsado</div>
    <div class="kv" style="color: var(--green)">R$ 4,607M</div>
    <div class="kd">37,2% do portfólio</div>
  </div>
</div>
```

---

## Variáveis a substituir

| Elemento | Descrição |
|---|---|
| `id="kpi-resumo"` | ID do container — um por seção (ex.: `kpi-c`, `kpi-a`, `kpi-mut`, `kpi-b`) |
| Textos `.kl` | Labels dos indicadores (ex.: "Total saídas", "Compra de ativos") |
| Cor em `style="color:..."` | Use `var(--green)`, `var(--red)`, `var(--accent)`, `var(--yellow)` ou `var(--t1)` |
| Textos `.kd` | Subtítulo opcional — descrição contextual do valor |
| `minmax(140px, 1fr)` em `.kgrid` | Largura mínima de cada card — ajuste conforme quantidade de KPIs |
