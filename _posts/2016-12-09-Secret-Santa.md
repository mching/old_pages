---
title: "Simple Secret Santa Picker"
layout: post
ext-js: //cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX
---

[A post about automating Secret Santa assignments](https://thepracticalr.wordpress.com/2016/12/07/secret-santa-picker-2-using-r/) popped up in my Facebook feed the other day. This being the Christmas season, I took a look. It seemed a bit complex, and I wondered if I could make it more efficient.

The [original function](https://thepracticalr.wordpress.com/2016/12/07/secret-santa-picker-2-using-r/) took in a list of people (e.g., Ann, Bill, Chris). Using a nested while-for loop, it randomly picked a name (Bill), and matched it to the first person in the list (Ann:Bill). If the same person was picked (Ann:Ann), it picked again to try to find another name. In the end it returned results like this:

```
> secret_santa(3, santas)
     [,1]    [,2]  [,3]   
[1,] "Ann"   "has" "Bill" 
[2,] "Bill"  "has" "Chris"
[3,] "Chris" "has" "Ann"  
```

I guessed that another way to do it might be to just randomize the list and then have each person be assigned the next one on the list. This is what I came up with.


```r
secret_santa <- function(santas) {
  random_santas <- sample(santas) # Randomize santas
  recipients <- random_santas[c(2:length(random_santas), 1)] # Move everyone down one
  gift_assignments <- paste(random_santas, "has", recipients) # Make readable list
  data.frame(sort(gift_assignments)) # Return alphabetically ordered list
}
```

Here's an example of how it works.


```r
set.seed(1)
santas <- c("Maria", "Tiffany", "Mike", "Bob", "Josh", "Melanie")
secret_santa(santas)
```

```
##   sort.gift_assignments.
## 1          Bob has Maria
## 2       Josh has Tiffany
## 3         Maria has Josh
## 4       Melanie has Mike
## 5           Mike has Bob
## 6    Tiffany has Melanie
```

In doing more research on this problem, it turns out that the assignment of Secret Santas is related to the mathematical concept of derangements. From [Wikipedia](https://en.wikipedia.org/wiki/Derangement): "In combinatorial mathematics, a derangement is a permutation of the elements of a set, such that no element appears in its original position." 

The number of derangements of a set of size *n* can be calculated as: 

$$!n=n!\sum _{i=0}^{n}{\frac {(-1)^{i}}{i!}}$$

My simplistic solution consists of one closed cycle of people (everyone just gives to the next person). However, there are also derangements such as A:B, B:A, C:D, D:C where you have a cycle of people that is 2 or more in length. 

This could be something you would want to avoid. For example you could have a big family Secret Santa and a husband could be paired to his wife and vice-versa. My solution would avoid there being a small closed cycle like this, but it would still permit husband:wife. 

My solution is a subset of the total possible derangements, and this begs the question, what percentage of all possible derangements are a cycle of the whole group? 

A quick Google search brought up [this paper](https://www.rose-hulman.edu/mathjournal/archives/2006/vol7-n1/paper5/v7n1-5pd.pdf) which in turn led me to [this paper](http://www.jstor.org/stable/3622033). My math skills are very rusty at this point, so I'm not sure what the answer is, but it seems that an exact solution can be calculated!
