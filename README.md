# Quantum-Safe Cryptography (Post-Quantum Cryptography)

<div align="center">

![IBM Quantum](https://img.shields.io/badge/IBM%20Quantum-052FAD?style=flat-square&logo=ibm&logoColor=white)
![Python](https://img.shields.io/badge/Python%203.9-1a1a2e?style=flat-square&logo=python&logoColor=4fc3f7)
![NIST](https://img.shields.io/badge/NIST-PQC%20Standards-4fc3f7?style=flat-square)
![Day 11](https://img.shields.io/badge/Day%2011-Complete-4fc3f7?style=flat-square)
![Day 12](https://img.shields.io/badge/Day%2012-Loading...-555555?style=flat-square)

</div>

<br/>

<div align="center">
<i>Part of the IBM Quantum 20-Day Learning Sprint · VIT Chennai</i>
</div>

---

## Overview

Quantum-safe cryptography — also called post-quantum cryptography (PQC) — refers to cryptographic algorithms that remain secure against attacks from both classical and quantum computers.

The urgency is not hypothetical. Quantum computers running **Shor's algorithm** can break RSA and ECC encryption in polynomial time. More critically, adversaries can collect encrypted data today and decrypt it once quantum hardware matures — a strategy known as **"harvest now, decrypt later"**. The transition to quantum-safe systems must begin now, before cryptographically relevant quantum computers exist.

---

## Table of Contents

- [Introduction](#introduction)
- [Why We Need Quantum-Safe Cryptography](#why-we-need-quantum-safe-cryptography)
- [Key Concepts](#key-concepts)
- [Mathematical Foundations](#mathematical-foundations)
- [Python Implementation](#python-implementation)
- [NIST Standardization](#nist-standardization)
- [CRYSTALS Suite](#crystals-suite)
- [Security Concepts](#security-concepts)
- [Hybrid Cryptography](#hybrid-cryptography)
- [Use Cases](#use-cases)
- [Summary](#summary)

---

## Introduction

### The Quantum Threat

Classical computers operate on bits — 0 or 1. Quantum computers operate on qubits, which can exist in superposition and leverage interference and entanglement to explore solution spaces exponentially faster for specific problems.

This is not a general speedup. For most tasks, quantum computers offer no advantage. But for the mathematical problems underlying modern cryptography, the impact is severe.

### Classical vs Quantum Attacks

| Attack Type | Classical Complexity | Quantum Complexity |
|-------------|---------------------|--------------------|
| RSA-2048 factoring | Exponential | Polynomial (Shor) |
| ECC discrete log | Exponential | Polynomial (Shor) |
| AES-256 brute force | 2²⁵⁶ | 2¹²⁸ (Grover) |
| SHA-256 collision | 2¹²⁸ | 2⁸⁵ (BHT) |

### Shor's Algorithm

Shor's algorithm solves integer factorization and discrete logarithm problems in **polynomial time** on a quantum computer. Both RSA and ECC derive their security from the classical hardness of these exact problems. A sufficiently large quantum computer running Shor's algorithm renders both algorithms insecure.

### Grover's Algorithm

Grover's algorithm provides a quadratic speedup for unstructured search problems. It does not break symmetric cryptography outright — it halves the effective key length. AES-256 becomes effectively AES-128-equivalent under Grover's attack, which remains computationally infeasible. The mitigation is simply to use longer keys.

---

## Why We Need Quantum-Safe Cryptography

### Vulnerable Algorithms

| Algorithm | Based On | Quantum Attack | Status |
|-----------|---------|----------------|--------|
| RSA | Integer factoring | Shor's algorithm | ❌ Broken |
| Diffie-Hellman | Discrete logarithm | Shor's algorithm | ❌ Broken |
| ECC | Elliptic curve discrete log | Shor's algorithm | ❌ Broken |
| DSA | Discrete logarithm | Shor's algorithm | ❌ Broken |

### Resilient Algorithms

| Algorithm | Type | Quantum Impact | Action Required |
|-----------|------|---------------|-----------------|
| AES-256 | Symmetric | Grover halves key strength | Use 256-bit keys |
| SHA-256 | Hash | BHT reduces collision resistance | Use SHA-512 |

### The "Harvest Now, Decrypt Later" Threat

Adversaries intercepting and storing encrypted traffic today can hold it until a cryptographically relevant quantum computer (CRQC) becomes available — estimated within 10–15 years. Data with long-term sensitivity (medical records, state secrets, financial history) encrypted today with RSA or ECC is at risk. This is why the migration to post-quantum cryptography must begin immediately, not when quantum computers arrive.

---

## Key Concepts

### Computational Complexity

**NP (Non-deterministic Polynomial time):** Problems whose proposed solutions can be verified in polynomial time. Many cryptographic problems belong to this class.

**NP-hard:** Problems at least as hard as the hardest problems in NP. No known efficient algorithm exists, even on quantum computers.

**NP-complete:** Problems that are both in NP and NP-hard. Solving one efficiently would solve all NP problems.

Classical asymmetric cryptography (RSA, ECC) relies on NP-intermediate problems — hard classically, but solvable by quantum algorithms. Post-quantum cryptography relies on NP-hard problems, which remain hard even for quantum computers.

### Average-Case vs Worst-Case Hardness

A problem is **worst-case hard** if only isolated instances are hard. A problem is **average-case hard** if most randomly drawn instances are hard.

Cryptographic security requires average-case hardness — an adversary must face a hard instance every time, not just occasionally. Lattice problems satisfy this requirement and additionally admit worst-case to average-case reductions, providing provable security guarantees.

---

## Mathematical Foundations

### Lattice-Based Cryptography

A lattice is a regular grid of points in high-dimensional space, defined by a set of basis vectors. Lattices can exist in hundreds or thousands of dimensions — the higher the dimension, the harder the associated problems.

Two fundamental hard problems underpin lattice cryptography:

**Shortest Vector Problem (SVP):** Find the shortest non-zero vector in the lattice. SVP is NP-hard in the worst case. Even with lattice-basis reduction algorithms (e.g., LLL), finding the exact shortest vector in high dimensions is computationally infeasible.

**Closest Vector Problem (CVP):** Given a target point not on the lattice, find the nearest lattice point. CVP is also NP-hard and is the basis for several cryptographic constructions.

Both SVP and CVP remain hard for quantum computers — no known quantum algorithm solves them in polynomial time.

### Learning With Errors (LWE)

LWE combines linear algebra with controlled noise to create a one-way function. The core idea:

```
Given:  A · s + e = b  (mod q)
Where:  A = public random matrix
        s = secret vector (private key)
        e = small random error vector
        b = public output

Task:   Recover s from (A, b)

Without noise e → trivially solvable (Gaussian elimination)
With noise e    → NP-hard, even for quantum computers
```

The error vector `e` is drawn from a discrete Gaussian distribution. Its presence makes the system computationally indistinguishable from random, providing the cryptographic hardness.

---

## Python Implementation

A minimal demonstration of LWE-based encryption and decryption.

### Parameter Setup

```python
import numpy as np

# Public parameters
n = 8        # security parameter (vector dimension)
q = 127      # modulus
N = int(1.1 * n * np.log(q))  # number of LWE samples
sigma = 1.0  # Gaussian noise standard deviation
```

### Noise Generation

```python
def sample_noise(sigma, modulus):
    """Sample error from discrete Gaussian distribution."""
    return round(np.random.randn() * sigma ** 2) % modulus
```

### Key Generation (Alice)

```python
# Private key: random vector in Z_q^n
alice_private_key = np.random.randint(0, high=q, size=n)

# Public key: N samples of (a, b = a·s + e mod q)
alice_public_key = []
for _ in range(N):
    a = np.random.randint(0, high=q, size=n)
    e = sample_noise(sigma, q)
    b = (np.dot(a, alice_private_key) + e) % q
    alice_public_key.append((a, b))
```

### Encryption (Bob)

```python
def encrypt(message_bit, public_key, q, N):
    """
    Encrypt a single bit using LWE public key.
    message_bit: 0 or 1
    """
    r = np.random.randint(0, 2, N)  # random binary mask

    sum_a = np.zeros(n, dtype=int)
    sum_b = 0
    for i in range(N):
        sum_a += r[i] * public_key[i][0]
        sum_b += r[i] * public_key[i][1]

    sum_a = [x % q for x in sum_a]
    ciphertext = (
        sum_a,
        (message_bit * int(np.floor(q / 2)) + sum_b) % q
    )
    return ciphertext
```

### Decryption (Alice)

```python
def decrypt(ciphertext, private_key, q):
    """
    Decrypt a single bit using LWE private key.
    """
    a_dot_s = np.dot(ciphertext[0], private_key) % q
    b_minus_as = (ciphertext[1] - a_dot_s) % q
    return round((2 * b_minus_as) / q) % 2
```

### End-to-End Example

```python
# Bob encrypts a message bit
message_bit = 1
ciphertext = encrypt(message_bit, alice_public_key, q, N)

# Alice decrypts
recovered_bit = decrypt(ciphertext, alice_private_key, q)

assert message_bit == recovered_bit
print(f"Original: {message_bit} | Decrypted: {recovered_bit}")
```

---

## NIST Standardization

In 2016, NIST initiated a public competition to standardize post-quantum cryptographic algorithms. Following a six-year evaluation by the global cryptography community, four algorithms were selected in 2022 and formally standardized in 2024.

| Algorithm | Family | Purpose | FIPS Standard |
|-----------|--------|---------|---------------|
| CRYSTALS-Kyber | Lattice (Module-LWE) | Key encapsulation | FIPS 203 |
| CRYSTALS-Dilithium | Lattice (Module-LWE) | Digital signatures | FIPS 204 |
| FALCON | Lattice (NTRU) | Compact digital signatures | — |
| SPHINCS+ | Hash-based | Digital signatures | FIPS 205 |

Three of four finalists are lattice-based, reflecting the maturity and security confidence of lattice mathematics. SPHINCS+ serves as an alternative built on a different mathematical structure, providing diversity in case lattice assumptions are weakened.

---

## CRYSTALS Suite

The CRYSTALS (Cryptographic Suite for Algebraic Lattices) suite includes two algorithms based on Module-LWE — a variant of LWE that balances efficiency and security assurance.

### CRYSTALS-Kyber (Key Encapsulation Mechanism)

Kyber is designed for **key exchange**, replacing RSA and Diffie-Hellman in this role. It implements a Key Encapsulation Mechanism (KEM) rather than general-purpose encryption.

```
Workflow:
1. Bob generates a key pair (public key, private key)
2. Alice uses Bob's public key to encapsulate a shared secret → ciphertext
3. Bob uses his private key to decapsulate → recovers shared secret
4. Both parties now hold the same secret, used for symmetric encryption
```

### CRYSTALS-Dilithium (Digital Signatures)

Dilithium replaces RSA and ECDSA for digital signature applications — verifying the authenticity and integrity of messages, software, and certificates.

---

## Security Concepts

### IND-CPA Security

**Indistinguishability under Chosen-Plaintext Attack.** A scheme is IND-CPA secure if an adversary who can request encryptions of chosen plaintexts cannot distinguish between encryptions of two different messages. LWE-based schemes achieve IND-CPA security inherently, because the random error vector `e` makes each encryption of the same plaintext produce a different ciphertext.

### IND-CCA Security

**Indistinguishability under Chosen-Ciphertext Attack.** A stronger notion — the adversary can also request decryptions of chosen ciphertexts. Kyber achieves IND-CCA2 security through the Fujisaki-Okamoto transform applied to an IND-CPA secure base scheme.

### Why LWE Is Inherently Randomized

Traditional schemes like RSA are deterministic — the same plaintext always produces the same ciphertext, requiring external randomization (padding). LWE-based encryption is non-deterministic by construction. Two sources of randomness are built into the protocol:

- The noise vector `e` sampled at key generation
- The random binary mask `r` sampled fresh at each encryption

This eliminates the need for padding schemes and simplifies the security proof.

---

## Hybrid Cryptography

Hybrid cryptography combines a classical algorithm (RSA, ECC) with a post-quantum algorithm (Kyber, Dilithium) in parallel. The resulting system is secure if either component remains unbroken.

**Why this matters:**

Post-quantum algorithms are relatively new. Despite extensive analysis, the cryptographic community has not had decades to study them as it has RSA. One NIST candidate (SIKE/SIDH) was broken in 2022 — after years of scrutiny — by a classical attack. Hybrid approaches mitigate this risk: if the post-quantum component is compromised, the classical component still provides protection, and vice versa.

Hybrid cryptography also eases compliance — existing standards mandating RSA or ECC can be satisfied while simultaneously adding quantum-safe protection.

---

## Use Cases

| Use Case | Current Risk | Quantum-Safe Solution |
|----------|-------------|----------------------|
| TLS/HTTPS (web traffic) | RSA/ECC key exchange | Kyber KEM |
| Software code signing | RSA/ECDSA signatures | Dilithium |
| Long-term data archives | "Harvest now, decrypt later" | Kyber + AES-256 |
| Government communications | State-level quantum adversaries | CRYSTALS suite |
| PKI certificates | RSA signatures | Dilithium / FALCON |
| IoT devices | ECC (compact keys) | FALCON (compact signatures) |

---

## Summary

The migration to quantum-safe cryptography is not a future concern — it is a present operational requirement. Asymmetric cryptosystems based on integer factoring and discrete logarithms (RSA, ECC, Diffie-Hellman) are structurally incompatible with a post-quantum world. Symmetric cryptography (AES-256, SHA-512) requires only parameter adjustments.

Lattice-based cryptography, grounded in the provably hard problems of SVP, CVP, and LWE, provides the strongest mathematical foundation for post-quantum systems. The CRYSTALS suite — standardized by NIST in 2024 — represents the first generation of drop-in replacements for RSA and ECC.

The transition is already underway. Google, Apple, Cloudflare, and IBM have begun integrating post-quantum algorithms into their products. Organizations handling sensitive long-lived data should begin cryptographic inventory and migration planning now.

---

## Sprint Progress

```
Day 01  ──  ✅  Qiskit setup · Hello Quantum · first IBM cloud circuit
Day 02  ──  ✅  Superposition · Entanglement · Multi-gate circuits
Day 03  ──  ✅  Gates deep-dive · Grover's algorithm
Day 04  ──  ✅  VQE · parametric circuits · COBYLA optimizer
Day 05  ──  ✅  QAOA · MaxCut · optimal partition found
Day 06  ──  ✅  Quantum Error Mitigation · noise models · ZNE
Day 07  ──  ✅  All 5 experiments on real IBM hardware · ibm_torino
Day 08  ──  ✅  QSVM · ZZFeatureMap · Iris dataset
Day 09  ──  ✅  VQC · Variational Quantum Classifier
Day 10  ──  ✅  Project 1 · QML Classifier · MNIST · v1.0
Day 11  ──  ✅  Quantum-Safe Cryptography · LWE · NIST PQC standards
Day 12  ──  ⬡   Quantum Finance · Portfolio Optimization · QAOA
·
·
Day 20  ──  ·   Final push · 50+ applications · LinkedIn article
```

---

## References

- [NIST Post-Quantum Cryptography Standards](https://csrc.nist.gov/projects/post-quantum-cryptography)
- [CRYSTALS-Kyber Specification](https://pq-crystals.org/kyber/)
- [CRYSTALS-Dilithium Specification](https://pq-crystals.org/dilithium/)
- [IBM Quantum Safe Cryptography Course](https://quantum.cloud.ibm.com/learning/en/courses/quantum-safe-cryptography)
- [Regev, O. — On Lattices, Learning with Errors (2005)](https://dl.acm.org/doi/10.1145/1060590.1060603)

---

<div align="center">

[![GitHub](https://img.shields.io/badge/Akhila707-181717?style=flat-square&logo=github)](https://github.com/Akhila707)
&nbsp;·&nbsp;
[![IBM Quantum](https://img.shields.io/badge/IBM%20Quantum-052FAD?style=flat-square&logo=ibm&logoColor=white)](https://quantum.ibm.com)

</div>