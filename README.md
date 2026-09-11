# DroneCalc

DroneCalc é uma aplicação para **dimensionamento, comparação e validação preliminar de drones DIY**. O objetivo é permitir que o usuário monte virtualmente uma configuração, informe ou selecione componentes e obtenha cálculos de massa, energia, corrente, potência, empuxo, relação empuxo/peso, autonomia estimada, centro de gravidade, compatibilidade e link budget de vídeo FPV.

O produto também terá um modo orientado por **estilo de voo**, no qual o usuário informa o comportamento desejado — por exemplo Freestyle, Racing, Cinematic, Long Range, Mini Long Range ou Cinewhoop — e o sistema transforma essa intenção em requisitos técnicos e critérios de recomendação.

> O DroneCalc é uma ferramenta de apoio a projeto e aprendizado. Resultados calculados ou estimados não substituem testes de bancada, documentação do fabricante, inspeção da montagem ou validação de segurança antes do voo. Estimativas RF baseadas em FSPL representam condições idealizadas de espaço livre e não garantem alcance real.

## Princípios do produto

- **Engenharia antes da interface:** o motor de cálculo é independente da camada visual.
- **Transparência:** todo resultado informa origem (`measured`, `manufacturer`, `calculated`, `interpolated` ou `estimated`) e nível de confiança.
- **Unidades consistentes:** o núcleo trabalha com unidades canônicas e a interface converte para unidades familiares ao usuário.
- **Dados reais quando disponíveis:** curvas de bancada de motor/hélice/bateria têm prioridade sobre aproximações genéricas.
- **Compatibilidade explicável:** alertas informam o problema, a regra utilizada e a margem encontrada.
- **Projeto por objetivo:** perfis de voo orientam metas, mas nunca escondem parâmetros técnicos.
- **RF auditável:** distância FPV, FSPL, potência, sensibilidade, ganhos e perdas devem ser explicitados; vídeo e rádio-controle permanecem links separados.
- **Identidade NEXO:** a interface reutiliza a linguagem visual e os tokens do NEXO Design System.
- **IA como assistente, não autoridade:** informações extraídas pelo Ollama passam por schema, staging e revisão antes de publicação.

## Modos de criação

1. **Por estilo de voo** — o DroneCalc coleta requisitos e transforma o objetivo em metas técnicas.
2. **Montagem manual** — o usuário escolhe cada componente e recebe análise contínua.
3. **Duplicar projeto** — cria uma variante para comparar motores, hélices, baterias, VTX, antenas ou payloads.

## Perfis iniciais

- Freestyle
- Racing
- Cinematic
- Long Range
- Mini Long Range
- Cinewhoop

Perfis futuros incluem Cruiser, Payload, Fotogrametria/Mapeamento, FPV Iniciante e Experimental DIY. Restrições como **sub-250 g** e **distância FPV desejada** são independentes do estilo e podem ser combinadas com perfis apropriados.

## Escopo do MVP ampliado

Além de responder:

1. Quanto o drone pesa?
2. Os componentes são eletricamente compatíveis?
3. Há empuxo suficiente para o objetivo escolhido?
4. Qual é a autonomia aproximada nas condições informadas?
5. Qual conjunto de propulsão é mais adequado ao projeto e uso?
6. O link de vídeo FPV fecha, em espaço livre, na distância desejada com a margem configurada?

O produto passa a prever catálogo persistente, cadastro assistido por URL, extração de imagens/dados, evidências, IA local via Ollama, heurísticas, geração de candidatos e cálculo RF auditável.

## Arquitetura vigente

Base tecnológica planejada:

- React + TypeScript + Vite no frontend;
- Node.js + TypeScript na API;
- Zod para validação;
- Zustand para estado de UI quando necessário;
- motor matemático em TypeScript puro e independente de React, banco e IA;
- PostgreSQL como fonte principal dos dados persistidos;
- MinIO/S3-compatible para imagens e documentos;
- Ollama atrás de `AiProvider` ou abstração equivalente;
- IndexedDB somente para cache, drafts, preferências e eventual suporte offline.

A mudança da estratégia local-only para backend/PostgreSQL está registrada no [`ADR 0002`](docs/adr/0002-postgresql-object-storage-ollama.md).

## Cadastro assistido por URL

Fluxo planejado:

```text
URL
→ fetch seguro
→ extração estruturada
→ Ollama opcional
→ validação de schema
→ staging
→ revisão humana
→ catálogo
```

Imagens ficam no object storage; o PostgreSQL guarda seus metadados e referências.

## Alcance FPV

O operador poderá informar a distância desejada e usar VTX, VRX/óculos e antenas cadastrados ou valores manuais. O sistema calculará FSPL, potência recebida, margem de enlace, potência teórica mínima de VTX, EIRP quando disponível e distância teórica máxima em espaço livre.

Regras importantes:

- `dBm`, `dB` e `dBi` têm semânticas distintas;
- `directivity`, `gain` e `realized gain` não são tratados como equivalentes;
- perdas de SWR/eficiência não podem ser contadas duas vezes;
- diversity não soma automaticamente ganhos de antenas;
- vídeo FPV não implica alcance equivalente do rádio-controle;
- resultado de espaço livre não é garantia de alcance real nem confirmação de legalidade da potência utilizada.

## Documentação

O índice completo está em [`docs/README.md`](docs/README.md). Documentos principais:

- [`PRD.md`](docs/PRD.md)
- [`PRODUCT_SPEC.md`](docs/PRODUCT_SPEC.md)
- [`ARCHITECTURE.md`](docs/ARCHITECTURE.md)
- [`DATA_MODEL.md`](docs/DATA_MODEL.md)
- [`DOMAIN_MODEL.md`](docs/DOMAIN_MODEL.md)
- [`CALCULATION_ENGINE.md`](docs/CALCULATION_ENGINE.md)
- [`PHYSICS_ENGINE.md`](docs/PHYSICS_ENGINE.md)
- [`BENCH_DATA_WORKSPACE.md`](docs/BENCH_DATA_WORKSPACE.md)
- [`PROPULSION_RECOMMENDATION.md`](docs/PROPULSION_RECOMMENDATION.md)
- [`FPV_LINK_BUDGET.md`](docs/FPV_LINK_BUDGET.md)
- [`FLIGHT_PROFILES.md`](docs/FLIGHT_PROFILES.md)
- [`COMPONENT_CATALOG.md`](docs/COMPONENT_CATALOG.md)
- [`NEXO_DESIGN_SYSTEM.md`](docs/NEXO_DESIGN_SYSTEM.md)
- [`TEST_STRATEGY.md`](docs/TEST_STRATEGY.md)
- [`ROADMAP.md`](docs/ROADMAP.md)
- [`AI_IMPLEMENTATION_GUIDE.md`](docs/AI_IMPLEMENTATION_GUIDE.md)
- [`docs/adr/`](docs/adr/README.md)

## Próxima etapa

**Etapa 1A — Bootstrap full-stack e workspace.**

A instrução pronta para outra IA está em:

[`docs/prompts/ETAPA_1A_BOOTSTRAP_FULLSTACK.md`](docs/prompts/ETAPA_1A_BOOTSTRAP_FULLSTACK.md)

A Etapa 1A cria Web + API + packages puros, mas **não implementa ainda PostgreSQL, MinIO, Ollama ou o link budget RF**. Esses recursos entram nas fases específicas do roadmap, evitando sobrecarregar o primeiro bootstrap.

## Status

**Fase atual:** documentação/arquitetura revisadas; pronta para início do bootstrap após revisão do PR correspondente.

## Repositório

`eduardobelopro-br/Dronecalc`

## Licença

Diretriz definida: uso livre para fins não comerciais; uso comercial deverá depender de autorização/licenciamento específico. O texto jurídico final da licença deve permanecer explícito no repositório antes da primeira distribuição pública relevante.
