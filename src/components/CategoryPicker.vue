

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

<div id="cat">

  <ul :style="gridStyle" class="card-list">
          <li v-for="(item) in items" :key="item.text" class="card-item" v-bind:class="{ selectedCategory: item.isSelected }">
          
            
            <div @contextmenu="openContextMenu(item)" class="swatch" :style="{ background: item.hex}" v-bind:value="value"  v-on:click="categoryClicked(item, this)"></div> {{item.text}}<br><br>
        </li>
  </ul>
</div>
<div>
  <input v-model="categoryName" id="addCategory" placeholder=" Tilføj Kategori"/> 
  <button type="button" class="btn btn-default" id="SaveBTN" @click="saveCat">Tilføj</button>


</div>
<ul class='custom-menu2'>
      <li v-on:click="customMenuClick()">Slet</li>
    </ul>
</div>
</template>

<script src="https://code.jquery.com/jquery-3.4.1.min.js"></script>

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
          colors: [ "#b8e186","#7fbc41","#4d9221","#276419", "#d9d9d9", "#bc80bd","#ccebc5", "#ffed6f"],
          LastClickedItem: "",
          numberOfColumns: 4,
          isSelected: false
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
        categoryClicked: function(item, _this){
          this.items.forEach( item => {
            this.$set(item, "isSelected", false)
            }
          )  
          this.$set(item, "isSelected", true)
          this.$emit('update:selected', this.isSelected)
          this.$emit('update:color', "")
          this.$emit('update:color', item.hex)
        }, openContextMenu(item){
          
          this.LastClickedItem = item; 
          event.preventDefault();  
          
            $(".custom-menu2").finish().toggle(100).
              // In the right position (the mouse)
              css({
                  top:  $(event.target).position().top + "px",
                  left:  $(event.target).position().left + "px"
              })
        },  customMenuClick(){
                var result = confirm("Kategorien bliver slettet permanent. Fortsæt?");
                if (result) {
                  var _this = this;
                  var index = this.items.indexOf(this.LastClickedItem);
                  if (index > -1) {
                    _this.$delete(this.items, index);
                  }
                }
                    
              
              $(".custom-menu2").finish().toggle(100)
                 
        },
      },
      computed: {
      gridStyle() {
        return {
          gridTemplateColumns: `repeat(${this.numberOfColumns}, minmax(100px, 1fr))`
        }
    },
    }
    }

$(document).click(function(event) {
  //if you click on anything except the modal itself or the "open modal" link, close the modal
  if ($(".custom-menu2").is(":visible")) {
    $(".custom-menu2").hide();
  }
});

</script>
