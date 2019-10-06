<template>
      <div class="modal" v-show="value">
        <div class="modal-content">
          <div class="modal-header">Opret Gentagelse</div>
            <div><p class="repChooser prop"> gentagelse: </p>
            <select class="repChooser" v-model="selected">
              <option value="day">Hver dag</option>
              <option value="week">Hver Uge</option>
              <option value="month">Hver måned</option>
            </select> 
          </div>
        <div> 
           <div><p class="repChooser prop"> Fra klokken </p> </div>
           <div><input type="time" v-model="timeStart"></div>   
        </div>
          <div> 
           <div><p class="repChooser prop"> Til klokken </p> </div>
           <div><input type="time" v-model="timeEnd" ></div>   
        </div>
        <div> 
           <div><p class="repChooser prop"> Gentagelse start: </p> </div>
           <div><datetime v-model="dateStart"></datetime> </div>    
        </div>
        <div> 
           <div><p class="repChooser prop"> Gentagelse slut: </p> </div>
           <div><datetime v-model="dateEnd"></datetime>  </div>   
        </div>
        <div>
          <button @click.prevent="close" class="mt-3 border-b border-teal font-semibold repChooser prop">Fortryd</button>
          <button @click="save" class="mt-3 border-b border-teal font-semibold repChooser">Gem</button>
        </div>
        </div>
      </div>
</template>


<script>

import Vue from 'vue'
import Datetime from 'vue-datetime'
// You need a specific loader for CSS files
import 'vue-datetime/dist/vue-datetime.css'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'

window.$ = require('jquery')
window.JQuery = require('jquery')
Vue.use(Datetime)

export default {
    name: "Modal", 
    props: ["value", "chosenRect", "catDict", "color", "lastChosenDay", "firstChosenDay", "startTime", "endTime"],
    watch: {
      lastChosenDay:{
        handler: function(){     
          this.dateEnd = this.lastChosenDay;
        }, 
      },
      firstChosenDay:{
        handler: function(){     
        this.dateStart = this.firstChosenDay;
          }
      },
      startTime:{
        handler: function(){     
         
        this.timeStart = this.startTime;
        
          }
      },
      endTime:{
        handler: function(){     
        this.timeEnd = this.endTime;
          }
      }

      
    },
     methods: {
     close() {
         this.$emit("input", !this.value);
        },
        save() {
            this.$emit("input", !this.value);
            var dates = this.getDates(this.dateStart, this.dateEnd);
            var hours = this.getHours(this.timeStart, this.timeEnd);

            dates.forEach( date => {
              hours.forEach ( time => {
                this.$set(this.catDict, [date, time], this.color)
              })
            })
        },
        getDates(startDate, stopDate){
          var dateArray = new Array();
          var currentDate = new Date(startDate);
          stopDate = new Date(stopDate);
          currentDate.setHours(0,0,0,0);
          stopDate.setHours(0,0,0,0);
          while (currentDate <= stopDate) {
            dateArray.push(new Date (currentDate));
            currentDate = new Date(currentDate.setDate(currentDate.getDate() + 1))
            currentDate.setHours(0,0,0,0)
          }
            return dateArray;
      }, getHours(timeStart, timeEnd){
          var hoursArray = new Array();
          
          var currentTime = parseInt(timeStart.split(":")[0]);
          var endTime = parseInt(timeEnd.split(":")[0]);

           while (currentTime <= endTime) {
            hoursArray.push(currentTime);
            currentTime += 1;
          }
          return hoursArray;
      }
     },
     data(){
    return {
            dateStart: "",
            dateEnd:"", 
            timeStart: "",
            timeEnd:"",
            selected: "week",
            
            items: [
            { text: 'Venner', hex:"#F4D03F"},
            { text: 'Familie', hex:"#229954"},
            { text: 'Arbejde', hex:"#9B59B6"}
        ]
        }
     }
};

</script>

