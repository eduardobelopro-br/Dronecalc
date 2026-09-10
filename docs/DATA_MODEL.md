# Modelo de Dados e Persistência — DroneCalc

**Versão:** 0.1

## 1. Objetivo

Definir como projetos, componentes, perfis e testes de bancada são persistidos, importados, exportados e migrados sem acoplar o domínio a uma tecnologia específica.

## 2. Estratégia MVP

Persistência local primeiro.

Recomendação:

- IndexedDB como armazenamento principal;
- repositórios abstratos na aplicação;
- JSON versionado para export/import;
- CSV para importação de tabelas de bancada;
- nenhuma conta obrigatória.

`localStorage` pode guardar preferências pequenas (tema, unidade), mas não deve ser a base de projetos complexos.

## 3. Stores lógicos

Independentemente do banco concreto:

- `projects`;
- `components`;
- `benchTests`;
- `flightProfiles`;
- `settings`;
- `migrations`/metadata.

## 4. Projeto persistido

Exemplo conceitual:

```json
{
  "schemaVersion": 1,
  "id": "project_x",
  "name": "Mini LR 4in",
  "motorCount": 4,
  "flightProfileId": "mini-long-range",
  "constraints": {
    "sub250g": true
  },
  "components": [],
  "assumptions": {
    "usableBatteryFraction": 0.8
  },
  "createdAt": "...",
  "updatedAt": "..."
}
```

## 5. Snapshots versus referências

Problema: um componente do catálogo pode mudar depois que um projeto foi criado.

Estratégia recomendada:

- projeto referencia `componentId`;
- ao executar análise, resolver versão atual;
- para reprodutibilidade futura, permitir salvar `componentRevision` ou snapshot dos campos determinantes;
- exportação de projeto deve incluir componentes necessários ou referências resolvíveis.

No MVP, preferir export autocontido para evitar que um arquivo deixe de funcionar porque um catálogo mudou.

## 6. Versionamento de schema

Todo objeto exportável relevante deve possuir `schemaVersion` inteiro.

Regras:

- mudanças compatíveis podem manter versão;
- rename/removal/alteração semântica exige migração;
- migrações são puras e testadas;
- nunca sobrescrever dados persistidos antes de validar migração completa;
- backups lógicos podem ser criados durante migrações de alto risco futuras.

## 7. Versão do motor

Análises exportadas devem registrar `analysisVersion`/`calculationEngineVersion`.

Isso permite explicar por que o mesmo projeto pode produzir resultado diferente após melhoria de um modelo matemático.

## 8. Catálogo de componentes

Um registro de catálogo deve conter:

- schemaVersion;
- id estável;
- tipo;
- fabricante/modelo;
- campos técnicos;
- source metadata;
- `custom`;
- datas/revisões.

Valores desconhecidos permanecem ausentes, não recebem defaults arbitrários.

## 9. Dados de bancada

Curvas são entidades independentes para evitar duplicar amostras em todo motor.

Chave lógica inclui:

- motor;
- hélice/descrição;
- tensão/condição;
- fonte;
- revisão.

A mesma combinação pode possuir múltiplos testes de fontes diferentes. Não mesclar automaticamente.

## 10. Perfis de voo

Perfis padrão podem ser distribuídos junto com a aplicação e carregados de JSON/TS validado.

Campos:

- schemaVersion;
- profileVersion;
- slug;
- priorities;
- targets;
- scoring rules.

Projetos devem registrar a versão utilizada ou snapshot dos objetivos derivados.

## 11. Settings

Preferências típicas:

```ts
interface UserSettings {
  schemaVersion: number
  theme: 'system' | 'light' | 'dark'
  locale: 'pt-BR' | string
  preferredUnits: UnitPreferences
}
```

Hipóteses técnicas globais devem ser usadas com cautela. Se influenciarem um resultado, precisam aparecer na análise e idealmente ser copiadas para o projeto ou analysis context.

## 12. Exportação JSON

Formato sugerido:

```ts
interface DroneCalcExport {
  format: 'dronecalc-project'
  schemaVersion: number
  exportedAt: string
  appVersion?: string
  calculationEngineVersion?: string
  project: DroneProject
  referencedComponents: BaseComponent[]
  referencedBenchTests: MotorPropBenchTest[]
  referencedFlightProfile?: FlightStyleProfile
}
```

O export deve ser autocontido sempre que possível.

## 13. Importação JSON

Pipeline obrigatório:

```text
arquivo
  ↓
parse seguro
  ↓
validação de formato
  ↓
migração por schemaVersion
  ↓
validação semântica
  ↓
preview/identificação de conflitos
  ↓
persistência
```

Nunca persistir parcialmente antes de todas as validações necessárias.

## 14. Importação CSV de bancada

Suportar mapeamento de colunas, pois fabricantes usam cabeçalhos diferentes.

Campos-alvo:

- throttlePercent;
- thrustG;
- currentA;
- voltageV;
- powerW;
- rpm.

Fluxo:

1. selecionar arquivo;
2. detectar delimiter/headers quando possível;
3. mapear colunas;
4. escolher unidades de origem;
5. mostrar preview convertido;
6. validar;
7. salvar teste.

Não assumir que `thrust` está em gramas; pode vir em kg, oz ou N.

## 15. IDs

Usar identificadores estáveis gerados pela aplicação (UUID/ULID ou solução equivalente). Não usar nome/modelo como chave primária.

## 16. Datas

Persistir timestamps em ISO 8601 UTC. Localização é responsabilidade da apresentação.

## 17. Deleção

Ao excluir componente customizado referenciado por projetos:

- impedir deleção destrutiva ou solicitar estratégia;
- opção preferida: arquivar/soft-delete catálogo e preservar referência;
- nunca deixar projeto silenciosamente inválido.

No MVP, componente referenciado pode ser protegido contra exclusão definitiva.

## 18. Conflitos de importação

Se um export contém `componentId` já existente mas conteúdo diferente:

- comparar revision/hash;
- não sobrescrever silenciosamente;
- importar como revisão/nova cópia ou pedir estratégia na UI futura.

## 19. Integridade

Validar:

- referências existentes;
- quantidades positivas;
- valores físicos mínimos;
- versões suportadas;
- IDs únicos;
- curva com amostras válidas.

## 20. Privacidade

MVP local não exige envio de projetos para servidor. Se telemetria for adicionada futuramente, deve ser documentada e evitar coletar conteúdo de projetos sem necessidade/consentimento apropriado.

## 21. Backup futuro

Possibilidades:

- export manual;
- pacote ZIP com projetos/catálogo;
- sincronização opcional com conta;
- integração desktop.

Nada disso deve impedir o usuário de manter um export local legível e versionado.
