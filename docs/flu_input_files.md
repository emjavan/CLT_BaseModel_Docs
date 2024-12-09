# Flu Model Input Files Overview

> **_Written by LP, updated 11/26/2024 (work in progress)_** 

Here we provide a mapping between the input variable names in the flu model's `JSON` files (and corresponding `dataclasses`), which become object attribute names in the model) and the mathematical variable. 

The folder `flu_demo_input_files` contains 3 `JSON` input files used to run `flu_demo.py`: `config.json`, `fixed_params.json`, and `state_variables_init_vals.json`. These input files respectively initialize `Config` instances, `FluFixedParams` instances, and `FluSimState` instances.

In general, for users to customize the *values* (not the structure) of the flu model given in `flu_components.py`, they must provide analogous 3 `JSON` input files with the following formats.

## Simulation configuration

The table below describes the `JSON` file used to initialize `Config` instances. These values specify the simulation experiment configuration. 

| Name                                  | Explanation                | Format                                   |
|---------------------------------------|----------------------------|------------------------------------------|
| `timesteps_per_day`                   | number of discretized timesteps per day -- larger values may increase accuracy of approximation to continuous time but slow down simulation           							 | positive `int`                           |
| `transition_type`           			| specifies distribution of stochastic or deterministic population transitions between compartments -- see [mathematical definition of transition types](math_flu_components.md) for specific formulas         | `str`, must have value in `base_components.TransitionTypes` to be valid                                    |
| `start_real_date`                 	| real-world date corresponding to start of simulation                  | `str`, must have format `"YYYY-MM-DD"` |
| `save_daily_history`          		| indicates if daily history is saved on `EpiCompartment` instances -- may want to strategically turn off to save time during performance runs          | `bool` |


## Fixed parameters

The table below has a variable name to math variable mapping for the `JSON` file used to initialize `FluFixedParams` instances. We can think of this file as the "Greek letter file" -- it specifies values such as transition rates as well as number of age groups and risk groups. We assume that these values do not change throughout the course of the simulation.  

| Name                            | Math Variable              | Dimension                                |
|---------------------------------|----------------------------|------------------------------------------|
| `num_age_groups`                | $\lvert A \rvert$          | positive `int`                           |
| `num_risk_groups`               | $\lvert L \rvert$          | positive `int`							  |
| `beta_baseline`                 | $\beta_0$                  | positive scalar                          |
| `total_population_val`          | $\boldsymbol{N}$           | $\boldsymbol{\tilde{\nu}}$ | $\lvert A \rvert                            |
| `humidity_impact`               | $\xi$                      | scalar                                   |
| `immunity_hosp_increase_factor` | $g^H$                      | scalar                                   |
| `immunity_inf_increase_factor`  | $g^I$                      | scalar                                   |
| `immunity_saturation_constant`  | $\boldsymbol{O}$           | $\lvert A \rvert \times \lvert L \rvert$ |
| `waning_factor_hosp`            | $w^H$                      | positive scalar 						  |
| `waning_factor_inf`             | $w^I$                      | positive scalar                          |
| `hosp_risk_reduction`           | $\boldsymbol{K}^H$         | $\lvert A \rvert \times \lvert L \rvert$ |
| `inf_risk_reduction`            | $\boldsymbol{K}^I$         | $\lvert A \rvert \times \lvert L \rvert$ |
| `death_risk_reduction`          | $\boldsymbol{K}^D$         | $\lvert A \rvert \times \lvert L \rvert$ |
| `R_to_S_rate`                   | $\eta$                     | positive scalar                          |
| `E_to_I_rate`                   | $\sigma$                   | positive scalar                          |
| `IP_to_IS_rate`				  | $\rho$					   | positive scalar						  |
| `IS_to_R_rate`                  | $\gamma$                   | positive scalar                          |
| `IA_to_R_rate`				  | $\gamma_{IA}$			   | positive scalar						  |
| `IS_to_H_rate`                  | $\zeta$                    | positive scalar                          |
| `H_to_R_rate`                   | $\gamma_H$                 | positive scalar                          |
| `H_to_D_rate`                   | $\pi$                      | positive scalar                          |
| `E_to_IA_prop`                  | $\tau$                     | scalar in $[0,1]$                        |
| `H_to_D_adjusted_prop`    	  | $\boldsymbol{\tilde{\nu}}$ | $\lvert A \rvert \times \lvert L \rvert$ |
| `IS_to_H_adjusted_prop`   	  | $\boldsymbol{\tilde{\mu}}$ | $\lvert A \rvert \times \lvert L \rvert$ |
| `IP_relative_inf`   			  | $r_{IP}$ 				   | positive scalar 						  |
| `IA_relative_inf`			   	  | $r_{IA}$ 				   | positive scalar                          |

## Initial values of state variables

The table below has a variable name to math variable mapping for the `JSON` file used to initialize `FluSimState` instances. This file specifies initial conditions for `StateVariable` objects (which includes `EpiCompartment`, `EpiMetric`, `Schedule`, and `DynamicVal` objects). 

Important notes:

- In `state_variables_init_vals.json` in `flu_demo_input_files`, `absolute_humidity` and `flu_contact_matrix` have initial values of `null` (which gets translated to `None` in `Python`). This is because absolute humidity and the flu contact matrix are `Schedule` instances, and get their values deterministically updated according to a schedule. At every simulation day `t`, their values are well-defined (taken from a schedule calendar), regardless of the initial value. Therefore, the initial value does not matter and we initialize these objects with a current value of `None`. However, these objects are indeed updated and used in the simulation.

- Currently, `beta_reduct` is not a part of the [mathematical formulation](math_flu_components.md). In the code, `beta_reduct` is a `DynamicVal` that decreases `beta_baseline` when a certain simulation state is triggered. This emulates a very simple staged-alert policy. In `flu_components.py`, `beta_reduct` is automatically disabled, so that the simple staged-alert policy is not in effect during default flu model runs. When enabled, `beta_reduct` has a value of `0.5` (corresponding to a $50\%$ decrease in transmission rates) when more than $5\%$ of the population is infected, and it has a value of `0.0` otherwise. 

| Name                       | Math Variable                        | Dimension                                |
|----------------------------|--------------------------------------|------------------------------------------|
| `S`                        | $\boldsymbol{S}(0)$                | $\lvert A \rvert \times \lvert L \rvert$ |
| `E`                        | $\boldsymbol{E}(0)$                | $\lvert A \rvert \times \lvert L \rvert$ |
| `IP`                        | $\boldsymbol{I^P}(0)$                | $\lvert A \rvert \times \lvert L \rvert$ |
| `IS`                        | $\boldsymbol{I^S}(0)$                | $\lvert A \rvert \times \lvert L \rvert$ |
| `IA`                        | $\boldsymbol{I^A}(0)$                | $\lvert A \rvert \times \lvert L \rvert$ |
| `H`                        | $\boldsymbol{H}(0)$                | $\lvert A \rvert \times \lvert L \rvert$ |
| `R`                        | $\boldsymbol{R}(0)$                | $\lvert A \rvert \times \lvert L \rvert$ |
| `D`                        | $\boldsymbol{D}(0)$                | $\lvert A \rvert \times \lvert L \rvert$ |
| `population_immunity_inf`  | $\boldsymbol{M}^I(0)$              | $\lvert A \rvert \times \lvert L \rvert$ |
| `population_immunity_hosp` | $\boldsymbol{M}^H(0)$              | $\lvert A \rvert \times \lvert L \rvert$ |
| `absolute_humidity`        | `absolute_humidity` $= \xi / q(0)$ | scalar                                   |
| `flu_contact_matrix`       | $\boldsymbol{\phi}(0)$ 			  | $\lvert A \rvert \times \lvert L \rvert \times \lvert A \rvert \times \lvert L \rvert$ |
| `beta_reduct`				 | N/A								  | scalar in $[0,1]$									 |

