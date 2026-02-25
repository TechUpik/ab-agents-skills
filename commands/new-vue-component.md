---
description: Cria um novo componente Vue no rep-upik-web
argument-hint: <NomeComponente> [--feature <NomeFeature>]
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Novo Componente Vue — rep-upik-web

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar um novo componente Vue.js reutilizável no repositório rep-upik-web seguindo os padrões Options API do projeto.

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome do componente e, se fornecido com `--feature`, a feature à qual pertence
2. Se `--feature` fornecida: criar em `src/components/{Feature}/{NomeComponente}.vue`
3. Caso contrário: criar em `src/components/{NomeComponente}.vue`
4. Leia componentes similares existentes em `src/components/` para entender o padrão
5. Verifique se o componente precisará de algum modal filho (para criar separadamente)

### Passo 2 — Levantamento de requisitos

Se o argumento não deixou claro, pergunte ao usuário:
- Qual o propósito deste componente? (card, lista, formulário, seção, etc.)
- Quais props ele recebe?
- Quais eventos ele emite para o pai?
- Ele faz chamadas de API próprias ou recebe dados via props?
- Há componentes filhos que ele usa?
- Qual perfil de usuário o usa? (para condicionar exibição com `Auth.getUserInfo().perfil`)

### Passo 3 — Gerar o arquivo

**Arquivo:** `src/components/{[Feature]/}{NomeComponente}.vue`
- Options API com `props` tipadas (`type`, `required`, `default`)
- `data()` para estado local
- `computed` para propriedades derivadas
- `watch` para reagir a mudanças de props
- `methods` com lógica e `$emit` para comunicação com pai
- Estilos LESS `scoped`
- Suporte a `v-model` se for componente de input (declarar `model: { prop, event }`)
- Bootstrap Vue para UI components
- Uso de `captureError` e `Swal.fire()` para tratamento de erros se houver chamadas API

### Passo 4 — Resumo

Após criar o arquivo, exibir:
- Caminho do arquivo criado
- Template de uso no componente pai (como importar e usar)
- Lista de props e eventos documentados
