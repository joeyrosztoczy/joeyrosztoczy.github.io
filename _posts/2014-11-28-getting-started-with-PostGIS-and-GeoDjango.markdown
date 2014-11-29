---
layout:     post
title:      Getting Started with PostGIS and GeoDjango
date:       2014-11-28 17:30:00
summary:    And why you should suck it up and use a spatial database
categories: django geojson python GIS
---

If you are thinking about writing a location-based application, or implementing any type of geospatial mapping on the web, 
[PostGIS][1] and [GeoDjango][2] are two of the greatest tools out there.
They are feature-rich and simple to use. 
This is a tutorial on getting started with GeoDjango and PostGIS. It starts from assuming you have a shell Django project or existing project. 
If you don't, check out their [awesome docs][5].

These two tools provide some serious awesomeness, specifically: *spacial queries from the django ORM*. 

I started out my project trying to use lighter weight alternatives like django-geojson. 
Trust me, its worth a little bit of extra effort to implement the real deal.

So, let's get started with configuring postGIS. Since you are naturally using [Homebrew][3], its pretty straightforward. 

		// Update homebrew first (of course)
		$ brew update
		$ brew doctor

		// Now lets get at that spatial database
		$ brew install postgis
		$ brew install gdal

After updating Homebrew, this gives us (1) PostgreSQL and its spatial extension PostGIS and (2) [GDAL][6] the geospatial data abstraction library. 
Once we have these backend dependencies installed, we also need to remember to get the psycopg2 library, it provides a Python adapter for PostgreSQL. 
FYI, I use [virtual env][4] to contain each of my django projects, but I'm going to ignore moving around in those for this tutorial.

		$ pip install psycopg2

Now we create the database:

		$ createdb `whoami`
		$ psql

		=# create database geometry_practice;
		=# \q

		$ CREATE EXTENSION postgis;

And with the database configured, we edit our Settings.py file to use the proper backend engine:

		'ENGINE': 'django.contrib.gis.db.backends.postgis',

And add ```'django.contrib.gis',``` to INSTALLED_APPS in the Settings.py file!

Alright, now we have an awesome backend to handle our Geometry objects and models. GeoDjango wraps the GEOS C++ library for use in Python! 
Personally, I interact with data on the front-end primarily in geoJSON, and it supports conversion to and from geoJSON.


It also means we can now do spacial queries on our geometry models such as filtering by bounding boxes, 
checking for overlap or intersection, or filter objects by distance from a point! Since we are using PostGIS, we can access
**every single spacial query that Django offers**. Check out [this comparison table][7].



Other solid tools for more advanced use cases that I might write about in the future:

1. [Leaflet](http://leafletjs.com/), an awesome choice for the front-end face of geospatial applications.
2. [ogr2ogr](http://www.gdal.org/ogr2ogr.html) a GDAL tool for converting between spatial file types/data types.
3. The [gis extension for the django-rest-framework](https://github.com/djangonauts/django-rest-framework-gis) that allows easy serialization to and from GeoJSON.


[1]: http://postgis.net/features
[2]: https://docs.djangoproject.com/en/dev/ref/contrib/gis/
[3]: http://brew.sh/
[4]: http://www.jeffknupp.com/blog/2012/02/09/starting-a-django-project-the-right-way/
[5]: https://docs.djangoproject.com/en/1.7/intro/tutorial01/
[6]: http://www.gdal.org/
[7]: https://docs.djangoproject.com/en/dev/ref/contrib/gis/db-api/#compatibility-tables