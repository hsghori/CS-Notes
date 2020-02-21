---
title: Machine Learning Notes
author: Haroon Ghori
geometry: margin=1in
---

# Supervised Learning

**Supervised learning** is a class of machine learning algorithms which uses a set of known examples to
come up with a mechanism to predict something about unknown examples. A supervised learning algorithm is
trained using a pre-labeled training set and uses that knowledge to make predictions about data that wasn't
in the training set.

Basic definitions:
- **Classification** is the process of taking some input $X$ and mapping it to some discrete label - for example
  identifying a disease based on a chest x ray image or MRI.
- **Regression** is the process of mapping some input $X$ to a real number value - for example predicting home
  prices.
- **Instances** are vectors of values that define the input space of a problem.
- **Concept** is an idealized function that maps from the input space to an output.
- **Target concept** is the concept that our algorithm is trying to achieve.
- **Hypothesis class** is all of the functions were are willing to consider as a target concept.
- **Sample** (aka training set) is a set of all inputs paired with a correct output.
- **Candidate** is a concept that might be the target concept
- **Testing set** is a set of inputs paired with a correct output which is not included in the training set.
- **Cross validation** is a method of training a machine learning model which attempts to get the best performing,
  general model. We hold out some of the training set as a stand in for the test data to determine how well our
  algorithm performs on unseen data. The general process is to split the data into K folds. Then we train $K - 1$
  folds and use the extra fold for validation. We can use different combinations of $K - 1$ folds to train
  different versions of the model, tune hyperparameters, etc.

## Decision Trees

A **decision tree** is a very simple type of machine learning algorithm which is used to classify samples.

```
                 O
            /         \
          O            O
        / | \        / | \
      [] []  []     [] [] []
```

Each `O` node is a decision point which splits the samples based on a specific feature. The `[]` nodes represent
classifications based on the samples which those particular feature sets. Consider the tree:

```
                 O
            /         \          f0  left x = 0, right x = 1
          O            O
        / | \        / | \       f1  left x = 0, mid x = 1, right x = 2
     [1] [2] [3]  [4] [5] [6]
```

Then `[1]` would have all samples with features `[0, 0, ...]`, `[4]` would have all samples with features
`[1, 0, ...]`, etc. The number of levels of a decision tree do not need to match the number of features in the
samples - the input `[0, 2, 1, 3, 4]` would be at node `[3]` in the decision tree above. Each of those nodes would
then correspond to a class - so classifying a sample is as simple as finding the matching leaf in the decision tree
and returning the associated class.

### Constructing Decision Trees (ID3 Algorithm)

Decision trees are very useful classification tools, but they need to be constructed carefully because they can be
very space inefficient. A "naive" decision trees for an N-feature boolean problem could take $O(2^{2^N})$ space which is computationally infeasible.

The general algorithm behind constructing a decision tree given a testing set is known as **ID3**. The basic
idea is for a particular set of sample data:
1. Select the *best* attribute.
2. Split the samples by the attribute.
3. Repeat for the sample subsets until the sample subset is "pure" (ie all the samples in the subsets are of the
   same class) or for some predetermined set of iterations.

Or in code:

```Python
def id3(node, samples):
    if is_pure(samples):
        # if all the samples have the same label
        node.label = get_label(samples)
        return node
    elif should_stop(node):
        # if an external stopping criteria is met
        node.label = get_majority_label(samples)
        return node

    # get the most informative attribute based on the samples
    attr = best_attribute(samples)
    node.decision_attr = attr
    for val in possible_values(attr):
        # for each value of the attribute create a new child node
        node.children.append(Node(), get_samples_where(samples, attr=attr, val=val))
    return node
```

Determining the "best" attribute is a bit tricky and there are a few ways to determine it. Generally speaking,
the "best" attribute is defined as the attribute which is the most "informative" or which allows us
to split the samples into "more pure" groups.

Note that this algorithm has a `should_stop` check which terminates the algorithm based on some external condition
and sets the leaf node to the most common label in the samples at that node. This stopping criterian is most
commonly based on a maximum depth parameter on the tree.

**Information gain** is a means of determining the most informative feature and is defined as:

$$gain(S, A) = entropy(S) - \sum_{v \in values(A)}{\frac{|S_v|}{|S|} entropy(S_{v})}$$

where **entropy** is a function which defines the randomness of a sample set.

$$entropy(S) = -\sum_{v \in values(S)}{p(v)\log{p(v)}}$$

Note here that the expression $p(v)\log{p(v)}$ gets more negative as $p(v) \to 0$ since $\lim_{x \to 0}{\log(x)} =
-\infty$. So a large number of possible values with a small probability of each value would result in a large
overall entropy value (since each item in the sum would be large and negative). But a small set of items where
the probabilities are more consolidated would result in a small entropy value. In other words, low entropy means
the samples are more "pure" whereas high entropy means that the samples are more intermixed.

![Graph of $plog(p)$](src/ml/entropy.png)

So $gain(S, A)$ essentially gives us the difference between the current entropy of the system and the expected
entropy after splitting over the value $A$. So the "best" feature is defined as the one where splitting resulted in
the highest information gain (since we want the expected entropy to be low).

ID3 generally has a bias towards generating trees with
- "Good" splits near the root
- Reasonably correct classifications (of the training data)
- Shorter trees (which is a product of the good splits near the root)

If the attributes are continuous we have to do some modifications - there are a lot of different ways to handle
continuous attributes for example defining ranges of values. For discrete valued attributes, an individual attribute
should only appear once in a path to a leaf. For continuous valued attributes, an individual attribute can appear
multiple times on a path to a leaf if the split criteria is different.

It is very easy for decision trees to overfit to training data - in decision trees overfitting happens when we
create too complex of a tree based on the training data. We can avoid this in a few ways.
- Use cross validation to create the trees by generating a bunch of trees on different "folds" of training data and
  selecting the tree that performed best on the "validation set".
- Check the tree against a validation set before every new expansion and stop the tree construction once the
  validation accuracy has hit a specific target.
- Prune the tree after creation.

We can also use decision trees for regression problems though we need to handle the leaves in a different way -
perhaps by using the average of the values at the leaves. We also may need to redefined our sense of purity to
include values that are similar to each other as opposed to strict equality.

## Linear Regression

In classification problems we're trying to group data into distinct buckets based on features. However, in a
regression problem we are more trying to come up with a continuous function which can map the features of a sample
to the output value.

Linear regression, is a method of regression where we try to fit a linear function (in 2 or more dimensions) to
a set of points.

![](src/ml/linear_regression.png)

We can actually solve this problem analytically using calculus by defining an **error function** which describes
the error between our ideal function and the sample points, then finding the parameters of the ideal function
which minimizes that error function.

So given $error(W, X, Y)$ we want to find $w' = W$ which minimizes $error$ for a fixed set of samples $X, Y$.
A very common error function is known as the **sum of least squares error function** which is defined as:

$$E(W) = \sum_{i}^{N}{(y_{i} - f(W, x_i))^2}$$

where $f(W, x_i)$ is the function we want to come up with. In linear regression, $f(W, x_i)$ is a linear function
of the form $f(x_i) = W^T x_i$. where
$W$ and $x_i$ are of length $M$ (so $W^T x_i$ is really the dot product between $W$ and $x_i$)

$$E(W) = \sum_{i}^{N}{(y_{i} - W^T x_i)^2}$$
$$\frac{\delta E}{\delta W} = \sum_{i}^{N}{2 x_i^{T} (y_{i} - W^{T} x_i)} = 0$$
$$\sum_{i}^{N}{x_i^{T} y_i} = W \sum_{i}^{N}x_i^{T}  x_i$$

Now this requires some linear algebra knowledge. We can simplify this expression by operating over the entire
matrix at once. Note that if $A$ is $N \times M$ and $B$ is $M \times 1$ then:

$$\sum{a^T b} = A^T B$$

So our equation becomes:

$$X^{T}Y = X^{T}XW$$
$$W = (X^{T}X)^{-1}X^{T}Y$$

which is quite easy for a computer to solve.

Our fit function doesn't necessarily need to be a linear function though. We can choose any kind of function as
the $f(x_i)$ and find the optimal parameters by solving for the set of parameters $W$. This can be generally done
by taking our training data, transforming it into some matrix $X$ - this can just be the data or can be some
function of the data like $x_i = [c_i, x_{i}, x_{i}^2]$ (for one dimensional samples).

## Logistic Regression

## Perceptrons

## Support Vector Machines

## Neural Networks and Deep Learning

# Randomized Optimization

**Randomized optimization** (RO) is the process of using randomized algorithms to find the maximum of a fitness
function. This is particularly important if your fitness function cannot be solved analytically (ie if it doesn't
have a derivative or the derivative is difficult to find).

Given an input space $X$ and a fitness function $f(z)$ we want to find $x'$ where

$$x' = argmax_{x \in X}{(f(x))}$$

This is a relatively simple and intuitive idea across many disciplines. We may need to do this kind of optimization
to tune parameters for chemical or biological processes, find routes, find functional solutions or roots, tune
hyperparameters for machine learning algorithms, tune weights for machine learning algorithms, etc.

## Hill climbing

**Hill climbing** is one of hte simplest RO algorithms.
1. Pick a random point $x \in X$.
2. Select two points within a "neighborhood" around $x$ - $N(x)$.
3. Let $x' = argmax_{z \in N(x)}{(f(z))}$
4. If $f(x') <= f(x)$, terminate. Otherwise set $x = x'$ and repeat from step 2.

This algorithm is very simple but has some clear drawbacks - specifically it is very easy for this algorithm to
get caught in local maxima.

**Random restart hill climbing** is a slight modification of hte standard hill climbing algorithm which attempts to
make hill climbing more robust against local maxima.
1. For i in range(N)
   1. Let x = the result of hill climbing
   2. If x > the best maximum currently found, update the best maximum

This algorithm is more robust against local maxima since we perform hill climbing from multiple places throughout
the input space. While obviously we cannot guarantee that we will hit the global maximum, we can be more certain
that the maximum we find after performing a series of random hill climbings is, at worst, a high local maximum.

## Simulated Annealing

**Simulated annealing** is an RO algorithm which attempts to solve the local maxima problem by allowing the
algorithm to "explore" sub-optimal paths in the hopes of finding a global maximum.

1. For a finite set of iterations
2. Sample a point $x' = N(x)$
3. Jump to the new point $x'$ based on a probability function $P(x', x, T)$
4. Decrease the "temperature" $T$

The probability function is piecewise:
$$P(x', x', T) =
\begin{cases}
    1&\ \text{if } f(x') \geq f(x)\\
    e^{\frac{f(x') - f(x)}{T}}&
\end{cases}
$$

This means that if the newly sampled point $x'$ is better than the old point, we should always jump to it. However,
if $x'$ is worse than the old point we should jump based on $P(x', x, T) = e^{\frac{f(x') - f(x)}{T}}$. This
function has a couple of very interesting properties.
- The numerator means that $P(x', x, T) \to 1$ as $f(x') \to f(x)$. This means that the probability of jumping is
  higher if the new point is "not so bad" (ie not much worse than the old point).
- The denominator means that $P(x', x, T) \to 0$ as $T \to 0$. This means that as we keep iterating (and $T$ keeps
  decreasing), the probability of jumping to bad points decreases. So as $T \to \infty$ simulated annealing acts as
  a random walk, and as $T \to 0$ it behaves like hill climbing. In our algorithm generally want to decrease $T$
  slowly so that we can let the algorithm find the best paths.

Simulated annealing has the interesting property that the probability of the simulatd annealing algorithm ending at
some point $z$ is based on the fitness function:

$$P(ending \ at \ z) = \frac{e^{f(z)}{T}}{Z_T}$$

This is known as the **Boltzman distribution**.

## Genetic Algorithms

**Genetic algorithms** are algorithms that incorporate techniques analogous to biological reproduction and evolution
to perform a local search. Genetic algorithms generally assume a multidimensional input space (for example n-bit
strings, vectors, etc).


1. Define a population $P$ from the input space.
2. Repeat until convergence:
   1. Compute $f(x) \forall x \in P$.
   2. Select the "most fit" samples from the population. This can be done by taking the samples with the best
      fitness scores, using a weight function to select samples etc.
   3. Replace the "less" fit examples by
      - "pairing up" samples from the more fit pool and performing **crossovers**.
      - performing **mutations** on the more fit or crossover samples.

**Crossover** in this context means combining the features of two samples to create two new samples. There are
many different techniques that can be used to do crossover.

Consider the samples `01101100` and `11010111`.

A **one point crossover** takes one point (or position) as a dividing point and swaps halves of the samples. This
kind of crossover assumes a structural locality to the data - it assumes that there is some value to maintaining
some of the original structure.

In our samples above if we picked position 4,

```
0110 | 1100
1101 | 0111

01100111
11011100
```

A **uniform crossover** considers each point in the samples individually and uses a uniform probability
distribution to determine whether to "keep" or "swap" the values at any position. This assumes that there is no
structural locality to the data.

## MIMIC

The randomized algorithms discussed above are great and finding individual points. However, they don't convey any
kind of structure and have unclear probability distributions.

**MIMIC** is an algorithm that attempts to solve these issues with the generic RO algorithms by refining a specific
probability function.

$$P(x) =
\begin{cases}
    \frac{1}{Z_{\theta}}&\ \text{if } f(x) > \theta\\
    0&
\end{cases}
$$

$\theta$ is a parameter that we modulate as part of the MIMIC algorithm. When $\theta = y_{min}$ - the
smallest possible value of $f(x)$, then $P(x)$ is essentially a uniform distribution across all the points in the
input space. When $\theta = y_max$ - the largest possible value of $f(x)$, then $P(x)$ is essentially a
distribution across all the optimal points in the input space. So the goal of MIMIC is to essentially start with a
low value of $\theta$ and refine the probability distribution to find the optimal points. This process helps convey
structure (as opposed to returning a single point) and has a more clear probability distribution than the other RO
algorithms.

1. Repeat until convergence
   1. Generate samples from $P^{\theta_{i}}(x)$
   2. Set $\theta_{i+1}$ based on the most fit samples and retain only those samples
   3. Estimate $P^{\theta_{i+1}}(x)$

THe tricky part here is how we estimate and sample from $P^{\theta}$ (which is an approximation of the $P(x)$
function above). To estimate $P^{\theta}$ we need to take all of the samples we currently have, find the mutual
information between each pair of features in the samples, put
that into a fully connected graph, and find the Maximum Spanning Tree of that graph. THat creates a "dependency
tree" where the probability of a given sample is given by $p(x) = \Pi_{i}^{N} {(p(x_i | \pi(x_i)))}$ where $\Pi$ is
a product and $\pi$ is the parents of a specific feature in the tree.

MIMIC does very well with structure, but it can get stuck in local optima (though it's not common). Additionally,
MIMIC takes much more time per iteration than other algorithms (like Simulated Annealing and Genetic Algorithms).
However, MIMIC gives a lot more information per iteration than other algorithms.

# Probabilistic Models

## Baysean Networks

## Markov Random Fields

## Markov Models

# Expectimation Maximization Algorithms and Clustering

## K Means Clustering

## Gaussian Mixture Model

## EM Algorithms
