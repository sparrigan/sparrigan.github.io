---
layout: post
title:  "Dynamic schema in SQLAlchemy"
date:   2016-01-03 13:51:14 +0000
categories: SQL SQLA
---

I'm quite a big fan of SQLAlchemy - especially when it teaches me something new about Python. So I was particularly delighted when figuring out how to dynamically create tables and their columns in SQLAlchemy's ORM.

A quick note before we start: It should be pointed out that in many cases, you probably should know the schema of your database way before you write code that will talk to it. My particular reason for wanting to *dynamically* create tables was in order to automate reading large amounts of data into an SQL database from another source. I had a lot of columns, and really didn't want to have to handwrite the SQLA classes and attributes for all the tables and columns in my schema!

## Type is powerful

Turns out that the key to dynamic schema in SQLAlchemy is the unassuming Python `type` function! When called with one argument it simply returns the type of that object...

{% highlight python %}
introspective_string = "In this transient world of code, who am I?"
type(introspective_string) # Returns 'str'. Happy now?
{% endhighlight %}


But like Clark Kent, or pre-mushroom-mario, `type` has a more powerful form... when called with three arguments it produces a new class object!

{% highlight python %}
MyClass = type('MyClass', (myBaseClass,), \
		{'attribute': 5, 'method' myFunctionName}) # type evolved!
{% endhighlight %}

The first argument is the name of the class, the second argument is any base we want for our class (which needs to be the first entry in a tuple), and the third argument is a dictionary in which we can pass attributes and methods we want the class to have (methods can be passed by their function name). For more information on usage, see [Sahand Saba's fantastic post][type_info].

## Dynamically creating classes

This is all super useful in SQLAlchemy, because one of its most useful features is that it can provide an Object Relational Mapper (ORM), wherein tables in an SQL database are 'represented' by Python classes. The idea is that you need only interact with methods and attributes of these classes and their instances, whilst SQLAlchemy takes care of querying the database with SQL behind the scenes.

Normally, you'd therefore just create a class that SQLAlchemy can 'map' to a table in your database like so:

{% highlight python %}
Base = declarative_base()

class MyTableClass(Base):
	__tablename__ = 'myTableName'
	myFirstCol = Column(Integer, primary_key=True)
	mySecondCol = Column(Integer, primary_key=True)


Base.metadata.create_table(engine)
{% endhighlight %}

In this example, `Column()` is the SQLAlchemy method allowing us to represent columns in the database table we are mapping to. Also, we specify that our table class will inherit from the base class created by SQLAlchemy's `declarative_base` method. This is necessary to implement the '[declarative][dec_link]' usage of the ORM, which allows us to just define a class for our tables and have SQLAlchemy do the remaining footwork to link it to our database table[^1]. One final thing: we call `Base.metadata.create_table(engine)` to create a table with the name `'myTableName'` in our database if one doesn't already exist.

After this code executes, we'll have created a table `myTableName` in our database (if it doesn't already exist), with our Python class linked to it.

But suppose that you want to create tables in your database at run-time, without having a full idea of the schema at before compiling (perhaps you don't know how many tables you'll want, what they'll be called, or what columns they will need to include).

`type` to the rescue! First we create a dict that has all the attributes we want for our ORM class:

{% highlight python %}
attr_dict = {'__tablename__': 'myTableName',
	     'myFirstCol': Column(Integer, primary_key=True),
	     'mySecondCol': Column(Integer)}
{% endhighlight %}

Then we pass this to the type function when dynamically creating our class, also making sure to inherit from `Base` since we want to make our lives easier by using the declarative base:

{% highlight python %}
Base = declarative_base()

MyTableClass = type('MyTableClass', (Base,), attr_dict)
{% endhighlight %}
(I know, that half-used tuple notation for the superclass feels weird right? C'est la vie).

And there you have it! Dynamically created tables, using SQLAlchemy and its declarative base. One last tip though...

## Dynamically naming columns

So what if, a-priori, you don't know the *names* of the columns for your tables either?

Well, *creating* tables with dynamic column names is fairly straightforward - we just use `type` to create classes mapping to our tables (as above), but now we pass variable column names into the attributes dictionary instead of string literals.

What takes a tiny bit more fiddling is when you come to *insert* values into the table columns that you don't know the names of.

Normally, to insert a row of values into a table using SQLAlchemy, we create an instance of the class associated with our table, passing the values we want to insert to the attributes associated with each column, like this:

{% highlight python %}
new_row_vals = MyTableClass(myFirstCol=14, mySecondCol=33)

session.add(new_row_vals) # Add to session
session.commit() # Commit everything in session
{% endhighlight %}

The session calls force the SQL for these changes to be emitted to the database.

Now suppose the names of the two columns are stored in variables `firstColName` and `secondColName` somewhere. Naturally we *CAN'T* do this..

{% highlight python %}
firstColName = "Ill_decide_later"
secondColName = "Seriously_quit_bugging_me"

new_row_vals = MyTableClass(firstColName=14, secondColName=33)
{% endhighlight %}

At this point, SQLAlchemy is looking for columns literally called 'firstColName' and 'secondColName'. To get around this we can use a neat little trick:

{% highlight python %}
firstColName = "Ill_decide_later"
secondColName = "Seriously_quit_bugging_me"

new_row_vals = MyTableClass(**{firstColName: 14, secondColName: 33})
{% endhighlight %}

Here we've made use of Python's dictionary literals, which allow us to specify variable names for our keys, and then we've used the amazing `**` to *unpack* the dictionary's `key:value` pairs into a number of `key=value` assignments that are passed to the class constructor.


So there you have it; a couple of small tricks to make creating tables easier when you're feeling indecisive.


[type_info]: http://sahandsaba.com/python-classes-metaclasses.html#metaclasses
[dec_link]: http://docs.sqlalchemy.org/en/rel_0_8/orm/extensions/declarative.html

[^1]: The declarative use of the ORM (where we have our table classes inherit from the `Base` class, generated by the SQLAlchemy `declarative_base()`) means that SQLAlchemy takes care of configuring the ORM and linking our class to the database table specified by `__tablename__`. Under the hood SQLAlchemy actually does this by creating a `Table` object that refers to our database table and then using `mapper` to link our class to it. We could do that ourselves, calling `Table()` and `mapper()` with appropriate parameters, but using declarative base takes care of it.
