# Flu Model Creation Walkthrough

> **_Written by LP, updated 12/05/2024_** 

In this section, we provide an explanation of how the flu model (in `flu_components.py`) is created from the base model components (in `base_components.py`). In doing so, we also demonstrate how to create other customized models from the base model components. We also provide justification for design choices, in a way intended to educate users on some of the implementation and computation challenges in writing scalable and adaptable compartmental models. 

Writing code to create a customized compartmental model requires correct use of inheritance, composition, and abstract base classes. We *strongly* recommend reading [this tutorial](https://realpython.com/inheritance-composition-python/) for a good primer on those subjects. We also recommend learning about the `dataclass` decorator in Python (see [this blogpost](https://www.dataquest.io/blog/how-to-use-python-data-classes/), for example).

## Creating a model from base components: details

Creating a compartment model requires the following steps.

0. Formulate the compartment model mathematically, and write down all `EpiCompartment`, `TransitionVariable`, `TransitionVariableGroup` (jointly sampled sets of transition variables), and `StateVariable` objects needed for the model. 

1. Create a new file that imports the `base_components` module. We import this module as `base`. 

2. Create two `JSON` files: one for fixed parameters and one for initial values of state variables. 

	a. For the fixed parameters `JSON` file, required fields are `"num_age_groups"`, `"num_risk_groups"`, and `"total_population_val"`. All other fields must correspond to fixed parameters in the model -- these are used throughout model computation.

	- See `flu_demo_input_files / fixed_params.json`.

	b. For the initial values of state variables `JSON` file, there must be a field for each `StateVariable` object used in the model. These values correspond to the initial values of these state variables. For example, this file contains initial populations of each `EpiCompartment`, expressed as a list of lists corresponding to age groups and risk groups. Similarly, this file contains starting values for each `EpiMetric`. We also need fields for all `DynamicVal` and `Schedule` objects in the model, although their values can be set to `null` since their initial state will be updated automatically (either according to a deterministic calendar or a simulate state trigger). We just need to include their names for bookkeeping purposes. 

	- See `flu_demo_input_files / state_variables_init_vals.json`.

3. Create subclasses to hold data. 

	a. Create a subclass of `base.FixedParams` to hold the model's fixed parameters. For each name in the fixed parameters `JSON` file, there should be a corresponding attribute (with the same name) in this subclass.
	
	- See `FluFixedParams` in `flu_components.py`.

	b. Create a subclass of `base.SimState` to hold the model's simulation state. For each name in the initial values of state variables `JSON` file, there should be a corresponding attribute (with the same name) in this subclass. 

	- See `FluSimState` in `flu_components.py`.

4. For each transition in the model, create a subclass of `base.TransitionVariable` and provide a concrete implementation of the abstract method `get_current_rate`. The function `get_current_rate` *returns* the value of the current rate, and is a function of a `FluSimState` instance and a `FluFixedParams` instance. These arguments allow us to access various values needed for the computation. 

	- See `SusceptibleToExposed` subclass in `flu_components.py`. The function `get_current_rate` implements the formula for computing the current rate that people transition from the susceptible to exposed compartment. For example, `get_current_rate` grabs `sim_state.IS`, `sim_state.IP`, and `sim_state.IA`, because the current values of the symptomatic, pre-symptomatic, and asymptomatic infectious compartments are needed for the rate computation.

5. For each dynamic value in the model, create a subclass of `base.DynamicVal` and provide a concrete implementation of the abstract method `update_current_val`. The function `update_current_val` also takes a `FluSimState` instances and `FluFixedParams` instance as arguments, analogously to `TransitionVariable` subclasses. It updates the `current_val` attribute *in-place.*

	- See `BetaReduct` in `flu_components.py`. Note here we have also customized the `_init_` method and added an attribute `permanent_lockdown`, to accommodate additional functionality. 

6. For each schedule in the model, create a subclass of `base.Schedule` and provide a concrete implementation of the abstract method `update_current_val`, which should be a function of the current date (a `datetime.date` instance). This function updates the `current_val` attribute *in place*.  

	- See `AbsoluteHumidity` in `flu_components.py`. Note we update the current value using the helper function `absolute_humidity_func`.   

7. Create a subclass of `base.ModelConstructor`. Customize the `_init_` function and provide concrete implementations for the six abstract methods (listed below). 

	- See `FluModelConstructor` in `flu_components.py`.

	a. Customize the `_init_` function.

	- See `_init_` in `FluModelConstructor`. This method first calls `super()._init_()`, which runs the initialization method of the parent class. Importantly, the parent class initialization creates dictionaries that hold our objects (for example, it creates the attribute `compartment_lookup`). Then we assign the `config`, `fixed_params`, and `sim_state` attributes according to the contents of user-specified `JSON` files.

	b. Provide a concrete implementation of `setup_epi_compartments`. Specifically, for every compartment, create an entry in `compartment_lookup`, with the name as the key and the value as the corresponding `EpiCompartment` instance. Names should match the compartment names in the initial values of state variables `JSON`.

	c. Provide a concrete implementation of `setup_dynamic_vals`. Specifically, for every dynamic value, create an entry in `dynamic_val_lookup`, with the name as the key and the value as the corresponding `DynamicVal` instance. Names should match the dynamic value names in the initial values of state variables `JSON`.

	- In `FluModelConstructor`, we create an entry in `dynamic_val_lookup` with the key of `"beta_reduct"` and the value of a properly initialized `BetaReduct` instance. Note that `BetaReduct` *is* a `DynamicVal`, but we cannot use `DynamicVal` directly, as it is an abstract class. We have to use the relevant concretely implemented subclass.

	d. Provide a concrete implementation of `setup_schedules` (instructions are analogous to those for `setup_dynamic_vals`).

	e. Provide a concrete implementation of `setup_transition_variables` (instructions are analogous to those for `setup_dynamic_vals`).

	f. Provide a concrete implementation of `setup_transition_variable_groups` (instructions are analogous to those for `setup_dynamic_vals`).      

	g. Provide a concrete implementation of `setup_epi_metrics` (instructions are analogous to those for `setup_dynamic_vals`).  


## Creating a model from base components: checklist

Here we provide a brief outline of steps for creating a compartment model, intended as a reminder checklist. Please see the previous section for explanatory details. 

0. Formulate the compartment model.

1. Create a new file that imports the `base_components` module. We import this module as `base`. 

2. Create two `JSON` files: one for fixed parameters and one for initial values of state variables. 

	- Initial values for schedules and dynamic values can be left as `null`, but they still must have a field in the `JSON` file. 

3. Create subclasses to hold data: a subclass of `base.FixedParams` and a subclass of `base.SimState`. 

	- Note that there should be a one-to-one mapping between the fixed parameters `JSON` file and the attributes of `base.FixedParams`, and similarly for the initial values of state variables `JSON` and the attributes of `base.SimState`. 

4. For each transition variable in the model, create a unique subclass of `base.TransitionVariable` corresponding to that transition and provide a concrete implementation of `get_current_rate`.

5. For each dynamic value in the model, create a subclass of `base.DynamicVal` and provide a concrete implementation of `update_current_val`.

6. For each schedule in the model, create a subclass of `base.Schedule` and provide a concrete implementation of `update_current_val`.

7. Create a subclass of `base.ModelConstructor`. Customize the `_init_` function, making sure to assign the `config`, `fixed_params`, and `sim_state` attributes to instances of `base.Config`, `base.FixedParams`, and `base.SimState`. Provide concrete implementations of `setup_epi_compartments`, `setup_dynamic_vals`, `setup_schedules`, `setup_transition_variables`, `setup_transition_variable_groups`, and `setup_epi_metrics`. 