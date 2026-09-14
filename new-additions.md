CTF / Cybersecurity Knowledge Map

I researched this against current training curricula and references from Hack The Box ecosystem material, pwn.college, CryptoHack, PortSwigger, Ghidra, OWASP, MITRE ATT&CK, NIST, and CTF-oriented references. The categories are not universally standardized. A CTF organizer can put the same technique under rev, pwn, misc, crypto, or forensics depending on the challenge. The taxonomy below is therefore a practical learning map, not an official universal classification. 

The three levels mean:

Level 1: Foundation: understand and solve straightforward challenges.

Level 2: Practitioner: combine techniques and solve unfamiliar variants.

Level 3: Advanced: research-level techniques, complex chains, custom tooling, and difficult CTF problems.


This is deliberately much broader than just rev and cry.


---

0. Core foundations

This is the layer underneath almost everything.

Level 1

Computing

Binary vs decimal vs hexadecimal

Bits and bytes

Binary arithmetic

Two's complement

Signed vs unsigned integers

Integer overflow

Endianness

ASCII

Unicode

Character encodings

Base64

Hex encoding

URL encoding

Serialization

Compression


Linux

Filesystem

Processes

Users/groups

Permissions

Environment variables

Pipes

Redirection

Signals

/proc

/sys

Shell scripting

SSH

Basic networking commands


Programming

Python

C basics

Data structures

Functions

Pointers

Memory

Bitwise operators

File I/O

Sockets

Regular expressions


Networking

IP

TCP

UDP

DNS

HTTP

TLS

Ports

Routing

NAT

Client/server architecture


Level 2

Linux internals

Processes vs threads

Virtual memory

ELF

Shared libraries

Dynamic linking

System calls

IPC

Signals

ptrace

/proc internals

Network packet structure

TCP state

HTTP internals

TLS concepts

DNS internals

Binary serialization formats

Compression algorithms


Level 3

Kernel internals

Virtual memory implementation

CPU architecture

Cache hierarchy

MMU

Page tables

Calling conventions

ABI

Linkers/loaders

Compiler behavior

OS internals

Network protocol implementation

Hardware/software boundaries

Side-channel fundamentals


pwn.college's current curriculum itself places computing fundamentals, web security, reverse engineering, binary exploitation, communications, and cryptography into a progression rather than treating them as isolated skills. 


---

1. MATH

This is particularly important for crypto, but also useful for algorithms, reversing, exploitation and constraint solving.

Level 1: Algebra and number theory

Arithmetic

Fractions

Exponents

Logarithms

Equations

Functions

Sets

Basic probability

Prime numbers

Factorization

Divisibility

GCD

LCM

Euclidean algorithm

Extended Euclidean algorithm

Modular arithmetic

Modular addition

Modular multiplication

Modular exponentiation

Modular inverse

Congruences

Fermat's little theorem

Euler's totient function


Level 2: Mathematical cryptography

Chinese Remainder Theorem

Quadratic residues

Legendre symbol

Modular square roots

Groups

Rings

Fields

Finite fields

Polynomial arithmetic

Discrete logarithms

Probability distributions

Entropy

Information theory

Combinatorics

Birthday paradox

Collision probability


CryptoHack's modular arithmetic curriculum specifically progresses through GCD, extended GCD, modular arithmetic, modular inversion, quadratic residues, Legendre symbols, modular square roots and CRT. 

Level 3: Higher mathematics

Abstract algebra

Group theory

Ring theory

Field theory

Galois fields

Algebraic structures

Elliptic curves

Algebraic geometry basics

Lattice mathematics

Linear algebra for cryptography

Number-theoretic algorithms

Computational number theory

Probability theory

Information theory

Complexity theory


CryptoHack explicitly progresses beyond basic modular arithmetic into RSA, Diffie-Hellman, elliptic curves, lattices, isogenies and zero-knowledge proofs. 


---

2. CRY / Cryptography

This is your friend's cry.

Level 1

Encoding and classical crypto

ASCII

Hex

Base64

XOR

Caesar cipher

Substitution

Vigenère

Frequency analysis

One-time pad

Randomness basics


Symmetric crypto

Block ciphers

Stream ciphers

AES concepts

DES concepts

Keys

IVs

Nonces

Modes of operation

ECB

CBC

CTR

GCM


Hashes

MD5

SHA family

Hash properties

Collision

Preimage

Password hashing

Salt


Public-key crypto

RSA basics

Public/private keys

Modular exponentiation

Diffie-Hellman basics


Level 2

RSA

Key generation

Euler phi

CRT

Small exponent attacks

Small message attacks

Common modulus

Shared primes

Weak primes

Wiener attack

Hastad's broadcast attack

Franklin-Reiter

RSA padding concepts


Symmetric attacks

ECB leakage

CBC bit flipping

Padding oracle

IV manipulation

Keystream reuse

XOR vulnerabilities

Weak PRNGs

Bad nonce reuse

Authentication failures


Hash attacks

Length extension

Hash collisions

Merkle-Damgård construction

HMAC

Password cracking

Rainbow tables

Hash-based protocols


Level 3

Advanced RSA attacks

Lattice attacks

Coppersmith's method

Hidden-number problems

Advanced discrete logarithm attacks

Pohlig-Hellman

Pollard rho

Index calculus

Elliptic-curve cryptanalysis

Invalid-curve attacks

Small-subgroup attacks

Fault attacks

Side-channel attacks

Timing attacks

Differential cryptanalysis

Linear cryptanalysis

Padding-oracle exploitation

Advanced protocol attacks

Zero-knowledge proofs

Lattice-based cryptography

Isogenies

Post-quantum cryptography


CryptoHack's current challenge taxonomy includes symmetric ciphers, mathematics, RSA, Diffie-Hellman, elliptic curves, hashes, crypto-on-the-web, lattices, isogenies and ZKPs. 


---

3. REV / Reverse Engineering

(Luvliest one)

Level 1

Binary basics

What an executable is

Machine code

Assembly

Registers

Stack

Heap

Functions

Calling conventions

Control flow

Branches

Loops

Strings

Symbols

Imports/exports


x86-64

mov

lea

push

pop

call

ret

cmp

test

jmp

Conditional jumps

Arithmetic

Bitwise operations

Stack frames


Tools

file

strings

readelf

objdump

GDB

Ghidra

Hex editors


Ghidra's beginner material specifically covers code analysis, decompilation, byte viewing, symbols, function graphs and call trees. 

Level 2

Binary formats

ELF

PE

Mach-O

Sections

Segments

Relocations

GOT

PLT

Dynamic linking

Static linking


Analysis

Static analysis

Dynamic analysis

Cross references

Call graphs

Data-flow analysis

Control-flow graphs

Debugging

Breakpoints

Watchpoints

Memory inspection

Register inspection

Patching


Obfuscation

String encryption

Control-flow obfuscation

Packing

Anti-debugging

Junk instructions

Opaque predicates

Dynamic API resolution

Self-modifying code


Level 3

Advanced binary analysis

Symbolic execution

Concolic execution

SMT solving

Constraint solving

Binary instrumentation

Emulation

DBI

Function similarity

Binary diffing

Devirtualization

VM-based obfuscation

Anti-analysis

Anti-VM

Anti-debugging

Custom loaders

Packers

Unpacking

Compiler optimization analysis

Malware reverse engineering

Firmware reversing

Kernel reversing

Architecture-specific reversing

ARM

AArch64

MIPS

RISC-V

DSPs


Ghidra itself has advanced material covering scripting, plugins, binary formats, loaders, filesystem loaders, custom languages and analyzers. 


---

4. PWN / Binary Exploitation

pwn is not really an acronym. It comes from "own", meaning gaining control of a target.

Level 1

Memory layout

Stack

Heap

Buffer overflow

Segmentation faults

Stack smashing

NX

ASLR

PIE

RELRO

Canaries

Shellcode

Basic format strings

Basic GDB

Basic ROP


Level 2

Memory corruption

Stack overflow

Format string exploitation

Integer overflow

Signedness bugs

Use-after-free

Double free

Out-of-bounds read

Out-of-bounds write

Type confusion


Control-flow exploitation

ROP

ret2libc

ret2csu

ret2plt

ret2dlresolve

SROP

Stack pivots

GOT overwrites


Heap

glibc allocator

tcache

fastbins

unsorted bins

large bins

heap metadata

heap leaks


Level 3

Advanced heap exploitation

House of Force

House of Spirit

House of Orange

Tcache poisoning

FILE structure exploitation

FSOP

Kernel exploitation

Kernel UAF

Kernel heap exploitation

Race-condition exploitation

Sandbox escapes

Seccomp bypass

JIT exploitation

Browser exploitation

Microarchitectural exploitation

Spectre

Meltdown

Advanced ROP

Blind ROP

Heap feng shui


pwn.college currently separates Reverse Engineering, ROP, Dynamic Allocator Misuse, Program Exploitation, Race Conditions, Sandboxing, Kernel Security and Microarchitecture Exploitation into dedicated modules. 


---

5. WEB

This is probably the largest CTF category in terms of practical vulnerability classes.

Level 1

Web fundamentals

HTTP

HTTP methods

Headers

Cookies

Sessions

Same-origin policy

CORS

HTML

JavaScript

SQL

REST APIs


Vulnerabilities

SQL injection

XSS

Path traversal

Command injection

Authentication flaws

Authorization flaws

IDOR

File upload

Information disclosure


Tools

Browser DevTools

Burp Suite

curl

wget

ffuf

gobuster


PortSwigger explicitly recommends beginners start with server-side topics such as SQL injection, authentication, path traversal and command injection. 

Level 2

Blind SQLi

Out-of-band SQLi

SSRF

XXE

SSTI

CSRF

JWT vulnerabilities

OAuth vulnerabilities

WebSockets

CORS attacks

Prototype pollution

NoSQL injection

GraphQL attacks

Race conditions

Business logic flaws

Cache poisoning

Cache deception

HTTP Host header attacks

Insecure deserialization


Level 3

HTTP request smuggling

HTTP/2 attacks

Client-side desynchronization

Advanced cache poisoning

Advanced SSRF

Browser exploitation

DOM clobbering

Prototype pollution chains

Advanced OAuth attacks

Advanced JWT attacks

Web cache architecture attacks

Parser differential attacks

Deserialization gadget chains

Web LLM attacks

Multi-vulnerability exploit chains


PortSwigger's current Academy includes SQLi, XSS, CSRF, XXE, SSRF, request smuggling, SSTI, deserialization, OAuth, JWT, prototype pollution, GraphQL, race conditions, NoSQL, API testing, Web LLM attacks and web cache deception. 


---

6. API SECURITY

This deserves its own track rather than being buried under web.

Level 1

REST

JSON

HTTP APIs

Authentication

Authorization

API keys

Tokens

CRUD

Object IDs

Rate limits

API enumeration

Swagger/OpenAPI


Level 2

BOLA / IDOR

Broken authentication

Broken object-property authorization

Broken function-level authorization

Mass assignment

Excessive data exposure

Rate-limit bypass

SSRF

API inventory problems

Unsafe API consumption

GraphQL

WebSockets


Level 3

API gateway attacks

Microservice trust boundaries

OAuth/OIDC attacks

JWT cryptographic weaknesses

GraphQL batching abuse

GraphQL authorization flaws

Service-to-service authentication

Request smuggling across API gateways

Protocol confusion

Distributed race conditions

Complex API exploit chains


OWASP's API Security Top 10 explicitly covers BOLA, broken authentication, object-property authorization, unrestricted resource consumption, function-level authorization, sensitive business flows, SSRF, misconfiguration, inventory management and unsafe API consumption. 


---

7. FORENSICS / DFIR

Level 1

Files

File signatures

Metadata

file

strings

Hex inspection

Archives

Deleted files

Timestamps


Images

EXIF

JPEG structure

PNG structure

LSB steganography


Network

PCAP

Wireshark

TCP streams

DNS traffic

HTTP traffic


Level 2

Disk

Filesystem structures

NTFS

FAT

EXT

Disk images

Deleted files

File carving

MFT

Windows Registry


Memory

RAM dumps

Processes

DLLs

Handles

Network connections

Command history


Network

Protocol reconstruction

DNS analysis

HTTP reconstruction

Credential extraction

Suspicious traffic

Beacon detection


Level 3

Advanced memory forensics

Kernel memory

Malware artifacts

Timeline reconstruction

Anti-forensics

Deleted filesystem recovery

Registry archaeology

Browser forensics

Cloud forensics

Mobile forensics

Advanced PCAP reconstruction

Incident reconstruction

Attribution analysis


NIST's forensic guidance organizes forensic work around sources such as files, operating systems, network traffic and applications, while modern NIST guidance describes collection, examination, analysis and reporting as core phases. 


---

8. STEGANOGRAPHY

Usually found under forensics or misc.

Level 1

Hidden text

Hidden files

Metadata

LSB

Image channels

Appended data

ZIP inside image

Base64 hidden in text

Audio spectrograms


Level 2

JPEG stego

PNG chunks

Palette manipulation

Alpha channels

WAV stego

MP3 metadata

PDF hiding

Polyglots

File format tricks

Bit-plane analysis


Level 3

Custom stego algorithms

Statistical detection

DCT-domain steganography

Advanced audio stego

Video stego

Steganalysis

Multi-layer payloads

Stego + crypto combinations


CTF steganography commonly involves hiding data inside images or audio, including LSB techniques. 


---

9. NETWORK / TRAFFIC

Level 1

TCP/IP

UDP

DNS

HTTP

TLS basics

ICMP

Ports

PCAP

Wireshark

Packet filtering

TCP streams


Level 2

ARP

DHCP

DNS attacks

HTTP internals

TLS analysis

Protocol reconstruction

Custom protocols

VPN traffic

Network authentication

IPv6

Routing


Level 3

Protocol reverse engineering

Encrypted traffic analysis

Traffic fingerprinting

Side-channel traffic analysis

Covert channels

DNS tunneling

Network exploitation

Advanced packet crafting

Protocol fuzzing

Custom network protocol exploitation



---

10. OSINT

Level 1

Search operators

Search engines

Username searches

Email investigation

Domain lookup

WHOIS

DNS

Metadata

Reverse image search

Social media investigation


Level 2

Infrastructure mapping

Subdomain discovery

Historical DNS

Certificate transparency

GitHub investigation

Leaked credentials awareness

Wayback Machine

Image geolocation

EXIF analysis

Username correlation

Digital identity correlation


Level 3

Infrastructure attribution

Passive DNS

Historical infrastructure

Advanced geolocation

Satellite imagery analysis

Corporate relationship mapping

Large-scale OSINT automation

Graph-based identity analysis

Multi-source intelligence correlation



---

11. MOBILE

Especially relevant if you're interested in Android development.

Level 1

Android

APK structure

Manifest

Activities

Services

Broadcast receivers

Intents

Permissions

Dalvik/ART

Java/Kotlin

Smali

JADX

APKTool

Frida basics


iOS

IPA structure

Objective-C

Swift

Mach-O

Entitlements

Basic runtime analysis


Level 2

Android static analysis

Android dynamic analysis

Native libraries

JNI

Frida hooks

SSL pinning

Root detection

Emulator detection

Local storage

SQLite

SharedPreferences

KeyStore

Deep links

IPC

Intent vulnerabilities

iOS runtime analysis


Level 3

Native ARM reversing

Advanced Frida

Anti-Frida

Anti-debugging

Obfuscation

Runtime patching

Native crypto extraction

Key extraction

Advanced IPC exploitation

Kernel interfaces

Jailbreak/root bypass

Mobile malware analysis


OWASP's MASTG explicitly covers mobile application testing, reverse engineering, tampering, cryptography, Android/iOS, static/dynamic analysis, Smali, Java decompilation and native-code disassembly. 


---

12. MALWARE / MALWARE RE

Level 1

Malware terminology

Static analysis

Dynamic analysis

Strings

Imports

Hashes

Processes

Files

Registry

Network connections

Basic sandboxing

IOC extraction


Level 2

PE analysis

DLL analysis

Persistence

Process injection

Command and control

Packing

Obfuscation

Configuration extraction

API tracing

Behavioral analysis

YARA

Sigma


Level 3

Advanced unpacking

Custom packers

Anti-analysis

Anti-debugging

Anti-VM

Reflective loading

Manual mapping

Shellcode analysis

Rootkits

Kernel malware

C2 protocol reversing

Malware family clustering

Automated malware analysis


MITRE ATT&CK's current enterprise framework organizes adversary behavior around tactics including reconnaissance, initial access, execution, persistence, privilege escalation, credential access, discovery, lateral movement, collection, command and control and exfiltration. 


---

13. WINDOWS / AD

Level 1

Windows architecture

Users/groups

NTFS

Processes

Services

Registry

PowerShell

Windows authentication

SMB

RDP

Basic privilege escalation


Level 2

Active Directory

Domains

Domain controllers

Kerberos

LDAP

NTLM

Group Policy

SPNs

Delegation

Shares

ACLs


Attacks

Kerberoasting

AS-REP roasting

Pass-the-hash

Pass-the-ticket

Credential dumping

Token impersonation

Lateral movement

Privilege escalation


Level 3

AD CS attacks

Resource-based constrained delegation

Unconstrained delegation

Constrained delegation

Golden ticket

Silver ticket

DCSync

DCShadow

NTLM relay

Advanced ACL abuse

Forest/domain trust attacks

Enterprise AD attack paths



---

14. LINUX / SYSTEM EXPLOITATION

Level 1

Linux permissions

SUID

SGID

Cron

PATH hijacking

Environment variables

Services

Basic privilege escalation

/proc

Capabilities


Level 2

Linux capabilities

Sudo abuse

Kernel versions

Shared libraries

LD_PRELOAD

Containers

Docker permissions

Namespace concepts

cgroups

Service exploitation


Level 3

Kernel exploitation

Kernel UAF

Kernel race conditions

Namespace escapes

Container escapes

eBPF attacks

LSM internals

Kernel heap exploitation

Hypervisor attacks



---

15. CLOUD

Level 1

AWS basics

Azure basics

GCP basics

IAM

Storage buckets

Security groups

Instance metadata

API keys

Cloud networking


Level 2

IAM privilege escalation

S3/bucket attacks

Metadata service attacks

SSRF → cloud credentials

Lambda

Serverless

Containers

Kubernetes

Secrets management

Cloud logging


Level 3

Cross-account attacks

IAM policy abuse

Role chaining

Kubernetes RBAC attacks

Container escapes

Cloud control-plane attacks

CI/CD compromise

Supply-chain attacks

Cloud persistence

Multi-cloud attack paths



---

16. HARDWARE / IoT / FIRMWARE

Level 1

Binary formats

Firmware extraction

UART

SPI

I²C

GPIO

Serial interfaces

Basic embedded Linux

Firmware filesystem extraction


Level 2

ARM reversing

MIPS reversing

Firmware emulation

U-Boot

JTAG

Flash memory

EEPROM

Hardware debugging

Embedded web interfaces

Firmware vulnerabilities


Level 3

Secure boot

TrustZone

Hardware security modules

Fault injection

Glitching

Side-channel attacks

Firmware exploitation

Bootloader exploitation

SoC security

Hardware-assisted exploitation

Chip-level reversing



---

17. BLOCKCHAIN / WEB3

Level 1

Blockchain basics

Transactions

Wallets

Public/private keys

Hashes

Ethereum

Smart contracts

Solidity

EVM basics


Level 2

Smart contract vulnerabilities

Reentrancy

Access-control bugs

Integer issues

Oracle manipulation

Flash loans

Delegatecall

Proxy contracts

Signature replay

Transaction ordering


Level 3

EVM internals

Bytecode reversing

DeFi protocol attacks

MEV

Cross-chain bridges

Consensus attacks

Cryptographic protocol failures

Advanced Solidity exploitation

Formal verification

Smart-contract symbolic execution



---

18. AI / LLM SECURITY

This is increasingly its own security domain.

Level 1

Prompt injection

System prompts

Tool permissions

Data leakage

Unsafe output handling

Basic jailbreak concepts

RAG security

Basic model abuse


Level 2

Indirect prompt injection

RAG poisoning

Tool-use attacks

Agent hijacking

Excessive agency

Insecure output handling

Model extraction

Sensitive information disclosure

Embedding attacks

Vector database security


Level 3

Agentic attack chains

Multi-agent exploitation

Model supply-chain attacks

Fine-tuning poisoning

Backdoors

Adversarial examples

Model extraction at scale

Membership inference

Data poisoning

Automated red teaming

LLM-to-infrastructure attack chains

AI-assisted vulnerability discovery


PortSwigger now includes Web LLM attacks among its web-security training topics, showing how LLM-specific attack surfaces are becoming integrated into mainstream application security training. 


---

19. FUZZING

Level 1

What fuzzing is

Mutation

Random input

Input validation

Crash detection

Corpus

Basic Python fuzzers


Level 2

Coverage-guided fuzzing

AFL

AFL++

libFuzzer

Sanitizers

Grammar-based fuzzing

Protocol fuzzing

File-format fuzzing

Harnesses

Crash triage


Level 3

Stateful fuzzing

Distributed fuzzing

Hybrid fuzzing

Symbolic execution + fuzzing

Snapshot fuzzing

Kernel fuzzing

Browser fuzzing

Protocol state machines

Automated exploitability analysis

Fuzzing custom hardware



---

20. MISC

The cursed category.

misc can contain essentially anything that doesn't cleanly belong elsewhere.

Level 1

Scripting

Automation

Encoding

Weird file formats

Simple programming puzzles

Basic OSINT

Basic stego

Custom text formats

QR/barcode

Logic puzzles


Level 2

Custom protocols

Esoteric languages

Sandbox puzzles

Automation

Constraint solving

Unicode weirdness

Parser behavior

Polyglots

Weird machines

Custom virtual machines


Level 3

Novel research

Mathematical puzzles

Custom cryptosystems

Custom interpreters

VM reversing

Hardware/software combinations

Multi-category chains

Challenges requiring you to invent the methodology


This is why misc cannot really be "mastered" as a fixed syllabus.


---

21. PROGRAMMING / AUTOMATION

This isn't always a CTF category, but it is one of the most important skills.

Level 1

Python

Bash

Regex

File processing

HTTP requests

JSON

Sockets

Parsing

Automation


Level 2

pwntools

Scapy

Requests

BeautifulSoup

Capstone

Keystone

Unicorn

Z3

Frida

GDB scripting

Ghidra scripting


Level 3

Custom debuggers

Custom disassemblers

Binary instrumentation

Symbolic execution

SMT modeling

Automated exploit generation

Custom protocol analyzers

Automated vulnerability discovery

Research tooling


This is particularly relevant to you because your existing Python/SWE background can become the glue connecting nearly every other category.


---

22. THE ACTUAL THREE-TIER MAP

If you want the whole thing compressed into the mental model:

LEVEL 1
│
├── Computing
├── Linux
├── Python
├── C
├── Networking
├── Algebra
├── Modular arithmetic
│
├── Web
├── Crypto
├── Rev
├── Pwn
├── Forensics
├── OSINT
├── Stego
├── Network
└── Misc

Then:

LEVEL 2
│
├── Advanced Web
├── API Security
├── RSA / DH / ECC
├── Binary Analysis
├── x86-64
├── GDB / Ghidra
├── Memory Corruption
├── ROP
├── Heap
├── Malware
├── Mobile
├── Windows / AD
├── Linux Exploitation
├── Cloud
├── Firmware
├── Fuzzing
└── Advanced Forensics

Then:

LEVEL 3
│
├── Advanced Cryptanalysis
├── Elliptic Curve Cryptanalysis
├── Lattices
├── Zero Knowledge
├── Symbolic Execution
├── Binary Instrumentation
├── VM / Obfuscation Reversing
├── Advanced Heap Exploitation
├── Kernel Exploitation
├── Browser Exploitation
├── Malware Research
├── Hardware Security
├── Firmware Exploitation
├── Cloud Attack Paths
├── Advanced AD
├── Protocol Research
├── Side Channels
├── AI Security
└── Novel / Research Techniques


---

What I would actually study first

Do not attempt to study all 22 branches simultaneously.

For the specific goal you just described, I'd make your primary route:

FOUNDATIONS
    ↓
MATH
    ↓
CRYPTO
    ↓
C + MEMORY
    ↓
ASSEMBLY
    ↓
REV
    ↓
PWN
    ↓
FUZZING
    ↓
ADVANCED CRYPTO
    ↓
REV + CRYPTO

And run Web alongside it because web security gives you a completely different way of thinking about vulnerabilities.

A particularly good practical progression is visible in the existing training ecosystems. pwn.college currently separates computing, web, communications, cryptography, reverse engineering and binary exploitation into hands-on modules, while CryptoHack has dedicated progression through modular arithmetic, symmetric crypto, public-key crypto and elliptic curves. 

For REV, use Ghidra plus x86-64 and GDB. OpenSecurityTraining's x86-64 material explicitly uses GDB and a binary-bomb-style reverse-engineering exercise as preparation for later reverse-engineering and exploitation work. 

For WEB, PortSwigger is unusually useful because the material and interactive labs are organized by vulnerability class and progression level. 

For MOBILE, OWASP MASTG is the natural deep reference because it combines theory, testing techniques, tools, reverse engineering and practical crackmes. 

And for crypto specifically, CryptoHack is almost exactly aligned with what you said you want to learn: mathematics → modular arithmetic → symmetric crypto → public-key crypto → elliptic curves → harder categories such as lattices and ZKPs. 

---

That is the skill stack we're building toward.
