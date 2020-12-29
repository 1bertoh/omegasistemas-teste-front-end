/* eslint-disable no-return-assign */
<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-img
          src="./static/corona-icon.png"
          class="my-"
          contain
          height="200"
        />
      </v-col>
    </v-row>

    <v-row>
      <v-col
      class="col-12 col-sm-6">
        <v-select
          dark
          :items="regioes"
          v-model="regiaoValueObj"
          label="Região"
          item-text="nome"
          return-object
          @input="alert('oi')"
        >
        </v-select>
      </v-col>

      <v-col
      class="col-12 col-sm-6">
        <v-select
        dark
          class=""
          :items="states"
          v-model="stateValueObj"
          label="Estado"
          item-text="nome"
          return-object
        >
        </v-select>
      </v-col>
    </v-row>

    <v-row class="d-flex justify-center">
      <v-card id="card-img"
      class="teal lighten-4 mb-3">
        <v-card-title dark class="teal darken-4 white--text text-h3"
        primary-title>
        
          {{title}}
        </v-card-title>

        <div id="cont">
          <v-img :src="mapUrl" id="map"
          class="" alt="" srcset="">
            <v-col cols="12" id="text3" class="d-flex justify-center"
              v-if="covidState === '' && RegiaoCases.length === 0">
              <p class="hidden-xs-only display-2 font-weight-black red--text">Confirmados {{covid.confirmed}}</p>
              <p class="hidden-sm-and-up text-h5 font-weight-black red--text " id="text4">Confirmados {{covid.confirmed}}</p>
            </v-col>
          </v-img>

      <v-card-text class="teal lighten-1 mt-3" style="height:12px">
        <v-card class="teal darken-4 pb-2" style="height:15px">
          
        </v-card>
        <v-row>
          <v-expand-x-transition>
          <v-col id="text1"
           v-show="expand"
          height="100"
          width="100"
          class="mx-auto ">
            <p class="title teal darken-4 white--text pl-4">Casos</p>
            <p class="title red--text font-weight-black pl-4" v-if="covidState ==='' && sumOfCasesByRegiao === 0">{{covid.cases}}</p>
            <p class="title red--text font-weight-black pl-4" v-if="sumOfCasesByRegiao === 0">{{covidState.cases}}</p>
            <p class="title red--text font-weight-black pl-4" v-if="sumOfCasesByRegiao != 0"> {{sumOfCasesByRegiao}}</p>
          </v-col>
          </v-expand-x-transition>

          <v-expand-x-transition>
          <v-col id="text2"
          v-show="expand"
          height="100"
          width="100"
          class="mx-auto ">
            <p class="title teal darken-4 white--text pl-4" >Mortes</p>
            <p class="title red--text font-weight-black pl-4" v-if="covidState === '' && sumOfCasesByRegiao === 0">{{covid.deaths}}</p>
            <p class="title red--text font-weight-black pl-4" v-if="sumOfDeathsByRegiao === 0">{{covidState.deaths }}</p>
            <p class="title red--text font-weight-black pl-4" v-if="sumOfDeathsByRegiao != 0">{{sumOfDeathsByRegiao}} </p>
          </v-col>
          </v-expand-x-transition>
          </v-row>
        </v-card-text>
        </div>
        
        
      </v-card>
    </v-row>
    </v-col>
  </v-container>
</template>

<script>
import axios from 'axios';

export default {
  name: 'HelloWorld',

  data: () => ({
    states: '',
    stateValueObj: '',
    statesName: [],
    regioes: '',
    regiaoValueObj: '',
    statesOfRegiao: '',
    covid: '',
    covidState: '',
    map: '',
    mapUrl: '',
    RegiaoCases: [],
    RegiaoDeaths: [],
    sumOfCasesByRegiao: 0,
    sumOfDeathsByRegiao: 0,
    title: 'Brasil',
    expand: false,
  }),
  mounted() {
    this.getStates();
    this.getBrazilCases();
    this.getRegioes();
    this.mapUrl = 'https://servicodados.ibge.gov.br/api/v3/malhas/paises/BR?intrarregiao=UF'
  },

  watch: {
    stateValueObj() {
      this.getMapUrl(this.stateValueObj.id);
      this.getCovid(this.stateValueObj.sigla);
      this.getMapInfo(this.stateValueObj.id);
    },
    regiaoValueObj() {
      this.getStatesofRegiao(this.regiaoValueObj.id);
      this.sumCasesbyRegiao(this.statesOfRegiao.length);
    },
    RegiaoCases() {
      this.sumCases();
    }
  },
  methods: {
    // gets all the 27 brazilian states by order
    getStates() {
      const url = 'https://servicodados.ibge.gov.br/api/v1/localidades/estados?orderBy=nome';
      axios
        .get(url)
        .then((response) => {
          this.states = response.data;
        });
    },

    // gets all brazilian cases of covid
    getBrazilCases() {
      const url = 'https://covid19-brazil-api.now.sh/api/report/v1/brazil';
      axios
        .get(url)
        .then((response) => {
          this.covid = response.data.data;
        });
        this.expand = true
    },
    // gets the brazilian states cases selected by select
    getCovid(sigla) {
      this.expand = false;
      const url = `https://covid19-brazil-api.now.sh/api/report/v1/brazil/uf/${sigla}`;
      axios
        .get(url)
        .then((response) => {
          this.covidState = response.data;
        }).finally( setTimeout(() => this.expand = true, 500));
        this.sumOfCasesByRegiao = 0;
        this.sumOfDeathsByRegiao = 0;
    },

    // gets the selected state map info from IBGE
    getMapInfo(id) {
      const url = `https://servicodados.ibge.gov.br/api/v3/malhas/estados/${id}`;
      axios
        .get(url)
        .then((response) => {
          this.map = response;
        });
        this.title = this.stateValueObj.nome;
    },

    // generates the url map that will be rendered by img
    getMapUrl(state) {
      const url = `https://servicodados.ibge.gov.br/api/v2/malhas/${state}?resolucao=5`;
      this.mapUrl = url;
    },

    // gets all the 5 regioes of Brazil From IBGE
    getRegioes() {
      const url = 'https://servicodados.ibge.gov.br/api/v1/localidades/regioes?orderBy=nome';
      axios
        .get(url)
        .then((response) => {
          this.regioes = response.data;
        });
    },

    // gets all the states from the regiao selected
    getStatesofRegiao(Id) {
      this.expand = false;
      const url = `https://servicodados.ibge.gov.br/api/v1/localidades/regioes/${Id}/estados`;
      axios
        .get(url)
        .then((response) => {
          this.RegiaoCases = [];
          this.RegiaoDeaths = [];
          this.statesOfRegiao = response.data;
          for (let i = 0; i < this.statesOfRegiao.length; i++) {
            let sum = [];
            axios
              .get(`https://covid19-brazil-api.now.sh/api/report/v1/brazil/uf/${this.statesOfRegiao[i].sigla}`)
              .then((r) => {
                const sumCase = r.data.cases;
                const sumDeaths = r.data.deaths;
                this.RegiaoCases.push(sumCase);
                this.RegiaoDeaths.push(sumDeaths);
              });
          }
        }).finally( setTimeout(() => this.expand = true, 500));;
      // this.stateValueObj = '';
      const urlMap = `http://servicodados.ibge.gov.br/api/v3/malhas/regioes/${this.regiaoValueObj.id}?intrarregiao=UF`;
      this.mapUrl = urlMap;
      this.title = this.regiaoValueObj.nome;
    },
    sumCases() {
      let totalCases = 0;
      let totalDeaths = 0;
      for (let i = 0; i < this.RegiaoCases.length; i++) {
        totalCases += this.RegiaoCases[i];
        totalDeaths += this.RegiaoDeaths[i];
      }
      this.sumOfCasesByRegiao = totalCases;
      this.sumOfDeathsByRegiao = totalDeaths;
    },
    
  },
};
</script>

<style lang="">
.v-select__slot {
  background-color:#017371;
}
#map {
  position: relative;
  display: inline-block;
  
}
#card-img {
  position: relative;
  display: inline-block;
  width: 80%;
}
#text1 {
  position: absolute;
  top: 20%;
  width: 13em;
  right: 0%;
}
#text2 {
  position: absolute;
  top: 40%;
  width: 13em;
  right: 0%;
}
#text3 {
  position: absolute;
  top: 35%;
  width: 13em;
  right: 35%;
}
#text4 {
  position: absolute;
  top: 0%;
  width: 10em;
  right:-20%;
}
.title {
  border-radius: 15px 15px 0 0px ;
}
</style>
