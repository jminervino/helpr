<div align="center">

# Helpr — Backend

**API REST para sistema de gerenciamento de chamados técnicos**
Construída com Spring Boot 2, Spring Security e JWT

[![Java](https://img.shields.io/badge/Java-11-ED8B00?logo=openjdk&logoColor=white)](https://openjdk.org/)
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-2.3.12-6DB33F?logo=spring-boot&logoColor=white)](https://spring.io/projects/spring-boot)
[![Spring Security](https://img.shields.io/badge/Spring_Security-JWT-6DB33F?logo=spring&logoColor=white)](https://spring.io/projects/spring-security)
[![MySQL](https://img.shields.io/badge/MySQL-8-4479A1?logo=mysql&logoColor=white)](https://www.mysql.com/)
[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

</div>

---

## Sumário

- [Sobre o Projeto](#sobre-o-projeto)
- [Funcionalidades](#funcionalidades)
- [Stack e Dependências](#stack-e-dependências)
- [Arquitetura](#arquitetura)
- [Estrutura de Pacotes](#estrutura-de-pacotes)
- [Modelo de Dados](#modelo-de-dados)
- [Endpoints da API](#endpoints-da-api)
- [Autenticação e Segurança](#autenticação-e-segurança)
- [Pré-requisitos](#pré-requisitos)
- [Como Rodar](#como-rodar)
- [Perfis de Configuração](#perfis-de-configuração)
- [Tratamento de Erros](#tratamento-de-erros)
- [Dados de Teste](#dados-de-teste)

---

## Sobre o Projeto

O **Helpr** é uma API REST para gestão de chamados técnicos, desenvolvida como POC (Proof of Concept) durante o bootcamp BCW18 da Soul Code Academy. A API gerencia técnicos, clientes e chamados de suporte com controle de acesso baseado em papéis via JWT.

Este repositório contém o **backend Spring Boot**, consumido pelo [frontend Angular](../).

---

## Funcionalidades

- Autenticação stateless com JWT (HS512, 24h de expiração)
- Controle de acesso por papel: ADMIN, TECNICO, CLIENTE
- CRUD completo de técnicos, clientes e chamados
- Validação de CPF brasileiro (anotação `@CPF`)
- Detecção de duplicidade de CPF e e-mail
- Relatórios de chamados por técnico e por cliente
- Tratamento centralizado de exceções com respostas padronizadas
- Suporte a dois perfis: `test` (H2 in-memory) e `dev` (MySQL)

---

## Stack e Dependências

| Dependência | Versão | Uso |
|------------|--------|-----|
| Java | 11 | Linguagem |
| Spring Boot | 2.3.12 | Framework principal |
| Spring Data JPA | — | ORM / persistência |
| Spring Security | — | Autenticação e autorização |
| Spring Validation | — | Validação de DTOs (Bean Validation) |
| JJWT | 0.7.0 | Geração e validação de tokens JWT |
| MySQL Connector | runtime | Driver de banco de dados (produção) |
| H2 Database | runtime | Banco em memória (testes) |
| Spring DevTools | dev | Hot reload |

---

## Arquitetura

A aplicação segue uma arquitetura em camadas clássica do Spring:

```
┌─────────────────────────────────────────────────────┐
│                  HTTP Request                        │
└────────────────────────┬────────────────────────────┘
                         │
              ┌──────────▼──────────┐
              │   Security Filters  │
              │  JWTAuthentication  │
              │  JWTAuthorization   │
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │      Resources      │  ← REST Controllers
              │  (TecnicoResource,  │
              │  ClienteResource,   │
              │  ChamadoResource)   │
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │      Services       │  ← Regras de negócio
              │  (TecnicoService,   │
              │  ClienteService,    │
              │  ChamadoService)    │
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │    Repositories     │  ← Spring Data JPA
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │  MySQL / H2 Database│
              └─────────────────────┘
```

---

## Estrutura de Pacotes

```
com.api.helpr/
├── config/
│   ├── SecurityConfig.java       # Spring Security + CORS + JWT filters
│   ├── DevConfig.java            # Perfil dev: inicializa dados de teste
│   └── TestConfig.java           # Perfil test: H2 in-memory
├── domain/
│   ├── Pessoa.java               # Entidade base abstrata (herança)
│   ├── Tecnico.java              # Entidade técnico
│   ├── Cliente.java              # Entidade cliente
│   ├── Chamado.java              # Entidade chamado/ticket
│   ├── dtos/
│   │   ├── TecnicoDTO.java
│   │   ├── ClienteDTO.java
│   │   ├── ChamadoDTO.java
│   │   └── CredenciaisDTO.java   # Login request
│   └── enums/
│       ├── Perfil.java           # ADMIN(0), CLIENTE(1), TECNICO(2)
│       ├── Status.java           # ABERTO(0), ANDAMENTO(1), ENCERRADO(2)
│       └── Prioridade.java       # BAIXA(0), MEDIA(1), ALTA(2)
├── repositories/
│   ├── PessoaRepository.java     # findByCpf, findByEmail
│   ├── TecnicoRepository.java
│   ├── ClienteRepository.java
│   └── ChamadoRepository.java    # Queries nativas por técnico e cliente
├── resources/
│   ├── TecnicoResource.java      # /service/tecnicos
│   ├── ClienteResource.java      # /service/clientes
│   ├── ChamadoResource.java      # /service/chamados
│   └── exceptions/
│       ├── ResourceExceptionHandler.java
│       ├── StandardError.java
│       ├── ValidationError.java
│       └── FieldMessage.java
├── security/
│   ├── JWTUtil.java              # Geração e validação de tokens
│   ├── JWTAuthenticationFilter.java  # POST /login → gera token
│   ├── JWTAuthorizationFilter.java   # Valida token em cada requisição
│   └── UserSS.java               # Implementação de UserDetails
├── services/
│   ├── TecnicoService.java
│   ├── ClienteService.java
│   ├── ChamadoService.java
│   ├── UserDetailsServiceImpl.java
│   ├── DBService.java            # Seed de dados para perfil test
│   └── exceptions/
│       ├── ObjectNotFoundException.java
│       └── DataIntegrityViolationException.java
└── HelprApplication.java
```

---

## Modelo de Dados

### Diagrama de Entidades

```
┌──────────────────────────────────────────────────┐
│                    Pessoa (abstract)              │
│  id | nome | cpf | email | senha | perfils | ... │
└──────────┬───────────────────┬───────────────────┘
           │                   │
    ┌──────▼──────┐     ┌──────▼──────┐
    │   Tecnico   │     │   Cliente   │
    │  (ROLE_     │     │  (ROLE_     │
    │  TECNICO)   │     │  CLIENTE)   │
    └──────┬──────┘     └──────┬──────┘
           │  1             1  │
           │    ┌──────────┐   │
           └───▶│  Chamado │◀──┘
           N    │  id      │    N
                │  titulo  │
                │  status  │
                │prioridade│
                └──────────┘
```

### Enums

| Enum | Valores |
|------|---------|
| `Perfil` | `ADMIN(0)`, `CLIENTE(1)`, `TECNICO(2)` |
| `Status` | `ABERTO(0)`, `ANDAMENTO(1)`, `ENCERRADO(2)` |
| `Prioridade` | `BAIXA(0)`, `MEDIA(1)`, `ALTA(2)` |

> Os enums são armazenados como inteiros no banco de dados e serializados como inteiros na API.

---

## Endpoints da API

### Autenticação

| Método | Rota | Autenticação | Body | Resposta |
|--------|------|-------------|------|---------|
| `POST` | `/login` | Nenhuma | `CredenciaisDTO` | Header `Authorization: Bearer {token}` |

**Body de login:**
```json
{
  "email": "usuario@exemplo.com",
  "senha": "senha123"
}
```

---

### Técnicos — `/service/tecnicos`

| Método | Rota | Autenticação | Body | Resposta |
|--------|------|-------------|------|---------|
| `GET` | `/` | Nenhuma | — | `TecnicoDTO[]` |
| `GET` | `/{id}` | Nenhuma | — | `TecnicoDTO` |
| `POST` | `/` | `ROLE_ADMIN` | `TecnicoDTO` | `201 TecnicoDTO` |
| `PUT` | `/{id}` | `ROLE_ADMIN` | `TecnicoDTO` | `200 TecnicoDTO` |
| `DELETE` | `/{id}` | `ROLE_ADMIN` | — | `204 No Content` |

**TecnicoDTO:**
```json
{
  "id": 1,
  "nome": "João Silva",
  "cpf": "111.111.111-11",
  "email": "joao@helpr.com",
  "senha": "senha123",
  "perfils": [0, 2],
  "dataCriacao": "01/01/2024"
}
```

> `senha` é `WRITE_ONLY` — não retorna nas respostas.

---

### Clientes — `/service/clientes`

| Método | Rota | Autenticação | Body | Resposta |
|--------|------|-------------|------|---------|
| `GET` | `/` | Nenhuma | — | `ClienteDTO[]` |
| `GET` | `/{id}` | Nenhuma | — | `ClienteDTO` |
| `POST` | `/` | `ROLE_TECNICO` | `ClienteDTO` | `201 ClienteDTO` |
| `PUT` | `/{id}` | `ROLE_TECNICO` | `ClienteDTO` | `200 ClienteDTO` |
| `DELETE` | `/{id}` | `ROLE_TECNICO` | — | `204 No Content` |

---

### Chamados — `/service/chamados`

| Método | Rota | Autenticação | Body | Resposta |
|--------|------|-------------|------|---------|
| `GET` | `/` | Nenhuma | — | `ChamadoDTO[]` |
| `GET` | `/{id}` | Nenhuma | — | `ChamadoDTO` |
| `GET` | `/relatorios/cliente/{id}` | Nenhuma | — | `ChamadoDTO[]` (status=ABERTO) |
| `GET` | `/relatorios/tecnico/{id}` | Nenhuma | — | `ChamadoDTO[]` (status=ANDAMENTO) |
| `POST` | `/` | `ROLE_TECNICO` | `ChamadoDTO` | `201 ChamadoDTO` |
| `PUT` | `/{id}` | `ROLE_TECNICO` | `ChamadoDTO` | `200 ChamadoDTO` |

**ChamadoDTO:**
```json
{
  "id": 1,
  "dataAbertura": "24/02/2026",
  "dataFechamento": null,
  "prioridade": 2,
  "status": 0,
  "titulo": "Sistema fora do ar",
  "observacoes": "Usuário não consegue acessar o sistema desde as 09h.",
  "tecnico": 1,
  "cliente": 3,
  "nomeTecnico": "João Silva",
  "nomeCliente": "Maria Souza"
}
```

> Quando `status` é alterado para `2` (ENCERRADO), `dataFechamento` é preenchida automaticamente.

---

## Autenticação e Segurança

### Fluxo JWT

```
POST /login
  → JWTAuthenticationFilter.attemptAuthentication()
  → UserDetailsServiceImpl.loadUserByUsername()
  → BCrypt password check
  → JWTUtil.generateToken(email)
  → Response header: Authorization: Bearer {token}

Requisições subsequentes:
  → JWTAuthorizationFilter.doFilterInternal()
  → Extrai Bearer token do header
  → JWTUtil.tokenValido()
  → Carrega UserDetails e seta SecurityContext
```

### Configuração JWT

| Propriedade | Valor |
|------------|-------|
| Algoritmo | HS512 |
| Expiração | 86400000 ms (24 horas) |
| Secret | Configurável via `jwt.secret` |

### CORS

A API aceita requisições de qualquer origem com os métodos: `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS`.

> Em produção, configure `allowedOrigins` com o domínio do frontend.

---

## Pré-requisitos

- Java 11+
- Maven 3.6+
- MySQL 8 (para perfil `dev`)

---

## Como Rodar

### 1. Clone o repositório

```bash
git clone https://github.com/seu-usuario/helpr-Front.git
cd helpr-Front/helpr
```

### 2. Perfil de teste (H2 — sem banco externo)

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=test
```

A API sobe em `http://localhost:8080` com dados de seed automáticos.
Console H2 disponível em `http://localhost:8080/h2-console`.

### 3. Perfil de desenvolvimento (MySQL)

Configure as propriedades em `src/main/resources/application-dev.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/helpr_db
spring.datasource.username=seu_usuario
spring.datasource.password=sua_senha
```

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev
```

### 4. Build do JAR

```bash
./mvnw clean package -DskipTests
java -jar target/helpr-0.0.1-SNAPSHOT.jar
```

---

## Perfis de Configuração

| Perfil | Banco | Schema | Seed |
|--------|-------|--------|------|
| `test` | H2 in-memory | Auto (Hibernate) | `DBService` popula automaticamente |
| `dev` | MySQL | Manual (ddl-auto=none) | Não popula automaticamente |

### Configuração obrigatória (`application.properties`)

```properties
spring.profiles.active=test   # ou dev

jwt.secret=TROQUE_POR_UM_SECRET_SEGURO
jwt.expiration=86400000
```

> **Atenção:** nunca commite o `jwt.secret` ou credenciais de banco no repositório.
> Utilize variáveis de ambiente em produção.

---

## Tratamento de Erros

Todas as exceções são tratadas pelo `ResourceExceptionHandler` e retornam um JSON padronizado.

### Formato de erro padrão

```json
{
  "timestamp": 1708780800000,
  "status": 404,
  "error": "Object Not Found",
  "message": "Técnico não encontrado. Id: 99",
  "path": "/service/tecnicos/99"
}
```

### Formato de erro de validação

```json
{
  "timestamp": 1708780800000,
  "status": 400,
  "error": "Validation Error",
  "message": "Erro de validação",
  "path": "/service/tecnicos",
  "errors": [
    {
      "fieldName": "email",
      "message": "O campo EMAIL é requerido"
    }
  ]
}
```

### Tabela de status HTTP

| Situação | Status |
|---------|--------|
| Recurso não encontrado | `404 Not Found` |
| CPF ou e-mail já cadastrado | `409 Conflict` |
| Campos obrigatórios ausentes/inválidos | `400 Bad Request` |
| Token ausente ou inválido | `401 Unauthorized` |
| Papel sem permissão para a ação | `403 Forbidden` |

---

## Dados de Teste

No perfil `test`, o `DBService` popula automaticamente o banco com:

### Técnicos

| Nome | E-mail | Papéis | Senha |
|------|--------|--------|-------|
| Valdir Cezar | valdir@mail.com | ADMIN, TECNICO | 123456 |
| Tulio Maravilha | tulio@mail.com | ADMIN, TECNICO | 123456 |
| Tadashi Timeout | tadashi@mail.com | TECNICO | 123456 |
| + 3 técnicos | — | TECNICO | 123456 |

### Clientes

| Nome | E-mail | Papel | Senha |
|------|--------|-------|-------|
| Marie Curie | marie@mail.com | CLIENTE | 123456 |
| + 3 clientes | — | CLIENTE | 123456 |

### Chamados

5 chamados com diferentes combinações de status e prioridade são criados automaticamente.

---

## Projeto Relacionado

- **Frontend (Angular):** [`../`](../) — Interface web com Angular 14 e Angular Material
