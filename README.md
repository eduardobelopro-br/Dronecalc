# DroneCalc

DroneCalc é uma aplicação para **dimensionamento, comparação e validação preliminar de drones DIY**. O objetivo é permitir que o usuário monte virtualmente uma configuração, informe ou selecione componentes e obtenha cálculos de massa, energia, corrente, potência, empuxo, relação empuxo/peso, autonomia estimada, centro de gravidade e compatibilidade.

O produto também terá um modo orientado por **estilo de voo**, no qual o usuário informa o comportamento desejado — por exemplo Freestyle, Racing, Cinematic, Long Range, Mini Long Range ou Cinewhoop — e o sistema transforma essa intenção em requisitos técnicos e critérios de recomendação.

> O DroneCalc é uma ferramenta de apoio a projeto e aprendizado. Resultados calculados ou estimados não substituem testes de bancada, documentação do fabricante, inspeção da montagem ou validação de segurança antes do voo.

## Princípios do produto

- **Engenharia antes da interface:** o motor de cálculo é independente da camada visual.
- **Transparência:** todo resultado informa origem (`measured`, `manufacturer`, `calculated`, `interpolated` ou `estimated`) e nível de confiança.
- **Unidades consistentes:** o núcleo trabalha com unidades canônicas e a interface converte para unidades familiares ao usuário.
- **Dados reais quando disponíveis:** curvas de bancada de motor/hélice/bateria têm prioridade sobre aproximações genéricas.
- **Compatibilidade explicável:** alertas informam o problema, a regra utilizada e a margem encontrada.
- **Projeto por objetivo:** perfis de voo orientam metas, mas nunca escondem os parâmetros técnicos.
- **Identidade NEXO:** a interface deve reutilizar a linguagem visual e os tokens do NEXO Design System.

## Modos de criação

1. **Por estilo de voo** — o DroneCalc coleta requisitos e sugere configurações adequadas.
2. **Montagem manual** — o usuário escolhe cada componente e recebe análise contínua.
3. **Duplicar projeto** — cria uma variante para comparar motores, hélices, baterias ou payloads.

## Perfis iniciais

- Freestyle
- Racing
- Cinematic
- Long Range
- Mini Long Range
- Cinewhoop

Perfis futuros incluem Cruiser, Payload, Fotogrametria/Mapeamento, FPV Iniciante e Experimental DIY. Restrições como **sub-250 g** são independentes do estilo e podem ser combinadas com qualquer perfil.

## Escopo do MVP

O primeiro MVP deve responder com qualidade a quatro perguntas:

1. Quanto o drone pesa?
2. Os componentes são eletricamente compatíveis?
3. Há empuxo suficiente para o objetivo escolhido?
4. Qual é a autonomia aproximada nas condições informadas?

Também deverá permitir salvar projetos, cadastrar componentes personalizados, comparar variantes e mostrar a procedência de cada resultado.

## Arquitetura proposta

Base tecnológica inicial:

- React
- TypeScript
- Vite
- Zod para validação de dados
- Zustand para estado de interface/projeto
- motor matemático em TypeScript puro, sem dependência de React
- persistência local inicialmente; banco/API podem ser adicionados quando houver necessidade real

A arquitetura é intencionalmente modular para permitir evolução futura para desktop com Tauri, API, catálogo sincronizado ou otimizador de configurações.

## Documentação

A documentação técnica e de produto está em [`docs/`](docs/README.md):

- [`PRD.md`](docs/PRD.md) — Product Requirements Document
- [`PRODUCT_SPEC.md`](docs/PRODUCT_SPEC.md) — especificação funcional
- [`ARCHITECTURE.md`](docs/ARCHITECTURE.md) — arquitetura e limites entre camadas
- [`DOMAIN_MODEL.md`](docs/DOMAIN_MODEL.md) — entidades e tipos do domínio
- [`CALCULATION_ENGINE.md`](docs/CALCULATION_ENGINE.md) — fórmulas, hipóteses e níveis de confiança
- [`FLIGHT_PROFILES.md`](docs/FLIGHT_PROFILES.md) — perfis e critérios de otimização
- [`NEXO_DESIGN_SYSTEM.md`](docs/NEXO_DESIGN_SYSTEM.md) — identidade visual e regras de UI
- [`UX_SPEC.md`](docs/UX_SPEC.md) — fluxos e experiência do usuário
- [`DATA_MODEL.md`](docs/DATA_MODEL.md) — persistência e versionamento dos dados
- [`COMPONENT_CATALOG.md`](docs/COMPONENT_CATALOG.md) — categorias e campos de componentes
- [`TEST_STRATEGY.md`](docs/TEST_STRATEGY.md) — estratégia de testes
- [`SAFETY_AND_LIMITATIONS.md`](docs/SAFETY_AND_LIMITATIONS.md) — limites e comunicação de incerteza
- [`ROADMAP.md`](docs/ROADMAP.md) — fases de implementação
- [`DEVELOPMENT.md`](docs/DEVELOPMENT.md) — padrões de desenvolvimento
- [`AI_IMPLEMENTATION_GUIDE.md`](docs/AI_IMPLEMENTATION_GUIDE.md) — instruções para agentes de IA

## Status

**Fase atual:** especificação e fundação do projeto.

A implementação deve seguir o roadmap e manter PRD, specs e decisões arquiteturais atualizados conforme o produto evoluir.

## Repositório

`eduardobelopro-br/Dronecalc`

## Licença

A licença ainda não foi definida. Não assumir uma licença de uso até que um arquivo `LICENSE` seja adicionado explicitamente ao repositório.
