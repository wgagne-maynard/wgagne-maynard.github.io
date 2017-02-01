<!DOCTYPE html>

<html>

<h1>San Francisco Bike Share Usage</h1>

<title>San Francisco Bike Share Usage</title>



<meta charset="utf-8">
<style>

body {
  font: 10px sans-serif;
}

.axis path,
.axis line {
  fill: none;
  stroke: #000;
  shape-rendering: crispEdges;
}

.x.axis path {
  display: none;
}

.axis text {
    font-family: sans-serif;
    font-size: 12px;
}

.line {
  fill: none;
  stroke: steelblue;
  stroke-width: 1.5px;
}

div.tooltip {   
  position: absolute;           
  text-align: center;           
  width: 140px;                  
  height: 45px;                 
  padding: 2px;             
  font: 14px sans-serif;
  font-style: bold;
  background-color: white;        
  border: 0px;      
  border-radius: 8px;           
  pointer-events: none;         
}

h1{
  font-size: 28px;
}

p{
  font-size: 14px;
  width: 960px;
}


</style>
<body>
<script src="http://d3js.org/d3.v3.js"></script>

<p>This graphic shows the number of rides along the ten most popular routes between  <a href="http://www.bayareabikeshare.com/datachallenge"> Bay Area Bike Share</a> stations during the first six months. Use the buttons below to see the breakdown of rides versus different units of time. Scroll over the lines to see the start and end points. Click on a line to change the color.</p>

<TABLE BORDER="0">
<TR>
<TD>
    <input name="HourButton" 
           type="button" 
           value="Time of Day" 
           onclick="Hour()" />
</TD>	

<TD>
    <input name="DayButton" 
           type="button" 
           value="Day of Week" 
           onclick="Day()" />
 </TD>

<TD>
    <input name="MonthButton" 
           type="button" 
           value="Month of Year" 
           onclick="Month()" />
</TD>


<TD>
    <input name="Reset" 
           type="button" 
           value="Reset" 
           onclick="Reset()" />
</TD>

</TR>

</TABLE>

<script>

var margin = {top: 20, right: 80, bottom: 30, left: 50},
    width = 1050 - margin.left - margin.right,
    height = 500 - margin.top - margin.bottom;

var parseDateHour = d3.time.format("%H").parse;

var parseDateDay = d3.time.format("%d").parse;

var parseDateMonth = d3.time.format("%m-%y").parse;

var x = d3.time.scale()
    .range([0, width]);

var y = d3.scale.linear()
    .range([height, 0]);

var color = d3.scale.category10();

var xAxis_day = d3.svg.axis()
    .scale(x)
    .orient("bottom").ticks(7)
    .tickFormat(d3.time.format("%A"));

var xAxis_hour = d3.svg.axis()
    .scale(x)
    .orient("bottom").ticks(12)
    .tickFormat(d3.time.format("%H:%M"));

var xAxis_month = d3.svg.axis()
    .scale(x)
    .orient("bottom").ticks(7)
    .tickFormat(d3.time.format("%B %Y"));

var yAxis = d3.svg.axis()
    .scale(y)
    .orient("left");

var line = d3.svg.line()
    .interpolate("basis")
    .x(function(d) { return x(d.date); })
    .y(function(d) { return y(d.number); });

var div = d3.select("body").append("div")   
    .attr("class", "tooltip")               
    .style("opacity", 0);

var svg = d3.select("body").append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

d3.tsv("totalday.tsv", function(error, data) {
  color.domain(d3.keys(data[0]).filter(function(key) { return key !== "date"; }));

  data.forEach(function(d) {
    d.date = parseDateDay(d.date);
  });

  var routes = color.domain().map(function(name) {
    return {
      name: name,
      values: data.map(function(d) {
        return {date: d.date, number: +d[name]};
      })
    };
  });

  x.domain(d3.extent(data, function(d) { return d.date; }));

  y.domain([
    d3.min(routes, function(c) { return d3.min(c.values, function(v) { return v.number; }); }),
    d3.max(routes, function(c) { return d3.max(c.values, function(v) { return v.number; }); })
  ]);

  svg.append("g")
      .attr("class", "x axis")
      .attr("transform", "translate(0," + height + ")")
      .call(xAxis_day);

  svg.append("g")
      .attr("class", "y axis")
      .call(yAxis)
    .append("text")
      .attr("transform", "rotate(-90)")
      .attr("font-size", 20)
      .attr("y", -45)
      .attr("x", -40)
      .attr("dy", ".71em")
      .style("text-anchor", "end")
      .text("Number of Rides");

  var route = svg.selectAll(".route")
      .data(routes)
    .enter().append("g")
      .attr("class", "route");

  route.append("path")
      .attr("class", "line")
      .attr("d", function(d) { return line(d.values); })
      .style("stroke", "steelblue")
      .style("opacity","0.6")
      .on("mouseover", function(d) {
        d3.select(this).style("stroke-width","6");
        d3.select(this).style("font-size","20");
        d3.select(this).style("opacity","0.9");
        div.transition()        
          .duration(200)      
          .style("opacity", 0.9);      
        div .html(d.name)  
          .style("left", (d3.event.pageX) + "px")     
          .style("top", (d3.event.pageY -28) + "px");})
      .on("mouseout", function(d) {
          d3.select(this).style("stroke-width","1.5"); 
          d3.select(this).style("opacity","0.6");
          div.transition()        
            .duration(200)      
            .style("opacity", 0);})
        .on("click", function(d) { d3.select(this).style("stroke-width","6");

        d3.select(this).style("font-size","20");
        d3.select(this).style("opacity","0.9");
        d3.select(this).style("stroke","darkorange");})


});

function Hour() {

    // Get the data again
    d3.tsv("totalhour.tsv", function(error, data) {
  color.domain(d3.keys(data[0]).filter(function(key) { return key !== "date"; }));

  data.forEach(function(d) {
    d.date = parseDateHour(d.date);
  });

  var routes = color.domain().map(function(name) {
    return {
      name: name,
      values: data.map(function(d) {
        return {date: d.date, number: +d[name]};
      })
    };
  });


    // Scale the range of the data again 
  x.domain(d3.extent(data, function(d) { return d.date; }));

  y.domain([
    d3.min(routes, function(c) { return d3.min(c.values, function(v) { return v.number; }); }),
    d3.max(routes, function(c) { return d3.max(c.values, function(v) { return v.number; }); })
  ]);


    // Select the section we want to apply our changes to
    var svg = d3.select("body").transition();
	    
        svg.select(".x.axis") // change the x axis
            .duration(750)
            .call(xAxis_hour);
        svg.select(".y.axis") // change the y axis
            .duration(750)
            .call(yAxis);

    	d3.selectAll(".line")   // change the line
	    .data(routes)
	    .transition()
              .duration(750)
              .attr("d", function(d) { return line(d.values); });

    });
}



function Day() {

    // Get the data again
    d3.tsv("totalday.tsv", function(error, data) {
  color.domain(d3.keys(data[0]).filter(function(key) { return key !== "date"; }));

  data.forEach(function(d) {
    d.date = parseDateDay(d.date);
  });

  var routes = color.domain().map(function(name) {
    return {
      name: name,
      values: data.map(function(d) {
        return {date: d.date, number: +d[name]};
      })
    };
  });


    // Scale the range of the data again 
  x.domain(d3.extent(data, function(d) { return d.date; }));

  y.domain([
    d3.min(routes, function(c) { return d3.min(c.values, function(v) { return v.number; }); }),
    d3.max(routes, function(c) { return d3.max(c.values, function(v) { return v.number; }); })
  ]);


    // Select the section we want to apply our changes to
    var svg = d3.select("body").transition();

       svg.select(".x.axis") // change the x axis
            .duration(750)
            .call(xAxis_day);
        svg.select(".y.axis") // change the y axis
            .duration(750)
            .call(yAxis);

    	d3.selectAll(".line")   // change the line
		.data(routes)
		.transition()
            .duration(750)
            .attr("d", function(d) { return line(d.values); });

    });
}


function Month() {

    // Get the data again
    d3.tsv("totalmonth.tsv", function(error, data) {
  color.domain(d3.keys(data[0]).filter(function(key) { return key !== "date"; }));

  data.forEach(function(d) {
    d.date = parseDateMonth(d.date);
  });

  var routes = color.domain().map(function(name) {
    return {
      name: name,
      values: data.map(function(d) {
        return {date: d.date, number: +d[name]};
      })
    };
  });


    // Scale the range of the data again 
  x.domain(d3.extent(data, function(d) { return d.date; }));

  y.domain([
    d3.min(routes, function(c) { return d3.min(c.values, function(v) { return v.number; }); }),
    d3.max(routes, function(c) { return d3.max(c.values, function(v) { return v.number; }); })
  ]);


    // Select the section we want to apply our changes to
    var svg = d3.select("body").transition();

        svg.select(".x.axis") // change the x axis
            .duration(750)
            .call(xAxis_month);
        svg.select(".y.axis") // change the y axis
            .duration(750)
            .call(yAxis);

    	d3.selectAll(".line")   // change the line
		.data(routes)
		.transition()
            .duration(750)
            .attr("d", function(d) { return line(d.values); });

    });
}


function Reset() {

    // Get the data again
    d3.tsv("totalday.tsv", function(error, data) {
  color.domain(d3.keys(data[0]).filter(function(key) { return key !== "date"; }));

  data.forEach(function(d) {
    d.date = parseDateDay(d.date);
  });

  var routes = color.domain().map(function(name) {
    return {
      name: name,
      values: data.map(function(d) {
        return {date: d.date, number: +d[name]};
      })
    };
  });


    // Select the section we want to apply our changes to
    var svg = d3.select("body").transition();

      d3.selectAll(".line")   // change the line
            //.data(routes)
            .transition()
            .duration(1)
            .attr("d", function(d) { return line(d.values); })
            .style("stroke", "steelblue");

    });
}


</script>

</html>
