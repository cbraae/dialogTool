
<template>
  <div id="row"> 
    
    <!-- TOP ROW: Timeline og knap-dahsboard -->
    <div id="row" class="Upper">
     

         

      
    <div class="tooltipz"> <p id="value">dette er en mega fed tooltip</p></div>
    </div>

      <!-- Nederste: SKEMA OG SMALL MULTIPLES -->

      
      <div id="chart"></div>
    

      <!-- menu til at fjerne brush-->
     <ul class='custom-menu'>
        <li data-action="first">Fjern Visning</li>
      </ul>
      <!-- menu til at slette small multiples-->
      <ul class='custom-menu-delete'>
        <li data-action="first">Slet</li>
      </ul> 
      
  </div>
</template>


<script>
import * as d3 from 'd3'
import moment from "moment";
import Vue from "vue";
import App from "../App.vue";
import "bootstrap";
import "bootstrap/dist/css/bootstrap.min.css";




export default {
  name: "CalendarWeek",
  props: ["trackingData", "parsedData", "chosenMonday", "chosenSunday"],
  components: {

  },
  data() {
    return {
      repModalOpen: false,
      catDict: {},
      color:"",
      brushes: [],
      displayingDrawings:false,
      calendarWidth: 1080,
      calendarHeight: 450,
      gridXTranslation: 55,
      gridYTranslation: 40,
      currentMonth: new Date().getMonth(),
      currentMonday: new Date(),
      currentSunday: new Date(),
      showDrawingsTitle: "Vis tidligere kommentarer",
      numberOfWeeks: 0,
      gridColor: "lightgrey",
      currentlyShownImages: [],
      brushImageDict: {},
      currentlyShownMondays: {},
      ImagePlaceInDict: {},
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
      publicGridWidth: 1010,
      canvasExists: false,
      publicGridHeight: 400,
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
      colorDict:{},
      dict: [],
      categoryDict: {},
      categoryOverviewDict:{},
      imgDict: {},
      svgDict: {},
      lastChosenDay: "", 
      firstChosenDay: "",
      startTime: "",
      endTime: "",
      observationsPerDay: {},
      categoryNumberDict: {},
      chosenColors: [],
      
    };
  },
  mounted() {
      
  },
  watch: {
    chosenMonday: {
        handler: function(){

         if(this.chosenMonday){
            this.cleanData = this.trackingData
            this.dict = this.getDayTimeDict();
            this.curr = this.getCurrentWeek(this.cleanData, this.chosenMonday);
            this.displayChosenWeek(this.chosenMonday,this.chosenSunday);
         }
        
        
      },
      deep: true,
      immediate: true
    },
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

      
            
        //this.observationsPerDay = this.getTotalForEachDay();
        

        if (localStorage.getItem('catDict')){
        this.catDict = JSON.parse(localStorage.getItem('catDict'));
        };

        if (localStorage.getItem('categories')){
        this.items = JSON.parse(localStorage.getItem('categories'));
        };

      },
      deep: true,
      immediate: true
    },
    color: {
       handler: function() {
         //this.enableDrawing(this.color, false);
       },
       deep: true
    }
  },
  output() {
    //return this.Chart();
  },
  methods: { 
    setupxAxisForWeek(startDate, endDate){
        var width = this.calendarWidth;
        
        var xScale = d3.scaleTime()
        .domain([
          startDate, endDate])
        .range([0, width-70]);

        var xAxis = d3.axisTop().scale(xScale).ticks(7).tickFormat(d3.timeFormat("%A %d %b"));

      var gridColor = this.gridColor
      
       this.calendarChart
        .append("g")
        .attr("class", "weekAxis")
        .attr(
          "transform",
          "translate(115,35)")
        .call(xAxis).call(g => g.select(".domain").remove())
        .selectAll("text")
        .style("fill", "#DE8D68")

        this.calendarChart.selectAll(".xaxis path").style("stroke", gridColor)
        this.calendarChart.selectAll(".xaxis line").style("stroke", gridColor)
    }, 
    drawCalender(){
        this.renderCalendarGrid();
        //this.data = this.getDataForWeek(this.curr);
        this.drawGraphsForMonthlyData(this.data);
        //this.setClickEventHandler(); 
        //this.drawCategoryGrid()
    },
    getDayTimeDict() {
      var dict = {};
      for (var y = 0; y < 24; y++) {
        for (var x = 0; x < 7; x++) {

          var date = new Date(new Date(this.chosenMonday).setDate(this.chosenMonday.getDate() + x));
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

          var date = new Date(new Date(this.chosenMonday).setDate(this.chosenMonday.getDate() + x));
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

      var dto = new Date(data[data.length - 1]);
      var diff = dto.getDay() - 1;
      dto.setDate(dto.getDate() - diff);
     
      return dto;
    },
    displayChosenWeek(startDate, endDate){
      this.currentMonday = startDate
      //Find startdato og slutdato
      //Vis data i dette interval
      this.curr = this.getCurrentWeek(
        this.cleanData,
        startDate
      );
      this.data = this.getDataForWeek(this.curr, startDate, endDate);
      this.deleteGraph();
      this.drawCalender();
      
      this.setupxAxisForWeek(startDate, endDate)

    },
    getCurrentWeek(data, monday) {
      
      var currentWeek = [];
      var stopFlag = false;
      var offset = 0;
      var currentMonday = monday;
      var currentSunday = -1;
      
      for (var i = 0; i < data.length; i++) {
        var timeStamp = data[i];
        var dt = new Date(timeStamp);
        if (dt >= monday) {
          
          var diff = dt.getDate() - 1

            var currentSunday = new Date(
              new Date(currentMonday).setDate(currentMonday.getDate() + 6)
            );
            currentSunday.setHours(23,59,59,59)
            stopFlag = true;
          }

          if (currentSunday !== -1 && dt <= currentSunday) {
            
            currentWeek.push(dt);
          }

      }
      
      return currentWeek;
    },
    getDataForWeek(curr, startDate,endDate) {
      
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
      d3.select("#calendar").remove();
      //this.clearDrawing();
    }, 
    drawGraphsForMonthlyData(dataForWeek) {
     
      var cellPositions = this.publicGridCellPositions();
      var gridXTranslation = this.gridXTranslation;
      var gridYTranslation = this.gridYTranslation;
      var cellWidth = this.publicGridWidth / 7;
      var outerRadius = cellWidth / 3;
      var innerRadius = 0;
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


      this.chartsGroup.selectAll("circle").remove();

      var _this = this;



      function handleMouseOut(d, i) {
            // Use D3 to select element, change color back to normal
            d3.select(this).style("fill", "black")

            $(".tooltipz").hide();
          }

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
          return (Math.sqrt(d *4/Math.PI)) * 2;
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
              gridYTranslation - 28+
              cellPositions[currentDataIndex][1] -
              20) +
            ")"
          );
        }).on("mouseover", handleMouseOver)
        .on("mouseout", handleMouseOut);



          
      function handleMouseOver(d, i) {  // Add interactivity

            // Use D3 to select element, change color and size
            d3.select(this).style("fill", "#DE8D68")

            var circle = this;
            var top = $(circle).position().top;
            var left = $(circle).position().left;

            d3.select(".tooltipz").style("left", left +20 +"px")
            .style("top", top+"px").select("#value").text(d)

           $(".tooltipz").show();

          }
  }, 
  
    renderCalendarGrid() {
      $(".tooltipz").hide();
     
      var cellPositions = this.publicGridCellPositions();
      var gridXTranslation = this.gridXTranslation;
      var gridYTranslation = this.gridYTranslation;
      var publicGridHeight =  this.publicGridHeight;
      var cellWidth = this.publicGridWidth / 7;
      var cellHeight = this.publicGridHeight / 24;
      var outerRadius = cellWidth / 3;
      var innerRadius = 0;
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
        .attr("id", "calendar")
        .attr("width", calendarWidth)
        .attr("height", calendarHeight)
        .append("g");

      var gridColor = this.gridColor;


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
        .style("stroke", gridColor)
        .style("fill", "none")
        .style("stroke-opacity", "0.3")
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
        .range([0, publicGridHeight]);

      var yAxis = d3.axisLeft().scale(y).ticks(24).tickFormat(d3.timeFormat("%H"));

      this.calendarChart
        .append("g")
        .attr("class", "yaxis")
        .attr(
          "transform",
          "translate(" + gridXTranslation + "," + gridYTranslation + ")"
        )
        .call(yAxis).call(g => g.select(".domain").remove())
        .selectAll("text")
        .style("fill", gridColor);

         this.calendarChart.selectAll(".yaxis path").style("stroke", gridColor)
         this.calendarChart.selectAll(".yaxis line").style("stroke", gridColor)


      var daysOfTheWeek = [
        "Mandag",
        "Tirsdag",
        "Onsdag",
        "Torsdag",
        "Fredag",
        "Lordag",
        "Sondag"
      ];
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

