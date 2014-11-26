---
layout:     post
title:      Why I follow Test-Driven-Development
date:       2014-11-25 17:30:00
summary:    Wild Goose Chases for days
categories: testing, BDD, TDD
---


I was a big proponent of Test Driven Development (TDD/BDD) coming into my latest project.
I was coding in Javascript, and love the Jasmine BDD tool.
The I was a big proponent of TDD, I just didn't practice it.

I started working on a project that introduced
a lot of exploratory coding for me. I was implementing fairly interactive mapping functionality using the amazing
[Leaflet.js library][1].

As I started the project, I thought I would get the code to work, learn how to use the library, and come back 
and implement testing no problem once I was comfortable. However, Leaflet.js wasn't the only new tool I was learning how to use.
The front-end of the application is in AngularJS, which was new to me. The backend is in Python with the Django framework. My background 
is in Ruby on Rails, with a hefty dash of jQuery to handle my javascript needs. 
Pre-testing, my head was filled pretty much to the brim with new tools and concepts.

I had a lot of my functionality implemented. I was feeling very comfortable in the leaflet and GIS world as well.
In fact, *I implemented the vast majority of the application before I realized I should probably start testing.*
So today, I sat down to write some unit tests. Let's look at that one more time:

> So today, I sat down to write some unit tests.

It still hurts, just thinking about it. I spent dozens of hours in a test config nightmare.
So here are some lessons I learned for you, at my own expense:

1. Get your test suites configured right away, just like early deployment. 
 	* Get your webdrivers going, your karma on the light side of the force.
	I learned a lot about Karma.conf.js today, because by the time I needed to get Karma to work,
	we had 30+ javascript files, 15 controllers, 7 angular modules, and plenty of directives and services.

	This wild goose chase included learning about: ng-html2js preprocessor and splats, thanks to the fact that I am using UI-router.

2. Test every new thing you learn, as you learn it. Plenty of tools have specialized needs for unit testing.
	Here is a sample of some of the things I picked up on this project:
	* States in UI Router
	* the $templateCache service
	* ngMock 
	* $httpBackend service - for mocking async calls to our API  


3. If you don't test things piece by piece, you will be faced with the monster as I now am:
	* I have to mock out our authentication endpoints
	* I have to mock all of the config endpoints


[1]: http://leafletjs.com/