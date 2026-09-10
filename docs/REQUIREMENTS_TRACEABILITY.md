# Matriz de Rastreabilidade de Requisitos — DroneCalc

**Versão:** 0.2

Este documento liga requisitos do PRD às especificações, etapas do roadmap e tipos de teste esperados.

| Req. | Resumo | Especificação principal | Roadmap | Teste esperado |
|---|---|---|---|---|
| RF-01 | Projetos | PRODUCT_SPEC §2 | Fase 3 | integração/UI |
| RF-02 | Componentes | COMPONENT_CATALOG | Fases 3, 5 e 6 | schema/UI/ingestão |
| RF-03 | Massa | CALCULATION_ENGINE §3 | Fase 4 | unitário + fixture |
| RF-04 | Energia | CALCULATION_ENGINE §5 | Fase 4 | unitário |
| RF-05 | Sistema elétrico | CALCULATION_ENGINE §16 | Fase 4 | unitário + limites |
| RF-06 | Propulsão | CALCULATION_ENGINE §§7–9 + PHYSICS_ENGINE | Fase 7 | unitário + fixture de curva/modelo |
| RF-07 | Autonomia | CALCULATION_ENGINE §12 | Fase 8 | unitário + regressão |
| RF-08 | Perfis de voo | FLIGHT_PROFILES | Fase 10 | unitário/scoring |
| RF-09 | Compatibilidade | PRODUCT_SPEC §7 | Fases 4–10 | unitário + integração |
| RF-10 | Proveniência | PRODUCT_SPEC §6 + ASSISTED_INGESTION | Todas as etapas técnicas | unitário + UI |
| RF-11 | Comparação | PRODUCT_SPEC §8 | Fase 12 | integração/UI |
| RF-12 | Unidades | CALCULATION_ENGINE §2 | Etapa 1D | unitário |
| RF-13 | Persistência | DATA_MODEL | Fase 2 | integração/migrations |
| RF-14 | Import/export | DATA_MODEL | Etapas 3D e 7B | integração |
| RF-15 | Cadastro assistido multimodal | ASSISTED_INGESTION + PRODUCT_SPEC §11 | Fases 5 e 6 | segurança + schema + integração/UI |

## Requisitos não funcionais

| Req. | Resumo | Evidência esperada |
|---|---|---|
| RNF-01 | Determinismo | testes do calculation engine |
| RNF-02 | Testabilidade | domínio/motor sem React e suite unitária |
| RNF-03 | Rastreabilidade | formulaId/modelVersion/proveniência/evidência por campo |
| RNF-04 | Performance | medições quando UI/análise/ingestão estiverem funcionais |
| RNF-05 | Acessibilidade | UX spec + testes/inspeção |
| RNF-06 | Internacionalização | regras sem dependência de strings pt-BR |
| RNF-07 | Evolução | perfis/data-driven, módulos desacoplados e AI Provider substituível |
| RNF-08 | Segurança de dados/ingestão | secrets ausentes, anti-SSRF, schema validation, staging e prompt-injection boundaries |

## Cenários de aceite de produto

### AC-01 — Projeto por estilo

**Dado** usuário criando novo projeto  
**Quando** escolhe Mini Long Range e ativa sub-250 g  
**Então** ambos ficam registrados separadamente: perfil e constraint.

Rastreia: RF-01, RF-08.

### AC-02 — Massa incompleta

**Dado** componente sem massa  
**Quando** análise é executada  
**Então** o sistema não soma 0 silenciosamente e informa dado ausente.

Rastreia: RF-03, RF-10, RNF-03.

### AC-03 — Tensão de bateria

**Dado** bateria e ESC  
**Quando** tensão cheia excede limite do ESC  
**Então** emitir `danger` com valores e regra.

Rastreia: RF-04, RF-05, RF-09.

### AC-04 — Curva de bancada

**Dado** motor, hélice e curva válida  
**Quando** hover exige ponto entre duas amostras  
**Então** interpolar dentro da curva e marcar origem `interpolated`.

Rastreia: RF-06, RF-10.

### AC-05 — Sem extrapolação

**Dado** hover requerido fora da faixa testada  
**Quando** análise de propulsão é executada  
**Então** não extrapolar por padrão e informar indisponibilidade/limitação.

Rastreia: RF-06, RF-10.

### AC-06 — Autonomia

**Dado** bateria e corrente média calculável  
**Quando** autonomia é estimada  
**Então** mostrar valor, hipótese de capacidade utilizável, origem e confiança.

Rastreia: RF-07, RF-10.

### AC-07 — C-rating

**Dado** bateria com C-rating e sem corrente medida  
**Quando** limite de corrente é derivado  
**Então** indicar que o limite vem do rating declarado/teórico, não de medição.

Rastreia: RF-04, RF-05, RF-10.

### AC-08 — Mini Long Range

**Dado** duas configurações, uma com maior empuxo máximo e outra mais eficiente no regime relevante  
**Quando** avaliadas para Mini Long Range  
**Então** o score deve refletir maior peso de eficiência/autonomia e explicar o resultado.

Rastreia: RF-08.

### AC-09 — Danger e score

**Dado** configuração com score teórico alto e incompatibilidade `danger`  
**Quando** ranking é exibido  
**Então** o score não pode mascarar a incompatibilidade.

Rastreia: RF-08, RF-09.

### AC-10 — Persistência

**Dado** projeto salvo  
**Quando** aplicação é reaberta  
**Então** projeto e referências necessárias são restaurados sem perda a partir da persistência autoritativa/cache conforme disponibilidade.

Rastreia: RF-01, RF-13.

### AC-11 — Import inválido

**Dado** arquivo JSON/CSV inválido  
**Quando** importado  
**Então** dados existentes permanecem intactos e o erro é explicado.

Rastreia: RF-14, RNF-08.

### AC-12 — Tema NEXO

**Dado** usuário alternando Claro/Escuro/Sistema  
**Quando** tema muda  
**Então** componentes mantêm tokens NEXO, contraste e semântica de status.

Rastreia: RNF-05 e NEXO_DESIGN_SYSTEM.

### AC-13 — Cadastro por URL com imagem técnica

**Dado** uma página de equipamento contendo ficha técnica em imagem  
**Quando** o import é executado com provider visual compatível  
**Então** campos reconhecidos são gravados apenas em staging, validados por schema e ligados ao asset/evidência de origem.

Rastreia: RF-02, RF-10, RF-15, RNF-03, RNF-08.

### AC-14 — Modelo sem visão

**Dado** um import que exige análise de imagem e um modelo sem capability visual  
**Quando** a etapa multimodal é solicitada  
**Então** o sistema informa indisponibilidade/falha explícita e não registra dados inventados.

Rastreia: RF-15, RNF-07, RNF-08.

### AC-15 — Conflito HTML × imagem

**Dado** HTML declarando `60 mΩ` e imagem declarando `64 mΩ` para o mesmo campo/condição  
**Quando** a reconciliação é executada  
**Então** o sistema preserva ambos, normaliza unidades e cria conflito `needs_review` sem escolher silenciosamente um valor.

Rastreia: RF-10, RF-15, RNF-03.

### AC-16 — Extraído versus derivado

**Dado** `KV = 1860` extraído de uma ficha  
**Quando** o Physics Engine calcula `Kt`  
**Então** KV permanece evidência de fonte e Kt recebe proveniência de cálculo/formulaId; o segundo não é apresentado como declaração do fabricante.

Rastreia: RF-06, RF-10, RF-15, RNF-03.

### AC-17 — Proteção anti-SSRF

**Dado** URL pública que redireciona para loopback/rede privada  
**Quando** o fetcher processa redirects  
**Então** o destino é bloqueado e nenhum conteúdo interno é entregue ao extractor/IA.

Rastreia: RF-15, RNF-08.

## Atualização da matriz

Ao adicionar requisito no PRD:

1. adicionar linha aqui;
2. apontar spec responsável;
3. apontar etapa ou issue;
4. definir evidência/teste;
5. não marcar como concluído apenas porque existe UI; regras técnicas exigem teste do motor/pipeline correspondente.
