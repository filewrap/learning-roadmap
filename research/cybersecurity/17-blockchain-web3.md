# Cybersecurity: Blockchain and Web3

## What this area covers

A **blockchain** is a shared digital record maintained by a network of computers rather than a single central database. Transactions are grouped into blocks and linked using cryptographic hashes; participants use a consensus process to agree on the accepted history. NIST describes blockchains as distributed ledgers designed to be tamper-evident and tamper-resistant—not magically impossible to change under every conceivable condition. [1]

**Web3** is a broad, evolving label for applications built around blockchain networks, user-controlled cryptographic accounts, and smart contracts. Ethereum is one such network. Its smart contracts are programs stored and run by the Ethereum Virtual Machine (EVM). Those programs can manage assets or enforce rules, but a flaw in a deployed program can be difficult to correct and losses may be difficult to reverse. [2]

This makes blockchain security a mixture of familiar software security, cryptography, distributed-systems assumptions, and economic design. Learning it helps developers build safer systems and helps analysts reason about incidents without confusing a code defect with a consensus failure or a user-interface scam.

## Safe-lab boundary

Do hands-on work only in an intentionally vulnerable local lab, an official practice environment, systems you own, or systems for which you have explicit written authorization. Use test accounts and valueless test data. Do not probe public contracts, wallets, bridges, or websites without permission; do not attempt to take funds, extract private information, evade monitoring, persist in systems, or disrupt services. Exercises below are design reviews and local simulations, not instructions for attacking real targets.

## Prerequisites and vocabulary

You do not need to know cryptocurrency trading. Helpful foundations are basic programming (conditions, functions, state), networking, and the idea that multiple computers may need to agree on shared data. Read the levels below in order; Solidity syntax can wait until the concepts make sense.

- **Ledger / state:** a ledger records transactions; state is the current result of applying them (for example, account balances).
- **Hash:** a fixed-length digest computed from data. A tiny change in input normally changes the digest. Hashes help link and check records, but do not encrypt data or prove who created it. [1]
- **Public/private key:** a key pair used for cryptographic operations. A signature made with a private key can be checked with its public key. The private key must remain secret; losing control of it can mean losing control of the associated account.
- **Wallet / account:** a wallet is software or hardware that manages keys and signs actions; it is not itself the blockchain balance. Ethereum distinguishes externally owned accounts (controlled by keys) and contract accounts (controlled by code). [3]
- **Transaction:** a signed request to change network state, such as sending value or calling a contract. Ethereum transactions include such fields as sender, destination, nonce, value, input data, and gas parameters. [3]
- **Consensus / finality:** consensus is how network participants settle on a valid history; finality is the protocol's assurance that a block is not expected to be reverted. These are protocol guarantees with assumptions, not absolute physical irreversibility.
- **Smart contract:** code deployed to a blockchain and executed by the network. **Solidity** is a language commonly used for EVM contracts. The **EVM** is the execution environment and instruction set that processes contract calls.
- **Gas:** a measure of computational work charged for execution. It limits resource use and makes a transaction's resource cost visible to its sender. [3]
- **Oracle:** a mechanism that brings information from outside a blockchain (such as an asset price) into contract logic. The oracle and its update assumptions become part of the security boundary. [4]
- **DeFi:** financial applications implemented using blockchain assets and contracts. A **flash loan** is a loan that must be repaid within the same transaction; its atomicity can be useful, but it can also let a transaction temporarily move markets or protocol state in ways a design failed to anticipate. [4]

## 1. How a blockchain records actions

Think of a blockchain as a shared notebook where each accepted page refers to the prior page by its hash. If a past page is altered, its digest and the following links no longer match. Replicas and consensus make undetected rewriting harder. This analogy is useful but incomplete: blockchains differ in how they choose participants, order transactions, handle forks, and decide when history is final. A hash link alone does not guarantee honest consensus, correct software, or truthful external data. [1]

Transactions are requests, not simply lines added to a database. On Ethereum, an account signs a transaction; nodes validate it, execute its state transition, and propagate the result. A nonce helps sequence an account's transactions, while gas bounds computation. A transaction calling a contract may run code whose behavior depends on current state and the transaction's position in a block. [3]

**Practice task:** Draw a flow from a user signing a transaction to a validator including it in a block and nodes applying the state change. Label where signature verification, contract execution, and consensus occur.

**Expected outcome:** You can explain why a valid signature says the key authorized a request, but does not say the contract logic is safe or the external data is correct.

## 2. Keys, wallets, and privacy

A public address is generally safe to share for receiving, but a private key or recovery phrase is equivalent to authority over the account. A signature is not a secret: it is a publicly verifiable authorization over particular data. Before signing, a user should understand what the wallet is being asked to authorize. Blindly approving a message or token allowance can be dangerous.

A token **allowance** is a persistent permission for a named spender contract to transfer up to a specified amount of a token; it is separate from connecting a wallet to a website. An unlimited allowance is convenient but increases the potential loss if the spender is compromised or malicious, and disconnecting a website does not revoke on-chain permission. A user-side review should check the chain, spender identity, token, and amount; grant only what is needed when possible, and revoke stale permissions using a trusted interface (revocation itself is a transaction and costs a fee). [12]

Public chains provide visibility, not anonymity. Contract storage and transaction data should be treated as public, even if source-level variables are marked `private`; that keyword controls access in the programming language, not confidentiality from observers. [5] Do not put passwords, personal records, or supposedly secret randomness on-chain unless a separate, carefully designed privacy system is in use.

**Exercise 1 — Classify the data.** For each item—public address, private key, signed transaction, contract variable marked `private`—say whether it is secret and why.

**Answer check:** The address and signed transaction are public identifiers/data; the private key is secret; the contract variable is visible through public chain state or execution, regardless of the `private` label. [3] [5]

## 3. Smart-contract security: reason about state and authority

A contract is a public program handling adversarial inputs. Security begins by writing down intended invariants: statements that must remain true (for example, “a user can withdraw no more than their recorded balance” or “only an authorized role can change the oracle address”). Review every state-changing path, caller identity, arithmetic boundary, external call, and failure case.

### Access control and logic

Functions exposed as `public` or `external` can be called by others; sensitive operations need explicit, correctly maintained authorization. A missed check can allow unauthorized minting, configuration changes, or withdrawals. The OWASP Smart Contract Top 10 names access-control weaknesses, oracle manipulation, logic errors, invalid input handling, reentrancy, unchecked external calls, flash-loan-related attacks, arithmetic errors, insecure randomness, and denial of service as risk categories. It is an awareness list, not a complete audit checklist or a claim that every contract has each defect. [4]

### Reentrancy and external calls

An external call can hand control to another contract. That callee may call back before the original function completes. A common defense is **Checks-Effects-Interactions**: validate conditions, update internal state, then make the external interaction. This lowers the risk of repeating an operation against stale state, though it is not a substitute for reasoning about all call paths and shared state. [5]

An illustrative withdrawal design first reads and clears the caller's recorded claim, then attempts to send the value and reverts if it fails. If execution reverts, state changes in that transaction are undone. Modern safe design also considers reentrancy guards, pull-payment patterns, and interactions with other contracts. Solidity's documentation emphasizes that calls to other contracts—not only Ether transfers—can create reentrancy. [5]

**Exercise 2 — Find the invariant.** A withdrawal function sends the user's amount and only afterward sets their balance to zero. What is the risk, and what ordering principle helps?

**Answer check:** A receiver contract could call back before the balance is cleared, potentially withdrawing repeatedly. Clear/update state before the external call (Checks-Effects-Interactions) and test adversarial callback behavior in a local test harness. [5]

### Arithmetic, gas, and denial of service

Integer types have finite ranges. Arithmetic can overflow or underflow depending on language version and operation; the correct response is not “newer compiler means arithmetic can be ignored,” but to inspect types, bounds, rounding, and assumptions. OWASP includes integer issues and denial of service among notable categories. [4]

Gas creates availability concerns. A state-dependent loop can grow until a required function exceeds the block's execution budget, potentially preventing users from completing an action. Prefer bounded work, pagination, or user-triggered withdrawal patterns where appropriate, and test realistic maximum states. [5]

### Oracles, flash loans, and economic assumptions

A contract cannot directly know off-chain facts. An oracle's source, freshness, update cadence, manipulation resistance, and fallback behavior all matter. A price feed that can be moved cheaply in the same transaction can cause a lending or trading contract to make unsafe decisions. [4]

Flash loans let a borrower temporarily access assets provided the loan is repaid by transaction end. They are not intrinsically a vulnerability; they expose designs whose invariants rely on a temporary market price, balance, or voting weight being difficult to change. Secure protocol logic should be robust to transient capital and reason about the entire atomic transaction, not just individual calls. [4]

## 4. Upgradeability, delegatecall, and signatures

A deployed contract may be immutable in practice, but a **proxy** pattern can keep state at one address while delegating execution to a replaceable implementation. With `delegatecall`, implementation code executes in the proxy's context: reads and writes affect proxy storage. This power creates risks around who can upgrade, initialization, and storage layout; incompatible layout changes can corrupt state. [6] Treat upgrade authority as a critical permission, disclose it, test upgrades against existing state, and ensure initialization cannot be improperly repeated. [6]

A **signature replay** occurs when a valid signed message is accepted more than once or in a different context than intended. Typed structured signing such as EIP-712 gives messages a readable schema and supports a domain separator that can bind data to a name/version, chain ID, and verifying contract. But EIP-712 explicitly does **not** provide replay protection: the application must enforce one-time use or idempotence, and where appropriate bind an action, expiry, and nonce to the intended context. [10] Always inspect the exact fields shown by the wallet and verify how the contract validates and consumes them; human-readable display alone does not make a malicious request safe.

**Exercise 3 — Review a proxy design.** List three questions before trusting an upgradeable contract.

**Success criteria:** identify who may upgrade; how initialization is restricted; and how storage layout compatibility and post-upgrade state are tested. Strong answers also ask whether the upgrade authority is a single key or governed process. [6]

## 5. Transaction ordering, MEV, and bridges

Transactions can have different effects when executed in different orders. **Maximal extractable value (MEV)** is value obtainable by including, excluding, or changing transaction order in block production beyond standard rewards and fees. It may support useful arbitrage or liquidations, but some ordering strategies worsen users' execution and can raise network-level concerns. [7] Defensive design sets realistic slippage limits, avoids relying on a particular ordering, and evaluates what an observer can infer from a pending transaction. This chapter explains risks, not how to exploit public transactions.

A **bridge** connects separate blockchain systems, typically by relaying messages or representing assets on another chain. The core question is how the destination chain decides that an origin-chain event is genuine. Trust models vary: some rely on a custodian or validator set; others use proofs and verification systems. Each has assumptions beyond the source chain itself. Risks include contract bugs, compromised signers or verifiers, incorrect message handling, and delays or censorship. Ethereum's bridge guide stresses that bridge designs have different trust assumptions and that bridge use carries smart-contract and technology risk. [8]

Consensus security is distinct from contract security: a contract may be perfectly implemented yet depend on a chain whose consensus or finality assumptions fail. Conversely, a healthy consensus protocol cannot repair a flawed contract. Cross-chain systems compose multiple chains and verification mechanisms, so assumptions multiply rather than disappear.

Finality is protocol-specific, not a universal number of confirmations. On Ethereum proof-of-stake, finalized checkpoints require votes representing at least two-thirds of staked ETH; reversing finalized history would entail severe slashing under the protocol's assumptions. Temporary head changes and delayed finalization are distinct from finality, and applications may choose to wait for stronger inclusion assurances according to the cost of a mistaken acceptance. [9] When assessing a chain-dependent workflow, ask which status it treats as sufficient (seen, included, confirmed, or finalized), what happens during a reorganization or stalled finality, and whether delayed settlement is acceptable. [9]

## 6. Testing, analysis, and advanced study

Build confidence in layers, beginning with a short threat model: list assets, privileged roles, external dependencies, trust boundaries, plausible adversarial callers, and the consequence of each failure. Write invariants and expected failure behavior before implementation. Unit tests check selected examples; property-based tests generate many inputs against invariants; fuzzing searches for unexpected combinations; static analysis inspects code without executing every path; symbolic execution explores paths using symbolic values; and formal verification attempts mathematical proofs against an explicit specification. Each method has limits, and a clean tool report is not proof that a system is safe. Ethereum's security guidance recommends testing, analysis, independent review, and treating audits as useful but not infallible. [2]

For a beginner's local test plan, include normal cases, zero/max values, unauthorized callers, repeated calls, failing external recipients, and sequences of multiple operations—not only isolated happy paths. For a stateful property, generate action sequences and assert after each step that, for example, total liabilities do not exceed assets or a claim cannot be paid twice. Record compiler version, dependency versions, and the exact test assumptions so results can be reproduced. OWASP's Smart Contract Security Verification Standard (SCSVS) is a useful way to turn broad areas such as architecture, access control, economic logic, communication, and denial of service into review questions; it is a framework, not a certification that a particular system is safe. [11]

For deeper study, inspect EVM execution and bytecode to understand what deployed code actually does; investigate protocol-level design and DeFi composability; and learn formal methods and symbolic execution. Treat tools, rankings, incident statistics, and ecosystem-specific recommendations as fast-changing. The stable skill is to identify assets, trust boundaries, invariants, adversarial inputs, and failure consequences.

**Practice task:** In a local toy contract, write a one-paragraph threat model and three properties in plain English before implementing tests—for example, “total claims never exceed deposited funds,” “only an authorized role changes the feed,” and “a claim can be paid at most once.” Test ordinary and boundary inputs, unauthorized calls, repeated actions, and a reverting recipient; then add a property-based test for a sequence of deposits and withdrawals. Keep the exercise local or in an official practice environment.

**Expected outcome:** Each property has a test that would fail if the condition were violated, and you can explain what the tests still cannot establish (such as oracle truth, compiler correctness, production configuration, or consensus safety).

## Common mistakes to avoid

- Treating blockchain records as absolutely unchangeable rather than resistant under specified network assumptions. [1]
- Confusing a wallet with funds stored on a device, or sharing a key because an address is public.
- Assuming an audit, test suite, compiler, or analysis tool guarantees safety. [2]
- Believing `private` Solidity state is confidential. [5]
- Checking only happy paths and ignoring callers, reentrancy, failed external calls, rounding, gas growth, and upgrade authority. [4] [5][6]
- Treating flash loans, bridges, or MEV as single bugs rather than examining the protocol's economic and trust assumptions. [4] [7][8]
- Using public deployments or third-party wallets as practice targets without explicit authorization.

## Concise next-step sequence

1. Learn hashes, signatures, state machines, and basic distributed consensus from a foundational overview. [1]
2. Trace Ethereum accounts and signed transactions; explain nonce, gas, and state change in your own words. [3]
3. Study Solidity security and write invariants for a tiny local contract before coding. [5]
4. Use an official local learning environment to test access control, reentrancy, arithmetic, and gas-bound cases.
5. Add static analysis, fuzzing, and independent review; record both findings and tool limitations. [2]
6. Progress to proxies, signature validation, oracles, MEV, and bridges, keeping a written threat model for each. [4] [6][7] [8]
7. Approach EVM reversing, protocol-level research, and formal verification only after fundamentals are solid.
8. For a user-security review, inspect the permissions a wallet request grants and distinguish a site connection from a token allowance; for a protocol review, use a threat model and a structured checklist such as SCSVS. [11] [12]

## References

[1]: https://csrc.nist.gov/pubs/ir/8202/final "NIST IR 8202: Blockchain Technology Overview"
[2]: https://ethereum.org/developers/docs/smart-contracts/security/ "Ethereum.org: Smart contract security"
[3]: https://ethereum.org/developers/docs/transactions/ "Ethereum.org: Transactions"
[4]: https://owasp.org/www-project-smart-contract-top-10/ "OWASP Smart Contract Top 10 (2025)"
[5]: https://docs.soliditylang.org/en/latest/security-considerations.html "Solidity Documentation: Security Considerations"
[6]: https://docs.openzeppelin.com/contracts/5.x/learn/upgrading-smart-contracts "OpenZeppelin Contracts: Learn Upgrading Smart Contracts"
[7]: https://ethereum.org/en/developers/docs/mev/ "Ethereum.org: Maximal extractable value (MEV)"
[8]: https://ethereum.org/en/bridges/ "Ethereum.org: Blockchain bridges"
[9]: https://ethereum.org/developers/docs/consensus-mechanisms/pos/ "Ethereum.org: Proof-of-stake (PoS)"
[10]: https://eips.ethereum.org/EIPS/eip-712 "EIP-712: Typed structured data hashing and signing"
[11]: https://scs.owasp.org/SCSVS/ "OWASP Smart Contract Security Verification Standard (SCSVS)"
[12]: https://ethereum.org/guides/how-to-revoke-token-access/ "Ethereum.org: How to revoke smart contract access to your crypto funds"
