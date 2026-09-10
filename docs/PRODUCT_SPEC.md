# Product Spec — DroneCalc

**Versão:** 0.1  
**Status:** pré-MVP

Este documento descreve o comportamento funcional esperado. O PRD define o porquê; este documento define **o que a aplicação deve fazer**.

## 1. Estrutura principal da aplicação

A aplicação terá cinco áreas principais:

1. **Projetos** — lista, criação, duplicação e gerenciamento.
2. **Builder** — montagem do drone por componentes.
3. **Análise** — resultados técnicos e alertas.
4. **Comparador** — comparação de variantes.
5. **Catálogo** — componentes, dados de fabricante e testes de bancada.

A navegação deverá manter o projeto ativo como contexto persistente.

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

## 4. Análise progressiva

A análise não deve exigir projeto completo.

Exemplos:

- se apenas massa estiver disponível, calcular peso;
- se bateria estiver disponível, calcular energia;
- se motor/ESC estiverem disponíveis, validar corrente/tensão conforme dados existentes;
- se curva motor+hélíce estiver disponível, liberar empuxo e autonomia mais confiável;
- se não houver curva, permitir estimativa apenas quando existir um modelo explicitamente suportado, marcando baixa/média confiança.

Nunca preencher dados técnicos ausentes silenciosamente.

## 5. Painel de resultados

Resultados mínimos:

### Massa
- massa seca;
- massa da bateria;
- payload;
- massa total de decolagem;
- distribuição percentual por categoria.

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
- throttle estimado de hover;
- eficiência g/W quando dados permitirem.

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

## 6. Proveniência e confiança

Todo `CalculationResult` deve possuir:

- valor;
- unidade;
- origem;
- confiança;
- versão do modelo;
- entradas relevantes ou referência para elas;
- observações opcionais.

Origens aceitas:

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

## 8. Comparador

Comparar no mínimo duas configurações. Mostrar apenas métricas comparáveis e destacar diferenças relevantes.

Métricas iniciais:

- massa total;
- energia Wh;
- empuxo máximo;
- TWR;
- corrente máxima;
- potência;
- hover estimado;
- autonomia;
- número e severidade dos alertas;
- score do perfil de voo, quando disponível.

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

O score de Mini Long Range não pode favorecer empuxo máximo de forma desproporcional. Eficiência e energia útil devem pesar mais que potência bruta.

## 11. Catálogo

Um componente pode ser:

- oficial/importado;
- comunitário futuro;
- customizado pelo usuário.

Campos devem distinguir:

- valor nominal;
- valor máximo;
- valor medido;
- fonte;
- data/revisão opcional;
- notas.

## 12. Testes de bancada

Um conjunto de teste motor/hélice deve incluir no mínimo:

- motor;
- hélice;
- tensão/células ou tensão medida;
- condições/observações quando conhecidas;
- amostras ordenadas.

Amostras típicas:

- throttle;
- thrust;
- current;
- voltage;
- power;
- RPM opcional;
- efficiency opcional ou derivável.

Interpolação só pode ocorrer dentro do intervalo coberto pelos dados, salvo modelo de extrapolação explicitamente documentado. O padrão do MVP é **não extrapolar**.

## 13. Unidades

Entradas e apresentação podem aceitar:

- g, kg, oz, lb;
- mm, cm, m, in;
- mAh, Ah;
- V;
- A;
- W, kW;
- Wh;
- g, N para empuxo conforme contexto de UI, com representação interna inequívoca.

O motor utiliza unidades canônicas definidas no domínio.

## 14. Estados vazios e incompletos

A UI deve diferenciar:

- `não calculado porque faltam dados`;
- `não aplicável`;
- `estimativa indisponível`;
- `erro de entrada`;
- valor zero real.

Nunca representar todos esses estados como `0`.

## 15. Persistência

MVP: armazenamento local versionado.

Requisitos:

- autosave com debounce;
- migrações entre versões de schema;
- exportação JSON;
- importação validada antes de substituir/salvar;
- recuperação segura após erro de parsing.

## 16. Acessibilidade

- foco visível;
- controles rotulados;
- estados não comunicados apenas por cor;
- tabelas com cabeçalhos semânticos;
- mensagens de validação associadas aos inputs;
- uso por teclado para fluxos principais.

## 17. Responsividade

Desktop é prioridade do MVP, pois comparação e builder exigem densidade de informação. Tablet deve permanecer utilizável. Mobile pode apresentar layout simplificado, mas não é requisito de paridade completa no primeiro ciclo.

## 18. Critérios de aceite gerais

Uma funcionalidade só está concluída quando:

- comportamento está de acordo com esta spec;
- tipos estão validados;
- testes apropriados existem;
- unidade e proveniência aparecem nos resultados;
- estados de erro/incompleto foram considerados;
- identidade NEXO foi respeitada;
- documentação foi atualizada se houve mudança de contrato.
