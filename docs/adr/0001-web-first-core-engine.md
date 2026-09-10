# ADR 0001 — Web-first com motor de cálculo independente

**Status:** accepted  
**Data:** 2026-09-10

## Contexto

O DroneCalc precisa começar simples, mas seu valor principal está no motor técnico. No futuro, esse motor pode ser reutilizado em aplicativo desktop, API, CLI ou otimizador. Introduzir backend, banco servidor ou acoplamento forte com React antes de validar fórmulas aumentaria complexidade sem benefício imediato.

## Decisão

Adotar arquitetura **web-first** para o MVP:

- React + TypeScript + Vite na interface;
- motor de cálculo em TypeScript puro;
- domínio independente de React/DOM;
- persistência local atrás de repositories;
- IndexedDB como implementação inicial sugerida;
- backend não obrigatório;
- design system local sincronizado com a identidade NEXO;
- possibilidade futura de empacotar com Tauri ou expor o motor por API.

## Consequências positivas

- menor complexidade inicial;
- testes matemáticos rápidos;
- funcionamento offline/local;
- desenvolvimento do motor sem dependência da UI;
- caminho de evolução para desktop/API;
- ausência de custo operacional de backend no pré-MVP.

## Consequências negativas

- sincronização entre dispositivos não existe no MVP;
- catálogo colaborativo exige infraestrutura futura;
- IndexedDB precisa de migrações cuidadosas;
- funções pesadas de otimização podem exigir worker ou backend depois.

## Alternativas consideradas

### Backend desde o início

Rejeitado no momento porque contas/sincronização/catálogo remoto não são necessárias para validar o produto central.

### Desktop-only

Rejeitado como primeira implementação por aumentar o custo de bootstrap. Tauri permanece opção futura.

### Fórmulas dentro das features React

Rejeitado por dificultar testes, reutilização e auditoria.

## Impacto

Este ADR fundamenta:

- `docs/ARCHITECTURE.md`;
- Etapas 1–7 de `docs/ROADMAP.md`;
- `docs/TEST_STRATEGY.md`;
- `docs/DEVELOPMENT.md`.

Qualquer adoção futura de backend não invalida necessariamente este ADR; o motor de cálculo deve continuar independente.
