<script type="text/ecmascript-6">
export default {
  props: [],
  data() {
    return {
      data: {},
      fields: {},
      errors: {},
      loading: false,
    };
  },
  mounted() { },
  methods: {
    submit() {
      this.loading = true;
      this.$emit('loading-status', this.loading);
      this.errors = {};
      this.$http
        .post("/api/query/exist", {
          chave: this.fields.chave,
        })
        .then((response) => {
          this.data = response.data;
          this.fields.chave = "";

          this.$emit("nfe-result", response.data);

          this.loading = false;
          this.$emit('loading-status', this.loading);
        })
        .catch((error) => {
          this.errors = error.response.data.errors || {};
          this.loading = false;
          console.log(error);
        });
    },
  },
  watch: {
    handler() {
      this.fields.chave = null;
    },
  },
};
</script>
<template>
  <form @submit.prevent="submit">
    <div class="form-group">
      <label for="chave">
        <strong>Chave de Acesso</strong>
      </label>
      <input type="text" class="form-control" name="chave" id="chave" v-model="fields.chave" />
      <div v-if="errors && errors.chave" class="text-danger">
        {{ errors.chave }}
      </div>
    </div>

    <button type="submit" class="btn btn-primary" :disabled="loading == true">
      Consultar
    </button>

    <div v-if="loading" class="d-flex align-items-center justify-content-center card-bg-secondary p-5 bottom-radius"
      id="loading">
      <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" class="icon spin mr-2 fill-text-color">
        <path
          d="M12 10a2 2 0 0 1-3.41 1.41A2 2 0 0 1 10 8V0a9.97 9.97 0 0 1 10 10h-8zm7.9 1.41A10 10 0 1 1 8.59.1v2.03a8 8 0 1 0 9.29 9.29h2.02zm-4.07 0a6 6 0 1 1-7.25-7.25v2.1a3.99 3.99 0 0 0-1.4 6.57 4 4 0 0 0 6.56-1.42h2.1z">
        </path>
      </svg>

      <span>Buscando Informações...</span>
    </div>
  </form>
</template>

<style>
#loading {
  float: right;
  padding: 0 !important;
}
</style>