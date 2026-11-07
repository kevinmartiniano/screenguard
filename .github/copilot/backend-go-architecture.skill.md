# Backend Go Architecture Skill

## Objetivo

Este projeto deve seguir uma arquitetura Go simples, escalável e alinhada com padrões amplamente utilizados na comunidade Go.

A arquitetura é inspirada por:

- golang-standards/project-layout
- ThreeDotsLabs/wild-workouts-go-ddd-example
- go-clean-arch

O objetivo NÃO é implementar Clean Architecture completa, DDD completo ou CQRS completo.

O objetivo é manter uma arquitetura pragmática para uma aplicação monolítica que possa evoluir com facilidade.

---

# Regras Gerais

## Go

Utilizar:

```text
Go 1.27+
```

Sempre utilizar recursos modernos da linguagem quando apropriado.

---

## HTTP

Priorizar:

```go
net/http
```

Evitar frameworks HTTP.

Somente utilizar bibliotecas externas mediante justificativa técnica.

---

## Logging

Utilizar:

```go
log/slog
```

Evitar:

```go
fmt.Println
```

para logs da aplicação.

---

## Context

Toda chamada deve propagar:

```go
context.Context
```

Fluxo:

```text
Handler
→ Service
→ Repository
```

---

## Dependency Injection

Dependências devem ser explícitas.

Não utilizar:

- singletons globais
- variáveis globais
- service locators dinâmicos

---

# Estrutura Oficial

```text
cmd/
└── app/
    └── main.go

internal/
├── app/
│   ├── app.go
│   └── router.go
│
├── container/
│   └── container.go
│
├── config/
│   └── config.go
│
├── server/
│   ├── server.go
│   ├── routes.go
│   └── middleware.go
│
├── routes/
│   ├── health.go
│   ├── auth.go
│   ├── devices.go
│   └── activities.go
│
├── handlers/
│   ├── health_handler.go
│   ├── auth_handler.go
│   ├── device_handler.go
│   └── activity_handler.go
│
├── services/
│   ├── auth_service.go
│   ├── device_service.go
│   └── activity_service.go
│
├── repositories/
│   ├── user_repository.go
│   ├── device_repository.go
│   └── activity_repository.go
│
├── database/
│   ├── connection.go
│   └── migrations.go
│
└── models/
    ├── user.go
    ├── device.go
    └── activity.go

utils/
├── json_response.go
├── errors.go
└── validator.go

deploy/
├── Dockerfile
└── compose.yml

docs/
└── proposals/
```

---

# Responsabilidades

## cmd/app/main.go

Responsável exclusivamente por:

- carregar configurações
- inicializar banco
- construir container
- inicializar servidor
- iniciar aplicação

Não implementar:

- regras de negócio
- rotas
- acesso ao banco

Exemplo de fluxo:

```text
Config
↓
Database
↓
Container
↓
Server
↓
Start
```

---

## internal/container

Responsável pelo registro de dependências.

Todas as dependências da aplicação devem ser criadas aqui.

Exemplo:

```go
type Container struct {
    DB *sql.DB

    UserRepository *repositories.UserRepository

    UserService *services.UserService
    UserHandler *handlers.UserHandler
}
```

Objetivos:

- centralizar bootstrap
- facilitar testes
- facilitar evolução da aplicação

---
## internal/app

Abstrações e componentes compartilhados da aplicação.

Exemplos:

- Router
- App
- Interfaces comuns

---

## internal/app/router.go

Implementar abstração sobre net/http.

Disponibilizar:

```go
router.Get(...)
router.Post(...)
router.Put(...)
router.Delete(...)
router.Patch(...)
```

Objetivo:
Padronizar o registro de rotas.

Evitar utilização direta de:

```go
mux.HandleFunc(...)
```

espalhada pelo sistema.

---

## internal/server

Responsável por:

- inicialização do servidor
- graceful shutdown
- middlewares
- registro das rotas

Não conter regras de negócio.

---

## internal/server/routes.go

Ponto único para registrar rotas.

Exemplo:

```go
func RegisterRoutes(router *app.Router,
    container *container.Container,
)
```

Todas as rotas devem ser registradas aqui.

---

## internal/routes

Responsável apenas pela definição de endpoints.

Exemplo:

```go
func RegisterHealthRoutes(
    router *app.Router,
    handler *handlers.HealthHandler,
)
```

Não implementar:

- SQL
- regras de negócio

---

## internal/handlers

Responsável por:
- ler requests
- validar entrada
- chamar services
- produzir responses

Não acessar banco diretamente.
Fluxo:

```text
Request
↓
Validation
↓
Service
↓
Response
```

---

3# internal/services

Responsável por:

- regras de negócio
- casos de uso
- orquestração

Exemplos:

```text
CreateDevice
RegisterActivity
CalculateScreenTime
BlockDevice
```
Toda regra de negócio deve ficar *esta camada.

---

## internal/repositories

Responsável por:

- persistência
- consultas SQL
- acesso ao banco

Não implementar regras de negócio.

---

## internal/databases
Responsável por:

- conexão
- migrations
- bootstrap do banco

Inicialmente:

```text
SQLite
```

Preparado para futura migração para:

```text
PostgreSQL
```

---

## internal/models

Entidades da aplicação.
Exemplo:

```go
type Device struct {
    ID string
    Name string
}```

Modelos devem ser simples.

---

# Responses

Utilizar helper compartilhado.

Arquivo:

```text
utils/json_response.go
```

Exemplo:

```go
jsonresponse.Success(
    w,
    http.StatusOK,
    payload,
)
```

ou

```go
jsonresponse.Error(
    w,
    http.StatusBadRequest,
    err,
)
```

Evitar repetição de:
```go
json.NewEncoder(...)
```

nos handlers.

---

# Banco

Inicialmente utilizar:

```text
SQLite
```
Acesso através de:

```go
database sql
```

Evitar ORMs inicialmente.
Preferir:

```go
queries explícitas
```

---

# Docker

Arquivo obrigatório:

```text
deploy/Dockerfile```

Requisitos:

- build multi-stage
- binário compilado
- imagem final mínima
- usuário não-root

---
# Docker Compose

Arquivo:

```text
deploy/compose.yml
```

Objetivo:
Executar ambiente local.

Inicialmente conter apenas:

```text
backend
```

Adicionar serviços conforme necessário.

---

# Testes

Priorizar testes em:

```text
services
repositories
```

Estrutura:

```text internal/services/*_test.go
internal/repositories/*_test.go
```

---
# Convenções

Sempre preferir:

- omposição
- injeção explícita de dependências
- responsabilidade única
- simplicidade
- código idiomática Go

Evitar:

- arquitetura excessivamente complexa
- abstrações prematuras
- interfaces sem necessidades
- dependências globais
- lógica em handlers
- lógica em repositories
---

# Fluxo Oficial

Novas funcionalidades devem seguir:

```text
Model
↓
Repository
↓
Service
↓
Handler
↓
Route
↓
Server
```

---

# Regra Principal

Ao gerar código para este projeto:

1. Respeitar a estrutura de diretórios definida nesta skill.
2. Registrar todas as dependências no Container.
3. Registrar todas as rotas em internal/server/routes.go.
4. Utilizar net/http como base.
5. Utilizar slog para logs.
6. Utilizar context.Context em todas as camadas.
7. Não implementar regras de negócio fora da camada Service.
8. Não acessar banco fora da camada Repository.
9. Produzir código idiomático Go priorizando simplicidade e manutenibilidade.
