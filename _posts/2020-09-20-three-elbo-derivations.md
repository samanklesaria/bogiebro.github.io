---
layout: post
---

Generative models make it easy to find $p(x\vert z)$, where $x$ is an observation and $z$ is a latent code. But it's often hard to find the posterior $p(z \vert x)$, so we try to approximate it. Stochastic variational inference tries to find the best approximation within some family of distributions $q_\phi(z)$.  What does 'best' mean here? There's a few different ways to look at it. 

1. Choose $\phi$ to minimize $E_x KL[q_\phi(z; x)  \vert \vert p(z \vert x)]$, where $E_x$ takes samples from the empirical data distribution.

   $$
   \begin{align*}
   KL[q_\phi(z)  \|  p(z \vert x)] &= E_{z \sim q} \log q(z) - \log p(z \vert x) \\
   &= E_{z \sim q} \log q(z) - \log p(z,x) + \log p(x) \\
   &= -E_{z \sim q} \log p(z,x) - \log q(z)+c \\
   &= -\text{ELBO} + c
   \end{align*}
   $$

   The value we'll call the ELBO is $E_{z \sim q} \log p(z,x) - \log q(z)$. If we maximize the expected ELBO when sampling from our data, we end up minimizing the expected KL divergence between our approximating posterior and the true posterior. 

2. Choose $q_\phi(z)$ as the proposal distribution maximizing the importance sampled estimate of the expected evidence $E_x p(x)$. We maximize the log of this value:

$$
\begin{align*}
\log E_x p(x) &\leq E_x \log p(x) &\text{ (Jenson's inequality)}\\
&= E_x \log E_{z \sim p(z)} p(x|z) \\
&= E_x \log E_{z \sim q} \frac{p(x|z) p(z)}{q(z)} &\text{ (importance sampling)}\\
& \geq E_x E_{z \sim q} \log \frac{p(x|z) p(z)}{q(z)} &\text{ (Jenson's inequality)} \\
&= E_x ELBO
\end{align*}
$$

   This justifies the ELBO's name (the **e**vidence **l**ower **bo**und). 

   Why, you may ask, do we apply Jenson's inequality twice? Why not just stop with the importance sampling estimate, rather than taking yet another lower bound? Well, that's what the [Importance Weighted Autoencoder](https://arxiv.org/pdf/1509.00519.pdf) does! It's obvious from this perspective that this is a closer bound on the log evidence than what the ELBO gives us. 

3. Let $q$ approximate the joint instead of the posterior, and choose $\phi$ to maximize $-KL[q_\phi(z,x) \vert \vert p(z,x)]$. 
   $$
   \begin{align*}
   -KL[q_\phi(z,x) \vert \vert p(z,x)] &= E_{(x,z) \sim q} [p(z,x) - \log q(z,x)] \\
   &= E_{(x,z) \sim q} [p(z,x) - \log q(z \vert x) - \log q(x)]
   \end{align*}
   $$
   
   If we let $q(x)$ be the empirical data distribution, the $\log q(x)$ term does not depend on $\phi$, so we end up maximizing $E_{(x,z) \sim q} [p(z,x) - \log q(z \vert x)]$. Again: the expected ELBO! This is the perspective taken by [Structured Distangled Representations](http://proceedings.mlr.press/v89/esmaeili19a/esmaeili19a.pdf) 
