<template>
    <div>
    </div>
</template>

<script>
export default {
    name: "ShowDrawingsComponent",
    props: ["startDate", "endDate", "brushId", "brushEnd"],
    watch: {
        startDate: {
            handler: function(){
                if(!this.brushEnd){
                    this.showDrawings(this.startDate, this.endDate, this.brushId)
                } else {
                    this.showDrawings(this.startDate, this.endDate, this.brushId)
                    this.showSmallMultiples(this.startDate, this.endDate, this.brushId)
                }
            }, 
            deep: true,
            immediate: true

        }
    }, mounted(){

      this.brushImageDict = {}
      this.currentlyShownMondays = {}
      localStorage.setItem("brushImageDict", JSON.stringify(this.brushImageDict));
      localStorage.setItem("currentlyShownMondays", JSON.stringify(this.currentlyShownMondays));


    }, data() {
      return {
        ImagePlaceInDict: {},
        brushImageDict: {},
        currentlyShownMondays: {},
        imgDict: {},
        svgDict: {},
        brushes: [], 
        drawingSaved: ""
    }
    },
    methods: {
    showDrawings(startDate, endDate, id){
         if (localStorage.getItem('imgDict')){
        this.imgDict = JSON.parse(localStorage.getItem('imgDict'));
          };
            
          if(localStorage.getItem('colorDict')){
            this.colorDict =JSON.parse(localStorage.getItem('colorDict'));
          }
          
          if (localStorage.getItem('svgDict')){
            this.svgDict = JSON.parse(localStorage.getItem('svgDict'));
          };
          if (localStorage.getItem('brushImageDict')){
        this.brushImageDict = JSON.parse(localStorage.getItem('brushImageDict'));
          };

           if (localStorage.getItem('currentlyShownMondays')){
        this.currentlyShownMondays = JSON.parse(localStorage.getItem('currentlyShownMondays'));
          };
        


        //Altid slet alle billeder først
        $(".bigImages").remove();
        

        var chosenDays = []
        for(var monday in this.imgDict) {
          var mday = new Date(new Date(monday))

          if(mday >= startDate && mday <= endDate){
              chosenDays.push(monday)
          }
        }
        
        this.ImagePlaceInDict = {}
        this.brushImageDict[id] = []
        this.currentlyShownMondays[id] = []
        
        for(var i = 0; i < chosenDays.length; i++) {
            var monday = chosenDays[i]
            var chosenMonday = this.imgDict[monday]

           for(var j =0; j< chosenMonday.length; j++ ){
              var currentMonday = chosenMonday[j]
              this.ImagePlaceInDict[currentMonday] = [monday,j]

              //Keep track of which brush holds which images
              this.brushImageDict[id].push(currentMonday)
              this.currentlyShownMondays[id].push(monday)
           }
        }
        

        if(Object.entries(this.imgDict).length > 0){
        var imageList = []
        var container = document.getElementById('chart')
        var smallMultiplesContainer = document.getElementById('categorySection')
        var zindex = 1;
        var width = 200;
     //Tegn eller Gentegn alle billeder i dict: 
      for(var brush in this.brushImageDict){
      var imagesFromBrush = this.brushImageDict[brush]
      for(var i =0; i < imagesFromBrush.length; i++){
              currentMonday = imagesFromBrush[i]
              

              var rows = document.getElementById("rows")
              rows.style.width = "400px";
              var totalCounter = 0;

              var drawing = document.createElement("IMG")
              drawing.style.zIndex=zindex;
              drawing.style.position = "absolute";
              drawing.style.left="0px";
              drawing.style.top="0px";
              drawing.className = "bigImages images "+ id.toString()  + " "  + currentMonday.toString()
              drawing.src = "data:image/png;base64," + currentMonday;
              container.appendChild(drawing);
           }

        }
        }
      else {
          $(".images").remove();
          //$("#chart").removeClass("showingDrawings");
      }

      if($(".bigImages").length > 0){
        $("#chart").addClass("showingDrawings")
      } else {
        $("#chart").removeClass("showingDrawings")
      }

      var _this = this;
        $("#chart, .bigImages")
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
                  left: event.pageX + "px"
              })
                 $(".custom-menu li").click(function(){
                   
                  switch($(this).attr("data-action")) {                
                // A case for each action. Your actions here
               
                case "first":  
                    for(var i = 0; i < _this.brushes.length-1; i++){
                       $("#brush-"+i).remove();

                    }
                    
                    $(".custom-menu").hide(100);
                    $(".images").remove();
                    $("#chart").removeClass("showingDrawings")
                   
                     // If all images are removed, slider should be removed too
                    if($(".smallImages").length == 0){
                       if($(".rows").hasClass("slick-initialized")){
                          $('.rows').slick("unslick")
                          $("#noDrawings").hide();
                        }
                    }

                  break;
                    }
                  });
            });

    localStorage.setItem("brushImageDict", JSON.stringify(_this.brushImageDict));
    localStorage.setItem("currentlyShownMondays", JSON.stringify(_this.currentlyShownMondays));
    },
    showSmallMultiples(startDate, endDate, id){
         if (localStorage.getItem('imgDict')){
        this.imgDict = JSON.parse(localStorage.getItem('imgDict'));
          };
            
          if(localStorage.getItem('colorDict')){
            this.colorDict =JSON.parse(localStorage.getItem('colorDict'));
          }
          
          if (localStorage.getItem('svgDict')){
            this.svgDict = JSON.parse(localStorage.getItem('svgDict'));
          };

            if (localStorage.getItem('brushImageDict')){
        this.brushImageDict = JSON.parse(localStorage.getItem('brushImageDict'));
          };

           if (localStorage.getItem('currentlyShownMondays')){
        this.currentlyShownMondays = JSON.parse(localStorage.getItem('currentlyShownMondays'));
          };
        

      if($(".rows").hasClass("slick-initialized")){
         $('.rows').slick("unslick")
       }
       $(".imgs").remove();
       $("#noDrawings").hide();
       $(".smallImages").off();
       console.log(this.brushImageDict)

        if(Object.entries(this.imgDict).length > 0){
        var imageList = []
        var container = document.getElementById('chart')
        var smallMultiplesContainer = document.getElementById('categorySection')


        var zindex = 1;
        var overlayLocation = 30
        var width = 200;

        var rows = document.getElementById("rows")
        rows.style.width = "400px";
        var totalCounter = 0;

      for(var brush in this.brushImageDict){
      var imagesFromBrush = this.brushImageDict[brush]
        for(var i =0; i < imagesFromBrush.length; i++){

              var currentMonday = imagesFromBrush[i]
              var monday = this.currentlyShownMondays[brush][i]
              
              if(column){
                    rows.appendChild(column)
              }
              
              
              var column = document.createElement('div');
              column.classList="column images imagetitles imgs "+id.toString()+ " showingDrawings "  + currentMonday.toString()

              var mondayt = new Date(monday)
              var sunday = new Date(new Date(mondayt).setDate(mondayt.getDate()+6))
              var currentMonth = parseInt(mondayt.getMonth())+1
              var title = document.createTextNode("Mandag d." + mondayt.getDate() + "/" + currentMonth + " - " + "Søndag d." + sunday.getDate() + "/" + currentMonth );
              
              //title.className = "imagetitles"

              var image = document.createElement("IMG")
              image.src = "data:image/svg+xml;base64,"+this.svgDict[monday]
              image.className = "images imgs " + id.toString()  + " " + currentMonday.toString()
              image.id = zindex.toString();
              

              //image.style.left = overlayLocation +"px"
              image.style.width = width-5 +"px";
              image.style.height = 100 +"px";
              smallMultiplesContainer.appendChild(image);
              column.style.width = 2*width-10+"px"
              column.appendChild(title);
              column.appendChild(image);
              image.style.opacity = "50%"
              
              var imageOverlay = document.createElement("IMG")
              imageOverlay.style.zIndex=zindex;
              imageOverlay.style.position = "absolute";
              imageOverlay.style.width = width-15+"px";
              imageOverlay.style.height = "100px";
            
              
              imageOverlay.src = "data:image/png;base64," + currentMonday;
              imageOverlay.className = "smallMutipless images smallImages imgs "+ id.toString()  + " " + currentMonday.toString()
              zindex +=1;
              overlayLocation+=width;
              column.appendChild(imageOverlay);
              
              totalCounter+=1; 
              
              
           }
        }
        }

      if(column){
            rows.appendChild(column)
            smallMultiplesContainer.appendChild(rows)
             }
        
          if($(".smallImages").length > 6){
              $('.rows').slick({
              speed: 800,
              infinite: true,
              vertical: true,
              slidesToShow: 3,
              verticalSwiping: true,
              arrows: true
              });
          }
        

          if($(".smallImages").length ==  0){
            $("#noDrawings").show();
          }
         
         var clickedItem = "old";
         var brushId = "old"
         var _this = this;
         
             $(".smallImages")
            .on("mousedown", function() {
                if (event.button === 2) { // only enable for right click
                  event.stopImmediatePropagation();
                }
            })
            .on("contextmenu", function(){
              event.preventDefault();
              var userHasAnswered = false;
              var classList = $(this)[0].classList;
              clickedItem = classList[classList.length-1]
              brushId = classList[classList.length-2]

                 $(".custom-menu-delete").show()
                  .css({
                  top: event.pageY + "px",
                  left: event.pageX + "px"
              })
                 $(".custom-menu-delete li").click(function(){
                   event.preventDefault();

                    switch($(this).attr("data-action")) {                
                // A case for each action. Your actions here
                case "first":  

                    if(!userHasAnswered){
                      userHasAnswered = true;
                      var result = confirm("Tegningen bliver slettet permanent. Fortsæt?");
                      
                    }
                    
                    
                    $(".custom-menu-delete").hide(100);

                    if (result) {
                        removeImage(_this.ImagePlaceInDict[clickedItem][0],_this.ImagePlaceInDict[clickedItem][1])

                        for(var i=0; i < _this.brushImageDict[brushId].length; i++ ){
                          if(_this.brushImageDict[brushId][i].toString().localeCompare(clickedItem) == 0){
                            var index = i;
                          }
                        }

                      
                      _this.brushImageDict[brushId].splice(index, 1);
                      _this.currentlyShownMondays[brushId].splice(index,1)
                      
                      if(_this.brushImageDict[brushId].length == 0){
                        delete _this.brushImageDict[brushId]
                        delete _this.currentlyShownMondays[brushId]
                      }
                      localStorage.setItem("brushImageDict", JSON.stringify(_this.brushImageDict));
                      localStorage.setItem("currentlyShownMondays", JSON.stringify(_this.currentlyShownMondays));

                        _this.showDrawings("", "", brushId, false);
                        _this.showSmallMultiples("", "", brushId);
                        
                       
                        //$('.'+brushId.toString()).remove();
                         // If all images are removed, slider should be removed too
                        if($(".smallImages").length == 0){
                          
                          $("#"+brushId.toString()).remove();
                          if($(".rows").hasClass("slick-initialized")){
                              $('.rows').slick("unslick")
                              $("#noDrawings").hide();
                              
                            }
                        }
                        
                    }
                    
                    
                  break;
                    }
            });
           

            });


          function removeImage(monday, arrayPlace){
              _this.$delete(_this.colorDict, _this.imgDict[monday][arrayPlace]);
              _this.$delete(_this.imgDict[monday], arrayPlace);
              
            if(_this.imgDict[monday].length == 0){
                _this.$delete(_this.imgDict, monday)
             } 

            localStorage.setItem("brushImageDict", JSON.stringify(_this.brushImageDict));
            localStorage.setItem("currentlyShownMondays", JSON.stringify(_this.currentlyShownMondays));
            localStorage.setItem("imgDict", JSON.stringify(_this.imgDict));
            this.$emit("drawingSaved", true)

            // _this.initialiseCategoryDict();
            // _this.drawCategoryGrid();
          }
      

        
    }
}   
}
</script>