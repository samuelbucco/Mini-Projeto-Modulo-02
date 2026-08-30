# Especificação do dashboard BPS 2020-2026

## Objetivo

Construir no Looker Studio um dashboard analítico para acompanhar os registros de compras de medicamentos e dispositivos médicos do Banco de Preços em Saúde entre 2020 e 2026.

O painel deve apoiar exploração e investigação. Diferenças de preço não devem ser apresentadas isoladamente como evidência de economia, sobrepreço ou irregularidade.

## Público e decisões apoiadas

- Gestores e analistas de compras públicas em saúde.
- Identificação de períodos, localidades, instituições e produtos com maior volume registrado.
- Priorização de itens comparáveis para investigação de diferenças de preço.
- Acompanhamento da participação de fornecedores, fabricantes e modalidades de compra.

## Estrutura proposta

### Página 1 - Visão geral

Objetivo: apresentar o panorama executivo e a evolução temporal.

KPIs obrigatórios:

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

Observação obrigatória: marcar 2026 como período parcial, com cobertura observada até 05/03/2026.

### Página 2 - Compradores e mercado fornecedor

Objetivo: analisar a concentração geográfica e os participantes das compras.

Visuais:

1. Ranking de municípios por valor total.
2. Ranking de instituições por valor total, identificadas pelo CNPJ.
3. Ranking de fornecedores por valor total.
4. Ranking de fabricantes por valor total.
5. Tabela detalhada com participação, quantidade e número de registros.

Drill-down recomendado:

```text
UF -> Município -> Instituição
```

### Página 3 - Produtos e preços

Objetivo: identificar produtos relevantes e oportunidades de investigação de preços.

Visuais:

1. Produtos com maior valor total.
2. Produtos com maior quantidade adquirida.
3. Evolução do preço unitário mediano para o produto selecionado.
4. Distribuição ou tabela comparativa de preços por fornecedor.
5. Tabela de candidatos a investigação, ordenada pela razão entre os percentis 90 e 10.

Para comparar preços, utilizar conjuntamente:

- `codigo_br`;
- `descricao_catmat`;
- `unidade_fornecimento_capacidade`;
- período selecionado;
- fabricante e fornecedor, quando aplicável.

Não utilizar a soma de `preco_unitario`.

## Filtros

Filtros globais:

- Ano ou intervalo da data de compra.
- UF.
- Município.
- Instituição compradora.
- Código BR/CATMAT ou descrição do produto.
- Unidade de fornecimento/capacidade.
- Modalidade da compra.
- Tipo da compra.
- Fornecedor.
- Fabricante.

Ordem recomendada na interface:

```text
Período | UF | Município | Produto | Modalidade | Mais filtros
```

Os filtros de instituição, fornecedor e fabricante podem ficar em uma área secundária para evitar excesso de controles na primeira leitura.

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

### Participação no valor total

```text
SUM(preco_total) / SUM(preco_total) OVER()
```

Se a fonte ou o conector não aceitar a função analítica, calcular a participação com comparação ao total do gráfico ou preparar o campo antes da conexão.

## Interações

- Seleções nos gráficos devem filtrar os demais visuais da página.
- Rankings devem permitir ordenação por valor, quantidade ou registros.
- O clique em uma UF deve restringir municípios e instituições.
- O clique em um produto deve restringir os gráficos de preço.
- Deve existir uma forma clara de limpar os filtros.
- Títulos precisam refletir o contexto selecionado sempre que possível.

## Hierarquia visual

1. Título, período coberto e aviso de parcialidade.
2. Filtros globais.
3. Seis cartões de KPI.
4. Visual principal da página.
5. Rankings e tabelas de apoio.
6. Nota metodológica e data de atualização.

## Identidade visual

- Fundo claro e alto contraste.
- Verde escuro como cor principal, associado à saúde e à gestão pública.
- Verde médio para séries principais.
- Azul como cor secundária de comparação.
- Laranja apenas para alertas e períodos parciais.
- Cinza para eixos, grades e informações secundárias.
- Evitar arco-íris categórico, efeitos 3D e excesso de bordas.

Paleta sugerida:

| Uso | Cor |
|---|---|
| Verde principal | `#1B5E20` |
| Verde de dados | `#43A047` |
| Azul secundário | `#1565C0` |
| Laranja de atenção | `#EF6C00` |
| Texto principal | `#1F2937` |
| Fundo | `#F7F9F8` |

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
- Validar amostras de produtos comparáveis diretamente na base.
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
