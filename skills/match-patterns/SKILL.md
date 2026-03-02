---
name: match-patterns
description: This skill should be used when the user asks about how the api-match repository works, asks about patterns, architecture, conventions, base classes, naming rules, or asks "how do we do X in this project". Also activate when the user mentions Match.Core, Match.Api, Match.MongoDb, IRepository, MongoDB, SaveOrUpdateAsync, GetClientQuery, IRequest, or describes working on the api-match repository.
version: 1.0.0
---

# Padrões do Repositório api-match

## Arquitetura Geral

O projeto usa **Clean Architecture** com **CQRS** via **MediatR** e **MongoDB** como banco de dados.

```
Match.sln
├── Match.Api       → Controllers, Models, Program.cs (entrada HTTP)
├── Match.Core      → Commands, Queries, Entities, DTOs, Services (lógica de negócio)
└── Match.MongoDb   → Repository genérico, configuração de coleções MongoDB
```

## Organização do Match.Core

Código organizado por **domínio/feature**, não por tipo:

```
Match.Core/
├── Clientes/
│   ├── Commands/   → CreateClientCommand.cs (Command + Handler no mesmo arquivo)
│   └── Querys/     → GetClientQuery.cs (Query + Handler no mesmo arquivo)
├── Ambientes/
│   ├── Commands/
│   └── Querys/
├── Auth/
├── Entities/       → Entidades do domínio (Client, Response, Environment, etc.)
├── DTOs/           → Data Transfer Objects
├── Services/       → Integrações externas (Google, Replicate, Sendinblue, ZApi)
├── Extensions/     → Métodos fluentes para queries (ex: .ComId(), .ComClientIdOpcional())
└── IAs/            → Integrações com IA (OpenAI, Google, ModelsLab)
```

> **Atenção:** A pasta de queries usa o nome `Querys` (não `Queries`) — seguir esse padrão.

## Repositório Genérico — IRepository\<T\>

Toda lógica de dados passa por `IRepository<T>` (injetado automaticamente via DI ao registrar a coleção no MongoDB).

```csharp
// Leitura — retorna null se não encontrar
var client = await _clients.FirstOrDefaultAsync(x => x.ComId(request.Id));

// Leitura — retorna lista
var lista = await _responses.ToListAsync(x =>
    x.Where(r => r.ClienteId == request.ClienteId)
     .OrderByDescending(r => r.Date));

// Escrita — upsert (cria ou atualiza)
await _clients.SaveOrUpdateAsync(entity, x => x.Id == entity.Id);

// Escrita em lote
await _environments.SaveBatchAsync(listaDeObjetos);

// Contagem
var total = await _responses.CountAsync(x => x.Where(r => r.ClienteId == clienteId));

// Verificação de existência
var existe = await _clients.AnyAsync(x => x.Where(c => c.Email == email));

// Exclusão
await _logs.DeleteAsync(x => x.Id == id);
```

## Convenção de Arquivo: Handler Aninhado

**Regra fundamental:** Command/Query e seu Handler vivem no **mesmo arquivo**, com o Handler como **classe aninhada pública**.

```
Match.Core/{Dominio}/Commands/{Acao}{Entidade}Command.cs
└── public class {Acao}{Entidade}Command : IRequest<TResponse>
    └── public class {Acao}{Entidade}CommandHandler : IRequestHandler<...>

Match.Core/{Dominio}/Querys/{Acao}{Entidade}Query.cs
└── public class {Acao}{Entidade}Query : IRequest<TResponse>
    └── public class {Acao}{Entidade}QueryHandler : IRequestHandler<...>
```

## Convenções de Nomenclatura

| Tipo | Padrão | Exemplos |
|------|--------|---------|
| Command | `{Acao}{Entidade}Command` | `CreateClientCommand`, `SalvarUsuarioCommand`, `GerarTokenCommand` |
| Query | `{Acao}{Entidade}Query` | `GetClientQuery`, `ObterUsuarioPorIdQuery`, `ListarLogsQuery` |
| Handler (Command) | `{Acao}{Entidade}CommandHandler` | `CreateClientCommandHandler` |
| Handler (Query) | `{Acao}{Entidade}QueryHandler` | `GetClientQueryHandler` |
| Result (query complexa) | `{Acao}{Entidade}Result` | `GetResponsesPaginatedResult` |
| Entity | Singular, sem sufixo | `Client`, `Response`, `Environment`, `Usuario` |
| DTO | `{Proposito}DTO` | `PerfilClienteDTO`, `RespostaImagensDTO` |
| Model (API) | `{Entidade}Model` | `ClientModel`, `ResponseModel`, `PromptGeracaoImagemModel` |
| Controller | `{Entidade}Controller` | `AmbientesController`, `PromptsController` |

> **Mix Português/Inglês:** O projeto usa os dois. Novas features seguem o padrão do domínio onde estão inseridas — se o domínio usa PT, continue em PT; se usa EN, continue em EN.

## IDs como String

Todos os IDs são `string` com formato ObjectId do MongoDB:

```csharp
// ✅ Correto
public string Id { get; set; }
public string ClienteId { get; set; }

// ❌ Errado
public Guid Id { get; set; }
```

## Dependency Injection — MediatR

**MediatR é auto-descoberto via Assembly scan** — nunca registrar Command/Query handler manualmente.

```csharp
// Program.cs — registro automático
services.AddMediatR(cfg =>
{
    cfg.RegisterServicesFromAssemblies(GetAssemblies().ToArray());
});
```

## Validação — Manual (sem FluentValidation)

O projeto **não usa FluentValidation**. Validações são feitas manualmente:

```csharp
// No controller
if (model is null)
    return BadRequest("O modelo não pode ser nulo.");

// No handler
if (string.IsNullOrEmpty(request.Email))
    throw new ArgumentException("Email é obrigatório.");
```

## Persistência — SaveOrUpdateAsync (Upsert)

Não existe `Add()` + `Update()` separados. Toda escrita usa upsert:

```csharp
// Cria se não existe, atualiza se existe (com base no filtro)
await _clients.SaveOrUpdateAsync(client, x => x.Id == client.Id && x.Email == client.Email);

// Sem filtro — sempre insere (evitar em geral)
await _clients.SaveOrUpdateAsync(client);
```

## Memory Cache nos Controllers

Controllers usam `IMemoryCache` via helper estático `Cache`:

```csharp
// Recuperar do cache
var cached = Cache.Recuperar(_memoryCache, "chave-cache");

// Atualizar cache (TTL em minutos)
Cache.Atualizar(_memoryCache, "chave-cache", resultado, 60);
```

## Padrão de Controller

```csharp
[ApiController]
[Route("api/[controller]")]
public class {Entidade}Controller : Controller
{
    private readonly IMediator _mediator;

    public {Entidade}Controller(IMediator mediator)
    {
        _mediator = mediator;
    }

    [HttpGet("acao")]
    public async Task<IActionResult> Acao()
    {
        try
        {
            var result = await _mediator.Send(new MinhaQuery());
            return Ok(result);
        }
        catch (Exception ex)
        {
            ModelState.AddModelError("erro", ex.Message);
            return BadRequest(ModelState);
        }
    }
}
```

## Autenticação

- Rotas **públicas**: sem atributo de autorização (padrão em `AmbientesController`)
- Rotas **B2B (autenticadas)**: `[Authorize]` — controllers em `Controllers/B2b/`
- JWT Bearer configurado em `Program.cs`

## Encadeamento via MediatR

Handlers podem chamar outros Commands/Queries via `_mediator`:

```csharp
// Dentro de um handler
var client = await _mediator.Send(new GetClientByEmailQuery(email));
await _mediator.Send(new LogarErroCommand(ex.Message));
```

## Extensões de Filtro Fluente

O projeto usa extension methods para compor queries de forma fluente:

```csharp
// Exemplo de uso em handler
var result = await _responses.FirstOrDefaultAsync(x =>
    x.ComClientId(request.ClienteId)
     .ComImagensGenerativasOpcional(request.ImagesGeneratives));
```

Extension methods ficam em `Match.Core/Extensions/`.
