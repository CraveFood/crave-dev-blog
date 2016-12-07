---
layout: post
title:  "Generic Relations to create a comment app"
date:   2016-12-06 22:10:46 -0300
comments: true
categories: python django models

---



When we have a foreign key, we are linking an instance of another model in this model. Right? So, we can access that other instance and other model very easily. So it would work like this:

```
class Author(models.Model):
    name = models.CharField(max_length=50)

class Book(models.Model):
    author = models.ForeignKey(Author)
    title = models.CharField(max_length=250)
    pages = models.IntegerField()
```

So, you have an instance of Author associated with all the information I have on my Book instance. 
Ok, this is cool because you can store several books, all linked to the same author. 

Imagine now that you want the same thing: associate an instance of a model with more information. But instead of having just an instance of Author (for example), you want the same information through several model classes. 

Here we have the GenericRelations! In a simple way we can say that GenericRelation is a foreign key, 
but with you can store any instance of any of your models.

And now you are asking yourself: why is this useful? You can simply add more fields in your original model or something like this. 
Yes, this is true in most cases. However, for some specific cases, Generic Relations can be really handy. 
In our case, we needed the hability to add comments in two apps of our system. We needed the same functionality, 
in a way it would be easy to mantain and without duplicating code: perfect time for GR.

We started by creating an Comment model such as:

```python
from django.contrib.contenttypes import generic
from django.contrib.contenttypes.models import ContentType
from django.db import models

class Comment(models.Model):
    content_type = models.ForeignKey(ContentType)
    object_id = models.Charfield(max_length=50)
    content_object = generic.GenericForeignKey('content_type', 'object_id')
    text = models.TextField(blank=True)
    message_from = models.ForeignKey(User)
```

On this model the `text` and `message_from` fields are normal fields from django, added as part of the message information.
The other fields, `content_type`, `object_id` and  `content_objects` are part of the Generic Relation we are adding here. 

Instances of ContentType represent and store information about the models installed on your project. 
Everytime a new model is created, new instances of ContentTypes are automatically created. 
Here, the `content_type` will be a Foreign Key to the model you want to associate. 
The `object_id`, by the other end, is a simple Charfield that will store an id of an object that 
is stored in your model. 

You have the model, you have the id of the object you want to access... so the `content_object` will actually represent the instance of that object on that model. The GenericForeignKey does the magic to you!
