---
layout: post
title: Schnorr Ring signatures - Part 3
---

Now, finishing the posts about ring signatures based on Schnorr authentication protocol we are able to overview the
_Multilayer Linkable Spontaneous Anonymous Group (MLSAG) signature_ protocol. This protocol provides an opportunity to
generate signature with several signers while they are all still hidden in the ring. So, given the ring $$R = \{K_
{i,j}\}$$ for $$i \in \{1,2,...n\}$$ and $$j \in \{1,2,...m\}$$ where we know the private keys $$k_{x,j}$$ for
corresponding public keys $$K_{x,j}$$ with secret position $$x$$ for $$j \in \{1,2,...m\}$$. Then, the MLSAG protocol has
a lot in common with bLSAG protocol:

![MLSAG signing]({{ site.url }}/assets/img/mlsag.png)

To verify such signature, verifier checks that $$\forall j \in \{1,2...m\}: l\cdot\hat{K}_j = 0$$ where $$l$$ is the
prime order of the big subgroup of our elliptic curve. Then, for the every $$i = 1, 2, ..., n$$ (replacing $$n + 1
\rightarrow 1$$) verifier calculates $$c_{i+1} = H(
m, [r_{i,1}\cdot G + c_i\cdot K_{i,1}], [r_{i,1}\cdot H_p(K_{i,1}) + c_i\cdot\hat{K}_1], ...)$$ and checks that $$c_1 =
c_1'$$.

Finally, if for the two different signatures $$S_1$$ and $$S_2$$ such indexes $$i,j$$ exist that $$\hat{K}_{S_1,i} =
\hat{K}_{S_2,j}$$ then these signatures are linked by signing from the same key.

In addition, using signatures linking and one-time stealth addresses Monero blockchain solves a double-spending problem:
if transaction contains the key image that already has been included into the previous block then in seems to be a
double-spending.

![Ring signatures using Schnorr]({{ site.url }}/assets/img/RingsCheatsheet.png)
