---
layout: post
title: Block cipher mode of operation
---

Block cipher is an algorithm that performs encryption and decryption of the open text by blocks (for example of 128
bit). It's obvious that to encode text that with different sizes we need a separate high-level module that will perform
split and append operations on the text in a couple with some other transformations. Such algorithms are called _block
cipher modes_.

In general, let's define the block cipher as follows:

- __Encrypt__ $$Enc(m, k) = c$$ - function that encrypts text $$m$$ using key $$k$$ and outputs ciphertext $$c$$.
- __Decrypt__ $$Dec(c, k) = m$$ - function that decrypts ciphertext $$c$$ using key $$k$$ and outputs text $$m$$.

## ECB (electronic cookbook) mode

The most obvious cipher mode is an ECB mode that performs encryption for every block separately.

$$c_i = Enc(m_i, k)$$

This mode is considered to be the worst modes from all. If two parts of input text are equal (that appears really often
for real texts) it produces same ciphertext that can be used by hackers to block your cypher.

## CBC (cipher block chaining) mode

One of the most popular modes is a CBC mode that performs xor operation for every next block of open text with previous
ciphertext. This mode lacks the disadvantage of the ECB mode and outputs the different cipher texts even for same open
texts.

$$c_i = Enc(m_i \oplus c_{i-1}, k)$$

$$m_i = Dec(c_i. k) \oplus c_{i-1}$$

For the initial block encrypting it uses special _initialization vector_ that have to be unique for every new
encryption. If you use same initialization vectors for different encryptions it can cause same problems as in the ECB
mode.

Let's overview several approaches to generate the initialization vector.

- Simple usage of the counter 0,1,2,3... is not recommended because of really week randomness. Because of peculiarities
  of the real texts usage of this initialization vector will not increase security significant.
- A random initialization vector can be really safe for most cases, but it requires access to a really good random
  number generator. Also, it adds one more cipher block to every encryption that increases the size of resulting
  ciphertext.
- Usage of nonce values combines two previous methods. For nonce (number used only once) you can select for example a
  message number and then generate initialization vector $$c_0$$ by separate encryption of this value. Then, you can add
  this number to the resulting ciphertext in a raw format an on the decryption side it will be possible to generate your
  initialization vector.

## OFB (output feedback) mode

Using this mode we will not encrypt your text directly. Instead, we will use block cipher to generate the pseudo-random
bytes stream and combine it with open text by xor operation (as in one-time-pad cipher). Such an approach is also
referred to stream cipher.

$$k_i = Enc(k_{i-1}, k)$$

$$c_i = m_i \oplus k_i$$

It's easy to observe that such an approach also requires the initialization vector as in CBC mode. One of the advantages
of this algorithm is that encryption and decryption methods are the same. Also, it does not require the appending to
the text to fill all block size - we can use as much as needed.

One of the highest vulnerability of this mode is that usage of same initialization vectors cause more security risks
than in all previous modes. For example for texts $$m, \hat{m}$$ and ciphertexts $$c, \hat{c}$$ generated with same
initialization vectors: $$c \oplus \hat{c} = m \oplus k \oplus \hat{m} \oplus k = m \oplus \hat{m}$$. So, with the knowledge of
one of
open texts, hacker can easily recover the other text.

Also, if one of the $$k_i$$ appears twice, it will cause the whole chain to repeat. This problem is more likely to occur
after a large number of encryptions, so it is obviously necessary to limit the maximum number of encryptions per key.

## CTR (counter) mode

Counter mode can be determined as a modification of the OFB mode because it utilizes the same approaches.

$$k_i = Enc(nonce | i, k)$$

$$c_i = m_i \oplus k_i$$

This mode requires $$nonce | i$$ be the same size as block. Then, for example, for 128-bit block we can use 48-bit for
message number, 16 bit for any nonce data and 64 bit for counter $$i$$. Then, we are able to encrypt maximum $$2^{48}$$
messages ber one key as well as maximum size of one message $$2^{64}$$ bit.

## CBC vs CTR

- __Appending__: CBC requires appending to the text to fill the block, CTR - not.
- __Speed__: CTR allows concurrent calculation of ciphertext blocks.
- __Implementation__: CTR requires only encryption function to implement (decryption is the same).
- __Reliability__: if information about nonce leaks then hacker will be able to receive more information about message
  in case of CTR.
- __Nonce__: In most implementation CBC requires nonce as well as CTR.

## Collisions
We've made a lot of focus on the probability that two ciphertexts for same keys appears to be same. In all cases it easy
to check that such match gives more information to hacker about the open text. But what is the resulting probability of
such collision? Imagine we've encrypted $$M$$ blocks of text then we can select about $$M(M - 1)/2$$ pairs from them.
For the block size $$n$$ the probability of two blocks to be equal is $$2^{-n}$$, so the entire probability to receive
same ciphertexts will be
$$\frac{M(M-1)}{2^{n+1}}$$. This value reaches one if $$M ~ 2^{n/2}$$. So, for the 128-bit cipher, we will receive
collision with high probability after $$2^{64}$$ encryptions. This observation ca be referred to the birthday paradox.
