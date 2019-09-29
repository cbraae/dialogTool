
<template>
  <div>
       <Modal ref="modal" v-model="modalOpen" :chosenRect="chosenRect"></Modal>
    <div>
      <button
        id="back"
        name="back"
        v-on:click="displayPreviousWeek"
        class="btn btn-light"
      >Sidste uge</button>
      <button
        id="forward"
        name="forward"
        v-on:click="displayNextWeek"
        class="btn btn-light"
      >Næste uge</button>
      <label id="currentMonth"></label>
    </div>

    <div id="chart"></div>
   
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


export default {
  name: "CalendarWeek",
  props: ["trackingData", "parsedData"],
  components: {
    Modal
  },
  data() {
    return {
      modalOpen: true,
      calendarWidth: 1000,
      calendarHeight: 600,
      gridXTranslation: 80,
      gridYTranslation: 40,
      currentMonth: new Date().getMonth(),
      currentMonday: "",
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
      chosenRect:""
    };
  },
  mounted() {},
  watch: {
    trackingData: {
      handler: function() {
        var cleanedData = this.trackingData.map(
          item => item[this.trackingData.columns[0]]
        );
        cleanedData.removeIf(function(item, idx) {
          return item == ";";
        });
        this.cleanData = cleanedData;
        this.renderCalendarGrid();
        this.currentMonday = this.findTheMonday(this.cleanData);
        this.curr = this.getCurrentWeek(this.cleanData, this.currentMonday);
        this.data = this.getDataForWeek(this.curr);
        this.drawGraphsForMonthlyData(this.data);
        d3.selectAll(".rect").on("click", this.openModal);
        
      },
      deep: true,
      immediate: true
    }
  },
  output() {
    return this.Chart();
  },
  methods: {
    openModal(event) {
      this.chosenRect = event;
      this.modalOpen = !this.modalOpen;
      
    },
    publicGridCellPositions() {
      var cellPositions = [];
      for (var y = 0; y < 8; y++) {
        for (var x = 0; x < 7; x++) {
          cellPositions.push([
            x * (this.publicGridWidth / 7),
            y * (this.publicGridHeight / 8)
          ]);
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
      // We keep track of user's "back" and "forward" presses in this counter
      this.curr = this.getCurrentWeek(
        this.cleanData,
        new Date(this.currentMonday.setDate(this.currentMonday.getDate() - 7))
      );
      this.data = this.getDataForWeek(this.curr);
      this.drawGraphsForMonthlyData(this.data);
    },
    displayNextWeek() {
      this.curr = this.getCurrentWeek(
        this.cleanData,
        new Date(this.currentMonday.setDate(this.currentMonday.getDate() + 7))
      );
      this.data = this.getDataForWeek(this.curr);
      this.drawGraphsForMonthlyData(this.data);
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
      for (var i = 0; i < 8; i++) {
        weekMatrix[i] = new Array(7).fill(0);
      }

      var dayOfWeek = 1;
      for (var i = 0; i < curr.length; i++) {
        var hours = curr[i].getHours();
        var currentDay = curr[i].getDay() - 1;
        if (currentDay == -1) {
          currentDay = 6;
        }
        switch (true) {
          case hours < 3:
            weekMatrix[0][currentDay] += 1;
            break;
          case hours < 6:
            weekMatrix[1][currentDay] += 1;
            break;
          case hours < 9:
            weekMatrix[2][currentDay] += 1;
            break;
          case hours < 12:
            weekMatrix[3][currentDay] += 1;
            break;
          case hours < 15:
            weekMatrix[4][currentDay] += 1;
            break;
          case hours < 18:
            weekMatrix[5][currentDay] += 1;
            break;
          case hours < 21:
            weekMatrix[6][currentDay] += 1;
            break;
          case hours < 24:
            weekMatrix[7][currentDay] += 1;
            break;
        }
      }
      return weekMatrix;
    },
    drawGraphsForMonthlyData(dataForWeek) {
      // Get some random data
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
          return d * 2;
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
              25) +
            ")"
          );
        });
    },
    renderCalendarGrid() {
      var cellPositions = this.publicGridCellPositions();
      var gridXTranslation = this.gridXTranslation;
      var gridYTranslation = this.gridYTranslation;
      var cellWidth = this.publicGridWidth / 7;
      var cellHeight = this.publicGridHeight / 8;
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

      // Add the svg element.
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
        .style("stroke", "#555")
        .style("fill", "white")
        .attr("id", function(d){
            return d[0].toString()+d[1].toString();
        })
        .attr(
          "transform",
          "translate(" + gridXTranslation + "," + gridYTranslation + ")"
        )
        .attr("class", "rect");

      var y = d3
        .scaleTime()
        .domain([new Date(2019, 0, 1), new Date(2019, 0, 2)])
        .range([0, calendarHeight - 45]);

      var yAxis = d3.axisLeft().scale(y);

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

