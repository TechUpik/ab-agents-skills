---
description: Cria um novo componente React no ab-upik-nova-jornada (Next.js)
argument-hint: <NomeComponente> [--template]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Novo Componente React — ab-upik-nova-jornada

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar um novo componente React TypeScript no repositório ab-upik-nova-jornada seguindo os padrões do projeto (functional components + styled-components + TypeScript).

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome do componente em PascalCase
2. Se `--template` foi passado: criar em `src/templates/{NomeComponente}/`; caso contrário: `src/components/{NomeComponente}/`
3. Leia componentes similares existentes (ex: `src/components/CardOrcamento/`, `src/components/CardProduto/`) para entender o padrão
4. Leia `src/styles/theme.ts` para confirmar as variáveis de tema disponíveis

### Passo 2 — Levantamento de requisitos

Se o argumento não deixou claro, pergunte ao usuário:
- Qual o propósito do componente? (card, formulário, lista, seção, botão, etc.)
- Quais props ele recebe? (nome, tipo, obrigatório/opcional)
- Faz chamadas de API próprias ou recebe dados via props?
- Usa algum hook (useAuth, useLocalStorage, outro)?
- Há estado interno que precisa gerenciar?
- Precisa ser responsivo? Quais breakpoints?

### Passo 3 — Gerar os arquivos

**Arquivo 1:** `src/{components|templates}/{NomeComponente}/index.tsx`
- Componente funcional TypeScript
- Props tipadas e exportadas (`export type {NomeComponente}Props = { ... }`)
- Destructuring com defaults para opcionais
- `useState` + `useEffect` se necessário
- `showNotification()` para erros em chamadas assíncronas
- Importar estilos como `import * as S from './styles'`

**Arquivo 2:** `src/{components|templates}/{NomeComponente}/styles.ts`
- Todos os elementos styled-components
- Uso de `${({ theme }) => css\`...\`}` com variáveis do tema
- Props de estilo com TypeScript quando necessário (`type WrapperProps = { ... }`)
- Responsividade com `styled-media-query`

### Passo 4 — Resumo

Após criar os arquivos, exibir:
- Caminhos dos arquivos criados
- Template de uso (como importar e usar no template/página pai)
- Tipo de props exportado para referência
