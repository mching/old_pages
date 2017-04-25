---
layout: post
title: Birthday Problem at Work
---

My coworker keeps a list of our coworkers' birthdays so she can plan a monthly celebration. There are 36 of us in the office, and it turns out that none of us have the same birthday. My coworker wondered what was the chance of that happening.

As it turns out, the [birthday problem](https://en.wikipedia.org/wiki/Birthday_problem) is a classic probability question. Given a certain number of people, what is the probability that two of them have the same birthday? 

## Derivation of Probability
It is easier to calculate what is the probability of no one having the same birthday, or all the birthdays being different. We can use an example of 3 people. Starting with the first person, the probability that the next person does not have the same birthday is 364/365. The probability that the third person does not share either of the first two birthdays is 363/365.

$$ P(birthdays \space are \space all \space different) = {365 \over 365} \times {364 \over 365} \times {363 \over 365} \approx 0.992$$

We can see that this generalizes to:

$$ P(n \space birthdays \space are \space all \space different) = {365 \over 365} \times {364 \over 365} \times \ldots \times {365 - n + 1 \over 365} $$
$$ = {365! \over 365^{n}(365-n)!}$$
To get the probability that there is at least two or more birthdays are the same, you can subtract this number from one.

$$ P(at \space least \space two \space birthdays \space are \space the \space same) = 1- {365! \over 365^{n}(365-n)!} $$

Because these are humongous numbers, there are various approximations that have been developed for these. One simple example is based on the Taylor series expansion of the exponential function:

$$ p(n) \approx 1 - e^{-n(n-1)/(2\times365)}$$

Fortunately there is an R function `pbirthday` that calculates this for us. Examining the code, we can see that in the case when we want to see if there are two birthdays (`if (k == 2)`), it actually provides the exact (non approximation) calculation.


```r
pbirthday
```

```
## function (n, classes = 365, coincident = 2) 
## {
##     k <- coincident
##     c <- classes
##     if (k < 2) 
##         return(1)
##     if (k == 2) 
##         return(1 - prod((c:(c - n + 1))/rep(c, n)))
##     if (k > n) 
##         return(0)
##     if (n > c * (k - 1)) 
##         return(1)
##     LHS <- n * exp(-n/(c * k))/(1 - n/(c * (k + 1)))^(1/k)
##     lxx <- k * log(LHS) - (k - 1) * log(c) - lgamma(k + 1)
##     -expm1(-exp(lxx))
## }
## <bytecode: 0x7fa3503fd260>
## <environment: namespace:stats>
```

## Answering the Original Question
The `pbirthday` function makes it trivial to calculate the answer to the original question. Because `pbirthday` returns the probability that at least 2 have the same birthday, to get the probability that none have the same birthday, we just subtract the result of the function from 1.


```r
1 - pbirthday(36)
```

```
## [1] 0.1678179
```

The probability that no two coworkers would have the same birthday in an office with 36 people is 16.8%. That's uncommon but by no means improbable. I guess we're just a special bunch of people!
