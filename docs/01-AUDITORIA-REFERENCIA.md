# Auditoria da Referência — Pronto Fácil Doctor

**Projeto novo:** PRONTUÁRIO PRO — Gestão Clínica Inteligente
**Artefato auditado:** `pronto-facil-doctor-main (1).zip` (311.983 bytes)
**Data da auditoria:** 08/11/2026
**Método:** extração do ZIP, leitura integral do código-fonte, contagem de arquivos, inspeção de dependências, rotas, migrações, dados e tokens de design.
**Regra:** nada abaixo foi inferido sem evidência no código. O que não pôde ser verificado está marcado como **NÃO VERIFICADO**.

---

## 1. Sumário executivo

O projeto de referência **não é um sistema de gestão clínica**. É uma **SPA de demonstração**, gerada por scaffolding (Lovable/Vite), com **4 telas** e um único fluxo real: escolher um paciente, escolher um modelo de evolução, preencher campos e imprimir.

Tudo o que parece “sistema” — pacientes, evoluções, histórico — vive **em memória no navegador** (`useState` + arrays mock). Ao recarregar a página, os dados desaparecem. O único dado persistido de verdade é o perfil do usuário (nome, CRM, especialidade e avatar), gravado no Supabase.

Não existem: agenda, atendimentos, profissionais, especialidades configuráveis, permissões, auditoria, financeiro, relatórios, backup, multi-instituição, prontuário estruturado, documentos, assinatura ou qualquer API própria. O backend é terceirizado (Supabase Auth + uma tabela `profiles`).

**Conclusão:** o valor da referência está na **organização visual** (lista de pacientes + área de trabalho, formulários guiados por template, prévia de impressão) e na **ideia de evolução estruturada com campos padrão**. Nada de código deve ser copiado — inclusive porque **o pacote não possui arquivo de licença**, o que, por padrão, reserva todos os direitos ao autor. A reconstrução será do zero, com identidade própria e arquitetura própria.

---

## 2. Evidências da auditoria

### 2.1 Inventário

| Métrica | Valor |
|---|---:|
| Arquivos TypeScript/TSX | 79 (63 `.tsx` + 16 `.ts`) |
| Primitivas shadcn/ui | ~50 (a maior parte do volume de código) |
| Páginas reais | 4 (`Auth`, `Index`, `Profile`, `NotFound`) |
| Componentes de domínio | 7 (`PatientCard`, `PatientForm`, `TemplateSelector`, `EvolutionForm`, `EvolutionHistory`, `PrintableEvolution`, `NavLink`) |
| Modelos de evolução | 8 (`general`, `cardio`, `pneumo`, `admissao`, `internacao`, `pos_cirurgico`, `transferencia`, `obito`) |
| Pacientes mock | 5 |
| Evoluções mock | 2 |
| Migrações SQL | 1 (apenas `profiles` + bucket `avatars`) |
| Testes | 1 trivial (`example.test.ts`, 143 bytes) |
| Dependências de runtime | 50 |
| Licença | **ausente** |

### 2.2 Arquivos determinantes

| Arquivo | Papel |
|---|---|
| `src/App.tsx` | 4 rotas, guards de sessão, providers |
| `src/pages/Index.tsx` | Tela principal: lista de pacientes + evolução + impressão |
| `src/pages/Auth.tsx` | Login e cadastro aberto via Supabase |
| `src/pages/Profile.tsx` | Edição de perfil e upload de avatar público |
| `src/data/patients.ts` | 5 pacientes mock em memória |
| `src/data/evolutions.ts` | 2 evoluções mock em memória |
| `src/data/evolutionTemplates.ts` | 8 templates com campos e texto gerado |
| `src/components/EvolutionForm.tsx` | Motor de formulário + geração de texto |
| `src/components/PrintableEvolution.tsx` | Prévia e impressão via `window.open` |
| `supabase/migrations/20260326003032_*.sql` | `profiles`, trigger, RLS e bucket público |
| `src/index.css` | Tokens de design (HSL) |

### 2.3 O que foi verificado

- Leitura de todos os arquivos de domínio, páginas, contexto de autenticação, migração e tokens.
- Contagem de dependências e verificação de uso real: `@tanstack/react-query`, `react-hook-form`, `zod` e `recharts` estão **instalados e não são usados**.
- Busca por `useQuery`/`useMutation`: **0 ocorrências**.
- Busca por `useForm`: **0 ocorrências**.
- Busca por `from "zod"`: **0 ocorrências**.

### 2.4 O que NÃO foi verificado

| Item | Motivo |
|---|---|
| Execução da aplicação no navegador | Auditoria feita por código; não há ambiente Supabase configurado |
| Funcionamento do Supabase remoto | Projeto externo; não foi acessado nem validado |
| Compatibilidade de impressão em impressoras reais | Depende de driver/navegador |
| Dados reais de pacientes | Não existem; a base é mock |
| Build de produção | Não executado nesta auditoria |

---

## 3. Stack tecnológica observada

| Camada | Tecnologia | Observação |
|---|---|---|
| Build | Vite 5 | SPA |
| UI | React 18.3 + TypeScript 5.8 | — |
| Estilo | Tailwind CSS 3.4 + shadcn/ui (Radix) | Base visual consistente |
| Rotas | React Router 6 | 4 rotas |
| Estado servidor | TanStack Query 5 | **instalado, não usado** |
| Formulários | react-hook-form 7 + zod 3 | **instalados, não usados** |
| Backend | Supabase (Auth + Postgres + Storage) | Sem API própria |
| Ícones | lucide-react | Uso consistente |
| Toasts | sonner + shadcn toaster | — |
| Gráficos | recharts | **instalado, não usado** |
| Testes | Vitest + Testing Library + Playwright | Configurados; 1 teste trivial |
| Desktop | **inexistente** | Não é Tauri/Electron; é web puro |
| Multi-instituição | **inexistente** | Uma única base Supabase |

**Observação de segurança:** o `.env` do pacote expõe `VITE_SUPABASE_URL`, `VITE_SUPABASE_PUBLISHABLE_KEY` e `VITE_SUPABASE_PROJECT_ID`. São chaves públicas de cliente (não `service_role`), mas **não devem ser reutilizadas**. O projeto novo terá backend próprio e nenhuma credencial versionada.

---

## 4. Arquitetura real da referência

```
Navegador
└── SPA React (Vite)
    ├── React Router: /auth, /, /perfil
    ├── AuthContext ──▶ Supabase Auth (login/cadastro)
    ├── Index.tsx
    │   ├── mockPatients (useState)        ← volátil
    │   ├── savedEvolutions (useState)     ← volátil
    │   └── evolutionTemplates (const)     ← estático
    └── Profile.tsx ──▶ tabela profiles + bucket avatars
```

- **Não há API de domínio.** O frontend fala direto com o Supabase.
- **Não há camada de serviço, repositório, DTO ou validação de schema.**
- **Não há separação por feature.** Tudo converge em `Index.tsx`.
- **Não há testes de fluxo.** O teste existente é um exemplo.

---

## 5. Telas e fluxos

### 5.1 `/auth`

- Login por e-mail/senha.
- **Cadastro aberto**: qualquer pessoa cria conta e escolhe CRM e especialidade livremente.
- Sem verificação de conselho, sem convite, sem aprovação, sem vínculo institucional.
- Sem recuperação de senha na interface (o Supabase oferece, mas não há tela).
- Sem MFA, sem política de senha além de `minLength=6`.

### 5.2 `/` (tela principal)

Fluxo único:

```
Lista de pacientes (mock, busca por nome/quarto/diagnóstico)
        │
        ▼
Seleciona paciente
        │
        ▼
Seleciona tipo de evolução (8 templates)
        │
        ▼
Preenche formulário estruturado (selects, números, multiselect, texto)
        │
        ├── Salvar → adiciona ao array em memória
        └── Salvar e imprimir → salva + abre prévia
                                    │
                                    ▼
                            window.open + document.write + print()
```

- Lista lateral fixa de 320 px; área de trabalho à direita.
- Histórico de evoluções filtrado por paciente.
- Cadastro de paciente em modal com 7 campos.
- **Não há edição de paciente, exclusão, duplicidade, convênio, responsável, anexos ou histórico cadastral.**

### 5.3 `/perfil`

- Nome, CRM, especialidade e avatar.
- Upload de avatar para bucket **público**.

### 5.4 `*`

- Página 404.

---

## 6. Funcionalidades: prometido × implementado

| Módulo do escopo novo | Existe na referência? | Evidência |
|---|---|---|
| Dashboard com indicadores | ❌ | Não há dashboard; a rota `/` é a tela de evolução |
| Cadastro completo de pacientes | ⚠️ parcial | 7 campos em modal, em memória |
| Prontuário eletrônico | ⚠️ parcial | Apenas evolução por template |
| Anamnese | ❌ | Não há |
| Histórico clínico consolidado | ❌ | Só lista de evoluções do paciente |
| Sinais vitais | ⚠️ parcial | Dentro do template de evolução |
| Exames | ❌ | Não há |
| Prescrições | ❌ | Não há |
| Diagnósticos (CID) | ❌ | Diagnóstico é texto livre no cadastro |
| Agenda | ❌ | Não há |
| Atendimentos / fila / status | ❌ | Não há |
| Profissionais | ⚠️ parcial | Só o próprio usuário logado |
| Especialidades configuráveis | ❌ | Texto livre no perfil |
| Permissões / perfis | ❌ | Não há |
| Auditoria | ❌ | Não há |
| Multi-instituição | ❌ | Não há |
| Documentos / atestados / receitas | ❌ | Só evolução impressa |
| Assinatura | ❌ | Só linha “CRM: ____” no papel |
| Financeiro | ❌ | Não há |
| Relatórios | ❌ | Não há |
| Backup | ❌ | Não há |
| Aplicativo desktop | ❌ | Web puro |
| Offline | ❌ | Não há |

**Leitura:** a referência cobre, no máximo, **1 dos 12 módulos** e de forma demonstrativa.

---

## 7. Design observado

### 7.1 Tokens

| Token | Valor (HSL) | Aparência |
|---|---|---|
| `--background` | `210 20% 98%` | cinza-azulado muito claro |
| `--card` | `0 0% 100%` | branco |
| `--foreground` | `210 40% 10%` | quase preto |
| `--primary` | `210 80% 45%` | azul clínico |
| `--accent` | `170 60% 42%` | verde-água |
| `--muted` | `210 15% 95%` | cinza claro |
| `--border` | `210 20% 88%` | cinza médio |
| `--radius` | `0.625rem` (10 px) | cantos suaves |
| `--medical-success/warning/info` | verde / âmbar / azul | estados clínicos |

### 7.2 Padrões de interface

- Header fixo de 56 px com marca, nome do profissional, perfil e sair.
- Container máximo de 1600 px.
- Cards brancos com borda de 1 px e raio de 12 px.
- Sombras discretas; sem gradientes.
- Tipografia sans-serif do sistema; títulos em `font-bold`.
- Lista lateral com busca e cards de paciente.
- Formulários em grid de 2 colunas em desktop, 1 em mobile.
- Multiselect por `Badge` clicável.
- Modal centralizado com overlay.
- Prévia de impressão em modal de 3xl com A4 simulado.
- Responsividade por breakpoint `lg` (empilha lista e conteúdo).

### 7.3 O que a referência faz bem

1. **Hierarquia clara:** lista de pacientes à esquerda, trabalho à direita.
2. **Formulário guiado por template:** reduz digitação e padroniza a evolução.
3. **Valores padrão clínicos:** campos já preenchidos para revisão.
4. **Separação entre preencher, salvar e imprimir.**
5. **Prévia de impressão fiel ao papel**, com cabeçalho, identificação e assinatura.
6. **Base shadcn/ui consistente**, com componentes acessíveis (Radix).

### 7.4 O que deve ser descartado

- Azul/verde como identidade (o novo produto terá magenta próprio).
- Lista lateral como navegação principal (o novo terá menu lateral de módulos).
- Cadastro aberto e ausência de vínculo institucional.
- Dados em memória.
- Impressão por `document.write` com interpolação direta de dados.
- Templates de evolução focados em **internação hospitalar** (leito/quarto), que não servem para consultório ambulatorial e multiespecialidade.

---

## 8. Qualidade de código

| Aspecto | Avaliação | Evidência |
|---|---|---|
| Organização | ⚠️ regular | Páginas e componentes separados; domínio concentrado em `Index.tsx` |
| Reutilização | ✅ boa na UI | shadcn/ui bem aplicado |
| Tipagem | ⚠️ regular | Interfaces locais; sem contratos compartilhados; `any` em erros |
| Validação | ❌ fraca | Sem zod; validação só por `required` do HTML |
| Estado | ❌ frágil | `useState` para dados que deveriam ser persistidos |
| Erros | ⚠️ parcial | Toasts genéricos; sem tratamento por código |
| Testes | ❌ inexistente | 1 teste de exemplo |
| Acessibilidade | ⚠️ parcial | Radix ajuda; faltam labels associadas e foco em modais |
| Performance | ⚠️ não avaliada | Sem paginação; listas pequenas |
| Documentação | ❌ ausente | README é o template do Lovable |

### 8.1 Bugs e riscos identificados no código

1. **Perda de dados ao recarregar:** pacientes e evoluções são estado local.
2. **Dupla gravação:** “Salvar e Imprimir” chama `onPreview`, que chama `handleSave`, e o formulário também expõe `onSave`; dependendo do caminho, a evolução pode ser salva duas vezes.
3. **Edição da prévia não persiste:** `editedText` é local ao modal e não atualiza a evolução salva.
4. **`currentText` calculado e não usado** em `PrintableEvolution.tsx`.
5. **XSS na janela de impressão:** `patient.name`, `evolution.doctor` e o texto clínico são interpolados em HTML sem escape.
6. **Bucket de avatares público:** qualquer pessoa com a URL acessa a imagem.
7. **Cadastro aberto:** não há controle de quem entra na base.
8. **CPF sem validação, normalização ou detecção de duplicidade.**
9. **Sem `updated_at` real de domínio** (só o perfil tem trigger).
10. **Sem paginação, índice ou busca no servidor** (porque não há servidor).

---

## 9. Segurança e privacidade na referência

| Controle | Situação |
|---|---|
| Autenticação | ✅ Supabase Auth (e-mail/senha) |
| Autorização | ❌ apenas “estar logado”; sem perfis/permissões |
| RLS | ⚠️ só em `profiles`; dados clínicos não existem no banco |
| Auditoria | ❌ inexistente |
| Criptografia em trânsito | ✅ HTTPS do Supabase |
| Criptografia em repouso | ⚠️ dependência do provedor; não declarada |
| Gestão de sessão | ⚠️ padrão do Supabase; sem revogação explícita na UI |
| MFA | ❌ |
| Recuperação de senha | ⚠️ existe no provedor, sem tela |
| Backup | ❌ não documentado |
| Retenção | ❌ não definida |
| Exportação controlada | ❌ |
| Log de acesso a prontuário | ❌ |
| LGPD | ❌ não tratada |

**Não é possível declarar conformidade** com LGPD, CFM ou conselhos profissionais a partir da referência. O novo projeto tratará esses requisitos como **requisitos de engenharia explícitos**, com validação jurídica posterior.

---

## 10. Limitações estruturais da referência

1. **Não é desktop.** É uma página web; não há instalador, ícone, auto-update ou integração com o sistema operacional.
2. **Não é multiusuário de verdade.** Não há vínculo entre usuários, instituição, unidade, paciente e profissional.
3. **Não é multiespecialidade.** Os templates são de internação (geral, cardio, pneumo, admissão, pós-cirúrgico, transferência, óbito).
4. **Não é auditável.** Nenhum registro guarda autoria confiável, versão ou hash.
5. **Não é escalável.** Sem paginação, índices, API ou banco de domínio.
6. **Não é seguro para PHI.** Dados clínicos nunca chegam a um banco com RLS e trilha.
7. **Não é um produto comercializável.** Falta o núcleo administrativo (instituições, unidades, permissões, financeiro, relatórios).

---

## 11. O que será aproveitado — e como

| Elemento | Como será tratado no PRONTUÁRIO PRO |
|---|---|
| Lista de pacientes + área de trabalho | Reimplementada com tabela paginada, filtros e painel de prontuário |
| Formulário guiado por template | Reimplementado como **modelos clínicos versionados por especialidade**, com schema JSON validado |
| Valores padrão clínicos | Mantidos como *defaults* explícitos e editáveis, nunca como fato registrado |
| Prévia de impressão | Reimplementada com **PDF gerado no servidor**, template versionado e hash |
| Base shadcn/ui | Reaproveitada como *biblioteca de componentes* (open source), não como código do projeto |
| Paleta clara e cards | Mantida como princípio; identidade nova em magenta |
| Busca de paciente | Reimplementada com `pg_trgm`, paginação e filtros no servidor |

**Não será aproveitado:** código-fonte, identidade visual, textos, dados mock, templates de internação, Supabase, chaves do `.env`, estrutura de rotas.

---

## 12. Conclusão da auditoria

A referência é útil como **estudo de caso de UX** e como **prova de conceito de evolução estruturada**. Não é base de código, não é arquitetura e não é produto.

O PRONTUÁRIO PRO será construído do zero, com:

- arquitetura cliente-servidor real;
- banco PostgreSQL com isolamento por instituição;
- autenticação, autorização granular e auditoria;
- prontuário imutável com adendos;
- documentos em PDF com hash;
- aplicativo desktop instalável.

O próximo documento (`02-ARQUITETURA.md`) define como isso será feito.
