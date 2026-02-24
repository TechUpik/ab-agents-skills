---
description: Cria uma feature completa no rep-adb-api (command + query + controller + DI)
argument-hint: <NomeDaFeature>
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Nova Feature Completa — rep-adb-api

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar uma feature end-to-end no repositório rep-adb-api: Commands, Queries e Controller, seguindo todos os padrões CQRS do projeto.

### Passo 1 — Descoberta e contexto

1. Leia a estrutura do projeto para entender o que já existe:
   - `src/AdB.Application/Commands/` — commands existentes
   - `src/AdB.Application/Queries/` — queries existentes
   - `src/AdB.API/Controllers/` — controllers existentes
2. Se a entidade já existe, leia os arquivos existentes para não duplicar
3. Leia `src/AdB.API/Configurations/DependencyInjections/CommandsDIConfig.cs`
4. Leia `src/AdB.API/Configurations/DependencyInjections/QueriesDIConfig.cs`

### Passo 2 — Levantamento de requisitos

**OBRIGATÓRIO:** Antes de criar qualquer arquivo, apresente ao usuário as perguntas:

1. **Entidade e operações:** Qual entidade esta feature envolve? Quais operações são necessárias? (CRUD completo, ou apenas algumas?)
2. **Commands necessários:** Quais ações de escrita? (criar, atualizar, cancelar, aprovar, enviar, excluir?)
3. **Queries necessárias:** Quais dados precisam ser lidos? (listagem com filtros, detalhe por ID, dashboard?)
4. **Campos:** Quais propriedades o command recebe? Quais campos aparecem na response da query?
5. **Validações:** Quais campos são obrigatórios? Há validações de formato (email, CPF, tamanho máximo)?
6. **Regras de negócio:** Há alguma regra especial? (não pode cancelar se X, deve notificar quando Y)
7. **Autenticação:** O controller é público ou requer `[Authorize]`? Há segmentação por perfil?
8. **Rota:** Qual o segmento de rota? (ex: `consultorias`, `upiker-config`)

**Aguardar respostas antes de prosseguir.**

### Passo 3 — Plano de implementação

Após colher as respostas, apresentar o plano completo:

```
Arquivos a criar:
├── src/AdB.Application/Commands/{Entidade}/
│   ├── Criar{Entidade}Command.cs
│   ├── Atualizar{Entidade}Command.cs     (se necessário)
│   ├── Cancelar{Entidade}Command.cs      (se necessário)
│   └── {Entidade}CommandHandler.cs
├── src/AdB.Application/Queries/{Modulo}/{Entidade}/
│   ├── Listar{Entidade}Query.cs          (query + response + handler)
│   └── Obter{Entidade}Query.cs           (query + result + handler)
└── src/AdB.API/Controllers/{Entidade}Controller.cs

Arquivos a editar:
├── src/AdB.API/Configurations/DependencyInjections/CommandsDIConfig.cs
└── src/AdB.API/Configurations/DependencyInjections/QueriesDIConfig.cs (se Dapper)
```

**Pedir aprovação do plano antes de implementar.**

### Passo 4 — Implementação

Na ordem:
1. **Commands** (mais críticos, definem o contrato de escrita)
2. **Queries** (definem o contrato de leitura)
3. **Controller** (orquestra commands e queries via IMediator)
4. **DI Registration** (registrar commands no `CommandsDIConfig.cs`)

Seguir todos os padrões documentados nas skills `adb-create-command`, `adb-create-query` e `adb-create-controller`.

### Passo 5 — Resumo final

Exibir tabela com:
| Arquivo | Tipo | Status |
|---------|------|--------|
| `Commands/Entidade/CriarEntidadeCommand.cs` | Command | ✅ Criado |
| `Commands/Entidade/EntidadeCommandHandler.cs` | Handler | ✅ Criado |
| `Queries/Modulo/Entidade/ListarEntidadeQuery.cs` | Query | ✅ Criado |
| `Controllers/EntidadeController.cs` | Controller | ✅ Criado |
| `CommandsDIConfig.cs` | DI | ✅ Atualizado |

Listar os endpoints criados com os paths completos:
```
GET  /api/v1/{recurso}/listar
GET  /api/v1/{recurso}/{id}
POST /api/v1/{recurso}/criar
PUT  /api/v1/{recurso}/{id}
```
