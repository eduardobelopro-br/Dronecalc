# Alcance FPV / RF Link Budget — DroneCalc

**Versão:** 0.1  
**Status:** especificação normativa para implementação futura  
**Escopo:** estimativa de enlace de vídeo FPV em espaço livre, cálculo de margem e dimensionamento inverso para uma distância-alvo.

## 1. Objetivo

O operador informa a distância máxima que deseja manter entre o piloto/estação de recepção e o drone. O DroneCalc usa dados do sistema FPV para responder, de forma auditável:

- qual potência recebida é esperada na distância-alvo;
- se a sensibilidade do VRX é atendida com a margem configurada;
- qual margem de enlace permanece;
- qual distância teórica de espaço livre seria suportada pela configuração;
- qual potência mínima de VTX seria necessária para a distância desejada;
- quais dados faltam para uma conclusão defensável.

O módulo deve ser genérico no núcleo de RF, mas a primeira UI é específica para **vídeo FPV**.

## 2. Não confundir alcance teórico com alcance real

O cálculo básico usa perda de espaço livre (FSPL) e representa um cenário idealizado de linha de visada.

Ele não modela automaticamente:

- obstáculos;
- terreno;
- multipath;
- sombra do corpo/frame/bateria;
- orientação dinâmica das antenas;
- interferência externa;
- ruído local do receptor;
- degradação por conectores/cabos não informados;
- zona de Fresnel obstruída;
- curvatura da Terra/horizonte de rádio;
- alterações atmosféricas;
- diversidade/combinação de receptores sem modelo explícito.

A UI deve usar termos como **alcance teórico em espaço livre**, **margem calculada** e **estimativa**, nunca `alcance garantido`.

## 3. Entradas mínimas

### 3.1 Objetivo do operador

```ts
interface FpvRangeTarget {
  targetDistanceKm: number
  desiredLinkMarginDb: number
}
```

A margem é uma política/objetivo do projeto, não uma constante física universal. Um valor default pode existir na UI, desde que seja claramente configurável e documentado.

### 3.2 Frequência

Usar a frequência real do canal quando conhecida:

```ts
frequencyMHz: number
```

Evitar usar apenas o rótulo da banda (`5.8 GHz`) quando o canal/frequência exata estiver disponível.

### 3.3 Transmissor/VTX

```ts
interface FpvTransmitter {
  txPowerDbm?: number
  txPowerMw?: number
  txFeedlineLossDb?: number
}
```

Se potência vier em mW, converter para dBm no calculation engine.

### 3.4 Antena do VTX

O schema precisa distinguir o significado do ganho informado:

```ts
type AntennaGainKind =
  | 'directivity'
  | 'gain'
  | 'realized-gain'
  | 'unknown'
```

Campos possíveis:

```ts
interface RfAntennaSpec {
  gainDbI?: number
  gainKind?: AntennaGainKind
  efficiencyFraction?: number
  swr?: number
  explicitMismatchLossDb?: number
  cableLossDb?: number
  connectorLossDb?: number
  polarization?: 'RHCP' | 'LHCP' | 'linear-horizontal' | 'linear-vertical' | 'unknown'
  explicitPolarizationLossDb?: number
}
```

## 4. Receptor/VRX/óculos

O usuário pode:

1. selecionar um VRX/óculos cadastrado no catálogo;
2. selecionar apenas a antena do óculos/receptor e preencher a sensibilidade manualmente;
3. informar todos os campos manualmente.

Campos centrais:

```ts
interface FpvReceiverSpec {
  sensitivityDbm: number
  sensitivityCondition?: string
  rxFeedlineLossDb?: number
  antenna: RfAntennaSpec
}
```

A sensibilidade precisa preservar contexto. Em sistemas digitais ela pode depender de modo, largura de banda, taxa/modulação ou firmware. Em vídeo analógico a degradação é progressiva e um único threshold não descreve toda a qualidade percebida.

Nunca inventar sensibilidade a partir do nome do VRX.

## 5. Conversão de potência

### 5.1 mW → dBm

```text
P_dBm = 10 × log10(P_mW)
```

### 5.2 dBm → mW

```text
P_mW = 10^(P_dBm / 10)
```

Identificadores sugeridos:

```text
RF_MW_TO_DBM_V1
RF_DBM_TO_MW_V1
```

## 6. Perda de espaço livre

Para distância em quilômetros e frequência em MHz:

```text
FSPL_dB = 32.44 + 20×log10(distance_km) + 20×log10(frequency_MHz)
```

A constante deve ser implementada com precisão/documentação coerentes e testada contra forma equivalente em SI.

Identificador sugerido:

```text
RF_FSPL_V1
```

## 7. Link budget direto

Forma conceitual:

```text
Pr_dBm =
  Pt_dBm
  + Gt_dBi
  + Gr_dBi
  - Ltx_dB
  - Lrx_dB
  - Lmisc_dB
  - FSPL_dB
```

Onde as perdas aplicadas devem ser semanticamente defensáveis e não duplicadas.

A condição básica de fechamento com margem é:

```text
Pr_dBm >= sensitivity_dBm + desired_margin_dB
```

Margem residual:

```text
available_margin_dB = Pr_dBm - sensitivity_dBm
operational_headroom_dB = available_margin_dB - desired_margin_dB
```

Estados possíveis:

```text
pass
borderline
fail
insufficient-data
```

Os thresholds de `borderline` são política de produto e devem ser configuráveis/versionados.

## 8. Distância máxima teórica em espaço livre

Primeiro calcular a perda máxima admissível:

```text
max_path_loss_dB =
  Pt_dBm
  + Gt_dBi
  + Gr_dBi
  - losses_dB
  - sensitivity_dBm
  - desired_margin_dB
```

Depois inverter FSPL:

```text
distance_km =
  10 ^ ((max_path_loss_dB - 32.44 - 20×log10(frequency_MHz)) / 20)
```

Nome obrigatório na UI:

**Distância teórica máxima em espaço livre**

Nunca `alcance garantido`.

Identificador sugerido:

```text
RF_FREE_SPACE_MAX_DISTANCE_V1
```

## 9. Potência mínima de VTX para a distância-alvo

Resolver o link budget para `Pt`:

```text
required_tx_power_dBm =
  sensitivity_dBm
  + desired_margin_dB
  + FSPL_dB
  + losses_dB
  - Gt_dBi
  - Gr_dBi
```

Converter opcionalmente para mW.

O resultado deve ser apresentado como:

**Potência teórica mínima de transmissão para o link budget informado**

Isso não significa que o valor seja legal, disponível no VTX, termicamente sustentável ou suficiente em ambiente real.

Identificadores sugeridos:

```text
RF_REQUIRED_TX_POWER_DBM_V1
RF_REQUIRED_TX_POWER_MW_V1
```

## 10. EIRP

Quando os dados permitirem:

```text
EIRP_dBm = Pt_dBm - tx_feedline_losses_dB + effective_tx_antenna_gain_dBi
```

Exibir EIRP calculada como dado técnico. Verificação regulatória é um módulo separado/futuro e não deve assumir automaticamente que determinada potência é permitida em uma região.

## 11. SWR e mismatch loss

Para SWR válido `>= 1`:

```text
Gamma = (SWR - 1) / (SWR + 1)
accepted_power_fraction = 1 - Gamma^2
mismatch_loss_dB = -10 × log10(accepted_power_fraction)
```

Exemplo: SWR 1.5 resulta em aproximadamente 0.18 dB de mismatch loss.

Identificador sugerido:

```text
RF_SWR_MISMATCH_LOSS_V1
```

### 11.1 Regra contra dupla contagem

Esse é um requisito crítico.

- `directivity`: não inclui eficiência de radiação nem mismatch; perdas podem precisar ser aplicadas separadamente;
- `gain`: normalmente já inclui eficiência de radiação, mas não necessariamente mismatch;
- `realized-gain`: inclui efeitos de mismatch e não deve receber novamente a mesma perda;
- `unknown`: não aplicar eficiência/SWR automaticamente sem confirmação; emitir warning.

O DroneCalc não deve copiar cegamente calculadoras que aceitam simultaneamente `antenna gain`, `antenna efficiency` e `SWR loss` sem saber a semântica do ganho.

Warnings sugeridos:

```text
RF_ANTENNA_GAIN_KIND_UNKNOWN
RF_POSSIBLE_DOUBLE_COUNTED_ANTENNA_LOSS
RF_SWR_INVALID
```

## 12. Eficiência da antena

Quando o input for `directivity` e houver eficiência de radiação explícita:

```text
radiation_efficiency_loss_dB = -10 × log10(efficiency_fraction)
```

Não aplicar novamente se o ganho informado já inclui essa eficiência.

Eficiência deve estar entre `0 < η <= 1`.

## 13. Polarização

A polarização deve ser registrada para TX e RX.

Não assumir uma perda universal para combinações reais sem dados suficientes. Se houver perda explícita do fabricante/ensaio, ela pode entrar no link budget.

Casos conhecidos de incompatibilidade de polarização devem gerar warning/danger conforme regra futura, mas o MVP não deve inventar um valor de perda em dB.

## 14. Diversity / múltiplas antenas

Óculos/VRX podem possuir diversity ou múltiplas entradas.

MVP:

- modelar cada branch separadamente;
- em diversity do tipo seleção, mostrar o melhor branch calculado sob as hipóteses disponíveis;
- não somar ganhos de duas antenas como se fossem um array coerente;
- sistemas de combining real exigem modelo específico futuro.

Warnings sugeridos:

```text
RF_DIVERSITY_MODEL_SIMPLIFIED
RF_MULTIPLE_RX_BRANCHES_INCOMPLETE
```

## 15. Catálogo necessário

Adicionar/expandir categorias:

```text
VTX
VRX / FPV Goggles
FPV Antenna
RF Cable / Pigtail (opcional)
```

Campos de catálogo relevantes:

### VTX
- frequências/canais;
- potências configuráveis em mW/dBm;
- potência por modo, quando aplicável;
- conector;
- massa;
- consumo elétrico;
- potência térmica/limites declarados quando disponíveis.

### VRX/óculos
- frequência/banda;
- sensibilidade e condição;
- tipo de receiver/diversity;
- número de entradas;
- conectores.

### Antena
- frequência/faixa;
- ganho + `gainKind`;
- polarização;
- SWR por frequência quando conhecido;
- eficiência quando realmente fornecida;
- conector;
- massa.

Dados importados por scraping/IA seguem staging e proveniência existentes.

## 16. Fluxo de UX

No projeto:

```text
Alcance FPV desejado: [ 5.0 ] km
Frequência/canal:     [ 5800 ] MHz
Margem desejada:      [ ... ] dB

VTX
[ selecionar do catálogo / preencher manualmente ]

Antena do drone
[ selecionar / preencher ]

VRX / Óculos
[ selecionar / preencher ]

Antena(s) do receptor
[ selecionar / preencher ]

[ Calcular enlace ]
```

Saída:

```text
Distância-alvo
FSPL
Potência recebida estimada
Sensibilidade utilizada
Margem disponível
Margem desejada
Headroom residual
Potência teórica mínima de VTX
EIRP calculada
Distância teórica máxima em espaço livre
Warnings/limitações
```

## 17. Recomendação de equipamento

Quando o catálogo estiver maduro, o sistema pode filtrar/recomendar combinações que atendam à distância-alvo.

Ordem:

```text
frequência compatível
→ polarização/conectores compatíveis
→ sensibilidade disponível
→ ganho/perdas conhecidas
→ link budget na distância-alvo
→ margem mínima do projeto
→ consumo/massa do VTX
→ ranking explicável
```

A recomendação de RF não deve ignorar o impacto elétrico e de massa do VTX no projeto.

## 18. Integração com perfis de voo

`targetFpvRangeKm` pode existir como constraint/intenção do projeto, especialmente em Long Range e Mini Long Range.

Mas:

- perfil não define distância automaticamente;
- alcance de vídeo e alcance do rádio-controle são links separados;
- o projeto deve indicar explicitamente qual link está sendo analisado;
- o menor link operacional pode limitar o uso real, mas a aplicação não deve fundi-los silenciosamente.

## 19. Analógico versus digital

O núcleo matemático de link budget é compartilhável, porém a interpretação muda.

### Analógico

- qualidade degrada progressivamente;
- sensibilidade/threshold pode representar uma condição subjetiva ou especificação particular;
- margem deve ser exibida com cautela.

### Digital

- pode existir comportamento tipo `cliff`;
- sensibilidade depende do modo/taxa/banda;
- usar o valor correspondente ao modo selecionado;
- não usar uma sensibilidade genérica se o fabricante fornece valores condicionais.

## 20. Validações obrigatórias

Recusar/marcar inválido:

- distância `<= 0`;
- frequência `<= 0`;
- potência mW `<= 0`;
- SWR `< 1`;
- eficiência `<= 0` ou `> 1`;
- sensibilidade ausente para cálculo de fechamento;
- ganho sem unidade/semântica quando necessário;
- perdas negativas salvo campo explicitamente definido como ganho;
- `NaN`, infinito ou overflow numérico.

## 21. Warnings mínimos

```text
RF_TARGET_DISTANCE_INVALID
RF_FREQUENCY_INVALID
RF_RX_SENSITIVITY_MISSING
RF_LINK_MARGIN_MISSING
RF_ANTENNA_GAIN_KIND_UNKNOWN
RF_POSSIBLE_DOUBLE_COUNTED_ANTENNA_LOSS
RF_SWR_INVALID
RF_POLARIZATION_UNKNOWN
RF_POLARIZATION_MISMATCH
RF_FREE_SPACE_ONLY_MODEL
RF_DIVERSITY_MODEL_SIMPLIFIED
RF_REQUIRED_TX_POWER_EXCEEDS_SELECTED_VTX
RF_DATA_COVERAGE_INCOMPLETE
RF_REGULATORY_STATUS_NOT_CHECKED
```

## 22. Confiança e proveniência

O cálculo FSPL é determinístico, mas o resultado de alcance depende da qualidade dos inputs.

Exemplo:

```text
FSPL: calculated / high
Sensibilidade: manufacturer / medium-high conforme fonte
Ganho da antena: manufacturer / medium
Perdas manuais: user-provided / variável
Distância máxima resultante: estimated / medium ou low
```

A distância final nunca deve receber confiança maior do que as entradas determinantes justificam.

## 23. Testes obrigatórios

Cobrir:

- mW ↔ dBm round-trip;
- 100 mW = 20 dBm;
- 200 mW ≈ 23.0103 dBm;
- FSPL aumenta ~6.02 dB ao dobrar distância na mesma frequência;
- FSPL aumenta ~6.02 dB ao dobrar frequência na mesma distância;
- link budget direto com fixture conhecida;
- solver inverso de distância retorna a distância original dentro da tolerância;
- solver de potência retorna potência que fecha exatamente a margem desejada;
- SWR 1.0 → 0 dB mismatch loss;
- SWR 1.5 → ~0.177 dB;
- realized gain não recebe mismatch/eficiência duplicados;
- gainKind desconhecido produz warning;
- distância/frequência inválidas são rejeitadas;
- sensibilidade ausente produz `dados insuficientes`;
- branch diversity não soma ganhos;
- analógico/digital preservam condição de sensibilidade;
- EIRP usa somente perdas/ganhos aplicáveis;
- mesma entrada e mesma versão geram o mesmo resultado.

## 24. Evoluções futuras

Sem bloquear o MVP:

- zona de Fresnel e clearance;
- radio horizon com alturas e modelo atmosférico explícito;
- terreno/LOS via mapa/DEM;
- interferência/noise floor medido;
- fade margin por ambiente;
- modelos de diversidade/combining;
- link de controle RC/ExpressLRS/Crossfire separado;
- verificação regulatória por região;
- recomendação conjunta vídeo + controle, mantendo links distintos.

## 25. Liberdade de implementação

A IA implementadora pode melhorar fórmulas auxiliares, contratos, estrutura de módulos ou UX se encontrar abordagem mais eficaz/eficiente, desde que:

1. preserve FSPL/link budget auditável;
2. preserve o solver inverso para distância-alvo/potência necessária;
3. não duplique perdas de antena;
4. não apresente espaço livre como alcance real garantido;
5. não invente sensibilidade, ganho ou perdas;
6. mantenha vídeo e RC separados;
7. mantenha fórmulas fora da UI;
8. implemente testes equivalentes ou melhores;
9. documente trade-offs e atualize ADR/spec quando necessário.
