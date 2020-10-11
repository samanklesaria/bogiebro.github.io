---
layout: post
---

Generative models make it easy to find $p(x\vert z)$, where $x$ is an observation and $z$ is a latent code. But it's often hard to find the posterior $p(z \vert x)$, so we try to approximate it. Variational inference tries to find the best approximation within some family of distributions $q_\phi(z)$.  What does 'best' mean here? There's two ways to look at it. 

1. Choose $\phi$ to minimize $KL[q_\phi(z)  \vert \vert p(z|x)]$. 
   $$
   \begin{align*}
   KL[q_\phi(z)  \|  p(z|x)] &= E_{z \sim q} \log q(z) - \log p(z|x) \\
   &= E_{z \sim q} \log q(z) - \log p(z,x) + \log p(x) \\
   &= -E_{z \sim q} \log p(z,x) - \log q(z)+c \\
   &= -\text{ELBO} + c
   \end{align*}
   $$

   where the value we'll call the ELBO is $E_{z \sim q} \log p(z,x) - \log q(z)$. If we maximize the ELBO, we end up minimizing the KL divergence between our approximating posterior and the true posterior. 

2. Choose $q_\phi(z)$ as the proposal distribution maximizing the importance sampled estimate of the evidence $p(x)$. 

$$
\begin{align*}
\log p(x) &= \log E_{z \sim p(z)} p(x|z) \\
&= \log E_{z \sim q} \frac{p(x|z) p(z)}{q(z)} &\text{ (importance sampling)}\\
& \geq E_{z \sim q} \log \frac{p(x|z) p(z)}{q(z)} &\text{ (Jenson's inequality)} \\
&= ELBO


\end{align*}
$$

   This justifies the ELBO's name (the **e**vidence **l**ower **bo**und). 
