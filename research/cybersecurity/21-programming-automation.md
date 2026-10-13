# Cybersecurity: Programming and Automation

## What this subject is—and why it matters

Cybersecurity programming is using code to understand, test, monitor, and improve software and systems. Automation means turning repeatable steps into a program or script, so work is consistent, measurable, and less dependent on memory. A small parser can summarize a folder of security logs; a test can check that malformed input is rejected; a local protocol client can help you understand how a deliberately vulnerable training service behaves.

This is a skill that connects many security areas. The supplied roadmap places Python, Bash, regular expressions, file processing, HTTP, JSON, sockets, parsing, and automation at the foundation; then names libraries and tools including pwntools, Scapy, Requests, BeautifulSoup, Capstone, Keystone, Unicorn, Z3, Frida, GDB scripting, and Ghidra scripting; and finally points toward custom debuggers, disassemblers, instrumentation, symbolic execution, constraint solving, automated vulnerability discovery, and research tooling. The progression is useful: first become reliable at handling data and processes, then use focused tools, then build deeper analysis systems.

Automation has two faces. Defenders use it to validate configuration, find patterns in authorized logs, reproduce tests, and check fixes. The same programming concepts can also be misused. MITRE ATT&CK is a knowledge base of adversary tactics and techniques based on real-world observations; its existence is a reminder that automation can support both offense and defense, so authorization and purpose matter as much as code [1].

> **Safe-lab boundary:** Run hands-on security tests only against intentionally vulnerable local labs, official practice environments, systems you own, or systems for which you have explicit written authorization. Keep examples below local and non-destructive. Do not use this material to attack real targets, collect credentials or data, evade detection, establish persistence, deploy malware, or damage systems.

## Prerequisites and vocabulary

You do not need to be an expert programmer. You should be comfortable with variables, conditionals, loops, functions, lists, dictionaries, and reading error messages in Python. Basic command-line use helps. Learn the difference between **text** (characters interpreted with an encoding such as UTF-8) and **bytes** (raw numeric values), because network protocols and binary files are byte sequences until decoded. The pwntools documentation emphasizes the practical importance of this distinction [2].

A **parser** turns input text or bytes into structured values. **Validation** checks whether those values meet defined rules. **Serialization** converts structured values back into a transport or storage format. A **protocol** is an agreed message format and interaction sequence, such as HTTP. A **socket** is an operating-system interface used by programs to communicate over a network. A **process** is a running program; a **subprocess** is one program started by another. A **return code** reports how a process ended. A **test fixture** is controlled sample input used to check program behavior. A **false positive** is an alert that is not a real issue; a **false negative** is a missed issue.

A **trust boundary** is where data crosses from a less-trusted context into one that makes decisions—for example, a request arriving at a service. A **schema** describes expected fields and types. **Syntax validation** checks form; **semantic validation** checks whether a value makes sense in context. These are separate from **authorization**, the check that a user or program is permitted to perform an action. OWASP recommends validating both syntax and meaning, and warns that validation is not a substitute for authorization, parameterized queries, or context-aware output encoding [3].

## A beginner-to-advanced sequence

### 1. Write small, repeatable Python programs

Start with scripts that take explicit input, produce predictable output, and report errors clearly. Prefer standard library functions for files, paths, JSON, regular expressions, and subprocesses before adding packages. For a file-processing script, define its input format, maximum expected size, output format, and failure behavior. Avoid silently skipping malformed records: count or report them in a way that helps a human decide what to do.

A useful pattern is **parse, validate, act**. First parse the data using a parser for its actual format. Next validate required fields, ranges, and relationships. Only then perform the intended action. For example, parsing a number does not prove it is within an acceptable range; OWASP explicitly distinguishes successful parsing from acceptable values and advises size limits before parsing untrusted input [3].

Regular expressions (regexes) are compact patterns for matching text. Use them for simple, constrained formats, and match the whole value rather than a substring when the whole field must conform. Bound the input length and test near-matches: some patterns can consume excessive resources on carefully shaped long inputs [3]. Regex is not a general-purpose HTML parser or a safe way to “remove bad characters.”

### 2. Make structured data and files boringly reliable

JSON is a common text format for nested data. Python's `json` module maps JSON objects to dictionaries and arrays to lists; malformed JSON raises `JSONDecodeError` [4]. Parsing establishes structure, not trust. After parsing, enforce a schema-like set of rules: allowed keys, required fields, expected types, sensible numeric limits, and array length limits. Treat unknown or malformed fields deliberately rather than allowing accidental behavior.

For a safe local exercise, imagine a file of inventory records with fields `asset` and `risk`. Parse each line as JSON, accept only a non-empty string `asset` and `risk` in the set `low`, `medium`, `high`, and write a count by risk category. A robust result includes a count of rejected records and never prints secret-bearing raw records. Add tests for a missing field, invalid JSON, unknown category, and empty input. Expected outcome: valid records are counted, invalid records are rejected predictably, and the script completes without crashing.

### 3. Automate processes without handing them control

Security scripts often call command-line tools. Python's `subprocess.run()` provides a high-level way to run a process, collect output, set a timeout, and inspect a return code [5]. Pass a list of arguments rather than building a shell command from untrusted text; the default `shell=False` avoids asking a shell to interpret that string. Check the program's result and handle timeouts. Keep the executable and options under your control, and avoid running with unnecessary privileges.

Argument separation prevents shell metacharacters from being interpreted by a shell, but it does not make every argument harmless: the called program may treat a value beginning with `-` as an option, or a path may escape the intended directory. Use a fixed executable, validate the argument according to that program's interface, and use a working-directory or resolved-path check when files are involved. Be deliberate about inherited environment variables and captured output; output may contain secrets, and unbounded output can consume memory. Python's `timeout` terminates and waits for the child when it expires, but process creation itself may take longer than the timeout on some platforms [5].

Example (runs a harmless local command with a fixed argument list):

```python
import subprocess

result = subprocess.run(
    ["python3", "--version"],
    capture_output=True,
    text=True,
    timeout=5,
    check=True,
)
print(result.stdout.strip())
```

The example uses a fixed command, not user-supplied command fragments. `check=True` raises an exception for a nonzero return code; a timeout bounds waiting, though process creation itself may not be interruptible immediately on every platform [5]. Do not “fix” command errors by blindly ignoring them. A well-designed automation reports what operation failed and leaves the system in a known state.

### 4. Use HTTP as a structured exchange, not a magic function

HTTP clients send requests and receive a status code, headers, and often a body. For authorized, low-impact work, use a maintained client library rather than hand-building protocol messages. Requests documentation notes that successful JSON decoding does not mean the HTTP operation succeeded: an error response can contain valid JSON. Check the status separately and set a timeout; without an explicit timeout, a request may wait indefinitely [6].

A safe local pattern is to send one request to a local test service you control, then check status, parse expected fields, and handle network and parse errors. Keep URLs fixed or validate them against an explicit local allowlist. Do not build a general-purpose scanner or issue requests to targets without authorization. For HTML, a maintained parser such as BeautifulSoup can extract structure more reliably than regex; still treat extracted content as untrusted data.

Make the request's **scope and budget** explicit: expected host, allowed method, maximum response size, timeout, and whether redirects are acceptable. A timeout bounds waiting for a response but not necessarily total work across retries; if retrying transient failures, cap the attempt count and total elapsed time, and avoid retrying non-idempotent actions blindly. Check the status code before trusting a decoded body, then validate the response schema and any values used to make decisions. These checks make a script's behavior auditable and prevent a successful parse from being mistaken for a successful or authorized operation [6].

### 5. Learn sockets and packet structure conceptually

A socket lets a program exchange bytes with another endpoint. A protocol adds framing and meaning to those bytes: where a message begins and ends, which fields it contains, and what replies mean. Begin with loopback-only client/server exercises that exchange a short fixed greeting. Specify maximum message sizes, timeouts, and how the connection closes. Test malformed or incomplete messages only against your own local program.

Scapy is a maintained packet-manipulation and protocol-analysis framework with documentation covering usage, layers, and adding protocol dissectors [7]. Learn by reading packet captures you generated in an isolated lab or by constructing harmless packets for a local setup. Packet crafting can affect networks; do not send traffic beyond the authorized lab. A packet analyzer is only as trustworthy as its assumptions about lengths, encodings, reassembly, and malformed input.

### 6. Move from scripts to domain tools

Once Python, bytes, process control, and tests feel natural, select tools that fit a specific question:

- **pwntools** is a Python CTF and exploit-development framework for rapid prototyping; its documentation includes process and socket communication (“tubes”), packing, architecture context, and assembly/disassembly helpers [2]. Restrict its use to local intentionally vulnerable exercises and authorized practice targets.
- **Capstone** disassembles machine-code bytes into instruction representations. **Keystone** assembles instruction text into bytes. **Unicorn** emulates processor instructions in a controlled environment. Together, these tools help inspect or test small binary examples without executing them on a real system; understand architecture, address, and calling-convention assumptions before trusting results.
- **GDB scripting** automates debugger observations and repeatable experiments. **Ghidra scripting** can automate navigation, labeling, and analysis over a binary. Start by scripting read-only summaries in toy programs, then verify conclusions manually.
- **Frida** instruments running programs to observe behavior. Instrumentation changes the observation context and may affect behavior; use only on software you own or are explicitly authorized to assess.

The tool names and features evolve. Consult the maintained project documentation for the installed version; do not assume a command or API from an old tutorial still applies. For CTF use, pwntools itself describes its intended audience as CTF and exploit-development work [2].

### 7. Understand symbolic execution and constraint solving

A **symbolic value** represents an unknown input rather than one concrete number. **Symbolic execution** follows program paths while accumulating conditions that inputs must satisfy. A **constraint solver** determines whether a set of mathematical constraints has a solution. **SMT** means satisfiability modulo theories: solving logical formulas that may include arithmetic, bit-vectors, arrays, and other theories.

Z3 is a theorem prover/SMT solver; its guide links an SMT-LIB tutorial and programming materials [8]. A tiny safe example is to ask for integer values `x` and `y` such that `x + y == 10`, `x > 0`, and `y > 0`. The solver returns one satisfying model, not necessarily the only one. This illustrates how constraints encode a question; it does not prove that a larger program is correct. For security analysis, model a small function or a toy validation condition first. State assumptions explicitly, including integer width and signedness when modeling machine arithmetic.

At the advanced end, custom debuggers and disassemblers, binary instrumentation, symbolic execution, SMT modeling, automated vulnerability discovery, and custom protocol analyzers combine these foundations. Automated exploit generation is a research topic with substantial safety implications; in this learning path, treat it as a way to understand why a bug is reachable and how to prevent it, not as a recipe for attacking deployed systems. Research tooling should be reproducible, scoped, auditable, and designed to report evidence rather than perform uncontrolled actions.

## Defensive design habits for every level

Treat every external value—including data from internal feeds—as untrusted. Set limits before expensive parsing or processing; validate expected types, lengths, ranges, and relationships; fail closed when required checks fail. OWASP emphasizes allowlist-oriented field rules and warns that denylisting suspicious characters does not make downstream operations safe [3]. A script that accepts a file path should constrain it to an intended working directory; one that accepts a URL should constrain scheme and destination where appropriate.

Log meaningful events for later diagnosis: what operation was attempted, when, its outcome, and enough context to connect related actions. Do not log passwords, access tokens, keys, or sensitive personal data. OWASP Logging guidance recommends useful event context while identifying secrets and sensitive material that should generally be excluded or masked [9]. Make logs structured where possible, escape untrusted text, and test that errors do not accidentally include secrets.

Test normal cases, boundary values, malformed inputs, timeouts, and repeated runs. Keep test data synthetic. Use version control and code review. NIST's Secure Software Development Framework (SSDF) is a set of high-level practices that can be integrated into different development lifecycles to reduce vulnerabilities and address their causes [10]. The stable lesson is to make security part of preparation, implementation, verification, and response; exact tools and pipeline choices change over time.

For a small Python project, separate pure parsing/validation functions from side effects such as file writes, network requests, and process launches. This makes edge cases inexpensive to test with temporary files and synthetic fixtures, without touching a real host. A useful progression is: unit tests for parsing and policy rules; integration tests against a local fixture or loopback service; then a controlled end-to-end check. Assert both the expected result and the failure behavior (exit status, bounded runtime, and no sensitive values in output). Python's `unittest` supports fixtures, independent cases, and aggregated test runs [11]. A passing test suite is evidence about tested cases, not proof that untested inputs or dependencies are safe.

### Automate the automation safely

Scripts run by a scheduler or continuous-integration (CI) system inherit that environment's privileges. Give each job only the permissions it needs, separate read-only checks from jobs that can publish or deploy, and avoid exposing credentials to untrusted code. For example, a CI job that checks formatting should not receive a write-capable repository token. GitHub's workflow guidance specifically warns that untrusted pull-request content combined with privileged workflow triggers or secrets can compromise a repository; treat event data and downloaded artifacts as untrusted, and review third-party actions and their permissions [12]. This is platform-specific guidance, but least privilege and trust-boundary separation apply to other automation systems too.

Treat credentials as managed lifecycle objects, not constants in source code or command-line arguments. Prefer a secrets manager or the platform's protected secret store, scope each credential narrowly, retrieve it only when required, keep it out of logs and artifacts, and plan how to revoke or rotate it. Redaction is not a substitute for preventing exposure: transformed values and error output may evade masking [12]. OWASP recommends least-privilege access to secrets and automated lifecycle practices such as rotation where practical [13]. If a secret is exposed, stop using it, revoke or rotate it, and inspect relevant logs; deleting a line from source history alone does not invalidate the credential.

## Worked mini-project: a local log summary

Create a directory containing a small synthetic JSON Lines file (one JSON object per line), with fields `time`, `event`, and `result`. Write a Python script that reads only that file, counts each event and result, rejects malformed lines, and prints a summary. Keep the input short and synthetic. Add explicit rules for allowed `result` values, line length, and required fields. Do not print complete malformed lines; report their line numbers and a generic reason. Then add tests that cover valid input, malformed JSON, missing keys, an unexpected result, oversized lines, and empty input. Keep the parser callable as a function so tests can supply fixtures without invoking the command-line entry point.

Success means: the script produces the expected counts for valid rows; rejected rows do not alter counts; the process exits successfully for a valid file; and a test demonstrates predictable handling of each invalid case. Run the tests repeatedly and confirm they do not depend on execution order or files left by earlier runs. Add a small structured log record for the script's own completion, without recording raw input. Finally, run it from a temporary working directory and verify that it reads and writes only the intended paths, and that failure messages contain neither raw records nor secrets. This project practices files, parsing, validation, dictionaries, testing, and safe logging together.

## Exercises and answer checks

1. **JSON validation:** Given `{"asset":"lab-vm","risk":"urgent"}`, explain why successful JSON parsing is not enough. **Check:** `risk` is outside the declared allowed set, so reject it before counting or acting [3] [4].
2. **Subprocess safety:** A script concatenates a user-provided filename into a shell command. Name two improvements. **Check:** pass a fixed executable and separate argument list with `shell=False`; constrain the path to the intended directory, use a timeout, and inspect the return code [5].
3. **HTTP result handling:** A response body parses as JSON but the status is an error. What should the client do? **Check:** treat status and body parsing as separate checks; raise/check the status, handle the error, and set a timeout [6].
4. **Logging:** A failed test response contains a bearer token. What belongs in the log? **Check:** record the event, outcome, and safe correlation/context fields, but omit or mask the token and avoid dumping the full body [9].
5. **Constraint thinking:** For `x + y == 10` with positive integers, give one solution and say whether it is unique. **Check:** `(x, y) = (4, 6)` works; it is not unique because `(1,9)` and other positive pairs work too. A solver returns a model, not a uniqueness proof [8].
6. **CI trust boundary:** A formatting check processes a contribution from an untrusted fork. Should it receive a deployment token? **Check:** no; run untrusted checks without secrets and with minimal read-only permissions, keeping privileged deployment work separate [12].
7. **Secret exposure:** A script accidentally prints a token in a test failure. Is removing the print statement enough? **Check:** no; revoke or rotate the exposed token, inspect logs/artifacts, and prevent recurrence with safe error handling and log review [12] [13].

## Common mistakes

- **Confusing parsing with validation.** A valid JSON document may still contain impossible, overlarge, or disallowed values. Apply field and resource limits.
- **Using regex as a universal parser.** Use a parser designed for JSON, HTML, or a protocol; reserve regex for simple field patterns and bound the input [3].
- **Ignoring timeouts and return codes.** A hung process or failed request can make a report misleading. Bound waiting and verify outcomes [5] [6].
- **Building shell commands from strings.** Shell metacharacters can change how text is interpreted. Keep executable and arguments separate; avoid shell invocation for ordinary subprocess work [5].
- **Assuming HTTP 200, parsed JSON, or an empty exception means the whole workflow is safe.** Verify expected status, schema, and authorization separately [3] [6].
- **Logging everything for convenience.** Logs are another place secrets can leak. Record enough for analysis, not full credentials or raw sensitive bodies [9].
- **Trusting a tool's output as proof.** Disassemblers, emulators, solvers, and scanners reflect assumptions and limits. Record assumptions and confirm critical findings independently.
- **Running a lab tool outside the lab.** Tool capability is not permission. Define the authorized scope before any network or binary experiment.
- **Assuming shell=False makes all arguments safe.** It prevents shell parsing, but the target program still interprets options and paths; validate those according to its own interface [5].
- **Giving every automation job the same credentials.** A low-risk test rarely needs deploy permissions. Separate privilege levels and keep secrets away from untrusted inputs [12] [13].

## Concise next-step sequence

1. Write small Python file-processing scripts with tests for malformed and boundary inputs.
2. Practice safe JSON validation, regex full-value matching, and structured, secret-conscious logs.
3. Add subprocess and HTTP exercises with explicit timeouts, fixed targets, and checked outcomes.
4. Build a loopback-only socket client/server and inspect your own traffic in an isolated lab.
5. Choose one specialist tool (pwntools, Scapy, a disassembler, debugger, or Z3) and complete its beginner material on a local toy target.
6. Test the project at unit and local integration levels; review its file, network, process, logging, and secret boundaries.
7. Read NIST SSDF and use its secure-development practices to review your own small project; only then explore advanced analysis research, always inside written scope [10].

## References

[1]: https://attack.mitre.org/ "MITRE ATT&CK: knowledge base of adversary tactics and techniques"
[2]: https://docs.pwntools.com/en/stable/ "pwntools documentation"
[3]: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html "OWASP Input Validation Cheat Sheet"
[4]: https://docs.python.org/3/library/json.html "Python json — JSON encoder and decoder"
[5]: https://docs.python.org/3/library/subprocess.html "Python subprocess — Subprocess management"
[6]: https://requests.readthedocs.io/en/latest/user/quickstart/ "Requests: Quickstart"
[7]: https://scapy.readthedocs.io/en/latest/ "Scapy documentation"
[8]: https://microsoft.github.io/z3guide/ "Z3 Guide"
[9]: https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html "OWASP Logging Cheat Sheet"
[10]: https://csrc.nist.gov/pubs/sp/800/218/final "NIST SP 800-218: Secure Software Development Framework (SSDF) Version 1.1"
[11]: https://docs.python.org/3/library/unittest.html "Python unittest — Unit testing framework"
[12]: https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions "GitHub Actions: Security hardening for GitHub Actions"
[13]: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html "OWASP Secrets Management Cheat Sheet"
