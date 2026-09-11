# PRD — DroneCalc

**Versão:** 0.5  
**Status:** pré-MVP  
**Produto:** DroneCalc  
**Repositório:** `eduardobelopro-br/Dronecalc`

## 1. Visão

DroneCalc é uma aplicação para apoiar o projeto de drones DIY por meio de cálculo técnico, comparação de configurações e validação preliminar de compatibilidade. O produto deve servir tanto ao usuário que já conhece componentes quanto à pessoa que sabe apenas o tipo de voo que deseja realizar.

A aplicação transforma componentes e objetivos de voo em uma análise explicável de massa, energia, potência, corrente, empuxo, relação empuxo/peso, autonomia, eficiência, centro de gravidade, margens elétricas e, quando configurado, link budget de vídeo FPV. Quando houver dados suficientes, também deve recomendar conjuntos de propulsão adequados ao peso real do projeto e ao uso pretendido.

## 2. Problema

A montagem DIY normalmente exige consultar múltiplas tabelas, páginas de fabricantes, imagens técnicas, testes de bancada e calculadoras isoladas. O usuário precisa combinar manualmente informações de frame, motor, hélice, ESC, bateria, eletrônica, payload, VTX, receptor e antenas, além de interpretar se a combinação faz sentido para o estilo de voo desejado.

Os principais problemas são:

- dados distribuídos e em unidades diferentes;
- especificações importantes disponíveis apenas em imagens, PDFs ou tabelas;
- dificuldade de comparar variantes de projeto;
- estimativas de autonomia frequentemente pouco transparentes;
- uso incorreto de valores de KV, corrente, C-rating e tensão;
- interpretação incorreta de dados de bancada, como confundir `gf/W` com eficiência percentual ou pitch speed com velocidade real do drone;
- escolha de motor baseada apenas em KV, empuxo máximo ou regra genérica, ignorando massa, hélice, tensão e ponto de operação;
- estimativas de alcance FPV que ignoram sensibilidade, margem de link, perdas e semântica do ganho da antena;
- risco de contar duas vezes perdas de eficiência/SWR ao usar ganho de antena sem saber se é `realized gain`;
- falta de rastreabilidade da origem de um resultado;
- recomendações genéricas que ignoram o objetivo de voo;
- dificuldade para iniciantes entenderem por que uma configuração é ou não adequada.

## 3. Proposta de valor

DroneCalc deve funcionar como uma **bancada virtual de dimensionamento**, permitindo experimentar antes de comprar ou montar.

O produto se diferencia por:

- combinar cálculo e catálogo de componentes;
- calcular automaticamente a massa conforme peças são adicionadas/substituídas;
- recomendar conjuntos motor + hélice + bateria/tensão com base no peso recalculado, compatibilidade, dados de propulsão e perfil de uso;
- calcular link budget de vídeo FPV a partir de distância-alvo, VTX, VRX/óculos, antenas, sensibilidade, margem e perdas;
- permitir cadastro assistido a partir de URL com extração de HTML, imagens e documentos técnicos;
- oferecer uma seção Bancada para cadastrar, revisar e visualizar curvas motor+hélíce;
- separar dado medido, dado de fabricante, dado extraído, cálculo, interpolação e estimativa;
- preservar evidência e revisão para dados importados;
- calcular com unidades canônicas e converter apenas na apresentação;
- utilizar curvas reais de bancada quando disponíveis;
- adaptar critérios ao estilo de voo;
- explicar alertas, margens e motivos do ranking;
- permitir configurações customizadas e dados próprios do usuário;
- manter uma identidade visual consistente com o NEXO Design System.

## 4. Público-alvo

### Primário

- entusiastas de drones DIY;
- pilotos FPV que montam ou modificam drones;
- makers e estudantes;
- usuários que desejam comparar peças antes da compra;
- usuários montando drones de Freestyle, Racing, Cinematic, Long Range, Mini Long Range e Cinewhoop.

### Secundário

- integradores de pequenos UAVs;
- professores e laboratórios educacionais;
- usuários de fotogrametria, payload e projetos experimentais.

## 5. Jobs to be Done

O usuário deve conseguir:

- descobrir uma configuração inicial a partir do estilo de voo;
- montar manualmente um drone peça por peça;
- saber o peso seco, operacional e com payload;
- receber atualização automática da massa ao adicionar/remover/substituir componentes;
- descobrir quais conjuntos de propulsão atendem ao peso e ao uso pretendido com melhor compromisso de consumo, eficiência, reserva e autonomia;
- verificar tensão e corrente entre bateria, ESC, motor e eletrônica;
- entender se há empuxo suficiente;
- estimar autonomia em cenários diferentes;
- informar quantos quilômetros pretende afastar o drone e avaliar se o link de vídeo FPV possui margem teórica suficiente;
- estimar a potência mínima teórica de VTX necessária para uma distância-alvo e configuração de antenas/VRX;
- comparar duas ou mais configurações;
- cadastrar/importar e revisar dados de bancada;
- visualizar curvas de empuxo, corrente, potência, RPM e eficiência estática com unidades explícitas;
- cadastrar componentes não existentes no catálogo;
- informar a página de um equipamento e obter um cadastro técnico pré-preenchido a partir de texto, tabelas, imagens e documentos disponíveis;
- revisar conflitos e evidências antes de publicar dados extraídos no catálogo;
- visualizar por que o sistema emitiu um alerta ou recomendou um candidato;
- saber o nível de confiança e cobertura de dados de um resultado;
- exportar ou compartilhar a configuração no futuro.

## 6. Objetivos do MVP ampliado

O MVP deve responder de forma confiável e explicável:

1. **Quanto pesa?**
2. **A alimentação elétrica é compatível?**
3. **A propulsão fornece empuxo adequado?**
4. **Qual autonomia pode ser esperada nas condições informadas?**
5. **Qual conjunto de propulsão é mais adequado ao peso e ao uso deste projeto entre os candidatos com dados disponíveis?**
6. **O link de vídeo FPV fecha, em espaço livre, na distância desejada com a margem configurada?**

Além disso, o MVP ampliado deve oferecer:

- criação e edição de projetos;
- perfis de voo;
- componentes customizados;
- catálogo persistente inicial;
- cadastro assistido por URL com staging/revisão;
- extração multimodal quando o provider/modelo configurado suportar visão;
- seção Bancada para curvas motor+hélíce e integração com análise;
- importação de CSV/tabela de bancada com preview e unidades explícitas;
- análise de compatibilidade;
- recomendação básica de propulsão orientada ao uso, com recálculo de massa por candidato e ranking explicável;
- calculadora de alcance FPV/link budget com solver inverso de potência e distância;
- comparação de variantes;
- PostgreSQL como fonte persistente principal;
- cache/drafts locais auxiliares;
- tema claro/escuro seguindo NEXO;
- testes automatizados do motor matemático, da Bancada, do recomendador, do link budget e do pipeline crítico de ingestão.

## 7. Fora do escopo do MVP

Não fazem parte do primeiro MVP:

- controle de voo em tempo real;
- comunicação com flight controller;
- configuração de Betaflight/ArduPilot/PX4;
- certificação de aeronavegabilidade;
- simulação CFD;
- cálculo estrutural por elementos finitos;
- previsão meteorológica;
- planejamento de rota/missão autônoma;
- terreno/DEM e ray tracing RF;
- modelagem avançada de zona de Fresnel/horizonte de rádio;
- cálculo regulatório automático por país;
- marketplace;
- otimização global de milhares de combinações em servidor;
- sincronização multiusuário;
- previsão de velocidade real/máxima da aeronave baseada apenas em pitch e RPM;
- unificação silenciosa de alcance FPV com alcance do rádio-controle.

A recomendação básica sobre candidatos do catálogo e o link budget FPV em espaço livre fazem parte do produto; otimização global, terreno RF e compliance regulatório permanecem posteriores ao MVP.

## 8. Modos de entrada

### 8.1 Projeto por estilo de voo

O usuário seleciona um perfil e informa restrições. O sistema transforma o objetivo em metas e critérios técnicos.

Perfis iniciais:

- Freestyle;
- Racing;
- Cinematic;
- Long Range;
- Mini Long Range;
- Cinewhoop.

Restrições independentes incluem:

- sub-250 g;
- tamanho máximo de hélice;
- tensão/células máximas;
- payload;
- autonomia mínima desejada;
- distância FPV desejada opcional;
- orçamento futuro;
- sistema FPV;
- química de bateria.

### 8.2 Montagem manual

O usuário escolhe ou cadastra cada componente. A análise é atualizada progressivamente e deve funcionar mesmo quando o projeto ainda está incompleto.

### 8.3 Duplicação/variante

Um projeto pode ser duplicado para experimentar uma alteração sem perder a configuração original.

### 8.4 Cadastro assistido de equipamento

Na área de catálogo, o usuário pode informar uma URL. O sistema coleta de forma controlada dados estruturados, texto, imagens e documentos elegíveis, produz candidatos validados por schema e os grava em staging.

A IA pode auxiliar na extração textual e visual, mas não publica diretamente no catálogo. O usuário/revisor deve conseguir ver evidência, conflitos e estado de revisão.

### 8.5 Dados de bancada

Na área Bancada, o usuário pode criar um ensaio manualmente, importar CSV/tabela ou revisar curvas recebidas pelo pipeline de ingestão.

O sistema deve preservar condições, unidades, fonte e estado de revisão por ensaio/amostra. Valores derivados devem ser identificados como cálculo, não como medição.

### 8.6 Recomendação de propulsão

A partir das peças já adicionadas e do perfil de uso, o usuário pode solicitar candidatos de propulsão. Para cada candidato, o sistema deve montar uma variante virtual, recalcular a massa, determinar o empuxo requerido por motor, verificar compatibilidade e avaliar consumo/eficiência/autonomia com os melhores dados disponíveis.

O comportamento detalhado é normatizado por `PROPULSION_RECOMMENDATION.md`.

### 8.7 Alcance FPV / Link Budget

O operador informa a distância-alvo e pode selecionar componentes já cadastrados ou fornecer manualmente:

- frequência/canal;
- potência de VTX;
- antena TX;
- VRX/óculos e sensibilidade;
- antena(s) RX;
- margem desejada;
- perdas adicionais conhecidas.

O sistema calcula FSPL, potência recebida, margens, distância teórica máxima em espaço livre, EIRP quando possível e potência teórica mínima de VTX para a distância-alvo.

O comportamento detalhado é normatizado por `FPV_LINK_BUDGET.md`.

## 9. Requisitos funcionais de alto nível

### RF-01 — Projetos
Criar, renomear, duplicar, editar e excluir projetos.

### RF-02 — Componentes
Selecionar componentes do catálogo e cadastrar componentes personalizados.

### RF-03 — Massa
Calcular massa por componente, categoria, massa seca, operacional e total de decolagem. A massa deve ser atualizada ao alterar componentes e recalculada separadamente para variantes/candidatos de propulsão.

### RF-04 — Energia
Calcular tensão nominal/cheia, energia em Wh e limites declarados da bateria.

### RF-05 — Sistema elétrico
Comparar tensões suportadas, corrente de motor, ESC, bateria e alimentação de eletrônica.

### RF-06 — Propulsão
Calcular empuxo total e relação empuxo/peso usando dados de bancada quando disponíveis e modelo físico apenas quando houver parâmetros suficientes.

### RF-07 — Autonomia
Estimar autonomia por modelo explícito e informar as hipóteses usadas.

### RF-08 — Perfis de voo
Aplicar critérios de avaliação diferentes conforme o objetivo do usuário.

### RF-09 — Compatibilidade
Gerar mensagens `success`, `info`, `warning` e `danger`, sempre com justificativa.

### RF-10 — Proveniência
Todo resultado deve indicar origem e confiança. Dados importados automaticamente devem preservar evidência suficiente para auditoria.

### RF-11 — Comparação
Comparar ao menos duas variantes lado a lado.

### RF-12 — Unidades
Aceitar e exibir unidades comuns sem misturar unidades internamente.

### RF-13 — Persistência
Persistir projetos, catálogo, revisões, evidências, staging e dados de bancada no backend PostgreSQL; IndexedDB/local storage ficam restritos a cache/drafts/preferências/offline auxiliar.

### RF-14 — Importação/exportação
Suportar formato JSON versionado; CSV será usado principalmente para tabelas de bancada.

### RF-15 — Cadastro assistido multimodal
Permitir iniciar cadastro de equipamento por URL, extrair primeiro fontes estruturadas e, quando necessário, usar IA para texto/imagens/documentos. Todo campo de IA deve passar por schema, staging e revisão antes de publicação. Conflitos entre fontes não podem ser resolvidos silenciosamente.

### RF-16 — Workspace Bancada
Permitir cadastrar, importar, revisar, visualizar e utilizar curvas de bancada motor+hélíce. A seção deve distinguir throttle, tensão, corrente, potência, empuxo, RPM e eficiência estática; derivar `P = V × I` e `gf/W` apenas com entradas defensáveis; tratar `pitch × RPM` somente como **velocidade teórica de passo**; não extrapolar curvas silenciosamente; e disponibilizar curvas aprovadas para análise de propulsão/hover/autonomia.

### RF-17 — Recomendação de propulsão orientada ao uso
Avaliar conjuntos motor + hélice + bateria/tensão a partir do projeto e perfil selecionado. Cada candidato deve recalcular sua própria massa de decolagem antes de calcular hover/TWR/consumo; hard constraints e incompatibilidades `danger` devem ser avaliados antes do score; eficiência deve ser considerada no regime relevante ao uso; score, cobertura de dados e confiança devem permanecer separados; e o ranking deve explicar por que cada conjunto foi recomendado.

### RF-18 — Alcance FPV / RF Link Budget
Permitir informar distância-alvo e calcular o link de vídeo FPV com frequência, potência de VTX, ganhos/perdas de antenas, sensibilidade do VRX e margem desejada. Deve calcular FSPL, potência recebida, margem, potência teórica mínima de VTX e distância teórica máxima em espaço livre; distinguir `directivity/gain/realized-gain`; evitar dupla contagem de perdas; tratar diversity de forma explícita; e nunca apresentar o resultado como alcance real garantido ou autorização regulatória.

## 10. Requisitos não funcionais

### RNF-01 — Determinismo
Mesmas entradas e mesma versão do motor devem produzir os mesmos resultados, ranking e link budget.

### RNF-02 — Testabilidade
O motor matemático, recomendador e RF link budget não podem depender de React ou DOM.

### RNF-03 — Rastreabilidade
Cálculos críticos devem registrar fórmula/modelo, entradas e versão do algoritmo. Imports automáticos e dados de bancada devem registrar fonte, método e evidência/proveniência quando aplicável. Rankings devem registrar perfil/versão, candidatos avaliados e razões determinantes. Resultados RF devem registrar frequência, distância, sensibilidade, ganhos/perdas e margem utilizadas.

### RNF-04 — Performance
Alterações simples do projeto devem atualizar a análise sem atraso perceptível em hardware desktop comum. Processos de ingestão/IA podem ser assíncronos. O recomendador inicial deve trabalhar com conjunto limitado de candidatos e o link budget básico deve ser cálculo local determinístico de baixo custo.

### RNF-05 — Acessibilidade
Navegação por teclado, foco visível, contraste adequado e semântica ARIA nos controles relevantes. Gráficos de bancada devem possuir tabela/resumo acessível. Resultados RF devem ser legíveis sem depender apenas de cor.

### RNF-06 — Internacionalização
Código não deve depender de texto em português para regras de negócio. UI inicial pode ser pt-BR.

### RNF-07 — Evolução
Novos tipos de componente, perfis, modelos RF e providers de IA não devem exigir reescrever o motor inteiro.

### RNF-08 — Segurança de dados e ingestão
Nenhum segredo deve ser armazenado em código ou exportações. Credenciais de PostgreSQL/MinIO/Ollama não são expostas ao frontend. Fetch remoto deve ser protegido contra SSRF e conteúdo remoto não pode alterar instruções/permissões do agente. IA não recebe acesso SQL direto.

## 11. Métricas de qualidade do produto

Antes de chamar o MVP de estável:

- 100% das fórmulas críticas cobertas por testes unitários;
- casos de referência validados manualmente;
- nenhum alerta crítico sem mensagem explicativa;
- nenhum cálculo apresentado sem unidade;
- nenhum valor estimado apresentado como medição;
- importação inválida não pode corromper projetos existentes;
- dado extraído por IA não pode ser publicado sem staging/revisão;
- controles críticos anti-SSRF e validação de schema possuem testes explícitos;
- curvas de bancada não podem extrapolar silenciosamente;
- `gf/W` deve aparecer com semântica/unidade explícita;
- pitch speed teórica não pode ser apresentada como velocidade real/máxima do drone;
- gráficos de bancada não devem usar escala única enganosa para grandezas incompatíveis;
- perfis de voo devem ser configuráveis por dados;
- ranking de propulsão deve recalcular massa por candidato;
- conjunto de maior empuxo máximo não pode vencer automaticamente perfis orientados à eficiência;
- incompatibilidade `danger` não pode ser mascarada por score;
- score, confiança e cobertura de dados devem ser apresentados separadamente;
- link budget deve validar dBm/mW, FSPL, SWR e solver inverso com fixtures conhecidas;
- realized gain não pode receber novamente perdas já embutidas;
- distância RF calculada deve ser rotulada como teórica em espaço livre;
- status regulatório deve permanecer `não verificado` até existir módulo específico.

## 12. Experiência desejada

O usuário deve conseguir começar pelo **objetivo**, não pela terminologia técnica. Ao mesmo tempo, um usuário avançado deve conseguir acessar todos os parâmetros, substituir valores e visualizar a origem do cálculo.

No catálogo, o usuário deve poder colar a página do equipamento e revisar um cadastro pré-preenchido, com indicação clara do que veio de HTML, imagem/documento, IA, cálculo ou edição humana.

Na Bancada, o usuário deve conseguir ler a curva sem ambiguidades de unidade e abrir a origem de cada valor ou cálculo derivado.

Na recomendação de propulsão, o usuário deve ver não apenas uma posição no ranking, mas massa final, ponto de hover, consumo, TWR/reserva, autonomia/eficiência disponíveis, compatibilidade, cobertura/confiança e uma explicação objetiva dos principais motivos.

No alcance FPV, o usuário informa a distância desejada e vê claramente potência recebida, sensibilidade, margem disponível, margem requerida, potência mínima teórica de VTX e as limitações do modelo de espaço livre.

Princípio de UX: **resumo simples na superfície; engenharia detalhada sob demanda**.

## 13. Critérios de sucesso do MVP

O MVP ampliado está pronto quando um usuário consegue:

1. criar um projeto Mini Long Range ou outro perfil;
2. selecionar/cadastrar frame, eletrônica, payload e demais peças e ver a massa atualizar automaticamente;
3. iniciar o cadastro de um componente por URL e revisar dados extraídos, inclusive de imagem quando houver provider visual configurado;
4. criar/importar e revisar uma curva de bancada motor+hélíce na seção Bancada;
5. visualizar corrente, potência, empuxo, RPM e `gf/W` com unidades e proveniência;
6. solicitar candidatos de propulsão e ter a massa recalculada para cada conjunto;
7. carregar/usar uma curva de bancada compatível sem extrapolação silenciosa;
8. obter empuxo/peso e autonomia estimada quando os dados permitirem;
9. receber ranking orientado ao perfil que não confunda maior empuxo com maior eficiência;
10. abrir a explicação do ranking e distinguir score de cobertura/confiança;
11. informar uma distância FPV desejada e obter link budget, margem e potência teórica mínima necessária;
12. selecionar VRX/óculos/antenas do catálogo ou preencher seus dados manualmente;
13. ver aviso explícito de que o alcance RF é teórico em espaço livre e de que regulamentação não foi verificada;
14. receber alertas elétricos explicáveis;
15. duplicar a configuração e comparar bateria, motor, hélice, VTX ou antena diferente;
16. fechar e reabrir a aplicação sem perder o projeto/dados persistidos.

## 14. Riscos de produto

- dados de fabricantes podem ser incompletos ou inconsistentes;
- IA pode interpretar incorretamente texto, imagem, gráfico ou unidade;
- páginas externas podem conter conteúdo malicioso/prompt injection ou tentar explorar o fetcher;
- C-rating pode superestimar desempenho real da bateria;
- uma combinação motor/hélice não pode ser inferida com boa precisão apenas por KV e tensão;
- curvas de bancada podem conter unidades ambíguas, tensão ausente ou valores derivados incorretamente rotulados;
- pitch speed pode ser confundida com velocidade real do drone;
- autonomia depende de aerodinâmica, temperatura, vento, eficiência de ESC/motor e perfil de pilotagem;
- ranking pode parecer preciso mesmo quando poucos candidatos possuem dados completos;
- otimizar apenas `gf/W`, corrente ou empuxo máximo pode produzir recomendação inadequada ao uso;
- sensibilidade de VRX pode variar por modo/condição e não estar disponível;
- ganho de antena pode ser `directivity`, `gain` ou `realized gain`, levando a dupla contagem de perdas se mal modelado;
- FSPL ignora obstáculos, multipath, orientação e interferência;
- usuário pode interpretar potência teórica necessária como autorização legal de transmissão;
- usuários podem interpretar uma estimativa/recomendação como garantia de segurança.

Mitigação: proveniência, evidência por campo, staging/revisão, validação de schema, controles anti-SSRF/prompt injection, confiança, cobertura de dados, mensagens de limitação, preferência por dados medidos, unidades explícitas, hard constraints antes do score, recálculo de massa por candidato, semântica explícita de ganho RF e validação de entrada.

## 15. Futuro do produto

Após o MVP, o DroneCalc poderá evoluir para:

- otimizador global de configurações e busca em grande escala;
- Pareto front e ranking multiobjetivo avançado;
- dimensionamento iterativo de bateria por meta de autonomia;
- zona de Fresnel e radio horizon;
- terreno/LOS com mapas/DEM;
- noise floor/interferência medidos;
- link RC/ExpressLRS/Crossfire separado;
- compliance regulatório por região;
- centro de gravidade 2D/3D;
- catálogo comunitário de testes;
- comparação de custo/disponibilidade;
- relatórios exportáveis;
- aplicativo desktop com Tauri;
- sincronização opcional;
- API do motor de cálculo;
- suporte a multirrotores além de quadricópteros;
- digitalização assistida de curvas/gráficos com validação humana;
- busca semântica/RAG técnico sobre evidências aprovadas.
