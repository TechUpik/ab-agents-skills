---
description: Cria um novo Command completo no rep-adb-api (classe + handler + DI)
argument-hint: <VerbEntidade> [--handler <NomeHandler>]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Novo Command — rep-adb-api

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar um Command completo no repositório rep-adb-api seguindo os padrões CQRS do projeto.

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome do Command (ex: `CancelarConsultoria` → Command `CancelarConsultoriaCommand`, entidade `Consultoria`)
2. Leia o diretório `src/AdB.Application/Commands/` para ver quais entidades já existem
3. Se existe um handler para a entidade (`{Entidade}CommandHandler.cs`), leia-o para entender as dependências já injetadas
4. Leia `src/AdB.API/Configurations/DependencyInjections/CommandsDIConfig.cs` para entender o padrão de registro

### Passo 2 — Perguntar ao usuário (se necessário)

Se o argumento não deixou claro, pergunte:
- Qual entidade este command afeta?
- Quais propriedades o command precisa receber no body da request?
- Quais validações são necessárias (campos obrigatórios, tamanho máximo, etc.)?
- Há alguma regra de negócio específica no handler?

### Passo 3 — Gerar os arquivos

**Arquivo 1:** `src/AdB.Application/Commands/{Entidade}/{Verbo}{Entidade}Command.cs`
- Herdar de `Command`
- Implementar `Validar()` com nested class `AbstractValidatorBase<T>`
- Usar `ValidationResource` e `TermsResource` para mensagens
- Usar `[JsonIgnore, SwaggerIgnore]` em props injetadas pelo controller

**Arquivo/Edição 2:** Handler em `src/AdB.Application/Commands/{Entidade}/{Entidade}CommandHandler.cs`
- Se o handler existe: adicionar novo `IRequestHandler<>` à lista e novo método `Handle()`
- Se não existe: criar o handler completo herdando de `CommandHandler`
- Sempre chamar `request.Validar()` como primeira linha
- Usar `PersistirDados(repository.UnitOfWork)` para persistir

**Edição 3:** `src/AdB.API/Configurations/DependencyInjections/CommandsDIConfig.cs`
- Adicionar: `services.AddScoped<IRequestHandler<{Verbo}{Entidade}Command, CommandResult>, {Entidade}CommandHandler>();`

### Passo 4 — Resumo

Após criar os arquivos, exibir:
- Arquivos criados/editados com seus caminhos
- Template do endpoint que o desenvolvedor precisará criar no controller
- Lembretes de qualquer coisa que o desenvolvedor precisa ajustar manualmente
