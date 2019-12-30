
<template>
    <div id="selector" class="left">  </div> 
</template>

<script>
import * as d3 from 'd3'

export default {
  name: "CalendarWeek",
  props: ["trackingData"],
  data() {
    return {
        svg: "",
        brushes: [],
        gridColor: "lightgrey", 
        firstMonday: "",
        lastSunday: "",
        data: {},
        xScale: "",
        yScale: "",
        weekbrush: "",
        justInitialized: true,
        myCircle: "",
        oldSelection: "",
        gBrushes: "",
        lastClickedId: "",
        clicked: false
    }
  },
  watch: {
      trackingData: {
      handler: function() {
        this.data = this.getTally()
        this.firstMonday = this.getMonday(d3.min(this.data, function(d){return d.date; }))
        this.lastSunday =  this.getSunday(d3.max(this.data, function(d){return d.date; }))
        this.xScale = this.getxScale();
        this.yScale = this.getyScale();
        this.drawTimeline();
      }
    }
  },
  methods: {
      getxScale(){
          var _this = this;

        return d3.scaleTime()
                .domain([
                _this.firstMonday,
                _this.lastSunday
                ])
                .range([50, 1080])
      },
      getyScale(){
         return d3
        .scaleLinear()
        .domain([24, 0])
        .range([100,0])
      },
    getTally() {
      var tally = [];
      var firstTime = true
      var lastDate = new Date( new Date(this.trackingData[0]))
      var counter = 0
      var cleanData = this.trackingData;

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
    drawSVGs(){
        d3.select("#selector svg").remove();

    
      
      var gridColor = this.gridColor;
      var data = this.data;
      var firstMonday = this.firstMonday
      var lastSunday = this.lastSunday
      var xScale = this.xScale
      var y = this.yScale       

      

        

        var line = d3.line()
        .x(function(d) { return xScale(d.date.setHours(0,0,0,0))})
        .y(function(d) { return yScale(d.count)})

        if(firstMonday.getDate()+6 == lastSunday.getDate()){
            var xAxis = d3.axisTop().scale(xScale).ticks(7).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)
            var bottomAxis =  d3.axisBottom().scale(xScale).ticks(7).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)      
        } else {
           var xAxis = d3.axisTop().scale(xScale).ticks(d3.timeMonday).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)
           var bottomAxis =  d3.axisBottom().scale(xScale).ticks(d3.timeMonday).tickFormat(d3.timeFormat("%a %d %b")).tickSizeOuter(0)
        }
        var yAxis = d3.axisLeft().scale(y).tickValues([6,12,18,24]).tickFormat(function(d){return "Kl."+d; }).tickSizeOuter(0)


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

        var gBrushes = svg.append('g')
          .attr("class", "brushes");

      this.gBrushes = gBrushes;
      this.svg = svg;
      this.myCircle = myCircle;
    }, 
    getMonday(d){
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
    getSunday(d){
         d = new Date(d);
            var day = d.getDay(),
            diff = d.getDate() - day
            var sunday = new Date(d.setDate(diff));
            sunday.setHours(23,59,59,59);         
        return sunday
    }, 
    addBrushes(){
         var _this = this;
         var xScale = this.xScale
         var lastSunday = this.lastSunday

        /*var gBrushes = this.svg.append('g')
          .attr("class", "brushes");*/
        _this.newBrush();
        _this.drawBrushes();
     

        var sundayCords = xScale(lastSunday)
        var monday = new Date(new Date(lastSunday).setDate(lastSunday.getDate() - 6))
        monday.setHours(0,0,0,0)
        var mondayCords = xScale(monday)
        var brushWidth = sundayCords-mondayCords

       var weekBrush = d3.brushX()
        .extent([[50, 66], [1080, 170]])
        .on("end", _this.dateBrush);

        this.weekBrush = weekBrush;

        var gBrush = this.svg.append("g")
        .attr("class", "weekbrush")
        .call(weekBrush)
        .call(weekBrush.move, [mondayCords, sundayCords])

        var hasBeenMoved = false;
        



    }, 
    newBrush() {
        var _this = this;
        
          var brush = d3.brushX()
            .extent([[50, 66], [1080, 170]])
            .on("brush", _this.brushed)
            .on("end", _this.brushend)

          _this.brushes.push({id: _this.brushes.length, brush: brush});

          
          d3.selectAll(".brushes")
            .on("mousedown", function() {
                if (d3.event.button === 2) { // only enable for right click
                  d3.event.stopImmediatePropagation();
                }
            })
            .on("contextmenu", function(){
              event.preventDefault();
               _this.lastClickedId = d3.event.target.parentNode.id
               _this.clicked = true;
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
                  if(_this.clicked){
                    var brush = document.getElementById(_this.lastClickedId);
                    
                    var index = _this.brushes.indexOf(brush.id)
                    
                    if (index > -1) {
                      _this.brushes.splice(index, 1);
                    }
                    //delete _this.brushImageDict[_this.lastClickedId];
                    //delete _this.currentlyShownMondays[_this.lastClickedId]
                    $("#"+_this.lastClickedId).remove();  
                    //_this.showDrawings(null, null, _this.lastClickedId, false);
                    //_this.showSmallMutiples(null, null, _this.lastClickedId);

                    _this.clicked = false;
                  }
                   

                  break;
            }
          
            // Hide it AFTER the action was triggered
            $(".custom-menu").hide(100);
          });
         
    },
    brushed() {
            var _this = this;

            if (d3.event.sourceEvent && d3.event.sourceEvent.type === "zoom") return; // ignore brush-by-zoom
            var k = d3.event.selection;
            var startDate = _this.xScale.invert(k[0])
            var endDate = _this.xScale.invert(k[1])
            //_this.showDrawings(startDate, endDate, this.id, false);
    }, 
    brushend(target) {
            var deleted = false;
            var _this = this; 

            if(!d3.event.sourceEvent) return;

            // Figure out if our latest brush has a selection
            var lastBrushID = _this.brushes[_this.brushes.length - 1].id;
            var lastBrush = document.getElementById('brush-' + lastBrushID);
            var selection = d3.brushSelection(lastBrush);
            


            // If it does, that means we need another one
            if (selection && selection[0] !== selection[1]) {
              _this.newBrush();
            }

            var k = d3.event.selection;
            var monday = _this.getMonday(_this.xScale.invert(k[0]))
            var sunday = _this.getSunday(_this.xScale.invert(k[1]))
            
            
            var mondayCords = _this.xScale(monday)
            var sundayCords = _this.xScale(sunday)
            
            
            if(deleted || monday > sunday){
                //delete _this.brushImageDict[lastBrush]
                //delete _this.currentlyShownMondays[lastBrush]
                $(this).remove();
                 //_this.showDrawings(null, null, this.id, false);
                 
                //_this.showSmallMutiples(null, null, this.id);

            } else {
                var brush = _this.brushes[_this.brushes.length - 1].brush;
                 d3.select("#brush-"+target.id).transition().call(brush.move, [mondayCords,sundayCords])
                //_this.showDrawings(monday, sunday, this.id, true);
                //_this.showSmallMutiples(monday, sunday, this.id);
            }
            
            // Always draw brushes
            _this.drawBrushes();

            
        }, 
    drawBrushes() {
        
            var _this = this;
            //var gBrushes = d3.selectAll(".brushes")

            var brushSelection = this.gBrushes
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
        },
    isBrushed(brush_coords, cx, cy) {
            
            var x0 = brush_coords[0],
                x1 = brush_coords[1]
            return x0 <= cx && x1 >= cx   
        }, 
    dateBrush() {
            //var justInitialized = true;
            
            var _this = this;
            

            d3.selectAll('.weekbrush>.handle').remove();
            // removes crosshair cursor
            d3.selectAll('.weekbrush>.overlay').remove();
            if(!_this.justInitialized) {
                
                if (!d3.event.sourceEvent) return;
                if (!d3.event.sourceEvent && d3.event.sourceEvent.type === "zoom") return; // ignore brush-by-zoom
            }
            var k = d3.event.selection;
            if(_this.oldSelection == k[0]) return;
            var monday = _this.getMonday(new Date(_this.xScale.invert(k[0])))
            var sunday = new Date(new Date(monday).setDate(monday.getDate() + 6))
            sunday.setHours(23,59,59)
            var mondayCords = _this.xScale(monday)
            var sundayCords = _this.xScale(sunday)
            var extent = d3.event.selection;
            var myCircle = _this.myCircle;
            
            myCircle.classed("selectedCircles", function(d){ return _this.isBrushed(extent, _this.xScale(new Date(d.date).setHours(0,0,0,0)),_this.yScale(new Date(d.date).getHours())) } )
            

            //_this.displayChosenWeek(monday,sunday);
            _this.oldSelection = k[0]
            var weekBrush = _this.weekBrush;
            
            d3.select(".weekbrush").transition().call(weekBrush.move, [mondayCords,sundayCords])
            
    },
    drawTimeline(){
        this.drawSVGs();
        this.addBrushes();
        this.drawBrushes();
    }
    } 
}


</script>