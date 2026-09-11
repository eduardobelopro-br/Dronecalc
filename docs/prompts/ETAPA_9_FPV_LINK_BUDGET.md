# Prompt de Implementação — Alcance FPV / RF Link Budget

> Executar quando unidades físicas básicas, calculation engine e schemas mínimos de VTX/VRX/antenas estiverem disponíveis. A UI pode ser entregue junto ou em etapa posterior, conforme o roadmap vigente.

Você é a IA implementadora do DroneCalc. Antes de alterar código, leia `docs/PRD.md`, `docs/PRODUCT_SPEC.md`, `docs/ARCHITECTURE.md`, `docs/DOMAIN_MODEL.md`, `docs/CALCULATION_ENGINE.md`, `docs/FPV_LINK_BUDGET.md`, `docs/COMPONENT_CATALOG.md`, `docs/TEST_STRATEGY.md`, `docs/ROADMAP.md` e os ADRs vigentes.

## Objetivo

Implementar um núcleo determinístico de RF Link Budget para vídeo FPV que permita:

- avaliar uma configuração na distância-alvo informada pelo operador;
- calcular FSPL;
- calcular potência recebida e margens;
- estimar distância teórica máxima em espaço livre;
- calcular a potência teórica mínima de VTX necessária para a distância-alvo;
- preservar proveniência, hipóteses e limitações;
- integrar posteriormente componentes cadastrados de VTX, VRX/óculos e antenas.

## Regras obrigatórias

- fórmulas ficam fora da UI;
- usar frequência real em MHz quando disponível;
- não inventar sensibilidade do receptor;
- distinguir `directivity`, `gain`, `realized-gain` e `unknown`;
- não descontar novamente eficiência/mismatch quando já estiverem incluídos no ganho informado;
- SWR só gera mismatch loss quando semanticamente aplicável;
- diversity de seleção não soma ganhos de antenas;
- vídeo FPV e rádio-controle são links separados;
- resultado baseado em FSPL deve ser chamado de espaço livre/estimativa, nunca alcance garantido;
- potência necessária não equivale a potência legal/autorizada;
- `NaN`, infinito, distância/frequência/potência inválidas devem ser rejeitados;
- todos os resultados críticos devem possuir `formulaId`/versão/proveniência conforme contratos do projeto.

## Fórmulas mínimas

```text
P_dBm = 10 log10(P_mW)
P_mW = 10^(P_dBm/10)
FSPL_dB = 32.44 + 20log10(d_km) + 20log10(f_MHz)
Pr_dBm = Pt_dBm + Gt_dBi + Gr_dBi - losses_dB - FSPL_dB
available_margin_dB = Pr_dBm - sensitivity_dBm
operational_headroom_dB = available_margin_dB - desired_margin_dB
required_tx_power_dBm = sensitivity_dBm + desired_margin_dB + FSPL_dB + losses_dB - Gt_dBi - Gr_dBi
```

Para SWR:

```text
Gamma = (SWR - 1) / (SWR + 1)
accepted = 1 - Gamma^2
mismatch_loss_dB = -10 log10(accepted)
```

## Testes mínimos

- 100 mW = 20 dBm;
- 200 mW ≈ 23.0103 dBm;
- round-trip mW↔dBm;
- dobrar distância aumenta FSPL em ~6.02 dB;
- dobrar frequência aumenta FSPL em ~6.02 dB;
- link budget de fixture conhecido;
- solver de distância é inverso do cálculo direto dentro da tolerância;
- solver de potência fecha exatamente a margem desejada;
- SWR 1.0 = 0 dB;
- SWR 1.5 ≈ 0.177 dB;
- realized gain não recebe perda duplicada;
- gain kind desconhecido gera warning;
- sensibilidade ausente => dados insuficientes;
- diversity não soma ganhos;
- entradas inválidas não geram NaN/infinito silencioso.

## Liberdade para melhorar

Você pode melhorar contratos, organização de módulos, precisão numérica, estratégia de validação ou UX se encontrar solução mais eficaz/eficiente. Qualquer alteração deve preservar os invariantes acima, ser justificada, testada e documentada. Não introduza dependência pesada sem necessidade e consulte documentação oficial das bibliotecas utilizadas.

## Fora de escopo

Não implementar automaticamente terreno/DEM, Fresnel, radio horizon, regulamentação regional, RC/ExpressLRS/Crossfire, cálculo de interferência espectral ou promessa de alcance real. Esses itens podem evoluir depois sem quebrar o núcleo de link budget.
