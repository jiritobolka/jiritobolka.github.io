---
layout: post
title:  "Get the Data from GoodData Project"
date:   2014-08-04 11:00:00
categories: tutorial
highlighter: true
prev_section: tutorials
next_section: tutorials/#set-up-viz
perex: learn how to get the data from GoodData Platform.
---

Open your GoodData Project and create a **metrics** that you want to visualize. You will need the metrics identifier (not URI!) to specify object for export. The easiest way to get the metric identifier is using our "grey pages". Go to the 

`https://secure.gooddata.com/gdc/md/{project-id}/query/metrics` 

then find your metric, open it and find “identifier” key and use it's value (store it somewhere for later usage in your Javascript code). You can definitely use API directly to get metrics identifiers.

We are going to use following metrics:

**Won Amount**: atX3I1GYg85J  
**# Activities** : acKjadJIgZUN  
**# Opportunities**: afdV48ABh8CN  
**# Won Opportunities**: abf0d42yaIkL  

We want to see the comparison of our sales reps based on metrics above. Select the sales rep **attribute** identifier as well from the Project:

**Opportunity Owner**:  label.opp_owner.id.name

Open the javascript file ([example file](link-to-example-package) you've downloaded) now and edit the first 10 rows. Fill in your Project ID, Credentials, Metrics and Attribute identifiers. See the code below:

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

Now, it is all about the Javascript methods that call GoodData APIs - authenticate and extract the data based on provided metrics and attributes. See the following script, especially `gooddata.user.login` method and `gooddata.execution.getData`.

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

You have successfully extracted the data from GoodData Platform and now having all data in the browser, you are able to use the D3.js (or any other viz framework) to draw the custom visualization. The structure that is being returned by the `getData()` method is an object. This object contains of two arrays:

- headers: array of objects with header information and data
- rawData: array with the data for each row
- isLoaded: parameter that you can check for better handling (true/false)

![Data Object Structure](/images/posts/data-object.png)

Basically, it is up to you how to process the data that are extracted from the GoodData Platform with the SDK. Anyway, we will show you multiple examples to inspire you.

