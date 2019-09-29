<template>
      <div class="modal" v-show="value">
        <div class="modal-content">
          <div class="modal-header">Kategoriser</div>
          <p> Vælg en kategori</p> 
          <CategoryPicker v-model="items" :items="items" :chosenRect="this.chosenRect"></CategoryPicker>
          <button @click.prevent="close" class="mt-3 border-b border-teal font-semibold">Close</button>
        </div>
      </div>
</template>


<script>
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import CategoryPicker from './CategoryPicker'

window.$ = require('jquery')
window.JQuery = require('jquery')

export default {
    name: "Modal", 
    props: ["value", "chosenRect"],
    components: {
        CategoryPicker
    },
     methods: {
     close() {
         this.$emit("input", !this.value);
        }
     },
     data(){
    return { items: [
            { text: 'Venner', hex:"#F4D03F"},
            { text: 'Familie', hex:"#229954"},
            { text: 'Arbejde', hex:"#9B59B6"}
        ]
        }
     }
};

/*
$(document).ready(function(){
     
    function alignModal(){
        var modalDialog = $(this).find(".modal-dialog");
        /* Applying the top margin on modal dialog to align it vertically center */
        
   /*     modalDialog.css("margin-top", Math.max(0, ($(window).height() - modalDialog.height()) / 2));
    }
    // Align modal when it is displayed
    $(".modal").on("shown.bs.modal", alignModal);
    
    // Align modal when user resize the window
    $(window).on("resize", function(){
        $(".modal:visible").each(alignModal);
    });   
}); */
</script>

