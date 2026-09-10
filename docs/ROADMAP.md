# Roadmap — DroneCalc

**Versão:** 0.1  
**Estratégia:** construir o núcleo técnico antes de ampliar a interface e o catálogo.

## Princípios de execução

- etapas pequenas e testáveis;
- evitar fórmulas na UI;
- cada fase atualiza documentação quando muda contrato;
- não iniciar otimização antes de validar o motor básico;
- preferir dados reais de bancada a heurísticas complexas;
- manter o projeto executável ao final de cada etapa.

## Etapa 0 — Especificação

**Status:** documentação-base criada.

Entregas:

- PRD;
- Product Spec;
- arquitetura;
- modelo de domínio;
- motor de cálculo;
- perfis de voo;
- identidade NEXO;
- UX;
- dados/catálogo;
- testes;
- segurança;
- padrões de desenvolvimento;
- guia para IA.

Critério de saída: escopo e contratos suficientes para iniciar bootstrap sem decisões críticas implícitas.

---

## Etapa 1A — Bootstrap da aplicação

Objetivo: criar projeto React + TypeScript + Vite com qualidade mínima.

Entregas:

- TypeScript strict;
- lint/format;
- Vitest;
- estrutura de diretórios;
- aliases;
- scripts `dev`, `build`, `test`, `typecheck`, `lint`;
- CI inicial;
- error boundary básico.

Critério: build e testes verdes.

## Etapa 1B — NEXO Design System base

Entregas:

- tokens NEXO sincronizados;
- temas light/dark/system;
- Button, Input, Select, Badge, Card, Alert;
- shell desktop inicial;
- documentação da revisão NEXO usada.

Critério: nenhuma feature usa cor hardcoded.

## Etapa 1C — Unidades e validação física

Entregas:

- tipos/brands de unidades;
- conversões massa/comprimento/capacidade;
- parser pt-BR;
- validações de números físicos;
- formatters.

Critério: suite de conversões e casos inválidos completa.

---

## Etapa 2 — Domínio e projetos

### 2A — Modelos de componentes

Implementar tipos do catálogo e schemas Zod.

### 2B — DroneProject

- projeto;
- instâncias de componente;
- constraints;
- assumptions;
- overrides.

### 2C — Persistência local

- repositories;
- IndexedDB;
- migrations;
- autosave;
- settings.

### 2D — Import/export inicial

- JSON versionado;
- validação;
- round-trip test.

Critério da etapa: criar/salvar/reabrir/duplicar um projeto vazio ou parcialmente montado.

---

## Etapa 3 — Massa

### 3A — Mass engine

- linhas;
- categorias;
- dry mass;
- battery mass;
- payload;
- takeoff mass;
- dados ausentes.

### 3B — UI de massa

- breakdown;
- componentes sem massa;
- percentuais;
- métricas.

Critério: fixtures manuais e automatizadas coincidem.

---

## Etapa 4 — Bateria e sistema elétrico básico

### 4A — Battery engine

- tensão nominal;
- tensão cheia;
- Ah;
- Wh;
- C-rating derivado;
- metadata de química.

### 4B — Electrical compatibility

- bateria × ESC;
- bateria × motor;
- ESC × motor;
- cargas auxiliares iniciais;
- warnings estáveis.

### 4C — UI de energia

- painéis;
- margens;
- proveniência.

Critério: incompatibilidades conhecidas são detectadas com testes de limite.

---

## Etapa 5 — Catálogo

### 5A — CRUD local de componentes

- listar;
- buscar;
- criar custom;
- duplicar;
- proteger referências.

### 5B — Qualidade de dados

- completeness;
- source metadata;
- revisão.

### 5C — Seeds de teste

Fixtures confiáveis, não catálogo massivo.

Critério: Builder pode selecionar peças do catálogo ou criar custom.

---

## Etapa 6 — Propulsão e curvas

### 6A — Bench data model

- motor+hélíce+tensão;
- samples;
- validação.

### 6B — Importador CSV

- mapping;
- unidades;
- preview;
- validation.

### 6C — Interpolação

- lookup por empuxo;
- current/power/throttle;
- sem extrapolação.

### 6D — Propulsion analysis

- empuxo total;
- TWR;
- hover thrust por motor;
- eficiência g/W.

### 6E — Curva visual

- pontos medidos;
- trecho interpolado;
- hover marker.

Critério: uma configuração com curva real/fixture produz análise de propulsão rastreável.

---

## Etapa 7 — Autonomia

### 7A — Hover endurance

- capacidade utilizável;
- carga auxiliar;
- corrente interpolada;
- estimate + confidence.

### 7B — Cenários

Adicionar cruzeiro/performance somente com modelo defensável.

### 7C — Sensibilidade

Futuro da etapa: mostrar impacto de capacidade/massa sem chamar isso de otimização global.

Critério: resultado de autonomia mostra todas as hipóteses.

---

## Etapa 8 — Perfis de voo

### 8A — Flight profile schema

- seis perfis do MVP;
- versões;
- priorities.

### 8B — Wizard

- escolher estilo;
- restrições;
- metas derivadas.

### 8C — Scoring preliminar

- normalização;
- cobertura de dados;
- penalties;
- explicabilidade.

### 8D — Mini Long Range

Caso de validação obrigatório demonstrando que eficiência/autonomia podem superar potência máxima no ranking.

Critério: score não esconde warnings e explica contribuições.

---

## Etapa 9 — Builder completo

- layout 3 painéis;
- component picker;
- overrides;
- resumo sticky;
- análise progressiva;
- empty states;
- atalhos de navegação.

Critério: usuário monta um quad completo sem editar JSON.

---

## Etapa 10 — Comparador

- duas ou mais variantes;
- delta de métricas;
- warnings;
- score por perfil;
- duplicar para variante.

Critério: trocar bateria/motor em variante e visualizar impacto com clareza.

---

## Etapa 11 — Centro de gravidade

- Position3D;
- CG x/y/z;
- representação 2D inicial;
- componentes sem posição;
- referência do frame.

Critério: fixture simétrico retorna centro e fixture assimétrico retorna deslocamento esperado.

---

## Etapa 12 — Relatórios e compartilhamento local

- export detalhado;
- relatório técnico;
- resumo de peças;
- warnings;
- fórmulas/model versions.

Evitar prometer “certificação”.

---

## Etapa 13 — Otimizador

Somente após motor e perfis estabilizados.

Entradas:

- perfil;
- restrições;
- catálogo elegível.

Pipeline:

```text
candidatos
→ filtros de incompatibilidade
→ cálculos
→ métricas normalizadas
→ score multiobjetivo
→ ranking explicável
```

Requisitos:

- `danger` conhecido elimina candidato;
- dados críticos ausentes reduzem elegibilidade/confiança;
- mostrar trade-offs;
- não buscar apenas maior score sem explicar.

---

## Etapa 14 — Catálogo ampliado

- fontes/revisões;
- importadores;
- deduplicação;
- testes comunitários futuros;
- possível backend.

Não coletar grande volume antes de definir governança de qualidade.

---

## Etapa 15 — Desktop / sincronização opcional

Avaliar Tauri e backend somente se houver necessidade.

Possíveis capacidades:

- armazenamento SQLite;
- backup/sync;
- contas;
- compartilhamento;
- compute de otimização.

---

## Dependências principais

```text
Bootstrap
  ├─ Design System
  └─ Units/Domain
       ├─ Mass
       ├─ Battery/Electrical
       └─ Catalog
            └─ Bench Curves
                 ├─ Propulsion
                 └─ Endurance
                      └─ Flight Profiles/Scoring
                           └─ Optimizer
```

## Definition of Done por etapa

Toda etapa deve:

- compilar;
- passar typecheck/lint/test;
- ter testes dos novos contratos;
- preservar dados existentes ou incluir migração;
- atualizar docs afetadas;
- não inserir fórmula na UI;
- manter identidade NEXO;
- registrar incerteza dos resultados.

## Prioridade imediata

Próximo trabalho de implementação: **Etapa 1A — Bootstrap**, seguida de **1B Design System** e **1C Unidades**.
