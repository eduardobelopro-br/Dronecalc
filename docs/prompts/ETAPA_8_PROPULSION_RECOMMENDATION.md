# Prompt de Implementação — Recomendação de Propulsão Orientada ao Uso

> Executar somente quando domínio, unidades, massa, catálogo, perfis e pelo menos uma rota válida de propulsão (Bancada e/ou Physics Engine conforme roadmap) estiverem disponíveis.

Você é a IA implementadora do DroneCalc. Antes de alterar código, leia obrigatoriamente `docs/PRD.md`, `docs/PRODUCT_SPEC.md`, `docs/ARCHITECTURE.md`, `docs/DOMAIN_MODEL.md`, `docs/CALCULATION_ENGINE.md`, `docs/PHYSICS_ENGINE.md`, `docs/BENCH_DATA_WORKSPACE.md`, `docs/PROPULSION_RECOMMENDATION.md`, `docs/FLIGHT_PROFILES.md`, `docs/COMPONENT_CATALOG.md`, `docs/TEST_STRATEGY.md`, `docs/ROADMAP.md` e os ADRs vigentes.

## Objetivo

Implementar o mecanismo que, a partir das peças já adicionadas ao projeto e do perfil de uso, recalcula a massa para cada candidato e recomenda conjuntos motor + hélice + bateria/tensão tecnicamente adequados e eficientes.

## Processo obrigatório

1. inspecione o código real antes de propor mudanças;
2. identifique contratos já existentes e não os duplique;
3. liste riscos, dependências e dados ausentes;
4. proponha a menor arquitetura coerente;
5. implemente fora da UI as regras de massa, compatibilidade, avaliação e ranking;
6. adicione testes unitários e de integração;
7. execute typecheck, lint, testes e build aplicáveis;
8. faça auto-auditoria de regressões, unidades, segurança e semântica física;
9. gere relatório da etapa e atualize documentação se necessário.

Não crie múltiplos agentes sem autorização expressa.

## Invariantes obrigatórios

- a massa final é recalculada para cada candidato;
- massa ausente nunca vira zero silenciosamente;
- a recomendação final é de conjunto de propulsão, não motor isolado;
- compatibilidades `danger` são avaliadas antes do score e não podem ser mascaradas;
- dados de bancada aprovados têm prioridade quando aplicáveis;
- Physics Engine só é usado quando houver parâmetros suficientes;
- heurística gera/filtra candidatos, não fabrica empuxo/corrente;
- interpolação não extrapola curva silenciosamente;
- eficiência deve ser avaliada no regime relevante, não apenas em empuxo máximo;
- fases operacionais sem dados não entram como consumo zero;
- `profileScore`, `dataCoverage` e `confidence` são conceitos separados;
- ranking deve ser determinístico para mesmas entradas e versões;
- cada recomendação deve explicar por que foi classificada naquela posição.

## Pipeline esperado

```text
catálogo/heurísticas
→ candidatos plausíveis
→ compatibilidade mecânica/elétrica
→ variante virtual do projeto
→ massa de decolagem do candidato
→ empuxo requerido por motor
→ TWR/reserva
→ bancada ou Physics Engine
→ corrente/potência/eficiência/autonomia disponíveis
→ score por perfil
→ ranking + explicação + cobertura/confiança
```

## Testes mínimos

Implemente casos que provem:

- motor mais pesado altera massa e hover requerido;
- motor de maior empuxo máximo não vence automaticamente Mini Long Range;
- candidato eficiente pode vencer com TWR menor quando ainda atende ao mínimo;
- Racing pode priorizar reserva/TWR conforme o perfil;
- troca de bateria altera massa/tensão/autonomia;
- `danger` invalida candidato;
- curva fora da faixa não é extrapolada;
- dados insuficientes produzem estado explícito;
- score não é inflado por baixa confiança;
- ranking e explicações são reproduzíveis.

## Liberdade para melhorar

Você pode modificar a abordagem, contratos, algoritmo de ranking, estratégia de busca ou estrutura de módulos se encontrar solução mais eficaz, eficiente, simples, segura ou testável. Antes de uma mudança relevante, explique a limitação da proposta atual e o trade-off da alternativa.

A liberdade de melhoria não autoriza:

- enfraquecer hard constraints;
- inventar valores físicos;
- mover fórmulas para React/UI;
- esconder falta de dados;
- misturar confiança com desempenho físico;
- recomendar motor sem contexto de hélice/tensão quando o ranking depende desses dados;
- introduzir dependência desnecessária sem consultar documentação oficial e justificar.

Atualize ADR/documentação quando a melhoria alterar uma decisão estrutural.

## Fora de escopo desta etapa

Não implemente automaticamente otimização global de milhares de combinações, compra automática, marketplace, CFD, previsão de velocidade real por pitch speed ou planejamento de rota. O objetivo é um recomendador técnico explicável sobre candidatos do catálogo e dados disponíveis.
