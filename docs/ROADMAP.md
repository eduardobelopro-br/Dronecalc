# Roadmap — DroneCalc

**Versão:** 0.6  
**Status:** planejamento técnico revisado  
**Estratégia:** construir uma fundação full-stack pequena e auditável, mantendo o motor de cálculo independente da UI, da persistência e da IA e evoluindo a propulsão por duas rotas: dados de bancada auditáveis e modelo físico validável.

## 1. Princípios de execução

- etapas pequenas, coesas e testáveis;
- nenhuma fórmula de engenharia dentro da UI;
- PostgreSQL será a fonte principal de dados persistidos do produto;
- IndexedDB poderá ser usado apenas para cache, preferências, drafts e suporte offline;
- imagens e documentos não devem ser armazenados como blobs grandes no PostgreSQL; usar object storage compatível com S3;
- Ollama será integrado por uma interface própria, sem dependência do domínio em um modelo específico;
- scraping/fetching de páginas de fabricantes é responsabilidade de um módulo de ingestão controlado; a IA não navega livremente nem decide destinos de rede;
- parsing determinístico de JSON-LD, tabelas e especificações estruturadas tem prioridade sobre IA;
- browser headless é fallback isolado e não dependência obrigatória do scraper inicial;
- dados extraídos por IA entram primeiro em staging e nunca são publicados automaticamente como verdade técnica;
- campos extraídos de imagens/documentos devem manter evidência rastreável e método de extração;
- valores extraídos e valores derivados pelo motor de cálculo são semanticamente distintos;
- conflitos HTML × imagem × documento nunca são resolvidos silenciosamente;
- páginas oficiais/datasheets de fabricantes têm alta prioridade de evidência, mas continuam sujeitos a validação, condições e revisão;
- snapshots sucessivos de uma página não sobrescrevem silenciosamente revisões já aprovadas;
- importação por URL deve ser tratada como superfície de segurança e protegida contra SSRF, redirects abusivos, payloads excessivos, tipos de conteúdo inesperados e prompt injection indireta;
- o scraper não deve ser projetado para burlar CAPTCHA, autenticação, paywall ou controles de acesso;
- preferir dados medidos e documentos de fabricante a heurísticas genéricas;
- heurística orienta candidatos; validação física decide compatibilidade;
- dados de bancada aplicáveis têm prioridade sobre previsões teóricas;
- modelo teórico de propulsão só produz resultado quando houver parâmetros suficientes e sempre informa modelo, hipóteses, proveniência e confiança;
- nunca inferir empuxo real apenas de `KV + diâmetro da hélice + tensão`;
- `gf/W` deve ser tratado como eficiência estática de empuxo, nunca como eficiência percentual;
- `pitch × RPM` produz apenas velocidade teórica de passo e nunca deve ser apresentado como velocidade real/máxima do drone;
- gráficos técnicos não devem misturar grandezas de unidades incompatíveis numa única escala Y por padrão;
- cada etapa atualiza documentação e ADR quando alterar contrato ou arquitetura;
- manter o projeto executável ao final de cada etapa.

---

## Etapa 0 — Especificação e decisões

**Status:** documentação-base criada e em evolução.

Entregas:

- PRD e Product Spec;
- arquitetura e ADRs;
- domínio, unidades e persistência;
- Calculation Engine e Physics Engine;
- perfis de voo;
- NEXO Design System e UX;
- catálogo de componentes;
- estratégia de testes e release gates;
- ingestão assistida por URL/imagens;
- scraping controlado de fabricantes em `docs/MANUFACTURER_SCRAPING.md`;
- workspace de dados de bancada em `docs/BENCH_DATA_WORKSPACE.md`;
- roadmap e prompts de implementação.

Critério: documentos superiores não se contradizem e qualquer IA implementadora consegue identificar a fonte normativa de cada decisão.

---

# FASE 1 — Fundação executável

## Etapa 1A — Bootstrap full-stack e workspace

Criar uma fundação mínima e executável, sem antecipar regras de negócio.

Entregas esperadas:

- workspace/monorepo ou estrutura equivalente justificada;
- Web React + TypeScript + Vite;
- API Node.js + TypeScript;
- packages/módulos independentes para domínio e calculation engine;
- TypeScript strict;
- lint/format/test/build;
- shell visual mínimo;
- documentação de execução;
- nenhum banco, Ollama, scraper ou fórmula física complexa nesta etapa.

Critério: instalação limpa, build, lint e testes passam; frontend e API executam; domínio não depende de React/API.

## Etapa 1B — Fundação visual NEXO

- tokens semânticos NEXO;
- dark/light/system;
- shell, superfícies, tipografia e estados;
- sem cores hardcoded em componentes de produto.

## Etapa 1C — Infraestrutura local Docker

- PostgreSQL;
- MinIO/S3-compatible;
- Ollama;
- health checks;
- volumes nomeados;
- configuração por `.env.example` sem segredos;
- frontend pode permanecer fora do Docker no desenvolvimento se isso simplificar o ciclo local.

Critério: serviços sobem/reiniciam sem perda indevida de dados e nenhum segredo é versionado.

## Etapa 1D — Unidades e contratos físicos básicos

- grandezas canônicas;
- conversões explícitas;
- validação de unidade;
- nenhuma aritmética de grandezas incompatíveis;
- testes de round-trip e limites.

---

# FASE 2 — Domínio e persistência

## Etapa 2A — Modelos centrais

- Component/CatalogComponent;
- ComponentVariant;
- ProjectComponentInstance;
- DroneProject;
- Source/Provenance/Confidence;
- CalculationResult;
- Warning/CompatibilityResult.

## Etapa 2B — PostgreSQL e migrations

- escolher ORM/query builder após avaliar o código real e documentar trade-off;
- migrations versionadas;
- constraints/índices;
- timestamps e IDs estáveis;
- repositories/interfaces desacoplados do domínio.

## Etapa 2C — Object storage

- abstraction de storage;
- MinIO local;
- metadados no PostgreSQL;
- hash e deduplicação quando aplicável;
- política de exclusão/retenção.

## Etapa 2D — Cache/drafts frontend

- IndexedDB somente para cache/drafts/preferências/offline auxiliar;
- banco principal continua PostgreSQL;
- estratégia de invalidação/sincronização simples e explícita.

---

# FASE 3 — Calculation Engine determinístico básico

## Etapa 3A — Massa

- dry mass;
- takeoff mass;
- breakdown por categoria;
- missing data não vira zero silenciosamente.

## Etapa 3B — Bateria básica

- tensão nominal/cheia por química;
- Ah/mAh;
- Wh;
- corrente teórica por C-rating com warning de limitação.

## Etapa 3C — Elétrica básica

- corrente total;
- potência elétrica;
- margem ESC;
- margem teórica da bateria;
- compatibilidades condicionais.

## Etapa 3D — Empuxo/TWR a partir de dados fornecidos

- soma de empuxo conhecido;
- TWR;
- hover target por motor;
- sem inferir empuxo a partir de KV.

## Etapa 3E — Autonomia básica

- capacidade utilizável configurável;
- corrente média/hover conhecida;
- reserva;
- resultado explicitamente estimado.

## Etapa 3F — Warnings

- códigos estáveis;
- severity;
- observado/limite/margem;
- `danger` nunca ocultado por score.

## Etapa 3G — Testes numéricos

- fixtures determinísticas;
- invariantes;
- limites;
- unidades;
- tolerâncias explícitas.

---

# FASE 4 — Catálogo técnico

## Etapa 4A — Schemas por categoria

- motor;
- propeller;
- ESC;
- battery;
- frame;
- FC;
- receiver/VTX;
- camera/GPS;
- custom.

## Etapa 4B — Catálogo e variantes

- componente ≠ instância do projeto;
- variante técnica explícita;
- overrides de projeto não alteram catálogo;
- pesquisa/filtros/índices.

## Etapa 4C — Proveniência e revisões

- fabricante;
- datasheet;
- URL;
- data de captura;
- revisão;
- raw/normalized values quando necessário.

## Etapa 4D — Fórmulas/derivações versionadas

- `formulaId`;
- versão;
- inputs;
- resultado;
- nunca armazenar derivado como se fosse declaração do fabricante.

Critério: cálculos determinísticos possuem IDs/versionamento e testes de limite.

---

# FASE 5 — Cadastro assistido por URL e assets

**Especificações obrigatórias:** `docs/ASSISTED_INGESTION.md` e `docs/MANUFACTURER_SCRAPING.md`.

## Etapa 5A — Source ingestion e scraping seguro de fabricante

Fluxo:

```text
URL informada
→ validação
→ fetch HTTP controlado
→ snapshot de evidência
→ extração determinística
→ staging
```

Controles mínimos:

- apenas HTTP/HTTPS;
- bloqueio de IPs/hosts privados, link-local e loopback;
- proteção contra DNS rebinding quando aplicável;
- revalidação após redirects;
- limite de redirects;
- timeout;
- tamanho máximo de resposta;
- MIME permitido;
- User-Agent identificável quando adequado;
- sem execução arbitrária de scripts;
- sem cookies/tokens privados do usuário;
- sem mecanismos para burlar CAPTCHA, autenticação ou paywall;
- logs sem segredos;
- conteúdo remoto tratado como input hostil, nunca como instrução para o agente.

Priorizar páginas oficiais e datasheets de fabricantes como evidência de alta autoridade, sem tratá-los como infalíveis.

Browser/headless não é requisito inicial. Caso necessário para páginas dinâmicas, deve ser introduzido depois como fallback isolado, com limites próprios e política de rede equivalente ou mais restritiva.

## Etapa 5B — Extração determinística

Prioridade antes de IA:

1. JSON-LD;
2. dados estruturados conhecidos;
3. meta tags;
4. tabelas/especificações legíveis;
5. conteúdo textual normalizado.

Preservar valor bruto, valor normalizado, unidade, fonte e método de extração.

Descobrir links públicos associados ao produto para PDFs, datasheets, manuais, tabelas de empuxo e assets técnicos, sem crawling indiscriminado do site.

## Etapa 5C — Descoberta, classificação e armazenamento de assets

- identificar imagens/documentos candidatos;
- classificar quando possível: produto, ficha técnica, tabela, desenho dimensional, diagrama/pinout, gráfico, etiqueta ou irrelevante;
- preservar URL de origem;
- baixar somente após validação anti-SSRF;
- validar MIME real, tamanho e dimensões;
- limitar quantidade de assets por import job;
- hash SHA-256;
- armazenamento em MinIO/S3-compatible quando permitido;
- metadados e `storageKey` no PostgreSQL;
- aprovação humana antes de publicação quando necessário.

## Etapa 5D — Staging, snapshots e evidência por campo

- `import_jobs`/equivalente;
- `source_snapshots`/equivalente;
- `extracted_fields`;
- vínculo campo → source/asset;
- método de extração;
- região de imagem opcional quando defensável;
- estado de revisão;
- separação entre catálogo publicado e staging;
- nova captura de uma página não sobrescreve silenciosamente revisão já aprovada;
- mudanças de especificação geram candidato de revisão/atualização.

Critério: um cadastro pode ser iniciado a partir de uma página oficial de fabricante; dados estruturados, imagens e documentos ficam auditáveis; e nenhuma informação não revisada é gravada como verdade técnica publicada.

---

# FASE 6 — Ollama e extração assistida por IA

**Especificações obrigatórias:** `docs/ASSISTED_INGESTION.md` e `docs/MANUFACTURER_SCRAPING.md`.

## Etapa 6A — AI Provider abstraction

Contrato deve suportar saída estruturada e evoluir para multimodal sem acoplar o domínio ao Ollama. O provider não controla scraping, destinos de rede nem persistência autoritativa.

Contrato conceitual inicial:

```ts
interface AiProvider {
  extractStructured<T>(request: StructuredExtractionRequest<T>): Promise<AiExtractionResult<T>>
}
```

A IA implementadora pode adotar contrato multimodal unificado se for mais simples/robusto, desde que preserve substituibilidade, validação de schema e testabilidade.

## Etapa 6B — Ollama local

- adapter HTTP;
- modelo configurável por ambiente;
- capability detection;
- timeout/cancelamento;
- saída estruturada validada por schema;
- nenhuma confiança automática em texto do modelo;
- registro de modelo/versão/prompt;
- falha da IA não corrompe staging;
- modelo sem visão não pode fingir análise visual.

## Etapa 6C — Extração multimodal/vision

- analisar fichas técnicas, tabelas, desenhos dimensionais, etiquetas e outros assets elegíveis;
- extrair somente campos previstos no schema da categoria;
- preservar asset/evidência e valor bruto;
- normalização de unidade fora do modelo quando determinística;
- condições fazem parte do dado, por exemplo `35 A @ 5S`;
- confidence da IA não equivale a aprovação;
- prompt injection em texto/imagem/documento não altera permissões ou workflow.

## Etapa 6D — Reconciliação cross-source

- comparar HTML × JSON-LD × tabela × imagem × documento;
- normalizar unidades antes de comparar;
- considerar variante/revisão/condição;
- valores divergentes geram conflito `needs_review`;
- não escolher silenciosamente o maior confidence;
- checks determinísticos podem apontar inconsistência (`P = V × I` etc.), mas não corrigem a fonte automaticamente;
- prioridade da fonte ajuda revisão, mas não substitui consistência física/contextual.

## Etapa 6E — Review workflow

Estados sugeridos:

```text
extracted
→ needs_review
→ approved
→ published
   ou rejected
```

Cada campo extraído deve poder guardar evidência e confiança.

Valor extraído e valor derivado são distintos. Exemplo: `KV` lido da ficha permanece dado de fonte; `Kt` calculado pertence ao Physics Engine com `formulaId`.

## Etapa 6F — Gráficos/curvas em imagem (incremental)

Pode ser adiada sem bloquear o cadastro multimodal básico.

Quando implementada:

- identificar eixos, unidades, escala e legenda;
- marcar pontos como `digitized-from-image` ou equivalente;
- guardar asset/região/método;
- não promover automaticamente para medição de alta confiança;
- preferir CSV/tabela original quando existir;
- exigir revisão humana antes de alimentar cálculos de alta confiança.

Critério da fase: IA sugere dados textuais/visuais estruturados com evidência; schema, reconciliação e revisão decidem o que é publicado.

---

# FASE 7 — Bench data, Workspace Bancada, propulsão e Physics Engine

**Especificações obrigatórias:** `docs/BENCH_DATA_WORKSPACE.md` e `docs/PHYSICS_ENGINE.md`.

Esta fase possui duas rotas complementares. A rota empírica baseada em bancada é preferida quando existe ensaio aplicável. A rota teórica só é usada quando há parâmetros suficientes e nunca deve fabricar coeficientes ou precisão.

## Etapa 7A — Contratos e curvas de bancada

- motor + variante;
- hélice + variante ou descrição estruturada;
- diâmetro/pitch/número de pás quando conhecidos;
- tensão/células/condições;
- ESC quando conhecido;
- throttle;
- empuxo;
- corrente;
- tensão por amostra preferencialmente medida;
- potência medida ou derivável;
- RPM;
- eficiência estática `gf/W` medida ou derivável;
- proveniência e status de revisão.

Regras:

- `throttle %` não é aceleração física;
- empuxo/unidade ambígua exige revisão;
- tensão ausente não vira tensão nominal silenciosamente;
- valores derivados não substituem a fonte original.

## Etapa 7B — Workspace Bancada UI

Criar a seção principal **Bancada** da aplicação.

Entregas:

- listagem/busca de ensaios;
- detalhe do ensaio motor+hélíce+condições;
- criação/edição de amostras;
- tabela com unidades explícitas;
- proveniência/status por dado;
- acesso pelo Catálogo e pela Análise;
- `Como foi calculado?` para valores derivados;
- layout NEXO e acessibilidade;
- gráficos técnicos separados por unidade/grandeza.

Gráficos padrão:

```text
Empuxo × throttle
Corrente/Potência × throttle
RPM × throttle
Eficiência estática (gf/W) × throttle ou empuxo
```

Não usar uma única escala Y para A, gf, RPM e `gf/W` como se fossem grandezas comparáveis.

## Etapa 7C — Importação de bench data

- CSV/tabela;
- preview antes de persistir;
- mapeamento de cabeçalhos;
- schema estrito;
- unidade obrigatória/confirmada quando ambígua;
- suporte a vírgula/ponto decimal conforme contratos do projeto;
- preservação do valor bruto;
- duplicatas/conflitos;
- rollback/atomicidade quando persistência final falhar;
- nenhuma extrapolação silenciosa.

Dados vindos de scraping, PDF ou imagem seguem staging/revisão das Fases 5 e 6.

## Etapa 7D — Métricas derivadas e validação de consistência

Implementar no calculation engine, não na UI:

### Potência elétrica

```text
P = V × I
```

### Eficiência estática de empuxo

```text
static_thrust_efficiency_gf_per_W = thrust_gf / power_W
```

Não interpretar como eficiência percentual.

### Velocidade teórica de passo

```text
pitch_m = pitch_in × 0.0254
pitch_speed_km_h = pitch_m × RPM × 60 / 1000
```

Nome obrigatório: **Velocidade teórica de passo**.

Deve carregar aviso de que não representa velocidade real/máxima da aeronave e ignora slip, arrasto, advance ratio e outros efeitos aerodinâmicos.

### Checks

Incluir warnings equivalentes a:

```text
BENCH_VOLTAGE_MISSING
BENCH_POWER_INCONSISTENT
BENCH_EFFICIENCY_UNIT_AMBIGUOUS
BENCH_THRUST_UNIT_AMBIGUOUS
BENCH_RPM_EXCEEDS_NO_LOAD_REFERENCE
BENCH_SAMPLE_DUPLICATE
BENCH_SAMPLE_INVALID
PITCH_SPEED_NOT_AIRCRAFT_SPEED
```

## Etapa 7E — Interpolação e propulsão medida

- interpolação apenas dentro da faixa medida/aprovada;
- fora da faixa → unavailable/warning;
- método/versionamento;
- testes de fronteira;
- thrust/current/power/RPM no ponto solicitado quando disponíveis;
- hover current por motor;
- eficiência `gf/W`;
- total do sistema;
- lookup preferencial por empuxo requerido para hover, não linearidade presumida de throttle.

## Etapa 7F — Contratos do Physics Engine

- parâmetros de bateria;
- parâmetros de ESC/drive;
- parâmetros eletromecânicos do motor;
- parâmetros aerodinâmicos da hélice;
- atmosfera;
- operating point/result/provenance.

## Etapa 7G — Bateria sob carga

- OCV/SOC quando houver modelo;
- resistência interna;
- sag `V_load ≈ V_oc - I × R_internal` como modelo inicial explícito;
- limites por química;
- temperatura apenas quando houver modelo/dados suficientes.

## Etapa 7H — Motor eletromecânico e torque

- KV;
- conversão cuidadosa para Ke/Kt em SI;
- resistência do enrolamento;
- corrente sem carga/perdas;
- back-EMF;
- torque;
- potência mecânica;
- eficiência;
- limites e hipóteses.

## Etapa 7I — Hélice aerodinâmica

Quando houver coeficientes/dados suficientes:

- diâmetro;
- pitch/geometria identificável;
- número de pás;
- Ct/Cq/Cp;
- densidade do ar;
- thrust/torque/power por RPM.

Sem coeficientes ou curva aplicável, não fabricar resultado de alta confiança.

## Etapa 7J — Solver do ponto de operação

Resolver numericamente o equilíbrio aproximado:

```text
torque disponível do motor(RPM, I, V)
=
torque requerido pela hélice(RPM, rho, geometry)
```

Entregas:

- convergência controlada;
- limites físicos;
- erro explícito quando não convergir;
- sem solução silenciosamente fora do envelope;
- resultado com RPM, corrente, torque, thrust, potência e eficiência quando calculáveis.

## Etapa 7K — Validação contra bancada

- comparar modelo teórico com ensaios reais;
- erro absoluto/relativo;
- tolerâncias documentadas;
- calibrar/limitar confiança;
- modelo que diverge além do gate não pode ser apresentado como alta confiança.

Critério de saída da Fase 7: o usuário consegue cadastrar/importar/revisar uma curva na seção Bancada, visualizar grandezas corretamente, usar dados aprovados para análise sem extrapolação silenciosa e comparar futuramente o modelo físico contra bancada real.

---

# FASE 8 — Perfis, heurísticas e Candidate Generator

## Etapa 8A — Knowledge/heuristic model

- heurísticas versionadas;
- source/evidence;
- confidence;
- condições;
- nenhuma heurística vira regra rígida automaticamente.

## Etapa 8B — Flight profiles

- Freestyle;
- Racing;
- Cinematic;
- Long Range;
- Mini Long Range;
- Cinewhoop;
- prioridades/targets configuráveis.

## Etapa 8C — Sub-250 e experiência

- sub-250 como constraint independente;
- nível de experiência influencia margem/recomendação, não física.

## Etapa 8D — Candidate Generator

```text
heurísticas
→ candidatos
→ compatibilidade
→ cálculo/physics
→ profile score
→ ranking
```

`danger` sempre elimina/penaliza de forma dominante conforme regra documentada; score não mascara incompatibilidade crítica.

---

# FASE 9 — Builder e análise de projeto

## Etapa 9A — Primeiro Builder UI

- criar/abrir projeto;
- adicionar componentes;
- overrides de instância;
- resumo de massa/energia/TWR;
- warnings.

## Etapa 9B — Perfil por estilo de voo

- escolha de perfil;
- sliders/traits secundários;
- recomendações explicáveis.

## Etapa 9C — Mini Long Range

- eficiência/endurance/baixo peso/estabilidade;
- 3.5–5\" como orientação flexível, não regra rígida;
- LiPo/Li-Ion conforme objetivo;
- ranking pode preferir menor TWR quando endurance/eficiência forem superiores e segurança continuar válida.

---

# FASE 10 — Comparação e centro de gravidade

## Etapa 10A — Duplicação/variantes

- copiar projeto;
- alterar motor/bateria/hélice;
- comparação lado a lado.

## Etapa 10B — Centro de gravidade

- posições x/y/z;
- CG ponderado por massa;
- visualização 2D inicialmente;
- 3D futuro se justificar complexidade.

---

# FASE 11 — Relatórios e exportação

- export/import versionado;
- relatório técnico;
- provenance/confidence;
- warnings;
- configuração usada;
- sem afirmar certificação/garantia de voo.

---

# FASE 12 — Otimizador

Somente após catálogo, dados e cálculo estarem maduros.

Inputs possíveis:

- payload;
- autonomia mínima;
- peso máximo;
- células/química;
- perfil de voo;
- limites de frame/hélice;
- disponibilidade de dados confiáveis.

Ranking multiobjetivo deve ser explicável e nunca sobrepor incompatibilidade crítica.

---

# FASE 13 — Evoluções opcionais

- pgvector/RAG quando houver necessidade comprovada;
- catálogo colaborativo;
- sincronização multi-dispositivo;
- API pública/privada;
- Tauri/desktop;
- mais classes de multirrotores;
- modelos térmicos;
- atmosfera avançada;
- custo/disponibilidade de peças.

---

# Gates transversais

Toda etapa deve passar por:

1. requisitos entendidos;
2. riscos levantados;
3. trade-offs registrados quando relevantes;
4. implementação pequena/coesa;
5. testes;
6. auto-auditoria de regressões/segurança;
7. documentação atualizada;
8. nenhum segredo no repositório.

Para ingestão remota, adicionar obrigatoriamente testes de SSRF/redirect/MIME/tamanho/prompt injection conforme a etapa. Para dados de bancada, adicionar testes de unidade, potência, `gf/W`, pitch speed, importação e ausência de extrapolação. Para Physics Engine, adicionar fixtures e comparação com dados reais quando disponíveis.

---

# Dependências principais

```text
1A Bootstrap workspace
 ├─ 1B NEXO
 ├─ 1C Docker services
 └─ 1D Units/contracts
       ↓
2 Domain/PostgreSQL/Object Storage
       ↓
3 Basic Calculation Engine
       ↓
4 Catalog
 ├──────────────┐
 ↓              ↓
5 URL/Scraping  7A Bench contracts
 ↓              ↓
6 Ollama        7B–7E Bancada/Measured Propulsion
 └───────┐      │
         └──────┤
                ↓
          7F–7K Physics Engine
                ↓
8 Profiles/Heuristics/Candidate Generator
                ↓
9 Builder/Analysis
                ↓
10–12 Comparison/Reports/Optimizer
```

---

# Próximo passo autorizado para implementação

**Etapa 1A — Bootstrap full-stack e workspace.**

Não antecipar PostgreSQL, MinIO, Ollama, scraping, Workspace Bancada ou Physics Engine para essa etapa. O objetivo é criar a fundação que permitirá implementar esses subsistemas sem acoplamento prematuro.

A orientação executável está em:

`docs/prompts/ETAPA_1A_BOOTSTRAP_FULLSTACK.md`

A implementação futura da seção Bancada deve seguir:

`docs/prompts/ETAPA_7_BENCH_DATA_WORKSPACE.md`

A IA implementadora pode propor estrutura alternativa mais eficaz/eficiente desde que preserve os requisitos, justifique trade-offs, implemente testes e atualize a documentação quando necessário.