# Roadmap — DroneCalc

**Versão:** 0.8  
**Status:** planejamento técnico revisado  
**Estratégia:** construir uma fundação full-stack pequena e auditável, mantendo o motor de cálculo independente da UI, persistência e IA; evoluir propulsão por dados de bancada/modelo físico; recomendar conjuntos com massa recalculada; e adicionar planejamento de enlace FPV por link budget auditável.

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
- recomendação final deve avaliar conjunto motor + hélice + bateria/tensão, não motor isolado;
- a massa deve ser recalculada para cada candidato antes de avaliar hover, TWR, consumo e autonomia;
- incompatibilidade `danger` é avaliada antes do score e nunca pode ser mascarada por ranking;
- score de adequação, cobertura de dados e confiança são dimensões separadas;
- `gf/W` deve ser tratado como eficiência estática de empuxo, nunca como eficiência percentual;
- `pitch × RPM` produz apenas velocidade teórica de passo e nunca deve ser apresentado como velocidade real/máxima do drone;
- alcance FPV por FSPL deve ser apresentado como estimativa de espaço livre, nunca garantia de alcance real;
- perdas de antena não podem ser aplicadas duas vezes por desconhecer a semântica de `gain/directivity/realized gain`;
- link de vídeo FPV e link de rádio-controle são análises separadas;
- potência RF necessária não implica autorização regulatória;
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
- recomendação de propulsão orientada ao uso em `docs/PROPULSION_RECOMMENDATION.md`;
- alcance FPV/link budget em `docs/FPV_LINK_BUDGET.md`;
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
- incluir unidades/logaritmos RF necessários posteriormente sem misturar dBm, dB e potência linear;
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
- atualização ao adicionar/remover/substituir peças;
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
- receiver de controle;
- VTX;
- VRX/FPV goggles;
- FPV antenna;
- RF cable/pigtail opcional;
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

## Etapa 4E — Campos RF básicos

### VTX
- frequência/canais;
- potência configurável mW/dBm;
- massa e consumo;
- conectores;
- limites declarados.

### VRX/óculos
- frequência;
- sensibilidade + condição/modo;
- branches/diversity;
- conectores.

### Antenas
- frequência/faixa;
- ganho dBi;
- `gainKind`: directivity/gain/realized-gain/unknown;
- polarização;
- SWR quando conhecido;
- eficiência quando realmente fornecida;
- conectores/massa.

Critério: cálculos determinísticos possuem IDs/versionamento e dados RF não perdem a semântica necessária para evitar dupla contagem de perdas.

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
- condições fazem parte do dado;
- confidence da IA não equivale a aprovação;
- prompt injection em texto/imagem/documento não altera permissões ou workflow.

## Etapa 6D — Reconciliação cross-source

- comparar HTML × JSON-LD × tabela × imagem × documento;
- normalizar unidades antes de comparar;
- considerar variante/revisão/condição;
- valores divergentes geram conflito `needs_review`;
- não escolher silenciosamente o maior confidence;
- checks determinísticos podem apontar inconsistência, mas não corrigem a fonte automaticamente.

## Etapa 6E — Review workflow

```text
extracted
→ needs_review
→ approved
→ published
   ou rejected
```

Cada campo extraído deve poder guardar evidência e confiança.

## Etapa 6F — Gráficos/curvas em imagem (incremental)

- identificar eixos, unidades, escala e legenda;
- marcar pontos como `digitized-from-image`;
- guardar asset/região/método;
- preferir CSV/tabela original quando existir;
- exigir revisão humana antes de alimentar cálculos de alta confiança.

Critério da fase: IA sugere dados estruturados com evidência; schema, reconciliação e revisão decidem o que é publicado.

---

# FASE 7 — Bench data, Workspace Bancada, propulsão e Physics Engine

**Especificações obrigatórias:** `docs/BENCH_DATA_WORKSPACE.md` e `docs/PHYSICS_ENGINE.md`.

## Etapa 7A — Contratos e curvas de bancada

- motor + variante;
- hélice + variante;
- diâmetro/pitch/número de pás;
- tensão/células/condições;
- ESC quando conhecido;
- throttle;
- empuxo;
- corrente;
- tensão por amostra preferencialmente medida;
- potência medida ou derivável;
- RPM;
- eficiência estática `gf/W`;
- proveniência/status.

## Etapa 7B — Workspace Bancada UI

- listagem/busca de ensaios;
- detalhe motor+hélíce+condições;
- criação/edição de amostras;
- tabela com unidades explícitas;
- proveniência/status;
- `Como foi calculado?`;
- gráficos separados por grandeza.

## Etapa 7C — Importação de bench data

- CSV/tabela;
- preview;
- mapeamento de cabeçalhos;
- schema estrito;
- unidades explícitas;
- raw values;
- duplicatas/conflitos;
- rollback/atomicidade;
- sem extrapolação silenciosa.

## Etapa 7D — Métricas derivadas

```text
P = V × I
static_thrust_efficiency_gf_per_W = thrust_gf / power_W
pitch_speed_km_h = pitch_m × RPM × 60 / 1000
```

Pitch speed deve ser rotulada como teórica, nunca velocidade real.

## Etapa 7E — Interpolação e propulsão medida

- interpolação apenas dentro da faixa aprovada;
- lookup preferencial por empuxo requerido;
- hover current;
- eficiência;
- total do sistema.

## Etapa 7F — Contratos do Physics Engine

- bateria;
- ESC/drive;
- motor eletromecânico;
- hélice;
- atmosfera;
- operating point/proveniência.

## Etapa 7G — Bateria sob carga

- OCV/SOC quando houver modelo;
- resistência interna;
- sag;
- limites por química.

## Etapa 7H — Motor eletromecânico e torque

- KV/Ke/Kt;
- resistência do enrolamento;
- corrente sem carga/perdas;
- back-EMF;
- torque;
- potência mecânica;
- eficiência.

## Etapa 7I — Hélice aerodinâmica

- diâmetro/pitch/geometria;
- número de pás;
- Ct/Cq/Cp quando disponíveis;
- densidade do ar;
- thrust/torque/power por RPM.

## Etapa 7J — Solver do ponto de operação

Resolver equilíbrio motor × hélice com convergência controlada e limites físicos.

## Etapa 7K — Validação contra bancada

- erro absoluto/relativo;
- tolerâncias;
- calibração de confiança;
- modelo divergente não recebe alta confiança.

Critério: dados medidos e modelo físico podem sustentar análises defensáveis de propulsão.

---

# FASE 8 — Perfis, heurísticas e recomendação de propulsão

**Especificação obrigatória:** `docs/PROPULSION_RECOMMENDATION.md`.

## Etapa 8A — Knowledge/heuristic model

- heurísticas versionadas;
- source/evidence;
- confidence;
- condições.

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
heurísticas/catálogo
→ motor + hélice + bateria/tensão (+ ESC)
→ candidatos
```

## Etapa 8E — Perfil operacional orientado ao uso

- pesos relativos de hover/cruzeiro/subida/agressivo/reserva;
- normalização/versionamento;
- fase sem dado não vira zero.

## Etapa 8F — Avaliação com recálculo de massa

```text
variante virtual
→ massa total
→ empuxo requerido
→ TWR/reserva
→ compatibilidade
→ bancada/Physics Engine
→ corrente/potência/eficiência/autonomia
```

## Etapa 8G — Ranking multiobjetivo explicável

- métricas normalizadas;
- pesos do perfil;
- `profileScore`, `dataCoverage` e `confidence` separados;
- motivos positivos/negativos;
- determinismo.

## Etapa 8H — Testes do recomendador

- massa por candidato;
- eficiência versus empuxo máximo;
- `danger` dominante;
- cobertura/confiança;
- explicabilidade.

Critério: recomendar conjuntos de propulsão de forma explicável conforme uso.

---

# FASE 9 — Alcance FPV / RF Link Budget

**Especificação obrigatória:** `docs/FPV_LINK_BUDGET.md`.

Esta fase dimensiona o **link de vídeo FPV**. Não representa nem substitui o link de rádio-controle.

## Etapa 9A — Contratos RF e conversões

Implementar no calculation engine:

```text
mW ↔ dBm
dB/dBi sem mistura com potência linear
antenna gain semantics
SWR → mismatch loss
```

Requisitos:

- `directivity`, `gain`, `realized-gain`, `unknown` explicitamente distintos;
- evitar dupla contagem de eficiência/mismatch;
- validação numérica de distância, frequência, potência, SWR e eficiência;
- formulaIds versionados.

Testes de referência:

```text
100 mW = 20 dBm
200 mW ≈ 23.0103 dBm
SWR 1.0 = 0 dB
SWR 1.5 ≈ 0.177 dB
```

## Etapa 9B — FSPL e link budget direto

Implementar:

```text
FSPL_dB = 32.44 + 20log10(d_km) + 20log10(f_MHz)
Pr = Pt + Gt + Gr - losses - FSPL
availableMargin = Pr - sensitivity
headroom = availableMargin - desiredMargin
```

Saída:

- FSPL;
- potência recebida estimada;
- sensibilidade usada + condição;
- margem disponível;
- margem desejada;
- headroom;
- `pass/borderline/fail/insufficient-data`.

Resultado é de espaço livre e deve carregar warning explícito.

## Etapa 9C — Solver inverso por distância-alvo

O operador informa `targetDistanceKm`.

Calcular:

- potência teórica mínima de VTX para fechar o link com margem desejada;
- distância teórica máxima em espaço livre para a configuração existente;
- potência necessária em dBm e mW;
- EIRP calculada quando inputs forem suficientes.

Não associar automaticamente potência necessária a legalidade/regulamentação.

## Etapa 9D — VRX/óculos e antenas do catálogo

- selecionar VRX/óculos cadastrado ou entrada manual;
- selecionar uma ou mais antenas de recepção;
- preservar sensibilidade condicionada por modo/sistema;
- suportar polarização;
- suportar perdas de cabo/conector;
- diversity do tipo seleção avalia branches separadamente;
- não somar ganhos de antennas diversity como array coerente.

## Etapa 9E — UI `Alcance FPV`

Entradas mínimas:

```text
Distância desejada (km)
Frequência/canal
Margem desejada
VTX/potência
Antena TX
VRX/óculos
Antena(s) RX
Perdas adicionais opcionais
```

Saídas:

```text
FSPL
Potência recebida
Sensibilidade
Margem disponível
Headroom
Potência VTX mínima teórica
EIRP
Distância teórica máxima em espaço livre
Warnings/hipóteses
```

A UI deve explicar `Como foi calculado?` e diferenciar campos medidos, fabricante, user-provided e calculados.

## Etapa 9F — Recomendação RF orientada à distância

Quando o catálogo permitir:

```text
frequência compatível
→ conectores/polarização
→ sensibilidade
→ ganho/perdas
→ link budget na distância-alvo
→ margem desejada
→ massa/consumo do VTX
→ ranking explicável
```

A recomendação de VTX/antena deve considerar impacto de massa e consumo no projeto quando disponível.

## Etapa 9G — Testes e limitações

Cobrir:

- mW/dBm;
- FSPL/inversão;
- solver de potência;
- SWR;
- realized gain sem dupla perda;
- sensibilidade ausente;
- diversity simplificado;
- analógico/digital com condição de sensibilidade;
- distância/frequência inválidas;
- warning `RF_FREE_SPACE_ONLY_MODEL`;
- warning de status regulatório não verificado.

Critério de saída da Fase 9: o operador informa a distância desejada e dados/peças do sistema FPV, e o DroneCalc calcula um link budget auditável, margem e potência teórica necessária sem apresentar o resultado como alcance garantido.

---

# FASE 10 — Builder e análise de projeto

## Etapa 10A — Primeiro Builder UI

- criar/abrir projeto;
- adicionar componentes;
- overrides de instância;
- massa atualizada ao alterar peças;
- resumo de massa/energia/TWR;
- warnings.

## Etapa 10B — Perfil por estilo de voo

- escolha de perfil;
- sliders/traits secundários;
- perfil operacional quando suportado;
- recomendações explicáveis;
- `targetFpvRangeKm` opcional como constraint/intenção separada.

## Etapa 10C — Recomendação de conjunto de propulsão na UI

- ação `Recomendar propulsão`;
- candidatos motor + hélice + bateria/tensão;
- massa final;
- hover/TWR/consumo/eficiência/autonomia;
- compatibilidade;
- score/cobertura/confiança;
- motivos.

## Etapa 10D — Integração Alcance FPV no projeto

- acesso à calculadora RF pelo projeto ativo;
- reaproveitar VTX/antenas selecionados no Builder;
- distância-alvo persistida no projeto quando o usuário escolher;
- warnings RF aparecem na Análise sem substituir warnings de controle RC;
- consumo/massa do VTX entram normalmente no orçamento elétrico/massa.

## Etapa 10E — Mini Long Range

- eficiência/endurance/baixo peso/estabilidade;
- 3.5–5\" como orientação flexível;
- LiPo/Li-Ion conforme objetivo;
- alcance FPV desejado pode ser constraint adicional, sem garantir alcance real;
- ranking de propulsão continua independente do link budget RF, embora ambos usem o mesmo projeto.

---

# FASE 11 — Comparação e centro de gravidade

## Etapa 11A — Duplicação/variantes

- copiar projeto;
- alterar motor/bateria/hélice/VTX/antena;
- comparação lado a lado.

## Etapa 11B — Centro de gravidade

- posições x/y/z;
- CG ponderado por massa;
- visualização 2D inicialmente;
- 3D futuro.

---

# FASE 12 — Relatórios e exportação

- export/import versionado;
- relatório técnico;
- provenance/confidence;
- warnings;
- configuração usada;
- recomendação/ranking com perfil e versão;
- link budget FPV com premissas, distância-alvo e aviso de espaço livre;
- sem afirmar certificação/garantia de voo ou alcance RF.

---

# FASE 13 — Otimizador global/avançado

Somente após catálogo, dados, recomendador básico e cálculo estarem maduros.

A Fase 8 entrega recomendação de propulsão; a Fase 9 entrega planejamento FPV. Esta fase amplia busca/otimização em escala.

Inputs possíveis:

- payload;
- autonomia mínima;
- peso máximo;
- células/química;
- perfil de voo;
- limites de frame/hélice;
- target FPV range;
- disponibilidade de dados confiáveis;
- custo/disponibilidade futuros.

Evoluções possíveis:

- dimensionamento iterativo de bateria;
- exploração de muitas combinações;
- Pareto front;
- otimização multiobjetivo;
- recomendação conjunta de propulsão + RF sem fundir suas físicas;
- poda/estratégias de busca eficientes.

Hard constraints continuam dominantes.

---

# FASE 14 — Evoluções opcionais

- zona de Fresnel e clearance;
- radio horizon/alturas;
- terreno/LOS com mapa/DEM;
- noise floor/interferência medida;
- link RC/ExpressLRS/Crossfire separado;
- verificação regulatória por região;
- pgvector/RAG quando necessário;
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

Para ingestão remota, testar SSRF/redirect/MIME/tamanho/prompt injection. Para bancada, testar unidade, potência, `gf/W`, pitch speed e ausência de extrapolação. Para Physics Engine, comparar com dados reais. Para recomendação de propulsão, testar massa por candidato, hard constraints e explicabilidade. Para RF, testar dBm/mW, FSPL, solver inverso, SWR, semântica de ganho, sensibilidade condicionada e avisos de espaço livre/regulamentação.

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
4 Catalog + RF schemas
 ├───────────────┬─────────────────┐
 ↓               ↓                 ↓
5 URL/Scraping   7 Bench/Physics   9A–9D RF Core
 ↓               ↓                 │
6 Ollama         8 Propulsion Rec  │
 └──────┐        │                 │
        └────────┴────────┬────────┘
                         ↓
                  9E–9G FPV UI/Tests
                         ↓
                 10 Builder/Analysis
                         ↓
             11–12 Compare/Reports
                         ↓
                13 Global Optimizer
```

---

# Próximo passo autorizado para implementação

**Etapa 1A — Bootstrap full-stack e workspace.**

Não antecipar PostgreSQL, MinIO, Ollama, scraping, Workspace Bancada, Physics Engine, recomendador ou RF Link Budget para essa etapa.

Prompts disponíveis:

- `docs/prompts/ETAPA_1A_BOOTSTRAP_FULLSTACK.md`;
- `docs/prompts/ETAPA_7_BENCH_DATA_WORKSPACE.md`;
- `docs/prompts/ETAPA_8_PROPULSION_RECOMMENDATION.md`;
- `docs/prompts/ETAPA_9_FPV_LINK_BUDGET.md`.

A IA implementadora pode propor estrutura alternativa mais eficaz/eficiente desde que preserve requisitos, justifique trade-offs, implemente testes e atualize documentação quando necessário.
