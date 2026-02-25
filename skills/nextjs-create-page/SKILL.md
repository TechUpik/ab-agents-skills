---
name: nextjs-create-page
description: This skill should be used when the user wants to create a new page or screen in the ab-upik-nova-jornada Next.js repository. Activate when the user mentions "criar página", "nova página next", "nova rota next", "criar tela next", "criar page next", "adicionar página", or describes building a new page/screen for the Next.js frontend.
version: 1.0.0
---

# Criar uma nova Page no ab-upik-nova-jornada

## O que gerar

Para criar uma nova página completa são necessários:
1. **Arquivo da Page** — em `src/pages/{nome-kebab-case}.tsx` (ou subpasta)
2. **Template** — em `src/templates/{NomeTemplate}/index.tsx` + `styles.ts`
3. **Serviço** (se necessário) — em `src/service/{dominio}/index.ts`

---

## Estrutura de Arquivos

```
src/
├── pages/
│   └── minha-pagina.tsx          ← Arquivo de rota (mínimo)
└── templates/
    └── MinhaPagina/
        ├── index.tsx             ← Lógica e composição da página
        └── styles.ts             ← Styled-components
```

---

## Passo 1 — Arquivo da Page

**Caminho:** `src/pages/{nome-da-pagina}.tsx`

### Página protegida (requer auth — mais comum)

```tsx
import withAuth from 'components/Auth/WithAuth'
import MinhaPaginaTemplate from 'templates/MinhaPagina'

function MinhaPagina() {
  return <MinhaPaginaTemplate />
}

export default withAuth(MinhaPagina)
```

### Página pública (sem auth)

```tsx
import MinhaPaginaTemplate from 'templates/MinhaPagina'

export default function MinhaPagina() {
  return <MinhaPaginaTemplate />
}
```

### Página com parâmetro de rota dinâmica

```
src/pages/entidade/[id].tsx
```

```tsx
import { useRouter } from 'next/router'
import withAuth from 'components/Auth/WithAuth'
import EntidadeDetalheTemplate from 'templates/EntidadeDetalhe'

function EntidadeDetalhe() {
  const router = useRouter()
  const { id } = router.query

  return <EntidadeDetalheTemplate id={id as string} />
}

export default withAuth(EntidadeDetalhe)
```

---

## Passo 2 — Template (lógica e layout)

**Caminho:** `src/templates/{NomeTemplate}/index.tsx`

### Template simples (listagem)

```tsx
import { useEffect, useState } from 'react'
import * as S from './styles'
import { showNotification } from 'utils/notification'
import { listarEntidades } from 'service/entidade'
import type { EntidadeResponse } from 'service/entidade'
import CardEntidade from 'components/CardEntidade'

const {NomeTemplate} = () => {
  const [loading, setLoading] = useState(false)
  const [items, setItems] = useState<EntidadeResponse[]>([])

  useEffect(() => {
    carregarDados()
  }, [])

  const carregarDados = async () => {
    setLoading(true)
    try {
      const { data } = await listarEntidades()
      setItems(data)
    } catch (error) {
      showNotification('Erro', 'Não foi possível carregar os dados.', 'error')
    } finally {
      setLoading(false)
    }
  }

  if (loading) {
    return (
      <S.Wrapper>
        <S.LoadingContainer>
          <p>Carregando...</p>
        </S.LoadingContainer>
      </S.Wrapper>
    )
  }

  return (
    <S.Wrapper>
      <S.Header>
        <S.Title>Título da Página</S.Title>
      </S.Header>

      <S.Content>
        {items.length === 0 ? (
          <S.EmptyState>Nenhum item encontrado.</S.EmptyState>
        ) : (
          <S.Grid>
            {items.map((item) => (
              <CardEntidade key={item.id} {...item} />
            ))}
          </S.Grid>
        )}
      </S.Content>
    </S.Wrapper>
  )
}

export default {NomeTemplate}
```

### Template com form de edição

```tsx
import { useEffect, useState } from 'react'
import { useForm, Controller } from 'react-hook-form'
import { yupResolver } from '@hookform/resolvers/yup'
import * as yup from 'yup'
import * as S from './styles'
import { showNotification } from 'utils/notification'
import { obterEntidade, atualizarEntidade } from 'service/entidade'
import useAuth from 'hooks/useAuth'

interface IFormInputs {
  nome: string
  descricao?: string
}

const schema = yup.object({
  nome: yup.string().required('Nome é obrigatório').label('nome'),
  descricao: yup.string().optional().label('descrição'),
}).required()

type Props = {
  id: string
}

const {NomeTemplate} = ({ id }: Props) => {
  const { usuario } = useAuth()
  const [loading, setLoading] = useState(false)

  const {
    handleSubmit,
    control,
    reset,
    formState: { errors, isSubmitting },
  } = useForm<IFormInputs>({
    resolver: yupResolver(schema),
    defaultValues: { nome: '', descricao: '' },
  })

  useEffect(() => {
    if (id) carregarDados()
  }, [id])

  const carregarDados = async () => {
    setLoading(true)
    try {
      const { data } = await obterEntidade(id)
      reset({ nome: data.nome, descricao: data.descricao })
    } catch {
      showNotification('Erro', 'Não foi possível carregar os dados.', 'error')
    } finally {
      setLoading(false)
    }
  }

  const onSubmit = async (data: IFormInputs) => {
    try {
      await atualizarEntidade(id, data)
      showNotification('', 'Dados salvos com sucesso!', 'success', true)
    } catch {
      showNotification('Erro', 'Não foi possível salvar.', 'error')
    }
  }

  return (
    <S.Wrapper>
      <S.Title>Editar {Entidade}</S.Title>

      <S.Form onSubmit={handleSubmit(onSubmit)}>
        <S.FormGroup>
          <label>Nome</label>
          <Controller
            name="nome"
            control={control}
            render={({ field }) => (
              <S.Input {...field} placeholder="Nome" />
            )}
          />
          {errors.nome && <S.ErrorMsg>{errors.nome.message}</S.ErrorMsg>}
        </S.FormGroup>

        <S.FormGroup>
          <label>Descrição</label>
          <Controller
            name="descricao"
            control={control}
            render={({ field }) => (
              <S.Textarea {...field} placeholder="Descrição" />
            )}
          />
        </S.FormGroup>

        <S.Button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Salvando...' : 'Salvar'}
        </S.Button>
      </S.Form>
    </S.Wrapper>
  )
}

export default {NomeTemplate}
```

---

## Passo 3 — Styled-components do Template

**Caminho:** `src/templates/{NomeTemplate}/styles.ts`

```typescript
import styled, { css } from 'styled-components'
import media from 'styled-media-query'

export const Wrapper = styled.div`
  ${({ theme }) => css`
    width: 100%;
    min-height: 100vh;
    padding: ${theme.spacings.medium};
    background-color: ${theme.colors.white};
  `}
`

export const Header = styled.header`
  ${({ theme }) => css`
    margin-bottom: ${theme.spacings.medium};
  `}
`

export const Title = styled.h1`
  ${({ theme }) => css`
    color: ${theme.colors.black};
    font-size: ${theme.font.sizes.xxlarge};
    font-weight: ${theme.font.bold};

    ${media.lessThan('medium')`
      font-size: ${theme.font.sizes.xlarge};
    `}
  `}
`

export const Content = styled.main`
  ${({ theme }) => css`
    width: 100%;
  `}
`

export const Grid = styled.div`
  ${({ theme }) => css`
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: ${theme.spacings.small};

    ${media.lessThan('medium')`
      grid-template-columns: 1fr;
    `}
  `}
`

export const EmptyState = styled.p`
  ${({ theme }) => css`
    color: ${theme.colors.black};
    text-align: center;
    padding: ${theme.spacings.xlarge};
    opacity: 0.5;
  `}
`

export const LoadingContainer = styled.div`
  ${({ theme }) => css`
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 200px;
  `}
`

export const Form = styled.form`
  ${({ theme }) => css`
    max-width: 600px;
  `}
`

export const FormGroup = styled.div`
  ${({ theme }) => css`
    margin-bottom: ${theme.spacings.small};

    label {
      display: block;
      margin-bottom: ${theme.spacings.xxsmall};
      font-weight: ${theme.font.bold};
    }
  `}
`

export const Input = styled.input`
  ${({ theme }) => css`
    width: 100%;
    padding: ${theme.spacings.xxsmall} ${theme.spacings.xsmall};
    border: 1px solid #ddd;
    border-radius: ${theme.border.radius};
    font-size: ${theme.font.sizes.medium};
  `}
`

export const Textarea = styled.textarea`
  ${({ theme }) => css`
    width: 100%;
    padding: ${theme.spacings.xxsmall} ${theme.spacings.xsmall};
    border: 1px solid #ddd;
    border-radius: ${theme.border.radius};
    font-size: ${theme.font.sizes.medium};
    min-height: 100px;
    resize: vertical;
  `}
`

export const ErrorMsg = styled.span`
  ${({ theme }) => css`
    color: #e00;
    font-size: ${theme.font.sizes.small};
    margin-top: 4px;
    display: block;
  `}
`

export const Button = styled.button`
  ${({ theme }) => css`
    background-color: ${theme.colors.primary};
    color: ${theme.colors.white};
    padding: ${theme.spacings.xxsmall} ${theme.spacings.medium};
    border-radius: ${theme.border.radius};
    border: none;
    font-size: ${theme.font.sizes.medium};
    cursor: pointer;
    transition: ${theme.transition.default};

    &:hover:not(:disabled) {
      background-color: ${theme.colors.primaryHover};
    }

    &:disabled {
      opacity: 0.6;
      cursor: not-allowed;
    }
  `}
`
```

---

## Roteamento Next.js (file-based)

| Arquivo | Rota |
|---------|------|
| `pages/minha-pagina.tsx` | `/minha-pagina` |
| `pages/entidade/[id].tsx` | `/entidade/:id` |
| `pages/entidade/index.tsx` | `/entidade` |
| `pages/entidade/[id]/editar.tsx` | `/entidade/:id/editar` |

**Acessar parâmetro de rota:**
```tsx
import { useRouter } from 'next/router'
const router = useRouter()
const { id } = router.query       // parâmetro da URL
const { tab } = router.query      // query string ?tab=valor
```

**Navegação:**
```tsx
import { useRouter } from 'next/router'
const router = useRouter()

router.push('/destino')
router.push(`/entidade/${id}`)
router.back()
```

---

## Checklist

- [ ] Arquivo de página criado em `src/pages/` com kebab-case
- [ ] `withAuth` aplicado se a página requer autenticação
- [ ] Template criado em `src/templates/{NomeTemplate}/` com `index.tsx` + `styles.ts`
- [ ] Loading state implementado (com condicional ou spinner)
- [ ] Erros tratados com `showNotification()`
- [ ] TypeScript: Props tipadas, interfaces para dados da API
- [ ] `useEffect` com array de dependências declarado
- [ ] Formulários usando `react-hook-form` + `yup`
