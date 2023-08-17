<script type="text/ecmascript-6">
import moment from "moment";

export default {
  props: ["nfe", "accessKey", "loading"],

  data() {
    return {
      fullAddress: null,
      recipientFullAddress: null,
      issuedAt: null,
      monitoredAt: null,
      createdAt: null,
      totalValue: 0,
      buyerPresence: null,
      type: null,
      shippingCompany: null,
    };
  },

  watch: {
    nfe: function (newest) {
      this.nfe = newest;
      this.prepareData();
    },
    accessKey: function (newest) {
      this.accessKey = newest;
    },
  },

  mounted() {
    this.prepareData();
  },

  methods: {
    prepareData() {
      this.fullAddress = this.nfe.payee.street + ", " + this.nfe.payee.number;
      this.recipientFullAddress =
        this.nfe.recipient.street + ", " + this.nfe.recipient.number;
      this.issuedAt = this.formatDate(this.nfe.issuedAt, true);
      this.monitoredAt = this.formatDate(this.nfe.monitoredAt, true);
      this.createdAt = this.formatDate(this.nfe.created_at);
      this.totalValue = this.formatPrice(this.nfe.totalValue);
      this.buyerPresence = this.formatBuyer(
        this.nfe.identification.buyerPresence
      );
      this.type = this.formatType(this.nfe.identification.type);
      this.shippingCompany = this.nfe.hasOwnProperty("shipping_company")
        ? this.nfe.shipping_company
        : null;
      console.log(this.shippingCompany, this.nfe.shipping_company);
    },
    formatDate(value, time = false) {
      let format = time ? "DD/MM/YYYY HH:mm:ss" : "DD/MM/YYYY";
      return moment(value).format(format);
    },
    formatPrice(value) {
      let val = (value / 1).toFixed(2).replace(".", ",");
      return "R$ " + val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
    },
    formatBuyer(value) {
      value = parseInt(value);
      switch (value) {
        case 0:
          return "Não se aplica";
        case 1:
          return "Operação Presencial";
        case 2:
          return "Não Presencial, Internet";
        case 3:
          return "Não Presencial, Teleatendimento";
        case 4:
          return "NFC-e Entrega em Domicílio";
        case 5:
          return "Operação Presencial, Fora do Estabelecimento";
        case 9:
          return "Não Presencial, Outros";
        default:
          break;
      }
    },
    formatType(value) {
      value = parseInt(value);
      switch (value) {
        case 0:
          return "0 - Entrada";
        case 1:
          return "1 - Saída";
        default:
          break;
      }
    },
  },
};
</script>

<template>
  <div class="card mt-4" id="result">
    <div class="card-header d-flex align-items-center justify-content-between">
      <h5>
        Nota Fiscal: <small>{{ accessKey }}</small>
      </h5>
    </div>
    <!-- Card -->
    <div class="card-bg-secondary">
      <!-- Documento | Valor  | Emissão -->
      <div class="d-flex align-items-center justify-content-between pt-2">
        <div class="col-7">
          <div class="form-group">
            <label>Natureza da Operação</label>
            <input type="text" readonly class="form-control" v-model="nfe.identification.natureOperation" />
          </div>
        </div>
        <div class="col-2">
          <div class="form-group">
            <label>Tipo de Operação</label>
            <input type="text" readonly class="form-control" v-model="type" />
          </div>
        </div>
        <div class="col-3">
          <div class="form-group">
            <label>Data de Emissão</label>
            <input type="text" readonly class="form-control" v-model="issuedAt" />
          </div>
        </div>
      </div>
      <!-- Modelo | Série | Documento | Valor -->
      <div class="d-flex align-items-center justify-content-between pt-2">
        <div class="col-1">
          <div class="form-group">
            <label>Modelo</label>
            <input type="text" readonly class="form-control" v-model="nfe.identification.model" />
          </div>
        </div>
        <div class="col-1">
          <div class="form-group">
            <label>Série</label>
            <input type="text" readonly class="form-control" v-model="nfe.identification.documentSeries" />
          </div>
        </div>
        <div class="col-2">
          <div class="form-group">
            <label>Número</label>
            <input type="text" readonly class="form-control" v-model="nfe.identification.number" />
          </div>
        </div>
        <div class="col-5">
          <div class="form-group">
            <label>Presença do Comprador</label>
            <input type="text" readonly class="form-control" v-model="buyerPresence" />
          </div>
        </div>
        <div class="col-3">
          <div class="form-group">
            <label>Valor Total</label>
            <input type="text" readonly class="form-control" v-model="totalValue" />
          </div>
        </div>
      </div>
      <!-- Efetivação | Monitoramento -->
      <div class="
          d-flex
          align-items-center
          justify-content-between
          pt-2
          border-bottom
        ">
        <div class="col-3">
          <div class="form-group">
            <label>Data de Efetivação <small>- BANPAR</small></label>
            <input type="text" readonly class="form-control" v-model="createdAt" />
          </div>
        </div>
        <div class="col-3">
          <div class="form-group">
            <label>Últ. Monitoramento <small>- SERPRO</small></label>
            <input type="text" readonly class="form-control" v-model="monitoredAt" />
          </div>
        </div>
      </div>
      <!-- Emitente | CNPJ -->
      <div class="d-flex align-items-center justify-content-between pt-2">
        <div class="col-8">
          <div class="form-group">
            <label><strong>Emitente</strong></label>
            <input type="text" readonly class="form-control" v-if="nfe.payee.name !== null" v-model="nfe.payee.name" />
          </div>
        </div>
        <div class="col-4">
          <div class="form-group">
            <label>CNPJ</label>
            <input type="text" readonly class="form-control" v-model="nfe.payee.cnpj" />
          </div>
        </div>
      </div>
      <!-- Logradouro | Complemento -->
      <div class="d-flex align-items-center justify-content-between pt-2">
        <div class="col-8">
          <div class="form-group">
            <label>Logradouro</label>
            <input type="text" readonly class="form-control" v-model="fullAddress" />
          </div>
        </div>
        <div class="col-4">
          <div class="form-group">
            <label>Complemento</label>
            <input type="text" readonly class="form-control" v-model="nfe.payee.complement" />
          </div>
        </div>
      </div>
      <!-- Bairro | Cidade | UF | CEP -->
      <div class="
          d-flex
          align-items-center
          justify-content-between
          pt-2
          border-bottom
        ">
        <div class="col-4">
          <div class="form-group">
            <label>Bairro</label>
            <input type="text" readonly class="form-control" v-model="nfe.payee.neighborhood" />
          </div>
        </div>
        <div class="col-4">
          <div class="form-group">
            <label>Cidade</label>
            <input type="text" readonly class="form-control" v-model="nfe.payee.city" />
          </div>
        </div>
        <div class="col-1">
          <div class="form-group">
            <label>UF</label>
            <input type="text" readonly class="form-control" v-model="nfe.payee.federationUnity" />
          </div>
        </div>
        <div class="col-3">
          <div class="form-group">
            <label>CEP</label>
            <input type="text" readonly class="form-control" v-model="nfe.payee.postalCode" />
          </div>
        </div>
      </div>
      <!-- Destinatario | CNPJ -->
      <div class="d-flex align-items-center justify-content-between pt-2">
        <div class="col-8">
          <div class="form-group">
            <label><strong>Destinatário</strong></label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.name" />
          </div>
        </div>
        <div class="col-4">
          <div class="form-group" v-if="nfe.recipient.cnpj">
            <label>CNPJ</label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.cnpj" />
          </div>
          <div class="form-group" v-else-if="nfe.recipient.cpf">
            <label>CPF</label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.cpf" />
          </div>
        </div>
      </div>
      <!-- Logradouro | Complemento -->
      <div class="d-flex align-items-center justify-content-between pt-2">
        <div class="col-8">
          <div class="form-group">
            <label>Logradouro</label>
            <input type="text" readonly class="form-control" v-model="recipientFullAddress" />
          </div>
        </div>
        <div class="col-4">
          <div class="form-group">
            <label>Complemento</label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.complement" />
          </div>
        </div>
      </div>
      <!-- Bairro | Cidade | UF | CEP -->
      <div class="
          d-flex
          align-items-center
          justify-content-between
          pt-2
          border-bottom
        ">
        <div class="col-4">
          <div class="form-group">
            <label>Bairro</label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.neighborhood" />
          </div>
        </div>
        <div class="col-4">
          <div class="form-group">
            <label>Cidade</label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.city" />
          </div>
        </div>
        <div class="col-1">
          <div class="form-group">
            <label>UF</label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.federationUnity" />
          </div>
        </div>
        <div class="col-3">
          <div class="form-group">
            <label>CEP</label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.postalCode" />
          </div>
        </div>
      </div>
      <!-- Destinatario | Dados de Contato -->
      <div class="d-flex align-items-center justify-content-between pt-2">
        <div class="col-8">
          <div class="form-group">
            <label>E-mail</label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.email" />
          </div>
        </div>
        <div class="col-4">
          <div class="form-group">
            <label>Telefone</label>
            <input type="text" readonly class="form-control" v-model="nfe.recipient.phone" />
          </div>
        </div>
      </div>

      <!-- Transportadora | Nome -->
      <div class="d-flex align-items-center justify-content-between pt-2" v-if="shippingCompany !== null">
        <div class="col-8">
          <div class="form-group">
            <label><strong>Transportadora</strong></label>
            <input type="text" readonly class="form-control" v-model="shippingCompany.name" />
          </div>
        </div>
        <!-- Transportadora | Documentos -->
        <div class="col-4">
          <div class="form-group" v-if="shippingCompany.cnpj">
            <label>CNPJ</label>
            <input type="text" readonly class="form-control" v-model="shippingCompany.cnpj" />
          </div>
          <div class="form-group" v-else-if="shippingCompany.cpf">
            <label>CPF</label>
            <input type="text" readonly class="form-control" v-model="shippingCompany.cpf" />
          </div>
          <div class="form-group" v-else>
            <label>Documento</label>
            <input type="text" readonly class="form-control" value="Não Informado" />
          </div>
        </div>
      </div>
      <!-- Transportadora | Endereço -->
      <div class="d-flex align-items-center justify-content-between pt-2" v-if="shippingCompany !== null">
        <div class="col-7">
          <div class="form-group">
            <label>Endereço Completo</label>
            <input type="text" readonly class="form-control" v-model="shippingCompany.fulladdress" />
          </div>
        </div>
        <div class="col-4">
          <div class="form-group">
            <label>Cidade</label>
            <input type="text" readonly class="form-control" v-model="shippingCompany.city" />
          </div>
        </div>
        <div class="col-1">
          <div class="form-group">
            <label>UF</label>
            <input type="text" readonly class="form-control" v-model="shippingCompany.federationUnity" />
          </div>
        </div>
      </div>

      <!-- Duplicatas -->
      <div class="d-flex mb-0 mt-3">
        <div class="col-12">
          <h6><strong>Duplicatas</strong></h6>
        </div>
      </div>
      <div class="d-flex align-items-center justify-content-between pt-2">
        <table class="table table-hover table-sm mb-0">
          <thead>
            <tr>
              <th>Nro.</th>
              <th>Valor</th>
              <th>Vencimento</th>
            </tr>
          </thead>
          <tbody v-if="nfe.bill_receipts.length">
            <tr v-for="receipt in nfe.bill_receipts" v-bind:key="receipt.id">
              <td>{{ receipt.number }}</td>
              <td class="table-fit">{{ formatPrice(receipt.value) }}</td>
              <td class="table-fit">{{ formatDate(receipt.dueDate) }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Produtos -->
      <div class="d-flex mb-0 mt-3">
        <div class="col-12">
          <h6><strong>Produtos</strong></h6>
        </div>
      </div>
      <div class="
          d-flex
          align-items-center
          justify-content-between
          pt-2
          border-bottom
        ">
        <table class="table table-hover table-sm mb-0">
          <thead>
            <tr>
              <th>Cod.</th>
              <th>Nome</th>
              <th>Quantidade</th>
              <th>Valor Unitário</th>
              <th>Total Bruto</th>
            </tr>
          </thead>
          <tbody v-if="nfe.products.length">
            <tr v-for="product in nfe.products" v-bind:key="product.id">
              <td>{{ product.productCode }}</td>
              <td>{{ product.productDesc }}</td>
              <td>{{ product.comercialQuantity }}</td>
              <td>{{ formatPrice(product.unitSalesValue) }}</td>
              <td>{{ formatPrice(product.grossValue) }}</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Eventos -->
      <div class="d-flex mb-0 mt-3">
        <div class="col-12">
          <h6><strong>Eventos</strong></h6>
        </div>
      </div>
      <div class="d-flex align-items-center justify-content-between pt-2">
        <table class="table table-hover table-sm mb-0">
          <thead>
            <tr>
              <th>Evento</th>
              <th>Data</th>
            </tr>
          </thead>
          <tbody v-if="nfe.events.length">
            <tr v-for="event in nfe.events" v-bind:key="event.id">
              <td>{{ event.event }}</td>
              <td>{{ formatDate(event.eventDate, true) }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <!-- Card End -->
  </div>
</template>
<style>
h5>small {
  font-weight: bold;
}
</style>