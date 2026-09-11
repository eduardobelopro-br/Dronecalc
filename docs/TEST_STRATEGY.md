# Estratégia de Testes — DroneCalc

**Versão:** 0.4

## 1. Objetivo

O DroneCalc produz resultados técnicos; portanto testes numéricos, testes da Bancada, do recomendador, do link budget RF e da integridade da ingestão são parte do produto, não apenas da implementação.

Prioridade:

1. motor de cálculo;
2. validação de dados e unidades;
3. dados/curvas de bancada;
4. RF link budget;
5. recomendação/scoring;
6. ingestão/staging/proveniência;
7. migrações/persistência;
8. integração entre análise e projeto;
9. UI e acessibilidade;
10. regressão visual crítica.

## 2. Pirâmide de testes

### Unitários — maioria

- conversões físicas;
- fórmulas;
- potência `V × I`;
- eficiência estática `gf/W`;
- pitch speed teórica;
- interpolação;
- compatibilidade;
- scoring;
- mW ↔ dBm;
- FSPL;
- SWR → mismatch loss;
- link budget direto;
- solver inverso de distância/potência;
- semântica de antenna gain;
- normalização;
- validators;
- parsers/extractors determinísticos;
- reconciliação de fontes;
- migrações puras.

### Integração

- `DroneProject` → análise completa;
- catálogo → resolução de componente;
- bench test aprovado → lookup/interpolação → análise;
- import CSV de bancada → preview → validação → persistência;
- VTX + antena TX + VRX/óculos + antena RX → link budget;
- projeto com `targetFpvRangeKm` → análise RF;
- recommendation candidate → massa/physics/profile → ranking;
- import JSON/CSV → domínio;
- URL/import job → staging;
- asset → object storage + metadados;
- AI provider → schema validation → staging;
- repository → persistência;
- profile → scoring.

### UI

- criação de projeto;
- adicionar/substituir componente;
- seção Bancada: tabela, edição/importação, warnings, proveniência e gráficos;
- alcance FPV: distância, equipamentos, resultado, warnings e “Como foi calculado?”;
- cadastro assistido e revisão de campos extraídos;
- exibir conflitos/evidências;
- exibir dados ausentes;
- abrir “Como foi calculado?”;
- alertas;
- comparação;
- tema/acessibilidade.

### E2E

Poucos fluxos críticos após MVP funcional.

## 3. Ferramentas sugeridas

- Vitest;
- Testing Library;
- Playwright para E2E futuro;
- axe ou ferramenta equivalente para verificações automatizadas de acessibilidade.

A escolha pode mudar via decisão arquitetural.

## 4. Precisão numérica

Nunca comparar floats complexos com igualdade estrita quando arredondamento estiver envolvido.

Usar tolerância explícita:

```ts
expect(actual).toBeCloseTo(expected, precision)
```

A tolerância deve refletir a fórmula, não mascarar erros.

Para cálculos logarítmicos RF, documentar tolerância em dB e evitar converter repetidamente linear↔log com arredondamento precoce.

## 5. Separar cálculo de apresentação

Testes do motor usam valores não arredondados. Arredondamento de UI é testado na camada de formatting.

Nunca arredondar cedo dentro do cálculo intermediário sem necessidade física/documentada.

## 6. Casos de referência obrigatórios

### Massa

- uma peça;
- quantidades;
- múltiplas categorias;
- massa ausente;
- massa zero válida quando aplicável;
- massa negativa inválida.

### Bateria

- mAh → Ah;
- tensão nominal;
- tensão cheia;
- Wh;
- C-rating;
- química com valores por célula diferentes;
- capacidade inválida.

### Potência elétrica

- V × A;
- entradas medidas;
- zero/negativos inválidos conforme contexto;
- potência declarada/medida divergente de `V × I` gera warning sem sobrescrever fonte.

### Bancada — eficiência estática

- empuxo `gf` + potência W → `gf/W`;
- divisão por zero bloqueada;
- potência ausente indisponibiliza cálculo;
- valor é rotulado com unidade;
- não é convertido para percentual;
- origem `calculated` quando derivado.

### Bancada — pitch speed

- `pitch_in → pitch_m`;
- `pitch_m × RPM → m/min`;
- conversão correta para km/h;
- pitch zero/inválido conforme contrato;
- RPM negativo inválido;
- resultado sempre identificado como **Velocidade teórica de passo**;
- teste de UI impede rótulo `Velocidade máxima`/`Velocidade do drone`;
- aviso `PITCH_SPEED_NOT_AIRCRAFT_SPEED` ou equivalente permanece visível/contextual.

### Bancada — RPM sem carga

- `KV × V` como referência ideal;
- não usado como RPM carregado;
- RPM de bancada pode ser comparado à referência apenas com tolerância/semântica adequada;
- `loaded_rpm_ratio` não é chamado de eficiência automaticamente.

### TWR

- quad simétrico;
- diferentes números de motores;
- empuxo insuficiente;
- massa zero bloqueada.

### Interpolação

- ponto exato inferior;
- ponto exato superior;
- ponto intermediário;
- múltiplos segmentos;
- alvo fora da faixa;
- dados duplicados;
- curva não ordenada;
- amostra inválida;
- ponto interpolado preserva referência aos pontos de origem;
- nenhum caminho padrão extrapola silenciosamente.

### Hover

- massa divisível igualmente;
- required thrust encontrado exato;
- required thrust interpolado;
- fora da curva;
- motor count inválido;
- lookup por empuxo não assume linearidade de throttle.

### Autonomia

- capacidade utilizável;
- reserva;
- corrente média;
- carga auxiliar;
- impedir reserva dupla;
- corrente zero inválida.

### CG

- massas simétricas → centro;
- peso deslocado;
- 3 eixos;
- posição ausente.

### RF — potência linear/log

Obrigatórios:

```text
1 mW = 0 dBm
10 mW = 10 dBm
100 mW = 20 dBm
200 mW ≈ 23.0103 dBm
1000 mW = 30 dBm
```

Cobrir:

- mW → dBm;
- dBm → mW;
- round-trip dentro de tolerância;
- potência `<= 0 mW` inválida;
- valores extremos não produzem `NaN`/infinito silencioso.

### RF — FSPL

Usar fixtures documentadas e testar invariantes:

- distância/frequência positivas;
- dobrar distância mantendo frequência aumenta FSPL em aproximadamente `6.0206 dB`;
- dobrar frequência mantendo distância aumenta FSPL em aproximadamente `6.0206 dB`;
- forma `km/MHz` é equivalente à forma SI dentro da tolerância;
- solver inverso recupera a distância original.

### RF — link budget

Cobrir:

- soma/subtração correta de `Pt + Gt + Gr - losses - FSPL`;
- sensibilidade negativa em dBm tratada corretamente;
- `availableMargin = Pr - sensitivity`;
- `headroom = availableMargin - desiredMargin`;
- `pass/borderline/fail` nos limites exatos;
- sensibilidade ausente → `insufficient-data`;
- margem desejada ausente segue política explícita e nunca default oculto no engine.

### RF — solver de potência

- potência calculada fecha exatamente o link na distância/margem alvo dentro da tolerância;
- conversão final para mW;
- resultado não recebe automaticamente status legal/regulatório;
- potência exigida acima do VTX selecionado gera warning.

### RF — SWR/mismatch

Fixtures obrigatórias:

```text
SWR 1.0 → 0 dB
SWR 1.5 → ~0.1773 dB
```

Cobrir:

- SWR `< 1` inválido;
- SWR muito alto permanece numericamente estável;
- mismatch só é aplicado quando semanticamente permitido.

### RF — semântica de ganho

- `directivity` pode receber eficiência/mismatch separados quando fornecidos;
- `gain` não recebe eficiência de radiação novamente;
- `realized-gain` não recebe novamente mismatch/eficiência já embutidos;
- `unknown` + SWR/eficiência separados gera warning em vez de dupla contagem silenciosa;
- cabo integrado já incluído no ganho medido não é descontado novamente quando metadado indicar isso.

### RF — diversity

- branches calculados separadamente;
- diversity de seleção escolhe melhor branch calculado conforme contrato;
- ganhos não são somados como array coerente;
- dados incompletos de um branch não contaminam silenciosamente o outro;
- limitation warning permanece presente.

### RF — analógico/digital

- sensibilidade preserva condição/mode;
- modo digital diferente pode selecionar sensibilidade diferente;
- analógico não usa threshold como garantia binária de qualidade real;
- ausência de condição relevante reduz confiança/cobertura.

## 7. Compatibilidade e warnings

Cada código de warning deve ter testes independentes.

Exemplos gerais:

- `BATTERY_ESC_VOLTAGE_EXCEEDED`;
- `MOTOR_VOLTAGE_EXCEEDED`;
- `ESC_CURRENT_EXCEEDED`;
- `ESC_CURRENT_MARGIN_LOW`;
- `BATTERY_CURRENT_MARGIN_LOW`;
- `PROP_FRAME_DIAMETER_EXCEEDED`;
- `INSUFFICIENT_DATA_*`.

Warnings de Bancada:

- `BENCH_VOLTAGE_MISSING`;
- `BENCH_POWER_INCONSISTENT`;
- `BENCH_EFFICIENCY_UNIT_AMBIGUOUS`;
- `BENCH_THRUST_UNIT_AMBIGUOUS`;
- `BENCH_RPM_EXCEEDS_NO_LOAD_REFERENCE`;
- `BENCH_SAMPLE_DUPLICATE`;
- `BENCH_SAMPLE_INVALID`;
- `BENCH_INTERPOLATION_OUT_OF_RANGE`;
- `PITCH_SPEED_NOT_AIRCRAFT_SPEED`.

Warnings RF mínimos:

- `RF_TARGET_DISTANCE_INVALID`;
- `RF_FREQUENCY_INVALID`;
- `RF_RX_SENSITIVITY_MISSING`;
- `RF_LINK_MARGIN_MISSING`;
- `RF_ANTENNA_GAIN_KIND_UNKNOWN`;
- `RF_POSSIBLE_DOUBLE_COUNTED_ANTENNA_LOSS`;
- `RF_SWR_INVALID`;
- `RF_POLARIZATION_UNKNOWN`;
- `RF_POLARIZATION_MISMATCH`;
- `RF_FREE_SPACE_ONLY_MODEL`;
- `RF_DIVERSITY_MODEL_SIMPLIFIED`;
- `RF_REQUIRED_TX_POWER_EXCEEDS_SELECTED_VTX`;
- `RF_DATA_COVERAGE_INCOMPLETE`;
- `RF_REGULATORY_STATUS_NOT_CHECKED`.

Testar limites exatamente iguais, imediatamente abaixo e imediatamente acima quando a regra tiver threshold.

## 8. Perfis, recomendação e scoring

Para cada perfil:

- pesos carregam e validam;
- normalização soma corretamente;
- incompatibilidade crítica aplica regra definida;
- score explica contribuições;
- dados ausentes reduzem cobertura/confiança;
- Mini Long Range favorece eficiência/autonomia sobre potência bruta em fixtures apropriadas.

Para recomendação de propulsão:

- cada candidato recalcula sua massa;
- motor mais pesado aumenta hover requerido;
- maior empuxo não vence automaticamente perfil de eficiência;
- `danger` invalida candidato;
- score/cobertura/confiança são independentes;
- ranking é determinístico e explicável.

## 9. Golden fixtures

Manter fixtures legíveis em `tests/fixtures/`:

```text
tests/fixtures/
├── projects/
├── components/
├── bench-curves/
├── rf/
├── ingestion/
└── profiles/
```

Criar referências:

- `minimal-valid-quad`;
- `mini-long-range-reference`;
- `electrical-incompatible`;
- `missing-bench-data`;
- `sub250-overweight`;
- `bench-valid-with-voltage`;
- `bench-missing-voltage`;
- `bench-ambiguous-units`;
- `bench-interpolation-out-of-range`;
- `rf-basic-5g8-link`;
- `rf-realized-gain-no-double-loss`;
- `rf-diversity-selection`;
- `rf-missing-sensitivity`.

Fixtures de ingestão devem usar conteúdo próprio/licenciado e não depender de internet pública no CI.

Não copiar planilhas/imagens de terceiros como fixture redistribuível sem permissão. Quando uma referência externa inspirar um caso, criar fixture sintética equivalente.

## 10. Regressão do motor

Quando alterar uma fórmula:

1. adicionar/alterar teste que demonstra o motivo;
2. incrementar versão do modelo se resultados mudarem materialmente;
3. executar fixtures de regressão;
4. documentar diferença esperada;
5. não atualizar snapshots numericamente sem revisar a causa.

Isso se aplica também a fórmulas RF (`FSPL`, SWR, conversões, solver inverso).

## 11. Property-based testing futuro

Útil para invariantes:

- massa total não diminui ao adicionar massa positiva;
- energia Wh cresce linearmente com capacidade mantendo tensão;
- potência DC cresce linearmente com corrente mantendo tensão;
- pitch speed teórica cresce linearmente com RPM mantendo pitch;
- interpolação retorna valor entre extremos para curva monotônica;
- autonomia básica diminui quando corrente aumenta mantendo demais entradas;
- TWR diminui quando massa aumenta mantendo empuxo;
- FSPL cresce monotonicamente com distância/frequência positivas;
- potência VTX requerida cresce com distância mantendo demais entradas;
- maior ganho efetivo reduz potência requerida mantendo demais entradas.

## 12. Importação de arquivos e Bancada

Testes de JSON/CSV:

- formato válido;
- schema desconhecido;
- migration válida;
- migration que falha sem persistir parcial;
- CSV com vírgula/ponto decimal;
- unidades diferentes;
- cabeçalhos mapeados;
- linhas inválidas;
- arquivo vazio;
- duplicatas.

Para CSV de bancada:

- preview antes de persistir;
- cancelamento não altera banco;
- `Aceleração` exige mapeamento para `throttlePercent` sem mudar semântica;
- `Empuxo (100g)` permanece ambíguo até confirmação;
- `Eficiência` sem unidade não é assumida como percentual nem `gf/W`;
- tensão ausente não é substituída silenciosamente;
- derivados só aparecem com entradas válidas.

## 13. Ingestão por URL e multimodal

Seguir `ASSISTED_INGESTION.md` e `MANUFACTURER_SCRAPING.md`.

### Segurança de fetch

Testar:

- protocolos não permitidos;
- loopback IPv4/IPv6;
- redes privadas/link-local;
- redirect público → privado;
- rebinding/revalidação quando aplicável;
- excesso de redirects;
- timeout;
- payload acima do limite;
- MIME inconsistente;
- assets excessivos;
- falha parcial sem publicação indevida.

### Extração determinística

- JSON-LD;
- tabela HTML;
- unidades;
- campo ausente permanece ausente;
- valor ambíguo não vira número silenciosamente;
- raw/normalized preservados.

### IA textual/visual

Usar adapter fake nos testes principais; não depender de Ollama real no CI unitário.

Cobrir schema inválido, timeout, provider indisponível, modelo sem visão, staging, prompt injection e ausência de acesso SQL.

### Evidência e conflitos

- campo aponta para source/asset;
- HTML/imagem equivalentes não criam conflito falso;
- valores divergentes criam revisão;
- condições distintas não são achatadas;
- reprocessamento não sobrescreve publicação.

### Dados RF importados

- potência VTX mantém modo/condição;
- sensibilidade VRX mantém modo/condição;
- ganho da antena mantém frequência e `gainKind`;
- `2 dBi` sem contexto não recebe semântica inventada;
- SWR/eficiência extraídos ficam separados e rastreáveis.

## 14. Persistência

- save/get/list/delete;
- autosave;
- migrations;
- falha de storage sem corrupção;
- referências preservadas;
- staging/publicado separados;
- bench tests/samples íntegros;
- object storage/metadata coerentes;
- `targetFpvRangeKm` e configurações RF persistidas quando o usuário optar;
- resultados RF persistidos/exportados guardam versão e inputs necessários à reprodução.

## 15. UI

Testar comportamento, não implementação interna.

Exemplos:

- projeto vazio mostra próximo passo;
- número sem unidade não vira métrica final;
- dados insuficientes não aparecem como zero;
- `danger` tem texto além de cor;
- “Como foi calculado?” abre fórmula/inputs;
- Bancada mostra tabela/unidades/proveniência;
- pitch speed é teórica;
- gráfico não mistura unidades incompatíveis por padrão;
- pontos interpolados diferem de medidos;
- formulário RF valida km/MHz/dBm/dBi/dB corretamente;
- resultado RF exibe espaço livre explicitamente;
- `pass/borderline/fail` possui texto e números, não apenas cor;
- realized gain não gera perda duplicada na explicação;
- vídeo FPV não é rotulado como alcance de controle;
- potência requerida não é rotulada como “legal” ou “permitida”.

## 16. Temas

Testar:

- system;
- light;
- dark;
- persistência;
- ausência de cores hardcoded em componentes críticos.

## 17. Cobertura

Cobertura percentual isolada não é objetivo suficiente.

Requisitos:

- 100% das fórmulas/regras críticas possuem casos explícitos;
- branches de validação/warnings críticos cobertos;
- fixtures de referência existem.

Caminhos críticos:

- Bancada: `V×I`, `gf/W`, pitch speed, unidade ambígua, tensão ausente, extrapolação;
- ingestão: anti-SSRF, staging/publicação, schema;
- recomendador: massa por candidato, hard constraints, score/cobertura/confiança;
- RF: mW/dBm, FSPL, solver, SWR, gain semantics, sensitivity, diversity, free-space warning.

## 18. CI futuro

```text
install
→ typecheck
→ lint
→ unit tests
→ integration tests
→ build
```

Adicionar E2E conforme estabilidade. Testes de ingestão não devem depender de internet pública.

## 19. Critério de release

Não liberar versão estável se:

- teste crítico falha;
- fórmula mudou sem versão/documentação;
- import/migration pode causar perda silenciosa;
- estimativa aparece como medição;
- Bancada apresenta `gf/W` como percentual;
- pitch speed aparece como velocidade real/máxima;
- curva extrapola silenciosamente;
- gráfico de bancada usa escala única enganosa;
- IA publica sem staging/revisão;
- fetch remoto alcança rede interna;
- evidência de import é perdida;
- existe regressão de compatibilidade elétrica;
- link budget conta perdas de antena duas vezes;
- alcance RF é apresentado como garantido;
- potência teórica é apresentada como automaticamente autorizada;
- link FPV é confundido com rádio-controle.
