# Especificação do dashboard BPS 2020-2026

## Objetivo

Construir no Looker Studio um dashboard analítico para acompanhar os registros de compras de medicamentos e dispositivos médicos do Banco de Preços em Saúde entre 2020 e 2026.

O painel deve apoiar exploração e investigação. Diferenças de preço não devem ser apresentadas isoladamente como evidência de economia, sobrepreço ou irregularidade.

## Público e decisões apoiadas

- Gestores e analistas de compras públicas em saúde.
- Identificação de períodos, localidades, instituições e produtos com maior volume registrado.
- Acompanhamento da evolução do preço unitário mediano de produtos comparáveis.
- Acompanhamento da participação de fornecedores, fabricantes e modalidades de compra.

## Estrutura implementada

### Página 1 - Visão geral

Objetivo: apresentar o panorama executivo e a evolução temporal.

KPIs:

1. Valor total registrado.
2. Quantidade total de itens comprados.
3. Número de registros de compra.
4. Instituições compradoras distintas.
5. Fornecedores distintos.
6. Preço unitário médio ponderado.

Visuais:

1. Série temporal do valor total por ano.
2. Barras horizontais do valor total por UF.
3. Barras do valor total por modalidade de compra.
4. Tabela resumida anual com valor, quantidade e registros.

O cabeçalho informa que 2026 é um período parcial, com cobertura observada até 05/03/2026.

### Página 2 - Compradores e mercado fornecedor

Objetivo: analisar a concentração geográfica e os participantes das compras.

Visuais:

1. Barras horizontais com o ranking de municípios por valor total.
2. Tabela de ranking de instituições por valor total, com nome e CNPJ.
3. Tabela de ranking de fornecedores por valor total, com nome e CNPJ.
4. Tabela de ranking de fabricantes por valor total.

Os rankings foram limitados ao espaço disponível e ordenados pelo valor registrado em ordem decrescente. A tabela detalhada inicialmente prevista foi retirada para preservar a legibilidade e evitar rolagem vertical excessiva.

### Página 3 - Produtos e preços

Objetivo: identificar produtos relevantes e acompanhar a evolução do preço unitário mediano em recortes comparáveis.

Visuais:

1. Produtos com maior valor total.
2. Produtos com maior quantidade adquirida.
3. Evolução do preço unitário mediano para o produto selecionado.

Os rankings são apresentados em tabelas, pois as descrições dos produtos são extensas. A comparação por fornecedor e a tabela de candidatos por razão P90/P10 não integram a versão final, para manter o dashboard em três páginas sem comprometer a legibilidade.

Para comparar preços, utilizar conjuntamente:

- `codigo_br`;
- `descricao_catmat`;
- `unidade_fornecimento_capacidade`;
- período selecionado;
- filtros geográficos e de modalidade, quando aplicáveis.

Não utilizar a soma de `preco_unitario`.

## Filtros implementados

Filtros comuns às três páginas:

- Ano ou intervalo da data de compra.
- UF.
- Município.
- Código BR/CATMAT ou descrição do produto.
- Modalidade da compra.

Ordem na interface:

```text
Período | UF | Município | Modalidade | Produto
```

Filtros específicos:

- Página 2: Instituição compradora.
- Página 3: Unidade de fornecimento/capacidade, no lugar do filtro de instituição.

A quantidade de controles foi deliberadamente limitada para preservar espaço e clareza. Tipo da compra, fornecedor e fabricante permanecem disponíveis como dimensões da fonte, mas não como filtros visíveis na versão final.

Na fonte otimizada para Google Sheets, os identificadores correspondentes são `id_instituicao`, `id_fornecedor` e `id_fabricante`. Seus valores recebem o prefixo `CNPJ ` para permanecerem textuais durante a importação. O campo `codigo_br` recebe o prefixo `BR ` pelo mesmo motivo.

## Campos calculados

### Valor total registrado

```text
SUM(preco_total)
```

### Quantidade total de itens comprados

```text
SUM(qtd_itens_comprados)
```

### Número de registros de compra

```text
Record Count
```

### Instituições compradoras

```text
COUNT_DISTINCT(id_instituicao)
```

### Fornecedores

```text
COUNT_DISTINCT(id_fornecedor)
```

### Preço unitário médio ponderado

```text
SUM(preco_total) / SUM(qtd_itens_comprados)
```

## Interações

- Seleções nos gráficos devem filtrar os demais visuais da página.
- Rankings são ordenados pela métrica principal em ordem decrescente.
- Os controles de UF e município restringem os visuais das respectivas páginas.
- O clique em um produto restringe o gráfico de preço mediano.
- Deve existir uma forma clara de limpar os filtros.
- Botões permitem avançar e retornar entre as páginas; a última página apresenta somente o retorno.

## Hierarquia visual

1. Título geral e título temático da página.
2. Período coberto e aviso de parcialidade.
3. Filtros.
4. Cartões de KPI na visão geral ou rankings nas páginas analíticas.
5. Gráficos e tabelas de apoio.
6. Fonte, nota metodológica e data de atualização.

## Identidade visual

- Fundo claro e alto contraste.
- Azul como cor principal nos botões, divisórias, cabeçalhos de tabelas e séries de dados.
- Cinza-escuro nos títulos e textos principais.
- Cinza-claro nos fundos, eixos, grades, bordas e linhas alternadas das tabelas.
- Branco no interior dos cartões, controles e áreas de visualização.
- Evitar arco-íris categórico, efeitos 3D e excesso de bordas.

Paleta de referência:

| Uso | Cor |
|---|---|
| Azul principal | `#4285E4` |
| Texto e títulos | `#616161` |
| Linhas e bordas | `#D0D0D0` |
| Fundo da página | `#F5F5F5` |
| Fundo dos componentes | `#FFFFFF` |

## Formatação

- Valores monetários: `R$` com unidades abreviadas nos cartões e valor completo no detalhe.
- Quantidades e contagens: separador de milhares e nenhuma casa decimal.
- Preços unitários: precisão adaptada ao produto; evitar arredondamento que esconda valores muito pequenos.
- Datas: `DD/MM/AAAA` na interface e tipos de data válidos na fonte.
- CNPJs e códigos: dimensões textuais, nunca medidas numéricas.

## Controles de qualidade

- Reconciliar os seis KPIs do dashboard com a tabela `kpis` do notebook.
- Confirmar que filtros alteram todos os cartões esperados.
- Verificar que nenhum gráfico soma `preco_unitario`.
- Conferir que 2026 aparece como parcial.
- Confirmar que a evolução do preço mediano responde à seleção de produto e unidade de fornecimento.
- Validar amostras de produtos comparáveis diretamente na base.
- Conferir a ordenação decrescente e a formatação monetária dos rankings.
- Testar todos os botões de navegação entre as três páginas.
- Manter as 12 inserções anteriores à compra sinalizadas, sem exclusão automática.

## Valores de referência para validação inicial

| KPI | Valor sem filtros |
|---|---:|
| Valor total registrado | R$ 78.557.477.974,09 |
| Quantidade total de itens | 57.127.143.721 |
| Registros de compra | 342.697 |
| Instituições compradoras | 831 |
| Fornecedores | 3.502 |
| Preço unitário médio ponderado | R$ 1,375134 por item |

Esses valores são pontos de reconciliação da versão atual da base e devem ser atualizados se o processo de tratamento for alterado.
