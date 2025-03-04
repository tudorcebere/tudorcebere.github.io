---
title: 'Exact coins '
date: 2024-06-12
layout: post
tags:
  - Probability
---
<!--more--> 
How to exactly toss a coin? How to sample random coins has been central to the theory of computing and I am definently not in the position to comment on that, however, I recently needed to exactly sample a coin with a probability $exp(-\lambda), \lambda > 0$. Even more, this probability had to be exact and floating-point stable. 

My approach to try to do that is standard and it requires the binary representation of the bias of the coin. However, computing the binary representation for $exp(-\lambda)$ given $\lambda$ is unapealling and has no shade of elegance attached to it.

However, I recently discovered an elegant way to do it in constant expected bits of randomness without the need of computing the binary representation. While the solution takes half a page in the discrete gaussian paper, I have decided to shed some light on it through this post so people can learn how to exactly sample Bern(exp($-\lambda$)).


# Coin sampling via binary expansion

Assume we want to compute Bern($\alpha$), $\alpha \in [0, 1]$. Let $\alpha_i$ be the binary expansion of $\alpha$. A simple way with outstanding computational performance is to sample fair coins and stop when the fair coin does not match the binary expansion of the exponent. In the idealised setting, we would want to sample u ~ U(0, 1) and compare with $\alpha$. Via the binary expansion, we do the same thing, but we decompose the binary number until we find a missmatch in the representation. Due to the exponential nature of this algorithm, we observe that the expected number of coin tosses to generate a sample is 2.
