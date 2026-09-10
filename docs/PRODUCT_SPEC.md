# Product Spec — DroneCalc

**Versão:** 0.4  
**Status:** pré-MVP

Este documento descreve o comportamento funcional esperado. O PRD define o porquê; este documento define **o que a aplicação deve fazer**.

## 1. Estrutura principal da aplicação

A aplicação terá seis áreas principais:

1. **Projetos** — lista, criação, duplicação e gerenciamento.
2. **Builder** — montagem do drone por componentes e acesso à recomendação de propulsão.
3. **Análise** — resultados técnicos e alertas.
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
- receptor;
- GPS;
- VTX;
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

O Builder deve oferecer acesso à recomendação de propulsão quando existirem dependências suficientes para avaliar candidatos.

## 4. Análise progressiva

A análise não deve exigir projeto completo.

Exemplos:

- se apenas massa estiver disponível, calcular peso;
- se bateria estiver disponível, calcular energia;
- se motor/ESC estiverem disponíveis, validar corrente/tensão conforme dados existentes;
- se curva motor+hélíce estiver disponível, liberar empuxo e autonomia mais confiável;
- se não houver curva, permitir estimativa apenas quando existir um modelo explicitamente suportado e parâmetros suficientes, marcando confiança adequada;
- se houver componentes suficientes para gerar candidatos, permitir recomendação parcial/completa conforme cobertura dos dados.

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

Para rankings, `profileScore`, `dataCoverage` e `confidence` devem ser apresentados separadamente.

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

Exemplo:

```text
ESC_CURRENT_MARGIN_LOW
Motor: 31 A
ESC contínuo: 35 A
Margem: 12,9%
Severidade: warning
```

Uma incompatibilidade `danger` não pode ser compensada por score de perfil alto no recomendador.

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
- cobertura/confiança quando a comparação usar dados de qualidades diferentes.

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

O sistema pode associar um perfil operacional versionado para ponderar regimes como hover/cruzeiro/subida/agressivo/reserva quando houver dados físicos defensáveis. Isso não é planejamento de rota/missão autônoma.

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

O score de Mini Long Range não pode favorecer empuxo máximo de forma desproporcional. Eficiência e energia útil devem pesar mais que potência bruta. Um candidato com menor TWR pode superar outro no ranking quando ambos atendem ao mínimo e o primeiro apresenta melhor compromisso de eficiência, autonomia e massa.

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

Para cada campo extraído visualmente, o sistema deve preservar, quando disponível:

- asset de origem;
- valor bruto;
- valor normalizado;
- unidade;
- método de extração;
- modelo/provider;
- confiança da extração;
- região da imagem opcional;
- estado de revisão.

Modelo sem visão deve informar indisponibilidade, não fingir análise.

### 11.4 Conflitos

Se HTML, imagem ou documento divergirem, mostrar conflito para revisão. Não escolher silenciosamente o valor com maior confidence do modelo.

### 11.5 Extraído versus derivado

Exemplo: `KV = 1860` lido da ficha é dado extraído. `Kt` calculado a partir desse KV é dado derivado pelo Physics Engine e deve registrar fórmula/versão. A UI e persistência não podem apresentá-los como se ambos tivessem sido declarados pelo fabricante.

## 12. Bancada e testes de propulsão

A seção **Bancada** é uma área própria da aplicação e deve seguir `BENCH_DATA_WORKSPACE.md`.

Um conjunto de teste motor/hélice deve incluir no mínimo:

- motor/variante;
- hélice/variante ou descrição estruturada;
- tensão/células ou tensão medida;
- condições/observações quando conhecidas;
- fonte/evidência;
- amostras ordenadas;
- status de revisão.

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
- `pitch × RPM` pode gerar **velocidade teórica de passo**, mas nunca deve ser apresentada como velocidade real/máxima do drone.

### 12.2 Velocidade teórica de passo

Quando pitch e RPM existirem:

```text
pitch_m = pitch_in × 0.0254
pitch_speed_m_min = pitch_m × RPM
pitch_speed_km_h = pitch_speed_m_min × 60 / 1000
```

A UI deve apresentar aviso de que esse valor ignora slip, arrasto, advance ratio e outros efeitos aerodinâmicos.

### 12.3 Interpolação

Interpolação só pode ocorrer dentro do intervalo coberto pelos dados válidos/aprovados, salvo modelo de extrapolação explicitamente documentado. O padrão é **não extrapolar**.

Para hover, o lookup/interpolação deve preferir empuxo requerido como variável-alvo, e não assumir linearidade em throttle.

### 12.4 Gráficos

Não usar por padrão uma única escala Y para grandezas incompatíveis como corrente, empuxo, RPM e `gf/W`.

Visualizações recomendadas:

- empuxo × throttle;
- corrente/potência × throttle;
- RPM × throttle;
- eficiência estática × throttle ou empuxo.

A tabela permanece representação auditável/acessível dos dados.

### 12.5 Importação e IA

A seção pode receber dados por:

- entrada manual;
- CSV/tabela;
- scraping de fabricante;
- PDF/datasheet;
- imagem técnica analisada por IA;
- digitalização de gráfico futura/revisada.

Curvas digitalizadas a partir de imagens devem manter essa proveniência e não recebem automaticamente o mesmo nível de confiança de dados tabulares originais.

### 12.6 Integração com análise

Curva válida/aprovada pode alimentar:

```text
empuxo requerido por motor
→ lookup/interpolação dentro da faixa
→ corrente + tensão + potência + RPM
→ TWR / hover / eficiência / autonomia
```

Se o alvo estiver fora da curva, retornar indisponibilidade/warning; não extrapolar silenciosamente.

## 13. Recomendação de propulsão orientada ao uso

A recomendação deve seguir `PROPULSION_RECOMMENDATION.md`.

A unidade de recomendação é o conjunto:

```text
motor + hélice + bateria/tensão (+ ESC quando necessário)
```

Para cada candidato o sistema deve:

1. criar uma variante virtual sem modificar o projeto do usuário;
2. recalcular massa seca/decolagem incluindo as próprias peças candidatas;
3. recalcular empuxo requerido por motor;
4. verificar frame/hélice, tensão, corrente, ESC/bateria e demais hard constraints;
5. localizar o ponto de operação em curva aprovada ou Physics Engine quando válido;
6. calcular consumo, eficiência, TWR e autonomia disponíveis;
7. aplicar prioridades do perfil de voo/uso;
8. gerar ranking explicável.

Regras obrigatórias:

- motor de maior empuxo máximo não vence automaticamente;
- eficiência é avaliada no regime relevante quando houver dados;
- fase operacional sem dados não é tratada como consumo zero;
- `danger` não é mascarado por score;
- `profileScore`, `dataCoverage` e `confidence` são separados;
- sem dados físicos suficientes, mostrar `dados insuficientes` em vez de inventar números;
- aplicar um candidato ao projeto exige ação explícita do usuário.

A UI deve permitir abrir `Por que foi recomendado?` e mostrar os principais trade-offs entre candidatos.

## 14. Unidades

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
- g/gf e N para empuxo conforme contexto de UI, com representação interna inequívoca.

O motor utiliza unidades canônicas definidas no domínio.

## 15. Estados vazios e incompletos

A UI deve diferenciar:

- `não calculado porque faltam dados`;
- `não aplicável`;
- `estimativa indisponível`;
- `erro de entrada`;
- `extraído aguardando revisão`;
- `conflito de fontes`;
- `candidato incompatível`;
- `ranking com cobertura incompleta`;
- valor zero real.

Nunca representar todos esses estados como `0`.

## 16. Persistência

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

Avaliações temporárias de candidatos podem ser calculadas em memória/cache. Se rankings forem persistidos/exportados, devem guardar versões de perfil/modelo e referências suficientes para reprodução.

## 17. Acessibilidade

- foco visível;
- controles rotulados;
- estados não comunicados apenas por cor;
- tabelas com cabeçalhos semânticos;
- mensagens de validação associadas aos inputs;
- uso por teclado para fluxos principais;
- conflitos/evidências de import legíveis sem depender apenas de cor;
- gráficos da Bancada acompanhados de tabela/resumo acessível;
- ranking e motivos legíveis sem depender apenas de cor/ordem visual.

## 18. Responsividade

Desktop é prioridade do MVP, pois comparação, builder, Bancada, recomendação e revisão de imports exigem densidade de informação. Tablet deve permanecer utilizável. Mobile pode apresentar layout simplificado, mas não é requisito de paridade completa no primeiro ciclo.

## 19. Critérios de aceite gerais

Uma funcionalidade só está concluída quando:

- comportamento está de acordo com esta spec;
- tipos estão validados;
- testes apropriados existem;
- unidade e proveniência aparecem nos resultados;
- estados de erro/incompleto foram considerados;
- dados extraídos por IA não pulam staging/revisão;
- evidência de campos importados é preservada;
- Bancada não apresenta pitch speed como velocidade real do drone;
- grandezas de unidades incompatíveis não são visualizadas numa escala única enganosa por padrão;
- recomendação recalcula massa por candidato e avalia hard constraints antes do score;
- score, cobertura e confiança não são misturados;
- ranking apresenta motivos/trade-offs;
- identidade NEXO foi respeitada;
- documentação foi atualizada se houve mudança de contrato.
