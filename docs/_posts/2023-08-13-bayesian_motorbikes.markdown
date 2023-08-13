---
layout: post
title: Zen and the Art of (Bayesian) Motorbike Maintenance.
date: 2023-08-13
categories: blog
---
{% include tex.html %}

> "There are three men on a train. One of them is an economist and one of them is a logician and one of them is a mathematician.

> And they have just crossed the border into Scotland (I don't know why they are going to Scotland) and they see a brown cow standing in a field from the window of the train (and the cow is standing parallel to the train).

> And the economist says, 'Look, the cows in Scotland are brown.' And the logician says, 'No. There are cows in Scotland of which at least one is brown.' And the mathematician says, 'No. There is at least one cow in Scotland, of which one side appears to be brown.'

> **The Curious Incident of the Dog in the Night-Time, Mark Haddon**

On a recent holiday to Vietnam, we hired motorbikes in a couple of different locations. In Vietnam, like much of South-East Asia, motorbikes are the de-facto method of transport, and hiring them was the cheapest and easiest way to get about, especially in the more rural areas. However, both motorbikes came with non-functioning speedometers. On the way back, this got me thinking; can I reasonably conclude that *any* speedometers work in Vietnam?

This is an easy question to answer under a Bayesian Framework. Suppose we consider $$ p $$ the probability that a given speedometer works. As $$ p $$ is a continuous random variable, $$ \mathbb{P}(p=0) $$ is 0, so instead are interested in $$ p $$ being negligible. In our case, we are interested in the two hypotheses

$$ H_0: p < 0.01 $$

and 

$$ H_1: p\geq 0.01 $$

We can assign our data a binomial distribution; we are counting the number of successful trials (finding a motorbike with a functioning speedometer) from $$ n $$ independent and identically distributed trials (hiring a motorbike), where each trial has success probability $$ p $$. We can take the conjugate prior for a Binomial distribution, and assign $$ p $$ a Beta distribution. In particular, we assign a non-informative uniform $$ Beta(1,1) $$ prior. Our model can thus be written as

$$
\begin{align*}
	p \sim Beta(1,1) \\
	x\vert p \sim Bin(n,p)
\end{align*}

$$

I will choose not to add to the thousands of existing derivations of the Beta-Binomial model, and instead simply tell you that for $$ n=2 $$ and $$ X = 0 $$ where $$ X $$ is the observed value of $$ x $$ 

$$ p\vert X  \sim Beta(1,3) $$

and

$$ f(p|X) = 3(1-p)^2 $$

Looking at this, our MAP estimate for $$ p $$ (the value of $$ p $$ that maximises $$ f(p\vert X) $$) is obvious: 0. But what about our hypotheses? Observe (using the `scipy.stats` library for calculations)

$$ \mathbb{P}(H_0) = \int_0^{0.01} f(p|X) dp = 0.03 $$

and 

$$ \mathbb{P}(H_1) = 1-H_0 = 0.97 $$

Hmm. Still a long way off accepting $$ H_0 $$. In fact, if we generalise the number of bikes observed with broken speedometers to $$ n $$ (giving us a posterior distribution of $$ Beta(1,n+1)) $$ we can write a quick python script to see how many broken speedometers we'd need to have observed to accept $$ H_0 $$ 

```
n = 2
p = 0
while p<=.5:
    n+=1
    p = beta.cdf(0.01, 1, n+1)
print(n)
```
which tells us we need to observe 68 consecutive broken speedometers before we can accept $$ H_0 $$. However, with data this limited, our model is going to be incredibly driven by our prior distribution. For example, if we had instead chosen the Jeffrey's prior as our prior distribution (a $$ Beta(0.5,0.5 $$)), we would only have needed to observe 23 broken bikes to accept $$ H_0 $$.
