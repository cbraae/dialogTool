
<template>
    <div></div>
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

         
      },
      deep: true,
      immediate: true
    }
  },
  
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