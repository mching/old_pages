---
title: "Cost of Energy to Operate Electric vs. Hybrid Car in Honolulu"
output: html_document
layout: post
---

Our family has an electric car (Nissan Leaf) and a hybrid (Toyota Prius), and we often have to choose which one we will take out on the weekend. I wondered which one was cheaper to operate and which one was better for the environment.

## Wallet Impact

The Leaf travels about 4.8 miles per kilowatt hour (kwh), and the Prius gets about 55 miles per gallon (mpg). Our electricity costs about $0.35 per kwh, and gas prices have been around $2.50 per gallon.

Based on these figures, the Leaf costs $0.35 per kwh divided by 4.8 miles per kwh, or $0.073 per mile. The Prius costs about $2.50 per gallon divided by 55 miles per gallon, or $0.045 per mile. 

The Prius is a lot cheaper with these figures, assuming we get the electricity from the electric company. But what if we have another source for electricity?

We have photovoltaic panels on our house, and they generate most of our electricity need. We spent about $15,000 out of pocket for the system after rebates, and it generates about 10-20 kwh per day. Let's just call it 12 kwh per day on average. The system is rated for 20 years, so I can break the costs down by the up front cost divided by the electricity generated over 20 years. After 20 years with the system, we could be expected to generate 12 * 365 * 20 or 8.76 x 10^4 kwh. Ignoring the discount rate, that makes our cost per kwh $0.17. At that rate, the cost per mile for the electric car is $0.036. So with our photovoltaic panels, the Leaf is actually cheaper than the $0.045/mile for the Prius!

### Nomogram for Electric vs Gasoline Car Energy Price
Electricity prices and gas prices have a habit of changing fairly often, so I created a figure to help decide which car to take.

To determine what gas price would have to be to match a given electricity price, we use the equations for cost per mile for gas and cost per mile for electricity, and set them equal to each other.

$$gas\:cost\:per\:mile = \frac{gas\:price\:per\:gallon}{mpg}$$

$$electricity\:cost\:per\:mile = \frac{electricity\:price\:per\:kwh}{miles\:per\:kwh}$$

The break-even point is when the gas cost per mile equals the electricity cost per mile

$$\frac{gas\:price\:per\:gallon}{mpg} = \frac{electricity\:price\:per\:kwh}{miles\:per\:kwh}$$

Solving for gas cost, we get:

$$gas\:price\:per\:gallon = \frac{electricity\:price\:per\:kwh}{miles\:per\:kwh} \times mpg$$

So if electricity costs $0.17 per kwh, then the break-even gas cost would be:

$$gas\:price\:per\:gallon = \frac{0.17}{4.8} \times 55 = 1.95$$

We can plot this equation.


```r
library(ggplot2)
mpg = 55
miles_per_kwh = 4.8
electricity_cost = seq(0.1, 0.5, by=0.001)
gas_cost = electricity_cost * mpg / miles_per_kwh
car_cost <- data.frame(gas_cost, electricity_cost)
ggplot(car_cost, aes(x = electricity_cost, y = gas_cost)) + geom_point() + 
  ggtitle("Equivalent Gas and Electricity Costs") +
  ylab("Gas price ($/gal)") +
  xlab("Electricity price ($/kwh)")
```

![](https://github.com/mching/mching.github.io/raw/master/images/car1.png)

For a given electricity price, if the gas price is above the line, then the gas car is more expensive to operate. If the gas price is below the line, the electric car is more expensive.

### A Big Wrench in the Calculation
There's one other big difference between the Prius and the Leaf, which is that the price of the two cars is actually quite a bit different. The Prius cost about $25,000 while the Leaf cost about $35,000. That $10,000 difference, when broken down over 100,000 miles is about $0.10 per mile. 

If we add that extra $0.10 to the Leaf price, well now we have the Leaf costing $0.136 per mile and the Prius costing $0.045 per mile.

### Cost Conclusion
Taking all of the above into account, I would conclude the Prius cost to operate is significantly cheaper than the Leaf. I'm out of time today, but I'll come back to the environmental impact question.

### Edit--November 30, 2016
I discovered a neat report on electric vehicles in Hawaii. It's available for download at [this link](http://evtc.fsec.ucf.edu/publications/documents/HI-09-16.pdf). 

I also discovered a cool link on the eGallon, which is defined as "the cost of fueling a vehicle with electricity compared to a similar vehicle that runs on gasoline." It's here on the [energy.gov website](http://energy.gov/maps/egallon). Basically the price of energy to run an electric vehicle is way cheaper everywhere in the USA except for Hawaii.
