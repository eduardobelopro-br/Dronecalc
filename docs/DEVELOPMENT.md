# Guia de Desenvolvimento — DroneCalc

**Versão:** 0.1

## 1. Objetivo

Definir padrões mínimos para que implementações humanas ou assistidas por IA mantenham o DroneCalc consistente, testável e tecnicamente auditável.

## 2. Stack planejada

- React;
- TypeScript strict;
- Vite;
- Zod;
- Zustand;
- Vitest;
- Testing Library;
- IndexedDB via adapter/repository.

Essas escolhas são baseline, não dogma. Alterações estruturais exigem justificativa e atualização de arquitetura/ADR.

## 3. Comandos esperados

A Etapa 1A deve criar scripts equivalentes a:

```bash
npm run dev
npm run build
npm run test
npm run test:watch
npm run typecheck
npm run lint
```

CI deve usar os mesmos comandos ou seus equivalentes.

## 4. TypeScript

- `strict: true`;
- evitar `any`;
- preferir `unknown` + narrowing nas bordas;
- schemas externos validados antes de cast;
- unions discriminadas para estados;
- tipos físicos/units explicitamente identificados;
- não usar non-null assertion para encobrir dados opcionais do domínio.

## 5. Organização de código

Regra de dependência:

```text
UI/features
  ↓
application
  ↓
domain + calculation-engine
  ↑
persistence/catalog adapters
```

O domínio não importa UI/persistência.

## 6. Fórmulas

Toda fórmula nova deve:

- ficar no calculation engine;
- ter `formulaId`/versão quando crítica;
- documentar unidades;
- ter teste;
- definir comportamento para dados inválidos/ausentes;
- declarar se é medição, cálculo, interpolação ou estimativa.

## 7. Números e arredondamento

- não arredondar cálculos intermediários para apresentação;
- formatting ocorre na borda/UI;
- tolerância numérica em testes deve ser explícita;
- usar funções centrais de formatação/unidades.

## 8. Validação

Validar nas bordas:

- formulários;
- import JSON;
- import CSV;
- dados seed;
- dados externos futuros.

Não espalhar validações duplicadas por componentes React.

## 9. Estado

Zustand é indicado para estado de aplicação/UI, mas dados persistidos devem passar por services/repositories quando necessário.

Evitar store global monolítico.

Separar:

- estado de projeto ativo;
- catálogo;
- configurações;
- estado transitório da UI.

## 10. Componentes React

- componentes de UI não contêm regras de engenharia;
- props tipadas;
- separar composição de data fetching/orquestração quando crescer;
- preferir componentes pequenos com responsabilidade clara;
- usar design-system primitives;
- não repetir estilos/token mapping em features.

## 11. CSS / NEXO

- tokens `var(--nexo-*)`;
- sem hex/rgb em features;
- claro/escuro/sistema desde o início;
- foco e acessibilidade como requisitos;
- estilos específicos do DroneCalc devem continuar semânticos.

## 12. Nomes

Código em inglês para tipos/funções/arquivos técnicos. UI e documentação podem ser pt-BR.

Exemplos:

```text
calculateBatteryEnergy
DroneProject
FlightStyleProfile
AnalysisWarning
```

Evitar abreviações obscuras fora de termos consolidados (ESC, GPS, VTX, TWR).

## 13. Erros

Distinguir:

- erro de programação;
- entrada inválida;
- dado ausente esperado;
- indisponibilidade de cálculo;
- falha de persistência.

Dados ausentes não devem gerar stack trace como fluxo normal.

## 14. Warnings estáveis

Códigos de warning funcionam como API interna.

Formato recomendado:

```text
DOMAIN_SUBJECT_CONDITION
ESC_CURRENT_MARGIN_LOW
BATTERY_ESC_VOLTAGE_EXCEEDED
PROP_FRAME_DIAMETER_EXCEEDED
```

Mensagem humana pode ser traduzida; código permanece estável.

## 15. Commits

Preferir commits pequenos e descritivos. Convenção sugerida:

```text
feat: ...
fix: ...
docs: ...
test: ...
refactor: ...
chore: ...
```

Mudança de fórmula deve mencionar isso claramente no PR/commit relevante.

## 16. Pull requests

PR deve informar:

- objetivo;
- escopo;
- arquivos/áreas principais;
- testes executados;
- impacto em fórmulas/dados;
- screenshots para mudanças visuais;
- migração se houver schema change;
- docs atualizadas.

## 17. Dependências

Antes de adicionar pacote:

- confirmar necessidade;
- avaliar manutenção/tamanho;
- evitar dependência para lógica trivial;
- não acoplar o calculation engine a biblioteca de UI;
- registrar decisão se for estrutural.

## 18. Segurança de software

- sem secrets no repositório;
- validar arquivos importados;
- não usar `eval`;
- sanitizar conteúdo se futuramente houver HTML externo;
- dependências atualizadas de forma controlada;
- export/import não executa código.

## 19. Performance

Evitar otimização prematura. Primeiro:

- funções puras;
- memoização apenas onde medido/necessário;
- análise incremental pode ser adicionada se projetos grandes causarem custo;
- curvas e otimizador futuro são candidatos a worker.

## 20. Documentação no código

Comentários devem explicar **por quê**, não repetir o código.

Fórmulas críticas devem mencionar `formulaId` e apontar para `docs/CALCULATION_ENGINE.md` quando apropriado.

## 21. Definition of Done

Uma alteração está pronta quando:

- typecheck passa;
- lint passa;
- testes passam;
- novos comportamentos têm testes;
- UI usa NEXO;
- dados externos são validados;
- mudanças de contrato atualizaram docs;
- nenhuma estimativa perdeu proveniência/confiança.
