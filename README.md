<script type="text/x-mathjax-config">
MathJax.Hub.Config({
  CommonHTML: {
    scale: 130
  }
});
</script>
<script type="text/x-mathjax-config">
  MathJax.Hub.Config({
    tex2jax: {
      inlineMath: [ ['$','$'], ["\\(","\\)"] ],
      processEscapes: true
    }
  });
</script>
<script type="text/javascript" async
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>


## Introduction

Many problems in science and engineering can be formulated as a mathematical optimization problem in which an optimal solution is sought, either locally or globally.
The field of global optimization is the application of applied mathematics and numerical analysis towards finding the overall optimal solution in a set of candidate solutions.
Local optimization is considered an easier problem, in which it suffices to find an optimum which is optimal with respect to its immediate vicinity.
Such a local optimum is obviously a suboptimal solution and, while harder to find, global optima are more preferred.

Generally, optimization problems are formulated as finding the optimal solution which minimizes, respectively maximizes, a criterion, which is commonly referred to as the objective function.
Further constraints on the the set of solutions can be formulated, such that only a subset of solutions are permissible as candidates for the optimum.

Optimization is commonly done in an iterative manner where the objective function is evaluated for multiple candidate solutions.
Due to the iterative nature, it becomes desirable to evaluate this function as few times as possible over the course of the entire optimization, which becomes even more crucial when the evaluation of the objective function itself is costly.
Therefore, it would be advantageous to infer information about the objective function beyond the evaluations themselves, which only provide punctual information.

Bayesian inference models provide such advantages since they compute predictive distributions instead of punctual evaluations.
One class of Bayesian inference models are Gaussian processes (GP), which can be applied to model previous evaluations of the objective function as a multi-variate Gaussian distribution.
Given such a Gaussian distribution over the previous evaluations, information can be inferred over all candidate solutions in the feasible set at once.

## Gaussian Processes

In most situations where observations have many small independent components, their distribution tends towards the Gaussian distribution.
Compared to other probability distributions, the Gaussian distribution is tractable and it's parameters have intuitive meaning.
The theory of the central limit theorem (CLT) makes the Gaussian distribution a versatile distribution which is used in numerous situations in science and engineering.

A convenient property of the Gaussian distribution for a random variable $X$ is its complete characterization by its mean $$\mu$$ and variance $\Sigma$:

$$
\begin{align}
     \mu &= \mathbb{E}[X] \\
     \Sigma &= \mathbb{E}[(X-\mu)^T(X-\mu)]
\end{align}
$$

Mathematically, a multivariate Gaussian for a vector $x \in \mathbb{R}^d$ is defined by its mean $\mu \in \mathbb{R}^d$ and covariance function $\Sigma \in \mathbb{R}^{d \times d}$:

$$
\begin{align}
          \mathcal{N}(x | \mu, \Sigma) &=
               \frac{1}{\sqrt{(2 \pi)^d |\Sigma|^2}}
               \exp \left[
               -\frac{1}{2}
               (x-\mu)^T \Sigma^{-1}(x-\mu)
               \right] \\
               &\propto
               \exp \left[
               -\frac{1}{2}
               (x-\mu)^T \Sigma^{-1}(x-\mu)
               \right]
\end{align}
$$

A useful property of the Gaussian distribution is that its shape is determined by its mean and covariance in the exponential term.
This allows us to omitt the normalization constant and determine the relevant mean and covariance terms from the exponential term.

Let $y=f(x)$, where $x \in \mathbb{R}^d$ and $y \in \mathbb{R}$ be the function which we want to estimate with a Gaussian Process.
Furthermore, let $\mathcal{D} = (X, y)$, $ = \{ (x_i, y_i) \}_{i=0}^N $, with $X \in \mathbb{R}^{N \times d}$ and $y \in \mathbb{R}^{N}$, be our training observations of the function $f$.
Lastly, let $\smash{\mathcal{D}_* = (X_*, y_*) = \{ (\x_j, y_j) \}_{j=0}^{N_*}$, with $X_* \in \mathbb{R}^{N_* \times d}$ and $y_* \in \mathbb{R}^{N_*}$, be the test observations at which we want to compute the predictive distributions of $y_* =f(X_s)$ for the function $f$.

A Gaussian process is defined as a stochastic process, such that every finite collection of realizations $X=\{ x_i \}_{i=0}^N, x_i \in \mathbb{R}^d$ of the random variables $X \sim \mathcal{N}( \cdot  |  \mu, \Sigma), X \in \mathbb{R}^d$ is a multivariate distribution.
A constraint of Gaussian processes as they are used in machine learning, which can be relaxed in specific cases, is that they are assumed to have a zero mean.
In order to compute a predictive distribution over $y_*$ we initially construct the joint distribution over the training observations $\mathcal{D} = (X,y)$ and test observations $\mathcal{D}_* = (X_*,y_*)$:
\begin{align}
     p(y_*, y, X_*, X) &= \frac{1}{\sqrt{(2 \pi)^{N+N_*} |\K|^2}}
     \exp \left[
     -\frac{1}{2}
     \begin{bmatrix}
          y \\
          y_*
     \end{bmatrix}^T
     \begin{bmatrix}
          K_{\X\X} & K_{\X\Xs} \\
          K_{\Xs\X} & K_{\Xs\Xs}
     \end{bmatrix}^{-1}
     \begin{bmatrix}
          \y \\
          \ys
     \end{bmatrix}
     \right] \\
     &\propto
     \exp \left[
     -\frac{1}{2}
     \begin{bmatrix}
          \y \\
          \ys
     \end{bmatrix}^T
     \begin{bmatrix}
          K_{\X\X} & K_{\X\Xs} \\
          K_{\Xs\X} & K_{\Xs\Xs}
     \end{bmatrix}^{-1}
     \begin{bmatrix}
          \y \\
          \ys
     \end{bmatrix}
     \right] \\
     &\propto
     \mathcal{N}
     \left(
     \begin{bmatrix}
          \y \\
          \ys
     \end{bmatrix} \middle|
     \mathbf{0}, \K
     \right)
\end{align}
