# Optimization Modeling

## Course Description:
This course explores modeling and solving business decision problems. It covers optimization models including linear programming, integer programming, transportation, assignment, network optimization, and nonlinear programming. Students learn to analyze scenarios, formulate models using software tools, interpret output, and present project reports.

## Learning Outcomes:
- Identify problems appropriate for optimization.
- Understand the challenges with solution techniques with optimization problems.
- Learn to convert written descriptions into optimization problems.
- Learn to solve optimization problems using open source software.

## Software:
- Microsoft Excel Solver 
- Open-souce software [Pyomo](http://www.pyomo.org/about)
- Python
  - Pandas: Importing and manipulating data from Excel files.
  - Pyomo: Formulating and solving optimization models.
  - Matplotlib: Creating plots and visualizations.
  - Seaborn: Generating scatterplots for location visualization.
  - Numpy: Performing numerical computations and array operations.

## Pyomo Optimal Quantity and Placement

### Data Source: Provided by Professor [Excel File](https://github.com/EvanElzenberger/OptimizationModeling/files/11349653/PyomoExam2.xlsx)

### Exam Highlights:
- Problem One: Flavors
  - Objective: Maximize profit by determining optimal production quantities for five flavors (Apple, Banana, Chocolate, Elderberry, and Fig).
  - Considerations: Pricing, costs, fixed setup costs, mixing, preparation, blending, and packaging constraints.
  - Demand Constraints: Ensuring the production quantities satisfy the demand for each flavor.
 ```python
## Initialize Model
model = pe.ConcreteModel()

        ## Define Decision Variables
model.x = pe.Var(DV_indexes, domain = pe.NonNegativeReals)
model.y = pe.Var(DV_indexes, domain = pe.Binary)
# Define Objective function
model.obj = pe.Objective(expr=sum([cost.loc['Price', i]*model.x[i] for i in DV_indexes])
                             - sum([cost.loc['Cost',i]*model.x[i] for i in DV_indexes])
                             - sum([cost.loc['Fixed Setup Cost',i]*model.y[i] for i in DV_indexes]),
                            sense=pe.maximize)
# Define Constraints
model.mixing = pe.Constraint(expr = sum([coef.loc['Mixing',i]*model.x[i]for i in DV_indexes]) 
                                        <= rhs.loc['Mixing', 'rhs'])
model.preperation = pe.Constraint(expr = sum([coef.loc['Preparation',i]*model.x[i]for i in DV_indexes]) 
                                             <= rhs.loc['Preparation', 'rhs'])
model.blending = pe.Constraint(expr = sum([coef.loc['Blending',i]*model.x[i]for i in DV_indexes]) 
                                          <= rhs.loc['Blending', 'rhs'])
model.packaging = pe.Constraint(expr = sum([coef.loc['Packaging',i]*model.x[i]for i in DV_indexes]) 
                                           <= rhs.loc['Packaging', 'rhs'])
# Demand Constraints
#Demand Constraints
model.LinkA = pe.Constraint(expr=model.x['Apple'] <= demand.loc['Apple', 'demand'] * model.y['Apple'])
model.LinkB = pe.Constraint(expr=model.x['Banana'] <= demand.loc['Banana', 'demand'] * model.y['Banana'])
model.LinkC = pe.Constraint(expr=model.x['Chocolate'] <= demand.loc['Chocolate', 'demand'] * model.y['Chocolate'])
model.LinkE = pe.Constraint(expr=model.x['Elderberry'] <= demand.loc['Elderberry', 'demand'] * model.y['Elderberry'])
model.LinkF = pe.Constraint(expr=model.x['Fig'] <= demand.loc['Fig', 'demand'] * model.y['Fig'])
opt = pe.SolverFactory('glpk')
result = opt.solve(model)
print(result.solver.status, result.solver.termination_condition)
```
  - Solution: The optimal production quantities and binary variables for each flavor.
- Problem Two: Optimal Placement
  - Objective: Identify the optimal location for a roasting facility considering distances to fifteen existing shops.
  - Distance Metric: Dk = |xk - x| + |yk - y|
  - Data: Locations of the existing shops in terms of (x, y) coordinates.
  - Solution: The optimal (x, y) coordinates representing the new roasting facility location

## Topic 2

### Data Source:

### Exam Highlights: 
- 
