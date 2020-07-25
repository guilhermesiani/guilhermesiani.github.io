---
layout: post
title:  "Quadratic equation and programming"
date:   2020-07-24 22:30:41 -0300
categories: tools
---
Programming is the best method ever to solve problems within a machine and whether combined with math, we can do magical things that Harry Potter would be jealous.

I'm not an expert in math, but when I remember some formulas, I search for on google and try to combine with my codes. It helps me create algorithms with better performance - I mean coding faster in this case - and design, so take a look at this example below.

### How to use the Quadratic Equation to solve a common problem

I could explain what is the Quadratic Equation and its history, but it does not focus here. So I will let this [link][link]{:target="_blank"} to Wikipedia if you want to know more about it.

The problem we will solve is to discover the max profit of a bus fleet when the price is $350 per passenger; the passenger limit is 60; we give a discount of $5 from 40th passenger.

Think, if we give a discount per pessenger from 40th, our profit must decrease after a specific number of passengers, and this number is what we want to know.

Put in mind that this graph is a parabole:

![image](/assets/images/max-profit-bus-fleet-parabole.png)

Defining the function, we have `f(x) = -5x² + 550x`

At this point, we have two tools in mind: Programming and math.

The math formula (Quadratic Equation) is `(-b ± √∆)/(2•a)`.

The programming language I'll use is this example is Golang (but could be any other by your choice).

As a good code has good tests, let's see how it looks like:

{% highlight go %}
package bus_fleet

import (
	"github.com/stretchr/testify/assert"
	"testing"
)

func TestCalcMaxProfit(t *testing.T) {
	assert.Equal(t, 55, CalcMaxProfit(350, 40, 5))
	assert.Equal(t, 60, CalcMaxProfit(350, 50, 5))
}
{% endhighlight %}

And let's see how the code that makes the test pass looks like using this formula:

{% highlight go %}
package bus_fleet

import "math"

func CalcMaxProfit(
	pricePerPassenger float64,
	startDiscount int,
	discount float64,
) int {
	// Find coefficients
	a := discount * -1
	b := pricePerPassenger + (discount * float64(startDiscount))

	// Apply Quadratic Equation formula
	x1, x2 := applyQuadraticEquation(a, b)

	return int((x1 + x2) / 2)
}

func applyQuadraticEquation(a, b float64) (float64, float64) {
	c := make(chan float64)
	go quadraticEquationFormulaByOp(a, b, "-", c)
	go quadraticEquationFormulaByOp(a, b, "+", c)
	return <-c, <-c
}

func quadraticEquationFormulaByOp(a, b float64, op string, c chan float64) {
	divisor := 2 * a
	sqrtDelta := math.Sqrt(math.Pow(b, 2))
	if op == "+" {
		c <- (-b + sqrtDelta) / divisor
	}
	c <- (-b - sqrtDelta) / divisor
}
{% endhighlight %}

I apply a simple design to clean my code in separated functions and use the goroutine to calculate the sum "+" and subtraction "-" operators simultaneously, but this is an extra just to use all the power of the language. In this case, I know that the gains are irrelevant.

Can you see how powerful this combination could be? How much time did I lose thinking about how to solve the math problem? Just the time for the search on Google and the time of learning curve on the programming language (Golang). The programming language, database systems, and other IT tools we can't avoid the learning curve, but the math all the time we have a shortcut, and especially when we don't have time to study all the existing methods and formulas.

I hope you liked this tip and that it helps you to have a better performance.

[link]: https://en.wikipedia.org/wiki/Quadratic_equation
