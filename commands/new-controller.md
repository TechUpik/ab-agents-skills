---
description: Cria um novo Controller no rep-adb-api com endpoints CRUD básicos
argument-hint: <NomeEntidade> [--readonly] [--admin]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Novo Controller — rep-adb-api

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar um Controller completo no repositório rep-adb-api seguindo os padrões do projeto.

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome da entidade (ex: `Produto` → `ProdutoController`)
2. Determine as flags:
   - `--readonly` → criar apenas endpoints GET (sem POST/PUT/PATCH/DELETE)
   - `--admin` → adicionar prefixo `painel-admin/` nas rotas e rota base com `/admin`
3. Leia `src/AdB.API/Controllers/` para ver controllers existentes e confirmar padrões
4. Leia `src/AdB.API/Controllers/v1/MainController.cs` para entender a classe base
5. Verifique quais Commands e Queries já existem para esta entidade em `src/AdB.Application/`

### Passo 2 — Perguntar ao usuário (se necessário)

Se necessário, pergunte:
- Qual deve ser o segmento de rota? (ex: `produtos`, `consultoria-config`)
- O controller é público (`[AllowAnonymous]`) ou requer autenticação (`[Authorize]`)?
- Quais endpoints são necessários? (listar, obterPorId, criar, atualizar, excluir, outros?)
- Há alguma dependência adicional além do IMediator? (queries Dapper, cache, etc.)

### Passo 3 — Gerar o arquivo

**Arquivo:** `src/AdB.API/Controllers/{Entidade}Controller.cs`

Estrutura obrigatória:
- Herdar de `MainController`
- `[Route("api/v{version:apiVersion}/{recurso}")]`
- `[OpenApiTag("{Entidade}")]`
- `[Authorize]` (ou `[AllowAnonymous]` se público)
- Injetar `IMediator` no construtor
- Cada endpoint com `[ProducesResponseType]` e `[OpenApiOperation]`
- Retornos via `CustomResponse()` ou `CustomResponse(resultado)`
- Try/catch em endpoints que não passam CommandResult diretamente

Endpoints a gerar (baseado nas flags e Commands/Queries existentes):
- `GET /listar` → `ListarAsync` com query params de filtro
- `GET /{id:guid}` → `ObterPorId` retornando `NotFound()` se null
- `POST /criar` → Recebe command no body, injeta dados do token
- `PUT /{id:guid}` → Recebe command no body, injeta ID da rota
- `PATCH /{id:guid}/cancelar` (ou outro verbo específico) → Ação de patch
- `DELETE /{id:guid}` → Soft delete ou exclusão

Se `--readonly`: apenas `GET /listar` e `GET /{id:guid}`
Se `--admin`: usar `painel-admin/` como prefixo nos paths

### Passo 4 — Resumo

Após criar o arquivo, exibir:
- Arquivo criado com o caminho completo
- Lista de endpoints gerados com seus paths completos
- Commands/Queries que existem e foram referenciados
- Commands/Queries que **não existem** e precisam ser criados (sugerir usar `/new-command` ou `/new-query`)
- Lembrar de verificar autenticação e autorização por perfil
