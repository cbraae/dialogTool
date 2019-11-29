
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
       let data = await d3.csv("/data/ptsd_filtered.csv")
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

