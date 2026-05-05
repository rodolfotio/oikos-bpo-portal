# timestamp-atualizacao

## Descrição

Exibição da data e hora da última atualização dos dados no formato "Dados de: DD/MM/AAAA às HH:MM". No arquivo original este dado aparece no `<div class="footer-note">` como texto estático ("Atualizado Abr/2026"). Este componente adapta e expande essa ideia para exibir um timestamp dinâmico, buscando a data do lançamento mais recente nos dados carregados ou, opcionalmente, consultando um endpoint da API para obter a data real da última sincronização com o ERP.

---

## HTML

```html
<!-- Variante 1: Rodapé fixo da página (padrão do HTML original) -->
<div class="footer-note" id="footer-ts">
  Oikos Consultoria de Valores Mobiliários — BPO TO Family Holding — Atualizado Abr/2026
</div>

<!-- Variante 2: Timestamp inline próximo ao seletor de mês ou ao topo da seção -->
<div class="ts-wrap">
  <span class="ts-label">Dados de:</span>
  <span class="ts-valor" id="ts-data">—</span>
</div>

<!-- Variante 3: Badge compacto na topbar -->
<span class="ts-badge" id="ts-badge" title="Última atualização dos dados"></span>
```

---

## CSS

```css
/* ── Variante 1: Rodapé (estilo original do HTML) ── */
.footer-note {
  text-align: center;
  font-size: 10px;
  color: var(--t2);              /* --t3 original: #4e4d60 */
  padding: 16px 0 0;
  font-family: 'DM Mono', monospace;
}

/* ── Variante 2: Timestamp inline ── */
.ts-wrap {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 10px;
  color: var(--t2);
  font-family: 'DM Mono', monospace;
  margin-bottom: 14px;           /* mesmo espaço que .mrow */
}

.ts-label {
  color: var(--t2);
  opacity: .7;
}

.ts-valor {
  color: var(--t1);
  font-weight: 500;
}

/* ── Variante 3: Badge compacto na topbar ── */
.ts-badge {
  font-size: 10px;
  color: var(--t2);
  font-family: 'DM Mono', monospace;
  white-space: nowrap;
  padding: 3px 8px;
  border-radius: 6px;
  background: var(--bg2);
  border: 1px solid rgba(255,255,255,0.07);
}
```

---

## JavaScript

```js
// ═══════════════════════════════════════════════════════════════════
// timestamp-atualizacao.js
// Lógica de busca e exibição do timestamp de atualização dos dados
// ═══════════════════════════════════════════════════════════════════

// ── Configuração ─────────────────────────────────────────────────
const API_URL = 'https://SEU_PROJETO.railway.app'; // URL do backend Railway

// ── Utilitários de formatação ────────────────────────────────────

/**
 * Formata uma data ISO (YYYY-MM-DD ou timestamp) para DD/MM/AAAA.
 * @param {string|Date} data
 * @returns {string} Ex.: "30/04/2026"
 */
function formatarData(data) {
  const d = data instanceof Date ? data : new Date(data + (data.length === 10 ? 'T12:00:00' : ''));
  return d.toLocaleDateString('pt-BR', { day: '2-digit', month: '2-digit', year: 'numeric' });
}

/**
 * Formata hora de um Date para HH:MM.
 * @param {Date} d
 * @returns {string} Ex.: "14:35"
 */
function formatarHora(d) {
  return d.toLocaleTimeString('pt-BR', { hour: '2-digit', minute: '2-digit' });
}

// ── Estratégia 1: Data do lançamento mais recente nos dados locais ──
/**
 * Determina a data de atualização como a data do lançamento mais recente
 * dentre todos os arrays de dados disponíveis.
 *
 * @param {...Array} arrays - Arrays de lançamentos { dt: 'YYYY-MM-DD' }
 * @returns {string} Texto formatado "DD/MM/AAAA" ou "—" se sem dados
 */
function tsDesDados(...arrays) {
  const todas = arrays.flat();
  if (!todas.length) return '—';

  const datas = todas
    .map(p => p.dt)
    .filter(Boolean)
    .sort()
    .reverse(); // mais recente primeiro

  return formatarData(datas[0]);
}

// ── Estratégia 2: Timestamp via API (última sincronização com ERP) ──
/**
 * Busca o timestamp de última atualização do backend e atualiza os elementos.
 *
 * Endpoints esperados (ajuste conforme sua API):
 *   GET /api/ultima-atualizacao?cliente=tofamily
 *   Resposta: { updated_at: "2026-04-30T21:15:00Z" }
 *
 * @param {string} clienteId - Identificador do cliente na API
 * @param {string[]} elementIds - IDs dos elementos a atualizar
 */
async function carregarTimestamp(clienteId, elementIds = ['ts-data', 'ts-badge', 'footer-ts']) {
  try {
    const token = sessionStorage.getItem('sb_access_token') || '';

    const res = await fetch(`${API_URL}/api/ultima-atualizacao?cliente=${clienteId}`, {
      headers: token ? { 'Authorization': `Bearer ${token}` } : {},
    });

    let texto;

    if (res.ok) {
      const data = await res.json();
      const dt = new Date(data.updated_at);
      texto = `Dados de: ${formatarData(dt)} às ${formatarHora(dt)}`;
    } else {
      // Fallback: usa data do lançamento mais recente nos dados locais
      // (PAGAS e RECEBIDAS devem estar disponíveis no escopo global)
      const dtLocal = tsDesDados(
        typeof PAGAS     !== 'undefined' ? PAGAS     : [],
        typeof RECEBIDAS !== 'undefined' ? RECEBIDAS : [],
      );
      texto = `Dados de: ${dtLocal}`;
    }

    // Atualiza todos os elementos configurados
    elementIds.forEach(id => {
      const el = document.getElementById(id);
      if (el) el.textContent = texto;
    });

  } catch {
    // Fallback silencioso — usa dados locais
    const dtLocal = tsDesDados(
      typeof PAGAS     !== 'undefined' ? PAGAS     : [],
      typeof RECEBIDAS !== 'undefined' ? RECEBIDAS : [],
    );
    const texto = `Dados de: ${dtLocal}`;
    elementIds.forEach(id => {
      const el = document.getElementById(id);
      if (el) el.textContent = texto;
    });
  }
}

// ── Estratégia 3: Timestamp estático (para portais sem backend) ──
/**
 * Define o timestamp diretamente a partir de uma string de data/hora.
 * Use quando o HTML for gerado estaticamente (ex.: export mensal).
 *
 * @param {string} dataISO - Ex.: "2026-04-30" ou "2026-04-30T21:15:00"
 */
function setTimestampEstatico(dataISO) {
  const d = new Date(dataISO.length === 10 ? dataISO + 'T12:00:00' : dataISO);
  const texto = dataISO.includes('T')
    ? `Dados de: ${formatarData(d)} às ${formatarHora(d)}`
    : `Dados de: ${formatarData(d)}`;

  ['ts-data', 'ts-badge', 'footer-ts'].forEach(id => {
    const el = document.getElementById(id);
    if (el) el.textContent = texto;
  });
}
```

---

## Exemplo de uso

```html
<!-- Rodapé da página (estilo original) -->
<div class="footer-note" id="footer-ts">
  Oikos Consultoria de Valores Mobiliários — BPO TO Family Holding
</div>

<!-- Timestamp inline acima da tabela -->
<div class="ts-wrap">
  <span class="ts-label">Dados de:</span>
  <span class="ts-valor" id="ts-data">—</span>
</div>
```

```js
// Opção A: Carrega da API (recomendado para portais com backend)
// Chamar após renderAll() no script principal
carregarTimestamp('tofamily', ['ts-data', 'footer-ts']);

// Opção B: Calcula a partir dos dados locais (sem fetch)
const dtAtual = tsDesDados(PAGAS, RECEBIDAS);
document.getElementById('ts-data').textContent = `Dados de: ${dtAtual}`;

// Opção C: Estático — para exports mensais gerados em batch
setTimestampEstatico('2026-04-30T18:00:00');

// No renderAll(), garanta que o timestamp é atualizado junto:
function renderAll() {
  atualizarLabels();
  renderResumo();
  renderContas();
  // ...
  carregarTimestamp('tofamily'); // ou use tsDesDados() para versão sem API
}
```

---

## Variáveis a substituir

| Elemento | Descrição |
|---|---|
| `API_URL` | URL do backend Railway com o endpoint `/api/ultima-atualizacao` |
| `clienteId` | Identificador do cliente na API (ex.: `'tofamily'`, `'cliente-abc'`) |
| `elementIds` | IDs dos elementos HTML que receberão o texto do timestamp |
| `/api/ultima-atualizacao` | Rota do endpoint — ajuste conforme a API do projeto |
| Texto do `.footer-note` | Nome do cliente e contexto — ex.: "BPO TO Family Holding" |
| `PAGAS` / `RECEBIDAS` | Arrays de dados locais usados no fallback de data mais recente |
| Formato do texto | `"Dados de: DD/MM/AAAA às HH:MM"` — personalize conforme preferência do cliente |
