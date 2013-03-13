---
layout: post
title: How to sort ratings
---

Evan Miller has a blog posted titled ["How Not to Sort By Average Rating"](http://www.evanmiller.org/how-not-to-sort-by-average-rating.html) where he explains two ways people commonly _incorrectly_ sort things by average rating, and then lays out a correct (but rather complicated) method for sorting by average rating that uses some rather advanced statistics.

I've always been a little weak at statistics and probability theory, so I wanted to try and get myself to understand the third, correct way of sorting by average rating laid out in Miller's post (the derivation of which is not really explained).

First, the two incorrect ways, and why they're wrong. For now, we'll be considering a binary rating system where we only have positive and negative voting (ie, I like it, or I don't like it), as opposed to a more elaborate 5-star rating scale or something like that.

## A bad way of doing things

Sort by (# of positive ratings) - (# of negative ratings).

This is wrong because it tends to arbitrarily be more favorable to products that have more reviews. For example, if one item has 100 positive ratings and 0 negative ratings, while another has 1,000 positive ratings and 850 negative ratings, the second item will be put before the first one, because its net score is 150, which is higher than the first product's net score of 100, despite the fact that the first product has received 0% negative ratings and the second has received 46% negative ratings.

A natural way one might think to solve this problem is to sort by percentage of positive ratings, and while it does fix this particular issue, it leads to another problem...

## A better way of doing things (that's still wrong)
Sort by (# of positive ratings)/(total # of ratings).

This sorting method will solve the problem caused by method #1, but it creates another problem: Suppose one item has 4 positive ratings and no negative ratings, while another item has 1,000 positive ratings and 24 negative ratings.

This scoring method will put the first item, which has 100% positive ratings, over the second item, which has only 97.6% positive ratings. Now, why is this bad? It boils down to an issue of <em>confidence</em>. While the first item does have 100% positive rating, this 100% is based off of only 4 people. It could be that the item got "lucky" and was only rated by the 4 people that happen to like it, whereas other people rate the product poorly. It's <em>much</em> less likely that we got 1,000 people to rate the second item favorably only by chance, and that really only a very small percentage of population would have actually rated that product favorably.

So the idea is that we'd gladly give up 2.4% of our 100% positive ratings to get the confidence of 1,024 people having rated the item. This issue should shed some light on why we'll have to turn to statistics and probability in order to correctly sort by average rating.

## Modeling the problem

The first thing we'll want to do for developing our correct method is to define the problem in terms of a **random variable**. A random variable is basically just a function whose value depends on the results of a particular real-world experiment. Here, the "experiment" is "does someone like this item or not". Given that, we define our random variable like so:

$$
X =
\cases{
  1 & \text{ if the item is liked}\cr
  0 & \text{ if the item is not liked}
}
$$

So if we perform the experiment and someone likes the item, we say $X = 1$, and if we perform the experiment and someone doesn't like the item, we say $X = 0$.

We can also talk about the **expected value** of our random variable, which is denoted $E(X)$. Basically, the expected value of a random variable is the average value that the variable would take on over multiple instances of the experiment.

For example, if 50% of people would like the item, and 50% of people would not, then 50% of the time $X$ would be $0$, and the other 50% of the time $X$ would be $1$. This would mean that $E(X) = 0.5\cdot 1 + 0.5\cdot 0 = 0.5 $. On the other hand, if 70% of people would like the item and 30% would not, then $E(X) = 0.7 \cdot 1 + 0.3 \cdot 0 = 0.7$. Generally speaking, if $p$ is the proportion of people that would like the product, then

$$E(X) = p \cdot 1 + (1 - p)\cdot 0 = p$$

So pretty conveniently, the expected value of $X$ is exactly the proportion of people that would like the product. This is a result of the fact that we picked 1 and 0 as the values that $X$ could take on. We could have, for example, defined $X$ to be 24 if someone likes the item and -9 otherwise, but picking the values 1 and 0 makes the math much easier, as you now hopefully see.

So $p = E(X)$ is essentially the value that we would like to sort on when sorting items by ratings. Unfortunately, knowing what $E(X)$ is is pretty hard. You'd basically literally have to ask everyone on earth (or maybe more accurately everyone who purchased the item or something like that) what they thought of that item, and record their responses.

We don't have access to this information. We only have access to a small _sample_ of ratings, based on who actually bothered to tell us what they think of the item. The size of our sample might range anywhere from 1 rating to 5,000,000 ratings. So our job (and in some sense the job of statistics in general) is to do our best at estimating the true value of $E(X)$, based only on the information we have from our sample.

## The central limit theorem

So what information do we have available to us? Presumably, we know the number of good ratings an item has received and the number of bad ratings an item has received. Using this, we can compute the proportion of people that favored the product _of those that bothered to rate it_. We'll call this proportion $\hat{p}$. What we're interested in is what $\hat{p}$ tells us about normal $p$.

To do this, we rely first and foremost on the [central limit theorem](http://en.wikipedia.org/wiki/Central_limit_theorem), which is a very important theorem in statistics. For our particular problem, the central limit theorem theorem says that $\hat{p}$ is **normally distributed** with a mean equal to $p$ and **variance** inversely proportional to $n$, the number of people we have ratings from, all for sufficiently large $n$.

Okay, what? That's a bit to digest. So first of all, instead of thinking of $\hat{p}$ as a specific value, think of it in terms of a random variable. In fact, let's call this random variable $\hat{X_n}$ (and again, $n$ is the number of people we have ratings from). The "experiment" that we perform to get values for $\hat{X_n}$ is "take a random sample of $n$ people from the world, ask them what they think of our item, and see what proportion of them liked it". Clearly, $\hat{p}$ is an instance of this random variable $\hat{X_n}$.

The central limit theorem says firstly that $E(\hat{X_n}) = E(X)$. That is, on average, $\hat{p}$, the proportion of people that like the product from our sample of $n$, will be "close to" $p$, the true proportion of people that like the product. To see how close $\hat{p}$ will be to $p$, we need the rest of the central limit theorem.

So the second thing that the central limit theorem tells us is $\hat{X_n}$ is roughly [normally distributed](http://en.wikipedia.org/wiki/Normal_distribution) (with the distribution being closer to a true normal distribution the larger $n$ is). It also tells us that ${\rm Var}(\hat{X_n})$, the **variance** of $\hat{X_n}$, is equal to ${\rm Var}(X)/n$.

## Wait, what's variance?

So we haven't talked about variance yet. To try and be as simple as possible, forget about all of these random variables for a bit, and go back to something really simple, like a list of numbers:

$$ 6, 8, 5, 6, 6, 7 $$

This list of numbers has an average, which I'm sure you know how to compute:

$$ \frac{6 + 8 + 5 + 6 + 6 + 7}{6} = \frac{38}{6} = 6{\frac{1}{3}} $$

Let's call the average $\mu = 6{\frac{1}{3}}$. We can, for each number in the list, compute the distance from the number to the average:

$$ (6-\mu) = -\frac{1}{3}, (8-\mu) = 1{\frac{2}{3}}, (5-\mu) = -1{\frac{1}{3}}, (6-\mu) = -\frac{1}{3}, (6-\mu) = -\frac{1}{3}, (7-\mu) = \frac{2}{3} $$

And, to make everything nice and positive, we can square all those distances:

$$ \left(-\frac{1}{3}\right)^2 = \frac{1}{9}, \left(1{\frac23}\right)^2 = \frac{25}{9}, \left(-1{\frac13}\right)^2 = \frac{16}{9}, \left(-\frac{1}{3}\right)^2 = \frac{1}{9}, \left(-\frac{1}{3}\right)^2 = \frac{1}{9}, \left(\frac23\right)^2 = \frac49$$

And, finally, we can take the average of all those squared distances:

$$ \frac{\frac19 + \frac{25}{9} + \frac{16}{9} + \frac19 + \frac19 + \frac49}{6} = \frac{\frac{48}{9}}{6} = \frac{48}{54} = \frac{8}{9}$$

This is what variance is: the average of the squared distances to the average. Variance is often denoted as $\sigma^2$, which makes sense once you realize that **standard deviation**, the square root of variance, is denoted as $\sigma$. So for the above list of numbers, $\sigma^2 = \frac{8}{9}$ and $\sigma = \sqrt{\frac{8}{9}} = \frac{2\sqrt{2}}{3}$.

This is variance for a finite list of numbers. To compute the variance for a random variable (denoted ${\rm Var}(X)$, as we've already seen), we do something similar. Obviously we can't do a straight average of our numbers since we don't actually have a particular set of numbers to work with. So instead of a normal average, we use $E(X)$, and we then say that ${\rm Var}(X) = E((X - E(X))^2)$ (ie, the variance is the expected value of the squared distance between $X$ and its own expected value). 

Recall that $E(X)$ is $p$, and so when $X$ is 1 (which happens with probability $p$), its squared distance to its expected value is $(1 - E(X))^2 = (1-p)^2$, and when $X$ is 0 (which happens with probability $(1-p)$), its squared distance to its expected value is $(0 - E(X))^2 = p^2$, and so

$${\rm Var}(X) = p\cdot((1-p)^2) + (1-p)\cdot(p^2) = p(1 -2p + p^2) + p^2 - p^3 $$ $$= p - 2p^2 + p^3 + p^2 - p^3 = p - p^2 = p(1-p)$$


## Confidence Intervals

So going back to the central limit theorem, we have that the random variable $\hat{X_n}$ is normally distributed with mean $E(X) = p$ and variance ${\rm Var}(X)/n$. Because of things we know about normal distributions, this can tell us a lot about how $\hat{p}$ (which is essentially an instance of the random variable $\hat{X_n}$) relates to $p$.

In particular, something well known about normal distributions is how likely something is to be within $z$ standard deviations of the mean. That is, for our normal distribution, our variance is ${\rm Var}(X)/n = p(1-p)/n$ (as we showed above). This means that the standard deviation of $\hat{X_n}$ is $\sigma = \sqrt{ \mbox{Var}(X)/n} = \sqrt{p(1-p)/n}$. Remember that our mean was $p$, and so we know what the odds are for $p$ and our specific value $\hat{p}$ to be within, say, one standard deviation of each other (the odds for one standard deviation happen to be around 68%). Ie, we know that: 

$$ \Pr(|p - \hat{p}| < \hat{\sigma}) \approx 0.68 $$

where $\hat{\sigma} = \sqrt{ {\rm Var}(\hat{X_n})}$ is the standard deviation of $\hat{X_n}$.

Well, sort of. The central limit theorem tells us that $\hat{X_n}$ is _roughly_ normal, so we don't know the probability is _exactly_ 68%, but we're _confident_ that it's around 68%.

We also know (are confident) that the probability of $\hat{p}$ being with _two_ standard deviations of $p$ is approximately 95%...

$$ \Pr(|p - \hat{p}| < 2\cdot\hat{\sigma}) \approx 0.95 $$

...and that the probability of being with _three_ standard deviations is approximately 99.7%

$$ \Pr(|p - \hat{p}| < 3\cdot\hat{\sigma}) \approx 0.997 $$

So in general, for any particular probability $\alpha$, it's possible to find a corresponding value $z$ (often written $z_{\alpha/2}$) such that the odds of $p$ and $\hat{p}$ being with $z$ standard deviations of each other is $\alpha$:

$$ \Pr(|p - \hat{p}| < z\cdot\hat{\sigma}) \approx \alpha $$

This leads to an _interval_ of values that we are $\alpha$ sure that $p$ is between. Ie, we know with probability $\alpha$ that

$$ \hat{p} - z\cdot\hat{\sigma} < p < \hat{p} + z\cdot\hat{\sigma} $$

The $z$ value corresponding to an $\alpha$ of 95%, for example, is 1.96. So this means that we can be 95% sure that

$$ \hat{p} - 1.96\hat{\sigma} < p < \hat{p} + 1.96\hat{\sigma} $$

So what's really going on here? We'd _like_ to know what $p$ is. But we don't. Instead, we know what $\hat{p}$ is. And we know that $\hat{p}$ should be roughly $p$, but we're not all that sure. Especially when we only have a few ratings to go on.

What we _can_ say, though, is that we are 95% sure that $p$ is no less than $\hat{p} - 1.96\hat{\sigma}$. And so this, this $\hat{p} - 1.96\hat{\sigma}$ value, _this_ is what we want to sort by. The lower bound of a 95% confidence interval.

Except there's a bit of a problem with that. Do you recall the formula for $\hat{\sigma}$, the standard deviation of $\hat{X_n}$? It was $\hat{\sigma} = \sqrt{p(1-p)/n}$. And the whole reason we're in this mess to begin with is because we don't know $p$, so we can't very well know $\hat{\sigma}$ either.

One way to cope with this is to do the simplest thing we can, and just replace $p$ with $\hat{p}$ for our calculation of $\hat{\sigma}$. That is, we let $\hat{\sigma} = \sqrt{\hat{p}(1-\hat{p})/n}$, and so the lower bound of our confidence interval becomes

$$\hat{p} - z\sqrt{\frac{\hat{p}(1-\hat{p})}{n}}$$

This is sometimes called the lower bound of the "normal approximation interval".

## Wilson score confidence interval

Our approximation of $\hat{\sigma} \approx \sqrt{\hat{p}(1-\hat{p})/n}$ is alright, but still not really good enough to use for sorting ratings. In particular, notice what happens at "extreme" values of $\hat{p}$ (ie, if $\hat{p}$ is 1 or 0). In these cases, $\hat{p}(1-\hat{p})$ will be 0, causing the lower bound of our interval to be just plain old $\hat{p}$.

And in fact, for items with only a few ratings, it's quite common to have extreme values for $\hat{p}$. Consider an item with just one positive rating and no negative ratings, for example. It will have a $\hat{p}$ of 1, meaning it would go to the top of our listing. This means that if we implemented this kind of sorting, we'd get a bunch of items with just one positive rating floating all the way to the top, above items with many more positive ratings but also a few negative ratings, which is exactly the kind of behavior we wanted to avoid.

Instead, we're going to have to do something else to deal with the fact that we don't know $\hat{\sigma}$. To this end, we use the "Wilson score confidence interval", which uses some clever algebra to get around having to compute $\hat{\sigma}$ directly. Recall the formula we had for our confidence intervals:

$$ |\hat{p} - p| < z\cdot \hat{\sigma} $$

We can square both sides of this inequality to get

$$(\hat{p} - p)^2 < z^2\cdot\hat{\sigma}^2$$
$$(\hat{p} - p)^2 < z^2\left(\frac{p(1-p)}{n}\right)$$

If we let $t = \frac{z^2}{n}$ (just to make things easier to write), we get

$$\hat{p}^2 - 2p\hat{p} + p^2 < t(p-p^2)$$
$$\hat{p}^2 - 2p\hat{p} + p^2 < tp-tp^2$$
$$\hat{p}^2 - (2\hat{p}+t)p + (1+t)p^2 < 0$$

This inequality is quadratic in $p$, which means we can use the quadratic formula to find the end-points (ie, the places where we are equal to 0, not less than it). The quadratic formula for solving $c + bp + ap^2 = 0$ is

$$ p = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

and so if we have $a = 1 + t$, $b = -(2\hat{p} + t)$, and $c = \hat{p}^2$, we get endpoints of

$$ \frac{(2\hat{p} + t) \pm \sqrt{(2\hat{p} + t)^2 - 4(1+t)\hat{p}^2}}{2(1+t)} = \frac{2\hat{p} + t \pm \sqrt{4\hat{p}^2 + 4\hat{p}t + t^2 - 4\hat{p}^2 - 4t\hat{p}^2}}{2(1+t)}$$
$$ = \frac{2(\hat{p} + \frac{t}{2}) \pm \sqrt{4(\hat{p}^2 + \hat{p}t + \frac{t^2}{4} - \hat{p}^2 - t\hat{p}^2)}}{2(1+t)} = \frac{\hat{p} + \frac{t}{2} \pm \sqrt{\hat{p}t + \frac{t^2}{4} - t\hat{p}^2}}{1+t}$$
$$ = \frac{\hat{p} + \frac{t}{2} \pm \sqrt{t(\hat{p}-\hat{p}^2) + \frac{t^2}{4}}}{1+t} = \frac{\hat{p} + \frac{t}{2} \pm \sqrt{t\hat{p}(1-\hat{p}) + \frac{t^2}{4}}}{1+t}$$

Now we just replace $t$ with $\frac{z^2}{n}$, and we're left with that scary looking equation at the bottom of Miller's post:

$$\frac{\hat{p} + \frac{z^2}{2n} \pm \sqrt{\frac{z^2\hat{p}(1-\hat{p})}{n} + \frac{z^4}{4n^2}}}{1+ z^2/n} = \frac{\hat{p} + \frac{z^2}{2n} \pm z\sqrt{\frac{\hat{p}(1-\hat{p})}{n} + \frac{z^2}{4n^2}}}{1+ z^2/n}$$

What that expression solves for is the values of $p$ for which $|\hat{p}-p|$ is exactly $z\cdot\hat{\sigma}$. If we want $|\hat{p} - p| < z\cdot\hat{\sigma}$, then we want $p$ to be in between those two values. Ie, we want


$$ \frac{\hat{p} + \frac{z^2}{2n} - z\sqrt{\frac{\hat{p}(1-\hat{p})}{n} + \frac{z^2}{4n^2}}}{1+ z^2/n} < p < \frac{\hat{p} + \frac{z^2}{2n} + z\sqrt{\frac{\hat{p}(1-\hat{p})}{n} + \frac{z^2}{4n^2}}}{1+ z^2/n} $$

So this tells us that (if $z = 1.96$) we are 95% sure that $p$ is at least

$$ \frac{\hat{p} + \frac{z^2}{2n} - z\sqrt{\frac{\hat{p}(1-\hat{p})}{n} + \frac{z^2}{4n^2}}}{1+ z^2/n} $$

This then is the lower bound of the Wilson score confidence interval, and what we want to sort by.