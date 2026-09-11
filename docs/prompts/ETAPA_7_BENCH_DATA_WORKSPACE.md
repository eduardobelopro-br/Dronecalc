# Prompt de Implementação — Workspace de Bancada

> Usar quando a fundação da aplicação, domínio, persistência e contratos de unidades necessários já existirem. Não executar prematuramente se as dependências do roadmap ainda não estiverem concluídas.

Você é a IA implementadora do DroneCalc. Antes de alterar código, leia obrigatoriamente:

- `docs/PRD.md`;
- `docs/PRODUCT_SPEC.md`;
- `docs/ARCHITECTURE.md`;
- `docs/DOMAIN_MODEL.md`;
- `docs/CALCULATION_ENGINE.md`;
- `docs/PHYSICS_ENGINE.md`;
- `docs/BENCH_DATA_WORKSPACE.md`;
- `docs/COMPONENT_CATALOG.md`;
- `docs/DATA_MODEL.md`;
- `docs/TEST_STRATEGY.md`;
- `docs/NEXO_DESIGN_SYSTEM.md`;
- `docs/UX_SPEC.md`;
- `docs/ROADMAP.md`.

## Objetivo

Implementar a seção **Bancada** para cadastro, revisão e visualização de ensaios motor + hélice + condições, sem misturar regras físicas na UI e sem apresentar pitch speed teórica como velocidade real da aeronave.

## Processo obrigatório

1. inspecione o código real e identifique as fronteiras existentes;
2. liste riscos e dependências antes de programar;
3. proponha a menor mudança coerente;
4. explique trade-offs relevantes;
5. implemente;
6. adicione/atualize testes;
7. execute typecheck, lint, unit/integration tests e build aplicáveis;
8. faça auto-auditoria de regressões, unidades, persistência e semântica física;
9. gere relatório da etapa.

Não crie múltiplos agentes sem autorização expressa.

## Requisitos funcionais mínimos

A seção deve permitir:

- criar/editar um `PropulsionBenchTest` ou contrato equivalente;
- selecionar motor/variante e hélice/variante;
- registrar diâmetro, pitch e número de pás quando a hélice não estiver catalogada integralmente;
- registrar células/química/tensão/ESC/condições quando disponíveis;
- adicionar, editar e remover amostras;
- importar CSV com preview e mapeamento de colunas/unidades;
- mostrar tabela de amostras;
- calcular potência `P = V × I` apenas quando tensão e corrente forem defensáveis;
- calcular eficiência estática `gf/W` apenas quando empuxo e potência forem defensáveis;
- calcular pitch speed teórica opcionalmente e mostrar aviso explícito de que não é velocidade real do drone;
- mostrar gráficos separados/adequados por unidade;
- mostrar proveniência e status de cada campo/ensaio;
- permitir abrir `Como foi calculado?` para valores derivados;
- rejeitar extrapolação silenciosa;
- permitir que curva aprovada seja consumida pelo calculation engine.

## Requisitos de domínio

Não usar strings soltas ou números sem unidade para contratos físicos críticos.

Preferir contratos equivalentes a:

```ts
interface PropulsionBenchSample {
  throttlePercent?: number
  voltageV?: number
  currentA?: number
  thrustN?: number
  rpm?: number
  measuredPowerW?: number
}
```

Valores derivados devem retornar `CalculationResult` ou contrato equivalente com:

```ts
value
unit
source
confidence
formulaId/modelVersion
inputs/provenance
notes
```

## Fórmulas permitidas nesta etapa

### Potência DC

```text
P = V × I
```

### Eficiência estática de empuxo

```text
static_thrust_efficiency_gf_per_W = thrust_gf / power_W
```

Não chamar isso de eficiência energética percentual.

### Pitch speed teórica

```text
pitch_m = pitch_in × 0.0254
m_per_min = pitch_m × RPM
km_per_h = m_per_min × 60 / 1000
```

Nome obrigatório de UI: **Velocidade teórica de passo**.

Adicionar aviso equivalente a:

`Não representa a velocidade real da aeronave; ignora slip, arrasto, advance ratio e demais efeitos aerodinâmicos.`

### RPM ideal sem carga

Se já houver função centralizada para `KV × V`, reutilize-a. Não duplique fórmula na UI.

## Validações obrigatórias

Cobrir ao menos:

```text
BENCH_VOLTAGE_MISSING
BENCH_POWER_INCONSISTENT
BENCH_EFFICIENCY_UNIT_AMBIGUOUS
BENCH_THRUST_UNIT_AMBIGUOUS
BENCH_RPM_EXCEEDS_NO_LOAD_REFERENCE
BENCH_SAMPLE_DUPLICATE
BENCH_SAMPLE_INVALID
BENCH_INTERPOLATION_OUT_OF_RANGE
PITCH_SPEED_NOT_AIRCRAFT_SPEED
```

Os nomes podem ser ajustados se já existir convenção melhor no código, preservando a semântica e documentando a alteração.

## Gráficos

Não colocar corrente, empuxo, RPM e `gf/W` numa única escala Y como se fossem numericamente comparáveis.

Preferir:

- Empuxo × throttle;
- Corrente/Potência × throttle com eixos claramente identificados ou gráficos separados;
- RPM × throttle;
- Eficiência estática × throttle ou empuxo.

A tabela deve continuar sendo a representação auditável/acessível dos dados.

## Importação CSV

O importador deve:

- suportar vírgula/ponto decimal conforme estratégia do projeto;
- permitir mapear cabeçalhos desconhecidos;
- exigir/confirmar unidade quando ambígua;
- preservar valor bruto;
- não interpretar `Empuxo (100g)` automaticamente sem regra/evidência clara;
- não interpretar `Eficiência` sem unidade automaticamente como percentual ou `gf/W`;
- permitir cancelar antes de persistir;
- não deixar persistência parcial se a validação final falhar.

## Persistência

Respeitar a arquitetura PostgreSQL/repositories já adotada quando essa camada existir.

Não acoplar o calculation engine ao ORM/schema SQL. Não persistir resultados derivados como fonte autoritativa se puderem ser reproduzidos com entradas + versão de fórmula, salvo decisão arquitetural documentada.

## NEXO

Usar exclusivamente tokens/componentes do NEXO Design System disponíveis no projeto. Não introduzir hex colors hardcoded na feature.

Estados de warning/danger/info devem ter texto/ícone e não depender apenas de cor.

## Testes mínimos

Adicionar testes para:

- `V × I`;
- `gf/W`;
- conversão de pitch e pitch speed;
- pitch speed não rotulado como velocidade real;
- tensão ausente;
- divisão por zero;
- unidades inválidas;
- potência medida divergente de `V × I`;
- importação CSV válida/inválida;
- cabeçalho/unidade ambígua;
- amostras duplicadas;
- interpolação dentro da faixa;
- alvo fora da faixa;
- ausência de extrapolação;
- acessibilidade básica da tabela/gráficos;
- persistência/reload quando aplicável.

## Liberdade para melhorar

Você **pode modificar a abordagem proposta** se encontrar solução mais eficaz, eficiente, simples, segura ou testável. Isso inclui estrutura de módulos, biblioteca de gráficos, modelagem de formulário, estratégia de parsing CSV, nomes de contratos e distribuição entre frontend/backend.

Porém, antes de mudar uma decisão relevante:

1. explique a limitação da proposta atual;
2. compare a alternativa;
3. preserve as regras físicas e de proveniência;
4. não transforme pitch speed em velocidade real;
5. não introduza extrapolação silenciosa;
6. não mova fórmulas para componentes React;
7. não enfraqueça validações;
8. consulte documentação oficial das dependências adotadas;
9. implemente testes equivalentes ou melhores;
10. atualize documentação/ADR quando estrutural.

## Fora de escopo

Não implementar nesta etapa, salvo se já existir como dependência concluída e for necessário integrar:

- CFD;
- cálculo real de velocidade máxima do drone;
- modelo completo de drag do frame;
- identificação automática de `Ct/Cq/Cp` sem fonte;
- extrapolação de curva;
- execução automatizada de bancada física;
- scraping/Ollama se as etapas correspondentes ainda não estiverem implementadas.

## Entrega esperada

Ao finalizar, gerar relatório com:

- arquivos alterados;
- arquitetura adotada;
- fórmulas/IDs usados;
- testes executados e resultados;
- warnings implementados;
- limitações restantes;
- divergências da especificação e justificativas;
- riscos/regressões encontrados na auto-auditoria.