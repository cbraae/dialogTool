

<template>
<div>

<div id="cat">

  <ul :style="gridStyle" class="card-list">
          <li v-for="(item) in items" :key="item.text" class="card-item" v-bind:class="{ selectedCategory: item.isSelected }">
            <div @contextmenu="openContextMenu(item)" class="swatch" :style="{ background: item.hex}" v-bind:value="value"  v-on:click="categoryClicked(item, this)"></div> <p class="categoryText"> {{item.text}}</p><br><br>
        </li>
  </ul>
</div>
<div>
  <div class="swatch chosenColor" :style="{ background: currentColor }" v-on:click="colorPicker" />
  <img src="../../public/data/drop-down-arrow.png" id="dropdownicon" width="6" height="6">
  <input v-model="categoryName" id="addCategory" placeholder=" Tilføj Kategori"/> 

   <button type="button" class="btn btn-default drawingsbuttons" id="SaveBTN" @click="saveCat">Tilføj</button>

</div>
<ul class='custom-menu2'>
      <li v-on:click="customMenuClick()">Slet</li>
</ul>
<br>
<div v-if="colorPickerChosen"> 
 <ul class="color-list" v-if="colorPickerChosen">
          <li v-for="item in colors" :key="item.index" class="color">
            <div class="swatch color-item" :style="{ background: item}" v-bind:value="value"  v-on:click="colorClicked(item, this)"></div>
        </li>
  </ul>
</div>
</div>
</template>


<script>

/*window.$ = require('jquery')
window.JQuery = require('jquery')*/

import "bootstrap";
import "bootstrap/dist/css/bootstrap.min.css";

    export default {
      name: "CategoryPicker",
      props: ["items", "chosenRect", "value", "catDict", "chosenDateTime","repModalOpen", "color"],
      components: {
      },data() {
        return {
          colorPickerChosen: false,
          modalOpen:false,
          categoryName: "",
          colorCounter: 0,
          currentColor: "",
          colors: [ "#1f78b4","#b2df8a","#33a02c","#fb9a99", "#e31a1c", "#b15928","#a6cee3", "#cab2d6"],
          LastClickedItem: "",
          numberOfColumns: 4,
          isSelected: false
        };
      }, mounted(){
        this.currentColor = this.colors[1]
      },
      
      methods: {
        saveCat: function(event){
          this.items.push({ text: this.categoryName, hex: this.currentColor});
          this.categoryName = "";
          //this.colorCounter++;
          
        },
        colorPicker: function() {
          this.colorPickerChosen = !this.colorPickerChosen;
        }, colorClicked: function(item){
            this.currentColor = item
            this.colorPickerChosen = false;
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
