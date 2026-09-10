# PRD — DroneCalc

**Versão:** 0.1  
**Status:** pré-MVP  
**Produto:** DroneCalc  
**Repositório:** `eduardobelopro-br/Dronecalc`

## 1. Visão

DroneCalc é uma aplicação para apoiar o projeto de drones DIY por meio de cálculo técnico, comparação de configurações e validação preliminar de compatibilidade. O produto deve servir tanto ao usuário que já conhece componentes quanto à pessoa que sabe apenas o tipo de voo que deseja realizar.

A aplicação transforma componentes e objetivos de voo em uma análise explicável de massa, energia, potência, corrente, empuxo, relação empuxo/peso, autonomia, eficiência, centro de gravidade e margens elétricas.

## 2. Problema

A montagem DIY normalmente exige consultar múltiplas tabelas, páginas de fabricantes, testes de bancada e calculadoras isoladas. O usuário precisa combinar manualmente informações de frame, motor, hélice, ESC, bateria, eletrônica e payload, além de interpretar se a combinação faz sentido para o estilo de voo desejado.

Os principais problemas são:

- dados distribuídos e em unidades diferentes;
- dificuldade de comparar variantes de projeto;
- estimativas de autonomia frequentemente pouco transparentes;
- uso incorreto de valores de KV, corrente, C-rating e tensão;
- falta de rastreabilidade da origem de um resultado;
- recomendações genéricas que ignoram o objetivo de voo;
- dificuldade para iniciantes entenderem por que uma configuração é ou não adequada.

## 3. Proposta de valor

DroneCalc deve funcionar como uma **bancada virtual de dimensionamento**, permitindo experimentar antes de comprar ou montar.

O produto se diferencia por:

- combinar cálculo e catálogo de componentes;
- separar dado medido, dado de fabricante, cálculo, interpolação e estimativa;
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
- importar dados de bancada;
- cadastrar componentes não existentes no catálogo;
- visualizar por que o sistema emitiu um alerta;
- saber o nível de confiança de um resultado;
- exportar ou compartilhar a configuração no futuro.

## 6. Objetivos do MVP

O MVP deve responder de forma confiável e explicável:

1. **Quanto pesa?**
2. **A alimentação elétrica é compatível?**
3. **A propulsão fornece empuxo adequado?**
4. **Qual autonomia pode ser esperada nas condições informadas?**

Além disso, o MVP deve oferecer:

- criação e edição de projetos;
- perfis de voo;
- componentes customizados;
- catálogo local inicial;
- análise de compatibilidade;
- comparação de variantes;
- persistência local;
- tema claro/escuro seguindo NEXO;
- testes automatizados do motor matemático.

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
- sincronização multiusuário.

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

## 9. Requisitos funcionais de alto nível

### RF-01 — Projetos
Criar, renomear, duplicar, editar e excluir projetos locais.

### RF-02 — Componentes
Selecionar componentes do catálogo e cadastrar componentes personalizados.

### RF-03 — Massa
Calcular massa por componente, categoria, massa seca, operacional e total de decolagem.

### RF-04 — Energia
Calcular tensão nominal/cheia, energia em Wh e limites declarados da bateria.

### RF-05 — Sistema elétrico
Comparar tensões suportadas, corrente de motor, ESC, bateria e alimentação de eletrônica.

### RF-06 — Propulsão
Calcular empuxo total e relação empuxo/peso usando dados de bancada quando disponíveis.

### RF-07 — Autonomia
Estimar autonomia por modelo explícito e informar as hipóteses usadas.

### RF-08 — Perfis de voo
Aplicar critérios de avaliação diferentes conforme o objetivo do usuário.

### RF-09 — Compatibilidade
Gerar mensagens `success`, `info`, `warning` e `danger`, sempre com justificativa.

### RF-10 — Proveniência
Todo resultado deve indicar origem e confiança.

### RF-11 — Comparação
Comparar ao menos duas variantes lado a lado.

### RF-12 — Unidades
Aceitar e exibir unidades comuns sem misturar unidades internamente.

### RF-13 — Persistência
Salvar projetos e componentes personalizados localmente.

### RF-14 — Importação/exportação
Planejar formato JSON versionado; CSV será usado principalmente para tabelas de bancada.

## 10. Requisitos não funcionais

### RNF-01 — Determinismo
Mesmas entradas e mesma versão do motor devem produzir os mesmos resultados.

### RNF-02 — Testabilidade
O motor matemático não pode depender de React ou DOM.

### RNF-03 — Rastreabilidade
Cálculos críticos devem registrar fórmula/modelo, entradas e versão do algoritmo.

### RNF-04 — Performance
Alterações simples do projeto devem atualizar a análise sem atraso perceptível em hardware desktop comum.

### RNF-05 — Acessibilidade
Navegação por teclado, foco visível, contraste adequado e semântica ARIA nos controles relevantes.

### RNF-06 — Internacionalização
Código não deve depender de texto em português para regras de negócio. UI inicial pode ser pt-BR.

### RNF-07 — Evolução
Novos tipos de componente e perfis não devem exigir reescrever o motor inteiro.

### RNF-08 — Segurança de dados
Nenhum segredo deve ser armazenado em código ou exportações. Persistência local é padrão no MVP.

## 11. Métricas de qualidade do produto

Antes de chamar o MVP de estável:

- 100% das fórmulas críticas cobertas por testes unitários;
- casos de referência validados manualmente;
- nenhum alerta crítico sem mensagem explicativa;
- nenhum cálculo apresentado sem unidade;
- nenhum valor estimado apresentado como medição;
- importação inválida não pode corromper projetos existentes;
- perfis de voo devem ser configuráveis por dados, não por condicionais espalhadas na UI.

## 12. Experiência desejada

O usuário deve conseguir começar pelo **objetivo**, não pela terminologia técnica. Ao mesmo tempo, um usuário avançado deve conseguir acessar todos os parâmetros, substituir valores e visualizar a origem do cálculo.

Princípio de UX: **resumo simples na superfície; engenharia detalhada sob demanda**.

## 13. Critérios de sucesso do MVP

O MVP está pronto quando um usuário consegue:

1. criar um projeto Mini Long Range ou outro perfil;
2. selecionar/cadastrar frame, motores, hélices, ESC e bateria;
3. ver peso e energia atualizados automaticamente;
4. carregar uma curva de bancada compatível;
5. obter empuxo/peso e autonomia estimada;
6. receber alertas elétricos explicáveis;
7. duplicar a configuração e comparar uma bateria ou motor diferente;
8. fechar e reabrir a aplicação sem perder o projeto.

## 14. Riscos de produto

- dados de fabricantes podem ser incompletos ou inconsistentes;
- C-rating pode superestimar desempenho real da bateria;
- uma combinação motor/hélice não pode ser inferida com boa precisão apenas por KV e tensão;
- autonomia depende de aerodinâmica, temperatura, vento, eficiência de ESC/motor e perfil de pilotagem;
- usuários podem interpretar uma estimativa como garantia de segurança.

Mitigação: proveniência, confiança, mensagens de limitação, preferência por dados medidos e validação de entrada.

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
- suporte a multirrotores além de quadricópteros.
