# Catálogo de Componentes — DroneCalc

**Versão:** 0.1

## 1. Objetivo

Definir categorias, campos mínimos, metadados de fonte e regras de qualidade do catálogo.

O catálogo não deve apenas armazenar nomes de peças. Ele deve fornecer os dados necessários para cálculos e deixar claro quando um dado é desconhecido.

## 2. Regras gerais

Todo componente possui:

- ID estável;
- tipo;
- fabricante opcional;
- modelo/nome;
- massa opcional;
- origem;
- flag `custom`;
- revisão/data;
- notas/tags opcionais.

Campos desconhecidos devem permanecer ausentes. Não usar zero como placeholder.

## 3. Frame

Campos úteis:

- massa;
- wheelbase;
- número de motores suportado;
- diâmetro máximo de hélice;
- padrões de montagem de motor;
- padrões de stack;
- dimensões úteis;
- material;
- suporte de bateria;
- observações.

## 4. Motor

Campos mínimos recomendados:

- massa;
- KV;
- faixa de células/tensão;
- corrente máxima contínua se declarada;
- burst se declarado;
- potência máxima se declarada;
- stator;
- shaft;
- padrão de montagem;
- links para testes de bancada.

Não armazenar “empuxo máximo do motor” sem contexto de hélice/tensão. Empuxo pertence à combinação testada.

## 5. Hélice

- diâmetro;
- pitch;
- número de pás;
- massa;
- material;
- furo/hub;
- direção;
- RPM máximo quando declarado;
- notas.

## 6. ESC

- layout individual/4-in-1;
- massa;
- corrente contínua por canal;
- burst;
- faixa de células/tensão;
- quantidade de saídas;
- protocolos;
- BEC integrado, se houver;
- dados térmicos relevantes, se conhecidos.

## 7. Bateria

- química;
- células em série;
- paralelo quando relevante;
- capacidade mAh;
- tensão nominal por célula;
- tensão cheia por célula;
- tensão mínima recomendada por célula quando disponível;
- C-rating contínuo e burst, se declarado;
- corrente contínua medida/recomendada quando disponível;
- massa;
- conector;
- resistência interna quando medida;
- dimensões opcionais.

## 8. Flight Controller

- massa;
- faixa de alimentação;
- corrente/consumo aproximado quando conhecido;
- formato/padrão de montagem;
- BECs disponíveis;
- sensores relevantes;
- número de UARTs e recursos podem ser catalogados futuramente.

Compatibilidade de firmware não é foco do primeiro MVP.

## 9. GPS

- massa;
- alimentação;
- corrente/consumo;
- protocolos/interface;
- magnetômetro integrado opcional;
- dimensões.

## 10. Receiver

- massa;
- tensão;
- consumo;
- protocolo/ecossistema;
- antena integrada/externa;
- telemetria opcional.

## 11. VTX

- massa;
- faixa de tensão;
- consumo por modo/potência quando conhecido;
- potência RF declarada;
- sistema analógico/digital;
- antena/conector;
- observações térmicas.

## 12. Câmera FPV / gravação

- massa;
- alimentação/consumo se elétrica;
- dimensões;
- sistema;
- montagem;
- observações.

Qualidade de imagem não entra no motor físico do MVP, mas pode ser metadado para filtros futuros.

## 13. BEC / Power Module

- massa;
- tensão de entrada;
- tensão de saída;
- corrente contínua;
- pico;
- eficiência quando conhecida;
- número de saídas.

No futuro, o grafo elétrico deve usar esses dados.

## 14. Cabos e conectores

No MVP podem ser representados como componente genérico com massa. Futuramente:

- gauge/AWG;
- comprimento;
- corrente admissível;
- conector;
- resistência estimada.

## 15. Landing gear / proteções / dutos

- massa;
- compatibilidade de frame;
- dimensões.

Para Cinewhoop, protetores/dutos são parte relevante da massa e não podem ser ignorados.

## 16. Payload

- massa;
- alimentação se necessário;
- consumo;
- posição;
- descrição.

Payload pode ser genérico (“carga 300 g”) mesmo sem modelo de produto.

## 17. Outros

Categoria de escape permitida, mas deve conter no mínimo nome, quantidade/massa e observações. Se categorias “other” se tornarem frequentes para um mesmo tipo, criar categoria formal.

## 18. Testes motor + hélice

Um teste deve ser entidade separada e referenciar o motor.

Campos:

- motor;
- hélice estruturada ou descrição textual;
- tensão/células;
- fonte;
- condições de ensaio quando conhecidas;
- amostras.

Amostra:

```text
throttle % | thrust | current | voltage | power | rpm
```

Power pode ser calculado se não fornecido.

## 19. Qualidade de dados

Sugestão de níveis:

### Completo
Dados suficientes para os cálculos principais da categoria.

### Parcial
Alguns cálculos podem ser feitos, mas faltam limites/atributos.

### Básico
Apenas identificação/massa ou poucos campos.

A UI deve indicar completude sem confundir com confiabilidade da fonte.

## 20. Fonte

Metadados de fonte:

```ts
interface SourceMetadata {
  kind: 'measured' | 'manufacturer' | 'user-provided' | 'other'
  sourceName?: string
  sourceUrl?: string
  reference?: string
  measuredAt?: string
  notes?: string[]
}
```

“Manufacturer” não significa necessariamente “medido pelo DroneCalc”.

## 21. Revisões

Se o fabricante altera especificações ou há variantes com mesmo nome, preservar revisão/identificador específico. Não substituir histórico silenciosamente.

## 22. Componentes personalizados

Usuário pode criar qualquer peça. Requisitos:

- destacar `custom`;
- validar unidades e limites;
- permitir duplicar peça existente para editar;
- nunca alterar o registro de catálogo original ao aplicar override no projeto.

## 23. Seeds do MVP

O catálogo inicial pode começar pequeno. É preferível possuir poucos componentes bem estruturados/testados do que centenas de registros incompletos copiados sem validação.

Prioridade inicial para seeds:

- componentes fictícios de teste para automatização;
- alguns exemplos reais somente quando a fonte estiver registrada;
- curvas de bancada verificáveis para validar o motor.

## 24. Importação futura de fontes externas

Qualquer importer deve:

- mapear campos explicitamente;
- converter unidades;
- preservar fonte;
- identificar duplicatas;
- não inventar campos;
- registrar revisão/import date;
- passar por validação de schema.

## 25. Critérios de aceite

- componentes têm IDs estáveis;
- nenhum empuxo sem contexto motor+hélíce+tensão;
- unidade é conhecida;
- fonte é armazenável;
- desconhecido é distinto de zero;
- custom e catálogo são distinguíveis;
- testes de bancada suportam múltiplas fontes;
- dados inválidos não entram no domínio.
