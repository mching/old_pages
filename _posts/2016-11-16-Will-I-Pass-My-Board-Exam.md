---
layout: post
title: Will I Pass My Board Exam?
---

I am in the process of studying for my pediatrics board recertification and the pass rate is rumored to be about 75% of the questions correct. I got 163/199 (82%) of the questions correct on my practice exam. What is the probability that I will get at least 75% on my 200 question board exam based on this performance?

Here's the exact binomial way to figure it out.

The idea is that if my probability of getting any question right is 0.82, what is the probability that I will be correct on at least 0.75 * 200 or 150 questions on the actual test?


```r
p <- 163/199
n <- 0.75 * 200
# In this case I needed to use `n - 1` because the upper tail does not include
# the actual boundary case.
pbinom(n - 1, 200, p, lower.tail = FALSE) 
```

```
## [1] 0.994354
```

It turns out I have a pretty good chance of passing! 

We can also do this by simulation. Let's run this simulation a million times and see how many times I would pass.


```r
sim <- rbinom(1e6, 200, 163/199)
1 - sum(sim < 150) / 1e6
```

```
## [1] 0.994392
```

This converges pretty well to the actual calculated solution. My test is on 11/21/16 and I'll find out whether I pass a few days later... Here's hoping that this analysis isn't tempting fate!
