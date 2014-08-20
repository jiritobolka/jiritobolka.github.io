---
layout: post
title:  "Building Parallel Coordinates"
date:   2014-08-15 11:00:00
categories: example
highlighter: true
prev_section: build-visualization/
next_section: example/chord-chart-to-analyze-sales
perex: Right place to start building amazing custom data visualizations.
---

Let's learn how to create a custom visualization with our Javascript Visualization SDK and embed it to your dashboard. The tutorial is divided into several logical parts based on steps that you have to accomplish. You can also <a href=”link-to-download-package”>download the complete example</a>.   

What we are going to build is the visualization that is called Parallel Coordinates. It shows one attribute measured by multiple metrics. Imagine a following use case:

“Chief Revenue Officer wants to compare Sales Reps not only by the Revenue but also by their activity and number of Opportunities and one other metrics.”

The result might looks like this:

![Parallel Coordinates graph](/images/posts/parallel-coordinates.png)

Remember that each line shows individual sales rep and each axis shows one metric that is used to measure sales representatives. We have a use case so let's build it!

To Run the Visualization directly:

1. [Download the example](link-to-the package) to your working directory
2. Edit your credentials and select your metrics/attributes you want to use in visualization 
3. Run the `grunt dev` command in you javascript SDK repository
4. Go to the following URL:  https://localhost:8443/parallel-coordinates/ to see it in action
5. Optionaly you can embed it to your dashboard. Some extra steps needs to be done to achieve it.

If you are totally new to the GoodData Visualization SDK, let's follow the steps in more detail. We will start with getting the data from GoodData Project.

!!! GETTING THE DATA

Now, it's time to start building the D3 visualization itself. If you are an expert in building the D3 custom visualization, this article will be super easy for you! We are going to draw the Parallel Coordinates chart to compare one attribute across four different metrics. 

If you have some Visualization prepared, you can skip this and continue with [embedding process](link-to-next-chapter).

{% highlight js %}

var m = [30, 10, 10, 10],
	w = 960 - m[1] - m[3],
    h = 500 - m[0] - m[2];

var x = d3.scale.ordinal().rangePoints([0, w], 1),
    	y = {},
    	dragging = {};

var line = d3.svg.line(),
    	axis = d3.svg.axis().orient("left"),
    	background,
    	foreground;

var svg = d3.select("body").append("svg:svg")
    	.attr("width", w + m[1] + m[3])
    	.attr("height", h + m[0] + m[2])
    	.append("svg:g")
    	.attr("transform", "translate(" + m[3] + "," + m[0] + ")");
{% endhighlight %}

The main method that draw the visualization.

{% highlight js %}

// the main method that extract data and draw the visualization
var parallel = function(dataResult) {

		// extract header titles for axis description    
        var headers = dataResult.headers.map(function(h) {
                return h.title;
            });
    		    
    		    // delete the attribute (first in returned array) description from header array - you need just metrics description
                headers.splice(0,1);
        
    	// Extract the list of dimensions and create a scale for each.  
    	x.domain(dimensions = d3.keys(dataResult.rawData[0]).filter(function(d) {
    	 	return d != "0" && (y[d] = d3.scale.linear()
    	 .domain(d3.extent(dataResult.rawData, function(p) { return +p[d]; }))
    	 .range([h, 0]));
    	 }));

    	  // Add grey background lines for context.
    	  background = svg.append("svg:g")
    	      .attr("class", "background")
    	    .selectAll("path")
    	      .data(dataResult.rawData)
    	    .enter().append("svg:path")
    	      .attr("d", path);
    	 
    	  // Add blue foreground lines for focus.
    	  foreground = svg.append("svg:g")
    	      .attr("class", "foreground")
    	    .selectAll("path")
    	      .data(dataResult.rawData)
    	    .enter().append("svg:path")
    	      .attr("d", path);
    	 
    	  // Add a group element for each dimension.
    	  var g = svg.selectAll(".dimension")
    	      .data(dimensions)
    	    .enter().append("svg:g")
    	      .attr("class", "dimension")
    	      .attr("transform", function(d) { return "translate(" + x(d) + ")"; })
    	      .call(d3.behavior.drag()
    	        .on("dragstart", function(d) {
    	          dragging[d] = this.__origin__ = x(d);
    	          background.attr("visibility", "hidden");
    	        })
    	        .on("drag", function(d) {
    	          dragging[d] = Math.min(w, Math.max(0, this.__origin__ += d3.event.dx));
    	          foreground.attr("d", path);
    	          dimensions.sort(function(a, b) { return position(a) - position(b); });
    	          x.domain(dimensions);
    	          g.attr("transform", function(d) { return "translate(" + position(d) + ")"; })
    	        })
    	        .on("dragend", function(d) {
    	          delete this.__origin__;
    	          delete dragging[d];
    	          transition(d3.select(this)).attr("transform", "translate(" + x(d) + ")");
    	          transition(foreground)
    	              .attr("d", path);
    	          background
    	              .attr("d", path)
    	              .transition()
    	              .delay(500)
    	              .duration(0)
    	              .attr("visibility", null);
    	        }));
    	 
    	  // Add an axis and title.
    	  g.append("svg:g")
    	      .attr("class", "axis")
    	      .each(function(d) { d3.select(this).call(axis.scale(y[d])); })
    	    .append("svg:text")
    	      .data(headers)
    	      .attr("text-anchor", "middle")
    	      .attr("y", -9)
    	      .text(String);
    	 
    	  // Add and store a brush for each axis.
    	  g.append("svg:g")
    	      .attr("class", "brush")
    	      .each(function(d) { d3.select(this).call(y[d].brush = d3.svg.brush().y(y[d]).on("brush", brush)); })
    	    .selectAll("rect")
    	      .attr("x", -8)
    	      .attr("width", 16);
    };
{% endhighlight %}		

Call the main method that draw the visualization.

{% highlight js %}

		// calling the main method
		parallel(dataResult);   

{% endhighlight %}		

Finalize the dragging and transition effects. 

{% highlight js %}

		function position(d) {
			var v = dragging[d];
			return v == null ? x(d) : v;
			}
 
			function transition(g) {
				return g.transition().duration(500);
			}
 
			// Returns the path for a given data point.
			function path(d) {
			  return line(dimensions.map(function(p) { return [position(p), y[p](d[p])]; }));
			}
			
			// Handles a brush event, toggling the display of foreground lines.
			function brush() {
			  var actives = dimensions.filter(function(p) { return !y[p].brush.empty(); }),
			      extents = actives.map(function(p) { return y[p].brush.extent(); });
			  foreground.style("display", function(d) {
			    return actives.every(function(p, i) {
			      return extents[i][0] <= d[p] && d[p] <= extents[i][1];
			    }) ? null : "none";
			  });
			}

		});
});

{% endhighlight %}

That's all. [Download the full example](link-to-the-example) to try it out easily.

