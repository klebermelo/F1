# Diretrizes Técnicas e Padrões de Engenharia de Dados com Python

Este documento estabelece as diretrizes técnicas, arquitetura de software, padrões de qualidade, regras de automação e boas práticas de Engenharia de Dados que devem ser seguidas obrigatoriamente por todos os desenvolvedores e agentes de IA no repositório.

O objetivo principal deste guia é garantir modularidade, manutenibilidade, alta testabilidade, idoneidade de dados, idempotência nos pipelines e reprodutibilidade do ambiente de desenvolvimento.

---

## 1. Arquitetura e Organização do Projeto

O projeto adota obrigatoriamente o padrão de layout em `src/`, utilizando o namespace principal `projeto`. Toda a lógica de processamento, ingestão, transformação e orquestração deve estar desacoplada da infraestrutura executora.

### 1.1. Estrutura de Diretórios de Referência

```text
projeto/
├── src/
│   └── projeto/
│       ├── __init__.py
│       ├── config/           # Centralização de configurações e schemas
│       ├── connectors/       # Conectores de bancos de dados, APIs e serviços de arquivos
│       ├── ingestion/        # Módulos de extração e ingestão de dados (Raw/Landing)
│       ├── transformations/  # Regras de negócio, limpezas e transformações puras
│       ├── validation/       # Validações de qualidade de dados, tipos e contratos
│       ├── pipelines/        # Orquestração das etapas de ETL/ELT
│       ├── utils/            # Utilitários compartilhados (logging, helpers)
│       └── ...
├── tests/
│   ├── unit/                 # Testes unitários isolados (sem I/O externo)
│   ├── integration/          # Testes de integração com mocks/fixtures complexas
│   └── ...
├── docs/                     # Documentação técnica em Markdown
│   ├── index.md
│   ├── getting-started.md
│   ├── architecture.md
│   ├── configuration.md
│   ├── development.md
│   ├── testing.md
│   ├── modules/
│   │   ├── ingestion.md
│   │   ├── transformations.md
│   │   └── pipelines.md
│   └── decisions/
├── pyproject.toml            # Configuração centralizada do projeto e ferramentas
├── poetry.lock               # Lockfile de dependências
├── mkdocs.yml                # Configuração do MkDocs Material
├── README.md                 # Visão geral do repositório
└── SKILL.md                  # Este documento de diretrizes técnicas
```

### 1.2. Regras de Design e Separação de Conceitos (SoC)

1. **Modularidade Rigorosa**: Cada pacote dentro de `src/projeto/` tem uma única responsabilidade. Funções não devem misturar conexão de rede com transformação de dados.
2. **Desacoplamento de Fontes de Dados**: As transformações (`transformations/`) devem operar sobre estruturas de dados em memória (ex: Dicionários, DataFrames, Dataclasses) e receber os dados purificados, sem realizar chamadas I/O diretamente.
3. **Proibição de Lógica de Negócio em Scripts/Notebooks**: Jupyter Notebooks e scripts de execução raiz servem apenas para experimentação ou ponto de entrada mínimo. Toda a lógica computacional deve residir nos módulos sob `src/projeto/`.
4. **Substituição de Valores Mágicos**: É proibido utilizar URLs, caminhos de arquivos, credenciais ou constantes diretamente no meio do código. Centralize-os em `config/` ou variáveis de ambiente.

---

## 2. Gerenciamento de Ambiente e Dependências

O **Poetry** é a ferramenta única e oficial para gerenciamento de dependências, empacotamento e ambientes virtuais.

### 2.1. Regras de Gerenciamento

* **Instalação Global Proibida**: É expressamente proibido o uso de `pip install`, `python -m venv` ou ambientes `Conda`. Todas as dependências devem ser gerenciadas via Poetry.
* **Versionamento de Lockfile**: O arquivo `poetry.lock` deve ser commitado no repositório para garantir reprodutibilidade exata entre ambientes de Dev, CI/CD e Produção.
* **Separação de Grupos**: Dependências exclusivas para execução de linters, testes e documentação devem ser adicionadas obrigatoriamente no grupo `dev` (`poetry add --group dev`).
* **Verificação Prévia de Dependências**: Antes de sugerir a adição de um novo pacote, agentes de IA e desenvolvedores devem verificar se a biblioteca padrão ou uma dependência já instalada supre a necessidade.

### 2.2. Comandos Operacionais

```bash
# Instalar todas as dependências declaradas (respeitando o poetry.lock)
poetry install

# Adicionar dependência de produção
poetry add nome-do-pacote

# Adicionar dependência de desenvolvimento
poetry add --group dev nome-do-pacote

# Remover dependência
poetry remove nome-do-pacote

# Executar scripts ou ferramentas dentro da virtualenv
poetry run python script.py
poetry run pytest
poetry run ruff check .
```

---

## 3. Padrões de Desenvolvimento Python

Todo código escrito no projeto deve estar em conformidade com as convenções consolidadas da comunidade Python: **PEP 8** (estilo), **PEP 257** (docstrings) e **PEP 484** (type hints).

### 3.1. Diretrizes de Implementação

* **Type Hints Obrigatórios**: Todas as assinaturas de funções e métodos públicos devem conter anotações explícitas de tipos nos parâmetros e no retorno.
* **Docstrings Padronizadas**: Módulos, classes e funções públicas devem possuir docstrings descrevendo o seu propósito, argumentos, exceções lançadas e retorno.
* **Funções Pequenas e Puras**: Priorize funções com responsabilidade única, puras (sem efeitos colaterais ocultos) e determinísticas nas camadas de transformação.
* **Gerenciamento Seguro de Recursos**: Utilize `context managers` (`with statement`) para manipuladores de arquivos, conexões de banco de dados ou sockets.
* **Tratamento Explicito de Exceções**: Não utilize blocos `except Exception:` genéricos sem motivo justificado. Capture a exceção específica e trate-a adequadamente ou registre log contextualizado.

---

## 4. Boas Práticas Específicas de Engenharia de Dados

### 4.1. Ingestão e Conectividade

* **Conectores Isolados**: Implemente classes de conectores em `connectors/` responsáveis exclusivamente pela autenticação, abertura, fechamento e controle de sessão das fontes.
* **Resiliência de Rede**: Implemente estratégias de *retry* com *exponential backoff* e *timeouts* explícitos em todas as requisições de rede ou banco de dados.
* **Segurança de Credenciais**: Credenciais jamais devem residir no código fonte. Utilize variáveis de ambiente (via `os.getenv` ou `pydantic-settings`).

### 4.2. Transformação de Dados

* **Imutabilidade**: Não altere as estruturas de dados de entrada (*in-place mutation*). Retorne novas estruturas transformadas.
* **Eficiência Computacional**: Evite carregar datasets inteiros em memória se o volume for incompatível com a RAM local/worker. Utilize streaming, iteradores ou geradores quando apropriado.
* **Tratamento de Dados Nulos e Inconsistentes**: Trate valores `NaN`, `NULL` e datas inválidas de forma explícita na camada de sanitização antes de passar para as regras de negócio.

### 4.3. Pipelines e Idempotência

* **Execução Idempotente**: Todo pipeline de dados deve ser projetado de forma que re-execuções com a mesma partição de entrada produzam exatamente o mesmo resultado no destino sem duplicar dados.
* **Orquestração Modular**: Quebre pipelines longos em etapas testáveis (`Ingestion` -> `Validation` -> `Transformation` -> `Persistence`).
* **Rastreabilidade**: Módulos de pipeline devem gerar métricas de execução (contagem de linhas processadas, duração das etapas e volumetria de dados rejeitados).

### 4.4. Qualidade e Confiabilidade dos Dados

* **Validação de Contrato**: Todos os dados recebidos das etapas de ingestão devem ser validados contra esquemas definidos em `validation/` (ex: verificando nomes de colunas, tipos de dados e valores nulos em campos obrigatórios) antes do processamento.

---

## 5. Lint, Formatação e Imports com Ruff

O **Ruff** é a ferramenta **exclusiva e oficial** para análise estática de código (*linting*), formatação e ordenação de imports. Ferramentas como `Flake8`, `Black` e `isort` **não devem ser utilizadas**.

### 5.1. Regras e Execução

Todas as configurações do Ruff devem estar centralizadas no `pyproject.toml`.

```bash
# Executar verificação de linting no projeto
poetry run ruff check .

# Aplicar correções automáticas de linting e organização de imports
poetry run ruff check . --fix

# Verificar conformidade de formatação (sem alterar arquivos)
poetry run ruff format . --check

# Aplicar a formatação de código no projeto inteiro
poetry run ruff format .
```

### 5.2. Ordenação PEP 8 de Imports

O Ruff está configurado para ordenar imports automaticamente seguindo a hierarquia PEP 8:
1. Módulos da biblioteca padrão do Python (`os`, `sys`, `typing`, etc.).
2. Pacotes de terceiros instalados via Poetry (`pandas`, `requests`, `pytest`, etc.).
3. Módulos internos do projeto (`projeto.connectors`, `projeto.transformations`, etc.).

---

## 6. Testes Automatizados com Pytest

O **pytest** é o framework oficial para execução de suítes de testes automatizados, integrado ao **pytest-cov** para medição de cobertura de código.

### 6.1. Requisitos Obrigatórios de Teste

* **Localização**: Todos os testes residem no diretório `tests/`, subdivididos em `unit/` (testes unitários) e `integration/` (testes de integração).
* **Isolamento Total em Testes Unitários**: Testes unitários em `tests/unit/` **jamais** devem realizar chamadas de rede reais, conectar a bancos de dados externos ou acessar o sistema de arquivos de produção. Utilize `pytest fixtures` e `unittest.mock`.
* **Cobertura Mínima Bloqueante**: A suíte de testes deve garantir **no mínimo 50% de cobertura** do código localizado sob `src/projeto/`. Implementações com cobertura inferior a 50% **serão rejeitadas**.

### 6.2. Comandos Oficiais de Teste

```bash
# Executar a suíte completa de testes
poetry run pytest

# Executar testes exibindo as linhas de código não cobertas no terminal
poetry run pytest --cov=projeto --cov-report=term-missing

# Executar verificação estrita de cobertura mínima (falha se < 50%)
poetry run pytest --cov=projeto --cov-report=html --cov-fail-under=50
```

---

## 7. Documentação com MkDocs Material

A documentação técnica do projeto deve ser gerada e mantida via **MkDocs** com o tema **MkDocs Material**.

### 7.1. Estrutura e Regras de Manutenção

* O arquivo de configuração principal é o `mkdocs.yml` na raiz do projeto.
* Todos os arquivos de documentação em Markdown devem residir dentro de `docs/`.
* É obrigatório atualizar a documentação sempre que um novo módulo, conector, pipeline ou variável de ambiente for adicionado ou alterado.

### 7.2. Comandos Oficiais de Documentação

```bash
# Iniciar o servidor local de desenvolvimento da documentação (com live-reload)
poetry run mkdocs serve

# Executar o build estático e validação dos links da documentação
poetry run mkdocs build
```

---

## 8. Logging, Configurações e Tratamento de Erros

### 8.1. Logging Operacional

* **Proibição de `print()`**: É estritamente proibido utilizar o comando `print()` para logs operacionais ou de depuração no código de produção.
* **Módulo `logging`**: Utilize o módulo padrão `logging` configurado com formatador estruturado.
* **Níveis Corretos**:
  * `DEBUG`: Detalhes finos para diagnósticos durante desenvolvimento.
  * `INFO`: Eventos operacionais normais (ex: início/fim de pipeline, total de registros processados).
  * `WARNING`: Eventos inesperados que não impedem a execução.
  * `ERROR`: Falhas em etapas que impedem a conclusão de uma tarefa específica.

### 8.2. Segurança em Logging e Dados

* **Vazamento de Segredos**: Nunca registre senhas, tokens de API, chaves de acesso ou dados pessoais sensíveis (PII) nos arquivos de log.

---

## 9. Segurança e Governança

1. **Anti-versionamento de Segredos**: Arquivos `.env`, chaves de API, certificados e credenciais de bancos de dados nunca devem ser enviados ao controle de versão (`.gitignore` deve estar atualizado).
2. **Princípio do Menor Privilégio**: Conexões com bancos de dados de ingestão ou destino devem utilizar credenciais com os menores privilégios necessários para a operação (ex: apenas `SELECT` na origem; apenas `INSERT/UPDATE` em schemas específicos no destino).
3. **Higienização de Entradas**: Não confie que os dados de entrada de arquivos CSV, APIs ou tabelas externas estejam formatados corretamente. Valide e sanitize todos os inputs na camada de `validation/`.

---

## 10. Fluxo Obrigatório de Desenvolvimento

Toda tarefa de desenvolvimento ou manutenção deve seguir este fluxo sequencial:

```text
[1. Analisar Arquitetura] -> [2. Checar Dependências] -> [3. Planejar Código/Módulo]
        │
        ▼
[4. Codificar em src/projeto/] -> [5. Criar Testes em tests/] -> [6. Rodar Ruff Lint & Format]
        │
        ▼
[7. Rodar Pytest + Coverage (>= 50%)] -> [8. Atualizar Docs/MkDocs] -> [9. Validar Build e Entregar]
```

### Detalhamento dos Passos:

1. **Análise de Arquitetura**: Inspecione o módulo existente em `src/projeto/` onde a alteração se encaixa.
2. **Verificação de Pacotes**: Verifique no `pyproject.toml` se a dependência necessária já existe antes de adicionar novas.
3. **Implementação**: Escreva o código produtivo aplicando Type Hints, Docstrings e boas práticas Python/Engenharia de Dados.
4. **Criação de Testes**: Escreva os testes unitários ou de integração correspondentes em `tests/`.
5. **Verificação de Qualidade**: Execute `poetry run ruff check . --fix` e `poetry run ruff format .`.
6. **Validação de Testes e Cobertura**: Execute `poetry run pytest --cov=projeto --cov-fail-under=50`. Se a cobertura for inferior a 50%, crie mais testes.
7. **Atualização da Documentação**: Escreva ou atualize as páginas `.md` correspondentes em `docs/`.
8. **Validação Final do Build**: Execute `poetry run mkdocs build`.

---

## 11. Configuração Centralizada (`pyproject.toml`)

Abaixo está o modelo oficial de referência para centralização das configurações de ferramentas do projeto:

```toml
[tool.poetry]
name = "projeto"
version = "0.1.0"
description = "Pipeline de Engenharia de Dados em Python"
authors = ["Engenharia de Dados <data-engineering@empresa.com>"]
readme = "README.md"
packages = [{include = "projeto", from = "src"}]

[tool.poetry.dependencies]
python = "^3.12"
pydantic = "^2.6.0"g
requests = "^2.31.0"

[tool.poetry.group.dev.dependencies]
ruff = "^0.3.0"
pytest = "^8.0.0"
pytest-cov = "^4.1.0"
mkdocs = "^1.5.0"
mkdocs-material = "^9.5.0"

[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

# ------------------------------------------------------------------------------
# Configuração do Ruff (Linter, Formatador e Imports)
# ------------------------------------------------------------------------------
[tool.ruff]
line-length = 88
target-version = "py312"
src = ["src", "tests"]

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # Pyflakes
    "I",   # isort (ordenação de imports)
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
]
ignore = []

[tool.ruff.lint.isort]
combine-as-imports = true
known-first-party = ["projeto"]

# ------------------------------------------------------------------------------
# Configuração do Pytest e Cobertura de Código
# ------------------------------------------------------------------------------
[tool.pytest.ini_options]
minversion = "8.0"
addopts = "-ra -q"
testpaths = ["tests"]
pythonpath = ["src"]

[tool.coverage.run]
source = ["src/projeto"]
branch = true

[tool.coverage.report]
show_missing = true
fail_under = 50
```

---

## 12. Critérios Objetivos de Conclusão de Tarefa

Uma tarefa só será considerada **pronta** e elegível para merge/entrega quando todos os critérios abaixo forem satisfeitos:

* [ ] Código-fonte implementado estritamente dentro da estrutura `src/projeto/`.
* [ ] Zero chamadas a `pip install` ou criação manual de `venv`. Dependências gerenciadas via Poetry.
* [ ] Conectores, transformações, validações e pipelines estão desacoplados em seus respectivos módulos.
* [ ] Assinaturas de funções possuem `type hints` e docstrings descritivas.
* [ ] Nenhum erro ou aviso apontado ao rodar `poetry run ruff check .`.
* [ ] Código devidamente formatado ao rodar `poetry run ruff format . --check`.
* [ ] Todos os testes unitários em `tests/` executam com sucesso via `poetry run pytest`.
* [ ] Cobertura de código atingiu ou superou os **50%** (`poetry run pytest --cov=projeto --cov-fail-under=50`).
* [ ] Documentação sob `docs/` atualizada e validada com `poetry run mkdocs build`.