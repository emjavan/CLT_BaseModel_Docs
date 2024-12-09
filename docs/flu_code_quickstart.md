# Flu Code Quickstart

> **_Written by LP, updated 11/26/2024 (work in progress)_**
> Note: figures below were generated with older flu model version without presymptomatic and asymptomatic infected compartments -- figures generated with updated code will look slightly different. 

Here we provide a quickstart tutorial to help users immediately start playing with the flu model in ```flu_components.py``` with "toy" inputs given by ```JSON``` files in ```flu_demo_input_files```. Recall that the *structure* (functional form) of the flu model is specified in ```flu_components.py```, and the input files simply specify fixed parameter values and initial values for state variables.  See [this page](math_flu_components.md) for the mathematical formulation of the flu model. For creating their own custom input files, users should refer to tables on [this page](flu_input_files.md) that map code variable names to their respective math variables and specify variable dimensions. 

The code snippet below is from ```flu_demo.py```. The script can be run directly from the terminal using 

```bash
python flu_demo.py
```

Here’s the basic procedure to populate the flu model using input files:

1. Create a ```FluModelConstructor``` instance that takes three file paths as input: the configuration file path, fixed parameter values file path, and initial values file path (all in ```JSON``` format).

2. Use this constructor to create a ```TransmissionModel``` instance whose functional form is specified by ```flu_components.py``` and whose fixed parameter values and initial values are specified in the three aforementioend files.

After this, we can simulate the model and plot its behavior. Below, we simulate 300 days. 

```python
import numpy as np
from pathlib import Path

from flu_components import FluModelConstructor
from plotting import create_basic_compartment_history_plot

# Obtain path to folder with JSON input files
base_path = Path(__file__).parent / "flu_demo_input_files"

# Get filepaths for configuration, fixed parameter values, and
#   initial values of state variables
config_filepath = base_path / "config.json"
fixed_params_filepath = base_path / "fixed_params.json"
state_vars_init_vals_filepath = base_path / "state_variables_init_vals.json"

# Create a constructor using these filepaths
flu_demo_constructor = \
    FluModelConstructor(config_filepath,
                        fixed_params_filepath,
                        state_vars_init_vals_filepath)

# Create TransmissionModel instance from the constructor,
#   using a random number generator with starting seed 888888
#   to generate random variables
flu_demo_model = flu_demo_constructor.create_transmission_model(888888)

# Simulate 300 days
flu_demo_model.simulate_until_time_period(300)

# Plot
create_basic_compartment_history_plot(flu_demo_model,
                                      "flu_demo_model.png")
```

![title](figs/flu_demo_model.png)

The following code snippets and outputs below replicate the experience of working in a Python interactive console.

After simulating 300 days, the ```current_simulation_day``` counter indeed is 300, and the attribute ```current_real_date``` gives us the corresponding real-world date associated with the simulation day counter.

```python
flu_demo_model.current_simulation_day
# 300

flu_demo_model.current_real_date
# datetime.date(2023, 6, 4)
```

Note that subsequent ```simulate_until_time_period``` calls will start from where the simulation last ended, and its argument ```last_simulation_day``` must be greater than the model's ```current_simulation_day```. For example, after running ```flu_demo_model.simulate_until_time_period(300)```, a command like ```flu_demo_model.simulate_until_time_period(100)``` is invalid (we cannot simulate backwards in time), but ```flu_demo_model.simulate_until_timeperiod(305)``` starts from day 300 and continues.

```TransmissionModel``` instances have a ```StateVariableManager``` that manages and holds StateVariables (EpiCompartments, EpiMetrics, DynamicVals, and Schedules). Using the command below, we can access the state of the simulation after we simulated the model for 300 days. 

```python
flu_demo_model.sim_state
# FluSimState(S=array([[306539.],
#       [261363.]]), E=array([[31501.],
#       [36027.]]), I=array([[105991.],
#       [114542.]]), H=array([[ 98354.],
#       [104663.]]), R=array([[175885.],
#       [186626.]]), D=array([[281730.],
#       [296779.]]), population_immunity_hosp=array([[0.11555325],
#       [0.12078151]]), population_immunity_inf=array([[0.11555325],
#       [0.12078151]]), absolute_humidity=12.31748, 
#		flu_contact_matrix=array([[[[2.], [0.5]]], [[[1.95], [1.4]]]]))
```

We can also use our model's built-in dictionary to look up simulation objects (any StateVariable, as well as any TransitionVariable and TransitionVariableGroup) by name. Here, ```S``` is an ```EpiCompartment``` instance, and we can see its current value at the current day of the simulation. 

```python
flu_demo_model.lookup_by_name["S"]
# <base_components.EpiCompartment object at 0x174a79750>

flu_demo_model.lookup_by_name["S"].current_val
# array([[306539.],
#        [261363.]])
```

We can also access a StateVariable's history with the attribute ```history_vals_list```. For example, ```flu_demo_model.lookup_by_name["S"].history_vals_list``` gives a list of all the previous current values of the "Susceptible" compartment. The $i$th element in the list holds the compartment's value at the end of simulation day $i$. 

Next, we look at the current value (most recent realization) of the TransitionVariable ```new_dead.``` We also look at its ```current_rate``` attribute, which is the rate that generated the most recent realization. 

```python
flu_demo_model.lookup_by_name["new_dead"].current_val
# array([[406.],
#        [435.]])

flu_demo_model.lookup_by_name["new_dead"].current_rate
# array([[0.00823744],
#        [0.00820603]])
```

We can reset the simulation to restart the simulation or clear the simulation's state and history to run a new replication on the same model, with the same initial conditions. *Important note: resetting the simulation does NOT reset the random number generator -- random numbers will continue where the generator last left off.*
```python
flu_demo_model.reset_simulation()
```

Notice that the current simulation day has returned to 0 and the simulation state has returned to its initial state. Each StateVariable's history has also been cleared.  
```python
flu_demo_model.current_simulation_day
# 0

flu_demo_model.sim_state
# FluSimState(S=array([[980000.],
#       [980000.]]), E=array([[10000.],
#       [10000.]]), I=array([[10000.],
#       [10000.]]), H=array([[0.],
#       [0.]]), R=array([[0.],
#       [0.]]), D=array([[0.],
#       [0.]]), population_immunity_hosp=array([[0.5],
#       [0.5]]), population_immunity_inf=array([[0.5],
#       [0.5]]), absolute_humidity=None, flu_contact_matrix=None)

flu_demo_model.lookup_by_name["S"].history_vals_list
# []
```

To reset the random number generator, we must use the ```modify_random_seed``` method -- and pass the initial random seed. This resets the numpy RNG object to a state given by this seed. We can also use the ```modify_random_seed``` method to handle random number generation more broadly, and ensure that each simulation replication uses independent random numbers. To handle random number generation responsibly, refer to these two links [here](https://numpy.org/doc/2.0/reference/random/bit_generators/index.html) and [here](https://blog.scientific-python.org/numpy/numpy-rng/).

Suppose that we want to modify configuration values, values of fixed parameters, or initial values. We can modify these values on our model constructor, and then create a new model instance. For example, below we change the transition type to "binomial_deterministic" and the beta baseline value to be $0$. By changing ```beta_baseline``` to $0$, no transmission occurs, and the plot verifies this. 

```python
flu_demo_constructor.config.transition_type = "binomial_deterministic"
flu_demo_constructor.fixed_params.beta_baseline = 0

flu_demo_model_beta_baseline_zero = \
    flu_demo_constructor.create_transmission_model(999999)

create_basic_compartment_history_plot(flu_demo_model_beta_baseline_zero,
                                      "flu_demo_model_beta_baseline_zero.png")
```

![flu_demo_model_beta_baseline_zero](figs/flu_demo_model_beta_baseline_zero.png)