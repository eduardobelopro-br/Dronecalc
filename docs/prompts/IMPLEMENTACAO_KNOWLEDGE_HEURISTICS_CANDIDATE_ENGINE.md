# Prompt de Implementação — Knowledge / Heuristics / Candidate Engine

**Projeto:** DroneCalc  
**Repositório:** `eduardobelopro-br/Dronecalc`  
**Status:** prompt mestre preparado para execução por fases  
**Importante:** não executar esta feature fora de ordem; primeiro respeitar os gates do roadmap global.

---

# 1. PROMPT PARA A IA IMPLEMENTADORA

Copie a seção abaixo integralmente para a IA responsável pela implementação.

```text
Você é o engenheiro de software responsável por implementar o subsistema de
Base de Conhecimento, Heurísticas e Geração de Candidatos do DroneCalc.

REPOSITÓRIO
eduardobelopro-br/Dronecalc

OBJETIVO
Construir um subsistema que transforme referências técnicas estruturadas em
heurísticas rastreáveis e use essas heurísticas para sugerir candidatos de
componentes/configurações, sem confundir recomendação com validação física.

ANTES DE ALTERAR QUALQUER CÓDIGO
1. Leia README.md.
2. Leia docs/PRD.md.
3. Leia docs/PRODUCT_SPEC.md.
4. Leia docs/ARCHITECTURE.md.
5. Leia docs/DOMAIN_MODEL.md.
6. Leia docs/CALCULATION_ENGINE.md.
7. Leia docs/COMPONENT_CATALOG.md.
8. Leia docs/FLIGHT_PROFILES.md.
9. Leia docs/KNOWLEDGE_HEURISTICS.md.
10. Leia docs/KNOWLEDGE_ENGINE_ROADMAP.md.
11. Leia docs/TEST_STRATEGY.md.
12. Leia docs/DEVELOPMENT.md.
13. Leia docs/AI_IMPLEMENTATION_GUIDE.md.
14. Inspecione o código real e determine quais etapas do roadmap já foram
    concluídas. O código é a fonte de verdade sobre o estado de implementação.

NÃO PRESUMA QUE O REPOSITÓRIO JÁ POSSUI A STACK PLANEJADA.
Verifique package.json, scripts, estrutura e dependências antes de importar ou
instalar qualquer biblioteca.

GATE DE EXECUÇÃO
- Se Etapa 1A ainda não estiver concluída, NÃO implemente o Knowledge Engine.
  Registre que a dependência não está satisfeita e indique Etapa 1A como próximo
  passo.
- Se o bootstrap existir, implemente somente a próxima fase K cujo pré-requisito
  esteja satisfeito.
- Não pule diretamente para Candidate Generator se os schemas de domínio,
  catálogo, constraints e perfis necessários ainda não existirem.

PRINCÍPIO CENTRAL
"normalmente se usa" != "é compatível" != "é a melhor configuração".

Heurística serve para gerar/priorizar candidatos. Validators de engenharia têm
precedência e podem rejeitar candidatos.

REGRAS INEGOCIÁVEIS
- Nenhuma fórmula física em componente React.
- Nenhuma heurística pode produzir um success de compatibilidade por si só.
- AnalysisWarning severity=danger conhecido elimina o candidato.
- Uma heurística nunca pode reabilitar um candidato rejeitado por hard
  constraint ou danger.
- Dados ausentes permanecem ausentes; nunca converter undefined/null em 0.
- KV sem contexto de tensão/células e hélice não pode aprovar nem filtrar
  positivamente uma combinação.
- Empuxo não é propriedade do motor isolado; requer combinação testada.
- Não extrapolar curvas/tabelas silenciosamente.
- Toda heurística deve possuir evidência e revisão.
- Dado extraído por IA começa como unreviewed.
- O motor deve ser determinístico.
- Resultados devem ser explicáveis.
- Não criar backend se a etapa não exigir.
- Não criar crawler, ML ou dependência de LLM em runtime para este MVP.
- Não criar commit, branch ou PR sem autorização explícita do usuário.

AUTONOMIA TÉCNICA DA IA
Você ESTÁ AUTORIZADO a modificar a arquitetura, os tipos e o código de referência
abaixo se encontrar uma solução objetivamente mais simples, mais eficiente,
mais segura, mais testável ou mais coerente com o código real do repositório.

Você pode:
- renomear tipos e módulos;
- alterar a estrutura de pastas;
- trocar representação de ranges;
- usar discriminated unions em vez de interfaces;
- melhorar algoritmos de matching;
- reduzir abstrações desnecessárias;
- utilizar estruturas indexadas para performance;
- ajustar schemas;
- criar value objects em vez de branded primitives;
- fundir ou separar módulos se isso melhorar coesão/acoplamento;
- escolher outro mecanismo de persistência já adotado pelo projeto;
- melhorar a estratégia de testes.

Mas qualquer mudança deve:
1. preservar requisitos funcionais e de segurança;
2. não reduzir rastreabilidade;
3. não ocultar dados ausentes;
4. manter validators físicos como autoridade;
5. preservar determinismo;
6. adicionar/ajustar testes;
7. atualizar documentação afetada;
8. explicar no relatório por que a alternativa é superior.

Se a documentação sugerir algo pior que o código real, não replique o erro.
Implemente a alternativa correta e atualize a documentação no mesmo conjunto de
mudanças.

PROCESSO OBRIGATÓRIO
Entender requisitos
→ inspecionar estado atual
→ identificar riscos
→ escolher menor fase implementável
→ definir contratos
→ implementar domínio puro
→ implementar validação/schema
→ testes unitários
→ integração apenas se pré-requisitos existirem
→ executar quality gates
→ auto-auditoria
→ atualizar docs
→ gerar relatório.

QUALITY GATES
Execute os scripts realmente existentes e registre o resultado. Quando
existirem, priorize:

npm run typecheck
npm run lint
npm run test
npm run build

Não afirme que passou se não executou.
Não remova nem enfraqueça teste para obter verde.

RELATÓRIO
Ao final, crie:

docs/reports/RELATORIO_KNOWLEDGE_ENGINE_KX_<NOME_DA_FASE>.md

com:
- objetivo;
- pré-requisitos verificados;
- estado anterior;
- arquivos criados/alterados;
- decisões técnicas;
- divergências do código de referência;
- justificativa de melhorias;
- testes adicionados;
- comandos executados e resultado real;
- riscos residuais;
- pendências;
- próxima fase recomendada.
```

---

# 2. CÓDIGO DE REFERÊNCIA

O código abaixo define **contratos de intenção**, não uma implementação obrigatória. A IA pode melhorá-lo conforme as regras do prompt.

## 2.1 Evidence

Arquivo sugerido:

```text
src/domain/knowledge/evidence.ts
```

```ts
export type EvidenceKind =
  | 'bench-test'
  | 'manufacturer'
  | 'technical-video'
  | 'technical-article'
  | 'community-forum'
  | 'user-provided'
  | 'academic'
  | 'other'

export type EvidenceReviewStatus =
  | 'unreviewed'
  | 'reviewed'
  | 'validated'
  | 'rejected'
  | 'superseded'

export interface EvidenceLocator {
  timestampSeconds?: number
  page?: number
  section?: string
}

export interface EvidenceReference {
  id: string
  schemaVersion: number
  kind: EvidenceKind
  title?: string
  authorOrPublisher?: string
  url?: string
  publishedAt?: string
  accessedAt?: string
  locator?: EvidenceLocator
  reviewStatus: EvidenceReviewStatus
  notes?: readonly string[]
}
```

### Invariantes

- `id` não vazio;
- `schemaVersion >= 1`;
- timestamp >= 0;
- datas externas validadas nas bordas;
- não obrigar URL para fonte manual;
- `validated` nunca deve ser inferido automaticamente de `kind`.

---

## 2.2 Numeric range

Arquivo sugerido:

```text
src/domain/common/numericRange.ts
```

```ts
export interface NumericRange {
  readonly min: number
  readonly max: number
}

export function isValidRange(range: NumericRange): boolean {
  return (
    Number.isFinite(range.min) &&
    Number.isFinite(range.max) &&
    range.min <= range.max
  )
}

export function contains(range: NumericRange, value: number): boolean {
  return value >= range.min && value <= range.max
}
```

Se o sistema de units já possuir um range genérico tipado, REUTILIZE-O em vez de criar duplicação.

---

## 2.3 Motor stator heuristic

```ts
import type { NumericRange } from '../common/numericRange'

export interface MotorStatorHeuristic {
  readonly diameterMm?: NumericRange
  readonly heightMm?: NumericRange
}
```

A notação `14xx` deve significar somente diâmetro conhecido; não inventar altura.

---

## 2.4 Sizing heuristic

Arquivo sugerido:

```text
src/domain/knowledge/sizingHeuristic.ts
```

```ts
import type { NumericRange } from '../common/numericRange'
import type { MotorStatorHeuristic } from './motorStatorHeuristic'

export type HeuristicConfidence = 'high' | 'medium' | 'low'
export type HeuristicStrength = 'weak' | 'medium' | 'strong'

export interface HeuristicDimensions {
  readonly frameWheelbaseMm?: NumericRange
  readonly propDiameterIn?: NumericRange
  readonly batterySeriesCells?: NumericRange
  readonly motorKv?: NumericRange
  readonly stator?: MotorStatorHeuristic
  readonly flightProfileIds?: readonly string[]
  readonly maxTakeoffMassG?: NumericRange
}

export interface SizingHeuristic {
  readonly id: string
  readonly schemaVersion: number
  readonly revision: number
  readonly title: string
  readonly applicability: HeuristicDimensions
  readonly suggestion: HeuristicDimensions
  readonly evidenceIds: readonly string[]
  readonly confidence: HeuristicConfidence
  readonly strength: HeuristicStrength
  readonly missingContext?: readonly string[]
  readonly notes?: readonly string[]
  readonly active: boolean
}
```

A IA pode preferir uma union por tipo de heurística se isso impedir estados inválidos. Essa é uma melhoria aceitável e provavelmente superior quando o domínio crescer.

---

## 2.5 Contexto normalizado para avaliação

Não faça o evaluator conhecer React, formulário ou DTO externo.

```ts
export interface HeuristicEvaluationContext {
  readonly frameWheelbaseMm?: number
  readonly propDiameterIn?: number
  readonly batterySeriesCells?: number
  readonly batteryNominalVoltageV?: number
  readonly motorKv?: number
  readonly motorStatorDiameterMm?: number
  readonly motorStatorHeightMm?: number
  readonly takeoffMassG?: number
  readonly flightProfileId?: string
}
```

Quando o sistema de units estiver pronto, adapte estes números ao contrato canônico vigente.

---

## 2.6 Resultado do evaluator

```ts
export type HeuristicMatchStatus =
  | 'match'
  | 'partial-match'
  | 'mismatch'
  | 'not-applicable'
  | 'insufficient-context'

export interface HeuristicEvaluation {
  readonly heuristicId: string
  readonly status: HeuristicMatchStatus
  readonly confidence: 'high' | 'medium' | 'low'
  readonly scoreDelta: number
  readonly reasons: readonly string[]
  readonly evidenceIds: readonly string[]
  readonly missingContext?: readonly string[]
}
```

---

## 2.7 Evaluator

Arquivo sugerido:

```text
src/calculation-engine/knowledge/evaluateHeuristic.ts
```

Referência de implementação:

```ts
import type {
  HeuristicEvaluation,
  HeuristicEvaluationContext,
  SizingHeuristic,
} from '../../domain/knowledge'

export function evaluateHeuristic(
  heuristic: SizingHeuristic,
  context: HeuristicEvaluationContext,
): HeuristicEvaluation {
  if (!heuristic.active) {
    return {
      heuristicId: heuristic.id,
      status: 'not-applicable',
      confidence: heuristic.confidence,
      scoreDelta: 0,
      reasons: ['Heurística desativada.'],
      evidenceIds: heuristic.evidenceIds,
    }
  }

  const missing = detectRequiredMissingContext(heuristic, context)

  if (missing.length > 0) {
    return {
      heuristicId: heuristic.id,
      status: 'insufficient-context',
      confidence: 'low',
      scoreDelta: 0,
      reasons: ['Contexto insuficiente para aplicar a heurística.'],
      evidenceIds: heuristic.evidenceIds,
      missingContext: missing,
    }
  }

  const checks = evaluateApplicableDimensions(heuristic, context)

  if (checks.length === 0) {
    return {
      heuristicId: heuristic.id,
      status: 'not-applicable',
      confidence: heuristic.confidence,
      scoreDelta: 0,
      reasons: ['Nenhuma dimensão aplicável ao contexto atual.'],
      evidenceIds: heuristic.evidenceIds,
    }
  }

  const matched = checks.filter((check) => check.matched).length

  if (matched === checks.length) {
    return {
      heuristicId: heuristic.id,
      status: 'match',
      confidence: heuristic.confidence,
      scoreDelta: scoreForStrength(heuristic.strength),
      reasons: checks.map((check) => check.reason),
      evidenceIds: heuristic.evidenceIds,
    }
  }

  if (matched > 0) {
    return {
      heuristicId: heuristic.id,
      status: 'partial-match',
      confidence: heuristic.confidence,
      scoreDelta: 0,
      reasons: checks.map((check) => check.reason),
      evidenceIds: heuristic.evidenceIds,
    }
  }

  return {
    heuristicId: heuristic.id,
    status: 'mismatch',
    confidence: heuristic.confidence,
    scoreDelta: 0,
    reasons: checks.map((check) => check.reason),
    evidenceIds: heuristic.evidenceIds,
  }
}
```

**Importante:** `detectRequiredMissingContext`, `evaluateApplicableDimensions` e `scoreForStrength` são nomes conceituais. Não invente APIs existentes. Implemente funções locais/puras ou substitua por desenho melhor.

### Regra especial de KV

Se uma heurística contém sugestão/faixa de KV mas a evidência declara falta de `battery-voltage-or-series-cells`, o evaluator não pode gerar score positivo por KV.

Uma implementação melhor pode representar essa limitação estruturalmente no schema para tornar o estado inválido impossível.

---

# 3. CANDIDATE GENERATOR

Implementar somente quando catálogo, projeto, constraints e validators necessários já existirem.

## 3.1 Entrada

```ts
export interface CandidateGenerationInput<TCatalogItem> {
  readonly flightProfileId?: string
  readonly constraints: ProjectConstraints
  readonly partialProject: DroneProject
  readonly catalogItems: readonly TCatalogItem[]
  readonly heuristics: readonly SizingHeuristic[]
}
```

A IA deve adaptar aos tipos reais do repositório, não duplicá-los.

## 3.2 Saída

```ts
export type CandidateEligibility =
  | 'eligible'
  | 'conditionally-eligible'
  | 'rejected'

export interface CandidateSuggestion<T> {
  readonly candidate: T
  readonly eligibility: CandidateEligibility
  readonly heuristicScore: number
  readonly heuristicEvaluations: readonly HeuristicEvaluation[]
  readonly engineeringWarnings: readonly AnalysisWarning[]
  readonly missingData: readonly string[]
  readonly evidenceIds: readonly string[]
}
```

## 3.3 Pipeline de referência

```ts
export function generateCandidates<T>(
  input: CandidateGenerationInput<T>,
  deps: CandidateGeneratorDependencies<T>,
): CandidateSuggestion<T>[] {
  return input.catalogItems
    .map((candidate) => {
      const constraintResult = deps.evaluateConstraints(candidate, input)

      if (constraintResult.rejected) {
        return deps.toRejectedSuggestion(candidate, constraintResult)
      }

      const engineeringWarnings = deps.validateEngineering(candidate, input)
      const hasDanger = engineeringWarnings.some(
        (warning) => warning.severity === 'danger',
      )

      if (hasDanger) {
        return deps.toRejectedSuggestion(candidate, {
          engineeringWarnings,
        })
      }

      const context = deps.buildHeuristicContext(candidate, input)
      const evaluations = input.heuristics.map((heuristic) =>
        evaluateHeuristic(heuristic, context),
      )

      return deps.toSuggestion({
        candidate,
        engineeringWarnings,
        evaluations,
      })
    })
    .sort(deps.compareSuggestions)
}
```

Esse desenho com `deps` é apenas uma opção para manter o núcleo testável. Se houver uma solução mais simples/coesa no código real, prefira-a.

### Determinismo

Se scores forem iguais, usar tie-break estável e explícito, por exemplo ID estável. Nunca depender incidentalmente da ordem de objeto/hash.

---

# 4. FIXTURE INICIAL DA REFERÊNCIA 4"

A tabela apresentada como referência técnica informou aproximadamente:

```text
frame 150–165 mm
hélice 4"
famílias 14xx / 15xx / 16xx
KV 2500–4500
```

Como a referência não informa tensão/células, o fixture deve preservar essa ausência.

Exemplo conceitual:

```ts
export const referenceFourInchHeuristic: SizingHeuristic = {
  id: 'ref-video-frame-prop-motor-4in-v1',
  schemaVersion: 1,
  revision: 1,
  title: 'Referência de dimensionamento para frame 150–165 mm e hélice 4"',
  applicability: {
    frameWheelbaseMm: { min: 150, max: 165 },
    propDiameterIn: { min: 4, max: 4 },
  },
  suggestion: {
    stator: {
      diameterMm: { min: 14, max: 16 },
    },
    motorKv: { min: 2500, max: 4500 },
  },
  evidenceIds: ['evidence-video-sizing-table-001'],
  confidence: 'low',
  strength: 'weak',
  missingContext: ['battery-voltage-or-series-cells'],
  notes: [
    'Faixa de KV não deve validar combinação sem tensão/células.',
    '14xx/15xx/16xx foram normalizados apenas por diâmetro de estator.',
  ],
  active: true,
}
```

### Cuidado

`diameterMm: 14–16` é uma aproximação contínua da notação de famílias. Se o domínio precisar distinguir somente famílias discretas 14/15/16, prefira:

```ts
statorDiameterMm: [14, 15, 16]
```

Isso pode ser mais correto que um range porque não sugere 14.5 mm como família explicitamente citada. A IA está autorizada a adotar essa melhoria.

---

# 5. TESTES DE REFERÊNCIA

## 5.1 KV sem bateria

```ts
it('does not positively score KV guidance when battery context is missing', () => {
  const result = evaluateHeuristic(referenceFourInchHeuristic, {
    frameWheelbaseMm: 160,
    propDiameterIn: 4,
    motorKv: 2750,
    motorStatorDiameterMm: 14,
  })

  expect(result.status).toBe('insufficient-context')
  expect(result.scoreDelta).toBe(0)
  expect(result.missingContext).toContain('battery-voltage-or-series-cells')
})
```

A asserção exata pode mudar se o schema for redesenhado, mas o comportamento de segurança deve permanecer.

## 5.2 Danger prevalece

```ts
it('rejects a candidate when engineering validation returns danger', () => {
  const result = generateCandidates(input, {
    ...deps,
    validateEngineering: () => [
      {
        code: 'ESC_INPUT_VOLTAGE_EXCEEDED',
        severity: 'danger',
        title: 'Tensão acima do limite',
        message: 'A tensão cheia da bateria excede o limite do ESC.',
      },
    ],
  })

  expect(result[0]?.eligibility).toBe('rejected')
})
```

## 5.3 Heurística não reabilita danger

Criar caso em que o candidato teria score heurístico máximo, mas possui `danger`. Resultado continua rejeitado.

## 5.4 Limites inclusivos

Testar exatamente 150 mm, 165 mm e 4".

## 5.5 Determinismo

Rodar a mesma entrada várias vezes e comparar saída profunda.

---

# 6. CASO DE ACEITE — MINI LONG RANGE

Criar fixtures controladas, não números de fabricante inventados.

Candidato A:

```text
maior empuxo máximo
maior consumo no regime de hover
cumpre constraints
```

Candidato B:

```text
menor empuxo máximo
melhor eficiência g/W em hover
melhor autonomia estimada
cumpre reserva mínima de potência
cumpre constraints
```

Para perfil Mini Long Range, B deve poder obter score final superior.

O teste deve provar que o sistema não confunde “mais potência” com “melhor para long range”.

---

# 7. ESTRUTURA DE DIRETÓRIOS DE REFERÊNCIA

Quando as fases correspondentes existirem:

```text
src/
├── domain/
│   └── knowledge/
│       ├── evidence.ts
│       ├── heuristic.ts
│       ├── evaluation.ts
│       └── index.ts
├── calculation-engine/
│   └── knowledge/
│       ├── evaluateHeuristic.ts
│       ├── matching.ts
│       └── index.ts
├── application/
│   └── candidate-generation/
│       ├── generateCandidates.ts
│       ├── candidateTypes.ts
│       └── index.ts
├── persistence/
│   └── repositories/
│       ├── EvidenceRepository.ts
│       └── HeuristicRepository.ts
└── catalog/
    └── seed/
        └── heuristics/
```

A IA deve adaptar a estrutura ao código real e evitar diretórios com um único arquivo sem benefício arquitetural.

---

# 8. AUTO-AUDITORIA OBRIGATÓRIA

Antes de encerrar, responda tecnicamente:

1. Algum `undefined` pode virar zero?
2. Alguma heurística consegue produzir compatibilidade positiva?
3. Algum `danger` pode ser mascarado por score?
4. KV está sendo avaliado sem tensão?
5. A notação `14xx` está inventando altura de estator?
6. Existe regra de domínio dentro de React?
7. Há dependência circular?
8. Ordem de candidatos é determinística?
9. Fonte/revisão da heurística é recuperável?
10. Dados extraídos por IA podem virar `validated` automaticamente?
11. Testes cobrem limites e contexto ausente?
12. Foi adicionada dependência que não era necessária?
13. A solução ficou mais abstrata que o problema real exige?
14. Há duplicação de tipos já existentes no domínio?
15. Documentação e código ainda concordam?

Se encontrar falha, corrija antes de declarar a etapa concluída.

---

# 9. CRITÉRIO DE CONCLUSÃO

A fase só está concluída quando:

```text
implementação da menor fase K elegível
+ schemas/tipos coerentes com domínio real
+ testes
+ quality gates executados
+ documentação atualizada
+ relatório de etapa
+ auto-auditoria sem achado crítico aberto
```

Não ampliar o escopo apenas para “entregar mais”. Correção, rastreabilidade e testabilidade têm prioridade.
