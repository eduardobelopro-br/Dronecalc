# Prompt de Implementação — Etapa 1A: Bootstrap Full-Stack do DroneCalc

**Uso:** entregar este arquivo a uma IA de desenvolvimento para executar a primeira etapa de implementação.  
**Repositório:** `eduardobelopro-br/Dronecalc`  
**Etapa:** `1A — Bootstrap full-stack e workspace`

---

## PROMPT PARA A IA IMPLEMENTADORA

Você é responsável por implementar a **Etapa 1A — Bootstrap full-stack e workspace** do projeto DroneCalc.

Não comece codificando imediatamente. Primeiro inspecione o estado real do repositório e leia a documentação relevante.

### 1. Leitura obrigatória

Leia, nesta ordem:

1. `README.md`;
2. `docs/README.md`;
3. `docs/PRD.md`;
4. `docs/PRODUCT_SPEC.md`;
5. `docs/ARCHITECTURE.md`;
6. `docs/ROADMAP.md`;
7. `docs/DEVELOPMENT.md`;
8. `docs/TEST_STRATEGY.md`;
9. `docs/AI_IMPLEMENTATION_GUIDE.md`;
10. `docs/adr/0001-web-first-core-engine.md`;
11. `docs/adr/0002-postgresql-object-storage-ollama.md`.

Considere o código real do repositório como fonte de verdade para o estado atual. A documentação define intenção e contratos, mas se encontrar divergência, registre-a e resolva-a de forma explícita.

### 2. Fluxo obrigatório

Siga:

```text
entender estado atual
→ levantar riscos
→ apresentar plano curto
→ implementar
→ adicionar testes
→ executar validações
→ auto-auditar
→ atualizar docs necessárias
→ gerar relatório
```

Não use múltiplos agentes em paralelo sem autorização explícita do usuário.

### 3. Objetivo da etapa

Criar a fundação técnica mínima para que o DroneCalc possa evoluir para:

- aplicação Web React;
- API TypeScript;
- PostgreSQL;
- MinIO/S3-compatible object storage;
- Ollama;
- packages compartilhados de domínio e calculation engine.

**A Etapa 1A NÃO deve implementar PostgreSQL, MinIO, Ollama, scraping, catálogo real ou fórmulas de drone.** Ela apenas prepara uma base limpa para essas etapas.

### 4. Estrutura conceitual desejada

A estrutura inicial recomendada é:

```text
Dronecalc/
├── apps/
│   ├── web/
│   └── api/
├── packages/
│   ├── domain/
│   ├── calculation-engine/
│   └── contracts/
├── infrastructure/
├── docs/
├── package.json
├── tsconfig.base.json
└── .env.example
```

Você está autorizado a modificar essa estrutura se encontrar uma organização objetivamente mais simples, eficiente ou robusta.

Se modificar:

1. explique a limitação da estrutura proposta;
2. mostre o trade-off;
3. preserve separação de responsabilidades;
4. não introduza complexidade sem benefício;
5. atualize `docs/ARCHITECTURE.md` e crie/atualize ADR se a alteração for estrutural.

### 5. Regras arquiteturais inegociáveis

`packages/domain` não pode depender de:

- React;
- DOM/browser;
- API HTTP;
- PostgreSQL/ORM;
- MinIO/S3;
- Ollama;
- Zustand.

`packages/calculation-engine` deve permanecer puro e reutilizável, dependendo apenas de domínio/contratos realmente necessários.

A UI não deve conter fórmulas de engenharia.

A API não deve conter regras matemáticas duplicadas que pertençam ao calculation engine.

### 6. Escolha do workspace/package manager

Comece avaliando a solução mais simples para o repositório vazio.

`npm workspaces` é uma referência válida por reduzir ferramentas extras. Porém você pode escolher `pnpm` ou alternativa madura se houver ganho claro de manutenção/performance/ergonomia.

Se escolher algo diferente do caminho mais simples:

- verifique a documentação oficial;
- explique o benefício;
- fixe versão/lockfile adequadamente;
- não misture package managers.

### 7. Frontend

Criar `apps/web` com:

- React;
- TypeScript strict;
- Vite;
- página mínima de bootstrap;
- error boundary ou estratégia equivalente simples;
- teste mínimo;
- sem implementar ainda o NEXO Design System completo, que pertence à Etapa 1B.

Não criar uma UI grande nesta etapa.

Exemplo mínimo de intenção, não obrigação de implementação literal:

```tsx
export function App() {
  return (
    <main>
      <h1>DroneCalc</h1>
      <p>Engineering workspace bootstrap</p>
    </main>
  )
}
```

Se houver maneira melhor de validar o bootstrap com menos código, use-a.

### 8. API

Criar `apps/api` com Node.js + TypeScript e um endpoint mínimo:

```text
GET /health
```

Resposta conceitual:

```json
{
  "status": "ok",
  "service": "dronecalc-api"
}
```

Você pode usar um framework HTTP pequeno e mantido ou Node nativo. Se adicionar framework:

- confirme API atual na documentação oficial;
- não invente métodos;
- justifique a dependência;
- mantenha estrutura preparada para application services/adapters;
- não crie ORM ou conexão de banco nesta etapa.

### 9. Contracts package

Se houver valor real em compartilhar o contrato de health check, pode criar algo como:

```ts
export interface HealthResponse {
  status: 'ok'
  service: 'dronecalc-api'
}
```

Não transforme `packages/contracts` em depósito genérico. Se o pacote não agregar valor na Etapa 1A, você pode deixá-lo apenas preparado ou adiar conteúdo, desde que a decisão seja registrada.

### 10. Domain package

Criar pacote compilável/testável isoladamente.

Não implemente ainda componentes reais de drones além do mínimo estritamente necessário para provar a estrutura.

Exemplo aceitável:

```ts
export const DOMAIN_PACKAGE_VERSION = '0.1.0' as const
```

É igualmente aceitável não criar placeholder sem utilidade se o build do workspace puder validar o pacote de outra forma.

Evite código morto criado só para preencher diretórios.

### 11. Calculation engine package

Mesma regra: estrutura independente, testes mínimos e zero fórmula de drone nesta etapa.

Não antecipe:

- TWR;
- bateria;
- massa;
- autonomia;
- interpolação;
- compatibilidade.

Essas implementações terão etapas próprias.

### 12. TypeScript

Exigir modo estrito.

Uma base aceitável:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "useUnknownInCatchVariables": true
  }
}
```

Você pode ajustar flags que causem atrito injustificado no ecossistema escolhido, mas qualquer relaxamento relevante deve ser explicado.

### 13. Scripts raiz

Objetivo de ergonomia:

```bash
npm run dev
npm run build
npm run test
npm run typecheck
npm run lint
```

ou equivalentes coerentes com o package manager escolhido.

Preferir comandos raiz que executem os workspaces necessários sem exigir que o desenvolvedor entre manualmente em cada pasta.

### 14. Testes

Configurar Vitest ou alternativa já prevista/justificada.

Testes mínimos:

- `apps/web` renderiza sem erro;
- `apps/api` possui teste do health handler/endpoint sem depender de banco;
- packages compartilhados compilam;
- pelo menos um teste demonstra que calculation engine não depende de browser.

Não adicionar testes vazios apenas para aumentar contagem.

### 15. Lint e formatação

Configurar uma abordagem simples e mantida.

Evitar uma pilha de plugins desnecessários. O objetivo é:

- erros comuns;
- imports coerentes;
- formatação previsível;
- CI reproduzível.

### 16. CI

Criar workflow inicial no GitHub Actions para executar, no mínimo:

```text
install reproducível
→ typecheck
→ lint
→ test
→ build
```

Use cache apenas se não complicar desnecessariamente a primeira versão.

### 17. Ambiente e segredos

Criar `.env.example` somente com variáveis que façam sentido nesta etapa ou estejam claramente marcadas como futuras.

Nunca versionar:

- senha real;
- token;
- cookie;
- credencial de MinIO;
- credencial de PostgreSQL;
- secrets de produção.

Não exponha variáveis privadas da API por prefixo público do Vite.

### 18. Não implementar nesta etapa

É proibido expandir 1A para implementar:

- Docker Compose dos serviços;
- PostgreSQL;
- migrations/ORM;
- MinIO;
- Ollama;
- pgvector;
- fetch de páginas externas;
- scraping;
- cadastro por URL;
- catálogo de peças;
- autenticação;
- fórmulas físicas;
- otimização;
- telas completas do Builder.

Se descobrir que algo dessa lista é indispensável para o bootstrap, explique antes de adicioná-lo e mantenha o mínimo possível.

### 19. Código de referência para organização

Uma separação de entrada da API pode ser:

```ts
// apps/api/src/app.ts
export function createApp() {
  // criar/configurar aplicação HTTP sem iniciar listener aqui
}
```

```ts
// apps/api/src/server.ts
import { createApp } from './app'

const app = createApp()
// iniciar listener / bootstrap
```

A vantagem é permitir testar a aplicação sem abrir porta real.

Se o framework escolhido possuir padrão oficial melhor, siga o padrão oficial.

### 20. Segurança no bootstrap

Mesmo sem features externas ainda:

- não habilite CORS `*` indiscriminadamente se não houver necessidade;
- não exponha stack traces como contrato de API;
- não coloque secrets no frontend;
- valide configuração de ambiente antes de uso;
- evite dependências abandonadas;
- não use `eval`, shell arbitrário ou código remoto.

### 21. Liberdade para melhorar a implementação

Você tem autorização explícita para melhorar o plano e o código quando encontrar solução mais eficaz, eficiente, simples, segura ou testável.

Não precisa copiar os snippets deste documento literalmente.

Pode alterar:

- package manager;
- estrutura de pastas;
- framework HTTP;
- estratégia de build;
- organização de packages;
- lint/formatter;
- configuração de testes;
- nomes internos.

Mas a melhoria só é aceita se:

1. não quebrar requisitos do PRD/Product Spec;
2. preservar independência do domínio/calculation engine;
3. não antecipar escopo desnecessário;
4. usar APIs reais/documentadas;
5. possuir testes;
6. reduzir ou justificar dependências;
7. atualizar documentação quando necessário;
8. explicar o motivo no relatório.

Não refatore por preferência estética apenas.

### 22. Auto-auditoria obrigatória

Antes de finalizar, revise explicitamente:

- imports circulares;
- dependência de React dentro de packages puros;
- dependência de Node dentro de package que precise ser universal;
- scripts quebrados;
- conflito de tsconfig;
- duas versões desnecessárias da mesma dependência;
- secrets acidentalmente adicionados;
- arquivos gerados commitados indevidamente;
- vulnerabilidades óbvias de configuração;
- testes que não executam de verdade;
- README com comandos incorretos.

### 23. Comandos finais

Execute os comandos reais existentes no projeto, preferencialmente equivalentes a:

```bash
npm run typecheck
npm run lint
npm run test
npm run build
```

Se algum falhar:

- corrija a causa;
- não remova teste/regra só para obter verde;
- se a falha for ambiental, documente evidência e limitação.

Nunca afirme que um comando passou se ele não foi executado.

### 24. Critérios de aceite

A Etapa 1A está concluída somente se:

- há um workspace coerente;
- web inicia/compila;
- API inicia/compila;
- `/health` funciona;
- domain e calculation engine são pacotes independentes;
- TypeScript strict está ativo;
- lint/test/typecheck/build executam pela raiz;
- CI corresponde aos comandos locais;
- não há segredos;
- não foi antecipada implementação de PostgreSQL/Ollama/MinIO;
- README foi ajustado aos comandos reais;
- documentação relevante permanece consistente;
- relatório da etapa foi gerado.

### 25. Relatório obrigatório

Crie:

`docs/reports/RELATORIO_ETAPA_1A_BOOTSTRAP_FULLSTACK.md`

Formato:

```text
# Relatório — Etapa 1A Bootstrap Full-Stack

## Objetivo
## Estado encontrado
## Plano executado
## Arquivos criados/alterados
## Stack e versões
## Decisões técnicas
## Diferenças em relação ao plano original
## Dependências adicionadas e justificativa
## Testes/comandos executados
## Resultados
## Auto-auditoria
## Riscos residuais
## Pendências
## Próximo passo recomendado
```

### 26. Entrega Git

Implemente em branch própria, por exemplo:

```text
feat/1a-bootstrap-fullstack
```

Não faça merge em `main` sem autorização do usuário.

Ao final, apresente um resumo objetivo e deixe a mudança pronta para revisão.

---

## RESULTADO ESPERADO DA PRIMEIRA ETAPA

Depois de 1A, o repositório deve possuir somente a **fundação executável**:

```text
Web React
      │
      ├── shared contracts
      │
API TypeScript

Domain (puro)
Calculation Engine (puro)

infraestrutura futura preparada por estrutura,
mas ainda sem PostgreSQL/MinIO/Ollama implementados.
```

A próxima etapa, após revisão e aceite, será **1B — NEXO Design System base**. A infraestrutura Docker/PostgreSQL/MinIO/Ollama entra em **1C** conforme o roadmap.
