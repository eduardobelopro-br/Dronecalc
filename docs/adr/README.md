# Architecture Decision Records (ADR)

Este diretório registra decisões arquiteturais relevantes do DroneCalc.

## Quando criar um ADR

Criar ADR quando uma decisão:

- altera dependências estruturais;
- muda estratégia de persistência;
- muda arquitetura de unidades;
- adiciona backend/sincronização;
- muda formato de catálogo/importação;
- altera estratégia de cálculo de forma estrutural;
- introduz tecnologia que afeta várias features;
- substitui uma decisão anterior importante.

## Formato

```text
# ADR NNNN — Título

Status: proposed | accepted | superseded | deprecated
Data: YYYY-MM-DD

## Contexto
## Decisão
## Consequências
## Alternativas consideradas
## Impacto em documentação/testes
```

## Regras

- ADR explica o motivo da decisão, não apenas o resultado;
- decisão superseded não é apagada;
- novo ADR referencia o anterior;
- mudanças de requisito de produto continuam pertencendo ao PRD/Product Spec, não apenas ao ADR.

## Índice

- [`0001-web-first-core-engine.md`](0001-web-first-core-engine.md) — Web-first e motor de cálculo independente. Continua válido nesses princípios; a estratégia de persistência/backend foi parcialmente substituída pelo ADR 0002.
- [`0002-postgresql-object-storage-ollama.md`](0002-postgresql-object-storage-ollama.md) — PostgreSQL como fonte principal, object storage para arquivos e Ollama atrás de interface substituível no MVP ampliado.
