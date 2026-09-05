---
title: Small fields in proving systems
aside:
   toc: false
tags:
  - algebra
  - zk
---

## Using KoalaBear field

> Linea and Gnark aim to operate over the KoalaBear field (over the prime modulus 2^31 - 2^24 + 1). The use of this field
> as the proving field (outer field) in modern proving backends may require implementing a field extension that
> increases
> the field size to \~128 bits. The goal they aim to achieve is to enable the use of fields where a field operation
> requires fewer uint64/uint32 manipulations than in default fields, where the element is a big integer represented by
> several smaller integers (which implement math in $\mathbb{Z}$).


The Gnark team has [already
implemented](https://github.com/Consensys/gnark-crypto/blob/master/field/koalabear/extensions/doc.go) such extensions (
both 64- and 128-bit). These fields operate over $\mathbb{F}[x_1]/(x_1^2-3)$ for a \~64-bit extension and $\mathbb{F}[x_2]^2/(
x_2^2-x_1)$ for a \~128-bit extension.

Let's review what the $\mathbb{F}[x]/(x^2-3)$ construction means exactly:

1. $\mathbb{F}$ — means our KoalaBear field
2. $\mathbb{F}[x]$ — means the field of all univariate polynomials with coefficients in $\mathbb{F}$
3. Starting from the beginning, the notation $aH$ represents a ring's ideal—an analogous structure to a coset in group
   theory. Basically, for the sub-ring $H$, it represents a sub-ring with elements $a\cdot h$ where $a\in\mathbb{F}$ and
   $b \in H$. The notation represents a factor-ring—the ring of all possible ideals by sub-ring $H$. In our case, $H =
   \{a(x^2-3)|a \in \mathbb{F}\}$, so the factor-ring $\mathbb{F}[x]/(x^2-3)$ contains all possible ideals by sub-ring
   $H = \{a(x^2-3)|a \in \mathbb{F}\}$.
4. Note that the ideal has the form $bH = \{ba(x^2-3)|a,b \in \mathbb{F}\}$ where $b$ is not in $H$. Otherwise, if $b\in
   H$, then $bH = H$, which makes no sense. The only elements in $\mathbb{F}$ that do not appear in $H = \{a(x^2-3)|a
   \in \mathbb{F}\}$ are the polynomials of the form $kx+b$. All other polynomials with degree $\geq 2$ already exist in
   $H$.
5. So, each element of such extension can be represented using two field elements $k$ and $b$.

As an example of such a field extension operation, let's check how multiplication is done for two elements $e_1 \in
\mathbb{F}[x]/(x^2-3)$ and $e_2 \in \mathbb{F}[x]/(x^2-3)$. We can represent them as $e_1 = (a_1, b_1)$ and $e_2 = (a_2,
b_2)$, which is equivalent to $a_1x+b_1$ and $a_2x+b_2$. The product of $e_1\cdot e_2$ is $a_1a_2x^2 + (
a_1b_2+a_2b_1)x + b_1b_2$, and we should also take it modulo $(x^2-3)$ to represent a unique element (see 4 above).
So finally, $e_1\cdot e_2 = (a_1b_2+a_2b_1)x + b_1b_2 + 3a_1a_2$.

In the KoalaBear case, using $\mathbb{F}[x]/(x^2-3)$ enables only a \~64-bit field, so we can use
two $\mathbb{F}[x]/(x^2-3)$ elements to act over $\mathbb{F}[x]^2/(x^2-x')$. You can understand it as the same extension
but over $\mathbb{F}[x]/(x^2-3)$ instead of $\mathbb{F}$.
