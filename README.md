# adb-api-skills

Plugin de skills para o repositório **rep-adb-api** da Upik.

Auxilia desenvolvedores a criar e manter Commands, Queries e Controllers seguindo os padrões **CQRS + MediatR + FluentValidation + Clean Architecture** do projeto.

---

## Instalação

### Via repositório remoto

No Claude Code, execute os dois comandos abaixo (substitua pela URL real do repositório):

```
/plugin marketplace add https://github.com/upik/ab-agents-skills
/plugin install adb-api-skills@ab-agents-skills
```

> Repositórios **privados** funcionam normalmente, desde que o `git clone` já funcione no seu terminal (SSH, GitHub CLI ou `GITHUB_TOKEN` configurado).

### Via clone local

```
/plugin marketplace add ./ab-agents-skills
/plugin install adb-api-skills@ab-agents-skills
```

Após a instalação, reinicie o Claude Code. As skills e commands estarão disponíveis automaticamente.

---

## Skills (ativadas automaticamente por contexto)

| Skill | Quando é ativada |
|-------|-----------------|
| `adb-patterns` | Perguntas sobre padrões, arquitetura, convenções gerais do projeto |
| `adb-create-command` | Criação de novo command / operação de escrita |
| `adb-create-query` | Criação de query MediatR com EF Core (listagem, detalhe, paginação) |
| `adb-dapper-query` | Criação de query Dapper (SQL complexo, dashboard, multi-mapping) |
| `adb-create-controller` | Criação de novo controller / endpoints HTTP |

## Commands (invocados via `/`)

| Command | Flags | Descrição |
|---------|-------|-----------|
| `/new-command <VerbEntidade>` | — | Gera command + handler + DI |
| `/new-query <VerbEntidade>` | `--dapper` `--paginado` `--detalhe` `--dashboard` | Gera query (MediatR ou Dapper) |
| `/new-controller <Entidade>` | `--readonly` `--admin` | Gera controller com endpoints CRUD |
| `/new-feature <NomeDaFeature>` | — | Feature completa (command + query + controller) |

## Agents (invocados pelo Claude)

| Agent | Função |
|-------|--------|
| `adb-reviewer` | Revisa código contra os padrões do repositório |

---

## Padrões do Repositório

### Arquitetura
```
AdB.Domain        → Entidades e domínio
AdB.Core          → Base classes (Command, CommandHandler, Message)
AdB.Data          → Repositórios, EF Core (AdBContext), Dapper (AdBDapperContext)
AdB.Application   → Commands, Queries, Services
AdB.API           → Controllers, DI configs
```

### Dois padrões de Query

| | MediatR + EF Core | Dapper Service |
|-|-------------------|----------------|
| **Arquivo** | Único (`Query + Handler + Response`) | Interface + Implementação |
| **DI** | Automático (não registrar) | Manual em `QueriesDIConfig.cs` |
| **SQL** | LINQ via EF Core | `SQLQueriesResource.resx` |
| **Quando** | Queries padrão, filtros dinâmicos | SQL complexo, dashboards, relatórios |

### Convenções de nomenclatura
- **Commands:** `{Verbo}{Entidade}Command` → `RegistrarAgendamentoCommand`
- **Handlers:** `{Entidade}CommandHandler` → `AgendamentoCommandHandler`
- **Queries MediatR:** `{Verbo}{Entidade}Query` → `ListarArquitetosQuery`
- **Queries Dapper:** `{Entidade}Queries` → `ClienteQueries`
- **Controllers:** `{Entidade}Controller` → `ProdutoController`
- **Response (listagem):** `{Entidade}ItemResponse`
- **Result (detalhe):** `Obter{Entidade}QueryResult`
- **Paginado:** `PagedResult<T>` ou `Resultado{Entidade}Response`

### Arquivos de DI
- Commands → `src/AdB.API/Configurations/DependencyInjections/CommandsDIConfig.cs`
- Queries Dapper → `src/AdB.API/Configurations/DependencyInjections/QueriesDIConfig.cs`
- Queries MediatR → auto-descobertas (não precisam de registro manual)
