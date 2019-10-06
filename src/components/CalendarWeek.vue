
<template>
  <div class="row">
       <Modal ref="modal" v-model="repModalOpen" :catDict.sync="catDict" :endTime="endTime" :startTime="startTime" :firstChosenDay="firstChosenDay" :lastChosenDay="lastChosenDay" :color="color"> </Modal>
    <div class="col col-sm">
      <div>
      <button
        id="back"
        name="back"
        v-on:click="displayPreviousWeek"
        class="btn btn-secondary"
      >Sidste uge</button>
      <span class="weekDate"> Viser ugen {{ this.currentMonday.toISOString().split("T")[0] }} til {{ new Date(new Date(currentMonday).setDate(this.currentMonday.getDate() + 6)).toISOString().split("T")[0] }} </span>
      <button
        id="forward"
        name="forward"
        v-on:click="displayNextWeek"
        class="btn btn-secondary"
      >Næste uge</button>
      <label id="currentMonth"></label>
    </div>

    <div id="chart"></div>
    </div>
    
    <div class="col col-sm" id="categorySection">
    <h4 id="categoryHeader"> KATEGORIER </h4>
    <CategoryPicker :chosenRect="chosenRect" :chosenDateTime="chosenDateTime" v-model="parentValue" :color.sync="color" :items="items" :repModalOpen.sync="repModalOpen" :catDict.sync="catDict"></CategoryPicker>
    </div>
  </div>
  
</template>

<script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>

<script>
import * as d3 from "d3";
import moment from "moment";
import Modal from "./Modal";
import Vue from "vue";
import App from "../App.vue";
import "bootstrap";
import "bootstrap/dist/css/bootstrap.min.css";
import CategoryPicker from './CategoryPicker.vue'


export default {
  name: "CalendarWeek",
  props: ["trackingData", "parsedData"],
  components: {
    Modal,
    CategoryPicker
  },
  data() {
    return {
      repModalOpen: false,
      catDict: {},
      color:"",
      calendarWidth: 1000,
      calendarHeight: 600,
      gridXTranslation: 80,
      gridYTranslation: 40,
      currentMonth: new Date().getMonth(),
      currentMonday: new Date(),
      currentSunday: new Date(),
      monthNames: [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December"
      ],
      publicGridWidth: 980,
      publicGridHeight: 560,
      publicCellWidth: this.publicGridWidth / 7,
      publicCellHeight: this.publicGridHeight / 8,
      cellPositions: [],
      cleanData: [],
      calendar: "",
      chartsGroup: "",
      curr: [],
      data: [],
      chosenRect: [],
      chosenDateTime: [],
       items: [
            { text: 'Venner', hex:"#F4D03F"},
            { text: 'Familie', hex:"#229954"},
            { text: 'Arbejde', hex:"#9B59B6"}
        ],
      parentValue: false,
      dict: [],
      categoryDict: {},
      lastChosenDay: "", 
      firstChosenDay: "",
      startTime: "",
      endTime: ""
    };
  },
  mounted() {
    if (localStorage.getItem('catDict')){
     this.catDict = JSON.parse(localStorage.getItem('catDict'));
    }
  },
  watch: {
    catDict:{
      handler: function(){
        
        if(Object.entries(this.catDict).length > 0) {
            localStorage.setItem('catDict', JSON.stringify(this.catDict));
        }
        
      },
      deep: true,
      immediate: true
    },

    trackingData: {
      handler: function() {
        var cleanedData = this.trackingData.map(
          item => item[this.trackingData.columns[0]]
        );
        cleanedData.removeIf(function(item, idx) {
          return item == ";";
        });
        this.cleanData = cleanedData;
        this.currentMonday = this.findTheMonday(this.cleanData);
        this.dict = this.getDayTimeDict();
        this.curr = this.getCurrentWeek(this.cleanData, this.currentMonday);
        this.drawCalender();
      },
      deep: true,
      immediate: true
    }
  },
  output() {
    return this.Chart();
  },
  methods: { 
    drawCalender(){
        this.addSVG();
        this.renderCalendarGrid();
        this.data = this.getDataForWeek(this.curr);
        this.drawGraphsForMonthlyData(this.data);
        this.setClickEventHandler(); 
    },
    handleSelection(event){
      if(this.parentValue){
        this.chosenRect = []
        this.chosenDateTime = []
        this.parentValue = false;
        }
        

      this.chosenRect.push(event);
      this.chosenDateTime.push(this.dict[[event[0],event[1]]] )
      this.chosenDateTime.sort(function(a,b){
        // Turn your strings into dates, and then subtract them
        // to get a value that is either negative, positive, or zero.
        return new Date(b) - new Date(a);
      });
      
      this.lastChosenDay = new Date(this.chosenDateTime[0][0]).toISOString();
      this.firstChosenDay = new Date(this.chosenDateTime[this.chosenDateTime.length-1][0]).toISOString();
      
      var time = this.chosenDateTime[0][1];
      var endTime = time +1
      time = time.toString()
      endTime = endTime.toString();
      if(time.length == 1){
        var prepend = "0" ;
        time = prepend + time;
      }
      this.startTime = time + ":00";

      if(endTime.length == 1){
        var prepend = "0" ;
        endTime = prepend + endTime
      }
      this.endTime = endTime + ":00";; 
      
      $("#"+Math.floor(event[0]).toString()+Math.floor(event[1]).toString()).toggleClass("selected");
    },
      setClickEventHandler() {
        d3.selectAll(".rect").on("click", this.handleSelection);
    },
    openModal(event) {
      this.chosenRect = event;
      this.modalOpen = !this.modalOpen;
      
    },
    getDayTimeDict() {
      var dict = {};
      for (var y = 0; y < 24; y++) {
        for (var x = 0; x < 7; x++) {

          var date = new Date(new Date(this.currentMonday).setDate(this.currentMonday.getDate() + x));
          date.setHours(0,0,0,0);

          dict[[x * (this.publicGridWidth / 7),
            y * (this.publicGridHeight / 24)]] = [date,y]
        }
      }      
      return dict;
    },
    getGridDict() {
      var dict = {};
      var colorDict = {};
      for (var y = 0; y < 24; y++) {
        for (var x = 0; x < 7; x++) {
          var date = new Date(new Date(this.currentMonday).setDate(this.currentMonday.getDate() + x));
          date.setHours(0,0,0,0);
          dict[[date,y]] = [x, y]
       }
      } 
      
      Object.keys(dict).forEach(key => {
        if(this.catDict[key]){
           colorDict[dict[key]] = this.catDict[key];
        }
      });     
      
      
      return colorDict
    },
    publicGridCellPositions() {
      var cellPositions = [];
      this.categoryDict = this.getGridDict();

      for (var y = 0; y < 24; y++) {
        for (var x = 0; x < 7; x++) {
          if(this.categoryDict[[x,y]]){
            cellPositions.push([
            x * (this.publicGridWidth / 7),
            y * (this.publicGridHeight / 24), this.categoryDict[[x,y]]
          ]);
          } else {
            cellPositions.push([
              x * (this.publicGridWidth / 7),
              y * (this.publicGridHeight / 24), "#ffffff"
            ]);
          }
        }
      }
      return cellPositions;
    },
    findTheMonday(data) {
      var dto = new Date(data[data.length - 1].split(";")[0]);
      var diff = dto.getDay() - 1;

      dto.setDate(dto.getDate() - diff);

      return dto;
    },
    displayPreviousWeek() {
      this.currentMonday = new Date(this.currentMonday.setDate(this.currentMonday.getDate() - 7));
      this.curr = this.getCurrentWeek(
        this.cleanData,
        this.currentMonday
      );
      this.data = this.getDataForWeek(this.curr);
      this.deleteGraph();
      this.drawCalender();
    },
    displayNextWeek() {
      this.currentMonday = new Date(this.currentMonday.setDate(this.currentMonday.getDate() + 7));
      this.curr = this.getCurrentWeek(
        this.cleanData,
        this.currentMonday
      );
      this.data = this.getDataForWeek(this.curr);
      this.deleteGraph();
      this.drawCalender();
    },
    getCurrentWeek(data, monday) {
      var currentWeek = [];
      var stopFlag = false;
      var offset = 0;
      var currentMonday = "";
      var currentSunday = -1;

      for (var i = 0; i < data.length; i++) {
        var timeStamp = data[i].split(";")[0];
        var dt = new Date(timeStamp);
        if (dt >= monday) {
          if (dt.getDay() == 1 && !stopFlag) {
            currentMonday = dt;
            var currentSunday = new Date(
              currentMonday.setDate(currentMonday.getDate() + 6)
            );
            stopFlag = true;
          }
          if (currentSunday !== -1 && dt < currentSunday) {
            currentWeek.push(dt);
          }
        }
      }

      return currentWeek;
    },
    getDataForWeek(curr) {
      var weekMatrix = [];
      for (var i = 0; i < 24; i++) {
        weekMatrix[i] = new Array(7).fill(0);
      }

      var dayOfWeek = 1;
      for (var i = 0; i < curr.length; i++) {
        var hours = curr[i].getHours();
        var currentDay = curr[i].getDay() - 1;
        if (currentDay == -1) {
          currentDay = 6;
        }

        weekMatrix[hours][currentDay] +=1;
        
      }
      return weekMatrix;
    },deleteGraph(){
      
      d3.select("svg").remove();
      }, 
      drawGraphsForMonthlyData(dataForWeek) {
     
      var cellPositions = this.publicGridCellPositions();
      var gridXTranslation = this.gridXTranslation;
      var gridYTranslation = this.gridYTranslation;
      var cellWidth = this.publicGridWidth / 7;
      var outerRadius = cellWidth / 3;
      var innerRadius = 0;
      var pie = d3.pie();
      var calendarWidth = this.calendarWidth;
      var calendarHeight = this.calendarHeight;
      var color = d3.scaleOrdinal(d3.schemeCategory10);
      var arc = d3
        .arc()
        .innerRadius(innerRadius)
        .outerRadius(outerRadius);

      // We need to index and group the pie charts and slices generated so that they can be rendered in
      // the appropriate cells. To do that, we call D3's 'pie' function of each of the data elements.
      var indexedPieData = [];
      for (var i = 0; i < dataForWeek.length; i++) {
        var pieSlices = pie(dataForWeek[i]);
      }

      this.chartsGroup.selectAll("circle").remove();

      var circles = this.chartsGroup
        .selectAll("circle")
        // use the indexed data so that each pie chart can be draw in a different cell and therefore for a different day
        .data(dataForWeek.flat())
        .enter()
        .append("circle")
        .attr("class", "arc")
        .attr("cx", 8)
        .attr("cy", 8)
        .attr("r", function(d, i) {
          return d;
        })
        .attr("transform", function(d, i) {
          var currentDataIndex = i;
          return (
            "translate(" +
            (outerRadius +
              gridXTranslation +
              cellPositions[currentDataIndex][0]) *
              1.02 +
            ", " +
            (outerRadius +
              gridYTranslation +
              cellPositions[currentDataIndex][1] -
              20) +
            ")"
          );
        });
    }, addSVG(){

    },
    renderCalendarGrid() {
      var cellPositions = this.publicGridCellPositions();
      var gridXTranslation = this.gridXTranslation;
      var gridYTranslation = this.gridYTranslation;
      var cellWidth = this.publicGridWidth / 7;
      var cellHeight = this.publicGridHeight / 24;
      var outerRadius = cellWidth / 3;
      var innerRadius = 0;
      var pie = d3.pie();
      var calendarWidth = this.calendarWidth;
      var calendarHeight = this.calendarHeight;
      var color = d3.scaleOrdinal(d3.schemeCategory10);
      var currentRect = "";
      var arc = d3
        .arc()
        .innerRadius(innerRadius)
        .outerRadius(outerRadius);
      
      this.calendarChart = d3
        .select("#chart")
        .append("svg")
        .attr("class", "calendar")
        .attr("width", calendarWidth + gridXTranslation)
        .attr("height", calendarHeight)
        .append("g");

      // Draw rectangles at the appropriate postions, starting from the top left corner. Since we want to leave some room for the heading and buttons,
      // use the gridXTranslation and gridYTranslation variables.
      this.calendarChart
        .selectAll("rect")
        .data(cellPositions)
        .enter()
        .append("rect")
        .attr("x", function(d) {
          return d[0];
        })
        .attr("y", function(d) {
          return d[1];
        })
        .attr("width", cellWidth)
        .attr("height", cellHeight)
        .style("stroke", "#ccc4c4")
        .style("fill", function(d){
          return d[2];
        })
        .style("fill-opacity", "0.5")
        .attr("id", function(d){
            return Math.floor(d[0]).toString()+Math.floor(d[1]).toString();
        })
        .attr(
          "transform",
          "translate(" + gridXTranslation + "," + gridYTranslation + ")"
        )
        .attr("class", "rect");

      var y = d3
        .scaleTime()
        .domain([new Date(2019, 0, 1), new Date(2019, 0, 2)])
        .nice(d3.timeDay)
        .range([0, calendarHeight - 45]);

      var yAxis = d3.axisLeft().scale(y).ticks(24).tickFormat(d3.timeFormat("%H"));

      this.calendarChart
        .append("g")
        .attr("class", "yaxis")
        .attr(
          "transform",
          "translate(" + gridXTranslation + "," + gridYTranslation + ")"
        )
        .call(yAxis);

      var daysOfTheWeek = [
        "Mandag",
        "Tirsdag",
        "Onsdag",
        "Torsdag",
        "Fredag",
        "Lørdag",
        "Søndag"
      ];

      // This adds the day of the week headings on top of the grid
      this.calendarChart
        .selectAll("headers")
        .data([0, 1, 2, 3, 4, 5, 6])
        .enter()
        .append("text")
        .attr("class", "headers")
        .attr("x", function(d) {
          return cellPositions[d][0];
        })
        .attr("y", function(d) {
          return cellPositions[d][1];
        })
        .attr("dx", gridXTranslation + 5) // right padding
        .attr("dy", 30) // vertical alignment : middle
        .text(function(d) {
          return daysOfTheWeek[d];
        });

      this.chartsGroup = this.calendarChart.append("svg:g");

    }
  }
};

Array.prototype.removeIf = function(callback) {
  var i = 0;
  while (i < this.length) {
    if (callback(this[i])) {
      this.splice(i, 1);
    } else {
      ++i;
    }
  }
};
</script>

