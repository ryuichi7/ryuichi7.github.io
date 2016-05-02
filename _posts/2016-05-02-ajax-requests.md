---
layout: page
#
# Content
#
subheadline: "AJAXify yourself"
header: false
title: "AJAX requests"
teaser: "learn some of the syntax and shorthand for making jQuery AJAX requests"
categories:
  - blog	
#tags:
#  - 
---

Recently, I've been introduced to javascript and the world of ansynchronous HTTP requests. Asynchronous JavaScript and XML, or AJAX for short, is a concept that allows you to make requests to a server and render the response in the DOM without reloading the page. The javascript library, jQuery, possesses a handy function just for these purposes, `jQuery.ajax()` .

###AJAX and callbacks

First, let's dicuss a little about how AJAX calls are made and how they utilize callbacks to render information. Normally when a request is made to a server, the whole page has to referesh to render the response. With AJAX, an XMLHttpRequest object is created and a request is made to a server but the DOM (Document Object Model) remains "unlocked" while waiting for the response. In turn, the user is free to continue browsing while server requests are being made. Once the response is returned, AJAX handles said response through the use of callbacks. 

### A little about callbacks...
A callback is a function that can be passed to another function and only fires upon being called. Callbacks are an integral part of javascript and stem from the functional programming paradigm. If you want to learn more about callbacks, there is a great article about them [here](http://javascriptissexy.com/understand-javascript-callback-functions-and-use-them/).

 
Now, when a response is received, a callback is fired and some type of action is performed. 

for example:

{% highlight js %}
$.ajax({
	url: 'someapi@website.com',
	type: 'get',
	dataType: 'json'
}).done(function(data) {
	//some awesome code here
}).fail(function(error){
	alert("there was an error!");
	console.log(error);
});
{% endhighlight %}

### How the call is made

Let's go through this snippet one by one.

If you haven't used jQuery before, the `$` is shorthand for jQuery. We call the `$.ajax()` method and pass it some settings in the form of key/value pairs. 

The `url` parameter, as it's name indicates, contains the url of the address you want to make the request to. This can be something within your app as well. For instance if you wanted the contents of a file 'things.html', in the root of your project, you could make a request to `url: 'things.html'` and you'd be returned an object containing the html on that page.

The `type` parameter states what type of HTTP request you'd like to make. By default jQuery sends a 'GET' request.

The `dataType` parameter lets you specify the dataType you'd like to receive. jQuery's Intelligent Guess looks for xml, json, script, or html by default.

These are just a few of the settings you can pass to the `$.ajax()` function. You can see a much more comprehensive list at [JQuery's api](http://api.jquery.com/jquery.ajax/)

### The response

`$.ajax()` returns a jQuery XMLHttpRequest(jqXHR) object upon which a number of methods can be called. The most common `jqXHR.done()` and `jqXHR.fail()` that are used in the example, allow you to deal with responses and fire any number of callbacks upon request completion. `jqXHR.done()` is called upon success of a request and returns data, statusText, and a jqXHR object. Note that the names of these arguments don't matter, just the order.

{% highlight js %}
$.ajax({
	url: 'someapi@website.com',
	type: 'get',
	dataType: 'json'
}).done(function(response, textOfStatus, elmo) {
	// response refers the data recieved
	// textOfStatus refers to statusText i.e. success
	// elmo points to the jqXHR object recieved
});
{% endhighlight %}

You can name them however you feel but the standard data, statusText, and xhr naming conventions are pretty explanatory and good practice to stick to.

On the off chance that your request fails, it's a good idea to have a way to handle the error so that the user is aware. That's where `jqXHR.fail()` comes in. If a request does not return a status of 200 the `jqXHR.done()` method is skipped over and the
`jqXHR.fail()` method is called. Here you can handle errors such as appending a message to the DOM notifying the user.

### Shorthand

`$.ajax()` also comes with a number of shorthand methods that make clean, succinct calls easier to accomplish. Instead of naming the `type` key/value pair in the settings passed to `$.ajax()`, you can call `$.get()` or `$.post()` as shorthand. Note that there is no shorthand for puts, patch, or delete. Those types must be explicitly stated in the settings. Here's an example of how we can clean up our original GET request example:

{% highlight js %}
$.get('someapi@website.com', function(response) {
	//some awesome code here
}).fail(function(error){
	alert("there was an error!");
	console.log(error);
});
{% endhighlight %}

As you can see we are able to omit a lot of the settings and create a much cleaner call to the server. Although this isn't always the best option, it's a nice, simple way to clean up our code.


### Conclusion

I hope that this gives a little insight as to the innerworkings of the `jQuery.ajax()` function and a little clarification on the syntax. I've only given a very simple example here and will explore more complex calls to various api's in future posts. Now go ajaxify yourself!



