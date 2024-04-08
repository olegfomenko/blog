---
layout: post
title: Bulletproofs++ implementation
---

Zero-knowledge range proofs can be shortly introduced as one of the most important algorithms for confidential
transactions. After Bitcoin launch lots of cryptographers started to create the solutions to make blockchain
transactions more and
more anonymous and one of such solution was to hide user balances in cryptographic commitments (in particular - Pedersen
commitments). This approach requires the way to check that amounts in transaction corresponds to each other without
group order overflowing. That is why a wide variety of range proof protocols appeared.

The [Bulletproofs](https://eprint.iacr.org/2017/1066.pdf) protocol published in 2017 introduced how to verify value
range committed in Pedersen commitment. Later, Blockstream cryptographers introduces more efficient
protocol [Bulletproofs+](https://eprint.iacr.org/2020/735.pdf) that currently implemented in many different platforms
e.g - Monero.

The last protocol generation (Bulletproofs++) that is designed to be more efficient the '+' version was introduced in
2022 [paper](https://eprint.iacr.org/2022/510.pdf). My partner, Mykhailo Sokolov, and I have spent the last three months
thoroughly studying this document in order to develop implementation libraries in Go and Rust that are suitable for use
in production projects.It was unexpected to find errors in the original document, rendering the described protocol
unusable in its current form. Consequently, we have developed a corrected version of the Bulletproofs++ protocol and are
currently in the process of drafting an article to showcase our revisions.

In conclusion, our solution has $$ 2G $$ points advantage over existing Bulletproofs and Bulletproofs+ protocols on proving
of one 64-bit value and this advantage will increase for more values per proof.

| Protocol | G  | F |
|----------|----|---|
| BP       | 16 | 5 |
| BP+      | 15 | 3 |
| Our BP++ | 13 | 3 |

Our implementation can be found here:

- [Rust](https://github.com/distributed-lab/bp-pp)
- [Go](https://github.com/distributed-lab/bulletproofs)

