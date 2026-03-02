---
name: match-create-command
description: This skill should be used when the user wants to create a new command, add a new write operation, or implement a new use case in the api-match repository. Activate when the user mentions "criar command match", "novo command match", "salvar", "criar operação match", or describes a state-changing action (create, save, update, delete, generate, send) in the context of the api-match/Match.Core repository.
version: 1.0.0
---

# Criar um novo Command no api-match

## Convenção fundamental: Handler aninhado no mesmo arquivo

Toda operação de escrita segue a convenção de **arquivo único** com o Handler como **classe aninhada**:

```
Match.Core/{Dominio}/Commands/{Acao}{Entidade}Command.cs
├── class {Acao}{Entidade}Command              → parâmetros de entrada (IRequest<TResponse>)
└── class {Acao}{Entidade}CommandHandler       → implementação (nested public class)
```

> **Não registrar no DI** — MediatR descobre automaticamente via Assembly scan.

---

## Estrutura do arquivo

**Caminho:** `Match.Core/{Dominio}/Commands/{Acao}{Entidade}Command.cs`

```csharp
namespace Match.Core.{Dominio}.Commands
{
    // 1. Command — parâmetros de entrada
    public class {Acao}{Entidade}Command : IRequest<{TipoRetorno}>
    {
        public {TipoPropriedade} {Propriedade} { get; set; }

        public {Acao}{Entidade}Command({TipoPropriedade} {propriedade})
        {
            {Propriedade} = {propriedade};
        }

        // 2. Handler — classe aninhada pública
        public class {Acao}{Entidade}CommandHandler : IRequestHandler<{Acao}{Entidade}Command, {TipoRetorno}>
        {
            private readonly IRepository<{Entidade}> _{entidade}s;
            private readonly IMediator _mediator;

            public {Acao}{Entidade}CommandHandler(
                IRepository<{Entidade}> {entidade}s,
                IMediator mediator)
            {
                _{entidade}s = {entidade}s;
                _mediator = mediator;
            }

            public async Task<{TipoRetorno}> Handle(
                {Acao}{Entidade}Command request,
                CancellationToken cancellationToken)
            {
                // 1. Verificar se já existe (quando aplicável)
                var existente = await _{entidade}s.FirstOrDefaultAsync(x =>
                    x.Where(e => e.Id == request.Id));

                if (existente is not null)
                    request.{Entidade}.Id = existente.Id;

                // 2. Persistir via upsert
                await _{entidade}s.SaveOrUpdateAsync(
                    request.{Entidade},
                    x => x.Id == request.{Entidade}.Id);

                // 3. Side effects (quando necessário)
                // await _mediator.Send(new LogarAcaoCommand(...));

                return request.{Entidade};
            }
        }
    }
}
```

---

## Tipos de retorno comuns

| Cenário | Tipo de retorno | Exemplo |
|---------|----------------|---------|
| Retornar a entidade salva | `{Entidade}` | `IRequest<Client>` |
| Retornar string (token, ID) | `string` | `IRequest<string>` |
| Retornar DTO | `{Nome}DTO` | `IRequest<PerfilClienteDTO>` |
| Apenas confirmar execução | `bool` | `IRequest<bool>` |
| Sem retorno | `Unit` | `IRequest<Unit>` |

---

## Exemplo real — CreateClientCommand

```csharp
namespace Match.Core.Clients.Commands
{
    public class CreateClientCommand : IRequest<Client>
    {
        public Client Cliente { get; set; }

        public CreateClientCommand(Client cliente)
        {
            Cliente = cliente;
        }

        public class CreateClientCommandHandler : IRequestHandler<CreateClientCommand, Client>
        {
            private readonly IMediator _mediator;
            private readonly IRepository<Client> _clients;

            public CreateClientCommandHandler(IMediator mediator, IRepository<Client> clients)
            {
                _mediator = mediator;
                _clients = clients;
            }

            public async Task<Client> Handle(CreateClientCommand request, CancellationToken cancellationToken)
            {
                // Verifica se cliente já existe pelo email
                var client = await _mediator.Send(new GetClientByEmailQuery(request.Cliente.Email));

                if (client is not null)
                {
                    // Preserva ID e data de criação do registro existente
                    request.Cliente.Id = client.Id;
                    request.Cliente.Date = client.Date;
                }

                await _clients.SaveOrUpdateAsync(
                    request.Cliente,
                    x => x.Id == request.Cliente.Id && x.Email == request.Cliente.Email);

                return request.Cliente;
            }
        }
    }
}
```

---

## Encadeamento com outros Commands/Queries

O handler pode acionar outros Commands/Queries via `IMediator`:

```csharp
// Consultar antes de salvar
var existente = await _mediator.Send(new GetClientQuery(request.Id));

// Logar erro após falha
await _mediator.Send(new LogarErroCommand(ex.Message, "contexto"));

// Enviar notificação após salvar
await _mediator.Send(new EnviarEmailCommand(request.Email, "assunto", "corpo"));
```

---

## Operações em lote

```csharp
// Salvar múltiplos objetos de uma vez
await _repository.SaveBatchAsync(listaDeObjetos);
```

---

## Operações com Azure Blob (upload de arquivo)

```csharp
// Disponível via IOptions<ConfiguracaoBlob>
var url = await AzureBlob.UploadAsync(
    _blobConfig.Value.ConnectionString,
    _blobConfig.Value.ContainerMatchUploads,
    nomeArquivo,
    streamDoArquivo);
```

---

## Padrão de validação manual

O projeto **não usa FluentValidation**. Validar com guard clauses simples:

```csharp
public async Task<Client> Handle(CreateClientCommand request, CancellationToken cancellationToken)
{
    if (request.Cliente is null)
        throw new ArgumentNullException(nameof(request.Cliente));

    if (string.IsNullOrEmpty(request.Cliente.Email))
        throw new ArgumentException("Email é obrigatório.");

    // ... resto da lógica
}
```

---

## Checklist

- [ ] Arquivo único em `Match.Core/{Dominio}/Commands/{Acao}{Entidade}Command.cs`
- [ ] Command implementa `IRequest<TResponse>` diretamente (sem classe base)
- [ ] Handler é **classe pública aninhada** dentro do Command
- [ ] Handler implementa `IRequestHandler<TCommand, TResponse>`
- [ ] IDs são `string` (não `Guid`)
- [ ] Persistência via `SaveOrUpdateAsync()` com filtro upsert
- [ ] Side effects via `_mediator.Send()` encadeado
- [ ] **Não registrar no DI** — MediatR descobre automaticamente
