# Otimização Relacional e Análise de KPIs — Reservas Hoteleiras

Projeto que pega uma planilha de **119.390 reservas hoteleiras** numa única tabela com 32 colunas, separa em **8 tabelas conectadas por IDs** (1 fato + 7 dimensões) usando SQL, faz **análise exploratória dos dados** identificando problemas de qualidade, e responde **5 perguntas de negócio** com SQL puro e 2 visualizações.

> **Stack:** Python · pandas · SQL (DuckDB) · matplotlib

## Por que esse projeto

A planilha original `hotel_bookings.csv` (do Kaggle) tem 119 mil linhas e mistura tudo em uma única tabela: dados da reserva, do hotel, do cliente, do canal de venda. Palavras como `'No Deposit'` ou `'Online TA'` aparecem repetidas em todas as linhas.

Numa empresa real isso é problema:
- Ocupa espaço desnecessário
- Dá margem pra erro de digitação (uma reserva como `'no deposit'` em minúsculo já quebra um relatório)
- Pra trocar o nome de um canal você teria que mexer em milhares de linhas

A solução é guardar cada categoria uma única vez numa tabela auxiliar e usar um ID na tabela principal pra referenciar. Isso é o conceito de **modelagem relacional**.

## O que tem nesse notebook

Um único notebook portável que faz **toda a transformação e análise** num pipeline SQL completo:

1. **Setup e carga** da base original
2. **Análise exploratória** com identificação de problemas de qualidade na coluna `adr`
3. **Criação das 7 tabelas dimensão** via SQL
4. **Criação da tabela fato** `fato_reserva` com JOINs
5. **Validação da modelagem** (3 checks de integridade)
6. **5 perguntas de negócio** respondidas com SQL + 2 visualizações
7. **Conclusões** com decisões de negócio documentadas

### Por que SQL via Python (DuckDB)

Em vez de usar blocos SQL nativos de uma ferramenta específica (Deepnote, BigQuery, etc.), uso a biblioteca **DuckDB** dentro do Python. Vantagens:

- **Portável:** o notebook roda em qualquer ambiente Jupyter (Deepnote, VS Code, Google Colab, JupyterLab)
- **Sem servidor:** roda SQL direto sobre arquivos CSV, sem precisar instalar banco de dados
- **Profissional:** é o mesmo padrão usado em pipelines de dados modernos

## Como rodar

**Pré-requisitos:** Python 3.10+

```bash
# 1. Clonar o repositório
git clone https://github.com/lucaneves1/reservas-hoteleiras.git
cd reservas-hoteleiras

# 2. Instalar as bibliotecas
pip install -r requirements.txt

# 3. Baixar o dataset
# https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand
# Salvar como hotel_bookings.csv na raiz do projeto

# 4. Abrir o notebook
jupyter notebook reservas_hoteleiras_analise.ipynb
```

Ou simplesmente abre o notebook no **Deepnote** / **Google Colab** fazendo upload da CSV junto.

## Decisões de negócio nas métricas

Uma parte importante do projeto é o **tratamento de problemas de qualidade** identificados na análise exploratória. A coluna `adr` (diária média) tem:
- Valores negativos (provável erro de cadastro)
- Valores zerados em reservas concretizadas (cortesias ou erros)
- Um outlier de €5.400 (provável erro de digitação)

As métricas de receita usam filtros explícitos:

| Métrica | Filtro aplicado | Justificativa |
|---|---|---|
| Receita Total | `cancelada = 0`, `adr > 0`, `adr < 500` | Receita real, sem outliers |
| Diária Média | `adr > 0`, `adr < 500` | Preço de mercado, sem extremos |
| Taxa Cancelamento | sem filtros | Métrica de operação |

## Perguntas de negócio respondidas

1. Qual canal de venda tem a maior taxa de cancelamento?
2. Qual segmento de mercado gera mais receita?
3. Reservas feitas com muita antecedência cancelam mais?
4. Quais meses têm maior receita?
5. Top 10 países que mais geram receita

## Estrutura do projeto

```
.
├── reservas_hoteleiras_analise.ipynb   # Notebook principal (tudo aqui)
├── hotel_bookings.csv                  # Dataset (baixar do Kaggle)
├── requirements.txt
├── .gitignore
└── README.md
```

## O que aprendi com esse projeto

- Aplicar conceitos de **modelagem relacional** (tabela fato + dimensões) em dados reais
- Escrever **SQL com JOIN, GROUP BY, CASE, window functions** (ROW_NUMBER) pra criar IDs e fazer análises
- Fazer **análise exploratória crítica** identificando problemas de qualidade antes de tirar conclusões
- Documentar **decisões de negócio** com justificativas técnicas
- Usar **DuckDB** pra rodar SQL portável sobre CSVs sem precisar de servidor

## Próximos passos

- Construir dashboard interativo (Power BI ou Looker Studio) consumindo o modelo
- Migrar pra banco persistente (PostgreSQL) com chaves estrangeiras forçadas
- Modelo de Machine Learning pra prever cancelamentos

## Sobre

Projeto desenvolvido como parte do meu portfólio de Análise de Dados.

**Luca** — estudante de Engenharia de Software (INFNET), foco em Sistemas Complexos. Mirando posição de Analista de Dados Pleno até 2029.
