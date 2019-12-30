<template>
<div id="categoryOverview" class="left"> </div>
</template>

<script>
import * as d3 from 'd3'

export default {
    name: "categoryOverview",
    props: ["trackingData", "drawingSaved"],
    watch: {
      drawingSaved: {
        handler: function(){
          if (localStorage.getItem('categories')){
          this.items = JSON.parse(localStorage.getItem('categories'));
          };
          this.numberOfWeeks = this.getAmountOfWeeks();
          this.drawCategoryGrid()
        }
      }, 
      deep: true,
      immediate: true,
      trackingData: {
        handler: function(){
          if(localStorage.getItem('categories')){
          this.items = JSON.parse(localStorage.getItem('categories'));
          };
          this.numberOfWeeks = this.getAmountOfWeeks();
          this.drawCategoryGrid()
        }
      }, deep: true, 
      immediate: true
      
    },
    data(){
      return {
        numberOfWeeks: "",
        categoryOverviewDict: {},
        categoryNumberDict: {},
        imgDict: {},
        colorDict: {},
        items: []

      }
    },

    methods: {
      getTotalForEachDay(){
      Date.prototype.addDays = function(days) {
          var date = new Date(this.valueOf());
          date.setDate(date.getDate() + days);
          return date;
      }
      function getDates(startDate, stopDate) {
          var sunday = new Date(stopDate.setDate(stopDate.getDate() + (7 - stopDate.getDay()) % 7))
          var dateArray = new Array();
          var currentDate = startDate;
          currentDate.setHours(0,0,0,0)
          while (currentDate <= sunday) {
              for(var i = 0; i < 24; i++){
              var currentDay = new Date (currentDate)
              currentDay.setHours(i)
                dateArray.push({date: currentDay, count: 0});
              }
              
              currentDate = currentDate.addDays(1);
              //currentDate.setHours(0,0,0,0)
          }
          return dateArray;
      }
      var datesInPeriod= getDates(new Date((this.trackingData[0])), new Date( this.trackingData[this.trackingData.length-1]))
     
      var tally = [];
      var firstTime = true
      var lastDate = new Date( new Date(this.trackingData[0]))
      var counter = 0
      var cleanData = this.trackingData;
      var counter = 0
    cleanData.forEach(function(line) {
              var date = new Date( new Date(line))
              if(lastDate.getHours()===date.getHours() ){
                counter++;
              } else { 
                tally.push({date:date,count:counter})
                counter = 0;
              }
              lastDate = date;
      });
      var lastDate = new Date( new Date(this.trackingData[0]))
    
        for(var i=0; i < tally.length; i++){
             for(var j = 0; j < datesInPeriod.length; j++ ) {
                  if(tally[i].date.getDate() == datesInPeriod[j].date.getDate() && tally[i].date.getHours() == datesInPeriod[j].date.getHours() && tally[i].date.getMonth() == datesInPeriod[j].date.getMonth()){
                    datesInPeriod[j].count = tally[i].count
                  }
             }
        }
      
      return datesInPeriod;
    },   
    categoryGridCells() {
      var cellPositions = [];
      var width = 1030;
      var height = 4;
      var numberOfWeeks = this.numberOfWeeks;
      var numberOfCategories = this.items.length;
      this.categoryOverviewDict = this.initialiseCategoryDict();
      var categoryDict = this.categoryOverviewDict;

      for (var y = 0; y < numberOfCategories; y++) {
        
        for (var x = 0; x < numberOfWeeks; x++) {
            
            cellPositions.push([
            x * ((width/ numberOfWeeks)),
            y * (height),categoryDict[x][y],
          ]);
        }
      }
      return cellPositions;
    },
    getAmountOfWeeks(){
      var data = this.getTotalForEachDay();
      return d3.timeSunday.count(new Date(data[0].date), new Date(data[data.length-1].date))
    },getMonday(d){
        d = new Date(d);
            if(d.getDay() === 1){
              d.setHours(0,0,0,0)
              return d;
            }

            var day = d.getDay(),
                diff = d.getDate() - day + (day == 0 ? -6:1); // adjust when day is sunday

            var monday = new Date(d.setDate(diff));
            monday.setHours(0,0,0,0);
            
            return monday
    }, 
    findCurrentWeekNumber(chosenDate){
          var monday = this.getMonday(chosenDate);
          return d3.timeMonday.count(new Date(this.trackingData[0]), monday)

    },
    initialiseCategoryDict(){
      
      if (localStorage.getItem('imgDict')){
        this.imgDict = JSON.parse(localStorage.getItem('imgDict'));
      };
      if(localStorage.getItem('colorDict')){
            this.colorDict =JSON.parse(localStorage.getItem('colorDict'));
      }


    var dict = {};
    var items = this.items;
    this.categoryNumberDict = this.initialiseCategoryNumberDict();
    for(var i = 0; i < this.numberOfWeeks; i++){
      var categoryObject = {}
      var iterator = 0;

      for(var key in items){
          var cat = items[key];
          if(cat.hex) {
            categoryObject[iterator] = "white"
          }
          iterator++;
      }
      dict[i] = categoryObject;
      }

      for(var monday in this.imgDict){
          var image = this.imgDict[monday]
          for(var img in image){
               var chosenColors = this.colorDict[image[img]];
               var numberInDict = this.findCurrentWeekNumber(monday);

          for(var color in chosenColors){
            var color = chosenColors[color];

            if(color.toString().includes("#")){
                var catNumber = this.categoryNumberDict[color]
                if(dict[numberInDict][catNumber]){
                  dict[numberInDict][catNumber] = color;
                }
                
            }
          }

        }
        
      }

      //localStorage.setItem("categoryOverviewDict", JSON.stringify(dict));

    return dict;
    },
    initialiseCategoryNumberDict(){
    var dict = {};
    for(var key in this.items){
     if(this.items[key]){
        var color = this.items[key].hex;
        dict[color] = key; 
        }
      }
    return dict;

 },
    drawCategoryGrid(){
      console.log("hej")
      $("#category").remove();
        this.categoryOverviewDict = this.initialiseCategoryDict();
        this.categoryNumberDict = this.initialiseCategoryNumberDict();
        var cellPositions = this.categoryGridCells();
        var numberOfWeeks = this.numberOfWeeks;
        var cellWidth = 1030/numberOfWeeks;
        var numberOfCategories = this.items.length;
        var cellHeight = 40/numberOfCategories;      

      var gridXTranslation = this.gridXTranslation;
      var gridYTranslation = this.gridYTranslation;

      var categoryHeight = 4 * numberOfCategories; 

      var categoryOverview = d3
        .select("#categoryOverview")
        .append("svg")
        .attr("class", "categories")
        .attr("id", "category")
        .attr("width", 1200)
        .attr("height", categoryHeight)
        .append("g");

      categoryOverview
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
        .style("stroke", function(d){
          return "white";
        })
        .style("fill", function(d){
          return d[2];
        })
        .attr("id", function(d){
            return Math.floor(d[0]).toString()+Math.floor(d[1]).toString();
        })
        .attr(
          "transform",
          "translate(" + 40 + "," + 0 + ")"
        )
        .attr("class", "rect");


    }
    
    }
}
</script>