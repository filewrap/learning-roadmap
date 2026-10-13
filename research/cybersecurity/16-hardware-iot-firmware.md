# Cybersecurity: hardware, IoT, and firmware

## What this area covers

A connected device is not just an app on a network. It is a system: a chip and circuit board, boot code and firmware, sometimes an operating system, local interfaces, and services that communicate with other devices or the cloud. **Firmware** is software closely tied to a device’s hardware; it may initialize components, control sensors, or provide the device’s main functions. **Embedded systems** are computing systems built into a larger product or task. The **Internet of Things (IoT)** is a broad name for physical devices that sense, act, or exchange data over networks.

Hardware, firmware, and network security meet at trust boundaries. A vulnerable web interface can expose a device; an exposed debug port can reveal or alter its software; a weak update process can put malicious or corrupted code into the boot chain. The goal of study is to understand these boundaries, assess authorized devices safely, and design or recommend protections. NIST’s device capability baseline is a useful frame: identity, configuration, data protection, logical access to interfaces, software update, and cybersecurity-state awareness are all part of device security—not optional extras added after networking [1].

This chapter follows the repository plan’s progression: Level 1 foundations (binary formats, firmware extraction, UART/SPI/I²C/GPIO, serial interfaces, embedded Linux, filesystem extraction); Level 2 analysis (ARM/MIPS reversing, emulation, U-Boot, JTAG, flash/EEPROM, hardware debugging, embedded web interfaces, vulnerabilities); Level 3 protections and advanced research (secure boot, TrustZone, hardware security modules, fault injection/glitching, side channels, firmware/bootloader exploitation, SoC and chip-level analysis). These are study topics, not permission to test arbitrary devices.

> **Safe-lab boundary:** Perform hands-on work only on intentionally vulnerable local labs, official practice environments, your own isolated development board/device, or systems covered by explicit written authorization. Do not probe public devices or networks, bypass access controls, access another person’s data, or modify production firmware. For physical work, follow the board and instrument documentation, use appropriate power limits, and avoid connecting unknown voltage levels.

## Prerequisites and vocabulary

You can begin without being an electronics expert. Helpful foundations are basic command-line use, files and directories, binary versus text data, IP networking, and simple programming (C or Python is useful). Learn hexadecimal notation: each pair of hexadecimal digits represents one byte. A **byte** is eight bits. A **binary format** is the layout by which bytes represent fields, instructions, or files; a firmware image may contain several formats nested together.

A **microcontroller (MCU)** commonly integrates a processor, memory, and peripherals for a focused task. A **system-on-chip (SoC)** integrates a broader set of computing components on one chip. **RAM** is working memory; **flash** retains contents without power and often stores firmware; **EEPROM** is non-volatile memory used for smaller data/configuration storage on some designs. These terms are not interchangeable in every device, so check the datasheet and board documentation.

A **bus** is a set of electrical and protocol conventions for communicating. **UART** is an asynchronous serial interface; **SPI** is a clocked serial bus commonly used between a controller and peripherals; **I²C** is a two-wire addressed bus; **GPIO** means general-purpose input/output pins. A pin’s function and voltage depend on the specific board. A **debug interface** is an interface intended to help development or diagnosis; JTAG is one standardized family used for testing/debugging. Its presence, wiring, and access controls vary.

A **bootloader** is early software that loads or starts later software. **Root of trust** means a small trusted starting point from which later security decisions are anchored. A **signature** is a cryptographic authenticity check made with a private key and checked with a corresponding public key; it is not the same thing as encryption. **Threat modeling** is a structured way to identify assets, possible threats, and protections.

## A beginner-to-advanced path

### 1. Begin with the device’s shape and trust boundaries

Draw a simple block diagram: power, processor/SoC, non-volatile storage, RAM, sensors/actuators, debug pins, network interfaces, and any cloud or phone application. Mark the data that matters (credentials, measurements, commands, keys) and who should be able to read or change each item. Ask what happens if a device is physically accessible, disconnected from the internet, or no longer receives updates.

NIST’s baseline gives practical questions: can the device be uniquely identified; can its configuration be changed only by authorized parties; is stored and transmitted data protected; can access to interfaces be controlled; can software be updated securely; and can the device report relevant security state? Use the baseline as a starting point, not a claim that every device needs identical controls [1].

### 2. Learn interfaces before touching hardware

Read a board’s schematic, pin labels, datasheet, and reference manual before connecting anything. UART, SPI, and I²C differ in signaling and protocol; a connector that looks familiar does not prove the pins, voltage, or function. GPIO can control real physical outputs, so careless toggling can have consequences. At this stage, identify interfaces from documentation or a known educational board and record what each is supposed to do.

Make a small interface inventory with columns for location, purpose, expected users, data or control exposed, and documented protection. For a debug port, ask whether it is needed in the shipped product, whether access is authenticated or physically restricted, and whether disabling it would also remove a needed recovery path. “Not exposed in the enclosure” is not equivalent to disabled, and a lock bit is only meaningful if its lifecycle and resistance to reset/rework are understood. Treat all such statements as design questions to verify from documentation or an authorized lab observation—not as invitations to connect to unknown equipment.

Practice reading a serial-console transcript supplied by a lab: distinguish boot messages, prompts, and application logs. Treat a console as a security boundary: an unrestricted maintenance prompt may expose powerful functions, while a locked-down interface may be useful only for diagnostics. Do not attempt to guess passwords on real devices.

### 3. Understand firmware as structured bytes

Firmware is often a container or image composed of headers, metadata, boot stages, a kernel, a root filesystem, configuration, and/or compressed assets. A file extension alone does not establish its format. Start by recording provenance and a cryptographic hash of a lab image; inspect a copy, preserve the original, and note tool output. Signature and file-identification tools can provide clues, but automated identification is not proof of a safe or complete interpretation.

Binwalk is a maintained firmware-analysis tool that identifies and can extract embedded files and data, and offers entropy analysis that may help locate compressed or encrypted regions [2]. Extraction can fail, produce false positives, or expose malformed content; treat every result as a hypothesis to verify. Entropy cannot by itself tell whether bytes are compressed, encrypted, or random. Work in a disposable, unprivileged lab directory and avoid executing extracted programs on your normal host.

Use a repeatable evidence trail: preserve the supplied image read-only, calculate a hash, record the tool/version and exact command, and work on a copy. Validate offsets and partition boundaries against the image layout or vendor/lab documentation; a plausible filesystem listing does not prove that all regions were found. When comparing two authorized firmware versions, compare manifests, partition maps, and hashes to distinguish expected change from unexplained change. Keep conclusions calibrated: “tool reported a filesystem at offset …” is evidence; “this is the complete firmware” needs independent corroboration. Never upload proprietary images or extracted secrets to public analysis services without permission.

Once a filesystem is identified, inventory its files and metadata. Look for configuration, service definitions, certificates, scripts, and libraries in the sample, then ask what function each serves. Do not publish secrets found in a lab image. When analyzing an authorized image, redact credentials and report the risk and remediation rather than reusing them.

### 4. Read the software environment and network surface

Some devices run embedded Linux; others run a small real-time operating system (RTOS), bare-metal code, or a vendor-specific environment. An embedded web interface is still a web application: authentication, authorization, input handling, transport protection, and update functions deserve review. The device may also expose local protocols or cloud APIs. Build an inventory of services and intended data flows from lab documentation rather than scanning unrelated networks.

OWASP’s IoT Security Testing Guide provides a structured testing methodology for IoT devices and is explicitly a project guide whose materials evolve; consult its current project documentation when planning an authorized assessment [3]. A useful defensive review asks: are credentials unique and changeable; are unused services disabled; are updates authenticated; is sensitive data protected; are error messages and logs safe; can the owner recover the device? Record evidence and scope for each finding.

Turn those prompts into a product-lifecycle checklist rather than a one-time port check. ETSI EN 303 645, for example, covers more than passwords: vulnerability-report handling, software updates, secure storage of security parameters, minimized attack surface, software integrity, resilience, and user data deletion are among its consumer-IoT provisions [8]. The standard is a useful baseline, not a guarantee or a universal legal requirement. For each applicable item, record the expected behavior, observed evidence, residual risk, and owner of the fix. A “not applicable” decision should have a reason (for example, no updateable component), not merely be left blank. Ask who receives vulnerability reports, how long updates are supported, and what happens when support ends; a device that is secure at launch can become exposed as dependencies and threats change.

### 5. Introduce architecture and emulation

Machine code is instruction bytes for a processor architecture. ARM and MIPS are processor architecture families named in the repository plan; binaries built for one architecture generally cannot simply run as native programs on another. **Reverse engineering** means inferring how a program works from its behavior or compiled representation. Beginners can first identify architecture and file metadata, inspect strings and symbols when present, and compare behavior in a lab before studying disassembly.

Emulation models hardware or software behavior. QEMU system emulation models a machine—including CPU, memory, and emulated devices—and can run a guest operating system; supported machine/device combinations matter, and options for one target do not necessarily apply to another [4]. Emulation is useful for repeatable observation and debugging, but it is not a perfect reproduction of the physical board: timing, peripherals, boot configuration, and hardware security features may differ. Treat an emulated result as evidence about that model, then state what still needs confirmation on authorized hardware.

### 6. Understand boot, updates, and resilience

A boot chain is a sequence of components that starts the device and verifies or loads later components. **Secure boot** usually refers to enforcing that only authorized code can execute at boot; **verified boot** emphasizes checking authenticity/integrity as code is loaded. Exact product terminology varies. U-Boot documentation describes verified boot as checking software loaded through the boot process, with signatures verified against a public key held in a trusted location; if that key can be tampered with, verification loses its value [5]. Chaining trust means an early stage verifies a later stage, which in turn may verify still later software.

Secure updates must fit this chain. A robust design considers authorization and integrity of an update, compatibility with the device, interruption/power-loss behavior, rollback policy, key protection and rotation, and recovery. A correctly signed vulnerable old image can still be undesirable; version and rollback policy matter. NIST firmware resiliency guidance groups the goals as protecting against unauthorized changes, detecting changes, and recovering securely and rapidly [6]. A signature alone does not guarantee confidentiality, bug-free code, safe recovery, or that a device can receive updates.

For a concrete review, follow one update from publisher to boot: identify how the image is authenticated, what component verifies it, where that verifier’s trust anchor is protected, which partitions/components are covered, and whether the device checks product model and version before installation. Then consider failure cases: interrupted write, full storage, rejected signature, damaged active slot, lost signing key, and attempted installation of an older vulnerable release. A/B slots or a protected recovery image can help availability, but only if selection and recovery themselves are authenticated and tested; atomicity, storage limits, and rollback rules are platform-specific. A hash is useful for detecting change only when its expected value arrives over a trusted channel. A signature establishes provenance/integrity only under sound key custody and policy; it does not establish that the code is safe. These checks make NIST’s protect-detect-recover model operational [6].

### 7. Explore isolation and hardware-backed protections

Hardware security modules (HSMs) and secure elements are terms for hardware that can protect keys or perform security operations; implementations and assurance vary. A trusted execution environment (TEE) aims to isolate sensitive execution from a less trusted environment. Arm TrustZone for AArch64 describes hardware-supported separation into Secure and Non-secure worlds, with system architecture and software required to manage the boundary; it is not an automatic guarantee that an application is secure [7]. Do not conflate TrustZone variants across processor profiles.

**JTAG** and other debug interfaces can provide powerful observation/control during development, so production designs may restrict, authenticate, or disable access according to threat and recovery needs. Flash and EEPROM need protections against unauthorized read/write, but the correct design depends on storage layout, chip features, and physical access assumptions. Learn by studying schematics and vendor documentation or using a deliberately vulnerable training board; never attach probes to an unknown device or attempt invasive access without authorization.

Hardware-backed key storage is a building block, not a complete design. A TPM specification defines capabilities that platform software can use; actual guarantees depend on the component, provisioning, firmware, and how the product uses it [9]. Distinguish **secure boot** (an execution policy that refuses unauthenticated code) from **measured boot** (recording measurements of components, typically for later assessment): measurement alone does not block a component. Similarly, a secure element or HSM can reduce key extraction risk, but cannot fix an application that authorizes the wrong operation or leaks a key after use. When evaluating a claim, ask which key is protected, what can request its use, whether access is gated on device state, and how replacement/revocation works. State the physical-access and attacker assumptions explicitly.

### 8. Advanced topics: learn concepts before techniques

Fault injection deliberately perturbs a system (for example, timing, voltage, or clock conditions) to study whether a security check can be disrupted. “Glitching” commonly refers to brief electrical/timing disturbance, but methods and risks are platform-specific. Side-channel analysis studies unintended information correlated with secret-dependent behavior, such as power use or timing. These subjects require specialized lab controls and can damage equipment or expose sensitive data; at this level focus on threat models, published defensive research, and mitigations rather than instructions for bypassing protections.

Firmware and bootloader vulnerabilities can arise from memory-safety errors, unsafe parsing, weak update validation, exposed maintenance paths, incorrect access control, or flawed key lifecycle. SoC security and chip-level reversing examine boundaries and implementation details below the operating system. A responsible report describes affected asset, conditions, impact, reproducible evidence in the authorized lab, and a fix or mitigation. Do not turn a lab observation into an attempt against a real product.

## Worked reasoning example: evaluating an update design

Suppose a local training device accepts a firmware update. Ask first who is authorized to publish it, then how the device checks that the image came from that authority and was not changed. Locate where the verifier’s public key is anchored and whether ordinary update code can replace it. Trace which boot stages are checked, what happens if power fails midway, whether an older vulnerable image can be installed, and how a failed update is recovered. This maps directly to authenticity, integrity, trust-chain continuity, rollback, and availability. A hash detects accidental change only if its expected value is trusted; a digital signature can authenticate an image only if signing keys and verification policy are managed correctly [5] [6].

## Practice tasks and success checks

1. **Device map (paper exercise):** Choose a documented educational board or a fictional sensor. Draw processor, storage, interfaces, data flows, and trust boundaries. **Success:** each asset has a plausible reader/writer and at least one protection question; unknowns are marked rather than guessed. Add lifecycle questions: who provisions identity, provides updates, receives vulnerability reports, and supports retirement?
2. **Image inventory (local lab image only):** Record the image’s source, hash, format clues, and likely partitions/filesystems using documentation for an analysis tool. **Success:** preserve the original; distinguish confirmed findings from tool hypotheses; do not execute extracted files.
3. **Interface vocabulary:** From a board manual, identify UART, SPI, I²C, and GPIO pins/functions and note the manual’s voltage information. **Success:** explain which are serial buses versus general-purpose pins and do not assume a pinout from connector shape.
4. **Boot-chain and update failure exercise:** Draw ROM/first stage, bootloader, kernel/application, and root filesystem for a documented platform. Mark every verification key and signed component you can confirm. For a hypothetical update, trace signature verification, version/rollback decision, power-loss behavior, and recovery. **Success:** show the root of trust, identify unverified links, and distinguish design documentation from observed lab evidence.
5. **Threat-model review:** For a sample embedded web dashboard, list one asset, one threat, one plausible control, and evidence needed to validate the control. **Success:** keep testing in a local intentionally vulnerable lab or written scope; distinguish design claim from observed evidence.
6. **Emulation comparison:** Use an official QEMU example or supplied lab image and record target machine/architecture and what is emulated. **Success:** explain one thing the emulated model can test and one hardware behavior it cannot establish.
7. **Device assurance checklist:** Using the ETSI provisions as a prompt, choose a documented fictional or training IoT product and mark a few requirements as applicable, not applicable with rationale, or unknown. **Success:** every rating has evidence or a clear evidence gap; do not probe devices outside authorized scope.

## Common mistakes

- Treating every IoT device as embedded Linux; many use RTOS or bare-metal software.
- Treating a file signature, extracted filesystem, or entropy graph as definitive proof of contents.
- Assuming UART/SPI/I²C labels imply safe voltage, correct wiring, or authorization to connect.
- Confusing hashing, encryption, and digital signatures: they address different properties.
- Assuming “secure boot” automatically makes updates safe, prevents all vulnerabilities, or enables recovery.
- Believing emulation perfectly reproduces physical timing, peripherals, or security hardware.
- Focusing only on the chip while ignoring cloud accounts, mobile apps, supply chain, update service, and device retirement.
- Treating a hardware security component as a substitute for correct provisioning, authorization policy, update design, or key lifecycle.
- Treating measured boot as enforcement, or assuming a signed firmware image is safe simply because it is authentic.
- Reporting a vulnerability without scope, evidence, impact, or a mitigation recommendation.

## Concise next-step sequence

1. Review binary/hex basics, command-line file handling, networking, and basic C or Python.
2. Read NIST’s IoT baseline and make a device asset/interface map [1].
3. Study UART/SPI/I²C/GPIO using a board manual; begin with diagrams and safe observation.
4. Analyze a purpose-built firmware sample with Binwalk and verify findings manually [2].
5. Read the OWASP IoT testing guide and practice a written, local-lab assessment plan [3].
6. Learn architecture-specific assembly and use QEMU only with a matching documented target [4].
7. Trace a documented U-Boot verified-boot chain; compare it with NIST’s protect-detect-recover model [5] [6].
8. Pursue TrustZone, JTAG, fault/side-channel research only with suitable training materials, authorization, and lab safeguards [7].
9. Use consumer-IoT guidance to review vulnerability handling, update support, data deletion, and end-of-life alongside device-level controls [8].

## References

[1]: https://csrc.nist.gov/pubs/ir/8259/a/final "NIST IR 8259A: IoT Device Cybersecurity Capability Core Baseline"
[2]: https://github.com/ReFirmLabs/binwalk "ReFirmLabs Binwalk firmware analysis tool"
[3]: https://owasp.org/projects/iot-security-testing-guide "OWASP IoT Security Testing Guide"
[4]: https://www.qemu.org/docs/master/system/introduction.html "QEMU System Emulation: Introduction"
[5]: https://docs.u-boot.org/en/latest/usage/fit/verified-boot.html "U-Boot Verified Boot"
[6]: https://csrc.nist.gov/pubs/sp/800/193/final "NIST SP 800-193: Platform Firmware Resiliency Guidelines"
[7]: https://developer.arm.com/documentation/102418/latest/ "Arm Learn the Architecture: TrustZone for AArch64"
[8]: https://www.etsi.org/deliver/etsi_en/303600_303699/303645/03.01.03_60/en_303645v030103p.pdf "ETSI EN 303 645 V3.1.3: Cyber Security for Consumer Internet of Things"
[9]: https://trustedcomputinggroup.org/resource/tpm-library-specification/ "Trusted Platform Module (TPM) 2.0 Library Specification"
