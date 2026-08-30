# Banco de Preços em Saúde - Análise e Visualização de Dados

Mini-projeto do Módulo 2 do curso de Análise e Visualização de Dados do programa SC Tec.

## Objetivo

Desenvolver uma solução de Business Intelligence para acompanhar as compras de medicamentos e dispositivos médicos registradas no Banco de Preços em Saúde (BPS) entre 2020 e 2026.

O projeto busca analisar a evolução dos valores registrados, a distribuição geográfica das compras, os principais produtos, instituições, fornecedores e fabricantes, além de identificar diferenças de preços que mereçam investigação.

## Contextualização

O BPS reúne informações de compras públicas e privadas na área da saúde. A transformação desses registros em indicadores e visualizações pode apoiar o planejamento, a negociação e o acompanhamento dos gastos.

Diferenças de preços não são interpretadas isoladamente como comprovação de economia, sobrepreço ou irregularidade. Comparações devem considerar produto, apresentação, unidade de fornecimento, fabricante, quantidade, localidade, modalidade e período.

## Fonte dos dados

- [Banco de Preços em Saúde - BPS](https://dadosabertos.saude.gov.br/dataset/bps)
- [Página institucional do BPS](https://www.gov.br/saude/pt-br/acesso-a-informacao/banco-de-precos)
- Arquivos anuais em CSV referentes a 2020, 2021, 2022, 2023, 2024, 2025 e 2026
- Dicionário local: [`docs/Metadados_BPS_07_04_2026.pdf`](docs/Metadados_BPS_07_04_2026.pdf)

## Estrutura do projeto

```text
.
├── config/
│   └── bigquery_schema.json
├── data/                  # Arquivos CSV anuais do BPS
├── docs/                  # Enunciado e dicionário de dados
│   └── dashboard_spec.md  # Arquitetura, métricas e regras do dashboard
├── notebooks/
│   └── data_analysis.ipynb
├── output/data/           # Base consolidada gerada e ignorada pelo Git
├── .gitignore
├── README.md
└── requirements.txt
```

## Procedimentos realizados

O notebook [`notebooks/data_analysis.ipynb`](notebooks/data_analysis.ipynb) contém um processo reproduzível com as seguintes etapas:

1. Localização dos CSVs com `glob`.
2. Leitura separada dos arquivos em UTF-8, utilizando `;` como separador.
3. Comparação das colunas, da ordem e dos tipos inferidos em cada ano.
4. Consolidação das bases somente após a validação de compatibilidade.
5. Preservação da base bruta em `bps_raw`.
6. Criação da base tratada `bps_clean`, disponível também pelo atalho `bps`.
7. Diagnóstico e tratamento auditável dos dados.
8. Análise exploratória e cálculo dos KPIs obrigatórios.

## Tratamentos e validações

- Preservação de `codigo_br`, `anvisa` e CNPJs como identificadores textuais.
- Conversão dos CNPJs para 14 dígitos, preservando zeros à esquerda.
- Conversão de `compra` e `insercao` para datas.
- Conversão de quantidades, capacidades e preços para tipos numéricos.
- Normalização de espaços em campos textuais.
- Preenchimento apenas de rótulos categóricos ausentes; medidas não foram imputadas.
- Remoção de 19 linhas exatamente duplicadas na cópia tratada.
- Criação de indicadores para inserções anteriores à compra e divergências no preço total.
- Reconciliação de `preco_total` com `qtd_itens_comprados × preco_unitario`.

Resultados das validações:

- 0 datas de compra inválidas.
- 0 divergências entre o ano e a data da compra.
- 0 quantidades ou preços negativos.
- 0 divergências entre o preço total informado e o valor recalculado.
- 12 inserções anteriores à data da compra, mantidas e sinalizadas para investigação.

## KPIs

| KPI | Resultado | Definição |
|---|---:|---|
| Valor total registrado | R$ 78.557.477.974,09 | Soma de `preco_total` |
| Quantidade total de itens | 57.127.143.721 | Soma de `qtd_itens_comprados` |
| Registros de compra | 342.697 | Quantidade de linhas da base tratada |
| Instituições compradoras | 831 | Contagem distinta de CNPJ da instituição |
| Fornecedores | 3.502 | Contagem distinta de CNPJ do fornecedor |
| Preço unitário médio ponderado | R$ 1,375134 por item | Valor total dividido pela quantidade total |

O preço unitário médio ponderado global mistura produtos e apresentações distintas. Sua interpretação exige filtros que mantenham os itens comparáveis.

## Análises exploratórias iniciais

- A base possui registros de 24 unidades federativas.
- Paraná, São Paulo e Ceará apresentam os maiores valores registrados.
- O ano de 2026 é parcial, com compras registradas até 5 de março de 2026.
- Quantidades e preços apresentam forte assimetria e valores extremos relevantes.
- A investigação de preços compara o mesmo código CATMAT e a mesma unidade/capacidade.
- Os candidatos são classificados pela razão entre os percentis 90 e 10, reduzindo a influência de extremos isolados.

## Limitações

- Os valores representam registros informados ao BPS e não necessariamente todo o universo de compras em saúde.
- O período de 2026 ainda está incompleto.
- `capacidade`, `unidade_medida`, `generico` e `anvisa` possuem ausência relevante.
- O dicionário não descreve de forma inequívoca todas as colunas presentes nos CSVs, como `esfera` e `unidade_fornecimento_capacidade`.
- Diferenças de preços exigem investigação contextual e não comprovam irregularidade.

## Como reproduzir

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Depois, abra `notebooks/data_analysis.ipynb`, selecione o kernel da `.venv` e execute as células em ordem.

## Próximas etapas

- Construir os visuais e filtros conforme [`docs/dashboard_spec.md`](docs/dashboard_spec.md).
- Gerar e carregar a base conforme [`docs/data_source_setup.md`](docs/data_source_setup.md).
- Construir o dashboard no Looker Studio ou Power BI.
- Documentar as descobertas e recomendações finais.
- Adicionar imagens e o link do dashboard.
- Gravar o vídeo de apresentação.
