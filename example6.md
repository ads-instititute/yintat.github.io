---
author:
- 'Dmitriy Drusvyatskiy[^1]'
bibliography: 'bibliography.bib'
title: The proximal point method revisited
---

Introduction
============

The proximal point method is a conceptually simple algorithm for
minimizing a function $$f$$ on $${\mathbb R}^d$$. Given an iterate $$x_t$$,
the method defines $$x_{t+1}$$ to be any minimizer of the proximal
subproblem

$$
\argmin_{x}~\left\{f(x)+\tfrac{1}{2\nu}\|x-x_t\|^2\right\},
$$ 

for an appropriately chosen parameter $$\nu>0$$. At first glance, each proximal
subproblem seems no easier than minimizing $$f$$ in the first place. On
the contrary, the addition of the quadratic penalty term often
regularizes the proximal subproblems and makes them well conditioned.
Case in point, the subproblem may become convex despite $$f$$ not being
convex; and even if $$f$$ were convex, the subproblem has a larger strong
convexity parameter thereby facilitating faster numerical methods.

Despite the improved conditioning, each proximal subproblem still
requires invoking an iterative solver. For this reason, the proximal
point method has predominantly been thought of as a
theoretical/conceptual algorithm, only guiding algorithm design and
analysis rather than being implemented directly. One good example is the
proximal bundle method (Lemarechal, Strodiot, and Bihain 1981), which
approximates each proximal subproblem by a cutting plane model. In the
past few years, this viewpoint has undergone a major revision. In a
variety of circumstances, the proximal point method (or a close variant)
with a judicious choice of the control parameter $$\nu>0$$ and an
appropriate iterative method for the subproblems can lead to practical
and theoretically sound numerical methods. In this blog, I will briefly
describe three recent examples of this trend:

-   a subgradient method for weakly convex stochastic approximation
    problems (Davis and Grimmer 2017),

-   the prox-linear algorithm for minimizing compositions of convex
    functions and smooth maps (Drusvyatskiy and Lewis 2016; Drusvyatskiy
    and Paquette 2016; Burke and Ferris 1995; Nesterov 2007; Lewis and
    Wright 2015; Cartis, Gould, and Toint 2011),

-   Catalyst generic acceleration schema (Lin, Mairal, and
    Harchaoui 2015) for regularized Empirical Risk Minimization.

Each section, discussing the examples above, is self-contained and can
be read independently of the others. A version of this blog series will
appear in SIAG/OPT Views and News 2018.

Notation
========

The following two constructions will play a basic role in the blog. For
any closed function $$f$$ on $${\mathbb R}^d$$, the *Moreau envelope* and
the *proximal map* are 

$$
\begin{aligned}
f_{\nu}(z)&:=\inf_{x}~\left\{f(x)+\tfrac{1}{2\nu}\|x-z\|^2\right\},\\
{\rm prox}_{\nu f}(z)&:=\argmin_{x}~\left\{f(x)+\tfrac{1}{2\nu}\|x-z\|^2\right\},
\end{aligned}
$$

respectively. In this notation, the proximal point method is simply the
fixed-point recurrence on the proximal map:[^2]

$$
{\bf Step\, }t: \qquad \textrm{choose }x_{t+1}\in {\rm prox}_{\nu f}(x_t).
$$

Clearly, in order to have any hope of solving the proximal subproblems,
one must ensure that they are convex. Consequently, the class of weakly
convex functions forms the natural setting for the proximal point
method.

 A function $$f$$ is called *$$\rho$$-weakly convex* if the assignment
$$x\mapsto f(x)+\frac{\rho}{2}\|x\|^2$$ is a convex function.

For example, a $$C^1$$-smooth function with $$\rho$$-Lipschitz gradient is
$$\rho$$-weakly convex, while a $$C^2$$-smooth function $$f$$ is $$\rho$$-weakly
convex precisely when the minimal eigenvalue of its Hessian is uniformly
bounded below by $$-\rho$$. In essence, weak convexity precludes functions
that have downward kinks. For instance, $$f(x):=-\|x\|$$ is not weakly
convex since no addition of a quadratic makes the resulting function
convex.

Whenever $$f$$ is $$\rho$$-weakly convex and the proximal parameter $$\nu$$
satisfies $$\nu<\rho^{-1}$$, each proximal subproblem is itself convex and
therefore globally tractable. Moreover, in this setting, the Moreau
envelope is $$C^1$$-smooth with the gradient 

$$\label{eqn:grad_form}
\nabla f_{\nu}(x)=\nu^{-1}(x-{\rm prox}_{\nu f}(x)).$$

Rearranging the
gradient formula yields the useful interpretation of the proximal point
method as gradient descent on the Moreau envelope

$$
x_{t+1}=x_t-\nu\nabla f_{\nu}(x_t).
$$

In summary, the Moreau envelope $$f_{\nu}$$ serves as a $$C^1$$-smooth
approximation of $$f$$ for all small $$\nu$$. Moreover, the two conditions

$$
\|\nabla f_{\nu}(x_{t})\|< \varepsilon
$$ 

and

$$
\|\nu^{-1}(x_t-x_{t+1})\|<\varepsilon,
$$ 

are equivalent for the
proximal point sequence $$\{x_t\}$$. Hence, the step-size
$$\|x_t-x_{t+1}\|$$ of the proximal point method serves as a convenient
termination criteria.

Examples of weakly convex functions
-----------------------------------

Weakly convex functions are widespread in applications and are typically
easy to recognize. One common source of weakly convex functions is the
composite problem class: 

$$\label{eqn:comp}
\min_{x}~ F(x):=g(x)+h(c(x)),
$$

where
$$g\colon {\mathbb R}^d\to{\mathbb R}\cup\{+\infty\}$$ is a closed convex
function, $$h\colon{\mathbb R}^m\to{\mathbb R}$$ is convex and
$$L$$-Lipschitz, and $$c\colon{\mathbb R}^d\to{\mathbb R}^m$$ is a
$$C^1$$-smooth map with $$\beta$$-Lipschitz gradient. An easy argument shows
that $$F$$ is $$L\beta$$-weakly convex. This is a worst case estimate. In
concrete circumstances, the composite function $$F$$ may have a much more
favorable weak convexity constant (e.g., phase retrieval (Duchi and Ruan
2017a Section 3.2)).

[\[exa:add\_comp\]]{#exa:add_comp label="exa:add_comp"}

The most prevalent example is additive composite minimization. In this
case, the map $$c$$ maps to the real line and $$h$$ is the identity
function: 

$$
\label{eqn:add_comp}
\min_{x}~ c(x)+g(x).
$$ 
        
Such problems appear often in statistical
learning and imaging. A variety of specialized algorithms are available;
see for example Beck and Teboulle (Beck and Teboulle 2012) or Nesterov
(Nesterov 2013).

[\[exa:nls\]]{#exa:nls label="exa:nls"}

The composite problem class also captures nonlinear least squares
problems with bound constraints: 

$$
\begin{aligned}
        \min_x~ \|c(x)\|_2\quad \textrm{subject to}\quad l_i\leq x_i\leq u_i ~\forall i.
\end{aligned}
$$ 

Such problems pervade engineering and scientific
applications.

[\[exa:ep\]]{#exa:ep label="exa:ep"} Consider a nonlinear optimization
problem: 

$$
\begin{aligned}
        \min_x~ \{f(x): G(x)\in \mathcal{K}\},
\end{aligned}
$$ 
        
        where $$f$$ and $$G$$ are smooth maps and
$$\mathcal{K}$$ is a closed convex cone. An accompanying *penalty
formulation* -- ubiquitous in nonlinear optimization -- takes the form

$$
\min_x~ f(x)+\lambda \cdot {\rm dist}_{\mathcal{K}}(G(x)),
$$ 

where
$${\rm dist}_{\mathcal{K}}(\cdot)$$ is the distance to $$\mathcal{K}$$ in
some norm. Historically, exact penalty formulations served as the early
motivation for the class
[\[eqn:comp\]](#eqn:comp){reference-type="eqref" reference="eqn:comp"}.

 Phase retrieval is a common computational problem, with applications in
diverse areas, such as imaging, X-ray crystallography, and speech
processing. For simplicity, I will focus on the version of the problem
over the reals. The (real) phase retrieval problem seeks to determine a
point $$x$$ satisfying the magnitude conditions,

$$
|\langle a_i,x\rangle|\approx b_i\quad \textrm{for }i=1,\ldots,m,
$$

where $$a_i\in {\mathbb R}^d$$ and $$b_i\in{\mathbb R}$$ are given. Whenever
there are gross outliers in the measurements $$b_i$$, the following robust
formulation of the problem is appealing (Eldar and Mendelson 2014; Duchi
and Ruan 2017a; Davis, Drusvyatskiy, and Paquette 2017):

$$
\min_x ~\tfrac{1}{m}\sum_{i=1}^m |\langle a_i,x\rangle^2-b_i^2|.
$$

Clearly, this is an instance of
[\[eqn:comp\]](#eqn:comp){reference-type="eqref" reference="eqn:comp"}.
For some recent perspectives on phase retrieval, see the survey (Luke
2017). There are numerous recent nonconvex approaches to phase
retrieval, which rely on alternate problem formulations; e.g., (Candès,
Li, and Soltanolkotabi 2015; Chen and Candès 2017; Sun, Qu, and Wright
2017).

 In robust principal component analysis, one seeks to identify sparse
corruptions of a low-rank matrix (Candès et al. 2011; Chandrasekaran et
al. 2011). One typical example is image deconvolution, where the
low-rank structure models the background of an image while the sparse
corruption models the foreground. Formally, given a $$m\times n$$ matrix
$$M$$, the goal is to find a decomposition $$M=L+S$$, where $$L$$ is low-rank
and $$S$$ is sparse. A common formulation of the problem reads:

$$
\min_{U\in {\mathbb R}^{m\times r},V\in {\mathbb R}^{n\times r}}~ \|UV^T-M\|_1,
$$

where $$r$$ is the target rank.

A synchronization problem over a graph is to estimate group elements
$$g_1,\ldots, g_n$$ from pairwise products $$g_ig_j^{-1}$$ over a set of
edges $$ij\in E$$. For a list of application of such problem see
(Bandeira, Boumal, and Voroninski 2016; Singer 2011; Abbe et al. 2014),
and references therein. A simple instance is $$\mathbb{Z}_2$$
synchronization, corresponding to the group on two elements $$\{-1,+1\}$$.
The popular problem of detecting communities in a network, within the
Binary Stochastic Block Model (SBM), can be modeled using $$\mathbb{Z}_2$$
synchronization.

Formally, given a partially observed matrix $$M$$, the goal is to recover
a vector $$\theta\in \{\pm 1\}^d$$, satisfying
$$M_{ij}\approx \theta_i \theta_j$$ for all $$ij\in E$$. When the entries of
$$M$$ are corrupted by adversarial sign flips, one can postulate the
following formulation

$$
\min_{\theta\in {\mathbb R}^{d}}~ \|P_{E}(\theta\theta^T-M)\|_1,
$$

where the operator $$P_E$$ records the entries indexed by the edge set
$$E$$. Clearly, this is again an instance of
[\[eqn:comp\]](#eqn:comp){reference-type="eqref" reference="eqn:comp"}.

The proximally guided subgradient method
========================================

As the first example of contemporary applications of the proximal point
method, consider the problem of minimizing the expectation:[^3]

$$
\min_{x\in {\mathbb R}^d}~ F(x)=\mathbb{E}_{\zeta} f(x,\zeta).
$$ 

Here,
$$\zeta$$ is a random variable, and the only access to $$F$$ is by sampling
$$\zeta$$. It is difficult to overstate the importance of this problem
class (often called *stochastic approximation*) in large-scale
optimization; see e.g. (Bottou and Bousquet 2008; Bartlett, Jordan, and
McAuliffe 2006).

When the problem is convex, the stochastic subgradient method (Polyak
and Juditsky 1992; Robbins and Monro 1951; Nemirovski et al. 2008) has
strong theoretical guarantees and is often the method of choice. In
contrast, when applied to nonsmooth and nonconvex problems, the behavior
of the method is poorly understood. The recent paper (Davis and Grimmer
2017) shows how to use the proximal point method to guide the
subgradient iterates in this broader setting, with rigorous guarantees.

Henceforth, assume that the function $$x\mapsto f(x,\zeta)$$ is
$$\rho$$-weakly convex and $$L$$-Lipschitz for each $$\zeta$$. Davis and
Grimmer (Davis and Grimmer 2017) proposed the scheme outlined below.

#### Proximally guided stochastic subgradient method

-   **Data**: $$x_0\in {\mathbb R}^d$$, $$\{j_t\}\subset\mathbb{N}$$,
    $$\{\alpha_j\}\subset{\mathbb R}_{++}$$

-   **For** $$t=0,\ldots,T$$ **do**

    -   Set $$y_0=x_t$$

    -   **For** $$j=0,\ldots,j_t-2$$ **do**

        -   Sample $$\zeta$$ and choose
            $$v_j\in\partial (f(\cdot,\zeta)+\rho\|\cdot-x_t\|^2)(y_j)$$

        -   $$y_{j+1}= y_j-\alpha_jv_j$$

    -   $$x_{t+1}= \frac{1}{j_t}\sum_{j=0}^{j_t-1}y_j$$

The method proceeds by applying a proximal point method with each
subproblem approximately solved by a stochastic subgradient method. The
intuition is that each proximal subproblem is $$\rho/2$$-strongly convex
and therefore according to well-known results (e.g. (Lacoste-Julien,
Schmidt, and Bach 2012; Rakhlin, Shamir, and Sridharan 2012; Hazan and
Kale 2011; Juditsky and Nesterov 2014)), the stochastic subgradient
method should converge at the rate $$O(\frac{1}{T})$$ on the subproblem,
in expectation. This intuition is not quite correct because the
objective function of the subproblem is not globally Lipschitz -- a key
assumption for the $$O(\frac{1}{T})$$ rate. Nonetheless, the authors show
that warm-starting the subgradient method for each proximal subproblem
with the current proximal iterate corrects this issue, yielding a
favorable guarantees (Davis and Grimmer 2017 Theorem 1).

To describe the rate of convergence, set
$$j_t=t+\lceil 648\log(648)\rceil$$ and $$\alpha_j=\tfrac{2}{\rho(j+49)}$$
in the Proximally guided stochastic subgradient method. Then the scheme
will generate an iterate $$x$$ satisfying

$$
\mathbb{E}_{\zeta}[\|\nabla F_{2\rho}(x)\|^2]\leq \varepsilon
$$ 

after
at most

$$
O\left(\frac{\rho^2(F(x_0)-\inf  F)^2}{\varepsilon^2}+\frac{L^4 \log^{4}(\varepsilon^{-1})}{\varepsilon^2}\right)
$$

subgradient evaluations. This rate agrees with analogous guarantees for
stochastic gradient methods for smooth nonconvex functions (Ghadimi and
Lan 2013). It is also worth noting that convex constraints on $$x$$ can be
easily incorporated into the Proximally guided stochastic subgradient
method by introducing a nearest-point projection in the definition of
$$y_{j+1}$$.

The prox-linear algorithm
=========================

For well-structured weakly convex problems, one can hope for faster
numerical methods than the subgradient scheme. In this section, I will
focus on the composite problem class
[\[eqn:comp\]](#eqn:comp){reference-type="eqref" reference="eqn:comp"}.
To simplify the exposition, I will assume $$L=1$$, which can always be
arranged by rescaling.

Since composite functions are weakly convex, one could apply the
proximal point method directly, while setting the parameter
$$\nu\leq\beta^{-1}$$. Even though the proximal subproblems are strongly
convex, they are not in a form that is most amenable to convex
optimization techniques. Indeed, most convex optimization algorithms are
designed for minimizing a sum of a convex function and a composition of
a convex function with a *linear* map. This observation suggests
introducing the following modification to the proximal-point algorithm.
Given a current iterate $$x_t$$, the *prox-linear method* sets

$$
\begin{aligned}
x_{t+1}=\argmin_x \{F(x;x_t)+\tfrac{\beta}{2}\|x-x_t\|^2\},
\end{aligned}
$$

where $$F(x;y)$$ is the local convex model

$$
F(x;y):=g(x)+h\left(c(y)+\nabla c(y)(x-y)\right).
$$ 

In other words,
each proximal subproblem is approximated by linearizing the smooth map
$$c$$ at the current iterate $$x_t$$.

The main advantage is that each subproblem is now a sum of a strongly
convex function and a composition of a Lipschitz convex function with a
linear map. A variety of methods utilizing this structure can be
formally applied; e.g. smoothing (Nesterov 2005), saddle-point
(Nemirovski 2004; Chambolle and Pock 2011), and interior point
algorithms (Nesterov and Nemirovskii 1994; Wright 1997). Which of these
methods is practical depends on the specifics of the problem, such as
the size and the cost of vector-matrix multiplications.

It is instructive to note that in the simplest setting of additive
composite problems
(Example [\[exa:add\_comp\]](#exa:add_comp){reference-type="ref"
reference="exa:add_comp"}), the prox-linear method reduces to the
popular proximal-gradient algorithm or ISTA (Beck and Teboulle 2012).
For nonlinear least squares, the prox-linear method is a close variant
of Gauss-Newton.

Recall that the step-size of the proximal point method provides a
convenient stopping criteria, since it directly relates to the gradient
of the Moreau envelope -- a smooth approximation of the objective
function. Is there such an interpretation for the prox-linear method?
This question is central, since termination criteria is not only used to
stop the method but also to judge its efficiency and to compare against
competing methods.

The answer is yes. Even though one can not evaluate the gradient
$$\|\nabla F_{\frac{1}{2\beta}}\|$$ directly, the scaled step-size of the
prox-linear method 

$$
\mathcal{G}(x):=\beta(x_{t+1}-x_t)
$$ 

is a good
surrogate (Drusvyatskiy and Paquette 2016 Theorem 4.5):

$$
\tfrac{1}{4} \|\nabla F_{\frac{1}{2\beta}}(x)\| \leq \|\mathcal{G}(x)\|\leq 3\|\nabla F_{\frac{1}{2\beta}}(x)\|.
$$

In particular, the prox-linear method will find a point $$x$$ satisfying
$$\|\nabla F_{\frac{1}{2\beta}}(x)\|^2\leq\varepsilon$$ after at most
$$O\left(\frac{\beta(F(x_0)-\inf F)}{\varepsilon}\right)$$ iterations. In
the simplest setting when $$g=0$$ and $$h(t)=t$$, this rate reduces to the
well-known convergence guarantee of gradient descent, which is black-box
optimal for $$C^1$$-smooth nonconvex optimization (Carmon et al. 2017b).

It is worthwhile to note that a number of improvements to the basic
prox-linear method were recently proposed. The authors of (Cartis,
Gould, and Toint 2011) discuss trust region variants and their
complexity guarantees, while (Duchi and Ruan 2017b) propose stochastic
extensions of the scheme and prove almost sure convergence. The paper
(Drusvyatskiy and Paquette 2016) discusses overall complexity guarantees
when the convex subproblems can only be solved by first-order methods,
and proposes an inertial variant of the scheme whose convergence
guarantees automatically adapt to the near-convexity of the problem.

Local rapid convergence
-----------------------

Under typical regularity conditions, the prox-linear method exhibits the
same types of rapid convergence guarantees as the proximal point method.
I will illustrate with two intuitive and widely used regularity
conditions, yielding local linear and quadratic convergence,
respectively.

 A local minimizer $$\bar x$$ of $$F$$ is *$$\alpha$$-tilt-stable* if there
exists $$r>0$$ such that the solution map

$$
M: v\mapsto \argmin_{x\in B_r(\bar x)} \left\{ F(x)-\langle v,x \rangle\right\}
$$

is $$1/\alpha$$-Lipschitz around $$0$$ with $$M(0)=\bar x$$.

This condition might seem unfamiliar to convex optimization specialist.
Though not obvious, tilt-stability is equivalent to a uniform quadratic
growth property and a subtle localization of strong convexity of $$F$$.
See (Drusvyatskiy and Lewis 2013) or (Drusvyatskiy, Mordukhovich, and
Nghia 2014) for more details on these equivalences. Under the
tilt-stability assumption, the prox-linear method initialized
sufficiently close to $$\bar x$$ produces iterates that converge at a
linear rate $$1-\alpha/\beta$$.

The second regularity condition models sharp growth of the function
around the minimizer. Let $$S$$ be the set of all stationary points of
$$F$$, meaning $$x$$ lies in $$S$$ if and only if the directional derivative
$$F'(x;v)$$ is nonnegative in every direction $$v\in {\mathbb R}^d$$.

 A local minimizer $$\bar x$$ of $$F$$ is *sharp* if there exists $$\alpha>0$$
and a neighborhood $$\mathcal{X}$$ of $$\bar x$$ such that

$$
F(x)\geq  F({\rm proj}_S(x))+c\cdot {\rm dist}(x,S)\qquad\forall x\in \mathcal{X}.
$$

Under the sharpness condition, the prox-linear method initialized
sufficiently close to $$\bar x$$ produces iterates that converge
quadratically.

For well-structured problems, one can hope to justify the two regularity
conditions above under statistical assumptions. The recent work of Duchi
and Ruan on the phase retrieval problem (Duchi and Ruan 2017a) is an
interesting recent example. Under mild statistical assumptions on the
data generating mechanism, sharpness is assured with high probability.
Therefore the prox-linear method (and even subgradient methods (Davis,
Drusvyatskiy, and Paquette 2017)) converge rapidly, when initialized
within a constant relative distance of an optimal solution.

Catalyst acceleration
=====================

The final example concerns inertial acceleration in convex optimization.
Setting the groundwork, consider a $$\mu$$-strongly convex function $$f$$
with a $$\beta$$-Lipschitz gradient map $$x\mapsto \nabla f(x)$$.
Classically, gradient descent will find a point $$x$$ satisfying
$$f(x)-\min f<\varepsilon$$ after at most

$$
O\left(\frac{\beta}{\mu}\ln(1/\varepsilon)\right)
$$ 

iterations.
Accelerated gradient methods, beginning with Nesterov (Nesterov 1983),
equip the gradient descent method with an inertial correction. Such
methods have the much lower complexity guarantee

$$
O\left(\sqrt{\frac{\beta}{\mu}}\ln(1/\varepsilon)\right),
$$ 

which is
optimal within the first-order oracle model of computation (Nemirovsky
and Yudin 1983).

It is natural to ask which other methods, aside from gradient descent,
can be "accelerated". For example, one may wish to accelerate coordinate
descent or so-called variance reduced methods for finite sum problems; I
will comment on the latter problem class shortly.

One appealing strategy relies on the proximal point method. Güler in
(Güler 1992) showed that the proximal point method itself can be
equipped with inertial steps leading to improved convergence guarantees.
Building on this work, Lin, Mairal, and Harchaoui (Lin, Mairal, and
Harchaoui 2015) explained how to derive the *total* complexity
guarantees for an inexact accelerated proximal point method that take
into account the cost of applying an arbitrary linearly convergent
algorithm $$\mathcal{M}$$ to the subproblems. Their *Catalyst
acceleration* framework is summarized below.

#### Catalyst Acceleration

-   **Data**: $$x_0\in {\mathbb R}^d$$, $$\kappa>0$$, algorithm
    $$\mathcal{M}$$

-   Set $$q= \mu/(\mu+\kappa)$$, $$\alpha_0=\sqrt{q}$$, and $$y_0=x_0$$

-   **For** $$t=0,\ldots,T$$ **do**

    -   Use $$\mathcal{M}$$ to approximately solve:

        $$
        \label{eqn:prox_subprob}
                        x_t\approx\argmin_{x\in {\mathbb R}^d} \left\{F(x)+\frac{\kappa}{2}\|x-y_{t-1}\|^2\right\}.\;
        $$

    -   Compute $$\alpha_t\in (0,1)$$ from the equation
        
        $$
        \alpha_t^2=(1-\alpha_t)\alpha_{t-1}^2+q\alpha_t.\;
        $$

    -   Compute: 

        $$
        \begin{aligned}
                    \beta_t&=\frac{\alpha_{t-1}(1-\alpha_{t-1})}{\alpha_{t-1}^2+\alpha_t},\\
                    y_t&=x_t+\beta_t(x_t-x_{t-1}). 
        \end{aligned}
        $$

To state the guarantees of this method, suppose that $$\mathcal{M}$$
converges on the proximal subproblem in function value at a linear rate
$$1-\tau\in (0,1)$$. Then a simple termination policy on the subproblems
[\[eqn:prox\_subprob\]](#eqn:prox_subprob){reference-type="eqref"
reference="eqn:prox_subprob"} yields an algorithm with overall
complexity 

$$
\label{eqn:compl}
\widetilde{O}\left(\frac{\sqrt{\mu+\kappa}}{\tau \sqrt{\mu}}\ln(1/\varepsilon)\right).
$$

That is, the expression
[\[eqn:compl\]](#eqn:compl){reference-type="eqref"
reference="eqn:compl"} describes the maximal number of iterations of
$$\mathcal{M}$$ used by the Catalyst algorithm until it finds a point $$x$$
satisfying $$f(x)-\inf f\leq \varepsilon$$. Typically $$\tau$$ depends on
$$\kappa$$; therefore the best choice of $$\kappa$$ is the one that
minimizes the ratio $$\frac{\sqrt{\mu+\kappa}}{\tau \sqrt{\mu}}$$.

The main motivation for the Catalyst framework, and its most potent
application, is the regularized Empirical Risk Minimization (ERM)
problem:

$$
\min_{x\in {\mathbb R}^d} f(x):=\frac{1}{m}\sum_{i=1}^m f_i(x)+g(x).
$$

Such large-finite sum problems are ubiquitous in machine learning and
high-dimensional statistics, where each function $$f_i$$ typically models
a misfit between predicted and observed data while $$g$$ promotes some low
dimensional structure on $$x$$, such as sparsity or low-rank.

Assume that $$f$$ is $$\mu$$-strongly convex and each individual $$f_i$$ is
$$C^1$$-smooth with $$\beta$$-Lipschitz gradient. Since $$m$$ is assumed to be
huge, the complexity of numerical methods is best measured in terms of
the total number of individual gradient evaluations $$\nabla f_i$$. In
particular, fast gradient methods have the worst-case complexity

$$
O\left(m\sqrt{\frac{\beta}{\mu}}\ln(1/\varepsilon)\right),
$$ 

since
each iteration requires evaluation of all the individual gradients
$$\{\nabla f_i(x)\}_{i=1}^m$$. Variance reduced algorithms, such as SAG
(Schmidt, Roux, and Bach 2013), SAGA (Defazio, Bach, and Lacoste-Julien
2014), SDCA (Shalev-Shwartz and Zhang 2012), SMART (Davis 2016), SVRG
(Johnson and Zhang 2013; Xiao and Zhang 2014), FINITO (Defazio, Domke,
and Caetano 2014), and MISO (Mairal 2015; Lin, Mairal, and Harchaoui
2015), aim to improve the dependence on $$m$$. In their raw form, all of
these methods exhibit a similar complexity

$$
O\left(\left(m+\frac{\beta}{\mu}\right)\ln(1/\varepsilon)\right),
$$

in
expectation, and differ only in storage requirements and in whether one
needs to know explicitly the strong convexity constant.

It was a long standing open question to determine if the dependence on
$$\beta/\mu$$ can be improved. This is not quite possible in full
generality, and instead one should expect a rate of the form

$$
O\left(\left(m+\sqrt{m\frac{\beta}{\mu}}\right)\ln(1/\varepsilon)\right).
$$

Indeed, such a rate would be optimal in an appropriate oracle model of
complexity (Woodworth and Srebro 2016; Arjevani 2017; Agarwal and Bottou
2015; Lan 2015). Thus acceleration for ERM problems is only beneficial
in the setting $$m< \beta/\mu$$.

Early examples for specific algorithms are the accelerated SDCA
(Shalev-Shwartz and Zhang 2015), APPA (Frostig et al. 2015), and RPDG
(Lan 2015).[^4] The accelerated SDCA and APPA, in particular, use a
specialized proximal-point construction.[^5] Catalyst generic
acceleration allows to accelerate all of the variance reduced methods
above in a single conceptually transparent framework. It is worth noting
that the first direct accelerated variance reduced methods for ERM
problems were recently proposed in (Allen-Zhu 2016; Defazio 2016).

In contrast to the convex setting, the role of inertia for nonconvex
problems is not nearly as well understood. In particular, gradient
descent is black-box optimal for $$C^1$$-smooth nonconvex minimization
(Carmon et al. 2017b), and therefore inertia can not help in the worst
case. On the other hand, the recent paper (Carmon et al. 2017a) presents
a first-order method for minimizing $$C^2$$ and $$C^3$$ smooth functions
that is provably faster than gradient descent. At its core, their
algorithm also combines inertia with the proximal point method. For a
partial extension of the Catalyst framework to weakly convex problems,
see (Paquette et al. 2017).

Conclusion
==========

The proximal point method has long been ingrained in the foundations of
optimization. Recent progress in large scale computing has shown that
the proximal point method is not only conceptual, but can guide
methodology. Though direct methods are usually preferable, proximally
guided algorithms can be equally effective and often lead to more easily
interpretable numerical methods. In this blog, I outlined three examples
of this viewpoint, where the proximal-point method guides both the
design and analysis of numerical methods.

The author thanks Damek Davis, John Duchi, and Zaid Harchaoui for their
helpful comments on an early draft.

\bibliographystyle{plain}
::: {#refs .references}
::: {#ref-abbe_band}
Abbe, E., A.S. Bandeira, A. Bracher, and A. Singer. 2014. "Decoding
Binary Node Labels from Censored Edge Measurements: Phase Transition and
Efficient Recovery." *IEEE Trans. Network Sci. Eng.* 1 (1):10--22.
<https://doi.org/10.1109/TNSE.2014.2368716>.
:::

::: {#ref-AgB}
Agarwal, A., and L. Bottou. 2015. "A Lower Bound for the Optimization of
Finite Sums." In *Proceedings of the 32nd International Conference on
Machine Learning, ICML 2015, Lille, France, 6-11 July 2015*, 78--86.
<http://leon.bottou.org/papers/agarwal-bottou-2015>.
:::

::: {#ref-accsvrg}
Allen-Zhu, Z. 2016. "Katyusha: The First Direct Acceleration of
Stochastic Gradient Methods." *Preprint arXiv:1603.05953 (Version 5)*.
:::

::: {#ref-yosi}
Arjevani, Y. 2017. "Limitations on Variance-Reduction and Acceleration
Schemes for Finite Sums Optimization." In *Advances in Neural
Information Processing Systems 30*, edited by I. Guyon, U. V. Luxburg,
S. Bengio, H. Wallach, R. Fergus, S. Vishwanathan, and R. Garnett,
3543--52. Curran Associates, Inc.
<http://papers.nips.cc/paper/6945-limitations-on-variance-reduction-and-acceleration-schemes-for-finite-sums-optimization.pdf>.
:::

::: {#ref-ban_boum}
Bandeira, A.S., N. Boumal, and V. Voroninski. 2016. "On the Low-Rank
Approach for Semidefinite Programs Arising in Synchronization and
Community Detection." In *Proceedings of the 29th Conference on Learning
Theory, COLT 2016, New York, Usa, June 23-26, 2016*, 361--82.
<http://jmlr.org/proceedings/papers/v49/bandeira16.html>.
:::

::: {#ref-jordan}
Bartlett, P.L., M.I. Jordan, and J.D. McAuliffe. 2006. "Convexity,
Classification, and Risk Bounds." *J. Amer. Statist. Assoc.* 101
(473):138--56.
<https://doi-org.offcampus.lib.washington.edu/10.1198/016214505000000907>.
:::

::: {#ref-smoothing_beckT}
Beck, A., and M. Teboulle. 2012. "Smoothing and First Order Methods: A
Unified Framework." *SIAM J. Optim.* 22 (2):557--80.
<https://doi.org/10.1137/100818327>.
:::

::: {#ref-BB}
Bottou, L., and O. Bousquet. 2008. "The Tradeoffs of Large Scale
Learning." In *Advances in Neural Information Processing Systems*,
161--68. <http://leon.bottou.org/publications/pdf/nips-2007.pdf>.
:::

::: {#ref-quad_conv}
Burke, J.V., and M.C. Ferris. 1995. "A Gauss-Newton Method for Convex
Composite Optimization." *Math. Programming* 71 (2, Ser. A):179--94.
<https://doi.org/10.1007/BF01585997>.
:::

::: {#ref-rob_cand}
Candès, E.J., X. Li, Y. Ma, and J. Wright. 2011. "Robust Principal
Component Analysis?" *J. ACM* 58 (3):Art. 11, 37.
<https://doi.org/10.1145/1970392.1970395>.
:::

::: {#ref-wirt_flow}
Candès, E.J., X. Li, and M. Soltanolkotabi. 2015. "Phase Retrieval via
Wirtinger Flow: Theory and Algorithms." *IEEE Trans. Inform. Theory* 61
(4):1985--2007. <https://doi.org/10.1109/TIT.2015.2399924>.
:::

::: {#ref-pmlr-v70-carmon17a}
Carmon, Y., J.C. Duchi, O. Hinder, and A. Sidford. 2017a. "'Convex Until
Proven Guilty': Dimension-Free Acceleration of Gradient Descent on
Non-Convex Functions." In *Proceedings of the 34th International
Conference on Machine Learning*, 70:654--63.
:::

::: {#ref-grad_desc_opt}
---------. 2017b. "Lower Bounds for Finding Stationary Points I."
*Preprint arXiv:1710.11606*.
:::

::: {#ref-composite_cart}
Cartis, C., N.I.M. Gould, and P.L. Toint. 2011. "On the Evaluation
Complexity of Composite Function Minimization with Applications to
Nonconvex Nonlinear Programming." *SIAM J. Optim.* 21 (4):1721--39.
<https://doi.org/10.1137/11082381X>.
:::

::: {#ref-cp}
Chambolle, A., and T. Pock. 2011. "A First-Order Primal-Dual Algorithm
for Convex Problems with Applications to Imaging." *J. Math. Imaging
Vision* 40 (1):120--45. <https://doi.org/10.1007/s10851-010-0251-1>.
:::

::: {#ref-chand}
Chandrasekaran, V., S. Sanghavi, P. A. Parrilo, and A.S. Willsky. 2011.
"Rank-Sparsity Incoherence for Matrix Decomposition." *SIAM J. Optim.*
21 (2):572--96. <https://doi.org/10.1137/090761793>.
:::

::: {#ref-rand_quad}
Chen, Y., and E.J. Candès. 2017. "Solving Random Quadratic Systems of
Equations Is Nearly as Easy as Solving Linear Systems." *Comm. Pure
Appl. Math.* 70 (5):822--83.
<https://doi-org.offcampus.lib.washington.edu/10.1002/cpa.21638>.
:::

::: {#ref-smart_davis}
Davis, D. 2016. "SMART: The Stochastic Monotone Aggregated Root-Finding
Algorithm." *Preprint arXiv:1601.00698*.
:::

::: {#ref-proj_weak_dim}
Davis, D., D. Drusvyatskiy, and C. Paquette. 2017. "The Nonsmooth
Landscape of Phase Retrieval." *Preprint arXiv:1711.03247*.
:::

::: {#ref-prox_guide_subgrad}
Davis, D., and B. Grimmer. 2017. "Proximally Guided Stochastic
Sbgradient Method for Nonsmooth, Nonconvex Problems." *Preprint,
arXiv:1707.03505*.
:::

::: {#ref-NIPS2016_6154}
Defazio, A. 2016. "A Simple Practical Accelerated Method for Finite
Sums." In *Advances in Neural Information Processing Systems 29*, edited
by D. D. Lee, M. Sugiyama, U. V. Luxburg, I. Guyon, and R. Garnett,
676--84. Curran Associates, Inc.
<http://papers.nips.cc/paper/6154-a-simple-practical-accelerated-method-for-finite-sums.pdf>.
:::

::: {#ref-SAGA2}
Defazio, A., F. Bach, and S. Lacoste-Julien. 2014. "SAGA: A Fast
Incremental Gradient Method with Support for Non-Strongly Convex
Composite Objectives." In *Advances in Neural Information Processing
Systems 27*, edited by Z. Ghahramani, M. Welling, C. Cortes, N. D.
Lawrence, and K. Q. Weinberger, 1646--54. Curran Associates, Inc.
:::

::: {#ref-finito}
Defazio, A., J. Domke, and T.S. Caetano. 2014. "Finito: A Faster,
Permutable Incremental Gradient Method for Big Data Problems." In
*ICML*, 1125--33.
:::

::: {#ref-tilt_adrian}
Drusvyatskiy, D., and A.S. Lewis. 2013. "Tilt Stability, Uniform
Quadratic Growth, and Strong Metric Regularity of the Subdifferential."
*SIAM J. Optim.* 23 (1):256--67. <https://doi.org/10.1137/120876551>.
:::

::: {#ref-prox_error}
---------. 2016. "Error Bounds, Quadratic Growth, and Linear Convergence
of Proximal Methods." *To Appear in Math. Oper. Res., arXiv:1602.06661*.
:::

::: {#ref-Dima_Ng}
Drusvyatskiy, D., B.S. Mordukhovich, and T.T.A. Nghia. 2014.
"Second-Order Growth, Tilt-Stability, and Metric Regularity of the
Subdifferential." *J. Convex Anal.* 21 (4):1165--92.
:::

::: {#ref-prox_lin_paq}
Drusvyatskiy, D., and C. Paquette. 2016. "Efficiency of Minimizing
Compositions of Convex Functions and Smooth Maps." *Preprint,
arXiv:1605.00125*.
:::

::: {#ref-duchi_ruan_PR}
Duchi, J.C., and F. Ruan. 2017a. "Solving (Most) of a Set of Quadratic
Equalities: Composite Optimization for Robust Phase Retrieval."
*Preprint arXiv:1705.02356*.
:::

::: {#ref-duchi_ruan}
---------. 2017b. "Stochastic Methods for Composite Optimization
Problems." *Preprint arXiv:1703.08570*.
:::

::: {#ref-eM}
Eldar, Y.C., and S. Mendelson. 2014. "Phase Retrieval: Stability and
Recovery Guarantees." *Appl. Comput. Harmon. Anal.* 36 (3):473--94.
<https://doi.org/10.1016/j.acha.2013.08.003>.
:::

::: {#ref-frostig}
Frostig, R., R. Ge, S.M. Kakade, and A. Sidford. 2015. "Un-Regularizing:
Approximate Proximal Point and Faster Stochastic Algorithms for
Empirical Risk Minimization." In *Proceedings of the 32nd International
Conference on Machine Learning (Icml)*.
:::

::: {#ref-gl_stoch}
Ghadimi, S., and G. Lan. 2013. "Stochastic First- and Zeroth-Order
Methods for Nonconvex Stochastic Programming." *SIAM J. Optim.* 23
(4):2341--68. <https://doi.org/10.1137/120880811>.
:::

::: {#ref-gul_prox_acc}
Güler, O. 1992. "New Proximal Point Algorithms for Convex Minimization."
*SIAM J. Optim.* 2 (4):649--64.
<https://doi-org.offcampus.lib.washington.edu/10.1137/0802032>.
:::

::: {#ref-hazan_subgrad}
Hazan, E., and S. Kale. 2011. "Beyond the Regret Minimization Barrier:
An Optimal Algorithm for Stochastic Strongly-Convex Optimization." In
*Proceedings of the 24th Annual Conference on Learning Theory*, edited
by Sham M. Kakade and Ulrike von Luxburg, 19:421--36. Proceedings of
Machine Learning Research. Budapest, Hungary: PMLR.
:::

::: {#ref-svrg}
Johnson, R., and T. Zhang. 2013. "Accelerating Stochastic Gradient
Descent Using Predictive Variance Reduction." In *Proceedings of the
26th International Conference on Neural Information Processing Systems*,
315--23. NIPS'13. USA: Curran Associates Inc.
<http://dl.acm.org/citation.cfm?id=2999611.2999647>.
:::

::: {#ref-MR3353214}
Juditsky, A., and Y. Nesterov. 2014. "Deterministic and Stochastic
Primal-Dual Subgradient Algorithms for Uniformly Convex Minimization."
*Stoch. Syst.* 4 (1):44--80.
<https://doi-org.offcampus.lib.washington.edu/10.1214/10-SSY010>.
:::

::: {#ref-a_simp_app}
Lacoste-Julien, S., M. Schmidt, and F. Bach. 2012. "A Simpler Approach
to Obtaining an $${O}(1/t)$$ Convergence Rate for the Projected Stochastic
Subgradient Method." *Arxiv arXiv:1212.2002*.
:::

::: {#ref-conjugategradient}
Lan, G. 2015. "An Optimal Randomized Incremental Gradient Method."
*arXiv:1507.02000*.
:::

::: {#ref-bundle}
Lemarechal, C., J.-J. Strodiot, and A. Bihain. 1981. "On a Bundle
Algorithm for Nonsmooth Optimization." In *Nonlinear Programming, 4
(Madison, Wis., 1980)*, 245--82. Academic Press, New York-London.
:::

::: {#ref-prox}
Lewis, A.S., and S.J. Wright. 2015. "A Proximal Method for Composite
Minimization." *Math. Program.* Springer Berlin Heidelberg, 1--46.
<https://doi.org/10.1007/s10107-015-0943-9>.
:::

::: {#ref-catalyst}
Lin, H., J. Mairal, and Z. Harchaoui. 2015. "A Universal Catalyst for
First-Order Optimization." In *Advances in Neural Information Processing
Systems*, 3366--74.
:::

::: {#ref-luke_news_views}
Luke, R. 2017. "Phase Retrieval, What's New?" *SIAG/OPT Views and News*
25 (1).
:::

::: {#ref-miso}
Mairal, J. 2015. "Incremental Majorization-Minimization Optimization
with Application to Large-Scale Machine Learning." *SIAM Journal on
Optimization* 25 (2):829--55.
:::

::: {#ref-mprox}
Nemirovski, A. 2004. "Prox-Method with Rate of Convergence $$O(1/t)$$ for
Variational Inequalities with Lipschitz Continuous Monotone Operators
and Smooth Convex-Concave Saddle Point Problems." *SIAM J. Optim.* 15
(1):229--51. <https://doi.org/10.1137/S1052623403425629>.
:::

::: {#ref-latest_subgrad}
Nemirovski, A., A. Juditsky, G. Lan, and A. Shapiro. 2008. "Robust
Stochastic Approximation Approach to Stochastic Programming." *SIAM J.
Optim.* 19 (4):1574--1609.
<https://doi-org.offcampus.lib.washington.edu/10.1137/070704277>.
:::

::: {#ref-complexity}
Nemirovsky, A.S., and D.B. Yudin. 1983. *Problem Complexity and Method
Efficiency in Optimization*. A Wiley-Interscience Publication. John
Wiley & Sons, Inc., New York.
:::

::: {#ref-nes_nem}
Nesterov, Y., and A. Nemirovskii. 1994. *Interior-Point Polynomial
Algorithms in Convex Programming*. Vol. 13. SIAM Studies in Applied
Mathematics. Society for Industrial; Applied Mathematics (SIAM),
Philadelphia, PA. <https://doi.org/10.1137/1.9781611970791>.
:::

::: {#ref-nest_orig}
Nesterov, Yu. 1983. "A Method for Solving the Convex Programming Problem
with Convergence Rate $$O(1/k^{2})$$." *Dokl. Akad. Nauk SSSR* 269
(3):543--47.
:::

::: {#ref-smooth_min_nonsmooth}
---------. 2005. "Smooth Minimization of Non-Smooth Functions." *Math.
Program.* 103 (1, Ser. A):127--52.
<https://doi.org/10.1007/s10107-004-0552-5>.
:::

::: {#ref-nest_GN}
---------. 2007. "Modified Gauss-Newton Scheme with Worst Case
Guarantees for Global Performance." *Optim. Methods Softw.* 22
(3):469--83. <https://doi.org/10.1080/08927020600643812>.
:::

::: {#ref-nest_conv_comp}
---------. 2013. "Gradient Methods for Minimizing Composite Functions."
*Math. Program.* 140 (1, Ser. B):125--61.
<https://doi.org/10.1007/s10107-012-0629-5>.
:::

::: {#ref-catalyst_2}
Paquette, C., H. Lin, D. Drusvyatskiy, J. Mairal, and Z. Harchaoui.
2017. "Catalyst Acceleration for Gradient-Based Non-Convex
Optimization." *Preprint arXiv:1703.10993*.
:::

::: {#ref-stochave}
Polyak, B.T., and A.B. Juditsky. 1992. "Acceleration of Stochastic
Approximation by Averaging." *SIAM J. Control Optim.* 30 (4):838--55.
<https://doi.org/10.1137/0330046>.
:::

::: {#ref-Rakhlin_subgrad}
Rakhlin, A., O. Shamir, and K. Sridharan. 2012. "Making Gradient Descent
Optimal for Strongly Convex Stochastic Optimization." In *Proceedings of
the 29th International Coference on International Conference on Machine
Learning*, 1571--8. ICML'12. USA: Omnipress.
<http://dl.acm.org/citation.cfm?id=3042573.3042774>.
:::

::: {#ref-rob_mon}
Robbins, H., and S. Monro. 1951. "A Stochastic Approximation Method."
*Ann. Math. Statistics* 22:400--407.
:::

::: {#ref-sag}
Schmidt, M., N. Le Roux, and F. Bach. 2013. "Minimizing Finite Sums with
the Stochastic Average Gradient." *arXiv:1309.2388*.
:::

::: {#ref-sdca}
Shalev-Shwartz, S., and T. Zhang. 2012. "Proximal Stochastic Dual
Coordinate Ascent." *arXiv:1211.2717*.
:::

::: {#ref-accsdca}
---------. 2015. "Accelerated Proximal Stochastic Dual Coordinate Ascent
for Regularized Loss Minimization." *Mathematical Programming*.
:::

::: {#ref-ang_sing}
Singer, A. 2011. "Angular Synchronization by Eigenvectors and
Semidefinite Programming." *Appl. Comput. Harmon. Anal.* 30 (1):20--36.
<https://doi.org/10.1016/j.acha.2010.02.001>.
:::

::: {#ref-phase_nonconv}
Sun, J., Q. Qu, and J. Wright. 2017. "A Geometric Analysis of Phase
Retrieval." *To Appear in Found. Comp. Math., arXiv:1602.06664*.
:::

::: {#ref-NIPS2016_6058}
Woodworth, B.E., and N. Srebro. 2016. "Tight Complexity Bounds for
Optimizing Composite Objectives." In *Advances in Neural Information
Processing Systems 29*, edited by D. D. Lee, M. Sugiyama, U. V. Luxburg,
I. Guyon, and R. Garnett, 3639--47. Curran Associates, Inc.
<http://papers.nips.cc/paper/6058-tight-complexity-bounds-for-optimizing-composite-objectives.pdf>.
:::

::: {#ref-wright_PD}
Wright, S.J. 1997. *Primal-Dual Interior-Point Methods*. Society for
Industrial; Applied Mathematics (SIAM), Philadelphia, PA.
<https://doi.org/10.1137/1.9781611971453>.
:::

::: {#ref-prox_SVRG}
Xiao, L., and T. Zhang. 2014. "A Proximal Stochastic Gradient Method
with Progressive Variance Reduction." *SIAM J. Optim.* 24 (4):2057--75.
<https://doi.org/10.1137/140961791>.
:::
:::

[^1]: University of Washington, Department of Mathematics, Seattle, WA
    98195; `www.math.washington.edu/.17exddrusv/`

[^2]: To ensure that $${\rm prox}_{\nu f}(\cdot)$$ is nonempty, it
    suffices to assume that $$f$$ is bounded from below.

[^3]: For simplicity of the exposition, the minimization problem is
    unconstrained. Simple constraints can be accommodated using a
    projection operation.

[^4]: Here, I am ignoring logarithmic terms in the convergence rate.

[^5]: The accelerated SDCA was the motivation for the Catalyst
    framework, while APPA appeared concurrently with Catalyst.