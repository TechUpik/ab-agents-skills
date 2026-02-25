---
name: vue-create-component
description: This skill should be used when the user wants to create a new reusable component in the rep-upik-web Vue.js repository. Activate when the user mentions "criar componente", "novo componente vue", "criar componente reutilizável", "componente vue", "criar card", "criar formulário vue", or describes building a reusable UI piece for the Vue frontend (excluding modals — use vue-create-modal for modals).
version: 1.0.0
---

# Criar um novo Componente no rep-upik-web

## Onde criar

| Tipo | Caminho |
|------|---------|
| Componente genérico/shared | `src/components/{NomeComponente}.vue` |
| Componente de uma feature específica | `src/components/{Feature}/{NomeComponente}.vue` |

---

## Estrutura padrão

```vue
<template>
  <div class="componente-wrapper">
    <!-- Conteúdo -->
  </div>
</template>

<script>
// Imports de utilitários quando necessário
import Auth from '../auth';
import config from '../config';
import helper from '../helper';

export default {
  // Props com tipos e defaults declarados
  props: {
    // Prop obrigatória
    itemId: {
      type: String,
      required: true,
    },
    // Prop com default
    titulo: {
      type: String,
      default: '',
    },
    // Prop objeto
    dataItem: {
      type: Object,
      default: null,
    },
    // Prop booleana
    readonly: {
      type: Boolean,
      default: false,
    },
  },

  // Componentes filhos
  components: {
    // ComponenteFilho,
  },

  data() {
    return {
      loading: false,
      // Estado local
    };
  },

  computed: {
    // Propriedades derivadas
  },

  watch: {
    // Reagir a mudanças de props
    itemId(novoId) {
      if (novoId) this.carregarDados();
    },
  },

  created() {
    // Inicialização
  },

  methods: {
    // Comunicar para o pai via $emit
    onAcao() {
      this.$emit('acao', dadosParaOPai);
    },

    onFechar() {
      this.$emit('hide');
    },
  },
};
</script>

<style scoped lang="less">
/* Estilos escopados — não vazam para outros componentes */
.componente-wrapper {
  /* ... */
}
</style>
```

---

## Exemplos por tipo de componente

### Card de listagem

```vue
<template>
  <div class="card mb-3">
    <div class="card-body d-flex align-items-center justify-content-between">
      <div>
        <h5 class="card-title mb-1">{{ item.nome }}</h5>
        <small class="text-muted">{{ item.criado | formatLocalDate }}</small>
      </div>
      <div>
        <span
          class="badge"
          :class="getStatusClass(item.status)"
        >{{ item.statusDescricao }}</span>
      </div>
      <div>
        <button
          class="btn btn-sm btn-outline-primary"
          @click="$emit('selecionar', item)"
        >
          Selecionar
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    item: {
      type: Object,
      required: true,
    },
  },

  methods: {
    getStatusClass(status) {
      const classes = {
        ativo: 'badge-success',
        inativo: 'badge-secondary',
        pendente: 'badge-warning',
      };
      return classes[status] || 'badge-secondary';
    },
  },
};
</script>
```

### Componente de formulário

```vue
<template>
  <div>
    <div class="form-group">
      <label>{{ label }}</label>
      <input
        v-model="valorInterno"
        :type="tipo"
        class="form-control"
        :class="{ 'is-invalid': erro }"
        :placeholder="placeholder"
        :disabled="disabled"
        @input="$emit('input', valorInterno)"
      />
      <div v-if="erro" class="invalid-feedback">{{ erro }}</div>
    </div>
  </div>
</template>

<script>
export default {
  // Suporte a v-model
  model: {
    prop: 'value',
    event: 'input',
  },

  props: {
    value: {
      type: String,
      default: '',
    },
    label: String,
    tipo: {
      type: String,
      default: 'text',
    },
    placeholder: String,
    erro: String,
    disabled: {
      type: Boolean,
      default: false,
    },
  },

  data() {
    return {
      valorInterno: this.value,
    };
  },

  watch: {
    value(novoValor) {
      this.valorInterno = novoValor;
    },
  },
};
</script>
```

### Componente com slot

```vue
<template>
  <div class="secao-container">
    <div class="secao-header d-flex align-items-center justify-content-between py-2 border-bottom">
      <h5 class="mb-0">{{ titulo }}</h5>
      <slot name="acoes" />
    </div>
    <div class="secao-conteudo pt-3">
      <slot />
    </div>
  </div>
</template>

<script>
export default {
  props: {
    titulo: {
      type: String,
      required: true,
    },
  },
};
</script>
```

**Uso no pai:**
```vue
<SecaoContainer titulo="Minha Seção">
  <template #acoes>
    <button class="btn btn-sm btn-primary" @click="novo">Novo</button>
  </template>

  <!-- Conteúdo padrão (slot default) -->
  <p>Conteúdo da seção</p>
</SecaoContainer>
```

---

## Padrão de comunicação

### Filho → Pai (`$emit`)

```javascript
// No filho: emitir
this.$emit('salvo', dadosRetorno);
this.$emit('cancelado');
this.$emit('update:valor', novoValor);  // .sync modifier

// No pai: escutar
<ComponenteFilho @salvo="onSalvo" @cancelado="showModal = false" />
```

### Pai → Filho (props)

```javascript
// No filho: declarar prop
props: {
  dataItem: Object,
}

// No pai: passar
<ComponenteFilho :dataItem="meuObjeto" />
```

---

## Regras para Componentes

- **Responsabilidade única** — um componente faz uma coisa bem
- **Props** sempre com `type` declarado; `required: true` para obrigatórias
- **Eventos** documentados nos comentários ou naming óbvio (`hide`, `salvo`, `cancelado`)
- **Estilos** sempre `scoped` para não vazar para outros componentes
- **Não chamar API diretamente** em componentes simples — preferir receber dados via props e emitir eventos para o pai chamar a API
- **Exceção:** componentes complexos (modais com formulário próprio) podem ter suas próprias chamadas API

## Checklist

- [ ] Arquivo criado no caminho correto (`components/` ou `components/{Feature}/`)
- [ ] Props com tipos declarados
- [ ] Props obrigatórias com `required: true`
- [ ] Estilos com `scoped`
- [ ] Comunicação com pai via `$emit` (não manipular props diretamente)
- [ ] Suporte a `v-model` se for componente de input (declarar `model: { prop, event }`)
