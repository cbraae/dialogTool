<template>

  <div id="app">
     <nav class="navbar navbar-dark bg-dark" id="header">
      <!--<button @click="toggleView" class="btn btn-secondary headerbtn"> Skift visning </button>  -->
      <label class="fileUpload"> Upload fil <FileUpload :loadData.sync="loadData"/> </label>
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
.custom-menu, .custom-menu2, .custom-menu-delete{
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
.custom-menu li, .custom-menu2 li, .custom-menu-delete li {
    padding: 8px 12px;
    cursor: pointer;
    list-style-type: none;
    transition: all .3s ease;
    user-select: none;
}

.custom-menu li:hover,  .custom-menu2 li:hover, .custom-menu-delete li:hover {
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
  border: 1.5px dotted !important;
}


.buttonGroup {
  float:left;
  visibility:hidden;
  margin-left:-30px;
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

.btn-secondary, .btn-primary, .btn-default, .fileupload, .custom-menu, .custom-menu2,  .custom-menu-delete{
  font-size:12px;
  margin:2px;
  background-color: white;
}
p, .imagetitles {
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
  height:455px;
  left: 20px;
  margin-top:40px;
}
#calendar {
  z-index:1;
}

#categorySection{
  float:right;
  width:400px;
  margin-top:-200px;
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

.gridHeader {
  float: left;
  margin-left: 110px;
}

.fileUpload {
  font-size:12px;
  color: black;
  border-radius:5px;
  margin-top:5px;
  width:100px;
  height:20px;;
  background-color:white;
  border: 1px solid #ccc;
    cursor: pointer;
}

input[type="file"] {
    display: none;
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
  fill: white !important;
  stroke: lightgray !important;
}

.brush .selection {
  fill:  #90ee900f !important; 
  stroke: lightgrey !important; 
}

.showingDrawings {
  border: 1px solid lightgrey; 
  background-color: #90ee9003;
 
}

.selected {
  fill: #007bff !important;
  fill-opacity: 0.1;
}

.tooltipz{
  display: none;
  position: absolute;
  display:block;
  width:20px;
  height:15px;
  z-index: 1000;
  background-color: white;
  font-size:12px;
  border: 1px solid lightgray;
  border-radius: 5px;
}

.tooltipz p {
  color: black;
  margin-left: 5px;
  margin-top: -2px;
  text-align: center;
}

.repChooser{
  width:48%
}

.prop {
  float: left;
}

#selector {
  margin-left: 10px; 
  fill: none;
}

#categoryOverview{
    margin-left:20px;
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

/* GRID STYLING */ 
.upper {
  display: flex;
  flex-wrap: wrap;
  padding: 0 4px;
  flex-direction: row;
}

.lower {
  display: flex;
  flex-wrap: wrap;
  padding: 0 4px;
  flex-direction: row;
}

.left {
  flex: 66%;
  max-width:66%;
  /*padding: 0 4px;*/
}
.right {
  flex: 33%;
  max-width:33%;
  /*padding: 0 4px;*/
}

/* small multiples*/

.rows {
 margin-top:58px;
  display: flex;
  flex-wrap: wrap;
  padding: 0 4px;
  flex-direction: row;
  float:right;
}

/* Create two equal columns that sits next to each other */
.column {
  margin-top:10px;
  margin-right:2px;
  flex: 48%;
  max-width:48%;
  /*padding: 0 4px;*/
}

.column img {
  vertical-align: middle;
  
}

.smallMutipless{
  position: absolute;
  margin-top:-100px;

}

</style>
