<template>
    <div class="buttonGoupContainer"> 
        <div class="buttonGroup left">
          <button
              id="save"
              name="save"
              v-on:click="saveDrawing"
              class="btn btn-default drawingbuttons"
            >Gem</button>
            <button
              id="cancel"
              name="cancel"
              v-on:click="cancel"
              class="btn btn-default drawingbuttons"
            >Fjern tegne-visning</button>    
            <button
              id="clear"
              name="clear"
              v-on:click="clearDrawing"
              class="btn btn-default drawingbuttons"
            >Ryd kanvas</button>
             <button
              id="undo"
              name="undo"
              v-on:click="undo"
              class="btn btn-default drawingbuttons"
            >Visk Ud</button>         
        </div>   
         </div> 
</template>

<script>

import * as d3 from 'd3'

export default {
    name:"CreateDrawing",
    props: ["color", "trackingData", "chosenMonday"],
    data() {
        return {
            chosenColors: [],
            calendarWidth: 1080,
            calendarHeight: 450, 
            drawingSaved: false,
            imgDict: {},
            colorDict: {},
            svgDict: {}

        }
    },
    watch: {
        color: {
           handler: function(){
               this.enableDrawing(this.color, false);
           }     
        }
    }, 
    methods: {
    removeCanvasOverlay(){
      $("#drawing-area").remove();
      this.canvasExists = false;
    },
    findCurrentWeekNumber(chosenDate){
          var monday = this.getMonday(chosenDate);
          return d3.timeMonday.count(new Date(this.trackingData[0]), monday)

    }, getMonday(d) {
            d = new Date(d);
            var day = d.getDay(),
                diff = d.getDate() - day + (day == 0 ? -6:1); // adjust when day is sunday
            var monday = new Date(d.setDate(diff));
            monday.setHours(0,0,0,0);
            return monday

    },
    createCanvasOverlay() {
      var canvasContainer = document.getElementById('chart');
      document.body.appendChild(canvasContainer);
      var myCanvas = document.createElement('canvas');
      myCanvas.id = "drawing-area";
      myCanvas.style.width = this.calendarWidth;
      myCanvas.style.height = "460px";
      myCanvas.style.zIndex="1080";
      myCanvas.style.left="0px";
      myCanvas.style.top="0px";
      myCanvas.width=this.calendarWidth;
      myCanvas.height=this.calendarHeight;
      myCanvas.style.overflow = 'visible';
      myCanvas.style.position = 'absolute';
      canvasContainer.appendChild(myCanvas);

      var _this = this;
       $("#drawing-area")
            .on("mousedown", function() {
                if (event.button === 2) { // only enable for right click
                  event.stopImmediatePropagation();
                }
                
            })
            .on("contextmenu", function(){
              event.preventDefault();
               /* Få fat i alle brushes og slet dem */ 
              
                
                 $(".custom-menu").show()
                  .css({
                  top: event.pageY + "px",
                  left: event.pageX + "px",
              }).css("z-index", 10000);
                 $(".custom-menu li").click(function(){
                    
                   
                  switch($(this).attr("data-action")) {                
                // A case for each action. Your actions here
               
                case "first":  
                    
                    $(".custom-menu").hide(100);
                    _this.cancel();
                    _this.items.forEach( item => {
                        _this.$set(item, "isSelected", false)
                      }
                    )
                     // If all images are removed, slider should be removed too
                    if($(".smallMutiples").length == 0){
                       if($(".rows").hasClass("slick-initialized")){
                          $('.rows').slick("unslick")
                        }
                    } 

                  break;
                    }
                  });
            });
        },
        cancel(){
        $(".buttonGroup").css("visibility", "hidden");
        $("#chart").removeClass("selectedCategory")
        this.items.forEach( item => {
            this.$set(item, "isSelected", false)
          }
        )
        this.chosenColors = [];
        this.drawCategoryGrid()
        this.removeCanvasOverlay();
        //$("#drawing-area").remove();
        //this.createCanvasOverlay()
    },
    saveDrawing(){
      $(".buttonGroup").css("visibility", "hidden");
      $("#chart").removeClass("selectedCategory")
        if (localStorage.getItem('imgDict')){
        this.imgDict = JSON.parse(localStorage.getItem('imgDict'));
          };
            
          if(localStorage.getItem('colorDict')){
            this.colorDict =JSON.parse(localStorage.getItem('colorDict'));
          }
          
          if (localStorage.getItem('svgDict')){
            this.svgDict = JSON.parse(localStorage.getItem('svgDict'));
          };


       this.drawingSaved = true;
       this.$emit("update:drawingSaved", "")
       this.$emit("update:drawingSaved", this.drawingSaved)
      
        const canvas = document.getElementById('drawing-area');
        var dataURL = canvas.toDataURL("image/png");
        var imgData = dataURL.replace(/^data:image\/(png|jpg);base64,/, "");

        if(this.imgDict[this.chosenMonday]){

          this.imgDict[this.chosenMonday].push(imgData);
          
          this.colorDict[imgData] = []
          
          for(var color in this.chosenColors){
              
              if(this.chosenColors[color].toString().includes("#")){
                this.colorDict[imgData].push(this.chosenColors[color])
                
              }

          }
          
         
        } else {
            this.imgDict[this.chosenMonday] = []
            this.imgDict[this.chosenMonday].push(imgData); 

            for(var color in this.chosenColors){
               if(this.chosenColors[color].toString().includes("#")){
                 if(!this.colorDict[imgData]){
                   this.colorDict[imgData] = []
                   this.colorDict[imgData].push(this.chosenColors[color])
                 } else {
                   this.colorDict[imgData].push(this.chosenColors[color])
                 }
              }
          }
         
        }
       

        var s = new XMLSerializer().serializeToString(document.getElementById("calendar"))
        var b64 = window.btoa(s);
        this.svgDict[this.chosenMonday] = b64;
        var numberInDict = this.findCurrentWeekNumber(this.chosenMonday);

        
        
        localStorage.setItem("imgDict", JSON.stringify(this.imgDict));
        localStorage.setItem("colorDict", JSON.stringify(this.colorDict));
        localStorage.setItem("svgDict", JSON.stringify(this.svgDict));

        this.chosenColors = [];
        //this.drawCategoryGrid()
        this.removeCanvasOverlay();

        //REMOVE shown drawings if exist
        if($("#chart").hasClass("showingDrawings")){
          for(var i = 0; i < this.brushes.length-1; i++){
            $("#brush-"+i).remove();
          }
                      
          $(".custom-menu").hide(100);
          $(".images").remove();
          $("#chart").removeClass("showingDrawings") 
        }
        var currentMonth = parseInt(this.chosenMonday.getMonth())+1
        var sunday = new Date(new Date(this.chosenMonday).setDate(this.chosenMonday.getDate()+6))
        alert("Tegningen til mandag d."+this.chosenMonday.getDate() + "/" + currentMonth+"- søndag d."+sunday.getDate()+"/"+currentMonth+" er gemt.")



    },clearDrawing(){
      const canvas = document.getElementById('drawing-area');
      if(canvas !== null) {
              const context = canvas.getContext('2d');
              context.clearRect(0,0,this.calendarWidth, this.calendarHeight);
      }
    }, undo(){

      this.enableDrawing("rgba(255,255,255,1)", true)

    }, 
    enableDrawing(color, eraser) {

       this.drawingSaved = false;
       this.$emit("update:drawingSaved", "")
       this.$emit("update:drawingSaved", this.drawingSaved)

       this.drawingSaved = false;
       if(!this.canvasExists){
         this.createCanvasOverlay()
         this.canvasExists = true;
       } 

      const canvas = document.getElementById('drawing-area');
      const canvasContext = canvas.getContext('2d');
      const state = {
        mousedown: false
      };

    // ===================
    // == Configuration ==
    // ===================
    var lineWidth = 1;
    const halfLineWidth = lineWidth / 2;
    const fillStyle = color;
    const strokeStyle = color;
    const shadowColor = '#333';
    canvasContext.globalCompositeOperation="source-over";
    

      if(eraser){
        lineWidth= 100;
        canvasContext.globalCompositeOperation = "destination-out";  
      } else{
        this.chosenColors.push(color)
      }
        

    // =====================
    // == Event Listeners ==
    // =====================
    canvas.addEventListener('mousedown', handleWritingStart);
    canvas.addEventListener('mousemove', handleWritingInProgress);
    canvas.addEventListener('mouseup', handleDrawingEnd);
    canvas.addEventListener('mouseout', handleDrawingEnd);

    canvas.addEventListener('touchstart', handleWritingStart);
    canvas.addEventListener('touchmove', handleWritingInProgress);
    canvas.addEventListener('touchend', handleDrawingEnd);

    var cntrlIsPressed = false;
    $(document).keydown(function(event){
        if(event.which=="16")
         cntrlIsPressed = true;
    });
        
    $(document).keyup(function(){
      cntrlIsPressed = false;
    });

    // ====================
    // == Event Handlers ==
    // ====================
    function handleWritingStart(event) {
      
      $(".buttonGroup").css("visibility", "visible");
      $("#chart").addClass("selectedCategory")
      
      

      event.preventDefault();

      const mousePos = getMosuePositionOnCanvas(event);
      
      canvasContext.beginPath();

      canvasContext.moveTo(mousePos.x, mousePos.y);

      canvasContext.lineWidth = lineWidth;
      canvasContext.strokeStyle = strokeStyle;
      canvasContext.shadowColor = null;

      
      canvasContext.fill();
      
      state.mousedown = true;
    }

    function handleWritingInProgress(event) {
      event.preventDefault();
      
      if (state.mousedown && cntrlIsPressed) {
        $("#chart").addClass("drawingCursor")
        const mousePos = getMosuePositionOnCanvas(event);

        canvasContext.lineTo(mousePos.x, mousePos.y);
        canvasContext.stroke();

      } else {
        $("#chart").removeClass("drawingCursor")
      }
    }

    function handleDrawingEnd(event) {
      event.preventDefault();

      if (state.mousedown) {
        canvasContext.shadowColor = shadowColor;
        canvasContext.stroke();
      }
      
      state.mousedown = false;
    }

    function handleClearButtonClick(event) {

      event.preventDefault();
      
      clearCanvas();
    }

    // ======================
    // == Helper Functions ==
    // ======================
    function getMosuePositionOnCanvas(event) {
      const clientX = event.clientX || event.touches[0].clientX;
      const clientY = event.clientY || event.touches[0].clientY;
      const { offsetLeft, offsetTop } = event.target;
      const canvasX = clientX - offsetLeft -30;
      const canvasY = clientY-274;

      return { x: canvasX, y: canvasY };
    }


        },
    }

}




</script>