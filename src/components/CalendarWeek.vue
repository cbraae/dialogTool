
<template>
  <div id="row"> 
      <Modal ref="modal" v-model="repModalOpen" :catDict.sync="catDict" :endTime="endTime" :startTime="startTime" :firstChosenDay="firstChosenDay" :lastChosenDay="lastChosenDay" :color="color" :chosenDateTime="chosenDateTime"> </Modal>
    <!-- TOP ROW: Timeline og knap-dahsboard -->
    <div id="row">
     
      <div id="selector" class="col-8">  </div> 
        
        <div  id="categorySection" class="col-4">
        <p id="categoryHeader"> Kategorier</p>
        <p>
         Tilføj en tegning ved at: <br> 1. Trykke på en kategori <br> 2. Tegn på kalenderen ved at holde venstre musetast + shift nede.
        </p>
        <CategoryPicker :chosenRect="chosenRect" :chosenDateTime="chosenDateTime" v-model="parentValue" :color.sync="color" :items.sync="items" :repModalOpen.sync="repModalOpen" :catDict.sync="catDict"></CategoryPicker>
      </div>
      </div>
    
    <!-- MidterRække: Kategori-overview -->
    <div class="row">
      <div id="categoryOverview" class="col-8"> </div>
      <div class="col-4 align-self-end buttonGroup">
          <button
              id="save"
              name="save"
              v-on:click="saveDrawing"
              class="btn btn-default drawingbuttons"
            >GEM</button>
            <button
              id="clear"
              name="clear"
              v-on:click="clearDrawing"
              class="btn btn-default drawingbuttons"
            >RYD</button>
             <button
              id="undo"
              name="undo"
              v-on:click="undo"
              class="btn btn-default drawingbuttons"
            >Visk Ud</button>         
        </div>
    </div>
    <div class="rows" id="rows"/>
    <!-- Nederste: Kategori-overview -->
    <!-- Går i stykker hvis man pakker det ind i en row  -->
      <div id="chart"></div>
      
     <ul class='custom-menu'>
      <li data-action="first">Slet</li>
    </ul>
  </div>

  
</template>

<script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>

<script>
import * as d3 from 'd3'
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
      displayingDrawings:false,
      calendarWidth: 1100,
      calendarHeight: 450,
      gridXTranslation: 55,
      gridYTranslation: 40,
      currentMonth: new Date().getMonth(),
      currentMonday: new Date(),
      currentSunday: new Date(),
      showDrawingsTitle: "Vis tidligere kommentarer",
      numberOfWeeks: 0,
      gridColor: "lightgrey",
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
            { text: 'Venner', hex:"#8e0152", isSelected: false},
            { text: 'Familie', hex:"#c51b7d", isSelected: false},
            { text: 'Arbejde', hex:"#de77ae", isSelected: false}
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
            this.cleanData = this.trackingData.filter(function (el) {
              return el != "";
            })

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
       }
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
      return tally;
    },
    getTotalForEachDay(){

      Date.prototype.addDays = function(days) {
          var date = new Date(this.valueOf());
          date.setDate(date.getDate() + days);
          //date.setHours(0,0,0,0)
          return date;
      }

      function getDates(startDate, stopDate) {
          var sunday = new Date(stopDate.setDate(stopDate.getDate() + (1 + 6 - stopDate.getDay()) % 6))
          
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

      var datesInPeriod= getDates(new Date(new Date(this.cleanData[0])) ,new Date( new Date(this.cleanData[this.cleanData.length-1])))
   

      var tally = [];
      var firstTime = true
      var lastDate = new Date( new Date(this.cleanData[0]))
      //lastDate.setHours(0,0,0,0)  
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
        .range([50, 1050])

        var y = d3
        .scaleLinear()
        .domain([0, 24])
        .range([100,0])


        var line = d3.line()
        .x(function(d) { return xScale(d.date.setHours(0,0,0,0))})
        .y(function(d) { return yScale(d.count)})

        var svg = d3.select("#selector").append("svg").attr("width", 1200).attr("height",250).attr("transform",
          "translate(0,10)");
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
          "translate(0,10)");

        if(firstMonday.getDate()+6 == lastSunday.getDate()){
            var xAxis = d3.axisBottom().scale(xScale).ticks(7).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)
          
        } else {
           var xAxis = d3.axisBottom().scale(xScale).ticks(d3.timeMonday).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)
        }


        // var xAxis = d3.axisBottom().scale(xScale).ticks(d3.timeMonday);
        var yAxis = d3.axisLeft().scale(y).tickValues([6,12,18,24]).tickFormat(function(d){return "Kl."+d; }).tickSizeOuter(0)
      
      

      svg
        .append("g")
        .attr("class", "xaxis")
        .attr(
          "transform",
          "translate(0,"+110+")")
        .style("stroke", gridColor)
        .call(xAxis)
        .selectAll("text")
        .attr('transform', 'translate(13,35)rotate(90)')
        .style("fill", gridColor)
        .style("stroke", "none")

      svg
        .append("g")
        .attr("class", "y")
        .attr(
          "transform",
          "translate(40,10)")
        .style("stroke", gridColor)
        .call(yAxis)
        .selectAll("text")
        .style("fill", gridColor)
        .style("stroke", "none")
        
   
        svg.selectAll(".y path").style("stroke", gridColor)
        svg.selectAll(".y line").style("stroke", gridColor)
        svg.selectAll(".xaxis path").style("stroke", gridColor)
        svg.selectAll(".xaxis line").style("stroke", gridColor)
     
      var _this = this;

        var gBrushes = svg.append('g')
          .attr("class", "brushes");

        var brushes = [];

        function newBrush() {
          var brush = d3.brushX()
            .extent([[50, 2], [1050, 110]])
            .on("brush", brushed)
            .on("end", brushend)

          brushes.push({id: brushes.length, brush: brush});
          
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
                    var index = brushes.indexOf(brush.id)
                   
                    if (index > -1) {
                      brushes.splice(index, 1);
                    }
                    $("#"+lastClickedId).remove();    
                       
                    _this.showDrawings(null, null, lastClickedId, false);
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
              var index = brushes.indexOf(this.id);
              if (index > -1) {
                brushes.splice(index, 1);
              }
              deleted = true;
            }

            // Figure out if our latest brush has a selection
            var lastBrushID = brushes[brushes.length - 1].id;
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
            
            if(deleted){
                $(this).remove();
                 _this.showDrawings(null, null, this.id, false);
            } else {
                d3.select(this).transition().call(brush.move, [mondayCords,sundayCords])
                _this.showDrawings(monday, sunday, this.id, true);
            }

            // Always draw brushes
            drawBrushes();

            
          }
        }

        function drawBrushes() {
          var brushSelection = gBrushes
            .selectAll('.brush')
            .data(brushes, function (d){return d.id});

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
                  if (brushObject.id === brushes.length-1 && brush !== undefined ) {
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
        .extent([[50, 0], [1050, 110]])
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
        .style("fill", gridColor)

        this.calendarChart.selectAll(".xaxis path").style("stroke", gridColor)
        this.calendarChart.selectAll(".xaxis line").style("stroke", gridColor)

    }, 
    saveDrawing(){
      $(".buttonGroup").hide();
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
        //var chosenColors = this.chosenColors;
        //this.initialiseCategoryDict();
        /*for(var i = 0; i < chosenColors.length; i++){
            var color = chosenColors[i];
            var catNumber = this.categoryNumberDict[color]
            this.categoryOverviewDict[numberInDict][catNumber] = color;
        }*/
        
        this.chosenColors = [];

        
        this.drawCategoryGrid()
        $("#drawing-area").remove();
        this.createCanvasOverlay()


    },clearDrawing(){
      const canvas = document.getElementById('drawing-area');
      if(canvas !== null) {
              const context = canvas.getContext('2d');
              context.clearRect(0,0,this.calendarWidth, this.calendarHeight);
      }

    },
    showDrawings(startDate, endDate, id, brushend){

        //$(".images").remove();
        $("."+id).remove();
       
        if(startDate){
           $("#chart").addClass("showingDrawings");
        } else {
           $("#chart").removeClass("showingDrawings");
        }

        var chosenDays = []
        for(var monday in this.imgDict) {
          var mday = new Date(new Date(monday))

          if(mday >= startDate && mday <= endDate){
                  chosenDays.push(monday)
          }
        }

        if(Object.entries(this.imgDict).length > 0) {
        var imageList = []
        var container = document.getElementById('chart')
        var smallMultiplesContainer = document.getElementById('categorySection')


        var zindex = 1;
        var overlayLocation = 30
        var width = 200;
        var ImagePlaceInDict = {}

          var rows = document.getElementById("rows")
          rows.style.width = "400px";
          var totalCounter = 0;

                

        for(var i = 0; i < chosenDays.length; i++) {
            var monday = chosenDays[i]
            var chosenMonday = this.imgDict[monday]

           for(var j =0; j< chosenMonday.length; j++ ){
              var currentMonday = chosenMonday[j]
              ImagePlaceInDict[currentMonday] = [monday,j]
            
              var drawing = document.createElement("IMG")
              drawing.style.zIndex=zindex;
              drawing.style.position = "absolute";
              drawing.style.left="60px";
              drawing.style.top="0px";
              drawing.className = "images "+ id.toString() + " " + id.toString()+j.toString()  + " "  + currentMonday.toString()
              drawing.src = "data:image/png;base64," + currentMonday;
              container.appendChild(drawing);

              if(totalCounter % 1==0 ){
                if(column){
                    rows.appendChild(column)
                    
                
                
              }
              var column = document.createElement('div');
                column.className="column" 

              var image = document.createElement("IMG")
              image.src = "data:image/svg+xml;base64,"+this.svgDict[monday]
              image.className = "images " + id.toString() + " " + id.toString()+j.toString()  + " "  + currentMonday.toString()
              image.id = zindex.toString();
              //image.style.left = overlayLocation +"px"
              image.style.width = width +"px";
              image.style.height = 100 +"px";
              smallMultiplesContainer.appendChild(image);
              column.style.width = 2*width+"px"
              
              column.appendChild(image);
              image.style.opacity = "50%"
              
              var imageOverlay = document.createElement("IMG")
              imageOverlay.style.zIndex=zindex;
              imageOverlay.style.position = "absolute";
              imageOverlay.style.width = width+"px";
              imageOverlay.style.height = "100px";
              imageOverlay.src = "data:image/png;base64," + currentMonday;
              imageOverlay.className = "smallMutipless images "+ id.toString() + " " + id.toString()+j.toString()  + " "  + currentMonday.toString()
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
          
        
        
        
      } else {
          $(".images").remove();
          //$("#chart").removeClass("showingDrawings");
      }
         
         var clickedItem = "old";
         var brushId = "old"
             $(".images")
            .on("mousedown", function() {
                if (event.button === 2) { // only enable for right click
                  event.stopImmediatePropagation();
                }
            })
            .on("contextmenu", function(){
              event.preventDefault();
              var classList = $(this)[0].classList;
              clickedItem = classList[classList.length-1]
              brushId = classList[classList.length-2]

                 $(".custom-menu").finish().toggle(100)
                  .css({
                  top: event.pageY + "px",
                  left: event.pageX + "px"
              })
                 $(".custom-menu li").click(function(){
                    switch($(this).attr("data-action")) {                
                // A case for each action. Your actions here
                case "first":  
                    $('.'+brushId.toString()).remove();
                    removeImage(ImagePlaceInDict[clickedItem][0],ImagePlaceInDict[clickedItem][1])
                  break;
  
            // Hide it AFTER the action was triggered
            $(".custom-menu").hide(100);
                    }
                 });

            });


          var _this = this;

          function removeImage(monday, arrayPlace){
              _this.$delete(_this.colorDict, _this.imgDict[monday][arrayPlace]);
              _this.$delete(_this.imgDict[monday], arrayPlace);
              
             localStorage.setItem("imgDict", JSON.stringify(_this.imgDict));
             _this.initialiseCategoryDict();
             _this.drawCategoryGrid();
          }
      

        this.displayingDrawings = !this.displayingDrawings;
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
      var width = 1000;
      var height = 2;
      var numberOfWeeks = this.numberOfWeeks;
      var numberOfCategories = this.items.length;
      var categoryDict = this.categoryOverviewDict;

      for (var y = 0; y < numberOfCategories; y++) {
        
        for (var x = 0; x < numberOfWeeks; x++) {
            
            cellPositions.push([
            x * (width/ numberOfWeeks),
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
      var dto = new Date(data[data.length - 1].split(";")[0]);
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

    $("#drawing-area").remove();
       
      this.createCanvasOverlay()

    },
    getCurrentWeek(data, monday) {
      var currentWeek = [];
      var stopFlag = false;
      var offset = 0;
      var currentMonday = monday;
      var currentSunday = -1;

      
      for (var i = 0; i < data.length; i++) {
        var timeStamp = data[i].split(";")[0];
        var dt = new Date(timeStamp);
        if (dt >= monday) {
          
          var diff = dt.getDate() - 1
          dt.setDate
          

            var currentSunday = new Date(
              new Date(currentMonday).setDate(currentMonday.getDate() + 7)
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
        var cellWidth = 1000/numberOfWeeks;
        var numberOfCategories = this.items.length;
        var cellHeight = 60/numberOfCategories;      

      var gridXTranslation = this.gridXTranslation;
      var gridYTranslation = this.gridYTranslation;

      var categoryHeight = 2 * numberOfCategories; 

      var categoryOverview = d3
        .select("#categoryOverview")
        .append("svg")
        .attr("class", "categories")
        .attr("id", "category")
        .attr("width", 1000+45)
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
              gridYTranslation - 8 +
              cellPositions[currentDataIndex][1] -
              20) +
            ")"
          );
        });
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
        .attr("width", calendarWidth + gridXTranslation)
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
        .style("fill", function(d){
          return d[2];
        })
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
      $(".buttonGroup").show();
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
      const canvasX = clientX - offsetLeft -5;
      const canvasY = clientY-282;

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

