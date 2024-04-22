---
layout: post
title: Edwards curves
---

[Twisted Edwards curves](https://en.wikipedia.org/wiki/Twisted_Edwards_curve) has a form of $$ax^2 + y^2 = 1 +
dx^2y^2$$ (for fields with characteristic not 2) where
the [curve order](https://www.getmonero.org/library/Zero-to-Monero-2-0-0.pdf) can be represented as $$l \cdot 2^c$$, where
$$c$$ is a natural number, $$l$$ is a big prime number.

For the classical EdDSA signature algorithm it uses the Ed25519 curve over the prime field $$F_{2^{255}-19}$$ with
definition: $$-x^2+y^2=1-\frac{121665}{121666}x^2y^2$$. The reason of this another one curve creation is the
optimization
of this curve efficiency compared to the other popular curves. And, while talking about EdDSA algorithm it's required to
highlight that it does not require any PRNG because the signature one-time keys is deterministically generated using
message and private key.

Let me show you how EdDSA protocol is related to the classical Schnorr authentication protocol.

The Schnorr authentication protocol has the following form:

- Alice owns a secret value $$k$$ and shares the public key $$K = kG$$, where $$G$$ is the group generator. Alice's goal
  is to prove to the Bob the knowledge of $$k$$ that corresponds to the provided public key $$K$$ without revealing any
  private information.
- Alice generates random key $$a$$ and sends $$A = aG$$ to Bob.
- Bob generates random challenge $$c$$ and sends it to Alice.
- Alice calculates the response $$r = a + kc$$ and sends is back to the Bob.
- Bob checks that $$rG = A + cK$$.

Later, after presenting of the Fiat-Shamir heuristics, where for the challenges generation prover can use the hash of
input data such that verifier can also calculate, it becomes able to describe Schnorr authentication protocol in the
non-interactive form:

Alice's flow

- Generate random key $$a$$ and public $$A = aG$$.
- Calculate challenge $$c = Hash(A)$$.
- Calculates the response $$r = a + kc$$.
- Send $$(A, r)$$ to the Bob.

Bob's flow

- Calculate challenge $$c = Hash(A)$$.
- Check that $$rG = A + cK$$.

Obviously, that using such algorithm we can add the message to the challenge generation and get the signature algorithm.
So, later, using the Ed25519 curve, the EdDSA signature protocol has been presented:

Alice's flow:

- Calculate $$a = Hash(Hash(k), m)$$, $$ch = Hash(aG, K, m)$$ and $$r = a + ch\cdot k$$.
- Send signature $$(aG, r)$$.

Bob's flow:

- Calculate $$ch = Hash(aG, K, m)$$.
- Check that $$2^c\cdot rG = 2^c\cdot aG + 2^c ch\cdot K$$. The $$2^c$$ cofactor is used to increase the security of the equation: it
  ensures that all points are in the lager prime curve subgroup.
