---
description: Cria uma nova página no ab-upik-nova-jornada (Next.js)
argument-hint: <nome-da-pagina> [--publica] [--param <nomeDoPar>]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Nova Page — ab-upik-nova-jornada

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar uma nova página no repositório ab-upik-nova-jornada seguindo os padrões Next.js + TypeScript + Styled-Components do projeto.

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome da página em kebab-case (ex: `minha-pagina`, `dados-pessoais`)
2. Verifique se `--publica` foi passado — se não, usar `withAuth`
3. Verifique se `--param` foi passado — se sim, criar com rota dinâmica `[nomeDoPar].tsx`
4. Leia `src/pages/` para ver as páginas existentes e o padrão adotado
5. Leia `src/templates/` para ver templates similares existentes
6. Leia `src/service/` para identificar serviços que podem ser reutilizados

### Passo 2 — Levantamento de requisitos

Se o argumento não deixou claro, pergunte ao usuário:
- Qual o propósito desta página? (listagem, formulário, detalhe, dashboard, etc.)
- Requer autenticação? (padrão: sim)
- Há parâmetro na rota? (ex: `/entidade/[id]`)
- Quais dados ela exibe? De qual endpoint?
- Quais componentes existentes ela deve reutilizar?
- Usa algum context (useAuth, outro)?

### Passo 3 — Gerar os arquivos

**Arquivo 1:** `src/pages/{nome-da-pagina}.tsx`
- Import do template
- `withAuth` se protegida (padrão)
- Passar parâmetros de rota via `useRouter()` se necessário

**Arquivo 2:** `src/templates/{NomeTemplate}/index.tsx`
- Componente funcional TypeScript
- Hooks React (`useState`, `useEffect`)
- Chamadas de API via `service/{dominio}`
- `showNotification()` para feedback de erros
- TypeScript: props e dados tipados
- `react-hook-form` + `yup` para formulários

**Arquivo 3:** `src/templates/{NomeTemplate}/styles.ts`
- Styled-components usando tema (`${({ theme }) => css\`...\`}`)
- Responsividade com `media.lessThan()` e `media.greaterThan()`

**Arquivo 4** (se necessário): `src/service/{dominio}/index.ts`
- Criar ou adicionar funções ao service do domínio
- Interfaces de request/response tipadas

### Passo 4 — Resumo

Após criar os arquivos, exibir:
- Arquivos criados com seus caminhos
- URL de acesso da nova página (`/{nome-da-pagina}`)
- Interfaces TypeScript criadas no service
- Componentes que o desenvolvedor pode precisar criar adicionalmente
