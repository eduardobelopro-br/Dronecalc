# Critérios de Release — DroneCalc

**Versão:** 0.2

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
- perda silenciosa de dados em import/persistência;
- link budget RF apresentando espaço livre como alcance real garantido;
- dupla contagem conhecida de perdas de antena;
- potência/EIRP calculada apresentada como autorização regulatória.

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
- dados ausentes tratados como estado, não zero;
- dBm/dB/dBi não misturados semanticamente;
- FSPL/solver inverso cobertos por fixtures quando o módulo RF estiver habilitado.

## 5. Gate de dados

- schema validado;
- fixtures passam validação;
- import inválido não persiste parcial;
- round-trip export/import testado;
- migration existente quando necessária;
- source metadata preservável;
- staging e catálogo publicado permanecem separados;
- dados RF preservam condição/modo e `gainKind` quando aplicável.

## 6. Gate de UI/UX

- fluxo principal por teclado;
- labels/unidades visíveis;
- empty states orientam próximo passo;
- `danger`, `warning`, `info` e `success` possuem texto/ícone além de cor;
- “Como foi calculado?” disponível para métricas críticas;
- NEXO tokens usados nas features;
- Claro/Escuro/Sistema funcionam;
- resultado RF informa espaço livre/limitações;
- vídeo FPV não é apresentado como alcance do rádio-controle.

## 7. Critérios do primeiro MVP ampliado

O MVP está apto a receber tag/release quando, no mínimo:

1. usuário cria projeto manual ou por estilo;
2. Mini Long Range está disponível como perfil próprio;
3. sub-250 g é constraint independente;
4. projetos são persistidos pela arquitetura autoritativa PostgreSQL/backend, com cache/drafts locais apenas conforme especificado;
5. componentes customizados podem ser usados;
6. massa total é calculada com dados ausentes tratados corretamente;
7. bateria fornece tensão nominal, cheia e Wh;
8. regras elétricas mínimas detectam limites conhecidos;
9. curva motor+hélíce pode ser carregada/importada;
10. hover/TWR são calculáveis a partir de dados compatíveis;
11. autonomia de hover é estimada com hipóteses visíveis;
12. análise mostra proveniência/confiança;
13. recomendação básica de propulsão recalcula massa por candidato e explica ranking;
14. cadastro assistido por URL usa staging/revisão e controles de segurança previstos;
15. alcance FPV permite informar distância-alvo e calcular FSPL, margem e potência teórica mínima quando dados suficientes existirem;
16. alcance FPV distingue realized gain e evita dupla contagem de perdas;
17. link FPV é explicitamente separado do link de controle RC;
18. comparação de variantes funciona;
19. testes de referência estão verdes;
20. documentação corresponde ao comportamento executável.

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

Módulos avançados como terreno/DEM, Fresnel, radio horizon, compliance regional ou link RC podem permanecer posteriores à 1.0 se isso for explicitamente decidido e documentado.

## 9. Mudança de cálculo entre releases

Se uma fórmula mudar materialmente:

- incrementar modelVersion;
- registrar no changelog;
- atualizar fixture esperada após revisão humana/técnica;
- informar impacto esperado;
- não esconder diferença como simples refactor.

Isso inclui mudanças em fórmulas RF, semântica de perdas/ganhos ou política de margem que alterem resultados.

## 10. Release checklist

```text
[ ] typecheck
[ ] lint
[ ] unit tests
[ ] integration tests
[ ] build
[ ] fixtures numéricas revisadas
[ ] fixtures RF revisadas (quando aplicável)
[ ] schemas/migrations revisados
[ ] staging/publicação revisados
[ ] light/dark verificados
[ ] acessibilidade básica verificada
[ ] docs atualizadas
[ ] changelog atualizado
[ ] limitações conhecidas registradas
[ ] nenhum segredo versionado
```
