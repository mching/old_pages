---
layout: post
title: Rolling 3 Doubles in Monopoly
---

Tonight my two kids and I were playing Monopoly. It's a terrible, brutal game that usually ends in bad feelings, but for some reason they always want to play it. One of the rules of the game is that if you roll a double (e.g., two 2s), you get another turn. However, if you roll doubles three times in a row, you are sent to jail.

I wondered what was the probability of rolling 3 doubles in a row. It seemed like it would be pretty unlikely, but each of us had this happen to us over the course of the game. Now what's the chances of that happening?

It seemed like an easy enough problem, but I was drinking a beer, eating pita chips, and watching Game 7 of the NBA Western Conference finals at the time. So I wasn't really focused on the game, but still that seemed pretty unlikely.

### Simulation of the Probability of a Triple-double Roll
Later, I went to the computer to try to sort it out. I figured it would be easy enough as a simulation, and so I tried it. Here's the code.

I wrote a helper function to decide if a vector of 6 numbers (representing rolls of a single die) were the same for the first two, the same for the second two, and the same for the third two. I used a cascading if clause to cut down on computation in case the first two were not the same, or if the second two were not the same. This way I wouldn't test all three pairs unnecessarily.


```r
double3 <- function(v) {
  # takes a vector v of length 6 and checks if the first and second elements are
  # equal, third and fourth are equal, and fifth and sixth are equal
  if(length(v) != 6) {
    return("input should be of length 6")
  }
  if(v[1] == v[2]) {
    if(v[3] == v[4]) {
      if(v[5] == v[6]) {
        return(T)
      }
    }
  }
  return(F)
}
```

Now for the simulation. I decided to run 100,000 trials of 6 die rolls. I created a matrix of 100,000 rows with 6 columns.

```r
NUMBER_OF_ROWS <- 100000
NUMBER_OF_COLS <- 6
x <- matrix(sample(1:6, NUMBER_OF_ROWS*NUMBER_OF_COLS, replace = T), 
            nrow = NUMBER_OF_ROWS)
```

I then used the magic of `apply` to figure out if the first two rolls were the same, the second two, and the third. I summed the result and divided by the number of trials to get an estimate for the probability. I thought it would be rare but not that rare!

```r
output <- apply(x, 1, double3)
sum(output) / NUMBER_OF_ROWS
```

```
## [1] 0.00422
```

### Calculating Exact Probability using Binomial Functions
In retrospect, it is a simple probability problem that has an easy solution. Once you've rolled the first die, you have a 1/6 chance of rolling the same number on the second die. So the probability of getting the same number on a roll of two dice is 1/6. And if you want to know what is the chance of rolling two die and getting doubles three times in a row, that's just 1/6 * 1/6 * 1/6 or 1/216. This is approximately 0.0046296.

### Probability of Observing at Least 3 Triple-doubles
Now the next question was what's the likelihood to have done this 3 times (or more) in a single Monopoly game. I'm not sure how many turns we took, but I had 10 properties at the point that someone started crying because she ran out of money. My son had about the same number. And she had only 5. So we had to have had at least 25 turns. Conservatively speaking, let's multiply that by 6 for 150 total turns. What is the probability that we would observe 3 or more jail-inducing triple doubles?

Here, the binomial functions are super helpful at producing the exact number. In this case, I used `pbinom` with the upper tail. It's important to remember that the function's default behavior is to return the probability of "x successes or fewer" so if we want 3 or more, we need to give the function "2 successes" rather than 3, and ask for the upper tail.


```r
pbinom(2, 150, p = 1/216, lower.tail = F)
```

```
## [1] 0.03310417
```

3%! Not that rare after all! I mean, it's pretty unlikely, but we observed it in the one time we played this game in 6 months.

Here's the probability that it will happen at least once in a game of 150 turns.

```r
pbinom(0, 150, 1/216, lower.tail = F)
```

```
## [1] 0.5014528
```

### Conclusion:
It's hard to roll three doubles in a row on any one turn (1/216), but over the course of a game of 150 turns, it's likely that it will happen at least once (about 50%).
