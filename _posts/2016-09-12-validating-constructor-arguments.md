---
title: "Validating constructor arguments."
tags:
  - java
  - design
---

When instantiating an object how, and when, should we validate that the arguments given are valid?

The most common approach these days seems to be: validate the values in the service and pass the values to the constructor of the class. 
Following this approach we end up with an anemic domain model which is why I don't prefer this approach. 
Another possibility is to call the constructor and after the object is instantiated call a validate method on the object. 
This approach has the downside that an invalid object can be alive. 
If a programmer forgets to call the validate method after instantiation we could end up with weird results.

To me the proper place to do the validation is in the constructor itself. 
Let's consider the following java class. 
The class is called Target and has three fields. 
Code and name are pretty self explanatory.
The publicIdentifier is something that will be used in a url so it should not contain any special characters.

{% gist codingtim/0332392086511eff19e4cf97e19bc760 %}

Now if we would like to add some validation we could end up with something like this: 

{% gist codingtim/37a82623dbfbe61a7bb264fe71522689 %}

Of course the validate method could move to a utility class so it can be reused in multiple classes. 
However this is not the only remark I have with this code. 
We are still passing three Strings into the constructor as arguments. 
These Strings have no business logic and can be swapped around without producing a compile error.

It also reminded me of the following quote of some speaker at DDD Europe: 

```
All the important stuff is modeled as String (or with a primitive type).
```

If we do something about this and model the important stuff differently we end up with:

{% gist codingtim/9de419c79f3ca3cc814190d9ce692ee7 %}

Later I will add javadoc to the class so we know why the regex is there and what the class should do.

This is just my current approach; thoughts, improvements or other mind blowing suggestions are always welcome! 
