
<template>
  <div id="row"> 
    
    <!-- TOP ROW: Timeline og knap-dahsboard -->
    <div id="row" class="Upper">
     
      <div id="selector" class="left">  </div> 
        
          <div  id="categorySection" class="right">
          <p id="categoryHeader"> KATEGORIER</p>
          <p class="desc">
          Tilføj en tegning ved at: <br> 1. Trykke på en kategori <br> 2. Tegn på kalenderen ved at holde shift + venstre musetast nede.
          </p>
          <CategoryPicker :chosenRect="chosenRect" :chosenDateTime="chosenDateTime" v-model="parentValue" class="categoryPicker" :color.sync="color" :items.sync="items" :repModalOpen.sync="repModalOpen" :catDict.sync="catDict"></CategoryPicker>
        </div>
<div id="categoryOverview" class="left"> </div>
      <div class="buttonGroup left">
          <button
              id="save"
              name="save"
              v-on:click="saveDrawing"
              class="btn btn-default drawingbuttons"
            >Gem</button>
            <button
              id="cancel"
              name="cancel"
              v-on:click="cancel"
              class="btn btn-default drawingbuttons"
            >Fjern tegne-visning</button>    
            <button
              id="clear"
              name="clear"
              v-on:click="clearDrawing"
              class="btn btn-default drawingbuttons"
            >Ryd kanvas</button>
             <button
              id="undo"
              name="undo"
              v-on:click="undo"
              class="btn btn-default drawingbuttons"
            >Visk Ud</button>         
        </div>   
    <div class="tooltipz"> <p id="value">dette er en mega fed tooltip</p></div>
    </div>

      <!-- Nederste: SKEMA OG SMALL MULTIPLES -->

      <div class="rows" id="rows"><p id="noDrawings"> Ingen tegninger i den valgte periode </p></div>
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
import CategoryPicker from './CategoryPicker.vue'






export default {
  name: "CalendarWeek",
  props: ["trackingData", "parsedData"],
  components: {
    CategoryPicker
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
      currentlyShownMondays: [],
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
      items: [ 
            { text: 'Venner', hex:"#a6cee3", isSelected: false},
            { text: 'Familie', hex:"#b15928", isSelected: false},
            { text: 'Arbejde', hex:"#1f78b4", isSelected: false}
        ],
      parentValue: false,
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
    catDict:{
      handler: function(){
        if(Object.entries(this.catDict).length > 0) {
            localStorage.setItem('catDict', JSON.stringify(this.catDict));
        }
        
      },
      deep: true,
      immediate: true
    },
    displayingDrawings: {
        handler:function() {

            if(this.displayingDrawings) {
              this.showDrawingsTitle = "Skjul tidligere kommentarer"
            } else {
              this.showDrawingsTitle = "Vis tidligere kommentarer"
            }
        }
    },
      items:{
      handler: function(){
        if(Object.entries(this.items).length > 3) {
            localStorage.setItem('categories', JSON.stringify(this.items));
        }
        
      },
      deep: true,
      immediate: true
    },

    repModalOpen: {
       handler: function(){
         if(!this.repModalOpen) {
           /*this.deleteGraph();
           this.drawCalender();*/
         }
       } 
    },
    trackingData: {
      handler: function() {
      if(this.trackingData.length > 1) {
        var trackData = this.trackingData.filter(function (el) {
          return el != "";
      })

        var _this = this; 
        this.cleanData = trackData.map(function(element) { 
          
          var elem = _this.parseDate(element)
          return elem;
        });
        
            
        this.currentMonday = this.findTheMonday(this.cleanData);
        this.dict = this.getDayTimeDict();
        
        this.curr = this.getCurrentWeek(this.cleanData, this.currentMonday);
            
        this.observationsPerDay = this.getTotalForEachDay();
        this.numberOfWeeks = this.getAmountOfWeeks();

        if (localStorage.getItem('catDict')){
        this.catDict = JSON.parse(localStorage.getItem('catDict'));
        };

        if (localStorage.getItem('categories')){
        this.items = JSON.parse(localStorage.getItem('categories'));
        };

        if (localStorage.getItem('imgDict')){
        this.imgDict = JSON.parse(localStorage.getItem('imgDict'));
        };

        if (localStorage.getItem('svgDict')){
        this.svgDict = JSON.parse(localStorage.getItem('svgDict'));

        };
        if(localStorage.getItem('colorDict')){
            this.colorDict =JSON.parse(localStorage.getItem('colorDict'));
        }
        
        /*if(localStorage.getItem("categoryOverviewDict")){
          this.categoryOverviewDict = JSON.parse(localStorage.getItem('categoryOverviewDict'));
        } else {
          
        }*/

          
          this.createTimeline()


        }
      },
      deep: true,
      immediate: true
    },
    color: {
       handler: function() {
         this.enableDrawing(this.color, false);
       },
       deep: true
    }
  },
  output() {
    //return this.Chart();
  },
  methods: { 
    undo(){

      this.enableDrawing("rgba(255,255,255,1)", true)
    }, getTally(){
      var tally = [];
      var firstTime = true
      var lastDate = new Date( new Date(this.cleanData[0]))
      //lastDate.setHours(0,0,0,0)  
      var counter = 0
      var cleanData = this.cleanData;

  
    cleanData.forEach(function(line) {
              
              var date = new Date( new Date(line))

              if(lastDate.getHours()===date.getHours() && lastDate.getDate() == date.getDate()){
                counter++;
              } else { 
                
                tally.push({date:lastDate,count:counter})
                counter = 1;
                lastDate = date;
              }
             
      });
      tally.push({date:lastDate,count:counter})
      return tally;
    },
    parseDate(input) {
      if(input.toString().includes("-")){
        input+="z"
        return new Date(input)
      } else 
        return new Date(Date.UTC(
          parseInt(input.slice(0, 4), 10),
          parseInt(input.slice(4, 6), 10) - 1,
          parseInt(input.slice(6, 8), 10),
          parseInt(input.slice(9, 11), 10),
          parseInt(input.slice(11, 13), 10),
          parseInt(input.slice(13,15), 10)
        ));
    }, getTotalForEachDay(){

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

      var datesInPeriod= getDates(new Date((this.cleanData[0])), new Date( this.cleanData[this.cleanData.length-1]))
     

      var tally = [];
      var firstTime = true
      var lastDate = new Date( new Date(this.cleanData[0]))
      var counter = 0
      var cleanData = this.cleanData;
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
      var lastDate = new Date( new Date(this.cleanData[0]))
    
        for(var i=0; i < tally.length; i++){
             for(var j = 0; j < datesInPeriod.length; j++ ) {

                  if(tally[i].date.getDate() == datesInPeriod[j].date.getDate() && tally[i].date.getHours() == datesInPeriod[j].date.getHours() && tally[i].date.getMonth() == datesInPeriod[j].date.getMonth()){
                    datesInPeriod[j].count = tally[i].count
                  }
             }
        }

      return datesInPeriod;
    },
    createTimeline(){

      //Remove old stuff:
      d3.select("#selector svg").remove();

      var justInitialized = true;
      var oldSelection = "";
      var observationsPerDay = this.observationsPerDay;
      var lastClickedId = ""
      var clicked = false;
      var gridColor = this.gridColor;


      var data = this.getTally();
      
      
        function getMonday(d) {
          
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
          }

          
        function getSunday(d) {
            d = new Date(d);
            var day = d.getDay(),
            diff = d.getDate() - day
            var sunday = new Date(d.setDate(diff));
            sunday.setHours(23,59,59,59);         
            return sunday
          }
        
        
        var firstMonday = getMonday(d3.min(observationsPerDay, function(d){return d.date; }))
        var lastSunday =  getSunday(d3.max(observationsPerDay, function(d){return d.date; }))

      var xScale = d3.scaleTime()
        .domain([
          firstMonday,
         lastSunday
        ])
        .range([50, 1080])

        var y = d3
        .scaleLinear()
        .domain([24, 0])
        .range([100,0])


        var line = d3.line()
        .x(function(d) { return xScale(d.date.setHours(0,0,0,0))})
        .y(function(d) { return yScale(d.count)})

        var svg = d3.select("#selector").append("svg").attr("width", 1200).attr("height",175).attr("transform",
          "translate(0,0)");
        var myCircle = svg.append('g')
        .selectAll("dot")
        .data(data)
        .enter()
        .append("circle")
          .attr("cx", function (d) {  if(d.count > 0) return 5+xScale(new Date(d.date).setHours(0,0,0,0)); } )
          .attr("cy", function (d) {  if(d.count > 0) return y(new Date(d.date).getHours()); } )
            .attr("r", function(d, i) {
              if(d.count > 0){
                  return (Math.sqrt((d.count/2)/Math.PI)) * 2;
              } else {
                return 0; 
              }
          
        })
          .style("fill", gridColor).attr("transform",
          "translate(0,70)")

         

        if(firstMonday.getDate()+6 == lastSunday.getDate()){
            var xAxis = axisTop().scale(xScale).ticks(7).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)
            var bottomAxis =  axisBottom().scale(xScale).ticks(7).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)      
        } else {
           var xAxis = d3.axisTop().scale(xScale).ticks(d3.timeMonday).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)
           var bottomAxis =  d3.axisBottom().scale(xScale).ticks(d3.timeMonday).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)
        }



        // var xAxis = d3.axisBottom().scale(xScale).ticks(d3.timeMonday);
        var yAxis = d3.axisLeft().scale(y).tickValues([6,12,18,24]).tickFormat(function(d){return "Kl."+d; }).tickSizeOuter(0)
      
      

      svg
        .append("g")
        .attr("class", "xaxis")
        .attr(
          "transform",
          "translate(0,65)")
        .style("stroke", gridColor)
        .call(xAxis)
        .selectAll("text")
        .attr('transform', 'translate(-10,-15)rotate(45)')
        .style("fill", gridColor)
        .style("stroke", "none")

      svg
        .append("g")
        .attr("class", "bottomAxis")
        .attr(
          "transform",
          "translate(0,170)")
        .style("stroke", gridColor)
        .call(bottomAxis)
        .selectAll("text")
        .attr('transform', 'translate(-10,-15)rotate(45)')
        .style("stroke", "none")
        .style("fill","none")



      svg
        .append("g")
        .attr("class", "y")
        .attr(
          "transform",
          "translate(40,65)")
        .style("stroke", gridColor)
        .call(yAxis)
        .selectAll("text")
        .style("fill", gridColor)
        .style("stroke","none")

        
   
        svg.selectAll(".y path").style("stroke", gridColor)
        svg.selectAll(".y line").style("stroke", gridColor)
        svg.selectAll(".xaxis path").style("stroke", gridColor)
        svg.selectAll(".xaxis line").style("stroke", gridColor)
        svg.selectAll(".bottomAxis path").style("stroke", gridColor)
        svg.selectAll(".bottomAxis line").style("stroke", gridColor)
     
      var _this = this;

        var gBrushes = svg.append('g')
          .attr("class", "brushes");

        function newBrush() {
          var brush = d3.brushX()
            .extent([[50, 66], [1080, 170]])
            .on("brush", brushed)
            .on("end", brushend)

          _this.brushes.push({id: _this.brushes.length, brush: brush});
          
            d3.selectAll(".brushes")
            .on("mousedown", function() {
                if (d3.event.button === 2) { // only enable for right click
                  d3.event.stopImmediatePropagation();
                }
            })
            .on("contextmenu", function(){
              event.preventDefault();
               lastClickedId = d3.event.target.parentNode.id
               clicked = true;
                 $(".custom-menu").finish().toggle(100).
              // In the right position (the mouse)
              css({
                  top: event.pageY + "px",
                  left: event.pageX + "px"
              })
            });
            
            $(".custom-menu li").click(function(){
    
            // This is the triggered action name
            switch($(this).attr("data-action")) {
                
                // A case for each action. Your actions here
                case "first":
                  if(clicked){
                    var brush = document.getElementById(lastClickedId);
                    
                    var index = _this.brushes.indexOf(brush.id)
                    
                    if (index > -1) {
                      _this.brushes.splice(index, 1);
                    }
                    delete _this.brushImageDict[lastClickedId];
                    $("#"+lastClickedId).remove();  
                    _this.showDrawings(null, null, lastClickedId, false);
                    _this.showSmallMutiples(null, null, lastClickedId);

                    /*
                                    

                      
                    $("."+lastClickedId+"smallMultiples").remove();   
                   

                    // If all images are removed, slider should be removed too
                    if($(".smallImages").length == 0){
                    
                      
                       if($(".rows").hasClass("slick-initialized")){
                          $('.rows').slick("unslick");
                          $(".images").remove();
                        }
                    }
                    $("#noDrawings").hide();
                    _this.showDrawings(null, null, lastClickedId, false); */
                    clicked = false;
                  }
                   

                  break;
            }
          
            // Hide it AFTER the action was triggered
            $(".custom-menu").hide(100);
          });

          function brushed() {
            if (d3.event.sourceEvent && d3.event.sourceEvent.type === "zoom") return; // ignore brush-by-zoom
            var k = d3.event.selection;
            var startDate = xScale.invert(k[0])
            var endDate = xScale.invert(k[1])
            _this.showDrawings(startDate, endDate, this.id, false);
                  
          }

          function brushend() {
            
            var deleted = false;
            if(!d3.event.sourceEvent) return;
            if(d3.event.sourceEvent.shiftKey){
              var index = _this.brushes.indexOf(this.id);
              if (index > -1) {
                _this.brushes.splice(index, 1);
              }
              deleted = true;
            }

            // Figure out if our latest brush has a selection
            var lastBrushID = _this.brushes[_this.brushes.length - 1].id;
            var lastBrush = document.getElementById('brush-' + lastBrushID);
            var selection = d3.brushSelection(lastBrush);

            // If it does, that means we need another one
            if (selection && selection[0] !== selection[1]) {
              newBrush();
            }

            var k = d3.event.selection;
            var monday = getMonday(xScale.invert(k[0]))
            var sunday = getSunday(xScale.invert(k[1]))
            
            
            var mondayCords = xScale(monday)
            var sundayCords = xScale(sunday)
            
            
            if(deleted || monday > sunday){
                delete _this.brushImageDict[lastBrush]
                $(this).remove();
                 _this.showDrawings(null, null, this.id, false);
                 
                //_this.showSmallMutiples(null, null, this.id);

            } else {
                
                d3.select(this).transition().call(brush.move, [mondayCords,sundayCords])
                _this.showDrawings(monday, sunday, this.id, true);
                _this.showSmallMutiples(monday, sunday, this.id);
            }

            // Always draw brushes
            drawBrushes();

            
          }
        }

        function drawBrushes() {
          var brushSelection = gBrushes
            .selectAll('.brush')
            .data(_this.brushes, function (d){return d.id});

          // Set up new brushes
          brushSelection.enter()
            .insert("g", '.brush')
            .attr('class', 'brush')
            .attr('id', function(brush){ return "brush-" + brush.id; })
            .each(function(brushObject) {
              //call the brush
              brushObject.brush(d3.select(this));
            });

          brushSelection
            .each(function (brushObject){
              d3.select(this)
                .attr('class', 'brush')
                .selectAll('.overlay')
                .style('pointer-events', function() {
                  var brush = brushObject.brush;
                  if (brushObject.id === _this.brushes.length-1 && brush !== undefined ) {
                    return 'all';
                  } else {
                    return 'none';
                  }
                })
            })

          brushSelection.exit()
            .remove();
        }

        newBrush();
        drawBrushes();
        var sundayCords = xScale(lastSunday)
        var monday = new Date(new Date(lastSunday).setDate(lastSunday.getDate() - 6))
        monday.setHours(0,0,0,0)
        var mondayCords = xScale(monday)
        var brushWidth = sundayCords-mondayCords

       var weekBrush = d3.brushX()
        .extent([[50, 66], [1080, 170]])
        .on("end", dateBrush);


        var gBrush = svg.append("g")
        .attr("class", "weekbrush")
        .call(weekBrush)
        .call(weekBrush.move, [mondayCords, sundayCords])

        var hasBeenMoved = false;

        // A function that return TRUE or FALSE according if a dot is in the selection or not
        function isBrushed(brush_coords, cx, cy) {
            
            var x0 = brush_coords[0],
                x1 = brush_coords[1]
            return x0 <= cx && x1 >= cx   // This return TRUE or FALSE depending on if the points is in the selected area
        }


        function dateBrush() {
                  
        d3.selectAll('.weekbrush>.handle').remove();
        // removes crosshair cursor
        d3.selectAll('.weekbrush>.overlay').remove();
          if(!justInitialized) {

             if (!d3.event.sourceEvent) return;
             if (!d3.event.sourceEvent && d3.event.sourceEvent.type === "zoom") return; // ignore brush-by-zoom
          
          }
          var k = d3.event.selection;
          if(oldSelection == k[0]) return;
          var monday = getMonday(new Date(xScale.invert(k[0])))
          var sunday = new Date(new Date(monday).setDate(monday.getDate() + 6))
          sunday.setHours(23,59,59)
          var mondayCords = xScale(monday)
          var sundayCords = xScale(sunday)
           var extent = d3.event.selection
           myCircle.classed("selectedCircles", function(d){ return isBrushed(extent, xScale(new Date(d.date).setHours(0,0,0,0)),y(new Date(d.date).getHours())) } )
          

          _this.displayChosenWeek(monday,sunday);
          oldSelection = k[0]
          d3.select(".weekbrush").transition().call(weekBrush.move, [mondayCords,sundayCords])
         }

    }, setupxAxisForWeek(startDate, endDate){
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

    }, cancel(){
        $(".buttonGroup").css("visibility", "hidden");
        $("#chart").removeClass("selectedCategory")
        this.items.forEach( item => {
            this.$set(item, "isSelected", false)
          }
        )
        this.chosenColors = [];
        this.drawCategoryGrid()
        this.removeCanvasOverlay();
        //$("#drawing-area").remove();
        //this.createCanvasOverlay()
    },
    saveDrawing(){
       
        
      $(".buttonGroup").css("visibility", "hidden");
      $("#chart").removeClass("selectedCategory")
      this.items.forEach( item => {
            this.$set(item, "isSelected", false)
        }
      ) 
      
        const canvas = document.getElementById('drawing-area');
        var dataURL = canvas.toDataURL("image/png");
        var imgData = dataURL.replace(/^data:image\/(png|jpg);base64,/, "");
        if(this.imgDict[this.currentMonday]){
          this.imgDict[this.currentMonday].push(imgData);
          this.colorDict[imgData] = []
          for(var color in this.chosenColors){
              
              if(this.chosenColors[color].toString().includes("#")){
                this.colorDict[imgData].push(this.chosenColors[color])
              }
              
          }
          
         
        } else {
            this.imgDict[this.currentMonday] = []
            this.imgDict[this.currentMonday].push(imgData); 

            for(var color in this.chosenColors){
               if(this.chosenColors[color].toString().includes("#")){
                this.colorDict[imgData] = []
                this.colorDict[imgData].push(this.chosenColors[color])
              }
          }
         
        }
        
        localStorage.setItem("imgDict", JSON.stringify(this.imgDict));

        var s = new XMLSerializer().serializeToString(document.getElementById("calendar"))
        var b64 = window.btoa(s);
        this.svgDict[this.currentMonday] = b64;
        localStorage.setItem("colorDict", JSON.stringify(this.colorDict));
        localStorage.setItem("svgDict", JSON.stringify(this.svgDict));
        var numberInDict = this.findCurrentWeekNumber(this.currentMonday);

        
        this.chosenColors = [];
        this.drawCategoryGrid()
        this.removeCanvasOverlay();

        //REMOVE shown drawings if exist
        if($("#chart").hasClass("showingDrawings")){
          for(var i = 0; i < this.brushes.length-1; i++){
            $("#brush-"+i).remove();
          }
                      
          $(".custom-menu").hide(100);
          $(".images").remove();
          $("#chart").removeClass("showingDrawings") 
        }
        var currentMonth = parseInt(this.currentMonday.getMonth())+1
        var sunday = new Date(new Date(this.currentMonday).setDate(this.currentMonday.getDate()+6))
        alert("Tegningen til mandag d."+this.currentMonday.getDate() + "/" + currentMonth+"- søndag d."+sunday.getDate()+"/"+currentMonth+" er gemt.")



    },clearDrawing(){
      const canvas = document.getElementById('drawing-area');
      if(canvas !== null) {
              const context = canvas.getContext('2d');
              context.clearRect(0,0,this.calendarWidth, this.calendarHeight);
      }

    }, showSmallMutiples(startDate, endDate, id){
      if($(".rows").hasClass("slick-initialized")){
         $('.rows').slick("unslick")
       }
       $(".imgs").remove();
       $("#noDrawings").hide();
       $(".smallImages").off();

        if(Object.entries(this.imgDict).length > 0){
        var imageList = []
        var container = document.getElementById('chart')
        var smallMultiplesContainer = document.getElementById('categorySection')


        var zindex = 1;
        var overlayLocation = 30
        var width = 200;

        var rows = document.getElementById("rows")
        rows.style.width = "400px";
        var totalCounter = 0;

      for(var brush in this.brushImageDict){
      var imagesFromBrush = this.brushImageDict[brush]
        for(var i =0; i < imagesFromBrush.length; i++){
              var currentMonday = imagesFromBrush[i]
              var monday = this.currentlyShownMondays[i]

              if(column){
                    rows.appendChild(column)
              }
              
              
              var column = document.createElement('div');
              column.classList="column images imagetitles imgs "+id.toString()+ " showingDrawings "  + currentMonday.toString()

              var mondayt = new Date(monday)
              var sunday = new Date(new Date(mondayt).setDate(mondayt.getDate()+6))
              var currentMonth = parseInt(mondayt.getMonth())+1
              var title = document.createTextNode("Mandag d." + mondayt.getDate() + "/" + currentMonth + " - " + "Søndag d." + sunday.getDate() + "/" + currentMonth );
              
              //title.className = "imagetitles"

              var image = document.createElement("IMG")
              image.src = "data:image/svg+xml;base64,"+this.svgDict[monday]
              image.className = "images imgs " + id.toString()  + " " + currentMonday.toString()
              image.id = zindex.toString();
              

              //image.style.left = overlayLocation +"px"
              image.style.width = width-5 +"px";
              image.style.height = 100 +"px";
              smallMultiplesContainer.appendChild(image);
              column.style.width = 2*width-10+"px"
              column.appendChild(title);
              column.appendChild(image);
              image.style.opacity = "50%"
              
              var imageOverlay = document.createElement("IMG")
              imageOverlay.style.zIndex=zindex;
              imageOverlay.style.position = "absolute";
              imageOverlay.style.width = width-15+"px";
              imageOverlay.style.height = "100px";
            
              
              imageOverlay.src = "data:image/png;base64," + currentMonday;
              imageOverlay.className = "smallMutipless images smallImages imgs "+ id.toString()  + " " + currentMonday.toString()
              zindex +=1;
              overlayLocation+=width;
              column.appendChild(imageOverlay);
              
              totalCounter+=1; 
              
              
           }
        }
        }

      if(column){
            rows.appendChild(column)
            smallMultiplesContainer.appendChild(rows)
             }
        
          if($(".smallImages").length > 6){
              $('.rows').slick({
              speed: 800,
              infinite: true,
              vertical: true,
              slidesToShow: 3,
              verticalSwiping: true,
              arrows: true
              });
          }
        

          if($(".smallImages").length ==  0){
            $("#noDrawings").show();
          }
         
         var clickedItem = "old";
         var brushId = "old"
         var _this = this;
         
             $(".smallImages")
            .on("mousedown", function() {
                if (event.button === 2) { // only enable for right click
                  event.stopImmediatePropagation();
                }
            })
            .on("contextmenu", function(){
              event.preventDefault();
              var userHasAnswered = false;
              var classList = $(this)[0].classList;
              clickedItem = classList[classList.length-1]
              brushId = classList[classList.length-2]

                 $(".custom-menu-delete").show()
                  .css({
                  top: event.pageY + "px",
                  left: event.pageX + "px"
              })
                 $(".custom-menu-delete li").click(function(){
                   event.preventDefault();

                    switch($(this).attr("data-action")) {                
                // A case for each action. Your actions here
                case "first":  

                    if(!userHasAnswered){
                      userHasAnswered = true;
                      var result = confirm("Tegningen bliver slettet permanent. Fortsæt?");
                    }
                    
                    
                    $(".custom-menu-delete").hide(100);

                    if (result) {
                       
                        removeImage(_this.ImagePlaceInDict[clickedItem][0],_this.ImagePlaceInDict[clickedItem][1])

                        for(var i=0; i < _this.brushImageDict[brushId].length; i++ ){
                          if(_this.brushImageDict[brushId][i].toString().localeCompare(clickedItem) == 0){
                            var index = i;
                          }
                        }

                      
                      _this.brushImageDict[brushId].splice(index, 1);
                      
                      if(_this.brushImageDict[brushId].length == 0){
                        delete _this.brushImageDict[brushId]
                      }

                        _this.showSmallMutiples("", "", brushId);
                        
                        
                       
                        //$('.'+brushId.toString()).remove();
                         // If all images are removed, slider should be removed too
                        if($(".smallImages").length == 0){
                          
                          $("#"+brushId.toString()).remove();
                          if($(".rows").hasClass("slick-initialized")){
                              $('.rows').slick("unslick")
                              $("#noDrawings").hide();
                              
                            }
                        }
                        
                    }
                    
                    
                  break;
                    }
            });
           

            });


          function removeImage(monday, arrayPlace){
              _this.$delete(_this.colorDict, _this.imgDict[monday][arrayPlace]);
              _this.$delete(_this.imgDict[monday], arrayPlace);
              
            if(_this.imgDict[monday].length == 0){
                _this.$delete(_this.imgDict, monday)
             } 
             

             localStorage.setItem("imgDict", JSON.stringify(_this.imgDict));
             _this.initialiseCategoryDict();
             _this.drawCategoryGrid();
          }
      

        this.displayingDrawings = !this.displayingDrawings;
    },
    showDrawings(startDate, endDate, id, brushend){

        //Altid slet alle billeder først
        $(".bigImages").remove();
        

        /*
        if(startDate){
           $("#chart").addClass("showingDrawings");
        } else {
           $("#chart").removeClass("showingDrawings");
        } */

        var chosenDays = []
        for(var monday in this.imgDict) {
          var mday = new Date(new Date(monday))

          if(mday >= startDate && mday <= endDate){
              chosenDays.push(monday)
          }
        }

        this.ImagePlaceInDict = {}
        this.brushImageDict[id] = []
        this.currentlyShownMondays = []
        for(var i = 0; i < chosenDays.length; i++) {
            var monday = chosenDays[i]
            var chosenMonday = this.imgDict[monday]

           for(var j =0; j< chosenMonday.length; j++ ){
              var currentMonday = chosenMonday[j]
              this.ImagePlaceInDict[currentMonday] = [monday,j]
              this.currentlyShownMondays.push(monday)
              //this.currentlyShownImages.push(currentMonday)

              //Keep track of which brush holds which images
              this.brushImageDict[id].push(currentMonday)
              

           }
        }
        

        if(Object.entries(this.imgDict).length > 0){
        var imageList = []
        var container = document.getElementById('chart')
        var smallMultiplesContainer = document.getElementById('categorySection')
        var zindex = 1;
        var width = 200;

     //Tegn eller Gentegn alle billeder i dict: 
      for(var brush in this.brushImageDict){
      var imagesFromBrush = this.brushImageDict[brush]
      for(var i =0; i < imagesFromBrush.length; i++){
              currentMonday = imagesFromBrush[i]
              

              var rows = document.getElementById("rows")
              rows.style.width = "400px";
              var totalCounter = 0;

              var drawing = document.createElement("IMG")
              drawing.style.zIndex=zindex;
              drawing.style.position = "absolute";
              drawing.style.left="0px";
              drawing.style.top="0px";
              drawing.className = "bigImages images "+ id.toString()  + " "  + currentMonday.toString()
              drawing.src = "data:image/png;base64," + currentMonday;
              container.appendChild(drawing);
 
           }

        }
        }
      else {
          $(".images").remove();
          //$("#chart").removeClass("showingDrawings");
      }

      var _this = this;
        $("#chart, .bigImages")
            .on("mousedown", function() {
                if (event.button === 2) { // only enable for right click
                  event.stopImmediatePropagation();
                }
                
            })
            .on("contextmenu", function(){
              event.preventDefault();

               /* Få fat i alle brushes og slet dem */ 

                
                 $(".custom-menu").show()
                  .css({
                  top: event.pageY + "px",
                  left: event.pageX + "px"
              })
                 $(".custom-menu li").click(function(){
                   
                  switch($(this).attr("data-action")) {                
                // A case for each action. Your actions here
               
                case "first":  
                    for(var i = 0; i < _this.brushes.length-1; i++){
                       $("#brush-"+i).remove();

                    }
                    
                    $(".custom-menu").hide(100);
                    $(".images").remove();
                    $("#chart").removeClass("showingDrawings")
                   
                     // If all images are removed, slider should be removed too
                    if($(".smallImages").length == 0){
                       if($(".rows").hasClass("slick-initialized")){
                          $('.rows').slick("unslick")
                          $("#noDrawings").hide();
                        }
                    }

                  break;
                    }
                  });
            });

    
    },
    drawCalender(){
        this.renderCalendarGrid();
        //this.data = this.getDataForWeek(this.curr);
        this.drawGraphsForMonthlyData(this.data);
        //this.setClickEventHandler(); 
        this.drawCategoryGrid()
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
    }, getAmountOfWeeks(){
      var data = this.observationsPerDay
      return d3.timeSunday.count(new Date(data[0].date), new Date(data[data.length-1].date))
    },
     categoryGridCells() {
      var cellPositions = [];
      var width = 1030;
      var height = 4;
      var numberOfWeeks = this.numberOfWeeks;
      var numberOfCategories = this.items.length;
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
    },displayChosenWeek(startDate, endDate){
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

      /*$("#drawing-area").remove();
       
      this.createCanvasOverlay()*/

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
      this.clearDrawing();
    }, drawCategoryGrid(){

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
    }, findCurrentWeekNumber(chosenDate){
        
          var monday = this.getMonday(chosenDate);
          return d3.timeMonday.count(new Date(this.cleanData[0]), monday)

    }, getMonday(d) {
            d = new Date(d);
            var day = d.getDay(),
                diff = d.getDate() - day + (day == 0 ? -6:1); // adjust when day is sunday
            var monday = new Date(d.setDate(diff));
            monday.setHours(0,0,0,0);
            return monday

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

        
    }, removeCanvasOverlay(){
      $("#drawing-area").remove();
      this.canvasExists = false;
    },
    createCanvasOverlay() {
      var canvasContainer = document.getElementById('chart');
      document.body.appendChild(canvasContainer);
      var myCanvas = document.createElement('canvas');
      myCanvas.id = "drawing-area";
      myCanvas.style.width = this.calendarWidth;
      myCanvas.style.height = this.calendarHeight;
      myCanvas.style.zIndex="1080";
      myCanvas.style.left="0px";
      myCanvas.style.top="0px";
      myCanvas.width=this.calendarWidth;
      myCanvas.height=this.calendarHeight;
      myCanvas.style.overflow = 'visible';
      myCanvas.style.position = 'absolute';
      canvasContainer.appendChild(myCanvas);

      var _this = this;
       $("#drawing-area")
            .on("mousedown", function() {
                if (event.button === 2) { // only enable for right click
                  event.stopImmediatePropagation();
                }
                
            })
            .on("contextmenu", function(){
              event.preventDefault();
               /* Få fat i alle brushes og slet dem */ 
              
                
                 $(".custom-menu").show()
                  .css({
                  top: event.pageY + "px",
                  left: event.pageX + "px",
              }).css("z-index", 10000);
                 $(".custom-menu li").click(function(){
                    
                   
                  switch($(this).attr("data-action")) {                
                // A case for each action. Your actions here
               
                case "first":  
                    
                    $(".custom-menu").hide(100);
                    _this.cancel();
                    _this.items.forEach( item => {
                        _this.$set(item, "isSelected", false)
                      }
                    )
                     // If all images are removed, slider should be removed too
                    if($(".smallMutiples").length == 0){
                       if($(".rows").hasClass("slick-initialized")){
                          $('.rows').slick("unslick")
                        }
                    } 

                  break;
                    }
                  });
            });

 }, initialiseCategoryDict(){
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
                dict[numberInDict][catNumber] = color;
            }
          }

        }
        
      }
      

      //localStorage.setItem("categoryOverviewDict", JSON.stringify(dict));

    return dict;
 }, initialiseCategoryNumberDict(){
    var dict = {};
    for(var key in this.items){
     if(this.items[key]){
        var color = this.items[key].hex;
        dict[color] = key; 
     }

    }
    return dict;
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

    }, enableDrawing(color, eraser) {

       if(!this.canvasExists){
         this.createCanvasOverlay()
         this.canvasExists = true;
       } 

      const canvas = document.getElementById('drawing-area');
      const canvasContext = canvas.getContext('2d');
      const state = {
        mousedown: false
      };

    // ===================
    // == Configuration ==
    // ===================
    var lineWidth = 1;
    const halfLineWidth = lineWidth / 2;
    const fillStyle = color;
    const strokeStyle = color;
    const shadowColor = '#333';
    canvasContext.globalCompositeOperation="source-over";
    

      if(eraser){
        lineWidth= 100;
        canvasContext.globalCompositeOperation = "destination-out";  
      } else{
        this.chosenColors.push(color)
      }


    // =====================
    // == Event Listeners ==
    // =====================
    canvas.addEventListener('mousedown', handleWritingStart);
    canvas.addEventListener('mousemove', handleWritingInProgress);
    canvas.addEventListener('mouseup', handleDrawingEnd);
    canvas.addEventListener('mouseout', handleDrawingEnd);

    canvas.addEventListener('touchstart', handleWritingStart);
    canvas.addEventListener('touchmove', handleWritingInProgress);
    canvas.addEventListener('touchend', handleDrawingEnd);

    var cntrlIsPressed = false;
    $(document).keydown(function(event){
        if(event.which=="16")
         cntrlIsPressed = true;
    });
        
    $(document).keyup(function(){
      cntrlIsPressed = false;
    });

    // ====================
    // == Event Handlers ==
    // ====================
    function handleWritingStart(event) {
      $(".buttonGroup").css("visibility", "visible");
      $("#chart").addClass("selectedCategory")

      

      event.preventDefault();

      const mousePos = getMosuePositionOnCanvas(event);
      
      canvasContext.beginPath();

      canvasContext.moveTo(mousePos.x, mousePos.y);

      canvasContext.lineWidth = lineWidth;
      canvasContext.strokeStyle = strokeStyle;
      canvasContext.shadowColor = null;

      
      canvasContext.fill();
      
      state.mousedown = true;
    }

    function handleWritingInProgress(event) {
      event.preventDefault();
      
      if (state.mousedown && cntrlIsPressed) {
        $("#chart").addClass("drawingCursor")
        const mousePos = getMosuePositionOnCanvas(event);

        canvasContext.lineTo(mousePos.x, mousePos.y);
        canvasContext.stroke();

      } else {
        $("#chart").removeClass("drawingCursor")
      }
    }

    function handleDrawingEnd(event) {
      event.preventDefault();

      if (state.mousedown) {
        canvasContext.shadowColor = shadowColor;
        canvasContext.stroke();
      }
      
      state.mousedown = false;
    }

    function handleClearButtonClick(event) {

      event.preventDefault();
      
      clearCanvas();
    }

    // ======================
    // == Helper Functions ==
    // ======================
    function getMosuePositionOnCanvas(event) {
      const clientX = event.clientX || event.touches[0].clientX;
      const clientY = event.clientY || event.touches[0].clientY;
      const { offsetLeft, offsetTop } = event.target;
      const canvasX = clientX - offsetLeft -18;
      const canvasY = clientY-294;

      return { x: canvasX, y: canvasY };
    }


        },
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

