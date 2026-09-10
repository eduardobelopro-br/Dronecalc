# Base de Conhecimento e Heurísticas — DroneCalc

**Versão:** 0.1  
**Status:** especificação planejada  
**Objetivo:** transformar referências técnicas incompletas — vídeos, tabelas, fabricantes, artigos e testes — em conhecimento rastreável, sem confundir heurística com validação de engenharia.

## 1. Motivação

Materiais técnicos frequentemente apresentam relações úteis como:

```text
frame → hélice → família de motor → faixa de KV
```

Essas relações ajudam a reduzir o espaço de busca, mas normalmente não contêm todas as condições necessárias para provar compatibilidade. Exemplo: uma faixa de KV sem tensão/células da bateria não pode aprovar uma combinação motor+hélíce.

O DroneCalc deve separar três conceitos:

```text
"normalmente se usa"
        ≠
"é compatível"
        ≠
"é a melhor configuração para este objetivo"
```

## 2. Pipeline arquitetural

```text
Fontes / Evidências
        ↓
Normalização e revisão
        ↓
Base de Conhecimento
        ↓
Heurísticas de dimensionamento
        ↓
Geração de candidatos
        ↓
Validação física / elétrica / mecânica
        ↓
Cálculos de desempenho
        ↓
Score por perfil de voo
        ↓
Ranking explicável
```

Heurísticas geram e priorizam candidatos. Elas **não substituem** validação física.

## 3. Regras inegociáveis

1. Nenhuma heurística isolada pode gerar `success` de compatibilidade.
2. Uma heurística não pode mascarar `danger` produzido pela validação de engenharia.
3. Falta de contexto deve reduzir aplicabilidade/confiança, não virar zero ou suposição silenciosa.
4. KV não pode ser usado como prova de adequação sem contexto de tensão/células e hélice.
5. Empuxo não pertence ao motor isoladamente; pertence à combinação testada motor+hélíce+tensão/condições.
6. Toda regra deve possuir fonte, revisão e estado de revisão.
7. Conhecimento derivado de vídeo/fórum é `reference`, não `measured`, salvo quando os dados de ensaio forem explicitamente reproduzíveis.
8. Regras de segurança/compatibilidade devem viver no motor de engenharia, não na base de heurísticas.
9. Heurística pode sugerir, pontuar ou desencorajar. Rejeição física pertence a constraints/validators.
10. Não extrapolar uma tabela além do domínio declarado sem marcar explicitamente a extrapolação como não suportada.

## 4. Modelo de evidência

Estrutura recomendada:

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

export interface EvidenceReference {
  id: string
  kind: EvidenceKind
  title?: string
  authorOrPublisher?: string
  url?: string
  publishedAt?: string
  accessedAt?: string
  locator?: {
    timestampSeconds?: number
    page?: number
    section?: string
  }
  reviewStatus: EvidenceReviewStatus
  notes?: string[]
}
```

`validated` significa que o conteúdo foi conferido segundo o processo do DroneCalc; não significa certificação de segurança.

## 5. Faixas estruturadas

Evitar strings livres como `2500 a 4500 KV` no motor.

```ts
export interface NumericRange {
  min: number
  max: number
}
```

Unidades devem ser definidas pelo contrato do campo ou por tipos branded do sistema de units.

## 6. Família de motor

Notações como `14xx` devem ser normalizadas para informação estruturada.

```ts
export interface MotorStatorHeuristic {
  diameterMm?: NumericRange
  heightMm?: NumericRange
}
```

Exemplos:

```text
14xx → diameterMm = 14; height desconhecida
1404 → diameterMm = 14; heightMm = 4
```

Não inferir automaticamente dimensões que a fonte não informou.

## 7. Heurística de dimensionamento

```ts
export interface SizingHeuristic {
  id: string
  schemaVersion: number
  revision: number
  title: string

  applicability: {
    frameWheelbaseMm?: NumericRange
    propDiameterIn?: NumericRange
    batterySeriesCells?: NumericRange
    motorKv?: NumericRange
    stator?: MotorStatorHeuristic
    flightProfileIds?: string[]
    maxTakeoffMassG?: NumericRange
  }

  suggestion: {
    propDiameterIn?: NumericRange
    motorKv?: NumericRange
    stator?: MotorStatorHeuristic
  }

  evidenceIds: string[]
  confidence: 'high' | 'medium' | 'low'
  strength: 'weak' | 'medium' | 'strong'
  missingContext?: string[]
  notes?: string[]
  active: boolean
}
```

### 7.1 Separar applicability de suggestion

A fonte pode dizer, por exemplo:

```text
frame 150–165 mm → hélice 4" → motor 14xx/15xx/16xx
```

Nesse caso, frame/hélice podem funcionar como condições e stator como sugestão.

### 7.2 KV sem tensão

Se uma fonte declarar:

```text
4" → 2500–4500 KV
```

mas não informar bateria, registrar:

```ts
missingContext: ['battery-voltage-or-series-cells']
confidence: 'low'
```

Essa faixa pode ser exibida como referência, mas não deve filtrar candidatos nem contribuir positivamente para compatibilidade até existir contexto suficiente e política explícita.

## 8. Resultado da avaliação heurística

```ts
export type HeuristicMatchStatus =
  | 'match'
  | 'partial-match'
  | 'mismatch'
  | 'not-applicable'
  | 'insufficient-context'

export interface HeuristicEvaluation {
  heuristicId: string
  status: HeuristicMatchStatus
  confidence: 'high' | 'medium' | 'low'
  scoreDelta: number
  reasons: string[]
  evidenceIds: string[]
  missingContext?: string[]
}
```

`scoreDelta` é apenas influência de recomendação. Nunca substitui um `AnalysisWarning`.

## 9. Geração de candidatos

Entrada conceitual:

```ts
export interface CandidateGenerationInput {
  flightProfileId?: string
  constraints: ProjectConstraints
  partialProject: DroneProject
  catalog: ComponentCatalogSnapshot
  heuristics: SizingHeuristic[]
}
```

Saída:

```ts
export type CandidateEligibility =
  | 'eligible'
  | 'conditionally-eligible'
  | 'rejected'

export interface CandidateSuggestion<T> {
  candidate: T
  eligibility: CandidateEligibility
  heuristicScore: number
  heuristicEvaluations: HeuristicEvaluation[]
  engineeringWarnings: AnalysisWarning[]
  missingData: string[]
  evidenceIds: string[]
}
```

## 10. Ordem de decisão

O Candidate Generator deve obedecer:

```text
1. constraints explícitas do usuário
2. compatibilidade física/elétrica conhecida
3. disponibilidade mínima de dados
4. heurísticas aplicáveis
5. adequação ao perfil de voo
6. ranking e trade-offs
```

Uma incompatibilidade `danger` conhecida elimina o candidato independentemente de sua pontuação heurística.

## 11. Hard constraints × soft heuristics

### Hard constraints

Exemplos:

- massa máxima definida pelo usuário;
- máximo de células;
- diâmetro máximo de hélice do frame;
- tensão do ESC;
- tensão do motor;
- padrão mecânico incompatível quando conhecido.

Podem rejeitar candidato.

### Soft heuristics

Exemplos:

- família de estator frequentemente usada com hélice 4";
- faixa de tamanho comum em Mini Long Range;
- preferência por baixo peso;
- tendência de menor KV em tensões maiores.

Apenas ordenam/priorizam e precisam ser explicáveis.

## 12. Mini Long Range como caso de validação

O perfil Mini Long Range deve demonstrar o pipeline completo.

Exemplo de intenção:

```text
Perfil: Mini Long Range
Prop: 4"
Restrição opcional: sub-250 g
Prioridade: autonomia/eficiência
```

A heurística pode gerar famílias candidatas de motores. Depois:

```text
candidatos
→ validar frame/prop
→ validar bateria/ESC/motor
→ localizar curvas motor+hélíce+tensão
→ calcular hover e TWR
→ calcular eficiência/autonomia quando possível
→ aplicar score Mini Long Range
→ explicar ranking
```

Um sistema menos potente pode vencer se apresentar melhor eficiência no regime relevante e ainda cumprir a reserva de potência requerida.

## 13. Confiança

Confiança de uma heurística deve considerar:

- qualidade da fonte;
- completude de contexto;
- número de fontes independentes;
- consistência entre fontes;
- existência de dados de bancada;
- revisão humana/técnica.

Não usar uma média cega. Uma fonte duplicada/copied não é evidência independente.

## 14. Conflito entre fontes

Não sobrescrever silenciosamente.

Exemplo:

```text
Fonte A: 1404 recomendado para 4"
Fonte B: 1505 recomendado para 4"
```

Ambas podem coexistir. O sistema deve avaliar contexto e mostrar divergência quando material.

## 15. Dados derivados de vídeo

Para um frame/tabela de vídeo, registrar:

- URL do vídeo;
- título quando conhecido;
- timestamp do frame;
- transcrição relevante resumida;
- dados estruturados extraídos;
- campos ausentes;
- status de revisão.

A imagem/tabela original não precisa ser redistribuída pelo catálogo para que a regra estruturada seja armazenada; respeitar direitos autorais e manter apenas a informação necessária e a referência da fonte.

## 16. Persistência

Repositórios sugeridos:

```ts
export interface EvidenceRepository {
  list(): Promise<EvidenceReference[]>
  get(id: string): Promise<EvidenceReference | null>
  save(value: EvidenceReference): Promise<void>
}

export interface HeuristicRepository {
  listActive(): Promise<SizingHeuristic[]>
  get(id: string): Promise<SizingHeuristic | null>
  save(value: SizingHeuristic): Promise<void>
}
```

No MVP, implementação local é suficiente.

## 17. Versionamento

Versionar separadamente:

- schema de evidência;
- schema de heurística;
- revisão da regra;
- versão do algoritmo de matching/scoring.

Projetos/análises persistidas devem conseguir informar quais revisões influenciaram uma recomendação.

## 18. Testes obrigatórios

### Schema

- range invertido;
- valor negativo quando fisicamente inválido;
- evidence inexistente;
- heurística sem ID/revisão;
- campos desconhecidos.

### Matching

- match exato;
- limites min/max;
- partial match;
- missing context;
- not applicable;
- heurística desativada;
- KV sem tensão não aprovando candidato.

### Candidate Generator

- hard constraint elimina candidato;
- `danger` elimina candidato;
- heurística não consegue reabilitar `danger`;
- múltiplas heurísticas são explicadas;
- dados faltantes reduzem elegibilidade/confiança;
- ordem determinística para mesma entrada.

### Mini Long Range

Fixture em que o candidato mais potente perde para outro mais eficiente, desde que ambos cumpram constraints técnicas.

## 19. Segurança e comunicação

A UI deve usar linguagem como:

```text
"Compatível segundo os dados disponíveis"
"Faixa sugerida por referência técnica"
"Dados insuficientes para validar"
```

Evitar:

```text
"100% seguro"
"motor garantidamente ideal"
"compatibilidade comprovada"
```

sem evidência adequada.

## 20. Critério de aceite arquitetural

- heurísticas são dados versionados, não `if/switch` espalhados;
- evidências são rastreáveis;
- Candidate Generator é independente de React;
- validators físicos continuam autoridade sobre compatibilidade;
- ausência de condição não produz aprovação;
- ranking possui explicação e evidência;
- determinismo coberto por testes;
- Mini Long Range possui fixture de referência.
