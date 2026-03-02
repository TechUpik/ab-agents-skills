---
name: match-create-controller
description: This skill should be used when the user wants to create a new controller, add new API endpoints, or expose new functionality via HTTP in the api-match repository. Activate when the user mentions "criar controller match", "novo controller match", "criar endpoint match", "nova rota match", "nova API match", or wants to add HTTP routes in the context of the Match.Api project.
version: 1.0.0
---

# Criar um novo Controller no api-match

## Estrutura base de um Controller

**Caminho:** `Match.Api/Controllers/{Entidade}Controller.cs`

```csharp
using MediatR;
using Match.Core.{Dominio}.Commands;
using Match.Core.{Dominio}.Querys;
using Microsoft.AspNetCore.Mvc;

namespace Match.Api.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class {Entidade}Controller : Controller
    {
        private readonly IMediator _mediator;

        public {Entidade}Controller(IMediator mediator)
        {
            _mediator = mediator;
        }

        [HttpGet("{id}")]
        public async Task<IActionResult> ObterPorId(string id)
        {
            try
            {
                var result = await _mediator.Send(new Get{Entidade}Query(id));
                if (result is null) return NotFound();
                return Ok(result);
            }
            catch (Exception ex)
            {
                ModelState.AddModelError("erro", ex.Message);
                return BadRequest(ModelState);
            }
        }

        [HttpPost("criar")]
        public async Task<IActionResult> Criar([FromBody] {Entidade}Model model)
        {
            try
            {
                var entidade = new {Entidade}
                {
                    // Mapear propriedades do model para a entidade
                };

                var result = await _mediator.Send(new Create{Entidade}Command(entidade));
                return Ok(result);
            }
            catch (Exception ex)
            {
                ModelState.AddModelError("erro", ex.Message);
                return BadRequest(ModelState);
            }
        }
    }
}
```

---

## Controller B2B (autenticado)

Controllers que exigem autenticação ficam em `Match.Api/Controllers/B2b/` e usam `[Authorize]`:

```csharp
using Microsoft.AspNetCore.Authorization;

namespace Match.Api.Controllers.B2b
{
    [Authorize]
    [ApiController]
    [Route("api/[controller]")]
    public class {Entidade}B2bController : Controller
    {
        private readonly IMediator _mediator;

        public {Entidade}B2bController(IMediator mediator)
        {
            _mediator = mediator;
        }

        // Endpoints autenticados aqui
    }
}
```

---

## Padrão de resposta

Todos os endpoints seguem o mesmo padrão de resposta:

```csharp
// ✅ Sucesso com dados
return Ok(result);

// ✅ Não encontrado
if (result is null) return NotFound();

// ✅ Sem conteúdo
return NoContent();

// ✅ Erro de validação / negócio
ModelState.AddModelError("campo", "mensagem");
return BadRequest(ModelState);

// ✅ Catch genérico (obrigatório em todos os endpoints)
catch (Exception ex)
{
    ModelState.AddModelError("erro", ex.Message);
    return BadRequest(ModelState);
}
```

---

## Controller com Memory Cache

Use quando o endpoint retorna dados estáveis (categorias, configurações, listas base):

```csharp
using Microsoft.Extensions.Caching.Memory;

[ApiController]
[Route("api/[controller]")]
public class {Entidade}Controller : Controller
{
    private readonly IMediator _mediator;
    private readonly IMemoryCache _memoryCache;

    public {Entidade}Controller(IMediator mediator, IMemoryCache memoryCache)
    {
        _mediator = mediator;
        _memoryCache = memoryCache;
    }

    [HttpGet("categorias")]
    public async Task<IActionResult> Categorias()
    {
        try
        {
            const string cacheKey = "{entidade}Categorias";
            var cached = Cache.Recuperar(_memoryCache, cacheKey);

            if (cached is not null)
                return Ok(cached);

            var result = await _mediator.Send(new Get{Entidade}CategoriasQuery());
            Cache.Atualizar(_memoryCache, cacheKey, result, 60); // 60 minutos

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

---

## Controller com upload de arquivo

```csharp
[HttpPost("importar")]
public async Task<IActionResult> Importar([FromForm] Importar{Entidade}Model model)
{
    try
    {
        if (model.Arquivo is null || model.Arquivo.Length == 0)
            return BadRequest("Arquivo não informado.");

        using var stream = model.Arquivo.OpenReadStream();
        var result = await _mediator.Send(new Importar{Entidade}Command(
            stream,
            model.Arquivo.FileName,
            model.OutrosDados));

        return Ok(result);
    }
    catch (Exception ex)
    {
        ModelState.AddModelError("erro", ex.Message);
        return BadRequest(ModelState);
    }
}
```

Model para upload:
```csharp
// Match.Api/Models/Importar{Entidade}Model.cs
public class Importar{Entidade}Model
{
    public IFormFile Arquivo { get; set; }
    public string OutrosDados { get; set; }
}
```

---

## Controller com listagem paginada

```csharp
[HttpGet("listar")]
public async Task<IActionResult> Listar(
    [FromQuery] int indicePaginacao = 0,
    [FromQuery] int quantidadePaginacao = 20,
    [FromQuery] string termoPesquisa = "")
{
    try
    {
        var result = await _mediator.Send(new Listar{Entidade}PaginadoQuery(
            indicePaginacao,
            quantidadePaginacao,
            termoPesquisa));

        return Ok(result);
    }
    catch (Exception ex)
    {
        ModelState.AddModelError("erro", ex.Message);
        return BadRequest(ModelState);
    }
}
```

---

## Models (request bodies)

Models ficam em `Match.Api/Models/` (ou `Match.Api/Models/B2b/` para B2B):

```csharp
// Match.Api/Models/{Entidade}Model.cs
public class {Entidade}Model
{
    public string Nome { get; set; }
    public string Descricao { get; set; }
    // ... campos do body
}
```

---

## Padrões de rota

| Tipo de endpoint | Exemplo de rota |
|-----------------|----------------|
| Listagem | `GET /api/{entidade}/listar` |
| Detalhe por ID | `GET /api/{entidade}/{id}` |
| Criação | `POST /api/{entidade}/criar` |
| Atualização | `PUT /api/{entidade}/atualizar` |
| Exclusão | `DELETE /api/{entidade}/{id}` |
| Ação específica | `POST /api/{entidade}/publicar` |
| Listagem por entidade pai | `GET /api/{entidade}/por-cliente/{clienteId}` |

---

## Checklist

- [ ] Arquivo em `Match.Api/Controllers/{Entidade}Controller.cs`
- [ ] Controladores B2B em `Match.Api/Controllers/B2b/` com `[Authorize]`
- [ ] Herda de `Controller` (não `ControllerBase`)
- [ ] Atributos `[ApiController]` e `[Route("api/[controller]")]`
- [ ] Todo endpoint tem `try/catch` com `ModelState.AddModelError` + `BadRequest(ModelState)`
- [ ] Retornos usam `Ok()`, `NotFound()`, `BadRequest()` ou `NoContent()`
- [ ] Cache com `IMemoryCache` para dados estáticos (TTL 60 min)
- [ ] Models de request em `Match.Api/Models/`
- [ ] IDs de rota são `string` (não `Guid`)
