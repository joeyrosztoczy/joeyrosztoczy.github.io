---
layout:     post
title:      Unique Together (PartyGoer Ecto/Elixir Model)
date:       2015-06-27 16:30:00
summary:    Implementing unique together to only allow one Party/User pair per party.
categories: Elixir
---
### One User per Party with Unique Together implementation

When a user joins a party, we POST a new PartyGoer. Unfortunately,
 we only want one User/Party combination at a time. I'm going to do a few blog posts on implementing this with different methods.

There are a lot of built in validators in Ecto, the database/query wrapper that works hand in hand with Phoenix. One of those is [validate_unique/3](http://hexdocs.pm/ecto/Ecto.Changeset.html#validate_unique/3). It takes a model changeset, a field to test uniqueness on, and an options map.

It wasn't immediately obvious to me how to leverage this to get unique_together functionality looking at both the Party and User fields. However, the last atom on the options map is ```:scope```. ```:scope``` allows us to include other fields in the uniqueness check. That leaves a unique_together validation for a PartyGoer looking as simple as:

~~~ ruby
  def changeset(model, params \\ :empty) do
    model
    |> cast(params, @required_fields, @optional_fields)
    |> validate_unique(:user_id, scope: [:party_id],
     on: GoodTimes.Repo)
  end
~~~  
