# Catálogo de Componentes — DroneCalc

**Versão:** 0.2

## 1. Objetivo

Definir categorias, campos mínimos, metadados de fonte e regras de qualidade do catálogo.

O catálogo não deve apenas armazenar nomes de peças. Ele deve fornecer os dados necessários para cálculos e deixar claro quando um dado é desconhecido.

O cadastro assistido por URL, imagem e documento segue `ASSISTED_INGESTION.md`.

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

Campos importados devem preservar contexto, unidade, fonte e condição quando a condição fizer parte do significado do dado.

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

Campos mínimos/recomendados conforme disponibilidade:

- massa e condição da medição/declarada quando relevante;
- KV;
- configuração de estator/ímãs quando declarada, por exemplo `12N14P`;
- diâmetro e comprimento do estator;
- faixa de células/tensão e química associada quando declarada;
- corrente máxima contínua se declarada, preservando condições;
- burst se declarado, com duração/condição quando conhecida;
- potência máxima/contínua se declarada, preservando condições;
- corrente sem carga e tensão do ensaio;
- resistência do enrolamento/interna do motor e temperatura/condição quando conhecida;
- `Kt` quando fornecido ou calculável, distinguindo explicitamente valor declarado de valor derivado;
- shaft: diâmetro/tipo/oco quando aplicável;
- dimensões externas;
- padrão de montagem;
- faixa/região de eficiência declarada, com corrente/tensão/condições;
- links para testes de bancada;
- evidências técnicas associadas.

Não armazenar “empuxo máximo do motor” sem contexto de hélice/tensão. Empuxo pertence à combinação testada/modelada.

Um limite como `35 A @ 5S` não deve virar um `maxCurrentA = 35` universal sem a condição. O modelo deve permitir limites condicionais/versionados.

## 5. Hélice

- diâmetro;
- pitch;
- número de pás;
- massa;
- material;
- furo/hub;
- direção;
- RPM máximo quando declarado;
- `Ct`, `Cq`, `Cp` somente quando houver fonte/convenção/condições adequadas;
- curvas por advance ratio quando disponíveis;
- notas.

Coeficientes aerodinâmicos não podem ser inventados a partir apenas de diâmetro/pitch/número de pás.

## 6. ESC

- layout individual/4-in-1;
- massa;
- corrente contínua por canal;
- burst e duração quando declarada;
- faixa de células/tensão;
- quantidade de saídas;
- protocolos;
- BEC integrado, se houver;
- eficiência quando conhecida;
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
- resistência interna quando medida, com condição/temperatura/SOC quando conhecida;
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

Power pode ser calculado se não fornecido, mas deve ser marcado como derivado.

Curvas digitalizadas de gráfico em imagem devem indicar explicitamente essa origem e não receber automaticamente o mesmo nível de confiança de um CSV/tabela de ensaio original.

## 19. Qualidade de dados

Sugestão de níveis:

### Completo
Dados suficientes para os cálculos principais da categoria.

### Parcial
Alguns cálculos podem ser feitos, mas faltam limites/atributos.

### Básico
Apenas identificação/massa ou poucos campos.

A UI deve indicar completude sem confundir com confiabilidade da fonte.

## 20. Fonte e evidência

Metadados de fonte de alto nível:

```ts
interface SourceMetadata {
  kind: 'measured' | 'manufacturer' | 'user-provided' | 'article' | 'video' | 'other'
  sourceName?: string
  sourceUrl?: string
  reference?: string
  measuredAt?: string
  notes?: string[]
}
```

“Manufacturer” não significa necessariamente “medido pelo DroneCalc”.

Para campos extraídos automaticamente, a evidência deve poder indicar adicionalmente HTML/tabela/imagem/documento, asset, método (`deterministic`, `ai-text`, `ai-vision`, `user`), confiança da extração e estado de revisão. Ver `ASSISTED_INGESTION.md`.

## 21. Revisões

Se o fabricante altera especificações ou há variantes com mesmo nome, preservar revisão/identificador específico. Não substituir histórico silenciosamente.

Reprocessar uma página/imagem com modelo de IA diferente não sobrescreve silenciosamente uma revisão publicada.

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

## 24. Importação de fontes externas

Qualquer importer deve:

- mapear campos explicitamente;
- converter unidades de forma determinística;
- preservar valor bruto e fonte;
- manter evidência por campo quando disponível;
- identificar duplicatas;
- detectar conflitos entre HTML, imagem e documento;
- não inventar campos;
- não transformar cálculo derivado em especificação declarada;
- registrar revisão/import date;
- passar por validação de schema;
- gravar primeiro em staging;
- exigir o workflow de revisão/publicação definido em `ASSISTED_INGESTION.md`.

## 25. Dados extraídos versus derivados

Exemplo:

```text
KV = 1860
kind = extracted
source = manufacturer/image
```

pode alimentar posteriormente:

```text
Kt ≈ função(KV)
kind = calculated
formulaId = MOTOR_KV_TO_KT_V1
```

O segundo valor pertence ao calculation/physics engine e não deve ser apresentado como se estivesse escrito na ficha do fabricante.

## 26. Critérios de aceite

- componentes têm IDs estáveis;
- nenhum empuxo sem contexto motor+hélíce+tensão/modelo;
- unidade é conhecida;
- fonte é armazenável;
- evidência de campo é preservável para imports automáticos;
- desconhecido é distinto de zero;
- custom e catálogo são distinguíveis;
- testes de bancada suportam múltiplas fontes;
- limites condicionais preservam suas condições;
- dados extraídos por IA não entram diretamente no catálogo publicado;
- dados inválidos não entram no domínio;
- valores derivados são semanticamente distintos de valores declarados/extraídos.
