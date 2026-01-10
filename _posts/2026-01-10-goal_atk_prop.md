---
title: Intro to the security properties (GOAL-ATK-PROP)
aside:
  toc: false
tags:
  - ciphers
  - signatures
---

Let's define the following list of configurations for each attack:

- **GOAL** — the goal of an attack;
- **ATK** — a type or a certain attack name;
- **PROP** — additional property for an attack (if needed).

We say that a cryptosystem is $v(\lambda)$-GOAL-ATK-PROP safe if for any PPT adverseries $\mathcal{A}$ the following
holds:

$$
\texttt{Adv}_\mathcal{A}^{\textnormal{GOAL-ATK-PROP}} \leq v(\lambda),
$$

where $v$ is a negligible function, $\lambda$ is a security parameter and \texttt{Adv} denotes advantage of an adversary
$\mathcal{A}$, or more simple — if $p$ is a probability of a successful attack we put $\texttt{Adv} = |p -
\frac{1}{2}|$.

Each GOAL-ATK-PROP basically assumes an existence of some protocol or a “game” played by the honest oracle and the
attacker. Often, the goal is defined by properties, and properties in turn define an answer or action that the attacker
should give/perform. The attack, in turn, defines which input the attacker receives and which requests from the attacker
can be executed by the oracle. Thus, in practice, we often specify only property and attack, mostly in the form PROP-ATK.

## Properties

Here are some of the most commonly used properties for ciphers and digital signatures:

- **Indistinguishability (IND)** — an attacker aims to distinguish the ciphertext of two plaintexts. Basically, it
  assumes a “game” where an attacker knows two messages $m_0, m_1$, receives the ciphertext $c$, and should determine
  whether it corresponds to $m_0$ or $m_1$.

- **Semantic Security (SEM)** — an attacker aims to extract any useful information from the ciphertext. It is basically
  equivalent to the IND, but less convenient for proving.

- **Pseudorandomness (PR)** — an attacker aims to distinguish a ciphertext from a random string. It also assumes a
  “game” where an attacker receives a string (or several strings) and should guess whether it is random or not.

- **Integrity (INT)** — an attacker aims to create a valid ciphertext.

- **Existential Unforgeability (EUF)** — an attacker aims to create a valid signature for any new message.

- **Strong Unforgeability (SUF)** — a stronger version of EUF. An attacker aims to either create a valid signature for
  any new message or create a new valid signature for any old message.

There are also some other properties, like key recovery or message recovery, which are less used because they
automatically follow from the IND or EUF resistance, which are easier to prove.

## Attacks

### Ciphers

- **No-message attack (NMA)** or **key-only attack (KOA)** — an attacker receives only public parameters of the
  cryptosystem.  
  Examples: *key-recovery attack in the NMA model*, *EUF-NMA*.

- **Ciphertext-only attack (COA)**, or **known ciphertext attack (KCA)** — an attacker receives ciphertext of several
  messages encrypted by the same key.  
  Examples: *IND-COA*, *INT-COA*, *message-* or *key-recovery in the COA model*.

- **Known-plaintext attack (KPA)** — an attacker has access to one or many plaintexts and their ciphertexts encrypted by
  the same key.  
  Examples: *IND-KPA* (the ciphertext and plaintexts to distinguish with overhearing probability will differ from the
  known ones).

- **Plaintext-checking attack (PCA)** — an attacker can sample a pair of entries from the ciphertext and plaintext
  spaces accordingly, and ask an oracle if the ciphertext is a valid encryption of the submitted plaintext (always
  assuming the usage of the same secret key).  
  Examples: *IND-PCA* (but can not ask for the messages and the received ciphertext to distinguish).

- **Chosen-plaintext attack (CPA)** — an attacker has access to one or many plaintexts and their ciphertexts encrypted
  by the same key, and also can ask an oracle to give the ciphertexts for the selected set of plaintexts.  
  In the adaptive version (Adaptive CPA), an attacker can ask several times until it gives the final answer for the
  game, aiming to receive more information from each response. In most cases, CPA assumes the adaptive CPA.  
  Examples: *IND-CPA* (or *CPA1* for non-adaptive CPA and *CPA2* for the adaptive CPA), *INT-CPA*.

- **Chosen-ciphertext attack (CCA)** — an attacker can ask an oracle to decrypt a set of selected ciphertexts.  
  In the adaptive version (Adaptive CCA), an attacker can ask several times until it gives the final answer for the
  game, aiming to receive more information from each response. In most cases, CCA assumes the adaptive CCA.  
  Examples: *IND-CCA* (or *CCA1* for non-adaptive CCA and *CCA2* for the adaptive CCA).

There are also various other types of attacks, such as combinations of CPA and CCA, as well as their adaptive versions,
and side-channel attacks (where an attacker has access, for example, to physical measurements of the user's computer).
The most used (and de facto standard for modern ciphers) are CCA2 (IND-adaptive-CCA) and INT-CPA (INT-adaptive-CPA). The
last one is also often called INT-CTXT, meaning the ability to modify the existing valid ciphertext given by the oracle
to receive another valid ciphertext (used for symmetric ciphers).

While resistance to CCA is the strongest security level, the resistance to other attacks follows automatically. For
example, if the cipher is IND-CCA secure, it is automatically IND-CPA secure. In turn, if the cipher is IND-CPA secure,
it is automatically IND-PCA secure and so on.

### Digital signatures

- **No-message attack (NMA)** or **key-only attack (KOA)** — an attacker receives only public parameters of the
  cryptosystem.  
  Examples: *EUF-NMA*.

- **Known-message attack (KMA)** — an attacker has access to one or many messages and their signatures by the same
  key.  
  Examples: *EUF-KPA*.

- **Chosen-message attack (CMA)** — an attacker has access to one or many messages and their signatures by the same key,
  and also can ask an oracle to give the signature for the selected set of messages.  
  In the adaptive version (Adaptive CMA), an attacker can ask several times until it gives the final answer for the
  game, aiming to receive more information from each response. In most cases, CMA assumes the adaptive CMA.  
  Examples: *EUF-CMA*, *SUF-CMA*.

The minimal requirement for all modern digital signatures is EUF-CMA, but the desired is always the SUF-CMA resistance.
