# login-portal

## Descrição

Tela de login completa do portal Oikos BPO. Exibe formulário com e-mail e senha, autentica via Supabase Auth (REST direto), armazena os tokens no `sessionStorage` e redireciona para a URL do dashboard após login bem-sucedido. Inclui refresh automático de JWT com intervalo configurável (padrão 50 min) e proteção contra expiração de sessão.

---

## HTML

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Oikos BPO — Login</title>
</head>
<body class="login-body">
  <div class="login-shell">
    <div class="login-card">
      <div class="login-logo">Oikos <span>BPO</span></div>
      <p class="login-sub">Acesse seu painel financeiro</p>

      <form id="login-form" autocomplete="off">
        <label class="login-label" for="email">E-mail</label>
        <input
          class="login-input"
          type="email"
          id="email"
          placeholder="seu@email.com"
          required
          autocomplete="username"
        >

        <label class="login-label" for="senha">Senha</label>
        <input
          class="login-input"
          type="password"
          id="senha"
          placeholder="••••••••"
          required
          autocomplete="current-password"
        >

        <div class="login-error" id="login-error" aria-live="polite"></div>

        <button class="login-btn" type="submit" id="login-btn">
          Entrar
        </button>
      </form>

      <p class="login-footer">
        Oikos Consultoria de Valores Mobiliários
      </p>
    </div>
  </div>
</body>
</html>
```

---

## CSS

```css
/* ── Variáveis de sistema Oikos (devem estar no :root global) ── */
:root {
  --bg0: #09090e;
  --bg1: #111118;
  --bg2: #18181f;
  --t1:  #eeedf5;
  --t2:  #8e8da0;
  --accent: #7c6ff7;   /* equivale a --ac no HTML original */
  --red:    #f87171;   /* equivale a --re */
}

/* ── Layout ── */
.login-body {
  margin: 0;
  padding: 0;
  background: var(--bg0);
  color: var(--t1);
  font-family: 'DM Sans', sans-serif;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}

.login-shell {
  width: 100%;
  max-width: 380px;
  padding: 24px 16px;
}

/* ── Card ── */
.login-card {
  background: var(--bg1);
  border: 1px solid rgba(255,255,255,0.07);
  border-radius: 14px;
  padding: 36px 32px 28px;
}

/* ── Logo ── */
.login-logo {
  font-size: 18px;
  font-weight: 600;
  letter-spacing: .04em;
  margin-bottom: 6px;
}
.login-logo span {
  color: var(--accent);
}

.login-sub {
  font-size: 12px;
  color: var(--t2);
  margin: 0 0 28px;
}

/* ── Form elements ── */
.login-label {
  display: block;
  font-size: 11px;
  color: var(--t2);
  margin-bottom: 6px;
  margin-top: 16px;
  letter-spacing: .03em;
}

.login-input {
  width: 100%;
  background: var(--bg2);
  border: 1px solid rgba(255,255,255,0.07);
  border-radius: 8px;
  padding: 10px 12px;
  color: var(--t1);
  font-family: 'DM Sans', sans-serif;
  font-size: 13px;
  outline: none;
  box-sizing: border-box;
  transition: border-color .15s;
}
.login-input:focus {
  border-color: var(--accent);
}

/* ── Erro ── */
.login-error {
  font-size: 11px;
  color: var(--red);
  margin-top: 10px;
  min-height: 16px;
}

/* ── Botão ── */
.login-btn {
  width: 100%;
  margin-top: 20px;
  padding: 11px;
  background: var(--accent);
  border: none;
  border-radius: 8px;
  color: #fff;
  font-family: 'DM Sans', sans-serif;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: opacity .15s;
}
.login-btn:hover   { opacity: .88; }
.login-btn:disabled { opacity: .5; cursor: not-allowed; }

/* ── Footer ── */
.login-footer {
  font-size: 10px;
  color: var(--t2);
  text-align: center;
  margin-top: 24px;
  opacity: .6;
}
```

---

## JavaScript

```js
// ── Configuração — substitua pelos valores reais ────────────────
const API_URL  = 'https://SEU_PROJETO.railway.app';   // URL do backend Railway
const SUPA_URL = 'https://XXXX.supabase.co';          // URL do projeto Supabase
const SUPA_ANON = 'eyJhbGci...';                      // Chave anon pública do Supabase
const DASHBOARD_URL = '/portal/CLIENTE/index.html';   // URL do dashboard pós-login

// ── Autenticação via Supabase REST ──────────────────────────────
const form     = document.getElementById('login-form');
const errEl    = document.getElementById('login-error');
const loginBtn = document.getElementById('login-btn');

form.addEventListener('submit', async (e) => {
  e.preventDefault();
  errEl.textContent = '';
  loginBtn.disabled = true;
  loginBtn.textContent = 'Entrando…';

  const email = document.getElementById('email').value.trim();
  const senha = document.getElementById('senha').value;

  try {
    const res = await fetch(`${SUPA_URL}/auth/v1/token?grant_type=password`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'apikey': SUPA_ANON,
      },
      body: JSON.stringify({ email, password: senha }),
    });

    const data = await res.json();

    if (!res.ok) {
      throw new Error(data.error_description || data.message || 'Credenciais inválidas');
    }

    // Armazena tokens na sessão
    sessionStorage.setItem('sb_access_token',  data.access_token);
    sessionStorage.setItem('sb_refresh_token', data.refresh_token);
    sessionStorage.setItem('sb_expires_at',    String(Date.now() + data.expires_in * 1000));

    // Registra acesso no backend (opcional)
    await fetch(`${API_URL}/api/registrar-acesso`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email }),
    }).catch(() => {}); // ignora falha de registro

    // Inicia refresh automático e redireciona
    agendarRefresh(data.expires_in);
    window.location.href = DASHBOARD_URL;

  } catch (err) {
    errEl.textContent = err.message;
    loginBtn.disabled = false;
    loginBtn.textContent = 'Entrar';
  }
});

// ── Refresh automático do JWT ────────────────────────────────────
// Chama refreshAccessToken() 10 min antes do vencimento
function agendarRefresh(expiresIn) {
  const delay = Math.max((expiresIn - 600) * 1000, 60000); // mín. 1 min
  setTimeout(async () => {
    await refreshAccessToken();
    // Agenda próximo ciclo (tokens novos têm expires_in=3600 por padrão)
    agendarRefresh(3600);
  }, delay);
}

async function refreshAccessToken() {
  const refreshToken = sessionStorage.getItem('sb_refresh_token');
  if (!refreshToken) return;

  const res = await fetch(`${SUPA_URL}/auth/v1/token?grant_type=refresh_token`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'apikey': SUPA_ANON },
    body: JSON.stringify({ refresh_token: refreshToken }),
  });

  if (!res.ok) {
    // Sessão inválida — desloga
    sessionStorage.clear();
    window.location.href = '/login.html';
    return;
  }

  const data = await res.json();
  sessionStorage.setItem('sb_access_token',  data.access_token);
  sessionStorage.setItem('sb_refresh_token', data.refresh_token);
  sessionStorage.setItem('sb_expires_at',    String(Date.now() + data.expires_in * 1000));
}
```

---

## Exemplo de uso

```html
<!-- Salvar como login.html na raiz do portal do cliente -->
<!-- Garantir que a fonte DM Sans está carregada no <head> -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&display=swap" rel="stylesheet">

<!-- Configurar as constantes no topo do script: -->
<script>
  const API_URL     = 'https://oikos-api.up.railway.app';
  const SUPA_URL    = 'https://abcdefgh.supabase.co';
  const SUPA_ANON   = 'eyJhbGci...sua_chave_anon';
  const DASHBOARD_URL = '/portal/tofamily/index.html';
</script>
```

---

## Variáveis a substituir

| Variável | Descrição |
|---|---|
| `API_URL` | URL base do backend Railway (ex.: `https://oikos-api.up.railway.app`) |
| `SUPA_URL` | URL do projeto Supabase (ex.: `https://abcd.supabase.co`) |
| `SUPA_ANON` | Chave anon pública do Supabase (sem segredo, pode ir no front) |
| `DASHBOARD_URL` | Caminho do dashboard pós-login (ex.: `/portal/tofamily/index.html`) |
| `login.html` | URL da página de login para redirect no logout/sessão expirada |
| Texto `"Acesse seu painel financeiro"` | Subtítulo customizável por cliente |
| Texto `"Oikos Consultoria de Valores Mobiliários"` | Rodapé com nome da empresa |
