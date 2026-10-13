# Cybersecurity: Cryptography

Cryptography is the study and practice of protecting information and communications using mathematical methods. It helps systems keep information confidential, detect changes, prove who created a message, and establish shared secrets across networks. It is not a substitute for access control, secure software, backups, or careful key handling: a sound algorithm can still be defeated by a stolen key, a flawed protocol, or an unsafe implementation. NIST’s key-management guidance treats protection and lifecycle management of keys as central parts of cryptographic security.[1]

## Learning goals and prerequisites

By the end, you should be able to distinguish encoding, encryption, hashing, and signing; explain the basic jobs of symmetric and public-key cryptography; recognize why nonces and authentication matter; read a simple cryptographic protocol description; and assess common implementation mistakes without trying to attack real systems.

You need only basic arithmetic, comfort with binary/hexadecimal notation, and a willingness to reason about what an attacker can observe or change. Helpful later: modular arithmetic, probability, and elementary programming. **Vocabulary:** plaintext is the readable input; ciphertext is encrypted output; a key is secret or public material controlling a cryptographic operation; a nonce is a value intended to be used once (sometimes called an IV, or initialization vector, depending on the construction); a cryptographic primitive is a building block such as a cipher or hash; a protocol is the larger sequence of messages and checks that uses such building blocks.

## 1. First separate representation from protection

Text encodings represent data in a different form. ASCII maps a limited set of characters to numbers; hexadecimal writes bytes using digits 0–9 and A–F; Base64 maps bytes into printable characters. These formats do **not** hide information. For example, Base64-decoding a secret is not breaking encryption: the representation is reversible without a key.

Classical ciphers are useful for learning, not for protecting modern data. A Caesar cipher shifts letters by a fixed amount, so trying the small set of shifts recovers the message. A substitution cipher replaces each letter consistently; language patterns and frequency analysis can reveal likely mappings. Vigenère varies shifts using a repeating key, but repetition and language structure can still leak information. XOR combines bits: for equal-length strings, `C = P XOR K`, and applying the same K again returns P. This identity is useful for understanding stream ciphers, but XOR with a repeated or predictable key is unsafe.

A one-time pad can provide perfect secrecy under strict conditions: a uniformly random key at least as long as the message, kept secret, used once, and never reused. Those requirements make it impractical for most routine systems. Reusing the pad cancels the key when ciphertexts are XORed together, exposing relationships between messages. This illustrates a lasting lesson: randomness and key use rules are part of the security design, not decoration.

## 2. Security goals and threat models

Confidentiality means unauthorized parties should not learn message content. Integrity means unauthorized changes are detected. Authentication means a receiver can establish that data came from an entity holding the relevant credential or key. These goals differ: encryption alone does not necessarily detect alteration, and a hash alone does not prove who computed it. A threat model states what an adversary can observe, alter, or control and what the system must protect.

Cryptography is usually designed to remain secure even when the algorithm is public; secrecy should reside in well-protected keys, not a hidden algorithm. Correctness depends on the full system: randomness, key storage, unique nonces where required, error handling, software libraries, and protocol design. Never invent a new cipher or glue primitives together casually; prefer a maintained, documented cryptographic library and a protocol designed for the intended purpose.[2]

## 3. Symmetric encryption: one shared secret

Symmetric cryptography uses the same secret key, or closely related secret keys, for encryption and decryption. It is efficient for protecting large amounts of data, but both parties need a safe way to obtain the key. AES is a standardized block cipher: AES-128, AES-192, and AES-256 each transform 128-bit blocks, with the number indicating the key length.[3] A block cipher alone handles only one block; a **mode of operation** specifies how to apply it to longer data.

Electronic Codebook (ECB) encrypts each block independently. Identical plaintext blocks under one key produce identical ciphertext blocks, revealing patterns; it should not be used for ordinary data protection. Cipher Block Chaining (CBC) and Counter mode (CTR) address some limitations of block processing but do not, by themselves, authenticate the message. Adding encryption without a sound authentication design can leave room for undetected tampering. OWASP recommends authenticated modes where available and warns against ECB in ordinary use.[2]

Authenticated encryption with associated data (AEAD) aims to provide confidentiality and integrity together. The inputs are a key, nonce, plaintext, and optional associated data; associated data is authenticated but remains visible. The decryption operation returns plaintext only when authentication succeeds, otherwise failure.[4] For instance, a record’s visible version number or routing metadata might be associated data, while its private payload is encrypted. Treat verification failure as a hard stop: do not parse, display, or act on candidate plaintext before the tag is verified. Both ends must supply exactly the same associated-data bytes; changing a version field or its encoding also makes verification fail.

**Nonce discipline is critical.** RFC 5116 requires distinct nonces for distinct invocations under a given key for the specified interface, except for constructions that use only zero-length nonces.[4] With common AES-GCM use, accidental nonce reuse under the same key can seriously undermine security: the counter-mode keystream repeats, exposing relations between plaintexts, and authentication guarantees can also be damaged. NIST’s GCM specification sets limits on message sizes and invocation usage, so uniqueness alone is not a complete usage policy.[9] Do not devise a counter scheme or random nonce policy by intuition. Counters need coordinated allocation across processes, restarts, and backups; random nonces have collision probability that grows with the number generated. Use the library’s documented construction and per-key limits. An IV and a nonce are related concepts, but exact requirements vary by algorithm—do not assume that every IV must be secret or that every nonce requirement is interchangeable.

## 4. Hashes, MACs, and password storage

A cryptographic hash maps arbitrary-length input to a fixed-length digest. Useful security properties include **preimage resistance** (given a digest, infeasible to find an input producing it), **second-preimage resistance** (given one input, infeasible to find a different input with the same digest), and **collision resistance** (infeasible to find any two different inputs with the same digest). A collision is not the same thing as recovering an input. Hashes are used for integrity checks and as components in protocols, but an unkeyed hash by itself does not authenticate a sender: anyone can recompute it after changing a message.

MD5 and SHA-1 are legacy hash functions and should not be selected for new collision-sensitive designs. The SHA-2 and SHA-3 families are distinct standardized hash families; choose an appropriate, maintained construction for the application rather than treating all “hashing” as interchangeable. A **message authentication code (MAC)** uses a secret key to authenticate a message. HMAC is a widely specified construction combining a hash function with a secret key; unlike a plain digest, a party without the key cannot ordinarily create a valid tag.

Password storage is a special case. Passwords are often low-entropy human choices, so a fast general-purpose hash lets guesses be tested cheaply. A password-hashing function deliberately raises the cost of each guess; a unique random **salt** (stored alongside the resulting verifier) prevents identical passwords from sharing identical stored outputs and frustrates reuse of precomputed tables. RFC 9106 specifies Argon2, a memory-hard password-hashing function, and recommends Argon2id where the user is uncertain about the type or side-channel exposure is a concern.[5] OWASP gives a practical Argon2id baseline (19 MiB memory, two iterations, one lane), but parameters must be tuned for the service’s actual capacity and login latency.[10] A verifier encoding should retain its algorithm and parameters so they can be upgraded after a successful login. Do not build your own password storage scheme. Salting is not encryption, and a salt does not make a weak password strong. An optional pepper is a separately stored secret that adds key-management and recovery obligations; it does not replace salts or a password-hashing function.[10]

## 5. Public-key cryptography and trust

Public-key systems use a public key that may be shared and a corresponding private key that must be protected. **RSA** relies on arithmetic involving large integers and modular exponentiation; explaining its mathematics helps learners, but real RSA encryption requires a secure padding scheme and careful implementation. A raw textbook RSA operation is not a safe application design. **Diffie–Hellman key agreement** lets two parties derive a shared secret over a public channel, but unauthenticated key agreement does not by itself establish who is on the other end: an active intermediary could establish separate secrets with each party.

A digital signature is made using a private signing key and checked with the corresponding public key. It supports integrity checking and signer authentication, subject to a trustworthy link between public key and identity. Signatures are not encryption: they do not conceal the signed message. NIST’s Digital Signature Standard specifies signature algorithms and describes signatures as a means to detect unauthorized modification and authenticate the signatory.[6] A certificate and its validation chain can bind a public key to an identity; trust configuration and validation rules matter as much as the signature arithmetic. Validation also depends on the expected name, validity period, constraints, and applicable status policy. A valid signature only proves that the corresponding private key signed those bytes; it does not establish that the key belongs to the expected person or that the content is safe.

In practice, public-key cryptography often authenticates parties or establishes key material, and symmetric cryptography then protects bulk traffic. TLS 1.3 is an example of a complete protocol: its specification uses authenticated-encryption record protection and a handshake that negotiates parameters and establishes traffic secrets.[7] In the usual full TLS handshake, certificate-based signatures authenticate the server while key agreement derives shared secrets; traffic keys then protect records. A certificate warning bypass or incorrect hostname validation can defeat the identity guarantee even if encryption works. This is why “use AES” is not a complete security plan: the system also needs authentication, key agreement, nonce management, a transcript, and correct verification behavior.

## 6. Attacks as lessons in design

The repository plan names attacks such as ECB leakage, CBC bit flipping and padding oracles, keystream reuse, weak pseudorandom generators, RSA small-message and shared-prime cases, hash length extension, and side channels. Treat these as **conceptual failure modes to recognize**, not procedures for probing real services. ECB leaks repeated structure; reusing a stream-cipher keystream exposes XOR relationships; predictable randomness can make keys or nonces guessable; and a padding oracle occurs when distinguishable error behavior leaks information about decryption. Length-extension behavior can affect some Merkle–Damgård hash constructions when an application incorrectly uses a plain hash as a message authenticator; HMAC is designed to address this class of construction mistake.

Other failures arise outside the math. A timing side channel leaks information through operation duration; fault attacks exploit incorrect computation under induced faults. Defensive implementations use reviewed libraries, authenticated constructions, constant-time operations where required, safe error handling, and key separation. Cryptographic keys need controlled generation, storage, distribution, rotation, and retirement; NIST explicitly treats these as key-management functions.[1] A compromise of the key can defeat otherwise strong encryption.

## 7. From classical public-key systems to post-quantum ideas

Advanced study may explore RSA factorization assumptions, discrete logarithms, elliptic curves, lattices, zero-knowledge proofs, and cryptanalysis methods. These topics demand mathematical foundations and careful bounds on what an attack model assumes. A toy example with small numbers can teach modular arithmetic, but it does not estimate the security of real parameters.

Post-quantum cryptography studies schemes intended to resist attacks by quantum computers as well as classical ones. NIST FIPS 203 specifies ML-KEM, a key-encapsulation mechanism (KEM): parties use it to establish a shared secret, which can then be used by symmetric algorithms. It is based on the Module Learning with Errors problem, and the standard’s publication page records a potential-updates notice; deployment choices should therefore consult current official guidance and errata rather than assume standards never change.[8] Post-quantum does not mean “quantum encryption,” and changing one algorithm in isolation does not make an entire protocol quantum-safe. Organizational migration starts by discovering where vulnerable public-key cryptography is used—in software, devices, certificates, and external services—then prioritizing sensitive data and dependencies, testing interoperability, and planning updates.[11] Inventory and testing are more useful than swapping a primitive in isolation: protocol and vendor support must also exist.

## Safe-lab boundary

Hands-on experiments belong only in a local toy program, an intentionally vulnerable local lab, an official practice environment, your own system, or a system covered by explicit written authorization. Do not test cryptographic weaknesses against public services, attempt to recover others’ secrets, or use these concepts to steal data, credentials, evade detection, establish persistence, deploy malware, or damage systems. Prefer synthetic keys and sample messages; reset local labs after use.

## Practice: small, safe checks

1. **Encoding versus encryption.** Convert the ASCII bytes for `OK` into hexadecimal, then Base64-encode them with a local tool or a short script. Expected check: decoding either representation reproduces the same two bytes without any key. Explain why this is not confidentiality.
2. **XOR identity.** Choose two short byte strings of equal length and compute `P XOR K`, then XOR the result with K again. Success: you recover P exactly. Repeat the same key against two toy messages and explain why XORing the ciphertexts cancels the key.
3. **Hash properties.** Hash two different sample messages using a maintained local cryptography tool. Change one input byte. Success: digests differ; explain that this observation is not a proof of collision resistance, and that the hash alone does not identify its author.
4. **AEAD conceptual check.** In a local example using a maintained library, encrypt a dummy message with an AEAD interface and associated data. Alter one bit of the ciphertext or associated data before decryption. Success: verification fails and the application does not release unauthenticated plaintext. Use only library-provided nonce handling and follow its documentation.
   - **Nonce-policy review:** for a toy design note the key scope, nonce construction, restart/multi-process behavior, and library usage limits. Success: explain how uniqueness is maintained or why the library owns that responsibility; do not reuse a real key or nonce.
5. **Password-verifier design review.** Inspect a sample record format containing algorithm, parameters, salt, and verifier (not a real credential). Success: you can identify why the salt need not be secret, why each account needs a distinct salt, and why a slow memory-hard password hash differs from reversible encryption.
6. **Protocol reading.** Read the TLS 1.3 overview and list which jobs are handled by the handshake and which by record protection. Success: your list distinguishes peer/parameter establishment from protected application records; it does not claim that encryption alone proves identity.[7]
7. **Trust-chain check.** In a local or official training example, identify the expected server name and trust anchor for a certificate. Success: explain why a valid signature without correct identity validation is insufficient. Never disable certificate checks on real services.

## Common mistakes to avoid

- Confusing Base64 or hexadecimal with encryption, or a hash with encryption.
- Assuming encryption automatically provides integrity or proves who sent a message.
- Reusing a nonce with a key when the algorithm requires uniqueness, or treating nonce generation as an afterthought.
- Using ECB, raw RSA, home-grown ciphers, or ad hoc encrypt-then-hash constructions.
- Storing passwords with a fast general-purpose hash, omitting salts, or believing salts must be secret.
- Hard-coding keys, keeping them indefinitely without a rotation plan, or assuming a strong algorithm rescues poor key management.[1] [2]
- Treating a successful toy demonstration as evidence that a production implementation is secure.

## Next steps

1. Practice bytes, binary, hexadecimal, XOR, and modular arithmetic with toy inputs.
2. Learn hash properties, HMAC, password hashing, and randomness; distinguish their goals.
3. Study symmetric encryption and AEAD, focusing on authentication and nonce rules.
4. Learn public/private keys, key agreement, signatures, certificates, and TLS at a protocol level.
5. Read key-management guidance and review the lifecycle of a test key from generation through retirement.
6. Only then approach advanced cryptanalysis, side channels, zero-knowledge proofs, lattices, and post-quantum standards—using synthetic data and authorized labs.

## References

[1]: https://csrc.nist.gov/pubs/sp/800/57/pt1/r5/final "NIST SP 800-57 Part 1 Revision 5: Recommendation for Key Management—General"
[2]: https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html "OWASP Cryptographic Storage Cheat Sheet"
[3]: https://csrc.nist.gov/pubs/fips/197/final "NIST FIPS 197: Advanced Encryption Standard (AES)"
[4]: https://www.rfc-editor.org/rfc/rfc5116 "RFC 5116: An Interface and Algorithms for Authenticated Encryption"
[5]: https://www.rfc-editor.org/rfc/rfc9106 "RFC 9106: Argon2 Memory-Hard Function for Password Hashing and Proof-of-Work Applications"
[6]: https://csrc.nist.gov/pubs/fips/186-5/final "NIST FIPS 186-5: Digital Signature Standard"
[7]: https://www.rfc-editor.org/rfc/rfc8446 "RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3"
[8]: https://csrc.nist.gov/pubs/fips/203/final "NIST FIPS 203: Module-Lattice-Based Key-Encapsulation Mechanism Standard"
[9]: https://csrc.nist.gov/pubs/sp/800/38/d/final "NIST SP 800-38D: Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode (GCM) and GMAC"
[10]: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html "OWASP Password Storage Cheat Sheet"
[11]: https://www.nccoe.nist.gov/applied-cryptography/migration-to-pqc "NIST NCCoE: Migration to Post-Quantum Cryptography"
