# Projeto DevOps — Pipeline CI/CD com GitHub Actions, Docker e AWS

**Disciplina:** Integração DevOps  
**Projeto:** devops-python-api  
**Repositório:** [Husty-09/devops_python_api](https://github.com/Husty-09/devops_python_api)  
**Versão atual da aplicação:** 1.1.0

---

## 1. Sobre o projeto

Este projeto tem como objetivo implementar uma pipeline completa de **Integração Contínua (CI) e Entrega Contínua (CD)** para uma API desenvolvida em Python.

A aplicação utiliza FastAPI e permite criar, listar e concluir tarefas por meio de requisições HTTP.

A solução automatiza as etapas de testes, construção de imagens Docker, publicação no Amazon Elastic Container Registry (ECR) e implantação em uma instância Amazon EC2.

O principal objetivo é garantir que alterações aprovadas e incorporadas à branch `main` sejam testadas e implantadas automaticamente, sem necessidade de executar comandos manuais de deploy no servidor.

## 2. Tecnologias utilizadas

| Tecnologia | Finalidade |
|---|---|
| Python 3.13 | Linguagem utilizada na aplicação |
| FastAPI | Desenvolvimento da API REST |
| Uvicorn | Servidor de execução da API |
| Pytest | Execução dos testes automatizados |
| Ruff | Análise estática e formatação do código |
| Git e GitHub | Versionamento e colaboração |
| GitHub Actions | Automação das pipelines CI/CD |
| Docker | Containerização da aplicação |
| Amazon ECR | Armazenamento privado das imagens Docker |
| Amazon EC2 | Hospedagem da aplicação |
| AWS IAM | Gerenciamento de permissões |
| AWS Systems Manager | Execução remota e automatizada do deploy |
| Vocareum | Ambiente educacional utilizado para acessar a AWS |

## 3. Arquitetura da solução

A aplicação utiliza a seguinte arquitetura de integração e entrega contínua:

```text
       DESENVOLVEDOR
             |
             v
     Branch de desenvolvimento
             |
             v
       Pull Request
             |
             v
      GITHUB ACTIONS
      Integração contínua
             |
       +-----+------+
       |            |
    Ruff Check    Pytest
       |            |
       +-----+------+
             |
       Testes aprovados
             |
             v
       Merge na main
             |
             v
       GITHUB ACTIONS
       Entrega contínua
             |
             v
       Docker Build
             |
             v
         AMAZON ECR
       Imagem publicada
             |
             v
     AWS Systems Manager
             |
             v
         AMAZON EC2
             |
             v
       Docker Pull
             |
             v
      Substituição do
      contêiner anterior
             |
             v
       API ATUALIZADA
          Porta 8000
```

O processo é iniciado automaticamente após alterações na branch principal do repositório.

## 4. Estrutura do repositório

```text
devops_python_api/
|
|-- .github/
|   |-- workflows/
|       |-- ci.yml
|       |-- deploy.yml
|
|-- app/
|   |-- __init__.py
|   |-- main.py
|
|-- tests/
|   |-- test_api.py
|
|-- docs/
|
|-- evidencias/
|   |-- 01-ci.png
|   |-- 02-merges.png
|   |-- 03-deploys.png
|   |-- 04-docker.png
|   |-- 05-V1.jpg
|   |-- 06-V2.jpg
|
|-- Dockerfile
|-- .dockerignore
|-- compose.yaml
|-- requirements.txt
|-- requirements-dev.txt
|-- pyproject.toml
|-- .python-version
|-- README.md
```

## 5. Funcionalidades da API

A API disponibiliza endpoints para gerenciamento de tarefas e verificação da aplicação.

| Método | Endpoint | Descrição |
|---|---|---|
| GET | `/` | Retorna informações da aplicação |
| GET | `/health` | Verifica o status e a versão |
| GET | `/tasks` | Lista as tarefas cadastradas |
| POST | `/tasks` | Cria uma nova tarefa |
| PATCH | `/tasks/{id}/done` | Marca uma tarefa como concluída |

### Exemplo de resposta do endpoint /health

```json
{
  "status": "ok",
  "version": "1.1.0"
}
```

O endpoint `/health` também é utilizado pela pipeline e pelo Docker para verificar o funcionamento da aplicação.

## 6. Execução local

### Pré-requisitos

- Git
- Python 3.13
- pip

### Clonar o repositório

```bash
git clone https://github.com/Husty-09/devops_python_api.git
cd devops_python_api
```

### Criar o ambiente virtual

No Windows PowerShell:

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

No Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Instalar as dependências

```bash
python -m pip install -r requirements-dev.txt
```

### Iniciar a aplicação

```bash
uvicorn app.main:app --reload
```

A aplicação ficará disponível em:

- API: http://localhost:8000
- Documentação Swagger: http://localhost:8000/docs
- Health Check: http://localhost:8000/health

## 7. Testes automatizados

O projeto utiliza Pytest para verificar o comportamento dos endpoints.

Os testes validam:

- Resposta do endpoint de saúde e versão.
- Listagem inicial de tarefas.
- Criação de novas tarefas.
- Validação de títulos inválidos.
- Conclusão de tarefas.
- Tratamento de tarefas inexistentes.
- Resposta do endpoint principal.

### Executar os testes

```bash
python -m pytest --cov=app --cov-report=term-missing
```

### Verificar a qualidade do código

```bash
python -m ruff check .
python -m ruff format --check .
```

Na validação local da versão 1.1.0, foram obtidos os seguintes resultados:

| Indicador | Resultado |
|---|---|
| Testes executados | 7 |
| Testes aprovados | 7 |
| Cobertura de código | 96,97% |
| Cobertura mínima configurada | 90% |
| Análise estática | Aprovada |
| Verificação de formatação | Aprovada |

Essas verificações também são executadas automaticamente pelo GitHub Actions.

## 8. Containerização com Docker

A aplicação utiliza um Dockerfile baseado na imagem `python:3.13-slim`.

A imagem instala as dependências necessárias e executa a API utilizando Uvicorn na porta 8000.

### Construir a imagem localmente

```bash
docker build -t devops-python-api:local .
```

### Executar o contêiner

```bash
docker run -d \
  --name devops-python-api \
  -p 8000:8000 \
  devops-python-api:local
```

### Verificar o contêiner

```bash
docker ps
```

### Testar a API

```bash
curl http://localhost:8000/health
```

### Execução alternativa com Docker Compose

```bash
docker compose up --build
```

Para encerrar:

```bash
docker compose down
```

## 9. Pipeline de Integração Contínua (CI)

**Arquivo:** `.github/workflows/ci.yml`

A pipeline CI é executada automaticamente na abertura ou atualização de Pull Requests destinados à branch `main`.

O workflow realiza as seguintes operações:

1. Baixa o código do repositório.
2. Configura o ambiente Python.
3. Instala as dependências do projeto.
4. Executa análise estática com Ruff.
5. Verifica a formatação dos arquivos.
6. Executa os testes com Pytest.
7. Verifica a cobertura de testes.
8. Constrói uma imagem Docker para teste.
9. Inicializa o contêiner e verifica o endpoint `/health`.
10. Disponibiliza relatórios da execução.

O processo de publicação e deploy não depende apenas da abertura do Pull Request.

**Uma alteração só deve ser incorporada à branch principal após a aprovação dos testes.**

Caso os testes falhem, o job de deploy do workflow de entrega contínua não será executado.

### Evidência da integração contínua

![Pipeline de integração contínua](evidencias/01-ci.png)

Execução da CI na atualização para a versão 1.1.0:

[Visualizar execução no GitHub Actions](https://github.com/Husty-09/devops_python_api/actions/runs/43334592186)

## 10. Versionamento e Pull Requests

O desenvolvimento foi realizado utilizando branches, commits e Pull Requests.

Esse procedimento permite validar alterações antes que sejam incorporadas à branch principal.

### Evidência dos Pull Requests e merges

![Histórico de Pull Requests e merges](evidencias/02-merges.png)

### Pull Requests utilizados

| Pull Request | Descrição |
|---|---|
| [PR #1](https://github.com/Husty-09/devops_python_api/pull/1) | Adição de teste automatizado |
| [PR #2](https://github.com/Husty-09/devops_python_api/pull/2) | Implementação do deploy automático na AWS |
| [PR #3](https://github.com/Husty-09/devops_python_api/pull/3) | Atualização da aplicação para a versão 1.1.0 |
| [PR #4](https://github.com/Husty-09/devops_python_api/pull/4) | Documentação técnica e evidências |

## 11. Publicação de imagens no Amazon ECR

O Amazon Elastic Container Registry é utilizado para armazenar as imagens Docker em um repositório privado.

**Configurações utilizadas:**

| Propriedade | Valor |
|---|---|
| Serviço | Amazon ECR |
| Região | us-east-1 |
| Tipo de repositório | Privado |
| Nome | devops-python-api |
| Registro | 502001549054.dkr.ecr.us-east-1.amazonaws.com |
| Identificação das imagens | SHA do commit |
| Mutabilidade das tags | Immutable |

### URI do repositório

```text
502001549054.dkr.ecr.us-east-1.amazonaws.com/devops-python-api
```

As imagens são construídas e publicadas automaticamente pelo GitHub Actions após a validação dos testes.

Cada imagem recebe uma tag associada ao commit, permitindo identificar a versão exata do código utilizado na implantação.

### Imagens dos dois deploys

| Versão | Tag do commit |
|---|---|
| 1.0.0 | `de2dc429a12c2d379866556e8ca4147cf8991be9` |
| 1.1.0 | `6571dc71b8b8a6e0b9d4bf383756dd2ae8a179f0` |

### Evidência do Amazon ECR

![Imagens Docker publicadas no Amazon ECR](evidencias/04-docker.png)

## 12. Implantação na Amazon EC2

A aplicação é hospedada em uma única instância Amazon EC2.

### Configuração da instância

| Configuração | Valor |
|---|---|
| Nome | devops-python-api-ec2 |
| Sistema operacional | Amazon Linux 2023 |
| Tipo | t3.micro |
| Região | us-east-1 |
| Aplicação | FastAPI |
| Contêiner | Docker |
| Porta HTTP | 8000 |
| IAM Role | LabRole |
| Gerenciamento | AWS Systems Manager |

A instância possui Docker instalado e utiliza uma IAM Role para autenticação na AWS.

O acesso administrativo e a execução de comandos remotos podem ser realizados por meio dos serviços de gerenciamento disponibilizados pela AWS.

### Funcionamento do deploy

O GitHub Actions utiliza o AWS Systems Manager para enviar comandos à instância EC2.

Durante a atualização, a instância:

1. Autentica o Docker no Amazon ECR.
2. Baixa a imagem correspondente ao commit.
3. Remove o contêiner anterior.
4. Inicializa um novo contêiner com a imagem atualizada.
5. Publica a aplicação na porta 8000.
6. Verifica a resposta do endpoint `/health`.
7. Retorna o resultado da execução ao GitHub Actions.

Esse processo ocorre automaticamente, sem necessidade de acessar manualmente a EC2 após cada alteração.

## 13. Pipeline de Entrega Contínua (CD)

**Arquivo:** `.github/workflows/deploy.yml`

O workflow é acionado automaticamente após um push na branch `main`, incluindo os merges realizados por Pull Requests.

A pipeline possui dois jobs principais.

### Job 1 — Validar aplicação

Realiza as verificações de qualidade e executa os testes automatizados.

### Job 2 — Publicar ECR e atualizar EC2

Esse job depende da aprovação do primeiro.

Suas responsabilidades são:

- Autenticar na AWS.
- Construir a imagem Docker.
- Publicar a imagem no Amazon ECR.
- Enviar comandos pelo Systems Manager.
- Baixar a imagem na EC2.
- Substituir o contêiner anterior.
- Verificar a aplicação atualizada.

A autenticação do GitHub Actions é realizada utilizando credenciais temporárias configuradas nos GitHub Actions Secrets.

### Evidência dos deploys

![Execuções automáticas de deploy](evidencias/03-deploys.png)

### Histórico de execução

**Primeira implantação — versão 1.0.0**

[Visualizar primeiro deploy](https://github.com/Husty-09/devops_python_api/actions/runs/43328734810)

**Segunda implantação — versão 1.1.0**

[Visualizar segundo deploy](https://github.com/Husty-09/devops_python_api/actions/runs/43334790582)

Os dois workflows de implantação foram concluídos com sucesso.

## 14. Demonstração de atualização automática

Para comprovar o funcionamento da entrega contínua, foram realizadas duas implantações da aplicação.

### Versão inicial — 1.0.0

A primeira versão foi publicada automaticamente após a implementação da pipeline.

O endpoint `/health` retornava:

```json
{
  "status": "ok",
  "version": "1.0.0"
}
```

**Evidência da primeira versão:**

![Primeira versão da API](evidencias/05-V1.jpg)

### Segunda versão — 1.1.0

Foi criada uma nova branch de desenvolvimento para alterar a versão da API de `1.0.0` para `1.1.0`.

Também foram atualizados os testes automatizados para validar a nova versão.

Após os testes serem aprovados pelo GitHub Actions, o Pull Request foi incorporado à branch `main`.

Isso iniciou automaticamente uma nova execução do workflow de entrega contínua.

A pipeline construiu uma nova imagem Docker, publicou a imagem no ECR e atualizou o contêiner executado na EC2.

O endpoint `/health` passou a retornar:

```json
{
  "status": "ok",
  "version": "1.1.0"
}
```

**Evidência da segunda versão:**

![Segunda versão da API](evidencias/06-V2.jpg)

A mudança entre as versões comprova a atualização da aplicação por meio da pipeline automatizada, sem execução manual dos comandos de implantação.

## 15. Acesso à aplicação publicada

A API foi disponibilizada utilizando o endereço IPv4 público da instância EC2 e a porta 8000.

**Endpoint de verificação:**

http://54.210.208.229:8000/health

**Documentação interativa Swagger:**

http://54.210.208.229:8000/docs

Para verificar a versão implantada, execute:

```bash
curl http://54.210.208.229:8000/health
```

**Resposta esperada da versão 1.1.0:**

```json
{
  "status": "ok",
  "version": "1.1.0"
}
```

**Observação:** como a aplicação utiliza um ambiente AWS educacional, o endereço IPv4 público pode mudar após a reinicialização da instância. A disponibilidade também depende de o laboratório e a EC2 estarem ativos.

Durante os testes, a aplicação foi acessível por conexão móvel 4G, enquanto a rede da instituição apresentou restrições de conectividade para a porta utilizada.

## 16. Segurança e limitações

A solução foi desenvolvida para fins acadêmicos e possui as seguintes características e limitações:

- **Credenciais:** as credenciais AWS são armazenadas nos GitHub Actions Secrets e não diretamente nos arquivos versionados.
- **IAM Role:** a EC2 utiliza a função `LabRole` fornecida pelo ambiente educacional.
- **Acesso ao ECR:** as imagens são armazenadas em um repositório privado.
- **Credenciais temporárias:** o Vocareum fornece credenciais com prazo de validade, exigindo renovação para novas implantações.
- **Dados em memória:** as tarefas cadastradas não são persistidas e podem ser perdidas durante a substituição do contêiner.
- **Disponibilidade:** a atualização substitui o contêiner atual, podendo causar uma breve interrupção.
- **Rollback:** a pipeline não possui um mecanismo automático de retorno à versão anterior.
- **HTTP:** a API foi publicada sem HTTPS, exclusivamente para demonstração acadêmica.
- **Recursos AWS:** a instância e os demais recursos devem ser interrompidos ou removidos após a conclusão e avaliação do trabalho, evitando cobranças desnecessárias.

Para um ambiente de produção, seriam recomendados HTTPS, armazenamento persistente, autenticação de longa duração baseada em funções IAM e OIDC, políticas de acesso mais restritivas, monitoramento e estratégias de rollback.

## 17. Resultado final

O projeto demonstrou a implementação de uma pipeline completa de CI/CD, integrando as ferramentas utilizadas na disciplina de Integração DevOps.

Foram realizados testes automatizados, análise de qualidade, containerização, publicação de imagens no Amazon ECR e implantação automática em uma instância EC2.

A atualização da versão `1.0.0` para `1.1.0` comprovou que alterações aprovadas e incorporadas à branch `main` podem ser publicadas automaticamente na infraestrutura AWS.

**Fluxo final implementado:**

```text
Alteração no código
       |
       v
Nova branch
       |
       v
Pull Request
       |
       v
Testes automatizados
       |
       v
Merge na main
       |
       v
Build Docker
       |
       v
Publicação no ECR
       |
       v
Deploy via SSM na EC2
       |
       v
Aplicação atualizada
```

O resultado demonstra a aplicação integrada de versionamento, testes automatizados, containerização, integração contínua e entrega contínua.

---

**Repositório GitHub:**  
https://github.com/Husty-09/devops_python_api

**Disciplina:** Integração DevOps  
**Trabalho final:** Implementação de uma Pipeline CI/CD com GitHub Actions, Docker, Amazon ECR e EC2
