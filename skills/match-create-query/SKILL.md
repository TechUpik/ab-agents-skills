---
name: match-create-query
description: This skill should be used when the user wants to create a new query, list data, get details, search or filter records, or implement a read operation in the api-match repository. Activate when the user mentions "criar query match", "nova query match", "listar match", "obter match", "buscar match", "filtrar match", "paginação match", or describes fetching/reading data in the context of the api-match/Match.Core repository.
version: 1.0.0
---

# Criar uma nova Query no api-match

## Convenção fundamental: Handler aninhado no mesmo arquivo

Toda operação de leitura segue a convenção de **arquivo único** com o Handler como **classe aninhada**:

```
Match.Core/{Dominio}/Querys/{Acao}{Entidade}Query.cs
├── class {Acao}{Entidade}Query              → parâmetros de entrada (IRequest<TResponse>)
├── class {Acao}{Entidade}Result             → DTO de retorno (quando necessário)
└── class {Acao}{Entidade}QueryHandler       → implementação (nested public class)
```

> **Pasta `Querys`** (com Y) — seguir o padrão já existente no projeto.
> **Não registrar no DI** — MediatR descobre automaticamente via Assembly scan.

---

## Tipo 1 — Busca simples por ID ou campo único

**Caminho:** `Match.Core/{Dominio}/Querys/Get{Entidade}Query.cs`

```csharp
namespace Match.Core.{Dominio}.Querys
{
    public class Get{Entidade}Query : IRequest<{Entidade}>
    {
        public string Id { get; set; }

        public Get{Entidade}Query(string id)
        {
            Id = id;
        }

        public class Get{Entidade}QueryHandler : IRequestHandler<Get{Entidade}Query, {Entidade}>
        {
            private readonly IRepository<{Entidade}> _{entidade}s;

            public Get{Entidade}QueryHandler(IRepository<{Entidade}> {entidade}s)
            {
                _{entidade}s = {entidade}s;
            }

            public async Task<{Entidade}> Handle(
                Get{Entidade}Query request,
                CancellationToken cancellationToken)
            {
                return await _{entidade}s.FirstOrDefaultAsync(x =>
                    x.Where(e => e.Id == request.Id));
            }
        }
    }
}
```

---

## Tipo 2 — Busca com filtros opcionais

```csharp
namespace Match.Core.{Dominio}.Querys
{
    public class Obter{Entidade}PorCategoriaQuery : IRequest<{Entidade}>
    {
        public string CategoriaId { get; set; }
        public string ClienteId { get; set; }

        public Obter{Entidade}PorCategoriaQuery(string categoriaId, string clienteId)
        {
            CategoriaId = categoriaId;
            ClienteId = clienteId;
        }

        public class Obter{Entidade}PorCategoriaQueryHandler
            : IRequestHandler<Obter{Entidade}PorCategoriaQuery, {Entidade}>
        {
            private readonly IRepository<{Entidade}> _{entidade}s;

            public Obter{Entidade}PorCategoriaQueryHandler(IRepository<{Entidade}> {entidade}s)
            {
                _{entidade}s = {entidade}s;
            }

            public async Task<{Entidade}> Handle(
                Obter{Entidade}PorCategoriaQuery request,
                CancellationToken cancellationToken)
            {
                return await _{entidade}s.FirstOrDefaultAsync(x =>
                    x.Where(e => e.CategoriaId == request.CategoriaId
                              && e.ClienteId == request.ClienteId));
            }
        }
    }
}
```

---

## Tipo 3 — Listagem com paginação e Result separado

Use quando a query precisa retornar **dados + metadados** (total de registros, paginação):

```csharp
namespace Match.Core.{Dominio}.Querys
{
    public class Listar{Entidade}PaginadoQuery : IRequest<Listar{Entidade}PaginadoResult>
    {
        public int IndicePaginacao { get; set; }
        public int QuantidadePaginacao { get; set; }
        public string TermoPesquisa { get; set; }

        public Listar{Entidade}PaginadoQuery(
            int indicePaginacao,
            int quantidadePaginacao,
            string termoPesquisa)
        {
            IndicePaginacao = indicePaginacao;
            QuantidadePaginacao = quantidadePaginacao;
            TermoPesquisa = termoPesquisa;
        }
    }

    // Result — classe separada no mesmo arquivo
    public class Listar{Entidade}PaginadoResult
    {
        public List<{Entidade}> Itens { get; set; }
        public int QuantidadeTotal { get; set; }
    }

    public class Listar{Entidade}PaginadoQueryHandler
        : IRequestHandler<Listar{Entidade}PaginadoQuery, Listar{Entidade}PaginadoResult>
    {
        private readonly IRepository<{Entidade}> _{entidade}s;

        public Listar{Entidade}PaginadoQueryHandler(IRepository<{Entidade}> {entidade}s)
        {
            _{entidade}s = {entidade}s;
        }

        public async Task<Listar{Entidade}PaginadoResult> Handle(
            Listar{Entidade}PaginadoQuery request,
            CancellationToken cancellationToken)
        {
            var skip = request.IndicePaginacao * request.QuantidadePaginacao;

            // Contar total antes de paginar
            var total = await _{entidade}s.CountAsync(x =>
            {
                var q = x.AsQueryable();
                if (!string.IsNullOrEmpty(request.TermoPesquisa))
                    q = q.Where(e => e.Nome.Contains(request.TermoPesquisa));
                return q;
            });

            // Buscar página
            var itens = await _{entidade}s.ToListAsync(x =>
            {
                var q = x.AsQueryable();
                if (!string.IsNullOrEmpty(request.TermoPesquisa))
                    q = q.Where(e => e.Nome.Contains(request.TermoPesquisa));
                return q
                    .OrderByDescending(e => e.Date)
                    .Skip(skip)
                    .Take(request.QuantidadePaginacao);
            });

            return new Listar{Entidade}PaginadoResult
            {
                Itens = itens,
                QuantidadeTotal = total
            };
        }
    }
}
```

---

## Tipo 4 — Query que acessa múltiplos repositórios

```csharp
public class GetPerfilClienteQueryHandler : IRequestHandler<GetPerfilClienteQuery, PerfilClienteDTO>
{
    private readonly IRepository<Client> _clients;
    private readonly IRepository<Response> _responses;
    private readonly IRepository<EnvironmentCategory> _categories;

    public GetPerfilClienteQueryHandler(
        IRepository<Client> clients,
        IRepository<Response> responses,
        IRepository<EnvironmentCategory> categories)
    {
        _clients = clients;
        _responses = responses;
        _categories = categories;
    }

    public async Task<PerfilClienteDTO> Handle(
        GetPerfilClienteQuery request,
        CancellationToken cancellationToken)
    {
        // Buscar em paralelo quando possível
        var clienteTask = _clients.FirstOrDefaultAsync(x => x.Where(c => c.Id == request.ClienteId));
        var respostaTask = _responses.FirstOrDefaultAsync(x => x.Where(r => r.ClienteId == request.ClienteId));

        await Task.WhenAll(clienteTask, respostaTask);

        var cliente = clienteTask.Result;
        var resposta = respostaTask.Result;

        if (cliente is null || resposta is null) return null;

        var categoria = await _categories.FirstOrDefaultAsync(x =>
            x.Where(c => c.Id == resposta.CategoryId));

        return new PerfilClienteDTO
        {
            Cliente = cliente,
            Resposta = resposta,
            AmbienteCategoria = categoria
        };
    }
}
```

---

## Métodos do IRepository\<T\> para leitura

```csharp
// Buscar um registro
await _repo.FirstOrDefaultAsync(x => x.Where(e => e.Id == id));

// Buscar lista
await _repo.ToListAsync(x => x.Where(e => e.ClienteId == clienteId).OrderBy(e => e.Date));

// Contar
await _repo.CountAsync(x => x.Where(e => e.Ativo));

// Verificar existência
await _repo.AnyAsync(x => x.Where(e => e.Email == email));

// Agregação
await _repo.SumAsync(x => x.Where(e => e.ClienteId == id).Select(e => e.Valor));
```

---

## Retornar null vs lançar exceção

| Situação | Abordagem |
|----------|-----------|
| Detalhe não encontrado | Retornar `null` — controller trata com `NotFound()` |
| Listagem sem resultados | Retornar lista vazia / result com `Itens = []` e `QuantidadeTotal = 0` |
| Dado obrigatório ausente | Lançar `ArgumentException` no início do Handle |

---

## Checklist

- [ ] Arquivo único em `Match.Core/{Dominio}/Querys/{Acao}{Entidade}Query.cs`
- [ ] Pasta chamada `Querys` (com Y)
- [ ] Query implementa `IRequest<TResponse>` diretamente (sem classe base)
- [ ] Handler é **classe pública aninhada** dentro do arquivo (pode ser fora do Query se houver Result)
- [ ] Handler implementa `IRequestHandler<TQuery, TResponse>`
- [ ] IDs são `string` (não `Guid`)
- [ ] Para queries paginadas: classe `Result` separada no mesmo arquivo
- [ ] Retornar `null` quando não encontrado (não lançar exceção)
- [ ] **Não registrar no DI** — MediatR descobre automaticamente
