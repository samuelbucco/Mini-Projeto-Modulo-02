# Preparação da fonte de dados do dashboard

## Decisão do conector

Para este projeto acadêmico e estático, a fonte recomendada é uma planilha Google conectada diretamente ao Looker Studio.

A base completa continua disponível para auditoria e para uma eventual carga no BigQuery. Para o Google Sheets, o notebook gera uma segunda exportação contendo somente os 19 campos usados pelos KPIs, filtros e visuais do dashboard.

Motivos:

- A fonte enxuta mantém as 342.697 linhas tratadas e todos os indicadores obrigatórios.
- As 19 colunas ocupam 6.511.262 células com o cabeçalho, equivalentes a 65,11% do limite de 10 milhões de células.
- A base não receberá atualizações periódicas.
- O conector Google Sheets é nativo e gratuito no Looker Studio.
- A solução evita a necessidade de projeto Google Cloud com faturamento habilitado.

O BigQuery permanece como alternativa caso a importação ou o desempenho da planilha não sejam satisfatórios.

## Arquivos preparados

- Base completa: `output/data/BPS_20_26_SamuelBucco.csv`
- Fonte enxuta: `output/data/BPS_20_26_SamuelBucco_GoogleSheets.csv`
- Esquema da alternativa BigQuery: `config/bigquery_schema.json`

Os arquivos CSV são gerados pelo notebook e permanecem fora do Git devido ao tamanho.

Validação da fonte enxuta:

| Propriedade | Resultado |
|---|---|
| Linhas de dados | 342.697 |
| Colunas | 19 |
| Células com cabeçalho | 6.511.262 |
| Ocupação do limite | 65,11% |
| Tamanho | 120,41 MB |
| SHA-256 | `ccd5077f7f218785a34aa7ce34d3c4d5498264b4c7bc4883cdc92f2ee62fc3fd` |

## Formato do CSV

- Codificação: UTF-8 sem BOM.
- Separador: vírgula.
- Cabeçalho: uma linha, com nomes únicos.
- Datas: `YYYY-MM-DD`.
- Quantidades: inteiros.
- Preços: valores decimais.
- Valores nulos: campos vazios.
- Quebras de linha internas: não permitidas.
- Identificadores: prefixos textuais `CNPJ ` e `BR ` evitam a perda de zeros e a conversão automática em medidas.

## Fluxo recomendado

```text
notebook -> CSV enxuto -> Google Sheets -> Looker Studio
```

### 1. Gerar a fonte

Execute todas as células de `notebooks/data_analysis.ipynb`. A seção 15 cria e valida o arquivo `BPS_20_26_SamuelBucco_GoogleSheets.csv` em `output/data/`.

### 2. Importar no Google Sheets

1. Criar uma planilha Google vazia.
2. Acessar **Arquivo > Importar > Fazer upload**.
3. Selecionar `BPS_20_26_SamuelBucco_GoogleSheets.csv`.
4. Escolher **Substituir a planilha** ou **Inserir nova(s) página(s)**.
5. Confirmar a vírgula como separador quando a detecção automática não a reconhecer.
6. Renomear a aba importada para `bps_dashboard`.

Não adicionar fórmulas, totais ou abas auxiliares ao arquivo que contém a fonte. O Looker Studio deve realizar as agregações.

### 3. Conferir a importação

Antes da conexão, confirmar:

- 342.697 linhas de dados e uma linha de cabeçalho;
- 19 colunas;
- `id_instituicao`, `id_fornecedor` e `id_fabricante` iniciando com `CNPJ `;
- `codigo_br` iniciando com `BR `;
- datas reconhecidas como data;
- quantidades e preços reconhecidos como números.

### 4. Conectar ao Looker Studio

1. Criar um relatório no Looker Studio.
2. Selecionar o conector **Google Sheets**.
3. Escolher a planilha e a aba `bps_dashboard`.
4. Manter a primeira linha como cabeçalho.
5. Revisar os tipos dos campos.
6. Criar os campos calculados descritos em `docs/dashboard_spec.md`.
7. Reconciliar os seis KPIs antes de construir os demais visuais.

## KPIs de referência

| KPI | Valor sem filtros |
|---|---:|
| Valor total registrado | R$ 78.557.477.974,09 |
| Quantidade total de itens | 57.127.143.721 |
| Registros de compra | 342.697 |
| Instituições compradoras | 831 |
| Fornecedores | 3.502 |
| Preço unitário médio ponderado | 1,375134 |

## Plano alternativo: BigQuery

Se o Google Sheets rejeitar a importação ou apresentar desempenho insuficiente, utilizar o fluxo:

```text
notebook -> CSV completo -> Cloud Storage -> BigQuery -> Looker Studio
```

Nesse caso, seguir o esquema `config/bigquery_schema.json`, preservar CNPJs e códigos como texto e considerar os custos de armazenamento e consulta.

## Referências oficiais

- Google Sheets e limite de células: https://support.google.com/drive/answer/37603
- Importação de bases grandes: https://support.google.com/docs/answer/12236443
- Conector Google Sheets do Looker Studio: https://cloud.google.com/looker/docs/studio/connect-to-google-sheets
- Conexão do Looker Studio ao BigQuery: https://cloud.google.com/looker/docs/studio/connect-to-google-bigquery
