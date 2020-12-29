<template>
  <v-app>
     <div>
    <v-toolbar
      dark
      prominent
      src="./static/coronavirus-4.jpg"
    >

      <v-toolbar-title >Covid19 Informa</v-toolbar-title>

      <v-spacer></v-spacer>
    </v-toolbar>
  </div>

    <v-main class="teal darken-3">
      <HelloWorld/>
    </v-main>

    <v-footer
    class=""
    dark
    padless
  >
    <v-card
      class="flex"
      flat
      tile
    >
      <v-card-title class="teal">
        <strong class="subheading">Use Mask!</strong>

        <v-spacer></v-spacer>
      </v-card-title>

      <v-card-text class="py-2 white--text text-center">
        {{ new Date().getFullYear() }} — <strong>Corona-Info</strong>
      </v-card-text>
    </v-card>
  </v-footer>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld';

export default {
  name: 'App',

  components: {
    HelloWorld,
  },

  data: () => ({
    //
  }),
};
</script>
