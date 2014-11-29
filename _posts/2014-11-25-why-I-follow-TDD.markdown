---
layout:     post
title:      A note on testing AngularJS
date:       2014-11-25 17:30:00
summary:    How not to get mired in testing hell, start early
categories: testing BDD TDD
---


I was a big proponent of Test Driven Development (TDD/BDD) coming into my latest project.
I was coding in Javascript, and love the Jasmine BDD tool. So, imagine my excitement at how testable I had heard AngularJS was.
**Well I was a big proponent of TDD, I just didn't practice it.**

I started working on a project that introduced
a lot of exploratory coding for me. I was implementing fairly interactive mapping functionality using the amazing
[Leaflet.js library][1].

As I started the project, I thought I would get some code to work, learn how to use the library, and come back 
and implement testing no problem once I was comfortable. 

One proplem, Leaflet.js wasn't the only new tool I was learning how to use.

*   The front-end of the application is in AngularJS, which was new to me. 
*   The backend is in Python with the Django framework. 
*   My background is in Ruby on Rails, with a hefty dash of jQuery to handle my javascript needs. 
*   No, I'm not being a nerd and using jQuery when I have AngularJS at my disposal.

Pre-testing, my head was filled pretty much to the brim with new tools and concepts.

I had a lot of my functionality implemented. I was feeling very comfortable in the leaflet and GIS world as well.
In fact, *I implemented the vast majority of the application before I realized I should probably start testing.*
So today, I sat down to write some unit tests. 

Let's look at that one more time:

> So today, I sat down to write some unit tests.

Yeah, right. 

I quickly found out I had a half dozen new tools to learn: [Protractor][2] and [Karma][3] were some of the finest among them.

It still hurts, just thinking about it. I spent dozens of hours in a test config nightmare. If I had been actually following
anything resembling TDD, I would have come across these tools roughly one at at time, instead I got my brains blasted out.
So here are some lessons I learned for you, each of which have their own tutorial somewhere on the interwebs:

1. Get your test suites configured right away, just like early deployment. 
 	Keep your karma on the light side of the force. Configuring tests suites early will give you the space to learn one new testing tool at a time.

	By the time I actually thought to get started,
	we had 30+ javascript files, 15 controllers, 7 angular modules, and plenty of directives and services. 
	I also had to be preprocessing all of the html templates to test with UI Router. Hurray.

2. Test every new thing you learn, as you learn it. Plenty of tools have specialized needs for unit testing.
	Here is a sample of some of the things I picked up on this project:
	* [inject Angular services into Unit Tests][7] 
	* [testing UI Router][4]
	* [States in UI Router][5]
	* [the $templateCache service][6]
	* [how to fake async calls to an external API][8]  

If you don't test things piece by piece, you will be faced with the monster as I now am.


[1]: http://leafletjs.com/
[2]: https://github.com/angular/protractor
[3]: http://karma-runner.github.io/0.12/index.html
[4]: http://bardo.io/posts/testing-your-ui-router-configuration/
[5]: https://gist.github.com/wilsonwc/8358542
[6]: http://jsg.azurewebsites.net/angular-template-caching-with-templatecache-and-gulp/
[7]: https://docs.angularjs.org/api/ngMock
[8]: https://docs.angularjs.org/api/ngMock/service/$httpBackend