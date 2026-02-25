---
description: Cria ou adiciona funções a um service no ab-upik-nova-jornada (Next.js)
argument-hint: <dominio> [--api <api|abv2|match>]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Novo Service — ab-upik-nova-jornada

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar ou expandir um arquivo de serviço no repositório ab-upik-nova-jornada seguindo o padrão de service layer com axios e TypeScript.

### Passo 1 — Entender o contexto

1. Extraia do argumento o domínio do service (ex: `clientes`, `consultorias`, `produtos`)
2. Verifique se `--api` foi passado para saber qual instância usar (`api`, `apiAbv2`, `apiMatch`)
3. Verifique se já existe `src/service/{dominio}/index.ts` — se sim, leia o arquivo para adicionar sem duplicar
4. Leia `src/service/api.js` para confirmar as instâncias disponíveis
5. Leia service files similares (ex: `src/service/clientes/index.ts`) para entender o padrão

### Passo 2 — Levantamento de requisitos

Se o argumento não deixou claro, pergunte ao usuário:
- Quais operações o service precisa ter? (listar, obter por ID, criar, atualizar, excluir, ação específica)
- Qual é a base da URL do endpoint? (ex: `/clientes/{id}/dados-pessoais`)
- Quais campos fazem parte do request? (payload do POST/PUT)
- Quais campos fazem parte do response? (dados retornados pela API)
- Há filtros/parâmetros de query para listagens?
- Qual instância de API usar? (`api` padrão, `apiAbv2`, `apiMatch`)

### Passo 3 — Gerar o arquivo

**Arquivo:** `src/service/{dominio}/index.ts`

- Se arquivo já existe: **adicionar** as novas interfaces e funções sem remover as existentes
- Se não existe: criar do zero com a estrutura completa

Estrutura obrigatória:
- Interfaces de request com sufixo `Request` (ex: `Criar{Entidade}Request`)
- Interfaces de response com sufixo `Response` (ex: `{Entidade}Response`, `{Entidade}ItemResponse`)
- Funções assíncronas com retorno `Promise<AxiosResponse<T>>`
- Sem try/catch — o erro propaga para quem chama
- Parâmetros de query via `{ params: { ... } }` no axios

### Passo 4 — Resumo

Após criar/editar o arquivo, exibir:
- Caminho do arquivo criado/editado
- Lista de funções criadas com suas assinaturas TypeScript
- Exemplo de uso nos componentes/templates
