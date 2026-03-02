---
description: Cria um novo Controller no api-match (Match.Api) com endpoints padrão
argument-hint: <NomeEntidade> [--b2b] [--readonly] [--cache]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Novo Controller — api-match

Argumento recebido: **$ARGUMENTS**

## Flags disponíveis

- `--b2b` → controller autenticado em `Controllers/B2b/` com `[Authorize]`
- `--readonly` → gera apenas endpoints GET (sem POST/PUT/DELETE)
- `--cache` → inclui `IMemoryCache` para cache dos endpoints de listagem

## Instruções

Você vai criar um Controller no repositório api-match seguindo os padrões do projeto.

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome da entidade (ex: `Produto` → `ProdutoController`)
2. Leia `Match.Api/Controllers/` (e `B2b/` se flag `--b2b`) para ver controllers existentes e entender o padrão de rota
3. Leia `Match.Core/{Dominio}/Commands/` e `Match.Core/{Dominio}/Querys/` para identificar quais Commands/Queries já existem para esta entidade
4. Leia `Match.Api/Models/` para ver quais models já existem

### Passo 2 — Perguntar ao usuário (se necessário)

Se o argumento não deixou claro, pergunte:
- Quais endpoints são necessários? (listar, obter por ID, criar, atualizar, excluir, upload)
- O controller é público ou autenticado (B2B)?
- Quais Commands/Queries o controller vai usar?
- Precisa de cache em algum endpoint?

### Passo 3 — Gerar os arquivos

**Arquivo 1:** `Match.Api/Controllers/{Entidade}Controller.cs`
(ou `Match.Api/Controllers/B2b/{Entidade}B2bController.cs` com `--b2b`)

Estrutura obrigatória:
- Herda de `Controller` (não `ControllerBase`)
- Atributos `[ApiController]` e `[Route("api/[controller]")]`
- `[Authorize]` apenas em controllers B2B
- Todo endpoint tem `try/catch` com `ModelState.AddModelError` + `BadRequest(ModelState)`
- Retornos via `Ok()`, `NotFound()`, `BadRequest()` ou `NoContent()`
- IDs de rota como `string` (nunca `Guid`)

**Arquivo 2 (se necessário):** `Match.Api/Models/{Entidade}Model.cs`
- Criar model de request se não existir

### Passo 4 — Resumo

Após criar os arquivos, exibir:
- Arquivos criados com seus caminhos
- Tabela de endpoints gerados com método HTTP e rota
- Lista de Commands/Queries que precisam ser criados (se ainda não existirem)
