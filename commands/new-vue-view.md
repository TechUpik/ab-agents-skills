---
description: Cria uma nova view/página no rep-upik-web (Vue.js)
argument-hint: <NomeDaView> [--publica]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Nova View — rep-upik-web

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar uma nova view/página no repositório rep-upik-web seguindo os padrões Vue.js 2 (Options API) do projeto.

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome da view (ex: `MinhaPagina`, `ListaArquitetos`)
2. Verifique se existe `--publica` no argumento — se sim, a view não requer autenticação
3. Leia `src/router.js` para entender as rotas existentes e o padrão de paths
4. Leia uma view similar existente em `src/views/` para entender o padrão (ex: `Agendamentos.vue`, `Consultorias.vue`)
5. Verifique se há componentes já criados em `src/components/` que podem ser reutilizados

### Passo 2 — Levantamento de requisitos

Se o argumento não deixou claro, pergunte ao usuário:
- Qual o propósito desta view? (listagem, formulário, detalhe, dashboard, etc.)
- Qual será o path da rota? (ex: `/minha-pagina`, `/entidade/:id`)
- A view exige autenticação? (`meta: { requiresAuth: true }`)
- Quais dados ela exibe? De qual endpoint da API?
- Precisará de modais, componentes filhos específicos?
- Qual perfil de usuário pode acessar? (cliente, arquiteto, admin, todos)

### Passo 3 — Gerar os arquivos

**Arquivo 1:** `src/views/{NomeDaView}.vue`
- Options API com `data()`, `methods`, `created()`
- `axios` direto nos métodos, usando `config.API_URL` ou `config.API_NOVA_JORNADA`
- Estado de loading com componente `<Loading />`
- Erros tratados com `captureError(e)` + `Swal.fire()`
- Filtros globais onde aplicável (`| formatDate`, `| formatMoney`)
- Bootstrap Vue para componentes UI
- Estilos LESS com `scoped`

**Edição 2:** `src/router.js`
- Adicionar a nova rota no array `routes`
- Lazy loading: `component: () => import('./views/{NomeDaView}.vue')`
- `meta: { requiresAuth: true }` (ou false se pública)
- Path correto baseado no contexto

### Passo 4 — Resumo

Após criar os arquivos, exibir:
- Arquivos criados/editados com seus caminhos
- URL de acesso da nova view
- Template do link de menu que o desenvolvedor pode adicionar ao sidebar
- Lista de componentes filhos que precisam ser criados separadamente (se houver)
