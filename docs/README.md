# Documentação do DroneCalc

Este diretório contém a fonte de verdade do produto e da implementação do DroneCalc.

## Ordem recomendada de leitura

1. [`PRD.md`](PRD.md) — problema, público, objetivos, escopo e critérios de sucesso.
2. [`PRODUCT_SPEC.md`](PRODUCT_SPEC.md) — comportamento funcional esperado da aplicação.
3. [`ARCHITECTURE.md`](ARCHITECTURE.md) — estrutura técnica e separação de responsabilidades.
4. [`DOMAIN_MODEL.md`](DOMAIN_MODEL.md) — tipos e entidades centrais.
5. [`CALCULATION_ENGINE.md`](CALCULATION_ENGINE.md) — regras matemáticas, fontes e confiança.
6. [`FLIGHT_PROFILES.md`](FLIGHT_PROFILES.md) — como estilos de voo influenciam o dimensionamento.
7. [`NEXO_DESIGN_SYSTEM.md`](NEXO_DESIGN_SYSTEM.md) — identidade visual herdada do NEXO.
8. [`UX_SPEC.md`](UX_SPEC.md) — fluxos, navegação e estados de interface.
9. [`DATA_MODEL.md`](DATA_MODEL.md) — persistência, importação/exportação e versionamento.
10. [`COMPONENT_CATALOG.md`](COMPONENT_CATALOG.md) — esquema de componentes e dados de bancada.
11. [`TEST_STRATEGY.md`](TEST_STRATEGY.md) — testes unitários, integração e validação numérica.
12. [`SAFETY_AND_LIMITATIONS.md`](SAFETY_AND_LIMITATIONS.md) — comunicação de limites e incerteza.
13. [`ROADMAP.md`](ROADMAP.md) — fases e dependências de implementação.
14. [`DEVELOPMENT.md`](DEVELOPMENT.md) — convenções para desenvolvimento.
15. [`AI_IMPLEMENTATION_GUIDE.md`](AI_IMPLEMENTATION_GUIDE.md) — contrato para agentes de IA que implementarem o projeto.

## Hierarquia de autoridade

Quando documentos entrarem em conflito, utilizar esta prioridade:

1. `PRD.md` para intenção de produto e escopo.
2. `PRODUCT_SPEC.md` para comportamento funcional.
3. `CALCULATION_ENGINE.md` para regras matemáticas.
4. `DOMAIN_MODEL.md` e `DATA_MODEL.md` para contratos de dados.
5. `ARCHITECTURE.md` para limites técnicos.
6. `NEXO_DESIGN_SYSTEM.md` e `UX_SPEC.md` para apresentação e interação.
7. `ROADMAP.md` para sequência de implementação.

Mudanças que alterem uma regra superior devem atualizar os documentos dependentes no mesmo pull request.

## Estado da especificação

Versão inicial: **0.1**  
Status: **planejamento / pré-MVP**

A documentação deve evoluir junto com o código. Não manter decisões relevantes apenas em issues, chats ou prompts de IA.
