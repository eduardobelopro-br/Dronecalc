# Ingestão Assistida de Componentes — URL, Imagens e IA

**Versão:** 0.1  
**Status:** especificação para implementação futura  
**Escopo:** cadastro assistido de equipamentos a partir de páginas, imagens e documentos técnicos.

## 1. Objetivo

O DroneCalc deve permitir que o usuário informe a URL da página de um equipamento e receba um cadastro técnico pré-preenchido. O sistema deve extrair dados não apenas do HTML/texto, mas também das imagens técnicas encontradas na página, incluindo fichas de especificação, tabelas, desenhos dimensionais, etiquetas e gráficos.

A IA pode extrair e estruturar dados, porém não é autoridade técnica. Todo dado remoto é não confiável até validação e todo dado extraído por IA entra primeiro em **staging** no PostgreSQL. Publicação no catálogo exige o workflow de revisão definido neste documento.

## 2. Princípios obrigatórios

1. HTML, JSON-LD, tabelas, imagens e documentos são fontes de evidência distintas.
2. Extração determinística tem prioridade quando a informação já está estruturada.
3. IA multimodal é usada para conteúdo visual e como fallback/assistência, não para substituir validação de schema.
4. Cada campo extraído deve ser rastreável até sua evidência.
5. A imagem/documento original deve ser preservado quando legal e tecnicamente permitido, com metadados no PostgreSQL e arquivo no object storage.
6. Valores conflitantes entre fontes não são reconciliados silenciosamente.
7. Valor literalmente presente na fonte e valor calculado pelo DroneCalc são entidades semânticas diferentes.
8. A IA não grava diretamente nas tabelas publicadas/autoritativas do catálogo.
9. Ausência de dado permanece ausência de dado; a IA não inventa defaults.
10. Gráficos digitalizados por visão computacional/IA não recebem automaticamente status de medição de alta confiança.

## 3. Pipeline canônico

```text
CADASTRAR EQUIPAMENTO
        │
        ▼
       URL
        │
        ▼
SOURCE INGESTION SEGURO
        │
        ├───────────────┬────────────────┐
        ▼               ▼                ▼
      HTML            IMAGENS         DOCUMENTOS
        │               │                │
        ▼               ▼                ▼
 parser/JSON-LD    Vision Provider    parser/IA
        │               │                │
        └───────────────┬────────────────┘
                        ▼
              EXTRACTION CANDIDATES
                        │
                        ▼
                 NORMALIZAÇÃO
                        │
                        ▼
                 SCHEMA VALIDATION
                        │
                        ▼
                CROSS-SOURCE CHECK
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
           válido    conflito    inválido
             │          │
             └────┬─────┘
                  ▼
          POSTGRESQL STAGING
                  │
                  ▼
                REVIEW
                  │
            ┌─────┴─────┐
            ▼           ▼
         APPROVED    REJECTED
            │
            ▼
          CATALOG
```

## 4. Source ingestion

A ingestão por URL segue os controles de segurança do roadmap:

- HTTP/HTTPS apenas;
- bloqueio de loopback, link-local, redes privadas e destinos internos;
- revalidação de destino após redirects e resolução DNS quando necessário;
- limites de redirects, tempo, tamanho e concorrência;
- allowlist de MIME/tipos suportados;
- nenhuma execução arbitrária de JavaScript remoto no backend;
- não reutilizar cookies/tokens do usuário para scraping;
- logs sem segredos;
- sanitização antes de renderizar conteúdo remoto.

Conteúdo remoto deve ser tratado como input hostil, inclusive texto que tente instruir o agente de IA.

## 5. Extração determinística

Antes de IA, tentar obter dados de:

1. JSON-LD/schema.org;
2. dados estruturados do fabricante;
3. meta tags;
4. tabelas HTML;
5. blocos de especificação;
6. texto normalizado.

Cada extractor retorna candidatos, não escreve diretamente no catálogo.

## 6. Descoberta e armazenamento de imagens

O sistema deve identificar imagens potencialmente úteis para o componente, diferenciando quando possível:

- foto do produto;
- ficha técnica;
- tabela de especificações;
- desenho dimensional;
- diagrama elétrico/pinout;
- gráfico/curva de bancada;
- etiqueta/modelo;
- imagem decorativa/irrelevante.

Antes de armazenar:

- validar URL/destino com as mesmas proteções anti-SSRF;
- validar MIME real, não apenas extensão;
- impor tamanho máximo e dimensões máximas;
- calcular SHA-256 para integridade/deduplicação;
- rejeitar formatos não suportados;
- limitar quantidade de assets por import job.

Arquivos ficam em MinIO/S3-compatible. PostgreSQL guarda metadados e `storageKey`, não blobs grandes.

## 7. Extração multimodal por IA

A abstração de IA deve evoluir para suportar texto e visão sem acoplar o domínio ao Ollama.

Contrato conceitual, não API obrigatória:

```ts
interface AiProvider {
  extractStructured<T>(
    request: StructuredExtractionRequest<T>
  ): Promise<AiExtractionResult<T>>

  extractStructuredFromImage?<T>(
    request: ImageExtractionRequest<T>
  ): Promise<AiExtractionResult<T>>
}
```

A IA implementadora pode propor contrato unificado multimodal se for mais simples e robusto. A decisão deve preservar substituibilidade do provider, validação de schema, timeout/cancelamento e testabilidade.

A implementação inicial pode usar Ollama quando o modelo configurado possuir capacidade visual adequada. O sistema deve detectar capability incompatível e falhar de forma explícita, nunca fingir que analisou uma imagem.

## 8. O que a IA deve extrair de imagens

Conforme a categoria do componente, a IA deve tentar identificar apenas campos previstos no schema correspondente.

Para motores, exemplos:

- fabricante/modelo;
- KV;
- configuração de estator/ímãs quando declarada, como `12N14P`;
- diâmetro e comprimento do estator;
- diâmetro/tipo de eixo;
- dimensões externas;
- massa e condição da massa, por exemplo “com 3 cm de fio”;
- corrente sem carga e tensão do ensaio;
- faixa de células/química;
- resistência do enrolamento/interna do motor, preservando o significado da fonte;
- limites de corrente/potência e respectivas condições;
- eficiência declarada e faixa operacional;
- montagem e dimensões presentes em desenhos técnicos.

Para outras categorias, o extractor usa o schema específico. Não transformar texto ambíguo em campo técnico sem registrar ambiguidade.

## 9. Exemplo de extração — motor

Uma ficha visual pode produzir um candidato conceitual:

```ts
const candidate = {
  category: 'motor',
  manufacturer: 'Wasp',
  model: 'Major 22.6-6.5 1860KV',
  specifications: {
    kvRpmPerVolt: 1860,
    stator: { diameterMm: 22.6, lengthMm: 6.5 },
    motorConfiguration: '12N14P',
    shaft: { diameterMm: 4, hollow: true },
    dimensions: { diameterMm: 28.49, lengthMm: 30.6 },
    massG: 30.5,
    electrical: {
      noLoadCurrentA: 1.1,
      noLoadCurrentTestVoltageV: 10,
      windingResistanceOhm: 0.064
    },
    batteryCompatibility: {
      minSeriesCells: 5,
      maxSeriesCells: 6,
      chemistry: 'LiPo'
    }
  }
}
```

Esse objeto é apenas candidato de staging. O schema final pode ser normalizado de forma diferente.

## 10. Condições fazem parte do dado

Limites não devem ser achatados em campos universais quando a fonte declara condição.

Exemplo conceitual:

```ts
interface OperatingLimitCandidate {
  metric: 'current' | 'power' | 'rpm' | 'temperature'
  value: number
  unit: string
  conditions?: {
    seriesCells?: number
    voltageV?: number
    durationS?: number
    propellerId?: string
    notes?: string[]
  }
}
```

Assim, “35 A @ 5S” não vira silenciosamente “motor suporta 35 A em qualquer tensão”.

## 11. Evidência por campo

Cada candidato deve poder apontar para a evidência específica que o originou.

Contrato conceitual:

```ts
interface ExtractedFieldCandidate<T = unknown> {
  fieldPath: string
  rawValue?: unknown
  normalizedValue?: T
  unit?: string

  evidence: {
    sourceId: string
    assetId?: string
    sourceKind: 'html' | 'json-ld' | 'table' | 'image' | 'document' | 'user'
    selectorOrReference?: string
    region?: {
      x: number
      y: number
      width: number
      height: number
      coordinateSpace: 'normalized' | 'pixel'
    }
  }

  extraction: {
    method: 'deterministic' | 'ai-text' | 'ai-vision' | 'user'
    confidence?: number
    provider?: string
    model?: string
    promptVersion?: string
    extractedAt: string
  }

  reviewStatus: 'extracted' | 'needs_review' | 'approved' | 'rejected'
}
```

A região da imagem é opcional e só deve ser armazenada quando o provider/implementação conseguir fornecê-la de maneira defensável.

## 12. Normalização e unidades

O pipeline deve preservar dois níveis:

```text
valor bruto da fonte
        ↓
parser/normalizador
        ↓
valor canônico + unidade conhecida
```

Exemplo:

```text
"64mΩ"
→ 64 mΩ
→ 0.064 Ω
```

O valor bruto continua disponível para auditoria.

Normalização deve usar os contratos de unidades do domínio e não depender da IA para aritmética simples ou conversão determinística.

## 13. Cross-source reconciliation

Se HTML, imagem ou documento fornecerem valores diferentes para o mesmo campo, o sistema não escolhe silenciosamente um vencedor.

Exemplo:

```text
INTERNAL_RESISTANCE
HTML: 60 mΩ
Imagem: 64 mΩ
→ conflict / needs_review
```

A reconciliação deve considerar:

- equivalência de unidades;
- variante/modelo/revisão;
- condição de ensaio;
- data da fonte;
- autoridade da fonte;
- confiança da extração;
- possibilidade de os valores representarem grandezas diferentes.

Valores aparentemente contraditórios podem ser válidos sob condições distintas; portanto o sistema deve preservar contexto antes de classificar como erro.

## 14. Validações cruzadas de consistência

Além de schema, o importador pode gerar warnings determinísticos sem alterar os valores originais.

Exemplos:

- potência, tensão e corrente declaradas que não correspondem ao mesmo ponto por `P = V × I`;
- faixa de células incompatível com tensão explicitamente declarada;
- massa negativa ou dimensão impossível;
- unidade ausente/ambígua;
- limite declarado sem condição suficiente;
- duplicata provável de componente/variante existente.

Esses checks são evidência para revisão, não autorização para “corrigir” automaticamente a fonte.

## 15. Dado extraído versus dado derivado

Regra obrigatória:

```text
EXTRAÍDO
"KV = 1860" presente na ficha
source = manufacturer/image
kind = extracted
```

não é o mesmo que:

```text
DERIVADO
Kt calculado a partir de KV
source = calculated
formulaId = MOTOR_KV_TO_KT_V1
inputs = [kv]
```

Valores derivados devem ser produzidos pelo calculation/physics engine, com `formulaId`, versão, inputs e proveniência. O importador não deve materializar cálculos como se fossem especificações do fabricante.

## 16. Gráficos e curvas de bancada em imagens

A IA pode identificar que uma imagem contém gráfico e sugerir metadados, e futuramente pode digitalizar pontos. Porém:

- pontos lidos visualmente devem ser marcados como `digitized-from-image` ou origem equivalente;
- não devem receber automaticamente `source = measured` de alta confiança;
- eixos, unidades, escalas e legenda precisam ser identificados;
- escala logarítmica deve ser detectada quando aplicável;
- pontos fora da área do gráfico são inválidos;
- digitalização deve guardar asset, região e versão do método;
- sempre que existir CSV/tabela original, ela tem preferência sobre digitalização visual;
- validação humana é obrigatória antes de uma curva digitalizada alimentar resultados de alta confiança.

## 17. Staging e publicação

Estados mínimos:

```text
extracted
→ needs_review
→ approved
→ published
   ou rejected
```

Aprovação pode ocorrer por campo e/ou por revisão do componente, conforme o modelo final.

O catálogo publicado deve referenciar a revisão aprovada e suas evidências. Reprocessar uma fonte com outro modelo de IA não sobrescreve silenciosamente uma revisão já publicada.

## 18. Modelo de persistência conceitual

O schema final será definido por migrations, mas deve suportar semanticamente:

```text
import_jobs
source_snapshots / evidence_sources
stored_objects
extraction_runs
extracted_fields
field_evidence
source_conflicts
review_decisions
component_revisions
```

Não é obrigatório usar esses nomes literalmente.

PostgreSQL armazena estrutura, estado, proveniência e relações. MinIO/S3-compatible armazena imagens/documentos.

## 19. Segurança específica de IA

Conteúdo de página, imagem, alt-text, PDF ou documento é **dados**, não instrução confiável para o agente.

O pipeline deve mitigar prompt injection indireta:

- prompt de sistema fixa o objetivo de extração;
- conteúdo remoto é delimitado como input não confiável;
- o modelo não recebe credenciais nem acesso direto ao banco;
- o modelo não executa ferramentas arbitrárias;
- saída é validada contra schema estrito;
- campos desconhecidos são rejeitados/ignorados de forma explícita;
- nenhuma instrução encontrada na página pode mudar permissões, destino de rede ou política de publicação.

## 20. Privacidade, direitos e retenção

A implementação deve permitir política configurável para retenção de snapshots/assets e respeitar restrições aplicáveis da fonte. O sistema não deve presumir que toda imagem pública pode ser redistribuída indefinidamente.

Quando não for apropriado preservar o arquivo, ainda pode ser possível guardar URL, hash/metadados e evidência textual permitida, conforme a política definida para a fonte.

## 21. Observabilidade

Cada import job deve registrar sem segredos:

- início/fim/duração;
- URL normalizada;
- status de fetch;
- número/tamanho de assets aceitos/rejeitados;
- extractors executados;
- provider/modelo de IA;
- warnings;
- quantidade de candidatos;
- conflitos;
- resultado da validação;
- status de revisão.

Falhas parciais não devem transformar dados incompletos em sucesso silencioso.

## 22. Critérios de aceite

A funcionalidade de ingestão multimodal só pode ser considerada concluída quando:

- uma URL válida pode iniciar um import job sem acesso a rede interna;
- HTML estruturado é extraído sem IA quando possível;
- imagens técnicas candidatas podem ser armazenadas com hash e proveniência;
- um provider visual compatível consegue produzir candidatos estruturados validados por schema;
- modelo sem capacidade visual falha explicitamente;
- cada campo de IA possui evidência e método de extração;
- valor bruto e normalizado permanecem auditáveis;
- conflitos HTML × imagem/documento são apresentados para revisão;
- dados de IA ficam em staging;
- nenhuma aprovação/publicação ocorre apenas porque a IA atribuiu alta confiança;
- valores derivados não são confundidos com valores declarados;
- imagem/documento não é armazenado como blob grande no PostgreSQL;
- falha de IA ou de um asset não corrompe o catálogo;
- testes cobrem SSRF, MIME/tamanho, schema inválido, conflito de fonte e fluxo de revisão.

## 23. Relação com o roadmap

Este documento detalha as **Fases 5 e 6** do `ROADMAP.md`.

A implementação deve ocorrer incrementalmente:

1. source ingestion seguro;
2. extração determinística;
3. descoberta/armazenamento seguro de assets;
4. contrato de AI Provider;
5. provider Ollama;
6. capability multimodal/vision;
7. extração de imagem para schema;
8. normalização e evidência por campo;
9. reconciliação cross-source;
10. review/publish.

Não é necessário implementar visão na primeira etapa de scraping. Segurança de fetch, staging e contratos devem existir antes da automação multimodal.

## 24. Regra para melhorias propostas por IA

A IA implementadora pode modificar contratos, estrutura de tabelas, estratégia de extração, provider ou ordem interna se encontrar abordagem mais eficaz, eficiente, segura ou testável, desde que:

1. preserve staging e revisão antes de publicação;
2. preserve evidência por campo e separação entre extraído/derivado;
3. não enfraqueça controles anti-SSRF/prompt injection;
4. mantenha domínio desacoplado do Ollama;
5. valide saída por schema;
6. explique trade-offs;
7. adicione testes equivalentes ou melhores;
8. atualize documentação/ADR quando a decisão for estrutural.
