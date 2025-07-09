+++
title = 'Do NOT Pickle in Dev'
date = 2025-04-29T11:11:38+10:00
draft = true
+++

The standard library object serialization library in python (`pickle`) provides a
simple interface for writing and loading python objects to disk. This could be useful for
saving a machine learning model that took a long time to train (although a mature ML libray
would probably offer this functionality directly) or interprocess communication, when you might
want to pass an object from one python process to another (although again, one feels that there
would be a better way).

I recently used it to implement a simple-as-possible database during early development of a
project and discovered a horrible new kind of bug. It goes like this:

### 1. Define and Pickle
First you define some class, create an instance of that object and then pickle it.
```python
import pickle

class Person:
    def __init__(self, name):
        self.name = name

# In some other file:
me = Person("Hugo")

with open("persistant_me", "wb+") as f:
    pickle.dump(me, f)
```

### 2. Continue Development and Unpickle
Later you realise that the `Person` class was not finished and you extend it:
```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

Pickle will allow you to load the object that you serialised earlier, since the pickled
object's class name matches one in the namespace:
```python
with open("persistant_me", "rb") as f:
    me = pickle.load(f)

print(type(me))
print(me.age)
```
which gives you `<class '__main__.Person'>` as expected and
> AttributeError: 'Person' object has no attribute 'age'

which can be quite confusing when you look at the class definition and see that
it very clearly does have this attribute!


