---
layout: post
title: Schnorr Ring signatures - Part 1
---

In previous posts I've taken a look on the Schnorr authentication protocol and its modifications. Continuing to discussing
this theme let's overview the ring signature protocols that can be built using Shcnorr signature.

Ring signature is a signature protocol where using some set of public keys (ring) $$R$$ some of them can sign the
message $$m$$, and it will not be possible to recognize signers' public keys. The simplest ring signature scheme uses
encryption protocols (RSA for example) to prove that xor over all encryption of provided scalars array is equal to
zero (using such an approach, one of the element in an array for the corresponding known private key will be calculated
to satisfy our xor equation). Another one approach is to modify Schnorr authentication protocol to be used as a ring
signature algorithm.

Staring from the beginning let's describe the form of Schnorr signature protocol that we will later modify:

- For the private key $$k$$ and public $$K = k\cdot G$$ let's sign the message $$m$$.
- Generate random key $$a$$ and public $$A = a\cdot G$$.
- Calculate challenge $$c = Hash(m, A)$$.
- Put the response $$r = a - kc$$.
- The signature will be $$(c, r)$$.

Then, verifier calculates $$c' = Hash(m, r\cdot G + c\cdot K)$$ and checks that $$c = c'$$.

Moving to the ring signature protocol, lets define the ring as $$R = \{K_1, K_2, ..., K_d\}$$ where we know the private
key $$k_x$$ of the key with secret position $$x$$. The, to generate the signature for the message $$m$$ where the signer
belongs
to the ring $$R$$ we will go through the following _Spontaneous Anonymous Group (SAG) signature_ protocol:

- Generate random key $$a$$ and $$r_i$$ for all $$i$$ except of $$i = x$$.
- Put $$c_{x+1} = H(R, m, a\cdot G)$$.
- Then, for every $$i = x+1, x+2, ..., d, 1, 2, ..., x-1$$ (replacing $$d + 1 \rightarrow 1$$) calculate $$c_{i+1} = H(
  R, m, r_i\cdot G + c_i\cdot K_i)$$.
- Put the response $$r_x = a - c_xk_x$$.
- The signature will be $$(c_1, r_1, ... , r_d)$$ and the ring $$R$$.

To verify this ring signature, verifier for the every $$i = 1, 2, ..., d$$ (replacing $$d + 1 \rightarrow 1$$) calculates $$c_
{i+1}' = H(R, m, r_i\cdot G + c_i\cdot K_i)$$ and checks that $$c_1 = c_1'$$.



