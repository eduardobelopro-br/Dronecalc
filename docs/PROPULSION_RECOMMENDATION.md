# Recomendação de Propulsão Orientada ao Uso — DroneCalc

**Versão:** 0.1  
**Status:** especificação normativa para implementação futura  
**Escopo:** cálculo iterativo de massa por candidato e recomendação explicável de conjunto motor + hélice + bateria/tensão conforme o uso do drone.

## 1. Objetivo

O DroneCalc deve calcular automaticamente a massa do projeto à medida que o usuário adiciona peças e, quando solicitado, avaliar quais conjuntos de propulsão são mais adequados ao projeto e ao perfil de uso escolhido.

A unidade de recomendação não é apenas o motor. O sistema deve avaliar, conforme os dados disponíveis:

```text
motor + hélice + bateria/tensão + ESC/limites aplicáveis
```

A recomendação deve considerar simultaneamente:

- massa final do candidato;
- empuxo requerido por motor;
- TWR e reserva de potência adequados ao perfil;
- corrente e potência no regime relevante;
- eficiência estática/dados de propulsão aplicáveis;
- autonomia estimada quando calculável;
- compatibilidade elétrica e mecânica;
- qualidade/proveniência dos dados;
- prioridades do perfil de voo e preferências do usuário.

O sistema não deve declarar um componente como “melhor” sem informar o critério.

## 2. Princípio central: peso depende do candidato

A massa usada na análise deve incluir o próprio conjunto candidato.

Exemplo conceitual:

```text
massa_base_do_projeto
+ massa_dos_motores_candidatos
+ massa_das_hélices_candidatas
+ massa_da_bateria_candidata
+ ESC/outros itens que mudarem com a variante
= massa_de_decolagem_do_candidato
```

Portanto, comparar motores usando uma massa fixa que ignora a diferença de peso entre os próprios motores é incorreto.

Para cada candidato o DroneCalc deve:

1. materializar uma variante virtual do projeto sem alterar o projeto do usuário;
2. substituir/adicionar os componentes pertencentes ao candidato;
3. recalcular massa seca e massa de decolagem;
4. recalcular empuxo requerido por motor;
5. avaliar a curva de bancada ou Physics Engine aplicável;
6. recalcular corrente, potência, TWR, eficiência e autonomia disponíveis;
7. executar compatibilidade;
8. calcular adequação ao perfil;
9. guardar explicação e proveniência do resultado.

Se a busca também dimensionar bateria por autonomia, o processo pode exigir iteração porque uma bateria maior aumenta a massa e altera o ponto de operação. Essa iteração deve possuir critério de convergência/limite e nunca executar indefinidamente.

## 3. Massa parcial e dados ausentes

O Builder deve atualizar a massa sempre que peças forem adicionadas, removidas ou substituídas.

Se algum componente não possuir massa conhecida:

- não assumir `0 g`;
- apresentar massa parcial/incompleta;
- identificar quais peças não possuem massa;
- impedir ranking que dependa de massa completa quando a incerteza tornar a comparação enganosa, ou reduzir explicitamente a cobertura/confiança conforme regra documentada.

## 4. Empuxo requerido

Para multirrotor simétrico e balanceado:

```text
required_hover_thrust_gf_per_motor ≈ takeoff_mass_g / motor_count
```

Esse valor é apenas o requisito de hover idealizado. O conjunto também precisa atender à reserva/TWR exigida pelo perfil e às margens elétricas/mecânicas.

O DroneCalc não deve escolher o motor de maior empuxo máximo por padrão. Deve analisar o consumo no ponto de operação relevante.

## 5. Dados de propulsão

Ordem de preferência:

1. curva de bancada aprovada aplicável à combinação motor + hélice + tensão/condição;
2. interpolação somente dentro da curva válida;
3. Physics Engine quando houver parâmetros suficientes e domínio validado;
4. heurística somente para gerar/filtrar candidatos, nunca para fabricar empuxo/corrente de alta confiança.

Sem dados suficientes, o candidato pode aparecer como `dados insuficientes`, mas não deve receber métricas físicas inventadas.

## 6. Perfil de uso versus perfil de voo

O perfil de voo define prioridades gerais. O perfil operacional descreve em quais regimes a eficiência deve ser avaliada.

Não confundir perfil operacional com planejamento de rota/missão autônoma. O MVP pode trabalhar com pesos relativos de regimes de uso, por exemplo:

```ts
interface OperatingProfile {
  id: string
  phases: Array<{
    kind: 'hover' | 'cruise' | 'climb' | 'aggressive' | 'reserve'
    weight: number
  }>
}
```

Os pesos devem ser normalizados e versionados. Uma fase só pode contribuir com consumo físico se houver modelo/dado defensável para aquele regime.

Exemplos conceituais:

### Mini Long Range

- eficiência e autonomia no regime de hover/cruzeiro: prioridade muito alta;
- baixo peso: prioridade muito alta;
- estabilidade: alta;
- reserva de potência: suficiente, não maximizada;
- empuxo máximo isolado: prioridade secundária.

### Racing

- TWR/reserva de potência: muito alta;
- resposta/agilidade: muito alta;
- baixo peso: muito alta;
- eficiência/autonomia: secundária.

### Cinematic

- estabilidade: muito alta;
- eficiência/autonomia: alta;
- operação longe de limites contínuos: alta;
- payload: alta.

### Freestyle

- TWR/reserva: alta;
- resposta: alta;
- robustez: alta;
- eficiência: moderada.

Os pesos exatos pertencem a `FLIGHT_PROFILES.md` e devem ser calibrados com dados reais, não tratados como constantes físicas.

## 7. Avaliação por ponto de operação

Quando o hover exigir determinado empuxo por motor, o sistema deve procurar esse empuxo na curva aprovada e interpolar, quando possível:

```text
required thrust
→ throttle correspondente
→ corrente
→ tensão/potência
→ RPM
→ eficiência estática
```

O lookup deve preferir empuxo como eixo-alvo. Não assumir linearidade de throttle.

Para regimes diferentes de hover, somente calcular consumo quando houver modelo físico ou dados adequados. Não inventar corrente de cruzeiro como porcentagem fixa do hover.

## 8. Consumo ponderado pelo uso

Quando houver dados válidos para múltiplos regimes, uma métrica agregada pode ser calculada por ponderação explícita:

```text
I_weighted = Σ(I_phase × weight_phase)
P_weighted = Σ(P_phase × weight_phase)
```

ou energia por fase quando duração estiver definida.

Requisitos:

- pesos normalizados;
- nenhuma fase sem dado é tratada como zero;
- cobertura das fases deve ser mostrada;
- modelo e versão devem ser registrados;
- a métrica agregada não substitui a apresentação dos pontos individuais.

## 9. Pipeline de candidatos

Ordem normativa:

```text
1. selecionar candidatos plausíveis por catálogo/heurística
2. validar hélice × frame/mecânica
3. validar tensão motor/ESC/bateria
4. validar limites de corrente conhecidos
5. montar variante virtual do candidato
6. recalcular massa total
7. calcular empuxo requerido por motor
8. verificar TWR/reserva mínima do perfil
9. localizar pontos de operação em bancada/Physics Engine
10. calcular consumo/eficiência/autonomia disponíveis
11. executar warnings e compatibilidade completa
12. calcular score de adequação ao uso
13. produzir ranking explicável
```

Uma incompatibilidade `danger` não pode ser mascarada por score alto. O candidato deve ser invalidado ou classificado como incompatível conforme a regra correspondente.

## 10. Candidate set

Contrato conceitual:

```ts
interface PropulsionCandidate {
  motorVariantId: string
  propellerVariantId: string
  batteryVariantId?: string
  escVariantId?: string
  motorCount: number
}
```

O schema final pode ser diferente, mas a análise precisa identificar exatamente quais variantes e condições formam o candidato.

## 11. Resultado de avaliação

Contrato conceitual:

```ts
interface PropulsionCandidateEvaluation {
  candidate: PropulsionCandidate

  takeoffMass: CalculationResult<number>
  requiredHoverThrustPerMotor: CalculationResult<number>

  twr?: CalculationResult<number>
  hoverCurrentPerMotor?: CalculationResult<number>
  totalHoverCurrent?: CalculationResult<number>
  hoverEfficiencyGfPerW?: CalculationResult<number>
  endurance?: CalculationResult<number>

  compatibility: CompatibilityResult[]

  profileScore?: number
  dataCoverage: number
  confidence: 'high' | 'medium' | 'low'

  reasons: RecommendationReason[]
}
```

`dataCoverage` e `confidence` devem permanecer separados de `profileScore`. Não alterar silenciosamente o desempenho físico para “punir” falta de dados.

## 12. Ranking multiobjetivo explicável

O score pode combinar dimensões normalizadas, por exemplo:

- eficiência no regime relevante;
- autonomia;
- massa;
- TWR/reserva;
- estabilidade/adequação ao perfil;
- payload;
- compactação;
- margens elétricas.

Regras:

- normalizar antes de combinar grandezas diferentes;
- pesos vêm do perfil/versionamento;
- restrições rígidas são avaliadas antes do score;
- score não substitui métricas físicas;
- cobertura/confiança é exibida separadamente;
- empates devem ser explicáveis;
- nenhum ranking deve favorecer uma estimativa fraca apenas por produzir número aparentemente superior.

## 13. Saída esperada na UI

Exemplo conceitual:

```text
1. Motor X + Hélice A + 4S
   Massa de decolagem: 238 g
   Hover: 34%
   Corrente hover: 1,8 A/motor
   Eficiência hover: 6,1 gf/W
   TWR: 2,4
   Autonomia estimada: 21,8 min
   Adequação Mini Long Range: 91/100
   Cobertura dos dados: 94%
   Confiança: alta

   Por que foi recomendado?
   + menor consumo no regime relevante
   + menor massa do conjunto
   + atende à reserva de empuxo
   - menor TWR máximo que o candidato #2
```

A UI deve permitir abrir:

- componentes exatos;
- curva/Physics Engine usado;
- fórmulas;
- warnings;
- dados ausentes;
- versão do perfil;
- motivos do ranking.

## 14. Recomendar conjunto, não motor isolado

O texto de produto deve preferir **“conjunto de propulsão recomendado”** em vez de “melhor motor”.

O mesmo motor pode ter comportamento muito diferente com outra hélice ou tensão. Uma recomendação de motor sem contexto de hélice/tensão não é suficiente para o ranking técnico final.

## 15. Relação com o otimizador avançado

A recomendação básica de propulsão faz parte do fluxo principal do produto e não deve esperar a futura otimização global.

O otimizador avançado pode posteriormente ampliar a busca para:

- dimensionar bateria/capacidade iterativamente;
- explorar milhares de combinações;
- custo/disponibilidade;
- múltiplas restrições simultâneas;
- Pareto front;
- otimização de payload/autonomia/peso.

A implementação inicial pode trabalhar com um conjunto limitado de candidatos do catálogo, desde que aplique corretamente massa, compatibilidade, dados de propulsão e perfil.

## 16. Warnings/estados mínimos

Prever códigos equivalentes a:

```text
PROPULSION_CANDIDATE_INCOMPATIBLE
PROPULSION_DATA_INSUFFICIENT
PROPULSION_MASS_INCOMPLETE
PROPULSION_HOVER_OUT_OF_BENCH_RANGE
PROPULSION_TWR_BELOW_PROFILE_MINIMUM
PROPULSION_BATTERY_CURRENT_INSUFFICIENT
PROPULSION_ESC_MARGIN_LOW
PROPULSION_OPERATING_PROFILE_INCOMPLETE
PROPULSION_RANKING_LOW_DATA_COVERAGE
```

Nomes podem evoluir desde que a semântica permaneça estável/documentada.

## 17. Testes obrigatórios

Cobrir no mínimo:

- massa do projeto muda ao trocar motor por variante mais pesada;
- candidato mais pesado aumenta o empuxo requerido de hover;
- ranking não usa massa-base fixa ignorando os motores candidatos;
- conjunto de maior empuxo máximo não vence automaticamente perfil de eficiência;
- Mini Long Range pode preferir conjunto de menor TWR quando ambos atendem à reserva mínima e o segundo é mais eficiente/leve;
- Racing pode preferir maior reserva/TWR conforme pesos do perfil;
- `danger` invalida candidato independentemente do score;
- curva de bancada é usada antes do Physics Engine quando aplicável;
- alvo fora da curva não é extrapolado silenciosamente;
- fase operacional sem dados não é tratada como consumo zero;
- score e confiança/cobertura são independentes;
- troca de bateria recalcula massa, tensão, corrente e autonomia;
- ranking é determinístico para mesmas entradas/versões;
- razões do ranking correspondem às métricas calculadas.

## 18. Segurança e limitações

A recomendação é apoio de engenharia e não certificação de aeronavegabilidade.

A aplicação deve lembrar que ensaios físicos de motor/hélice exigem bancada apropriada e procedimentos de segurança. Hélices devem permanecer removidas durante configuração eletrônica comum de bancada; testes de propulsão com hélice exigem equipamento específico, contenção e procedimento apropriado.

## 19. Liberdade de implementação

A IA implementadora pode substituir algoritmos, estruturas de dados, estratégia de busca ou normalização por abordagem mais eficaz, eficiente, segura e testável, desde que:

1. preserve o recálculo de massa por candidato;
2. preserve hard constraints antes do score;
3. não confunda motor isolado com conjunto de propulsão;
4. não invente dados físicos ausentes;
5. preserve prioridade de bancada sobre modelo teórico aplicável;
6. mantenha score explicável e confiança/cobertura separadas;
7. implemente testes equivalentes ou melhores;
8. documente trade-offs e atualize ADR/spec quando houver mudança estrutural.
