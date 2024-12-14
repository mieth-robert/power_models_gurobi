---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.11.5
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

Notation used in this chapter:

| **Variables/Parameters** |               |
|-------------------------|--------------------------------|
| $p$                    | Active power production        |
| $c(p)$                 | Cost of power production       |
| $D$                    | Demand                         |
| $\overline{P}$         | Production capacity            |
| $G$                    | Number of generators           |

| **Indices**            |                                |
|-------------------------|--------------------------------|
| $i$                    | Generator index                |
| $t$                    | Timestep index                 |



# Economic Dispatch

Economic dispatch is concerned with finding the least-cost combination of generation resources to match a given demand. 

## Economic Dispatch

In its most basic form, economic dispatch is written as 


\begin{align}
\min \quad 
    & \sum_{i\in[G]}c_i(p_i) \\
\text{s.t.} \quad 
    & \sum_{i\in[G]}p_i = D \\
    & p_i \le \overline{P}_i && \forall i
\end{align}



```{code-cell}
# import needed librarys

import numpy as np
import gurobipy as gp
from gurobipy import GRB

# define paramters

n_gen = 4 # number of generators 
c2 = np.array([0.148, 0.034, 0.158, 0.106]) # quadratic term of generator cost function 
c1 = np.array([199.4, 232.7, 246.1, 168.0]) # linear term of generator cost function 
c0 = np.array([16625, 16806, 15483, 16689]) # constat term of generator cost function

D = 1500 # demand 

# define the model

m = gp.Model()
m.setParam('OutputFlag', 0)

# variables
p = m.addVars(4, lb=0, ub=GRB.INFINITY, name="p") # generator production 

# constraints
m.addConstr(p.sum() == D, name="lam") # energy balance

# define objective
m.setObjective(
    sum(c2[i]*(p[i]**2) + c1[i]*p[i] + c0[i] for i in range(n_gen)),
    GRB.MINIMIZE
)

# run the model
m.optimize()

# prepare results
p_res = [m.getVarByName(f'p[{i}]').X for i in range(n_gen)]

# explore results
for i, value in enumerate(p_res):
    print(f"Production of generator {i + 1}: {value:.2f} MW")
```




