---
layout: post
title:  "Dynamic dc.js Visualization"
date:   2014-08-14 11:00:00
categories: example
highlighter: true
prev_section: /example/chord-chart-to-analyze-sales
perex: Learn how to pull data from GoodData and visualize it in D3.js Chord chart 
---

Talking about visualization frameworks and libraries the [dc.js](http://dc-js.github.io/dc.js/) together with [Crossfilter](https://github.com/square/crossfilter/wiki/API-Reference) is great combo on the top of D3.js framework. There is a solid amount of examples around the internet so that you can start playing around immediately.

I took the example from the dc.js homepage which looks pretty cool and is [greatly and fully described](http://dc-js.github.io/dc.js/docs/stock.html) and try to bring it to the next level. What I did? The original example is based on static CSV file. What is really cool? You can quite easily connect it to the GoodData Platform, get the data from your Project and use the visualization embedded to your dashboard. This is what we called **Visualization Extensibility**.

The potential is huge! If you miss some visualization, just build it and embed it directly to the GoodData Project!

<img src="/images/posts/dc-js-example.png" alt="Dynamic Visualizations" />

[Download the full example]() to see how it looks connected to the GoodData.   

## Prerequisites 

1. Downloaded [dc.js](http://dc-js.github.io/dc.js/) & [Crossfilter](https://github.com/square/crossfilter/wiki/API-Reference)
1. Have the data in the GoodData Project that you can visualize

## Integration

The main change is in the beggining of the script, you basically specify which metrics are being extracted from the GoodData:

{% highlight js %}

var projectId = 'project-id', 
    user = 'username@company.com',
	passwd = 'password';

// Report elements identifiers from which we execute a GD report
var metric1 = 'aiAY9GSReqiT',
	metric2 = 'aaEY9xX2e5FE',
	metric3 = 'aa0Y9Pe6etWb',
	metric4 = 'aHZY9nzNeg3f',
    metric5 = 'aitZm37nbyhn',
    attr1 = 'date.date.mmddyyyy';
    
var elements = [attr1, metric1, metric2, metric3, metric4, metric5];

{% endhighlight %}

Remember that order you specify in the elements array is the same that the elements will be exported from the GoodData.

Let's download the data now. It is the same as for our othe examples. We need to do a little work with the output so that it fits the Crossfilter library.

{% highlight js %}

gooddata.user.login(user, passwd).then(function() {

    $('div.login-loader').remove();
    $('body').append('<div class="loading" style="text-align: center; margin: 30px;">Loading data...</div>');

    // Ask for data for the given metric and attributes from the GoodSales project
    gooddata.execution.getData(projectId, elements).then(function(dataResult) {
        // Yay, data arrived
        
        var headers = dataResult.headers.map(function(h) {
                    return h.title;
                }),
		data = [];
		
		//Object -> Array so that dc.js can consume it
		dataResult.rawData.forEach(function (arr) {
					var rv = {};
					for	(var i = 0; i < arr.length; ++i)
					rv[i] = arr[i];
					data.push(rv);
					});
					
		// Remove loading labels
        $('div.loading').remove();
		
		var dateFormat = d3.time.format("%m/%d/%Y");
		var numberFormat = d3.format(".2f");

		data.forEach(function (d) {
    		d.dd = dateFormat.parse(d[0]);
    		d.month = d3.time.month(d.dd); // pre-calculate month for better performance
    		d[4] = +d[4]; // coerce to number
    		d[1] = +d[1];
    		});
    		
{% endhighlight %}

The rest of the script is basically the same as in the original example so that you can read the details. Only change is the name on keys in the objects because the Javascript SDK returns you numeric keys so the names are different. 

[Download the full example](https://github.com/gooddata/gooddata-js/tree/develop/examples/dc-js) to start playing around! You can also find it directly in your repository (_example_ folder).

