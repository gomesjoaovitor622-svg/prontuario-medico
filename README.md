# PRONTUÁRIO PRO — Gestão Clínica Inteligente

Sistema de prontuário eletrônico multiespecialidades, multiusuário e multi-instituição.
Aplicativo desktop (Windows) com dados centralizados em servidor.

> **Status:** Fase 1 (auditoria da referência) e Fase 2 (arquitetura) concluídas.
> Nenhum código de produção foi escrito ainda. O desenvolvimento começa após a aprovação da arquitetura.

---

## Documentação

| Documento | Conteúdo |
|---|---|
| [`docs/00-INDICE.md`](docs/00-INDICE.md) | Índice e trilha de leitura |
| [`docs/01-AUDITORIA-REFERENCIA.md`](docs/01-AUDITORIA-REFERENCIA.md) | Auditoria técnica do projeto de referência (Pronto Fácil Doctor) |
| [`docs/02-ARQUITETURA.md`](docs/02-ARQUITETURA.md) | Arquitetura proposta, stack, segurança, multi-tenancy, comunicação |
| [`docs/03-MODELO-DADOS.md`](docs/03-MODELO-DADOS.md) | Modelo relacional PostgreSQL, constraints, índices, RLS, imutabilidade |
| [`docs/04-PLANO-IMPLEMENTACAO.md`](docs/04-PLANO-IMPLEMENTACAO.md) | Fases, entregáveis, critérios de aceite e portões de qualidade |
| [`docs/05-RISCOS-E-SEGURANCA.md`](docs/05-RISCOS-E-SEGURANCA.md) | Riscos técnicos, de segurança, LGPD e jurídicos, com mitigações |
| [`docs/06-ADRS.md`](docs/06-ADRS.md) | Decisões arquiteturais registradas (ADRs) |
| [`docs/07-DIRECAO-VISUAL.md`](docs/07-DIRECAO-VISUAL.md) | Proposta de identidade visual e design system |

---

## Escopo do produto

1. Dashboard
2. Cadastro de pacientes
3. Prontuário eletrônico
4. Agenda e atendimentos
5. Gestão de profissionais
6. Administração, permissões e auditoria
7. Relatórios e documentos
8. Financeiro
9. Configurações, backup e segurança

Perfis: Superadministrador, Administrador da instituição, Gestor, Recepcionista,
Profissional de saúde, Supervisor, Financeiro e Auditor — com permissões granulares
por usuário, além do perfil.

---

## Arquitetura em uma frase

Aplicativo **Tauri 2 + React + TypeScript** instalado nos postos de trabalho, consumindo
uma **API NestJS** que concentra autenticação, autorização, auditoria e regras de negócio
sobre um **PostgreSQL 16** com isolamento por instituição (RLS) e armazenamento de arquivos
protegido.

```
Desktop (Tauri + React)  ──HTTPS/JSON──▶  API (NestJS)  ──▶  PostgreSQL 16
                                             │
                                             └──▶  Object storage (anexos/PDF)
```

---

## Princípios inegociáveis

- Dados oficiais centralizados; **não** existe banco clínico por computador.
- **Sem escrita clínica offline** até existir sincronização segura, resolução de conflitos e rastreabilidade.
- Toda operação sensível é validada **no servidor**; esconder botão não é controle de acesso.
- Registro clínico finalizado é **imutável**: correção por adendo/revisão, com autoria e hash.
- Nenhuma senha em texto puro; nenhuma credencial no repositório.
- Nenhum dado simulado em produção; nenhum botão sem ação.
- Conformidade legal **não é declarada** sem validação jurídica e dos conselhos profissionais.

---

## Licença

Projeto proprietário. Todos os direitos reservados.
O projeto de referência **não** é distribuído neste repositório e **não** teve código copiado.
