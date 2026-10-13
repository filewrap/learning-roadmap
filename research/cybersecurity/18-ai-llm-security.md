# Cybersecurity: AI and LLM Security

## What this area covers

AI and large language model (LLM) security is the practice of protecting the models, data, software, and people around AI-enabled applications. It combines familiar application-security work—identity, authorization, secure coding, privacy, supply-chain review, and monitoring—with risks arising from models that interpret flexible language and generate probabilistic outputs. A language model is not a conventional policy engine: its response may vary with context, and a sentence that looks like an instruction may be treated as data or vice versa. The security boundary therefore belongs in the surrounding system, not only in a prompt. OWASP’s current LLM risk catalog groups concerns such as prompt injection, disclosure, supply chain, poisoning, improper output handling, excessive agency, vector weaknesses, misinformation, and unbounded consumption. [1]

The goal is not to make every model response perfect. It is to limit what an unexpected response can cause, protect information, evaluate realistic use cases, and recover safely. NIST’s Generative AI Profile places security alongside privacy, information integrity, harmful bias, and other risks, and emphasizes documented evaluation and ongoing review. Its suggested actions are not one-size-fits-all: choose controls based on the actor’s role, use case, and risk tolerance, and document what is in or out of scope. [2]

## Prerequisites and vocabulary

You do not need to train a model to learn this subject. Basic comfort with web applications, HTTP APIs, authentication, authorization, databases, and the idea of trusted versus untrusted input is useful. Familiarity with Python or another scripting language helps when practicing safe validation, but the core threat-modeling ideas are language-independent.

- **Prompt:** The collection of instructions and content presented to a model for a response.
- **System prompt:** Application-provided instructions intended to shape the assistant’s behavior. It is not a secret vault or a reliable authorization boundary.
- **Prompt injection:** Input that changes model behavior in an unintended way. **Direct** injection comes from the user’s message; **indirect** injection arrives inside content the model is asked to process, such as a document or webpage. [3]
- **Jailbreak:** A prompt-injection attempt aimed at getting a model to ignore safety constraints. The labels overlap in common usage; distinguish the intended behavior change rather than relying on the label. [3]
- **Tool use / agency:** The model can request that the application call functions—such as searching, reading records, or sending messages. Agency is the ability to affect the outside world through those capabilities.
- **RAG (retrieval-augmented generation):** The application retrieves relevant passages from a separate knowledge source and supplies them as context to the model. Retrieved material can improve grounding but is still untrusted input.
- **Embedding / vector store:** An embedding is a numeric representation used to compare semantic similarity; a vector store indexes such representations and associated records. Access control must apply to the underlying records, not just to similarity search.
- **Data or model poisoning:** Maliciously or mistakenly influencing training, fine-tuning, or retrieval data so the model or application behaves incorrectly. [1] [4]
- **Adversarial example:** An input crafted to cause a machine-learning system to make an unwanted prediction or response. NIST’s adversarial-ML taxonomy helps separate attack goals, lifecycle stages, and attacker capabilities. [4]
- **Model extraction / inference:** Repeatedly querying a model to infer its behavior or approximate aspects of the model. **Membership inference** asks whether a particular record was included in training. These privacy and intellectual-property questions are distinct from prompt injection. [4]

## A beginner-to-advanced learning path

### 1. Start with the system, not the prompt

Map a typical AI application as a flow: user input → application code → model → retrieval/data sources → optional tools → output rendered to a person or another system. Mark each trust boundary. Ask what data enters, who may access it, what decisions the model influences, and what side effects tools can perform.

A system prompt can help steer behavior, but it is not a substitute for access control. If a user must not read a record, enforce that rule in the data-access layer for every request. Do not expose secrets in prompts and assume that asking the model to keep them private will secure them. OWASP distinguishes prompt leakage and sensitive-information disclosure from the ordinary quality problem of an incorrect answer. [1]

**Safe design rule:** use ordinary deterministic code for authentication, authorization, input constraints, and high-impact decisions. Use the model to interpret or draft, not to grant itself permission.

### 2. Understand prompt injection and trust boundaries

A prompt is an instruction-and-data mixture. A model can confuse untrusted content with directions; such content need not be visibly suspicious, and multimodal systems may process instructions hidden in images or other media. For example, a summarizer asked to summarize a user-supplied memo may encounter a sentence in the memo that tells the assistant to ignore its task. That is an indirect-injection risk even if the sentence is visually ordinary. OWASP notes that RAG and fine-tuning do not eliminate prompt injection; its impact depends heavily on the application’s connected capabilities. There is no dependable prompt-only fix, so design on the assumption that a filter or model safeguard can miss an attack. [3] [7]

Think of the model as a helpful but distractible clerk reading both a work order and papers handed in by visitors. A label saying “visitor paper” helps, but the clerk may still follow a persuasive instruction on that paper. The robust safeguard is to keep the clerk from independently opening restricted files or sending consequential messages. In technical terms, separate and label external content, minimize permissions, and require application-side checks or human approval for consequential actions. Filtering and labeling are useful defense-in-depth, not a substitute for the enforcing control at the tool boundary. [3] [5] [7]

**System-prompt leakage** is worth handling, but the prompt itself should not contain credentials or be treated as a security boundary. If an instruction is important to protect, implement it as an enforceable rule outside the model. [1]

### 3. Secure outputs and tools

Model output is untrusted input to whatever consumes it next. If the application inserts generated text into HTML, SQL, shell commands, file paths, or an email workflow without appropriate validation and context-specific escaping, familiar injection and authorization flaws can arise. OWASP calls this improper output handling. [1]

For a tool-using assistant, do not let model-generated arguments directly authorize an operation. Define a narrow allowlist of tools and schemas; validate arguments in code; check the current user’s authorization independently; use scoped, short-lived credentials; cap resource use; and log security-relevant decisions without credentials or unnecessary sensitive prompt content. A syntactically valid tool call is not necessarily safe: check target, ownership, amount/scope, and caller authority at execution time. For a high-impact action such as sending an external message, show the exact recipient and content in a confirmation step, and bind approval to that precise operation so later model changes cannot silently alter it. This implements least privilege and meaningful human review rather than hoping the model will always decline a risky request. OWASP and Microsoft recommend layered controls and minimal privileges; such controls add friction and need testing for missed attacks and false positives. [3] [5] [7]

**Worked example:** A helpdesk assistant can search tickets. The safe design makes the search service enforce the signed-in user’s ticket permissions, regardless of what the prompt asks. The model may propose a search query, but application code checks the user identity and allowed fields before the query runs. A prompt instruction such as “never show another customer’s ticket” is useful guidance, but cannot replace that check.

### 4. Secure RAG, embeddings, and data

RAG adds a data pipeline: ingestion, chunking, metadata, embeddings, indexing, retrieval, and model context. Threats include a poisoned or stale source, documents retrieved across tenant boundaries, misleading passages, and a vector index that is exposed or poorly permissioned. OWASP tracks vector and embedding weaknesses and data/model poisoning as distinct risks. [1]

Apply data provenance and access policy through the full pipeline. Authenticate sources; review who can add or change documents; retain source identifiers, version, tenant, and access labels; filter retrieval by the caller’s permissions before content reaches the model; and test that deletion, revocation, and tenant separation behave correctly. Filtering only after retrieval is too late: the unauthorized passage may already have influenced the response. Treat retrieved passages as evidence, not instructions. A useful retrieval trace records which document IDs and versions were selected and why, so a wrong or leaked answer can be investigated. Show sources where useful, but verify citations and claims: generated citations can be wrong. NIST recommends grounded data provenance and review of output sources as part of evaluation and monitoring. [2] [7]

**Practice design:** create a tiny local knowledge base with two users and two clearly labeled fictional documents. Test authorized retrieval, unauthorized retrieval, stale content, deletion/revocation, and a document containing an instruction that conflicts with the task. Record retrieved IDs before context construction. Expected outcome: unauthorized content never enters model context; a revoked record stops appearing; the injected instruction is treated as untrusted text; source attribution points to the actual retrieved record and version.

### 5. Abuse, availability, and privacy

An attacker or an accidental user can cause excessive consumption through very long prompts, repeated requests, expensive retrieval, or looping tool calls. Bound input and output sizes, request rates, tool-call counts, timeouts, and per-request work. Provide safe fallbacks and monitor usage. OWASP identifies unbounded consumption as an application risk, not just a model-quality issue. [1]

Data privacy requires a clear inventory of what the application sends to the model, stores in conversation history, places in retrieval indexes, and records in logs. Minimize sensitive data, define retention and deletion behavior, restrict access, and verify provider and organizational controls before use. A model’s response may inadvertently reveal sensitive data, and NIST treats privacy impacts as a generative-AI risk. [2]

### 6. Supply chain, poisoning, and model-level threats

The AI supply chain can include model weights, hosted APIs, libraries, plugins/tools, datasets, evaluation sets, and deployment infrastructure. Track sources and versions, review provenance and licenses, pin and verify dependencies, restrict who can publish artifacts, and test updates before deployment. A model or data source is not trustworthy merely because it is popular or appears in a registry. OWASP’s catalog explicitly covers supply-chain risks and poisoning. [1]

At more advanced levels, distinguish attacks by lifecycle stage, attacker goal, and knowledge rather than treating “AI attack” as one category. Poisoning influences training or other data; a backdoor is a hidden behavior activated by a trigger; evasion seeks an unwanted prediction or response at inference; extraction and membership inference target model or training-data confidentiality. Attacker access and assumptions matter: a result against a locally accessible model does not automatically apply to a hosted API. NIST’s taxonomy organizes attack types, lifecycle stages, capabilities, and mitigations. [4] Such work belongs in controlled experiments with synthetic data and models, never against third-party systems.

### 7. Agents, attack chains, and evaluation

An agent combines a model with tools, memory, planning, and repeated steps. A single injection may therefore influence a chain: untrusted content affects a plan, a tool reads data, and another tool changes state. “Agent hijacking” is best understood as compromising the intended task or control flow, not as a magical model takeover. MITRE ATLAS documents AI-related tactics and techniques, including direct and indirect prompt injection, RAG poisoning, and agent tool invocation; it is a living knowledge base, so use it as a threat-discovery aid rather than a guarantee that every technique applies to every product. [6]

Build an explicit action policy outside the model: which tools may be called, with what arguments, under whose identity, and which operations require confirmation. Limit each step’s permissions and keep read access separate from write access. For multiple agents, treat messages between agents as untrusted data and enforce authorization at each tool boundary; extra agents do not automatically create extra security.

Evaluation should reflect the real deployment context, not only a handful of clever prompts. Maintain a versioned test set covering ordinary tasks, boundary cases, direct and indirect injections (including retrieved/tool-returned content), data access, tool authorization, output handling, privacy, and resource limits. For each case specify setup, user identity, expected outcome, and observable evidence: e.g., denial before a data read, no unauthorized record in model context, or no side effect before exact-action approval. Run these checks after model, prompt, data, tool, or policy changes; track both attack success and legitimate-task failures. NIST cautions against generalizing from narrow anecdotal tests; it recommends documented measurement in conditions similar to deployment, source verification, and ongoing monitoring. [2] Automated red-team tools can help generate and replay tests, but passing a test suite is evidence about those tests—not proof of security. Use frameworks such as MITRE ATLAS to discover potentially relevant threat scenarios, not as a claim that every listed technique applies. [6]

## Common mistakes

- **Treating a prompt as an access-control system.** Prompts guide behavior; application code must enforce permissions.
- **Assuming RAG or fine-tuning cures injection.** Retrieved and trained content may still be misleading or malicious. [3]
- **Giving an agent broad credentials “for convenience.”** Scope tools and credentials to the user, task, and minimum required action. [5]
- **Trusting generated output because it is valid JSON or sounds confident.** Validate semantics and authorization as well as syntax; verify claims and citations. [1] [2]
- **Checking access after retrieval.** Unauthorized material already placed in model context may leak in an answer; enforce authorization before retrieval.
- **Testing only the model.** Vulnerabilities often arise in glue code, tool configuration, retrieval filters, logs, and user interface.
- **Treating one defense as complete.** Filtering, system prompts, classifiers, and human review can each fail; use layered deterministic controls where feasible. [3] [5]
- **Confusing a successful demonstration with a real-world risk estimate.** Test representative workflows, permissions, and failure modes. [2]

## Exercises and answer checks

Perform these only in a deliberately vulnerable local application or an official practice environment. Use synthetic data and no live credentials.

1. **Draw a data-flow map.** Include user, model, retrieval store, tools, and output sink. Mark trusted and untrusted inputs. **Check:** every sensitive read and side effect has an identified enforcement point outside the model.
2. **Review a fictional document-summary workflow.** The input document may contain an instruction that conflicts with summarization. **Check:** the workflow labels retrieved content as untrusted, does not grant it tools, and still returns a summary based on document facts rather than its embedded directions.
3. **Design a tool permission test.** A fictional assistant can look up a ticket and draft a reply. **Check:** the service rejects a ticket outside the user’s authorization; sending is a separate confirmed action; the model cannot choose a broader identity.
4. **Review a RAG index.** Specify ingestion provenance, per-user filters, deletion, and citation checks. **Check:** an unauthorized record is excluded before context construction, and each answer’s cited source resolves to a record actually retrieved.
5. **Create a small evaluation matrix.** Include normal request, malformed input, indirect injection, sensitive-data request, unauthorized tool request, and oversized request. **Check:** each test has an expected outcome, repeatable setup, and a log that captures decision metadata without secrets.
6. **Test action approval.** In a local toy workflow, have the assistant draft (not send) a fictional message. Change the recipient or message after approval. **Check:** execution rejects any operation that differs from the exact operation the user reviewed; cancellation and timeout fail closed.
7. **Review a change.** Change a prompt, model version, retrieval corpus, or tool permission in the toy system and rerun the matrix. **Check:** results are recorded per version, security regressions are visible, and successful tests are not presented as proof against unknown attacks.

## Safe-lab boundary

Hands-on testing must stay inside intentionally vulnerable local labs, official practice environments, systems you own, or systems covered by explicit written authorization. Use fictional content and synthetic credentials. Do not test public AI services, other people’s accounts, or production data without explicit authorization. This chapter focuses on defensive understanding, secure design, and safe verification; it does not authorize real-target exploitation.

## Next steps

1. Learn ordinary web security fundamentals: authentication, authorization, injection prevention, secrets management, and logging.
2. Draw a threat model for one small AI feature, including data flows and tool side effects.
3. Implement deterministic permission checks and output validation around a local toy assistant.
4. Add RAG provenance, access-filter, deletion, and injection test cases.
5. Use OWASP’s risk catalog and MITRE ATLAS to expand a deployment-specific test plan, then measure and revisit it as the system changes. [1] [6]

## References

[1]: https://genai.owasp.org/llm-top-10/ "OWASP Top 10 for LLMs and Gen AI Apps (2025)"
[2]: https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf "Artificial Intelligence Risk Management Framework: Generative Artificial Intelligence Profile"
[3]: https://genai.owasp.org/llmrisk/llm01-prompt-injection/ "LLM01:2025 Prompt Injection - OWASP Gen AI Security Project"
[4]: https://csrc.nist.gov/pubs/ai/100/2/e2025/final "Adversarial Machine Learning: A Taxonomy and Terminology of Attacks and Mitigations"
[5]: https://learn.microsoft.com/en-us/security/zero-trust/sfi/defend-indirect-prompt-injection "Defend against indirect prompt injection attacks - Microsoft Learn"
[6]: https://atlas.mitre.org/ "MITRE ATLAS: Adversarial Threat Landscape for AI Systems"
[7]: https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html "LLM Prompt Injection Prevention Cheat Sheet - OWASP Cheat Sheet Series"
