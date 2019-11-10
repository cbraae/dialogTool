

<template>
<div>

<!-- Modal asking for repetition-->
  <div class="modal modal-dialog modal-sm float-left" id="myModal" v-if="modalOpen">
    <div class="modal-dialog">
      <div class="modal-content">
        <!-- Modal Header -->
        <div class="modal-header">
          <b>Gentag?</b>
          <button type="button" class="close" @click="closeModal" data-dismiss="modal">&times;</button>
        </div>
        <!-- Modal footer -->
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" @click="openRepModal">Ja</button>
          <button type="button" class="btn btn-secondary" data-dismiss="modal" @click="closeModal">Nej</button>
        </div>
      </div>
    </div>
  </div>

<ul id="cat">
      <li v-for="item in items" :key="item.text">
          <div class="swatch" :style="{ background: item.hex}" v-bind:value="value" v-on:click="categoryClicked(item.hex)"></div> {{item.text}} <br><br>
      </li>    
</ul>
<div>
  <input v-model="categoryName" id="addCategory" placeholder="Tilføj Kategori"/> 
  <button type="button" class="btn btn-secondary" id="SaveBTN" @click="saveCat">Tilføj</button>


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
          modalOpen:false,
          categoryName: "",
          colorCounter: 0,
          colors: [ "#b8e186","#7fbc41","#4d9221","#276419", "#d9d9d9", "#bc80bd","#ccebc5", "#ffed6f"]
        };
      },
      
      methods: {
        saveCat: function(event){
          this.items.push({ text: this.categoryName, hex: this.colors[this.colorCounter]});
          this.categoryName = "";
          this.colorCounter++;
        },
        getRandomColor: function() {
          var letters = '0123456789ABCDEF';
          var color = '#';
          for (var i = 0; i < 6; i++) {
            color += letters[Math.floor(Math.random() * 16)];
          }
          return color;
        },
        closeModal: function() {
          this.modalOpen = false;
        },
        openRepModal: function() {
          this.modalOpen = false;
          this.$emit('update:repModalOpen', !this.repModalOpen);

        },
        
        categoryClicked: function(color){
         this.color = color;
          if(this.chosenRect.length > 0) {
          this.modalOpen = !this.modalOpen;
          
          

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
          
          this.$emit('input', !this.value)
          }
          this.$emit('update:color', this.color)
        }
      }
    }

</script>
