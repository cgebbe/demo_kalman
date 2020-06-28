# About

Simple documentation (maybe later code) to better understand the Kalman filter. Questions in particular:

- What exactly are the similarities and differences between the Kalman filter and the Particle Filter?
- Why is the Kalman gain calculated the way it is?
- How does the EKF and UKF work?



### Bayes filter

The Kalman filter is a special type of Bayes filter, so we start defining the Bayes filter first. The following information is mainly taken from https://en.wikipedia.org/wiki/Recursive_Bayesian_estimation.  

##### What is the Bayes filter?

> [The bayes filter] is a general probabilistic approach for estimating an unknown probability density function [of a system state] recursively over time.  [It uses]  (1) incoming [imperfect] measurements and (2) a mathematical process model.

In other words, the Bayes filter can be used to estimate a system state. Therefore, it uses (1) some kind of system-dependent measurements and (2) a system model. In order to that, the Bayes-filter assumes that the (unobservable) system state <img src="/tex/332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode&sanitize=true" align=middle width=9.39498779999999pt height=14.15524440000002pt/> can be modeled by a Markov process and that the measurements <img src="/tex/f93ce33e511096ed626b4719d50f17d2.svg?invert_in_darkmode&sanitize=true" align=middle width=8.367621899999993pt height=14.15524440000002pt/> are observations of this hidden Markov process (see image below).

![representation of hidden markov model](img/hmm.png)

The Markov property says that each state only depends on the previous state (and NOT the ones before!), so the probability of the state <img src="/tex/41a0912d0f46af38c7fa2115d8f0386e.svg?invert_in_darkmode&sanitize=true" align=middle width=16.66101689999999pt height=14.15524440000002pt/> and the measurement <img src="/tex/0d2605019e1d5873e2678162c8213642.svg?invert_in_darkmode&sanitize=true" align=middle width=14.91068204999999pt height=14.15524440000002pt/> can be expressed as

<p align="center"><img src="/tex/471b97e409a44f28729d432b754409bc.svg?invert_in_darkmode&sanitize=true" align=middle width=450.1688268pt height=99.05725005pt/></p>

As you can see, the Bayes filter depends on two conditional probability models, namely ...

1. ... the measurement model <img src="/tex/78892285ebaa2c3fde4b0c118b5e6739.svg?invert_in_darkmode&sanitize=true" align=middle width=53.60745884999999pt height=24.65753399999998pt/>, which tells us what measurement value we can expect given a certain system state and
2. ... the system model <img src="/tex/ca21b6ec130c23621ee168b0553ed8bb.svg?invert_in_darkmode&sanitize=true" align=middle width=72.18433364999999pt height=24.65753399999998pt/>, which tells us how the system evolves over time given an initial system state.

We have to mathematically define both models in order to be able to apply the Bayes filter! On the other hand, any mathematically definable model can be dealt with using a Bayes filter, see some examples in the table below.



| System description   | system model               | measurement model                       |
| -------------------- | -------------------------- | --------------------------------------- |
| car position         | movement of car over time  | position --> inaccurate GPS measurement |
| battery charge level | self-discharge over time   | charge level --> voltage                |
| daily weather        | weather dynamics over time | weather --> Clothes people wear         |



##### How to perform estimation?

So how do we estimate the system state <img src="/tex/41a0912d0f46af38c7fa2115d8f0386e.svg?invert_in_darkmode&sanitize=true" align=middle width=16.66101689999999pt height=14.15524440000002pt/> given (1)  the measurements <img src="/tex/beb1daf4cf026e9ac7350ddaced7e634.svg?invert_in_darkmode&sanitize=true" align=middle width=90.08591625pt height=14.15524440000002pt/> as well as (2) the measurement and the system model ? Let's start from the very beginning, where we don't have any measurements yet, but only <img src="/tex/e714a3139958da04b41e3e607a544455.svg?invert_in_darkmode&sanitize=true" align=middle width=15.94753544999999pt height=14.15524440000002pt/> (this has to be defined, in the simplest case as constant!). In this case, we can **predict** the next system state in a very simple way according to the [law of total probability](https://en.wikipedia.org/wiki/Law_of_total_probability):
<p align="center"><img src="/tex/68b5cf2ac86f21cb9b58703a73369397.svg?invert_in_darkmode&sanitize=true" align=middle width=389.0419764pt height=110.33822744999999pt/></p>



Next, we receive the measurement <img src="/tex/9846e95013d0238ac53659ac26ee63f2.svg?invert_in_darkmode&sanitize=true" align=middle width=14.197200599999992pt height=14.15524440000002pt/>. Based on this measurement, we can **update** our estimate for <img src="/tex/277fbbae7d4bc65b6aa601ea481bebcc.svg?invert_in_darkmode&sanitize=true" align=middle width=15.94753544999999pt height=14.15524440000002pt/> via
<p align="center"><img src="/tex/a0a4ceb2a5953cb0f79d072e64096edc.svg?invert_in_darkmode&sanitize=true" align=middle width=452.49353655000004pt height=137.30063159999997pt/></p>

The normalization term <img src="/tex/2a3735736bdf0eaf273cc43c85f42706.svg?invert_in_darkmode&sanitize=true" align=middle width=36.075113249999994pt height=24.65753399999998pt/> only serves to ensure that the probability density function adds up to one, i. e.  <img src="/tex/577a50e6f219109e8068abbee4736d99.svg?invert_in_darkmode&sanitize=true" align=middle width=126.57160889999999pt height=26.48417309999999pt/>.  It can be computed via
<p align="center"><img src="/tex/5b3fcccb1bfad349f1519c9b0b638bcd.svg?invert_in_darkmode&sanitize=true" align=middle width=256.10880899999995pt height=169.38121859999998pt/></p>







These two steps, prediction step and update step, can be applied to estimate all following states <img src="/tex/41a0912d0f46af38c7fa2115d8f0386e.svg?invert_in_darkmode&sanitize=true" align=middle width=16.66101689999999pt height=14.15524440000002pt/> as well. For time <img src="/tex/e021cb770c745ec45faa5ae82936a9b8.svg?invert_in_darkmode&sanitize=true" align=middle width=39.21220214999999pt height=22.831056599999986pt/> , the **prediction step** simply becomes
<p align="center"><img src="/tex/3d1cb4fe5c62fe39e68c5b497f84209c.svg?invert_in_darkmode&sanitize=true" align=middle width=408.62731305pt height=110.33822744999999pt/></p>

More generally, the prediction step can be defined as  
<p align="center"><img src="/tex/fd52bc8e2da65b1392b1844ec5180eaa.svg?invert_in_darkmode&sanitize=true" align=middle width=523.6407462pt height=36.53007435pt/></p>



In a similar manner, the **update step** for <img src="/tex/e021cb770c745ec45faa5ae82936a9b8.svg?invert_in_darkmode&sanitize=true" align=middle width=39.21220214999999pt height=22.831056599999986pt/> becomes
<p align="center"><img src="/tex/976b02df03bbdc79567684eef13c9ef9.svg?invert_in_darkmode&sanitize=true" align=middle width=476.23407315pt height=182.71091355pt/></p>

More generally, the update step can be defined as 
<p align="center"><img src="/tex/4a0bccea49799621cc2605d78e7cc0e3.svg?invert_in_darkmode&sanitize=true" align=middle width=397.16180625pt height=38.83491479999999pt/></p>

This can go on indefinitely. If we don't start from the beginning, but instead want to directly estimate the system state <img src="/tex/332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode&sanitize=true" align=middle width=9.39498779999999pt height=14.15524440000002pt/> at time <img src="/tex/63bb9849783d01d91403bc9a5fea12a2.svg?invert_in_darkmode&sanitize=true" align=middle width=9.075367949999992pt height=22.831056599999986pt/>, we have to estimate all previous states <img src="/tex/10b4cbf2c6d1e4afbcab1d61adb43742.svg?invert_in_darkmode&sanitize=true" align=middle width=120.18280064999999pt height=14.15524440000002pt/> in a recursive manner. Therefore, the Bayes filter is also called "recursive Bayes estimation".

##### Incorporating actions on system

Up until now, our system evolved over time without any interference from outside. An example would be a car with an initial velocity and steering wheel angle which never breaks, accelerates or turns.  However, in practice we often do interfere with the system using certain actions. Therefore, we have to expand our initial model to incorporate observable actions <img src="/tex/6dbb78540bd76da3f1625782d42d6d16.svg?invert_in_darkmode&sanitize=true" align=middle width=9.41027339999999pt height=14.15524440000002pt/> (see image below). To recap: We do know all actions <img src="/tex/6dbb78540bd76da3f1625782d42d6d16.svg?invert_in_darkmode&sanitize=true" align=middle width=9.41027339999999pt height=14.15524440000002pt/> and observations / measurements <img src="/tex/f93ce33e511096ed626b4719d50f17d2.svg?invert_in_darkmode&sanitize=true" align=middle width=8.367621899999993pt height=14.15524440000002pt/>, but we don't know the hidden system state <img src="/tex/332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode&sanitize=true" align=middle width=9.39498779999999pt height=14.15524440000002pt/>. 



![representation of HMM with actions](img/hmm_with_action.png)




| system description   | system model               | example action                  |
| -------------------- | -------------------------- | ------------------------------- |
| car position         | movement of car over time  | acceleration of car             |
| battery charge level | self-discharge over time   | charging battery                |
| daily weather        | weather dynamics over time | stratospheric aerosol injection |


These actions have to be considered in the prediction and update step, of course. The very first prediction step now becomes
<p align="center"><img src="/tex/5222b30755fae324e8b7f63ca79b613b.svg?invert_in_darkmode&sanitize=true" align=middle width=434.48354234999994pt height=153.4436376pt/></p>

As the equation suggests, our system model now not only needs to specify how the system evolves over time, but also how the system reacts to an action <img src="/tex/6dbb78540bd76da3f1625782d42d6d16.svg?invert_in_darkmode&sanitize=true" align=middle width=9.41027339999999pt height=14.15524440000002pt/>!


Next, we receive the measurement <img src="/tex/9846e95013d0238ac53659ac26ee63f2.svg?invert_in_darkmode&sanitize=true" align=middle width=14.197200599999992pt height=14.15524440000002pt/>. Based on this measurement, we can **update** our estimate for <img src="/tex/277fbbae7d4bc65b6aa601ea481bebcc.svg?invert_in_darkmode&sanitize=true" align=middle width=15.94753544999999pt height=14.15524440000002pt/> via
<p align="center"><img src="/tex/b830d115c1a15b1edc44d9f0f2753118.svg?invert_in_darkmode&sanitize=true" align=middle width=476.5841454pt height=182.71091355pt/></p>





More generally, the prediction step can be written as...
<p align="center"><img src="/tex/62f35d1bc17322f6b2df870cc557e46b.svg?invert_in_darkmode&sanitize=true" align=middle width=615.0397571999999pt height=79.6354845pt/></p>

and the update step as ...
<p align="center"><img src="/tex/78afbf3e91d69bb27927c7725554a2d7.svg?invert_in_darkmode&sanitize=true" align=middle width=477.68343974999993pt height=127.92031559999998pt/></p>

In many [sources](http://ais.informatik.uni-freiburg.de/teaching/ws12/mapping/pdf/slam02-bayes-filter-short.pdf)  the notation of belief is introduced, making the recursive nature of the algorithm even more obvious:

<p align="center"><img src="/tex/9758ced3ec07da7e127401351e5dfb0e.svg?invert_in_darkmode&sanitize=true" align=middle width=0.0pt height=0.0pt/></p>



### Kalman filter

XXX