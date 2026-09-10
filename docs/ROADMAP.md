# Roadmap — DroneCalc

**Versão:** 0.3  
**Status:** planejamento técnico revisado  
**Estratégia:** construir uma fundação full-stack pequena e auditável, mantendo o motor de cálculo independente da UI, da persistência e da IA e evoluindo a propulsão por duas rotas: dados de bancada e modelo físico validável.

## 1. Princípios de execução

- etapas pequenas, coesas e testáveis;
- nenhuma fórmula de engenharia dentro da UI;
- PostgreSQL será a fonte principal de dados persistidos do produto;
- IndexedDB poderá ser usado apenas para cache, preferências, drafts e suporte offline;
- imagens e documentos não devem ser armazenados como blobs grandes no PostgreSQL; usar object storage compatível com S3;
- Ollama será integrado por uma interface própria, sem dependência do domínio em um modelo específico;
- dados extraídos por IA entram primeiro em staging e nunca são publicados automaticamente como verdade técnica;
- importação por URL deve ser tratada como superfície de segurança e protegida contra SSRF, redirects abusivos, payloads excessivos e tipos de conteúdo inesperados;
- preferir dados medidos e documentos de fabricante a heurísticas genéricas;
- heurística orienta candidatos; validação física decide compatibilidade;
- dados de bancada aplicáveis têm prioridade sobre previsões teóricas;
- modelo teórico de propulsão só produz resultado quando houver parâmetros suficientes e sempre informa modelo, hipóteses, proveniência e confiança;
- nunca inferir empuxo real apenas de `KV + diâmetro da hélice + tensão`;
- cada etapa atualiza documentação e ADR quando alterar contrato ou arquitetura;
- manter o projeto executável ao final de cada etapa.

---

## Etapa 0 — Especificação e decisões

**Status:** documentação-base criada e em evolução.

Entregas:

- PRD e Product Spec;
- arquitetura e modelo de domínio;
- motor de cálculo;
- Physics Engine para bateria/motor/torque/hélice/propulsão;
- perfis de voo;
- identidade NEXO;
- UX, dados, catálogo, testes e segurança;
- guia de implementação por IA;
- ADRs de decisões estruturais.

Critério de saída: decisões críticas não ficam implícitas em chat ou prompt temporário.

---

# FASE 1 — Fundação do repositório

## Etapa 1A — Bootstrap full-stack e workspace

**Objetivo:** criar a estrutura mínima do produto sem implementar ainda banco, scraping, IA ou fórmulas.

Estrutura-alvo conceitual:

```text
Dronecalc/
├── apps/
│   ├── web/
│   └── api/
├── packages/
│   ├── domain/
│   ├── calculation-engine/
│   └── contracts/
├── infrastructure/
├── docs/
└── package.json
```

Entregas:

- workspace JavaScript/TypeScript;
- `apps/web`: React + TypeScript + Vite;
- `apps/api`: Node.js + TypeScript com endpoint mínimo de health check;
- `packages/domain`: pacote puro sem React/DOM/PostgreSQL/Ollama;
- `packages/calculation-engine`: pacote puro dependente apenas de domínio/contratos necessários;
- `packages/contracts`: contratos compartilháveis quando fizer sentido;
- TypeScript strict;
- lint/format;
- Vitest;
- scripts raiz `dev`, `build`, `test`, `typecheck`, `lint`;
- `.env.example` sem segredos;
- CI inicial;
- README atualizado com comandos reais;
- relatório da etapa.

**Fora de escopo:** ORM, migrations, PostgreSQL, MinIO, Ollama, scraping, catálogo real e fórmulas de drone.

Critério de saída: todos os workspaces compilam, testes mínimos passam e a API responde health check.

## Etapa 1B — NEXO Design System base

Entregas:

- tokens NEXO sincronizados e versionados no próprio DroneCalc;
- temas light/dark/system;
- Button, Input, Select, Badge, Card e Alert;
- shell desktop inicial;
- estados de foco/acessibilidade;
- registro da revisão NEXO usada.

Critério: nenhuma feature usa cor hardcoded fora do design system.

## Etapa 1C — Infraestrutura local com Docker

**Objetivo:** disponibilizar os serviços de infraestrutura sem acoplar o domínio a eles.

Serviços previstos:

```text
PostgreSQL
MinIO
Ollama
```

Entregas:

- `docker-compose.yml` ou equivalente;
- volumes persistentes;
- health checks;
- variáveis em `.env.example`;
- credenciais locais de desenvolvimento não sensíveis/documentadas;
- nenhuma credencial real versionada;
- rede local mínima necessária;
- documentação de subida/parada/limpeza;
- testes de conectividade da API apenas quando pertinente.

Não adicionar Redis, fila ou serviços extras sem problema concreto.

Critério: ambiente sobe de forma reproduzível e cada serviço reporta saúde.

## Etapa 1D — Unidades, validação física e contratos-base

Entregas:

- tipos/brands de unidades;
- conversões de massa, comprimento, capacidade, tensão, corrente, potência, energia, força, torque, velocidade angular e rotação;
- parser de entrada pt-BR;
- formatters;
- `Availability<T>`;
- proveniência/confiança base;
- validações de números fisicamente inválidos.

Critério: suíte de conversões e casos inválidos completa, incluindo RPM ↔ rad/s e N·m.

---

# FASE 2 — Plataforma de dados

## Etapa 2A — PostgreSQL e migrations

Objetivo: tornar PostgreSQL a fonte de verdade persistida.

Entregas:

- escolha documentada de biblioteca de acesso/migrations;
- migration inicial;
- convenções de IDs e timestamps;
- transaction boundaries;
- configuração por ambiente;
- testes de migration em banco limpo.

A biblioteca pode ser Prisma, Drizzle, query builder ou SQL tipado equivalente, desde que a escolha seja baseada em manutenção, transparência, migrations e ergonomia do código real. Não escolher por moda.

## Etapa 2B — Repository interfaces e adapters

Entregas:

- interfaces no application/domain quando cabível;
- implementações PostgreSQL no backend;
- nenhum import de driver SQL no domínio;
- testes de integração;
- tratamento de concorrência e transações.

## Etapa 2C — Object Storage

Entregas:

- abstração para arquivos/imagens;
- MinIO como implementação local;
- PostgreSQL armazena metadados e `storageKey`, não arquivos grandes;
- hash SHA-256 para deduplicação/integridade quando útil;
- limites de tamanho e MIME.

## Etapa 2D — Cache/drafts no frontend

IndexedDB fica restrito a:

- draft em edição;
- preferências;
- último projeto;
- cache controlado do catálogo;
- eventual fila offline.

PostgreSQL permanece fonte autoritativa quando houver conexão com backend.

---

# FASE 3 — Domínio, projetos e catálogo

## Etapa 3A — Modelos de componentes

Implementar tipos e schemas validados para:

- frame;
- motor;
- hélice;
- ESC;
- bateria;
- FC;
- GPS;
- receiver;
- VTX;
- câmera;
- BEC/power module;
- payload;
- componente custom.

Os modelos de motor/hélice/bateria devem permitir adicionar posteriormente parâmetros físicos opcionais sem inventar defaults: resistência interna, corrente sem carga, `Kt`, coeficientes `Ct/Cq/Cp`, curvas por advance ratio e condições de ensaio.

## Etapa 3B — DroneProject

- projeto;
- instâncias de componentes;
- constraints;
- assumptions;
- overrides por projeto;
- duplicação/variantes.

## Etapa 3C — Catálogo persistente

- CRUD;
- busca/filtros;
- manufacturers/models/variants;
- source metadata;
- revisão e completude;
- componentes custom separados do catálogo de referência.

## Etapa 3D — Import/export versionado

- JSON autocontido;
- schemaVersion;
- validação de entrada;
- round-trip tests.

Critério da fase: criar, salvar, reabrir e duplicar um projeto usando componentes persistidos.

---

# FASE 4 — Motor técnico básico

## Etapa 4A — Massa

- linhas e quantidades;
- dry mass;
- battery mass;
- payload;
- takeoff mass;
- totais parciais quando faltarem dados.

## Etapa 4B — Bateria

- química;
- tensão nominal/cheia;
- Ah/Wh;
- C-rating derivado;
- corrente medida/recomendada quando disponível.

## Etapa 4C — Compatibilidade elétrica

- bateria × ESC;
- bateria × motor;
- ESC × motor;
- cargas auxiliares;
- margens;
- warnings explicáveis.

Critério: cálculos determinísticos possuem IDs/versionamento e testes de limite.

---

# FASE 5 — Cadastro assistido por URL

## Etapa 5A — Source ingestion seguro

Fluxo:

```text
URL informada
→ validação
→ fetch controlado
→ snapshot de evidência
→ extração determinística
→ staging
```

Controles mínimos:

- apenas HTTP/HTTPS;
- bloqueio de IPs/hosts privados e loopback;
- proteção contra DNS rebinding quando aplicável;
- limite de redirects;
- timeout;
- tamanho máximo de resposta;
- MIME permitido;
- User-Agent identificável;
- sem execução arbitrária de scripts;
- logs sem segredos.

## Etapa 5B — Extração determinística

Prioridade antes de IA:

1. JSON-LD;
2. dados estruturados conhecidos;
3. meta tags;
4. tabelas/especificações legíveis;
5. conteúdo textual normalizado.

## Etapa 5C — Extração e validação de imagens

- identificar imagens candidatas;
- preservar URL de origem;
- baixar somente após validação;
- MIME e tamanho limitados;
- hash;
- armazenamento em MinIO;
- aprovação humana antes de publicação quando necessário.

Critério: um cadastro pode ser iniciado a partir de URL sem gravar informação não revisada como verdade técnica.

---

# FASE 6 — Ollama e extração assistida por IA

## Etapa 6A — AI Provider abstraction

Contrato conceitual:

```ts
interface AiProvider {
  extractStructured<T>(request: StructuredExtractionRequest<T>): Promise<AiExtractionResult<T>>
}
```

O domínio não conhece Ollama.

## Etapa 6B — Ollama local

- adapter HTTP;
- modelo configurável por ambiente;
- timeout/cancelamento;
- saída estruturada validada por schema;
- nenhuma confiança automática em texto do modelo;
- registro de modelo/versão/prompt;
- falha da IA não corrompe staging.

## Etapa 6C — Review workflow

Estados sugeridos:

```text
extracted
→ needs_review
→ approved
→ published
   ou rejected
```

Cada campo extraído deve poder guardar evidência e confiança.

Critério: IA sugere; schema e revisão decidem.

---

# FASE 7 — Bench data, propulsão e Physics Engine

Esta fase possui duas rotas complementares. A rota empírica baseada em bancada é preferida quando existe ensaio aplicável. A rota teórica só é usada quando há parâmetros suficientes e nunca deve fabricar coeficientes ou precisão.

## Etapa 7A — Bench test model

- motor + hélice + tensão/condições;
- samples;
- fonte;
- revisão.

## Etapa 7B — Importador CSV

- mapeamento;
- unidades;
- preview;
- validação.

## Etapa 7C — Interpolação

- lookup por empuxo;
- current/power/throttle;
- sem extrapolação silenciosa.

## Etapa 7D — Propulsion analysis por bancada

- empuxo total;
- TWR;
- hover thrust por motor;
- eficiência g/W.

Critério: resultado é rastreável até curva e amostras de origem.

## Etapa 7E — Contratos do Physics Engine

**Especificação obrigatória:** `docs/PHYSICS_ENGINE.md`.

Entregas:

- contratos SI para força, torque, RPM/velocidade angular e densidade do ar;
- `PhysicsContext`;
- estado operacional de bateria;
- parâmetros eletromecânicos do motor;
- parâmetros/coefs de hélice;
- `PropulsionOperatingPoint`;
- IDs e versionamento dos modelos;
- política de seleção `bench data > modelo físico > dados insuficientes`.

Nenhuma fórmula avançada deve ser implementada antes de seus contratos, unidades e casos de teste estarem definidos.

## Etapa 7F — Bateria sob carga

Implementar de forma incremental:

- tensão de circuito aberto quando disponível;
- resistência interna do pack;
- modelo de sag equivalente de Thévenin inicial;
- potência entregue;
- margem para cutoff;
- SOC/temperatura apenas quando houver dados/modelo defensável.

Proibido usar C-rating como substituto de resistência interna.

## Etapa 7G — Modelo eletromecânico do motor

Implementar:

- conversão `KV ↔ Kt` com convenções explícitas;
- RPM ↔ rad/s;
- modelo DC equivalente versionado;
- back-EMF;
- corrente/torque;
- `P_mech = τ × ω`;
- eficiência quando os dados forem coerentes;
- limites e validações.

A aproximação de motor DC equivalente deve ser identificada como modelo, não como medição de um BLDC real.

## Etapa 7H — Modelo aerodinâmico da hélice

Primeiro modelo permitido: coeficientes adimensionais conhecidos.

```text
T = Ct × ρ × n² × D⁴
Q = Cq × ρ × n² × D⁵
P = Cp × ρ × n³ × D⁵
```

Entregas:

- regime estático;
- densidade do ar como contexto;
- `Ct/Cq/Cp` com origem e convenção;
- advance ratio `J` preparado para evolução;
- validação de unidades/coeficientes.

Se existirem apenas diâmetro, pitch e número de pás sem coeficientes/curva/geometria suficiente, retornar dados insuficientes ou baixa confiança conforme modelo explicitamente calibrado. Não inventar `Ct/Cq/Cp`.

## Etapa 7I — Solver do ponto de operação motor × hélice

Resolver o equilíbrio:

```text
τ_motor(ω, V, ...) = Q_propeller(ω, ρ, V_forward, ...)
```

Requisitos:

- intervalo físico limitado;
- solver determinístico;
- tolerância e limite de iterações explícitos;
- detecção de ausência de raiz;
- falha de convergência não produz resultado válido;
- método bracketed robusto quando aplicável;
- versão do solver registrada.

Saídas esperadas:

- RPM;
- corrente;
- torque;
- empuxo;
- potência elétrica;
- potência mecânica quando disponível;
- eficiência;
- proveniência/confiança.

## Etapa 7J — Validação e reconciliação bancada × modelo

- fixtures físicos versionados;
- comparar previsão com ensaios reais;
- erro absoluto/relativo por regime;
- não alterar dado medido para “encaixar” no modelo;
- permitir calibração versionada separada;
- reduzir confiança fora do domínio validado;
- selecionar automaticamente curva medida quando aplicável.

Critério de saída da Fase 7: o DroneCalc consegue usar dados reais de bancada como rota preferencial e, quando eles faltarem mas existirem parâmetros físicos suficientes, produzir uma estimativa teórica rastreável de bateria → motor → torque → hélice → ponto de operação → empuxo. Se nenhuma rota for defensável, retorna `dados insuficientes`.

---

# FASE 8 — Autonomia

- capacidade utilizável;
- carga auxiliar;
- corrente interpolada no hover quando houver bancada;
- corrente do Physics Engine quando a rota teórica for válida;
- tensão sob carga/sag quando o modelo estiver disponível;
- estimativa de autonomia;
- cenários adicionais apenas com modelo defensável;
- análise de sensibilidade.

Critério: toda autonomia mostra hipóteses, fonte, modelo e confiança.

---

# FASE 9 — Knowledge Base e heurísticas

## Etapa 9A — Evidence / Reference Knowledge

Armazenar conhecimento vindo de:

- fabricante;
- ensaio medido;
- usuário;
- artigo/vídeo/referência técnica;
- heurística curada.

Toda entrada carrega fonte, data, revisão e confiança.

## Etapa 9B — Sizing heuristics

Exemplo de uso:

```text
frame 150–165 mm
+ hélice ~4"
→ sugerir famílias de motor 14xx–16xx como candidatos
```

Essas regras nunca significam “compatível” ou “seguro”.

## Etapa 9C — Candidate Generator

Pipeline:

```text
intenção/restrições
→ heurísticas
→ candidatos de catálogo
→ filtros mecânicos/elétricos conhecidos
→ candidatos elegíveis
```

Critério: o sistema diferencia claramente `recommended by heuristic`, `compatible`, `modelled` e `validated by bench data`.

---

# FASE 10 — Perfis de voo e scoring

## Etapa 10A — Flight profile schema

Perfis MVP:

- Freestyle;
- Racing;
- Cinematic;
- Long Range;
- Mini Long Range;
- Cinewhoop.

## Etapa 10B — Wizard por objetivo

- estilo de voo;
- prioridade;
- payload;
- autonomia desejada;
- tamanho máximo;
- bateria preferida;
- nível do usuário.

## Etapa 10C — Scoring explicável

- métricas normalizadas;
- cobertura dos dados;
- penalties;
- confidence;
- incompatibilidade `danger` nunca mascarada por score.

## Etapa 10D — Caso obrigatório Mini Long Range

Validar que eficiência/autonomia perto de hover/cruzeiro podem superar empuxo máximo no ranking.

---

# FASE 11 — Builder completo

- layout técnico NEXO;
- component picker;
- busca no catálogo;
- overrides;
- resumo sticky;
- análise progressiva;
- warnings;
- estados incompletos.

---

# FASE 12 — Comparador

- variantes;
- deltas;
- warnings;
- score por perfil;
- explicação de trade-offs;
- distinguir resultados medidos, interpolados e modelados.

---

# FASE 13 — Centro de gravidade

- Position3D;
- CG x/y/z;
- visualização 2D;
- cálculo parcial explicitamente marcado.

---

# FASE 14 — Otimizador

Somente após catálogo, propulsão, autonomia, heurísticas e perfis estabilizados.

```text
restrições
→ candidate generator
→ incompatibilidades eliminatórias
→ cálculo técnico
→ métricas normalizadas
→ score multiobjetivo
→ ranking explicável
```

Requisitos:

- `danger` conhecido elimina candidato;
- dados críticos ausentes reduzem elegibilidade/confiança;
- mostrar trade-offs e não apenas maior score;
- resultado reproduzível por versões de perfil/motor;
- candidatos baseados apenas em modelo teórico devem ser distinguidos daqueles validados por bancada.

---

# FASE 15 — Busca semântica e pgvector

Somente quando houver volume de evidências que justifique.

Possíveis usos:

- pesquisa em datasheets e fontes;
- recuperação de evidências para Ollama;
- componentes semelhantes;
- RAG técnico com referências.

PostgreSQL + pgvector deve ser tentado antes de introduzir banco vetorial separado.

---

# FASE 16 — Relatórios, operação e evolução

- relatório técnico;
- export de evidências;
- observabilidade;
- backups;
- políticas de retenção;
- performance de consultas;
- eventual Tauri;
- eventual sincronização/contas.

Não chamar o relatório de certificação ou homologação.

---

## Dependências principais

```text
1A Bootstrap workspace
 ├─ 1B NEXO
 ├─ 1C Docker services
 └─ 1D Units/contracts
       ↓
2 PostgreSQL/Object Storage
       ↓
3 Domain/Catalog/Projects
 ├─────────────┐
 ↓             ↓
4 Basic       5 URL ingestion
Engine          ↓
 ↓            6 Ollama extraction
7A–7D Bench      │
 └──────┐        │
        ↓        │
7E–7J Physics Engine
        ↓        │
8 Endurance     9 Knowledge/Heuristics
       └─────────┬─────────┘
                 ↓
          10 Flight Profiles
                 ↓
            11/12 UI flows
                 ↓
            14 Optimizer
                 ↓
          15 Semantic Search
```

## Definition of Done por etapa

Toda etapa deve:

- compilar;
- passar typecheck, lint, testes e build aplicáveis;
- possuir testes dos novos contratos;
- preservar comportamento não relacionado;
- incluir migration quando schema persistido mudar;
- atualizar documentação afetada;
- não inserir fórmulas na UI;
- manter identidade NEXO;
- não armazenar segredos no repositório;
- registrar incerteza/proveniência dos resultados;
- versionar modelos físicos que alterem resultados;
- validar invariantes de unidades, potência, torque e eficiência quando aplicáveis;
- fazer auto-auditoria de regressões e segurança.

## Regra para melhorias propostas por IA

A IA implementadora pode substituir a estrutura, biblioteca, solver ou abordagem indicada neste roadmap se identificar solução objetivamente melhor, desde que:

1. explique a limitação da abordagem original;
2. compare trade-offs;
3. preserve os requisitos e fronteiras arquiteturais;
4. não amplie escopo sem necessidade;
5. verifique documentação oficial das dependências adotadas;
6. implemente testes equivalentes ou melhores;
7. atualize docs/ADR quando a decisão for estrutural;
8. registre a alteração no relatório da etapa;
9. para modelos físicos, valide numericamente contra casos de referência e, quando disponível, bancada real.

## Prioridade imediata

Próxima implementação: **Etapa 1A — Bootstrap full-stack e workspace**.

A orientação executável está em:

`docs/prompts/ETAPA_1A_BOOTSTRAP_FULLSTACK.md`

A implementação futura de torque/propulsão avançada deve seguir:

`docs/PHYSICS_ENGINE.md`
