# Registro de Decisões Arquiteturais (ADRs)

**Produto:** PRONTUÁRIO PRO
**Data:** 08/11/2026

Formato: contexto → decisão → consequências. Decisões aceitas podem ser revistas com novo ADR.

---

## ADR-001 — Monorepo com `apps` e `packages`

**Contexto:** frontend desktop, API e contratos compartilhados precisam evoluir juntos.
**Decisão:** monorepo com `apps/api`, `apps/desktop` e `packages/shared`; dependências unidirecionais.
**Consequências:** um único PR pode alterar contrato e consumo; exige disciplina de build e CI. Alternativa (repositórios separados) foi descartada por custo de coordenação nesta fase.

---

## ADR-002 — Tauri 2 como shell desktop

**Contexto:** app Windows com instalador, ícone, updater e acesso ao SO, sem carregar um Chromium inteiro.
**Decisão:** Tauri 2 + WebView2.
**Consequências:** binário leve e integração nativa; exige Rust/MSVC no build e WebView2 no Windows 10. Electron foi descartado por tamanho e consumo.

---

## ADR-003 — React + TypeScript + Vite + Tailwind + shadcn/ui

**Contexto:** UI profissional, acessível e de evolução rápida.
**Decisão:** React 19, TS, Vite, Tailwind 4, shadcn/ui (Radix), Lucide.
**Consequências:** ecossistema maduro e controle do código dos componentes; exige padronização de tokens e revisão de acessibilidade.

---

## ADR-004 — TanStack Router e TanStack Query

**Contexto:** rotas tipadas, guards por permissão e estado de servidor previsível.
**Decisão:** TanStack Router para rotas; TanStack Query para cache/invalidação; Zustand para estado de UI.
**Consequências:** menos código manual de loading/erro; curva de aprendizado inicial.

---

## ADR-005 — NestJS 11 com Fastify

**Contexto:** API modular, testável, com DI, guards e OpenAPI.
**Decisão:** NestJS com adapter Fastify.
**Consequências:** estrutura clara por módulo e boa performance; Fastify limita alguns middlewares Express — usar equivalents nativos.

---

## ADR-006 — Prisma como ORM, SQL versionado para RLS/triggers

**Contexto:** produtividade com tipagem, mas o banco precisa de RLS, constraints de exclusão e triggers.
**Decisão:** Prisma para CRUD e migrações; SQL explícito na mesma migração para políticas, triggers, índices e enums.
**Consequências:** melhor DX sem abrir mão de recursos do PostgreSQL; exige teste que valide que RLS não foi esquecida.

---

## ADR-007 — Multi-tenancy por `institution_id` + RLS

**Contexto:** várias clínicas na mesma instalação; risco crítico de vazamento.
**Decisão:** coluna `institution_id` em todas as tabelas operacionais + políticas RLS + guard na API + testes de isolamento.
**Consequências:** defesa em profundidade; toda query precisa de contexto de tenant. Custo aceitável frente ao risco.

---

## ADR-008 — JWT curto + refresh rotativo com hash no banco

**Contexto:** sessão segura e revogável em app desktop.
**Decisão:** access token de 15 min em memória; refresh opaco de 30 dias, hash no banco, rotação a cada uso, armazenado no keychain do SO.
**Consequências:** revogação real e detecção de reutilização; exige endpoint de refresh robusto e tratamento de corrida no cliente.

---

## ADR-009 — Autorização RBAC com overrides individuais

**Contexto:** perfis fixos não cobrem a realidade de clínicas.
**Decisão:** perfis + `user_permissions` com `allow`/`deny`; `deny` vence; admin não concede o que não possui.
**Consequências:** flexibilidade com regra clara; UI precisa explicar o efeito efetivo e a API sempre revalidar.

---

## ADR-010 — Imutabilidade clínica com adendo

**Contexto:** prontuário não pode ser silenciosamente alterado.
**Decisão:** rascunho editável; finalização grava hash e congela; correção cria nova revisão com `amends_id` e justificativa; trigger impede update/delete.
**Consequências:** rastreabilidade e segurança jurídica; fluxo de correção mais burocrático (proposital).

---

## ADR-011 — PDF gerado no servidor

**Contexto:** documento oficial precisa ser idêntico em qualquer posto e verificável.
**Decisão:** HTML versionado + Chromium headless no servidor; PDF armazenado com hash; desktop apenas abre/imprime.
**Consequências:** consistência e auditabilidade; custo de CPU no servidor e necessidade de fila para picos.

---

## ADR-012 — Sem escrita clínica offline na v1

**Contexto:** clínicas têm internet instável; offline seria conveniente, mas perigoso sem sincronização.
**Decisão:** somente leitura em cache; escrita exige conexão.
**Consequências:** evita conflito e perda de dado clínico; pode gerar atrito operacional — mitigado com banner, retry e servidor local.

---

## ADR-013 — Assinatura eletrônica simples na v1

**Contexto:** assinatura qualificada ICP-Brasil exige certificado e processo jurídico.
**Decisão:** implementar assinatura eletrônica simples com autoria, hash, IP e auditoria; rotular claramente; deixar interface para qualificada futura.
**Consequências:** entrega valor imediato sem alegar conformidade indevida; ICP-Brasil vira módulo próprio.

---

## ADR-014 — Storage S3-compatível com URLs assinadas

**Contexto:** anexos e PDFs não devem viver no banco nem em pasta pública.
**Decisão:** MinIO/S3 com chave por UUID, bucket privado, URL assinada de curta duração, hash e exclusão lógica.
**Consequências:** segurança e escalabilidade; exige backup do storage e política de retenção.

---

## ADR-015 — Testes de integração com PostgreSQL real

**Contexto:** RLS, triggers, constraints e transações não são testáveis em SQLite.
**Decisão:** Testcontainers com PostgreSQL real no CI; SQLite não é usado.
**Consequências:** confiança alta no comportamento do banco; CI mais pesado.

---

## ADR-016 — Código da referência não será reutilizado

**Contexto:** o ZIP auditado não possui licença e é um protótipo com dados em memória.
**Decisão:** reimplementar padrões de UX; não copiar código, textos, identidade ou dados.
**Consequências:** risco jurídico reduzido; esforço maior de design e implementação — assumido.
