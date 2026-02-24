---
name: adb-create-command
description: This skill should be used when the user wants to create a new command, add a new operation, create a new use case, or implement a new write action in the rep-adb-api repository. Activate when the user mentions "criar command", "novo command", "criar operação", "implementar command", "adicionar command", "registrar command", or describes a new action that modifies state (create, update, delete, cancel, confirm, send, approve, reject, register).
version: 1.0.0
---

# Criar um novo Command no rep-adb-api

## O que gerar

Para criar um Command completo são necessários:
1. **Arquivo do Command** — em `AdB.Application/Commands/{Entidade}/`
2. **Atualização do Handler** — no `{Entidade}CommandHandler.cs` existente (ou criar novo)
3. **Registro no DI** — em `AdB.API/Configurations/DependencyInjections/CommandsDIConfig.cs`

## Passo 1 — Arquivo do Command

**Caminho:** `src/AdB.Application/Commands/{Entidade}/{Verbo}{Entidade}Command.cs`

```csharp
using AdB.Core.Messages;
using AdB.Messages;
using FluentValidation;
using Newtonsoft.Json;
using NSwag.Annotations;

namespace AdB.Application.Commands.{Entidade};

public class {Verbo}{Entidade}Command : Command
{
    // Propriedades injetadas pelo Controller (não expostas na API)
    [JsonIgnore, SwaggerIgnore]
    public Guid {EntidadeRelacionada}Id { get; set; }

    // Propriedades do body da request
    public string Campo1 { get; set; }
    public Guid Campo2Id { get; set; }

    public override bool Validar()
    {
        ValidationResult = new {Verbo}{Entidade}CommandValidation().Validate(this);
        return ValidationResult.IsValid;
    }

    internal class {Verbo}{Entidade}CommandValidation : AbstractValidatorBase<{Verbo}{Entidade}Command>
    {
        public {Verbo}{Entidade}CommandValidation()
        {
            RuleFor(x => x.Campo1)
                .NotEmpty()
                .WithMessage(ValidationResource.Obrigatorio.FormatField(new string[] { TermsResource.Campo1 }))
                .MaximumLength(300)
                .WithMessage(ValidationResource.MaximoCaracteres.FormatField(new string[] { TermsResource.Campo1, "300" }));

            RuleFor(x => x.Campo2Id)
                .NotEmpty()
                .WithMessage(ValidationResource.Obrigatorio.FormatField(new string[] { TermsResource.Campo2 }));
        }
    }
}
```

### Regras para o Command
- Herda de `Command` (que já implementa `IRequest<CommandResult>`)
- **Sempre** sobrescrever `Validar()` chamando a nested class de validação
- Usar `[JsonIgnore, SwaggerIgnore]` em propriedades setadas pelo controller (ex: IDs extraídos do token/rota)
- Nested class de validação herda de `AbstractValidatorBase<T>` (não `AbstractValidator<T>`)
- Usar `ValidationResource` e `TermsResource` para mensagens — nunca strings literais

## Passo 2 — Handler

### Opção A: Adicionar a um Handler existente (preferido)

Se já existe um `{Entidade}CommandHandler.cs`, adicionar uma nova implementação de `IRequestHandler`:

```csharp
public class {Entidade}CommandHandler : CommandHandler,
    IRequestHandler<ComandoExistente1Command, CommandResult>,
    IRequestHandler<ComandoExistente2Command, CommandResult>,
    IRequestHandler<{Verbo}{Entidade}Command, CommandResult>  // ← Adicionar aqui
{
    // ... dependências existentes ...

    // Adicionar novo Handle method
    public async Task<CommandResult> Handle({Verbo}{Entidade}Command request, CancellationToken cancellationToken)
    {
        if (!request.Validar()) return request.ValidationResult;

        // 1. Buscar entidades necessárias
        var entidade = await _{entidade}Repository.ObterPorId(request.{Entidade}Id);
        if (entidade is null)
            return LancarErro(ValidationResource.NaoLocalizado, TermsResource.{Entidade});

        // 2. Regras de negócio / validações adicionais
        if (entidade.Cancelado)
            return LancarErro("Não é possível operar em uma entidade cancelada");

        // 3. Chamar método de domínio na entidade
        entidade.{Verbo}(request.Campo1, request.Campo2Id);

        // 4. Persistir
        _{entidade}Repository.Atualizar(entidade);
        var result = await PersistirDados(_{entidade}Repository.UnitOfWork);

        // 5. Side effects (notificações, logs, eventos) — só após persistência bem-sucedida
        if (result.IsValid)
        {
            await _mediator.Send(new CadastrarLogConsultoriaCommand(...));
            await _discordService.Notificar{Verbo}{Entidade}(...);
        }

        return result;
    }
}
```

### Opção B: Criar novo Handler (quando entidade não tem handler)

**Caminho:** `src/AdB.Application/Commands/{Entidade}/{Entidade}CommandHandler.cs`

```csharp
using AdB.Core.Messages;
using AdB.Domain.Interfaces;
using AdB.Messages;
using MediatR;

namespace AdB.Application.Commands.{Entidade};

public class {Entidade}CommandHandler : CommandHandler,
    IRequestHandler<{Verbo}{Entidade}Command, CommandResult>
{
    private readonly I{Entidade}Repository _{entidade}Repository;
    private readonly IMediator _mediator;

    public {Entidade}CommandHandler(
        I{Entidade}Repository {entidade}Repository,
        IMediator mediator)
    {
        _{entidade}Repository = {entidade}Repository;
        _mediator = mediator;
    }

    public async Task<CommandResult> Handle({Verbo}{Entidade}Command request, CancellationToken cancellationToken)
    {
        if (!request.Validar()) return request.ValidationResult;

        // Implementação...

        return await PersistirDados(_{entidade}Repository.UnitOfWork);
    }
}
```

### Regras para o Handler
- Herda de `CommandHandler` (nunca implementar `IRequestHandler` sem herdar `CommandHandler`)
- **Sempre** chamar `if (!request.Validar()) return request.ValidationResult;` como primeira linha
- Chamar `PersistirDados(repository.UnitOfWork)` para commitar — nunca chamar `Commit()` diretamente
- Side effects (Discord, RD Station, logs) sempre **depois** de `PersistirDados` e condicionais ao `result.IsValid`
- Encadear outros commands via `await _mediator.Send(new OutroCommand(...))`

## Passo 3 — Registro no DI

**Arquivo:** `src/AdB.API/Configurations/DependencyInjections/CommandsDIConfig.cs`

```csharp
// Adicionar dentro do método RegisterCommandServices()
services.AddScoped<IRequestHandler<{Verbo}{Entidade}Command, CommandResult>, {Entidade}CommandHandler>();
```

> **Nota:** Se o Handler já está registrado para outros commands da mesma entidade, apenas adicionar a nova linha para o novo command.

## Checklist completo

- [ ] Arquivo `{Verbo}{Entidade}Command.cs` criado em `AdB.Application/Commands/{Entidade}/`
- [ ] Command herda de `Command`
- [ ] Método `Validar()` implementado com nested class de validação
- [ ] Validação usa `AbstractValidatorBase<T>`, `ValidationResource` e `TermsResource`
- [ ] Handler implementa `IRequestHandler<{Verbo}{Entidade}Command, CommandResult>`
- [ ] Handler chama `request.Validar()` antes de qualquer lógica
- [ ] Handler usa `PersistirDados()` para persistência
- [ ] Registro adicionado em `CommandsDIConfig.cs`
