<!DOCTYPE html>
<meta charset="utf-8">

<head>
  <!-- Carga D3 -->
  <script src="d3.js"></script>
  <!-- Fuente para tooltip fallido -->
  <!-- <script src="http://labratrevenge.com/d3-tip/javascripts/d3.tip.v0.6.3.js"></script> -->
	<style>

  /*CSS para los ejes*/
	.axis {
	  font: 10px sans-serif;
	}

	.axis path,
	.axis line {
	  fill: none;
	  stroke: #000;
	  shape-rendering: crispEdges;
	}

   /*CSS de un tooltip fallido*/
  .d3-tip {
  line-height: 1;
  font-weight: bold;
  padding: 12px;
  background: rgba(0, 0, 0, 0.8);
  color: #fff;
  border-radius: 2px;
}

/*CSS para el tooltip*/
#tooltip {
        position: absolute;
        width: 100px;
        height: auto;
        padding: 10px;
        background-color: white;
        -webkit-border-radius: 10px;
        -moz-border-radius: 10px;
        border-radius: 10px;
        -webkit-box-shadow: 4px 4px 10px rgba(0, 0, 0, 0.4);
        -moz-box-shadow: 4px 4px 10px rgba(0, 0, 0, 0.4);
        box-shadow: 4px 4px 10px rgba(0, 0, 0, 0.4);
        pointer-events: none;
      }
      
      #tooltip.hidden {
        display: none;
      }
      
      #tooltip p {
        margin: 0;
        font-family: sans-serif;
        font-size: 16px;
        line-height: 20px;
      }

	</style>
</head>

<body>
	
<!-- Define tooltip. Cambiar 'A la fecha' por la fecha real -->
<div id="tooltip" class="hidden">
      <p><strong>Fecha X:</strong></p>
      <p><span id="value"> </span> %</p>
    </div>

<p>Crecimiento porcentual diario:</p>

<!-- Inicia script de d3 -->

<script type="text/javascript">


//Define margenes

var margin = {top: 20, right: 20, bottom: 70, left: 40},
w = 600 - margin.left - margin.right,
h = 300 - margin.top - margin.bottom;

//Define función de escala. Solo se especifica el tipo y el rango. Dominio sera definido en funcion de los datos. Idealmente, la escala de la fecha debiera ser ordinal.

var xScale = d3.scaleLinear().range([0,w],.05);
var yScale = d3.scaleLinear().range([h,0]);

//Define las funciones ejes

var xAxis = d3.axisBottom()
          .scale(xScale)
          .ticks(10);

      var yAxis = d3.axisLeft()
        .scale(yScale)
        .ticks(10);

//Define SVG que contiene al grafico

var svg = d3.select("body")
            .append("svg")
            .attr("width",w+ margin.left + margin.right)
            .attr("height",h+ margin.top + margin.bottom);

//Carga los datos y dibuja el grafico. Observar que todo el codigo del grafico esta concatenado con la funcion d3.csv()

var datos =
  d3.csv("serie-valpo.csv",function(d){
    return [+d.Crono,+d.CasosPorc];
  }).
  then(function(data){

    //Define dominio de la escala

    xScale.domain([0,d3.max(data, function(d){return d[0]+1})]);
    yScale.domain([0, d3.max(data, function(d) { return d[1]; })]);

    //Agrega eje x

    svg.append("g")
            .attr("class", "x axis")
          .attr("transform", "translate("+margin.left+"," + h + ")")
          .call(xAxis)
          .selectAll("text")
          .style("text-anchor", "end")
          .attr("dx", "-.8em")
          .attr("dy", "-.55em")
          .attr("transform", "rotate(-90)" );

    //Agrega eje y

    svg.append("g")
          .attr("class", "y axis")
          .call(yAxis)
          .attr("transform","translate("+margin.left+",0)")
          .append("text")
          .attr("transform", "rotate(-90)")
          .attr("y", 6)
          .attr("dy", ".71em")
          .style("text-anchor", "end")
          .text("Crecimiento porcentual");

    //Intento fallido de tooltip
    //var tip = d3.tip()
    // .attr('class', 'd3-tip');

    //Agrega barras y tooltip

    svg.selectAll("rect")
      .data(data)
      .enter()
      .append("rect")
      .attr("x", function(d) { return xScale(d[0])+margin.left; })
      .attr("width", 10)
      .attr("y", function(d) { return yScale(d[1]); })
      .attr("height", function(d){return h - yScale(d[1])})
      .attr("fill","steelblue")
      .on("mouseover", function(d) {

          //Define la posicion del tooltip

          var xPosition = parseFloat(d3.select(this).attr("x"));
          var yPosition = parseFloat(d3.select(this).attr("y"));

          //Ajuste de la posicion y valor del tooltip

          d3.select("#tooltip")
            .style("left", xPosition + "px")
            .style("top", h-yPosition/4 + "px")           
            .select("#value")
            .text(d[1]);
         
          //Muestra el tooltip

          d3.select("#tooltip").classed("hidden", false);

         })
      .on("mouseout", function() {
         
          //Oculta el tooltip

          d3.select("#tooltip").classed("hidden", true);
          
         })

    

});



</script>

<p>Modifiqué los valores demasiado grandes a mano para que se aprecien los valores pequeños. Esto se puede lograr modificando la escala adecuadamente. Para valores muy dispares, el cambio de posición brusco del tooltip molesta.</p>

</body>