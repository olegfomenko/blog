---
title: Using Montgomery multiplication for implementing foreign-field arithmetic
aside:
  toc: false
tags:
  - algebra
---

## Introduction

Recently, I found a very interesting application of how Montgomery multiplication can help implement arithmetic operations modulo $p$ with 256 bit integers, if the outer system already supports operations modulo $n$.

In my particular case, the Liquid Network (actually, the Simplicity extension) supports operations over the secp256k1 scalar field (say$\mod n_r$) and base field (say$\mod n_p$). Unfortunately, it doesn't support arbitrary field arithmetic over 256 bit integers. So, if, say, I would like to implement BN254 field arithmetic, what should I do? Initially, I came to the naive solution -- implementing arithmetic using 64 bit words (yeah, it does support operations like these). Following the gnark's approach, we did <https://github.com/distributed-lab/simplicity-bn254>, and frankly speaking -- it wasn't efficient enough. And recently, I found that another team implemented it using the Montgomery multiplication approach, with some additional tricks.

So, this post is about **how to implement a smaller field if you have operations over larger field(s)**.

## Montgomery method

Let's start from the beginning. The Montgomery multiplication is a method that allows you to do cheaper multiplication if you provide values in the Montgomery form. Similar to the projective coordinates, it becomes much more efficient when you have to perform multiple operations over values, and after -- convert to the final result.

The idea is pretty simple. Imagine we have to perform multiplication modulo $n$. The problem is that such operations may become pretty inefficient, so we want to reimplement them using more efficient operations like bit shifts. We select a special constant $R > n$, usually $R = 2^k$. Then, we represent each value as

$$
x^{\prime} = x\cdot R \mod n.
$$

Note that it is very important that $R$ has to be coprime to $n$, or in other words $\gcd(R, n) = 1$. Of course, such conversion is not efficient, but it saves a lot of time if you want to perform multiple multiplications in the future. Note that addition is straightforward:

$$
x^{\prime} + y^{\prime} = x\cdot R + y\cdot R = (x+y)\cdot R \mod n \equiv (x+y)^{\prime}.
$$

Unfortunately, the multiplication gives us an additional $R$ factor:

$$
x^{\prime}\cdot y^{\prime} = (x\cdot y)\cdot R^2 \mod n,
$$

so now the result does not follow the Montgomery form. To reduce the result, we do the following observations. By putting $t = (x\cdot y)\cdot R^2$ we want to obtain $t^{\prime} = t\cdot R^{-1} \mod n,$ where obviously $t^{\prime} = (x\cdot y)\cdot R$ is a Montgomery representation of $x\cdot y$. Because $\gcd(R, n) = 1$ by selection, we know from Bezout's lemma that there exist such $\hat{R}, \hat{n} \in \mathbb{Z}$ that

$$
R\cdot\hat{R} + n\cdot\hat{n} = \gcd(R, n) = 1.
$$

Using the extended Euclidean algorithm, we can easily find both values. We also know that $\hat{R}$ is $R^{-1}$ by modulo $n$, as well as $\hat{n}$ is $n^{-1}$ by modulo $R$ -- and they both are **constants**.
Then, let's rewrite our definition of $t^{\prime}$ as follows:

$$
\begin{gathered}
    t^{\prime} = t\cdot R^{-1} \mod n \\
    t^{\prime} = t\cdot R \cdot R^{-1} / R\\
    =  t\cdot(1 - n\cdot\hat{n})/R = (t - t\cdot n\cdot\hat{n}) / R\\
    \equiv (t + l\cdot R\cdot n - t\cdot n\cdot\hat{n}) / R \mod n\\
    \equiv (t + n(l\cdot R -t\cdot\hat{n}))/R \mod n,
\end{gathered}
$$

where $l \in \mathbb{Z}$. We note that subtracting or adding $l\cdot R$ for any $l \in \mathbb{Z}$ to the term $t\cdot\hat{n}$ actually equals taking it modulo $R$. So, to obtain $t^{\prime}$ we can:

1. Put $q = -t\cdot\hat{n} \mod R$,
2. Evaluate $t^{\prime} = (t + n\cdot q) / R$,
3. If $t^{\prime} \geq n$ put $t^{\prime} = t^{\prime} - n$.

The last step is required because we know that $t < Rn$ and $nq < Rn$ which means that $t + n\cdot q < 2Rn$.

So, why is it efficient? We earlier said that usually Montgomery multiplication uses $R = 2^k$, which means that division by $R$ or taking a value modulo $R$ are efficient bit operations.

## Implementing BN254

Returning back to our question, we have the following problem statement:

- We are able to perform $+\_{n_p}, +\_{n_r}, \ast\_{n_p}, \ast\_{n_r}$
- We want to perform $+\_{p}$ and $\ast\_{p}$ where $p$ is coprime to both $n_p$ and $n_r$ ($p$ can be either BN254 scalar field or base field).

Let's put $R = n_r$. Then, we represent each value as $x^{\prime} = x\cdot R \mod p$ as in the Montgomery method. We already know that addition comes for free: $x^{\prime} +\_{n_r} y^{\prime} = (x+y)\cdot n_r \mod p$ (obviously, only if $x^{\prime} + y^{\prime}$ doesn't overflow $n_r$; check the end of the article for details). The multiplication, unfortunately, is trickier. Following the Chinese Remainder Theorem

$$
\begin{gathered}
    s_r = x^{\prime}\ast_{n_r}y^{\prime}\\
    s_p = x^{\prime}\ast_{n_p}y^{\prime}
\end{gathered}
$$

we know that $s_r$ and $s_p$ uniquely represent $x^{\prime}\cdot y^{\prime}$ that we will use later. Let's firstly try to use Montgomery reduction and find $t^{\prime}=x^{\prime}\cdot y^{\prime}\cdot(n_r)^{-1} \mod p$ trivially:

1. Put $q = (-s_r \mod n_r) \ast\_{n_r}\hat{p}$, since $s_r \equiv x^{\prime}\cdot y^{\prime} \mod n_r$,
2. Put $a = (x^{\prime}\cdot y^{\prime} + p\cdot q)$, then evaluate $t^{\prime} = a / n_r \equiv a \cdot (n_r)^{-1} \mod p$.

The last division (multiplication by $(n_r)^{-1}$ by modulo $p$) is the remaining problem. Also, $x^{\prime}\cdot y^{\prime}$ is a 512 bit integer which we can not represent. We solve it by using $s_p$ and moving under $n_p$ modulus:

1. Put $\hat{q} = q \ast\_{n_p}p$,
2. Put $\hat{a} = (s_p +\_{n_p} \hat{q}) = (s_p +\_{n_p} (q \ast\_{n_p}p)) \equiv a \mod n_p$, then evaluate $t^{\prime} = a/n_r \equiv \hat{a} \ast\_{n_p} (n_r)^{-1} \mod n_p$, where $(n_r)^{-1}$ is taken modulo $n_p$.

Last equation gives us $t^{\prime} \mod n_p$, which is a valid result if $t^{\prime} < n_p$:

$$
t^{\prime} \leq \frac{x^{\prime} \cdot y^{\prime} + p \cdot n_r}{n_r} = \frac{x^{\prime} \cdot y^{\prime}}{n_r} + p,
$$

which holds until $x^{\prime} \cdot y^{\prime} < (n_p - p)\cdot n_r$. This rule must be enforced by checking that each value everywhere is less than some $k\cdot p < n_p$, where $k > 1 \in \mathbb{R}$. The real parameter $k$ for your application should be evaluated depending on your concrete equations, following the rule that the result before taking the modulus inside the equation can not overflow this modulus. As was noted above, the same rule must be applied to the addition as well.
