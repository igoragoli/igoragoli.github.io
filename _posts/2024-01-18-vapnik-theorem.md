---
layout: post
title:  "Demystifying Vapnik's Key Theorem"
math: true
categories: ml
---

I was reviewing some of my notes on the ML course at my uni and I came across Vapnik's Key Theorem. After some months without studying statistical learning theory, it took a while to wrap my head around it. As I couldn't find any simplified explanation for it on the internet, here is my attempt at it:

### Formalizing the Learning Problem

The learning problem can be described with three components:

1. A generator of random vectors $$x \in \mathcal{X}$$, drawn independently from a fixed but unknown distribution $$P(x)$$.
2. A supervisor that returns an output vector $$y \in \mathcal{Y}$$ for every input vector $$x$$, according to a conditional distribution function $$P(y ｜ x)$$, also fixed but unknown. If $$\mathcal{Y} = \mathbb{R}$$, we're talking about a regression problem. If $$\mathcal{Y}$$ is a finite set, it's a classification problem.
3. A learning machine capable of implementing a set of functions $$\mathcal{H} = \{f: \mathcal{X} \rightarrow \mathcal{Y}\}$$. We call $$\mathcal{H}$$ the *hypothesis space.*

*Learning* is finding $$f \in \mathcal{H}$$ that best predicts the supervisor's output. In *supervised learning*, we need to have a set of i.i.d. data points drawn from $$P(x, y)$$ with which we'll train our learning machine.

### Empirical Risk Minimization

We use the *risk functional* to quantify how good our choice of $$f$$ is.

$$ \mathcal{R}(f) = \int_{\mathcal{X} \times \mathcal{Y}} L(y, f(x))dP(x,y) = \mathrm{E}[L(Y, f(X))]$$

The smaller the risk, the better $$f$$ predicts the supervisor's output. Therefore, learning consists of minimizing the risk:

$$f_0 = \text{argmin}_{f \in \mathcal{H}} \mathcal{R}(f)$$

Unfortunately, $$P(x, y)$$ is unknown (if it was known, why would we even bother with machine learning?) and the risk cannot be analytically minimized. Instead, given a dataset $$\{(x_i, y_i)_{1 \le i \le n}\}$$, we minimize the *empirical risk*:

$$R_n(f) = \frac{1}{n} \sum_{i=1}^n L(y_i, f(x_i))$$

We call the *empirical risk minimization (ERM) principle* the principle of approximating $$f_0$$, a function that minimizes the risk $$\mathcal{R}$$, by $$f_n$$, a function that minimizes the empirical risk $$\mathcal{R_n}$$.

But wait, what guarantees us that a function $$f_n$$ that minimizes empirical risk **also minimizes the real risk**? In other words, what guarantees the consistency of the ERM principle?

This is where *Vapnik's Key Theorem of the learning theory* comes in. It specifies necessary and sufficient conditions for the ERM principle to be consistent.

### Vapnik's Key Theorem of the Learning Theory

First, it's useful to make our notation more general. Let $$z = (x, y)$$, $$\mathcal{Z = X \times Y}$$ and $$Q(z, f) = L(y, f(x))$$ . Now the joint distribution $$P(x, y)$$ is $$P(z)$$, the real risk is 

$$\mathcal{R(f)} = \int_{\mathcal{Z}} Q(z, f) dP(z)$$

and the empirical risk is

$$ \mathcal{R}_n(f) = \frac{1}{n} \sum_{i=1}^n Q(z_i, f) $$

Here's the theorem as in [1], with modifications to match our notation:

*The Key Theorem:* Let $$Q(z, f), f \in \mathcal{H}$$ be a set of functions that has a bounded loss for probability measure $$P(z)$$

$$ A \le \int_{\mathcal{Z}} Q(z, f) dP(z) \le B $$

Then for the ERM principle to be consistent it is necessary and sufficient that the empirical risk $$\mathcal{R}_n(f)$$ converge *uniformly* to the actual risk $$\mathcal{R}(f)$$ over the set $$Q(z, f), f \in \mathcal{H}$$ as follows:

$$ \lim_{n \rightarrow \infty} \text{Prob} \bigg\{ \text{sup}_{f \in \mathcal{H}} (\mathcal{R}(f) - \mathcal{R}_n(f)) > \varepsilon \bigg\} = 0, \forall \varepsilon $$

### Why Is It Important?

Vapnik notes that

> This theorem is called the Key theorem because it asserts that any analysis of the convergence properties of the ERM principle must be a *worst case analysis*. The necessary condition for consistency (not only the sufficient condition) depends on whether or not the deviation [of the risk] over the given set of functions converges in probability to zero.

In essence, Vapnik's Key Theorem provides us with a framework to prove that our learning machines can learn. It gives us mathematical confidence to deploy machine learning algorithms (that minimize empirical risk) to solve complex, real-world problems (whose optimal solutions minimize real risk).

In practice, quantifying risks and confirming a machine learning problem meets the uniform convergence condition require extra considerations. Although these aspects go beyond this post's scope, frameworks like PAC learnability are handy for addressing these practical issues.


# References

[1] V. N. Vapnik, "An overview of statistical learning theory," in _IEEE Transactions on Neural Networks_, vol. 10, no. 5, pp. 988-999, Sept. 1999, doi: 10.1109/72.788640.

[2] Personal notes from CentraleSupélec's ML course.