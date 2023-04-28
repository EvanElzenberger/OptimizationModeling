# Optimization Modeling

## Course Description:
This course explores modeling and solving business decision problems. It covers optimization models including linear programming, integer programming, transportation, assignment, network optimization, and nonlinear programming. Students learn to analyze scenarios, formulate models using software tools, interpret output, and present project reports.

## Skills Learned:
- Identify problems appropriate for optimization.
- Understand the challenges with solution techniques with optimization problems.
- Learn to convert written descriptions into optimization problems.
- Learn to solve optimization problems using open source software.

## Software:
- Microsoft Excel Solver 
- Open-souce software [Pyomo](http://www.pyomo.org/about), [SciPy](https://docs.scipy.org/doc/scipy/tutorial/general.html)
- Python
  - Pandas: Used for data import, manipulation and analysis.
  - Pyomo: Formulating and solving optimization models.
  - Matplotlib: Creating plots and visualizations.
  - Seaborn: Generating scatterplots for location visualization.
  - Numpy: Performing numerical computations and array operations.
  - Matplotlib: Creating visualizations, such as graphs and charts, to analyze and present data.
  - Scipy: Optimization algorithms and other scientific computing tools.

## Optimal NHL Team for Draftkings Exam Grade: 100%
The project focuses on solving an optimization problem in the context of Daily Fantasy Sports on DraftKings, specifically for the National Hockey League (NHL) contests. The objective is to select an optimal lineup of players based on certain constraints and rules. The project utilizes Python and optimization models to solve the problem. It involves exploratory data analysis using historical data and applies optimization techniques to select lineups. Two optimization problems are solved: one based on historical information and the other based on actual game data.

### Data Source:  [TBA](TBA)

### Exam Highlights:
Create the optimal team based on the actual data and following constrainsts that generates to maximum amount of points
- The fantasy team must have nine players.
- The salary of your nine players may not exceed $50,000.
- The makeup of the team in terms of positions is:
  - Two Centers (C)
  - Three Wings (W)
  - Two Defenders (D)
  - One Utility (C, W, or D)
  - One Goalie

```python
## Initialize Model
model = pe.ConcreteModel()

##Define Decision Variables
model.x = pe.Var(PSActual.index, domain = pe.Binary)

## Define Objective Function
model.obj = pe.Objective(expr = sum([PSActual.p.loc[i] * model.x[i] for i in PSActual.index]),
                                 sense = pe.maximize)
                                 
## Salary Constraint 
model.salary = pe.Constraint(expr = sum([PSActual.Salary.loc[i] * model.x[i] for i in PSActual.index]) <= 50000)

## Player Util Constraint
model.LinkF = pe.Constraint(expr=sum([model.x[i] for i in PSActual.index]) ==9)

## Players Constraints
model.C = pe.Constraint(expr=sum([PSActual.C.loc[i]*model.x[i] for i in PSActual.index]) >=2)
model.W = pe.Constraint(expr=sum([PSActual.W.loc[i]*model.x[i] for i in PSActual.index]) >=3)
model.D = pe.Constraint(expr=sum([PSActual.D.loc[i]*model.x[i] for i in PSActual.index]) >=2)
model.G = pe.Constraint(expr=sum([PSActual.G.loc[i]*model.x[i] for i in PSActual.index]) ==1)

##Solve
opt = pe.SolverFactory('glpk')      
result = opt.solve(model)
obj_val = model.obj.expr()

## Create a report of the selected team
team = []
for DV in model.component_objects(pe.Var):     
    for i in DV:
        if DV[i].value == 1:
            team.append([i, PSActual.p[i], PSActual.TeamAbbrev[i], PSActual.Roster_Position[i], PSActual.Salary[i]])
report = pd.DataFrame(team)
report.columns = ['Name', 'Points', 'Team', 'Position', 'Salary']
```
|  Count  |         Name         |  Points | Team | Position | Salary |
|:---:|:--------------------|:---:|:----:|:--------:|-------:|
|  1  |  Michael McLeod      |  2  |  NJ  | C/UTIL   |  2500  |
|  2  |  Kyle Connor         |  2  | WPG  | W/UTIL   |  8600  |
|  3  |  William Karlsson    |  2  | VGK  | C/UTIL   |  4500  |
|  4  |  Patrik Laine        |  3  | CLS  | W/UTIL   |  4800  |
|  5  |  Elvis Merzlikins    |  0  | CLS  | G        |  7100  |
|  6  |  Adam Boqvist        |  2  | CLS  | D/UTIL   |  4000  |
|  7  |  Nic Dowd            |  2  | WAS  | C/UTIL   |  2500  |
|  8  |  Jake Guentzel       |  2  | PIT  | W/UTIL   |  7500  |
|  9  |  Oliver Ekman-Larsson|  3  | VAN  | D/UTIL   |  3100  |

Report the total salary spent and points per game generated
```python
obj_val = model.obj.expr()
print(f'The total salary used is ${sum(report.Salary)}.')
print(f'The optimal team average points per game is {obj_val:.2f}.')
```
The total salary used is $44600.

The optimal teams' total points per game is 18.00.


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
#Chocolate or Elderberry
model.ChocEld = pe.Constraint(expr = model.x['Chocolate']+model.x['Elderberry'] >= 1)
#Demand Constraints
model.LinkA = pe.Constraint(expr=model.x['Apple'] <= demand.loc['Apple', 'demand'] * model.y['Apple'])
model.LinkB = pe.Constraint(expr=model.x['Banana'] <= demand.loc['Banana', 'demand'] * model.y['Banana'])
model.LinkC = pe.Constraint(expr=model.x['Chocolate'] <= demand.loc['Chocolate', 'demand'] * model.y['Chocolate'])
model.LinkE = pe.Constraint(expr=model.x['Elderberry'] <= demand.loc['Elderberry', 'demand'] * model.y['Elderberry'])
model.LinkF = pe.Constraint(expr=model.x['Fig'] <= demand.loc['Fig', 'demand'] * model.y['Fig'])
# Solve
opt = pe.SolverFactory('glpk')      
result = opt.solve(model)
obj_val = model.obj.expr()
print(f'The optimal objective value for maximum profit = ${obj_val:.2f}')
DV_solution = pd.DataFrame()
for DV in model.component_objects(pe.Var):
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
