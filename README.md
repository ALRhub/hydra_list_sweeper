# List Sweeper plugin for Hydra

Sweeper plugin for Hydra which creates a list option additionally to the cartesian product ("grid"), 
which allows to sweep over the zipped list of parameters. 
This allows to test only a subset of the cartesian product and it is useful for small hyperparameter searches.

## Installation
```bash
pip install hydra-list-sweeper
```
This will install the plugin in your current environment.

You can check if the plugin is installed by adding `--info plugins` to your command line.
The plugin should be listed in the output as `hydra_plugins.list_sweeper_plugin.list_sweeper`.

In order to enable this plugin, you need to override the default sweeper in your configuration file:
```yaml
defaults:
  - _self_
  # override this to use the new list sweeper:
  - override hydra/sweeper: list
```

## Usage

`List sweeper`  uses the a similar syntax as the standard sweeper, but instead of a `params` key, it uses `grid_params` and `list_params`:
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
      algorithm.beta_1: [0.9, 0.99]  # both notations work
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
You can additionally overwrite single values with command line arguments, and even define your grid_params in the command line:
```yaml
hydra:
  mode: MULTIRUN
  sweeper:
    # additional list sweeper
    list_params:
      algorithm.lr: 0.001, 0.0001
      algorithm.beta_1: [0.9, 0.99]  # both notations work
```
Combined with this command
```bash
python my_app.py env=5_clubs_juggling,balancing_stick
```
 will produce the same results as the first example. Also, you can override configs with the command line and the grid_params:

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
      algorithm.beta_1: [0.9, 0.99]  # both notations work
```
 Combined with this command:
```bash
python my_app.py algorithm.epsilon=1.0e-4
```

will produce the same results as the first example, but epsilon will be set to 1.0e-4 for all jobs.

If you remove the `list_params` section, it will behave exactly as the standard grid sweeper (at least it should do, if you find a bug, please report it).

# Ablative params
You can additionaly define a `ablative_params` section. This must be a list of dictionaries. For example
```yaml
    ablative_params:
      - algorithm.beta_2: 0.5
        algorithm.epsilon: 1e-4
      - algorithm.beta_1: 0.3
```
If the `ablative_params` are present, it will 
1. sweep over all the jobs generated by list and grid ignoring the ablative params
2. For each dictionary in `ablative_params`, it will replace or add the key-value pairs in the dictionary to all the jobs generated in step 1.

In the example above, it will generate 4 jobs from the list and grid params. Since 2 ablative dictionaries are present, it will in total generate $4 * (1 +1+ 1)$ jobs.
4 jobs from the list and grid params, and 4 jobs per dictionary in the `ablative_params` section.

## Limitations
In the `ablative_parmas` section, you can only specify concrete parameters, changing a complete sub-config is not implemented currently.
