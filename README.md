# Projeto - Recicla Aqui

Aplicação REST API desenvolvida em Java 21 com Spring Boot, containerizada com Docker e com pipeline CI/CD automatizado via GitHub Actions.

---

## Como executar localmente com Docker

### Pré-requisitos

- [Docker](https://www.docker.com/) instalado
- [Git](https://git-scm.com/) instalado

### Passo a passo

1. Clone o repositório:

```sh
git clone https://github.com/seu-usuario/simple-api-java.git
cd simple-api-java
```

2. Suba a aplicação com Docker Compose:

```sh
docker compose up --build
```

3. Acesse a API:

```
http://localhost:8080
```

4. Acesse a documentação Swagger:

```
http://localhost:8080/swagger-ui/index.html
```

> Para encerrar os containers, utilize `docker compose down`.

---

## Pipeline CI/CD

### Ferramenta utilizada

**GitHub Actions** — ferramenta de automação integrada ao GitHub, configurada via arquivos `.yml` na pasta `.github/workflows/`.

### Etapas do pipeline

```
Push na branch → Build → Testes → Deploy Staging → Deploy Produção
```

| Etapa | Descrição |
|---|---|
| **Build** | Compila o código e gera o artefato `.jar` com Maven |
| **Testes** | Executa os testes unitários com `./mvnw test` |
| **Deploy Staging** | Deploy automático no ambiente de homologação |
| **Deploy Produção** | Deploy automático no ambiente de produção após aprovação |

### Funcionamento

- O pipeline é disparado automaticamente a cada `push` ou `pull request`
- Se o **build ou testes falharem**, o pipeline é interrompido e o deploy não acontece
- O deploy em **staging** ocorre automaticamente após os testes passarem
- O deploy em **produção** ocorre após validação no ambiente de staging

---

## Containerização

### Dockerfile

```dockerfile
# ---- Etapa 1: Build ----
FROM maven:3.9.8-eclipse-temurin-21 AS build

RUN mkdir /opt/app
COPY . /opt/app
WORKDIR /opt/app
RUN mvn clean package

# ---- Etapa 2: Runtime ----
FROM eclipse-temurin:21-jre-alpine

RUN mkdir /opt/app
COPY --from=build /opt/app/target/app.jar /opt/app/app.jar
WORKDIR /opt/app

ENV PROFILE=dev

EXPOSE 8080

ENTRYPOINT ["java", "-Dspring.profiles.active=${PROFILE}", "-jar", "app.jar"]
```

### Estratégias adotadas

**Multi-stage build**
O Dockerfile possui duas etapas separadas: uma para compilar o projeto com Maven e outra apenas para executar. Isso garante que ferramentas de build não vão para a imagem final, reduzindo seu tamanho e superfície de ataque.

**Imagem base leve (`eclipse-temurin:21-jre-alpine`)**
- `JRE` ao invés de `JDK`: apenas o runtime necessário para executar, sem compilador ou ferramentas de desenvolvimento
- `Alpine`: distribuição Linux minimalista (~7MB), reduzindo drasticamente o tamanho da imagem final
- `eclipse-temurin`: substituta oficial da imagem `openjdk`, descontinuada no Docker Hub

**Variável de ambiente para perfil**
O `PROFILE` é configurado via variável de ambiente, permitindo que a mesma imagem rode em diferentes ambientes (dev, staging, produção) sem alteração no código.

### docker-compose.yml

```yaml
services:
  db:
    container_name: mysql
    image: "mysql"
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=root_pass

  api:
    build: .
    ports:
      - "8080:8080"
    environment:
      - PROFILE=dev
      - DATABASE_URL=jdbc:mysql://db:3306/api?createDatabaseIfNotExist=true
      - DATABASE_USER=root
      - DATABASE_PWD=root_pass
```

### Recursos utilizados

| Recurso | Aplicação |
|---|---|
| **Variáveis de ambiente** | Credenciais do banco, perfil da aplicação e URL de conexão configurados via `environment` |
| **Redes** | O Docker Compose cria uma rede interna automaticamente, permitindo que o serviço `api` se comunique com o `db` pelo nome do serviço |
| **Múltiplos serviços** | Aplicação e banco de dados orquestrados juntos, subindo com um único comando |

---

### Swagger (documentação da API)
- `http://localhost:8080/swagger-ui/index.html`

---

## Tecnologias utilizadas

| Tecnologia | Finalidade |
|---|---|
| **Java 21** | Linguagem principal da aplicação |
| **Spring Boot** | Framework para criação da API REST |
| **Maven** | Gerenciamento de dependências e build |
| **MySQL** | Banco de dados relacional |
| **Docker** | Containerização da aplicação |
| **Docker Compose** | Orquestração local dos serviços |
| **GitHub Actions** | Pipeline de CI/CD automatizado |
| **Swagger / OpenAPI** | Documentação interativa da API |
| **eclipse-temurin (Alpine)** | Imagem base leve para o container de produção |