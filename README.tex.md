# About

Simple documentation (maybe later code) to better understand the Kalman filter. Questions in particular:

- What exactly are the similarities and differences between the Kalman filter and the Particle Filter?
- Why is the Kalman gain calculated the way it is?
- How does the EKF and UKF work?



### Bayes filter

The Kalman filter is a special type of Bayes filter, so we start defining the Bayes filter first. The following information is mainly taken from https://en.wikipedia.org/wiki/Recursive_Bayesian_estimation.  

##### What is the Bayes filter?

> [The bayes filter] is a general probabilistic approach for estimating an unknown probability density function [of a system state] recursively over time.  [It uses]  (1) incoming [imperfect] measurements and (2) a mathematical process model.

In other words, the Bayes filter can be used to estimate a system state. Therefore, it uses (1) some kind of system-dependent measurements and (2) a system model. In order to that, the Bayes-filter assumes that the (unobservable) system state $x$ can be modeled by a Markov process and that the measurements $z$ are observations of this hidden Markov process (see image below).

![representation of hidden markov model](img/hmm.png)

The Markov property says that each state only depends on the previous state (and NOT the ones before!), so the probability of the state $x_k$ and the measurement $z_k$ can be expressed as

$$
\begin{aligned}
p(x_k|x_{k-1},x_{k-2}, ... ,x_0) &= p(x_k|x_{k-1}) \\
p(z_k|x_k, x_{k-1}, ... ,x_0) &= p(z_k|x_k) \\
\Rightarrow p(x_k, x_{k-1}, ... x_0, z_k, z_{k-1}, ... z_0)
&= p(x_0) \prod_{i=1}^{k}p(z_i|x_i)p(x_i|x_{i-1})
\\
\end{aligned}
$$

As you can see, the Bayes filter depends on two conditional probability models, namely ...

1. ... the measurement model $p(z_i|x_i)$, which tells us what measurement value we can expect given a certain system state and
2. ... the system model $p(x_i|x_{i-1})$, which tells us how the system evolves over time given an initial system state.

We have to mathematically define both models in order to be able to apply the Bayes filter! On the other hand, any mathematically definable model can be dealt with using a Bayes filter, see some examples in the table below.



| System description   | system model               | measurement model                       |
| -------------------- | -------------------------- | --------------------------------------- |
| car position         | movement of car over time  | position --> inaccurate GPS measurement |
| battery charge level | self-discharge over time   | charge level --> voltage                |
| daily weather        | weather dynamics over time | weather --> Clothes people wear         |



##### How to perform estimation?

So how do we estimate the system state $x_k$ given (1)  the measurements $z_1, z_2, ... z_{k-1}$ as well as (2) the measurement and the system model ? Let's start from the very beginning, where we don't have any measurements yet, but only $x_0$ (this has to be defined, in the simplest case as constant!). In this case, we can **predict** the next system state in a very simple way:
$$
\begin{aligned}
p(x_1|x_0) =& p(x_1|x_0)p(x_0) \\
\\
& p(x_1|x_0) \text{ can be computed using system model} \\
& p(x_0) \text{ is known} \\
\end{aligned}
$$



Next, we receive the measurement $z_1$. Based on this measurement, we can **update** our estimate via
$$
\begin{aligned}
p(x_1|z_1, x_0) =& \frac{p(z_1|x_1, x_0)p(x_1|x_0)}{p(z_1|x_0)}  \\
=& \frac{p(z_1|x_1)p(x_1|x_0)}{p(z_1)} \qquad \text{(Markov property!)} \\
\\
& p(z_1|x_1) \text{ can be computed using measurement model} \\
& p(x_1|x_0) \text{ is known from previous oprediction step} \\
& p(z_1) \text{ is only a normalization term independent of $x_1$} \\
\end{aligned}
$$

The normalization term $p(z_1)$ only serves to ensure that the probability density function adds up to one, i. e.  .$\int p(x_1|z_1x_0) dz_1 =1$.  It can be computed via $p(z_1)=\int p(z_1|x_1)dx_1$.





These two steps, prediction step and update step, can be applied to estimate all following states $x_k$ as well. For time $k=2$ , the **prediction step** simply becomes
$$
\begin{aligned}
p(x_2|x_1, x_0, z_1) 
=& p(x_2|x_1,R) \qquad \text{ with } R=x_0, z_1 \\
=&  p(x_2|x_1) p(x_1|R) \\
=&  p(x_2|x_1) p(x_1|x_0, z_1) \\
\\
& p(x_2|x_1) \text{ can be computed using system model} \\
& p(x_1|x_0, z_1)  \text{ is known from last update step} \\
\end{aligned}
$$

More generally, the prediction step can be defined as  
$$
\begin{aligned}
p(x_k|x_{k-1}, x_{k-2}, ..., x_0, z_{k-1}, z_{k-2},.., z_1) =&  p(x_k|x_{k-1:0},z_{k-1:1}) \\
=& p(x_k|x_{k-1}) p(x_{k-1}|x_{k-2:0},z_{k-1:1})
\end{aligned}
$$



In a similar manner, the **update step** for $k=2$ becomes
$$
\begin{aligned}
p(x_2|z_2, z_1, x_1, x_0) 
=& p(x_2|z_2, R) \qquad \text{ with } R=z_1,x_1,x_0   \\
=& \frac{p(z_2|x_2, R)p(x_2|R)}{p(z_2|R)}  \\
=& \frac{p(z_2|x_2, z_1,x_1,x_0)p(x_2|z_1,x_1,x_0)}{p(z_2|z_1,x_1,x_0)}  \\
=& \frac{p(z_2|x_2)p(x_2|z_1,x_1,x_0)}{p(z_2)} \qquad \text{(Markov property!)} \\
\\
& p(z_2|x_2) \text{ can be computed using measurement model} \\
& p(x_2|z_1,x_1,x_0) \text{ is known from previous prediction step} \\
& p(z_2) \text{ is only a normalization term independent of $x_2$} \\
\end{aligned}
$$

More generally, the update step can be defined as 
$$
\begin{aligned}
p(x_k|x_{k-1}, x_{k-2}, ..., x_0, z_k, z_{k-1},.., z_1)
=&  p(x_k|x_{k-1:0},z_{k:1}) \\
=& \frac{p(z_k|x_k)p(x_k|x_{k-1:0},z_{k-1:1})}{p(z_k)} \\
\end{aligned}
$$

This can go on indefinitely. If we don't start from the beginning, but instead want to directly estimate the system state $x$ at time $k$, we have estimate all previous states $x_{k-1}, x_{k-2}, ..., x_0$ in a recursive manner. Therefore, the Bayes filter is also called "recursive Bayes estimation".

##### Incorporating actions on system

Up until now, our system evolved over time without any interference from outside. An example would be a car with an initial velocity and steering wheel angle which never breaks, accelerates or turns.  However, in practice we often do interfere with the system using certain actions. Therefore, we have to expand our initial model to incorporate observable actions $u$ (see image below). To recap: We do know all actions $u$ and observations / measurements $z$, but we don't know the hidden system state $x$. 



![representation of HMM with actions](img/hmm_with_action.png)




| system description   | system model               | example action                  |
| -------------------- | -------------------------- | ------------------------------- |
| car position         | movement of car over time  | acceleration of car             |
| battery charge level | self-discharge over time   | charging battery                |
| daily weather        | weather dynamics over time | stratospheric aerosol injection |


These actions have to be considered in the prediction and update step, of course. The very first prediction step now becomes
$$
\begin{aligned}
p(x_1|x_0,u_1) =& p(x_1|x_0,u_1)p(x_0) \\
\\
& p(x_1|x_0) \text{ can be computed using system model} \\
& p(x_0) \text{ is known} \\
\end{aligned}
$$





# Eqn array

$$
\begin{aligned}
\dot{x} & = \sigma(y-x) & 12\\
\dot{y} & = \rho x - y - xz & 34\\
\dot{z} & = -\beta z + xy
\end{aligned}
$$

foo bar