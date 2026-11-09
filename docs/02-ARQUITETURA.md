# Arquitetura — PRONTUÁRIO PRO

**Versão:** 1.0
**Data:** 08/11/2026
**Estado:** proposta para aprovação (Portão G2)
**Premissa:** produto novo, construído do zero. A referência auditada serve apenas como inspiração de UX.

---

## 1. Princípios que governam a arquitetura

1. **Servidor é a fonte da verdade.** Nenhum dado clínico oficial vive apenas no computador do usuário.
2. **Segurança no servidor.** Permissões são verificadas na API e reforçadas no banco (RLS). A UI apenas reflete o que é permitido.
3. **Isolamento por instituição.** Toda consulta operacional é limitada por `institution_id`.
4. **Imutabilidade clínica.** Registro finalizado não é sobrescrito nem apagado; correção é adendo versionado.
5. **Rastreabilidade total.** Autoria, data/hora do servidor, hash e trilha de auditoria em operações sensíveis.
6. **Sem offline clínico.** O app funciona offline apenas para leitura em cache; escrita clínica exige conexão.
7. **Falha segura.** Sem permissão, sem dado. Sem conexão, sem gravação. Sem backup verificado, backup não existe.
8. **Evolução incremental.** Cada fase termina com o sistema executável, testado e documentado.

---

## 2. Visão geral

```
┌──────────────────────────────────────────────────────────────────────┐
│ POSTO DE TRABALHO (Windows 10/11)                                    │
│                                                                      │
│  Tauri 2 (Rust)                                                      │
│  ├── WebView2 ─ React 19 + TypeScript + Tailwind + shadcn/ui         │
│  ├── Keychain do SO (refresh token)                                  │
│  ├── Updater assinado                                                │
│  └── Ponte HTTP (somente HTTPS)                                      │
└───────────────────────────────────┬──────────────────────────────────┘
                                    │ HTTPS / JSON / JWT
                                    ▼
┌──────────────────────────────────────────────────────────────────────┐
│ SERVIDOR                                                             │
│                                                                      │
│  Reverse proxy (Caddy/Nginx) ─ TLS, HSTS, rate limit, logs           │
│        │                                                             │
│        ▼                                                             │
│  API NestJS 11 (Fastify)                                             │
│  ├── Auth · Sessões · MFA · Recuperação de senha                     │
│  ├── RBAC + permissões individuais                                   │
│  ├── Módulos de domínio (pacientes, agenda, prontuário, ...)         │
│  ├── Auditoria · Logs estruturados · OpenAPI                         │
│  ├── Geração de PDF · Assinatura eletrônica simples                  │
│  └── Storage de anexos (S3/MinIO ou filesystem protegido)            │
│        │                                                             │
│        ├──▶ PostgreSQL 16 (RLS, constraints, auditoria, backups)     │
│        └──▶ Object storage (anexos, PDFs, versões)                   │
└──────────────────────────────────────────────────────────────────────┘
```

**Por que cliente + servidor:** vários profissionais precisam ver o mesmo paciente. Banco por computador criaria ilhas de dados, duplicidade e impossibilidade de auditoria.

---

## 3. Stack e justificativas

| Camada | Escolha | Versão-alvo | Por quê |
|---|---|---|---|
| Shell desktop | **Tauri 2** | 2.x | Instalador leve, WebView2 nativo, Rust para recursos do SO, updater assinado |
| UI | **React + TypeScript** | React 19 / TS 5.7+ | Ecossistema, tipagem, produtividade |
| Build | **Vite** | 6.x | Rápido, padrão do ecossistema |
| Estilo | **Tailwind CSS** | 4.x | Tokens, consistência, produtividade |
| Componentes | **shadcn/ui + Radix** | atual | Acessibilidade, controle total do código |
| Ícones | **Lucide** | atual | Consistência e leveza |
| Rotas | **TanStack Router** | 1.x | Rotas tipadas, guards, busca tipada |
| Estado servidor | **TanStack Query** | 5.x | Cache, invalidação, retry, estados de rede |
| Estado de UI | **Zustand** | 5.x | Simples, sem boilerplate |
| Formulários | **react-hook-form + zod** | atuais | Validação compartilhada com o backend |
| API | **NestJS 11 + Fastify** | 11.x | Módulos, DI, guards, testabilidade, OpenAPI |
| ORM | **Prisma** | 6.x | Tipagem, migrações versionadas, produtividade |
| Banco | **PostgreSQL** | 16+ | Relacional, JSONB, RLS, constraints, `pg_trgm`, `btree_gist` |
| Hash de senha | **argon2id** | — | Padrão atual recomendado |
| Tokens | **JWT curto + refresh rotativo** | — | Sessão revogável e rastreável |
| PDF | **Playwright/Chromium (HTML→PDF)** | — | Layout fiel, templates versionados |
| Storage | **MinIO (S3) ou filesystem** | — | Anexos fora do banco, URLs assinadas |
| Logs | **Pino** | — | Estruturado, com *redaction* de PHI |
| Testes | **Vitest, Supertest, Testcontainers, Playwright** | — | Unidade, integração real e E2E |
| CI/CD | **GitHub Actions** | — | Lint, testes, build, instalador |
| Containers | **Docker Compose** | — | Ambiente reproduzível |

### 3.1 Decisões conscientes

- **Prisma em vez de TypeORM:** melhor tipagem e migrações; RLS e triggers entram por SQL versionado no mesmo fluxo de migração.
- **Fastify em vez do Express padrão:** melhor throughput e validação; o NestJS mantém a estrutura.
- **Tailwind 4 em vez de 3.4:** tokens nativos e melhor performance; shadcn/ui já suporta.
- **TanStack Router em vez de React Router:** rotas e parâmetros tipados, guards declarativos.
- **PDF no servidor:** o documento oficial precisa ser idêntico para todos, versionado e com hash — não pode depender do navegador do posto.

---

## 4. Estrutura do monorepo

```
prontuario-pro/
├─ apps/
│  ├─ api/                       # NestJS
│  │  ├─ src/
│  │  │  ├─ main.ts
│  │  │  ├─ app.module.ts
│  │  │  ├─ common/              # guards, interceptors, filters, decorators
│  │  │  ├─ config/              # env validado por zod
│  │  │  ├─ database/            # PrismaService, tenant context, migrations
│  │  │  ├─ modules/
│  │  │  │  ├─ auth/
│  │  │  │  ├─ institutions/
│  │  │  │  ├─ units/
│  │  │  │  ├─ users/
│  │  │  │  ├─ rbac/
│  │  │  │  ├─ patients/
│  │  │  │  ├─ professionals/
│  │  │  │  ├─ specialties/
│  │  │  │  ├─ scheduling/
│  │  │  │  ├─ encounters/
│  │  │  │  ├─ records/
│  │  │  │  ├─ documents/
│  │  │  │  ├─ files/
│  │  │  │  ├─ finance/
│  │  │  │  ├─ reports/
│  │  │  │  ├─ notifications/
│  │  │  │  ├─ audit/
│  │  │  │  └─ settings/
│  │  │  └─ test/
│  │  └─ prisma/
│  │     ├─ schema.prisma
│  │     ├─ migrations/
│  │     └─ seed/
│  └─ desktop/                   # Tauri + React
│     ├─ src/
│     │  ├─ app/                 # rotas, providers, layout
│     │  ├─ features/            # uma pasta por módulo
│     │  ├─ components/          # design system
│     │  ├─ lib/                 # api client, auth, formatters
│     │  ├─ stores/
│     │  └─ styles/
│     └─ src-tauri/
│        ├─ src/
│        ├─ icons/
│        ├─ tauri.conf.json
│        └─ capabilities/
├─ packages/
│  ├─ shared/                    # zod schemas, DTOs, permissões, enums
│  └─ config/                    # tsconfig, eslint, tailwind presets
├─ infra/
│  ├─ docker-compose.yml
│  ├─ caddy/
│  ├─ scripts/                   # backup, restore, verify
│  └─ postgres/init/
├─ docs/
└─ .github/workflows/
```

**Regra de dependência:** `desktop → shared`, `api → shared`. O `shared` não importa nada de `api` nem de `desktop`.

---

## 5. Aplicativo desktop

### 5.1 Responsabilidades do shell Tauri

- Abrir a janela principal (1440×900, mínimo 1180×720).
- Guardar o **refresh token** no cofre do sistema operacional (plugin de keychain/stronghold), nunca em `localStorage`.
- Verificar e aplicar atualizações assinadas.
- Detectar estado de rede e exibir banner de “sem conexão”.
- Instância única (evita dois logins concorrentes no mesmo posto).
- Abrir PDFs no visualizador padrão.
- Bloquear navegação para origens não autorizadas (CSP + allowlist).

### 5.2 Camadas do frontend

```
features/<modulo>/
├── components/     # UI específica
├── hooks/          # queries/mutations
├── api.ts          # chamadas tipadas
├── schemas.ts      # zod (re-export do shared quando possível)
└── routes.tsx      # rota tipada
```

- Nenhum componente chama `fetch` diretamente; tudo passa por `lib/api-client.ts`.
- `api-client` injeta access token, trata 401 com refresh único, trata 403, 409, 422 e 5xx com mensagens compreensíveis.
- Erros de rede nunca viram “tela branca”; viram estado com ação de retry.
- Cache do TanStack Query é limpo no logout.

### 5.3 Sessão no desktop

```
Login → access token (15 min, memória) + refresh token (30 dias, keychain)
     → renovação silenciosa a cada <15 min
     → logout/revogação → refresh invalidado no servidor + keychain limpo
```

- Access token **nunca** vai para disco.
- Refresh token é rotativo: cada uso invalida o anterior.
- Reutilização de refresh token revogado derruba a família de sessões e gera evento de auditoria.

### 5.4 Offline

| Operação | Offline |
|---|---|
| Abrir o app e ver tela de login | ✅ |
| Ler dados já carregados em cache | ✅ (somente leitura, com selo “dados podem estar desatualizados”) |
| Criar/editar paciente, agenda, prontuário | ❌ bloqueado com mensagem clara |
| Emitir documento/assinar | ❌ bloqueado |
| Sincronizar depois | ❌ não existe na v1 |

Motivo: gravação clínica offline exige resolução de conflitos, ordenação causal e rastreabilidade — não será improvisada.

---

## 6. API (NestJS)

### 6.1 Camadas

```
Controller  → valida DTO, aplica guards, retorna contrato
Service     → regra de negócio, transação, autorização fina
Repository  → Prisma, escopo por instituição
Database    → RLS como segunda barreira
```

- Controllers não contêm regra de negócio.
- Services recebem o **contexto de execução** (`userId`, `institutionId`, `permissions`, `requestId`) por `AsyncLocalStorage`.
- Toda escrita relevante abre transação e grava auditoria na mesma transação.
- Nenhum endpoint retorna entidade crua do Prisma; sempre um DTO de resposta.

### 6.2 Convenções de API

- Base: `/api/v1`.
- JSON, datas em ISO 8601 UTC (`timestamptz` no banco).
- Paginação por cursor (`?cursor=&limit=`), máximo 100.
- Filtros explícitos e validados; nada de SQL concatenado.
- Erros no formato:

```json
{
  "error": {
    "code": "PATIENT_DUPLICATE_CPF",
    "message": "Já existe um paciente com este CPF nesta instituição.",
    "requestId": "req_...",
    "details": [{ "field": "cpf", "issue": "duplicate" }]
  }
}
```

- OpenAPI gerado automaticamente em `/api/docs` (protegido fora de desenvolvimento).
- Idempotência em operações críticas via header `Idempotency-Key` (pagamentos, finalização de documento).

### 6.3 Superfície inicial de endpoints

| Módulo | Endpoints principais |
|---|---|
| Auth | `POST /auth/login`, `/auth/refresh`, `/auth/logout`, `/auth/password/forgot`, `/auth/password/reset`, `/auth/mfa/enroll`, `/auth/mfa/verify`, `GET /auth/me` |
| Instituições | `GET/POST/PATCH /institutions`, `GET/POST /institutions/:id/units` |
| Usuários | `GET/POST/PATCH /users`, `POST /users/:id/roles`, `PUT /users/:id/permissions`, `POST /users/:id/block` |
| RBAC | `GET /roles`, `POST /roles`, `PUT /roles/:id/permissions`, `GET /permissions` |
| Pacientes | `GET/POST/PATCH /patients`, `GET /patients/:id`, `POST /patients/:id/merge`, `GET /patients/:id/timeline`, `POST /patients/:id/files` |
| Profissionais | `GET/POST/PATCH /professionals`, `GET /professionals/:id/availability` |
| Especialidades | `GET/POST/PATCH /specialties` |
| Agenda | `GET/POST/PATCH /appointments`, `POST /appointments/:id/confirm`, `/cancel`, `/check-in`, `/no-show`, `GET /schedule?from=&to=&professionalId=` |
| Atendimentos | `GET/POST /encounters`, `POST /encounters/:id/start`, `/finish` |
| Prontuário | `GET /patients/:id/record`, `POST /records/:id/anamnesis`, `POST /records/:id/evolutions`, `POST /evolutions/:id/finalize`, `POST /evolutions/:id/amend`, `GET /evolutions/:id/revisions` |
| Sinais vitais | `POST /encounters/:id/vital-signs`, `GET /patients/:id/vital-signs` |
| Diagnósticos | `POST /records/:id/diagnoses`, `PATCH /diagnoses/:id/resolve` |
| Prescrições | `POST /records/:id/prescriptions`, `GET /prescriptions/:id/pdf` |
| Documentos | `POST /documents`, `GET /documents/:id`, `POST /documents/:id/finalize`, `GET /documents/:id/pdf`, `POST /documents/:id/sign` |
| Arquivos | `POST /files`, `GET /files/:id/url`, `DELETE /files/:id` |
| Financeiro | `GET/POST /services`, `GET/POST /invoices`, `POST /invoices/:id/payments`, `GET /receipts/:id/pdf` |
| Relatórios | `GET /reports/appointments`, `/reports/productivity`, `/reports/finance`, `/reports/patients` |
| Auditoria | `GET /audit-logs`, `GET /record-access-logs` |
| Configurações | `GET/PUT /settings` |
| Notificações | `GET /notifications`, `POST /notifications/:id/read` |
| Saúde | `GET /health`, `GET /health/ready` |

---

## 7. Multi-instituição e isolamento

### 7.1 Modelo

- `institutions` é o *tenant*.
- Toda tabela operacional carrega `institution_id NOT NULL`.
- O usuário pertence a **uma** instituição (exceto superadministrador de plataforma).
- O superadministrador **não** acessa PHI por padrão; quando necessário, o acesso é explícito, justificado e auditado.

### 7.2 Três barreiras

1. **Token:** `institution_id` vem assinado no JWT e é conferido pelo guard.
2. **Serviço:** toda query recebe `institutionId` do contexto, nunca do corpo da requisição.
3. **Banco (RLS):** políticas comparam `institution_id` com `current_setting('app.institution_id')`.

```sql
ALTER TABLE patients ENABLE ROW LEVEL SECURITY;

CREATE POLICY patients_tenant_isolation ON patients
  USING (institution_id = current_setting('app.institution_id')::uuid)
  WITH CHECK (institution_id = current_setting('app.institution_id')::uuid);
```

### 7.3 Como o contexto chega ao banco

O Prisma usa transação interativa para operações com RLS:

```ts
await prisma.$transaction(async (tx) => {
  await tx.$executeRaw`SELECT set_config('app.institution_id', ${institutionId}, true)`;
  await tx.$executeRaw`SELECT set_config('app.user_id', ${userId}, true)`;
  return tx.patient.findMany({ where: { deletedAt: null } });
});
```

- `set_config(..., true)` é local à transação; não vaza entre conexões do pool.
- O usuário de aplicação **não** é superusuário e **não** tem `BYPASSRLS`.
- Migrações rodam com usuário separado (`migrator`).

### 7.4 Testes obrigatórios de isolamento

- Usuário da instituição A não lê, altera, lista, exporta nem referencia IDs da instituição B.
- Tentativa de acesso cruzado retorna **404** (não 403), para não revelar existência.
- Toda tentativa cruzada gera evento de segurança.

---

## 8. Autenticação e sessões

### 8.1 Fluxo

```
POST /auth/login
  ├── valida e-mail/senha (argon2id)
  ├── verifica bloqueio, status e tentativas
  ├── exige MFA se habilitado/perfil administrativo
  ├── cria sessão (refresh hash + metadados do dispositivo)
  └── retorna access JWT (15 min) + refresh opaco (30 dias)

POST /auth/refresh
  ├── valida hash do refresh
  ├── rotaciona (invalida o anterior)
  └── retorna novo par

POST /auth/logout
  └── revoga a sessão atual
```

### 8.2 Controles

| Controle | Regra |
|---|---|
| Senha | mínimo 12 caracteres, bloqueio de senhas comuns, argon2id |
| Tentativas | 5 falhas → bloqueio progressivo (1, 5, 15, 60 min) |
| Mensagens | login sempre “credenciais inválidas” (não revela e-mail) |
| MFA | TOTP obrigatório para superadmin/admin; opcional para os demais |
| Sessões | listagem e revogação pelo próprio usuário; admin revoga por usuário |
| Recuperação | token de uso único, hash no banco, expiração de 30 min, invalida sessões |
| JWT | RS256 ou HS256 com segredo forte; `jti`, `iat`, `exp`, `institution_id`, `roles`, `permissions` |
| Transporte | HTTPS obrigatório; HSTS; TLS 1.2+ |
| Auditoria | login, falha, bloqueio, MFA, logout, revogação |

### 8.3 Credenciais

- Segredos apenas em variáveis de ambiente / cofre.
- `.env` nunca versionado; `.env.example` sem valores reais.
- Rotação de chaves documentada.
- Nenhuma chave de Supabase, banco ou storage no frontend.

---

## 9. Autorização: RBAC + permissões individuais

### 9.1 Chaves de permissão

Formato `recurso.ação`, por exemplo:

```
patients.read            patients.write           patients.merge
schedule.read            schedule.write           schedule.manage
encounters.read          encounters.write         encounters.finish
records.read             records.write            records.finalize
records.amend            records.export
documents.read           documents.issue          documents.sign
prescriptions.issue      exams.request
professionals.read       professionals.write
finance.read             finance.write            finance.export
reports.read             reports.export
users.read               users.write              users.permissions
institutions.manage      settings.manage          audit.read
backup.manage            platform.admin
```

### 9.2 Avaliação

```
permissão efetiva =
    (permissões dos perfis do usuário)
  + (allow explícito em user_permissions)
  − (deny explícito em user_permissions)
```

- `deny` individual **sempre** vence.
- Administrador de instituição só pode conceder permissões que ele próprio possui.
- Superadministrador não pode ser criado por administrador de instituição.
- A UI recebe a lista efetiva em `GET /auth/me` e esconde o que não pode; a API **sempre** revalida.

### 9.3 Matriz inicial de perfis

| Perfil | Núcleo de permissões |
|---|---|
| Superadministrador | `platform.admin`, instituições, auditoria global; sem PHI por padrão |
| Administrador da instituição | usuários, permissões, unidades, especialidades, configurações, relatórios |
| Gestor | indicadores, relatórios, agenda, profissionais (leitura/gestão operacional) |
| Recepcionista | pacientes (criar/editar), agenda, check-in, fila; sem prontuário clínico |
| Profissional de saúde | prontuário dos próprios atendimentos/pacientes autorizados, evoluções, documentos |
| Supervisor | leitura e validação de registros, solicitação de correção |
| Financeiro | serviços, cobranças, pagamentos, relatórios financeiros |
| Auditor | leitura autorizada + trilhas de auditoria; sem escrita clínica |

---

## 10. Auditoria e imutabilidade

### 10.1 O que é auditado

- Login, logout, falha, bloqueio, MFA.
- Criação/edição de usuário, perfil, permissão.
- Acesso a prontuário (leitura de PHI) — em tabela própria de alto volume.
- Criação, finalização, adendo, impressão e exportação de documento.
- Alteração de paciente, agenda, atendimento.
- Operações financeiras.
- Backup, restauração e alteração de configuração.
- Tentativas de acesso cruzado entre instituições.

### 10.2 Estrutura do evento

```json
{
  "id": "evt_...",
  "institutionId": "uuid",
  "actorUserId": "uuid",
  "actorRole": "profissional",
  "action": "record.evolution.finalize",
  "entityType": "clinical_evolution",
  "entityId": "uuid",
  "patientId": "uuid",
  "requestId": "req_...",
  "ip": "203.0.113.10",
  "userAgent": "ProntuarioPro/1.0 (Windows)",
  "metadata": { "revision": 2, "hash": "sha256:..." },
  "createdAt": "2026-11-08T14:32:11Z"
}
```

- **Sem PHI no log.** Metadados, IDs, nomes de campos e hashes — nunca o conteúdo clínico.
- `audit_logs` é *append-only*: sem `UPDATE`, sem `DELETE` pela aplicação.
- Particionada por mês; retenção definida em política.

### 10.3 Imutabilidade clínica

- Rascunho (`draft`) pode ser editado pelo autor.
- Ao finalizar: grava `content_hash` (SHA-256), `finalized_at`, `finalized_by`, congela a versão.
- Trigger no banco impede `UPDATE` de conteúdo e `DELETE` em registro finalizado.
- Correção = **novo registro** (`amends_id`) com justificativa obrigatória; o original permanece visível.
- Documento emitido guarda versão, hash e assinatura; reemissão gera nova versão.

---

## 11. Arquivos, PDF e assinatura

### 11.1 Anexos

- Upload via API (multipart) → valida tamanho, MIME real (magic bytes) e extensão.
- Nome original preservado apenas como metadado; o objeto é gravado com UUID.
- Chave: `institutions/<institution_id>/patients/<patient_id>/<uuid>`.
- Download por URL assinada de curta duração; nunca bucket público.
- Hash SHA-256 registrado; opção de antivírus (ClamAV) antes de liberar.
- Exclusão é lógica, auditada; retenção conforme política.

### 11.2 PDF

- HTML versionado + Chromium headless no servidor.
- Cabeçalho com instituição/unidade, identificação do paciente, título, data/hora, autor e registro profissional.
- Rodapé com página, hash curto e código de verificação.
- PDF armazenado como arquivo imutável vinculado ao documento.
- Impressão no desktop = abrir o PDF; o layout não depende da máquina.

### 11.3 Assinatura — distinção obrigatória

| Tipo | O que é | Como será tratado |
|---|---|---|
| **Assinatura eletrônica simples** | Autenticação + registro de autoria + hash + auditoria (Lei 14.063/2020, nível simples) | Implementada na v1, rotulada como **“assinatura eletrônica simples”** |
| **Assinatura digital qualificada ICP-Brasil** | Certificado A1/A3, padrão PAdES/CAdES | Fora da v1; integração futura com biblioteca e certificado do profissional |

A interface **nunca** chamará a assinatura simples de “assinatura digital” nem exibirá selo de conformidade não obtido.

---

## 12. Backup e recuperação

| Item | Decisão |
|---|---|
| Banco | `pgBackRest` (full + incremental + WAL) ou `pg_dump` diário no mínimo |
| Arquivos | versionamento no bucket + replicação para destino secundário |
| Frequência | diária completa; WAL contínuo para RPO baixo |
| Retenção | 30 diários, 12 mensais, 5 anuais (configurável) |
| Criptografia | backup criptografado em repouso |
| Verificação | `pg_restore --list` + restauração automática em ambiente de teste + contagem/checksum |
| Registro | cada backup/restauração gera evento de auditoria |
| Regra | **backup não verificado = backup inexistente** |

Scripts em `infra/scripts/` com `backup.sh`, `restore.sh`, `verify-backup.sh` e agendamento documentado.

---

## 13. Ambientes e implantação

| Ambiente | Uso | Dados |
|---|---|---|
| `local` | desenvolvimento | seed sintético, nunca PHI real |
| `test` | CI | efêmero (Testcontainers) |
| `staging` | homologação | dados sintéticos ou anonimizados |
| `production` | clínica | dados reais, TLS, backup, monitoramento |

- **Docker Compose** para API + PostgreSQL + MinIO + proxy.
- Instalação em servidor Windows Server ou Linux; o desktop só precisa de HTTPS.
- Configuração por ambiente validada com zod na inicialização (falha rápido se faltar segredo).
- Migrações rodam no deploy, com backup prévio e *lock*.
- Atualização do desktop por updater assinado, com canal `stable`/`beta`.

---

## 14. Observabilidade

- Logs JSON (Pino) com `requestId`, `userId`, `institutionId`, latência e status.
- **Redaction** de Authorization, senha, tokens e campos clínicos.
- Health checks: liveness e readiness (banco + storage).
- Métricas: latência p95, erros 5xx, falhas de login, fila de PDF, uso de conexões.
- Alertas: banco indisponível, backup falhou, disco > 80%, pico de 401/403.
- Trilha de auditoria separada de log técnico.

---

## 15. Estratégia de testes

| Nível | Ferramenta | O que cobre |
|---|---|---|
| Unidade | Vitest / Jest | regras de permissão, validações, cálculos, mascaramento |
| Integração | Supertest + Testcontainers PostgreSQL | endpoints reais, RLS, transações, constraints, auditoria |
| Contrato | OpenAPI + schemas zod | compatibilidade API ↔ desktop |
| E2E web | Playwright | login, paciente, agenda, evolução, finalização, PDF |
| E2E desktop | Playwright + build Tauri | smoke de instalação, abertura, login, offline |
| Segurança | suíte dedicada | isolamento entre instituições, escalada de privilégio, IDOR, injeção |
| Backup | script de verificação | restauração real em base limpa |

**Cobertura mínima:** 80% em módulos de domínio; 100% das permissões críticas com teste explícito.

---

## 16. CI/CD

```
push/PR
 ├── lint + typecheck
 ├── testes unitários
 ├── testes de integração (PostgreSQL service)
 ├── build da API
 ├── build web do desktop
 └── build Tauri (windows-latest) → artefato .msi/.exe

tag v*
 ├── gera instalador assinado
 ├── publica release
 └── atualiza manifesto do updater
```

- Nenhum merge sem CI verde.
- Nenhum segredo em log.
- Dependências auditadas (`npm audit`/Dependabot).

---

## 17. Requisitos não funcionais

| Requisito | Meta |
|---|---|
| Tempo de resposta p95 (leitura) | < 400 ms |
| Tempo de resposta p95 (escrita) | < 800 ms |
| Abertura do app | < 3 s em máquina comum |
| Disponibilidade alvo | 99,5% (single server) |
| RPO | ≤ 15 min com WAL; ≤ 24 h no mínimo aceitável |
| RTO | ≤ 4 h |
| Capacidade inicial | 50 usuários simultâneos, 100 mil pacientes |
| Acessibilidade | WCAG 2.1 AA nos fluxos principais |
| Idiomas | pt-BR (estrutura preparada para i18n) |
| Formatos | A4, PDF/A quando aplicável |

---

## 18. Direção visual (resumo)

Identidade própria, sem reutilizar a paleta azul/verde da referência:

- Branco predominante, superfícies cinza-claro, **magenta como cor de destaque**.
- Menu lateral fixo com módulos; cabeçalho com instituição, unidade, busca global e usuário.
- Tipografia Inter; números tabulares em tabelas.
- Cards com borda suave, sombra discreta, raio 10–12 px.
- Estados claros: sucesso, atenção, erro, informação, rascunho, finalizado, cancelado.
- Detalhes em `07-DIRECAO-VISUAL.md`.

---

## 19. Premissas e pontos que dependem de decisão

1. **Hospedagem:** servidor próprio da clínica, VPS contratada ou nuvem? Muda backup, TLS e custo.
2. **Assinatura:** apenas eletrônica simples na v1 (recomendado) ou já prever ICP-Brasil?
3. **Financeiro:** controle de recebíveis próprio ou integração futura com gateway/ERP?
4. **Convênios/TISS:** a v1 precisa de faturamento TISS ou apenas particular/convênio simples?
5. **Migração de dados:** existe base legada a importar? (Se sim, precisamos de um mapeamento.)
6. **Multi-instituição:** o produto será vendido como instalação dedicada por clínica (single-tenant físico) ou SaaS multi-tenant? A arquitetura suporta ambos; a operação muda.
7. **Certificado de servidor:** domínio próprio para HTTPS é requisito para produção.

Nenhuma dessas decisões bloqueia a Fase 3 (fundação), mas devem ser fechadas antes da Fase 9.
