# Physics Engine — Bateria, Motor, Torque, Hélice e Empuxo

**Versão:** 0.1  
**Status:** especificação para implementação futura  
**Escopo:** modelo físico avançado complementar ao motor baseado em dados de bancada.

## 1. Objetivo

O DroneCalc deve ser capaz de analisar um conjunto propulsivo por duas rotas complementares:

1. **rota empírica**, baseada em curvas de bancada reais de motor + hélice + tensão/condições;
2. **rota teórica**, baseada em modelos elétricos, eletromecânicos e aerodinâmicos quando existirem parâmetros suficientes.

A rota teórica não substitui dados medidos. Sempre que houver ensaio aplicável e confiável, ele possui prioridade para o ponto de operação correspondente.

O objetivo deste documento é impedir que a implementação tente inferir empuxo real apenas de `KV + diâmetro da hélice + tensão`.

## 2. Hierarquia de evidência

Para uma grandeza como empuxo, corrente, potência, RPM ou eficiência, a ordem preferencial é:

```text
ensaio medido aplicável
        ↓
dado de fabricante com condições reproduzíveis
        ↓
interpolação dentro de curva conhecida
        ↓
modelo físico calibrado/coeficientes conhecidos
        ↓
modelo físico genérico com hipóteses explícitas
        ↓
heurística de dimensionamento
```

Uma heurística pode gerar candidatos, mas não deve ser apresentada como validação física.

## 3. Arquitetura conceitual

```text
Battery Model
     ↓
ESC / Drive Model
     ↓
Motor Electrical Model
     ↓
Motor Torque Model
     ↓
Propeller Load Model
     ↓
Operating-Point Solver
     ↓
RPM + Current + Torque
     ↓
Thrust + Power + Efficiency
     ↓
Aircraft Performance
```

Implementação sugerida dentro de `packages/calculation-engine`:

```text
physics/
├── battery/
├── esc/
├── motor/
├── propeller/
├── atmosphere/
├── operating-point/
├── propulsion/
└── validation/
```

A IA implementadora pode alterar essa organização se encontrar uma solução mais simples ou robusta, desde que preserve as fronteiras, os contratos, a testabilidade e a independência de UI/persistência.

## 4. Unidades

Os modelos físicos devem trabalhar internamente em SI:

- tensão: V;
- corrente: A;
- resistência: Ω;
- velocidade angular: rad/s;
- rotação: rev/s ou RPM somente com conversão explícita;
- torque: N·m;
- potência: W;
- força/empuxo: N;
- diâmetro: m;
- densidade do ar: kg/m³;
- temperatura absoluta: K quando usada em equações termodinâmicas.

A UI pode exibir g, gf, mm, polegadas, mAh etc., mas conversões não devem contaminar as equações internas.

## 5. Modelo de bateria

### 5.1 Nível básico

Mantém os cálculos já definidos em `CALCULATION_ENGINE.md`:

```text
V_nominal = S × V_cell_nominal
capacity_Ah = capacity_mAh / 1000
energy_Wh = V_nominal × capacity_Ah
```

### 5.2 Modelo sob carga

Quando houver parâmetros suficientes, utilizar um modelo equivalente de Thévenin inicial:

```text
V_load ≈ V_oc(SOC, T) - I × R_internal(SOC, T)
```

Onde:

- `V_oc`: tensão de circuito aberto;
- `SOC`: estado de carga;
- `T`: temperatura;
- `R_internal`: resistência interna equivalente do pack;
- `I`: corrente do pack.

O primeiro modelo pode assumir `R_internal` constante somente quando essa hipótese estiver explícita. Modelos futuros podem usar curvas por SOC e temperatura.

### 5.3 Regras

- C-rating não é resistência interna;
- tensão nominal não representa tensão sob carga;
- resistência por célula e resistência do pack não podem ser confundidas;
- ligações série/paralelo devem ser consideradas explicitamente;
- dados ausentes não recebem valores típicos silenciosos;
- LiPo, LiHV, Li-Ion e outras químicas podem ter parâmetros diferentes.

### 5.4 Resultados possíveis

- tensão estimada sob carga;
- sag estimado;
- potência entregue;
- margem de tensão/cutoff;
- energia utilizável estimada.

## 6. Modelo do ESC / drive

O ESC não deve ser tratado como componente ideal quando o modelo avançado estiver ativo.

Parâmetros futuros possíveis:

- tensão de entrada;
- limite contínuo/burst;
- eficiência ou mapa de eficiência;
- duty cycle;
- perdas;
- temperatura, quando houver modelo defensável.

Na ausência de curva de eficiência, o sistema pode manter o ESC como ideal para uma análise simplificada, mas deve registrar essa hipótese.

Não inventar uma eficiência fixa universal.

## 7. Modelo eletromecânico do motor

### 7.1 Parâmetros desejáveis

```text
KV          rpm/V
Kt          N·m/A
Rm          resistência de enrolamento equivalente
I0          corrente sem carga em condição conhecida
V/I/RPM     dados medidos adicionais
limites térmicos quando disponíveis
```

### 7.2 Conversão KV ↔ Kt

Para constantes expressas de forma compatível e sob a aproximação ideal de motor:

```text
Kt ≈ 60 / (2π × KV)
```

com `KV` em rpm/V e `Kt` em N·m/A.

**Cuidado:** convenções de medição de motores BLDC, valores linha-linha/fase, ESC, tolerâncias e método do fabricante podem introduzir diferenças. O sistema não deve elevar a confiança apenas porque `Kt` foi derivado matematicamente de `KV`.

### 7.3 Equação elétrica simplificada

Modelo DC equivalente inicial:

```text
V ≈ I × Rm + Ke × ω
```

onde `Ke` deve estar em unidade compatível com SI e `ω` em rad/s.

Em um BLDC real acionado por ESC, essa é uma aproximação. Duty cycle, comutação, resistência do ESC, forma de onda e convenções de fase podem exigir modelo mais detalhado.

### 7.4 Torque

Modelo inicial:

```text
τ_motor ≈ Kt × I_torque
```

A definição de `I_torque` deve ser coerente com o modelo adotado. Se a corrente sem carga `I0` for usada para representar perdas internas, a implementação deve documentar exatamente como ela é descontada; não aplicar `I - I0` de forma automática em qualquer condição sem validar a convenção dos dados.

### 7.5 Potência mecânica

```text
P_mech = τ × ω
```

Eficiência do motor, quando entradas forem coerentes:

```text
η_motor = P_mech / P_electrical
```

Resultados > 1 ou negativos indicam dado/modelo inconsistente e devem falhar na validação.

## 8. Modelo da hélice

### 8.1 Dados mínimos desejáveis

- diâmetro `D`;
- pitch nominal;
- número de pás;
- modelo/geometria;
- coeficientes ou curvas do fabricante/ensaio quando disponíveis;
- regime estático ou velocidade de avanço;
- densidade do ar.

Diâmetro e pitch sozinhos não descrevem completamente uma hélice.

### 8.2 Modelo por coeficientes adimensionais

Quando `Ct`, `Cq`/`Cp` forem conhecidos para a condição adequada:

```text
T = Ct × ρ × n² × D⁴
Q = Cq × ρ × n² × D⁵
P = Cp × ρ × n³ × D⁵
```

Onde:

- `T`: empuxo em N;
- `Q`: torque resistente em N·m;
- `P`: potência no eixo em W;
- `ρ`: densidade do ar em kg/m³;
- `n`: rotações por segundo;
- `D`: diâmetro em metros.

Quando as definições dos coeficientes forem compatíveis, `Cp` e `Cq` possuem relação derivável pela velocidade angular. A implementação deve validar a convenção da fonte antes de converter entre coeficientes.

### 8.3 Voo com avanço

Para voo não estático, introduzir o advance ratio:

```text
J = V_forward / (n × D)
```

Nesse regime, `Ct`, `Cq` e `Cp` podem variar com `J`. Não reutilizar coeficiente estático como se fosse curva completa de cruzeiro.

### 8.4 Sem coeficientes

Se existirem apenas `diâmetro + pitch + pás`, o DroneCalc não deve fingir que conhece `Ct/Cq` com precisão.

Opções permitidas:

1. informar dados insuficientes para previsão teórica confiável;
2. usar modelo empírico explicitamente calibrado e versionado;
3. no futuro, usar BEMT/momentum theory com geometria de pá suficiente;
4. recorrer a curva de bancada aplicável.

## 9. Atmosfera

Empuxo depende da densidade do ar.

Primeira versão pode aceitar `ρ` como entrada/contexto e oferecer condição padrão claramente identificada somente como default de simulação, não como medição do ambiente.

Evoluções:

- altitude;
- pressão;
- temperatura;
- umidade;
- modelo ISA quando aplicável.

A condição atmosférica deve fazer parte da proveniência da simulação.

## 10. Ponto de operação motor × hélice

O RPM real não é `KV × V` com hélice instalada.

O ponto de equilíbrio deve satisfazer, dentro do modelo escolhido:

```text
τ_motor(ω, V, ...) = Q_propeller(ω, ρ, V_forward, ...)
```

Ao mesmo tempo, bateria/ESC/motor determinam a tensão e corrente disponíveis.

Fluxo conceitual:

```text
SOC / bateria / demanda
        ↓
V_load
        ↓
ESC / duty
        ↓
motor: corrente e torque em função de ω
        ↓
hélice: torque resistente em função de ω
        ↓
resolver equilíbrio
        ↓
RPM, I, τ, T, P, η
```

## 11. Solver numérico

O solver deve ser determinístico, limitado e testável.

Requisitos:

- intervalo físico de RPM/ω;
- detecção de ausência de raiz;
- tolerância explícita;
- limite de iterações;
- resultado não convergente não vira número válido;
- evitar depender exclusivamente de método sensível a chute inicial;
- preferir método bracketed robusto (por exemplo, bisseção/Brent ou equivalente) quando a função permitir;
- registrar versão e tolerâncias do solver.

A IA pode selecionar biblioteca numérica madura ou implementar rotina pequena, desde que verifique documentação, licença, estabilidade e testes. Não adicionar dependência pesada apenas para uma única raiz escalar sem justificativa.

## 12. Integração com dados de bancada

O Physics Engine e o Bench Engine não competem; eles se complementam.

```text
Existe curva medida aplicável?
  ├─ sim → usar/interpolar curva
  │        + opcionalmente comparar modelo teórico
  └─ não → existem parâmetros físicos suficientes?
           ├─ sim → simular e marcar estimated/modelled
           └─ não → dados insuficientes
```

Uma comparação entre modelo e bancada deve permitir calcular erro e, futuramente, calibrar parâmetros sem alterar os dados originais.

## 13. Modelo de confiança

Orientação inicial:

- bancada independente/reprodutível: `high` quando condições e fonte são adequadas;
- interpolação dentro de bancada: normalmente `medium` a `high` conforme qualidade;
- modelo físico calibrado contra dados: `medium` quando dentro do domínio validado;
- modelo por coeficientes de fabricante: `medium` ou menor conforme condições;
- modelo genérico com hipóteses: `low`;
- heurística: `low` e nunca equivalente a validação.

A confiança composta não pode exceder a qualidade efetiva dos parâmetros determinantes sem justificativa.

## 14. IDs/versionamento de modelos

Sugestões iniciais:

```text
BATTERY_THEVENIN_V1
MOTOR_DC_EQUIVALENT_V1
MOTOR_KT_FROM_KV_V1
PROP_COEFFICIENT_STATIC_V1
PROP_ADVANCE_RATIO_V1
PROPULSION_OPERATING_POINT_V1
AIR_DENSITY_CONTEXT_V1
```

Toda mudança que possa alterar numericamente resultados persistidos exige nova versão do modelo ou migração/compatibilidade explicitamente documentada.

## 15. Contratos conceituais

```ts
interface PhysicsContext {
  airDensityKgM3: number
  forwardSpeedMS?: number
  ambientTemperatureK?: number
}

interface BatteryOperatingState {
  soc: number
  openCircuitVoltageV: number
  internalResistanceOhm?: number
}

interface MotorElectricalModel {
  kvRpmPerV: number
  resistanceOhm?: number
  noLoadCurrentA?: number
  ktNmPerA?: number
}

interface PropellerCoefficientPoint {
  advanceRatio?: number
  ct: number
  cq?: number
  cp?: number
}

interface PropulsionOperatingPoint {
  rpm: number
  currentA: number
  torqueNm: number
  thrustN: number
  electricalPowerW: number
  mechanicalPowerW?: number
  source: 'measured' | 'manufacturer' | 'calculated' | 'interpolated' | 'estimated'
  confidence: 'high' | 'medium' | 'low'
  modelVersion: string
  notes: string[]
}
```

Esses contratos são conceituais. A implementação deve reutilizar os tipos canônicos de unidades, disponibilidade e proveniência definidos no domínio, evitando `number` sem semântica quando a infraestrutura de units já existir.

## 16. Validações obrigatórias

Recusar ou marcar inválido:

- `KV <= 0`;
- resistência negativa;
- `Kt <= 0`;
- corrente impossível/negativa fora de regime regenerativo explicitamente suportado;
- eficiência < 0 ou > 1 fora de tolerância numérica;
- diâmetro de hélice <= 0;
- densidade do ar <= 0;
- coeficientes incompatíveis com schema/convenção;
- solver sem convergência;
- potência mecânica maior que elétrica sem explicação/modelo válido;
- operação acima de limites conhecidos de motor/ESC/bateria;
- extrapolação de mapa/curva sem política explícita.

## 17. Testes obrigatórios

### Unitários

- conversão RPM ↔ rad/s;
- `KV → Kt` com caso de referência;
- potência torque × velocidade angular;
- equações `Ct/Cq/Cp` com dados sintéticos conhecidos;
- sag de bateria com resistência conhecida;
- solver com raízes sintéticas e casos sem raiz;
- invariantes de potência/eficiência.

### Integração

- bateria → motor → hélice → ponto de operação;
- comparação de simulação com fixture de bancada;
- queda de confiança quando parâmetros são estimados;
- seleção automática da rota medida quando existe curva aplicável;
- fallback para `dados insuficientes` quando faltam parâmetros críticos.

### Regressão

Fixtures físicos versionados devem impedir alterações silenciosas de resultados após refatorações.

## 18. Critério para implementação

Não implementar o Physics Engine avançado inteiro em uma única etapa.

Sequência recomendada:

1. contratos e unidades;
2. bateria sob carga;
3. motor DC equivalente/Kt;
4. hélice por coeficientes;
5. solver de ponto de operação;
6. integração e proveniência;
7. validação contra curvas de bancada;
8. somente depois, modelos de cruzeiro/atmosfera mais avançados.

## 19. Fora do primeiro escopo

- CFD;
- simulação estrutural de pá;
- BEMT completo sem geometria adequada;
- modelo térmico de alta fidelidade;
- interação aerodinâmica frame/hélice;
- propwash entre rotores;
- coaxial detalhado;
- transientes completos do ESC/motor;
- dinâmica de voo 6-DOF.

Esses itens podem ser adicionados somente quando houver caso de uso, dados e validação.

## 20. Regra de ouro

O Physics Engine existe para produzir uma estimativa fisicamente defensável quando os dados permitem — não para fabricar precisão.

Se não houver curva de bancada nem parâmetros suficientes para um modelo teórico validável, o resultado correto continua sendo:

**dados insuficientes**.
