# Cybersecurity: Reverse Engineering

## What reverse engineering is—and why it matters

Software reverse engineering (RE) means examining a program to infer how it is built and behaves when its source code or design is unavailable. In cybersecurity, analysts use it to understand suspicious files, investigate vulnerabilities, verify software behavior, assess firmware, and explain how an incident may have occurred. The goal here is understanding and safe verification, not reproducing harmful behavior.

A compiled executable is not the original source code. It is a structured file containing machine instructions and related data. A processor executes machine instructions; a disassembler renders them as assembly-language mnemonics; a decompiler attempts to reconstruct a higher-level, source-like view. These are different representations of the same underlying program, and neither assembly nor decompiled output automatically tells you the original variable names, comments, or programmer intent. Ghidra describes its capabilities as including disassembly, decompilation, graphing and scripting across processor and executable formats [1].

**Safe-lab boundary:** Only analyze programs you wrote, intentionally vulnerable local lab samples, official practice environments, or files you have explicit written authorization to examine. Do not run an unknown sample on your everyday computer or on a network with sensitive systems. Keep experiments isolated and disposable; do not use analysis as a pretext to access systems, data, or accounts without authorization. For real incidents, follow your organization’s incident-response process and containment rules [6].

## Prerequisites and vocabulary

You do not need to be an expert programmer to begin. It helps to know basic C or another compiled language, binary and hexadecimal notation, and the ideas of variables, conditions, loops, functions, pointers, and arrays. Comfort with a command line is useful. Learn one CPU family first—this chapter uses x86-64 examples—then compare others.

A **binary** is a file represented as bytes; an **executable** is a program file the operating system can load and run. A **byte** is eight bits. **Hexadecimal** (base 16) is a compact way of writing byte values, such as `0x2a`. **Machine code** is the byte-level instruction encoding understood by a processor. **Assembly** is a human-readable notation for those instructions. A **register** is a small, fast storage location inside the CPU. The **instruction pointer** (called `RIP` in x86-64) tracks where execution proceeds; the **stack pointer** (`RSP`) tracks the top of a stack used for calls, return addresses, and temporary data. A **stack frame** is a function’s working area on that stack. The **heap** is a separate region commonly used for dynamically allocated data.

A **symbol** is a name associated with code or data; symbols may be removed (“stripped”) from a shipped program. An **import** is a reference to code or data supplied by another library; an **export** is something a library makes available to other programs. A **calling convention** is the platform-and-architecture agreement for passing arguments, returning values, and preserving registers. Calling conventions vary, so verify the relevant ABI rather than assuming a universal rule.

## A beginner-to-advanced path

### 1. Build an execution model before reading instructions

Think of a program as a state machine: at any moment it has register values, memory contents, and a current instruction address. An instruction changes some part of that state, then execution continues at the next address or branches elsewhere. This mental model is more dependable than trying to “translate” every line into imagined source code.

On x86-64, `mov` copies a value, `lea` computes an address-like expression, `push` and `pop` adjust the stack while saving or restoring a value, `call` transfers control to a function and records a return location, and `ret` returns. `cmp` and `test` set condition flags; a conditional jump uses those flags to choose a path. `jmp` is an unconditional transfer. Arithmetic and bitwise instructions transform values. Consult the processor reference for exact operand, flag, and encoding semantics; mnemonic intuition can be misleading [2].

For a deliberately simple conceptual fragment:

```asm
cmp edi, 0
jle done
add eax, edi
done:
ret
```

Read it as: compare the value in `EDI` with zero; if it is less than or equal to zero, branch to `done`; otherwise add it to `EAX`; then return. The exact source-level types and purpose are not recoverable from these lines alone. A compiler may use different registers, omit a familiar stack frame, combine operations, or transform the logic.

A function call also depends on its calling convention. In a function graph or debugger, track where arguments appear, which register or memory location carries a return value, and which values survive a call. Never infer a convention solely from one code fragment: architecture, operating system, compiler, optimization, and function signature matter.

For a concrete first experiment, compile the same small C function twice (once with debug-friendly settings and once optimized), and compare its disassembly. On common x86-64 System V builds, integer/pointer arguments often begin in `RDI`, `RSI`, `RDX`, `RCX`, `R8`, and `R9`, and an integer result commonly returns in `RAX`; this is an ABI convention, not an instruction-set law, and Windows x64 and other ABIs differ. Check the target ABI before assigning meanings to registers. Optimization can inline a function, eliminate a value, or reorder work, so debug symbols and source lines are aids rather than ground truth [7].

### 2. Learn the executable’s container

An executable is not just a flat instruction list. File formats describe metadata, code and data, addresses, dependencies, and information used by the loader. Common families include ELF on many Unix-like systems, PE on Windows, and Mach-O on Apple platforms. Within ELF, sections organize link-time material such as symbol and relocation tables, while program headers describe segments relevant to loading and memory permissions. The ELF ABI documents headers, sections, symbols, relocations, program headers, dynamic linking, the Global Offset Table (GOT) and Procedure Linkage Table (PLT) [3]. Those structures are related but not interchangeable: a section is not simply a runtime memory segment.

A **relocation** is information used to adjust address-dependent references when code or data is placed or linked. **Static linking** incorporates library code or references during the build; **dynamic linking** uses shared libraries and loader/linker machinery at runtime. GOT and PLT are common ELF mechanisms associated with resolving external references; their detailed operation depends on platform and build choices. Begin by identifying format and architecture, then inspect headers, sections/segments, symbols, imports/exports, and dependencies before interpreting code.

Keep three address notions separate while reading a file: a **file offset** locates bytes on disk; a **virtual address** is an address in the process's memory image; and a **section name** is an organization label, not itself an address. A loadable ELF program header maps a file range into memory; the memory range can be larger, with the excess initialized to zero (common for zero-initialized global data). Sections may be grouped into segments, and stripped binaries can lack a useful section table while retaining program headers needed for loading. When a tool shows an address, ask which address space it means and whether the mapping has a load bias, such as under position-independent loading/ASLR; do not copy a raw file offset into a debugger as if it were a runtime pointer [3] [8].

### 3. Combine static analysis with controlled execution

**Static analysis** means examining a file without executing it: inspect metadata, strings, disassembly, cross-references, and control flow. A **cross-reference** (“xref”) tells you where a code or data address is used. A **control-flow graph** represents possible transfers among basic blocks (straight-line instruction regions). A **call graph** represents function-to-function calls. **Data-flow analysis** follows how values are produced, moved, and consumed. Together, these views help answer focused questions, such as “Which function uses this string?” or “What condition reaches this return?”

**Dynamic analysis** observes a program while it runs in a controlled environment. A debugger can pause execution at a **breakpoint**, inspect registers, stack frames and memory, and step through instructions. A **watchpoint** pauses when a selected value or memory location changes. GDB’s documentation covers running and stopping programs, stack inspection, data, and debugging optimized code [4]. Dynamic observations are evidence about that particular run and environment, not proof that every execution behaves identically.

A practical loop is: form a question, inspect static evidence, choose a safe and narrow dynamic observation if needed, compare results, and record what is known versus inferred. In Ghidra, use the disassembly and decompiler together, inspect function graphs and call trees, follow xrefs, and rename functions only when evidence supports a useful name. Decompiled C-like output is an aid—not recovered original source. Confirm important interpretations against machine instructions and, where safe, runtime observations [1].

Make each hypothesis falsifiable. For example, “this branch appears to reject zero” predicts outcomes for chosen inputs. Identify the comparison, flags or condition, and destination block, then run only your own program with those inputs in a debugger. If prediction fails, revisit signed versus unsigned conditions, operand width, and the path leading to the branch. Record build options and inputs: runtime behavior depends on both. Static analysis can cover paths not reached in one run, while execution can expose behavior static inspection did not make obvious; neither evidence type alone is complete. Label direct observations, interpretations, and unresolved alternatives separately [4] [7].

### 4. Recognize ambiguity and deliberate complexity

Compilers optimize code: they may inline functions, remove unused work, reuse registers, reorder operations, and transform loops. Thus, source-level constructs may not map one-to-one to functions or instruction sequences. A **packed** program stores or compresses some code in a form that is reconstructed during execution; **obfuscation** intentionally makes analysis harder. Examples named in the plan include encrypted strings, control-flow obfuscation, junk instructions, opaque predicates (conditions designed to be difficult to reason about), dynamic API resolution, and self-modifying code. These ideas are useful to recognize defensively, but they do not prove malicious intent by themselves. MITRE ATT&CK describes software packing as an obfuscation technique and documents defensive detection context [5].

At this stage, treat odd-looking code as a question, not a verdict. Compare metadata, code references, execution behavior, and known-good baselines. Do not try to bypass protections or deploy unpacking methods against systems or files outside an authorized lab.

Treat a packer label or unusual region as an indicator, not a diagnosis: legitimate programs also use compression and protection, and custom or modified packers may evade known signatures. A packed file may expose only a loader stub statically; behavior can differ after code is reconstructed in memory. For beginner work, note the clue, explain what evidence is missing, and stop rather than attempt a generic unpacking recipe. Any authorized lab execution should use a disposable isolated environment with no sensitive accounts, shared folders, or unnecessary network access; follow institutional incident handling if material is part of a real case [5] [6].

### 5. Extend to advanced analysis only after fundamentals

Once you can explain ordinary functions and data flow, explore symbolic execution (reasoning about program paths with symbolic rather than fixed input values), concolic execution (combining concrete execution with symbolic constraints), and SMT solving (using a solver for logical constraints). Binary instrumentation and emulation let an analyst observe or model execution; dynamic binary instrumentation (DBI) inserts observation or analysis code during execution. Function similarity and binary diffing help compare versions or related code. Firmware, kernels, and non-x86 architectures such as ARM/AArch64, MIPS, and RISC-V bring different file formats, instruction sets, execution environments, and calling conventions. The same method remains: identify the architecture and format, establish the execution model, test hypotheses against evidence, and document uncertainty.

The plan also names devirtualization, custom loaders, packers, anti-analysis and architecture-specific reversing. These are specialist areas, not prerequisites. Advanced work should stay within isolated, explicitly authorized training targets. Tool interfaces and supported architectures change over time; rely on the current official manuals and project documentation for exact behavior rather than memorizing a particular release’s menus or command syntax [1] [2][4].

## A safe, evidence-led workflow

1. **Establish permission and containment.** Confirm the sample is yours, an official lab artifact, or explicitly authorized. Choose an isolated, disposable environment appropriate to the risk; do not connect unknown samples to sensitive networks or accounts [6].
2. **Preserve and identify.** Keep an untouched copy and record a cryptographic hash, filename, source, acquisition time, and analysis copy. Use harmless metadata tools such as `file` to identify likely format and architecture. A label or extension is not proof of contents. For evidence handled under an organizational process, follow its chain-of-custody and retention requirements.
3. **Survey before interpreting.** Use string extraction and format-aware inspection (`readelf` for ELF, for example), then a disassembler such as `objdump` or Ghidra. Strings are clues, not a behavioral explanation; a lack of readable strings proves little.
4. **Map the program.** Locate entry points, functions, imports, exports, references to interesting data, and branches. Sketch a small call/control-flow graph and write down what each node appears to do, with confidence labels.
5. **Test one hypothesis safely.** If static evidence leaves a specific question, run only a benign, authorized sample in a controlled lab. Prefer your own program first; for another lab sample, confirm written scope and containment before execution. Use GDB or a suitable debugger to inspect registers, stack and memory around a relevant function. Record input, build, environment, and observation; stop if the environment or behavior is not understood. A debugger can perturb behavior, and one run cannot establish what all paths do.
6. **Report and preserve reversibility.** Separate observation (“instruction compares a register with zero”) from interpretation (“likely rejects nonpositive input”). Record tool settings and unresolved questions. Do not patch or modify evidence unless a lab exercise explicitly calls for it; preserve the original.

## Practice tasks and exercises

Use only a program you wrote or a supplied benign training binary in a local lab. The following tasks are designed to have checkable outcomes without contacting any external system.

### Task 1: Inventory a small executable

Compile a tiny “add two integers and print the result” program that you wrote. Use `file`, `strings`, and the format’s inspection tool (`readelf` for ELF); then inspect it in Ghidra or `objdump`.

**Success check:** You can state the architecture and file format based on metadata, identify at least one string or explain why none is present, find a likely entry/function area, and distinguish observed facts from guesses about source-level intent. The result should not depend on an unexplained filename extension.

### Task 2: Explain a branch

In that benign program, add a condition such as “if the first number is positive, add it; otherwise return zero.” Locate the relevant comparison and branch in assembly, then draw two paths through a small control-flow graph.

**Success check:** For one positive and one nonpositive input, your branch explanation predicts which path executes. The assembly-level explanation should mention the comparison/flags and conditional transfer, rather than assuming a particular source statement from one mnemonic.

**Extra check:** Verify signedness and operand width from the actual condition and operands; signed and unsigned conditional jumps differ. Test zero and a negative value, then explain which branch each takes.

### Task 3: Follow a value across a call

Add a helper function that transforms an integer and returns it. In GDB, set a breakpoint on the helper in your own build, inspect the backtrace and relevant registers/stack, step through the function, and inspect the return value. Use a build with debug information for an easier first pass; repeat with optimization only after the basic case makes sense.

**Success check:** You can identify the caller and callee, explain how the input reaches the helper and how the result returns on your platform, and note any differences between optimized and unoptimized builds. Do not generalize the observed calling convention to every platform.

**Extra check:** Compare compiler output with your platform ABI documentation. If the helper disappears into its caller, look for an inlined body or tail call rather than concluding the source function was absent; repeat with optimization disabled for a baseline [7].

### Task 4: Compare static and dynamic evidence

Choose a harmless branch in your program. Predict its behavior from disassembly, then run only your own program under a debugger with two test inputs and compare the observations.

**Success check:** Your note has three parts: static prediction, what happened in each controlled run, and any limitation (for example, only those inputs and that build were tested). A debugger’s output should corroborate, not replace, your reasoning.

### Task 5: Small analyst note

Create a one-page report containing sample identity/hash, format and architecture, three observations with evidence locations, a simple call or control-flow sketch, confidence/unknowns, and lab conditions.

**Success check:** A peer can reproduce the observations in the same authorized lab without relying on claims such as “the decompiler says so.” No secrets or real user data are included.

## Common mistakes to avoid

- Treating decompiler output as original source. It is an approximation; verify key claims in disassembly and execution evidence [1].
- Assuming one instruction equals one source statement, or that every function retains a visible stack frame. Optimization changes the shape of code [4].
- Confusing file offsets, virtual addresses, sections and loadable segments. Use the file-format documentation and tool annotations to establish which address space you are viewing [3].
- Assuming a string, import, suspicious name, or packer indicator proves malicious behavior. It is a lead to investigate, not a conclusion [5].
- Believing a single dynamic run proves all possible behavior. State the inputs, build, and environment you observed.
- Guessing calling conventions or instruction semantics from familiarity. Check the relevant ABI and processor manual [2] [3].
- Treating file offsets, section-relative values, link-time addresses, and runtime virtual addresses as interchangeable. Establish the image mapping and any load bias before comparing tool views [3] [8].
- Treating one successful debugger run as proof a sample is safe or as a complete account of behavior. Containment and the run's limited inputs remain essential [6].
- Making edits to the only sample copy, or running unknown code on a normal workstation. Preserve originals and maintain containment [6].
- Trying to learn every tool at once. Start with one architecture, one format, a few instructions, and small programs you understand.

## Next steps

1. Write and compile tiny programs with functions, branches, loops, strings, and arrays; compare source, assembly, and debugger observations.
2. Learn x86-64 registers, flags, stack behavior, and the platform’s calling convention using the processor manual and ABI.
3. Practice ELF or your operating system’s native format: headers, sections/segments, symbols, relocations, imports and linking.
4. Repeat a static-analysis workflow in Ghidra: functions, graphs, xrefs, strings and decompiler output; then use GDB to test a narrow hypothesis on your own program.
5. Move to official beginner CTF or reverse-engineering labs, keeping activity inside their stated scope. After ordinary binaries feel familiar, choose one advanced area—such as symbolic execution, emulation, or firmware—and study it with its maintained project docs.

## References

[1]: https://github.com/NationalSecurityAgency/ghidra "Ghidra Software Reverse Engineering Framework"
[2]: https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html "Intel® 64 and IA-32 Architectures Software Developer Manuals"
[3]: https://refspecs.linuxfoundation.org/elf/gabi4+/contents.html "System V Application Binary Interface: ELF Object Files and Program Loading"
[4]: https://sourceware.org/gdb/current/onlinedocs/gdb.html/ "Debugging with GDB"
[5]: https://attack.mitre.org/techniques/T1027/002/ "MITRE ATT&CK: Software Packing"
[6]: https://csrc.nist.gov/pubs/sp/800/83/r1/final "NIST SP 800-83 Rev. 1: Guide to Malware Incident Prevention and Handling for Desktops and Laptops"
[7]: https://sourceware.org/gdb/current/onlinedocs/gdb.html/Optimized-Code.html "GDB Manual: Debugging Optimized Code"
[8]: https://man7.org/linux/man-pages/man5/elf.5.html "elf(5): Linux manual page"
