---
name: nextjs-create-service
description: This skill should be used when the user wants to create a new service file, add new API calls, or create a new API integration in the ab-upik-nova-jornada Next.js repository. Activate when the user mentions "criar service", "novo service next", "nova chamada de api", "integrar api", "criar endpoint", "service layer", "axios next", or describes adding API calls to the Next.js frontend.
version: 1.0.0
---

# Criar um novo Service no ab-upik-nova-jornada

## Onde criar

```
src/service/
└── {dominio}/
    └── index.ts        ← Interfaces + funções de serviço
```

Exemplos existentes:
- `src/service/clientes/index.ts` → cliente endpoints
- `src/service/consultorias/index.ts` → consultoria endpoints
- `src/service/jobs/index.ts` → job endpoints

---

## Instâncias de API disponíveis

**Arquivo:** `src/service/api.js`

```typescript
import api from 'service/api'            // API principal (NEXT_PUBLIC_API_BASE_URL)
import { apiAbv2 } from 'service/api'    // API adb v2 (NEXT_PUBLIC_API_BASE_URL_ABV2)
import { apiMatch } from 'service/api'   // API match (NEXT_PUBLIC_API_BASE_URL_MATCH)
```

**Regra:** usar `api` por padrão, `apiAbv2` para endpoints da API AdB v2, `apiMatch` para serviço de match.

---

## Estrutura do arquivo de serviço

```typescript
// src/service/{dominio}/index.ts

import { AxiosResponse } from 'axios'
import api, { apiAbv2 } from 'service/api'

// ─────────────────────────────────────────
// Interfaces de Request
// ─────────────────────────────────────────

export interface Criar{Entidade}Request {
  nome: string
  descricao?: string
  // ... campos do payload
}

export interface Atualizar{Entidade}Request {
  nome?: string
  descricao?: string
}

// ─────────────────────────────────────────
// Interfaces de Response
// ─────────────────────────────────────────

export interface {Entidade}Response {
  id: string
  nome: string
  descricao: string
  status: number
  statusDescricao: string
  criadoEm: string
  atualizadoEm?: string
}

export interface {Entidade}ItemResponse {
  id: string
  nome: string
  status: number
  statusDescricao: string
}

export interface Lista{Entidade}Response {
  total: number
  itens: {Entidade}ItemResponse[]
}

// ─────────────────────────────────────────
// Funções de Serviço
// ─────────────────────────────────────────

// GET — Listagem
export async function listar{Entidades}(
  filtro?: string
): Promise<AxiosResponse<{Entidade}ItemResponse[]>> {
  return api.get(`/{entidades}`, {
    params: filtro ? { termoPesquisa: filtro } : undefined,
  })
}

// GET — Detalhe por ID
export async function obter{Entidade}(
  id: string
): Promise<AxiosResponse<{Entidade}Response>> {
  return api.get(`/{entidades}/${id}`)
}

// POST — Criar
export async function criar{Entidade}(
  dados: Criar{Entidade}Request
): Promise<AxiosResponse<{Entidade}Response>> {
  return api.post(`/{entidades}`, dados)
}

// PUT — Atualizar
export async function atualizar{Entidade}(
  id: string,
  dados: Atualizar{Entidade}Request
): Promise<AxiosResponse<{Entidade}Response>> {
  return api.put(`/{entidades}/${id}`, dados)
}

// DELETE — Excluir
export async function excluir{Entidade}(
  id: string
): Promise<AxiosResponse<void>> {
  return api.delete(`/{entidades}/${id}`)
}

// POST — Ação específica (ex: cancelar, confirmar, aprovar)
export async function cancelar{Entidade}(
  id: string,
  motivo?: string
): Promise<AxiosResponse<void>> {
  return api.post(`/{entidades}/${id}/cancelar`, { motivo })
}
```

---

## Variações por caso de uso

### Serviço com query params

```typescript
export interface Filtro{Entidade} {
  termoPesquisa?: string
  status?: number
  pagina?: number
  itensPorPagina?: number
}

export async function listar{Entidades}(
  filtro: Filtro{Entidade} = {}
): Promise<AxiosResponse<Lista{Entidade}Response>> {
  return api.get(`/{entidades}`, { params: filtro })
}
```

### Serviço para upload de arquivo

```typescript
export async function uploadArquivo{Entidade}(
  id: string,
  arquivo: File
): Promise<AxiosResponse<{ url: string }>> {
  const formData = new FormData()
  formData.append('arquivo', arquivo)
  formData.append('entidadeId', id)

  return api.post(`/{entidades}/${id}/arquivo`, formData, {
    headers: { 'Content-Type': 'multipart/form-data' },
  })
}
```

### Serviço usando apiAbv2

```typescript
import { apiAbv2 } from 'service/api'

export async function obterDetalhes{Entidade}Abv2(
  consultoriaId: string
): Promise<AxiosResponse<Detalhes{Entidade}Abv2Response>> {
  return apiAbv2.get(`/consultorias/${consultoriaId}/detalhes`)
}
```

### Serviço com endpoint relativo ao usuário logado

```typescript
// Quando o endpoint depende do clienteId, o token já injeta automaticamente
// mas às vezes é necessário passar o id do usuário na URL:

export async function obterDadosCliente(
  clienteId: string
): Promise<AxiosResponse<DadosPessoais>> {
  return api.get(`/clientes/${clienteId}/dadosPessoais`)
}
```

---

## Como usar nos componentes/templates

```tsx
import { listar{Entidades}, criar{Entidade} } from 'service/{dominio}'
import type { {Entidade}ItemResponse, Criar{Entidade}Request } from 'service/{dominio}'

// Listar
const { data } = await listar{Entidades}()
setItems(data)

// Criar
const payload: Criar{Entidade}Request = { nome: 'teste' }
const { data: novo } = await criar{Entidade}(payload)

// Tratar erros
try {
  const { data } = await obter{Entidade}(id)
  setDados(data)
} catch (error) {
  showNotification('Erro', 'Não foi possível carregar os dados.', 'error')
}
```

---

## Tipos comuns para interfaces

```typescript
// Paginação
export interface PagedResponse<T> {
  total: number
  pagina: number
  itensPorPagina: number
  itens: T[]
}

// Resposta genérica de sucesso
export interface CommandResponse {
  sucesso: boolean
  mensagem?: string
  errors?: string[]
}

// Datas como string ISO 8601
criadoEm: string           // "2024-01-15T10:30:00Z"
dataAgendamento?: string   // opcional — usar `string | null` ou `string | undefined`

// Enums numerados (espelhar o backend)
status: number             // 1 = Ativo, 2 = Inativo, 3 = Cancelado
// + statusDescricao: string  ← sempre ter o campo de descrição junto
```

---

## Regras para Services

- **Sempre tipado:** todas as funções com `Promise<AxiosResponse<T>>`
- **Interfaces de request e response** definidas no mesmo arquivo
- **Sem try/catch no service** — o componente/template que chama é responsável pelo tratamento de erro
- **Nomes de função:** verbo + entidade em camelCase (`listarConsultorias`, `obterCliente`, `criarAgendamento`)
- **Uma responsabilidade por arquivo** — um arquivo por domínio
- **Não acessar localStorage** no service — autenticação é gerenciada pelo interceptor em `api.js`
- **Parâmetros opcionais** com `?:` e defaults nos parâmetros da função

## Checklist

- [ ] Pasta `src/service/{dominio}/` criada
- [ ] `index.ts` com interfaces de request, response e funções de serviço
- [ ] Todas as funções com tipo de retorno `Promise<AxiosResponse<T>>`
- [ ] Interfaces exportadas para uso nos componentes
- [ ] Instância de API correta usada (`api`, `apiAbv2` ou `apiMatch`)
- [ ] Parâmetros de query usando o campo `params` do axios
- [ ] Sem tratamento de erro no service — deixar para quem chama
