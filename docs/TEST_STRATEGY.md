# Estratégia de Testes — DroneCalc

**Versão:** 0.1

## 1. Objetivo

O DroneCalc produz resultados técnicos; portanto testes numéricos são parte do produto, não apenas da implementação.

Prioridade:

1. motor de cálculo;
2. validação de dados;
3. migrações/persistência;
4. integração entre análise e projeto;
5. UI e acessibilidade;
6. regressão visual crítica.

## 2. Pirâmide de testes

### Unitários — maioria

- conversões;
- fórmulas;
- interpolação;
- compatibilidade;
- scoring;
- normalização;
- validators;
- migrações puras.

### Integração

- `DroneProject` → análise completa;
- catálogo → resolução de componente;
- import JSON/CSV → domínio;
- repository → persistência;
- profile → scoring.

### UI

- criação de projeto;
- adicionar/substituir componente;
- exibir dados ausentes;
- abrir “Como foi calculado?”;
- alertas;
- comparação;
- tema/acessibilidade.

### E2E

Poucos fluxos críticos após MVP funcional.

## 3. Ferramentas sugeridas

- Vitest;
- Testing Library;
- Playwright para E2E futuro;
- axe ou ferramenta equivalente para verificações automatizadas de acessibilidade.

A escolha pode mudar via decisão arquitetural.

## 4. Precisão numérica

Nunca comparar floats complexos com igualdade estrita quando arredondamento estiver envolvido.

Usar tolerância explícita:

```ts
expect(actual).toBeCloseTo(expected, precision)
```

A tolerância deve refletir a fórmula, não mascarar erros.

## 5. Separar cálculo de apresentação

Testes do motor usam valores não arredondados. Arredondamento para `21,4 min` é testado na camada de formatting/UI.

Nunca arredondar cedo dentro do cálculo intermediário sem necessidade física/documentada.

## 6. Casos de referência obrigatórios

### Massa

- uma peça;
- quantidades;
- múltiplas categorias;
- massa ausente;
- massa zero válida quando aplicável;
- massa negativa inválida.

### Bateria

- mAh → Ah;
- tensão nominal;
- tensão cheia;
- Wh;
- C-rating;
- química com valores por célula diferentes;
- capacidade inválida.

### Potência

- V × A;
- entradas medidas;
- zero/negativos inválidos conforme contexto.

### TWR

- quad simétrico;
- diferentes números de motores;
- empuxo insuficiente;
- massa zero bloqueada.

### Interpolação

- ponto exato inferior;
- ponto exato superior;
- ponto intermediário;
- múltiplos segmentos;
- alvo fora da faixa;
- dados duplicados;
- curva não ordenada;
- amostra inválida.

### Hover

- massa divisível igualmente;
- required thrust encontrado exato;
- required thrust interpolado;
- fora da curva;
- motor count inválido.

### Autonomia

- capacidade utilizável;
- reserva;
- corrente média;
- carga auxiliar;
- impedir reserva dupla;
- corrente zero inválida.

### CG

- massas simétricas → centro;
- peso deslocado;
- 3 eixos;
- posição ausente.

## 7. Compatibilidade

Cada código de warning deve ter testes independentes.

Exemplos:

- `BATTERY_ESC_VOLTAGE_EXCEEDED`;
- `MOTOR_VOLTAGE_EXCEEDED`;
- `ESC_CURRENT_EXCEEDED`;
- `ESC_CURRENT_MARGIN_LOW`;
- `BATTERY_CURRENT_MARGIN_LOW`;
- `PROP_FRAME_DIAMETER_EXCEEDED`;
- `INSUFFICIENT_DATA_*`.

Testar limites exatamente iguais, imediatamente abaixo e imediatamente acima.

## 8. Perfis e scoring

Para cada perfil:

- pesos carregam e validam;
- normalização soma corretamente;
- incompatibilidade crítica aplica regra definida;
- score explica contribuições;
- dados ausentes reduzem cobertura/confiança;
- Mini Long Range favorece eficiência/autonomia sobre potência bruta em fixtures projetadas para demonstrar essa diferença.

## 9. Golden fixtures

Manter fixtures legíveis em `tests/fixtures/`:

```text
tests/fixtures/
├── projects/
├── components/
├── bench-curves/
└── profiles/
```

Criar projetos de referência:

- `minimal-valid-quad`;
- `mini-long-range-reference`;
- `electrical-incompatible`;
- `missing-bench-data`;
- `sub250-overweight`.

Os números esperados devem ser documentados e revisados quando a versão do motor mudar.

## 10. Regressão do motor

Quando alterar uma fórmula:

1. adicionar/alterar teste que demonstra o motivo;
2. incrementar versão do modelo se resultados mudarem materialmente;
3. executar fixtures de regressão;
4. documentar diferença esperada;
5. não atualizar snapshots numericamente sem revisar a causa.

## 11. Property-based testing futuro

Útil para invariantes:

- massa total não diminui ao adicionar massa positiva;
- energia Wh cresce linearmente com capacidade mantendo tensão;
- interpolação retorna valor entre extremos para curva monotônica;
- autonomia básica diminui quando corrente aumenta mantendo demais entradas;
- TWR diminui quando massa aumenta mantendo empuxo.

## 12. Importação

Testes de JSON/CSV:

- formato válido;
- schema desconhecido;
- migration válida;
- migration que falha sem persistir parcial;
- CSV com vírgula/ponto decimal;
- unidades diferentes;
- cabeçalhos mapeados;
- linhas inválidas;
- arquivo vazio;
- duplicatas.

## 13. Persistência

- save/get/list/delete;
- autosave não perde alteração final;
- migration mantém dados;
- falha de storage é apresentada sem corromper estado em memória;
- componentes referenciados não desaparecem silenciosamente.

## 14. UI

Testar comportamento, não implementação interna.

Exemplos:

- projeto vazio mostra próximo passo;
- número sem unidade não é aceito/renderizado como métrica final;
- dados insuficientes não aparecem como zero;
- `danger` tem texto além de cor;
- modal de fórmula tem foco correto;
- formulário apresenta mensagem associada ao campo inválido.

## 15. Temas

Testar pelo menos:

- theme system;
- light;
- dark;
- persistência da preferência;
- ausência de cores hardcoded em componentes críticos via lint/review.

## 16. Cobertura

Cobertura percentual isolada não é objetivo suficiente. Requisito do motor:

- 100% das fórmulas/regras críticas possuem casos explícitos;
- branches de validação e warnings críticos são cobertos;
- fixtures de referência existem.

Threshold automatizado pode ser adotado depois, sem substituir revisão de casos.

## 17. CI futuro

Pipeline mínimo:

```text
install
→ typecheck
→ lint
→ unit tests
→ integration tests
→ build
```

Adicionar E2E conforme estabilidade.

## 18. Critério de release

Não liberar versão marcada como estável se:

- teste crítico falha;
- fórmula mudou sem versão/documentação;
- import/migration pode causar perda silenciosa;
- análise apresenta estimativa como medição;
- existe regressão conhecida de compatibilidade elétrica.
