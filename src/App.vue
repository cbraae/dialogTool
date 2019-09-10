<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <Chart :trackingData=loadData />
  </div>
</template>

<script>
import Chart from './components/Chart'
import * as d3 from 'd3'
// import data from "/data/ptsd_filtered.csv";



export default {
  name: 'app',
  components: {
    Chart
  }, data(){
    return {
      loadData: {}
    };
  }, mounted() {
    this.fetchData(); 

  },methods: {
    async fetchData() {
       let data = await d3.csv("/data/itching_in_nose.csv");
       this.loadData = data;  
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
