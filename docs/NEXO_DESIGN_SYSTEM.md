# Identidade Visual — NEXO Design System no DroneCalc

**Versão:** 0.1  
**Fonte de referência:** `eduardobelopro-br/nexo/frontend/src/design-system/`

## 1. Regra central

DroneCalc deve utilizar a **mesma linguagem visual do NEXO**, adaptada ao contexto de engenharia de drones. Não criar um design system concorrente e não inventar paletas específicas por estilo de voo.

A implementação do DroneCalc deve manter uma cópia própria e rastreável dos tokens/componentes necessários para não criar dependência de runtime entre repositórios.

## 2. Fonte de verdade NEXO

Estrutura observada no NEXO:

```text
frontend/src/design-system/
├── components/
├── layouts/
├── themes/
├── tokens/
└── index.css
```

Tokens existentes:

- `tokens/colors.css`;
- `tokens/shadows.css`;
- `tokens/spacing.css`;
- `tokens/typography.css`.

Temas:

- `themes/dark.css`;
- `themes/light.css`.

## 3. Política de sincronização

1. NEXO é a referência visual.
2. DroneCalc mantém seus arquivos em `src/design-system/`.
3. Não importar diretamente código do repositório NEXO em runtime.
4. Mudanças futuras na identidade NEXO devem ser incorporadas conscientemente ao DroneCalc e registradas em commit/ADR quando afetarem comportamento visual.
5. Componentes específicos de drone podem ser adicionados, mas devem consumir tokens NEXO.

## 4. Cores — tema escuro de referência

Os tokens atuais do NEXO usam:

```css
--nexo-primary: #2563eb;
--nexo-primary-hover: #3b82f6;
--nexo-primary-soft: rgba(37, 99, 235, 0.1);
--nexo-primary-border: rgba(59, 130, 246, 0.3);

--nexo-secondary: #1e293b;
--nexo-secondary-hover: #334155;

--nexo-background: #020617;
--nexo-surface: #0f172a;
--nexo-surface-alt: #1e293b;
--nexo-surface-elevated: #0f172a;

--nexo-text-primary: #f1f5f9;
--nexo-text-secondary: #94a3b8;
--nexo-text-muted: #64748b;
--nexo-text-on-primary: #ffffff;

--nexo-border: #1e293b;
--nexo-border-soft: #334155;

--nexo-success: #4ade80;
--nexo-warning: #fbbf24;
--nexo-danger: #f87171;
--nexo-info: #60a5fa;
--nexo-neutral: #94a3b8;
```

Também existem variantes `*-bg` e `*-border` para estados semânticos.

## 5. Tema claro de referência

Principais tokens atuais:

```css
--nexo-primary: #2563eb;
--nexo-primary-hover: #1d4ed8;
--nexo-background: #f1f5f9;
--nexo-surface: #ffffff;
--nexo-surface-alt: #f8fafc;
--nexo-text-primary: #0f172a;
--nexo-text-secondary: #475569;
--nexo-text-muted: #64748b;
--nexo-border: #e2e8f0;
--nexo-border-soft: #cbd5e1;
--nexo-success: #16a34a;
--nexo-warning: #d97706;
--nexo-danger: #dc2626;
--nexo-info: #2563eb;
--nexo-neutral: #64748b;
```

A seleção explícita usa `[data-theme="light"]` e `[data-theme="dark"]`. O modo sistema pode seguir `prefers-color-scheme`.

## 6. Tipografia

Fonte sans atual:

```css
--nexo-font-sans: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
```

Fonte mono:

```css
--nexo-font-mono: ui-monospace, "SFMono-Regular", Menlo, Consolas, monospace;
```

Escala:

```css
--nexo-text-xs: 0.75rem;
--nexo-text-sm: 0.875rem;
--nexo-text-base: 1rem;
--nexo-text-lg: 1.125rem;
--nexo-text-xl: 1.25rem;
--nexo-text-2xl: 1.5rem;
```

Valores técnicos, fórmulas, identificadores e pequenos blocos numéricos podem usar a fonte mono quando isso aumentar legibilidade.

## 7. Espaçamento estrutural e raios

Tokens atuais:

```css
--nexo-radius-sm: 0.5rem;
--nexo-radius-md: 0.75rem;
--nexo-radius-lg: 1rem;
--nexo-radius-xl: 1.25rem;

--nexo-sidebar-width: 15rem;
--nexo-header-height: 4rem;
```

DroneCalc pode adicionar tokens de spacing ausentes, desde que use prefixo NEXO/semântico e preserve a linguagem existente.

## 8. Sombras

```css
--nexo-shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.2);
--nexo-shadow-md: 0 4px 12px rgba(0, 0, 0, 0.25);
--nexo-shadow-glow-primary: 0 0 15px rgba(59, 130, 246, 0.2);
--nexo-shadow-glow-danger: 0 0 15px rgba(239, 68, 68, 0.2);
```

Glows devem ser usados com moderação; dashboards técnicos não devem ficar visualmente ruidosos.

## 9. Semântica de estados no DroneCalc

Mapeamento obrigatório:

- `success` — regra verificada e margem adequada;
- `info` — informação técnica, hipótese ou verificação não concluída;
- `warning` — margem baixa, dado fraco ou condição a revisar;
- `danger` — incompatibilidade ou limite excedido;
- `neutral` — informação sem julgamento técnico.

A cor nunca é o único sinal. Sempre combinar cor com ícone, texto, rótulo ou estrutura acessível.

## 10. Uso de cor por estilo de voo

Não atribuir uma paleta permanente diferente para Racing, Long Range, Mini Long Range etc.

O perfil selecionado recebe destaque via `--nexo-primary`. Cores success/warning/danger são reservadas para significado semântico.

Isso impede que, por exemplo, uma cor vermelha de branding de Racing seja confundida com erro crítico.

## 11. Shell do aplicativo

Estrutura desejada em desktop:

```text
┌───────────────────────────────────────────────────────────────┐
│ NEXO / DroneCalc                      Projeto ativo   Tema    │
├────────────────┬──────────────────────────────────────────────┤
│ Navegação      │ Workspace                                   │
│                │                                             │
│ Projetos       │ Builder / Análise / Comparador              │
│ Componentes    │                                             │
│ Propulsão      │                                             │
│ Energia        │                                             │
│ Compatibilidade│                                             │
│ Comparador     │                                             │
└────────────────┴──────────────────────────────────────────────┘
```

O shell segue a lógica NEXO de header/sidebar/superfícies, mas o workspace do DroneCalc é especializado.

## 12. Componentes visuais necessários

Base compartilhável:

- Button;
- IconButton;
- Input;
- NumberInput;
- Select;
- Checkbox;
- Radio/SegmentedControl;
- Badge;
- Alert;
- Card;
- Tooltip;
- Modal/Dialog;
- Tabs;
- Table;
- EmptyState;
- Skeleton;
- Progress/MetricBar.

Especializados DroneCalc:

- ComponentCard;
- MetricCard;
- CompatibilityAlert;
- ConfidenceBadge;
- ProvenanceBadge;
- FlightProfileCard;
- BenchCurveChart;
- ComponentPicker;
- MassBreakdown;
- ProjectHealthSummary;
- ComparisonMetricRow.

Componentes especializados devem ser compostos a partir dos primitives NEXO.

## 13. Cards técnicos

Um card de métrica deve apresentar:

1. rótulo;
2. valor + unidade;
3. status opcional;
4. origem/confiança opcional;
5. acesso a “Como foi calculado?”.

Exemplo conceitual:

```text
Autonomia estimada
21,4 min
Estimado · confiança média
[Como foi calculado?]
```

## 14. Densidade de informação

DroneCalc é ferramenta técnica. A UI pode ser mais densa que uma landing page, porém deve preservar:

- alinhamento consistente;
- unidades próximas dos valores;
- hierarquia tipográfica;
- agrupamento por contexto;
- espaços suficientes para leitura;
- ausência de decoração sem função.

## 15. Formulários

- labels persistentes;
- unidade no controle ou adjacente;
- validação inline;
- permitir entrada decimal local e normalizar internamente;
- não depender apenas de placeholder;
- estado de foco com `--nexo-primary` / `--nexo-primary-soft`;
- números não devem mudar acidentalmente com scroll quando o campo não está intencionalmente ativo.

## 16. Gráficos

Curvas de bancada devem:

- ter eixos e unidades;
- indicar pontos medidos;
- distinguir trecho interpolado;
- nunca desenhar extrapolação como se fosse dado real;
- permitir destacar o ponto de hover;
- funcionar em claro/escuro usando tokens;
- não depender apenas de cor para diferenciar séries.

## 17. Acessibilidade

- contraste verificável nos dois temas;
- foco visível;
- tamanho de alvo adequado;
- tooltips não podem conter informação exclusiva essencial;
- status com texto/ícone além de cor;
- tabelas técnicas com cabeçalhos corretos;
- suporte a redução de movimento.

## 18. Responsividade

Desktop é a experiência primária.

- sidebar pode recolher em larguras menores;
- grids de métricas reduzem colunas progressivamente;
- comparador pode usar scroll horizontal controlado;
- builder não deve esconder unidades ou status críticos para caber em mobile.

## 19. Proibições

- cores hex/rgb hardcoded dentro de features, salvo visualizações onde tokens dinâmicos sejam tecnicamente impossíveis e isso esteja documentado;
- novo sistema de tema paralelo;
- status comunicado somente por cor;
- gradientes/decorativos excessivos;
- diferentes componentes de input com comportamento divergente sem motivo;
- “score verde” quando existe alerta `danger` não resolvido.

## 20. Implementação inicial

Criar em `src/design-system/`:

```text
design-system/
├── tokens/
│   ├── colors.css
│   ├── spacing.css
│   ├── typography.css
│   └── shadows.css
├── themes/
│   ├── dark.css
│   └── light.css
├── components/
├── layouts/
└── index.css
```

Copiar inicialmente os tokens compatíveis do NEXO e registrar no cabeçalho dos arquivos a origem e data/revisão de sincronização.
