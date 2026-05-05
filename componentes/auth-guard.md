# auth-guard

## Descrição

Funções de guarda de autenticação para páginas do portal. `ensureValidToken()` deve ser chamada no topo de cada página protegida — ela verifica se existe token no `sessionStorage`, checa se está próximo do vencimento (menos de 5 min) e executa o refresh antes de prosseguir. `refreshAccessToken()` troca o refresh token por um novo par de tokens via Supabase Auth REST. Se qualquer etapa falhar, redireciona para o login.

---

## HTML

Sem HTML específico — este módulo é puramente JavaScript. Inclua o script antes do código de renderização da página:

```html
<!-- No <head> ou no topo do <body>, antes do script principal -->
<script src="/portal/shared/auth-guard.js"></script>
```

Ou inline no topo do script da página:

```html
<script>
  // Cole aqui o conteúdo da seção JavaScript abaixo
  // e chame ensureValidToken() como primeira instrução
</script>
```

---

## CSS

Nenhum CSS necessário para este componente.

---

## JavaScript

```js
// ════════════════════════════════════════════════════════════════
// auth-guard.js — Oikos BPO Portal
// Inclua antes de qualquer lógica de página que exija autenticação
// ════════════════════════════════════════════════════════════════

// ── Configuração — substitua pelos valores reais ────────────────
const SUPA_URL  = 'https://XXXX.supabase.co';   // URL do projeto Supabase
const SUPA_ANON = 'eyJhbGci...';                 // Chave anon pública
const LOGIN_URL = '/login.html';                 // Página de login do portal

// Margem de segurança em ms: se o token vence em menos de 5 min, renova já
const REFRESH_MARGIN_MS = 5 * 60 * 1000;

// ── ensureValidToken ─────────────────────────────────────────────
/**
 * Garante que existe um access_token válido na sessão.
 *
 * Fluxo:
 * 1. Lê access_token e expires_at do sessionStorage.
 * 2. Se não existir token → redireciona para login.
 * 3. Se o token vence em menos de REFRESH_MARGIN_MS → chama refreshAccessToken().
 * 4. Retorna o access_token atual (já renovado, se necessário).
 *
 * @returns {Promise<string>} access_token válido
 */
async function ensureValidToken() {
  const token     = sessionStorage.getItem('sb_access_token');
  const expiresAt = Number(sessionStorage.getItem('sb_expires_at') || 0);

  // Sem token → vai para login
  if (!token) {
    window.location.href = LOGIN_URL;
    return '';
  }

  // Token próximo do vencimento → renova antes de prosseguir
  const tempoRestante = expiresAt - Date.now();
  if (tempoRestante < REFRESH_MARGIN_MS) {
    console.debug('[auth-guard] Token próximo do vencimento — renovando…');
    await refreshAccessToken();
  }

  // Retorna token atualizado (pode ter sido substituído pelo refresh)
  return sessionStorage.getItem('sb_access_token') || '';
}

// ── refreshAccessToken ───────────────────────────────────────────
/**
 * Troca o refresh_token por um novo par access_token / refresh_token.
 *
 * Fluxo:
 * 1. Lê refresh_token do sessionStorage.
 * 2. Envia para o endpoint Supabase Auth /token?grant_type=refresh_token.
 * 3. Em caso de sucesso → atualiza sessionStorage com novos tokens e expires_at.
 * 4. Em caso de falha → limpa sessão e redireciona para login.
 *
 * @returns {Promise<void>}
 */
async function refreshAccessToken() {
  const refreshToken = sessionStorage.getItem('sb_refresh_token');

  // Sem refresh token → sessão morta
  if (!refreshToken) {
    _deslogar();
    return;
  }

  try {
    const res = await fetch(`${SUPA_URL}/auth/v1/token?grant_type=refresh_token`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'apikey': SUPA_ANON,
      },
      body: JSON.stringify({ refresh_token: refreshToken }),
    });

    if (!res.ok) {
      // Refresh inválido ou revogado → desloga
      console.warn('[auth-guard] Refresh falhou com status', res.status);
      _deslogar();
      return;
    }

    const data = await res.json();

    // Persiste novos tokens
    sessionStorage.setItem('sb_access_token',  data.access_token);
    sessionStorage.setItem('sb_refresh_token', data.refresh_token);
    sessionStorage.setItem('sb_expires_at',
      String(Date.now() + (data.expires_in ?? 3600) * 1000));

    console.debug('[auth-guard] Token renovado com sucesso.');
  } catch (err) {
    // Erro de rede → não desloga imediatamente (pode ser offline temporário)
    console.error('[auth-guard] Erro ao renovar token:', err);
  }
}

// ── _deslogar (uso interno) ──────────────────────────────────────
function _deslogar() {
  sessionStorage.clear();
  window.location.href = LOGIN_URL;
}

// ── Agendamento automático de refresh ───────────────────────────
/**
 * Agenda um refresh automático 10 minutos antes do vencimento do token.
 * Chame esta função uma vez após o login bem-sucedido.
 *
 * @param {number} expiresIn - Segundos até o vencimento (vindo da resposta do Supabase)
 */
function agendarRefreshAutomatico(expiresIn) {
  const delay = Math.max((expiresIn - 600) * 1000, 60_000); // mín. 1 min

  setTimeout(async () => {
    await refreshAccessToken();
    // Token renovado tem expires_in padrão de 3600s
    agendarRefreshAutomatico(3600);
  }, delay);
}
```

---

## Exemplo de uso

```html
<!-- index.html de uma página protegida -->
<script src="/portal/shared/auth-guard.js"></script>
<script>
  // Primeira instrução: garante sessão válida antes de qualquer render
  (async () => {
    const token = await ensureValidToken();
    if (!token) return; // ensureValidToken já redirecionou para login

    // A partir daqui, token está válido
    // Pode fazer fetch autenticado:
    const res = await fetch('/api/dados', {
      headers: { 'Authorization': `Bearer ${token}` }
    });
    const dados = await res.json();
    renderDashboard(dados);
  })();
</script>
```

```js
// Em um fetch helper reutilizável:
async function apiFetch(path, opts = {}) {
  const token = await ensureValidToken();
  return fetch(path, {
    ...opts,
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
      ...(opts.headers || {}),
    },
  });
}
```

---

## Variáveis a substituir

| Variável | Descrição |
|---|---|
| `SUPA_URL` | URL do projeto Supabase (ex.: `https://abcd.supabase.co`) |
| `SUPA_ANON` | Chave anon pública do Supabase |
| `LOGIN_URL` | Caminho da página de login (ex.: `/login.html` ou `/portal/login.html`) |
| `REFRESH_MARGIN_MS` | Margem antes do vencimento para acionar refresh (padrão: 5 min = 300 000 ms) |
