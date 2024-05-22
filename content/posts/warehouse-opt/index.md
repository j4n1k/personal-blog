---
title: "The Storage Location Assignment Problem: A MILP Formulation"
description: "Discover how to optimize warehouse storage with our MILP formulation for the Storage Location Assignment Problem (SLAP). Learn about order picking strategies, mathematical modeling, and efficient item-to-storage assignments to reduce costs and improve logistics."
date: 2024-05-20
draft: false
summary: "Explore the complexities of the Storage Location Assignment Problem (SLAP) and how Mixed Integer Linear Programming (MILP) can provide solutions for optimizing warehouse storage. Enhance your logistics strategy with this comprehensive guide."
tags: ["optimization", "metaheuristics", "logistics", "SLAP", "MILP", "Supply Chain"]
series: ["SLAP"]
series_order: 2
showViews: true
---
{{< katex >}}
The code for this series is available here:
{{< github repo="j4n1k/beware" >}}
## The Problem

Warehouse managers face increasing pressures due to rising costs, evolving customer demands, shorter product lifecycles, and workforce turnover. Since **order picking** within a warehouse **accounts for up to 50%** of the expenses, reducing picking times is a critical lever to cut costs and streamline processes \[1\]. Shortening pick times can be achieved through **improved picking strategies** and **efficient assignment of items to storage locations**. In this post we want to explore how to enhance item-to-storage location assignments, starting with an overview of the underlying decision problem and diving into solutions.

## Understanding the SLAP

The Storage Location Assignment Problem (SLAP) revolves around the allocation of products to storage locations. It depends on various factors, including warehouse characteristics, product types, and demand profiles. 

Often it is aimed to place the products in such a way that products with high demand are placed in the most favorable positions, e.g. close to the input- and output-points where the pickers start end end their picking routes. Solving the SLAP for this objective comes down to sorting the products by historic demand and assigning the item with the highest demand to the storage location that is closest to the I/O-point. This is pretty straight forward and doesn't require advanced methods.

In this post we want to consider a more challenging problem where we want to assign products in such a way that products that are frequently ordered are placed close to each other. 

This problem shares similarities with the Quadratic Assignment Problem and is therefore NP-hard. We will see later what implications that brings for solving the problem. 

Mathematically, the SLAP can be expressed as follows:

**MIP formulation of the SLAP**

Sets:
$$
I = \text{Items}
$$

$$
J = \text{Storage Locations}
$$
Parameters:
$$
a_{hi} = \text{Affinity between item h and i}
$$

$$
d{jk} = \text{Distance between storage location j and k}
$$
Variables:
$$
y_{h,j} =
\begin{cases} 
1 & \text{if item } h \text{stored in location } j \\
0 & \text{else}
\end{cases}
$$

$$
y_{i,k} =
\begin{cases} 
1 & \text{if item } i \text{stored in location } k \\
0 & \text{else}
\end{cases}
$$

Objective function:

The objective is to minimize the total cost associated with storing items in the warehouse. This cost is typically related to the distance traveled to retrieve items that are frequently used together (i.e., items with high affinity should be stored close to each other).
$$
 \min Z = \sum_{h \in I} \sum_{i \in I, i \neq h} \sum_{j \in J} \sum_{k \in J, k \neq j} d_{j,k} \cdot a_{h,i} \cdot y_{i,k} \cdot y_{h,j}
$$
Constraints:

Each item is assigned to exactly one location:  
$$
 \sum_{j \in K} y_{h,j} = 1, \quad \forall h \in I
$$

Each location holds exactly one item:
$$
\sum_{h \in I} y_{h,j} = 1, \quad \forall j \in K
$$

Binary assignment variables:

These constraints ensure that the decision variables are binary, meaning that an item is either assigned to a location (1) or not (0).
$$
y_{h,j} \in \{0,1\}, \quad \forall h \in I, \forall j \in K
$$

$$
y_{i,k} \in \{0,1\}, \quad \forall i \in I, \forall k \in K
$$


## Toy Problem
In this section we will construct a simple setup for generating warehouse layouts and historic order data. 
For the rest of this post we will work with a small toy problem for which we can solve the SLAP. Lets consider a warehouse with the following properties:

*   one aisle
*   two racks
*   the aisles can be accessed from top and bottom
*   every rack has three storage locations resulting in six storage locations in total
### Layout Generation
To quickly generate simple warehouse layouts I created a python program that takes as the input the dimensions of the warehouse in x, y and z dimensions. 

It outputs a "Layout" object that contains a representation of the layout as a multidimensional numpy array. In the numpy array storage locations are encoded by the dummy value "-1" and walkable locations by "0". Later we can replace the dummy value of the storage location by the id of the product that is stored. The first and last row are cross-aisles that are used to access the aisles. Left and right to the aisles the storage locations are placed.

The code snippet below shows the creation of a Layout object and the numpy representation of the simple warehouse we will use in this article. It can be called by calling the "grid" property of the Layout object.
```python
warehouse = Layout(x=5, y=3, z=1) 
double_deep = "False"
path = os.path.join("data/cache/dist_mats/", "dist_mat_" + str(warehouse.layout_grid.shape) + "_" + str(double_deep) + ".npy")
if os.path.exists(path):
    dist_mat = np.load(path)
else:
    dist_mat = warehouse.gen_dist_mat(path)
warehouse.grid
      [[ 0.,  0.,  0.],
       [-1.,  0., -1.],
       [-1.,  0., -1.],
       [-1.,  0., -1.],
       [ 0.,  0.,  0.]]
```
The layout matrix is transformed into a graph representation that is used to calculate a distance matrix between each location. The matrix is saved to disc and can be loaded to reduce the runtime. 

### Data Preperation
Finding item-to-storage location assignments requires historical order data. From this data, information such as item order frequencies or item affinity is calculated. For our toy problem it is sufficient to generate the data ourselves. In reality order data can be retrieved from a WMS or online sources like Kaggle.

We first generate the affinity between the items in storage. We set the maximum affinity to 9, that means the highest value two items can be ordered together is 9. Then we loop though all the items and randomly select a value for the affinity.

```python
size = len(warehouse.storage_locs)
product_pairs_frequency = np.zeros((size, size))
upper_bound = 9
for i in range(size):
    for j in range(size):
        if i == j: 
            product_pairs_frequency[i][j] = 0
        # since problem is a one-one problem, mapping (1,2) == (2,1) so there's no need to create duplicate mappings
        #if (j,i) in product_pairs_frequency: continue
        else:
            product_pairs_frequency[i][j] = random.randrange(0, upper_bound)
```
In this example we consider a warehouse where there is an item stored at every storage locations. Therefore the number of storage locations and the number of items is equal.
The output will be an array like this:
```python
print(product_pairs_frequency)
[0., 1., 1., 4., 0., 0.],
[5., 0., 5., 3., 3., 4.],
[6., 1., 0., 6., 6., 7.],
[3., 5., 5., 0., 4., 3.],
[4., 0., 0., 7., 0., 6.],
[8., 4., 6., 7., 7., 0.] 
```

We can retrieve the affinity of two items by indexing the matrix with the item indices:
```python
print(product_pairs_frequency[0][3])
4
```
Item 0 and 3 have been ordered 4 times together!

In addition to the product affinity we also need to know how often each item was odered historically.
This can now be determined by going through the product\_pairs\_frequency matrix for every item.

```python
product_frequency = {}
for i in range(size):
    product_frequency[i] = product_pairs_frequency[i].sum()
```

To analyze the layout and order data better I plot them as a heatmap. 
For our exaxmple this results in the follwing heatmap where the black nodes represent walkable locations and the colored nodes represent storage locations. The colors represent the order frequency of the items, with darker colors representing lower order volumes and lighter colors representing higher volumes. Try to hover over the nodes for more information!

{{< load-plotly >}}
{{< plotly json="layout.json" height="400px" >}}

## Solving the SLAP
### Mixed Integer Programming
Now let's start solving the problem.
First let us consider the mixed integer programming formulation proposed in the beginning. The model can be formulated like this in Gurobi:

```python
import gurobipy as gp  
from gurobipy import GRB  
  
model = gp.Model("SLAP_PA")  
  
# decision variables  
y_i_j = {}  
for i in products:  
    for j in storage_locs:   
        y_i_j[i, j] = model.addVar(vtype=GRB.BINARY, name=f"Item{i}_at_Loc_{j}")   
  
# objective function  
  
model.setObjective(gp.quicksum(y_i_j[h, j] * y_i_j[i, k] * dist_mat[j, k] * product_pairs_frequency[h, i]   
                               for h in products for i in products for j in storage_locs for k in storage_locs)),  GRB.MINIMIZE)  
  
# constraints  
for j in storage_locs: #storage_locs_new  
    model.addConstr(gp.quicksum(y_i_j[i, j] for i in products) == 1, name=f"Location{i}_constraint")  
  
for i in products:  
    model.addConstr(gp.quicksum(y_i_j[i, j] for j in storage_locs) == 1, name=f"Item{j}_constraint")  
  
model.optimize()
```

After the problem is solved we can print the solution:

```python
# Print the optimal solution  
if model.status == GRB.OPTIMAL:  
    print("Optimal Solution Found:")  
    for i in products:  
        for j in storage_locs:  
            if y_i_j[i, j].x > 0.5:  # Check if the variable is assigned to 1 (approximately)  
                print(f"Item {i} stored at storage location {j}")  
else:  
    print("No optimal solution found.") 
```

This will result in something like this (depending on the item affinity that is generated at random):

```python
Optimal Solution Found:
Item 0 stored at storage location 11
Item 1 stored at storage location 12
Item 2 stored at storage location 6
Item 3 stored at storage location 9
Item 4 stored at storage location 8
Item 5 stored at storage location 5
```

Let's visualize the new assignment and compare it to the affinity matrix. 
{{< load-plotly >}}
{{< plotly json="new_layout.json" height="400px" >}}

As we can see the items 5 and 2 are stored adjendent to each other, when we consult the affinity matrix we can see that they are also orderd the most often together! 
This looks promising but a warehouse with six storage locations isn’t very useful. When we try to solve this problem with larger warehouses we will quickly see what it means when a problem is NP-complete.

We can not solve large instances of this problem in a reasonable time. Solving this problem with our toy problem with only six locatios takes a few seconds. Solving it with ten locations already takes nine minutes. A model with eleven locations already takes almost one hour to solve! It is obvious that an optimal solution can not be computed in reasonable time.

{{< chart >}}
type: 'bar',
data: {
  labels: ['6', '10', '11'],
  datasets: [{
    label: 'Minutes',
    data: [1, 9, 55],
  }]
},
options: {
  plugins: {
    title: {
      display: true,
      text: 'Runtime of MIP model for different instances'
    }
  },
  scales: {
    x: {
      title: {
        display: true,
        text: '# Storage Locations'
      }
    }
  }
}
{{< /chart >}}


We have shown that due to the NP-hard nature of SLAP, finding an optimal solution within a reasonable time frame is challenging, especially for large-scale problems. This complexity calls for the use of advanced optimization techniques such as heuristic methods, metaheuristic algorithms (e.g., genetic algorithms, simulated annealing), and approximation algorithms. These approaches can provide near-optimal solutions more efficiently compared to exact methods, which may be computationally prohibitive for large instances.

Therefore in the next part of this series we will look into a genetic algorithm for solving the SLAP.

# References
\[1\] R. de Koster, T. Le-Duc, and K. J. Roodbergen, “Design and control of warehouse order picking: A literature review,” European Journal of Operational Research, vol. 182, pp. 481–501, Oct. 2007.

\[2\] M. Kofler, A. Beham, S. Wagner, and M. Affenzeller, “Affinity based slotting in warehouses with dynamic order patterns,” in Topics in Intelligent Engineering and Informatics, pp. 123–143, Springer International Publishing, 2014.
