# Cybersecurity: Core Foundations

## What this subject is—and why it matters

Cybersecurity is the practice of understanding and managing risks to computers, networks, software, and information. It is not a single product or a contest in finding bugs: it combines technical safeguards with decisions about what matters, who may do what, how problems are noticed, and how services are restored. NIST’s Cybersecurity Framework (CSF) organizes outcomes into six connected functions: **Govern, Identify, Protect, Detect, Respond, and Recover**. These are a flexible map for risk management, not a universal checklist or a strict sequence. [1]

The stakes include privacy, safety, reliable services, money, and trust. A small flaw can have large effects when systems depend on one another. Foundational knowledge helps you explain how a failure happened, choose proportionate protections, and verify a fix rather than merely memorizing tool commands. Security is therefore a continuing process: identify important assets and threats, reduce likely harm, notice what prevention misses, and prepare to respond and recover.

## Safe-lab boundary

Do hands-on security work only in intentionally vulnerable local labs, official practice environments, systems you own, or systems covered by explicit written authorization. Stay within the stated scope and stop if you encounter real user data, unexpected systems, or effects outside the exercise. Do not test public addresses, accounts, or networks merely because they are reachable. The exercises below use arithmetic, local files, and conceptual diagrams; later practical work should remain in a safe lab.

## Prerequisites and vocabulary

No prior security experience is assumed. Comfort using a computer and willingness to read simple code help; advanced mathematics is not required at the start. Work through computing, operating-system, programming, and networking foundations before specializing. The supplied roadmap identifies binary/decimal/hexadecimal, bits and bytes, integer representation and overflow, endianness, ASCII/Unicode and encodings, Base64/hex/URL encoding, serialization and compression; Linux filesystems, processes, users/groups, permissions, environment variables, pipes, redirection, signals, `/proc`, `/sys`, shell scripting, SSH, and basic network commands; Python and C basics, data structures, functions, pointers, memory, bitwise operators, file I/O, sockets, and regular expressions; plus IP, TCP/UDP, DNS, HTTP, TLS, ports, routing, NAT, and client/server architecture.

These topics need not all be mastered before beginning. Learn enough to follow data from representation, through a program and operating system, across a network, and back. Later study can deepen Linux internals (threads, virtual memory, ELF, shared libraries, dynamic linking, system calls, IPC and `ptrace`), packet structure and protocol state, and eventually kernel internals, CPU architecture, caches, MMU/page tables, calling conventions, ABI, linkers/loaders, compiler behavior, and side-channel fundamentals. A source curriculum such as pwn.college likewise presents computing foundations as a basis for later areas such as web security, reverse engineering, binary exploitation, communications, and cryptography. [6]

**Vocabulary to keep straight:** an **asset** is something valuable to protect; a **threat** is a possible cause of harm; a **vulnerability** is a weakness that could be used or triggered; an **impact** is the consequence; and **risk** is a way of reasoning about likelihood and impact in context. **Authentication** asks who or what is presenting an identity; **authorization** asks what that identity is allowed to do. **Confidentiality** limits disclosure, **integrity** protects against unauthorized or accidental change, and **availability** supports timely, reliable access. These three aims are often called the CIA triad. **Encoding** changes representation, not secrecy; **encryption** uses a key to protect confidentiality; **hashing** maps data to a digest and is not, by itself, reversible encryption.

Risk is not a precise universal number. A useful beginner assessment states its assumptions: what threat event is considered, which weakness or predisposing condition matters, how plausible the event is over a stated period, and what consequences follow. For example, “a lost laptop might expose unencrypted project files, causing a confidentiality impact” is more actionable than “laptop risk is high.” Consider safety, privacy, service disruption, recovery cost, and obligations—not just money. NIST describes risk assessment as decision support; estimates depend on available evidence and context, and should be revised when systems or threats change. [7]

## A learning sequence: from bits to risk

### 1. Understand representations before interpreting data

Computers store and move bit patterns. A bit has two possible values; a byte is commonly eight bits. The same bits can stand for an integer, text, an instruction, or part of a network message depending on the format and context. Hexadecimal is a compact way to write binary values: each hex digit represents four bits. For example, decimal 13 is binary `1101` and hexadecimal `D`.

Representation matters for security because software parses data at boundaries. Signed and unsigned integers interpret some bit patterns differently; fixed-width arithmetic can overflow, wrapping or otherwise behaving according to the language and operation. Endianness describes byte order for multi-byte values. Text encodings map numbers to characters; Unicode assigns code points, while formats such as UTF-8 encode those code points as bytes. Confusing bytes with characters can produce truncation, validation, or comparison bugs.

Base64, hex, and URL encoding are representational transformations, not encryption. Anyone who knows the format can generally decode them. Serialization turns structured values into a byte representation; compression reduces representation size. Security-sensitive parsers must agree on boundaries and formats: ambiguous or malformed input can be interpreted differently by different components. Do not assume a string is safe because it looks obscure or encoded.

### 2. Learn what the operating system enforces

An operating system manages processes, memory, files, devices, and network access. A **process** is a running program with its own execution context; **threads** are execution paths within a process. Users and groups help assign identity and permissions. The principle of **least privilege** means giving each person or program only the access needed for its task. File permissions, service accounts, and process boundaries are practical controls, but configuration mistakes can weaken them. A practical decision is whether a task needs an administrator-level identity at all: everyday browsing and document work usually should not require elevated privileges. Separate service identities also limit the damage if one service is misused. Least privilege reduces the scope of possible harm; it does not prevent bugs or replace backups and monitoring.

Linux practice should emphasize observation and safe local experiments: inspect files and permissions, understand which user owns a process, and trace how input and output move through pipes and redirection. Environment variables can configure programs but may also expose sensitive values if mishandled. Signals request process actions; `/proc` and `/sys` expose system and kernel-related information through virtual filesystems. Do not treat every file or process as yours to modify. Shell scripts automate commands, so check what each command changes before running it. SSH is a remote-login protocol; use it only for systems where you have permission and protect its credentials and keys.

### 3. Follow data through a program

Programs receive input, transform it, and produce effects. Functions and data structures organize that work; file I/O and sockets connect programs to storage and networks. C makes memory and pointers visible: a pointer refers to a memory location, and mistakes in bounds, lifetime, or type assumptions can cause unsafe behavior. Python hides many low-level details but still has parsing, validation, and authorization risks. Bitwise operators manipulate individual bits and are useful for flags and protocol fields.

A secure design treats input as untrusted until validated for the intended meaning and context. Validation is not a substitute for safe APIs or correct authorization. For example, a web application should not construct a database command by concatenating user text into executable query syntax; parameterized queries keep data distinct from instructions. Validation should be specific to the field (such as a date range or identifier), reject malformed values clearly, and happen server-side even when a browser also validates. Output encoding is context-specific too: data inserted into HTML is not automatically safe in a script or URL context. This is a defensive principle, not an invitation to probe a live application. OWASP’s Top 10 is an awareness document for web application risks; its 2025 categories include broken access control, misconfiguration, software supply-chain failures, cryptographic failures, and injection. It is a consensus awareness list, not a complete catalog or a guarantee that unlisted flaws are unimportant. [3]

### 4. Build a network mental model

A client requests a service; a server responds. IP addresses identify network endpoints for routing, while ports distinguish services or conversations at a host. Routers forward traffic between networks; network address translation (NAT) changes address information at a boundary and is not, by itself, a security policy. DNS maps names to data such as addresses. TCP provides an ordered byte stream with connection state and retransmission behavior; UDP sends datagrams without TCP’s delivery and ordering guarantees. Applications choose protocols according to their needs.

HTTP defines request/response conventions used by web applications. TLS can protect a channel above a transport: TLS 1.3’s specification describes server authentication, confidentiality, and integrity for established traffic; client authentication is optional. Its handshake authenticates the communicating party as configured, negotiates parameters, and establishes keying material, while the record protocol protects traffic. The guarantee relies on correct protocol implementation and certificate/name validation by the client; a secure channel to the wrong or malicious endpoint is not useful. TLS does not generally hide traffic volume or timing, and encryption in transit does not prove that application behavior is safe, a user is authorized, or a device is uncompromised. [5]

A packet is not simply “the Internet”: it has layers and fields interpreted by protocols and endpoints. Later, study headers, TCP state, DNS behavior, HTTP internals, certificate validation, binary serialization, and compression algorithms. When troubleshooting, distinguish name-resolution, routing, transport, TLS, and application failures rather than assuming every connection error has the same cause.

### 5. Apply security reasoning: identity, access, and risk

Authentication factors are commonly grouped as something you know, have, or are. Multifactor authentication (MFA) combines distinct factor types; asking for two passwords is not two factors. A password manager can help users maintain distinct secrets. NIST’s digital identity guidance describes authenticators, account binding, recovery, and phishing resistance. In particular, manually entered one-time codes are not considered phishing-resistant in that guidance because the code is not bound to the specific verifier/session and may be relayed. A phishing-resistant option based on cryptographic protocol binding reduces dependence on a user noticing a deceptive page; availability, recovery and device loss still need planning. Keep a second approved authenticator where practical and protect recovery channels, since attackers may target account recovery as an alternate route. [4] CISA’s public guidance recommends phishing awareness, unique passwords, MFA, and updates as protective habits; its linked page is archived, so consult current official advice for policy details. [2]

Authorization must be checked for each sensitive action and object, not inferred merely from a successful login. A user who can sign in should not automatically be able to read every record or administer every setting. Design for least privilege, deny access by default when appropriate, and test both allowed and denied cases in a local lab.

Risk reasoning begins with context: what asset matters, what could go wrong, how plausible is it, and what would the impact be? Record uncertainty rather than pretending to know a precise probability. Compare risks using a consistent scale only when that helps prioritize; a low-likelihood event with severe safety consequences may deserve attention. Choose safeguards by considering expected risk reduction, cost, usability, and dependencies, then note residual risk and who accepts it. A vulnerability is not the same as an incident; an incident is an event that compromises or threatens security. NIST CSF 2.0 is deliberately outcomes-oriented: it does not prescribe a single checklist or order. GOVERN sets context, priorities, roles, and oversight; IDENTIFY informs choices across the cycle; PROTECT, DETECT, RESPOND, and RECOVER reinforce one another. [1, 7]

### 6. Think beyond prevention

Prevention is essential, but defenses should assume that errors and compromises can still occur. **Detection** uses relevant logs and signals to identify unusual events; **response** contains and analyzes an incident; **recovery** restores affected assets and services. Logs are useful only when the right events are recorded, timestamps can be correlated, retention is adequate, and access is protected; excessive collection can create privacy and storage risks. A response plan should name decision-makers, escalation contacts, communication channels, and recovery priorities, and should be exercised as a tabletop scenario. For a suspected incident, follow the organization’s plan and preserve evidence; do not improvise disruptive actions on systems you do not administer. Backups should be protected from the same failure or account compromise as production, and restoration should be tested on a schedule. CISA recommends offline, encrypted backups and regular tests because ransomware may target accessible backups. [8]

Cryptography supports confidentiality, integrity, and authentication, but using a familiar algorithm is not enough. Key generation, storage, rotation, protocol choices, certificate validation, and implementation all matter. Prefer well-reviewed standard mechanisms and maintained libraries rather than inventing cryptography. A hash can help detect changes if the trusted digest is protected; an unauthenticated digest does not prove who produced the file. A digital signature can support origin and integrity checks under an appropriate trust model, but does not encrypt the signed content.

Security is also a property of design and supply chains. A dependency or update can introduce risk; systems should track components, apply trusted updates, and review configuration. Fast-changing threat reports, product defaults, and tools should be checked against current maintained documentation. Stable concepts—least privilege, careful parsing, authentication versus authorization, and layered recovery—outlast individual tool interfaces.

## Worked examples

**Encoding versus encryption.** Suppose a note is represented in Base64. Decoding returns the original bytes; there is no secret key and no confidentiality. If the note must be private, use an appropriate encryption design and protect its key. This distinction prevents a common mistake: mistaking an unusual-looking representation for protection.

**A login is not an access decision.** Imagine a library account successfully authenticates. Authentication establishes that the user controls an accepted authenticator; authorization still decides whether that user may view a particular borrower record or change account settings. A well-designed system checks permissions at the operation and object boundary.

**A small risk review.** For a laptop containing project files, list the asset (files), plausible threat events (loss, theft, malicious software, accidental deletion), weaknesses (unpatched software, broad account access, no tested backup), and impacts (exposure or lost work). State assumptions—for example, “loss during travel is plausible; disclosure would be serious”—then select proportionate safeguards such as screen lock and disk protection, updates, limited privileges, and a tested backup. Decide what signal could reveal loss or compromise, who would be notified, and how work would be restored. This is structured reasoning, not a claim that any one control guarantees safety.

## Practice tasks and answer checks

1. **Number systems (paper only).** Convert decimal 42 to binary and hexadecimal. Check: binary `101010`; hex `2A` (since `2×16 + 10 = 42`).
2. **Encoding check (local toy data).** Write a short harmless word as UTF-8 bytes, then express the bytes in hex. Check: each byte is represented by two hex digits; changing to hex has not hidden the word from someone who can decode it.
3. **Permissions reasoning.** For a fictional report readable only by its owner and a designated group, state who should have read access and who should not. Check: access is limited to the intended owner/group; unrelated accounts and broadly available permissions are excluded. Do not change permissions on shared or system files as an exercise.
4. **Network flow sketch.** Draw a browser request to a named HTTPS service: name resolution, IP routing, transport connection, TLS handshake, then HTTP request/response. Check: DNS, routing, TCP, TLS, and HTTP are distinct steps/layers; TLS protection does not itself decide application authorization.
5. **Risk register (fictional service).** Name one asset, threat event, weakness, impact, and assumption about likelihood; then propose one prevention, detection, response, and recovery measure. Check: entries form a plausible chain; the estimate admits uncertainty; recovery includes a restoration test; and residual risk is stated rather than claiming perfection.
6. **Authentication review.** Explain why password plus another password is not MFA, and why a manually typed one-time code can still be phished. Check: the factors are the same type, and the typed code is not bound to a particular verifier/session according to NIST guidance. [4]
7. **Least-privilege decision (fictional app).** A report-generation service only needs to read a designated folder and write generated reports. Describe a narrowly scoped account and one thing it should not be allowed to do. Check: it does not run as a general administrator or read unrelated users’ files; identify that this limits blast radius but does not eliminate application defects.
8. **Recovery tabletop (fictional).** Imagine a critical service is unavailable and recent backups may be affected. List who decides priority, what should be checked before restore, and how restoration will be verified. Check: use the authorized response plan, restore to a clean environment, validate data and service function, and communicate status; do not experiment on live systems.

## Common mistakes to avoid

- Treating Base64, hex, or URL encoding as encryption, or treating a hash as a reversible secret.
- Confusing authentication with authorization, or assuming a successful login implies broad access.
- Believing HTTPS makes a site trustworthy or its application logic correct; TLS protects a channel under specified assumptions, not every action at either endpoint. [5]
- Treating NAT, a firewall, or any single product as a complete security strategy.
- Assuming a vulnerability automatically means a compromise, or assuming no alert means no compromise.
- Learning commands without understanding their inputs, privileges, and side effects.
- Using security tools against systems without explicit authorization. Even apparently harmless scanning can affect services or violate rules.
- Memorizing a ranked list as if it were timeless. OWASP Top 10 is a changing awareness resource; categories and versions are not a substitute for threat modeling or secure design. [3]

## Concise next-step sequence

1. Practice binary/hex, bytes, text encodings, integer ranges, and safe parsing with toy inputs.
2. Learn Linux files, users, permissions, processes, pipes, and basic networking on a local machine.
3. Write small Python programs; study C, memory, pointers, and file/socket I/O when ready.
4. Trace DNS → IP/TCP or UDP → TLS → HTTP in diagrams and official documentation.
5. Apply least privilege, authentication/authorization distinctions, logging, backups, and risk reasoning to a fictional or local system.
6. Continue into focused modules—web security, communications, cryptography, reverse engineering, or binary exploitation—using official practice environments and staying within their rules. The supplied plan correctly treats these as later branches built on computing foundations. [6]

## References

[1]: https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf "The NIST Cybersecurity Framework (CSF) 2.0"
[2]: https://www.cisa.gov/secure-our-world "CISA Secure Our World"
[3]: https://owasp.org/Top10/2025/en/ "OWASP Top 10:2025"
[4]: https://pages.nist.gov/800-63-4/sp800-63b.html "NIST SP 800-63B: Authentication and Authenticator Management"
[5]: https://www.rfc-editor.org/rfc/rfc8446 "RFC 8446: The Transport Layer Security (TLS) Protocol Version 1.3"
[6]: https://pwn.college/ "pwn.college: Cybersecurity Education Platform"
[7]: https://csrc.nist.gov/pubs/sp/800/30/r1/final "NIST SP 800-30 Rev. 1: Guide for Conducting Risk Assessments"
[8]: https://www.cisa.gov/stopransomware/ransomware-guide "CISA StopRansomware Guide"
