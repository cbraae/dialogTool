

<template>
  <div id="app">
    
    <CalendarWeek :trackingData=loadData />

  </div>
</template>

<script>
// import Chart from './components/Chart'
import CalendarWeek from './components/CalendarWeek'
import Modal from './components/Modal'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import * as d3 from 'd3'
// import data from "/data/ptsd_filtered.csv";

window.$ = require('jquery')
window.JQuery = require('jquery')

export default {
  name: 'app',
  components: {
    CalendarWeek,
    Modal
  }, data(){
    return {
       modalOpen: false,
      loadData: {}
    };
    
  }, mounted() {
    this.fetchData(); 

   
  },methods: {
    async fetchData() {
       let data = await d3.csv("/data/ptsd_filtered.csv");
       this.loadData = data;  
    },
     openModal() {
            this.modalOpen = !this.modalOpen;
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

.modal {
  position: absolute;
  float: left;
  left: 33%;
  top: 30%;
  width: 100% 
}

body{
    overflow: hidden;
}

.clicked {
  fill: cadetblue !important;
}

.headers {
  font-size: 12px;

}

</style>
