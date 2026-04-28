# Trivium Stream Cipher

A Python implementation of the **Trivium stream cipher** combined with a **One-Time Pad (OTP)** encryption scheme. 

---

## Overview

Trivium is a hardware-oriented stream cipher designed by Bart Preneel and Christophe De Cannière, submitted to the **eSTREAM competition** (EU ECRYPT Network, 2004–2008). It is standardized under **ISO/IEC 29192-3**.

This implementation covers:
- Full Trivium key stream generation
- OTP encryption using the generated key stream
- OTP decryption to recover the original plaintext

---

## Specifications

| Parameter | Value |
|---|---|
| Key size | Up to 80 bits (hex input) |
| IV / Nonce size | 80 bits (randomly generated) |
| Internal state | 288 bits |
| Warm-up rounds | 1152 |
| Key stream length | 8 × plaintext length (bits) |
| Encryption scheme | One-Time Pad (XOR) |


## How to Run

### Requirements
- Python 3.14
- Jupyter Notebook or JupyterLab (no external libraries needed — uses only `os`)

### Steps

1. Open `main.ipynb` in Jupyter.
2. Run **Cell 1** and **Cell 2** first to load all helpers and the Trivium core.
3. Run **Cell 3** for encryption (Part 1).
4. Run **Cell 4** for decryption (Part 2).
5. Optionally run **Cell 5** to verify output against the lab's sample values.

---

## Usage

### Part 1 — Encryption

```
Enter plaintext: HelloCryptography
Use default key (all 1-bits)? [y/n]: n
Enter key in HEX (up to 20 hex chars = 80 bits): 646A616D61656A797A61

Generated Nonce/IV:  721ED8A325EF88583E2D
Output Key Stream:   225CED4D0AB13A68BF782D851EFD13D1
Output Ciphertext:   6A39812165F24811CF0C42E26C9C63B9D4
```

> **Copy and save the Key Stream** — you will need it for decryption.

### Part 2 — Decryption

```
Enter Key Stream (hex):   225CED4D0AB13A68BF782D851EFD13D1
Enter Ciphertext (hex):   6A39812165F24811CF0C42E26C9C63B9D4

Output Plaintext: HelloCryptography
```

---

## How It Works

### 1. Initialization
The 288-bit internal state is populated as follows:

```
S[001–080]  ← Key (LSB-first, zero-padded to 80 bits)
S[094–173]  ← IV  (LSB-first, randomly generated)
S[286–288]  ← 111 (hardcoded)
All others  ← 0
```

### 2. Warm-up (1152 rounds)
Before generating any output, Trivium runs **1152 rounds** (= 4 × 288) to fully diffuse the key and IV throughout the state. Output bits during this phase are discarded.

### 3. Each Round
Per the Trivium spec, each round computes:

```
t1 = S[066] ⊕ S[093]
t2 = S[162] ⊕ S[177]
t3 = S[243] ⊕ S[288]

output_bit = t1 ⊕ t2 ⊕ t3

t1 = t1 ⊕ (S[091] ∧ S[092]) ⊕ S[171]
t2 = t2 ⊕ (S[175] ∧ S[176]) ⊕ S[264]
t3 = t3 ⊕ (S[286] ∧ S[287]) ⊕ S[069]

Right-shift state by 1:
  S[001] ← t3
  S[094] ← t1
  S[178] ← t2
```

The **AND gates** (∧) introduce nonlinearity, which is the source of Trivium's cryptographic strength.

### 4. Encryption / Decryption (One-Time Pad)

```
Encrypt:  c = m ⊕ k
Decrypt:  m = c ⊕ k
```

Where `m` = plaintext bits, `k` = Trivium keystream bits, `c` = ciphertext bits.
XOR is self-inverse, so the same operation encrypts and decrypts.

---

## Key Design Decisions

| Decision | Reason |
|---|---|
| LSB-first key/IV loading | Required by the Trivium specification |
| `os.urandom()` for nonce | Cryptographically secure random generation |
| Key defaults to all 1-bits | As specified in the lab activity |
| Hex-only key input | Lab requirement — no plaintext keys allowed |
| Keystream length = 8 × plaintext length | One key bit per plaintext bit for perfect secrecy |

---

## Sample Test Values (from Lab PDF)

| Field | Value |
|---|---|
| Plaintext | `HelloCryptography` |
| Key | `646A616D61656A797A61` |
| Nonce/IV | `721ED8A325EF88583E2D` |
| Key Stream | `225CED4D0AB13A68BF782D851EFD13D1` |
| Ciphertext | `6A39812165F24811CF0C42E26C9C63B9D4` |

Run **Cell 5** in the notebook to automatically verify these values.

---

## References

- [Trivium Specification — cr.yp.to](https://cr.yp.to/streamciphers/trivium/desc.pdf)
- [The Stream Cipher Trivium Explained — YouTube](https://www.youtube.com/watch?v=YCnUKCki_rg)
- ISO/IEC 29192-3 — Lightweight Cryptography: Stream Ciphers
- eSTREAM Project — EU ECRYPT Network (2004–2008)