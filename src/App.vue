<template>
  <div id="app">
    <div class="row">
      <div class="col"><CalendarWeek :trackingData=loadData /></div>
      <div class="col"><CategoryPicker v-model="items" :items="items"></CategoryPicker></div>
    </div>
  </div>
</template>

<script>
import CalendarWeek from './components/CalendarWeek'
import CategoryPicker from './components/CategoryPicker'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import * as d3 from 'd3'

window.$ = require('jquery')
window.JQuery = require('jquery')

export default {
  name: 'app',
  components: {
    CalendarWeek,
    CategoryPicker
  }, data(){
    return {
      loadData: {},
        items: [
            { text: 'Venner', hex:"#F4D03F"},
            { text: 'Familie', hex:"#229954"},
            { text: 'Arbejde', hex:"#9B59B6"}
        ]
    };
    
  }, mounted() {
    
    this.fetchData(); 

   
  },methods: {
    async fetchData() {
       let data = await d3.csv("/data/ptsd_filtered.csv");
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

.modal {
  position: absolute;
  float: left;
  left: 33%;
  top: 30%;
  display:block;
  width:40%;
  height:50%;
}


#cat {
  width:38%;
}

li {list-style-type: none;}

body{
    overflow: hidden;
}

.swatch{
  height:20px;
  width: 20px;
  position:fixed;
}

.clicked {
  fill: cadetblue !important;
}

.headers, #cat {
  font-size: 12px;
  
}

</style>
