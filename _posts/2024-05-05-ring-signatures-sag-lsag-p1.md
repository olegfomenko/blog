---
layout: post
title: Schnorr Ring signatures - Part 1
---

In previous posts I've covered the Schnorr authentication protocol and its modifications. Continuing
this topic let's take a look at the ring signature protocols that can be built using Shcnorr signature.

A _ring signature_ is a signature protocol in which, using a set of public keys (ring) $$R$$, some of them can sign the
message $$m$$ without revealing any information about who exactly it was.
The [simplest](https://en.wikipedia.org/wiki/Ring_signature) ring signature scheme uses encryption protocols (RSA for
example) to prove that xor over all encryption of provided scalars array is equal to
zero (according to this protocol, one of the element in an array for the corresponding known private key will be
calculated to satisfy our xor equation).

More complex and flexible ring signatures can be built as a modification of Schnorr authentication protocol. Staring
from the beginning let's describe the form of Schnorr signature that we will later modify:

- For the private key $$k$$ and public $$K = k\cdot G$$ let's sign the message $$m$$.
- Generate random key $$a$$ and public $$A = a\cdot G$$.
- Calculate challenge $$c = Hash(m, A)$$.
- Put the response $$r = a - kc$$.
- The signature will be $$(c, r)$$.

Then, verifier calculates $$c' = Hash(m, [r\cdot G + c\cdot K])$$ and checks that $$c = c'$$.

Moving to the ring signature protocol, lets define the ring as $$R = \{K_1, K_2, ..., K_d\}$$ where we know the private
key $$k_x$$ of the key with secret position $$x$$. The, to generate the signature for the message $$m$$ where the signer
belongs to the ring $$R$$ we will go through the following _Spontaneous Anonymous Group (SAG) signature_ protocol:

1. Generate random key $$a$$ and $$r_i$$ for all $$i$$ except of $$i = x$$.
2. Put $$c_{x+1} = H(R, m, [a\cdot G])$$.
3. Then, for every $$i = x+1, x+2, ..., d, 1, 2, ..., x-1$$ (replacing $$d + 1 \rightarrow 1$$) calculate $$c_{i+1} = H(
   R, m, [r_i\cdot G + c_i\cdot K_i])$$.
4. Put the response $$r_x = a - c_xk_x$$.
5. The signature will be $$(c_1, r_1, ... , r_d)$$ and the ring $$R$$.

To check this ring signature, verifier for the every $$i = 1, 2, ..., d$$ (replacing $$d + 1 \rightarrow 1$$)
calculates $$c_
{i+1}' = H(R, m, [r_i\cdot G + c_i\cdot K_i])$$ and checks that $$c_1 = c_1'$$.



