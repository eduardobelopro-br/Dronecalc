# Arquitetura — DroneCalc

**Versão:** 0.1  
**Status:** arquitetura proposta para o MVP

## 1. Objetivos arquiteturais

A arquitetura deve priorizar:

- precisão e testabilidade do motor matemático;
- independência entre domínio e interface;
- evolução incremental sem backend obrigatório no MVP;
- dados versionados e validados;
- possibilidade futura de reutilizar o cálculo em Web, desktop, CLI ou API;
- rastreabilidade da origem dos resultados;
- baixo acoplamento entre perfis de voo, catálogo e UI.

## 2. Stack-base

Proposta inicial:

- **React** para interface;
- **TypeScript** em modo estrito;
- **Vite** como toolchain web;
- **Zod** para validação de entrada, importação e schemas externos;
- **Zustand** para estado de aplicação/projeto;
- **Vitest** para testes unitários;
- **Testing Library** para testes de componentes;
- **IndexedDB** por camada de persistência local no MVP;
- biblioteca de gráficos escolhida apenas quando as primeiras curvas forem implementadas.

A stack pode ser alterada se houver justificativa técnica objetiva e registro da decisão.

## 3. Regra estrutural central

O motor de cálculo não pode importar React, DOM, Zustand ou componentes visuais.

Fluxo desejado:

```text
UI → Application Services → Domain / Calculation Engine → Resultados
                         ↘ Persistence / Catalog
```

Dependências devem apontar para dentro do domínio, nunca do domínio para a UI.

## 4. Estrutura inicial de diretórios

```text
src/
├── app/
│   ├── router/
│   ├── providers/
│   └── bootstrap/
├── design-system/
│   ├── tokens/
│   ├── themes/
│   ├── components/
│   └── layouts/
├── features/
│   ├── projects/
│   ├── builder/
│   ├── analysis/
│   ├── comparison/
│   ├── catalog/
│   └── flight-profiles/
├── domain/
│   ├── common/
│   ├── units/
│   ├── components/
│   ├── project/
│   ├── flight-profile/
│   └── analysis/
├── calculation-engine/
│   ├── mass/
│   ├── battery/
│   ├── electrical/
│   ├── propulsion/
│   ├── endurance/
│   ├── center-of-gravity/
│   ├── compatibility/
│   └── scoring/
├── application/
│   ├── analyze-project/
│   ├── compare-projects/
│   └── import-export/
├── persistence/
│   ├── repositories/
│   ├── indexeddb/
│   └── migrations/
├── catalog/
│   ├── schemas/
│   ├── importers/
│   └── seed/
└── shared/
    ├── errors/
    └── utils/
```

## 5. Camadas

### 5.1 Domain

Contém entidades, value objects, enums e contratos puros.

Não pode depender de:

- UI;
- persistência concreta;
- rede;
- ambiente do navegador.

### 5.2 Calculation Engine

Funções puras sempre que possível. Recebem tipos do domínio e retornam resultados tipados.

Exemplos:

```ts
calculateTotalMass(project)
calculateBatteryEnergy(battery)
calculateThrustToWeight(input)
interpolateBenchPoint(curve, targetThrust)
estimateHover(project, benchData)
validateElectricalCompatibility(project)
```

### 5.3 Application

Orquestra domínio, catálogo e persistência. Decide quais cálculos estão disponíveis para o estado atual do projeto.

### 5.4 Features/UI

Renderiza fluxos e interações. Não deve conter fórmulas de engenharia.

### 5.5 Persistence

Implementa repositórios abstratos. O domínio não conhece IndexedDB.

## 6. Contratos recomendados

### CalculationResult

```ts
interface CalculationResult<T> {
  value: T
  unit: string
  source: 'measured' | 'manufacturer' | 'calculated' | 'interpolated' | 'estimated'
  confidence: 'high' | 'medium' | 'low'
  modelVersion: string
  notes?: string[]
}
```

### AnalysisWarning

```ts
interface AnalysisWarning {
  code: string
  severity: 'success' | 'info' | 'warning' | 'danger'
  title: string
  message: string
  componentIds?: string[]
  actual?: number
  limit?: number
  unit?: string
  recommendation?: string
}
```

### Analyzer

```ts
interface Analyzer<TInput, TOutput> {
  analyze(input: TInput): TOutput
}
```

## 7. Sistema de unidades

Não usar números “soltos” quando houver risco de ambiguidade.

Há duas estratégias aceitáveis:

1. value objects (`Mass`, `Voltage`, `Current` etc.);
2. valores numéricos com unidade canônica rigidamente definida no tipo e documentada.

Recomendação: começar com tipos/constructors leves e funções de conversão centralizadas para evitar complexidade excessiva.

Exemplo:

```ts
type Grams = number & { readonly __brand: 'Grams' }
type Volts = number & { readonly __brand: 'Volts' }
```

Conversão deve ocorrer nas bordas da aplicação.

## 8. Dados incompletos

O motor deve trabalhar com disponibilidade parcial sem lançar erro para ausência esperada.

Preferir resultado discriminado:

```ts
type Availability<T> =
  | { status: 'available'; result: T }
  | { status: 'missing-data'; missing: string[] }
  | { status: 'not-applicable'; reason: string }
  | { status: 'invalid'; errors: string[] }
```

Isso evita transformar falta de informação em zero.

## 9. Interpolação

A interpolação de curvas deve ficar isolada do restante da aplicação. O MVP deve usar interpolação apenas dentro do intervalo observado. Extrapolação é bloqueada por padrão.

O módulo deve ser determinístico e extensivamente testado em:

- limites exatos;
- pontos intermediários;
- curva não monotônica;
- duplicidade de pontos;
- dados ausentes;
- alvo fora do intervalo.

## 10. Perfis de voo

Perfis não devem ser implementados em `switch` espalhados. Devem existir como configuração versionada validada por schema.

O motor de scoring recebe:

- perfil;
- métricas normalizadas;
- restrições;
- penalidades.

E retorna score explicável por dimensão.

## 11. Design System

DroneCalc deve copiar/adaptar a estrutura do NEXO Design System como módulo próprio e rastreável, usando os mesmos nomes de tokens `--nexo-*` e mecanismos de tema.

Não criar cores hardcoded dentro de features.

## 12. Persistência

Definir interfaces antes da implementação concreta:

```ts
interface ProjectRepository {
  list(): Promise<DroneProject[]>
  get(id: string): Promise<DroneProject | null>
  save(project: DroneProject): Promise<void>
  delete(id: string): Promise<void>
}
```

O IndexedDB será detalhe de infraestrutura.

## 13. Versionamento

Versionar:

- schema de projeto;
- schema de catálogo;
- schema de curva de bancada;
- perfis de voo;
- motor de cálculo quando mudanças alterarem resultados.

Exports devem incluir `schemaVersion`.

## 14. Estratégia de backend

Nenhum backend é necessário para o MVP. Isso reduz custo e complexidade enquanto o motor ainda está sendo validado.

Um backend passa a fazer sentido quando houver:

- contas/sincronização;
- catálogo colaborativo;
- telemetria opcional;
- computação pesada de otimização;
- compartilhamento de projetos.

Quando isso ocorrer, o cálculo central deve permanecer reutilizável e testável independentemente da API.

## 15. Desktop futuro

A arquitetura web deve ser compatível com empacotamento futuro via Tauri. Evitar APIs de navegador acopladas ao domínio facilita essa migração.

## 16. Gestão de decisões

Mudanças importantes de arquitetura devem ser documentadas em ADRs futuros (`docs/adr/`). Exemplos:

- escolha de persistência;
- adoção de backend;
- formato de catálogo;
- biblioteca de gráficos;
- mudança de estratégia de unidades;
- motor de otimização.

## 17. Critérios arquiteturais de aceite

- nenhuma fórmula relevante em componente React;
- domínio compilável/testável isoladamente;
- imports sem ciclos entre camadas;
- validação de entrada externa nas bordas;
- resultados com origem/confiança;
- persistência acessada por contrato;
- perfis carregados por dados;
- nenhum token visual hardcoded em features.
