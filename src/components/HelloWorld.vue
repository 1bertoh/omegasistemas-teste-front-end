/* eslint-disable no-return-assign */
<template>
  <v-container>
    <v-row class="text-center">
      <v-col cols="12">
        <v-img
          :src="require('../assets/logo.svg')"
          class="my-3"
          contain
          height="200"
        />
      </v-col>
    </v-row>

    <v-row>
      <v-col
      class="col-12 col-sm-6">
        <v-select
          :items="states"
          v-model="stateValueObj"
          label="label"
          item-text="nome"
          return-object
        >
        </v-select>
      </v-col>

      <v-col
      class="col-12 col-sm-6">
        <v-select
          :items="cities"
          v-model="cityValueObj"
          label="label"
          item-text="nome"
          return-object
        >
        </v-select>
      </v-col>
      <img :src="mapUrl" alt="" :srcset="map.data">
      {{map.data}}
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
    cities: '',
    cityValueObj: '',
    citiesName: [],
    covid: '',
    map: '',
    mapUrl: '',
  }),
  mounted() {
    this.getStates();
    this.getBrazilCases();
    this.getMap();
  },

  watch: {
    stateValueObj() {
      this.getCity();
      this.getMapUrl(this.stateValueObj.id)
    },
    cityValueObj() {
      this.getCovid();
    },
  },
  methods: {
    getStates() {
      const url = 'https://servicodados.ibge.gov.br/api/v1/localidades/estados?orderBy=nome';
      axios
        .get(url)
      // eslint-disable-next-line no-return-assign
        .then((response) => {
          this.states = response.data;
        });
    },

    getBrazilCases() {
      const url = 'https://covid19-brazil-api.now.sh/api/report/v1/brazil';
      axios
        .get(url)
      // eslint-disable-next-line no-return-assign
        .then((response) => {
          this.covid = response.data.data;
        });
    },

    getCity() {
      const url = `https://servicodados.ibge.gov.br/api/v1/localidades/estados/${this.stateValueObj.id}/municipios`;
      axios
        .get(url)
        .then((response) => {
          this.cities = response.data;
        });
    },

    getCovid() {
      const url = `https://covid19-brazil-api.now.sh/api/report/v1/brazil/uf/${this.stateValueObj.sigla}`;
      axios
        .get(url)
        .then((response) => {
          this.covid = response.data;
        });
    },
    getMap() {
      const url = 'https://servicodados.ibge.gov.br/api/v3/malhas/estados/33';
      axios
        .get(url)
        .then((response) => {
          this.map = response;
        });
    },
    getMapUrl(state) {
      const url = `https://servicodados.ibge.gov.br/api/v3/malhas/estados/${state}`;
      this.mapUrl = url;
    }
  },
};
</script>
