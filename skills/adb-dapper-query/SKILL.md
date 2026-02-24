---
name: adb-dapper-query
description: This skill should be used when the user wants to create a complex SQL query, a dashboard, a report, or any read operation that uses raw SQL with Dapper in the rep-adb-api repository. Activate when the user mentions "Dapper", "query SQL", "SQL complexo", "relatório", "dashboard", "consulta otimizada", "multi-mapping", "splitOn", "SQLQueriesResource", "AdBDapperContext", "IClienteQueries", "join SQL", or describes a read operation where EF Core would be too slow or too complex.
version: 1.0.0
---

# Criar Query Dapper no rep-adb-api

## Quando usar Dapper vs MediatR/EF Core

| Use Dapper quando... | Use EF Core (MediatR) quando... |
|----------------------|---------------------------------|
| SQL com múltiplos JOINs complexos | Filtros dinâmicos simples |
| Dashboards com agregações | CRUD e listagens padrão |
| Relatórios com agrupamentos | Queries com 1-2 relacionamentos |
| Performance crítica | Prototipagem rápida |
| Subqueries correlacionadas | Entidade única com Include |

---

## Estrutura de arquivos

```
src/AdB.Application/Queries/
├── I{Entidade}Queries.cs         → Interface (contrato público)
└── {Entidade}Queries.cs          → Implementação com Dapper

src/AdB.API/Configurations/DependencyInjections/
└── QueriesDIConfig.cs            → Registro obrigatório no DI

src/AdB.Application/Resources/
├── SQLQueriesResource.resx       → Arquivo de strings SQL
└── SQLQueriesResource.Designer.cs→ Gerado automaticamente
```

---

## Padrão 1 — Query simples (um tipo de retorno)

### Interface

```csharp
// Arquivo: src/AdB.Application/Queries/I{Entidade}Queries.cs
namespace AdB.Application.Queries;

public interface I{Entidade}Queries
{
    Task<IEnumerable<{Entidade}Response>> ObterPorConsultoriaId(Guid consultoriaId);
    Task<{Entidade}DetalheResponse?> ObterDetalhePorId(Guid id);
    Task<IEnumerable<{Entidade}Response>> ListarComFiltro(Guid? filtroId, DateTime? dataInicio, string? termo);
}
```

### Implementação

```csharp
// Arquivo: src/AdB.Application/Queries/{Entidade}Queries.cs
using AdB.Data.Context;
using Dapper;

namespace AdB.Application.Queries;

public class {Entidade}Queries : I{Entidade}Queries
{
    private readonly AdBDapperContext _adBDapperContext;

    public {Entidade}Queries(AdBDapperContext adBDapperContext)
    {
        _adBDapperContext = adBDapperContext;
    }

    public async Task<IEnumerable<{Entidade}Response>> ObterPorConsultoriaId(Guid consultoriaId)
    {
        string query = SQLQueriesResource.Obter{Entidade}PorConsultoriaId;

        using var conn = _adBDapperContext.DapperConnection;
        return await conn.QueryAsync<{Entidade}Response>(
            query,
            new { ConsultoriaId = consultoriaId }
        );
    }

    public async Task<{Entidade}DetalheResponse?> ObterDetalhePorId(Guid id)
    {
        string query = SQLQueriesResource.Obter{Entidade}DetalhesPorId;

        using var conn = _adBDapperContext.DapperConnection;
        return await conn.QueryFirstOrDefaultAsync<{Entidade}DetalheResponse>(
            query,
            new { Id = id }
        );
    }

    public async Task<IEnumerable<{Entidade}Response>> ListarComFiltro(
        Guid? filtroId,
        DateTime? dataInicio,
        string? termo)
    {
        string query = SQLQueriesResource.Listar{Entidade}ComFiltro;

        using var conn = _adBDapperContext.DapperConnection;
        return await conn.QueryAsync<{Entidade}Response>(
            query,
            new
            {
                FiltroId = filtroId,
                DataInicio = dataInicio,
                TermoPesquisa = $"%{termo}%"  // LIKE pattern direto no param
            }
        );
    }
}
```

---

## Padrão 2 — Multi-mapping (múltiplos tipos em um único resultado)

Use quando o SQL retorna colunas de múltiplas entidades relacionadas que precisam ser mapeadas em objetos distintos.

```csharp
public async Task<DetalhesConsultoriaResponse?> ObterDetalhes(Guid clienteId, Guid consultoriaId)
{
    string query = SQLQueriesResource.ObterDetalhesConsultoria;

    DetalhesConsultoriaResponse? resultado = null;

    using var conn = _adBDapperContext.DapperConnection;

    // QueryAsync<T1, T2, T3, TReturn> — até 7 tipos suportados
    await conn.QueryAsync<
        DetalhesConsultoriaResponse,      // Tipo 1 — objeto raiz
        ArquivosRecebidosResponse,         // Tipo 2 — relacionado
        ArquivosEntregaResponse,           // Tipo 3 — relacionado
        Tour360Response,                   // Tipo 4 — relacionado
        DetalhesConsultoriaResponse        // TReturn — tipo de retorno da lambda
    >(
        query,
        // Lambda: acumulador — chamado para cada linha do resultado
        (detalhe, arquivosRecebidos, arquivosEntrega, tour360) =>
        {
            // Inicializar objeto raiz na primeira linha
            if (resultado is null)
                resultado = detalhe;

            // Deduplicação antes de adicionar coleções
            if (arquivosRecebidos is not null &&
                !resultado.ArquivosRecebidos.Any(x => x.Id == arquivosRecebidos.Id))
            {
                resultado.ArquivosRecebidos.Add(arquivosRecebidos);
            }

            if (arquivosEntrega is not null &&
                !resultado.ArquivosEntrega.Any(x => x.Id == arquivosEntrega.Id))
            {
                arquivosEntrega.AtualizarTipo();   // método de transformação
                resultado.ArquivosEntrega.Add(arquivosEntrega);
            }

            if (tour360 is not null &&
                !resultado.Tours360.Any(x => x.Id == tour360.Id))
            {
                resultado.Tours360.Add(tour360);
            }

            return detalhe;
        },
        // splitOn: colunas que indicam início de cada novo objeto (na ordem dos tipos)
        splitOn: "ArquivosRecebidosId,ArquivosEntregaId,Tour360Id",
        param: new { ClienteId = clienteId, ConsultoriaId = consultoriaId }
    );

    return resultado;
}
```

### Regras do splitOn
- `splitOn` lista os **nomes das colunas** no resultado SQL que indicam onde começa cada novo tipo
- A ordem de `splitOn` deve corresponder à ordem dos tipos genéricos em `QueryAsync<>`
- Se o SQL usa alias de coluna, o alias deve coincidir com o nome no `splitOn`
- O primeiro tipo **não** tem entrada no `splitOn` (é sempre o início)

### SQL correspondente
```sql
SELECT
    c.Id,
    c.DataAgendamento,
    -- splitOn: "ArquivosRecebidosId"
    ar.Id AS ArquivosRecebidosId,
    ar.NomeArquivo,
    -- splitOn: "ArquivosEntregaId"
    ae.Id AS ArquivosEntregaId,
    ae.Url,
    -- splitOn: "Tour360Id"
    t.Id AS Tour360Id,
    t.LinkTour
FROM Consultoria c
LEFT JOIN ArquivosRecebidos ar ON ar.ConsultoriaId = c.Id AND ar.Excluido = 0
LEFT JOIN ArquivosEntrega ae ON ae.ConsultoriaId = c.Id
LEFT JOIN Tour360 t ON t.ConsultoriaId = c.Id
WHERE c.ClienteId = @ClienteId
  AND c.Id = @ConsultoriaId
```

---

## Padrão 3 — Dashboard com agrupamento em memória

```csharp
public async Task<DashboardResponse> ObterDashboard(Guid clienteId)
{
    string query = SQLQueriesResource.ObterDashboard{Entidade};

    using var conn = _adBDapperContext.DapperConnection;

    // Resultado flat do banco
    var linhas = await conn.QueryAsync<{Entidade}LinhaResponse>(
        query,
        new { ClienteId = clienteId }
    );

    // Agrupamento e agregação em memória via LINQ
    var grupos = linhas.GroupBy(l => l.Categoria);

    var resultado = new DashboardResponse
    {
        Total = linhas.Count(),
        Categorias = new List<CategoriaResponse>()
    };

    foreach (var grupo in grupos)
    {
        resultado.Categorias.Add(new CategoriaResponse
        {
            Nome = grupo.Key,
            Quantidade = grupo.Count(),
            ValorTotal = grupo.Sum(l => l.Valor),
            Itens = grupo.OrderBy(l => l.Nome).ToList()
        });
    }

    resultado.Categorias = resultado.Categorias.OrderBy(c => c.Nome).ToList();

    return resultado;
}
```

---

## Response DTOs para Dapper

Dapper mapeia **por nome de coluna** → nomes das propriedades devem coincidir com aliases no SQL.

### DTO simples
```csharp
public class {Entidade}Response
{
    public Guid Id { get; set; }
    public string Nome { get; set; }
    public decimal Valor { get; set; }
    public DateTime DataCriacao { get; set; }
}
```

### DTO com coleções (para multi-mapping)
```csharp
public class Detalhe{Entidade}Response
{
    public Guid ConsultoriaId { get; set; }
    public string NomeProduto { get; set; }

    // Inicializar coleções para evitar NullReferenceException no acumulador
    public ICollection<Item1Response> Itens1 { get; set; } = new List<Item1Response>();
    public ICollection<Item2Response> Itens2 { get; set; } = new List<Item2Response>();
}
```

### DTO com propriedade computada
```csharp
public class ResumoResponse
{
    public Guid ConsultoriaId { get; set; }
    public bool ConsultoriaAntiga { get; set; } = false;   // default
    public DateTime? DataConfirmacao { get; set; }
    public bool Confirmado => DataConfirmacao.HasValue;    // computed — não mapeado pelo Dapper
}
```

---

## SQL Resource File

**Arquivo:** `src/AdB.Application/Resources/SQLQueriesResource.resx`

Cada SQL é adicionado como uma entrada de string:
- **Nome (key):** `Obter{Entidade}{Descricao}` — ex: `ObterOrcamentosPorConsultoriaId`
- **Valor:** O SQL completo

```xml
<!-- No .resx, adicionar entrada: -->
<data name="Obter{Entidade}PorConsultoriaId" xml:space="preserve">
  <value>
    SELECT
        o.Id,
        o.Valor,
        o.Descricao,
        o.DataCriacao
    FROM Orcamento o
    WHERE o.ConsultoriaId = @ConsultoriaId
      AND o.Excluido = 0
    ORDER BY o.DataCriacao DESC
  </value>
</data>
```

**Após adicionar no .resx**, o Designer.cs é regenerado automaticamente e a propriedade fica disponível:
```csharp
string query = SQLQueriesResource.Obter{Entidade}PorConsultoriaId;
```

> **Nunca colocar SQL inline** no arquivo `.cs` — sempre em `SQLQueriesResource`.

---

## Registro no DI (obrigatório)

**Arquivo:** `src/AdB.API/Configurations/DependencyInjections/QueriesDIConfig.cs`

```csharp
public static void RegisterQueriesServices(this IServiceCollection services, IConfiguration configuration)
{
    // ... registros existentes ...

    // Adicionar:
    services.AddScoped<I{Entidade}Queries, {Entidade}Queries>();
}
```

---

## Injeção no Controller

```csharp
[Route("api/v{version:apiVersion}/{recurso}")]
public class {Entidade}Controller : MainController
{
    private readonly IMediator _mediator;
    private readonly I{Entidade}Queries _{entidade}Queries;

    public {Entidade}Controller(
        IMediator mediator,
        I{Entidade}Queries {entidade}Queries)
    {
        _mediator = mediator;
        _{entidade}Queries = {entidade}Queries;
    }

    [HttpGet("dashboard")]
    public async Task<ActionResult> Dashboard([FromQuery] Guid consultoriaId)
    {
        try
        {
            var resultado = await _{entidade}Queries.ObterDashboard(consultoriaId);
            return CustomResponse(resultado);
        }
        catch (Exception ex)
        {
            AdicionarErroTratado("erro", ex.Message);
            return CustomResponse();
        }
    }
}
```

---

## Métodos Dapper — referência rápida

| Método | Quando usar |
|--------|-------------|
| `QueryAsync<T>()` | Lista de resultados, tipo único |
| `QueryFirstOrDefaultAsync<T>()` | Resultado único, retorna `null` se não encontrar |
| `QueryFirstAsync<T>()` | Resultado único, lança exceção se não encontrar |
| `QueryAsync<T1,T2,...,TReturn>()` | Multi-mapping — múltiplos tipos por linha |
| `ExecuteAsync()` | INSERT/UPDATE/DELETE via Dapper (raro — use EF Core) |
| `ExecuteScalarAsync<T>()` | Retorna valor único (COUNT, SUM, etc.) |

---

## Checklist

- [ ] Interface `I{Entidade}Queries` criada em `Queries/`
- [ ] Implementação `{Entidade}Queries` usando `AdBDapperContext`
- [ ] SQL adicionado no `SQLQueriesResource.resx` (nunca inline)
- [ ] `using var conn = _adBDapperContext.DapperConnection;` em cada método
- [ ] Parâmetros passados via objeto anônimo `new { Param = value }`
- [ ] Coleções nos DTOs inicializadas (`= new List<T>()`) nos de multi-mapping
- [ ] `splitOn` corresponde aos aliases de coluna no SQL (para multi-mapping)
- [ ] Deduplicação no acumulador (`!collection.Any(x => x.Id == item.Id)`)
- [ ] Registro adicionado em `QueriesDIConfig.cs`
