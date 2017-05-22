# Economical Iced Coffee



My wife and I like iced coffee but drinking it at Starbucks is expensive. I've been making it myself at home and it's a reasonable facsimile at what I think is a much lower price. I decided to run the calculations to see how much we're saving.

## Grounds
I tried a bunch of different coffees to see what would work the best: [whole bean Kirkland (Costco) brand](https://www.costco.com/Kirkland-Signature-House-Blend-Coffee-2-lb%2c-2-pack.product.100232740.html), whole bean Kona coffee, ground [Lion coffee](http://www.lioncoffee.com/) (a local brand), ground Folgers, and [ground Kirkland](https://www.costco.com/Kirkland-Signature-Colombian-Coffee-3-lb%2c-2-pack.product.100120060.html). I found that the whole bean tended to be too weak, possibly because I didn't grind it finely enough. The ground Folgers is much finer, but it had a lot of residual grit that made it slow to filter.

My favorite has been the [pre-ground Kirkland brand](https://www.costco.com/Kirkland-Signature-Colombian-Coffee-3-lb%2c-2-pack.product.100120060.html). It comes in a 1.36 kg can for about $10. The grind is fine, and it doesn't seem to have too much dust that clogged the filter. It makes a nice dark flavorful brew, and it's hard to beat that price.

## Extraction
Based on various recipes on the internet (e.g., [NY Times](https://cooking.nytimes.com/recipes/1017355-cold-brewed-iced-coffee), [GH](http://www.goodhousekeeping.com/food-recipes/easy/a19854/iced-coffee-recipe/)), I place 2/3 cup (67 g) in a container and pour 3 cups (710 mL) of water over it. I tried steeping it for 12 hours, but this wasn't strong enough for our taste. I steep for 24 hours, and it's perfect.

## Filtering
I strain first through a wire mesh strainer to remove the coarse grounds. The resulting product is still pretty gritty because of the fine particles. This goes through a pour-over coffee filter with a cone filter in it. Because the ground coffee absorbs water, I get about 600 mL of coffee concentrate after filtering.

## Dilution
I like to dilute my coffee concentrate about 1:1 with water before adding ice. I use 6 oz (177 mL) of concentrate with 6 oz of water and then some ice to make a 12 oz drink.

## Cost is 95% Cheaper
The cost of the coffee per 600 mL recipe is $0.49 (67 g * $10 / 1360 g). The cone filters are about $0.10 each. This breaks down to $0.17 per 177 mL concentrate to make one serving of coffee, a 95% discount from the iced coffee at Starbucks! In Hawaii, our sales tax is 4.5%, so the cost of homemade coffee is approximately the same as the sales tax on store-bought iced coffee. 

Here's the calculation:


```r
coffee_price <- 10/1360 # cost per gram
coffee_price_per_recipe <- coffee_price * 67 # 67 g in 2/3 cup
volume_made_per_recipe <- 600 # mL
serving <- 177 # mL
filter_price <- 0.10 # dollars
cost_per_serving <- (coffee_price_per_recipe + filter_price) / volume_made_per_recipe * serving
cost_per_serving # in dollars
```

```
## [1] 0.1748309
```

## Discussion
Over one year, the savings is considerable. If we drank iced coffee only every other day, we would save $531.

Actually probably any coffee you can buy at the grocery store is going to be cheaper than Starbucks, so it's not a surprise. Even if you use coffee that's $20 for 10 oz (9 times more expensive than the Kirkland brand ground), it'll be cheaper ($1.63 per serving). 

## Conclusion
Making iced coffee at home costs about $0.17 per serving, a savings of about 95% of the cost of having iced coffee at Starbucks. 
