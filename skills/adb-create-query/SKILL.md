---
name: adb-create-query
description: This skill should be used when the user wants to create a new MediatR query, list data, get details, search or filter records, or implement a read operation using EF Core in the rep-adb-api repository. Activate when the user mentions "criar query", "nova query", "listar", "obter por id", "buscar", "filtrar", "endpoint de listagem", "endpoint de consulta", "paginação", "IRequest", or describes fetching/reading data with EF Core. For SQL-heavy Dapper queries, also activate the adb-dapper-query skill.
version: 1.0.0
---

# Criar Query MediatR no rep-adb-api

## Convenção fundamental: arquivo único

Toda Query MediatR segue a convenção de **arquivo único** contendo as três classes:

```
src/AdB.Application/Queries/{Modulo}/{Entidade}/{Verbo}{Entidade}Query.cs
```

```
[mesmo arquivo]
├── class {Verbo}{Entidade}Query          → parâmetros de entrada
├── class {Verbo}{Entidade}QueryResponse  → DTO de retorno
└── class {Verbo}{Entidade}QueryHandler   → implementação
```

> **Não registrar no DI** — MediatR descobre automaticamente via Assembly scan.

---

## Tipo 1 — Listagem com filtros (mais comum)

```csharp
using AdB.Data.Context;
using MediatR;
using Microsoft.EntityFrameworkCore;

namespace AdB.Application.Queries.{Modulo}.{Entidade};

// 1. Query — Parâmetros/filtros como propriedades com setter
public class Listar{Entidade}Query : IRequest<IEnumerable<{Entidade}ItemResponse>>
{
    public string? TermoPesquisa { get; set; }
    public bool? Ativo { get; set; }
    public {TipoEnum}? Status { get; set; }
    public int? Pagina { get; set; }
    public int? QuantidadePorPagina { get; set; }
}

// 2. Response — DTO de retorno (nunca expor entidade de domínio)
public class {Entidade}ItemResponse
{
    public Guid Id { get; set; }
    public string Nome { get; set; }
    public bool Ativo { get; set; }
    // ... campos necessários
}

// 3. Handler
public class Listar{Entidade}QueryHandler : IRequestHandler<Listar{Entidade}Query, IEnumerable<{Entidade}ItemResponse>>
{
    private readonly AdBContext _context;

    public Listar{Entidade}QueryHandler(AdBContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<{Entidade}ItemResponse>> Handle(
        Listar{Entidade}Query request,
        CancellationToken cancellationToken)
    {
        var query = _context.{Entidades}
            .AsNoTracking()       // ← SEMPRE em queries de leitura
            .AsQueryable();

        // Filtro de texto com múltiplos campos
        if (!string.IsNullOrWhiteSpace(request.TermoPesquisa))
        {
            var termo = request.TermoPesquisa.Trim().ToLower();
            query = query.Where(x =>
                x.Nome.Contains(termo) ||
                x.Email.Contains(termo) ||
                x.CPF.Contains(termo));
        }

        // Filtro nullable — apenas se fornecido
        if (request.Ativo.HasValue)
            query = query.Where(x => x.Ativo == request.Ativo.Value);

        if (request.Status.HasValue)
            query = query.Where(x => x.Status == request.Status.Value);

        // Paginação
        if (request.Pagina.HasValue && request.QuantidadePorPagina.HasValue)
        {
            var skip = (request.Pagina.Value - 1) * request.QuantidadePorPagina.Value;
            query = query.Skip(skip).Take(request.QuantidadePorPagina.Value);
        }

        return await query
            .OrderBy(x => x.Nome)
            .Select(x => new {Entidade}ItemResponse     // ← Projeção no banco (eficiente)
            {
                Id = x.Id,
                Nome = x.Nome,
                Ativo = x.Ativo
            })
            .ToListAsync(cancellationToken);
    }
}
```

---

## Tipo 2 — Listagem paginada com contagem total (PagedResult)

Use quando o frontend precisa saber o total de registros para paginação.

```csharp
public class Listar{Entidade}PaginadoQuery : IRequest<PagedResult<{Entidade}ItemResponse>>
{
    public string? TermoPesquisa { get; }
    public int Pagina { get; }
    public int TamanhoPagina { get; }

    public Listar{Entidade}PaginadoQuery(string? termoPesquisa, int pagina = 1, int tamanhoPagina = 20)
    {
        TermoPesquisa = termoPesquisa;
        Pagina = pagina <= 0 ? 1 : pagina;
        TamanhoPagina = tamanhoPagina <= 0 ? 20 : tamanhoPagina;
    }
}

public class Listar{Entidade}PaginadoQueryHandler : IRequestHandler<Listar{Entidade}PaginadoQuery, PagedResult<{Entidade}ItemResponse>>
{
    public async Task<PagedResult<{Entidade}ItemResponse>> Handle(
        Listar{Entidade}PaginadoQuery request,
        CancellationToken cancellationToken)
    {
        var query = _context.{Entidades}
            .AsNoTracking()
            .Where(x => !x.Excluido);

        if (!string.IsNullOrWhiteSpace(request.TermoPesquisa))
        {
            var termo = request.TermoPesquisa.Trim().ToLower();
            query = query.Where(x => x.Nome.ToLower().Contains(termo));
        }

        // Contar ANTES de paginar
        var total = await query.CountAsync(cancellationToken);

        var itens = await query
            .OrderByDescending(x => x.Criado)
            .Skip((request.Pagina - 1) * request.TamanhoPagina)
            .Take(request.TamanhoPagina)
            .Select(x => new {Entidade}ItemResponse { Id = x.Id, Nome = x.Nome })
            .ToListAsync(cancellationToken);

        return new PagedResult<{Entidade}ItemResponse>
        {
            Items = itens,
            Total = total,
            Page = request.Pagina,
            PageSize = request.TamanhoPagina
        };
    }
}
```

---

## Tipo 3 — Detalhe por ID

```csharp
// Query com ID via construtor (imutável)
public class Obter{Entidade}Query : IRequest<Obter{Entidade}QueryResult>
{
    public Guid Id { get; }

    public Obter{Entidade}Query(Guid id)
    {
        Id = id;
    }
}

// Result detalhado (diferente do ItemResponse da listagem)
public class Obter{Entidade}QueryResult
{
    public Guid Id { get; set; }
    public string Nome { get; set; }
    // ... todos os campos do detalhe

    // Coleções de entidades relacionadas
    public ICollection<{EntidadeRelacionada}Response> Itens { get; set; } = new List<{EntidadeRelacionada}Response>();
}

public class Obter{Entidade}QueryHandler : IRequestHandler<Obter{Entidade}Query, Obter{Entidade}QueryResult>
{
    public async Task<Obter{Entidade}QueryResult> Handle(
        Obter{Entidade}Query request,
        CancellationToken cancellationToken)
    {
        var entidade = await _context.{Entidades}
            .AsNoTracking()
            .Include(x => x.{NavegacaoRelacionada})
                .ThenInclude(r => r.{OutraNavegacao})
            .FirstOrDefaultAsync(x => x.Id == request.Id, cancellationToken);

        // Retornar null → Controller trata com NotFound()
        if (entidade is null) return null;

        return new Obter{Entidade}QueryResult
        {
            Id = entidade.Id,
            Nome = entidade.Nome,
            Itens = entidade.{NavegacaoRelacionada}
                .Select(r => new {EntidadeRelacionada}Response { Id = r.Id })
                .ToList()
        };
    }
}
```

---

## Tipo 4 — Two-pass (performance para listagens com relacionamentos)

Use quando a listagem paginada precisa de vários `.Include()` — evita explosão cartesiana.

```csharp
public async Task<ResultadoResponse> Handle(ListarQuery request, CancellationToken cancellationToken)
{
    // PASSO 1: Buscar apenas IDs com filtros (query leve, sem Include)
    var idsQuery = _context.{Entidades}
        .AsNoTracking()
        .Where(x => !x.Excluido)
        .Where(x => x.ClienteNavigation.CPF.Contains(request.TermoPesquisa));

    var total = await idsQuery.CountAsync(cancellationToken);

    var skip = (request.Pagina ?? 0) * (request.QuantidadePorPagina ?? 20);
    var idsPaginados = await idsQuery
        .Skip(skip)
        .Take(request.QuantidadePorPagina ?? 20)
        .Select(x => x.Id)
        .ToListAsync(cancellationToken);

    // PASSO 2: Carregar dados completos apenas para os IDs paginados
    var itens = await _context.{Entidades}
        .AsNoTracking()
        .Include(x => x.ClienteNavigation)
            .ThenInclude(c => c.IdEnderecoNavigation)
        .Include(x => x.ConsultoriaNavigation)
            .ThenInclude(c => c.ArquitetoNavigation)
        .Where(x => idsPaginados.Contains(x.Id))
        .ToListAsync(cancellationToken);

    return new ResultadoResponse
    {
        Total = total,
        QtdFiltro = total,
        Dados = itens.Select(x => MapToResponse(x)).ToList()
    };
}
```

---

## Padrões EF Core

### Filtros dinâmicos
```csharp
// Nullable enum/bool
if (request.Status.HasValue)
    query = query.Where(x => x.Status == request.Status.Value);

// String nula ou vazia
if (!string.IsNullOrWhiteSpace(request.TermoPesquisa))
    query = query.Where(x => x.Nome.Contains(request.TermoPesquisa));

// Guid opcional
if (request.ArquitetoId.HasValue)
    query = query.Where(x => x.ArquitetoId == request.ArquitetoId.Value);

// Lista de valores (IN)
if (request.StatusSelecionados?.Any() == true)
    query = query.Where(x => request.StatusSelecionados.Contains((int)x.Status));
```

### Include / ThenInclude
```csharp
// Simples
.Include(x => x.ArquitetoNavigation)

// Dois níveis
.Include(x => x.ConsultoriaNavigation)
    .ThenInclude(c => c.ArquitetoNavigation)

// Múltiplos includes
.Include(x => x.ClienteNavigation)
    .ThenInclude(c => c.IdEnderecoNavigation)
.Include(x => x.ConsultoriaNavigation)
    .ThenInclude(c => c.ArquitetoRevisorNavigation)
```

### Select projection (preferido — busca só o necessário)
```csharp
return await query
    .Select(x => new ItemResponse
    {
        Id = x.Id,
        Nome = x.Nome,
        ArquitetoNome = x.ConsultoriaNavigation.ArquitetoNavigation.Nome,  // navega via EF
        Total = x.Itens.Count()                                             // agregação inline
    })
    .ToListAsync(cancellationToken);
```

---

## Convenções de nomenclatura de Response/Result

| Sufixo | Uso |
|--------|-----|
| `{Entidade}ItemResponse` | DTO de item em listagem |
| `Obter{Entidade}QueryResult` | DTO de detalhe por ID |
| `{Entidade}Response` | DTO genérico (Dapper, retornos simples) |
| `Resumo{Entidade}Response` | Resumo/dashboard |
| `Detalhes{Entidade}Response` | Detalhe completo com coleções |
| `PagedResult<T>` | Wrapper de paginação com Total + Items |
| `Resultado{Entidade}Response` | Wrapper com QtdFiltro + QtdTotal + Dados |

### Response simples (apenas propriedades auto)
```csharp
public class {Entidade}ItemResponse
{
    public Guid Id { get; set; }
    public string Nome { get; set; }
    public bool Ativo { get; set; }
}
```

### Response com coleção inicializada
```csharp
public class Detalhe{Entidade}Response
{
    public Guid Id { get; set; }
    public string Nome { get; set; }
    public ICollection<ItemResponse> Itens { get; set; } = new List<ItemResponse>();
}
```

### Response com propriedade computada
```csharp
public class ResumoConsultoriaResponse
{
    public Guid ConsultoriaId { get; set; }
    public DateTime? DataHoraConfirmacao { get; set; }
    public bool Confirmado => DataHoraConfirmacao.HasValue;  // Computed — sem setter
}
```

---

## Estratégias para não encontrado

| Situação | Abordagem |
|----------|-----------|
| Detalhe por ID | Retornar `null` — controller trata com `if (resultado is null) return NotFound();` |
| Listagem sem resultados | Retornar `IEnumerable` vazio / `PagedResult` com `Items = []` e `Total = 0` |
| Query com dado obrigatório ausente | Lançar `Exception` (raro — use nos handlers MediatR complexos) |

---

## Checklist

- [ ] Arquivo único: Query + Response + Handler no mesmo `.cs`
- [ ] Query herda `IRequest<TResponse>` — nunca `Command`
- [ ] Handler herda apenas `IRequestHandler<>` — nunca `CommandHandler`
- [ ] `AsNoTracking()` em **toda** consulta EF Core
- [ ] Projeção via `.Select()` no banco sempre que possível
- [ ] Filtros nullable com `.HasValue` antes de aplicar `.Where()`
- [ ] Paginação com `.Skip()` / `.Take()` e `CountAsync()` quando necessário
- [ ] Include somente das navegações realmente necessárias
- [ ] **Não registrar no DI** — MediatR descobre automaticamente
- [ ] Para listagens com muitos `Include`: usar two-pass (IDs primeiro)
