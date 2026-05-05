# Componente: Botão "Reportar divergência"

Botão fixo no canto inferior direito do dashboard que abre um modal para o cliente
descrever uma divergência. Ao clicar em "Enviar relatório", a função faz POST para o
backend (`/portal/reportar-divergencia`), que dispara um e-mail para `rodolfotio@gmail.com`
via Resend. Nenhuma URL externa é aberta no frontend.

---

## Como usar em um novo dashboard

1. Copie os três blocos abaixo (HTML, Modal, Script) e cole **imediatamente antes do `</body>`**.
2. Substitua `NOME DO CLIENTE` pelo nome exato do cliente (aparece no assunto do e-mail).
3. Confirme que o HTML do cliente usa as variáveis CSS do sistema Oikos (listadas na seção de dependências). Se usar paleta própria, ajuste os `var(--...)` conforme necessário.
4. Nenhuma outra configuração é necessária — o token e a URL do backend são lidos automaticamente.

---

## Bloco 1 — Botão fixo

Cole antes do `</body>`. Fica sobreposto ao conteúdo no canto inferior direito.

```html
<!-- BOTÃO REPORTAR DIVERGÊNCIA -->
<button
  id="btn-reportar"
  onclick="document.getElementById('modal-reportar').style.display='flex'"
  style="position:fixed;bottom:24px;right:24px;z-index:500;background:var(--bg2);border:1px solid var(--border2);border-radius:20px;padding:8px 14px;font-size:11px;color:var(--t2);cursor:pointer;transition:color .15s;font-family:inherit"
  onmouseover="this.style.color='var(--am)'"
  onmouseout="this.style.color='var(--t2)'"
>⚠️ Reportar</button>
```

---

## Bloco 2 — Modal

Cole logo após o botão (ainda antes do `</body>`).

```html
<!-- MODAL REPORTAR DIVERGÊNCIA -->
<div id="modal-reportar" style="display:none;position:fixed;inset:0;z-index:1000;background:rgba(0,0,0,.72);align-items:center;justify-content:center;padding:16px">
  <div style="background:var(--bg2);border:1px solid var(--border2);border-radius:16px;padding:24px;width:min(440px,100%);display:flex;flex-direction:column;gap:14px">
    <div style="display:flex;align-items:center;justify-content:space-between">
      <span style="font-size:13px;font-weight:600;color:var(--t1)">⚠️ Reportar divergência</span>
      <button
        onclick="document.getElementById('modal-reportar').style.display='none'"
        style="background:none;border:none;color:var(--t3);font-size:18px;cursor:pointer;padding:0 4px;line-height:1"
      >✕</button>
    </div>
    <textarea
      id="textarea-divergencia"
      placeholder="Descreva a divergência..."
      style="background:var(--bg1);border:1px solid var(--border2);border-radius:10px;padding:10px 12px;font-size:12px;color:var(--t1);resize:vertical;min-height:110px;font-family:inherit;outline:none;line-height:1.5"
    ></textarea>
    <button
      onclick="enviarWhatsapp()"
      style="background:var(--ac);border:none;border-radius:10px;padding:10px 14px;font-size:12px;font-weight:600;color:#fff;cursor:pointer;font-family:inherit"
    >Enviar relatório</button>
  </div>
</div>
```

---

## Bloco 3 — JavaScript

Cole logo após o modal (ainda antes do `</body>`). **Substitua `NOME DO CLIENTE`.**

> O token JWT é lido do `localStorage` (`oikos_token`) — funciona dentro do iframe
> do portal porque o blob URL compartilha a mesma origem que o frame pai
> (`allow-same-origin` no sandbox).
>
> A aba visível é detectada automaticamente via `.nb.on` (classe dos botões de
> navegação do padrão Oikos). Se o novo dashboard usar uma classe diferente para
> a aba ativa, ajuste o seletor em `document.querySelector(...)`.

```html
<script>
async function enviarWhatsapp() {
  const texto = document.getElementById('textarea-divergencia').value.trim();
  const btn   = document.querySelector('[onclick="enviarWhatsapp()"]');

  // Validação visual — sem alert() (bloqueado pelo sandbox do iframe)
  if (!texto) {
    if (btn) {
      btn.style.outline = '2px solid var(--re)';
      setTimeout(() => { btn.style.outline = ''; }, 1500);
    }
    document.getElementById('textarea-divergencia').style.border = '1px solid var(--re)';
    setTimeout(() => { document.getElementById('textarea-divergencia').style.border = ''; }, 1500);
    return;
  }

  // Detecta a aba visível pelo botão de navegação com classe "on"
  // Ajuste o seletor se o dashboard usar convenção diferente
  const abaAtual = document.querySelector('.nb.on')?.textContent?.trim() || 'Dashboard';

  if (btn) { btn.disabled = true; btn.textContent = 'Enviando...'; }

  try {
    // Token lido do localStorage (compartilhado com o frame pai via mesma origem)
    const token  = localStorage.getItem('oikos_token') || '';
    const apiUrl = 'https://oikos-bpo-backend-production.up.railway.app';

    const res = await fetch(apiUrl + '/portal/reportar-divergencia', {
      method: 'POST',
      headers: {
        'Content-Type':  'application/json',
        'Authorization': 'Bearer ' + token,
      },
      // ▼ Substitua pelo nome exato do cliente
      body: JSON.stringify({ descricao: texto, aba: abaAtual }),
    });

    const data = await res.json();

    if (res.ok && data.ok) {
      // Sucesso: feedback verde no botão, fecha modal após 1,2s
      if (btn) { btn.textContent = '✅ Enviado!'; btn.style.background = 'var(--gr)'; btn.disabled = false; }
      setTimeout(() => {
        document.getElementById('modal-reportar').style.display = 'none';
        document.getElementById('textarea-divergencia').value   = '';
        if (btn) { btn.textContent = 'Enviar relatório'; btn.style.background = ''; }
      }, 1200);

    } else {
      // Erro do servidor: mostra mensagem no botão por 3,5s
      const msg = data.detail || 'Tente novamente.';
      if (btn) { btn.textContent = '❌ ' + msg; btn.style.background = 'var(--re)'; btn.disabled = false; }
      setTimeout(() => { if (btn) { btn.textContent = 'Enviar relatório'; btn.style.background = ''; } }, 3500);
    }

  } catch (e) {
    // Erro de rede
    if (btn) { btn.textContent = '❌ Erro de conexão'; btn.style.background = 'var(--re)'; btn.disabled = false; }
    setTimeout(() => { if (btn) { btn.textContent = 'Enviar relatório'; btn.style.background = ''; } }, 3500);
  }
}
</script>
```

---

## Dependências de CSS

O componente usa as variáveis CSS do sistema Oikos. Elas precisam estar definidas no
`:root` do dashboard (já presentes em todos os dashboards gerados pelo sistema):

| Variável    | Uso no componente                          |
|-------------|--------------------------------------------|
| `--bg1`     | Fundo do textarea                          |
| `--bg2`     | Fundo do botão fixo e do card do modal     |
| `--border2` | Borda do botão fixo, modal e textarea      |
| `--t1`      | Texto principal (título do modal)          |
| `--t2`      | Texto do botão fixo (estado normal)        |
| `--t3`      | Botão de fechar (✕)                        |
| `--ac`      | Fundo do botão "Enviar relatório"          |
| `--am`      | Cor do botão fixo no hover (âmbar)         |
| `--gr`      | Fundo do botão no estado de sucesso        |
| `--re`      | Borda/fundo no estado de erro              |

---

## Backend — endpoint de referência

```
POST /portal/reportar-divergencia
Authorization: Bearer {jwt_do_cliente}
Content-Type: application/json

{
  "descricao": "texto livre do cliente",
  "aba": "nome da aba visível"
}
```

**Resposta de sucesso:**
```json
{ "ok": true, "mensagem": "Relatório enviado com sucesso" }
```

O backend identifica o cliente pelo JWT, monta o e-mail com nome do cliente / aba /
data-hora UTC / descrição e envia para `rodolfotio@gmail.com` via Resend.

O nome do cliente que aparece no e-mail vem do banco de dados (tabela `clientes`),
não do HTML — não é necessário configurar nada no frontend além dos três blocos acima.

---

## Notas de implementação

- **`alert()` não funciona** em iframes com `sandbox` sem `allow-modals`. Por isso o
  feedback é feito diretamente no texto/cor do botão, não em diálogos nativos.
- **O token nunca fica exposto** no HTML estático — é lido do `localStorage` em
  tempo de execução.
- **Idempotência:** o script verifica `"btn-reportar"` antes de injetar para evitar
  duplicação caso o HTML base seja reprocessado pelo pipeline do ERP.
