<template>

  <div id="app">
    
     <CalendarWeek :trackingData=loadData />
  </div>
</template>

<script>
import CalendarWeek from './components/CalendarWeek'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import * as d3 from 'd3'


window.$ = require('jquery')
window.JQuery = require('jquery')

export default {
  name: 'app',
  components: {
    CalendarWeek,
  }, data(){
    return {
      loadData: {},
       
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

.modal{
  position: absolute;
  float: left;
  left: 33%;
  top: 30%;
  display:block;
  width:30%;
  font-size: 12px;
  /*height:50%;*/
}

#addCategory {
  float:left;
  width:150px;
  height:36px;
}
#SaveBTN {
  float:left;
}

::placeholder{
  font-size:12px;
}

.modal-sm {
  margin-left:-130px;
}

.selector {
  width:100px;
}

#chart, .headers {
  color: #ccc4c4; 
}

.propleft {
  float:left;
  margin-left:10px;
}

.btn-secondary, .btn-primary {
  font-size:12px;
  margin:2px;
}

#forward {
  float:right;
  margin-right:20px;
}
#back {
  float:left;
  margin-left:55px;
}


#categoryHeader {
 float:left;
  margin-top:75px;
  font-family: "Lucida Sans Unicode";
  color:#746d6d;
}

#cat {
  width:38%;
  margin-top:130px;
  margin-left:-40px;
}

li {list-style-type: none;}

body{
    overflow: hidden;
}


.swatch{
  height:20px;
  width: 20px;
  position:fixed;
  opacity: 0.5
}

.clicked {
  fill: cadetblue !important;
}

.headers, #cat, .weekDate {
  font-size: 12px;
  
}


.selected {
  fill: #007bff !important;
  fill-opacity: 0.1;
}

.repChooser{
  width:48%
}

.prop {
  float: left;
}

</style>
