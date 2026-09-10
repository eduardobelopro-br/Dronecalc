# Licenciamento — DroneCalc

**Status:** intenção de licenciamento definida; licença jurídica final pendente de confirmação específica.

## 1. Intenção do projeto

O DroneCalc deverá ser disponibilizado para **uso gratuito em finalidades não comerciais**.

Uso comercial deverá exigir autorização expressa ou licenciamento comercial separado do titular do projeto.

## 2. Consequência terminológica

Uma licença que proíbe uso comercial não deve ser descrita como "open source" no sentido usado pela Open Source Initiative. Para evitar ambiguidade, a documentação pública deve preferir termos como:

- `source-available`;
- `uso gratuito para fins não comerciais`;
- `noncommercial license`.

## 3. Candidata recomendada

A candidata inicial é **PolyForm Noncommercial License 1.0.0**.

Motivos:

- foi criada especificamente para software;
- permite uso para finalidades não comerciais;
- permite modificações para finalidades permitidas;
- permite distribuição de cópias e modificações sob suas condições;
- contém cláusulas padronizadas de aviso, patente, responsabilidade e violação.

Fonte oficial:

`https://polyformproject.org/licenses/noncommercial/1.0.0/`

## 4. Decisão pendente antes de criar LICENSE

Antes de inserir o texto jurídico definitivo no repositório, confirmar explicitamente que o titular deseja também permitir, para fins não comerciais:

1. modificação do código;
2. criação de trabalhos derivados;
3. redistribuição de cópias;
4. redistribuição de versões modificadas;
5. uso por instituições de ensino, pesquisa, governo e organizações sem fins lucrativos conforme a definição da licença escolhida.

Se a intenção for apenas permitir execução/uso pessoal, sem redistribuição ou modificação, outra licença pode ser mais adequada.

## 5. Regra para uso comercial

A documentação deve comunicar de forma simples:

```text
Uso não comercial: permitido conforme LICENSE.
Uso comercial: requer autorização/licença separada.
```

Não inventar automaticamente preços, royalties ou condições comerciais. Esses termos ficam fora do repositório até decisão específica.

## 6. Dependências

Antes da primeira release pública considerada estável:

- confirmar licença final;
- adicionar `LICENSE` ou `LICENSE.md` com texto oficial íntegro;
- atualizar `README.md`;
- revisar compatibilidade das licenças das dependências;
- garantir que assets, ícones, fontes e datasets incluídos tenham licenças compatíveis;
- documentar componentes/dados de terceiros quando necessário.

## 7. Dados e referências técnicas

A licença do código do DroneCalc não concede automaticamente direitos sobre:

- manuais de fabricantes;
- tabelas copiadas de vídeos/artigos;
- imagens;
- logotipos;
- datasets de terceiros;
- curvas de bancada licenciadas por terceiros.

O catálogo deve preferir dados estruturados, fatos técnicos e referências de origem, evitando redistribuir conteúdo protegido desnecessariamente.

## 8. Critério de fechamento

Este documento deixa de estar `pendente` quando o usuário confirmar explicitamente a licença padronizada a ser adotada e o arquivo `LICENSE` for adicionado ao repositório.
