# Bulding a Forward-planning Agent

Second project from the Udacity's Artificial Intelligence Nanodegree building a Forward-Planning agent using AIMA planning library. 

Project sections:

- Problem understanding
- Project structure
- Running experiments
- Results report

## Problem understanding

Planning is an important topic in AI because intelligent agents are expected to automatically plan their own actions in uncertain domains. Planning and scheduling systems are commonly used in automation and logistics operations, robotics and self-driving cars, and for aerospace applications like the Hubble telescope and NASA Mars rovers. This project development combine symbolic logic and classical search to implement an agent that performs progression search to solve planning problems.


## Project structure

The project structure is based on the Udacity's project template:

```
+ aimacode: aima code library + logic.py
                              + planning.py
                              + search.py
                              + search.pyc
                              + utils.py          
+ tests                       + test_my_planning_graph.py:  Test cases for my_planning_graph implementation

+ _utils.py
+ air_cargo_problems.py:                                    Contains the cargo problme scenarios 
+ layers.py
+ my_panning_graph.py:                                      Planning implementation 
+ run_search.py:                                            Main script to run the simulation          

```

## Functions developed

###Action layer functions

#### function _inconsistent_effects
Return True if an effect of one action negates an effect of the other
        Hints:
            (1) `~Literal` can be used to logically negate a literal
            (2) `self.children` contains a map from actions to effects

          ```python
          def _inconsistent_effects(self, actionA, actionB):
              for effectA in actionA.effects:       
                  for effectB in actionB.effects:
                      if effectA == ~effectB:
                          return True
          ```

#### function _interference
Return True if the effects of either action negate the preconditions of the other 

        Hints:
            (1) `~Literal` can be used to logically negate a literal
            (2) `self.parents` contains a map from actions to preconditions

        ```
        for effectA in actionA.effects:
            if ~effectA in actionB.preconditions: return True
 
        for effectB in actionB.effects:
            if ~effectB in actionA.preconditions: return True
 
        return False
        ```

#### function _competing_needs

Return True if the preconditions of the actions are all pairwise mutex in the parent layer 

```python
def _competing_needs(self, actionA, actionB):
    
    for preconditionA in actionA.preconditions:

        for preconditionB in actionB.preconditions:

            if self.parent_layer.is_mutex(preconditionA, preconditionB):
                return True

```


### Literal layer

#### function _inconsistent_support
Return True if all ways to achieve both literals are pairwise mutex in the parent layer

        Hints:
            (1) `self.parent_layer` contains a reference to the previous action layer
            (2) `self.parents` contains a map from literals to actions in the parent layer

          ```
          for parentA in self.parents[literalA]:
                    for parentB in self.parents[literalB]:
                              if not(self.parent_layer.is_mutex(parentB,parentA)):  
                              return False
          return True
          ```


#### function _negation
Return True if two literals are negations of each other

          ```
          if literalA == ~literalB: return True
          else: return False
          ```


### Planning Graph heuristics implementations


#### function h_levelsum

Calculate the level sum heuristic for the planning graph

        The level sum is the sum of the level costs of all the goal literals combined. The "level cost" to achieve any single goal literal is the         level at which the literal first appears in the planning graph. Note that the level cost is **NOT** the minimum number of actions to achieve a single goal literal.
        
        For example, if Goal_1 first appears in level 0 of the graph (i.e., it is satisfied at the root of the planning graph) and Goal_2 first         appears in level 3, then the levelsum is 0 + 3 = 3.

        Hints
        -----
          (1) See the pseudocode folder for help on a simple implementation
          (2) You can implement this function more efficiently than the sample pseudocode if you expand the graph one level at a time and accumulate the level cost of each goal rather than filling the whole graph at the start.


          ```
          self.fill()
          cost_level_sum = 0
          
          for go in self.goal:
                    for cost, layer in enumerate(self.literal_layers):
                              if go in layer:
                                        cost_level_sum += cost
                              break
          return cost_level_sum
          ```

#### function h_maxlevel

Calculate the max level heuristic for the planning graph

        The max level is the largest level cost of any single goal fluent. The "level cost" to achieve any single goal literal is the level at which the literal first appears in the planning graph. Note that the level cost is **NOT** the minimum number of actions to achieve a single goal literal.

        For example, if Goal1 first appears in level 1 of the graph and Goal2 first appears in level 3, then the levelsum is max(1, 3) = 3.

        Hints
        -----
          (1) See the pseudocode folder for help on a simple implementation
          (2) You can implement this function more efficiently if you expand the graph one level at a time until the last goal is met rather than filling the whole graph at the start.

```
        self.fill()
        level_cost = 0
        for go in self.goal:
            for cost, layer in enumerate(self.literal_layers):
                if go in layer:
                    level_cost = max(cost, level_cost)
                    break
        return level_cost
```

#### function h_setlevel
 Calculate the set level heuristic for the planning graph

        The set level of a planning graph is the first level where all goals appear such that no pair of goal literals are mutex in the last layer of the planning graph.

        Hints
        -----
          (1) See the pseudocode folder for help on a simple implementation
          (2) You can implement this function more efficiently if you expand the graph one level at a time until you find the set level rather than filling the whole graph at the start.

          ```
               _graph = self
                  currentLevel = 0
                  _goals = [goal for goal in _graph.goal]
                  setLevelFound = False

                  while not(setLevelFound):
                      concurrentGoals = True
                      mutexGoals = False

                      _level = _graph.literal_layers[currentLevel]

                      for goal in _graph.goal:
                          if goal not in _level: 
                              concurrentGoals = False
                              break

                      if concurrentGoals:
                          for goalA in _graph.goal:
                              for goalB in _graph.goal:
                                  if _level.is_mutex(goalA, goalB): 
                                      mutexGoals = True
                                      break                          
                              if mutexGoals: break

                      if concurrentGoals and not(mutexGoals):
                          setLevelFound = True

                      if not(concurrentGoals) or mutexGoals:
                          _graph._extend()
                          currentLevel += 1

                  return currentLevel
          ```

## Running the experiments

To run the experiments run the `run_search.py` script. The script can be executed manually or in batch mode:

  - Run the search experiment manually (you will be prompted to select problems & search algorithms)
```
$ python run_search.py -m
```

  - You can also run specific problems & search algorithms - e.g., to run breadth first search and UCS on problems 1 and 2:
```
$ python run_search.py -p 1 2 -s 1 2
```

  - Running all the experiments
```
$ python run_search.py -p 1 2 3 4 -s 1 2 3 4 5 6 7 8 9 10 11
```

## Results report








## Author 
Fernando Bonilla [linkedin](https://www.linkedin.com/in/fer-bonilla/)
