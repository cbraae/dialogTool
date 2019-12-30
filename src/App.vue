
<template>
  <div id="app">
     <nav class="navbar navbar-dark bg-dark" id="header">
    
      <label class="fileUpload"> Upload fil <FileUpload :loadData.sync="loadData"/> </label>
      </nav>

      <Timeline :trackingData=loadData />
     <CalendarWeek :trackingData=loadData />
    
     
  </div>
</template>

<script>
import CalendarWeek from './components/CalendarWeek'
import FileUpload from './components/FileUpload'
import Timeline from './components/Timeline'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import * as d3 from 'd3'

export default {
  name: 'app',
  components: {
    CalendarWeek,
    FileUpload, 
    Timeline
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