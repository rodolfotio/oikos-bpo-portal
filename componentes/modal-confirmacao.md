# modal-confirmacao

## Descrição

Modal genérico dark mode com overlay, título, texto descritivo, botão de confirmar e botão de cancelar. Fecha ao clicar fora do painel ou no botão cancelar. Aceita uma função de callback para a ação de confirmação. O arquivo original não contém um modal explícito — este componente foi extraído e adaptado do padrão visual dark (variáveis `--bg1`, `--bg2`, `--border`) e dos botões existentes (`.nb`, `.btn-sair`) para criar um modal consistente com o sistema de design Oikos BPO.

---

## HTML

```html
<!-- Overlay + painel do modal — coloque uma vez no <body> -->
<div id="modal-overlay" class="modal-overlay" role="dialog" aria-modal="true" aria-labelledby="modal-titulo" hidden>
  <div class="modal-panel">

    <!-- Cabeçalho -->
    <div class="modal-header">
      <span class="modal-titulo" id="modal-titulo">Confirmar ação</span>
      <button class="modal-close" onclick="fecharModal()" title="Fechar">×</button>
    </div>

    <!-- Corpo -->
    <div class="modal-body">
      <p class="modal-texto" id="modal-texto">
        Tem certeza que deseja realizar esta ação? Esta operação não pode ser desfeita.
      </p>
    </div>

    <!-- Rodapé com botões -->
    <div class="modal-footer">
      <button class="modal-btn modal-btn-cancelar" onclick="fecharModal()">Cancelar</button>
      <button class="modal-btn modal-btn-confirmar" id="modal-btn-confirmar">Confirmar</button>
    </div>

  </div>
</div>
```

---

## CSS

```css
/* ── Overlay (fundo escurecido) ── */
.modal-overlay {
  position: fixed;
  inset: 0;                             /* top/right/bottom/left: 0 */
  background: rgba(0, 0, 0, 0.65);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;

  /* Animação de entrada */
  animation: overlay-in .15s ease;
}
.modal-overlay[hidden] { display: none; }

@keyframes overlay-in {
  from { opacity: 0; }
  to   { opacity: 1; }
}

/* ── Painel central ── */
.modal-panel {
  background: var(--bg1);
  border: 1px solid rgba(255,255,255,0.13);  /* --border2 */
  border-radius: 14px;
  width: 100%;
  max-width: 420px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.5);

  /* Animação de entrada do painel */
  animation: panel-in .2s ease;
}

@keyframes panel-in {
  from { opacity: 0; transform: scale(.95) translateY(-8px); }
  to   { opacity: 1; transform: scale(1)  translateY(0); }
}

/* ── Cabeçalho ── */
.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 18px 20px 14px;
  border-bottom: 1px solid rgba(255,255,255,0.07);
}

.modal-titulo {
  font-size: 14px;
  font-weight: 600;
  color: var(--t1);
  letter-spacing: .02em;
}

.modal-close {
  background: none;
  border: none;
  color: var(--t2);
  font-size: 20px;
  cursor: pointer;
  line-height: 1;
  padding: 2px 4px;
  transition: color .15s;
}
.modal-close:hover { color: var(--t1); }

/* ── Corpo ── */
.modal-body {
  padding: 16px 20px;
}

.modal-texto {
  font-size: 13px;
  color: var(--t2);
  line-height: 1.6;
  margin: 0;
}

/* ── Rodapé com botões ── */
.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  padding: 14px 20px 18px;
}

/* ── Botões ── */
.modal-btn {
  padding: 8px 18px;
  border-radius: 8px;
  font-family: 'DM Sans', sans-serif;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all .15s;
}

.modal-btn-cancelar {
  background: var(--bg2);
  border: 1px solid rgba(255,255,255,0.07);
  color: var(--t2);
}
.modal-btn-cancelar:hover {
  background: var(--bg3, #22222c);
  color: var(--t1);
}

.modal-btn-confirmar {
  background: var(--accent);
  border: 1px solid transparent;
  color: #fff;
}
.modal-btn-confirmar:hover  { opacity: .88; }
.modal-btn-confirmar:active { opacity: .75; }

/* Variante destrutiva (vermelho) — adicione a classe .modal-btn-destrutivo */
.modal-btn-destrutivo {
  background: var(--red);
}
```

---

## JavaScript

```js
// ── Referência ao callback de confirmação atual ──────────────────
let _modalCallback = null;

/**
 * Abre o modal de confirmação.
 *
 * @param {Object}   opcoes
 * @param {string}   opcoes.titulo     - Título do modal
 * @param {string}   opcoes.texto      - Texto descritivo da ação
 * @param {string}   [opcoes.labelConfirmar='Confirmar'] - Rótulo do botão de confirmação
 * @param {boolean}  [opcoes.destrutivo=false] - Se true, botão confirmar fica vermelho
 * @param {Function} opcoes.onConfirmar - Callback executado ao confirmar
 */
function abrirModal({ titulo, texto, labelConfirmar = 'Confirmar', destrutivo = false, onConfirmar }) {
  document.getElementById('modal-titulo').textContent = titulo;
  document.getElementById('modal-texto').textContent  = texto;

  const btnConfirmar = document.getElementById('modal-btn-confirmar');
  btnConfirmar.textContent = labelConfirmar;
  btnConfirmar.classList.toggle('modal-btn-destrutivo', destrutivo);

  // Guarda o callback
  _modalCallback = onConfirmar;

  // Exibe o overlay
  const overlay = document.getElementById('modal-overlay');
  overlay.hidden = false;
  overlay.focus();

  // Fecha ao clicar fora do painel
  overlay.addEventListener('click', _fecharAoClicarFora, { once: true });
}

/**
 * Fecha o modal sem executar ação.
 */
function fecharModal() {
  const overlay = document.getElementById('modal-overlay');
  overlay.hidden = true;
  _modalCallback = null;
}

/**
 * Handler: fecha se o clique foi no overlay (fora do painel).
 */
function _fecharAoClicarFora(e) {
  if (e.target === document.getElementById('modal-overlay')) {
    fecharModal();
  }
}

// ── Confirmar ────────────────────────────────────────────────────
document.getElementById('modal-btn-confirmar')?.addEventListener('click', () => {
  if (typeof _modalCallback === 'function') {
    _modalCallback();
  }
  fecharModal();
});

// ── Fechar com Escape ────────────────────────────────────────────
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') fecharModal();
});
```

---

## Exemplo de uso

```html
<!-- HTML (uma vez no body) -->
<div id="modal-overlay" class="modal-overlay" role="dialog" aria-modal="true"
     aria-labelledby="modal-titulo" hidden>
  <div class="modal-panel">
    <div class="modal-header">
      <span class="modal-titulo" id="modal-titulo"></span>
      <button class="modal-close" onclick="fecharModal()">×</button>
    </div>
    <div class="modal-body">
      <p class="modal-texto" id="modal-texto"></p>
    </div>
    <div class="modal-footer">
      <button class="modal-btn modal-btn-cancelar" onclick="fecharModal()">Cancelar</button>
      <button class="modal-btn modal-btn-confirmar" id="modal-btn-confirmar">Confirmar</button>
    </div>
  </div>
</div>
```

```js
// Confirmação simples
abrirModal({
  titulo: 'Exportar relatório',
  texto: 'Deseja exportar os dados do mês de Abril/2026 em PDF?',
  labelConfirmar: 'Exportar',
  onConfirmar: () => exportarPDF(),
});

// Ação destrutiva (botão vermelho)
abrirModal({
  titulo: 'Encerrar sessão',
  texto: 'Tem certeza que deseja sair? Sua sessão será encerrada.',
  labelConfirmar: 'Sair',
  destrutivo: true,
  onConfirmar: () => { sessionStorage.clear(); window.location.href = '/login.html'; },
});
```

---

## Variáveis a substituir

| Elemento | Descrição |
|---|---|
| `id="modal-overlay"` | ID do overlay — único por página |
| `id="modal-titulo"` | ID do elemento de título (preenchido via JS) |
| `id="modal-texto"` | ID do elemento de texto (preenchido via JS) |
| `id="modal-btn-confirmar"` | ID do botão de confirmação |
| `max-width: 420px` | Largura máxima do painel do modal |
| `labelConfirmar` padrão | Rótulo padrão do botão de ação (personalize por contexto) |
| `/login.html` no exemplo de logout | URL de redirecionamento pós-logout |
