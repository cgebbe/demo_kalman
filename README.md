# About

Documentation (maybe later code) to better understand the Kalman filter. Questions in particular:

- What is the relationship between a Kalman filter and the Bayes filter
- Why is the Kalman gain calculated the way it is?
- How does the EKF and UKF work?
- How does a particle filter differ from the Kalman filter?



## Bayes filter

The Kalman filter is a special type of Bayes filter, so we start defining the Bayes filter first. The following information is mainly taken from https://en.wikipedia.org/wiki/Recursive_Bayesian_estimation.  

#### What is the Bayes filter?

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



#### How to perform estimation?

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

#### Incorporating actions on system

Up until now, our system evolved over time without any interference from outside. An example would be a car with an initial velocity and steering wheel angle which never breaks, accelerates or turns.  However, in practice we often do interfere with the system using certain actions. Therefore, we have to expand our initial model to incorporate observable actions <img src="/tex/6dbb78540bd76da3f1625782d42d6d16.svg?invert_in_darkmode&sanitize=true" align=middle width=9.41027339999999pt height=14.15524440000002pt/> (see image below). To recap: We do know all actions <img src="/tex/6dbb78540bd76da3f1625782d42d6d16.svg?invert_in_darkmode&sanitize=true" align=middle width=9.41027339999999pt height=14.15524440000002pt/> and observations / measurements <img src="/tex/f93ce33e511096ed626b4719d50f17d2.svg?invert_in_darkmode&sanitize=true" align=middle width=8.367621899999993pt height=14.15524440000002pt/>, but we don't know the hidden system state <img src="/tex/332cc365a4987aacce0ead01b8bdcc0b.svg?invert_in_darkmode&sanitize=true" align=middle width=9.39498779999999pt height=14.15524440000002pt/>. 



![representation of HMM with actions](img/hmm_with_action.png)




| system description   | system model               | example action                  |
| -------------------- | -------------------------- | ------------------------------- |
| car position         | movement of car over time  | acceleration of car             |
| battery charge level | self-discharge over time   | charging battery                |
| daily weather        | weather dynamics over time | stratospheric aerosol injection |


These actions have to be considered in the prediction and update step, of course. The very first prediction step now becomes
<p align="center"><img src="/tex/70cc65374ccfd826ff5311eb9321cc97.svg?invert_in_darkmode&sanitize=true" align=middle width=434.48354234999994pt height=153.4436376pt/></p>

As the equation suggests, our system model now not only needs to specify how the system evolves over time, but also how the system reacts to an action <img src="/tex/6dbb78540bd76da3f1625782d42d6d16.svg?invert_in_darkmode&sanitize=true" align=middle width=9.41027339999999pt height=14.15524440000002pt/>!


Next, we receive the measurement <img src="/tex/9846e95013d0238ac53659ac26ee63f2.svg?invert_in_darkmode&sanitize=true" align=middle width=14.197200599999992pt height=14.15524440000002pt/>. Based on this measurement, we can **update** our estimate for <img src="/tex/277fbbae7d4bc65b6aa601ea481bebcc.svg?invert_in_darkmode&sanitize=true" align=middle width=15.94753544999999pt height=14.15524440000002pt/> via
<p align="center"><img src="/tex/8c1d4bf2c2d5ec4a2301e58920e0347d.svg?invert_in_darkmode&sanitize=true" align=middle width=476.5841454pt height=182.71091355pt/></p>





More generally, the prediction step can be written as...
<p align="center"><img src="/tex/d378c40abc99d5e0d8ec20b8eea92ccc.svg?invert_in_darkmode&sanitize=true" align=middle width=631.8662955pt height=79.6354845pt/></p>

and the update step as ...
<p align="center"><img src="/tex/ef3dd4d167ca32fa67be2534e0410150.svg?invert_in_darkmode&sanitize=true" align=middle width=477.16766294999996pt height=127.92031559999998pt/></p>

In many [sources](http://ais.informatik.uni-freiburg.de/teaching/ws12/mapping/pdf/slam02-bayes-filter-short.pdf)  the notation of belief is introduced, making the recursive nature of the algorithm even more obvious:

<p align="center"><img src="/tex/f8defb71ba97ec0a12f181f856df818c.svg?invert_in_darkmode&sanitize=true" align=middle width=440.96368635pt height=61.91801715pt/></p>



#### Type of probability distribution

Up until now, we have never further specified how exactly the probability distribution <img src="/tex/ec17af2e0155abf8fc68813133526228.svg?invert_in_darkmode&sanitize=true" align=middle width=38.53893284999999pt height=24.65753399999998pt/> shall look like.  The probability distribution can in theory take any form, for example

- a normal distribution (Kalman filter!)
- a number of discrete points sampled from an arbitrary probability distribution (particle filter!)
- a gamma distribution (?)
- ... 

Depending on the type of probability distribution, some useful properties emerge, which we will analyze in the following.

## Kalman filter

#### Assumptions 

As mentioned above, the Kalman filter assumes that the probability density function ("PDF") is a (multivariate) normal distribution of the following form
<p align="center"><img src="/tex/3ab8a916bb252fe76affbe828e65a15c.svg?invert_in_darkmode&sanitize=true" align=middle width=433.94487675pt height=42.4111644pt/></p>
, where <img src="/tex/b8e7943cc8e0043fe55d353165cbe678.svg?invert_in_darkmode&sanitize=true" align=middle width=66.69513509999999pt height=22.648391699999998pt/> and <img src="/tex/c621a26ecacdba7a3f747a8d43dccb99.svg?invert_in_darkmode&sanitize=true" align=middle width=70.36156379999998pt height=26.17730939999998pt/> . During the prediction and update step of the Bayes filter we perform three types of calculations

1. We multiply two probability density functions together
2. We calculate <img src="/tex/2da3d24881616489274f806086ff7f6a.svg?invert_in_darkmode&sanitize=true" align=middle width=102.2187276pt height=24.65753399999998pt/> in the prediction step using the system model
3. We calculate <img src="/tex/bc4f592922fbe07b70337e7da4f5ed11.svg?invert_in_darkmode&sanitize=true" align=middle width=58.83775424999998pt height=24.65753399999998pt/> in the update step using the measurement model

Concerning the first point: Multiplying two normal distributions always yields another normal distribution ([source1](https://math.stackexchange.com/questions/157172/product-of-two-multivariate-gaussians-distributions), [source2 on p.331](https://mitpress.mit.edu/books/introduction-autonomous-mobile-robots)):
<p align="center"><img src="/tex/616b253551e77de4372ad37b2f23bad8.svg?invert_in_darkmode&sanitize=true" align=middle width=239.48218799999998pt height=168.90799694999998pt/></p>
Concerning point 2 and point 3. In general, it cannot be guaranteed that the resulting PDF of the prediction and update step is also a normal PDF. In order to ensure this, we have to constrain the type of system model and measurement model. These models must be a linear combination of the [following form](https://en.wikipedia.org/wiki/Kalman_filter#Underlying_dynamical_system_model), otherwise the resulting PDF is not a normal PDF anymore:

- <img src="/tex/ac3b01dd6cb4536d11129ca12b4f40fc.svg?invert_in_darkmode&sanitize=true" align=middle width=231.7525089pt height=24.65753399999998pt/>  for the system model

- <img src="/tex/6589e3c73a106a305a12177ba88c58e4.svg?invert_in_darkmode&sanitize=true" align=middle width=154.3227939pt height=24.65753399999998pt/> for the measurement model

, where <img src="/tex/a88dc4b54d7ad179ff6e0a62c7ff8e41.svg?invert_in_darkmode&sanitize=true" align=middle width=107.07783404999999pt height=24.65753399999998pt/> and <img src="/tex/0a719e96e317d100c8baa00da0a3214d.svg?invert_in_darkmode&sanitize=true" align=middle width=102.76344209999999pt height=24.65753399999998pt/> are assumed to be process noise with zero mean.

#### Estimation

Since the PDFs are always a normal distribution, which can be modeled with only two parameters (<img src="/tex/4d67dbdc945ea6d80cf99757b756e777.svg?invert_in_darkmode&sanitize=true" align=middle width=29.08298579999999pt height=22.465723500000017pt/>), the calculations become very computationally efficient. We start with the initial PDF 
<p align="center"><img src="/tex/5311b733bd0bea627a6be9554f2b692f.svg?invert_in_darkmode&sanitize=true" align=middle width=132.27027329999999pt height=16.438356pt/></p>
The first prediction step is then 
<p align="center"><img src="/tex/30116ce748fc9bbaa2fb4fd5fba28621.svg?invert_in_darkmode&sanitize=true" align=middle width=442.87413225pt height=224.358981pt/></p>




The first update step becomes
<p align="center"><img src="/tex/9cffb13598fbdaa68dce83edc45b2c1d.svg?invert_in_darkmode&sanitize=true" align=middle width=378.4132869pt height=196.23909689999996pt/></p>

The variable <img src="/tex/d6328eaebbcd5c358f426dbea4bdbf70.svg?invert_in_darkmode&sanitize=true" align=middle width=15.13700594999999pt height=22.465723500000017pt/> is often referred to as *Kalman gain*. The prediction and update step can be continued indefinitely.  See general [notation and derivation on wikipedia](https://en.wikipedia.org/wiki/Kalman_filter#Details). Other helpful sources might be

- https://arxiv.org/pdf/1910.03558.pdf 
- [http://web.mit.edu/kirtley/kirtley/binlustuff/literature/control/Kalman%20filter.pdf](http://web.mit.edu/kirtley/kirtley/binlustuff/literature/control/Kalman filter.pdf)

#### Properties of the Kalman filter

The Kalman filter is optimal in the sense that **if** the system and measurement models are accurately described it will converge to the exact solution.

#### Extended Kalman filter (EKF)

In the normal Kalman filter we needed to constrain the type of system and measurement model to linear combinations in order to keep the resulting PDF a normal PDF. This is a problem because many systems in practice are not simple linear combinations but behave nonlinearly. Generally, we can write the models as follows
<p align="center"><img src="/tex/d955ee83ba4a038cb9f4bfd77c281cc6.svg?invert_in_darkmode&sanitize=true" align=middle width=364.63686435pt height=41.09589pt/></p>

In order to cope with such nonlinear systems, the extended Kalman filter performs a small trick:

- The mean of the resulting normal PDF can be calculated using the non-linear functions
- In order to calculate its variance, the non-linear functions are simply linearly approximated around the mean using a Taylor approximation

This results in the following prediction and update step ([source: wikipedia](https://en.wikipedia.org/wiki/Extended_Kalman_filter)):

![1593376614336](README.tex.assets/1593376614336.png)



#### Unscented Kalman filter (UKF)

XXX