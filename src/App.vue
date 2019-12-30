
<template>
  <div id="app">
     <nav class="navbar navbar-dark bg-dark" id="header">
    
      <label class="fileUpload"> Upload fil <FileUpload :loadData.sync="loadData"/> </label>
      </nav>

      <Timeline :trackingData=loadData :chosenMonday.sync="chosenMonday" :chosenSunday.sync="chosenSunday"
      :startDate.sync="startDate" :endDate.sync="endDate" :brushId.sync="brushId" :brushEnd.sync="brushEnd" :data.sync="data"/>
       <CalendarWeek :trackingData=loadData :chosenMonday=chosenMonday :chosenSunday=chosenSunday />
       <div id="categorySection" class="right">
          <p id="categoryHeader"> KATEGORIER</p>
          <p class="desc">
          Tilføj en tegning ved at: <br> 1. Trykke på en kategori <br> 2. Tegn på kalenderen ved at holde shift + venstre musetast nede.
          </p>
          <CategoryPicker class="categoryPicker" :color.sync="color" :items.sync="items" :catDict.sync="catDict" :drawingSaved.sync="drawingSaved"></CategoryPicker>
      </div>
      <DrawingsoverlayComponent :startDate=startDate :endDate=endDate :brushId=brushId :brushEnd=brushEnd imgDict.sync="imgDict" svgDict.sync="svgDict" brushes.sync="brushes"  :drawingSaved.sync="drawingSaved"/>
      <CategoryOverview :drawingSaved=drawingSaved :items=items :trackingData=loadData /> 
      
         
      <createDrawingComponent :trackingData=loadData :color=color  :chosenMonday=chosenMonday :drawingSaved.sync="drawingSaved" imgDict.sync="imgDict" colorDict.sync="colorDict" svgDict.sync="svgDict"/>
        
     
  </div>
</template>

<script>
import CalendarWeek from './components/CalendarWeek'
import FileUpload from './components/FileUpload'
import Timeline from './components/Timeline'
import CategoryPicker from './components/CategoryPicker.vue'
import createDrawingComponent from './components/CreateDrawingComponent.vue'
import DrawingsoverlayComponent from './components/DrawingsOverlayComponent.vue'
import CategoryOverview from './components/CategoryOverview.vue'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import * as d3 from 'd3'

export default {
  name: 'app',
  components: {
    CalendarWeek,
    FileUpload, 
    Timeline,
    CategoryPicker,
    createDrawingComponent,
    DrawingsoverlayComponent,
    CategoryOverview
   
  }, data(){
    return {
      loadData: {},
      drawingSaved: "",
      showCalendar: true,
      chosenMonday: "",
      chosenSunday: "",
      color: "",
      startDate: "",
      endDate:"",
      brushId:"",
      brushEnd:false,
      catDict: {},
      items: [],
      imgDict: {},
      colorDict: {},
      svgDict: {},
      data: {}
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
       let data = await d3.csv("/data/ptsd_filtered.csv")
       var firstDataPoint = data[0]

       var cleanedData = data.map(
         
          item => item[data.columns[0]].split("Z")[0]

        );
        cleanedData.removeIf(function(item, idx) {
          return item == ";";
        });

      if(cleanedData.length > 1) {
        var trackData = cleanedData.filter(function (el) {
          return el != "";
      })

        var _this = this; 
        var cleanData = trackData.map(function(element) { 
          
          var elem = _this.parseDate(element)
          return elem;
        });

        this.loadData = cleanData; 
      }
    },
      parseDate(input) {
      
      if(input.toString().includes("-")){
        input+="z"
        return new Date(input)
      } else 
        return new Date(Date.UTC(
          parseInt(input.toString().slice(0, 4), 10),
          parseInt(input.toString().slice(4, 6), 10) - 1,
          parseInt(input.toString().slice(6, 8), 10),
          parseInt(input.toString().slice(9, 11), 10),
          parseInt(input.toString().slice(11, 13), 10),
          parseInt(input.toString().slice(13,15), 10)
        ));
  }
  }
}

</script>