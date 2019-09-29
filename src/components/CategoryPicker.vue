

<template>
<ul id="cat">
 
    <li v-for="item in items" :key="item.text">
        <div class="swatch" :style="{ background: item.hex }" v-on:click="categoryClicked(item.hex)"></div> {{item.text}} <br><br>
        </li>    
</ul>
</template>

<script>


    export default {
      name: "CategoryPicker",
      props: ["items", "chosenRect"],
      methods: {
        categoryClicked: function(color){
          
          $("#"+this.chosenRect[0].toString()+this.chosenRect[1].toString()).css("fill", color);
        }
      }
    }

</script>