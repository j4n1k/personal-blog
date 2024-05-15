---
title: "Optimizing Warehouse Efficiency: The Storage Location Assignment Problem"
date: 2023-08-14
draft: false
summary: ""
tags: ["optimization", metaheuristics", "logistics"]
---

Optimizing Warehouse Efficiency: The Storage Location Assignment Problem
========================================================================

[![BeWare](https://miro.medium.com/v2/resize:fill:88:88/1*ey6dArhBLeF_sdCsdI4GJg.png)

](https://medium.com/@beware-sim?source=post_page-----47ac683a7f56--------------------------------)

[BeWare](https://medium.com/@beware-sim?source=post_page-----47ac683a7f56--------------------------------)

·

[Follow](https://medium.com/m/signin?actionUrl=https%3A%2F%2Fmedium.com%2F_%2Fsubscribe%2Fuser%2F438cf4160c0c&operation=register&redirect=https%3A%2F%2Fmedium.com%2F%40beware-sim%2Foptimizing-warehouse-efficiency-the-storage-location-assignment-problem-47ac683a7f56&user=BeWare&userId=438cf4160c0c&source=post_page-438cf4160c0c----47ac683a7f56---------------------post_header-----------)

5 min read·Oct 8, 2023

\--

1

Listen

Share

Photo by [Adrian Sulyok](https://unsplash.com/de/@sulyok_imaging?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com/de/fotos/c_4eaGRDSVU?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash)

_This article is part of a series about optimizing the placement of items in a warehouse with Python. (_[_Part 1_](https://medium.com/@s.saci95/optimizing-warehouse-operations-with-python-part-1-83d02d001845)_) Follow me for Part 2!_

**Table of Contents**
=====================

1.  Introduction
2.  Warehouse Modeling
3.  Solving the SLAP
4.  References

**Introduction**
================

Warehouse managers face increasing pressures due to rising costs, evolving customer demands, shorter product lifecycles, and workforce turnover. Since **order picking** within a warehouse **accounts for up to 50%** of the expenses, reducing picking times is a critical lever to cut costs and streamline processes \[1\]. Shortening pick times can be achieved through **improved picking strategies** and **efficient assignment of items to storage locations**. In this post we want to explore how to enhance item-to-storage location assignments, starting with an overview of the underlying decision problem and diving into solutions.

**Understanding the SLAP**

The Storage Location Assignment Problem (SLAP) revolves around the allocation of products to storage locations. It depends on various factors, including warehouse characteristics, product types, and demand profiles. This problem shares similarities with the Quadratic Assignment Problem and is, therefore, NP-hard. We will see later what implications that brings for solving the problem. Mathematically, it can be expressed as follows:

**Figure 1:** MIP formulation of the SLAP \[2\]

Warehouse Modeling
==================

To develop and compare solutions for SLAP, it’s crucial to find a way to represent a real warehouse. Consider Figure 2 for an example for how a real life warehouse could look like.

**Figure 2:** Conventional warehouse layout with two aisles and three racks

It is not hard to imagine a warehouse like this as a graph like we can see in Figure 3:

**Figure 3:** Graphical representation of the underlying graph

**Toy Problem**
---------------

For the rest of this post we will work with a small toy problem. Lets consider a warehouse with the following properties:

*   one aisle
*   two racks
*   the aisles can be accessed from top and bottom
*   every rack has three storage locations resulting in six storage locations in total

This will result in the follwing layout where the black tiles represent nodes that can be walked on and the blue tiles represent storage locations.

Warehouse layout

This results in the following distance matrix between the storage locations

Distance matrix

**Solving the SLAP**
====================

Data Preperation
----------------

Finding item-to-storage location assignments requires historical order data. From this data, information such as item order frequencies or item affinity is calculated. For our toy problem it is sufficient to generate the data ourselves. In reality order data can be retrieved from a WMS or online sources like Kaggle.

First lets generate a item for every location in our small warehouse:

Now we can generate the affinity between the items. We set the maximum affinity to 9, that means the highest value two items can be ordered together is 9. Then we loop though all the items and randomly select a value for the affinity.

```
size = len(storage\_locs)  
product\_pairs\_frequency = np.zeros((size, size))  
upper\_bound = 9  
for i in range(size):  
    for j in range(size):  
        if i == j:   
            product\_pairs\_frequency\[i\]\[j\] = 0  
        else:  
            product\_pairs\_frequency\[i\]\[j\] = random.randrange(0, upper\_bound)
```

The item order frequency can now be determined by going through the product\_pairs\_frequency matrix for every item.

```
product\_frequency = {}  
for i in range(size):  
    product\_frequency\[i\] = product\_pairs\_frequency\[i\].sum()
```

**Mixed Integer Programming**

First let us consider the mixed integer programming formulation proposed in the beginning. The model can easily formulated like this:

```
import gurobipy as gp  
from gurobipy import GRB  
  
model = gp.Model("SLAP\_PA")  
  
\# decision variables  
y\_i\_j = {}  
for i in products:  
    for j in storage\_locs:   
        y\_i\_j\[i, j\] = model.addVar(vtype=GRB.BINARY, name=f"Item{i}\_at\_Loc\_{j}")   
  
\# objective function  
  
model.setObjective(gp.quicksum(y\_i\_j\[h, j\] \* y\_i\_j\[i, k\] \* dist\_mat\[j, k\] \* product\_pairs\_frequency\[h, i\]   
                               for h in products for i in products for j in storage\_locs for k in storage\_locs)),  GRB.MINIMIZE)  
  
\# constraints  
for j in storage\_locs: #storage\_locs\_new  
    model.addConstr(gp.quicksum(y\_i\_j\[i, j\] for i in products) == 1, name=f"Location{i}\_constraint")  
  
for i in products:  
    model.addConstr(gp.quicksum(y\_i\_j\[i, j\] for j in storage\_locs) == 1, name=f"Item{j}\_constraint")  
  
model.optimize()
```

After the problem is solved we can print the solution:

```
\# Print the optimal solution  
if model.status == GRB.OPTIMAL:  
    print("Optimal Solution Found:")  
    for i in products:  
        for j in storage\_locs:  
            if y\_i\_j\[i, j\].x > 0.5:  # Check if the variable is assigned to 1 (approximately)  
                print(f"Item {i} stored at storage location {j}")  
else:  
    print("No optimal solution found.") 
```

This will result something like this (depending on the item affinity that is generated at random):

Example solution for the SLAP

When we check the affinity matrix for product 0 we can see that it was ordered eight times with product 1!

This looks promising but a warehouse with six storage locations isn’t very useful. When we try to solve this problem with larger warehouses we will quickly see what it means when a problem is NP-complete:

We can not solve large instances of this problem in a reasonable time. Solving this problem with our toy problem with only six locatios takes a few seconds. Solving it with ten locations already takes nine minutes. A model with eleven locations already takes almost one hour to solve! It is obvious that an optimal solution can not be computed in reasonable time.

Runtime of MIP model

Given the complexity of SLAP, solving it typically involves heuristics or metaheuristic search methods. Therefore in the next part of this series we will look into a genetic algorithm for solving the SLAP.

About Me
========

I’m a student passionate about the possibilities of using Operations Research and Machine Learning to improve logistics and supply chain processes.

If you have a similar problem let’s connect on [Linkedin](https://www.linkedin.com/in/janik-b-996a5617b/) or shoot me a [mail](http://janik.bischoff@googlemail.de)!

References
==========

\[1\] R. de Koster, T. Le-Duc, and K. J. Roodbergen, “Design and control of warehouse order picking: A literature review,” European Journal of Operational Research, vol. 182, pp. 481–501, Oct. 2007.

\[2\] M. Kofler, A. Beham, S. Wagner, and M. Affenzeller, “Affinity based slotting in warehouses with dynamic order patterns,” in Topics in Intelligent Engineering and Informatics, pp. 123–143, Springer International Publishing, 2014.
