<template>
  <q-page>
    <br>
    <div class="row">
      <div class="col-12">
        <q-input filled v-model="pesquisa" label="Digite o nome do produto"
                 @keyup.enter="getConsultaProdutoSefaz(pesquisa)"/>
        <br>
      </div>
    </div>
    <div class="flex flex-center">
      <q-btn color="primary" label="Pesquisar" @click="getConsultaProdutoSefaz(pesquisa)"/>
    </div>
    <br>
    <div class="row flex flex-center">
      <q-table
        grid
        card-class="bg-primary text-white"
        :data="dadosConsultaProdutoSefaz"
        :columns="columns"
        row-key="name"
        :loading="loading"
        no-data-label="Pesquise um produto"
        loading-label="Buscando..."
        rows-per-page-label="Qtd. por página"
        :rows-per-page-options="[10, 20, 30, 40, 0]"
        hide-header
      >
        <template v-slot:no-data="{ icon, message }">
          <div class="full-width row flex-center text-primary q-gutter-sm">
            <q-icon size="2em" name="sentiment_dissatisfied"/>
            <span>
              {{ message }}
          </span>
          </div>
        </template>
      </q-table>
    </div>
    <div>
      <q-dialog v-model="alert" transition-show="flip-down" transition-hide="flip-up">
        <q-card class="bg-primary text-white">
          <q-bar>
            <q-toolbar-title><span class="text-weight-bold">Busca</span> Preço</q-toolbar-title>
            <q-space/>
            <q-btn dense flat icon="close" v-close-popup>
              <q-tooltip content-class="bg-white text-primary">Fechar</q-tooltip>
            </q-btn>
          </q-bar>
          <q-card-section/>
          <q-card-section class="q-pt-none">
            Preencha o campo de pesquisa para buscar por seu produto.
          </q-card-section>
          <q-card-actions align="right">
            <q-btn flat label="OK" color="white" v-close-popup/>
          </q-card-actions>
        </q-card>
      </q-dialog>

    </div>
    <!--    <div>-->
    <!--      GPS position: <strong>{{ position }}</strong>-->
    <!--    </div>-->
  </q-page>
</template>

<script>
import { Plugins } from '@capacitor/core'

const { Geolocation } = Plugins

export default {
  name: 'PageIndex',

  data () {
    return {
      columns: [
        {
          name: 'nomFantasia',
          label: 'Nome Fantasia',
          align: 'left',
          field: 'nomFantasia',
          sortable: true
        },
        {
          name: 'nomRazaoSocial',
          label: 'Razão Social',
          align: 'left',
          field: 'nomRazaoSocial',
          sortable: true
        },
        {
          name: 'dscProduto',
          label: 'Descrição do Produto',
          align: 'left',
          field: 'dscProduto',
          sortable: true
        },
        {
          name: 'valUltimaVenda',
          label: 'Valor Útilma Venda',
          align: 'left',
          field: row => {
            return 'R$ ' + row.valUltimaVenda
          },
          sortable: true
        },
        {
          name: 'numTelefone',
          label: 'Telefone',
          align: 'left',
          field: 'numTelefone',
          sortable: true
        },
        {
          name: 'local',
          label: 'Local',
          align: 'left',
          field: row => {
            return row.nomLogradouro + ', ' + row.nomBairro + '/' + row.nomMunicipio
          },
          sortable: true
        }

      ],
      pesquisa: '',
      loading: false,
      dadosConsultaProdutoSefaz: [],
      position: [],
      alert: false
    }
  },
  mounted () {
    this.getCurrentPosition()
    this.geoId = Geolocation.watchPosition({}, (position, err) => {
      this.position = position
    })
  },
  beforeDestroy () {
    Geolocation.clearWatch(this.geoId)
  },
  methods: {
    getCurrentPosition () {
      Geolocation.getCurrentPosition().then(position => {
        this.position = position
      })
    },
    getConsultaProdutoSefaz (descricao) {
      let lat = 0
      let long = 0
      this.dadosConsultaProdutoSefaz = []

      if (!descricao) {
        this.alert = true
        return
      }

      if (this.position) {
        lat = this.position.coords.latitude
        long = this.position.coords.longitude
        // console.log('entrou')
      } else {
        lat = -9.6432331
        long = -35.7190686
      }

      this.loading = true
      const header = {
        headers: {
          AppToken: '84a57ce11ac1cf23adc5bd70b822f9db50dca9dd'
        }
      }
      const params = {
        descricao: descricao,
        codGetin: '7891149103102',
        dias: 1,
        latitude: lat,
        longitude: long,
        raio: 10
      }
      this.$axios.post('http://api.sefaz.al.gov.br/sfz_nfce_api/api/public/consultarPrecosPorDescricao', params, header)
        .then((response) => {
          // console.log(response.data)
          this.dadosConsultaProdutoSefaz = response.data
          this.loading = false
        })
        .catch(() => {
          this.loading = false
        })
    }
  }
}
</script>
