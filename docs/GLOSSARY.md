# Glossário — DroneCalc

**Versão:** 0.1

Este glossário padroniza termos usados na UI, documentação e código.

## Ah
Ampere-hora. Unidade de capacidade elétrica usada no cálculo. Baterias são frequentemente anunciadas em mAh.

## Autonomia / Endurance
Tempo estimado ou medido de operação do drone. No DroneCalc, deve sempre indicar se é estimado ou medido.

## BEC
Battery Eliminator Circuit. Conversor/regulador usado para alimentar eletrônica em tensão adequada.

## Bench test / teste de bancada
Ensaio de uma combinação de motor, hélice, tensão e condições conhecidas, normalmente registrando empuxo, corrente, potência, RPM e throttle.

## C-rating
Multiplicador declarado de descarga de bateria. A corrente teórica pode ser calculada por `Ah × C`, mas o valor declarado não deve ser tratado automaticamente como desempenho real medido.

## CG / Centro de gravidade
Ponto resultante da distribuição de massa do conjunto.

## Cinewhoop
Multirrotor compacto orientado a voo controlado/cinemático, normalmente com proteção de hélices/dutos.

## Cinematic
Perfil que prioriza suavidade, estabilidade, margem de payload e qualidade operacional sobre agressividade máxima.

## Continuous current
Corrente que um componente declara suportar continuamente dentro de condições especificadas. Diferente de burst/pico.

## Disk loading
Carga por área total de disco dos rotores. Deve ser calculada com grandezas físicas coerentes.

## Dry mass
Massa seca conforme definição do projeto, normalmente sem bateria e payload destacável. A definição usada pelo DroneCalc deve ser consistente em toda a UI.

## ESC
Electronic Speed Controller. Controla a potência entregue ao motor.

## Estimated
Resultado produzido por modelo/simplificação e não por medição direta.

## Flight profile / perfil de voo
Conjunto de prioridades e critérios usados pelo DroneCalc para avaliar uma configuração conforme o objetivo do usuário.

## Freestyle
Perfil de voo orientado a manobras, resposta, robustez e reserva de potência.

## g/W
Gramas-força equivalentes por watt, usado como indicador prático de eficiência em um ponto de teste.

## Hover
Condição aproximada em que o empuxo total equilibra o peso do drone.

## KV
Constante de velocidade do motor, geralmente expressa em RPM por volt em condição teórica sem carga. KV sozinho não determina empuxo.

## Li-Ion
Química de bateria de íons de lítio. Valores de tensão devem vir do modelo da bateria, não de uma constante global rígida.

## LiHV
Bateria de lítio de alta tensão com tensão cheia por célula diferente de LiPo convencional.

## LiPo
Bateria de polímero de lítio amplamente usada em drones.

## Long Range
Perfil orientado principalmente a autonomia, eficiência e estabilidade para voos prolongados/distantes.

## Mini Long Range
Perfil próprio do DroneCalc para plataformas compactas e leves que buscam elevada eficiência/autonomia. Não é sinônimo obrigatório de sub-250 g ou 4 polegadas.

## mAh
Miliampere-hora. `1000 mAh = 1 Ah`.

## Manufacturer data
Dado declarado pelo fabricante. Não é automaticamente equivalente a medição independente.

## Mass / massa
Quantidade de matéria, normalmente apresentada em g/kg no DroneCalc. Não deve ser confundida conceitualmente com força/peso.

## MTOW / Maximum Takeoff Weight
Massa/peso máximo de decolagem conforme contexto. Quando usado no DroneCalc, deve ser distinguido entre limite definido pelo usuário/fabricante e massa atual calculada do projeto.

## Payload
Carga útil adicional transportada pelo drone.

## Power / potência
Taxa de uso/transferência de energia. Em circuito DC simples, `P = V × I`.

## Propeller pitch / passo
Geometria nominal da hélice relacionada ao avanço teórico por volta; não equivale diretamente à velocidade real do drone.

## Provenance / proveniência
Informação sobre a origem de um dado ou resultado.

## Racing
Perfil orientado a baixo peso, resposta, aceleração e reserva de potência.

## RPM
Rotações por minuto.

## Sag
Queda de tensão da bateria sob carga.

## Sub-250 g
Restrição de massa do projeto inferior a 250 g conforme meta escolhida. No DroneCalc é independente do perfil de voo e não implica automaticamente conformidade regulatória.

## Throttle
Comando relativo de potência/aceleração. Em curvas de bancada pode ser registrado em porcentagem, mas não é uma medida física universal de empuxo.

## Thrust / empuxo
Força produzida pelo conjunto propulsivo. Na UI pode aparecer em grama-força equivalente, mas o domínio deve manter distinção entre massa e força.

## TWR / Thrust-to-Weight Ratio
Relação entre empuxo total e peso. É adimensional.

## Usable battery fraction
Fração da capacidade nominal considerada utilizável pelo modelo de autonomia.

## VTX
Video Transmitter. Transmissor de vídeo do sistema FPV.

## Wh
Watt-hora. Unidade de energia. Energia nominal simplificada: `V_nominal × Ah`.

## Warning
Resultado de uma regra de compatibilidade/qualidade. Severidades do DroneCalc: `success`, `info`, `warning`, `danger`.
