# Notes on An introduction to data analysis with aggregation functions in R

> [Book link](https://link.springer.com/book/10.1007/978-3-319-46762-7)

- [Aggregating data with averaging functions](#aggregating-data-with-averaging-functions)
	- [Notation](#notation)
	- [Arithmetic mean](#arithmetic-mean)
		- [Properties](#properties)
	- [Median](#median)
	- [Geometric and Harmonic means](#geometric-and-harmonic-means)
	- [Harmonic mean](#harmonic-mean)
		- [Case](#case)
	- [Aggregation function](#aggregation-function)
- [Transforming data](#transforming-data)
	- [Multicriteria eval](#multicriteria-eval)
		- [Which is better? Higher or Lower?](#which-is-better-higher-or-lower)
	- [Negations and Utility transformations](#negations-and-utility-transformations)
	- [Scaling, Standardization and Normalization](#scaling-standardization-and-normalization)
		- [Linear feature scaling](#linear-feature-scaling)
		- [Standardization](#standardization)
	- [Transoformation](#transoformation)
		- [Power mean](#power-mean)
		- [Quasi-Arithmetic mean (Kolmogorov mean)](#quasi-arithmetic-mean-kolmogorov-mean)
- [Weighted averaging](#weighted-averaging)
	- [Weighing vector](#weighing-vector)
	- [Weighted arithmetic mean](#weighted-arithmetic-mean)
	- [Weighted power mean](#weighted-power-mean)
	- [Weighted median](#weighted-median)
	- [Simpson's Dominance Index](#simpsons-dominance-index)
	- [Borda count](#borda-count)
- [Averaging with Interaction](#averaging-with-interaction)
	- [Trimmed and Winsorized mean](#trimmed-and-winsorized-mean)
	- [Ordered weighted average](#ordered-weighted-average)
	- [Orness](#orness)
	- [Quantifier function](#quantifier-function)
	- [Fuzzy measure](#fuzzy-measure)
	- [Choquet integral](#choquet-integral)
	- [Shapley value](#shapley-value)
- [Fitting aggregation functions to empirical data](#fitting-aggregation-functions-to-empirical-data)
	- [Distance](#distance)
	- [Euclidean distance](#euclidean-distance)
	- [Mikowski distance](#mikowski-distance)
	- [Linear programming](#linear-programming)
		- [Sample use case](#sample-use-case)
	- [Evaluating and interpreting model's accuracy](#evaluating-and-interpreting-models-accuracy)
		- [Sum of squared errors](#sum-of-squared-errors)
		- [Root mean squared error](#root-mean-squared-error)
		- [Sum of absolute errors](#sum-of-absolute-errors)
		- [Pearson correlation](#pearson-correlation)
		- [Spearman's Rank correlation coefficient](#spearmans-rank-correlation-coefficient)
	- [Learning weights for aggregation functions](#learning-weights-for-aggregation-functions)
	- [Are We Evaluating an Approach or the Model Itself?](#are-we-evaluating-an-approach-or-the-model-itself)


## Aggregating data with averaging functions

> Aggregation functions are mathematical computations that involve a range of values, resulting in a single value expressing the significance of the accumulated data.

### Notation

Vector notation: $f(\langle x_1, x_2, x_3 \rangle)$

Multivariate functions $f: [0, 1]^3 \to [0, 1]$

> This means that $f$ is a function of 3 arguments, which all have values in the interval [0, 1], and whose output is in the interval [0, 1].

### Arithmetic mean

- The arithmetic mean is the value obtained when we add up all out inputs and divide by how many there are.

For an input vector $x = \langle x_1, x_2,..., x_n \rangle$, The average mean is given by


$$AM(x) = \frac{1}{n} \sum_{i=1}^n x_i = \frac{(x_1 + x_2 + ... + x_n)}{n}$$

#### Properties

- Symmetry
    - The order of arugments doesn't affect the output.
    - $AM(x) = AM(x_\sigma)$, for all $\sigma$
    - $\sigma$ indicates a permutation of the input vector

- Translation invariance
    - If a constant is added to every input and we take the average, the output will change by that same constant
    - $AM(x_1+t, x_2+t, ..., x_n+t) = AM(x_1, x_2, ..., x_n) + t$

- Homogeneity
    - If you multiply every input by a constant, the output will change by the same factor
    - $AM(\lambda x_1, \lambda x_2, ..., \lambda x_n) = \lambda AM(x_1, x_2, ..., x_n)$

- Monotonicity
    - Increasing one of the inputs does not result in a decrease to the output
    - If $x \le y$ then $AM(x) \le AM(y)$ 

- Idempotency
    - If all the inputs are the same then the output will be the same too 

### Median

If our aim is to produce an output that is ‘representative’, the arithmetic mean’s susceptibility to skewed distributions and outliers has the practical implication that a single extreme value can distort the output. 
This is a standard criticism against any reporting methods that evaluate performance using the mean.
The median is one function that is used instead of the arithmetic mean in situations where we have reason to believe the data is not normally distributed or might contain ‘outliers’

> The median is the "middle" number after we arrange our data in order (either ascending or descending)


For an input vector $x = \langle x_1, x_2,..., x_n \rangle$

$$
Median(x) = 
\begin{cases} 
    x_{(k)}, & \text n=2k-1 \\
    \frac{x_{(k)}+x_{(k+1)}}{2}, & \text n=2k 
\end{cases}
$$

where k is a natural number, i.e. 1; 2; 3, etc., and the index notation $(.)$ means we first rearrange the data in order
Denoting even and odd with $n=2k, n=2k-1$

### Geometric and Harmonic means

The need for alternative measures of centre when there are outliers or skewed data is well known in introductory statistics courses, however there are other reasons why it sometimes just isn’t appropriate to use the arithmetic mean to find the ‘average’.

Consider the following scenarios:

- Your pay goes up by 20 % one year and 10 % the next. What is the average pay increase?
- Leia can paint a house in 3 h, Luke can paint one in 5 h. How long would it take them to paint two houses if they work together?

> Informal definition

For two or more arguments, the geometric mean is the value obtained when we multiply all of the inputs together and then take the n-th root (where n is how many values we have).


For an input vector $x = \langle x_1, x_2,..., x_n \rangle$, The geometric mean is given by

$$
GM(x) = (\prod_{i=1}^n x_i)^{1/n} = (x_1 x_2 ... x_n)^{1/n} = \sqrt[n]{x_1 \times x_2 \times ... \times x_n}
$$

> Note: Percentage need to be converted to decimal before running the equation. Example 10% increase in salary would mean 1.10

### Harmonic mean

#### Case

Leia can paint a house in 3 h, Luke can paint one in 5, and we want to know how long it would take them to paint two houses if they work together.

**Thinking Out Loud**

If we surmized that the average pace should be 1 house in 4 h (the arithmetic mean of 3 and 5), thinking things through we would see that in this time, Leia can paint $1 \frac{3}{1}$ houses, while Luke would have painted $\frac{4}{5}$. 

This would put us over two houses.The problem is that the rates are given to us in a somewhat reciprocal fashion. Usually we give rates with respect to time, so we have:

- Leia paints a house at the rate of $\frac{1}{3}$ of a house per hour.
- Luke paints at the rate of $\frac{1}{5}$ of a house per hour.

So put together they can paint $\frac{8}{15}$ houses for 2 houses it would be 3.75

> The key aspect in both situations is that the way we combine the numbers to find the ‘average’ is reciprocal to the way in which they are given or their units.

**Another example**

Let's say you have a car that travels a total distance of 200 km. For the first 100 km, the car travels at a speed of 100 km/h. For the next 100 km, the car travels at a speed of 80 km/h.

- Calculating Average Speed
    - First Segment (100 km at 100 km/h):
        - Speed = 100 km/h
        - Distance = 100 km
        - Time = Distance / Speed = 100 km / 100 km/h = 1 hour
    - Second Segment (100 km at 80 km/h):
        - Speed = 80 km/h
        - Distance = 100 km
        - Time = Distance / Speed = 100 km / 80 km/h = 1.25 hours
    - Total Time and Distance:
        - Total Distance = 100 km + 100 km = 200 km
        - Total Time = 1 hour + 1.25 hours = 2.25 hours
    - Average Speed:
        - Average Speed = Total Distance / Total Time
        - Average Speed = 200 km / 2.25 hours

For an input vector $x = \langle x_1, x_2,..., x_n \rangle$, The harmonic mean is given by


$$HM(x) = n \left (\sum_{i=1}^n \frac{1}{x_i} \right)^{-1} = \frac{n}{\frac{1}{x_1} + \frac{1}{x_2} + ... + \frac{1}{x_n}}$$


### Aggregation function

For an input vector $x = \langle x_1, x_2,..., x_n \rangle$, a multivariate function $A(x)$ can be referred to as an aggregation function if it:

- Is a monotone increasing (in either a strict or non-strict sense), i.e. an increase to any of the inputs cannot result in a decrease to the output. If $x \le y$ then $A(x) le A(y)$
- Satisfies the boundary counditions $A(a, a, ..., a) = a, A(b, b, ..., b) = b$ where $a$ and $b$ are the minimum and maximum values possible
- An aggregation function $A(x)$ is averaging if it's output is bounded between the minimum and maximum of its inputs, i.e. $min(x) \le A(x) \le max(x)$
> Example of averaging aggregation function - $f(x_1, x_2, x_3) = \frac{x_1}{2}(x_2 + x_3)$

> **Note:** A log transformation is useful in situations where there may be some infrequent but very high values.
For example, if we are looking at incomes at a large scale company, most of the employees will have low to moderate incomes, while the CEO might be earning millions. Similarly in ecology, there may be some very common
species, with populations substantially higher than the next most frequently occurring. By using a geometric mean, we limit the ability of a single very high value to bring up the entire average.

## Transforming data

### Multicriteria eval

| Student | Sprint | Height | Serving | Endurance | AM    |
| :------ | :----- | :----- | :------ | :-------- | :---- |
| Mizuho  | 15.78  | 148    | 94      | 17        | 68.55 |
| Yukie   | 21.15  | 147    | 94      | 20        | 70.43 |
| Megumi  | 14.30  | 134    | 91      | 17        | 64.36 |
| Sakura  | 19.59  | 174    | 88      | 16        | 74.50 |
| Izumi   | 10.96  | 145    | 93      | 16        | 66.37 |
| Yukiko  | 19.17  | 158    | 83      | 12        | 68.06 |
| Yumiko  | 18.35  | 157    | 99      | 20        | 73.44 |
| Kayoko  | 14.09  | 177    | 82      | 23        | 73.92 |

#### Which is better? Higher or Lower?

> Observations

- Mizuho is a lot slower than Izumi and has similar height, serving score and endurance, however her final rating is better because slower times add more points. Since we are talking about volleyball teams, we probably want to reward quicker sprinting times.

- If we look at the range of the heights, not only are the values higher, but Megumi and Kayoko differ by 43 cm.

- Individual variable distributions are different and so is the scale

- The majority of students performed their sprint in between 10 and 20 s, however we would think the difference between a 10 and 15 s runner is more significant than the difference between a 20 and 25 s runner.

### Negations and Utility transformations

Negation functions transform the data so that high values become low and low values become high.

If the values are given over the unit interval (between 0 and 1), then the standard negation is given by $N(t) = 1-t$

A strict negation $N$ defined over a real interval $[a, b]$ is a function that:
- Is monotone decreasing, i.e. If $x < y$ then $N(x) > N(y)$; and 
- Satisfies boundary conditions $N(a) = b$ and $N(b) = a$

> Note: $N(N(t)) = t$

### Scaling, Standardization and Normalization

#### Linear feature scaling


For a set of values $x_j = \langle x_{1,j}, x_{2,j},..., x_{m,j} \rangle$, relating to a single feature, we let $a = min(x_j)$ and $b=max(x_j)-min(x_j)$. Each $x_{i,j}$ in $x_j$ can be scaled so that it takes a new value $x_{i,j}^{'} = f(x_{i,j})$, where $f$ is the single-variate function 

$$
f(t) = \frac{t-a}{b} 
$$

> Other way of looking at this $x_{new} = \frac{x_{old} - x_{min}}{x_{max}-x_{min}}$

#### Standardization

Standardization usually refers to the process of subtracting the mean and dividing by the standard deviation for each value, which centers the data at zero and scales the standard deviation to 1. 
Applying this technique to different variables makes them commensurable, and we then essentially interpret them in terms of how abnormal they are.

$$
f(t) = \frac{t - AM(x_j)}{SD(x_j)}
$$

> where $f(t)$ is the new value $t$ is the old one

### Transoformation

For **Log transformation** have functions like $f(t) = ln(t)$

> Note: Knowing when to use a log transformation is not straight forward. In some statistical approaches when looking for relationships between variables, the aim is usually to use transformation functions like log and then check if the relationship appears linear.

For **The piece wise-linear transformations** we use will usually be monotone functions where the domain is split into intervals and a different linear function 

$$
f(t) = 
\begin{cases}
    q\frac{t-a}{b-a}, a \le t \lt b, \\
    q+(1-q)\frac{t-b}{c-b}, b \le t \le c
\end{cases}
$$

> $[a, c]$ domain is plit into two intervals in this case

The dual aggregation function uses a negation (usually the standard negation). First, all of the inputs are transformed using that negation, the data is aggregated, and then finally the negation is used on the result.

For an aggregation function $A$ with inputs given over the unit interval $[0, 1]$, it's dual $A^d$ using standard negation is given by:

$$
A^d(x_i) = 1 - A(1-x_1, 1-x_2, ..., 1-x_n) 
$$

For dual aggregation function on arithmetic mean

$$
AM^d(x_1, x2) = 1 - AM(1-x_1, 1-x_2) \\
= 1 - \frac{1}{2}(1-x_1 + 1-x_2) \\
= \frac{1}{2}(x_1 + x_2)
$$

> Dual function with arithmetic mean is a special case

Similarly for Geometric mean

$$
GM^d(x_1, x_2) = 1 - GM(1-x_1, 1-x_2) \\
= 1 - \sqrt{(1-x_1)(1-x_2)}
$$

#### Power mean

> The power mean is not just one function but a family of functions, with each member being determined by the value of $p$

$$
PM(x) = \left( \frac{1}{n} \sum_{i=1}^{n} x_i^p \right)^{\frac{1}{p}} \\
= \left( \frac{x_1^p + x_2^p + \ldots + x_n^p}{n} \right)^{\frac{1}{p}}
$$

**NOTE**

For $p=1$, we obtain the arithmetic mean, For $p=-1$, we have the harmonic mean, and for $p=0$ (a limiting case) we actually obtain the geometric mean. Furthermore, as $p \to -\infty$ we approach the minimum function and $p \to \infty$ we approach the maximum. 


#### Quasi-Arithmetic mean (Kolmogorov mean)

> Side note
> I don't see any applications of it with actual use case but a generlized equation

$$
M_f(x_1, x_2, \ldots, x_n) = f^{-1}\left(\frac{1}{n} \sum_{i=1}^n f(x_i)\right)
$$

> Arithmetic mean

$$
M_f(x_1, x_2, \ldots, x_n) = \frac{1}{n} \sum_{i=1}^n x_i
$$

> Geometric mean $f(x) = \log(x)$

$$
M_f(x_1, x_2, \ldots, x_n) = \exp\left(\frac{1}{n} \sum_{i=1}^n \log(x_i)\right)
$$

> Harmonic mean $f(x) = \frac{1}{x}$

$$
M_f(x_1, x_2, \ldots, x_n) = \left(\frac{1}{n} \sum_{i=1}^n \frac{1}{x_i}\right)^{-1}
$$

## Weighted averaging

### Weighing vector

An n-dimensional weighing vector $w = \langle w_1, w_2,..., w_n \rangle$, is a vector such that $w_i \ge 1$ for all $i$ and 

$$
\sum_{i=1}^n w_i = 1
$$

> There are some circumstances where we might have similar objects where the weights do not add to 1 but the term ‘weighting vector’ is still used. We can always normalize a vector of non-negative weights

### Weighted arithmetic mean

For an input $x = \langle x_1, x_2,..., x_n \rangle$, and the weighing vector $w$, The weighted arithmetic mean is given by

$$
\sum_{i=1}^n x_i w_i = w_1 x_1 + w_2 x_2 + ... + w_n x_n
$$

### Weighted power mean

For an input $x = \langle x_1, x_2,..., x_n \rangle$, and the weighing vector $w$, The weighted power mean is given by

$$
PM_{w,p}(x) = \left(\sum_{i=1}^n w_i x_i^p \right) ^ \frac{1}{p}
$$

> For the case of $p=0$ (Geometric mean)

$$
PM_{w,0}{x} = \prod_{i=1}^n x_i^{w_i} = x_1^{w_1} x_2^{w_2} \ldots {x_n}^{w_n}
$$

### Weighted median

> Note: $x$ is arranged in ascending order.
> There can be some ambiguous cases where we can name the previous one as lower-weighted median and upper weighted median

| w(x) | 0.20 | 0.10 | 0.06 | 0.24 | 0.08 | 0.32 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| x    | 0.03 | 0.10 | 0.27 | 0.45 | 0.46 | 0.78 |


Look at the cumulative sum of the weights from left to right and look to the point we have above 50%. This occurs at the w entry that is equal to 0.24, since $0.20 + 0.10 + 0.06 = 0.36$ and then $0.36 + 0.24 = 0.6 \gt 0.5$. So in this case the weighted median output will be 0.45 (the 4th highest input).


For an input $x = \langle x_1, x_2,..., x_n \rangle$, and the weighted vector $w$, The lower weighing median is given by:

> Inputs are reordered in a non-decreasing order, $x_(1) \le x_(2) \le \ldots \le x_(n)$ and $k$ is index such that

$$
Med_w(x) = x_k
$$

$$
\sum_{i=1}^{k-1} w_i \lt \frac{1}{2} \quad and \quad
\sum_{i=k+1}^{n} w_i \le \frac{1}{2}
$$

### Simpson's Dominance Index

> Applications in ecology and economics

$$
q_i = \frac{x_i}{\sum_{i=1}^n x_i}
$$

$$
Simp(q) = \sum_{i=1}^n q_i^2
$$

### Borda count

Here's an example of the Borda count:
Suppose we have a group of four voters and three candidates (A, B, and C). The voters' preferences are as follows:
- Voter 1: A > B > C
- Voter 2: B > C > A
- Voter 3: C > A > B
- Voter 4: A > C > B

To calculate the Borda count, we assign points to each candidate based on their ranking in each voter's preference:
- A: 3 points (1st choice) + 2 points (2nd choice) + 1 point (3rd choice) = 7 points
- B: 2 points (2nd choice) + 3 points (1st choice) + 1 point (3rd choice) = 6 points
- C: 1 point (3rd choice) + 2 points (2nd choice) + 3 points (1st choice) = 6 points

The candidate with the highest Borda count is A.

## Averaging with Interaction

> Some inputs will be correlated, which means that increases to one variable tend to co-occur with increases or decreases in another.

### Trimmed and Winsorized mean

> Use case

A diver receives the scores $x = \langle 9, 8.8, 9.6, 4.3, 7.6, 8 \rangle$, for one her dives. What'll be the trimmed mean?

After ordering - $x_{ord} = \langle 4.3, 7.6, 8, 8.8, 9, 9.6 \rangle$.

Remove the highest and lowest evaluations.

Final score is given after trimming both ends $AM(7.6, 8, 8.8, 9) = 8.35$
> Associated weighing vector $w=\langle 0, 1/4, 1/4, 1/4, 1/4, 0\rangle$

For Winsorized mean using the same vector we'll have $h=2$. Replace lowest inputs with next one and heighest input with preivous one. 

$$
AM(7.6, 7.6, 8, 8.8, 9, 9) = 8.3
$$

> Associated weighing vector $w=\langle 0, 2/n, 1/n, 1/n, 2/n, 0 \rangle$

### Ordered weighted average

To calculate the output for the OWA, we first re-order the inputs lowest to highest. We then multiply each input by its corresponding weight. 

### Orness

The concept of ‘orness’ has been introduced to characterize averaging functions (not just the OWA) in terms of how similar to the maximum function they are. 
The maximum has an orness degree of 1, while the minimum function has an orness degree of 0. The arithmetic mean, which treats high and low inputs equally has an orness of 0.5, while a function like the geometric mean, which is affected more by lower inputs, has an orness level of between 0 and 0.5.

We use the term ‘or- ness’ because in some extended logical frameworks, the ‘OR’ operation is modelled by the maximum, while the ‘AND’ operation is modelled by the minimum.


For a given weighing vector $w = \langle w_1, w_2,..., w_n \rangle$, the degree of orness of the associated OWA function is given by

$$
orness(w) = \sum_{i=1}^n w_i \frac{i-1}{n-1}
$$

So if $w = \langle 1, 0, ..., 0\rangle$, it is clear we have an $orness(w) = 0$

### Quantifier function

A quantifier function $Q$ is an increasing function satisfying $Q(0) = 0$ and $Q(1) = 1$. For a given $n$, the OWA weights associated with the quantifier can be calculated for each $i$ using

$$
w_i = Q \left(\frac{i}{n} \right) - Q \left(\frac{i-1}{n} \right) \\

i \to [1, n]
$$

For example, if our quantifier was $Q(t) = t^2$ and $n=4$, then $w = \langle 0, 0.25, 0.5, 0.75, 1 \rangle$


### Fuzzy measure

A fuzzy measure allocates a value between 0 and 1 to each of the subsets of ${1: n}$. To be considered a fuzzy measure, two conditions must be satisfied.

- Firstly, the value of the whole $set(\{1:n\})=1$ and value of empty set $\emptyset=0$

- Secondly, it must satisfy monotonicity with respect to adding new elements to a subset. 

For example, if $S$ is the set consisting of the three members, 1, 2 and 4, and T includes 1, 2, 4 as well but also includes 6, then $v(T)$ should not be less than $v(S)$.


Let $X = \{1, 2, 3\}$ be a set. Define a fuzzy measure $\mu$ on $X$ as follows:

$$
\begin{align*}
    \mu(\emptyset) &= 0 \\
    \mu(\{1\}) &= 0.2 \\
    \mu(\{2\}) &= 0.3 \\
    \mu(\{3\}) &= 0.4 \\
    \mu(\{1, 2\}) &= 0.5 \\
    \mu(\{1, 3\}) &= 0.6 \\
    \mu(\{2, 3\}) &= 0.7 \\
    \mu(\{1, 2, 3\}) &= 1
\end{align*}
$$

### Choquet integral

The Choquet integral is designed to work in contexts where the importance of different factors is not simply additive, as in the case of the Riemann integral.

Imagine you're evaluating cars based on various factors (like safety, cost, comfort). Unlike measuring something like distance, where adding up small distances always makes sense, the importance of each factor in car evaluation might change depending on what other factors are present.

For example, the importance of safety might be viewed differently depending on whether the car is comfortable or not. Or, you might value cost differently in a luxury car versus a budget car. The Choquet integral allows this kind of nuanced aggregation.

In many real-world scenarios, especially involving human preferences or subjective evaluations, the total value or importance isn't just a simple sum of individual parts. The Choquet integral captures this complexity.

$$
C_\mu(f) = \sum_{i=1}^{n} \left( f(x_{(i)}) - f(x_{(i-1)}) \right) \mu(A_{(i)})
$$

- $C_\mu(f)$: The Choquet integral of the function $f$ with respect to the fuzzy measure $\mu$.

- $n$: $n$ is the number of elements in the set over which $f$ is defined.

- $f(x_{(i)}) - f(x_{(i-1)})$: The difference in the function values at successive points. Here, $x_{(i)}$ is the $i^{th}$ largest value of the function $f$, and $x_{(i-1)}$ is the value just before it in this ordering. By convention, $f(x_{(0)})$ is often taken to be 0.

- $\mu(A_{(i)})$: The value of the fuzzy measure $\mu$ on the set $A_{(i)}$. The set $A_{(i)}$ typically consists of all elements in the domain of $f$ such that their function value is greater than or equal to $f(x_{(i)})$. As $i$ increases, $A_{(i)}$ generally represents sets with lower function values.

> couldn't find any proper applications or use cases in the internet. This is probably handled in some decision theory or game theory textbooks

### Shapley value

Shapley value from a fuzzy measure perspective provide a way to measure the importance of each criterion in terms of its contribution to the score of each group of criteria when using a fuzzy measure. This means we can have fuzzy measures that are very difficult to interpret in terms of the importance of variables and subsets of variables where $n$ is large.

> To be filled. These concepts depend on Game theory. The reason why some of these concepts were introduced need to be understood from a cooperative game theoreric's perspective. The book doesn't cover these in-depth. Just provides the equation.

> It is also not clear how to apply this and where this can be applied.

## Fitting aggregation functions to empirical data

> Empirical data: Empirical data refers to information that is gathered through observation or experimentation, and is based on what can be experienced or directly observed

### Distance

- Non-negativity: The distance between any two points is never negative.
- Identity of indiscernibles: The distance is zero if and only if the two points are the same.
- Symmetry: The distance from point A to point B is the same as the distance from point B to point A.
- Triangle inequality: The distance from point A to point C is less than or equal to the sum of the distances from point A to point B and from point B to point C.

$$
d: X \times X \to \mathbb{R} \text{ such that for all } x, y, z \in X, \text{ the following properties hold:} \\
\begin{align*}
&  d(x, y) \geq 0 \quad \text{(Non-negativity)} \\
&  d(x, y) = 0 \iff x = y \quad \text{(Identity of indiscernibles)} \\
&  d(x, y) = d(y, x) \quad \text{(Symmetry)} \\
&  d(x, z) \leq d(x, y) + d(y, z) \quad \text{(Triangle inequality)} \\
\end{align*}
$$

### Euclidean distance

$$
d(x, y) = \sqrt{\sum_{i=1}^{n} (x_i - y_i)^2}
$$

### Mikowski distance

> The Euclidean distance is a special case of the Minkowski distance where  $p=2$

$$
D(X,Y) = \left( \sum_{i=1}^{n} |x_i - y_i|^p \right)^{\frac{1}{p}}
$$

### Linear programming

> Linear programming or linear optimization, is a method to achieve the best outcome.

#### Sample use case

[video reference youtube](https://www.youtube.com/watch?v=Bzzqx1F23a8&t=474s)

$$
\text{Maximize} \quad Z = 20x + 30y \\

\begin{align}
\text{Subject to:} \\
& 5x + 4y \leq 40 \quad \text{(Labor Constraint)} \\
& 10x + 20y \leq 120 \quad \text{(Material Constraint)} \\
& x \geq 0, y \geq 0 \quad \text{(Non-Negativity Constraints)}

\end{align}
$$

```r
# Use the lpSolve package
lp(
    direction = "max",         # Maximizing the objective function
    objective.in = c(20, 30),  # Coefficients of the objective function
    const.mat = matrix(c(5, 10, 4, 20), nrow = 2, byrow = TRUE),  # Constraint coefficients
    const.dir = c("<=", "<="),  # Constraint directions
    const.rhs = c(40, 120),     # Right-hand side of constraints
    all.int = TRUE
)            
```

### Evaluating and interpreting model's accuracy

#### Sum of squared errors

$$
\text{SSE} = \sum_{i=1}^{n} (y_i - \hat{y}_i)^2
$$

#### Root mean squared error

$$
\text{RMSE} = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2}
$$

#### Sum of absolute errors

$$
\text{SAE} = \sum_{i=1}^{n} |y_i - \hat{y}_i|
$$

#### Pearson correlation

$$
r = \frac{\sum_{i=1}^{n} (X_i - \bar{X})(Y_i - \bar{Y})}{\sqrt{\sum_{i=1}^{n} (X_i - \bar{X})^2 \sum_{i=1}^{n} (Y_i - \bar{Y})^2}}
$$

> Where $\bar{X}$ and $\bar{Y}$ are arithmetic means of the dataset

#### Spearman's Rank correlation coefficient

[scipy reference](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.spearmanr.html)

> There isn't much data available on how to implement or use it in some data other than some pre-cooked R function that I can use to find the coefficient.

> There are other correlation coefficeints like Kendall, Somers', Goodman and Kruskals. Not sure where these are implemented. Much of the data is buried deep in academic literature.

Rank correlation is a statistical measure that assesses the degree of association between two variables using the ranks of their values rather than the actual values themselves.

**Spearman's rank correlation coefficient ($\rho$)**: This method works by ranking each value in each variable and then calculating the correlation coefficient based on these ranks. It assesses how well the relationship between two variables can be described using a monotonic function. If the ranks are identical for each pair, Spearman's rho is 1. If the ranks are perfectly opposite, Spearman's rho is -1.

### Learning weights for aggregation functions

The book talks about various equations and using Quasi-Arithmetic mean to fit the aggregation functions, i.e, finding the weights for those OWA kind of functions directly from data.
There is also equations given to find fuzzy measures from data using Mobius transforms in Choquet integral but these are not explained in detail in the book and couldn't find any data in internet as well.

I'm wondering is it because machine learning and probably other causal inference methods are taking over. Probably those techniques yield better results and more widely used?

### Are We Evaluating an Approach or the Model Itself?

We can look to build aggregation models from data for multiple reasons. On the one hand, we may have data where we know (or have an intuition) that one variable in some way summarizes the others or is dependent on the others, and we want to understand the nature of this relationship. 

For example: 
- Can movie box office takings be predicted from an aggregation of social media analytics? 
- Can tumors be classified as benign or malignant based on texture features from MRI scans? 
- Can we predict whether a potential employee will perform well in our company? 

In such cases, we might be more interested in the parameters of the model itself, i.e. which analytics are the most important and useful? 
- Which texture features or other indicators are most important when it comes to classifying tumors? 
- What are the characteristics when we are looking for new employees? 

We will want the functions to fit very closely in order to make reliable interpretations and we will want to ensure that we have enough data to safely say that the model is stable.