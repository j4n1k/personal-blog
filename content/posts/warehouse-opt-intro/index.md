---
title: "A Gentle Introduction to the Storage Location Assignment Problem"
date: 2024-05-19
draft: false
summary: ""
tags: ["optimization", "metaheuristics", "logistics"]
series: ["SLAP"]
series_order: 3
---
{{< katex >}}
## The Problem

Warehouse managers face increasing pressures due to rising costs, evolving customer demands, shorter product lifecycles, and workforce turnover. Since **order picking** within a warehouse **accounts for up to 50%** of the expenses, reducing picking times is a critical lever to cut costs and streamline processes \[1\]. Shortening pick times can be achieved through **improved picking strategies** and **efficient assignment of items to storage locations**. In this post we want to explore how to enhance item-to-storage location assignments, starting with an overview of the underlying decision problem and diving into solutions.

## Understanding the SLAP

The Storage Location Assignment Problem (SLAP) revolves around the allocation of products to storage locations. It depends on various factors, including warehouse characteristics, product types, and demand profiles. 

Often it is aimed to place the products in such a way that products with high demand are placed in the most favorable positions, e.g. close to the input- and output-points where the pickers start and end their picking routes. Solving the SLAP for this objective comes down to sorting the products by historic demand and assigning the item with the highest demand to the storage location that is closest to the I/O-point. This is pretty straight forward. 

## Mathematical Model

**Sets:**

A set of items is required that are placed at storage locations. We will denote this with "I":
$$
I = \text{Items}
$$

Further we need a set of storage locations, denoted by "J":
$$
J = \text{Storage Locations}
$$

**Parameters:**
First we need to know how often each item has been ordered:
$$
f_{i} = \text{Historic Order Frequency of item i}
$$

Similarly we need a distance matrix that tells us the distance of every location to the picking depot:
$$
d_{j} = \text{Distance between storage location j and the depot}
$$

**Variables:**

The actual decision will be where to store each item. This is a binary decision and can either be "0" if an item is not stored at a certain location or "1" if it is. 

$$
y_{i,j} =
\begin{cases} 
1 & \text{if item } i \text{ stored in location } j \\
0 & \text{else}
\end{cases}
$$

**Objective function:**

The objective is to minimize the total cost associated with storing items in the warehouse. For this problem the cost is related to the distance traveled to retrieve items that are frequently ordered.
$$
 \min Z = \sum_{i \in I} \sum_{j \in J} d_j \cdot f_i \cdot y_{i,j}
$$

**Constraints:**

Finnaly, we have to consider some constraints.

First, we have to make sure that each item is assigned to exactly one location:  
$$
 \sum_{j \in J} y_{i,j} = 1, \quad \forall i \in I
$$

Second, we need to enforce that each location holds exactly one item:
$$
\sum_{i \in I} y_{i,j} = 1, \quad \forall j \in J
$$

These constraints ensure that the decision variables are binary, meaning that an item is either assigned to a location (1) or not (0).
$$
y_{i,j} \in \{0,1\}, \quad \forall i \in I, \forall j \in J
$$

## Implementation
I implemented the model in pyomo:

```python
from pulp import LpProblem, LpMinimize, LpVariable, lpSum, LpBinary

prob = LpProblem("SLAP", LpMinimize)
y_i_j = LpVariable.dicts("Storage location i occupied by product j", (storage_locs, products), 0, 1, LpBinary)
prob += lpSum(y_i_j[i][j] * dist_mat[depot, i] * product_frequency[j] for i in storage_locs for j in products)

for i in storage_locs:
    prob += lpSum(y_i_j[i][j] for j in products) == 1

for j in products:
    prob += lpSum(y_i_j[i][j] for i in storage_locs) == 1
``` 

After solving it we can vizualize the assignment with a 3-D scatter plot:

{{< load-plotly >}}
{{< plotly json="warehouse_3d_sorted.json" height="400px" >}}

If you want to try it out for yourself feel free to do so in this google colab notebook:
{{< gist j4n1k 12a12d9a155b88d98416ba9e23aedcf0 >}}

