---
description: Cria um novo modal Vue no rep-upik-web
argument-hint: <NomeModal> --feature <NomeFeature>
allowed-tools: [Read, Glob, Grep, Write, Edit, Bash]
---

# Novo Modal Vue — rep-upik-web

Argumento recebido: **$ARGUMENTS**

## Instruções

Você vai criar um novo componente modal no repositório rep-upik-web seguindo os padrões Bootstrap Vue do projeto.

### Passo 1 — Entender o contexto

1. Extraia do argumento o nome do modal e a feature à qual pertence
2. Caminho de criação: `src/components/{Feature}/Modal{Descricao}.vue`
3. Leia modais existentes na mesma feature (ex: `src/components/Consultorias/ModalNotaFiscal.vue`) para entender o padrão
4. Leia o componente pai onde o modal será usado para entender quais dados estão disponíveis

### Passo 2 — Levantamento de requisitos

Se o argumento não deixou claro, pergunte ao usuário:
- Qual o tipo do modal? (formulário de criação, edição, detalhe somente leitura, confirmação de ação)
- Quais dados o pai passa para o modal (via props)?
- Quais eventos o modal emite para o pai? (`@hide`, `@salvo`, `@criado`, `@atualizado`)
- O modal faz chamadas de API? (GET para carregar dados, POST/PUT para salvar)
- Qual o tamanho do modal? (sm, padrão, lg, xl)
- Há validações de formulário necessárias?

### Passo 3 — Gerar os arquivos

**Arquivo 1:** `src/components/{Feature}/Modal{Descricao}.vue`
- Componente `<b-modal>` do Bootstrap Vue
- Prop `showModal: Boolean` obrigatória
- `@hidden="hideModal"` para capturar fechamento
- `$emit('hide')` no método de fechar
- `$emit('atualizado')` ou `$emit('criado')` após operação bem-sucedida
- Loading separado: `loading` para carregar dados, `loadingSalvar` para ação de salvar
- `watch: { showModal }` para carregar dados ao abrir
- `resetForm()` ao fechar o modal
- Chamadas axios com `captureError` + `Swal.fire()` para feedback

**Edição 2** (opcional): Adicionar uso do modal no componente pai se o usuário pedir

### Passo 4 — Resumo

Após criar os arquivos, exibir:
- Caminho do arquivo criado
- Template de uso no componente pai (importar + registrar + template HTML)
- Eventos emitidos pelo modal documentados
