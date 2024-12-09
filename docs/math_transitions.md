# Stochastic Transitions

> **_Written by LP, updated 11/21/2024 (work in progress)_** 

## Marginal transition types

Our codebase currently implements six types of transitions: three stochastic transitions and three deterministic counterparts.

First we consider marginal (not joint) transition variables. For the following table, consider $y_{\texttt{C}\rightarrow\texttt{C}^\prime, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)$ from compartment $\texttt{C}$ to $\texttt{C}^\prime$. Let $\texttt{b} = \texttt{b}_{a, \ell}(t)$ be its current value of its base count and $\texttt{r} = \texttt{r}_{a, \ell}(t)$ be its current value of its rate. Then depending on its transition type, its distribution (or deterministic value) is given below.

Note that $\alpha$ is a function that converts a "rate" into a probability. It is given by:
$$
\alpha(\texttt{r}, \Delta t) = 1 - \exp(-\texttt{r} \cdot \Delta t)
$$
and corresponds to the probability that a Poisson process with rate $\texttt{r}$ produces at least one event in an interval of length $\Delta t$. 

The following table provides the mathematical distribution or deterministic output for each transition type. 

|                                      | Output                                                                           |
|--------------------------------------|----------------------------------------------------------------------------------|
| Binomial                             | $\sim \text{Binom}\left(n = \texttt{b}, p = \alpha(\texttt{r}, \Delta t)\right)$ |
| Binomial  Taylor Approx              | $\sim \text{Binom}\left(n = \texttt{b}, p = \texttt{r} \cdot \Delta t\right)$        |
| Poisson                              | $\sim \text{Poisson}\left(\lambda = \texttt{b} \cdot \texttt{r} \cdot \Delta t\right)$      |
| Binomial Deterministic               | $\texttt{b} \cdot \alpha(\texttt{r}, \Delta t)$                                        |
| Binomial Taylor Approx Deterministic | $\texttt{b} \cdot \texttt{r} \cdot \Delta t$                                                 |
| Poisson Deterministic                | $\texttt{b} \cdot \texttt{r} \cdot \Delta t$                                               |

## Joint transition types

Here we describe joint transition variables. For the following table, consider a compartment $\texttt{C}_0$ with two outgoing compartments, $\texttt{C}_1$ and $\texttt{C}_2$. We describe $2$ transition variables: $y_{\texttt{C}_0\rightarrow\texttt{C}_1, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)$ and $y_{\texttt{C}_0\rightarrow\texttt{C}_2, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)$. They respectively correspond to the number of people that transition from $\texttt{C}_0$ to $\texttt{C}_1$ and the number that transition from $\texttt{C}_0$ to $\texttt{C}_2$ at a given simulate state. Let their rates be $\texttt{r}_1$ and $\texttt{r}_2$ respectively.  

We can also write the number that remain in $\texttt{C}_0$ explicitly as 
$y_{\texttt{C}_0\rightarrow\texttt{C}_0, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right) = \texttt{C}_{0, a, \ell}(t) - y_{\texttt{C}_0\rightarrow\texttt{C}_1, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right) - y_{\texttt{C}_0\rightarrow\texttt{C}_2, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)$. Note that this quantity is purely for parametrizing multinomial distributions (which needs to have its probability parameters sum to $1$) and does not need to be literally modeled as a transition variable either mathematically or in code. 

Let $\alpha$ be defined as in the previous section on marginal transition types.

The following table gives the parameters for multinomial transitions when $\texttt{C}_0$ has two outgoing compartments $\texttt{C}_1$ and $\texttt{C}_2$. The values $[y_{\texttt{C}_0\rightarrow\texttt{C}_1, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)$, $y_{\texttt{C}_0\rightarrow\texttt{C}_2, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)$, $y_{\texttt{C}_0\rightarrow\texttt{C}_0, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)]$ are sampled jointly, using $\texttt{b} = \texttt{b}_{a, \ell}(t) = \texttt{C}_{0, a, \ell}(t)$ as the "number of trials" parameter $n$ in a multinomial distribution.

|                                             Transition  Variable Group (Jointly Sampled)                                           |                                 Multinomial Probability Parameter                                | Multinomial with Taylor Approx Probability Parameter |
|:-----------------------------------------------------------------------------------------------------------:|:------------------------------------------------------------------------------------------------:|:----------------------------------------------------:|
| $y_{\texttt{C}_0\rightarrow\texttt{C}_1, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)$ | $\frac{\texttt{r}_1}{\texttt{r}_1 + \texttt{r}_2} \alpha(\texttt{r}_1 + \texttt{r}_2, \Delta t)$ | $\texttt{r}_1 \cdot \Delta t$                              |
| $y_{\texttt{C}_0\rightarrow\texttt{C}_2, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)$ | $\frac{\texttt{r}_2}{\texttt{r}_1 + \texttt{r}_2} \alpha(\texttt{r}_1 + \texttt{r}_2, \Delta t)$ | $\texttt{r}_2 \cdot \Delta t$                              |
| $y_{\texttt{C}_0\rightarrow\texttt{C}_0, a, \ell}\left(\boldsymbol{\mathcal X}(t), \Delta t, \omega\right)$ | $1 - \alpha(\texttt{r}_1 + \texttt{r}_2, \Delta t)$                                              | $1 - (\texttt{r}_1 + \texttt{r}_2) \Delta t$         |

IMPORTANT:

- The transition variable group table formulas generalize to an arbitrary number of outgoing compartments -- however, this is not shown here.

- Note that Poisson is not included in the above table. Recall the splitting property of Poisson processes. If the total outflow (going to either compartment, $\texttt{C}_1$ or $\texttt{C}_2$) is Poisson, we can split the process into two independent Poisson processes. So joint sampling is not actually needed. 

- In contrast, the binomial distribution does not have such a splitting property. If the total outflow is binomial with parameters $n$ and $p$, then we cannot use a similar Poisson splitting technique to create two independent binomial random variables with the same number of trial parameters $n$. Thus, joint sampling is needed. 

- An advantage of binomial/multinomial transition variables (with joint sampling when there are multiple outflows from a compartment) is that we cannot have more people leaving the origin compartment than are actually in the compartment. We set the "number of trials" parameter in the multinomial distribution to be the current number of people in the compartment. This is ***NOT*** the case for Poisson transition variables, which are unbounded! Thus, we recommend using binomial/multinomial transition types.