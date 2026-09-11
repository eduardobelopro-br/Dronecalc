# Matriz de Rastreabilidade de Requisitos — DroneCalc

**Versão:** 0.5

Este documento liga requisitos do PRD às especificações, etapas do roadmap e tipos de teste esperados.

| Req. | Resumo | Especificação principal | Roadmap | Teste esperado |
|---|---|---|---|---|
| RF-01 | Projetos | PRODUCT_SPEC §2 | Fases 2 e 10 | integração/UI |
| RF-02 | Componentes | COMPONENT_CATALOG | Fases 4, 5, 6 e 10 | schema/UI/ingestão |
| RF-03 | Massa | CALCULATION_ENGINE §3 + PROPULSION_RECOMMENDATION §2–3 | Fases 3, 8 e 10 | unitário + fixture |
| RF-04 | Energia | CALCULATION_ENGINE §5 | Fase 3 | unitário |
| RF-05 | Sistema elétrico | CALCULATION_ENGINE §16 + PHYSICS_ENGINE | Fases 3 e 7 | unitário + limites |
| RF-06 | Propulsão | CALCULATION_ENGINE §§7–9 + PHYSICS_ENGINE + BENCH_DATA_WORKSPACE | Fase 7 | unitário + fixture de curva/modelo |
| RF-07 | Autonomia | CALCULATION_ENGINE §12 | Fases 3, 7, 8 e 10 | unitário + regressão |
| RF-08 | Perfis de voo | FLIGHT_PROFILES + PROPULSION_RECOMMENDATION §6 | Fase 8 | unitário/scoring |
| RF-09 | Compatibilidade | PRODUCT_SPEC §7 + PROPULSION_RECOMMENDATION §9 + FPV_LINK_BUDGET | Fases 3, 7, 8, 9 e 10 | unitário + integração |
| RF-10 | Proveniência | PRODUCT_SPEC §6 + ASSISTED_INGESTION + BENCH_DATA_WORKSPACE + PROPULSION_RECOMMENDATION + FPV_LINK_BUDGET | Todas as etapas técnicas | unitário + UI |
| RF-11 | Comparação | PRODUCT_SPEC §8 | Fase 11 | integração/UI |
| RF-12 | Unidades | CALCULATION_ENGINE §2 + FPV_LINK_BUDGET §§5–6 | Etapa 1D + Fase 9 | unitário |
| RF-13 | Persistência | DATA_MODEL | Fase 2 | integração/migrations |
| RF-14 | Import/export | DATA_MODEL + BENCH_DATA_WORKSPACE | Etapa 7C e Fase 12 | integração |
| RF-15 | Cadastro assistido multimodal | ASSISTED_INGESTION + MANUFACTURER_SCRAPING + PRODUCT_SPEC §11 | Fases 5 e 6 | segurança + schema + integração/UI |
| RF-16 | Workspace Bancada | BENCH_DATA_WORKSPACE + PRODUCT_SPEC §12 + UX_SPEC | Etapas 7A–7E | unitário + integração + UI |
| RF-17 | Recomendação de propulsão orientada ao uso | PROPULSION_RECOMMENDATION + PRD §8.6 | Etapas 8D–8H e 10C | unitário + integração + ranking/UI |
| RF-18 | Alcance FPV / RF Link Budget | FPV_LINK_BUDGET + PRD §8.7 | Etapas 9A–9G e 10D | unitário + integração + UI |

## Requisitos não funcionais

| Req. | Resumo | Evidência esperada |
|---|---|---|
| RNF-01 | Determinismo | testes do calculation engine, ranking e RF link budget |
| RNF-02 | Testabilidade | domínio/motor/recomendador/RF sem React e suite unitária |
| RNF-03 | Rastreabilidade | formulaId/modelVersion/proveniência/evidência por campo + perfil/versão/motivos do ranking + inputs RF |
| RNF-04 | Performance | medições quando UI/análise/ingestão/recomendador estiverem funcionais; link budget básico é cálculo determinístico de baixo custo |
| RNF-05 | Acessibilidade | UX spec + testes/inspeção, inclusive Bancada, ranking e resultados RF |
| RNF-06 | Internacionalização | regras sem dependência de strings pt-BR |
| RNF-07 | Evolução | perfis/data-driven, módulos desacoplados, AI Provider substituível e núcleo RF extensível |
| RNF-08 | Segurança de dados/ingestão | secrets ausentes, anti-SSRF, schema validation, staging e prompt-injection boundaries |

## Cenários de aceite de produto

### AC-01 — Projeto por estilo

**Dado** usuário criando novo projeto  
**Quando** escolhe Mini Long Range e ativa sub-250 g  
**Então** ambos ficam registrados separadamente: perfil e constraint.

Rastreia: RF-01, RF-08.

### AC-02 — Massa incompleta

**Dado** componente sem massa  
**Quando** análise é executada  
**Então** o sistema não soma 0 silenciosamente e informa dado ausente.

Rastreia: RF-03, RF-10, RNF-03.

### AC-03 — Tensão de bateria

**Dado** bateria e ESC  
**Quando** tensão cheia excede limite do ESC  
**Então** emitir `danger` com valores e regra.

Rastreia: RF-04, RF-05, RF-09.

### AC-04 — Curva de bancada

**Dado** motor, hélice e curva válida  
**Quando** hover exige ponto entre duas amostras  
**Então** interpolar dentro da curva e marcar origem `interpolated`.

Rastreia: RF-06, RF-10, RF-16.

### AC-05 — Sem extrapolação

**Dado** hover requerido fora da faixa testada  
**Quando** análise de propulsão é executada  
**Então** não extrapolar por padrão e informar indisponibilidade/limitação.

Rastreia: RF-06, RF-10, RF-16.

### AC-06 — Autonomia

**Dado** bateria e corrente média calculável  
**Quando** autonomia é estimada  
**Então** mostrar valor, hipótese de capacidade utilizável, origem e confiança.

Rastreia: RF-07, RF-10.

### AC-07 — C-rating

**Dado** bateria com C-rating e sem corrente medida  
**Quando** limite de corrente é derivado  
**Então** indicar que o limite vem do rating declarado/teórico, não de medição.

Rastreia: RF-04, RF-05, RF-10.

### AC-08 — Mini Long Range

**Dado** duas configurações, uma com maior empuxo máximo e outra mais eficiente no regime relevante  
**Quando** avaliadas para Mini Long Range  
**Então** o score deve refletir maior peso de eficiência/autonomia e explicar o resultado.

Rastreia: RF-08, RF-17.

### AC-09 — Danger e score

**Dado** configuração com score teórico alto e incompatibilidade `danger`  
**Quando** ranking é exibido  
**Então** o score não pode mascarar a incompatibilidade.

Rastreia: RF-08, RF-09, RF-17.

### AC-10 — Persistência

**Dado** projeto salvo  
**Quando** aplicação é reaberta  
**Então** projeto e referências necessárias são restaurados sem perda a partir da persistência autoritativa/cache conforme disponibilidade.

Rastreia: RF-01, RF-13.

### AC-11 — Import inválido

**Dado** arquivo JSON/CSV inválido  
**Quando** importado  
**Então** dados existentes permanecem intactos e o erro é explicado.

Rastreia: RF-14, RNF-08.

### AC-12 — Tema NEXO

**Dado** usuário alternando Claro/Escuro/Sistema  
**Quando** tema muda  
**Então** componentes mantêm tokens NEXO, contraste e semântica de status.

Rastreia: RNF-05 e NEXO_DESIGN_SYSTEM.

### AC-13 — Cadastro por URL com imagem técnica

**Dado** uma página de equipamento contendo ficha técnica em imagem  
**Quando** o import é executado com provider visual compatível  
**Então** campos reconhecidos são gravados apenas em staging, validados por schema e ligados ao asset/evidência de origem.

Rastreia: RF-02, RF-10, RF-15, RNF-03, RNF-08.

### AC-14 — Modelo sem visão

**Dado** um import que exige análise de imagem e um modelo sem capability visual  
**Quando** a etapa multimodal é solicitada  
**Então** o sistema informa indisponibilidade/falha explícita e não registra dados inventados.

Rastreia: RF-15, RNF-07, RNF-08.

### AC-15 — Conflito HTML × imagem

**Dado** HTML declarando `60 mΩ` e imagem declarando `64 mΩ` para o mesmo campo/condição  
**Quando** a reconciliação é executada  
**Então** o sistema preserva ambos, normaliza unidades e cria conflito `needs_review` sem escolher silenciosamente um valor.

Rastreia: RF-10, RF-15, RNF-03.

### AC-16 — Extraído versus derivado

**Dado** `KV = 1860` extraído de uma ficha  
**Quando** o Physics Engine calcula `Kt`  
**Então** KV permanece evidência de fonte e Kt recebe proveniência de cálculo/formulaId; o segundo não é apresentado como declaração do fabricante.

Rastreia: RF-06, RF-10, RF-15, RNF-03.

### AC-17 — Proteção anti-SSRF

**Dado** URL pública que redireciona para loopback/rede privada  
**Quando** o fetcher processa redirects  
**Então** o destino é bloqueado e nenhum conteúdo interno é entregue ao extractor/IA.

Rastreia: RF-15, RNF-08.

### AC-18 — Potência e eficiência de bancada

**Dado** uma amostra com tensão, corrente e empuxo válidos da mesma condição  
**Quando** a Bancada calcula valores derivados  
**Então** `P = V × I` é identificado como cálculo e `thrust_gf / P_W` aparece como **eficiência estática (gf/W)**, nunca como percentual.

Rastreia: RF-06, RF-10, RF-16, RNF-01, RNF-03.

### AC-19 — Tensão ausente

**Dado** uma curva com corrente/empuxo/RPM, mas sem tensão por ponto ou tensão de fonte explicitamente declarada  
**Quando** potência/eficiência derivada é solicitada  
**Então** o sistema não inventa tensão nominal e informa dado insuficiente/warning apropriado.

Rastreia: RF-10, RF-12, RF-16, RNF-03.

### AC-20 — Pitch speed não é velocidade de voo

**Dado** pitch da hélice e RPM disponíveis  
**Quando** a métrica geométrica é calculada  
**Então** ela é exibida apenas como **Velocidade teórica de passo**, com aviso de que não representa velocidade real/máxima da aeronave.

Rastreia: RF-10, RF-16, RNF-03, RNF-05.

### AC-21 — Gráfico de bancada

**Dado** curva contendo corrente, empuxo, RPM e `gf/W`  
**Quando** o usuário abre a visualização  
**Então** a UI não usa por padrão uma única escala Y para essas grandezas incompatíveis e mantém tabela acessível com unidades explícitas.

Rastreia: RF-12, RF-16, RNF-05.

### AC-22 — Massa por candidato

**Dado** dois motores candidatos com massas diferentes  
**Quando** o recomendador avalia ambos  
**Então** cada variante recebe sua própria massa de decolagem e seu próprio empuxo requerido de hover; o sistema não usa uma massa-base fixa que ignore os motores candidatos.

Rastreia: RF-03, RF-17, RNF-01.

### AC-23 — Eficiência versus empuxo máximo

**Dado** dois conjuntos compatíveis para Mini Long Range, um com maior empuxo máximo e outro mais leve/eficiente no regime relevante  
**Quando** ambos atendem ao TWR/reserva mínima  
**Então** o segundo pode ocupar posição superior e a UI explica a diferença de consumo, massa, autonomia e reserva.

Rastreia: RF-08, RF-17, RNF-03.

### AC-24 — Cobertura e confiança separadas do score

**Dado** candidato com score físico aparentemente alto mas dados incompletos/estimados  
**Quando** o ranking é exibido  
**Então** score, cobertura de dados e confiança aparecem separadamente e a falta de dados não é convertida silenciosamente em vantagem numérica.

Rastreia: RF-10, RF-17, RNF-03.

### AC-25 — Fase operacional sem dados

**Dado** perfil operacional contendo hover e cruzeiro, mas sem modelo defensável para cruzeiro  
**Quando** consumo ponderado é solicitado  
**Então** o cruzeiro não é tratado como corrente zero; a cobertura é marcada como incompleta e o resultado é limitado/indisponível conforme contrato.

Rastreia: RF-07, RF-08, RF-17, RNF-03.

### AC-26 — Link budget na distância-alvo

**Dado** frequência, potência VTX, ganhos/perdas, sensibilidade e margem desejada válidos  
**Quando** o operador informa uma distância-alvo  
**Então** o sistema calcula FSPL, potência recebida, margem disponível e headroom, indicando `pass/borderline/fail` conforme regra versionada.

Rastreia: RF-12, RF-18, RNF-01, RNF-03.

### AC-27 — Solver de potência VTX

**Dado** distância-alvo, frequência, antenas/perdas, sensibilidade e margem  
**Quando** o usuário solicita a potência necessária  
**Então** o sistema retorna potência teórica mínima em dBm e mW e informa que isso não representa autorização regulatória nem garantia de alcance real.

Rastreia: RF-18, RNF-03, RNF-05.

### AC-28 — Realized gain sem dupla perda

**Dado** antena com ganho marcado como `realized-gain` e SWR/eficiência também disponíveis  
**Quando** o link budget é calculado  
**Então** o sistema não desconta novamente perdas já incluídas no realized gain e registra a decisão semântica.

Rastreia: RF-10, RF-18, RNF-01, RNF-03.

### AC-29 — Gain kind desconhecido

**Dado** ganho de antena sem indicação se é directivity/gain/realized-gain  
**Quando** SWR ou eficiência separados são informados  
**Então** o sistema não aplica perdas potencialmente duplicadas silenciosamente e emite warning para revisão.

Rastreia: RF-10, RF-18, RNF-03.

### AC-30 — Diversity simplificado

**Dado** óculos/VRX com duas antenas em diversity de seleção  
**Quando** o cálculo é executado  
**Então** cada branch é avaliado separadamente e o sistema não soma os ganhos das duas antenas como array coerente.

Rastreia: RF-18, RNF-01, RNF-03.

### AC-31 — Espaço livre não é alcance garantido

**Dado** qualquer distância máxima derivada de FSPL  
**Quando** exibida ao usuário  
**Então** o rótulo informa **distância teórica máxima em espaço livre** e a UI apresenta limitações de obstáculos, orientação, multipath/interferência e status regulatório não verificado.

Rastreia: RF-18, RNF-03, RNF-05.

### AC-32 — Vídeo e rádio-controle separados

**Dado** projeto Long Range com link FPV calculado  
**Quando** a análise é exibida  
**Então** o sistema não afirma que o rádio-controle possui o mesmo alcance e não funde os dois links silenciosamente.

Rastreia: RF-18, RNF-03, RNF-07.

## Atualização da matriz

Ao adicionar requisito no PRD:

1. adicionar linha aqui;
2. apontar spec responsável;
3. apontar etapa ou issue;
4. definir evidência/teste;
5. não marcar como concluído apenas porque existe UI; regras técnicas exigem teste do motor/pipeline correspondente.
