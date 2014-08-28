---
layout: post
title:  "Get the Data from GoodData Project"
date:   2014-08-04 11:00:00
categories: tutorial
highlighter: true
prev_section: build-visualization
next_section: build-visualization/#set-up-viz
perex: learn how to get data from GoodData Platform.
---

Open your GoodData Project and create the metrics that you want to visualize. You will need identifiers (not URIs!) to specify the objects to export. The easiest way to get the metric identifier is to use our grey pages here: 

`https://secure.gooddata.com/gdc/md/{project-id}/query/metrics` 

navigate to your metric, open it and locate its identifier key and copy its value. You can also use API directly to get metric identifiers.

In our example we want to compare our sales reps using the following metrics:

**Won Amount**: atX3I1GYg85J  
**# Activities** : acKjadJIgZUN  
**# Opportunities**: afdV48ABh8CN  
**# Won Opportunities**: abf0d42yaIkL  

Select the sales rep attribute identifier from the project:

**Opportunity Owner**:  label.opp_owner.id.name

Now, open the javascript [file](link-to-example-package) that you've downloaded and edit the first 10 rows. Fill in your project ID, credentials, metrics and attribute identifiers. See the code below:

{% highlight javascript %}

var projectId = 'PROJECT_ID', 
      user = 'USERNAME,
	    passwd = 'PASSWORD';

// Report elements identifiers from which we execute a GD report
var metric1 = 'METRIC-IDENTIFIER',
      metric2 = 'METRIC-IDENTIFIER',
      metric3 = 'METRIC-IDENTIFIER',
      metric4 = 'METRIC-IDENTIFIER',
      attr1 = 'ATTRIBUTE-IDENTIFIER';

{% endhighlight %}

The next step is all about the Javascript methods that call the GoodData APIs to authenticate and extract the data based on the metrics and attributes from the code above. 

See the following script, particularly the `gooddata.user.login` and `gooddata.execution.getData` methods.

{% highlight javascript %}
var elements = [attr1, metric1, metric2, metric3, metric4];

// Insert info label
$('body').append('<div class="login-loader">Logging in...</div>');

gooddata.user.login(user, passwd).then(function() {

    $('div.login-loader').remove();
    $('body').append('<div class="loading">Loading data...</div>');

    // Ask for data for the given metric and attributes from the GoodSales project
    gooddata.execution.getData(projectId, elements).then(function(dataResult) {
        // Yay, data arrived

        // Remove loading labels
        $('div.loading').remove();

{% endhighlight %}

You have successfully extracted the data from GoodData Platform and now have all data that you have specified in the browser. You can now use the D3.js (or any other viz framework) to create the custom visualization. The structure that the `getData()` method returns to you is an object. This object contains of two arrays:

- headers: array of objects with header information and data  
- rawData: array with the data for each row  
- isLoaded: parameter that you can check for better handling (true/false)  

![Data Object Structure](/images/posts/data-object.png)

Perfect. Your data is extracted from the GoodData Platform with the SDK, and now it's up to you how you transform it. We have created [multiple examples](link-to-examples) to inspire you.

