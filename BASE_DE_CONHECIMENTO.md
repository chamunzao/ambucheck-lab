# Base de Conhecimento — Aprendizados do AmbuCheck

> Referencia para criacao de novos aplicativos web.
> Extraido do projeto AmbuCheck (SPA com Firebase, dark theme, multi-tenant).

---

## 1. ARQUITETURA

### 1.1 SPA (Single Page Application) sem framework

**Padrao usado:**
```
index.html (unico arquivo)
├── <script type="module">  → Firebase SDK (ESM imports)
├── <style>                 → CSS completo com variaveis
├── HTML                    → Todas as paginas como <div class="page">
└── <script>                → Logica JS vanilla
```

**Vantagens comprovadas:**
- Deploy instantaneo (1 arquivo, GitHub Pages, Firebase Hosting)
- Zero build tools, zero dependencias locais
- Funciona offline com localStorage

**Problemas encontrados:**
- Arquivo ficou com 2200+ linhas — dificulta manutencao
- Codigo duplicado (auto-login apareceu 2x)
- Sem separacao de responsabilidades

**Recomendacao para novos apps:**
- Ate ~800 linhas: arquivo unico OK
- 800-2000 linhas: separar em `style.css` + `app.js` + `index.html`
- 2000+ linhas: considerar modulos ES6 (`auth.js`, `checklist.js`, `stock.js`)

### 1.2 Navegacao SPA

```javascript
// Padrao: paginas como divs, mostrar/esconder com classe
.page { display: none }
.page.active { display: block }

// Paineis dentro da pagina app
.panel { display: none }
.panel.active { display: block }

// Funcao de navegacao
function showPanel(id) {
  document.querySelectorAll('.panel').forEach(p => p.classList.remove('active'));
  document.getElementById('panel-' + id).classList.add('active');
}
```

### 1.3 Firebase — Setup Client-Side

```javascript
// Padrao: importar via CDN ESM, expor para script nao-modular
import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.12.0/firebase-app.js';
import { getAuth, signInWithEmailAndPassword, ... } from '.../firebase-auth.js';
import { getFirestore, doc, getDoc, setDoc, ... } from '.../firebase-firestore.js';

const app = initializeApp(firebaseConfig);
window._fbAuth = getAuth(app);
window._fbDb = getFirestore(app);
window._fbFns = { signInWithEmailAndPassword, doc, getDoc, setDoc, ... };
window._fbReady = true;
window.dispatchEvent(new Event('firebase-ready'));
```

**Licao aprendida:** Nunca confiar que Firebase estara pronto no momento do uso.
Sempre checar `window._fbReady` antes de chamar funcoes Firebase.

---

## 2. DESIGN SYSTEM (Dark Theme)

### 2.1 Paleta de Cores (CSS Variables)

```css
:root {
  /* Cor primaria (identidade) */
  --primary: #E02020;
  --primary-dark: #B01010;
  --primary-light: #FF4040;

  /* Backgrounds (do mais escuro ao mais claro) */
  --dark:   #080A0F;   /* body background */
  --dark2:  #0F1219;   /* cards, sidebar */
  --dark3:  #161B26;   /* headers de card, inputs bg */
  --dark4:  #1E2536;   /* hover states */

  /* Textos e bordas */
  --border: #232B3E;
  --text:   #F0F2F8;   /* texto principal */
  --muted:  #8892A4;   /* texto secundario */
  --muted2: #50596A;   /* placeholders, separadores */

  /* Semanticas */
  --green:  #22C55E;   /* sucesso, OK */
  --green2: #15803D;   /* botoes verdes */
  --yellow: #F5A623;   /* alerta, pendente */
  --blue:   #2563EB;   /* informacao */
  --purple: #7C3AED;   /* especial, premium */
}
```

### 2.2 Tipografia

**Regra:** fonte *display* (Syne, Bebas, etc.) deixa o app com cara de produto de consumo/lifestyle. Pra apps **corporativos / B2B / saude / operacionais**, use uma sans neutra (Inter, DM Sans, Manrope) com pesos moderados.

```css
/* Titulos — corporate (recomendado pra SaaS, saude, B2B) */
font-family: 'Inter', 'DM Sans', sans-serif;
font-weight: 600 | 700;
letter-spacing: -0.2px;  /* tighten um pouco pra parecer "oficial" */

/* Corpo */
font-family: 'DM Sans', sans-serif;
font-weight: 300 | 400 | 500 | 600;

/* Google Fonts import */
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@500;600;700&family=DM+Sans:wght@300;400;500;600;700&display=swap');
```

**Licao AmbuCheck:** comecou com Syne 800 em todos os titulos — usuario descreveu como "feia e pouco profissional". Trocar por Inter 600-700 com letter-spacing justo resolveu a percepcao sem mudar estrutura.

Pra apps de *branding / marketing / criativo*, ai sim fonte display faz sentido.

### 2.3 Componentes Reutilizaveis

#### Card
```css
.card {
  background: var(--dark2);
  border: 1px solid var(--border);
  border-radius: 12px;
  margin-bottom: 16px;
  overflow: hidden;
}
.card-header {
  background: var(--dark3);
  padding: 12px 18px;
  display: flex;
  align-items: center;
  gap: 9px;
  border-bottom: 1px solid var(--border);
}
.card-body {
  padding: 16px 18px;
  display: flex;
  flex-direction: column;
  gap: 13px;
}
```

#### Input
```css
.input {
  width: 100%;
  background: #0A0C12;
  border: 1px solid var(--border);
  border-radius: 7px;
  color: var(--text);
  font-family: 'DM Sans', sans-serif;
  font-size: 0.87rem;
  padding: 9px 12px;
  outline: none;
  transition: border-color 0.2s;
}
.input:focus { border-color: var(--primary) }
.input::placeholder { color: var(--muted2) }
```

#### Badge
```css
.badge {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 0.68rem;
  font-weight: 700;
}
.badge-ok    { background: rgba(34,197,94,0.12);  color: var(--green);  border: 1px solid rgba(34,197,94,0.2) }
.badge-error { background: rgba(224,32,32,0.12);  color: #FF7070;      border: 1px solid rgba(224,32,32,0.2) }
.badge-info  { background: rgba(37,99,235,0.12);  color: var(--blue);   border: 1px solid rgba(37,99,235,0.2) }
.badge-warn  { background: rgba(245,166,35,0.12); color: var(--yellow); border: 1px solid rgba(245,166,35,0.2) }
```

#### Botao
```css
.btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 9px 18px;
  border-radius: 8px;
  font-family: 'DM Sans', sans-serif;
  font-size: 0.85rem;
  font-weight: 600;
  cursor: pointer;
  border: none;
  transition: all 0.2s;
}
.btn-primary { background: var(--primary); color: white }
.btn-ghost   { background: transparent; border: 1px solid var(--border); color: var(--muted) }
.btn-danger  { background: rgba(224,32,32,0.1); color: #FF7070; border: 1px solid rgba(224,32,32,0.2) }
```

#### Stat Card (Dashboard)
```css
.stat {
  background: var(--dark2);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 16px;
  position: relative;
  overflow: hidden;
}
.stat::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 2px;
  background: var(--primary); /* cor muda por tipo */
}
```

#### Toast (Notificacao)
```css
.toast {
  position: fixed;
  bottom: 24px; right: 24px;
  background: var(--dark3);
  border: 1px solid var(--border);
  border-radius: 10px;
  padding: 12px 18px;
  box-shadow: 0 8px 28px rgba(0,0,0,0.5);
  z-index: 500;
  display: none;
}
.toast.show {
  display: flex;
  animation: slideIn 0.22s ease;
}
```

#### Modal
```css
.modal-overlay {
  display: none;
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.8);
  z-index: 300;
  align-items: center;
  justify-content: center;
  backdrop-filter: blur(3px);
}
.modal-overlay.show { display: flex }
.modal-box {
  background: var(--dark3);
  border: 1px solid var(--border);
  border-radius: 16px;
  padding: 28px;
  width: 90%; max-width: 460px;
  max-height: 90vh;
  overflow-y: auto;
  animation: popIn 0.22s cubic-bezier(.175,.885,.32,1.275);
}
```

### 2.4 Layout Responsivo

```css
/* Desktop: sidebar fixa + conteudo a direita */
.app-wrap { display: flex; min-height: 100vh }
.sidebar  { width: 232px; position: fixed; top: 0; bottom: 0; left: 0 }
.app-main { margin-left: 232px; flex: 1 }

/* Mobile: sidebar esconde, bottom nav aparece */
@media (max-width: 768px) {
  .sidebar    { transform: translateX(-100%); transition: transform 0.25s }
  .sidebar.open { transform: translateX(0) }
  .app-main   { margin-left: 0; padding-bottom: 70px }
  .bottom-nav { display: flex; position: fixed; bottom: 0; left: 0; right: 0; height: 60px }
}
@media (min-width: 769px) {
  .bottom-nav { display: none }
}
```

### 2.5 Grids Responsivos
```css
.grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 12px }
.grid-3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 12px }
.stats  { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px }

@media (max-width: 768px) {
  .grid-2, .grid-3 { grid-template-columns: 1fr }
  .stats { grid-template-columns: 1fr 1fr }
}
```

---

## 3. AUTENTICACAO & MULTI-TENANT

### 3.1 Estrutura Multi-Empresa

```
companies (array no localStorage, collection no Firestore)
├── company_1
│   ├── users[]        → todos os usuarios desta empresa
│   ├── ambs[]         → ambulancias
│   ├── checklists[]   → registros
│   ├── stock[]        → estoque
│   ├── config{}       → nome, RT principal, URL drive
│   └── ...
├── company_2
│   └── ...
```

**Padrao de acesso isolado por empresa:**
```javascript
const cid = () => currentUser.companyId;
const getUsers = () => DB.getC(cid(), 'users') || [];
const setUsers = (v) => DB.setC(cid(), 'users', v);
// Cada empresa so acessa seus proprios dados
```

### 3.2 Roles (Papeis)

```javascript
const ROLES = {
  superadmin:    { label: 'Super Admin',  pill: 'rp-super',  access: ['*'] },
  Administrador: { label: 'Admin',        pill: 'rp-admin',  access: ['*'] },
  RT:            { label: 'Responsavel Tecnico', pill: 'rp-rt', access: ['dashboard','checklist','aprovacoes','alteracoes','estoque','usuarios','ambulancias','config'] },
  Enfermeiro:    { label: 'Enfermeiro',   pill: 'rp-func',   access: ['dashboard','checklist','plantao','alteracoes','estoque'] },
  Tecnico:       { label: 'Tecnico',      pill: 'rp-func',   access: ['dashboard','checklist','sugerir','alteracoes','estoque'] },
  Condutor:      { label: 'Condutor',     pill: 'rp-func',   access: ['dashboard','checklist','sugerir','alteracoes','estoque'] },
};
```

**Licao 1:** controle de acesso visual NAO e seguranca. Sempre validar role tambem no backend/Firestore Rules.

**Licao 2:** papeis que parecem "iguais" (Condutor/Tecnico/Enfermeiro) frequentemente precisam ver **blocos diferentes** do mesmo formulario — nao so paineis diferentes. Ver secao 11.4 (visibilidade de campos por papel).

### 3.3 Fluxo de Auth

```
1. Tentar Firebase Auth (signInWithEmailAndPassword)
2. Se falhar → fallback localStorage (busca em todas as empresas)
3. loginOk() → salva sessao SEM senha
4. Auto-login: checa sessao salva ao carregar
```

### 3.4 Planos SaaS + Trial (liberado por Super Admin)

```javascript
const PLANS = [
  { id: 'basico',  name: 'Basico',  price: 49,  maxAmbs: 2,        desc: 'ate 2 ambulancias' },
  { id: 'padrao',  name: 'Padrao',  price: 129, maxAmbs: 6,        desc: 'ate 6 ambulancias' },
  { id: 'premium', name: 'Premium', price: 249, maxAmbs: Infinity, desc: 'ilimitado' },
];

// Trial: super-admin libera acesso de teste com data limite.
// Campo opcional `trialAte` na empresa (ISO date). Se ausente → plano normal.
// Se presente e futura → trial ativo. Se passada → bloqueia novos cadastros.
function getCompanyTrial(coId){
  const co = getAllCos().find(c => c.id === coId);
  if(!co)          return {ok:false, reason:'Empresa nao encontrada'};
  if(!co.trialAte) return {ok:true,  trial:false};
  const hoje = new Date().toISOString().slice(0,10);
  const ativo = co.trialAte >= hoje;
  const dias  = Math.ceil((new Date(co.trialAte) - new Date(hoje)) / 86400000);
  return ativo
    ? {ok:true,  trial:true, ate:co.trialAte, dias}
    : {ok:false, trial:true, ate:co.trialAte, dias, reason:`Trial expirou em ${co.trialAte}.`};
}

// Uso no ponto de criacao (bloqueio):
const st = getCompanyTrial(cid());
if(!st.ok){ showToast('Bloqueado: ' + st.reason); return; }
```

**Padrao:** trial **nao bloqueia leitura** — so `create`/`update` de entidades novas. Assim o feedback coletado durante o teste nao some quando expira, e o RT consegue mostrar o que tem pra decidir se vai pagar.

**UX:** banner no topbar indicando "Trial - Xd restantes" (verde) ou "Trial expirou" (vermelho).

---

## 4. PERSISTENCIA DE DADOS

### 4.1 Padrao: localStorage + Firebase Sync

```javascript
const DB = {
  get(key) {
    try {
      const raw = localStorage.getItem('prefix_' + key);
      return raw ? JSON.parse(raw) : null;
    } catch { return null }
  },
  set(key, value) {
    try {
      localStorage.setItem('prefix_' + key, JSON.stringify(value));
    } catch (e) {
      if (e.name === 'QuotaExceededError') {
        alert('Armazenamento local cheio');
      }
    }
  },
  // Versao com sync Firestore
  setC(companyId, key, value) {
    this.set('co_' + companyId + '_' + key, value);
    // Background sync
    if (window._fbReady) {
      setDoc(doc(db, 'companies', companyId, key, 'data'),
        { value: JSON.stringify(value), updated: Date.now() },
        { merge: true }
      ).catch(e => console.warn('Sync:', e));
    }
  }
};
```

**Licao:** localStorage tem limite de ~5-10MB. Para apps com muitos dados (200+ itens de estoque x multiplas empresas), priorizar Firestore como fonte principal.

### 4.2 Padrao CRUD

```javascript
// GET (com fallback)
const getItems = () => DB.getC(companyId, 'items') || [];

// CREATE
function addItem(item) {
  const items = getItems();
  items.unshift({ id: generateId(), ...item, criado: new Date().toLocaleString('pt-BR') });
  setItems(items);
}

// UPDATE
function updateItem(id, changes) {
  const items = getItems();
  const idx = items.findIndex(i => i.id === id);
  if (idx >= 0) items[idx] = { ...items[idx], ...changes };
  setItems(items);
}

// DELETE
function removeItem(id) {
  setItems(getItems().filter(i => i.id !== id));
}
```

### 4.3 IDs Unicos

```javascript
const generateId = () =>
  Date.now().toString(36).toUpperCase() +
  Math.random().toString(36).slice(2, 4).toUpperCase();
// Resultado: "M5X2HKAB"
```

---

## 5. PADROES DE UI/UX

### 5.1 Formulario com Radio Buttons Estilizados

```html
<label class="radio-btn">
  <input type="radio" name="grupo" value="opcao1" onchange="update()">
  <span>Opcao 1</span>
</label>
```
```css
.radio-btn input { display: none }
.radio-btn span {
  display: inline-block;
  padding: 6px 13px;
  border-radius: 6px;
  border: 1px solid var(--border);
  background: #0A0C12;
  color: var(--muted);
  cursor: pointer;
  transition: all 0.15s;
}
.radio-btn input:checked + span {
  background: var(--primary);
  border-color: var(--primary);
  color: white;
}
```

### 5.2 Barra de Progresso

```html
<div class="progress-row">
  <span class="progress-label">Progresso</span>
  <div class="progress-bar">
    <div class="progress-fill" id="fill" style="width: 0%"></div>
  </div>
  <span class="progress-count" id="count">0/0</span>
</div>
```

### 5.3 Tabela com Acoes

```html
<div class="table-wrap">
  <table>
    <thead><tr><th>Nome</th><th>Status</th><th>Acoes</th></tr></thead>
    <tbody id="tbody"></tbody>
  </table>
</div>
```
```javascript
function renderTable(data) {
  const tbody = document.getElementById('tbody');
  if (!data.length) {
    tbody.innerHTML = '<tr><td colspan="3" class="empty">Nenhum registro.</td></tr>';
    return;
  }
  tbody.innerHTML = data.map(item => `<tr>
    <td>${esc(item.name)}</td>
    <td><span class="badge badge-ok">${esc(item.status)}</span></td>
    <td><button class="btn btn-danger btn-sm" onclick="remove('${esc(item.id)}')">Remover</button></td>
  </tr>`).join('');
}
```

### 5.4 Fluxo de Aprovacao

```
Funcionario sugere item
    ↓
Sugestao fica "pendente"
    ↓
RT/Admin vê na tela de aprovacoes (badge com contagem)
    ↓
Abre modal → Aprovar / Rejeitar (com observacao)
    ↓
Se aprovado → item adicionado automaticamente ao checklist e/ou estoque
```

### 5.5 Controle de Estoque com Alerta

```javascript
// Cada item tem: qty (atual) e min (minimo)
const lowStock = stock.filter(item => item.qty <= item.min);

// Exibir alerta se houver itens baixos
if (lowStock.length) {
  showAlert(`${lowStock.length} item(ns) com estoque baixo`);
}

// Grid de estoque com botoes +/-
function changeQty(id, delta) {
  const item = stock.find(i => i.id === id);
  item.qty = Math.max(0, item.qty + delta);
  saveStock(stock);
  renderStock();
}
```

---

## 6. SEGURANCA — LICOES APRENDIDAS

### 6.1 OBRIGATORIO em todo app

```javascript
// 1. Sanitizar TODA saida HTML (previne XSS)
function esc(s) {
  if (s == null) return '';
  return String(s)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}

// 2. Anti-spam em formularios
const locks = {};
function lockSubmit(key, ms) {
  if (locks[key]) return false;
  locks[key] = true;
  setTimeout(() => { locks[key] = false }, ms || 2500);
  return true;
}

// 3. Validar URLs externas
function isValidUrl(url, allowedDomain) {
  try {
    const u = new URL(url);
    return u.protocol === 'https:' && u.hostname.endsWith(allowedDomain);
  } catch { return false }
}

// 4. Hash de senhas (nunca armazenar texto plano)
async function hashPassword(password) {
  const enc = new TextEncoder().encode(password);
  const buf = await crypto.subtle.digest('SHA-256', enc);
  return Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2, '0')).join('');
}

// 5. Sessao segura (nunca armazenar senha)
function saveSession(user) {
  const safe = { id: user.id, email: user.email, nome: user.nome, role: user.role };
  localStorage.setItem('session', JSON.stringify(safe));
}
```

### 6.2 Erros que foram cometidos (nao repetir)

| Erro | Consequencia | Correcao |
|------|-------------|----------|
| Senha do admin no codigo-fonte | Qualquer pessoa ve a senha | Usar hash SHA-256 |
| innerHTML com dados do usuario | XSS — roubo de sessao | Sempre usar `esc()` |
| Senha salva no localStorage | Exposta pelo DevTools | Salvar apenas id/email/role |
| Auto-login duplicado | Executa 2x, potencial conflito | Remover duplicatas |
| Sem rate-limiting | Spam de registros | lockSubmit() em todo formulario |
| URL do Drive sem validacao | Redirecionamento malicioso | Whitelist de dominio |
| Sem validacao de email | Registros com "aaa" como email | Regex de validacao |
| Sem limites de tamanho | Payload gigante no storage | maxLength em campos |

### 6.3 Firestore Security Rules (modelo)

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Usuarios so leem seus proprios dados
    match /users/{userId} {
      allow read, write: if request.auth.uid == userId;
    }
    // Dados da empresa: apenas membros
    match /companies/{companyId}/{document=**} {
      allow read, write: if request.auth != null
        && get(/databases/$(database)/documents/users/$(request.auth.uid)).data.companyId == companyId;
    }
  }
}
```

---

## 7. PADROES DE DADOS (SCHEMAS)

### 7.1 Usuario
```json
{
  "id": "u_M5X2HK",
  "nome": "Leonardo",
  "email": "leo@empresa.com",
  "role": "Administrador",
  "companyId": "co_ABC123",
  "criado": "16/04/2026, 14:30:00"
}
```

### 7.2 Empresa
```json
{
  "id": "co_ABC123",
  "nome": "SAMU Regional",
  "plano": "Padrao",
  "criado": "16/04/2026, 10:00:00"
}
```

### 7.3 Item de Estoque
```json
{
  "id": "epinefrina",
  "name": "Epinefrina 1mg/ml",
  "cat": "Medicamento",
  "qty": 20,
  "min": 3,
  "unit": "amp"
}
```

### 7.4 Checklist
```json
{
  "id": "CHK-M5X2HK",
  "dt": "16/04/2026, 07:15:00",
  "condutor": "Carlos",
  "amb": "SA-01 (JBV 5I90)",
  "turno": "07:00 MANHA",
  "combustivel": "COMPLETO",
  "m_agua": "NORMAL",
  "m_oleo": "NORMAL",
  "hodometro": "45740",
  "cil01": "150 kgf",
  "pneus": "SIM",
  "intercorrencias": ""
}
```

### 7.5 Sugestao (Workflow de Aprovacao)
```json
{
  "id": "SUG-M5X2HK",
  "nome": "Desfibrilador",
  "categoria": "Equipamento",
  "tipo": "checklist",
  "justificativa": "Necessario para atendimento cardiaco",
  "solicitante": "Carlos",
  "status": "pendente | aprovado | rejeitado",
  "obsRT": "Aprovado conforme protocolo",
  "dtSolicitacao": "16/04/2026",
  "dtDecisao": "16/04/2026"
}
```

### 7.6 Alteracao/Manutencao
```json
{
  "id": "ALT-M5X2HK",
  "amb": "SA-01 (JBV 5I90)",
  "tipo": "Mecanica | Eletrica | Equip. Medico | Carroceria | Acidente",
  "grav": "Leve | Moderada | Grave",
  "desc": "Freio traseiro com folga",
  "resp": "Oficina Central",
  "status": "Pendente | Em Andamento | Resolvido",
  "custo": "350.00"
}
```

---

## 8. INTEGRACOES

### 8.1 Google Apps Script (Exportar para Drive)

```javascript
async function sendToDrive(payload) {
  const url = config.gas_url;
  if (!url || !isValidUrl(url, 'script.google.com')) return;
  try {
    await fetch(url, {
      method: 'POST',
      body: JSON.stringify(payload),
      headers: { 'Content-Type': 'application/json' }
    });
  } catch (e) {
    console.warn('Erro Drive:', e.message);
  }
}
```

### 8.2 PWA (Progressive Web App)

```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="NomeDoApp">
<meta name="theme-color" content="#E02020">
```

---

## 9. CHECKLIST PARA NOVOS APPS

### Antes de comecar
- [ ] Definir roles e permissoes
- [ ] Definir se e multi-tenant (varias empresas)
- [ ] Escolher persistencia (localStorage, Firebase, Supabase, etc)
- [ ] Definir paleta de cores (adaptar as variaveis CSS acima)

### Estrutura minima
- [ ] Tela de login/registro
- [ ] Dashboard com stats
- [ ] Sidebar + topbar (ou bottom nav mobile)
- [ ] Sistema de toast/notificacoes
- [ ] Modais para acoes

### Seguranca (obrigatorio)
- [ ] Funcao `esc()` para sanitizar HTML
- [ ] `lockSubmit()` em todos os formularios
- [ ] Validacao de email com regex
- [ ] Limites de tamanho em todos os campos
- [ ] Sessao sem dados sensiveis
- [ ] Senhas nunca em texto plano
- [ ] URLs externas validadas por whitelist
- [ ] Firestore Rules configuradas

### Qualidade
- [ ] Nenhum codigo duplicado
- [ ] Tratamento de erros visivel ao usuario
- [ ] Funciona offline (localStorage)
- [ ] Responsivo (mobile + desktop)
- [ ] Dados sanitizados antes de exibir

---

## 10. SNIPPETS PRONTOS PARA COPIAR

### Helper: ID unico
```javascript
const uid = () => Date.now().toString(36).toUpperCase() + Math.random().toString(36).slice(2,4).toUpperCase();
```

### Helper: Data/Hora formatada (pt-BR)
```javascript
const nowStr  = () => new Date().toLocaleString('pt-BR');
const nowDate = () => new Date().toISOString().slice(0,10);
const nowTime = () => new Date().toTimeString().slice(0,5);
```

### Helper: Query selectors curtos
```javascript
const qs  = s => document.querySelector(s);
const qid = id => document.getElementById(id);
const gR  = name => { const e = qs(`input[name="${name}"]:checked`); return e ? e.value : '' };
```

### Helper: Toast
```javascript
function showToast(msg, duration = 3200) {
  const t = document.getElementById('toast');
  document.getElementById('toast-msg').textContent = msg;
  t.classList.add('show');
  clearTimeout(t._t);
  t._t = setTimeout(() => t.classList.remove('show'), duration);
}
```

### Helper: Modal
```javascript
function openModal(id)  { document.getElementById(id).classList.add('show') }
function closeModal(id) { document.getElementById(id).classList.remove('show') }
```

---

## 11. PADROES DE MULTI-HIERARQUIA E UX GATING

> Secao extraida do ciclo de features: multi-RT, trial, escala,
> seletor de ambulancia e visibilidade de campos por papel.

### 11.1 Hierarquia multi-nivel (tenant → sub-owner → recursos)

Padrao: **empresa → RT → (ambulancias | funcionarios)**. Generalizavel pra qualquer app que tenha "uma organizacao com varios responsaveis/filiais/gerentes, cada um dono de seus proprios recursos".

```javascript
// Schema: entidade aponta pro sub-owner via campo `rt` (ou `managerId`, `filialId`, etc)
{ id:'a_xyz', prefixo:'SA-01', rt:'u_rt1', ... }       // ambulancia do RT
{ id:'u_xyz', nome:'Carlos',  role:'Condutor', rt:'u_rt1' }  // funcionario do RT

// Scoping automatico: se quem loga e sub-owner, filtra tudo por ele
function getScopedAmbs(){
  const all = getAmbs();
  if(CU.role === 'Administrador' || CU.role === 'superadmin') return all;
  if(CU.role === 'RT') return all.filter(a => a.rt === CU.id);
  // Funcionario ve so onde esta na equipe
  return all.filter(a => (a.equipe||[]).some(e => e.userId === CU.id));
}
```

**Chave:** mesma funcao `getScoped*()` aplicada em TODOS os renderizadores. Nenhum `render*` chama `getAll*` direto.

**Admin** tambem ganha um dropdown "Filtrar por RT" pra simular a visao de cada sub-owner — essencial pra debug e suporte.

### 11.2 Constante enum com metadata (evita `if/else` espalhados)

Padrao: valores "enum-like" que precisam de icone, cor, label, etc.

```javascript
const TIPOS = {
  ASB: {icon:'🚑', cor:'#22C55E', label:'Suporte Basico',    equipe:['Condutor','Tecnico']},
  ASA: {icon:'🚑', cor:'#E02020', label:'Suporte Avancado',  equipe:['Condutor','Enfermeiro','Medico']},
  UTI: {icon:'🏥', cor:'#7C3AED', label:'UTI Movel',          equipe:['Condutor','Enfermeiro','Medico']},
};

// Render: um helper unico consome a metadata
function tipoPill(t){
  const ti = TIPOS[t];
  if(!ti) return '<span class="badge">-</span>';
  return `<span class="badge" style="background:${ti.cor}22;color:${ti.cor};border:1px solid ${ti.cor}44">
    ${ti.icon} ${t}
  </span>`;
}
```

**Bonus:** o campo `equipe` vira preenchimento automatico de formulario quando o tipo e escolhido.

### 11.3 Gating de formulario (click-to-expand como primeiro passo)

Padrao: em vez de mostrar um formulario de 6 blocos, forcar o usuario a escolher **uma coisa** primeiro (contexto) antes que o resto apareca.

```html
<!-- Primeiro passo: card grande clicavel -->
<div class="gate-card" onclick="toggleGate()">
  <span class="gate-num">1</span>
  <span class="gate-title" id="gate-title">Selecione X</span>
  <span class="gate-info" id="gate-info">Clique para abrir</span>
  <span class="gate-caret">▾</span>
</div>
<div class="gate-list" id="gate-list" style="display:none"></div>

<!-- Resto do form — so aparece depois da escolha -->
<div id="gated-form" style="display:none"> ... </div>
```

```javascript
function escolher(itemId){
  qid('gate-selected').value = itemId;
  qid('gate-title').textContent = 'Selecionado';
  qid('gate-info').innerHTML = `<b>${nomeDoItem}</b>`;
  qid('gate-list').style.display = 'none';
  qid('gated-form').style.display = 'block';   // <- libera o resto
}
```

**Ganhos:** (1) reduz ansiedade visual ao abrir o app, (2) garante que a escolha critica foi feita, (3) permite filtrar opcoes do resto do form baseado na escolha do gate.

**Quando usar:** formularios longos onde uma escolha define o contexto das demais (ex: selecionar cliente antes de fazer pedido, selecionar equipamento antes de checklist).

### 11.4 Visibilidade de campos por papel (nao so de paineis)

Papeis geralmente precisam ver **blocos diferentes do mesmo formulario**, nao so paineis diferentes no menu.

```html
<!-- Dentro do mesmo card de Identificacao -->
<div class="card-body">
  <!-- Comum: todos veem -->
  <input id="data">
  <input id="horario">

  <!-- So condutor/admin: envolver em wrapper com id -->
  <div id="bloco-campos-condutor">
    <input id="km-inicio">
    <input id="condutor-nome">
    <input id="temp-interna">
  </div>

  <!-- So enfermeiro/admin -->
  <div id="bloco-campos-enfermeiro" style="display:none">
    <input id="medicacao-admin">
  </div>
</div>
```

```javascript
function setupFormByRole(){
  const role = CU.role;
  const showCondutor  = role === 'Condutor'  || role === 'RT' || role === 'Administrador';
  const showEnf       = role === 'Enfermeiro' || role === 'RT' || role === 'Administrador';

  qid('bloco-campos-condutor').style.display  = showCondutor ? '' : 'none';
  qid('bloco-campos-enfermeiro').style.display = showEnf      ? '' : 'none';
}
```

**Impacto na validacao:** campos escondidos NAO devem ser obrigatorios. Checar visibilidade no submit:

```javascript
const exigeCondutor = qid('bloco-campos-condutor').style.display !== 'none';
if(exigeCondutor && !qid('km-inicio').value) missing.push('KM');
```

### 11.5 Lista colapsavel com detalhe por item (+/-)

Padrao: linha principal enxuta, detalhes abrem via botao `+`/`-` na propria linha.

```html
<div class="item-row">
  <span>Nome do item</span>
  <span>Qtd: 3</span>
  <span>Data: 15/06</span>
  <button onclick="toggleDetail('item1')" id="btn-item1">+</button>
</div>
<div id="detail-item1" style="display:none">
  <!-- Sub-itens / lotes / variacoes -->
</div>
```

```javascript
function toggleDetail(id){
  const el = qid('detail-' + id);
  const btn = qid('btn-' + id);
  const aberto = el.style.display !== 'none';
  el.style.display = aberto ? 'none' : 'block';
  btn.textContent = aberto ? '+' : '−';
}
```

**Bonus no AmbuCheck:** clicar no `+` com `qty > 1` e sem detalhes ainda → **auto-cria** detalhes (ex: lote por unidade do estoque) com datas em branco. UX reduzida a um clique.

### 11.6 Filtro inline no header da tabela

Em vez de uma barra de filtros separada, colocar selects direto no header do card. Populacao feita uma unica vez na primeira renderizacao.

```html
<div class="card-header">
  <span class="card-title">Usuarios</span>
  <select id="filtro-rt" onchange="renderUsers()" style="margin-left:auto">
    <option value="">Todos RTs</option>
  </select>
  <button>+ Adicionar</button>
</div>
```

```javascript
function renderUsers(){
  // Popular filtro so uma vez
  const sel = qid('filtro-rt');
  if(sel && sel.options.length <= 1){
    getRTs().forEach(u => {
      const o = document.createElement('option');
      o.value = u.id; o.textContent = u.nome;
      sel.appendChild(o);
    });
  }
  const filtro = sel?.value || '';
  let users = getUsers();
  if(filtro) users = users.filter(u => u.rt === filtro);
  // render tbody...
}
```

### 11.7 Agrupamento visual por categoria dentro de tabela

Padrao: tabela plana fica ruim quando tem 20+ linhas misturando tipos. Agrupar com headers de secao mantem densidade alta sem perder clareza.

```javascript
function renderAgrupado(users){
  const grupos = {};
  users.forEach(u => { (grupos[u.role] = grupos[u.role] || []).push(u); });

  // Ordem fixa de prioridade
  const ordem = ['Administrador','RT','Enfermeiro','Tecnico','Condutor'];
  const chaves = Object.keys(grupos).sort((a,b) => {
    const ia = ordem.indexOf(a), ib = ordem.indexOf(b);
    return (ia < 0 ? 99 : ia) - (ib < 0 ? 99 : ib);
  });

  tbody.innerHTML = chaves.map(role => {
    const header = `<tr><td colspan="N" class="group-header">${role} - ${grupos[role].length}</td></tr>`;
    const rows   = grupos[role].map(u => `<tr>...</tr>`).join('');
    return header + rows;
  }).join('');
}
```

### 11.8 Mensagem de convite via WhatsApp (formatacao nativa)

Padrao BR: cadastro gera senha provisoria e envia credenciais via WhatsApp. O WhatsApp suporta `*negrito*`, `_italico_`, `~riscado~`, `` `mono` ``.

```javascript
function gerarConvite(nome, email, senha, role, empresa, url){
  const primeiroNome = (nome||'').split(' ')[0];
  return `Prezado(a) ${primeiroNome},

Seu acesso ao *AmbuCheck* foi liberado pela *${empresa}*.

*CREDENCIAIS DE ACESSO*
Funcao:  ${role}
E-mail:  ${email}
Senha:   ${senha}

*LINK DE ACESSO*
${url}

_Orientacoes:_
• Acesse em qualquer navegador
• Altere sua senha no primeiro acesso

Atenciosamente,
*${empresa}*`;
}

function abrirWhatsApp(whatsappNumero, mensagem){
  const num = (whatsappNumero||'').replace(/\D/g,'');
  const msg = encodeURIComponent(mensagem);
  const url = num ? `https://wa.me/55${num}?text=${msg}` : `https://wa.me/?text=${msg}`;
  window.open(url, '_blank');
}
```

**Armadilhas aprendidas:**
- **Evitar emojis em mensagens corporativas** — aparecem como `�` em muitos aparelhos/web. Usar so formatacao nativa (`*`, `_`) e bullets ASCII (`•`).
- **Usar so primeiro nome** — evita bugs tipo "Olá, EnfJoao!" quando o nome foi digitado errado.
- **Tom corporate** ("Prezado(a)", "Atenciosamente") em vez de "Olá! 👋" parece muito mais profissional pro usuario final.
- **`text=` com `encodeURIComponent`** — senao quebra acentos e quebras de linha.

### 11.9 CSS do gating/expandir (reusavel)

```css
.gate-card{
  background:#05070B; border:1px solid var(--border); border-radius:12px;
  padding:18px 22px; cursor:pointer; display:flex; align-items:center; gap:14px;
  transition:border-color 0.2s, background 0.2s;
}
.gate-card:hover{ border-color:var(--primary); }
.gate-card.selected{ border-color:var(--primary); background:linear-gradient(135deg,#0A0C12,#0F0608); }
.gate-num{
  display:inline-flex; align-items:center; justify-content:center;
  width:30px; height:30px; border-radius:7px;
  background:var(--primary); color:white; font-weight:700;
}
.gate-caret{ font-size:1rem; color:var(--muted); transition:transform 0.2s; }
.gate-card.open .gate-caret{ transform:rotate(180deg); }
.gate-list{
  margin-top:8px; background:var(--dark2); border:1px solid var(--border);
  border-radius:10px; padding:8px; display:flex; flex-direction:column; gap:4px;
}
.gate-opt{
  padding:12px 14px; border-radius:7px; cursor:pointer;
  display:flex; align-items:center; gap:10px; border:1px solid transparent;
}
.gate-opt:hover{ background:var(--dark3); border-color:var(--border); }
.gate-opt.active{ background:rgba(224,32,32,0.08); border-color:rgba(224,32,32,0.25); }
```

### 11.10 Licoes meta do ciclo

- **Codigo orfao e caro.** No AmbuCheck, varias funcoes referenciavam constantes (`TIPOS`, `getEscalas`) e IDs do DOM (`modal-amb-title`, `ma-prefixo`) que nao existiam — provavelmente restos de refatoracao incompleta. Resultado: botoes que nao faziam nada, silenciosamente. **Antes de refatorar, grep do nome da funcao/id. Se tiver referencia sem definicao, corrigir antes de adicionar feature nova.**

- **Scope creep em SaaS e real.** A cada 2-3 features, surgia uma nova dimensao (multi-RT, trial, escala). Uma vez que o app tem **multi-tenant + multi-role + multi-owner + trial/plano**, qualquer feature nova tem que perguntar: "como isso se comporta em cada combinacao?". Sem essa disciplina, bugs se acumulam em silencio.

- **Deploy por arquivo unico + GitHub Pages acelera feedback.** Push → 30s → cliente testa. Sem CI, sem staging. Perfeito pra MVP / beta.

- **localStorage mente.** Pode ter dados de testes antigos com schemas incompatveis. Sempre usar getters que tolerem campos ausentes (`{...i, lotes: i.lotes||[]}`) em vez de assumir shape.
