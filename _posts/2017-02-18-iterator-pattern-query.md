---
title: "Using the Iterator pattern to query for data."
tags:
  - java
  - design
---

Everyone knows the Iterator interface in Java but outside the Collections of java the pattern is not often used.

The Iterator pattern is one of the patterns described in the GoF-book. According to them this pattern: 

```
Provides a way to access the elements of an aggregate object sequentially without exposing its underlying representation.
```

Iterator pattern enhances the encapsualtion and promotes single responsibility.

Now let's look at an example of how this pattern can be used.
For example a data source provides some result set when you call it.
This data source could be a database or a web service.
Most often the data source interface will have a method like this:

{% gist codingtim/0cf5b37599a2b84621a5a30237411b1f %}

When the result set is rather large or when we want to retrieve all the data an additional Paging parameter is added.
The paging object will contain information about the limit and offset to use. 

{% gist codingtim/884b657db46af8a4ddf7b1a0ae4ebb09 %} 

The code calling and consuming the result sets would look like this:

{% gist codingtim/1e24c419eefffa7d1b4a171b27627bf5 %}

The problem with this code is that it does more than one thing. 
Not only does it consume the result set, it also needs to know how to call the data provider for a next batch of data.
Now how can this be improved? 

We can use the Iterator pattern to split the business logic and the looping logic. 

First we create a new data source interface that returns an Iterator instead of a List.

{% gist codingtim/f359a87ccb9f173b2a641983e3ef6e5f %}

The Iterator can be backed by a paged implementation similar to the one before. 

{% gist codingtim/bb5ad37e2b57c782a03036b20230faa7 %}

Lastly the user of the interface can now focus on business logic instead of both the paging and the business logic.

{% gist codingtim/55bd6ecbc158fcdebe9cef6aeb4a0e8f %}
