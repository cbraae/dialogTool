<template>

  <div id="app">
     <nav class="navbar navbar-dark bg-dark" id="header">
      <button @click="toggleView" class="btn btn-secondary headerbtn"> Skift visning </button>  
      </nav>

     <CalendarWeek v-if="showCalendar" :trackingData=loadData />
     <CalendarMonth v-if="!showCalendar" :trackingData=loadData />
     
  </div>
</template>

<script>
import CalendarWeek from './components/CalendarWeek'
import CalendarMonth from './components/CalendarMonth'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import * as d3 from 'd3'

window.$ = require('jquery')
window.JQuery = require('jquery')

export default {
  name: 'app',
  components: {
    CalendarMonth, 
    CalendarWeek
  }, data(){
    return {
      loadData: {},
      showCalendar: true,
    }; 
  }, mounted() {
    this.fetchData(); 
  },methods: {
   
    async fetchData() {
       let data = await d3.csv("/data/ptsd_2w.csv");
       var cleanedData = data.map(
          item => item[data.columns[0]].split("Z")[0]
        );
        cleanedData.removeIf(function(item, idx) {
          return item == ";";
        });
        this.loadData = cleanedData;
    }, toggleView() { 
        this.showCalendar = !this.showCalendar;
    }
  }
}

</script>

<style>

/* CSS3 */

/* The whole thing */
.custom-menu {
    display: none;
    z-index: 1000;
    position: absolute;
    overflow: hidden;
    border: 1px solid #CCC;
    white-space: nowrap;
    font-family: sans-serif;
    background: #FFF;
    color: #333;
    border-radius: 5px;
    padding: 0;
}

/* Each of the items in the list */
.custom-menu li {
    padding: 8px 12px;
    cursor: pointer;
    list-style-type: none;
    transition: all .3s ease;
    user-select: none;
}

.custom-menu li:hover {
    background-color: #DEF;
}

.drawing-area-class {
  width: 100%;
}

.calendar {
    position: absolute;
    left: 0px;
    top: 0px;
}

.smallMutipless{
  margin-top:548px;
  position: absolute;

}


#header {
  position: relative;
  margin: 0 auto 0;
  padding-top: 0px;
  /*height: 60px;*/
  /*margin-bottom: 80px;*/
}

body{
    margin:0;
    overflow-y:scroll;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}



.buttonGroup {
  float:right;
    margin-top:-180px;
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

.headerbtn{
  float:right;
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

#chart, .headers, #month {
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

#forward, .headerbtn {
  float:right;
  margin-right:20px;
}
#back {
  float:left;
  margin-left:55px;
}

#chart {
  position:relative;
  width:1000px;
  left: 0px;
}

#categorySection{
  float:right;
  width:400px;
}

.drawingbuttons{
  float:left;
}

#save{
  margin-left:50px;
}

.weekAxis .tick line, .yaxis .tick line {
        display:none;
    }

.weekAxis line {
  display: none;
}

#categoryHeader {
  float:left;
  margin-top:30px;
  font-family: "Lucida Sans Unicode";
  color:#746d6d;
}

#cat {
  width:50%;
  margin-top:70px;
  margin-left:-40px;
}

li {list-style-type: none;}



.swatch{
  height:20px;
  width: 20px;
  position:absolute;

}

.clicked {
  fill: cadetblue !important;
}

.headers, #cat, .weekDate {
  font-size: 12px;
  
}

.weekbrush .selection{
  fill: crimson !important;
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

#selector {

  fill: none;
}

#categoryOverview{
    margin-top:-70px;
    margin-left:12px;
}

.brush {
  min-width: 100px;
}

</style>
