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

## Pyomo Optimal Quantity and Placement Exam Grade: 100%

### Data Source: Provided by Professor [Excel File](https://github.com/EvanElzenberger/OptimizationModeling/files/11349653/PyomoExam2.xlsx)

### Exam Highlights:
#### Problem One: Flavors
- Objective: Maximize profit by determining optimal production quantities for five flavors (Apple, Banana, Chocolate, Elderberry, and Fig).
- Considerations: Pricing, costs, fixed setup costs, mixing, preparation, blending, and packaging constraints.
- Demand Constraints: Ensuring the production quantities satisfy the demand for each flavor.
- Solution: The optimal production quantities and binary variables for each flavor.
Part A. Determine the optimal quantities of each flavor to maximize profit.
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
model.LinkA = pe.Constraint(expr=model.x['Apple'] <= demand.loc['Apple', 'demand'] * model.y['Apple'])
model.LinkB = pe.Constraint(expr=model.x['Banana'] <= demand.loc['Banana', 'demand'] * model.y['Banana'])
model.LinkC = pe.Constraint(expr=model.x['Chocolate'] <= demand.loc['Chocolate', 'demand'] * model.y['Chocolate'])
model.LinkE = pe.Constraint(expr=model.x['Elderberry'] <= demand.loc['Elderberry', 'demand'] * model.y['Elderberry'])
model.LinkF = pe.Constraint(expr=model.x['Fig'] <= demand.loc['Fig', 'demand'] * model.y['Fig'])
opt = pe.SolverFactory('glpk')
result = opt.solve(model)
obj_val = model.obj.expr()
print(f'The optimal objective value for maximum profit = ${obj_val:.2f}')
dv_keys = list(model.x.keys())
solution = pd.DataFrame()
for DV in model.component_objects(pe.Var):
    for var in DV:
        solution.loc[DV.name, var] = DV[var].value
solution
```
- The optimal objective value for maximum profit = $1217.20

|       | Apple | Banana | Chocolate | Elderberry |  Fig  |
|-------|-------|--------|-----------|------------|-------|
|   x   | 200.0 | 150.0  |    0.0    |    0.0     | 200.0 |
|   y   |  1.0  |  1.0   |    0.0    |    0.0     |  1.0  |

Part B. Modify the model to ensure that at least one of Chocolate or Elderberry is produced.
```python 
## Initialize Model
model2 = pe.ConcreteModel()

        ## Define Decision Variables
model2.x = pe.Var(DV_indexes, domain = pe.NonNegativeReals)
model2.y = pe.Var(DV_indexes, domain = pe.Binary)
# Define Objective function
model2.obj = pe.Objective(expr=sum([cost.loc['Price', i]*model2.x[i] for i in DV_indexes])
                             - sum([cost.loc['Cost',i]*model2.x[i] for i in DV_indexes])
                             - sum([cost.loc['Fixed Setup Cost',i]*model2.y[i] for i in DV_indexes]),
                            sense=pe.maximize)
# Define Constraints
model2.mixing = pe.Constraint(expr = sum([coef.loc['Mixing',i]*model2.x[i]for i in DV_indexes]) 
                                        <= rhs.loc['Mixing', 'rhs'])
model2.preperation = pe.Constraint(expr = sum([coef.loc['Preparation',i]*model2.x[i]for i in DV_indexes]) 
                                             <= rhs.loc['Preparation', 'rhs'])
model2.blending = pe.Constraint(expr = sum([coef.loc['Blending',i]*model2.x[i]for i in DV_indexes]) 
                                          <= rhs.loc['Blending', 'rhs'])
model2.packaging = pe.Constraint(expr = sum([coef.loc['Packaging',i]*model2.x[i]for i in DV_indexes]) 
                                           <= rhs.loc['Packaging', 'rhs'])
#Chocolate or Elderberry
model2.ChocEld = pe.Constraint(expr = model2.x['Chocolate']+model2.x['Elderberry'] >= 1)
#Demand Constraints
model2.LinkA = pe.Constraint(expr=model2.x['Apple'] <= demand.loc['Apple', 'demand'] * model2.y['Apple'])
model2.LinkB = pe.Constraint(expr=model2.x['Banana'] <= demand.loc['Banana', 'demand'] * model2.y['Banana'])
model2.LinkC = pe.Constraint(expr=model2.x['Chocolate'] <= demand.loc['Chocolate', 'demand'] * model2.y['Chocolate'])
model2.LinkE = pe.Constraint(expr=model2.x['Elderberry'] <= demand.loc['Elderberry', 'demand'] * model2.y['Elderberry'])
model2.LinkF = pe.Constraint(expr=model2.x['Fig'] <= demand.loc['Fig', 'demand'] * model2.y['Fig'])
# Solve
opt2 = pe.SolverFactory('glpk')      
result2 = opt2.solve(model2)
obj_val2 = model2.obj.expr()
print(f'The optimal objective value for maximum profit = ${obj_val2:.2f}')
DV_solution = pd.DataFrame()
for DV in model2.component_objects(pe.Var):
    for var in DV:
        DV_solution.loc[DV.name, var] = DV[var].value
DV_solution
```
- The optimal objective value for maximum profit = $1212.05

|       | Apple | Banana | Chocolate | Elderberry |  Fig  |
|-------|-------|--------|-----------|------------|-------|
|   x   | 200.0 | 149.5  |    0.0    |    1.0     | 200.0 |
|   y   |  1.0  |  1.0   |    0.0    |    1.0     |  1.0  |

#### Problem Two: Optimal Placement
 - Objective: Identify the optimal location for a roasting facility considering distances to fifteen existing shops.
 - Distance Metric: Dk = |xk - x| + |yk - y|
 - Data: Locations of the existing shops in terms of (x, y) coordinates.
 - Solution: The optimal (x, y) coordinates representing the new roasting facility location
Part A: Use a vizualization package to plot the locations of the coffee shops based on the provided data.
```python


plt.figure(figsize=(8,5))
sns.scatterplot(x = 'x', y = 'y', data = dfloc)
plt.grid()
plt.ylabel('Location Y')
plt.title('Location Y by Location X')
plt.xlabel('Location X')
plt.show()
```
Part B: Identify the optimal location for the roasting facility by minimizing the total distance to each shop.
```python
# Create the Decision Variables and the Objective Function. 
model = pe.ConcreteModel()

# Decision variables
dfindexes = ['x', 'y']
model.x = pe.Var(dfindexes, domain = pe.NonNegativeReals)
                 
# Objective function 
model.obj = pe.Objective(expr=sum([abs(model.x['x'] -dfloc.loc[index, 'x']) for index in dfloc.index]
                                  + [abs(model.x['y'] -dfloc.loc[index, 'y']) for index in dfloc.index]),
                        sense = pe.minimize)
# Solve the model
opt = pe.SolverFactory('ipopt')
result = opt.solve(model)
obj_val = model.obj.expr()
print(f'The optimal objective value for maximum profit = ${obj_val:.2f}')
print()
solution = pd.DataFrame()
for DV in model.component_objects(pe.Var):
    for c in DV:
        solution.loc[DV.name, c] = round(DV[c].value, 1)
solution
```
- The optimal objective value for maximum profit = $665.00

|   x  |   y  |
|------|------|
| 43.4 | 59.0 |

Part C: Plot the locations with the selected location marked as a red asterisk.
```python
.figure(figsize = (8,5))
sns.scatterplot(x = 'x', y = 'y', data = dfloc)
plt.xticks(np.arange(0, 110, 10))
plt.yticks(np.arange(0, 110, 10))
plt.scatter(x = 43.4, y = 59.0, marker = '*', color = 'red')
plt.grid()
plt.ylabel('Location Y')
plt.title('Location Y by Location X')
plt.xlabel('Location X')
plt.show()
```
