---
layout:     post
title:      GoodTimes's Setup (Beginning Elixir)
date:       2015-06-27 16:30:00
summary:    This post sets the stage for microposts on beginning elixir topics.
categories: Elixir
---

Source: [Good Times App](https://github.com/last-night/good-times)

Today was my first time playing around with the Phoenix framework.
 [Phoenix](http://www.phoenixframework.org/v0.13.1) is a beautiful convention over configuration framework
 for the [Elixir](http://elixir-lang.org/) programming language. If you like Rails, you'll love Phoenix.
  If you don't like Rails, Elixir and functional programming paradigms do a lot to improve on the experience, so give it a chance.

I wanted to reimplement a simple CRUD app for drinking games I had built in Sinatra about a year ago. I'm going to use this idea to model a lot of beginning Elixir and Phoenix concepts. 

### The Setup:


1. **Users** can login to the app.
2. User's can create **Parties** that they and other users can join
3. a user can be a part of many parties (and a party has many users). In addition we can drink at a party. Let's call this join table **PartyGoer**.

A PartyGoer is a join between a User, and a Party. Here's what the relationships look like on each model:

~~~ ruby
	# web/models/user.ex

	defmodule GoodTimes.User do
	  use GoodTimes.Web, :model

	  schema "users" do
	    has_many :party_goers, PartyGoer
	    has_many :parties, through: [:party_goers, :parties]
	    ...
    
	# web/models/party.ex    

	defmodule GoodTimes.Party do
	  use GoodTimes.Web, :model

	  schema "parties" do
	    has_many :party_goers, PartyGoer
	    has_many :users, through: [:party_goers, :users]

	    field :name, :string
	    field :location, :string

	    timestamps
	  end
	  ...
	  
	#web/models/party_goer.ex

	defmodule GoodTimes.PartyGoer do
	  use GoodTimes.Web, :model

	  schema "party_goers" do
	    belongs_to :party, GoodTimes.Party
	    belongs_to :user, GoodTimes.User

	    field :drink_count, :integer

	    timestamps
	  end
	  ...
~~~

When using the has_many through syntax to define a relationship, we have to previously define the join table we will be going through. For the Party model, this means we define the join table first, and then the has_many through relationship to Users.

Notice the atoms used in the association struct ```has_many :users, through: [:party_goers, :users]``` line. This tells us we are going to refer to the relation data via ```Part.users```, but to get there we need to tell it which table we are going through ```:party_goers``` and which table from the join we need to end up at ```:users```.