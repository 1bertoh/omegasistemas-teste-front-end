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
      <v-card
      class="teal lighten-4">
        <v-card-title dark class="teal darken-4 white--text text-h3"
        primary-title>
        
          {{title}}
        </v-card-title>

        <v-img :src="mapUrl" id="map" max-width="500px"
      class="" alt="" srcset=""></v-img>

      <v-card-text>
        <v-row>
          <v-col cols="12" class="d-flex justify-center"
          v-if="covidState === '' && RegiaoCases.length === 0">
            <p class="sm-display-1 display-2 font-weight-black red--text">Confirmados {{covid.confirmed}}</p>
          </v-col>

          <v-col>
            <p class="title pink">Casos</p>
            <p v-if="covidState ==='' && sumOfCasesByRegiao === 0">{{covid.cases}}</p>
            <p v-if="sumOfCasesByRegiao === 0">{{covidState.cases}}</p>
            <p v-if="sumOfCasesByRegiao != 0"> {{sumOfCasesByRegiao}}</p>
          </v-col>

          <v-col>
            <p class="title">Mortes</p>
            <p v-if="covidState === '' && sumOfCasesByRegiao === 0">{{covid.deaths}}</p>
            <p v-if="sumOfDeathsByRegiao === 0">{{covidState.deaths }}</p>
            <p v-if="sumOfDeathsByRegiao != 0">{{sumOfDeathsByRegiao}} </p>
          </v-col>
        </v-row>
      </v-card-text>
      </v-card>
    </v-row>

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
      this.getMapInfo(this.stateValueObj.id)
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
    },
    // gets the brazilian states cases selected by select
    getCovid(sigla) {
      const url = `https://covid19-brazil-api.now.sh/api/report/v1/brazil/uf/${sigla}`;
      axios
        .get(url)
        .then((response) => {
          this.covidState = response.data;
        });
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
        });
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
}
    
  },
};
</script>

<style lang="">
.v-select__slot {
  background-color:#017371;
}
</style>
