# Changelog

Todas as mudanças relevantes do DroneCalc serão registradas aqui.

O formato segue a ideia de **Keep a Changelog**, sem assumir versionamento de release antes da primeira versão executável.

## [Unreleased]

### Added

- documentação-base do produto;
- PRD e especificação funcional;
- arquitetura web-first/full-stack com motor matemático independente;
- PostgreSQL como persistência principal planejada, MinIO/S3-compatible para assets e Ollama como provider local de IA atrás de abstração;
- modelo de domínio e persistência;
- especificação do motor de cálculo;
- Physics Engine para bateria sob carga, motor, torque, hélice e propulsão;
- perfis de voo, incluindo Mini Long Range;
- adoção da identidade visual NEXO;
- UX spec;
- catálogo de componentes;
- ingestão assistida por URL/imagens/documentos com staging e revisão;
- scraping controlado de páginas oficiais de fabricantes;
- seção **Bancada** para curvas motor+hélíce, importação, validação, proveniência e visualização;
- regras explícitas para potência `V × I`, eficiência estática `gf/W`, RPM ideal sem carga e velocidade teórica de passo;
- proibição de apresentar `pitch × RPM` como velocidade real/máxima do drone;
- estratégia de testes ampliada para dados de bancada, ingestão e segurança;
- roadmap v0.6;
- guia de desenvolvimento e contribuição;
- contrato para implementação assistida por IA;
- prompt futuro para implementação do Workspace Bancada;
- ADRs e matriz de rastreabilidade.

## Política futura

Quando a primeira versão executável for definida, adotar versionamento semântico quando fizer sentido para o produto e registrar alterações que afetem:

- resultados matemáticos;
- schemas de projeto/export;
- dados/curvas de bancada;
- compatibilidade;
- perfis/scoring;
- experiência do usuário;
- APIs públicas futuras.