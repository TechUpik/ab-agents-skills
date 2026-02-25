---
name: nextjs-create-component
description: This skill should be used when the user wants to create a new React component in the ab-upik-nova-jornada Next.js repository. Activate when the user mentions "criar componente", "novo componente react", "criar componente next", "criar card", "criar componente typescript", "criar componente styled", or describes building a new reusable UI component for the Next.js frontend.
version: 1.0.0
---

# Criar um novo Componente no ab-upik-nova-jornada

## Onde criar

```
src/components/{NomeComponente}/
├── index.tsx       ← Lógica do componente
└── styles.ts       ← Styled-components
```

Componentes de layout/página vão em `templates/` com a mesma estrutura.

---

## Passo 1 — index.tsx

```tsx
// src/components/{NomeComponente}/index.tsx

import { useState, useEffect } from 'react'
import * as S from './styles'

// 1. Tipos das props (exportar para reutilização)
export type {NomeComponente}Props = {
  // Props obrigatórias
  titulo: string
  // Props opcionais
  descricao?: string
  onClick?: () => void
  children?: React.ReactNode
}

// 2. Componente funcional
const {NomeComponente} = ({
  titulo,
  descricao,
  onClick,
  children,
}: {NomeComponente}Props) => {
  // Estado local se necessário
  const [ativo, setAtivo] = useState(false)

  return (
    <S.Wrapper>
      <S.Title>{titulo}</S.Title>
      {descricao && <S.Description>{descricao}</S.Description>}
      {children}
      {onClick && (
        <S.Button onClick={onClick}>Ação</S.Button>
      )}
    </S.Wrapper>
  )
}

export default {NomeComponente}
```

---

## Passo 2 — styles.ts

```typescript
// src/components/{NomeComponente}/styles.ts

import styled, { css } from 'styled-components'
import media from 'styled-media-query'

export const Wrapper = styled.div`
  ${({ theme }) => css`
    width: 100%;
    padding: ${theme.spacings.small};
    background-color: ${theme.colors.white};
    border-radius: ${theme.border.radius};
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  `}
`

export const Title = styled.h3`
  ${({ theme }) => css`
    color: ${theme.colors.black};
    font-size: ${theme.font.sizes.large};
    font-weight: ${theme.font.bold};
    margin-bottom: ${theme.spacings.xxsmall};
  `}
`

export const Description = styled.p`
  ${({ theme }) => css`
    color: ${theme.colors.black};
    font-size: ${theme.font.sizes.medium};
    opacity: 0.7;
    margin-bottom: ${theme.spacings.xsmall};
  `}
`

export const Button = styled.button`
  ${({ theme }) => css`
    background-color: ${theme.colors.primary};
    color: ${theme.colors.white};
    padding: ${theme.spacings.xxsmall} ${theme.spacings.small};
    border-radius: ${theme.border.radius};
    border: none;
    font-size: ${theme.font.sizes.small};
    font-weight: ${theme.font.bold};
    cursor: pointer;
    transition: ${theme.transition.default};

    &:hover {
      background-color: ${theme.colors.primaryHover};
    }
  `}
`
```

---

## Exemplos por tipo de componente

### Card de dados

```tsx
// src/components/Card{Entidade}/index.tsx
import * as S from './styles'

export type Card{Entidade}Props = {
  id: string
  nome: string
  status: string
  statusDescricao: string
  data?: string
  onSelecionar?: (id: string) => void
}

const Card{Entidade} = ({
  id,
  nome,
  status,
  statusDescricao,
  data,
  onSelecionar,
}: Card{Entidade}Props) => {
  return (
    <S.Wrapper>
      <S.Header>
        <S.Nome>{nome}</S.Nome>
        <S.Badge status={status}>{statusDescricao}</S.Badge>
      </S.Header>
      {data && <S.Data>{data}</S.Data>}
      {onSelecionar && (
        <S.ButtonContainer>
          <S.Button onClick={() => onSelecionar(id)}>Ver detalhe</S.Button>
        </S.ButtonContainer>
      )}
    </S.Wrapper>
  )
}

export default Card{Entidade}
```

```typescript
// src/components/Card{Entidade}/styles.ts
import styled, { css } from 'styled-components'

type BadgeProps = { status: string }

export const Wrapper = styled.div`
  ${({ theme }) => css`
    padding: ${theme.spacings.small};
    border: 1px solid #eee;
    border-radius: ${theme.border.radius};
    margin-bottom: ${theme.spacings.xsmall};
  `}
`

export const Header = styled.div`
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 8px;
`

export const Nome = styled.h4`
  ${({ theme }) => css`
    font-size: ${theme.font.sizes.medium};
    font-weight: ${theme.font.bold};
    color: ${theme.colors.black};
  `}
`

export const Badge = styled.span<BadgeProps>`
  ${({ theme, status }) => css`
    padding: 4px 8px;
    border-radius: 12px;
    font-size: ${theme.font.sizes.xsmall};
    background-color: ${
      status === 'ativo' ? '#e6f4ea' :
      status === 'pendente' ? '#fff3e0' :
      '#f5f5f5'
    };
    color: ${
      status === 'ativo' ? '#2e7d32' :
      status === 'pendente' ? '#e65100' :
      '#666'
    };
  `}
`

export const Data = styled.p`
  ${({ theme }) => css`
    font-size: ${theme.font.sizes.small};
    color: ${theme.colors.black};
    opacity: 0.6;
  `}
`

export const ButtonContainer = styled.div`
  margin-top: 12px;
`

export const Button = styled.button`
  ${({ theme }) => css`
    background-color: transparent;
    color: ${theme.colors.primary};
    border: 1px solid ${theme.colors.primary};
    padding: 4px 12px;
    border-radius: ${theme.border.radius};
    font-size: ${theme.font.sizes.small};
    cursor: pointer;
    transition: ${theme.transition.fast};

    &:hover {
      background-color: ${theme.colors.primary};
      color: ${theme.colors.white};
    }
  `}
`
```

### Componente com estado assíncrono

```tsx
import { useState, useEffect } from 'react'
import * as S from './styles'
import { showNotification } from 'utils/notification'
import { listarItens } from 'service/meuDominio'
import type { ItemResponse } from 'service/meuDominio'

export type ListaProps = {
  filtro?: string
  onItemSelecionado?: (id: string) => void
}

const Lista{Entidade} = ({ filtro, onItemSelecionado }: ListaProps) => {
  const [loading, setLoading] = useState(false)
  const [items, setItems] = useState<ItemResponse[]>([])

  useEffect(() => {
    carregar()
  }, [filtro])

  const carregar = async () => {
    setLoading(true)
    try {
      const { data } = await listarItens(filtro)
      setItems(data)
    } catch {
      showNotification('', 'Erro ao carregar itens.', 'error', true)
    } finally {
      setLoading(false)
    }
  }

  if (loading) return <S.Loading>Carregando...</S.Loading>

  return (
    <S.Wrapper>
      {items.map((item) => (
        <S.Item
          key={item.id}
          onClick={() => onItemSelecionado?.(item.id)}
          role="button"
        >
          {item.nome}
        </S.Item>
      ))}
    </S.Wrapper>
  )
}

export default Lista{Entidade}
```

### Componente com props opcionais e renderização condicional

```tsx
export type SectionProps = {
  titulo: string
  subtitulo?: string
  actions?: React.ReactNode
  children: React.ReactNode
}

const Section = ({ titulo, subtitulo, actions, children }: SectionProps) => {
  return (
    <S.Wrapper>
      <S.Header>
        <S.TitleGroup>
          <S.Title>{titulo}</S.Title>
          {subtitulo && <S.Subtitle>{subtitulo}</S.Subtitle>}
        </S.TitleGroup>
        {actions && <S.Actions>{actions}</S.Actions>}
      </S.Header>
      <S.Content>{children}</S.Content>
    </S.Wrapper>
  )
}
```

---

## Uso do componente (no template/page)

```tsx
import Card{Entidade} from 'components/Card{Entidade}'

// Com props
<Card{Entidade}
  id={item.id}
  nome={item.nome}
  status={item.status}
  statusDescricao={item.statusDescricao}
  onSelecionar={(id) => handleSelecionar(id)}
/>

// Com children
<Section titulo="Minha Seção" subtitulo="Detalhes">
  <p>Conteúdo</p>
</Section>
```

---

## Styled-components: Props tipadas

```typescript
// Props de estilo com TypeScript
type WrapperProps = {
  variant?: 'primary' | 'secondary'
  fullWidth?: boolean
}

export const Wrapper = styled.div<WrapperProps>`
  ${({ theme, variant = 'primary', fullWidth }) => css`
    width: ${fullWidth ? '100%' : 'auto'};
    background-color: ${
      variant === 'primary' ? theme.colors.primary : theme.colors.secondary
    };
  `}
`
```

---

## Regras para Componentes

- **Pasta por componente:** sempre `{NomeComponente}/index.tsx` + `styles.ts`
- **Props tipadas e exportadas** para facilitar reutilização
- **Props opcionais com `?`**, defaults inline na destructuring
- **Styled-components com tema** via `${({ theme }) => css\`...\`}`
- **Sem chamadas de API diretas** em componentes simples — receber dados via props
- **Exceção:** componentes complexos com estado próprio podem fazer suas chamadas
- **Responsividade** com `media.lessThan()` e `media.greaterThan()`
- **Sem CSS inline** — tudo via styled-components

## Checklist

- [ ] Pasta `src/components/{NomeComponente}/` criada
- [ ] `index.tsx` com componente funcional e tipos de props exportados
- [ ] `styles.ts` com todos os elementos styled definidos
- [ ] Uso de `${({ theme }) => css\`...\`}` em todos os styled-components
- [ ] Props opcionais com `?` e defaults na destructuring
- [ ] `useEffect` com array de dependências se houver efeitos
- [ ] Erros tratados com `showNotification()` se houver chamadas assíncronas
