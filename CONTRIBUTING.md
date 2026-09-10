# Contribuindo com o DroneCalc

Obrigado pelo interesse no DroneCalc. Como o projeto combina software e cálculos técnicos, contribuições devem preservar rastreabilidade, unidades e testes.

## Antes de contribuir

Leia:

1. `README.md`;
2. `docs/PRD.md`;
3. `docs/PRODUCT_SPEC.md`;
4. `docs/ARCHITECTURE.md`;
5. `docs/DEVELOPMENT.md`;
6. os documentos específicos da área alterada.

Mudanças matemáticas exigem leitura de `docs/CALCULATION_ENGINE.md`. Mudanças visuais exigem `docs/NEXO_DESIGN_SYSTEM.md` e `docs/UX_SPEC.md`.

## Regras essenciais

- não colocar fórmulas de engenharia na UI;
- não inventar valores para dados ausentes;
- manter unidade explícita;
- distinguir medição, fabricante, cálculo, interpolação e estimativa;
- não extrapolar curvas silenciosamente;
- usar tokens NEXO na interface;
- adicionar testes para novos comportamentos;
- atualizar documentação quando contratos mudarem.

## Fluxo sugerido

1. crie branch específica;
2. implemente mudança pequena/coesa;
3. execute typecheck/lint/test/build;
4. atualize documentação;
5. abra PR descrevendo impactos.

## Pull request

Inclua:

- objetivo;
- o que mudou;
- decisões técnicas;
- testes executados e resultado;
- impacto em cálculo/schema;
- screenshots para mudanças visuais;
- migração quando aplicável;
- limitações ou pendências.

## Fórmulas

Toda mudança de fórmula crítica deve possuir:

- identificador/versão;
- unidades;
- hipóteses;
- caso de referência;
- testes de limite;
- atualização de `docs/CALCULATION_ENGINE.md`.

## Catálogo

Dados reais devem registrar fonte. Não adicionar grandes lotes de componentes sem governança/validação. É preferível um registro parcial explicitamente marcado do que um valor preenchido por suposição.

## Segurança

Não descreva resultados calculados como garantia de segurança ou certificação. Consulte `docs/SAFETY_AND_LIMITATIONS.md`.

## IA

Contribuições assistidas por IA são bem-vindas. O agente deve seguir `docs/AI_IMPLEMENTATION_GUIDE.md`. A responsabilidade por revisar alterações, testes e fontes permanece no fluxo do projeto.
