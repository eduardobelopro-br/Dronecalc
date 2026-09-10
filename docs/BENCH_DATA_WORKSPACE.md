# Workspace de Bancada — Curvas Motor, Hélice e Propulsão

**Versão:** 0.1  
**Status:** especificação para implementação futura  
**Escopo:** cadastro, inspeção, validação e visualização de curvas de bancada de motor + hélice + tensão/condições.

## 1. Objetivo

O DroneCalc deve possuir uma seção própria **Bancada** para trabalhar com dados experimentais de propulsão. Essa seção não é um simulador de velocidade de voo: ela organiza e valida medições/declarações de bancada, produz apenas derivações matemáticas defensáveis e fornece dados rastreáveis ao Calculation Engine e ao Physics Engine.

A seção deve permitir:

- cadastrar um ensaio manualmente;
- importar CSV/tabela quando disponível;
- receber dados extraídos de página, imagem ou documento pelo pipeline de ingestão assistida;
- revisar cada amostra e sua proveniência;
- visualizar curvas sem misturar grandezas incompatíveis de forma enganosa;
- calcular potência elétrica e eficiência estática de empuxo quando as entradas existirem;
- calcular **velocidade teórica de passo** da hélice somente como métrica educacional/derivada;
- interpolar apenas dentro da faixa de dados válida;
- disponibilizar a curva aprovada para análise de hover, TWR, corrente e autonomia.

## 2. Posição na aplicação

A navegação principal deve incluir:

```text
Projetos
Builder
Análise
Comparador
Catálogo
Bancada
Configurações
```

A seção também deve ser acessível a partir de:

- detalhe de um motor → `Ver testes de bancada`;
- detalhe de uma hélice → `Ver testes de bancada`;
- análise de propulsão → `Abrir curva usada`;
- cadastro/importação → `Revisar curva extraída`.

`Bancada` é um workspace técnico; não deve duplicar o cadastro de componentes do Catálogo.

## 3. Entidade de ensaio

Um ensaio representa uma combinação identificável de motor, hélice e condições.

Contrato conceitual:

```ts
interface PropulsionBenchTest {
  id: string
  motorVariantId: string
  propellerVariantId?: string

  propellerDescription?: {
    diameterIn?: number
    pitchIn?: number
    bladeCount?: number
    model?: string
  }

  escId?: string

  conditions: {
    batteryChemistry?: string
    seriesCells?: number
    supplyVoltageV?: number
    ambientTemperatureC?: number
    airDensityKgM3?: number
    testType: 'static' | 'other'
  }

  samples: PropulsionBenchSample[]
  source: SourceReference
  reviewStatus: 'draft' | 'needs_review' | 'approved' | 'rejected'
}
```

O schema final pode ser normalizado de forma diferente no PostgreSQL. A implementação não deve acoplar o calculation engine ao schema SQL.

## 4. Amostra de bancada

Campos canônicos desejáveis:

```ts
interface PropulsionBenchSample {
  throttlePercent?: number
  voltageV?: number
  currentA?: number
  thrustN?: number
  rpm?: number

  measuredPowerW?: number

  derived?: {
    electricalPowerW?: CalculationResult<number>
    staticThrustEfficiencyGfPerW?: CalculationResult<number>
    theoreticalPitchSpeedKmh?: CalculationResult<number>
  }
}
```

A UI pode mostrar empuxo em `gf/g` por conveniência, mas a força interna deve permanecer inequívoca. Se uma fonte usar coluna ambígua como `Empuxo (100g)`, o valor bruto deve ser preservado e o mapeamento de unidade exige revisão explícita.

## 5. Grandezas obrigatoriamente distintas

O DroneCalc deve distinguir semanticamente:

- `throttlePercent` — comando relativo ao ESC; não é aceleração física;
- `currentA` — corrente elétrica;
- `voltageV` — tensão medida/condição do ponto;
- `electricalPowerW` — potência elétrica;
- `thrust` — força de empuxo;
- `rpm` — rotação carregada medida/declarada;
- `staticThrustEfficiencyGfPerW` — razão de empuxo estático por potência;
- `theoreticalPitchSpeedKmh` — velocidade geométrica ideal de passo;
- velocidade real do drone — grandeza diferente e **não derivável apenas de pitch × RPM**.

A palavra genérica `Eficiência` sem unidade não deve ser persistida como métrica final normalizada.

## 6. Potência elétrica por amostra

Quando tensão e corrente da mesma amostra estiverem disponíveis:

```text
P = V × I
```

Usar `POWER_DC_V1` ou formula ID equivalente versionado.

Regras:

- preferir tensão medida no ponto;
- não assumir tensão nominal constante se a curva não declarar isso;
- se houver potência declarada/medida e potência derivada, preservar ambas;
- divergência além de tolerância configurada gera warning, não sobrescrita automática.

## 7. Eficiência estática de empuxo

Quando empuxo e potência elétrica da mesma amostra existirem:

```text
static_thrust_efficiency_gf_per_W = thrust_gf / electrical_power_W
```

Formula ID sugerido:

```text
STATIC_THRUST_EFFICIENCY_V1
```

Essa métrica:

- deve exibir unidade `gf/W` ou `g/W` de forma consistente na UI;
- não é eficiência energética/termodinâmica percentual;
- é útil para comparar pontos de uma mesma família de ensaios sob condições semelhantes;
- não deve ser apresentada como `509%`, `341%` etc.;
- deve informar origem `calculated` quando derivada.

## 8. Velocidade teórica de passo

Quando pitch geométrico e RPM estiverem disponíveis, o DroneCalc pode oferecer uma métrica educacional:

```text
pitch_m = pitch_in × 0.0254
pitch_speed_m_per_min = pitch_m × RPM
pitch_speed_km_per_h = pitch_speed_m_per_min × 60 / 1000
```

Formula ID sugerido:

```text
PROPELLER_THEORETICAL_PITCH_SPEED_V1
```

Nome obrigatório de apresentação:

**Velocidade teórica de passo**

Nunca usar apenas `Velocidade`, `Velocidade do drone` ou `Velocidade máxima`.

A UI deve apresentar aviso permanente/contextual:

> Velocidade teórica de passo é uma relação geométrica ideal entre pitch e RPM. Não representa a velocidade real da aeronave e ignora slip, arrasto, advance ratio e demais efeitos aerodinâmicos.

Classificação sugerida:

```text
source = calculated
confidence = low
```

para uso como aproximação educacional, mesmo quando RPM foi medido.

## 9. RPM ideal sem carga

Quando KV e tensão forem conhecidos:

```text
RPM_no_load_ideal ≈ KV × V
```

Esse valor é somente referência teórica e não substitui RPM carregado.

A UI deve usar nome explícito:

**RPM ideal teórico sem carga**

Se existir RPM real de bancada no mesmo ponto, a aplicação pode mostrar a razão:

```text
loaded_rpm_ratio = measured_loaded_rpm / ideal_no_load_rpm
```

sem interpretar automaticamente essa razão como eficiência do motor.

## 10. Tensão por ponto

A tensão deve ser tratada como dado de primeira classe.

Para curvas importadas sem tensão por amostra:

- aceitar `supplyVoltageV` global somente se a fonte realmente declarar tensão controlada/constante;
- se apenas número de células estiver disponível, não inventar uma tensão medida;
- potência e `gf/W` derivadas ficam indisponíveis quando a tensão necessária não for defensável;
- emitir `BENCH_VOLTAGE_MISSING` quando isso impedir cálculos relevantes.

Isso evita ocultar voltage sag e evita inferência falsa de potência.

## 11. Validações cruzadas

Warnings/códigos sugeridos:

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

Regras importantes:

- corrente negativa é inválida;
- tensão <= 0 é inválida para amostra energizada;
- RPM negativo é inválido;
- empuxo negativo é inválido no modelo estático convencional;
- throttle fora de 0–100% exige contexto/modelo explícito;
- `P_measured` versus `V × I` divergente gera warning com ambos os valores;
- RPM carregado acima da referência ideal `KV × V` é suspeito, considerando tolerância/instrumentação e natureza aproximada do próprio KV;
- comportamento não monotônico não é automaticamente inválido: pode existir ruído, mudança de regime ou erro de medição e deve ser revisado.

## 12. Interpolação

O workspace pode solicitar ao calculation engine pontos interpolados para análise.

Regras:

- interpolar somente dentro do domínio medido/aprovado;
- preservar os pontos originais;
- não interpolar sobre amostras inválidas/rejeitadas;
- não extrapolar silenciosamente;
- indicar claramente o ponto interpolado e os dois pontos de origem;
- a variável-alvo preferencial para hover é empuxo requerido, não throttle.

## 13. Interface — cabeçalho do ensaio

Campos/controles esperados:

```text
Motor                 [ ... ]
Hélice                 [ ... ]
Bateria / fonte        [ ... ]
ESC                    [ ... ]
Tipo de ensaio         [ Estático ]
Fonte / evidência      [ ... ]
Status                 [ Draft / Review / Approved ]

[Importar CSV] [Importar fonte] [Adicionar amostra]
```

A UI deve mostrar condições ausentes relevantes sem inventar defaults.

## 14. Interface — tabela

Colunas recomendadas:

```text
Throttle (%)
Tensão (V)
Corrente (A)
Potência (W)
Empuxo (gf ou N)
RPM
Eficiência estática (gf/W)
Pitch speed teórica (km/h) [opcional]
Fonte/status
```

Cada célula derivada deve ser visualmente distinguível de valor medido/declarado e permitir `Como foi calculado?`.

Quando uma coluna não possuir dados suficientes, mostrar `— / dados insuficientes`, nunca `0` por conveniência.

## 15. Interface — gráficos

Não usar, por padrão, uma única escala Y para grandezas de unidades distintas como A, gf, RPM e gf/W.

Visualizações recomendadas separadamente:

1. **Empuxo × throttle**;
2. **Corrente e potência × throttle** — com escalas explícitas ou gráficos separados quando necessário;
3. **RPM × throttle**;
4. **Eficiência estática (gf/W) × throttle ou empuxo**;
5. comparação opcional de duas curvas compatíveis.

A tabela é a fonte acessível e auditável; o gráfico é visualização secundária.

Gráficos devem:

- exibir unidade nos eixos;
- permitir identificar pontos medidos versus interpolados;
- não conectar lacunas inválidas como se fossem medições;
- possuir resumo textual/tabela acessível;
- evitar gráfico multiunidade que sugira comparabilidade numérica falsa.

## 16. Importação

Fontes aceitas incrementalmente:

```text
manual
CSV
HTML/tabela do fabricante
PDF/datasheet
imagem técnica
curva digitalizada de gráfico (futuro/revisado)
```

Prioridade:

- tabela/CSV original > digitalização visual;
- dado medido com condições reproduzíveis > valor inferido;
- valor extraído por IA entra em staging/review conforme `ASSISTED_INGESTION.md`.

## 17. Proveniência

Cada amostra/campo deve poder indicar:

- fonte;
- URL/asset/documento;
- método de extração;
- medido/declarado/calculado/interpolado/digitalizado;
- data/revisão;
- confiança;
- status de revisão;
- formula ID para valores derivados.

A aplicação deve permitir responder: **“de onde veio este número?”**.

## 18. Integração com Análise

Uma curva só pode alimentar resultados normais de análise quando estiver válida para a combinação e condições necessárias.

Fluxo:

```text
massa do drone
→ empuxo requerido por motor
→ localizar curva compatível
→ procurar/interpolar dentro do domínio aprovado
→ corrente + tensão + potência + RPM
→ TWR / hover / eficiência / autonomia
```

Se o empuxo alvo estiver fora do domínio:

```text
Dados insuficientes — empuxo requerido fora da faixa ensaiada.
```

Não extrapolar por padrão.

## 19. Persistência

PostgreSQL deve ser a fonte autoritativa para ensaios publicados/aprovados.

Separar semanticamente:

```text
bench_tests
bench_test_samples
source/evidence
review status
calculated/derived values quando persistidos por necessidade de cache/auditoria
```

Valores derivados podem preferencialmente ser recalculados a partir de entradas + `formulaId/modelVersion`, evitando duplicação autoritativa desnecessária.

## 20. Exemplo de leitura correta de uma planilha

Uma tabela com colunas como:

```text
Aceleração | Corrente | Empuxo (100g) | Eficiência | RPM
```

não deve ser importada cegamente.

O importador/revisor deve solicitar ou inferir somente com evidência:

- `Aceleração` provavelmente mapeia para `throttlePercent`;
- `Empuxo (100g)` é unidade/escala ambígua e exige confirmação;
- `Eficiência` exige unidade; valores compatíveis com `gf/W` devem continuar candidatos até validação;
- tensão ausente impede reconstrução segura de potência por ponto;
- cálculos de `pitch × RPM` são classificados como pitch speed teórica, não velocidade real do drone.

## 21. Segurança operacional

Dados de bancada não autorizam execução física do ensaio pelo software.

Qualquer documentação sobre testes reais deve lembrar:

- remover hélices para configuração/calibração que não exija hélice;
- quando ensaio de hélice for realmente necessário, usar bancada de empuxo protegida e procedimento específico;
- manter pessoas fora do plano da hélice;
- dimensionar ESC, fonte/bateria, cabos e conectores para a corrente esperada;
- não tratar limite de fabricante como garantia absoluta de operação segura.

## 22. Critérios de aceite

A seção Bancada só está concluída quando:

- ensaio motor + hélice + condições pode ser criado e salvo;
- amostras podem ser adicionadas/editadas com unidades explícitas;
- CSV válido pode ser importado com preview/mapeamento;
- tensão ausente não vira nominal silenciosamente;
- `P = V × I` pode ser derivado e auditado;
- `gf/W` é rotulado como eficiência estática de empuxo, não percentual;
- pitch speed é nomeado e alertado como teórico, nunca velocidade real do drone;
- gráficos não misturam unidades em uma escala única por padrão;
- pontos interpolados são identificáveis e nunca extrapolados silenciosamente;
- cada dado possui proveniência/status apropriado;
- curva aprovada pode ser utilizada pelo cálculo de hover/propulsão;
- testes cobrem fórmulas, unidades, limites, importação e warnings.

## 23. Liberdade da IA implementadora

A IA pode propor estrutura de componentes, biblioteca de gráficos, modelo de estado, schema de formulário ou estratégia de importação mais eficaz/eficiente, desde que:

1. preserve as semânticas físicas e unidades deste documento;
2. preserve a separação entre medido/declarado/calculado/interpolado;
3. não transforme pitch speed em velocidade de aeronave;
4. não extrapole curvas por padrão;
5. mantenha calculation engine independente da UI;
6. use tokens NEXO e acessibilidade;
7. implemente testes equivalentes ou melhores;
8. justifique trade-offs e atualize docs/ADR se alterar arquitetura.