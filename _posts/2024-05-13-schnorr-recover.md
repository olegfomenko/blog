---
layout: post
title: EdDSA signature challenge
---

For the EdDSA signature protocol (that is based on Schnorr authentication scheme) it is required to understand why and
how the challenge is generated. I've mentioned earlier that for the challenge generation in the non-interactive protocol
we are using the Fiat-Shamir heuristics. The basic idea is to generate challenges using the hash of all input data. It
is necessary to highlight that we have to include __all input data__ into the hash, otherwise malicious user can be able
to manipulate information that was not included and fixed in hash.

Let's take a look to the example of challenge generation in EdDSA: it uses the $$c = Hash(aG, K, m)$$ and $$r = a + c*
k$$ to verify later that $$rG = aG + \hat{c}K$$. Using such an approach the signer can not manipulate signature $$(A,
r)$$ because it's data is fixed in the challenge. But imagine if we are not including the $$aG$$ into the challenge.
Then, malicious signer without any knowledge of private key $$k$$ can select a random $$r$$ and put the signature as $$(
A = rG - cK, r)$$. It's easy to check that such signature passes the verification because verifier can not check that
$$A$$ value has been generated randomly and before challenge generation.
