# Índice da documentação

**Produto:** PRONTUÁRIO PRO — Gestão Clínica Inteligente
**Versão da documentação:** 1.0
**Data:** 08/11/2026
**Estado:** Fases 1 e 2 concluídas — aguardando aprovação para iniciar a Fase 3

---

## Trilha de leitura recomendada

1. **Decisão e contexto** → `01-AUDITORIA-REFERENCIA.md`
   O que o projeto de referência realmente é, o que serve de inspiração e o que não deve ser reutilizado.

2. **Como o sistema será construído** → `02-ARQUITETURA.md`
   Stack, camadas, multi-instituição, autenticação, permissões, auditoria, arquivos, PDF, offline, deploy.

3. **Onde os dados vivem** → `03-MODELO-DADOS.md`
   Tabelas, relacionamentos, enums, índices, constraints, RLS, imutabilidade e retenção.

4. **Em que ordem será entregue** → `04-PLANO-IMPLEMENTACAO.md`
   Fases, critérios de aceite, definição de pronto e dependências.

5. **O que pode dar errado** → `05-RISCOS-E-SEGURANCA.md`
   Riscos técnicos, de segurança, LGPD e jurídicos, com mitigação.

6. **Por que cada escolha foi feita** → `06-ADRS.md`
   Registro de decisões arquiteturais.

7. **Qual será a cara do produto** → `07-DIRECAO-VISUAL.md`
   Identidade, paleta, tipografia, componentes e estados.

---

## Portões de aprovação

| Portão | Critério | Estado |
|---|---|---|
| G1 | Auditoria da referência aceita | ✅ entregue |
| G2 | Arquitetura, modelo de dados e plano aceitos | ⏳ aguardando |
| G3 | Fundação (banco, auth, RBAC, auditoria) validada por testes | pendente |
| G4 | Módulos clínicos com prontuário imutável | pendente |
| G5 | Módulos administrativos e financeiros | pendente |
| G6 | Instalador Windows + testes de segurança e isolamento | pendente |
| G7 | Documentação operacional e aceite final | pendente |

---

## Glossário mínimo

- **Instituição (tenant):** clínica/rede dona dos dados. Todo dado operacional é isolado por `institution_id`.
- **Unidade:** filial/local físico de uma instituição.
- **Profissional:** vínculo entre um usuário e uma especialidade/registro de conselho.
- **Atendimento (encounter):** episódio assistencial de um paciente com um profissional.
- **Prontuário (medical record):** contêiner clínico do paciente/atendimento.
- **Evolução:** registro clínico datado, com autoria, que compõe o prontuário.
- **Adendo:** correção de registro finalizado, sem apagar o original.
- **RLS:** Row-Level Security do PostgreSQL — isolamento aplicado no banco.
- **PHI:** Protected Health Information / dado pessoal sensível de saúde.
