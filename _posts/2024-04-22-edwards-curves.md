---
layout: post
title: Edwards curves and EdDSA verification trick
---

[Twisted Edwards curves](https://en.wikipedia.org/wiki/Twisted_Edwards_curve) has a form of $$ax^2 + y^2 = 1 +
dx^2y^2$$ (for fields with characteristic not 2) where
the [curve order](https://www.getmonero.org/library/Zero-to-Monero-2-0-0.pdf) can be represented as $$l \cdot 2^c$$,
where $$c$$ is a natural number, $$l$$ is a big prime number. So, it is obvious that our elliptic group has two
subgroups and for the cryptography purposes wew always may select the group defined by the large prime $$l$$.

Nevertheless, an attacker can select points from smaller subgroup and this points will be eligible for using in group
operations. To ensure that this never can happen, in EdDSA protocol verifier additionally uses the $$2^c$$ cofactor to
increase the security of the equation $$2^c\cdot rG = 2^c\cdot aG + 2^c t \cdot K$$. It ensures that all points are in
the lager prime curve subgroup, otherwise multiplication by the cofactor results with infinite point.
