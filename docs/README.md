# Documentação do DroneCalc

Este diretório contém a fonte de verdade do produto e da implementação do DroneCalc.

## Ordem recomendada de leitura

1. [`PRD.md`](PRD.md) — problema, público, objetivos, escopo e critérios de sucesso.
2. [`PRODUCT_SPEC.md`](PRODUCT_SPEC.md) — comportamento funcional esperado da aplicação.
3. [`USER_STORIES.md`](USER_STORIES.md) — histórias de usuário e critérios de aceite.
4. [`REQUIREMENTS_TRACEABILITY.md`](REQUIREMENTS_TRACEABILITY.md) — ligação entre requisitos, specs, roadmap e testes.
5. [`ARCHITECTURE.md`](ARCHITECTURE.md) — estrutura técnica e separação de responsabilidades.
6. [`DOMAIN_MODEL.md`](DOMAIN_MODEL.md) — entidades, tipos e contratos centrais.
7. [`CALCULATION_ENGINE.md`](CALCULATION_ENGINE.md) — fórmulas, hipóteses, proveniência e confiança.
8. [`FLIGHT_PROFILES.md`](FLIGHT_PROFILES.md) — estilos de voo e critérios de avaliação.
9. [`NEXO_DESIGN_SYSTEM.md`](NEXO_DESIGN_SYSTEM.md) — identidade visual herdada do NEXO.
10. [`UX_SPEC.md`](UX_SPEC.md) — fluxos, navegação e estados de interface.
11. [`DATA_MODEL.md`](DATA_MODEL.md) — persistência, importação/exportação e versionamento.
12. [`COMPONENT_CATALOG.md`](COMPONENT_CATALOG.md) — categorias, campos e dados de bancada.
13. [`TEST_STRATEGY.md`](TEST_STRATEGY.md) — testes unitários, integração e validação numérica.
14. [`SAFETY_AND_LIMITATIONS.md`](SAFETY_AND_LIMITATIONS.md) — limites, incerteza e comunicação responsável.
15. [`RELEASE_CRITERIA.md`](RELEASE_CRITERIA.md) — gates para etapas, MVP e releases.
16. [`ROADMAP.md`](ROADMAP.md) — fases e dependências de implementação.
17. [`DEVELOPMENT.md`](DEVELOPMENT.md) — padrões de desenvolvimento.
18. [`AI_IMPLEMENTATION_GUIDE.md`](AI_IMPLEMENTATION_GUIDE.md) — contrato geral para agentes de IA.
19. [`prompts/ETAPA_1A_BOOTSTRAP_FULLSTACK.md`](prompts/ETAPA_1A_BOOTSTRAP_FULLSTACK.md) — primeira orientação executável para a IA implementadora.
20. [`GLOSSARY.md`](GLOSSARY.md) — terminologia técnica padronizada.
21. [`adr/`](adr/README.md) — Architecture Decision Records.

Na raiz do repositório também existem:

- [`../CONTRIBUTING.md`](../CONTRIBUTING.md) — guia de contribuição;
- [`../CHANGELOG.md`](../CHANGELOG.md) — histórico de mudanças relevantes.

## Arquitetura vigente

O MVP ampliado adota:

- Web React/TypeScript;
- API Node.js/TypeScript;
- PostgreSQL como fonte principal de dados;
- MinIO/S3-compatible para imagens e documentos;
- Ollama atrás de interface substituível para IA local;
- IndexedDB apenas para cache/drafts/preferências/offline auxiliar;
- calculation engine e domínio independentes de UI, banco e IA.

A mudança é detalhada no [`adr/0002-postgresql-object-storage-ollama.md`](adr/0002-postgresql-object-storage-ollama.md).

## Hierarquia de autoridade

Quando documentos entrarem em conflito, utilizar esta prioridade:

1. `PRD.md` para intenção de produto e escopo.
2. `PRODUCT_SPEC.md` para comportamento funcional.
3. `CALCULATION_ENGINE.md` para regras matemáticas.
4. `DOMAIN_MODEL.md` e `DATA_MODEL.md` para contratos de dados.
5. `ARCHITECTURE.md` e ADRs para limites e decisões técnicas.
6. `NEXO_DESIGN_SYSTEM.md` e `UX_SPEC.md` para apresentação e interação.
7. `ROADMAP.md` para sequência de implementação.

Prompts de implementação operacionalizam os documentos superiores e nunca devem contradizê-los silenciosamente.

## Regra de atualização

Mudanças que alterem uma regra superior devem atualizar os documentos dependentes no mesmo pull request. Em especial:

- fórmula alterada → `CALCULATION_ENGINE.md`, testes e changelog quando material;
- schema alterado → `DOMAIN_MODEL.md`/`DATA_MODEL.md`, migration e testes;
- requisito alterado → PRD/Product Spec, user stories e rastreabilidade;
- arquitetura alterada → `ARCHITECTURE.md` e ADR;
- UI/identidade alterada → `NEXO_DESIGN_SYSTEM.md`/`UX_SPEC.md`;
- etapa concluída → `ROADMAP.md` e relatório da etapa;
- orientação da IA alterada → prompt correspondente e `AI_IMPLEMENTATION_GUIDE.md` quando a regra for geral.

## Estado da especificação

Versão da arquitetura/roadmap: **0.2**  
Status: **planejamento / pré-MVP**

A documentação deve evoluir junto com o código. Não manter decisões relevantes apenas em issues, chats ou prompts temporários.
