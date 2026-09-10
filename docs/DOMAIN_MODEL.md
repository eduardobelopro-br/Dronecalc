# Modelo de Domínio — DroneCalc

**Versão:** 0.1

## 1. Objetivo

Definir entidades e contratos sem dependência de UI ou persistência específica. Os exemplos TypeScript são orientativos; a implementação pode melhorar nomes ou granularidade desde que preserve a semântica.

## 2. Identificadores e metadados

```ts
type Id = string

type DataSourceKind =
  | 'measured'
  | 'manufacturer'
  | 'calculated'
  | 'interpolated'
  | 'estimated'
  | 'user-provided'

type Confidence = 'high' | 'medium' | 'low'

interface SourceMetadata {
  kind: DataSourceKind
  sourceName?: string
  sourceUrl?: string
  reference?: string
  measuredAt?: string
  notes?: string[]
}
```

## 3. Componente base

```ts
interface BaseComponent {
  id: Id
  type: ComponentType
  manufacturer?: string
  model: string
  displayName?: string
  massG?: number
  tags?: string[]
  source?: SourceMetadata
  custom: boolean
  createdAt: string
  updatedAt: string
}
```

`massG` ausente é diferente de `0`.

## 4. Tipos de componente

```ts
type ComponentType =
  | 'frame'
  | 'motor'
  | 'propeller'
  | 'esc'
  | 'battery'
  | 'flight-controller'
  | 'receiver'
  | 'gps'
  | 'vtx'
  | 'fpv-camera'
  | 'recording-camera'
  | 'power-module'
  | 'bec'
  | 'antenna'
  | 'wiring'
  | 'landing-gear'
  | 'prop-guard'
  | 'payload'
  | 'other'
```

## 5. Motor

```ts
interface Motor extends BaseComponent {
  type: 'motor'
  kv: number
  minCellCount?: number
  maxCellCount?: number
  minVoltageV?: number
  maxVoltageV?: number
  maxContinuousCurrentA?: number
  maxBurstCurrentA?: number
  maxPowerW?: number
  stator?: {
    widthMm: number
    heightMm: number
  }
  shaftDiameterMm?: number
  mountingPatternMm?: string
  benchTestIds: Id[]
}
```

KV é propriedade do motor e não deve ser usado sozinho para estimar empuxo.

## 6. Hélice

```ts
interface Propeller extends BaseComponent {
  type: 'propeller'
  diameterIn: number
  pitchIn?: number
  blades: number
  material?: string
  direction?: 'cw' | 'ccw' | 'bidirectional'
  maxRpm?: number
  hubDiameterMm?: number
  shaftHoleMm?: number
}
```

## 7. ESC

```ts
interface Esc extends BaseComponent {
  type: 'esc'
  layout: 'individual' | '4in1' | 'other'
  continuousCurrentA: number
  burstCurrentA?: number
  minCellCount?: number
  maxCellCount?: number
  minVoltageV?: number
  maxVoltageV?: number
  protocol?: string[]
  motorOutputs?: number
}
```

Para ESC 4-em-1, corrente por canal e restrições térmicas devem ser modeladas de forma inequívoca.

## 8. Bateria

```ts
type BatteryChemistry = 'lipo' | 'lihv' | 'liion' | 'other'

interface Battery extends BaseComponent {
  type: 'battery'
  chemistry: BatteryChemistry
  seriesCells: number
  parallelCells?: number
  capacityMah: number
  nominalCellVoltageV: number
  fullCellVoltageV: number
  minimumCellVoltageV?: number
  continuousCRating?: number
  burstCRating?: number
  measuredContinuousCurrentA?: number
  connector?: string
  internalResistanceMilliOhm?: number
}
```

Não assumir uma tensão por célula global para todas as químicas. O objeto da bateria fornece as tensões utilizadas nos cálculos.

## 9. Frame

```ts
interface Frame extends BaseComponent {
  type: 'frame'
  motorCountSupported?: number[]
  wheelbaseMm?: number
  maxPropDiameterIn?: number
  motorMountPatternMm?: string[]
  stackMountPatternsMm?: string[]
  recommendedBatteryMount?: string
}
```

## 10. Eletrônica genérica

```ts
interface ElectricalLoad extends BaseComponent {
  minInputVoltageV?: number
  maxInputVoltageV?: number
  nominalCurrentA?: number
  maxCurrentA?: number
  nominalPowerW?: number
}
```

Flight controller, GPS, receptor, VTX e câmeras podem estender esse contrato com campos próprios.

## 11. Payload

```ts
interface Payload extends BaseComponent {
  type: 'payload'
  powerRequired?: boolean
  minInputVoltageV?: number
  maxInputVoltageV?: number
  nominalPowerW?: number
  position?: Position3D
}
```

## 12. Posição e centro de gravidade

```ts
interface Position3D {
  xMm: number
  yMm: number
  zMm: number
}
```

Convenção obrigatória a definir na implementação:

- origem no centro geométrico de referência do frame;
- eixos e sinais documentados e desenhados na UI;
- posição opcional até o módulo de CG ser usado.

## 13. Instância de componente no projeto

O catálogo descreve um modelo de peça; o projeto descreve uma instância/uso.

```ts
interface ProjectComponent {
  id: Id
  componentId: Id
  quantity: number
  overrideMassG?: number
  position?: Position3D
  notes?: string
}
```

Overrides devem ser explícitos e rastreáveis.

## 14. Projeto de drone

```ts
interface DroneProject {
  schemaVersion: number
  id: Id
  name: string
  description?: string
  motorCount: number
  flightProfileId?: Id
  constraints: ProjectConstraints
  components: ProjectComponent[]
  assumptions: ProjectAssumptions
  createdAt: string
  updatedAt: string
}
```

## 15. Restrições de projeto

```ts
interface ProjectConstraints {
  maxTakeoffMassG?: number
  sub250g?: boolean
  minEnduranceMinutes?: number
  maxPropDiameterIn?: number
  maxSeriesCells?: number
  payloadTargetG?: number
  preferredBatteryChemistry?: BatteryChemistry[]
}
```

`sub250g` é uma intenção/restrição separada do perfil de voo.

## 16. Hipóteses

```ts
interface ProjectAssumptions {
  usableBatteryFraction?: number
  ambientTemperatureC?: number
  reserveFraction?: number
  notes?: string[]
}
```

Valores padrão, quando existirem, devem ser visíveis e versionados.

## 17. Dados de bancada

```ts
interface MotorPropBenchTest {
  id: Id
  motorId: Id
  propellerId?: Id
  propellerDescription: string
  batteryChemistry?: BatteryChemistry
  seriesCells?: number
  testVoltageV?: number
  source: SourceMetadata
  samples: BenchSample[]
  notes?: string[]
}

interface BenchSample {
  throttlePercent?: number
  thrustG: number
  currentA: number
  voltageV: number
  powerW?: number
  rpm?: number
}
```

`powerW` pode ser calculado de `voltageV * currentA`, mas a origem deve distinguir dado fornecido de derivado.

## 18. Perfil de voo

```ts
interface FlightStyleProfile {
  schemaVersion: number
  id: Id
  slug: string
  name: string
  description: string
  priorities: ProfilePriorities
  targets: ProfileTargets
  scoring: ScoringRule[]
}

interface ProfilePriorities {
  endurance: number
  efficiency: number
  agility: number
  payload: number
  stability: number
  compactness: number
  lowWeight: number
  powerReserve: number
}
```

Pesos devem ser normalizados pelo motor de scoring antes da comparação.

## 19. Resultado de análise

```ts
interface DroneAnalysis {
  analysisVersion: string
  projectId: Id
  generatedAt: string
  mass: MassAnalysis
  battery?: BatteryAnalysis
  propulsion?: PropulsionAnalysis
  electrical?: ElectricalAnalysis
  endurance?: EnduranceAnalysis
  centerOfGravity?: CgAnalysis
  profileScore?: ProfileScore
  warnings: AnalysisWarning[]
}
```

## 20. Resultado calculado

```ts
interface CalculationResult<T> {
  value: T
  unit: string
  source: 'measured' | 'manufacturer' | 'calculated' | 'interpolated' | 'estimated'
  confidence: Confidence
  modelVersion: string
  notes?: string[]
}
```

Para resultados derivados de múltiplas fontes, o domínio pode evoluir para `sources[]`.

## 21. Disponibilidade

```ts
type MetricAvailability<T> =
  | { status: 'available'; result: CalculationResult<T> }
  | { status: 'missing-data'; missing: string[] }
  | { status: 'not-applicable'; reason: string }
  | { status: 'invalid'; errors: string[] }
```

## 22. Warnings

```ts
type WarningSeverity = 'success' | 'info' | 'warning' | 'danger'

interface AnalysisWarning {
  code: string
  severity: WarningSeverity
  title: string
  message: string
  componentIds?: Id[]
  actual?: number
  limit?: number
  unit?: string
  recommendation?: string
}
```

Códigos são parte do contrato e não devem mudar sem migração/testes.

## 23. Regras gerais

- ausência não equivale a zero;
- unidade deve ser conhecida em todo número físico;
- overrides precisam ser rastreáveis;
- valores calculados não sobrescrevem a fonte original;
- dados de catálogo não devem ser mutados ao editar um projeto;
- schemas externos são validados antes de entrar no domínio;
- alterações incompatíveis exigem aumento de `schemaVersion`.
