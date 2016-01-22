---
layout: post
title:  "Consumer complaints 1:<br> Matrix plots and categorical data"
date:   2016-01-16 11:29:14 +0000
categories: DataScience EDA Python Matplotlib
---


A little while back, the US Consumer Financial Protection Bureau (CFPB) opened up [their excellent database of consumer complaints][1], containing date-stamped complaints made by US citizens about various financial products. Turns out that people love to moan - especially about money - and the dataset contains hundreds of thousands of entries spanning the past five years. Each entry has a wealth of information, ranging from the financial product under complaint to the location of complainer, and how the complaint was submitted. Over the next few posts I'll use the CFPB data to discuss a few different techniques and tools that I think are noteworthy.

One thing that immediately stands out about the consumer complaints data is that the majority of the raw data is *categorical* - having no intrinsic ordering. In this post I'll focus on how to get a quick and dirty overview of the dataset using matrix plots - which are well suited to initial explorations of categorical data. I'll be using python, along with the holy trinity (the [NumPy][numpy], [pandas][2] and [matplotlib][3] libraries). Relevant excerpts of code are given below inline, but everything is collected together in a single IPython notebook [here][4].


## Importing the data

At the time I acquired it, CFPB provided the data in a singe csv file, so the convenient pandas `read_csv` method can be used to import this directly into a dataframe. When importing csv files, pandas can very occasionally struggle (or at least take a while) identifying the datatypes of columns. We can use the `dtype` parameters of `read_csv` to help pandas out with typing of these columns.

{% highlight python %}
import pandas as pd
fname = "<path>/cfpb_Consumer_Complaints.csv"
#Read CSV file. Specify datatypes of mixed columns.
cc_df = pd.read_csv(fname,index_col='Date received', /
			dtype={4:object, 5:object, 6:object, 9: object})
{% endhighlight %}

And hark! A dataframe is born:

<div>
<table cellpadding="2" border="1" class="dataframe"  style="border-collapse: collapse; font-size: 10pt; text-align: center;">
  <thead>
    <tr>
      <th></th>
      <th>Product</th>
      <th>Sub-product</th>
      <th>Issue</th>
      <th>Sub-issue</th>
      <th>Consumer complaint narrative</th>
      <th>Company public response</th>
      <th>Company</th>
      <th>State</th>
      <th>Submitted via</th>
    </tr>
    <tr>
      <th>Date received</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>07/13/2015</th>
      <td>Student loan</td>
      <td>Non-federal student loan</td>
      <td>Can't repay my loan</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Transworld Systems Inc.</td>
      <td>PA</td>
      <td>Web</td>
    </tr>
    <tr>
      <th>07/13/2015</th>
      <td>Debt collection</td>
      <td>NaN</td>
      <td>Cont'd attempts collect debt not owed</td>
      <td>Debt is not mine</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Transworld Systems Inc.</td>
      <td>NY</td>
      <td>Web</td>
    </tr>
    <tr>
      <th>07/13/2015</th>
      <td>Debt collection</td>
      <td>Credit card</td>
      <td>Taking/threatening an illegal action</td>
      <td>Sued w/o proper notification of suit</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Transworld Systems Inc.</td>
      <td>KY</td>
      <td>Web</td>
    </tr>
  </tbody>
</table>
</div>

<br>

Of course it's actually a bit bigger than what I've shown here - about $$200,000$$ times bigger to be precise - but from this small snippet the categorical nature of most of the columns is still clear. A great tool for exploring relationships between the other $$418,522$$ rows and several columns we're missing is a [scatterplot matrix][5]. But unfortunately scatterplots are only really suited to quantitative data. What we could really use here is the tongue-twisting and much coveted *matrix-plot matrix*.

## Matrix-plot matrix

<img style="float: right; padding-left: 15px;" src="http://www.nicharrigan.com/github_images/22ident.png">
A useful alternative option to scatterplots in purely categorical cases is the *matrixplot*. Doing exactly what they say on the tin - matrix-plots represent the magnitudes of values from a matrix (or array if you're more of a computer scientist) by points of color in a visual grid. For example, the image to the right is a matrix-plot of a $$2\times{2}$$ identity matrix. Matrix-plots can be generated in matplotlib through `pylab.matshow`. I've used these a lot before for visualizing some [pretty weird (but pretty pretty) matrices][6] tied to the linear algebra of quantum mechanics -  but they work just as well in less esoteric settings. Here we can use them to visualize the number of coincident complaints that occur between categories from any two of our dataframe columns. In fact, using matplotlib's sub-plots tools, we can even put together a matrix-plot equivalent of a scatterplot matrix - a *'matrix-plot matrix'* if you will.

Here's what we get if we produce a 'matrix-plot matrix' for nine of the most interesting columns of the CFPB consumer complaints dataframe (code available [here][7]):

<div style="text-align:center"><img src="http://www.nicharrigan.com/github_images/matrixplotmatrix.png"></div>

<br>

As expected, the plot matrix is symmetric with diagonalized blocks along its main diagonal. Note that:

* There are no category labels within each column (if there were it'd be more word-cloud than matrix-plot!). The purpose of the plot is to help us see patterns, and guide us on which columns to investigate further with individual labelled plots.

* There is no colorbar (key).  The darker the color, the higher the number of complaints, but the exact mapping is (linearly) normalized for each plot according to it's own max/min values - so there is no 'one' colorbar key for all the plots.

Its also important to point out that some of the dataframe columns have relatively large number of categories (for example, there are 96 different categories of **Issue**). To make the above plot feasible, only the top ten categories for each column were used. Selecting the top ten categories for a column only takes a couple of lines with pandas[^foot1]:

{% highlight python %}
# Don't consider nan/null entries
not_null = dataframe[column][dataframe[column].notnull()]
# Top ten for column
top_ten = not_null.value_counts()[:10].index
{% endhighlight %}

## Logarithmic Colormaps

Several of the subplots look as though they might contain some interesting features. Let's take a more detailed look at **Company** versus **Product**:

{% highlight python %}
plt.matshow(data, cmap=cm.gray_r)
{% endhighlight %}

![Company versus product - Linear Colormap](http://www.nicharrigan.com/github_images/comp_prod_lin.png)

This shows the number of complaints about a given company regarding a given financial product. There's a couple of problems with this matrix-plot. First - and most obviously - a lot of the detail seems 'washed out'. This problem stems from the colormap we're using. To be more precise, a [colormap][8] is a rule for mapping values (in this case number of complaints) to a color that we will display. Here we are using a (reversed grayscale) *linear* map. But if we plot a quick histogram of the orders of magnitude that appear in the range of complaints in our company versus product matrix-plot, we can see the problem:

{% highlight python %}
log_data = np.log10(data)
plt.hist(log_data, bins=int(sqrt(len(log_data))), normed=True, color="Burlywood", linewidth=2);
{% endhighlight %}

![Histogram of magnitudes](http://www.nicharrigan.com/github_images/data_hist.png)

Clearly there's a large variance in the orders of magnitude contained within our data, with a skew towards lower values. So a linear scale isn't the best choice for our colormap - being bound to 'wash-out' detail from one end of the data. Instead, we're better off using a logarithmically scaled colormap, where changes in the *order of magnitude* of our data are mapped to changes along our colormap. With `pylab.matshow` this is easy enough to achieve by setting the parameter `norm=Lognorm()`. This gives us the following, much more readable matrix-plot (note the colorbar key below the figure is now logarithmic):

{% highlight python %}
from matplotlib.colors import LogNorm
plt.matshow(data_array, cmap=cm.gray_r, norm=LogNorm())
{% endhighlight %}

![Company versus product- Log Colormap](http://www.nicharrigan.com/github_images/comp_prod_log.png)

Although we can now visualize the data more easily, there's still a subtler problem. Ideally we'd like to be able to look at this matrix-plot and make fair comparisons between complaints made to the different companies (regarding various products). But the problem is that as well as our colormap, *the data itself* needs to be normalized. At the very least we should take into account the number of customers each company has (to give a 'per-capita' number of complaints), and also consider the fraction of their business that is involved with any given product.

Estimating the customer-base of a company is an interesting (but hefty) data project in itself, so lets instead take a look at another promising sub-plot from our matrix-plot matrix - **State** versus **Product**:

{% highlight python %}
plt.matshow(state_data_array, cmap=cm.gray_r, norm=LogNorm())
{% endhighlight %}

![State versus product - Normalized log colormap](http://www.nicharrigan.com/github_images/state_prod_log_gray.png)

This plot uses a log-scaled colormap, *and* the underlying data has been normalized. In this case, since we'd like to fairly compare complaints from different states, we've normalized data from each state by the state population - which was scraped from wikipedia[^foot2] using the python library [beautifulSoup][9] (scraper code [here][10]). This gives us 'per-capita' complaints for each product.

Clearly mortgages are the subject of most complaints in almost every state, whilst the residents of D.C are pretty vocal across the board. Also of note is that there are zero complaints regarding payday loans in North Dakota, whilst there are a significant number of complaints in D.C and Georgia. In all three states payday loans are [effectively illegal][11]. It could be interesting to try and tease out whether these differences can be attributed to state-by-state approaches to the status of payday loans.   


## Choosing a colormap

So far we've stuck with using a grayscale colormap - albeit one with a logarithmic scale (in fact we've used `gray_r` - *reversed* grayscale). There's a lot to be said for grayscale. Most importantly, it's a *sequential* scale, meaning that the perceived luminosity varies monotonically throughout the scale. This helps prevent [luminosity artifacts][12] in our matrix plots - where regions might stand out purely because of the nature of the colormap rather than because of legitimate changes in the data.

But nevertheless, lets try and get a bit more color in our lives. The relationship between the human perception of 'lightness' and coordinates in colorspace is a fascinating, complex, and messy subject[^foot3], but the [matplotlib documentation][8] offers a great breakdown of different colormap schemes and how their luminosity varies.

There are legitimate reasons to want something with a bit more pizazz. Although `gray_r` does a great job, it isn't a *uniform* colormap - meaning that equal changes in the data do not always map to equal changes in perceived lightness at different points along the `gray_r` colormap. But if we wander out of the shadows and into the rainbow then we can find colormaps that are both sequential *and* uniform. But take care! They're not the norm - in fact the default colormap used by matlab, and all but the most recent versions of matplotlib - called `jet` - isn't even *sequential*, let alone uniform. You don't have to spend long on data-viz blogs before you encounter some pretty full on `jet`-hate.

Fortunately though, matplotlib (v1.5+) does include some colormaps that are both sequential and uniform. Namely - `viridis`, `inferno`, `plasma` and `magma`.

So let's use a logarithmic scale and the `magma` colormap to inspect the matrix-plot between **State** and **Submitted via** (how the complaint was registered):

{% highlight python %}
plt.matshow(data, cmap="magma", norm=LogNorm())
{% endhighlight %}

![State versus product - Normalized log colormap](http://www.nicharrigan.com/github_images/state_submit_magma_log.png)

Notice that along the bottom row (the **Email** category) there are some blank spaces. Looking at the scale for the magma colormap, you might think that these must correspond to relatively high values - but in fact, looking up the relevant dataframe entries, we find that they're zero. This is a potential problem with using a logarithmic scale: $$\log_b(0)$$ is undefined for any base, $$b$$. In fact, this problem was also present in the plots we created using the  `gray_r` colormap - we just didn't notice it, since `gray_r` *would* have also associated zero with white. To get around this, we can replace the `LogNorm()` scale with `SymLogNorm()` (Symmetric LogNorm - [doc][13]). `SymLogNorm` is essentially a piecewise function that is linear within some region about the origin and logarithmic otherwise. We pass `SymLogNorm()` an argument for our 'threshold' - how close a point needs to be to the origin to be mapped linearly, rather than logarithmically. Choosing the value of this threshold requires us to think a little about how our data is distributed. A good threshold choice for the **State** vs **Submission Type** matrix-plot is $$10^{-6}$$, which gives us the following:

{% highlight python %}
plt.matshow(state_data_array, cmap="magma", norm=SymLogNorm(10**(-6)))
{% endhighlight %}

![State versus submission type - SymLogNorm](http://www.nicharrigan.com/github_images/state_submit_magma_symlog.png)

One thing you'll notice if you try to reproduce the above plot is that the colormap tick-marks go a bit haywire. Unfortunately `SymLogNorm()` seems to break the default tick-mark placement, and you'll need to correct it yourself with something like this:

{% highlight python %}
{% raw %}
tick_locations=([0]+[(10**x) for x in xrange(minlog+1,maxlog+1)]);
cbar = plt.colorbar(im, ticks=tick_locations, orientation='horizontal', \
			pad=0.1, shrink=0.3)
cbar.ax.set_xticklabels(['$0$']+ \
			['$10^{%r}$' %x for x in xrange(minlog+1,maxlog+1)]);
{% endraw %}
{% endhighlight %}

Bingo! Data-inspection-perfection (well, imporvement at least). And straight away some interesting points stand out in our colorful new matrix-plot. For example, New Hampshire seems to be quite keen on complaining through referals. Whilst Michigan and Delaware are pretty retro with all that faxing.

## Next time

I've found matrix-plots to be great tools for getting a quick overview of the data and either showing trends, or just suggesting where it might be fruitful to look to find them. But of course ultimately we're really interested in using statistics to verify, refute or utilise more quantitative relationships.

In the next blog post, I'll continue using the CFPB dataset to look at how registered complaints have varied over time, leveraging some of the power of the pandas `datetime` object and more matplotlib goodness.

[1]: http://www.consumerfinance.gov/complaintdatabase/
[2]: http://pandas.pydata.org/
[3]: http://matplotlib.org/
[4]: TODO: LINK TO IPYTHON NOTEBOOK
[5]: http://www.r-bloggers.com/scatterplot-matrices/
[6]: http://arxiv.org/abs/0709.1149
[7]: https://gist.github.com/sparrigan/fd8d94c4921fb6017182
[8]: http://matplotlib.org/users/colormaps.html
[9]: http://www.crummy.com/software/BeautifulSoup/
[10]: https://gist.github.com/sparrigan/de6301c7ac4ddf83e346#file-wiki_state_scraper-py
[11]: http://www.paydayloaninfo.org/state-information
[12]: https://mycarta.wordpress.com/2012/10/06/the-rainbow-is-deadlong-live-the-rainbow-part-3/
[13]: http://matplotlib.org/api/colors_api.html#matplotlib.colors.SymLogNorm
[14]: http://www.amazon.co.uk/First-Steps-Seeing-R-W-Rodieck/dp/0878937579
[numpy]: http://www.numpy.org/
[^foot1]: Our selecting of the top ten also explains why many of the subplots appear to have 'smooth gradients' within our grayscale colormap. This is a little unexpected, since the categories for each column are unordered - so we wouldn't expect any correlation between number of complaints and the proximity of categories on the plot. However, the pandas `value_counts()` method returns categories sorted in order of number of counts (i.e. complaints). One could argue that it would be a good idea to randomize this ordering to prevent apparent trends where none exist, but with a little caution, this ordering can be useful to aid comparing the spread of values from different categories.
[^foot2]: Wikipedia's `robots.txt` states *"Friendly, low-speed bots are welcome viewing article pages, but not dynamically-generated pages please."*. My bot was not only friendly, it was polite and well mannered (offered to take Wikipedia out for a steak dinner).
[^foot3]: Human vision itself is just insanely and amazingly interesting - even modulo neural networks. One of the best books I've ever read is [Rodieck's "The First Steps in Seeing"][14], which I fervently recommend to anyone remotely interested in how things work, computing, or just being amazed.
