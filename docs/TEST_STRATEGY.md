# Estratégia de Testes — DroneCalc

**Versão:** 0.3

## 1. Objetivo

O DroneCalc produz resultados técnicos; portanto testes numéricos, testes da Bancada e testes de integridade da ingestão são parte do produto, não apenas da implementação.

Prioridade:

1. motor de cálculo;
2. validação de dados e unidades;
3. dados/curvas de bancada;
4. ingestão/staging/proveniência;
5. migrações/persistência;
6. integração entre análise e projeto;
7. UI e acessibilidade;
8. regressão visual crítica.

## 2. Pirâmide de testes

### Unitários — maioria

- conversões;
- fórmulas;
- potência `V × I`;
- eficiência estática `gf/W`;
- pitch speed teórica;
- interpolação;
- compatibilidade;
- scoring;
- normalização;
- validators;
- parsers/extractors determinísticos;
- reconciliação de fontes;
- migrações puras.

### Integração

- `DroneProject` → análise completa;
- catálogo → resolução de componente;
- bench test aprovado → lookup/interpolação → análise;
- import CSV de bancada → preview → validação → persistência;
- import JSON/CSV → domínio;
- URL/import job → staging;
- asset → object storage + metadados;
- AI provider → schema validation → staging;
- repository → persistência;
- profile → scoring.

### UI

- criação de projeto;
- adicionar/substituir componente;
- seção Bancada: tabela, edição/importação, warnings, proveniência e gráficos;
- cadastro assistido e revisão de campos extraídos;
- exibir conflitos/evidências;
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
- zero/negativos inválidos conforme contexto;
- potência declarada/medida divergente de `V × I` gera warning sem sobrescrever fonte.

### Bancada — eficiência estática

- empuxo `gf` + potência W → `gf/W`;
- divisão por zero bloqueada;
- potência ausente indisponibiliza cálculo;
- valor é rotulado com unidade;
- não é convertido para percentual;
- origem `calculated` quando derivado.

### Bancada — pitch speed

- `pitch_in → pitch_m`;
- `pitch_m × RPM → m/min`;
- conversão correta para km/h;
- pitch zero/inválido conforme contrato;
- RPM negativo inválido;
- resultado sempre identificado como **Velocidade teórica de passo**;
- teste de UI impede rótulo `Velocidade máxima`/`Velocidade do drone` para essa fórmula;
- aviso `PITCH_SPEED_NOT_AIRCRAFT_SPEED` ou equivalente permanece visível/contextual.

### Bancada — RPM sem carga

- `KV × V` como referência ideal;
- não usado como RPM carregado;
- RPM de bancada pode ser comparado à referência apenas com tolerância/semântica adequada;
- `loaded_rpm_ratio` não é chamado de eficiência automaticamente.

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
- amostra inválida;
- ponto interpolado preserva referência aos pontos de origem;
- nenhum caminho padrão extrapola silenciosamente.

### Hover

- massa divisível igualmente;
- required thrust encontrado exato;
- required thrust interpolado;
- fora da curva;
- motor count inválido;
- lookup por empuxo não assume linearidade de throttle.

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

## 7. Compatibilidade e warnings

Cada código de warning deve ter testes independentes.

Exemplos gerais:

- `BATTERY_ESC_VOLTAGE_EXCEEDED`;
- `MOTOR_VOLTAGE_EXCEEDED`;
- `ESC_CURRENT_EXCEEDED`;
- `ESC_CURRENT_MARGIN_LOW`;
- `BATTERY_CURRENT_MARGIN_LOW`;
- `PROP_FRAME_DIAMETER_EXCEEDED`;
- `INSUFFICIENT_DATA_*`.

Warnings esperados para Bancada incluem equivalentes a:

- `BENCH_VOLTAGE_MISSING`;
- `BENCH_POWER_INCONSISTENT`;
- `BENCH_EFFICIENCY_UNIT_AMBIGUOUS`;
- `BENCH_THRUST_UNIT_AMBIGUOUS`;
- `BENCH_RPM_EXCEEDS_NO_LOAD_REFERENCE`;
- `BENCH_SAMPLE_DUPLICATE`;
- `BENCH_SAMPLE_INVALID`;
- `BENCH_INTERPOLATION_OUT_OF_RANGE`;
- `PITCH_SPEED_NOT_AIRCRAFT_SPEED`.

Testar limites exatamente iguais, imediatamente abaixo e imediatamente acima quando a regra tiver threshold.

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
├── ingestion/
└── profiles/
```

Criar projetos/curvas de referência:

- `minimal-valid-quad`;
- `mini-long-range-reference`;
- `electrical-incompatible`;
- `missing-bench-data`;
- `sub250-overweight`;
- `bench-valid-with-voltage`;
- `bench-missing-voltage`;
- `bench-ambiguous-units`;
- `bench-interpolation-out-of-range`.

Fixtures de ingestão devem incluir HTML/imagens de teste próprias ou licenciadas para teste, sem depender da disponibilidade de sites externos no CI.

Não copiar planilhas/imagens de terceiros como fixture redistribuível sem verificar permissão. Quando uma referência externa inspirar um caso, criar fixture sintética equivalente.

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
- potência DC cresce linearmente com corrente mantendo tensão;
- pitch speed teórica cresce linearmente com RPM mantendo pitch;
- interpolação retorna valor entre extremos para curva monotônica;
- autonomia básica diminui quando corrente aumenta mantendo demais entradas;
- TWR diminui quando massa aumenta mantendo empuxo.

## 12. Importação de arquivos e Bancada

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

Para CSV de bancada, cobrir adicionalmente:

- preview antes de persistir;
- cancelamento não altera o banco;
- cabeçalho `Aceleração` pode exigir mapeamento para `throttlePercent`, sem alterar semântica física;
- `Empuxo (100g)` é tratado como unidade/escala ambígua até confirmação/regra confiável;
- coluna `Eficiência` sem unidade não é assumida automaticamente como percentual nem `gf/W`;
- tensão ausente não é substituída por nominal silenciosamente;
- power/efficiency derivados só aparecem quando entradas válidas existem.

## 13. Ingestão por URL e multimodal

Seguir `ASSISTED_INGESTION.md` e `MANUFACTURER_SCRAPING.md`.

### Segurança de fetch

Testar obrigatoriamente:

- rejeição de `file:`, `ftp:` e protocolos não permitidos;
- loopback IPv4/IPv6;
- redes privadas/link-local;
- redirect público → destino privado;
- resolução/rebinding quando a implementação exigir revalidação;
- excesso de redirects;
- timeout;
- payload acima do limite;
- MIME declarado diferente do conteúdo quando detectável;
- imagem/documento excessivamente grande;
- quantidade excessiva de assets;
- falha parcial sem persistência autoritativa indevida.

### Extração determinística

- JSON-LD válido;
- tabela HTML;
- unidades pt-BR/internacionais;
- campo ausente permanece ausente;
- valor ambíguo não vira número silenciosamente;
- valor bruto e normalizado são preservados.

### IA textual/visual

Usar adapter fake/determinístico nos testes principais; não depender de um modelo Ollama real no CI unitário.

Cobrir:

- saída válida;
- JSON/schema inválido;
- campo extra não permitido;
- timeout/cancelamento;
- provider indisponível;
- modelo sem capability visual;
- imagem técnica → campos esperados no staging;
- confidence alta não publica automaticamente;
- prompt injection presente no conteúdo remoto não altera permissões/fluxo;
- provider não recebe credenciais/acesso SQL.

### Evidência e conflitos

- campo extraído aponta para source/asset;
- região de imagem opcional é validada quando presente;
- HTML e imagem com mesmo valor/unidade equivalente não criam falso conflito;
- HTML `60 mΩ` × imagem `64 mΩ` cria conflito/revisão;
- valores diferentes sob condições distintas não são achatados em um único campo;
- decisão de revisão mantém histórico;
- reprocessamento por modelo diferente não sobrescreve revisão publicada.

### Extraído versus derivado

- `KV` extraído permanece dado da fonte;
- `Kt` calculado recebe `formulaId`/proveniência de cálculo;
- cálculo derivado não é persistido como declaração do fabricante;
- warning de consistência não altera automaticamente valores importados.

### Gráficos em imagem

Quando digitalização for implementada:

- eixos/unidades ausentes impedem promoção automática;
- escala logarítmica é tratada explicitamente;
- pontos ficam ligados ao asset/região/método;
- CSV/tabela original tem preferência sobre pontos digitalizados;
- curva digitalizada não recebe automaticamente confiança de medição alta.

## 14. Persistência

- save/get/list/delete;
- autosave não perde alteração final;
- migration mantém dados;
- falha de storage é apresentada sem corromper estado em memória;
- componentes referenciados não desaparecem silenciosamente;
- staging e catálogo publicado permanecem separados;
- bench test e samples mantêm integridade referencial;
- object storage e metadados PostgreSQL não ficam inconsistentes após falha transacional/compensação prevista.

## 15. UI

Testar comportamento, não implementação interna.

Exemplos:

- projeto vazio mostra próximo passo;
- número sem unidade não é aceito/renderizado como métrica final;
- dados insuficientes não aparecem como zero;
- `danger` tem texto além de cor;
- modal de fórmula tem foco correto;
- formulário apresenta mensagem associada ao campo inválido;
- revisão de import mostra fonte/evidência e conflito sem depender apenas de cor;
- Bancada mostra tabela com unidades e estado/proveniência;
- valor derivado abre explicação da fórmula;
- pitch speed aparece como teórica e com aviso;
- não existe gráfico padrão colocando A, gf, RPM e `gf/W` numa única escala Y;
- pontos interpolados são distinguíveis dos medidos.

## 16. Temas

Testar pelo menos:

- theme system;
- light;
- dark;
- persistência da preferência;
- ausência de cores hardcoded em componentes críticos via lint/review.

## 17. Cobertura

Cobertura percentual isolada não é objetivo suficiente. Requisito do motor:

- 100% das fórmulas/regras críticas possuem casos explícitos;
- branches de validação e warnings críticos são cobertos;
- fixtures de referência existem.

Para Bancada, `V × I`, `gf/W`, pitch speed, unidade ambígua, tensão ausente e bloqueio de extrapolação são caminhos críticos.

Para ingestão, controles anti-SSRF, staging/publicação e validação de schema são caminhos críticos e exigem casos explícitos.

Threshold automatizado pode ser adotado depois, sem substituir revisão de casos.

## 18. CI futuro

Pipeline mínimo:

```text
install
→ typecheck
→ lint
→ unit tests
→ integration tests
→ build
```

Adicionar E2E conforme estabilidade. Testes de ingestão não devem depender de internet pública no CI.

## 19. Critério de release

Não liberar versão marcada como estável se:

- teste crítico falha;
- fórmula mudou sem versão/documentação;
- import/migration pode causar perda silenciosa;
- análise apresenta estimativa como medição;
- Bancada apresenta `gf/W` como percentual;
- pitch speed é apresentada como velocidade real/máxima do drone;
- curva pode extrapolar silenciosamente;
- gráfico de bancada usa escala única enganosa para grandezas incompatíveis por padrão;
- dado extraído por IA pode ser publicado sem staging/revisão prevista;
- fetch remoto permite acesso indevido a rede interna;
- evidência de campos importados é perdida;
- existe regressão conhecida de compatibilidade elétrica.