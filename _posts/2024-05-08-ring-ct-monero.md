---
layout: post
title: How RingCT from Monero works?
---

It wasn't a coincidence that we were talking about *SAG signatures earlier. MLSAG signatures became one of the bases for
Monero blockchain confidentiality and anonymity. Using them, users can create ring signature for spending assets from
one of the keys and nobody will be able to answer "what tokens have been spent?" or "have some certain tokens been
spent?". This post will describe how RingCT works and how RingCT transactions are built.

First of all, let's take a look on the storage model. Monero uses UTXO-based architecture for storing balances as well
as Bitcoin, so in transactions inputs users operate other transaction outputs. Transaction output constitutes a pair of
elliptic curve points that represents user stealth (one-time) address and hidden amount in _Pedersen commitment_ of form
$$C^a = x\cdot G + a\cdot H$$, where $$x$$ is a commitment blinding.

_Stealth addresses_ is an approach when sender can transfer money to the generated one-time address of recipient and
nobody will not be able to understand that certain two users had any kind of interaction. More precisely, receiver can
share two keys $$(K^v, K^s)$$ - view key and spend key. Received during the transaction creation will generate a random
value $$r$$ and using it will generate a one-time recipient address as $$K^O = H(r\cdot K^v)\cdot G + K^s$$. Also,
$$r\cdot G$$ will be added to the transaction extra data and called "transaction public key". Receiver, in his order
should monitor all transactions and using public $$K^O$$ and $$rG$$ checks if $$K^O - H(k^v\cdot rG)\cdot G = K^s$$. If
yes - user understands that he is a receiver of these coins and can also calculate the private key $$k^O = H(k^v\cdot
rG) + k^S$$.

So, imagine user wants to transfer some coins using $$(K^a_i, C^a_i)$$ where $$i\in\{1,...,m\}$$ for the input and $$(
K^b_j, C^b_j)$$ where $$j\in\{1,...,p\}$$ for the output ($$K^b_i$$ is a receiver one-time address and $$C^b_j = y\cdot
G + b\cdot H$$). To achieve additional confidentiality, users generates $$m$$ _pseudo-output commitments_ $$\hat{C}^a_
i$$ with same amounts but different blinding and in a such way that $$\sum x_i' - \sum y_j = 0$$. It's obvious that
using such construction $$\sum \hat{C}^a_i - \sum C^b_j = 0$$, so we can convince verifier that sum of input equals to
the output. Also, note that for every $$i$$ sender knows the private key for zero-value commitment $$C^a_i -\hat{C}^a_i=
x_i - x_i' = z_i$$.

Then, for every input $$j$$ sender selects a random ring of size $$v+1$$ with form
$$R = \{\{K_{1,j}, (C_{1,j} - \hat{C}^a_{\pi,j})\},...,\{K_{\pi,j}, (C^a_{\pi,j} - \hat{C}^a_{\pi,j})\},...,\{K_
{v+1,j}, (C_{v+1,j} - \hat{C}^a_{\pi,j})\}\}$$. User can generate a MLSAG signature for it because of knowledge of
secrets for $$\pi$$ position: from public key $$K_{\pi,j}$$ and from zero-value commitment $$C^a_{\pi,j} - \hat{C}^a_
{\pi,j} = x_{\pi,j} - x'_{\pi,j} = z_j$$. Also, user attaches the key image for his key $$K_{\pi,j}$$. Because this key
should be a one-time address we can consider that it can be used only once. So, if there is any included into the block
transaction exist with same key image then we faced the try of double-spending of some output. Finally, the RingCT
transaction (4 equals to type `RCTTypeBulletproof2`) consist of:

| type 	 | input                               	 | output                                                          	 | fee               	 | extra                      	 |
|--------|---------------------------------------|-------------------------------------------------------------------|---------------------|------------------------------|
| 4    	 | for each input $$i \in \{1...m\}$$: 	 | for each output $$j \in \{1...m\}$$:                            	 | public fee amount 	 | Tx public key $$rG$$, etc. 	 |
| 	      | - Ring members                      	 | - One time address $$K^O_j$$                                    	 | 	                   | 	                            |
| 	      | - MLSAG signature                   	 | - Output commitment $$C^b_j$$                                   	 | 	                   | 	                            |
| 	      | - Key image                         	 | - Encoded amount (see docs fro more info)                       	 | 	                   | 	                            |
| 	      | - Pseudo output $$\hat{C}^a_i$$     	 | - Range proof that committed amount lies in $$[0..2^64)$$ range 	 | 	                   | 	                            |
