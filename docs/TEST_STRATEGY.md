# Estratégia de Testes — DroneCalc

**Versão:** 0.2

## 1. Objetivo

O DroneCalc produz resultados técnicos; portanto testes numéricos e testes de integridade da ingestão são parte do produto, não apenas da implementação.

Prioridade:

1. motor de cálculo;
2. validação de dados;
3. ingestão/staging/proveniência;
4. migrações/persistência;
5. integração entre análise e projeto;
6. UI e acessibilidade;
7. regressão visual crítica.

## 2. Pirâmide de testes

### Unitários — maioria

- conversões;
- fórmulas;
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
- import JSON/CSV → domínio;
- URL/import job → staging;
- asset → object storage + metadados;
- AI provider → schema validation → staging;
- repository → persistência;
- profile → scoring.

### UI

- criação de projeto;
- adicionar/substituir componente;
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
├── ingestion/
└── profiles/
```

Criar projetos de referência:

- `minimal-valid-quad`;
- `mini-long-range-reference`;
- `electrical-incompatible`;
- `missing-bench-data`;
- `sub250-overweight`.

Fixtures de ingestão devem incluir HTML/imagens de teste próprias ou licenciadas para teste, sem depender da disponibilidade de sites externos no CI.

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

## 12. Importação de arquivos

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

## 13. Ingestão por URL e multimodal

Seguir `ASSISTED_INGESTION.md`.

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
- revisão de import mostra fonte/evidência e conflito sem depender apenas de cor.

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
- dado extraído por IA pode ser publicado sem staging/revisão prevista;
- fetch remoto permite acesso indevido a rede interna;
- evidência de campos importados é perdida;
- existe regressão conhecida de compatibilidade elétrica.
