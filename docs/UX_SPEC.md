# UX Spec — DroneCalc

**Versão:** 0.1

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
- Configurações.

No contexto de um projeto, Builder e Análise são as áreas centrais.

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
- tamanho máximo/preferido;
- sub-250 g;
- bateria preferida/opcional;
- nível de experiência.

### Passo 3 — Metas derivadas

Mostrar o que o DroneCalc entendeu, sem fingir que já escolheu peças:

```text
Mini Long Range
Prioridade: autonomia
Sub-250 g: ativo
Payload: 25 g

Critérios de projeto
• eficiência: prioridade muito alta
• massa: prioridade muito alta
• estabilidade: alta
• reserva de potência: média
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
│ Bateria      │ Bateria ...                  │                  │
└──────────────┴──────────────────────────────┴──────────────────┘
```

O resumo técnico é sticky quando houver espaço.

## 7. Seleção de componente

O picker deve permitir:

- busca;
- filtro por categoria;
- filtros técnicos relevantes;
- visualizar dados críticos antes de selecionar;
- criar componente customizado;
- indicar componentes sem dados suficientes.

Não mostrar “compatível” apenas por não existir regra para detectar incompatibilidade.

## 8. Edição de componente no projeto

Permitir:

- quantidade;
- override de massa;
- posição para CG futuro;
- notas;
- substituição da peça.

Overrides devem ter indicação visual e opção “restaurar valor do catálogo”.

## 9. Resumo técnico

Métricas principais, quando disponíveis:

- massa de decolagem;
- TWR;
- autonomia;
- energia;
- corrente máxima;
- perfil/score;
- maior severidade de alerta.

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
7. Compatibilidade;
8. Perfil de voo;
9. Dados e confiança.

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

Exemplo:

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

## 13. Comparador

Usuário seleciona dois ou mais projetos/variantes.

Cabeçalho fixa nome e perfil. Linhas agrupadas:

- massa;
- bateria;
- propulsão;
- autonomia;
- eficiência;
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

## 15. Entrada numérica

Regras:

- aceitar vírgula e ponto na UI pt-BR conforme estratégia de parsing;
- normalizar para número internamente;
- unidade sempre visível;
- não limpar valor válido ao perder foco;
- não converter silenciosamente entre unidade escolhida e interna sem exibir resultado correto;
- números inválidos recebem mensagem específica;
- zero só é aceito quando fisicamente permitido.

## 16. Estados de carregamento/persistência

Como MVP é local, operações devem ser rápidas, mas ainda prever:

- salvando;
- salvo;
- falha ao salvar;
- importando;
- importação inválida.

Não bloquear toda a aplicação por autosave.

## 17. Empty states

Exemplo em Propulsão:

```text
Ainda não é possível calcular empuxo.
Adicione motor, hélice e uma curva de bancada compatível.
[Selecionar motor] [Adicionar teste]
```

Empty state deve orientar o próximo passo.

## 18. Educação contextual

Termos como KV, C-rating, TWR e g/W podem ter tooltips/links “Entenda”.

Explicação curta primeiro; conteúdo aprofundado pode ser expandido.

Não misturar tutorial com alerta de segurança.

## 19. Configurações

Inicialmente:

- tema: Sistema / Claro / Escuro;
- unidades preferidas;
- idioma futuro;
- defaults de reserva/capacidade utilizável, se permitidos, sempre visíveis nas análises;
- opções de exportação.

## 20. Acessibilidade

- fluxo completo por teclado;
- ordem de foco previsível;
- skip links quando aplicável;
- modal prende foco e devolve foco ao fechar;
- elementos clicáveis usam semântica correta;
- status não depende apenas de cor;
- gráficos têm resumo textual/tabela acessível.

## 21. Responsividade

Desktop: layout completo com painéis paralelos.  
Tablet: sidebar recolhível e resumo técnico abaixo/overlay controlado.  
Mobile: foco em consulta/edição simples; comparador e tabelas podem usar scroll horizontal. Paridade mobile total não é requisito do MVP.

## 22. Microcopy

Preferir linguagem técnica clara:

- “Dados insuficientes” em vez de “Erro” quando falta informação;
- “Estimado” sempre que houver modelo;
- “Máximo declarado pelo fabricante” em vez de “Máximo seguro”;
- “Verificação não disponível” em vez de “Compatível” quando campos faltarem.

## 23. Critérios de aceite UX

- usuário consegue criar um projeto sem conhecer todos os termos;
- usuário avançado encontra entradas e fórmulas;
- nenhuma unidade crítica fica implícita;
- alertas explicam causa;
- dados ausentes são distinguíveis de zero;
- tema claro/escuro preserva legibilidade;
- ações destrutivas são confirmadas;
- qualquer score possui explicação acessível.
