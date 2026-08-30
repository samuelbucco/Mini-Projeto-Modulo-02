# Preparação da fonte de dados do dashboard

## Decisão do conector

A fonte recomendada para o dashboard é uma tabela nativa no BigQuery conectada ao Looker Studio.

Motivos:

- A base tratada contém 342.697 linhas e 28 colunas.
- O CSV consolidado possui 134,48 MB e ultrapassa o limite de 100 MB do conector de upload de arquivos do Looker Studio.
- Uma planilha com 342.697 linhas e 28 colunas teria 9.595.516 células de dados, deixando pouca margem para o limite de 10 milhões de células do Google Sheets e oferecendo menor previsibilidade de desempenho.
- O BigQuery preserva tipos, permite consultas sobre a base completa e possui conector nativo com o Looker Studio.

O BigQuery exige um projeto Google Cloud com faturamento habilitado e pode gerar custos de armazenamento e consulta. Antes da publicação, confirmar se o programa ou a turma fornece um projeto ou orientação institucional.

## Arquivos preparados

- Base gerada localmente: `output/data/BPS_20_26_SamuelBucco.csv`
- Esquema explícito: `config/bigquery_schema.json`

O CSV é gerado pelo notebook e não deve ser versionado no Git devido ao tamanho. O esquema é versionado para preservar os tipos de identificadores, datas, medidas e indicadores de qualidade.

Validação da exportação atual:

| Propriedade | Resultado |
|---|---|
| Linhas | 342.697 |
| Colunas | 28 |
| Tamanho | 134,48 MB |
| SHA-256 | `278233ec087aada9f1fa47a2aa071fc866ba09f0c212716468e45fd234c63ec9` |

## Formato do CSV

- Codificação: UTF-8 sem BOM.
- Separador: vírgula.
- Cabeçalho: uma linha, com nomes únicos contendo letras e sublinhados.
- Datas: `YYYY-MM-DD`.
- CNPJs, código BR e Anvisa: texto.
- Quantidades: inteiros.
- Preços: valores decimais.
- Valores nulos: campos vazios.
- Quebras de linha internas: não permitidas.

## Fluxo recomendado

```text
notebook -> CSV tratado -> Cloud Storage -> tabela BigQuery -> Looker Studio
```

### 1. Gerar a base

Execute todas as células de `notebooks/data_analysis.ipynb`. A última seção cria e valida o CSV em `output/data/`.

### 2. Criar recursos no Google Cloud

1. Criar ou selecionar um projeto Google Cloud.
2. Confirmar que o faturamento está habilitado.
3. Criar um bucket no Cloud Storage.
4. Criar um dataset no BigQuery.
5. Manter bucket e dataset na mesma localização.

### 3. Enviar o CSV ao Cloud Storage

O upload pode ser feito pelo Console do Google Cloud ou pela ferramenta `gcloud` quando configurada.

Não versionar credenciais, chaves, nomes privados de projetos ou arquivos `.env`.

### 4. Criar a tabela no BigQuery

Na criação da tabela:

- Origem: Google Cloud Storage.
- Formato: CSV.
- Linha de cabeçalho a ignorar: `1`.
- Separador: vírgula.
- Codificação: UTF-8.
- Esquema: utilizar `config/bigquery_schema.json`.
- Correspondência das colunas: pelo nome quando disponível.

Utilizar esquema explícito em vez de autodetecção, especialmente para evitar que CNPJs e códigos sejam convertidos em números e percam zeros à esquerda.

### 5. Conectar ao Looker Studio

1. Criar um relatório no Looker Studio.
2. Selecionar o conector BigQuery.
3. Escolher projeto, dataset e tabela.
4. Revisar os tipos dos campos.
5. Criar os campos calculados descritos em `docs/dashboard_spec.md`.
6. Reconciliar os seis KPIs com os valores de referência antes de construir os demais visuais.

## Alternativa sem BigQuery

Se não houver acesso a um projeto Google Cloud com faturamento, uma alternativa para fins acadêmicos é criar fontes agregadas menores por página do dashboard. Essa alternativa reduz a interatividade e exige documentar claramente quais dimensões e filtros permanecem disponíveis.

Não é recomendável usar o upload direto do CSV completo no Looker Studio porque a base consolidada excede o limite oficial. Também não é recomendável depender de uma única Google Sheet tão próxima do limite de células.

## Referências oficiais

- Upload de CSV no Looker Studio: https://docs.cloud.google.com/data-studio/upload-csv-files
- Conexão do Looker Studio ao BigQuery: https://cloud.google.com/looker/docs/studio/connect-to-google-bigquery
- Carregamento de CSV do Cloud Storage no BigQuery: https://docs.cloud.google.com/bigquery/docs/loading-data-cloud-storage-csv
- Limites de arquivos do Google Sheets: https://support.google.com/drive/answer/37603
