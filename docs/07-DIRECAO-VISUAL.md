# Direção Visual — PRONTUÁRIO PRO

**Versão:** 0.1 (proposta)
**Data:** 08/11/2026
**Estado:** pré-design system — será detalhado e implementado na Fase 4

---

## 1. Conceito

**“Clínico, calmo e preciso.”**

O produto deve parecer um instrumento de trabalho — não um site de marketing. A interface precisa transmitir:

- **confiança** (dados sensíveis, ambiente regulado);
- **clareza** (muita informação sem ruído);
- **velocidade** (recepção e profissional trabalham sob pressão);
- **hierarquia** (o que é urgente, pendente, finalizado).

A inspiração de organização vem da referência auditada (lista + área de trabalho, formulários guiados, prévia de documento), mas a identidade é própria: **magenta**, não azul/verde.

---

## 2. Paleta

### 2.1 Marca

| Token | Hex | Uso |
|---|---|---|
| `brand-50` | `#FDF2F8` | fundos sutis, seleção |
| `brand-100` | `#FCE7F3` | hover de item ativo |
| `brand-500` | `#D61F86` | **cor de destaque principal** |
| `brand-600` | `#B0186E` | botão primário, links |
| `brand-700` | `#8E1258` | pressed |
| `brand-900` | `#4A0A2E` | títulos sobre fundo claro (opcional) |

### 2.2 Neutros

| Token | Hex | Uso |
|---|---|---|
| `ink-900` | `#0F172A` | texto principal |
| `ink-700` | `#334155` | texto secundário forte |
| `ink-500` | `#64748B` | texto auxiliar |
| `ink-300` | `#CBD5E1` | placeholders, ícones inativos |
| `surface-0` | `#FFFFFF` | cards, modais |
| `surface-50` | `#F8FAFC` | fundo da aplicação |
| `surface-100` | `#F1F5F9` | cabeçalho de tabela, hover |
| `border` | `#E2E8F0` | bordas |
| `border-strong` | `#CBD5E1` | divisores |

### 2.3 Estados

| Token | Hex | Uso |
|---|---|---|
| `success-600` | `#15803D` | finalizado, pago, ativo |
| `success-50` | `#F0FDF4` | fundo de sucesso |
| `warning-600` | `#B45309` | pendente, atenção |
| `warning-50` | `#FFFBEB` | fundo de atenção |
| `danger-600` | `#B91C1C` | erro, cancelado, alergia grave |
| `danger-50` | `#FEF2F2` | fundo de erro |
| `info-600` | `#1D4ED8` | informação, agendado |
| `info-50` | `#EFF6FF` | fundo informativo |

### 2.4 Uso semântico

- **Magenta** = ação e identidade. Não usar em alertas.
- **Vermelho** = risco clínico/erro. Nunca usar como decoração.
- **Âmbar** = pendência. Nunca usar para “sucesso”.
- **Verde** = concluído. Nunca usar para “dinheiro” sem contexto.

Contraste mínimo AA (4.5:1) para texto; 3:1 para elementos de interface.

---

## 3. Tipografia

| Uso | Fonte | Peso | Tamanho |
|---|---|---|---|
| Títulos de página | Inter | 600 | 24 px / 1.25 |
| Títulos de seção | Inter | 600 | 18 px / 1.3 |
| Corpo | Inter | 400 | 14 px / 1.5 |
| Corpo denso (tabelas) | Inter | 400 | 13 px / 1.4 |
| Rótulos de formulário | Inter | 500 | 13 px / 1.4 |
| Auxiliar | Inter | 400 | 12 px / 1.4 |
| Números | Inter com `font-variant-numeric: tabular-nums` | — | — |

- Escala base 4 px.
- Nunca usar texto abaixo de 12 px.
- Tabelas e valores monetários sempre tabulares.

---

## 4. Espaçamento, raio e sombra

| Token | Valor |
|---|---|
| `space-1` | 4 px |
| `space-2` | 8 px |
| `space-3` | 12 px |
| `space-4` | 16 px |
| `space-6` | 24 px |
| `space-8` | 32 px |
| `radius-sm` | 6 px |
| `radius-md` | 10 px |
| `radius-lg` | 12 px |
| `radius-full` | 9999 px |
| `shadow-xs` | `0 1px 2px rgba(15,23,42,.06)` |
| `shadow-sm` | `0 1px 3px rgba(15,23,42,.10)` |
| `shadow-md` | `0 4px 12px rgba(15,23,42,.08)` |

Sombras discretas. Nada de `box-shadow` pesada ou gradientes decorativos.

---

## 5. Layout

```
┌──────────────────────────────────────────────────────────────┐
│ HEADER 64px: marca · instituição/unidade · busca · notificações · usuário │
├──────────────┬───────────────────────────────────────────────┤
│ SIDEBAR      │ CONTEÚDO                                      │
│ 260px        │ max-width 1440px, padding 24px                │
│              │                                               │
│ Dashboard    │ ┌─ título + ações ──────────────────────────┐ │
│ Pacientes    │ │                                          │ │
│ Agenda       │ │  cards / tabela / formulário             │ │
│ Atendimentos │ │                                          │ │
│ Prontuários  │ │                                          │ │
│ Profissionais│ │                                          │ │
│ Financeiro   │ │                                          │ │
│ Relatórios   │ │                                          │ │
│ Administração│ │                                          │ │
│ Configurações│ └──────────────────────────────────────────┘ │
└──────────────┴───────────────────────────────────────────────┘
```

- Sidebar colapsável (ícone + tooltip).
- Item ativo com fundo `brand-50`, texto `brand-700` e barra de 3 px à esquerda.
- Header fixo; conteúdo com scroll próprio.
- Em telas < 1180 px, sidebar vira drawer.
- Densidade “confortável”: linhas de tabela 44–48 px.

---

## 6. Componentes

### 6.1 Botões

| Variante | Aparência |
|---|---|
| Primário | fundo `brand-600`, texto branco, hover `brand-700` |
| Secundário | fundo branco, borda, texto `ink-900` |
| Fantasma | sem fundo, hover `surface-100` |
| Destrutivo | fundo `danger-600`, texto branco |
| Link | texto `brand-600`, sublinhado no hover |

Altura 36 px (padrão) e 32 px (compacto). Ícone + texto com gap de 8 px. Sempre com estado de foco visível.

### 6.2 Cards de indicador

```
┌────────────────────────────┐
│ Pacientes ativos      [ícone] │
│ 1.284                        │
│ +12 este mês   ▲             │
└────────────────────────────┘
```

- Rótulo 12 px `ink-500`; valor 28 px 600 `ink-900`; delta com cor semântica.
- Nunca usar gráfico decorativo sem dado real.

### 6.3 Tabelas

- Cabeçalho `surface-100`, 12 px 600, caixa alta discreta.
- Linha com hover `surface-50`; seleção `brand-50`.
- Coluna de ações à direita, com menu “⋯”.
- Paginação por cursor, com “Carregar mais” ou paginação numerada.
- Estado vazio com ícone, explicação e ação primária.
- Nunca esconder coluna crítica em telas pequenas; usar scroll horizontal controlado.

### 6.4 Formulários

- Rótulo acima, 13 px 500.
- Campo 36–40 px, borda `border`, foco `brand-500` com anel de 2 px.
- Texto de ajuda 12 px `ink-500`.
- Erro abaixo do campo, `danger-600`, com ícone.
- Obrigatório marcado com `*` e `aria-required`.
- Agrupamento por seções com títulos claros; formulários longos em etapas quando fizer sentido.
- Salvamento explícito; nunca salvar automaticamente dado clínico sem indicar.

### 6.5 Modais e drawers

- Modal para decisões curtas; drawer para edição contextual (ex.: paciente).
- Largura máxima 560 px (modal) e 480 px (drawer).
- Foco preso, ESC fecha, clique fora confirma se houver alteração não salva.
- Título, descrição e ações no rodapé.

### 6.6 Badges de status

| Status | Cor |
|---|---|
| Agendado | info |
| Confirmado | brand |
| Aguardando | warning |
| Em atendimento | brand |
| Finalizado | success |
| Cancelado | neutro/riscado |
| Falta | danger |
| Rascunho | neutro |
| Finalizado (documento) | success |
| Adendado | warning |

---

## 7. Ícones

- Lucide, traço 1.75–2 px, tamanho 16/18/20 px.
- Um ícone nunca substitui um rótulo em ação destrutiva.
- Ícones de especialidade: usar símbolos genéricos (estetoscópio, coração, cérebro, criança, dente) sem caricatura.

---

## 8. Movimento

- Transições de 120–180 ms, `ease-out`.
- Sem animações chamativas em área clínica.
- Respeitar `prefers-reduced-motion`.

---

## 9. Acessibilidade

- Contraste AA.
- Navegação completa por teclado.
- Foco visível em todos os elementos interativos.
- `aria-label` em ícones sem texto.
- Não usar cor como único indicador de status.
- Mensagens de erro associadas ao campo.
- Tabelas com cabeçalhos semânticos.
- Testes com leitor de tela nos fluxos de login, paciente e evolução.

---

## 10. Documentos e impressão

- A4, margens de 20 mm.
- Cabeçalho: instituição, unidade, CNPJ, endereço; título do documento.
- Identificação do paciente em bloco destacado.
- Corpo em 11 pt, entrelinha 1.5.
- Rodapé: página, data/hora, autor, registro profissional, hash curto.
- Assinatura: linha, nome, conselho/registro e tipo de assinatura (**eletrônica simples**).
- Marca d'água discreta em exportações (“cópia”) quando aplicável.
- Nunca usar a identidade da referência.

---

## 11. Marca

- Nome provisório: **PRONTUÁRIO PRO**.
- Símbolo: monograma “P” com uma linha de pulso integrada, em magenta.
- Wordmark em Inter 700, caixa alta com tracking leve.
- Variações: horizontal (sidebar/header), símbolo isolado (ícone do app), monocromático.
- O ícone do instalador será gerado em `.ico` multi-resolução.

---

## 12. Tokens em CSS (prévia)

```css
:root {
  --brand-50: #FDF2F8;
  --brand-100: #FCE7F3;
  --brand-500: #D61F86;
  --brand-600: #B0186E;
  --brand-700: #8E1258;

  --ink-900: #0F172A;
  --ink-700: #334155;
  --ink-500: #64748B;

  --surface-0: #FFFFFF;
  --surface-50: #F8FAFC;
  --surface-100: #F1F5F9;
  --border: #E2E8F0;

  --success-600: #15803D;
  --warning-600: #B45309;
  --danger-600: #B91C1C;
  --info-600: #1D4ED8;

  --radius-md: 10px;
  --shadow-sm: 0 1px 3px rgba(15, 23, 42, .10);
}
```

---

## 13. O que evitar

- Reproduzir a paleta azul/verde da referência.
- Excesso de cards e gráficos sem dado.
- Gradientes, glassmorphism e sombras pesadas.
- Ícones sem rótulo em ações críticas.
- Texto cinza claro sobre branco.
- Tabelas sem paginação.
- Modais dentro de modais.
- Notificações em excesso.
