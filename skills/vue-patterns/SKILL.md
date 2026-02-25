---
name: vue-patterns
description: This skill should be used when the user asks about how the rep-upik-web Vue.js repository works, asks about patterns, architecture, conventions, component structure, API calls, routing, or asks "how do we do X in this project". Also activate when the user mentions Vue, Options API, Bootstrap Vue, axios, router, views, components, auth.js, config.js, helper.js, filters, LESS, or describes working on the rep-upik-web frontend.
version: 1.0.0
---

# Padrões do Repositório rep-upik-web (Vue.js 2)

## Arquitetura Geral

O projeto é uma SPA em **Vue.js 2** com **Options API**, sem Vuex. Estado é gerenciado via `data()`, props, eventos e `localStorage`.

```
/src
├── views/              → Páginas/views (PascalCase.vue)
├── components/         → Componentes reutilizáveis
│   └── {Feature}/      → Subpastas para features complexas (ex: Consultorias/)
├── core/
│   ├── languages/      → i18n (JSON por página)
│   ├── validators/     → Validadores custom (CPF, CNPJ, CEP, etc.)
│   └── consts/         → Constantes
├── styles/             → LESS global
├── assets/             → Imagens, fontes, SVGs
├── white-label/        → Customizações por parceiro
├── auth.js             → Autenticação (localStorage + axios header)
├── config.js           → Config por ambiente (development/staging/production)
├── router.js           → Vue Router com guards de autenticação
├── filters.js          → Filtros globais (formatDate, formatMoney, etc.)
├── helper.js           → Funções utilitárias
├── main.js             → Entry point (plugins, filtros globais)
└── captureError.js     → Tracking de erros
```

## Estrutura de Componente (Options API)

```vue
<template>
  <!-- Template usando Bootstrap Vue e Bootstrap 4 -->
</template>

<script>
import axios from 'axios';
import Auth from '../auth';
import config from '../config';
import helper from '../helper';
import ComponenteFilho from '../components/ComponenteFilho.vue';

export default {
  // Componentes filhos registrados aqui
  components: {
    ComponenteFilho,
  },

  // Props recebidas do pai
  props: {
    showModal: Boolean,
    dataItem: Object,
    itemId: String,
  },

  // Estado local reativo
  data() {
    return {
      loading: false,
      items: [],
      form: {
        campo1: '',
        campo2: null,
      },
      // Expor utilitários no template quando necessário
      Auth,
      config,
      helper,
    };
  },

  // Propriedades calculadas
  computed: {
    isAdmin() {
      return Auth.getUserInfo().perfil === config.PERFIL_ADMIN;
    },
  },

  // Watchers reativos
  watch: {
    itemId(newVal) {
      if (newVal) this.carregarDados();
    },
  },

  // Ciclo de vida
  created() {
    this.carregarDados();
  },
  destroyed() {
    window.removeEventListener('resize', this.onResize);
  },

  // Lógica
  methods: {
    async carregarDados() {
      this.loading = true;
      try {
        const response = await axios.get(`${config.API_URL}/Entidade/${this.itemId}`);
        this.items = response.data;
      } catch (e) {
        captureError(e);
        this.$swal({ icon: 'error', title: 'Erro', text: 'Não foi possível carregar os dados.' });
      } finally {
        this.loading = false;
      }
    },

    // Emitir evento para o pai
    fechar() {
      this.$emit('hide');
    },
  },
};
</script>

<style scoped lang="less">
/* Styles LESS escopados ao componente */
.container-custom {
  padding: 16px;
}
</style>
```

## Chamadas de API

**Padrão: axios direto no método, sem service layer centralizado**

```javascript
// Config de endpoint
import config from '../config';

// Endpoints disponíveis em config.js:
// config.API_URL              → AdB API principal (ex: adb-api-v1-dev.azurewebsites.net/api)
// config.API_NOVA_JORNADA     → Nova jornada API (ex: localhost:5001/api)
// config.API_CHAT_INTERNO     → Chat interno
// config.API_MATCH            → Serviço de match

// GET
const response = await axios.get(`${config.API_URL}/Entidade/${id}`);
const dados = response.data;

// POST
const response = await axios.post(`${config.API_URL}/Entidade`, payload);

// PUT
await axios.put(`${config.API_URL}/Entidade/${id}`, payload);

// DELETE
await axios.delete(`${config.API_URL}/Entidade/${id}`);

// Com then/catch (quando não usa async/await)
axios.put(`${config.API_URL}/Entidade/${id}`, payload)
  .then(response => {
    // Sucesso
  })
  .catch(e => {
    captureError(e);
    Swal.fire({ icon: 'error', title: 'Oops...', text: 'Mensagem de erro.' });
  });
```

**Authorization:** Automaticamente injetada via `auth.js`:
```javascript
axios.defaults.headers.common['Authorization'] = `Bearer ${token}`;
```

## Roteamento

**Arquivo:** `src/router.js`

```javascript
// Rota simples
{
  path: '/minha-pagina',
  component: () => import('./views/MinhaPagina.vue'),
  meta: { requiresAuth: true },
}

// Rota com parâmetro
{
  path: '/entidade/:id/detalhe',
  component: () => import('./views/DetalheEntidade.vue'),
  meta: { requiresAuth: true },
}

// Rota pública (sem auth)
{
  path: '/pagina-publica',
  component: PaginaPublica,   // import estático para carregamento imediato
  meta: { requiresAuth: false },
}
```

**Lazy loading** (preferido para a maioria das views):
```javascript
component: () => import('./views/MinhaView.vue')
```

**Guards de autenticação** já configurados globalmente — apenas setar `meta.requiresAuth`.

**Navegação programática:**
```javascript
this.$router.push('/destino');
this.$router.push({ path: `/entidade/${id}` });
this.$router.go(-1);  // voltar
```

## Autenticação

```javascript
import Auth from '../auth';

// Verificar login
if (!Auth.isLoggedIn()) { /* redirecionar */ }

// Dados do usuário
const userInfo = Auth.getUserInfo();
// { nome, perfil, id, clienteId, email }

// Verificar perfil
if (Auth.getUserInfo().perfil === config.PERFIL_CLIENTE) { /* */ }
if (Auth.getUserInfo().perfil === config.PERFIL_ARQUITETO) { /* */ }
if (Auth.getUserInfo().perfil === config.PERFIL_ADMIN) { /* */ }

// Login/Logout
Auth.login(token, dtExpiry, userInfo);
Auth.logout();
```

## Filtros Globais

Disponíveis em qualquer template via `{{ valor | filtro }}`:

```vue
{{ item.dataCriacao | formatDate }}         <!-- DD/MM/YYYY -->
{{ item.dataCriacao | formatDateTime }}      <!-- DD/MM/YYYY HH:mm -->
{{ item.dataCriacao | formatLocalDate }}     <!-- Converte UTC → local DD/MM/YYYY -->
{{ item.dataCriacao | formatLocalDateTime }} <!-- Converte UTC → local DD/MM/YYYY HH:mm -->
{{ item.valor | formatMoney }}              <!-- R$ 1.234,56 -->
{{ item.telefone | formatPhone }}           <!-- (11) 99999-9999 -->
{{ item.cpf | formatCpf }}                 <!-- 000.000.000-00 -->
{{ item.cnpj | formatCnpj }}               <!-- 00.000.000/0001-00 -->
```

## Helper (helper.js)

```javascript
import helper from '../helper';

helper.obterPrimeirosCaracteres(texto, 50)  // truncar texto
helper.obterPrimeiroNome(nomeCompleto)       // primeiro nome
helper.formatarTelefone(valor)               // formatar telefone
helper.formatarMoeda(valor)                 // formatar moeda
helper.getConsultoriaStatusStyle(idStatus)  // cor por status
```

## Bootstrap Vue (UI Framework)

```vue
<!-- Modal -->
<b-modal
  :id="`modal-${identificador}`"
  :visible="showModal"
  title="Título"
  size="lg"
  :hide-footer="true"
  @hidden="onHidden"
>
  <template #modal-footer>
    <b-button variant="secondary" @click="fechar">Cancelar</b-button>
    <b-button variant="primary" @click="salvar" :disabled="loading">Salvar</b-button>
  </template>
</b-modal>

<!-- Botões -->
<b-button variant="primary" :disabled="loading" @click="acao">Texto</b-button>
<b-button variant="outline-danger" size="sm" @click="remover">Remover</b-button>

<!-- Formulário -->
<b-form-checkbox v-model="ativo" name="ativo">Ativo</b-form-checkbox>
<b-badge variant="secondary">Status</b-badge>

<!-- Grid Bootstrap 4 -->
<div class="container-fluid">
  <div class="row">
    <div class="col-md-6">...</div>
    <div class="col-md-6">...</div>
  </div>
</div>
```

## Sweetalert2

```javascript
import Swal from 'sweetalert2';

// Confirmação
const result = await Swal.fire({
  icon: 'warning',
  title: 'Confirmar?',
  text: 'Deseja confirmar esta ação?',
  showCancelButton: true,
  confirmButtonText: 'Sim',
  cancelButtonText: 'Cancelar',
});
if (result.isConfirmed) { /* ação */ }

// Erro
Swal.fire({ icon: 'error', title: 'Oops...', text: 'Mensagem de erro.' });

// Sucesso
Swal.fire({ icon: 'success', title: 'Sucesso!', text: 'Operação realizada.' });
```

## Conventions de Nomenclatura

| Tipo | Padrão | Exemplos |
|------|--------|----------|
| Views | `PascalCase.vue` | `Consultorias.vue`, `Dashboard.vue` |
| Componentes simples | `PascalCase.vue` na pasta `/components` | `Loading.vue`, `Footer.vue` |
| Componentes de feature | pasta + arquivo em `/components/{Feature}/` | `Consultorias/ModalNotaFiscal.vue` |
| Modais | `Modal{Descricao}.vue` | `ModalRegistrarAgendamento.vue` |
| Dados locais | camelCase | `loading`, `items`, `showModal` |
| Métodos | camelCase, verbo primeiro | `carregarDados()`, `onClickSalvar()`, `hideModal()` |
| Eventos emitidos | kebab-case | `$emit('hide')`, `$emit('dados-atualizados', dados)` |
| Props | camelCase | `showModal`, `dataConsultoria` |

## Diretivas Comuns

```vue
<!-- Two-way binding -->
<input v-model="campo" />

<!-- Condicional -->
<div v-if="loading">Carregando...</div>
<div v-else>Conteúdo</div>
<div v-show="isVisible">Visível/oculto sem remover DOM</div>

<!-- Lista -->
<div v-for="item in items" :key="item.id">{{ item.nome }}</div>

<!-- Bind de classe -->
<div :class="{ 'active': isActive, 'disabled': isDisabled }"></div>
<div :class="[classeBase, classeDinamica]"></div>

<!-- Eventos -->
<button @click="acao">Clique</button>
<button @click.prevent="acao">Sem default</button>
<input @keyup.enter="acao" />

<!-- Perfil check no template -->
<div v-if="Auth.getUserInfo().perfil == config.PERFIL_CLIENTE">
  Apenas clientes
</div>
```

## i18n / Textos

```javascript
// Importar JSON de linguagem
import languagePage from '../core/languages/pages/nomePagina.json';

data() {
  return {
    language: languagePage,
  };
}
// No template: {{ language.titulo }}
```

## Configuração por Ambiente

```javascript
import config from '../config';

// Endpoints
config.API_URL              // API principal
config.API_NOVA_JORNADA     // Nova jornada
config.API_CHAT_INTERNO     // Chat interno
config.API_MATCH            // Match

// Feature flags
config.ENABLE_MARKETING_TRACKERS

// Perfis
config.PERFIL_CLIENTE
config.PERFIL_ARQUITETO
config.PERFIL_ADMIN

// Status de consultoria
config.CONSULTORIA_AGENDADA
config.CONSULTORIA_CONFIRMADA
// etc.
```
