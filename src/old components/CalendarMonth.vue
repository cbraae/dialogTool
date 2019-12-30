<template>


<div class="row">
  <div class="col col-sm col-8">

    <div class="buttonGroup">
      <button
        id="back"
        name="back"
        v-on:click="displayPreviousMonth"
        class="btn btn-secondary"
      >Sidste måned</button>
      <span class="weekDate"> Viser {{ monthNames[this.currentMonth]}} {{this.currentYear }}</span>
      <button
        id="forward"
        name="forward"
        v-on:click="displayNextMonth"
        class="btn btn-secondary"
      >Næste måned</button>
      <label id="currentMonth"></label>
    </div>

    <div id="month">
    </div>
    </div>
    
   <div class="col col-sm col-4" id="categorySection">
    <h4 id="categoryHeader"> KATEGORIER </h4>
    <CategoryPicker :chosenRect="chosenRect" :chosenDateTime="chosenDateTime" v-model="parentValue" :color.sync="color" :items.sync="items" :repModalOpen.sync="repModalOpen" :catDict.sync="catDict"></CategoryPicker>
    </div></div>    
</template>

<script>
import * as d3 from "d3";
import CategoryPicker from './CategoryPicker.vue'

export default {
  name: "CalendarMonth",
  props: ["trackingData"],
  components: {
    CategoryPicker,
  },
  data() {
    return {
      chosenRect: "",
      chosenDateTime:"",
      parentValue: "",
      color:"",
      items:"",
      repModalOpen:"",
      currentMonthData: [],
      DaysInMonth: [],
      currentMonth: "",
      currentYear: "",
      calendarWidth: 1000,
      calendarHeight: 500,
      gridXTranslation: 80,
      gridYTranslation: 40,
      monthNames: [
        "Januar",
        "Februar",
        "Marts",
        "April",
        "Maj",
        "Juni",
        "Juli",
        "August",
        "September",
        "Oktober",
        "November",
        "December"
      ],
      publicGridWidth: 980,
      publicGridHeight: 460,
      publicCellWidth: this.publicGridWidth / 7,
      publicCellHeight: this.publicGridHeight / 8,
      cellPositions: [],
      calendar: "",
      monthGroup: "",
      monthDataMatrix : [],
      categoryDict: {},
      catDict: {}, 
      items: [
            { text: 'Venner', hex:"#F4D03F"},
            { text: 'Familie', hex:"#229954"},
            { text: 'Arbejde', hex:"#9B59B6"}
        ],
    };
  },
  mounted() {
    if (localStorage.getItem('catDict')){
     this.catDict = JSON.parse(localStorage.getItem('catDict'));
    };
    if (localStorage.getItem('categories')){
     this.items = JSON.parse(localStorage.getItem('categories'));
     
    };
    this.drawCalender();

  },
  watch: {
      trackingData: {
      handler: function() {
        this.currentMonth = new Date(this.trackingData[this.trackingData.length -1]).getMonth();
        this.currentYear = new Date(this.trackingData[this.trackingData.length -1]).getFullYear();

        this.firstDayOfMonth = new Date(this.currentYear, this.currentMonth, 1);
        this.CurrentMonthData = this.getCurrentMonthData(this.currentMonth);
        this.DaysInMonth = this.getDaysInCurrentMonth(this.currentMonth, this.currentYear);
        this.monthDataMatrix = this.getDataForMonthMatrix(this.CurrentMonthData, this.DaysInMonth.length)
        
      },
      deep: true,
      immediate: true
    },
  },
  output() {
    return this.Chart();
  },
  methods: { 
    displayPreviousMonth() {
      this.firstDayOfMonth = new Date(this.firstDayOfMonth.setMonth(this.firstDayOfMonth.getMonth()-1));
      
      this.CurrentMonthData = this.getCurrentMonthData(this.firstDayOfMonth.getMonth());
      this.currentMonth = this.firstDayOfMonth.getMonth();
      this.currentYear = this.firstDayOfMonth.getFullYear();
      this.DaysInMonth = this.getDaysInCurrentMonth(this.firstDayOfMonth.getMonth(), this.firstDayOfMonth.getFullYear());
      this.monthDataMatrix = this.getDataForMonthMatrix(this.CurrentMonthData, this.DaysInMonth.length)
      this.deleteGraph();
      this.drawCalender();
    },
    displayNextMonth() {
      this.firstDayOfMonth = new Date(this.firstDayOfMonth.setMonth(this.firstDayOfMonth.getMonth()+1));
      
      this.CurrentMonthData = this.getCurrentMonthData(this.firstDayOfMonth.getMonth());
      this.currentMonth = this.firstDayOfMonth.getMonth();
      this.currentYear = this.firstDayOfMonth.getFullYear();
      this.DaysInMonth = this.getDaysInCurrentMonth(this.firstDayOfMonth.getMonth(), this.firstDayOfMonth.getFullYear());
      this.monthDataMatrix = this.getDataForMonthMatrix(this.CurrentMonthData, this.DaysInMonth.length)
      this.deleteGraph();
      this.drawCalender();
    },
    drawCalender(){
        this.renderCalendarGrid();
        this.drawGraphsForMonthlyData(this.monthDataMatrix);
    },
     getDataForMonthMatrix(curr, lengthOfMonth) {
      var monthMatrix = [];

      for (var i = 0; i < 24; i++) {
        monthMatrix[i] = new Array(lengthOfMonth).fill(0);
      }

      var dayOfWeek = 1;
      for (var i = 0; i < curr.length; i++) {
        var hours = curr[i].getHours();
        var currentDay = curr[i].getDate()-1;
        monthMatrix[hours][currentDay] +=1;
        
      }
      
      return monthMatrix;
    },
    getCurrentMonthData(currMonth){
      var currMonthArray = []
      this.trackingData.forEach(element => {
        
        if(new Date(element).getMonth() == currMonth) {
          currMonthArray.push(new Date(element));
        }
      });
      return currMonthArray;
    },
    getDaysInCurrentMonth(month, year){
     var date = new Date(Date.UTC(year, month, 1));
     var days = [];
     while (date.getMonth() === month) {
        days.push(new Date(date));
        date.setDate(date.getDate() + 1);
     }
     return days;
    },
    publicGridCellPositions() {
      var cellPositions = [];
      this.categoryDict = this.getGridDict();

      for (var y = 0; y < 24; y++) {
        for (var x = 0; x < this.DaysInMonth.length; x++) {
          if(this.categoryDict[[x,y]]){
            cellPositions.push([
            x * (this.publicGridWidth / this.DaysInMonth.length),
            y * (this.publicGridHeight / 24), this.categoryDict[[x,y]]
          ]);
          } else {
            cellPositions.push([
              x * (this.publicGridWidth / this.DaysInMonth.length),
              y * (this.publicGridHeight / 24), "#ffffff"
            ]);
          }
        }
      }
      return cellPositions;
    },
    getGridDict() {
      var dict = {};
      var colorDict = {};
      for (var y = 0; y < 24; y++) {
        for (var x = 0; x < this.DaysInMonth.length; x++) {
          
          var date = new Date(new Date(this.firstDayOfMonth).setDate(this.firstDayOfMonth.getDate() + x));
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

    deleteGraph(){
      d3.select("svg").remove();
    }, 
    drawGraphsForMonthlyData(dataForWeek) {
     var dataForMonth = this.monthDataMatrix;
      var cellPositions = this.publicGridCellPositions();  
      var gridXTranslation = this.gridXTranslation;
      var gridYTranslation = this.gridYTranslation;
      var cellWidth = this.publicGridWidth / 7;
      var outerRadius = cellWidth / 5;
      var innerRadius = 0;
      var pie = d3.pie();
      var calendarWidth = this.calendarWidth;
      var calendarHeight = this.calendarHeight;
      var color = d3.scaleOrdinal(d3.schemeCategory10);
      var arc = d3
        .arc()
        .innerRadius(innerRadius)
        .outerRadius(outerRadius);

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
          return (Math.sqrt(d *4/Math.PI));
        })
        .attr("transform", function(d, i) {
          var currentDataIndex = i;
          return (
            "translate(" +
            (10+
              gridXTranslation +
              cellPositions[currentDataIndex][0]) +
            ", " +
            (outerRadius +
              gridYTranslation -28 +
              cellPositions[currentDataIndex][1]) +
            ")"
          );
        });
    }, 
    renderCalendarGrid() {
      var cellPositions = this.publicGridCellPositions();
      var gridXTranslation = this.gridXTranslation;
      var gridYTranslation = this.gridYTranslation;
      var cellWidth = this.publicGridWidth / this.DaysInMonth.length;
      var cellHeight = this.publicGridHeight / 24;
      var outerRadius = cellWidth / 5;
      var innerRadius = 0;
      var pie = d3.pie();
      var calendarWidth = this.calendarWidth;
      var calendarHeight = this.calendarHeight;
      var color = d3.scaleOrdinal(d3.schemeCategory10);
      var currentRect = "";
      var daysInMonth = this.DaysInMonth;
      var arc = d3
        .arc()
        .innerRadius(innerRadius)
        .outerRadius(outerRadius);
      
      this.monthCalendar = d3
        .select("#month")
        .append("svg")
        .attr("class", "calendar")
        .attr("width", calendarWidth + gridXTranslation)
        .attr("height", calendarHeight)
        .append("g");

      this.monthCalendar
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

      
      var mindate = new Date(daysInMonth[0].setDate(daysInMonth[0].getDate())),
            maxdate = new Date(daysInMonth[daysInMonth.length-1]);

      var xScale = d3.scaleTime()
          .domain([mindate, maxdate])
          .nice(d3.timeDay)
          .range([0,calendarWidth-18]); ;    // values between for month of january

      var xAxis = d3.axisBottom().scale(xScale).ticks(daysInMonth.length).tickSizeInner(0).tickSizeOuter(0)
            

      var yAxis = d3.axisLeft().scale(y).ticks(24).tickFormat(d3.timeFormat("%H"));

      this.monthCalendar
        .append("g")
        .attr("class", "yaxis")
        .attr(
          "transform",
          "translate(" + gridXTranslation  + "," + gridYTranslation + ")"
        )
        .call(yAxis);

      this.monthCalendar
        .append("g")
        .attr("class", "xaxis")
        .attr(
          "transform",
          "translate(" + gridXTranslation+ ")"
        )
        .call(xAxis)
        .call(g => g.select(".domain").remove());


        this.monthCalendar.selectAll(".xaxis text")  // select all the text elements for the xaxis
          .attr("transform", function(d) {
              return "translate(" + this.getBBox().height*2 + "," + this.getBBox().height + ")rotate(-45)";
        });

      var daysOfTheWeek = [
        "Mandag",
        "Tirsdag",
        "Onsdag",
        "Torsdag",
        "Fredag",
        "Lørdag",
        "Søndag"
      ];

      this.chartsGroup = this.monthCalendar.append("svg:g");
    }
  }
};



</script>