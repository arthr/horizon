<script type="text/ecmascript-6">
    import NfeForm from "./nfe-form";
    import NfeResult from "./nfe-result";

    export default {
      components: {
        NfeForm,
        NfeResult,
      },

      data() {
        return {
          user: {},
          nfe: {},
          accessKey: null,
          loadingStatus: true
        };
      },

      watch: {},

      mounted() {
        document.title = "Consulta NFe";
      },

      computed: {},

      methods: {
        nfeResult(payload) {
          this.nfe = payload.nfe;
          this.accessKey = payload.nfe.accessKey;
        },
        loading(payload) {
          this.loadingStatus = payload;
        },
      },
    };
</script>

<template>
  <div>
    <div class="card">
      <div class="card-header d-flex align-items-center justify-content-between">
        <h5>Consulta de Nota Fiscal</h5>
      </div>

      <div class="card-body card-bg-secondary">
        <nfe-form @nfe-result="nfeResult" @loading-status="loading"></nfe-form>
      </div>
    </div>

    <nfe-result v-if="nfe.accessKey && !loadingStatus" :nfe="nfe" :accessKey="accessKey"
      :loading="loadingStatus"></nfe-result>

    <div class="card mt-4" id="disclaimer">
      <div class="card-header d-flex align-items-center justify-content-between">
        <h5>Observações</h5>
      </div>
      <div class="card-body card-bg-secondary">
        <ol>
          <li>
            <strong>Chave de Acesso:</strong> deve ser informado o número de 44
            dígitos presentes no DANFE.
          </li>
        </ol>
      </div>
    </div>
  </div>
</template>
