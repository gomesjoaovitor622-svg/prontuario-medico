# Plano de Implementação — PRONTUÁRIO PRO

**Versão:** 1.0
**Data:** 08/11/2026
**Estado:** proposta para aprovação (Portão G2)

---

## 1. Estratégia

O sistema será construído em **fases com portões de qualidade**. Cada fase termina com:

- código executável;
- migrações aplicáveis do zero;
- testes automatizados verdes;
- documentação atualizada;
- demonstração do fluxo ao responsável.

Nenhuma fase começa antes do aceite da anterior. Nenhum módulo é “fachada”: botão sem ação, tela sem persistência ou dado mock em produção são tratados como defeito bloqueante.

### 1.1 Ordem de valor

```
Fundação (segurança + dados)
   → Identidade (login, perfis, permissões)
      → Pacientes + Profissionais
         → Agenda + Atendimento
            → Prontuário (coração do produto)
               → Documentos + PDF
                  → Administração + Auditoria
                     → Financeiro + Relatórios
                        → Desktop empacotado + hardening
```

O prontuário vem **antes** do financeiro porque é o núcleo clínico e o maior risco técnico/legal.

---

## 2. Fase 0 — Preparação do ambiente (1–2 dias)

### Entregáveis

- Workspace de desenvolvimento fora do OneDrive (recomendado `C:\dev\prontuario-pro`).
- Node 24 + npm/pnpm funcionando (política de execução ajustada ou uso de `npm.cmd`).
- Rust + toolchain MSVC + WebView2 (Tauri).
- PostgreSQL 16 local **ou** Docker Desktop com Compose.
- Git configurado; repositório remoto conectado.
- CI inicial (lint + typecheck).

### Critérios de aceite

- `cargo --version`, `psql --version` (ou `docker compose version`) e `npm.cmd -v` respondem.
- `npm.cmd run build` executa em um projeto vazio.
- Push no GitHub funciona sem expor token.

### Bloqueios atuais

| Item | Estado |
|---|---|
| Rust/cargo | ❌ não instalado |
| PostgreSQL | ❌ não instalado |
| Docker | ❌ não instalado |
| `gh` CLI | ❌ não instalado (push pode ser via Git + PAT) |
| PowerShell `.ps1` | ❌ bloqueado (usar `npm.cmd` ou ajustar política) |
| WebView2 | ✅ instalado (157.x) |
| Node/npm | ✅ Node 24 / npm 11.17 (`npm.cmd`) |
| Git | ✅ 2.54 |

---

## 3. Fase 1 — Auditoria da referência ✅ concluída

**Entrega:** `docs/01-AUDITORIA-REFERENCIA.md`

- Stack, telas, fluxos, design, limitações e riscos da referência.
- Conclusão: referência é protótipo web com dados em memória; nada de código será reutilizado.

**Portão G1:** aceite da auditoria.

---

## 4. Fase 2 — Arquitetura e modelo de dados ⏳ em aprovação

**Entrega:** `docs/02-ARQUITETURA.md`, `docs/03-MODELO-DADOS.md`, `docs/04-PLANO-IMPLEMENTACAO.md`, `docs/05-RISCOS-E-SEGURANCA.md`, `docs/06-ADRS.md`, `docs/07-DIRECAO-VISUAL.md`.

**Portão G2:** aprovação da arquitetura, do modelo e do plano.

---

## 5. Fase 3 — Fundação (2–3 semanas)

### Entregáveis

- Monorepo criado (`apps/api`, `apps/desktop`, `packages/shared`).
- API NestJS com configuração validada, health check, tratamento global de erros e logs.
- PostgreSQL 16 + Prisma + migração inicial completa.
- RLS aplicada e testada.
- Autenticação: login, refresh rotativo, logout, recuperação de senha, bloqueio por tentativas, MFA TOTP.
- RBAC + permissões individuais (allow/deny) com catálogo seed.
- Auditoria de login e de operações administrativas.
- Seed de instituição, unidade, perfis e superadministrador.
- Suíte de testes de integração com Testcontainers.

### Critérios de aceite

- Login funciona ponta a ponta (API + teste).
- Token expirado renova sem perder a sessão.
- Refresh reutilizado revoga a família e gera auditoria.
- Usuário bloqueado não autentica.
- Permissão `deny` vence `allow`.
- RLS comprovada: instituição A não enxerga B em nenhuma tabela.
- Migração roda do zero em base limpa e é idempotente no seed.

**Portão G3.**

---

## 6. Fase 4 — Casca desktop e design system (1–2 semanas)

### Entregáveis

- Tauri 2 configurado (janela, ícone, CSP, instância única, updater preparado).
- React + Vite + Tailwind 4 + shadcn/ui com tokens do produto.
- Layout: menu lateral, cabeçalho, breadcrumb, área de conteúdo.
- Rotas protegidas por autenticação e permissão.
- Tela de login, troca de senha obrigatória, MFA.
- Componentes base: tabela paginada, formulário, modal, drawer, toast, estados vazio/erro/carregando, badges de status.
- Cliente HTTP com refresh, tratamento de erros e estado de conexão.
- Tema claro (escuro opcional depois).
- Testes de componente e E2E de login.

### Critérios de aceite

- App abre em < 3 s.
- Login real contra a API, com refresh e logout.
- Menu reflete permissões efetivas.
- Nenhum botão sem ação.
- Acessibilidade básica: foco, teclado, labels, contraste AA.

**Portão G3.5.**

---

## 7. Fase 5 — Pacientes, profissionais e especialidades (2–3 semanas)

### Entregáveis

- Cadastro completo de paciente (dados pessoais, documentos, contatos, endereço, responsável, convênio, observações).
- Validação de CPF/CNS, normalização e detecção de duplicidade.
- Busca por nome/CPF com paginação e filtros.
- Histórico cadastral e linha do tempo do paciente.
- Anexos do paciente com URL assinada.
- Fusão de pacientes duplicados com trilha.
- CRUD de profissionais, especialidades, unidades e vínculos.
- Disponibilidade e bloqueios de agenda.
- Permissões aplicadas e testadas por perfil.

### Critérios de aceite

- Recepcionista cadastra e edita paciente, mas não acessa prontuário.
- Profissional vê apenas pacientes autorizados.
- CPF duplicado é bloqueado com mensagem clara.
- Anexo não é acessível por URL pública.
- Toda alteração cadastral gera auditoria.

**Portão G4a.**

---

## 8. Fase 6 — Agenda e atendimentos (2–3 semanas)

### Entregáveis

- Calendário diário, semanal e mensal.
- Agendamento, reagendamento, confirmação, cancelamento, encaixe e falta.
- Filtros por profissional, especialidade e unidade.
- Bloqueio de horários e disponibilidade.
- Constraint anti-overbooking no banco.
- Fila de espera e check-in.
- Atendimento: aberto → em andamento → finalizado/cancelado.
- Vínculo atendimento ↔ paciente ↔ profissional ↔ unidade ↔ especialidade.
- Notificações internas de agenda.

### Critérios de aceite

- Não é possível sobrepor dois agendamentos do mesmo profissional.
- Reagendamento preserva o histórico.
- Check-in muda status e registra autor/hora.
- Atendimento só finaliza com prontuário consistente.
- Todos os fluxos têm teste de integração.

**Portão G4b.**

---

## 9. Fase 7 — Prontuário eletrônico (4–6 semanas) — núcleo

### Entregáveis

- Prontuário por paciente com identificação, alertas (alergias), histórico e linha do tempo.
- Anamnese estruturada (queixa, HDA, antecedentes, hábitos, alergias, medicamentos).
- Sinais vitais com faixas e gráfico de evolução.
- Evolução clínica com modelos por especialidade, campos estruturados + texto livre, rascunho e finalização.
- Diagnósticos CID-10 com status (ativo/suspeito/resolvido).
- Prescrições (simples e controladas, respeitando habilitação).
- Encaminhamentos e solicitações de exames.
- Correção por adendo com justificativa e revisões.
- Hash, autoria, data/hora do servidor e imutabilidade.
- Controle de acesso por paciente/atendimento.
- Registro de leitura de prontuário (`record_access_logs`).

### Critérios de aceite

- Rascunho é editável; finalizado não pode ser alterado nem apagado (teste no banco).
- Adendo cria revisão nova e mantém o original visível.
- Hash confere com o conteúdo.
- Profissional não acessa prontuário de paciente fora do seu escopo.
- Auditor registra leitura sem poder escrever.
- Nenhum dado clínico é gerado automaticamente como fato.

**Portão G4c — marco clínico.**

---

## 10. Fase 8 — Documentos, PDF e assinatura (2–3 semanas)

### Entregáveis

- Modelos de documento por especialidade e instituição.
- Geração de PDF no servidor (ficha, anamnese, evolução, relatório, atestado, declaração, encaminhamento, prescrição).
- Versionamento de documento e hash.
- Assinatura eletrônica simples com registro de autoria, IP, dispositivo e auditoria.
- Rótulo explícito distinguindo assinatura simples de digital qualificada.
- Impressão a partir do PDF.
- Exportação autorizada e auditada.

### Critérios de aceite

- PDF idêntico em qualquer posto.
- Documento reemitido gera nova versão, sem apagar a anterior.
- Assinatura simples não é exibida como ICP-Brasil.
- Exportação exige permissão e gera auditoria.

**Portão G5a.**

---

## 11. Fase 9 — Administração, permissões e auditoria (2 semanas)

### Entregáveis

- Gestão de instituições, unidades e departamentos.
- Gestão de usuários, bloqueio/desbloqueio, reset de senha, MFA.
- Gestão de perfis e permissões individuais com pré-visualização do efeito.
- Tela de auditoria com filtros por usuário, ação, paciente e período.
- Tela de sessões ativas e revogação.
- Configurações da instituição (dados, agenda, notificações, documentos).
- Política de senha e segurança.

### Critérios de aceite

- Administrador não consegue se autopromover a superadmin.
- Administrador não concede permissão que não possui.
- Toda mudança de permissão é auditada.
- Auditoria é somente leitura para a aplicação.

**Portão G5b.**

---

## 12. Fase 10 — Financeiro e relatórios (2–3 semanas)

### Entregáveis

- Serviços e tabela de preços.
- Cobranças, itens, pagamentos, recibos.
- Contas a receber e status.
- Controle por profissional e unidade.
- Relatórios: atendimentos, pacientes, agendamentos, faltas, produtividade, especialidades, unidades, financeiro.
- Filtros, paginação e exportação autorizada.
- Idempotência em pagamentos.

### Critérios de aceite

- Valores em centavos, sem erro de arredondamento.
- Pagamento duplicado com mesma chave de idempotência não duplica.
- Financeiro não acessa prontuário clínico.
- Relatórios respeitam permissão e instituição.

**Portão G6a.**

---

## 13. Fase 11 — Desktop, backup e hardening (2–3 semanas)

### Entregáveis

- Instalador Windows (MSI/NSIS) assinado.
- Ícone, splash, versão, updater assinado.
- Detecção de rede e modo somente leitura.
- Backup automático + manual + verificação de restauração.
- Scripts de restore documentados e testados.
- Hardening: rate limit, CSP, headers, validação, auditoria, revisão de dependências.
- Teste de isolamento entre instituições em toda a API.
- Teste de escalada de privilégio e IDOR.
- Documentação técnica e operacional.

### Critérios de aceite

- Instalador instala, abre, loga e desinstala em Windows 10 e 11 limpos.
- Restauração de backup em base limpa é comprovada com contagem e checksum.
- Nenhuma credencial no repositório ou no instalador.
- Relatório de testes executados (não declarados).

**Portão G6b.**

---

## 14. Fase 12 — Validação e entrega (1–2 semanas)

### Entregáveis

- Teste por perfil (8 perfis) com roteiro e evidências.
- Teste de permissões e isolamento.
- Teste de cadastros, agenda, atendimento, prontuário, documentos, financeiro.
- Teste de backup/restauração e integridade.
- Teste de instalação e atualização.
- Manual de instalação, utilização, backup e restauração.
- Relatório de testes.
- Checklist de segurança e LGPD para validação jurídica.

**Portão G7 — aceite final.**

---

## 15. Cronograma macro (referência)

| Fase | Duração estimada | Acumulado |
|---|---:|---:|
| 0 — Ambiente | 1–2 dias | 2 d |
| 1 — Auditoria | concluída | — |
| 2 — Arquitetura | concluída | — |
| 3 — Fundação | 2–3 semanas | ~3 sem |
| 4 — Casca desktop | 1–2 semanas | ~5 sem |
| 5 — Pacientes/profissionais | 2–3 semanas | ~8 sem |
| 6 — Agenda/atendimentos | 2–3 semanas | ~11 sem |
| 7 — Prontuário | 4–6 semanas | ~17 sem |
| 8 — Documentos/PDF | 2–3 semanas | ~20 sem |
| 9 — Administração | 2 semanas | ~22 sem |
| 10 — Financeiro/relatórios | 2–3 semanas | ~25 sem |
| 11 — Desktop/backup/hardening | 2–3 semanas | ~28 sem |
| 12 — Validação/entrega | 1–2 semanas | ~30 sem |

> Estimativas para uma equipe pequena (2–4 pessoas). O ritmo real depende de validações jurídicas, infraestrutura e disponibilidade para homologação.

---

## 16. Definição de pronto (por história)

Uma funcionalidade só é considerada pronta quando:

1. Tem contrato de API documentado.
2. Tem validação de entrada no servidor.
3. Tem autorização testada.
4. Persiste em PostgreSQL (nada em memória).
5. Gera auditoria quando sensível.
6. Tem teste unitário e de integração.
7. Tem tratamento de erro e estado de carregamento na UI.
8. Funciona com teclado e leitor de tela no fluxo principal.
9. Não expõe PHI em log.
10. Foi demonstrada em ambiente de homologação.

---

## 17. O que preciso de você para começar a Fase 3

1. **Aprovação** deste conjunto de documentos (Portão G2).
2. **Local do código:** manter em `Projeto Padrão` (OneDrive) ou mover para `C:\dev\prontuario-pro`? (Recomendo mover.)
3. **Ambiente:** posso instalar Rust/PostgreSQL/Docker via `winget`, ou você prefere instalar manualmente?
4. **GitHub:** o push exige um token/credencial sua. O repositório `prontuario-medico` está vazio e pronto.
5. **Decisões de produto** (podem ficar para depois, mas ajudam): hospedagem, assinatura na v1, TISS, migração de dados.

Com a aprovação, a Fase 3 começa pela **migração inicial do banco + autenticação + RBAC + auditoria**, com testes reais rodando contra PostgreSQL.
