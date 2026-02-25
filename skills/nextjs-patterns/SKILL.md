---
name: nextjs-patterns
description: This skill should be used when the user asks about how the ab-upik-nova-jornada Next.js repository works, asks about patterns, architecture, conventions, component structure, hooks, contexts, services, styled-components, TypeScript types, or asks "how do we do X in this project". Also activate when the user mentions Next.js, React, TypeScript, styled-components, hooks, contexts, useAuth, service layer, Ant Design, react-hook-form, yup, withAuth, or describes working on the ab-upik-nova-jornada frontend.
version: 1.0.0
---

# Padrões do Repositório ab-upik-nova-jornada (Next.js)

## Arquitetura Geral

Aplicação em **Next.js** com **React** funcional, **TypeScript** e **Styled-Components**.

```
src/
├── pages/              → Roteamento file-based (Next.js)
├── components/         → Componentes React
│   └── {NomeComponente}/
│       ├── index.tsx   → Lógica do componente
│       └── styles.ts   → Styled-components
├── templates/          → Layouts/templates de página
│   └── {NomeTemplate}/
│       ├── index.tsx
│       └── styles.ts
├── contexts/           → Context API
│   └── {NomeContext}/
│       ├── context.tsx  → Definição do contexto e tipos
│       └── provider.tsx → Provider com implementação
├── hooks/              → Custom hooks
├── service/            → Chamadas de API
│   ├── api.js          → Instâncias axios (api, apiAbv2, apiMatch)
│   ├── clientes/       → Serviços por domínio
│   ├── consultorias/
│   └── jobs/
├── styles/
│   ├── theme.ts        → Tema (cores, espaçamentos, fontes)
│   └── global.ts       → Estilos globais
├── types/              → TypeScript types globais
└── utils/
    ├── notification.ts  → showNotification, Alert (Swal)
    ├── analytics.ts    → Tracking GTM/dataLayer
    └── helpers.ts      → Funções utilitárias
```

## Estrutura de Componente

**Padrão:** componente funcional TypeScript com styled-components em arquivo separado.

```tsx
// components/MeuComponente/index.tsx

import * as S from './styles'

// 1. Tipos das props exportados
export type MeuComponenteProps = {
  titulo: string
  descricao?: string          // prop opcional
  onClick: () => void
  items?: ItemType[]
}

// 2. Componente funcional
const MeuComponente = ({
  titulo,
  descricao,
  onClick,
  items = []                  // default para opcionais
}: MeuComponenteProps) => {
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    // efeito ao montar
  }, [])                      // deps array sempre declarado

  return (
    <S.Wrapper>
      <S.Title>{titulo}</S.Title>
      {descricao && <S.Description>{descricao}</S.Description>}
      <S.Button onClick={onClick}>Ação</S.Button>
    </S.Wrapper>
  )
}

export default MeuComponente
```

```typescript
// components/MeuComponente/styles.ts

import styled, { css } from 'styled-components'
import media from 'styled-media-query'

export const Wrapper = styled.div`
  ${({ theme }) => css`
    width: 100%;
    padding: ${theme.spacings.small};
    background-color: ${theme.colors.white};
    border-radius: ${theme.border.radius};
  `}
`

export const Title = styled.h2`
  ${({ theme }) => css`
    color: ${theme.colors.black};
    font-size: ${theme.font.sizes.large};
    font-weight: ${theme.font.bold};

    ${media.lessThan('medium')`
      font-size: ${theme.font.sizes.medium};
    `}
  `}
`

export const Button = styled.button`
  ${({ theme }) => css`
    background-color: ${theme.colors.primary};
    color: ${theme.colors.white};
    padding: ${theme.spacings.xxsmall} ${theme.spacings.small};
    border-radius: ${theme.border.radius};
    border: none;
    cursor: pointer;

    &:hover {
      background-color: ${theme.colors.primaryHover};
    }
  `}
`
```

## Tema (styled-components)

Valores disponíveis em `styles/theme.ts` via `${({ theme }) => ...}`:

```typescript
theme.colors.primary          // '#7F0BA4'
theme.colors.primaryHover     // '#6e0b93'
theme.colors.secondary        // '#F9CC3B'
theme.colors.white            // '#FFFFFF'
theme.colors.black            // '#030517'

theme.font.sizes.xsmall       // '1.2rem'
theme.font.sizes.small        // '1.4rem'
theme.font.sizes.medium       // '1.6rem'
theme.font.sizes.large        // '1.8rem'
theme.font.sizes.xlarge       // '2.0rem'
theme.font.sizes.xxlarge      // '2.8rem'
theme.font.light              // 300
theme.font.normal             // 400
theme.font.bold               // 700

theme.spacings.xxsmall        // '0.8rem'
theme.spacings.xsmall         // '1.6rem'
theme.spacings.small          // '2.4rem'
theme.spacings.medium         // '3.2rem'
theme.spacings.large          // '4.0rem'
theme.spacings.xlarge         // '4.8rem'

theme.border.radius           // '0.7rem'
theme.border.radiusModal      // '1.0rem'

theme.layers.base             // 10
theme.layers.menu             // 20
theme.layers.overlay          // 30
theme.layers.modal            // 40
theme.layers.alwaysOnTop      // 50

theme.transition.default      // '0.3s ease-in-out'
theme.transition.fast         // '0.1s ease-in-out'
```

## Service Layer (API)

**Arquivo base:** `src/service/api.js`

```javascript
// Instâncias disponíveis:
import api from 'service/api'           // API principal nova jornada
import { apiAbv2 } from 'service/api'   // API adb v2
import { apiMatch } from 'service/api'  // API de match
```

**Padrão de service file:**

```typescript
// service/minhaEntidade/index.ts
import { AxiosResponse } from 'axios'
import api, { apiAbv2 } from 'service/api'

// 1. Interfaces de request/response
export interface MinhaEntidadeRequest {
  campo1: string
  campo2?: number
}

export interface MinhaEntidadeResponse {
  id: string
  campo1: string
  campo2: number
  criadoEm: string
}

// 2. Funções de serviço (async, retornam AxiosResponse tipado)
export async function listarMinhaEntidade(
  clienteId: string
): Promise<AxiosResponse<MinhaEntidadeResponse[]>> {
  return api.get(`/minha-entidade?clienteId=${clienteId}`)
}

export async function obterMinhaEntidade(
  id: string
): Promise<AxiosResponse<MinhaEntidadeResponse>> {
  return api.get(`/minha-entidade/${id}`)
}

export async function criarMinhaEntidade(
  dados: MinhaEntidadeRequest
): Promise<AxiosResponse<MinhaEntidadeResponse>> {
  return api.post(`/minha-entidade`, dados)
}

export async function atualizarMinhaEntidade(
  id: string,
  dados: Partial<MinhaEntidadeRequest>
): Promise<AxiosResponse<MinhaEntidadeResponse>> {
  return api.put(`/minha-entidade/${id}`, dados)
}
```

**Token:** injetado automaticamente via interceptor em `api.js` — não gerenciar manualmente.

## Contextos (Context API)

**Padrão: dois arquivos por contexto**

```typescript
// contexts/MinhaFeature/context.tsx

import { createContext, useContext } from 'react'

// Tipos do contexto
export interface MinhaFeatureContextData {
  dados: DadosTipo | null
  loading: boolean
  atualizar: (novosDados: DadosTipo) => void
}

// Criação do contexto
const MinhaFeatureContext = createContext<MinhaFeatureContextData>(
  {} as MinhaFeatureContextData
)

export default MinhaFeatureContext
```

```typescript
// contexts/MinhaFeature/provider.tsx

import { useState, useCallback } from 'react'
import MinhaFeatureContext from './context'
import type { MinhaFeatureContextData } from './context'

interface Props {
  children: React.ReactNode
}

export const MinhaFeatureProvider = ({ children }: Props) => {
  const [dados, setDados] = useState<DadosTipo | null>(null)
  const [loading, setLoading] = useState(false)

  const atualizar = useCallback((novosDados: DadosTipo) => {
    setDados(novosDados)
  }, [])

  return (
    <MinhaFeatureContext.Provider value={{ dados, loading, atualizar }}>
      {children}
    </MinhaFeatureContext.Provider>
  )
}
```

## Custom Hooks

```typescript
// hooks/useMeuHook.tsx

import { useContext } from 'react'
import MinhaFeatureContext from '../contexts/MinhaFeature/context'

const useMeuHook = () => {
  const ctx = useContext(MinhaFeatureContext)
  return ctx
}

export default useMeuHook
```

**Hooks disponíveis:**
```typescript
import useAuth from 'hooks/useAuth'
// useAuth() → { logado, usuario, signIn, signOut, loading, signInGoogle }

import useLocalStorage from 'hooks/useLocalStorage'
// const [valor, setValor, removeValor] = useLocalStorage('@chave: nome')
```

## Páginas (Next.js)

```tsx
// pages/minha-pagina.tsx

import { GetServerSideProps } from 'next'
import MinhaPaginaTemplate from 'templates/MinhaPagina'
import withAuth from 'components/Auth/WithAuth'

// Página protegida (requer autenticação)
function MinhaPagina() {
  return <MinhaPaginaTemplate />
}

export default withAuth(MinhaPagina)

// Página pública (sem auth)
function PaginaPublica() {
  return <PaginaPublicaTemplate />
}

export default PaginaPublica
```

## Notificações

```typescript
import { showNotification } from 'utils/notification'

// Toast (não bloqueia)
showNotification('', 'Operação realizada com sucesso', 'success', true)
showNotification('', 'Erro ao processar', 'error', true)

// Modal (bloqueia — padrão)
showNotification('Atenção', 'Você tem certeza?', 'warning')
showNotification('Sucesso!', 'Dados salvos com sucesso.', 'success')

// Swal direto (para confirmações)
import { Alert } from 'utils/notification'
const result = await Alert.fire({
  title: 'Confirmar?',
  text: 'Esta ação não pode ser desfeita.',
  icon: 'warning',
  showCancelButton: true,
  confirmButtonText: 'Sim, confirmar',
  cancelButtonText: 'Cancelar',
})
if (result.isConfirmed) { /* ação */ }
```

## Formulários (react-hook-form + yup)

```tsx
import { useForm, Controller } from 'react-hook-form'
import { yupResolver } from '@hookform/resolvers/yup'
import * as yup from 'yup'

interface IFormInputs {
  nome: string
  email: string
  valor?: number
}

const schema = yup.object({
  nome: yup.string().required('Nome é obrigatório').label('nome'),
  email: yup.string().email('Email inválido').required().label('email'),
  valor: yup.number().optional().positive().label('valor'),
}).required()

const MeuForm = () => {
  const { handleSubmit, control, formState: { errors } } = useForm<IFormInputs>({
    resolver: yupResolver(schema),
    defaultValues: { nome: '', email: '' }
  })

  const onSubmit = async (data: IFormInputs) => {
    // processar data
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Controller
        name="nome"
        control={control}
        render={({ field }) => (
          <input {...field} placeholder="Nome" />
        )}
      />
      {errors.nome && <span>{errors.nome.message}</span>}
      <button type="submit">Salvar</button>
    </form>
  )
}
```

## TypeScript — Padrões de Tipos

```typescript
// Tipos de domínio → em service/{dominio}/index.ts
export interface EntidadeResponse { ... }
export interface EntidadeRequest { ... }

// Props de componentes → no próprio component/index.tsx
export type MeuComponenteProps = { ... }

// Tipos globais/Window → em types/index.d.ts
declare global {
  interface Window {
    dataLayer?: any[]
  }
}
```

## Analytics / GTM

```typescript
// Evento dataLayer
window.dataLayer = window.dataLayer || []
window.dataLayer.push({
  event: 'nome_do_evento',
  item_id: id,
  item_name: nome,
})
```

## Conventions de Nomenclatura

| Tipo | Padrão | Exemplos |
|------|--------|----------|
| Páginas | kebab-case.tsx | `dados-pessoais.tsx`, `minhas-consultorias.tsx` |
| Componentes | PascalCase/ (pasta) | `CardProduto/`, `FormLogin/` |
| Templates | PascalCase/ (pasta) | `Dashboard/`, `MinhasConsultorias/` |
| Contextos | PascalCase/ (pasta) | `Auth/`, `Consultoria3d/` |
| Hooks | camelCase.tsx | `useAuth.tsx`, `useLocalStorage.tsx` |
| Services | snake-case ou camelCase | `clientes/index.ts`, `consultorias/index.ts` |
| Types de props | `{Componente}Props` | `CardProdutoProps`, `FormLoginProps` |
| Funções | camelCase, verbo primeiro | `obterDados()`, `salvarAlteracao()` |
| Interfaces | PascalCase | `DadosPessoais`, `LoginResponse` |

## Responsive (styled-media-query)

```typescript
import media from 'styled-media-query'

// Breakpoints disponíveis: 'small' (480px), 'medium' (768px), 'large' (1170px)
export const Wrapper = styled.div`
  display: flex;

  ${media.lessThan('medium')`
    flex-direction: column;
  `}

  ${media.greaterThan('large')`
    max-width: 1200px;
  `}
`
```
