# UX Spec — DroneCalc

**Versão:** 0.3

## 1. Princípio de experiência

**Resumo simples na superfície; engenharia detalhada sob demanda.**

O DroneCalc deve ser utilizável por quem conhece apenas o objetivo do drone e, ao mesmo tempo, não esconder parâmetros de usuários avançados.

## 2. Arquitetura de informação

Navegação principal:

- Projetos;
- Builder;
- Análise;
- Comparador;
- Catálogo;
- Bancada;
- Configurações.

No contexto de um projeto, Builder e Análise são as áreas centrais. Bancada é um workspace técnico para curvas motor+hélíce e deve ser acessível também a partir de componentes e da análise de propulsão.

O cálculo de **Alcance FPV** fica dentro do contexto de Análise/Projeto, não como produto isolado. Deve reutilizar VTX/antenas selecionados no Builder quando existirem.

## 3. Dashboard de projetos

Cada projeto exibe:

- nome;
- perfil de voo;
- status de completude;
- massa total quando disponível;
- autonomia quando disponível;
- maior severidade de alerta;
- última alteração.

Ações rápidas:

- abrir;
- duplicar;
- renomear;
- exportar;
- excluir.

Exclusão exige confirmação e deve indicar claramente qual projeto será removido.

## 4. Novo projeto

Tela inicial:

```text
Como deseja começar?

[ Por estilo de voo ]
O DroneCalc ajuda a transformar o objetivo em requisitos.

[ Montagem manual ]
Escolha cada peça e acompanhe os cálculos.

[ Duplicar projeto ]
Crie uma variante de um projeto existente.
```

## 5. Wizard por estilo de voo

### Passo 1 — Perfil

Cards:

- Freestyle;
- Racing;
- Cinematic;
- Long Range;
- Mini Long Range;
- Cinewhoop.

Cada card informa três prioridades em linguagem simples.

### Passo 2 — Objetivo

Campos variam conforme perfil, mantendo núcleo consistente:

- prioridade: autonomia / equilíbrio / performance;
- payload;
- autonomia desejada;
- distância FPV desejada opcional;
- tamanho máximo/preferido;
- sub-250 g;
- bateria preferida/opcional;
- nível de experiência.

Distância FPV desejada deve ser apresentada como uma intenção separada. Não deve implicar automaticamente alcance do rádio-controle nem conformidade legal de potência RF.

### Passo 3 — Metas derivadas

Mostrar o que o DroneCalc entendeu, sem fingir que já escolheu peças:

```text
Mini Long Range
Prioridade: autonomia
Sub-250 g: ativo
Payload: 25 g
Alcance FPV desejado: 5 km

Critérios de projeto
• eficiência: prioridade muito alta
• massa: prioridade muito alta
• estabilidade: alta
• reserva de potência: média
• link FPV: analisar separadamente quando VTX/VRX estiverem definidos
```

### Passo 4 — Criar projeto

O usuário entra no Builder com o perfil salvo.

## 6. Builder

Layout desktop sugerido:

```text
┌──────────────┬──────────────────────────────┬──────────────────┐
│ Categorias   │ Componentes do projeto      │ Resumo técnico   │
│              │                              │                  │
│ Frame        │ Frame ...                    │ 238 g            │
│ Motores      │ 4 × Motor ...                │ TWR 2,8          │
│ Hélices      │ 4 × Hélice ...               │ 18,4 min         │
│ ESC          │ ESC ...                      │ ⚠ 2 avisos       │
│ Bateria      │ Bateria ...                  │ FPV: analisar    │
│ FPV          │ VTX + antena ...             │                  │
└──────────────┴──────────────────────────────┴──────────────────┘
```

O resumo técnico é sticky quando houver espaço.

Componentes de estação do operador, como óculos/VRX, podem ser associados ao projeto para cálculo RF sem entrar na massa da aeronave.

## 7. Seleção de componente

O picker deve permitir:

- busca;
- filtro por categoria;
- filtros técnicos relevantes;
- visualizar dados críticos antes de selecionar;
- criar componente customizado;
- indicar componentes sem dados suficientes.

Não mostrar “compatível” apenas por não existir regra para detectar incompatibilidade.

Para componentes RF, filtros relevantes incluem banda/frequência, potência, polarização, ganho, conector e sensibilidade quando aplicável.

## 8. Edição de componente no projeto

Permitir:

- quantidade;
- override de massa;
- posição para CG futuro;
- notas;
- substituição da peça.

Overrides devem ter indicação visual e opção “restaurar valor do catálogo”.

Dados RF manuais/overrides devem indicar claramente quando substituem valores de fabricante e preservar unidade/semântica (`dBm`, `dBi`, `dB`, `mW`).

## 9. Resumo técnico

Métricas principais, quando disponíveis:

- massa de decolagem;
- TWR;
- autonomia;
- energia;
- corrente máxima;
- perfil/score;
- maior severidade de alerta;
- status FPV na distância-alvo, quando configurado.

Cada métrica deve aceitar estado:

- disponível;
- faltam dados;
- inválido;
- não aplicável.

## 10. Tela de análise

Seções:

1. Visão geral;
2. Massa;
3. Propulsão;
4. Energia e bateria;
5. Sistema elétrico;
6. Autonomia;
7. **Alcance FPV**;
8. Compatibilidade;
9. Perfil de voo;
10. Dados e confiança.

Na seção Propulsão, quando uma curva de bancada tiver sido utilizada, deve existir ação **Abrir curva usada** que leva à seção Bancada preservando o contexto do ensaio.

Na seção Alcance FPV, deve existir ação **Configurar enlace** quando faltarem dados.

## 11. “Como foi calculado?”

Toda métrica calculada abre detalhe com:

- fórmula/modelo;
- versão;
- valores de entrada;
- unidades;
- origem das entradas;
- confiança;
- hipóteses;
- limitações relevantes.

Exemplo de autonomia:

```text
Autonomia em hover: 18,4 min

Modelo: ENDURANCE_CURRENT_MODEL_V1
Capacidade: 3,0 Ah
Fração utilizável: 0,80
Corrente de hover: 7,82 A
Cargas auxiliares: 0,42 A

Origem: estimada a partir de curva de bancada
Confiança: média
```

Exemplo RF:

```text
Link FPV a 5,0 km: PASS

Modelo: RF_FSPL_V1 + RF_LINK_BUDGET_V1
Frequência: 5800 MHz
VTX: 23,01 dBm
Ganho TX efetivo: ... dBi
Ganho RX efetivo: ... dBi
FSPL: ... dB
Potência recebida: ... dBm
Sensibilidade: ... dBm
Margem disponível: ... dB
Margem desejada: ... dB

Limitação: modelo de espaço livre; não considera obstáculos/interferência.
```

## 12. Alertas

Formato:

```text
⚠ Margem de corrente do ESC baixa
Motor estimado: 31 A
ESC contínuo: 35 A
Margem: 12,9%

Recomendação: revisar a combinação ou usar ESC com maior margem.
```

Alertas devem ter ação contextual quando possível (“Ver ESC”, “Ver bateria”, “Dados ausentes”).

Para RF, exemplos de ação contextual:

- “Informar sensibilidade do VRX”;
- “Revisar tipo de ganho da antena”;
- “Ver antena TX”;
- “Aumentar margem desejada”;
- “Revisar potência do VTX”.

## 13. Comparador

Usuário seleciona dois ou mais projetos/variantes.

Cabeçalho fixa nome e perfil. Linhas agrupadas:

- massa;
- bateria;
- propulsão;
- autonomia;
- eficiência;
- alcance FPV, quando comparável;
- alertas;
- score do perfil.

Diferença numérica deve mostrar unidade e delta. Destaque de “melhor” só quando o critério é inequívoco e contextualizado.

## 14. Catálogo

Lista de componentes com:

- tipo;
- fabricante/modelo;
- massa;
- principais limites;
- completude dos dados;
- número de curvas/testes associados;
- origem.

Componentes customizados devem ser distinguíveis dos dados de fabricante/importados.

No detalhe de motores e hélices, disponibilizar `Ver testes de bancada` quando existirem curvas associadas e `Adicionar teste` quando o usuário tiver permissão para cadastrar dados.

Para VTX/VRX/antenas, exibir atributos RF com unidade e condição. Não reduzir “ganho 2 dBi” a um único número sem expor `gainKind` quando disponível.

## 14.1 Bancada

A seção **Bancada** é normatizada por `BENCH_DATA_WORKSPACE.md`.

Objetivos de UX:

- visualizar claramente a combinação motor + hélice + condições do ensaio;
- cadastrar/editar amostras;
- importar CSV/tabela com preview;
- revisar dados vindos de scraping/IA antes de aprovação;
- distinguir valor medido/declarado de valor calculado/interpolado;
- permitir auditoria da fonte;
- mostrar warnings de unidade/consistência sem modificar silenciosamente a fonte.

Layout desktop sugerido:

```text
┌──────────────────────────────────────────────────────────────────┐
│ Bancada — Motor X + Hélice Y                                    │
│ Condições / fonte / status                         [Importar CSV] │
├────────────────────────────────┬─────────────────────────────────┤
│ Tabela de amostras             │ Resumo do ponto selecionado     │
│ Throttle | V | A | W | gf ...  │ Empuxo / Corrente / RPM / gf/W │
├────────────────────────────────┴─────────────────────────────────┤
│ Gráficos técnicos separados por grandeza/unidade                │
└──────────────────────────────────────────────────────────────────┘
```

Regras visuais obrigatórias:

- não chamar `throttle %` de aceleração física;
- não chamar `gf/W` apenas de “Eficiência” sem unidade;
- não mostrar pitch speed como velocidade real ou máxima do drone;
- não usar por padrão uma única escala Y para A, gf, RPM e gf/W;
- pontos medidos e interpolados devem ser distinguíveis;
- a tabela permanece representação acessível/auditável;
- valor derivado abre `Como foi calculado?`.

Microcopy obrigatória para pitch speed:

```text
Velocidade teórica de passo
Não representa a velocidade real da aeronave; ignora slip, arrasto,
advance ratio e outros efeitos aerodinâmicos.
```

## 14.2 Alcance FPV

A experiência é normatizada por `FPV_LINK_BUDGET.md`.

Layout desktop sugerido:

```text
┌──────────────────────────────────────────────────────────────────┐
│ Alcance FPV                                                     │
│ Distância desejada [ 5,0 ] km       Frequência [ 5800 ] MHz     │
│ Margem desejada   [ ... ] dB                                   │
├───────────────────────────────┬──────────────────────────────────┤
│ Transmissão                   │ Recepção                         │
│ VTX / potência               │ VRX / Óculos                     │
│ Antena TX / ganho / tipo     │ Antena RX / ganho / tipo         │
│ SWR/cabo/perdas opcionais    │ Sensibilidade / diversity        │
├───────────────────────────────┴──────────────────────────────────┤
│ Resultado                                                         │
│ Estado do link | Pr | Sensibilidade | Margem | Headroom           │
│ Potência mínima teórica | EIRP | Distância teórica espaço livre  │
└──────────────────────────────────────────────────────────────────┘
```

### Entrada progressiva

O usuário pode:

- selecionar VTX/VRX/óculos/antenas do catálogo;
- substituir campos manualmente;
- usar apenas antena do óculos + sensibilidade manual quando o VRX não estiver catalogado;
- cadastrar componente ausente pela própria área de Catálogo.

### Resultado

Mostrar prioritariamente:

```text
Distância-alvo: 5,0 km
Status: PASS / BORDERLINE / FAIL / DADOS INSUFICIENTES
Margem disponível: ... dB
Margem desejada: ... dB
Headroom: ... dB
```

Detalhes expandíveis:

- FSPL;
- potência recebida;
- sensibilidade usada e condição;
- potência mínima teórica de VTX em dBm/mW;
- EIRP;
- distância teórica máxima em espaço livre;
- perdas consideradas;
- branch RX selecionado em diversity;
- proveniência/confiança.

### Regras visuais obrigatórias

- `dBm` deve ser apresentado como potência logarítmica, `dB` como razão/perda e `dBi` como ganho de antena;
- não chamar distância FSPL de “alcance real” ou “alcance garantido”;
- não chamar potência mínima calculada de “potência permitida/legal”;
- não somar ganhos de duas antenas diversity;
- se `gainKind` for desconhecido, mostrar warning antes de aplicar perdas potencialmente duplicadas;
- realized gain deve indicar que mismatch/eficiência já podem estar embutidos;
- vídeo FPV e rádio-controle devem ser visualmente separados.

Microcopy obrigatória:

```text
Distância teórica em espaço livre
Estimativa baseada em linha de visada ideal. Obstáculos, orientação das antenas,
multipath, interferência, Fresnel e condições reais podem reduzir o alcance.
```

E:

```text
Status regulatório não verificado
A potência/EIRP calculada é um resultado técnico e não confirma autorização de uso.
```

## 15. Entrada numérica

Regras:

- aceitar vírgula e ponto na UI pt-BR conforme estratégia de parsing;
- normalizar para número internamente;
- unidade sempre visível;
- não limpar valor válido ao perder foco;
- não converter silenciosamente entre unidade escolhida e interna sem exibir resultado correto;
- números inválidos recebem mensagem específica;
- zero só é aceito quando fisicamente permitido;
- para RF, validar explicitamente `km`, `MHz/GHz`, `mW/W`, `dBm`, `dB`, `dBi` e `SWR`;
- impedir que um campo de dBi seja interpretado como dB de perda e vice-versa.

## 16. Estados de carregamento/persistência

Operações devem prever:

- salvando;
- salvo;
- falha ao salvar;
- importando;
- importação inválida;
- aguardando revisão;
- conflito de fonte quando aplicável.

Não bloquear toda a aplicação por autosave. O backend/PostgreSQL é a persistência autoritativa quando online; IndexedDB serve apenas aos casos definidos na arquitetura.

## 17. Empty states

Propulsão:

```text
Ainda não é possível calcular empuxo.
Adicione motor, hélice e uma curva de bancada compatível.
[Selecionar motor] [Adicionar teste]
```

Bancada:

```text
Nenhuma curva de bancada cadastrada para esta combinação.
[Adicionar teste] [Importar CSV]
```

Alcance FPV:

```text
Ainda não é possível calcular o link FPV.
Informe a sensibilidade do VRX/óculos e a frequência do sistema.
[Selecionar VRX] [Preencher manualmente]
```

Empty state deve orientar o próximo passo.

## 18. Educação contextual

Termos como KV, C-rating, TWR, g/W, pitch speed, dBm, dBi, SWR e link margin podem ter tooltips/links “Entenda”.

Explicação curta primeiro; conteúdo aprofundado pode ser expandido.

Não misturar tutorial com alerta de segurança/legalidade.

## 19. Configurações

Inicialmente:

- tema: Sistema / Claro / Escuro;
- unidades preferidas;
- idioma futuro;
- defaults de reserva/capacidade utilizável, se permitidos, sempre visíveis nas análises;
- margem RF default opcional apenas como preferência explícita, nunca constante escondida no engine;
- opções de exportação.

## 20. Acessibilidade

- fluxo completo por teclado;
- ordem de foco previsível;
- skip links quando aplicável;
- modal prende foco e devolve foco ao fechar;
- elementos clicáveis usam semântica correta;
- status não depende apenas de cor;
- gráficos têm resumo textual/tabela acessível;
- tabela de bancada deve expor cabeçalhos/unidades semanticamente;
- estado RF PASS/BORDERLINE/FAIL deve ter texto, números e explicação.

## 21. Responsividade

Desktop: layout completo com painéis paralelos.  
Tablet: sidebar recolhível e resumo técnico abaixo/overlay controlado.  
Mobile: foco em consulta/edição simples; comparador e tabelas podem usar scroll horizontal. Paridade mobile total não é requisito do MVP.

Bancada e Alcance FPV podem priorizar leitura/resumo no mobile e reservar edição de alta densidade para desktop/tablet.

## 22. Microcopy

Preferir linguagem técnica clara:

- “Dados insuficientes” em vez de “Erro” quando falta informação;
- “Estimado” sempre que houver modelo;
- “Máximo declarado pelo fabricante” em vez de “Máximo seguro”;
- “Verificação não disponível” em vez de “Compatível” quando campos faltarem;
- “Eficiência estática (gf/W)” em vez de “Eficiência” para a razão empuxo/potência;
- “Velocidade teórica de passo” em vez de “Velocidade” para `pitch × RPM`;
- “Distância teórica máxima em espaço livre” em vez de “Alcance máximo” para inversão de FSPL;
- “Potência teórica mínima de VTX” em vez de “Potência necessária garantida”;
- “Status regulatório não verificado” quando não houver módulo de compliance.

## 23. Critérios de aceite UX

- usuário consegue criar um projeto sem conhecer todos os termos;
- usuário avançado encontra entradas e fórmulas;
- nenhuma unidade crítica fica implícita;
- alertas explicam causa;
- dados ausentes são distinguíveis de zero;
- tema claro/escuro preserva legibilidade;
- ações destrutivas são confirmadas;
- qualquer score possui explicação acessível;
- Bancada diferencia dados medidos, declarados, calculados e interpolados;
- pitch speed nunca é apresentada como velocidade real da aeronave;
- gráficos de bancada não misturam grandezas incompatíveis numa única escala por padrão;
- Alcance FPV distingue dB/dBi/dBm/mW;
- realized gain não leva a dupla perda silenciosa;
- distância RF calculada é rotulada como teórica em espaço livre;
- link FPV não é apresentado como alcance do rádio-controle;
- potência RF calculada não é apresentada como autorização legal.
