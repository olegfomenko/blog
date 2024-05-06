---
layout: post
title: Schnorr Ring signatures - Part 2
---

_Linkability_ is a property that describes relation between two signatures. If the protocol has such property
then given two different signatures (ring signatures) it becomes possible to check that them have been produced by the
same signer. Linkability in a couple with _anonymity_ property gives verifier an opportunity to check this relation
without revealing any information about signer.

_Back’s Linkable Spontaneous Anonymous Group_ (bLSAG) signature protocol as a modification of the
described in previous post SAG protocol introduces the ring signature with characterized by anonymity, linkability
properties. Before going thought the protocol let's describe the special hash function $$H_p(x) \rightarrow \mathbb{G}$$
the gives as the result a point in curve (DL problem can not be solved with overwhelming probability). So, for the given
ring $$R = \{K_1, K_2, ..., K_d\}$$ where we know the private key $$k_x$$ of the key with secret position $$x$$, bLSAG
protocol can be defined as follows:

1. Calculate key image $$\hat{K} = k_x\cdot H_p(K_x)$$.
2. Generate random key $$a$$ and $$r_i$$ for all $$i$$ except of $$i = x$$.
3. Put $$c_{x+1} = H(R, m, [a\cdot G], [a\cdot H_p(K_x)])$$.
4. Then, for every $$i = x+1, x+2, ..., d, 1, 2, ..., x-1$$ (replacing $$d + 1 \rightarrow 1$$) calculate $$c_{i+1} = H(
   m, [r_i\cdot G + c_i\cdot K_i], [r_i\cdot H_p(K_i) + c_i\cdot\hat{K}])$$.
5. Put the response $$r_x = a - c_xk_x$$.
6. Define the signature $$(c_1, r_1, ... , r_d)$$, ring $$R$$ and key image $$\hat{K}$$.

To verify such signature firstly verifier check that $$l\cdot\hat{K} = 0$$ where $$l$$ is the prime order of the big
subgroup of our elliptic curve. That is required because malicious signer can select points from the subgroup of small
cofactor $$h$$ that can affect linkability property. Then, for the every $$i = 1, 2, ..., d$$ (replacing $$d + 1
\rightarrow 1$$) calculates $$c_
{i+1}' = H(m, [r_i\cdot G + c_i\cdot K_i], [r_i\cdot H_p(K_i) + c_i\cdot\hat{K}])$$ and checks that $$c_1 = c_1'$$.

Finally, if the two different signatures (even with different rings) have been produced by the same signer then the both
will
have the same key images $$\hat{K}$$.
