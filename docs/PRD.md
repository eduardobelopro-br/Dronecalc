# PRD — DroneCalc

**Versão:** 0.3  
**Status:** pré-MVP  
**Produto:** DroneCalc  
**Repositório:** `eduardobelopro-br/Dronecalc`

## 1. Visão

DroneCalc é uma aplicação para apoiar o projeto de drones DIY por meio de cálculo técnico, comparação de configurações e validação preliminar de compatibilidade. O produto deve servir tanto ao usuário que já conhece componentes quanto à pessoa que sabe apenas o tipo de voo que deseja realizar.

A aplicação transforma componentes e objetivos de voo em uma análise explicável de massa, energia, potência, corrente, empuxo, relação empuxo/peso, autonomia, eficiência, centro de gravidade e margens elétricas.

## 2. Problema

A montagem DIY normalmente exige consultar múltiplas tabelas, páginas de fabricantes, imagens técnicas, testes de bancada e calculadoras isoladas. O usuário precisa combinar manualmente informações de frame, motor, hélice, ESC, bateria, eletrônica e payload, além de interpretar se a combinação faz sentido para o estilo de voo desejado.

Os principais problemas são:

- dados distribuídos e em unidades diferentes;
- especificações importantes disponíveis apenas em imagens, PDFs ou tabelas;
- dificuldade de comparar variantes de projeto;
- estimativas de autonomia frequentemente pouco transparentes;
- uso incorreto de valores de KV, corrente, C-rating e tensão;
- interpretação incorreta de dados de bancada, como confundir `gf/W` com eficiência percentual ou pitch speed com velocidade real do drone;
- falta de rastreabilidade da origem de um resultado;
- recomendações genéricas que ignoram o objetivo de voo;
- dificuldade para iniciantes entenderem por que uma configuração é ou não adequada.

## 3. Proposta de valor

DroneCalc deve funcionar como uma **bancada virtual de dimensionamento**, permitindo experimentar antes de comprar ou montar.

O produto se diferencia por:

- combinar cálculo e catálogo de componentes;
- permitir cadastro assistido a partir de URL com extração de HTML, imagens e documentos técnicos;
- oferecer uma seção Bancada para cadastrar, revisar e visualizar curvas motor+hélíce;
- separar dado medido, dado de fabricante, dado extraído, cálculo, interpolação e estimativa;
- preservar evidência e revisão para dados importados;
- calcular com unidades canônicas e converter apenas na apresentação;
- utilizar curvas reais de bancada quando disponíveis;
- adaptar critérios ao estilo de voo;
- explicar alertas e margens;
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
- verificar tensão e corrente entre bateria, ESC, motor e eletrônica;
- entender se há empuxo suficiente;
- estimar autonomia em cenários diferentes;
- comparar duas ou mais configurações;
- cadastrar/importar e revisar dados de bancada;
- visualizar curvas de empuxo, corrente, potência, RPM e eficiência estática com unidades explícitas;
- cadastrar componentes não existentes no catálogo;
- informar a página de um equipamento e obter um cadastro técnico pré-preenchido a partir de texto, tabelas, imagens e documentos disponíveis;
- revisar conflitos e evidências antes de publicar dados extraídos no catálogo;
- visualizar por que o sistema emitiu um alerta;
- saber o nível de confiança de um resultado;
- exportar ou compartilhar a configuração no futuro.

## 6. Objetivos do MVP ampliado

O MVP deve responder de forma confiável e explicável:

1. **Quanto pesa?**
2. **A alimentação elétrica é compatível?**
3. **A propulsão fornece empuxo adequado?**
4. **Qual autonomia pode ser esperada nas condições informadas?**

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
- comparação de variantes;
- PostgreSQL como fonte persistente principal;
- cache/drafts locais auxiliares;
- tema claro/escuro seguindo NEXO;
- testes automatizados do motor matemático, da Bancada e do pipeline crítico de ingestão.

## 7. Fora do escopo do MVP

Não fazem parte do primeiro MVP:

- controle de voo em tempo real;
- comunicação com flight controller;
- configuração de Betaflight/ArduPilot/PX4;
- certificação de aeronavegabilidade;
- simulação CFD;
- cálculo estrutural por elementos finitos;
- previsão meteorológica;
- planejamento de missão;
- cálculo regulatório automático por país;
- marketplace;
- otimização global de milhares de combinações em servidor;
- sincronização multiusuário;
- previsão de velocidade real/máxima da aeronave baseada apenas em pitch e RPM.

Esses itens podem ser avaliados após o núcleo matemático estar validado.

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

## 9. Requisitos funcionais de alto nível

### RF-01 — Projetos
Criar, renomear, duplicar, editar e excluir projetos.

### RF-02 — Componentes
Selecionar componentes do catálogo e cadastrar componentes personalizados.

### RF-03 — Massa
Calcular massa por componente, categoria, massa seca, operacional e total de decolagem.

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

## 10. Requisitos não funcionais

### RNF-01 — Determinismo
Mesmas entradas e mesma versão do motor devem produzir os mesmos resultados.

### RNF-02 — Testabilidade
O motor matemático não pode depender de React ou DOM.

### RNF-03 — Rastreabilidade
Cálculos críticos devem registrar fórmula/modelo, entradas e versão do algoritmo. Imports automáticos e dados de bancada devem registrar fonte, método e evidência/proveniência quando aplicável.

### RNF-04 — Performance
Alterações simples do projeto devem atualizar a análise sem atraso perceptível em hardware desktop comum. Processos de ingestão/IA podem ser assíncronos e devem expor estado/progresso adequado.

### RNF-05 — Acessibilidade
Navegação por teclado, foco visível, contraste adequado e semântica ARIA nos controles relevantes. Gráficos de bancada devem possuir tabela/resumo acessível.

### RNF-06 — Internacionalização
Código não deve depender de texto em português para regras de negócio. UI inicial pode ser pt-BR.

### RNF-07 — Evolução
Novos tipos de componente, perfis e providers de IA não devem exigir reescrever o motor inteiro.

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
- dado extraído por IA não pode ser publicado sem o workflow de staging/revisão;
- controles críticos anti-SSRF e validação de schema possuem testes explícitos;
- curvas de bancada não podem extrapolar silenciosamente;
- `gf/W` deve aparecer com semântica/unidade explícita;
- pitch speed teórica não pode ser apresentada como velocidade real/máxima do drone;
- gráficos de bancada não devem usar escala única enganosa para grandezas incompatíveis;
- perfis de voo devem ser configuráveis por dados, não por condicionais espalhadas na UI.

## 12. Experiência desejada

O usuário deve conseguir começar pelo **objetivo**, não pela terminologia técnica. Ao mesmo tempo, um usuário avançado deve conseguir acessar todos os parâmetros, substituir valores e visualizar a origem do cálculo.

No catálogo, o usuário deve poder colar a página do equipamento e revisar um cadastro pré-preenchido, com indicação clara do que veio de HTML, imagem/documento, IA, cálculo ou edição humana.

Na Bancada, o usuário deve conseguir ler a curva sem ambiguidades de unidade e abrir a origem de cada valor ou cálculo derivado.

Princípio de UX: **resumo simples na superfície; engenharia detalhada sob demanda**.

## 13. Critérios de sucesso do MVP

O MVP ampliado está pronto quando um usuário consegue:

1. criar um projeto Mini Long Range ou outro perfil;
2. selecionar/cadastrar frame, motores, hélices, ESC e bateria;
3. iniciar o cadastro de um componente por URL e revisar dados extraídos, inclusive de imagem quando houver provider visual configurado;
4. criar/importar e revisar uma curva de bancada motor+hélíce na seção Bancada;
5. visualizar corrente, potência, empuxo, RPM e `gf/W` com unidades e proveniência;
6. ver peso e energia atualizados automaticamente;
7. carregar/usar uma curva de bancada compatível sem extrapolação silenciosa;
8. obter empuxo/peso e autonomia estimada quando os dados permitirem;
9. receber alertas elétricos explicáveis;
10. duplicar a configuração e comparar uma bateria ou motor diferente;
11. fechar e reabrir a aplicação sem perder o projeto/dados persistidos.

## 14. Riscos de produto

- dados de fabricantes podem ser incompletos ou inconsistentes;
- IA pode interpretar incorretamente texto, imagem, gráfico ou unidade;
- páginas externas podem conter conteúdo malicioso/prompt injection ou tentar explorar o fetcher;
- C-rating pode superestimar desempenho real da bateria;
- uma combinação motor/hélice não pode ser inferida com boa precisão apenas por KV e tensão;
- curvas de bancada podem conter unidades ambíguas, tensão ausente ou valores derivados incorretamente rotulados;
- pitch speed pode ser confundida com velocidade real do drone;
- autonomia depende de aerodinâmica, temperatura, vento, eficiência de ESC/motor e perfil de pilotagem;
- usuários podem interpretar uma estimativa como garantia de segurança.

Mitigação: proveniência, evidência por campo, staging/revisão, validação de schema, controles anti-SSRF/prompt injection, confiança, mensagens de limitação, preferência por dados medidos, unidades explícitas e validação de entrada.

## 15. Futuro do produto

Após o MVP, o DroneCalc poderá evoluir para:

- otimizador de configurações;
- ranking multiobjetivo;
- centro de gravidade 2D/3D;
- catálogo comunitário de testes;
- comparação de custo;
- relatórios exportáveis;
- aplicativo desktop com Tauri;
- sincronização opcional;
- API do motor de cálculo;
- suporte a multirrotores além de quadricópteros;
- digitalização assistida de curvas/gráficos com validação humana;
- busca semântica/RAG técnico sobre evidências aprovadas.