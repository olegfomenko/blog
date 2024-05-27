---
layout: post
title: Schnorr Ring signatures - Part 2
---

_Linkability_ is a property that describes relation between two signatures. For the protocols with such property it
becomes possible to check that two different signatures have been signed with the same public keys. Linkability in a
couple with _anonymity_ property gives verifier an opportunity to check this relation without revealing any information
about signer.

_Back’s Linkable Spontaneous Anonymous Group (bLSAG) signature_ protocol, as a modification of the
described in previous post SAG protocol, introduces the ring signature witch follows anonymity and linkability
properties. Before going thought the protocol let's describe the special hash function $$H_p(x) \rightarrow \mathbb{G}$$
the gives as the result a point in curve (Discrete-Log problem can not be solved with overwhelming probability). So, for
the given ring $$R = \{K_1, K_2, ..., K_d\}$$ where we know the private key $$k_x$$ of the key with secret position
$$x$$, bLSAG protocol can be defined as follows:

1. Calculate key image $$\hat{K} = k_x\cdot H_p(K_x)$$.
2. Generate random key $$a$$ and $$r_i$$ for all $$i$$ except of $$i = x$$.
3. Put $$c_{x+1} = H(m, [a\cdot G], [a\cdot H_p(K_x)])$$.
4. Then, for every $$i = x+1, x+2, ..., d, 1, 2, ..., x-1$$ (replacing $$d + 1 \rightarrow 1$$) calculate $$c_{i+1} = H(
   m, [r_i\cdot G + c_i\cdot K_i], [r_i\cdot H_p(K_i) + c_i\cdot\hat{K}])$$.
5. Put the response $$r_x = a - c_xk_x$$.
6. Define the signature $$(c_1, r_1, ... , r_d)$$, ring $$R$$ and key image $$\hat{K}$$.

To verify this signature, verifier checks that $$l\cdot\hat{K} = 0$$ where $$l$$ is the prime order of the big
subgroup of our elliptic curve. This is necessary because the malicious user can choose points from a subgroup with small
cofactor $$h$$ that can affect linkability property. After, for the every $$i = 1, 2, ..., d$$ (replacing $$d + 1
\rightarrow 1$$) verifier calculates $$c_
{i+1}' = H(m, [r_i\cdot G + c_i\cdot K_i], [r_i\cdot H_p(K_i) + c_i\cdot\hat{K}])$$ and checks that $$c_1 = c_1'$$.

Finally, if the two different signatures (even with different rings) have been produced by the same signer then the both
will have the same key images $$\hat{K}$$.
