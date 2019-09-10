
<template>
    <div id="container">
        
    </div>
</template>


<script> 
import * as d3 from 'd3'
import moment from 'moment';

export default {
  name: 'Chart',
  props: ['trackingData', 'parsedData'],
  data() {
      return {
          height: 600,
          width: 400
      };
  }, created() {
        this.colourScale = d3.scaleOrdinal().range(["#5EAFC6", "FE9922", "#92C464", "#75739F"]);
  },
  watch: {
  	trackingData: {
      handler: function() {
          
          var cleanedData = this.trackingData.map(item => (item[this.trackingData.columns[0]]));
          cleanedData.removeIf( function(item, idx){
              return item == ";";
          })
          drawCalendar();  
         
      },
      deep: true,
      immediate: true
    }
  },output() {
      return this.Chart();
    }
}

function drawCalendar(){
var margin = {top: 40, right: 40, bottom: 40, left: 40},
    width = 960,
    height = 500;

var x = d3.scaleTime()
.domain([new Date(2013, 0, 1), new Date(2014, 0, 1)])
.range([0, width])

var y = d3.scaleTime()
.domain([new Date(2013,0,1), new Date(2013, 0,31)])
.range([0, height])

var xAxis = d3.axisTop()
.scale(x)

var yAxis = d3.axisLeft()
.scale(y)
.ticks(d3.timeDay.every(1))
.tickFormat(d3.timeFormat('%e'))

 var svg = d3.select("#container").append("svg")
 .attr("width", width + margin.left + margin.right)
.attr("height", height + margin.top + margin.bottom)
 .append("g")
.attr("transform", "translate(" + margin.left + "," + margin.top + ")");

function makeXaxis() {
    return d3.axisTop()
    .scale(x)
}

function makeYaxis() {
    return d3.axisLeft()
    .scale(y)
    .ticks(d3.timeDay.every(1))
    .tickFormat(d3.timeFormat('%e'))
 }

 svg.append("g")
  .attr("class", "x axis")
  // .attr("transform", "translate(0," + height + ")")
  .call(xAxis);

svg.append("g")
.attr("class", "y axis")
.attr("transform", "translate(0,0)")
.call(yAxis);

 svg.append("g")
 .attr("class", "grid")
.call(makeYaxis()
  .tickSize(-width, 0, 0)
  .tickFormat("")
  .tickPadding(8)
  )

svg.append("g")
.attr("class", "grid")
.call(makeXaxis()
  .tickSize(-height, 0, 0)
  .tickFormat("")
  )
}




function parseTime (timeStamp, offset) {
  const timeString = timeStamp.split('T')[1].slice(0, -1);
  const value = offset || (timeStamp.split(';')[1]).split(';')[0];
  return moment.utc(`${timeString}`, 'HH:mm:ss').utcOffset(value).format('HH:mm:ss');
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