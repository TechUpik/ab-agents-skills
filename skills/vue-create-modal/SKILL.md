---
name: vue-create-modal
description: This skill should be used when the user wants to create a new modal component in the rep-upik-web Vue.js repository. Activate when the user mentions "criar modal", "novo modal vue", "modal de confirmação", "modal de formulário", "modal de detalhe", "modal vue", or describes building any modal/dialog for the Vue frontend.
version: 1.0.0
---

# Criar um novo Modal no rep-upik-web

## Localização

Os modais ficam em `src/components/{Feature}/Modal{Descricao}.vue`.

Exemplos reais do projeto:
- `src/components/Consultorias/ModalNotaFiscal.vue`
- `src/components/Consultorias/ModalRegistrarAgendamento.vue`
- `src/components/Agenda/ModalDetalhes.vue`

---

## Estrutura padrão

```vue
<template>
  <b-modal
    :visible="showModal"
    title="{Título do Modal}"
    size="lg"
    :hide-footer="true"
    @hidden="hideModal"
  >
    <!-- Loading -->
    <div v-if="loading" class="text-center py-4">
      <Loading width="60" />
      <p class="mt-2">Carregando...</p>
    </div>

    <!-- Conteúdo -->
    <div v-if="!loading">
      <!-- Conteúdo do modal aqui -->
    </div>

    <!-- Footer com botões (se não usar :hide-footer="true") -->
    <template #modal-footer>
      <b-button variant="secondary" @click="hideModal">Cancelar</b-button>
      <b-button
        variant="primary"
        :disabled="loadingSalvar"
        @click="onClickSalvar"
      >
        <span v-if="loadingSalvar">
          <span class="spinner-border spinner-border-sm" role="status"></span>
          Salvando...
        </span>
        <span v-else>Salvar</span>
      </b-button>
    </template>
  </b-modal>
</template>

<script>
import axios from 'axios';
import config from '../../config';
import captureError from '../../captureError';
import Loading from '../Loading';

export default {
  components: {
    Loading,
  },

  props: {
    showModal: {
      type: Boolean,
      required: true,
    },
    // Dados passados pelo pai
    dataItem: {
      type: Object,
      default: null,
    },
  },

  data() {
    return {
      loading: false,
      loadingSalvar: false,
      dados: null,
      // Estado do formulário
      form: {
        campo1: '',
        campo2: null,
      },
    };
  },

  watch: {
    // Recarregar dados quando o modal abre com novo item
    showModal(isVisible) {
      if (isVisible && this.dataItem) {
        this.carregarDados();
      }
    },
  },

  methods: {
    async carregarDados() {
      this.loading = true;
      try {
        const response = await axios.get(
          `${config.API_URL}/{Entidade}/${this.dataItem.id}`
        );
        this.dados = response.data;
        // Preencher formulário se necessário
        this.form.campo1 = response.data.campo1;
        this.form.campo2 = response.data.campo2;
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

    async onClickSalvar() {
      // Validação básica
      if (!this.form.campo1) {
        this.$swal({ icon: 'warning', title: 'Atenção', text: 'Campo1 é obrigatório.' });
        return;
      }

      this.loadingSalvar = true;
      try {
        await axios.put(`${config.API_URL}/{Entidade}/${this.dataItem.id}`, {
          campo1: this.form.campo1,
          campo2: this.form.campo2,
        });

        this.$swal({ icon: 'success', title: 'Salvo!', text: 'Dados atualizados com sucesso.' });
        this.$emit('atualizado');  // avisar o pai para recarregar lista
        this.hideModal();
      } catch (e) {
        captureError(e);
        this.$swal({
          icon: 'error',
          title: 'Oops...',
          text: 'Não foi possível salvar as alterações.',
        });
      } finally {
        this.loadingSalvar = false;
      }
    },

    hideModal() {
      this.$emit('hide');
      this.resetForm();
    },

    resetForm() {
      this.form = {
        campo1: '',
        campo2: null,
      };
      this.dados = null;
    },
  },
};
</script>

<style scoped lang="less">
/* Estilos do modal */
</style>
```

---

## Tipos de Modal

### Modal de confirmação (sem formulário)

```vue
<template>
  <b-modal
    :visible="showModal"
    title="Confirmar Ação"
    @hidden="$emit('hide')"
  >
    <p>Tem certeza que deseja {{ descricaoAcao }}?</p>
    <p class="text-muted">Esta ação não pode ser desfeita.</p>

    <template #modal-footer>
      <b-button variant="secondary" @click="$emit('hide')">Cancelar</b-button>
      <b-button
        variant="danger"
        :disabled="loading"
        @click="confirmar"
      >
        Confirmar
      </b-button>
    </template>
  </b-modal>
</template>

<script>
import axios from 'axios';
import config from '../../config';
import captureError from '../../captureError';

export default {
  props: {
    showModal: { type: Boolean, required: true },
    dataItem: { type: Object, default: null },
    descricaoAcao: { type: String, default: 'realizar esta ação' },
  },

  data() {
    return { loading: false };
  },

  methods: {
    async confirmar() {
      this.loading = true;
      try {
        await axios.post(`${config.API_URL}/{Entidade}/${this.dataItem.id}/{acao}`);
        this.$emit('confirmado');
        this.$emit('hide');
      } catch (e) {
        captureError(e);
        this.$swal({ icon: 'error', title: 'Erro', text: 'Não foi possível realizar a ação.' });
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>
```

### Modal de detalhe (somente leitura)

```vue
<template>
  <b-modal
    :visible="showModal"
    title="Detalhe"
    :hide-footer="true"
    size="xl"
    @hidden="$emit('hide')"
  >
    <div v-if="loading" class="text-center py-4">
      <Loading width="60" />
    </div>

    <div v-if="!loading && dados">
      <div class="row">
        <div class="col-md-6">
          <strong>Nome:</strong> {{ dados.nome }}
        </div>
        <div class="col-md-6">
          <strong>Status:</strong> {{ dados.statusDescricao }}
        </div>
      </div>
      <!-- Mais campos de detalhe -->
    </div>
  </b-modal>
</template>

<script>
export default {
  props: {
    showModal: { type: Boolean, required: true },
    itemId: { type: String, default: null },
  },
  // ...
};
</script>
```

---

## Uso no Pai (View ou outro componente)

```vue
<!-- Na view pai: importar, registrar e usar -->
<template>
  <div>
    <button @click="abrirModal(item)">Ver detalhe</button>

    <Modal{Descricao}
      :showModal="showModal{Descricao}"
      :dataItem="itemSelecionado"
      @hide="showModal{Descricao} = false"
      @atualizado="carregarDados"
    />
  </div>
</template>

<script>
import Modal{Descricao} from '../components/{Feature}/Modal{Descricao}.vue';

export default {
  components: { Modal{Descricao} },

  data() {
    return {
      showModal{Descricao}: false,
      itemSelecionado: null,
    };
  },

  methods: {
    abrirModal(item) {
      this.itemSelecionado = item;
      this.showModal{Descricao} = true;
    },
  },
};
</script>
```

---

## Tamanhos do Modal (Bootstrap Vue)

| `size` | Largura |
|--------|---------|
| `sm` | 300px |
| (padrão) | 500px |
| `lg` | 800px |
| `xl` | 1140px |

---

## Regras para Modais

- Nome sempre `Modal{Descricao}.vue` — ex: `ModalRegistrarAgendamento.vue`
- Sempre `@hidden="hideModal"` (não `@hide`) para capturar fechamento por backdrop/ESC
- Sempre emitir `$emit('hide')` quando fechar
- Emitir `$emit('atualizado')` ou `$emit('criado')` para o pai recarregar a lista
- Loading separado para carregamento (`loading`) e para ação de salvar (`loadingSalvar`)
- Usar `watch: { showModal }` para carregar dados quando o modal abrir
- Chamar `resetForm()` ao fechar para limpar o estado
- Caminhos de import com `../../` pois modais ficam em subpastas de `components/`

## Checklist

- [ ] Arquivo criado em `src/components/{Feature}/Modal{Descricao}.vue`
- [ ] Props `showModal` (Boolean, required) declarada
- [ ] `@hidden` no `b-modal` conectado a `hideModal()` method
- [ ] `$emit('hide')` chamado no método de fechar
- [ ] `$emit('atualizado')` emitido após operação bem-sucedida
- [ ] Loading state para ação de salvar (botão desabilitado + spinner)
- [ ] Erros tratados com `captureError` + `$swal`
- [ ] `resetForm()` chamado ao fechar o modal
- [ ] Modal registrado e usado corretamente no componente pai
