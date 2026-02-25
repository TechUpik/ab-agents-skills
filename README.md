# upik-agents-skills

Plugin de skills para os repositórios da Upik — backend e frontend.

Auxilia desenvolvedores a criar e manter código seguindo os padrões estabelecidos em cada projeto:
- **rep-adb-api** (C# .NET) → Commands, Queries, Controllers com CQRS + MediatR + Clean Architecture
- **rep-upik-web** (Vue.js 2) → Views, Componentes e Modais com Options API + Bootstrap Vue
- **ab-upik-nova-jornada** (Next.js) → Pages, Componentes e Services com React + TypeScript + Styled-Components

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
ln -s $(pwd) ~/.claude/plugins/marketplaces/local/plugins/adb-api-skills
```

> Execute dentro do diretório clonado (`ab-agents-skills`), ou substitua `$(pwd)` pelo caminho absoluto completo (ex: `/Users/seu-usuario/...`). Não use `~` ou variáveis — o symlink precisa de um caminho absoluto resolvido.

### 4. Registre o marketplace e instale o plugin no Claude Code

Execute os dois comandos **separadamente** dentro do Claude Code:

```
/plugin marketplace add ~/.claude/plugins/marketplaces/local
```

Depois:

```
/plugin install adb-api-skills@local
```

> **Atenção:** não cole os dois comandos juntos — execute um de cada vez.

Após a instalação, reinicie o Claude Code. As skills e commands estarão disponíveis automaticamente.

---

## Skills (ativadas automaticamente por contexto)

### Backend — rep-adb-api (C# .NET)

| Skill | Quando é ativada |
|-------|-----------------|
| `adb-patterns` | Perguntas sobre padrões, arquitetura, convenções gerais |
| `adb-create-command` | Criação de novo command / operação de escrita |
| `adb-create-query` | Criação de query MediatR com EF Core (listagem, detalhe, paginação) |
| `adb-dapper-query` | Criação de query Dapper (SQL complexo, dashboard, multi-mapping) |
| `adb-create-controller` | Criação de novo controller / endpoints HTTP |

### Frontend Vue — rep-upik-web (Vue.js 2)

| Skill | Quando é ativada |
|-------|-----------------|
| `vue-patterns` | Perguntas sobre padrões, arquitetura, convenções do projeto Vue |
| `vue-create-view` | Criação de nova view/página com rota |
| `vue-create-component` | Criação de novo componente Vue reutilizável |
| `vue-create-modal` | Criação de novo modal Bootstrap Vue |

### Frontend Next.js — ab-upik-nova-jornada (Next.js + TypeScript)

| Skill | Quando é ativada |
|-------|-----------------|
| `nextjs-patterns` | Perguntas sobre padrões, arquitetura, convenções do projeto Next.js |
| `nextjs-create-page` | Criação de nova página com template e roteamento |
| `nextjs-create-component` | Criação de componente React com styled-components |
| `nextjs-create-service` | Criação ou expansão de service layer com axios |

## Commands (invocados via `/`)

### Backend
| Command | Flags | Descrição |
|---------|-------|-----------|
| `/new-command <VerbEntidade>` | — | Gera command + handler + DI |
| `/new-query <VerbEntidade>` | `--dapper` `--paginado` `--detalhe` `--dashboard` | Gera query (MediatR ou Dapper) |
| `/new-controller <Entidade>` | `--readonly` `--admin` | Gera controller com endpoints CRUD |
| `/new-feature <NomeDaFeature>` | — | Feature completa (command + query + controller) |

### Frontend Vue
| Command | Flags | Descrição |
|---------|-------|-----------|
| `/new-vue-view <NomeDaView>` | `--publica` | Gera view + rota no router.js |
| `/new-vue-component <NomeComponente>` | `--feature <Feature>` | Gera componente Vue reutilizável |
| `/new-vue-modal <NomeModal>` | `--feature <Feature>` | Gera modal Bootstrap Vue completo |

### Frontend Next.js
| Command | Flags | Descrição |
|---------|-------|-----------|
| `/new-nextjs-page <nome-da-pagina>` | `--publica` `--param <nome>` | Gera page + template + styles |
| `/new-nextjs-component <NomeComponente>` | `--template` | Gera componente React com styled-components |
| `/new-nextjs-service <dominio>` | `--api <api\|abv2\|match>` | Gera ou expande service layer |

## Agents (invocados pelo Claude)

| Agent | Repositório | Função |
|-------|-------------|--------|
| `adb-reviewer` | rep-adb-api | Revisa código C# contra padrões CQRS |
| `vue-reviewer` | rep-upik-web | Revisa código Vue.js contra padrões do projeto |
| `nextjs-reviewer` | ab-upik-nova-jornada | Revisa código Next.js/TypeScript contra padrões do projeto |

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
