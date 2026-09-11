# User Stories — DroneCalc

**Versão:** 0.3

## 1. Objetivo

Traduzir o PRD em histórias testáveis, úteis para issues, sprints e agentes de implementação.

## US-01 — Criar projeto por estilo de voo

Como usuário, quero escolher um estilo de voo para que o DroneCalc transforme meu objetivo em critérios técnicos.

**Aceite:**

- exibe os seis perfis do MVP;
- mostra descrição curta de cada perfil;
- salva `flightProfileId` no projeto;
- permite seguir sem conhecer motor/hélice;
- perfil não escolhe silenciosamente componentes reais sem explicar critérios.

## US-02 — Criar Mini Long Range sub-250 g

Como usuário de Mini Long Range, quero combinar o perfil com uma restrição sub-250 g.

**Aceite:**

- `mini-long-range` e `sub250g` são campos independentes;
- sistema alerta quando massa calculada atinge/excede a restrição configurada;
- não afirma conformidade regulatória apenas por massa.

## US-03 — Montagem manual

Como usuário avançado, quero iniciar um projeto vazio e escolher todas as peças.

**Aceite:**

- projeto pode existir incompleto;
- análises disponíveis são calculadas progressivamente;
- análises indisponíveis explicam quais dados faltam.

## US-04 — Cadastrar peça personalizada

Como maker, quero cadastrar componente que não existe no catálogo.

**Aceite:**

- tipo, nome/modelo e campos técnicos aplicáveis;
- unidades validadas;
- origem `user-provided`/custom;
- campos desconhecidos podem ficar vazios;
- peça fica disponível para projetos locais.

## US-05 — Ver massa total

Como usuário, quero saber o impacto de cada peça na massa do drone.

**Aceite:**

- mostra massa por linha e categoria;
- mostra massa seca/bateria/payload/decolagem conforme definição;
- atualiza ao adicionar/remover/substituir peças;
- componente sem massa não vira 0 silenciosamente;
- lista dados ausentes.

## US-06 — Ver energia da bateria

Como usuário, quero entender a energia disponível na bateria.

**Aceite:**

- calcula Ah, tensão nominal, tensão cheia e Wh;
- usa tensão por célula da química/modelo;
- exibe fórmula/origem;
- C-rating é tratado separadamente de energia.

## US-07 — Verificar tensão bateria × ESC

Como usuário, quero saber se a tensão máxima da bateria excede o ESC.

**Aceite:**

- comparação usa tensão cheia quando limite é em volts;
- incompatibilidade conhecida gera `danger`;
- mensagem mostra valor e limite;
- falta de dados gera “verificação não disponível”, não `success`.

## US-08 — Verificar corrente motor × ESC

Como usuário, quero avaliar a margem de corrente do ESC.

**Aceite:**

- usa corrente relevante da combinação quando disponível;
- contínuo e burst não são confundidos;
- margem é calculada e exibida;
- threshold de warning é documentado/configurável.

## US-09 — Importar teste de bancada

Como usuário, quero importar CSV de teste para usar dados reais.

**Aceite:**

- permite mapear colunas;
- permite indicar unidades;
- preview antes de salvar;
- valida valores;
- arquivo inválido não corrompe catálogo;
- fonte do teste é preservada.

## US-10 — Calcular hover por curva

Como usuário, quero saber aproximadamente em qual ponto o drone sustentará voo estacionário.

**Aceite:**

- calcula empuxo requerido por motor;
- interpola dentro da curva;
- não extrapola silenciosamente;
- mostra throttle/corrente/potência quando disponíveis;
- resultado indica origem `interpolated`.

## US-11 — Calcular TWR

Como usuário, quero comparar reserva de empuxo com o peso do drone.

**Aceite:**

- usa grandezas coerentes;
- mostra razão adimensional;
- perfil interpreta adequação;
- não usa tabela universal rígida como verdade.

## US-12 — Estimar autonomia

Como usuário, quero estimar tempo de voo.

**Aceite:**

- usa capacidade utilizável explícita;
- usa corrente média defensável;
- inclui cargas auxiliares quando conhecidas;
- mostra hipóteses;
- marca resultado como estimado;
- não inventa cruzeiro se modelo não existir.

## US-13 — Entender um cálculo

Como usuário, quero abrir “Como foi calculado?” para auditar um resultado.

**Aceite:**

- mostra formulaId/modelVersion;
- entradas e unidades;
- origem/confiança;
- hipóteses;
- observações/limitações relevantes.

## US-14 — Comparar variantes

Como usuário, quero duplicar um projeto e trocar uma peça para comparar impacto.

**Aceite:**

- duplicação não altera original;
- comparador mostra delta de métricas;
- warnings aparecem por variante;
- “melhor” só é destacado com critério explicável.

## US-15 — Score Mini Long Range

Como usuário, quero avaliar adequação da configuração ao Mini Long Range.

**Aceite:**

- eficiência/autonomia/baixo peso têm pesos elevados;
- potência máxima não domina ranking;
- score mostra componentes por dimensão;
- `danger` não é mascarado;
- falta de dados reduz cobertura/confiança.

## US-16 — Alterar unidades

Como usuário, quero visualizar unidades familiares sem alterar o resultado físico.

**Aceite:**

- conversões centralizadas;
- g/kg e mm/cm/in entre unidades suportadas;
- valores persistidos permanecem coerentes;
- troca de unidade não perde precisão indevidamente.

## US-17 — Salvar e reabrir projeto

Como usuário, quero continuar meu projeto depois.

**Aceite:**

- autosave ou save confiável;
- reabertura restaura componentes/constraints;
- schema versionado;
- falha de persistência é informada.

## US-18 — Exportar projeto

Como usuário, quero criar um arquivo portátil do projeto.

**Aceite:**

- JSON versionado;
- inclui referências/dados necessários para reabertura;
- inclui versões relevantes;
- import round-trip preserva projeto.

## US-19 — Tema NEXO

Como usuário, quero utilizar claro, escuro ou sistema.

**Aceite:**

- tokens NEXO;
- preferência persistida;
- nenhum status depende apenas de cor;
- contraste e foco continuam funcionais.

## US-20 — Centro de gravidade futuro

Como usuário, quero posicionar componentes e estimar CG.

**Aceite:**

- convenção de eixos visível;
- massa/posição obrigatórias para resultado completo;
- CG parcial é identificado;
- fixture simétrico retorna centro esperado.

## US-21 — Recomendar conjunto de propulsão

Como usuário, quero que o DroneCalc considere as peças que já escolhi e o uso pretendido para recomendar conjuntos de propulsão adequados e eficientes.

**Aceite:**

- candidato é motor + hélice + bateria/tensão, incluindo ESC quando necessário à análise;
- cada candidato recebe uma variante virtual própria;
- massa de decolagem é recalculada incluindo a massa dos próprios componentes candidatos;
- empuxo requerido/hover/TWR são recalculados após a massa;
- compatibilidade elétrica/mecânica é avaliada antes do score;
- dados de bancada aprovados têm prioridade quando aplicáveis;
- conjunto de maior empuxo máximo não vence automaticamente;
- perfil de voo/uso influencia o ranking;
- candidato com dados insuficientes não recebe valores físicos inventados;
- aplicar a recomendação ao projeto exige ação explícita.

## US-22 — Entender o ranking de propulsão

Como usuário, quero entender por que um conjunto foi recomendado acima de outro.

**Aceite:**

- mostra massa final do candidato;
- mostra consumo/eficiência/hover/TWR/autonomia disponíveis;
- mostra warnings e hard constraints;
- mostra `profileScore`, cobertura e confiança separadamente;
- apresenta principais vantagens/desvantagens;
- informa curva/modelo/perfil e versões relevantes;
- `danger` nunca é escondido por score.

## US-23 — Perfil operacional incompleto

Como usuário, quero que o sistema seja transparente quando não possui dados para todos os regimes do meu uso.

**Aceite:**

- fase sem dado/modelo não é tratada como corrente zero;
- cobertura incompleta é mostrada;
- cruzeiro não é inventado como porcentagem fixa de hover;
- ranking/resultados são limitados conforme a falta de dados.

## US-24 — Definir alcance FPV desejado

Como operador, quero informar quantos quilômetros pretendo afastar o drone para avaliar se meu sistema de vídeo possui margem teórica suficiente.

**Aceite:**

- aceita `targetDistanceKm > 0`;
- aceita frequência/canal real quando conhecido;
- permite selecionar VTX, VRX/óculos e antenas do catálogo ou preencher manualmente;
- exige sensibilidade para concluir fechamento do link;
- margem desejada é configurável e não tratada como constante física universal;
- resultado informa que o modelo básico é de espaço livre.

## US-25 — Calcular link budget FPV

Como operador, quero saber a potência recebida e a margem do link na distância desejada.

**Aceite:**

- calcula FSPL;
- calcula potência recebida em dBm;
- mostra sensibilidade e condição usadas;
- mostra margem disponível e headroom após margem desejada;
- exibe `pass/borderline/fail/insufficient-data` conforme regra versionada;
- permite abrir “Como foi calculado?”.

## US-26 — Dimensionar potência mínima de VTX

Como operador, quero saber qual potência teórica de VTX seria necessária para a distância-alvo.

**Aceite:**

- resolve potência mínima em dBm;
- converte para mW;
- calcula EIRP quando dados permitirem;
- não sugere que a potência calculada é automaticamente legal/autorizada;
- não apresenta o valor como garantia de alcance real.

## US-27 — Usar dados corretos de antena

Como usuário avançado, quero que o DroneCalc não conte perdas de antena duas vezes.

**Aceite:**

- distingue `directivity`, `gain`, `realized-gain` e `unknown`;
- realized gain não recebe novamente mismatch/eficiência já incluídos;
- SWR inválido é rejeitado;
- gain kind desconhecido com perdas separadas gera warning;
- polarização é preservada e não recebe perda inventada sem modelo/dado.

## US-28 — Avaliar diversity do VRX

Como usuário com óculos diversity, quero analisar minhas antenas sem somar ganhos de forma incorreta.

**Aceite:**

- cada branch é calculado separadamente;
- diversity de seleção pode usar o melhor branch sob as hipóteses existentes;
- ganhos de duas antenas não são somados como array coerente;
- limitação do modelo simplificado é exibida.

## US-29 — Separar vídeo FPV de rádio-controle

Como usuário Long Range, quero entender que alcance de vídeo e alcance do controle são links diferentes.

**Aceite:**

- análise FPV não declara alcance do rádio-controle;
- warnings e resultados identificam explicitamente o link analisado;
- futuro módulo RC pode reutilizar núcleo RF sem misturar sensibilidade/protocolo automaticamente.

## US-30 — Otimização global futura

Como usuário, quero informar objetivos e receber configurações candidatas explorando um espaço muito maior de combinações.

**Aceite futuro:**

- parte do recomendador básico já validado;
- candidatos incompatíveis conhecidos são removidos;
- ranking é multiobjetivo;
- trade-offs são explicados;
- fontes e confiança acompanham resultados;
- pode dimensionar bateria/constraints iterativamente com critério de convergência;
- pode incluir constraint de alcance FPV mantendo física RF separada da propulsão;
- usuário pode transformar candidato em projeto editável.

## Uso em issues

Ao criar issue de implementação, referenciar `US-XX`, requisitos `RF-XX` relacionados e etapa do roadmap. Isso mantém produto, código e testes conectados.
