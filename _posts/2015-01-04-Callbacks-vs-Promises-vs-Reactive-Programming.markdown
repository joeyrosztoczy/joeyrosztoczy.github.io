---
layout:     post
title:      Exploring Async Programming on the Web
date:       2015-01-04 17:50:00
summary:    Nested callbacks vs Promises vs Reactive Programming
categories: Reactive-Programming
---

I love Javascript. It has amazing features like lambda's with closure, and **non-blocking
IO with callbacks**. 

Many new JS developers go through a learning curve with the second feature. It's one of the gateways to asynchronous programming.
Asynchronous programming can be puzzling. 

>"What do you mean it's not finishing before this other function I wrote **afterwords**?"
>-*facePalm*

We are often exposed to asynchronous programming by writing callbacks with AJAX to retreive data from a server. But callbacks give a lot of JS developers huge problems, namely [callback-hell][callback-hell-explanation]. 

Usually when dealing with an API we still need to handle data in a specific order in time. Which is literally the opposite of asynchrony.

The first place we often go to restore balance to the force is nesting callback functions within eachother. 

>"Let the error/response propogate up the chain" - they say.
>
>"Only at the end do you realize the power of the Dark Side" - we say.


Once I started building out some halfway serious JS applications, I realized that once you get 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4 or
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;more

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;callbacks 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;deep 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nesting 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gets 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;really 

nasty. 

Then along came promises (back in 1976). [Promises][Promises-detail] are a great way to abstract out and wait for execution of async functions. But they really only shine when we are looking for one return value. This limits their usefulness for async programming beyond stuff like API AJAX calls. 

But don't fear, there's an exciting alternative known as [Reactive Programming (RP)][rp-intro] that seems to be even more powerful than promises and with a design paradigm to match. RP abstracts our code to adhere to the [observer design pattern][about-odp].

So lets compare the three methods of dealing with asynchronous data in order through time. Nesting callbacks, deferring promises, and Reactive Programming using the RxJS library. We will keep it lightweight with code that depends on 3 nested callbacks. We want to asynchronously

1. Get the company our client belongs to
2. Use the company to bring up the clients shops
3. Use the data from the returned shops to look at profit and loss information for one of the shops.

Ah. I see you noticed, we actually need to execute this psuedocode in a specific order (synchronously). Bantha fodder.

Note: The $http functions are from the AngularJS library. Rx functions are from the RxJS library.


##Option 1: Nested Callbacks

{% highlight javascript %}
var whatWeReallyWant;

// 1:
$http.get(/companies, {params: {client_id: client.id}})
 .success(function(clientCompany){
	var clientCompanyUrl = '/companies/' +
	 clientCompany.id + '/shops'
	
	// 2:	
	$http.get(clientCompanyUrl)
	 .success(function(clientShops){
		var specificShop = clientShops.pop()
		var specificShopUrl = '/shops/' + specificShop.id

		// 3:
		$http.get(specificShopUrl)
		  .success(function(shopData){
			// Wooo now we can return shop specific data
			whatWeReallyWant = shopData.quarterlyProfit
		})
	})
})
{% endhighlight %}

**Strengths:** Really no abstraction or design/theory to parse beyond how callbacks work.

**Weaknesses:** Becomes unmaneagable and irritating for any apps beyond tutorials.
				
##Option 2: Deferred Promises

Ok, fine. The Angular $http service really wraps the requests in a promise anyway, so they aren't just pure callbacks. So lets refactor the above code using two patterns: chaining promises and using the $q.all() function to defer resolution on the combined promises.

###Flat Chaining Promises

The key to chaining promises is often overlooked by padawans. If you are using promises, but nesting the .then() functions, pay attention. Promises handlers can **return promises. This is what allows us to chain the .then() method while flattening out the tree.** 
{% highlight javascript %}
var whatWeReallyWant;

// 1:
$http.get(/companies, {params: {client_id: client.id}})
 .then(function(clientCompany){
	var clientCompanyUrl = '/companies/' +
	 clientCompany.id + '/shops'
	
	return $http.get(clientCompanyUrl));
 }) // 2: 
 .then(function(clientShops){
	var specificShop = clientShops.pop()
	var specificShopUrl = '/shops/' + specificShop.id
	
	return $http.get(specificShopUrl);
 }) // 3:
 .then(function(shopData){
	whatWeReallyWant = shopData.quarterlyProfit
 });

// we maintain error propagation and could
// easily add something like .catch()
{% endhighlight %}

**Strengths:** We don't have to climb a nasty nest. '.then()' is almost like a domain specific language that reads very nicely for humans.

**Weaknesses:** This still looks like uncomfortable code. Where does synchronous method chaining ever look this bad, and with so many repeated chains?

###Combining all promises into a single promise with ordered resolution

Each promise comes with an API where we can watch for signalling errors or resolution. $q.all(promise1, promise2) creates a single promise that watches for resolution of promise1, then resolution of promise2, and then resolves itself. This can be really handy if .then() chaining feels contrived.

{% highlight javascript %}
var whatWeReallyWant, clientCompanyUrl, specificShopUrl;

// 1ish:
var clientCompanyPromise = $http
	.get(/companies, {params: {client_id: client.id}})
	.then(function(clientCompany){
		clientCompanyUrl = '/companies/' 
		+ clientCompany.id + '/shops'
	});

// 2ish:	
var clientShopsPromise = $http
	.get(clientCompanyUrl).then(function(clientShops){
		var specificShop = clientShops.pop()
		
		specificShopUrl = '/shops/' + specificShop.id
	})

// 3ish:
var profitPromise = $http
	.get(specificShopUrl)
	.then(function(shopData){
		whatWeReallyWant = shopData
	})

// Resolves promises in the same order
// they are passed into the array
// 1,2,3: 
$q.all(
	[clientCompanyPromise,clientShopsPromise,profitPromise]
).then(
	console.log(whatWeReallyWant);
);
{% endhighlight %}

**Strengths:** Clearly defined functions, much more readable, output is clear.

**Weaknesses:** $q.all() only resolves in the order passed to the array. Gross.

##Option 3: Reactive Programming (RP)
 *Library disclaimer, using RxJS but there are other awesome ones out there as well!*
 
With RP, we set up request and response streams to handle dataflow. These are known as 'observables'. Observables send out notifications in the form of emitted values, errors, or completion signals. Observers receive emitted objects from observables. 
 
We are going to create a request stream to find our company, just one, although the beauty of RP is that it could be a series of requests with pretty much the same code.
 
From that request stream we will generate a response stream that is an Observable that can emit the value the API returns, in the first case it returns the client's company. We extend this logic to get the client's shops, and finally the profit data as well.
 
One of the keys here is the function flatMap(). This takes each emitted value of the stream it is run on and maps them all into a new stream of their own. There are a host of functions in RxJS and RP in general that generate new streams from others. In this case we are generating the streams from values, so it mimics synchronous programming or at least the concept of depending on everything executing in a specific order in time.
 
 
{% highlight javascript %}
var whatWeReallyWant;
var clientCompanyReqStrm = Rx.Observable.just('/companies');

// clientCompanyRespStrm is an observable that will
// return values emitted from the /companies API

// 1:
var clientCompanyRespStrm = clientCompanyReqStrm
 .flatMap(function(clientCompany){
	var clientCompanyUrl = '/companies/' + clientCompany.id +
	 '/shops'
		
	return Rx.Observable.fromPromise($http.get(clientCompanyUrl));
})

// 2:
var clientShopsRespStrm = clientCompanyRespStrm
 .flatMap(function(clientShops){
	var specificShop = clientShops.pop()
	var specificShopUrl = '/shops/' + specificShop.id
	
	return Rx.Observable.fromPromise($http.get(specificShopUrl))
})

// subscribing to a stream creates an observer 
// that notifies us of emitted values and lets us render the data. 

// 3:
clientShopRespStrm.subscribe(function(shopData){
	whatWeReallyWant = shopData;
})
{% endhighlight %}
**Strengths:** Gets us out of the 'lets make it look synchronous mindset' and provides new context from which we can solve async problems. All the functionality of Promises, but doesn't care about single or multi-responses. We can use it for GUI interaction, API calls, WebSockets, all within the same paradigm.

**Weaknesses:** Higher learning curve than the other options.

What I really like about the Reactive Programming style is that there are no mind games. We don't have to chase down weird nested structures. We don't have to maintain a long list of .then() functions for the sake of looking cool. We don't have to use this magical $q.all() function that combines the promises and waits for resolution.

We can use Reactive Programming for all types of asynchronous programming: GUI events, data streaming, you name it.

We aren't pretending to make async programming synchronous for our brains to be happy. We are solving the problem from a totally different angle.

As our async functions emit values, we spin off new stream(s) as the values get emitted, but only when values are actually emitted. 

The physical order of the code above is the actual order of execution.

[callback-hell-explanation]: http://stackoverflow.com/questions/25098066/what-is-callback-hell-and-how-and-why-rx-solves-it
[Promises-detail]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[rp-intro]: https://gist.github.com/staltz/868e7e9bc2a7b8c1f754
[about-odp]: http://en.wikipedia.org/wiki/Observer_pattern