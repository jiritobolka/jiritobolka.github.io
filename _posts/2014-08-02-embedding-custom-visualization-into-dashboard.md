---
layout: post
title:  "Embedding Viz into the Dashboard"
date:   2014-08-02 11:00:00
categories: tutorial
highlighter: true
prev_section: tutorials
next_section: tutorials/#examples
perex: learn how to setup D3 visualization.
---

### How it works

You store the custom visualization script on a remote location (i.e. S3 bucket) where you can execute it and access it from the browser.

<img src="/images/posts/embedded-js-viz.png" width="650" />

As a final step, I can embed the custom visualization to the GoodData Dashboard in the same way I embed any other custom HTML code. 

Unfortunately, there is some limitation:

- Whitelabeling feature is activated to my Organization (I have custom domain i.e. _analytics.mycompany.com_)  
- I'm hosting the code in my storage (GoodData doesn't provide hosting solution)

We allow [CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) per request for your Organization.

### Set up on your side:

Let's asumme I'm using the Amazon S3 to store my custom visualizations. 

1) Put the code to the web server (i.e. Amazon S3 bucket) where the custom visualization will live. Add following line of code to set your custom GoodData domain.

{% highlight js %}
// Fill in your gooddata CORS enabled url (e.g. https://gd.domain.tld)
gooddata.config.setCustomDomain('<your_gooddata_endpoint>');
{% endhighlight %}

2) Embed the code to the Dashboard (link to your S3 bucket)  

<img src="/images/posts/dashboard-embed-dialog.png" width="600" alt="Embedding Dialog" />

Once CORS is set up you are able to place any other visualization to your S3 bucket and use it in the same way.

### Local Development

You don't need to have CORS allowed when you are running the script using `localhost` and `grunt` task. It is automatically set up to access the secure.gooddata.com or the server that you choose. See the diagram to understand the difference.

<img src="/images/posts/localhost-development.png" width="650" />
