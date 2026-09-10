# Guia para Implementação Assistida por IA — DroneCalc

**Versão:** 0.1

## 1. Objetivo

Este documento é o contrato operacional para qualquer agente de IA utilizado para implementar, revisar ou refatorar o DroneCalc.

A IA **pode propor e aplicar uma solução mais eficiente** do que a sugerida na documentação quando houver benefício técnico claro, desde que preserve os requisitos do produto, explique a mudança, mantenha compatibilidade ou migração adequada e atualize os documentos afetados.

## 2. Ordem obrigatória de leitura

Antes de alterar código relevante, ler:

1. `README.md`;
2. `docs/PRD.md`;
3. `docs/PRODUCT_SPEC.md`;
4. `docs/ARCHITECTURE.md`;
5. documento específico da área alterada;
6. `docs/TEST_STRATEGY.md`;
7. `docs/DEVELOPMENT.md`.

Mudanças visuais exigem também `docs/NEXO_DESIGN_SYSTEM.md` e `docs/UX_SPEC.md`.

Mudanças matemáticas exigem `docs/CALCULATION_ENGINE.md`.

## 3. Regras inegociáveis

A IA não deve:

- colocar fórmulas de engenharia dentro de componentes React;
- inventar dados técnicos ausentes para produzir resultado;
- tratar `undefined` como zero;
- apresentar estimativa como medição;
- extrapolar curvas de bancada silenciosamente;
- hardcodar cores nas features em vez de tokens NEXO;
- introduzir backend sem necessidade da etapa;
- alterar schemas persistidos sem migração/versionamento;
- remover testes apenas para fazer CI passar;
- trocar arquitetura inteira sem justificativa e documentação;
- afirmar que uma combinação é segura quando apenas foi calculada como compatível segundo os dados.

## 4. Liberdade para melhorar a solução

A IA está autorizada a melhorar:

- estrutura de pastas;
- nomes de tipos/funções;
- algoritmos;
- estratégia de units;
- abstrações de repository;
- composição de componentes;
- performance;
- estratégia de testes;
- biblioteca auxiliar;

se a melhoria:

1. reduz complexidade ou aumenta correção/manutenibilidade;
2. não viola PRD/specs;
3. não reduz transparência dos cálculos;
4. possui testes;
5. é explicada no relatório da etapa;
6. atualiza arquitetura/ADR se estrutural.

## 5. Regra para divergência da documentação

Se a IA identificar uma abordagem melhor:

```text
1. identificar a limitação atual;
2. explicar a alternativa;
3. verificar impacto nos requisitos;
4. implementar a alternativa se segura e dentro do escopo;
5. atualizar documentação;
6. registrar testes e riscos residuais.
```

Não manter documentação sabidamente desatualizada apenas para “seguir o plano”.

## 6. Fluxo de trabalho por etapa

### Antes

- verificar branch/status;
- ler documentos relevantes;
- inspecionar código atual;
- identificar dependências da etapa;
- não repetir implementação já existente.

### Durante

- fazer mudanças coesas;
- manter build utilizável;
- adicionar testes junto da implementação;
- evitar escopo lateral não necessário;
- preservar compatibilidade de dados.

### Depois

Executar, quando disponíveis:

```bash
npm run typecheck
npm run lint
npm run test
npm run build
```

Registrar resultados reais. Não afirmar que testes passaram se não foram executados.

## 7. Relatório obrigatório da etapa

Cada etapa relevante deve gerar/atualizar relatório contendo:

```text
# Relatório — Etapa X

## Objetivo
## Estado anterior
## Implementado
## Arquivos alterados
## Decisões técnicas
## Melhorias em relação ao plano original
## Testes executados
## Resultados
## Pendências
## Riscos
## Próximo passo recomendado
```

Relatórios temporários podem ficar em `docs/reports/` se o projeto adotar essa prática.

## 8. Alteração matemática

Ao criar/mudar fórmula:

- nomear formula/model version;
- indicar unidades das entradas e saída;
- documentar hipótese;
- criar caso de referência;
- testar limites;
- atualizar `CALCULATION_ENGINE.md`;
- indicar se resultados existentes podem mudar.

## 9. Alteração de perfis

Ao mudar pesos/targets:

- versionar perfil;
- testar score;
- validar que `danger` não é mascarado;
- atualizar `FLIGHT_PROFILES.md`;
- explicar impacto em rankings.

## 10. Alteração visual

Ao criar tela/componente:

- reutilizar NEXO primitives/tokens;
- suportar claro/escuro/sistema;
- testar teclado/foco;
- não usar apenas cor para status;
- manter unidades visíveis;
- implementar estados vazio/erro/incompleto.

## 11. Persistência

Ao mudar dados:

- schema explícito;
- migration se necessário;
- round-trip test;
- import inválido não persiste parcialmente;
- export continua autocontido quando aplicável.

## 12. Prompt-base para uma etapa

Usar como template:

```text
Você está implementando uma etapa do projeto DroneCalc.

Repositório: eduardobelopro-br/Dronecalc

Leia primeiro README.md e a documentação em docs/, priorizando PRD.md,
PRODUCT_SPEC.md, ARCHITECTURE.md, DEVELOPMENT.md e os documentos específicos
da etapa.

Implemente a etapa solicitada de forma completa, coesa e testável.

Regras:
- o motor matemático deve permanecer independente de React;
- não invente dados técnicos ausentes;
- todo resultado físico deve ter unidade;
- resultados devem preservar origem e confiança;
- não extrapole curvas de bancada silenciosamente;
- UI deve seguir o NEXO Design System e usar tokens --nexo-*;
- adicione/atualize testes;
- execute typecheck, lint, test e build quando os scripts existirem;
- não remova testes para fazer a implementação passar;
- atualize documentação quando mudar contratos.

Você tem liberdade para substituir a solução sugerida por uma abordagem mais
eficiente, simples ou robusta se identificar uma melhoria clara. Nesse caso,
explique o motivo, preserve os requisitos, teste a alternativa e atualize os
documentos afetados.

Ao final, gere um relatório com o que foi implementado, decisões, testes,
pendências e o próximo passo recomendado.
```

## 13. Prompt específico — próxima etapa 1A

```text
Implemente a Etapa 1A — Bootstrap do DroneCalc conforme docs/ROADMAP.md.

Objetivos:
- inicializar React + TypeScript + Vite;
- TypeScript strict;
- configurar lint/format de forma simples;
- configurar Vitest;
- criar a estrutura arquitetural mínima de diretórios;
- criar scripts dev/build/test/typecheck/lint;
- garantir que build e teste inicial funcionem;
- não implementar ainda fórmulas de drone além de scaffolding estritamente necessário;
- não criar backend.

Preserve README.md e docs/ existentes.

Se a stack/documentação sugerir uma dependência desnecessária nesta etapa,
você pode adiar sua instalação, desde que explique a decisão e não bloqueie as
etapas seguintes.

Ao final, gere docs/reports/RELATORIO_ETAPA_1A_BOOTSTRAP.md com comandos
executados, resultados e próximo passo.
```

## 14. Prompt específico — Etapa 1B

```text
Implemente a Etapa 1B — base visual NEXO.

Use docs/NEXO_DESIGN_SYSTEM.md como especificação e o repositório NEXO apenas
como referência da fonte visual. Mantenha uma cópia própria dos tokens no
DroneCalc; não crie dependência runtime entre repositórios.

Implemente theme system/light/dark e os primitives mínimos definidos no roadmap.
Nenhuma feature deve usar cores hardcoded.

Registre a revisão/origem dos tokens sincronizados e teste os principais estados.
```

## 15. Revisão por IA

Uma IA revisora deve procurar explicitamente:

- unidade incorreta;
- arredondamento prematuro;
- confusão massa × força;
- tensão nominal usada onde deveria ser tensão cheia;
- C-rating tratado como corrente medida;
- extrapolação;
- `undefined → 0`;
- score ocultando incompatibilidade;
- fórmula duplicada na UI;
- cores não-NEXO;
- migration ausente;
- falta de teste de limite.

## 16. Critério de qualidade

O objetivo não é apenas “fazer funcionar”. O código deve deixar claro **o que sabemos, o que calculamos e o que apenas estimamos**.
