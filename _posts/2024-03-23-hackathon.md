---
layout: post
title: Untraceable transaction with hidden amounts (2024 March 23-24)
---

The [Confidential assets paper](https://blockstream.com/bitcoin17-final41.pdf)
and [Ring Confidential Transactions](https://www.getmonero.org/es/resources/research-lab/pubs/MRL-0005.pdf) introduces
two flows how to anonymize transactions in blockchain. The first one explains the solution to hide transaction amount
via [Pedersen commitment](https://link.springer.com/content/pdf/10.1007/3-540-46766-1_9.pdf) while the second introduces
the untraceable coin transferring using stealth addresses.

On the recent hackathon in Distributed Lab my team (Oleg Fomenko, Mykhailo Sokolov, Serhii Pomohaiev and Oleksandra Ryzhova) introduced the way
how to combine this two approaches to make transactions hidden and untraceable but still verifiable.

Briefly, it works in the following way: imagine we have all assets owned by users stored in a
pairs of `<Curve point, Curve point>`. First point will be a Pedersen commitment while the second will be an associated
public key. Then, imagine Alice owns and `<C_1, A_2>` where `A_2 = a_2G`, `C_1 = vH + a_1G` and `A_1 = a_1G`  (`v` is an
amount of owned tokens). It requires `sig(a_2)` to be spent besides the part of aggregated signature for `a_1`. Bob
generates `B_1 = b_1G` and `B_2 = b_2G`.

To perform anonymous untraceable transfer between Alice and Bob they exchange their addresses: Alice
sends `A2` to Bob and Bob sends `B1`, `B2` to the Alice. Then both can compute Diffie-Hellman
secrets as `k_1 = Hash(B_1 * a_2)` or `k_1 = Hash(A_2 * b_1)` and `k_2 = Hash(B_2 * a_2)` or `k_2 = Hash(A_2 * b_2)`.

Then, Alice calculates receiver parameters as `C_2 = vH + k_1G` and `PK_B = k_2G + B_2` and makes a transfer transaction
from `<C_1, A_2>` to `<C_2, PK_B>`, attaching `sig(a_2)`, `sig(a_1 - k_1)` and range proof for `C_2`. Currently, Alice
knows all openings for destination commitment `C_2` but can't spend it because of unknown `PK_B` secret. On the other
side, Bob knows all openings for both `C_2` and `PK_B` and after transaction execution can use received tokens.

One of the most important feature is that presented approach also gives us a non-interactivity in communication: Alice
can generate aggregated signature and range proof by herself unlike default "confidential assets" protocol.

This solution was implemented in Golang Cosmos module as a proof of concept and can be explored in
this [repository](https://github.com/olegfomenko/rarimo-core/tree/feature/hackathon-condidential-transfers-fixed).
