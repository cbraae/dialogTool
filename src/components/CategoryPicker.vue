

<template>
<div>
<ul id="cat">
 
    <li v-for="item in items" :key="item.text">
        <div class="swatch" :style="{ background: item.hex}" v-bind:value="value" v-on:click="categoryClicked(item.hex)"></div> {{item.text}} <br><br>
        </li>    
</ul>


<!-- Modal asking for repetition-->
  <div class="modal modal-dialog modal-sm" id="myModal" v-if="modalOpen">
  <div class="modal-dialog">
    <div class="modal-content">
      <!-- Modal Header -->
      <div class="modal-header">
        <h4 class="modal-title">Gentag?</h4>
        <button type="button" class="close" @click="closeModal" data-dismiss="modal">&times;</button>
      </div>
      <!-- Modal footer -->
      <div class="modal-footer">
        <button type="button" class="btn" @click="openRepModal">Ja</button>
         <button type="button" class="btn" data-dismiss="modal" @click="closeModal">Nej</button>
      </div>
    </div>
  </div>
</div>

</div>
</template>

<script>

import "bootstrap";
import "bootstrap/dist/css/bootstrap.min.css";
import Modal from './Modal'

    export default {
      name: "CategoryPicker",
      props: ["items", "chosenRect", "value", "catDict", "chosenDateTime","repModalOpen", "color"],
      components: {
        Modal 
      },data() {
        return {
          modalOpen:false
        };
      },
      
      methods: {
        closeModal: function() {
          this.modalOpen = false;
        },
        openRepModal: function() {
          this.modalOpen = false;
          this.$emit('update:repModalOpen', !this.repModalOpen);

        },
        
        categoryClicked: function(color){
          this.modalOpen = !this.modalOpen;
          this.color = color;
          this.chosenRect.forEach(element => {

            if($("#"+Math.floor(element[0]).toString()+Math.floor(element[1]).toString()).hasClass("selected"))
            {

            $("#"+Math.floor(element[0]).toString()+Math.floor(element[1]).toString()).removeClass("selected");
            $("#"+Math.floor(element[0]).toString()+Math.floor(element[1]).toString()).css({"fill" :color, "fill-opacity": 0.5});

            }
             
            });

          $('#repeat').hide();

         this.chosenDateTime.forEach(date => {
              this.$set(this.catDict, date, color)
            
         })

          this.$emit('update:catDict', this.catDict)
          this.$emit('update:color', this.color)
          this.$emit('input', !this.value)
        }
      }
    }

</script>