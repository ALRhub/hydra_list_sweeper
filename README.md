# Hydra example Launcher plugin

Sweeper plugin for Hydra wich creates a list option additionally to the cartesian product ("grid").
This allows to test only a subset of the cartesian product. It uses the a similar syntax as the standard sweeper:
```yaml
hydra:
  mode: MULTIRUN
  sweeper:
    # standard grid search
    grid_params:
      env: 5_clubs_juggling, balancing_stick
    # additional list sweeper
    list_params:
      algorithm.lr: 0.001, 0.0001
      algorithm.beta_1: 0.9, 0.99
```
This configuration will create 4 jobs:
```text
env=5_clubs_juggling, algorithm.lr=0.001, algorithm.beta_1=0.9
env=5_clubs_juggling, algorithm.lr=0.0001, algorithm.beta_1=0.99
env=balancing_stick, algorithm.lr=0.001, algorithm.beta_1=0.9
env=balancing_stick, algorithm.lr=0.0001, algorithm.beta_1=0.99
``` 

Basically, it grids over all grid params, creating the standard cartesian product, 
and then for each of these combinations, it creates a job for each of the list params.
 