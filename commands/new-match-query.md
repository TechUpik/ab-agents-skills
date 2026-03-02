---
description: Cria uma nova Query no api-match (Match.Core) com Handler aninhado
argument-hint: <AcaoEntidade> [--dominio <NomeDominio>] [--paginado] [--lista]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Nova Query — api-match

Argumento recebido: **$ARGUMENTS**

## Flags disponíveis

- `--paginado` → gera query com paginação e classe Result separada (IndicePaginacao + QuantidadePaginacao + QuantidadeTotal)
- `--lista` → gera query que retorna `List<T>` sem paginação
- (sem flag) → gera query que retorna um único objeto (`FirstOrDefaultAsync`)

## Instruções

Você vai criar uma Query completa no repositório api-match seguindo os padrões do projeto.

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome da Query (ex: `GetClient` → `GetClientQuery`, entidade `Client`, domínio `Clients`)
2. Leia o diretório `Match.Core/` para identificar o domínio correto
3. Se existir `Match.Core/{Dominio}/Querys/`, leia as queries existentes para entender o padrão de parâmetros e filtros do domínio
4. Identifique quais `IRepository<T>` serão necessários

### Passo 2 — Perguntar ao usuário (se necessário)

Se o argumento não deixou claro, pergunte:
- Em qual domínio esta query pertence?
- Quais filtros/parâmetros a query recebe?
- Qual é o tipo de retorno? (entidade direta, DTO, lista, paginado)
- Precisa acessar múltiplos repositórios?

### Passo 3 — Gerar o arquivo

**Arquivo:** `Match.Core/{Dominio}/Querys/{Acao}{Entidade}Query.cs`

Estrutura obrigatória:
- Pasta chamada `Querys` (com Y)
- `{Acao}{Entidade}Query` implementa `IRequest<TResponse>`
- Para `--paginado`: adicionar classe `{Acao}{Entidade}Result` no mesmo arquivo
- `{Acao}{Entidade}QueryHandler` é **classe pública aninhada** (dentro do Query ou logo abaixo do Result)
- Handler implementa `IRequestHandler<{Acao}{Entidade}Query, TResponse>`
- IDs sempre como `string` (nunca `Guid`)
- Leitura via `FirstOrDefaultAsync()`, `ToListAsync()`, `CountAsync()` do IRepository
- Retornar `null` quando não encontrado (controller trata com `NotFound()`)
- Sem registro manual no DI (MediatR auto-descobre)

### Passo 4 — Resumo

Após criar o arquivo, exibir:
- Caminho do arquivo criado
- Exemplo de como chamar a query: `await _mediator.Send(new {Acao}{Entidade}Query(...))`
- Template do endpoint do controller correspondente
