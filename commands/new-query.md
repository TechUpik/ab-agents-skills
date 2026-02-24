---
description: Cria uma nova Query no rep-adb-api (MediatR EF Core ou Dapper)
argument-hint: <VerbEntidade> [--dapper] [--paginado] [--detalhe] [--dashboard]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Nova Query — rep-adb-api

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar uma Query completa no repositório rep-adb-api. Determine o tipo baseado nas flags:

| Flag | Tipo |
|------|------|
| (nenhuma) | MediatR EF Core — listagem simples |
| `--paginado` | MediatR EF Core — listagem com PagedResult<T> |
| `--detalhe` | MediatR EF Core — Obter por ID |
| `--dapper` | Dapper — serviço com SQL em SQLQueriesResource |
| `--dashboard` | Dapper — dashboard/relatório com agrupamento |

---

## Passo 1 — Contexto

1. Extraia do argumento o nome da Query (ex: `ListarConsultorias`, `ObterArquiteto`, `DashboardCliente`)
2. Leia `src/AdB.Application/Queries/` para ver queries similares existentes
3. Para EF Core: leia `src/AdB.Data/Context/AdBContext.cs` para confirmar nomes dos DbSets e navegações
4. Para Dapper: leia `src/AdB.Application/Queries/` para ver a estrutura de Queries de serviço existentes
5. Leia `src/AdB.API/Configurations/DependencyInjections/QueriesDIConfig.cs` (somente para Dapper)

---

## Passo 2 — Perguntar ao usuário (se necessário)

Pergunte apenas o que não ficou claro no argumento:

- O que esta query retorna? (lista, item único, dashboard, contagem)
- Quais filtros são necessários? Quais são obrigatórios vs opcionais?
- Precisa de paginação? Com contagem total (PagedResult) ou sem?
- Quais campos devem aparecer na response?
- Há relacionamentos/navegações que precisam ser incluídos?
- A query é nova ou é uma evolução de uma query existente?

---

## Passo 3 — Gerar os arquivos

### Para MediatR EF Core (padrão, `--paginado`, `--detalhe`)

**Arquivo único:** `src/AdB.Application/Queries/{Modulo}/{Entidade}/{Verbo}{Entidade}Query.cs`

Contém no mesmo arquivo:
1. `{Verbo}{Entidade}Query : IRequest<TResponse>` — parâmetros/filtros
2. `{Entidade}ItemResponse` ou `Obter{Entidade}QueryResult` — DTO de retorno
3. `{Verbo}{Entidade}QueryHandler : IRequestHandler<TQuery, TResponse>` — implementação

Padrões obrigatórios:
- `AsNoTracking()` em todas as consultas EF Core
- Filtros nullable com `if (request.Prop.HasValue)` antes de aplicar `.Where()`
- String com `if (!string.IsNullOrWhiteSpace(request.Termo))`
- Projeção via `.Select()` no banco sempre que possível
- `--detalhe`: Retornar `null` se não encontrado (controller trata com `NotFound()`)
- `--paginado`: `CountAsync()` antes de `Skip().Take()`, retornar `PagedResult<T>` com `Total`
- Listagem com muitos `Include`: usar two-pass (buscar IDs paginados primeiro, depois carregar completo)

**Não registrar no DI** — MediatR descobre automaticamente.

---

### Para Dapper (`--dapper`, `--dashboard`)

**Arquivo 1 — Interface:** `src/AdB.Application/Queries/I{Entidade}Queries.cs`

**Arquivo 2 — Implementação:** `src/AdB.Application/Queries/{Entidade}Queries.cs`
- Construtor recebe `AdBDapperContext _adBDapperContext`
- Cada método: `string query = SQLQueriesResource.{NomeDoSQL};`
- `using var conn = _adBDapperContext.DapperConnection;`
- Parâmetros via objeto anônimo: `new { ConsultoriaId = id, TermoPesquisa = $"%{termo}%" }`

**SQL Resource:** Adicionar entrada no `src/AdB.Application/Resources/SQLQueriesResource.resx`
- Key: `Obter{Entidade}PorXxx` ou `Listar{Entidade}Xxx`
- Valor: SQL completo com parâmetros `@NomeParam`
- Depois de adicionar: avisar ao dev para recompilar para regenerar o Designer.cs

**`--dashboard`:** Após o `QueryAsync`, aplicar `.GroupBy()` em memória para agregação.

**Multi-mapping** (quando SQL tem JOINs com objetos distintos):
- Usar `QueryAsync<T1, T2, ..., TReturn>(sql, lambda, splitOn: "Col1,Col2")`
- Lambda acumuladora: verificar duplicatas com `.Any(x => x.Id == item.Id)` antes de `.Add()`
- Inicializar coleções nos DTOs raiz: `= new List<T>()`

**Edição obrigatória:** `src/AdB.API/Configurations/DependencyInjections/QueriesDIConfig.cs`
```csharp
services.AddScoped<I{Entidade}Queries, {Entidade}Queries>();
```

---

## Passo 4 — Resumo

Após criar os arquivos, exibir:
- Arquivos criados com caminhos completos
- Template de como chamar a query no controller
- Para Dapper: lembrar que precisa adicionar o SQL no `.resx` e recompilar
- Para multi-mapping: listar os aliases de coluna necessários no SQL para o `splitOn`
