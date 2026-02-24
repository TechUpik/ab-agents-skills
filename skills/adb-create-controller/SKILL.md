---
name: adb-create-controller
description: This skill should be used when the user wants to create a new controller, add new API endpoints, create new routes, or expose new functionality via HTTP in the rep-adb-api repository. Activate when the user mentions "criar controller", "novo controller", "criar endpoint", "nova rota", "nova API", "expor endpoint", or wants to add HTTP GET/POST/PUT/PATCH/DELETE routes.
version: 1.0.0
---

# Criar um novo Controller no rep-adb-api

## Estrutura de um Controller

**Caminho:** `src/AdB.API/Controllers/{Entidade}Controller.cs`

```csharp
using AdB.API.Controllers.v1;
using AdB.Application.Commands.{Entidade};
using AdB.Application.Queries.{Modulo}.{Entidade};
using MediatR;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using NSwag.Annotations;

namespace AdB.API.Controllers;

[Authorize]
[Route("api/v{version:apiVersion}/{recurso}")]
[OpenApiTag("{Entidade}")]
public class {Entidade}Controller : MainController
{
    private readonly IMediator _mediator;

    public {Entidade}Controller(IMediator mediator)
    {
        _mediator = mediator;
    }

    // GET — Listagem
    [HttpGet("listar")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [OpenApiOperation("Listar {entidades}", "Retorna lista de {entidades}")]
    public async Task<ActionResult> Listar(
        [FromQuery] string? termoPesquisa = null,
        [FromQuery] bool? ativo = null,
        [FromQuery] int pagina = 1,
        [FromQuery] int tamanhoPagina = 10)
    {
        try
        {
            var query = new Listar{Entidade}Query
            {
                TermoPesquisa = termoPesquisa,
                Ativo = ativo,
                Pagina = pagina,
                TamanhoPagina = tamanhoPagina
            };

            var resultado = await _mediator.Send(query);
            return CustomResponse(resultado);
        }
        catch (Exception ex)
        {
            AdicionarErroTratado("erro", ex.Message);
            return CustomResponse();
        }
    }

    // GET — Detalhe por ID
    [HttpGet("{id:guid}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [OpenApiOperation("Obter {entidade}", "Retorna dados de uma {entidade} pelo ID")]
    public async Task<ActionResult> ObterPorId(Guid id)
    {
        try
        {
            var resultado = await _mediator.Send(new Obter{Entidade}Query(id));
            if (resultado is null) return NotFound();
            return CustomResponse(resultado);
        }
        catch (Exception ex)
        {
            AdicionarErroTratado("erro", ex.Message);
            return CustomResponse();
        }
    }

    // POST — Criação
    [HttpPost("criar")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [OpenApiOperation("Criar {entidade}", "Cria uma nova {entidade}")]
    public async Task<ActionResult> Criar([FromBody] Criar{Entidade}Command command)
    {
        // Injetar dados do contexto (token, rota) que não vêm do body
        command.{EntidadeRelacionada}Id = User.IdUsuarioLogado().Value;

        return CustomResponse(await _mediator.Send(command));
    }

    // PUT — Atualização completa
    [HttpPut("{id:guid}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [OpenApiOperation("Atualizar {entidade}", "Atualiza uma {entidade}")]
    public async Task<ActionResult> Atualizar(Guid id, [FromBody] Atualizar{Entidade}Command command)
    {
        command.{Entidade}Id = id;
        return CustomResponse(await _mediator.Send(command));
    }

    // PATCH — Atualização parcial
    [HttpPatch("{id:guid}/cancelar")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [OpenApiOperation("Cancelar {entidade}", "Cancela uma {entidade}")]
    public async Task<ActionResult> Cancelar(Guid id)
    {
        var command = new Cancelar{Entidade}Command { {Entidade}Id = id };
        return CustomResponse(await _mediator.Send(command));
    }

    // DELETE — Exclusão
    [HttpDelete("{id:guid}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    [OpenApiOperation("Excluir {entidade}", "Exclui uma {entidade}")]
    public async Task<ActionResult> Excluir(Guid id)
    {
        var command = new Excluir{Entidade}Command { {Entidade}Id = id };
        return CustomResponse(await _mediator.Send(command));
    }
}
```

## Padrões de Route

| Perfil de acesso | Rota | Exemplo |
|------------------|------|---------|
| Público | `[AllowAnonymous]` | cadastro, login |
| Usuário logado | `[Authorize]` (padrão) | perfil, consultoria |
| Admin/Painel | `painel-admin/` no path | `/api/v1/clientes/painel-admin/listar` |
| Consultor/Arquiteto | injetado via `User.IdUsuarioLogado()` | path com `{consultorId}` |

```csharp
// Rota com segmento de painel admin
[Route("api/v{version:apiVersion}/clientes")]
// Endpoint: GET /api/v1/clientes/painel-admin/listar
[HttpGet("painel-admin/listar")]

// Rota com ID de entidade pai
[Route("api/v{version:apiVersion}/consultoria/{consultoriaId}/agendamentos")]
// Endpoint: POST /api/v1/consultoria/{id}/agendamentos/criar
[HttpPost("criar")]
```

## Extraindo dados do Token JWT

```csharp
// ID do usuário logado
var usuarioId = User.IdUsuarioLogado().Value;

// ID da consultoria (claim customizada)
var consultoriaId = User.ConsultoriaId().Value;
```

## CustomResponse — Referência

```csharp
// ✅ Query retornou dados
return CustomResponse(resultado);

// ✅ Command retornou CommandResult (mapeia erros automaticamente)
return CustomResponse(await _mediator.Send(command));

// ✅ Não encontrado
if (resultado is null) return NotFound();

// ✅ Erro controlado
AdicionarErroTratado("campo", "mensagem de erro");
return CustomResponse();

// ✅ Catch genérico
catch (Exception ex)
{
    AdicionarErroTratado("erro", ex.Message);
    return CustomResponse();
}
```

## Controller com múltiplas dependências

```csharp
public class {Entidade}Controller : MainController
{
    private readonly IMediator _mediator;
    private readonly I{Entidade}Queries _{entidade}Queries;  // Para queries Dapper
    private readonly ICacheService _cache;                    // Para cache
    private readonly IHostEnvironment _hostEnvironment;       // Para ambiente

    public {Entidade}Controller(
        IMediator mediator,
        I{Entidade}Queries {entidade}Queries,
        ICacheService cache,
        IHostEnvironment hostEnvironment)
    {
        _mediator = mediator;
        _{entidade}Queries = {entidade}Queries;
        _cache = cache;
        _hostEnvironment = hostEnvironment;
    }
}
```

## Checklist

- [ ] Herda de `MainController` (não de `Controller` ou `ControllerBase` diretamente)
- [ ] Atributo `[Route("api/v{version:apiVersion}/{recurso}")]`
- [ ] Atributo `[OpenApiTag("{Entidade}")]` para agrupamento no Swagger
- [ ] Atributo `[Authorize]` ou `[AllowAnonymous]` conforme necessidade
- [ ] Cada endpoint tem `[ProducesResponseType]` declarado
- [ ] Cada endpoint tem `[OpenApiOperation]` com nome e descrição
- [ ] IDs de rota/usuário injetados no command antes de enviar ao mediator
- [ ] Retornos usam `CustomResponse()` ou `CustomResponse(resultado)`
- [ ] Exceções capturadas com `AdicionarErroTratado` + `CustomResponse()`
