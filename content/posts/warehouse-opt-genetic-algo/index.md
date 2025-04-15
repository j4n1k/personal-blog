---
title: "Solving the Storage Location Assignment Problem with Genetic Algorithms"
description: "Discover how to optimize warehouse storage with our MILP formulation for the Storage Location Assignment Problem (SLAP). Learn about order picking strategies, mathematical modeling, and efficient item-to-storage assignments to reduce costs and improve logistics."
date: 2025-04-15
draft: false
summary: "Explore how genetic algorithms can solve the Storage Location Assignment Problem (SLAP) more efficiently. Learn to implement a GA in Python, use a MILP-based fitness function, and optimize warehouse logistics by improving item placement and reducing picking costs."
tags: ["optimization", "metaheuristics", "logistics", "SLAP", "MILP", "Supply Chain", "Genetic Algorithm"]
series: ["SLAP"]
series_order: 5
showViews: true
---

<div class="newsletter-box">
  <h3>Subscribe to our Newsletter</h3>
  <!-- Paste your MailerLite embed code here -->
  <form action="https://YOUR_MAILERLITE_FORM_URL" method="post" target="_blank">
    <input type="email" name="fields[email]" placeholder="Your email address" required>
    <button type="submit">Subscribe</button>
  </form>
</div>

## Introduction
In a previous blog post, we explored the Storage Location Assignment Problem, a common logistics challenge faced by businesses. We discussed its importance and how it can be solved using optimization tools like Gurobi. We saw that while it is possible to formulate the problem as a mixed integer problem and solve it optimally, this approach takes a lot of time to compute.

In this sequel, we’ll take a different approach and delve into the world of genetic algorithms to tackle the same problem. Genetic algorithms are a class of optimization techniques inspired by the principles of natural selection and evolution. Let’s explore how they can be used to find solutions to the Storage Location Assignment Problem.

We will cover genetic algorithm basics, repairing mechanisms, how to encode a MILP objective function as a fitness function and much more! Furthermore you will learn how to code up your own genetic algorithms for combinatorial problems in python.

## Genetic Algorithms Basics
To start, it’s essential to understand the core concepts of genetic algorithms. I will only provide you with the basics, if you are interested in learning more you can read about it in depth here.

At their core, genetic algorithms mimic the process of evolution in nature. They operate with a population of potential solutions, represented as chromosomes. These chromosomes contain genes, which encode information about a candidate solution.

While genetic algorithms are often individual to a given problem tehy often follow the same building blocks. These are:

*   **Selection**: Genetic algorithms favor individuals (solutions) with higher fitness. This is similar to the survival of the fittest in nature.
*   **Crossover**: This operation combines genetic material from two parent solutions to create one or more offspring solutions.
*   **Mutation**: Random changes are introduced in an attempt to explore new areas of the solution space and maintain diversity.


## Adapting the Problem to Genetic Algorithms
For the Storage Location Assignment Problem, we need to adapt the problem to fit the genetic algorithm framework. First, we must define a representation for potential solutions (chromosomes) and a way to evaluate their quality (fitness function).

*   **Representation**: For genetic algorithms often a binary representation of the solution is choosen. For this problem we want to encode the solution a little bit different. A chromosome can represent a set of storage locations for products. Each gene corresponds to a product and encodes its assigned storage location.
*   **Fitness Function**: The fitness function should evaluate how well a solution assigns storage locations while considering factors like distance, capacity, and cost. The good news is we already formulated a fitness function in our last part on MILP! We’ll simply use the objective function of the SLAP-MILP as the fitness function.


## Genetic Algorithm Implementation
Now we have all building blocks to actually code our GA. Implementing a genetic algorithm involves several steps:

<div style="background-color:white; padding: 20px">
{{< mermaid >}}
flowchart TD
    A([Begin])
    B[Initial population]
    C[Calculate the fitness value]
    D[Selection]
    E[Crossover]
    F[Mutation]
    G{Is termination<br>criteria satisfied?}
    H([End])

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G -- No --> C
    G -- Yes --> H
{{< /mermaid >}}
</div>

Let's implement this loop step by step.

### Initialization

First we need to create an initial population of potential solutions. This can be random or based on heuristics. We have to choose the number of individuals in the population. We will assign the products to random storage locations. To that we simply shuffle the list with the storage locations and draw a random location for each product. Then we append the location to the chromosome. Now each index of the chromosome represents one product with a storage location as the value.

```python
import random
import numpy as np

products = []
locations_list = []
def generate_genome(products, locations_list):
    chromosome = []
    random.shuffle(locations_list)
    for i in range(len(products)):
        storage_loc = locations_list[i]
        chromosome.append(storage_loc)
    return chromosome

initial_population = [generate_genome(products, storage_locs, warehouse) for i in range(200)]
```

### Selection
Now we have 200 individuals with randomly assigned storage locations. How do we know which individual has the best assignment and should be used for reproduction? Easy — we use our fitness function!

First we need to code our fitness function to score a given individual. Recall the mathematical description from the last part:

{{< figure
    src="mat_model.png"
    alt="Mathematical Model"
    caption="Mathematical Model of the SLAP"
    >}}

For our fitness function we need all the sets and parameters. In the first part I showed you how to generate the distance between storage locations, order frequencies and how to calculate the product affinity. Please have a look at part one if you are interested in how to do it. For now we will simply assume to have this data available.

With the data available we can formulate the fitness function as follows:

```python
def fitness(individual, dist_mat, product_affinity):
    score = 0
    for p in range(len(individual)):
        for p2 in range(len(individual)):
            if p == p2 or product_affinity[p][p2] == 0:
                continue
            else:
                score += product_affinity[p][p2] * dist_mat[individual[p]][individual[p2]]
    return score
```

Just like in the MIP formulation we loop twice over the individual. That means we iterate over every product combination (recall that each index of a individual represents a product and the value at this index is the location). To make the code more efficient we skip products that do not have an affinity, meaning they were never ordered together. For non zero product pairs we can calculate their score by multiplying the product affinity with the distance of the product pair to each other. By doing this for every product and summing it up we get an indication of fitness for an individual.

Now remember that we need to score our total population — that means we need to call fitness_function() for every individual in our starting population. We can do this with a simple list comprehension:

```python
fitness_result = [fitness(individual, dist_mat, product_affinity) for individual in initial_population]
```

Now we have the fitness values for all individuals in the initial population. As we formulated the SLAP as a minimization problem we now need to select the individuals with the lowest fitness value. Let us formulate a function to that for us:

```python
def selection(population, scores, n=2):
    best_score = np.inf
    scores_np = np.array(scores)
    index_min = np.argpartition(scores_np, n)[:n]
    selected = [] 
    for i in range(n):
        if scores[index_min[i]] < best_score:
            selected.append(population[index_min[i]])
    return selected

parents = selection(initial_population, fitness_result)
```

What we did here is called a rank selection: we basically take the two values with the lowest value and return them. There are a lot more advanced selection methods, but that is a topic for another time.

### Crossover
What is better than a good individual? Two good individuals! So why not combine their genes? That’s exactly what the crossover step of a genetic algorithm does. It combines genetic information from the selected individuals in the previous step. This resembles the natural process of reproduction as this process creates offspring solutions that inherit traits from their parents. Again there are a lot of different approaches to this. We will stick with a simple version, the random point crossover. Like the name suggests we determine a random index in the individual and take the genes of the left side of this point from the first individual and the genes of the right side from the second individual. As this is maybe confusing I provided a illustration below:

{{< figure
    src="crossover.png"
    alt="Crossover Vis"
    caption="Crossover visualization [1]"
    >}}

This is very simple in python:

```python
def random_point_crossover(parents):
    i = random.randint(0,len(parents[0]))
    child = parents[0][:i] + parents[1][i:]
    return child
```

One problem that is very important here is that the children created by the crossover are most likely invalid! Often binary representations are used where the values of a chromosome are either 0 or 1. In our case we have the storage locations as values that are individual. Simply crossing them leads to possible duplicates and values missing from the solution space. This is a common problem for problems with a combinatorial solution space. Therefore we have to repair the individuals.

```python
def find_duplicates(l):
    counts = dict(Counter(l))
    duplicates = {key:value for key, value in counts.items() if value > 1}
    return duplicates

def repair(child, parents):
    duplicates = find_duplicates(child)
    present_locs = set(child)
    all_locs = set(parents[0])
    not_present = list(all_locs - present_locs)
    for dup in duplicates.keys():
        child[child.index(dup)] = not_present.pop(0)
    return child

def crossover(parents):
    child = random_point_crossover(parents)
    child = naive_conflict_resolver(child, parents)
    return child

child = crossover(parents)
```

Again there a numerous approaches and repair mechanisms. Advanced methods include local search approaches that will improve the solution by replacing the duplicate values in a way that the solution will be improved [2]. I propose a very simple solution. First we need to find all duplicates in the child. Then we find out what locations are missing from the child. Then we replace all duplicates with a randomly selected location that is missing. Now we have a intact gene and can move on.

### Mutation

To ensure that our solution doesn’t get stuck in local optima we need to introduce random changes to some individuals to explore new solutions. We will stick with the basics and us a random mutation.

```python
def mutation(to_mutate):
    idx_1, idx_2 = random.sample(range(0, len(to_mutate)-1), 2)
    node1 = to_mutate[idx_1]
    node2 = to_mutate[idx_2]
    to_mutate[idx_1] = node2
    to_mutate[idx_2] = node1
    return to_mutate
```

### Putting it all together

Now we have all components and can write our optimization loop.

```python
fitness_tracker = []
min_fitness = np.inf
n_generations = 500

# initialize the population
initial_population = [generate_genome(products, storage_locs, warehouse) for i in range(200)]

for g in range(n_generations):
    new_pop = []
    pop_results = []
    
    fitness_result = [ga.fitness(individual, dist_mat, product_affinity) for pop in initial_population]
    print(g, min(fitness_result))
    fitness_tracker.append(min(fitness_result))
    for _ in range(100):
        parents = selection(initial_population, fitness_result)
        child = crossover(parents)
        mutated = mutation(child)
        new_pop.append(mutated)
    initial_population = new_pop

min_fitness = min(fitness_result)
min_child = initial_population[fitness_result.index(min_fitness)]
```

### Fine-Tuning the Algorithm

Fine-tuning is crucial to ensure that the genetic algorithm performs well for your specific problem. You’ll need to experiment with parameters like population size, mutation rate, and selection methods to achieve the best results. I only implemented simple approaches to the different steps of the GA, feel free to integrate more advanced solutions to your specific problem!

## Results and Performance Analysis

Let us first do a simple sanity check. We take again a small toy problem with six storage locations and compare the objective values of the Gurobi solution with our fitness value.

First let us look at the Gurobi solution. After a runtime of a few seconds we get an objective value of 810.

```
Optimal Objective Value (Gurobi): 810.0
```

Let’s compare this to the fitness value of our individual after 500 generations:

```
Generation    Fitness
-----------------------
496           810.0
497           810.0
498           810.0
499           810.0
```

Awesome, our algorithm seems to do the right thing! But where our approach really shines is the solution of large instances. So let us look at how our solution performs on a large problem. For that we generate a problem with 2964 storage locations.

We let our algorithm run for 500 generations. This takes about 45 minutes. In the figure above you can see the fitness value for each episode. As expected the value gets minimized over time.

{{< figure
    src="ga_fitness_lineplot.png"
    alt="GA fitness value over 500 generations"
    caption="GA fitness value over 500 generations"
    >}}

Even if 45 minutes could still be considered as a rather long run time you have to compare this to the MILP solution: It took us nearly one hour to solve this problem for 11 items and storage locations!

## Comparing Genetic Algorithms to Gurobi

Now, let’s compare the genetic algorithm approach to solving the Storage Location Assignment Problem with the Gurobi-based method discussed in the previous post.

*    Genetic algorithms offer a different approach, leveraging the principles of evolution.
*    Gurobi is a powerful optimization tool that provides exact solutions but may not be suitable for highly complex or large-scale problems. We have seen that it becomes nearly impossible to solve instances of more then 11 items/ storage locations
*    Our GA can handle much larger instances but we don’t know how good our solution is when we don’t have a optimal solution to compare to
*    Genetic algorithms are more versatile and can handle a wider range of problems, including those with non-linear objectives and constraints.

## Conclusion

In this post, we explored the application of genetic algorithms to the Storage Location Assignment Problem. We discussed the core concepts of genetic algorithms, their adaptation to this problem, and the implementation steps. While Gurobi offers exact solutions, genetic algorithms provide an alternative approach for optimization problems, especially when dealing with complex, dynamic, or large-scale scenarios. Experimenting with parameters and fine-tuning the algorithm can lead to impressive results. As you explore the world of optimization, remember that different problems may benefit from different tools, and genetic algorithms offer a powerful option in your toolkit. I encourage you to try implementing genetic algorithms for similar optimization challenges and share your experiences with me.


## References
\[1\] https://www.tutorialspoint.com/genetic_algorithms/genetic_algorithms_crossover.htm

\[2\] https://www.researchgate.net/publication/361467152_Local_Search_in_Selected_Crossover_Operators?_tp=eyJjb250ZXh0Ijp7ImZpcnN0UGFnZSI6InB1YmxpY2F0aW9uIiwicGFnZSI6InB1YmxpY2F0aW9uIn19