---
layout: post
title: 'Game AI: Snake Game'
comments: true
---

![snake]({{ site.url }}/{{ site.baseurl }}/images/2017-07-08-Game-AI:Snake-Game/snake.gif){: .center-image}

{:refdef: style="text-align: center;"}
[https://github.com/leook0209/Genetic-Algorithm-Snake](https://github.com/leook0209/Genetic-Algorithm-Snake)
{: refdef}

Snake game is a simple game in which the player moves the head of the snake up, down, right or left to eat a randomly generated food. The snake grows its size by one every time it eats the food, and the snake dies once it hits any part of its body. This project is about training an utility-based snake game agent using a genetic algorithm with a number of heuristics.


# Fitness

Goal(fitness) of each game is to have the snake’s length as long as possible, while taking as minimum as possible turns to finish the game. Fitness is calculated by following way:

<center> fitness = Length of Snake- α * ( Number of Turns taken) </center>

<center> α: weight for Number of Turns Taken </center>

The reason for minimizing the number of turns is that snake game has a very easy strategy to beat, which is to circle the snake around the edge and eat the food at the inner side of the field in a safe manner. Hence, α should be set with a reasonable number in order to prevent the agent to take an easy way out.

At each turn, the agent calculates a heuristic value for moving each direction: up, down, left and right. If a direction leads to the snake’s death, the heuristic value is NEGATIVE INFINITY. Even if a position next to the head contains the food, the snake might not decide to take it if it leads to a less desirable future state (e.g. creating a dead end). To prevent the Snake from taking the same motion over and over again, it is designed to be more attracted towards the food with time.


# Heuristics

There are 6 heuristics the agent uses to calculate the fitness:

1. Manhattan distance between the Snake’s head and Food
2. The position of the Snake’s head from the center of the field
3. Squareness of the Snake
4. Compactness of the Snake
5. Connectivity of the field
6. Dead End Indicator
7. First two heuristics are quite intuitive to understand while next four heuristics are not. Those are heuristics concepts I created for this agent.


## Squareness

Squareness is an indicator of how the Snake’s body is orientated in a square/rectangular manager.

<center> O – Empty Space,  S – Snake,  X – Blank Space,  H – Snake’s Head

{% highlight js %}
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O S S S H O O O O O O O O O
O O O O O O O S S S S O O O O O O O O O
O O O O O O O S S S S O O O O O O O O O
O O O O O O O S S S S O O O O O O O O O
O O O O O O O S S S S O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
{% endhighlight %}

</center>

The above example’s snake is oriented in a perfect rectangular manner. In this case, the squareness value is 0.

<center>
{% highlight js %}
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O S X X X O O O O O O O O O
O O O O O O O S X X X O O O O O O O O O
O O O O O O O S X X H O O O O O O O O O
O O O O O O O S S S S O O O O O O O O O
O O O O O O O S S S S O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
{% endhighlight %}
</center>

The squareness value is the number of blank spaces that is within the Snake’s square boundaries but not filled by the snake. The square boundaries refers to the rectangular space taken up by the leftest, rightest, upper most, and lower most part of the snake. For the above case, the squareness value is the number of Xs, which is 8.


## Compactness

Compactness is an indicator of how compactly the Snake’s body is oriented. It is the number of cases where one body part of the Snake is placed next to another body part of the Snake, without double counting.

<center> O – Empty Space,  S – Snake,  H – Snake’s Head

{% highlight js %}
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O S S S H O O O O O O O O O
O O O O O O O S S S S O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
O O O O O O O O O O O O O O O O O O O O
{% endhighlight %}
</center>

For the above case, the compactness of the Snake is 10.


## Connectivity

Connectivity is an indicator of how connected each part of the field is, and whether the Snake is separating one part of the field from another.

<center> O – Empty Space,  S – Snake,  H – Snake’s Head,  X – Space Chosen by Agent

{% highlight js %}
O O O O O O O O O S S H O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O X O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O S S O O O O O O O O O O
{% endhighlight %}
</center>

At each turn, the agent pick a random empty space in the field, and count how many spaces are disconnected from that space as they are blocked by the Snake’s body. For above case, the connectivity is 148.


## Dead End indicator

Dead End Indicator represents how many spaces are unreachable by the snake based on the current orientation.

<center> O – Empty Space,  S – Snake,  H – Snake’s Head

{% highlight js %}
O O O O O O O O O S S H O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O O S O O O O O O O O O O
O O O O O O O O S S O O O O O O O O O O
{% endhighlight %}
</center>

Dead End indicator is calculated in a similar manner to the Connectivity, except that instead of choosing a random empty space, it checks connectivity from the Snake’s head. For above case, the left side of the field is unreachable from the Snake’s head, hence the Dead End Indicator value is 134.

After a few rounds of training, the genetic algorithm shows a trend to maximize the Compactness and minimize the Distance from Food, Squareness, Connectivity and Dead End, while it does not really care about the Distance from the center of the field.


# Genetic Algorithm
The genetic algorithm for the Snake Game agent has a population size of 500, mutation rate of 0.05, survival rate of 0.5. For each weight sets, the game is played for 3 times and taken the arithmetic average, in order to minimize the effect of randomly generated food positions.


## Crossover
At each generation, the population is sorted based on their fitness value, and two parents are chosen from the 50% of the surviving population, while a Snake with higher fitness having a proportionally higher chance of being chosen. A child is created by taking the weighted average of each heuristic weight of the parents’.

## Mutation
Each heuristic weight of the Snake in the population has 5% chance of being mutated by ±0.2.

The biggest learning point was that it is a bad idea to create a computation intensive program with Web (JavaScript). Due to the performance limitation of the Chrome browser, it took so much time to train the agent with the Genetic Algorithm of population 500. Even after the training, the elite Snake still could not finish the game. The most difficult part is that the food can be generated at an unreachable position. Hence, it is wise to minimize the number of holes generated by the Snake’s body (Connectivity) but then it takes too many steps to clear the game.
