# Critérios de Release — DroneCalc

**Versão:** 0.1

## 1. Objetivo

Definir quando uma etapa, MVP ou release pode ser considerada pronta sem depender apenas de “funciona na máquina do desenvolvedor”.

## 2. Gate geral

Nenhuma release deve ser considerada estável se existir:

- teste crítico falhando;
- erro de typecheck;
- build quebrado;
- regressão conhecida de cálculo sem documentação;
- mudança de schema sem migração;
- warning crítico que não explica causa;
- resultado estimado exibido como medição;
- incompatibilidade elétrica conhecida não detectada em fixture de referência;
- perda silenciosa de dados em import/persistência.

## 3. Gate técnico por PR

Quando os scripts existirem:

```text
typecheck: PASS
lint: PASS
tests: PASS
build: PASS
```

Mudanças de UI críticas também devem ter inspeção acessível nos temas claro e escuro.

## 4. Gate do motor de cálculo

- fórmulas críticas com testes explícitos;
- casos-limite testados;
- unidade de entrada/saída documentada;
- formula/model version definida quando aplicável;
- sem arredondamento prematuro;
- sem extrapolação implícita;
- dados ausentes tratados como estado, não zero.

## 5. Gate de dados

- schema validado;
- fixtures passam validação;
- import inválido não persiste parcial;
- round-trip export/import testado;
- migration existente quando necessária;
- source metadata preservável.

## 6. Gate de UI/UX

- fluxo principal por teclado;
- labels/unidades visíveis;
- empty states orientam próximo passo;
- `danger`, `warning`, `info` e `success` possuem texto/ícone além de cor;
- “Como foi calculado?” disponível para métricas críticas;
- NEXO tokens usados nas features;
- Claro/Escuro/Sistema funcionam.

## 7. Critérios do primeiro MVP

O MVP está apto a receber tag/release quando, no mínimo:

1. usuário cria projeto manual ou por estilo;
2. Mini Long Range está disponível como perfil próprio;
3. sub-250 g é constraint independente;
4. projetos são persistidos localmente;
5. componentes customizados podem ser usados;
6. massa total é calculada com dados ausentes tratados corretamente;
7. bateria fornece tensão nominal, cheia e Wh;
8. regras elétricas mínimas detectam limites conhecidos;
9. curva motor+hélíce pode ser carregada/importada;
10. hover/TWR são calculáveis a partir de dados compatíveis;
11. autonomia de hover é estimada com hipóteses visíveis;
12. análise mostra proveniência/confiança;
13. comparação de duas variantes funciona;
14. testes de referência estão verdes;
15. documentação corresponde ao comportamento executável.

## 8. Critérios para versão 1.0 futura

A definição exata deve ser revisada posteriormente, mas deverá exigir:

- estabilidade de schemas;
- política clara de versão do calculation engine;
- catálogo mínimo validado;
- UX consolidada;
- suíte E2E dos fluxos principais;
- estratégia de backup/export madura;
- changelog consistente;
- documentação de instalação/uso atualizada;
- ausência de issues críticas conhecidas.

## 9. Mudança de cálculo entre releases

Se uma fórmula mudar materialmente:

- incrementar modelVersion;
- registrar no changelog;
- atualizar fixture esperada após revisão humana/técnica;
- informar impacto esperado;
- não esconder diferença como simples refactor.

## 10. Release checklist

```text
[ ] typecheck
[ ] lint
[ ] unit tests
[ ] integration tests
[ ] build
[ ] fixtures numéricas revisadas
[ ] schemas/migrations revisados
[ ] light/dark verificados
[ ] acessibilidade básica verificada
[ ] docs atualizadas
[ ] changelog atualizado
[ ] limitações conhecidas registradas
```
