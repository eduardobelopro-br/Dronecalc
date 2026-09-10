# Guia para Implementação Assistida por IA — DroneCalc

**Versão:** 0.2

## 1. Objetivo

Este documento é o contrato operacional para qualquer agente de IA utilizado para implementar, revisar ou refatorar o DroneCalc.

A IA **pode propor e aplicar uma solução mais eficiente** do que a sugerida na documentação quando houver benefício técnico claro, desde que preserve os requisitos do produto, explique a mudança, mantenha compatibilidade/migração adequada, teste a alternativa e atualize os documentos afetados.

## 2. Ordem obrigatória de leitura

Antes de alterar código relevante, ler:

1. `README.md`;
2. `docs/README.md`;
3. `docs/PRD.md`;
4. `docs/PRODUCT_SPEC.md`;
5. `docs/ARCHITECTURE.md`;
6. `docs/ROADMAP.md`;
7. documento específico da área;
8. `docs/TEST_STRATEGY.md`;
9. `docs/DEVELOPMENT.md`;
10. ADRs aplicáveis.

Mudanças visuais exigem `docs/NEXO_DESIGN_SYSTEM.md` e `docs/UX_SPEC.md`.

Mudanças matemáticas exigem `docs/CALCULATION_ENGINE.md`.

Mudanças de persistência/infraestrutura exigem considerar `docs/adr/0002-postgresql-object-storage-ollama.md`.

## 3. Arquitetura vigente resumida

- Web: React + TypeScript + Vite;
- API: Node.js + TypeScript;
- PostgreSQL: fonte principal de dados persistidos;
- MinIO/S3-compatible: imagens/documentos;
- Ollama: primeiro provider de IA local, atrás de interface substituível;
- IndexedDB: cache/drafts/preferências/offline auxiliar;
- domínio e calculation engine independentes de UI, banco e IA.

## 4. Regras inegociáveis

A IA não deve:

- colocar fórmulas de engenharia dentro de componentes React;
- inventar dados técnicos ausentes;
- tratar `undefined` como zero;
- apresentar estimativa como medição;
- extrapolar curvas silenciosamente;
- hardcodar cores de features em vez de tokens NEXO;
- acessar PostgreSQL diretamente a partir do domínio;
- acoplar domínio/calculation engine a Ollama;
- armazenar imagens grandes no PostgreSQL por padrão;
- publicar dados extraídos por IA sem schema/staging/revisão;
- criar fetch por URL sem proteção anti-SSRF;
- alterar schemas persistidos sem migration/versionamento;
- remover testes para fazer CI passar;
- introduzir secrets no repositório;
- afirmar que uma configuração é segura apenas porque passou em cálculos de compatibilidade.

## 5. Liberdade para melhorar a solução

A IA pode melhorar:

- estrutura de pastas/workspaces;
- nomes de tipos/funções;
- algoritmos;
- estratégia de units;
- abstrações de repository;
- framework HTTP;
- ORM/query builder/migration tool;
- composição de componentes;
- performance;
- estratégia de testes;
- biblioteca auxiliar;

se a melhoria:

1. reduz complexidade ou aumenta correção/manutenibilidade;
2. não viola PRD/specs;
3. preserva fronteiras arquiteturais;
4. não reduz transparência dos cálculos;
5. usa APIs reais verificadas em documentação oficial;
6. possui testes;
7. é explicada no relatório da etapa;
8. atualiza arquitetura/ADR quando estrutural.

## 6. Regra para divergência da documentação

Se a IA identificar abordagem melhor:

```text
identificar limitação
→ explicar alternativa
→ comparar trade-offs
→ verificar requisitos
→ implementar se segura e dentro do escopo
→ testar
→ atualizar documentação
→ registrar riscos residuais
```

Não manter documentação sabidamente desatualizada apenas para seguir um plano antigo.

## 7. Fluxo de trabalho por etapa

### Antes

- verificar branch/status;
- ler docs/ADRs relevantes;
- inspecionar código atual;
- identificar dependências da etapa;
- não repetir implementação existente;
- apresentar plano curto antes de mudança grande.

### Durante

- mudanças coesas;
- build utilizável;
- testes junto da implementação;
- escopo lateral mínimo;
- compatibilidade de dados;
- segurança por padrão.

### Depois

Executar comandos reais equivalentes a:

```bash
npm run typecheck
npm run lint
npm run test
npm run build
```

Registrar resultados reais. Nunca afirmar que teste/comando passou sem execução.

## 8. Relatório obrigatório

Cada etapa relevante gera relatório em `docs/reports/` contendo:

```text
# Relatório — Etapa X

## Objetivo
## Estado encontrado
## Plano executado
## Implementado
## Arquivos alterados
## Decisões técnicas
## Melhorias sobre o plano original
## Dependências adicionadas
## Testes/comandos executados
## Resultados
## Auto-auditoria
## Pendências
## Riscos
## Próximo passo recomendado
```

## 9. Alteração matemática

Ao criar/mudar fórmula:

- nomear formula/model version;
- declarar unidades;
- documentar hipótese;
- criar caso de referência;
- testar limites;
- atualizar `CALCULATION_ENGINE.md`;
- indicar impacto em resultados existentes.

## 10. Alteração de perfis/heurísticas

Ao mudar pesos/targets/regras:

- versionar;
- testar score/candidate generation;
- validar que `danger` não é mascarado;
- separar heurística de compatibilidade;
- atualizar `FLIGHT_PROFILES.md`/docs de knowledge;
- explicar impacto em ranking/recomendação.

## 11. Alteração visual

- reutilizar NEXO primitives/tokens;
- suportar claro/escuro/sistema;
- testar teclado/foco;
- não usar apenas cor para status;
- manter unidades visíveis;
- implementar estados vazio/erro/incompleto.

## 12. Persistência e infraestrutura

Ao mudar dados:

- schema explícito;
- migration;
- testes de integração;
- transaction boundaries;
- import inválido não persiste parcialmente.

Ao lidar com arquivos:

- guardar binário em object storage;
- metadados no PostgreSQL;
- validar MIME/tamanho;
- considerar hash/deduplicação.

## 13. Ingestão por URL

Obrigatório considerar:

- somente HTTP/HTTPS;
- loopback/private/link-local bloqueados;
- redirects revalidados;
- timeout;
- limite de bytes;
- tipos de conteúdo permitidos;
- sanitização da apresentação;
- nenhum cookie/token pessoal reaproveitado;
- logs sem credenciais.

## 14. Ollama/IA

- adapter atrás de `AiProvider` ou abstração equivalente;
- modelo configurável;
- saída estruturada validada;
- timeout/cancelamento;
- prompt/model/version rastreáveis;
- falha da IA não corrompe persistência;
- IA sugere e extrai, não vira autoridade técnica automaticamente.

## 15. Prompt-base para uma etapa

```text
Você está implementando uma etapa do projeto DroneCalc.

Repositório: eduardobelopro-br/Dronecalc

Leia README.md, docs/README.md, PRD, Product Spec, Architecture, Roadmap,
Development, Test Strategy e os ADRs/documentos específicos da etapa.

Inspecione o código real antes de propor mudanças.

Implemente a etapa de forma completa, coesa, segura e testável.

Regras principais:
- domínio e calculation engine independentes de React, PostgreSQL e Ollama;
- PostgreSQL acessado por adapters/repositories;
- imagens/documentos em object storage, não como blobs grandes no banco por padrão;
- não invente dados técnicos ausentes;
- resultados físicos preservam unidade, origem e confiança;
- não extrapole curvas silenciosamente;
- UI segue NEXO Design System;
- ingestão por URL deve ser segura contra SSRF e payloads abusivos;
- saída de IA passa por schema/staging/revisão;
- adicione/atualize testes;
- execute typecheck, lint, test e build quando aplicáveis;
- não remova testes para obter CI verde;
- não versione secrets;
- atualize documentação quando mudar contrato/arquitetura.

Você tem liberdade explícita para substituir a solução sugerida por uma abordagem
mais eficiente, simples, segura ou robusta. Se fizer isso, explique o motivo,
compare trade-offs, preserve requisitos, verifique documentação oficial,
teste a alternativa e atualize docs/ADR quando necessário.

Ao final, gere relatório com implementação, decisões, testes, auto-auditoria,
riscos, pendências e próximo passo recomendado.
```

## 16. Próxima etapa — orientação oficial

A orientação executável atual para a primeira implementação está em:

`docs/prompts/ETAPA_1A_BOOTSTRAP_FULLSTACK.md`

Ela substitui o antigo prompt de 1A que tratava o DroneCalc como frontend-only.

O escopo da Etapa 1A é **bootstrap de workspace + Web + API + packages puros**, sem ainda implementar PostgreSQL, MinIO ou Ollama. Esses serviços entram na Etapa 1C, depois da base e do NEXO Design System.

## 17. Revisão por IA

Uma IA revisora deve procurar explicitamente:

- unidade incorreta;
- arredondamento prematuro;
- confusão massa × força;
- tensão nominal usada onde deveria ser tensão cheia;
- C-rating tratado como medição;
- extrapolação;
- `undefined → 0`;
- score ocultando incompatibilidade;
- heurística tratada como compatibilidade;
- fórmula duplicada na UI/API;
- cores não-NEXO;
- migration ausente;
- SQL/ORM vazando para domínio;
- Ollama vazando para domínio;
- blobs grandes no PostgreSQL;
- SSRF no importador de URL;
- trust indevido em saída de IA;
- segredos/configuração insegura;
- falta de teste de limite.

## 18. Critério de qualidade

O objetivo não é apenas fazer funcionar. O código deve deixar claro **o que sabemos, o que calculamos, o que extraímos, o que inferimos e o que apenas estimamos**.
