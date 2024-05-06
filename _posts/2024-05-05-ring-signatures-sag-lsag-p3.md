---
layout: post
title: Schnorr Ring signatures - Part 3
---

Finally, finishing the posts about ring signatures based on Schnorr authentication protocol we are able to overview the
_Multilayer Linkable Spontaneous Anonymous Group (MLSAG) signature_ protocol. This protocol provides an opportunity to
generate signature with several signers while they are all still hidden in the ring. So, given the ring $$R = \{K_
{i,j}\}$$ for $$i \in \{1,2,...n\}$$ and $$j \in \{1,2,...m\}$$ where we know the private keys $$k_{x,j}$$ of the keys
with secret position $$x$$ for corresponding public keys $$K_{x,j}$$ for $$j \in \{1,2,...m\}$$. The MLSAG protocol has
a lot in common with bLSAG protocol:

1. Calculate key image $$\hat{K}_j = k_{x,j}\cdot H_p(K_{x,j})$$ for $$j \in \{1,2,...m\}$$.
2. Generate random key $$a_j$$ for $$j \in \{1,2,...m\}$$.
3. For $$i \in \{1,2,...n\} \setminus x$$ and $$j \in \{1,2,...m\}$$ generate random $$r_{i,j}$$.
4. Put $$c_{x+1} = H(m, [a_1\cdot G], [a_1\cdot H_p(K_{x,1})], [a_2\cdot G], [a_2\cdot H_p(K_{x,2})],...)$$.
5. Then, for every $$i = x+1, x+2, ..., n, 1, 2, ..., x-1$$ (replacing $$n + 1 \rightarrow 1$$) calculate $$c_{i+1} = H(
   m, [r_{i,1}\cdot G + c_i\cdot K_{i,1}], [r_{i,1}\cdot H_p(K_{i,1}) + c_i\cdot\hat{K}_1], ...)$$.
6. Put the response $$r_{x,j} = a - c_xk_{x,j}$$ for $$j \in \{1,2,...m\}$$.
7. Define the signature $$(c_1, r_{i,j})$$, ring $$R$$ and key images $$\hat{K}_j$$.

To verify such signature firstly verifier checks $$\forall j \in \{1,2...m\}: l\cdot\hat{K}_j = 0$$ where $$l$$ is the
prime order of the big subgroup of our elliptic curve. Then, for the every $$i = 1, 2, ..., n$$ (replacing $$n + 1
\rightarrow 1$$) verifier calculates $$c_{i+1} = H(
m, [r_{i,1}\cdot G + c_i\cdot K_{i,1}], [r_{i,1}\cdot H_p(K_{i,1}) + c_i\cdot\hat{K}_1], ...)$$ and checks that $$c_1 =
c_1'$$.

Finally, if for the two different signatures $$S_1$$ and $$S_2$$ such indexes $$i,j$$ exist that $$\hat{K}_{S_1,i} =
\hat{K}_{S_2,j}$$ then these signatures are linked by signing from the same key.

![Ring signatures using Schnorr]({{ site.url }}/assets/img/RingsCheatsheet.png)
