# Roadmap — Base de Conhecimento, Heurísticas e Recomendação

**Versão:** 0.1  
**Status:** planejado  
**Dependência:** este roadmap complementa `docs/ROADMAP.md` e não substitui a sequência global do projeto.

## 1. Objetivo

Criar um subsistema capaz de transformar referências técnicas em conhecimento estruturado e utilizá-lo para gerar candidatos de configuração, preservando a separação entre:

```text
referência / heurística
        ↓
gerador de candidatos
        ↓
validação de engenharia
        ↓
cálculo de desempenho
        ↓
score por perfil
```

## 2. Pré-requisitos globais

Antes da implementação completa deste subsistema, o DroneCalc deve possuir:

- Etapa 1A — bootstrap;
- Etapa 1C — units/validação física;
- Etapa 2A/2B — modelos de componentes e DroneProject;
- Etapa 5A — catálogo básico;
- Etapa 8A — schema de perfis antes da integração final com scoring.

Partes puras de domínio/schema podem ser preparadas antes, mas não devem forçar dependências prematuras.

## 3. Fase K0 — Especificação

**Status:** concluída nesta revisão.

Entregas:

- `KNOWLEDGE_HEURISTICS.md`;
- separação entre evidência, heurística e validação;
- modelo inicial de fonte;
- modelo de regra;
- regras de segurança;
- caso de referência Mini Long Range.

Critério de saída: contratos claros sem implementação runtime.

---

## 4. Fase K1 — Modelo de evidência

### Objetivo

Implementar tipos e schemas Zod para fontes/evidências.

### Entregas

- `EvidenceKind`;
- `EvidenceReviewStatus`;
- `EvidenceReference`;
- schemas de validação;
- IDs estáveis;
- timestamp/page/section como locator;
- testes de schema;
- fixtures de evidência.

### Critério de aceite

Uma referência de fabricante, vídeo técnico ou teste de bancada pode ser validada, serializada e reaberta sem perda.

---

## 5. Fase K2 — Modelo de heurística

### Objetivo

Representar conhecimento de dimensionamento sem hardcode disperso.

### Entregas

- `NumericRange`;
- `MotorStatorHeuristic`;
- `SizingHeuristic`;
- confidence/strength;
- missing context;
- revisão/versionamento;
- schemas e testes.

### Critério de aceite

Uma regra como `frame 150–165 mm + prop 4" → stator 14xx/15xx/16xx` pode ser representada sem inventar altura de estator, bateria ou condições ausentes.

---

## 6. Fase K3 — Avaliador de heurísticas

### Objetivo

Criar motor puro e determinístico que decide se uma heurística é aplicável.

### Entregas

- `evaluateHeuristic()`;
- estados `match`, `partial-match`, `mismatch`, `not-applicable`, `insufficient-context`;
- reasons explicáveis;
- scoreDelta restrito a recomendação;
- bloqueio de uso positivo de KV sem contexto necessário;
- testes de limites.

### Critério de aceite

Duas execuções com a mesma entrada produzem a mesma saída e nenhuma heurística produz `AnalysisWarning.success`.

---

## 7. Fase K4 — Persistência e curadoria local

### Objetivo

Persistir evidências e heurísticas por interfaces de repository.

### Entregas

- `EvidenceRepository`;
- `HeuristicRepository`;
- implementação local/IndexedDB conforme arquitetura vigente;
- migrations/versionamento;
- CRUD técnico inicial;
- status review/superseded;
- round-trip tests.

### Critério de aceite

Revisões antigas não são sobrescritas silenciosamente e referências permanecem rastreáveis.

---

## 8. Fase K5 — Importação assistida de referências

### Objetivo

Facilitar transformar uma tabela/descrição técnica em dados estruturados sem automatizar confiança.

### MVP

- entrada manual/JSON;
- preview antes de persistir;
- validação;
- campos faltantes explícitos;
- associação a EvidenceReference.

### Futuro

- CSV;
- extração assistida por IA;
- análise de transcrições;
- deduplicação semântica.

### Regra

Extração por IA gera estado `unreviewed`; nunca `validated` automaticamente.

---

## 9. Fase K6 — Candidate Generator

### Objetivo

Gerar candidatos a partir de projeto parcial, catálogo, constraints e heurísticas.

### Pipeline

```text
catálogo elegível
→ hard constraints
→ validators disponíveis
→ disponibilidade mínima de dados
→ heurísticas
→ candidato + razões + evidências
```

### Entregas

- `CandidateGenerationInput`;
- `CandidateSuggestion<T>`;
- eligibility;
- missingData;
- deterministic ordering;
- cobertura de evidência;
- testes.

### Critério de aceite

Um `danger` conhecido elimina candidato e nenhuma heurística consegue restaurá-lo.

---

## 10. Fase K7 — Integração com perfis de voo

### Objetivo

Usar perfil de voo para priorizar candidatos sem misturar scoring com validação.

### Entregas

- profile-aware candidate ranking;
- normalização dos sinais;
- cobertura de dados;
- explanations;
- integração com Etapa 8C do roadmap global.

### Caso obrigatório

Mini Long Range: candidato de menor potência máxima pode vencer por melhor eficiência/autonomia se cumprir constraints e reserva de potência.

---

## 11. Fase K8 — UI explicável

### Objetivo

Mostrar por que algo foi sugerido.

### Exemplo

```text
Motor 1404 / candidato

Por que apareceu:
✓ tamanho frequentemente associado a hélice 4"
✓ massa compatível com objetivo de baixo peso
ℹ KV da referência original não tinha tensão informada e não foi usado para validar

Validação:
✓ tensão compatível segundo catálogo
⚠ curva de bancada ainda ausente

Confiança da recomendação: média
```

### Entregas

- cards de recomendação;
- evidence drill-down;
- missing-context state;
- NEXO tokens;
- acessibilidade.

---

## 12. Fase K9 — Consolidação de múltiplas fontes

### Objetivo

Permitir convergência/divergência entre referências.

### Entregas

- agrupamento por regra equivalente;
- independência de fontes;
- divergência explícita;
- superseded;
- política de confiança.

Não implementar média cega de fontes.

---

## 13. Fase K10 — Integração com Otimizador

O otimizador global da Etapa 13 deve consumir o Candidate Generator, não reimplementar filtros heurísticos.

```text
Candidate Generator
      ↓
Engineering Validation
      ↓
Calculation Engine
      ↓
Profile Scoring
      ↓
Multi-objective Optimizer
```

### Critério de aceite

Não há duas implementações divergentes de geração/filtro de candidatos.

---

## 14. Dependências

```text
Units / Domain
      ↓
Catalog
      ↓
K1 Evidence ─→ K2 Heuristics ─→ K3 Evaluator
      ↓                         ↓
K4 Persistence             Flight Profiles
      ↓                         ↓
K5 Import                   K6 Candidate Generator
                                  ↓
                         Engineering Validators
                                  ↓
                         Calculation Engine
                                  ↓
                         K7 Profile Ranking
                                  ↓
                         K8 Explainability UI
                                  ↓
                         K10 Optimizer
```

## 15. O que não fazer cedo demais

- não criar crawler de internet;
- não criar backend apenas para heurísticas;
- não importar centenas de tabelas sem governança;
- não usar LLM como runtime obrigatório para recomendação;
- não criar ML antes de possuir dados validados;
- não prever empuxo apenas por KV/diâmetro;
- não criar ranking sem hard constraints e missing-data semantics.

## 16. Definition of Done de cada fase

- TypeScript strict;
- schemas externos validados;
- funções puras testadas;
- nenhuma fórmula em React;
- documentação atualizada;
- dados ausentes explícitos;
- evidência rastreável;
- nenhuma heurística supera `danger`;
- comandos de qualidade executados quando existentes;
- relatório de etapa gerado.

## 17. Sequência recomendada no roadmap global

Inserir os marcos assim:

```text
Etapa 5D  → K1/K2: evidências + modelo de heurísticas
Etapa 5E  → K3/K4/K5: evaluator + persistência + curadoria
Etapa 8E  → K6/K7: Candidate Generator + perfis
Etapa 9+  → K8: UI explicável no Builder
Etapa 13  → K9/K10: consolidação e otimizador
```

## 18. Próxima implementação real do repositório

A prioridade global continua sendo **Etapa 1A — Bootstrap**. Este roadmap deixa o subsistema de conhecimento pronto para ser implementado na posição correta, sem antecipar dependências e sem comprometer a arquitetura.
