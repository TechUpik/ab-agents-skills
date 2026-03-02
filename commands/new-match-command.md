---
description: Cria um novo Command completo no api-match (Match.Core) com Handler aninhado
argument-hint: <AcaoEntidade> [--dominio <NomeDominio>]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Novo Command — api-match

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar um Command completo no repositório api-match seguindo os padrões do projeto.

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome do Command (ex: `CreateClient` → `CreateClientCommand`, entidade `Client`, domínio `Clients`)
2. Leia o diretório `Match.Core/` para identificar o domínio correto onde o command deve ser criado
3. Se existir pasta `Match.Core/{Dominio}/Commands/`, leia os commands existentes para entender o padrão de nomenclatura e as dependências já usadas no domínio
4. Identifique quais `IRepository<T>` serão necessários injetando no handler

### Passo 2 — Perguntar ao usuário (se necessário)

Se o argumento não deixou claro, pergunte:
- Em qual domínio este command pertence? (ex: Clients, Ambientes, Auth, Respostas)
- Quais propriedades o command precisa receber?
- Qual é o tipo de retorno esperado? (entidade, DTO, bool, string, Unit)
- Há alguma regra de negócio específica? (verificar duplicidade, encadear outros commands, etc.)

### Passo 3 — Gerar o arquivo

**Arquivo:** `Match.Core/{Dominio}/Commands/{Acao}{Entidade}Command.cs`

Estrutura obrigatória:
- `{Acao}{Entidade}Command` implementa `IRequest<TResponse>`
- `{Acao}{Entidade}CommandHandler` é **classe pública aninhada** dentro do Command
- Handler implementa `IRequestHandler<{Acao}{Entidade}Command, TResponse>`
- IDs sempre como `string` (nunca `Guid`)
- Persistência via `SaveOrUpdateAsync()` com filtro de upsert
- Sem FluentValidation — usar guard clauses simples se necessário
- Sem registro manual no DI (MediatR auto-descobre)

### Passo 4 — Resumo

Após criar o arquivo, exibir:
- Caminho do arquivo criado
- Template do endpoint que o desenvolvedor precisará criar no controller
- Exemplo de como chamar o command via `_mediator.Send(new {Acao}{Entidade}Command(...))`
- Lembretes de qualquer ajuste manual necessário (ex: registrar nova coleção MongoDB se a entidade for nova)
