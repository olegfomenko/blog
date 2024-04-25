---
layout: post
title: Edwards curves point compression
---

In addition to the post about Edwards curves, lats take a look on a useful feature that this curves allows. As we said
earlier, the Ed25519 for example, is defined over $$F_{2^{255}-19}$$ field, where every value requires 255 bit for the
binary representation. So every point will take about 510 bit. Using the point compression algorithm, we can encode
every point using only 256 bits by transferring only the $$y$$ coordinate.

Using the Edwards curves equation $$ax^2 + y^2 = 1 + dx^2y^2$$ we can express the $$x$$ coordinate (with an assumption
that $$a = -1$$) as $$x^2 = \frac{y^2 - 1}{dy^2 + 1}$$, which means that $$x$$ can have two possible solutions (+
and -). In the modular terms (since we have an odd modulo $$q$$) it means that $$x$$ can have the odd and even
representations. This information can be encoded into one bit.

Then, to decode the point coordinates, after separating parity bit from $$y$$ coordinate, we have to calculate the root
square. There are several methods to deal with the square root in
$$F_q$$. [One of them](https://ed25519.cr.yp.to/ed25519-20110705.pdf) is to use the feature that $$q =
2^{255} - 19 \equiv 5 (mod 8)$$. In the $$F_q$$ multiplicative group we have $$q - 1$$ elements (then for every $$x \in
F_q: x^q = x$$), so for every square $$\alpha =
\beta^2$$ we have $$\alpha^4 = \beta^8 \rightarrow \alpha^{p+3} = \beta^8 \rightarrow \alpha^\frac{p+3}{8} = \beta$$.
From the equation $$\alpha^4 = \beta^8$$ we can observe that there can be two possible solutions $$\pm\alpha =
\beta^2$$.
In the case when calculated $$\beta$$ satisfies $$\beta^2 = -\alpha$$ we should multiply it on $$\sqrt{-1}$$.

For the $$\sqrt{-1}$$ we can follow the same transformations: $$x^2 = -1 \rightarrow x^4 = 1 \rightarrow x^4 = 2^{q-1}
\rightarrow x = 2^\frac{q-1}{4}$$. The base $$2$$ element was selected as the most comfortable element for
multiplication.

So finally, for the equation $$x^2 = \frac{u}{v}$$ we can calculate $$x' = \sqrt{u/v}$$ and check that
$$v\cdot x' = -u$$ and if so multiply the $$x'$$ on $$\sqrt{-1}$$. On the final stage, we use the parity bit to check
that we've recovered same coordinate with same parity.



