---
layout: post
title: Reinforcement Learning is MAP Inference
---

Let's say you have a generative model to explain some observations $x$ in terms of some latent $z$:


$$
\begin{align*}
z_1 &\sim p(z_1) \\
x_1 &\sim p(x_1 \vert z_1) \\
z_2 &\sim p(z_2 \vert z_1) \\
x_2 &\sim p(x_2 \vert z_2) \\
&\dots \\
z_N &\sim p(z_N \vert z_{N-1}) \\
x_N &\sim p(x_N | z_N)
\end{align*}
$$


You want to find a point estimate of $z$ that maximizes the joint log probability $\log p(x,z)$. This doesn't seem so bad: just choose the best possible $z_1$, then the best possible $z_2$ given $z_1$, and so on. More precisely, at each step, you want to choose $z_i$ to be $\arg \max_{z_i} \max_{z_{i+1:N}}\log p(x_{i:N}, z_{i:N} \vert z_{i-1})$.



This gives us a nice recurrence for dynamic programming:


$$
\max_{z_{i+1:N}} \log p(x_{i:N}, z_{i:N} \vert z_{i-1})= \log p(x_i, z_i \vert z_{i-1}) + \max_{z_{i+2:N}} \log p(x_{i+1:N}, z_{i+1:N} \vert z_i)
$$


We can rewrite this recurrence in a more suggestive fashion:


$$
Q^*(z_{i-1},z_i)=R_i + \max_{z_{i+1}}Q^*(z_i, z_{i+1})
$$


Here, $Q^*(z_{i-1}, z_i)$ is the maximum joint log probability possible for $(z_{i:N},x_{i:N})$ given $z_{i-1}$.   $R_i=\log p(x_i, z_i \vert z_{i-1})$.  This is just the Bellman optimality equation for the action-value function in reinforcement learning! MAP inference, from this perspective, is just a Markov decision process. The states are instantiations for previously sampled latent variables $z_i$. The actions are choices for $z_{i+1}$. And the value function $V(z_i)$ for a policy that chooses actions $z_{i+1} \dots z_N$ is the undiscounted sum of the rewards, giving $\log p(x_{i+1:N}, z_{i+1:N} \vert z_i)$. So the inference task of finding $z_{i:N}$ maximizing $\log p(x_{i:N}, z_{i:N} \vert z_{1:i})$ is the same as the reinforcement learning task of finding an optimal policy. 



This means that all the tools we have for doing approximate MAP inference can be re-purposed for reinforcement learning. It also means that all the tools we have for doing reinforcement learning can be brought to bear on inference problems. I'll explore some examples of this in future posts. 