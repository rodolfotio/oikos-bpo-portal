# toast-notificacao

## Descrição

Componente de notificação toast que aparece no canto inferior direito da tela e desaparece automaticamente após 3 segundos. Suporta variantes de tipo (sucesso, erro, aviso, info) com ícone e cor correspondentes. Pode ser empilhado (múltiplos toasts simultâneos). O arquivo original não contém um toast explícito — este componente foi extraído e adaptado do padrão de feedback de ação usado no portal (ex.: após salvar, exportar ou ao exibir erros de autenticação).

---

## HTML

```html
<!-- Container fixo — coloque uma vez no <body>, antes do fechamento </body> -->
<div id="toast-container" aria-live="polite" aria-atomic="false"></div>
```

Estrutura de um toast individual (gerada via JS):

```html
<div class="toast toast-sucesso" role="alert">
  <span class="toast-icon">✓</span>
  <span class="toast-msg">Dados atualizados com sucesso.</span>
  <button class="toast-close" onclick="this.parentElement.remove()">×</button>
</div>
```

---

## CSS

```css
/* ── Container de toasts (fixo no canto inferior direito) ── */
#toast-container {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 9999;
  display: flex;
  flex-direction: column;
  gap: 8px;
  pointer-events: none; /* deixa cliques passarem enquanto sem toasts */
}

/* ── Toast base ── */
.toast {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 16px;
  border-radius: 10px;
  border: 1px solid rgba(255,255,255,0.07);
  font-family: 'DM Sans', sans-serif;
  font-size: 12px;
  color: var(--t1);
  min-width: 240px;
  max-width: 340px;
  pointer-events: all;
  box-shadow: 0 4px 24px rgba(0,0,0,0.4);

  /* Animação de entrada */
  animation: toast-in .2s ease;
}

/* ── Variantes de cor ── */
.toast-sucesso {
  background: rgba(74, 222, 128, 0.12);
  border-color: rgba(74, 222, 128, 0.25);
}
.toast-sucesso .toast-icon { color: var(--green); }

.toast-erro {
  background: rgba(248, 113, 113, 0.12);
  border-color: rgba(248, 113, 113, 0.25);
}
.toast-erro .toast-icon { color: var(--red); }

.toast-aviso {
  background: rgba(251, 191, 36, 0.12);
  border-color: rgba(251, 191, 36, 0.25);
}
.toast-aviso .toast-icon { color: var(--yellow); }

.toast-info {
  background: var(--bg2);
  border-color: rgba(255,255,255,0.13);
}
.toast-info .toast-icon { color: var(--accent); }

/* ── Ícone ── */
.toast-icon {
  font-size: 14px;
  font-weight: 700;
  flex-shrink: 0;
}

/* ── Mensagem ── */
.toast-msg {
  flex: 1;
  line-height: 1.4;
  color: var(--t1);
}

/* ── Botão de fechar ── */
.toast-close {
  background: none;
  border: none;
  color: var(--t2);
  font-size: 16px;
  cursor: pointer;
  padding: 0 2px;
  line-height: 1;
  flex-shrink: 0;
  transition: color .15s;
}
.toast-close:hover { color: var(--t1); }

/* ── Animação de entrada ── */
@keyframes toast-in {
  from { opacity: 0; transform: translateY(12px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* ── Animação de saída (aplicada via JS) ── */
.toast-saindo {
  animation: toast-out .2s ease forwards;
}
@keyframes toast-out {
  from { opacity: 1; transform: translateY(0); }
  to   { opacity: 0; transform: translateY(8px); }
}
```

---

## JavaScript

```js
/**
 * Exibe um toast de notificação.
 *
 * @param {string} mensagem  - Texto a exibir
 * @param {'sucesso'|'erro'|'aviso'|'info'} tipo - Variante visual (padrão: 'info')
 * @param {number} duracaoMs - Tempo em ms até sumir automaticamente (padrão: 3000)
 */
function toast(mensagem, tipo = 'info', duracaoMs = 3000) {
  const container = document.getElementById('toast-container');
  if (!container) return;

  // Ícones por tipo
  const icones = {
    sucesso: '✓',
    erro:    '✕',
    aviso:   '⚠',
    info:    'ℹ',
  };

  // Cria elemento
  const el = document.createElement('div');
  el.className = `toast toast-${tipo}`;
  el.setAttribute('role', 'alert');
  el.innerHTML = `
    <span class="toast-icon">${icones[tipo] ?? 'ℹ'}</span>
    <span class="toast-msg">${mensagem}</span>
    <button class="toast-close" title="Fechar">×</button>
  `;

  // Botão fechar
  el.querySelector('.toast-close').addEventListener('click', () => removerToast(el));

  container.appendChild(el);

  // Remove automaticamente após duracaoMs
  setTimeout(() => removerToast(el), duracaoMs);
}

/**
 * Remove um toast com animação de saída.
 * @param {HTMLElement} el
 */
function removerToast(el) {
  if (!el || !el.parentElement) return;
  el.classList.add('toast-saindo');
  el.addEventListener('animationend', () => el.remove(), { once: true });
}

// ── Atalhos por tipo ─────────────────────────────────────────────
const toastSucesso = (msg, ms = 3000) => toast(msg, 'sucesso', ms);
const toastErro    = (msg, ms = 4000) => toast(msg, 'erro',    ms);
const toastAviso   = (msg, ms = 3500) => toast(msg, 'aviso',   ms);
const toastInfo    = (msg, ms = 3000) => toast(msg, 'info',    ms);
```

---

## Exemplo de uso

```html
<!-- 1. Adicione o container no body -->
<body>
  <!-- ... conteúdo do portal ... -->
  <div id="toast-container"></div>
</body>
```

```js
// 2. Chame em qualquer ação do dashboard:

// Após salvar dados
toastSucesso('Dados atualizados com sucesso.');

// Ao detectar erro de autenticação
toastErro('Sessão expirada. Faça login novamente.', 5000);

// Aviso de dados do mês sem lançamentos
toastAviso('Nenhum lançamento encontrado para este mês.');

// Informação genérica
toastInfo('Exportação em andamento...');

// Uso completo com parâmetros
toast('Token renovado automaticamente.', 'info', 2000);
```

---

## Variáveis a substituir

| Elemento | Descrição |
|---|---|
| `id="toast-container"` | ID do container fixo — deve ser único por página |
| `duracaoMs` padrão (`3000`) | Tempo padrão de exibição em milissegundos |
| `bottom: 20px; right: 20px` | Posição do container (pode ser `left`, `center` via `calc`) |
| `min-width: 240px; max-width: 340px` | Largura do toast — ajuste conforme viewport do cliente |
| Ícones por tipo (`✓`, `✕`, `⚠`, `ℹ`) | Substitua por ícones SVG ou de uma lib (ex.: Lucide, Heroicons) |
| Cores das variantes | Alinhadas com `--green`, `--red`, `--yellow`, `--accent` — ajuste se o cliente tiver paleta diferente |
