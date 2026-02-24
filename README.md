# adb-api-skills

Plugin de skills para o repositório **rep-adb-api** da Upik.

Auxilia desenvolvedores a criar e manter Commands, Queries e Controllers seguindo os padrões **CQRS + MediatR + FluentValidation + Clean Architecture** do projeto.

---

## Instalação

O plugin usa um **marketplace local** registrado no Claude Code. O marketplace fica em `~/.claude/plugins/marketplaces/local/` e aponta via symlink para este repositório.

### 1. Clone o repositório

```bash
git clone <url-do-repositorio>
```

### 2. Crie a estrutura do marketplace local (uma vez por máquina)

```bash
mkdir -p ~/.claude/plugins/marketplaces/local/{.claude-plugin,plugins}
```

Crie o arquivo `~/.claude/plugins/marketplaces/local/.claude-plugin/marketplace.json`:

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "local",
  "description": "Plugins locais do projeto Upik",
  "owner": {
    "name": "Upik Dev Team"
  },
  "plugins": [
    {
      "name": "adb-api-skills",
      "description": "Skills para auxiliar desenvolvimento no repositório rep-adb-api (CQRS, Commands, Queries, Controllers com padrões Clean Architecture + MediatR + FluentValidation)",
      "source": "./plugins/adb-api-skills",
      "category": "development"
    }
  ]
}
```

### 3. Crie o symlink do plugin no marketplace

```bash
ln -s /caminho/para/ab-agents-skills \
  ~/.claude/plugins/marketplaces/local/plugins/adb-api-skills
```

### 4. Registre o marketplace e instale o plugin no Claude Code

```
/plugin marketplace add file://$HOME/.claude/plugins/marketplaces/local
/plugin install adb-api-skills@local
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
