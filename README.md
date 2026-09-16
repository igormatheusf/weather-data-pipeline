# Weather Data Pipeline

Pipeline ETL desenvolvido com Pandas e orquestrado pelo Apache Airflow para coletar dados meteorológicos de São Paulo usando a API do OpenWeather, transformar o JSON em um DataFrame tabular e carregar os dados em uma tabela PostgreSQL.

## Tecnologias

- Python 3.12+
- Apache Airflow 3.3.1
- Docker Compose
- PostgreSQL
- Pandas e SQLAlchemy
- API OpenWeather

## Arquitetura

O pipeline é executado pelo DAG `pipeline_weather`, com agendamento de hora em
hora:

1. **Extract:** consulta a API do OpenWeather e salva o retorno em
	 `data/weather_data.json`.
2. **Transform:** normaliza os campos aninhados, ajusta os nomes das colunas e
	 converte os timestamps para o fuso `America/Sao_Paulo`.
3. **Load:** grava os dados na tabela `sp_weather` do PostgreSQL.

![Fluxo ETL do pipeline](docs/images/fluxo-etl-weather-data-pipeline.png)
![Arquitetura de orquestração local](docs/images/arquitetura-orquestracao-local.png)

## Pré-requisitos

- Docker e Docker Compose instalados.
- Uma chave de API do [OpenWeather](https://openweathermap.org/api).
- Um PostgreSQL acessível pelo container do Airflow em `host.docker.internal:5432`.
- Pelo menos 4 GB de memória disponíveis para o Docker, conforme a configuração
	padrão do Airflow.

## Configuração

Crie o arquivo `config/.env` com as credenciais da API e do PostgreSQL:

```env
API_KEY=sua_chave_openweather
user=seu_usuario_postgres
password=sua_senha_postgres
database=seu_banco_postgres
```

O arquivo `config/.env` não deve ser versionado. A configuração do Docker
Compose também pode usar um arquivo `.env` na raiz para variáveis do Airflow,
como `AIRFLOW_UID`, `FERNET_KEY` e as credenciais do usuário administrador.

## Como executar

Inicialize os serviços:

```bash
docker compose up airflow-init
docker compose up -d
```

Abra a interface do Airflow em [http://localhost:8080](http://localhost:8080).
Por padrão, o usuário e a senha são `airflow`. Ative o DAG `pipeline_weather` e
dispare uma execução manual ou aguarde o agendamento horário.

Para acompanhar os logs:

```bash
docker compose logs -f airflow-scheduler
docker compose logs -f airflow-worker
```

Para encerrar os serviços:

```bash
docker compose down
```

## Estrutura do projeto

```text
.
├── dags/                 # DAGs do Airflow
├── src/                  # Extração, transformação e carga
├── data/                 # JSON coletado e arquivo temporário do pipeline
├── docs/                 # Imagens e JSON Excalidraw com a arquitetura do projeto
├── config/               # Configurações do Airflow e variáveis locais
├── logs/                 # Logs das execuções do Airflow
├── notebooks/            # Análises exploratórias
├── docker-compose.yaml   # Serviços do Airflow, Redis e PostgreSQL interno
└── pyproject.toml        # Dependências do projeto Python
```

## Desenvolvimento local

Instale as dependências com `uv`:

```bash
uv sync
```

Os módulos em `src/` dependem dos arquivos gerados pelo pipeline e das
variáveis de ambiente. Para uma execução completa, prefira usar o ambiente
orquestrado pelo Airflow.

## Observações

- O DAG está configurado para não executar períodos anteriores (`catchup=False`).
- As tentativas com falha são repetidas até duas vezes, com intervalo de cinco
	minutos.
- A tabela `sp_weather` é criada ou complementada pelo `pandas.DataFrame.to_sql`.
- Antes de executar em produção, revise as credenciais padrão do Airflow, a
	política de persistência do PostgreSQL e o tratamento de falhas da API.
