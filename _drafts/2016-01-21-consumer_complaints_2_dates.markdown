---
layout: post
title:  "Consumer complaints 1: <br>Investigating categorical data"
date:   2016-01-16 11:29:14 +0000
categories: DataScience EDA Python Matplotlib
---



{% highlight python %}
import pandas as pd
fname = "<path>/cfpb_Consumer_Complaints.csv"
#Read CSV file. Specify datatypes of mixed columns.
cc_df = pd.read_csv(fname,index_col='Date received', /
			dtype={4:object, 5:object, 6:object, 9: object})
{% endhighlight %}

We've also indexed the data by the date of each complaint (currently strings). Pandas has some really powerful tools for dealing with dates, and we can take advantage of them later on if we convert this index of dates from being strings to `datetime` objects.	There's actually a built-in function, `pd.to_datetime()` for converting string representations of dates to `datetime` objects. We *could* consider mapping that to each date in our index, but interestingly I've found the following to be considerably (about 4x) faster:
1. Use python's `datetime.strptime()` function with a list comprehension to produce a list of *python* datetime objects
2. Convert this list of python datetime objects to a pandas `DatetimeIndex` with the `DatetimeIndex()` method.

{% highlight python %}
from datetime import datetime
# Create list of python datetime objects from date strings in our DF index
dt_objs = [datetime.strptime(x,'%m/%d/%Y') for x in cc_df.index]
# Create pandas DatetimeIndex from this list and assign it to our DF index
cc_df.index = pd.DatetimeIndex(dt_objs)
{% endhighlight %}

Although this seems a bit circuitous, it does the job, and leaves our dataframe with a very useful pandas *DatetimeIndex*. Note that in using `datetime.strptime()` we pass it a formatting parameter telling it how our date strings are arranged. Having been brought up on the UK 'backwards' convention of (DD/MM/YY) I think this is a great idea.
