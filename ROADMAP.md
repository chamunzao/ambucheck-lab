# Roadmap Comercial — AmbuCheck

> Análise feita em **2026-04-17** sobre o estado do projeto no commit `013e7e0`.
> Documento vivo: atualizar à medida que itens forem concluídos.

---

## Veredicto curto

O produto funcional existe (checklist, estoque, validades, escala, plantão, aprovações, PDF), mas **não está pronto para vender**. Faltam bloqueadores críticos de segurança, compliance e monetização.

Estimativa até o primeiro cliente pagante: **4–8 semanas** de trabalho focado, dependendo do escopo mínimo aceito.

---

## 🚨 Bloqueadores — sem isso NÃO vende

### 1. Regras de segurança do Firestore (crítico de segurança)
Não há `firestore.rules` no repo. Se as regras do Firebase estão em modo test (`allow read, write: if true`), **qualquer pessoa com o link do app pode ler/escrever dados de qualquer empresa**. Vazamento de dados de paciente → multa LGPD até 2% do faturamento (máx R$ 50M) + processo.

**Ação:** escrever regras que isolem `companies/{companyId}` por `auth.uid` e papel. Versionar `firestore.rules` no repo.

### 2. LGPD + termos legais
Sem termo de uso, política de privacidade, DPO ou canal de exclusão de dados. Você vai tratar dados sensíveis (prontuário, paciente, medicamento controlado com lacre) sem base legal documentada.

**Ação:** termo de uso + política de privacidade + fluxo de exportação/exclusão de dados (LGPD art. 18).

### 3. Pagamento real
Hoje o campo `plano` é texto livre, sem gateway, sem webhook, sem enforcement de limite. Cobrar por fora (boleto/PIX manual) funciona para 1–3 clientes; não escala.

**Ação:** integração com **Asaas** ou **Mercado Pago** (ambos têm PIX recorrente, cartão, e emitem NF-e). Stripe Brasil é opção mas cobra 4,99% + R$ 0,39 vs. Asaas ~2,99%.

### 4. Enforcement de limites por plano
Plano Básico diz "até 2 ambulâncias" mas ninguém valida. Cliente paga R$ 49 e cadastra 50.

**Ação:** `saveAmb()` precisa checar `getAmbs().length` vs. limite do plano antes de inserir. Idem para usuários (se for o caso).

### 5. CNPJ + Nota Fiscal
Sem CNPJ você não emite NF-e. Cliente PJ precisa de nota pra pagar.

**Ação:** abrir MEI (limite R$ 81k/ano) ou ME Simples Nacional. CNAE 6202-3/00 (consultoria em TI) ou 6319-4/00 (outras atividades de TI).

---

## ⚠️ Alto impacto — faltando, mas não bloqueadores absolutos

### 6. Modo offline para o checklist
Ambulância em deslocamento = sinal instável. Hoje o app trava se o Firebase cair no meio de um checklist. `localStorage` já é usado como fallback de login, mas o checklist não persiste offline.

**Ação:** Service Worker + IndexedDB para checklist. É o recurso que mais diferencia AmbuCheck de ferramentas genéricas.

### 7. Foto anexada no checklist
Estoque vencido, dano na ambulância, intercorrência — checklist sem foto é auditoria fraca. O `storageBucket` do Firebase já está configurado mas sem upload implementado.

**Ação:** upload em cards de alteração, estoque e sugestão.

### 8. Notificação por e-mail/WhatsApp
Quando RT recebe sugestão pra aprovar, ele não sabe. Quando trial vai expirar, cliente não recebe aviso. Hoje tudo depende de o usuário abrir o app.

**Ação:** EmailJS (grátis até 200/mês) ou SendGrid. Para WhatsApp, manter o "Gerar mensagem e abrir WhatsApp Web" que já existe até ter volume pra justificar API oficial.

### 9. Assinatura digital com valor jurídico
O PDF tem campo de assinatura, mas é só desenho. Para valer em fiscalização ANVISA/COREN precisa ser ICP-Brasil (caro) ou ao menos hash + timestamp + log de auditoria imutável.

**Ação mínima:** registrar hash SHA-256 do checklist + timestamp do servidor + IP + user agent no Firestore, e mostrar no PDF. Não substitui ICP-Brasil, mas já é evidência.

### 10. Auditoria (log de ações)
Quem apagou o item? Quem trocou o RT da ambulância? Hoje não dá pra saber.

**Ação:** coleção `audit/{companyId}` com `{userId, action, entity, before, after, timestamp}`.

### 11. Domínio + hospedagem profissional
Rodando em `localhost` não vende. Firebase Hosting é grátis e já está conectado (`ambucheck-47802.firebaseapp.com`). Falta domínio próprio (ex.: `ambucheck.com.br` — R$ 40/ano no Registro.br).

---

## 📋 Desejável — para ficar competitivo

12. Onboarding guiado (vídeo de 90s + checklist de primeiros passos).
13. Suporte ao cliente (Crisp grátis até 2 agentes, ou WhatsApp Business).
14. Landing page separada do app com pitch, preços, depoimentos.
15. Export CSV de checklists e estoque (cliente vai pedir).
16. Relatório gerencial mensal (PDF com métricas agregadas).
17. Multi-base (uma empresa com bases SAMU em cidades diferentes).
18. API pública (para integrar com ERP do cliente grande) — só depois de ter cliente pedindo.
19. Monitoramento de erro (Sentry grátis para projetos pequenos).

---

## 💰 Análise de preço — mercado brasileiro

Seu preço atual (R$ 49 / R$ 129 / R$ 249) está **subprecificado** para B2B healthcare no Brasil. Margem insuficiente para sustentar suporte + compliance.

### Comparáveis de mercado

| Categoria | Exemplo | Preço/mês |
|---|---|---|
| Gestão clínica | iClinic, Doctoralia | R$ 179–599 |
| Frota genérica | Cobli, Omnilink | R$ 30–80/veículo |
| Farmácia/estoque hospitalar | Controle.net, Logic | R$ 200–800 |
| Prontuário eletrônico | ProntMed, ProDoctor | R$ 150–450 |
| **Seu nicho (ambulância/SAMU)** | ~vazio no Brasil | — |

**Ponto forte:** nicho com pouca concorrência direta. Quem gerencia frota SAMU hoje usa Excel + WhatsApp.

### Sugestão de precificação realista (2026)

| Plano | Perfil | Ambulâncias | Preço |
|---|---|---|---|
| **Essencial** | Remoção particular, 1 base | até 2 | **R$ 129/mês** (anual R$ 1.290 = 2 meses grátis) |
| **Operacional** | Clínica com remoção, UTI móvel pequena | até 6 | **R$ 349/mês** (anual R$ 3.490) |
| **Frota** | SAMU regional, grupos hospitalares | até 15 | **R$ 799/mês** (anual R$ 7.990) |
| **Corporativo** | Redes grandes, multi-base | ilimitado + multi-base + API | **sob consulta** (a partir de R$ 1.500/mês + setup R$ 2.000) |

**Por quê esses números:**
- R$ 129 é o menor que cobre custo de processar pagamento + suporte básico. Abaixo disso, cada ticket de suporte come a margem.
- R$ 349 é ancorado em iClinic (R$ 299 Padrão) — profissional de saúde já aceita esse ticket.
- R$ 799 é o preço de "não pensar" para quem tem 10 ambulâncias (R$ 80/amb vs. Cobli R$ 30–80/veículo).
- Corporativo com setup capta base SAMU pública, que exige customização e tem orçamento.

### Adicionais que se pagam sozinhos

| Adicional | Preço |
|---|---|
| Assinatura digital ICP-Brasil | +R$ 49/mês |
| Multi-base | +R$ 99/mês |
| Integração ERP via API | +R$ 249/mês |
| Treinamento in-company | R$ 800/sessão |

### Política de trial
**14 dias grátis, sem cartão**, com todos os recursos. Trial menor que isso não dá tempo de treinar a equipe.

---

## 🛣️ Roadmap sugerido (ordem de execução)

### Sprint 1 — Fundação jurídica (1 semana)
- [ ] Regras do Firestore restritivas (versionar em `firestore.rules`)
- [ ] Termo de uso + política de privacidade (template de advogado SaaS, R$ 800–2.000)
- [ ] Abrir MEI

### Sprint 2 — Monetização (2 semanas)
- [ ] Integração Asaas (PIX + cartão recorrente)
- [ ] Enforcement de limite por plano (ambulâncias, talvez usuários)
- [ ] Tela "Minha assinatura" (fatura, trocar cartão, cancelar)
- [ ] Webhook de status de assinatura (ativa, inadimplente, cancelada)

### Sprint 3 — Confiança operacional (2 semanas)
- [ ] Modo offline do checklist (Service Worker + IndexedDB)
- [ ] Upload de foto em checklist/alteração
- [ ] Log de auditoria
- [ ] Hash SHA-256 + timestamp em PDF
- [ ] E-mail transacional (aprovação pendente, trial expirando)

### Sprint 4 — Go-to-market (1 semana)
- [ ] Landing page (Framer ou Webflow)
- [ ] Domínio `ambucheck.com.br`
- [ ] Onboarding video
- [ ] Canal de suporte (Crisp)

### Depois (iterativo, conforme cliente pedir)
- Multi-base
- API pública
- Assinatura ICP-Brasil
- Relatórios gerenciais avançados
- Export CSV

---

## Custo estimado até o primeiro cliente pagante

| Item | Custo |
|---|---|
| Advogado (termo + política) | R$ 800–2.000 |
| MEI/Simples (abertura + contador inicial) | R$ 300 + R$ 120/mês |
| Domínio | R$ 40/ano |
| Firebase (free tier suficiente até ~50 empresas) | R$ 0 |
| Asaas (taxa sobre transação) | 2,99% + R$ 1,49/boleto |
| Landing page (Framer) | R$ 100/mês |
| Crisp suporte (free até 2 agentes) | R$ 0 |
| **Total antes do primeiro cliente** | **~R$ 2.000–3.500** |

Com 10 clientes no plano **Operacional** (R$ 349), o faturamento é R$ 3.490/mês — já cobre custo operacional e paga seu tempo. **Break-even realista: 5–8 clientes.**

---

## Estado do código na data da análise

- Único arquivo de produto: `index.html` (~5.013 linhas, tudo inline).
- Único doc adicional: `BASE_DE_CONHECIMENTO.md`.
- Nenhum `package.json`, `firestore.rules`, `.gitignore`, teste ou pipeline CI.
- Firebase: `ambucheck-47802.firebaseapp.com` já configurado; `storageBucket` presente mas não utilizado.
- Dois commits de redesign já feitos (`15f6be6` baseline, `013e7e0` onda 1 do interface-design).

---

## Próximos passos imediatos (quando retomar)

1. Verificar no console do Firebase se as regras do Firestore estão em modo test (se sim: **trocar imediatamente**, é o maior risco aberto agora).
2. Decidir entre Asaas vs. Mercado Pago para gateway.
3. Escolher a ordem dos sprints (sugerido acima) e começar pelo Sprint 1.
