# Modelo de Dados e Persistência — DroneCalc

**Versão:** 0.2

## 1. Objetivo

Definir como projetos, componentes, evidências, imports, perfis e testes de bancada são persistidos, importados, exportados e migrados sem acoplar o domínio a uma tecnologia concreta.

## 2. Estratégia vigente

O MVP ampliado adota:

- PostgreSQL como fonte principal de dados persistidos;
- repositories/adapters entre application/domain e banco;
- MinIO/S3-compatible para imagens e documentos;
- IndexedDB apenas para cache, drafts, preferências e suporte offline auxiliar;
- JSON versionado para export/import;
- CSV para importação de tabelas de bancada;
- staging obrigatório para dados extraídos por URL/IA.

Essa decisão substitui a estratégia inicial de IndexedDB como armazenamento principal e está registrada no ADR 0002.

## 3. Agregados/áreas lógicas

O modelo físico será detalhado nas migrations, mas deve cobrir ao menos:

- `projects`;
- `project_components`;
- `components`;
- `manufacturers`;
- `component_variants` quando necessário;
- `bench_tests`;
- `bench_test_samples`;
- `flight_profiles`/versões quando persistidos;
- `evidence_sources`;
- `imports`;
- `extracted_fields`/staging;
- `stored_objects` ou metadados equivalentes de mídia;
- `heuristics` e revisões futuras;
- metadata de migrations/versionamento.

Não é obrigatório usar esses nomes literalmente. O schema deve ser normalizado na medida em que consultas, integridade e manutenção se beneficiem disso.

## 4. Projeto persistido

Exemplo conceitual de export, não schema SQL literal:

```json
{
  "schemaVersion": 1,
  "id": "project_x",
  "name": "Mini LR 4in",
  "motorCount": 4,
  "flightProfileId": "mini-long-range",
  "constraints": {
    "sub250g": true
  },
  "components": [],
  "assumptions": {
    "usableBatteryFraction": 0.8
  },
  "createdAt": "...",
  "updatedAt": "..."
}
```

## 5. Catálogo versus instância no projeto

Catálogo e projeto são conceitos diferentes.

- catálogo descreve um componente/revisão de referência;
- projeto referencia esse componente;
- overrides de massa, quantidade, posição ou parâmetros pertencem à instância do projeto;
- alterar override não modifica silenciosamente o catálogo.

Para reprodutibilidade, projeto/análise pode guardar `componentRevision` ou snapshot dos campos determinantes.

## 6. PostgreSQL

### 6.1 Princípios

- chaves primárias estáveis;
- foreign keys quando representarem integridade real;
- constraints para invariantes simples;
- tipos adequados para campos usados em filtros/cálculo;
- JSONB apenas para metadados flexíveis;
- migrations versionadas;
- transações para operações multi-registro que precisem atomicidade;
- índices baseados em consultas reais, não adicionados preventivamente sem evidência.

### 6.2 IDs

Usar UUID, ULID ou solução equivalente estável. A escolha final deve ser registrada na etapa de persistência. Não usar nome/modelo como chave primária.

### 6.3 Datas

Persistir timestamps em UTC. Localização é responsabilidade da apresentação.

## 7. Object Storage

Imagens e documentos grandes ficam fora do PostgreSQL por padrão.

Metadados persistidos podem incluir:

```ts
interface StoredObjectMetadata {
  id: string
  sourceId?: string
  storageKey: string
  originalUrl?: string
  mimeType: string
  sizeBytes: number
  sha256?: string
  width?: number
  height?: number
  createdAt: string
}
```

MinIO é a implementação local inicial. O contrato deve permitir storage S3-compatible futuro.

## 8. Fontes e evidências

Toda informação técnica importada deve poder indicar origem.

Exemplo conceitual:

```ts
interface EvidenceSource {
  id: string
  kind: 'manufacturer' | 'measured' | 'user-provided' | 'article' | 'video' | 'other'
  url?: string
  title?: string
  capturedAt?: string
  notes?: string[]
}
```

Uma fonte não é automaticamente confiável apenas por existir. Confiabilidade e completude são metadados distintos.

## 9. Ingestão por URL e staging

Dados extraídos nunca devem ser escritos diretamente como catálogo publicado.

Fluxo:

```text
source
→ import job
→ evidence snapshot/metadata
→ extracted fields
→ validation
→ needs_review
→ approved/rejected
→ published
```

Um campo extraído deve poder guardar:

- valor bruto;
- valor normalizado;
- unidade;
- método de extração;
- evidência/origem;
- confiança;
- estado de revisão.

## 10. IA/Ollama

Resultados de IA são dados derivados, não autoridade.

Persistir quando útil:

- provider;
- model;
- model/version/tag identificável;
- prompt/template version;
- timestamp;
- schema esperado;
- resultado validado;
- warnings/erros;
- ligação à fonte.

Não é necessário armazenar raciocínio interno do modelo.

## 11. Dados de bancada

Curvas são entidades independentes.

Chave lógica inclui:

- motor/revisão;
- hélice/revisão ou descrição estruturada;
- tensão/células/condições;
- fonte;
- revisão.

A mesma combinação pode possuir múltiplos testes de fontes diferentes. Não mesclar automaticamente.

Samples típicos:

```text
throttle_percent
thrust
gcurrent
voltage
power
rpm
```

O schema real deve usar nomes/unidades inequívocos e constraints físicas adequadas.

## 12. Perfis e heurísticas

Perfis e heurísticas devem ser versionáveis.

Projetos/análises devem registrar a versão usada ou snapshot dos objetivos derivados quando necessário para reprodutibilidade.

Heurística não deve compartilhar o mesmo status semântico de dado medido/compatibilidade validada.

## 13. Versionamento de schema

Todo objeto exportável relevante possui `schemaVersion`.

Regras:

- rename/removal/mudança semântica exige migration;
- migration deve ser testada;
- banco novo deve chegar ao schema atual do zero;
- upgrade suportado deve ser reproduzível;
- não destruir dados antes de validação adequada.

## 14. Versão do motor

Análises exportadas devem registrar `calculationEngineVersion`/formula IDs relevantes.

## 15. Exportação JSON

Formato conceitual:

```ts
interface DroneCalcExport {
  format: 'dronecalc-project'
  schemaVersion: number
  exportedAt: string
  appVersion?: string
  calculationEngineVersion?: string
  project: DroneProject
  referencedComponents: BaseComponent[]
  referencedBenchTests: MotorPropBenchTest[]
  referencedFlightProfile?: FlightStyleProfile
}
```

O export de projeto deve ser autocontido sempre que razoável.

## 16. Importação JSON

```text
arquivo
→ parse seguro
→ validação de formato
→ migration por schemaVersion
→ validação semântica
→ preview/conflitos
→ transação de persistência
```

Nunca persistir parcialmente antes das validações necessárias.

## 17. Importação CSV de bancada

Campos-alvo típicos:

- throttlePercent;
- thrust;
- current;
- voltage;
- power;
- rpm.

O importador deve solicitar/detectar unidades e converter explicitamente. Não assumir que `thrust` está em gramas.

## 18. Conflitos/revisões

Se um componente com mesmo identificador lógico apresentar dados diferentes:

- comparar revisão/hash/fonte;
- não sobrescrever silenciosamente;
- criar nova revisão/candidato ou pedir decisão;
- manter histórico quando o dado anterior já fundamentou projeto/análise.

## 19. Deleção

Preferir archive/soft-delete para catálogo referenciado.

Não deixar projetos existentes com referência destruída silenciosamente.

## 20. Integridade

Validar ao menos:

- referências;
- IDs únicos;
- valores físicos permitidos;
- versões suportadas;
- curvas válidas;
- status de staging/publicação;
- metadados de object storage coerentes.

## 21. IndexedDB/local storage

IndexedDB pode guardar:

- draft atual;
- preferências e sessão local não sensível;
- cache do catálogo;
- último projeto aberto;
- futura fila offline.

`localStorage` deve ser limitado a preferências pequenas quando apropriado.

Nenhum dos dois deve conter secrets da API ou substituir a fonte autoritativa PostgreSQL quando online.

## 22. Privacidade e segurança

- não guardar cookies/tokens capturados de páginas externas;
- não expor credenciais PostgreSQL/MinIO no frontend;
- dados importados por URL são não confiáveis até validação;
- sanitizar conteúdo remoto antes de exibição;
- imports com falha não deixam estado parcial inconsistente.

## 23. Backup futuro

Planejar separadamente:

- backup PostgreSQL;
- backup/versionamento do object storage;
- export manual de projetos;
- retenção de snapshots/evidências;
- recuperação testada.

O usuário deve continuar podendo manter export local legível/versionado mesmo com backend.

## 24. Decisão de implementação

Detalhes como ORM/query builder, layout exato de tabelas, UUID versus ULID e estratégia de migration serão escolhidos na Etapa 2A com base em código real, documentação oficial e requisitos de consulta. Qualquer escolha estrutural relevante deve gerar ADR.
