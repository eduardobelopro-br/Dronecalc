# ADR 0002 — PostgreSQL, Object Storage e Ollama como infraestrutura do MVP ampliado

**Status:** accepted  
**Data:** 2026-09-10

## Contexto

O escopo do DroneCalc evoluiu além de uma calculadora local. O produto passa a prever:

- catálogo crescente de componentes;
- projetos e variantes persistidos;
- curvas de bancada;
- fontes/evidências e revisões;
- cadastro de equipamentos a partir de URL;
- extração de imagens e dados;
- staging e revisão humana;
- uso de IA local via Ollama;
- heurísticas e base de conhecimento;
- possível busca semântica futura.

Nesse cenário, IndexedDB como persistência principal exigiria migração posterior e dificultaria governança de dados, ingestão backend e processamento por IA.

## Decisão

Adotar:

- **PostgreSQL** como fonte principal de dados persistidos;
- **MinIO** no ambiente local para object storage compatível com S3;
- outro storage S3-compatible pode substituir MinIO em produção futura;
- **Ollama** como primeiro runtime de IA local, sempre atrás de uma interface `AiProvider`;
- **IndexedDB** apenas como cache/draft/preferências/offline auxiliar;
- backend TypeScript como responsável por persistência, importação por URL, object storage e integração de IA;
- monorepo/workspace como estrutura inicial recomendada para compartilhar contratos e calculation engine sem duplicação.

## Regra de dados

PostgreSQL armazena dados estruturados e metadados. Arquivos/imagens grandes não devem ser armazenados diretamente como blobs no banco por padrão.

Dados extraídos por IA não entram diretamente no catálogo oficial. Devem passar por:

```text
extraction → schema validation → staging → human review → publication
```

## Segurança da importação por URL

A aplicação deverá tratar URLs fornecidas pelo usuário como entrada não confiável. O fetcher deve incluir proteção contra SSRF, redirects para redes privadas, timeout, limite de bytes e validação de tipo de conteúdo.

## Consequências positivas

- escala maior de catálogo sem migração estrutural precoce;
- integridade relacional e migrations explícitas;
- consultas e filtros técnicos mais adequados;
- melhor rastreabilidade de fontes e revisões;
- suporte natural a ingestão backend;
- imagens fora do banco;
- possibilidade futura de pgvector sem banco vetorial separado;
- IA local substituível.

## Consequências negativas

- maior custo de bootstrap;
- necessidade de Docker/serviços locais;
- migrations e operação do banco desde cedo;
- API passa a fazer parte da arquitetura do MVP ampliado;
- testes de integração exigem infraestrutura adicional.

## Alternativas consideradas

### IndexedDB como banco principal

Rejeitado para o escopo atual. Continua útil como cache/drafts, mas não como fonte autoritativa do catálogo e das evidências.

### SQLite como banco principal

Seria simples para desktop/local, mas o escopo de API, catálogo central, jobs de ingestão e crescimento relacional favorece PostgreSQL. SQLite pode ser reconsiderado para um modo desktop totalmente autônomo no futuro por meio de adapter próprio.

### Armazenar imagens no PostgreSQL

Rejeitado como padrão para evitar crescimento desnecessário do banco e complicar backup/serving de mídia. Metadados ficam no PostgreSQL e binários no object storage.

### Serviço de IA em nuvem como dependência principal

Não adotado como requisito. Ollama permite operação local e privacidade. A abstração `AiProvider` preserva a possibilidade de outros providers no futuro.

## Relação com ADR 0001

ADR 0001 permanece válido quanto a:

- interface web-first;
- TypeScript;
- motor de cálculo independente de React;
- capacidade futura de reutilização em API/desktop/CLI.

Ficam supersedidas para o MVP ampliado as partes que tratavam backend como não necessário e IndexedDB como persistência principal inicial.

## Impacto

Este ADR altera ou fundamenta:

- `docs/ARCHITECTURE.md`;
- `docs/ROADMAP.md`;
- futuras specs de persistência;
- cadastro por URL;
- AI/Ollama;
- política de object storage;
- futura decisão sobre pgvector.
