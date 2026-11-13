# API Python para demonstrar DevOps

RepositÃ³rio didÃ¡tico para apresentar, em uma Ãºnica aplicaÃ§Ã£o, trÃªs ideias que se conectam:

1. uma API HTTP escrita em Python;
2. integraÃ§Ã£o contÃ­nua com GitHub Actions;
3. execuÃ§Ã£o reproduzÃ­vel com Docker.

O projeto utiliza FastAPI, pytest, Ruff, GitHub Actions e Docker. Os dados ficam em memÃ³ria para manter a atenÃ§Ã£o da aula no fluxo DevOps, sem introduzir banco de dados.

## O que os estudantes devem observar

- O cÃ³digo funciona localmente e no runner do GitHub.
- A mesma suÃ­te de testes Ã© executada pelo desenvolvedor e pelo pipeline.
- O Dockerfile descreve o ambiente necessÃ¡rio para executar a API.
- O pipeline sÃ³ testa o container depois que qualidade e testes foram aprovados.
- Logs, cobertura e relatÃ³rios funcionam como evidÃªncias, nÃ£o apenas como mensagens coloridas.

## Estrutura do repositÃ³rio

```text
.
â”œâ”€â”€ .github/workflows/ci.yml   # pipeline de integraÃ§Ã£o contÃ­nua
â”œâ”€â”€ app/
â”‚   â”œâ”€â”€ __init__.py
â”‚   â””â”€â”€ main.py                # API FastAPI
â”œâ”€â”€ docs/
â”‚   â”œâ”€â”€ API.md
â”‚   â”œâ”€â”€ DOCKER.md
â”‚   â”œâ”€â”€ GITHUB-ACTIONS.md
â”‚   â””â”€â”€ ROTEIRO-AULA.md
â”œâ”€â”€ tests/test_api.py          # testes automatizados
â”œâ”€â”€ .dockerignore
â”œâ”€â”€ .python-version
â”œâ”€â”€ compose.yaml
â”œâ”€â”€ Dockerfile
â”œâ”€â”€ pyproject.toml             # configuraÃ§Ã£o de pytest, cobertura e Ruff
â”œâ”€â”€ requirements.txt           # dependÃªncias de execuÃ§Ã£o
â””â”€â”€ requirements-dev.txt       # dependÃªncias de desenvolvimento e CI
```

## 1. Executar localmente

PrÃ©-requisitos: Git e Python 3.13.

### Linux e macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --requirement requirements-dev.txt
uvicorn app.main:app --reload
```

### Windows PowerShell

```powershell
py -3.13 -m venv .venv
.venv\Scripts\Activate.ps1
python -m pip install --requirement requirements-dev.txt
uvicorn app.main:app --reload
```

Acesse:

- API: <http://localhost:8000>
- documentaÃ§Ã£o Swagger: <http://localhost:8000/docs>
- verificaÃ§Ã£o de saÃºde: <http://localhost:8000/health>

## 2. Experimentar a API

```bash
curl http://localhost:8000/health

curl -X POST http://localhost:8000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Estudar integraÃ§Ã£o contÃ­nua"}'

curl http://localhost:8000/tasks

curl -X PATCH http://localhost:8000/tasks/1/done
```

Endpoints:

| MÃ©todo | Caminho | Objetivo | Resposta esperada |
|---|---|---|---|
| GET | `/` | Apresentar a API | `200` |
| GET | `/health` | Informar saÃºde e versÃ£o | `200` |
| GET | `/tasks` | Listar tarefas | `200` |
| POST | `/tasks` | Criar uma tarefa | `201` |
| PATCH | `/tasks/{id}/done` | Concluir uma tarefa | `200` ou `404` |

Veja a explicaÃ§Ã£o do cÃ³digo em [docs/API.md](docs/API.md).

## 3. Executar as verificaÃ§Ãµes locais

```bash
ruff check .
ruff format --check .
pytest --cov=app --cov-report=term-missing
```

Esses comandos sÃ£o deliberadamente iguais aos usados no GitHub Actions. Se uma verificaÃ§Ã£o falhar localmente, ela tambÃ©m deverÃ¡ falhar no pipeline.

Para aplicar a formataÃ§Ã£o automaticamente:

```bash
ruff format .
```

## 4. Publicar no GitHub e acionar a CI

O diretÃ³rio jÃ¡ Ã© inicializado como repositÃ³rio Git local. Crie um repositÃ³rio vazio no GitHub e execute:

```bash
git remote add origin https://github.com/SEU-USUARIO/devops-python-api.git
git push -u origin main
```

O arquivo `.github/workflows/ci.yml` dispara o pipeline em trÃªs situaÃ§Ãµes:

- `push` na branch `main`;
- `pull_request` destinado Ã  `main`;
- execuÃ§Ã£o manual pela aba **Actions**.

Na aba **Actions**, abra a execuÃ§Ã£o e faÃ§a a leitura nesta ordem:

1. workflow;
2. job;
3. primeiro step que falhou;
4. comando e mensagem que explicam a falha.

A explicaÃ§Ã£o detalhada do YAML estÃ¡ em [docs/GITHUB-ACTIONS.md](docs/GITHUB-ACTIONS.md).

## 5. Executar com Docker

```bash
docker build -t devops-python-api:v1 .
docker run --rm -p 8000:8000 --name devops-python-api devops-python-api:v1
```

Em outro terminal:

```bash
curl http://localhost:8000/health
docker ps
docker logs devops-python-api
```

Com Docker Compose:

```bash
docker compose up --build
docker compose down
```

Veja a leitura detalhada do Dockerfile em [docs/DOCKER.md](docs/DOCKER.md).

## Falhas intencionais para demonstrar

### Falha de teste

Em `app/main.py`, altere temporariamente a versÃ£o do health check para `2.0.0`. O teste espera `1.1.0`, entÃ£o o pytest e o pipeline devem falhar.

### Falha de qualidade

Adicione um import nÃ£o utilizado no inÃ­cio de `app/main.py`:

```python
import os
```

O step `ruff check .` localizarÃ¡ o problema.

### Falha de rede no container

No `Dockerfile`, troque `--host 0.0.0.0` por `--host 127.0.0.1`. O processo iniciarÃ¡, mas o serviÃ§o nÃ£o ficarÃ¡ acessÃ­vel pela porta publicada no host.

## PrÃ³ximos desafios

- Adicionar um endpoint `DELETE /tasks/{id}` e seus testes.
- Executar os testes em uma matriz com Python 3.12, 3.13 e 3.14.
- Publicar a imagem em um registry somente apÃ³s a aprovaÃ§Ã£o dos testes.
- Substituir o armazenamento em memÃ³ria por um banco de dados.

## ReferÃªncias oficiais

- [GitHub Actions: criar e testar Python](https://docs.github.com/pt/actions/tutorials/build-and-test-code/building-and-testing-python)
- [actions/checkout](https://github.com/actions/checkout)
- [actions/setup-python](https://github.com/actions/setup-python)
- [actions/upload-artifact](https://github.com/actions/upload-artifact)
- [FastAPI](https://fastapi.tiangolo.com/)
- [Docker para Python](https://docs.docker.com/language/python/)

---

## Arquitetura CI/CD na AWS

Este projeto implementa integraÃ§Ã£o e entrega contÃ­nuas usando GitHub Actions, Docker, Amazon ECR, Amazon EC2 e AWS Systems Manager.

### Fluxo automatizado

1. O desenvolvedor cria uma branch e abre um Pull Request para main.
2. O GitHub Actions executa verificaÃ§Ãµes Ruff, testes pytest e cobertura.
3. ApÃ³s aprovaÃ§Ã£o e merge, o workflow deploy.yml Ã© iniciado.
4. Os testes sÃ£o executados antes de qualquer publicaÃ§Ã£o.
5. Uma imagem Docker Ã© construÃ­da e identificada pelo SHA do commit.
6. A imagem Ã© publicada no Amazon ECR privado.
7. O AWS Systems Manager envia comandos Ã  EC2.
8. A EC2 baixa a imagem, substitui o contÃªiner e verifica /health.

### Infraestrutura

| Componente | ConfiguraÃ§Ã£o |
|---|---|
| RegiÃ£o AWS | us-east-1 |
| Amazon ECR | devops-python-api |
| Amazon EC2 | Amazon Linux 2023 |
| Tipo de instÃ¢ncia | t3.micro |
| Porta HTTP | 8000 |
| Gerenciamento remoto | AWS Systems Manager |
| AutenticaÃ§Ã£o EC2 | IAM Role LabRole |

### Workflows

**IntegraÃ§Ã£o contÃ­nua:** .github/workflows/ci.yml

ResponsÃ¡vel por validar Pull Requests com anÃ¡lise estÃ¡tica, formataÃ§Ã£o, testes automatizados, cobertura e teste do contÃªiner.

**Entrega contÃ­nua:** .github/workflows/deploy.yml

Executado apÃ³s atualizaÃ§Ãµes da branch main. Realiza testes, construÃ§Ã£o da imagem, publicaÃ§Ã£o no ECR e implantaÃ§Ã£o automÃ¡tica na EC2 via SSM.

### Acesso Ã  aplicaÃ§Ã£o

Endpoint: http://54.210.208.229:8000/health

DocumentaÃ§Ã£o Swagger: http://54.210.208.229:8000/docs

O IPv4 pÃºblico pode mudar quando a instÃ¢ncia Ã© interrompida e reiniciada.

### EvidÃªncias da automaÃ§Ã£o

Primeiro deploy:
https://github.com/Husty-09/devops_python_api/actions/runs/43328734810

Segundo deploy:
https://github.com/Husty-09/devops_python_api/actions/runs/43334790582

CI da atualizaÃ§Ã£o:
https://github.com/Husty-09/devops_python_api/actions/runs/43334592186

### Pull Requests

PR #1 - Testes:
https://github.com/Husty-09/devops_python_api/pull/1

PR #2 - Deploy AWS:
https://github.com/Husty-09/devops_python_api/pull/2

PR #3 - AtualizaÃ§Ã£o 1.1.0:
https://github.com/Husty-09/devops_python_api/pull/3

### Verificar a versÃ£o implantada

Execute:

curl http://IP_PUBLICO:8000/health

A resposta esperada apÃ³s o segundo deploy Ã©:

{"status":"ok","version":"1.1.0"}

### LimitaÃ§Ãµes e seguranÃ§a

- As tarefas sÃ£o armazenadas em memÃ³ria e desaparecem quando o contÃªiner Ã© substituÃ­do.
- O laboratÃ³rio Vocareum utiliza credenciais AWS temporÃ¡rias.
- A automaÃ§Ã£o depende da validade dessas credenciais e da disponibilidade da EC2.
- As credenciais sÃ£o configuradas nos GitHub Actions Secrets.
- A aplicaÃ§Ã£o Ã© exposta via HTTP para demonstraÃ§Ã£o acadÃªmica.
- O deploy atual nÃ£o implementa rollback automÃ¡tico.
- Os recursos AWS devem ser interrompidos ou excluÃ­dos apÃ³s a avaliaÃ§Ã£o.

## EvidÃªncias da implementaÃ§Ã£o CI/CD

As imagens abaixo demonstram a integraÃ§Ã£o contÃ­nua, os Pull Requests e a implantaÃ§Ã£o automÃ¡tica na AWS.

### 1. IntegraÃ§Ã£o contÃ­nua
![CI aprovada](evidencias/01-ci.png)

### 2. Pull Requests e merges
![Pull Requests](evidencias/02-merges.png)

### 3. Deploys automÃ¡ticos
![Deploys](evidencias/03-deploys.png)

### 4. Imagens Docker no ECR
![Docker no ECR](evidencias/04-docker.png)

### 5. Primeira versÃ£o - 1.0.0
![API v1](evidencias/05-V1.jpg)

### 6. Segunda versÃ£o - 1.1.0
![API v2](evidencias/06-V2.jpg)

### Resultado

A API foi implantada inicialmente na versÃ£o 1.0.0.

ApÃ³s a aprovaÃ§Ã£o dos testes e o merge na main,
o GitHub Actions publicou uma nova imagem no Amazon ECR
e atualizou automaticamente a EC2 para a versÃ£o 1.1.0.

