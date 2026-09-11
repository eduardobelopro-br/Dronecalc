# Scraping de Fabricantes e Coleta de Evidências

**Versão:** 0.1  
**Status:** especificação para implementação futura  
**Escopo:** coleta automatizada de dados técnicos diretamente de páginas oficiais de fabricantes para o pipeline de ingestão assistida do DroneCalc.

## 1. Objetivo

O DroneCalc deve permitir que o usuário informe a URL oficial de uma peça/equipamento e o backend colete, de forma controlada e auditável, as informações públicas necessárias para pré-preencher o cadastro técnico.

O **scraper/fetcher é responsável por coletar conteúdo**. A IA não navega livremente nem recebe acesso direto ao banco. Parsers determinísticos extraem primeiro o que já estiver estruturado; IA textual/multimodal é usada somente para interpretar conteúdo não estruturado ou visual.

Fluxo normativo:

```text
URL oficial do fabricante
        ↓
validação de origem/destino
        ↓
fetch/scraping controlado
        ↓
HTML + JSON-LD + tabelas + assets + PDFs
        ↓
extração determinística
        ↓
IA somente onde necessário
        ↓
normalização + schema validation
        ↓
reconciliação entre fontes
        ↓
PostgreSQL STAGING
        ↓
revisão/aprovação
        ↓
catálogo publicado
```

Este documento complementa `ASSISTED_INGESTION.md` e não substitui suas regras de staging, evidência, segurança ou revisão.

## 2. Prioridade das fontes

A procedência deve ser registrada explicitamente. Como orientação de autoridade, usar:

1. página oficial do fabricante e datasheet oficial;
2. dados de bancada independentes verificáveis;
3. distribuidor oficial/autorizado;
4. revendedor;
5. publicação técnica identificável;
6. vídeo técnico com evidência recuperável;
7. comunidade/fórum;
8. heurística do DroneCalc.

Essa ordem é uma **prioridade de evidência**, não autorização para aceitar dados cegamente. Fabricantes podem publicar informações incompletas, inconsistentes, referentes a outra revisão ou sob condições não declaradas.

Conflitos devem permanecer explícitos e seguir o workflow de revisão.

## 3. Estratégia de scraping

A implementação deve seguir uma estratégia incremental e de menor privilégio.

### 3.1 Primeira tentativa — HTTP estático

Preferir requisição HTTP controlada e parsing do documento retornado. Tentar obter:

- JSON-LD/schema.org;
- metadados;
- tabelas HTML;
- blocos de especificações;
- texto técnico;
- links de imagens;
- links para PDFs/datasheets/manuais;
- links ou endpoints públicos de dados quando claramente associados à própria página.

Não usar IA para converter uma tabela HTML simples em JSON se um parser determinístico puder fazê-lo com segurança.

### 3.2 Conteúdo carregado dinamicamente

Quando a resposta estática não contiver os dados apresentados ao usuário, o sistema pode tentar identificar recursos públicos usados pela própria página, desde que isso não envolva contornar autenticação, controles de acesso ou proteções do site.

### 3.3 Browser headless como fallback

Renderização em browser/headless é fallback, não caminho padrão. Só deve ser introduzida quando houver necessidade demonstrada e em ambiente isolado, com limites de CPU, memória, tempo, rede, downloads e navegação.

A implementação inicial não precisa de browser headless para ser considerada válida.

## 4. Dados técnicos procurados

O scraper deve ser orientado pelo schema da categoria. Exemplos gerais:

- fabricante, modelo, variante e revisão;
- massa e dimensões;
- tensão/células/química;
- corrente, potência e limites com suas condições;
- conectores e padrões de montagem;
- especificações elétricas;
- especificações mecânicas;
- compatibilidades declaradas;
- imagens técnicas;
- datasheets/manuais;
- tabelas e curvas de teste.

Para motores, procurar também KV, geometria do estator, configuração declarada, eixo, resistência do enrolamento, corrente sem carga, limites elétricos, eficiência e tabelas motor × hélice × tensão quando existirem.

Para hélices, preservar diâmetro, pitch, número de pás, geometria/modelo e dados de Ct/Cq/Cp ou ensaios quando oficialmente disponibilizados.

Para baterias, preservar química, S, capacidade, tensão nominal/cheia, C-rating, massa, dimensões, conectores, corrente/energia e resistência interna quando declarada, sempre com contexto de ensaio.

## 5. Imagens e documentos

O scraper deve descobrir assets ligados ao produto e classificá-los quando possível:

- foto do produto;
- ficha técnica em imagem;
- tabela;
- desenho dimensional;
- pinout/diagrama;
- gráfico/curva;
- etiqueta;
- PDF/datasheet/manual;
- irrelevante/decorativo.

Downloads passam pelos mesmos controles anti-SSRF da página principal. Validar MIME real, tamanho, quantidade, redirects e hash.

Arquivos aceitos ficam em MinIO/S3-compatible quando a política da fonte permitir. PostgreSQL guarda metadados, hash, origem, relações e `storageKey`.

PDFs e imagens podem ser encaminhados ao pipeline de extração determinística ou ao provider multimodal, conforme capacidade e necessidade.

## 6. Snapshots, revisões e mudanças da fonte

O DroneCalc não deve sobrescrever silenciosamente uma revisão publicada quando uma página mudar.

Cada ingestão deve poder registrar semanticamente:

```text
source
├── URL canônica
├── fabricante/domínio
└── snapshots
    ├── snapshot A — data/hora/hash
    └── snapshot B — data/hora/hash
```

Quando uma nova captura divergir de uma revisão aprovada, gerar candidatos de atualização e revisão. Exemplos:

```text
Peso anterior: 16.2 g
Peso atual:    17.1 g
→ possível revisão de produto ou correção da página
→ needs_review
```

A implementação deve preservar evidência suficiente para distinguir alteração da fonte, nova variante, erro de extração e mudança real do produto.

## 7. Separação scraper × IA

Responsabilidades obrigatórias:

```text
SCRAPER/FETCHER
- controla rede
- coleta recursos
- aplica limites
- produz snapshots/assets

PARSERS
- extraem JSON-LD/tabelas/campos estruturados
- normalizam formatos determinísticos

AI PROVIDER
- interpreta texto não estruturado
- interpreta imagens quando possuir visão
- retorna candidatos estruturados

VALIDATORS/RECONCILIATION
- validam schema
- convertem unidades determinísticas
- detectam inconsistências/conflitos

PERSISTENCE
- grava staging/evidência/revisões
- nunca recebe escrita direta do modelo de IA
```

A IA não recebe credenciais do banco, não decide destinos de rede e não publica diretamente no catálogo.

## 8. Segurança de rede

O backend deve impedir que uma URL de cadastro se transforme em SSRF.

Controles mínimos:

- apenas HTTP/HTTPS;
- rejeitar loopback, link-local, redes privadas, metadata endpoints e destinos internos;
- resolver e validar destino antes da conexão;
- revalidar redirects;
- considerar DNS rebinding na implementação;
- limite de redirects;
- timeout de conexão/leitura;
- limite de bytes por resposta;
- limite de quantidade/tamanho de assets;
- limite de concorrência;
- allowlist de MIME/tipos processados;
- sem `file://`, `ftp://` ou protocolos arbitrários;
- sem reutilização de cookies, tokens ou sessões privadas do usuário;
- logs sem segredos;
- conteúdo remoto tratado como input hostil.

Qualquer browser headless futuro deve possuir isolamento adicional e política de rede equivalente ou mais restritiva.

## 9. Acesso e restrições da fonte

O DroneCalc não deve ser projetado para:

- burlar CAPTCHA;
- contornar autenticação;
- contornar paywall;
- explorar endpoints privados;
- reutilizar sessão autenticada sem fluxo explicitamente projetado e autorizado;
- ignorar deliberadamente restrições técnicas/contratuais de acesso.

A implementação deve permitir política por fonte/domínio e considerar termos aplicáveis, robots/rate limits e direitos sobre conteúdo/arquivos. O fato de um recurso ser publicamente acessível não implica direito ilimitado de redistribuição ou retenção.

## 10. Rate limiting e comportamento responsável

O scraper deve evitar carga desnecessária sobre fabricantes:

- cachear snapshots quando apropriado;
- evitar baixar repetidamente o mesmo asset/hash;
- limitar concorrência por domínio;
- usar backoff para erros transitórios;
- não executar crawling amplo quando o usuário forneceu uma página específica;
- identificar o User-Agent do DroneCalc quando adequado;
- permitir bloqueio/disable de domínios problemáticos.

O objetivo é importar um produto, não indexar indiscriminadamente um site inteiro.

## 11. Reconciliação e autoridade

Exemplo:

```text
Página HTML oficial: resistência = 60 mΩ
Ficha técnica oficial: resistência = 64 mΩ
```

O sistema deve registrar duas evidências e marcar conflito/revisão. Não escolher automaticamente a página apenas por ser HTML nem a imagem apenas por parecer mais recente.

Checks físicos determinísticos podem apontar inconsistências, por exemplo `P = V × I`, mas não reescrevem a declaração original.

## 12. Critérios de aceite

A primeira versão de scraping de fabricante só pode ser considerada concluída quando:

- uma URL oficial pode iniciar importação sem expor rede interna;
- HTML/JSON-LD/tabelas são extraídos deterministicamente quando disponíveis;
- imagens e PDFs elegíveis são descobertos com proveniência;
- dados ficam em staging e não no catálogo publicado;
- cada campo pode apontar para sua fonte/evidência;
- alterações posteriores da página não sobrescrevem silenciosamente revisão aprovada;
- conflitos entre HTML, imagem e documento são apresentados para revisão;
- falha do provider de IA não impede persistência segura dos dados determinísticos já coletados;
- nenhuma dependência de browser headless é obrigatória para páginas estáticas;
- testes cobrem SSRF, redirects, MIME/tamanho, timeout, deduplicação, conflito e revisão;
- nenhuma automação tenta burlar CAPTCHA/autenticação/paywall.

## 13. Relação com o roadmap

Esta especificação detalha principalmente as Fases 5 e 6 do `ROADMAP.md`:

```text
5A source ingestion seguro
5B extração determinística
5C descoberta/armazenamento de assets
5D staging/evidência
6A AI Provider
6B Ollama
6C visão/multimodal
6D reconciliação
6E revisão/publicação
```

A ordem recomendada continua sendo: **scraping determinístico e seguro primeiro; IA depois**.

## 14. Liberdade controlada da IA implementadora

A IA implementadora pode trocar biblioteca HTTP, parser HTML, estratégia de extração, arquitetura interna, política de cache ou solução de browser fallback se encontrar alternativa mais eficaz, eficiente, segura ou testável.

A alteração só é aceitável se:

1. não enfraquecer SSRF, isolamento ou limites de recursos;
2. preservar staging, proveniência e revisão;
3. manter a IA sem acesso direto ao banco/rede arbitrária;
4. priorizar parsing determinístico quando adequado;
5. justificar trade-offs;
6. usar APIs/bibliotecas reais e documentação confiável;
7. adicionar testes equivalentes ou melhores;
8. atualizar documentação/ADR quando houver mudança arquitetural.