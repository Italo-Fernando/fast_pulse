<div align="center">

# ⚡ Fast Pulse

**API REST de gerenciamento de usuários e tarefas (todos), construída com FastAPI.**

[![Python](https://img.shields.io/badge/Python-3.14-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.136-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-4169E1?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

</div>

---

## 📑 Índice

- [Sobre o projeto](#-sobre-o-projeto)
- [Tecnologias](#-tecnologias)
- [Estrutura do projeto](#-estrutura-do-projeto)
- [Variáveis de ambiente](#-variáveis-de-ambiente)
- [🐳 Subindo com Docker (recomendado)](#-subindo-com-docker-recomendado)
- [💻 Rodando localmente (branch `dev`)](#-rodando-localmente-branch-dev)
- [📡 Endpoints da API](#-endpoints-da-api)
- [🧪 Testes e qualidade de código](#-testes-e-qualidade-de-código)
- [📄 Licença](#-licença)

---

## 🎯 Sobre o projeto

O **Fast Pulse** é uma API assíncrona que oferece:

- 👤 **Gestão de usuários** — cadastro, listagem, atualização e remoção.
- 🔐 **Autenticação JWT** — login via OAuth2 (password flow) com tokens de acesso e *refresh*.
- ✅ **Gestão de tarefas (todos)** — CRUD de tarefas vinculadas ao usuário autenticado.

O versionamento do banco é feito com **Alembic** e a documentação interativa é gerada
automaticamente pelo FastAPI (Swagger UI e ReDoc).

---

## 🛠 Tecnologias

| Categoria         | Ferramenta                                                  |
| ----------------- | ----------------------------------------------------------- |
| Linguagem         | Python 3.14                                                 |
| Framework web     | FastAPI                                                     |
| ORM               | SQLAlchemy 2.0 (modo assíncrono)                            |
| Migrações         | Alembic                                                     |
| Banco de dados    | PostgreSQL 17 (produção) · SQLite (desenvolvimento local)   |
| Autenticação      | PyJWT + pwdlib (Argon2)                                     |
| Configuração      | Pydantic Settings                                           |
| Gerenciador de deps | Poetry                                                    |
| Qualidade         | Ruff · Typos · Taskipy                                      |
| Testes            | Pytest · pytest-cov · factory-boy · testcontainers          |

---

## 📁 Estrutura do projeto

```
fast_pulse/
├── fast_pulse/             # Código-fonte da aplicação
│   ├── app.py              # Ponto de entrada (instância FastAPI)
│   ├── settings.py         # Configurações via Pydantic Settings
│   ├── database.py         # Engine e sessão assíncrona
│   ├── models.py           # Modelos SQLAlchemy
│   ├── schemas.py          # Schemas Pydantic
│   ├── security.py         # Hash de senha e JWT
│   └── routers/            # Rotas: users, auth, todos
├── migrations/             # Migrações Alembic
├── tests/                  # Suíte de testes
├── compose.yaml            # Orquestração Docker (app + PostgreSQL)
├── Dockerfile              # Imagem da aplicação
├── entrypoint.sh           # Migrações + inicialização do servidor
├── alembic.ini             # Configuração do Alembic
└── pyproject.toml          # Dependências e tarefas (taskipy)
```

---

## 🔑 Variáveis de ambiente

A aplicação carrega as configurações de um arquivo `.env` na raiz do projeto.

| Variável                      | Descrição                                         |
| ----------------------------- | ------------------------------------------------- |
| `DATABASE_URL`                | String de conexão do banco de dados               |
| `SECRET_KEY`                  | Chave secreta usada na assinatura dos tokens JWT  |
| `ALGORITHM`                   | Algoritmo de assinatura do JWT (ex.: `HS256`)     |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Tempo de expiração do token de acesso (em minutos)|

Variáveis adicionais usadas pelo container do PostgreSQL (no modo Docker):

| Variável            | Descrição                       |
| ------------------- | ------------------------------- |
| `POSTGRES_USER`     | Usuário do banco                |
| `POSTGRES_DB`       | Nome do banco                   |
| `POSTGRES_PASSWORD` | Senha do banco                  |

> 💡 Gere uma `SECRET_KEY` segura com: `python -c "import secrets; print(secrets.token_hex(32))"`

---

## 🐳 Subindo com Docker (recomendado)

Esta é a forma mais simples de rodar o projeto: o `docker compose` sobe a aplicação
**e** um banco PostgreSQL já configurados, executa as migrações automaticamente e
expõe a API na porta `8000`.

### Pré-requisitos
- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/) (incluso no Docker Desktop)

### Passo a passo

1. **Clone o repositório** (a branch `prd` já contém o setup Docker):

   ```bash
   git clone <url-do-repositorio> fast_pulse
   cd fast_pulse
   git checkout prd
   ```

2. **Crie o arquivo `.env`** na raiz com as variáveis abaixo:

   ```env
   # Banco de dados (container PostgreSQL)
   POSTGRES_USER=app_user
   POSTGRES_DB=app_db
   POSTGRES_PASSWORD=app_password

   # Aplicação
   DATABASE_URL="postgresql+psycopg://app_user:app_password@fastzero_database:5432/app_db"
   SECRET_KEY="troque-por-uma-chave-segura"
   ALGORITHM="HS256"
   ACCESS_TOKEN_EXPIRE_MINUTES=30
   ```

   > ℹ️ No Docker, o host do banco é o nome do serviço **`fastzero_database`** (definido no `compose.yaml`), e não `localhost`.

3. **Suba os containers:**

   ```bash
   docker compose up --build
   ```

   O `entrypoint.sh` aplica as migrações do Alembic (`alembic upgrade head`) e inicia o
   servidor Uvicorn automaticamente.

4. **Acesse a aplicação:**

   - API: <http://localhost:8000>
   - Documentação (Swagger): <http://localhost:8000/docs>
   - Documentação (ReDoc): <http://localhost:8000/redoc>

### Comandos úteis

```bash
docker compose up -d --build   # Sobe em segundo plano
docker compose logs -f         # Acompanha os logs
docker compose down            # Para e remove os containers
docker compose down -v         # Para e remove também o volume do banco
```

---

## 💻 Rodando localmente (branch `dev`)

> ⚠️ **Use a branch `dev` para o desenvolvimento local.** Ela utiliza **SQLite** (via `aiosqlite`),
> dispensando a configuração de um servidor PostgreSQL e dos arquivos Docker.

### Pré-requisitos
- [Python 3.14+](https://www.python.org/downloads/)
- [Poetry](https://python-poetry.org/docs/#installation)

### Passo a passo

1. **Clone o repositório e mude para a branch `dev`:**

   ```bash
   git clone <url-do-repositorio> fast_pulse
   cd fast_pulse
   git checkout dev
   ```

2. **Instale as dependências** (Poetry cria o ambiente virtual automaticamente):

   ```bash
   poetry install
   ```

3. **Crie o arquivo `.env`** na raiz com uma configuração voltada para SQLite:

   ```env
   DATABASE_URL="sqlite+aiosqlite:///database.db"
   SECRET_KEY="troque-por-uma-chave-segura"
   ALGORITHM="HS256"
   ACCESS_TOKEN_EXPIRE_MINUTES=30
   ```

4. **Aplique as migrações do banco:**

   ```bash
   poetry run alembic upgrade head
   ```

5. **Inicie o servidor de desenvolvimento** (com *hot reload*):

   ```bash
   poetry run task run
   ```

   > O comando `task run` equivale a `fastapi dev fast_pulse/app.py`.

6. **Acesse a aplicação:**

   - API: <http://localhost:8000>
   - Documentação (Swagger): <http://localhost:8000/docs>
   - Documentação (ReDoc): <http://localhost:8000/redoc>

---

## 📡 Endpoints da API

### Raiz
| Método | Rota | Descrição                |
| ------ | ---- | ------------------------ |
| `GET`  | `/`  | Health check / mensagem  |

### Usuários — `/users`
| Método   | Rota                | Descrição                       |
| -------- | ------------------- | ------------------------------- |
| `POST`   | `/users/`           | Cria um novo usuário            |
| `GET`    | `/users/`           | Lista os usuários               |
| `GET`    | `/users/{user_id}`  | Busca um usuário pelo id        |
| `PUT`    | `/users/{user_id}`  | Atualiza um usuário             |
| `DELETE` | `/users/{user_id}`  | Remove um usuário               |

### Autenticação — `/auth`
| Método | Rota                   | Descrição                          |
| ------ | ---------------------- | ---------------------------------- |
| `POST` | `/auth/token`          | Gera o token de acesso (login)     |
| `POST` | `/auth/refresh_token`  | Renova o token de acesso           |

### Tarefas — `/todos`
| Método   | Rota                | Descrição                       |
| -------- | ------------------- | ------------------------------- |
| `POST`   | `/todos/`           | Cria uma nova tarefa            |
| `GET`    | `/todos/`           | Lista as tarefas do usuário     |
| `PATCH`  | `/todos/{todo_id}`  | Atualiza parcialmente a tarefa  |
| `DELETE` | `/todos/{todo_id}`  | Remove uma tarefa               |

> 🔎 A documentação completa e interativa está sempre disponível em `/docs`.

---

## 🧪 Testes e qualidade de código

O projeto usa **Taskipy** para padronizar os comandos. Todas as tarefas são executadas
com `poetry run task <nome>`:

| Comando                  | Descrição                                                    |
| ------------------------ | ------------------------------------------------------------ |
| `poetry run task run`    | Inicia o servidor em modo desenvolvimento                    |
| `poetry run task lint`   | Verifica o código com Ruff (precedido por `typos`)           |
| `poetry run task format` | Formata o código com Ruff (corrige problemas de lint antes)  |
| `poetry run task test`   | Roda a suíte de testes com cobertura (lint roda antes)       |

```bash
# Executa toda a suíte de testes com relatório de cobertura
poetry run task test
```

Após os testes, um relatório de cobertura em HTML é gerado em `htmlcov/index.html`.

---

## 📄 Licença

Distribuído sob a licença **MIT**. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

<div align="center">
Feito com ⚡ e FastAPI
</div>
