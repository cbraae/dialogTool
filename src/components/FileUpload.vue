<template>
  <div id="fileUpload">
    <input type="file" @change="onFileChange">
</div>
</template>


<script>



import * as d3 from 'd3'
export default {
name: "FileUpload",
  data() {
    return {
      loadData: ''
    }
    
  },
  methods: {
    onFileChange(e) {
      var files = e.target.files || e.dataTransfer.files;
      if (!files.length)
        return;
      this.fetchData(files[0])
    },
    async fetchData(file) {
      var _this = this;
      var reader = new FileReader(); 
      
        reader.readAsText(file); 
        reader.onloadend = async function() {
        let data = await d3.csvParse(reader.result);
        
        var cleanedData = data.map(
            item => item[data.columns[0]].split("Z")[0]
          );
          cleanedData.removeIf(function(item, idx) {
            return item == ";";
          });
          
          _this.loadData = cleanedData;
          _this.$emit('update:loadData', _this.loadData)
      }
       
        
    },
    createImage(file) {
      var image = new Image();
      var reader = new FileReader();
      var vm = this;

      reader.onload = (e) => {
        vm.image = e.target.result;
      };
      reader.readAsDataURL(file);
    },
    removeImage: function (e) {
      this.image = '';
    }
  }
}

</script>