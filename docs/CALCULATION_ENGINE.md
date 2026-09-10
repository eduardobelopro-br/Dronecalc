# Motor de Cálculo — DroneCalc

**Versão:** 0.1  
**Objetivo:** definir fórmulas, premissas, disponibilidade, proveniência e limites dos cálculos.

## 1. Princípios

1. Fórmulas ficam fora da UI.
2. Toda grandeza física possui unidade conhecida.
3. Dados medidos têm prioridade sobre aproximações.
4. Ausência de dado nunca é convertida silenciosamente em zero.
5. Interpolação e estimativa são identificadas explicitamente.
6. O motor não deve extrapolar curva de bancada por padrão.
7. Resultados dependentes de hipóteses exibem as hipóteses.
8. A versão do algoritmo acompanha cada análise persistida/exportada.

## 2. Unidades canônicas

Recomendação inicial do núcleo:

- massa: gramas (`g`) ou quilogramas internamente de forma única por módulo;
- comprimento: milímetros (`mm`) para geometria de montagem e metros (`m`) para fórmulas SI quando necessário;
- tensão: volts (`V`);
- corrente: amperes (`A`);
- potência: watts (`W`);
- energia: watt-hora (`Wh`);
- capacidade: ampere-hora (`Ah`) no cálculo, mAh apenas em entrada/apresentação;
- tempo: segundos/minutos conforme contrato explícito;
- empuxo: Newtons para física rigorosa ou grama-força equivalente na UI. Nunca confundir massa com força no domínio.

A implementação deve centralizar conversões.

## 3. Massa

### 3.1 Massa total de uma linha

```text
mass_line = unit_mass × quantity
```

### 3.2 Massa total do projeto

```text
mass_total = Σ mass_line
```

Separar quando possível:

- `dryMass`: estrutura + propulsão + eletrônica sem bateria/payload destacável;
- `batteryMass`;
- `payloadMass`;
- `takeoffMass`: massa operacional completa.

O significado exato das categorias deve permanecer documentado e consistente.

### 3.3 Componente sem massa

Se uma peça não possuir massa conhecida:

- não assumir 0 g;
- retornar total parcial com aviso ou marcar cálculo como incompleto, conforme contexto;
- mostrar quais componentes faltam.

## 4. Centro de gravidade

Quando massa e posição forem conhecidas:

```text
CGx = Σ(mi × xi) / Σ(mi)
CGy = Σ(mi × yi) / Σ(mi)
CGz = Σ(mi × zi) / Σ(mi)
```

Requisitos:

- convenção de eixos única;
- posições no mesmo sistema de referência;
- apenas componentes com posição conhecida entram em um CG completo;
- CG parcial deve ser explicitamente identificado como parcial.

## 5. Bateria

A bateria fornece valores por célula conforme sua química/modelo. Não fixar globalmente 3,7 V ou 4,2 V para todas as baterias.

### 5.1 Tensão nominal

```text
V_nominal = series_cells × nominal_cell_voltage
```

### 5.2 Tensão cheia

```text
V_full = series_cells × full_cell_voltage
```

### 5.3 Capacidade

```text
capacity_Ah = capacity_mAh / 1000
```

### 5.4 Energia nominal

```text
energy_Wh = V_nominal × capacity_Ah
```

Esse valor é nominal e não representa toda a energia efetivamente utilizável em voo.

### 5.5 Corrente teórica por C-rating

Quando o fabricante fornecer C-rating:

```text
I_theoretical = capacity_Ah × C
```

Classificação obrigatória: valor de fabricante/derivado, não medição. A UI deve avisar que C-rating pode não refletir a capacidade contínua real da bateria.

Quando existir `measuredContinuousCurrentA`, preferi-lo para análise prática, preservando ambos os dados.

## 6. Potência elétrica

```text
P = V × I
```

Se `V` e `I` vierem de uma amostra de bancada, potência calculada pode receber origem `calculated` com referências às entradas medidas.

## 7. Empuxo e relação empuxo/peso

### 7.1 Empuxo total

Para motores equivalentes no mesmo ponto:

```text
T_total = T_motor × motor_count
```

Para motores não equivalentes:

```text
T_total = Σ T_motor_i
```

### 7.2 Thrust-to-weight ratio (TWR)

Fisicamente:

```text
TWR = total_thrust_force / weight_force
```

Quando a UI usa empuxo em grama-força equivalente e massa em gramas, o valor numérico da razão é:

```text
TWR ≈ thrust_gf / takeoff_mass_g
```

O perfil de voo determina se um TWR é adequado. O núcleo não deve impor uma tabela universal de “bom” ou “ruim”.

## 8. Hover

Para multirrotor simétrico e balanceado, primeiro alvo:

```text
required_thrust_per_motor = weight_force / motor_count
```

Em representação equivalente para lookup em curva em gramas:

```text
required_thrust_gf_per_motor ≈ takeoff_mass_g / motor_count
```

O motor procura na curva motor+hélíce o ponto que produz esse empuxo e interpola corrente, potência e throttle.

A hipótese de distribuição uniforme deve ser registrada.

## 9. Interpolação de curva de bancada

Dados devem ser ordenados pelo eixo utilizado.

Interpolação linear entre dois pontos:

```text
f(x) = y0 + (x - x0) × (y1 - y0) / (x1 - x0)
```

Regras MVP:

- apenas entre pontos conhecidos;
- sem extrapolação por padrão;
- rejeitar pontos duplicados ambíguos;
- detectar valores fisicamente inválidos (corrente negativa, tensão <= 0 etc.);
- preservar a curva original;
- indicar `source = interpolated`.

Para hover, preferência por interpolar usando **empuxo como eixo-alvo**, pois o requisito físico é o empuxo necessário. Throttle é um dado secundário.

## 10. Corrente total

Estimativa simplificada:

```text
I_propulsion = Σ I_motor_i
I_total = I_propulsion + I_auxiliary
```

`I_auxiliary` inclui cargas elétricas conhecidas como FC, VTX, GPS, câmera, receptor e acessórios. Cargas alimentadas por BEC precisam respeitar a cadeia elétrica e eficiência quando o modelo evoluir.

No MVP, se carga auxiliar não estiver disponível, retornar explicitamente que a corrente total é parcial ou estimada.

## 11. Potência total

```text
P_total ≈ Σ P_motor_i + P_auxiliary
```

Evitar calcular potência total usando tensão nominal quando a curva fornece tensão medida sob carga; utilizar o conjunto de dados mais coerente disponível.

## 12. Autonomia

Modelo básico baseado em corrente média:

```text
usable_capacity_Ah = capacity_Ah × usable_fraction
flight_time_h = usable_capacity_Ah / average_current_A
flight_time_min = flight_time_h × 60
```

Se existir reserva separada:

```text
usable_fraction = 1 - reserve_fraction
```

ou outro modelo explicitamente configurado. Não aplicar duas vezes a mesma reserva.

### 12.1 Hover

`average_current_A` vem do ponto interpolado de hover somado às cargas auxiliares.

### 12.2 Cruzeiro

Só deve ser apresentado se houver modelo/entrada que diferencie cruzeiro de hover. Não inventar “corrente de cruzeiro” como porcentagem fixa silenciosa.

### 12.3 Limitações

Autonomia real depende de:

- sag de tensão;
- resistência interna;
- temperatura;
- vento;
- velocidade;
- aerodinâmica;
- eficiência de hélice/motor/ESC;
- estado e idade da bateria;
- comportamento do piloto;
- cutoff configurado.

Portanto autonomia deve normalmente ser `estimated`, mesmo quando usa curva medida.

## 13. Eficiência

Quando potência e empuxo estão disponíveis:

```text
efficiency_g_per_W = thrust_gf / power_W
```

Esse indicador é válido para comparar pontos sob condições semelhantes. Não deve ser usado isoladamente para comparar sistemas em regimes muito diferentes sem contexto.

Mini Long Range deve dar peso elevado à eficiência próxima ao regime de hover/cruzeiro, não apenas ao ponto de máxima potência.

## 14. Área de disco e disk loading

Para hélice com diâmetro `D` em metros:

```text
A_one = π × (D / 2)^2
A_total = motor_count × A_one
```

Disk loading físico:

```text
disk_loading = weight_force / A_total
```

A apresentação pode oferecer unidades amigáveis, mas a fórmula deve usar grandezas coerentes.

O cálculo assume discos não sobrepostos; configurações coaxiais exigirão modelo específico futuro.

## 15. RPM sem carga

Aproximação educacional:

```text
RPM_no_load ≈ KV × V
```

Esse valor **não é RPM real com hélice** e não deve ser utilizado para prever empuxo. Classificar como estimativa de baixa confiança/valor teórico.

## 16. Compatibilidade elétrica

### 16.1 Tensão bateria × motor

Verificar contra faixa de células e/ou tensão do motor. Se dados de células e tensão divergirem, preferir validação explícita e emitir inconsistência de catálogo.

### 16.2 Tensão bateria × ESC

A tensão máxima cheia da bateria deve ser compatível com o limite de entrada do ESC, não apenas a tensão nominal.

### 16.3 Corrente motor × ESC

Quando corrente máxima do motor é conhecida para a combinação relevante:

```text
margin_ratio = (ESC_continuous_A - motor_current_A) / motor_current_A
```

A severidade de margem baixa deve ser configurável e documentada. Não considerar burst como substituto automático do contínuo.

### 16.4 Bateria × demanda

Comparar corrente estimada do sistema com:

1. corrente contínua medida/recomendada da bateria, quando disponível;
2. corrente derivada de C-rating como fallback menos confiável.

## 17. Compatibilidade mecânica inicial

Regras possíveis:

- diâmetro da hélice <= limite do frame;
- padrão de montagem do motor compatível com frame;
- shaft/hub de hélice compatível quando os dados existirem;
- padrão de stack compatível para eletrônica quando modelado.

Falta de dado não deve gerar `success`; deve gerar verificação indisponível/info.

## 18. Níveis de confiança

Orientação inicial:

### High
- valor diretamente medido e fonte identificada;
- cálculo determinístico simples a partir de entradas confiáveis, quando a incerteza adicional é desprezível.

### Medium
- interpolação dentro de curva medida;
- cálculo com hipóteses razoáveis e dados de fabricante.

### Low
- estimativa baseada em informação incompleta;
- dado de fabricante sem contexto suficiente;
- aproximação teórica como RPM sem carga.

A confiança final de um cálculo composto não pode ser maior que a confiabilidade efetiva das entradas determinantes sem justificativa explícita.

## 19. Proveniência composta

O MVP pode começar com `source` único, mas deve ser projetado para evoluir para:

```ts
interface Provenance {
  primaryKind: DataSourceKind
  inputSources: SourceReference[]
  formulaId?: string
  modelVersion: string
}
```

## 20. Fórmulas identificáveis

Cada cálculo crítico deverá ter um identificador estável, por exemplo:

- `MASS_TOTAL_V1`;
- `BATTERY_ENERGY_NOMINAL_V1`;
- `BATTERY_C_CURRENT_V1`;
- `POWER_DC_V1`;
- `TWR_V1`;
- `HOVER_BENCH_INTERPOLATION_V1`;
- `ENDURANCE_CURRENT_MODEL_V1`;
- `CG_WEIGHTED_POSITION_V1`.

Isso facilita testes, auditoria e comparação entre versões.

## 21. Casos inválidos obrigatórios

O motor deve recusar ou marcar como inválido:

- massa negativa;
- quantidade negativa;
- tensão <= 0 em bateria ativa;
- capacidade <= 0;
- número de células <= 0;
- C-rating negativo;
- corrente negativa;
- curva sem pontos suficientes para interpolação;
- divisão por zero;
- alvo de interpolação fora da curva quando extrapolação estiver desabilitada.

## 22. Regra de ouro

Se o DroneCalc não possui dados suficientes para produzir um resultado defensável, a resposta correta é **“dados insuficientes”**, acompanhada do que falta — não um número inventado.
