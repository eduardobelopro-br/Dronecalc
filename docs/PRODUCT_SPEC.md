# Product Spec — DroneCalc

**Versão:** 0.5  
**Status:** pré-MVP

Este documento descreve o comportamento funcional esperado. O PRD define o porquê; este documento define **o que a aplicação deve fazer**.

## 1. Estrutura principal da aplicação

A aplicação terá seis áreas principais:

1. **Projetos** — lista, criação, duplicação e gerenciamento.
2. **Builder** — montagem do drone por componentes, acesso à recomendação de propulsão e contexto do sistema FPV.
3. **Análise** — resultados técnicos, alertas e acesso ao cálculo de alcance FPV.
4. **Comparador** — comparação de variantes.
5. **Catálogo** — componentes, cadastro assistido, evidências e dados de fabricante.
6. **Bancada** — curvas motor+hélíce, amostras, importação, validação, visualização e integração com análise.

A navegação deverá manter o projeto ativo como contexto persistente quando aplicável.

## 2. Criação de projeto

Ao selecionar “Novo drone”, apresentar três opções:

- **Por estilo de voo**;
- **Montagem manual**;
- **Duplicar projeto existente**.

### 2.1 Por estilo de voo

Passos mínimos:

1. escolher perfil;
2. informar restrições;
3. visualizar metas técnicas derivadas;
4. iniciar projeto com essas metas associadas.

Campos iniciais:

- perfil principal;
- prioridade interna do perfil;
- peso máximo opcional;
- opção sub-250 g;
- payload adicional;
- autonomia desejada opcional;
- distância FPV desejada opcional;
- tamanho de hélice preferido/opcional;
- número de motores, inicialmente 4 por padrão, editável;
- química/tensão da bateria opcional;
- observações.

O usuário pode continuar sem preencher restrições não obrigatórias.

### 2.2 Montagem manual

Criar um projeto vazio. O usuário adiciona componentes. A análise deve aceitar projeto incompleto e informar quais dados faltam para cada cálculo.

## 3. Builder

Categorias mínimas do MVP:

- frame;
- motor;
- hélice;
- ESC;
- bateria;
- flight controller;
- receptor de controle;
- GPS;
- VTX;
- VRX/óculos quando usado como referência de projeto;
- antena FPV;
- câmera FPV;
- câmera de gravação;
- cabos/conectores;
- landing gear/proteções;
- payload;
- outros.

Cada item adicionado deve exibir:

- nome/modelo;
- quantidade;
- massa unitária e total;
- origem dos dados;
- indicador de completude dos campos necessários.

A massa do projeto deve ser recalculada sempre que uma peça for adicionada, removida, substituída ou tiver quantidade/massa alterada. Componente sem massa conhecida não pode ser tratado silenciosamente como `0 g`.

O Builder deve oferecer acesso à recomendação de propulsão quando existirem dependências suficientes para avaliar candidatos e deve permitir reutilizar VTX/antena selecionados no cálculo de alcance FPV.

## 4. Análise progressiva

A análise não deve exigir projeto completo.

Exemplos:

- se apenas massa estiver disponível, calcular peso;
- se bateria estiver disponível, calcular energia;
- se motor/ESC estiverem disponíveis, validar corrente/tensão conforme dados existentes;
- se curva motor+hélíce estiver disponível, liberar empuxo e autonomia mais confiável;
- se não houver curva, permitir estimativa apenas quando existir modelo explicitamente suportado e parâmetros suficientes;
- se houver componentes suficientes para gerar candidatos, permitir recomendação parcial/completa conforme cobertura;
- se houver dados RF suficientes, permitir link budget FPV mesmo que o projeto ainda esteja incompleto em outras áreas.

Nunca preencher dados técnicos ausentes silenciosamente.

## 5. Painel de resultados

Resultados mínimos:

### Massa
- massa seca;
- massa da bateria;
- payload;
- massa total de decolagem;
- distribuição percentual por categoria;
- indicação explícita de massa parcial quando houver componentes sem massa.

### Energia
- tensão nominal;
- tensão cheia;
- capacidade;
- energia nominal em Wh;
- corrente teórica por C-rating quando aplicável;
- corrente recomendada/medida quando fornecida.

### Propulsão
- empuxo por motor no ponto analisado;
- empuxo total;
- TWR;
- throttle estimado de hover quando suportado;
- eficiência estática `gf/W` quando dados permitirem;
- acesso à curva de bancada usada quando aplicável.

### Elétrica
- corrente por motor;
- corrente total estimada;
- potência por motor;
- potência total;
- margens de ESC e bateria.

### Autonomia
- hover;
- cruzeiro, quando modelo/dados suportarem;
- cenário de maior potência, quando suportado;
- capacidade utilizável considerada.

### Recomendação
Quando solicitada e suportada:
- conjunto motor + hélice + bateria/tensão;
- massa final específica do candidato;
- hover/TWR/reserva;
- consumo/eficiência/autonomia disponíveis;
- compatibilidade;
- score do perfil;
- cobertura dos dados;
- confiança;
- motivos do ranking.

### Alcance FPV
Quando solicitado e suportado:
- distância-alvo;
- frequência/canal;
- FSPL;
- potência recebida estimada;
- sensibilidade utilizada e condição;
- margem disponível;
- margem desejada;
- headroom residual;
- potência teórica mínima de VTX;
- EIRP quando calculável;
- distância teórica máxima em espaço livre;
- warnings e limitações.

## 6. Proveniência e confiança

Todo `CalculationResult` deve possuir:

- valor;
- unidade;
- origem;
- confiança;
- versão do modelo;
- entradas relevantes ou referência para elas;
- observações opcionais.

Origens aceitas incluem:

- `measured`;
- `manufacturer`;
- `calculated`;
- `interpolated`;
- `estimated`.

Confiança:

- `high`;
- `medium`;
- `low`.

A UI deve permitir abrir “Como foi calculado?”.

Dados de catálogo extraídos automaticamente também devem permitir abrir sua evidência/origem, sem confundir confiança da extração com confiança física do dado.

Para rankings, `profileScore`, `dataCoverage` e `confidence` devem ser apresentados separadamente. Para RF, a confiança final deve refletir sensibilidade, ganho/perdas e hipóteses utilizadas.

## 7. Alertas de compatibilidade

Severidades:

- `success` — verificação concluída e dentro da regra;
- `info` — observação técnica ou falta de dado não crítica;
- `warning` — baixa margem, hipótese fraca ou condição que merece revisão;
- `danger` — incompatibilidade conhecida ou limite excedido.

Cada alerta deve conter:

- título curto;
- explicação;
- componentes envolvidos;
- valor encontrado;
- limite/regra;
- recomendação objetiva quando possível;
- código estável para testes e traduções.

Uma incompatibilidade `danger` não pode ser compensada por score alto no recomendador.

Warnings RF devem distinguir falta de dados, margem insuficiente, semântica de ganho desconhecida, polarização, modelo de espaço livre e status regulatório não verificado.

## 8. Comparador

Comparar no mínimo duas configurações. Mostrar apenas métricas comparáveis e destacar diferenças relevantes.

Métricas iniciais:

- massa total;
- energia Wh;
- empuxo máximo quando suportado;
- TWR;
- corrente máxima;
- potência;
- hover estimado;
- autonomia;
- número e severidade dos alertas;
- score do perfil de voo, quando disponível;
- cobertura/confiança quando a comparação usar dados de qualidades diferentes;
- link budget FPV quando frequência, distância e condições forem comparáveis.

O comparador não deve declarar um “vencedor” sem explicar o critério.

## 9. Perfis de voo

Perfis são dados de configuração, não hardcode visual.

Cada perfil contém:

- identificador;
- nome;
- descrição;
- prioridades normalizadas;
- alvos/faixas preferidas;
- penalidades;
- campos de entrada recomendados;
- critérios para score;
- texto educativo.

A primeira versão suporta:

- `freestyle`;
- `racing`;
- `cinematic`;
- `long-range`;
- `mini-long-range`;
- `cinewhoop`.

O sistema pode associar um perfil operacional versionado para ponderar regimes de uso quando houver dados físicos defensáveis. Isso não é planejamento de rota/missão autônoma.

A distância FPV desejada pode ser uma constraint/intenção separada; perfis Long Range/Mini Long Range não devem inventar uma distância automaticamente.

## 10. Mini Long Range

Deve ser perfil próprio.

Prioriza:

- baixo peso;
- eficiência no ponto de hover/cruzeiro;
- autonomia;
- tamanho compacto;
- estabilidade;
- margem elétrica razoável.

Tamanho de hélice pode ter faixa preferida, mas não deve ser regra absoluta. A opção sub-250 g é uma restrição separada.

O score de Mini Long Range não pode favorecer empuxo máximo de forma desproporcional. Um candidato com menor TWR pode superar outro quando ambos atendem ao mínimo e o primeiro apresenta melhor compromisso de eficiência, autonomia e massa.

Alcance FPV desejado pode ser analisado separadamente, sem fundir link de vídeo com link de controle.

## 11. Catálogo

Um componente pode ser:

- oficial/importado e revisado;
- comunitário futuro;
- customizado pelo usuário.

Campos devem distinguir:

- valor nominal;
- valor máximo;
- valor medido;
- valor declarado;
- valor extraído automaticamente;
- valor derivado/calculado;
- fonte/evidência;
- condição associada ao valor;
- data/revisão opcional;
- notas.

### 11.1 Cadastro manual

O usuário pode criar/duplicar um componente e preencher os campos permitidos pela categoria.

### 11.2 Cadastro assistido por URL

O catálogo deve oferecer uma ação de cadastro assistido onde o usuário informa a URL da página do equipamento.

Fluxo esperado:

```text
URL
→ coleta segura
→ extração determinística
→ descoberta de imagens/documentos
→ IA textual/visual quando necessária e suportada
→ normalização
→ validação de schema
→ comparação entre fontes
→ staging
→ revisão
→ publicação
```

O comportamento detalhado é normatizado por `ASSISTED_INGESTION.md` e `MANUFACTURER_SCRAPING.md`.

### 11.3 Imagens técnicas

A aplicação deve poder utilizar imagens como fonte de dados quando o provider configurado possuir capacidade visual. Exemplos: ficha técnica, tabela, desenho dimensional, etiqueta e gráfico.

Para cada campo extraído visualmente, o sistema deve preservar asset, valor bruto/normalizado, unidade, método, provider/modelo, confiança e estado de revisão.

Modelo sem visão deve informar indisponibilidade, não fingir análise.

### 11.4 Conflitos

Se HTML, imagem ou documento divergirem, mostrar conflito para revisão. Não escolher silenciosamente o valor com maior confidence do modelo.

### 11.5 Extraído versus derivado

Exemplo: `KV = 1860` lido da ficha é dado extraído. `Kt` calculado a partir desse KV é derivado pelo Physics Engine e deve registrar fórmula/versão.

### 11.6 Catálogo RF

Categorias mínimas adicionais:

- VTX;
- VRX/FPV goggles;
- FPV antenna;
- RF cable/pigtail opcional.

Campos relevantes seguem `FPV_LINK_BUDGET.md`, incluindo frequência, potência, sensibilidade condicionada, ganho e `gainKind`, SWR, polarização e perdas de cabos/conectores.

## 12. Bancada e testes de propulsão

A seção **Bancada** é uma área própria da aplicação e deve seguir `BENCH_DATA_WORKSPACE.md`.

Um conjunto de teste motor/hélice deve incluir no mínimo motor/variante, hélice/variante, tensão/células, condições, fonte/evidência, amostras e status.

Amostras típicas:

- throttle (%);
- thrust;
- current;
- voltage;
- power;
- RPM;
- eficiência estática `gf/W` quando medida ou derivável.

### 12.1 Semântica obrigatória

- `throttle %` é comando do ESC, não aceleração física;
- `gf/W` é eficiência estática de empuxo, não eficiência energética percentual;
- potência derivada usa `P = V × I` da mesma amostra;
- tensão medida por ponto deve ser preferida quando disponível;
- ausência de tensão não autoriza assumir silenciosamente tensão nominal;
- `KV × V` é apenas referência de RPM ideal sem carga;
- `pitch × RPM` pode gerar **velocidade teórica de passo**, nunca velocidade real/máxima do drone.

### 12.2 Interpolação

Interpolação só pode ocorrer dentro do intervalo coberto pelos dados válidos/aprovados. Para hover, o lookup deve preferir empuxo requerido como variável-alvo.

### 12.3 Gráficos

Não usar por padrão uma única escala Y para grandezas incompatíveis como corrente, empuxo, RPM e `gf/W`.

### 12.4 Importação e IA

A seção pode receber dados por entrada manual, CSV/tabela, scraping, PDF/datasheet, imagem técnica e futura digitalização de gráfico revisada.

### 12.5 Integração com análise

Curva válida/aprovada pode alimentar:

```text
empuxo requerido por motor
→ lookup/interpolação
→ corrente + tensão + potência + RPM
→ TWR / hover / eficiência / autonomia
```

Se o alvo estiver fora da curva, retornar indisponibilidade/warning; não extrapolar silenciosamente.

## 13. Recomendação de propulsão orientada ao uso

A recomendação deve seguir `PROPULSION_RECOMMENDATION.md`.

A unidade de recomendação é:

```text
motor + hélice + bateria/tensão (+ ESC quando necessário)
```

Para cada candidato o sistema deve criar variante virtual, recalcular massa, recalcular empuxo requerido, verificar hard constraints, localizar ponto de operação, calcular métricas disponíveis e gerar ranking explicável.

Regras obrigatórias:

- maior empuxo máximo não vence automaticamente;
- eficiência é avaliada no regime relevante quando houver dados;
- fase sem dados não vira consumo zero;
- `danger` não é mascarado por score;
- `profileScore`, `dataCoverage` e `confidence` são separados;
- sem dados suficientes, mostrar `dados insuficientes`;
- aplicar candidato exige ação explícita do usuário.

## 14. Alcance FPV / RF Link Budget

A análise deve seguir `FPV_LINK_BUDGET.md`.

### 14.1 Entrada principal

O operador informa:

```text
Distância desejada (km)
Frequência/canal
Margem desejada
VTX/potência
Antena TX
VRX/óculos + sensibilidade
Antena(s) RX
Perdas adicionais opcionais
```

Componentes podem vir do catálogo ou ser preenchidos manualmente.

### 14.2 Cálculos

O calculation engine deve fornecer:

- mW ↔ dBm;
- FSPL;
- potência recebida;
- margem disponível;
- headroom após margem desejada;
- distância teórica máxima em espaço livre;
- potência teórica mínima de VTX para a distância-alvo;
- EIRP quando dados forem suficientes;
- mismatch loss a partir de SWR quando semanticamente aplicável.

### 14.3 Semântica de antena

O sistema deve distinguir:

```text
directivity
gain
realized-gain
unknown
```

Não descontar novamente eficiência/mismatch quando já estiverem incluídos no valor de ganho. Se a semântica for desconhecida e existirem perdas separadas, emitir warning em vez de assumir silenciosamente.

### 14.4 VRX/óculos

A sensibilidade deve preservar condição/mode. Não inventar sensibilidade a partir do nome do equipamento.

Para diversity de seleção:

- avaliar branches separadamente;
- mostrar melhor branch sob as hipóteses;
- não somar ganhos como array coerente.

### 14.5 Analógico versus digital

O link budget básico pode ser compartilhado, mas a interpretação da sensibilidade/qualidade muda. Em digital, usar sensibilidade correspondente ao modo selecionado. Em analógico, tratar threshold como condição específica, não como fronteira absoluta de qualidade.

### 14.6 Limitações obrigatórias

A UI deve exibir:

- **modelo de espaço livre**;
- obstáculos/terrain/multipath/interferência não modelados;
- orientação/polarização podem alterar o resultado;
- alcance de vídeo não é alcance do rádio-controle;
- potência teórica necessária não significa autorização regulatória.

### 14.7 Integração com projeto

`targetFpvRangeKm` pode ser persistido como constraint/intenção. VTX e antena do drone entram normalmente no orçamento de massa e energia. VRX/óculos do operador podem permanecer como equipamento de estação e não entram na massa do drone.

## 15. Unidades

Entradas e apresentação podem aceitar:

- g, kg, oz, lb;
- mm, cm, m, in;
- mAh, Ah;
- V;
- A;
- W, kW;
- Wh;
- N·m;
- RPM;
- gf/W;
- km/h apenas com semântica explícita;
- km/m para distância RF;
- MHz/GHz para frequência, com unidade canônica explícita;
- mW/W/dBm para potência RF;
- dB para razões/perdas e dBi para ganho de antena;
- g/gf e N para empuxo conforme contexto.

O motor utiliza unidades canônicas definidas no domínio. dBm, dB e dBi não são intercambiáveis.

## 16. Estados vazios e incompletos

A UI deve diferenciar:

- `não calculado porque faltam dados`;
- `não aplicável`;
- `estimativa indisponível`;
- `erro de entrada`;
- `extraído aguardando revisão`;
- `conflito de fontes`;
- `candidato incompatível`;
- `ranking com cobertura incompleta`;
- `link RF com dados insuficientes`;
- valor zero real.

Nunca representar todos esses estados como `0`.

## 17. Persistência

PostgreSQL é a fonte principal de dados persistidos do produto.

Requisitos:

- migrations versionadas;
- projects/catalog/staging/evidências/bench tests persistidos no backend;
- MinIO/S3-compatible para imagens/documentos grandes;
- IndexedDB apenas para cache, drafts, preferências e suporte offline auxiliar;
- autosave/drafts sem perda silenciosa;
- exportação JSON;
- importação validada antes de persistência autoritativa;
- recuperação segura após erro de parsing/processamento.

Avaliações temporárias de candidatos/link budget podem ser calculadas em memória/cache. Se persistidas/exportadas, guardar versões, inputs e referências suficientes para reprodução.

## 18. Acessibilidade

- foco visível;
- controles rotulados;
- estados não comunicados apenas por cor;
- tabelas com cabeçalhos semânticos;
- mensagens de validação associadas aos inputs;
- uso por teclado para fluxos principais;
- conflitos/evidências de import legíveis sem depender apenas de cor;
- gráficos da Bancada acompanhados de tabela/resumo acessível;
- ranking e motivos legíveis sem depender apenas de cor/ordem visual;
- estado RF `pass/borderline/fail` acompanhado de texto e valores.

## 19. Responsividade

Desktop é prioridade do MVP, pois comparação, Builder, Bancada, recomendação, RF link budget e revisão de imports exigem densidade de informação. Tablet deve permanecer utilizável. Mobile pode apresentar layout simplificado.

## 20. Critérios de aceite gerais

Uma funcionalidade só está concluída quando:

- comportamento está de acordo com esta spec;
- tipos estão validados;
- testes apropriados existem;
- unidade e proveniência aparecem nos resultados;
- estados de erro/incompleto foram considerados;
- dados extraídos por IA não pulam staging/revisão;
- evidência de campos importados é preservada;
- Bancada não apresenta pitch speed como velocidade real do drone;
- grandezas incompatíveis não são visualizadas numa escala única enganosa por padrão;
- recomendação recalcula massa por candidato e avalia hard constraints antes do score;
- score, cobertura e confiança não são misturados;
- ranking apresenta motivos/trade-offs;
- link budget não duplica perdas de antena nem inventa sensibilidade;
- alcance FPV é rotulado como estimativa de espaço livre;
- vídeo FPV e rádio-controle permanecem separados;
- status regulatório não é inferido sem módulo/dados específicos;
- identidade NEXO foi respeitada;
- documentação foi atualizada se houve mudança de contrato.
