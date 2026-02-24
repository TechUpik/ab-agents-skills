---
name: adb-patterns
description: This skill should be used when the user asks about how the rep-adb-api repository works, asks about patterns, architecture, conventions, base classes, naming rules, or asks "how do we do X in this project". Also activate when the user mentions AdB, AdBContext, CommandHandler, MainController, MediatR, CQRS, IRequestHandler, CommandResult, or AbstractValidatorBase.
version: 1.0.0
---

# Padrões do Repositório rep-adb-api

## Arquitetura Geral

O projeto usa **Clean Architecture** com **CQRS** via **MediatR**.

```
/src
├── AdB.Domain          → Entidades e regras de domínio
├── AdB.Core            → Base classes (Command, CommandHandler, Message, etc.)
├── AdB.Data            → Repositórios, EF Core (AdBContext), Dapper (AdBDapperContext)
├── AdB.Application     → Commands, Queries, Services
└── AdB.API             → Controllers, Configurações de DI
```

## Hierarquia de Classes Base

```
Message (abstract)
├── Command : Message, IRequest<CommandResult>
└── Event : Message

CommandHandler (abstract)
└── {Entity}CommandHandler : CommandHandler, IRequestHandler<...>

Controller
└── MainController : Controller
    └── {Entity}Controller : MainController
```

## Namespaces Padrão

| Tipo | Namespace |
|------|-----------|
| Commands | `AdB.Application.Commands.{Entidade}` |
| Queries (MediatR) | `AdB.Application.Queries.{Modulo}.{Entidade}` |
| Queries (Service) | `AdB.Application.Queries` |
| Controllers | `AdB.API.Controllers` |
| DI Commands | `AdB.API.Configurations.DependencyInjections` |

## Convenções de Nomenclatura

| Tipo | Padrão | Exemplos |
|------|--------|----------|
| Command | `{Verbo}{Entidade}Command` | `RegistrarAgendamentoCommand`, `CancelarPresenteCommand` |
| Handler de Command | `{Entidade}CommandHandler` | `AgendamentoCommandHandler`, `PresenteCommandHandler` |
| Query (MediatR) | `{Verbo}{Entidade}Query` | `ListarArquitetosQuery`, `ObterArquitetoQuery` |
| Query Result | `{Verbo}{Entidade}QueryResult` ou `{Entidade}ItemResponse` | `ObterArquitetoQueryResult` |
| Query (Service) | `{Entidade}Queries` | `ClienteQueries`, `OrcamentoQueries` |
| Controller | `{Entidade}Controller` | `ProdutoController`, `AgendamentoController` |
| Repository (interface) | `I{Entidade}Repository` | `IConsultoriaRepository` |
| Service (interface) | `I{Nome}Service` | `IGoogleService`, `IDiscordService` |

## Métodos de Repositório (Padrão)

```csharp
ObterPorId(Guid id)
ObterTodos()
Adicionar(T entity)
Atualizar(T entity)
Excluir(T entity)
```

## Recursos de String

Sempre usar resource strings para mensagens:
- `ValidationResource.Obrigatorio` → campo obrigatório
- `ValidationResource.Invalido` → campo inválido
- `ValidationResource.NaoLocalizado` → entidade não encontrada
- `ValidationResource.MaximoCaracteres` → máximo de caracteres
- `TermsResource.{NomeDoCampo}` → nome do campo em PT-BR

Uso com interpolação:
```csharp
ValidationResource.Obrigatorio.FormatField(new string[] { TermsResource.Nome })
ValidationResource.NaoLocalizado.FormatField(new string[] { TermsResource.Consultoria })
```

## Fluxo de Validação de Commands

```
Controller → _mediator.Send(command)
              ↓
Handler.Handle() → if (!request.Validar()) return request.ValidationResult;
                    ↓
                   AbstractValidatorBase<T>.Validate() → CommandResult(errors)
```

## Fluxo de Persistência de Commands

```csharp
_repository.Adicionar(entity);           // 1. Adiciona ao contexto
var result = await PersistirDados(_repository.UnitOfWork); // 2. Commit
return result;                           // 3. Retorna CommandResult
```

## Retorno de Erros em Handlers

```csharp
// Com 1 termo
return LancarErro(ValidationResource.NaoLocalizado, TermsResource.Consultoria);

// Direto com mensagem
return LancarErro("Mensagem de erro direto");

// Adicionar erro sem retornar
AdicionarErro("Mensagem de erro");
```

## CustomResponse em Controllers

```csharp
// Retorna 200 OK com dados
return CustomResponse(dados);

// Retorna 200 OK com CommandResult (mapeia erros automaticamente)
return CustomResponse(await _mediator.Send(command));

// Adicionar erro manualmente e retornar 400
AdicionarErroTratado("campo", "mensagem");
return CustomResponse();
```

## Registro de DI

- **Commands** → `src/AdB.API/Configurations/DependencyInjections/CommandsDIConfig.cs`
- **Queries (Service)** → `src/AdB.API/Configurations/DependencyInjections/QueriesDIConfig.cs`
- **Queries MediatR** → Auto-registradas via `AddMediatR` + Assembly scanning
- **Outros services** → `src/AdB.API/Configurations/DependencyInjections/DIConfig.cs`

## Dois Padrões de Query

### 1. MediatR (preferido para novas features)
Query + Handler + Response no mesmo arquivo `.cs`.
Handler implementa `IRequestHandler<TQuery, TResponse>`.
Auto-descoberto pelo MediatR — não precisa registrar manualmente.

### 2. Service com Dapper (para queries complexas com SQL)
Interface `I{Entidade}Queries` + implementação `{Entidade}Queries`.
Usa `AdBDapperContext` com SQL em `SQLQueriesResource`.
Deve ser registrado em `QueriesDIConfig.cs`.
