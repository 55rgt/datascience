<template>
  <div id="app">
    <link rel="stylesheet" href="//fonts.googleapis.com/css?family=Roboto:400,500,700,
  400italic|Material+Icons">
    <router-view/>
  </div>
</template>

<script>

import Vue from 'vue';
import VueMaterial from 'vue-material';
import 'vue-material/dist/vue-material.min.css';
import { store } from './vuex/store';

Vue.use(VueMaterial);


export default {
  name: 'App',
  store,
  methods: {}
};
</script>

<style>
* {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}


#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
