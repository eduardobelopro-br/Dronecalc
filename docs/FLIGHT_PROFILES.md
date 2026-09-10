# Perfis de Voo — DroneCalc

**Versão:** 0.1

## 1. Objetivo

Perfis de voo transformam intenção do usuário em critérios técnicos de avaliação. Eles não são presets rígidos de componentes e não substituem análise física.

Um perfil deve responder: **quais características são mais importantes para este tipo de drone?**

## 2. Princípios

- perfis são dados versionados;
- restrições do usuário têm prioridade sobre preferências do perfil;
- scores devem ser explicáveis por dimensão;
- nenhum perfil pode ocultar uma incompatibilidade `danger`;
- faixas de hélice, TWR ou autonomia são preferências, não verdades universais;
- “sub-250 g” é restrição independente;
- perfil principal pode futuramente receber características secundárias/híbridas.

## 3. Estrutura sugerida

```ts
interface FlightStyleProfile {
  schemaVersion: number
  id: string
  slug: string
  name: string
  description: string
  priorities: {
    endurance: number
    efficiency: number
    agility: number
    payload: number
    stability: number
    compactness: number
    lowWeight: number
    powerReserve: number
    durability: number
  }
  targets: ProfileTargets
  scoring: ScoringRule[]
}
```

Pesos podem usar escala 0–10 na configuração e ser normalizados em runtime.

## 4. Perfis do MVP

### 4.1 Freestyle

Objetivo: manobras, resposta previsível, robustez e reserva de potência.

Prioridades indicativas:

| Dimensão | Peso inicial |
|---|---:|
| Agilidade | 10 |
| Reserva de potência | 9 |
| Durabilidade | 9 |
| Estabilidade | 6 |
| Eficiência | 5 |
| Baixo peso | 6 |
| Autonomia | 4 |
| Payload | 2 |
| Compactação | 6 |

O perfil deve evitar premiar potência se isso gerar incompatibilidade elétrica ou massa desnecessária.

### 4.2 Racing

Objetivo: aceleração, resposta, baixo peso e alta relação empuxo/peso.

| Dimensão | Peso inicial |
|---|---:|
| Agilidade | 10 |
| Baixo peso | 10 |
| Reserva de potência | 10 |
| Eficiência | 4 |
| Estabilidade | 5 |
| Durabilidade | 6 |
| Autonomia | 2 |
| Payload | 1 |
| Compactação | 8 |

### 4.3 Cinematic

Objetivo: voo suave, estabilidade, margem para câmera/payload e operação sem levar componentes continuamente ao limite.

| Dimensão | Peso inicial |
|---|---:|
| Estabilidade | 10 |
| Eficiência | 8 |
| Autonomia | 7 |
| Payload | 7 |
| Reserva de potência | 6 |
| Durabilidade | 6 |
| Agilidade | 4 |
| Baixo peso | 5 |
| Compactação | 5 |

### 4.4 Long Range

Objetivo: autonomia, eficiência e estabilidade ao longo do voo.

| Dimensão | Peso inicial |
|---|---:|
| Autonomia | 10 |
| Eficiência | 10 |
| Estabilidade | 9 |
| Reserva de potência | 5 |
| Baixo peso | 6 |
| Payload | 4 |
| Durabilidade | 5 |
| Agilidade | 3 |
| Compactação | 3 |

Não confundir “long range” com simplesmente instalar bateria de maior capacidade; massa adicional pode reduzir eficiência e alterar o ponto de hover.

### 4.5 Mini Long Range

Mini Long Range é perfil próprio.

Objetivo: obter elevada autonomia e eficiência em plataforma compacta e leve, frequentemente com hélices menores que long-range tradicional.

| Dimensão | Peso inicial |
|---|---:|
| Autonomia | 10 |
| Eficiência | 10 |
| Baixo peso | 10 |
| Compactação | 9 |
| Estabilidade | 8 |
| Reserva de potência | 5 |
| Durabilidade | 4 |
| Agilidade | 4 |
| Payload | 3 |

Regras específicas:

- eficiência na região de hover/cruzeiro recebe peso maior que empuxo máximo;
- aumento de capacidade da bateria deve ser avaliado junto com aumento de massa;
- `sub250g` pode ser ativado, mas não define o perfil;
- faixa preferida de hélice pode começar em aproximadamente 3,5–5 polegadas como heurística configurável, nunca como bloqueio universal;
- GPS, receptor e sistemas de vídeo entram no orçamento de massa e energia normalmente;
- configurações que dependam de margem elétrica mínima devem ser penalizadas mesmo se a autonomia teórica parecer boa.

### 4.6 Cinewhoop

Objetivo: estabilidade, voo controlado, proteção de hélices e operação com payload de câmera.

| Dimensão | Peso inicial |
|---|---:|
| Estabilidade | 10 |
| Durabilidade/proteção | 9 |
| Payload | 7 |
| Compactação | 8 |
| Eficiência | 6 |
| Reserva de potência | 7 |
| Agilidade | 5 |
| Baixo peso | 6 |
| Autonomia | 4 |

Dutos/protetores devem ser modelados como componentes e não como “peso invisível”.

## 5. Restrições independentes

Perfis e restrições são ortogonais.

Exemplos:

- `mini-long-range + sub250g`;
- `cinematic + sub250g`;
- `freestyle + maxPropDiameter=5`;
- `long-range + payloadTarget=400g`.

Restrições iniciais:

```ts
interface ProjectConstraints {
  maxTakeoffMassG?: number
  sub250g?: boolean
  minEnduranceMinutes?: number
  maxPropDiameterIn?: number
  maxSeriesCells?: number
  payloadTargetG?: number
  preferredBatteryChemistry?: BatteryChemistry[]
}
```

## 6. Preferências do usuário

Além do perfil, o assistente poderá perguntar:

- prioridade: autonomia, equilíbrio, velocidade/agilidade;
- tamanho máximo;
- payload;
- autonomia desejada;
- bateria preferida;
- sistema FPV;
- nível de experiência.

Esses dados ajustam pesos ou adicionam restrições, sem alterar a definição global do perfil.

## 7. Perfil híbrido futuro

Permitir combinar um perfil principal com sliders de intenção:

```text
Autonomia      90
Agilidade      40
Payload        60
Estabilidade   80
Compactação    75
```

O resultado deve ser salvo no projeto como uma cópia normalizada dos objetivos, de forma que uma futura mudança no preset global não altere silenciosamente um projeto antigo.

## 8. Score

Score recomendado: 0–100, calculado por dimensões normalizadas.

Modelo conceitual:

```text
score = Σ(weight_i × metric_score_i) / Σ(weight_i) - penalties
```

Nunca calcular diretamente sobre métricas de unidades incompatíveis sem normalização.

### Penalidades obrigatórias

- incompatibilidade `danger` pode invalidar o score global ou limitar o máximo;
- `warning` pode aplicar penalidade configurada;
- falta de dado reduz confiança e deve aparecer separadamente do score.

## 9. Explicabilidade

Ao mostrar “87/100 para Mini Long Range”, a UI deve detalhar algo como:

```text
Autonomia       94
Eficiência      91
Baixo peso      96
Compactação     92
Estabilidade    82
Reserva         64

Penalidades
- margem de bateria baixa: -4
```

## 10. Dados ausentes

Score incompleto não deve parecer completo.

Exemplo:

```text
Adequação preliminar: 81/100
Confiança: média
2 dimensões ainda sem dados: eficiência e autonomia
```

Alternativamente, suspender score global até existir cobertura mínima definida.

## 11. Configuração e versionamento

Perfis devem possuir:

- `schemaVersion`;
- `profileVersion`;
- data de alteração opcional;
- changelog quando thresholds/weights mudarem materialmente.

Projetos devem armazenar qual versão do perfil foi usada na análise.

## 12. Perfis futuros

- Cruiser;
- Payload;
- FPV Iniciante;
- Fotogrametria/Mapeamento;
- Experimental DIY;
- Indoor;
- Endurance otimizado;
- plataforma pesada/industrial, somente quando o domínio estiver preparado.
