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
- Open-souce software [Pyomo](http://www.pyomo.org/about) [SciPy](https://docs.scipy.org/doc/scipy/tutorial/general.html)
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
Create a team based on the actual data and following constrainsts
- The fantasy team must have nine players.
- The salary of your nine players may not exceed $50,000.
- You need players from at least three different teams.
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

## PLayer Util Constraint
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
report.columns = ['Name', 'P', 'Team', 'Position', 'Salary']
```
|  #  |         Name         |  P  | Team | Position | Salary |
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
The optimal teams' average points per game is 18.00.
