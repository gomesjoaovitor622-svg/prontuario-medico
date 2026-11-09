# Aprovação — Portão G2

**Data:** 08/11/2026
**Produto:** PRONTUÁRIO PRO — Gestão Clínica Inteligente

Use este checklist para aprovar a arquitetura antes do início da Fase 3.

---

## 1. Documentos entregues

- [ ] `01-AUDITORIA-REFERENCIA.md` — auditoria da referência
- [ ] `02-ARQUITETURA.md` — stack, camadas, segurança, multi-tenancy
- [ ] `03-MODELO-DADOS.md` — tabelas, constraints, índices, RLS, imutabilidade
- [ ] `04-PLANO-IMPLEMENTACAO.md` — fases, prazos, critérios de aceite
- [ ] `05-RISCOS-E-SEGURANCA.md` — riscos e mitigações
- [ ] `06-ADRS.md` — decisões arquiteturais
- [ ] `07-DIRECAO-VISUAL.md` — identidade e design system

---

## 2. Decisões que precisam de "sim" explícito

| # | Decisão | Alternativa | Recomendação |
|---|---|---|---|
| D1 | Tauri 2 + React + NestJS + PostgreSQL | Electron + Node; Flutter + FastAPI | **Tauri + NestJS** |
| D2 | Multi-instituição desde o início (RLS) | single-tenant e refatorar depois | **multi-instituição** |
| D3 | Prisma + SQL versionado para RLS/triggers | TypeORM; Drizzle | **Prisma + SQL** |
| D4 | Assinatura eletrônica simples na v1 | ICP-Brasil já na v1 | **simples na v1** |
| D5 | Sem escrita clínica offline | offline com sincronização | **sem offline clínico** |
| D6 | PDF gerado no servidor | PDF no navegador | **servidor** |
| D7 | Financeiro próprio simples (sem TISS) | TISS na v1 | **sem TISS na v1** |

---

## 3. Ambiente

- [ ] Autorizo mover o projeto para `C:\dev\prontuario-pro` (recomendado) ou manter no OneDrive.
- [ ] Autorizo instalar Rust, PostgreSQL e/ou Docker via `winget`.
- [ ] Vou fornecer credencial de push do GitHub (PAT) quando solicitado.
- [ ] O repositório `gomesjoaovitor622-svg/prontuario-medico` é o destino correto.

---

## 4. Produto

Responder quando possível (não bloqueia a Fase 3):

1. Hospedagem: servidor da clínica, VPS ou nuvem?
2. A v1 precisa de TISS/convênios ou apenas particular e convênio simples?
3. Existe base legada de pacientes a importar?
4. A v1 terá telemedicina? (recomendação: não)
5. Há exigência de assinatura ICP-Brasil desde o início?
6. Quantas instituições/unidades no primeiro cliente?
7. Qual o volume estimado de pacientes e atendimentos/mês?

---

## 5. Critério de liberação da Fase 3

A Fase 3 começa quando:

- [ ] este checklist estiver aprovado;
- [ ] o ambiente tiver Node, PostgreSQL (ou Docker) e Rust disponíveis;
- [ ] o repositório estiver conectado ao GitHub;
- [ ] não houver dúvida bloqueante sobre tenancy, auth ou modelo de dados.

**Próxima entrega após aprovação:** migração inicial + autenticação + RBAC + auditoria, com testes de integração reais contra PostgreSQL.
