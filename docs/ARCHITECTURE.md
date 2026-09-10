# Arquitetura — DroneCalc

**Versão:** 0.2  
**Status:** arquitetura proposta para o MVP ampliado

## 1. Objetivos arquiteturais

A arquitetura deve priorizar:

- precisão e testabilidade do motor matemático;
- independência entre domínio, cálculo, persistência, UI e IA;
- PostgreSQL como fonte principal dos dados persistidos;
- object storage para imagens/documentos;
- Ollama como integração substituível, não como dependência do domínio;
- dados versionados e validados;
- ingestão por URL segura e auditável;
- possibilidade de reutilizar cálculo em Web, desktop, CLI ou API;
- rastreabilidade de origem, evidência e confiança;
- baixo acoplamento entre perfis, catálogo e interface.

## 2. Stack-base

Proposta inicial:

- **React + TypeScript + Vite** para a interface Web;
- **Node.js + TypeScript** para a API;
- **Zod** para validação de entradas e contratos externos;
- **Zustand** para estado da UI quando necessário;
- **Vitest** para testes unitários;
- **Testing Library** para componentes;
- **PostgreSQL** como banco principal;
- **MinIO** como object storage local compatível com S3;
- **Ollama** como runtime de IA local;
- **IndexedDB** apenas para cache/drafts/preferências/offline controlado;
- biblioteca de acesso ao PostgreSQL escolhida na Etapa 2A mediante avaliação técnica;
- biblioteca de gráficos somente quando as primeiras curvas forem implementadas.

A stack pode ser alterada se houver justificativa objetiva, testes e registro da decisão.

## 3. Regra estrutural central

O motor de cálculo não pode importar React, DOM, Zustand, PostgreSQL, ORM, MinIO ou Ollama.

Fluxo principal:

```text
Web UI
  ↓
Application / API
  ├── Catalog / Projects → PostgreSQL
  ├── Files / Images     → Object Storage
  ├── AI Extraction      → AiProvider → Ollama
  └── Analysis           → Domain + Calculation Engine
```

Dependências apontam para contratos internos; infraestrutura implementa adapters.

## 4. Estrutura inicial recomendada

```text
Dronecalc/
├── apps/
│   ├── web/
│   │   └── src/
│   │       ├── app/
│   │       ├── design-system/
│   │       └── features/
│   └── api/
│       └── src/
│           ├── application/
│           ├── http/
│           ├── persistence/
│           ├── catalog/
│           ├── ingestion/
│           ├── ai/
│           └── object-storage/
├── packages/
│   ├── domain/
│   ├── calculation-engine/
│   └── contracts/
├── infrastructure/
│   ├── docker/
│   └── migrations/
├── docs/
└── package.json
```

A IA implementadora pode propor outra organização se reduzir complexidade sem violar as fronteiras de dependência.

## 5. Camadas

### 5.1 Domain

Entidades, value objects, enums e contratos puros.

Não depende de:

- UI;
- rede;
- banco;
- browser;
- Ollama;
- object storage.

### 5.2 Calculation Engine

Funções puras sempre que possível. Recebe tipos do domínio e devolve resultados tipados.

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

Orquestra casos de uso, repositories, ingestão, IA e cálculo.

### 5.4 Web UI

Renderiza fluxos e interações. Não contém fórmulas de engenharia.

### 5.5 API / Infrastructure

Implementa HTTP, persistência PostgreSQL, object storage, fetch controlado de fontes e adapters de IA.

## 6. Contratos-base

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

### Availability

```ts
type Availability<T> =
  | { status: 'available'; result: T }
  | { status: 'missing-data'; missing: string[] }
  | { status: 'not-applicable'; reason: string }
  | { status: 'invalid'; errors: string[] }
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

## 7. PostgreSQL

PostgreSQL é a fonte principal para:

- catálogo;
- fabricantes/modelos/variantes;
- projetos;
- componentes de projeto;
- fontes/evidências;
- imports e staging;
- curvas de bancada e samples;
- heurísticas;
- revisões;
- metadados de arquivos/imagens;
- configurações de perfis versionadas quando persistidas.

Campos usados em cálculo, filtros e joins devem ser tipados. JSONB é permitido para metadados flexíveis, não como substituto indiscriminado do modelo relacional.

O domínio não conhece SQL/ORM.

## 8. Object Storage

Arquivos grandes não devem ser persistidos em `BYTEA` por padrão.

PostgreSQL mantém metadados:

```text
id
source_id
storage_key
original_url
mime_type
size_bytes
sha256
width/height quando aplicável
created_at
```

O binário fica em MinIO localmente ou outro backend compatível com S3 no futuro.

## 9. Ollama e AI Provider

Ollama é um adapter de IA local.

Contrato conceitual:

```ts
interface AiProvider {
  extractStructured<T>(request: StructuredExtractionRequest<T>): Promise<AiExtractionResult<T>>
}
```

Regras:

- modelo configurável fora do domínio;
- saída sempre validada por schema;
- prompt/model/version registrados quando relevante;
- timeout e cancelamento;
- falha da IA não publica dados;
- IA nunca transforma ausência em valor técnico inventado;
- resultados entram em staging e passam por revisão humana.

## 10. Ingestão por URL

A feature de cadastro de equipamento por URL é executada no backend.

Pipeline:

```text
URL
→ validação de destino
→ fetch limitado
→ snapshot/evidência
→ extração determinística
→ extração assistida por IA opcional
→ schema validation
→ staging
→ revisão
→ publicação
```

### Controles de segurança mínimos

- somente `http` e `https`;
- bloquear loopback, link-local, redes privadas e endpoints internos;
- revalidar destino após redirects;
- limite de redirects;
- timeout;
- limite de bytes;
- MIME/content-type permitido;
- não aceitar `file:`, `ftp:` ou esquemas arbitrários;
- impedir que HTML remoto execute código no backend;
- sanitizar qualquer HTML exibido na UI;
- logs sem tokens/cookies/credenciais;
- rate limiting quando a feature for exposta a múltiplos usuários.

O objetivo é reduzir risco de SSRF, DoS e ingestão de conteúdo malicioso.

## 11. Staging e publicação

Dados extraídos não entram diretamente no catálogo.

Estados sugeridos:

```text
fetched
→ extracted
→ needs_review
→ approved
→ published
```

Estados alternativos:

```text
rejected
failed
```

A revisão deve mostrar valor, unidade, fonte/evidência e confiança.

## 12. Sistema de unidades

Conversões ficam centralizadas. Não usar números soltos quando houver risco de ambiguidade.

Exemplo:

```ts
type Grams = number & { readonly __brand: 'Grams' }
type Volts = number & { readonly __brand: 'Volts' }
```

Conversão ocorre nas bordas da aplicação.

## 13. Interpolação

A interpolação fica isolada no calculation engine.

Regras MVP:

- determinística;
- apenas dentro do intervalo observado;
- extrapolação bloqueada por padrão;
- pontos duplicados/ambíguos tratados explicitamente;
- dados fisicamente inválidos rejeitados;
- origem marcada como `interpolated`.

## 14. Perfis e heurísticas

Perfis de voo são dados versionados, não `switch` espalhado pelo código.

Heurísticas de dimensionamento são conhecimento auxiliar e devem ser separadas da validação física.

```text
heurística → candidato
validação elétrica/mecânica → compatibilidade
bench data → evidência de desempenho
perfil → score de adequação
```

Um candidato recomendado por heurística não recebe automaticamente status de compatível.

## 15. IndexedDB

IndexedDB pode existir no frontend para:

- drafts;
- preferências;
- cache;
- último projeto aberto;
- futura fila offline.

Não é a fonte principal do catálogo quando a API/PostgreSQL estiver disponível.

## 16. Versionamento

Versionar:

- schemas de projeto e catálogo;
- migrations SQL;
- schema de curvas;
- perfis e heurísticas;
- prompts/model metadata quando afetarem extração;
- motor de cálculo quando mudanças alterarem resultado.

Exports devem incluir `schemaVersion`.

## 17. Segurança e segredos

- `.env.example` contém apenas nomes/exemplos seguros;
- `.env` e segredos reais nunca são commitados;
- credenciais de banco/object storage não aparecem no frontend;
- URL ingestion não reutiliza cookies pessoais do usuário;
- respostas de erro não expõem stack trace em produção;
- dependências externas devem ser verificadas na documentação oficial.

## 18. Desktop futuro

A independência do domínio/cálculo mantém viável empacotamento futuro com Tauri. A existência de backend não impede um modo local: PostgreSQL, MinIO e Ollama podem ser serviços locais ou substituídos por adapters adequados no futuro.

## 19. Gestão de decisões

Mudanças estruturais devem gerar ADR. Em especial:

- workspace/package manager;
- framework HTTP;
- ORM/query builder/migration tool;
- política de object storage;
- mudança de provider de IA;
- pgvector;
- backend remoto/contas;
- estratégia offline.

## 20. Critérios arquiteturais de aceite

- nenhuma fórmula relevante em componente React;
- domínio e calculation engine testáveis isoladamente;
- ausência de ciclos indevidos entre camadas;
- entrada externa validada nas bordas;
- PostgreSQL acessado por adapters/repositories;
- arquivos grandes fora do banco por padrão;
- Ollama atrás de interface substituível;
- conteúdo importado por URL passa por controles anti-SSRF;
- dados de IA passam por schema/staging/revisão;
- resultados físicos preservam unidade, origem e confiança;
- nenhum token visual hardcoded em features.

## 21. Relação com ADR 0001

O ADR 0001 continua válido quanto a **web-first e independência do motor de cálculo**, mas sua decisão de “backend não obrigatório/IndexedDB como persistência inicial” foi revista após o escopo crescer para catálogo persistente, ingestão por URL, armazenamento de evidências e IA local. A nova decisão é registrada no ADR 0002.
