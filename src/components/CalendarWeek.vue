
<template>
<div>
    <div>
       
        <button id="back" name="back" click="displayPreviousWeek()" class="btn btn-light"> Sidste uge </button>
        <button id="forward" name="forward" click="displayNextWeek()" class="btn btn-light">Næste uge</button>
        <label id="currentMonth"></label>
        
    </div>

    <div id="chart"> </div>
    <Modal ref="modal" v-model="modalOpen"></Modal>
    </div>
</template>

<script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>

<script> 

import * as d3 from 'd3'
import moment from 'moment';
import Modal from './Modal'
import Vue from 'vue'
import App from '../App.vue'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'


export default {
  name: 'CalendarWeek',
  props: ['trackingData', 'parsedData'],
  components: {
      Modal
  },
  data() {
      return {
          height: 600,
          width: 400,
          modalOpen: false
      };
  }, created() {
        
  },
  watch: {
  	trackingData: {
      handler: function() {
          
          var cleanedData = this.trackingData.map(item => (item[this.trackingData.columns[0]]));
          cleanedData.removeIf( function(item, idx){
              return item == ";";
          })
         drawCalendar(cleanedData);  
         
      },
      deep: true,
      immediate: true
    }
  },output() {
      return this.Chart();
    }, methods: {
      
    }
}


function drawCalendar(data){
       
        // Revealing module pattern to store some global data that will be shared between different functions.
        var d3CalendarGlobals = function() {
            var calendarWidth = 1000, 
            calendarHeight = 600,
            gridXTranslation = 80,
            gridYTranslation = 40,
            cellColorForCurrentMonth = '#EAEAEA',
            cellColorForPreviousMonth = '#FFFFFF',
            counter = 0, // Counter is used to keep track of the number of "back" and "forward" button presses and to calculate the month to display.
            currentMonth = new Date().getMonth(),
            monthNames = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"],
            datesGroup;

           
            


            function publicCalendarWidth() { return calendarWidth; }
            function publicCalendarHeight() { return calendarHeight; }
            function publicGridXTranslation() { return gridXTranslation; }
            function publicGridYTranslation() { return gridYTranslation; }
            function publicGridWidth() { return calendarWidth - 10; }
            function publicGridHeight() { return calendarHeight - 40; }
            function publicCellWidth() { return publicGridWidth() / 7; }
            function publicCellHeight() { return publicGridHeight() / 8; }
            function publicGetDatesGroup() {
                return datesGroup;
            }
            function publicSetDatesGroup(value) {
                datesGroup = value;
            }
            function publicIncrementCounter() { counter = counter + 1; }
            function publicDecrementCounter() { counter = counter - 1; }
            function publicMonthToDisplay() {
                var dateToDisplay = new Date();
                // We use the counter that keep tracks of "back" and "forward" presses to get the month to display.
                dateToDisplay.setMonth(currentMonth + counter);
                return dateToDisplay.getMonth();
            }
            function publicMonthToDisplayAsText() { return monthNames[publicMonthToDisplay()]; }
            function publicYearToDisplay() {
                var dateToDisplay = new Date();
                // We use the counter that keep tracks of "back" and "forward" presses to get the year to display.
                dateToDisplay.setMonth(currentMonth + counter);
                return dateToDisplay.getFullYear();
            }
            function publicGridCellPositions() {

                // We store the top left positions of a 7 by 5 grid. These positions will be our reference points for drawing
                // various objects such as the rectangular grids, the text indicating the date etc.
                var cellPositions = [];
                for (var y = 0; y < 8; y++) {
                    for (var x = 0; x < 7; x++) {
                        cellPositions.push([x * publicCellWidth(), y * publicCellHeight()]);
                    }
                }

                return cellPositions;
            }



            function findTheMonday(){
                
                var dto = new Date(data[data.length-1].split(";")[0]);
                var diff = dto.getDay()-1;

                dto.setDate(dto.getDate()-diff);
    
                return dto;
            }

            return {
                calendarWidth: publicCalendarWidth(),
                calendarHeight: publicCalendarHeight(),
                gridXTranslation :publicGridXTranslation(),
                gridYTranslation :publicGridYTranslation(),
                gridWidth :publicGridWidth(),
                gridHeight :publicGridHeight(),
                cellWidth :publicCellWidth(),
                cellHeight :publicCellHeight(),
                getDatesGroup : publicGetDatesGroup,
                setDatesGroup: publicSetDatesGroup,
                incrementCounter : publicIncrementCounter,
                decrementCounter : publicDecrementCounter,
                monthToDisplay : publicMonthToDisplay(),
                monthToDisplayAsText : publicMonthToDisplayAsText,
                yearToDisplay: publicYearToDisplay,
                gridCellPositions: publicGridCellPositions(),
                currentMonday: findTheMonday(),
                trackingDat: data
            }
        }();


        $(document).ready( function (){
                            renderCalendarGrid();
                            renderDaysOfMonth();
                            $('#back').click(displayPreviousWeek);
                            $('#forward').click(displayNextWeek);


 });

            function displayPreviousWeek() {
            console.log("dd")
            // We keep track of user's "back" and "forward" presses in this counter
            var curr = getCurrentWeek(d3CalendarGlobals.trackingDat, new Date(d3CalendarGlobals.currentMonday.setDate(d3CalendarGlobals.currentMonday.getDate()-7)));
            var data = getDataForWeek(curr);
            console.log(data)
            drawGraphsForMonthlyData(data);
            
          
        }

        function displayNextWeek(){

            var curr = getCurrentWeek(d3CalendarGlobals.trackingDat, new Date(d3CalendarGlobals.currentMonday.setDate(d3CalendarGlobals.currentMonday.getDate()+7)));
            var data = getDataForWeek(curr);
            drawGraphsForMonthlyData(data);

        }


function getCurrentWeek(data, monday){
            console.log(monday)
            var currentWeek = []
            var stopFlag = false;
            var offset = 0
            var currentMonday = ""
            var currentSunday = -1 
            for(var i = 0; i < data.length; i++){
                var timeStamp = data[i].split(";")[0];
                var dt = new Date(timeStamp)
                if(dt >= monday) { 
                
         
                if(dt.getDay() == 1 && !stopFlag){
                    currentMonday = dt;
                    var currentSunday = new Date(currentMonday.setDate(currentMonday.getDate()+6))
                    stopFlag = true
               
                }
                if(currentSunday !== -1 && dt < currentSunday){
                    currentWeek.push(dt)
                }
                }

            }
             return currentWeek;
           }
            
            function getDataForWeek(curr){
            var weekMatrix = [];
                for(var i=0; i<8; i++) {
                    weekMatrix[i] = new Array(7).fill(0);
                }
                
                
                var dayOfWeek = 1; 
                for(var i = 0; i < curr.length; i++ ){

                        var hours = curr[i].getHours();
                        var currentDay = curr[i].getDay()-1;
                        if(currentDay == -1){
                            currentDay = 6
                        }
                        switch(true){
                            case(hours < 3):
                                weekMatrix[0][currentDay] +=1
                                break;
                            case(hours < 6):
                                weekMatrix[1][currentDay] +=1
                                break;
                            case(hours < 9):
                                weekMatrix[2][currentDay] +=1
                                break;
                            case(hours < 12):
                                weekMatrix[3][currentDay] +=1
                                break;
                            case(hours < 15):
                                weekMatrix[4][currentDay] +=1
                                break;
                            case(hours < 18):
                                weekMatrix[5][currentDay] +=1
                                break;
                            case(hours < 21):
                                weekMatrix[6][currentDay] +=1
                                break;
                            case(hours < 24):
                                weekMatrix[7][currentDay] +=1
                                break;
                        }
                    

                }
                return weekMatrix;
                }
                

        // This function is responsible for rendering the days of the month in the grid.
        function renderDaysOfMonth(month, year) {
            $('#currentMonth').text(d3CalendarGlobals.monthToDisplayAsText() + ' ' + d3CalendarGlobals.yearToDisplay());
            // We get the days for the month we need to display based on the number of times the user has pressed
            // the forward or backward button.


            
             
          
            
        }

        function drawGraphsForMonthlyData(dataForWeek) {
            
            // Get some random data
            //var data = getDataForMonth();
            // Set up variables required to draw a pie chart
            var outerRadius = d3CalendarGlobals.cellWidth / 3;
            var innerRadius = 0;
            var pie = d3.pie();
            var color = d3.scaleOrdinal(d3.schemeCategory10);
            var arc = d3.arc().innerRadius(innerRadius).outerRadius(outerRadius);

            // We need to index and group the pie charts and slices generated so that they can be rendered in
            // the appropriate cells. To do that, we call D3's 'pie' function of each of the data elements.
            var indexedPieData = [];
            for (var i = 0; i < data.length; i++) {
                var pieSlices = pie(data[i]);
                // This loop is to store an index (j) for each of the slices of a given pie chart. Two different charts
                // on two different days will have the the same set of numbers for slices (eg: 0,1,2). This will help us
                // pick the same colors for the slices for two independent charts. Otherwise, the colors of the slices
                // will be different each day.
               /* for (var j = 0; j < pieSlices.length; j++) {
                    indexedPieData.push([pieSlices[j], i, j]);
                } */
            }
            
            var cellPositions = d3CalendarGlobals.gridCellPositions;

            d3CalendarGlobals.chartsGroup
                    .selectAll("circle")
                    .remove();

            var circles = d3CalendarGlobals.chartsGroup.selectAll("circle")
                          // use the indexed data so that each pie chart can be draw in a different cell and therefore for a different day
                          .data(dataForWeek.flat())
                          .enter()
                          .append("circle")
                          .attr("class", "arc")
                          .attr("cx", 8)
                          .attr("cy", 8)
                          .attr("r", function(d,i){

                                //return d3CalendarGlobals.dataForWeek[i][0]
                                return d*4;
                          })
                          .attr("transform", function (d, i) {
                              // This is where we use the index here to translate the pie chart and rendere it in the appropriate cell. 
                              // Normally, the chart would be squashed up against the top left of the cell, obscuring the text that shows the day of the month.
                              // We use the gridXTranslation and gridYTranslation and multiply it by a factor to move it to the center of the cell. There is probably
                              // a better way of doing this though.
                              var currentDataIndex = i;
                              return "translate(" +  ((outerRadius + d3CalendarGlobals.gridXTranslation + cellPositions[currentDataIndex][0])) * 1.02 + ", " +  ((outerRadius + d3CalendarGlobals.gridYTranslation + cellPositions[currentDataIndex][1]) - 25) + ")";
                          });/*$(".modal").toggle(); */  

        }

        // This is the initializing function. It adds an svg element, draws a set of rectangles to form the calendar grid,
        // puts text in each cell representing the date and does the initial rendering of the pie charts.
        function renderCalendarGrid(month, year) {

            // Add the svg element.
            d3CalendarGlobals.calendar = d3.select("#chart")
                         .append("svg")
                         .attr("class", "calendar")
                         .attr("width", d3CalendarGlobals.calendarWidth + d3CalendarGlobals.gridXTranslation )
                         .attr("height", d3CalendarGlobals.calendarHeight)
                         .append("g");

            // Cell positions are generated and stored globally because they are used by other functions as a reference to render different things.
            var cellPositions = d3CalendarGlobals.gridCellPositions;

            
             

            // Draw rectangles at the appropriate postions, starting from the top left corner. Since we want to leave some room for the heading and buttons,
            // use the gridXTranslation and gridYTranslation variables.
            d3CalendarGlobals.calendar.selectAll("rect")
                    .data(cellPositions)
                    .enter()
                    .append("rect")
                    .attr("x", function (d) { return d[0]; })
                    .attr("y", function (d) { return d[1]; })
                    .attr("width", d3CalendarGlobals.cellWidth)
                    .attr("height", d3CalendarGlobals.cellHeight)
                    .style("stroke", "#555")
                    .style("fill", "white") 
                    .attr("transform", "translate(" + d3CalendarGlobals.gridXTranslation + "," + d3CalendarGlobals.gridYTranslation + ")")
                    .attr("class","rect").on("click", function(){console.log("clicked"); $(this).toggleClass('clicked'); });
                    
            
               var y = d3.scaleTime()
                    .domain([new Date(2019, 0, 1), new Date(2019, 0, 2)])
                    .range([0, d3CalendarGlobals.calendarHeight-45]);

                var yAxis = d3.axisLeft()
                    .scale(y);
            
            d3CalendarGlobals.calendar
                .append("g")
                .attr("class", "yaxis")
                .attr("transform", "translate(" + d3CalendarGlobals.gridXTranslation + "," + d3CalendarGlobals.gridYTranslation + ")")
                .call(yAxis);

            var daysOfTheWeek = ["Mandag", "Tirsdag", "Onsdag", "Torsdag", "Fredag", "Lørdag", "Søndag"];
            // This adds the day of the week headings on top of the grid
            d3CalendarGlobals.calendar.selectAll("headers")
                 .data([0, 1, 2, 3, 4, 5, 6])
                 .enter().append("text")
                 .attr("class", "headers")
                 .attr("x", function (d) { return cellPositions[d][0]; })
                 .attr("y", function (d) { return cellPositions[d][1]; })
                 .attr("dx", d3CalendarGlobals.gridXTranslation + 5) // right padding
                 .attr("dy", 30) // vertical alignment : middle
                 .text(function (d) { return daysOfTheWeek[d] });



    d3CalendarGlobals.chartsGroup = d3CalendarGlobals.calendar.append("svg:g");
            // Call the function to draw the charts in the cells. This will be called again each time the user presses the forward or backward buttons.
        
            var curr = getCurrentWeek(d3CalendarGlobals.trackingDat, d3CalendarGlobals.currentMonday);
            var data = getDataForWeek(curr);
            drawGraphsForMonthlyData(data);
            
        }

}



Array.prototype.removeIf = function(callback) {
    var i = 0;
    while (i < this.length) {
        if (callback(this[i])) {
            this.splice(i, 1);
        }
        else {
            ++i;
        }
    }
};


</script>

