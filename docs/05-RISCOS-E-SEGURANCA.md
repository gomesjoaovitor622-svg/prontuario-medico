# Riscos, Segurança e LGPD — PRONTUÁRIO PRO

**Versão:** 1.0
**Data:** 08/11/2026
**Estado:** proposta para aprovação (Portão G2)

> Este documento é técnico. **Não constitui parecer jurídico** e **não declara conformidade** com LGPD, CFM, conselhos profissionais ou ICP-Brasil. A conformidade depende de validação jurídica, contratos, políticas institucionais e auditoria independente.

---

## 1. Riscos de ambiente (bloqueiam a Fase 3)

| ID | Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|---|
| A1 | Rust/cargo ausente → Tauri não compila | alta | alto | `winget install Rustlang.Rustup` + MSVC Build Tools; validar `cargo --version` antes da Fase 4 |
| A2 | PostgreSQL ausente → sem testes reais | alta | alto | instalar PostgreSQL 16 local **ou** Docker Desktop; CI usa serviço PostgreSQL |
| A3 | Docker ausente | alta | médio | instalar Docker Desktop ou usar PostgreSQL local; Compose é para reprodutibilidade |
| A4 | `npm.ps1` bloqueado pela ExecutionPolicy | confirmado | médio | usar `npm.cmd`; opcionalmente `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` (requer consentimento) |
| A5 | Projeto dentro do OneDrive | confirmado | alto | mover para `C:\dev\prontuario-pro`; OneDrive sincroniza `node_modules`, trava arquivos e quebra builds Rust |
| A6 | Caminho com acento/espaço (`João`, `Projeto Padrão`) | confirmado | alto | mesmo que A5: usar caminho ASCII sem espaços |
| A7 | `gh` CLI ausente | confirmado | baixo | push via Git + Personal Access Token, ou instalar `gh` |
| A8 | WebView2 ausente em Windows 10 antigo | média | médio | instalador do Tauri inclui bootstrapper ou documenta pré-requisito |
| A9 | Sem certificado de servidor/domínio | média | alto | exigir domínio + TLS antes de produção; autoassinado só em homologação |
| A10 | Máquina de desenvolvimento sem recurso | baixa | médio | mínimo 8 GB RAM, 4 vCPU, 20 GB livres |

---

## 2. Riscos técnicos

| ID | Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|---|
| T1 | Vazamento entre instituições (multi-tenant) | média | **crítico** | RLS + guard + testes dedicados + revisão de toda query |
| T2 | IDOR (acesso por ID de outro paciente) | média | **crítico** | autorização por recurso no serviço; nunca confiar no ID do cliente; 404 em vez de 403 |
| T3 | Escalada de privilégio | média | **crítico** | `deny` vence; admin não concede o que não tem; superadmin protegido; testes |
| T4 | Registro clínico sobrescrito | média | **crítico** | trigger de imutabilidade + hash + revisão + testes de banco |
| T5 | Perda de dados por backup não verificado | média | **crítico** | restauração automática em base limpa + checksum + auditoria |
| T6 | Corrupção por escrita concorrente | média | alto | transações, constraints, versionamento otimista |
| T7 | PDF divergente entre postos | alta | médio | geração no servidor, template versionado |
| T8 | Upload malicioso | média | alto | validação de MIME real, limite de tamanho, antivírus, URL assinada |
| T9 | Performance em listas grandes | média | médio | paginação por cursor, índices compostos, trigram |
| T10 | Acoplamento Prisma ↔ RLS | média | médio | SQL versionado para políticas; teste que percorre catálogo |
| T11 | Complexidade de 12 módulos | alta | alto | fases com portões; nada de “big bang” |
| T12 | Dependência de fornecedor de nuvem | baixa | médio | storage S3-compatível; Postgres padrão; sem serviços proprietários obrigatórios |
| T13 | Atualização quebrada no desktop | média | alto | updater assinado, canal beta, rollback documentado |
| T14 | Fuso horário errado em agenda | média | alto | UTC no banco, timezone da instituição na borda, testes de DST |
| T15 | Duplicidade de paciente | alta | médio | CPF/CNS normalizados, índice único, busca por similaridade, fusão auditada |

---

## 3. Riscos de segurança (modelo de ameaça)

### 3.1 Ativos

- Credenciais e sessões.
- PHI (prontuário, diagnósticos, prescrições, anexos).
- Documentos assinados e PDFs.
- Trilhas de auditoria.
- Backups.
- Chaves e segredos de infraestrutura.

### 3.2 Ameaças e controles

| Ameaça | Vetor | Controle |
|---|---|---|
| Roubo de sessão | malware no posto, token em disco | refresh no keychain, access em memória, rotação, revogação, expiração curta |
| Credential stuffing | login | rate limit, bloqueio progressivo, MFA para admin, senha forte |
| Engenharia social | suporte/recepção | treinamento, princípio do menor privilégio, confirmação de identidade |
| Insider malicioso | funcionário | auditoria de leitura, escopo por paciente, exportação controlada, segregação de funções |
| SQL injection | filtros e buscas | Prisma parametrizado, validação zod, proibição de SQL concatenado |
| XSS | campos livres, documentos | escape por padrão, sanitização de HTML de documento, CSP |
| CSRF | API com Bearer | não usa cookie de sessão; ainda assim, origem allowlist |
| SSRF | importação por URL (futuro) | não permitir URLs arbitrárias |
| Path traversal | upload/download | storage key gerada por UUID, sem nome do usuário no caminho |
| Ransomware | servidor | backup offsite imutável, menor privilégio, EDR, restauração testada |
| DDoS | API pública | proxy com rate limit, timeouts, limites de payload |
| Vazamento em log | logs técnicos | redaction, proibição de PHI, revisão de logger |
| Vazamento em backup | mídia | criptografia, controle de acesso, rotação de chaves |

### 3.3 Requisitos de segurança obrigatórios

- TLS 1.2+ em todas as comunicações; HSTS.
- Senhas com argon2id; nunca reversíveis.
- Segredos fora do código; `.env` ignorado.
- Sessões revogáveis e auditadas.
- Autorização no servidor em **toda** operação.
- RLS como segunda barreira.
- Princípio do menor privilégio por padrão.
- Logs sem PHI.
- Backup criptografado e verificado.
- Dependências auditadas e atualizadas.
- Revisão de código com foco em autorização.
- Teste de segurança antes de produção.

---

## 4. LGPD — requisitos de engenharia

### 4.1 Bases legais e finalidade

O sistema trata dados pessoais e **dados pessoais sensíveis de saúde**. As bases legais aplicáveis (tutela da saúde, obrigação legal, consentimento quando cabível) devem ser definidas pela instituição controladora com apoio jurídico. O software deve permitir:

- registrar finalidade e base legal por operação;
- limitar acesso ao mínimo necessário;
- registrar consentimento quando a base for consentimento;
- atender direitos do titular.

### 4.2 Direitos do titular

| Direito | Implementação |
|---|---|
| Confirmação e acesso | exportação do prontuário em PDF/JSON |
| Correção | adendo/revisão, nunca sobrescrita |
| Anonimização/bloqueio | fluxo `data_subject_requests`, com validação jurídica |
| Portabilidade | exportação estruturada autorizada |
| Eliminação | somente quando não houver obrigação legal de retenção |
| Informação sobre compartilhamento | relatório de acessos e exportações |

### 4.3 Controles técnicos

- Minimização: telas mostram apenas o necessário; listas não exibem diagnóstico sem permissão.
- Segregação: recepção não vê prontuário; financeiro não vê clínico.
- Auditoria de acesso a PHI (`record_access_logs`).
- Retenção configurável e documentada.
- Exportação com marca d'água, registro de quem exportou e para onde.
- Criptografia em trânsito; em repouso no banco, disco e backup.
- Pseudonimização em ambientes de teste.
- Política de incidentes: detecção, contenção, registro, comunicação à ANPD e titulares quando aplicável.

### 4.4 O que o software **não** resolve sozinho

- Contrato de operador/controlador entre clínica e fornecedor.
- Política de privacidade e avisos de cookies (não há site).
- Designação de DPO/encarregado.
- Base legal e consentimento.
- Treinamento de pessoal.
- Auditoria independente.
- Avaliação de impacto (RIPD).

---

## 5. Riscos jurídicos e regulatórios

| ID | Tema | Risco | Mitigação |
|---|---|---|---|
| J1 | Prontuário eletrônico | requisitos do CFM e de conselhos profissionais não atendidos | mapear requisitos com jurídico; validar antes de produção |
| J2 | Assinatura | chamar assinatura simples de “digital” gera risco jurídico | rótulos explícitos; ICP-Brasil fora da v1 |
| J3 | Guarda do prontuário | prazo de retenção incorreto | política definida com jurídico; referência CFM (20 anos) a confirmar |
| J4 | Telemedicina | se houver atendimento remoto, regras específicas | fora da v1; módulo separado com requisitos próprios |
| J5 | TISS/convênios | faturamento incompatível | não prometer TISS na v1; integração futura |
| J6 | Prescrição controlada | requisitos de receita especial | validar por conselho/UF; não emitir sem regra |
| J7 | LGPD | tratamento sem base legal | governança do cliente + controles técnicos |
| J8 | Propriedade intelectual | copiar código da referência (sem licença) | **não reutilizar código**; reimplementar |
| J9 | Certificação SBIS/CFM | alegar certificação sem auditoria | não alegar; buscar certificação se comercial |
| J10 | Termos e contratos | ausência de contrato de suporte/DPA | minuta jurídica antes da venda |

### 5.1 Referências a verificar (vigência e aplicabilidade)

- Lei nº 13.709/2018 (LGPD).
- Lei nº 14.063/2020 (assinaturas eletrônicas).
- Lei nº 12.965/2014 (Marco Civil).
- Resoluções do CFM sobre prontuário eletrônico e guarda de documentos (verificar a vigência da Resolução nº 2.299/2021 e correlatas).
- Normas dos conselhos profissionais aplicáveis (CRM, CRP, COREN, CREFITO, CRN, CRO etc.).
- Requisitos da ICP-Brasil para assinatura qualificada.
- Regulamentações da ANPD.

**Nenhuma dessas referências é declarada como atendida** até validação formal.

---

## 6. Riscos de produto e operação

| ID | Risco | Prob. | Impacto | Mitigação |
|---|---|---|---|---|
| P1 | Escopo crescer sem controle | alta | alto | fases e portões; mudança só com impacto declarado |
| P2 | Clínica não homologar | média | alto | ambiente de staging e roteiro de aceite por perfil |
| P3 | Usuários resistirem à migração | média | médio | treinamento, manual, importação assistida |
| P4 | Dados legados inconsistentes | média | alto | diagnóstico de dados antes da importação |
| P5 | Suporte sem equipe | média | alto | documentação operacional + contrato de suporte |
| P6 | Internet instável na clínica | média | alto | leitura em cache, banner claro, servidor local opcional |
| P7 | Falta de energia no servidor | baixa | alto | nobreak, backup, monitoramento |
| P8 | Credenciais compartilhadas entre funcionários | alta | alto | contas individuais obrigatórias, auditoria, política |
| P9 | Impressora/layout | média | baixo | PDF padronizado, teste em impressoras reais |
| P10 | Custos de nuvem | baixa | médio | dimensionamento e monitoramento |

---

## 7. Plano de resposta a incidentes (resumo)

1. **Detectar:** alerta de acesso anômalo, falha de backup, erro 5xx, reclamação.
2. **Conter:** revogar sessões, bloquear usuário, isolar serviço, preservar evidências.
3. **Avaliar:** quais dados, quais titulares, qual instituição, qual janela de tempo.
4. **Notificar:** encarregado/DPO, instituição controladora, ANPD e titulares quando exigido.
5. **Corrigir:** patch, rotação de credenciais, restauração de backup se necessário.
6. **Registrar:** post-mortem sem PHI, ações e prazos.
7. **Prevenir:** teste de regressão e ajuste de controle.

---

## 8. Checklist de segurança antes de produção

- [ ] TLS válido e HSTS.
- [ ] Segredos fora do repositório e rotacionáveis.
- [ ] Senhas em argon2id.
- [ ] MFA habilitado para perfis administrativos.
- [ ] Sessões revogáveis e auditadas.
- [ ] RBAC + permissões individuais testados.
- [ ] RLS ativa em todas as tabelas com `institution_id`.
- [ ] Teste de isolamento entre instituições aprovado.
- [ ] Teste de IDOR e escalada aprovado.
- [ ] Registro finalizado imutável (teste de banco).
- [ ] Auditoria append-only.
- [ ] Logs sem PHI.
- [ ] Upload validado e armazenamento privado.
- [ ] Backup automático + restauração verificada.
- [ ] Retenção definida.
- [ ] Exportação controlada e auditada.
- [ ] Dependências auditadas.
- [ ] Instalador assinado.
- [ ] Documentação operacional entregue.
- [ ] Validação jurídica e dos conselhos concluída.

---

## 9. Riscos aceitos conscientemente na v1

| Item | Decisão | Justificativa |
|---|---|---|
| Sem escrita offline | aceito | risco clínico e de conflito maior que o benefício |
| Assinatura apenas eletrônica simples | aceito | ICP-Brasil exige certificado e processo próprio; será módulo futuro |
| Sem TISS | aceito | complexidade regulatória; v1 cobre particular/convênio simples |
| Sem telemedicina | aceito | requisitos próprios; fora do escopo inicial |
| Single server (não cluster) | aceito | adequado ao porte inicial; arquitetura permite evoluir |
| Sem app mobile | aceito | escopo desktop primeiro |

---

## 10. Conclusão

Os riscos mais graves não são de interface: são de **isolamento entre instituições, autorização, imutabilidade do prontuário e backup**. Por isso a Fase 3 começa por banco, autenticação, RBAC, RLS e auditoria — e não por telas.

Nenhuma alegação de conformidade será feita sem evidência. Nenhum teste será declarado sem execução. Nenhum dado simulado será usado em produção.
