

<template>
      <div class="modal modal-dialog" v-show="value">
        <div class="modal-content">
          <div class="modal-header">Rediger</div>
          <p> Her er en test modal</p> 
          <button @click.prevent="close" class="mt-3 border-b border-teal font-semibold">Close</button>
        </div>
      </div>
    </template>


<script>
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'


window.$ = require('jquery')
window.JQuery = require('jquery')

export default {
    name: "Modal", 
    props: {
        value: {
            required: true
        }
    },
     methods: {
     close() {
         this.$emit("input", !this.value);
        }

    }
};


$(document).ready(function(){
    function alignModal(){
        var modalDialog = $(this).find(".modal-dialog");
        /* Applying the top margin on modal dialog to align it vertically center */
        
        modalDialog.css("margin-top", Math.max(0, ($(window).height() - modalDialog.height()) / 2));
    }
    // Align modal when it is displayed
    $(".modal").on("shown.bs.modal", alignModal);
    
    // Align modal when user resize the window
    $(window).on("resize", function(){
        $(".modal:visible").each(alignModal);
    });   
});
</script>

