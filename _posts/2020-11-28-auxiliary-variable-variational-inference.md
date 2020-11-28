---
layout: post
---

Let's say you're doing some variational inference. You want to find an approximation for $p(x\vert z)$. Conventionally,  you'd have a variational family $q(z\vert x)$. But let's say it's easier to define your posterior approximation by first sampling an auxiliary variable $\theta$, and then sampling $q(z\vert \theta, x)$, producing a sample from the joint distribution $q(\theta, z\vert  x)$. How do you use this joint distribution in variational inference?

You could just take a Monte Carlo estimate. Sample $k$ different $\theta$. $q(z \vert  x) \approx \sum_{k=1}^K q(\theta_k, z \vert  x) / q(\theta_k)$. That's the approach taken [here](https://arxiv.org/pdf/1801.03612.pdf). But if you have an idea how to estimate $\theta$ from $z$ and $x$, there's a lower-variance way to do it.

Instead of trying to marginalize out the $\theta$ from $q$, let's expand our model to work on the joint space. Pick another variational family $g(\theta \vert  x, z)$. Assume the true joint posterior of $x, z$ and $\theta$ is given by $p(z\vert x)g(\theta \vert  x, z)$. As usual, variational inference tries to minimize the KL divergence between this true posterior and our approximation $q(\theta, z \vert  x)$. That's
$$
E_q \log q(\theta, z\vert x) - \log p(z\vert x) - \log g(\theta \vert  x,z) = \\
E_q \log q(z\vert x) - \log p(z\vert x) + \log q(\theta\vert z, x) - \log g(\theta \vert  x,z) = \\
KL[q(z\vert x)\vert \vert  p(z\vert x)] + KL[q(\theta\vert z,x) \vert \vert  g(\theta\vert x,z)]\\
$$
So $p(x) = \text{ELBO} + KL[q(z\vert x) \vert \vert  p(z\vert x)] +  KL[q(\theta\vert z,x) \vert \vert  g(\theta\vert x,z)]$, where ELBO is $E_q \log p(x,z) + \log g(\theta \vert  x, z) - \log q(\theta, z  \vert  x)$. We've found another evidence lower bound.  

This trick also works for importance re-sampling, as discussed in [MetaPPL](https://popl20.sigplan.org/details/lafi-2020/14/MetaPPL-Inference-Algorithms-as-First-Class-Generative-Models). Your importance weights become
$$
\frac{p(x,z)g(\theta\vert x,z)}{q(\theta, z\vert x)}
$$
where the best (least variance) $g$ possible is $q(\theta \vert  x,z)$. 

