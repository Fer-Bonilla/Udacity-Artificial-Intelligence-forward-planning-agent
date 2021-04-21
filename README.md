# Udacity-Artificial-Intelligence-forward-planning-agent

Second project from the Udacity's Artificial Intelligence Nanodegree building a Forward-Planning agent using AIMA planning library. 

Project sections:

- Problem understanding
- Data description
- Database Model
- Project structure
- ETL Pipeline description
- Instructions to run the pipeline

## Problem understanding

Planning is an important topic in AI because intelligent agents are expected to automatically plan their own actions in uncertain domains. Planning and scheduling systems are commonly used in automation and logistics operations, robotics and self-driving cars, and for aerospace applications like the Hubble telescope and NASA Mars rovers. This project development combine symbolic logic and classical search to implement an agent that performs progression search to solve planning problems.


## Project structure

The project structure is based on the Udacity's project template:

```
+ airflow + dags
          + plugings  + helpers   + sql_queries.py: Insert sql stament definitions
                      + operators + data_quality.py: This operator implements the data quality verification task, based on the BaseOperator
                                  + load_dimension.py: This operator implements the LoadDimensionOperator class that execute the data load process from staging tables to dimension tables.
                                  + load_fact.py: This operator implements the LoadFactOperator class that execute the data load process from staging tables to fact table.
                                  + stage_redshift.py: This operator implements the data quality verification task, based on the BaseOperator
          + create_tables.sql : drops and creates your tables. You run this file to reset your tables before each time you run your ETL scripts.

```

## Functions developed

## Inconsistent Effects
Return True if an effect of one action negates an effect of the other

```python
def _inconsistent_effects(self, actionA, actionB):

    for effectA in actionA.effects:
        
        for effectB in actionB.effects:
            
            if effectA == ~effectB:
                return True
```

## Interference
Return True if the effects of either action negate the preconditions of the other 

```python
def _interference(self, actionA, actionB):
    
    for effect in actionA.effects:
        
        for precondition in actionB.preconditions:
            
            if precondition == ~effect:
                return True
```

## Competing Needs

Return True if the preconditions of the actions are all pairwise mutex in the parent layer 

```python
def _competing_needs(self, actionA, actionB):
    
    for preconditionA in actionA.preconditions:

        for preconditionB in actionB.preconditions:

            if self.parent_layer.is_mutex(preconditionA, preconditionB):
                return True

```


## Inconsistent Support
Return True if all ways to achieve both literals are pairwise mutex in the parent layer

```python
def _inconsistent_support(self, literalA, literalB):
    
    for actionA in self.parents[literalA]:

        for actionB in self.parents[literalB]:

            if not self.parent_layer.is_mutex(actionA, actionB):
                return False

    return True
```
## Negation
Return True if two literals are negations of each other.

```python
def _negation(self, literalA, literalB):
    
    if literalA == ~literalB and literalB == ~literalA:
        return True
```


# Heuristics 

## Level Sum

Calculate the level sum heuristic for the planning graph

The level sum is the sum of the level costs of all the goal literals
combined. The "level cost" to achieve any single goal literal is the
level at which the literal first appears in the planning graph. Note
that the level cost is **NOT** the minimum number of actions to
achieve a single goal literal.

For example, if Goal1 first appears in level 0 of the graph (i.e.,
it is satisfied at the root of the planning graph) and Goal2 first
appears in level 3, then the levelsum is 0 + 3 = 3.


```python
def h_levelsum(self):

    graph = self.fill()

    levelsum = 0

    for goal in self.goal:
        levelsum = levelsum + self.levelcost(graph, goal)

    return levelsum
```

## Max Level

Calculate the max level heuristic for the planning graph

The max level is the largest level cost of any single goal fluent.
The "level cost" to achieve any single goal literal is the level at
which the literal first appears in the planning graph. Note that
the level cost is **NOT** the minimum number of actions to achieve
a single goal literal.

For example, if Goal1 first appears in level 1 of the graph and
Goal2 first appears in level 3, then the levelsum is max(1, 3) = 3.

```python
def h_maxlevel(self):

    graph = self.fill()

    costs = []

    for goal in self.goal:
        costs.append(self.levelcost(graph, goal))

    return max(costs)
```

## Set Level
Calculate the set level heuristic for the planning graph

The set level of a planning graph is the first level where all goals
appear such that no pair of goal literals are mutex in the last
layer of the planning graph.

```python
def h_setlevel(self):

    while not self._is_leveled:
        
        layer = self.literal_layers[-1]

        if self.goal.issubset(layer):
            
            no_pairmutex = True

            for goal1 in self.goal:
                for goal2 in self.goal:
                    if layer.is_mutex(goal1, goal2):
                        no_pairmutex = False
                        break

            if no_pairmutex:
                return len(self.literal_layers) - 1

        self._extend()

    return len(self.literal_layers) - 1   
```

### ETL pipeline diagram

![ETL pipeline diagram](https://github.com/Fer-Bonilla/Udacity-Data-Engineering-data-pipelines-with-airflow/blob/main/images/airflow_pipeline.png)



## Running the experiments

Experiment with different search algorithms using the `run_search.py` script. (See example usage below.) The goal of your experiment is to understand the tradeoffs in speed, optimality, and complexity of progression search as problem size increases. You will record your results in a report (described below in [Report Requirements](#report-requirements)).

  - Run the search experiment manually (you will be prompted to select problems & search algorithms)
```
$ python run_search.py -m
```

  - You can also run specific problems & search algorithms - e.g., to run breadth first search and UCS on problems 1 and 2:
```
$ python run_search.py -p 1 2 -s 1 2
```

## Results report








## Author 
Fernando Bonilla [linkedin](https://www.linkedin.com/in/fer-bonilla/)
