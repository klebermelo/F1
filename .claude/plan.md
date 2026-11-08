# Plano de Execução do Projeto: OpenF1 Data Pipeline

Este documento estabelece o plano passo a passo para a criação e desenvolvimento do projeto de Engenharia de Dados voltado ao consumo, processamento e análise dos dados da **OpenF1 API**. 

O desenvolvimento segue rigorosamente o padrão arquitetural, as ferramentas e os critérios de qualidade definidos no `SKILL.md`.

---

## 1. Visão Geral e Objetivos do Projeto

### 1.1. Objetivo Principal
Construir um pipeline de dados em Python, modular, idempotente e testável, capaz de consumir dados históricos e em tempo real da API OpenF1 (telemetria `/car_data`, voltas `/laps`, compostos de pneus `/stints`, sessões `/sessions` e clima `/weather`), realizar sanitização, validação de esquema, transformações de métricas de desempenho e persistência dos dados purificados.

### 1.2. Requisitos de Qualidade (Alinhados ao `SKILL.md`)
* **Gerenciamento de Ambiente:** Exclusivamente via **Poetry** (Python 3.12+).
* **Estrutura:** Layout sob a pasta `src/projeto/`.
* **Qualidade de Código:** Linting, formatação e ordenação de imports via **Ruff**.
* **Testabilidade:** Suíte de testes automatizados via **pytest** com cobertura $\ge 50\%$ gerenciada pelo **pytest-cov**.
* **Documentação:** Documentação estática interativa via **MkDocs Material**.

---

## 2. Estrutura de Diretórios Alvo

```text
openf1_pipeline/
├── src/
│   └── projeto/
│       ├── __init__.py
│       ├── config/               # Definições de constantes, URLs base e Schemas (Pydantic)
│       │   ├── __init__.py
│       │   ├── settings.py
│       │   └── schemas.py
│       ├── connectors/           # Cliente HTTP resiliente para consumir openf1.org
│       │   ├── __init__.py
│       │   └── openf1_client.py
│       ├── ingestion/            # Módulos de extração por endpoint (car_data, laps, sessions)
│       │   ├── __init__.py
│       │   └── extractors.py
│       ├── validation/           # Sanitização, checagem de tipos e nulos
│       │   ├── __init__.py
│       │   └── validators.py
│       ├── transformations/      # Cálculo de estatísticas (velocidade máxima, deltas, pneus)
│       │   ├── __init__.py
│       │   └── telemetry_metrics.py
│       ├── pipelines/            # Orquestração das etapas (ETL/ELT)
│       │   ├── __init__.py
│       │   └── f1_session_pipeline.py
│       └── utils/                # Logging estruturado e helpers de data
│           ├── __init__.py
│           └── logging.py
├── tests/
│   ├── unit/                     # Testes unitários com mocks de API (sem I/O real)
│   │   ├── test_connectors.py
│   │   ├── test_validators.py
│   │   └── test_transformations.py
│   └── integration/              # Testes de integração de pipeline
│       └── test_pipeline_execution.py
├── docs/                         # Documentação técnica MkDocs
│   ├── index.md
│   ├── architecture.md
│   ├── setup.md
│   ├── api_reference.md
│   └── testing.md
├── pyproject.toml                # Configurações centralizadas (Poetry, Ruff, Pytest)
├── poetry.lock
├── mkdocs.yml
├── README.md
└── SKILL.md
```

---

## 3. Fases de Implementação

### Fase 1: Inicialização do Ambiente e Estrutura
* [ ] Inicializar o projeto utilizando Poetry (`poetry new` ou `poetry init` com layout `src`).
* [ ] Configurar a versão do Python no `pyproject.toml` (`^3.12`).
* [ ] Adicionar dependências de produção: `requests`, `pydantic`, `pandas`.
* [ ] Adicionar dependências de desenvolvimento: `ruff`, `pytest`, `pytest-cov`, `mkdocs`, `mkdocs-material`.
* [ ] Criar e centralizar a configuração das ferramentas no `pyproject.toml` (Ruff, Pytest, Coverage $\ge 50\%$).
* [ ] Criar o arquivo `mkdocs.yml` e a estrutura inicial em `docs/`.

### Fase 2: Módulo Core (Conectores e Configurações)
* [ ] **`src/projeto/config/settings.py`**: Definir URLs base (`https://api.openf1.org/v1`), timeouts padrão e parâmetros configuráveis.
* [ ] **`src/projeto/utils/logging.py`**: Configurar o sistema de logging estruturado (sem `print`).
* [ ] **`src/projeto/connectors/openf1_client.py`**: Criar a classe `OpenF1Client` utilizando `requests.Session`, implementando políticas de *retry* (backoff exponencial) e tratamento rigoroso de exceções de rede (`Timeout`, `HTTPError`).

### Fase 3: Ingestão e Validação de Dados (Raw Layer)
* [ ] **`src/projeto/config/schemas.py`**: Definir contratos de dados usando Pydantic para os endpoints `/car_data`, `/laps`, `/sessions` e `/weather`.
* [ ] **`src/projeto/ingestion/extractors.py`**: Criar extratores específicos que aplicam os filtros da OpenF1 API (ex: `driver_number`, `session_key`, operadores `>=`).
* [ ] **`src/projeto/validation/validators.py`**: Implementar funções de higienização para identificar e tratar `NULLs`, dados corrompidos ou leituras de telemetria inválidas.

### Fase 4: Transformação de Dados e Regras de Negócio
* [ ] **`src/projeto/transformations/telemetry_metrics.py`**: Implementar funções puras em Python/Pandas para:
  * Agregação de velocidades máximas e médias por piloto (`speed`, `rpm`).
  * Cálculo de deltas entre voltas e tempo de permanência em pista (`stints`).
  * Correlação entre dados meteorológicos (`track_temperature`) e desgaste de pneus.

### Fase 5: Orquestração e Pipelines (Idempotência)
* [ ] **`src/projeto/pipelines/f1_session_pipeline.py`**: Criar o orquestrador do pipeline responsável pela sequência:
  $$\text{Ingestão} \longrightarrow \text{Validação} \longrightarrow \text{Transformação} \longrightarrow \text{Exportação/Persistência}$$
* [ ] Garantir idempotência: a re-execução para uma mesma `session_key` deve sobrescrever/atualizar os dados no destino sem duplicação.

### Fase 6: Suíte de Testes Automatizados e Qualidade
* [ ] Escrever fixtures com dados falsos (*mocks* em JSON baseados na documentação da OpenF1).
* [ ] Criar testes unitários sob `tests/unit/` garantindo o isolamento da rede.
* [ ] Executar e validar a cobertura de código:
  ```bash
  poetry run pytest --cov=projeto --cov-fail-under=50
  ```
* [ ] Executar auditoria estática e formatação:
  ```bash
  poetry run ruff check . --fix
  poetry run ruff format .
  ```

### Fase 7: Documentação e Conclusão
* [ ] Atualizar `docs/` com a explicação dos módulos, guias de setup e exemplos de uso da API.
* [ ] Validar o build da documentação estática:
  ```bash
  poetry run mkdocs build
  ```
* [ ] Validar o checklist do `SKILL.md`.

---

## 4. Matriz de Mapeamento de Endpoints

| Endpoint OpenF1 | Módulo de Ingestão | Finalidade da Transformação |
| :--- | :--- | :--- |
| `/sessions` | `ingestion/extractors.py` | Mapeamento de metadados da corrida/treino e filtro inicial de `session_key`. |
| `/car_data` | `ingestion/extractors.py` | Métricas de velocidade (`speed`), rotação (`rpm`), uso de acelerador/freio e DRS. |
| `/laps` | `ingestion/extractors.py` | Análise de ritmo de prova por setor (`sector_1`, `sector_2`, `sector_3`). |
| `/stints` | `ingestion/extractors.py` | Ciclo de vida e degradação dos compostos de pneus (`SOFT`, `MEDIUM`, `HARD`). |
| `/weather` | `ingestion/extractors.py` | Avaliação de impacto de temperatura de pista e chuva na performance. |

---

## 5. Critérios de Conclusão da Fase de Planejamento

- [x] Arquitetura definida respeitando o padrão `src/projeto`.
- [x] Ferramentas de CI/CD e Dev alinhadas (`Poetry`, `Ruff`, `pytest`, `pytest-cov`, `MkDocs Material`).
- [x] Cobertura estipulada em no mínimo 50%.
- [x] Estrutura pronta para início do desenvolvimento prático.