# Segurança, Incerteza e Limitações — DroneCalc

**Versão:** 0.1

## 1. Objetivo

O DroneCalc auxilia dimensionamento preliminar. Ele não certifica aeronavegabilidade e não garante que uma montagem seja segura apenas porque os cálculos apresentados estão dentro de faixas esperadas.

A aplicação deve comunicar limites sem alarmismo e sem transformar estimativas em garantias.

## 2. Regra de comunicação

Todo resultado deve ser classificado quanto à origem e confiança.

A UI nunca deve usar linguagem como:

- “100% seguro”;
- “garantido”;
- “autonomia garantida”;
- “compatibilidade certificada”;

quando o sistema apenas fez cálculo ou comparação de especificações.

Preferir:

- “compatível segundo os dados informados”;
- “estimativa”;
- “máximo declarado pelo fabricante”;
- “dados insuficientes para verificar”;
- “recomendamos validar em bancada”.

## 3. Categorias de origem

### `measured`
Valor medido em teste identificável.

### `manufacturer`
Valor declarado em documentação do fabricante.

### `calculated`
Resultado determinístico obtido por fórmula a partir de entradas conhecidas.

### `interpolated`
Valor obtido entre pontos de uma curva existente.

### `estimated`
Resultado dependente de modelo, simplificação ou hipótese relevante.

## 4. Confiança

Confiança não é probabilidade estatística automática. É um indicador qualitativo da robustez da evidência/modelo.

- **Alta:** medição direta ou cálculo simples com entradas robustas;
- **Média:** interpolação ou combinação de dados razoáveis com hipóteses limitadas;
- **Baixa:** modelo aproximado, dados incompletos ou contexto pouco conhecido.

## 5. Limitações principais

### 5.1 C-rating

C-rating de bateria pode ser otimista e variar entre fabricantes. O valor derivado `Ah × C` deve ser identificado como teórico/declarado quando não houver teste independente.

### 5.2 Empuxo

Empuxo depende da combinação específica de:

- motor;
- hélice;
- tensão real sob carga;
- ESC;
- condições do teste;
- densidade do ar;
- montagem.

Não inferir empuxo confiável apenas por KV.

### 5.3 Autonomia

Autonomia é altamente sensível a:

- massa real;
- vento;
- velocidade;
- temperatura;
- estado da bateria;
- sag;
- resistência interna;
- comportamento do piloto;
- cutoff;
- aerodinâmica;
- eficiência do conjunto.

Sempre apresentar como estimativa, salvo se a aplicação estiver exibindo um tempo efetivamente medido e identificado como tal.

### 5.4 ESC e corrente

Corrente máxima declarada do ESC não substitui análise térmica real. Ventilação, duração da carga, layout 4-in-1, firmware e temperatura afetam operação.

### 5.5 Tensão

Verificação de tensão deve considerar bateria totalmente carregada quando comparada ao limite máximo do componente.

### 5.6 Centro de gravidade

CG depende de posição real e massa real dos componentes. Posições aproximadas produzem resultado aproximado.

### 5.7 Mecânica

Compatibilidade dimensional no catálogo não confirma resistência estrutural, qualidade de parafusos, torque, folga real da hélice, ressonância ou integridade do frame.

## 6. Dados insuficientes

Quando uma verificação não pode ser feita, a aplicação deve dizer isso explicitamente.

Exemplo correto:

```text
Não foi possível verificar a corrente do ESC.
Falta a corrente da combinação motor + hélice na tensão selecionada.
```

Exemplo incorreto:

```text
✓ ESC compatível
```

quando o dado necessário não existe.

## 7. Margens

Thresholds de warning são heurísticas configuráveis e não devem ser apresentados como normas universais.

O sistema deve exibir:

- valor real/estimado;
- limite conhecido;
- margem calculada;
- origem do limite;
- razão do alerta.

## 8. Validação física antes do voo

A documentação e UI podem recomendar boas práticas gerais:

- conferir documentação dos componentes;
- inspecionar polaridade e conexões;
- testar consumo/temperatura em bancada de forma apropriada;
- verificar fixação e folga das hélices;
- realizar testes progressivos;
- respeitar procedimentos seguros para baterias.

O DroneCalc não substitui esses passos.

## 9. Alterações no motor matemático

Qualquer melhoria que altere resultados deve:

- atualizar versão do modelo;
- ter teste de regressão;
- registrar mudança;
- evitar recalcular silenciosamente análises históricas sem informar versão quando comparações forem relevantes.

## 10. Dados importados

Dados externos podem conter erro. Importadores devem preservar fonte e permitir revisão. A aplicação não deve “promover” automaticamente um dado importado a medição confiável.

## 11. Recomendações automáticas futuras

O otimizador não pode recomendar uma configuração que viole uma incompatibilidade conhecida apenas porque o score multiobjetivo é alto.

Regra sugerida:

```text
danger conhecido → configuração inelegível
warning → elegível com penalidade e explicação
missing-data crítico → elegibilidade condicional / confiança reduzida
```

## 12. Regulamentação

Regulação de drones varia por país e pode mudar. O MVP não oferece validação legal/regulatória automática. Caso esse módulo seja adicionado, deverá usar fonte oficial, região e data da regra.

## 13. Privacidade

No MVP, projetos ficam localmente por padrão. Se sincronização, contas ou telemetria forem adicionadas, a coleta e finalidade dos dados devem ser documentadas antes da ativação.

## 14. Texto de disclaimer sugerido

> O DroneCalc fornece cálculos e estimativas com base nos dados informados. Resultados não substituem documentação do fabricante, testes de bancada e validação física da montagem. Condições reais de voo podem produzir valores diferentes.

Esse texto pode ser resumido na UI e detalhado nesta documentação.

## 15. Critério de segurança de produto

Na dúvida entre apresentar um número frágil e informar que faltam dados, o DroneCalc deve preferir **não inventar precisão**.
