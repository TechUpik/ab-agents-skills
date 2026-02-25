---
name: vue-create-view
description: This skill should be used when the user wants to create a new page, view, or screen in the rep-upik-web Vue.js repository. Activate when the user mentions "criar view", "nova view", "nova página", "nova tela", "criar página vue", "adicionar rota", "criar tela vue", or describes building a new page/screen for the Vue frontend.
version: 1.0.0
---

# Criar uma nova View no rep-upik-web

## O que gerar

Para criar uma nova View/Página completa são necessários:
1. **Arquivo da View** — em `src/views/{NomeDaView}.vue`
2. **Registro da Rota** — em `src/router.js`
3. **Componentes filhos** (se necessário) — em `src/components/{Feature}/`

---

## Passo 1 — Arquivo da View

**Caminho:** `src/views/{NomeDaView}.vue`

### View simples (listagem com chamada API)

```vue
<template>
  <div class="container-fluid">
    <div class="row py-3">
      <div class="col-12">
        <h4>{Título da Página}</h4>
      </div>
    </div>

    <!-- Loading state -->
    <div v-if="loading" class="text-center py-5">
      <Loading width="60" />
      <p class="mt-2">Carregando...</p>
    </div>

    <!-- Conteúdo principal -->
    <div v-if="!loading">
      <div class="row">
        <div v-for="item in items" :key="item.id" class="col-md-6 col-lg-4 mb-3">
          <!-- Card de item -->
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">{{ item.nome }}</h5>
              <p class="card-text">{{ item.descricao }}</p>
              <button
                class="btn btn-primary btn-sm"
                @click="onClickDetalhe(item)"
              >
                Ver detalhe
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Sem resultados -->
      <div v-if="items.length === 0" class="text-center py-5">
        <p class="text-muted">Nenhum item encontrado.</p>
      </div>
    </div>

    <!-- Modal de detalhe -->
    <Modal{Entidade}
      :showModal="showModalDetalhe"
      :dataItem="itemSelecionado"
      @hide="showModalDetalhe = false"
      @atualizado="carregarDados"
    />
  </div>
</template>

<script>
import axios from 'axios';
import config from '../config';
import captureError from '../captureError';
import Loading from '../components/Loading';
import Modal{Entidade} from '../components/{Feature}/Modal{Entidade}.vue';

export default {
  components: {
    Loading,
    Modal{Entidade},
  },

  data() {
    return {
      loading: false,
      items: [],
      showModalDetalhe: false,
      itemSelecionado: null,
    };
  },

  created() {
    this.carregarDados();
  },

  methods: {
    async carregarDados() {
      this.loading = true;
      try {
        const response = await axios.get(`${config.API_URL}/{Entidade}`);
        this.items = response.data;
      } catch (e) {
        captureError(e);
        this.$swal({
          icon: 'error',
          title: 'Oops...',
          text: 'Não foi possível carregar os dados.',
        });
      } finally {
        this.loading = false;
      }
    },

    onClickDetalhe(item) {
      this.itemSelecionado = item;
      this.showModalDetalhe = true;
    },
  },
};
</script>

<style scoped lang="less">
/* Estilos específicos desta view */
</style>
```

### View com filtros e busca

```vue
<template>
  <div class="container-fluid">
    <!-- Filtros -->
    <div class="row py-3">
      <div class="col-md-4">
        <input
          v-model="filtro.termoPesquisa"
          type="text"
          class="form-control"
          placeholder="Buscar..."
          @keyup.enter="carregarDados"
        />
      </div>
      <div class="col-md-3">
        <select v-model="filtro.status" class="form-control" @change="carregarDados">
          <option :value="null">Todos</option>
          <option value="ativo">Ativo</option>
          <option value="inativo">Inativo</option>
        </select>
      </div>
      <div class="col-md-2">
        <button class="btn btn-primary" @click="carregarDados">Filtrar</button>
      </div>
    </div>

    <!-- Tabela -->
    <div class="row">
      <div class="col-12">
        <table class="table table-sm table-hover">
          <thead>
            <tr>
              <th>Nome</th>
              <th>Status</th>
              <th>Data</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in items" :key="item.id">
              <td>{{ item.nome }}</td>
              <td>{{ item.status }}</td>
              <td>{{ item.criado | formatDate }}</td>
              <td>
                <button class="btn btn-sm btn-outline-primary" @click="onClickEditar(item)">
                  Editar
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import config from '../config';
import captureError from '../captureError';

export default {
  data() {
    return {
      loading: false,
      items: [],
      filtro: {
        termoPesquisa: '',
        status: null,
      },
    };
  },

  created() {
    this.carregarDados();
  },

  methods: {
    async carregarDados() {
      this.loading = true;
      try {
        const params = {};
        if (this.filtro.termoPesquisa) params.termoPesquisa = this.filtro.termoPesquisa;
        if (this.filtro.status) params.status = this.filtro.status;

        const response = await axios.get(`${config.API_URL}/{Entidade}`, { params });
        this.items = response.data;
      } catch (e) {
        captureError(e);
      } finally {
        this.loading = false;
      }
    },

    onClickEditar(item) {
      // lógica de edição
    },
  },
};
</script>
```

---

## Passo 2 — Registro da Rota

**Arquivo:** `src/router.js`

Adicionar dentro do array `routes`:

```javascript
// View protegida (lazy loading — preferido)
{
  path: '/minha-view',
  component: () => import('./views/{NomeDaView}.vue'),
  meta: { requiresAuth: true },
},

// View com parâmetro de rota
{
  path: '/entidade/:id',
  component: () => import('./views/{NomeDaView}.vue'),
  meta: { requiresAuth: true },
},

// View pública (import estático)
{
  path: '/pagina-publica',
  component: PaginaPublica,
  meta: { requiresAuth: false },
},
```

**Acesso ao parâmetro de rota na view:**
```javascript
// No created() ou mounted()
const id = this.$route.params.id;

// No template
{{ $route.params.id }}
```

---

## Passo 3 — Link de navegação (se necessário)

**No menu/sidebar:**
```vue
<router-link to="/minha-view">Nome do Menu</router-link>

<!-- Com parâmetro -->
<router-link :to="`/entidade/${item.id}`">Ver detalhe</router-link>
```

**Navegação programática:**
```javascript
this.$router.push('/minha-view');
this.$router.push({ path: `/entidade/${id}` });
```

---

## Regras para Views

- Nome do arquivo: **PascalCase** (`MinhaView.vue`)
- Sempre verificar perfil de usuário quando necessário: `Auth.getUserInfo().perfil`
- Usar `v-if="loading"` + componente `Loading` para feedback
- Tratar erros com `captureError(e)` + `Swal.fire(...)` para feedback visual
- Filtros de formatação via pipes: `{{ valor | formatDate }}`, `{{ valor | formatMoney }}`
- Lazy loading na rota (`() => import(...)`) para todas as views, exceto as carregadas no início
- `meta: { requiresAuth: true }` em todas as views que requerem login

## Checklist

- [ ] Arquivo `{NomeDaView}.vue` criado em `src/views/`
- [ ] Rota adicionada em `src/router.js` com `meta.requiresAuth` correto
- [ ] `loading` state com componente `Loading` enquanto carrega dados
- [ ] Erros tratados com `captureError(e)` + feedback visual ao usuário
- [ ] `v-for` sempre com `:key` único
- [ ] Filtros de formatação usados onde aplicável
- [ ] Componentes filhos importados e registrados em `components: {}`
- [ ] Link/item de menu adicionado (se necessário)
