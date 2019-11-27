<template>

  <div id="app">
     <nav class="navbar navbar-dark bg-dark" id="header">
      <!--<button @click="toggleView" class="btn btn-secondary headerbtn"> Skift visning </button>  -->
      <FileUpload :loadData.sync="loadData"/>
      </nav>

     <CalendarWeek v-if="showCalendar" :trackingData=loadData />
     <CalendarMonth v-if="!showCalendar" :trackingData=loadData />
     
  </div>
</template>

<script>
import CalendarWeek from './components/CalendarWeek'
import CalendarMonth from './components/CalendarMonth'
import FileUpload from './components/FileUpload'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import * as d3 from 'd3'

window.$ = require('jquery')
window.JQuery = require('jquery')

export default {
  name: 'app',
  components: {
    CalendarMonth, 
    CalendarWeek,
    FileUpload
  }, data(){
    return {
      loadData: {},
      showCalendar: true,
    }; 
  }, mounted() {
    this.fetchData(); 
  },
  watch: {
    loadData: {
          handler: function() {

          }
    }
  },
  methods: {
   
    async fetchData() {
       let data = await d3.csv("/data/ptsd_filtered.csv");
       var cleanedData = data.map(
          item => item[data.columns[0]].split("Z")[0]
        );
        cleanedData.removeIf(function(item, idx) {
          return item == ";";
        });
        this.loadData = cleanedData; 
    }, 
    toggleView() { 
        this.showCalendar = !this.showCalendar;
    }
  }
}


</script>

<style>

/* CSS3 */


#header {
  height: 30px;
}

/* The whole thing */
.custom-menu, .custom-menu2{
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
.custom-menu li, .custom-menu2 li {
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

/*
.smallMutipless{
  margin-top:548px;
  position: absolute;

}*/

.drawingCursor {
  cursor: crosshair;
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
    overflow-y:hidden;
    overflow-x:hidden;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

.selected {
  background-color: #1471ed66 !important;
}

.selectedCategory {
  border: 1.5px dotted;
}


.buttonGroup {
  float:right;
  margin-top:-250px;
  display:none;
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
  margin-top: 3px;
  float:left;
  width:150px;
  height:30px;
  border: 1px solid lightgrey;
  color: lightgrey;
  border-radius: 5px; 
}
#SaveBTN {
  float:left;
}

::placeholder{
  font-size:12px;
  color: lightgray;
}

.modal-sm {
  margin-left:-130px;
}

.selector {
  width:100px;
}
/*
#chart, .headers, #month {
  color: #ccc4c4; 
}*/

.propleft {
  float:left;
  margin-left:10px;
}

.btn-secondary, .btn-primary, .btn-default  {
  font-size:12px;
  margin:2px;
}
p {
  font-size: 12px;
  text-align:left;
  float:left;
  color: lightgrey;
}

.btn-default {
  border: 1px solid lightgray;
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
  width:1080px;
  height:520px;
  left: 20px;

}

#categorySection{
  float:right;
  width:400px;
  margin-top:-277px;
  margin-right:55px;
}

.drawingbuttons{
  float:left;
}

#save{
  margin-left:50px;
}

.weekAxis .tick line, .yaxis .tick line, .y .tick line {
        display:none;
    }

.weekAxis line {
  display: none;
}
.xAxis text{
  fill:#e6e9ed !important
}  

.xAxis path {
        stroke: e6e9ed !important;
}

.selectedCircles {
  fill: black !important;
}

#categoryHeader {
  float:left;
  margin-top:30px;
  /*font-family: "Lucida Sans Unicode";*/
  /*color:#746d6d;*/
  font-weight:100;
  font-size:18px;
}

#cat {
  width:50%;
  margin-top:70px;
  margin-left:-20px;
}

li {list-style-type: none;}

.fileupload {
  width:55px;
}

.swatch{
  border-radius: 50%;
  height:20px;
  width: 20px;
  position:absolute;
  cursor: pointer;
}

.clicked {
  fill: cadetblue !important;
}

.headers, #cat, .weekDate {
  font-size: 12px;
  
}

.weekbrush .selection{
  fill: lightgrey !important;
  stroke: lightgray !important;
}

.brush .selection {
  fill: none !important; 
  stroke: black !important; 
}

.showingDrawings {
  border: 1px solid black; 
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
    margin-top:-60px;
    margin-left:28px;
}

.brush {
  min-width: 100px;
}

ul {
  list-style-type: none;
}

.card-list {
  display: grid;
  grid-gap: 1em;
}

.card-item {
  border-radius: 5px;
  margin-left:-20px;
  width:85px;
  height:24px;
  /*border: 1px solid grey*/
}


/* small multiples*/

.rows {
 margin-top:92px;
  display: flex;
  flex-wrap: wrap;
  padding: 0 4px;
  flex-direction: row;
  float:right;
  margin-right:-40px;
}

/* Create two equal columns that sits next to each other */
.column {
  flex: 50%;
  max-width:50%;
  /*padding: 0 4px;*/
}

.column img {
  margin-top: 8px;
  vertical-align: middle;
  
}

.smallMutipless{
  position: absolute;
  margin-top:-100px !important;
  margin-left:-99px;

}

</style>
