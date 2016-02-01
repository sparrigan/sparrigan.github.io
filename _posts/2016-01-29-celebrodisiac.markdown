---
layout: post
title:  "Celebrodisiac"
date:   2016-01-29 16:36:00 +0000
categories: DataScience Python JS Web Celebrodisiac
---

This post explains my motivation for building a web-app called [celebrodisiac][2]. If you haven’t seen it, then pop over, take a look and have a play around. I came up with celebrodisiac to try and help solve a larger project I’ve been working on in my spare time. Bear with me and I’ll take you on a journey steeped in history, glamour and babies... *lots* of babies.

The [US Social Security baby-names dataset][3] contains information on the number of different baby names registered each year in the US, and is a popular test-bed for practicing statistical methods. Whilst exploring it to hone my [pandas][4] skills, I noticed something pretty interesting. Whilst mooching around, I ended up creating the below plot, showing the number of baby names that **end** in the various letters of the alphabet each year.

![Time-series for last letters of baby names](http://www.nicharrigan.com/github_images/last_lett.png)


Weirdly, around 1910, there’s a sudden drop in names ending in the letters **a** and **e** (and a small increase in names ending in other letters). My first thought was the ever-exciting: *“that's odd...”*.  I actually have a deerstalker next to my desk precisely for moments like this, and so I donned it with no small pleasure. My initial speculation was  that this could perhaps be due to several big-hitter names decreasing in popularity. But looking at which different names contributed to the change this didn’t seem likely, and besides, it’d be quite a coincidence. Perhaps then, the decrease in a and e ending names is a symptom of something deeper. Spurred on by this, I looked at how last letters are distributed if we consider the space of all possible names that people ever choose, and this is what I found:

![Distribution of last letters in whole namespace](http://www.nicharrigan.com/github_images/name_last_lett_dist.png)

Names ending in **a** and **e** predominate. So we would *expect* **a** and **e** ending names to stick out if peoples choices of baby-names were reasonably well spread across the ‘namespace’. So perhaps, around 1910, something convinced the masses to specialize and preferentially choose from a smaller subset of names. If this subset was small enough and significant enough, there’d be a chance it could misrepresent the general **a** and **e** trend enough to cause the noticeable change.

*“Come Watson!...”* I shouted at no-one in particular, as the deerstalker slipped over my glasses and obscured my vision, *“…the game is afoot!”*. And in this case, the name of the game is entropy. I really needed a little bit more convincing that this hypothesized name ‘specialization’ was worth investigating, and so I turned to one of my favorite information-theoretic measures. Although it has a bad rep for un-tidying things in physics, entropy is an incredibly useful tool. In this case you can more or less think of it as a measure of how ‘fair’ peoples choices of baby-names are. If during one year everyone picked a name for their baby from the namespace completely at random, then the entropy of baby-names that year would be high. At the other extreme, if everyone just picked the same name (ultra-specialization!) then the entropy would be very low (zero in fact).

We can calculate the entropy of the baby-names people choose each year (in fact, the entropy is a biased estimator, and we need to use a correction[^foot1] called the Miller-Madow estimator). Here is what we get:

![Millow_Madow estimator over time](http://www.nicharrigan.com/github_images/entropy.png)

This adds weight to our hypothesis that, around 1910, people's choices of baby-name seems to have become a lot more focused. One more figure that adds weight to the idea is an overlay of plots of the Cumulative Distribution Function (CDF) for the most common baby-names each year:

![CDFs of most popular names (ordered) each year](http://www.nicharrigan.com/github_images/cumsum_yrs.png)

In this plot we’ve ordered baby-names by their popularity within a given year, and then generated the cumulative sum of occurrences. Plots for each year are overlaid and distinguished by the colormap, with 1910 being shown in red. It’s clear that there is a transition to the names chosen being more concentrated at a set of more popular names (corresponding to the CDF having more weight for lower values on the x-axis) – with the turning point being roughly around 1910.

Having more reliably established an effect, was there some historical event that could be a candidate for causing it[^foot2]? As I gently puffed on my bubble-pipe, I pondered [the possibilities][6]. First I thought of immigration – maybe an influx of people with a preferential set of traditional names could skew the stats? But that would presumably take a lot of immigrants, and besides - a short investigation revealed no big influxes into the US around that time.

What else could so influence peoples name choices? Then it hit me – the cinema! Wasn’t this around the time that movies began? Perhaps movie-star celebrities were starting to influence peoples choices of baby-names for the first time. The two biggest turning points in the history of film were the first real silent-movie in 1900, and the first ‘talkie’ (movie with sound) in 1927:

![Key moments in cinema](http://www.nicharrigan.com/github_images/key_cinema.png)

“Curses!”, I cried - neither of those events seem close enough to the 1910 event for them to form a decent hypothesis. But just as I reached to hang-up my trusty deerskin, a picture in the article I was reading caught my eye, and I read on with interest…

<img style="float: right; padding-left: 15px;" src="http://www.nicharrigan.com/github_images/pickford.jpg">
This is [Mary Pickford][7], and she is, quite frankly, awesome. Herself and [Florence Lawrence][8] were considered the great early actresses – Lawrence was hailed as “America's foremost moving picture star”. But despite their successes, no-one knew who Mary or Florence were. Despite the fact their faces were widely beloved to the masses, their names were kept an industry secret. In an uncannily prophetic act, early movie studios foresaw the cult of celebrity, and were scared that crediting actors and actresses in their productions would lead to them leveraging astronomical wages. So despite the fact that silent movie stars had been around since 1900, the studios had quashed celebrity by making sure that no-one was able to put names to the faces. That was, until Florence Lawrence negotiated a deal at a new studio which guaranteed she would be credited in her future films. Mary Pickford followed suit the next year, and thus began the inexorable descent into celebrity and personal promotion. Oh, and by the way, Florence Lawrence’s landmark move? It happened in **1910**.

Now this doesn’t mean anything on it’s own, it could all be just mere coincidence - all I really still have is just an intriguing hypothesis. Much more evidence is needed to support this idea – which is where [celebrodisiac][2] comes in. My ultimate aim is to hone this tool on more modern celebrities (a work in progress) – and then employ the finished article in early cinema, to measure the effects the most prominent early celebrities had on the baby namespace. It might then be possible to statistically determine whether the effects of these early movie stars could explain the observed change in entropy. There’s plenty of hurdles in this analysis. One of the trickiest parts will be accurately choosing ‘golden years’ for early movie-stars. It’s tricky to do that today, but there’s a huge lack of data for retroactively figuring out which of a stars movies were the most influential. Plus, any neural network trained on more modern releases is unlikely to be accurate if naively applied to the Hollywood of a hundred years ago (even *if* the same features were available). But nevertheless, the deerskin hasn’t come off yet, and this is the first step in an ongoing and compelling investigation. If nothing else, it's introduced me to the genius of [Buster Keaton][9].


[^foot1]: David Darmon has [an awesome blog-post][1] on the Miller-madow estimator - and just an awesome blog all round.
[^foot2]: I'm aware that the [thirteenth US census][5] was conducted in 1910, but I don't see any reason why this might lead to a data artifact in the *baby names* that had been recorded (and continued to be recorded) for many years before and after this. Also, the effect isn't localised to 1910, and the decrease in entropy persists for many years afterwards. That said, I'd love to hear from anyone who is more knowledgeable about the relevant inner workings of the 1910 census!


[1]: http://thirdorderscientist.org/homoclinic-orbit/2013/7/16/analyzing-and-bootstrapping-our-way-to-a-better-entropy-estimator-mdash-part-i-analytical
[2]: http://www.celebrodisiac.com
[3]: http://www.census.gov/data.html
[4]: http://pandas.pydata.org/
[5]: https://en.wikipedia.org/wiki/1910_United_States_Census
[6]: https://en.wikipedia.org/wiki/1910_in_the_United_States
[7]: https://en.wikipedia.org/wiki/Mary_Pickford
[8]: https://en.wikipedia.org/wiki/Florence_Lawrence
[9]: https://www.youtube.com/watch?v=UWEjxkkB8Xs
